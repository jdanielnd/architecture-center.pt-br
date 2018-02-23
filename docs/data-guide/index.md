---
title: Guia de arquitetura de dados do Azure
description: 
author: zoinerTejada
ms:date: 02/12/2018
layout: LandingPage
ms.openlocfilehash: 848601f27faf56ea069852d8983e4d10fbad9d77
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/14/2018
---
# <a name="azure-data-architecture-guide"></a>Guia de arquitetura de dados do Azure

Este guia apresenta uma abordagem estruturada para a criação de soluções centradas em dados no Microsoft Azure. Ele se baseia em práticas comprovadas obtidas em engajamentos com clientes.

## <a name="introduction"></a>Introdução

A nuvem está mudando a maneira como os aplicativos são criados, incluindo como os dados são processados e armazenados. Em vez de um único banco de dados de uso geral que manipula todos os dados da solução, as soluções de _persistência poliglota_ usam vários armazenamentos de dados especializados, cada um otimizado para fornecer funcionalidades específicas. A perspectiva sobre os dados na solução muda como resultado disso. Não existem mais várias camadas de lógica de negócios que leem e gravam em uma única camada de dados. Em vez disso, as soluções são projetadas em torno de um *pipeline de dados* que descreve como os dados fluem por uma solução, o local em que são processados, o local em que são armazenados e como eles são consumidos pelo próximo componente do pipeline. 

## <a name="how-this-guide-is-structured"></a>Como este guia é estruturado

Este guia é estruturado em torno de um pivô básico: a diferença entre dados *relacionais* e dados *não relacionais*. 

![](./images/guide-steps.svg)

Dados relacionais geralmente são armazenados em um RDBMS tradicional ou um data warehouse. Ele tem um esquema predefinido ("esquema na gravação") com um conjunto de restrições para manter a integridade referencial. A maioria dos bancos de dados relacionais usa a linguagem SQL para consulta. Entre as soluções que usam bancos de dados relacionais estão o OLTP (processamento de transações online) e o OLAP (processamento analítico online).

Dados não relacionais são todos os dados que não usam o [modelo relacional](https://en.wikipedia.org/wiki/Relational_model) encontrado em sistemas RDBMS tradicionais. Isso pode incluir dados de chave-valor, dados JSON, dados de gráfico, dados de série temporal e outros tipos de dados. O termo *NoSQL* refere-se a bancos de dados que foram projetados para armazenar vários tipos de dados não relacionais. No entanto, o termo não é totalmente preciso, porque muitos armazenamentos de dados não relacionais dão suporte a consultas compatíveis com SQL. Os dados não relacionais e os bancos de dados NoSQL geralmente são mencionados em discussões sobre soluções de *Big Data*. Uma arquitetura de Big Data foi projetada para lidar com ingestão, processamento e análise de dados grandes ou complexos demais para sistemas de banco de dados tradicionais. 

Em cada uma dessas duas categorias principais, o Guia de Arquitetura de Dados contém as seguintes seções:

- **Conceitos.** Artigos de visão geral que apresentam os principais conceitos que você precisa entender ao trabalhar com esse tipo de dados.
- **Cenários.** Um conjunto representativo dos cenários de dados, incluindo uma discussão sobre os serviços relevantes do Azure e a arquitetura apropriada para o cenário.
- **Opções de tecnologia.** Comparações detalhadas de várias tecnologias de dados disponíveis no Azure, incluindo opções de software livre. Em cada categoria, descrevemos os principais critérios de seleção e uma matriz de funcionalidades, para ajudá-lo a escolher a tecnologia certa para seu cenário.

Este guia não se destina a ensinar a teoria de ciência de dados ou de banco de dados &mdash; você pode encontrar livros exclusivos sobre esses temas. Em vez disso, a meta é ajudar você a escolher a arquitetura de dados ou o pipeline de dados certo para seu cenário e, em seguida, escolher os serviços e as tecnologias do Azure que melhor atendam aos seus requisitos. Se você já tem uma arquitetura em mente, vá diretamente para as opções de tecnologia.

## <a name="traditional-rdbms"></a>RDBMS tradicional

### <a name="concepts"></a>Conceitos

- [Dados relacionais](./concepts/relational-data.md) 
- [Dados transacionais](./concepts/transactional-data.md) 
- [Modelagem semântica](./concepts/semantic-modeling.md) 

### <a name="scenarios"></a>Cenários

- [OLAP (processamento analítico online)](./scenarios/online-analytical-processing.md)
- [OLTP (processamento de transações online)](./scenarios/online-transaction-processing.md) 
- [Data warehouse e data marts](./scenarios/data-warehousing.md)
- [ETL](./scenarios/etl.md) 

## <a name="big-data-and-nosql"></a>Big Data e NoSQL

### <a name="concepts"></a>Conceitos

- [Armazenamentos de dados não relacionais](./concepts/non-relational-data.md)
- [Trabalhando com arquivos CSV e JSON](./concepts/csv-and-json.md)
- [Arquiteturas de Big Data](./concepts/big-data.md)
- [Análise avançada](./concepts/advanced-analytics.md) 
- [Aprendizado de máquina em escala](./concepts/machine-learning-at-scale.md)

### <a name="scenarios"></a>Cenários

- [Processamento em lotes](./scenarios/batch-processing.md)
- [Processamento em tempo real](./scenarios/real-time-processing.md)
- [Pesquisa de texto de forma livre](./scenarios/search.md)
- [Exploração interativa de dados](./scenarios/interactive-data-exploration.md)
- [Processamento de idioma natural](./scenarios/natural-language-processing.md)
- [Soluções de série temporal](./scenarios/time-series.md)

## <a name="cross-cutting-concerns"></a>Interesses paralelos

- [Transferência de dados](./scenarios/data-transfer.md) 
- [Estendendo soluções de dados locais para a nuvem](./scenarios/hybrid-on-premises-and-cloud.md) 
- [Protegendo soluções de dados](./scenarios/securing-data-solutions.md) 
