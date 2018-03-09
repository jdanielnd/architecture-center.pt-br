---
title: "Implementação de uma topologia de rede hub-spoke com serviços compartilhados no Azure"
description: "Como implementar uma topologia de rede hub-spoke com serviços compartilhados no Azure."
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: c0fb1d1ddd7c70ed914d58e7c73b10475b91aedf
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/08/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a>Implementar uma topologia de rede hub-spoke com serviços compartilhados no Azure

Essa arquitetura de referência se baseia na arquitetura de referência do [hub-spoke] [ guidance-hub-spoke] para incluir serviços compartilhados no hub que pode ser consumido por todos os spokes. Como uma primeira etapa para migrar um datacenter para a nuvem, e criando um [datacenter virtual], os primeiros serviços que você precisa compartilhar são identidade e segurança. Esta arquitetura de referência mostra como estender seus serviços do Active Directory de seu datacenter local para o do Azure e como adicionar uma solução de virtualização de rede (NVA) que pode agir como um firewall, em uma topologia hub-spoke.  [**Implantar esta solução**](#deploy-the-solution).

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

Uma implantação para essa arquitetura está disponível no [GitHub][ref-arch-repo]. Ela usa VMs Ubuntu em cada VNet para testar a conectividade. Não há nenhum serviço hospedado na sub-rede **shared-services** na **VNet do hub**.

### <a name="prerequisites"></a>pré-requisitos

Antes de implantar a arquitetura de referência para sua própria assinatura, você deve executar as etapas a seguir.

1. Clone, crie fork ou baixe o arquivo zip para as [arquiteturas de referência AzureCAT][ref-arch-repo] no repositório GitHub.

2. Verifique se a CLI do Azure 2.0 está instalada no computador. Para obter instruções de instalação da CLI, consulte [Instalar a CLI 2.0 do Azure][azure-cli-2].

3. Instale os pacote npm dos [Blocos de construção do Azure][azbb].

4. Em um prompt de comando, bash prompt ou prompt do PowerShell, faça logon na sua conta do Azure usando o comando abaixo e siga os prompts.

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a>Implantar o datacenter local simulado usand azbb

Para implantar o datacenter local simulado como uma rede virtual do Azure, siga as seguintes as etapas:

1. Navegue até a pasta `hybrid-networking\shared-services-stack\` do repositório que você baixou na etapa de pré-requisitos acima.

2. Abra o arquivo `onprem.json` e insira um nome de usuário e senha entre aspas nas linhas 45 e 46, conforme mostrado abaixo e salve o arquivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Execute `azbb` para implantar o ambiente simulado local, conforme mostrado abaixo.

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > Se você decidir usar um nome do grupo de recursos distinto (diferente de `onprem-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.

4. Aguarde até que a implantação seja concluída. Essa implantação cria uma rede virtual, uma máquina virtual que executa Windows e um gateway de VPN. A criação do gateway de VPN pode levar mais de 40 minutos para ser concluída.

### <a name="azure-hub-vnet"></a>VNet do hub do Azure

Para implantar a VNet de hub e conectar-se à VNet local simulada criada acima, execute as seguintes etapas.

1. Abra o arquivo `hub-vnet.json` e insira um nome de usuário e senha entre aspas nas linhas 50 e 51, conforme mostrado abaixo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. Na linha 52, para `osType`, digite `Windows` ou `Linux` para instalar o Windows Server 2016 Datacenter ou o Ubuntu 16.04 como o sistema operacional para o jumpbox.

3. Insira uma chave compartilhada entre as aspas na linha 83, conforme mostrado abaixo, e salve o arquivo.

  ```bash
  "sharedKey": "",
  ```

4. Execute `azbb` para implantar o ambiente simulado local, conforme mostrado abaixo.

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > Se você decidir usar um nome do grupo de recursos distinto (diferente de `hub-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.

5. Aguarde até que a implantação seja concluída. Essa implantação cria uma rede virtual, uma máquina virtual, um gateway de VPN e uma conexão com o gateway criado na seção anterior. A criação do gateway de VPN pode levar mais de 40 minutos para ser concluída.

### <a name="adds-in-azure"></a>Exibir no ADDS

Para implantar os controladores de domínio ADDS no Azure, execute as etapas a seguir.

1. Abra o arquivo `hub-adds.json` e insira um nome de usuário e senha entre aspas nas linhas 14 e 15, conforme mostrado abaixo, e salve o arquivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. Execute `azbb` para implantar os controladores de domínio ADDS, conforme mostrado abaixo.

  ```bash
  azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
  ```
  
  > [!NOTE]
  > Se você decidir usar um nome do grupo de recursos distinto (diferente de `hub-adds-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.

  > [!NOTE]
  > Esta parte da implantação pode levar vários minutos, pois requer a união das duas máquinas virtuais ao domínio hospedado no datacenter local simulado, instalando o AD DS neles.

### <a name="nva"></a>NVA

Para implantar uma NVA na sub-rede `dmz`, execute as seguintes etapas:

1. Abra o arquivo `hub-nva.json` e insira um nome de usuário e senha entre aspas nas linhas 13 e 14, conforme mostrado abaixo, e salve o arquivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. Execute `azbb` para implantar a VM NVA e as rotas de usuário definidas.

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > Se você decidir usar um nome do grupo de recursos distinto (diferente de `hub-nva-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.

### <a name="azure-spoke-vnets"></a>VNets de spoke do Azure

Para implantar as redes virtuais spoke, execute as etapas a seguir.

1. Abra o arquivo `spoke1.json` e insira um nome de usuário e senha entre aspas nas linhas 52 e 53, conforme mostrado abaixo, e salve o arquivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. Na linha 54, para `osType`, digite `Windows` ou `Linux` para instalar o Windows Server 2016 Datacenter ou o Ubuntu 16.04 como o sistema operacional para o jumpbox.

3. Execute `azbb` para implantar o primeiro ambiente de rede virtual spoke, conforme mostrado abaixo.

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > Se você decidir usar um nome do grupo de recursos distinto (diferente de `spoke1-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.

3. Repita a etapa 1 acima para o arquivo `spoke2.json`.

4. Execute `azbb` para implantar o segundo ambiente de rede virtual spoke, conforme mostrado abaixo.

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > Se você decidir usar um nome do grupo de recursos distinto (diferente de `spoke2-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a>Emparelhamento VNet do hub do Azure para VNets de spoke

Para criar uma conexão de emparelhamento da rede virtual do hub para as redes virtuais spoke, execute as etapas a seguir.

1. Abra o arquivo `hub-vnet-peering.json` e verifique se o nome do grupo de recursos e o nome da rede virtual para cada um dos emparelhamentos de rede virtual, começando na linha 29, estão corretos.

2. Execute `azbb` para implantar o primeiro ambiente de rede virtual spoke, conforme mostrado abaixo.

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > Se você decidir usar um nome do grupo de recursos distinto (diferente de `hub-vnet-rg`), pesquise todos os arquivos de parâmetro que usam esse nome e edite-os para usar seu próprio nome do grupo de recursos.

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

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Topologia de serviços compartilhados no Azure"
[3]: ./images/hub-spokehub-spoke.svg "Topologia hub-spoke-hub-spoke no Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
