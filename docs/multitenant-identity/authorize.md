---
title: Autorização em aplicativos multilocatário
description: Como executar a autorização em um aplicativo multilocatário.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 1238f1cd3bc8e6f5f3d174ea5fa0c98b4da795f3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488606"
---
# <a name="role-based-and-resource-based-authorization"></a><span data-ttu-id="d395c-103">Autorização baseada em funções e recursos</span><span class="sxs-lookup"><span data-stu-id="d395c-103">Role-based and resource-based authorization</span></span>

<span data-ttu-id="d395c-104">[Código de exemplo do ![GitHub](../_images/github.png)][sample application]</span><span class="sxs-lookup"><span data-stu-id="d395c-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="d395c-105">Nossa [implementação de referência] é um aplicativo ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="d395c-105">Our [reference implementation] is an ASP.NET Core application.</span></span> <span data-ttu-id="d395c-106">Neste artigo, examinaremos duas abordagens gerais para autorização, usando a APIs fornecidas no ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="d395c-106">In this article we'll look at two general approaches to authorization, using the authorization APIs provided in ASP.NET Core.</span></span>

* <span data-ttu-id="d395c-107">**Autorização baseada em função**.</span><span class="sxs-lookup"><span data-stu-id="d395c-107">**Role-based authorization**.</span></span> <span data-ttu-id="d395c-108">Autorização de uma ação baseada nas funções atribuídas a um usuário.</span><span class="sxs-lookup"><span data-stu-id="d395c-108">Authorizing an action based on the roles assigned to a user.</span></span> <span data-ttu-id="d395c-109">Por exemplo, algumas ações requerem uma função de administrador.</span><span class="sxs-lookup"><span data-stu-id="d395c-109">For example, some actions require an administrator role.</span></span>
* <span data-ttu-id="d395c-110">**Autorização baseada em recursos**.</span><span class="sxs-lookup"><span data-stu-id="d395c-110">**Resource-based authorization**.</span></span> <span data-ttu-id="d395c-111">Autorização de uma ação baseada em determinado recurso.</span><span class="sxs-lookup"><span data-stu-id="d395c-111">Authorizing an action based on a particular resource.</span></span> <span data-ttu-id="d395c-112">Por exemplo, cada recurso tem um proprietário.</span><span class="sxs-lookup"><span data-stu-id="d395c-112">For example, every resource has an owner.</span></span> <span data-ttu-id="d395c-113">O proprietário pode excluir o recurso, mas os outros usuários não.</span><span class="sxs-lookup"><span data-stu-id="d395c-113">The owner can delete the resource; other users cannot.</span></span>

<span data-ttu-id="d395c-114">Um aplicativo típico utiliza uma mistura das duas abordagens.</span><span class="sxs-lookup"><span data-stu-id="d395c-114">A typical app will employ a mix of both.</span></span> <span data-ttu-id="d395c-115">Por exemplo, para excluir um recurso, o usuário deve ser o proprietário do recurso *ou* um administrador.</span><span class="sxs-lookup"><span data-stu-id="d395c-115">For example, to delete a resource, the user must be the resource owner *or* an admin.</span></span>

## <a name="role-based-authorization"></a><span data-ttu-id="d395c-116">Autorização baseada em função</span><span class="sxs-lookup"><span data-stu-id="d395c-116">Role-Based Authorization</span></span>

