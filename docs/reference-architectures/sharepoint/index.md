---
title: Executar um farm do SharePoint Server 2016 de alta disponibilidade no Azure
description: Práticas comprovadas para configurar uma farm do SharePoint Server 2016 de alta disponibilidade no Azure.
author: njray
ms.date: 08/01/2017
ms.openlocfilehash: 9fe4fc09cf3babdf3ec8e8f27049f90e0047e9f0
ms.sourcegitcommit: 776b8c1efc662d42273a33de3b82ec69e3cd80c5
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/12/2018
ms.locfileid: "38987702"
---
# <a name="run-a-high-availability-sharepoint-server-2016-farm-in-azure"></a>Executar um farm do SharePoint Server 2016 de alta disponibilidade no Azure

Essa arquitetura de referência mostra um conjunto de práticas comprovadas para configurar um farm do SharePoint Server 2016 de alta disponibilidade no Azure, usando a topologia de MinRole e grupos de disponibilidade Always On do SQL Server. O farm do SharePoint é implantado em uma rede virtual protegida sem nenhuma presença ou ponto de extremidade para a Internet. [**Implante essa solução**.](#deploy-the-solution) 

![](./images/sharepoint-ha.png)

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura

Essa arquitetura baseia-se naquela mostrada em [Executar máquinas virtuais do Windows para um aplicativo de N camadas][windows-n-tier]. Ela implanta um farm do SharePoint Server 2016 com alta disponibilidade dentro de uma rede virtual do Azure (VNet). Essa arquitetura é adequada para um ambiente de teste ou de produção, uma infraestrutura híbrida do SharePoint com o Office 365 ou como base para um cenário de recuperação de desastres.

Essa arquitetura consiste nos seguintes componentes:

- **Grupos de recursos**. Um [grupo de recursos][resource-group] é um contêiner que armazena os recursos relacionados ao Azure. Um grupo de recursos é usado para os servidores do SharePoint e outro grupo de recursos é usado para os componentes de infraestrutura que são independentes das VMs, como a rede virtual e os balanceadores de carga.

- **Rede virtual (VNet)**. As VMs são implantadas em uma rede virtual com um espaço de endereço da intranet exclusivo. A rede virtual é, posteriormente, subdividida em sub-redes. 

- **Máquinas Virtuais (VMs)**. As VMs são implantadas na rede virtual e os endereços IP estáticos são atribuídos a todas as VMs. Os endereços IP estáticos são recomendados para as máquinas virtuais que executam o SQL Server e o SharePoint Server 2016, para evitar problemas com armazenamento em cache de endereço IP e alterações de endereços após uma reinicialização.

- **Conjuntos de disponibilidade**. Coloque as VMs para cada função do SharePoint em [conjuntos de disponibilidade][availability-set] separados e forneça pelo menos duas máquinas virtuais (VMs) para cada função. Isso torna as VMs qualificadas para um contrato de nível de serviço (SLA) mais alto. 

- **Balanceador Interno de carga**. O [balanceador de carga][load-balancer] distribui o tráfego de solicitação do SharePoint da rede local para os servidores Web de front-end do farm do SharePoint. 

- **Grupos de segurança de rede (NSG)**. Para cada sub-rede que contém máquinas virtuais, um [grupo de segurança de rede][nsg] é criado. Use os NSGs para restringir o tráfego de rede na VNet, para isolar as sub-redes. 

- **Gateway**. O gateway fornece uma conexão entre a sua rede local e a rede virtual do Azure. A conexão pode usar ExpressRoute ou VPN site a site. Para obter mais informações, consulte [Conectar uma rede local ao Azure][hybrid-ra].

- **Controladores de domínio do Active Directory (AD) do Windows Server**. Essa arquitetura de referência implanta os controladores de domínio do AD do Windows Server. Esses controladores de domínio são executados na rede virtual do Azure e possuem uma relação de confiança com a floresta do AD do Windows Server local. As solicitações da Web do cliente por recursos de farm do SharePoint podem ser autenticadas na rede virtual, em vez de enviar esse tráfego de autenticação pela conexão do gateway para a rede local. No DNS, os registros CNAME ou intranet um são criados para que os usuários da intranet possam resolver o nome do farm do SharePoint para o endereço IP privado do balanceador de carga interno.

  O SharePoint Server 2016 também suporta o uso do [Azure Active Directory Domain Services](/azure/active-directory-domain-services/). O Azure Active Directory Domain Services fornece serviços de domínio gerenciado para que você não precise implantar e gerenciar controladores de domínio no Azure.

- **Grupo de Disponibilidade Always On do SQL Server**. Para obter alta disponibilidade do banco de dados do SQL Server, recomendamos os [Grupos de Disponibilidade AlwaysOn do SQL Server][sql-always-on]. Duas máquinas virtuais são usadas para o SQL Server. Uma contém a réplica do banco de dados primário e a outra contém a réplica secundária. 

- **Nó principal da VM**. Essa VM permite que o cluster de failover estabeleça um quorum. Para saber mais, consulte [Noções básicas sobre configurações de quorum em um cluster de failover][sql-quorum].

- **Servidores do SharePoint**. Os servidores do SharePoint executam funções de front-end da Web, armazenamento em cache, aplicativo e pesquisa. 

- **Jumpbox**. Também chamado de um [host bastião][bastion-host]. Esta é uma VM segura na rede que os administradores usam para se conectar às outras VMs. O jumpbox tem um NSG que permite o tráfego remoto apenas de endereços IP públicos em uma lista segura. O NSG deve permitir o tráfego de RDP (área de trabalho remota).

## <a name="recommendations"></a>Recomendações

Seus requisitos podem ser diferentes dos requisitos da arquitetura descrita aqui. Use essas recomendações como ponto de partida.

### <a name="resource-group-recommendations"></a>Recomendações do grupo de recursos

É recomendável separar grupos de recursos de acordo com a função de servidor e ter um grupo de recursos separado para os componentes de infraestrutura que são recursos globais. Nessa arquitetura, os recursos do SharePoint formam um grupo, enquanto o SQL Server e outros ativos de utilitário formam o outro.

### <a name="virtual-network-and-subnet-recommendations"></a>Recomendações para a rede virtual e a sub-rede

Use uma sub-rede para cada função do SharePoint, além de uma sub-rede para o gateway e uma para o jumpbox. 

A sub-rede de gateway deve ser nomeada *GatewaySubnet*. Atribua o espaço de endereço de sub-rede de gateway da última parte do espaço de endereço de rede virtual. Para obter mais informações, consulte [Conectar uma rede local ao Azure usando um gateway de VPN][hybrid-vpn-ra].

### <a name="vm-recommendations"></a>Recomendações de VM

Com base nos tamanhos de máquina virtual da série Standard DSv2, essa arquitetura exige um mínimo de 38 núcleos:

- 8 servidores do SharePoint em Standard_DS3_v2 (4 núcleos cada) = 32 núcleos
- 2 controladores de domínio do Active Directory em Standard_DS1_v2 (1 núcleo cada) = 2 núcleos
- 2 VMs do SQL Server em Standard_DS1_v2 = 2 núcleos
- 1 nó principal em Standard_DS1_v2 = 1 núcleo
- 1 servidor de gerenciamento em Standard_DS1_v2 = 1 núcleo

O número total de núcleos dependerá dos tamanhos de VM que você selecionar. Para obter mais informações, consulte [Recomendações de SharePoint Server](#sharepoint-server-recommendations) abaixo.

Certifique-se de que sua assinatura do Azure tem suficiente cota de núcleo da VM para a implantação ou a implantação falhará. Consulte [Assinatura do Azure e limites de serviço, cotas e restrições][quotas]. 
 
### <a name="nsg-recommendations"></a>Recomendações do NSG

É recomendável ter um NSG para cada sub-rede que contém máquinas virtuais, para permitir o isolamento de sub-rede. Se você quiser configurar o isolamento de sub-rede, adicione regras NSG que definem o tráfego de entrada ou saída permitido ou negado para cada sub-rede. Para obter mais informações, consulte [Filtrar o tráfego de rede com grupos de segurança de rede][virtual-networks-nsg]. 

Não atribua um NSG à sub-rede de gateway ou o gateway irá parar de funcionar. 

### <a name="storage-recommendations"></a>Recomendações de armazenamento

A configuração de armazenamento das VMs no farm devem corresponder às melhores práticas apropriadas usadas para implantações locais. Servidores do SharePoint devem ter um disco separado de logs. Os servidores do SharePoint que hospedam funções de índice de pesquisa requerem espaço em disco adicional para o índice de pesquisa a ser armazenado. Para o SQL Server, a prática padrão é separar os dados dos logs. Adicione mais discos para armazenamento de backup do banco de dados e use um disco separado para [tempdb][tempdb].

Para melhor confiabilidade, é recomendável usar o [Azure Managed Disks][managed-disks]. Os discos gerenciados garantem que os discos para VMs dentro de um conjunto de disponibilidade são isolados para evitar pontos únicos de falha. 

> [!NOTE]
> Atualmente o modelo do Resource Manager para esta arquitetura de referência não utiliza discos gerenciados. Estamos planejando atualizar o modelo para usar discos gerenciados.

Use discos gerenciados Premium para todas as VMs do SQL Server e do SharePoint. Você pode usar o Standard Managed Disks para o servidor de nós principal, os controladores de domínio e o servidor de gerenciamento. 

### <a name="sharepoint-server-recommendations"></a>Recomendações do SharePoint Server

Antes de configurar o farm do SharePoint, verifique se você tem uma conta de serviço do Active Directory do Windows Server por serviço. Para essa arquitetura, é necessário no mínimo as seguintes contas de nível de domínio para isolar o privilégio por função:

- Conta de serviço do SQL Server
- Conta de usuário de configuração
- Conta do Farm de servidor
- Conta de serviço de pesquisa
- Conta de acesso de conteúdo
- Contas de pool de aplicativos Web
- Contas de pool de aplicativos de serviço
- Conta de superusuário do cache
- Conta de superleitor do cache

Para todas as funções, exceto o indexador da pesquisa, é recomendável usar o tamanho de VM [Standard_DS3_v2][vm-sizes-general]. O indexador da pesquisa deve ser pelo menos do tamanho [Standard_DS13_v2][vm-sizes-memory]. 

> [!NOTE]
> O modelo do Resource Manager para esta arquitetura de referência usa o menor tamanho DS3 para o indexador de pesquisa, para fins de teste a implantação. Para uma implantação de produção, use o tamanho DS13 ou maior. 

Para cargas de trabalho de produção, consulte [Requisitos de hardware e software para o SharePoint Server 2016][sharepoint-reqs]. 

Para atender ao requisito de suporte para a taxa de transferência de disco de 200 MB por segundo mínimo, certifique-se de planejar a arquitetura de pesquisa. Consulte [Planejar a arquitetura de pesquisa no SharePoint Server 2013][sharepoint-search]. Siga também as diretrizes em [Melhores práticas para rastreamento no SharePoint Server 2016][sharepoint-crawling].

Além disso, armazene os dados do componente de pesquisa em um volume de armazenamento separado ou partição com alto desempenho. Para reduzir a carga e melhorar a taxa de transferência, configure as contas de usuário de cache do objeto, que são necessárias nesta arquitetura. Dividir os arquivos do sistema operacional Windows Server, arquivos de programa do SharePoint Server 2016 e logs de diagnóstico em três partições ou volumes de armazenamento separados com desempenho normal. 

Para obter mais informações sobre essas recomendações, consulte [Contas de serviço e administrativa da implantação inicial no SharePoint Server 2016][sharepoint-accounts].

### <a name="hybrid-workloads"></a>Cargas de trabalho híbridas

Essa arquitetura de referência implanta um farm do SharePoint Server 2016 que pode ser usado como um [ambiente híbrido do SharePoint][sharepoint-hybrid] &mdash; ou seja, estendendo o SharePoint Server 2016 para o Office 365 SharePoint Online. Se você tiver um Servidor do Office Online, consulte [Suporte para Aplicativos Web do Office e Servidor Online do Office no Azure][office-web-apps].

Os aplicativos de serviço padrão nesta implantação foram projetados para oferecer suporte a cargas de trabalho híbridas. Todas as cargas de trabalho híbridas do SharePoint Server 2016 e Office 365 podem ser implantadas para este farm sem alterações na infraestrutura do SharePoint, com uma exceção: O aplicativo de serviço de pesquisa de nuvem híbrida não deve ser implantado em servidores que hospedam uma topologia de pesquisa existente. Portanto, uma ou mais VMs com base em função de pesquisa devem ser adicionadas ao farm para oferecer suporte a esse cenário híbrido.

### <a name="sql-server-always-on-availability-groups"></a>Grupos de Disponibilidade Always On do SQL Server

Essa arquitetura usa máquinas virtuais do SQL Server porque o SharePoint Server 2016 não pode usar o Banco de Dados do SQL. Para dar suporte à alta disponibilidade no SQL Server, é recomendável usar Grupos de Disponibilidade AlwaysOn, que especificam um conjunto de bancos de dados que fazem failover juntos, tornando-os, altamente disponíveis e recuperáveis. Nessa arquitetura de referência, os bancos de dados são criados durante a implantação, mas você deve habilitar manualmente os Grupos de Disponibilidade Always On e adicionar os bancos de dados do SharePoint a um grupo de disponibilidade. Para saber mais, consulte [Criar o grupo de disponibilidade e adicionar os bancos de dados do SharePoint][create-availability-group].

Também é recomendável adicionar um endereço IP de ouvinte ao cluster, que é o endereço IP privado do balanceador de carga interno para máquinas virtuais do SQL Server.

Para saber os tamanhos de VM recomendados e outras recomendações de desempenho para o SQL Server em execução no Azure, consulte [Práticas recomendadas de desempenho para o SQL Server em máquinas virtuais do Azure][sql-performance]. Além disso, siga as recomendações em [Melhores práticas para o SQL Server em um farm do SharePoint Server 2016][sql-sharepoint-best-practices].

É recomendável que o servidor de nós principais resida em um computador separado dos parceiros de replicação. O servidor permite que o servidor parceiro de replicação secundária em uma sessão de modo de alta segurança reconheça se deve iniciar um failover automático. Ao contrário dos dois parceiros, o servidor de nós principais não atende o banco de dados, mas em vez disso, oferece suporte a failover automático. 

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Para dimensionar os servidores existentes, basta alterar o tamanho da VM. 

Com o recurso [MinRoles][minroles] no SharePoint Server 2016, você pode expandir servidores com base na função do servidor e também remover servidores de uma função. Quando você adiciona servidores a uma função, você pode especificar qualquer uma das funções individuais ou uma das funções combinadas. Se você adicionar servidores para a função de pesquisa, no entanto, você deverá também reconfigurar a topologia de pesquisa usando o PowerShell. Você também pode converter funções usando MinRoles. Para obter mais informações, consulte [Gerenciamento de um farm de servidores MinRole no SharePoint Server 2016][sharepoint-minrole].

Observe que o SharePoint Server 2016 não oferece suporte ao uso de conjuntos de escala de máquina virtual para o dimensionamento automático.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Essa arquitetura de referência oferece suporte à alta disponibilidade em uma região do Azure, porque cada função possui pelo menos duas VMs implantadas em um conjunto de disponibilidade.

Para proteger-se contra uma falha regional, crie um farm de recuperação de desastres separado em uma região do Azure diferente. Seus objetivos de tempo de recuperação (RTOs) e objetivos de ponto de recuperação (RPOs) determinarão os requisitos de configuração. Para obter detalhes, consulte [Escolher uma estratégia de recuperação de desastres para o SharePoint 2016][sharepoint-dr]. A região secundária deve ser uma *região emparelhada* com a região principal. No caso de uma interrupção ampla, é priorizada a recuperação de uma região de cada par. Para saber mais, consulte [Continuidade dos negócios e recuperação de desastres (BCDR): Regiões Emparelhadas do Azure][paired-regions].

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Para operar e manter servidores, farms de servidores e sites, siga as práticas recomendadas para operações do SharePoint. Para obter mais informações, consulte [Operações para o Sharepoint Server 2016][sharepoint-ops].

As tarefas a serem considerados no gerenciamento do SQL Server em um ambiente do SharePoint podem ser diferentes daquelas que normalmente são consideradas para um aplicativo de banco de dados. Uma prática recomendada é fazer backup total de todos os bancos de dados SQL semanalmente com backups incrementais todas as noites. Faça um backup de logs de transações a cada 15 minutos. Outra prática é implementar as tarefas de manutenção do SQL Server nos bancos de dados ao mesmo tempo em que aquelas que são internas do SharePoint são desativadas. Para obter mais informações, consulte [Configuração e planejamento de capacidade de armazenamento e do SQL Server][sql-server-capacity-planning]. 

## <a name="security-considerations"></a>Considerações de segurança

As contas de serviço de nível de domínio usadas para executar o SharePoint Server 2016 exigem controladores de domínio do AD do Windows Server para processos de autenticação e associação de domínio. O Azure Active Directory Domain Services não pode ser usado para essa finalidade. Para estender a infraestrutura de identidade do AD do Windows Server já em vigor na intranet, essa arquitetura usa dois controladores de domínio de réplica do AD do Windows Server de uma floresta do AD do Windows Server local existente.

Além disso, sempre é aconselhável planejar para se ter uma segurança mais alta. Outras recomendações incluem:

- Adicionar regras para os NSGs para isolar sub-redes e funções.
- Não atribuir endereços IP públicos para máquinas virtuais.
- Para a detecção de intrusão e a análise de conteúdo, considere o uso de uma solução de virtualização de rede na frente de servidores Web de front-end em vez de um balanceador de carga interno do Azure.
- Como opção, use as políticas de IPsec para criptografia de tráfego de texto não criptografado entre servidores. Se você também estiver fazendo o isolamento de sub-rede, atualize suas regras de grupo de segurança de rede para permitir o tráfego IPsec.
- Instale agentes antimalware nas VMs.

## <a name="deploy-the-solution"></a>Implantar a solução

Os scripts de implantação para essa arquitetura de referência estão disponíveis no [GitHub][github]. 

Você pode implantar essa arquitetura incrementalmente ou de uma só vez. Na primeira vez, é recomendável uma implantação incremental, para que você possa ver o que faz cada implantação. Especificar o incremento usando um dos seguintes parâmetros de *modo*.

| Mode           | O que faz                                                                                                            |
|----------------|-------------------------------------------------------------------------------------------------------------------------|
| onprem         | (Opcional) Implanta um ambiente de rede local simulado, para teste ou avaliação. Essa etapa não se conecta a uma rede local real. |
| infrastructure | Implanta a infraestrutura de rede do SharePoint 2016 e a jumpbox do Azure.                                                |
| createvpn      | Implanta um gateway de rede virtual em redes locais e do SharePoint e as conecta. Executar esta etapa somente se você executou a etapa `onprem`.                |
| workload       | Implanta os servidores do SharePoint na rede do SharePoint.                                                               |
| segurança       | Implanta o grupo de segurança de rede na rede do SharePoint.                                                           |
| tudo            | Implanta todas as implantações anteriores.                            


Para implantar a arquitetura de forma incremental com um ambiente de rede local simulado, execute as seguintes etapas na ordem:

1. onprem
2. infrastructure
3. createvpn
4. workload
5. segurança

Para implantar a arquitetura de forma incremental sem um ambiente de rede local simulado, execute as seguintes etapas na ordem:

1. infrastructure
2. workload
3. segurança

Para implantar tudo em uma única etapa, use `all`. Observe que todo o processo pode levar várias horas.

### <a name="prerequisites"></a>pré-requisitos

* Instale a versão mais recente do [Azure PowerShell][azure-ps].

* Antes de implantar essa arquitetura de referência, verifique se sua assinatura possui uma cota suficiente — pelo menos 38 núcleos. Se você não tem o suficiente, use o portal do Azure para enviar uma solicitação de suporte para mais cota.

* Para estimar o custo da implantação, consulte a [Calculadora de Preços do Azure][azure-pricing].

### <a name="deploy-the-reference-architecture"></a>Implantar a arquitetura de referência

1.  Baixe ou clone o [repositório GitHub][github] no computador local.

2.  Abra uma janela do PowerShell e navegue até a pasta `/sharepoint/sharepoint-2016`.

3.  Execute o comando do PowerShell a seguir. Para \<subscription-id:\>, use a ID de sua assinatura do Azure. Para \<location\>, especifique uma região do Azure, tais como `eastus` ou `westus`. Para \<modo\>, especifique `onprem`, `infrastructure`, `createvpn`, `workload`, `security` ou `all`.

    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```   
4. Quando solicitado, entre em sua conta do Azure. Os scripts de implantação podem demorar várias horas para serem concluídos, dependendo do modo selecionado.

5. Quando a implantação for concluída, execute os scripts para configurar os Grupos de Disponibilidade Always On do SQL Server. Consulte o [Leiame][readme] para obter detalhes.

> [!WARNING]
> Os arquivos de parâmetro incluem uma senha embutida (`AweS0me@PW`) em vários locais. Altere esses valores antes de fazer a implantação.


## <a name="validate-the-deployment"></a>Validar a implantação

Depois de implantar essa arquitetura de referência, os grupos de recursos a seguir são listados sob a assinatura que você usou:

| Grupo de recursos        | Finalidade                                                                                         |
|-----------------------|-------------------------------------------------------------------------------------------------|
| ra-onprem-sp2016-rg   | Rede local simulada com o Active Directory, federada com a rede do SharePoint 2016 |
| ra-sp2016-network-rg  | Infraestrutura para dar suporte à implantação do SharePoint                                                 |
| ra-sp2016-workload-rg | Recursos de suporte e SharePoint                                                             |

### <a name="validate-access-to-the-sharepoint-site-from-the-on-premises-network"></a>Validar o acesso ao site do SharePoint a partir da rede local

1. No [portal do Azure][azure-portal], em **Grupos de recursos**, selecione o grupo de recursos `ra-onprem-sp2016-rg`.

2. Na lista de recursos, selecione o recurso de VM chamado `ra-adds-user-vm1`. 

3. Conecte-se à VM, conforme descrito em [Conectar-se à máquina virtual][connect-to-vm]. O nome de usuário é `\onpremuser`.

5.  Quando a conexão remota com a máquina virtual for estabelecida, abra um navegador na VM e navegue até `http://portal.contoso.local`.

6.  Na caixa **Segurança do Windows**, faça logon no portal do SharePoint usando `contoso.local\testuser` para o nome de usuário.

Esse logon faz um túnel do domínio Fabrikam.com usado pela rede local até o domínio contoso.local usado pelo portal do SharePoint. Quando o site do SharePoint abre, você verá o site de demonstração de raiz.

### <a name="validate-jumpbox-access-to-vms-and-check-configuration-settings"></a>Validar acesso à jumpbox para VMs e verificar as definições de configuração

1.  No [portal do Azure][azure-portal], em **Grupos de recursos**, selecione o grupo de recursos `ra-sp2016-network-rg`.

2.  Na lista de recursos, selecione o recurso de VM chamado `ra-sp2016-jb-vm1`, que é a jumpbox.

3. Conecte-se à VM, conforme descrito em [Conectar-se à máquina virtual][connect-to-vm]. O nome de usuário é `testuser`.

4.  Depois que você fizer logon na jumpbox, abra uma sessão RDP a partir da jumpbox. Conecte-se a outras VMs na rede virtual. O nome de usuário é `testuser`. Você pode ignorar o aviso sobre o certificado de segurança do computador remoto.

5.  Quando a conexão remota para a máquina virtual abrir, examine a configuração e faça alterações usando as ferramentas administrativas, tais como o Gerenciador do Servidor.

A tabela a seguir mostra as VMs que são implantadas. 

| Nome do Recurso      | Finalidade                                   | Grupo de recursos        | Nome da VM                       |
|--------------------|-------------------------------------------|-----------------------|-------------------------------|
| Ra-sp2016-ad-vm1   | Active Directory + DNS                    | Ra-sp2016-network-rg  | Ad1.contoso.local             |
| Ra-sp2016-ad-vm2   | Active Directory + DNS                    | Ra-sp2016-network-rg  | Ad2.contoso.local             |
| Ra-sp2016-fsw-vm1  | SharePoint                                | Ra-sp2016-network-rg  | Fsw1.contoso.local            |
| Ra-sp2016-jb-vm1   | Jumpbox                                   | Ra-sp2016-network-rg  | Jb (use o IP público para fazer logon) |
| Ra-sp2016-sql-vm1  | SQL Always On - Failover                  | Ra-sp2016-network-rg  | Sq1.contoso.local             |
| Ra-sp2016-sql-vm2  | SQL Always On - Primário                   | Ra-sp2016-network-rg  | Sq2.contoso.local             |
| Ra-sp2016-app-vm1  | MinRole de aplicativo do SharePoint 2016       | Ra-sp2016-workload-rg | App1.contoso.local            |
| Ra-sp2016-app-vm2  | MinRole de aplicativo do SharePoint 2016       | Ra-sp2016-workload-rg | App2.contoso.local            |
| Ra-sp2016-dch-vm1  | MinRole de Cache distribuído do SharePoint 2016 | Ra-sp2016-workload-rg | Dch1.contoso.local            |
| Ra-sp2016-dch-vm2  | MinRole de Cache distribuído do SharePoint 2016 | Ra-sp2016-workload-rg | Dch2.contoso.local            |
| Ra-sp2016-srch-vm1 | MinRole de pesquisa do SharePoint 2016            | Ra-sp2016-workload-rg | Srch1.contoso.local           |
| Ra-sp2016-srch-vm2 | MinRole de pesquisa do SharePoint 2016            | Ra-sp2016-workload-rg | Srch2.contoso.local           |
| Ra-sp2016-wfe-vm1  | MinRole de front-end da Web do SharePoint 2016     | Ra-sp2016-workload-rg | Wfe1.contoso.local            |
| Ra-sp2016-wfe-vm2  | MinRole de front-end da Web do SharePoint 2016     | Ra-sp2016-workload-rg | Wfe2.contoso.local            |


**_Colaboradores dessa arquitetura de referência_** &mdash; Joe Davies, Bob Fox, Neil Hodgkinson, Paul Stork

<!-- links -->

[availability-set]: /azure/virtual-machines/windows/manage-availability
[azure-portal]: https://portal.azure.com
[azure-ps]: /powershell/azure/overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[bastion-host]: https://en.wikipedia.org/wiki/Bastion_host
[create-availability-group]: https://technet.microsoft.com/library/mt793548(v=office.16).aspx
[connect-to-vm]: /azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine
[github]: https://github.com/mspnp/reference-architectures
[hybrid-ra]: ../hybrid-networking/index.md
[hybrid-vpn-ra]: ../hybrid-networking/vpn.md
[load-balancer]: /azure/load-balancer/load-balancer-internal-overview
[managed-disks]: /azure/storage/storage-managed-disks-overview
[minroles]: https://technet.microsoft.com/library/mt346114(v=office.16).aspx
[nsg]: /azure/virtual-network/virtual-networks-nsg
[office-web-apps]: https://support.microsoft.com/help/3199955/office-web-apps-and-office-online-server-supportability-in-azure
[paired-regions]: /azure/best-practices-availability-paired-regions
[readme]: https://github.com/mspnp/reference-architectures/tree/master/sharepoint/sharepoint-2016
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[quotas]: /azure/azure-subscription-service-limits
[sharepoint-accounts]: https://technet.microsoft.com/library/ee662513(v=office.16).aspx
[sharepoint-crawling]: https://technet.microsoft.com/library/dn535606(v=office.16).aspx
[sharepoint-dr]: https://technet.microsoft.com/library/ff628971(v=office.16).aspx
[sharepoint-hybrid]: https://aka.ms/sphybrid
[sharepoint-minrole]: https://technet.microsoft.com/library/mt743705(v=office.16).aspx
[sharepoint-ops]: https://technet.microsoft.com/library/cc262289(v=office.16).aspx
[sharepoint-reqs]: https://technet.microsoft.com/library/cc262485(v=office.16).aspx
[sharepoint-search]: https://technet.microsoft.com/library/dn342836.aspx
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server
[sql-performance]: /virtual-machines/windows/sql/virtual-machines-windows-sql-performance
[sql-server-capacity-planning]: https://technet.microsoft.com/library/cc298801(v=office.16).aspx
[sql-quorum]: https://technet.microsoft.com/library/cc731739(v=ws.11).aspx
[sql-sharepoint-best-practices]: https://technet.microsoft.com/library/hh292622(v=office.16).aspx
[tempdb]: /sql/relational-databases/databases/tempdb-database
[virtual-networks-nsg]: /azure/virtual-network/virtual-networks-nsg
[visio-download]: https://archcenter.blob.core.windows.net/cdn/Sharepoint-2016.vsdx
[vm-sizes-general]: /azure/virtual-machines/windows/sizes-general
[vm-sizes-memory]: /azure/virtual-machines/windows/sizes-memory
[windows-n-tier]: ../virtual-machines-windows/n-tier.md
