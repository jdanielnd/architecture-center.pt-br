---
title: "Recuperação de dados corrompidos ou exclusão acidental"
description: "Artigo com noções básicas sobre como recuperar dados corrompidos ou de exclusão acidental e como criar aplicativos resilientes, altamente disponíveis, com tolerância a falhas, bem como planejamento de recuperação de desastres"
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: b75c774f85c42f64472167897f08a7302ab50a3f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="azure-resiliency-technical-guidance-recovery-from-data-corruption-or-accidental-deletion"></a>Orientações técnicas de resiliência do Azure: recuperação de dados corrompidos ou exclusão acidental
Parte de um plano robusto da continuidade de negócios é ter um plano, caso os dados sejam corrompidos ou excluídos acidentalmente. Veja a seguir informações sobre recuperação depois que dados forem corrompidos ou excluídos acidentalmente devido a erros do aplicativo ou do operador.

## <a name="virtual-machines"></a>Máquinas Virtuais
Para proteger as Máquinas Virtuais do Azure (às vezes chamadas de VM de infraestrutura como serviço) de erros de aplicativo ou exclusão acidental, use o [Backup do Azure](https://azure.microsoft.com/services/backup/). O Backup do Azure permite a criação de backups que são consistentes em vários discos de VM. Além disso, o Cofre de Backup pode ser replicado entre regiões para fornecer recuperação de perda de região.

## <a name="storage"></a>Armazenamento
Observe que, enquanto o armazenamento do Azure fornece resiliência de dados por meio de réplicas automáticas, isso não impede que o código do aplicativo (ou desenvolvedores/usuários) corrompa dados por meio de exclusão acidental ou involuntária, atualização e assim por diante. Manter a fidelidade dos dados no caso de erro do usuário ou aplicativo requer técnicas mais avançadas, como copiar os dados para um local de armazenamento secundário com um log de auditoria. Os desenvolvedores podem aproveitar a [capacidade de instantâneo](https://msdn.microsoft.com/library/azure/ee691971.aspx)do blob, que pode criar instantâneos pontuais somente leitura de conteúdos de blob. Isso pode ser usado como a base de uma solução de fidelidade de dados para blobs de armazenamento do Azure.

### <a name="blob-and-table-storage-backup"></a>Backup de armazenamento de tabela e de blob
Embora blobs e tabelas sejam altamente duráveis, sempre representam o estado atual dos dados. A recuperação de modificação indesejada ou exclusão de dados pode exigir a restauração dos dados para um estado anterior. Isso pode ser obtido aproveitando as capacidades fornecidas pelo Azure para armazenar e manter cópias pontuais.

Para Blobs do Azure, você pode executar backups pontuais usando o [recurso de instantâneo de blob](https://msdn.microsoft.com/library/ee691971.aspx). Para cada instantâneo, você é cobrado apenas pelo armazenamento necessário para armazenar as diferenças no blob desde o último estado do instantâneo. Os instantâneos são dependentes da existência do blob original nos quais se baseiam, portanto, uma operação de cópia para outro blob ou até mesmo outra conta de armazenamento é aconselhável. Isso garante que os dados de backup estejam adequadamente protegidos contra exclusão acidental. Para tabelas do Azure, você pode fazer cópias pontuais em uma tabela diferente ou nos Blobs do Azure. Diretrizes e exemplos mais detalhados de como efetuar backups de aplicativo de tabelas e blobs podem ser encontrados aqui:

* [Protegendo suas tabelas contra erros de aplicativo](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/05/03/protecting-your-tables-against-application-errors/)
* [Protegendo seus blobs contra erros de aplicativo](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/04/29/protecting-your-blobs-against-application-errors/)

## <a name="database"></a>Banco de dados
Há várias opções de [continuidade de negócios](/azure/sql-database/sql-database-business-continuity/) (backup, restauração) disponíveis para o banco de dados SQL do Azure. Os bancos de dados podem ser copiados usando a funcionalidade de [Cópia de Banco de Dados](/azure/sql-database/sql-database-copy/) ou [exportando](/azure/sql-database/sql-database-export/) e [importando](https://msdn.microsoft.com/library/hh710052.aspx) um arquivo bacpac do SQL Server. A Cópia de Banco de Dados fornece resultados transacionalmente consistentes, enquanto um bacpac (por meio do serviço de importação/exportação), não. Essas duas opções são executadas como serviços baseados em fila no datacenter e atualmente não fornecem um SLA de tempo de conclusão.

> [!NOTE]
> As opções de importação/exportação e cópia de banco de dados impõem um grau significativo de carga no banco de dados de origem. Elas podem disparar a contenção de recursos ou eventos de limitação.
> 
> 

### <a name="sql-database-backup"></a>Backup do banco de dados SQL
Backups pontuais para o Banco de Dados SQL do Microsoft Azure são obtidos ao [Copiar seu banco de dados SQL do Azure](/azure/sql-database/sql-database-copy/). Você pode usar esse comando para criar uma cópia transacionalmente consistente de um banco de dados no mesmo servidor de banco de dados lógico ou em um servidor diferente. Em ambos os casos, a cópia do banco de dados é totalmente funcional e completamente independente do banco de dados de origem. Cada cópia que você cria representa uma opção de recuperação pontual. Você pode recuperar o estado do banco de dados completamente renomeando o novo banco de dados com o nome do banco de dados de origem. Como alternativa, você pode recuperar um subconjunto específico de dados do novo banco de dados usando consultas Transact-SQL. Para obter detalhes adicionais sobre Banco de Dados SQL, consulte [Visão geral da continuidade de negócios com o Banco de Dados SQL](/azure/sql-database/sql-database-business-continuity/).

### <a name="sql-server-on-virtual-machines-backup"></a>SQL Server em backup de máquinas virtuais
Para o SQL Server usado com as máquinas virtuais de infraestrutura como serviço do Azure (geralmente chamadas de IaaS ou VMs de IaaS), há duas opções: backups tradicionais e envio de logs. O uso de backups tradicionais permite restaurar para um ponto específico no tempo, mas o processo de recuperação é lento. A restauração de backups tradicionais exige começar com um backup inicial completo e aplicar quaisquer backups feitos depois disso. A segunda opção é configurar uma sessão de envio de logs para atrasar a restauração de backups de log (por exemplo, por duas horas). Isso fornece uma janela para recuperação de erros feitos no primário.

## <a name="other-azure-platform-services"></a>Outros serviços da plataforma Azure
Alguns serviços da plataforma Azure armazenam informações em uma conta de armazenamento controlada pelo usuário ou banco de dados SQL do Azure. Se o recurso de conta ou armazenamento for excluído ou corrompido, isso poderá causar erros graves no serviço. Nesses casos, é importante manter os backups que permitiriam recriar esses recursos caso eles sejam excluídos ou corrompidos.

Para sites do Azure e Serviços Móveis do Azure, você deve fazer o backup e a manutenção dos bancos de dados associados. Para o serviço de mídia do Azure e máquinas virtuais, você deve manter a conta de armazenamento do Azure associada e todos os recursos dessa conta. Por exemplo, para Máquinas Virtuais, você deve fazer backup e gerenciar os discos de VM no armazenamento de blobs do Azure.

## <a name="checklists-for-data-corruption-or-accidental-deletion"></a>Listas de verificação de dados corrompidos ou exclusão acidental
## <a name="virtual-machines-checklist"></a>Lista de verificação de Máquinas Virtuais
1. Confira a seção Máquinas Virtuais deste documento.
2. Faça backup e manutenção dos discos de VM com o Backup do Azure (ou seu próprio sistema de backup usando o armazenamento de blobs do Azure e instantâneos de VHD).

## <a name="storage-checklist"></a>Lista de verificação de armazenamento
1. Examine a seção Armazenamento deste documento.
2. Faça backups regulares dos recursos de armazenamento críticos.
3. Considere o uso do recurso de instantâneo para blobs.

## <a name="database-checklist"></a>Lista de verificação de banco de dados
1. Examine a seção Banco de dados deste documento.
2. Crie backups pontuais usando o comando Cópia de Banco de Dados.

## <a name="sql-server-on-virtual-machines-backup-checklist"></a>Lista de verificação do SQL Server em backup de máquinas virtuais
1. Leia a seção SQL Server em backup de máquinas virtuais deste documento.
2. Use técnicas tradicionais de backup e restauração.
3. Crie uma sessão de envio de logs em atraso.

## <a name="web-apps-checklist"></a>Lista de verificação de aplicativos Web
1. Faça backup e manutenção do banco de dados associado, se houver.

## <a name="media-services-checklist"></a>Lista de verificação dos Serviços de Mídia
1. Faça backup e manutenção dos recursos de armazenamento associados.

## <a name="more-information"></a>Mais informações
Para saber mais sobre recursos de backup e restauração no Azure, confira [Cenários de armazenamento, backup e recuperação](https://azure.microsoft.com/documentation/scenarios/storage-backup-recovery/).


