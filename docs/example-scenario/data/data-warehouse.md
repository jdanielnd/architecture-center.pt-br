---
title: Data warehouse e análise de vendas e marketing
titleSuffix: Azure Example Scenarios
description: Consolide dados de várias fontes e otimize a análise de dados.
author: alexbuckgit
ms.date: 09/15/2018
ms.openlocfilehash: 5727b6ab475224541e272c6da6243cabe851b919
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/20/2018
ms.locfileid: "53643978"
---
# <a name="data-warehousing-and-analytics-for-sales-and-marketing"></a>Data warehouse e análise de vendas e marketing

Este cenário de exemplo demonstra um pipeline de dados que integra a grandes quantidades de dados de várias fontes em uma plataforma de análise unificada no Azure. Este cenário específico se baseia em uma solução de vendas e marketing, mas os padrões de design são relevantes para muitos setores que exigem análise avançada de grandes conjuntos de dados, como serviços de saúde, varejo e comércio eletrônico.

Este exemplo demonstra uma empresa de vendas e marketing que cria programas de incentivo. Esses programas recompensam os clientes, fornecedores, vendedores e funcionários. Os dados são fundamentais para esses programas e a empresa deseja melhorar as informações obtidas por meio da análise de dados usando o Azure.

A empresa precisa de uma abordagem moderna dos dados de análise para que as decisões sejam tomadas usando os dados certos no momento certo. As metas da empresa incluem:

- Combinar tipos diferentes de fontes de dados em uma plataforma em escala de nuvem.
- Transformar os dados de origem em uma estrutura e taxonomia comum, para deixar os dados consistentes e facilitar a comparação.
- Carregar os dados usando uma abordagem altamente paralelizada que pode dar suporte a milhares de programas de incentivo, sem os altos custos de implantação e manutenção de infraestrutura local.
- Reduzir significativamente o tempo necessário para reunir e transformar dados, para que você possa se concentrar na análise de dados.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Essa abordagem também pode ser usada para:

- Estabeleça um data warehouse para ser a única fonte de verdade para seus dados.
- Integre as fontes de dados relacionais com outros conjuntos de dados não estruturados.
- Use a modelagem semântica e as ferramentas de visualização poderosas para uma análise de dados mais simples.

## <a name="architecture"></a>Arquitetura

![Arquitetura para um cenário de análise e data warehousing no Azure][architecture]

Os dados fluem pela solução da seguinte maneira:

1. Para cada fonte de dados, todas as atualizações são exportadas periodicamente em uma área de preparo no Armazenamento de Blobs do Azure.
2. O Data Factory carrega incrementalmente os dados do Armazenamento de Blobs em tabelas de preparo no SQL Data Warehouse. Os dados são limpos e transformados durante esse processo. O Polybase pode paralelizar o processo para grandes conjuntos de dados.
3. Depois de carregar um novo lote de dados no warehouse, um modelo de tabela do Analysis Services criado anteriormente é atualizado. Este modelo semântico simplifica a análise de dados de negócios e relações.
4. Os analistas de negócios usam o Microsoft Power BI para analisar os dados escalonados por meio do modelo semântico do Analysis Services.

### <a name="components"></a>Componentes

A empresa tem fontes de dados em várias plataformas diferentes:

- SQL Server local
- Oracle local
- Banco de Dados SQL do Azure
- Armazenamento de tabelas do Azure
- Cosmos DB

Os dados são carregados destas fontes de dados diferentes usando diversos componentes do Azure:

