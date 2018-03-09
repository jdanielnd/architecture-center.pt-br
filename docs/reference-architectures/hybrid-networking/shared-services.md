---
title: "Implementação de uma topologia de rede hub-spoke com serviços compartilhados no Azure"
description: "Como implementar uma topologia de rede hub-spoke com serviços compartilhados no Azure."
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: c0fb1d1ddd7c70ed914d58e7c73b10475b91aedf
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/08/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="d6985-103">Implementar uma topologia de rede hub-spoke com serviços compartilhados no Azure</span><span class="sxs-lookup"><span data-stu-id="d6985-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="d6985-104">Essa arquitetura de referência se baseia na arquitetura de referência do [hub-spoke] [ guidance-hub-spoke] para incluir serviços compartilhados no hub que pode ser consumido por todos os spokes.</span><span class="sxs-lookup"><span data-stu-id="d6985-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="d6985-105">Como uma primeira etapa para migrar um datacenter para a nuvem, e criando um [datacenter virtual], os primeiros serviços que você precisa compartilhar são identidade e segurança.</span><span class="sxs-lookup"><span data-stu-id="d6985-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="d6985-106">Esta arquitetura de referência mostra como estender seus serviços do Active Directory de seu datacenter local para o do Azure e como adicionar uma solução de virtualização de rede (NVA) que pode agir como um firewall, em uma topologia hub-spoke.</span><span class="sxs-lookup"><span data-stu-id="d6985-106">This reference archiecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="d6985-107">[**Implantar esta solução**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="d6985-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="d6985-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="d6985-108">![[0]][0]</span></span>

<span data-ttu-id="d6985-109">*Baixar um [arquivo Visio][visio-download] dessa arquitetura*</span><span class="sxs-lookup"><span data-stu-id="d6985-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="d6985-110">Os benefícios dessa topologia incluem:</span><span class="sxs-lookup"><span data-stu-id="d6985-110">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="d6985-111">**Redução de custos** com a centralização de serviços que podem ser compartilhados por diversas cargas de trabalho, tais como NVAs (soluções de virtualização de rede) e servidores DNS, em um único local.</span><span class="sxs-lookup"><span data-stu-id="d6985-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="d6985-112">**Superar os limites de assinaturas** emparelhando VNets de assinaturas diferentes com o hub central.</span><span class="sxs-lookup"><span data-stu-id="d6985-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="d6985-113">**Separação de preocupações** entre TI central (SecOps, InfraOps) e cargas de trabalho (DevOps).</span><span class="sxs-lookup"><span data-stu-id="d6985-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="d6985-114">Alguns usos típicos dessa arquitetura:</span><span class="sxs-lookup"><span data-stu-id="d6985-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="d6985-115">As cargas de trabalho implantadas em diferentes ambientes, como desenvolvimento, teste e produção, que exigem serviços compartilhados como DNS, IDS, NTP ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="d6985-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="d6985-116">Os serviços compartilhados são colocados na VNet do hub, enquanto cada ambiente é implantado em um spoke para manter o isolamento.</span><span class="sxs-lookup"><span data-stu-id="d6985-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="d6985-117">Cargas de trabalho que não exigem conectividade entre si, mas precisam de acesso aos serviços compartilhados.</span><span class="sxs-lookup"><span data-stu-id="d6985-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="d6985-118">Empresas que exigem o controle centralizado sobre aspectos de segurança, como um firewall no hub como uma DMZ e gerenciamento separado para cargas de trabalho em cada spoke.</span><span class="sxs-lookup"><span data-stu-id="d6985-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="d6985-119">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="d6985-119">Architecture</span></span>

<span data-ttu-id="d6985-120">A arquitetura consiste nos componentes a seguir.</span><span class="sxs-lookup"><span data-stu-id="d6985-120">The architecture consists of the following components.</span></span>

