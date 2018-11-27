---
title: Escolha uma solução para conectar uma rede local ao Azure
description: Compara arquiteturas de referência para conectar uma rede local ao Azure.
author: telmosampaio
ms.date: 07/02/2018
ms.openlocfilehash: a9e2a212d65530e714635bbfae3a57766e77c3a6
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295473"
---
# <a name="connect-an-on-premises-network-to-azure"></a>Conectar uma rede local ao Azure

Este artigo compara opções para conectar uma rede local a uma VNet (rede Virtual) do Azure. Para cada opção, existe uma arquitetura de referência mais detalhada disponível.

## <a name="vpn-connection"></a>Conexão VPN

Um [gateway de VPN](/azure/vpn-gateway/vpn-gateway-about-vpngateways) é um tipo de gateway de rede virtual que envia o tráfego criptografado entre sua rede virtual do Azure e um caminho local. O tráfego criptografado percorre a Internet pública.

Essa arquitetura é adequada a aplicativos híbridos em que o tráfego entre o hardware local e a nuvem provavelmente será leve ou se você estiver disposto a trocar um latência levemente maior pela flexibilidade e a potência de processamento da nuvem.

**Benefícios**

- Simples de configurar.

**Desafios**

- Requer um dispositivo VPN local.
- Embora a Microsoft garanta disponibilidade de 99,9% para cada Gateway de VPN, este [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) só cobre o gateway VPN e não sua conexão de rede para o gateway.
- Uma conexão VPN sobre o Gateway de VPN do Azure atualmente tem suporte para um máximo de 1,25 Gbps de largura de banda. Talvez seja necessário particionar sua rede virtual do Azure em várias conexões VPN caso você pretenda exceder essa taxa de transferência.

**Arquitetura de referência**

- [Rede híbrida usando um gateway de VPN](./vpn.md)

## <a name="azure-expressroute-connection"></a>Conexão do Azure ExpressRoute

Conexões do [ExpressRoute](/azure/expressroute/) usam uma conexão privada dedicada por meio de um provedor de conectividade de terceiros. A conexão privada estende sua rede local para o Azure. 

Essa arquitetura é adequada para aplicativos híbridos executando cargas de trabalho críticas em grande escala que exigem um alto grau de escalabilidade. 

**Benefícios**

- Largura de banda muito maior disponível; até 10 Gbps, dependendo do provedor de conectividade.
- Dá suporte a dimensionamento dinâmico da largura de banda para ajudar a reduzir os custos durante períodos de menor demanda. No entanto, nem todos os provedores de conectividade têm essa opção.
- Pode permitir o acesso direto da sua organização a nuvens nacionais, dependendo do provedor de conectividade.
- SLA de 99,9% de disponibilidade em toda a conexão.

**Desafios**

- A configuração pode ser complexa. Criar uma conexão do ExpressRoute requer trabalhar com um provedor de conectividade de terceiros. O provedor é responsável por provisionar a conexão de rede.
- Requer roteadores de alta largura de banda localmente.

**Arquitetura de referência**

- [Rede híbrida com o ExpressRoute](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a>Failover do ExpressRoute com VPN

Esta opção combina as duas anteriores, usando o ExpressRoute em condições normais, mas fazendo failover para uma conexão VPN se houver uma perda de conectividade no circuito do ExpressRoute.

Essa arquitetura é adequada para aplicativos híbridos que precisam de maior largura de banda do ExpressRoute e também exigem conectividade de rede altamente disponível. 

**Benefícios**

- Alta disponibilidade se o circuito do ExpressRoute falhar, embora a conexão de fallback esteja em uma rede de largura de banda inferior.

**Desafios**

- Complexo de configurar. Você precisa configurar uma conexão VPN e um circuito ExpressRoute.
- Requer hardware redundante (dispositivos VPN) e uma conexão de Gateway VPN do Azure redundante pela qual você paga encargos.

**Arquitetura de referência**

- [Rede híbrida com o ExpressRoute e failover de VPN](./expressroute-vpn-failover.md)


## <a name="hub-spoke-network-topology"></a>Topologia de rede hub-spoke

Uma topologia de rede hub-spoke é uma maneira de isolar as cargas de trabalho enquanto compartilha os serviços, como a identidade e a segurança. O hub é uma VNet (rede virtual) no Azure que atua como ponto central de conectividade para sua rede local. Os spokes são VNets que se emparelham com o hub. Os serviços compartilhados são implantados no hub, enquanto as cargas de trabalho individuais são implantadas como spokes.


**Arquiteturas de referência**

- [Topologia hub-spoke](./hub-spoke.md)
- [Topologia hub-spoke com serviços compartilhados](./shared-services.md)
