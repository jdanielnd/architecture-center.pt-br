---
title: Solução de problemas para o Azure Databricks usando o Azure Monitor de desempenho
titleSuffix: ''
description: Use painéis do Grafana para solucionar problemas de desempenho no Azure Databricks
author: petertaylor9999
ms.date: 04/02/2019
ms.topic: ''
ms.service: ''
ms.subservice: ''
ms.openlocfilehash: 49ec63d0c45ab388ca83b3ab0562428327539619
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/18/2019
ms.locfileid: "59740367"
---
# <a name="troubleshoot-performance-bottlenecks-in-azure-databricks"></a>Solucionar problemas de gargalos de desempenho no Azure Databricks

Este artigo descreve como usar os painéis de monitoramento para localizar gargalos de desempenho em trabalhos do Spark no Azure Databricks.

[O Azure Databricks](/azure/azure-databricks/) é um [Apache Spark](https://spark.apache.org/)– com base em serviço de análise que torna mais fácil desenvolver e implantar a análise de big data rapidamente. Monitorar e solucionar problemas de desempenho são fundamental durante a operação de cargas de trabalho de produção do Azure Databricks. Para identificar problemas de desempenho comuns, é útil usar visualizações de monitoramento com base em dados de telemetria.

## <a name="prerequisites"></a>Pré-requisitos

Para configurar os painéis do Grafana mostrados neste artigo:

- Configure o cluster do Databricks para enviar telemetria para um espaço de trabalho do Log Analytics, usando a biblioteca de monitoramento do Azure Databricks. Para obter detalhes, consulte [configurar o Azure Databricks para enviar métricas para o Azure Monitor](./configure-cluster.md).

- Implante o Grafana em uma máquina virtual. Ver [usar painéis para visualizar as métricas do Azure Databricks](./dashboards.md).

O painel do Grafana que é implantado inclui um conjunto de visualizações de série temporal. Cada gráfico é um gráfico de série temporal de métricas relacionadas a um trabalho do Apache Spark, os estágios do trabalho e tarefas que compõem cada estágio.

## <a name="azure-databricks-performance-overview"></a>Visão geral de desempenho do Azure Databricks

O Azure Databricks se baseia no Apache Spark, um sistema genérico de computação distribuído. Código do aplicativo, conhecido como uma **trabalho**, é executado em um cluster do Apache Spark, coordenado pelo Gerenciador de cluster. Em geral, um trabalho é a unidade de nível mais alto de computação. Um trabalho representa a concluir a operação executada pelo aplicativo Spark. Uma operação típica inclui a leitura de uma fonte de dados, Aplicando transformações de dados e gravar os resultados para o armazenamento ou outro destino.

Trabalhos são divididos em **estágios**. O trabalho avança pelas fases sequencialmente, o que significa que estágios posteriores devem aguardar estágios anteriores concluir. Estágios contêm grupos de idênticos **tarefas** que podem ser executadas em paralelo em vários nós do cluster Spark. Tarefas são a unidade mais granular de execução está ocorrendo em um subconjunto dos dados.

As seções a seguir descrevem algumas visualizações de painel que são úteis para solucionar problemas de desempenho.

## <a name="job-and-stage-latency"></a>Latência de estágio e de trabalho

Latência do trabalho é a duração da execução de um trabalho de quando ele é iniciado até que ela seja concluída. Ele é mostrado como percentuais de execução de um trabalho por ID do aplicativo e o cluster, para permitir que a visualização de exceções. O gráfico a seguir mostra um histórico de trabalho em que o 90 º percentil atingido 50 segundos, mesmo que o 50 º percentil foi consistentemente cerca de 10 segundos.

![Gráfico mostrando a latência do trabalho](./_images/grafana-job-latency.png)

Investigar a execução do trabalho ao cluster e aplicativo, procura picos na latência. Depois de clusters e aplicativos com alta latência são identificados, passe para investigar a latência de estágio.

Latência de estágio também é mostrada como percentuais para permitir que a visualização de exceções. Latência de estágio é dividida por cluster, o aplicativo e o nome do estágio. Identificar picos na latência de tarefa no gráfico para determinar quais tarefas estão mantendo volta a conclusão do estágio.

![Gráfico mostrando a latência de estágio](./_images/grafana-stage-latency.png)

O gráfico de taxa de transferência de cluster mostra o número de trabalhos, estágios e as tarefas concluídas por minuto. Isso ajuda você a entender a carga de trabalho em termos de número relativo de etapas e tarefas por trabalho. Aqui você pode ver que o número de trabalhos por minuto de intervalos entre 2 e 6, enquanto o número de estágios é cerca de 12 &ndash; 24 por minuto.

![Taxa de transferência do grafo mostrando cluster](./_images/grafana-cluster-throughput.png)

## <a name="sum-of-task-execution-latency"></a>Soma de latência de execução de tarefa

Esta visualização mostra a soma da latência de execução de tarefa por execução em um cluster de host. Use este gráfico para detectar as tarefas executadas lentamente devido à lentidão host em um cluster ou um misallocation de tarefas por executor. No gráfico a seguir, a maioria dos hosts tem uma soma de cerca de 30 segundos. No entanto, dois dos hosts têm somas que passe o mouse cerca de 10 minutos. Os hosts estão muito lentos ou o número de tarefas por executor é misallocated.

![Soma do grafo mostrando da execução de tarefa por host](./_images/grafana-sum-task-exec.png)

O número de tarefas por executor mostra que dois executores são atribuídos a um número desproporcional de tarefas, causando um afunilamento.

![Gráfico mostrando tarefas por executor](./_images/grafana-tasks-per-exec.png)

## <a name="task-metrics-per-stage"></a>Métricas de tarefa por estágio

A visualização de métricas de tarefa fornece a análise de custo para uma execução de tarefa. Você pode usá-lo consulte o tempo relativo gasto em tarefas, como a serialização e desserialização. Esses dados poderá mostrar oportunidades para otimizar &mdash; por exemplo, usando [difundir variáveis](https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#broadcast-variables) para evitar o envio de dados. As métricas de tarefa também mostram o tamanho dos dados em ordem aleatória para uma tarefa, e a ordem aleatória de leitura e gravação vezes. Se esses valores forem altos, isso significa que muitos dados são movidos pela rede.

Outra métrica de tarefa é o atraso de Agendador, que mede o quanto tempo demora para agendar uma tarefa. Idealmente, esse valor deve ser baixo em comparação com o tempo de computação de executor, que é o tempo gasto na execução, na verdade, a tarefa.

O gráfico a seguir mostra um tempo de espera do Agendador (3.7 s) que exceda o tempo de computação do executor (1.1 s). Isso significa que mais tempo é gasto aguardando tarefas sejam agendadas que fazendo o trabalho real.

![Gráfico que mostra as métricas de tarefa por estágio](./_images/grafana-metrics-per-stage.png)

Nesse caso, o problema foi causado por ter muitas partições, o que causou muita sobrecarga. Reduzindo o número de partições reduzido o tempo de atraso do Agendador. O gráfico a seguir mostra que a maior parte do tempo gasto na execução da tarefa.

![Gráfico que mostra as métricas de tarefa por estágio](./_images/grafana-metrics-per-stage2.png)

## <a name="streaming-throughput-and-latency"></a>Streaming de taxa de transferência e latência

Streaming de taxa de transferência está diretamente relacionado ao fluxo estruturado. Há duas métricas importantes associadas com streaming de taxa de transferência: Linhas de entrada por segundo e processadas linhas por segundo. Se as linhas de entrada por segundo outpaces linhas processadas por segundo, isso significa que o sistema de processamento de fluxo está atrasado. Além disso, se os dados de entrada provêm de Hubs de eventos ou Kafka, em seguida, as linhas de entrada por segundo devem manter a taxa de ingestão de dados no front-end.

Dois trabalhos podem ter uma taxa de transferência de cluster semelhante, mas as métricas de transmissão muito diferentes. Captura de tela a seguir mostra duas cargas de trabalho diferentes. Elas são semelhantes em termos de taxa de transferência de cluster (trabalhos, etapas e tarefas por minuto). Mas a segunda execução processa linhas de 12.000/seg contra 4.000 linhas/s.

![Taxa de transferência de gráfico mostrando streaming](./_images/grafana-streaming-throughput.png)

Streaming de taxa de transferência geralmente é uma métrica de negócios melhor que a taxa de transferência de cluster, porque ele mede o número de registros de dados que são processados.

## <a name="resource-consumption-per-executor"></a>Consumo de recursos por executor

Essas métricas ajudam a compreender o trabalho que executa cada executor.

**As métricas de percentual** medir quanto tempo um executor gasta em várias coisas, expressadas como uma proporção de tempo gasto em comparação com o tempo de computação geral executor. As métricas são:

- % Tempo de serializar
- % Tempo de desserializar
- % Tempo da CPU executor
- % De tempo da JVM

Estas visualizações mostram o quanto cada uma dessas métricas contribui para o executor geral de processamento.

![Gráfico que mostra as métricas de percentual](./_images/grafana-percentage.png)

**Métricas de embaralhar** métricas relacionadas aos dados, a ordem aleatória entre os executores.

- E/s de ordem aleatória
- Memória de ordem aleatória
- Uso do sistema de arquivos
- Uso do disco

## <a name="common-performance-bottlenecks"></a>Gargalos de desempenho comuns

São duas gargalos de desempenho comuns no Spark *intervalos irregulares de tarefa* e uma *contagem de partições de ordem aleatória não ideal*.

### <a name="task-stragglers"></a>Intervalos irregulares de tarefa

Os estágios de um trabalho são executados sequencialmente, com bloqueio de estágios posteriores de estágios anteriores. Se uma tarefa for executada mais lentamente do que outras tarefas de uma partição de ordem aleatória, todas as tarefas no cluster devem aguardar a tarefa lenta ficar atualizado antes que o estágio pode terminar. Isso pode ocorrer pelos seguintes motivos:

1. Um host ou grupo de hosts estão muito lentos. Sintomas: Tarefa alta, estágio, ou latência do trabalho e taxa de transferência baixa do cluster. A adição de latências de tarefas por host não será distribuída uniformemente. No entanto, o consumo de recursos será distribuído uniformemente entre os executores.

1. Tarefas têm uma agregação cara executar (distorção de dados). Sintomas: Tarefa alta, estágio, ou trabalho e taxa de transferência baixa de cluster, mas o somatório de latências de cada host é distribuído uniformemente. Consumo de recursos será distribuído uniformemente entre os executores.

1. Se as partições são de tamanho desigual, uma partição maior pode causar a execução de tarefa desbalanceada (inclinação de partição). Sintomas: Consumo de recursos do executor é alto em comparação com outros executores em execução no cluster. Todas as tarefas em execução em que o executor serão executado lentamente e manter a execução do estágio no pipeline. Esses estágios são considerados *estágio barreiras*.

### <a name="non-optimal-shuffle-partition-count"></a>Contagem de partições não ideal em ordem aleatória

Durante uma consulta de streaming estruturada, a atribuição de uma tarefa para um executor é uma operação de uso intensivo de recursos para o cluster. Se os dados de ordem aleatória não forem o tamanho ideal, a quantidade de atraso para uma tarefa afetará negativamente taxa de transferência e latência. Se houver muito poucas partições, os núcleos do cluster serão subutilizados que pode resultar em processamento ineficiência. Por outro lado, se houver um número excessivo de partições, há uma grande quantidade de sobrecarga de gerenciamento para um pequeno número de tarefas.

Use métricas de consumo de recursos para solucionar problemas de distorção de partição e misallocation de executores no cluster. Se uma partição estiverem distorcida, recursos do executor de serão elevados em comparação com outros executores em execução no cluster.

Por exemplo, o gráfico a seguir mostra que a memória usada pelo embaralhamento nos dois primeiros executores é 90 X maior do que os outros executores:

![Gráfico que mostra as métricas de percentual](./_images/grafana-shuffle-memory.png)