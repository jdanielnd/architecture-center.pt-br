---
title: Lote de pontuação com modelos do R no Azure
description: Execute a pontuação com modelos do R usando o lote do Azure e um conjunto de dados com base na previsão de vendas do varejo repositório em lote.
author: njray
ms.date: 03/29/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: 4fa57168c337b01c8e7d0fc86ba54fee59a7ae47
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887914"
---
# <a name="batch-scoring-of-r-machine-learning-models-on-azure"></a>Lote de pontuação de modelos de aprendizado de máquina do R no Azure

Essa arquitetura de referência mostra como executar o lote de pontuação com modelos do R usando o lote do Azure. O cenário se baseia na previsão de vendas do varejo store, mas essa arquitetura pode ser generalizada para qualquer cenário que exigem a geração de previsões em um grande scaler usando modelos do R. Há uma implantação de referência para essa arquitetura de referência disponível no [GitHub][github].

![Diagrama da arquitetura][0]

**Cenário**: Uma cadeia de supermercado deve prever as vendas de produtos em futuro trimestre. A previsão permite que a empresa a gerenciar melhor o sua cadeia de fornecimento e certifique-se de que ele pode atender à demanda de produtos em cada um de seus repositórios. A empresa atualiza suas previsões de cada semana conforme novos dados de vendas da semana anterior se torna disponíveis e o produto a estratégia de marketing para o próximo trimestre é definido. Previsões de quantil são geradas para estimar a incerteza das previsões de vendas individuais.

O processamento envolve as seguintes etapas:

1. Um aplicativo lógico dispara o processo de geração de previsão de uma vez por semana.

1. O aplicativo lógico inicia uma instância de contêiner do Azure executando o Agendador de contêiner do Docker, que dispara os pontuação de trabalhos no cluster em lotes.

1. Pontuação de trabalhos executados em paralelo em todos os nós do cluster em lotes. Cada nó:

    1. Extrai o trabalhador imagem do Docker do Hub do Docker e inicia um contêiner.

    1. Lê dados de entrada e previamente treinados modelos de R do armazenamento de BLOBs do Azure.

    1. Classifica os dados para produzir as previsões.

    1. Grava os resultados de previsão no armazenamento de blob.

A figura abaixo mostra as vendas previstas para quatro produtos (SKUs) em um repositório. A linha preta é o histórico de vendas, a linha tracejada é a mediana (q50) de previsão, a banda rosa representa os percentuais de 25 e o quinto setenta e faixa azul representa os quinto e o quinto noventa percentis.

![Previsões de vendas][1]

## <a name="architecture"></a>Arquitetura

Essa arquitetura é formada pelos componentes a seguir.

[O Azure Batch] [ batch] é usado para executar trabalhos de geração de previsão em paralelo em um cluster de máquinas virtuais. As previsões são feitas usando a máquina previamente treinada implementados em R. o lote do Azure de modelos de aprendizado podem dimensionar automaticamente o número de VMs com base no número de trabalhos enviados para o cluster. Em cada nó, um script R é executado dentro de um contêiner do Docker para pontuar dados e gerar previsões.

[O armazenamento de BLOBs do Azure] [ blob] é usado para armazenar os dados de entrada, os modelos de aprendizado de máquina previamente treinado e os resultados de previsão. Ele oferece armazenamento muito econômico para o desempenho que exige essa carga de trabalho.

[As instâncias de contêiner do Azure] [ aci] fornecem computação sem servidor sob demanda. Nesse caso, uma instância de contêiner é implantada em um agendamento para disparar os trabalhos em lotes que geram as previsões. Os trabalhos em lotes são disparados de um script de R usando o [doAzureParallel][doAzureParallel] pacote. A instância de contêiner será desligado automaticamente depois que os trabalhos tiverem terminado.

[Aplicativos lógicos do Azure] [ logic-apps] disparar o fluxo de trabalho inteiro ao implantar as instâncias de contêiner em um agendamento. Um conector de instâncias de contêiner do Azure em aplicativos lógicos permite que uma instância a ser implantado em um intervalo de eventos de gatilho.

## <a name="performance-considerations"></a>Considerações sobre o desempenho

### <a name="containerized-deployment"></a>Implantação em contêineres

