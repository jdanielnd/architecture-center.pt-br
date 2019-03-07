---
title: Treinamento distribuído de modelos de aprendizado profundo no Azure
description: Essa arquitetura de referência mostra como realizar treinamento distribuído de modelos de aprendizado profundo em clusters de VMs habilitadas para GPU usando IA do Lote do Azure.
author: njray
ms.date: 01/14/19
ms.custom: azcat-ai
ms.openlocfilehash: 800defeb851f5a31dc730038c3699e1a3d54b923
ms.sourcegitcommit: d5ea427c25f9f7799cc859b99f328739ca2d8c1c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/15/2019
ms.locfileid: "54307737"
---
# <a name="distributed-training-of-deep-learning-models-on-azure"></a>Treinamento distribuído de modelos de aprendizado profundo no Azure

Essa arquitetura de referência mostra como realizar o treinamento distribuído de modelos de aprendizado profundo em clusters de VMs habilitadas para GPU. O cenário é a classificação de imagens, mas a solução pode ser adaptada para outros cenários de aprendizado profundo, como detecção e segmentação de objetos.

Há uma implantação de referência para essa arquitetura de referência disponível no [GitHub][github].

![Arquitetura para aprendizado profundo distribuído][0]

**Cenário**: A classificação de imagens é uma técnica amplamente aplicada na pesquisa visual computacional e é geralmente realizada com o treinamento de uma CNN (rede neural convolucional). Para modelos muito grandes com grandes conjuntos de dados, o processo de treinamento pode levar semanas ou meses em uma única GPU. Em algumas situações, os modelos são tão grandes que não é possível encaixar tamanhos de lote razoáveis na GPU. O uso do treinamento distribuído nessas situações pode reduzir o tempo de treinamento.

Neste cenário específico, um [modelo CNN ResNet50][resnet] é treinado usando [Horovod][horovod] no [conjunto de dados Imagenet][imagenet] e em dados sintéticos. A implementação de referência mostra como realizar essa tarefa usando três das mais populares estruturas de aprendizado profundo: TensorFlow, Keras e PyTorch.

Há várias maneiras de treinar um modelo de aprendizado profundo de forma distribuída, incluindo abordagens de dados em paralelo e modelo em paralelo baseadas em atualizações síncronas ou assíncronas. Atualmente, o cenário mais comum é formado por dados em paralelo com atualizações síncronas. Essa abordagem é mais fácil de implementar e é suficiente para a maioria dos casos de uso.

No treinamento distribuído de dados em paralelo com atualizações síncronas, o modelo é replicado em *n* dispositivos de hardware. Um minilote de exemplos de treinamento é dividido em *n* microlotes. Cada dispositivo executa as passagens para frente e para trás de um microlote. Quando um dispositivo conclui o processo, ele compartilha as atualizações com os demais dispositivos. Esses valores são usados para calcular os pesos atualizados de todo o minilote, e os pesos são sincronizados em todos os modelos. Esse cenário é abordado no repositório do [GitHub][github].

![Treinamento distribuído de dados em paralelo][1]

Essa arquitetura também pode ser usada em atualizações assíncronas ou de modelos em paralelo. No treinamento distribuído do modelo em paralelo, o modelo é dividido em *n* dispositivos de hardware, com cada dispositivo mantendo uma parte do modelo. Na implementação mais simples, cada dispositivo pode conter uma camada da rede e as informações são passadas entre os dispositivos durante a passagem para frente e para trás. Redes neurais maiores podem ser treinadas dessa maneira, mas à custa do desempenho, já que os dispositivos aguardam constantemente um pelo outro para concluir a passagem para frente ou para trás. Algumas técnicas avançadas tentam aliviar parcialmente esse problema usando gradientes sintéticos.

As etapas do treinamento são:

1. Crie scripts que serão executados no cluster e treine seu modelo. Em seguida, transfira-os para o armazenamento de arquivos.

1. Grave os dados no Armazenamento de Blobs.

1. Crie um servidor de arquivos de IA do Lote e baixe os dados do Armazenamento de Blobs para ele.

1. Crie os contêineres do Docker para cada framework de aprendizado profundo e transfira-os para um registro de contêiner (Hub do Docker).

