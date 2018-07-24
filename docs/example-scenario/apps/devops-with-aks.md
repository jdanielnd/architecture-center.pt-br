---
title: Pipeline de CI/CD para cargas de trabalho baseados em contêiner
description: Cenário comprovado para a criação de um pipeline DevOps para um aplicativo Web Node.js que usa Jenkins, Registro de Contêiner do Azure, Serviço de Kubernetes do Azure, Cosmos DB e Grafana.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: d9f6571234a0c3e67a233cfda1a37f6fb32929a3
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060754"
---
# <a name="cicd-pipeline-for-container-based-workloads"></a>Pipeline de CI/CD para cargas de trabalho baseados em contêiner

Este cenário de exemplo é aplicável a empresas que desejam modernizar o desenvolvimento de aplicativos usando contêineres e fluxos de trabalho de DevOps. Neste cenário, um aplicativo Web do Node.js é criado e implantado pelo Jenkins em um Registro de Contêiner do Azure e no Serviço de Kubernetes do Azure. O Azure Cosmos DB é usado para oferecer uma camada de banco de dados distribuído globalmente. Para monitorar e solucionar problemas de desempenho do aplicativo, o Azure Monitor integra-se a uma instância e um painel do Grafana.

Entre os cenários de uso de exemplo estão o fornecimento de um ambiente de desenvolvimento automatizado, a validação de novas confirmações de código e o envio por push de novas implantações em ambientes de preparo ou produção. Tradicionalmente, as empresas tinham que criar e compilar aplicativos e atualizações e manter uma base de código grande e monolítica. Com uma abordagem moderna para o desenvolvimento de aplicativos que usa CI (integração contínua) e CD (implantação contínua), você pode criar, testar e implantar serviços mais rapidamente. Essa abordagem moderna permite liberar aplicativos e atualizações para seus clientes mais rapidamente e responder às mudanças nas demandas dos negócios de forma mais ágil.

Usando serviços do Azure, como o Serviço de Kubernetes do Azure, o Registro de Contêiner e o Cosmos DB, as empresas podem usar o que há de mais recente em ferramentas e técnicas de desenvolvimento de aplicativos para simplificar o processo de implementação de alta disponibilidade.

## <a name="related-use-cases"></a>Casos de uso relacionados

Considere este cenário para os casos de uso a seguir:

* Modernizar práticas de desenvolvimento de aplicativos para uma abordagem de microsserviços baseada em contêiner.
* Acelerar o desenvolvimento de aplicativos e ciclos de vida de implantação.
* Automatizar implantações de teste ou ambientes de aceitação para validação.

## <a name="architecture"></a>Arquitetura

![Visão geral de arquitetura dos componentes do Azure envolvidos em um cenário de DevOps usando o Jenkins, o Registro de Contêiner do Azure e o Serviço de Kubernetes do Azure][architecture]

Este cenário trata de um pipeline DevOps para um aplicativo Web do Node.js e um back-end de banco de dados. O fluxo de dados neste cenário ocorre da seguinte forma:

1. Um desenvolvedor faz alterações no código-fonte do aplicativo Web Node.js.
2. A alteração de código é confirmada em um repositório de controle do código-fonte, como o GitHub.
3. Para iniciar o processo de CI (integração contínua), um webhook do GitHub dispara uma compilação de projeto Jenkins.
4. O trabalho de compilação do Jenkins usa um agente de build dinâmico no Serviço de Kubernetes do Azure para executar um processo de compilação do contêiner.
5. Uma imagem de contêiner é criada a partir do código no controle do código-fonte e, em seguida, é enviada por push para um Registro de Contêiner do Azure.
6. Por meio da CD (implantação contínua), o Jenkins implanta essa imagem de contêiner atualizada no cluster Kubernetes.
7. O aplicativo Web Node.js usa o Azure Cosmos DB como seu back-end. Tanto o Cosmos DB quanto o Serviço de Kubernetes do Azure enviam métricas para o Azure Monitor.
8. Uma instância do Grafana fornece painéis visuais sobre o desempenho do aplicativo com base nos dados do Azure Monitor.

### <a name="components"></a>Componentes

