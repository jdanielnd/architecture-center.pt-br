---
title: 'CAF: Gerenciamento de acesso aos recursos no Azure'
description: 'Explicação dos constructos de gerenciamento de acesso a recursos no Azure: Azure Resource Manager, assinaturas, grupos de recursos e recursos.'
author: petertaylor9999
ms.date: 2/11/2019
ms.openlocfilehash: f23540a03c82fbc46872645ac0fd82d574db353a
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898112"
---
# <a name="resource-access-management-in-azure"></a><span data-ttu-id="e179a-103">Gerenciamento de acesso aos recursos no Azure</span><span class="sxs-lookup"><span data-stu-id="e179a-103">Resource access management in Azure</span></span>

<span data-ttu-id="e179a-104">Em [O que é governança de recursos de nuvem?](what-is-governance.md), você aprendeu que governança é o processo contínuo de gerenciar, monitorar e auditar o uso de recursos do Azure para atender às metas e requisitos da sua organização.</span><span class="sxs-lookup"><span data-stu-id="e179a-104">In [what is resource governance?](what-is-governance.md), you learned that governance refers to the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the goals and requirements of your organization.</span></span> <span data-ttu-id="e179a-105">Antes de prosseguir para aprender a criar um modelo de governança, é importante entender os controles de gerenciamento de acesso a recursos no Azure.</span><span class="sxs-lookup"><span data-stu-id="e179a-105">Before you move on to learn how to design a governance model, it's important to understand the resource access management controls in Azure.</span></span> <span data-ttu-id="e179a-106">A configuração desses controles de gerenciamento de acesso a recursos constitui a base do seu modelo de governança.</span><span class="sxs-lookup"><span data-stu-id="e179a-106">The configuration of these resource access management controls forms the basis of your governance model.</span></span>

<span data-ttu-id="e179a-107">Vamos começar examinando mais detalhadamente como os recursos são implantados no Azure.</span><span class="sxs-lookup"><span data-stu-id="e179a-107">Let's begin by taking a closer look at how resources are deployed in Azure.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-azure-resource"></a><span data-ttu-id="e179a-108">O que é um recurso do Azure?</span><span class="sxs-lookup"><span data-stu-id="e179a-108">What is an Azure resource?</span></span>

<span data-ttu-id="e179a-109">No Azure, o termo **recurso** refere-se a uma entidade gerenciada pelo Azure.</span><span class="sxs-lookup"><span data-stu-id="e179a-109">In Azure, the term **resource** refers to an entity managed by Azure.</span></span> <span data-ttu-id="e179a-110">Por exemplo, máquinas virtuais, redes virtuais e contas de armazenamento são todas consideradas recursos do Azure.</span><span class="sxs-lookup"><span data-stu-id="e179a-110">For example, virtual machines, virtual networks, and storage accounts are all referred to as Azure resources.</span></span>

<span data-ttu-id="e179a-111">![](../_images/governance-1-9.png)
*Figura 1. Um recurso.*</span><span class="sxs-lookup"><span data-stu-id="e179a-111">![](../_images/governance-1-9.png)
*Figure 1. A resource.*</span></span>

## <a name="what-is-an-azure-resource-group"></a><span data-ttu-id="e179a-112">O que é um grupo de recursos do Azure?</span><span class="sxs-lookup"><span data-stu-id="e179a-112">What is an Azure resource group?</span></span>

