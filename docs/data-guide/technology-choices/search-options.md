---
title: Escolhendo um armazenamento de dados de pesquisa
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 9576bbba9609a04ccc7851d55dd28853ffc6b701
ms.sourcegitcommit: f6be2825bf2d37dfe25cfab92b9e3973a6b51e16
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/08/2018
ms.locfileid: "48858173"
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
| É um serviço gerenciado | SIM | Não  | sim | SIM |  
| API REST | SIM | sim | sim | Não  |
| Programação | .NET | Java | Java | T-SQL | 
| Indexadores de documento para tipos de arquivo comuns (PDF, DOCX, TXT e assim por diante) | SIM | Não  | Sim | Não  |

### <a name="manageability-capabilities"></a>Funcionalidade de capacidade de gerenciamento

| | Azure Search | Elasticsearch | HDInsight com Solr | Banco de dados SQL | 
| --- | --- | --- | --- | --- |
| Esquema atualizável | Não  | sim | sim | SIM |
| Dá suporte à expansão  | SIM | sim | sim | Não  |

### <a name="analytic-workload-capabilities"></a>Funcionalidades de carga de trabalho analítica

| | Azure Search | Elasticsearch | HDInsight com Solr | Banco de dados SQL | 
| --- | --- | --- | --- | --- | 
| Dá suporte à análise além da pesquisa de texto completo | Não  | sim | sim | SIM |
| Parte de uma pilha de análise de log | Não  | Sim (ELK) |  Não  | Não  |
| Dá suporte à pesquisa semântica | Sim (encontrar apenas documentos semelhantes) | SIM | sim | SIM | 

### <a name="security-capabilities"></a>Funcionalidades de segurança

| | Azure Search | Elasticsearch | HDInsight com Solr | Banco de dados SQL | 
| --- | --- | --- | --- | --- | 
| Segurança em nível de linha | Partial (exige que a consulta de aplicativo filtre por ID de grupo) | Partial (exige que a consulta de aplicativo filtre por ID de grupo) | SIM | SIM | 
| Transparent Data Encryption | Não  | Não | Não  | SIM |  
| Restringir o acesso a endereços IP específicos | Não  | sim | sim | SIM |   
| Restringir o acesso para permitir apenas o acesso da rede virtual | Não  | sim | sim | SIM |  
| Autenticação do Active Directory (autenticação integrada) | Não  | Não | Não  | SIM | 

## <a name="see-also"></a>Consulte também

[Processando um texto de forma livre para pesquisa](../scenarios/search.md)
