---
title: Escolhendo uma tecnologia de processamento de idioma natural
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: b1cb019164285d16b6e9d34eae220801785adab9
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902299"
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a>Escolhendo uma tecnologia de processamento de idioma natural no Azure

O processamento de texto de forma livre é feito em documentos que contêm parágrafos de texto, normalmente, para fins de pesquisa de apoio, mas também é usado para executar outras tarefas de NLP (processamento de idioma natural) como análise de sentimento, detecção de tópico, detecção de idioma, extração de frases-chave e categorização de documento. Este artigo se concentra nas opções de tecnologia que servem para dar suporte às tarefas de NLP.

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a>Quais são as opções disponíveis ao escolher um serviço NLP?

No Azure, os seguintes serviços fornecem funcionalidades de NLP (processamento de idioma natural):

- [Azure HDInsight com Spark e Spark MLlib](/azure/hdinsight/spark/apache-spark-overview)
- [Azure Databricks](/azure/azure-databricks/what-is-azure-databricks)
- [Serviços Cognitivos da Microsoft](/azure/cognitive-services/welcome)

## <a name="key-selection-criteria"></a>Principais critérios de seleção

Para restringir as opções, comece respondendo a estas perguntas:

- Você deseja usar modelos predefinidos? Em caso afirmativo, considere o uso das APIs oferecidas pelos Serviços Cognitivos da Microsoft.

- Você precisa treinar modelos personalizados em um corpo grande de dados de texto? Em caso afirmativo, considere o uso do Azure HDInsight com Spark MLlib e Spark NLP.

- Você precisa de funcionalidades de NLP de baixo nível como geração de tokens, lematização e TF/IDF (frequência de termo/frequência de documento inverso)? Em caso afirmativo, considere o uso do Azure HDInsight com Spark MLlib e Spark NLP.

- Você precisa de funcionalidades de NLP simples e de alto nível, como identificação de entidade e intenção, detecção de tópico, verificação ortográfica ou análise de sentimento? Em caso afirmativo, considere o uso das APIs oferecidas pelos Serviços Cognitivos da Microsoft.

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades.  

### <a name="general-capabilities"></a>Funcionalidades gerais

| | Azure HDInsight | Serviços Cognitivos da Microsoft |
| --- | --- | --- |
| Fornece modelos pré-treinados como serviço | Não  | SIM |
| API REST | SIM | SIM |
| Programação | Python, Scala, Java | C#, Java, Node.js, Python, PHP, Ruby |
| Dá suporte ao processamento de conjuntos de Big Data e documentos grandes | SIM | Não  |

### <a name="low-level-natural-language-processing-capabilities"></a>Funcionalidades de processamento de idioma natural de baixo nível

| | Azure HDInsight | Serviços Cognitivos da Microsoft |  
| --- | --- | --- | 
| Gerador de token | Sim (Spark NLP) | Sim (API de Análise Linguística) |
| Lematizador | Sim (Spark NLP) | Não  |
| Lematizador | Sim (Spark NLP) | Não  |
| Marcação de parte do discurso | Sim (Spark NLP) | Sim (API de Análise Linguística) |
| TF/IDF (frequência de termo/frequência de documento inverso) | Sim (Spark MLlib) | Não  |
| Similaridade de cadeia de caracteres &mdash; editar cálculo de distância | Sim (Spark MLlib) | Não  |
| Cálculo de N-gram | Sim (Spark MLlib) | Não  |
| Remoção de palavra irrelevante (stop word) | Sim (Spark MLlib) | Não  |

### <a name="high-level-natural-language-processing-capabilities"></a>Funcionalidades de processamento de idioma natural de alto nível

| | Azure HDInsight | Serviços Cognitivos da Microsoft |
| --- | --- | --- | 
| Identificação de entidade/intenção e extração | Não  | Sim [API de LUIS (Serviço Inteligente de Reconhecimento Vocal)] |    
| Detecção de tópico | Sim (Spark NLP) | Sim (API de Análise de Texto) |
| Verificação ortográfica | Sim (Spark NLP) | Sim (API de Verificação Ortográfica do Bing) |
| Análise de sentimento | Sim (Spark NLP) | Sim (API de Análise de Texto) |
| Detecção de idioma | Não  | Sim (API de Análise de Texto) |
| Dá suporte a vários idiomas além do inglês | Não  | Sim (varia de acordo com a API) |

## <a name="see-also"></a>Consulte também

[Processamento de idioma natural](../scenarios/natural-language-processing.md)
