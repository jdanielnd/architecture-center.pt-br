---
title: Escolhendo um armazenamento de dados de pesquisa
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 7fe5952c880921984beb30c71458fd1ef72ef239
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-search-data-store-in-azure"></a>Escolhendo um armazenamento de dados de pesquisa no Azure

Este artigo compara as opções de tecnologia de armazenamentos de dados de pesquisa no Azure. Um armazenamento de dados de pesquisa é usado para criar e armazenar índices especializados para executar pesquisas em texto de forma livre. O texto indexado pode residir em um armazenamento de dados separado, como um armazenamento de blobs. Um aplicativo envia uma consulta para o armazenamento de dados de pesquisa e o resultado é uma lista de documentos correspondentes. Para obter mais informações sobre esse cenário, consulte [Processando texto de forma livre para pesquisa](../scenarios/search.md). 

## <a name="what-are-your-options-when-choosing-a-search-data-store"></a>Quais são as opções disponíveis ao escolher um armazenamento de dados de pesquisa?
No Azure, todos os seguintes armazenamentos de dados atendem aos requisitos básicos de pesquisa em dados de texto de forma livre, fornecendo um índice de pesquisa:
- [Azure Search](/azure/search/search-what-is-azure-search)
- [Elasticsearch](https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch?tab=Overview)
- [HDInsight com Solr](/azure/hdinsight/hdinsight-hadoop-solr-install-linux)
- [Banco de Dados SQL do Azure com pesquisa de texto completo](/sql/relational-databases/search/full-text-search)


## <a name="key-selection-criteria"></a>Principais critérios de seleção

Para cenários de pesquisa, comece escolhendo o armazenamento de dados de pesquisa apropriado para suas necessidades, respondendo a essas perguntas:

- Você deseja ter um serviço gerenciado em vez de gerenciar seus próprios servidores?

- Você pode especificar o esquema de índice em tempo de design? Caso contrário, escolha uma opção que dá suporte a esquemas atualizáveis.

- Você precisa de um índice somente para a pesquisa de texto completo ou também precisa de agregação rápida de dados numéricos e outras análises? Caso precise de funcionalidade além da pesquisa de texto completo, considere as opções que dão suporte à análise adicional.

- Você precisa de um índice de pesquisa para a análise de log, com suporte para a coleta de log, agregação e visualizações em dados indexados? Nesse caso, considere o uso do Elasticsearch, que faz parte de uma pilha de análise de log.

- Você precisa indexar dados em formatos de documentos comuns, como PDF, Word, PowerPoint e Excel? Em caso afirmativo, escolha uma opção que fornece indexadores de documento.

- O banco de dados tem necessidades específicas de segurança? Em caso afirmativo, considere os recursos de segurança listados abaixo.

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades.

### <a name="general-capabilities"></a>Funcionalidades gerais
| | Azure Search | Elasticsearch | HDInsight com Solr | Banco de dados SQL | 
| --- | --- | --- | --- | --- | 
| É um serviço gerenciado | sim | Não  | sim | sim |  
| API REST | sim | sim | sim | Não  |
| Programação | .NET | Java | Java | T-SQL | 
| Indexadores de documento para tipos de arquivo comuns (PDF, DOCX, TXT e assim por diante) | sim | Não  | sim | Não  |

### <a name="manageability-capabilities"></a>Funcionalidade de capacidade de gerenciamento
| | Azure Search | Elasticsearch | HDInsight com Solr | Banco de dados SQL | 
| --- | --- | --- | --- | --- |
| Esquema atualizável | Não  | sim | sim | sim |
| Dá suporte à expansão  | sim | sim | sim | Não  |

### <a name="analytic-workload-capabilities"></a>Funcionalidades de carga de trabalho analítica
| | Azure Search | Elasticsearch | HDInsight com Solr | Banco de Dados SQL | 
| --- | --- | --- | --- | --- | 
| Dá suporte à análise além da pesquisa de texto completo | Não  | sim | sim | sim |
| Parte de uma pilha de análise de log | Não  | Sim (ELK) |  Não  | Não  |
| Dá suporte à pesquisa semântica | Sim (encontrar apenas documentos semelhantes) | sim | sim | sim | 

### <a name="security-capabilities"></a>Funcionalidades de segurança
| | Azure Search | Elasticsearch | HDInsight com Solr | Banco de Dados SQL | 
| --- | --- | --- | --- | --- | 
| Segurança em nível de linha | Partial (exige que a consulta de aplicativo filtre por ID de grupo) | Partial (exige que a consulta de aplicativo filtre por ID de grupo) | sim | sim | 
| Transparent Data Encryption | Não  | Não  | Não  | sim |  
| Restringir o acesso a endereços IP específicos | Não  | sim | sim | sim |   
| Restringir o acesso para permitir apenas o acesso da rede virtual | Não  | sim | sim | sim |  
| Autenticação do Active Directory (autenticação integrada) | Não  | Não  | Não  | sim | 

## <a name="see-also"></a>Consulte também

[Processando um texto de forma livre para pesquisa](../scenarios/search.md)