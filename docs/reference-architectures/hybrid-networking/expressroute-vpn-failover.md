---
title: Implementar de uma arquitetura de rede híbrida altamente disponível
description: Como implementar uma arquitetura de rede site a site segura que abranja uma rede virtual do Azure e uma rede local conectada usando o ExpressRoute com failover de gateway VPN.
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
ms.openlocfilehash: 31ed1dbf59c4fa2b7fa86b9ceb2fed7b36e75c8c
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428815"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a><span data-ttu-id="538c4-103">Conectar uma rede local ao Azure usando o ExpressRoute com failover de VPN</span><span class="sxs-lookup"><span data-stu-id="538c4-103">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span>

<span data-ttu-id="538c4-104">Essa arquitetura de referência mostra como conectar uma rede local a uma VNet (rede virtual) do Azure usando o ExpressRoute, com uma VPN (rede virtual privada) site a site como uma conexão de failover.</span><span class="sxs-lookup"><span data-stu-id="538c4-104">This reference architecture shows how to connect an on-premises network to an Azure virtual network (VNet) using ExpressRoute, with a site-to-site virtual private network (VPN) as a failover connection.</span></span> <span data-ttu-id="538c4-105">O tráfego flui entre a rede local e a rede virtual do Azure por meio de uma conexão do ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="538c4-105">Traffic flows between the on-premises network and the Azure VNet through an ExpressRoute connection.</span></span> <span data-ttu-id="538c4-106">Se houver uma perda de conectividade no circuito do ExpressRoute, o tráfego será roteado por um túnel VPN IPsec.</span><span class="sxs-lookup"><span data-stu-id="538c4-106">If there is a loss of connectivity in the ExpressRoute circuit, traffic is routed through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="538c4-107">**Implante essa solução**.</span><span class="sxs-lookup"><span data-stu-id="538c4-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="538c4-108">Observe que, se o circuito do ExpressRoute não estiver disponível, a rota VPN administrará apenas conexões de emparelhamento privadas.</span><span class="sxs-lookup"><span data-stu-id="538c4-108">Note that if the ExpressRoute circuit is unavailable, the VPN route will only handle private peering connections.</span></span> <span data-ttu-id="538c4-109">Conexões de emparelhamento público e da Microsoft passarão pela Internet.</span><span class="sxs-lookup"><span data-stu-id="538c4-109">Public peering and Microsoft peering connections will pass over the Internet.</span></span> 

<span data-ttu-id="538c4-110">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="538c4-110">![[0]][0]</span></span>

<span data-ttu-id="538c4-111">*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*</span><span class="sxs-lookup"><span data-stu-id="538c4-111">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="538c4-112">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="538c4-112">Architecture</span></span> 

<span data-ttu-id="538c4-113">A arquitetura consiste nos componentes a seguir.</span><span class="sxs-lookup"><span data-stu-id="538c4-113">The architecture consists of the following components.</span></span>

