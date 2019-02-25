---
title: 'CAF: Empresas de grande porte - política corporativa inicial por trás da estratégia de governança'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: Empresas de grande porte - política corporativa inicial por trás da estratégia de governança.
author: BrianBlanchard
ms.openlocfilehash: d3bc31f95229a82cbc2f6ac6e684a87107783c07
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900372"
---
# <a name="large-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a><span data-ttu-id="eee89-103">Empresas de grande porte: Política corporativa inicial por trás da estratégia de governança</span><span class="sxs-lookup"><span data-stu-id="eee89-103">Large enterprise: Initial corporate policy behind the governance strategy</span></span>

<span data-ttu-id="eee89-104">A política corporativa a seguir define uma posição inicial de governança que é o ponto de partida para esse percurso.</span><span class="sxs-lookup"><span data-stu-id="eee89-104">The following corporate policy defines the initial governance position, which is the starting point for this journey.</span></span> <span data-ttu-id="eee89-105">Este artigo define o estágio inicial de riscos, instruções de política inicial e processos antecipados para impor as declarações de política.</span><span class="sxs-lookup"><span data-stu-id="eee89-105">This article defines early-stage risks, initial policy statements, and early processes to enforce policy statements.</span></span>

> [!NOTE]
><span data-ttu-id="eee89-106">A política corporativa não é um documento técnico, mas conduz muitas decisões técnicas.</span><span class="sxs-lookup"><span data-stu-id="eee89-106">The corporate policy is not a technical document, but it drives many technical decisions.</span></span> <span data-ttu-id="eee89-107">A governança MVP descrita na [visão geral](./overview.md) deriva, por fim, essa política.</span><span class="sxs-lookup"><span data-stu-id="eee89-107">The governance MVP described in the [overview](./overview.md) ultimately derives from this policy.</span></span> <span data-ttu-id="eee89-108">Antes de implementar uma governança MVP, sua organização deve desenvolver uma política corporativa com base em seus próprios objetivos e riscos de negócios.</span><span class="sxs-lookup"><span data-stu-id="eee89-108">Before implementing a governance MVP, your organization should develop a corporate policy based on your own objectives and business risks.</span></span>

## <a name="cloud-governance-team"></a><span data-ttu-id="eee89-109">Equipe de governança de nuvem</span><span class="sxs-lookup"><span data-stu-id="eee89-109">Cloud Governance team</span></span>

<span data-ttu-id="eee89-110">A CIO realizou recentemente uma reunião com a equipe de governança de TI para entender o histórico das políticas de missão crítica e de PII e examinar o impacto de alteração dessas políticas.</span><span class="sxs-lookup"><span data-stu-id="eee89-110">The CIO recently held a meeting with the IT Governance team to understand the history of the PII and mission-critical policies and review the impact of changing those policies.</span></span> <span data-ttu-id="eee89-111">Ela também discutiu o potencial geral da nuvem para IT e a empresa.</span><span class="sxs-lookup"><span data-stu-id="eee89-111">She also discussed the overall potential of the cloud for IT and the company.</span></span>

<span data-ttu-id="eee89-112">Após a reunião, os dois membros da equipe de governança de TI solicitaram permissão para pesquisar e dar suporte a iniciativas de planejamento de nuvem.</span><span class="sxs-lookup"><span data-stu-id="eee89-112">After the meeting, two members of the IT Governance team requested permission to research and support the cloud planning efforts.</span></span> <span data-ttu-id="eee89-113">Ao reconhecer a necessidade de governança e uma oportunidade para limitar a TI invisível, o Diretor de Governança de TI apoiou essa ideia.</span><span class="sxs-lookup"><span data-stu-id="eee89-113">Recognizing the need for governance and an opportunity to limit shadow IT, the Director of IT Governance supported this idea.</span></span> <span data-ttu-id="eee89-114">Com isso, a equipe de governança de nuvem nasceu.</span><span class="sxs-lookup"><span data-stu-id="eee89-114">With that, the Cloud Governance team was born.</span></span> <span data-ttu-id="eee89-115">Nos próximos meses, ela herdará a limpeza dos muitos erros cometidos durante a exploração na nuvem de uma perspectiva de governança.</span><span class="sxs-lookup"><span data-stu-id="eee89-115">Over the next several months, they will inherit the cleanup of many mistakes made during exploration in the cloud from a governance perspective.</span></span> <span data-ttu-id="eee89-116">Isso lhes dará o apelido de Responsáveis da Nuvem.</span><span class="sxs-lookup"><span data-stu-id="eee89-116">This will earn them the moniker of Cloud Custodians.</span></span> <span data-ttu-id="eee89-117">Nas evoluções posteriores, essa jornada mostrará como suas funções mudam ao longo do tempo.</span><span class="sxs-lookup"><span data-stu-id="eee89-117">In later evolutions, this journey will show how their roles change over time.</span></span>

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a><span data-ttu-id="eee89-118">Indicadores de tolerância</span><span class="sxs-lookup"><span data-stu-id="eee89-118">Tolerance indicators</span></span>

