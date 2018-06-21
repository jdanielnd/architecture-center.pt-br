---
title: Escolha uma solução para conectar uma rede local ao Azure
description: Compara arquiteturas de referência para conectar uma rede local ao Azure.
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541042"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="3095e-103">Escolha uma solução para conectar uma rede local ao Azure</span><span class="sxs-lookup"><span data-stu-id="3095e-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="3095e-104">Este artigo compara opções para conectar uma rede local a uma VNet (rede Virtual) do Azure.</span><span class="sxs-lookup"><span data-stu-id="3095e-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="3095e-105">Nós fornecemos uma arquitetura de referência e uma solução implantável para cada opção.</span><span class="sxs-lookup"><span data-stu-id="3095e-105">We provide a reference architecture and a deployable solution for each option.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="3095e-106">Conexão VPN</span><span class="sxs-lookup"><span data-stu-id="3095e-106">VPN connection</span></span>

<span data-ttu-id="3095e-107">Use uma VPN (rede virtual privada) para conectar sua rede local a uma VNet do Azure por meio de um túnel VPN IPsec.</span><span class="sxs-lookup"><span data-stu-id="3095e-107">Use a virtual private network (VPN) to connect your on-premises network with an Azure VNet through an IPSec VPN tunnel.</span></span>

<span data-ttu-id="3095e-108">Essa arquitetura é adequada a aplicativos híbridos em que o tráfego entre o hardware local e a nuvem provavelmente será leve ou se você estiver disposto a trocar um latência levemente maior pela flexibilidade e a potência de processamento da nuvem.</span><span class="sxs-lookup"><span data-stu-id="3095e-108">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="3095e-109">**Benefícios**</span><span class="sxs-lookup"><span data-stu-id="3095e-109">**Benefits**</span></span>

- <span data-ttu-id="3095e-110">Simples de configurar.</span><span class="sxs-lookup"><span data-stu-id="3095e-110">Simple to configure.</span></span>

<span data-ttu-id="3095e-111">**Desafios**</span><span class="sxs-lookup"><span data-stu-id="3095e-111">**Challenges**</span></span>

- <span data-ttu-id="3095e-112">Requer um dispositivo VPN local.</span><span class="sxs-lookup"><span data-stu-id="3095e-112">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="3095e-113">Embora a Microsoft garanta disponibilidade de 99,9% para cada Gateway de VPN, este SLA só trata cobre o gateway VPN e não sua conexão de rede para o gateway.</span><span class="sxs-lookup"><span data-stu-id="3095e-113">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this SLA only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="3095e-114">Uma conexão VPN sobre o Gateway de VPN do Azure atualmente tem suporte para um máximo de 200 Mbps de largura de banda.</span><span class="sxs-lookup"><span data-stu-id="3095e-114">A VPN connection over Azure VPN Gateway currently supports a maximum of 200 Mbps bandwidth.</span></span> <span data-ttu-id="3095e-115">Talvez seja necessário particionar sua rede virtual do Azure em várias conexões VPN caso você pretenda exceder essa taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="3095e-115">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="3095e-116">**[Leia mais…][vpn]**</span><span class="sxs-lookup"><span data-stu-id="3095e-116">**[Read more...][vpn]**</span></span>

## <a name="azure-expressroute-connection"></a><span data-ttu-id="3095e-117">Conexão do Azure ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="3095e-117">Azure ExpressRoute connection</span></span>

<span data-ttu-id="3095e-118">Conexões do ExpressRoute usam uma conexão privada dedicada por meio de um provedor de conectividade de terceiros.</span><span class="sxs-lookup"><span data-stu-id="3095e-118">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="3095e-119">A conexão privada estende sua rede local para o Azure.</span><span class="sxs-lookup"><span data-stu-id="3095e-119">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="3095e-120">Essa arquitetura é adequada para aplicativos híbridos executando cargas de trabalho críticas em grande escala que exigem um alto grau de escalabilidade.</span><span class="sxs-lookup"><span data-stu-id="3095e-120">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="3095e-121">**Benefícios**</span><span class="sxs-lookup"><span data-stu-id="3095e-121">**Benefits**</span></span>