<span data-ttu-id="d395c-117">Por exemplo, o aplicativo [Tailspin Surveys][Tailspin] define as seguintes funções:</span><span class="sxs-lookup"><span data-stu-id="d395c-117">The [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="d395c-118">Administrador.</span><span class="sxs-lookup"><span data-stu-id="d395c-118">Administrator.</span></span> <span data-ttu-id="d395c-119">Pode executar todas as operações CRUD em qualquer pesquisa que pertença ao locatário.</span><span class="sxs-lookup"><span data-stu-id="d395c-119">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="d395c-120">Criador.</span><span class="sxs-lookup"><span data-stu-id="d395c-120">Creator.</span></span> <span data-ttu-id="d395c-121">Pode criar novas pesquisas</span><span class="sxs-lookup"><span data-stu-id="d395c-121">Can create new surveys</span></span>
* <span data-ttu-id="d395c-122">Leitor.</span><span class="sxs-lookup"><span data-stu-id="d395c-122">Reader.</span></span> <span data-ttu-id="d395c-123">Pode ler todas as pesquisas que pertençam ao locatário</span><span class="sxs-lookup"><span data-stu-id="d395c-123">Can read any surveys that belong to that tenant</span></span>

<span data-ttu-id="d395c-124">As funções se aplicam a *usuários* do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d395c-124">Roles apply to *users* of the application.</span></span> <span data-ttu-id="d395c-125">No aplicativo Surveys, o usuário pode ser um administrador, um criador ou um leitor.</span><span class="sxs-lookup"><span data-stu-id="d395c-125">In the Surveys application, a user is either an administrator, creator, or reader.</span></span>

<span data-ttu-id="d395c-126">Para uma discussão sobre como definir e gerenciar funções, confira [Funções de aplicativo].</span><span class="sxs-lookup"><span data-stu-id="d395c-126">For a discussion of how to define and manage roles, see [Application roles].</span></span>

<span data-ttu-id="d395c-127">Independentemente de como você gerenciar as funções, seu código de autorização terá uma aparência semelhante.</span><span class="sxs-lookup"><span data-stu-id="d395c-127">Regardless of how you manage the roles, your authorization code will look similar.</span></span> <span data-ttu-id="d395c-128">O ASP.NET Core tem uma abstração chamada [políticas de autorização][policies].</span><span class="sxs-lookup"><span data-stu-id="d395c-128">ASP.NET Core has an abstraction called [authorization policies][policies].</span></span> <span data-ttu-id="d395c-129">Com esse recurso, você define políticas de autorização no código e depois aplica-as às ações do controlador.</span><span class="sxs-lookup"><span data-stu-id="d395c-129">With this feature, you define authorization policies in code, and then apply those policies to controller actions.</span></span> <span data-ttu-id="d395c-130">A política é dissociada do controlador.</span><span class="sxs-lookup"><span data-stu-id="d395c-130">The policy is decoupled from the controller.</span></span>

### <a name="create-policies"></a><span data-ttu-id="d395c-131">Criar políticas</span><span class="sxs-lookup"><span data-stu-id="d395c-131">Create policies</span></span>

<span data-ttu-id="d395c-132">Para definir uma política, primeiro crie uma classe que implemente `IAuthorizationRequirement`.</span><span class="sxs-lookup"><span data-stu-id="d395c-132">To define a policy, first create a class that implements `IAuthorizationRequirement`.</span></span> <span data-ttu-id="d395c-133">É mais fácil derivar de `AuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="d395c-133">It's easiest to derive from `AuthorizationHandler`.</span></span> <span data-ttu-id="d395c-134">No método `Handle` , examine as solicitações relevantes.</span><span class="sxs-lookup"><span data-stu-id="d395c-134">In the `Handle` method, examine the relevant claim(s).</span></span>

<span data-ttu-id="d395c-135">Veja um exemplo do aplicativo Tailspin Surveys:</span><span class="sxs-lookup"><span data-stu-id="d395c-135">Here is an example from the Tailspin Surveys application:</span></span>

```csharp
public class SurveyCreatorRequirement : AuthorizationHandler<SurveyCreatorRequirement>, IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, SurveyCreatorRequirement requirement)
    {
        if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin) ||
            context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

<span data-ttu-id="d395c-136">Essa classe define o requisito para um usuário criar uma nova pesquisa.</span><span class="sxs-lookup"><span data-stu-id="d395c-136">This class defines the requirement for a user to create a new survey.</span></span> <span data-ttu-id="d395c-137">O usuário deve ter a função SurveyAdmin ou SurveyCreator.</span><span class="sxs-lookup"><span data-stu-id="d395c-137">The user must be in the SurveyAdmin or SurveyCreator role.</span></span>

