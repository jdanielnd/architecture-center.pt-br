---
title: Árvore de decisão para serviços de computação do Azure
description: Um fluxograma para selecionar um serviço de computação
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: e601dcb653ed1809ea3f9bbda8db8b40efb460a5
ms.sourcegitcommit: 3846a0ab2b2b2552202a3c9c21af0097a145ffc6
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/29/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="410a5-103">Árvore de decisão para serviços de computação do Azure</span><span class="sxs-lookup"><span data-stu-id="410a5-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="410a5-104">O Azure oferece várias maneiras de hospedar o código do seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="410a5-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="410a5-105">O termo *computação* refere-se ao modelo de hospedagem para os recursos de computação em que seu aplicativo é executado.</span><span class="sxs-lookup"><span data-stu-id="410a5-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="410a5-106">O fluxograma a seguir o ajudará a escolher um serviço de computação para o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="410a5-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="410a5-107">O fluxograma orienta você por um conjunto de critérios de decisão essenciais que geram uma recomendação.</span><span class="sxs-lookup"><span data-stu-id="410a5-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="410a5-108">**Trate esse fluxograma como um ponto de partida.**</span><span class="sxs-lookup"><span data-stu-id="410a5-108">**Treat this flowchart as a stating point.**</span></span> <span data-ttu-id="410a5-109">Cada aplicativo tem requisitos exclusivos e, portanto use a recomendação como um ponto de partida.</span><span class="sxs-lookup"><span data-stu-id="410a5-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="410a5-110">Em seguida, execute uma avaliação mais detalhada, observando aspectos como:</span><span class="sxs-lookup"><span data-stu-id="410a5-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="410a5-111">Conjunto de recursos</span><span class="sxs-lookup"><span data-stu-id="410a5-111">Feature set</span></span>
- [<span data-ttu-id="410a5-112">Limites de serviço</span><span class="sxs-lookup"><span data-stu-id="410a5-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="410a5-113">Custo</span><span class="sxs-lookup"><span data-stu-id="410a5-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="410a5-114">SLA</span><span class="sxs-lookup"><span data-stu-id="410a5-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="410a5-115">Disponibilidade regional</span><span class="sxs-lookup"><span data-stu-id="410a5-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="410a5-116">Habilidades da equipe e o ecossistema do desenvolvedor</span><span class="sxs-lookup"><span data-stu-id="410a5-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="410a5-117">Tabelas de comparação de computação</span><span class="sxs-lookup"><span data-stu-id="410a5-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="410a5-118">Se o aplicativo tem várias cargas de trabalho, avalie cada carga de trabalho separadamente.</span><span class="sxs-lookup"><span data-stu-id="410a5-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="410a5-119">Uma solução completa pode incorporar dois ou mais serviços de computação.</span><span class="sxs-lookup"><span data-stu-id="410a5-119">A complete solution may incorporate two or more compute services.</span></span>

![](../images/compute-decision-tree.svg)

