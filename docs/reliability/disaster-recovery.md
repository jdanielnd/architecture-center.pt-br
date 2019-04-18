---
title: Falha e recuperação de desastres para aplicativos do Azure
description: Abordagens de visão geral da recuperação de desastres no Azure
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: f00341b06b8092c798d6260317226190cbace66b
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646480"
---
# <a name="failure-and-disaster-recovery-for-azure-applications"></a>Falha e recuperação de desastres para aplicativos do Azure

*Recuperação de desastres* é o processo de restauração em andamento de uma perda catastrófica de funcionalidade do aplicativo.

Sua tolerância para funcionalidade reduzida durante um desastre é uma decisão de negócios que varia de um aplicativo para o próximo. Ele pode ser aceitável para alguns aplicativos para ficar completamente indisponível ou ser parcialmente disponível com funcionalidade reduzida ou processamento por um período de tempo com atraso. Para outros aplicativos, qualquer funcionalidade reduzida é inaceitável.

## <a name="disaster-recovery-plan"></a>Plano de recuperação de desastre

Comece criando um plano de recuperação. O plano será considerado concluído depois que ele foi totalmente testado. Inclua a pessoas, processos e aplicativos necessários para restaurar a funcionalidade dentro do contrato de nível de serviço (SLA) que você definiu para seus clientes.

Considere as sugestões a seguir ao criar e testar seu plano de recuperação de desastres:

- No seu plano, inclua o processo para contatar o suporte e escalar problemas. Essas informações ajudarão a evitar o tempo de inatividade prolongado conforme você trabalha o processo de recuperação pela primeira vez.
- Avalie o impacto das falhas de aplicativos sobre os negócios.
- Escolha uma arquitetura de recuperação entre regiões para aplicativos de missão crítica.
- Identifique um proprietário específico do plano de recuperação de desastres, incluindo automação e testes.
- Documente o processo, especialmente em alguma etapa manual.
- Automatize o processo tanto quanto possível.
- Estabelecer uma estratégia de backup para todos os referência dados transacionais e de teste de restauração de backup regularmente.
- Configure alertas para a pilha de serviços do Azure consumidos pelo seu aplicativo.
- Treinar a equipe de operações para executar o plano.
- Execute regularmente simulações de desastres para validar e aprimorar o plano.

Se você estiver usando [Azure Site Recovery](/azure/site-recovery/) para replicar máquinas virtuais (VMs), crie um plano totalmente automatizado de recuperação para fazer failover de todo o aplicativo.

## <a name="manual-responses"></a>Respostas manuais

Embora a automação é ideal, algumas estratégias para recuperação de desastres exigem respostas manuais.

### <a name="alerts"></a>Alertas

Monitore seu aplicativo, observando sinais de aviso que podem exigir intervenção proativa. Por exemplo, se o banco de dados SQL ou o Azure Cosmos DB limita repetidamente o seu aplicativo, talvez precise aumentar a capacidade do seu banco de dados ou otimizar suas consultas. Mesmo que o aplicativo consiga manipular erros de limitação de forma transparente, sua telemetria deve gerar um alerta para que você possa acompanhar.

Para limites de serviço e limites de cota, é recomendável configurar alertas em logs de diagnóstico e métricas de recursos do Azure. Quando possível, configure alertas em métricas, que são a menor latência que os logs de diagnóstico.

Por meio [Resource Health](/azure/service-health/resource-health-checks-resource-types), o Azure fornece alguns integridade interna verificações de status que podem ajudá-lo a diagnosticar problemas de limitação de serviço do Azure.

### <a name="failover"></a>Failover

Configure uma estratégia de recuperação de desastre para cada aplicativo do Azure e seus serviços do Azure. Estratégias de implantação aceitável para dar suporte à recuperação de desastre podem variar com base nos SLAs necessários para todos os componentes de cada aplicativo.  

