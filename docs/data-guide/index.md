---
title: Guia de arquitetura de dados do Azure
description: ''
author: zoinerTejada
ms:date: 02/12/2018
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 0975d056aa971c627ca91c9fc05c4e16e07b6fc0
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/07/2018
---
# <a name="azure-data-architecture-guide"></a>Guia de arquitetura de dados do Azure

Este guia apresenta uma abordagem estruturada para a criação de soluções centradas em dados no Microsoft Azure. Ele se baseia em práticas comprovadas obtidas em engajamentos com clientes.

## <a name="introduction"></a>Introdução

A nuvem está mudando a maneira como os aplicativos são criados, incluindo como os dados são processados e armazenados. Em vez de um único banco de dados de uso geral que manipula todos os dados da solução, as soluções de _persistência poliglota_ usam vários armazenamentos de dados especializados, cada um otimizado para fornecer funcionalidades específicas. A perspectiva sobre os dados na solução muda como resultado disso. Não existem mais várias camadas de lógica de negócios que leem e gravam em uma única camada de dados. Em vez disso, as soluções são projetadas em torno de um *pipeline de dados* que descreve como os dados fluem por uma solução, o local em que são processados, o local em que são armazenados e como eles são consumidos pelo próximo componente do pipeline. 

## <a name="how-this-guide-is-structured"></a>Como este guia é estruturado

Este guia é estruturado em torno de duas categorias gerais de solução de dados, *cargas de trabalho do RDBMS tradicional* e *soluções de Big Data*. 

**[Cargas de trabalho do RDBMS tradicional](./relational-data/index.md)**. Entre as cargas de trabalho estão o OLTP (processamento de transações online) e o OLAP (processamento analítico online). Os dados em sistemas OLTP geralmente são relacionais, com um esquema predefinido e um conjunto de restrições para manter a integridade referencial. Muitas vezes, dados de várias origens da organização podem ser consolidados em um data warehouse, usando um processo de ETL para mover e transformar os dados de origem.

![](./images/guide-rdbms.svg)

**[Soluções de Big Data](./big-data/index.md)**. Uma arquitetura de Big Data foi projetada para lidar com ingestão, processamento e análise de dados grandes ou complexos demais para sistemas de banco de dados tradicionais. Os dados podem ser processados em lote ou em tempo real. Soluções de Big Data geralmente envolvem uma grande quantidade de dados não relacionais, como dados de valor-chave, documentos JSON ou dados de série temporal. Muitas vezes, sistemas de RDBMS tradicional não são apropriados para armazenar esse tipo de dados. O termo *NoSQL* se refere a uma família de bancos de dados projetada para armazenar dados não relacionais. (O termo não está totalmente preciso, porque muitos armazenamentos de dados não relacionais oferecem suporte a consultas compatíveis com SQL.)

![](./images/guide-big-data.svg)

Essas duas categorias não são mutuamente exclusivas, e há sobreposição entre elas, mas acreditamos que seja uma maneira útil de enquadrar a discussão. Dentro de cada categoria, o guia discute os **cenários comuns**, incluindo serviços relevantes do Azure e a arquitetura apropriada para o cenário. Além disso, o guia compara **opções de tecnologia** para soluções de dados no Azure, incluindo opções de código aberto. Em cada categoria, descrevemos os principais critérios de seleção e uma matriz de funcionalidades, para ajudá-lo a escolher a tecnologia certa para seu cenário. 

Este guia não se destina a ensinar a teoria de ciência de dados ou de banco de dados &mdash; você pode encontrar livros exclusivos sobre esses temas. Em vez disso, a meta é ajudar você a escolher a arquitetura de dados ou o pipeline de dados certo para seu cenário e, em seguida, escolher os serviços e as tecnologias do Azure que melhor atendam aos seus requisitos. Se você já tem uma arquitetura em mente, vá diretamente para as opções de tecnologia.
