---
title: Padrões de design na nuvem
titleSuffix: Azure Architecture Center
description: Padrões de design de nuvem para o Microsoft Azure
keywords: Azure
author: dragon119
ms.date: 12/10/2018
ms.custom: seodec18
ms.openlocfilehash: 873d4cf02690a2cc3ffe4f35b044dedf70700fb5
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011014"
---
# <a name="cloud-design-patterns"></a>Padrões de design na nuvem

[!INCLUDE [header](../../_includes/header.md)]

Esses padrões de design são úteis para a criação de aplicativos confiáveis, dimensionáveis e seguros na nuvem.

Cada padrão descreve o problema ao qual o padrão se destina, as considerações para a aplicação do padrão e um exemplo com base no Microsoft Azure. A maioria dos padrões inclui exemplos de código ou snippets de código que mostram como implementar o padrão no Azure. No entanto, a maioria dos padrões é relevante para qualquer sistema distribuído, se hospedados no Azure ou em outras plataformas de nuvem.

## <a name="problem-areas-in-the-cloud"></a>Áreas de problema na nuvem

<!-- markdownlint-disable MD033 -->

<ul id="categories" class="panel">
{%-categoria nas categorias %}
    <li>
    {% incluem 'cartão de categoria padrão' %}
    </li>
{%- endfor %}
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="catalog-of-patterns"></a>Catálogo de padrões

| Padrão | Resumo |
|---------|---------|
|         |         |

{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}