O Azure fornece recursos diferentes em muitos serviços do Azure para permitir o failover manual, tais como [redis cache réplicas geográficas](/azure/azure-cache-for-redis/cache-how-to-geo-replication), ou para o failover automatizado, tais como [grupos de failover automático do SQL](/azure/sql-database/sql-database-auto-failover-group). Por exemplo: 

- Para um aplicativo que usa principalmente máquinas virtuais, você pode usar o Azure Site Recovery para as camadas web e lógica. Para obter mais informações, consulte [arquitetura de recuperação de desastre do Azure para](/azure/site-recovery/azure-to-azure-architecture). Para o SQL Server em máquinas virtuais, use [grupos de disponibilidade do SQL Server Always On](/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-dr).
- Para um aplicativo que usa o serviço de aplicativo e o banco de dados SQL, você pode usar um plano de serviço de aplicativo de camada menor configurado na região secundária, que é dimensionado automaticamente quando ocorrer um failover. Use grupos de failover para a camada de banco de dados.

Em qualquer cenário, uma [Gerenciador de tráfego do Azure](/azure/traffic-manager/traffic-manager-overview) perfil fornece o failover automático entre regiões. [Balanceadores de carga](/azure/load-balancer/load-balancer-overview) ou [gateways de aplicativo](/azure/application-gateway/overview) deve ser configurado na região secundária para dar suporte à disponibilidade mais rápida após o failover.

### <a name="operational-readiness-testing"></a>Testes de preparação operacional

Execute um teste de preparação operacional para o failover para a região secundária e para o failback para a região primária. Muitos serviços do Azure são compatíveis com failover manual ou failover de teste para exercícios de recuperação de desastre. Como alternativa, você pode simular uma interrupção desligar ou remover os serviços do Azure.

## <a name="application-failure"></a>Falha do aplicativo

Falhas de aplicativo são recuperáveis ou não recuperável. Você pode atenuar erros recuperáveis, mas erros irrecuperáveis desligarão o aplicativo.

- Algumas falhas podem ser tratadas de forma transparente pelo tratamento automaticamente de falhas e adotando ações alternativas. Por exemplo, o Gerenciador de tráfego controla as falhas que resultam do software subjacente do hardware ou sistema operacional na máquina virtual do host.
- Com alguns erros, o aplicativo pode continuar a tratar as solicitações do usuário com funcionalidade reduzida.
- Mais graves interrupções de serviço processem o aplicativo não está disponível.

Um sistema bem projetado separa as responsabilidades no nível de serviço &mdash; em tempo de design e em tempo de execução. Essa separação impede que uma interrupção do serviço dependente interrompa o aplicativo inteiro. Por exemplo, considere um aplicativo de comércio da web com os seguintes módulos:

![ADICIONAR LINK PARA A ARTE](./_images/disaster-recovery.png)

Se o banco de dados hospeda pedidos ficar inativo, o serviço de processamento de pedidos não pode processar transações de vendas. Dependendo da arquitetura, ele pode ser impossível para os serviços de submissão de pedido e processamento de pedidos continuar. No entanto, se os dados de produto são armazenados em um local diferente, o catálogo de produtos ainda estará disponível, mesmo que outras partes do aplicativo podem não estar disponíveis.

Cabe a você para determinar como o aplicativo vai informar os usuários sobre problemas temporários e criar notificações apropriadas para o sistema. No exemplo anterior, o aplicativo poderia permitir para exibir produtos e adicioná-los a um carrinho de compras. No entanto, quando o cliente tenta fazer uma compra, o aplicativo deve notificam que a funcionalidade de ordenação está temporariamente indisponível. Embora não seja ideal para o cliente, essa abordagem impede que uma interrupção de serviço de todo o aplicativo.

## <a name="data-corruption-and-restoration"></a>Restauração e corrupção de dados

Se um repositório de dados falhar, pode haver inconsistências de dados quando ele estiver disponível novamente, especialmente se os dados foram replicados. Noções básicas sobre o objetivo de tempo de recuperação (RTO) e o ponto de recuperação objetivo (RPO) de armazenamentos de dados replicados pode ajudá-lo a prever a quantidade de perda de dados.

