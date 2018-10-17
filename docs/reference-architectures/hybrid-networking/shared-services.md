---
title: Implementação de uma topologia de rede hub-spoke com serviços compartilhados no Azure
description: Como implementar uma topologia de rede hub-spoke com serviços compartilhados no Azure.
author: telmosampaio
ms.date: 06/19/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: 0238c5d6f28bacbc32268d4586b30395de36384b
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876861"
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="71969-103">Implementar uma topologia de rede hub-spoke com serviços compartilhados no Azure</span><span class="sxs-lookup"><span data-stu-id="71969-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="71969-104">Essa arquitetura de referência se baseia na arquitetura de referência do [hub-spoke] [ guidance-hub-spoke] para incluir serviços compartilhados no hub que pode ser consumido por todos os spokes.</span><span class="sxs-lookup"><span data-stu-id="71969-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="71969-105">Como uma primeira etapa para migrar um datacenter para a nuvem, e criando um [datacenter virtual], os primeiros serviços que você precisa compartilhar são identidade e segurança.</span><span class="sxs-lookup"><span data-stu-id="71969-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="71969-106">Esta arquitetura de referência mostra como estender seus serviços do Active Directory de seu datacenter local para o do Azure e como adicionar uma NVA (solução de virtualização de rede), que pode agir como um firewall, em uma topologia hub-spoke.</span><span class="sxs-lookup"><span data-stu-id="71969-106">This reference architecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="71969-107">[**Implantar esta solução**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="71969-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="71969-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="71969-108">![[0]][0]</span></span>

<span data-ttu-id="71969-109">*Baixar um [arquivo Visio][visio-download] dessa arquitetura*</span><span class="sxs-lookup"><span data-stu-id="71969-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="71969-110">Os benefícios dessa topologia incluem:</span><span class="sxs-lookup"><span data-stu-id="71969-110">The benefits of this topology include:</span></span>

* <span data-ttu-id="71969-111">**Redução de custos** com a centralização de serviços que podem ser compartilhados por diversas cargas de trabalho, tais como NVAs (soluções de virtualização de rede) e servidores DNS, em um único local.</span><span class="sxs-lookup"><span data-stu-id="71969-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="71969-112">**Superar os limites de assinaturas** emparelhando VNets de assinaturas diferentes com o hub central.</span><span class="sxs-lookup"><span data-stu-id="71969-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="71969-113">**Separação de preocupações** entre TI central (SecOps, InfraOps) e cargas de trabalho (DevOps).</span><span class="sxs-lookup"><span data-stu-id="71969-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="71969-114">Alguns usos típicos dessa arquitetura:</span><span class="sxs-lookup"><span data-stu-id="71969-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="71969-115">As cargas de trabalho implantadas em diferentes ambientes, como desenvolvimento, teste e produção, que exigem serviços compartilhados como DNS, IDS, NTP ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="71969-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="71969-116">Os serviços compartilhados são colocados na VNet do hub, enquanto cada ambiente é implantado em um spoke para manter o isolamento.</span><span class="sxs-lookup"><span data-stu-id="71969-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="71969-117">Cargas de trabalho que não exigem conectividade entre si, mas precisam de acesso aos serviços compartilhados.</span><span class="sxs-lookup"><span data-stu-id="71969-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="71969-118">Empresas que exigem o controle centralizado sobre aspectos de segurança, como um firewall no hub como uma DMZ e gerenciamento separado para cargas de trabalho em cada spoke.</span><span class="sxs-lookup"><span data-stu-id="71969-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="71969-119">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="71969-119">Architecture</span></span>

<span data-ttu-id="71969-120">A arquitetura consiste nos componentes a seguir.</span><span class="sxs-lookup"><span data-stu-id="71969-120">The architecture consists of the following components.</span></span>

