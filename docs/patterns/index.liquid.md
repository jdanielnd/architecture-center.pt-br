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
# <a name="cloud-design-patterns"></a><span data-ttu-id="90665-104">Padrões de design na nuvem</span><span class="sxs-lookup"><span data-stu-id="90665-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="90665-105">Esses padrões de design são úteis para a criação de aplicativos confiáveis, dimensionáveis e seguros na nuvem.</span><span class="sxs-lookup"><span data-stu-id="90665-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="90665-106">Cada padrão descreve o problema ao qual o padrão se destina, as considerações para a aplicação do padrão e um exemplo com base no Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="90665-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="90665-107">A maioria dos padrões inclui exemplos de código ou trechos de código que mostram como implementar o padrão no Azure.</span><span class="sxs-lookup"><span data-stu-id="90665-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="90665-108">No entanto, a maioria dos padrões é relevante para qualquer sistema distribuído, se hospedados no Azure ou em outras plataformas de nuvem.</span><span class="sxs-lookup"><span data-stu-id="90665-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="90665-109">Áreas de problema na nuvem</span><span class="sxs-lookup"><span data-stu-id="90665-109">Problem areas in the cloud</span></span>

<ul id="categories" class="panel">
<span data-ttu-id="90665-110">{%-categoria nas categorias %}</span><span class="sxs-lookup"><span data-stu-id="90665-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="90665-111">{% incluem 'cartão de categoria padrão' %}</span><span class="sxs-lookup"><span data-stu-id="90665-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="90665-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="90665-112">{%- endfor %}</span></span>
</ul>

## <a name="catalog-of-patterns"></a><span data-ttu-id="90665-113">Catálogo de padrões</span><span class="sxs-lookup"><span data-stu-id="90665-113">Catalog of patterns</span></span>

| <span data-ttu-id="90665-114">Padrão</span><span class="sxs-lookup"><span data-stu-id="90665-114">Pattern</span></span> | <span data-ttu-id="90665-115">Resumo</span><span class="sxs-lookup"><span data-stu-id="90665-115">Summary</span></span> |
|---------|---------|
|         |         |

<span data-ttu-id="90665-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="90665-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>
