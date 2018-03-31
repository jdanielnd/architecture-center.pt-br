---
title: Escolhendo um data warehouse
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 9cb3d4d0196b02da76d85c7f7f0e4a2a69d531e9
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-data-warehouse-in-azure"></a>Escolhendo um data warehouse no Azure

Um data warehouse é um repositório central, organizacional e relacional de dados integrados de uma ou mais fontes diferentes. Este tópico compara as opções de data warehouses no Azure.

> [!NOTE]
> Para obter mais informações sobre quando usar um data warehouse, consulte [Data warehouse e data marts](../scenarios/data-warehousing.md).

## <a name="what-are-your-options-when-choosing-a-data-warehouse"></a>Quais são as opções disponíveis ao escolher um data warehouse?

Há várias opções para a implementação de um data warehouse no Azure, dependendo de suas necessidades. As listas a seguir são divididas em duas categorias, [SMP](https://en.wikipedia.org/wiki/Symmetric_multiprocessing) (multiprocessamento simétrico) e [MPP](https://en.wikipedia.org/wiki/Massively_parallel) (processamento paralelo em massa). 

SMP:

- [Banco de Dados SQL do Azure](/azure/sql-database/)
- [SQL Server em uma máquina virtual](/sql/sql-server/sql-server-technical-documentation)

MPP:

- [Data Warehouse do Azure](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Apache Hive no HDInsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [Consulta Interativa (Hive LLAP) no HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

Como regra geral, warehouses baseados no SMP são mais adequados para conjuntos de dados pequenos a médios (até 4-100 TB), enquanto o MPP é frequentemente usado para Big Data. A delineação entre dados pequenos/médios e Big Data parcialmente está relacionada à definição e à infraestrutura de suporte de sua organização. (Consulte [Escolhendo um armazenamento de dados OLTP](oltp-data-stores.md#scalability-capabilities).) 

Além dos tamanhos de dados, o tipo de padrão de carga de trabalho provavelmente é um fator determinante maior. Por exemplo, consultas complexas podem ser muito lentas para uma solução SMP e exigem uma solução MPP. Sistemas baseados em MPP têm a probabilidade de impor uma penalidade de desempenho com tamanhos de dados pequenos, devido à maneira como os trabalhos são distribuídos e consolidados em nós. Se os tamanhos de dados já excedem 1 TB e devem aumentar continuamente, considere a possibilidade de escolher uma solução MPP. No entanto, se os tamanhos de dados são menores que isso, mas as cargas de trabalho excedem os recursos disponíveis da solução SMP, o MPP também pode ser a melhor opção.

Os dados acessados ou armazenados pelo data warehouse podem ser recebidos de várias fontes de dados, incluindo um data lake, como o [Azure Data Lake Store](/azure/data-lake-store/). Para obter uma sessão de vídeo que compara os diferentes pontos fortes de serviços MPP que podem usar o Azure Data Lake, consulte [Azure Data Lake e Data Warehouse do Azure: aplicando práticas modernas ao seu aplicativo](https://azure.microsoft.com/resources/videos/build-2016-azure-data-lake-and-azure-data-warehouse-applying-modern-practices-to-your-app/).

Sistemas SMP são caracterizados por uma única instância de um RDBMS que compartilha todos os recursos (CPU/Memória/Disco). Você pode escalar um sistema SMP verticalmente. Para o SQL Server em execução em uma VM, você pode dimensionar o tamanho da VM. Para o Banco de Dados SQL do Azure, você pode escalar verticalmente selecionando outra camada de serviço. 

Sistemas MPP podem ser escalados horizontalmente pela adição de mais nós de computação (que têm seus próprios subsistemas de CPU, memória e E/S). Há limitações físicas para a escala vertical de um servidor e, nesse momento, a escala horizontal é preferível, dependendo da carga de trabalho. No entanto, as soluções MPP exigem um conjunto de qualificações diferente, devido a variações em consulta, modelagem, particionamento de dados e outros fatores exclusivos ao processamento paralelo. 

Ao decidir qual solução SMP será usada, consulte [Uma análise mais aprofundada do Banco de Dados SQL do Azure e do SQL Server em VMs do Azure](/azure/sql-database/sql-database-paas-vs-sql-server-iaas#a-closer-look-at-azure-sql-database-and-sql-server-on-azure-vms). 

O SQL Data Warehouse do Azure também pode ser usado para conjuntos de dados pequenos e médios, em que a carga de trabalho faz uso intensivo de computação e memória. Leia mais sobre os padrões e cenários comuns do SQL Data Warehouse:

- [Padrões e antipadrões do SQL Data Warehouse](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/)
- [Padrões e estratégias de carregamento do SQL Data Warehouse](https://blogs.msdn.microsoft.com/sqlcat/2017/05/17/azure-sql-data-warehouse-loading-patterns-and-strategies/)
- [Migrando dados para o Azure SQL Data Warehouse](https://blogs.msdn.microsoft.com/sqlcat/2016/08/18/migrating-data-to-azure-sql-data-warehouse-in-practice/)
- [Padrões comuns de aplicativo ISV usando o SQL Data Warehouse do Azure](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/common-isv-application-patterns-using-azure-sql-data-warehouse/)

## <a name="key-selection-criteria"></a>Principais critérios de seleção

Para restringir as opções, comece respondendo a estas perguntas:

- Você deseja ter um serviço gerenciado em vez de gerenciar seus próprios servidores?

- Você está trabalhando com conjuntos de dados muito grandes ou consultas altamente complexas e de execução longa? Nesse caso, considere uma opção de MPP. 

- Para um conjunto grande de dados, a fonte de dados é estruturada ou não estruturada? Os dados não estruturados podem precisar ser processados em um ambiente de Big Data como o Spark no HDInsight, Azure Databricks, Hive LLAP no HDInsight ou Azure Data Lake Analytics. Tudo isso pode servir como ELT (Extrair, Carregar, Transformar) e mecanismos de ETL. Eles podem gerar os dados processados em dados estruturados, facilitando seu carregamento no SQL Data Warehouse ou em uma das outras opções. Para dados estruturados, o SQL Data Warehouse tem um nível de desempenho chamado Otimizado para Computação, para cargas de trabalho de computação intensiva que exigem desempenho muito alto.

- Você deseja separar os dados históricos dos dados operacionais atuais? Nesse caso, escolha uma das opções em que a [orquestração](pipeline-orchestration-data-movement.md) é necessária. Esses são warehouses autônomos otimizados para acesso de leitura intenso e são mais adequados como um armazenamento de dados históricos separado.

- Você precisa integrar dados de várias fontes, além de seu armazenamento de dados OLTP? Nesse caso, considere opções que integram várias fontes de dados com facilidade. 

- Você tem um requisito de multilocação? Nesse caso, o SQL Data Warehouse não é ideal para esse requisito. Para obter mais informações, consulte [Padrões e antipadrões do SQL Data Warehouse](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/).

- Você prefere um armazenamento de dados relacionais? Nesse caso, restrinja as opções àquelas com um armazenamento de dados relacionais, mas observe também que você pode usar uma ferramenta como o PolyBase para consultar armazenamentos de dados não relacionais, se necessário. No entanto, se você decidir usar o PolyBase, execute testes de desempenho nos conjuntos de dados não estruturados da carga de trabalho.

- Você tem requisitos de relatórios em tempo real? Caso precise de tempos de resposta de consulta rápida em grandes volumes de inserções singleton, restrinja as opções àquelas que podem dar suporte a relatórios em tempo real.

- Você precisa dar suporte a uma grande quantidade de usuários e conexões simultâneas? A capacidade de dar suporte a diversos usuários/conexões simultâneas depende de vários fatores. 

    - Para o Banco de Dados SQL do Azure, veja os [limites de recursos documentados](/azure/sql-database/sql-database-resource-limits) de acordo com a camada de serviço. 
    
    - O SQL Server permite um máximo de 32.767 conexões de usuário. Quando estiver em execução em uma VM, o desempenho dependerá do tamanho da VM e de outros fatores. 
    
    - O SQL Data Warehouse tem limites de consultas e conexões simultâneas. Para obter mais informações, consulte [Simultaneidade e gerenciamento de carga de trabalho no SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-develop-concurrency). Considere o uso de serviços complementares, como o [Azure Analysis Services](/azure/analysis-services/analysis-services-overview) para superar os limites no SQL Data Warehouse.

- Que tipo de carga de trabalho você tem? Em geral, as soluções de warehouse baseadas em MPP são mais adequadas para cargas de trabalho analíticas, orientadas por lote. Se as cargas de trabalho são transacionais por natureza, com muitas operações pequenas de leitura/gravação ou várias operações linha por linha, considere o uso de uma das opções de SMP. Uma exceção a essas diretrizes é durante o uso do processamento de fluxo em um cluster HDInsight, como o Spark Streaming e o armazenamento dos dados em uma tabela do Hive.

## <a name="capability-matrix"></a>Matriz de Funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades.

### <a name="general-capabilities"></a>Funcionalidades gerais

| | Banco de Dados SQL do Azure | SQL Server (VM) | SQL Data Warehouse | Apache Hive no HDInsight | Hive LLAP no HDInsight |
| --- | --- | --- | --- | --- | --- | -- |
| É um serviço gerenciado | sim | Não  | sim | Sim <sup>1</sup> | Sim <sup>1</sup> |
| Exige a orquestração de dados (armazena uma cópia dos dados/dados históricos) | Não  | Não  | sim | sim | sim |
| É integrado com facilidade a várias fontes de dados | Não  | Não  | sim | sim | sim |
| Dá suporte à pausa de computação | Não  | Não  | sim | Não <sup>2</sup> | Não <sup>2</sup> |
| Armazenamento de dados relacionais | sim | sim |  sim | Não  | Não  |
| Relatórios em tempo real | sim | sim | Não  | Não  | sim |
| Pontos de restauração de backup flexíveis | sim | sim | Não <sup>3</sup> | Sim <sup>4</sup> | Sim <sup>4</sup> |
| SMP/MPP | SMP | SMP | MPP | MPP | MPP |

[1] Configuração manual e dimensionamento.

[2] Os clusters HDInsight poderão ser excluídos quando não forem necessários e, em seguida, recriados. Anexe um armazenamento de dados externo ao cluster para que os dados sejam retidos quando você excluir o cluster. Use o Azure Data Factory para automatizar o ciclo de vida do cluster por meio da criação de um cluster HDInsight sob demanda para processar a carga de trabalho. Em seguida, exclua-o quando o processamento for concluído.

[3] Com o SQL Data Warehouse, você pode restaurar um banco de dados para qualquer ponto de restauração disponível nos últimos sete dias. Os instantâneos iniciam a cada 4-8 horas e permanecem disponíveis por sete dias. Quando um instantâneo possui mais de sete dias, ele expira e seu ponto de restauração fica indisponível.

[4] Considere o uso de um [metastore externo do Hive](/azure/hdinsight/hdinsight-hadoop-provision-linux-clusters#use-hiveoozie-metastore) que pode ser copiado em backup e restaurado, conforme necessário. As opções de backup e restauração padrão que se aplicam ao Armazenamento de Blobs ou ao Data Lake Store podem ser usadas para os dados ou podem ser usadas soluções terceiros de backup e restauração do HDInsight, como o [Imanis Data](https://azure.microsoft.com/blog/imanis-data-cloud-migration-backup-for-your-big-data-applications-on-azure-hdinsight/), para obter maior flexibilidade e facilidade de uso.

### <a name="scalability-capabilities"></a>Funcionalidades de escalabilidade

| | Banco de Dados SQL do Azure | SQL Server (VM) |  SQL Data Warehouse | Apache Hive no HDInsight | Hive LLAP no HDInsight |
| --- | --- | --- | --- | --- | --- | -- |
| Servidores regionais redundantes para alta disponibilidade  | sim | sim | sim | Não  | Não  |
| Dá suporte à expansão da consulta (consultas distribuídas)  | Não  | Não  | sim | sim | sim |
| Escalabilidade dinâmica (escalar verticalmente)  | sim | Não  | Sim <sup>1</sup> | Não  | Não  |
| Dá suporte ao cache em memória de dados | sim |  sim | Não  | sim | sim |

[1] O SQL Data Warehouse permite escalar ou reduzir verticalmente ajustando o número de DWUs (unidades de data warehouse). Consulte [Gerenciar o poder de computação no SQL Data Warehouse do Azure](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview).

### <a name="security-capabilities"></a>Funcionalidades de segurança

| | Banco de Dados SQL do Azure | SQL Server em uma máquina virtual | SQL Data Warehouse | Apache Hive no HDInsight | Hive LLAP no HDInsight |
| --- | --- | --- | --- | --- | --- | -- |
| Autenticação  | SQL/Azure AD (Azure Active Directory) | SQL/Azure AD/Active Directory | SQL/Azure AD | local/Azure AD <sup>1</sup> | local/Azure AD <sup>1</sup> |
| Autorização  | sim | sim | sim | sim | Sim <sup>1</sup> | Sim <sup>1</sup> |
| Auditoria  | sim | sim | sim | sim | Sim <sup>1</sup> | Sim <sup>1</sup> |
| Criptografia de dados em repouso | Sim <sup>2</sup> | Sim <sup>2</sup> | Sim <sup>2</sup> | Sim <sup>2</sup> | Sim <sup>1</sup> | Sim <sup>1</sup> |
| Segurança em nível de linha | sim | sim | sim | Não  | Sim <sup>1</sup> | Sim <sup>1</sup> |
| Dá suporte a firewalls | sim | sim | sim | sim | Sim <sup>3</sup> | Sim <sup>3</sup> |
| Mascaramento de dados dinâmicos | sim | sim | sim | Não  | Sim <sup>1</sup> | Sim <sup>1</sup> |

[1] Exige o uso de um [cluster HDInsight ingressado no domínio](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Exige o uso de TDE (Transparent Data Encryption) para criptografar e descriptografar os dados em repouso.

[3] Compatível quando [usado em uma Rede Virtual do Azure](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).

Leia mais sobre como proteger seu data warehouse:

* [Protegendo o Banco de Dados SQL](/azure/sql-database/sql-database-security-overview#connection-security)
* [Proteger um banco de dados no SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security)
* [Estender o Azure HDInsight usando uma Rede Virtual do Azure](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)
* [Segurança do Hadoop em nível empresarial com clusters HDInsight ingressados no domínio](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)

