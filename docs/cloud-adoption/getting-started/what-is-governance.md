---
title: 'CAF: O que é a governança dos recursos da nuvem?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.date: 02/11/2019
description: Governança de recursos de nuvem de explicação no Azure
author: petertaylor9999
ms.openlocfilehash: 0989a5aad8a6290cce07fd71771c690bbd615e0d
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/08/2019
ms.locfileid: "59068847"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-cloud-resource-governance"></a><span data-ttu-id="28490-103">O que é a governança dos recursos da nuvem?</span><span class="sxs-lookup"><span data-stu-id="28490-103">What is cloud resource governance?</span></span>

<span data-ttu-id="28490-104">Na [como funciona o Azure?](what-is-azure.md), você aprendeu que o Azure é uma coleção de servidores e hardware que executa o software e hardware virtualizado em nome dos usuários de rede.</span><span class="sxs-lookup"><span data-stu-id="28490-104">In [How does Azure work?](what-is-azure.md), you learned that Azure is a collection of servers and networking hardware running virtualized hardware and software on behalf of users.</span></span> <span data-ttu-id="28490-105">O Azure permite o desenvolvimento de aplicativos da sua organização e aos departamentos de TI seja ágil, tornando mais fácil criar, ler, atualizar e excluir recursos conforme necessário.</span><span class="sxs-lookup"><span data-stu-id="28490-105">Azure enables your organization's application development and IT departments to be agile by making it easy to create, read, update, and delete resources as needed.</span></span>

<span data-ttu-id="28490-106">No entanto, enquanto o acesso irrestrito aos recursos pode tornar os desenvolvedores muito ágil, ele também pode levar a custos inesperados.</span><span class="sxs-lookup"><span data-stu-id="28490-106">However, while unrestricted access to resources can make developers very agile, it can also lead to unexpected costs.</span></span> <span data-ttu-id="28490-107">Por exemplo, uma equipe de desenvolvimento pode ser aprovada para implantar um conjunto de recursos para teste, mas esquecer de excluí-las quando o teste for concluído.</span><span class="sxs-lookup"><span data-stu-id="28490-107">For example, a development team might be approved to deploy a set of resources for testing but forget to delete them when testing is complete.</span></span> <span data-ttu-id="28490-108">Esses recursos continuarão a acumular custos, mesmo que eles não são mais necessários ou aprovado.</span><span class="sxs-lookup"><span data-stu-id="28490-108">These resources will continue to accrue costs even though they are no longer approved or necessary.</span></span>

<span data-ttu-id="28490-109">A solução é o controle de acesso de recursos.</span><span class="sxs-lookup"><span data-stu-id="28490-109">The solution is resource access governance.</span></span> <span data-ttu-id="28490-110">**Governança** é o processo contínuo de gerenciamento, monitoramento e auditoria do uso de recursos do Azure para atender aos requisitos da sua organização.</span><span class="sxs-lookup"><span data-stu-id="28490-110">**Governance** is the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the requirements of your organization.</span></span>

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94]

<!-- markdownlint-enable MD034 -->

<span data-ttu-id="28490-111">Esses requisitos são exclusivos de cada organização, portanto, uma abordagem única para a governança não é útil.</span><span class="sxs-lookup"><span data-stu-id="28490-111">These requirements are unique to each organization, so a one-size-fits-all approach to governance isn't helpful.</span></span> <span data-ttu-id="28490-112">Em vez disso, cabe a cada organização para criar seu modelo de governança usando duas ferramentas de governança primária do Azure: **controle de acesso baseado em função (RBAC)** e **política de recurso**.</span><span class="sxs-lookup"><span data-stu-id="28490-112">Instead, it's up to each organization to design their governance model using Azure's two primary governance tools: **role-based access control (RBAC)** and **resource policy**.</span></span>

