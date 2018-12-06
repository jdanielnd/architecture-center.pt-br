---
title: Autorização em aplicativos multilocatário
description: Como executar a autorização em um aplicativo multilocatário
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: bbf702fe6651625a1aeceff7e4e321dd08c38544
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902486"
---
# <a name="role-based-and-resource-based-authorization"></a>Autorização baseada em funções e recursos

[Código de exemplo do ![GitHub](../_images/github.png)][sample application]

Nossa [implementação de referência] é um aplicativo ASP.NET Core. Neste artigo, examinaremos duas abordagens gerais para autorização, usando a APIs fornecidas no ASP.NET Core.

* **Autorização baseada em função**. Autorização de uma ação baseada nas funções atribuídas a um usuário. Por exemplo, algumas ações requerem uma função de administrador.
* **Autorização baseada em recursos**. Autorização de uma ação baseada em determinado recurso. Por exemplo, cada recurso tem um proprietário. O proprietário pode excluir o recurso, mas os outros usuários não.

Um aplicativo típico utiliza uma mistura das duas abordagens. Por exemplo, para excluir um recurso, o usuário deve ser o proprietário do recurso *ou* um administrador.

## <a name="role-based-authorization"></a>Autorização baseada em função
Por exemplo, o aplicativo [Tailspin Surveys][Tailspin] define as seguintes funções:

* Administrador. Pode executar todas as operações CRUD em qualquer pesquisa que pertença ao locatário.
* Criador. Pode criar novas pesquisas
* Leitor. Pode ler todas as pesquisas que pertençam ao locatário

As funções se aplicam a *usuários* do aplicativo. No aplicativo Surveys, o usuário pode ser um administrador, um criador ou um leitor.

Para uma discussão sobre como definir e gerenciar funções, confira [Funções de aplicativo].

Independentemente de como você gerenciar as funções, seu código de autorização terá uma aparência semelhante. O ASP.NET Core tem uma abstração chamada [políticas de autorização][policies]. Com esse recurso, você define políticas de autorização no código e depois aplica-as às ações do controlador. A política é dissociada do controlador.

### <a name="create-policies"></a>Criar políticas
Para definir uma política, primeiro crie uma classe que implemente `IAuthorizationRequirement`. É mais fácil derivar de `AuthorizationHandler`. No método `Handle` , examine as solicitações relevantes.

Veja um exemplo do aplicativo Tailspin Surveys:

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

Essa classe define o requisito para um usuário criar uma nova pesquisa. O usuário deve ter a função SurveyAdmin ou SurveyCreator.

Em sua classe de inicialização, defina uma política nomeada que inclua um ou mais requisitos. Caso existam vários requisitos, o usuário deve atender a *todos* os requisitos para ser autorizado. O código abaixo define duas políticas:

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

Esse código também define o esquema de autenticação, que diz ao ASP.NET qual middleware de autenticação deverá ser executado se a autorização falhar. Nesse caso, especificamos o middleware de autenticação de cookie, porque o middleware de autenticação de cookie pode redirecionar o usuário para uma página "Proibida". O local da página Proibida é definido na opção `AccessDeniedPath` para o middleware de cookie; consulte [Configurando o middleware de autenticação].

### <a name="authorize-controller-actions"></a>Autorizar ações do controlador
Por fim, para autorizar uma ação em um controlador MVC, defina a política no atributo `Authorize` :

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

Em versões anteriores do ASP.NET, você definiria a propriedade **Funções** do atributo:

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

Ainda existe suporte para isso no ASP.NET Core, mas existem algumas desvantagens em comparação a políticas de autorização:

* Ela pressupõe um tipo de declaração específico. As políticas podem verificar qualquer tipo de declaração. As funções são apenas um tipo de declaração.
* O nome da função é embutido em código no atributo. Com as políticas, a lógica da autorização está em um só lugar, tornando mais fácil atualizar ou, até mesmo, carregar as definições de configuração.
* As políticas permitem decisões de autorização mais complexas (por exemplo, idade maior ou igual a 21), que não podem ser expressas pela associação de função simples.

## <a name="resource-based-authorization"></a>Autorização baseada em recursos
*Autorização baseada em recursos* ocorre sempre que a autorização depender de um recurso específico que será afetado por uma operação. No aplicativo Tailspin Surveys, cada pesquisa tem um proprietário e de zero a muitos colaboradores.

* O proprietário pode ler, atualizar, excluir, publicar e cancelar a publicação da pesquisa.
* O proprietário pode atribuir colaboradores à pesquisa.
* Os colaboradores podem ler e atualizar a pesquisa.

Observe que "proprietário" e "colaborador" não são funções de aplicativo; são armazenados por pesquisa, no banco de dados do aplicativo. Para verificar se um usuário pode excluir uma pesquisa, por exemplo, o aplicativo verifica se o usuário é o proprietário daquela pesquisa.

No ASP.NET Core, implemente a autorização baseada em recursos derivando de **AuthorizationHandler** e substituindo o método **Handle**.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

Observe que essa classe é fortemente tipada para objetos do Survey.  Registre a classe de injeção de dependência na inicialização:

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

Para executar verificações de autorização, use a interface **IAuthorizationService** , que você pode injetar em seus controladores. O código a seguir verifica se um usuário pode ler uma pesquisa:

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

Uma vez que passamos um objeto `Survey`, essa chamada invocará o `SurveyAuthorizationHandler`.

No código de autorização, uma boa abordagem é agregar todas as permissões do usuário baseadas em função e baseadas em recursos e depois verificar a agregação definida em relação à operação desejada.
Veja um exemplo do aplicativo Surveys. O aplicativo define vários tipos de permissão:

* Administrador
* Colaborador
* Criador
* Proprietário
* Leitor

O aplicativo também define um conjunto de possíveis operações nas pesquisas:

* Criar
* Ler
* Atualização
* Excluir
* Publicar
* Não publicar

O código a seguir cria uma lista de permissões para uma pesquisa e usuário específicos. Observe que esse código examina as funções de aplicativo do usuário e os campos de proprietário/colaborador da pesquisa.

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

Em um aplicativo multilocatário, você deve garantir que as permissões não "vazem" para os dados de outro locatário. No aplicativo de Pesquisas, a permissão de Colaborador é permitida entre os locatários &mdash;. Você pode atribuir alguém de outro locatário como um colaborador. Os outros tipos de permissão são restritos a recursos que pertencem ao locatário do usuário. Para impor a esse requisito, o código verifica a ID de locatário antes de conceder a permissão. (O campo `TenantId` é atribuído quando a pesquisa é criada.)

A próxima etapa é verificar a operação (leitura, atualização, exclusão, etc.) em relação às permissões. O aplicativo Surveys implementa esta etapa usando uma tabela de pesquisa de funções:

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

[**Avançar**][web-api]

<!-- Links -->
[Tailspin]: tailspin.md

[Funções de aplicativo]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[implementação de referência]: tailspin.md
[Configurando o middleware de autenticação]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
