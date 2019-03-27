---
title: Funções de aplicativo
description: Como executar a autorização usando funções de aplicativo.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: signup
pnp.series.next: authorize
ms.openlocfilehash: 097b150f3706614b0814bb83d0af01f9b502c5ea
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58248419"
---
# <a name="application-roles"></a><span data-ttu-id="f0262-103">Funções de aplicativo</span><span class="sxs-lookup"><span data-stu-id="f0262-103">Application roles</span></span>

<span data-ttu-id="f0262-104">[Código de exemplo do ![GitHub](../_images/github.png)][sample application]</span><span class="sxs-lookup"><span data-stu-id="f0262-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="f0262-105">As funções do aplicativo são usadas para atribuir permissões aos usuários.</span><span class="sxs-lookup"><span data-stu-id="f0262-105">Application roles are used to assign permissions to users.</span></span> <span data-ttu-id="f0262-106">Por exemplo, o aplicativo [Tailspin Surveys][tailspin] define as seguintes funções:</span><span class="sxs-lookup"><span data-stu-id="f0262-106">For example, the [Tailspin Surveys][tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="f0262-107">Administrador.</span><span class="sxs-lookup"><span data-stu-id="f0262-107">Administrator.</span></span> <span data-ttu-id="f0262-108">Pode executar todas as operações CRUD em qualquer pesquisa que pertença ao locatário.</span><span class="sxs-lookup"><span data-stu-id="f0262-108">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="f0262-109">Criador.</span><span class="sxs-lookup"><span data-stu-id="f0262-109">Creator.</span></span> <span data-ttu-id="f0262-110">Pode criar novas pesquisas.</span><span class="sxs-lookup"><span data-stu-id="f0262-110">Can create new surveys.</span></span>
* <span data-ttu-id="f0262-111">Leitor.</span><span class="sxs-lookup"><span data-stu-id="f0262-111">Reader.</span></span> <span data-ttu-id="f0262-112">Pode ler todas as pesquisas que pertençam ao locatário.</span><span class="sxs-lookup"><span data-stu-id="f0262-112">Can read any surveys that belong to that tenant.</span></span>

<span data-ttu-id="f0262-113">Você pode ver que as funções são definitivamente traduzidas em permissões durante a [autorização].</span><span class="sxs-lookup"><span data-stu-id="f0262-113">You can see that roles ultimately get translated into permissions, during [authorization].</span></span> <span data-ttu-id="f0262-114">No entanto, a primeira pergunta é como atribuir e gerenciar funções.</span><span class="sxs-lookup"><span data-stu-id="f0262-114">But the first question is how to assign and manage roles.</span></span> <span data-ttu-id="f0262-115">Identificamos as três opções principais:</span><span class="sxs-lookup"><span data-stu-id="f0262-115">We identified three main options:</span></span>

