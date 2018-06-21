---
title: Padrões de design na nuvem
description: Padrões de design de nuvem para o Microsoft Azure
keywords: Azure
ms.openlocfilehash: 4747c896fc6fc5866be782d76c5290d6b49ad451
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30848250"
---
# <a name="cloud-design-patterns"></a>Padrões de design na nuvem

[!INCLUDE [header](../../_includes/header.md)]

Esses padrões de design são úteis para a criação de aplicativos confiáveis, dimensionáveis e seguros na nuvem.

Cada padrão descreve o problema ao qual o padrão se destina, as considerações para a aplicação do padrão e um exemplo com base no Microsoft Azure. A maioria dos padrões inclui exemplos de código ou trechos de código que mostram como implementar o padrão no Azure. No entanto, a maioria dos padrões é relevante para qualquer sistema distribuído, se hospedados no Azure ou em outras plataformas de nuvem.

## <a name="problem-areas-in-the-cloud"></a>Áreas de problema na nuvem

<ul id="categories" class="panel">
{%-categoria nas categorias %}
    <li>
    {% incluem 'cartão de categoria padrão' %}
    </li>
{%- endfor %}
</ul>

## <a name="catalog-of-patterns"></a>Catálogo de padrões

| Padrão | Resumo |
|---------|---------|
|         |         |

{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}