Para entender se o failover entre regiões é iniciado manualmente ou pela Microsoft, examine os SLAs de serviço do Azure. Para serviços com nenhum SLA para failover entre regiões, a Microsoft normalmente decide quando executar failover e geralmente prioriza a recuperação de dados na região primária. Se os dados na região primária são considerados irrecuperáveis, Microsoft fará failover para a região secundária.

### <a name="restoring-data-from-backups"></a>Restaurar dados de backups

Os backups protegem você contra a perda de um componente do aplicativo devido a exclusão acidental ou corrupção de dados. Preservar a uma versão funcional do componente de um momento anterior, o que você pode usar para restaurá-lo.

Estratégias de recuperação de desastre não são uma substituição para backups, mas alguns cenários de recuperação de desastres de suporte de backups regulares de dados do aplicativo. Suas opções de armazenamento de backup devem se basear sua estratégia de recuperação de desastres.

A frequência de execução do processo de backup determina o RPO. Por exemplo, se você executar backups a cada hora e um desastre ocorre dois minutos antes do backup, você perderá 58 minutos de dados. Seu plano de recuperação de desastres deve incluir a como você abordará a perda de dados.

É comum para os dados em um repositório de dados para dados de referência em outro repositório. Por exemplo, considere um banco de dados SQL com uma coluna que se vincula a um blob no armazenamento do Azure. Se backups não acontecerem simultaneamente, o banco de dados pode ter um ponteiro para um blob que não foi feito backup antes da falha. O aplicativo ou o plano de recuperação de desastres deve implementar processos para tratar dessa inconsistência após uma recuperação.

> [!NOTE]
> Em alguns cenários, como a de VMs por backup usando o [Backup do Azure](/azure/backup/backup-azure-vms-first-look-arm), você pode restaurar apenas a partir de um backup na mesma região. Serviços de outros do Azure, como [Cache do Azure para Redis](/azure/azure-cache-for-redis/cache-how-to-geo-replication), fornecer backups replicados geograficamente, o que você pode usar para restaurar os serviços em regiões.

### <a name="azure-storage-and-azure-sql-database"></a>Banco de dados SQL do Azure e armazenamento do Azure

Azure armazena automaticamente os dados de armazenamento do Azure e banco de dados SQL três vezes em diferentes domínios de falha na mesma região. Se você usar a replicação geográfica, os dados serão armazenados mais três vezes em uma região diferente. No entanto, se os dados for corrompidos ou excluídos na cópia primária (por exemplo, devido ao erro de usuário), a replicação das alterações nas outras cópias.

Você tem duas opções para gerenciar possíveis dados corrompidos ou exclusão:

- **Crie uma estratégia de backup personalizada.** Você pode armazenar seus backups no Azure ou no local, dependendo de suas necessidades de negócios e das normas governamentais.
- **Use a opção de restauração point-in-time** para recuperar um banco de dados SQL.

#### <a name="azure-storage-recovery"></a>Recuperação de armazenamento do Azure

Você pode desenvolver um processo de backup personalizado para o armazenamento do Azure ou usar uma das diversas ferramentas de backup de terceiros.

O armazenamento do Azure fornece resiliência de dados por meio de réplicas automáticas, mas ele não impede que os usuários ou código do aplicativo devido à corrupção de dados. Manter dados de fidelidade após erro do usuário ou aplicativo requer técnicas mais avançadas, como copiar os dados para um local de armazenamento secundário com um log de auditoria. Você tem várias opções:

- [**Blobs de blocos.**](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs) Crie um instantâneo pontual de cada blob de blocos. Para cada instantâneo, você é cobrado somente para o armazenamento necessário para armazenar as diferenças no blob desde o estado do instantâneo anterior. Os instantâneos são dependentes no blob original, portanto, é recomendável copiar para outro blob ou até mesmo outra conta de armazenamento. Essa abordagem garante que os dados de backup são protegidos contra exclusão acidental. Use [AzCopy](/azure/storage/common/storage-use-azcopy) ou [do Azure PowerShell](/azure/storage/common/storage-powershell-guide-full) para copiar os blobs para outra conta de armazenamento.

    Para obter mais informações, consulte [Criar um instantâneo de um blob](/rest/api/storageservices/creating-a-snapshot-of-a-blob).

