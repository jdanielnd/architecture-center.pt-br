---
title: 'CAF: Gerenciamento de acesso de recurso no Azure'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: 'Explicação dos constructos de gerenciamento de acesso de recurso no Azure: Azure Resource Manager, assinaturas, grupos de recursos e recursos'
author: petertaylor9999
ms.openlocfilehash: b98cdc94d6d3a37c1e65da1d4de35d5d9520d6eb
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900408"
---
# <a name="resource-access-management-in-azure"></a><span data-ttu-id="fd664-103">Gerenciamento de acesso a recursos no Azure</span><span class="sxs-lookup"><span data-stu-id="fd664-103">Resource access management in Azure</span></span>

<span data-ttu-id="fd664-104">A [Governança de Nuvem](../overview.md) descreve as cinco disciplinas da Governança de Nuvem, dentre as quais inclui o Gerenciamento de Recursos.</span><span class="sxs-lookup"><span data-stu-id="fd664-104">[Cloud Governance](../overview.md) outlines the five disciplines of Cloud Governance, which includes Resource Management.</span></span>  <span data-ttu-id="fd664-105">[O que é governança de acesso de recurso](overview.md) explica como o gerenciamento de acesso de recurso se enquadra na disciplina de gerenciamento de recursos.</span><span class="sxs-lookup"><span data-stu-id="fd664-105">[What is resource access governance](overview.md) furthers explains how resource access management fits into the resource management discipline.</span></span> <span data-ttu-id="fd664-106">Antes de prosseguir para aprender a criar um modelo de governança, é importante entender os controles de gerenciamento de acesso a recursos no Azure.</span><span class="sxs-lookup"><span data-stu-id="fd664-106">Before you move on to learn how to design a governance model, it's important to understand the resource access management controls in Azure.</span></span> <span data-ttu-id="fd664-107">A configuração desses controles de gerenciamento de acesso de recurso constitui a base do seu modelo de governança.</span><span class="sxs-lookup"><span data-stu-id="fd664-107">The configuration of these resource access management controls forms the basis of your governance model.</span></span>

<span data-ttu-id="fd664-108">Comece examinando mais detalhadamente como os recursos são implantados no Azure.</span><span class="sxs-lookup"><span data-stu-id="fd664-108">Begin by taking a closer look at how resources are deployed in Azure.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-azure-resource"></a><span data-ttu-id="fd664-109">O que é um recurso do Azure?</span><span class="sxs-lookup"><span data-stu-id="fd664-109">What is an Azure resource?</span></span>

<span data-ttu-id="fd664-110">No Azure, o termo **recurso** refere-se a uma entidade gerenciada pelo Azure.</span><span class="sxs-lookup"><span data-stu-id="fd664-110">In Azure, the term **resource** refers to an entity managed by Azure.</span></span> <span data-ttu-id="fd664-111">Por exemplo, máquinas virtuais, redes virtuais e contas de armazenamento são todas consideradas recursos do Azure.</span><span class="sxs-lookup"><span data-stu-id="fd664-111">For example, virtual machines, virtual networks, and storage accounts are all referred to as Azure resources.</span></span>

<span data-ttu-id="fd664-112">![Diagrama de um recurso](../../_images/governance-1-9.png)
*Figura 1. Um recurso.*</span><span class="sxs-lookup"><span data-stu-id="fd664-112">![Diagram of a resource](../../_images/governance-1-9.png)
*Figure 1. A resource.*</span></span>

## <a name="what-is-an-azure-resource-group"></a><span data-ttu-id="fd664-113">O que é um grupo de recursos do Azure?</span><span class="sxs-lookup"><span data-stu-id="fd664-113">What is an Azure resource group?</span></span>

