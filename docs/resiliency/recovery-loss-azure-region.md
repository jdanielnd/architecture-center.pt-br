---
title: "Recuperação de perda de uma região do Azure"
description: "Artigo sobre compreensão e design de aplicativos resilientes, altamente disponíveis e tolerante a falhas, bem como planejamento de recuperação de desastre"
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: 42a7d865e101b43279f3198f3dd75df1b15a8565
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="azure-resiliency-technical-guidance-recovery-from-a-region-wide-service-disruption"></a>Orientações técnicas de resiliência do Azure: recuperação de uma interrupção de serviço em toda a região
O Azure é dividido fisicamente e logicamente em unidades chamadas de regiões. Uma região consiste em um ou mais datacenters nas proximidades. 

Em raras circunstâncias é possível que instalações em toda a região se tornem inacessíveis, por exemplo, devido a falhas de rede. Ou instalações podem ser inteiramente perdidas, por exemplo, devido a um desastre natural. Esta seção explica os recursos do Azure para criar aplicativos que são distribuídos entre regiões. Essa distribuição ajuda a minimizar a possibilidade de que uma falha em uma região afete outras regiões.

## <a name="cloud-services"></a>Serviços de Nuvem
### <a name="resource-management"></a>Gerenciamento de recursos
Você pode distribuir as instâncias de computação entre regiões criando um serviço de nuvem separado em cada região de destino e publicando o pacote de implantação em cada serviço de nuvem. No entanto, observe que a distribuição de tráfego pelos serviços de nuvem em diferentes regiões deve ser implementado pelo desenvolvedor do aplicativo ou com um serviço de gerenciamento de tráfego.

Determinar o número de instâncias de função de reposição a implantar com antecedência para recuperação de desastre é um aspecto importante do planejamento de capacidade. Ter uma implantação secundária completa garante que a capacidade esteja disponível quando necessário; no entanto, isso dobra efetivamente o custo. Um padrão comum é ter uma implantação secundária pequena, mas grande o suficiente para executar serviços críticos. Essa implantação secundária pequena é uma boa ideia, para reservar capacidade e para testar a configuração do ambiente secundário.

> [!NOTE]
> A cota da assinatura não é uma garantia de capacidade. A cota é simplesmente um limite de crédito. Para garantir a capacidade, o número necessário de funções deve ser definido no modelo de serviço, e as funções devem ser implantadas.
> 
> 

