---
title: Pontuação em tempo real dos modelos do Python
titleSuffix: Azure Reference Architectures
description: Essa arquitetura de referência mostra como implantar modelos de Python como serviços Web no Azure para fazer previsões em tempo real.
author: msalvaris
ms.date: 11/09/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: 135e86b447684efd9f54340eda4b6bf6e4c35bbb
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487671"
---
# <a name="real-time-scoring-of-python-scikit-learn-and-deep-learning-models-on-azure"></a>Pontuação em tempo real de Python Scikit-Learn e modelos de aprendizado profundo (deep learning) no Azure

Essa arquitetura de referência mostra como implantar modelos de Python como serviços Web para fazer previsões em tempo real. Dois cenários são abordados: implantar modelos de Python normais e os requisitos específicos de implantação de modelos de aprendizado profundo. Ambos os cenários usam a arquitetura mostrada.

Duas implementações de referência para essa arquitetura estão disponíveis no GitHub, uma para [modelos normais de Python][github-python] e outro para [modelos de aprendizado profundo][github-dl].

![Diagrama da arquitetura para pontuação em tempo real dos modelos em Python no Azure](./_images/python-model-architecture.png)

## <a name="scenarios"></a>Cenários

As implementações de referência demonstram dois cenários usando essa arquitetura.

**Cenário 1: Correspondência de perguntas frequentes**. Este cenário mostra como implantar um modelo de correspondência de perguntas frequentes (FAQ) como um serviço Web para fornecer previsões para as perguntas do usuário. Para este cenário, os “Dados de entrada” no diagrama da arquitetura referem-se a cadeias de caracteres de texto contendo perguntas do usuário que devem ser associadas a uma lista de perguntas frequentes. Esse cenário foi desenvolvido para a biblioteca de Machine Learning [scikit-learn][scikit] para Python, mas pode ser generalizada para qualquer cenário que use modelos de Python para fazer previsões em tempo real.

Esse cenário usa um subconjunto dos dados de pergunta do Stack Overflow que inclui perguntas originais marcadas como JavaScript, suas perguntas duplicadas e suas respostas. Ele treina um pipeline de scikit-learn para prever a probabilidade de correspondência de uma pergunta duplicada com cada uma das perguntas originais. Essas previsões são feitas em tempo real usando um ponto de extremidade da API REST.

O fluxo de aplicativo para essa arquitetura é o seguinte:

1. O cliente envia uma solicitação HTTP POST com os dados codificados da pergunta.

2. O aplicativo Flask extrai a pergunta da solicitação.

3. A questão é enviada para o modelo de pipeline de scikit-learn para personalização e pontuação.

4. As perguntas mais frequentes correspondentes às suas pontuações são redirecionadas para um objeto JSON e retornadas ao cliente.

Aqui está uma captura de tela do aplicativo de exemplo que consome os resultados:

![Captura de tela do aplicativo de exemplo](./_images/python-faq-matches.png)

**Cenário 2: Classificação de imagens**. Este cenário mostra como implantar um modelo de Rede Neural Convolucional (CNN) como um serviço Web para fornecer previsões em imagens. Para este cenário, os “Dados de entrada” no diagrama da arquitetura referem-se aos arquivos de imagem. CNNs são muito eficazes em pesquisa visual computacional para tarefas como classificação de imagem e detecção de objetos. Este cenário foi desenvolvido para as estruturas TensorFlow, Keras (com back-end do TensorFlow) e PyTorch. No entanto, ele pode ser generalizado para qualquer cenário que use modelos de aprendizado profundo para fazer previsões em tempo real.

Esse cenário usa um modelo ResNet-152 pré-treinado no conjunto de dados ImageNet-1K (1.000 classes) para prever a qual categoria (veja a figura abaixo) uma imagem pertence. Essas previsões são feitas em tempo real usando um ponto de extremidade da API REST.

![Exemplo de previsões](./_images/python-example-predictions.png)

O fluxo do aplicativo para o modelo de aprendizado profundo é o seguinte:

1. O cliente envia uma solicitação HTTP POST com os dados de imagem codificados.

2. O aplicativo Flask extrai a imagem da solicitação.

3. A imagem é pré-processada e enviada para o modelo de pontuação.

4. O resultado da pontuação é redirecionado para um objeto JSON e retornado ao cliente.

## <a name="architecture"></a>Arquitetura

Essa arquitetura é formada pelos componentes a seguir.

**[Máquina virtual][vm]** (VM). A VM é mostrada como um exemplo de um dispositivo &mdash; local ou na nuvem &mdash; que pode enviar uma solicitação HTTP.

