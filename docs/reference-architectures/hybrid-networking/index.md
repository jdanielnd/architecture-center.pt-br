---
title: Conectar uma rede local ao Azure
titleSuffix: Azure Reference Architectures
description: Comparar arquiteturas de referência para conectar uma rede local ao Azure.
author: telmosampaio
ms.date: 07/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: networking
ms.openlocfilehash: 6172866b08197b0ca1cd3aabb3c14c01b4f06f9c
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486821"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="1a17c-103">Escolha uma solução para conectar uma rede local ao Azure</span><span class="sxs-lookup"><span data-stu-id="1a17c-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="1a17c-104">Este artigo compara opções para conectar uma rede local a uma VNet (rede Virtual) do Azure.</span><span class="sxs-lookup"><span data-stu-id="1a17c-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="1a17c-105">Para cada opção, existe uma arquitetura de referência mais detalhada disponível.</span><span class="sxs-lookup"><span data-stu-id="1a17c-105">For each option, a more detailed reference architecture is available.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="1a17c-106">Conexão VPN</span><span class="sxs-lookup"><span data-stu-id="1a17c-106">VPN connection</span></span>

<span data-ttu-id="1a17c-107">Um [gateway de VPN](/azure/vpn-gateway/vpn-gateway-about-vpngateways) é um tipo de gateway de rede virtual que envia o tráfego criptografado entre sua rede virtual do Azure e um caminho local.</span><span class="sxs-lookup"><span data-stu-id="1a17c-107">A [VPN gateway](/azure/vpn-gateway/vpn-gateway-about-vpngateways) is a type of virtual network gateway that sends encrypted traffic between an Azure virtual network and an on-premises location.</span></span> <span data-ttu-id="1a17c-108">O tráfego criptografado percorre a Internet pública.</span><span class="sxs-lookup"><span data-stu-id="1a17c-108">The encrypted traffic goes over the public Internet.</span></span>

<span data-ttu-id="1a17c-109">Essa arquitetura é adequada a aplicativos híbridos em que o tráfego entre o hardware local e a nuvem provavelmente será leve ou se você estiver disposto a trocar um latência levemente maior pela flexibilidade e a potência de processamento da nuvem.</span><span class="sxs-lookup"><span data-stu-id="1a17c-109">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

### <a name="benefits"></a><span data-ttu-id="1a17c-110">Benefícios</span><span class="sxs-lookup"><span data-stu-id="1a17c-110">Benefits</span></span>

- <span data-ttu-id="1a17c-111">Simples de configurar.</span><span class="sxs-lookup"><span data-stu-id="1a17c-111">Simple to configure.</span></span>

### <a name="challenges"></a><span data-ttu-id="1a17c-112">Desafios</span><span class="sxs-lookup"><span data-stu-id="1a17c-112">Challenges</span></span>

- <span data-ttu-id="1a17c-113">Requer um dispositivo VPN local.</span><span class="sxs-lookup"><span data-stu-id="1a17c-113">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="1a17c-114">Embora a Microsoft garanta disponibilidade de 99,9% para cada Gateway de VPN, este [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) só cobre o gateway VPN e não sua conexão de rede para o gateway.</span><span class="sxs-lookup"><span data-stu-id="1a17c-114">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="1a17c-115">Uma conexão VPN sobre o Gateway de VPN do Azure atualmente tem suporte para um máximo de 1,25 Gbps de largura de banda.</span><span class="sxs-lookup"><span data-stu-id="1a17c-115">A VPN connection over Azure VPN Gateway currently supports a maximum of 1.25 Gbps bandwidth.</span></span> <span data-ttu-id="1a17c-116">Talvez seja necessário particionar sua rede virtual do Azure em várias conexões VPN caso você pretenda exceder essa taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="1a17c-116">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

### <a name="reference-architecture"></a><span data-ttu-id="1a17c-117">Arquitetura de referência</span><span class="sxs-lookup"><span data-stu-id="1a17c-117">Reference architecture</span></span>

