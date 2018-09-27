---
title: Conectar uma rede local ao Azure usando VPN
description: Como implementar uma arquitetura de rede site a site segura que abranja uma rede virtual do Azure e uma rede local conectada usando uma VPN.
author: RohitSharma-pnp
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: d0966791642ee174d020c960c9fc6e72d8768539
ms.sourcegitcommit: b38ba378c9d6110da2dfd50b4233fadd94604bb0
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/25/2018
ms.locfileid: "47167431"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a><span data-ttu-id="cd434-103">Conectar uma rede local ao Azure usando um Gateway de VPN</span><span class="sxs-lookup"><span data-stu-id="cd434-103">Connect an on-premises network to Azure using a VPN gateway</span></span>

<span data-ttu-id="cd434-104">Essa arquitetura de referência mostra como estender uma rede local para o Azure, usando uma VPN (rede virtual privada) site a site.</span><span class="sxs-lookup"><span data-stu-id="cd434-104">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span> <span data-ttu-id="cd434-105">O tráfego flui entre a rede local e uma VNet (Rede Virtual) do Azure por meio de um túnel de VPN IPsec.</span><span class="sxs-lookup"><span data-stu-id="cd434-105">Traffic flows between the on-premises network and an Azure Virtual Network (VNet) through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="cd434-106">**Implante essa solução**.</span><span class="sxs-lookup"><span data-stu-id="cd434-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="cd434-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="cd434-107">![[0]][0]</span></span>

<span data-ttu-id="cd434-108">*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*</span><span class="sxs-lookup"><span data-stu-id="cd434-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="cd434-109">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="cd434-109">Architecture</span></span> 

<span data-ttu-id="cd434-110">A arquitetura consiste nos componentes a seguir.</span><span class="sxs-lookup"><span data-stu-id="cd434-110">The architecture consists of the following components.</span></span>

