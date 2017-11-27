---
title: Implantar o SAP NetWeaver e o SAP HANA no Azure
description: "Práticas comprovadas para executar o SAP HANA em um ambiente de alta disponibilidade no Azure."
author: njray
ms.date: 06/29/2017
ms.openlocfilehash: 27a97103c0c6f305cb8e830d670c8d0ba7e22aa5
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="deploy-sap-netweaver-and-sap-hana-on-azure"></a>Implantar o SAP NetWeaver e o SAP HANA no Azure

Essa arquitetura de referência mostra um conjunto de práticas comprovadas para executar o SAP HANA em um ambiente de alta disponibilidade no Azure. [**Implantar esta solução**.](#deploy-the-solution)

![0][0]

*Baixar um [arquivo Visio][visio-download] dessa arquitetura.*

> [!NOTE]
> Implantar essa arquitetura de referência requer um licenciamento apropriado de produtos SAP e outras tecnologias que não são da Microsoft. Para obter informações sobre a parceria entre a Microsoft e o SAP, consulte [SAP HANA no Azure][sap-hana-on-azure].

## <a name="architecture"></a>Arquitetura

Essa arquitetura consiste nos seguintes componentes.

- **Rede virtual (VNet)**. Uma rede virtual é uma representação de uma rede isolada logicamente no Azure. Todas as VMs nesta arquitetura de referência são implantadas para a mesma rede virtual. A rede virtual é, posteriormente, subdividida em sub-redes. Crie uma sub-rede separada para cada camada, incluindo o aplicativo (SAP NetWeaver), o banco de dados (SAP HANA), gerenciamento (o jumpbox) e o Active Directory.

- **Máquinas Virtuais (VMs)**. As VMs dessa arquitetura são agrupadas em vários camadas distintas.

    - **SAP NetWeaver**. Inclui SAP ASCS, SAP Web Dispatcher e os servidores de aplicativos SAP. 
    
    - **SAP HANA**. Essa arquitetura de referência usa o SAP HANA para a camada de banco de dados, em execução em uma única instância [GS5][vm-sizes-mem] instância. O SAP HANA é certificado para cargas de trabalho de produção OLAP em GS5 ou [HANA SAP no Azure grandes instâncias][azure-large-instances]. Essa arquitetura de referência é para máquinas virtuais do Azure na série G e série M. Para obter informações sobre o SAP HANA Instâncias Grandes do Azure, consulte [Visão geral do SAP HANA (grandes instâncias) e arquitetura no Azure][azure-large-instances].
   
    - **Jumpbox**. Também chamada de um host bastião. Esta é uma VM segura na rede que os administradores usam para se conectar às outras VMs. 
     
    - **Controladores de domínio do Active Directory (AD) do Windows Server.** Os controladores de domínio são usados para configurar o Cluster de Failover do Windows Server (veja abaixo).
 
- **Conjuntos de disponibilidade**. Coloque as máquinas virtuais para as funções de SAP Web Dispatcher, servidor de aplicativos SAP e SAP Web Dispatcher em conjuntos de disponibilidade separados e provisione pelo menos duas VMs para cada função. Isso torna as VMs qualificadas para um contrato de nível de serviço (SLA) mais alto.
    
- **NICs.** As máquinas virtuais que executam o SAP NetWeaver e o SAP HANA exigem duas interfaces de rede (NICs). Cada NIC é atribuído a uma sub-rede diferente, para separar os diferentes tipos de tráfego. Consulte as [Recomendações](#recommendations) abaixo para obter mais informações.

- **Cluster de Failover do Windows Server**. As VMs que executam ACSC SAP são configuradas como um cluster de failover para alta disponibilidade. Para dar suporte ao cluster de failover, a SIOS DataKeeper Cluster Edition executa a função de volume de cluster compartilhado (CSV) replicando discos independentes pertencentes a nós de cluster. Para obter detalhes, consulte [Executar aplicativos do SAP na plataforma da Microsoft][running-sap].
    
- **Balanceadores de carga.** Duas instâncias do [Azure Load Balancer][azure-lb] são usadas. A primeira, mostrado à esquerda do diagrama, distribui o tráfego para VMs do SAP Web Dispatcher. Essa configuração implementa a opção de dispatcher Web paralelo descrita em [Alta Disponibilidade do SAP Web Dispatcher][sap-dispatcher-ha]. O segundo balanceador de carga, exibido à direita, permite o failover no Cluster de Failover do Windows Server, direcionando as conexões de entrada para o nó ativo/íntegro.

- **Gateway de VPN.** Um Gateway de VPN estende sua rede local para a rede virtual do Azure. Você também pode usar o ExpressRoute, que usa uma conexão privada dedicada que não passa pela Internet pública. A solução de exemplo não implanta o gateway. Para obter mais informações, consulte [Conectar uma rede local ao Azure][hybrid-networking].

## <a name="recommendations"></a>Recomendações

Seus requisitos podem ser diferentes dos requisitos da arquitetura descrita aqui. Use essas recomendações como ponto de partida.

### <a name="load-balancers"></a>Balanceadores de carga

O [SAP Web Dispatcher][sap-dispatcher] lida com o balanceamento de carga do tráfego HTTP(S) para servidores de pilha dupla (ABAP e Java). A empresa SAP tem defendido servidores de aplicativos de pilha única por anos, então muito poucos aplicativos são executados em um modelo de implantação de pilha dupla hoje em dia. O balanceador de carga do Azure mostrado no diagrama da arquitetura implementa o cluster de alta disponibilidade para o SAP Web Dispatcher.

O balanceamento de carga de tráfego para os servidores de aplicativo é manipulado no SAP. Para o tráfego de clientes SAPGUI conectando-se a um servidor SAP por meio de DIAG e Remote Function Calls (RFC), o servidor de mensagens SCS equilibra a carga por meio da criação do Servidor de Aplicativos SAP [Grupos de Logon][logon-groups]. 

SMLG é uma transação SAP ABAP usada para gerenciar o recurso de balanceamento de carga de logon do SAP Central Services. O pool de back-end do grupo de logon tem mais de um servidor de aplicativo ABAP. Clientes que acessam os serviços de cluster ASCS conectam-se ao balanceador de carga do Azure por meio de um endereço IP de front-end. O nome de rede virtual do cluster ASCS também possui um endereço IP. Opcionalmente, esse endereço pode ser associado a um endereço IP adicional no balanceador de carga do Azure, para que o cluster possa ser gerenciado remotamente.  

### <a name="nics"></a>NICs

As funções de gerenciamento de paisagem SAP exigem diferenciação de tráfego do servidor nas diferentes NICs. Por exemplo, dados de negócios devem ser separados de tráfego administrativo e de backup. Atribuir várias NICs em diferentes sub-redes permite a diferenciação de dados. Para obter mais informações, consulte "Rede" em [Criar Alta Disponibilidade para o SAP NetWeaver e o SAP HANA][sap-ha] (PDF).

Atribua a NIC de administração à sub-rede de gerenciamento e atribua a NIC de comunicação de dados a uma sub-rede separada. Para obter detalhes sobre a configuração, consulte [Criar e gerencia uma máquina virtual do Windows que tem várias NICs][multiple-vm-nics].

### <a name="azure-storage"></a>Armazenamento do Azure

Recomendamos usar o Armazenamento Premium do Azure para latência de leitura/gravação consistente em todas as VMs do servidor de banco de dados. Para servidores de aplicativos SAP, incluindo as máquinas virtuais (A)SCS, você pode usar o Armazenamento Padrão do Azure, já que a execução do aplicativo ocorre na memória e utiliza discos somente para registro em log.

Para melhor confiabilidade, é recomendável usar o [Azure Managed Disks][managed-disks]. Os discos gerenciados garantem que os discos para VMs dentro de um conjunto de disponibilidade são isolados para evitar pontos únicos de falha.

> [!NOTE]
> Atualmente o modelo do Resource Manager para esta arquitetura de referência não utiliza discos gerenciados. Estamos planejando atualizar o modelo para usar discos gerenciados.

Para obter IOPS e taxa de transferência do disco da largura de banda altos, as práticas comuns na otimização de desempenho do volume de armazenamento se aplicam ao layout de armazenamento do Azure. Por exemplo, a distribuição de vários discos em conjunto para criar um volume de disco maior melhora o desempenho de E/S. Habilitar o cache de leitura do conteúdo de armazenamento que raramente é alterado melhora a velocidade de recuperação de dados. Para obter detalhes sobre os requisitos de desempenho, consulte [Nota SAP 1943937 - Ferramenta de verificação de configuração de hardware][sap-1943937].

Para o armazenamento de dados de backup, é recomendável usar a [camada de armazenamento esporádico][cool-blob-storage] do Armazenamento de Blobs do Azure. A camada de armazenamento esporádico é uma forma econômica para armazenar dados que são acessados com menos frequência e de longa duração.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Na camada de aplicativo SAP, o Azure oferece uma ampla variedade de tamanhos de máquina virtual para expansão. Para obter uma lista inclusiva, consulte [Nota SAP 1928533 - Aplicativos SAP no Azure: Tipos de VM do Azure e produtos com suporte][sap-1928533]. Faça a expansão adicionando mais VMs ao conjunto de disponibilidade.

Para o SAP HANA em máquinas virtuais do Azure com aplicativos OLTP e OLAP SAP, o tamanho da máquina de virtual com certificação SAP é GS5 com uma única instância VM. Para cargas de trabalho maiores, a Microsoft também oferece [Grandes Instâncias do Azure][azure-large-instances] para SAP HANA em servidores físicos co-localizados em um datacenter certificado do Microsoft Azure, que fornece até 4 TB de capacidade de memória para uma única instância no momento. Uma configuração de vários nós também é possível com uma capacidade de memória total de até 32 TB.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Nesta instalação distribuída do aplicativo SAP em um banco de dados centralizado, a instalação básica é replicada para alcançar alta disponibilidade. Para cada camada da arquitetura, o design de alta disponibilidade varia:

- **Web Dispatcher.** A alta disponibilidade é obtida com instâncias redundantes do SAP Web Dispatcher com o tráfego do aplicativo SAP. Consulte [SAP Web Dispatcher][swd] na documentação do SAP.

- **ASCS.** Para obter uma alta disponibilidade do ASCS em máquinas virtuais do Windows Azure, o Clustering de Failover do Windows Server é usado com o SIOS DataKeeper para implementar o volume compartilhado de cluster. Para obter detalhes de implementação, consulte [Clustering do SAP ASCS no Azure][clustering].

- **Servidores de aplicativos.** A alta disponibilidade é obtida pelo tráfego de balanceamento de carga dentro de um pool de servidores de aplicativos.

- **Camada de banco de dados.** Essa arquitetura de referência implanta uma única instância de banco de dados do SAP HANA. Para obter alta disponibilidade, faça a implantação de mais de uma instância e usar a Replicação de Sistema do HANA (HSR) para implementar o failover manual. Para habilitar o failover automático, uma extensão de alta disponibilidade para a distribuição de Linux específica é necessária.

### <a name="disaster-recovery-considerations"></a>Considerações de recuperação de desastres

Cada camada usa uma estratégia diferente para fornecer proteção de recuperação de desastres.

- **Servidores de aplicativos.** Servidores de aplicativos SAP não contêm dados corporativos. No Azure, uma estratégia simples de recuperação de desastres é criar servidores de aplicativos SAP em outra região. Após quaisquer alterações de configuração ou atualizações de kernel no servidor de aplicativo primário, as mesmas alterações devem ser copiadas para VMs na região de recuperação de desastres. Por exemplo, os executáveis de kernel copiados para as VMs de recuperação de desastres.

- **SAP Central Services.** Este componente da pilha do aplicativo SAP também não mantém dados corporativos. Você pode criar uma VM na região de recuperação de desastres para executar a função SCS. O único conteúdo do nó SCS primário a ser sincronizado é o conteúdo compartilhado **/sapmnt**. Além disso, se as alterações de configuração ou atualizações de kernel ocorrem em servidores do SCS primários, elas devem ser repetidas no SCS de recuperação de desastres. Para sincronizar os dois servidores, basta usar um trabalho de cópia agendado regularmente para copiar **/sapmnt** para o lado de recuperação de desastres. Para obter detalhes sobre a criação, cópia e o processo de failover de teste, baixe [SAP NetWeaver: Criando uma solução de recuperação de desastres baseada em Hyper-V e no Microsoft Azure][sap-netweaver-dr] e consulte "4.3. Camada SAP SPOF (ASCS)."

- **Camada de banco de dados.** Use soluções de replicação com suporte HANA como HSR ou Replicação de Armazenamento. 

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

O SAP HANA possui um recurso de backup que usa a infraestrutura subjacente do Azure. Para fazer backup do banco de dados do SAP HANA em execução em máquinas virtuais do Azure, o instantâneo do armazenamento do Azure e o instantâneo do SAP HANA são usados para garantir a consistência dos arquivos de backup. Para obter detalhes, consulte o [Guia de backup para o SAP HANA em máquinas virtuais do Azure][hana-backup] e [Perguntas frequentes sobre o serviço de Backup do Azure][backup-faq].

O Azure fornece várias funções para [monitoramento e diagnóstico][monitoring] da infra-estrutura geral. Além disso, o monitoramento avançado de máquinas virtuais do Azure (Linux ou Windows) é tratado pelo Azure Operations Management Suite (OMS).

Para fornecer monitoramento de recursos baseado em SAP e desempenho de serviço da infraestrutura do SAP, use a extensão de monitoramento aprimorado do SAP do Azure. Essa extensão fornece as estatísticas de monitoramento do Azure para o aplicativo SAP para monitoramento do sistema operacional e as funções da ferramenta cockpit de DBA. 

## <a name="security-considerations"></a>Considerações de segurança

O SAP tem seu próprio mecanismo de gerenciamento de usuários (UME) para controlar o acesso e a autorização baseados em função do aplicativo SAP. Para obter detalhes, consulte [Segurança do SAP HANA - Uma visão geral][sap-security]. (Uma conta do SAP Service Marketplace é necessária para o acesso.)

Para segurança da infraestrutura, os dados são protegidos em trânsito e em repouso. A seção "Considerações sobre segurança" do [SAP NetWeaver nas máquinas virtuais (VMs) do Azure – Guia de planejamento e implementação][netweaver-on-azure] começa a tratar da segurança de rede. O guia também especifica as portas de rede que você deve abrir nos firewalls para permitir a comunicação do aplicativo. 

Para criptografar os discos de máquina virtual do IaaS Windows e Linux, você pode usar o [Azure Disk Encryption][disk-encryption]. O Azure Disk Encryption usa o recurso BitLocker do Windows e o recurso DM-Crypt do Linux para fornecer uma criptografia de volume para o sistema operacional e os discos de dados. A solução também funciona com o Azure Key Vault para ajudá-lo a controlar e gerenciar os segredos e chaves de criptografia de disco em sua assinatura do Key Vault. Os dados em discos da máquina virtual são criptografados em repouso no armazenamento do Azure.

Para criptografia de dados em repouso do SAP HANA, é recomendável usar a tecnologia de criptografia nativa do SAP HANA.

> [!NOTE]
> Não use a criptografia de dados em repouso HANA com criptografia de disco do Azure no mesmo servidor.

Considere o uso de [grupos de segurança de rede][nsg] (NSGs) para restringir o tráfego entre as várias sub-redes na rede virtual.

## <a name="deploy-the-solution"></a>Implantar a solução 

Os scripts de implantação para essa arquitetura de referência estão disponíveis no [GitHub][github].


### <a name="prerequisites"></a>Pré-requisitos

- Você deve ter acesso ao Centro de Download de Software SAP para concluir a instalação.
 
- Instale a versão mais recente do [Azure PowerShell][azure-ps]. 

- Essa implantação exige 51 núcleos. Antes de implantar, verifique se sua assinatura tem cota suficiente de núcleos VM. Caso não tenha, use o portal do Azure para enviar uma solicitação de suporte para mais cota.
 
- Essa implantação usa a máquina virtual da série GS. Verifique a disponibilidade da série GS por região [aqui][region-availability].

- Para estimar o custo da implantação, consulte a [Calculadora de Preços do Azure][azure-pricing]. 
 
Essa arquitetura de referência implanta as máquinas virtuais a seguir:

| Nome do recurso | Tamanho da VM | Finalidade  |
|---------------|---------|----------|
| `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN` | DS11v2 | SAP Central Services |
| `ra-sapApps-vm1` ... `ra-sapApps-vmN` | DS11v2 | Aplicativo SAP NetWeaver |
| `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN` | DS11v2 | SAP Web Dispatcher |
| `ra-sap-data-vm1` | GS5 | Instância de banco de dados do SAP HANA |
| `ra-sap-jumpbox-vm1` | DS1V2 | Jumpbox |

Uma única instância do SAP HANA é implantada. Para as VMs de aplicativo, o número de instâncias a implantar é especificado nos parâmetros do modelo.

### <a name="deploy-sap-infrastructure"></a>Implantar a infraestrutura SAP

Você pode implantar essa arquitetura incrementalmente ou de uma só vez. Na primeira vez, é recomendável uma implantação incremental, para que você possa ver o que cada etapa da implantação faz. Especificar o incremento usando um dos seguintes parâmetros de *modo*

| Modo           | O que faz                                                                                                            |
|----------------|-----------------------------------------------------|
| infrastructure | Definir a infraestrutura de rede no Azure.        |
| workload       | Implanta os servidores SAP na rede.             |
| tudo            | Implanta todas as implantações anteriores.              |

Para implantar a solução, execute as etapas a seguir:

1. Baixe ou clone o [repositório GitHub][github] no computador local.

2. Abra uma janela do PowerShell e navegue até a pasta `/sap/sap-hana/`.

3. Execute o seguinte cmdlet do PowerShell. Para `<subscription id>`, insira sua ID da assinatura do Azure. Para `<location>`, especifique uma região do Azure, tal como `eastus` ou `westus`. Para `<mode>`, especifique um dos modos listados acima.

    ```powershell
     .\Deploy-ReferenceArchitecture -SubscriptionId <subscription id> -Location <location> -ResourceGroupName <resource group> <mode>
    ```

4.  Quando solicitado, entre em sua conta do Azure. 

Os scripts de implantação podem demorar várias horas para serem concluídos, dependendo do modo selecionado.

> [!WARNING]
> Os arquivos de parâmetro incluem uma senha embutida (`AweS0me@PW`) em vários locais. Altere esses valores antes de fazer a implantação.
 
### <a name="configure-sap-applications-and-database"></a>Configurar o banco de dados e aplicativos SAP

Depois de implantar a infraestrutura do SAP, instale e configure seus aplicativos do SAP e o banco de dados do HANA nas máquinas virtuais, conforme a seguir.

> [!NOTE]
> Para obter instruções de instalação do SAP, você deve ter um nome de usuário do Portal de suporte do SAP e uma senha para baixar o [guias de instalação do SAP][sap-guide].

1. Faça logon na jumpbox (`ra-sap-jumpbox-vm1`). Você usará o jumpbox para fazer logon em outras VMs. 

2.  Para cada VM denominada `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN`, faça logon na VM e instale e configure a instância do SAP Web Dispatcher usando as etapas descritas no wiki [Instalação do Web Dispatcher][sap-dispatcher-install].

3.  Faça logon na VM denominada `ra-sap-data-vm1`. Instalar e configurar a instância de banco de dados do SAP HANA usando o [Guia de atualização e instalação do servidor do SAP HANA][hana-guide].

4. Para cada VM denominada `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN`, faça logon na VM e instale e configure o SAP Central Services (SCS) usando os [guias de instalação do SAP][sap-guide].

5.  Para cada VM denominada `ra-sapApps-vm1` ... `ra-sapApps-vmN`, faça logon na VM e instale e configure o aplicativo SAP NetWeaver usando os [guias de instalação do SAP][sap-guide].

**_Colbaoradores desta arquitetura de referência_**&mdash; Rick Rainey Ross Sponholtz, Ben Trinh

[azure-large-instances]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[azure-lb]: /azure/load-balancer/load-balancer-overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[azure-ps]: /powershell/azure/overview
[backup-faq]: /azure/backup/backup-azure-backup-faq
[clustering]: https://blogs.msdn.microsoft.com/saponsqlserver/2015/05/20/clustering-sap-ascs-instance-using-windows-server-failover-cluster-on-microsoft-azure-with-sios-datakeeper-and-azure-internal-load-balancer/
[cool-blob-storage]: /azure/storage/storage-blob-storage-tiers
[disk-encryption]: /azure/security/azure-security-disk-encryption
[github]: https://github.com/mspnp/reference-architectures/tree/master/sap/sap-hana
[hana-backup]: /azure/virtual-machines/workloads/sap/sap-hana-backup-guide
[hana-guide]: https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.01/en-US/7eb0167eb35e4e2885415205b8383584.html
[hybrid-networking]: ../hybrid-networking/index.md
[logon-groups]: https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing
[managed-disks]: /azure/storage/storage-managed-disks-overview
[monitoring]: /azure/architecture/best-practices/monitoring
[multiple-vm-nics]: /azure/virtual-machines/windows/multiple-nics
[netweaver-on-azure]: /azure/virtual-machines/workloads/sap/planning-guide
[nsg]: /azure/virtual-network/virtual-networks-nsg
[region-availability]: https://azure.microsoft.com/regions/services/
[running-SAP]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/06/07/sap-on-sql-general-update-for-customers-partners-june-2016/
[sap-1943937]: https://launchpad.support.sap.com/#/notes/1943937
[sap-1928533]: https://launchpad.support.sap.com/#/notes/1928533
[sap-dispatcher]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm
[sap-dispatcher-ha]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/9a9a6b48c673e8e10000000a42189b/frameset.htm
[sap-dispatcher-install]: https://wiki.scn.sap.com/wiki/display/SI/Web+Dispatcher+Installation
[sap-guide]: https://service.sap.com/instguides
[sap-ha]: https://support.sap.com/content/dam/SAAP/SAP_Activate/AGS_70.pdf
[sap-hana-on-azure]: https://azure.microsoft.com/services/virtual-machines/sap-hana/
[sap-netweaver-dr]: http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx
[sap-security]: https://archive.sap.com/documents/docs/DOC-62943
[visio-download]: https://archcenter.azureedge.net/cdn/SAP-HANA-architecture.vsdx
[vm-sizes-mem]: /azure/virtual-machines/windows/sizes-memory
[swd]: https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm
[0]: ./images/sap-hana.png "Arquitetura SAP HANA usando Microsoft Azure"
