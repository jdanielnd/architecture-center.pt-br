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
# <a name="cloud-design-patterns"></a><span data-ttu-id="2fdbe-104">Padrões de design na nuvem</span><span class="sxs-lookup"><span data-stu-id="2fdbe-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="2fdbe-105">Esses padrões de design são úteis para a criação de aplicativos confiáveis, dimensionáveis e seguros na nuvem.</span><span class="sxs-lookup"><span data-stu-id="2fdbe-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="2fdbe-106">Cada padrão descreve o problema ao qual o padrão se destina, as considerações para a aplicação do padrão e um exemplo com base no Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="2fdbe-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="2fdbe-107">A maioria dos padrões inclui exemplos de código ou snippets de código que mostram como implementar o padrão no Azure.</span><span class="sxs-lookup"><span data-stu-id="2fdbe-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="2fdbe-108">No entanto, a maioria dos padrões é relevante para qualquer sistema distribuído, se hospedados no Azure ou em outras plataformas de nuvem.</span><span class="sxs-lookup"><span data-stu-id="2fdbe-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="2fdbe-109">Áreas de problema na nuvem</span><span class="sxs-lookup"><span data-stu-id="2fdbe-109">Problem areas in the cloud</span></span>

<!-- markdownlint-disable MD033 -->

<ul id="categories" class="panel">
<span data-ttu-id="2fdbe-110">{%-categoria nas categorias %}</span><span class="sxs-lookup"><span data-stu-id="2fdbe-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="2fdbe-111">{% incluem 'cartão de categoria padrão' %}</span><span class="sxs-lookup"><span data-stu-id="2fdbe-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="2fdbe-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="2fdbe-112">{%- endfor %}</span></span>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="catalog-of-patterns"></a><span data-ttu-id="2fdbe-113">Catálogo de padrões</span><span class="sxs-lookup"><span data-stu-id="2fdbe-113">Catalog of patterns</span></span>

| <span data-ttu-id="2fdbe-114">Padrão</span><span class="sxs-lookup"><span data-stu-id="2fdbe-114">Pattern</span></span> | <span data-ttu-id="2fdbe-115">Resumo</span><span class="sxs-lookup"><span data-stu-id="2fdbe-115">Summary</span></span> |
|---------|---------|
|         |         |

<span data-ttu-id="2fdbe-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="2fdbe-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>
