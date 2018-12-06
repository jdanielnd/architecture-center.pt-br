---
title: Escolhendo um armazenamento de dados analíticos
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 166361c73a3a9c812e07445f6b039e843e5e32f8
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902322"
---
# <a name="choosing-an-analytical-data-store-in-azure"></a>Escolhendo um armazenamento de dados analíticos no Azure

Em uma arquitetura de [Big Data](../big-data/index.md), geralmente, há a necessidade de um armazenamento de dados analíticos que forneça dados processados em um formato estruturado que pode ser consultado com ferramentas analíticas. Os armazenamentos de dados analíticos que dão suporte à consulta de dados de caminho quente e frio são chamados em conjunto de camada de serviço ou armazenamento de serviço de dados.

A camada de serviço lida com os dados processados dos caminhos quente e frio. Na [arquitetura lambda](../big-data/index.md#lambda-architecture), a camada de serviço é subdividida em uma camada de _serviço de velocidade_, que armazena os dados que foram processados de forma incremental e uma camada de _serviço de lote_, que contém a saída processada em lotes. A camada de serviço exige suporte forte para leituras aleatórias com baixa latência. O armazenamento de dados para a camada de velocidade também deve dar suporte a gravações aleatórias, porque o carregamento em lotes dos dados nesse repositório introduzirá atrasos indesejados. Por outro lado, o armazenamento de dados da camada de lote não precisa dar suporte a gravações aleatórias, mas gravações em lotes.

Não há uma única opção de gerenciamento de dados que seja a melhor para todas as tarefas de armazenamento de dados. Soluções de gerenciamento de dados diferentes são otimizadas para tarefas diferentes. A maioria dos aplicativos de nuvem do mundo real e processos de Big Data tem uma variedade de requisitos de armazenamento de dados e geralmente usa uma combinação de soluções de armazenamento de dados.

## <a name="what-are-your-options-when-choosing-an-analytical-data-store"></a>Quais são as opções disponíveis ao escolher um armazenamento de dados analíticos?

Há várias opções de armazenamento de serviço de dados no Azure, dependendo de suas necessidades:

- [SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Banco de Dados SQL do Azure](/azure/sql-database/)
- [SQL Server em VM do Azure](/sql/sql-server/sql-server-technical-documentation)
- [HBase/Phoenix no HDInsight](/azure/hdinsight/hbase/apache-hbase-overview)
- [Hive LLAP no HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [Azure Cosmos DB](/azure/cosmos-db/)

Essas opções fornecem vários modelos de banco de dados que são otimizados para diferentes tipos de tarefas:

- Bancos de dados de [chave/valor](https://msdn.microsoft.com/library/dn313285.aspx#sec7) armazenam um único objeto serializado para cada valor de chave. Eles são bons para armazenar grandes volumes de dados, quando você deseja obter um item para determinado valor de chave e não precisa consultar com base em outras propriedades do item.
- Bancos de dados de [documentos](https://msdn.microsoft.com/library/dn313285.aspx#sec8) são bancos de dados de chave/valor nos quais os valores são *documentos*. Um "documento" neste contexto é uma coleção de campos nomeados e valores. Normalmente, o banco de dados armazena os dados em um formato como XML, YAML, JSON ou BSON, mas é possível usar um texto sem formatação. Os bancos de dados de documentos podem consultar em campos não chave e definir índices secundários para tornar a consulta mais eficiente. Isso torna um banco de dados de documentos mais adequado para aplicativos que precisam recuperar dados com base em critérios mais complexos do que o valor da chave do documento. Por exemplo, você pode consultar em campos como ID do produto (product ID), ID do cliente ou nome do cliente.
- Os bancos de dados de [família de colunas](https://msdn.microsoft.com/library/dn313285.aspx#sec9) são armazenamentos de dados de chave/valor que estruturam o armazenamento de dados em coleções de colunas relacionadas chamadas famílias de colunas. Por exemplo, um banco de dados de censo pode ter um grupo de colunas para o nome de uma pessoa (nome, sobrenome), um grupo para o endereço da pessoa e um grupo para as informações de perfil da pessoa (data de nascimento, gênero). O banco de dados pode armazenar cada família de colunas em uma partição separada, enquanto mantém todos os dados de uma pessoa relacionados à mesma chave. Um aplicativo pode ler uma única família de colunas sem ler todos os dados de uma entidade.
- Os bancos de dados de [gráficos](https://msdn.microsoft.com/library/dn313285.aspx#sec10) armazenam informações como uma coleção de objetos e relações. Um banco de dados de gráficos pode executar com eficiência consultas que atravessam a rede de objetos e as relações entre eles. Por exemplo, os objetos podem ser funcionários em um banco de dados de recursos humanos e talvez você deseje facilitar consultas como "encontrar todos os funcionários que trabalham direta ou indiretamente para Scott".

## <a name="key-selection-criteria"></a>Principais critérios de seleção

Para restringir as opções, comece respondendo a estas perguntas:

- Você precisa de um armazenamento de serviço que pode atuar como um caminho quente para os dados? Em caso afirmativo, restrinja as opções àquelas que são otimizadas para uma camada de serviço de velocidade.

- Você precisa de suporte de MPP (Processamento Paralelo Maciço), em que as consultas são distribuídas automaticamente entre vários processos ou nós? Em caso afirmativo, selecione uma opção que dá suporte à expansão da consulta.

- Você prefere usar um armazenamento de dados relacionais? Nesse caso, restrinja as opções àquelas com um modelo de banco de dados relacional. No entanto, observe que alguns armazenamentos não relacionais dão suporte à sintaxe SQL para consulta e ferramentas como o PolyBase podem ser usadas para consultar armazenamentos de dados não relacionais.

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades.

### <a name="general-capabilities"></a>Funcionalidades gerais

| | Banco de dados SQL | SQL Data Warehouse | HBase/Phoenix no HDInsight | Hive LLAP no HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| É um serviço gerenciado | SIM | SIM | Sim <sup>1</sup> | Sim <sup>1</sup> | SIM | SIM |
| Modelo de banco de dados primário | Relacional (formato de coluna ao usar índices columnstore) | Tabelas relacionais com armazenamento de coluna | Repositório de coluna grande | Hive/em memória | Modelos de tabela/semântica MOLAP | Repositório de documentos, gráfico, repositório de chave-valor, repositório de coluna grande |
| Suporte à linguagem SQL | SIM | SIM | Sim (usando o driver JDBC [Phoenix](https://phoenix.apache.org/)) | SIM | Não  | SIM |
| Otimizado para a camada de serviço de velocidade | Sim <sup>2</sup> | Não  | sim | sim | Não  | SIM |

[1] Com configuração manual e dimensionamento.

[2] Usando tabelas com otimização de memória e hash ou índices não clusterizados.
 
### <a name="scalability-capabilities"></a>Funcionalidades de escalabilidade

|                                                  | Banco de dados SQL | SQL Data Warehouse | HBase/Phoenix no HDInsight | Hive LLAP no HDInsight | Azure Analysis Services | Cosmos DB |
|--------------------------------------------------|--------------|--------------------|----------------------------|------------------------|-------------------------|-----------|
| Servidores regionais redundantes para alta disponibilidade |     SIM      |        sim         |            sim             |           Não           |           Não             |    SIM    |
|             Dá suporte à expansão da consulta             |      Não       |        sim         |            sim             |          sim           |           sim           |    SIM    |
|          Escalabilidade dinâmica (escalar verticalmente)          |     SIM      |        sim         |             Não             |           Não            |           sim           |    SIM    |
|        Dá suporte ao cache em memória de dados        |     SIM      |        sim         |             Não              |          sim           |           sim           |    Não      |

### <a name="security-capabilities"></a>Funcionalidades de segurança

| | Banco de dados SQL | SQL Data Warehouse | HBase/Phoenix no HDInsight | Hive LLAP no HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| Autenticação  | SQL/Azure AD (Azure Active Directory) | SQL/Azure AD | local/Azure AD <sup>1</sup> | local/Azure AD <sup>1</sup> | AD do Azure | usuários do banco de dados/Azure AD por meio do Controle de Acesso (IAM) |
| Criptografia de dados em repouso | Sim <sup>2</sup> | Sim <sup>2</sup> | Sim <sup>1</sup> | Sim <sup>1</sup> | SIM | SIM |
| Segurança em nível de linha | SIM | Não  | Sim <sup>1</sup> | Sim <sup>1</sup> | Sim (por meio da segurança no nível do objeto no modelo) | Não  |
| Dá suporte a firewalls | SIM | SIM | Sim <sup>3</sup> | Sim <sup>3</sup> | SIM | SIM |
| Mascaramento de dados dinâmicos | SIM | Não  | Sim <sup>1</sup> | Sim * | Não  | Não  |

[1] Exige o uso de um [cluster HDInsight ingressado no domínio](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Exige o uso de TDE (Transparent Data Encryption) para criptografar e descriptografar os dados em repouso.

[3] Quando usado em uma Rede Virtual do Azure. Consulte [Estender o Azure HDInsight usando uma Rede Virtual do Azure](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
