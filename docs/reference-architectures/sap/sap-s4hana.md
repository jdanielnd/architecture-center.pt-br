---
title: SAP S/4HANA para Máquinas Virtuais do Linux no Azure
titleSuffix: Azure Reference Architectures
description: Práticas comprovadas para executar o SAP S/4HANA em um ambiente Linux no Azure com alta disponibilidade.
author: lbrader
ms.date: 05/11/2018
ms.custom: seodec18
ms.openlocfilehash: 9eb73ddaf5b1cb815f037f46c7e187f61d126876
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644165"
---
# <a name="sap-s4hana-for-linux-virtual-machines-on-azure"></a>SAP S/4HANA para máquinas virtuais com Linux no Azure

Essa arquitetura de referência mostra um conjunto de práticas comprovadas para executar o S/4HANA em um ambiente de alta disponibilidade no Azure que dê suporte à recuperação de desastre no Azure. Essa arquitetura é implantada com tamanhos específicos de VM (máquina virtual) que podem ser alterados para acomodar as necessidades da sua organização.

![Arquitetura de referência para SAP S/4HANA para máquinas virtuais com Linux no Azure](./images/sap-s4hana.png)

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

> [!NOTE]
> Implantar essa arquitetura de referência requer um licenciamento apropriado de produtos SAP e outras tecnologias que não são da Microsoft.

## <a name="architecture"></a>Arquitetura

Essa arquitetura de referência descreve um nível empresarial, o sistema de nível de produção. De acordo com a necessidade da sua empresa, essa configuração poderá ser reduzida para uma única máquina virtual. No entanto, os seguintes componentes são necessários:

**Rede virtual**. O serviço [Rede Virtual do Azure](/azure/virtual-network/virtual-networks-overview) conecta com segurança os recursos do Azure entre si. Nessa arquitetura, a rede virtual se conecta a um ambiente local por meio de um gateway implantado no hub de uma [tecnologia hub-spoke](../hybrid-networking/hub-spoke.md). O spoke é a rede virtual usada para os aplicativos SAP.

**Sub-redes**. A rede virtual é subdividida em [sub-redes](/azure/virtual-network/virtual-network-manage-subnet) separadas para cada camada: gateway, aplicativo, banco de dados e serviços compartilhados.

**Máquinas virtuais**. Essa arquitetura usa máquinas virtuais que executam Linux para a camada de aplicativo e a camada de banco de dados, agrupadas da seguinte maneira:

- **Camada de aplicativo**. Inclui o pool de Servidores de Front-end Fiori, o pool do SAP Web Dispatcher, o pool de servidores de aplicativo e o cluster do SAP Central Services. Para alta disponibilidade do Central Services nas Máquinas Virtuais do Linux do Azure, é necessário um serviço NFS (Network File System) altamente disponível.
- **Cluster NFS**. Essa arquitetura usa um servidor [NFS](/azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs) em execução em um cluster Linux para armazenar dados compartilhados entre os sistemas SAP. Este cluster centralizado pode ser compartilhado por vários sistemas SAP. Para alta disponibilidade do serviço NFS, é usada a Extensão de Alta Disponibilidade apropriada para a distribuição de Linux selecionada.
- **SAP HANA**. A camada de banco de dados usa duas ou mais máquinas virtuais do Linux em um cluster para obter alta disponibilidade. A HSR (Replicação de Sistema do HANA) é usada para replicar conteúdo entre os sistemas HANA primários e secundários. O clustering do Linux é usado para detectar falhas de sistema e facilitar o failover automático. Um mecanismo de isolamento com base em nuvem ou em armazenamento pode ser usado para garantir que o sistema com falha seja isolado ou desligado para evitar a condição de separação de cluster.
- **Jumpbox**. Também chamada de um host bastião. Essa é uma máquina virtual segura na rede que os administradores usam para se conectar às outras máquinas virtuais. Ele pode executar o Windows ou o Linux. Use um jumpbox do Windows para a conveniência de navegação na Web ao usar as ferramentas de gerenciamento do HANA Cockpit ou do HANA Studio.

