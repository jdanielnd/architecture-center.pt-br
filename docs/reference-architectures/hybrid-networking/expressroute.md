---
title: Conectar uma rede local ao Azure usando o ExpressRoute
titleSuffix: Azure Reference Architectures
description: Implementar uma arquitetura de rede site a site segura que abranja uma rede virtual do Azure e uma rede local conectada usando o Azure ExpressRoute.
author: telmosampaio
ms.date: 10/22/2017
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, networking
ms.openlocfilehash: eff7d3e88cc9578b6d5ff83628f7d03b00717b5f
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487841"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute"></a>Conectar uma rede local ao Azure usando o ExpressRoute

Essa arquitetura de referência mostra como conectar uma rede local a redes virtuais no Azure usando o [Azure ExpressRoute][expressroute-introduction]. Conexões do ExpressRoute usam uma conexão privada dedicada por meio de um provedor de conectividade de terceiros. A conexão privada estende sua rede local para o Azure. [**Implantar esta solução**](#deploy-the-solution).

![[0]][0]

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura

A arquitetura consiste nos componentes a seguir.

- **Rede corporativa local**. Uma rede de área local privada em execução dentro de uma organização.

- **Circuito do ExpressRoute**. Um circuito de camada 2 ou de camada 3 fornecido pelo provedor de conectividade que une a rede local com o Azure por meio de roteadores de borda. O circuito usa a infraestrutura de hardware gerenciada pelo provedor de conectividade.

- **Roteadores de borda locais**. Roteadores que se conectam a rede local ao circuito gerenciado pelo provedor. Dependendo de como sua conexão é provisionada, talvez seja necessário fornecer endereços IP públicos usados pelos roteadores.
- **Roteadores de borda da Microsoft**. Dois roteadores em uma configuração altamente disponível de tipo ativo-ativo. Esses roteadores permitem que um provedor de conectividade conecte seus circuitos diretamente ao seu datacenter. Dependendo de como sua conexão é provisionada, talvez seja necessário fornecer endereços IP públicos usados pelos roteadores.

- **Redes virtuais do Azure (VNets)**. Cada VNet reside em uma única região do Azure e pode hospedar várias camadas de aplicativos. As camadas de aplicativo podem ser segmentadas usando sub-redes em cada VNet.

- **Serviços públicos do Azure**. Serviços do Azure que podem ser usados em um aplicativo híbrido. Esses serviços também estão disponíveis na Internet, mas acessá-los usando um circuito do ExpressRoute fornece baixa latência e desempenho mais previsível, porque o tráfego não passa pela Internet. Conexões são executadas usando [emparelhamento público][expressroute-peering], com os endereços pertencentes à sua organização ou fornecidos pelo seu provedor de conectividade.

- **Serviços do Office 365**. Os serviços fornecidos pela Microsoft e aplicativos do Office 365 publicamente disponíveis. Conexões são realizadas usando [emparelhamento da Microsoft][expressroute-peering], com os endereços pertencentes à sua organização ou fornecidos pelo seu provedor de conectividade. Você também pode se conectar diretamente ao Microsoft CRM Online por meio do emparelhamento da Microsoft.

- **Provedores de conectividade** (não mostrados). Empresas que fornecem uma conexão usando conectividade de camada 2 ou camada 3 entre o seu datacenter e um datacenter do Azure.

## <a name="recommendations"></a>Recomendações

As seguintes recomendações aplicam-se à maioria dos cenários. Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.

### <a name="connectivity-providers"></a>Provedores de conectividade

Selecione um provedor de conectividade do ExpressRoute adequado para sua localização. Para obter uma lista de provedores de conectividade disponíveis no local, use o seguinte comando do Azure PowerShell:

```powershell
Get-AzureRmExpressRouteServiceProvider
```

Provedores de conectividade do ExpressRoute conectam seu datacenter à Microsoft das seguintes maneiras:

- **Colocalizado em uma troca de nuvem**. Se você estiver colocalizado em uma instalação que tenha uma troca de nuvem, poderá solicitar conexões cruzadas virtuais ao Azure por meio da troca Ethernet do provedor da colocalização. Os provedores da colocalização podem oferecer conexões cruzadas de camada 2 ou conexões cruzadas gerenciadas de camada 3 entre sua infraestrutura na instalação de colocalização e o Azure.
- **Conexões Ethernet ponto a ponto**. Você pode conectar seus data centers/escritórios locais ao Azure por meio de links de Ethernet ponto a ponto. Os provedores de Ethernet ponto a ponto podem oferecer conexões de camada 2 ou conexões gerenciadas de camada 3 entre seu site e o Azure.
- **Redes qualquer para qualquer (IPVPN)**. Você pode integrar sua WAN (rede de longa distância) com o Azure. Provedores de Internet IPVPN (rede virtual privada de protocolo IP), normalmente uma VPN de mudança de rótulo multiprotocolo, oferecem conectividade de qualquer para qualquer entre suas filiais e data centers. O Azure pode ser interconectado à sua WAN para que ela fique exatamente igual a qualquer outra filial. Normalmente, os provedores de rede WAN oferecem conectividade gerenciada de camada 3.

Para obter mais informações sobre provedores de conectividade, consulte a [Introdução ao ExpressRoute][expressroute-introduction].

### <a name="expressroute-circuit"></a>Circuito do ExpressRoute

Verifique se a sua organização cumpre os [pré-requisitos do ExpressRoute][expressroute-prereqs] para se conectar ao Azure.

Se você ainda não o fez, adicione uma sub-rede denominada `GatewaySubnet` à sua VNet do Azure e crie um gateway de rede virtual do ExpressRoute usando o serviço de gateway VPN do Azure. Para obter mais informações sobre este processo, consulte [Fluxos de trabalho do ExpressRoute para provisionamento e estados do circuito][ExpressRoute-provisioning].

Crie um circuito do ExpressRoute conforme descrito a seguir:

1. Execute o seguinte comando do PowerShell:

    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. Envie o `ServiceKey` para o novo circuito para o provedor de serviços.

3. Aguarde até que o provedor provisione o circuito. Para verificar o estado de provisionamento de um circuito, execute o seguinte comando do PowerShell:

    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    O campo `Provisioning state` na seção `Service Provider` da saída será alterado de `NotProvisioned` para `Provisioned` quando o circuito estiver pronto.

    > [!NOTE]
    > Se você estiver usando uma conexão de camada 3, o provedor deverá configurar e gerenciar o roteamento para você. Você fornece as informações necessárias para habilitar o provedor a implementar as rotas apropriadas.
    >
    >

4. Se você estiver usando uma conexão de camada 2:

    1. Reserve duas sub-redes /30 compostas de endereços IP públicos válidos para cada tipo de emparelhamento que você deseja implementar. Essas sub-redes /30 serão usadas para fornecer endereços IP para os roteadores usados para o circuito. Se você estiver implementando emparelhamento privado, público e da Microsoft, você precisará de seis sub-redes /30 com endereços IP públicos válidos.

    2. Configure o roteamento para o circuito do ExpressRoute. Execute os seguinte comandos do PowerShell para cada tipo de emparelhamento a configurar (privado, público e da Microsoft). Para obter mais informações, consulte [Criar e modificar o roteamento de um circuito do ExpressRoute][configure-expressroute-routing].

        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. Reserve outro pool de endereços IP públicos válidos a ser usado para NAT (conversão de endereços de rede) para emparelhamento público e da Microsoft. É recomendável ter um pool diferente para cada emparelhamento. Especifica o pool para seu provedor de conectividade, assim ele pode configurar anúncios do Protocolo BGP (Border Gateway Protocol) para esses intervalos.

5. Execute os comandos do PowerShell a seguir para vincular suas VNets privadas ao circuito do ExpressRoute. Para obter mais informações, consulte [Vincular uma Rede Virtual a um circuito do ExpressRoute][link-vnet-to-expressroute].

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

Você pode conectar múltiplas VNets localizadas em regiões diferentes ao mesmo circuito do ExpressRoute, desde que todas as VNets e o circuito do ExpressRoute estejam localizados na mesma região geopolítica.

### <a name="troubleshooting"></a>solução de problemas

Se um circuito do ExpressRoute que funcionava anteriormente agora falha ao se conectar sem que tenha ocorrido nenhuma alteração de configuração local ou dentro de sua VNet privada, você provavelmente precisará entrar em contato com o provedor de conectividade e trabalhar com eles para corrigir o problema. Use os seguintes comandos do Powershell para verificar se o circuito do ExpressRoute foi configurado:

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

A saída desse comando mostra várias propriedades para o circuito, incluindo `ProvisioningState`, `CircuitProvisioningState` e `ServiceProviderProvisioningState` conforme mostrado abaixo.

```powershell
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
                                   }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : NotProvisioned
```

Se o `ProvisioningState` não está definido como `Succeeded` após uma tentativa de criar um novo circuito, remova o circuito usando o comando a seguir e tente criá-lo novamente.

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

Se seu provedor já tinha provisionado o circuito e o `ProvisioningState` está definido para `Failed` ou então, se o `CircuitProvisioningState` não é `Enabled`, entre em contato com seu provedor para obter assistência adicional.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Os circuitos do ExpressRoute fornecem um caminho de alta largura de banda entre redes. Em geral, quanto maior a largura de banda, maior o custo.

O ExpressRoute oferece dois [planos de preços][expressroute-pricing] para clientes, um plano limitado e um plano de dados ilimitado. Os encargos variam de acordo com a largura de banda do circuito. A largura de banda disponível provavelmente será diferente para cada provedor. Use o cmdlet `Get-AzureRmExpressRouteServiceProvider` para ver os provedores disponíveis na sua região e as larguras de banda que eles oferecem.

Um único circuito do ExpressRoute pode dar suporte a um determinado número de emparelhamentos e links de VNet. Para obter mais informações, consulte [limites do ExpressRoute](/azure/azure-subscription-service-limits).

A um custo adicional, o complemento ExpressRoute Premium oferece algumas funcionalidades adicionais:

- Aumento dos limites de rota para emparelhamento público e privado.
- Aumento do número de links de VNet por circuito do ExpressRoute.
- Conectividade global para serviços.

Consulte [Preços do ExpressRoute][expressroute-pricing] para obter detalhes.

Circuitos de ExpressRoute são projetados para permitir intermitências de rede temporárias de até duas vezes o limite de largura de banda que você adquiriu, sem nenhum custo adicional. Isso é alcançado por meio de links redundantes. No entanto, nem todos os provedores de conectividade dão suporte a esse recurso. Verifique se que o seu provedor de conectividade permite esse recurso antes de depender dele.

Embora alguns provedores permitam que você altere a largura de banda, verifique se você escolheu uma largura de banda inicial que ultrapassa a suas necessidades e deixa margem para crescimento. Se você precisar aumentar a largura de banda no futuro, haverá duas opções:

- Aumentar a largura de banda. Evite essa opção sempre que possível; inclusive, nem todos os provedores permitem aumentar dinamicamente a largura de banda. Mas se for necessário um aumento de largura de banda, verifique com seu provedor se eles dão suporte à alteração de propriedades de largura de banda do ExpressRoute por meio de comandos do Powershell. Se esse for o caso, execute os comandos abaixo.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    Você pode aumentar a largura de banda sem perda de conectividade. Fazer downgrade da largura de banda resultará em interrupções na conectividade, porque você deverá excluir o circuito e recriá-lo com a nova configuração.

- Alterar o plano de preços e/ou fazer upgrade para Premium. Para fazer isso, execute os comandos a seguir. A propriedade `Sku.Tier` pode ser `Standard` ou `Premium`; a propriedade `Sku.Name` pode ser `MeteredData` ou `UnlimitedData`.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > Verifique se a propriedade `Sku.Name` corresponde a `Sku.Tier` e `Sku.Family`. Se você alterar a família e camada, mas não o nome, a conexão será desabilitada.
    >
    >

    Você pode fazer upgrade do SKU sem interrupções, mas você não pode mudar do plano de preços ilimitado para o limitado. Ao fazer o downgrade do SKU, o consumo de largura de banda deve permanecer dentro do limite padrão do SKU padrão.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

O ExpressRoute não dá suporte a protocolos de redundância de roteador como HSRP (protocolo de roteamento de espera ativa) e VRRP (protocolo de redundância de roteador virtual) para implementar a alta disponibilidade. Em vez disso, ele usa um par redundante de sessões de BGP por emparelhamento. Para facilitar as conexões de alta disponibilidade à sua rede, o Azure provisiona duas portas redundantes em dois roteadores (parte do Microsoft edge) em uma configuração ativo-ativo.

Por padrão, as sessões BGP usam um valor de tempo limite de ociosidade de 60 segundos. Se uma sessão expirar três vezes (total de 180 segundos), o roteador será marcado como não disponível e todo o tráfego será redirecionado para o roteador restante. Esse tempo limite de 180 segundos pode ser muito longo para aplicativos críticos. Nesse caso, você pode alterar as configurações de tempo limite de BGP no roteador local para um valor menor.

Você poderá configurar a alta disponibilidade para a conexão do Azure de maneiras diferentes, dependendo do tipo de provedor que você usar e do número de circuitos do ExpressRoute e de conexões de gateway de rede virtual que você pretender configurar. O exemplo a seguir resume as opções de disponibilidade:

- Se você estiver usando uma conexão de camada 2, implante roteadores redundantes em sua rede local em uma configuração ativo-ativo. Conecte o circuito primário a um roteador e o circuito secundário ao outro. Isso lhe dará uma conexão altamente disponível em ambas as extremidades da conexão. Isso é necessário se você precisar de contrato de nível de serviço (SLA) do ExpressRoute. Consulte [SLA para o Azure ExpressRoute][sla-for-expressroute] para obter detalhes.

    O diagrama a seguir mostra uma configuração com roteadores locais redundantes conectados aos circuitos primários e secundários. Cada circuito manipula o tráfego para um emparelhamento público e um emparelhamento privado (um par de espaços de endereço /30 é designado a cada emparelhamento, conforme descrito na seção anterior).

    ![[1]][1]

- Se você estiver usando uma conexão de camada 3, verifique se que ele forneça sessões de BGP redundantes que controlem a disponibilidade para você.

- Conecte a VNet a vários circuitos do ExpressRoute fornecidos por diferentes provedores de serviço. Essa estratégia fornece funcionalidades adicionais de recuperação de desastre e alta disponibilidade.

- Configure uma VPN site a site como um caminho de failover para o ExpressRoute. Para obter mais informações sobre essa opção, consulte [Conectar uma rede local ao Azure usando o ExpressRoute com failover de VPN][highly-available-network-architecture]. Essa opção só se aplica ao emparelhamento privado. Para serviços do Azure e o Office 365, a Internet é o único caminho de failover.

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Você pode usar o [Kit de ferramentas do Azure conectividade (AzureCT)][azurect] para monitorar a conectividade entre o datacenter local e o Azure.

## <a name="security-considerations"></a>Considerações de segurança

Você pode configurar opções de segurança para sua conexão do Azure de maneiras diferentes, dependendo de suas necessidades de conformidade e questões de segurança.

O ExpressRoute funciona na camada 3. Ameaças na camada de aplicativo podem ser evitadas por meio de um dispositivo de segurança de rede que restringe o tráfego a recursos legítimos. Além disso, as conexões do ExpressRoute usando o emparelhamento público podem ser iniciadas apenas localmente. Isso impede que um serviço invasor acesse e comprometa dados locais da Internet.

Para maximizar a segurança, adicione dispositivos de segurança de rede entre a rede local e os roteadores de borda do provedor. Isso ajudará a restringir o fluxo de entrada de tráfego não autorizado da VNet:

![[2]][2]

Para fins de auditoria ou a conformidade, pode ser necessário impedir o acesso direto de componentes em execução na VNet à Internet e implementar [túnel forçado][forced-tunneling]. Nessa situação, o tráfego de Internet deve ser redirecionado por meio de um proxy executado localmente, de modo que possa ser auditado. O proxy pode ser configurado para bloquear a saída de tráfego não autorizado e filtrar tráfego de entrada potencialmente mal-intencionado.

![[3]][3]

Para maximizar a segurança, não habilite um endereço IP público para suas VMs e use os NSGs para garantir que essas VMs não fiquem acessíveis publicamente. VMs só devem estar disponíveis usando o endereço IP interno. Esses endereços podem se tornar acessíveis através da rede de do ExpressRoute, permitindo que a equipe de DevOps local realize a configuração ou a manutenção.

Se você deve expor pontos de extremidade de gerenciamento para VMs a uma rede externa, use NSGs ou listas de controle de acesso para restringir a visibilidade dessas portas para uma lista de permissões de endereços IP ou redes.

> [!NOTE]
> Por padrão, VMs do Azure implantadas por meio do Portal do Azure incluem um endereço IP público que fornece acesso de logon.
>

## <a name="deploy-the-solution"></a>Implantar a solução

**Pré-requisitos**. Você deve ter uma infraestrutura local existente já configurada com um dispositivo de rede adequado.

Para implantar a solução, execute as etapas a seguir.

<!-- markdownlint-disable MD033 -->

1. Clique no botão abaixo:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>

2. Aguarde até o link abrir no Portal do Azure e siga estas etapas:
   - O nome do **Grupo de recursos** já está definido no arquivo de parâmetros, portanto, selecione **Criar novo** e digite `ra-hybrid-er-rg` na caixa de texto.
   - Selecione a região na caixa suspensa **Local**.
   - Não edite as caixas de texto **URI da raiz do modelo** ou **URI da raiz do parâmetro**.
   - Analise os termos e condições e clique na caixa de seleção **Concordo com os termos e condições declarados acima**.
   - Clique no botão **Comprar**.

3. Aguarde até que a implantação seja concluída.

4. Clique no botão abaixo:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>

5. Aguarde até o link abrir no Portal do Azure e siga estas etapas:
   - Selecione **Usar existente** na seção **Grupo de recursos** e digite `ra-hybrid-er-rg` na caixa de texto.
   - Selecione a região na caixa suspensa **Local**.
   - Não edite as caixas de texto **URI da raiz do modelo** ou **URI da raiz do parâmetro**.
   - Analise os termos e condições e clique na caixa de seleção **Concordo com os termos e condições declarados acima**.
   - Clique no botão **Comprar**.

6. Aguarde até que a implantação seja concluída.

<!-- markdownlint-enable MD033 -->

<!-- links -->

[forced-tunneling]: ../dmz/secure-vnet-hybrid.md
[highly-available-network-architecture]: ./expressroute-vpn-failover.md

[expressroute-technical-overview]: /azure/expressroute/expressroute-introduction
[expressroute-prereqs]: /azure/expressroute/expressroute-prerequisites
[configure-expressroute-routing]: /azure/expressroute/expressroute-howto-routing-arm
[sla-for-expressroute]: https://azure.microsoft.com/support/legal/sla/expressroute
[link-vnet-to-expressroute]: /azure/expressroute/expressroute-howto-linkvnet-arm
[ExpressRoute-provisioning]: /azure/expressroute/expressroute-workflows
[expressroute-introduction]: /azure/expressroute/expressroute-introduction
[expressroute-peering]: /azure/expressroute/expressroute-circuit-peerings
[expressroute-pricing]: https://azure.microsoft.com/pricing/details/expressroute/
[expressroute-limits]: /azure/azure-subscription-service-limits#networking-limits
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: /powershell/azure/overview
[azure-cli]: /cli/azure/install-azure-cli

[0]: ./images/expressroute.png "Arquitetura de rede híbrida usando o Azure ExpressRoute"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "Usando roteadores redundantes com circuitos primários e secundários do ExpressRoute"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "Adicionar dispositivos de segurança para a rede local"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "Usando túnel forçado para auditar o tráfego direcionado à Internet"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "Localizando a ServiceKey de um circuito do ExpressRoute"