O **[Serviço de Kubernetes do Azure][aks]** (AKS) é usado para implantar o aplicativo em um cluster Kubernetes. O AKS simplifica a implantação e as operações do Kubernetes. O cluster pode ser configurado usando VMs somente de CPU para modelos de Python normais ou VMs habilitadas para GPU para modelos de aprendizado profundo.

**[Balanceador de carga][lb]**. Um balanceador de carga provisionado pelo AKS é usado para expor o serviço externamente. O tráfego do balanceador de carga é direcionado para os pods de back-end.

O **[Docker Hub][docker]** é usado para armazenar a imagem do Docker que é implantada no cluster Kubernetes. O Docker Hub foi escolhido para essa arquitetura por ser fácil de usar e ser o repositório de imagens padrão para usuários do Docker. O [Registro de Contêiner do Azure][acr] também pode ser usado para essa arquitetura.

## <a name="performance-considerations"></a>Considerações sobre o desempenho

Para arquiteturas de pontuação em tempo real, o desempenho de taxa de transferência se torna uma consideração dominante. Para modelos de Python normais, é geralmente aceito que CPUs são suficientes para lidar com a carga de trabalho.

No entanto, para cargas de trabalho de aprendizado profundo, quando a velocidade é um gargalo, GPUs geralmente fornecem melhor [desempenho][gpus-vs-cpus] em comparação com CPUs. Para corresponder ao desempenho de GPU usando CPUs, geralmente é necessário um cluster com um grande número de CPUs.

Você pode usar as CPUs para essa arquitetura em qualquer cenário, mas para modelos de aprendizado profundo, as GPUs fornecem valores de taxa de transferência significativamente superiores em comparação com um cluster de CPU de custo semelhante. O AKS oferece suporte ao uso de GPUs, que é uma das vantagens de usar AKS para essa arquitetura. Além disso, as implantações de aprendizado profundo geralmente usam modelos com um grande número de parâmetros. Usar GPUs impede a contenção de recursos entre o modelo e o serviço Web, que é um problema em implantações somente de CPU.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Para modelos de Python normais, em que o cluster do AKS é provisionado com VMs de CPU, tome cuidado quando [aumentar o número de pods][manually-scale-pods]. O objetivo é utilizar totalmente o cluster. O dimensionamento depende das solicitações de CPU e dos limites definidos para os pods. O Kubernetes dá suporte ao [dimensionamento automático][autoscale-pods] de pods para ajustar o número de pods em uma implantação, dependendo da utilização da CPU ou de outras métricas selecionadas. O [dimensionador automático de cluster][autoscaler] (em versão prévia) faz isso colocando em escala seus nós de agente com base em pods pendentes.

Para cenários de aprendizado profundo, usando as máquinas virtuais habilitadas para GPU, os limites de recursos nos pods fazem com que uma GPU seja atribuída a um pod. Dependendo do tipo de VM usado, você deve [dimensionar os nós do cluster][scale-cluster] para atender à demanda para o serviço. Você pode fazer isso facilmente usando a CLI do Azure e o kubectl.

## <a name="monitoring-and-logging-considerations"></a>Considerações de monitoramento e registro em log

### <a name="aks-monitoring"></a>Monitoramento do AKS

Para visibilidade sobre o desempenho do AKS, use o recurso de [Azure Monitor para contêineres][monitor-containers]. Ele coleta métricas de processador e memória de controladores, nós e contêineres disponíveis no Kubernetes por meio da API Métricas.

Ao implantar seu aplicativo, monitore o cluster do AKS para verificar se ele está funcionando conforme o esperado, se todos os nós estão operacionais e se todos os pods estão em execução. Embora você possa usar a ferramenta de linha de comando [kubectl][kubectl] para recuperar o status de pod, o Kubernetes também inclui um painel Web para o monitoramento básico do status do cluster e gerenciamento.

![Captura de tela do painel do Kubernetes](./_images/python-kubernetes-dashboard.png)

Para ver o estado geral do cluster e dos nós, vá até a seção **Nós** no painel do Kubernetes. Se um nó está inativo ou falhou, você pode exibir os logs de erros nessa página. Da mesma forma, vá até as seções **Pods** e **Implantações** para obter informações sobre o número de pods e o status da implantação.

### <a name="aks-logs"></a>Logs do AKS

O AKS registra automaticamente todos os stdout/stderr para os logs dos pods no cluster. Use kubectl para vê-los e ver também os eventos de nível de nó e logs. Para obter detalhes, confira as etapas de implantação.

Use o [Azure Monitor para contêineres][monitor-containers] para coletar métricas e logs por meio de uma versão em contêineres do agente do Log Analytics para o Linux, que é armazenado em seu workspace do Log Analytics.