<span data-ttu-id="d395c-138">Em sua classe de inicialização, defina uma política nomeada que inclua um ou mais requisitos.</span><span class="sxs-lookup"><span data-stu-id="d395c-138">In your startup class, define a named policy that includes one or more requirements.</span></span> <span data-ttu-id="d395c-139">Caso existam vários requisitos, o usuário deve atender a *todos* os requisitos para ser autorizado.</span><span class="sxs-lookup"><span data-stu-id="d395c-139">If there are multiple requirements, the user must meet *every* requirement to be authorized.</span></span> <span data-ttu-id="d395c-140">O código abaixo define duas políticas:</span><span class="sxs-lookup"><span data-stu-id="d395c-140">The following code defines two policies:</span></span>

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy(PolicyNames.RequireSurveyCreator,
        policy =>
        {
            policy.AddRequirements(new SurveyCreatorRequirement());
            policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement
            // By adding the CookieAuthenticationDefaults.AuthenticationScheme, if an authenticated
            // user is not in the appropriate role, they will be redirected to a "forbidden" page.
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });

    options.AddPolicy(PolicyNames.RequireSurveyAdmin,
        policy =>
        {
            policy.AddRequirements(new SurveyAdminRequirement());
            policy.RequireAuthenticatedUser();  
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });
});
```

<span data-ttu-id="d395c-141">Esse código também define o esquema de autenticação, que diz ao ASP.NET qual middleware de autenticação deverá ser executado se a autorização falhar.</span><span class="sxs-lookup"><span data-stu-id="d395c-141">This code also sets the authentication scheme, which tells ASP.NET which authentication middleware should run if authorization fails.</span></span> <span data-ttu-id="d395c-142">Nesse caso, especificamos o middleware de autenticação de cookie, porque o middleware de autenticação de cookie pode redirecionar o usuário para uma página "Proibida".</span><span class="sxs-lookup"><span data-stu-id="d395c-142">In this case, we specify the cookie authentication middleware, because the cookie authentication middleware can redirect the user to a "Forbidden" page.</span></span> <span data-ttu-id="d395c-143">O local da página Proibida é definido na opção `AccessDeniedPath` para o middleware de cookie; consulte [Configurando o middleware de autenticação].</span><span class="sxs-lookup"><span data-stu-id="d395c-143">The location of the Forbidden page is set in the `AccessDeniedPath` option for the cookie middleware; see [Configuring the authentication middleware].</span></span>

### <a name="authorize-controller-actions"></a><span data-ttu-id="d395c-144">Autorizar ações do controlador</span><span class="sxs-lookup"><span data-stu-id="d395c-144">Authorize controller actions</span></span>

<span data-ttu-id="d395c-145">Por fim, para autorizar uma ação em um controlador MVC, defina a política no atributo `Authorize` :</span><span class="sxs-lookup"><span data-stu-id="d395c-145">Finally, to authorize an action in an MVC controller, set the policy in the `Authorize` attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

<span data-ttu-id="d395c-146">Em versões anteriores do ASP.NET, você definiria a propriedade **Funções** do atributo:</span><span class="sxs-lookup"><span data-stu-id="d395c-146">In earlier versions of ASP.NET, you would set the **Roles** property on the attribute:</span></span>

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

<span data-ttu-id="d395c-147">Ainda existe suporte para isso no ASP.NET Core, mas existem algumas desvantagens em comparação a políticas de autorização:</span><span class="sxs-lookup"><span data-stu-id="d395c-147">This is still supported in ASP.NET Core, but it has some drawbacks compared with authorization policies:</span></span>

* <span data-ttu-id="d395c-148">Ela pressupõe um tipo de declaração específico.</span><span class="sxs-lookup"><span data-stu-id="d395c-148">It assumes a particular claim type.</span></span> <span data-ttu-id="d395c-149">As políticas podem verificar qualquer tipo de declaração.</span><span class="sxs-lookup"><span data-stu-id="d395c-149">Policies can check for any claim type.</span></span> <span data-ttu-id="d395c-150">As funções são apenas um tipo de declaração.</span><span class="sxs-lookup"><span data-stu-id="d395c-150">Roles are just a type of claim.</span></span>
* <span data-ttu-id="d395c-151">O nome da função é embutido em código no atributo.</span><span class="sxs-lookup"><span data-stu-id="d395c-151">The role name is hard-coded into the attribute.</span></span> <span data-ttu-id="d395c-152">Com as políticas, a lógica da autorização está em um só lugar, tornando mais fácil atualizar ou, até mesmo, carregar as definições de configuração.</span><span class="sxs-lookup"><span data-stu-id="d395c-152">With policies, the authorization logic is all in one place, making it easier to update or even load from configuration settings.</span></span>
* <span data-ttu-id="d395c-153">As políticas permitem decisões de autorização mais complexas (por exemplo, idade maior ou igual a 21), que não podem ser expressas pela associação de função simples.</span><span class="sxs-lookup"><span data-stu-id="d395c-153">Policies enable more complex authorization decisions (e.g., age >= 21) that can't be expressed by simple role membership.</span></span>

## <a name="resource-based-authorization"></a><span data-ttu-id="d395c-154">Autorização baseada em recursos</span><span class="sxs-lookup"><span data-stu-id="d395c-154">Resource based authorization</span></span>

<span data-ttu-id="d395c-155">*Autorização baseada em recursos* ocorre sempre que a autorização depender de um recurso específico que será afetado por uma operação.</span><span class="sxs-lookup"><span data-stu-id="d395c-155">*Resource based authorization* occurs whenever the authorization depends on a specific resource that will be affected by an operation.</span></span> <span data-ttu-id="d395c-156">No aplicativo Tailspin Surveys, cada pesquisa tem um proprietário e de zero a muitos colaboradores.</span><span class="sxs-lookup"><span data-stu-id="d395c-156">In the Tailspin Surveys application, every survey has an owner and zero-to-many contributors.</span></span>

* <span data-ttu-id="d395c-157">O proprietário pode ler, atualizar, excluir, publicar e cancelar a publicação da pesquisa.</span><span class="sxs-lookup"><span data-stu-id="d395c-157">The owner can read, update, delete, publish, and unpublish the survey.</span></span>
* <span data-ttu-id="d395c-158">O proprietário pode atribuir colaboradores à pesquisa.</span><span class="sxs-lookup"><span data-stu-id="d395c-158">The owner can assign contributors to the survey.</span></span>
* <span data-ttu-id="d395c-159">Os colaboradores podem ler e atualizar a pesquisa.</span><span class="sxs-lookup"><span data-stu-id="d395c-159">Contributors can read and update the survey.</span></span>

<span data-ttu-id="d395c-160">Observe que "proprietário" e "colaborador" não são funções de aplicativo; são armazenados por pesquisa, no banco de dados do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d395c-160">Note that "owner" and "contributor" are not application roles; they are stored per survey, in the application database.</span></span> <span data-ttu-id="d395c-161">Para verificar se um usuário pode excluir uma pesquisa, por exemplo, o aplicativo verifica se o usuário é o proprietário daquela pesquisa.</span><span class="sxs-lookup"><span data-stu-id="d395c-161">To check whether a user can delete a survey, for example, the app checks whether the user is the owner for that survey.</span></span>

<span data-ttu-id="d395c-162">No ASP.NET Core, implemente a autorização baseada em recursos derivando de **AuthorizationHandler** e substituindo o método **Handle**.</span><span class="sxs-lookup"><span data-stu-id="d395c-162">In ASP.NET Core, implement resource-based authorization by deriving from **AuthorizationHandler** and overriding the **Handle** method.</span></span>

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

<span data-ttu-id="d395c-163">Observe que essa classe é fortemente tipada para objetos do Survey.</span><span class="sxs-lookup"><span data-stu-id="d395c-163">Notice that this class is strongly typed for Survey objects.</span></span>  <span data-ttu-id="d395c-164">Registre a classe de injeção de dependência na inicialização:</span><span class="sxs-lookup"><span data-stu-id="d395c-164">Register the class for DI on startup:</span></span>

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

<span data-ttu-id="d395c-165">Para executar verificações de autorização, use a interface **IAuthorizationService** , que você pode injetar em seus controladores.</span><span class="sxs-lookup"><span data-stu-id="d395c-165">To perform authorization checks, use the **IAuthorizationService** interface, which you can inject into your controllers.</span></span> <span data-ttu-id="d395c-166">O código a seguir verifica se um usuário pode ler uma pesquisa:</span><span class="sxs-lookup"><span data-stu-id="d395c-166">The following code checks whether a user can read a survey:</span></span>

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

<span data-ttu-id="d395c-167">Uma vez que passamos um objeto `Survey`, essa chamada invocará o `SurveyAuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="d395c-167">Because we pass in a `Survey` object, this call will invoke the `SurveyAuthorizationHandler`.</span></span>

