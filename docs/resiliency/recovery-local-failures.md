---
title: 'Orientações técnicas: Recuperação de falhas locais no Azure'
description: Artigo sobre compreensão e design de aplicativos resilientes, altamente disponíveis e tolerante a falhas, bem como planejamento de recuperação de desastre concentrado em falhas locais no Azure.
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: c5e26eefb8d5a8d424534081ddd5d1ea0454c17e
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112848"
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-local-failures-in-azure"></a>Orientações técnicas de resiliência do Azure: Recuperação de falhas locais no Azure

Há duas ameaças básicas à disponibilidade de aplicativos:

- A falha de dispositivos, como unidades e servidores
- O esgotamento de recursos essenciais, como a computação em condições de picos de carga

O Azure fornece uma combinação de gerenciamento de recursos, elasticidade, balanceamento de carga e particionamento para permitir alta disponibilidade sob essas circunstâncias. Alguns desses recursos são executados automaticamente para todos os serviços do Azure. No entanto, em alguns casos, o desenvolvedor de aplicativo precisa fazer alguns ajustes para se beneficiar deles.

## <a name="cloud-services"></a>Serviços de Nuvem

Os Serviços de Nuvem do Azure consistem em conjuntos de uma ou mais funções de trabalho ou web. Uma ou mais instâncias de uma função podem ser executadas simultaneamente. A configuração determina o número de instâncias. As instâncias de função são monitoradas e gerenciadas por meio de um componente chamado de controlador de malha. O controlador de malha detecta e responde automaticamente às falhas de software e hardware.

Cada instância de função é executada em sua própria VM (máquina virtual) e se comunica com seu controlador de malha por meio de um agente convidado. O agente convidado coleta métricas de recurso e de nó, incluindo uso, status, logs, uso de recursos, exceções e condições de falha da VM. O controlador de malha consulta o agente convidado em intervalos configuráveis e reinicia a VM se o agente convidado não responder. No caso de falha de hardware, o controlador de malha associado move todas as instâncias de função afetadas para um novo nó de hardware e reconfigura a rede para rotear o tráfego para lá.

Para tirar proveito desses recursos, os desenvolvedores devem garantir que todas as funções de serviço evitem armazenar o estado nas instâncias de função. Em vez disso, todos os dados persistentes devem ser acessados do armazenamento durável, como Armazenamento do Azure ou Banco de Dados SQL do Azure. Isso permite que qualquer função manipule solicitações. Isso também significa que as instâncias de função podem ficar inoperantes a qualquer momento sem criar inconsistências no estado transiente ou persistente do serviço.

O requisito para armazenar o estado externamente nas funções tem várias implicações. Significa, por exemplo, que todas as alterações relacionadas a uma tabela de Armazenamento do Azure devem ser modificadas em uma única transação de grupo de entidades, se possível. Obviamente, nem sempre é possível fazer todas as alterações em uma única transação. Você deve tomar cuidado especial para garantir que falhas de instância de função não causem problemas quando interromperem operações de longa execução abrangendo duas ou mais atualizações para o estado persistente do serviço. Se outra função tentar repetir essa operação, ela deverá prever e tratar do caso em que o trabalho foi parcialmente concluído.

Por exemplo, considere um serviço que particione dados entre vários repositórios. Se uma função de trabalho falhar enquanto estiver realocando um fragmento, talvez a realocação do fragmento não seja finalizada. Ou a realocação pode ser repetida desde seu início por uma função de trabalho diferente, possivelmente causando uma corrupção de dados ou deixando dados órfãos. Para evitar problemas, as operações de longa execução devem ser dos tipos a seguir:

- *Idempotente*: repetíveis sem efeitos colaterais. Para ser idempotente, uma operação de longa execução deve ter o mesmo efeito, não importando quantas vezes ela é executada, mesmo que ela seja interrompida durante a execução.
- *Reinicializável incrementalmente*: capaz de continuar do ponto de falha mais recente. Para ser reinicializável incrementalmente, uma operação longa execução deve consistir em uma sequência de operações atômicas menores. Ela também deve registrar seu progresso no armazenamento durável, de modo que cada invocação subsequente obtenha seu predecessor interrompido.

