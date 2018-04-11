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
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a>Escolha uma solução para conectar uma rede local ao Azure

Este artigo compara opções para conectar uma rede local a uma VNet (rede Virtual) do Azure. Nós fornecemos uma arquitetura de referência e uma solução implantável para cada opção.

## <a name="vpn-connection"></a>Conexão VPN

Use uma VPN (rede virtual privada) para conectar sua rede local a uma VNet do Azure por meio de um túnel VPN IPsec.

Essa arquitetura é adequada a aplicativos híbridos em que o tráfego entre o hardware local e a nuvem provavelmente será leve ou se você estiver disposto a trocar um latência levemente maior pela flexibilidade e a potência de processamento da nuvem.

**Benefícios**

- Simples de configurar.

**Desafios**

- Requer um dispositivo VPN local.
- Embora a Microsoft garanta disponibilidade de 99,9% para cada Gateway de VPN, este SLA só trata cobre o gateway VPN e não sua conexão de rede para o gateway.
- Uma conexão VPN sobre o Gateway de VPN do Azure atualmente tem suporte para um máximo de 200 Mbps de largura de banda. Talvez seja necessário particionar sua rede virtual do Azure em várias conexões VPN caso você pretenda exceder essa taxa de transferência.

**[Leia mais…][vpn]**

## <a name="azure-expressroute-connection"></a>Conexão do Azure ExpressRoute

Conexões do ExpressRoute usam uma conexão privada dedicada por meio de um provedor de conectividade de terceiros. A conexão privada estende sua rede local para o Azure. 

Essa arquitetura é adequada para aplicativos híbridos executando cargas de trabalho críticas em grande escala que exigem um alto grau de escalabilidade. 

**Benefícios**

- Largura de banda muito maior disponível; até 10 Gbps, dependendo do provedor de conectividade.
- Dá suporte a dimensionamento dinâmico da largura de banda para ajudar a reduzir os custos durante períodos de menor demanda. No entanto, nem todos os provedores de conectividade têm essa opção.
- Pode permitir o acesso direto da sua organização a nuvens nacionais, dependendo do provedor de conectividade.
- SLA de 99,9% de disponibilidade em toda a conexão.

**Desafios**

- A configuração pode ser complexa. Criar uma conexão do ExpressRoute requer trabalhar com um provedor de conectividade de terceiros. O provedor é responsável por provisionar a conexão de rede.
- Requer roteadores de alta largura de banda localmente.

**[Leia mais…][expressroute]**

## <a name="expressroute-with-vpn-failover"></a>Failover do ExpressRoute com VPN

Esta opção combina as duas anteriores, usando o ExpressRoute em condições normais, mas fazendo failover para uma conexão VPN se houver uma perda de conectividade no circuito do ExpressRoute.

Essa arquitetura é adequada para aplicativos híbridos que precisam de maior largura de banda do ExpressRoute e também exigem conectividade de rede altamente disponível. 

**Benefícios**

- Alta disponibilidade se o circuito do ExpressRoute falhar, embora a conexão de fallback esteja em uma rede de largura de banda inferior.

**Desafios**

- Complexo de configurar. Você precisa configurar uma conexão VPN e um circuito ExpressRoute.
- Requer hardware redundante (dispositivos VPN) e uma conexão de Gateway VPN do Azure redundante pela qual você paga encargos.

**[Leia mais…][expressroute-vpn-failover]**

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md