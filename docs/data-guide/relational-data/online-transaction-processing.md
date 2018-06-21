---
title: OLTP (processamento de transações online)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 8650b919fc1a59240343015493a1fe41c8729a72
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30848692"
---
# <a name="online-transaction-processing-oltp"></a>OLTP (processamento de transações online)

O gerenciamento de dados transacionais usando sistemas de computador é conhecido como OLTP. Os sistemas OLTP registram as interações de negócios conforme elas ocorrem na operação diária da organização e dão suporte à consulta desses dados para criar inferências.

## <a name="transactional-data"></a>Dados transacionais

Dados transacionais são informações que acompanham as interações relacionadas às atividades de uma organização. Normalmente, essas interações são transações comerciais, como pagamentos recebidos de clientes, pagamentos feitos a fornecedores e movimentação de produtos pelo inventário, pedidos feitos ou serviços entregues. Eventos transacionais, que representam as transações em si, normalmente contêm uma dimensão temporal, alguns valores numéricos e referências a outros dados. 

As transações normalmente precisam ser *atômicas* e *consistentes*. Atomicidade significa que uma transação inteira sempre tem êxito ou falha como uma unidade de trabalho e nunca é deixada em um estado concluído incompleto. Se uma transação não pode ser concluída, o sistema de banco de dados precisa reverter todas as etapas que já foram realizadas como parte dessa transação. Em um RDBMS tradicional, essa reversão ocorre automaticamente se uma transação não pode ser concluída. Consistência significa que as transações sempre deixam os dados em um estado válido. (Essas são descrições muito informais de atomicidade e consistência. Há definições mais formais dessas propriedades, como [ACID](https://en.wikipedia.org/wiki/ACID).)

Bancos de dados transacionais podem dar suporte à coerência forte em transações que usam várias estratégias de bloqueio, como bloqueio pessimista, a fim de garantir que todos os dados sejam altamente consistentes dentro do contexto da empresa, para todos os usuários e processos. 

A arquitetura de implantação mais comum que usa dados transacionais é a camada de armazenamento de dados em uma arquitetura de três camadas. Uma arquitetura de três camadas normalmente consiste em uma camada de apresentação, uma camada de lógica de negócios e uma camada de armazenamento de dados. Uma arquitetura de implantação relacionada é a arquitetura [de N camadas](/azure/architecture/guide/architecture-styles/n-tier), que pode ter várias camadas intermediárias que manipulam a lógica de negócios.

## <a name="typical-traits-of-transactional-data"></a>Características comuns de dados transacionais

Os dados transacionais tendem a ter as seguintes características:

| Requisito | DESCRIÇÃO |
| --- | --- |
| Normalização | Altamente normalizado |
| Esquema | Esquema na gravação, altamente imposto|
| Consistência | Coerência forte, garantia de ACID |
| Integridade | Alta integridade |
| Usa transações | sim |
| Estratégia de bloqueio | Pessimista ou otimista|
| Atualizável | sim |
| Acrescentável | sim |
| Carga de trabalho | Gravações intensas, leituras moderadas |
| Indexação | Índices primários e secundários |
| Tamanho do dado | De pequeno a médio |
| Modelo | Relacional |
| Forma dos dados | Tabular |
| Flexibilidade de consulta | Altamente flexível |
| Escala | Pequeno (MBs) a grande (alguns TBs) | 

## <a name="when-to-use-this-solution"></a>Quando usar esta solução

Escolha o OLTP quando precisar processar e armazenar transações comerciais com eficiência, disponibilizando-as imediatamente para aplicativos cliente de uma maneira consistente. Use essa arquitetura quando um atraso tangível no processamento tiver um impacto negativo sobre as operações diárias da empresa.

Os sistemas OLTP foram projetados para processar e armazenar transações com eficiência, bem como consultar dados transacionais. A meta de processar e armazenar transações individuais com eficiência por um sistema OLTP é parcialmente realizada pela normalização de dados &mdash; ou seja, sua divisão em partes menores que são menos redundantes. Isso dá suporte à eficiência, porque permite que o sistema OLTP processe grandes quantidades de transações de forma independente e evita o processamento extra necessário para manter a integridade dos dados na presença de dados redundantes.

## <a name="challenges"></a>Desafios
A implementação e o uso de um sistema OLTP pode apresentar alguns desafios:

- Os sistemas OLTP nem sempre são bons para manipular agregações em grandes volumes de dados, embora haja exceções, como uma solução bem planejada baseada no SQL Server. A análise dos dados, que se baseia em cálculos de agregação em milhões de transações individuais, faz uso muito intensivo de recursos de um sistema OLTP. Eles podem ser lentos para serem executados e podem causar lentidão, bloqueando outras transações no banco de dados.
- Ao realizar a análise e os relatórios sobre dados altamente normalizados, as consultas tendem a ser complexas, pois a maioria das consultas precisa desnormalizar os dados usando junções. Além disso, as convenções de nomenclatura para objetos de banco de dados em sistemas OLTP tendem a ser concisas e sucintas. O aumento da normalização, juntamente com convenções de nomenclatura concisas, dificulta a consulta dos sistemas OLTP por parte dos usuários empresariais, sem a ajuda de um DBA ou desenvolvedor de dados.
- O armazenamento do histórico de transações por tempo indeterminado e o armazenamento de excesso de dados em uma tabela qualquer podem levar à redução do desempenho da consulta, dependendo da quantidade de transações armazenadas. A solução comum é manter uma janela de tempo relevante (como o ano fiscal atual) no sistema OLTP e descarregar dados históricos em outros sistemas, como um data mart ou [data warehouse](./data-warehousing.md).

## <a name="oltp-in-azure"></a>OLTP no Azure

Aplicativos, como sites hospedados em [Aplicativos Web do Serviço de Aplicativo](/azure/app-service/app-service-web-overview), APIs REST em execução no Serviço de Aplicativo ou aplicativos móveis ou da área de trabalho, se comunicam com o sistema OLTP, normalmente por meio de um intermediário da API REST.

Na prática, a maioria das cargas de trabalho não é puramente OLTP. Também há uma tendência de existir um componente analítico. Além disso, há uma demanda cada vez maior por relatórios em tempo real, como a execução de relatórios no sistema operacional. Isso também é chamado de HTAP (Processamento Transacional e Analítico Híbrido). Para obter mais informações, consulte [OLAP (processamento analítico online)](./online-analytical-processing.md).

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

|                              | Banco de Dados SQL do Azure | SQL Server em uma máquina virtual do Azure | Banco de Dados do Azure para MySQL | Banco de Dados do Azure para PostgreSQL |
|------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|      É um Serviço Gerenciado      |        sim         |                   Não                    |           sim            |              sim              |
|       É executado na plataforma       |        N/D         |         Windows, Linux, Docker         |           N/D            |              N/D              |
| Programação <sup>1</sup> |   T-SQL, .NET, R   |         T-SQL, .NET, R, Python         |  T-SQL, .NET, R, Python  |              SQL              |

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

|                                                                                                             | Banco de Dados SQL do Azure | SQL Server em uma máquina virtual do Azure | Banco de Dados do Azure para MySQL | Banco de Dados do Azure para PostgreSQL |
|-------------------------------------------------------------------------------------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|                                             Segurança em nível de linha                                              |        sim         |                  sim                   |           sim            |              sim              |
|                                                Mascaramento de dados                                                 |        sim         |                  sim                   |            Não            |              Não                |
|                                         Transparent Data Encryption                                         |        sim         |                  sim                   |           sim            |              sim              |
|                                  Restringir o acesso a endereços IP específicos                                   |        sim         |                  sim                   |           sim            |              sim              |
|                                  Restringir o acesso para permitir apenas o acesso da VNET                                  |        sim         |                  sim                   |            Não            |              Não                |
|                                    Autenticação do Azure Active Directory                                    |        sim         |                  sim                   |            Não            |              Não                |
|                                       Autenticação do Active Directory                                       |         Não          |                  sim                   |            Não            |              Não                |
|                                         Autenticação multifator                                         |        sim         |                  sim                   |            Não            |              Não                |
| Dá suporte ao [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) |        sim         |                  sim                   |           sim            |              Não                |
|                                                 IP Privado                                                  |         Não          |                  sim                   |           sim            |              Não                |