Por fim, todas as operações de longa execução devem ser invocadas repetidamente até que sejam bem-sucedidas. Por exemplo, uma operação de provisionamento pode ser colocada em uma fila do Azure e depois ser removida da fila por uma função de trabalho somente quando for bem-sucedida. A coleta de lixo pode ser necessária para limpar os dados criados por operações interrompidas.

### <a name="elasticity"></a>Elasticidade

O número inicial de instâncias em execução para cada função é determinado na configuração de cada função. Os administradores devem configurar inicialmente cada uma das funções para execução com duas ou mais instâncias baseadas na carga esperada. Porém, é possível escalar verticalmente instâncias de função facilmente à medida que os padrões de uso mudam. Você pode fazer isso manualmente no portal do Azure ou pode automatizar o processo usando o Windows PowerShell, a API de Gerenciamento de Serviços ou ferramentas de terceiros. Para saber mais, confira [Como dimensionar automaticamente um serviço de nuvem](/azure/cloud-services/cloud-services-how-to-scale/).

### <a name="partitioning"></a>Particionamento

O controlador de malha do Azure usa dois tipos de partição:

- Um *domínio de atualização* é usado para atualizar instâncias de função de um serviço em grupos. O Azure implanta instâncias do serviço em vários domínios de atualização. Para uma atualização in-loco, o controlador de malha desativa todas as instâncias em um domínio de atualização, as atualiza e as reinicia antes de passar para o próximo domínio de atualização. Essa abordagem impede que o serviço inteiro fique indisponível durante o processo de atualização.
- Um *domínio de falha* define pontos potenciais de falha de hardware ou de rede. Para qualquer função que tenha mais de uma instância, o controlador de malha garante que as instâncias sejam distribuídas entre vários domínios de falha, a fim de evitar que falhas de hardware isoladas interrompam o serviço. Os domínios de falha administram toda a exposição de falhas de servidor e cluster.

