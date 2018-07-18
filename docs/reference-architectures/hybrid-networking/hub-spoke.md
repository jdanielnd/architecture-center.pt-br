---
title: Implementação de uma topologia de rede hub-spoke no Azure
description: Como implementar uma topologia de rede hub-spoke no Azure.
author: telmosampaio
ms.date: 04/09/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 9105748f434e5d655b09b1fe0775417f33a912b0
ms.sourcegitcommit: f7fa67e3bdbc57d368edb67bac0e1fdec63695d2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/05/2018
ms.locfileid: "37843585"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="2fb8a-103">Implementar uma topologia de rede hub-spoke no Azure</span><span class="sxs-lookup"><span data-stu-id="2fb8a-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="2fb8a-104">Essa arquitetura de referência mostra como implementar uma topologia hub-spoke no Azure.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="2fb8a-105">O *hub* é uma VNet (rede virtual) no Azure que atua como ponto central de conectividade para sua rede local.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="2fb8a-106">Os *spokes* são VNets que se emparelham com o hub e podem ser usadas para isolar as cargas de trabalho.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="2fb8a-107">Fluxos de tráfego entre o datacenter local e o hub por meio de uma conexão de ExpressRoute ou gateway de VPN.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="2fb8a-108">[**Implantar esta solução**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="2fb8a-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="2fb8a-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="2fb8a-109">![[0]][0]</span></span>

<span data-ttu-id="2fb8a-110">*Baixar um [arquivo Visio][visio-download] dessa arquitetura*</span><span class="sxs-lookup"><span data-stu-id="2fb8a-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="2fb8a-111">Os benefícios dessa topologia incluem:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="2fb8a-112">**Redução de custos** com a centralização de serviços que podem ser compartilhados por diversas cargas de trabalho, tais como NVAs (soluções de virtualização de rede) e servidores DNS, em um único local.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="2fb8a-113">**Superar os limites de assinaturas** emparelhando VNets de assinaturas diferentes com o hub central.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="2fb8a-114">**Separação de preocupações** entre TI central (SecOps, InfraOps) e cargas de trabalho (DevOps).</span><span class="sxs-lookup"><span data-stu-id="2fb8a-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="2fb8a-115">Alguns usos típicos dessa arquitetura:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="2fb8a-116">As cargas de trabalho implantadas em diferentes ambientes, como desenvolvimento, teste e produção, que exigem serviços compartilhados como DNS, IDS, NTP ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="2fb8a-117">Os serviços compartilhados são colocados na VNet do hub, enquanto cada ambiente é implantado em um spoke para manter o isolamento.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="2fb8a-118">Cargas de trabalho que não exigem conectividade entre si, mas precisam de acesso aos serviços compartilhados.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="2fb8a-119">Empresas que exigem o controle centralizado sobre aspectos de segurança, como um firewall no hub como uma DMZ e gerenciamento separado para cargas de trabalho em cada spoke.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="2fb8a-120">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="2fb8a-120">Architecture</span></span>

