---
title: Pontuação em tempo real dos modelos do Machine Learning em R
description: Implemente um serviço de previsão em tempo real em R usando o Machine Learning Server em execução no Serviço de Kubernetes do Azure (AKS).
author: njray
ms.date: 12/12/18
ms.custom: azcat-ai
ms.openlocfilehash: a6069704c48fbc1f1a1e4b5df428011d6b5b883d
ms.sourcegitcommit: 62d2211badd1d6950e8cb819d70c9a4ab1ee01d9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/12/2018
ms.locfileid: "53318985"
---
# <a name="real-time-scoring-of-r-machine-learning-models"></a>Pontuação em tempo real dos modelos do Machine Learning em R

Esta arquitetura de referência mostra como implementar um serviço de previsão em tempo real (síncrono) em R usando o Microsoft Machine Learning Server em execução no Serviço de Kubernetes do Azure (AKS). Essa arquitetura pretende ser genérica e adequada para qualquer modelo de previsão compilado em R que você queria implantar como um serviço em tempo real. **[Implante essa solução][github]**.

## <a name="architecture"></a>Arquitetura

![Pontuação em tempo real dos modelos do Machine Learning em R no Azure][0]

Essa arquitetura de referência usa uma abordagem baseada em contêiner. Uma imagem do Docker é compilada contendo o R, bem como os vários artefatos necessários para pontuar novos dados. Isso inclui o próprio objeto de modelo e um script de pontuação. Essa imagem é enviada por push para um registro do Docker hospedado no Azure e, em seguida, implantado em um cluster Kubernetes, também no Azure.

A arquitetura desse fluxo de trabalho inclui os componentes a seguir.

- O **[Registro de Contêiner do Azure][acr]** é usado para armazenar as imagens para esse fluxo de trabalho. Os registros criados com o Registro de Contêiner podem ser gerenciados por meio da [API do Docker Registry V2][docker] padrão e o cliente.

- O **[Serviço de Kubernetes do Azure][aks]** é usado para hospedar a implantação e o serviço. Clusters criados com o AKS podem ser gerenciados usando a [API do Kubernetes][k-api] padrão e o cliente (kubectl).

- O **[Microsoft Machine Learning Server][mmls]** é usado para definir a API REST para o serviço e inclui [Operacionalização do Modelo][operationalization]. Esse processo de servidor Web orientado ao serviço escuta solicitações que, depois, são entregues a outros processos em segundo plano que executam o código R real para gerar os resultados. Todos esses processos são executados em um único nó nesta configuração, encapsulado em um contêiner. Para obter detalhes sobre como usar esse serviço fora de um ambiente de desenvolvimento ou teste, entre em contato com seu representante da Microsoft.

## <a name="performance-considerations"></a>Considerações sobre o desempenho

Cargas de trabalho de aprendizado de máquina tendem a usar muitos recursos de computação, tanto durante o treinamento quanto ao pontuar novos dados. Como regra geral, tente não executar mais de um processo de pontuação por núcleo. O Machine Learning Server permite definir o número de processos de R em execução em cada contêiner. O padrão é de cinco processos. Ao criar um modelo relativamente simples, como uma regressão linear com um pequeno número de variáveis, ou uma pequena árvore de decisões, você pode aumentar o número de processos. Monitore a carga da CPU em seus nós de cluster para determinar o limite apropriado do número de contêineres.

Um cluster habilitado para GPU pode acelerar alguns tipos de cargas de trabalho e, em particular, modelos de aprendizado avançado. Nem todas as cargas de trabalho podem aproveitar as GPUs &mdash;, somente aquelas que usam muita álgebra de matriz. Por exemplo, modelos baseados em árvore, incluindo florestas aleatórias e modelos de reforço, geralmente não obtêm vantagens com GPUs.

Alguns tipos de modelo, como as florestas aleatórias, são altamente paralelizáveis em CPUs. Nesses casos, acelere a pontuação de uma única solicitação distribuindo a carga de trabalho entre vários núcleos. No entanto, isso reduz sua capacidade para lidar com várias solicitações de pontuação, devido a um tamanho fixo de cluster.

Em geral, os modelos de R de software livre armazenam todos os seus dados na memória. Portanto, certifique-se de que seus nós tenham memória suficiente para acomodar os processos que você planeja executar simultaneamente. Se você estiver usando o Machine Learning Server para ajustar seus modelos, use as bibliotecas que podem processar dados em disco, em vez de lê-los na memória. Isso pode ajudar a reduzir consideravelmente os requisitos da memória. Independentemente de você usar o Machine Learning Server ou o R de software livre, monitore seus nós para garantir que seus processos de pontuação não são fiquem sem memória.

## <a name="security-considerations"></a>Considerações de segurança

### <a name="network-encryption"></a>Criptografia de rede