* O [Jenkins] [ jenkins] é um servidor de automação de software livre que pode se integrar aos serviços do Azure para habilitar a CI (integração contínua) e a CD (implantação contínua). Neste cenário, o Jenkins orquestra a criação de novas imagens de contêiner com base nas confirmações no controle do código-fonte, envia por push essas imagens para o Registro de Contêiner do Azure e atualiza as instâncias do aplicativo no Serviço de Kubernetes do Azure.
* As [Máquinas Virtuais Linux do Azure] [ azurevm-docs] são usadas para executar as instâncias do Jenkins e do Grafana.
* O [Registro de Contêiner do Azure] [ azureacr-docs] armazena e gerencia imagens de contêiner que são usadas pelo cluster do Serviço de Kubernetes do Azure. As imagens são armazenadas com segurança e podem ser replicadas em outras regiões pela plataforma do Azure para acelerar os tempos de implantação.
* O [Serviço de Kubernetes do Azure] [ azureaks-docs] é uma plataforma Kubernetes gerenciada que permite implantar e gerenciar aplicativos em contêineres sem conhecimento de orquestração de contêiner. Como um serviço Kubernetes hospedado, o Azure lida com as tarefas críticas para você, como o monitoramento da integridade e a manutenção.
* O [Azure Cosmos DB] [ azurecosmosdb-docs] é um banco de dados multimodelo distribuído globalmente que permite que você escolha entre vários modelos de banco de dados e de consistência para atender às suas necessidades. Com o Cosmos DB, os dados podem ser replicados globalmente e não há a necessidade de implantação ou configuração de nenhum cluster de gerenciamento ou replicação.
* O [Azure Monitor][azuremonitor-docs] ajuda-o a rastrear o desempenho, manter a segurança e identificar tendências. As métricas obtidas pelo Monitor podem ser usadas por outros recursos e ferramentas, como o Grafana.
* O [Grafana] [ grafana] é uma solução de software livre para consultar, visualizar, alertar e entender métricas. Um plug-in da fonte de dados para o Azure Monitor permite que o Grafana crie painéis visuais para monitorar o desempenho de seus aplicativos em execução no Serviço de Kubernetes do Azure que usam o Cosmos DB.

### <a name="alternatives"></a>Alternativas

* O [Visual Studio Team Services] [ vsts] e o Team Foundation Server ajudam a implementar um pipeline de CI (integração contínua), teste e implantação (CD) em qualquer aplicativo.
* O [Kubernetes] [ kubernetes] pode ser executado diretamente em VMs do Azure em vez de usar um serviço gerenciado, se você quiser mais controle sobre o cluster.
* O [Service Fabric] [ service-fabric] é outro orquestrador de contêiner alternativo que pode substituir o AKS.

## <a name="considerations"></a>Considerações

### <a name="availability"></a>Disponibilidade

Para monitorar o desempenho do aplicativo e relatar problemas, esse cenário combina o Azure Monitor com o Grafana para oferecer painéis visuais. Essas ferramentas permitem monitorar e solucionar problemas de desempenho que podem exigir atualizações de código, e podem todas ser implantadas com o pipeline de CI/CD.

Como parte do cluster do Serviço de Kubernetes do Azure, um balanceador de carga distribui o tráfego do aplicativo para um ou mais contêineres (pods) que executam o aplicativo. Essa abordagem à execução de aplicativos em contêineres no Kubernetes oferece uma infraestrutura altamente disponível para seus clientes.

Para outros tópicos de disponibilidade, confira a [lista de verificação de disponibilidade][availability] disponível no Architecture Center.

### <a name="scalability"></a>Escalabilidade

O Serviço de Kubernetes do Azure permite que você dimensione o número de nós de cluster para atender às demandas dos aplicativos. À medida que seu aplicativo aumenta, você pode escalar horizontalmente o número de nós de Kubernetes que executa o serviço.

Os dados do aplicativo são armazenados no Azure Cosmos DB, um banco de dados multimodelo distribuído globalmente que pode ser dimensionado globalmente. O cosmos DB abstrai a necessidade de dimensionar sua infraestrutura, como é feito com os componentes de banco de dados tradicional, e você pode optar por replicar o Cosmos DB globalmente para atender às demandas de seus clientes.

Para outros tópicos de escalabilidade, confira a [lista de verificação de escalabilidade] [ scalability] disponível no Architecture Center.

### <a name="security"></a>Segurança

Para minimizar a superfície de ataque, esses cenários não expõem a instância da VM Jenkins em HTTP. Para as tarefas de gerenciamento que exigem a interação com o Jenkins, você deve criar uma conexão remota segura usando um túnel SSH em seu computador local. Somente a autenticação de chave pública SSH é permitida para as instâncias de VM Grafana e Jenkins. Os logons com base em senha são desabilitados. Para obter mais informações, confira [Executar um servidor Jenkins no Azure](../../reference-architectures/jenkins/index.md).

Para separar credenciais e permissões, este cenário usa uma entidade de serviço dedicada do Azure AD (Active Directory). As credenciais para essa entidade de serviço são armazenadas como um objeto de credencial segura no Jenkins para não serem diretamente expostas e ficarem visíveis dentro de scripts ou no pipeline de compilação.

Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].

