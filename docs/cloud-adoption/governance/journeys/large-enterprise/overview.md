---
title: 'CAF: Percurso de governança de empresas de grande porte'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Percurso de governança de empresas de grande porte
author: BrianBlanchard
ms.openlocfilehash: 689b600524c3be6c505ade8c5aaa447d772c6e35
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245237"
---
# <a name="caf-large-enterprise-governance-journey"></a><span data-ttu-id="51133-103">CAF: Percurso de governança de empresas de grande porte</span><span class="sxs-lookup"><span data-stu-id="51133-103">CAF: Large enterprise governance journey</span></span>

## <a name="best-practice-overview"></a><span data-ttu-id="51133-104">Visão geral de prática recomendada</span><span class="sxs-lookup"><span data-stu-id="51133-104">Best practice overview</span></span>

<span data-ttu-id="51133-105">Essa jornada de governança segue as experiências de uma empresa fictícia por vários estágios de maturidade de governança.</span><span class="sxs-lookup"><span data-stu-id="51133-105">This governance journey follows the experiences of a fictional company through various stages of governance maturity.</span></span> <span data-ttu-id="51133-106">Ela se baseia em percursos de clientes reais.</span><span class="sxs-lookup"><span data-stu-id="51133-106">It is based on real customer journeys.</span></span> <span data-ttu-id="51133-107">As práticas recomendadas sugeridas baseiam-se nas restrições e necessidades da empresa fictícia.</span><span class="sxs-lookup"><span data-stu-id="51133-107">The suggested best practices are based on the constraints and needs of the fictional company.</span></span>

<span data-ttu-id="51133-108">Como um ponto de partida rápido, esta visão geral define um produto mínimo viável (MVP) para um controle com base em práticas recomendadas.</span><span class="sxs-lookup"><span data-stu-id="51133-108">As a quick starting point, this overview defines a minimum viable product (MVP) for governance based on best practices.</span></span> <span data-ttu-id="51133-109">Também fornece links para algumas evoluções de governança que adicionam ainda mais as práticas recomendadas conforme novos negócios ou riscos técnicos surgirem.</span><span class="sxs-lookup"><span data-stu-id="51133-109">It also provides links to some governance evolutions that add further best practices as new business or technical risks emerge.</span></span>

> [!WARNING]
> <span data-ttu-id="51133-110">Este MVP é um ponto de partida de linha de base, com base em um conjunto de suposições.</span><span class="sxs-lookup"><span data-stu-id="51133-110">This MVP is a baseline starting point, based on a set of assumptions.</span></span> <span data-ttu-id="51133-111">Até mesmo esse conjunto mínimo de práticas recomendadas tem como base as políticas corporativas orientadas por riscos comerciais exclusivos e as tolerâncias de risco.</span><span class="sxs-lookup"><span data-stu-id="51133-111">Even this minimal set of best practices is based on corporate policies driven by unique business risks and risk tolerances.</span></span> <span data-ttu-id="51133-112">Para ver se essas suposições se aplicam a você, leia a [narrativa mais longa](./narrative.md) que acompanha este artigo.</span><span class="sxs-lookup"><span data-stu-id="51133-112">To see if these assumptions apply to you, read the [longer narrative](./narrative.md) that follows this article.</span></span>

### <a name="governance-best-practice"></a><span data-ttu-id="51133-113">Prática recomendada de governança</span><span class="sxs-lookup"><span data-stu-id="51133-113">Governance best practice</span></span>

<span data-ttu-id="51133-114">Esta prática recomendada serve como uma base que uma organização pode usar para adicionar as grades de proteção de governança de forma rápida e consistente entre várias assinaturas do Azure.</span><span class="sxs-lookup"><span data-stu-id="51133-114">This best practice serves as a foundation that an organization can use to quickly and consistently add governance guardrails across multiple Azure subscriptions.</span></span>

### <a name="resource-organization"></a><span data-ttu-id="51133-115">Organização do recurso</span><span class="sxs-lookup"><span data-stu-id="51133-115">Resource organization</span></span>

<span data-ttu-id="51133-116">O diagrama a seguir mostra a hierarquia de MVP de governança para organizar recursos.</span><span class="sxs-lookup"><span data-stu-id="51133-116">The following diagram shows the governance MVP hierarchy for organizing resources.</span></span>

![Diagrama de organização do recurso](../../../_images/governance/resource-organization.png)

