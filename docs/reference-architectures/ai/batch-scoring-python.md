---
title: Pontuação em lote de modelos Python no Azure
description: Compile uma solução escalonável para modelos de pontuação em lote com agendamento em paralelo usando o Serviço Azure Machine Learning.
author: njray
ms.date: 01/30/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai, AI
ms.openlocfilehash: 9341b9e4c17025e9623902a6202076c352b237b9
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640534"
---
# <a name="batch-scoring-of-python-machine-learning-models-on-azure"></a>Lote de pontuação de modelos de aprendizado de máquina do Python no Azure

Esta arquitetura de referência mostra como compilar uma solução escalonável para pontuar vários modelos em lote com agendamento em paralelo usando o Serviço Azure Machine Learning. A solução pode ser usada como um modelo e pode ser generalizada para problemas diferentes.

Há uma implantação de referência para essa arquitetura de referência disponível no [GitHub][github].

![Pontuação em lote de modelos Python no Azure](./_images/batch-scoring-python.png)

**Cenário**: A solução monitora a operação de um grande número de dispositivos em uma configuração de IoT, em que cada dispositivo envia leituras do sensor continuamente. Assume-se que cada dispositivo é associado com modelos de detecção de anomalias pré-treinados que precisam ser usados para prever se uma série de medidas, agregadas em um intervalo de tempo predefinido, correspondem a uma anomalia ou não. Em cenários do mundo real, isso seria um fluxo de leituras de sensor que precisariam ser filtrados e agregados antes de serem usados em treinamento ou pontuação em tempo real. Para manter a simplicidade, essa solução usa o mesmo arquivo de dados na execução de trabalhos de pontuação.

Essa arquitetura de referência foi projetada para cargas de trabalho que são disparadas de forma agendada. O processamento envolve as seguintes etapas:

1. Envie leituras do sensor para ingestão para os Hubs de Eventos do Azure.
2. Execute o processamento do fluxo e armazene os dados brutos.
3. Envie os dados para um cluster de Machine Learning que esteja pronto para começar a fazer o trabalho. Cada nó no cluster executa um trabalho de pontuação para um sensor específico. 
4. Execute o pipeline de pontuação, que executa os trabalhos de pontuação em paralelo usando scripts Python de Machine Learning. O pipeline é criado, publicado e agendado para ser executado em um intervalo de tempo predefinido.
5. Gere previsões e armazene-as no armazenamento de Blobs para consumo posterior.

## <a name="architecture"></a>Arquitetura

Essa arquitetura é formada pelos seguintes componentes:

[Hubs de Eventos do Azure][event-hubs]. Esse serviço de ingestão de mensagens pode incluir milhões de mensagens de eventos por segundo. Nesta arquitetura, os sensores enviam um fluxo de dados para o hub de eventos.

[Azure Stream Analytics][stream-analytics]. Um mecanismo de processamento de eventos. Um trabalho do Stream Analytics lê os fluxos de dados do hub de eventos e executa o processamento de fluxo.

[Banco de Dados SQL do Azure][sql-database]. Os dados das leituras do sensor são carregados no Banco de Dados SQL. SQL é uma maneira familiar para armazenar os dados transmitidos e processados (que são estruturados e tabulares), mas outros armazenamentos de dados podem ser usados.

[Serviço do Azure Machine Learning][amls]. Machine Learning é um serviço de nuvem para treinamento, pontuação, implantação e gerenciamento de modelos de machine learning em escala. No contexto de pontuação de lote, o Machine Learning cria um cluster de máquinas virtuais sob demanda com uma opção de dimensionamento automático, em que cada nó no cluster executa um trabalho de pontuação para um sensor específico. Os trabalhos de pontuação são executados em paralelo como etapas de script do Python que são enfileiradas e gerenciadas pelo Machine Learning. Essas etapas são parte de um pipeline do Machine Learning que é criado, publicado e agendado para ser executado em um intervalo predefinido de tempo.

[Armazenamento de Blobs do Azure][storage]. Os contêineres de blob são usados para armazenar os modelos pré-treinados, os dados e as previsões de saída. Os modelos são carregados no Armazenamento de Blobs no notebook [01_create_resources.ipynb][create-resources]. Esses modelos de [SVM de classe única][one-class-svm] são treinados com dados que representam valores de sensores diferentes em dispositivos diferentes. A solução pressupõe que os valores de dados foram agregados em um intervalo fixo de tempo.

[Registro de Contêiner do Azure][acr]. O [script][pyscript] de pontuação em Python é executado em contêineres do Docker que são criados em cada nó do cluster, onde ele lê os dados de sensor relevantes, gera previsões e as armazena no Armazenamento de Blobs.

## <a name="performance-considerations"></a>Considerações sobre o desempenho

Para modelos Python normais, é geralmente aceito que as CPUs são suficientes para lidar com a carga de trabalho. Esta arquitetura usa CPUs. No entanto, para [cargas de trabalho de aprendizado profundo][deep], GPUs geralmente superam as CPUs por uma quantidade considerável &mdash; um cluster considerável de CPUs normalmente é necessário para obter um desempenho comparável.