<span data-ttu-id="2fb8a-121">A arquitetura consiste nos componentes a seguir.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="2fb8a-122">**Rede local**.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-122">**On-premises network**.</span></span> <span data-ttu-id="2fb8a-123">Uma rede de área local privada em execução dentro de uma organização.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="2fb8a-124">**Dispositivo VPN**.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-124">**VPN device**.</span></span> <span data-ttu-id="2fb8a-125">Um dispositivo ou serviço que fornece conectividade externa para a rede local.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="2fb8a-126">O dispositivo VPN pode ser um dispositivo de hardware ou uma solução de software, como o RRAS (Serviço de Acesso Remoto e Roteamento) do Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="2fb8a-127">Para obter uma lista de dispositivos de VPN com suporte e informações sobre como configurar dispositivos de VPN selecionados para se conectar ao Azure, consulte [Sobre dispositivos VPN para conexões de Gateway de VPN Site a Site][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="2fb8a-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="2fb8a-128">**Gateway de rede virtual de VPN ou gateway de ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="2fb8a-129">O gateway de rede virtual permite que a VNet se conecte ao dispositivo VPN ou ao circuito de ExpressRoute, usado para conectividade com a rede local.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="2fb8a-130">Para obter mais informações, consulte [Conectar uma rede local a uma rede virtual do Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="2fb8a-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="2fb8a-131">Os scripts de implantação dessa arquitetura de referência usam um gateway VPN para conectividade e uma VNet no Azure para simular a rede local.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="2fb8a-132">**VNet do hub**.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-132">**Hub VNet**.</span></span> <span data-ttu-id="2fb8a-133">VNet do Azure usada como o hub na topologia hub-spoke.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="2fb8a-134">O hub é o ponto central de conectividade para sua rede local e um local para hospedar os serviços que podem ser consumidos por diferentes cargas de trabalho hospedadas nas VNets do spoke.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="2fb8a-135">**Sub-rede do gateway**.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-135">**Gateway subnet**.</span></span> <span data-ttu-id="2fb8a-136">Os gateways de rede virtual são mantidos na mesma sub-rede.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="2fb8a-137">**VNets de spoke**.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-137">**Spoke VNets**.</span></span> <span data-ttu-id="2fb8a-138">Uma ou mais VNets do Azure que são usadas como spokes na topologia hub-spoke.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="2fb8a-139">Spokes podem ser usados para isolar as cargas de trabalho em suas próprias VNets, gerenciadas separadamente de outros spokes.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="2fb8a-140">Cada carga de trabalho pode incluir várias camadas, com várias sub-redes conectadas por meio de balanceadores de carga do Azure.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="2fb8a-141">Para obter mais informações sobre a infraestrutura do aplicativo, consulte [Execução de cargas de trabalho de VM do Windows][windows-vm-ra] e [Execução de cargas de trabalho de VM do Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="2fb8a-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="2fb8a-142">**Emparelhamento VNet**.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-142">**VNet peering**.</span></span> <span data-ttu-id="2fb8a-143">Duas VNets na mesma região do Azure podem ser conectadas usando uma [conexão de emparelhamento][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="2fb8a-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="2fb8a-144">Conexões de emparelhamento são conexões não transitivas de baixa latência entre VNets.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="2fb8a-145">Quando emparelhadas, as VNets trocam tráfego usando o backbone do Azure sem precisar de um roteador.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="2fb8a-146">Em uma topologia de rede hub-spoke, é necessário usar o emparelhamento VNet para conectar o hub a cada spoke.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="2fb8a-147">Este artigo aborda apenas implantações do [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mas também é possível conectar uma VNet clássica a uma VNet do Resource Manager na mesma assinatura.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="2fb8a-148">Dessa forma, seus spokes podem hospedar implantações clássicas e ainda aproveitar os serviços compartilhados no hub.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="2fb8a-149">Recomendações</span><span class="sxs-lookup"><span data-stu-id="2fb8a-149">Recommendations</span></span>

<span data-ttu-id="2fb8a-150">As seguintes recomendações aplicam-se à maioria dos cenários.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="2fb8a-151">Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="2fb8a-152">Grupos de recursos</span><span class="sxs-lookup"><span data-stu-id="2fb8a-152">Resource groups</span></span>

<span data-ttu-id="2fb8a-153">A VNet do hub e a VNet de cada spoke, podem ser implementadas em diferentes grupos de recursos, e até mesmo em assinaturas diferentes, contanto que elas pertençam ao mesmo locatário do Azure AD (Azure Active Directory) na mesma região do Azure.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="2fb8a-154">Isso possibilita o gerenciamento descentralizado de cada carga de trabalho, compartilhando os serviços mantidos na VNet do hub.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="2fb8a-155">VNet e GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="2fb8a-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="2fb8a-156">Crie uma sub-rede denominada *GatewaySubnet* com um intervalo de endereços de /27.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="2fb8a-157">Essa sub-rede é necessária para o gateway de rede virtual.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="2fb8a-158">Alocar 32 endereços para esta sub-rede ajudará a evitar que as limitações de tamanho de gateway sejam atingidas no futuro.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="2fb8a-159">Para obter mais informações sobre como configurar o gateway, consulte as seguintes arquiteturas de referência, dependendo do seu tipo de conexão:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="2fb8a-160">[Rede híbrida usando o ExpressRoute][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="2fb8a-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="2fb8a-161">[Rede híbrida usando um gateway de VPN][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="2fb8a-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="2fb8a-162">Para maior disponibilidade, você pode usar o ExpressRoute e uma VPN para failover.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="2fb8a-163">Consulte [Conectar uma rede local ao Azure usando o ExpressRoute com failover de VPN][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="2fb8a-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="2fb8a-164">Uma topologia hub-spoke também pode ser usada sem um gateway, se você não precisar de conectividade com a rede local.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="2fb8a-165">Emparelhamento VNet</span><span class="sxs-lookup"><span data-stu-id="2fb8a-165">VNet peering</span></span>

<span data-ttu-id="2fb8a-166">O emparelhamento VNet é uma relação não transitiva entre duas VNets.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="2fb8a-167">Se você precisar que os spokes se conectem entre si, considere adicionar uma conexão de emparelhamento separada entre os spokes.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="2fb8a-168">No entanto, se houver vários spokes que precisam se conectar entre si, você ficará sem conexões de emparelhamento possíveis rapidamente devido à [limitação do número de emparelhamentos de VNet por VNet][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="2fb8a-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="2fb8a-169">Nesse cenário, considere usar UDRs (rotas definidas pelo usuário) para forçar o tráfego destinado a um spoke a ser enviado para uma NVA que atua como um roteador na VNet do hub.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="2fb8a-170">Isso permitirá que os spokes se conectem entre si.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="2fb8a-171">Você também pode configurar os spokes para usar o gateway de VNet do hub para se comunicar com redes remotas.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="2fb8a-172">Para permitir que o tráfego de gateway flua do spoke para o hub e se conecte a redes remotas, é necessário fazer o seguinte:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="2fb8a-173">Configure a conexão de emparelhamento VNet no hub para **permitir trânsito de gateways**.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="2fb8a-174">Configure a conexão de emparelhamento VNet em cada spoke para **usar os gateways remotos**.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="2fb8a-175">Configure todas as conexões de emparelhamento VNet para **permitir tráfego encaminhado**.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="2fb8a-176">Considerações</span><span class="sxs-lookup"><span data-stu-id="2fb8a-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="2fb8a-177">Conectividade de spoke</span><span class="sxs-lookup"><span data-stu-id="2fb8a-177">Spoke connectivity</span></span>

<span data-ttu-id="2fb8a-178">Se você precisar de conectividade entre spokes, considere implementar uma NVA para roteamento no hub e usar UDRs no spoke para encaminhar o tráfego para o hub.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="2fb8a-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="2fb8a-179">![[2]][2]</span></span>

<span data-ttu-id="2fb8a-180">Nesse cenário, você precisa configurar todas as conexões de emparelhamento para **permitir tráfego encaminhado**.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="2fb8a-181">Superar os limites de emparelhamento VNet</span><span class="sxs-lookup"><span data-stu-id="2fb8a-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="2fb8a-182">Não deixe de considerar a [limitação do número de emparelhamentos de VNet por VNet][vnet-peering-limit] no Azure.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="2fb8a-183">Se você precisar de mais spokes do que o permitido pelo limite, considere criar uma topologia hub-spoke-hub-spoke, na qual o primeiro nível de spokes também funciona como hubs.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="2fb8a-184">O diagrama a seguir mostra essa abordagem.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="2fb8a-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="2fb8a-185">![[3]][3]</span></span>

<span data-ttu-id="2fb8a-186">Além disso, considere quais serviços compartilhados no hub, para garantir que o hub seja dimensionado para um número maior de spokes.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="2fb8a-187">Por exemplo, se seu hub fornecer serviços de firewall, considere os limites de largura de banda da sua solução de firewall ao adicionar vários spokes.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="2fb8a-188">Pode ser útil mover alguns desses serviços compartilhados para um segundo nível de hubs.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="2fb8a-189">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="2fb8a-189">Deploy the solution</span></span>

<span data-ttu-id="2fb8a-190">Uma implantação para essa arquitetura está disponível no [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="2fb8a-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="2fb8a-191">Ela usa VMs em cada VNet para testar a conectividade.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-191">It uses VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="2fb8a-192">Não há nenhum serviço hospedado na sub-rede **shared-services** na **VNet do hub**.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

<span data-ttu-id="2fb8a-193">A implantação cria os seguintes grupos de recursos em sua assinatura:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-193">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="2fb8a-194">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="2fb8a-194">hub-nva-rg</span></span>
- <span data-ttu-id="2fb8a-195">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="2fb8a-195">hub-vnet-rg</span></span>
- <span data-ttu-id="2fb8a-196">onprem-jb-rg</span><span class="sxs-lookup"><span data-stu-id="2fb8a-196">onprem-jb-rg</span></span>
- <span data-ttu-id="2fb8a-197">onprem-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="2fb8a-197">onprem-vnet-rg</span></span>
- <span data-ttu-id="2fb8a-198">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="2fb8a-198">spoke1-vnet-rg</span></span>
- <span data-ttu-id="2fb8a-199">spoke2-vent-rg</span><span class="sxs-lookup"><span data-stu-id="2fb8a-199">spoke2-vent-rg</span></span>

<span data-ttu-id="2fb8a-200">Os arquivos de parâmetros de modelo se referem a esses nomes e, portanto, se você alterá-los, atualize os arquivos de parâmetros para que correspondam.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-200">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="2fb8a-201">pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="2fb8a-201">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="2fb8a-202">Implantar o datacenter local simulado</span><span class="sxs-lookup"><span data-stu-id="2fb8a-202">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="2fb8a-203">Para implantar o datacenter local simulado como uma rede virtual do Azure, siga as seguintes as etapas:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-203">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="2fb8a-204">Navegue até a pasta `hybrid-networking/hub-spoke` do repositório de arquiteturas de referência.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-204">Navigate to the `hybrid-networking/hub-spoke` folder of the reference architectures repository.</span></span>

2. <span data-ttu-id="2fb8a-205">Abra o arquivo `onprem.json` .</span><span class="sxs-lookup"><span data-stu-id="2fb8a-205">Open the `onprem.json` file.</span></span> <span data-ttu-id="2fb8a-206">Substitua os valores de `adminUsername` e `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-206">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. <span data-ttu-id="2fb8a-207">(Opcional) Para uma implantação do Linux, defina `osType` como `Linux`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-207">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

4. <span data-ttu-id="2fb8a-208">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-208">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. <span data-ttu-id="2fb8a-209">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-209">Wait for the deployment to finish.</span></span> <span data-ttu-id="2fb8a-210">Essa implantação cria uma rede virtual, uma máquina virtual e um gateway de VPN.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-210">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="2fb8a-211">Pode levar cerca de 40 minutos para criar o gateway de VPN.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-211">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="2fb8a-212">Implantar a VNET do hub</span><span class="sxs-lookup"><span data-stu-id="2fb8a-212">Deploy the hub VNet</span></span>

<span data-ttu-id="2fb8a-213">Para implantar a VNet do hub, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-213">To deploy the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="2fb8a-214">Abra o arquivo `hub-vnet.json` .</span><span class="sxs-lookup"><span data-stu-id="2fb8a-214">Open the `hub-vnet.json` file.</span></span> <span data-ttu-id="2fb8a-215">Substitua os valores de `adminUsername` e `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-215">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="2fb8a-216">(Opcional) Para uma implantação do Linux, defina `osType` como `Linux`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-216">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="2fb8a-217">Localize as duas instâncias de `sharedKey` e insira uma chave compartilhada para a conexão VPN.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-217">Find both instances of `sharedKey` and enter a shared key for the VPN connection.</span></span> <span data-ttu-id="2fb8a-218">Os valores devem ser correspondentes.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-218">The values must match.</span></span>

    ```bash
    "sharedKey": "",
    ```

4. <span data-ttu-id="2fb8a-219">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-219">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. <span data-ttu-id="2fb8a-220">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-220">Wait for the deployment to finish.</span></span> <span data-ttu-id="2fb8a-221">Essa implantação cria uma rede virtual, uma máquina virtual, um gateway de VPN e uma conexão com o gateway.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-221">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway.</span></span>  <span data-ttu-id="2fb8a-222">Pode levar cerca de 40 minutos para criar o gateway de VPN.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-222">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="test-connectivity-with-the-hub"></a><span data-ttu-id="2fb8a-223">Testar a conectividade com o hub</span><span class="sxs-lookup"><span data-stu-id="2fb8a-223">Test connectivity with the hub</span></span>

<span data-ttu-id="2fb8a-224">Testar a conexão do ambiente local simulado para a VNET do hub.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-224">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

<span data-ttu-id="2fb8a-225">**Implantação no Windows**</span><span class="sxs-lookup"><span data-stu-id="2fb8a-225">**Windows deployment**</span></span>

1. <span data-ttu-id="2fb8a-226">Usar o portal do Azure para encontrar a VM denominada `jb-vm1` no grupo de recursos `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-226">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="2fb8a-227">Clique em `Connect` para abrir uma sessão de área de trabalho remota para a VM.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-227">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="2fb8a-228">Use a senha que você especificou no arquivo de parâmetro `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-228">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="2fb8a-229">Abra um console do PowerShell na VM e use o cmdlet `Test-NetConnection` para verificar se você pode se conectar à VM jumpbox na Vnet do hub.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-229">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
<span data-ttu-id="2fb8a-230">O resultado deve ser semelhante ao seguinte:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-230">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="2fb8a-231">Por padrão, as máquinas virtuais do Windows Server não permitem respostas do ICMP no Azure.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-231">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="2fb8a-232">Se você quiser usar `ping` para testar a conectividade, você precisa habilitar o tráfego ICMP no Firewall Avançado do Windows para cada VM.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-232">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="2fb8a-233">**Implantação no Linux**</span><span class="sxs-lookup"><span data-stu-id="2fb8a-233">**Linux deployment**</span></span>

1. <span data-ttu-id="2fb8a-234">Usar o portal do Azure para encontrar a VM denominada `jb-vm1` no grupo de recursos `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-234">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="2fb8a-235">Clique em `Connect` e copie o comando `ssh` mostrado no portal.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-235">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="2fb8a-236">Em um prompt do Linux, execute `ssh` para se conectar ao ambiente local simulado.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-236">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="2fb8a-237">Use a senha que você especificou no arquivo de parâmetro `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-237">Use the password that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="2fb8a-238">Use o comando `ping` para testar a conectividade com a VM jumpbox na VNET do hub:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-238">Use the `ping` command to test connectivity to the jumpbox VM in the hub VNet:</span></span>

   ```bash
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="2fb8a-239">Implantar as redes virtuais spoke</span><span class="sxs-lookup"><span data-stu-id="2fb8a-239">Deploy the spoke VNets</span></span>

<span data-ttu-id="2fb8a-240">Para implantar as redes virtuais spoke, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-240">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="2fb8a-241">Abra o arquivo `spoke1.json` .</span><span class="sxs-lookup"><span data-stu-id="2fb8a-241">Open the `spoke1.json` file.</span></span> <span data-ttu-id="2fb8a-242">Substitua os valores de `adminUsername` e `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-242">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="2fb8a-243">(Opcional) Para uma implantação do Linux, defina `osType` como `Linux`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-243">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="2fb8a-244">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-244">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. <span data-ttu-id="2fb8a-245">Repita as etapas 1 e 2 para o arquivo `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-245">Repeat steps 1-2 for the `spoke2.json` file.</span></span>

5. <span data-ttu-id="2fb8a-246">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-246">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. <span data-ttu-id="2fb8a-247">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-247">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="2fb8a-248">Testar a conectividade</span><span class="sxs-lookup"><span data-stu-id="2fb8a-248">Test connectivity</span></span>

<span data-ttu-id="2fb8a-249">Testar a conexão do ambiente local simulado para as redes virtuais spoke.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-249">Test conectivity from the simulated on-premises environment to the spoke VNets.</span></span>

<span data-ttu-id="2fb8a-250">**Implantação no Windows**</span><span class="sxs-lookup"><span data-stu-id="2fb8a-250">**Windows deployment**</span></span>

1. <span data-ttu-id="2fb8a-251">Usar o portal do Azure para encontrar a VM denominada `jb-vm1` no grupo de recursos `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-251">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="2fb8a-252">Clique em `Connect` para abrir uma sessão de área de trabalho remota para a VM.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-252">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="2fb8a-253">Use a senha que você especificou no arquivo de parâmetro `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-253">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="2fb8a-254">Abra um console do PowerShell na VM e use o cmdlet `Test-NetConnection` para verificar se você pode se conectar à VM jumpbox na rede virtual spoke.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-254">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VMs in the spoke VNets.</span></span>

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="2fb8a-255">**Implantação no Linux**</span><span class="sxs-lookup"><span data-stu-id="2fb8a-255">**Linux deployment**</span></span>

<span data-ttu-id="2fb8a-256">Para testar a conectividade do ambiente simulado no local para a redes virtuais do spoke usando as máquinas virtuais do Linux, execute as etapas a seguir:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-256">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="2fb8a-257">Usar o portal do Azure para encontrar a VM denominada `jb-vm1` no grupo de recursos `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-257">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="2fb8a-258">Clique em `Connect` e copie o comando `ssh` mostrado no portal.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-258">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="2fb8a-259">Em um prompt do Linux, execute `ssh` para se conectar ao ambiente local simulado.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-259">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="2fb8a-260">Use a senha que você especificou no arquivo de parâmetro `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-260">Use the password that you specified in the `onprem.json` parameter file.</span></span>

5. <span data-ttu-id="2fb8a-261">Use o comando `ping` para testar a conectividade com a VM jumpbox em cada spoke:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-261">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke:</span></span>

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="2fb8a-262">Adicionar conectividade entre spokes</span><span class="sxs-lookup"><span data-stu-id="2fb8a-262">Add connectivity between spokes</span></span>

<span data-ttu-id="2fb8a-263">Esta etapa é opcional.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-263">This step is optional.</span></span> <span data-ttu-id="2fb8a-264">Se você quiser permitir que spokes se conectem uns aos outros, precisará usar uma NVA (solução de virtualização de rede) como um roteador na VNET do hub e forçar o tráfego de spokes para o roteador ao tentar conectar-se a outro spoke.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-264">If you want to allow spokes to connect to each other, you must use a newtwork virtual appliance (NVA) as a router in the hub VNet, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="2fb8a-265">Para implantar um NVA de amostra básico como uma única VM junto com as UDRs (rotas definidas pelo usuário) e permitir que as duas redes virtuais de spoke se conectem, execute as seguintes etapas:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-265">To deploy a basic sample NVA as a single VM, along with user-defined routes (UDRs) to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="2fb8a-266">Abra o arquivo `hub-nva.json` .</span><span class="sxs-lookup"><span data-stu-id="2fb8a-266">Open the `hub-nva.json` file.</span></span> <span data-ttu-id="2fb8a-267">Substitua os valores de `adminUsername` e `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="2fb8a-267">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="2fb8a-268">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="2fb8a-268">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Topologia hub-spoke no Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Topologia hub-spoke no Azure com o roteamento transitivo"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Topologia hub-spoke no Azure com o roteamento transitivo usando uma NVA"
[3]: ./images/hub-spokehub-spoke.svg "Topologia hub-spoke-hub-spoke no Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
