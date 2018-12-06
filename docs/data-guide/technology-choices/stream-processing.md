---
title: Escolhendo uma tecnologia de processamento de fluxo
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 2e0d142bc5cd462703ef1ca4530a2104efdf3be3
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902620"
---
# <a name="choosing-a-stream-processing-technology-in-azure"></a>Escolhendo uma tecnologia de processamento de fluxo no Azure

Este artigo compara as opções de tecnologia de processamento de fluxo em tempo real no Azure.

O processamento de fluxo em tempo real consome mensagens do armazenamento baseado em fila ou em arquivo, processa as mensagens e encaminha o resultado para outra fila de mensagens, repositório de arquivos ou banco de dados. O processamento pode incluir consulta, filtragem e agregação de mensagens. Os mecanismos de processamento de fluxo precisam conseguir consumir intermináveis fluxos de dados e produzir resultados com latência mínima. Para obter mais informações, consulte [Processamento em tempo real](../big-data/real-time-processing.md).

## <a name="what-are-your-options-when-choosing-a-technology-for-real-time-processing"></a>Quais são as opções disponíveis ao escolher uma tecnologia de processamento em tempo real?
No Azure, todos os seguintes armazenamentos de dados atenderão aos requisitos básicos que dão suporte ao processamento em tempo real:
- [Stream Analytics do Azure](/azure/stream-analytics/)
- [HDInsight com Spark Streaming](/azure/hdinsight/spark/apache-spark-streaming-overview)
- [Apache Spark no Azure Databricks](/azure/azure-databricks/)
- [HDInsight com Storm](/azure/hdinsight/storm/apache-storm-overview)
- [Funções do Azure](/azure/azure-functions/functions-overview)
- [WebJobs no Serviço de Aplicativo do Azure](/azure/app-service/web-sites-create-web-jobs)

## <a name="key-selection-criteria"></a>Principais Critérios de Seleção

Para cenários de processamento em tempo real, comece escolhendo o serviço apropriado para suas necessidades respondendo a estas perguntas:

- Você prefere uma abordagem declarativa ou imperativa para a criação da lógica de processamento de fluxo?

- Você precisa de suporte interno para as janelas ou o processamento temporal?

- Os dados são recebidos em formatos além do Avro, JSON ou CSV? Nesse caso, considere opções que dão suporte a qualquer formato que usa um código personalizado.

- Você precisa dimensionar o processamento além de 1 GB/s? Nesse caso, considere as opções que podem ser dimensionadas com o tamanho do cluster. 

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades. 

### <a name="general-capabilities"></a>Funcionalidades gerais

| | Stream Analytics do Azure | HDInsight com Spark Streaming | Apache Spark no Azure Databricks | HDInsight com Storm | Funções do Azure | WebJobs no Serviço de Aplicativo do Azure |
| --- | --- | --- | --- | --- | --- | --- | 
| Programação | Linguagem de consulta do Stream Analytics, JavaScript | Scala, Python, Java | Scala, Python, Java, R | Java, C# | C#, F#, Node.js | C#, Node.js, PHP, Java, Python |
| Paradigma de programação | Declarativo | Combinação de declarativo e imperativo | Combinação de declarativo e imperativo | Imperativo | Imperativo | Imperativo |    
| Modelo de preços | [Unidades de streaming](https://azure.microsoft.com/pricing/details/stream-analytics/) | Por hora de cluster | [Unidades do Databricks](https://azure.microsoft.com/pricing/details/databricks/) | Por hora de cluster | Por execução de função e consumo de recursos | Por hora do plano de serviço de aplicativo |  

### <a name="integration-capabilities"></a>Funcionalidades de integração

| | Stream Analytics do Azure | HDInsight com Spark Streaming | Apache Spark no Azure Databricks | HDInsight com Storm | Funções do Azure | WebJobs no Serviço de Aplicativo do Azure |
| --- | --- | --- | --- | --- | --- | --- | 
| Entradas | Hubs de Eventos do Azure, Hub IoT do Azure e Armazenamento de Blobs do Azure  | Hubs de Eventos, Hub IoT, Kafka, HDFS, Blobs de Armazenamento, Azure Data Lake Store  | Hubs de Eventos, Hub IoT, Kafka, HDFS, Blobs de Armazenamento, Azure Data Lake Store  | Hubs de Eventos, Hub IoT, Blobs de Armazenamento, Azure Data Lake Store  | [Associações compatíveis](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Barramento de Serviço, Filas de Armazenamento, Blobs de Armazenamento, Hubs de Eventos, WebHooks, Cosmos DB, Arquivos |
| Coletores |  Azure Data Lake Store, Banco de Dados SQL do Azure, Blobs de armazenamento, Hubs de Eventos, Power BI, Armazenamento de Tabelas, Filas do Barramento de Serviço, Tópicos do Barramento de Serviço, Cosmos DB, Azure Functions  | HDFS, Kafka, Blobs de armazenamento, Azure Data Lake Store, Cosmos DB | HDFS, Kafka, Blobs de armazenamento, Azure Data Lake Store, Cosmos DB | Hubs de Eventos, Barramento de Serviço, Kafka | [Associações compatíveis](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Barramento de Serviço, Filas de Armazenamento, Blobs de Armazenamento, Hubs de Eventos, WebHooks, Cosmos DB, Arquivos | 

### <a name="processing-capabilities"></a>Funcionalidades de processamento

| | Stream Analytics do Azure | HDInsight com Spark Streaming | Apache Spark no Azure Databricks | HDInsight com Storm | Funções do Azure | WebJobs no Serviço de Aplicativo do Azure |
| --- | --- | --- | --- | --- | --- | --- | 
| Suporte a janelas internas/temporal | SIM | sim | sim | sim | Não | Não  |
| Formatos de dados de entrada | Avro, JSON ou CSV, codificados em UTF-8 | Qualquer formato que usa um código personalizado | Qualquer formato que usa um código personalizado | Qualquer formato que usa um código personalizado | Qualquer formato que usa um código personalizado | Qualquer formato que usa um código personalizado |
| Escalabilidade | [Partições de consulta](/azure/stream-analytics/stream-analytics-parallelization) | Limitado pelo tamanho do cluster | Limitado pela configuração de escala de cluster do Databricks | Limitado pelo tamanho do cluster | Até 200 instâncias de aplicativo de funções processadas em paralelo | Limitado pela capacidade do plano de serviço de aplicativo | 
| Suporte à chegada tardia e manipulação de eventos fora de ordem | SIM | sim | sim | sim | Não | Não  |

Consulte também:

- [Escolhendo uma tecnologia de ingestão de mensagens em tempo real](./real-time-ingestion.md)
- [Processamento em tempo real](../big-data/real-time-processing.md)