- <span data-ttu-id="3095e-122">Largura de banda muito maior disponível; até 10 Gbps, dependendo do provedor de conectividade.</span><span class="sxs-lookup"><span data-stu-id="3095e-122">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="3095e-123">Dá suporte a dimensionamento dinâmico da largura de banda para ajudar a reduzir os custos durante períodos de menor demanda.</span><span class="sxs-lookup"><span data-stu-id="3095e-123">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="3095e-124">No entanto, nem todos os provedores de conectividade têm essa opção.</span><span class="sxs-lookup"><span data-stu-id="3095e-124">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="3095e-125">Pode permitir o acesso direto da sua organização a nuvens nacionais, dependendo do provedor de conectividade.</span><span class="sxs-lookup"><span data-stu-id="3095e-125">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="3095e-126">SLA de 99,9% de disponibilidade em toda a conexão.</span><span class="sxs-lookup"><span data-stu-id="3095e-126">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="3095e-127">**Desafios**</span><span class="sxs-lookup"><span data-stu-id="3095e-127">**Challenges**</span></span>

- <span data-ttu-id="3095e-128">A configuração pode ser complexa.</span><span class="sxs-lookup"><span data-stu-id="3095e-128">Can be complex to set up.</span></span> <span data-ttu-id="3095e-129">Criar uma conexão do ExpressRoute requer trabalhar com um provedor de conectividade de terceiros.</span><span class="sxs-lookup"><span data-stu-id="3095e-129">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="3095e-130">O provedor é responsável por provisionar a conexão de rede.</span><span class="sxs-lookup"><span data-stu-id="3095e-130">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="3095e-131">Requer roteadores de alta largura de banda localmente.</span><span class="sxs-lookup"><span data-stu-id="3095e-131">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="3095e-132">**[Leia mais…][expressroute]**</span><span class="sxs-lookup"><span data-stu-id="3095e-132">**[Read more...][expressroute]**</span></span>

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="3095e-133">Failover do ExpressRoute com VPN</span><span class="sxs-lookup"><span data-stu-id="3095e-133">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="3095e-134">Esta opção combina as duas anteriores, usando o ExpressRoute em condições normais, mas fazendo failover para uma conexão VPN se houver uma perda de conectividade no circuito do ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="3095e-134">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="3095e-135">Essa arquitetura é adequada para aplicativos híbridos que precisam de maior largura de banda do ExpressRoute e também exigem conectividade de rede altamente disponível.</span><span class="sxs-lookup"><span data-stu-id="3095e-135">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="3095e-136">**Benefícios**</span><span class="sxs-lookup"><span data-stu-id="3095e-136">**Benefits**</span></span>

- <span data-ttu-id="3095e-137">Alta disponibilidade se o circuito do ExpressRoute falhar, embora a conexão de fallback esteja em uma rede de largura de banda inferior.</span><span class="sxs-lookup"><span data-stu-id="3095e-137">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="3095e-138">**Desafios**</span><span class="sxs-lookup"><span data-stu-id="3095e-138">**Challenges**</span></span>

- <span data-ttu-id="3095e-139">Complexo de configurar.</span><span class="sxs-lookup"><span data-stu-id="3095e-139">Complex to configure.</span></span> <span data-ttu-id="3095e-140">Você precisa configurar uma conexão VPN e um circuito ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="3095e-140">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="3095e-141">Requer hardware redundante (dispositivos VPN) e uma conexão de Gateway VPN do Azure redundante pela qual você paga encargos.</span><span class="sxs-lookup"><span data-stu-id="3095e-141">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="3095e-142">**[Leia mais…][expressroute-vpn-failover]**</span><span class="sxs-lookup"><span data-stu-id="3095e-142">**[Read more...][expressroute-vpn-failover]**</span></span>

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md