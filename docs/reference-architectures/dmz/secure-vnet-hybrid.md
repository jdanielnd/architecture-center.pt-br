---
title: Implementar uma arquitetura de rede híbrida segura
titleSuffix: Azure Reference Architectures
description: Implemente uma arquitetura de rede híbrida segura no Azure.
author: telmosampaio
ms.date: 10/22/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, networking
ms.openlocfilehash: 82327cca08e614bfe5226c9ca1a414388878a7c2
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243507"
---
# <a name="implement-a-dmz-between-azure-and-your-on-premises-datacenter"></a>Implementar uma rede de perímetro entre o Azure e o datacenter local

Essa arquitetura de referência mostra uma rede híbrida segura que estende uma rede local para o Azure. A arquitetura implementa uma rede de perímetro, também chamada de *DMZ*, entre a rede local e uma VNet (rede virtual) do Azure. O DMZ inclui NVAs (soluções de virtualização de rede) que implementam a funcionalidade de segurança, como firewalls e inspeção de pacotes. Todo o tráfego de saída da VNet é enviado por túnel forçadamente à Internet por meio da rede local, para que ele possa ser auditado. [**Implantar esta solução**](#deploy-the-solution).

> [!NOTE]
> Este cenário também pode ser obtido usando o [Firewall do Azure](/azure/firewall/), um serviço de segurança de rede baseado em nuvem.

![Proteger arquitetura de rede híbrida](./images/dmz-private.png)

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

Essa arquitetura exige uma conexão com o datacenter local, usando um [Gateway de VPN][ra-vpn] ou uma conexão do [ExpressRoute][ra-expressroute]. Alguns usos típicos dessa arquitetura:

- Aplicativos híbridos nos quais as cargas de trabalho são executadas parcialmente localmente e parcialmente no Azure.
- Infraestrutura que requer um controle granular sobre o tráfego que entra em uma VNet do Azure proveniente de um datacenter local.
- Aplicativos que precisam auditar o tráfego de saída. Isso geralmente é um requisito de regulamentação de muitos sistemas comerciais e pode ajudar a evitar a divulgação de informações privadas.

## <a name="architecture"></a>Arquitetura

A arquitetura consiste nos componentes a seguir.

- **Rede local**. Uma rede local privada implementada em uma organização.
- **VNet (rede virtual) do Azure**. A VNet hospeda o aplicativo e outros recursos em execução no Azure.
- **Gateway**. O gateway fornece conectividade entre os roteadores na rede local e na VNet.
- **NVA (solução de virtualização de rede)**. NVA é um termo genérico que descreve uma VM que executa tarefas como permitir ou negar acesso como um firewall, otimizar operações de WAN (rede de longa distância) (incluindo compactação de rede), roteamento personalizado ou outros recursos de rede.
- **Sub-redes de camada da Web, de camada de negócios e de camada de dados**. Sub-redes que hospedam as VMs e os serviços que implementam um aplicativo de três camadas de exemplo em execução na nuvem. Consulte [Executar VMs do Windows para um aplicativo de N camadas no Azure][ra-n-tier] para obter mais informações.
- **UDR (rotas definidas pelo usuário)**. As [rotas definidas pelo usuário][udr-overview] definem o fluxo de tráfego de IP nas VNets do Azure.

    > [!NOTE]
    > Dependendo dos requisitos da sua conexão de VPN, você pode configurar rotas de BGP (Border Gateway Protocol) em vez de usar UDRs para implementar as regras de encaminhamento que retornam o tráfego por meio da rede local.
    >

- **Sub-rede de gerenciamento**. Essa sub-rede contém VMs que implementam recursos de gerenciamento e monitoramentos para os componentes em execução na VNet.

## <a name="recommendations"></a>Recomendações

As seguintes recomendações aplicam-se à maioria dos cenários. Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.

### <a name="access-control-recommendations"></a>Recomendações de controle de acesso

Use o [RBAC (controle de acesso baseado em função)][rbac] para gerenciar os recursos em seu aplicativo. Considere criar as seguintes [funções personalizadas][rbac-custom-roles]:

- Uma função de DevOps com permissões para administrar a infraestrutura do aplicativo, implantar os componentes do aplicativo e monitorar e reiniciar VMs.

- Uma função de administrador de TI centralizada para gerenciar e monitorar recursos de rede.

- Uma função de administrador de TI de segurança para gerenciar recursos de rede segura, como as NVAs.

As funções de DevOps e de administrador de TI não devem ter acesso aos recursos de NVA. Esse acesso deve ser restrito à função de administrador de TI de segurança.

### <a name="resource-group-recommendations"></a>Recomendações do grupo de recursos

Os recursos do Azure como VMs, VNets e balanceadores de carga podem ser gerenciados facilmente ao serem agrupados em grupos de recursos. Atribua funções de RBAC a cada grupo de recursos para restringir o acesso.

É recomendável criar os grupos de recursos a seguir:

- Um grupo de recursos que contenha a VNet (excluindo as VMs), os NSGs e os recursos de gateway para conectar-se à rede local. Atribua a função de administrador de TI centralizada a esse grupo de recursos.
- Um grupo de recursos que contenha as VMs para as NVAs (incluindo o balanceador de carga), o jumpbox e outras VMs de gerenciamento e a UDR da sub-rede do gateway que force todo o tráfego pelas NVAs. Atribua a função de administrador de TI de segurança a este grupo de recursos.
- Grupos de recursos separados para cada camada de aplicativo que contenha o balanceador de carga e as VMs. Observe que esse grupo de recursos não deve incluir as sub-redes de cada camada. Atribua a função de DevOps a este grupo de recursos.

### <a name="virtual-network-gateway-recommendations"></a>Recomendações do gateway de rede virtual

O tráfego local passa para a VNet por meio de um gateway de rede virtual. É recomendável um [Gateway de VPN do Azure][guidance-vpn-gateway] ou um [gateway do Azure ExpressRoute][guidance-expressroute].

### <a name="nva-recommendations"></a>Recomendações de NVA

As NVAs fornecem diferentes serviços para gerenciar e monitorar o tráfego de rede. O [Azure Marketplace][azure-marketplace-nva] oferece vários NVAs de fornecedores terceiros que você pode usar. Se nenhuma dessas NVAs de terceiros atender aos seus requisitos, você poderá criar uma NVA personalizada usando VMs.

Por exemplo, a implantação da solução para essa arquitetura de referência implementa uma NVA com a seguinte funcionalidade em uma VM:

- O tráfego é roteado usando [encaminhamento de IP][ip-forwarding] nos NICs (adaptadores de rede) da NVA.
- O tráfego terá permissão para passar pela NVA somente se ele for apropriado para isso. Cada VM da NVA na arquitetura de referência é um roteador simples do Linux. O tráfego de entrada chega no adaptador de rede *eth0* e o tráfego de saída corresponde às regras definidas por scripts personalizados expedidos por meio da adaptador de rede *eth1*.
- As NVAs só podem ser configuradas por meio da sub-rede de gerenciamento.
- O tráfego roteado para a sub-rede de gerenciamento não passa pelas NVAs. Caso contrário, se as NVAs falharem, não haverá nenhuma rota para a sub-rede de gerenciamento para corrigi-las.
- As VMs da NVA são colocadas em um [conjunto de disponibilidade][availability-set] atrás de um balanceador de carga. A UDR na sub-rede do gateway direciona as solicitações da NVA ao balanceador de carga.

Inclua uma NVA da camada 7 para encerrar conexões de aplicativo no nível da NVA e manter a afinidade com as camadas de back-end. Isso garante a conectividade simétrica, na qual o tráfego de resposta das camadas de back-end retorna por meio da NVA.

Outra opção a ser considerada é conectar várias NVAs em série, com cada NVA executando uma tarefa de segurança especializada. Isso permite que cada função de segurança seja gerenciada considerando cada NVA. Por exemplo, uma NVA que implementa um firewall pode ser colocada em série com uma NVA que executa serviços de identidade. A compensação para facilitar o gerenciamento é a adição de saltos de rede extras que podem aumentar a latência, portanto, verifique se isso não afeta o desempenho do aplicativo.

### <a name="nsg-recommendations"></a>Recomendações de NSG

O Gateway de VPN expõe um endereço IP público para a conexão com a rede local. É recomendável criar um NSG (Grupo de Segurança de Rede) para a sub-rede da NVA de entrada, com regras para bloquear todo o tráfego não originado na rede local.

Também é recomendável que os NSGs de cada sub-rede forneçam um segundo nível de proteção contra o tráfego de entrada ignorando uma NVA desabilitada ou configurada incorretamente. Por exemplo, a sub-rede de camada da Web na arquitetura de referência implementa um NSG com uma regra para ignorar todas as solicitações que não sejam as recebidas da rede local (192.168.0.0/16) ou da VNet, e outra regra que ignora todas as solicitações que não são feitas na porta 80.

### <a name="internet-access-recommendations"></a>Recomendações de acesso à Internet

[Force por túnel][azure-forced-tunneling] todo o tráfego de Internet de saída pela rede local usando o túnel de VPN site a site e rotear à Internet usando a NAT (conversão de endereços de rede). Isso impede a perda acidental de informações confidenciais armazenadas na camada de dados e permite a inspeção e a auditoria de todo o tráfego de saída.

> [!NOTE]
> Não bloqueie completamente o tráfego de Internet das camadas de aplicativo, pois isso impedirá que essas camadas usem os serviços de PaaS do Azure que se baseiam em endereços IP públicos, como o log de diagnóstico de VM, o download de extensões de VM e outras funcionalidades. O diagnóstico do Azure também requer que componentes possam ler e gravar em uma conta de armazenamento do Azure.

Verifique se o tráfego de saída da Internet é forçado ao túnel corretamente. Se você estiver usando uma conexão de VPN com o [serviço de roteamento e acesso remoto][routing-and-remote-access-service] em um servidor local, use uma ferramenta como o [WireShark][wireshark] ou o [Microsoft Message Analyzer](https://www.microsoft.com/download/details.aspx?id=44226).

### <a name="management-subnet-recommendations"></a>Recomendações da sub-rede de gerenciamento

A sub-rede de gerenciamento contém um jumpbox que executa as funcionalidades de gerenciamento e de monitoramento. Restrinja a execução de todas as tarefas de gerenciamento seguro ao jumpbox.

Não crie um endereço IP público para o jumpbox. Nesse caso, crie uma rota para acessar o jumpbox por meio do gateway de entrada. Crie regras do NSG para que a sub-rede de gerenciamento só responda a solicitações da rota permitida.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

A arquitetura de referência usa um balanceador de carga para direcionar o tráfego de rede local a um pool de dispositivos NVA, que roteiam o tráfego. As NVAs são colocadas em um [conjunto de disponibilidade][availability-set]. Esse design permite que você monitore a produtividade das NVAs ao longo do tempo e adicione dispositivos NVA em resposta ao aumento de carga.

O Gateway de VPN da SKU Standard dá suporte para produtividade prolongada de até 100 Mbps. A SKU de alto desempenho oferece até 200 Mbps. Para larguras de banda superiores, considere atualizar para um gateway do ExpressRoute. O ExpressRoute fornece até 10 Gbps de largura de banda com uma latência menor do que uma conexão de VPN.

Para obter mais informações sobre a escalabilidade dos gateways do Azure, consulte a seção de consideração de escalabilidade em [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-scalability] (Implementando uma arquitetura de rede híbrida com o Azure e VPN local) e [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-scalability] (Implementando uma arquitetura de rede híbrida com o Azure ExpressRoute).

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Conforme mencionado, a arquitetura de referência usa um pool de dispositivos NVA atrás de um balanceador de carga. O balanceador de carga usa uma investigação de integridade para monitorar cada NVA e removerá todas as NVAs sem resposta do pool.

Se você estiver usando o Azure ExpressRoute para fornecer conectividade entre a VNet e a rede local, [configure um Gateway de VPN para fornecer failover][ra-vpn-failover] se a conexão do ExpressRoute ficar indisponível.

Para obter informações específicas de como manter a disponibilidade para conexões de VPN e do ExpressRoute, consulte as considerações de disponibilidade [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-availability] (Implementando uma arquitetura de rede híbrida com o Azure e VPN local) e [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-availability] (Implementando uma arquitetura de rede híbrida com o Azure ExpressRoute).

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Todo o monitoramento de aplicativos e de recursos deve ser realizado pelo jumpbox na sub-rede de gerenciamento. Dependendo dos requisitos do aplicativo, talvez sejam necessários recursos de monitoramentos adicionais na sub-rede de gerenciamento. Nesse caso, esses recursos deverão ser acessados por meio do jumpbox.

Se a conectividade de gateway da rede local para o Azure estiver inativa, você ainda poderá acessar o jumpbox implantando um endereço IP público, adicionando-o ao jumpbox e comunicando-se remotamente pela Internet.

A sub-rede de cada camada na arquitetura de referência é protegida por regras de NSG. Talvez seja necessário criar uma regra para abrir a porta 3389 para acesso do protocolo RDP em VMs do Windows ou a porta 22 para acesso de SSH (Secure Shell) em VMs do Linux. Outras ferramentas de gerenciamento e monitoramentos podem exigir regras para abrir portas adicionais.

Se você estiver usando o ExpressRoute para fornecer a conectividade entre o datacenter local e o Azure, use o [AzureCT (Kit de ferramentas de conectividade do Azure)][azurect] para monitorar e solucionar problemas de conexão.

Você pode encontrar informações adicionais especificamente destinadas ao monitoramento e ao gerenciamento de conexões de VPN e do ExpressRoute nos artigos [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-manageability] (Implementando uma arquitetura de rede híbrida com o Azure e VPN local) e [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-manageability] (Implementando uma arquitetura de rede híbrida com o Azure ExpressRoute).

## <a name="security-considerations"></a>Considerações de segurança

Essa arquitetura de referência implementa vários níveis de segurança.

### <a name="routing-all-on-premises-user-requests-through-the-nva"></a>Roteando todas as solicitações de usuário locais por meio da NVA

A UDR na sub-rede do gateway bloqueia todas as solicitações de usuário que não são recebidas do local. A UDR passa as solicitações permitidas para as NVAs na sub-rede do DMZ privado e essas solicitações somente serão passadas para o aplicativo se forem permitidas pelas regras da NVA. Você pode adicionar outras rotas para a UDR, mas não ignore as NVAs nem bloqueie o tráfego administrativo destinado à sub-rede de gerenciamento acidentalmente.

O balanceador de carga na frente das NVAs também atua como um dispositivo de segurança ignorando o tráfego nas portas que não estão abertas nas regras de balanceamento de carga. Os balanceadores de carga na arquitetura de referência só escutam as solicitações HTTP na porta 80 e as solicitações HTTPS na porta 443. Documente todas as regras adicionais que você adicionar aos balanceadores de carga e monitore o tráfego para garantir que não haja nenhum problema de segurança.

### <a name="using-nsgs-to-blockpass-traffic-between-application-tiers"></a>Usando os NSGs para bloquear/aprovar o tráfego entre as camadas de aplicativo

O tráfego entre camadas é restringido usando NSGs. A camada de negócios bloqueia todo o tráfego que não se origina na camada da Web e a camada de dados bloqueia todo o tráfego que não se origina na camada de negócios. Se você tiver o requisito de expandir as regras do NSG para ampliar o acesso a essas camadas, pondere esses requisitos em relação aos riscos de segurança. Cada novo caminho de entrada representa uma oportunidade para danos de aplicativo ou perda de dados acidental ou intencional.

### <a name="devops-access"></a>Acesso de DevOps

Use o [RBAC][rbac] para restringir as operações que DevOps pode executar em cada camada. Ao conceder permissões, use o [princípio de privilégios mínimos][security-principle-of-least-privilege]. Registre em log todas as operações administrativas e execute auditorias regulares para confirmar se todas as alterações de configuração foram planejadas.

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para uma arquitetura de referência que implementa essas recomendações está disponível no [GitHub][github-folder].

### <a name="prerequisites"></a>Pré-requisitos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a>Implantação de recursos

1. Navegue para a pasta `/dmz/secure-vnet-hybrid` do repositório GitHub de arquiteturas de referência.

2. Execute o comando a seguir:

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. Execute o comando a seguir:

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a>Conectar os gateways do Azure e locais

Nesta etapa, você conectará os dois gateways de rede locais.

1. No portal do Azure, navegue até o grupo de recursos que você criou.

2. Localize o recurso denominado `ra-vpn-vgw-pip` e copie o endereço IP mostrado na folha **Visão geral**.

3. Encontre o recurso denominado `onprem-vpn-lgw`.

4. Clique na folha **Configuração**. No **Endereço IP**, cole o endereço IP da etapa 2.

    ![Captura de tela do campo Endereço IP](./images/local-net-gw.png)

5. Clique em **Salvar** e aguarde a conclusão da operação. Isso pode levar cerca de 5 minutos.

6. Encontre o recurso denominado `onprem-vpn-gateway1-pip`. Copie o endereço IP mostrado na folha **Visão geral**.

7. Encontre o recurso denominado `ra-vpn-lgw`.

8. Clique na folha **Configuração**. No **Endereço IP**, cole o endereço IP da etapa 6.

9. Clique em **Salvar** e aguarde a conclusão da operação.

10. Para verificar a conexão, vá até a folha **Conexões** de cada gateway. O status deve ser **Conectado**.

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a>Verificar se o tráfego de rede atinge a camada da Web

1. No portal do Azure, navegue até o grupo de recursos que você criou.

2. Encontre o recurso denominado `int-dmz-lb`, que é o balanceador de carga na frente da DMZ privada. Copie o endereço IP privado da folha **Visão geral**.

3. Encontre a VM denominada `jb-vm1`. Clique em **Conectar** e use a Área de Trabalho Remota para se conectar à VM. O nome de usuário e a senha são especificadas no arquivo onprem.json.

4. Na sessão de Área de Trabalho Remota, abra um navegador da Web e navegue até o endereço IP da etapa 2. Você deve ver a home page padrão do servidor Apache2.

## <a name="next-steps"></a>Próximas etapas

- Aprenda a implementar uma [rede de perímetro entre o Azure e a Internet](./secure-vnet-dmz.md).
- Aprenda a implementar uma [arquitetura de rede híbrida altamente disponível][ra-vpn-failover].
- Para obter mais informações de como gerenciar a segurança da rede com o Azure, consulte [Segurança de rede e serviços em nuvem da Microsoft][cloud-services-network-security].
- Para obter informações detalhadas sobre como proteger recursos no Azure, consulte [Introdução à segurança do Microsoft Azure][getting-started-with-azure-security].
- Para obter detalhes adicionais de como solucionar preocupações de segurança em uma conexão de gateway do Azure, consulte [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-security] (Implementando uma arquitetura de rede híbrida com o Azure e VPN local) e [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-security] (Implementando uma arquitetura de rede híbrida com o Azure ExpressRoute).
- [Solução de problemas de virtualização de rede no Azure](/azure/virtual-network/virtual-network-troubleshoot-nva)

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[azure-forced-tunneling]: https://azure.microsoft.com/en-gb/documentation/articles/vpn-gateway-forced-tunneling-rm/
[azure-marketplace-nva]: https://azuremarketplace.microsoft.com/marketplace/apps/category/networking
[cloud-services-network-security]: /azure/best-practices-network-security
[getting-started-with-azure-security]: /azure/security/azure-security-getting-started
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-hybrid
[guidance-expressroute]: ../hybrid-networking/expressroute.md
[guidance-expressroute-availability]: ../hybrid-networking/expressroute.md#availability-considerations
[guidance-expressroute-manageability]: ../hybrid-networking/expressroute.md#manageability-considerations
[guidance-expressroute-security]: ../hybrid-networking/expressroute.md#security-considerations
[guidance-expressroute-scalability]: ../hybrid-networking/expressroute.md#scalability-considerations
[guidance-vpn-gateway]: ../hybrid-networking/vpn.md
[guidance-vpn-gateway-availability]: ../hybrid-networking/vpn.md#availability-considerations
[guidance-vpn-gateway-manageability]: ../hybrid-networking/vpn.md#manageability-considerations
[guidance-vpn-gateway-scalability]: ../hybrid-networking/vpn.md#scalability-considerations
[guidance-vpn-gateway-security]: ../hybrid-networking/vpn.md#security-considerations
[ip-forwarding]: /azure/virtual-network/virtual-networks-udr-overview#ip-forwarding
[ra-expressroute]: ../hybrid-networking/expressroute.md
[ra-n-tier]: ../virtual-machines-windows/n-tier.md
[ra-vpn]: ../hybrid-networking/vpn.md
[ra-vpn-failover]: ../hybrid-networking/expressroute-vpn-failover.md
[rbac]: /azure/active-directory/role-based-access-control-configure
[rbac-custom-roles]: /azure/active-directory/role-based-access-control-custom-roles
[routing-and-remote-access-service]: https://technet.microsoft.com/library/dd469790(v=ws.11).aspx
[security-principle-of-least-privilege]: https://msdn.microsoft.com/library/hdb58b2f(v=vs.110).aspx#Anchor_1
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview
[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
[wireshark]: https://www.wireshark.org/