- [**Arquivos do Azure.**](/azure/storage/files/storage-files-introduction) Use [instantâneos de compartilhamento](/azure/storage/files/storage-snapshots-files), o AzCopy ou o PowerShell para copiar os arquivos para outra conta de armazenamento.
- [**Armazenamento de tabela do Azure.**](/azure/storage/tables/table-storage-overview) Use AzCopy para exportar os dados da tabela para outra conta de armazenamento em outra região.

#### <a name="sql-database-recovery"></a>Recuperação de banco de dados SQL

Para proteger sua empresa contra perda de dados, banco de dados SQL automaticamente executa uma combinação de backups de banco de dados completos semanais, backups de banco de dados diferenciais por hora, log de transações e backups cada 5 a 10 minutos. Para as camadas Basic, Standard e Premium banco de dados SQL, use point-in-time restore para restaurar um banco de dados para um momento anterior. Consulte os seguintes artigos para obter mais informações:

- [Recuperar um banco de dados SQL do Azure usando backups de banco de dados automatizados](/azure/sql-database/sql-database-recovery-using-backups)
- [Visão geral da continuidade dos negócios com o Banco de Dados SQL do Azure](/azure/sql-database/sql-database-business-continuity)

Outra opção é usar a replicação geográfica ativa para banco de dados SQL, que replica automaticamente alterações de banco de dados para bancos de dados secundários na região do Azure iguais ou diferente. Para obter mais informações, consulte [criando e usando a replicação geográfica ativa](/azure/sql-database/sql-database-active-geo-replication).

Você também pode usar uma abordagem mais manual para backup e restauração:

- Use o **cópia de banco de dados** comando para criar uma cópia de backup do banco de dados com consistência transacional.
- Use o serviço de importação/exportação do Azure SQL banco de dados, que dá suporte à exportação de bancos de dados para arquivos BACPAC (arquivos compactados contendo o esquema de banco de dados e dados associados) que são armazenados no armazenamento de BLOBs do Azure. Para proteger contra uma interrupção do serviço de toda a região, copie os arquivos BACPAC para uma região alternativa.

### <a name="sql-server-on-vms"></a>SQL Server em VMs

Para o SQL Server em execução em VMs, você tem duas opções: backups tradicionais e envio de logs.

- Com backups tradicionais, você pode restaurar para um ponto específico no tempo, mas o processo de recuperação é lento. Restauração de backups tradicionais exige que você comece com um backup completo inicial e, em seguida, aplique quaisquer backups incrementais.
- Você pode configurar uma sessão de envio de logs para atrasar a restauração de backups de log. Isso fornece uma janela para recuperação de erros feitos na réplica primária.

### <a name="azure-database-for-mysql-and-azure-database-for-postgresql"></a>Banco de dados do Azure para MySQL e banco de dados do Azure para PostgreSQL

No banco de dados do Azure para MySQL e banco de dados do Azure para PostgreSQL, o serviço de banco de dados faz automaticamente um backup a cada cinco minutos. Você pode usar esses backups automatizados para restaurar o servidor e seus bancos de dados de um ponto anterior no tempo para um novo servidor. Para obter mais informações, consulte:

- [Como fazer backup e restaurar um servidor no Banco de Dados do Azure para MySQL usando o portal do Azure](/azure/mysql/howto-restore-server-portal)
- [Como fazer backup e restaurar um servidor no banco de dados do Azure para PostgreSQL usando o portal do Azure](/azure/postgresql/howto-restore-server-portal)

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

