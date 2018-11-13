---
title: Escolhendo uma tecnologia de processamento em lotes
description: ''
author: zoinerTejada
ms:date: 11/03/2018
ms.openlocfilehash: 2314a1413fa674f43bd7a4bcb868a5322ad99497
ms.sourcegitcommit: 225251ee2dd669432a9c9abe3aa8cd84d9e020b7
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/04/2018
ms.locfileid: "50981990"
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>Escolhendo uma tecnologia de processamento em lotes no Azure

Com frequência, as soluções de Big Data usam trabalhos em lotes de execução longa para filtrar, agregar e, de outro modo, preparar os dados para análise. Normalmente, esses trabalhos envolvem a leitura de arquivos de origem de um armazenamento escalonável (como o HDFS, Azure Data Lake Store e Armazenamento do Azure), seu processamento e gravação da saída em novos arquivos no armazenamento escalonável. 

O principal requisito desses mecanismos de processamento em lotes é a capacidade de expandir cálculos, para manipular um grande volume de dados. No entanto, ao contrário do processamento em tempo real, o processamento em lotes deve ter latências (o tempo entre a ingestão de dados e o cálculo de um resultado) que são medidas em minutos a horas.

## <a name="technology-choices-for-batch-processing"></a>Opções de tecnologia para o processamento em lotes

### <a name="azure-sql-data-warehouse"></a>SQL Data Warehouse do Azure

O [SQL Data Warehouse](/azure/sql-data-warehouse/) é um sistema distribuído projetado para executar uma análise em grandes quantidades de dados. Ele dá suporte a MPP (processamento altamente paralelo), que o torna adequado para executar análise de alto desempenho. Quando você tiver grandes quantidades de dados (mais de 1 TB) e estiver executando uma carga de trabalho de análise que será beneficiada pelo paralelismo, considere o SQL Data Warehouse.

### <a name="azure-data-lake-analytics"></a>Análise Azure Data Lake

O [Data Lake Analytics](/azure/data-lake-analytics/data-lake-analytics-overview) é um serviço do trabalho de análise sob demanda. Ele é otimizado para o processamento distribuído de grandes conjuntos de dados armazenados no Azure Data Lake Store. 