1. Crie um pool de IA do Lote que também monte o servidor de arquivos de IA do Lote.

1. Enviar trabalhos. Cada um deles efetua pull da imagem e dos scripts apropriados do Docker.

1. Depois que o trabalho for concluído, grave todos os resultados no armazenamento de arquivos.

## <a name="architecture"></a>Arquitetura

Essa arquitetura é formada pelos componentes a seguir.

A **[IA do Lote do Azure][batch-ai]** desempenha o papel principal nesta arquitetura, escalando os recursos verticalmente ou reduzindo-os verticalmente, dependendo da necessidade. A IA do Lote é um serviço que ajuda a provisionar e gerenciar clusters de VMs, agendar trabalhos, coletar resultados, dimensionar recursos, lidar com falhas e criar o armazenamento apropriado. Ela oferece suporte a VMs habilitadas para GPU para cargas de trabalho de aprendizado profundo. Um SDK do Python e uma interface de linha de comando (CLI) estão disponíveis para a IA do Lote.

> [!NOTE]
> Os serviços de IA do Lote do Azure serão desativados em março de 2019 e seu treinamento em escala, assim como suas funcionalidades de pontuação, já estão disponíveis no [Serviço do Azure Machine Learning][amls]. Essa arquitetura de referência será atualizada em breve para uso de Machine Learning, que oferece um destino de computação gerenciada chamado [Computação do Machine Learning do Azure][aml-compute] para treinamento, implantação e pontuação de modelos de aprendizado de máquina.

O **[Armazenamento de Blobs][azure-blob]** é usado para preparar os dados. Esses dados são baixados para um servidor de arquivos da IA do Lote durante o treinamento.

Os **[Arquivos do Azure][files]** são usados para armazenar os scripts, os logs e os resultados finais do treinamento. O armazenamento de arquivos funciona bem para armazenar logs e scripts, mas não é tão eficiente quanto o Armazenamento de Blobs. Por isso, não deve ser usado para tarefas que exigem muitos dados.

O **[Servidor de arquivos de IA do Lote][batch-ai-files]** é um compartilhamento NFS de nó único usado nesta arquitetura para armazenar os dados de treinamento. A IA do Lote cria um compartilhamento NFS e monta-o no cluster. Os servidores de arquivos da IA do Lote são a maneira recomendada de fornecer dados ao cluster com a taxa de transferência necessária.

O **[Hub do Docker][docker]** é usado para armazenar a imagem do Docker que a IA do Lote usa para executar o treinamento. O Docker Hub foi escolhido para essa arquitetura por ser fácil de usar e ser o repositório de imagens padrão para usuários do Docker. O [Registro de Contêiner do Azure][acr] também pode ser usado.

## <a name="performance-considerations"></a>Considerações sobre o desempenho

O Azure fornece quatro [tipos de VM habilitados para GPU][gpu] adequados para o treinamento de modelos de aprendizado profundo. Eles variam em preço e velocidade, que podem ser baixos e altos, da seguinte forma:

| **Série de VMs do Azure** | **GPU NVIDIA** |
|---------------------|----------------|
| NC                  | K80            |
| ND                  | P40            |
| NCv2                | P100           |
| NCv3                | V100           |

Recomendamos dimensionar seu treinamento verticalmente antes de dimensioná-lo horizontalmente. Por exemplo, experimente um único V100 antes de experimentar um cluster de K80s.

O gráfico a seguir mostra as diferenças de desempenho para diferentes tipos de GPU baseados em [testes de parâmetro de comparação][benchmark] realizados usando o TensorFlow e Horovod na IA do Lote. O gráfico mostra a taxa de transferência de 32 clusters GPU em vários modelos, em diferentes tipos de GPU e versões de MPI. Foram implementados modelos no TensorFlow 1.9

![Resultados da taxa de transferência para modelos do TensorFlow em clusters GPU][2]

Cada série de VMs exibida na tabela anterior inclui uma configuração com o InfiniBand. Use as configurações do InfiniBand ao executar o treinamento distribuído e obtenha uma comunicação mais rápida entre os nós. O InfiniBand também aumenta a eficiência do dimensionamento do treinamento para os frameworks que podem aproveitá-lo. Para obter detalhes, confira o [parâmetro de comparação][benchmark] do Infiniband.

