---
title: "Explicador: o que é uma assinatura do Azure?"
description: Explica as assinaturas, contas e ofertas do Azure
author: abuck
ms.openlocfilehash: 7b88e9489b40b100eecb76602b45901566b3f37f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-subscription"></a><span data-ttu-id="44dfe-103">Explicador: o que é uma assinatura do Azure?</span><span class="sxs-lookup"><span data-stu-id="44dfe-103">Explainer: What is an Azure subscription?</span></span>

<span data-ttu-id="44dfe-104">No artigo do explicador [O que é um locatário do Azure Active Directory?](tenant-explainer.md), você aprendeu que a identidade digital de sua organização é armazenada em um locatário do Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="44dfe-104">In the [what is an Azure Active Directory tenant?](tenant-explainer.md) explainer article, you learned that digital identity for your organization is stored in an Azure Active Directory tenant.</span></span> <span data-ttu-id="44dfe-105">Você também aprendeu que o Azure confia no Azure Active Directory para autenticar solicitações do usuário para criar, ler, atualizar ou excluir um recurso.</span><span class="sxs-lookup"><span data-stu-id="44dfe-105">You also learned that Azure trusts Azure Active Directory to authenticate user requests to create, read, update, or delete a resource.</span></span> 

<span data-ttu-id="44dfe-106">Compreendemos a essência da necessidade de restringir o acesso a essas operações em um recurso – impedir que usuários não autenticados e não autorizados acessem nossos recursos.</span><span class="sxs-lookup"><span data-stu-id="44dfe-106">We fundamentally understand why it's necessary to restrict access to these operations on a resource - to prevent unauthenticated and unauthorized users from accessing our resources.</span></span> <span data-ttu-id="44dfe-107">No entanto, essas operações de recurso têm outras propriedades que uma organização deseja controlar, como o número de recursos que um usuário ou grupo de usuários tem permissão para criar e o custo de execução desses recursos.</span><span class="sxs-lookup"><span data-stu-id="44dfe-107">However, these resource operations have other properties that an organization would like to control, such as the number of resources a user or group of users is allowed to create, and, the cost to run those resources.</span></span> 

<span data-ttu-id="44dfe-108">O Azure implementa esse controle e ele é chamado de uma **assinatura**.</span><span class="sxs-lookup"><span data-stu-id="44dfe-108">Azure implements this control, and it is named a **subscription**.</span></span> <span data-ttu-id="44dfe-109">Uma assinatura agrupa usuários e os recursos que foram criados pelos usuários.</span><span class="sxs-lookup"><span data-stu-id="44dfe-109">A subscription groups together users and the resources that have been created by those users.</span></span> <span data-ttu-id="44dfe-110">Cada um desses recursos contribui para um [limite geral][subscription-service-limits] nesse recurso específico.</span><span class="sxs-lookup"><span data-stu-id="44dfe-110">Each of those resources contributes to an [overall limit][subscription-service-limits] on that particular resource.</span></span>

<span data-ttu-id="44dfe-111">As organizações podem usar assinaturas para gerenciar os custos e a criação de recursos por usuários, equipes, projetos ou usando muitas outras estratégias.</span><span class="sxs-lookup"><span data-stu-id="44dfe-111">Organizations can use subscriptions to manage costs and creation of resource by users, teams, projects, or using many other strategies.</span></span> <span data-ttu-id="44dfe-112">Essas estratégias serão discutidas nos artigos sobre os estágios intermediário e avançado de adoção.</span><span class="sxs-lookup"><span data-stu-id="44dfe-112">These strategies will be discussed in the intermediate and advanced adoption stage articles.</span></span> 

## <a name="next-steps"></a><span data-ttu-id="44dfe-113">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="44dfe-113">Next steps</span></span>

* <span data-ttu-id="44dfe-114">Agora que você aprendeu sobre as assinaturas do Azure, saiba mais sobre [como criar uma assinatura](subscription.md) antes de criar seus primeiros recursos do Azure.</span><span class="sxs-lookup"><span data-stu-id="44dfe-114">Now that you have learned about Azure subscriptions, learn more about [creating a subscription](subscription.md) before you create your first Azure resources..</span></span>

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/en-us/get-started/
[azure-offers]: https://azure.microsoft.com/en-us/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/en-us/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/en-us/azure/active-directory/sign-up-organization