<span data-ttu-id="d395c-168">No código de autorização, uma boa abordagem é agregar todas as permissões do usuário baseadas em função e baseadas em recursos e depois verificar a agregação definida em relação à operação desejada.</span><span class="sxs-lookup"><span data-stu-id="d395c-168">In your authorization code, a good approach is to aggregate all of the user's role-based and resource-based permissions, then check the aggregate set against the desired operation.</span></span>
<span data-ttu-id="d395c-169">Veja um exemplo do aplicativo Surveys.</span><span class="sxs-lookup"><span data-stu-id="d395c-169">Here is an example from the Surveys app.</span></span> <span data-ttu-id="d395c-170">O aplicativo define vários tipos de permissão:</span><span class="sxs-lookup"><span data-stu-id="d395c-170">The application defines several permission types:</span></span>

* <span data-ttu-id="d395c-171">Administrador</span><span class="sxs-lookup"><span data-stu-id="d395c-171">Admin</span></span>
* <span data-ttu-id="d395c-172">Colaborador</span><span class="sxs-lookup"><span data-stu-id="d395c-172">Contributor</span></span>
* <span data-ttu-id="d395c-173">Criador</span><span class="sxs-lookup"><span data-stu-id="d395c-173">Creator</span></span>
* <span data-ttu-id="d395c-174">Proprietário</span><span class="sxs-lookup"><span data-stu-id="d395c-174">Owner</span></span>
* <span data-ttu-id="d395c-175">Leitor</span><span class="sxs-lookup"><span data-stu-id="d395c-175">Reader</span></span>

