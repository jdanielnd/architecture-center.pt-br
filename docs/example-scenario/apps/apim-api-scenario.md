---
title: Migrando um aplicativo Web para uma arquitetura baseada em API
titleSuffix: Azure Example Scenarios
description: Use o Gerenciamento de API do Azure para modernizar um aplicativo Web herdado.
author: begim
ms.date: 09/13/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-apim-api-scenario.png
ms.openlocfilehash: cf3d4b7ed7fce04e6688f68e382caeec78abd100
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54907955"
---
# <a name="migrating-a-legacy-web-application-to-an-api-based-architecture-on-azure"></a><span data-ttu-id="b6d58-103">Migrando um aplicativo Web herdado para uma arquitetura baseada em API no Azure</span><span class="sxs-lookup"><span data-stu-id="b6d58-103">Migrating a legacy web application to an API-based architecture on Azure</span></span>

<span data-ttu-id="b6d58-104">Uma empresa de comércio eletrônico no setor de viagem está modernizando sua pilha de softwares herdados baseados em navegador.</span><span class="sxs-lookup"><span data-stu-id="b6d58-104">An e-commerce company in the travel industry is modernizing their legacy browser-based software stack.</span></span> <span data-ttu-id="b6d58-105">Embora sua pilha existente seja principalmente monolítica, existem alguns [serviços HTTP baseados em SOAP][soap] de um projeto recente.</span><span class="sxs-lookup"><span data-stu-id="b6d58-105">While their existing stack is mostly monolithic, some [SOAP-based HTTP services][soap] exist from a recent project.</span></span> <span data-ttu-id="b6d58-106">A empresa está considerando a criação de fluxos de receita adicional para monetizar parte da propriedade intelectual interna que foi desenvolvida.</span><span class="sxs-lookup"><span data-stu-id="b6d58-106">They are considering the creation of additional revenue streams to monetize some of the internal intellectual property that's been developed.</span></span>

<span data-ttu-id="b6d58-107">As metas do projeto incluem endereçar a dívida técnica, melhorar a manutenção contínua e acelerar o desenvolvimento de recursos com menos erros de regressão.</span><span class="sxs-lookup"><span data-stu-id="b6d58-107">Goals for the project include addressing technical debt, improving ongoing maintenance, and accelerating feature development with fewer regression bugs.</span></span> <span data-ttu-id="b6d58-108">O projeto usará um processo iterativo para evitar o risco, com algumas etapas executadas em paralelo:</span><span class="sxs-lookup"><span data-stu-id="b6d58-108">The project will use an iterative process to avoid risk, with some steps performed in parallel:</span></span>

- <span data-ttu-id="b6d58-109">A equipe de desenvolvimento modernizará o aplicativo back-end, que é composto por bancos de dados relacionais hospedados em VMs.</span><span class="sxs-lookup"><span data-stu-id="b6d58-109">The development team will modernize the application back end, which is composed of relational databases hosted on VMs.</span></span>
- <span data-ttu-id="b6d58-110">A equipe de desenvolvimento interno criará a nova funcionalidade do negócio, que será exposta por novas APIs de HTTP.</span><span class="sxs-lookup"><span data-stu-id="b6d58-110">The in-house development team will write new business functionality that will be exposed over new HTTP APIs.</span></span>
- <span data-ttu-id="b6d58-111">Uma equipe de desenvolvimento de contrato criará uma nova interface do usuário com base em navegador, que será hospedada no Azure.</span><span class="sxs-lookup"><span data-stu-id="b6d58-111">A contract development team will build a new browser-based UI, which will be hosted in Azure.</span></span>

<span data-ttu-id="b6d58-112">Os novos recursos do aplicativo serão entregues em estágios.</span><span class="sxs-lookup"><span data-stu-id="b6d58-112">New application features will be delivered in stages.</span></span> <span data-ttu-id="b6d58-113">Estes recursos substituirão gradualmente a funcionalidade existente da interface do usuário cliente-servidor baseada em navegador (hospedado no local) que capacita o negócio de comércio eletrônico atualmente.</span><span class="sxs-lookup"><span data-stu-id="b6d58-113">These features will gradually replace the existing browser-based client-server UI functionality (hosted on-premises) that powers their e-commerce business today.</span></span>

