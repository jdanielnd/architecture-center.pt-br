---
title: Conectar uma rede local ao Azure usando VPN
description: Como implementar uma arquitetura de rede site a site segura que abranja uma rede virtual do Azure e uma rede local conectada usando uma VPN.
author: RohitSharma-pnp
ms.date: 10/22/2018
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: 6e705c40663eff421e79067f916a1ebad6e72822
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916542"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a>Conectar uma rede local ao Azure usando um Gateway de VPN

Essa arquitetura de referência mostra como estender uma rede local para o Azure, usando uma VPN (rede virtual privada) site a site. O tráfego flui entre a rede local e uma VNet (Rede Virtual) do Azure por meio de um túnel de VPN IPsec. [**Implante essa solução**.](#deploy-the-solution)

![[0]][0]

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura 

A arquitetura consiste nos componentes a seguir.

* **Rede local**. Uma rede de área local privada em execução dentro de uma organização.

* **Dispositivo de VPN**. Um dispositivo ou serviço que fornece conectividade externa com a rede local. O dispositivo de VPN pode ser um dispositivo de hardware ou uma solução de software, como o RRAS (Serviço de Roteamento e Acesso Remoto) do Windows Server 2012. Para obter uma lista de dispositivos de VPN com suporte e informações de como configurá-los para se conectar a um Gateway de VPN do Azure, consulte as instruções para o dispositivo selecionado no artigo [Sobre dispositivos VPN para conexões do Gateway de VPN site a site][vpn-appliance].

* **Rede virtual (VNet)**. O aplicativo de nuvem e os componentes do Gateway de VPN do Azure residem na mesma [VNet][azure-virtual-network].

* **Gateway de VPN do Azure**. O serviço do [Gateway de VPN][azure-vpn-gateway] permite conectar a VNet à rede local por meio de um dispositivo de VPN. Para obter mais informações, consulte [Conectar uma rede local a uma rede virtual do Microsoft Azure][connect-to-an-Azure-vnet]. O Gateway de VPN inclui os seguintes elementos:
  
  * **Gateway de rede virtual**. Um recurso que fornece um dispositivo de VPN virtual para a VNet. Ele é responsável por rotear o tráfego da rede local para a VNet.
  * **Gateway de rede local**. Uma abstração do dispositivo de VPN local. O tráfego de rede do aplicativo de nuvem para a rede local é roteado por esse gateway.
  * **Conexão**. A conexão tem propriedades que especificam o tipo de conexão (IPsec) e a chave compartilhada com o dispositivo de VPN local para criptografar o tráfego.
  * **Gateway de sub-rede**. O gateway de rede virtual é mantido em sua própria sub-rede, que está sujeita a vários requisitos, descritos na seção Recomendações abaixo.

* **Aplicativo de nuvem**. O aplicativo hospedado no Azure. Ele pode incluir várias camadas, com várias sub-redes conectadas por meio de balanceadores de carga do Azure. Para obter mais informações sobre a infraestrutura do aplicativo, consulte [Execução de cargas de trabalho de VM do Windows][windows-vm-ra] e [Execução de cargas de trabalho de VM do Linux][linux-vm-ra].

* **Balanceador Interno de carga**. O tráfego de rede do Gateway de VPN é roteado para o aplicativo de nuvem por meio de um balanceador de carga interno. O balanceador de carga está localizado na sub-rede de front-end do aplicativo.

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
> 

Selecione a SKU do Gateway de VPN do Azure que melhor corresponde aos seus requisitos de produtividade. Para obter mais informações, confira [SKUs do gateway][azure-gateway-skus]

> [!NOTE]
> A SKU básica não é compatível com o Azure ExpressRoute. Você poderá [alterar a SKU][changing-SKUs] depois que o gateway for criado.
> 
> 

Você é cobrado com base no período de tempo em que o gateway é provisionado e fica disponível. Consulte [Preços do Gateway de VPN][azure-gateway-charges].

Crie regras de roteamento para a sub-rede de gateway que direcionem o tráfego do aplicativo de entrada do gateway para o balanceador de carga interno, em vez de permitirem que as solicitações passem diretamente para as VMs de aplicativo.

### <a name="on-premises-network-connection"></a>Conexão de rede local

Criar um gateway de rede local. Especifique o endereço IP público do dispositivo de VPN local e o espaço de endereço da rede local. Observe que o dispositivo de VPN local deve ter um endereço IP público que possa ser acessado pelo gateway de rede local no Gateway de VPN do Azure. O dispositivo VPN não pode estar localizado atrás de um dispositivo de NAT (conversão de endereços de rede).

Crie uma conexão site a site para o gateway de rede virtual e o gateway de rede local. Selecione o tipo de conexão site a site (IPsec) e especifique a chave compartilhada. A criptografia site a site com o Gateway de VPN do Azure é baseada no protocolo IPsec, usando chaves pré-compartilhadas para autenticação. Especifique a chave ao criar o Gateway de VPN do Azure. Você deve configurar o dispositivo de VPN em execução local com a mesma chave. Atualmente, não há suporte para outros mecanismos de autenticação.

Verifique se a infraestrutura de roteamento local está configurada para encaminhar as solicitações destinadas a endereços na VNet do Azure para o dispositivo VPN.

Abra todas as portas exigidas pelo aplicativo de nuvem na rede local.

Teste a conexão para verificar se:

* O dispositivo de VPN local roteia corretamente o tráfego para o aplicativo de nuvem por meio do Gateway de VPN do Azure.
* A VNet roteia corretamente o tráfego de volta para a rede local.
* O tráfego proibido em ambas as direções é bloqueado corretamente.

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

![[2]][2]

Monitore a conectividade e acompanhe os eventos de falha de conectividade. Você pode usar um pacote de monitoramento, como o [Nagios][nagios] para capturar e relatar essas informações.

## <a name="security-considerations"></a>Considerações de segurança

Gere uma chave compartilhada diferente para cada Gateway de VPN. Use uma chave compartilhada forte para ajudar na resistência contra ataques de força bruta.

> [!NOTE]
> No momento, não é possível usar o Azure Key Vault para pré-compartilhar chaves do Gateway de VPN do Azure.
> 
> 

Verifique se o dispositivo de VPN local usa um método de criptografia [compatível com o Gateway de VPN do Azure][vpn-appliance-ipsec]. Para roteamento baseado em políticas, o Gateway de VPN do Azure dá suporte aos algoritmos de criptografia 3DES, AES256 e AES128. Os gateways baseados em rota dão suporte para AES256 e 3DES.

Se seu dispositivo de VPN local estiver em uma DMZ (rede de perímetro) com um firewall entre a rede de perímetro e a Internet, poderá ser necessário configurar [regras de firewall adicionais][additional-firewall-rules] para permitir a conexão de VPN site a site.

Se o aplicativo na VNet envia dados para a Internet, considere [implementar um túnel forçado][forced-tunneling] para rotear todo o tráfego associado à Internet por meio da rede local. Essa abordagem permite auditar as solicitações de saída feitas pelo aplicativo da infraestrutura local.

> [!NOTE]
> O túnel forçado pode afetar a conectividade para serviços do Azure (o serviço de armazenamento, por exemplo) e o Gerenciador de licenças do Windows.
> 
> 


## <a name="troubleshooting"></a>solução de problemas 

Para obter informações gerais de como solucionar erros comuns relacionados à VPN, consulte [Troubleshooting common VPN related error][troubleshooting-vpn-errors] (Solucionando erros comuns relacionados à VPN).

As recomendações a seguir são úteis para determinar se o dispositivo de VPN local está funcionando corretamente.

- **Verifique se há erros ou falhas em todos os arquivos de log gerados pelo dispositivo de VPN.**

    Isso ajudará a determinar se o dispositivo de VPN está funcionando corretamente. O local dessas informações variam de acordo com o dispositivo. Por exemplo, se você estiver usando o RRAS no Windows Server 2012, será possível usar o seguinte comando do PowerShell para exibir informações de evento de erro para o serviço RRAS:

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    A propriedade *Mensagem* de cada entrada fornece uma descrição do erro. Alguns exemplos comuns são:

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

    Você também pode obter informações de log de eventos sobre tentativas de conexão por meio do serviço RRAS usando o seguinte comando do PowerShell: 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    Em caso de falha de conexão, esse log conterá erros semelhantes ao seguinte:

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

- **Verifique a conectividade e o roteamento no Gateway de VPN.**

    O dispositivo de VPN pode não estar roteando o tráfego corretamento pelo Gateway de VPN do Azure. Use uma ferramenta como o [PsPing][psping] para verificar a conectividade e o roteamento no Gateway de VPN. Por exemplo, para testar a conectividade de um computador local com um servidor Web localizado na VNet, execute o seguinte comando (substituindo `<<web-server-address>>` pelo endereço do servidor Web):

    ```
    PsPing -t <<web-server-address>>:80
    ```

    Se o computador local puder rotear o tráfego para o servidor Web, será exibida uma saída semelhante à seguinte:

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

    Se o computador local não puder se comunicar com o destino especificado, serão exibidas mensagens assim:

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

- **Verifique se o firewall local permite a passagem do tráfego da VPN e se as portas corretas estão abertas.**

- **Verifique se o dispositivo de VPN local usa um método de criptografia [compatível com o Gateway de VPN do Azure][vpn-appliance].** Para roteamento baseado em políticas, o Gateway de VPN do Azure dá suporte aos algoritmos de criptografia 3DES, AES256 e AES128. Os gateways baseados em rota dão suporte para AES256 e 3DES.

As recomendações a seguir são úteis para determinar se há algum problema com o Gateway de VPN do Azure:

- **Examine os [logs de diagnóstico do Gateway de VPN do Azure][gateway-diagnostic-logs] para buscar possíveis problemas.**

- **Verifique se o dispositivo de VPN local e o Gateway de VPN do Azure estão configurados com a mesma chave de autenticação compartilhada.**

    Você pode exibir a chave compartilhada armazenada pelo Gateway de VPN do Azure usando o seguinte comando da CLI do Azure:

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    Use o comando apropriado para seu dispositivo de VPN local para mostrar a chave compartilhada configurada para esse dispositivo.

    Verifique se a sub-rede *GatewaySubnet* que mantém o Gateway de VPN do Azure não está associada a um NSG.

    Você pode exibir os detalhes da sub-rede usando o seguinte comando da CLI do Azure:

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    Verifique se não há nenhum campo de dados denominado *ID do Grupo de Segurança de Rede*. O exemplo a seguir mostra os resultados de uma instância da *GatewaySubnet* que tem um NSG atribuído (*VPN-Gateway-Group*). Isso poderá impedir que o gateway funcione corretamente se houver regras definidas para este NSG.

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

- **Verifique se as máquinas virtuais na VNet do Azure estão configuradas para permitir o tráfego proveniente de fora da VNet.**

    Verifique as regras do NSG associadas às sub-redes que contêm essas máquinas virtuais. Você pode exibir todas as regras do NSG usando o seguinte comando da CLI do Azure:

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- **Verifique se o Gateway de VPN do Azure está conectado.**

    Você pode usar o seguinte comando do Azure PowerShell para verificar o status atual da conexão de VPN do Azure. O parâmetro `<<connection-name>>` é o nome da conexão de VPN do Azure que vincula o gateway de rede virtual e o gateway local.

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    Os snippets de código a seguir realçam a saída gerada quando o gateway está conectado (o primeiro exemplo) e desconectado (o segundo exemplo):

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

As recomendações a seguir são úteis para determinar se há algum problema com a configuração da VM host, a utilização da largura de banda da rede ou o desempenho do aplicativo:

- **Verifique se o firewall no sistema operacional convidado em execução nas VMs do Azure na sub-rede está configurado corretamente para permitir o tráfego permitido dos intervalos de IP locais.**

- **Verifique se o volume de tráfego não está próximo ao limite da largura de banda disponível para o Gateway de VPN do Azure.**

    A maneira de verificar isso depende do dispositivo de VPN em execução local. Por exemplo, se você estiver usando o RRAS no Windows Server 2012, será possível usar o Monitor de Desempenho para acompanhar o volume de dados recebido e transmitido pela conexão de VPN. Usando o objeto *RAS Total*, selecione os contadores *Bytes recebidos/s* e *Bytes transmitidos/s*:

    ![[3]][3]

    Compare os resultados com a largura de banda disponível para o Gateway de VPN (100 Mbps para SKUs do plano Básico e Standard e 200 Mbps para a SKU de alto desempenho):

    ![[4]][4]

- **Verifique se você implantou o número de VMs e os tamanhos corretos para sua carga de aplicativo.**

    Determine se alguma das máquinas virtuais na VNet do Azure está com a execução lenta. Se sim, elas podem estar sobrecarregadas, pode haver um número muito pequeno delas para lidar com a carga ou os balanceadores de carga podem não estar configurados corretamente. Para determinar isso, [capture e analise as informações de diagnóstico][azure-vm-diagnostics]. Você pode examinar os resultados usando o portal do Azure, mas muitas ferramentas de terceiros também estão disponíveis para fornecer informações detalhadas sobre os dados de desempenho.

- **Verifique se que o aplicativo está fazendo um uso eficiente dos recursos de nuvem.**

    Instrumente o código do aplicativo em execução em cada VM para determinar se os aplicativos estão fazendo o melhor uso possível dos recursos. Você pode usar ferramentas como [Application Insights][application-insights].

## <a name="deploy-the-solution"></a>Implantar a solução


**Pré-requisitos.** Você deve ter uma infraestrutura local existente já configurada com um dispositivo de rede adequado.

Para implantar a solução, execute as etapas a seguir.

1. Clique no botão abaixo:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Aguarde até o link abrir no Portal do Azure e siga estas etapas: 
   * O nome do **Grupo de recursos** já está definido no arquivo de parâmetros, portanto, selecione **Criar novo** e digite `ra-hybrid-vpn-rg` na caixa de texto.
   * Selecione a região na caixa suspensa **Local**.
   * Não edite as caixas de texto **URI da raiz do modelo** ou **URI da raiz do parâmetro**.
   * Analise os termos e condições e clique na caixa de seleção **Concordo com os termos e condições declarados acima**.
   * Clique no botão **Comprar**.
3. Aguarde até que a implantação seja concluída.



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
[azure-vpn-gateway-diagnostics]: https://blogs.technet.microsoft.com/keithmayer/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell/
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: https://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: https://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: https://blogs.technet.microsoft.com/keithmayer/2016/10/12/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/
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
