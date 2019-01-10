---
title: Business intelligence (BI) corporativo automatizado
titleSuffix: Azure Reference Architectures
description: Automatize um fluxo de trabalho de extração, carga e transformação (ELT) no Azure usando o Azure Data Factory com o SQL Data Warehouse.
author: MikeWasson
ms.date: 11/06/2018
ms.custom: seodec18
ms.openlocfilehash: 579ef0361ec44d0eb82b9076490eed5a6d88df35
ms.sourcegitcommit: cd3de23543f739a95a1daf38886561f67add9d64
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/10/2019
ms.locfileid: "54183585"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a>Enterprise BI automatizada com o SQL Data Warehouse e Azure Data Factory

Essa arquitetura de referência mostra como executar um carregamento incremental em um pipeline [ELT (extrair-carregar-transformar)](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt). Ela usa o Azure Data Factory para automatizar o pipeline ELT. O pipeline move incrementalmente os dados de OLTP mais recentes de um banco de dados do SQL Server local no SQL Data Warehouse. Os dados transacionais são transformados em um modelo de tabela para análise.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2Gnz2]

Há uma implantação de referência para essa arquitetura de referência disponível no [GitHub][github].

![Diagrama de arquitetura para BI corporativo automatizado com o SQL Data Warehouse e Azure Data Factory](./images/enterprise-bi-sqldw-adf.png)

Essa arquitetura é compilada naquela mostrada em [Enterprise BI com o SQL Data Warehouse](./enterprise-bi-sqldw.md), mas também adiciona alguns recursos que são importantes para cenários de data warehouse de dados da empresa.

- Automação do pipeline usando o Data Factory.
- Carregamento incremental.
- Integrando várias fontes de dados.
- Carregando dados binários como imagens e dados geoespaciais.

## <a name="architecture"></a>Arquitetura

A arquitetura consiste nos componentes a seguir.

### <a name="data-sources"></a>Fontes de dados

**SQL Server local**. A fonte de dados está localizada em um banco de dados SQL Server local. Para simular o ambiente local, os scripts de implantação dessa arquitetura provisionam uma máquina virtual no Azure com o SQL Server instalado. O [banco de dados de exemplo de OLTP de World Wide Importers][wwi] é usado como o banco de dados de origem.

**Dados externos**. Um cenário comum para data warehouses é integrar várias fontes de dados. Essa arquitetura de referência carrega um conjunto de dados externos que contém as populações de cidade por ano e integra-se aos dados do banco de dados OLTP. Você pode usar esses dados para insights, como: "O crescimento das vendas em cada região corresponde ou excede o crescimento demográfico?"

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

![Captura de tela do pipeline no Azure Data Factory](./images/adf-pipeline.png)

## <a name="incremental-loading"></a>Carregamento incremental

Quando você executa um processo automatizado de ETL ou ELT, é mais eficiente carregar apenas os dados alterados desde a execução anterior. Isso é chamado de um *carregamento incremental* em vez de um carregamento completo que carrega todos os dados. Para executar um carregamento incremental, você precisa de uma maneira de identificar quais dados foram alterados. A abordagem mais comum é usar um valor *alto de marca d'água*, o que significa acompanhar o valor mais recente de alguma coluna na tabela de origem, uma coluna de data e hora ou uma coluna de inteiro exclusivo.

Começando com o SQL Server 2016, você pode usar [tabelas temporais](/sql/relational-databases/tables/temporal-tables). Tratam-se de tabelas com versão do sistema que mantêm um histórico completo das alterações de dados. O mecanismo de banco de dados registra automaticamente o histórico de todas as alterações em uma tabela de histórico separada. Você pode consultar os dados históricos com a adição de uma cláusula FOR SYSTEM_TIME para uma consulta. Internamente, o mecanismo de banco de dados consulta a tabela do histórico, mas isso é transparente ao aplicativo.

> [!NOTE]
> Para versões anteriores do SQL Server, você pode usar a CDC ([Captura de Dados de Alterações](/sql/relational-databases/track-changes/about-change-data-capture-sql-server)). Essa abordagem é menos conveniente do que as tabelas temporais porque você tem de consultar uma tabela separada, e as alterações são controladas por um número de sequência de log em vez de um carimbo de data/hora.
>

Tabelas temporais são úteis para dados de dimensão, os quais podem ser alterados ao longo do tempo. Tabelas de fatos geralmente representam uma transação imutável, como uma venda, caso em que não faz sentido manter o histórico de versão do sistema. Em vez disso, as transações normalmente contam com uma coluna que representa a data da transação, que pode ser usada como o valor de marca d'água. Por exemplo, no banco de dados OLTP da Wide World Importers, as tabelas Sales.Invoices e Sales.InvoiceLines têm um campo `LastEditedWhen` que é padrão para `sysdatetime()`.