<span data-ttu-id="b6d58-114">A equipe de gerenciamento não deseja modernizar desnecessariamente.</span><span class="sxs-lookup"><span data-stu-id="b6d58-114">The management team does not want to modernize unnecessarily.</span></span> <span data-ttu-id="b6d58-115">Ela também quer manter o controle de escopo e custos.</span><span class="sxs-lookup"><span data-stu-id="b6d58-115">They also want to maintain control of scope and costs.</span></span> <span data-ttu-id="b6d58-116">Para isso, a equipe decidiu preservar os serviços HTTP de SOAP existentes.</span><span class="sxs-lookup"><span data-stu-id="b6d58-116">To do this, they have decided to preserve their existing SOAP HTTP services.</span></span> <span data-ttu-id="b6d58-117">Ela também pretende minimizar as alterações na interface do usuário existente.</span><span class="sxs-lookup"><span data-stu-id="b6d58-117">They also intend to minimize changes to the existing UI.</span></span> <span data-ttu-id="b6d58-118">O [Gerenciamento de API do Azure (APIM)][apim] pode ser utilizado para atender muitos dos requisitos e restrições do projeto.</span><span class="sxs-lookup"><span data-stu-id="b6d58-118">[Azure API Management (APIM)][apim] can be utilized to address many of the project's requirements and constraints.</span></span>

## <a name="architecture"></a><span data-ttu-id="b6d58-119">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="b6d58-119">Architecture</span></span>

![Diagrama da arquitetura][architecture]

<span data-ttu-id="b6d58-121">A nova interface do usuário será hospedada como um aplicativo da plataforma como serviço (PaaS) no Azure e dependerá das APIs de HTTP existentes e novas.</span><span class="sxs-lookup"><span data-stu-id="b6d58-121">The new UI will be hosted as a platform as a service (PaaS) application on Azure, and will depend on both existing and new HTTP APIs.</span></span> <span data-ttu-id="b6d58-122">Essas APIs serão fornecidas com um conjunto de interfaces melhor projetado, permitindo a melhoria do desempenho, integração mais fácil e extensibilidade futura.</span><span class="sxs-lookup"><span data-stu-id="b6d58-122">These APIs will ship with a better-designed set of interfaces enabling better performance, easier integration, and future extensibility.</span></span>

### <a name="components-and-security"></a><span data-ttu-id="b6d58-123">Segurança e componentes</span><span class="sxs-lookup"><span data-stu-id="b6d58-123">Components and Security</span></span>

1. <span data-ttu-id="b6d58-124">O aplicativo web local existente continuará a consumir diretamente os serviços web locais existentes.</span><span class="sxs-lookup"><span data-stu-id="b6d58-124">The existing on-premises web application will continue to directly consume the existing on-premises web services.</span></span>
2. <span data-ttu-id="b6d58-125">As chamadas do aplicativo web existente para os serviços HTTP existentes permanecerão inalteradas.</span><span class="sxs-lookup"><span data-stu-id="b6d58-125">Calls from the existing web app to the existing HTTP services will remain unchanged.</span></span> <span data-ttu-id="b6d58-126">Essas chamadas são internas na rede corporativa.</span><span class="sxs-lookup"><span data-stu-id="b6d58-126">These calls are internal to the corporate network.</span></span>
3. <span data-ttu-id="b6d58-127">Chamadas de entrada são feitas do Azure para os serviços internos existentes:</span><span class="sxs-lookup"><span data-stu-id="b6d58-127">Inbound calls are made from Azure to the existing internal services:</span></span>
    - <span data-ttu-id="b6d58-128">A equipe de segurança permite que o tráfego da instância do APIM passe pelo firewall corporativo para os serviços locais existentes [usando o transporte seguro (HTTPS/SSL)][apim-ssl].</span><span class="sxs-lookup"><span data-stu-id="b6d58-128">The security team allows traffic from the APIM instance to pass through the corporate firewall to the existing on-premises services [using secure transport (HTTPS/SSL)][apim-ssl].</span></span>
    - <span data-ttu-id="b6d58-129">A equipe de operações permitirá as chamadas de entrada para os serviços somente da instância do APIM.</span><span class="sxs-lookup"><span data-stu-id="b6d58-129">The operations team will allow inbound calls to the services only from the APIM instance.</span></span> <span data-ttu-id="b6d58-130">Esse requisito é atendido pela [ lista de permissões do endereço IP da instância do APIM][apim-whitelist-ip] dentro do perímetro da rede corporativa.</span><span class="sxs-lookup"><span data-stu-id="b6d58-130">This requirement is met by [white-listing the IP address of the APIM instance][apim-whitelist-ip] within the corporate network perimeter.</span></span>
    - <span data-ttu-id="b6d58-131">Um novo módulo é configurado no pipeline de solicitação de serviços HTTP locais (para agir **somente** nas conexões originadas externamente), que validará [um certificado fornecido pelo APIM][apim-mutualcert-auth].</span><span class="sxs-lookup"><span data-stu-id="b6d58-131">A new module is configured into the on-premises HTTP services request pipeline (to act upon **only** those connections originating externally), which will validate [a certificate which APIM will provide][apim-mutualcert-auth].</span></span>
