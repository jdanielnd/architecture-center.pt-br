---
title: Gerenciamento de Identidades para Aplicativos Multilocatários
description: Práticas recomendadas para autenticação, autorização e gerenciamento de identidades em aplicativos multilocatários.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: f8875612ad6b1a71fdb6f7a768078ae599eb70b5
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54480533"
---
# <a name="manage-identity-in-multitenant-applications"></a><span data-ttu-id="06778-103">Gerenciar a identidade em aplicativos multilocatários</span><span class="sxs-lookup"><span data-stu-id="06778-103">Manage identity in multitenant applications</span></span>

<span data-ttu-id="06778-104">Esta série de artigos descreve as práticas recomendadas para multilocação, ao usar o Azure AD para autenticação e gerenciamento de identidades.</span><span class="sxs-lookup"><span data-stu-id="06778-104">This series of articles describes best practices for multitenancy, when using Azure AD for authentication and identity management.</span></span>

<span data-ttu-id="06778-105">[Código de exemplo do ![GitHub](../_images/github.png)][sample-application]</span><span class="sxs-lookup"><span data-stu-id="06778-105">[![GitHub](../_images/github.png) Sample code][sample-application]</span></span>

<span data-ttu-id="06778-106">Quando estiver compilando um aplicativo multilocatário, um dos primeiros desafios é gerenciar identidades de usuário, porque agora cada usuário pertence a um locatário.</span><span class="sxs-lookup"><span data-stu-id="06778-106">When you're building a multitenant application, one of the first challenges is managing user identities, because now every user belongs to a tenant.</span></span> <span data-ttu-id="06778-107">Por exemplo: </span><span class="sxs-lookup"><span data-stu-id="06778-107">For example:</span></span>

- <span data-ttu-id="06778-108">Os usuários entram com suas credenciais organizacionais.</span><span class="sxs-lookup"><span data-stu-id="06778-108">Users sign in with their organizational credentials.</span></span>
- <span data-ttu-id="06778-109">Eles devem ter acesso aos dados de sua organização, mas não aos dados que pertencem a outros locatários.</span><span class="sxs-lookup"><span data-stu-id="06778-109">Users should have access to their organization's data, but not data that belongs to other tenants.</span></span>
- <span data-ttu-id="06778-110">Uma organização pode se inscrever para obter o aplicativo e, em seguida, atribuir funções do aplicativo a seus membros.</span><span class="sxs-lookup"><span data-stu-id="06778-110">An organization can sign up for the application, and then assign application roles to its members.</span></span>

<span data-ttu-id="06778-111">O Azure AD (Azure Active Directory) tem alguns excelentes recursos que dão suporte a todos esses cenários.</span><span class="sxs-lookup"><span data-stu-id="06778-111">Azure Active Directory (Azure AD) has some great features that support all of these scenarios.</span></span>

<span data-ttu-id="06778-112">Para acompanhar esta série de artigos, criamos uma [implementação de ponta a ponta][sample-application] completa de um aplicativo multilocatário.</span><span class="sxs-lookup"><span data-stu-id="06778-112">To accompany this series of articles, we created a complete [end-to-end implementation][sample-application] of a multitenant application.</span></span> <span data-ttu-id="06778-113">Os artigos refletem o que aprendemos no processo de compilação do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="06778-113">The articles reflect what we learned in the process of building the application.</span></span> <span data-ttu-id="06778-114">Para começar a usar o aplicativo, veja [Executar o aplicativo Surveys][running-the-app].</span><span class="sxs-lookup"><span data-stu-id="06778-114">To get started with the application, see [Run the Surveys application][running-the-app].</span></span>

## <a name="introduction"></a><span data-ttu-id="06778-115">Introdução</span><span class="sxs-lookup"><span data-stu-id="06778-115">Introduction</span></span>

<span data-ttu-id="06778-116">Digamos que você esteja gravando um aplicativo SaaS corporativo para ser hospedado na nuvem.</span><span class="sxs-lookup"><span data-stu-id="06778-116">Let's say you're writing an enterprise SaaS application to be hosted in the cloud.</span></span> <span data-ttu-id="06778-117">É óbvio que o aplicativo terá usuários:</span><span class="sxs-lookup"><span data-stu-id="06778-117">Of course, the application will have users:</span></span>

![Usuários](./images/users.png)

