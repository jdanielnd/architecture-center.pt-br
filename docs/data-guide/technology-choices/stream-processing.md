---
title: Escolhendo uma tecnologia de processamento de fluxo
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: e06f46e2951159219bd8cc430102e2ec0c5d6d4d
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-stream-processing-technology-in-azure"></a>Escolhendo uma tecnologia de processamento de fluxo no Azure

Este artigo compara as opções de tecnologia de processamento de fluxo em tempo real no Azure.

O processamento de fluxo em tempo real consome mensagens do armazenamento baseado em fila ou em arquivo, processa as mensagens e encaminha o resultado para outra fila de mensagens, repositório de arquivos ou banco de dados. O processamento pode incluir consulta, filtragem e agregação de mensagens. Os mecanismos de processamento de fluxo precisam conseguir consumir intermináveis fluxos de dados e produzir resultados com latência mínima. Para obter mais informações, consulte [Processamento em tempo real](../scenarios/real-time-processing.md).

## <a name="what-are-your-options-when-choosing-a-technology-for-real-time-processing"></a>Quais são as opções disponíveis ao escolher uma tecnologia de processamento em tempo real?
No Azure, todos os seguintes armazenamentos de dados atenderão aos requisitos básicos que dão suporte ao processamento em tempo real:
- [Stream Analytics do Azure](/azure/stream-analytics/)
- [HDInsight com Spark Streaming](/azure/hdinsight/spark/apache-spark-streaming-overview)
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
| | Stream Analytics do Azure | HDInsight com Spark Streaming | HDInsight com Storm | Funções do Azure | WebJobs no Serviço de Aplicativo do Azure |
| --- | --- | --- | --- | --- | --- | 
| Programação | Linguagem de consulta do Stream Analytics, JavaScript | Scala, Python, Java | Java, C# | C#, F#, Node.js | C#, Node.js, PHP, Java, Python |
| Paradigma de programação | Declarativo | Combinação de declarativo e imperativo | Imperativo | Imperativo | Imperativo |    
| Modelo de preços | Por unidades de streaming | Por hora de cluster | Por hora de cluster | Por execução de função e consumo de recursos | Por hora do plano de serviço de aplicativo |  

### <a name="integration-capabilities"></a>Funcionalidades de integração
| | Stream Analytics do Azure | HDInsight com Spark Streaming | HDInsight com Storm | Funções do Azure | WebJobs no Serviço de Aplicativo do Azure |
| --- | --- | --- | --- | --- | --- | 
| Entradas | [Entradas do Stream Analytics](/azure/stream-analytics/stream-analytics-define-inputs)  | Hubs de Eventos, Hub IoT, Kafka, HDFS  | Hubs de Eventos, Hub IoT, Blobs de Armazenamento, Azure Data Lake Store  | [Associações compatíveis](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Barramento de Serviço, Filas de Armazenamento, Blobs de Armazenamento, Hubs de Eventos, WebHooks, Cosmos DB, Arquivos |
| Coletores |  [Saídas do Stream Analytics](/azure/stream-analytics/stream-analytics-define-outputs) | HDFS | Hubs de Eventos, Barramento de Serviço, Kafka | [Associações compatíveis](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Barramento de Serviço, Filas de Armazenamento, Blobs de Armazenamento, Hubs de Eventos, WebHooks, Cosmos DB, Arquivos | 

### <a name="processing-capabilities"></a>Funcionalidades de processamento
| | Stream Analytics do Azure | HDInsight com Spark Streaming | HDInsight com Storm | Funções do Azure | WebJobs no Serviço de Aplicativo do Azure |
| --- | --- | --- | --- | --- | --- | 
| Suporte a janelas internas/temporal | sim | sim | sim | Não  | Não  |
| Formatos de dados de entrada | Avro, JSON ou CSV, codificados em UTF-8 | Qualquer formato que usa um código personalizado | Qualquer formato que usa um código personalizado | Qualquer formato que usa um código personalizado | Qualquer formato que usa um código personalizado |
| Escalabilidade | [Partições de consulta](/azure/stream-analytics/stream-analytics-parallelization) | Limitado pelo tamanho do cluster | Limitado pelo tamanho do cluster | Até 200 instâncias de aplicativo de funções processadas em paralelo | Limitado pela capacidade do plano de serviço de aplicativo | 
| Suporte à chegada tardia e manipulação de eventos fora de ordem | sim | sim | sim | Não  | Não  |

Consulte também:

- [Escolhendo uma tecnologia de ingestão de mensagens em tempo real](./real-time-ingestion.md)
- [Comparando o Apache Storm com o Azure Stream Analytics](/azure/stream-analytics/stream-analytics-comparison-storm)
- [Processamento em tempo real](../scenarios/real-time-processing.md)
