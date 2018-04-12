---
title: Escolhendo um armazenamento de dados OLTP
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 1c27d7d5f3b78f40822de6b77664dbf49b1367f6
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/31/2018
---
# <a name="choosing-an-oltp-data-store-in-azure"></a>Escolhendo um armazenamento de dados OLTP no Azure

O OLTP (processamento de transações online) é o gerenciamento de dados transacionais e o processamento de transações. Este tópico compara opções de soluções OLTP no Azure.

> [!NOTE]
> Para obter mais informações sobre quando usar um armazenamento de dados OLTP, consulte [Processamento de transações online](../scenarios/online-analytical-processing.md).

## <a name="what-are-your-options-when-choosing-an-oltp-data-store"></a>Quais são as opções disponíveis ao escolher um armazenamento de dados OLTP?

No Azure, todos os seguintes armazenamentos de dados atendem aos requisitos básicos de OLTP e do gerenciamento de dados de transação:

- [Banco de Dados SQL do Azure](/azure/sql-database/)
- [SQL Server em uma máquina virtual do Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Banco de Dados do Azure para MySQL](/azure/mysql/)
- [Banco de Dados do Azure para PostgreSQL](/azure/postgresql/)

## <a name="key-selection-criteria"></a>Principais critérios de seleção

Para restringir as opções, comece respondendo a estas perguntas:

- Você deseja ter um serviço gerenciado em vez de gerenciar seus próprios servidores?

- Sua solução tem dependências específicas para compatibilidade com o Microsoft SQL Server, MySQL ou PostgreSQL? Seu aplicativo pode limitar os armazenamentos de dados que você pode escolher com base nos fatores condutores que dão suporte à comunicação com o armazenamento de dados ou nas suposições que ele faz sobre qual banco de dados é usado.

- Seus requisitos de taxa de transferência de gravação são particularmente altos? Em caso afirmativo, escolha uma opção que fornece tabelas em memória. 

- Sua solução é de multilocatário? Nesse caso, considere opções que dão suporte a pools de capacidade, em que várias instâncias de banco de dados são extraídas de um pool elástico de recursos, em vez de recursos fixos por banco de dados. Isso pode ajudá-lo a distribuir melhor a capacidade em todas as instâncias de banco de dados e pode tornar a solução mais econômica.

- Seus dados precisam ser legíveis com baixa latência em várias regiões? Em caso afirmativo, escolha uma opção que dá suporte a réplicas secundárias legíveis.

- Seu banco de dados precisa estar altamente disponível em todas as regiões geográficas? Em caso afirmativo, escolha uma opção que dá suporte à replicação geográfica. Considere também as opções que dão suporte ao failover automático da réplica primária para uma réplica secundária.

- O banco de dados tem necessidades específicas de segurança? Em caso afirmativo, examine as opções que fornecem funcionalidades como segurança em nível de linha, máscara de dados e Transparent Data Encryption.

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades.

### <a name="general-capabilities"></a>Funcionalidades gerais 
| | Banco de Dados SQL do Azure | SQL Server em uma máquina virtual do Azure | Banco de Dados do Azure para MySQL | Banco de Dados do Azure para PostgreSQL |
| --- | --- | --- | --- | --- | --- |
| É um Serviço Gerenciado | sim | Não  | sim | sim |
| É executado na plataforma | N/D | Windows, Linux, Docker | N/D | N/D |
| Programação <sup>1</sup> | T-SQL, .NET, R | T-SQL, .NET, R, Python | T-SQL, .NET, R, Python | SQL | SQL |

[1] Não incluindo o suporte de driver do cliente, que permite que muitas linguagens de programação se conectem e usem o armazenamento de dados OLTP.

### <a name="scalability-capabilities"></a>Funcionalidades de escalabilidade
| | Banco de Dados SQL do Azure | SQL Server em uma máquina virtual do Azure| Banco de Dados do Azure para MySQL | Banco de Dados do Azure para PostgreSQL|
| --- | --- | --- | --- | --- | --- |
| Tamanho máximo da instância do banco de dados | [4 TB](/azure/sql-database/sql-database-resource-limits) | 256 TB | [1 TB](/azure/mysql/concepts-limits) | [1 TB](/azure/postgresql/concepts-limits) |
| Dá suporte a pools de capacidade  | sim | sim | Não | Não  |
| Dá suporte à expansão de clusters  | Não  | sim | Não | Não  |
| Escalabilidade dinâmica (escalar verticalmente)  | sim | Não  | sim | sim |

### <a name="analytic-workload-capabilities"></a>Funcionalidades de carga de trabalho analítica
| | Banco de Dados SQL do Azure | SQL Server em uma máquina virtual do Azure| Banco de Dados do Azure para MySQL | Banco de Dados do Azure para PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Tabelas temporais | sim | sim | Não | Não  |
| Tabelas em memória (com otimização de memória) | sim | sim | Não | Não  |
| Suporte de columnstore | sim | sim | Não | Não  |
| Processamento de consulta adaptável | sim | sim | Não | Não  |

### <a name="availability-capabilities"></a>Recursos de disponibilidade
| | Banco de Dados SQL do Azure | SQL Server em uma máquina virtual do Azure| Banco de Dados do Azure para MySQL | Banco de Dados do Azure para PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Secundários legíveis | sim | sim | Não | Não  | 
| Replicação geográfica | sim | sim | Não | Não  | 
| Failover automático para o secundário | sim | Não | Não  | Não |
| Restauração pontual | sim | sim | sim | sim |

### <a name="security-capabilities"></a>Funcionalidades de segurança
| | Banco de Dados SQL do Azure | SQL Server em uma máquina virtual do Azure| Banco de Dados do Azure para MySQL | Banco de Dados do Azure para PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Segurança em nível de linha | sim | sim | sim | sim |
| Mascaramento de dados | sim | sim | Não | Não  |
| Transparent Data Encryption | sim | sim | sim | sim |
| Restringir o acesso a endereços IP específicos | sim | sim | sim | sim |
| Restringir o acesso para permitir apenas o acesso da VNET | sim | sim | Não | Não  |
| Autenticação do Azure Active Directory | sim | sim | Não | Não  |
| Autenticação do Active Directory | Não  | sim | Não | Não  |
| Autenticação multifator | sim | sim | Não | Não  |
| Dá suporte ao [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) | sim | sim | sim | Não | Não  |
| IP Privado | Não  | sim | sim | Não | Não  |