### <a name="resiliency"></a>Resiliência

Este cenário usa o Serviço de Kubernetes do Azure para seu aplicativo. O Kubernetes tem componentes de resiliência embutidos que monitoram e reiniciam os contêineres (pods) em caso de problema. Combinado com vários nós do Kubernetes em execução, seu aplicativo pode tolerar a indisponibilidade de um pod ou nó.

Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].

## <a name="deploy-the-scenario"></a>Implantar o cenário

**Pré-requisitos.**

* Você deve ter uma conta do Azure já criada. Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.
* Você precisa de um par de chaves públicas SSH. Para obter etapas sobre como criar um par de chaves públicas, confira [Criar e usar um par de chaves SSH para VMs Linux][sshkeydocs].
* Você precisa de uma entidade de serviço do Azure AD (Active Directory) para a autenticação de serviço e recursos. Se necessário, crie uma entidade de serviço com [az ad sp create-for-rbac][createsp]

    ```azurecli-interactive
    az ad sp create-for-rbac --name myDevOpsScenario
    ```

    Anote o *appId* e a *senha* na saída desse comando. Forneça esses valores ao modelo quando implantar o cenário.

Para implantar esse cenário com um modelo do Azure Resource Manager, execute as etapas a seguir.

1. Clique no botão **Implantar no Azure**:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fdevops-with-aks%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Aguarde até que a implantação de modelo seja aberta no portal do Azure e conclua as seguintes etapas:
   * Escolha **Criar novo** recurso de grupo e insira um nome como *myAKSDevOpsScenario* na caixa de texto.
   * Selecione uma região na caixa suspensa **Local**.
   * Insira sua ID de entidade de segurança de aplicativo de serviço e a senha no comando `az ad sp create-for-rbac`.
   * Forneça um nome de usuário e uma senha segura para a instância do Jenkins e o console do Grafana.
   * Forneça uma chave SSH para proteger logons nas VMs Linux.
   * Analise os termos e condições e marque a opção **Concordo com os termos e condições declarados acima**.
   * Selecione o botão **Comprar**.

Pode levar 15 a 20 minutos para a implantação ser concluída.

## <a name="pricing"></a>Preços

Para explorar o custo de executar esse cenário, todos os serviços são pré-configurados na calculadora de custos. Para ver como o preço seria alterado em seu caso de uso específico, altere as variáveis apropriadas de acordo com o tráfego esperado. corresponde ao tráfego esperado.

Nós fornecemos três perfis de custo de exemplo com base no número de imagens de contêiner a serem armazenados e nós de Kubernetes para executar seus aplicativos.

* [Pequeno][small-pricing]: referente a mil compilações de contêiner por mês.
* [Pequeno][medium-pricing]: referente a 100 mil compilações de contêiner por mês.
* [Pequeno][large-pricing]: referente a um milhão de compilações de contêiner por mês.

## <a name="related-resources"></a>Recursos relacionados

Este cenário usou o Registro de Contêiner do Azure e o Serviço de Kubernetes do Azure para armazenar e executar seus aplicativos baseados em contêiner. As Instâncias de Contêiner do Azure também podem ser usadas para executar aplicativos baseados em contêiner, sem precisar provisionar componentes de orquestração. Para obter mais informações, confira [Visão geral das Instâncias de Contêiner do Azure][azureaci-docs].

<!-- links -->
[architecture]: ./media/devops-with-aks/architecture-devops-with-aks.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[azureaci-docs]: /azure/container-instances/container-instances-overview
[azureacr-docs]: /azure/container-registry/container-registry-intro
[azurecosmosdb-docs]: /azure/cosmos-db/introduction
[azureaks-docs]: /azure/aks/intro-kubernetes
[azuremonitor-docs]: /azure/monitoring-and-diagnostics/monitoring-overview
[azurevm-docs]: /azure/virtual-machines/linux/overview
[createsp]: /cli/azure/ad/sp#az-ad-sp-create
[grafana]: https://grafana.com/
[jenkins]: https://jenkins.io/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sshkeydocs]: /azure/virtual-machines/linux/mac-create-ssh-keys
[vsts]: /vsts/?view=vsts
[kubernetes]: https://kubernetes.io/
[service-fabric]: /azure/service-fabric/

[small-pricing]: https://azure.com/e/841f0a75b1ea4802ba1ac8f7918a71e7
[medium-pricing]: https://azure.com/e/eea0e6d79b4e45618a96d33383ec77ba
[large-pricing]: https://azure.com/e/3faab662c54c473da55a1e93a27e0e64