### <a name="parallelizing-across-vms-versus-cores"></a>Paralelização entre VMs em vez de núcleos

Ao executar processos de pontuação de vários modelos em modo de lote, os trabalhos precisam ser colocados em paralelo entre VMs. Há duas abordagens possíveis:

* Criar um cluster maior usando VMs de baixo custo.

* Criar um cluster menor usando VMs de alto desempenho com mais núcleos disponíveis em cada uma.

Em geral, a pontuação de modelos Python padrão não é tão exigente quanto a pontuação de modelos de aprendizado profundo, e um pequeno cluster deve ser capaz de lidar com um grande número de modelos em fila eficientemente. Você pode aumentar o número de nós de cluster de acordo com o aumento dos tamanhos do conjunto de dados.

Por questões de conveniência neste cenário, uma tarefa de pontuação é enviada dentro de uma única etapa de pipeline do Machine Learning. No entanto, pode ser mais eficiente pontuar várias partes de dados dentro da mesma etapa de pipeline. Nesses casos, escreva código personalizado para ler vários conjuntos de dados e executar o script de pontuação para eles durante uma mesma execução de etapa.

## <a name="management-considerations"></a>Considerações de gerenciamento

- **Monitorar trabalhos**. É importante monitorar o progresso de trabalhos em execução, mas pode ser um desafio monitorá-los em um cluster de nós ativos. Para inspecionar o estado de nós no cluster, use o [Portal do Azure][portal] para gerenciar o [workspace do machine learning][ml-workspace]. Se um nó estiver inativo ou se um trabalho falhar, os logs de erro serão salvos no armazenamento de blobs e também poderão ser acessados na seção Pipelines. Para um monitoramento mais avançado, conecte os logs ao [Application Insights][app-insights], ou execute processos separados para sondar o estado do cluster e de seus trabalhos.
- **Registro em log**. Os logs do Serviço Machine Learning registram todos os stdout/stderr da conta do Armazenamento do Azure associada. Para navegar facilmente nos arquivos de log, use uma ferramenta de navegação de armazenamento, como o [Gerenciador de Armazenamento do Azure][explorer].

## <a name="cost-considerations"></a>Considerações de custo

Os componentes mais caros usados nesta arquitetura de referência são os recursos de computação. O tamanho do cluster de cálculo é aumentado e reduzido dependendo dos trabalhos na fila. Habilite o dimensionamento automático por meio de programação com o SDK do Python, modificando a configuração de provisionamento da computação. Ou use a [CLI do Azure][cli] para definir os parâmetros de dimensionamento automático do cluster.

Para trabalho que não exige processamento imediato, configure a fórmula de dimensionamento automático a fim de definir o estado padrão (mínimo) como um cluster sem nós. Com essa configuração, o cluster começa com zero nós e só pode ser escalado verticalmente quando detecta os trabalhos na fila. Se o processo de pontuação do lote acontecer apenas algumas vezes por dia, essa configuração permitirá uma economia considerável.

Talvez o dimensionamento automático não seja apropriado para trabalhos em lote que aconteçam muito próximos uns dos outros. O tempo de ativação e desativação de um cluster também incorre em um custo, portanto, se uma carga de trabalho do lote começar apenas alguns minutos após o término do trabalho anterior, talvez seja mais econômico manter o cluster em execução entre os trabalhos. Isso depende da frequência de agendamento dos processos de pontuação, com alta frequência (a cada hora, por exemplo) ou com baixa frequência (uma vez por mês, por exemplo).

## <a name="deployment"></a>Implantação

Para implantar essa arquitetura de referência, execute as etapas descritas no [repositório do GitHub][github].

[acr]: /azure/container-registry/container-registry-intro
[ai]: /azure/application-insights/app-insights-overview
[aml-compute]: /azure/machine-learning/service/how-to-set-up-training-targets#amlcompute
[amls]: /azure/machine-learning/service/overview-what-is-azure-ml
[automatic-scaling]: /azure/batch/batch-automatic-scaling
[azure-files]: /azure/storage/files/storage-files-introduction
[cli]: /cli/azure
[create-resources]: https://github.com/Microsoft/AMLBatchScoringPipeline/blob/master/01_create_resources.ipynb
[deep]: /azure/architecture/reference-architectures/ai/batch-scoring-deep-learning
[event-hubs]: /azure/event-hubs/event-hubs-geo-dr
[explorer]: https://azure.microsoft.com/en-us/features/storage-explorer/
[github]: https://github.com/Microsoft/AMLBatchScoringPipeline
[one-class-svm]: http://scikit-learn.org/stable/modules/generated/sklearn.svm.OneClassSVM.html
[portal]: https://portal.azure.com
[ml-workspace]: /azure/machine-learning/studio/create-workspace
[python-script]: https://github.com/Azure/BatchAIAnomalyDetection/blob/master/batchai/predict.py
[pyscript]: https://github.com/Microsoft/AMLBatchScoringPipeline/blob/master/scripts/predict.py
[storage]: /azure/storage/blobs/storage-blobs-overview
[stream-analytics]: /azure/stream-analytics/
[sql-database]: /azure/sql-database/
[app-insights]: /azure/application-insights/app-insights-overview