* <span data-ttu-id="538c4-114">**Rede local**.</span><span class="sxs-lookup"><span data-stu-id="538c4-114">**On-premises network**.</span></span> <span data-ttu-id="538c4-115">Uma rede de área local privada em execução dentro de uma organização.</span><span class="sxs-lookup"><span data-stu-id="538c4-115">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="538c4-116">**Dispositivo de VPN**.</span><span class="sxs-lookup"><span data-stu-id="538c4-116">**VPN appliance**.</span></span> <span data-ttu-id="538c4-117">Um dispositivo ou serviço que fornece conectividade externa com a rede local.</span><span class="sxs-lookup"><span data-stu-id="538c4-117">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="538c4-118">O dispositivo VPN pode ser um dispositivo de hardware ou pode ser uma solução de software, como RRAS (Serviço de Acesso Remoto e Roteamento) do Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="538c4-118">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="538c4-119">Para obter uma lista de dispositivos de VPN com suporte e informações sobre como configurar dispositivos de VPN selecionados para se conectar ao Azure, consulte [Sobre dispositivos VPN para conexões de Gateway de VPN Site a Site][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="538c4-119">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="538c4-120">**Circuito do ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="538c4-120">**ExpressRoute circuit**.</span></span> <span data-ttu-id="538c4-121">Um circuito de camada 2 ou de camada 3 fornecido pelo provedor de conectividade que une a rede local com o Azure por meio de roteadores de borda.</span><span class="sxs-lookup"><span data-stu-id="538c4-121">A layer 2 or layer 3 circuit supplied by the connectivity provider that joins the on-premises network with Azure through the edge routers.</span></span> <span data-ttu-id="538c4-122">O circuito usa a infraestrutura de hardware gerenciada pelo provedor de conectividade.</span><span class="sxs-lookup"><span data-stu-id="538c4-122">The circuit uses the hardware infrastructure managed by the connectivity provider.</span></span>

* <span data-ttu-id="538c4-123">**Gateway de rede virtual do ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="538c4-123">**ExpressRoute virtual network gateway**.</span></span> <span data-ttu-id="538c4-124">O gateway de rede virtual ExpressRoute permite que a VNet se conecte ao circuito ExpressRoute usado para conectividade com a rede local.</span><span class="sxs-lookup"><span data-stu-id="538c4-124">The ExpressRoute virtual network gateway enables the VNet to connect to the ExpressRoute circuit used for connectivity with your on-premises network.</span></span>

* <span data-ttu-id="538c4-125">**Gateway de rede virtual VPN**.</span><span class="sxs-lookup"><span data-stu-id="538c4-125">**VPN virtual network gateway**.</span></span> <span data-ttu-id="538c4-126">O gateway de rede virtual VPN permite que a VNet conecte-se ao dispositivo VPN na rede local.</span><span class="sxs-lookup"><span data-stu-id="538c4-126">The VPN virtual network gateway enables the VNet to connect to the VPN appliance in the on-premises network.</span></span> <span data-ttu-id="538c4-127">O gateway de rede virtual VPN está configurado para aceitar solicitações da rede local apenas por meio do dispositivo VPN.</span><span class="sxs-lookup"><span data-stu-id="538c4-127">The VPN virtual network gateway is configured to accept requests from the on-premises network only through the VPN appliance.</span></span> <span data-ttu-id="538c4-128">Para obter mais informações, consulte [Conectar uma rede local à rede virtual do Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="538c4-128">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

* <span data-ttu-id="538c4-129">**Conexão VPN**.</span><span class="sxs-lookup"><span data-stu-id="538c4-129">**VPN connection**.</span></span> <span data-ttu-id="538c4-130">A conexão tem propriedades que especificam o tipo de conexão (IPsec) e a chave compartilhada com o dispositivo VPN local para criptografar o tráfego.</span><span class="sxs-lookup"><span data-stu-id="538c4-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>

* <span data-ttu-id="538c4-131">**Rede virtual do Azure (VNet)**.</span><span class="sxs-lookup"><span data-stu-id="538c4-131">**Azure Virtual Network (VNet)**.</span></span> <span data-ttu-id="538c4-132">Cada VNet reside em uma única região do Azure e pode hospedar várias camadas de aplicativos.</span><span class="sxs-lookup"><span data-stu-id="538c4-132">Each VNet resides in a single Azure region, and can host multiple application tiers.</span></span> <span data-ttu-id="538c4-133">As camadas de aplicativo podem ser segmentadas usando sub-redes em cada VNet.</span><span class="sxs-lookup"><span data-stu-id="538c4-133">Application tiers can be segmented using subnets in each VNet.</span></span>

* <span data-ttu-id="538c4-134">**Gateway de sub-rede**.</span><span class="sxs-lookup"><span data-stu-id="538c4-134">**Gateway subnet**.</span></span> <span data-ttu-id="538c4-135">Os gateways de rede virtual são mantidos na mesma sub-rede.</span><span class="sxs-lookup"><span data-stu-id="538c4-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="538c4-136">**Aplicativo na nuvem**.</span><span class="sxs-lookup"><span data-stu-id="538c4-136">**Cloud application**.</span></span> <span data-ttu-id="538c4-137">O aplicativo hospedado no Azure.</span><span class="sxs-lookup"><span data-stu-id="538c4-137">The application hosted in Azure.</span></span> <span data-ttu-id="538c4-138">Ele pode incluir várias camadas, com várias sub-redes conectadas por meio de balanceadores de carga do Azure.</span><span class="sxs-lookup"><span data-stu-id="538c4-138">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="538c4-139">Para obter mais informações sobre a infraestrutura do aplicativo, consulte [Execução de cargas de trabalho de VM do Windows][windows-vm-ra] e [Execução de cargas de trabalho de VM do Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="538c4-139">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

## <a name="recommendations"></a><span data-ttu-id="538c4-140">Recomendações</span><span class="sxs-lookup"><span data-stu-id="538c4-140">Recommendations</span></span>

<span data-ttu-id="538c4-141">As seguintes recomendações aplicam-se à maioria dos cenários.</span><span class="sxs-lookup"><span data-stu-id="538c4-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="538c4-142">Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.</span><span class="sxs-lookup"><span data-stu-id="538c4-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="538c4-143">VNet e GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="538c4-143">VNet and GatewaySubnet</span></span>

<span data-ttu-id="538c4-144">Crie o gateway de rede virtual ExpressRoute e o gateway de rede virtual VPN na mesma VNet.</span><span class="sxs-lookup"><span data-stu-id="538c4-144">Create the ExpressRoute virtual network gateway and the VPN virtual network gateway in the same VNet.</span></span> <span data-ttu-id="538c4-145">Isso significa que eles devem compartilhar a mesma sub-rede chamada *GatewaySubnet*.</span><span class="sxs-lookup"><span data-stu-id="538c4-145">This means that they should share the same subnet named *GatewaySubnet*.</span></span>

<span data-ttu-id="538c4-146">Se a VNet já incluir uma sub-rede chamada *GatewaySubnet*, garanta que ela tenha um espaço de endereço /27 ou maior.</span><span class="sxs-lookup"><span data-stu-id="538c4-146">If the VNet already includes a subnet named *GatewaySubnet*, ensure that it has a /27 or larger address space.</span></span> <span data-ttu-id="538c4-147">Se a sub-rede existente for muito pequena, use o seguinte comando do PowerShell para removê-la:</span><span class="sxs-lookup"><span data-stu-id="538c4-147">If the existing subnet is too small, use the following PowerShell command to remove the subnet:</span></span> 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

<span data-ttu-id="538c4-148">Se a VNet não contiver uma sub-rede denominada **GatewaySubnet**, crie uma nova usando o seguinte comando do Powershell:</span><span class="sxs-lookup"><span data-stu-id="538c4-148">If the VNet does not contain a subnet named **GatewaySubnet**, create a new one using the following Powershell command:</span></span>

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a><span data-ttu-id="538c4-149">Gateways VPN e ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="538c4-149">VPN and ExpressRoute gateways</span></span>

<span data-ttu-id="538c4-150">Verifique se a sua organização cumpre os [requisitos de pré-requisitos do ExpressRoute][expressroute-prereq] para se conectar ao Azure.</span><span class="sxs-lookup"><span data-stu-id="538c4-150">Verify that your organization meets the [ExpressRoute prerequisite requirements][expressroute-prereq] for connecting to Azure.</span></span>

<span data-ttu-id="538c4-151">Se você já tiver um gateway de rede virtual VPN na sua VNet do Azure, use o seguinte comando do Powershell para removê-la:</span><span class="sxs-lookup"><span data-stu-id="538c4-151">If you already have a VPN virtual network gateway in your Azure VNet, use the following  Powershell command to remove it:</span></span>

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

<span data-ttu-id="538c4-152">Siga as instruções em [Implementar uma arquitetura de rede híbrida com o Azure ExpressRoute][implementing-expressroute] para estabelecer a conexão do ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="538c4-152">Follow the instructions in [Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute] to establish your ExpressRoute connection.</span></span>

<span data-ttu-id="538c4-153">Siga as instruções em [Implementar uma arquitetura de rede híbrida com o Azure e VPN local][implementing-vpn] para estabelecer sua conexão de gateway de rede virtual VPN.</span><span class="sxs-lookup"><span data-stu-id="538c4-153">Follow the instructions in [Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn] to establish your VPN virtual network gateway connection.</span></span>

<span data-ttu-id="538c4-154">Depois de estabelecer as conexões de gateway de rede virtual, teste o ambiente da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="538c4-154">After you have established the virtual network gateway connections, test the environment as follows:</span></span>

1. <span data-ttu-id="538c4-155">Verifique se que você pode conectar sua rede local à VNet do Azure.</span><span class="sxs-lookup"><span data-stu-id="538c4-155">Make sure you can connect from your on-premises network to your Azure VNet.</span></span>
2. <span data-ttu-id="538c4-156">Contate o provedor para interromper a conectividade do ExpressRoute para teste.</span><span class="sxs-lookup"><span data-stu-id="538c4-156">Contact your provider to stop ExpressRoute connectivity for testing.</span></span>
3. <span data-ttu-id="538c4-157">Verifique se que você ainda pode se conectar da sua rede local à VNet do Azure usando a conexão de gateway de rede virtual VPN.</span><span class="sxs-lookup"><span data-stu-id="538c4-157">Verify that you can still connect from your on-premises network to your Azure VNet using the VPN virtual network gateway connection.</span></span>
4. <span data-ttu-id="538c4-158">Contate o provedor para restabelecer a conectividade do ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="538c4-158">Contact your provider to reestablish ExpressRoute connectivity.</span></span>

## <a name="considerations"></a><span data-ttu-id="538c4-159">Considerações</span><span class="sxs-lookup"><span data-stu-id="538c4-159">Considerations</span></span>

<span data-ttu-id="538c4-160">Para considerações sobre o ExpressRoute, consulte a orientação [Implementar uma arquitetura de rede híbrida com o Azure ExpressRoute][guidance-expressroute].</span><span class="sxs-lookup"><span data-stu-id="538c4-160">For ExpressRoute considerations, see the [Implementing a Hybrid Network Architecture with Azure ExpressRoute][guidance-expressroute] guidance.</span></span>

<span data-ttu-id="538c4-161">Para considerações sobre VPN site a site, consulte a orientação [Implementar uma arquitetura de rede híbrida com a VPN do Azure e local][guidance-vpn].</span><span class="sxs-lookup"><span data-stu-id="538c4-161">For site-to-site VPN considerations, see the [Implementing a Hybrid Network Architecture with Azure and On-premises VPN][guidance-vpn] guidance.</span></span>

<span data-ttu-id="538c4-162">Para considerações gerais de segurança do Azure, consulte [Serviços em nuvem da Microsoft e segurança de rede][best-practices-security].</span><span class="sxs-lookup"><span data-stu-id="538c4-162">For general Azure security considerations, see [Microsoft cloud services and network security][best-practices-security].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="538c4-163">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="538c4-163">Deploy the solution</span></span>

<span data-ttu-id="538c4-164">**Pré-requisitos.**</span><span class="sxs-lookup"><span data-stu-id="538c4-164">**Prequisites.**</span></span> <span data-ttu-id="538c4-165">Você deve ter uma infraestrutura local existente já configurada com um dispositivo de rede adequado.</span><span class="sxs-lookup"><span data-stu-id="538c4-165">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="538c4-166">Para implantar a solução, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="538c4-166">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="538c4-167">Clique no botão abaixo:</span><span class="sxs-lookup"><span data-stu-id="538c4-167">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="538c4-168">Aguarde até o link abrir no Portal do Azure e siga estas etapas:</span><span class="sxs-lookup"><span data-stu-id="538c4-168">Wait for the link to open in the Azure portal, then follow these steps:</span></span>   
   * <span data-ttu-id="538c4-169">O nome do **Grupo de recursos** já está definido no arquivo de parâmetros, portanto, selecione **Criar novo** e digite `ra-hybrid-vpn-er-rg` na caixa de texto.</span><span class="sxs-lookup"><span data-stu-id="538c4-169">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="538c4-170">Selecione a região na caixa suspensa **Local**.</span><span class="sxs-lookup"><span data-stu-id="538c4-170">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="538c4-171">Não edite as caixas de texto **URI da raiz do modelo** ou **URI da raiz do parâmetro**.</span><span class="sxs-lookup"><span data-stu-id="538c4-171">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="538c4-172">Analise os termos e condições e clique na caixa de seleção **Concordo com os termos e condições declarados acima**.</span><span class="sxs-lookup"><span data-stu-id="538c4-172">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="538c4-173">Clique no botão **Comprar**.</span><span class="sxs-lookup"><span data-stu-id="538c4-173">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="538c4-174">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="538c4-174">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="538c4-175">Clique no botão abaixo:</span><span class="sxs-lookup"><span data-stu-id="538c4-175">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="538c4-176">Aguarde até o link abrir no portal do Azure e siga estas etapas:</span><span class="sxs-lookup"><span data-stu-id="538c4-176">Wait for the link to open in the Azure portal, then enter then follow these steps:</span></span>
   * <span data-ttu-id="538c4-177">Selecione **Usar existente** na seção **Grupo de recursos** e digite `ra-hybrid-vpn-er-rg` na caixa de texto.</span><span class="sxs-lookup"><span data-stu-id="538c4-177">Select **Use existing** in the **Resource group** section and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="538c4-178">Selecione a região na caixa suspensa **Local**.</span><span class="sxs-lookup"><span data-stu-id="538c4-178">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="538c4-179">Não edite as caixas de texto **URI da raiz do modelo** ou **URI da raiz do parâmetro**.</span><span class="sxs-lookup"><span data-stu-id="538c4-179">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="538c4-180">Analise os termos e condições e clique na caixa de seleção **Concordo com os termos e condições declarados acima**.</span><span class="sxs-lookup"><span data-stu-id="538c4-180">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="538c4-181">Clique no botão **Comprar**.</span><span class="sxs-lookup"><span data-stu-id="538c4-181">Click the **Purchase** button.</span></span>

<!-- links -->

[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md


[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[expressroute-prereq]: /azure/expressroute/expressroute-prerequisites
[implementing-expressroute]: ./expressroute.md
[implementing-vpn]: ./vpn.md
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[best-practices-security]: /azure/best-practices-network-security
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[0]: ./images/expressroute-vpn-failover.png "Arquitetura de rede híbrida altamente disponível usando o gateway VPN e ExpressRoute"