<span data-ttu-id="51133-118">Todos os aplicativos devem ser implantados na área apropriada do grupo de gerenciamento, assinatura e hierarquia de grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="51133-118">Every application should be deployed in the proper area of the management group, subscription, and resource group hierarchy.</span></span> <span data-ttu-id="51133-119">Durante o planejamento da implantação, a equipe de governança de nuvem criará os nós necessários na hierarquia para capacitar as equipes de adoção da nuvem.</span><span class="sxs-lookup"><span data-stu-id="51133-119">During deployment planning, the Cloud Governance team will create the necessary nodes in the hierarchy to empower the cloud adoption teams.</span></span>

1. <span data-ttu-id="51133-120">Um grupo de gerenciamento para cada unidade de negócios com uma hierarquia detalhada que reflete a geografia e em seguida, o tipo de ambiente (produção, não produção).</span><span class="sxs-lookup"><span data-stu-id="51133-120">A management group for each business unit with a detailed hierarchy that reflects geography then environment type (Production, Non-Production).</span></span>
2. <span data-ttu-id="51133-121">Uma assinatura para cada combinação exclusiva de unidade de negócios, geografia, ambiente e "Categorização de aplicativo".</span><span class="sxs-lookup"><span data-stu-id="51133-121">A subscription for each unique combination of business unit, geography, environment, and "Application Categorization."</span></span>
3. <span data-ttu-id="51133-122">Um grupo de recursos separado para cada aplicativo.</span><span class="sxs-lookup"><span data-stu-id="51133-122">A separate resource group for each application.</span></span>
4. <span data-ttu-id="51133-123">A nomenclatura consistente deve ser aplicada em cada nível dessa hierarquia de agrupamento.</span><span class="sxs-lookup"><span data-stu-id="51133-123">Consistent nomenclature should be applied at each level of this grouping hierarchy.</span></span>

![Diagrama de organização de recurso de empresa de grande porte](../../../_images/governance/large-enterprise-resource-organization.png)

<span data-ttu-id="51133-125">Esses padrões fornecem espaço para crescimento sem complicar a hierarquia desnecessariamente.</span><span class="sxs-lookup"><span data-stu-id="51133-125">These patterns provide room for growth without complicating the hierarchy unnecessarily.</span></span>

[!INCLUDE [governance-of-resources](../../../../../includes/cloud-adoption/governance/governance-of-resources.md)]

## <a name="governance-evolutions"></a><span data-ttu-id="51133-126">Evoluções de governança</span><span class="sxs-lookup"><span data-stu-id="51133-126">Governance evolutions</span></span>

<span data-ttu-id="51133-127">Quando esse MVP tiver sido implantado, camadas adicionais de governança podem ser incorporadas rapidamente ao ambiente.</span><span class="sxs-lookup"><span data-stu-id="51133-127">Once this MVP has been deployed, additional layers of governance can be quickly incorporated into the environment.</span></span> <span data-ttu-id="51133-128">Aqui estão algumas maneiras de desenvolver o MVP para atender às necessidades específicas de negócios:</span><span class="sxs-lookup"><span data-stu-id="51133-128">Here are some ways to evolve the MVP to meet specific business needs:</span></span>

- [<span data-ttu-id="51133-129">Linha de base para dados protegidos</span><span class="sxs-lookup"><span data-stu-id="51133-129">Security Baseline for protected data</span></span>](./security-baseline-evolution.md)
- [<span data-ttu-id="51133-130">Configurações de recurso para aplicativos de missão crítica</span><span class="sxs-lookup"><span data-stu-id="51133-130">Resource configurations for mission-critical applications</span></span>](./resource-consistency-evolution.md)
- [<span data-ttu-id="51133-131">Controles para Gerenciamento de Custos</span><span class="sxs-lookup"><span data-stu-id="51133-131">Controls for Cost Management</span></span>](./cost-management-evolution.md)
- [<span data-ttu-id="51133-132">Controles para evolução de várias nuvens</span><span class="sxs-lookup"><span data-stu-id="51133-132">Controls for multi-cloud evolution</span></span>](./multi-cloud-evolution.md)

<!-- markdownlint-disable MD026 -->

## <a name="what-does-this-best-practice-do"></a><span data-ttu-id="51133-133">O que faz essa prática recomendada?</span><span class="sxs-lookup"><span data-stu-id="51133-133">What does this best practice do?</span></span>

