---
title: Solucionar problemas de uma conexão VPN híbrida
titleSuffix: Azure Reference Architectures
description: Solucionar uma conexão de gateway VPN entre uma rede local e o Azure.
author: telmosampaio
ms.date: 10/08/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: networking
ms.openlocfilehash: 49dfcf444665ef526e76cac0f4ad13104a142840
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485920"
---
# <a name="troubleshoot-a-hybrid-vpn-connection"></a>Solucionar problemas de uma conexão VPN híbrida

Este artigo fornece algumas dicas para solução de problemas de uma conexão de gateway VPN entre uma rede local e o Azure. Para obter informações gerais de como solucionar erros comuns relacionados à VPN, consulte [Troubleshooting common VPN related error][troubleshooting-vpn-errors] (Solucionando erros comuns relacionados à VPN).

## <a name="verify-the-vpn-appliance-is-functioning-correctly"></a>Verifique se o dispositivo de VPN está funcionando corretamente

As recomendações a seguir são úteis para determinar se o dispositivo de VPN local está funcionando corretamente.

**Verifique se há erros ou falhas em todos os arquivos de log gerados pelo dispositivo de VPN.** Isso ajudará a determinar se o dispositivo de VPN está funcionando corretamente. O local dessas informações variam de acordo com o dispositivo. Por exemplo, se você estiver usando o RRAS no Windows Server 2012, será possível usar o seguinte comando do PowerShell para exibir informações de evento de erro para o serviço RRAS:

```PowerShell
Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
```

A propriedade *Mensagem* de cada entrada fornece uma descrição do erro. Alguns exemplos comuns são:

- Incapacidade de se conectar, possivelmente devido a um endereço IP incorreto especificado para o gateway de VPN do Azure na configuração do adaptador de rede RRAS VPN.

  ```console
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

- A chave compartilhada incorreta especificada na configuração do adaptador de rede RRAS VPN.

  ```console
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

```powershell
Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
```

Em caso de falha de conexão, esse log conterá erros semelhantes ao seguinte:

```console
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

## <a name="verify-connectivity"></a>Verificar conectividade

**Verifique a conectividade e o roteamento no Gateway de VPN.** O dispositivo de VPN pode não estar roteando o tráfego corretamento pelo Gateway de VPN do Azure. Use uma ferramenta como o [PsPing][psping] para verificar a conectividade e o roteamento no Gateway de VPN. Por exemplo, para testar a conectividade de um computador local com um servidor Web localizado na VNet, execute o seguinte comando (substituindo `<<web-server-address>>` pelo endereço do servidor Web):

```console
PsPing -t <<web-server-address>>:80
```

Se o computador local puder rotear o tráfego para o servidor Web, será exibida uma saída semelhante à seguinte:

```console
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

```console
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

**Verifique se o firewall local permite a passagem do tráfego da VPN e se as portas corretas estão abertas.**

**Verifique se o dispositivo de VPN local usa um método de criptografia compatível com o Gateway de VPN do Azure.** Para roteamento baseado em políticas, o Gateway de VPN do Azure dá suporte aos algoritmos de criptografia 3DES, AES256 e AES128. Os gateways baseados em rota dão suporte para AES256 e 3DES. Para saber mais, consulte [Sobre dispositivos VPN e os parâmetros IPsec/IKE para conexões do Gateway de VPN Site a Site][vpn-appliance].

## <a name="check-for-problems-with-the-azure-vpn-gateway"></a>Verifique se há problemas com o gateway de VPN do Azure

As recomendações a seguir são úteis para determinar se há algum problema com o Gateway de VPN do Azure:

**Examine os logs de diagnóstico do gateway de VPN do Azure para encontrar possíveis problemas.** Confira [Passo a passo: Capturar logs de diagnóstico do gateway de rede virtual do Azure Resource Manager][gateway-diagnostic-logs].

**Verifique se o dispositivo de VPN local e o Gateway de VPN do Azure estão configurados com a mesma chave de autenticação compartilhada.** Você pode exibir a chave compartilhada armazenada pelo Gateway de VPN do Azure usando o seguinte comando da CLI do Azure:

```azurecli
azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
```

Use o comando apropriado para seu dispositivo de VPN local para mostrar a chave compartilhada configurada para esse dispositivo.

Verifique se a sub-rede *GatewaySubnet* que mantém o Gateway de VPN do Azure não está associada a um NSG.

Você pode exibir os detalhes da sub-rede usando o seguinte comando da CLI do Azure:

```azurecli
azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
```

Verifique se não há nenhum campo de dados denominado *ID do Grupo de Segurança de Rede*. O exemplo a seguir mostra os resultados de uma instância da *GatewaySubnet* que tem um NSG atribuído (*VPN-Gateway-Group*). Isso poderá impedir que o gateway funcione corretamente se houver regras definidas para este NSG.

```console
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

**Verifique se as máquinas virtuais na VNet do Azure estão configuradas para permitir o tráfego proveniente de fora da VNet.** Verifique as regras do NSG associadas às sub-redes que contêm essas máquinas virtuais. Você pode exibir todas as regras do NSG usando o seguinte comando da CLI do Azure:

```azurecli
azure network nsg show -g <<resource-group>> -n <<nsg-name>>
```

**Verifique se o Gateway de VPN do Azure está conectado.** Você pode usar o seguinte comando do Azure PowerShell para verificar o status atual da conexão de VPN do Azure. O parâmetro `<<connection-name>>` é o nome da conexão de VPN do Azure que vincula o gateway de rede virtual e o gateway local.

```powershell
Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
```

Os snippets de código a seguir realçam a saída gerada quando o gateway está conectado (o primeiro exemplo) e desconectado (o segundo exemplo):

```powershell
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

```powershell
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

## <a name="miscellaneous-issues"></a>Problemas diversos

As recomendações a seguir são úteis para determinar se há algum problema com a configuração da VM host, a utilização da largura de banda da rede ou o desempenho do aplicativo:

**Verificar a configuração do firewall.** Verifique se o firewall no sistema operacional convidado em execução nas VMs do Azure na sub-rede está configurado corretamente para permitir o tráfego permitido dos intervalos de IP locais.

**Verifique se o volume de tráfego não está próximo ao limite da largura de banda disponível para o Gateway de VPN do Azure.** A maneira de verificar isso depende do dispositivo de VPN em execução local. Por exemplo, se você estiver usando o RRAS no Windows Server 2012, será possível usar o Monitor de Desempenho para acompanhar o volume de dados recebido e transmitido pela conexão de VPN. Usando o objeto *RAS Total*, selecione os contadores *Bytes recebidos/s* e *Bytes transmitidos/s*:

![Contadores de desempenho para monitorar o tráfego de rede da VPN](../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png)

Compare os resultados com a largura de banda disponível para o Gateway de VPN (de 100 Mbps para SKUs do plano Básico e Standard a 1,25 Gbps para SKUs do VpnGw3):

![Gráfico de desempenho de rede da VPN de exemplo](../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png)

**Verifique se você implantou o número de VMs e os tamanhos corretos para sua carga de aplicativo.** Determine se alguma das máquinas virtuais na VNet do Azure está com a execução lenta. Se sim, elas podem estar sobrecarregadas, pode haver um número muito pequeno delas para lidar com a carga ou os balanceadores de carga podem não estar configurados corretamente. Para determinar isso, [capture e analise as informações de diagnóstico][azure-vm-diagnostics]. Você pode examinar os resultados usando o portal do Azure, mas muitas ferramentas de terceiros também estão disponíveis para fornecer informações detalhadas sobre os dados de desempenho.

**Verifique se que o aplicativo está fazendo um uso eficiente dos recursos de nuvem.** Instrumente o código do aplicativo em execução em cada VM para determinar se os aplicativos estão fazendo o melhor uso possível dos recursos. Você pode usar ferramentas como [Application Insights][application-insights].

<!-- links -->

[application-insights]: /azure/application-insights/app-insights-overview-usage
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[gateway-diagnostic-logs]: https://blogs.technet.microsoft.com/keithmayer/2016/10/12/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/
[psping]: https://technet.microsoft.com/sysinternals/jj729731.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