<span data-ttu-id="d395c-176">O aplicativo também define um conjunto de possíveis operações nas pesquisas:</span><span class="sxs-lookup"><span data-stu-id="d395c-176">The application also defines a set of possible operations on surveys:</span></span>

* <span data-ttu-id="d395c-177">Criar</span><span class="sxs-lookup"><span data-stu-id="d395c-177">Create</span></span>
* <span data-ttu-id="d395c-178">Ler</span><span class="sxs-lookup"><span data-stu-id="d395c-178">Read</span></span>
* <span data-ttu-id="d395c-179">Atualização</span><span class="sxs-lookup"><span data-stu-id="d395c-179">Update</span></span>
* <span data-ttu-id="d395c-180">Excluir</span><span class="sxs-lookup"><span data-stu-id="d395c-180">Delete</span></span>
* <span data-ttu-id="d395c-181">Publicar</span><span class="sxs-lookup"><span data-stu-id="d395c-181">Publish</span></span>
* <span data-ttu-id="d395c-182">Cancelar Publicação</span><span class="sxs-lookup"><span data-stu-id="d395c-182">Unpublish</span></span>

<span data-ttu-id="d395c-183">O código a seguir cria uma lista de permissões para uma pesquisa e usuário específicos.</span><span class="sxs-lookup"><span data-stu-id="d395c-183">The following code creates a list of permissions for a particular user and survey.</span></span> <span data-ttu-id="d395c-184">Observe que esse código examina as funções de aplicativo do usuário e os campos de proprietário/colaborador da pesquisa.</span><span class="sxs-lookup"><span data-stu-id="d395c-184">Notice that this code looks at both the user's app roles, and the owner/contributor fields in the survey.</span></span>

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement requirement, Survey resource)
    {
        var permissions = new List<UserPermissionType>();
        int surveyTenantId = context.User.GetSurveyTenantIdValue();
        int userId = context.User.GetSurveyUserIdValue();
        string user = context.User.GetUserName();

        if (resource.TenantId == surveyTenantId)
        {
            // Admin can do anything, as long as the resource belongs to the admin's tenant.
            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin))
            {
                context.Succeed(requirement);
                return Task.FromResult(0);
            }

            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
            {
                permissions.Add(UserPermissionType.Creator);
            }
            else
            {
                permissions.Add(UserPermissionType.Reader);
            }

            if (resource.OwnerId == userId)
            {
                permissions.Add(UserPermissionType.Owner);
            }
        }
        if (resource.Contributors != null && resource.Contributors.Any(x => x.UserId == userId))
        {
            permissions.Add(UserPermissionType.Contributor);
        }

        if (ValidateUserPermissions[requirement](permissions))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