- [<span data-ttu-id="1a17c-118">Rede híbrida usando um gateway de VPN</span><span class="sxs-lookup"><span data-stu-id="1a17c-118">Hybrid network with VPN gateway</span></span>](./vpn.md)

<!-- markdownlint-disable MD024 -->

## <a name="azure-expressroute-connection"></a><span data-ttu-id="1a17c-119">Conexão do Azure ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="1a17c-119">Azure ExpressRoute connection</span></span>

<span data-ttu-id="1a17c-120">Conexões do [ExpressRoute](/azure/expressroute/) usam uma conexão privada dedicada por meio de um provedor de conectividade de terceiros.</span><span class="sxs-lookup"><span data-stu-id="1a17c-120">[ExpressRoute](/azure/expressroute/) connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="1a17c-121">A conexão privada estende sua rede local para o Azure.</span><span class="sxs-lookup"><span data-stu-id="1a17c-121">The private connection extends your on-premises network into Azure.</span></span>

<span data-ttu-id="1a17c-122">Essa arquitetura é adequada para aplicativos híbridos executando cargas de trabalho críticas em grande escala que exigem um alto grau de escalabilidade.</span><span class="sxs-lookup"><span data-stu-id="1a17c-122">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span>

### <a name="benefits"></a><span data-ttu-id="1a17c-123">Benefícios</span><span class="sxs-lookup"><span data-stu-id="1a17c-123">Benefits</span></span>

- <span data-ttu-id="1a17c-124">Largura de banda muito maior disponível; até 10 Gbps, dependendo do provedor de conectividade.</span><span class="sxs-lookup"><span data-stu-id="1a17c-124">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="1a17c-125">Dá suporte a dimensionamento dinâmico da largura de banda para ajudar a reduzir os custos durante períodos de menor demanda.</span><span class="sxs-lookup"><span data-stu-id="1a17c-125">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="1a17c-126">No entanto, nem todos os provedores de conectividade têm essa opção.</span><span class="sxs-lookup"><span data-stu-id="1a17c-126">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="1a17c-127">Pode permitir o acesso direto da sua organização a nuvens nacionais, dependendo do provedor de conectividade.</span><span class="sxs-lookup"><span data-stu-id="1a17c-127">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="1a17c-128">SLA de 99,9% de disponibilidade em toda a conexão.</span><span class="sxs-lookup"><span data-stu-id="1a17c-128">99.9% availability SLA across the entire connection.</span></span>

### <a name="challenges"></a><span data-ttu-id="1a17c-129">Desafios</span><span class="sxs-lookup"><span data-stu-id="1a17c-129">Challenges</span></span>

- <span data-ttu-id="1a17c-130">A configuração pode ser complexa.</span><span class="sxs-lookup"><span data-stu-id="1a17c-130">Can be complex to set up.</span></span> <span data-ttu-id="1a17c-131">Criar uma conexão do ExpressRoute requer trabalhar com um provedor de conectividade de terceiros.</span><span class="sxs-lookup"><span data-stu-id="1a17c-131">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="1a17c-132">O provedor é responsável por provisionar a conexão de rede.</span><span class="sxs-lookup"><span data-stu-id="1a17c-132">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="1a17c-133">Requer roteadores de alta largura de banda localmente.</span><span class="sxs-lookup"><span data-stu-id="1a17c-133">Requires high-bandwidth routers on-premises.</span></span>

### <a name="reference-architecture"></a><span data-ttu-id="1a17c-134">Arquitetura de referência</span><span class="sxs-lookup"><span data-stu-id="1a17c-134">Reference architecture</span></span>

