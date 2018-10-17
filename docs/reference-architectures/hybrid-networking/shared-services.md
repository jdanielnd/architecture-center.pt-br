---
title: Implementação de uma topologia de rede hub-spoke com serviços compartilhados no Azure
description: Como implementar uma topologia de rede hub-spoke com serviços compartilhados no Azure.
author: telmosampaio
ms.date: 06/19/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: 0238c5d6f28bacbc32268d4586b30395de36384b
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876861"
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a>Implementar uma topologia de rede hub-spoke com serviços compartilhados no Azure

Essa arquitetura de referência se baseia na arquitetura de referência do [hub-spoke] [ guidance-hub-spoke] para incluir serviços compartilhados no hub que pode ser consumido por todos os spokes. Como uma primeira etapa para migrar um datacenter para a nuvem, e criando um [datacenter virtual], os primeiros serviços que você precisa compartilhar são identidade e segurança. Esta arquitetura de referência mostra como estender seus serviços do Active Directory de seu datacenter local para o do Azure e como adicionar uma NVA (solução de virtualização de rede), que pode agir como um firewall, em uma topologia hub-spoke.  [**Implantar esta solução**](#deploy-the-solution).

![[0]][0]

*Baixar um [arquivo Visio][visio-download] dessa arquitetura*

Os benefícios dessa topologia incluem:

* **Redução de custos** com a centralização de serviços que podem ser compartilhados por diversas cargas de trabalho, tais como NVAs (soluções de virtualização de rede) e servidores DNS, em um único local.
* **Superar os limites de assinaturas** emparelhando VNets de assinaturas diferentes com o hub central.
* **Separação de preocupações** entre TI central (SecOps, InfraOps) e cargas de trabalho (DevOps).

Alguns usos típicos dessa arquitetura:

* As cargas de trabalho implantadas em diferentes ambientes, como desenvolvimento, teste e produção, que exigem serviços compartilhados como DNS, IDS, NTP ou AD DS. Os serviços compartilhados são colocados na VNet do hub, enquanto cada ambiente é implantado em um spoke para manter o isolamento.
* Cargas de trabalho que não exigem conectividade entre si, mas precisam de acesso aos serviços compartilhados.
* Empresas que exigem o controle centralizado sobre aspectos de segurança, como um firewall no hub como uma DMZ e gerenciamento separado para cargas de trabalho em cada spoke.

## <a name="architecture"></a>Arquitetura

A arquitetura consiste nos componentes a seguir.

* **Rede local**. Uma rede de área local privada em execução dentro de uma organização.

* **Dispositivo VPN**. Um dispositivo ou serviço que fornece conectividade externa para a rede local. O dispositivo VPN pode ser um dispositivo de hardware ou uma solução de software, como o RRAS (Serviço de Acesso Remoto e Roteamento) do Windows Server 2012. Para obter uma lista de dispositivos de VPN com suporte e informações sobre como configurar dispositivos de VPN selecionados para se conectar ao Azure, consulte [Sobre dispositivos VPN para conexões de Gateway de VPN Site a Site][vpn-appliance].

* **Gateway de rede virtual de VPN ou gateway de ExpressRoute**. O gateway de rede virtual permite que a VNet se conecte ao dispositivo VPN ou ao circuito de ExpressRoute, usado para conectividade com a rede local. Para obter mais informações, consulte [Conectar uma rede local a uma rede virtual do Microsoft Azure][connect-to-an-Azure-vnet].

> [!NOTE]
> Os scripts de implantação dessa arquitetura de referência usam um gateway VPN para conectividade e uma VNet no Azure para simular a rede local.

* **VNet do hub**. VNet do Azure usada como o hub na topologia hub-spoke. O hub é o ponto central de conectividade para sua rede local e um local para hospedar os serviços que podem ser consumidos por diferentes cargas de trabalho hospedadas nas VNets do spoke.

* **Sub-rede do gateway**. Os gateways de rede virtual são mantidos na mesma sub-rede.

* **Sub-rede serviços compartilhados**. Uma sub-rede na VNet do hub usada para hospedar os serviços que podem ser compartilhados entre todos os spokes, como DNS ou AD DS.

* **Sub-rede de perímetro**. Uma sub-rede na rede virtual do hub usada para hospedar NVAs que podem agir como dispositivos de segurança, como firewalls.

* **VNets de spoke**. Uma ou mais VNets do Azure que são usadas como spokes na topologia hub-spoke. Spokes podem ser usados para isolar as cargas de trabalho em suas próprias VNets, gerenciadas separadamente de outros spokes. Cada carga de trabalho pode incluir várias camadas, com várias sub-redes conectadas por meio de balanceadores de carga do Azure. Para obter mais informações sobre a infraestrutura do aplicativo, consulte [Execução de cargas de trabalho de VM do Windows][windows-vm-ra] e [Execução de cargas de trabalho de VM do Linux][linux-vm-ra].

* **Emparelhamento VNet**. Duas VNets na mesma região do Azure podem ser conectadas usando uma [conexão de emparelhamento][vnet-peering]. Conexões de emparelhamento são conexões não transitivas de baixa latência entre VNets. Quando emparelhadas, as VNets trocam tráfego usando o backbone do Azure sem precisar de um roteador. Em uma topologia de rede hub-spoke, é necessário usar o emparelhamento VNet para conectar o hub a cada spoke.

> [!NOTE]
> Este artigo aborda apenas implantações do [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mas também é possível conectar uma VNet clássica a uma VNet do Resource Manager na mesma assinatura. Dessa forma, seus spokes podem hospedar implantações clássicas e ainda aproveitar os serviços compartilhados no hub.

## <a name="recommendations"></a>Recomendações

Todas as recomendações para a arquitetura de referência do [hub-spoke] [ guidance-hub-spoke] também se aplicam à arquitetura de referência de serviços compartilhados. 

Além disso, as seguintes recomendações se aplicam para a maioria dos cenários em serviços compartilhados. Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.

### <a name="identity"></a>Identidade

A maioria das empresas tem um ambiente de Active Directory Domain Services (ADDS) em seu datacenter local. Para facilitar o gerenciamento de ativos movidos para o Azure de sua rede local que dependem do ADDS, é recomendável hospedar os controladores de domínio ADDS no Azure.

Se você usar objetos da Política de Grupo que você deseja controlar separadamente para o Azure e seu ambiente local, use um site diferente do AD para cada região do Azure. Coloque os controladores de domínio em uma rede virtual central (hub) que pode ser acessada por cargas de trabalho dependentes.

### <a name="security"></a>Segurança

Ao mover cargas de trabalho do seu ambiente local para o Azure, algumas dessas cargas de trabalho exigem ser hospedadas em máquinas virtuais. Por motivos de conformidade, talvez seja necessário impor restrições sobre o tráfego que passa por essas cargas de trabalho. 

Você pode usar soluções de virtualização de rede (NVAs) no Azure para hospedar tipos diferentes de segurança e de serviços de desempenho. Se você estiver atualmente familiarizado com um determinado conjunto de dispositivos locais, é recomendável usar os mesmos dispositivos virtualizados no Azure, onde aplicável.

> [!NOTE]
> Os scripts de implantação para esta arquitetura de referência usam uma VM Ubuntu com encaminhamento de IP ativado para simular uma solução de virtualização de rede.

## <a name="considerations"></a>Considerações

### <a name="overcoming-vnet-peering-limits"></a>Superar os limites de emparelhamento VNet

Não deixe de considerar a [limitação do número de emparelhamentos de VNet por VNet][vnet-peering-limit] no Azure. Se você precisar de mais spokes do que o permitido pelo limite, considere criar uma topologia hub-spoke-hub-spoke, na qual o primeiro nível de spokes também funciona como hubs. O diagrama a seguir mostra essa abordagem.

![[3]][3]

Além disso, considere quais serviços compartilhados no hub, para garantir que o hub seja dimensionado para um número maior de spokes. Por exemplo, se seu hub fornecer serviços de firewall, considere os limites de largura de banda da sua solução de firewall ao adicionar vários spokes. Pode ser útil mover alguns desses serviços compartilhados para um segundo nível de hubs.

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para essa arquitetura está disponível no [GitHub][ref-arch-repo]. A implantação cria os seguintes grupos de recursos em sua assinatura:

- hub-adds-rg
- hub-nva-rg
- hub-vnet-rg
- onprem-vnet-rg
- spoke1-vnet-rg
- spoke2-vent-rg

Os arquivos de parâmetros de modelo se referem a esses nomes e, portanto, se você alterá-los, atualize os arquivos de parâmetros para que correspondam.

### <a name="prerequisites"></a>Pré-requisitos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a>Implantar o datacenter local simulado usand azbb

Esta etapa implanta o datacenter local simulado como uma rede virtual do Azure.

1. Navegue até a pasta `hybrid-networking\shared-services-stack\` do repositório do GitHub.

2. Abra o arquivo `onprem.json` . 

3. Pesquise todas as instâncias de `UserName`, `adminUserName`, `Password` e `adminPassword`. Insira valores para o nome de usuário e a senha nos parâmetros e salve o arquivo. 

4. Execute o comando a seguir:

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
   ```
5. Aguarde até que a implantação seja concluída. Essa implantação cria uma rede virtual, uma máquina virtual que executa Windows e um gateway de VPN. A criação do gateway de VPN pode levar mais de 40 minutos para ser concluída.

### <a name="deploy-the-hub-vnet"></a>Implantar a VNET do hub

Esta etapa implanta o hub de rede virtual e conecta-se à rede virtual local simulada.

1. Abra o arquivo `hub-vnet.json` . 

2. Pesquise `adminPassword` e insira um nome de usuário e senha nos parâmetros. 

3. Pesquise todas as instâncias de `sharedKey` e insira um valor para uma chave compartilhada. Salve o arquivo.

   ```bash
   "sharedKey": "abc123",
   ```

4. Execute o comando a seguir:

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
   ```

5. Aguarde até que a implantação seja concluída. Essa implantação cria uma rede virtual, uma máquina virtual, um gateway de VPN e uma conexão com o gateway criado na seção anterior. O gateway de VPN pode levar mais de 40 minutos para ser concluído.

### <a name="deploy-ad-ds-in-azure"></a>Implantar o Active Directory Domain Services no Azure

Esta etapa implanta os controladores de domínio do Active Directory Domain Services no Azure.

1. Abra o arquivo `hub-adds.json` .

2. Pesquise todas as instâncias de `Password` e `adminPassword`. Insira valores para o nome de usuário e a senha nos parâmetros e salve o arquivo. 

3. Execute o comando a seguir:

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg -l <location> -p hub-adds.json --deploy
   ```
  
Esta etapa de implantação poderá levar vários minutos, porque adiciona as duas máquinas virtuais ao domínio hospedado no datacenter simulado local e instala o AD DS neles.

### <a name="deploy-the-spoke-vnets"></a>Implantar as redes virtuais spoke

Esta etapa implanta as redes virtuais spoke.

1. Abra o arquivo `spoke1.json` .

2. Pesquise `adminPassword` e insira um nome de usuário e senha nos parâmetros. 

3. Execute o comando a seguir:

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. Repita as etapas 1 e 2 para o arquivo `spoke2.json`.

5. Execute o comando a seguir:

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

### <a name="peer-the-hub-vnet-to-the-spoke-vnets"></a>Nivelar a rede virtual do hub com as redes virtuais spoke

Para criar uma conexão de emparelhamento da rede virtual do hub para as redes virtuais spoke, execute o comando a seguir:

```bash
azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
```

### <a name="deploy-the-nva"></a>Implantar a NVA

Esta etapa implanta uma NVA na sub-rede `dmz`.

1. Abra o arquivo `hub-nva.json` .

2. Pesquise `adminPassword` e insira um nome de usuário e senha nos parâmetros. 

3. Execute o comando a seguir:

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

### <a name="test-connectivity"></a>Testar a conectividade 

Testar a conexão do ambiente local simulado para a VNET do hub.

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

Repita as mesmas etapas para testar a conectividade com as redes virtuais spoke:

```powershell
Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
```


<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[guidance-hub-spoke]: ./hub-spoke.md
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[datacenter virtual]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Topologia de serviços compartilhados no Azure"
[3]: ./images/hub-spokehub-spoke.svg "Topologia hub-spoke-hub-spoke no Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
