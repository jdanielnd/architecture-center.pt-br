---
title: Pontuação em lote para modelos de aprendizado profundo
titleSuffix: Azure Reference Architectures
description: Essa arquitetura de referência mostra como aplicar a transferência de estilo neural a um vídeo usando a IA do Lote do Azure.
author: jiata
ms.date: 10/02/2018
ms.custom: azcat-ai
ms.openlocfilehash: 0396903a39d00a4131df65872a63f4b3fde8dce7
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/08/2018
ms.locfileid: "53119873"
---
# <a name="batch-scoring-on-azure-for-deep-learning-models"></a>Pontuação em lote para modelos de aprendizado profundo do Azure

Essa arquitetura de referência mostra como aplicar a transferência de estilo neural a um vídeo usando a IA do Lote do Azure. A *Transferência de estilo* é uma técnica de aprendizado profundo que compõe uma imagem existente no estilo de outra imagem. Essa arquitetura pode ser generalizada para qualquer cenário que usa a pontuação do lote com aprendizado profundo. [**Implantar esta solução**](#deploy-the-solution).

![Diagrama da arquitetura para modelos de aprendizado profundo usando a IA do Lote do Azure](./_images/batch-ai-deep-learning.png)

**Cenário**: uma organização de mídia tem um vídeo cujo estilo ela quer alterar para procurar uma pintura específica. A organização quer ser capaz de aplicar esse estilo a todos os quadros do vídeo em tempo hábil e de uma forma automatizada. Para saber mais sobre os algoritmos de transferência de estilo neural, consulte [Transferência de estilo de imagem usando redes neurais convolucionais neurais][image-style-transfer] (PDF).

| Imagem do estilo: | Vídeo de entrada/conteúdo: | Vídeo de saída: |
|--------|--------|---------|
| <img src="https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/style_image.jpg" width="300"> | [<img src="https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/input_video_image_0.jpg" width="300" height="300">](https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/input_video.mp4 "Vídeo de entrada") *clique para ver o vídeo* | [<img src="https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/output_video_image_0.jpg" width="300" height="300">](https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/output_video.mp4 "Vídeo de saída") *clique para ver o vídeo* |

Essa arquitetura de referência foi projetada para cargas de trabalho que são disparadas pela presença da nova mídia no Armazenamento do Azure. O processamento envolve as seguintes etapas:

1. Carregue uma imagem de estilo específica (como uma pintura do Van Gogh) e um script de transferência de estilo no Armazenamento de Blobs.
1. Crie um cluster de dimensionamento automático da IA do Lote que esteja pronto para começar a fazer o trabalho.
1. Divida o arquivo de vídeo em quadros individuais e carregue-os no Armazenamento de Blobs.
1. Após o upload de todos os quadros, carregue um arquivo de gatilho no Armazenamento de Blobs.
1. Esse arquivo dispara um Aplicativo Lógico que cria um contêiner em execução em Instâncias de Contêiner do Azure.
1. O contêiner executa um script que cria os trabalhos de IA do Lote. Cada trabalho aplica a transferência de estilo neural em paralelo em todos os nós do cluster da IA do Lote.
1. Após a geração das imagens, elas são salvas no Armazenamento de Blobs.
1. Faça o download dos quadros gerados e insira as imagens de volta em um vídeo.

## <a name="architecture"></a>Arquitetura

Essa arquitetura é formada pelos componentes a seguir.

### <a name="compute"></a>Computação

**[IA do Lote do Azure][batch-ai]** é usada para executar o algoritmo de transferência de estilo neural. A IA do Lote dá suporte a cargas de trabalho de aprendizado profundo fornecendo ambientes em contêineres que são pré-configurados para estruturas de aprendizado profundo, em VMs habilitadas para GPU. A IA do Lote também pode conectar o cluster de computação ao Armazenamento de Blobs.

### <a name="storage"></a>Armazenamento

**[Armazenamento de Blobs][blob-storage]** é usado para armazenar todas as imagens (imagens de entrada, imagens de estilo e imagens de saída) bem como todos os logs gerados na IA do Lote. O Armazenamento de Blobs se integra à IA do Lote por meio do [blobfuse][blobfuse], um sistema de arquivos virtual de software livre que é apoiado pelo Armazenamento de Blobs. O Armazenamento de Blobs também é muito econômico para o desempenho exigido por essa carga de trabalho.

### <a name="trigger--scheduling"></a>Gatilho/agendamento

**[Aplicativo Lógico do Azure][logic-apps]** é usado para disparar o fluxo de trabalho. Quando o Aplicativo Lógico detecta que um blob foi adicionado ao contêiner, ele dispara o processo da IA do Lote. O Aplicativo Lógico é ótimo para esta arquitetura de referência porque é uma maneira fácil de detectar alterações no armazenamento de blobs e fornece um processo fácil para alterar o gatilho.

**[As Instâncias de Contêiner do Azure][container-instances]** é usado para executar os scripts de Python que criam os trabalhos da IA do Lote. A execução desses scripts dentro de um contêiner do Docker é uma maneira conveniente de executá-los sob demanda. Para essa arquitetura, usamos as Instâncias de Contêiner porque não há um conector de Aplicativo Lógico pré-criados para ela, o que permite que o Aplicativo Lógico dispare o trabalho da IA do Lote. As Instâncias de Contêiner podem ativar rapidamente os processos sem estado.

**[DockerHub][dockerhub]** é usado para armazenar a imagem do Docker usada pelas Instâncias de Contêiner para executar o processo de criação do trabalho. O DockerHub foi escolhido para essa arquitetura por ser fácil de usar e ser o repositório de imagens padrão para usuários do Docker. O [Registro de Contêiner do Azure][container-registry] também pode ser usado.

### <a name="data-preparation"></a>Preparação dos dados

Essa arquitetura de referência usa filmagens de um orangotango em uma árvore. Baixe a filmagem [aqui][source-video] e processe-a no fluxo de trabalho executando estas etapas:

1. Use o [AzCopy][azcopy] para baixar o vídeo do blob público.
2. Use [FFmpeg][ffmpeg] para extrair o arquivo de áudio, para que possa ser inseri-lo de volta posteriormente no vídeo de saída.
3. Use o FFmpeg para dividir o vídeo em quadros individuais. Os quadros serão processados de forma independente, em paralelo.
4. Use o AzCopy para copiar os quadros individuais em seu contêiner de blob.

Nesse estágio, a filmagem está em um formato que pode ser usado para transferência de estilo neural.

## <a name="performance-considerations"></a>Considerações sobre o desempenho

### <a name="gpu-vs-cpu"></a>GPU versus CPU

Para cargas de trabalho de aprendizado profundo, as GPUs geralmente superam bastante as CPUs, chegando até a exigir um cluster considerável de CPUs para obter um desempenho comparável. Embora seja uma opção usar apenas CPUs nessa arquitetura, as GPUs fornecerão um perfil de custo/desempenho muito melhor. Recomendamos o uso de VMs otimizadas para GPU mais recentes [série NCv3]vm-sizes-gpu.

As GPUs não estão habilitadas por padrão em todas as regiões. Selecione uma região com GPUs habilitadas. Além disso, as assinaturas têm uma cota padrão de zero núcleos para VMs otimizadas para GPU. Eleve essa cota abrindo uma solicitação de suporte. Verifique se a sua assinatura tem cota suficiente para executar sua carga de trabalho.

### <a name="parallelizing-across-vms-vs-cores"></a>Paralelização em VMs versus núcleos

Ao executar um processo de transferência de estilo como um trabalho em lotes, será necessário paralelizar entre VMs os trabalhos executadas principalmente em GPUs. Há duas abordagens possíveis: você pode criar um cluster maior usando VMs que têm uma única GPU, ou criar um cluster menor usando VMs com várias GPUs.

Para essa carga de trabalho, essas duas opções terão um desempenho comparável. Usar menos VMs com mais GPUs por VM pode ajudar a reduzir a movimentação de dados. No entanto, o volume de dados por trabalho para essa carga de trabalho não é muito grande, portanto, você não observará muita limitação por Armazenamento de Blobs.

### <a name="images-batch-size-per-batch-ai-job"></a>Tamanho do lote de imagens por trabalho de IA do Lote

Outro parâmetro deve ser configurado é o número de imagens a serem processadas por um trabalho da IA do Lote. Por um lado, convém garantir que o trabalho seja distribuído amplamente pelos nós e, se um trabalho falhar, não será necessário repetir muitas imagens. Isso indica ter vários trabalhos da IA do Lote e, portanto, menos imagens para processar por trabalho. Por outro lado, se poucas imagens forem processadas por trabalho, o tempo de configuração/inicialização ficará aumentará desproporcionalmente. Defina o número de trabalhos igual ao número máximo de nós no cluster. Essa opção proporcionará o melhor desempenho, supondo que nenhum trabalho falhe, pois minimiza o custo de configuração/inicialização. No entanto, se um trabalho falhar, talvez seja necessário reprocessar uma grande quantidade de imagens.

### <a name="file-servers"></a>Servidores de arquivos

Ao usar a IA do Lote, você pode escolher várias opções de armazenamento, dependendo da taxa de transferência necessária para o seu cenário. Para cargas de trabalho que precisam de uma taxa de transferência baixa, usar o Armazenamento de Blobs (por meio do blobfuse) deve ser suficiente. Como alternativa, a IA do Lote também dá suporte a um Servidor de Arquivos da IA do Lote, um NFS gerenciado de nó único, que pode ser montado automaticamente em nós de cluster para fornecer um local de armazenamento de acesso centralizado para os trabalhos. Na maioria dos casos, somente um servidor de arquivos é necessário em um espaço de trabalho e você pode separar os dados de seus trabalhos de treinamento em diretórios diferentes. Se o NFS de nó único não for adequado para suas cargas de trabalho, a IA do Lote dá suporte a outras opções de armazenamento, incluindo o Arquivos do Azure ou soluções personalizadas como um sistema de arquivos Gluster ou Lustre.

## <a name="security-considerations"></a>Considerações de segurança

### <a name="restricting-access-to-azure-blob-storage"></a>Restrição do acesso ao Armazenamento de Blobs do Azure

Nessa arquitetura de referência, o Armazenamento de Blobs do Azure é o componente de armazenamento principal que precisa ser protegido. A implantação de linha de base mostrada no repositório do GitHub usa chaves da conta de armazenamento para acessar o Armazenamento de Blobs. Para obter mais controle e proteção, considere o uso de uma SAS (Assinatura de Acesso Compartilhado). Isso concede acesso limitado a objetos no armazenamento, sem a necessidade de codificar as chaves da conta ou salvá-las em um texto não criptografado. Essa abordagem é especialmente útil porque as chaves da conta são visíveis em texto não criptografado dentro da interface do designer do Aplicativo Lógico. Usar uma SAS também ajuda a garantir que a conta de armazenamento tenha uma governança adequada, e que o acesso seja concedido somente para as pessoas certas.

Para cenários com os dados mais confidenciais, verifique se todas as chaves de armazenamento estão protegidas, pois essas chaves concedem acesso completo a todos os dados de entrada e de saída da carga de trabalho.

### <a name="data-encryption-and-data-movement"></a>Criptografia de dados e movimentação de dados

Essa arquitetura de referência usa transferência de estilo como um exemplo de um processo de pontuação do lote. Para cenários com dados mais confidenciais, os dados no armazenamento devem ser criptografados em repouso. Sempre que os dados são movidos de um local para o próximo, use SSL para proteger a transferência de dados. Para saber mais, confira o [Guia de segurança de Armazenamento do Azure][storage-security].

### <a name="securing-data-in-a-virtual-network"></a>Proteção dos dados em uma rede virtual

Ao implantar o cluster da IA do Lote, você pode configurar seu cluster para provisionamento dentro de uma sub-rede de uma rede virtual. Isso permite que os nós de computação no cluster se comuniquem com segurança com outras máquinas virtuais, ou até mesmo com uma rede local. Você também pode usar [pontos de extremidade de serviço][service-endpoints] com o armazenamento de blobs para conceder acesso de uma rede virtual ou usar um NFS de nó único dentro da VNET com a IA do Lote para garantir que os dados estejam sempre protegidos.

### <a name="protecting-against-malicious-activity"></a>Proteção contra atividades mal-intencionadas

Em cenários com vários usuários, proteja os dados confidenciais contra atividades mal-intencionadas. Se outros usuários receberem acesso a essa implantação para personalizar os dados de entrada, observe as seguintes precauções e considerações:

- Use o RBAC para limitar o acesso de usuários somente aos recursos necessários.
- Provisione duas contas de armazenamento separadas. Armazene dados de entrada e saída na primeira conta. Os usuários externos podem obter acesso a essa conta. Armazene scripts executáveis e arquivos de log de saída na outra conta. Os usuários externos não devem ter acesso a essa conta. Isso garantirá que os usuários externos não possam modificar os arquivos executáveis (para injetar código mal-intencionado) e não tenham acesso a arquivos de log, que podem conter informações confidenciais.
- Os usuários mal-intencionados podem executar DDOS na fila de trabalho ou injetar mensagens suspeitas malformadas na fila de trabalho, fazendo com que o sistema trave ou causando erros de remoção da fila.

## <a name="monitoring-and-logging"></a>Monitoramento e registro em log

### <a name="monitoring-batch-ai-jobs"></a>Monitoramento de trabalhos da IA do Lote

Durante a execução do seu trabalho, é importante monitorar o progresso e certificar-se de que as coisas estão funcionando conforme o esperado. No entanto, pode ser um desafio monitorar um cluster de nós ativos.

Para ter uma ideia do estado geral do cluster, vá até a folha de IA do Lote do Portal do Azure para inspecionar o estado dos nós no cluster. Se um nó estiver inativo ou se um trabalho falhar, os logs de erro serão salvos no armazenamento de blobs e também poderão ser acessados na folha Trabalhos no portal do Azure.

É possível melhorar ainda mais o monitoramento conectando os logs ao Application Insights ou executando processos separados para sondar o estado do cluster de IA do Lote e seus trabalhos.

### <a name="logging-in-batch-ai"></a>Registro em log na IA do Lote

A IA do Lote registrará automaticamente todos os stdout/stderr para a conta de armazenamento de blob associada. O uso de uma ferramenta de navegação de armazenamento, como o Gerenciador de Armazenamento, fornecerá uma experiência muito mais fácil para navegação em arquivos de log.

As etapas de implantação para essa arquitetura de referência também mostra como configurar um sistema mais simples de registro em log, de modo que todos os logs em diferentes trabalhos são salvos no mesmo diretório em seu contêiner de blob, conforme mostrado abaixo. Use esses logs para monitorar quanto tempo demora para processar cada imagem e cada trabalho. Isso lhe dará uma ideia melhor de como otimizar ainda mais o processo.

![Captura de tela de registro em log para a IA do Lote do Azure](./_images/batch-ai-logging.png)

## <a name="cost-considerations"></a>Considerações de custo

Em comparação com os componentes de armazenamento e agendamento, os recursos de computação usados nesta arquitetura de referência dominam em termos de custos. Um dos principais desafios é paralelizar com eficiência o trabalho em um cluster de computadores habilitados para GPU.

O tamanho do cluster da IA do Lote pode aumentar e diminuir automaticamente dependendo dos trabalhos na fila. Você pode habilitar o dimensionamento automático com a IA do Lote escolhendo entre duas maneiras. Você pode isso programaticamente, que pode ser configurado no arquivo `.env` que faz parte das [etapas de implantação][deployment], ou você pode alterar a fórmula de dimensionamento diretamente no portal após a criação do cluster.

Para o trabalho que não exige processamento imediato, configure a fórmula de dimensionamento automático, para que o estado padrão (mínimo) seja um cluster sem nós. Com essa configuração, o cluster começa com zero nós e só pode ser escalado verticalmente quando detecta os trabalhos na fila. Se o processo de pontuação do lote acontecer apenas algumas vezes por dia, essa configuração permitirá uma economia considerável.

Talvez o dimensionamento automático não seja apropriado para trabalhos em lote que aconteçam muito próximos uns dos outros. O tempo de ativação e desativação de um cluster também incorre em um custo, portanto, se uma carga de trabalho do lote começar apenas alguns minutos após o término do trabalho anterior, talvez seja mais econômico manter o cluster em execução entre os trabalhos.

## <a name="deploy-the-solution"></a>Implantar a solução

Para implantar essa arquitetura de referência, execute as etapas descritas no [repositório do GitHub][deployment].

<!-- links -->

[azcopy]: /azure/storage/common/storage-use-azcopy-linux
[batch-ai]: /azure/batch-ai/
[blobfuse]: https://github.com/Azure/azure-storage-fuse
[blob-storage]: /azure/storage/blobs/storage-blobs-introduction
[container-instances]: /azure/container-instances/
[container-registry]: /azure/container-registry/
[deployment]: https://github.com/Azure/batch-scoring-for-dl-models
[dockerhub]: https://hub.docker.com/
[ffmpeg]: https://www.ffmpeg.org/
[image-style-transfer]: https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Gatys_Image_Style_Transfer_CVPR_2016_paper.pdf
[logic-apps]: /azure/logic-apps/
[service-endpoints]: /azure/storage/common/storage-network-security?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network
[source-video]: https://happypathspublic.blob.core.windows.net/videos/orangutan.mp4
[storage-security]: /azure/storage/common/storage-security-guide
[vm-sizes-gpu]: /azure/virtual-machines/windows/sizes-gpu