- [<span data-ttu-id="1a17c-135">Rede híbrida com o ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="1a17c-135">Hybrid network with ExpressRoute</span></span>](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="1a17c-136">Failover do ExpressRoute com VPN</span><span class="sxs-lookup"><span data-stu-id="1a17c-136">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="1a17c-137">Esta opção combina as duas anteriores, usando o ExpressRoute em condições normais, mas fazendo failover para uma conexão VPN se houver uma perda de conectividade no circuito do ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="1a17c-137">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="1a17c-138">Essa arquitetura é adequada para aplicativos híbridos que precisam de maior largura de banda do ExpressRoute e também exigem conectividade de rede altamente disponível.</span><span class="sxs-lookup"><span data-stu-id="1a17c-138">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span>

### <a name="benefits"></a><span data-ttu-id="1a17c-139">Benefícios</span><span class="sxs-lookup"><span data-stu-id="1a17c-139">Benefits</span></span>

- <span data-ttu-id="1a17c-140">Alta disponibilidade se o circuito do ExpressRoute falhar, embora a conexão de fallback esteja em uma rede de largura de banda inferior.</span><span class="sxs-lookup"><span data-stu-id="1a17c-140">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

### <a name="challenges"></a><span data-ttu-id="1a17c-141">Desafios</span><span class="sxs-lookup"><span data-stu-id="1a17c-141">Challenges</span></span>

- <span data-ttu-id="1a17c-142">Complexo de configurar.</span><span class="sxs-lookup"><span data-stu-id="1a17c-142">Complex to configure.</span></span> <span data-ttu-id="1a17c-143">Você precisa configurar uma conexão VPN e um circuito ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="1a17c-143">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="1a17c-144">Requer hardware redundante (dispositivos VPN) e uma conexão de Gateway VPN do Azure redundante pela qual você paga encargos.</span><span class="sxs-lookup"><span data-stu-id="1a17c-144">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

### <a name="reference-architecture"></a><span data-ttu-id="1a17c-145">Arquitetura de referência</span><span class="sxs-lookup"><span data-stu-id="1a17c-145">Reference architecture</span></span>

- [<span data-ttu-id="1a17c-146">Rede híbrida com o ExpressRoute e failover de VPN</span><span class="sxs-lookup"><span data-stu-id="1a17c-146">Hybrid network with ExpressRoute and VPN failover</span></span>](./expressroute-vpn-failover.md)

<!-- markdownlint-disable MD024 -->

## <a name="hub-spoke-network-topology"></a><span data-ttu-id="1a17c-147">Topologia de rede hub-spoke</span><span class="sxs-lookup"><span data-stu-id="1a17c-147">Hub-spoke network topology</span></span>

<span data-ttu-id="1a17c-148">Uma topologia de rede hub-spoke é uma maneira de isolar as cargas de trabalho enquanto compartilha os serviços, como a identidade e a segurança.</span><span class="sxs-lookup"><span data-stu-id="1a17c-148">A hub-spoke network topology is a way to isolate workloads while sharing services such as identity and security.</span></span> <span data-ttu-id="1a17c-149">O hub é uma VNet (rede virtual) no Azure que atua como ponto central de conectividade para sua rede local.</span><span class="sxs-lookup"><span data-stu-id="1a17c-149">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="1a17c-150">Os spokes são VNets que se emparelham com o hub.</span><span class="sxs-lookup"><span data-stu-id="1a17c-150">The spokes are VNets that peer with the hub.</span></span> <span data-ttu-id="1a17c-151">Os serviços compartilhados são implantados no hub, enquanto as cargas de trabalho individuais são implantadas como spokes.</span><span class="sxs-lookup"><span data-stu-id="1a17c-151">Shared services are deployed in the hub, while individual workloads are deployed as spokes.</span></span>

### <a name="reference-architectures"></a><span data-ttu-id="1a17c-152">Arquiteturas de referência</span><span class="sxs-lookup"><span data-stu-id="1a17c-152">Reference architectures</span></span>

- [<span data-ttu-id="1a17c-153">Topologia hub-spoke</span><span class="sxs-lookup"><span data-stu-id="1a17c-153">Hub-spoke topology</span></span>](./hub-spoke.md)
- [<span data-ttu-id="1a17c-154">Topologia hub-spoke com serviços compartilhados</span><span class="sxs-lookup"><span data-stu-id="1a17c-154">Hub-spoke with shared services</span></span>](./shared-services.md)