O [SLA (contrato de nível de serviço) do Azure](https://azure.microsoft.com/support/legal/sla/) garante que quando duas ou mais instâncias de função web são implantadas em diferentes domínios de falha e atualização, elas tenham conectividade externa, pelo menos, 99,95% do tempo. Ao contrário do que ocorre nos domínios de atualização, não há nenhuma maneira de controlar o número de domínios de falha. O Azure aloca domínios de falha e distribui instâncias de função entre eles automaticamente. Pelo menos as primeiras duas instâncias de cada função são colocadas em diferentes domínios de falha e de atualização para garantir que qualquer função com pelo menos duas instâncias atenda ao SLA. Isso é representado no diagrama a seguir.

![Exibição simplificada de isolamento de domínio de falha](./images/technical-guidance-recovery-local-failures/partitioning-1.png)

### <a name="load-balancing"></a>Balanceamento de carga

Todo o tráfego de entrada para uma função web passa por um balanceador de carga sem estado, que distribui solicitações de cliente entre instâncias de função. Instâncias de função individuais não têm endereços IP públicos e não podem ser endereçáveis diretamente da Internet. As funções web não tem estado, de modo que qualquer solicitação de cliente possa ser roteada para qualquer instância de função. Um evento [StatusCheck](https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleenvironment.statuscheck.aspx) é gerado a cada 15 segundos. Você pode usá-lo para indicar se a função está pronta para receber tráfego ou se ela está ocupada e deve ser retirada da rotação do balanceador de carga.

## <a name="virtual-machines"></a>Máquinas Virtuais

As Máquinas Virtuais do Azure são diferentes de funções de computação de PaaS (plataforma como serviço) em vários aspectos, no que se refere à alta disponibilidade. Em alguns casos, você precisa fazer trabalho adicional para garantir a alta disponibilidade.

### <a name="disk-durability"></a>Durabilidade de disco

Diferentemente do que ocorre nas instâncias de função de PaaS, os dados armazenados em unidades de máquina virtual são persistentes mesmo quando a máquina virtual é realocada. As máquinas virtuais do Azure usam discos de VM que existem como blobs no Armazenamento do Azure. Devido às características de disponibilidade do Armazenamento do Azure, os dados armazenados nas unidades de uma máquina virtual também têm alta disponibilidade.

Observe que a unidade D (em VMs do Windows) é a exceção a essa regra. A unidade D é na verdade o armazenamento físico no servidor de rack que hospeda a VM, e seus dados serão perdidos se a VM for reciclada. A unidade D destina-se somente a armazenamento temporário. No Linux, o Azure "geralmente" (mas nem sempre) expõe o disco temporário local como o dispositivo de bloco /dev/sdb. Geralmente, ele é montado pelo Agente Linux do Azure como pontos de montagem /mnt/resource ou /mnt (configuráveis via /etc/waagent.conf).

<!-- markdownlint-disable MD024 -->

### <a name="partitioning"></a>Particionamento

O Azure compreende nativamente as camadas em um aplicativo de PaaS (função web e função de trabalho) e, portanto, pode distribuí-las entre domínios de falha e atualização. Em contrapartida, as camadas em um aplicativos de IaaS (infraestrutura como serviço) devem ser definidas manualmente por meio de conjuntos de disponibilidade. Os conjuntos de disponibilidade são necessários para um SLA de IaaS.

![Conjuntos de disponibilidade para máquinas virtuais do Azure](./images/technical-guidance-recovery-local-failures/partitioning-2.png)

No diagrama anterior, a camada de IIS (Serviços de Informações da Internet) (que funciona como uma camada de aplicativo Web) e a camada SQL (que funciona como uma camada de dados) são atribuídas a diferentes conjuntos de disponibilidade. Isso garante que todas as instâncias de cada camada tenham redundância de hardware na distribuição de máquinas virtuais entre domínios de falha e que as camadas inteiras não sejam desativadas durante uma atualização.

### <a name="load-balancing"></a>Balanceamento de carga

Se as VMs devem ter tráfego distribuído entre elas, você deve agrupá-las em um aplicativo e realizar o balanceamento de carga por um ponto de extremidade TCP ou UDP específico. Para saber mais, confira [Balanceamento de carga de máquinas virtuais](/azure/virtual-machines/virtual-machines-linux-load-balance/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Se as VMs receberem entrada de outra fonte (por exemplo, um mecanismo de enfileiramento), não será necessário um balanceador de carga. O balanceador de carga usará uma verificação básica de integridade para determinar se o tráfego deve ser enviado para o nó. Também é possível criar suas próprias investigações para implementar as métricas de integridade específicas de aplicativo que determinam se a VM deve receber tráfego.

## <a name="storage"></a>Armazenamento

O Armazenamento do Azure é o serviço de dados durável de linha de base do Azure. Ele fornece blob, tabela, fila e armazenamento de VM em disco. Ele usa uma combinação de replicação e gerenciamento de recursos para fornecer alta disponibilidade em um único datacenter. O SLA de disponibilidade do Armazenamento do Azure garante que em pelo menos 99,9% do tempo:

- As solicitações corretamente formatadas para adicionar, atualizar, ler e excluir dados serão ser bem-sucedidas e processadas corretamente.
- As contas de armazenamento terão conectividade com o gateway de Internet.

### <a name="replication"></a>Replicação

O Armazenamento do Azure facilita a durabilidade de dados mantendo várias cópias de todos os dados em diferentes unidades pelos subsistemas físicos de armazenamento totalmente independentes na região. Os dados são replicados de forma síncrona e todas as cópias são confirmadas antes da confirmação da gravação. O Armazenamento do Azure é altamente consistente, o que significa que há garantia de que as leituras reflitam as gravações mais recentes. Além disso, cópias de dados são verificadas continuamente para detectar e reparar bits corrompidos, uma ameaça frequentemente desconsiderada à integridade dos dados armazenados.

Os serviços se beneficiam da replicação apenas pelo uso do Armazenamento do Azure. O desenvolvedor do serviço não precisa fazer mais nada para se recuperar de uma falha local.

### <a name="resource-management"></a>Gerenciamento de recursos

As contas de armazenamento criadas depois de maio de 2014 podem ter até 500 TB (o máximo anterior era 200 TB). Se espaço adicional for necessário, os aplicativos deverão ser criados para usar várias contas de armazenamento.

### <a name="virtual-machine-disks"></a>Discos de máquina virtual

O disco de uma máquina virtual é armazenado como um blob de páginas no Armazenamento do Azure, o que dá a ele todas as mesmas propriedades de durabilidade e escalabilidade do armazenamento de Blobs. Esse design torna os dados em um disco de máquina virtual persistentes, mesmo se o servidor que executa a VM falhar e a VM precisa ser reiniciada em outro servidor.

## <a name="database"></a>Banco de dados

### <a name="sql-database"></a>Banco de dados SQL

O Banco de Dados SQL do Azure fornece banco de dados como serviço. Ele permite que aplicativos sejam provisionados rapidamente, insiram dados e consultem bancos de dados relacionais. Ele fornece muitos dos recursos e funcionalidades conhecidos do SQL Server, ao mesmo tempo que abstrai a carga de hardware, configuração, aplicação de patch e resiliência.

> [!NOTE]
> O Banco de Dados do SQL Azure não fornece paridade de recurso um para um com o SQL Server. Ele foi desenvolvido para atender a um conjunto diferente de requisitos — um que seja exclusivamente adequado para aplicativos de nuvem (escala elástica, banco de dados como serviço para reduzir custos de manutenção e assim por diante). Para mais informações, confira [Escolher uma opção do SQL Server de nuvem: Banco de dados do Azure SQL (PaaS) ou SQL Server em máquinas virtuais do Azure (IaaS)](/azure/sql-database/sql-database-paas-vs-sql-server-iaas/).

#### <a name="replication"></a>Replicação

O Banco de Dados SQL do Azure fornece resiliência interna a falhas no nível de nó. Todas as gravações em um banco de dados são replicadas automaticamente em dois ou mais nós em segundo plano por meio de uma técnica de confirmação de quorum. (A réplica primária, e pelo menos uma secundária, devem confirmar que a atividade foi gravada no log de transações antes que a transação seja considerada bem-sucedida e retorne.) No caso de falha de nó, o banco de dados faz failover automaticamente para uma das réplicas secundárias. Isso causa uma interrupção de conexão transitória para aplicativos cliente. Por esse motivo, todos os clientes do Banco de Dados SQL do Azure devem implementar alguma forma de tratamento de conexão transitória. Para saber mais, confira [Diretriz específica do serviço de repetição](/azure/best-practices-retry-service-specific/).

#### <a name="resource-management"></a>Gerenciamento de recursos

Cada banco de dados, quando criado, é configurado com um limite de tamanho superior. O tamanho máximo disponível atualmente é de 1 TB (os limites de tamanho variam com base em sua camada de serviço; confira [camadas de serviço e níveis de desempenho de Bancos de Dados do SQL Azure](/azure/sql-database/sql-database-resource-limits/#service-tiers-and-performance-levels). Quando um banco de dados atinge seu limite de tamanho superior, ele rejeita os comandos adicionais INSERT ou UPDATE. (Ainda é possível consultar e excluir dados).

Em um banco de dados, o Banco de Dados SQL do Azure usa uma malha para gerenciar recursos. No entanto, em vez de um controlador de malha, ele usa uma topologia de anel para detectar falhas. Cada réplica em um cluster tem dois vizinhos e é responsável por detectar quando eles são desativados. Quando uma réplica é desativada, seus vizinhos disparam um agente de reconfiguração para recriá-la em outro computador. A limitação de mecanismo é fornecida para garantir que um servidor lógico não use recursos demais em um computador ou exceda seus limites físicos.

### <a name="elasticity"></a>Elasticidade

Se o aplicativo exigir mais do que o limite de banco de dados de 1 TB, ele deverá implementar uma abordagem de escalonamento horizontal. Escale horizontalmente com o Banco de Dados SQL particionando manualmente os dados, também conhecido como fragmentação, entre vários bancos de dados SQL. Essa abordagem de escalonamento horizontal dá oportunidade de atingir o crescimento de custo praticamente linear com escala. O crescimento elástico ou a capacidade sob demanda podem crescer com custos incrementais conforme necessário, porque os bancos de dados são cobrados com base no tamanho real médio usado por dia, e não no tamanho máximo possível.

## <a name="sql-server-on-virtual-machines"></a>SQL Server em máquinas virtuais

A instalar o SQL Server (versão 2014 ou posterior) nas Máquinas Virtuais do Azure, você pode aproveitar os recursos tradicionais de disponibilidade do SQL Server. Esses recursos incluem Grupos de Disponibilidade do AlwaysOn e espelhamento de banco de dados. Observe que as VMs, o armazenamento e a rede do Azure têm características operacionais diferentes de uma infraestrutura de TI local não virtualizada. Uma implementação bem-sucedida de uma solução do SQL Server de HA/DR (alta disponibilidade/recuperação de desastre) no Azure requer que você compreenda essas diferenças e crie sua solução para acomodá-las.

### <a name="high-availability-nodes-in-an-availability-set"></a>Nós de alta disponibilidade em um conjunto de disponibilidade

Quando você implementa uma solução de alta disponibilidade no Azure, é possível usar o conjunto de disponibilidade no Azure para colocar os nós de alta disponibilidade em domínios de falha e domínios de atualização separados. Para ser claro, o conjunto de disponibilidade é um conceito do Azure. É uma prática recomendada que deve ser seguida para garantir que seus bancos de dados sejam, de fato, altamente disponibilizados, esteja você usando grupos de disponibilidade AlwaysOn, espelhamento de banco de dados ou outro meio. Não seguir essa prática recomendada pode fazê-lo supor erroneamente que seu sistema esteja altamente disponível. Na realidade, os nós podem falhar simultaneamente, pois pode acontecer de serem colocados no mesmo domínio de falha na região do Azure.

Essa recomendação não se aplica integralmente ao envio de logs. Como um recurso de recuperação de desastre, você deve garantir que os servidores estejam sendo executados em regiões separadas do Azure. Por definição, essas regiões são domínios de falha separados.

Para VMs de Serviços de Nuvem do Azure implantadas por meio do portal clássico para estarem no mesmo conjunto de disponibilidade, você deve implantá-las no mesmo Serviço de Nuvem. VMs implantadas por meio do Azure Resource Manager (o portal atual) não têm essa limitação. Para VMs implantadas do portal clássico no Serviço de Nuvem do Azure, apenas os nós no mesmo Serviço de Nuvem podem participar do mesmo conjunto de disponibilidade. Além disso, as VMs dos Serviços de Nuvem deve estar na mesma rede virtual para garantir que mantenham seus IPs mesmo após a recuperação do serviço. Isso evita interrupções de atualização de DNS.

### <a name="azure-only-high-availability-solutions"></a>Somente Azure: soluções de alta disponibilidade

Você pode ter uma solução de alta disponibilidade para seus bancos de dados do SQL Server no Azure usando Grupos de Disponibilidade AlwaysOn ou o espelhamento de banco de dados.

O diagrama a seguir mostra a arquitetura dos Grupos de Disponibilidade AlwaysOn em execução em Máquinas Virtuais do Azure. Esse diagrama foi retirado do artigo detalhado sobre o assunto, [Alta disponibilidade e recuperação de desastre para SQL Server nas Máquinas Virtuais do Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![Grupos de disponibilidade AlwaysOn no Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-1.png)

Você também pode provisionar automaticamente a implantação de um Grupo de Disponibilidade AlwaysOn completo em VMs do Azure usando o modelo AlwaysOn no portal do Azure. Para obter mais informações, consulte [Oferta do AlwaysOn do SQL Server na Galeria do Portal do Microsoft Azure](https://blogs.technet.microsoft.com/dataplatforminsider/2014/08/25/sql-server-alwayson-offering-in-microsoft-azure-portal-gallery/).

O diagrama a seguir demonstra o uso de espelhamento de banco de dados em Máquinas Virtuais do Azure. Ele também foi extraído do tópico detalhado sobre [Alta disponibilidade e recuperação de desastre para SQL Server nas Máquinas Virtuais do Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![Espalhamento de banco de dados no Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-2.png)

> [!NOTE]
> Ambas as arquiteturas exigem um controlador de domínio. No entanto, com o espelhamento de banco de dados, é possível usar certificados de servidor para eliminar a necessidade de um controlador de domínio.

## <a name="other-azure-platform-services"></a>Outros serviços da plataforma Azure

Os aplicativos criados no Azure se beneficiam dos recursos de plataforma para se recuperar de falhas locais. Em alguns casos, você pode executar ações específicas para aumentar a disponibilidade para seu cenário específico.

### <a name="service-bus"></a>Barramento de Serviço

Para atenuar uma interrupção temporária do Barramento de Serviço do Azure, considere a criação de uma fila durável do lado do cliente. Isso usa temporariamente um mecanismo de armazenamento local alternativo para armazenar mensagens que não podem ser adicionadas à fila de Barramento de Serviço. O aplicativo pode decidir como lidar com as mensagens armazenadas temporariamente depois que o serviço for restaurado. Para saber mais, confira [Práticas recomendadas para melhorias de desempenho usando o sistema de mensagens agenciado do Barramento de Serviço](/azure/service-bus-messaging/service-bus-performance-improvements/) e [Barramento de Serviço (recuperação de desastre)](recovery-loss-azure-region.md#other-azure-platform-services).

### <a name="hdinsight"></a>HDInsight

Os dados associados ao HDInsight do Azure são armazenados por padrão no armazenamento de Blobs do Azure. O Armazenamento do Azure especifica as propriedades de durabilidade e alta disponibilidade para o armazenamento de Blobs. O processamento de vários nós associado aos trabalhos do Hadoop MapReduce ocorre em um HDFS (Sistema de Arquivos Distribuídos Hadoop) transitório que é provisionado quando necessário pelo HDInsight. Os resultados de um trabalho MapReduce também são armazenados por padrão no armazenamento de Blobs do Azure para que os dados processados sejam duráveis e permaneçam altamente disponíveis depois que o cluster Hadoop for desprovisionado. Para saber mais, confira [HDInsight (recuperação de desastre)](recovery-loss-azure-region.md#other-azure-platform-services).

## <a name="checklists-for-local-failures"></a>Listas de verificação para falhas locais

### <a name="cloud-services-checklist"></a>Lista de verificação de Serviços de Nuvem

1. Examinar a seção Serviços de Nuvem deste documento.
2. Configure pelo menos duas instâncias para cada função.
3. Mantenha o estado no armazenamento durável, não em instâncias de função.
4. Trate corretamente o evento StatusCheck.
5. Encapsule alterações relacionadas em transações quando possível.
6. Verifique se as tarefas de função de trabalho são idempotentes e reinicializáveis.
7. Continue invocando operações até que tenham êxito.
8. Considere estratégias de dimensionamento automático.

### <a name="virtual-machines-checklist"></a>Lista de verificação de Máquinas Virtuais

1. Examinar a seção Máquinas Virtuais deste documento.
2. Não use a unidade D para armazenamento persistente.
3. Agrupe computadores em uma camada de serviço em um conjunto de disponibilidade.
4. Configure o balanceamento de carga e investigações opcionais.

### <a name="storage-checklist"></a>Lista de verificação de armazenamento

1. Examinar a seção Armazenamento deste documento.
2. Use várias contas de armazenamento quando dados ou largura de banda excederem as cotas.

### <a name="sql-database-checklist"></a>Lista de verificação de Banco de Dados SQL

1. Examinar a seção Banco de Dados SQL deste documento.
2. Implemente uma política de repetição para tratar de erros transitórios.
3. Use particionamento/fragmentação como uma estratégia de escalonamento horizontal.

### <a name="sql-server-on-virtual-machines-checklist"></a>Lista de verificação do SQL Server em Máquinas Virtuais

1. Examinar a seção SQL Server em máquinas virtuais deste documento.
2. Siga as recomendações anteriores para máquinas virtuais.
3. Use recursos de alta disponibilidade do SQL Server, como o AlwaysOn.

### <a name="service-bus-checklist"></a>Lista de verificação do Barramento de Serviço

1. Examinar a seção Barramento de Serviço deste documento.
2. Considere a criação de uma fila durável do lado do cliente como um backup.

### <a name="hdinsight-checklist"></a>Lista de verificação do HDInsight

1. Examinar a seção HDInsight deste documento.
2. Nenhuma etapa adicional de disponibilidade é necessária para falhas locais.

<!-- markdownlint-enable MD024 -->