### <a name="load-balancing"></a>Balanceamento de Carga
Balancear a carga de tráfego entre regiões requer uma solução de gerenciamento de tráfego. O Azure oferece o [Gerenciador de Tráfego do Azure](https://azure.microsoft.com/services/traffic-manager/). Você também pode tirar proveito dos serviços de terceiros que fornecem recursos de gerenciamento de tráfego semelhantes.

### <a name="strategies"></a>Estratégias
Muitas estratégias alternativas estão disponíveis para implementação de computação distribuída entre regiões. Essas estratégias devem ser personalizadas segundo os requisitos específicos de negócios e as circunstâncias do aplicativo. Em um nível elevado, as abordagens podem ser divididas nas seguintes categorias:

* **Reimplantar em caso de desastre**: nessa abordagem, o aplicativo é reimplantado do zero no momento do desastre. Isso é adequado para aplicativos não críticos, que não exigem um tempo de recuperação garantido.
* **Reposição morna (ativo/passivo)**: um serviço hospedado secundário é criado em uma região alternativa e funções são implantadas para garantir a capacidade mínima; no entanto, as funções não recebem tráfego de produção. Essa abordagem é útil para aplicativos que não foram projetados para distribuir tráfego entre regiões.
* **Reposição quente (ativo/ativo)**: o aplicativo foi projetado para receber a carga de produção em várias regiões. Os serviços de nuvem em cada região podem ser configurados para capacidade maior do que o necessário para fins de recuperação de desastre. Como alternativa, os serviços de nuvem podem escalar horizontalmente conforme o necessário no momento de um desastre e failover. Essa abordagem exige um investimento substancial no design do aplicativo, mas tem benefícios significativos. Isso inclui tempo de recuperação baixo e garantido, testes contínuos de todos os locais de recuperação e uso eficiente da capacidade.

Uma discussão completa sobre design distribuído está fora do escopo deste documento. Para saber mais, confira [Recuperação de Desastre e Alta Disponibilidade para Aplicativos do Azure](https://aka.ms/drtechguide).

## <a name="virtual-machines"></a>Máquinas Virtuais
A recuperação de VMs (máquinas virtuais) de IaaS (infraestrutura como um serviço) é semelhante à recuperação de computação de PaaS (plataforma como serviço) em muitos aspectos. No entanto, há diferenças importantes, pois uma VM IaaS consiste na VM e no disco de VM.

* **Use o Backup do Azure para criar backups entre regiões consistentes com aplicativos**.
  [Backup do Azure](https://azure.microsoft.com/services/backup/) habilita os clientes a criar backups consistentes com aplicativos em vários discos de VM e dar suporte à replicação de backups entre regiões. Você pode fazer isso optando por replicar geograficamente o cofre de backup no momento da criação. Observe que a replicação do cofre de backup deve ser configurada no momento da criação. Ele não pode ser definido mais tarde. Se uma região for perdida, a Microsoft disponibilizará os backups para os clientes. Os clientes poderão restaurar para qualquer um dos seus pontos de restauração configurados.
* **Separar os discos de dados do disco do sistema operacional**. Uma consideração importante para VMs IaaS é que você não pode alterar o disco do sistema operacional sem recriar a VM. Isso não será um problema se sua estratégia de recuperação for a de reimplantar após desastre. No entanto, poderá ser um problema se você estiver usando a abordagem de reposição morna para capacidade reserva. Para implementar isso adequadamente, você deve ter o disco de sistema operacional correto implantado nos locais primário e secundário, e os dados de aplicativos devem ser armazenados em uma unidade separada. Se possível, use uma configuração de sistema operacional padrão, que possa ser fornecida em ambos os locais. Após um failover, você deve anexar a unidade de dados às VMs IaaS existentes no controlador de domínio secundário. Use o AzCopy para copiar instantâneos dos discos de dados para um site remoto.
* **Estar ciente dos possíveis problemas de consistência após um failover geográfico de vários Discos de VM**. Os Discos de VM são implementados como blobs de Armazenamento do Azure e têm a mesma característica de replicação geográfica. A menos que o [Backup do Azure](https://azure.microsoft.com/services/backup/) seja usado, não há garantia de consistência entre discos, pois a replicação geográfica é assíncrona e é replicada de maneira independente. Discos de VM individuais têm a garantia de estar em um estado com controle de falhas após um failover geográfico, mas não consistente entre discos. Isso pode causar problemas em alguns casos (por exemplo, no caso de divisão entre discos).

## <a name="storage"></a>Armazenamento
### <a name="recovery-by-using-geo-redundant-storage-of-blob-table-queue-and-vm-disk-storage"></a>Recuperação usando armazenamento com Redundância Geográfica de blob, tabela, fila e armazenamento em disco de VM
Em blobs do Azure, tabelas, filas e discos de VM são todos replicados geograficamente por padrão. Isso é conhecido como GRS (armazenamento com redundância geográfica). O GRS replica dados de armazenamento para um datacenter emparelhado a centenas de milhas de distância dentro de uma região geográfica específica. O GRS é destinado a fornecer durabilidade adicional caso ocorra um desastre de grandes proporções no datacenter. A Microsoft controla quando ocorre o failover, que se limita a grandes desastres em que o local principal de origem é considerado irrecuperável em um período de tempo razoável. Em alguns cenários, isso pode consistir em vários dias. Os dados normalmente são replicados em poucos minutos, embora o intervalo de sincronização ainda não seja coberto por um contrato de nível de serviço.

No caso de um failover geográfico, não haverá alteração na maneira como a conta é acessada (a URL e a chave da conta não serão alteradas). No entanto, a conta de armazenamento estará em uma região diferente após o failover. Isso pode afetar aplicativos que exigem afinidade regional com sua conta de armazenamento. Mesmo para serviços e aplicativos que não exigem uma conta de armazenamento no mesmo datacenter, os encargos de largura de banda e a latência entre datacenters podem ser motivos convincentes para mover temporariamente o tráfego para a região de failover. Isso pode influenciar uma estratégia geral de recuperação de desastre.

Além do failover automático fornecido pelo GRS, o Azure introduziu um serviço que fornece a você acesso de leitura à cópia dos dados no local de armazenamento secundário. Isso é chamado de RA-GRS (armazenamento com redundância geográfica com acesso de leitura).

Para saber mais sobre as opções de armazenamento GRS e RA-GRS, confira [Replicação do Armazenamento do Azure](/azure/storage/storage-redundancy/).

### <a name="geo-replication-region-mappings"></a>Mapeamentos de região de Replicação Geográfica:
É importante saber onde os dados são replicados geograficamente para saber onde implantar as outras instâncias de dados que exigem afinidade regional com o armazenamento. Para obter mais informações, consulte [Regiões emparelhadas do Azure](/azure/best-practices-availability-paired-regions).

### <a name="geo-replication-pricing"></a>Preços da Replicação Geográfica:
A replicação geográfica está incluída no preço atual do Armazenamento do Azure. Isso se chama GRS (Armazenamento com Redundância Geográfica). Se não quiser que os dados sejam replicados geograficamente, você poderá desabilitar a replicação geográfica para sua conta. Isso é chamado de Armazenamento com Redundância Local e é cobrado com um preço com desconto em comparação com o GRS.

### <a name="determining-if-a-geo-failover-has-occurred"></a>Determinando se um failover geográfico ocorreu
Se ocorrer um failover geográfico, isso será postado no [Painel de Integridade de Serviço do Azure](https://azure.microsoft.com/status/). No entanto, os aplicativos podem implementar um meio automatizado de detectar isso monitorando a região geográfica da conta de armazenamento. Isso pode ser usado para disparar outras operações de recuperação, como a ativação de recursos de computação na região geográfica para a qual o armazenamento foi movido. Você pode executar uma consulta para isso da API de gerenciamento de serviços, usando [Obter Propriedades de Conta de Armazenamento](https://msdn.microsoft.com/library/ee460802.aspx). As propriedades relevantes são:

    <GeoPrimaryRegion>primary-region</GeoPrimaryRegion>
    <StatusOfPrimary>[Available|Unavailable]</StatusOfPrimary>
    <LastGeoFailoverTime>DateTime</LastGeoFailoverTime>
    <GeoSecondaryRegion>secondary-region</GeoSecondaryRegion>
    <StatusOfSecondary>[Available|Unavailable]</StatusOfSecondary>

### <a name="vm-disks-and-geo-failover"></a>Discos de VM e failover geográfico
Conforme discutido na seção sobre discos de VM, não há garantias de consistência de dados entre discos de VM após um failover. Para garantir a exatidão de backups, um produto de backup como o Data Protection Manager deve ser usado para fazer backup de dados de aplicativos e restaurá-los.

## <a name="database"></a>Banco de dados
### <a name="sql-database"></a>Banco de Dados SQL
O Banco de Dados SQL do Azure fornece dois tipos de recuperação: Restauração Geográfica e Replicação Geográfica Ativa.

#### <a name="geo-restore"></a>Restauração geográfica
[Restauração geográfica](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) também está disponível para bancos de dados Basic, Standard e Premium. Ele fornece a opção de recuperação padrão quando o banco de dados está indisponível devido a um incidente na região em que o banco de dados está hospedado. De forma semelhante à Recuperação Pontual, a Restauração Geográfica conta com backups de banco de dados no armazenamento do Azure com redundância geográfica. Ele restaura da cópia de backup replicado geograficamente e, assim, é resistente a falhas de armazenamento na região primária. Para obter mais detalhes, consulte [Restaurar um banco de dados SQL ou fazer failover para um secundário](/azure/sql-database/sql-database-disaster-recovery/).

#### <a name="active-geo-replication"></a>Replicação geográfica ativa
[Replicação geográfica ativa](/azure/sql-database/sql-database-geo-replication-overview/) está disponível para todas as camadas de banco de dados. Ela foi criada para aplicativos que têm requisitos de recuperação mais agressivos do que a Restauração Geográfica pode oferecer. Com a replicação geográfica ativa, você pode criar até quatro secundários legíveis nos servidores em regiões diferentes. Você pode iniciar o failover para qualquer um dos secundários. Além disso, a replicação geográfica ativa pode ser usada para suportar a atualização do aplicativo ou os cenários de realocação, bem como o balanceamento de carga para cargas de trabalho somente leitura. Para obter detalhes, veja [configurar a Replicação Geográfica](/azure/sql-database/sql-database-geo-replication-portal/) e [failover para o banco de dados secundário](/azure/sql-database/sql-database-geo-replication-failover-portal/). Confira [Criar um aplicativo para recuperação de desastre na nuvem usando a Replicação Geográfica ativa no Banco de Dados SQL](/azure/sql-database/sql-database-designing-cloud-solutions-for-disaster-recovery/) e [Gerenciando as atualizações sem interrupção de aplicativos na nuvem usando a Replicação Geográfica Ativa do Banco de Dados SQL](/azure/sql-database/sql-database-manage-application-rolling-upgrade/) para obter detalhes sobre como criar e implementar aplicativos e a atualização de aplicativos sem tempo de inatividade.

### <a name="sql-server-on-virtual-machines"></a>SQL Server em máquinas virtuais
Várias opções estão disponíveis para recuperação e alta disponibilidade para o SQL Server 2012 (e posterior) em execução em Máquinas Virtuais do Azure. Para saber mais, confira [Alta disponibilidade e recuperação de desastres para o SQL Server em Máquinas Virtuais do Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

## <a name="other-azure-platform-services"></a>Outros serviços da plataforma Azure
Ao tentar executar seu serviço de nuvem em várias regiões do Azure, você deve considerar as implicações para cada uma de suas dependências. Nas seções a seguir, as diretrizes específicas do serviço pressupõem que você deve usar o mesmo serviço do Azure em um datacenter alternativo do Azure. Isso envolve tanto tarefas de configuração quanto de replicação de dados.

> [!NOTE]
> Em alguns casos, essas etapas podem ajudar a atenuar uma interrupção de serviço específico no lugar de um evento no datacenter inteiro. Da perspectiva do aplicativo, uma interrupção de serviço específico pode ser igualmente limitadora e exigiria a migração temporária do serviço para uma região alternativa do Azure.
> 
> 

### <a name="service-bus"></a>Barramento de Serviço
O Barramento de Serviço do Azure usa um namespace exclusivo que não abrange regiões do Azure. Portanto, o primeiro requisito é configurar os namespaces do barramento de serviço necessários na região alternativa. No entanto, também há considerações sobre a durabilidade das mensagens na fila. Há várias estratégias para replicar mensagens entre regiões do Azure. Para obter detalhes sobre essas estratégias de replicação e outras estratégias de recuperação de desastre, confira [Práticas recomendadas para isolar aplicativos contra interrupções e desastres do Barramento de Serviço](/azure/service-bus-messaging/service-bus-outages-disasters/). Para obter outras considerações sobre disponibilidade, confira [Barramento de Serviço (Disponibilidade)](recovery-local-failures.md#other-azure-platform-services).

### <a name="app-service"></a>Serviço de Aplicativo
Para migrar um aplicativo do Serviço de Aplicativo do Azure, como Aplicativos Web ou Aplicativos Móveis, para uma região do Azure secundária, você deve ter um backup do site disponível para publicação. Se a interrupção não envolver todo o datacenter do Azure, talvez seja possível usar o FTP para baixar um backup recente do conteúdo do site. Em seguida, crie um novo aplicativo na região alternativa, a menos que você tenha feito isso anteriormente para a capacidade reserva. Publique o site na nova região e faça quaisquer alterações de configuração necessárias. Essas alterações podem incluir cadeias de conexão de banco de dados ou outras configurações específicas de região. Se necessário, adicione o certificado SSL do site e altere o registro DNS CNAME para que o nome de domínio personalizado aponte para a URL do aplicativo Web do Azure reimplantado.

### <a name="hdinsight"></a>HDInsight
Os dados associados ao HDInsight são armazenados por padrão no armazenamento de Blobs do Azure. O HDInsight requer que um cluster Hadoop processando trabalhos de MapReduce esteja colocalizado na mesma região da conta de armazenamento que contém os dados sendo analisados. Desde que você use o recurso de replicação geográfica disponível para o Armazenamento do Azure, você poderá acessar os dados na região secundária onde os dados foram replicados se, por algum motivo, a região primária não estiver mais disponível. Você pode criar um novo cluster Hadoop na região em que os dados foram replicados e continuar processando-os. Para obter outras considerações sobre disponibilidade, confira [HDInsight (Disponibilidade)](recovery-local-failures.md#other-azure-platform-services).

### <a name="sql-reporting"></a>Relatórios SQL
Neste momento, a recuperação da perda de uma região do Azure exige várias instâncias de Relatórios SQL em diferentes regiões do Azure. Essas instâncias de Relatórios SQL devem acessar os mesmos dados, e esses dados devem ter seu próprio plano de recuperação em caso de desastre. Você também pode manter cópias de backup externas do arquivo RDL para cada relatório.

### <a name="media-services"></a>Serviços de mídia
Os Serviços de Mídia do Azure têm abordagens de recuperação diferentes para codificação e streaming. Normalmente, a transmissão é mais crítica durante uma interrupção regional. Para se preparar para isso, você deve ter uma conta de Serviços de Mídia em duas regiões do Azure diferentes. O conteúdo codificado deve estar localizado em ambas as regiões. Durante uma falha, você pode redirecionar o tráfego de streaming para a região alternativa. A codificação pode ser executada em qualquer região do Azure. Se a codificação for sensível ao tempo, por exemplo, durante o processamento de eventos ao vivo, você deverá estar preparado para enviar trabalhos para um datacenter alternativo durante eventuais falhas.

### <a name="virtual-network"></a>Rede virtual
Os arquivos de configuração fornecem a maneira mais rápida de configurar uma rede virtual em uma região alternativa do Azure. Após configurar a rede virtual na região primária do Azure, [exporte as configurações de rede virtual](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) da rede atual para um arquivo de configuração de rede. Caso ocorra uma interrupção na região primária, [restaure a rede virtual](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) usando o arquivo de configuração armazenado. Em seguida, configure outros serviços de nuvem, máquinas virtuais ou configurações entre locais para trabalhar com a nova rede virtual.

## <a name="checklists-for-disaster-recovery"></a>Listas de verificação para recuperação de desastre
## <a name="cloud-services-checklist"></a>Lista de verificação de Serviços de Nuvem
1. Examinar a seção Serviços de Nuvem deste documento.
2. Criar uma estratégia de recuperação de desastre entre regiões.
3. Compreender as compensações em reservar capacidade em regiões alternativas.
4. Usar ferramentas de roteamento de tráfego, como o Gerenciador de Tráfego do Azure.

## <a name="virtual-machines-checklist"></a>Lista de verificação de Máquinas Virtuais
1. Examinar a seção Máquinas Virtuais deste documento.
2. Usar [Backup do Azure](https://azure.microsoft.com/services/backup/) para criar backups consistentes com aplicativos em regiões.

## <a name="storage-checklist"></a>Lista de verificação de armazenamento
1. Examinar a seção Armazenamento deste documento.
2. Não desabilitar a replicação geográfica dos recursos de armazenamento.
3. Compreender a região alternativa para a replicação geográfica em caso de failover.
4. Criar estratégias de backup personalizadas para estratégias de failover controlado pelo usuário.

## <a name="sql-database-checklist"></a>Lista de verificação de Banco de Dados SQL
1. Examinar a seção Banco de Dados SQL deste documento.
2. Usar a [Restauração Geográfica](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) ou a [Replicação Geográfica](/azure/sql-database/sql-database-geo-replication-overview/) conforme apropriado.

## <a name="sql-server-on-virtual-machines-checklist"></a>Lista de verificação do SQL Server em Máquinas Virtuais
1. Examinar a seção SQL Server em máquinas virtuais deste documento.
2. Usar Grupos de Disponibilidade AlwaysOn entre regiões ou o espelhamento de banco de dados.
3. Como alternativa, usar o backup e restauração para o armazenamento de blobs.

## <a name="service-bus-checklist"></a>Lista de verificação do Barramento de Serviço
1. Examinar a seção Barramento de Serviço deste documento.
2. Configurar um namespace do Barramento de Serviço em uma região alternativa.
3. Considerar estratégias de replicação personalizadas para mensagens entre regiões.

## <a name="app-service-checklist"></a>Lista de verificação do Serviço de Aplicativo
1. Examinar a seção de Serviço de Aplicativo deste documento.
2. Manter os backups de sites fora da região primária.
3. Se a interrupção for parcial, tentar recuperar o site atual com FTP.
4. Planejar implantar o site em um site novo ou existente em uma região alternativa.
5. Planejar alterações de configuração tanto de aplicativo quanto dos registros DNS CNAME.

## <a name="hdinsight-checklist"></a>Lista de verificação do HDInsight
1. Examinar a seção HDInsight deste documento.
2. Criar um novo cluster Hadoop na região com dados replicados.

## <a name="sql-reporting-checklist"></a>Lista de verificação de Relatórios SQL
1. Examinar a seção Relatórios SQL deste documento.
2. Manter uma instância de Relatórios SQL alternativa em uma região diferente.
3. Manter um plano separado para replicar o destino para essa região.

## <a name="media-services-checklist"></a>Lista de verificação de Serviços de Mídia
1. Examinar a seção Serviços de Mídia deste documento.
2. Criar uma conta de Serviços de Mídia em uma região alternativa.
3. Codificar o mesmo conteúdo em ambas as regiões para dar suporte a failover de streaming.
4. Enviar trabalhos de codificação para uma região alternativa no caso de uma interrupção do serviço.

## <a name="virtual-network-checklist"></a>Lista de verificação de Rede Virtual
1. Examinar a seção Rede Virtual deste documento.
2. Usar as configurações da rede virtual exportadas para recriá-la em outra região.