<span data-ttu-id="e179a-113">Cada recurso no Azure deve pertencer a um [grupo de recurso](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span><span class="sxs-lookup"><span data-stu-id="e179a-113">Each resource in Azure must belong to a [resource group](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span></span> <span data-ttu-id="e179a-114">Um grupo de recursos é simplesmente um constructo lógico que agrupa vários recursos para que eles possam ser gerenciados como uma única entidade.</span><span class="sxs-lookup"><span data-stu-id="e179a-114">A resource group is simply a logical construct that groups multiple resources together so they can be managed as a single entity.</span></span> <span data-ttu-id="e179a-115">Por exemplo, os recursos que compartilham um ciclo de vida semelhante, como os recursos para um [aplicativo de n camadas](/azure/architecture/guide/architecture-styles/n-tier) podem ser criados ou excluídos como um grupo.</span><span class="sxs-lookup"><span data-stu-id="e179a-115">For example, resources that share a similar lifecycle, such as the resources for an [n-tier application](/azure/architecture/guide/architecture-styles/n-tier) may be created or deleted as a group.</span></span>

<span data-ttu-id="e179a-116">![](../_images/governance-1-10.png)
*Figura 2. Um grupo de recursos contém um recurso.*</span><span class="sxs-lookup"><span data-stu-id="e179a-116">![](../_images/governance-1-10.png)
*Figure 2. A resource group contains a resource.*</span></span>

<span data-ttu-id="e179a-117">Os grupos de recursos e os recursos que eles contêm são associados a uma **assinatura** do Azure.</span><span class="sxs-lookup"><span data-stu-id="e179a-117">Resource groups and the resources they contain are associated with an Azure **subscription**.</span></span>

## <a name="what-is-an-azure-subscription"></a><span data-ttu-id="e179a-118">O que é uma assinatura do Azure?</span><span class="sxs-lookup"><span data-stu-id="e179a-118">What is an Azure subscription?</span></span>

<span data-ttu-id="e179a-119">Uma assinatura do Azure é semelhante a um grupo de recursos em que ele é um constructo lógico que agrupa os grupos de recursos e seus recursos.</span><span class="sxs-lookup"><span data-stu-id="e179a-119">An Azure subscription is similar to a resource group in that it's a logical construct that groups together resource groups and their resources.</span></span> <span data-ttu-id="e179a-120">No entanto, uma assinatura do Azure também é associada aos controles usados pelo Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="e179a-120">However, an Azure subscription is also associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="e179a-121">O que isso significa?</span><span class="sxs-lookup"><span data-stu-id="e179a-121">What does this mean?</span></span> <span data-ttu-id="e179a-122">Vamos examinar mais detalhadamente o Gerenciador de Recursos para saber mais sobre a relação entre ele e uma assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="e179a-122">Let's take a closer look at Resource Manager to learn about the relationship between it and an Azure subscription.</span></span>

<span data-ttu-id="e179a-123">![](../_images/governance-1-11.png)
*Figure 3. Uma assinatura do Azure.*</span><span class="sxs-lookup"><span data-stu-id="e179a-123">![](../_images/governance-1-11.png)
*Figure 3. An Azure subscription.*</span></span>

## <a name="what-is-azure-resource-manager"></a><span data-ttu-id="e179a-124">O que é o Azure Resource Manager?</span><span class="sxs-lookup"><span data-stu-id="e179a-124">What is Azure Resource Manager?</span></span>

<span data-ttu-id="e179a-125">Em [Como funciona o Azure?](what-is-azure.md) você aprendeu que o Azure inclui um "front-end" com muitos serviços que orquestram todas as funções do Azure.</span><span class="sxs-lookup"><span data-stu-id="e179a-125">In [how does Azure work?](what-is-azure.md) you learned that Azure includes a "front end" with many services that orchestrate all the functions of Azure.</span></span> <span data-ttu-id="e179a-126">Um desses serviços é o [Gerenciador de Recursos](/azure/azure-resource-manager/), que hospeda a API RESTful usada pelos clientes para gerenciar recursos.</span><span class="sxs-lookup"><span data-stu-id="e179a-126">One of these services is [Resource Manager](/azure/azure-resource-manager/), and this service hosts the RESTful API used by clients to manage resources.</span></span>

<span data-ttu-id="e179a-127">![](../_images/governance-1-12.png)
*Figura 4. Azure Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="e179a-127">![](../_images/governance-1-12.png)
*Figure 4. Azure resource manager.*</span></span>

<span data-ttu-id="e179a-128">A figura a seguir mostra três clientes: [PowerShell](/powershell/azure/overview), [o portal do Azure](https://portal.azure.com) e a [interface de linha de comando (CLI) do Azure](/cli/azure):</span><span class="sxs-lookup"><span data-stu-id="e179a-128">The following figure shows three clients: [PowerShell](/powershell/azure/overview), the [Azure portal](https://portal.azure.com), and the [Azure command line interface (CLI)](/cli/azure):</span></span>

<span data-ttu-id="e179a-129">![](../_images/governance-1-13.png)
*Figura 5. Os clientes do Azure conectam-se à API RESTful do Gerenciador de Recursos.*</span><span class="sxs-lookup"><span data-stu-id="e179a-129">![](../_images/governance-1-13.png)
*Figure 5. Azure clients connect to the Resource Manager RESTful API.*</span></span>

<span data-ttu-id="e179a-130">Embora esses clientes se conectem ao Gerenciador de Recursos usando a API RESTful, o Gerenciador de Recursos não inclui a funcionalidade para gerenciar diretamente os recursos.</span><span class="sxs-lookup"><span data-stu-id="e179a-130">While these clients connect to Resource Manager using the RESTful API, Resource Manager does not include functionality to manage resources directly.</span></span> <span data-ttu-id="e179a-131">Em vez disso, a maioria dos tipos de recursos no Azure têm seus próprios [ **provedores de recursos**](/azure/azure-resource-manager/resource-group-overview#terminology).</span><span class="sxs-lookup"><span data-stu-id="e179a-131">Rather, most resource types in Azure have their own [**resource provider**](/azure/azure-resource-manager/resource-group-overview#terminology).</span></span>

<span data-ttu-id="e179a-132">\*
*Figura 6. Provedores de recursos do Azure.*</span><span class="sxs-lookup"><span data-stu-id="e179a-132">![](../_images/governance-1-14.png)
*Figure 6. Azure resource providers.*</span></span>

<span data-ttu-id="e179a-133">Quando um cliente faz uma solicitação para gerenciar um recurso específico, o Gerenciador de Recursos se conecta ao provedor de recursos para esse tipo de recurso concluir a solicitação.</span><span class="sxs-lookup"><span data-stu-id="e179a-133">When a client makes a request to manage a specific resource, Resource Manager connects to the resource provider for that resource type to complete the request.</span></span> <span data-ttu-id="e179a-134">Por exemplo, se um cliente fizer uma solicitação para gerenciar um recurso de máquina virtual, o Gerenciador de Recursos se conecta ao provedor de recursos **Microsoft.compute**.</span><span class="sxs-lookup"><span data-stu-id="e179a-134">For example, if a client makes a request to manage a virtual machine resource, Resource Manager connects to the **Microsoft.Compute** resource provider.</span></span>

<span data-ttu-id="e179a-135">![](../_images/governance-1-15.png)
*Figura 7. O Gerenciador de Recursos se conecta ao provedor de recursos **Microsoft.Compute** para gerenciar o recurso especificado na solicitação do cliente.*</span><span class="sxs-lookup"><span data-stu-id="e179a-135">![](../_images/governance-1-15.png)
*Figure 7. Resource Manager connects to the **Microsoft.Compute** resource provider to manage the resource specified in the client request.*</span></span>

<span data-ttu-id="e179a-136">O Gerenciador de Recursos exige que o cliente especifique um identificador para a assinatura e o grupo de recursos para gerenciar o recurso de máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="e179a-136">Resource Manager requires the client to specify an identifier for both the subscription and the resource group in order to manage the virtual machine resource.</span></span>

<span data-ttu-id="e179a-137">Agora que você tem uma compreensão de como funciona o Gerenciador de Recursos, vamos voltar para nossa discussão de como uma assinatura do Azure está associada aos controles usados pelo Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="e179a-137">Now that you have an understanding of how Resource Manager works, let's return to our discussion of how an Azure subscription is associated with the controls used by Azure resource manager.</span></span> <span data-ttu-id="e179a-138">Antes que qualquer solicitação de gerenciamento de recursos possa ser executada pelo Azure Resource Manager, um conjunto de controles será verificado.</span><span class="sxs-lookup"><span data-stu-id="e179a-138">Before any resource management request can be executed by Resource Manager, a set of controls are checked.</span></span>

<span data-ttu-id="e179a-139">O primeiro controle é que a solicitação deve ser feita por um usuário validado e o Azure Resource Manager tem uma relação confiável com [Azure Active Directory](/azure/active-directory/) (Azure AD) para fornecer a funcionalidade de identidade do usuário.</span><span class="sxs-lookup"><span data-stu-id="e179a-139">The first control is that a request must be made by a validated user, and Resource Manager has a trusted relationship with [Azure Active Directory](/azure/active-directory/) (Azure AD) to provide user identity functionality.</span></span>

<span data-ttu-id="e179a-140">![](../_images/governance-1-16.png)
*Figura 8. Azure Active Directory.*</span><span class="sxs-lookup"><span data-stu-id="e179a-140">![](../_images/governance-1-16.png)
*Figure 8. Azure Active Directory.*</span></span>

<span data-ttu-id="e179a-141">No Azure AD, os usuários são segmentados em **locatários**.</span><span class="sxs-lookup"><span data-stu-id="e179a-141">In Azure AD, users are segmented into **tenants**.</span></span> <span data-ttu-id="e179a-142">Um locatário é um constructo lógico que representa uma instância segura e dedicada do Azure AD geralmente associada a uma organização.</span><span class="sxs-lookup"><span data-stu-id="e179a-142">A tenant is a logical construct that represents a secure, dedicated instance of Azure AD typically associated with an organization.</span></span> <span data-ttu-id="e179a-143">Cada assinatura está associada a um locatário do Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="e179a-143">Each subscription is associated with an Azure AD tenant.</span></span>

<span data-ttu-id="e179a-144">![](../_images/governance-1-17.png)
*Figura 9. Um locatário do Microsoft Azure Active Directory associado a uma assinatura.*</span><span class="sxs-lookup"><span data-stu-id="e179a-144">![](../_images/governance-1-17.png)
*Figure 9. An Azure AD tenant associated with a subscription.*</span></span>

<span data-ttu-id="e179a-145">Cada solicitação de cliente para gerenciar um recurso em uma assinatura específica exige que o usuário tenha uma conta associada ao locatário do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="e179a-145">Each client request to manage a resource in a particular subscription requires that the user has an account in the associated Azure AD tenant.</span></span>

<span data-ttu-id="e179a-146">O próximo controle é uma verificação de que o usuário tem permissão suficiente para fazer a solicitação.</span><span class="sxs-lookup"><span data-stu-id="e179a-146">The next control is a check that the user has sufficient permission to make the request.</span></span> <span data-ttu-id="e179a-147">As permissões são atribuídas aos usuários que usam o [controle de acesso baseado em função (RBAC)](/azure/role-based-access-control/).</span><span class="sxs-lookup"><span data-stu-id="e179a-147">Permissions are assigned to users using [role-based access control (RBAC)](/azure/role-based-access-control/).</span></span>

<span data-ttu-id="e179a-148">![](../_images/governance-1-18.png)
*Figura 10. Cada usuário no locatário é atribuído a uma ou mais funções RBAC.*</span><span class="sxs-lookup"><span data-stu-id="e179a-148">![](../_images/governance-1-18.png)
*Figure 10. Each user in the tenant is assigned one or more RBAC roles.*</span></span>

<span data-ttu-id="e179a-149">Uma função RBAC especifica um conjunto de permissões que um usuário pode executar em um recurso específico.</span><span class="sxs-lookup"><span data-stu-id="e179a-149">An RBAC role specifies a set of permissions a user may take on a specific resource.</span></span> <span data-ttu-id="e179a-150">Quando a função é atribuída ao usuário, essas permissões são aplicadas.</span><span class="sxs-lookup"><span data-stu-id="e179a-150">When the role is assigned to the user, those permissions are applied.</span></span> <span data-ttu-id="e179a-151">Por exemplo, a [função de **proprietário** interna](/azure/role-based-access-control/built-in-roles#owner) permite que um usuário realize qualquer ação em um recurso.</span><span class="sxs-lookup"><span data-stu-id="e179a-151">For example, the [built-in **owner** role](/azure/role-based-access-control/built-in-roles#owner) allows a user to perform any action on a resource.</span></span>

<span data-ttu-id="e179a-152">O próximo controle é uma verificação de que a solicitação será permitida sob as configurações especificadas da [política de recursos do Azure](/azure/governance/policy/).</span><span class="sxs-lookup"><span data-stu-id="e179a-152">The next control is a check that the request is allowed under the settings specified for [Azure resource policy](/azure/governance/policy/).</span></span> <span data-ttu-id="e179a-153">As políticas de recursos do Azure especificam as operações permitidas para um recurso específico.</span><span class="sxs-lookup"><span data-stu-id="e179a-153">Azure resource policies specify the operations allowed for a specific resource.</span></span> <span data-ttu-id="e179a-154">Por exemplo, uma política de recursos do Azure pode especificar que os usuários só têm permissão para implantar um tipo específico de máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="e179a-154">For example, an Azure resource policy can specify that users are only allowed to deploy a specific type of virtual machine.</span></span>

<span data-ttu-id="e179a-155">![](../_images/governance-1-19.png)
*Figura 11. Políticas de recursos do Azure.*</span><span class="sxs-lookup"><span data-stu-id="e179a-155">![](../_images/governance-1-19.png)
*Figure 11. Azure resource policy.*</span></span>

<span data-ttu-id="e179a-156">O próximo controle é uma verificação de que a solicitação não excede um [limite de assinatura do Azure](/azure/azure-subscription-service-limits).</span><span class="sxs-lookup"><span data-stu-id="e179a-156">The next control is a check that the request does not exceed an [Azure subscription limit](/azure/azure-subscription-service-limits).</span></span> <span data-ttu-id="e179a-157">Por exemplo, cada assinatura tem um limite de 980 grupos de recursos por assinatura.</span><span class="sxs-lookup"><span data-stu-id="e179a-157">For example, each subscription has a limit of 980 resource groups per subscription.</span></span> <span data-ttu-id="e179a-158">Se uma solicitação é recebida para implantar um grupo de recursos adicionais depois que o limite for atingido, ela será negada.</span><span class="sxs-lookup"><span data-stu-id="e179a-158">If a request is received to deploy an additional resource group once the limit has been reached, it is denied.</span></span>

<span data-ttu-id="e179a-159">![](../_images/governance-1-20.png)
*Figura 12. Limites de recursos do Azure.*</span><span class="sxs-lookup"><span data-stu-id="e179a-159">![](../_images/governance-1-20.png)
*Figure 12. Azure resource limits.*</span></span>

<span data-ttu-id="e179a-160">O controle final é uma verificação de que a solicitação está dentro do compromisso financeiro associado à assinatura.</span><span class="sxs-lookup"><span data-stu-id="e179a-160">The final control is a check that the request is within the financial commitment associated with the subscription.</span></span> <span data-ttu-id="e179a-161">Por exemplo, se a solicitação é para implantar uma máquina virtual, o Gerenciador de Recursos verifica se a assinatura tem informações suficientes de pagamento.</span><span class="sxs-lookup"><span data-stu-id="e179a-161">For example, if the request is to deploy a virtual machine, Resource Manager verifies that the subscription has sufficient payment information.</span></span>

<span data-ttu-id="e179a-162">![](../_images/governance-1-21.png)
*Figura 13. Um compromisso financeiro é associado a uma assinatura.*</span><span class="sxs-lookup"><span data-stu-id="e179a-162">![](../_images/governance-1-21.png)
*Figure 13. A financial commitment is associated with a subscription.*</span></span>

## <a name="summary"></a><span data-ttu-id="e179a-163">Resumo</span><span class="sxs-lookup"><span data-stu-id="e179a-163">Summary</span></span>

<span data-ttu-id="e179a-164">Neste artigo, você aprendeu sobre como o acesso a recursos é gerenciado no Azure usando o Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="e179a-164">In this article, you learned about how resource access is managed in Azure using Azure Resource Manager.</span></span>

## <a name="next-steps"></a><span data-ttu-id="e179a-165">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="e179a-165">Next steps</span></span>

<span data-ttu-id="e179a-166">Agora que você sabe como o acesso a recursos é gerenciado no Azure, saiba mais sobre como criar um modelo de governança.</span><span class="sxs-lookup"><span data-stu-id="e179a-166">Now that you understand how resource access is managed in Azure, move on to learn how to design a governance model.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="e179a-167">Governança de nuvem</span><span class="sxs-lookup"><span data-stu-id="e179a-167">Cloud governance</span></span>](../governance/overview.md)
