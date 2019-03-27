---
title: Árvore de decisão para serviços de computação do Azure
titleSuffix: Azure Application Architecture Guide
description: Um fluxograma para selecionar um serviço de computação.
author: MikeWasson
ms.date: 11/03/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: e3df1cbdd049e8c40597a85eca29899d8d0d1bc3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245587"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="185fc-103">Árvore de decisão para serviços de computação do Azure</span><span class="sxs-lookup"><span data-stu-id="185fc-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="185fc-104">O Azure oferece várias maneiras de hospedar o código do seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="185fc-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="185fc-105">O termo *computação* refere-se ao modelo de hospedagem para os recursos de computação em que seu aplicativo é executado.</span><span class="sxs-lookup"><span data-stu-id="185fc-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="185fc-106">O fluxograma a seguir o ajudará a escolher um serviço de computação para o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="185fc-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="185fc-107">O fluxograma orienta você por um conjunto de critérios de decisão essenciais que geram uma recomendação.</span><span class="sxs-lookup"><span data-stu-id="185fc-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span>

<span data-ttu-id="185fc-108">**Trate esse fluxograma como um ponto de partida.**</span><span class="sxs-lookup"><span data-stu-id="185fc-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="185fc-109">Cada aplicativo tem requisitos exclusivos e, portanto use a recomendação como um ponto de partida.</span><span class="sxs-lookup"><span data-stu-id="185fc-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="185fc-110">Em seguida, execute uma avaliação mais detalhada, observando aspectos como:</span><span class="sxs-lookup"><span data-stu-id="185fc-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>

- <span data-ttu-id="185fc-111">Conjunto de recursos</span><span class="sxs-lookup"><span data-stu-id="185fc-111">Feature set</span></span>
- [<span data-ttu-id="185fc-112">Limites de serviço</span><span class="sxs-lookup"><span data-stu-id="185fc-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="185fc-113">Custo</span><span class="sxs-lookup"><span data-stu-id="185fc-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="185fc-114">SLA</span><span class="sxs-lookup"><span data-stu-id="185fc-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="185fc-115">Disponibilidade regional</span><span class="sxs-lookup"><span data-stu-id="185fc-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="185fc-116">Habilidades da equipe e o ecossistema do desenvolvedor</span><span class="sxs-lookup"><span data-stu-id="185fc-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="185fc-117">Tabelas de comparação de computação</span><span class="sxs-lookup"><span data-stu-id="185fc-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="185fc-118">Se o aplicativo tem várias cargas de trabalho, avalie cada carga de trabalho separadamente.</span><span class="sxs-lookup"><span data-stu-id="185fc-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="185fc-119">Uma solução completa pode incorporar dois ou mais serviços de computação.</span><span class="sxs-lookup"><span data-stu-id="185fc-119">A complete solution may incorporate two or more compute services.</span></span>

<span data-ttu-id="185fc-120">Para obter mais informações sobre as opções de hospedagem de contêineres no Azure, confira [Contêineres do Azure](https://azure.microsoft.com/overview/containers/).</span><span class="sxs-lookup"><span data-stu-id="185fc-120">For more information about your options for hosting containers in Azure, see [Azure Containers](https://azure.microsoft.com/overview/containers/).</span></span>

## <a name="flowchart"></a><span data-ttu-id="185fc-121">Fluxograma</span><span class="sxs-lookup"><span data-stu-id="185fc-121">Flowchart</span></span>

![Árvore de decisão para serviços de computação do Azure](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="185fc-123">Definições</span><span class="sxs-lookup"><span data-stu-id="185fc-123">Definitions</span></span>

- <span data-ttu-id="185fc-124">**Lift and shift** é uma estratégia de migração de uma carga de trabalho para a nuvem sem recriar o aplicativo ou fazer alterações de código.</span><span class="sxs-lookup"><span data-stu-id="185fc-124">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="185fc-125">Também chamado de *nova hospedagem*.</span><span class="sxs-lookup"><span data-stu-id="185fc-125">Also called *rehosting*.</span></span> <span data-ttu-id="185fc-126">Para obter mais informações, consulte a [Central de migrações do Azure](https://azure.microsoft.com/migration/).</span><span class="sxs-lookup"><span data-stu-id="185fc-126">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="185fc-127">**Nuvem otimizada** é uma estratégia de migração para a nuvem por meio da refatoração de um aplicativo para aproveitar os recursos e funcionalidades de nuvem nativos.</span><span class="sxs-lookup"><span data-stu-id="185fc-127">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="185fc-128">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="185fc-128">Next steps</span></span>

<span data-ttu-id="185fc-129">Para conhecer critérios adicionais a serem considerados, consulte [Critérios para escolher um serviço de computação do Azure](./compute-comparison.md).</span><span class="sxs-lookup"><span data-stu-id="185fc-129">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