O cosmos DB faz automaticamente um backup em intervalos regulares. Backups são armazenados separadamente em outro serviço de armazenamento e são replicados globalmente para se proteger contra desastres regionais. Caso exclua seu banco de dados ou coleção acidentalmente, é possível criar um tíquete de suporte ou ligar para o suporte do Azure a fim de restaurar os dados usando o último backup automático. Para obter mais informações, consulte [backup e restauração sob demanda no Azure Cosmos DB Online](/azure/cosmos-db/online-backup-and-restore).

### <a name="azure-virtual-machines"></a>Máquinas Virtuais do Azure

Para proteger máquinas virtuais do Azure de erros de aplicativo ou exclusão acidental, use [Backup do Azure](/azure/backup/). Os backups criados são consistentes em vários discos de VM. Além disso, o Cofre de Backup do Azure pode ser replicado entre regiões para dar suporte à recuperação de uma perda regional.

## <a name="network-outage"></a>Interrupção de rede

Quando partes da rede do Azure estiverem inacessíveis, você não poderá acessar seu aplicativo ou dados. Nessa situação, é recomendável criar a estratégia de recuperação de desastres para executar a maioria dos aplicativos com funcionalidade reduzida.

Se a redução da funcionalidade não é uma opção, as opções restantes são o tempo de inatividade do aplicativo ou o failover para uma região alternativa.

Em um cenário de funcionalidade reduzida:

- Se seu aplicativo não pode acessar os dados devido a uma interrupção de rede do Azure, você poderá executar localmente com funcionalidade reduzida do aplicativo usando dados armazenados em cache.
- Você poderá armazenar dados em um local alternativo até que a conectividade for restaurada.

## <a name="dependent-service-failure"></a>Falha do serviço dependente