Com essa arquitetura, todos os scripts de R executados no [Docker](https://www.docker.com/) contêineres. Isso garante que os scripts executados em um ambiente consistente, com a mesma versão do R e as versões de pacotes, cada vez. Imagens do Docker separadas são usadas para os contêineres de agendador e de trabalho, porque cada um tem um conjunto diferente de dependências de pacotes de R.

As instâncias de contêiner do Azure fornece um ambiente sem servidor para executar o contêiner de Agendador. O contêiner de Agendador executa um script R que dispara os trabalhos de pontuação individuais em execução em um cluster de lote do Azure.

Cada nó do cluster em lotes executa o contêiner de trabalho que executa o script de pontuação.

### <a name="parallelizing-the-workload"></a>Paralelizar a carga de trabalho

Quando o lote de pontuação dados com modelos do R, considere como paralelizar a carga de trabalho. Os dados de entrada devem ser particionados de alguma forma para que a operação de pontuação pode ser distribuída entre os nós do cluster. Tente diferentes abordagens para descobrir a melhor opção para distribuir sua carga de trabalho. Em uma base caso a caso, considere o seguinte:

- A quantidade de dados pode ser carregado e processado na memória de um único nó.
- A sobrecarga de cada trabalho em lotes de inicialização.
- A sobrecarga de carregar os modelos de R.

O cenário usado neste exemplo, os objetos de modelo são grandes e demora apenas alguns segundos para gerar uma previsão de produtos individuais. Por esse motivo, você pode agrupar os produtos e executar um único trabalho de lote por nó. Um loop dentro de cada trabalho gera previsões para os produtos em sequência. Esse método acaba sendo a maneira mais eficiente paralelizar essa carga de trabalho específica. Isso evita a sobrecarga de iniciar vários trabalhos em lotes menores e repetidamente ao carregar os modelos de R.

Uma abordagem alternativa é disparar um trabalho de lote por produto. O lote do Azure automaticamente uma fila de trabalhos de formulários e enviá-los a ser executado no cluster como nós se tornam disponíveis. Use [dimensionamento automático] [ autoscale] para ajustar o número de nós no cluster, dependendo do número de trabalhos. Essa abordagem faz mais sentido se demorar um tempo relativamente longo para concluir cada operação de pontuação, justificar a sobrecarga de iniciar os trabalhos e recarregar os objetos de modelo. Essa abordagem também é mais simples de implementar e oferece a flexibilidade para usar o dimensionamento automático — uma consideração importante se o tamanho da carga de trabalho total não é conhecido de antemão.

## <a name="monitoring-and-logging-considerations"></a>Considerações de monitoramento e registro em log

### <a name="monitoring-azure-batch-jobs"></a>Monitorando trabalhos de lote do Azure

Monitorar e encerrar trabalhos em lotes do **trabalhos** painel da conta do lote no portal do Azure. Monitorar o cluster de lote, incluindo o estado de nós individuais, do **Pools** painel.

### <a name="logging-with-doazureparallel"></a>Registro em log com doAzureParallel

O pacote doAzureParallel automaticamente coleta logs de todos os stdout/stderr para todos os trabalhos enviados no lote do Azure. Eles podem ser encontrados na conta de armazenamento criada na instalação. Para exibi-los, use uma ferramenta de navegação de armazenamento, como [Azure Storage Explorer] [ storage-explorer] ou portal do Azure.

Para depurar rapidamente os trabalhos em lote durante o desenvolvimento, imprimir os logs em sua sessão R local usando o [getJobFiles][getJobFiles] função de doAzureParallel.

## <a name="cost-considerations"></a>Considerações de custo

Os recursos de computação usados nesta arquitetura de referência são os componentes de custo mais alto. Para este cenário, um cluster de tamanho fixo é criado sempre que o trabalho é disparado e, em seguida, desligado após a conclusão do trabalho. O custo é incorrido apenas enquanto os nós de cluster estão iniciando, executando ou desligar. Essa abordagem é adequada para um cenário em que os recursos de computação necessários para gerar as previsões permanecem relativamente constantes de um trabalho para o trabalho.

Em cenários em que a quantidade de computação necessária para concluir o trabalho não é conhecida de antemão, pode ser mais adequado ao usar o dimensionamento automático. Com essa abordagem, o tamanho do cluster é dimensionado para cima ou para baixo, dependendo do tamanho do trabalho. O lote do Azure dá suporte a uma variedade de fórmula de dimensionamento automático que você pode definir ao definir o cluster usando o [doAzureParallel][doAzureParallel] API.

Para alguns cenários, o tempo entre trabalhos pode ser muito curto para desligar e iniciar o cluster. Nesses casos, manter o cluster em execução entre trabalhos, se apropriado.

DoAzureParallel e lote do Azure suportam o uso de VMs de baixa prioridade. Essas VMs são fornecidos com um desconto significativo, mas a riscos que está sendo apropriados por outras cargas de trabalho de prioridade mais alta. Portanto, o uso dessas VMs não são recomendados para cargas de trabalho de produção críticos. No entanto, eles são muito úteis para experimentais ou cargas de trabalho de desenvolvimento.

## <a name="deployment"></a>Implantação

Para implantar essa arquitetura de referência, siga as etapas descritas a [GitHub][github] repositório.


[0]: ./_images/batch-scoring-r-models.png
[1]: ./_images/sales-forecasts.png
[aci]: /azure/container-instances/container-instances-overview
[autoscale]: /azure/batch/batch-automatic-scaling
[batch]: /azure/batch/batch-technical-overview
[blob]: /azure/storage/blobs/storage-blobs-introduction
[doAzureParallel]: https://github.com/Azure/doAzureParallel/blob/master/docs/32-autoscale.md
[getJobFiles]: /azure/machine-learning/service/how-to-train-ml-models
[github]: https://github.com/Azure/RBatchScoring
[logic-apps]: /azure/logic-apps/logic-apps-overview
[storage-explorer]: /azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows