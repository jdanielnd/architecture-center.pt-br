---
title: Estendendo soluções de dados locais para a nuvem
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 66fc225dc123202ba587d82f15ea0883e1bbf3b5
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/14/2018
ms.locfileid: "29289682"
---
# <a name="extending-on-premises-data-solutions-to-the-cloud"></a>Estendendo soluções de dados locais para a nuvem

Quando as organizações movem cargas de trabalho e dados para a nuvem, seus datacenters locais geralmente continuam desempenhando um papel importante. O termo *nuvem híbrida* refere-se a uma combinação de nuvem pública e data centers locais, para criar um ambiente de TI integrado que abrange ambos. Algumas organizações usam a nuvem híbrida como um caminho para migrar seu datacenter inteiro para a nuvem ao longo do tempo. Outras organizações usam serviços de nuvem para estender sua infraestrutura local existente. 

Este artigo descreve algumas considerações e melhores práticas para o gerenciamento de dados em uma solução de nuvem híbrida.

## <a name="when-to-use-a-hybrid-solution"></a>Quando usar uma solução híbrida

Considere o uso de uma solução híbrida nos seguintes cenários:

* Como uma estratégia de transição durante uma migração de longo prazo para uma solução nativa totalmente na nuvem.
* Quando regulamentações ou políticas não permitem mover dados ou cargas de trabalho específicas para a nuvem.
* Para recuperação de desastre e tolerância a falhas, por meio da replicação de dados e serviços entre ambientes locais e na nuvem.
* Para reduzir a latência entre o data center local e locais remotos, armazenando parte da arquitetura no Azure.

## <a name="challenges"></a>Desafios

* Criar um ambiente consistente em termos de segurança, gerenciamento e desenvolvimento e evitar a duplicação de trabalho.

* Criar uma conexão de dados confiável, segura e de baixa latência entre ambientes locais e na nuvem.

* Replicar os dados e modificar aplicativos e ferramentas para que eles usem os armazenamentos de dados certos em cada ambiente.

* Proteger e criptografar os dados hospedados na nuvem, mas que são acessados localmente ou vice-versa.

## <a name="on-premises-data-stores"></a>Armazenamentos de dados locais

Os armazenamentos de dados locais incluem bancos de dados e arquivos. Pode haver várias razões para mantê-los localmente. Pode haver regulamentações ou políticas que não permitem mover dados ou cargas de trabalho específicas para a nuvem. Questões de segurança, privacidade ou soberania de dados podem favorecer o posicionamento local. Durante uma migração, é recomendável manter alguns dados localmente em um aplicativo que ainda não tenha sido migrado.

As considerações sobre a colocação de dados do aplicativo em uma nuvem pública incluem:

* **Custo**. O custo de armazenamento no Azure pode ser consideravelmente menor do que o custo de manutenção do armazenamento com características semelhantes em um datacenter local. É claro que muitas empresas já têm investimentos em redes SANs de alto nível e, portanto, essas vantagens de custo podem não se materializar por completo até que o hardware existente seja desativado.
* **Escala elástica**. O planejamento e o gerenciamento do aumento da capacidade de dados em um ambiente local podem ser um desafio, especialmente quando o aumento dos dados é difícil de ser previsto. Esses aplicativos podem aproveitar o armazenamento praticamente ilimitado e de capacidade sob demanda disponível na nuvem. Essa consideração é menos relevante para os aplicativos que consistem em conjuntos de dados de dimensionamento relativamente estático.
* **Recuperação de desastre**. Os dados armazenados no Azure podem ser replicados automaticamente em uma região do Azure e em várias regiões geográficas. Em ambientes híbridos, essas mesmas tecnologias podem ser usadas para replicar entre armazenamentos de dados locais e baseados em nuvem.

## <a name="extending-data-stores-to-the-cloud"></a>Estendendo armazenamentos de dados para a nuvem

Há várias opções para estender os armazenamentos de dados locais para a nuvem. Uma opção é ter réplicas locais e na nuvem. Isso pode ajudar a atingir um alto nível de tolerância a falhas, mas pode exigir alterações em aplicativos para se conectar ao armazenamento de dados apropriado no caso de failover.

Outra opção é mover uma parte dos dados para o armazenamento em nuvem, mantendo os dados mais atuais ou acessados com mais frequência localmente. Esse método pode fornecer uma opção mais econômica de armazenamento de longo prazo, além de melhorar os tempos de resposta de acesso a dados reduzindo o conjunto de dados operacionais.

Uma terceira opção é manter todos os dados localmente, mas usar a computação em nuvem para hospedar aplicativos. Para fazer isso, você hospeda seu aplicativo na nuvem e conecta-o ao armazenamento de dados local em uma conexão segura. 

## <a name="azure-stack"></a>Azure Stack

Para uma solução de nuvem híbrida completa, considere o uso do [Microsoft Azure Stack](/azure/azure-stack/). O Azure Stack é uma plataforma de nuvem híbrida que possibilita fornecer serviços do Azure por meio do datacenter. Isso ajuda a manter a consistência entre o local e o Azure, usando ferramentas idênticas e não exigindo nenhuma alteração de código. 

Estes são alguns casos de uso para do Azure e do Azure Stack:

* **Soluções desconectadas e de borda**. Atenda aos requisitos de latência e conectividade processando dados localmente no Azure Stack e então agregando-os ao Azure para análise adicional, com uma lógica comum do aplicativo entre ambos. 
* **Aplicativos de nuvem que atendem a regulamentos variados**. Desenvolva e implante aplicativos no Azure, com a flexibilidade de implantar os mesmos aplicativos locais no Azure Stack para atender a requisitos regulatórios ou de política.
* **Modelo de aplicativo de nuvem local**. Use o Azure para atualizar e estender aplicativos existentes ou criar novos. Use um processo de DevOps consistente no Azure na nuvem e no Azure Stack local.

## <a name="sql-server-data-stores"></a>Armazenamentos de dados do SQL Server

Se estiver executando o SQL Server localmente, use o serviço Armazenamento de Blobs do Microsoft Azure para backup e restauração. Para obter mais informações, consulte [Backup e restauração do SQL Server com o serviço de Armazenamento de Blobs do Microsoft Azure](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service). Essa funcionalidade oferece armazenamento ilimitado fora do site e a capacidade de compartilhar os mesmos backups entre o SQL Server em execução local e o SQL Server em execução em uma máquina virtual no Azure. 

O [Banco de Dados SQL do Azure](/azure/sql-database/) é um banco de dados como serviço relacional gerenciado. Como o Banco de Dados SQL do Azure usa o Mecanismo do Microsoft SQL Server, os aplicativos podem acessar dados da mesma maneira com ambas as tecnologias. O Banco de Dados SQL do Azure também pode ser combinado com o SQL Server de maneiras úteis. Por exemplo, o recurso [SQL Server Stretch Database](/sql/sql-server/stretch-database/stretch-database) permite que um aplicativo acesse o que parece ser uma única tabela em um banco de dados SQL Server, enquanto algumas ou todas as linhas da tabela podem ser armazenadas no Banco de Dados SQL do Azure. Essa tecnologia move automaticamente para a nuvem os dados que não são acessados por um período definido. Os aplicativos que leem esses dados não estão cientes de que nenhum dado foi movido para a nuvem.

A manutenção de armazenamentos de dados local e na nuvem pode ser um desafio quando você deseja manter os dados sincronizados. Resolva esse problema com a [Sincronização de Dados SQL](/azure/sql-database/sql-database-sync-data), um serviço baseado no Banco de Dados SQL do Azure que permite sincronizar os dados selecionados bidirecionalmente em vários bancos de dados SQL do Azure e instâncias do SQL Server. Embora a Sincronização de Dados facilite manter os dados atualizados entre esses vários armazenamentos de dados, ela não deve ser usada para recuperação de desastre ou para migrar do SQL Server local para o Banco de Dados SQL do Azure.

Para a recuperação de desastre e continuidade dos negócios, use [Grupos de Disponibilidade AlwaysOn](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) para replicar dados entre duas ou mais instâncias do SQL Server, algumas das quais podem ser executadas em máquinas virtuais do Azure em outra região geográfica.

## <a name="network-shares-and-file-based-data-stores"></a>Compartilhamentos de rede e armazenamentos de dados baseados em arquivo

Em uma arquitetura de nuvem híbrida, é comum que uma organização mantenha os arquivos mais recentes locais, enquanto arquiva os arquivos mais antigos na nuvem. Isso, às vezes, é chamado de camada de arquivo, em que há acesso contínuo aos dois conjuntos de arquivos, locais e hospedados na nuvem. Essa abordagem ajuda a minimizar o uso de largura de banda de rede e os tempos de acesso dos arquivos mais recentes, que devem ser acessados com mais frequência. Ao mesmo tempo, você obtém os benefícios do armazenamento baseado em nuvem para os dados arquivados. 

As organizações também podem desejar mover seus compartilhamentos de rede completamente para a nuvem. Isso será desejável, por exemplo, se os aplicativos que os acessam também estiverem localizados na nuvem. Esse procedimento pode ser feito com ferramentas de [orquestração de dados](../technology-choices/pipeline-orchestration-data-movement.md).


O [Azure StorSimple](/azure/storsimple/) oferece a solução de armazenamento integrado mais completa para gerenciar tarefas de armazenamento entre os dispositivos locais e o armazenamento em nuvem do Azure. O StorSimple é uma solução de rede de área de armazenamento (SAN) eficiente, econômica e facilmente gerenciável que elimina vários problemas e despesas associados à proteção de dados e ao armazenamento empresarial. Ele usa o dispositivo proprietário da série StorSimple 8000, é integrado aos serviços de nuvem e fornece um conjunto de ferramentas de gerenciamento integradas.

Outra maneira de usar compartilhamentos de rede locais junto com o armazenamento de arquivos baseado em nuvem é com os [Arquivos do Azure](/azure/storage/files/storage-files-introduction). Os Arquivos do Azure oferecem compartilhamentos de arquivos totalmente gerenciados que podem ser acessados com o [protocolo SMB](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx?f=255&MSPPError=-2147217396) padrão (às vezes conhecido como CIFS). Monte Arquivos do Azure como um compartilhamento de arquivos no computador local ou use-os com os aplicativos existentes que acessam arquivos de compartilhamento de rede ou locais.

Para sincronizar compartilhamentos de arquivos nos Arquivos do Azure com Windows Servers locais, use a [Sincronização de arquivos do Azure](/azure/storage/files/storage-sync-files-planning). Um benefício importante da Sincronização de arquivos do Azure é a capacidade de dispor os arquivos de camadas entre o servidor de arquivos local e os Arquivos do Azure. Isso permite que você mantenha apenas os arquivos mais recentes e acessados mais recentemente no local. 

Para obter mais informações, consulte [Decidindo quando usar o armazenamento de Blobs do Azure, os Arquivos do Azure ou os Discos do Azure](/azure/storage/common/storage-decide-blobs-files-disks).

## <a name="hybrid-networking"></a>Rede híbrida

Este artigo se concentra nas soluções de dados híbridas, mas outra consideração é como estender a rede local para o Azure. Para obter mais informações sobre esse aspecto das soluções híbridas, consulte os seguintes tópicos:

- [Escolher uma solução para conectar uma rede local ao Azure](../../reference-architectures/hybrid-networking/considerations.md)
- [Arquiteturas de referência de rede híbrida](../../reference-architectures/hybrid-networking/index.md)