* <span data-ttu-id="71969-121">**Rede local**.</span><span class="sxs-lookup"><span data-stu-id="71969-121">**On-premises network**.</span></span> <span data-ttu-id="71969-122">Uma rede de área local privada em execução dentro de uma organização.</span><span class="sxs-lookup"><span data-stu-id="71969-122">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="71969-123">**Dispositivo VPN**.</span><span class="sxs-lookup"><span data-stu-id="71969-123">**VPN device**.</span></span> <span data-ttu-id="71969-124">Um dispositivo ou serviço que fornece conectividade externa para a rede local.</span><span class="sxs-lookup"><span data-stu-id="71969-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="71969-125">O dispositivo VPN pode ser um dispositivo de hardware ou uma solução de software, como o RRAS (Serviço de Acesso Remoto e Roteamento) do Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="71969-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="71969-126">Para obter uma lista de dispositivos de VPN com suporte e informações sobre como configurar dispositivos de VPN selecionados para se conectar ao Azure, consulte [Sobre dispositivos VPN para conexões de Gateway de VPN Site a Site][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="71969-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="71969-127">**Gateway de rede virtual de VPN ou gateway de ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="71969-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="71969-128">O gateway de rede virtual permite que a VNet se conecte ao dispositivo VPN ou ao circuito de ExpressRoute, usado para conectividade com a rede local.</span><span class="sxs-lookup"><span data-stu-id="71969-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="71969-129">Para obter mais informações, consulte [Conectar uma rede local a uma rede virtual do Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="71969-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="71969-130">Os scripts de implantação dessa arquitetura de referência usam um gateway VPN para conectividade e uma VNet no Azure para simular a rede local.</span><span class="sxs-lookup"><span data-stu-id="71969-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="71969-131">**VNet do hub**.</span><span class="sxs-lookup"><span data-stu-id="71969-131">**Hub VNet**.</span></span> <span data-ttu-id="71969-132">VNet do Azure usada como o hub na topologia hub-spoke.</span><span class="sxs-lookup"><span data-stu-id="71969-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="71969-133">O hub é o ponto central de conectividade para sua rede local e um local para hospedar os serviços que podem ser consumidos por diferentes cargas de trabalho hospedadas nas VNets do spoke.</span><span class="sxs-lookup"><span data-stu-id="71969-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="71969-134">**Sub-rede do gateway**.</span><span class="sxs-lookup"><span data-stu-id="71969-134">**Gateway subnet**.</span></span> <span data-ttu-id="71969-135">Os gateways de rede virtual são mantidos na mesma sub-rede.</span><span class="sxs-lookup"><span data-stu-id="71969-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="71969-136">**Sub-rede serviços compartilhados**.</span><span class="sxs-lookup"><span data-stu-id="71969-136">**Shared services subnet**.</span></span> <span data-ttu-id="71969-137">Uma sub-rede na VNet do hub usada para hospedar os serviços que podem ser compartilhados entre todos os spokes, como DNS ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="71969-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="71969-138">**Sub-rede de perímetro**.</span><span class="sxs-lookup"><span data-stu-id="71969-138">**DMZ subnet**.</span></span> <span data-ttu-id="71969-139">Uma sub-rede na rede virtual do hub usada para hospedar NVAs que podem agir como dispositivos de segurança, como firewalls.</span><span class="sxs-lookup"><span data-stu-id="71969-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

* <span data-ttu-id="71969-140">**VNets de spoke**.</span><span class="sxs-lookup"><span data-stu-id="71969-140">**Spoke VNets**.</span></span> <span data-ttu-id="71969-141">Uma ou mais VNets do Azure que são usadas como spokes na topologia hub-spoke.</span><span class="sxs-lookup"><span data-stu-id="71969-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="71969-142">Spokes podem ser usados para isolar as cargas de trabalho em suas próprias VNets, gerenciadas separadamente de outros spokes.</span><span class="sxs-lookup"><span data-stu-id="71969-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="71969-143">Cada carga de trabalho pode incluir várias camadas, com várias sub-redes conectadas por meio de balanceadores de carga do Azure.</span><span class="sxs-lookup"><span data-stu-id="71969-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="71969-144">Para obter mais informações sobre a infraestrutura do aplicativo, consulte [Execução de cargas de trabalho de VM do Windows][windows-vm-ra] e [Execução de cargas de trabalho de VM do Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="71969-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="71969-145">**Emparelhamento VNet**.</span><span class="sxs-lookup"><span data-stu-id="71969-145">**VNet peering**.</span></span> <span data-ttu-id="71969-146">Duas VNets na mesma região do Azure podem ser conectadas usando uma [conexão de emparelhamento][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="71969-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="71969-147">Conexões de emparelhamento são conexões não transitivas de baixa latência entre VNets.</span><span class="sxs-lookup"><span data-stu-id="71969-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="71969-148">Quando emparelhadas, as VNets trocam tráfego usando o backbone do Azure sem precisar de um roteador.</span><span class="sxs-lookup"><span data-stu-id="71969-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="71969-149">Em uma topologia de rede hub-spoke, é necessário usar o emparelhamento VNet para conectar o hub a cada spoke.</span><span class="sxs-lookup"><span data-stu-id="71969-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="71969-150">Este artigo aborda apenas implantações do [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mas também é possível conectar uma VNet clássica a uma VNet do Resource Manager na mesma assinatura.</span><span class="sxs-lookup"><span data-stu-id="71969-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="71969-151">Dessa forma, seus spokes podem hospedar implantações clássicas e ainda aproveitar os serviços compartilhados no hub.</span><span class="sxs-lookup"><span data-stu-id="71969-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="71969-152">Recomendações</span><span class="sxs-lookup"><span data-stu-id="71969-152">Recommendations</span></span>

<span data-ttu-id="71969-153">Todas as recomendações para a arquitetura de referência do [hub-spoke] [ guidance-hub-spoke] também se aplicam à arquitetura de referência de serviços compartilhados.</span><span class="sxs-lookup"><span data-stu-id="71969-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span> 

<span data-ttu-id="71969-154">Além disso, as seguintes recomendações se aplicam para a maioria dos cenários em serviços compartilhados.</span><span class="sxs-lookup"><span data-stu-id="71969-154">Also, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="71969-155">Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.</span><span class="sxs-lookup"><span data-stu-id="71969-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="71969-156">Identidade</span><span class="sxs-lookup"><span data-stu-id="71969-156">Identity</span></span>

<span data-ttu-id="71969-157">A maioria das empresas tem um ambiente de Active Directory Domain Services (ADDS) em seu datacenter local.</span><span class="sxs-lookup"><span data-stu-id="71969-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="71969-158">Para facilitar o gerenciamento de ativos movidos para o Azure de sua rede local que dependem do ADDS, é recomendável hospedar os controladores de domínio ADDS no Azure.</span><span class="sxs-lookup"><span data-stu-id="71969-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="71969-159">Se você usar objetos da Política de Grupo que você deseja controlar separadamente para o Azure e seu ambiente local, use um site diferente do AD para cada região do Azure.</span><span class="sxs-lookup"><span data-stu-id="71969-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="71969-160">Coloque os controladores de domínio em uma rede virtual central (hub) que pode ser acessada por cargas de trabalho dependentes.</span><span class="sxs-lookup"><span data-stu-id="71969-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="71969-161">Segurança</span><span class="sxs-lookup"><span data-stu-id="71969-161">Security</span></span>

<span data-ttu-id="71969-162">Ao mover cargas de trabalho do seu ambiente local para o Azure, algumas dessas cargas de trabalho exigem ser hospedadas em máquinas virtuais.</span><span class="sxs-lookup"><span data-stu-id="71969-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="71969-163">Por motivos de conformidade, talvez seja necessário impor restrições sobre o tráfego que passa por essas cargas de trabalho.</span><span class="sxs-lookup"><span data-stu-id="71969-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span> 

<span data-ttu-id="71969-164">Você pode usar soluções de virtualização de rede (NVAs) no Azure para hospedar tipos diferentes de segurança e de serviços de desempenho.</span><span class="sxs-lookup"><span data-stu-id="71969-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="71969-165">Se você estiver atualmente familiarizado com um determinado conjunto de dispositivos locais, é recomendável usar os mesmos dispositivos virtualizados no Azure, onde aplicável.</span><span class="sxs-lookup"><span data-stu-id="71969-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="71969-166">Os scripts de implantação para esta arquitetura de referência usam uma VM Ubuntu com encaminhamento de IP ativado para simular uma solução de virtualização de rede.</span><span class="sxs-lookup"><span data-stu-id="71969-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enabled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="71969-167">Considerações</span><span class="sxs-lookup"><span data-stu-id="71969-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="71969-168">Superar os limites de emparelhamento VNet</span><span class="sxs-lookup"><span data-stu-id="71969-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="71969-169">Não deixe de considerar a [limitação do número de emparelhamentos de VNet por VNet][vnet-peering-limit] no Azure.</span><span class="sxs-lookup"><span data-stu-id="71969-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="71969-170">Se você precisar de mais spokes do que o permitido pelo limite, considere criar uma topologia hub-spoke-hub-spoke, na qual o primeiro nível de spokes também funciona como hubs.</span><span class="sxs-lookup"><span data-stu-id="71969-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="71969-171">O diagrama a seguir mostra essa abordagem.</span><span class="sxs-lookup"><span data-stu-id="71969-171">The following diagram shows this approach.</span></span>

<span data-ttu-id="71969-172">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="71969-172">![[3]][3]</span></span>

<span data-ttu-id="71969-173">Além disso, considere quais serviços compartilhados no hub, para garantir que o hub seja dimensionado para um número maior de spokes.</span><span class="sxs-lookup"><span data-stu-id="71969-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="71969-174">Por exemplo, se seu hub fornecer serviços de firewall, considere os limites de largura de banda da sua solução de firewall ao adicionar vários spokes.</span><span class="sxs-lookup"><span data-stu-id="71969-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="71969-175">Pode ser útil mover alguns desses serviços compartilhados para um segundo nível de hubs.</span><span class="sxs-lookup"><span data-stu-id="71969-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="71969-176">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="71969-176">Deploy the solution</span></span>

<span data-ttu-id="71969-177">Uma implantação para essa arquitetura está disponível no [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="71969-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="71969-178">A implantação cria os seguintes grupos de recursos em sua assinatura:</span><span class="sxs-lookup"><span data-stu-id="71969-178">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="71969-179">hub-adds-rg</span><span class="sxs-lookup"><span data-stu-id="71969-179">hub-adds-rg</span></span>
- <span data-ttu-id="71969-180">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="71969-180">hub-nva-rg</span></span>
- <span data-ttu-id="71969-181">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="71969-181">hub-vnet-rg</span></span>
- <span data-ttu-id="71969-182">onprem-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="71969-182">onprem-vnet-rg</span></span>
- <span data-ttu-id="71969-183">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="71969-183">spoke1-vnet-rg</span></span>
- <span data-ttu-id="71969-184">spoke2-vent-rg</span><span class="sxs-lookup"><span data-stu-id="71969-184">spoke2-vent-rg</span></span>

<span data-ttu-id="71969-185">Os arquivos de parâmetros de modelo se referem a esses nomes e, portanto, se você alterá-los, atualize os arquivos de parâmetros para que correspondam.</span><span class="sxs-lookup"><span data-stu-id="71969-185">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="71969-186">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="71969-186">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="71969-187">Implantar o datacenter local simulado usand azbb</span><span class="sxs-lookup"><span data-stu-id="71969-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="71969-188">Esta etapa implanta o datacenter local simulado como uma rede virtual do Azure.</span><span class="sxs-lookup"><span data-stu-id="71969-188">This step deploys the simulated on-premises datacenter as an Azure VNet.</span></span>

1. <span data-ttu-id="71969-189">Navegue até a pasta `hybrid-networking\shared-services-stack\` do repositório do GitHub.</span><span class="sxs-lookup"><span data-stu-id="71969-189">Navigate to the `hybrid-networking\shared-services-stack\` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="71969-190">Abra o arquivo `onprem.json` .</span><span class="sxs-lookup"><span data-stu-id="71969-190">Open the `onprem.json` file.</span></span> 

3. <span data-ttu-id="71969-191">Pesquise todas as instâncias de `UserName`, `adminUserName`, `Password` e `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="71969-191">Search for all instances of `UserName`, `adminUserName`,`Password`, and `adminPassword`.</span></span> <span data-ttu-id="71969-192">Insira valores para o nome de usuário e a senha nos parâmetros e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="71969-192">Enter values for the user name and password in the parameters and save the file.</span></span> 

4. <span data-ttu-id="71969-193">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="71969-193">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
   ```
5. <span data-ttu-id="71969-194">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="71969-194">Wait for the deployment to finish.</span></span> <span data-ttu-id="71969-195">Essa implantação cria uma rede virtual, uma máquina virtual que executa Windows e um gateway de VPN.</span><span class="sxs-lookup"><span data-stu-id="71969-195">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="71969-196">A criação do gateway de VPN pode levar mais de 40 minutos para ser concluída.</span><span class="sxs-lookup"><span data-stu-id="71969-196">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="71969-197">Implantar a VNET do hub</span><span class="sxs-lookup"><span data-stu-id="71969-197">Deploy the hub VNet</span></span>

<span data-ttu-id="71969-198">Esta etapa implanta o hub de rede virtual e conecta-se à rede virtual local simulada.</span><span class="sxs-lookup"><span data-stu-id="71969-198">This step deploys the hub VNet and connects it to the simulated on-premises VNet.</span></span>

1. <span data-ttu-id="71969-199">Abra o arquivo `hub-vnet.json` .</span><span class="sxs-lookup"><span data-stu-id="71969-199">Open the `hub-vnet.json` file.</span></span> 

2. <span data-ttu-id="71969-200">Pesquise `adminPassword` e insira um nome de usuário e senha nos parâmetros.</span><span class="sxs-lookup"><span data-stu-id="71969-200">Search for `adminPassword` and enter a user name and password in the parameters.</span></span> 

3. <span data-ttu-id="71969-201">Pesquise todas as instâncias de `sharedKey` e insira um valor para uma chave compartilhada.</span><span class="sxs-lookup"><span data-stu-id="71969-201">Search for all instances of `sharedKey` and enter a value for a shared key.</span></span> <span data-ttu-id="71969-202">Salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="71969-202">Save the file.</span></span>

   ```bash
   "sharedKey": "abc123",
   ```

4. <span data-ttu-id="71969-203">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="71969-203">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
   ```

5. <span data-ttu-id="71969-204">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="71969-204">Wait for the deployment to finish.</span></span> <span data-ttu-id="71969-205">Essa implantação cria uma rede virtual, uma máquina virtual, um gateway de VPN e uma conexão com o gateway criado na seção anterior.</span><span class="sxs-lookup"><span data-stu-id="71969-205">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="71969-206">O gateway de VPN pode levar mais de 40 minutos para ser concluído.</span><span class="sxs-lookup"><span data-stu-id="71969-206">The VPN gateway can take more than 40 minutes to complete.</span></span>

### <a name="deploy-ad-ds-in-azure"></a><span data-ttu-id="71969-207">Implantar o Active Directory Domain Services no Azure</span><span class="sxs-lookup"><span data-stu-id="71969-207">Deploy AD DS in Azure</span></span>

<span data-ttu-id="71969-208">Esta etapa implanta os controladores de domínio do Active Directory Domain Services no Azure.</span><span class="sxs-lookup"><span data-stu-id="71969-208">This step deploys AD DS domain controllers in Azure.</span></span>

1. <span data-ttu-id="71969-209">Abra o arquivo `hub-adds.json` .</span><span class="sxs-lookup"><span data-stu-id="71969-209">Open the `hub-adds.json` file.</span></span>

2. <span data-ttu-id="71969-210">Pesquise todas as instâncias de `Password` e `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="71969-210">Search for all instances of `Password` and `adminPassword`.</span></span> <span data-ttu-id="71969-211">Insira valores para o nome de usuário e a senha nos parâmetros e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="71969-211">Enter values for the user name and password in the parameters and save the file.</span></span> 

3. <span data-ttu-id="71969-212">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="71969-212">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg -l <location> -p hub-adds.json --deploy
   ```
  
<span data-ttu-id="71969-213">Esta etapa de implantação poderá levar vários minutos, porque adiciona as duas máquinas virtuais ao domínio hospedado no datacenter simulado local e instala o AD DS neles.</span><span class="sxs-lookup"><span data-stu-id="71969-213">This deployment step may take several minutes, because it joins the two VMs to the domain hosted in the simulated on-premises datacenter, and installs AD DS on them.</span></span>

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="71969-214">Implantar as redes virtuais spoke</span><span class="sxs-lookup"><span data-stu-id="71969-214">Deploy the spoke VNets</span></span>

<span data-ttu-id="71969-215">Esta etapa implanta as redes virtuais spoke.</span><span class="sxs-lookup"><span data-stu-id="71969-215">This step deploys the spoke VNets.</span></span>

1. <span data-ttu-id="71969-216">Abra o arquivo `spoke1.json` .</span><span class="sxs-lookup"><span data-stu-id="71969-216">Open the `spoke1.json` file.</span></span>

2. <span data-ttu-id="71969-217">Pesquise `adminPassword` e insira um nome de usuário e senha nos parâmetros.</span><span class="sxs-lookup"><span data-stu-id="71969-217">Search for `adminPassword` and enter a user name and password in the parameters.</span></span> 

3. <span data-ttu-id="71969-218">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="71969-218">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. <span data-ttu-id="71969-219">Repita as etapas 1 e 2 para o arquivo `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="71969-219">Repeat steps 1 and 2 for the file `spoke2.json`.</span></span>

5. <span data-ttu-id="71969-220">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="71969-220">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

### <a name="peer-the-hub-vnet-to-the-spoke-vnets"></a><span data-ttu-id="71969-221">Nivelar a rede virtual do hub com as redes virtuais spoke</span><span class="sxs-lookup"><span data-stu-id="71969-221">Peer the hub VNet to the spoke VNets</span></span>

<span data-ttu-id="71969-222">Para criar uma conexão de emparelhamento da rede virtual do hub para as redes virtuais spoke, execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="71969-222">To create a peering connection from the hub VNet to the spoke VNets, run the following command:</span></span>

```bash
azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
```

### <a name="deploy-the-nva"></a><span data-ttu-id="71969-223">Implantar a NVA</span><span class="sxs-lookup"><span data-stu-id="71969-223">Deploy the NVA</span></span>

<span data-ttu-id="71969-224">Esta etapa implanta uma NVA na sub-rede `dmz`.</span><span class="sxs-lookup"><span data-stu-id="71969-224">This step deploys an NVA in the `dmz` subnet.</span></span>

1. <span data-ttu-id="71969-225">Abra o arquivo `hub-nva.json` .</span><span class="sxs-lookup"><span data-stu-id="71969-225">Open the `hub-nva.json` file.</span></span>

2. <span data-ttu-id="71969-226">Pesquise `adminPassword` e insira um nome de usuário e senha nos parâmetros.</span><span class="sxs-lookup"><span data-stu-id="71969-226">Search for `adminPassword` and enter a user name and password in the parameters.</span></span> 

3. <span data-ttu-id="71969-227">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="71969-227">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="71969-228">Testar a conectividade</span><span class="sxs-lookup"><span data-stu-id="71969-228">Test connectivity</span></span> 

<span data-ttu-id="71969-229">Testar a conexão do ambiente local simulado para a VNET do hub.</span><span class="sxs-lookup"><span data-stu-id="71969-229">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

1. <span data-ttu-id="71969-230">Usar o portal do Azure para encontrar a VM denominada `jb-vm1` no grupo de recursos `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="71969-230">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="71969-231">Clique em `Connect` para abrir uma sessão de área de trabalho remota para a VM.</span><span class="sxs-lookup"><span data-stu-id="71969-231">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="71969-232">Use a senha que você especificou no arquivo de parâmetro `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="71969-232">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="71969-233">Abra um console do PowerShell na VM e use o cmdlet `Test-NetConnection` para verificar se você pode se conectar à VM jumpbox na Vnet do hub.</span><span class="sxs-lookup"><span data-stu-id="71969-233">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
<span data-ttu-id="71969-234">O resultado deve ser semelhante ao seguinte:</span><span class="sxs-lookup"><span data-stu-id="71969-234">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="71969-235">Por padrão, as máquinas virtuais do Windows Server não permitem respostas do ICMP no Azure.</span><span class="sxs-lookup"><span data-stu-id="71969-235">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="71969-236">Se você quiser usar `ping` para testar a conectividade, você precisa habilitar o tráfego ICMP no Firewall Avançado do Windows para cada VM.</span><span class="sxs-lookup"><span data-stu-id="71969-236">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="71969-237">Repita as mesmas etapas para testar a conectividade com as redes virtuais spoke:</span><span class="sxs-lookup"><span data-stu-id="71969-237">Repeat the sames steps to test connectivity to the spoke VNets:</span></span>

```powershell
Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
```


<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[guidance-hub-spoke]: ./hub-spoke.md
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[datacenter virtual]: https://aka.ms/vdc
[virtual datacenter]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Topologia de serviços compartilhados no Azure"
[3]: ./images/hub-spokehub-spoke.svg "Topologia hub-spoke-hub-spoke no Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