* <span data-ttu-id="cd434-111">**Rede local**.</span><span class="sxs-lookup"><span data-stu-id="cd434-111">**On-premises network**.</span></span> <span data-ttu-id="cd434-112">Uma rede de área local privada em execução dentro de uma organização.</span><span class="sxs-lookup"><span data-stu-id="cd434-112">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="cd434-113">**Dispositivo de VPN**.</span><span class="sxs-lookup"><span data-stu-id="cd434-113">**VPN appliance**.</span></span> <span data-ttu-id="cd434-114">Um dispositivo ou serviço que fornece conectividade externa com a rede local.</span><span class="sxs-lookup"><span data-stu-id="cd434-114">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="cd434-115">O dispositivo de VPN pode ser um dispositivo de hardware ou uma solução de software, como o RRAS (Serviço de Roteamento e Acesso Remoto) do Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="cd434-115">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="cd434-116">Para obter uma lista de dispositivos de VPN com suporte e informações de como configurá-los para se conectar a um Gateway de VPN do Azure, consulte as instruções para o dispositivo selecionado no artigo [Sobre dispositivos VPN para conexões do Gateway de VPN site a site][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="cd434-116">For a list of supported VPN appliances and information on configuring them to connect to an Azure VPN gateway, see the instructions for the selected device in the article [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="cd434-117">**Rede virtual (VNet)**.</span><span class="sxs-lookup"><span data-stu-id="cd434-117">**Virtual network (VNet)**.</span></span> <span data-ttu-id="cd434-118">O aplicativo de nuvem e os componentes do Gateway de VPN do Azure residem na mesma [VNet][azure-virtual-network].</span><span class="sxs-lookup"><span data-stu-id="cd434-118">The cloud application and the components for the Azure VPN gateway reside in the same [VNet][azure-virtual-network].</span></span>

* <span data-ttu-id="cd434-119">**Gateway de VPN do Azure**.</span><span class="sxs-lookup"><span data-stu-id="cd434-119">**Azure VPN gateway**.</span></span> <span data-ttu-id="cd434-120">O serviço do [Gateway de VPN][azure-vpn-gateway] permite conectar a VNet à rede local por meio de um dispositivo de VPN.</span><span class="sxs-lookup"><span data-stu-id="cd434-120">The [VPN gateway][azure-vpn-gateway] service enables you to connect the VNet to the on-premises network through a VPN appliance.</span></span> <span data-ttu-id="cd434-121">Para obter mais informações, consulte [Conectar uma rede local a uma rede virtual do Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="cd434-121">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span> <span data-ttu-id="cd434-122">O Gateway de VPN inclui os seguintes elementos:</span><span class="sxs-lookup"><span data-stu-id="cd434-122">The VPN gateway includes the following elements:</span></span>
  
  * <span data-ttu-id="cd434-123">**Gateway de rede virtual**.</span><span class="sxs-lookup"><span data-stu-id="cd434-123">**Virtual network gateway**.</span></span> <span data-ttu-id="cd434-124">Um recurso que fornece um dispositivo de VPN virtual para a VNet.</span><span class="sxs-lookup"><span data-stu-id="cd434-124">A resource that provides a virtual VPN appliance for the VNet.</span></span> <span data-ttu-id="cd434-125">Ele é responsável por rotear o tráfego da rede local para a VNet.</span><span class="sxs-lookup"><span data-stu-id="cd434-125">It is responsible for routing traffic from the on-premises network to the VNet.</span></span>
  * <span data-ttu-id="cd434-126">**Gateway de rede local**.</span><span class="sxs-lookup"><span data-stu-id="cd434-126">**Local network gateway**.</span></span> <span data-ttu-id="cd434-127">Uma abstração do dispositivo de VPN local.</span><span class="sxs-lookup"><span data-stu-id="cd434-127">An abstraction of the on-premises VPN appliance.</span></span> <span data-ttu-id="cd434-128">O tráfego de rede do aplicativo de nuvem para a rede local é roteado por esse gateway.</span><span class="sxs-lookup"><span data-stu-id="cd434-128">Network traffic from the cloud application to the on-premises network is routed through this gateway.</span></span>
  * <span data-ttu-id="cd434-129">**Conexão**.</span><span class="sxs-lookup"><span data-stu-id="cd434-129">**Connection**.</span></span> <span data-ttu-id="cd434-130">A conexão tem propriedades que especificam o tipo de conexão (IPsec) e a chave compartilhada com o dispositivo de VPN local para criptografar o tráfego.</span><span class="sxs-lookup"><span data-stu-id="cd434-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>
  * <span data-ttu-id="cd434-131">**Gateway de sub-rede**.</span><span class="sxs-lookup"><span data-stu-id="cd434-131">**Gateway subnet**.</span></span> <span data-ttu-id="cd434-132">O gateway de rede virtual é mantido em sua própria sub-rede, que está sujeita a vários requisitos, descritos na seção Recomendações abaixo.</span><span class="sxs-lookup"><span data-stu-id="cd434-132">The virtual network gateway is held in its own subnet, which is subject to various requirements, described in the Recommendations section below.</span></span>

* <span data-ttu-id="cd434-133">**Aplicativo de nuvem**.</span><span class="sxs-lookup"><span data-stu-id="cd434-133">**Cloud application**.</span></span> <span data-ttu-id="cd434-134">O aplicativo hospedado no Azure.</span><span class="sxs-lookup"><span data-stu-id="cd434-134">The application hosted in Azure.</span></span> <span data-ttu-id="cd434-135">Ele pode incluir várias camadas, com várias sub-redes conectadas por meio de balanceadores de carga do Azure.</span><span class="sxs-lookup"><span data-stu-id="cd434-135">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="cd434-136">Para obter mais informações sobre a infraestrutura do aplicativo, consulte [Execução de cargas de trabalho de VM do Windows][windows-vm-ra] e [Execução de cargas de trabalho de VM do Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="cd434-136">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="cd434-137">**Balanceador Interno de carga**.</span><span class="sxs-lookup"><span data-stu-id="cd434-137">**Internal load balancer**.</span></span> <span data-ttu-id="cd434-138">O tráfego de rede do Gateway de VPN é roteado para o aplicativo de nuvem por meio de um balanceador de carga interno.</span><span class="sxs-lookup"><span data-stu-id="cd434-138">Network traffic from the VPN gateway is routed to the cloud application through an internal load balancer.</span></span> <span data-ttu-id="cd434-139">O balanceador de carga está localizado na sub-rede de front-end do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="cd434-139">The load balancer is located in the front-end subnet of the application.</span></span>

## <a name="recommendations"></a><span data-ttu-id="cd434-140">Recomendações</span><span class="sxs-lookup"><span data-stu-id="cd434-140">Recommendations</span></span>

<span data-ttu-id="cd434-141">As seguintes recomendações aplicam-se à maioria dos cenários.</span><span class="sxs-lookup"><span data-stu-id="cd434-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="cd434-142">Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.</span><span class="sxs-lookup"><span data-stu-id="cd434-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gateway-subnet"></a><span data-ttu-id="cd434-143">VNet e sub-rede do gateway</span><span class="sxs-lookup"><span data-stu-id="cd434-143">VNet and gateway subnet</span></span>

<span data-ttu-id="cd434-144">Crie uma VNet do Azure com um espaço de endereço grande o suficiente para todos os recursos necessários.</span><span class="sxs-lookup"><span data-stu-id="cd434-144">Create an Azure VNet with an address space large enough for all of your required resources.</span></span> <span data-ttu-id="cd434-145">Verifique se o espaço de endereço da VNet tem espaço suficiente para aumentar caso haja necessidade de adicionar mais VMs no futuro.</span><span class="sxs-lookup"><span data-stu-id="cd434-145">Ensure that the VNet address space has sufficient room for growth if additional VMs are likely to be needed in the future.</span></span> <span data-ttu-id="cd434-146">O espaço de endereço da VNet não deve se sobrepor à rede local.</span><span class="sxs-lookup"><span data-stu-id="cd434-146">The address space of the VNet must not overlap with the on-premises network.</span></span> <span data-ttu-id="cd434-147">Por exemplo, o diagrama acima usa o espaço de endereço 10.20.0.0/16 para a VNet.</span><span class="sxs-lookup"><span data-stu-id="cd434-147">For example, the diagram above uses the address space 10.20.0.0/16 for the VNet.</span></span>

<span data-ttu-id="cd434-148">Crie uma sub-rede denominada *GatewaySubnet*, com um intervalo de endereços de /27.</span><span class="sxs-lookup"><span data-stu-id="cd434-148">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="cd434-149">Essa sub-rede é necessária para o gateway de rede virtual.</span><span class="sxs-lookup"><span data-stu-id="cd434-149">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="cd434-150">Alocar 32 endereços para essa sub-rede ajudará a evitar que as limitações de tamanho do gateway sejam atingidas no futuro.</span><span class="sxs-lookup"><span data-stu-id="cd434-150">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span> <span data-ttu-id="cd434-151">Além disso, evite colocar essa sub-rede no meio do espaço de endereço.</span><span class="sxs-lookup"><span data-stu-id="cd434-151">Also, avoid placing this subnet in the middle of the address space.</span></span> <span data-ttu-id="cd434-152">Uma prática recomendada é definir o espaço de endereço para a sub-rede de gateway na extremidade superior do espaço de endereço da VNet.</span><span class="sxs-lookup"><span data-stu-id="cd434-152">A good practice is to set the address space for the gateway subnet at the upper end of the VNet address space.</span></span> <span data-ttu-id="cd434-153">O exemplo mostrado no diagrama usa 10.20.255.224/27.</span><span class="sxs-lookup"><span data-stu-id="cd434-153">The example shown in the diagram uses 10.20.255.224/27.</span></span>  <span data-ttu-id="cd434-154">Aqui está um procedimento rápido para calcular o [CIDR]:</span><span class="sxs-lookup"><span data-stu-id="cd434-154">Here is a quick procedure to calculate the [CIDR]:</span></span>

1. <span data-ttu-id="cd434-155">Defina os bits variáveis no espaço de endereço da VNet como 1, até os bits que estão sendo usados pela sub-rede do gateway e, em seguida, configure os bits restantes como 0.</span><span class="sxs-lookup"><span data-stu-id="cd434-155">Set the variable bits in the address space of the VNet to 1, up to the bits being used by the gateway subnet, then set the remaining bits to 0.</span></span>
2. <span data-ttu-id="cd434-156">Converta os bits resultantes em decimais e expresse-os como um espaço de endereço com o comprimento do prefixo definido para o tamanho da sub-rede do gateway.</span><span class="sxs-lookup"><span data-stu-id="cd434-156">Convert the resulting bits to decimal and express it as an address space with the prefix length set to the size of the gateway subnet.</span></span>

<span data-ttu-id="cd434-157">Por exemplo, para uma VNet com um intervalo de endereços IP igual a 10.20.0.0/16, a aplicação da etapa nº 1 acima transforma-o em 10.20.0b11111111.0b11100000.</span><span class="sxs-lookup"><span data-stu-id="cd434-157">For example, for a VNet with an IP address range of 10.20.0.0/16, applying step #1 above becomes 10.20.0b11111111.0b11100000.</span></span>  <span data-ttu-id="cd434-158">Convertê-lo em decimal e expressá-lo como um espaço de endereço produz 10.20.255.224/27.</span><span class="sxs-lookup"><span data-stu-id="cd434-158">Converting that to decimal and expressing it as an address space yields 10.20.255.224/27.</span></span> 

> [!WARNING]
> <span data-ttu-id="cd434-159">Não implante nenhuma VM na sub-rede do gateway.</span><span class="sxs-lookup"><span data-stu-id="cd434-159">Do not deploy any VMs to the gateway subnet.</span></span> <span data-ttu-id="cd434-160">Além disso, não atribua um NSG a essa sub-rede, pois faria com que o gateway parasse de funcionar.</span><span class="sxs-lookup"><span data-stu-id="cd434-160">Also, do not assign an NSG to this subnet, as it will cause the gateway to stop functioning.</span></span>
> 
> 

### <a name="virtual-network-gateway"></a><span data-ttu-id="cd434-161">Gateway de rede virtual</span><span class="sxs-lookup"><span data-stu-id="cd434-161">Virtual network gateway</span></span>

<span data-ttu-id="cd434-162">Aloque um endereço IP público para o gateway de rede virtual.</span><span class="sxs-lookup"><span data-stu-id="cd434-162">Allocate a public IP address for the virtual network gateway.</span></span>

<span data-ttu-id="cd434-163">Crie o gateway de rede virtual na sub-rede do gateway e atribua-o ao endereço IP público recém-alocado.</span><span class="sxs-lookup"><span data-stu-id="cd434-163">Create the virtual network gateway in the gateway subnet and assign it the newly allocated public IP address.</span></span> <span data-ttu-id="cd434-164">Use o tipo de gateway que melhor corresponde aos seus requisitos e que está habilitado pelo seu dispositivo de VPN:</span><span class="sxs-lookup"><span data-stu-id="cd434-164">Use the gateway type that most closely matches your requirements and that is enabled by your VPN appliance:</span></span>

- <span data-ttu-id="cd434-165">Crie um [gateway baseado em políticas][policy-based-routing] se você precisar controlar atentamente como as solicitações são roteadas com base em critérios de política, como prefixos de endereço.</span><span class="sxs-lookup"><span data-stu-id="cd434-165">Create a [policy-based gateway][policy-based-routing] if you need to closely control how requests are routed based on policy criteria such as address prefixes.</span></span> <span data-ttu-id="cd434-166">Os gateways baseados em políticas usam o roteamento estático e funcionam apenas com conexões site a site.</span><span class="sxs-lookup"><span data-stu-id="cd434-166">Policy-based gateways use static routing, and only work with site-to-site connections.</span></span>

- <span data-ttu-id="cd434-167">Crie um [gateway baseado em rota][route-based-routing] se você se conecta à rede local usando o RRAS, dá suporte a conexões multisites ou entre regiões ou implementa conexões de VNet para VNet (incluindo rotas que atravessam várias VNets).</span><span class="sxs-lookup"><span data-stu-id="cd434-167">Create a [route-based gateway][route-based-routing] if you connect to the on-premises network using RRAS, support multi-site or cross-region connections, or implement VNet-to-VNet connections (including routes that traverse multiple VNets).</span></span> <span data-ttu-id="cd434-168">Os gateways baseados em rota usam o roteamento dinâmico para o tráfego direto entre redes.</span><span class="sxs-lookup"><span data-stu-id="cd434-168">Route-based gateways use dynamic routing to direct traffic between networks.</span></span> <span data-ttu-id="cd434-169">Eles conseguem tolerar falhas no caminho da rede melhor do que as rotas estáticas, porque eles podem tentar rotas alternativas.</span><span class="sxs-lookup"><span data-stu-id="cd434-169">They can tolerate failures in the network path better than static routes because they can try alternative routes.</span></span> <span data-ttu-id="cd434-170">Os gateways baseados em rota também conseguem reduzir a sobrecarga de gerenciamento porque é possível que as rotas não precisem ser atualizadas manualmente quando os endereços de rede são alterados.</span><span class="sxs-lookup"><span data-stu-id="cd434-170">Route-based gateways can also reduce the management overhead because routes might not need to be updated manually when network addresses change.</span></span>

<span data-ttu-id="cd434-171">Para obter uma lista de dispositivos de VPN com suporte, consulte [Sobre dispositivos VPN para conexões do Gateway de VPN site a site][vpn-appliances].</span><span class="sxs-lookup"><span data-stu-id="cd434-171">For a list of supported VPN appliances, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliances].</span></span>

> [!NOTE]
> <span data-ttu-id="cd434-172">Depois que o gateway for criado, você não poderá mais alterar o tipo de gateway sem excluir e recriar o gateway.</span><span class="sxs-lookup"><span data-stu-id="cd434-172">After the gateway has been created, you cannot change between gateway types without deleting and re-creating the gateway.</span></span>
> 
> 

<span data-ttu-id="cd434-173">Selecione a SKU do Gateway de VPN do Azure que melhor corresponde aos seus requisitos de produtividade.</span><span class="sxs-lookup"><span data-stu-id="cd434-173">Select the Azure VPN gateway SKU that most closely matches your throughput requirements.</span></span> <span data-ttu-id="cd434-174">Para obter mais informações, confira [SKUs do gateway][azure-gateway-skus]</span><span class="sxs-lookup"><span data-stu-id="cd434-174">For more informayion, see [Gateway SKUs][azure-gateway-skus]</span></span>

> [!NOTE]
> <span data-ttu-id="cd434-175">A SKU básica não é compatível com o Azure ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="cd434-175">The Basic SKU is not compatible with Azure ExpressRoute.</span></span> <span data-ttu-id="cd434-176">Você poderá [alterar a SKU][changing-SKUs] depois que o gateway for criado.</span><span class="sxs-lookup"><span data-stu-id="cd434-176">You can [change the SKU][changing-SKUs] after the gateway has been created.</span></span>
> 
> 

<span data-ttu-id="cd434-177">Você é cobrado com base no período de tempo em que o gateway é provisionado e fica disponível.</span><span class="sxs-lookup"><span data-stu-id="cd434-177">You are charged based on the amount of time that the gateway is provisioned and available.</span></span> <span data-ttu-id="cd434-178">Consulte [Preços do Gateway de VPN][azure-gateway-charges].</span><span class="sxs-lookup"><span data-stu-id="cd434-178">See [VPN Gateway Pricing][azure-gateway-charges].</span></span>

<span data-ttu-id="cd434-179">Crie regras de roteamento para a sub-rede de gateway que direcionem o tráfego do aplicativo de entrada do gateway para o balanceador de carga interno, em vez de permitirem que as solicitações passem diretamente para as VMs de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="cd434-179">Create routing rules for the gateway subnet that direct incoming application traffic from the gateway to the internal load balancer, rather than allowing requests to pass directly to the application VMs.</span></span>

### <a name="on-premises-network-connection"></a><span data-ttu-id="cd434-180">Conexão de rede local</span><span class="sxs-lookup"><span data-stu-id="cd434-180">On-premises network connection</span></span>

<span data-ttu-id="cd434-181">Criar um gateway de rede local.</span><span class="sxs-lookup"><span data-stu-id="cd434-181">Create a local network gateway.</span></span> <span data-ttu-id="cd434-182">Especifique o endereço IP público do dispositivo de VPN local e o espaço de endereço da rede local.</span><span class="sxs-lookup"><span data-stu-id="cd434-182">Specify the public IP address of the on-premises VPN appliance, and the address space of the on-premises network.</span></span> <span data-ttu-id="cd434-183">Observe que o dispositivo de VPN local deve ter um endereço IP público que possa ser acessado pelo gateway de rede local no Gateway de VPN do Azure.</span><span class="sxs-lookup"><span data-stu-id="cd434-183">Note that the on-premises VPN appliance must have a public IP address that can be accessed by the local network gateway in Azure VPN Gateway.</span></span> <span data-ttu-id="cd434-184">O dispositivo VPN não pode estar localizado atrás de um dispositivo de NAT (conversão de endereços de rede).</span><span class="sxs-lookup"><span data-stu-id="cd434-184">The VPN device cannot be located behind a network address translation (NAT) device.</span></span>

<span data-ttu-id="cd434-185">Crie uma conexão site a site para o gateway de rede virtual e o gateway de rede local.</span><span class="sxs-lookup"><span data-stu-id="cd434-185">Create a site-to-site connection for the virtual network gateway and the local network gateway.</span></span> <span data-ttu-id="cd434-186">Selecione o tipo de conexão site a site (IPsec) e especifique a chave compartilhada.</span><span class="sxs-lookup"><span data-stu-id="cd434-186">Select the site-to-site (IPSec) connection type, and specify the shared key.</span></span> <span data-ttu-id="cd434-187">A criptografia site a site com o Gateway de VPN do Azure é baseada no protocolo IPsec, usando chaves pré-compartilhadas para autenticação.</span><span class="sxs-lookup"><span data-stu-id="cd434-187">Site-to-site encryption with the Azure VPN gateway is based on the IPSec protocol, using preshared keys for authentication.</span></span> <span data-ttu-id="cd434-188">Especifique a chave ao criar o Gateway de VPN do Azure.</span><span class="sxs-lookup"><span data-stu-id="cd434-188">You specify the key when you create the Azure VPN gateway.</span></span> <span data-ttu-id="cd434-189">Você deve configurar o dispositivo de VPN em execução local com a mesma chave.</span><span class="sxs-lookup"><span data-stu-id="cd434-189">You must configure the VPN appliance running on-premises with the same key.</span></span> <span data-ttu-id="cd434-190">Atualmente, não há suporte para outros mecanismos de autenticação.</span><span class="sxs-lookup"><span data-stu-id="cd434-190">Other authentication mechanisms are not currently supported.</span></span>

<span data-ttu-id="cd434-191">Verifique se a infraestrutura de roteamento local está configurada para encaminhar as solicitações destinadas a endereços na VNet do Azure para o dispositivo VPN.</span><span class="sxs-lookup"><span data-stu-id="cd434-191">Ensure that the on-premises routing infrastructure is configured to forward requests intended for addresses in the Azure VNet to the VPN device.</span></span>

<span data-ttu-id="cd434-192">Abra todas as portas exigidas pelo aplicativo de nuvem na rede local.</span><span class="sxs-lookup"><span data-stu-id="cd434-192">Open any ports required by the cloud application in the on-premises network.</span></span>

<span data-ttu-id="cd434-193">Teste a conexão para verificar se:</span><span class="sxs-lookup"><span data-stu-id="cd434-193">Test the connection to verify that:</span></span>

* <span data-ttu-id="cd434-194">O dispositivo de VPN local roteia corretamente o tráfego para o aplicativo de nuvem por meio do Gateway de VPN do Azure.</span><span class="sxs-lookup"><span data-stu-id="cd434-194">The on-premises VPN appliance correctly routes traffic to the cloud application through the Azure VPN gateway.</span></span>
* <span data-ttu-id="cd434-195">A VNet roteia corretamente o tráfego de volta para a rede local.</span><span class="sxs-lookup"><span data-stu-id="cd434-195">The VNet correctly routes traffic back to the on-premises network.</span></span>
* <span data-ttu-id="cd434-196">O tráfego proibido em ambas as direções é bloqueado corretamente.</span><span class="sxs-lookup"><span data-stu-id="cd434-196">Prohibited traffic in both directions is blocked correctly.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="cd434-197">Considerações sobre escalabilidade</span><span class="sxs-lookup"><span data-stu-id="cd434-197">Scalability considerations</span></span>

<span data-ttu-id="cd434-198">Você pode obter uma escalabilidade vertical limitada passando as SKUs do Gateway de VPN do plano Básico ou Standard para a SKU de VPN de alto desempenho.</span><span class="sxs-lookup"><span data-stu-id="cd434-198">You can achieve limited vertical scalability by moving from the Basic or Standard VPN Gateway SKUs to the High Performance VPN SKU.</span></span>

<span data-ttu-id="cd434-199">Para VNets que esperam um grande volume de tráfego de VPN, considere distribuir as diferentes cargas de trabalho em VNets menores separadas e configurar um Gateway de VPN para cada uma delas.</span><span class="sxs-lookup"><span data-stu-id="cd434-199">For VNets that expect a large volume of VPN traffic, consider distributing the different workloads into separate smaller VNets and configuring a VPN gateway for each of them.</span></span>

<span data-ttu-id="cd434-200">Você pode particionar a VNet horizontal ou verticalmente.</span><span class="sxs-lookup"><span data-stu-id="cd434-200">You can partition the VNet either horizontally or vertically.</span></span> <span data-ttu-id="cd434-201">Para particionar horizontalmente, mova algumas instâncias de VM de cada camada para as sub-redes da nova VNet.</span><span class="sxs-lookup"><span data-stu-id="cd434-201">To partition horizontally, move some VM instances from each tier into subnets of the new VNet.</span></span> <span data-ttu-id="cd434-202">O resultado é que cada VNet terá as mesmas estrutura e funcionalidade.</span><span class="sxs-lookup"><span data-stu-id="cd434-202">The result is that each VNet has the same structure and functionality.</span></span> <span data-ttu-id="cd434-203">Para particionar verticalmente, recrie cada camada para dividir a funcionalidade em áreas lógicas diferentes (como para tratamento de pedidos, faturamento, gerenciamento de contas de cliente e assim por diante).</span><span class="sxs-lookup"><span data-stu-id="cd434-203">To partition vertically, redesign each tier to divide the functionality into different logical areas (such as handling orders, invoicing, customer account management, and so on).</span></span> <span data-ttu-id="cd434-204">Assim, cada área funcional poderá ser colocada em sua própria VNet.</span><span class="sxs-lookup"><span data-stu-id="cd434-204">Each functional area can then be placed in its own VNet.</span></span>

<span data-ttu-id="cd434-205">A replicação de um controlador de domínio do Active Directory local na VNet e a implementação de DNS na VNet podem ajudar a reduzir parte do tráfego relacionado à segurança e administrativo que flui do local para a nuvem.</span><span class="sxs-lookup"><span data-stu-id="cd434-205">Replicating an on-premises Active Directory domain controller in the VNet, and implementing DNS in the VNet, can help to reduce some of the security-related and administrative traffic flowing from on-premises to the cloud.</span></span> <span data-ttu-id="cd434-206">Para obter mais informações, consulte [Estendendo o AD DS (Active Directory Domain Services) para o Azure][adds-extend-domain].</span><span class="sxs-lookup"><span data-stu-id="cd434-206">For more information, see [Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="cd434-207">Considerações sobre disponibilidade</span><span class="sxs-lookup"><span data-stu-id="cd434-207">Availability considerations</span></span>

<span data-ttu-id="cd434-208">Se você precisa garantir que a rede local permaneça disponível para o Gateway de VPN do Azure, implemente um cluster de failover para o Gateway de VPN local.</span><span class="sxs-lookup"><span data-stu-id="cd434-208">If you need to ensure that the on-premises network remains available to the Azure VPN gateway, implement a failover cluster for the on-premises VPN gateway.</span></span>

<span data-ttu-id="cd434-209">Se sua organização tiver vários sites locais, crie [conexões multisites][vpn-gateway-multi-site] para uma ou mais VNets do Azure.</span><span class="sxs-lookup"><span data-stu-id="cd434-209">If your organization has multiple on-premises sites, create [multi-site connections][vpn-gateway-multi-site] to one or more Azure VNets.</span></span> <span data-ttu-id="cd434-210">Essa abordagem requer o roteamento dinâmico (baseado em rota), portanto, verifique se o Gateway de VPN local dá suporte a esse recurso.</span><span class="sxs-lookup"><span data-stu-id="cd434-210">This approach requires dynamic (route-based) routing, so make sure that the on-premises VPN gateway supports this feature.</span></span>

<span data-ttu-id="cd434-211">Para obter detalhes sobre os contratos de nível de serviço, consulte [SLA para Gateway de VPN][sla-for-vpn-gateway].</span><span class="sxs-lookup"><span data-stu-id="cd434-211">For details about service level agreements, see [SLA for VPN Gateway][sla-for-vpn-gateway].</span></span> 

## <a name="manageability-considerations"></a><span data-ttu-id="cd434-212">Considerações sobre capacidade de gerenciamento</span><span class="sxs-lookup"><span data-stu-id="cd434-212">Manageability considerations</span></span>

<span data-ttu-id="cd434-213">Monitore as informações de diagnóstico usando dispositivos de VPN locais.</span><span class="sxs-lookup"><span data-stu-id="cd434-213">Monitor diagnostic information from on-premises VPN appliances.</span></span> <span data-ttu-id="cd434-214">Esse processo depende dos recursos fornecidos pelo dispositivo de VPN.</span><span class="sxs-lookup"><span data-stu-id="cd434-214">This process depends on the features provided by the VPN appliance.</span></span> <span data-ttu-id="cd434-215">Por exemplo, se você estiver usando o roteamento e o serviço de acesso remoto no Windows Server 2012, o [log de RRAS][rras-logging].</span><span class="sxs-lookup"><span data-stu-id="cd434-215">For example, if you are using the Routing and Remote Access Service on Windows Server 2012, [RRAS logging][rras-logging].</span></span>

<span data-ttu-id="cd434-216">Use [diagnóstico do Gateway de VPN do Azure][gateway-diagnostic-logs] para capturar informações sobre problemas de conectividade.</span><span class="sxs-lookup"><span data-stu-id="cd434-216">Use [Azure VPN gateway diagnostics][gateway-diagnostic-logs] to capture information about connectivity issues.</span></span> <span data-ttu-id="cd434-217">Esses logs podem ser usados para rastrear informações como a origem e os destinos das solicitações de conexão, o protocolo usado e como a conexão foi estabelecida (ou por que a tentativa falhou).</span><span class="sxs-lookup"><span data-stu-id="cd434-217">These logs can be used to track information such as the source and destinations of connection requests, which protocol was used, and how the connection was established (or why the attempt failed).</span></span>

<span data-ttu-id="cd434-218">Monitore os logs operacionais do Gateway de VPN do Azure usando os logs de auditoria disponíveis no portal do Azure.</span><span class="sxs-lookup"><span data-stu-id="cd434-218">Monitor the operational logs of the Azure VPN gateway using the audit logs available in the Azure portal.</span></span> <span data-ttu-id="cd434-219">Logs separados estão disponíveis para o gateway de rede local, o gateway de rede do Azure e a conexão.</span><span class="sxs-lookup"><span data-stu-id="cd434-219">Separate logs are available for the local network gateway, the Azure network gateway, and the connection.</span></span> <span data-ttu-id="cd434-220">Essas informações podem ser usadas para acompanhar as alterações feitas no gateway e poderão ser úteis se um gateway que estava em funcionamento parar de funcionar por alguma razão.</span><span class="sxs-lookup"><span data-stu-id="cd434-220">This information can be used to track any changes made to the gateway, and can be useful if a previously functioning gateway stops working for some reason.</span></span>

<span data-ttu-id="cd434-221">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="cd434-221">![[2]][2]</span></span>

<span data-ttu-id="cd434-222">Monitore a conectividade e acompanhe os eventos de falha de conectividade.</span><span class="sxs-lookup"><span data-stu-id="cd434-222">Monitor connectivity, and track connectivity failure events.</span></span> <span data-ttu-id="cd434-223">Você pode usar um pacote de monitoramento, como o [Nagios][nagios] para capturar e relatar essas informações.</span><span class="sxs-lookup"><span data-stu-id="cd434-223">You can use a monitoring package such as [Nagios][nagios] to capture and report this information.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="cd434-224">Considerações de segurança</span><span class="sxs-lookup"><span data-stu-id="cd434-224">Security considerations</span></span>

<span data-ttu-id="cd434-225">Gere uma chave compartilhada diferente para cada Gateway de VPN.</span><span class="sxs-lookup"><span data-stu-id="cd434-225">Generate a different shared key for each VPN gateway.</span></span> <span data-ttu-id="cd434-226">Use uma chave compartilhada forte para ajudar na resistência contra ataques de força bruta.</span><span class="sxs-lookup"><span data-stu-id="cd434-226">Use a strong shared key to help resist brute-force attacks.</span></span>

> [!NOTE]
> <span data-ttu-id="cd434-227">No momento, não é possível usar o Azure Key Vault para pré-compartilhar chaves do Gateway de VPN do Azure.</span><span class="sxs-lookup"><span data-stu-id="cd434-227">Currently, you cannot use Azure Key Vault to preshare keys for the Azure VPN gateway.</span></span>
> 
> 

<span data-ttu-id="cd434-228">Verifique se o dispositivo de VPN local usa um método de criptografia [compatível com o Gateway de VPN do Azure][vpn-appliance-ipsec].</span><span class="sxs-lookup"><span data-stu-id="cd434-228">Ensure that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance-ipsec].</span></span> <span data-ttu-id="cd434-229">Para roteamento baseado em políticas, o Gateway de VPN do Azure dá suporte aos algoritmos de criptografia 3DES, AES256 e AES128.</span><span class="sxs-lookup"><span data-stu-id="cd434-229">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="cd434-230">Os gateways baseados em rota dão suporte para AES256 e 3DES.</span><span class="sxs-lookup"><span data-stu-id="cd434-230">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="cd434-231">Se seu dispositivo de VPN local estiver em uma DMZ (rede de perímetro) com um firewall entre a rede de perímetro e a Internet, poderá ser necessário configurar [regras de firewall adicionais][additional-firewall-rules] para permitir a conexão de VPN site a site.</span><span class="sxs-lookup"><span data-stu-id="cd434-231">If your on-premises VPN appliance is on a perimeter network (DMZ) that has a firewall between the perimeter network and the Internet, you might have to configure [additional firewall rules][additional-firewall-rules] to allow the site-to-site VPN connection.</span></span>

<span data-ttu-id="cd434-232">Se o aplicativo na VNet envia dados para a Internet, considere [implementar um túnel forçado][forced-tunneling] para rotear todo o tráfego associado à Internet por meio da rede local.</span><span class="sxs-lookup"><span data-stu-id="cd434-232">If the application in the VNet sends data to the Internet, consider [implementing forced tunneling][forced-tunneling] to route all Internet-bound traffic through the on-premises network.</span></span> <span data-ttu-id="cd434-233">Essa abordagem permite auditar as solicitações de saída feitas pelo aplicativo da infraestrutura local.</span><span class="sxs-lookup"><span data-stu-id="cd434-233">This approach enables you to audit outgoing requests made by the application from the on-premises infrastructure.</span></span>

> [!NOTE]
> <span data-ttu-id="cd434-234">O túnel forçado pode afetar a conectividade para serviços do Azure (o serviço de armazenamento, por exemplo) e o Gerenciador de licenças do Windows.</span><span class="sxs-lookup"><span data-stu-id="cd434-234">Forced tunneling can impact connectivity to Azure services (the Storage Service, for example) and the Windows license manager.</span></span>
> 
> 


## <a name="troubleshooting"></a><span data-ttu-id="cd434-235">solução de problemas</span><span class="sxs-lookup"><span data-stu-id="cd434-235">Troubleshooting</span></span> 

<span data-ttu-id="cd434-236">Para obter informações gerais de como solucionar erros comuns relacionados à VPN, consulte [Troubleshooting common VPN related error][troubleshooting-vpn-errors] (Solucionando erros comuns relacionados à VPN).</span><span class="sxs-lookup"><span data-stu-id="cd434-236">For general information on troubleshooting common VPN-related errors, see [Troubleshooting common VPN related errors][troubleshooting-vpn-errors].</span></span>

<span data-ttu-id="cd434-237">As recomendações a seguir são úteis para determinar se o dispositivo de VPN local está funcionando corretamente.</span><span class="sxs-lookup"><span data-stu-id="cd434-237">The following recommendations are useful for determining if your on-premises VPN appliance is functioning correctly.</span></span>

- <span data-ttu-id="cd434-238">**Verifique se há erros ou falhas em todos os arquivos de log gerados pelo dispositivo de VPN.**</span><span class="sxs-lookup"><span data-stu-id="cd434-238">**Check any log files generated by the VPN appliance for errors or failures.**</span></span>

    <span data-ttu-id="cd434-239">Isso ajudará a determinar se o dispositivo de VPN está funcionando corretamente.</span><span class="sxs-lookup"><span data-stu-id="cd434-239">This will help you determine if the VPN appliance is functioning correctly.</span></span> <span data-ttu-id="cd434-240">O local dessas informações variam de acordo com o dispositivo.</span><span class="sxs-lookup"><span data-stu-id="cd434-240">The location of this information will vary according to your appliance.</span></span> <span data-ttu-id="cd434-241">Por exemplo, se você estiver usando o RRAS no Windows Server 2012, será possível usar o seguinte comando do PowerShell para exibir informações de evento de erro para o serviço RRAS:</span><span class="sxs-lookup"><span data-stu-id="cd434-241">For example, if you are using RRAS on Windows Server 2012, you can use the following PowerShell command to display error event information for the RRAS service:</span></span>

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    <span data-ttu-id="cd434-242">A propriedade *Mensagem* de cada entrada fornece uma descrição do erro.</span><span class="sxs-lookup"><span data-stu-id="cd434-242">The *Message* property of each entry provides a description of the error.</span></span> <span data-ttu-id="cd434-243">Alguns exemplos comuns são:</span><span class="sxs-lookup"><span data-stu-id="cd434-243">Some common examples are:</span></span>

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {41, 3, 0, 0}
        Index              : 14231
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: The network connection between your computer and
                             the VPN server could not be established because the remote server is not responding. This could
                             be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                             and the remote server is not configured to allow VPN connections. Please contact your
                             Administrator or your service provider to determine which device may be causing the problem.
        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                             your computer and the VPN server could not be established because the remote server is not
                             responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                             between your computer and the remote server is not configured to allow VPN connections. Please
                             contact your Administrator or your service provider to determine which device may be causing the
                             problem.}
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:26:02 PM
        TimeWritten        : 3/18/2016 1:26:02 PM
        UserName           :
        Site               :
        Container          :
        ```

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {233, 53, 0, 0}
        Index              : 14245
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                             unacceptable.
                             }
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:34:22 PM
        TimeWritten        : 3/18/2016 1:34:22 PM
        UserName           :
        Site               :
        Container          :
        ```

    <span data-ttu-id="cd434-244">Você também pode obter informações de log de eventos sobre tentativas de conexão por meio do serviço RRAS usando o seguinte comando do PowerShell:</span><span class="sxs-lookup"><span data-stu-id="cd434-244">You can also obtain event log information about attempts to connect through the RRAS service using the following PowerShell command:</span></span> 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    <span data-ttu-id="cd434-245">Em caso de falha de conexão, esse log conterá erros semelhantes ao seguinte:</span><span class="sxs-lookup"><span data-stu-id="cd434-245">In the event of a failure to connect, this log will contain errors that look similar to the following:</span></span>

    ```
    EventID            : 20227
    MachineName        : on-prem-vm
    Data               : {}
    Index              : 4203
    Category           : (0)
    CategoryNumber     : 0
    EntryType          : Error
    Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                         AzureGateway that has failed. The error code returned on failure is 809.
    Source             : RasClient
    ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
    InstanceId         : 20227
    TimeGenerated      : 3/18/2016 1:29:21 PM
    TimeWritten        : 3/18/2016 1:29:21 PM
    UserName           :
    Site               :
    Container          :
    ```

- <span data-ttu-id="cd434-246">**Verifique a conectividade e o roteamento no Gateway de VPN.**</span><span class="sxs-lookup"><span data-stu-id="cd434-246">**Verify connectivity and routing across the VPN gateway.**</span></span>

    <span data-ttu-id="cd434-247">O dispositivo de VPN pode não estar roteando o tráfego corretamento pelo Gateway de VPN do Azure.</span><span class="sxs-lookup"><span data-stu-id="cd434-247">The VPN appliance may not be correctly routing traffic through the Azure VPN Gateway.</span></span> <span data-ttu-id="cd434-248">Use uma ferramenta como o [PsPing][psping] para verificar a conectividade e o roteamento no Gateway de VPN.</span><span class="sxs-lookup"><span data-stu-id="cd434-248">Use a tool such as [PsPing][psping] to verify connectivity and routing across the VPN gateway.</span></span> <span data-ttu-id="cd434-249">Por exemplo, para testar a conectividade de um computador local com um servidor Web localizado na VNet, execute o seguinte comando (substituindo `<<web-server-address>>` pelo endereço do servidor Web):</span><span class="sxs-lookup"><span data-stu-id="cd434-249">For example, to test connectivity from an on-premises machine to a web server located on the VNet, run the following command (replacing `<<web-server-address>>` with the address of the web server):</span></span>

    ```
    PsPing -t <<web-server-address>>:80
    ```

    <span data-ttu-id="cd434-250">Se o computador local puder rotear o tráfego para o servidor Web, será exibida uma saída semelhante à seguinte:</span><span class="sxs-lookup"><span data-stu-id="cd434-250">If the on-premises machine can route traffic to the web server, you should see output similar to the following:</span></span>

    ```
    D:\PSTools>psping -t 10.20.0.5:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.0.5:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.0.5:80 (warmup): 6.21ms
    Connecting to 10.20.0.5:80: 3.79ms
    Connecting to 10.20.0.5:80: 3.44ms
    Connecting to 10.20.0.5:80: 4.81ms

      Sent = 3, Received = 3, Lost = 0 (0% loss),
      Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
    ```

    <span data-ttu-id="cd434-251">Se o computador local não puder se comunicar com o destino especificado, serão exibidas mensagens assim:</span><span class="sxs-lookup"><span data-stu-id="cd434-251">If the on-premises machine cannot communicate with the specified destination, you will see messages like this:</span></span>

    ```
    D:\PSTools>psping -t 10.20.1.6:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.1.6:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80:
      Sent = 3, Received = 0, Lost = 3 (100% loss),
      Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
    ```

- <span data-ttu-id="cd434-252">**Verifique se o firewall local permite a passagem do tráfego da VPN e se as portas corretas estão abertas.**</span><span class="sxs-lookup"><span data-stu-id="cd434-252">**Verify that the on-premises firewall allows VPN traffic to pass and that the correct ports are opened.**</span></span>

- <span data-ttu-id="cd434-253">**Verifique se o dispositivo de VPN local usa um método de criptografia [compatível com o Gateway de VPN do Azure][vpn-appliance].**</span><span class="sxs-lookup"><span data-stu-id="cd434-253">**Verify that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance].**</span></span> <span data-ttu-id="cd434-254">Para roteamento baseado em políticas, o Gateway de VPN do Azure dá suporte aos algoritmos de criptografia 3DES, AES256 e AES128.</span><span class="sxs-lookup"><span data-stu-id="cd434-254">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="cd434-255">Os gateways baseados em rota dão suporte para AES256 e 3DES.</span><span class="sxs-lookup"><span data-stu-id="cd434-255">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="cd434-256">As recomendações a seguir são úteis para determinar se há algum problema com o Gateway de VPN do Azure:</span><span class="sxs-lookup"><span data-stu-id="cd434-256">The following recommendations are useful for determining if there is a problem with the Azure VPN gateway:</span></span>

- <span data-ttu-id="cd434-257">**Examine os [logs de diagnóstico do Gateway de VPN do Azure][gateway-diagnostic-logs] para buscar possíveis problemas.**</span><span class="sxs-lookup"><span data-stu-id="cd434-257">**Examine [Azure VPN gateway diagnostic logs][gateway-diagnostic-logs] for potential issues.**</span></span>

- <span data-ttu-id="cd434-258">**Verifique se o dispositivo de VPN local e o Gateway de VPN do Azure estão configurados com a mesma chave de autenticação compartilhada.**</span><span class="sxs-lookup"><span data-stu-id="cd434-258">**Verify that the Azure VPN gateway and on-premises VPN appliance are configured with the same shared authentication key.**</span></span>

    <span data-ttu-id="cd434-259">Você pode exibir a chave compartilhada armazenada pelo Gateway de VPN do Azure usando o seguinte comando da CLI do Azure:</span><span class="sxs-lookup"><span data-stu-id="cd434-259">You can view the shared key stored by the Azure VPN gateway using the following Azure CLI command:</span></span>

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    <span data-ttu-id="cd434-260">Use o comando apropriado para seu dispositivo de VPN local para mostrar a chave compartilhada configurada para esse dispositivo.</span><span class="sxs-lookup"><span data-stu-id="cd434-260">Use the command appropriate for your on-premises VPN appliance to show the shared key configured for that appliance.</span></span>

    <span data-ttu-id="cd434-261">Verifique se a sub-rede *GatewaySubnet* que mantém o Gateway de VPN do Azure não está associada a um NSG.</span><span class="sxs-lookup"><span data-stu-id="cd434-261">Verify that the *GatewaySubnet* subnet holding the Azure VPN gateway is not associated with an NSG.</span></span>

    <span data-ttu-id="cd434-262">Você pode exibir os detalhes da sub-rede usando o seguinte comando da CLI do Azure:</span><span class="sxs-lookup"><span data-stu-id="cd434-262">You can view the subnet details using the following Azure CLI command:</span></span>

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    <span data-ttu-id="cd434-263">Verifique se não há nenhum campo de dados denominado *ID do Grupo de Segurança de Rede*. O exemplo a seguir mostra os resultados de uma instância da *GatewaySubnet* que tem um NSG atribuído (*VPN-Gateway-Group*).</span><span class="sxs-lookup"><span data-stu-id="cd434-263">Ensure there is no data field named *Network Security Group id*. The following example shows the results for an instance of the *GatewaySubnet* that has an assigned NSG (*VPN-Gateway-Group*).</span></span> <span data-ttu-id="cd434-264">Isso poderá impedir que o gateway funcione corretamente se houver regras definidas para este NSG.</span><span class="sxs-lookup"><span data-stu-id="cd434-264">This can prevent the gateway from working correctly if there are any rules defined for this NSG.</span></span>

    ```
    C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
        info:    Executing command network vnet subnet show
        + Looking up virtual network "profx-vnet"
        + Looking up the subnet "GatewaySubnet"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
        data:    Name                            : GatewaySubnet
        data:    Provisioning state              : Succeeded
        data:    Address prefix                  : 10.20.3.0/27
        data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
        info:    network vnet subnet show command OK
    ```

- <span data-ttu-id="cd434-265">**Verifique se as máquinas virtuais na VNet do Azure estão configuradas para permitir o tráfego proveniente de fora da VNet.**</span><span class="sxs-lookup"><span data-stu-id="cd434-265">**Verify that the virtual machines in the Azure VNet are configured to permit traffic coming in from outside the VNet.**</span></span>

    <span data-ttu-id="cd434-266">Verifique as regras do NSG associadas às sub-redes que contêm essas máquinas virtuais.</span><span class="sxs-lookup"><span data-stu-id="cd434-266">Check any NSG rules associated with subnets containing these virtual machines.</span></span> <span data-ttu-id="cd434-267">Você pode exibir todas as regras do NSG usando o seguinte comando da CLI do Azure:</span><span class="sxs-lookup"><span data-stu-id="cd434-267">You can view all NSG rules using the following Azure CLI command:</span></span>

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- <span data-ttu-id="cd434-268">**Verifique se o Gateway de VPN do Azure está conectado.**</span><span class="sxs-lookup"><span data-stu-id="cd434-268">**Verify that the Azure VPN gateway is connected.**</span></span>

    <span data-ttu-id="cd434-269">Você pode usar o seguinte comando do Azure PowerShell para verificar o status atual da conexão de VPN do Azure.</span><span class="sxs-lookup"><span data-stu-id="cd434-269">You can use the following Azure PowerShell command to check the current status of the Azure VPN connection.</span></span> <span data-ttu-id="cd434-270">O parâmetro `<<connection-name>>` é o nome da conexão de VPN do Azure que vincula o gateway de rede virtual e o gateway local.</span><span class="sxs-lookup"><span data-stu-id="cd434-270">The `<<connection-name>>` parameter is the name of the Azure VPN connection that links the virtual network gateway and the local gateway.</span></span>

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    <span data-ttu-id="cd434-271">Os snippets de código a seguir realçam a saída gerada quando o gateway está conectado (o primeiro exemplo) e desconectado (o segundo exemplo):</span><span class="sxs-lookup"><span data-stu-id="cd434-271">The following snippets highlight the output generated if the gateway is connected (the first example), and disconnected (the second example):</span></span>

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : Connected
    EgressBytesTransferred     : 55254803
    IngressBytesTransferred    : 32227221
    ProvisioningState          : Succeeded
    ...
    ```

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : NotConnected
    EgressBytesTransferred     : 0
    IngressBytesTransferred    : 0
    ProvisioningState          : Succeeded
    ...
    ```

<span data-ttu-id="cd434-272">As recomendações a seguir são úteis para determinar se há algum problema com a configuração da VM host, a utilização da largura de banda da rede ou o desempenho do aplicativo:</span><span class="sxs-lookup"><span data-stu-id="cd434-272">The following recommendations are useful for determining if there is an issue with Host VM configuration, network bandwidth utilization, or application performance:</span></span>

- <span data-ttu-id="cd434-273">**Verifique se o firewall no sistema operacional convidado em execução nas VMs do Azure na sub-rede está configurado corretamente para permitir o tráfego permitido dos intervalos de IP locais.**</span><span class="sxs-lookup"><span data-stu-id="cd434-273">**Verify that the firewall in the guest operating system running on the Azure VMs in the subnet is configured correctly to allow permitted traffic from the on-premises IP ranges.**</span></span>

- <span data-ttu-id="cd434-274">**Verifique se o volume de tráfego não está próximo ao limite da largura de banda disponível para o Gateway de VPN do Azure.**</span><span class="sxs-lookup"><span data-stu-id="cd434-274">**Verify that the volume of traffic is not close to the limit of the bandwidth available to the Azure VPN gateway.**</span></span>

    <span data-ttu-id="cd434-275">A maneira de verificar isso depende do dispositivo de VPN em execução local.</span><span class="sxs-lookup"><span data-stu-id="cd434-275">How to verify this depends on the VPN appliance running on-premises.</span></span> <span data-ttu-id="cd434-276">Por exemplo, se você estiver usando o RRAS no Windows Server 2012, será possível usar o Monitor de Desempenho para acompanhar o volume de dados recebido e transmitido pela conexão de VPN.</span><span class="sxs-lookup"><span data-stu-id="cd434-276">For example, if you are using RRAS on Windows Server 2012, you can use Performance Monitor to track the volume of data being received and transmitted over the VPN connection.</span></span> <span data-ttu-id="cd434-277">Usando o objeto *RAS Total*, selecione os contadores *Bytes recebidos/s* e *Bytes transmitidos/s*:</span><span class="sxs-lookup"><span data-stu-id="cd434-277">Using the *RAS Total* object, select the *Bytes Received/Sec* and *Bytes Transmitted/Sec* counters:</span></span>

    <span data-ttu-id="cd434-278">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="cd434-278">![[3]][3]</span></span>

    <span data-ttu-id="cd434-279">Compare os resultados com a largura de banda disponível para o Gateway de VPN (100 Mbps para SKUs do plano Básico e Standard e 200 Mbps para a SKU de alto desempenho):</span><span class="sxs-lookup"><span data-stu-id="cd434-279">You should compare the results with the bandwidth available to the VPN gateway (100 Mbps for the Basic and Standard SKUs, and 200 Mbps for the High Performance SKU):</span></span>

    <span data-ttu-id="cd434-280">![[4]][4]</span><span class="sxs-lookup"><span data-stu-id="cd434-280">![[4]][4]</span></span>

- <span data-ttu-id="cd434-281">**Verifique se você implantou o número de VMs e os tamanhos corretos para sua carga de aplicativo.**</span><span class="sxs-lookup"><span data-stu-id="cd434-281">**Verify that you have deployed the right number and size of VMs for your application load.**</span></span>

    <span data-ttu-id="cd434-282">Determine se alguma das máquinas virtuais na VNet do Azure está com a execução lenta.</span><span class="sxs-lookup"><span data-stu-id="cd434-282">Determine if any of the virtual machines in the Azure VNet are running slowly.</span></span> <span data-ttu-id="cd434-283">Se sim, elas podem estar sobrecarregadas, pode haver um número muito pequeno delas para lidar com a carga ou os balanceadores de carga podem não estar configurados corretamente.</span><span class="sxs-lookup"><span data-stu-id="cd434-283">If so, they may be overloaded, there may be too few to handle the load, or the load-balancers may not be configured correctly.</span></span> <span data-ttu-id="cd434-284">Para determinar isso, [capture e analise as informações de diagnóstico][azure-vm-diagnostics].</span><span class="sxs-lookup"><span data-stu-id="cd434-284">To determine this, [capture and analyze diagnostic information][azure-vm-diagnostics].</span></span> <span data-ttu-id="cd434-285">Você pode examinar os resultados usando o portal do Azure, mas muitas ferramentas de terceiros também estão disponíveis para fornecer informações detalhadas sobre os dados de desempenho.</span><span class="sxs-lookup"><span data-stu-id="cd434-285">You can examine the results using the Azure portal, but many third-party tools are also available that can provide detailed insights into the performance data.</span></span>

- <span data-ttu-id="cd434-286">**Verifique se que o aplicativo está fazendo um uso eficiente dos recursos de nuvem.**</span><span class="sxs-lookup"><span data-stu-id="cd434-286">**Verify that the application is making efficient use of cloud resources.**</span></span>

    <span data-ttu-id="cd434-287">Instrumente o código do aplicativo em execução em cada VM para determinar se os aplicativos estão fazendo o melhor uso possível dos recursos.</span><span class="sxs-lookup"><span data-stu-id="cd434-287">Instrument application code running on each VM to determine whether applications are making the best use of resources.</span></span> <span data-ttu-id="cd434-288">Você pode usar ferramentas como [Application Insights][application-insights].</span><span class="sxs-lookup"><span data-stu-id="cd434-288">You can use tools such as [Application Insights][application-insights].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="cd434-289">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="cd434-289">Deploy the solution</span></span>


<span data-ttu-id="cd434-290">**Pré-requisitos.**</span><span class="sxs-lookup"><span data-stu-id="cd434-290">**Prequisites.**</span></span> <span data-ttu-id="cd434-291">Você deve ter uma infraestrutura local existente já configurada com um dispositivo de rede adequado.</span><span class="sxs-lookup"><span data-stu-id="cd434-291">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="cd434-292">Para implantar a solução, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="cd434-292">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="cd434-293">Clique no botão abaixo:</span><span class="sxs-lookup"><span data-stu-id="cd434-293">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="cd434-294">Aguarde até o link abrir no Portal do Azure e siga estas etapas:</span><span class="sxs-lookup"><span data-stu-id="cd434-294">Wait for the link to open in the Azure portal, then follow these steps:</span></span> 
   * <span data-ttu-id="cd434-295">O nome do **Grupo de recursos** já está definido no arquivo de parâmetros, portanto, selecione **Criar novo** e digite `ra-hybrid-vpn-rg` na caixa de texto.</span><span class="sxs-lookup"><span data-stu-id="cd434-295">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-rg` in the text box.</span></span>
   * <span data-ttu-id="cd434-296">Selecione a região na caixa suspensa **Local**.</span><span class="sxs-lookup"><span data-stu-id="cd434-296">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="cd434-297">Não edite as caixas de texto **URI da raiz do modelo** ou **URI da raiz do parâmetro**.</span><span class="sxs-lookup"><span data-stu-id="cd434-297">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="cd434-298">Analise os termos e condições e clique na caixa de seleção **Concordo com os termos e condições declarados acima**.</span><span class="sxs-lookup"><span data-stu-id="cd434-298">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="cd434-299">Clique no botão **Comprar**.</span><span class="sxs-lookup"><span data-stu-id="cd434-299">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="cd434-300">Aguarde até que a implantação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="cd434-300">Wait for the deployment to complete.</span></span>



<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[azure-gateway-skus]: /azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: http://blogs.technet.com/b/keithmayer/archive/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell.aspx
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: http://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: http://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: http://blogs.technet.com/b/keithmayer/archive/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
[virtualNetworkGateway-parameters]: https://github.com/mspnp/hybrid-networking/vpn/parameters/virtualNetworkGateway.parameters.json
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "Rede híbrida que abrange as infraestruturas locais e do Azure"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Logs de auditoria no portal do Azure"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "Contadores de desempenho para monitorar o tráfego de rede da VPN"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "Gráfico de desempenho de rede da VPN de exemplo"
