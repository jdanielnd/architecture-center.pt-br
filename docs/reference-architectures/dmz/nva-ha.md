---
title: Implantar soluções de virtualização de rede de alta disponibilidade
description: Como implantar soluções de virtualização de rede em alta disponibilidade.
author: telmosampaio
ms.date: 12/06/2016
pnp.series.title: Network DMZ
pnp.series.prev: secure-vnet-dmz
cardTitle: Deploy highly available network virtual appliances
ms.openlocfilehash: fe279eea3f9cb024d6c6c14943013b9b9a87bc9c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
---
# <a name="deploy-highly-available-network-virtual-appliances"></a>Implantar soluções de virtualização de rede altamente disponíveis

Este artigo mostra como implantar um conjunto de NVAs (soluções de virtualização de rede) para alta disponibilidade no Azure. Uma NVA normalmente é usada para controlar o fluxo de tráfego de rede de uma rede de perímetro, também conhecida como DMZ, para outras redes ou sub-redes. Para saber mais sobre a implementação de uma DMZ no Azure, consulte [Serviços em nuvem da Microsoft e segurança de rede][cloud-security]. O artigo inclui as arquiteturas de exemplo apenas de entrada, apenas de saída e de entrada e saída. 

<strong>Pré-requisitos:</strong> este artigo pressupõe um entendimento básico de redes do Azure, [balanceadores de carga do Azure][lb-overview] e UDRs [(rotas definidas pelo usuário)][udr-overview]. 


## <a name="architecture-diagrams"></a>Diagramas de arquitetura

Uma NVA pode ser implantada em uma DMZ em muitas arquiteturas diferentes. Por exemplo, a figura a seguir ilustra o uso de uma [única NVA][nva-scenario] para entrada. 

![[0]][0]

Nessa arquitetura, a NVA fornece um limite de rede segura marcando todo tráfego de rede de entrada e saída e passando apenas o tráfego que atende às regras de segurança de rede. No entanto, o fato de que todo o tráfego de rede deve passar pela NVA significa que ela é um ponto único de falha na rede. Se a NVA falhar, não haverá nenhum outro caminho para o tráfego de rede e todas as sub-redes de back-end ficarão indisponíveis.

Para tornar uma NVA altamente disponível, implante mais de uma NVA em um conjunto de disponibilidade.    

As arquiteturas a seguir descrevem os recursos e a configuração necessários para NVAs altamente disponíveis:

| Solução | Benefícios | Considerações |
| --- | --- | --- |
| [Entrada com NVAs na camada 7][ingress-with-layer-7] |Todos os nós de NVA estão ativos |Requer uma NVA que pode encerrar conexões e usar SNAT</br> Requer um conjunto separado de NVAs para o tráfego proveniente da Internet e do Azure </br> Só pode ser usado para tráfego proveniente de fora do Azure |
| [Saída com NVAs na camada 7][egress-with-layer-7] |Todos os nós de NVA estão ativos | Requer uma NVA que pode encerrar conexões e implementa SNAT (conversão de endereços de rede de origem)
| [Entrada-saída com NVAs na camada 7][ingress-egress-with-layer-7] |Todos os nós estão ativos<br/>Capaz de lidar com tráfego originado no Azure |Requer uma NVA que pode encerrar conexões e usar SNAT<br/>Requer um conjunto separado de NVAs para o tráfego proveniente da Internet e do Azure |
| [Opção PIP-UDR][pip-udr-switch] |Conjunto único de NVAs para todo o tráfego<br/>Pode lidar com todo o tráfego (sem limite nas regras de porta) |Ativo-passivo<br/>Requer um processo de failover |

## <a name="ingress-with-layer-7-nvas"></a>Entrada com NVAs na camada 7

A figura a seguir mostra uma arquitetura de alta disponibilidade que implementa uma DMZ de entrada por atrás de um balanceador de carga voltado para a Internet. Essa arquitetura é projetada para fornecer conectividade a cargas de trabalho do Azure para o tráfego da camada 7, como HTTP ou HTTPS:

![[1]][1]

A vantagem dessa arquitetura é que todas as NVAs estão ativas e, se uma falhar, o balanceador de carga direciona o tráfego de rede para a outra NVA. Ambas NVAs encaminham o tráfego para o balanceador de carga interno, portanto, desde que uma NVA esteja ativa, o tráfego continuará a fluir. As NVAs são necessárias para terminar o tráfego SSL destinado a VMs da camada da Web. Essas NVAs não podem ser estendidas para lidar com o tráfego local porque este requer outro conjunto dedicado de NVAs com suas próprias rotas de rede.

> [!NOTE]
> Essa arquitetura é usada nas arquiteturas de referência de [DMZ entre o Azure e seu datacenter local][dmz-on-prem] e de [DMZ entre o Azure e a Internet][dmz-internet]. Todas essas arquiteturas de referência incluem uma solução de implantação que você pode usar. Siga os links abaixo para obter mais informações.

## <a name="egress-with-layer-7-nvas"></a>Saída com NVAs na camada 7

A arquitetura anterior pode ser expandida para fornecer uma DMZ de saída para solicitações originadas na carga de trabalho do Azure. A arquitetura a seguir é projetada para fornecer alta disponibilidade de NVAs na DMZ para o tráfego de camada 7, como HTTP ou HTTPS:

![[2]][2]

Nessa arquitetura, todo o tráfego originado no Azure é encaminhado para um balanceador de carga interno. O balanceador de carga distribui solicitações de saída entre um conjunto de NVAs. Essas NVAs direcionam o tráfego para a Internet usando os endereços IP públicos individuais.

