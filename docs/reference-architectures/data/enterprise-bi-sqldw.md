---
title: Enterprise BI com o SQL Data Warehouse
description: Use o Azure para obter insights de negócios de dados relacionais armazenados localmente
author: alexbuckgit
ms.date: 04/13/2018
ms.openlocfilehash: b5e5aa32fc9cc8c7b8b5a42c9a4fc3e0216b2f72
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/16/2018
---
# <a name="enterprise-bi-with-sql-data-warehouse"></a>Enterprise BI com o SQL Data Warehouse
 
Essa arquitetura de referência implementa um pipeline [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (extrair-carregar-transformar) que move os dados de um banco de dados SQL Server local para o SQL Data Warehouse e os transforma para análise. [**Implante essa solução**.](#deploy-the-solution)

![](./images/enterprise-bi-sqldw.png)

**Cenário**: uma organização tem um grande conjunto de dados OLTP armazenado em um banco de dados SQL Server local. A organização deseja usar o SQL Data Warehouse para fazer análise usando o Power BI. 

Essa arquitetura de referência serve para trabalhos únicos ou sob demanda. Se você precisa mover os dados de forma contínua (por hora ou diariamente), recomendamos usar o Azure Data Factory para definir um fluxo de trabalho automatizado.

## <a name="architecture"></a>Arquitetura

A arquitetura consiste nos componentes a seguir.

**SQL Server**. A fonte de dados está localizada em um banco de dados SQL Server local. Para simular o ambiente local, os scripts de implantação dessa arquitetura provisionam uma máquina virtual no Azure com o SQL Server instalado. 

**Armazenamento de Blobs**. O armazenamento de blobs é usado como uma área de transferência para copiar os dados antes de carregá-los no SQL Data Warehouse.

**SQL Data Warehouse do Azure**. O [SQL Data Warehouse](/azure/sql-data-warehouse/) é um sistema distribuído projetado para executar uma análise em grandes quantidades de dados. Ele dá suporte a MPP (processamento altamente paralelo), que o torna adequado para executar análise de alto desempenho. 

**Azure Analysis Services**. O [Analysis Services](/azure/analysis-services/) é um serviço totalmente gerenciado que fornece recursos de modelagem de dados. Use o Analysis Services para criar um modelo semântico de que os usuários podem consultar. O Analysis Services é especialmente útil em um cenário de painel de BI. Nessa arquitetura, o Analysis Services lê dados do data warehouse para processar o modelo semântico e atende a consultas ao painel de maneira eficiente. Ele também dá suporte à simultaneidade elástica, expandindo réplicas para acelerar o processamento das consultas.

Atualmente, o Azure Analysis Services dá suporte a modelos tabulares, mas não a modelos multidimensionais. Os modelos tabulares usam constructos de modelagem relacional (tabelas e colunas), ao passo que os modelos multidimensionais utilizam constructos de modelagem OLAP (cubos, dimensões e medidas). Se você precisar de modelos multidimensionais, use o SSAS (SQL Server Analysis Services). Para saber mais, confira [Comparando soluções tabulares e multidimensionais](/sql/analysis-services/comparing-tabular-and-multidimensional-solutions-ssas).

**Power BI**. O Power BI é um conjunto de ferramentas de análise de negócios para analisar dados a fim de obter informações comerciais. Nessa arquitetura, ele consulta o modelo semântico armazenado no Analysis Services.

O **Azure AD** (Azure Active Directory) autentica os usuários que se conectam ao servidor do Analysis Services por meio do Power BI.

## <a name="data-pipeline"></a>Pipeline de dados
 
Essa arquitetura de referência usa o banco de dados de exemplo [WorldWideImporters](/sql/sample/world-wide-importers/wide-world-importers-oltp-database) como fonte de dados. O pipeline de dados tem os seguintes estágios:

1. Exporte os dados do SQL Server para arquivos simples (utilitário bcp).
2. Copie os arquivos simples para o Armazenamento de Blobs do Azure (AzCopy).
3. Carregue os dados no SQL Data Warehouse (PolyBase).
4. Transforme os dados em um esquema em estrela (T-SQL).
5. Carregue um modelo semântico no Analysis Services (SQL Server Data Tools).

![](./images/enterprise-bi-sqldw-pipeline.png)
 
> [!NOTE]
> Para obter as etapas 1 &ndash; 3, considere o uso do Data Platform Studio da Redgate. O Data Platform Studio aplica-se as correções de compatibilidade e otimizações mais apropriada, portanto, é a maneira mais rápida de começar a usar o SQL Data Warehouse. Para saber mais, confira [Carregar dados com o Data Platform Studio da Redgate](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate). 

As seções a seguir descrevem esses estágios mais detalhadamente.

### <a name="export-data-from-sql-server"></a>Exportar dados do SQL Server

O utilitário [bcp](/sql/tools/bcp-utility) (programa de cópia em massa) é uma maneira rápida de criar arquivos de texto simples de tabelas SQL. Nesta etapa, você seleciona as colunas que você deseja exportar, mas não transforma os dados. As transformações de dados devem ocorrer no SQL Data Warehouse.

**Recomendações**

Se possível, agende a extração de dados fora do horário de pico para minimizar a contenção de recursos no ambiente de produção. 

Evite executar bcp no servidor de banco de dados. Em vez disso, execute-o em outro computador. Grave os arquivos em uma unidade local. Verifique se você te, recursos de E/S suficientes para lidar com as gravações simultâneas. Para um melhor desempenho, exporte os arquivos para unidades de armazenamento rápido dedicadas.

Você pode acelerar a transferência de rede salvando os dados exportados no formato compactado Gzip. No entanto, o carregamento de arquivos compactados no warehouse é mais lento do que o carregamento de arquivos descompactados e, portanto, há uma compensação entre a transferência de rede mais rápida em comparação com o carregamento mais rápido. Se você decidir usar compactação Gzip, não crie um arquivo Gzip único. Em vez disso, divida os dados em vários arquivos compactados.

### <a name="copy-flat-files-into-blob-storage"></a>Copiar arquivos simples no armazenamento de blobs

O utilitário [AzCopy](/azure/storage/common/storage-use-azcopy) foi projetado para cópia de alto desempenho dos dados no armazenamento de blobs do Azure.

**Recomendações**

Crie a conta de armazenamento em uma região perto do local da fonte de dados. Implante a conta de armazenamento e a instância do SQL Data Warehouse na mesma região. 

Não execute o AzCopy no mesmo computador que executa suas cargas de trabalho de produção, pois o consumo de CPU e E/S pode interferir na carga de trabalho de produção. 

Teste o carregamento primeiro para ver como está a velocidade de upload. Você pode usar a opção /NC no AzCopy para especificar o número de operações de cópia simultâneas. Comece com o valor padrão e faça experiências com essa configuração para ajustar o desempenho. Em um ambiente com pouca largura de banda, o grande número de operações simultâneas pode sobrecarregar a conexão de rede, evitando que as operações sejam concluídas.  

O AzCopy move os dados para o armazenamento pela Internet pública. Se isso não for rápido o suficiente, considere configurar um circuito [ExpressRoute](/azure/expressroute/). O ExpressRoute é um serviço que encaminha os dados por uma conexão privada dedicada para o Azure. Outra opção, se sua conexão de rede estiver lenta, é enviar os dados em disco fisicamente para um datacenter do Azure. Para saber mais, confira [Transferindo dados bidirecionalmente no Azure](/azure/architecture/data-guide/scenarios/data-transfer).

Durante uma operação de cópia, o AzCopy cria um arquivo de diário temporário, que permite a ele reiniciar a operação se for interrompido (por exemplo, devido a um erro de rede). Verifique se há espaço em disco suficiente para armazenar os arquivos de diário. Você pode usar a opção /Z para especificar onde os arquivos de diário serão gravados.

### <a name="load-data-into-sql-data-warehouse"></a>Carregar dados no SQL Data Warehouse

Use [PolyBase](/sql/relational-databases/polybase/polybase-guide) para carregar os arquivos do armazenamento de blobs no data warehouse. O PolyBase foi projetado para aproveitar a arquitetura MPP (processamento altamente paralelo) do SQL Data Warehouse, o que o torna a maneira mais rápida de carregar dados no SQL Data Warehouse. 

O carregamento de dados é um processo de duas etapas:

1. Crie um conjunto de tabelas externas para os dados. Uma tabela externa é uma definição de tabela que aponta para dados armazenados fora do warehouse &mdash;, neste caso, os arquivos simples no armazenamento de blobs. Essa etapa não move os dados para o warehouse.
2. Crie tabelas de preparo e carregue os dados nas tabelas de preparo. Essa etapa copia os dados para o warehouse.

**Recomendações**

Quando você tiver grandes quantidades de dados (mais de 1 TB) e estiver executando uma carga de trabalho de análise que se beneficiam de paralelismo, considere o SQL Data Warehouse. O SQL Data Warehouse não é uma boa opção para cargas de trabalho OLTP ou conjuntos de dados menores (< 250GB). Para conjuntos de dados menores que 250GB, considere o Banco de Dados SQL ou o SQL Server. Para saber mais, confira [Armazenagem de Dados](../../data-guide/relational-data/data-warehousing.md).

Crie as tabelas de preparo como tabelas de heap, que não são indexadas. As consultas que criam as tabelas de produção resultarão em uma verificação completa e, portanto, não há motivo para indexar as tabelas de preparo.

O PolyBase se beneficia do paralelismo no warehouse automaticamente. O desempenho de carga é dimensionado conforme você aumenta as DWUs. Para um melhor desempenho, use uma única operação de carregamento. Não há nenhum benefício de desempenho em dividir os dados de entrada em partes e executar vários carregamentos simultâneos.

O PolyBase pode ler arquivos compactados em Gzip. No entanto, somente um único leitor é usado por arquivo compactado, já que a descompactação do arquivo é uma operação de thread único. Portanto, evite o carregamento de um único arquivo compactado grande. Em vez disso, divida os dados em vários arquivos compactados, para se beneficiar do paralelismo. 

Esteja ciente das seguintes limitações:

- O PolyBase dá suporte a um tamanho máximo de coluna de `varchar(8000)`, `nvarchar(4000)` ou `varbinary(8000)`. Se você tiver dados que excedem esses limites, uma opção é dividir os dados em partes quando exportá-los e remontar os fragmentos após a importação. 

- O PolyBase usa um terminador de linha fixo de \n ou newline. Isso poderá causar problemas se houver caracteres de newline na fonte de dados.

- A esquema da fonte de dados pode conter tipos de dados que não têm suporte no SQL Data Warehouse.

Para contornar essas limitações, você pode criar um procedimento armazenado que realiza as conversões necessárias. Faça referência a esse procedimento armazenado ao executar o bcp. Como alternativa, o [Data Platform Studio da Redgate](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate) converte automaticamente os tipos de dados que não têm suporte no SQL Data Warehouse.

Para obter mais informações, consulte os seguintes artigos:

- [Melhores práticas para carregar dados no SQL Data Warehouse do Azure](/azure/sql-data-warehouse/guidance-for-loading-data).
- [Migrar seus esquemas para o SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-migrate-schema)
- [Diretrizes para definir tipos de dados para tabelas no SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types)

### <a name="transform-the-data"></a>Transformar os dados

Transforme os dados e mova-os para tabelas de produção. Nesta etapa, os dados são transformados em um esquema em estrela com tabelas de dimensões e tabelas de fatos, adequadas para modelagem semântica.

Crie as tabelas de produção com índices columnstore clusterizados, que oferecem o melhor desempenho geral de consulta. Os índices ColumnStore são otimizados para consultas que examinam muitos registros. Os índices ColumnStore não tem o mesmo desempenho em pesquisas singleton (isto é, pesquisa de uma única linha). Se você precisar realizar pesquisas singleton frequentes, adicione um índice não clusterizado a uma tabela. As pesquisas singleton podem ficar significativamente mais rápidas usando um índice não clusterizado. No entanto, as pesquisas singleton são geralmente menos comuns em cenários de data warehouse do que as cargas de trabalho OLTP. Para saber mais, confira [Indexando tabelas no SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-index).

> [!NOTE]
> As tabelas columnstore clusterizadas não dão suporte aos tipos de dados `varchar(max)`, `nvarchar(max)` ou `varbinary(max)`. Nesse caso, considere a possibilidade de usar um heap ou índice clusterizado. Você pode colocar essas colunas em uma tabela separada.

Como o banco de dados de exemplo não é muito grande, criamos tabelas replicadas sem partições. Para cargas de trabalho de produção, o uso de tabelas distribuídas provavelmente melhorará o desempenho da consulta. Confira [Diretrizes para criar tabelas distribuídas no SQL Data Warehouse do Azure](/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute). Os scripts de exemplo executam as consultas usando uma [classe de recurso](/azure/sql-data-warehouse/resource-classes-for-workload-management) estática.

### <a name="load-the-semantic-model"></a>Carregar o modelo semântico

Carregue os dados em um modelo tabular no Azure Analysis Services. Nesta etapa, você pode criar um modelo de dados semântico usando o SSDT (SQL Server Data Tools). Você também pode criar um modelo importando-o de um arquivo do Power BI Desktop. Como o SQL Data Warehouse não dá suporte a chaves estrangeiras, você deve adicionar as relações ao modelo semântico para poder unir entre tabelas.

### <a name="use-power-bi-to-visualize-the-data"></a>Usar o Power BI para visualizar os dados

O Power BI dá suporte a duas opções de conexão com o Azure Analysis Services:

- Importação. Os dados são importados para o modelo do Power BI.
- Conexão dinâmica. Os dados são recuperados diretamente do Analysis Services.

Recomendamos a Conexão dinâmica porque ela não requer a cópia de dados no modelo do Power BI. Além disso, o uso do DirectQuery garante que os resultados sejam sempre consistentes com os dados de origem mais recentes. Para saber mais, confira [Conectar com o Power BI](/azure/analysis-services/analysis-services-connect-pbi).

**Recomendações**

Evite executar consultas ao painel de BI diretamente no data warehouse. Os painéis de BI exigem tempos de resposta muito baixos, que as consultas diretas ao warehouse poderão não conseguir atender. Além disso, a atualização do painel afetará o número de consultas simultâneas, o que pode afetar o desempenho. 

O Azure Analysis Services foi projetado para lidar com os requisitos de consulta de um painel de BI e, portanto, a prática recomendada é consultar o Analysis Services do Power BI.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

### <a name="sql-data-warehouse"></a>SQL Data Warehouse

Com o SQL Data warehouse, você pode expandir seus recursos de computação sob demanda. O mecanismo de consulta otimiza as consultas para processamento paralelo com base no número de nós de computação e move dados entre os nós conforme a necessidade. Para saber mais, confira [Gerenciar computação no SQL Data Warehouse do Azure](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview).

### <a name="analysis-services"></a>Serviços de análise

Para cargas de trabalho de produção, recomendamos a camada Standard do Azure Analysis Services, pois ele dá suporte a particionamento e DirectQuery. Dentro de uma camada, o tamanho da instância determina a memória e a potência de processamento. A capacidade de processamento é medida em QPUs (unidades de processamento de consulta). Monitore o uso de QPUs para selecionar o tamanho apropriado. Para saber mais, confira [Monitorar métricas do servidor](/azure/analysis-services/analysis-services-monitor).

Com uma alta carga, o desempenho de consulta pode se degradar devido à simultaneidade de consultas. Você pode expandir o Analysis Services criando um conjunto de réplicas para processar consultas, para que mais consultas possam ser executadas simultaneamente. O trabalho de processamento do modelo de dados sempre ocorre no servidor primário. Por padrão, o servidor primário também lida com consultas. Opcionalmente, você pode designar o servidor primário para executar o processamento de modo exclusivo, para que o pool de consultas lide com todas as consultas. Se você tiver altos requisitos de processamento, separe o processamento do pool de consultas. Se você tiver cargas de consulta pesadas e processamento relativamente leve, poderá incluir o servidor primário no pool de consultas. Para saber mais, confira [Expansão do Azure Analysis Services](/azure/analysis-services/analysis-services-scale-out). 

Para reduzir a quantidade de processamento desnecessária, considere usar partições para dividir o modelo tabular em partes lógicas. Cada partição pode ser processada separadamente. Para saber mais, confira [Partições](/sql/analysis-services/tabular-models/partitions-ssas-tabular).

## <a name="security-considerations"></a>Considerações de segurança

### <a name="ip-whitelisting-of-analysis-services-clients"></a>Lista de permissões de IPs de clientes do Analysis Services

Considere usar o recurso de firewall do Analysis Services para colocar endereços IP do cliente na lista de permissões. Se habilitada, o firewall bloqueará todas as conexões de cliente que não sejam as especificadas nas regras de firewall. As regras padrão colocam o serviço Power BI na lista de permissões, mas você poderá desabilitar essa regra se desejar. Para saber mais, confira [Aumentar a proteção do Azure Analysis Services com o novo recurso de firewall](https://azure.microsoft.com/blog/hardening-azure-analysis-services-with-the-new-firewall-capability/).

### <a name="authorization"></a>Autorização

O Azure Analysis Services usa o Azure AD (Azure Active Directory) para autenticar usuários que se conectam a um servidor do Analysis Services. Você pode restringir quais dados um usuário específico é capaz de exibir criando funções e atribuindo os usuários ou grupos do Azure AD a essas funções. Para cada função, você pode: 

- Proteger tabelas ou colunas individualmente. 
- Proteger linhas individualmente com base em expressões de filtro. 

Para saber mais, confira [Gerenciar usuários e funções de banco de dados](/azure/analysis-services/analysis-services-database-users).

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para essa arquitetura de referência está disponível no [GitHub][ref-arch-repo-folder]. Ela implanta o seguinte:

  * Uma VM Windows para simular um servidor de banco de dados local. Ela inclui o SQL Server 2017 e ferramentas relacionadas, juntamente com o Power BI Desktop.
  * Uma conta de armazenamento do Azure que fornece armazenamento de blobs para armazenar os dados exportados do banco de dados SQL Server.
  * Uma instância do SQL Data Warehouse do Azure.
  * Uma instância do Azure Analysis Services.

### <a name="prerequisites"></a>pré-requisitos

1. Clone, crie fork ou baixe o arquivo zip para as [arquiteturas de referência  do Azure][ref-arch-repo] no repositório GitHub.

2. Instale os [Blocos de Construção do Azure][azbb-wiki] (azbb).

3. Em um prompt de comando, prompt do bash ou prompt do PowerShell, faça logon na sua conta do Azure usando o comando abaixo e siga as instruções.

  ```bash
  az login  
  ```

### <a name="deploy-the-simulated-on-premises-server"></a>Implantar o servidor local simulado

Primeiro, você implantará uma VM como um servidor local simulado, que inclui o SQL Server 2017 e as ferramentas relacionadas. Essa etapa também carrega o exemplo de [banco de dados OLTP da Wide World Importers](/sql/sample/world-wide-importers/wide-world-importers-oltp-database) no SQL Server.

1. Navegue até a pasta `data\enterprise-bi-sqldw\onprem\templates` do repositório que você baixou nos pré-requisitos acima.

2. No arquivo `onprem.parameters.json`, substitua os valores de `adminUsername` e `adminPassword`. Altere também os valores da seção `SqlUserCredentials` para corresponder ao nome de usuário e à senha. Observe o prefixo `.\\` na propriedade userName.
    
    ```bash
    "SqlUserCredentials": {
      "userName": ".\\username",
      "password": "password"
    }
    ```

3. Execute `azbb` conforme mostrado a seguir para implantar o servidor local.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <location> -p onprem.parameters.json --deploy
    ```

4. A implantação pode levar 20 a 30 minutos para ser concluída, o que inclui a execução do script [DSC](/powershell/dsc/overview) para instalar as ferramentas e restaurar o banco de dados. Verifique a implantação no portal do Azure revendo os recursos no grupo de recursos. Você deve ver a máquina virtual `sql-vm1` e seus recursos associados.

### <a name="deploy-the-azure-resources"></a>Implantar os recursos do Azure

Esta etapa provisiona o SQL Data Warehouse do Azure o Azure Analysis Services, junto com uma conta de armazenamento. Se desejar, você pode executar esta etapa em paralelo com a etapa anterior.

1. Navegue até a pasta `data\enterprise-bi-sqldw\azure\templates` do repositório que você baixou nos pré-requisitos acima.

2. Execute o comando da CLI do Azure a seguir para criar um grupo de recursos, substituindo os parâmetros entre colchetes especificados. Observe que você pode implantar em um grupo de recursos diferente do usado para o servidor local na etapa anterior. 

    ```bash
    az group create --name <resource_group_name> --location <location>  
    ```

3. Execute o comando da CLI do Azure a seguir para implantar os recursos do Azure, substituindo os parâmetros entre colchetes especificados. O parâmetro `storageAccountName` deve seguir as [regras de nomenclatura](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) de contas de armazenamento. Para o parâmetro `analysisServerAdmin`, use seu nome UPN do Azure Active Directory.

    ```bash
    az group deployment create --resource-group <resource_group_name> --template-file azure-resources-deploy.json --parameters "dwServerName"="<server_name>" "dwAdminLogin"="<admin_username>" "dwAdminPassword"="<password>" "storageAccountName"="<storage_account_name>" "analysisServerName"="<analysis_server_name>" "analysisServerAdmin"="user@contoso.com"
    ```

4. Verifique a implantação no portal do Azure revendo os recursos no grupo de recursos. Você verá uma conta de armazenamento, a instância do SQL Data Warehouse do Azure e a instância do Analysis Services.

5. Use o portal do Azure para obter a chave de acesso da conta de armazenamento. Selecione a conta de armazenamento para abrir. Em **Configurações**, selecione **Chaves de acesso**. Copie o valor da chave primária. Você o usará na próxima seção.

### <a name="export-the-source-data-to-azure-blob-storage"></a>Exportar os dados de origem para o Armazenamento de Blobs do Azure 

Nesta etapa, você executará um script do PowerShell que usa o bcp para exportar o banco de dados SQL para arquivos simples na máquina virtual e, em seguida, usará o AzCopy para copiar os arquivos no Armazenamento de Blobs do Azure.

1. Use a Área de Trabalho Remota para se conectar à VM local simulada.

2. Enquanto estiver conectado à VM, execute os comandos a seguir em uma janela do PowerShell.  

    ```powershell
    cd 'C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\onprem'

    .\Load_SourceData_To_Blob.ps1 -File .\sql_scripts\db_objects.txt -Destination 'https://<storage_account_name>.blob.core.windows.net/wwi' -StorageAccountKey '<storage_account_key>'
    ```

    Para o parâmetro `Destination`, substitua `<storage_account_name>` pelo nome da conta de armazenamento criada anteriormente. Para o parâmetro `StorageAccountKey`, use a chave de acesso para essa conta de armazenamento.

3. No portal do Azure, verifique se a fonte de dados foi copiada para o armazenamento de blobs navegando para a conta de armazenamento, selecionando o serviço Blob e abrindo o contêiner `wwi`. Você deve ver uma lista de tabelas precedida de `WorldWideImporters_Application_*`.

### <a name="execute-the-data-warehouse-scripts"></a>Executar os scripts de data warehouse

1. Inicie o SSMS (SQL Server Management Studio) da sua sessão da Área de Trabalho Remota. 

2. Conectar ao SQL Data Warehouse

    - Tipo de servidor: mecanismo de banco de dados
    
    - Nome do servidor: `<dwServerName>.database.windows.net`, em que `<dwServerName>` é o nome que você especificou quando implantou os recursos do Azure. Você pode obter esse nome no Portal do Azure.
    
    - Autenticação: autenticação do SQL Server. Use as credenciais que você especificou quando implantou os recursos do Azure nos parâmetros `dwAdminLogin` e `dwAdminPassword`.

2. Navegue até a pasta `C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\azure\sqldw_scripts` na VM. Execute os scripts nesta pasta em ordem numérica, da `STEP_1` à `STEP_7`.

3. Selecione o banco de dados `master` no SSMS e abra o script `STEP_1`. Altere o valor da senha na linha a seguir e execute o script.

    ```sql
    CREATE LOGIN LoaderRC20 WITH PASSWORD = '<change this value>';
    ```

4. Selecione o banco de dados `wwi` no SSMS. Abra o script `STEP_2` e execute-o. Se você receber um erro, verifique se está executando o script em relação ao banco de dados `wwi` e não ao `master`.

5. Abra uma nova conexão com o SQL Data Warehouse usando o usuário e a senha `LoaderRC20` indicados no script `STEP_1`.

6. Usando essa conexão, abra o script `STEP_3`. Defina os seguintes valores no script:

    - SECRET: use a chave de acesso da sua conta de armazenamento.
    - LOCATION: use o nome da conta de armazenamento da seguinte maneira: `wasbs://wwi@<storage_account_name>.blob.core.windows.net`.

7. Usando a mesma conexão, execute os scripts `STEP_4` a `STEP_7` sequencialmente. Verifique se cada script foi concluído com êxito antes de executar o seguinte.

No SMSS, você deve ver um conjunto de tabelas `prd.*` no banco de dados `wwi`. Para verificar se os dados foram gerados, execute a seguinte consulta: 

```sql
SELECT TOP 10 * FROM prd.CityDimensions
```

### <a name="build-the-azure-analysis-services-model"></a>Criar o modelo do Azure Analysis Services

Nesta etapa, você criará um modelo tabular que importa dados do data warehouse. Em seguida, você implantará o modelo no Azure Analysis Services.

1. Inicie o SQL Server Data Tools 2015 na sua sessão da Área de Trabalho Remota.

2. Selecione **Arquivo** > **Novo** > **Projeto**.

3. No diálogo **Novo Projeto**, em **Modelos**, selecione **Business Intelligence** > **Analysis Services**  >  **Projeto Tabular do Analysis Services**. 

4. Nomeie o projeto e clique em **OK**.

5. No diálogo **Designer de modelo tabular**, selecione **Espaço de trabalho integrado** e defina o **Nível de compatibilidade** como `SQL Server 2017 / Azure Analysis Services (1400)`. Clique em **OK**.

6. Na janela **Gerenciador de Modelos Tabulares** janela, clique com o botão direito do mouse e selecione **Importar da Fonte de Dados**.

7. Selecione **SQL Data Warehouse do Azure** e clique em **Conectar**.

8. Para **Servidor**, digite o nome totalmente qualificado do seu servidor do SQL Data Warehouse do Azure. Para **Banco de Dados**, digite `wwi`. Clique em **OK**.

9. No próximo diálogo, escolha a autenticação de **Banco de Dados**, insira seu nome de usuário e senha do SQL Data Warehouse do Azure e clique em **OK**.

10. No diálogo **Navegador**, marque as caixas de seleção **prd. CityDimensions**, **prd. DateDimensions** e **prd. SalesFact**. 

    ![](./images/analysis-services-import.png)

11. Clique em **Carregar**. Quando o processamento for concluído, clique em **Fechar**. Agora você deve ver uma exibição tabular dos dados.

12. Na janela **Gerenciador de Modelos Tabulares**, clique com o botão direito do mouse e selecione **Exibição do Modelo** > **Exibição de Diagrama**.

13. Arraste o campo **[prd.SalesFact].[WWI City ID]**  para o campo **[prd.CityDimensions].[WWI City ID]**  a fim de criar uma relação.  

14. Arraste o campo **[prd.SalesFact].[Invoice Date Key]** para o campo **[prd.DateDimensions].[Date]** .  
    ![](./images/analysis-services-relations.png)

15. No menu **Arquivo**, escolha **Salvar Tudo**.  

16. No **Gerenciador de Soluções**, clique com o botão direito do mouse no projeto e selecione **Propriedades**. 

17. Em **Server**, insira a URL da sua instância do Azure Analysis Services. Você pode obter esse valor no Portal do Azure. No portal, selecione o recurso Analysis Services, clique no painel Visão geral e procure a propriedade **Nome do Servidor**. Ele será semelhante a `asazure://westus.asazure.windows.net/contoso`. Clique em **OK**.

    ![](./images/analysis-services-properties.png)

18. No **Gerenciador de Soluções**, clique com o botão direito do mouse no nome do projeto e selecione **Implantar**. Entre no Azure, se solicitado. Quando o processamento for concluído, clique em **Fechar**.

19. No portal do Azure, exiba os detalhes da sua instância do Azure Analysis Services. Verifique se seu modelo aparece na lista de modelos.

    ![](./images/analysis-services-models.png)

### <a name="analyze-the-data-in-power-bi-desktop"></a>Analisar os dados no Power BI Desktop

Nesta etapa, você usará o Power BI para criar um relatório com os dados no Analysis Services.

1. Inicie o Power BI Desktop na sua sessão de Área de Trabalho Remota.

2. Na tela de boas-vindas, clique em **Obter Dados**.

3. Selecione **Azure** > **Banco de dados do Azure Analysis Services**. Clique em **Conectar**

    ![](./images/power-bi-get-data.png)

4. Insira a URL da instância do Analysis Services e clique em **OK**. Entre no Azure, se solicitado.

5. No diálogo **Navegador**, expanda o projeto tabular implantado, selecione o modelo que você criou e clique em **OK**.

2. No painel **Visualizações**, selecione o ícone **Gráfico de Barras Empilhadas**. No modo de exibição Relatório, redimensione a visualização para aumentá-la.

6. No painel **Campos**, expanda **prd.CityDimensions**.

7. Arraste **prd.CityDimensions** > **WWI City ID** para o **poço Eixo**.

8. Arraste **prd.CityDimensions** > **City** para o poço **Legenda**.

9. No painel Campos, expanda **prd.SalesFact**.

10. Arraste **prd.SalesFact** > **Total com Dedução de Imposto** para o poço **Valor**.

    ![](./images/power-bi-visualization.png)

11. Em **Filtros no Nível Visual**, selecione **WWI City ID**.

12. Defina o **Tipo de Filtro** como `Top N`e defina **Mostrar Itens** como `Top 10`.

13. Arraste **prd.SalesFact** > **Total com Dedução de Imposto** para o poço **Por Valor**

    ![](./images/power-bi-visualization2.png)

14. Clique em **Aplicar Filtro**. A visualização mostra os dez principais totais de vendas por cidade.

    ![](./images/power-bi-report.png)

Para saber mais sobre como usar o Power BI Desktop, confira [Introdução ao Power BI Desktop](/power-bi/desktop-getting-started).

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre como implantar essa arquitetura de referência, visite nosso [Repositório GitHub][ref-arch-repo-folder].
- Saiba mais sobre os [Blocos de Construção do Azure][azbb-repo].

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw

