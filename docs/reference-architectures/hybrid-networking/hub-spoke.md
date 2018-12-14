---
title: Implementar uma topologia de rede hub-spoke no Azure
titleSuffix: Azure Reference Architectures
description: Implemente uma topologia de rede hub-spoke no Azure.
author: telmosampaio
ms.date: 10/08/2018
ms.custom: seodec18
ms.openlocfilehash: fe56630b621f02fe71b864642b75688ba1965862
ms.sourcegitcommit: 8d951fd7e9534054b160be48a1881ae0857561ef
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/13/2018
ms.locfileid: "53329425"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a>Implementar uma topologia de rede hub-spoke no Azure

Essa arquitetura de referência mostra como implementar uma topologia hub-spoke no Azure. O *hub* é uma VNet (rede virtual) no Azure que atua como ponto central de conectividade para sua rede local. Os *spokes* são VNets que se emparelham com o hub e podem ser usadas para isolar as cargas de trabalho. Fluxos de tráfego entre o datacenter local e o hub por meio de uma conexão de ExpressRoute ou gateway de VPN. [**Implantar esta solução**](#deploy-the-solution).

![[0]][0]

*Baixar um [arquivo Visio][visio-download] dessa arquitetura*

Os benefícios dessa topologia incluem:

- **Redução de custos** com a centralização de serviços que podem ser compartilhados por diversas cargas de trabalho, tais como NVAs (soluções de virtualização de rede) e servidores DNS, em um único local.
- **Superar os limites de assinaturas** emparelhando VNets de assinaturas diferentes com o hub central.
- **Separação de preocupações** entre TI central (SecOps, InfraOps) e cargas de trabalho (DevOps).

Alguns usos típicos dessa arquitetura:

- As cargas de trabalho implantadas em diferentes ambientes, como desenvolvimento, teste e produção, que exigem serviços compartilhados como DNS, IDS, NTP ou AD DS. Os serviços compartilhados são colocados na VNet do hub, enquanto cada ambiente é implantado em um spoke para manter o isolamento.
- Cargas de trabalho que não exigem conectividade entre si, mas precisam de acesso aos serviços compartilhados.
- Empresas que exigem o controle centralizado sobre aspectos de segurança, como um firewall no hub como uma DMZ e gerenciamento separado para cargas de trabalho em cada spoke.

## <a name="architecture"></a>Arquitetura

A arquitetura consiste nos componentes a seguir.

- **Rede local**. Uma rede de área local privada em execução dentro de uma organização.

- **Dispositivo VPN**. Um dispositivo ou serviço que fornece conectividade externa para a rede local. O dispositivo VPN pode ser um dispositivo de hardware ou uma solução de software, como o RRAS (Serviço de Acesso Remoto e Roteamento) do Windows Server 2012. Para obter uma lista de dispositivos de VPN com suporte e informações sobre como configurar dispositivos de VPN selecionados para se conectar ao Azure, consulte [Sobre dispositivos VPN para conexões de Gateway de VPN Site a Site][vpn-appliance].

- **Gateway de rede virtual de VPN ou gateway de ExpressRoute**. O gateway de rede virtual permite que a VNet se conecte ao dispositivo VPN ou ao circuito de ExpressRoute, usado para conectividade com a rede local. Para obter mais informações, consulte [Conectar uma rede local a uma rede virtual do Microsoft Azure][connect-to-an-Azure-vnet].

> [!NOTE]
> Os scripts de implantação dessa arquitetura de referência usam um gateway VPN para conectividade e uma VNet no Azure para simular a rede local.

- **VNet do hub**. VNet do Azure usada como o hub na topologia hub-spoke. O hub é o ponto central de conectividade para sua rede local e um local para hospedar os serviços que podem ser consumidos por diferentes cargas de trabalho hospedadas nas VNets do spoke.

- **Sub-rede do gateway**. Os gateways de rede virtual são mantidos na mesma sub-rede.

- **VNets de spoke**. Uma ou mais VNets do Azure que são usadas como spokes na topologia hub-spoke. Spokes podem ser usados para isolar as cargas de trabalho em suas próprias VNets, gerenciadas separadamente de outros spokes. Cada carga de trabalho pode incluir várias camadas, com várias sub-redes conectadas por meio de balanceadores de carga do Azure. Para obter mais informações sobre a infraestrutura do aplicativo, consulte [Execução de cargas de trabalho de VM do Windows][windows-vm-ra] e [Execução de cargas de trabalho de VM do Linux][linux-vm-ra].

- **Emparelhamento VNet**. Duas redes virtuais podem ser conectadas usando uma [conexão de emparelhamento][vnet-peering]. Conexões de emparelhamento são conexões não transitivas de baixa latência entre VNets. Quando emparelhadas, as VNets trocam tráfego usando o backbone do Azure sem precisar de um roteador. Em uma topologia de rede hub-spoke, é necessário usar o emparelhamento VNet para conectar o hub a cada spoke. Você pode parear redes virtuais na mesma região ou em regiões diferentes. Para saber mais, confira [Requisitos e restrições][vnet-peering-requirements].

> [!NOTE]
> Este artigo aborda apenas implantações do [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mas também é possível conectar uma VNet clássica a uma VNet do Resource Manager na mesma assinatura. Dessa forma, seus spokes podem hospedar implantações clássicas e ainda aproveitar os serviços compartilhados no hub.

## <a name="recommendations"></a>Recomendações

As seguintes recomendações aplicam-se à maioria dos cenários. Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.

### <a name="resource-groups"></a>Grupos de recursos

A VNet do hub e a VNet de cada spoke podem ser implementadas em diferentes grupos de recursos e até mesmo em diferentes assinaturas. Ao usar redes virtuais em diferentes assinaturas, ambas as assinaturas podem ser associadas ao mesmo locatário ou a um locatário diferente do Azure Active Directory. Isso possibilita o gerenciamento descentralizado de cada carga de trabalho, compartilhando os serviços mantidos na VNet do hub.

### <a name="vnet-and-gatewaysubnet"></a>VNet e GatewaySubnet

Crie uma sub-rede denominada *GatewaySubnet* com um intervalo de endereços de /27. Essa sub-rede é necessária para o gateway de rede virtual. Alocar 32 endereços para esta sub-rede ajudará a evitar que as limitações de tamanho de gateway sejam atingidas no futuro.

Para obter mais informações sobre como configurar o gateway, consulte as seguintes arquiteturas de referência, dependendo do seu tipo de conexão:

- [Rede híbrida usando o ExpressRoute][guidance-expressroute]
- [Rede híbrida usando um gateway de VPN][guidance-vpn]

Para maior disponibilidade, você pode usar o ExpressRoute e uma VPN para failover. Consulte [Conectar uma rede local ao Azure usando o ExpressRoute com failover de VPN][hybrid-ha].

Uma topologia hub-spoke também pode ser usada sem um gateway, se você não precisar de conectividade com a rede local.

### <a name="vnet-peering"></a>Emparelhamento VNet

O emparelhamento VNet é uma relação não transitiva entre duas VNets. Se você precisar que os spokes se conectem entre si, considere adicionar uma conexão de emparelhamento separada entre os spokes.

No entanto, se houver vários spokes que precisam se conectar entre si, você ficará sem conexões de emparelhamento possíveis rapidamente devido à [limitação do número de emparelhamentos de VNet por VNet][vnet-peering-limit]. Nesse cenário, considere usar UDRs (rotas definidas pelo usuário) para forçar o tráfego destinado a um spoke a ser enviado para uma NVA que atua como um roteador na VNet do hub. Isso permitirá que os spokes se conectem entre si.

Você também pode configurar os spokes para usar o gateway de VNet do hub para se comunicar com redes remotas. Para permitir que o tráfego de gateway flua do spoke para o hub e se conecte a redes remotas, é necessário fazer o seguinte:

- Configure a conexão de emparelhamento VNet no hub para **permitir trânsito de gateways**.
- Configure a conexão de emparelhamento VNet em cada spoke para **usar os gateways remotos**.
- Configure todas as conexões de emparelhamento VNet para **permitir tráfego encaminhado**.

## <a name="considerations"></a>Considerações

### <a name="spoke-connectivity"></a>Conectividade de spoke

Se você precisar de conectividade entre spokes, considere implementar uma NVA para roteamento no hub e usar UDRs no spoke para encaminhar o tráfego para o hub.

![[2]][2]

Nesse cenário, você precisa configurar todas as conexões de emparelhamento para **permitir tráfego encaminhado**.

### <a name="overcoming-vnet-peering-limits"></a>Superar os limites de emparelhamento VNet

Não deixe de considerar a [limitação do número de emparelhamentos de VNet por VNet][vnet-peering-limit] no Azure. Se você precisar de mais spokes do que o permitido pelo limite, considere criar uma topologia hub-spoke-hub-spoke, na qual o primeiro nível de spokes também funciona como hubs. O diagrama a seguir mostra essa abordagem.

![[3]][3]

Além disso, considere quais serviços compartilhados no hub, para garantir que o hub seja dimensionado para um número maior de spokes. Por exemplo, se seu hub fornecer serviços de firewall, considere os limites de largura de banda da sua solução de firewall ao adicionar vários spokes. Pode ser útil mover alguns desses serviços compartilhados para um segundo nível de hubs.

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para essa arquitetura está disponível no [GitHub][ref-arch-repo]. Ela usa VMs em cada VNet para testar a conectividade. Não há nenhum serviço hospedado na sub-rede **shared-services** na **VNet do hub**.

A implantação cria os seguintes grupos de recursos em sua assinatura:

- hub-nva-rg
- hub-vnet-rg
- onprem-jb-rg
- onprem-vnet-rg
- spoke1-vnet-rg
- spoke2-vent-rg

Os arquivos de parâmetros de modelo se referem a esses nomes e, portanto, se você alterá-los, atualize os arquivos de parâmetros para que correspondam.

### <a name="prerequisites"></a>Pré-requisitos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a>Implantar o datacenter local simulado

Para implantar o datacenter local simulado como uma rede virtual do Azure, siga as seguintes as etapas:

1. Navegue até a pasta `hybrid-networking/hub-spoke` do repositório de arquiteturas de referência.

2. Abra o arquivo `onprem.json` . Substitua os valores de `adminUsername` e `adminPassword`.

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. (Opcional) Para uma implantação do Linux, defina `osType` como `Linux`.

4. Execute o comando a seguir:

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. Aguarde até que a implantação seja concluída. Essa implantação cria uma rede virtual, uma máquina virtual e um gateway de VPN. Pode levar cerca de 40 minutos para criar o gateway de VPN.

### <a name="deploy-the-hub-vnet"></a>Implantar a VNET do hub

Para implantar a VNet do hub, execute as etapas a seguir.

1. Abra o arquivo `hub-vnet.json` . Substitua os valores de `adminUsername` e `adminPassword`.

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (Opcional) Para uma implantação do Linux, defina `osType` como `Linux`.

3. Localize as duas instâncias de `sharedKey` e insira uma chave compartilhada para a conexão VPN. Os valores devem ser correspondentes.

    ```json
    "sharedKey": "",
    ```

4. Execute o comando a seguir:

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. Aguarde até que a implantação seja concluída. Essa implantação cria uma rede virtual, uma máquina virtual, um gateway de VPN e uma conexão com o gateway.  Pode levar cerca de 40 minutos para criar o gateway de VPN.

### <a name="test-connectivity-with-the-hub"></a>Testar a conectividade com o hub

Testar a conexão do ambiente local simulado para a VNET do hub.

**Implantação no Windows**

1. Usar o portal do Azure para encontrar a VM denominada `jb-vm1` no grupo de recursos `onprem-jb-rg`.

2. Clique em `Connect` para abrir uma sessão de área de trabalho remota para a VM. Use a senha que você especificou no arquivo de parâmetro `onprem.json`.

3. Abra um console do PowerShell na VM e use o cmdlet `Test-NetConnection` para verificar se você pode se conectar à VM jumpbox na Vnet do hub.

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```

O resultado deve ser semelhante ao seguinte:

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> Por padrão, as máquinas virtuais do Windows Server não permitem respostas do ICMP no Azure. Se você quiser usar `ping` para testar a conectividade, você precisa habilitar o tráfego ICMP no Firewall Avançado do Windows para cada VM.

**Implantação no Linux**

1. Usar o portal do Azure para encontrar a VM denominada `jb-vm1` no grupo de recursos `onprem-jb-rg`.

2. Clique em `Connect` e copie o comando `ssh` mostrado no portal. 

3. Em um prompt do Linux, execute `ssh` para se conectar ao ambiente local simulado. Use a senha que você especificou no arquivo de parâmetro `onprem.json`.

4. Use o comando `ping` para testar a conectividade com a VM jumpbox na VNET do hub:

   ```shell
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a>Implantar as redes virtuais spoke

Para implantar as redes virtuais spoke, execute as etapas a seguir.

1. Abra o arquivo `spoke1.json` . Substitua os valores de `adminUsername` e `adminPassword`.

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (Opcional) Para uma implantação do Linux, defina `osType` como `Linux`.

3. Execute o comando a seguir:

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```

4. Repita as etapas 1 e 2 para o arquivo `spoke2.json`.

5. Execute o comando a seguir:

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. Execute o comando a seguir:

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a>Testar a conectividade

Testar a conexão do ambiente local simulado para as redes virtuais spoke.

**Implantação no Windows**

1. Usar o portal do Azure para encontrar a VM denominada `jb-vm1` no grupo de recursos `onprem-jb-rg`.

2. Clique em `Connect` para abrir uma sessão de área de trabalho remota para a VM. Use a senha que você especificou no arquivo de parâmetro `onprem.json`.

3. Abra um console do PowerShell na VM e use o cmdlet `Test-NetConnection` para verificar se você pode se conectar à VM jumpbox na rede virtual spoke.

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

**Implantação no Linux**

Para testar a conectividade do ambiente simulado no local para a redes virtuais do spoke usando as máquinas virtuais do Linux, execute as etapas a seguir:

1. Usar o portal do Azure para encontrar a VM denominada `jb-vm1` no grupo de recursos `onprem-jb-rg`.

2. Clique em `Connect` e copie o comando `ssh` mostrado no portal.

3. Em um prompt do Linux, execute `ssh` para se conectar ao ambiente local simulado. Use a senha que você especificou no arquivo de parâmetro `onprem.json`.

4. Use o comando `ping` para testar a conectividade com a VM jumpbox em cada spoke:

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a>Adicionar conectividade entre spokes

Esta etapa é opcional. Se você quiser permitir que spokes se conectem uns aos outros, precisará usar uma NVA (solução de virtualização de rede) como um roteador na VNET do hub e forçar o tráfego de spokes para o roteador ao tentar conectar outro spoke. Para implantar um NVA de amostra básico como uma única VM junto com as UDRs (rotas definidas pelo usuário) e permitir que as duas redes virtuais de spoke se conectem, execute as seguintes etapas:

1. Abra o arquivo `hub-nva.json` . Substitua os valores de `adminUsername` e `adminPassword`.

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. Execute o comando a seguir:

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vnet-peering-requirements]: /azure/virtual-network/virtual-network-manage-peering#requirements-and-constraints
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures

[0]: ./images/hub-spoke.png "Topologia hub-spoke no Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Topologia hub-spoke no Azure com o roteamento transitivo"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Topologia hub-spoke no Azure com o roteamento transitivo usando uma NVA"
[3]: ./images/hub-spokehub-spoke.svg "Topologia hub-spoke-hub-spoke no Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