* [<span data-ttu-id="f0262-116">Funções de Aplicativo do Azure AD</span><span class="sxs-lookup"><span data-stu-id="f0262-116">Azure AD App Roles</span></span>](#roles-using-azure-ad-app-roles)
* [<span data-ttu-id="f0262-117">Grupos de segurança do AD Azure</span><span class="sxs-lookup"><span data-stu-id="f0262-117">Azure AD security groups</span></span>](#roles-using-azure-ad-security-groups)
* <span data-ttu-id="f0262-118">[Gerenciador de funções do aplicativo](#roles-using-an-application-role-manager).</span><span class="sxs-lookup"><span data-stu-id="f0262-118">[Application role manager](#roles-using-an-application-role-manager).</span></span>

## <a name="roles-using-azure-ad-app-roles"></a><span data-ttu-id="f0262-119">Funções usando funções de aplicativo do Azure AD</span><span class="sxs-lookup"><span data-stu-id="f0262-119">Roles using Azure AD App Roles</span></span>

<span data-ttu-id="f0262-120">Essa é a abordagem que usamos no aplicativo Tailspin Surveys.</span><span class="sxs-lookup"><span data-stu-id="f0262-120">This is the approach that we used in the Tailspin Surveys app.</span></span>

<span data-ttu-id="f0262-121">Nessa abordagem, o provedor de SaaS define as funções do aplicativo adicionando-as ao manifesto do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="f0262-121">In this approach, The SaaS provider defines the application roles by adding them to the application manifest.</span></span> <span data-ttu-id="f0262-122">Depois que um cliente se inscrever, um administrador do diretório do AD do cliente atribuirá usuários às funções.</span><span class="sxs-lookup"><span data-stu-id="f0262-122">After a customer signs up, an admin for the customer's AD directory assigns users to the roles.</span></span> <span data-ttu-id="f0262-123">Quando um usuário entrar, as funções atribuídas ao usuário serão enviadas como declarações.</span><span class="sxs-lookup"><span data-stu-id="f0262-123">When a user signs in, the user's assigned roles are sent as claims.</span></span>

> [!NOTE]
> <span data-ttu-id="f0262-124">Se o cliente tiver o Azure AD Premium, o administrador poderá atribuir um grupo de segurança a uma função e os membros do grupo herdarão a função do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="f0262-124">If the customer has Azure AD Premium, the admin can assign a security group to a role, and members of the group will inherit the app role.</span></span> <span data-ttu-id="f0262-125">Essa é uma maneira conveniente de gerenciar funções porque o proprietário do grupo não precisa ser um administrador do AD.</span><span class="sxs-lookup"><span data-stu-id="f0262-125">This is a convenient way to manage roles, because the group owner doesn't need to be an AD admin.</span></span>

<span data-ttu-id="f0262-126">As vantagens dessa abordagem:</span><span class="sxs-lookup"><span data-stu-id="f0262-126">Advantages of this approach:</span></span>

* <span data-ttu-id="f0262-127">Modelo de programação simples.</span><span class="sxs-lookup"><span data-stu-id="f0262-127">Simple programming model.</span></span>
* <span data-ttu-id="f0262-128">As funções são específicas do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="f0262-128">Roles are specific to the application.</span></span> <span data-ttu-id="f0262-129">As declarações de função para um aplicativo não são enviadas para outro aplicativo.</span><span class="sxs-lookup"><span data-stu-id="f0262-129">The role claims for one application are not sent to another application.</span></span>
* <span data-ttu-id="f0262-130">Se o cliente remover o aplicativo do seu locatário do AD, as funções desaparecerão.</span><span class="sxs-lookup"><span data-stu-id="f0262-130">If the customer removes the application from their AD tenant, the roles go away.</span></span>
* <span data-ttu-id="f0262-131">O aplicativo não precisa de permissões extras do Active Directory além da leitura do perfil do usuário.</span><span class="sxs-lookup"><span data-stu-id="f0262-131">The application doesn't need any extra Active Directory permissions, other than reading the user's profile.</span></span>

<span data-ttu-id="f0262-132">As desvantagens:</span><span class="sxs-lookup"><span data-stu-id="f0262-132">Drawbacks:</span></span>

* <span data-ttu-id="f0262-133">Os clientes que não têm o Azure AD Premium não podem atribuir grupos de segurança a funções.</span><span class="sxs-lookup"><span data-stu-id="f0262-133">Customers without Azure AD Premium cannot assign security groups to roles.</span></span> <span data-ttu-id="f0262-134">Para esses clientes, todas as atribuições de usuário devem ser feitas por um administrador do AD.</span><span class="sxs-lookup"><span data-stu-id="f0262-134">For these customers, all user assignments must be done by an AD administrator.</span></span>
* <span data-ttu-id="f0262-135">Se você tiver uma API Web de back-end, que é separada do aplicativo Web, as atribuições de função para o aplicativo Web não se aplicarão à API Web.</span><span class="sxs-lookup"><span data-stu-id="f0262-135">If you have a backend web API, which is separate from the web app, then role assignments for the web app don't apply to the web API.</span></span> <span data-ttu-id="f0262-136">Para obter mais informações sobre este ponto, confira [Protegendo uma API Web de back-end].</span><span class="sxs-lookup"><span data-stu-id="f0262-136">For more discussion of this point, see [Securing a backend web API].</span></span>

### <a name="implementation"></a><span data-ttu-id="f0262-137">Implementação</span><span class="sxs-lookup"><span data-stu-id="f0262-137">Implementation</span></span>

<span data-ttu-id="f0262-138">**Defina as funções.**</span><span class="sxs-lookup"><span data-stu-id="f0262-138">**Define the roles.**</span></span> <span data-ttu-id="f0262-139">O provedor de SaaS declara as funções de aplicativo no [manifesto do aplicativo].</span><span class="sxs-lookup"><span data-stu-id="f0262-139">The SaaS provider declares the app roles in the [application manifest].</span></span> <span data-ttu-id="f0262-140">Por exemplo, esta é a entrada do manifesto do aplicativo Surveys:</span><span class="sxs-lookup"><span data-stu-id="f0262-140">For example, here is the manifest entry for the Surveys app:</span></span>

```json
"appRoles": [
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Creators can create Surveys",
    "displayName": "SurveyCreator",
    "id": "1b4f816e-5eaf-48b9-8613-7923830595ad",
    "isEnabled": true,
    "value": "SurveyCreator"
  },
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Administrators can manage the Surveys in their tenant",
    "displayName": "SurveyAdmin",
    "id": "c20e145e-5459-4a6c-a074-b942bbd4cfe1",
    "isEnabled": true,
    "value": "SurveyAdmin"
  }
],
```

<span data-ttu-id="f0262-141">A propriedade `value` aparece na declaração de função.</span><span class="sxs-lookup"><span data-stu-id="f0262-141">The `value`  property appears in the role claim.</span></span> <span data-ttu-id="f0262-142">A propriedade `id` é o identificador exclusivo para a função definida.</span><span class="sxs-lookup"><span data-stu-id="f0262-142">The `id` property is the unique identifier for the defined role.</span></span> <span data-ttu-id="f0262-143">Gere sempre um novo valor de GUID para `id`.</span><span class="sxs-lookup"><span data-stu-id="f0262-143">Always generate a new GUID value for `id`.</span></span>

<span data-ttu-id="f0262-144">**Atribua usuários**.</span><span class="sxs-lookup"><span data-stu-id="f0262-144">**Assign users**.</span></span> <span data-ttu-id="f0262-145">Quando um novo cliente se inscrever, o aplicativo será registrado no locatário do AD do cliente.</span><span class="sxs-lookup"><span data-stu-id="f0262-145">When a new customer signs up, the application is registered in the customer's AD tenant.</span></span> <span data-ttu-id="f0262-146">Neste ponto, um administrador do AD para o locatário poderá atribuir usuários a funções.</span><span class="sxs-lookup"><span data-stu-id="f0262-146">At this point, an AD admin for that tenant can assign users to roles.</span></span>

> [!NOTE]
> <span data-ttu-id="f0262-147">Como observado anteriormente, os clientes com o Azure AD Premium também podem atribuir grupos de segurança a funções.</span><span class="sxs-lookup"><span data-stu-id="f0262-147">As noted earlier, customers with Azure AD Premium can also assign security groups to roles.</span></span>

<span data-ttu-id="f0262-148">A captura de tela a seguir no portal do Azure mostra os usuários e os grupos para o aplicativo Pesquisa.</span><span class="sxs-lookup"><span data-stu-id="f0262-148">The following screenshot from the Azure portal shows users and groups for the Survey application.</span></span> <span data-ttu-id="f0262-149">Administrador e Criador são grupos, atribuídos às funções SurveyAdmin e SurveyCreator, respectivamente.</span><span class="sxs-lookup"><span data-stu-id="f0262-149">Admin and Creator are groups, assigned to SurveyAdmin and SurveyCreator roles respectively.</span></span> <span data-ttu-id="f0262-150">Alice é uma usuária que foi atribuída diretamente à função SurveyAdmin.</span><span class="sxs-lookup"><span data-stu-id="f0262-150">Alice is a user who was assigned directly to the SurveyAdmin role.</span></span> <span data-ttu-id="f0262-151">Paulo e Davi são usuários que não foram atribuídos diretamente a uma função.</span><span class="sxs-lookup"><span data-stu-id="f0262-151">Bob and Charles are users that have not been directly assigned to a role.</span></span>

![Usuários e Grupos](./images/running-the-app/users-and-groups.png)

<span data-ttu-id="f0262-153">Conforme mostrado na seguinte captura de tela, Davi faz parte do grupo de administração, portanto ele herda a função SurveyAdmin.</span><span class="sxs-lookup"><span data-stu-id="f0262-153">As shown in the following screenshot, Charles is part of the Admin group, so he inherits the SurveyAdmin role.</span></span> <span data-ttu-id="f0262-154">No caso de Paulo, ele ainda não teve uma função atribuída.</span><span class="sxs-lookup"><span data-stu-id="f0262-154">In the case of Bob, he has not been assigned a role yet.</span></span>

![Membros do grupo Admin](./images/running-the-app/admin-members.png)

> [!NOTE]
> <span data-ttu-id="f0262-156">Como abordagem alternativa, o aplicativo pode atribuir funções programaticamente usando a API do Graph do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="f0262-156">An alternative approach is for the application to assign roles programmatically, using the Azure AD Graph API.</span></span> <span data-ttu-id="f0262-157">No entanto, isso requer que o aplicativo obtenha permissões de gravação para o diretório do AD do cliente.</span><span class="sxs-lookup"><span data-stu-id="f0262-157">However, this requires the application to obtain write permissions for the customer's AD directory.</span></span> <span data-ttu-id="f0262-158">Um aplicativo com essas permissões poderia causar muitos danos &mdash; o cliente confia que o aplicativo não bagunçará seu diretório.</span><span class="sxs-lookup"><span data-stu-id="f0262-158">An application with those permissions could do a lot of mischief &mdash; the customer is trusting the app not to mess up their directory.</span></span> <span data-ttu-id="f0262-159">É provável que muitos clientes não queiram conceder esse nível de acesso.</span><span class="sxs-lookup"><span data-stu-id="f0262-159">Many customers might be unwilling to grant this level of access.</span></span>

<span data-ttu-id="f0262-160">**Obtenha declarações de função**.</span><span class="sxs-lookup"><span data-stu-id="f0262-160">**Get role claims**.</span></span> <span data-ttu-id="f0262-161">Quando um usuário entra, o aplicativo recebe as funções atribuídas ao usuário em uma declaração com o tipo `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`.</span><span class="sxs-lookup"><span data-stu-id="f0262-161">When a user signs in, the application receives the user's assigned role(s) in a claim with type `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`.</span></span>

<span data-ttu-id="f0262-162">Um usuário pode ter várias funções ou nenhuma função.</span><span class="sxs-lookup"><span data-stu-id="f0262-162">A user can have multiple roles, or no role.</span></span> <span data-ttu-id="f0262-163">Em seu código de autorização, não presuma que o usuário tenha exatamente uma função de declaração.</span><span class="sxs-lookup"><span data-stu-id="f0262-163">In your authorization code, don't assume the user has exactly one role claim.</span></span> <span data-ttu-id="f0262-164">Em vez disso, grave um código que verifique se há um valor de declaração específico presente:</span><span class="sxs-lookup"><span data-stu-id="f0262-164">Instead, write code that checks whether a particular claim value is present:</span></span>

```csharp
if (context.User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
```

## <a name="roles-using-azure-ad-security-groups"></a><span data-ttu-id="f0262-165">As funções que usam grupos de segurança do Azure AD</span><span class="sxs-lookup"><span data-stu-id="f0262-165">Roles using Azure AD security groups</span></span>

<span data-ttu-id="f0262-166">Nessa abordagem, as funções são representadas como grupos de segurança do AD.</span><span class="sxs-lookup"><span data-stu-id="f0262-166">In this approach, roles are represented as AD security groups.</span></span> <span data-ttu-id="f0262-167">O aplicativo atribui permissões a usuários com base em suas associações a grupos de segurança.</span><span class="sxs-lookup"><span data-stu-id="f0262-167">The application assigns permissions to users based on their security group memberships.</span></span>

<span data-ttu-id="f0262-168">Vantagens:</span><span class="sxs-lookup"><span data-stu-id="f0262-168">Advantages:</span></span>

* <span data-ttu-id="f0262-169">Para os clientes que não têm o Azure AD Premium, essa abordagem permite que o cliente use grupos de segurança para gerenciar as atribuições de função.</span><span class="sxs-lookup"><span data-stu-id="f0262-169">For customers who do not have Azure AD Premium, this approach enables the customer to use security groups to manage role assignments.</span></span>

<span data-ttu-id="f0262-170">As desvantagens:</span><span class="sxs-lookup"><span data-stu-id="f0262-170">Disadvantages:</span></span>

* <span data-ttu-id="f0262-171">Complexidade.</span><span class="sxs-lookup"><span data-stu-id="f0262-171">Complexity.</span></span> <span data-ttu-id="f0262-172">Como todos os locatários enviam diferentes declarações de grupo, o aplicativo deverá manter o controle de quais grupos de segurança correspondem a quais funções de aplicativo para cada locatário.</span><span class="sxs-lookup"><span data-stu-id="f0262-172">Because every tenant sends different group claims, the app must keep track of which security groups correspond to which application roles, for each tenant.</span></span>
* <span data-ttu-id="f0262-173">Se o cliente remover o aplicativo do seu locatário do AD, os grupos de segurança serão deixados no diretório do AD dele.</span><span class="sxs-lookup"><span data-stu-id="f0262-173">If the customer removes the application from their AD tenant, the security groups are left in their AD directory.</span></span>

<!-- markdownlint-disable MD024 -->

### <a name="implementation"></a><span data-ttu-id="f0262-174">Implementação</span><span class="sxs-lookup"><span data-stu-id="f0262-174">Implementation</span></span>

<!-- markdownlint-enable MD024 -->

<span data-ttu-id="f0262-175">No manifesto do aplicativo, defina a propriedade `groupMembershipClaims` como "SecurityGroup".</span><span class="sxs-lookup"><span data-stu-id="f0262-175">In the application manifest, set the `groupMembershipClaims` property to "SecurityGroup".</span></span> <span data-ttu-id="f0262-176">Isso é necessário para obter as declarações de associação de grupo do AAD.</span><span class="sxs-lookup"><span data-stu-id="f0262-176">This is needed to get group membership claims from AAD.</span></span>

```json
{
   // ...
   "groupMembershipClaims": "SecurityGroup",
}
```

<span data-ttu-id="f0262-177">Quando um novo cliente se inscrever, o aplicativo instruirá o cliente a criar grupos de segurança para as funções exigidas pelo aplicativo.</span><span class="sxs-lookup"><span data-stu-id="f0262-177">When a new customer signs up, the application instructs the customer to create security groups for the roles needed by the application.</span></span> <span data-ttu-id="f0262-178">Em seguida, o cliente precisará inserir as IDs de objeto de grupo no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="f0262-178">The customer then needs to enter the group object IDs into the application.</span></span> <span data-ttu-id="f0262-179">O aplicativo as armazena em uma tabela que mapeia as IDs de grupo para as funções de aplicativo por locatário.</span><span class="sxs-lookup"><span data-stu-id="f0262-179">The application stores these in a table that maps group IDs to application roles, per tenant.</span></span>

> [!NOTE]
> <span data-ttu-id="f0262-180">Como alternativa, o aplicativo poderia criar os grupos programaticamente usando a API do Azure AD Graph.</span><span class="sxs-lookup"><span data-stu-id="f0262-180">Alternatively, the application could create the groups programmatically, using the Azure AD Graph API.</span></span>  <span data-ttu-id="f0262-181">Isso seria menos propenso a erros.</span><span class="sxs-lookup"><span data-stu-id="f0262-181">This would be less error prone.</span></span> <span data-ttu-id="f0262-182">No entanto, isso requer que o aplicativo obtenha permissões de “leitura e de gravação em todos os grupos” para o diretório do AD do cliente.</span><span class="sxs-lookup"><span data-stu-id="f0262-182">However, it requires the application to obtain "read and write all groups" permissions for the customer's AD directory.</span></span> <span data-ttu-id="f0262-183">É provável que muitos clientes não queiram conceder esse nível de acesso.</span><span class="sxs-lookup"><span data-stu-id="f0262-183">Many customers might be unwilling to grant this level of access.</span></span>

<span data-ttu-id="f0262-184">Quando um usuário entra:</span><span class="sxs-lookup"><span data-stu-id="f0262-184">When a user signs in:</span></span>

1. <span data-ttu-id="f0262-185">O aplicativo recebe os grupos do usuário como declarações.</span><span class="sxs-lookup"><span data-stu-id="f0262-185">The application receives the user's groups as claims.</span></span> <span data-ttu-id="f0262-186">O valor de cada declaração é a ID do objeto de um grupo.</span><span class="sxs-lookup"><span data-stu-id="f0262-186">The value of each claim is the object ID of a group.</span></span>
2. <span data-ttu-id="f0262-187">O Azure AD limita o número de grupos enviados no token.</span><span class="sxs-lookup"><span data-stu-id="f0262-187">Azure AD limits the number of groups sent in the token.</span></span> <span data-ttu-id="f0262-188">Se o número de grupos exceder esse limite, o Azure AD enviará uma declaração "excedente" especial.</span><span class="sxs-lookup"><span data-stu-id="f0262-188">If the number of groups exceeds this limit, Azure AD sends a special "overage" claim.</span></span> <span data-ttu-id="f0262-189">Se essa declaração estiver presente, o aplicativo deverá consultar a API do Azure AD Graph para obter todos os grupos aos quais o usuário pertence.</span><span class="sxs-lookup"><span data-stu-id="f0262-189">If that claim is present, the application must query the Azure AD Graph API to get all of the groups to which that user belongs.</span></span> <span data-ttu-id="f0262-190">Para obter detalhes, confira [Autorização em Aplicativos de Nuvem usando grupos do AD], na seção intitulada "Excedente de declarações de grupos".</span><span class="sxs-lookup"><span data-stu-id="f0262-190">For details, see [Authorization in Cloud Applications using AD Groups], under the section titled "Groups claim overage".</span></span>
3. <span data-ttu-id="f0262-191">O aplicativo procura as IDs do objeto em seu próprio banco de dados para localizar as funções de aplicativo correspondentes a serem atribuídas ao usuário.</span><span class="sxs-lookup"><span data-stu-id="f0262-191">The application looks up the object IDs in its own database, to find the corresponding application roles to assign to the user.</span></span>
4. <span data-ttu-id="f0262-192">O aplicativo adiciona um valor de declaração personalizada à entidade de usuário que expressa a função de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="f0262-192">The application adds a custom claim value to the user principal that expresses the application role.</span></span> <span data-ttu-id="f0262-193">Por exemplo: `survey_role` = "SurveyAdmin".</span><span class="sxs-lookup"><span data-stu-id="f0262-193">For example: `survey_role` = "SurveyAdmin".</span></span>

<span data-ttu-id="f0262-194">As políticas de autorização devem usar a declaração de função personalizada e não a declaração de grupo.</span><span class="sxs-lookup"><span data-stu-id="f0262-194">Authorization policies should use the custom role claim, not the group claim.</span></span>

## <a name="roles-using-an-application-role-manager"></a><span data-ttu-id="f0262-195">Funções que usam um aplicativo do gerenciador de funções</span><span class="sxs-lookup"><span data-stu-id="f0262-195">Roles using an application role manager</span></span>

<span data-ttu-id="f0262-196">Com essa abordagem, as funções de aplicativo não são armazenadas no Azure AD.</span><span class="sxs-lookup"><span data-stu-id="f0262-196">With this approach, application roles are not stored in Azure AD at all.</span></span> <span data-ttu-id="f0262-197">Em vez disso, o aplicativo armazena as atribuições de função para cada usuário em seu próprio banco de dados &mdash; por exemplo, usando a classe **RoleManager** no ASP.NET Identity.</span><span class="sxs-lookup"><span data-stu-id="f0262-197">Instead, the application stores the role assignments for each user in its own DB &mdash; for example, using the **RoleManager** class in ASP.NET Identity.</span></span>

<span data-ttu-id="f0262-198">Vantagens:</span><span class="sxs-lookup"><span data-stu-id="f0262-198">Advantages:</span></span>

* <span data-ttu-id="f0262-199">O aplicativo tem controle total sobre as funções e as atribuições de usuário.</span><span class="sxs-lookup"><span data-stu-id="f0262-199">The app has full control over the roles and user assignments.</span></span>

<span data-ttu-id="f0262-200">As desvantagens:</span><span class="sxs-lookup"><span data-stu-id="f0262-200">Drawbacks:</span></span>

* <span data-ttu-id="f0262-201">Mais complexo e mais difícil de manter.</span><span class="sxs-lookup"><span data-stu-id="f0262-201">More complex, harder to maintain.</span></span>
* <span data-ttu-id="f0262-202">Não é possível usar grupos de segurança do AD para gerenciar atribuições de função.</span><span class="sxs-lookup"><span data-stu-id="f0262-202">Cannot use AD security groups to manage role assignments.</span></span>
* <span data-ttu-id="f0262-203">Armazena informações de usuário no banco de dados do aplicativo, onde podem ficar fora de sincronia com o diretório do AD do locatário à medida que usuários são adicionados ou removidos.</span><span class="sxs-lookup"><span data-stu-id="f0262-203">Stores user information in the application database, where it can get out of sync with the tenant's AD directory, as users are added or removed.</span></span>

<span data-ttu-id="f0262-204">[**Avançar**][autorização]</span><span class="sxs-lookup"><span data-stu-id="f0262-204">[**Next**][authorization]</span></span>

<!-- links -->

[tailspin]: tailspin.md
[autorização]: authorize.md
[authorization]: authorize.md
[Protegendo uma API Web de back-end]: web-api.md
[Securing a backend web API]: web-api.md
[manifesto do aplicativo]: /azure/active-directory/active-directory-application-manifest/
[application manifest]: /azure/active-directory/active-directory-application-manifest/
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
