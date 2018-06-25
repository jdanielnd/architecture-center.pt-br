---
title: Árvore de decisão para serviços de computação do Azure
description: Um fluxograma para selecionar um serviço de computação
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: 60bb84d4bf210888d3d43498db043b6e452f6a80
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206515"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="0062f-103">Árvore de decisão para serviços de computação do Azure</span><span class="sxs-lookup"><span data-stu-id="0062f-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="0062f-104">O Azure oferece várias maneiras de hospedar o código do seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="0062f-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="0062f-105">O termo *computação* refere-se ao modelo de hospedagem para os recursos de computação em que seu aplicativo é executado.</span><span class="sxs-lookup"><span data-stu-id="0062f-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="0062f-106">O fluxograma a seguir o ajudará a escolher um serviço de computação para o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="0062f-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="0062f-107">O fluxograma orienta você por um conjunto de critérios de decisão essenciais que geram uma recomendação.</span><span class="sxs-lookup"><span data-stu-id="0062f-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="0062f-108">**Trate esse fluxograma como um ponto de partida.**</span><span class="sxs-lookup"><span data-stu-id="0062f-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="0062f-109">Cada aplicativo tem requisitos exclusivos e, portanto use a recomendação como um ponto de partida.</span><span class="sxs-lookup"><span data-stu-id="0062f-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="0062f-110">Em seguida, execute uma avaliação mais detalhada, observando aspectos como:</span><span class="sxs-lookup"><span data-stu-id="0062f-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="0062f-111">Conjunto de recursos</span><span class="sxs-lookup"><span data-stu-id="0062f-111">Feature set</span></span>
- [<span data-ttu-id="0062f-112">Limites de serviço</span><span class="sxs-lookup"><span data-stu-id="0062f-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="0062f-113">Custo</span><span class="sxs-lookup"><span data-stu-id="0062f-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="0062f-114">SLA</span><span class="sxs-lookup"><span data-stu-id="0062f-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="0062f-115">Disponibilidade regional</span><span class="sxs-lookup"><span data-stu-id="0062f-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="0062f-116">Habilidades da equipe e o ecossistema do desenvolvedor</span><span class="sxs-lookup"><span data-stu-id="0062f-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="0062f-117">Tabelas de comparação de computação</span><span class="sxs-lookup"><span data-stu-id="0062f-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="0062f-118">Se o aplicativo tem várias cargas de trabalho, avalie cada carga de trabalho separadamente.</span><span class="sxs-lookup"><span data-stu-id="0062f-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="0062f-119">Uma solução completa pode incorporar dois ou mais serviços de computação.</span><span class="sxs-lookup"><span data-stu-id="0062f-119">A complete solution may incorporate two or more compute services.</span></span>

## <a name="flowchart"></a><span data-ttu-id="0062f-120">Fluxograma</span><span class="sxs-lookup"><span data-stu-id="0062f-120">Flowchart</span></span>

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="0062f-121">Definições</span><span class="sxs-lookup"><span data-stu-id="0062f-121">Definitions</span></span>

- <span data-ttu-id="0062f-122">**Greenfield** descreve um projeto de software que é completamente novo e criado do zero.</span><span class="sxs-lookup"><span data-stu-id="0062f-122">**Greenfield** describes a software project that is completely new and built from scratch.</span></span> <span data-ttu-id="0062f-123">Ele não inclui códigos herdados.</span><span class="sxs-lookup"><span data-stu-id="0062f-123">It does not include legacy code.</span></span> 

- <span data-ttu-id="0062f-124">**Brownfield** descreve um projeto de software que se baseia em um aplicativo existente.</span><span class="sxs-lookup"><span data-stu-id="0062f-124">**Brownfield** describes a software project that builds on an existing application.</span></span> <span data-ttu-id="0062f-125">Ele pode herdar um código herdado ou estruturas.</span><span class="sxs-lookup"><span data-stu-id="0062f-125">It may inherit legacy code or frameworks.</span></span>

- <span data-ttu-id="0062f-126">**Lift and shift** é uma estratégia de migração de uma carga de trabalho para a nuvem sem recriar o aplicativo ou fazer alterações de código.</span><span class="sxs-lookup"><span data-stu-id="0062f-126">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="0062f-127">Também chamado de *nova hospedagem*.</span><span class="sxs-lookup"><span data-stu-id="0062f-127">Also called *rehosting*.</span></span> <span data-ttu-id="0062f-128">Para obter mais informações, consulte a [Central de migrações do Azure](https://azure.microsoft.com/migration/).</span><span class="sxs-lookup"><span data-stu-id="0062f-128">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="0062f-129">**Nuvem otimizada** é uma estratégia de migração para a nuvem por meio da refatoração de um aplicativo para aproveitar os recursos e funcionalidades de nuvem nativos.</span><span class="sxs-lookup"><span data-stu-id="0062f-129">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="0062f-130">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="0062f-130">Next steps</span></span>

<span data-ttu-id="0062f-131">Para conhecer critérios adicionais a serem considerados, consulte [Critérios para escolher um serviço de computação do Azure](./compute-comparison.md).</span><span class="sxs-lookup"><span data-stu-id="0062f-131">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