4. <span data-ttu-id="b6d58-132">A nova API:</span><span class="sxs-lookup"><span data-stu-id="b6d58-132">The new API:</span></span>
    - <span data-ttu-id="b6d58-133">É exibida apenas por meio da instância APIM, que fornecerá a fachada de API.</span><span class="sxs-lookup"><span data-stu-id="b6d58-133">Is surfaced only through the APIM instance, which will provide the API facade.</span></span> <span data-ttu-id="b6d58-134">A nova API não será acessada diretamente.</span><span class="sxs-lookup"><span data-stu-id="b6d58-134">The new API won't be accessed directly.</span></span>
    - <span data-ttu-id="b6d58-135">É desenvolvida e publicada como um [aplicativo de API Web de PaaS do Azure][azure-api-apps].</span><span class="sxs-lookup"><span data-stu-id="b6d58-135">Is developed and published as an [Azure PaaS Web API App][azure-api-apps].</span></span>
    - <span data-ttu-id="b6d58-136">Está na lista de permissões (via [Configurações de aplicativo Web][azure-appservice-ip-restrict]) para aceitar somente a [VIP do APIM][apim-faq-vip].</span><span class="sxs-lookup"><span data-stu-id="b6d58-136">Is white-listed (via [Web App settings][azure-appservice-ip-restrict]) to accept only the [APIM VIP][apim-faq-vip].</span></span>
    - <span data-ttu-id="b6d58-137">É hospedada em aplicativos Web do Azure com o SSL/transporte seguro ativado.</span><span class="sxs-lookup"><span data-stu-id="b6d58-137">Is hosted in Azure Web Apps with Secure Transport/SSL turned on.</span></span>
    - <span data-ttu-id="b6d58-138">Tem autorização habilitada, [fornecida pelo Serviço de Aplicativo do Azure][azure-appservice-auth] usando o Azure Active Directory e OAuth2.</span><span class="sxs-lookup"><span data-stu-id="b6d58-138">Has authorization enabled, [provided by the Azure App Service][azure-appservice-auth] using Azure Active Directory and OAuth2.</span></span>
5. <span data-ttu-id="b6d58-139">O novo aplicativo web baseado em navegador dependerá da instância de Gerenciamento de API do Azure para **ambas** as APIs HTTP, a nova e a existente.</span><span class="sxs-lookup"><span data-stu-id="b6d58-139">The new browser-based web application will depend on the Azure API Management instance for **both** the existing HTTP API and the new API.</span></span>

<span data-ttu-id="b6d58-140">A instância do APIM será configurada para mapear os serviços de HTTP herdados para um novo contrato de API.</span><span class="sxs-lookup"><span data-stu-id="b6d58-140">The APIM instance will be configured to map the legacy HTTP services to a new API contract.</span></span> <span data-ttu-id="b6d58-141">Assim, a nova interface do usuário da Web desconhece a integração com um conjunto de serviços/APIs herdadas e as novas APIs.</span><span class="sxs-lookup"><span data-stu-id="b6d58-141">By doing this, the new Web UI is unaware of the integration with a set of legacy services/APIs and new APIs.</span></span> <span data-ttu-id="b6d58-142">No futuro, a equipe de projeto transferirá gradualmente a funcionalidade de porta para as novas APIs e desativará os serviços originais.</span><span class="sxs-lookup"><span data-stu-id="b6d58-142">In the future, the project team will gradually port functionality to the new APIs and retire the original services.</span></span> <span data-ttu-id="b6d58-143">Essas alterações serão tratadas dentro da configuração do APIM, deixando a interface do usuário de front-end inalterada e evitando trabalho de redesenvolvimento.</span><span class="sxs-lookup"><span data-stu-id="b6d58-143">These changes will be handled within APIM configuration, leaving the front-end UI unaffected and avoiding redevelopment work.</span></span>

### <a name="alternatives"></a><span data-ttu-id="b6d58-144">Alternativas</span><span class="sxs-lookup"><span data-stu-id="b6d58-144">Alternatives</span></span>

