---
title: "Implementação de uma topologia de rede hub-spoke no Azure"
description: Como implementar uma topologia de rede hub-spoke no Azure.
author: telmosampaio
ms.date: 02/23/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 1a2855f0d4a903fc4d7a022aef20ea73fe763e2c
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/08/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="8e264-103">Implementar uma topologia de rede hub-spoke no Azure</span><span class="sxs-lookup"><span data-stu-id="8e264-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="8e264-104">Essa arquitetura de referência mostra como implementar uma topologia hub-spoke no Azure.</span><span class="sxs-lookup"><span data-stu-id="8e264-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="8e264-105">O *hub* é uma VNet (rede virtual) no Azure que atua como ponto central de conectividade para sua rede local.</span><span class="sxs-lookup"><span data-stu-id="8e264-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="8e264-106">Os *spokes* são VNets que se emparelham com o hub e podem ser usadas para isolar as cargas de trabalho.</span><span class="sxs-lookup"><span data-stu-id="8e264-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="8e264-107">Fluxos de tráfego entre o datacenter local e o hub por meio de uma conexão de ExpressRoute ou gateway de VPN.</span><span class="sxs-lookup"><span data-stu-id="8e264-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="8e264-108">[**Implantar esta solução**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="8e264-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="8e264-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="8e264-109">![[0]][0]</span></span>

<span data-ttu-id="8e264-110">*Baixar um [arquivo Visio][visio-download] dessa arquitetura*</span><span class="sxs-lookup"><span data-stu-id="8e264-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="8e264-111">Os benefícios dessa topologia incluem:</span><span class="sxs-lookup"><span data-stu-id="8e264-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="8e264-112">**Redução de custos** com a centralização de serviços que podem ser compartilhados por diversas cargas de trabalho, tais como NVAs (soluções de virtualização de rede) e servidores DNS, em um único local.</span><span class="sxs-lookup"><span data-stu-id="8e264-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="8e264-113">**Superar os limites de assinaturas** emparelhando VNets de assinaturas diferentes com o hub central.</span><span class="sxs-lookup"><span data-stu-id="8e264-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="8e264-114">**Separação de preocupações** entre TI central (SecOps, InfraOps) e cargas de trabalho (DevOps).</span><span class="sxs-lookup"><span data-stu-id="8e264-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="8e264-115">Alguns usos típicos dessa arquitetura:</span><span class="sxs-lookup"><span data-stu-id="8e264-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="8e264-116">As cargas de trabalho implantadas em diferentes ambientes, como desenvolvimento, teste e produção, que exigem serviços compartilhados como DNS, IDS, NTP ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="8e264-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="8e264-117">Os serviços compartilhados são colocados na VNet do hub, enquanto cada ambiente é implantado em um spoke para manter o isolamento.</span><span class="sxs-lookup"><span data-stu-id="8e264-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="8e264-118">Cargas de trabalho que não exigem conectividade entre si, mas precisam de acesso aos serviços compartilhados.</span><span class="sxs-lookup"><span data-stu-id="8e264-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="8e264-119">Empresas que exigem o controle centralizado sobre aspectos de segurança, como um firewall no hub como uma DMZ e gerenciamento separado para cargas de trabalho em cada spoke.</span><span class="sxs-lookup"><span data-stu-id="8e264-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="8e264-120">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="8e264-120">Architecture</span></span>