<span data-ttu-id="28490-113">Define as funções RBAC e as funções definem os recursos de cada usuário atribuídos a essa função.</span><span class="sxs-lookup"><span data-stu-id="28490-113">RBAC defines roles, and roles define the capabilities of each user assigned that role.</span></span> <span data-ttu-id="28490-114">Por exemplo, o **proprietário** função permite que todos os recursos (criar, ler, atualizar e excluir) para um recurso, enquanto o **leitor** função permite que apenas a capacidade de leitura.</span><span class="sxs-lookup"><span data-stu-id="28490-114">For example, the **owner** role allows all capabilites (create, read, update, and delete) for a resource, while the  **reader** role allows only the read capability.</span></span> <span data-ttu-id="28490-115">As funções podem ser definidas com um amplo escopo que se aplica a muitos tipos de recurso ou um escopo estreito que se aplica a alguns.</span><span class="sxs-lookup"><span data-stu-id="28490-115">Roles can be defined with a broad scope that applies to many resource types, or a narrow scope that applies to a few.</span></span>

<span data-ttu-id="28490-116">As políticas de recursos definem regras para a criação de recursos.</span><span class="sxs-lookup"><span data-stu-id="28490-116">Resource policies define rules for resource creation.</span></span> <span data-ttu-id="28490-117">Por exemplo, uma política de recurso pode limitar a SKU de uma máquina virtual com um determinado tamanho previamente aprovada.</span><span class="sxs-lookup"><span data-stu-id="28490-117">For example, a resource policy can limit the SKU of a virtual machine to a particular pre-approved size.</span></span> <span data-ttu-id="28490-118">Outra política de recursos pode impor a aplicação de uma marca para um centro de custo atribuído quando a solicitação é feita para criar o recurso.</span><span class="sxs-lookup"><span data-stu-id="28490-118">Another resource policy could enforce the application of a tag for an assigned cost center when the request is made to create the resource.</span></span>

<span data-ttu-id="28490-119">Ao configurar essas ferramentas, é importante equilibrar a governança e a agilidade organizacional.</span><span class="sxs-lookup"><span data-stu-id="28490-119">When configuring these tools, it is important to balance governance and organizational agility.</span></span> <span data-ttu-id="28490-120">A configuração mais restritiva sua diretiva de governança, menos agile seus desenvolvedores e profissionais de TI será.</span><span class="sxs-lookup"><span data-stu-id="28490-120">The more restrictive your governance policy, the less agile your developers and IT workers will be.</span></span> <span data-ttu-id="28490-121">Uma política de governança restritiva pode exigir mais etapas manuais, como exigir que um desenvolvedor preencher um formulário ou envie um email para um membro da equipe de governança para criar manualmente um recurso.</span><span class="sxs-lookup"><span data-stu-id="28490-121">A restrictive governance policy may require more manual steps like requiring a developer to fill out a form or send an email to a member of the governance team to manually create a resource.</span></span> <span data-ttu-id="28490-122">A equipe de governança tem capacidade finita e pode se tornar um gargalo, resultando em equipes de desenvolvimento aguardando unproductively seus recursos para ser criados ou desnecessários recursos acumular custos antes de serem excluídos.</span><span class="sxs-lookup"><span data-stu-id="28490-122">The governance team has finite capacity and may become a bottleneck, resulting in development teams waiting unproductively for their resources to be created or unneeded resources accruing costs before they are deleted.</span></span>

## <a name="next-steps"></a><span data-ttu-id="28490-123">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="28490-123">Next steps</span></span>

<span data-ttu-id="28490-124">Agora que você entende o conceito de governança de recursos de nuvem, saiba mais sobre como o acesso aos recursos é gerenciado no Azure.</span><span class="sxs-lookup"><span data-stu-id="28490-124">Now that you understand the concept of cloud resource governance, learn more about how resource access is managed in Azure.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="28490-125">Saiba mais sobre o acesso aos recursos do Azure</span><span class="sxs-lookup"><span data-stu-id="28490-125">Learn about resource access in Azure</span></span>](azure-resource-access.md)