<span data-ttu-id="fd664-114">Cada recurso no Azure deve pertencer a um [grupo de recurso](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span><span class="sxs-lookup"><span data-stu-id="fd664-114">Each resource in Azure must belong to a [resource group](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span></span> <span data-ttu-id="fd664-115">Um grupo de recursos é simplesmente um constructo lógico que agrupa vários recursos para que eles possam ser gerenciados como uma única entidade.</span><span class="sxs-lookup"><span data-stu-id="fd664-115">A resource group is simply a logical construct that groups multiple resources together so they can be managed as a single entity.</span></span> <span data-ttu-id="fd664-116">Por exemplo, os recursos que compartilham um ciclo de vida semelhante, como os recursos para um [aplicativo de n camadas](/azure/architecture/guide/architecture-styles/n-tier) podem ser criados ou excluídos como um grupo.</span><span class="sxs-lookup"><span data-stu-id="fd664-116">For example, resources that share a similar lifecycle, such as the resources for an [n-tier application](/azure/architecture/guide/architecture-styles/n-tier) may be created or deleted as a group.</span></span>

<span data-ttu-id="fd664-117">![Diagrama de um grupo de recursos contendo um recurso](../../_images/governance-1-10.png)
*Figura 2. Um grupo de recursos contém um recurso.*</span><span class="sxs-lookup"><span data-stu-id="fd664-117">![Diagram of a resource group containing a resource](../../_images/governance-1-10.png)
*Figure 2. A resource group contains a resource.*</span></span>

<span data-ttu-id="fd664-118">Os grupos de recursos e os recursos que eles contêm são associados a uma **assinatura** do Azure.</span><span class="sxs-lookup"><span data-stu-id="fd664-118">Resource groups and the resources they contain are associated with an Azure **subscription**.</span></span>

## <a name="what-is-an-azure-subscription"></a><span data-ttu-id="fd664-119">O que é uma assinatura do Azure?</span><span class="sxs-lookup"><span data-stu-id="fd664-119">What is an Azure subscription?</span></span>

<span data-ttu-id="fd664-120">Uma assinatura do Azure é semelhante a um grupo de recursos em que ele é um constructo lógico que agrupa os grupos de recursos e seus recursos.</span><span class="sxs-lookup"><span data-stu-id="fd664-120">An Azure subscription is similar to a resource group in that it's a logical construct that groups together resource groups and their resources.</span></span> <span data-ttu-id="fd664-121">No entanto, uma assinatura do Azure também é associada aos controles usados pelo Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="fd664-121">However, an Azure subscription is also associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="fd664-122">O que isso significa?</span><span class="sxs-lookup"><span data-stu-id="fd664-122">What does this mean?</span></span> <span data-ttu-id="fd664-123">Examine detalhadamente o Azure Resource Manager para saber mais sobre o relacionamento entre ele e uma assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="fd664-123">Take a closer look at Azure Resource Manager to learn about the relationship between it and an Azure subscription.</span></span>

<span data-ttu-id="fd664-124">![Diagrama de uma assinatura do Azure](../../_images/governance-1-11.png)
*Figura 3. Uma assinatura do Azure.*</span><span class="sxs-lookup"><span data-stu-id="fd664-124">![Diagram of an Azure subscription](../../_images/governance-1-11.png)
*Figure 3. An Azure subscription.*</span></span>

## <a name="what-is-azure-resource-manager"></a><span data-ttu-id="fd664-125">O que é o Azure Resource Manager?</span><span class="sxs-lookup"><span data-stu-id="fd664-125">What is Azure Resource Manager?</span></span>

<span data-ttu-id="fd664-126">Em [Como funciona o Azure?](../../getting-started/what-is-azure.md) você aprendeu que o Azure inclui um "front-end" com muitos serviços que orquestram todas as funções do Azure.</span><span class="sxs-lookup"><span data-stu-id="fd664-126">In [how does Azure work?](../../getting-started/what-is-azure.md) you learned that Azure includes a "front end" with many services that orchestrate all the functions of Azure.</span></span> <span data-ttu-id="fd664-127">Um desses serviços é o [Azure Resource Manager](/azure/azure-resource-manager/), que hospeda a API RESTful usada pelos clientes para gerenciar recursos.</span><span class="sxs-lookup"><span data-stu-id="fd664-127">One of these services is [Azure Resource Manager](/azure/azure-resource-manager/), and this service hosts the RESTful API used by clients to manage resources.</span></span>

<span data-ttu-id="fd664-128">![Diagrama do Azure Resource Manager](../../_images/governance-1-12.png)
*Figura 4. Azure Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="fd664-128">![Diagram of Azure Resource Manager](../../_images/governance-1-12.png)
*Figure 4. Azure Resource Manager.*</span></span>

<span data-ttu-id="fd664-129">A figura a seguir mostra três clientes: O [PowerShell](/powershell/azure/overview), o [portal do Azure](https://portal.azure.com) e a [CLI (interface de linha de comando) do Azure](/cli/azure):</span><span class="sxs-lookup"><span data-stu-id="fd664-129">The following figure shows three clients: [PowerShell](/powershell/azure/overview), the [Azure portal](https://portal.azure.com), and the [Azure command-line interface (CLI)](/cli/azure):</span></span>

<span data-ttu-id="fd664-130">![Diagrama de clientes do Azure conectando a API do Azure Resource Manager](../../_images/governance-1-13.png)
*Figura 5. Clientes do Azure conectam a API RESTful do Azure Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="fd664-130">![Diagram of Azure clients connecting to the Azure Resource Manager API](../../_images/governance-1-13.png)
*Figure 5. Azure clients connect to the Azure Resource Manager RESTful API.*</span></span>

<span data-ttu-id="fd664-131">Embora esses clientes conectem o Azure Resource Manager usando a API RESTful, o Azure Resource Manager não inclui a funcionalidade para gerenciar recursos diretamente.</span><span class="sxs-lookup"><span data-stu-id="fd664-131">While these clients connect to Azure Resource Manager using the RESTful API, Azure Resource Manager does not include functionality to manage resources directly.</span></span> <span data-ttu-id="fd664-132">Em vez disso, a maioria dos tipos de recursos no Azure têm seus próprios [ **provedores de recursos**](/azure/azure-resource-manager/resource-group-overview#terminology).</span><span class="sxs-lookup"><span data-stu-id="fd664-132">Rather, most resource types in Azure have their own [**resource provider**](/azure/azure-resource-manager/resource-group-overview#terminology).</span></span>

<span data-ttu-id="fd664-133">![Provedores de recursos do Azure](../../_images/governance-1-14.png)
*Figura 6. Provedores de recursos do Azure.*</span><span class="sxs-lookup"><span data-stu-id="fd664-133">![Azure resource providers](../../_images/governance-1-14.png)
*Figure 6. Azure resource providers.*</span></span>

<span data-ttu-id="fd664-134">Quando um cliente faz uma solicitação para gerenciar um recurso específico, o Azure Resource Manager conecta o provedor de recursos daquele tipo de recurso para concluir a solicitação.</span><span class="sxs-lookup"><span data-stu-id="fd664-134">When a client makes a request to manage a specific resource, Azure Resource Manager connects to the resource provider for that resource type to complete the request.</span></span> <span data-ttu-id="fd664-135">Por exemplo, se um cliente fizer uma solicitação para gerenciar um recurso de máquina virtual, o Azure Resource Manager conectará o provedor de recursos **Microsoft.Compute**.</span><span class="sxs-lookup"><span data-stu-id="fd664-135">For example, if a client makes a request to manage a virtual machine resource, Azure Resource Manager connects to the **Microsoft.Compute** resource provider.</span></span>

<span data-ttu-id="fd664-136">![O Azure Resource Manager conectando o provedor de recursos Microsoft.Compute](../../_images/governance-1-15.png)
*Figura 7. O Azure Resource Manager conecta o provedor de recursos **Microsoft.Compute** para gerenciar o recurso especificado na solicitação do cliente.*</span><span class="sxs-lookup"><span data-stu-id="fd664-136">![Azure Resource Manager connecting to the Microsoft.Compute resource provider](../../_images/governance-1-15.png)
*Figure 7. Azure Resource Manager connects to the **Microsoft.Compute** resource provider to manage the resource specified in the client request.*</span></span>

<span data-ttu-id="fd664-137">O Azure Resource Manager requer que o cliente especifique um identificador para a assinatura e o grupo de recursos para gerenciar o recurso da máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="fd664-137">Azure Resource Manager requires the client to specify an identifier for both the subscription and the resource group in order to manage the virtual machine resource.</span></span>

<span data-ttu-id="fd664-138">Agora que você tem uma compreensão de como o Azure Resource Manager funciona, retorne à discussão sobre como uma assinatura do Azure está associada aos controles usados pelo Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="fd664-138">Now that you have an understanding of how Azure Resource Manager works, return to the discussion of how an Azure subscription is associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="fd664-139">Antes que qualquer solicitação de gerenciamento de recursos possa ser executada pelo Azure Resource Manager, um conjunto de controles será verificado.</span><span class="sxs-lookup"><span data-stu-id="fd664-139">Before any resource management request can be executed by Azure Resource Manager, a set of controls are checked.</span></span>

<span data-ttu-id="fd664-140">O primeiro controle é que uma solicitação deve ser feita por um usuário validado e o Azure Resource Manager tem uma relação de confiança com o [AD (Azure Active Directory)](/azure/active-directory/) para fornecer funcionalidade de identidade do usuário.</span><span class="sxs-lookup"><span data-stu-id="fd664-140">The first control is that a request must be made by a validated user, and Azure Resource Manager has a trusted relationship with [Azure Active Directory (Azure AD)](/azure/active-directory/) to provide user identity functionality.</span></span>

<span data-ttu-id="fd664-141">![Azure Active Directory](../../_images/governance-1-16.png)
*Figura 8. Azure Active Directory.*</span><span class="sxs-lookup"><span data-stu-id="fd664-141">![Azure Active Directory](../../_images/governance-1-16.png)
*Figure 8. Azure Active Directory.*</span></span>

<span data-ttu-id="fd664-142">No Azure AD, os usuários são segmentados em **locatários**.</span><span class="sxs-lookup"><span data-stu-id="fd664-142">In Azure AD, users are segmented into **tenants**.</span></span> <span data-ttu-id="fd664-143">Um locatário é um constructo lógico que representa uma instância segura e dedicada do Azure AD geralmente associada a uma organização.</span><span class="sxs-lookup"><span data-stu-id="fd664-143">A tenant is a logical construct that represents a secure, dedicated instance of Azure AD typically associated with an organization.</span></span> <span data-ttu-id="fd664-144">Cada assinatura está associada a um locatário do Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="fd664-144">Each subscription is associated with an Azure AD tenant.</span></span>

<span data-ttu-id="fd664-145">![Um locatário do Azure AD associado a uma assinatura](../../_images/governance-1-17.png)
*Figura 9. Um locatário do Microsoft Azure Active Directory associado a uma assinatura.*</span><span class="sxs-lookup"><span data-stu-id="fd664-145">![An Azure AD tenant associated with a subscription](../../_images/governance-1-17.png)
*Figure 9. An Azure AD tenant associated with a subscription.*</span></span>

<span data-ttu-id="fd664-146">Cada solicitação de cliente para gerenciar um recurso em uma assinatura específica exige que o usuário tenha uma conta associada ao locatário do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="fd664-146">Each client request to manage a resource in a particular subscription requires that the user has an account in the associated Azure AD tenant.</span></span>

<span data-ttu-id="fd664-147">O próximo controle é uma verificação de que o usuário tem permissão suficiente para fazer a solicitação.</span><span class="sxs-lookup"><span data-stu-id="fd664-147">The next control is a check that the user has sufficient permission to make the request.</span></span> <span data-ttu-id="fd664-148">As permissões são atribuídas aos usuários que usam o [controle de acesso baseado em função (RBAC)](/azure/role-based-access-control/).</span><span class="sxs-lookup"><span data-stu-id="fd664-148">Permissions are assigned to users using [role-based access control (RBAC)](/azure/role-based-access-control/).</span></span>

<span data-ttu-id="fd664-149">![Usuários atribuídos a funções RBAC](../../_images/governance-1-18.png)
*Figura 10. Cada usuário no locatário é atribuído a uma ou mais funções RBAC.*</span><span class="sxs-lookup"><span data-stu-id="fd664-149">![Users assigned to RBAC roles](../../_images/governance-1-18.png)
*Figure 10. Each user in the tenant is assigned one or more RBAC roles.*</span></span>

<span data-ttu-id="fd664-150">Uma função RBAC especifica um conjunto de permissões que um usuário pode executar em um recurso específico.</span><span class="sxs-lookup"><span data-stu-id="fd664-150">An RBAC role specifies a set of permissions a user may take on a specific resource.</span></span> <span data-ttu-id="fd664-151">Quando a função é atribuída ao usuário, essas permissões são aplicadas.</span><span class="sxs-lookup"><span data-stu-id="fd664-151">When the role is assigned to the user, those permissions are applied.</span></span> <span data-ttu-id="fd664-152">Por exemplo, a [função de **proprietário** interna](/azure/role-based-access-control/built-in-roles#owner) permite que um usuário realize qualquer ação em um recurso.</span><span class="sxs-lookup"><span data-stu-id="fd664-152">For example, the [built-in **owner** role](/azure/role-based-access-control/built-in-roles#owner) allows a user to perform any action on a resource.</span></span>

<span data-ttu-id="fd664-153">O próximo controle é uma verificação de que a solicitação será permitida sob as configurações especificadas da [política de recursos do Azure](/azure/governance/policy/).</span><span class="sxs-lookup"><span data-stu-id="fd664-153">The next control is a check that the request is allowed under the settings specified for [Azure resource policy](/azure/governance/policy/).</span></span> <span data-ttu-id="fd664-154">As políticas de recursos do Azure especificam as operações permitidas para um recurso específico.</span><span class="sxs-lookup"><span data-stu-id="fd664-154">Azure resource policies specify the operations allowed for a specific resource.</span></span> <span data-ttu-id="fd664-155">Por exemplo, uma política de recursos do Azure pode especificar que os usuários só têm permissão para implantar um tipo específico de máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="fd664-155">For example, an Azure resource policy can specify that users are only allowed to deploy a specific type of virtual machine.</span></span>

<span data-ttu-id="fd664-156">![Política de recursos do Azure](../../_images/governance-1-19.png)
*Figura 11. Políticas de recursos do Azure.*</span><span class="sxs-lookup"><span data-stu-id="fd664-156">![Azure resource policy](../../_images/governance-1-19.png)
*Figure 11. Azure resource policy.*</span></span>

<span data-ttu-id="fd664-157">O próximo controle é uma verificação de que a solicitação não excede um [limite de assinatura do Azure](/azure/azure-subscription-service-limits).</span><span class="sxs-lookup"><span data-stu-id="fd664-157">The next control is a check that the request does not exceed an [Azure subscription limit](/azure/azure-subscription-service-limits).</span></span> <span data-ttu-id="fd664-158">Por exemplo, cada assinatura tem um limite de 980 grupos de recursos por assinatura.</span><span class="sxs-lookup"><span data-stu-id="fd664-158">For example, each subscription has a limit of 980 resource groups per subscription.</span></span> <span data-ttu-id="fd664-159">Se uma solicitação é recebida para implantar um grupo de recursos adicionais depois que o limite for atingido, ela será negada.</span><span class="sxs-lookup"><span data-stu-id="fd664-159">If a request is received to deploy an additional resource group once the limit has been reached, it is denied.</span></span>

<span data-ttu-id="fd664-160">![Limites de recursos do Azure](../../_images/governance-1-20.png)
*Figura 12. Limites de recursos do Azure.*</span><span class="sxs-lookup"><span data-stu-id="fd664-160">![Azure resource limits](../../_images/governance-1-20.png)
*Figure 12. Azure resource limits.*</span></span>

<span data-ttu-id="fd664-161">O controle final é uma verificação de que a solicitação está dentro do compromisso financeiro associado à assinatura.</span><span class="sxs-lookup"><span data-stu-id="fd664-161">The final control is a check that the request is within the financial commitment associated with the subscription.</span></span> <span data-ttu-id="fd664-162">Por exemplo, se a solicitação for implantar uma máquina virtual, o Azure Resource Manager verificará se a assinatura tem informações de pagamento suficientes.</span><span class="sxs-lookup"><span data-stu-id="fd664-162">For example, if the request is to deploy a virtual machine, Azure Resource Manager verifies that the subscription has sufficient payment information.</span></span>

<span data-ttu-id="fd664-163">![Um compromisso financeiro associado a uma assinatura](../../_images/governance-1-21.png)
*Figura 13. Um compromisso financeiro é associado a uma assinatura.*</span><span class="sxs-lookup"><span data-stu-id="fd664-163">![A financial commitment associated with a subscription](../../_images/governance-1-21.png)
*Figure 13. A financial commitment is associated with a subscription.*</span></span>

## <a name="summary"></a><span data-ttu-id="fd664-164">Resumo</span><span class="sxs-lookup"><span data-stu-id="fd664-164">Summary</span></span>

<span data-ttu-id="fd664-165">Neste artigo, você aprendeu sobre como o acesso a recursos é gerenciado no Azure usando o Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="fd664-165">In this article, you learned about how resource access is managed in Azure using Azure Resource Manager.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fd664-166">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="fd664-166">Next steps</span></span>

<span data-ttu-id="fd664-167">Agora que você sabe como gerenciar acesso de recurso no Azure, prossiga para aprender como criar um modelo de controle [para uma carga de trabalho simples](governance-simple-workload.md) ou para [várias equipe](governance-multiple-teams.md) usando esses serviços.</span><span class="sxs-lookup"><span data-stu-id="fd664-167">Now that you understand how to manage resource access in Azure, move on to learn how to design a governance model [for a simple workload](governance-simple-workload.md) or for [multiple teams](governance-multiple-teams.md) using these services.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="fd664-168">Uma visão geral de governança</span><span class="sxs-lookup"><span data-stu-id="fd664-168">An overview of governance</span></span>](../overview.md)
