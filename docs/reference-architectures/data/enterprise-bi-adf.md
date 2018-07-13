---
title: Enterprise BI automatizada com o SQL Data Warehouse e Azure Data Factory
description: Automatizar um fluxo de trabalho ELT no Azure usando o Azure Data Factory
author: MikeWasson
ms.date: 07/01/2018
ms.openlocfilehash: ffd75ba8c57a9afbc6abad61f21f738c644c9bc8
ms.sourcegitcommit: 58d93e7ac9a6d44d5668a187a6827d7cd4f5a34d
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/02/2018
ms.locfileid: "37142261"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a>Enterprise BI automatizada com o SQL Data Warehouse e Azure Data Factory

Essa arquitetura de referência mostra como executar um carregamento incremental em um pipeline [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (extrair-carregar-transformar). Ela usa o Azure Data Factory para automatizar o pipeline ELT. O pipeline move incrementalmente os dados de OLTP mais recentes de um banco de dados do SQL Server local no SQL Data Warehouse. Os dados transacionais são transformados em um modelo de tabela para análise. [**Implante essa solução**.](#deploy-the-solution)

![](./images/enterprise-bi-sqldw-adf.png)

Essa arquitetura é compilada naquela mostrada em [Enterprise BI com o SQL Data Warehouse](./enterprise-bi-sqldw.md), mas também adiciona alguns recursos que são importantes para cenários de data warehouse de dados da empresa.

-   Automação do pipeline usando o Data Factory.
-   Carregamento incremental.
-   Integrando várias fontes de dados.
-   Carregando dados binários como imagens e dados geoespaciais.

## <a name="architecture"></a>Arquitetura

A arquitetura consiste nos componentes a seguir.

### <a name="data-sources"></a>Fontes de dados

**SQL Server local**. A fonte de dados está localizada em um banco de dados SQL Server local. Para simular o ambiente local, os scripts de implantação dessa arquitetura provisionam uma máquina virtual no Azure com o SQL Server instalado. O [banco de dados de exemplo de OLTP de World Wide Importers][wwi] é usado como o banco de dados de origem.

**Dados externos**. Um cenário comum para data warehouses é integrar várias fontes de dados. Essa arquitetura de referência carrega um conjunto de dados externos que contém as populações de cidade por ano e integra-se aos dados do banco de dados OLTP. Você pode usar esses dados para insights, como: “O crescimento das vendas em cada região corresponde ou excede o crescimento demográfico?”

### <a name="ingestion-and-data-storage"></a>Ingestão e armazenamento de dados

**Armazenamento de Blobs**. O armazenamento de blobs é usado como uma área de preparo dos dados de origem antes de carregá-los no SQL Data Warehouse.

**SQL Data Warehouse do Azure**. O [SQL Data Warehouse](/azure/sql-data-warehouse/) é um sistema distribuído projetado para executar uma análise em grandes quantidades de dados. Ele dá suporte a MPP (processamento altamente paralelo), que o torna adequado para executar análise de alto desempenho. 

**Azure Data Factory**. O [Data Factory][adf] é um serviço gerenciado que orquestra e automatiza a movimentação e a transformação de dados. Nessa arquitetura, ele coordena os diversos estágios do processo de ELT.

### <a name="analysis-and-reporting"></a>Análise e relatórios

**Azure Analysis Services**. O [Analysis Services](/azure/analysis-services/) é um serviço totalmente gerenciado que fornece recursos de modelagem de dados. O modelo semântico é carregado no Analysis Services.

**Power BI**. O Power BI é um conjunto de ferramentas de análise de negócios para analisar dados a fim de obter informações comerciais. Nessa arquitetura, ele consulta o modelo semântico armazenado no Analysis Services.

### <a name="authentication"></a>Autenticação

O **Azure AD** (Azure Active Directory) autentica os usuários que se conectam ao servidor do Analysis Services por meio do Power BI.

O Data Factory também pode usar o Azure AD para autenticar no SQL Data Warehouse por meio de uma entidade de serviço ou uma Identidade de Serviço Gerenciada (MSI). Para manter a simplicidade, a implantação de exemplo usa a autenticação do SQL Server.

## <a name="data-pipeline"></a>Pipeline de dados

No [Azure Data Factory][adf], um pipeline é um agrupamento lógico de atividades usadas para coordenar uma tarefa &mdash; nesse caso, carregando e transformando dados no SQL Data Warehouse. 

Essa arquitetura de referência define um pipeline mestre que executa uma sequência de pipelines filho. Cada pipeline filho carrega dados em uma ou mais tabelas do data warehouse.

![](./images/adf-pipeline.png)

## <a name="incremental-loading"></a>Carregamento incremental

Quando você executa um processo automatizado de ETL ou ELT, é mais eficiente carregar apenas os dados alterados desde a execução anterior. Isso é chamado de um *carregamento incremental* em vez de um carregamento completo que carrega todos os dados. Para executar um carregamento incremental, você precisa de uma maneira de identificar quais dados foram alterados. A abordagem mais comum é usar um valor *alto de marca d'água*, o que significa acompanhar o valor mais recente de alguma coluna na tabela de origem, uma coluna de data e hora ou uma coluna de inteiro exclusivo. 

Começando com o SQL Server 2016, você pode usar [tabelas temporais](/sql/relational-databases/tables/temporal-tables). Tratam-se de tabelas com versão do sistema que mantêm um histórico completo das alterações de dados. O mecanismo de banco de dados registra automaticamente o histórico de todas as alterações em uma tabela de histórico separada. Você pode consultar os dados históricos com a adição de uma cláusula FOR SYSTEM_TIME para uma consulta. Internamente, o mecanismo de banco de dados consulta a tabela do histórico, mas isso é transparente ao aplicativo. 

> [!NOTE]
> Para versões anteriores do SQL Server, você pode usar a CDC ([Captura de Dados de Alterações](/sql/relational-databases/track-changes/about-change-data-capture-sql-server)). Essa abordagem é menos conveniente do que as tabelas temporais porque você tem de consultar uma tabela separada, e as alterações são controladas por um número de sequência de log em vez de um carimbo de data/hora. 

Tabelas temporais são úteis para dados de dimensão, os quais podem ser alterados ao longo do tempo. Tabelas de fatos geralmente representam uma transação imutável, como uma venda, caso em que não faz sentido manter o histórico de versão do sistema. Em vez disso, as transações normalmente contam com uma coluna que representa a data da transação, que pode ser usada como o valor de marca d'água. Por exemplo, no banco de dados de OLTP de Wide World Importers, as tabelas Sales.Invoices e Sales.InvoiceLines têm um campo `LastEditedWhen` que é padrão para `sysdatetime()`. 

Aqui está o fluxo geral para o pipeline ELT:

1. Para cada tabela no banco de dados de origem, acompanhe o tempo de corte quando o último trabalho ELT foi executado. Armazene essas informações no data warehouse. (Durante a instalação inicial, todas as horas são definidas como '1-1-1900'.)

2. Durante a etapa de exportação de dados, a hora de corte é passada como um parâmetro para um conjunto de procedimentos armazenados no banco de dados de origem. Esses procedimentos armazenados consultam todos os registros que foram alterados ou criados após a hora de corte. Para a tabela de fatos de Vendas, é usada a coluna `LastEditedWhen`. Para os dados de dimensão, são usadas tabelas temporais com versão do sistema.

3. Quando a migração de dados for concluída, atualize a tabela que armazena os tempos de corte.

Também é útil registrar uma *linhagem* para cada execução de ELT. Para um determinado registro, a linhagem associa esse registro com a execução de ELT que produziu os dados. Para cada execução de ETL, um novo registro de linhagem é criado para cada tabela, mostrando os tempos de carregamento iniciais e finais. As chaves de linhagem para cada registro são armazenadas nas tabelas de dimensões e de fatos.

![](./images/city-dimension-table.png)

Depois que um novo lote de dados for carregado no warehouse, atualize o modelo de tabela do Analysis Services. Consulte [Atualização assíncrona com a API REST](/azure/analysis-services/analysis-services-async-refresh).

## <a name="data-cleansing"></a>Limpeza de dados

A limpeza de dados deve ser parte do processo de ELT. Nessa arquitetura de referência, uma fonte de dados incorretos é a tabela de população da cidade, em que algumas cidades apresentam zero população, talvez porque nenhum dado estivesse disponível. Durante o processamento, o pipeline ELT remove essas cidades da tabela de população da cidade. Execute a limpeza de dados em tabelas de preparo em vez de tabelas externas.

Aqui está o procedimento armazenado que remove as cidades com zero população da tabela de população da cidade. (Você pode encontrar o arquivo de origem [aqui](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql).) 

```sql
DELETE FROM [Integration].[CityPopulation_Staging]
WHERE RowNumber in (SELECT DISTINCT RowNumber
FROM [Integration].[CityPopulation_Staging]
WHERE POPULATION = 0
GROUP BY RowNumber
HAVING COUNT(RowNumber) = 4)
```

## <a name="external-data-sources"></a>Fontes de dados externas

Os data warehouses geralmente consolidam dados de várias fontes. Essa arquitetura de referência carrega uma fonte de dados externa que contém dados demográficos dos dados. Esse conjunto de dados está disponível no armazenamento de blobs do Azure como parte do exemplo [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase).

O Azure Data Factory pode copiar diretamente do armazenamento de blob usando o [conector do armazenamento de blobs](/azure/data-factory/connector-azure-blob-storage). No entanto, o conector requer uma cadeia de conexão ou uma assinatura de acesso compartilhado para que não possa ser usado para copiar um blob com acesso de leitura público. Como alternativa, você pode usar o PolyBase para criar uma tabela externa no armazenamento de blobs e, em seguida, copiar as tabelas externas no SQL Data Warehouse. 

## <a name="handling-large-binary-data"></a>Manipulação de dados binários grandes 

No banco de dados de origem, a tabela de Cidades tem uma coluna de local que contém um tipo de dados espaciais de [geografia](/sql/t-sql/spatial-geography/spatial-types-geography). O SQL Data Warehouse não oferece suporte ao tipo **geografia** nativamente, portanto, esse campo é convertido em um tipo **varbinary** durante o carregamento. (Consulte [Soluções alternativas para tipos de dados sem suporte](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types).)

No entanto, o PolyBase dá suporte a um tamanho máximo de coluna de `varbinary(8000)`, que significa que alguns dados podem ser truncados. Uma solução alternativa para esse problema é dividir os dados em partes durante a exportação e remontar as partes, da seguinte maneira:

1. Crie uma tabela de preparo temporária para a coluna Local.

2. Para cada cidade, divida os dados de localização em partes de 8.000 bytes, resultando em 1 &ndash; N linhas para cada cidade.

3. Para remontar as partes, use o operador T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) para converter linhas em colunas e depois concatenar os valores de coluna para cada cidade.

O desafio é que cada cidade será dividida em um número diferente de linhas, dependendo do tamanho dos dados de geografia. Para que o operador PIVOT funcione, cada cidade deve ter o mesmo número de linhas. Para isso funcionar, a consulta T-SQL (que você pode exibir [here][MergeLocation]) realiza alguns truques para preencher as linhas com valores em branco, de modo que cada cidade tenha o mesmo número de colunas após a dinamização. A consulta resultante acaba sendo muito mais rápida do que executar o loop pelas linhas uma por vez.

A mesma abordagem é usada para dados de imagem.

## <a name="slowly-changing-dimensions"></a>Dimensões de alteração lenta

Dados de dimensão são relativamente estáticos, mas isso pode mudar. Por exemplo, um produto pode ser reatribuído a uma categoria de produto diferente. Há várias abordagens para o tratamento de dimensões de alteração lenta. Uma técnica comum, chamada [Tipo 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), é adicionar um novo registro sempre que uma dimensão for alterada. 

Para implementar a abordagem Tipo 2, as tabelas de dimensões precisam de colunas adicionais que especifiquem o intervalo de datas efetivas para um determinado registro. Além disso, as chaves primárias do banco de dados de origem serão duplicadas, portanto, a tabela de dimensão deve ter uma chave primária artificial.

A imagem a seguir mostra a tabela Dimension.City. A coluna `WWI City ID` é a chave primária do banco de dados de origem. A coluna `City Key` é uma chave artificial gerada durante o pipeline ETL. Além disso, observe que a tabela tem as colunas `Valid From` e `Valid To`, que definem o intervalo de quando cada linha era válida. Os valores atuais têm um `Valid To` igual a “9999-12-31”.

![](./images/city-dimension-table.png)

A vantagem dessa abordagem é que ela preserva os dados históricos, que podem ser valiosos para análise. No entanto, também significa que haverá várias linhas para a mesma entidade. Por exemplo, aqui estão os registros que correspondem a `WWI City ID` = 28561:

![](./images/city-dimension-table-2.png)

Para cada fato de Vendas, você associa o fato a uma única linha na tabela de dimensões Cidade, correspondente à data da nota fiscal. Como parte do processo de ETL, crie uma coluna adicional que 

A consulta de T-SQL a seguir cria uma tabela temporária que associa cada fatura com a chave Cidade correta da tabela de dimensões Cidade.

```sql
CREATE TABLE CityHolder
WITH (HEAP , DISTRIBUTION = HASH([WWI Invoice ID]))
AS
SELECT DISTINCT s1.[WWI Invoice ID] AS [WWI Invoice ID],
                c.[City Key] AS [City Key]
    FROM [Integration].[Sale_Staging] s1
    CROSS APPLY (
                SELECT TOP 1 [City Key]
                    FROM [Dimension].[City]
                WHERE [WWI City ID] = s1.[WWI City ID]
                    AND s1.[Last Modified When] > [Valid From]
                    AND s1.[Last Modified When] <= [Valid To]
                ORDER BY [Valid From], [City Key] DESC
                ) c

```

Essa tabela é usada para preencher uma coluna na tabela de fatos Vendas:

```sql
UPDATE [Integration].[Sale_Staging]
SET [Integration].[Sale_Staging].[WWI Customer ID] =  CustomerHolder.[WWI Customer ID]
```

Essa coluna permite que uma consulta do Power BI encontre o registro correto de Cidade para uma nota fiscal de vendas específica.

## <a name="security-considerations"></a>Considerações de segurança

Para obter mais segurança, você pode usar [pontos de extremidade de serviço de rede Virtual](/azure/virtual-network/virtual-network-service-endpoints-overview) para proteger os recursos de serviço do Azure apenas na sua rede virtual. Isso remove totalmente o acesso público via Internet a esses recursos, permitindo o tráfego somente da sua rede virtual.

Com essa abordagem, você cria uma VNet no Azure e, em seguida, cria pontos de extremidade de serviço privados para serviços do Azure. Esses serviços são então restringidos ao tráfego de rede virtual. Você também pode alcançá-los a partir da sua rede local por meio de um gateway.

Esteja ciente das seguintes limitações:

- No momento que essa arquitetura de referência foi criada, os pontos de extremidade de serviço da VNet contam com suporte para Armazenamento do Azure e SQL Data Warehouse do Azure, mas não para o Azure Analysis Service. Verificar o status mais recente [aqui](https://azure.microsoft.com/updates/?product=virtual-network). 

- Se os pontos de extremidade de serviço estão habilitados para o Armazenamento do Azure, o PolyBase não pode copiar dados do armazenamento no SQL Data Warehouse. Há uma mitigação para esse problema. Para obter mais informações, consulte [Impacto de usar pontos de extremidade de serviço de VNet com Armazenamento do Azure](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage). 

- Para mover dados do local para o Armazenamento do Azure, você precisará listar permissões dos endereços IP públicos do seu local ou ExpressRoute. Para obter detalhes, consulte [Protegendo serviços do Azure em redes virtuais](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).

- Para habilitar o Analysis Services para ler dados do SQL Data Warehouse, implante uma VM do Windows para a rede virtual que contém o ponto de extremidade de serviço do SQL Data Warehouse. Instale [Gateway de Dados Local do Azure](/azure/analysis-services/analysis-services-gateway) nessa VM. Depois conecte seu serviço do Azure Analysis para o gateway de dados.

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para essa arquitetura de referência está disponível em [GitHub][ref-arch-repo-folder]. Ela implanta o seguinte:

  * Uma VM Windows para simular um servidor de banco de dados local. Ela inclui o SQL Server 2017 e ferramentas relacionadas, juntamente com o Power BI Desktop.
  * Uma conta de armazenamento do Azure que fornece armazenamento de blobs para armazenar os dados exportados do banco de dados SQL Server.
  * Uma instância do SQL Data Warehouse do Azure.
  * Uma instância do Azure Analysis Services.
  * O Azure Data Factory e o pipeline do Data Factory para o trabalho ELT.

### <a name="prerequisites"></a>pré-requisitos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="variables"></a>variáveis

As etapas a seguir incluem algumas variáveis definidas pelo usuário. Você precisará substituí-las por valores que você definir.

- `<data_factory_name>`. Nome do Data Factory.
- `<analysis_server_name>`. Nome do servidor do Analysis Services.
- `<active_directory_upn>`. Seu nome UPN do Azure Active Directory. Por exemplo, `user@contoso.com`.
- `<data_warehouse_server_name>`. Nome do servidor do SQL Data Warehouse.
- `<data_warehouse_password>`. Senha de administrador do SQL Data Warehouse.
- `<resource_group_name>`. O nome do grupo de recursos.
- `<region>`. A região do Azure onde os recursos serão implantados.
- `<storage_account_name>`. Nome da conta de armazenamento. Deve seguir as [regras de nomenclatura](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) contas de armazenamento.
- `<sql-db-password>`. Senha de logon do SQL Server.

### <a name="deploy-azure-data-factory"></a>Implantar o Azure Data Factory

1. Navegue até a pasta `data\enterprise_bi_sqldw_advanced\azure\templates` do [GitHub repository][ref-arch-repo].

2. Execute o seguinte comando da CLI do Azure para criar um grupo de recursos.  

    ```bash
    az group create --name <resource_group_name> --location <region>  
    ```

    Especifique uma região que dê suporte ao SQL Data Warehouse, Azure Analysis Services e Data Factory v2. Consulte [Produtos do Azure por região](https://azure.microsoft.com/global-infrastructure/services/)

3. Execute o comando a seguir

    ```
    az group deployment create --resource-group <resource_group_name> \
        --template-file adf-create-deploy.json \
        --parameters factoryName=<data_factory_name> location=<location>
    ```

Depois, use o portal do Azure para obter a chave de autenticação para o [tempo de execução de integração](/azure/data-factory/concepts-integration-runtime) do Azure Data Factory, da seguinte maneira:

1. No [portal do Azure](https://portal.azure.com/), navegue até a instância do Data Factory.

2. Na folha Data Factory, clique em **Criar e Monitorar**. Isso abre o portal do Azure Data Factory em outra janela do navegador.

    ![](./images/adf-blade.png)

3. No portal do Azure Data Factory, selecione o ícone de lápis (“Autor”). 

4. Clique em **Conexões**, depois selecione **Integration Runtimes**.

5. Sob **sourceIntegrationRuntime**, clique no ícone de lápis (“Editar”).

    > [!NOTE]
    > O portal mostrará o status como “não disponível”. Isso é esperado até que você implante o servidor local.

6. Encontre **Key1** e copie o valor da chave de autenticação.

Você precisará da chave de autenticação para a próxima etapa.

### <a name="deploy-the-simulated-on-premises-server"></a>Implantar o servidor local simulado

Essa etapa implanta uma VM como um servidor local simulado, que inclui o SQL Server 2017 e as ferramentas relacionadas. Também carrega o exemplo de [Wide World Importers OLTP database][wwi] no SQL Server.

1. Navegue até a pasta `data\enterprise_bi_sqldw_advanced\onprem\templates` do repositório.

2. No arquivo `onprem.parameters.json`, procure `adminPassword`. Essa é a senha para fazer logon na VM do SQL Server. Substitua o valor com outra senha.

3. No mesmo arquivo, procure `SqlUserCredentials`. Essa propriedade especifica as credenciais da conta do SQL Server. Substitua a senha por um valor diferente.

4. No mesmo arquivo, cole a chave de autenticação do Integration Runtime no parâmetro `IntegrationRuntimeGatewayKey`, conforme mostrado abaixo:

    ```json
    "protectedSettings": {
        "configurationArguments": {
            "SqlUserCredentials": {
                "userName": ".\\adminUser",
                "password": "<sql-db-password>"
            },
            "IntegrationRuntimeGatewayKey": "<authentication key>"
        }
    ```

5. Execute o comando a seguir.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.parameters.json --deploy
    ```

Essa etapa pode levar de 20 a 30 minutos para ser concluída. Ela inclui a execução de um script [DSC](/powershell/dsc/overview) para instalar as ferramentas e restaurar o banco de dados. 

### <a name="deploy-azure-resources"></a>Implantar recursos do Azure

Essa etapa provisiona o SQL Data Warehouse, o Azure Analysis Services e o Data Factory.

1. Navegue até a pasta `data\enterprise_bi_sqldw_advanced\azure\templates` do [GitHub repository][ref-arch-repo].

2. Execute o comando da CLI do Azure a seguir. Substitua os valores de parâmetro mostrados entre colchetes angulares.

    ```bash
    az group deployment create --resource-group <resource_group_name> \
     --template-file azure-resources-deploy.json \
     --parameters "dwServerName"="<data_warehouse_server_name>" \
     "dwAdminLogin"="adminuser" "dwAdminPassword"="<data_warehouse_password>" \ 
     "storageAccountName"="<storage_account_name>" \
     "analysisServerName"="<analysis_server_name>" \
     "analysisServerAdmin"="<user@contoso.com>"
    ```

    - O parâmetro `storageAccountName` deve seguir as [regras de nomenclatura](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) de contas de armazenamento. 
    - Para o parâmetro `analysisServerAdmin`, use seu nome UPN do Azure Active Directory.

3. Execute o seguinte comando da CLI do Azure para obter a chave de acesso da conta de armazenamento. Você usará essa chave na próxima etapa.

    ```bash
    az storage account keys list -n <storage_account_name> -g <resource_group_name> --query [0].value
    ```

4. Execute o comando da CLI do Azure a seguir. Substitua os valores de parâmetro mostrados entre colchetes angulares. 

    ```bash
    az group deployment create --resource-group <resource_group_name> \
    --template-file adf-pipeline-deploy.json \
    --parameters "factoryName"="<data_factory_name>" \
    "sinkDWConnectionString"="Server=tcp:<data_warehouse_server_name>.database.windows.net,1433;Initial Catalog=wwi;Persist Security Info=False;User ID=adminuser;Password=<data_warehouse_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" \
    "blobConnectionString"="DefaultEndpointsProtocol=https;AccountName=<storage_account_name>;AccountKey=<storage_account_key>;EndpointSuffix=core.windows.net" \
    "sourceDBConnectionString"="Server=sql1;Database=WideWorldImporters;User Id=adminuser;Password=<sql-db-password>;Trusted_Connection=True;"
    ```

    As cadeias de conexão têm subcadeias de caracteres mostradas entre colchetes angulares que devem ser substituídos. Para `<storage_account_key>`, use a chave que você obteve na etapa anterior. Para `<sql-db-password>`, use a senha da conta do SQL Server que você especificou no arquivo `onprem.parameters.json` anteriormente.

### <a name="run-the-data-warehouse-scripts"></a>Executar os scripts de data warehouse

1. No [Portal do Azure](https://portal.azure.com/), localize a VM local, que é chamada de `sql-vm1`. O nome de usuário e a senha para a VM são especificadas no arquivo `onprem.parameters.json`.

2. Clique em **Conectar** e use a Área de Trabalho Remota para se conectar à VM.

3. Na sua sessão de Área de Trabalho Remota, abra um prompt de comando e navegue até a pasta a seguir na VM:

    ```
    cd C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw_advanced\azure\sqldw_scripts
    ```

4. Execute o comando a seguir:

    ```
    deploy_database.cmd -S <data_warehouse_server_name>.database.windows.net -d wwi -U adminuser -P <data_warehouse_password> -N -I
    ```

    Para `<data_warehouse_server_name>` e `<data_warehouse_password>`, use o nome do servidor do data warehouse e a senha de antes.

Para verificar essa etapa, é possível usar o SSMS (SQL Server Management Studio) para se conectar ao banco de dados do SQL Data Warehouse. Você deve ver os esquemas de tabela do banco de dados.

### <a name="run-the-data-factory-pipeline"></a>Executar o pipeline da Data Factory

1. Na mesma sessão de Área de Trabalho Remota, abra uma janela do PowerShell.

2. Execute o comando do PowerShell a seguir. Clique em **Sim** quando for solicitado.

    ```powershell
    Install-Module -Name AzureRM -AllowClobber
    ```

3. Execute o comando do PowerShell a seguir. Inserir as credenciais do Azure quando solicitado.

    ```powershell
    Connect-AzureRmAccount 
    ```

4. Execute os comandos do PowerShell a seguir. Substitua os valores entre colchetes angulares.

    ```powershell
    Set-AzureRmContext -SubscriptionId <subscription id>

    Invoke-AzureRmDataFactoryV2Pipeline -DataFactory <data-factory-name> -PipelineName "MasterPipeline" -ResourceGroupName <resource_group_name>

5. In the Azure Portal, navigate to the Data Factory instance that was created earlier.

6. In the Data Factory blade, click **Author & Monitor**. This opens the Azure Data Factory portal in another browser window.

    ![](./images/adf-blade.png)

7. In the Azure Data Factory portal, click the **Monitor** icon. 

8. Verify that the pipeline completes successfully. It can take a few minutes.

    ![](./images/adf-pipeline-progress.png)


## Build the Analysis Services model

In this step, you will create a tabular model that imports data from the data warehouse. Then you will deploy the model to Azure Analysis Services.

**Create a new tabular project**

1. From your Remote Desktop session, launch SQL Server Data Tools 2015.

2. Select **File** > **New** > **Project**.

3. In the **New Project** dialog, under **Templates**, select  **Business Intelligence** > **Analysis Services** > **Analysis Services Tabular Project**. 

4. Name the project and click **OK**.

5. In the **Tabular model designer** dialog, select **Integrated workspace**  and set **Compatibility level** to `SQL Server 2017 / Azure Analysis Services (1400)`. 

6. Click **OK**.


**Import data**

1. In the **Tabular Model Explorer** window, right-click the project and select **Import from Data Source**.

2. Select **Azure SQL Data Warehouse** and click **Connect**.

3. For **Server**, enter the fully qualified name of your Azure SQL Data Warehouse server. You can get this value from the Azure Portal. For **Database**, enter `wwi`. Click **OK**.

4. In the next dialog, choose **Database** authentication and enter your Azure SQL Data Warehouse user name and password, and click **OK**.

5. In the **Navigator** dialog, select the checkboxes for the **Fact.\*** and **Dimension.\*** tables.

    ![](./images/analysis-services-import-2.png)

6. Click **Load**. When processing is complete, click **Close**. You should now see a tabular view of the data.

**Create measures**

1. In the model designer, select the **Fact Sale** table.

2. Click a cell in the the measure grid. By default, the measure grid is displayed below the table. 

    ![](./images/tabular-model-measures.png)

3. In the formula bar, enter the following and press ENTER:

    ```
    Total Sales:=SUM('Fact Sale'[Total Including Tax])
    ```

4. Repeat these steps to create the following measures:

    ```
    Number of Years:=(MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber]))+1
    
    Beginning Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MIN('Fact CityPopulation'[YearNumber])))
    
    Ending Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MAX('Fact CityPopulation'[YearNumber])))
    
    CAGR:=IFERROR((([Ending Population]/[Beginning Population])^(1/[Number of Years]))-1,0)
    ```

    ![](./images/analysis-services-measures.png)

For more information about creating measures in SQL Server Data Tools, see [Measures](/sql/analysis-services/tabular-models/measures-ssas-tabular).

**Create relationships**

1. In the **Tabular Model Explorer** window, right-click the project and select **Model View** > **Diagram View**.

2. Drag the **[Fact Sale].[City Key]** field to the **[Dimension City].[City Key]** field to create a relationship.  

3. Drag the **[Face CityPopulation].[City Key]** field to the **[Dimension City].[City Key]** field.  

    ![](./images/analysis-services-relations-2.png)

**Deploy the model**

1. From the **File** menu, choose **Save All**.

2. In **Solution Explorer**, right-click the project and select **Properties**. 

3. Under **Server**, enter the URL of your Azure Analysis Services instance. You can get this value from the Azure Portal. In the portal, select the Analysis Services resource, click the Overview pane, and look for the **Server Name** property. It will be similar to `asazure://westus.asazure.windows.net/contoso`. Click **OK**.

    ![](./images/analysis-services-properties.png)

4. In **Solution Explorer**, right-click the project and select **Deploy**. Sign into Azure if prompted. When processing is complete, click **Close**.

5. In the Azure portal, view the details for your Azure Analysis Services instance. Verify that your model appears in the list of models.

    ![](./images/analysis-services-models.png)

## Analyze the data in Power BI Desktop

In this step, you will use Power BI to create a report from the data in Analysis Services.

1. From your Remote Desktop session, launch Power BI Desktop.

2. In the Welcome Scren, click **Get Data**.

3. Select **Azure** > **Azure Analysis Services database**. Click **Connect**

    ![](./images/power-bi-get-data.png)

4. Enter the URL of your Analysis Services instance, then click **OK**. Sign into Azure if prompted.

5. In the **Navigator** dialog, expand the tabular project, select the model, and click **OK**.

2. In the **Visualizations** pane, select the **Table** icon. In the Report view, resize the visualization to make it larger.

6. In the **Fields** pane, expand **Dimension City**.

7. From **Dimension City**, drag **City** and **State Province** to the **Values** well.

9. In the **Fields** pane, expand **Fact Sale**.

10. From **Fact Sale**, drag **CAGR**, **Ending Population**,  and **Total Sales** to the **Value** well.

11. Under **Visual Level Filters**, select **Ending Population**. Set the filter to "is greater than 100000" and click **Apply filter**.

12. Under **Visual Level Filters**, select **Total Sales**. Set the filter to "is 0" and click **Apply filter**.

![](./images/power-bi-report-2.png)

The table now shows cities with population greater than 100,000 and zero sales. CAGR  stands for Compounded Annual Growth Rate and measures the rate of population growth per city. You could use this value to find cities with high growth rates, for example. However, note that the values for CAGR in the model aren't accurate, because they are derived from sample data.

To learn more about Power BI Desktop, see [Getting started with Power BI Desktop](/power-bi/desktop-getting-started).


[adf]: //azure/data-factory
[azure-cli-2]: //azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[MergeLocation]: https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/city/%5BIntegration%5D.%5BMergeLocation%5D.sql
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw_advanced
[wwi]: //sql/sample/world-wide-importers/wide-world-importers-oltp-database