Para cada serviço dependente, você deve compreender as implicações de uma interrupção do serviço e a maneira que o aplicativo responderá. Muitos serviços incluem recursos que dão suporte a resiliência e a disponibilidade, para que avaliar cada serviço de forma independente é provavelmente melhorará o seu plano de recuperação de desastres. Por exemplo, os Hubs de eventos do Azure suporta [failover](/azure/event-hubs/event-hubs-geo-dr#setup-and-failover-flow) para o namespace secundário.

## <a name="region-wide-service-disruptions"></a>Interrupções de serviço em toda a região

Número de falhas é gerenciável dentro da mesma região do Azure. No entanto, no caso improvável de uma interrupção do serviço de toda a região, as cópias localmente redundantes dos seus dados não estão disponíveis. Se você tiver habilitado a replicação geográfica, há três cópias adicionais dos seus blobs e tabelas em uma região diferente. Se a Microsoft declara a região como perdida, o Azure remapeia todas as entradas DNS para a região secundária.

> [!NOTE]
> Esse processo ocorre somente para interrupções de serviço em toda a região e não está dentro do seu controle. Considere o uso do [Azure Site Recovery](/azure/site-recovery/) para alcançar melhor RPO e RTO. Usando o Site Recovery, você decide o que é uma interrupção aceitável e quando executar failover para as VMs replicadas.

Sua resposta a uma interrupção do serviço de toda a região depende de sua implantação e seu plano de recuperação de desastres.

- Como uma estratégia de controle de custos, para aplicativos não críticos que não exigem um tempo de recuperação garantido, ele pode fazer sentido para reimplantar uma região diferente.
- Para aplicativos que são hospedados em outra região com funções implantadas, mas não distribuir o tráfego entre regiões (*implantação ativo/passivo*), alterne para o serviço hospedado secundário na região alternativa.
- Para aplicativos que têm uma implantação secundária completa em outra região (*implantação ativo/ativo*), rotear o tráfego para essa região.

Para saber mais sobre como recuperar de uma interrupção de serviço em toda a região, consulte [recuperar de uma interrupção do serviço de toda a região](../resiliency/recovery-loss-azure-region.md).

### <a name="vm-recovery"></a>Recuperação de VM

Para aplicativos críticos, planeje a recuperação de VMs em caso de uma interrupção do serviço de toda a região.

- Use o Backup do Azure ou outro método de backup para criar backups entre regiões consistentes com aplicativos. (Replicação do Cofre de Backup deve ser configurada no momento da criação).
- Use o Site Recovery para replicar entre regiões para failover de aplicativo em um único clique e teste de failover.
- Use o Gerenciador de tráfego para automatizar o failover de tráfego do usuário para outra região.

Para obter mais informações, consulte [recuperar de uma interrupção do serviço de toda a região, as máquinas virtuais](../resiliency/recovery-loss-azure-region.md#virtual-machines).

### <a name="storage-recovery"></a>Recuperação de armazenamento

Para proteger seu armazenamento em caso de uma interrupção do serviço de toda a região:

- Use o armazenamento com redundância geográfica.
- Sabe onde o armazenamento é replicado geograficamente. Isso afeta onde você implanta outras instâncias de dados que exigem afinidade regional com seu armazenamento.
- Verificar dados para fins de consistência após o failover e, se necessário, restaurar de um backup.

Para obter mais informações, consulte [criando aplicativos altamente disponíveis usando RA-GRS](/azure/storage/common/storage-designing-ha-apps-with-ragrs).

### <a name="sql-database-and-sql-server"></a>Banco de dados SQL e SQL Server

O Banco de dados SQL do Azure fornece dois tipos de recuperação:

- Use a restauração geográfica para restaurar um banco de dados de uma cópia de backup em outra região. Para obter mais informações, consulte [recuperar um banco de dados SQL do Azure usando backups automatizados do banco de dados](/azure/sql-database/sql-database-recovery-using-backups).
- Use a replicação geográfica ativa para fazer failover para um banco de dados secundário. Para obter mais informações, consulte [criando e usando a replicação geográfica ativa](/azure/sql-database/sql-database-active-geo-replication).

Para o SQL Server em execução em VMs, consulte [alta disponibilidade e recuperação de desastres para SQL Server em máquinas virtuais Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

## <a name="service-specific-guidance"></a>Diretriz específica do serviço

Os artigos a seguir descrevem a recuperação de desastres para serviços específicos do Azure:

| Serviço                       | Artigo                                                                                                                                                                                       |
|-------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Banco de Dados do Azure para MySQL      | [Visão geral da continuidade dos negócios com o Banco de Dados do Azure para MySQL](/azure/mysql/concepts-business-continuity)                                                  |
| Banco de Dados do Azure para PostgreSQL | [Visão geral da continuidade dos negócios com o Banco de Dados do Azure para PostgreSQL](/azure/postgresql/concepts-business-continuity)                                        |
| Serviços de nuvem do Azure          | [O que fazer no caso de uma interrupção de serviço do Azure que afete os Serviços de Nuvem do Azure](/azure/cloud-services/cloud-services-disaster-recovery-guidance) |
| Cosmos DB                     | [Alta disponibilidade com o Azure Cosmos DB](/azure/cosmos-db/high-availability)                                                                                |
| Cofre da Chave do Azure               | [Redundância e disponibilidade de Azure Key Vault](/azure/key-vault/key-vault-disaster-recovery-guidance)                                                        |
| Armazenamento do Azure                 | [Recuperação de desastre e failover de conta de armazenamento (versão prévia) no Armazenamento do Azure](/azure/storage/common/storage-disaster-recovery-guidance)                       |
| Banco de dados SQL                  | [Restaurar um banco de dados SQL ou fazer failover para uma região secundária](/azure/sql-database/sql-database-disaster-recovery)                                       |
| Máquinas Virtuais              | [No caso de um Azure interrupção de serviço o que afeta a nuvem do Azure](/azure/cloud-services/cloud-services-disaster-recovery-guidance)               |
| Rede Virtual do Azure         | [Rede Virtual – Continuidade de Negócios](/azure/virtual-network/virtual-network-disaster-recovery-guidance)                                                  |

## <a name="next-steps"></a>Próximos passos

- [Recuperar dados corrompidos ou exclusão acidental](../resiliency/recovery-data-corruption.md)
- [Recuperar de uma interrupção do serviço de toda a região](../resiliency/recovery-loss-azure-region.md)