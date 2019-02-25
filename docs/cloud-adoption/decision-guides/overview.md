---
title: 'CAF: Guias de decisão de arquitetura'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Saiba mais sobre guias de decisão de arquitetura no Cloud Adoption Framework.
author: rotycenh
ms.openlocfilehash: b43047b5fc5b636bc84b9f28ec24846730e63672
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900359"
---
# <a name="architectural-decision-guides"></a><span data-ttu-id="37436-103">Guias de decisão de arquitetura</span><span class="sxs-lookup"><span data-stu-id="37436-103">Architectural decision guides</span></span>

<span data-ttu-id="37436-104">Guias de decisão de arquitetura em que a estrutura de adoção da nuvem descrevem padrões e modelos que ajudam durante a criação de diretrizes de design de governança de nuvem.</span><span class="sxs-lookup"><span data-stu-id="37436-104">The architectural decision guides in the Cloud Adoption Framework describe patterns and models that help when creating cloud governance design guidance.</span></span> <span data-ttu-id="37436-105">Cada guia de decisão se concentra em um componente de infraestrutura de núcleo de implantações de nuvem e lista potenciais padrões ou modelos que se destinam a dar suporte a cenários de implantação de nuvem específica.</span><span class="sxs-lookup"><span data-stu-id="37436-105">Each decision guide focuses on one core infrastructure component of cloud deployments and lists potential patterns or models intended to support specific cloud deployment scenarios.</span></span>

<span data-ttu-id="37436-106">Quando você começa a estabelecer a governança de nuvem para sua organização, percursos de governança acionáveis fornecem um roteiro de linha de base.</span><span class="sxs-lookup"><span data-stu-id="37436-106">When you begin to establish cloud governance for your organization,  actionable governance journeys provide a baseline roadmap.</span></span> <span data-ttu-id="37436-107">No entanto, esses percursos fazem suposições sobre os requisitos e as prioridades que podem não refletir àquelas de sua organização.</span><span class="sxs-lookup"><span data-stu-id="37436-107">However, these journeys make assumptions about requirements and priorities that may not reflect those of your organization.</span></span>
<span data-ttu-id="37436-108">Esses guias de decisão complementam os percursos de governança de exemplo, fornecendo padrões alternativos e modelos que ajudam você a alinhar as escolhas de design arquitetônico feitas nas diretrizes de design de exemplo pelos seus próprios requisitos.</span><span class="sxs-lookup"><span data-stu-id="37436-108">These decision guides supplement the sample governance journeys by providing alternative patterns and models that help you align the architectural design choices made in the example design guidance with your own requirements.</span></span>

## <a name="design-guidance-categories"></a><span data-ttu-id="37436-109">Categorias de diretrizes de design</span><span class="sxs-lookup"><span data-stu-id="37436-109">Design guidance categories</span></span>

<span data-ttu-id="37436-110">Cada uma das categorias a seguir representa uma tecnologia de base de todas as implantações de nuvem.</span><span class="sxs-lookup"><span data-stu-id="37436-110">Each of the following categories represents a foundational technology of all cloud deployments.</span></span> <span data-ttu-id="37436-111">Para cada opção nos percursos do design de exemplo que não correspondem aos seus requisitos, examine as opções na seção relevante abaixo para escolher um padrão ou o modelo melhor adequado às suas necessidades.</span><span class="sxs-lookup"><span data-stu-id="37436-111">For each choice in the example design journeys that doesn’t match your requirements, examine the options in the relevant section below to choose a pattern or model better suited to your needs.</span></span>

<span data-ttu-id="37436-112">[Assinaturas](./subscriptions/overview.md): Planeje a estrutura de design e conta de assinatura da sua implantação para corresponder com a propriedade da sua organização, faturamento e capacidades de gerenciamento.</span><span class="sxs-lookup"><span data-stu-id="37436-112">[Subscriptions](./subscriptions/overview.md): Plan your cloud deployment's subscription design and account structure to match your organization's ownership, billing, and management capabilities.</span></span>

<span data-ttu-id="37436-113">[Identidade](./identity/overview.md): Integre os serviços de identidade baseado em nuvem aos seus recursos de identidade existentes para dar suporte ao controle de acesso em seu ambiente de TI.</span><span class="sxs-lookup"><span data-stu-id="37436-113">[Identity](./identity/overview.md): Integrate cloud-based identity services with your existing identity resources to support access control within your IT environment.</span></span>

