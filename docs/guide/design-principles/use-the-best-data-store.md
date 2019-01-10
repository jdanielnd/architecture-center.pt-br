---
title: Usar o melhor armazenamento de dados para o trabalho
titleSuffix: Azure Application Architecture Guide
description: Escolha a tecnologia de armazenamento que é o mais adequado para seus dados e como ele será usado.
author: MikeWasson
ms.date: 08/30/2018
ms.custom: seojan19
ms.openlocfilehash: ab7cbe7005a00bcc2bfd7bad97f3eaf125f53e12
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113290"
---
# <a name="use-the-best-data-store-for-the-job"></a>Usar o melhor armazenamento de dados para o trabalho

## <a name="pick-the-storage-technology-that-is-the-best-fit-for-your-data-and-how-it-will-be-used"></a>Escolha a tecnologia de armazenamento mais adequada aos seus dados e como eles serão usados

Já se foram os dias em que você simplesmente colocaria todos os seus dados em um grande banco de dados SQL relacional. Bancos de dados relacionais são muito bons no que fazem &mdash; fornecer garantias ACID para transações sobre dados relacionais. Porém, isso tem alguns custos:

- As consultas podem exigir junções caras.
- Os dados devem ser normalizados e estar em conformidade com um esquema predefinido (esquema na gravação).
- A contenção de bloqueio pode afetar o desempenho.

Em qualquer solução de grande, é provável que uma tecnologia de armazenamento de dados única não atenda a todas as suas necessidades. Alternativas a bancos de dados relacionais incluem repositórios de chave/valor, bancos de dados de documentos, bancos de dados de mecanismos de pesquisa, bancos de dados de séries de tempo, bancos de dados de família de colunas e bancos de dados de gráficos. Cada um tem vantagens e desvantagens, e diferentes tipos de dados se encaixam mais naturalmente em um ou em outro.

Por exemplo, você pode armazenar um catálogo de produtos em um banco de dados de documentos, como o Cosmos DB, que permite que um esquema flexível. Nesse caso, cada descrição de produto é um documento independente. Para consultas em todo o catálogo, você pode indexar o catálogo e armazenar o índice no Azure Search. O inventário de produto pode ir para um banco de dados SQL, porque esses dados exigem garantias ACID.

Lembre-se de que os dados incluem mais do que apenas os dados de aplicativo persistentes. Eles também incluem logs de aplicativo, eventos, mensagens e caches.

## <a name="recommendations"></a>Recomendações

**Não use um banco de dados relacional para tudo**. Considere outros repositórios de dados quando apropriado. Consulte [Escolher o armazenamento de dados correto][data-store-overview].

**Adotar persistência poliglota**. Em qualquer solução de grande, é provável que uma tecnologia de armazenamento de dados única não atenda a todas as suas necessidades.

**Considere o tipo de dados**. Por exemplo, colocar os dados transacionais em SQL, colocar documentos JSON em um banco de dados de documento, colocar dados de telemetria em um banco de dados de série de tempo, colocar logs de aplicativo em Elasticsearch e colocar blobs no Armazenamento de Blobs do Azure.

**Prefira disponibilidade a consistência (forte)**. O teorema CAP implica que um sistema distribuído deve equilibrar disponibilidade e consistência. (Partições de rede, o outro segmento do teorema CAP, nunca podem ser completamente evitadas.) Geralmente, você pode atingir maior disponibilidade adotando um modelo de *consistência eventual*.

**Considere o conjunto de habilidades da equipe de desenvolvimento**. Há vantagens em usar a persistência poliglota, mas é possível exagerar. Adotar uma nova tecnologia de armazenamento de dados requer um novo conjunto de habilidades. A equipe de desenvolvimento deve entender como tirar o máximo proveito da tecnologia. Ela deve entender os padrões de uso apropriados, como otimizar consultas, ajustar o desempenho e assim por diante. Considere isso ao avaliar tecnologias de armazenamento.

**Use transações de compensação**. Um efeito colateral de persistência poliglota é que uma única transação pode gravar dados em vários repositórios. Se algo falhar, use transações de compensação para desfazer todas as etapas já concluídas.

**Examinar contextos limitados**. *Contexto limitado* é um termo de design conduzido por domínio. Um contexto associado é um limite explícito em torno de um modelo de domínio e define a quais partes do domínio o modelo se aplica. Idealmente, um contexto limitado é mapeado para um subdomínio do domínio da empresa. Os contextos limitados no seu sistema são um lugar natural considerar persistência poliglota. Por exemplo, "produtos" podem aparecer no subdomínio do Catálogo de Produtos e no subdomínio de Estoque de Produtos, mas é muito provável que esses dois subdomínios tenham requisitos diferentes para armazenar, atualizar e consultar produtos.

[data-store-overview]: ../technology-choices/data-store-overview.md
