---
title: Migrando um aplicativo Web herdado para uma arquitetura baseada em API no Azure
description: Use o Gerenciamento de API do Azure para modernizar um aplicativo Web herdado.
author: begim
ms.date: 09/13/2018
ms.openlocfilehash: f468b3c6dc1c58e03555613b152882316ae2a017
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610576"
---
# <a name="migrating-a-legacy-web-application-to-an-api-based-architecture-on-azure"></a>Migrando um aplicativo Web herdado para uma arquitetura baseada em API no Azure

Uma empresa de comércio eletrônico no setor de viagem está modernizando sua pilha de softwares herdados baseados em navegador. Embora sua pilha existente seja principalmente monolítica, existem alguns [serviços HTTP baseados em SOAP][soap] de um projeto recente. A empresa está considerando a criação de fluxos de receita adicional para monetizar parte da propriedade intelectual interna que foi desenvolvida.

As metas do projeto incluem endereçar a dívida técnica, melhorar a manutenção contínua e acelerar o desenvolvimento de recursos com menos erros de regressão. O projeto usará um processo iterativo para evitar o risco, com algumas etapas executadas em paralelo:

* A equipe de desenvolvimento modernizará o aplicativo back-end, que é composto por bancos de dados relacionais hospedados em VMs.
* A equipe de desenvolvimento interno criará a nova funcionalidade do negócio, que será exposta por novas APIs de HTTP.
* Uma equipe de desenvolvimento de contrato criará uma nova interface do usuário com base em navegador, que será hospedada no Azure.

Os novos recursos do aplicativo serão entregues em estágios. Estes recursos substituirão gradualmente a funcionalidade existente da interface do usuário cliente-servidor baseada em navegador (hospedado no local) que capacita o negócio de comércio eletrônico atualmente.

A equipe de gerenciamento não deseja modernizar desnecessariamente. Ela também quer manter o controle de escopo e custos. Para isso, a equipe decidiu preservar os serviços HTTP de SOAP existentes. Ela também pretende minimizar as alterações na interface do usuário existente. O [Gerenciamento de API do Azure (APIM)][apim] pode ser utilizado para atender muitos dos requisitos e restrições do projeto.

## <a name="architecture"></a>Arquitetura

![Diagrama da arquitetura][architecture]

A nova interface do usuário será hospedada como um aplicativo da plataforma como serviço (PaaS) no Azure e dependerá das APIs de HTTP existentes e novas. Essas APIs serão fornecidas com um conjunto de interfaces melhor projetado, permitindo a melhoria do desempenho, integração mais fácil e extensibilidade futura.

### <a name="components-and-security"></a>Segurança e componentes

1. O aplicativo web local existente continuará a consumir diretamente os serviços web locais existentes.
2. As chamadas do aplicativo web existente para os serviços HTTP existentes permanecerão inalteradas. Essas chamadas são internas na rede corporativa.
3. Chamadas de entrada são feitas do Azure para os serviços internos existentes:
    * A equipe de segurança permite que o tráfego da instância do APIM passe pelo firewall corporativo para os serviços locais existentes [usando o transporte seguro (HTTPS/SSL)][apim-ssl].
    * A equipe de operações permitirá as chamadas de entrada para os serviços somente da instância do APIM. Esse requisito é atendido pela [ lista de permissões do endereço IP da instância do APIM][apim-whitelist-ip] dentro do perímetro da rede corporativa.
    * Um novo módulo é configurado no pipeline de solicitação de serviços HTTP locais (para agir **somente** nas conexões originadas externamente), que validará [um certificado fornecido pelo APIM][apim-mutualcert-auth].
1. A nova API:
    * É exibida apenas por meio da instância APIM, que fornecerá a fachada de API. A nova API não será acessada diretamente.
    * É desenvolvida e publicada como um [aplicativo de API Web de PaaS do Azure][azure-api-apps].
    * Está na lista de permissões (via [Configurações de aplicativo Web][azure-appservice-ip-restrict]) para aceitar somente a [VIP do APIM][apim-faq-vip].
    * É hospedada em aplicativos Web do Azure com o SSL/transporte seguro ativado.
    * Tem autorização habilitada, [fornecida pelo Serviço de Aplicativo do Azure][azure-appservice-auth] usando o Azure Active Directory e OAuth2.
2. O novo aplicativo web baseado em navegador dependerá da instância de Gerenciamento de API do Azure para **ambas** as APIs HTTP, a nova e a existente.

A instância do APIM será configurada para mapear os serviços de HTTP herdados para um novo contrato de API. Assim, a nova interface do usuário da Web desconhece a integração com um conjunto de serviços/APIs herdadas e as novas APIs. No futuro, a equipe de projeto transferirá gradualmente a funcionalidade de porta para as novas APIs e desativará os serviços originais. Essas alterações serão tratadas dentro da configuração do APIM, deixando a interface do usuário de front-end inalterada e evitando trabalho de redesenvolvimento.

### <a name="alternatives"></a>Alternativas

* Se a organização planeja mover totalmente sua infraestrutura para o Azure, incluindo as VMs que hospedam os aplicativos herdados, ainda assim o APIM ainda seria uma ótima opção, pois ele pode atuar como uma fachada para qualquer ponto de extremidade HTTP endereçável.
* Se o cliente decidiu manter os pontos de extremidade privados existentes e não mostrá-los publicamente, sua instância de Gerenciamento de API pode ser vinculada a uma [Rede Virtual do Azure (VNet)][azure-vnet]:
  * Em um [cenário de comparação e deslocamento do Azure][azure-vm-lift-shift] vinculado à sua Rede Virtual do Azure implantada, o cliente pode endereçar diretamente o serviço back-end por meio de endereços IP privados.
  * No cenário local, a instância de Gerenciamento de API pode alcançar o serviço interno de modo privado por meio de um [Gateway de VPN do Azure e da conexão VPN IPSec de site a site][azure-vpn] ou do [ExpressRoute][azure-er] fazendo disto um [cenário local e híbrido do Azure][azure-hybrid].
