---
title: "Padrões de design na nuvem"
description: "Padrões de design de nuvem para o Microsoft Azure"
keywords: As tabelas
ms.openlocfilehash: 264b8296a428f9c1b87314b782efcabc89cf010f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
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
| ------- | ------- |
{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}