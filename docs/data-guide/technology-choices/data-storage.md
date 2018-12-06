---
title: Escolhendo uma tecnologia de armazenamento de dados
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: c97249228ca45a7a17822b6dd55acad6360c6f6b
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902639"
---
# <a name="choosing-a-big-data-storage-technology-in-azure"></a>Escolhendo uma tecnologia de armazenamento de Big Data no Azure

Este tópico compara as opções de armazenamento de dados para soluções de Big Data &mdash; especificamente, o armazenamento de dados para ingestão de dados em massa e processamento em lotes, em detrimento dos [armazenamentos de dados analíticos](./analytical-data-stores.md) ou da [ingestão de streaming em tempo real](./real-time-ingestion.md).

## <a name="what-are-your-options-when-choosing-data-storage-in-azure"></a>Quais são as opções disponíveis ao escolher um armazenamento de dados no Azure?

Há várias opções para a ingestão de dados no Azure, dependendo de suas necessidades:

**Armazenamento de arquivos**

- [Blobs do Armazenamento do Azure](/azure/storage/blobs/storage-blobs-introduction)
- [Repositório Azure Data Lake](/azure/data-lake-store/)

**Bancos de dados NoSQL**

- [Azure Cosmos DB](/azure/cosmos-db/)
- [HBase no HDInsight](https://hbase.apache.org/)

## <a name="azure-storage-blobs"></a>Blobs do Armazenamento do Azure

O Armazenamento do Azure é um serviço de armazenamento gerenciado altamente disponível, seguro, durável, escalonável e redundante. A Microsoft cuida da manutenção e lida com problemas importantes para você. O Armazenamento do Azure é a solução de armazenamento com maior presença oferecida pelo Azure, devido à quantidade de serviços e ferramentas que podem ser usados com ele.

Há vários serviços do Armazenamento do Azure que podem ser usados para armazenar dados. A opção mais flexível para armazenar blobs de uma variedade de fontes de dados é o [armazenamento de Blobs](/azure/storage/blobs/storage-blobs-introduction). Blobs são basicamente arquivos. Eles armazenam imagens, documentos, arquivos HTML, VHDs (discos rígidos virtuais), Big Data como logs, backups de banco de dados &mdash; praticamente qualquer coisa. Blobs são armazenados em contêineres, que são semelhantes às pastas. Um contêiner fornece um agrupamento de um conjunto de blobs. Uma conta de armazenamento pode conter um número ilimitado de contêineres e um contêiner pode armazenar um número ilimitado de blobs.

O Armazenamento do Azure é uma boa escolha para soluções de Big Data e análise, devido à sua flexibilidade, alta disponibilidade e baixo custo. Ele fornece camadas de armazenamento frio, quente e de arquivos para diferentes casos de uso. Para obter mais informações, consulte [Armazenamento de Blobs do Azure: camadas de armazenamento quente, frio e de arquivos](/azure/storage/blobs/storage-blob-storage-tiers).

O armazenamento de Blobs do Azure pode ser acessado por meio do Hadoop (disponível no HDInsight). O HDInsight pode usar um contêiner de blobs no Armazenamento do Azure como o sistema de arquivos padrão para o cluster. Por meio de uma interface HDFS (Sistema de Arquivos Distribuído Hadoop) fornecida por um driver WASB, o conjunto completo de componentes no HDInsight pode operar diretamente sobre os dados estruturados ou não estruturados armazenados como blobs. O armazenamento de Blobs do Azure também pode ser acessado por meio do SQL Data Warehouse do Azure usando seu recurso PolyBase.

Outros recursos que tornam o Armazenamento do Azure uma boa opção são os seguintes:

- [Várias estratégias de simultaneidade](/azure/storage/common/storage-concurrency?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [Opções de recuperação de desastre e alta disponibilidade](/azure/storage/common/storage-disaster-recovery-guidance?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [Criptografia em repouso](/azure/storage/common/storage-service-encryption?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [RBAC (Controle de Acesso Baseado em Função)](/azure/storage/common/storage-security-guide?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#management-plane-security) para controlar o acesso usando usuários e grupos do Azure Active Directory.

## <a name="azure-data-lake-store"></a>Repositório Azure Data Lake

O [Azure Data Lake Store](/azure/data-lake-store/) é um repositório em hiper-escala corporativo para cargas de trabalho de análise de big data. O Data Lake permite que você capture dados de qualquer tamanho, tipo e velocidade de ingestão em um único lugar [seguro](/azure/data-lake-store/data-lake-store-overview#DataLakeStoreSecurity) para análises operacionais e exploratórias.

O Data Lake Store não impõe nenhum limite de tamanho de conta, tamanho de arquivo ou a quantidade de dados que podem ser armazenados em um data lake. Os dados são armazenados permanentemente por meio de várias cópias e não há limite de tempo de armazenamento dos dados no Data Lake. Além de fazer várias cópias de arquivos para proteger contra falhas inesperadas, o Data Lake distribui partes de um arquivo em vários servidores de armazenamento individuais. Isso melhora a produtividade da leitura do arquivo em paralelo para a realização de análises de dados.

O Data Lake Store pode ser acessado por meio do Hadoop (disponível no HDInsight) usando as APIs REST compatíveis com WebHDFS. Considere a possibilidade de usar isso como uma alternativa ao Armazenamento do Azure, quando os tamanhos de arquivo individuais ou combinados excederem o limite com suporte no Armazenamento do Azure. No entanto, há [diretrizes de ajuste de desempenho](/azure/data-lake-store/data-lake-store-performance-tuning-guidance#optimizing-io-intensive-jobs-on-hadoop-and-spark-workloads-on-hdinsight) que devem ser seguidas ao usar o Data Lake Store como o armazenamento primário para um cluster HDInsight, com diretrizes específicas para o [Spark](/azure/data-lake-store/data-lake-store-performance-tuning-spark), [Hive](/azure/data-lake-store/data-lake-store-performance-tuning-hive), [MapReduce](/azure/data-lake-store/data-lake-store-performance-tuning-mapreduce) e [Storm](/azure/data-lake-store/data-lake-store-performance-tuning-storm). Além disso, verifique a [disponibilidade regional](https://azure.microsoft.com/regions/#services) do Data Lake Store, porque ele não está disponível no mesmo número de regiões que o Armazenamento do Azure e precisa estar localizado na mesma região do cluster HDInsight.

Acoplado com o Azure Data Lake Analytics, o Data Lake Store foi especificamente projetado para permitir a análise de dados armazenados e é ajustado quanto ao desempenho para cenários de análise de dados. O Data Lake Store também pode ser acessado por meio do SQL Data Warehouse do Azure usando seu recurso PolyBase.

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

O [Azure Cosmos DB](/azure/cosmos-db/) é o banco de dados multimodelo globalmente distribuído da Microsoft. O Cosmos DB garante latências de milissegundo de um dígito no 99º percentil em qualquer parte do mundo, oferece vários modelos de consistência bem definidos para ajustar o desempenho e garante alta disponibilidade com funcionalidades de hospedagem múltipla.

O Azure Cosmos DB é independente de esquema. Ele indexa todos os dados automaticamente, sem a necessidade de lidar com o gerenciamento de esquema e de índice. Também é multimodelo, dando suporte nativo a modelos de dados de documentos, de chave-valor, de grafos e de família de colunas. 

Recursos do Azure Cosmos DB:

- [Replicação geográfica](/azure/cosmos-db/distribute-data-globally)
- [Dimensionamento elástico de taxa de transferência e armazenamento](/azure/cosmos-db/partition-data) em todo o mundo
- [Cinco níveis de consistência bem definidos](/azure/cosmos-db/consistency-levels)

## <a name="hbase-on-hdinsight"></a>HBase no HDInsight

O [Apache HBase](https://hbase.apache.org/) é um banco de dados NoSQL de software livre, que se baseia no Hadoop e é modelado com base no Google BigTable. O HBase fornece acesso aleatório e coerência forte para grandes quantidades de dados não estruturados e semiestruturados em um banco de dados sem esquema, organizado por famílias de colunas.

Os dados são armazenados nas linhas de uma tabela e os dados contidos nas linhas são agrupados por famílias de colunas. O HBase é sem esquema, pois as colunas nem o tipo de dados armazenados nelas precisam ser definidos antes de serem utilizados. O código-fonte aberto é dimensionado linearmente para lidar com petabytes de dados em milhares de nós. Ele pode fazer uso da redundância de dados, do processamento em lote e de outros recursos que são fornecidos por aplicativos distribuídos no ecossistema do Hadoop.

A [implementação do HDInsight](/azure/hdinsight/hbase/apache-hbase-overview) utiliza a arquitetura de expansão do HBase para fornecer a fragmentação automática de tabelas, coerência forte para leituras e gravações e failover automático. O desempenho é aprimorado pelo cache na memória para leituras e streaming de alta produtividade para gravações. Na maioria dos casos, é recomendável [criar o cluster HBase dentro de uma rede virtual](/azure/hdinsight/hbase/apache-hbase-provision-vnet), de forma que outros clusters HDInsight e aplicativos possam acessar diretamente as tabelas.

## <a name="key-selection-criteria"></a>Principais critérios de seleção

Para restringir as opções, comece respondendo a estas perguntas:

- Você precisa de um armazenamento baseado em nuvem gerenciado e de alta velocidade para qualquer tipo de dados de texto ou binários? Em caso afirmativo, selecione uma das opções de armazenamento de arquivos.

- Você precisa de um armazenamento de arquivos otimizado para cargas de trabalho de análise paralela e alto IOPS/taxa de transferência? Em caso afirmativo, escolha uma opção ajustada para o desempenho da carga de trabalho de análise.

- Você precisa armazenar dados não estruturados ou semiestruturados em um banco de dados sem esquema? Nesse caso, selecione uma das opções não relacionais. Compare opções de modelos de banco de dados e indexação. Dependendo do tipo de dados que você precisa armazenar, os modelos de banco de dados primário podem ser o fator proeminente.

- Você pode usar o serviço em sua região? Verifique a disponibilidade regional para cada serviço do Azure. Consulte [Produtos disponíveis por região](https://azure.microsoft.com/regions/services/).

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades.

### <a name="file-storage-capabilities"></a>Funcionalidades de armazenamento de arquivos

|  | Repositório Azure Data Lake | Contêineres do Armazenamento de Blobs do Azure |
| --- | --- | --- |
| Finalidade | Armazenamento otimizado para cargas de trabalho de análise de big data |Repositório de objetos de finalidade geral para uma ampla variedade de cenários de armazenamento |
| Casos de uso | Lote, análise de streaming e dados de aprendizado de máquina, como arquivos de log, dados de IoT, fluxos de cliques, conjuntos de dados grandes | Qualquer tipo de dados de texto ou binários, como back-end de aplicativo, dados de backup, armazenamento de mídia para streaming e dados de uso geral |
| Estrutura | Sistema de arquivos hierárquico | Repositório de objetos com namespace simples |
| Autenticação | Com base em [Identidades do Azure Active Directory](/azure/active-directory/active-directory-authentication-scenarios) | Com base em [Chaves de Acesso da Conta](/azure/storage/common/storage-create-storage-account#manage-your-storage-account) e [Chaves de Assinatura de Acesso Compartilhado](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) dos segredos compartilhados e em [RBAC (Controle de Acesso Baseado em Função)](/azure/security/security-storage-overview) |
| Protocolo de autenticação | OAuth 2.0. As chamadas precisam conter um JWT (Token Web JSON) válido emitido pelo Azure Active Directory | HMAC (código de autenticação de mensagem baseado em hash). As chamadas devem conter um hash SHA-256 codificado na Base64 em uma parte da solicitação HTTP. |
| Autorização | ACLs (listas de controle de acesso) do POSIX. ACLs baseadas em identidades do Azure Active Directory podem ser definidas com nível de arquivo e pasta. | Para autorização no nível de conta, use [Chaves de Acesso da Conta](/azure/storage/common/storage-create-storage-account#manage-your-storage-account). Para autorização de conta, contêiner ou blob, use [Chaves de Assinatura de Acesso Compartilhado](/azure/storage/common/storage-dotnet-shared-access-signature-part-1). |
| Auditoria | Disponível.  |Disponível |
| Criptografia em repouso | Transparente, do lado do servidor | Transparente, do lado do servidor; criptografia do lado do cliente |
| SDKs de desenvolvedor | .NET, Java, Python, Node.js | .Net, Java, Python, Node.js, C++, Ruby |
| Desempenho da carga de trabalho de análise | Desempenho otimizado para cargas de trabalho de análise paralela, Alta Taxa de Transferência e IOPS | Não otimizado para cargas de trabalho de análise |
| Limites de tamanho | Sem limites para tamanhos de conta, tamanhos de arquivo ou número de arquivos | Limites específicos documentados [aqui](/azure/azure-subscription-service-limits#storage-limits) |
| Redundância geográfica | Com redundância local (várias cópias dos dados em uma região do Azure) | LRS (localmente redundante), GRS (globalmente redundante), RA-GRS (acesso de leitura globalmente redundante). Veja mais informações [aqui](/azure/storage/common/storage-redundancy) |

### <a name="nosql-database-capabilities"></a>Funcionalidades de banco de dados NoSQL

|                                    |                                           Azure Cosmos DB                                           |                                                             HBase no HDInsight                                                             |
|------------------------------------|-----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
|       Modelo de banco de dados primário       |                      Repositório de documentos, gráfico, repositório de chave-valor, repositório de coluna grande                      |                                                             Repositório de coluna grande                                                              |
|         Índices secundários          |                                                 SIM                                                 |                                                                     Não                                                                      |
|        Suporte à linguagem SQL        |                                                 SIM                                                 |                                     Sim (usando o driver JDBC [Phoenix](https://phoenix.apache.org/))                                      |
|            Consistência             |                   Forte, desatualização limitada, sessão, prefixo consistente, eventual                   |                                                                   Strong                                                                   |
| Integração nativa com o Azure Functions |                        [Sim](/azure/cosmos-db/serverless-computing-database)                        |                                                                     Não                                                                      |
|   Distribuição global automática    |                          [Sim](/azure/cosmos-db/distribute-data-globally)                           | Não [A replicação de cluster HBase pode ser configurada](/azure/hdinsight/hbase/apache-hbase-replication) em regiões com consistência eventual |
|           Modelo de preços            | RUs (unidades de solicitação) elasticamente escalonáveis cobradas por segundo, conforme necessário, armazenamento elasticamente escalonável |                              Preços por minuto do cluster HDInsight (dimensionamento horizontal de nós), armazenamento                               |