* <span data-ttu-id="d6985-121">**Rede local**.</span><span class="sxs-lookup"><span data-stu-id="d6985-121">**On-premises network**.</span></span> <span data-ttu-id="d6985-122">Uma rede de área local privada em execução dentro de uma organização.</span><span class="sxs-lookup"><span data-stu-id="d6985-122">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="d6985-123">**Dispositivo VPN**.</span><span class="sxs-lookup"><span data-stu-id="d6985-123">**VPN device**.</span></span> <span data-ttu-id="d6985-124">Um dispositivo ou serviço que fornece conectividade externa para a rede local.</span><span class="sxs-lookup"><span data-stu-id="d6985-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="d6985-125">O dispositivo VPN pode ser um dispositivo de hardware ou uma solução de software, como o RRAS (Serviço de Acesso Remoto e Roteamento) do Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="d6985-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="d6985-126">Para obter uma lista de dispositivos de VPN com suporte e informações sobre como configurar dispositivos de VPN selecionados para se conectar ao Azure, consulte [Sobre dispositivos VPN para conexões de Gateway de VPN Site a Site][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="d6985-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="d6985-127">**Gateway de rede virtual de VPN ou gateway de ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="d6985-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="d6985-128">O gateway de rede virtual permite que a VNet se conecte ao dispositivo VPN ou ao circuito de ExpressRoute, usado para conectividade com a rede local.</span><span class="sxs-lookup"><span data-stu-id="d6985-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="d6985-129">Para obter mais informações, consulte [Conectar uma rede local a uma rede virtual do Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="d6985-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="d6985-130">Os scripts de implantação dessa arquitetura de referência usam um gateway VPN para conectividade e uma VNet no Azure para simular a rede local.</span><span class="sxs-lookup"><span data-stu-id="d6985-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="d6985-131">**VNet do hub**.</span><span class="sxs-lookup"><span data-stu-id="d6985-131">**Hub VNet**.</span></span> <span data-ttu-id="d6985-132">VNet do Azure usada como o hub na topologia hub-spoke.</span><span class="sxs-lookup"><span data-stu-id="d6985-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="d6985-133">O hub é o ponto central de conectividade para sua rede local e um local para hospedar os serviços que podem ser consumidos por diferentes cargas de trabalho hospedadas nas VNets do spoke.</span><span class="sxs-lookup"><span data-stu-id="d6985-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="d6985-134">**Sub-rede do gateway**.</span><span class="sxs-lookup"><span data-stu-id="d6985-134">**Gateway subnet**.</span></span> <span data-ttu-id="d6985-135">Os gateways de rede virtual são mantidos na mesma sub-rede.</span><span class="sxs-lookup"><span data-stu-id="d6985-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="d6985-136">**Sub-rede serviços compartilhados**.</span><span class="sxs-lookup"><span data-stu-id="d6985-136">**Shared services subnet**.</span></span> <span data-ttu-id="d6985-137">Uma sub-rede na VNet do hub usada para hospedar os serviços que podem ser compartilhados entre todos os spokes, como DNS ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="d6985-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="d6985-138">**Sub-rede de perímetro**.</span><span class="sxs-lookup"><span data-stu-id="d6985-138">**DMZ subnet**.</span></span> <span data-ttu-id="d6985-139">Uma sub-rede na rede virtual do hub usada para hospedar NVAs que podem agir como dispositivos de segurança, como firewalls.</span><span class="sxs-lookup"><span data-stu-id="d6985-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

* <span data-ttu-id="d6985-140">**VNets de spoke**.</span><span class="sxs-lookup"><span data-stu-id="d6985-140">**Spoke VNets**.</span></span> <span data-ttu-id="d6985-141">Uma ou mais VNets do Azure que são usadas como spokes na topologia hub-spoke.</span><span class="sxs-lookup"><span data-stu-id="d6985-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="d6985-142">Spokes podem ser usados para isolar as cargas de trabalho em suas próprias VNets, gerenciadas separadamente de outros spokes.</span><span class="sxs-lookup"><span data-stu-id="d6985-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="d6985-143">Cada carga de trabalho pode incluir várias camadas, com várias sub-redes conectadas por meio de balanceadores de carga do Azure.</span><span class="sxs-lookup"><span data-stu-id="d6985-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="d6985-144">Para obter mais informações sobre a infraestrutura do aplicativo, consulte [Execução de cargas de trabalho de VM do Windows][windows-vm-ra] e [Execução de cargas de trabalho de VM do Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="d6985-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="d6985-145">**Emparelhamento VNet**.</span><span class="sxs-lookup"><span data-stu-id="d6985-145">**VNet peering**.</span></span> <span data-ttu-id="d6985-146">Duas VNets na mesma região do Azure podem ser conectadas usando uma [conexão de emparelhamento][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="d6985-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="d6985-147">Conexões de emparelhamento são conexões não transitivas de baixa latência entre VNets.</span><span class="sxs-lookup"><span data-stu-id="d6985-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="d6985-148">Quando emparelhadas, as VNets trocam tráfego usando o backbone do Azure sem precisar de um roteador.</span><span class="sxs-lookup"><span data-stu-id="d6985-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="d6985-149">Em uma topologia de rede hub-spoke, é necessário usar o emparelhamento VNet para conectar o hub a cada spoke.</span><span class="sxs-lookup"><span data-stu-id="d6985-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="d6985-150">Este artigo aborda apenas implantações do [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mas também é possível conectar uma VNet clássica a uma VNet do Resource Manager na mesma assinatura.</span><span class="sxs-lookup"><span data-stu-id="d6985-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="d6985-151">Dessa forma, seus spokes podem hospedar implantações clássicas e ainda aproveitar os serviços compartilhados no hub.</span><span class="sxs-lookup"><span data-stu-id="d6985-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="d6985-152">Recomendações</span><span class="sxs-lookup"><span data-stu-id="d6985-152">Recommendations</span></span>

<span data-ttu-id="d6985-153">Todas as recomendações para a arquitetura de referência do [hub-spoke] [ guidance-hub-spoke] também se aplicam à arquitetura de referência de serviços compartilhados.</span><span class="sxs-lookup"><span data-stu-id="d6985-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span> 

<span data-ttu-id="d6985-154">Além disso, as seguintes recomendações se aplicam para a maioria dos cenários em serviços compartilhados.</span><span class="sxs-lookup"><span data-stu-id="d6985-154">ALso, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="d6985-155">Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.</span><span class="sxs-lookup"><span data-stu-id="d6985-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="d6985-156">Identidade</span><span class="sxs-lookup"><span data-stu-id="d6985-156">Identity</span></span>

<span data-ttu-id="d6985-157">A maioria das empresas tem um ambiente de Active Directory Domain Services (ADDS) em seu datacenter local.</span><span class="sxs-lookup"><span data-stu-id="d6985-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="d6985-158">Para facilitar o gerenciamento de ativos movidos para o Azure de sua rede local que dependem do ADDS, é recomendável hospedar os controladores de domínio ADDS no Azure.</span><span class="sxs-lookup"><span data-stu-id="d6985-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="d6985-159">Se você usar objetos da Política de Grupo que você deseja controlar separadamente para o Azure e seu ambiente local, use um site diferente do AD para cada região do Azure.</span><span class="sxs-lookup"><span data-stu-id="d6985-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="d6985-160">Coloque os controladores de domínio em uma rede virtual central (hub) que pode ser acessada por cargas de trabalho dependentes.</span><span class="sxs-lookup"><span data-stu-id="d6985-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="d6985-161">Segurança</span><span class="sxs-lookup"><span data-stu-id="d6985-161">Security</span></span>

<span data-ttu-id="d6985-162">Ao mover cargas de trabalho do seu ambiente local para o Azure, algumas dessas cargas de trabalho exigem ser hospedadas em máquinas virtuais.</span><span class="sxs-lookup"><span data-stu-id="d6985-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="d6985-163">Por motivos de conformidade, talvez seja necessário impor restrições sobre o tráfego que passa por essas cargas de trabalho.</span><span class="sxs-lookup"><span data-stu-id="d6985-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span> 

<span data-ttu-id="d6985-164">Você pode usar soluções de virtualização de rede (NVAs) no Azure para hospedar tipos diferentes de segurança e de serviços de desempenho.</span><span class="sxs-lookup"><span data-stu-id="d6985-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="d6985-165">Se você estiver atualmente familiarizado com um determinado conjunto de dispositivos locais, é recomendável usar os mesmos dispositivos virtualizados no Azure, onde aplicável.</span><span class="sxs-lookup"><span data-stu-id="d6985-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="d6985-166">Os scripts de implantação para esta arquitetura de referência usam uma VM Ubuntu com encaminhamento de IP ativado para simular uma solução de virtualização de rede.</span><span class="sxs-lookup"><span data-stu-id="d6985-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enbaled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="d6985-167">Considerações</span><span class="sxs-lookup"><span data-stu-id="d6985-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="d6985-168">Superar os limites de emparelhamento VNet</span><span class="sxs-lookup"><span data-stu-id="d6985-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="d6985-169">Não deixe de considerar a [limitação do número de emparelhamentos de VNet por VNet][vnet-peering-limit] no Azure.</span><span class="sxs-lookup"><span data-stu-id="d6985-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="d6985-170">Se você precisar de mais spokes do que o permitido pelo limite, considere criar uma topologia hub-spoke-hub-spoke, na qual o primeiro nível de spokes também funciona como hubs.</span><span class="sxs-lookup"><span data-stu-id="d6985-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="d6985-171">O diagrama a seguir mostra essa abordagem.</span><span class="sxs-lookup"><span data-stu-id="d6985-171">The following diagram shows this approach.</span></span>

<span data-ttu-id="d6985-172">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="d6985-172">![[3]][3]</span></span>

<span data-ttu-id="d6985-173">Além disso, considere quais serviços compartilhados no hub, para garantir que o hub seja dimensionado para um número maior de spokes.</span><span class="sxs-lookup"><span data-stu-id="d6985-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="d6985-174">Por exemplo, se seu hub fornecer serviços de firewall, considere os limites de largura de banda da sua solução de firewall ao adicionar vários spokes.</span><span class="sxs-lookup"><span data-stu-id="d6985-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="d6985-175">Pode ser útil mover alguns desses serviços compartilhados para um segundo nível de hubs.</span><span class="sxs-lookup"><span data-stu-id="d6985-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="d6985-176">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="d6985-176">Deploy the solution</span></span>

<span data-ttu-id="d6985-177">Uma implantação para essa arquitetura está disponível no [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="d6985-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="d6985-178">Ela usa VMs Ubuntu em cada VNet para testar a conectividade.</span><span class="sxs-lookup"><span data-stu-id="d6985-178">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="d6985-179">Não há nenhum serviço hospedado na sub-rede **shared-services** na **VNet do hub**.</span><span class="sxs-lookup"><span data-stu-id="d6985-179">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="d6985-180">pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="d6985-180">Prerequisites</span></span>

<span data-ttu-id="d6985-181">Antes de implantar a arquitetura de referência para sua própria assinatura, você deve executar as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="d6985-181">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="d6985-182">Clone, crie fork ou baixe o arquivo zip para as [arquiteturas de referência AzureCAT][ref-arch-repo] no repositório GitHub.</span><span class="sxs-lookup"><span data-stu-id="d6985-182">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="d6985-183">Verifique se a CLI do Azure 2.0 está instalada no computador.</span><span class="sxs-lookup"><span data-stu-id="d6985-183">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="d6985-184">Para obter instruções de instalação da CLI, consulte [Instalar a CLI 2.0 do Azure][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="d6985-184">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="d6985-185">Instale os pacote npm dos [Blocos de construção do Azure][azbb].</span><span class="sxs-lookup"><span data-stu-id="d6985-185">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="d6985-186">Em um prompt de comando, bash prompt ou prompt do PowerShell, faça logon na sua conta do Azure usando o comando abaixo e siga os prompts.</span><span class="sxs-lookup"><span data-stu-id="d6985-186">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="d6985-187">Implantar o datacenter local simulado usand azbb</span><span class="sxs-lookup"><span data-stu-id="d6985-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="d6985-188">Para implantar o datacenter local simulado como uma rede virtual do Azure, siga as seguintes as etapas:</span><span class="sxs-lookup"><span data-stu-id="d6985-188">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="d6985-189">Navegue até a pasta `hybrid-networking\shared-services-stack\` do repositório que você baixou na etapa de pré-requisitos acima.</span><span class="sxs-lookup"><span data-stu-id="d6985-189">Navigate to the `hybrid-networking\shared-services-stack\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="d6985-190">Abra o arquivo `onprem.json` e insira um nome de usuário e senha entre aspas nas linhas 45 e 46, conforme mostrado abaixo e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="d6985-190">Open the `onprem.json` file and enter a username and password between the quotes in line 45 and 46, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="d6985-191">Execute `azbb` para implantar o ambiente simulado local, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="d6985-191">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="d6985-192">Se você decidir usar um nome do grupo de recursos distinto (diferente de `onprem-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="d6985-192">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="d6985-193">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="d6985-193">Wait for the deployment to finish.</span></span> <span data-ttu-id="d6985-194">Essa implantação cria uma rede virtual, uma máquina virtual que executa Windows e um gateway de VPN.</span><span class="sxs-lookup"><span data-stu-id="d6985-194">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="d6985-195">A criação do gateway de VPN pode levar mais de 40 minutos para ser concluída.</span><span class="sxs-lookup"><span data-stu-id="d6985-195">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="d6985-196">VNet do hub do Azure</span><span class="sxs-lookup"><span data-stu-id="d6985-196">Azure hub VNet</span></span>

<span data-ttu-id="d6985-197">Para implantar a VNet de hub e conectar-se à VNet local simulada criada acima, execute as seguintes etapas.</span><span class="sxs-lookup"><span data-stu-id="d6985-197">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="d6985-198">Abra o arquivo `hub-vnet.json` e insira um nome de usuário e senha entre aspas nas linhas 50 e 51, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="d6985-198">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 50 and 51, as shown below.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="d6985-199">Na linha 52, para `osType`, digite `Windows` ou `Linux` para instalar o Windows Server 2016 Datacenter ou o Ubuntu 16.04 como o sistema operacional para o jumpbox.</span><span class="sxs-lookup"><span data-stu-id="d6985-199">On line 52, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="d6985-200">Insira uma chave compartilhada entre as aspas na linha 83, conforme mostrado abaixo, e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="d6985-200">Enter a shared key between the quotes in line 83, as shown below, then save the file.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="d6985-201">Execute `azbb` para implantar o ambiente simulado local, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="d6985-201">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="d6985-202">Se você decidir usar um nome do grupo de recursos distinto (diferente de `hub-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="d6985-202">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="d6985-203">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="d6985-203">Wait for the deployment to finish.</span></span> <span data-ttu-id="d6985-204">Essa implantação cria uma rede virtual, uma máquina virtual, um gateway de VPN e uma conexão com o gateway criado na seção anterior.</span><span class="sxs-lookup"><span data-stu-id="d6985-204">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="d6985-205">A criação do gateway de VPN pode levar mais de 40 minutos para ser concluída.</span><span class="sxs-lookup"><span data-stu-id="d6985-205">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="adds-in-azure"></a><span data-ttu-id="d6985-206">Exibir no ADDS</span><span class="sxs-lookup"><span data-stu-id="d6985-206">ADDS in Azure</span></span>

<span data-ttu-id="d6985-207">Para implantar os controladores de domínio ADDS no Azure, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="d6985-207">To deploy the ADDS domain controllers in Azure, perform the following steps.</span></span>

1. <span data-ttu-id="d6985-208">Abra o arquivo `hub-adds.json` e insira um nome de usuário e senha entre aspas nas linhas 14 e 15, conforme mostrado abaixo, e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="d6985-208">Open the `hub-adds.json` file and enter a username and password between the quotes in lines 14 and 15, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="d6985-209">Execute `azbb` para implantar os controladores de domínio ADDS, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="d6985-209">Run `azbb` to deploy the ADDS domain controllers as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="d6985-210">Se você decidir usar um nome do grupo de recursos distinto (diferente de `hub-adds-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="d6985-210">If you decide to use a different resource group name (other than `hub-adds-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

  > [!NOTE]
  > <span data-ttu-id="d6985-211">Esta parte da implantação pode levar vários minutos, pois requer a união das duas máquinas virtuais ao domínio hospedado no datacenter local simulado, instalando o AD DS neles.</span><span class="sxs-lookup"><span data-stu-id="d6985-211">This part of the deployment may take several minutes, since it requires joining the two VMs to the domain hosted int he simulated on-premises datacenter, then installing AD DS on them.</span></span>

### <a name="nva"></a><span data-ttu-id="d6985-212">NVA</span><span class="sxs-lookup"><span data-stu-id="d6985-212">NVA</span></span>

<span data-ttu-id="d6985-213">Para implantar uma NVA na sub-rede `dmz`, execute as seguintes etapas:</span><span class="sxs-lookup"><span data-stu-id="d6985-213">To deploy an NVA in the `dmz` subnet, perform the following steps:</span></span>

1. <span data-ttu-id="d6985-214">Abra o arquivo `hub-nva.json` e insira um nome de usuário e senha entre aspas nas linhas 13 e 14, conforme mostrado abaixo, e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="d6985-214">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. <span data-ttu-id="d6985-215">Execute `azbb` para implantar a VM NVA e as rotas de usuário definidas.</span><span class="sxs-lookup"><span data-stu-id="d6985-215">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="d6985-216">Se você decidir usar um nome do grupo de recursos distinto (diferente de `hub-nva-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="d6985-216">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="d6985-217">VNets de spoke do Azure</span><span class="sxs-lookup"><span data-stu-id="d6985-217">Azure spoke VNets</span></span>

<span data-ttu-id="d6985-218">Para implantar as redes virtuais spoke, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="d6985-218">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="d6985-219">Abra o arquivo `spoke1.json` e insira um nome de usuário e senha entre aspas nas linhas 52 e 53, conforme mostrado abaixo, e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="d6985-219">Open the `spoke1.json` file and enter a username and password between the quotes in lines 52 and 53, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="d6985-220">Na linha 54, para `osType`, digite `Windows` ou `Linux` para instalar o Windows Server 2016 Datacenter ou o Ubuntu 16.04 como o sistema operacional para o jumpbox.</span><span class="sxs-lookup"><span data-stu-id="d6985-220">On line 54, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="d6985-221">Execute `azbb` para implantar o primeiro ambiente de rede virtual spoke, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="d6985-221">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="d6985-222">Se você decidir usar um nome do grupo de recursos distinto (diferente de `spoke1-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="d6985-222">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

3. <span data-ttu-id="d6985-223">Repita a etapa 1 acima para o arquivo `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="d6985-223">Repeat step 1 above for file `spoke2.json`.</span></span>

4. <span data-ttu-id="d6985-224">Execute `azbb` para implantar o segundo ambiente de rede virtual spoke, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="d6985-224">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="d6985-225">Se você decidir usar um nome do grupo de recursos distinto (diferente de `spoke2-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="d6985-225">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="d6985-226">Emparelhamento VNet do hub do Azure para VNets de spoke</span><span class="sxs-lookup"><span data-stu-id="d6985-226">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="d6985-227">Para criar uma conexão de emparelhamento da rede virtual do hub para as redes virtuais spoke, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="d6985-227">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="d6985-228">Abra o arquivo `hub-vnet-peering.json` e verifique se o nome do grupo de recursos e o nome da rede virtual para cada um dos emparelhamentos de rede virtual, começando na linha 29, estão corretos.</span><span class="sxs-lookup"><span data-stu-id="d6985-228">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="d6985-229">Execute `azbb` para implantar o primeiro ambiente de rede virtual spoke, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="d6985-229">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > <span data-ttu-id="d6985-230">Se você decidir usar um nome do grupo de recursos distinto (diferente de `hub-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="d6985-230">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

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
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Topologia de serviços compartilhados no Azure"
[3]: ./images/hub-spokehub-spoke.svg "Topologia hub-spoke-hub-spoke no Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