<span data-ttu-id="37436-114">[Imposição de Política](./policy-enforcement/overview.md): Definir e impor regras de política organizacional de recursos e cargas de trabalho que você implanta na nuvem.</span><span class="sxs-lookup"><span data-stu-id="37436-114">[Policy Enforcement](./policy-enforcement/overview.md): Define and enforce organizational policy rules for resources and workloads that you deploy to the cloud.</span></span>

<span data-ttu-id="37436-115">[Consistência de Recursos](./resource-consistency/overview.md): Certifique-se que implantação e a organização dos seus recursos baseados em nuvem se alinham para impor requisitos de gerenciamento e a política de recursos.</span><span class="sxs-lookup"><span data-stu-id="37436-115">[Resource Consistency](./resource-consistency/overview.md): Ensure that deployment and organization of your cloud-based resources align to enforce resource management and policy requirements.</span></span>

<span data-ttu-id="37436-116">[Marcação de Recursos](./resource-tagging/overview.md): Organize seus recursos baseados em nuvem para dar suporte a modelos de cobrança, abordagens de contabilidade da nuvem, gerenciamento, controle de acesso e eficiência operacional.</span><span class="sxs-lookup"><span data-stu-id="37436-116">[Resource Tagging](./resource-tagging/overview.md): Organize your cloud-based resources to support billing models, cloud accounting approaches, management, access control, and operational efficiency.</span></span> <span data-ttu-id="37436-117">A marcação de recursos exige um esquema de nomenclatura e de metadados consistente e bem organizado.</span><span class="sxs-lookup"><span data-stu-id="37436-117">Resource tagging requires a consistent and well-organized naming and metadata scheme.</span></span>

<span data-ttu-id="37436-118">[Redes definidas por software](./software-defined-network/overview.md): Implante cargas de trabalho seguras para a nuvem usando a implantação rápida e modificação de recursos de rede virtualizados.</span><span class="sxs-lookup"><span data-stu-id="37436-118">[Software Defined Networks](./software-defined-network/overview.md): Deploy secure workloads to the cloud using rapid deployment and modification of virtualized networking capabilities.</span></span> <span data-ttu-id="37436-119">Redes definidas por software (SDNs) podem dar suporte a fluxos de trabalho ágeis, isolar os recursos e integrar sistemas baseados em nuvem com sua infraestrutura de TI existente.</span><span class="sxs-lookup"><span data-stu-id="37436-119">Software-defined networks (SDNs) can support agile workflows, isolate resources, and integrate cloud-based systems with your existing IT infrastructure.</span></span>

<span data-ttu-id="37436-120">[Criptografia](./encryption/overview.md): Proteja seus dados confidenciais usando a criptografia, um aspecto importante da segurança dentro de uma implantação de nuvem.</span><span class="sxs-lookup"><span data-stu-id="37436-120">[Encryption](./encryption/overview.md): Secure your sensitive data using encryption, an important aspect of security within a cloud deployment.</span></span>

<span data-ttu-id="37436-121">[Logs e relatórios](./log-and-report/overview.md): Monitorar dados de log gerados pelos recursos baseados em nuvem.</span><span class="sxs-lookup"><span data-stu-id="37436-121">[Logs and Reporting](./log-and-report/overview.md): Monitor log data generated by cloud-based resources.</span></span> <span data-ttu-id="37436-122">A análise de dados fornece insights relacionados à integridade nas operações, a manutenção e o status de imposição da política de cargas de trabalho.</span><span class="sxs-lookup"><span data-stu-id="37436-122">Analyzing data provides health-related insights into the operations, maintenance, and policy enforcement status of workloads.</span></span>

## <a name="next-steps"></a><span data-ttu-id="37436-123">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="37436-123">Next steps</span></span>

<span data-ttu-id="37436-124">Saiba como as assinaturas e contas servem como a base de uma implantação de nuvem.</span><span class="sxs-lookup"><span data-stu-id="37436-124">Learn how subscriptions and accounts serve as the base of a cloud deployment.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="37436-125">Design de assinaturas</span><span class="sxs-lookup"><span data-stu-id="37436-125">Subscriptions design</span></span>](subscriptions/overview.md)