Aqui está o fluxo geral para o pipeline ELT:

1. Para cada tabela no banco de dados de origem, acompanhe o tempo de corte quando o último trabalho ELT foi executado. Armazene essas informações no data warehouse. (Durante a instalação inicial, todas as horas são definidas como '1-1-1900'.)

2. Durante a etapa de exportação de dados, a hora de corte é passada como um parâmetro para um conjunto de procedimentos armazenados no banco de dados de origem. Esses procedimentos armazenados consultam todos os registros que foram alterados ou criados após a hora de corte. Para a tabela de fatos de Vendas, é usada a coluna `LastEditedWhen`. Para os dados de dimensão, são usadas tabelas temporais com versão do sistema.

3. Quando a migração de dados for concluída, atualize a tabela que armazena os tempos de corte.

Também é útil registrar uma *linhagem* para cada execução de ELT. Para um determinado registro, a linhagem associa esse registro com a execução de ELT que produziu os dados. Para cada execução de ETL, um novo registro de linhagem é criado para cada tabela, mostrando os tempos de carregamento iniciais e finais. As chaves de linhagem para cada registro são armazenadas nas tabelas de dimensões e de fatos.

![Captura de tela da tabela de dimensões da cidade](./images/city-dimension-table.png)

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

O desafio é que cada cidade será dividida em um número diferente de linhas, dependendo do tamanho dos dados de geografia. Para que o operador PIVOT funcione, cada cidade deve ter o mesmo número de linhas. Para isso funcionar, a consulta T-SQL (que você pode exibir [aqui][MergeLocation]) realiza alguns truques para preencher as linhas com valores em branco, de modo que cada cidade tenha o mesmo número de colunas após a dinamização. A consulta resultante acaba sendo muito mais rápida do que executar o loop pelas linhas uma por vez.

A mesma abordagem é usada para dados de imagem.

## <a name="slowly-changing-dimensions"></a>Dimensões de alteração lenta

Dados de dimensão são relativamente estáticos, mas isso pode mudar. Por exemplo, um produto pode ser reatribuído a uma categoria de produto diferente. Há várias abordagens para o tratamento de dimensões de alteração lenta. Uma técnica comum, chamada [Tipo 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), é adicionar um novo registro sempre que uma dimensão for alterada.

Para implementar a abordagem Tipo 2, as tabelas de dimensões precisam de colunas adicionais que especifiquem o intervalo de datas efetivas para um determinado registro. Além disso, as chaves primárias do banco de dados de origem serão duplicadas, portanto, a tabela de dimensão deve ter uma chave primária artificial.

A imagem a seguir mostra a tabela Dimension.City. A coluna `WWI City ID` é a chave primária do banco de dados de origem. A coluna `City Key` é uma chave artificial gerada durante o pipeline ETL. Além disso, observe que a tabela tem as colunas `Valid From` e `Valid To`, que definem o intervalo de quando cada linha era válida. Os valores atuais têm um `Valid To` igual a “9999-12-31”.

![Captura de tela da tabela de dimensões da cidade](./images/city-dimension-table.png)

A vantagem dessa abordagem é que ela preserva os dados históricos, que podem ser valiosos para análise. No entanto, também significa que haverá várias linhas para a mesma entidade. Por exemplo, aqui estão os registros que correspondem a `WWI City ID` = 28561:

![Segunda captura de tela da tabela de dimensões da cidade](./images/city-dimension-table-2.png)

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

Para a implantação e execução da implementação de referência, siga as etapas em [Leia-me do GitHub][github]. Ela implanta o seguinte:

- Uma VM Windows para simular um servidor de banco de dados local. Ela inclui o SQL Server 2017 e ferramentas relacionadas, juntamente com o Power BI Desktop.
- Uma conta de armazenamento do Azure que fornece armazenamento de blobs para armazenar os dados exportados do banco de dados SQL Server.
- Uma instância do SQL Data Warehouse do Azure.
- Uma instância do Azure Analysis Services.
- O Azure Data Factory e o pipeline do Data Factory para o trabalho ELT.

## <a name="related-resources"></a>Recursos relacionados

O ideal é examinar os seguintes [cenários de exemplo do Azure](/azure/architecture/example-scenario), que demonstram soluções específicas usando algumas das mesmas tecnologias:

- [Data warehouse e análise para vendas e marketing](/azure/architecture/example-scenario/data/data-warehouse)
- [ETL Híbrido com Azure Data Factory e SSIS local existentes](/azure/architecture/example-scenario/data/hybrid-etl-with-adf)

<!-- links -->

[adf]: /azure/data-factory
[github]: https://github.com/mspnp/azure-data-factory-sqldw-elt-pipeline
[MergeLocation]: https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/city/%5BIntegration%5D.%5BMergeLocation%5D.sql
[wwi]: /sql/sample/world-wide-importers/wide-world-importers-oltp-database