<span data-ttu-id="06778-119">Mas esses usuários pertencem a organizações:</span><span class="sxs-lookup"><span data-stu-id="06778-119">But those users belong to organizations:</span></span>

![Usuários da organização](./images/org-users.png)

<span data-ttu-id="06778-121">Exemplo: A Tailspin vende assinaturas para seu aplicativo SaaS.</span><span class="sxs-lookup"><span data-stu-id="06778-121">Example: Tailspin sells subscriptions to its SaaS application.</span></span> <span data-ttu-id="06778-122">A Contoso e a Fabrikam inscrevem-se no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="06778-122">Contoso and Fabrikam sign up for the app.</span></span> <span data-ttu-id="06778-123">Quando Alice (`alice@contoso`) entra, o aplicativo deve saber que Alice faz parte da Contoso.</span><span class="sxs-lookup"><span data-stu-id="06778-123">When Alice (`alice@contoso`) signs in, the application should know that Alice is part of Contoso.</span></span>

- <span data-ttu-id="06778-124">Alice *deve* ter acesso aos dados da Contoso.</span><span class="sxs-lookup"><span data-stu-id="06778-124">Alice *should* have access to Contoso data.</span></span>
- <span data-ttu-id="06778-125">Alice *não deve* ter acesso aos dados da Fabrikam.</span><span class="sxs-lookup"><span data-stu-id="06778-125">Alice *should not* have access to Fabrikam data.</span></span>

<span data-ttu-id="06778-126">Essa diretriz mostra como gerenciar as identidades dos usuários em um aplicativo multilocatário, usando o [Azure Active Directory (Azure AD)](/azure/active-directory) para lidar com a entrada e a autenticação.</span><span class="sxs-lookup"><span data-stu-id="06778-126">This guidance will show you how to manage user identities in a multitenant application, using [Azure Active Directory (Azure AD)](/azure/active-directory) to handle sign-in and authentication.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-is-multitenancy"></a><span data-ttu-id="06778-127">O que é a multilocação?</span><span class="sxs-lookup"><span data-stu-id="06778-127">What is multitenancy?</span></span>

<!-- markdownlint-enable MD026 -->

<span data-ttu-id="06778-128">Um *locatário* é um grupo de usuários.</span><span class="sxs-lookup"><span data-stu-id="06778-128">A *tenant* is a group of users.</span></span> <span data-ttu-id="06778-129">Em um aplicativo SaaS, o locatário é um assinante ou o cliente do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="06778-129">In a SaaS application, the tenant is a subscriber or customer of the application.</span></span> <span data-ttu-id="06778-130">*Multilocação* é uma arquitetura onde vários locatários compartilham a mesma instância física do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="06778-130">*Multitenancy* is an architecture where multiple tenants share the same physical instance of the app.</span></span> <span data-ttu-id="06778-131">Embora locatários compartilhem recursos físicos (como VMs ou armazenamento), cada locatário obtém sua própria instância lógica do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="06778-131">Although tenants share physical resources (such as VMs or storage), each tenant gets its own logical instance of the app.</span></span>

<span data-ttu-id="06778-132">Normalmente, os dados de aplicativo são compartilhados entre os usuários dentro de um locatário, mas não com outros locatários.</span><span class="sxs-lookup"><span data-stu-id="06778-132">Typically, application data is shared among the users within a tenant, but not with other tenants.</span></span>

![Multilocatário](./images/multitenant.png)

<span data-ttu-id="06778-134">Compare essa arquitetura com uma arquitetura de locatário único, onde cada locatário tem uma instância física dedicada.</span><span class="sxs-lookup"><span data-stu-id="06778-134">Compare this architecture with a single-tenant architecture, where each tenant has a dedicated physical instance.</span></span> <span data-ttu-id="06778-135">Em uma arquitetura de locatário único, você pode adicionar locatários gerando novas instâncias do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="06778-135">In a single-tenant architecture, you add tenants by spinning up new instances of the app.</span></span>