<span data-ttu-id="8e264-121">A arquitetura consiste nos componentes a seguir.</span><span class="sxs-lookup"><span data-stu-id="8e264-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="8e264-122">**Rede local**.</span><span class="sxs-lookup"><span data-stu-id="8e264-122">**On-premises network**.</span></span> <span data-ttu-id="8e264-123">Uma rede de área local privada em execução dentro de uma organização.</span><span class="sxs-lookup"><span data-stu-id="8e264-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="8e264-124">**Dispositivo VPN**.</span><span class="sxs-lookup"><span data-stu-id="8e264-124">**VPN device**.</span></span> <span data-ttu-id="8e264-125">Um dispositivo ou serviço que fornece conectividade externa para a rede local.</span><span class="sxs-lookup"><span data-stu-id="8e264-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="8e264-126">O dispositivo VPN pode ser um dispositivo de hardware ou uma solução de software, como o RRAS (Serviço de Acesso Remoto e Roteamento) do Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="8e264-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="8e264-127">Para obter uma lista de dispositivos de VPN com suporte e informações sobre como configurar dispositivos de VPN selecionados para se conectar ao Azure, consulte [Sobre dispositivos VPN para conexões de Gateway de VPN Site a Site][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="8e264-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="8e264-128">**Gateway de rede virtual de VPN ou gateway de ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="8e264-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="8e264-129">O gateway de rede virtual permite que a VNet se conecte ao dispositivo VPN ou ao circuito de ExpressRoute, usado para conectividade com a rede local.</span><span class="sxs-lookup"><span data-stu-id="8e264-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="8e264-130">Para obter mais informações, consulte [Conectar uma rede local a uma rede virtual do Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="8e264-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="8e264-131">Os scripts de implantação dessa arquitetura de referência usam um gateway VPN para conectividade e uma VNet no Azure para simular a rede local.</span><span class="sxs-lookup"><span data-stu-id="8e264-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="8e264-132">**VNet do hub**.</span><span class="sxs-lookup"><span data-stu-id="8e264-132">**Hub VNet**.</span></span> <span data-ttu-id="8e264-133">VNet do Azure usada como o hub na topologia hub-spoke.</span><span class="sxs-lookup"><span data-stu-id="8e264-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="8e264-134">O hub é o ponto central de conectividade para sua rede local e um local para hospedar os serviços que podem ser consumidos por diferentes cargas de trabalho hospedadas nas VNets do spoke.</span><span class="sxs-lookup"><span data-stu-id="8e264-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="8e264-135">**Sub-rede do gateway**.</span><span class="sxs-lookup"><span data-stu-id="8e264-135">**Gateway subnet**.</span></span> <span data-ttu-id="8e264-136">Os gateways de rede virtual são mantidos na mesma sub-rede.</span><span class="sxs-lookup"><span data-stu-id="8e264-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="8e264-137">**VNets de spoke**.</span><span class="sxs-lookup"><span data-stu-id="8e264-137">**Spoke VNets**.</span></span> <span data-ttu-id="8e264-138">Uma ou mais VNets do Azure que são usadas como spokes na topologia hub-spoke.</span><span class="sxs-lookup"><span data-stu-id="8e264-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="8e264-139">Spokes podem ser usados para isolar as cargas de trabalho em suas próprias VNets, gerenciadas separadamente de outros spokes.</span><span class="sxs-lookup"><span data-stu-id="8e264-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="8e264-140">Cada carga de trabalho pode incluir várias camadas, com várias sub-redes conectadas por meio de balanceadores de carga do Azure.</span><span class="sxs-lookup"><span data-stu-id="8e264-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="8e264-141">Para obter mais informações sobre a infraestrutura do aplicativo, consulte [Execução de cargas de trabalho de VM do Windows][windows-vm-ra] e [Execução de cargas de trabalho de VM do Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="8e264-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="8e264-142">**Emparelhamento VNet**.</span><span class="sxs-lookup"><span data-stu-id="8e264-142">**VNet peering**.</span></span> <span data-ttu-id="8e264-143">Duas VNets na mesma região do Azure podem ser conectadas usando uma [conexão de emparelhamento][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="8e264-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="8e264-144">Conexões de emparelhamento são conexões não transitivas de baixa latência entre VNets.</span><span class="sxs-lookup"><span data-stu-id="8e264-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="8e264-145">Quando emparelhadas, as VNets trocam tráfego usando o backbone do Azure sem precisar de um roteador.</span><span class="sxs-lookup"><span data-stu-id="8e264-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="8e264-146">Em uma topologia de rede hub-spoke, é necessário usar o emparelhamento VNet para conectar o hub a cada spoke.</span><span class="sxs-lookup"><span data-stu-id="8e264-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="8e264-147">Este artigo aborda apenas implantações do [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mas também é possível conectar uma VNet clássica a uma VNet do Resource Manager na mesma assinatura.</span><span class="sxs-lookup"><span data-stu-id="8e264-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="8e264-148">Dessa forma, seus spokes podem hospedar implantações clássicas e ainda aproveitar os serviços compartilhados no hub.</span><span class="sxs-lookup"><span data-stu-id="8e264-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="8e264-149">Recomendações</span><span class="sxs-lookup"><span data-stu-id="8e264-149">Recommendations</span></span>

<span data-ttu-id="8e264-150">As seguintes recomendações aplicam-se à maioria dos cenários.</span><span class="sxs-lookup"><span data-stu-id="8e264-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="8e264-151">Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.</span><span class="sxs-lookup"><span data-stu-id="8e264-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="8e264-152">Grupos de recursos</span><span class="sxs-lookup"><span data-stu-id="8e264-152">Resource groups</span></span>

<span data-ttu-id="8e264-153">A VNet do hub e a VNet de cada spoke, podem ser implementadas em diferentes grupos de recursos, e até mesmo em assinaturas diferentes, contanto que elas pertençam ao mesmo locatário do Azure AD (Azure Active Directory) na mesma região do Azure.</span><span class="sxs-lookup"><span data-stu-id="8e264-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="8e264-154">Isso possibilita o gerenciamento descentralizado de cada carga de trabalho, compartilhando os serviços mantidos na VNet do hub.</span><span class="sxs-lookup"><span data-stu-id="8e264-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="8e264-155">VNet e GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="8e264-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="8e264-156">Crie uma sub-rede denominada *GatewaySubnet* com um intervalo de endereços de /27.</span><span class="sxs-lookup"><span data-stu-id="8e264-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="8e264-157">Essa sub-rede é necessária para o gateway de rede virtual.</span><span class="sxs-lookup"><span data-stu-id="8e264-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="8e264-158">Alocar 32 endereços para esta sub-rede ajudará a evitar que as limitações de tamanho de gateway sejam atingidas no futuro.</span><span class="sxs-lookup"><span data-stu-id="8e264-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="8e264-159">Para obter mais informações sobre como configurar o gateway, consulte as seguintes arquiteturas de referência, dependendo do seu tipo de conexão:</span><span class="sxs-lookup"><span data-stu-id="8e264-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="8e264-160">[Rede híbrida usando o ExpressRoute][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="8e264-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="8e264-161">[Rede híbrida usando um gateway de VPN][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="8e264-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="8e264-162">Para maior disponibilidade, você pode usar o ExpressRoute e uma VPN para failover.</span><span class="sxs-lookup"><span data-stu-id="8e264-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="8e264-163">Consulte [Conectar uma rede local ao Azure usando o ExpressRoute com failover de VPN][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="8e264-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="8e264-164">Uma topologia hub-spoke também pode ser usada sem um gateway, se você não precisar de conectividade com a rede local.</span><span class="sxs-lookup"><span data-stu-id="8e264-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="8e264-165">Emparelhamento VNet</span><span class="sxs-lookup"><span data-stu-id="8e264-165">VNet peering</span></span>

<span data-ttu-id="8e264-166">O emparelhamento VNet é uma relação não transitiva entre duas VNets.</span><span class="sxs-lookup"><span data-stu-id="8e264-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="8e264-167">Se você precisar que os spokes se conectem entre si, considere adicionar uma conexão de emparelhamento separada entre os spokes.</span><span class="sxs-lookup"><span data-stu-id="8e264-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="8e264-168">No entanto, se houver vários spokes que precisam se conectar entre si, você ficará sem conexões de emparelhamento possíveis rapidamente devido à [limitação do número de emparelhamentos de VNet por VNet][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="8e264-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="8e264-169">Nesse cenário, considere usar UDRs (rotas definidas pelo usuário) para forçar o tráfego destinado a um spoke a ser enviado para uma NVA que atua como um roteador na VNet do hub.</span><span class="sxs-lookup"><span data-stu-id="8e264-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="8e264-170">Isso permitirá que os spokes se conectem entre si.</span><span class="sxs-lookup"><span data-stu-id="8e264-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="8e264-171">Você também pode configurar os spokes para usar o gateway de VNet do hub para se comunicar com redes remotas.</span><span class="sxs-lookup"><span data-stu-id="8e264-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="8e264-172">Para permitir que o tráfego de gateway flua do spoke para o hub e se conecte a redes remotas, é necessário fazer o seguinte:</span><span class="sxs-lookup"><span data-stu-id="8e264-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="8e264-173">Configure a conexão de emparelhamento VNet no hub para **permitir trânsito de gateways**.</span><span class="sxs-lookup"><span data-stu-id="8e264-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="8e264-174">Configure a conexão de emparelhamento VNet em cada spoke para **usar os gateways remotos**.</span><span class="sxs-lookup"><span data-stu-id="8e264-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="8e264-175">Configure todas as conexões de emparelhamento VNet para **permitir tráfego encaminhado**.</span><span class="sxs-lookup"><span data-stu-id="8e264-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="8e264-176">Considerações</span><span class="sxs-lookup"><span data-stu-id="8e264-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="8e264-177">Conectividade de spoke</span><span class="sxs-lookup"><span data-stu-id="8e264-177">Spoke connectivity</span></span>

<span data-ttu-id="8e264-178">Se você precisar de conectividade entre spokes, considere implementar uma NVA para roteamento no hub e usar UDRs no spoke para encaminhar o tráfego para o hub.</span><span class="sxs-lookup"><span data-stu-id="8e264-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="8e264-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="8e264-179">![[2]][2]</span></span>

<span data-ttu-id="8e264-180">Nesse cenário, você precisa configurar todas as conexões de emparelhamento para **permitir tráfego encaminhado**.</span><span class="sxs-lookup"><span data-stu-id="8e264-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="8e264-181">Superar os limites de emparelhamento VNet</span><span class="sxs-lookup"><span data-stu-id="8e264-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="8e264-182">Não deixe de considerar a [limitação do número de emparelhamentos de VNet por VNet][vnet-peering-limit] no Azure.</span><span class="sxs-lookup"><span data-stu-id="8e264-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="8e264-183">Se você precisar de mais spokes do que o permitido pelo limite, considere criar uma topologia hub-spoke-hub-spoke, na qual o primeiro nível de spokes também funciona como hubs.</span><span class="sxs-lookup"><span data-stu-id="8e264-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="8e264-184">O diagrama a seguir mostra essa abordagem.</span><span class="sxs-lookup"><span data-stu-id="8e264-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="8e264-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="8e264-185">![[3]][3]</span></span>

<span data-ttu-id="8e264-186">Além disso, considere quais serviços compartilhados no hub, para garantir que o hub seja dimensionado para um número maior de spokes.</span><span class="sxs-lookup"><span data-stu-id="8e264-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="8e264-187">Por exemplo, se seu hub fornecer serviços de firewall, considere os limites de largura de banda da sua solução de firewall ao adicionar vários spokes.</span><span class="sxs-lookup"><span data-stu-id="8e264-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="8e264-188">Pode ser útil mover alguns desses serviços compartilhados para um segundo nível de hubs.</span><span class="sxs-lookup"><span data-stu-id="8e264-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="8e264-189">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="8e264-189">Deploy the solution</span></span>

<span data-ttu-id="8e264-190">Uma implantação para essa arquitetura está disponível no [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="8e264-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="8e264-191">Ela usa VMs Ubuntu em cada VNet para testar a conectividade.</span><span class="sxs-lookup"><span data-stu-id="8e264-191">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="8e264-192">Não há nenhum serviço hospedado na sub-rede **shared-services** na **VNet do hub**.</span><span class="sxs-lookup"><span data-stu-id="8e264-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="8e264-193">pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="8e264-193">Prerequisites</span></span>

<span data-ttu-id="8e264-194">Antes de implantar a arquitetura de referência para sua própria assinatura, você deve executar as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="8e264-194">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="8e264-195">Clone, crie fork ou baixe o arquivo zip para as [arquiteturas de referência AzureCAT][ref-arch-repo] no repositório GitHub.</span><span class="sxs-lookup"><span data-stu-id="8e264-195">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="8e264-196">Verifique se a CLI do Azure 2.0 está instalada no computador.</span><span class="sxs-lookup"><span data-stu-id="8e264-196">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="8e264-197">Para obter instruções de instalação da CLI, consulte [Instalar a CLI 2.0 do Azure][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="8e264-197">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="8e264-198">Instale os pacote npm dos [Blocos de construção do Azure][azbb].</span><span class="sxs-lookup"><span data-stu-id="8e264-198">Install the [Azure buulding blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="8e264-199">Em um prompt de comando, bash prompt ou prompt do PowerShell, faça logon na sua conta do Azure usando o comando abaixo e siga os prompts.</span><span class="sxs-lookup"><span data-stu-id="8e264-199">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="8e264-200">Implantar o datacenter local simulado usand azbb</span><span class="sxs-lookup"><span data-stu-id="8e264-200">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="8e264-201">Para implantar o datacenter local simulado como uma rede virtual do Azure, siga as seguintes as etapas:</span><span class="sxs-lookup"><span data-stu-id="8e264-201">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="8e264-202">Navegue até a pasta `hybrid-networking\hub-spoke\` do repositório que você baixou na etapa de pré-requisitos acima.</span><span class="sxs-lookup"><span data-stu-id="8e264-202">Navigate to the `hybrid-networking\hub-spoke\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="8e264-203">Abra o arquivo `onprem.json` e insira um nome de usuário e senha entre aspas nas linhas 36 e 37, conforme mostrado abaixo e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="8e264-203">Open the `onprem.json` file and enter a username and password between the quotes in line 36 and 37, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="8e264-204">Na linha 38, para `osType`, digite `Windows` ou `Linux` para instalar o Windows Server 2016 Datacenter ou o Ubuntu 16.04 como o sistema operacional para o jumpbox.</span><span class="sxs-lookup"><span data-stu-id="8e264-204">On line 38, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

4. <span data-ttu-id="8e264-205">Execute `azbb` para implantar o ambiente simulado local, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="8e264-205">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="8e264-206">Se você decidir usar um nome do grupo de recursos distinto (diferente de `onprem-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="8e264-206">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="8e264-207">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="8e264-207">Wait for the deployment to finish.</span></span> <span data-ttu-id="8e264-208">Essa implantação cria uma rede virtual, uma máquina virtual e um gateway de VPN.</span><span class="sxs-lookup"><span data-stu-id="8e264-208">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="8e264-209">A criação do gateway de VPN pode levar mais de 40 minutos para ser concluída.</span><span class="sxs-lookup"><span data-stu-id="8e264-209">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="8e264-210">VNet do hub do Azure</span><span class="sxs-lookup"><span data-stu-id="8e264-210">Azure hub VNet</span></span>

<span data-ttu-id="8e264-211">Para implantar a VNet de hub e conectar-se à VNet local simulada criada acima, execute as seguintes etapas.</span><span class="sxs-lookup"><span data-stu-id="8e264-211">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="8e264-212">Abra o arquivo `hub-vnet.json` e insira um nome de usuário e senha entre aspas nas linhas 39 e 40, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="8e264-212">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 39 and 40, as shown below.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="8e264-213">Na linha 41, para `osType`, digite `Windows` ou `Linux` para instalar o Windows Server 2016 Datacenter ou o Ubuntu 16.04 como o sistema operacional para o jumpbox.</span><span class="sxs-lookup"><span data-stu-id="8e264-213">On line 41, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="8e264-214">Insira uma chave compartilhada entre as aspas na linha 72, conforme mostrado abaixo, e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="8e264-214">Enter a shared key between the quotes in line 72, as shown below, then save the file.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="8e264-215">Execute `azbb` para implantar o ambiente simulado local, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="8e264-215">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="8e264-216">Se você decidir usar um nome do grupo de recursos distinto (diferente de `hub-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="8e264-216">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="8e264-217">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="8e264-217">Wait for the deployment to finish.</span></span> <span data-ttu-id="8e264-218">Essa implantação cria uma rede virtual, uma máquina virtual, um gateway de VPN e uma conexão com o gateway criado na seção anterior.</span><span class="sxs-lookup"><span data-stu-id="8e264-218">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="8e264-219">A criação do gateway de VPN pode levar mais de 40 minutos para ser concluída.</span><span class="sxs-lookup"><span data-stu-id="8e264-219">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="optional-test-connectivity-from-onprem-to-hub"></a><span data-ttu-id="8e264-220">(Opcional) Testar a conectividade do local para o hub</span><span class="sxs-lookup"><span data-stu-id="8e264-220">(Optional) Test connectivity from onprem to hub</span></span>

<span data-ttu-id="8e264-221">Para testar a conectividade do ambiente simulado no local para a rede virtual do hub usando as máquinas virtuais do Windows, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="8e264-221">To test conectivity from the simulated on-premises environment to the hub VNet using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="8e264-222">No portal do Azure, navegue até o grupo de recursos `onprem-jb-rg`, em seguida, clique no recurso `jb-vm1` da máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="8e264-222">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2.  <span data-ttu-id="8e264-223">No canto superior esquerdo da sua folha de VM no portal, clique em `Connect` e siga os prompts para usar a área de trabalho remota para conectar-se à VM.</span><span class="sxs-lookup"><span data-stu-id="8e264-223">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="8e264-224">Certifique-se de usar o nome de usuário e a senha especificados nas linhas 36 e 37 no arquivo `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="8e264-224">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="8e264-225">Abra um console do PowerShell na VM e use o cmdlet `Test-NetConnection` para verificar se você pode se conectar à VM jumpbox do hub conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="8e264-225">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

  ```powershell
  Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
  ```
  > [!NOTE]
  > <span data-ttu-id="8e264-226">Por padrão, as máquinas virtuais do Windows Server não permitem respostas do ICMP no Azure.</span><span class="sxs-lookup"><span data-stu-id="8e264-226">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="8e264-227">Se você quiser usar `ping` para testar a conectividade, você precisa habilitar o tráfego ICMP no Firewall Avançado do Windows para cada VM.</span><span class="sxs-lookup"><span data-stu-id="8e264-227">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="8e264-228">Para testar a conectividade do ambiente simulado no local para a rede virtual do hub usando as máquinas virtuais do Linux, execute as etapas a seguir:</span><span class="sxs-lookup"><span data-stu-id="8e264-228">To test conectivity from the simulated on-premises environment to the hub VNet using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="8e264-229">No portal do Azure, navegue até o grupo de recursos `onprem-jb-rg`, em seguida, clique no recurso `jb-vm1` da máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="8e264-229">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="8e264-230">No canto superior esquerdo da sua folha de VM no portal, clique em `Connect` e, em seguida, copie o comando `ssh` mostrados no portal.</span><span class="sxs-lookup"><span data-stu-id="8e264-230">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="8e264-231">Em um prompt do Linux, execute `ssh` para se conectar ao jumpbox do ambiente simulado local com as informações que você copiou na etapa 2 acima, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="8e264-231">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

  ```bash
  ssh <your_user>@<public_ip_address>
  ```

4. <span data-ttu-id="8e264-232">Use a senha especificada na linha 37 no arquivo `onprem.json` para se conectar à VM.</span><span class="sxs-lookup"><span data-stu-id="8e264-232">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

5. <span data-ttu-id="8e264-233">Use o comando `ping` para testar a conectividade com o jumbox do hub, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="8e264-233">Use the `ping` command to test connectivity to the hub jumpbox, as shown below.</span></span>

  ```bash
  ping 10.0.0.68
  ```

### <a name="azure-spoke-vnets"></a><span data-ttu-id="8e264-234">VNets de spoke do Azure</span><span class="sxs-lookup"><span data-stu-id="8e264-234">Azure spoke VNets</span></span>

<span data-ttu-id="8e264-235">Para implantar as redes virtuais spoke, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="8e264-235">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="8e264-236">Abra o arquivo `spoke1.json` e insira um nome de usuário e senha entre aspas nas linhas 47 e 48, conforme mostrado abaixo, e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="8e264-236">Open the `spoke1.json` file and enter a username and password between the quotes in lines 47 and 48, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="8e264-237">Na linha 49, para `osType`, digite `Windows` ou `Linux` para instalar o Windows Server 2016 Datacenter ou o Ubuntu 16.04 como o sistema operacional para o jumpbox.</span><span class="sxs-lookup"><span data-stu-id="8e264-237">On line 49, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="8e264-238">Execute `azbb` para implantar o primeiro ambiente de rede virtual spoke, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="8e264-238">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="8e264-239">Se você decidir usar um nome do grupo de recursos distinto (diferente de `spoke1-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="8e264-239">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

3. <span data-ttu-id="8e264-240">Repita a etapa 1 acima para o arquivo `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="8e264-240">Repeat step 1 above for file `spoke2.json`.</span></span>

4. <span data-ttu-id="8e264-241">Execute `azbb` para implantar o segundo ambiente de rede virtual spoke, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="8e264-241">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="8e264-242">Se você decidir usar um nome do grupo de recursos distinto (diferente de `spoke2-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="8e264-242">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="8e264-243">Emparelhamento VNet do hub do Azure para VNets de spoke</span><span class="sxs-lookup"><span data-stu-id="8e264-243">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="8e264-244">Para criar uma conexão de emparelhamento da rede virtual do hub para as redes virtuais spoke, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="8e264-244">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="8e264-245">Abra o arquivo `hub-vnet-peering.json` e verifique se o nome do grupo de recursos e o nome da rede virtual para cada um dos emparelhamentos de rede virtual, começando na linha 29, estão corretos.</span><span class="sxs-lookup"><span data-stu-id="8e264-245">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="8e264-246">Execute `azbb` para implantar o primeiro ambiente de rede virtual spoke, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="8e264-246">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > <span data-ttu-id="8e264-247">Se você decidir usar um nome do grupo de recursos distinto (diferente de `hub-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="8e264-247">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="test-connectivity"></a><span data-ttu-id="8e264-248">Testar a conectividade</span><span class="sxs-lookup"><span data-stu-id="8e264-248">Test connectivity</span></span>

<span data-ttu-id="8e264-249">Para testar a conectividade do ambiente simulado no local para a redes virtuais do spoke usando as máquinas virtuais do Windows, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="8e264-249">To test conectivity from the simulated on-premises environment to the spoke VNets using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="8e264-250">No portal do Azure, navegue até o grupo de recursos `onprem-jb-rg`, em seguida, clique no recurso `jb-vm1` da máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="8e264-250">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2.  <span data-ttu-id="8e264-251">No canto superior esquerdo da sua folha de VM no portal, clique em `Connect` e siga os prompts para usar a área de trabalho remota para conectar-se à VM.</span><span class="sxs-lookup"><span data-stu-id="8e264-251">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="8e264-252">Certifique-se de usar o nome de usuário e a senha especificados nas linhas 36 e 37 no arquivo `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="8e264-252">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="8e264-253">Abra um console do PowerShell na VM e use o cmdlet `Test-NetConnection` para verificar se você pode se conectar à VM jumpbox do hub conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="8e264-253">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

  ```powershell
  Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
  Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
  ```

<span data-ttu-id="8e264-254">Para testar a conectividade do ambiente simulado no local para a redes virtuais do spoke usando as máquinas virtuais do Linux, execute as etapas a seguir:</span><span class="sxs-lookup"><span data-stu-id="8e264-254">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="8e264-255">No portal do Azure, navegue até o grupo de recursos `onprem-jb-rg`, em seguida, clique no recurso `jb-vm1` da máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="8e264-255">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="8e264-256">No canto superior esquerdo da sua folha de VM no portal, clique em `Connect` e, em seguida, copie o comando `ssh` mostrados no portal.</span><span class="sxs-lookup"><span data-stu-id="8e264-256">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="8e264-257">Em um prompt do Linux, execute `ssh` para se conectar ao jumpbox do ambiente simulado local com as informações que você copiou na etapa 2 acima, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="8e264-257">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

  ```bash
  ssh <your_user>@<public_ip_address>
  ```

5. <span data-ttu-id="8e264-258">Use a senha especificada na linha 37 no arquivo `onprem.json` para se conectar à VM.</span><span class="sxs-lookup"><span data-stu-id="8e264-258">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

6. <span data-ttu-id="8e264-259">Use o comando `ping` para testar a conectividade com as máquinas virtuais do jumbox, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="8e264-259">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke, as shown below.</span></span>

  ```bash
  ping 10.1.0.68
  ping 10.2.0.68
  ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="8e264-260">Adicionar conectividade entre spokes</span><span class="sxs-lookup"><span data-stu-id="8e264-260">Add connectivity between spokes</span></span>

<span data-ttu-id="8e264-261">Se você quiser permitir que spokes e conectem uns aos outros, você precisa usar uma solução de virtualização de rede (NVA) como um roteador na rede virtual do hub e forçar o tráfego de spokes para o roteador ao tentar conectar-se a outro spoke.</span><span class="sxs-lookup"><span data-stu-id="8e264-261">If you want to allow spokes to connect to each other, you need to use a newtwork virtual appliance (NVA) as a router in the hub virtual netowrk, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="8e264-262">Para implantar um NVA de amostra básico como uma única VM e as rotas necessárias definidas abaixo para permitir que as duas redes virtuais de spoke se conectem, execute as seguintes etapas:</span><span class="sxs-lookup"><span data-stu-id="8e264-262">To deploy a basic sample NVA as a single VM, and the necessary uder defined routes to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="8e264-263">Abra o arquivo `hub-nva.json` e insira um nome de usuário e senha entre aspas nas linhas 13 e 14, conforme mostrado abaixo, e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="8e264-263">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. <span data-ttu-id="8e264-264">Execute `azbb` para implantar a VM NVA e as rotas de usuário definidas.</span><span class="sxs-lookup"><span data-stu-id="8e264-264">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="8e264-265">Se você decidir usar um nome do grupo de recursos distinto (diferente de `hub-nva-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="8e264-265">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

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

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Topologia hub-spoke no Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Topologia hub-spoke no Azure com o roteamento transitivo"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Topologia hub-spoke no Azure com o roteamento transitivo usando uma NVA"
[3]: ./images/hub-spokehub-spoke.svg "Topologia hub-spoke-hub-spoke no Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
