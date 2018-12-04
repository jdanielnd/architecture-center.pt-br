---
title: Lista de verificação de resiliência para serviços do Azure
description: Lista de verificação que fornece orientação de resiliência para vários serviços do Azure.
author: petertaylor9999
ms.date: 03/02/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: 53a37595bd6e70fa3a43e9a72b2ae47d2225009f
ms.sourcegitcommit: 1b5411f07d74f0a0680b33c266227d24014ba4d1
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/26/2018
ms.locfileid: "52305920"
---
# <a name="resiliency-checklist-for-specific-azure-services"></a>Lista de verificação de resiliência para serviços específicos do Azure

Resiliência é a capacidade de um sistema de se recuperar de falhas e continuar funcionando, além de ser um dos [pilares da qualidade de software](../guide/pillars.md). Cada tecnologia tem seus próprios modos de falha específicos, o que você deve considerar ao projetar e implementar seu aplicativo. Use esta lista de verificação para revisar as considerações de resiliência para serviços específicos do Azure. Revise também a [lista de verificação de resiliência geral](./resiliency.md).

## <a name="app-service"></a>Serviço de Aplicativo

**Use a camada Standard ou Premium.** Essas camadas dão suporte a slots de preparo e backups automáticos. Para obter mais informações, consulte [Visão geral aprofundada de planos do Serviço de Aplicativo do Azure](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/)

**Evite expandir ou reduzir verticalmente.** Em vez disso, selecione uma camada e o tamanho de instância que atendem aos seus requisitos de desempenho sob carga típica e, em seguida, [aumente](/azure/app-service-web/web-sites-scale/) as instâncias para manipular as alterações no volume de tráfego. Expandir e reduzir verticalmente pode disparar uma reinicialização do aplicativo.  

**Armazene a configuração como configurações de aplicativo.** Use as configurações de aplicativo para manter definições de configuração como configurações de aplicativo. Defina as configurações em seus modelos do Resource Manager ou usando o PowerShell, para que você possa aplicá-las como parte de um processo automatizado de implantação/atualização, que é mais confiável. Para obter mais informações, consulte [Configurar aplicativos Web no Serviço de Aplicativo do Azure](/azure/app-service-web/web-sites-configure/).

**Crie Planos do Serviço de Aplicativo separados para produção e teste.** Não use slots em sua implantação de produção para teste.  Todos os aplicativos dentro do mesmo Plano do Serviço de Aplicativo compartilham as mesmas instâncias de VM. Se você colocar a produção e implantações de teste no mesmo plano, isso poderá afetar negativamente a implantação de produção. Por exemplo, testes de carga podem prejudicar o site de produção dinâmica. Colocando as implantações de teste em um plano separado, você as isola da versão de produção.  

**Separe aplicativos Web de APIs da Web.** Se sua solução tem um front-end da Web e uma API da Web, considere a possibilidade de decompô-los em aplicativos de Serviço de Aplicativo separados. Esse design torna mais fácil para decompor a solução pela carga de trabalho. Você pode executar o aplicativo Web e a API em Planos do Serviço de Aplicativo separados, de modo que eles possam ser dimensionados de forma independente. Se você não precisar desse nível de escalabilidade inicialmente, poderá implantar os aplicativos no mesmo plano e movê-los para planos separados posteriormente, se necessário.

**Evite usar o recurso de Serviço de Aplicativo de backup para fazer backup de Bancos de Dados SQL do Azure.** Em vez disso, use os [Backups automatizados do Banco de Dados SQL][sql-backup]. O backup do Serviço de Aplicativo exporta o banco de dados para um arquivo .bacpac do SQL, que custa DTUs.  

**Implante em um slot de preparo.** Crie um slot de implantação para preparo. Implante atualizações de aplicativo para o slot de preparo e verifique a implantação antes de trocá-la para produção. Isso reduz a chance de uma atualização inválida em produção. Isso também garante que todas as instâncias sejam aquecidas antes de serem trocadas para produção. Muitos aplicativos têm um tempo significativo de aquecimento e de resfriamento. Para obter mais informações, consulte [Configurar ambientes de preparo para aplicativos Web no Serviço de Aplicativo do Azure](/azure/app-service-web/web-sites-staged-publishing/).