> [!NOTE]
> Essa arquitetura é usada nas arquiteturas de referência de [DMZ entre o Azure e seu datacenter local][dmz-on-prem] e de [DMZ entre o Azure e a Internet][dmz-internet]. Todas essas arquiteturas de referência incluem uma solução de implantação que você pode usar. Siga os links abaixo para obter mais informações.

## <a name="ingress-egress-with-layer-7-nvas"></a>Entrada-saída com NVAs na camada 7

Nas duas arquiteturas anteriores, havia uma DMZ separada para entrada e saída. A arquitetura a seguir demonstra como criar uma DMZ que pode ser usada tanto para entrada quanto saída para o tráfego da camada 7, como HTTP ou HTTPS: 

![[4]][4]

Nesta arquitetura, as NVAs processam solicitações de entrada do gateway de aplicativo. As NVAs também processam solicitações de saída de VMs de carga de trabalho no pool de back-end do balanceador de carga. Como o tráfego de entrada é roteado com um gateway de aplicativo e o tráfego de saída é roteado com um balanceador de carga, os NVAs são responsáveis por manter a afinidade de sessão. Ou seja, o gateway de aplicativo mantém um mapeamento de solicitações de entrada e saída para que ele possa encaminhar a resposta correta para o solicitante original. No entanto, o balanceador de carga interno não tem acesso aos mapeamentos de gateway de aplicativo e usa sua própria lógica para enviar respostas para as NVAs. É possível que o balanceador de carga possa enviar uma resposta para uma NVA que não recebeu a solicitação do gateway de aplicativo inicialmente. Nesse caso, as NVAs devem se comunicar e transferir a resposta entre elas para que a NVA correta possa encaminhar a resposta para o gateway de aplicativo.

> [!NOTE]
> Você também pode resolver o problema de roteamento assimétrico garantindo que as NVAs executem a SNAT (conversão de endereços de rede de origem) de entrada. Isso substituiria o IP de origem original do solicitante por um dos endereços IP da NVA usada no fluxo de entrada. Isso garante que você possa usar várias NVAs por vez, preservando ainda a simetria de rota.

## <a name="pip-udr-switch-with-layer-4-nvas"></a>Opção PIP-UDR com NVAs da camada 4

A arquitetura a seguir demonstra uma arquitetura com uma NVA ativa e uma passiva. Essa arquitetura lida trata tanto a entrada quanto a saída para o tráfego da camada 4: 

![[3]][3]

Essa arquitetura é semelhante à primeira arquitetura discutida neste artigo. Aquela arquitetura incluía uma única NVA que aceitava e filtrava solicitações de entrada da camada 4. Essa arquitetura adiciona uma segunda NVA passiva para fornecer alta disponibilidade. Se a NVA ativa falhar, a NVA passiva fica ativa e o PIP e o UDR são alterados para apontar para os NICs na NVA que está ativa no momento. Essas alterações do UDR e PIP podem ser feitas manualmente ou usando um processo automatizado. O processo automatizado normalmente é daemon ou outro serviço de monitoramento em execução no Azure. Ele consulta uma investigação de integridade na NVA ativa e executa a opção UDR e PIP quando detecta uma falha da NVA. 

A figura anterior mostra um exemplo de cluster [ZooKeeper][zookeeper] que fornece um daemon de alta disponibilidade. Dentro do cluster do ZooKeeper, um quorum de nós elege um líder. Se o líder falhar, os nós restantes elegerão um novo líder. Para essa arquitetura, o nó líder executa o daemon que consulta o ponto de extremidade de integridade na NVA. Se a NVA não responder à investigação de integridade, o daemon ativará a NVA passiva. Em seguida, o daemon chama a API REST do Azure para remover o PIP da NVA com falha e a anexa ao NVA recém-ativado. O daemon modifica então o UDR para apontar para o endereço IP interno da NVA recém-ativada.

> [!NOTE]
> Não inclua os nós do ZooKeeper em uma sub-rede que só é acessível por meio de uma rota que inclui a NVA. Caso contrário, os nós do ZooKeeper ficarão inacessíveis se a NVA falhar. Caso o daemon falhe por algum motivo, não será possível acessar nenhum nó do ZooKeeper para diagnosticar o problema. 

<!--### Solution Deployment-->

<!-- instructions for deploying this solution here --> 

## <a name="next-steps"></a>Próximas etapas
* Saiba como [implementar uma DMZ entre o Azure e o datacenter local][dmz-on-prem] usando NVAs da camada 7.
* Saiba como [implementar uma DMZ entre o Azure e a Internet][dmz-internet] usando NVAs da camada 7.

<!-- links -->
[cloud-security]: /azure/best-practices-network-security
[dmz-on-prem]: ./secure-vnet-hybrid.md
[dmz-internet]: ./secure-vnet-dmz.md
[egress-with-layer-7]: #egress-with-layer-7-nvas
[ingress-with-layer-7]: #ingress-with-layer-7-nvas
[ingress-egress-with-layer-7]: #ingress-egress-with-layer-7-nvas
[lb-overview]: /azure/load-balancer/load-balancer-overview/
[nva-scenario]: /azure/virtual-network/virtual-network-scenario-udr-gw-nva/
[pip-udr-switch]: #pip-udr-switch-with-layer-4-nvas
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview/
[zookeeper]: https://zookeeper.apache.org/

<!-- images -->
[0]: ./images/nva-ha/single-nva.png "Arquitetura NVA única"
[1]: ./images/nva-ha/l7-ingress.png "Entrada da camada 7"
[2]: ./images/nva-ha/l7-ingress-egress.png "Saída da camada 7"
[3]: ./images/nva-ha/active-passive.png "Cluster ativo-passivo"
[4]: ./images/nva-ha/l7-ingress-egress-ag.png