- <span data-ttu-id="b6d58-145">Se a organização planeja mover totalmente sua infraestrutura para o Azure, incluindo as VMs que hospedam os aplicativos herdados, ainda assim o APIM ainda seria uma ótima opção, pois ele pode atuar como uma fachada para qualquer ponto de extremidade HTTP endereçável.</span><span class="sxs-lookup"><span data-stu-id="b6d58-145">If the organization was planning to move their infrastructure entirely to Azure, including the VMs hosting the legacy applications, then APIM would still be a great option since it can act as a facade for any addressable HTTP endpoint.</span></span>
- <span data-ttu-id="b6d58-146">Se o cliente decidiu manter os pontos de extremidade privados existentes e não mostrá-los publicamente, sua instância de Gerenciamento de API pode ser vinculada a uma [Rede Virtual do Azure (VNet)][azure-vnet]:</span><span class="sxs-lookup"><span data-stu-id="b6d58-146">If the customer had decided to keep the existing endpoints private and not expose them publicly, their API Management instance could be linked to an [Azure Virtual Network (VNet)][azure-vnet]:</span></span>
  - <span data-ttu-id="b6d58-147">Em um [cenário de comparação e deslocamento do Azure][azure-vm-lift-shift] vinculado à sua Rede Virtual do Azure implantada, o cliente pode endereçar diretamente o serviço back-end por meio de endereços IP privados.</span><span class="sxs-lookup"><span data-stu-id="b6d58-147">In an [Azure lift and shift scenario][azure-vm-lift-shift] linked to their deployed Azure Virtual Network, the customer could directly address the back-end service through private IP addresses.</span></span>
  - <span data-ttu-id="b6d58-148">No cenário local, a instância de Gerenciamento de API pode alcançar o serviço interno de modo privado por meio de um [Gateway de VPN do Azure e da conexão VPN IPSec de site a site][azure-vpn] ou do [ExpressRoute][azure-er] fazendo disto um [cenário local e híbrido do Azure][azure-hybrid].</span><span class="sxs-lookup"><span data-stu-id="b6d58-148">In the on-premises scenario, the API Management instance could reach back to the internal service privately via an [Azure VPN gateway and site-to-site IPSec VPN connection][azure-vpn] or [ExpressRoute][azure-er] making this a [hybrid Azure and on-premises scenario][azure-hybrid].</span></span>
- <span data-ttu-id="b6d58-149">A instância de gerenciamento de API pode ser mantida privada ao implantar a instância de Gerenciamento de API no modo interno.</span><span class="sxs-lookup"><span data-stu-id="b6d58-149">The API Management instance can be kept private by deploying the API Management instance in Internal mode.</span></span> <span data-ttu-id="b6d58-150">A implantação, então, pode ser usada com um [Gateway de Aplicativo do Azure] [azure-appgw] para habilitar o acesso público para algumas APIs, enquanto outras permanecem internas.</span><span class="sxs-lookup"><span data-stu-id="b6d58-150">The deployment could then be used with an [Azure Application Gateway][azure-appgw] to enable public access for some APIs while others remain internal.</span></span> <span data-ttu-id="b6d58-151">Para saber mais, veja [Conectar a APIM no modo interno para uma VNET][apim-vnet-internal].</span><span class="sxs-lookup"><span data-stu-id="b6d58-151">For more information, see [Connecting APIM in internal mode to a VNET][apim-vnet-internal].</span></span>

> [!NOTE]
> <span data-ttu-id="b6d58-152">Para obter informações gerais sobre como conectar o Gerenciamento de API a uma VNET, [confira aqui][apim-vnet].</span><span class="sxs-lookup"><span data-stu-id="b6d58-152">For general information on connecting API Management to a VNET, [see here][apim-vnet].</span></span>

### <a name="availability-and-scalability"></a><span data-ttu-id="b6d58-153">Disponibilidade e escalabilidade</span><span class="sxs-lookup"><span data-stu-id="b6d58-153">Availability and scalability</span></span>