Nessa arquitetura de referência, o HTTPS será habilitado para a comunicação com o cluster, e um certificado de preparo de [Let’s Encrypt][encrypt] será usado. Para fins de produção, substitua seu próprio certificado por uma autoridade de assinatura apropriada.

### <a name="authentication-and-authorization"></a>Autenticação e autorização

A [Operacionalização de Modelo][operationalization] do Machine Learning Server exige a autenticação das solicitações de pontuação. Nessa implantação, usamos um nome de usuário e senha. Em uma configuração empresarial, você pode habilitar a autenticação usando o [Azure Active Directory][AAD] ou criar um front-end separado usando o [Gerenciamento de API do Azure][API].

Para que a Operacionalização de Modelo funcione corretamente com o Machine Learning Server em contêineres, você deve instalar um certificado JSON Web Token (JWT). Essa implantação usa um certificado fornecido pela Microsoft. Em um ambiente de produção, forneça o seu próprio.

Para o tráfego entre o AKS e o Registro de Contêiner, considere habilitar o [controle de acesso baseado em função][rbac] (RBAC) para limitar os privilégios de acesso apenas para quem precisa. 

### <a name="separate-storage"></a>Armazenamento separado

Essa arquitetura de referência agrupa o aplicativo (R) e os dados (objeto de modelo e script de pontuação) em uma única imagem. Em alguns casos, talvez seja melhor separá-los. Você pode colocar os dados e o código do modelo no [armazenamento][storage] de blobs ou arquivo do Azure, e recuperá-los na inicialização do contêiner. Nesse caso, verifique se a conta de armazenamento está definida para permitir apenas o acesso autenticado e exigir HTTPS.

## <a name="monitoring-and-logging-considerations"></a>Considerações de monitoramento e registro em log

Use o [painel do Kubernetes][dashboard] para monitorar o status geral do cluster do AKS. Consulte a folha de visão geral do cluster no portal do Azure para obter mais detalhes. Os recursos do [GitHub][github] também mostram como exibir o painel de R.

Embora o painel forneça uma exibição da integridade geral do seu cluster, também é importante acompanhar o status dos contêineres individuais. Para fazer isso, habilite o [Insights do Azure Monitor][monitor] na folha de visão geral do cluster no portal do Azure, ou confira [Azure Monitor para contêineres][monitor-containers] (em versão prévia).

## <a name="cost-considerations"></a>Considerações de custo

O Machine Learning Server é licenciado por núcleo, e todos os núcleos no cluster que executarão o Machine Learning Server contam para isso. Se você for um cliente empresarial do Machine Learning Server ou do Microsoft SQL Server, entre em contato com seu representante da Microsoft para obter detalhes sobre preços.

Uma alternativa de software livre ao Machine Learning Server é [Plumber][plumber], um pacote R que transforma seu código em uma API REST. O Plumber tem menos recursos do que o Machine Learning Server. Por exemplo, por padrão, ele não inclui todos os recursos que fornecem autenticação da solicitação. Se você usar Plumber, recomendamos que você habilite o [Gerenciamento de API do Azure][API] para lidar com detalhes de autenticação.

Além do licenciamento, a principal consideração de custo são os recursos de computação do cluster Kubernetes. O cluster deve ser grande o suficiente para lidar com o volume de solicitação esperado em horários de pico, mas essa abordagem deixa recursos ociosos em outros momentos. Para limitar o impacto dos recursos ociosos, habilite o [dimensionador automático horizontal][autoscaler] para o cluster usando a ferramenta kubectl. Ou, use o [dimensionador automático de cluster][cluster-autoscaler] do AKS.

## <a name="deploy-the-solution"></a>Implantar a solução

Há uma implantação de referência para essa arquitetura disponível no [GitHub][github]. Execute as etapas descritas para implantar um modelo de previsão simples como um serviço.

<!-- links -->
[AAD]: /azure/active-directory/fundamentals/active-directory-whatis
[API]: /azure/api-management/api-management-key-concepts
[ACR]: /azure/container-registry/container-registry-intro
[AKS]: /azure/aks/intro-kubernetes
[autoscaler]: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
[cluster-autoscaler]: /azure/aks/autoscaler
[monitor]: /azure/monitoring/monitoring-container-insights-overview
[dashboard]: /azure/aks/kubernetes-dashboard
[docker]: https://docs.docker.com/registry/spec/api/
[encrypt]: https://letsencrypt.org/
[gitHub]: https://github.com/Azure/RealtimeRDeployment
[K-API]: https://kubernetes.io/docs/reference/
[MMLS]: /machine-learning-server/what-is-machine-learning-server
[monitor-containers]: /azure/azure-monitor/insights/container-insights-overview
[operationalization]: /machine-learning-server/what-is-operationalization
[plumber]: https://www.rplumber.io
[RBAC]: /azure/role-based-access-control/overview
[storage]: /azure/storage/common/storage-introduction
[0]: ./_images/realtime-scoring-r.png