* A instância de gerenciamento de API pode ser mantida privada ao implantar a instância de Gerenciamento de API no modo interno. A implantação, então, pode ser usada com um [Gateway de Aplicativo do Azure] [azure-appgw] para habilitar o acesso público para algumas APIs, enquanto outras permanecem internas. Para saber mais, veja [Conectar a APIM no modo interno para uma VNET][apim-vnet-internal].

> [!NOTE]
> Para obter informações gerais sobre como conectar o Gerenciamento de API a uma VNET, [confira aqui][apim-vnet].

### <a name="availability-and-scalability"></a>Disponibilidade e escalabilidade

* O Gerenciamento de API do Azure pode ser [escalado horizontalmente][apim-scaleout] Como escolher um tipo de preço e, em seguida, adicionando as unidades.
* A escalabilidade também acontece [automaticamente com o dimensionamento automático][apim-autoscale].
* A [implantação em várias regiões][apim-multi-regions] permitirá opções de failover e pode ser realizada na [camada Premium][apim-pricing].
* Considere a [Integração com o Azure Application Insights][azure-apim-ai], que também exibe as métricas para monitoramento por meio do [Azure Monitor][azure-mon].

## <a name="deployment"></a>Implantação

Para começar, [crie uma instância de Gerenciamento de API do Azure no portal.][apim-create]

Como alternativa, é possível escolher um [modelo de início rápido][azure-quickstart-templates-apim] existente do Azure Resource Manager que se encaixa no seu caso de uso específico.

## <a name="pricing"></a>Preços

O Gerenciamento de API é fornecido em quatro camadas: desenvolvedor, básica, standard e premium. Você pode encontrar orientações detalhadas sobre a diferença dos níveis [aqui nas diretrizes de preços de Gerenciamento de API do Azure.][apim-pricing]

Os clientes podem escalonar o Gerenciamento de API adicionando e removendo unidades. Cada unidade tem capacidades que dependem de sua camada.

> [!NOTE]
> A camada de Desenvolvedor pode ser usada para avaliação dos recursos de Gerenciamento de API. A camada de desenvolvedor não deve ser usada para produção.

Para exibir os custos projetados e personalizar as suas necessidades de implantação, você pode modificar o número de unidades de escala e de instâncias de Serviço do Aplicativo na [Calculadora de Preços do Azure][pricing-calculator].

## <a name="related-resources"></a>Recursos relacionados

Revise a extensa [documentação e os artigos de referência][apim] sobre Gerenciamento de API do Azure.


<!-- links -->
[architecture]: ./media/architecture-apim-api-scenario.png
[apim-create]: /azure/api-management/get-started-create-service-instance
[apim-git]: /azure/api-management/api-management-configuration-repository-git
[apim-multi-regions]: /azure/api-management/api-management-howto-deploy-multi-region
[apim-autoscale]: /azure/api-management/api-management-howto-autoscale
[apim-scaleout]: /azure/api-management/upgrade-and-scale
[azure-apim-ai]: /azure/api-management/api-management-howto-app-insights
[azure-ai]: /azure/application-insights/
[azure-mon]: /azure/monitoring-and-diagnostics/monitoring-overview
[azure-appgw]: /azure/application-gateway/application-gateway-introduction
[apim-vnet-internal]: /azure/api-management/api-management-howto-integrate-internal-vnet-appgateway
[apim-vnet]: /azure/api-management/api-management-using-with-vnet
[azure-hybrid]: /azure/architecture/reference-architectures/hybrid-networking/
[azure-er]: /azure/expressroute/expressroute-introduction
[azure-vpn]: /azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal
[azure-vnet]: /azure/virtual-network/virtual-networks-overview
[azure-appservice-auth]: /azure/app-service/app-service-authentication-overview#identity-providers
[apim-faq-vip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[azure-appservice-ip-restrict]: /azure/app-service/app-service-ip-restrictions
[azure-api-apps]: /azure/app-service/
[apim-ssl]: /azure/api-management/api-management-howto-manage-protocols-ciphers
[apim-mutualcert-auth]: /azure/api-management/api-management-howto-mutual-certificates
[apim-whitelist-ip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[anti-corruption-layer-pattern]: /azure/architecture/patterns/anti-corruption-layer
[apim]: /azure/api-management/api-management-key-concepts
[apim-api-design-guidance]: /azure/architecture/best-practices/api-design
[visualstudio-youtube-solid-design]: https://youtu.be/agkWYPUcLpg
[azure-vm-lift-shift]: https://azure.microsoft.com/resources/azure-virtual-datacenter-lift-and-shift-guide/
[standard-pricing-calc]: https://azure.com/e/
[premium-pricing-calc]: https://azure.com/e/
[apim-pricing]: https://azure.microsoft.com/pricing/details/api-management/
[azure-quickstart-templates-apim]: https://azure.microsoft.com/resources/templates/?term=API+Management&pageNumber=1
[soap]: https://en.wikipedia.org/wiki/SOAP
[pricing-calculator]: https://azure.com/e/0e916a861fac464db61342d378cc0bd6
