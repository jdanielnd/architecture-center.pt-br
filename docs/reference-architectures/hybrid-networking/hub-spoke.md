---
title: "Implementação de uma topologia de rede hub-spoke no Azure"
description: Como implementar uma topologia de rede hub-spoke no Azure.
author: telmosampaio
ms.date: 05/05/2017
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: e6f07a7962dd5728226b023700268340590d97a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a>Implementar uma topologia de rede hub-spoke no Azure

Essa arquitetura de referência mostra como implementar uma topologia hub-spoke no Azure. O *hub* é uma VNet (rede virtual) no Azure que atua como ponto central de conectividade para sua rede local. Os *spokes* são VNets que se emparelham com o hub e podem ser usadas para isolar as cargas de trabalho. Fluxos de tráfego entre o datacenter local e o hub por meio de uma conexão de ExpressRoute ou gateway de VPN.  [**Implantar esta solução**](#deploy-the-solution).

![[0]][0]

*Baixar um [arquivo Visio][visio-download] dessa arquitetura*


Os benefícios dessa topologia incluem:

* **Redução de custos** com a centralização de serviços que podem ser compartilhados por diversas cargas de trabalho, tais como NVAs (soluções de virtualização de rede) e servidores DNS, em um único local.
* **Superar os limites de assinaturas** emparelhando VNets de assinaturas diferentes com o hub central.
* **Separação de preocupações** entre TI central (SecOps, InfraOps) e cargas de trabalho (DevOps).

Alguns usos típicos para essa arquitetura:

* As cargas de trabalho implantadas em diferentes ambientes, como desenvolvimento, teste e produção, que exigem serviços compartilhados como DNS, IDS, NTP ou AD DS. Os serviços compartilhados são colocados na VNet do hub, enquanto cada ambiente é implantado em um spoke para manter o isolamento.
* Cargas de trabalho que não exigem conectividade entre si, mas precisam de acesso aos serviços compartilhados.
* Empresas que exigem o controle centralizado sobre aspectos de segurança, como um firewall no hub como uma DMZ e gerenciamento separado para cargas de trabalho em cada spoke.

## <a name="architecture"></a>Arquitetura

Essa arquitetura consiste nos seguintes componentes.

* **Rede local**. Uma rede de área local privada em execução dentro de uma organização.

* **Dispositivo VPN**. Um dispositivo ou serviço que fornece conectividade externa para a rede local. O dispositivo VPN pode ser um dispositivo de hardware ou uma solução de software, como o RRAS (Serviço de Acesso Remoto e Roteamento) do Windows Server 2012. Para obter uma lista de dispositivos de VPN com suporte e informações sobre como configurar dispositivos de VPN selecionados para se conectar ao Azure, consulte [Sobre dispositivos VPN para conexões de Gateway de VPN Site a Site][vpn-appliance].

* **Gateway de rede virtual de VPN ou gateway de ExpressRoute**. O gateway de rede virtual permite que a VNet se conecte ao dispositivo VPN ou ao circuito de ExpressRoute, usado para conectividade com a rede local. Para obter mais informações, consulte [Conectar uma rede local à rede virtual do Microsoft Azure][connect-to-an-Azure-vnet].

> [!NOTE]
> Os scripts de implantação dessa arquitetura de referência usam um gateway VPN para conectividade e uma VNet no Azure para simular a rede local.

* **VNet do hub**. VNet do Azure usada como o hub na topologia hub-spoke. O hub é o ponto central de conectividade para sua rede local e um local para hospedar os serviços que podem ser consumidos por diferentes cargas de trabalho hospedadas nas VNets do spoke.

* **Sub-rede do gateway**. Os gateways de rede virtual são mantidos na mesma sub-rede.

* **Sub-rede serviços compartilhados**. Uma sub-rede na VNet do hub usada para hospedar os serviços que podem ser compartilhados entre todos os spokes, como DNS ou AD DS.

* **VNets de spoke**. Uma ou mais VNets do Azure que são usadas como spokes na topologia hub-spoke. Spokes podem ser usados para isolar as cargas de trabalho em suas próprias VNets, gerenciadas separadamente de outros spokes. Cada carga de trabalho pode incluir várias camadas, com várias sub-redes conectadas por meio de balanceadores de carga do Azure. Para obter mais informações sobre a infraestrutura do aplicativo, consulte [Executar cargas de trabalho de VM no Windows][windows-vm-ra] e [Executar cargas de trabalho de VM no Linux][linux-vm-ra].

* **Emparelhamento VNet**. Duas VNets na mesma região do Azure podem ser conectadas usando uma [conexão de emparelhamento][vnet-peering]. Conexões de emparelhamento são conexões não transitivas de baixa latência entre VNets. Quando emparelhadas, as VNets trocam tráfego usando o backbone do Azure sem precisar de um roteador. Em uma topologia de rede hub-spoke, é necessário usar o emparelhamento VNet para conectar o hub a cada spoke.

> [!NOTE]
> Este artigo aborda apenas implantações do [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mas também é possível conectar uma VNet clássica a uma VNet do Resource Manager na mesma assinatura. Dessa forma, seus spokes podem hospedar implantações clássicas e ainda aproveitar os serviços compartilhados no hub.


## <a name="recommendations"></a>Recomendações

As seguintes recomendações aplicam-se à maioria dos cenários. Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.

### <a name="resource-groups"></a>Grupos de recursos

A VNet do hub e a VNet de cada spoke, podem ser implementadas em diferentes grupos de recursos, e até mesmo em assinaturas diferentes, contanto que elas pertençam ao mesmo locatário do Azure AD (Azure Active Directory) na mesma região do Azure. Isso possibilita o gerenciamento descentralizado de cada carga de trabalho, compartilhando os serviços mantidos na VNet do hub.

### <a name="vnet-and-gatewaysubnet"></a>VNet e GatewaySubnet

Crie uma sub-rede denominada *GatewaySubnet* com um intervalo de endereços de /27. Essa sub-rede é exigida pelo gateway de rede virtual. Alocar 32 endereços para esta sub-rede ajudará a evitar que as limitações de tamanho de gateway sejam atingidas no futuro.

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

Uma implantação para essa arquitetura está disponível no [GitHub][ref-arch-repo]. Ela usa VMs Ubuntu em cada VNet para testar a conectividade. Não há nenhum serviço hospedado na sub-rede **shared-services** na **VNet do hub**.

### <a name="prerequisites"></a>Pré-requisitos

Antes de implantar a arquitetura de referência para sua própria assinatura, você deve executar as etapas a seguir.

1. Clone, crie fork ou baixe o arquivo zip para as [arquiteturas de referência AzureCAT][ref-arch-repo] no repositório GitHub.

2. Se você preferir usar a CLI do Azure, verifique se a CLI do Azure 2.0 está instalada no computador. Para instalar a CLI, siga as instruções em [Instalar a CLI do Azure 2.0][azure-cli-2].

3. Se você preferir usar o PowerShell, verifique se você tem o módulo do PowerShell mais recente para Azure instalado no computador. Para instalar o módulo do Azure PowerShell mais recente, siga as instruções em [Instalar o PowerShell para Azure][azure-powershell].

4. Em um prompt de comando, bash prompt ou prompt do PowerShell, faça logon na sua conta do Azure usando um dos comandos abaixo e siga os prompts.

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>Implantar o datacenter local simulado

Para implantar o datacenter local simulado como uma VNet do Azure, execute as etapas a seguir.

1. Navegue até a pasta `hybrid-networking\hub-spoke\onprem` do repositório que você baixou na etapa de pré-requisitos acima.

2. Abra o arquivo `onprem.vm.parameters.json` e insira um nome de usuário e senha entre aspas nas linhas 11 e 12, conforme mostrado abaixo e salve o arquivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Execute o bash ou o comando do PowerShell abaixo para implantar o ambiente local simulado como uma VNet no Azure. Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > Se você decidir usar um nome do grupo de recursos distinto (diferente de `ra-onprem-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.

4. Aguarde até que a implantação seja concluída. Essa implantação cria uma rede virtual, uma máquina virtual que executa Ubuntu e um gateway de VPN. A criação do gateway de VPN pode levar mais de 40 minutos para ser concluída.

### <a name="azure-hub-vnet"></a>VNet do hub do Azure

Para implantar a VNet de hub e conectar-se à VNet local simulada criada acima, execute as seguintes etapas.

1. Navegue até a pasta `hybrid-networking\hub-spoke\hub` do repositório que você baixou na etapa de pré-requisitos acima.

2. Abra o arquivo `hub.vm.parameters.json` e insira um nome de usuário e senha entre aspas nas linhas 11 e 12, conforme mostrado abaixo e salve o arquivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Abra o arquivo `hub.gateway.parameters.json` e insira uma chave compartilhada entre as aspas na linha 23, conforme mostrado abaixo e salve o arquivo. Anote esse valor, pois você precisará usá-lo mais tarde na implantação.

  ```bash
  "sharedKey": "",
  ```

4. Execute o bash ou o comando do PowerShell abaixo para implantar o ambiente local simulado como uma VNet no Azure. Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > Se você decidir usar um nome do grupo de recursos distinto (diferente de `ra-hub-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.

5. Aguarde até que a implantação seja concluída. Essa implantação cria uma rede virtual, uma máquina virtual que executa Ubuntu, um gateway de VPN e uma conexão com o gateway criado na seção anterior. A criação do gateway de VPN pode levar mais de 40 minutos para ser concluída.

### <a name="connection-from-on-premises-to-the-hub"></a>Conexão do local para o hub

Para conectar-se do datacenter local simulado para a VNet do hub, execute as etapas a seguir.

1. Navegue até a pasta `hybrid-networking\hub-spoke\onprem` do repositório que você baixou na etapa de pré-requisitos acima.

2. Abra o arquivo `onprem.connection.parameters.json` e insira uma chave compartilhada entre as aspas na linha 9, conforme mostrado abaixo e salve o arquivo. Esse valor de chave compartilhada deve ser o mesmo usado no gateway local implantado anteriormente.

  ```bash
  "sharedKey": "",
  ```

3. Execute o bash ou o comando do PowerShell abaixo para implantar o ambiente local simulado como uma VNet no Azure. Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > Se você decidir usar um nome do grupo de recursos distinto (diferente de `ra-onprem-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.

4. Aguarde até que a implantação seja concluída. Essa implantação cria uma conexão entre a VNet usada para simular um datacenter local e a VNet do hub.

### <a name="azure-spoke-vnets"></a>VNets de spoke do Azure

Para implantar as VNets de spoke e conectar-se à VNet do hub criada acima, execute as seguintes etapas.

1. Alterne para a pasta `hybrid-networking\hub-spoke\spokes` do repositório que você baixou na etapa de pré-requisitos acima.

2. Abra o arquivo `spoke1.web.parameters.json` e insira um nome de usuário e senha entre aspas nas linhas 53 e 54, conforme mostrado abaixo e salve o arquivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Repita a etapa anterior para o arquivo `spoke2.web.parameters.json`.

4. Execute o bash ou o comando do PowerShell abaixo para implantar o primeiro spoke e conectá-lo ao hub. Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > Se você decidir usar um nome do grupo de recursos distinto (diferente de `ra-spoke1-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.

5. Aguarde até que a implantação seja concluída. Essa implantação cria uma rede virtual, um balanceador de carga, três máquinas virtuais que executam Ubuntu e Apache e uma conexão de emparelhamento VNet para a VNet de hub criada na seção anterior. A implantação pode levar mais de 20 minutos.

6. Execute o bash ou o comando do PowerShell abaixo para implantar o primeiro spoke e conectá-lo ao hub. Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > Se você decidir usar um nome do grupo de recursos distinto (diferente de `ra-spoke2-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.

5. Aguarde até que a implantação seja concluída. Essa implantação cria uma rede virtual, um balanceador de carga, três máquinas virtuais que executam Ubuntu e Apache e uma conexão de emparelhamento VNet para a VNet de hub criada na seção anterior. A implantação pode levar mais de 20 minutos.

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a>Emparelhamento VNet do hub do Azure para VNets de spoke

Para implantar as conexões de emparelhamento VNet para a VNet do hub, execute as etapas a seguir.

1. Alterne para a pasta `hybrid-networking\hub-spoke\hub` do repositório que você baixou na etapa de pré-requisitos acima.

2. Execute o bash ou o comando do PowerShell abaixo para implantar a conexão de emparelhamento para o primeiro spoke. Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. Execute o bash ou o comando do PowerShell abaixo para implantar a conexão de emparelhamento para o segundo spoke. Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a>Testar a conectividade

Para verificar se a topologia hub-spoke conectada a uma implantação de datacenter local funcionou, execute estas etapas.

1. No [Portal do Azure][portal], conecte-se à sua assinatura e navegue até a máquina virtual `ra-onprem-vm1` no grupo de recursos `ra-onprem-rg`.

2. No folha `Overview`, anote o `Public IP address` da VM.

3. Use um cliente SSH para se conectar ao endereço IP que você anotou acima usando o nome de usuário e senha especificados durante a implantação.

4. No prompt de comando na VM à qual você se conectou, execute o comando abaixo para testar a conectividade da VNet local para VNet Spoke1.

  ```bash
  ping 10.1.1.37
  ```

### <a name="add-connectivity-between-spokes"></a>Adicionar conectividade entre spokes

Se você quiser permitir que spokes se conectem entre si, será necessário implantar UDRs para cada spoke que encaminha o tráfego destinado a outros spokes para o gateway na VNet do hub. Execute as seguintes etapas para verificar se atualmente não é possível conectar-se de um spoke para outro e, em seguida, implante os UDRs e teste a conectividade novamente.

1. Repita as etapas 1 a 4 acima se você não estiver mais conectado à VM de jumpbox.

2. Conecte-se a um dos servidores Web no spoke 1.

  ```bash
  ssh 10.1.1.37
  ```

3. Teste a conectividade entre o spoke 1 e o spoke 2. Ela deve falhar.

  ```bash
  ping 10.1.2.37
  ```

4. Retorne para o prompt de comando do computador.

5. Alterne para a pasta `hybrid-networking\hub-spoke\spokes` do repositório que você baixou na etapa de pré-requisitos acima.

6. Execute o bash ou o comando do PowerShell abaixo para implantar um UDR no primeiro spoke. Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```

7. Execute o bash ou o comando do PowerShell abaixo para implantar um UDR no segundo spoke. Substitua os valores por sua assinatura, nome do grupo de recursos e região do Azure.

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```

8. Volte para o terminal de ssh.

9. Teste a conectividade entre o spoke 1 e o spoke 2. Ela deve ter êxito.

  ```bash
  ping 10.1.2.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
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
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Topologia hub-spoke no Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Topologia hub-spoke no Azure com o roteamento transitivo"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Topologia hub-spoke no Azure com o roteamento transitivo usando uma NVA"
[3]: ./images/hub-spokehub-spoke.svg "Topologia hub-spoke-hub-spoke no Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