- <span data-ttu-id="b6d58-154">O Gerenciamento de API do Azure pode ser [escalado horizontalmente][apim-scaleout] Como escolher um tipo de preço e, em seguida, adicionando as unidades.</span><span class="sxs-lookup"><span data-stu-id="b6d58-154">Azure API Management can be [scaled out][apim-scaleout] by choosing a pricing tier and then adding units.</span></span>
- <span data-ttu-id="b6d58-155">A escalabilidade também acontece [automaticamente com o dimensionamento automático][apim-autoscale].</span><span class="sxs-lookup"><span data-stu-id="b6d58-155">Scaling also happen [automatically with auto scaling][apim-autoscale].</span></span>
- <span data-ttu-id="b6d58-156">A [implantação em várias regiões][apim-multi-regions] permitirá opções de failover e pode ser realizada na [camada Premium][apim-pricing].</span><span class="sxs-lookup"><span data-stu-id="b6d58-156">[Deploying across multiple regions][apim-multi-regions] will enable fail over options and can be done in the [Premium tier][apim-pricing].</span></span>
- <span data-ttu-id="b6d58-157">Considere a [Integração com o Azure Application Insights][azure-apim-ai], que também exibe as métricas para monitoramento por meio do [Azure Monitor][azure-mon].</span><span class="sxs-lookup"><span data-stu-id="b6d58-157">Consider [Integrating with Azure Application Insights][azure-apim-ai], which also surfaces metrics through [Azure Monitor][azure-mon] for monitoring.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="b6d58-158">Implantar o cenário</span><span class="sxs-lookup"><span data-stu-id="b6d58-158">Deploy the scenario</span></span>

<span data-ttu-id="b6d58-159">Para começar, [crie uma instância de Gerenciamento de API do Azure no portal.][apim-create]</span><span class="sxs-lookup"><span data-stu-id="b6d58-159">To get started, [create an Azure API Management instance in the portal.][apim-create]</span></span>

<span data-ttu-id="b6d58-160">Como alternativa, é possível escolher um [modelo de início rápido][azure-quickstart-templates-apim] existente do Azure Resource Manager que se encaixa no seu caso de uso específico.</span><span class="sxs-lookup"><span data-stu-id="b6d58-160">Alternatively, you can choose from an existing Azure Resource Manager [quickstart template][azure-quickstart-templates-apim] that aligns to your specific use case.</span></span>

## <a name="pricing"></a><span data-ttu-id="b6d58-161">Preços</span><span class="sxs-lookup"><span data-stu-id="b6d58-161">Pricing</span></span>

<span data-ttu-id="b6d58-162">O Gerenciamento de API é fornecido em quatro camadas: desenvolvedor, básica, standard e premium.</span><span class="sxs-lookup"><span data-stu-id="b6d58-162">API Management is offered in four tiers: developer, basic, standard, and premium.</span></span> <span data-ttu-id="b6d58-163">Você pode encontrar orientações detalhadas sobre a diferença dos níveis [aqui nas diretrizes de preços de Gerenciamento de API do Azure.][apim-pricing]</span><span class="sxs-lookup"><span data-stu-id="b6d58-163">You can find detailed guidance on the difference in these tiers at the [Azure API Management pricing guidance here.][apim-pricing]</span></span>

<span data-ttu-id="b6d58-164">Os clientes podem escalonar o Gerenciamento de API adicionando e removendo unidades.</span><span class="sxs-lookup"><span data-stu-id="b6d58-164">Customers can scale API Management by adding and removing units.</span></span> <span data-ttu-id="b6d58-165">Cada unidade tem capacidades que dependem de sua camada.</span><span class="sxs-lookup"><span data-stu-id="b6d58-165">Each unit has capacity that depends on its tier.</span></span>

> [!NOTE]
> <span data-ttu-id="b6d58-166">A camada de Desenvolvedor pode ser usada para avaliação dos recursos de Gerenciamento de API.</span><span class="sxs-lookup"><span data-stu-id="b6d58-166">The Developer tier can be used for evaluation of the API Management features.</span></span> <span data-ttu-id="b6d58-167">A camada de desenvolvedor não deve ser usada para produção.</span><span class="sxs-lookup"><span data-stu-id="b6d58-167">The Developer tier should not be used for production.</span></span>

<span data-ttu-id="b6d58-168">Para exibir os custos projetados e personalizar as suas necessidades de implantação, você pode modificar o número de unidades de escala e de instâncias de Serviço do Aplicativo na [Calculadora de Preços do Azure][pricing-calculator].</span><span class="sxs-lookup"><span data-stu-id="b6d58-168">To view projected costs and customize to your deployment needs, you can modify the number of scale units and App Service instances in the [Azure Pricing Calculator][pricing-calculator].</span></span>

## <a name="related-resources"></a><span data-ttu-id="b6d58-169">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="b6d58-169">Related resources</span></span>

<span data-ttu-id="b6d58-170">Revise a extensa [documentação e os artigos de referência][apim] sobre Gerenciamento de API do Azure.</span><span class="sxs-lookup"><span data-stu-id="b6d58-170">Review the extensive Azure API Management [documentation and reference articles][apim].</span></span>

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