<span data-ttu-id="eee89-119">A tolerância a riscos atual é alta e o apetite de investir em governança de nuvem é baixo.</span><span class="sxs-lookup"><span data-stu-id="eee89-119">The current risk tolerance is high and the appetite for investing in cloud governance is low.</span></span> <span data-ttu-id="eee89-120">Dessa forma, os indicadores de tolerância atuam como um sistema de avisos antecipados para disparar mais investimento de tempo e energia.</span><span class="sxs-lookup"><span data-stu-id="eee89-120">As such, the tolerance indicators act as an early warning system to trigger the investment of time and energy.</span></span> <span data-ttu-id="eee89-121">Se os seguintes indicadores forem observados, você deve evoluir a estratégia de governança.</span><span class="sxs-lookup"><span data-stu-id="eee89-121">If the following indicators are observed, it would be wise to evolve the governance strategy.</span></span>

- <span data-ttu-id="eee89-122">Gerenciamento de Custos: A escala de implantação excede 1.000 ativos para a nuvem ou os gastos mensal excedem 100 milhões de dólares por mês.</span><span class="sxs-lookup"><span data-stu-id="eee89-122">Cost Management: Scale of deployment exceeds 1,000 assets to the cloud, or monthly spending exceeds $10,000 USD per month.</span></span>
- <span data-ttu-id="eee89-123">Linha de base de identidade: Inclusão de aplicativos com requisitos de autenticação multifator herdada ou de terceiros (MFA).</span><span class="sxs-lookup"><span data-stu-id="eee89-123">Identity Baseline: Inclusion of applications with legacy or third-party multifactor authentication (MFA) requirements.</span></span>
- <span data-ttu-id="eee89-124">Linha de Base de Segurança: Inclusão dos dados protegidos em planos de adoção definida da nuvem.</span><span class="sxs-lookup"><span data-stu-id="eee89-124">Security Baseline: Inclusion of protected data in defined cloud adoption plans.</span></span>
- <span data-ttu-id="eee89-125">Consistência de Recursos: Inclusão de todos os aplicativos críticos em planos de adoção definida da nuvem.</span><span class="sxs-lookup"><span data-stu-id="eee89-125">Resource Consistency: Inclusion of any mission-critical applications in defined cloud adoption plans.</span></span>

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a><span data-ttu-id="eee89-126">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="eee89-126">Next steps</span></span>

<span data-ttu-id="eee89-127">Essa política corporativa prepara a equipe de governança de nuvem para implementar a governança MVP, que será a base para a adoção.</span><span class="sxs-lookup"><span data-stu-id="eee89-127">This corporate policy prepares the Cloud Governance team to implement the governance MVP, which will be the foundation for adoption.</span></span> <span data-ttu-id="eee89-128">A próxima etapa é implementar esse MVP.</span><span class="sxs-lookup"><span data-stu-id="eee89-128">The next step is to implement this MVP.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="eee89-129">Melhor prática explicada</span><span class="sxs-lookup"><span data-stu-id="eee89-129">Best practice explained</span></span>](./best-practice-explained.md)