<span data-ttu-id="d395c-185">Em um aplicativo multilocatário, você deve garantir que as permissões não "vazem" para os dados de outro locatário.</span><span class="sxs-lookup"><span data-stu-id="d395c-185">In a multi-tenant application, you must ensure that permissions don't "leak" to another tenant's data.</span></span> <span data-ttu-id="d395c-186">No aplicativo de Pesquisas, a permissão de Colaborador é permitida entre os locatários &mdash;. Você pode atribuir alguém de outro locatário como um colaborador.</span><span class="sxs-lookup"><span data-stu-id="d395c-186">In the Surveys app, the Contributor permission is allowed across tenants &mdash; you can assign someone from another tenant as a contributor.</span></span> <span data-ttu-id="d395c-187">Os outros tipos de permissão são restritos a recursos que pertencem ao locatário do usuário.</span><span class="sxs-lookup"><span data-stu-id="d395c-187">The other permission types are restricted to resources that belong to that user's tenant.</span></span> <span data-ttu-id="d395c-188">Para impor a esse requisito, o código verifica a ID de locatário antes de conceder a permissão.</span><span class="sxs-lookup"><span data-stu-id="d395c-188">To enforce this requirement, the code checks the tenant ID before granting the permission.</span></span> <span data-ttu-id="d395c-189">(O campo `TenantId` é atribuído quando a pesquisa é criada.)</span><span class="sxs-lookup"><span data-stu-id="d395c-189">(The `TenantId` field as assigned when the survey is created.)</span></span>

<span data-ttu-id="d395c-190">A próxima etapa é verificar a operação (leitura, atualização, exclusão, etc.) em relação às permissões.</span><span class="sxs-lookup"><span data-stu-id="d395c-190">The next step is to check the operation (read, update, delete, etc) against the permissions.</span></span> <span data-ttu-id="d395c-191">O aplicativo Surveys implementa esta etapa usando uma tabela de pesquisa de funções:</span><span class="sxs-lookup"><span data-stu-id="d395c-191">The Surveys app implements this step by using a lookup table of functions:</span></span>

```csharp
static readonly Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>> ValidateUserPermissions
    = new Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>>

    {
        { Operations.Create, x => x.Contains(UserPermissionType.Creator) },

        { Operations.Read, x => x.Contains(UserPermissionType.Creator) ||
                                x.Contains(UserPermissionType.Reader) ||
                                x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Update, x => x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Delete, x => x.Contains(UserPermissionType.Owner) },

        { Operations.Publish, x => x.Contains(UserPermissionType.Owner) },

        { Operations.UnPublish, x => x.Contains(UserPermissionType.Owner) }
    };
```

<span data-ttu-id="d395c-192">[**Avançar**][web-api]</span><span class="sxs-lookup"><span data-stu-id="d395c-192">[**Next**][web-api]</span></span>

<!-- links -->

[Tailspin]: tailspin.md

[Funções de aplicativo]: app-roles.md
[Application roles]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[implementação de referência]: tailspin.md
[reference implementation]: tailspin.md
[Configurando o middleware de autenticação]: authenticate.md#configure-the-auth-middleware
[Configuring the authentication middleware]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