![Locatário único](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a><span data-ttu-id="06778-137">Multilocação e dimensionamento horizontal</span><span class="sxs-lookup"><span data-stu-id="06778-137">Multitenancy and horizontal scaling</span></span>

<span data-ttu-id="06778-138">É comum adicionar mais instâncias para dimensionar na nuvem.</span><span class="sxs-lookup"><span data-stu-id="06778-138">To achieve scale in the cloud, it’s common to add more physical instances.</span></span> <span data-ttu-id="06778-139">Isso é conhecido como *escalonamento horizontal* ou *expansão*. Considere um aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="06778-139">This is known as *horizontal scaling* or *scaling out*. Consider a web app.</span></span> <span data-ttu-id="06778-140">Para lidar com mais tráfego, você pode adicionar mais VMs de servidor e colocá-las atrás de um balanceador de carga.</span><span class="sxs-lookup"><span data-stu-id="06778-140">To handle more traffic, you can add more server VMs and put them behind a load balancer.</span></span> <span data-ttu-id="06778-141">Cada VM executa uma instância física separada do aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="06778-141">Each VM runs a separate physical instance of the web app.</span></span>

![Balancear a carga de um site](./images/load-balancing.png)

<span data-ttu-id="06778-143">Qualquer solicitação pode ser roteada para qualquer instância.</span><span class="sxs-lookup"><span data-stu-id="06778-143">Any request can be routed to any instance.</span></span> <span data-ttu-id="06778-144">Em conjunto, o sistema funciona como uma única instância lógica.</span><span class="sxs-lookup"><span data-stu-id="06778-144">Together, the system functions as a single logical instance.</span></span> <span data-ttu-id="06778-145">Você pode subdividir uma VM ou criar uma nova VM sem afetar os usuários.</span><span class="sxs-lookup"><span data-stu-id="06778-145">You can tear down a VM or spin up a new VM, without affecting users.</span></span> <span data-ttu-id="06778-146">Nesta arquitetura, cada instância física é multilocatário e você pode dimensionar adicionando mais instâncias.</span><span class="sxs-lookup"><span data-stu-id="06778-146">In this architecture, each physical instance is multi-tenant, and you scale by adding more instances.</span></span> <span data-ttu-id="06778-147">Se uma instância falhar, não deverá afetar nenhum locatário.</span><span class="sxs-lookup"><span data-stu-id="06778-147">If one instance goes down, it should not affect any tenant.</span></span>

## <a name="identity-in-a-multitenant-app"></a><span data-ttu-id="06778-148">Identidade em um aplicativo multilocatário</span><span class="sxs-lookup"><span data-stu-id="06778-148">Identity in a multitenant app</span></span>

<span data-ttu-id="06778-149">Em um aplicativo multilocatário, você deve considerar os usuários no contexto de locatários.</span><span class="sxs-lookup"><span data-stu-id="06778-149">In a multitenant app, you must consider users in the context of tenants.</span></span>

### <a name="authentication"></a><span data-ttu-id="06778-150">Autenticação</span><span class="sxs-lookup"><span data-stu-id="06778-150">Authentication</span></span>

- <span data-ttu-id="06778-151">Os usuários entram no aplicativo com suas credenciais da organização.</span><span class="sxs-lookup"><span data-stu-id="06778-151">Users sign into the app with their organization credentials.</span></span> <span data-ttu-id="06778-152">Eles não precisam criar novos perfis de usuário para o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="06778-152">They don't have to create new user profiles for the app.</span></span>
- <span data-ttu-id="06778-153">Os usuários dentro da mesma organização fazem parte do mesmo locatário.</span><span class="sxs-lookup"><span data-stu-id="06778-153">Users within the same organization are part of the same tenant.</span></span>
- <span data-ttu-id="06778-154">Quando um usuário faz logon, o aplicativo sabe a qual locatário o usuário pertence.</span><span class="sxs-lookup"><span data-stu-id="06778-154">When a user signs in, the application knows which tenant the user belongs to.</span></span>

### <a name="authorization"></a><span data-ttu-id="06778-155">Autorização</span><span class="sxs-lookup"><span data-stu-id="06778-155">Authorization</span></span>

- <span data-ttu-id="06778-156">Ao autorizar ações de um usuário (digamos, exibição de um recurso), o aplicativo deve levar em conta o locatário do usuário.</span><span class="sxs-lookup"><span data-stu-id="06778-156">When authorizing a user's actions (say, viewing a resource), the app must take into account the user's tenant.</span></span>
- <span data-ttu-id="06778-157">Os usuários podem ser atribuídos a funções dentro do aplicativo, como "Admin" ou "Usuário Padrão".</span><span class="sxs-lookup"><span data-stu-id="06778-157">Users might be assigned roles within the application, such as "Admin" or "Standard User".</span></span> <span data-ttu-id="06778-158">As atribuições de função devem ser gerenciadas pelo cliente e não pelo provedor de SaaS.</span><span class="sxs-lookup"><span data-stu-id="06778-158">Role assignments should be managed by the customer, not by the SaaS provider.</span></span>

<span data-ttu-id="06778-159">**Exemplo:**</span><span class="sxs-lookup"><span data-stu-id="06778-159">**Example.**</span></span> <span data-ttu-id="06778-160">Alice, uma funcionária da Contoso, navega para o aplicativo em seu navegador e clica no botão "Entrar".</span><span class="sxs-lookup"><span data-stu-id="06778-160">Alice, an employee at Contoso, navigates to the application in her browser and clicks the “Log in” button.</span></span> <span data-ttu-id="06778-161">Ela é redirecionada para uma tela de logon onde insere suas credenciais corporativas (nome de usuário e senha).</span><span class="sxs-lookup"><span data-stu-id="06778-161">She is redirected to a login screen where she enters her corporate credentials (username and password).</span></span> <span data-ttu-id="06778-162">Neste ponto, ele está conectada ao aplicativo como `alice@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="06778-162">At this point, she is logged into the app as `alice@contoso.com`.</span></span> <span data-ttu-id="06778-163">O aplicativo também sabe que Alice é um usuário administrador deste aplicativo.</span><span class="sxs-lookup"><span data-stu-id="06778-163">The application also knows that Alice is an admin user for this application.</span></span> <span data-ttu-id="06778-164">Por ser administrador, ela pode ver uma lista de todos os recursos que pertencem à Contoso.</span><span class="sxs-lookup"><span data-stu-id="06778-164">Because she is an admin, she can see a list of all the resources that belong to Contoso.</span></span> <span data-ttu-id="06778-165">No entanto, ela não poderá exibir recursos da Fabrikam pois é admin somente dentro de seu locatário.</span><span class="sxs-lookup"><span data-stu-id="06778-165">However, she cannot view Fabrikam's resources, because she is an admin only within her tenant.</span></span>

<span data-ttu-id="06778-166">Neste guia, vamos examinar especificamente o uso do Azure AD para gerenciamento de identidades.</span><span class="sxs-lookup"><span data-stu-id="06778-166">In this guidance, we'll look specifically at using Azure AD for identity management.</span></span>

- <span data-ttu-id="06778-167">Supomos que o cliente armazene seus perfis de usuário no Azure AD (incluindo locatários do Office 365 e Dynamics CRM)</span><span class="sxs-lookup"><span data-stu-id="06778-167">We assume the customer stores their user profiles in Azure AD (including Office365 and Dynamics CRM tenants)</span></span>
- <span data-ttu-id="06778-168">Os clientes com AD (Active Directory) local podem usar o [Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity) para sincronizar seu AD local com o Azure AD.</span><span class="sxs-lookup"><span data-stu-id="06778-168">Customers with on-premise Active Directory (AD) can use [Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity) to sync their on-premise AD with Azure AD.</span></span>

<span data-ttu-id="06778-169">Se um cliente com o AD local não conseguir usar o Azure AD Connect (devido à política de TI corporativa ou por outros motivos), o provedor de SaaS poderá federar com o AD do cliente por meio do AD FS (Serviços de Federação do Active Directory).</span><span class="sxs-lookup"><span data-stu-id="06778-169">If a customer with on-premise AD cannot use Azure AD Connect (due to corporate IT policy or other reasons), the SaaS provider can federate with the customer's AD through Active Directory Federation Services (AD FS).</span></span> <span data-ttu-id="06778-170">Essa opção é descrita em [Federar com o AD FS de um cliente](adfs.md).</span><span class="sxs-lookup"><span data-stu-id="06778-170">This option is described in [Federating with a customer's AD FS](adfs.md).</span></span>

<span data-ttu-id="06778-171">Este guia não considera outros aspectos da multilocação, como particionamento de dados, configuração por locatário e assim por diante.</span><span class="sxs-lookup"><span data-stu-id="06778-171">This guidance does not consider other aspects of multitenancy such as data partitioning, per-tenant configuration, and so forth.</span></span>

[<span data-ttu-id="06778-172">**Avançar**</span><span class="sxs-lookup"><span data-stu-id="06778-172">**Next**</span></span>](./tailspin.md)

<!-- links -->

[sample-application]: https://github.com/mspnp/multitenant-saas-guidance
[running-the-app]: ./run-the-app.md