Embora a IA do Lote possa montar o armazenamento de Blobs usando o adaptador [blobfuse][blobfuse], não recomendamos o uso do Armazenamento de Blobs dessa maneira para treinamento distribuído, já que o desempenho não é bom o suficiente para tratar da taxa de transferência necessária. Em vez disso, mova os dados para um servidor de arquivos de IA do Lote, conforme mostrado no diagrama da arquitetura.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

A eficiência do dimensionamento do treinamento distribuído é sempre menor que 100%, devido à sobrecarga da rede &mdash; a sincronização do modelo inteiro entre os dispositivos forma um gargalo. Portanto, o treinamento distribuído é mais adequado para modelos grandes que não podem ser treinados usando um tamanho de lote razoável em uma única GPU ou para problemas que não podem ser resolvidos com a distribuição do modelo de maneira simples e paralela.

O treinamento distribuído não é recomendado para a execução de pesquisas de hiperparâmetro. A eficiência do dimensionamento afeta o desempenho e torna a abordagem distribuída menos eficiente do que o treinamento separado de várias configurações de modelo.

Uma maneira de aumentar a eficiência do dimensionamento é aumentar o tamanho do lote. No entanto, isso deve ser feito com cuidado, já que aumentar o tamanho do lote sem ajustar os demais parâmetros pode prejudicar o desempenho final do modelo.

## <a name="storage-considerations"></a>Considerações de armazenamento

Ao treinar modelos de aprendizado profundo, um aspecto frequentemente negligenciado é o local onde os dados são armazenados. Se o armazenamento for muito lento para acompanhar as demandas das GPUs, o desempenho do treinamento poderá diminuir.

A IA do Lote é compatível com muitas soluções de armazenamento. Essa arquitetura usa um servidor de arquivos da IA do Lote porque oferece a melhor relação entre facilidade de uso e desempenho. Para ter um desempenho melhor, carregue os dados localmente. No entanto, isso pode ser complicado porque todos os nós devem fazer o download dos dados do Armazenamento de Blobs e, com o conjunto de dados do ImageNet, isso pode levar horas. O [Armazenamento de Blobs do Azure Premium][blob] (visualização pública limitada) é outra boa opção a ser considerada.

Não monte o armazenamento de Blobs e de arquivos como repositórios de dados para o treinamento distribuído. Eles são muito lentos e atrapalham o desempenho do treinamento.

## <a name="security-considerations"></a>Considerações de segurança

### <a name="restrict-access-to-azure-blob-storage"></a>Restringir acesso ao Armazenamento de Blobs do Azure

Essa arquitetura usa [chaves de conta de armazenamento][security-guide] para acessar o armazenamento de Blobs. Para obter mais controle e proteção, considere o uso de uma SAS (Assinatura de Acesso Compartilhado). Isso concede acesso limitado a objetos no armazenamento, sem a necessidade de codificar as chaves da conta ou salvá-las em um texto não criptografado. Usar uma SAS também ajuda a garantir que a conta de armazenamento tenha uma governança adequada e que o acesso seja concedido somente às pessoas certas.

Para cenários com os dados mais confidenciais, verifique se todas as chaves de armazenamento estão protegidas, pois essas chaves concedem acesso completo a todos os dados de entrada e de saída da carga de trabalho.

### <a name="encrypt-data-at-rest-and-in-motion"></a>Criptografar dados em repouso e em movimento

Em cenários que usam dados confidenciais, criptografe os dados em repouso &mdash; ou seja, os dados no armazenamento. Sempre que os dados forem movidos de um local para o próximo, use SSL para proteger a transferência de dados. Para saber mais, confira o [Guia de segurança de Armazenamento do Azure][security-guide].

### <a name="secure-data-in-a-virtual-network"></a>Proteger dados em uma rede virtual

Em implantações de produção, considere implantar o cluster da IA do Lote em uma sub-rede de uma rede virtual que você especificar. Isso permite que os nós de computação no cluster se comuniquem com segurança com outras máquinas virtuais ou com uma rede local. Você também pode usar [pontos de extremidade de serviço][endpoints] com o armazenamento de blobs para conceder acesso de uma rede virtual ou usar um NFS de nó único dentro da rede virtual com a IA do Lote.