**Balanceadores de carga**. Tanto os balanceadores de carga internos do SAP quanto o [Azure Load Balancer](/azure/load-balancer/load-balancer-overview) são usados para obter a alta disponibilidade. As instâncias do Azure Load Balancer são usadas para distribuir o tráfego para máquinas virtuais na sub-rede de camada de aplicativo.

**Conjuntos de disponibilidade**. As máquinas virtuais para todos os pools e clusters (Web Dispatcher, servidores de aplicativos SAP, Central Services, NFS e HANA) são agrupadas em [conjuntos de disponibilidades](/azure/virtual-machines/windows/tutorial-availability-sets) separados, e pelo menos duas máquinas virtuais são provisionadas por função. Isso torna as máquinas virtuais qualificadas para um [SLA (Contrato de Nível de Serviço)](https://azure.microsoft.com/support/legal/sla/virtual-machines) mais elevado.

**NICs**. Os [cartões de interface de rede](/azure/virtual-network/virtual-network-network-interface) (NICs) permitem toda a comunicação de máquinas virtuais em uma rede virtual.

**Grupos de segurança de rede**. Para restringir o tráfego de entrada, saída e dentro da sub-rede na rede virtual, é possível criar [grupos de segurança de rede](/azure/virtual-network/virtual-networks-nsg) (NSGs).

**Gateway**. Um gateway estende sua rede local para a rede virtual do Azure. O [ExpressRoute](/azure/architecture/reference-architectures/hybrid-networking/expressroute) é o serviço do Azure recomendado para criar conexões privadas que não passem pela Internet pública, mas uma conexão [Site a Site](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal) também pode ser usada.

**Armazenamento do Azure**. Para fornecer um armazenamento persistente de um VHD (disco rígido virtual) de uma máquina virtual, é necessário o [Armazenamento do Azure](/azure/storage/).

## <a name="recommendations"></a>Recomendações

Essa arquitetura descreve a implantação de uma empresa com um nível de produção pequeno. Sua implantação será diferente de acordo com suas necessidades de negócios. Use essas recomendações como ponto de partida.

### <a name="virtual-machines"></a>Máquinas virtuais

Nos pools e clusters de servidor de aplicativos, ajuste o número de máquinas virtuais com base nos seus requisitos. O [guia de planejamento e implementação de Máquinas Virtuais do Azure](/azure/virtual-machines/workloads/sap/planning-guide) inclui detalhes sobre a execução do SAP NetWeaver em máquinas virtuais, mas as informações também se aplicam ao SAP S/4HANA.

Para obter detalhes sobre o suporte da SAP para tipos de máquina virtual do Azure e métricas de taxa de transferência (SAPS), consulte [Nota SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533).

### <a name="sap-web-dispatcher-pool"></a>Pool do SAP Web Dispatcher

O componente Web Dispatcher é usado como um balanceador de carga para tráfego da SAP entre os servidores de aplicativos SAP. Para obter alta disponibilidade para o componente Web Dispatcher, o Azure Load Balancer será usado para implementar a configuração paralela do Web Dispatcher em uma configuração de round-robin para distribuição de tráfego de HTTP(S) entre os Web Dispatchers disponíveis no pool de back-end dos balanceadores.

### <a name="fiori-front-end-server"></a>Servidor Front-end Fiori

O Servidor de Front-end Fiori usa um [NetWeaver Gateway](https://help.sap.com/doc/saphelp_gateway20sp12/2.0/en-US/76/08828d832e4aa78748e9f82204a864/content.htm?no_cache=true). Para implantações pequenas, ele pode ser carregado no servidor Fiori. Para grandes implantações, um servidor separado para o NetWeaver Gateway pode ser implantado na frente do pool de Servidores de Front-end Fiori.

### <a name="application-servers-pool"></a>Pool de servidores de aplicativos

Para gerenciar grupos de logon para servidores de aplicativos ABAP, é usada a transação SMLG. Ela usa a função de balanceamento de carga no servidor de mensagens do Central Services para distribuir a carga de trabalho entre o pool de servidores de aplicativos SAP para o tráfego de SAPGUIs e RFC. A conexão do servidor de aplicativos para o Central Services altamente disponível é feita através do nome de rede virtual do cluster. Isso evita a necessidade de alterar o perfil de servidor de aplicativos para a conectividade do Central Services após um failover local.

### <a name="sap-central-services-cluster"></a>Cluster do SAP Central Services

O Central Services poderá ser implantado em uma única máquina virtual quando não for um requisito de alta disponibilidade. No entanto, a máquina virtual única se torna um potencial ponto único de falha (SPOF) para o ambiente do SAP. Para uma implantação do Central Services altamente disponível, um cluster NFS altamente disponível e um cluster do Central Services altamente disponível são usados.

### <a name="nfs-cluster"></a>Cluster NFS

O DRBD (Distributed Replicated Block Device) é usado para a replicação entre os nós do cluster de NFS.

### <a name="availability-sets"></a>Conjuntos de disponibilidade

Os conjuntos de disponibilidade distribuem os servidores a diferentes grupos de infraestrutura e atualizam grupos para melhorar a disponibilidade de serviço. Use máquinas virtuais que executem a mesma função em conjuntos de disponibilidade para ajudar a proteger contra o tempo de inatividade causado por uma manutenção da infraestrutura do Azure e para atender às [SLAs](https://azure.microsoft.com/support/legal/sla/virtual-machines). É recomendável ter duas ou mais máquinas virtuais por conjunto de disponibilidade.

Todas as máquinas virtuais em um conjunto devem executar a mesma função. Não misture servidores de funções diferentes no mesmo conjunto de disponibilidade. Por exemplo, não coloque um nó do ASCS no mesmo conjunto de disponibilidade com o servidor de aplicativos.

### <a name="nics"></a>NICs

As paisagens tradicionais locais da SAP implementam várias NICs (placas de interface de rede) por computador para separar o tráfego administrativo do tráfego de negócios. No Azure, a rede virtual é uma rede definida pelo software que envia todo o tráfego pela mesma malha de rede. Portanto, é desnecessário o uso de vários NICs. No entanto, se sua organização precisar separar o tráfego, você pode implantar vários NICs por VM, conectar cada NIC a uma sub-rede diferente e, então, usar os NSGs para impor diferentes políticas de controle de acesso.

### <a name="subnets-and-nsgs"></a>Sub-redes e NSGs

Essa arquitetura subdivide o espaço de endereço da rede virtual em sub-redes. Cada sub-rede pode ser associada a um NSG que define as políticas de acesso para a sub-rede. Coloque os servidores de aplicativos em uma sub-rede separada para poder protegê-los mais facilmente por meio do gerenciamento das políticas de segurança de sub-rede, não dos servidores individuais.

Quando um NSG está associado a uma sub-rede, então ele se aplica a todos os servidores dentro dessa sub-rede. Para saber mais sobre como usar os NSGs para um controle refinado sobre os servidores em uma sub-rede, veja [Filtrar o tráfego de rede com os grupos de segurança de rede](https://azure.microsoft.com/blog/multiple-vm-nics-and-network-virtual-appliances-in-azure/).

Veja também [Planejamento e design para o Gateway de VPN](/azure/vpn-gateway/vpn-gateway-plan-design).

### <a name="load-balancers"></a>Balanceadores de carga

O [SAP Web Dispatcher](https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm) lida com o balanceamento de carga do tráfego HTTP(S) incluindo aplicativos do estilo do Fiori a um pool de servidores de aplicativos SAP.

Para o tráfego de clientes do SAP GUI se conectando a um servidor SAP via DIAG ou RFC (Remote Function Calls), o servidor de mensagens do Central Services faz o balanceamento da carga por meio de [grupos de logon](https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing) do servidor de aplicativos SAP, não sendo necessário nenhum balanceador de carga adicional.

### <a name="azure-storage"></a>Armazenamento do Azure

É recomendável usar o Armazenamento Premium do Azure para as máquinas de virtuais do servidor de banco de dados. O armazenamento Premium oferece latência de leitura/gravação consistente. Para obter detalhes sobre o uso do Armazenamento Premium para os discos do sistema operacional e os discos de dados de uma máquina de instância única, veja [SLA para máquinas virtuais](https://azure.microsoft.com/support/legal/sla/virtual-machines/).

Para todos os sistemas SAP de produção, é recomendável usar o [Azure Managed Disks](/azure/storage/storage-managed-disks-overview) Premium. Os Managed Disks são usados para gerenciar os arquivos VHD dos discos, adicionando confiabilidade. Eles também garantem que os discos para máquinas virtuais dentro de um conjunto de disponibilidade fiquem isolados para evitar pontos únicos de falha.

Para servidores de aplicativos SAP, incluindo as máquinas virtuais do Central Services, é possível usar o Armazenamento Padrão do Azure para reduzir os custos, uma vez que a execução do aplicativo ocorre na memória e usa os discos somente para registro em log. No entanto, neste momento, o Armazenamento Padrão só está certificado para armazenamento não gerenciado. Uma vez que os servidores de aplicativos não hospedam nenhum dado, também é possível usar os discos menores de Armazenamento Premium P4 e P6 para ajudar a minimizar o custo.

Para o repositório de dados de backup, é recomendável usar a [camada de acesso esporádico e/ou o armazenamento da camada de acesso de arquivos](/azure/storage/storage-blob-storage-tiers) do Azure. Essas camadas de armazenamento são maneiras econômicas para armazenar dados de longa vida útil que são acessados com menos frequência.

## <a name="performance-considerations"></a>Considerações sobre o desempenho

Os servidores de aplicativos SAP realizam constantes comunicações com os servidores de banco de dados. Para as máquinas virtuais de banco de dados HANA, considere habilitar o [Acelerador de Gravação](/azure/virtual-machines/linux/how-to-enable-write-accelerator) para melhorar a latência da gravação de logs. Para otimizar as comunicações entre servidores, use a [Rede Acelerada](https://azure.microsoft.com/blog/linux-and-windows-networking-performance-enhancements-accelerated-networking/). Observe que esses aceleradores estão disponíveis apenas para determinadas séries de VM.

Para obter altas IOPS e taxas de transferências de largura de banda de disco, as práticas comuns na [otimização de desempenho](/azure/virtual-machines/linux/premium-storage-performance) do volume de armazenamento se aplicam ao layout de armazenamento do Azure. Por exemplo, a combinação de vários discos em conjunto para criar um volume de discos distribuídos melhora o desempenho de E/S. Habilitar o cache de leitura do conteúdo de armazenamento que raramente é alterado melhora a velocidade de recuperação de dados. Para obter detalhes sobre os requisitos de desempenho, consulte [Nota SAP 1943937 - Ferramenta de verificação de configuração de hardware](https://launchpad.support.sap.com/#/notes/1943937) (conta do SAP Service Marketplace necessária para o acesso).

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Na camada de aplicativo SAP, o Azure oferece uma ampla variedade de tamanhos de máquina virtual para escalar vertical ou horizontalmente. Para obter uma lista inclusiva, confira a [Nota SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533) - Aplicativos SAP no Azure: Produtos e tipos de VM do Azure com suporte (é necessário ter uma conta do SAP Service Marketplace para ter acesso). Conforme continuamos a certificar mais tipos de máquinas virtuais, você pode expandir ou reduzir com a mesma implantação de nuvem.

Na camada de banco de dados, essa arquitetura executa o HANA em máquinas virtuais. Se sua carga de trabalho exceder o tamanho máximo da VM, a Microsoft também oferecerá [Instâncias Grandes do Azure](/azure/virtual-machines/workloads/sap/hana-overview-architecture) para o SAP HANA. Esses servidores físicos são colocados em um datacenter certificado do Microsoft Azure e, no momento da redação deste artigo, forneça até 20 TB de capacidade de memória para uma única instância. Uma configuração de vários nós também é possível com uma capacidade de memória total de até 60 TB.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

A redundância de recursos é o tema geral em soluções de infraestrutura altamente disponíveis. Para empresas que contam com um SLA menos rigoroso, as máquinas virtuais de instância única do Azure oferecem um SLA de tempo de atividade. Para obter mais informações, confira [Contrato de Nível de Serviço do Azure](https://azure.microsoft.com/support/legal/sla/).

Nessa instalação distribuída do aplicativo SAP, a instalação básica é replicada para obter a alta disponibilidade. O design de alta disponibilidade varia para cada camada da arquitetura.

### <a name="application-tier"></a>Camada de aplicativo

- Web Dispatcher. A alta disponibilidade é obtida com instâncias redundantes do Web Dispatcher. Veja [SAP Web Dispatcher](https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm) na documentação do SAP.
- Servidores Fiori. A alta disponibilidade é obtida pelo tráfego de balanceamento de carga dentro de um pool de servidores.
- Central Services. Para a alta disponibilidade do Central Services em Máquinas Virtuais do Linux do Azure, a Extensão de Alta Disponibilidade apropriada para a distribuição Linux selecionada é usada, e o cluster NFS altamente disponível hospeda o armazenamento DRBD.
- Servidores de aplicativos. A alta disponibilidade é obtida pelo tráfego de balanceamento de carga dentro de um pool de servidores de aplicativos.

### <a name="database-tier"></a>Camada de banco de dados

Essa arquitetura de referência descreve um sistema altamente disponível de banco de dados SAP HANA com duas máquinas virtuais do Azure. O recurso de replicação de sistema nativo da camada de banco de dados fornece o failover manual ou automático entre nós replicados:

- Para failover manual, implante mais de uma instância do HANA e use a Replicação de Sistema do HANA (HSR).
- Para failover automático, use HSR e a Extensão de Alta Disponibilidade (HAE) do Linux para sua distribuição Linux. O HAE Linux fornece os serviços de cluster para os recursos do HANA, detecção de eventos de falha e orquestração do failover de serviços problemáticos para o nó íntegro.

Veja [Certificações e configurações da SAP em execução no Microsoft Azure](/azure/virtual-machines/workloads/sap/sap-certifications).

### <a name="disaster-recovery-considerations"></a>Considerações de recuperação de desastres

Cada camada usa uma estratégia diferente para fornecer proteção de recuperação de desastres.

- **Camada de servidores de aplicativos**. Os servidores de aplicativos SAP não contêm dados corporativos. No Azure, uma estratégia simples de DR é criar servidores de aplicativos SAP na região secundária, depois tirá-los do ar. Após quaisquer alterações de configuração ou atualizações de kernel no servidor de aplicativo primário, as mesmas alterações devem ser aplicadas às máquinas virtuais na região secundária. Por exemplo, copie os executáveis de kernel do SAP para as máquinas virtuais de recuperação de desastre. Para a replicação automática dos servidores de aplicativos para uma região secundária, o [Azure Site Recovery](/azure/site-recovery/site-recovery-overview) é a solução recomendada. No momento da preparação deste documento, o ASR ainda não dá suporte à replicação da configuração de Rede Acelerada em VMs do Azure.

- **Central Services**. Esse componente da pilha do aplicativo SAP também não persiste dados corporativos. Você pode criar uma VM na região secundária para executar a função do Central Services. O único conteúdo do nó do Central Services primário a ser sincronizado é o conteúdo compartilhado /sapmnt. Além disso, se as configurações mudarem ou ocorrerem atualizações de kernel em servidores primários do Central Services, elas devem ser repetidas na VM na região secundária sendo executada no Central Services. Para sincronizar os dois servidores, você pode usar o Azure Site Recovery para replicar os nós de cluster ou simplesmente usar um trabalho de cópia agendado regularmente para copiar /sapmnt para o lado da recuperação de desastre. Para obter detalhes sobre a compilação, cópia e teste do processo de failover, baixe [SAP NetWeaver: Compilar uma solução de recuperação de desastre baseada em Hyper-V e no Microsoft Azure](https://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx) e consulte a seção 4.3, "Camada SAP SPOF (ASCS)". Este artigo se aplica ao NetWeaver em execução no Windows, mas você pode criar a configuração equivalente para Linux. Para o Central Services, use o [Azure Site Recovery](/en-us/azure/site-recovery/site-recovery-overview) para replicar os nós e o armazenamento do cluster. Para o Linux, crie um cluster geográfico de três nós usando uma Extensão de Alta Disponibilidade.

- **Camada de banco de dados do SAP**. Use o HSR para replicação com suporte do HANA. Além de uma configuração de alta disponibilidade local de dois nós, a HSR oferece suporte à replicação de várias camadas, em que um terceiro nó em uma região separada do Azure age como uma entidade estrangeira, que não faz parte do cluster, e é registrado na réplica secundária do par da HSR com cluster como seu destino de replicação. Isso forma uma corrente encadeada de replicação. O failover para o nó de recuperação de desastre é um processo manual.

Para usar o Azure Site Recovery para compilar automaticamente um site de produção totalmente replicado do original, você deve executar [scripts de implantação](/azure/site-recovery/site-recovery-runbook-automation) personalizados. Primeiro a recuperação de site implanta as máquinas virtuais em conjuntos de disponibilidade, depois executa scripts para adicionar recursos, como balanceadores de carga.

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

O SAP HANA tem um recurso de backup que usa a infraestrutura subjacente do Azure. Para fazer backup do banco de dados do SAP HANA em execução em máquinas virtuais do Azure, o instantâneo do armazenamento do Azure e o instantâneo do SAP HANA são usados para garantir a consistência dos arquivos de backup. Para obter detalhes, veja o [Guia de backup para o SAP HANA em máquinas virtuais do Azure](/azure/virtual-machines/workloads/sap/sap-hana-backup-guide) e [Perguntas frequentes sobre o serviço de Backup do Azure](/azure/backup/backup-azure-backup-faq). Somente as implantações de contêiner único do HANA oferecem suporte a instantâneos de armazenamento do Azure.

### <a name="identity-management"></a>Gerenciamento de identidades

Controle o acesso aos recursos usando um sistema de gerenciamento centralizado de identidades em todos os níveis:

- Forneça acesso aos recursos do Azure por meio de [controle de acesso baseado em função](/azure/active-directory/role-based-access-control-what-is) (RBAC).
- Conceda acesso às VMs do Azure por meio do LDAP, Azure Active Directory, Kerberos ou outro sistema.
- Ofereça suporte ao acesso de dentro dos próprios apps por meio dos serviços fornecidos pela SAP ou use o [OAuth 2.0 e o Azure Active Directory](/azure/active-directory/develop/active-directory-protocols-oauth-code).

### <a name="monitoring"></a>Monitoramento

O Azure oferece várias funções para [monitoramento e diagnóstico](/azure/architecture/best-practices/monitoring) da infraestrutura geral. Além disso, o monitoramento avançado de máquinas virtuais do Azure (Linux ou Windows) é tratado pelo Azure Operations Management Suite (OMS).

Para fornecer monitoramento de recursos baseado na SAP e desempenho de serviço da infraestrutura da SAP, é usada a extensão [Monitoramento Avançado da SAP do Azure](/azure/virtual-machines/workloads/sap/deployment-guide#d98edcd3-f2a1-49f7-b26a-07448ceb60ca). Essa extensão fornece as estatísticas de monitoramento do Azure para o aplicativo SAP para monitoramento do sistema operacional e as funções da ferramenta cockpit de DBA. O monitoramento avançado do SAP é um pré-requisito obrigatório para executar o SAP no Azure. Verifique os detalhes na [nota do SAP 2191498](https://launchpad.support.sap.com/#/notes/2191498) – "SAP no Linux com o Azure: Monitoramento avançado."

## <a name="security-considerations"></a>Considerações de segurança

O SAP tem seu próprio mecanismo de gerenciamento de usuários (UME) para controlar o acesso e a autorização baseados em função do aplicativo SAP. Para obter detalhes, veja [Segurança do SAP HANA — uma visão geral](https://archive.sap.com/documents/docs/DOC-62943) (uma conta da SAP Service Marketplace é necessária para o acesso).

Para a segurança da rede adicional, cogite implementar uma [Rede de Perímetro](/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid), a qual usa um dispositivo de rede virtual para criar um firewall na frente da sub-rede para o Web Dispatcher e pools de Servidores de Front-end Fiori.

Para a segurança da infraestrutura, os dados são criptografados em trânsito e em repouso. A seção "Considerações sobre segurança" do [SAP NetWeaver nas máquinas virtuais do Azure – Guia de planejamento e implementação](/azure/virtual-machines/workloads/sap/planning-guide) começa a tratar da segurança de rede e a aplica ao S/4HANA. O guia também especifica as portas de rede que você deve abrir nos firewalls para permitir a comunicação do aplicativo.

Para criptografar os discos de máquina virtual do IaaS Linux, você pode usar o [Azure Disk Encryption](/azure/security/azure-security-disk-encryption). Ele usa o recurso DM-Crypt do Linux para fornecer criptografia de volume para o sistema operacional e os discos de dados. A solução também funciona com o Azure Key Vault para ajudá-lo a controlar e gerenciar os segredos e chaves de criptografia de disco em sua assinatura do Key Vault. Os dados em discos da máquina virtual são criptografados em repouso no armazenamento do Azure.

Para criptografia de dados em repouso do SAP HANA, é recomendável usar a tecnologia de criptografia nativa do SAP HANA.

> [!NOTE]
> Não use a criptografia de dados em repouso HANA com Azure Disk Encryption no mesmo servidor. Para o HANA, use somente a criptografia de dados do HANA.

## <a name="communities"></a>Comunidades

As comunidades podem responder a perguntas e ajudá-lo a configurar uma implantação bem-sucedida. Considere o seguinte:

- [Como executar aplicativos SAP no blog Microsoft Platform](https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/)
- [Suporte da Comunidade do Azure](https://azure.microsoft.com/support/community/)
- [Comunidade do SAP](https://www.sap.com/community.html)
- [Stack Overflow](https://stackoverflow.com/tags/sap/)

## <a name="related-resources"></a>Recursos relacionados

Talvez seja melhor examinar os seguintes [cenários de exemplo do Azure](/azure/architecture/example-scenario), que demonstram soluções específicas usando algumas das mesmas tecnologias:

- [Executando cargas de trabalho de produção do SAP usando um Oracle Database no Azure](/azure/architecture/example-scenario/apps/sap-production)
- [Ambientes de desenvolvimento/teste para cargas de trabalho do SAP no Azure](/azure/architecture/example-scenario/apps/sap-dev-test)

<!-- links -->

[visio-download]: https://archcenter.blob.core.windows.net/cdn/sap-reference-architectures.vsdx
