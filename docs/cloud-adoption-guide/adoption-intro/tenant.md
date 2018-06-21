---
title: 'Diretrizes: design de locatário do Azure AD'
description: Diretrizes de design de locatário do Azure como parte de uma estratégia básica de adoção da nuvem
author: telmosampaio
ms.openlocfilehash: 9ac52e9fd44bd8b9c777625002d5960f4f269be2
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/27/2018
ms.locfileid: "29563448"
---
# <a name="guidance-azure-ad-tenant-design"></a><span data-ttu-id="deb2c-103">Diretrizes: design de locatário do Azure AD</span><span class="sxs-lookup"><span data-stu-id="deb2c-103">Guidance: Azure AD tenant design</span></span>

<span data-ttu-id="deb2c-104">Um locatário do Azure AD fornece serviços de identidade digital e namespaces usados para uma ou mais [assinaturas do Azure](subscription-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="deb2c-104">An Azure AD tenant provides digital identity services and namespaces used for one or more [Azure subscriptions](subscription-explainer.md).</span></span> <span data-ttu-id="deb2c-105">Se estiver seguindo a descrição básica de adoção, você já aprendeu [a obter um locatário do Azure AD][how-to-get-aad-tenant].</span><span class="sxs-lookup"><span data-stu-id="deb2c-105">If you are following the foundational adoption outline, you have already learned [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 

## <a name="design-considerations"></a><span data-ttu-id="deb2c-106">Considerações sobre o design</span><span class="sxs-lookup"><span data-stu-id="deb2c-106">Design considerations</span></span>

- <span data-ttu-id="deb2c-107">No estágio básico de adoção, você pode começar com um único locatário do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="deb2c-107">At the foundational adoption stage, you can begin with a single Azure AD tenant.</span></span> <span data-ttu-id="deb2c-108">Caso sua organização tenha uma assinatura do Office 365 ou uma assinatura do Azure, você já tem um locatário do Azure AD que poderá usar.</span><span class="sxs-lookup"><span data-stu-id="deb2c-108">If your organization has an existing Office 365 subscription or an Azure subscription, you already have an Azure AD tenant that you can use.</span></span> <span data-ttu-id="deb2c-109">Caso não tenha nenhuma dessas, saiba mais sobre [como obter um locatário do Azure AD][how-to-get-aad-tenant].</span><span class="sxs-lookup"><span data-stu-id="deb2c-109">If you do not have either or of these, you can learn more about [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 
- <span data-ttu-id="deb2c-110">Nos estágios intermediário e avançado de adoção, você aprenderá a sincronizar ou federar diretórios locais com o Azure AD.</span><span class="sxs-lookup"><span data-stu-id="deb2c-110">In the intermediate and advanced adoption stages, you will learn how to synchronize or federate on-premises directories with Azure AD.</span></span> <span data-ttu-id="deb2c-111">Isso permitirá que você use a identidade digital local no Azure AD.</span><span class="sxs-lookup"><span data-stu-id="deb2c-111">This will allow you to use on-premises digital identity in Azure AD.</span></span> <span data-ttu-id="deb2c-112">No entanto, no estágio básico, você adicionará novos usuários que têm identidade apenas no locatário único do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="deb2c-112">However, at the foundational stage, you will be adding new users that only have identity your single Azure AD tenant.</span></span> <span data-ttu-id="deb2c-113">Você será responsável por gerenciar essas identidades.</span><span class="sxs-lookup"><span data-stu-id="deb2c-113">You will be responsible for managing those identities.</span></span> <span data-ttu-id="deb2c-114">Por exemplo, você precisará integrar novos usuários do Azure AD, transferir os usuários do Azure AD que você não deseja mais que tenham acesso aos recursos do Azure e outras alterações nas permissões de usuário.</span><span class="sxs-lookup"><span data-stu-id="deb2c-114">For example, you will have to on-board new Azure AD users, off-board Azure AD users that you no longer wish to have access to Azure resources, and other changes to user permissions.</span></span>

## <a name="next-steps"></a><span data-ttu-id="deb2c-115">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="deb2c-115">Next Steps</span></span>

* <span data-ttu-id="deb2c-116">Agora que você tem um locatário do Azure AD, saiba [como adicionar um usuário][azure-ad-add-user].</span><span class="sxs-lookup"><span data-stu-id="deb2c-116">Now that you have an Azure AD tenant, learn [how to add a user][azure-ad-add-user].</span></span> <span data-ttu-id="deb2c-117">Depois de adicionar um ou mais novos usuários ao locatário do Azure AD, a próxima etapa é aprender sobre [assinaturas do Azure](subscription-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="deb2c-117">After you have added one or more new users to your Azure AD tenant, your next step is learning about [Azure subscriptions](subscription-explainer.md).</span></span>

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