<span data-ttu-id="51133-134">No MVP, nas práticas recomendadas e nas ferramentas da disciplina [Aceleração de implantação](../../deployment-acceleration/overview.md), são estabelecidas para aplicar rapidamente a política corporativa.</span><span class="sxs-lookup"><span data-stu-id="51133-134">In the MVP, practices and tools from the [Deployment Acceleration](../../deployment-acceleration/overview.md) discipline are established to quickly apply corporate policy.</span></span> <span data-ttu-id="51133-135">Em particular, o MVP usa o Azure Blueprints, Azure Policy e grupos de gerenciamento do Azure para aplicar algumas políticas corporativas básicas, conforme definido na narrativa para esta empresa fictícia.</span><span class="sxs-lookup"><span data-stu-id="51133-135">In particular, the MVP uses Azure Blueprints, Azure Policy, and Azure management groups to apply a few basic corporate policies, as defined in the narrative for this fictional company.</span></span> <span data-ttu-id="51133-136">Essas políticas corporativas são aplicadas usando modelos do Azure Resource Manager e as políticas do Azure para estabelecer uma linha de base muito pequena para identidade e segurança.</span><span class="sxs-lookup"><span data-stu-id="51133-136">Those corporate policies are applied using Azure Resource Manager templates and Azure policies to establish a very small baseline for identity and security.</span></span>

![Exemplo de MVP de governança incremental](../../../_images/governance/governance-mvp.png)

## <a name="evolving-the-best-practice"></a><span data-ttu-id="51133-138">Evoluindo a prática recomendada</span><span class="sxs-lookup"><span data-stu-id="51133-138">Evolving the best practice</span></span>

<span data-ttu-id="51133-139">Ao longo do tempo, esse controle MVP será usado para desenvolver as práticas de governança.</span><span class="sxs-lookup"><span data-stu-id="51133-139">Over time, this governance MVP will be used to evolve the governance practices.</span></span> <span data-ttu-id="51133-140">Conforme a adoção avança, aumenta o risco de negócios.</span><span class="sxs-lookup"><span data-stu-id="51133-140">As adoption advances, business risk grows.</span></span> <span data-ttu-id="51133-141">Várias disciplinas dentro do modelo de governança CAF evoluem para atenuar esses riscos.</span><span class="sxs-lookup"><span data-stu-id="51133-141">Various disciplines within the CAF governance model will evolve to mitigate those risks.</span></span> <span data-ttu-id="51133-142">Artigos mais adiante nesta série discutem a evolução da política corporativa que afeta a empresa fictícia.</span><span class="sxs-lookup"><span data-stu-id="51133-142">Later articles in this series discuss the evolution of corporate policy affecting the fictional company.</span></span> <span data-ttu-id="51133-143">Três evoluções ocorrem em três disciplinas:</span><span class="sxs-lookup"><span data-stu-id="51133-143">These evolutions happen across three disciplines:</span></span>

- <span data-ttu-id="51133-144">Linha de base de identidade, como dependências da migração evolui a narrativa</span><span class="sxs-lookup"><span data-stu-id="51133-144">Identity Baseline, as migration dependencies evolve in the narrative</span></span>
- <span data-ttu-id="51133-145">Gerenciamento de Custos como escalas de adoção.</span><span class="sxs-lookup"><span data-stu-id="51133-145">Cost Management, as adoption scales.</span></span>
- <span data-ttu-id="51133-146">Linha de base de segurança, como os dados protegidos são implantados.</span><span class="sxs-lookup"><span data-stu-id="51133-146">Security Baseline, as protected data is deployed.</span></span>
- <span data-ttu-id="51133-147">Consistência de recursos, como operações de TI começam a dar suporte a cargas de trabalho de missão crítica.</span><span class="sxs-lookup"><span data-stu-id="51133-147">Resource Consistency, as IT Operations begins supporting mission-critical workloads.</span></span>

![Exemplo de MVP de governança incremental](../../../_images/governance/governance-evolution-large.png)

## <a name="next-steps"></a><span data-ttu-id="51133-149">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="51133-149">Next steps</span></span>

<span data-ttu-id="51133-150">Agora que você está familiarizado com a governança MVP e tem uma ideia das evoluções de governança a seguir, leia a narrativa de suporte para o contexto adicional.</span><span class="sxs-lookup"><span data-stu-id="51133-150">Now that you’re familiar with the governance MVP and have an idea of the governance evolutions to follow, read the supporting narrative for additional context.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="51133-151">Leia a narrativa de suporte</span><span class="sxs-lookup"><span data-stu-id="51133-151">Read the supporting narrative</span></span>](./narrative.md)
