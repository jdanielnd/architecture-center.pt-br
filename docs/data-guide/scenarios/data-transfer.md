---
title: "Escolhendo uma tecnologia de transferência de dados"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: bb0732b0f771a4c9e1a4e565875576c08484490a
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/14/2018
---
# <a name="transferring-data-to-and-from-azure"></a>Transferindo dados bidirecionalmente no Azure

Há várias opções de transferência de dados bidirecionalmente no Azure, dependendo de suas necessidades.

## <a name="physical-transfer"></a>Transferência física

O uso do hardware físico para transferir dados para o Azure é uma boa opção quando:

- A rede está lenta ou não é confiável.
- A obtenção da largura de banda de rede adicional é cara.
- As políticas organizacionais ou de segurança não permitem conexões de saída ao lidar com os dados confidenciais. 

Se a principal preocupação é a duração de transferência dos dados, é recomendável executar um teste para verificar se a transferência de rede é realmente mais lenta do que o transporte físico.

Há duas opções principais para o transporte físico de dados para o Azure:
- **Importação/Exportação do Azure**. O [serviço de Importação/Exportação do Azure](/azure/storage/common/storage-import-export-service) permite que você transfira grandes quantidades de dados com segurança para o Armazenamento de Blobs do Azure pelo envio de HDDs ou SDDs SATA internos para um datacenter do Azure. Também use esse serviço para transferir dados do Armazenamento do Azure para unidades de disco rígido e enviá-los a você para o carregamento local.

