---
title: Conectar uma rede local ao Azure usando VPN
titleSuffix: Azure Reference Architectures
description: Implementar uma arquitetura de rede site a site segura que abranja uma rede virtual do Azure e uma rede local conectada usando uma VPN.
author: RohitSharma-pnp
ms.date: 10/22/2018
ms.openlocfilehash: 5d3c8eeeb04398c29a25e90956888d9f79572e4f
ms.sourcegitcommit: 8d951fd7e9534054b160be48a1881ae0857561ef
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/13/2018
ms.locfileid: "53329391"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a>Conectar uma rede local ao Azure usando um Gateway de VPN

Essa arquitetura de referência mostra como estender uma rede local para o Azure, usando uma VPN (rede virtual privada) site a site. O tráfego flui entre a rede local e uma VNet (Rede Virtual) do Azure por meio de um túnel de VPN IPsec. [**Implantar esta solução**](#deploy-the-solution).

![Rede híbrida que abrange as infraestruturas locais e do Azure](./images/vpn.png)

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura

A arquitetura consiste nos componentes a seguir.

- **Rede local**. Uma rede de área local privada em execução dentro de uma organização.

- **Dispositivo de VPN**. Um dispositivo ou serviço que fornece conectividade externa com a rede local. O dispositivo de VPN pode ser um dispositivo de hardware ou uma solução de software, como o RRAS (Serviço de Roteamento e Acesso Remoto) do Windows Server 2012. Para obter uma lista de dispositivos de VPN com suporte e informações de como configurá-los para se conectar a um Gateway de VPN do Azure, consulte as instruções para o dispositivo selecionado no artigo [Sobre dispositivos VPN para conexões do Gateway de VPN site a site][vpn-appliance].

- **Rede virtual (VNet)**. O aplicativo de nuvem e os componentes do Gateway de VPN do Azure residem na mesma [VNet][azure-virtual-network].

- **Gateway de VPN do Azure**. O serviço do [Gateway de VPN][azure-vpn-gateway] permite conectar a VNet à rede local por meio de um dispositivo de VPN. Para obter mais informações, consulte [Conectar uma rede local a uma rede virtual do Microsoft Azure][connect-to-an-Azure-vnet]. O Gateway de VPN inclui os seguintes elementos:

  - **Gateway de rede virtual**. Um recurso que fornece um dispositivo de VPN virtual para a VNet. Ele é responsável por rotear o tráfego da rede local para a VNet.
  - **Gateway de rede local**. Uma abstração do dispositivo de VPN local. O tráfego de rede do aplicativo de nuvem para a rede local é roteado por esse gateway.
  - **Conexão**. A conexão tem propriedades que especificam o tipo de conexão (IPsec) e a chave compartilhada com o dispositivo de VPN local para criptografar o tráfego.
  - **Gateway de sub-rede**. O gateway de rede virtual é mantido em sua própria sub-rede, que está sujeita a vários requisitos, descritos na seção Recomendações abaixo.

- **Aplicativo de nuvem**. O aplicativo hospedado no Azure. Ele pode incluir várias camadas, com várias sub-redes conectadas por meio de balanceadores de carga do Azure. Para obter mais informações sobre a infraestrutura do aplicativo, consulte [Execução de cargas de trabalho de VM do Windows][windows-vm-ra] e [Execução de cargas de trabalho de VM do Linux][linux-vm-ra].

- **Balanceador Interno de carga**. O tráfego de rede do Gateway de VPN é roteado para o aplicativo de nuvem por meio de um balanceador de carga interno. O balanceador de carga está localizado na sub-rede de front-end do aplicativo.

## <a name="recommendations"></a>Recomendações

As seguintes recomendações aplicam-se à maioria dos cenários. Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.

### <a name="vnet-and-gateway-subnet"></a>VNet e sub-rede do gateway

Crie uma VNet do Azure com um espaço de endereço grande o suficiente para todos os recursos necessários. Verifique se o espaço de endereço da VNet tem espaço suficiente para aumentar caso haja necessidade de adicionar mais VMs no futuro. O espaço de endereço da VNet não deve se sobrepor à rede local. Por exemplo, o diagrama acima usa o espaço de endereço 10.20.0.0/16 para a VNet.

Crie uma sub-rede denominada *GatewaySubnet*, com um intervalo de endereços de /27. Essa sub-rede é necessária para o gateway de rede virtual. Alocar 32 endereços para essa sub-rede ajudará a evitar que as limitações de tamanho do gateway sejam atingidas no futuro. Além disso, evite colocar essa sub-rede no meio do espaço de endereço. Uma prática recomendada é definir o espaço de endereço para a sub-rede de gateway na extremidade superior do espaço de endereço da VNet. O exemplo mostrado no diagrama usa 10.20.255.224/27.  Aqui está um procedimento rápido para calcular o [CIDR]:

1. Defina os bits variáveis no espaço de endereço da VNet como 1, até os bits que estão sendo usados pela sub-rede do gateway e, em seguida, configure os bits restantes como 0.
2. Converta os bits resultantes em decimais e expresse-os como um espaço de endereço com o comprimento do prefixo definido para o tamanho da sub-rede do gateway.

Por exemplo, para uma VNet com um intervalo de endereços IP igual a 10.20.0.0/16, a aplicação da etapa nº 1 acima transforma-o em 10.20.0b11111111.0b11100000.  Convertê-lo em decimal e expressá-lo como um espaço de endereço produz 10.20.255.224/27.

> [!WARNING]
> Não implante nenhuma VM na sub-rede do gateway. Além disso, não atribua um NSG a essa sub-rede, pois faria com que o gateway parasse de funcionar.
>

### <a name="virtual-network-gateway"></a>Gateway de rede virtual

Aloque um endereço IP público para o gateway de rede virtual.

Crie o gateway de rede virtual na sub-rede do gateway e atribua-o ao endereço IP público recém-alocado. Use o tipo de gateway que melhor corresponde aos seus requisitos e que está habilitado pelo seu dispositivo de VPN:

- Crie um [gateway baseado em políticas][policy-based-routing] se você precisar controlar atentamente como as solicitações são roteadas com base em critérios de política, como prefixos de endereço. Os gateways baseados em políticas usam o roteamento estático e funcionam apenas com conexões site a site.

- Crie um [gateway baseado em rota][route-based-routing] se você se conecta à rede local usando o RRAS, dá suporte a conexões multisites ou entre regiões ou implementa conexões de VNet para VNet (incluindo rotas que atravessam várias VNets). Os gateways baseados em rota usam o roteamento dinâmico para o tráfego direto entre redes. Eles conseguem tolerar falhas no caminho da rede melhor do que as rotas estáticas, porque eles podem tentar rotas alternativas. Os gateways baseados em rota também conseguem reduzir a sobrecarga de gerenciamento porque é possível que as rotas não precisem ser atualizadas manualmente quando os endereços de rede são alterados.

Para obter uma lista de dispositivos de VPN com suporte, consulte [Sobre dispositivos VPN para conexões do Gateway de VPN site a site][vpn-appliances].

> [!NOTE]
> Depois que o gateway for criado, você não poderá mais alterar o tipo de gateway sem excluir e recriar o gateway.
>

Selecione a SKU do Gateway de VPN do Azure que melhor corresponde aos seus requisitos de produtividade. Para obter mais informações, confira [SKUs do gateway][azure-gateway-skus]

> [!NOTE]
> A SKU básica não é compatível com o Azure ExpressRoute. Você poderá [alterar a SKU][changing-SKUs] depois que o gateway for criado.
>

Você é cobrado com base no período de tempo em que o gateway é provisionado e fica disponível. Consulte [Preços do Gateway de VPN][azure-gateway-charges].

Crie regras de roteamento para a sub-rede de gateway que direcionem o tráfego do aplicativo de entrada do gateway para o balanceador de carga interno, em vez de permitirem que as solicitações passem diretamente para as VMs de aplicativo.

### <a name="on-premises-network-connection"></a>Conexão de rede local

Criar um gateway de rede local. Especifique o endereço IP público do dispositivo de VPN local e o espaço de endereço da rede local. Observe que o dispositivo de VPN local deve ter um endereço IP público que possa ser acessado pelo gateway de rede local no Gateway de VPN do Azure. O dispositivo VPN não pode estar localizado atrás de um dispositivo de NAT (conversão de endereços de rede).

Crie uma conexão site a site para o gateway de rede virtual e o gateway de rede local. Selecione o tipo de conexão site a site (IPsec) e especifique a chave compartilhada. A criptografia site a site com o Gateway de VPN do Azure é baseada no protocolo IPsec, usando chaves pré-compartilhadas para autenticação. Especifique a chave ao criar o Gateway de VPN do Azure. Você deve configurar o dispositivo de VPN em execução local com a mesma chave. Atualmente, não há suporte para outros mecanismos de autenticação.

Verifique se a infraestrutura de roteamento local está configurada para encaminhar as solicitações destinadas a endereços na VNet do Azure para o dispositivo VPN.

Abra todas as portas exigidas pelo aplicativo de nuvem na rede local.

Teste a conexão para verificar se:

- O dispositivo de VPN local roteia corretamente o tráfego para o aplicativo de nuvem por meio do Gateway de VPN do Azure.
- A VNet roteia corretamente o tráfego de volta para a rede local.
- O tráfego proibido em ambas as direções é bloqueado corretamente.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Você pode obter uma escalabilidade vertical limitada passando as SKUs do Gateway de VPN do plano Básico ou Standard para a SKU de VPN de alto desempenho.

Para VNets que esperam um grande volume de tráfego de VPN, considere distribuir as diferentes cargas de trabalho em VNets menores separadas e configurar um Gateway de VPN para cada uma delas.

Você pode particionar a VNet horizontal ou verticalmente. Para particionar horizontalmente, mova algumas instâncias de VM de cada camada para as sub-redes da nova VNet. O resultado é que cada VNet terá as mesmas estrutura e funcionalidade. Para particionar verticalmente, recrie cada camada para dividir a funcionalidade em áreas lógicas diferentes (como para tratamento de pedidos, faturamento, gerenciamento de contas de cliente e assim por diante). Assim, cada área funcional poderá ser colocada em sua própria VNet.

A replicação de um controlador de domínio do Active Directory local na VNet e a implementação de DNS na VNet podem ajudar a reduzir parte do tráfego relacionado à segurança e administrativo que flui do local para a nuvem. Para obter mais informações, consulte [Estendendo o AD DS (Active Directory Domain Services) para o Azure][adds-extend-domain].

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Se você precisa garantir que a rede local permaneça disponível para o Gateway de VPN do Azure, implemente um cluster de failover para o Gateway de VPN local.

Se sua organização tiver vários sites locais, crie [conexões multisites][vpn-gateway-multi-site] para uma ou mais VNets do Azure. Essa abordagem requer o roteamento dinâmico (baseado em rota), portanto, verifique se o Gateway de VPN local dá suporte a esse recurso.

Para obter detalhes sobre os contratos de nível de serviço, consulte [SLA para Gateway de VPN][sla-for-vpn-gateway].

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Monitore as informações de diagnóstico usando dispositivos de VPN locais. Esse processo depende dos recursos fornecidos pelo dispositivo de VPN. Por exemplo, se você estiver usando o roteamento e o serviço de acesso remoto no Windows Server 2012, o [log de RRAS][rras-logging].

Use [diagnóstico do Gateway de VPN do Azure][gateway-diagnostic-logs] para capturar informações sobre problemas de conectividade. Esses logs podem ser usados para rastrear informações como a origem e os destinos das solicitações de conexão, o protocolo usado e como a conexão foi estabelecida (ou por que a tentativa falhou).

Monitore os logs operacionais do Gateway de VPN do Azure usando os logs de auditoria disponíveis no portal do Azure. Logs separados estão disponíveis para o gateway de rede local, o gateway de rede do Azure e a conexão. Essas informações podem ser usadas para acompanhar as alterações feitas no gateway e poderão ser úteis se um gateway que estava em funcionamento parar de funcionar por alguma razão.

![Logs de auditoria no portal do Azure](../_images/guidance-hybrid-network-vpn/audit-logs.png)

Monitore a conectividade e acompanhe os eventos de falha de conectividade. Você pode usar um pacote de monitoramento, como o [Nagios][nagios] para capturar e relatar essas informações.

## <a name="security-considerations"></a>Considerações de segurança

Gere uma chave compartilhada diferente para cada Gateway de VPN. Use uma chave compartilhada forte para ajudar na resistência contra ataques de força bruta.

> [!NOTE]
> No momento, não é possível usar o Azure Key Vault para pré-compartilhar chaves do Gateway de VPN do Azure.
>

Verifique se o dispositivo de VPN local usa um método de criptografia [compatível com o Gateway de VPN do Azure][vpn-appliance-ipsec]. Para roteamento baseado em políticas, o Gateway de VPN do Azure dá suporte aos algoritmos de criptografia 3DES, AES256 e AES128. Os gateways baseados em rota dão suporte para AES256 e 3DES.

Se seu dispositivo de VPN local estiver em uma DMZ (rede de perímetro) com um firewall entre a rede de perímetro e a Internet, poderá ser necessário configurar [regras de firewall adicionais][additional-firewall-rules] para permitir a conexão de VPN site a site.

Se o aplicativo na VNet envia dados para a Internet, considere [implementar um túnel forçado][forced-tunneling] para rotear todo o tráfego associado à Internet por meio da rede local. Essa abordagem permite auditar as solicitações de saída feitas pelo aplicativo da infraestrutura local.

> [!NOTE]
> O túnel forçado pode afetar a conectividade para serviços do Azure (o serviço de armazenamento, por exemplo) e o Gerenciador de licenças do Windows.
>

## <a name="deploy-the-solution"></a>Implantar a solução

**Pré-requisitos**. Você deve ter uma infraestrutura local existente já configurada com um dispositivo de rede adequado.

Para implantar a solução, execute as etapas a seguir.

<!-- markdownlint-disable MD033 -->

1. Clique no botão abaixo:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Aguarde até o link abrir no Portal do Azure e siga estas etapas:
   - O nome do **Grupo de recursos** já está definido no arquivo de parâmetros, portanto, selecione **Criar novo** e digite `ra-hybrid-vpn-rg` na caixa de texto.
   - Selecione a região na caixa suspensa **Local**.
   - Não edite as caixas de texto **URI da raiz do modelo** ou **URI da raiz do parâmetro**.
   - Analise os termos e condições e clique na caixa de seleção **Concordo com os termos e condições declarados acima**.
   - Clique no botão **Comprar**.
3. Aguarde até que a implantação seja concluída.

<!-- markdownlint-enable MD033 -->

Para solucionar problemas de conexão, confira [Solucionar problemas de uma conexão VPN híbrida](./troubleshoot-vpn.md).

<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[azure-gateway-skus]: /azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: https://blogs.technet.microsoft.com/keithmayer/2016/10/12/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
