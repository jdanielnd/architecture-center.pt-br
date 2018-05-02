---
title: Recuperação de dados corrompidos ou exclusão acidental
description: Artigo com noções básicas sobre como recuperar dados corrompidos ou de exclusão acidental e como criar aplicativos resilientes, altamente disponíveis, com tolerância a falhas, bem como planejamento de recuperação de desastres
author: MikeWasson
ms.date: 01/10/2018
ms.openlocfilehash: b0716de39fe69d607b9a63e51356d28bbcdbfeae
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/16/2018
---
# <a name="recover-from-data-corruption-or-accidental-deletion"></a>Recuperação de dados corrompidos ou exclusão acidental 

Parte de um plano robusto da continuidade de negócios é ter um plano, caso os dados sejam corrompidos ou excluídos acidentalmente. Veja a seguir informações sobre recuperação depois que dados forem corrompidos ou excluídos acidentalmente devido a erros do aplicativo ou do operador.

## <a name="virtual-machines"></a>Máquinas Virtuais

Para proteger as Máquinas Virtuais do Azure (VMs) de erros de aplicativo ou exclusão acidental, use o [Backup do Azure](/azure/backup/). O Backup do Azure permite a criação de backups que são consistentes em vários discos de VM. Além disso, o Cofre de Backup pode ser replicado entre regiões para fornecer recuperação de perda de região.

## <a name="storage"></a>Armazenamento

O Armazenamento do Azure fornece resiliência de dados por meio de réplicas automatizadas. No entanto, isso não impede que o código do aplicativo ou os usuários corrompam dados, seja acidental ou maliciosamente. Manter a fidelidade dos dados no caso de erro do usuário ou aplicativo requer técnicas mais avançadas, como copiar os dados para um local de armazenamento secundário com um log de auditoria. 

- **Blobs de blocos**. Crie um instantâneo pontual de cada blob de blocos. Para obter mais informações, consulte [Criar um instantâneo de um blob](/rest/api/storageservices/creating-a-snapshot-of-a-blob). Para cada instantâneo, você é cobrado apenas pelo armazenamento necessário para armazenar as diferenças no blob desde o último estado do instantâneo. Os instantâneos são dependentes da existência do blob original nos quais se baseiam, portanto, uma operação de cópia para outro blob ou até mesmo outra conta de armazenamento é aconselhável. Isso garante que os dados de backup estejam adequadamente protegidos contra exclusão acidental. É possível usar [AzCopy](/azure/storage/common/storage-use-azcopy) ou [Azure PowerShell](/azure/storage/common/storage-powershell-guide-full) para copiar os blobs para outra conta de armazenamento.

- **Arquivos**. Use [instantâneos de compartilhamento](/azure/storage/files/storage-snapshots-files) ou use AzCopy ou PowerShell a fim de copiar os arquivos para outra conta de armazenamento.

- **Tabelas**. Use AzCopy para exportar os dados da tabela para outra conta de armazenamento em outra região.

## <a name="database"></a>Banco de dados

### <a name="azure-sql-database"></a>Banco de Dados SQL do Azure 

O Banco de Dados SQL executa automaticamente uma combinação de backups de banco de dados semanais, backups de bancos de dados diferenciais por hora e backups de logs de transação a cada cinco a dez minutos para proteger sua empresa contra a perda de dados. Use a restauração pontual para restaurar um banco de dados para um momento anterior. Para obter mais informações, consulte:

- [Recuperar um banco de dados SQL do Azure usando backups de banco de dados automatizados](/azure/sql-database/sql-database-recovery-using-backups)

- [Visão geral da continuidade dos negócios com o Banco de Dados SQL do Azure](/azure/sql-database/sql-database-business-continuity)

### <a name="sql-server-on-vms"></a>SQL Server em VMs

Para o SQL Server em execução em VMs, há duas opções: backups tradicionais e envio de logs. Backups tradicionais permitem a restauração para um ponto específico no tempo, mas o processo de recuperação é lento. A restauração de backups tradicionais exige começar com um backup inicial completo e aplicar quaisquer backups feitos depois disso. A segunda opção é configurar uma sessão de envio de logs para atrasar a restauração de backups de log (por exemplo, por duas horas). Isso fornece uma janela para recuperação de erros feitos no primário.

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

O Azure Cosmos DB faz backups automaticamente em intervalos regulares. Os backups são armazenados separadamente em outro serviço de armazenamento, e esses backups são replicados globalmente para resiliência contra desastres regionais. Caso exclua seu banco de dados ou coleção acidentalmente, é possível criar um tíquete de suporte ou ligar para o suporte do Azure a fim de restaurar os dados usando o último backup automático. Para obter mais informações, consulte [Backup e restauração online automáticos com o Azure Cosmos DB](/azure/cosmos-db/online-backup-and-restore).

### <a name="azure-database-for-mysql-azure-database-for-postresql"></a>Banco de Dados do Azure para MySQL, Banco de Dados do Azure para PostreSQL

Ao usar o Banco de Dados do Azure para MySQL ou o Banco de Dados para PostreSQL, o serviço banco de dados faz um backup do serviço automaticamente a cada cinco minutos. Com esse recurso de backup automático você pode restaurar o servidor e todos os seus bancos de dados em um novo servidor em um ponto anterior no tempo. Para obter mais informações, consulte:

- [Como fazer backup e restaurar um servidor no Banco de Dados do Azure para MySQL usando o portal do Azure](/azure/mysql/howto-restore-server-portal)

- [Como fazer backup e restaurar um servidor no Banco de Dados do Azure para PostgreSQL usando o portal do Azure](/azure/postgresql/howto-restore-server-portal)

