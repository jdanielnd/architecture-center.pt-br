---
title: Executar um servidor Jenkins no Azure
description: Essa arquitetura de referência mostra como implantar e operar um servidor Jenkins escalonável, de nível corporativo, no Azure protegido com o logon único.
author: njray
ms.date: 01/21/18
ms.openlocfilehash: 5f9c54e71a8750e88de1ae633ccc1316f8375d3a
ms.sourcegitcommit: 0de300b6570e9990e5c25efc060946cb9d079954
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/03/2018
---
# <a name="run-a-jenkins-server-on-azure"></a>Executar um servidor Jenkins no Azure

Essa arquitetura de referência mostra como implantar e operar um servidor Jenkins escalonável, de nível corporativo, no Azure protegido com o logon único. A arquitetura também usa o Azure Monitor para monitorar o estado do servidor Jenkins. [**Implante essa solução**.](#deploy-the-solution)

![Servidor Jenkins em execução no Azure][0]

*Faça o download do [arquivo Visio](https://archcenter.blob.core.windows.net/cdn/Jenkins-architecture.vsdx) que contém esse diagrama de arquitetura.*

Essa arquitetura oferece suporte à recuperação de desastres com serviços do Azure, mas não abrange cenários de expansão mais avançados que envolvem vários mestres ou alta disponibilidade sem tempo de inatividade. Para obter insights gerais sobre os vários componentes do Azure, incluindo um tutorial passo a passo sobre como criar um pipeline de CI/CD no Azure, confira [Jenkins no Azure][jenkins-on-azure].

Este documento tem como foco as principais operações do Azure que são necessárias para dar suporte ao Jenkins, incluindo o uso do Armazenamento do Microsoft Azure para manter os artefatos de compilação, os itens de segurança necessários para logon único, outros serviços que podem ser integrados e a escalabilidade para o pipeline. A arquitetura é projetada para funcionar com um repositório de controle do código-fonte existente. Por exemplo, é comum iniciar trabalhos do Jenkins com base nas confirmações do GitHub.

## <a name="architecture"></a>Arquitetura

Essa arquitetura consiste nos seguintes componentes:

- **Grupo de recursos.** Um [grupo de recursos][rg] agrupa os ativos do Azure para que eles possam ser gerenciados por tempo de vida, proprietário e outros critérios. Use grupos de recursos para implantar e monitorar ativos do Azure como um grupo e controlar custos de cobrança por grupo de recursos. Também é possível excluir recursos como um conjunto, o que é muito útil para implantações de teste.

- **Servidor Jenkins**. Uma máquina virtual é implantada para executar o [Jenkins][azure-market] como um servidor de automação e servir como o mestre Jenkins. Esta arquitetura de referência usa o [modelo de solução para Jenkins no Azure][solution], instalado em uma máquina virtual do Linux (Ubuntu 16.04 LTS) no Azure. Há outras ofertas de Jenkins disponíveis no Azure Marketplace.

  > [!NOTE]
  > O Nginx é instalado na máquina virtual para servir como um proxy reverso para o Jenkins. Você pode configurar o Nginx para habilitar o SSL para o servidor Jenkins.
  > 
  > 

- **Rede virtual**. Uma [rede virtual][vnet] conecta os recursos do Azure entre si e fornece isolamento lógico. Nessa arquitetura, o servidor do Jenkins é executado em uma rede virtual.

- **Sub-redes**. O servidor do Jenkins é isolado em uma [sub-rede][subnet] para facilitar o gerenciamento e a segregação do tráfego de rede sem afetar o desempenho.

- <strong>NSGs</strong>. Use [NSGs (grupos de segurança de rede)][nsg] para restringir o tráfego de rede da Internet à sub-rede de uma rede virtual.

- **Discos gerenciados**. Um [disco gerenciado][managed-disk] é um VHD (disco rígido virtual) persistente usado para armazenar aplicativos, manter o estado do servidor do Jenkins e fornecer recuperação de desastres. Os discos de dados são armazenados no Armazenamento do Microsoft Azure. Para o alto desempenho, é recomendado o [armazenamento premium][premium].

- **Armazenamento de Blobs do Azure**. O [plug-in do Armazenamento do Microsoft Azure][configure-storage] usa o Armazenamento de Blobs do Azure para armazenar os artefatos de compilação criados e compartilhados com outras compilações do Jenkins.

- <strong>Azure Active Directory (Azure AD)</strong>. O [Azure AD][azure-ad] dá suporte à autenticação de usuário, permitindo que você configure o recurso de logon único. As [entidades de serviço][service-principal] do Azure AD definem a política e as permissões para cada autorização de função no fluxo de trabalho usando o [RBAC (controle de acesso baseado em função)][rbac]. Toda entidade de serviço está associada a um trabalho Jenkins.

- **Azure Key Vault.** Para gerenciar as chaves criptográficas e os segredos usados para provisionar recursos do Azure quando são necessários segredos, essa arquitetura usa o [Azure Key Vault][key-vault]. Para obter mais ajuda com o armazenamento de segredos associados ao aplicativo no pipeline, consulte também o plug-in das [Credenciais do Azure][configure-credential] para o Jenkins.

- **Serviços de monitoramento do Azure**. Esse serviço [monitora][monitor] a máquina virtual do Azure que hospeda o Jenkins. Essa implantação monitora o status da máquina virtual e a utilização da CPU e envia alertas.

## <a name="recommendations"></a>Recomendações

As seguintes recomendações aplicam-se à maioria dos cenários. Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.

### <a name="azure-ad"></a>AD do Azure

O locatário do [Azure AD][azure-ad] referente à sua assinatura é usado para habilitar o logon único para usuários do Jenkins e configurar [entidades de serviço][service-principal] que permitem aos trabalhos do Jenkins acessar os recursos do Azure.

A autenticação e a autorização do logon único são implementadas pelo plug-in do Microsoft Azure Active Directory, instalado no servidor Jenkins. O logon único permite que você se autentique usando as credenciais de organização do Microsoft Azure Active Directory ao fazer logon no servidor Jenkins. Ao configurar o plug-in do Azure AD, você pode especificar o nível do acesso autorizado de um usuário para o servidor Jenkins.

Para fornecer aos trabalhos Jenkins acesso aos recursos do Azure, um administrador do Microsoft Azure Active Directory cria entidades de serviço. Essas entidades concedem aos aplicativos — nesse caso, os trabalhos Jenkins — [acesso autenticado e autorizado][ad-sp] aos recursos do Azure.

O [RBAC][rbac] define e controla o acesso aos recursos do Azure para usuários ou entidades de serviço por meio da função que lhes foi atribuída. Há suporte para as funções internas e personalizadas. As funções também ajudam a proteger o pipeline e a garantir que as responsabilidades de um usuário ou do agente sejam atribuídas e autorizadas corretamente. Além disso, você pode configurar o RBAC para limitar o acesso aos ativos do Azure. Por exemplo, um usuário pode ser limitado a trabalhar apenas com os ativos em um determinado grupo de recursos.

### <a name="storage"></a>Armazenamento

Use o [plug-in do Armazenamento do Microsoft Azure][storage-plugin] do Jenkins, que é instalado por meio do Azure Marketplace, para armazenar os artefatos de compilação que podem ser compartilhados com testes e outras compilações. É preciso haver uma conta configurada do Armazenamento do Microsoft Azure para esse plug-in ser usado por trabalhos do Jenkins.

### <a name="jenkins-azure-plugins"></a>Plug-ins do Azure para o Jenkins

O modelo de solução para o Jenkins no Azure instala vários plug-ins do Azure. A equipe do Azure DevOps cria e mantém o modelo de solução e os seguintes plug-ins, que funcionam com outras ofertas Jenkins no Azure Marketplace, bem como qualquer mestre Jenkins configurado no local:

-   O [plug-in do Microsoft Azure Active Directory][configure-azure-ad] permite que o servidor Jenkins suporte o logon único para usuários com base no Microsoft Azure Active Directory.

-   O plug-in [Agentes de VM do Azure][configure-agent] usa um modelo Azure Resource Manager para criar agentes do Jenkins nas máquinas virtuais do Azure.

-   O plug-in de [Credenciais do Azure][configure-credential] permite que você armazene as credenciais da entidade de serviço do Microsoft Azure no Jenkins.

-   O [plug-in do Armazenamento do Microsoft Azure][configure-storage] carrega artefatos de compilação ou faz o download de dependências de compilação do [Armazenamento de Blobs do Azure][blob].

Também recomendamos que confira a lista crescente de todos os plug-ins do Azure disponíveis que funcionam com os recursos do Azure. Para ver a lista mais recente, visite [Índice de plug-in do Jenkins][index] e pesquise por Azure. Por exemplo, os seguintes plug-ins estão disponíveis para implantação:

-   Os [Agentes de Contêiner do Azure][container-agents] ajudam a executar um contêiner como um agente no Jenkins.

-   A [implantação contínua do Kubernetes](https://aka.ms/azjenkinsk8s) implanta as configurações de recursos em um cluster Kubernetes.

-   O [Serviço de Contêiner do Azure][acs] implanta configurações no Serviço de Contêiner do Azure com o Kubernetes, DC/OS com o Marathon ou o Docker Swarm.

-   O [Azure Functions][functions] implanta seu projeto na Função do Azure.

-   O [Serviço de Aplicativo do Azure][app-service] implanta no Serviço de Aplicativo do Azure.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

O Jenkins pode ser dimensionado para suportar cargas de trabalho muito grandes. Para compilações elásticas, não execute compilações no servidor mestre do Jenkins. Em vez disso, descarregue tarefas de compilação nos agentes Jenkins, que podem ser reduzidos e expandidos de forma elástica conforme necessário. Considere duas opções para o dimensionamento de agentes:

- Use o plug-in [Agentes de VM do Azure][vm-agent] para criar agentes Jenkins que são executados nas VMs do Azure. Esse plug-in permite a expansão elástica dos agentes e pode usar tipos distintos de máquinas virtuais. Você pode selecionar uma imagem base diferente do Azure Marketplace ou usar uma imagem personalizada. Para obter detalhes sobre como os agentes Jenkins são dimensionados, consulte [Arquitetura para o dimensionamento][scale] na documentação do Jenkins.

- Use o plug-in [Agentes de Contêiner do Azure][container-agents] para executar um contêiner como um agente no [Serviço de Contêiner do Azure com o Kubernetes](/azure/container-service/kubernetes/) ou nas [Instâncias de Contêiner do Azure](/azure/container-instances/).

O dimensionamento das máquinas virtuais geralmente é mais caro que o dos contêineres. Para usar os contêineres para o dimensionamento, no entanto, o processo de compilação deve executar com os contêineres.

Além disso, use o Armazenamento do Microsoft Azure para compartilhar artefatos de compilação que podem ser usados no próximo estágio do pipeline por outros agentes de compilação.

### <a name="scaling-the-jenkins-server"></a>Dimensionamento do servidor Jenkins 

Você pode dimensionar a VM de servidor Jenkins vertical ou horizontalmente alterando o tamanho da VM. O [modelo de solução para Jenkins no Azure][azure-market] especifica o tamanho de DS2 v2 (com duas CPUs, 7 GB) por padrão. Esse tamanho lida com uma carga de trabalho de equipe pequena a média. Altere o tamanho da VM escolhendo uma opção diferente na criação do servidor. 

A seleção do tamanho de servidor correto depende do tamanho da carga de trabalho esperada. A comunidade Jenkins mantém um [guia de seleção][selection-guide] para ajudar a identificar a configuração que melhor atenda às suas necessidades. O Azure oferece muitos [tamanhos de VMs do Linux][sizes-linux] de modo que todos os requisitos sejam atendidos. Para obter mais informações sobre como dimensionar o mestre Jenkins, consulte a comunidade Jenkins de [práticas recomendadas][best-practices], que também inclui detalhes sobre como o dimensionar o mestre Jenkins.


## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Disponibilidade no contexto de um servidor Jenkins significa ser capaz de recuperar as informações do estado associadas a seu fluxo de trabalho, como os resultados do teste, bibliotecas criadas ou outros artefatos. O estado do fluxo de trabalho crítico ou artefatos devem ser mantidos para recuperar o fluxo de trabalho, caso o servidor Jenkins falhe. Para avaliar seus requisitos de disponibilidade, considere duas métricas comuns:

-   O RTO (Objetivo de Tempo de Recuperação) especifica quanto tempo você pode prosseguir sem o Jenkins.

-   O RPO (Objetivo de Ponto de Recuperação) indica a quantidade de dados que você pode perder caso uma interrupção no serviço afete o Jenkins.

Na prática, o RTO e o RPO implicam redundância e backup. A disponibilidade não é uma questão de recuperação de hardware, que faz parte do Azure, mas sim de garantir que você mantenha o estado do servidor Jenkins. A Microsoft oferece um [contrato de nível de serviço][sla] (SLA) para as instâncias únicas da VM. Se esse SLA não atender aos seus requisitos de tempo de atividade, garanta que tem um plano de recuperação de desastres ou considere o uso da implantação de um [servidor Jenkins de vários mestres][multi-master] (não abordado neste documento).

Considere o uso de [scripts][disaster] da recuperação de desastres na etapa 7 da implantação para criar uma conta de Armazenamento do Microsoft Azure com discos gerenciados para armazenar o estado do servidor Jenkins. Se o Jenkins falhar, poderá ser restaurado para o estado armazenado nesta conta de armazenamento separada.

## <a name="security-considerations"></a>Considerações de segurança

Use as seguintes abordagens para ajudar a bloquear a segurança em um servidor Jenkins básico, já que em seu estado básico ele não é seguro.

-   Configure uma maneira segura de fazer logon no servidor Jenkins. Essa arquitetura usa HTTP e tem um IP público, mas o HTTP não é seguro por padrão. Considere a configuração de [HTTPS no servidor Nginx][nginx] que está sendo usado para um logon seguro.

    > [!NOTE]
    > Ao adicionar o SSL ao servidor, crie uma regra NSG para a sub-rede Jenkins a fim de abrir a porta 443. Para obter mais informações, consulte [Como abrir portas em uma máquina virtual com o portal do Azure][port443].
    > 

-   Garanta que a configuração Jenkins impeça a solicitação intersite forjada (Gerenciar Jenkins \> Configurar a segurança Global). Esse é o padrão para o Servidor Jenkins da Microsoft.

-   Configure o acesso somente leitura ao painel Jenkins usando o [plug-in de estratégia de autorização de matriz][matrix].

-   Instale o plug-in das [Credenciais do Azure][configure-credential] para usar o Key Vault para lidar com segredos dos ativos do Azure, com os agentes no pipeline e com os componentes de terceiros.

-   Use o RBAC para restringir o acesso da entidade de serviço para o mínimo necessário para executar os trabalhos. Isso ajuda a limitar o escopo de danos de um trabalho invasor.

Os trabalhos Jenkins geralmente requerem segredos para acessar os serviços do Azure que exigem autorização, como o Serviço de Contêiner do Azure. Use o [Key Vault][key-vault] juntamente com o [plug-in de Credencial do Azure][configure-credential] para gerenciar esses segredos com segurança. Use o Key Vault para armazenar credenciais da entidade de serviço, senhas, tokens e outros segredos.

Use a [Central de Segurança do Azure][security-center] para obter uma exibição central do estado da segurança de seus recursos do Azure. A Central de Segurança monitora problemas de segurança potenciais e fornece uma visão abrangente da integridade de segurança de sua implantação. A Central de Segurança é configurada por assinatura do Azure. Habilite a coleta de dados de segurança, conforme descrito no [Guia de início rápido da Central de Segurança do Azure][quick-start]. Depois que a coleta de dados for habilitada, a Central de Segurança examinará automaticamente todas as máquinas virtuais criadas nessa assinatura.

O servidor Jenkins tem seu próprio sistema de gerenciamento de usuário, e a comunidade Jenkins fornece as práticas recomendadas para a [proteção de uma instância Jenkins no Azure][secure-jenkins]. O modelo de solução para Jenkins no Azure implementa essas práticas recomendadas.

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Use grupos de recursos para organizar os recursos do Azure implantados. Implante ambientes de produção e ambientes de desenvolvimento/teste em grupos de recursos separados, para que você possa monitorar os recursos de cada ambiente e acumular custos de cobrança por grupo de recursos. Também é possível excluir recursos como um conjunto, o que é muito útil para implantações de teste.

O Azure fornece vários recursos para [monitoramento e diagnóstico][monitoring-diag] da infraestrutura geral. Para monitorar o uso da CPU, essa arquitetura implanta o Azure Monitor. Por exemplo, você pode usar o Azure Monitor para monitorar a utilização da CPU e enviar uma notificação se o uso da CPU exceder os 80 por cento. (Um elevado uso da CPU indica que você deseja dimensionar verticalmente a VM do servidor Jenkins.) Você também pode notificar um usuário designado se a VM falhar ou ficar indisponível.

## <a name="communities"></a>Comunidades

As comunidades podem responder a perguntas e ajudá-lo a configurar uma implantação bem-sucedida. Considere o seguinte:

-   [Blog da comunidade Jenkins](https://jenkins.io/node/)
-   [Fórum do Azure](https://azure.microsoft.com/support/forums/)
-   [Stack Overflow Jenkins](https://stackoverflow.com/tags/jenkins/info)

Para mais práticas recomendadas da comunidade Jenkins, visite [Práticas recomendadas Jenkins][jenkins-best].

## <a name="deploy-the-solution"></a>Implantar a solução

Para implantar essa arquitetura, siga as etapas abaixo para instalar o [modelo de solução para o Jenkins no Azure][azure-market]. Em seguida, instale os scripts que configuram o monitoramento e a recuperação de desastres nas etapas abaixo.

### <a name="prerequisites"></a>pré-requisitos

- Essa arquitetura de referência requer uma assinatura do Azure. 
- Para criar uma entidade de serviço do Azure, você deve ter direitos de administrador para o locatário do Microsoft Azure Active Directory que está associado ao servidor Jenkins implantado.
- Essas instruções pressupõem que o administrador Jenkins também é um usuário do Azure com, pelo menos, privilégios de Colaborador.

### <a name="step-1-deploy-the-jenkins-server"></a>Etapa 1: Implantar o servidor Jenkins

1.  Abra a [imagem do Marketplace do Azure para Jenkins][azure-market] em seu navegador da Web e selecione **OBTER AGORA** no lado esquerdo da página.

2.  Examine os detalhes de preços e selecione **Continuar**. Em seguida, selecione **Criar** para configurar o servidor Jenkins no portal do Azure.

Para obter instruções detalhadas, consulte [Criar um servidor Jenkins em uma VM Linux do Azure no portal do Azure][create-jenkins]. Para essa arquitetura de referência, executar o servidor com o logon de administrador é suficiente. Em seguida, você poderá provisioná-lo para usar vários outros serviços.

### <a name="step-2-set-up-sso"></a>Etapa 2: Configurar o logon único

A etapa é executada pelo administrador Jenkins, que também deve ter uma conta de usuário no diretório do Microsoft Azure Active Directory da assinatura e deve receber a função de Colaborador.

Use o [Plug-in do Azure AD][configure-azure-ad] no Centro de Atualização Jenkins no servidor Jenkins e siga as instruções para configurar o SSO.

### <a name="step-3-provision-jenkins-server-with-azure-vm-agent-plugin"></a>Etapa 3: Provisionar ao servidor Jenkins um plug-in do Agente de VM do Azure

O administrador do Jenkins executa a etapa para configurar o plug-in do agente de VM do Azure, que já está instalado.

[Siga estas etapas para configurar o plug-in][configure-agent]. Para obter um tutorial sobre como configurar as entidades de serviço para o plug-in, consulte [Dimensionar as implantações Jenkins para atender à demanda com agentes de VM do Azure][scale-agent].

### <a name="step-4-provision-jenkins-server-with-azure-storage"></a>Etapa 4: Provisionar ao servidor Jenkins um Armazenamento do Microsoft Azure

O administrador do Jenkins executa a etapa, que configura o plug-in do Armazenamento do Microsoft Azure, que já está instalado.

[Siga estas etapas para configurar o plug-in][configure-storage].

### <a name="step-5-provision-jenkins-server-with-azure-credential-plugin"></a>Etapa 5: Provisionar ao servidor Jenkins um plug-in de Credencial do Azure

O administrador do Jenkins executa a etapa para configurar o plug-in do agente de Credencial do Azure, que já está instalado.

[Siga estas etapas para configurar o plug-in][configure-credential].

### <a name="step-6-provision-jenkins-server-for-monitoring-by-the-azure-monitor-service"></a>Etapa 6: Provisione o servidor Jenkins para o monitoramento pelo Serviço do Azure Monitor

Para configurar o monitoramento para seu servidor Jenkins, siga as instruções em [Criar alertas de métrica no Azure Monitor para os serviços do Azure][create-metric].

### <a name="step-7-provision-jenkins-server-with-managed-disks-for-disaster-recovery"></a>Etapa 7: Provisione Managed Disks ao servidor Jenkins para a recuperação de desastres

O grupo do produtos Microsoft Jenkins criou scripts de recuperação de desastres que compilam um disco gerenciado usado para salvar o estado de Jenkins. Se o servidor desligar, poderá ser restaurado a seu último estado.

Baixe e execute os scripts de recuperação de desastres do [GitHub][disaster].

[acs]: https://aka.ms/azjenkinsacs
[ad-sp]: /azure/active-directory/develop/active-directory-integrating-applications
[app-service]: https://plugins.jenkins.io/azure-app-service
[azure-ad]: /azure/active-directory/
[azure-market]: https://azuremarketplace.microsoft.com/marketplace/apps/azure-oss.jenkins?tab=Overview
[best-practices]: https://jenkins.io/doc/book/architecting-for-scale/
[blob]: /azure/storage/common/storage-java-jenkins-continuous-integration-solution
[configure-azure-ad]: https://plugins.jenkins.io/azure-ad
[configure-agent]: https://plugins.jenkins.io/azure-vm-agents
[configure-credential]: https://plugins.jenkins.io/azure-credentials
[configure-storage]: https://plugins.jenkins.io/windows-azure-storage
[container-agents]: https://aka.ms/azcontaineragent
[create-jenkins]: /azure/jenkins/install-jenkins-solution-template
[create-metric]: /azure/monitoring-and-diagnostics/insights-alerts-portal
[disaster]: https://github.com/Azure/jenkins/tree/master/disaster_recovery
[functions]: https://aka.ms/azjenkinsfunctions
[index]: https://plugins.jenkins.io
[jenkins-best]: https://wiki.jenkins.io/display/JENKINS/Jenkins+Best+Practices
[jenkins-on-azure]: /azure/jenkins/
[key-vault]: /azure/key-vault/
[managed-disk]: /azure/virtual-machines/linux/managed-disks-overview
[matrix]: https://plugins.jenkins.io/matrix-auth
[monitor]: /azure/monitoring-and-diagnostics/
[monitoring-diag]: /azure/architecture/best-practices/monitoring
[multi-master]: https://jenkins.io/doc/book/architecting-for-scale/
[nginx]: https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04
[nsg]: /azure/virtual-network/virtual-networks-nsg
[quick-start]: /azure/security-center/security-center-get-started
[port443]: /azure/virtual-machines/windows/nsg-quickstart-portal
[premium]: /azure/virtual-machines/linux/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rg]: /azure/azure-resource-manager/resource-group-overview
[scale]: https://jenkins.io/doc/book/architecting-for-scale/
[scale-agent]: /azure/jenkins/jenkins-azure-vm-agents
[selection-guide]: https://jenkins.io/doc/book/hardware-recommendations/
[service-principal]: /azure/active-directory/develop/active-directory-application-objects
[secure-jenkins]: https://jenkins.io/blog/2017/04/20/secure-jenkins-on-azure/
[security-center]: /azure/security-center/security-center-intro
[sizes-linux]: /azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[solution]: https://azure.microsoft.com/blog/announcing-the-solution-template-for-jenkins-on-azure/
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
[storage-plugin]: https://wiki.jenkins.io/display/JENKINS/Windows+Azure+Storage+Plugin
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[vm-agent]: https://wiki.jenkins.io/display/JENKINS/Azure+VM+Agents+plugin
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/jenkins-server.png 
