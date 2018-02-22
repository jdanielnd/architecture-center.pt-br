---
title: "Implementação de uma topologia de rede hub-spoke no Azure"
description: Como implementar uma topologia de rede hub-spoke no Azure.
author: telmosampaio
ms.date: 02/14/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: c03ecd4ba5ddbe50cfb17e56d75c18102b751cfb
ms.sourcegitcommit: 475064f0a3c2fac23e1286ba159aaded287eec86
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/19/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="f1c07-103">Implementar uma topologia de rede hub-spoke no Azure</span><span class="sxs-lookup"><span data-stu-id="f1c07-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="f1c07-104">Essa arquitetura de referência mostra como implementar uma topologia hub-spoke no Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="f1c07-105">O *hub* é uma VNet (rede virtual) no Azure que atua como ponto central de conectividade para sua rede local.</span><span class="sxs-lookup"><span data-stu-id="f1c07-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="f1c07-106">Os *spokes* são VNets que se emparelham com o hub e podem ser usadas para isolar as cargas de trabalho.</span><span class="sxs-lookup"><span data-stu-id="f1c07-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="f1c07-107">Fluxos de tráfego entre o datacenter local e o hub por meio de uma conexão de ExpressRoute ou gateway de VPN.</span><span class="sxs-lookup"><span data-stu-id="f1c07-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="f1c07-108">[**Implantar esta solução**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="f1c07-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="f1c07-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="f1c07-109">![[0]][0]</span></span>

<span data-ttu-id="f1c07-110">*Baixar um [arquivo Visio][visio-download] dessa arquitetura*</span><span class="sxs-lookup"><span data-stu-id="f1c07-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="f1c07-111">Os benefícios dessa topologia incluem:</span><span class="sxs-lookup"><span data-stu-id="f1c07-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="f1c07-112">**Redução de custos** com a centralização de serviços que podem ser compartilhados por diversas cargas de trabalho, tais como NVAs (soluções de virtualização de rede) e servidores DNS, em um único local.</span><span class="sxs-lookup"><span data-stu-id="f1c07-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="f1c07-113">**Superar os limites de assinaturas** emparelhando VNets de assinaturas diferentes com o hub central.</span><span class="sxs-lookup"><span data-stu-id="f1c07-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="f1c07-114">**Separação de preocupações** entre TI central (SecOps, InfraOps) e cargas de trabalho (DevOps).</span><span class="sxs-lookup"><span data-stu-id="f1c07-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="f1c07-115">Alguns usos típicos dessa arquitetura:</span><span class="sxs-lookup"><span data-stu-id="f1c07-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="f1c07-116">As cargas de trabalho implantadas em diferentes ambientes, como desenvolvimento, teste e produção, que exigem serviços compartilhados como DNS, IDS, NTP ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="f1c07-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="f1c07-117">Os serviços compartilhados são colocados na VNet do hub, enquanto cada ambiente é implantado em um spoke para manter o isolamento.</span><span class="sxs-lookup"><span data-stu-id="f1c07-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="f1c07-118">Cargas de trabalho que não exigem conectividade entre si, mas precisam de acesso aos serviços compartilhados.</span><span class="sxs-lookup"><span data-stu-id="f1c07-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="f1c07-119">Empresas que exigem o controle centralizado sobre aspectos de segurança, como um firewall no hub como uma DMZ e gerenciamento separado para cargas de trabalho em cada spoke.</span><span class="sxs-lookup"><span data-stu-id="f1c07-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="f1c07-120">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="f1c07-120">Architecture</span></span>

<span data-ttu-id="f1c07-121">A arquitetura consiste nos componentes a seguir.</span><span class="sxs-lookup"><span data-stu-id="f1c07-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="f1c07-122">**Rede local**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-122">**On-premises network**.</span></span> <span data-ttu-id="f1c07-123">Uma rede de área local privada em execução dentro de uma organização.</span><span class="sxs-lookup"><span data-stu-id="f1c07-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="f1c07-124">**Dispositivo VPN**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-124">**VPN device**.</span></span> <span data-ttu-id="f1c07-125">Um dispositivo ou serviço que fornece conectividade externa para a rede local.</span><span class="sxs-lookup"><span data-stu-id="f1c07-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="f1c07-126">O dispositivo VPN pode ser um dispositivo de hardware ou uma solução de software, como o RRAS (Serviço de Acesso Remoto e Roteamento) do Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="f1c07-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="f1c07-127">Para obter uma lista de dispositivos de VPN com suporte e informações sobre como configurar dispositivos de VPN selecionados para se conectar ao Azure, consulte [Sobre dispositivos VPN para conexões de Gateway de VPN Site a Site][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="f1c07-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="f1c07-128">**Gateway de rede virtual de VPN ou gateway de ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="f1c07-129">O gateway de rede virtual permite que a VNet se conecte ao dispositivo VPN ou ao circuito de ExpressRoute, usado para conectividade com a rede local.</span><span class="sxs-lookup"><span data-stu-id="f1c07-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="f1c07-130">Para obter mais informações, consulte [Conectar uma rede local a uma rede virtual do Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="f1c07-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="f1c07-131">Os scripts de implantação dessa arquitetura de referência usam um gateway VPN para conectividade e uma VNet no Azure para simular a rede local.</span><span class="sxs-lookup"><span data-stu-id="f1c07-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="f1c07-132">**VNet do hub**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-132">**Hub VNet**.</span></span> <span data-ttu-id="f1c07-133">VNet do Azure usada como o hub na topologia hub-spoke.</span><span class="sxs-lookup"><span data-stu-id="f1c07-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="f1c07-134">O hub é o ponto central de conectividade para sua rede local e um local para hospedar os serviços que podem ser consumidos por diferentes cargas de trabalho hospedadas nas VNets do spoke.</span><span class="sxs-lookup"><span data-stu-id="f1c07-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="f1c07-135">**Sub-rede do gateway**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-135">**Gateway subnet**.</span></span> <span data-ttu-id="f1c07-136">Os gateways de rede virtual são mantidos na mesma sub-rede.</span><span class="sxs-lookup"><span data-stu-id="f1c07-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="f1c07-137">**Sub-rede serviços compartilhados**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-137">**Shared services subnet**.</span></span> <span data-ttu-id="f1c07-138">Uma sub-rede na VNet do hub usada para hospedar os serviços que podem ser compartilhados entre todos os spokes, como DNS ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="f1c07-138">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="f1c07-139">**VNets de spoke**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-139">**Spoke VNets**.</span></span> <span data-ttu-id="f1c07-140">Uma ou mais VNets do Azure que são usadas como spokes na topologia hub-spoke.</span><span class="sxs-lookup"><span data-stu-id="f1c07-140">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="f1c07-141">Spokes podem ser usados para isolar as cargas de trabalho em suas próprias VNets, gerenciadas separadamente de outros spokes.</span><span class="sxs-lookup"><span data-stu-id="f1c07-141">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="f1c07-142">Cada carga de trabalho pode incluir várias camadas, com várias sub-redes conectadas por meio de balanceadores de carga do Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-142">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="f1c07-143">Para obter mais informações sobre a infraestrutura do aplicativo, consulte [Execução de cargas de trabalho de VM do Windows][windows-vm-ra] e [Execução de cargas de trabalho de VM do Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="f1c07-143">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="f1c07-144">**Emparelhamento VNet**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-144">**VNet peering**.</span></span> <span data-ttu-id="f1c07-145">Duas VNets na mesma região do Azure podem ser conectadas usando uma [conexão de emparelhamento][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="f1c07-145">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="f1c07-146">Conexões de emparelhamento são conexões não transitivas de baixa latência entre VNets.</span><span class="sxs-lookup"><span data-stu-id="f1c07-146">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="f1c07-147">Quando emparelhadas, as VNets trocam tráfego usando o backbone do Azure sem precisar de um roteador.</span><span class="sxs-lookup"><span data-stu-id="f1c07-147">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="f1c07-148">Em uma topologia de rede hub-spoke, é necessário usar o emparelhamento VNet para conectar o hub a cada spoke.</span><span class="sxs-lookup"><span data-stu-id="f1c07-148">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="f1c07-149">Este artigo aborda apenas implantações do [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mas também é possível conectar uma VNet clássica a uma VNet do Resource Manager na mesma assinatura.</span><span class="sxs-lookup"><span data-stu-id="f1c07-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="f1c07-150">Dessa forma, seus spokes podem hospedar implantações clássicas e ainda aproveitar os serviços compartilhados no hub.</span><span class="sxs-lookup"><span data-stu-id="f1c07-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>


## <a name="recommendations"></a><span data-ttu-id="f1c07-151">Recomendações</span><span class="sxs-lookup"><span data-stu-id="f1c07-151">Recommendations</span></span>

<span data-ttu-id="f1c07-152">As seguintes recomendações aplicam-se à maioria dos cenários.</span><span class="sxs-lookup"><span data-stu-id="f1c07-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="f1c07-153">Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.</span><span class="sxs-lookup"><span data-stu-id="f1c07-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="f1c07-154">Grupos de recursos</span><span class="sxs-lookup"><span data-stu-id="f1c07-154">Resource groups</span></span>

<span data-ttu-id="f1c07-155">A VNet do hub e a VNet de cada spoke, podem ser implementadas em diferentes grupos de recursos, e até mesmo em assinaturas diferentes, contanto que elas pertençam ao mesmo locatário do Azure AD (Azure Active Directory) na mesma região do Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="f1c07-156">Isso possibilita o gerenciamento descentralizado de cada carga de trabalho, compartilhando os serviços mantidos na VNet do hub.</span><span class="sxs-lookup"><span data-stu-id="f1c07-156">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="f1c07-157">VNet e GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="f1c07-157">VNet and GatewaySubnet</span></span>

<span data-ttu-id="f1c07-158">Crie uma sub-rede denominada *GatewaySubnet* com um intervalo de endereços de /27.</span><span class="sxs-lookup"><span data-stu-id="f1c07-158">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="f1c07-159">Essa sub-rede é necessária para o gateway de rede virtual.</span><span class="sxs-lookup"><span data-stu-id="f1c07-159">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="f1c07-160">Alocar 32 endereços para esta sub-rede ajudará a evitar que as limitações de tamanho de gateway sejam atingidas no futuro.</span><span class="sxs-lookup"><span data-stu-id="f1c07-160">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="f1c07-161">Para obter mais informações sobre como configurar o gateway, consulte as seguintes arquiteturas de referência, dependendo do seu tipo de conexão:</span><span class="sxs-lookup"><span data-stu-id="f1c07-161">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="f1c07-162">[Rede híbrida usando o ExpressRoute][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="f1c07-162">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="f1c07-163">[Rede híbrida usando um gateway de VPN][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="f1c07-163">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="f1c07-164">Para maior disponibilidade, você pode usar o ExpressRoute e uma VPN para failover.</span><span class="sxs-lookup"><span data-stu-id="f1c07-164">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="f1c07-165">Consulte [Conectar uma rede local ao Azure usando o ExpressRoute com failover de VPN][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="f1c07-165">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="f1c07-166">Uma topologia hub-spoke também pode ser usada sem um gateway, se você não precisar de conectividade com a rede local.</span><span class="sxs-lookup"><span data-stu-id="f1c07-166">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="f1c07-167">Emparelhamento VNet</span><span class="sxs-lookup"><span data-stu-id="f1c07-167">VNet peering</span></span>

<span data-ttu-id="f1c07-168">O emparelhamento VNet é uma relação não transitiva entre duas VNets.</span><span class="sxs-lookup"><span data-stu-id="f1c07-168">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="f1c07-169">Se você precisar que os spokes se conectem entre si, considere adicionar uma conexão de emparelhamento separada entre os spokes.</span><span class="sxs-lookup"><span data-stu-id="f1c07-169">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="f1c07-170">No entanto, se houver vários spokes que precisam se conectar entre si, você ficará sem conexões de emparelhamento possíveis rapidamente devido à [limitação do número de emparelhamentos de VNet por VNet][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="f1c07-170">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="f1c07-171">Nesse cenário, considere usar UDRs (rotas definidas pelo usuário) para forçar o tráfego destinado a um spoke a ser enviado para uma NVA que atua como um roteador na VNet do hub.</span><span class="sxs-lookup"><span data-stu-id="f1c07-171">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="f1c07-172">Isso permitirá que os spokes se conectem entre si.</span><span class="sxs-lookup"><span data-stu-id="f1c07-172">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="f1c07-173">Você também pode configurar os spokes para usar o gateway de VNet do hub para se comunicar com redes remotas.</span><span class="sxs-lookup"><span data-stu-id="f1c07-173">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="f1c07-174">Para permitir que o tráfego de gateway flua do spoke para o hub e se conecte a redes remotas, é necessário fazer o seguinte:</span><span class="sxs-lookup"><span data-stu-id="f1c07-174">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="f1c07-175">Configure a conexão de emparelhamento VNet no hub para **permitir trânsito de gateways**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-175">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="f1c07-176">Configure a conexão de emparelhamento VNet em cada spoke para **usar os gateways remotos**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-176">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="f1c07-177">Configure todas as conexões de emparelhamento VNet para **permitir tráfego encaminhado**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-177">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="f1c07-178">Considerações</span><span class="sxs-lookup"><span data-stu-id="f1c07-178">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="f1c07-179">Conectividade de spoke</span><span class="sxs-lookup"><span data-stu-id="f1c07-179">Spoke connectivity</span></span>

<span data-ttu-id="f1c07-180">Se você precisar de conectividade entre spokes, considere implementar uma NVA para roteamento no hub e usar UDRs no spoke para encaminhar o tráfego para o hub.</span><span class="sxs-lookup"><span data-stu-id="f1c07-180">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="f1c07-181">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="f1c07-181">![[2]][2]</span></span>

<span data-ttu-id="f1c07-182">Nesse cenário, você precisa configurar todas as conexões de emparelhamento para **permitir tráfego encaminhado**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-182">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="f1c07-183">Superar os limites de emparelhamento VNet</span><span class="sxs-lookup"><span data-stu-id="f1c07-183">Overcoming VNet peering limits</span></span>

<span data-ttu-id="f1c07-184">Não deixe de considerar a [limitação do número de emparelhamentos de VNet por VNet][vnet-peering-limit] no Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-184">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="f1c07-185">Se você precisar de mais spokes do que o permitido pelo limite, considere criar uma topologia hub-spoke-hub-spoke, na qual o primeiro nível de spokes também funciona como hubs.</span><span class="sxs-lookup"><span data-stu-id="f1c07-185">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="f1c07-186">O diagrama a seguir mostra essa abordagem.</span><span class="sxs-lookup"><span data-stu-id="f1c07-186">The following diagram shows this approach.</span></span>

<span data-ttu-id="f1c07-187">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="f1c07-187">![[3]][3]</span></span>

<span data-ttu-id="f1c07-188">Além disso, considere quais serviços compartilhados no hub, para garantir que o hub seja dimensionado para um número maior de spokes.</span><span class="sxs-lookup"><span data-stu-id="f1c07-188">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="f1c07-189">Por exemplo, se seu hub fornecer serviços de firewall, considere os limites de largura de banda da sua solução de firewall ao adicionar vários spokes.</span><span class="sxs-lookup"><span data-stu-id="f1c07-189">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="f1c07-190">Pode ser útil mover alguns desses serviços compartilhados para um segundo nível de hubs.</span><span class="sxs-lookup"><span data-stu-id="f1c07-190">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="f1c07-191">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="f1c07-191">Deploy the solution</span></span>

<span data-ttu-id="f1c07-192">Uma implantação para essa arquitetura está disponível no [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="f1c07-192">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="f1c07-193">Ela usa VMs Ubuntu em cada VNet para testar a conectividade.</span><span class="sxs-lookup"><span data-stu-id="f1c07-193">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="f1c07-194">Não há nenhum serviço hospedado na sub-rede **shared-services** na **VNet do hub**.</span><span class="sxs-lookup"><span data-stu-id="f1c07-194">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f1c07-195">pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="f1c07-195">Prerequisites</span></span>

<span data-ttu-id="f1c07-196">Antes de implantar a arquitetura de referência para sua própria assinatura, você deve executar as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="f1c07-196">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="f1c07-197">Clone, crie fork ou baixe o arquivo zip para as [arquiteturas de referência AzureCAT][ref-arch-repo] no repositório GitHub.</span><span class="sxs-lookup"><span data-stu-id="f1c07-197">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="f1c07-198">Se você preferir usar a CLI do Azure, verifique se a CLI do Azure 2.0 está instalada no computador.</span><span class="sxs-lookup"><span data-stu-id="f1c07-198">If you prefer to use the Azure CLI, make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="f1c07-199">Para instalar a CLI, siga as instruções em [Instalar a CLI do Azure 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="f1c07-199">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="f1c07-200">Se você preferir usar o PowerShell, verifique se você tem o módulo do PowerShell mais recente para Azure instalado no computador.</span><span class="sxs-lookup"><span data-stu-id="f1c07-200">If you prefer to use PowerShell, make sure you have the latest PowerShell module for Azure installed on you computer.</span></span> <span data-ttu-id="f1c07-201">Para instalar o módulo do Azure PowerShell mais recente, siga as instruções em [Instalar o PowerShell para Azure][azure-powershell].</span><span class="sxs-lookup"><span data-stu-id="f1c07-201">To install the latest Azure PowerShell module, follow the instructions in [Install PowerShell for Azure][azure-powershell].</span></span>

4. <span data-ttu-id="f1c07-202">Em um prompt de comando, bash prompt ou prompt do PowerShell, faça logon na sua conta do Azure usando um dos comandos abaixo e siga os prompts.</span><span class="sxs-lookup"><span data-stu-id="f1c07-202">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="f1c07-203">Implantar o datacenter local simulado</span><span class="sxs-lookup"><span data-stu-id="f1c07-203">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="f1c07-204">Para implantar o datacenter local simulado como uma VNet do Azure, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="f1c07-204">To deploy the simulated on-premises datacenter as an Azure VNet, perform the following steps.</span></span>

1. <span data-ttu-id="f1c07-205">Navegue até a pasta `hybrid-networking\hub-spoke\onprem` do repositório que você baixou na etapa de pré-requisitos acima.</span><span class="sxs-lookup"><span data-stu-id="f1c07-205">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f1c07-206">Abra o arquivo `onprem.vm.parameters.json` e insira um nome de usuário e senha entre aspas nas linhas 11 e 12, conforme mostrado abaixo e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="f1c07-206">Open the `onprem.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="f1c07-207">Execute o bash ou o comando do PowerShell abaixo para implantar o ambiente local simulado como uma VNet no Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-207">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="f1c07-208">Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-208">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="f1c07-209">Se você decidir usar um nome do grupo de recursos distinto (diferente de `ra-onprem-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="f1c07-209">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="f1c07-210">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="f1c07-210">Wait for the deployment to finish.</span></span> <span data-ttu-id="f1c07-211">Essa implantação cria uma rede virtual, uma máquina virtual que executa Ubuntu e um gateway de VPN.</span><span class="sxs-lookup"><span data-stu-id="f1c07-211">This deployment creates a virtual network, a virtual machine running Ubuntu, and a VPN gateway.</span></span> <span data-ttu-id="f1c07-212">A criação do gateway de VPN pode levar mais de 40 minutos para ser concluída.</span><span class="sxs-lookup"><span data-stu-id="f1c07-212">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="f1c07-213">VNet do hub do Azure</span><span class="sxs-lookup"><span data-stu-id="f1c07-213">Azure hub VNet</span></span>

<span data-ttu-id="f1c07-214">Para implantar a VNet de hub e conectar-se à VNet local simulada criada acima, execute as seguintes etapas.</span><span class="sxs-lookup"><span data-stu-id="f1c07-214">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="f1c07-215">Navegue até a pasta `hybrid-networking\hub-spoke\hub` do repositório que você baixou na etapa de pré-requisitos acima.</span><span class="sxs-lookup"><span data-stu-id="f1c07-215">Navigate to the `hybrid-networking\hub-spoke\hub` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f1c07-216">Abra o arquivo `hub.vm.parameters.json` e insira um nome de usuário e senha entre aspas nas linhas 11 e 12, conforme mostrado abaixo e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="f1c07-216">Open the `hub.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="f1c07-217">Abra o arquivo `hub.gateway.parameters.json` e insira uma chave compartilhada entre as aspas na linha 23, conforme mostrado abaixo e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="f1c07-217">Open the `hub.gateway.parameters.json` file and enter a shared key between the quotes in line 23, as shown below, then save the file.</span></span> <span data-ttu-id="f1c07-218">Anote esse valor, pois você precisará usá-lo mais tarde na implantação.</span><span class="sxs-lookup"><span data-stu-id="f1c07-218">Keep a note of this value, you will need to use it later in the deployment.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="f1c07-219">Execute o bash ou o comando do PowerShell abaixo para implantar o ambiente local simulado como uma VNet no Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-219">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="f1c07-220">Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-220">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="f1c07-221">Se você decidir usar um nome do grupo de recursos distinto (diferente de `ra-hub-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="f1c07-221">If you decide to use a different resource group name (other than `ra-hub-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="f1c07-222">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="f1c07-222">Wait for the deployment to finish.</span></span> <span data-ttu-id="f1c07-223">Essa implantação cria uma rede virtual, uma máquina virtual que executa Ubuntu, um gateway de VPN e uma conexão com o gateway criado na seção anterior.</span><span class="sxs-lookup"><span data-stu-id="f1c07-223">This deployment creates a virtual network, a virtual machine running Ubuntu, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="f1c07-224">A criação do gateway de VPN pode levar mais de 40 minutos para ser concluída.</span><span class="sxs-lookup"><span data-stu-id="f1c07-224">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="connection-from-on-premises-to-the-hub"></a><span data-ttu-id="f1c07-225">Conexão do local para o hub</span><span class="sxs-lookup"><span data-stu-id="f1c07-225">Connection from on-premises to the hub</span></span>

<span data-ttu-id="f1c07-226">Para conectar-se do datacenter local simulado para a VNet do hub, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="f1c07-226">To connect from the simulated on-premises datacenter to the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="f1c07-227">Navegue até a pasta `hybrid-networking\hub-spoke\onprem` do repositório que você baixou na etapa de pré-requisitos acima.</span><span class="sxs-lookup"><span data-stu-id="f1c07-227">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f1c07-228">Abra o arquivo `onprem.connection.parameters.json` e insira uma chave compartilhada entre as aspas na linha 9, conforme mostrado abaixo e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="f1c07-228">Open the `onprem.connection.parameters.json` file and enter a shared key between the quotes in line 9, as shown below, then save the file.</span></span> <span data-ttu-id="f1c07-229">Esse valor de chave compartilhada deve ser o mesmo usado no gateway local implantado anteriormente.</span><span class="sxs-lookup"><span data-stu-id="f1c07-229">This shared key value must be the same used in the on-premises gateway you deployed previously.</span></span>

  ```bash
  "sharedKey": "",
  ```

3. <span data-ttu-id="f1c07-230">Execute o bash ou o comando do PowerShell abaixo para implantar o ambiente local simulado como uma VNet no Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-230">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="f1c07-231">Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-231">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="f1c07-232">Se você decidir usar um nome do grupo de recursos distinto (diferente de `ra-onprem-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="f1c07-232">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="f1c07-233">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="f1c07-233">Wait for the deployment to finish.</span></span> <span data-ttu-id="f1c07-234">Essa implantação cria uma conexão entre a VNet usada para simular um datacenter local e a VNet do hub.</span><span class="sxs-lookup"><span data-stu-id="f1c07-234">This deployment creates a connection between the VNet used to simulate an on-premises datacenter, and the hub VNet.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="f1c07-235">VNets de spoke do Azure</span><span class="sxs-lookup"><span data-stu-id="f1c07-235">Azure spoke VNets</span></span>

<span data-ttu-id="f1c07-236">Para implantar as VNets de spoke e conectar-se à VNet do hub criada acima, execute as seguintes etapas.</span><span class="sxs-lookup"><span data-stu-id="f1c07-236">To deploy the spoke VNets, and connect to the hub VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="f1c07-237">Alterne para a pasta `hybrid-networking\hub-spoke\spokes` do repositório que você baixou na etapa de pré-requisitos acima.</span><span class="sxs-lookup"><span data-stu-id="f1c07-237">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f1c07-238">Abra o arquivo `spoke1.web.parameters.json` e insira um nome de usuário e senha entre aspas nas linhas 53 e 54, conforme mostrado abaixo e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="f1c07-238">Open the `spoke1.web.parameters.json` file and enter a username and password between the quotes in line 53 and 54, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="f1c07-239">Repita a etapa anterior para o arquivo `spoke2.web.parameters.json`.</span><span class="sxs-lookup"><span data-stu-id="f1c07-239">Repeat the previous step for file `spoke2.web.parameters.json`.</span></span>

4. <span data-ttu-id="f1c07-240">Execute o bash ou o comando do PowerShell abaixo para implantar o primeiro spoke e conectá-lo ao hub.</span><span class="sxs-lookup"><span data-stu-id="f1c07-240">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="f1c07-241">Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-241">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > <span data-ttu-id="f1c07-242">Se você decidir usar um nome do grupo de recursos distinto (diferente de `ra-spoke1-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="f1c07-242">If you decide to use a different resource group name (other than `ra-spoke1-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="f1c07-243">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="f1c07-243">Wait for the deployment to finish.</span></span> <span data-ttu-id="f1c07-244">Essa implantação cria uma rede virtual, um balanceador de carga, três máquinas virtuais que executam Ubuntu e Apache e uma conexão de emparelhamento VNet para a VNet de hub criada na seção anterior.</span><span class="sxs-lookup"><span data-stu-id="f1c07-244">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="f1c07-245">A implantação pode levar mais de 20 minutos.</span><span class="sxs-lookup"><span data-stu-id="f1c07-245">This deployment may take over 20 minutes.</span></span>

6. <span data-ttu-id="f1c07-246">Execute o bash ou o comando do PowerShell abaixo para implantar o primeiro spoke e conectá-lo ao hub.</span><span class="sxs-lookup"><span data-stu-id="f1c07-246">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="f1c07-247">Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-247">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > <span data-ttu-id="f1c07-248">Se você decidir usar um nome do grupo de recursos distinto (diferente de `ra-spoke2-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="f1c07-248">If you decide to use a different resource group name (other than `ra-spoke2-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="f1c07-249">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="f1c07-249">Wait for the deployment to finish.</span></span> <span data-ttu-id="f1c07-250">Essa implantação cria uma rede virtual, um balanceador de carga, três máquinas virtuais que executam Ubuntu e Apache e uma conexão de emparelhamento VNet para a VNet de hub criada na seção anterior.</span><span class="sxs-lookup"><span data-stu-id="f1c07-250">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="f1c07-251">A implantação pode levar mais de 20 minutos.</span><span class="sxs-lookup"><span data-stu-id="f1c07-251">This deployment may take over 20 minutes.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="f1c07-252">Emparelhamento VNet do hub do Azure para VNets de spoke</span><span class="sxs-lookup"><span data-stu-id="f1c07-252">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="f1c07-253">Para implantar as conexões de emparelhamento VNet para a VNet do hub, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="f1c07-253">To deploy the VNet peering connections for the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="f1c07-254">Alterne para a pasta `hybrid-networking\hub-spoke\hub` do repositório que você baixou na etapa de pré-requisitos acima.</span><span class="sxs-lookup"><span data-stu-id="f1c07-254">Switch to the `hybrid-networking\hub-spoke\hub` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f1c07-255">Execute o bash ou o comando do PowerShell abaixo para implantar a conexão de emparelhamento para o primeiro spoke.</span><span class="sxs-lookup"><span data-stu-id="f1c07-255">Run the bash or PowerShell command below to deploy the peering connection to the first spoke.</span></span> <span data-ttu-id="f1c07-256">Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-256">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. <span data-ttu-id="f1c07-257">Execute o bash ou o comando do PowerShell abaixo para implantar a conexão de emparelhamento para o segundo spoke.</span><span class="sxs-lookup"><span data-stu-id="f1c07-257">Run the bash or PowerShell command below to deploy the peering connection to the second spoke.</span></span> <span data-ttu-id="f1c07-258">Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.</span><span class="sxs-lookup"><span data-stu-id="f1c07-258">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a><span data-ttu-id="f1c07-259">Testar a conectividade</span><span class="sxs-lookup"><span data-stu-id="f1c07-259">Test connectivity</span></span>

<span data-ttu-id="f1c07-260">Para verificar se a topologia hub-spoke conectada a uma implantação de datacenter local funcionou, execute estas etapas.</span><span class="sxs-lookup"><span data-stu-id="f1c07-260">To verify that the hub-spoke topology connected to an on-premises datacenter deployment worked, follow these steps.</span></span>

1. <span data-ttu-id="f1c07-261">No [Portal do Azure][portal], conecte-se à sua assinatura e navegue até a máquina virtual `ra-onprem-vm1` no grupo de recursos `ra-onprem-rg`.</span><span class="sxs-lookup"><span data-stu-id="f1c07-261">From the [Azure portal][portal], connect to your subscription, and navigate to the `ra-onprem-vm1` virtual machine in the `ra-onprem-rg` resource group.</span></span>

2. <span data-ttu-id="f1c07-262">No folha `Overview`, anote o `Public IP address` da VM.</span><span class="sxs-lookup"><span data-stu-id="f1c07-262">In the `Overview` blade, note the `Public IP address` for the VM.</span></span>

3. <span data-ttu-id="f1c07-263">Use um cliente SSH para se conectar ao endereço IP que você anotou acima usando o nome de usuário e senha especificados durante a implantação.</span><span class="sxs-lookup"><span data-stu-id="f1c07-263">Use an SSH client to connect to the IP address you noted above using the user name and password you specified during deployment.</span></span>

4. <span data-ttu-id="f1c07-264">No prompt de comando na VM à qual você se conectou, execute o comando abaixo para testar a conectividade da VNet local para VNet Spoke1.</span><span class="sxs-lookup"><span data-stu-id="f1c07-264">From the command prompt on the VM you connected to, run the command below to test connectivity from the on-premises VNet to the Spoke1 VNet.</span></span>

  ```bash
  ping 10.1.1.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
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
