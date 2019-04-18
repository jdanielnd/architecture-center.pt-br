---
title: 'CAF: Empresa de pequeno a médio porte - diretiva corporativa inicial por trás da estratégia de governança'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Empresa de pequeno a médio porte - diretiva corporativa inicial por trás da estratégia de governança
author: BrianBlanchard
ms.openlocfilehash: 2133145c9933ad450ea3cc55ecd68b8a667df783
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640015"
---
# <a name="small-to-medium-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a><span data-ttu-id="72875-103">Empresas de pequeno a médio porte: Política corporativa inicial por trás da estratégia de governança</span><span class="sxs-lookup"><span data-stu-id="72875-103">Small-to-medium enterprise: Initial corporate policy behind the governance strategy</span></span>

<span data-ttu-id="72875-104">A política corporativa a seguir define uma posição inicial de governança que é o ponto de partida para esse percurso.</span><span class="sxs-lookup"><span data-stu-id="72875-104">The following corporate policy defines an initial governance position, which is the starting point for this journey.</span></span> <span data-ttu-id="72875-105">Este artigo define o estágio inicial de riscos, instruções de política inicial e processos antecipados para impor as declarações de política.</span><span class="sxs-lookup"><span data-stu-id="72875-105">This article defines early-stage risks, initial policy statements, and early processes to enforce policy statements.</span></span>

> [!NOTE]
><span data-ttu-id="72875-106">A política corporativa não é um documento técnico, mas conduz muitas decisões técnicas.</span><span class="sxs-lookup"><span data-stu-id="72875-106">The corporate policy is not a technical document, but it drives many technical decisions.</span></span> <span data-ttu-id="72875-107">A governança MVP descrita na [visão geral](./overview.md) deriva, por fim, essa política.</span><span class="sxs-lookup"><span data-stu-id="72875-107">The governance MVP described in the [overview](./overview.md) ultimately derives from this policy.</span></span> <span data-ttu-id="72875-108">Antes de implementar uma governança MVP, sua organização deve desenvolver uma política corporativa com base em seus próprios objetivos e riscos de negócios.</span><span class="sxs-lookup"><span data-stu-id="72875-108">Before implementing a governance MVP, your organization should develop a corporate policy based on your own objectives and business risks.</span></span>

## <a name="cloud-governance-team"></a><span data-ttu-id="72875-109">Equipe de governança de nuvem</span><span class="sxs-lookup"><span data-stu-id="72875-109">Cloud Governance team</span></span>

<span data-ttu-id="72875-110">Nesta narrativa, a equipe de governança de nuvem é composta de administradores de dois sistemas que reconheceram a necessidade de governança.</span><span class="sxs-lookup"><span data-stu-id="72875-110">In this narrative, the Cloud Governance team is comprised of two systems administrators who have recognized the need for governance.</span></span> <span data-ttu-id="72875-111">Nos próximos meses, herdarão o trabalho de limpeza da governança de presença de nuvem da empresa, ganhando o título de Responsáveis da Nuvem.</span><span class="sxs-lookup"><span data-stu-id="72875-111">Over the next several months, they will inherit the job of cleaning up the governance of the company’s cloud presence, earning them the title of Cloud Custodians.</span></span> <span data-ttu-id="72875-112">No evoluções subsequentes, esse título provavelmente será alterado.</span><span class="sxs-lookup"><span data-stu-id="72875-112">In subsequent evolutions, this title will likely change.</span></span>

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a><span data-ttu-id="72875-113">Indicadores de tolerância</span><span class="sxs-lookup"><span data-stu-id="72875-113">Tolerance indicators</span></span>

<span data-ttu-id="72875-114">A tolerância a riscos atual é alta e o apetite de investir em governança de nuvem é baixo.</span><span class="sxs-lookup"><span data-stu-id="72875-114">The current tolerance for risk is high and the appetite for investing in cloud governance is low.</span></span> <span data-ttu-id="72875-115">Dessa forma, os indicadores de tolerância a atuam como um sistema de avisos antecipados para disparar mais investimento de tempo e energia.</span><span class="sxs-lookup"><span data-stu-id="72875-115">As such, the tolerance indicators act as an early warning system to trigger more investment of time and energy.</span></span> <span data-ttu-id="72875-116">Se e quando os seguintes indicadores forem observados, você deve evoluir a estratégia de governança.</span><span class="sxs-lookup"><span data-stu-id="72875-116">If and when the following indicators are observed, you should evolve the governance strategy.</span></span>

- <span data-ttu-id="72875-117">Gerenciamento de custos: A escala de implantação excede 100 ativos para a nuvem ou de gastos mensal excede 100 milhões de dólares por mês.</span><span class="sxs-lookup"><span data-stu-id="72875-117">Cost Management: The scale of deployment exceeds 100 assets to the cloud, or monthly spending exceeds $1,000 USD per month.</span></span>
- <span data-ttu-id="72875-118">Linha de base de segurança: Inclusão dos dados protegidos em planos de adoção definida da nuvem.</span><span class="sxs-lookup"><span data-stu-id="72875-118">Security Baseline: Inclusion of protected data in defined cloud adoption plans.</span></span>
- <span data-ttu-id="72875-119">Consistência de Recursos: Inclusão de todos os aplicativos críticos em planos de adoção definida da nuvem.</span><span class="sxs-lookup"><span data-stu-id="72875-119">Resource Consistency: Inclusion of any mission-critical applications in defined cloud adoption plans.</span></span>

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a><span data-ttu-id="72875-120">Próximos passos</span><span class="sxs-lookup"><span data-stu-id="72875-120">Next steps</span></span>

<span data-ttu-id="72875-121">Essa política corporativa prepara a equipe de governança de nuvem para implementar a governança MVP, que será a base para a adoção.</span><span class="sxs-lookup"><span data-stu-id="72875-121">This corporate policy prepares the Cloud Governance team to implement the governance MVP, which will be the foundation for adoption.</span></span> <span data-ttu-id="72875-122">A próxima etapa é implementar esse MVP.</span><span class="sxs-lookup"><span data-stu-id="72875-122">The next step is to implement this MVP.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="72875-123">Melhor prática explicada</span><span class="sxs-lookup"><span data-stu-id="72875-123">Best practice explained</span></span>](./best-practice-explained.md)