- O [Armazenamento de Blobs](/azure/storage/blobs/storage-blobs-introduction) é usado para transferir dados de origem antes de serem carregados no SQL Data Warehouse.
- O [Data Factory](/azure/data-factory) coordena a transformação de dados preparados em uma estrutura comum no SQL Data Warehouse. O Data Factory [usa o Polybase ao carregar dados no SQL Data Warehouse](/azure/data-factory/connector-azure-sql-data-warehouse#use-polybase-to-load-data-into-azure-sql-data-warehouse) para maximizar a taxa de transferência.
- O [SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is) é um sistema distribuído para armazenar e analisar grandes conjuntos de dados. O uso que ele faz do MPP (processamento altamente paralelo) o torna adequado para a execução de análises de alto desempenho. O SQL Data Warehouse pode usar [PolyBase](/sql/relational-databases/polybase/polybase-guide) para carregar rapidamente os dados do Armazenamento de Blobs.
- O [Analysis Services](/azure/analysis-services) fornece um modelo semântico para seus dados. Ele também pode aumentar o desempenho do sistema ao analisar seus dados.
- [Power BI](/power-bi) é um conjunto de ferramentas de análise de negócios para analisar dados e compartilhar informações. O Power BI pode consultar um modelo semântico armazenado no Analysis Services ou pode consultar diretamente o SQL Data Warehouse.
- O [Azure AD (Azure Active Directory)](/azure/active-directory) autentica os usuários que se conectam ao servidor do Analysis Services por meio do Power BI. O Data Factory também pode usar o Azure AD para autenticar no SQL Data Warehouse por meio de uma entidade de serviço ou uma [Identidade gerenciada para recursos do Azure](/azure/active-directory/managed-identities-azure-resources/overview).

### <a name="alternatives"></a>Alternativas

- O pipeline de exemplo inclui vários tipos diferentes de fontes de dados. Essa arquitetura pode manusear uma grande variedade de fontes de dados relacionais e não relacionais.
- O Data Factory coordena os fluxos de trabalho para o pipeline de dados. Se você deseja carregar os dados apenas uma vez ou sob demanda, poderá usar ferramentas como o SQL Server de cópia em massa (bcp) e o AzCopy para copiar dados para o Armazenamento de Blobs. Em seguida, é possível carregar os dados diretamente no SQL Data Warehouse usando o Polybase.
- Se você tiver grandes conjuntos de dados, considere o uso de [Data Lake Storage](/azure/storage/data-lake-storage/introduction), que fornece armazenamento ilimitado para dados de análise.
- Um dispositivo [SQL Server Parallel Data Warehouse](/sql/analytics-platform-system) local também pode ser usado para processamento de big data. No entanto, os custos operacionais geralmente são muito menores com uma solução baseada em nuvem gerenciada, como o SQL Data Warehouse.
- O SQL Data Warehouse não é uma boa opção para cargas de trabalho OLTP ou conjuntos de dados menores do que 250 GB. Para esses casos, você deve usar o banco de dados SQL do Azure ou o SQL Server.
- Para comparações de outras alternativas, consulte:

  - [Como escolher uma tecnologia de orquestração do pipeline de dados no Azure](/azure/architecture/data-guide/technology-choices/pipeline-orchestration-data-movement)
  - [Como escolher uma tecnologia de processamento em lotes no Azure](/azure/architecture/data-guide/technology-choices/batch-processing)
  - [Como escolher um armazenamento de dados analíticos no Azure](/azure/architecture/data-guide/technology-choices/analytical-data-stores)
  - [Como escolher uma tecnologia de análise de dados no Azure](/azure/architecture/data-guide/technology-choices/analysis-visualizations-reporting)

## <a name="considerations"></a>Considerações

As tecnologias nesta arquitetura foram escolhidas porque atendem aos requisitos de escalabilidade e disponibilidade da empresa, e ao mesmo tempo, ajudam a controlar os custos.

- A [arquitetura de processamento altamente paralelo](/azure/sql-data-warehouse/massively-parallel-processing-mpp-architecture) do SQL Data Warehouse oferece alto desempenho e escalabilidade.
- O SQL Data Warehouse tem [SLAs de garantia](https://azure.microsoft.com/support/legal/sla/sql-data-warehouse) e [práticas recomendadas para alcançar alta disponibilidade](/azure/sql-data-warehouse/sql-data-warehouse-best-practices).
- Quando a atividade de análise estiver baixa, a empresa pode [dimensionar o SQL Data Warehouse sob demanda](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview), reduzindo ou até mesmo interrompendo a computação para reduzir os custos.
- O Azure Analysis Services pode ser [escalado horizontalmente](/azure/analysis-services/analysis-services-scale-out) para reduzir os tempos de resposta durante altas cargas de trabalho de consulta. Também é possível separar o processamento do pool de consultas, para que as consultas de clientes não fiquem mais lentas devido às operações de processamento.
- O Azure Analysis Services tem [SLAs de garantia](https://azure.microsoft.com/support/legal/sla/analysis-services) e [práticas recomendadas para alcançar alta disponibilidade](/azure/analysis-services/analysis-services-bcdr).
- O [modelo de segurança do SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security) fornece segurança, [autenticação e autorização de conexão](/azure/sql-data-warehouse/sql-data-warehouse-authentication) por meio do Azure AD ou da autenticação do SQL Server e da criptografia. O [Azure Analysis Services](/azure/analysis-services/analysis-services-manage-users) usa o Azure AD para o gerenciamento de identidade e a autenticação de usuário.

## <a name="pricing"></a>Preços

Analise um [exemplo de preço para um cenário de armazenamento de dados][calculator] na Calculadora de Preços do Azure. Ajuste os valores para ver como seus requisitos afetam os custos.

- O [SQL Data Warehouse](https://azure.microsoft.com/pricing/details/sql-data-warehouse/gen2) permite que você dimensione seus níveis de computação e armazenamento independentemente. Os recursos de computação são cobrados por hora e você pode dimensioná-los ou interrompê-los sob demanda. Os recursos de armazenamento são cobrados por terabyte, assim seus custos aumentam à medida que você insere mais dados.
- [Data Factory](https://azure.microsoft.com/pricing/details/data-factory) os custos são baseados no número de operações de leitura/gravação, monitoramento e atividades de orquestração realizadas em uma carga de trabalho. Os custos de Data Factory aumentam com cada fluxo de dados adicional e a quantidade de dados processados por cada um.
- O [Analysis Services](https://azure.microsoft.com/pricing/details/analysis-services) está disponível nas camadas desenvolvedor, básico e standard. As instâncias são cobradas com base em QPUs (unidades de processamento de consulta) e na memória disponível. Para manter os custos reduzidos, minimize o número de consultas executadas, a quantidade de dados processada e a frequência de execução.
- O [Power BI](https://powerbi.microsoft.com/pricing) tem diferentes opções de produto para diversos requisitos. O [Power BI Embedded](https://azure.microsoft.com/pricing/details/power-bi-embedded) fornece uma opção baseada no Azure para incorporar a funcionalidade do Power BI em seus aplicativos. Uma instância do Power BI Embedded está incluída no exemplo de preço acima.

## <a name="next-steps"></a>Próximas etapas

- Analise a [Arquitetura de referência do Azure para Enterprise BI automatizada](/azure/architecture/reference-architectures/data/enterprise-bi-adf), que inclui instruções para implantar uma instância dessa arquitetura no Azure.
- Leia a [história de cliente de Soluções de Motivação da Maritz][source-document]. Essa história descreve uma abordagem semelhante para gerenciamento de dados do cliente.
- Encontre orientações abrangentes de arquitetura em pipelines de dados, data warehouse, processamento analítico online (OLAP) e Big Data no [Guia de arquitetura de dados do Azure](/azure/architecture/data-guide).

<!-- links -->
[source-document]: https://customers.microsoft.com/story/maritz
[calculator]: https://azure.com/e/b798fb70c53e4dd19fdeacea4db78276
[architecture]: ./media/architecture-data-warehouse.png