## <a name="monitoring-considerations"></a>Considerações de monitoramento

Durante a execução do seu trabalho, é importante monitorar o progresso e certificar-se de que as coisas estão funcionando conforme o esperado. No entanto, pode ser um desafio monitorar um cluster de nós ativos.

Os servidores de arquivos da IA do Lote podem ser gerenciados pelo portal do Azure ou pela [CLI do Azure][cli] e pelo SDK do Python. Para ter uma ideia do estado geral do cluster, navegue até a **IA do Lote** no portal do Azure para inspecionar o estado dos nós de cluster. Se um nó estiver inativo ou se um trabalho falhar, os logs de erro serão salvos no armazenamento de blobs e também poderão ser acessados no portal do Azure em **Trabalhos**.

É possível melhorar ainda mais o monitoramento conectando os logs ao [Azure Application Insights][ai] ou executando processos separados para sondar o estado do cluster da IA do Lote e seus trabalhos.

A IA do Lote registra automaticamente todos os stdout/stderr para a conta de armazenamento de blobs associada. O uso de uma ferramenta de navegação de armazenamento, como o [Gerenciador de Armazenamento do Azure][storage-explorer], fornecerá uma experiência muito mais fácil para navegação em arquivos de log.

Também é possível transmitir os logs para cada trabalho. Para obter detalhes sobre essa opção, confira as etapas de desenvolvimento no [GitHub][github].

## <a name="deployment"></a>Implantação

Há uma implantação de referência para essa arquitetura disponível no [GitHub][github]. Siga as etapas descritas para executar o treinamento distribuído de modelos de aprendizado profundo em clusters de VMs habilitadas para GPU.

## <a name="next-steps"></a>Próximas etapas

A saída dessa arquitetura é um modelo treinado salvo no armazenamento de blobs. É possível operacionalizar esse modelo para pontuação em tempo real ou pontuação do lote. Para obter mais informações, confira as seguintes arquiteturas de referência:

- [Pontuação em tempo real do Python Scikit-Learn e modelos de aprendizado profundo no Azure][real-time-scoring]
- [Pontuação em lote para modelos de aprendizado profundo do Azure][batch-scoring]

[0]: ./_images/distributed_dl_architecture.png
[1]: ./_images/distributed_dl_flow.png
[2]: ./_images/distributed_dl_tests.png
[acr]: /azure/container-registry/container-registry-intro
[ai]: /azure/application-insights/app-insights-overview
[aml-compute]: /azure/machine-learning/service/how-to-set-up-training-targets#amlcompute
[amls]: /azure/machine-learning/service/overview-what-is-azure-ml
[azure-blob]: /azure/storage/blobs/storage-blobs-introduction
[batch-ai]: /azure/batch-ai/overview
[batch-ai-files]: /azure/batch-ai/resource-concepts#file-server
[batch-scoring]: /azure/architecture/reference-architectures/ai/batch-scoring-deep-learning
[benchmark]: https://github.com/msalvaris/BatchAIHorovodBenchmark
[blob]: https://azure.microsoft.com/en-gb/blog/introducing-azure-premium-blob-storage-limited-public-preview/
[blobfuse]: https://github.com/Azure/azure-storage-fuse
[cli]: https://github.com/Azure/BatchAI/blob/master/documentation/using-azure-cli-20.md
[docker]: https://hub.docker.com/
[endpoints]: /azure/storage/common/storage-network-security?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network
[files]: /azure/storage/files/storage-files-introduction
[github]: https://github.com/Azure/DistributedDeepLearning/
[gpu]: /azure/virtual-machines/windows/sizes-gpu
[horovod]: https://github.com/uber/horovod
[imagenet]: http://www.image-net.org/
[real-time-scoring]: /azure/architecture/reference-architectures/ai/realtime-scoring-python
[resnet]: https://arxiv.org/abs/1512.03385
[security-guide]: /azure/storage/common/storage-security-guide
[storage-explorer]: /azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows
[tutorial]: https://github.com/Azure/DistributedDeepLearning