- Linguagens: [U-SQL](/azure/data-lake-analytics/data-lake-analytics-u-sql-get-started) (inclusive extensões Python, R e C#).
-  Integra-se com: Azure Data Lake Store, blobs do Armazenamento do Azure, Banco de Dados SQL do Azure e SQL Data Warehouse.
- O modelo de preços é por trabalho.

### <a name="hdinsight"></a>HDInsight

O HDInsight é um serviço gerenciado do Hadoop. Use-o para implantar e gerenciar clusters Hadoop no Azure. Para o processamento em lotes, você pode usar [Spark](/azure/hdinsight/spark/apache-spark-overview), [Hive](/azure/hdinsight/hadoop/hdinsight-use-hive), [Hive LLAP](/azure/hdinsight/interactive-query/apache-interactive-query-get-started), [MapReduce](/azure/hdinsight/hadoop/hdinsight-use-mapreduce).

- Linguagens: R, Python, Java, Scala, SQL
- Autenticação Kerberos com o Active Directory, controle de acesso baseado no Apache Ranger
- Fornece controle total do cluster do Hadoop

### <a name="azure-databricks"></a>Azure Databricks 

O [Azure Databricks](/azure/azure-databricks/) é uma plataforma de análise baseada no Apache Spark. Você pode imaginá-la como o “Spark como um serviço”. É a maneira mais fácil de usar o Spark na plataforma do Azure.  

- Linguagens: R, Python, Java, Scala, Spark SQL
- Horas de início, término automático e dimensionamento automático rápidos do cluster.
- Gerencia o cluster Spark para você.
- Integração interna com o Armazenamento de Blobs do Azure, Azure Data Lake Storage (ADLS), Data Warehouse do Azure SQL DW (SQL) e outros serviços. Confira [Fontes de Dados](https://docs.azuredatabricks.net/spark/latest/data-sources/index.html).
- Autenticação do usuário com o Azure Active Directory.
- [Blocos de notas](https://docs.azuredatabricks.net/user-guide/notebooks/index.html) baseados na Web para colaboração e exploração de dados. 
- Tem suporte para [clusters habilitados para GPU](https://docs.azuredatabricks.net/user-guide/clusters/gpu.html)

### <a name="azure-distributed-data-engineering-toolkit"></a>Kit de ferramentas de engenharia de dados distribuídos do Azure 

O [Kit de ferramentas de engenharia de dados distribuídos](https://github.com/azure/aztk) (AZTK) é uma ferramenta para o provisionamento sob demanda do Spark em clusters do Docker no Azure. 

O AZTK não é um serviço do Azure. Trata-se de uma ferramenta do lado do cliente com uma interface de SDK da CLI e do Python, criada no Lote do Azure. Essa opção oferece mais controle sobre a infraestrutura ao implantar um cluster do Spark.

- Traga sua própria imagem do Docker.
- Use VMs de baixa prioridade para um desconto de 80%.
- Clusters de modo misto que usam VMs de baixa prioridade e dedicadas.
- Suporte interno para conexão de Armazenamento de Blobs do Azure e o Azure Data Lake.

## <a name="key-selection-criteria"></a>Principais critérios de seleção

Para restringir as opções, comece respondendo a estas perguntas:

- Você deseja ter um serviço gerenciado em vez de gerenciar seus próprios servidores?

- Você deseja criar a lógica do processamento em lotes de forma declarativa ou imperativa?

- Você executará o processamento em lotes em intermitências? Em caso afirmativo, considere opções que permitam que você encerre o cluster automaticamente ou cujo modelo de preço seja por trabalho em lotes.

- Você precisa consultar armazenamentos de dados relacionais junto com o processamento em lotes, por exemplo, para pesquisar dados de referência? Nesse caso, considere as opções que permitem consultar relational stores externos.

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades. 

### <a name="general-capabilities"></a>Funcionalidades gerais

| | Análise Azure Data Lake | SQL Data Warehouse do Azure | HDInsight | Azure Databricks |
| --- | --- | --- | --- | --- | --- |
| É um serviço gerenciado | SIM | SIM | Sim <sup>1</sup> | SIM | 
| Armazenamento de dados relacionais | SIM | sim | Não | Não  |
| Modelo de preços | Por trabalho em lotes | Por hora de cluster | Por hora de cluster | Unidade do Databricks<sup>2</sup> + hora de cluster |

[1] Com configuração manual e dimensionamento.

[2] Uma DBU (unidade do Databricks) é uma unidade de capacidade de processamento por hora.

### <a name="capabilities"></a>Funcionalidades

| | Análise Azure Data Lake | SQL Data Warehouse | HDInsight com Spark | HDInsight com Hive | HDInsight com Hive LLAP | Azure Databricks |
| --- | --- | --- | --- | --- | --- | --- |
| Dimensionamento automático | Não  | Não | Não | Não | Não  | SIM |
| Granularidade de expansão  | Por trabalho | Por cluster | Por cluster | Por cluster | Por cluster | Por cluster |
| Cache em memória de dados | Não  | sim | sim | Não  | sim | SIM |
| Consulta por meio de relational stores externos | SIM | Não  | Sim | Não | Não  | SIM |
| Autenticação  | AD do Azure | SQL/Azure AD | Não  | Azure AD<sup>1</sup> | Azure AD<sup>1</sup> | AD do Azure |
| Auditoria  | SIM | sim | Não  | Sim <sup>1</sup> | Sim <sup>1</sup> | SIM |
| Segurança em nível de linha | Não  | Não | Não  | Sim <sup>1</sup> | Sim <sup>1</sup> | Não  |
| Dá suporte a firewalls | SIM | sim | SIM | Sim <sup>2</sup> | Sim <sup>2</sup> | Não  |
| Mascaramento de dados dinâmicos | Não  | Não | Não  | Sim <sup>1</sup> | Sim <sup>1</sup> | Não  |

[1] Exige o uso de um [cluster HDInsight ingressado no domínio](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Compatível quando [usado em uma Rede Virtual do Azure](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