- **Azure Data Box**. O [Azure Data Box](https://azure.microsoft.com/services/storage/databox/) é um dispositivo fornecido pela Microsoft que funciona de modo muito semelhante ao serviço de Importação/Exportação do Azure. A Microsoft fornece um dispositivo proprietário, seguro e de transferência resistente a adulterações e cuida da logística de ponta a ponta, que você pode controlar por meio do portal. Um benefício do serviço Azure Data Box é a facilidade de uso. Você não precisa comprar várias unidades de disco rígido, prepará-los e transferir arquivos para cada uma. O Azure Data Box é compatível com vários parceiros líderes do setor do Azure para facilitar a utilização perfeita do transporte offline para a nuvem de seus produtos. 

## <a name="command-line-tools-and-apis"></a>Ferramentas de linha de comando e APIs

Considere estas opções quando desejar fazer uma transferência de dados programática e com script.

- **CLI do Azure**. A [CLI do Azure](/azure/hdinsight/hdinsight-upload-data#commandline) é uma ferramenta de multiplaforma que permite que você gerencie os serviços do Azure e carregue dados para o Armazenamento do Azure. 

- **AzCopy**. Use o AzCopy em uma linha de comando do [Windows](/azure/storage/common/storage-use-azcopy?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) ou [Linux](/azure/storage/common/storage-use-azcopy-linux?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) para copiar dados com facilidade bidirecionalmente no armazenamento de Blobs, Arquivos e Tabelas do Azure com um desempenho ideal. O AzCopy dá suporte à simultaneidade e ao paralelismo e à capacidade de retomar as operações de cópia quando elas forem interrompidas. Ele também é mais rápido do que a maioria das outras opções. Para o acesso programático, a [Biblioteca de Movimentação de Dados do Armazenamento do Microsoft Azure](/azure/storage/common/storage-use-data-movement-library) é a estrutura básica que habilita o AzCopy. Ela é fornecida como uma biblioteca .NET Core. 

- **PowerShell**. O [cmdlet `Start-AzureStorageBlobCopy` do PowerShell](/powershell/module/azure.storage/start-azurestorageblobcopy?view=azurermps-5.0.0) é uma opção para os administradores do Windows que estão acostumados com o PowerShell.  

- **AdlCopy**. O [AdlCopy](/azure/data-lake-store/data-lake-store-copy-data-azure-storage-blob) permite a cópia de dados de Azure Storage Blobs para o Data Lake Store. Ele também pode ser usado para copiar dados entre duas contas do Azure Data Lake Store. No entanto, não pode ser usado para copiar dados do Data Lake Store para Storage Blobs.

- **Distcp**. Se você tem um cluster HDInsight com acesso ao Data Lake Store, use as ferramentas do ecossistema do Hadoop, como o [Distcp](/azure/data-lake-store/data-lake-store-copy-data-wasb-distcp), para copiar dados bidirecionalmente em um armazenamento de cluster do HDInsight (WASB) para uma conta do Data Lake Store.

- **Sqoop**. O [Sqoop](/azure/hdinsight/hadoop/hdinsight-use-sqoop) é um projeto do Apache e uma parte do ecossistema do Hadoop. Ele vem pré-instalado em todos os clusters HDInsight. Permite a transferência de dados entre um cluster HDInsight e bancos de dados relacionais, como o SQL, Oracle, MySQL e assim por diante. O Sqoop é uma coleção de ferramentas relacionadas, incluindo importação e exportação. O Sqoop funciona com clusters HDInsight usando blobs do Armazenamento do Azure ou o armazenamento anexado do Data Lake Store.

- **PolyBase**. O [PolyBase](/sql/relational-databases/polybase/get-started-with-polybase) é uma tecnologia que acessa dados fora do banco de dados por meio da linguagem T-SQL. No SQL Server 2016, ele permite executar consultas em dados externos no Hadoop ou importar/exportar dados do Armazenamento de Blobs do Azure. No SQL Data Warehouse do Azure, você pode importar/exportar dados do Armazenamento de Blobs do Azure e do Azure Data Lake Store. Atualmente, o PolyBase é o método mais rápido de importar dados para o SQL Data Warehouse.

- **Linha de comando do Hadoop**. Quando você tem dados que residem em um nó de cabeçalho do cluster HDInsight, use o comando `hadoop -copyFromLocal` para copiar os dados para o armazenamento anexado do cluster, como o Azure Storage Blob ou o Azure Data Lake Store. Para usar o comando do Hadoop, primeiro você deve se conectar ao nó de cabeçalho. Depois de conectado, você pode carregar um arquivo no armazenamento.

## <a name="graphical-interface"></a>Interface gráfica

Considere as opções a seguir caso esteja transferindo apenas alguns arquivos ou objetos de dados e não precisar automatizar o processo.

- **Gerenciador de Armazenamento do Azure**. O [Gerenciador de Armazenamento do Azure](https://azure.microsoft.com/features/storage-explorer/) é uma ferramenta de multiplaforma que permite que você gerencie o conteúdo de suas contas de armazenamento do Azure. Ele permite carregar, baixar e gerenciar blogs, arquivos, filas, tabelas e entidades do Azure Cosmos DB. Use-o com o armazenamento de Blobs para gerenciar blobs e pastas, bem como carregar e baixar blobs entre o sistema de arquivos local e o armazenamento de Blobs ou entre contas de armazenamento.

- **Portal do Azure**. O armazenamento de Blobs e o Data Lake Store fornecem uma interface baseada na Web para explorar arquivos e carregar novos arquivos um de cada vez. Essa é uma boa opção se você não deseja instalar nenhuma ferramenta nem emitir comandos para explorar rapidamente os arquivos ou para simplesmente carregar uma série de novos.

## <a name="data-pipeline"></a>Pipeline de dados

**Azure Data Factory**. O [Azure Data Factory](/azure/data-factory/) é um serviço gerenciado mais adequado para a transferência regular de arquivos entre diversos serviços do Azure, locais ou uma combinação dos dois. Usando o Azure Data Factory, você pode criar e agendar fluxos de trabalho controlados por dados (chamados de pipelines) que ingerem dados de diferentes armazenamentos de dados. Ele pode processar e transformar dados usando serviços de computação como o Azure HDInsight Hadoop, Spark, Azure Data Lake Analytics e Azure Machine Learning. Crie fluxos de trabalho controlados por dados para [orquestrar](../technology-choices/pipeline-orchestration-data-movement.md) e automatizar a movimentação e transformação de dados.

## <a name="key-selection-criteria"></a>Principais Critérios de Seleção

Para cenários de transferência de dados, escolha o sistema apropriado para suas necessidades respondendo a essas perguntas:

- Você precisa transferir grandes quantidades de dados e fazer isso em uma conexão com a Internet levará muito tempo, será muito caro ou não confiável? Em caso afirmativo, considere o uso da transferência física.

- Você prefere gerar o script das tarefas de transferência de dados, para que elas sejam reutilizáveis? Nesse caso, selecione uma das opções de linha de comando ou o Azure Data Factory.

- Você precisa transferir uma grande quantidade de dados em uma conexão de rede? Nesse caso, selecione uma opção otimizada para Big Data.

- Você precisa transferir dados bidirecionalmente em um banco de dados relacional? Em caso afirmativo, escolha uma opção que dá suporte a um ou mais bancos de dados relacionais. Observe que algumas dessas opções também exigem um cluster Hadoop.

- Você precisa de uma orquestração automatizada de pipeline de dados ou de fluxo de trabalho? Em caso afirmativo, considere o uso do Azure Data Factory.

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades.

### <a name="physical-transfer"></a>Transferência física

| | Serviço de Importação/exportação do Azure | Azure Data Box |
| --- | --- | --- |
| Fator forma | HDDs ou SDDs SATA internos | Dispositivo de único hardware, seguro e à prova de adulteração |
| A Microsoft gerencia a logística de envio | Não  | sim |
| É integrado a produtos de parceiros | Não  | sim |
| Dispositivo personalizado | Não  | sim |

### <a name="command-line-tools"></a>Ferramentas de linha de comando

**Hadoop/HDInsight**

| | Distcp | Sqoop | CLI do Hadoop |
| --- | --- | --- | --- |
| Otimizado para Big Data | sim | sim |  sim |
| Copiar para o banco de dados relacional |  Não  | sim | Não  |
| Copiar do banco de dados relacional |  Não  | sim | Não  |
| Copiar para o armazenamento de Blobs |  sim | sim | sim |
| Copiar do armazenamento de Blobs | sim |  sim | Não  |
| Copiar para o Data Lake Store | sim | sim | sim |
| Copiar do Data Lake Store | sim | sim | Não  |

**Outros**

| | CLI do Azure | AzCopy | PowerShell | AdlCopy | PolyBase |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Plataformas compatíveis | Linux, OS X, Windows | Linux, Windows | Windows | Linux, OS X, Windows | SQL Server, SQL Data Warehouse do Azure | 
| Otimizado para Big Data | Não  | Não  | Não  | Sim <sup>1</sup> | Sim <sup>2</sup> |
| Copiar para o banco de dados relacional | Não  | Não  | Não  | Não  | sim | 
| Copiar do banco de dados relacional | Não  | Não  | Não  | Não  | sim | 
| Copiar para o armazenamento de Blobs | sim | sim | sim | Não  | sim | 
| Copiar do armazenamento de Blobs | sim | sim | sim | sim | sim |
| Copiar para o Data Lake Store | Não  | Não  | sim | sim |  sim | 
| Copiar do Data Lake Store | Não  | Não  | sim | sim | sim | 


[1] O AdlCopy é otimizado para a transferência de Big Data quando usado com uma conta do Data Lake Analytics.

[2] O [desempenho do PolyBase pode ser aumentado](/sql/relational-databases/polybase/polybase-guide#performance) com o push de computação para o Hadoop e o uso de [grupos de escala horizontal do PolyBase](/sql/relational-databases/polybase/polybase-scale-out-groups) para permitir a transferência paralela de dados entre instâncias do SQL Server e nós do Hadoop.

### <a name="graphical-interface-and-azure-data-factory"></a>Interface gráfica e o Azure Data Factory

| | Gerenciador de Armazenamento do Azure | Portal do Azure * | Fábrica de dados do Azure |
| --- | --- | --- | --- |
| Otimizado para Big Data | Não  | Não  | sim | 
| Copiar para o banco de dados relacional | Não  | Não  | sim |
| Copiar para o banco de dados relacional | Não  | Não  | sim |
| Copiar para o armazenamento de Blobs | sim | Não  | sim |
| Copiar do armazenamento de Blobs | sim | Não  | sim |
| Copiar para o Data Lake Store | Não  | Não  | sim |
| Copiar do Data Lake Store | Não  | Não  | sim |
| Carregar no armazenamento de Blobs | sim | sim | sim |
| Upload para o Data Lake Store | sim | sim | sim |
| Orquestrar transferências de dados | Não  | Não  | sim |
| Transformações de dados personalizadas | Não  | Não  | sim |
| Modelo de preços | Grátis | Grátis | Pagamento por uso |

\* Nesse caso, o portal do Azure significa o uso de ferramentas de exploração baseadas na Web para o armazenamento de Blobs e o Data Lake Store.