## <a name="security-considerations"></a>Considerações de segurança

Use a [Central de Segurança do Azure][security-center] para obter uma exibição central do estado da segurança de seus recursos do Azure. A Central de Segurança monitora problemas de segurança potenciais e fornece uma visão abrangente da integridade de segurança de sua implantação, mas ela não monitora os nós do agente do AKS. A Central de Segurança é configurada por assinatura do Azure. Habilite a coleta de dados de segurança conforme descrito em [Integrar a assinatura do Azure à Central de Segurança Standard][get-started]. Depois que a coleta de dados for habilitada, a Central de Segurança examinará automaticamente todas as VMs criadas nessa assinatura.

**Operações**. Para se conectar a um cluster AKS usando o token de autenticação do Azure Active Directory (Azure AD), configure o AKS para usar o Azure AD para [autenticação do usuário][aad-auth]. Os administradores de cluster também podem configurar o controle de acesso baseado em função (RBAC) do Kubernetes com base na identidade de um usuário ou associação no grupo de diretórios.

Use o [RBAC][rbac] para controlar o acesso aos recursos do Azure que você implanta. O RBAC permite atribuir funções de autorização aos membros de sua equipe de DevOps. Um usuário pode ser atribuído a várias funções, e você pode criar funções personalizadas para [permissões] ainda mais refinadas.

**HTTPS**. Como prática recomendada de segurança, o aplicativo deve impor HTTPS e redirecionar as solicitações HTTP. Use um [controlador de entrada][ingress-controller] para implantar um proxy reverso que termina o SSL e redireciona as solicitações HTTP. Para obter mais informações, confira [Criar um controlador de ingresso HTTPS no Serviço de Kubernetes do Azure (AKS)][https-ingress].

**Autenticação**. Essa solução não restringe o acesso aos pontos de extremidade. Para implantar a arquitetura em uma configuração empresarial, proteja os pontos de extremidade por meio de chaves de API e adicione alguma forma de autenticação do usuário para o aplicativo cliente.

**Registro de contêiner**. Essa solução usa um registro público para armazenar a imagem do Docker. O código do qual o aplicativo depende, bem como o modelo, estão contidos dentro dessa imagem. Aplicativos empresariais devem usar um registro privado para ajudar na proteção contra a execução de código mal-intencionado e para ajudar a impedir que as informações dentro do contêiner sejam comprometidas.

**Proteção contra DDoS**. Considere habilitar a [Proteção contra DDoS Standard][ddos]. Embora a proteção contra DDoS básica seja habilitada como parte da plataforma Azure, a Proteção contra DDoS Standard fornece funcionalidades de mitigação ajustadas especificamente para os recursos da rede virtual do Azure.

**Registro em log**. Use as práticas recomendadas antes de armazenar dados de log, como remover as senhas de usuário e outras informações que podem ser usadas para cometer fraudes de segurança.

## <a name="deployment"></a>Implantação

Para implantar essa arquitetura de referência, execute as etapas descritas no repositório do GitHub:

- [Modelos de Python normais][github-python]
- [Modelos de aprendizado profundo][github-dl]

<!-- links -->

[aad-auth]: /azure/aks/aad-integration
[acr]: /azure/container-registry/
[something]: https://kubernetes.io/docs/reference/access-authn-authz/authentication/
[aks]: /azure/aks/intro-kubernetes
[autoscaler]: /azure/aks/autoscaler
[autoscale-pods]: /azure/aks/tutorial-kubernetes-scale#autoscale-pods
[azcopy]: /azure/storage/common/storage-use-azcopy-linux
[ddos]: /azure/virtual-network/ddos-protection-overview
[docker]: https://hub.docker.com/
[get-started]: /azure/security-center/security-center-get-started
[github-python]: https://github.com/Azure/MLAKSDeployment
[github-dl]: https://github.com/Microsoft/AKSDeploymentTutorial
[gpus-vs-cpus]: https://azure.microsoft.com/en-us/blog/gpus-vs-cpus-for-deployment-of-deep-learning-models/
[https-ingress]: /azure/aks/ingress-tls
[ingress-controller]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[lb]: /azure/load-balancer/load-balancer-overview
[manually-scale-pods]: /azure/aks/tutorial-kubernetes-scale#manually-scale-pods
[monitor-containers]: /azure/monitoring/monitoring-container-insights-overview
[permissões]: /azure/aks/concepts-identity
[rbac]: /azure/active-directory/role-based-access-control-what-is
[scale-cluster]: /azure/aks/scale-cluster
[scikit]: https://pypi.org/project/scikit-learn/
[security-center]: /azure/security-center/security-center-intro
[vm]: /azure/virtual-machines/
