---
title: 'CAF: Alinhar os guias de design com a política.'
description: Como alinhar os guias de design com a política?
author: BrianBlanchard
ms.date: 01/04/2019
ms.openlocfilehash: 77a35597585e5f58967ea79d3c7b0fa17b6ab80e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241587"
---
<!---
I've established policies. How to help developers adopt these policies?
Draft an architecture design guide.

[Aspirational statement] If you're using Azure, you can use one of ours as a starting point. The choose one of the following 6 as a starting point and mold it to fit your policies.
--->

<!-- markdownlint-disable MD026 -->

# <a name="how-do-you-align-design-guides-with-policy"></a><span data-ttu-id="6e5a2-103">Como alinhar os guias de design com a política?</span><span class="sxs-lookup"><span data-stu-id="6e5a2-103">How do you align design guides with policy?</span></span>

<span data-ttu-id="6e5a2-104">Depois de ter [definido as políticas de nuvem](define-policy.md) com base nos seus [riscos identificados](understanding-business-risk.md), você precisará gerar orientação prática que se alinha a essas políticas para sua equipe de TI e desenvolvedores se refiram.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-104">After you've [defined cloud policies](define-policy.md) based on your [identified risks](understanding-business-risk.md), you'll need to generate actionable guidance that aligns with these policies for your IT staff and developers to refer to.</span></span> <span data-ttu-id="6e5a2-105">Esboçar um guia de design de governança de nuvem permite que você especifique escolhas específicas, estruturais, tecnológicas e processuais com base nas instruções de política que você gerou para cada uma das cinco disciplinas de governança.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-105">Drafting a cloud governance design guide allows you to specify specific structural, technological, and process choices based on the policy statements you generated for each of the five governance disciplines.</span></span>

<span data-ttu-id="6e5a2-106">Um guia de design de governança de nuvem deve estabelecer as escolhas de arquitetura e padrões de design para cada um dos componentes básicos da infraestrutura de implantações de nuvem que melhor atendem às suas necessidades de política.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-106">A cloud governance design guide should establish the architecture choices and design patterns for each of the core infrastructure components of cloud deployments that best meet your policy requirements.</span></span> <span data-ttu-id="6e5a2-107">De forma paralela, forneça uma explicação de alto nível a respeito das ferramentas de tecnologia e processos que darão suporte a essas decisões de design.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-107">Alongside these you should provide a high-level explanation of the technology, tools, and processes that will support each of these design decisions.</span></span>

<span data-ttu-id="6e5a2-108">Embora suas instruções de análise e a política de risco possam, até certo ponto, ser independente de plataforma de nuvem, o guia de design deve fornecer os detalhes de implementação específicos da plataforma da sua equipe de TI e desenvolvimento.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-108">Although your risk analysis and policy statements may, to some degree, be cloud platform agnostic, your design guide should provide platform-specific implementation details that your IT and dev.</span></span> <span data-ttu-id="6e5a2-109">Concentre-se na arquitetura, ferramentas e recursos da sua plataforma escolhidos ao tomar a decisão de design e fornecimento de diretrizes.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-109">Focus on the architecture, tools, and features of your chosen platform when making design decision and providing guidance.</span></span>

<span data-ttu-id="6e5a2-110">Enquanto os guias de design de nuvem devem levar em consideração alguns dos detalhes técnicos associados a cada componente de infraestrutura, eles não devem ser abranger documentos técnicos ou especificações.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-110">While cloud design guides should take into account some of the technical details associated with each infrastructure component, they are not meant to be extensive technical documents or specifications.</span></span> <span data-ttu-id="6e5a2-111">Certifique-se de que suas guias abordem todas as instruções de política e informem claramente as decisões de design em formato fácil para a equipe entender e referenciar.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-111">Make sure your guides address all of your policy statements and clearly state design decisions in a format easy for staff to understand and reference.</span></span>

<!-- markdownlint-enable MD033 -->

## <a name="using-the-actionable-governance-journeys"></a><span data-ttu-id="6e5a2-112">Usar os percursos de governança acionáveis</span><span class="sxs-lookup"><span data-stu-id="6e5a2-112">Using the actionable governance journeys</span></span>

<span data-ttu-id="6e5a2-113">Se você estiver planejando usar a plataforma do Azure para a adoção da nuvem, o CAF fornece [percursos de governança](../journeys/overview.md) que ilustram a abordagem incremental do modelo de governança do CAF.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-113">If you're planning to use the Azure platform for your cloud adoption, the CAF provides [governance journeys](../journeys/overview.md) illustrating the incremental approach of the CAF governance model.</span></span> <span data-ttu-id="6e5a2-114">Esses percursos de narrativa abrangem uma variedade de cenários comuns de adoção, incluindo os riscos de negócios, requisitos de tolerância e instruções de políticas que se tornaram a criação de um produto viável mínimo de governança (MVP).</span><span class="sxs-lookup"><span data-stu-id="6e5a2-114">These narrative journeys cover a range of common adoption scenarios, including the business risks, tolerance requirements, and policy statements that went into creating a governance minimum viable product (MVP).</span></span> <span data-ttu-id="6e5a2-115">Esses percursos representam uma síntese de experiência de clientes do mundo real do processo de adoção de nuvem no Azure.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-115">These journeys represent a synthesis of real-world customer experience of the cloud adoption process in Azure.</span></span>

<span data-ttu-id="6e5a2-116">Enquanto cada adoção de nuvem tiver metas exclusivas, prioridades e desafios, estes exemplos devem fornecer um bom modelo para converter sua política em orientação.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-116">While every cloud adoption has unique goals, priorities, and challenges, these samples should provide a good template for converting your policy into guidance.</span></span> <span data-ttu-id="6e5a2-117">Escolha o cenário mais próximo à sua situação como ponto de partida e molde-o para atender às suas necessidades de política específica.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-117">Pick the closest scenario to your situation as a starting point, and mold it to fit your specific policy needs.</span></span>

## <a name="next-steps"></a><span data-ttu-id="6e5a2-118">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="6e5a2-118">Next steps</span></span>

<span data-ttu-id="6e5a2-119">Com a diretriz de design no local, estabeleça processos de adesão da política para garantir a conformidade de política.</span><span class="sxs-lookup"><span data-stu-id="6e5a2-119">With design guidance in place, establish policy adherence processes to ensure policy compliance.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="6e5a2-120">Processos de conformidade de política</span><span class="sxs-lookup"><span data-stu-id="6e5a2-120">Policy adherence processes</span></span>](processes.md)