**Crie um slot de implantação para conter a LKG (última implantação válida).** Quando você implanta uma atualização para produção, mova a implantação de produção anterior para o slot da LKG. Isso torna mais fácil reverter uma implantação incorreta. Se você descobrir um problema posteriormente, você poderá reverter rapidamente para a versão LKG. Para obter mais informações, veja [Basic web application](../reference-architectures/app-service-web-app/basic-web-app.md) (Aplicativo Web básico).

**Habilite o log de diagnósticos**, incluindo o log de aplicativo e o log do servidor Web. O registro em log é importante para monitoramento e diagnóstico. Consulte [Habilitar o registro de log de diagnóstico para aplicativos Web no Serviço de Aplicativo do Azure](/azure/app-service-web/web-sites-enable-diagnostic-log/)

**Registre em log para o Armazenamento de Blobs.** Isso torna mais fácil coletar e analisar os dados.

**Criar uma conta de armazenamento separada para logs.** Não use a mesma conta de armazenamento para logs e dados de aplicativo. Isso ajuda a impedir o registro em log de reduzir o desempenho do aplicativo.

**Monitore o desempenho.** Use um serviço de monitoramento de desempenho como o [New Relic](https://newrelic.com/) ou o [Application Insights](/azure/application-insights/app-insights-overview/) para monitorar o desempenho e o comportamento do aplicativo sob carga.  O monitoramento de desempenho fornece informações sobre o aplicativo em tempo real. Ele permite a você diagnosticar problemas e executar a análise de causa raiz de falhas.

## <a name="application-gateway"></a>Gateway de Aplicativo

**Provisione pelo menos duas instâncias.** Implante um Gateway de Aplicativo com pelo menos duas instâncias. Uma única instância é um ponto único de falha. Use duas ou mais instâncias para redundância e escalabilidade. Para se qualificar para o [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway), você deverá provisionar duas ou mais instâncias médias ou maiores.

## <a name="cosmos-db"></a>Cosmos DB

**Replique o banco de dados entre regiões.** O Cosmos DB permite associar qualquer número de regiões do Azure a uma conta de banco de dados do Cosmos DB. Um banco de dados do Azure Cosmos DB pode ter várias regiões de leitura e uma de gravação. Se houver uma falha na região de gravação, você poderá ler de outra réplica. O SDK do cliente lida com isso automaticamente. Você também pode fazer failover da região de gravação para outra região. Para obter mais informações, confira [Como distribuir os dados globalmente com o Azure Cosmos DB](/azure/cosmos-db/distribute-data-globally).

## <a name="event-hubs"></a>Hubs de Eventos

**Usar pontos de verificação**.  Um consumidor de evento deve gravar sua posição atual no armazenamento persistente em um intervalo predefinido. Dessa forma, se o consumidor apresentar uma falha (por exemplo, falhas do consumidor ou do host), uma nova instância poderá retomar a leitura do fluxo da última posição gravada. Para obter mais informações, consulte [Consumidores de evento](/azure/event-hubs/event-hubs-features#event-consumers).

**Tratar mensagens duplicadas.** Se um consumidor de evento falhar, o processamento de mensagem é retomado do último ponto de verificação gravado. As mensagens que já foram processadas após o último ponto de verificação serão processadas novamente. Portanto, sua lógica de processamento de mensagem deve ser idempotente, ou o aplicativo deve ser capaz de eliminar a duplicação de mensagens.

**Tratar exceções.** . Um consumidor de evento normalmente processa um lote de mensagens em um loop. Você deve tratar exceções neste loop de processamento para evitar a perda de um lote inteiro de mensagens se uma única mensagem causar uma exceção.

**Use uma fila de mensagens mortas.** Se o processamento de uma mensagem resultar em uma falha não transitória, coloque a mensagem sobre uma fila de mensagens mortas, para que você possa acompanhar o status. Dependendo do cenário, você pode tentar novamente a mensagem mais tarde, aplicar uma transação de compensação ou executar alguma outra ação. Observe que os Hubs de Eventos não têm nenhuma funcionalidade interna de fila de mensagens mortas. Você pode usar o Armazenamento de Filas do Azure ou o Barramento de Serviço para implementar uma fila de mensagens mortas ou usar o Azure Functions ou outro mecanismo de eventos.  

**Implementar recuperação de desastres efetuando failover para um namespace secundário de Hubs de Eventos.** Para mais informações consulte [Recuperação de desastre em área geográfica dos Hubs de Eventos](/azure/event-hubs/event-hubs-geo-dr).

## <a name="redis-cache"></a>Cache Redis

**Configurar a replicação geográfica**. A Replicação geográfica fornece um mecanismo para vincular duas instâncias de Cache Redis do Azure de camada Premium. Os dados gravados no cache primário são replicados para um cache secundário de somente leitura. Para obter mais informações, consulte [Como configurar a replicação geográfica do Cache Redis do Azure](/azure/redis-cache/cache-how-to-geo-replication)

**Configurar a persistência de dados.** A persistência do Redis permite persistir os dados armazenados no Redis. Você também pode tirar instantâneos e fazer backup dos dados, que podem ser carregados em caso de falha de hardware. Para obter mais informações, consulte [Como configurar a persistência de dados para um Cache Redis do Azure Premium](/azure/redis-cache/cache-how-to-premium-persistence)

Caso esteja usando o Cache Redis como um cache de dados temporário e não como um armazenamento persistente, essas recomendações podem não ser aplicadas. 

## <a name="search"></a>Search

**Provisione mais de uma réplica.** Use pelo menos duas réplicas para alta disponibilidade de leitura ou três para alta disponibilidade de leitura/gravação.

**Configure indexadores para implantações de várias regiões.** Se você tiver uma implantação de várias regiões, considere as opções para a continuidade na indexação.

  * Se a fonte de dados é com replicação geográfica, você geralmente deve apontar cada indexador de cada serviço do Azure Search regional para a respectiva réplica de fonte de dados local. No entanto, essa abordagem não é recomendada para grandes conjuntos de dados armazenados no Banco de Dados SQL do Azure. O motivo é que o Azure Search não pode executar a indexação incremental de réplicas de Banco de Dados SQL secundárias, somente de réplicas primárias. Em vez disso, aponte todos os indexadores para a réplica primária. Após um failover, aponte os indexadores do Azure Search para a nova réplica primária.  
  * Se a fonte de dados não é com replicação geográfica, aponte vários indexadores na fonte de dados, de forma que os serviços do Azure Search façam a indexação da fonte de dados de modo contínuo e independente. Para obter mais informações, consulte [Considerações de desempenho e otimização do Azure Search][search-optimization].

## <a name="service-bus"></a>Barramento de Serviço

**Use a camada Premium para cargas de trabalho de produção**. [Mensagens Premium do Barramento de Serviço](/azure/service-bus-messaging/service-bus-premium-messaging) fornecem recursos de processamento dedicado e reservado e a capacidade de memória para dar suporte a desempenho e taxa de transferência previsíveis. A camada de Mensagens Premium também oferece acesso a novos recursos que estão disponíveis somente para clientes Premium no início. Você pode decidir o número de unidades do sistema de mensagens com base em cargas de trabalho esperadas.

**Tratar mensagens duplicadas**. Se um editor falha imediatamente após enviar uma mensagem ou enfrenta problemas de rede ou do sistema, pode deixar de registrar erroneamente que a mensagem foi entregue e pode enviar a mesma mensagem ao sistema duas vezes. O Barramento de Serviço pode lidar com esse problema habilitando a detecção de duplicatas. Para saber mais, confira [Detecção de duplicatas](/azure/service-bus-messaging/duplicate-detection).

**Tratar exceções**. As APIs de mensagens geram exceções quando ocorre um erro de usuário, um erro de configuração ou outro erro. O código do cliente (remetentes e receptores) deve tratar essas exceções em seu código. Isso é especialmente importante no processamento em lotes, em que o tratamento de exceção pode ser usado para evitar a perda de um lote inteiro de mensagens. Para saber mais, veja [Exceções de mensagens do Barramento de Serviço](/azure/service-bus-messaging/service-bus-messaging-exceptions).

**Política de repetição**. O Barramento de Serviço permite que você escolha a melhor política de repetição para os aplicativos. A política padrão é permitir no máximo nove tentativas de repetição e aguardar 30 segundos, mas isso pode ser ajustado. Para saber mais, confira [Política de repetição – Barramento de Serviço](/azure/architecture/best-practices/retry-service-specific#service-bus).

**Use uma fila de mensagens mortas**. Se uma mensagem não pode ser processada ou entregue a nenhum receptor após várias tentativas, é movida para uma fila de mensagens mortas. Implemente um processo para ler mensagens da fila de mensagens mortas, inspecioná-las e corrigir o problema. Dependendo do cenário, você pode repetir a mensagem como está, fazer alterações e tentar novamente ou descartar a mensagem. Para saber mais, confira [Visão geral das filas de mensagens mortas do Barramento de Serviço](/azure/service-bus-messaging/service-bus-dead-letter-queues).

**Usar a recuperação de desastre em área geográfica**. A recuperação de desastre em área geográfica garante que o processamento de dados continue a operar em uma região ou datacenter diferente, se uma região inteira do Azure ou o datacenter se tornar indisponível devido a um desastre. Para mais informações consulte [Recuperação de desastre em área geográfica do Barramento de Serviço do Azure](/azure/service-bus-messaging/service-bus-geo-dr).


## <a name="storage"></a>Armazenamento

**Para dados de aplicativo, use RA-GRS (armazenamento com redundância geográfica com acesso de leitura).** O armazenamento de RA-GRS replica os dados para uma região secundária e fornece acesso somente leitura da região secundária. Se há uma interrupção de armazenamento na região primária, o aplicativo pode ler os dados da região secundária. Para obter mais informações, consulte [Replicação do Armazenamento do Azure](/azure/storage/storage-redundancy/).

**Para discos de VM, use os Managed Disks.** Os [Managed Disks][managed-disks] fornecem melhor confiabilidade para as VMs em um conjunto de disponibilidade porque os discos ficam suficientemente isolados entre si para evitar pontos únicos de falha. Além disso, os Managed Disks não estão sujeitos a limites de IOPS de VHDs criados em uma conta de armazenamento. Para saber mais, consulte [Gerenciar a disponibilidade de máquinas virtuais Windows no Azure][vm-manage-availability].

**Para o Armazenamento de Filas, crie uma fila de backup em outra região.** Para o armazenamento de fila, a utilidade de uma réplica somente leitura é limitada, porque não é possível enfileirar itens nem removê-los da fila. Em vez disso, crie uma fila de backup em uma conta de armazenamento em outra região. Se houver uma interrupção de armazenamento, o aplicativo poderá usar a fila de backup até que a região primária fique disponível novamente. Dessa forma, o aplicativo ainda poderá processar novas solicitações.  

## <a name="sql-database"></a>Banco de dados SQL

**Use a camada Standard ou Premium.** Essas camadas fornecem um período de Recuperação Pontual mais longo (35 dias). Para obter mais informações, consulte [Opções e desempenho de Banco de Dados SQL](/azure/sql-database/sql-database-service-tiers/).

**Habilite a auditoria do Banco de Dados SQL.** A auditoria pode ser usada para diagnosticar erros humanos ou ataques mal-intencionados. Para saber mais, confira [Introdução à Auditoria do Banco de Dados SQL](/azure/sql-database/sql-database-auditing-get-started/).

**Use a Replicação Geográfica Ativa**. Use a replicação geográfica ativa para criar uma réplica secundária legível em uma região diferente.  Se o seu banco de dados primário falhar ou simplesmente precisar ser colocado offline, faça um failover manual para qualquer o banco de dados secundário.  O banco de dados secundário permanece somente leitura até você fazer o failover.  Para saber mais, confira [Replicação Geográfica Ativa para o Banco de Dados SQL](/azure/sql-database/sql-database-geo-replication-overview/).

**Use fragmentação.** Considere o uso de fragmentação para particionar o banco de dados horizontalmente. A fragmentação pode proporcionar o isolamento de falhas. Para saber mais, confira [Expansão com o Banco de Dados SQL do Azure](/azure/sql-database/sql-database-elastic-scale-introduction/).

**Use a Recuperação Pontual para se recuperar de erro humano.**  A Recuperação Pontual retorna seu banco de dados para um ponto anterior no tempo. Para obter mais informações, consulte [Recuperar um Banco de Dados SQL do Azure usando backups de banco de dados automatizados][sql-restore].

**Use a restauração geográfica para recuperar-se de uma interrupção de serviço.** A restauração geográfica restaura um banco de dados de um backup com redundância geográfica.  Para obter mais informações, consulte [Recuperar um Banco de Dados SQL do Azure usando backups de banco de dados automatizados][sql-restore].

## <a name="sql-data-warehouse"></a>SQL Data Warehouse

**Não desabilite o backup geográfico.** Por padrão, o SQL Data Warehouse usa um backup completo de seus dados a cada 24 horas para a recuperação de desastres. Não é recomendável desativar esse recurso. Para obter mais informações, veja [Backups geográficos](/azure/sql-data-warehouse/backup-and-restore#geo-backups).

## <a name="sql-server-running-in-a-vm"></a>SQL Server em execução em uma VM

**Replicar o banco de dados.** Use os Grupos de Disponibilidade Always On do SQL Server para replicar o banco de dados. Fornece alta disponibilidade se uma instância do SQL Server falha. Para obter mais informações, consulte [Executar VMs do Windows para um aplicativo de N camadas](../reference-architectures/virtual-machines-windows/n-tier.md)

**Fazer backup do banco de dados**. Se você já estiver usando [Backup do Azure](https://azure.microsoft.com/documentation/services/backup/) para fazer backup de suas VMs, considere usar o [Backup do Azure para cargas de trabalho do SQL Server usando o DPM](/azure/backup/backup-azure-backup-sql/). Com essa abordagem, há uma função de administrador de backup para a organização e um procedimento de recuperação unificado para as VMs e o SQL Server. Caso contrário, use o [Backup gerenciado pelo SQL Server para o Microsoft Azure](https://msdn.microsoft.com/library/dn449496.aspx).

## <a name="traffic-manager"></a>Gerenciador de Tráfego

**Faça o failback manual.** Após um failover do Gerenciador de Tráfego, faça o failback manual em vez de automaticamente. Verifique se todos os subsistemas de aplicativo estão íntegros antes de fazer o failback.  Caso contrário, você pode criar uma situação em que o aplicativo fica alternando entre os data centers. Para obter mais informações, consulte [Executar VMs em várias regiões para ter alta disponibilidade](../reference-architectures/virtual-machines-windows/multi-region-application.md).

**Criar um ponto de extremidade de investigação de integridade.** Crie um ponto de extremidade personalizado que relata a integridade geral do aplicativo. Isso permite que o Gerenciador de Tráfego faça failover se qualquer caminho crítico falha, não apenas o front-end. O ponto de extremidade deve retornar um código de erro HTTP se qualquer dependência crítica está não íntegra ou inacessível. No entanto, não relate erros para serviços não críticos. Caso contrário, a investigação de integridade poderá disparar o failover quando ele não for necessário, criando falsos positivos. Para obter mais informações, consulte [Monitoramento e failover do ponto de extremidade do Gerenciador de Tráfego](/azure/traffic-manager/traffic-manager-monitoring/).

## <a name="virtual-machines"></a>Máquinas Virtuais

**Evite executar uma carga de trabalho de produção em uma única VM.** Uma única implantação de VM não é resiliente para manutenção (planejada ou não). Em vez disso, coloque várias VMs em um conjunto de disponibilidade ou [conjunto de dimensionamento de VM](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/), com um balanceador de carga na frente.

**Especifique um conjunto de disponibilidade ao provisionar a VM.** Atualmente, não há nenhuma maneira de adicionar uma VM a um conjunto de disponibilidade depois que a VM é provisionada. Quando você adiciona uma nova VM a um conjunto de disponibilidade existente, verifique se você criou um adaptador de rede para a VM e adicione o adaptador de rede ao pool de endereços de back-end no balanceador de carga. Caso contrário, o balanceador de carga não roteará o tráfego de rede para essa VM.

**Coloque cada camada de aplicativo em um conjunto de disponibilidade separado.** Em um aplicativo de N camadas, não coloque as VMs de camadas diferentes no mesmo conjunto de disponibilidade. VMs em um conjunto de disponibilidade são colocadas em FDs (domínios de falha) e UDs (domínios de atualização). No entanto, para obter o benefício de redundância de FDs e UDs, todas as VMs no conjunto de disponibilidade devem ser capazes de lidar com as mesmas solicitações de cliente.

**Replique VMs usando o Azure Site Recovery.** Quando você replicar VMs do Azure usando a [Recuperação de Site][site-recovery], todos os discos de VM serão replicados continuamente para a região de destino assincronamente. Os pontos de replicação são criados a cada poucos minutos. Isso fornece um RPO (Objetivo de Ponto de Recuperação) na ordem de minutos. Realize a recuperação de desastre quantas vezes quiser sem afetar o aplicativo de produção ou a replicação contínua. Para saber mais, confira [Realizar uma análise detalhada da recuperação de desastre para o Azure][site-recovery-test].

**Escolha o tamanho de VM correto com base nos requisitos de desempenho.** Ao mover uma carga de trabalho existente para o Azure, comece com o tamanho da VM que mais se aproxima de seus servidores locais. Em seguida, meça o desempenho da carga de trabalho real com relação à CPU, memória e IOPS e ajuste o tamanho, se necessário. Isso ajuda a garantir que o aplicativo se comporte como esperado em um ambiente de nuvem. Além disso, se você precisar de várias NICs, esteja ciente do limite de NIC para cada tamanho.

**Use Managed Disks para VHDs.** Os [Managed Disks][managed-disks] fornecem melhor confiabilidade para as VMs em um conjunto de disponibilidade porque os discos ficam suficientemente isolados entre si para evitar pontos únicos de falha. Além disso, os Managed Disks não estão sujeitos a limites de IOPS de VHDs criados em uma conta de armazenamento. Para saber mais, consulte [Gerenciar a disponibilidade de máquinas virtuais Windows no Azure][vm-manage-availability].

**Instale aplicativos em um disco de dados, não no disco de SO.** Caso contrário, você poderá atingir o limite de tamanho do disco.

**Use o Backup do Azure para fazer backup de VMs.** Os backups protegem contra perda de dados acidental. Para obter mais informações, consulte [Proteger VMs do Azure com um cofre de serviços de recuperação](/azure/backup/backup-azure-vms-first-look-arm/).

**Habilite logs de diagnóstico**, incluindo métricas de integridade básicas, logs de infraestrutura e [diagnóstico de inicialização][boot-diagnostics]. O diagnóstico de inicialização poderá ajudar a diagnosticar uma falha de inicialização se sua VM entrar em um estado não inicializável. Para obter mais informações, consulte [Visão Geral dos Logs de Diagnóstico do Azure][diagnostics-logs].

**Use a Extensão AzureLogCollector.** (Somente VMs do Windows.) Essa extensão agrega logs da plataforma do Azure e carrega-os no Armazenamento do Azure, sem que o operador faça logon remotamente na VM. Para obter mais informações, consulte [Extensão do AzureLogCollector](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="virtual-network"></a>Rede Virtual

**Para incluir endereços IP públicos na lista de permissões ou bloqueá-los, adicione um NSG à sub-rede.** Bloqueie o acesso de usuários mal-intencionados ou permita o acesso somente de usuários que têm o privilégio de acessar o aplicativo.  

**Crie uma investigação de integridade personalizada.** As investigações de integridade do balanceador de carga podem testar HTTP ou TCP. Se uma VM for executada em um servidor HTTP, a investigação HTTP será um melhor indicador do status de integridade do que uma investigação TCP. Para uma investigação HTTP, use um ponto de extremidade personalizado que relata a integridade geral do aplicativo, incluindo todas as dependências críticas. Para obter mais informações, veja [Visão geral do Azure Load Balancer](/azure/load-balancer/load-balancer-overview/).

**Não bloqueie a investigação de integridade.** A investigação de integridade do balanceador de carga é enviada de um endereço IP conhecido, 168.63.129.16. Não bloqueie o tráfego de ou para esse IP em nenhuma política de firewall ou regra de NSG (Grupo de Segurança de Rede). Bloquear a investigação de integridade faria com que o balanceador de carga remova a VM da rotação.

**Habilite o registro em log do balanceador de carga.** Os logs de mostram quantas muitas VMs no back-end não estão recebendo o tráfego de rede devido a respostas de investigação com falha. Para obter mais informações, consulte [Log Analytics para o Azure Load Balancer](/azure/load-balancer/load-balancer-monitor-log/).

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[search-optimization]: /azure/search/search-performance-optimization/
[site-recovery]: /azure/site-recovery/
[site-recovery-test]: /azure/site-recovery/site-recovery-test-failover-to-azure
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
