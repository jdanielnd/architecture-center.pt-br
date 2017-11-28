---
title: "Proteger uma API Web de back-end em um aplicativo multilocatário"
description: Como proteger uma API Web de back-end
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authorize
pnp.series.next: token-cache
ms.openlocfilehash: 65529280c5849e36ed7ff23de08a0b485034d0d8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="secure-a-backend-web-api"></a><span data-ttu-id="3fab4-103">Proteger uma API Web de back-end</span><span class="sxs-lookup"><span data-stu-id="3fab4-103">Secure a backend web API</span></span>

<span data-ttu-id="3fab4-104">[Código de exemplo do ![GitHub](../_images/github.png)][sample application]</span><span class="sxs-lookup"><span data-stu-id="3fab4-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="3fab4-105">O aplicativo [Tailspin Surveys] usa uma API Web de back-end para gerenciar operações CRUD em pesquisas.</span><span class="sxs-lookup"><span data-stu-id="3fab4-105">The [Tailspin Surveys] application uses a backend web API to manage CRUD operations on surveys.</span></span> <span data-ttu-id="3fab4-106">Por exemplo, quando um usuário clica em "Minhas Pesquisas", o aplicativo Web envia uma solicitação HTTP para a API Web:</span><span class="sxs-lookup"><span data-stu-id="3fab4-106">For example, when a user clicks "My Surveys", the web application sends an HTTP request to the web API:</span></span>

```
GET /users/{userId}/surveys
```

<span data-ttu-id="3fab4-107">A API Web retorna um objeto JSON:</span><span class="sxs-lookup"><span data-stu-id="3fab4-107">The web API returns a JSON object:</span></span>

```
{
  "Published":[],
  "Own":[
    {"Id":1,"Title":"Survey 1"},
    {"Id":3,"Title":"Survey 3"},
    ],
  "Contribute": [{"Id":8,"Title":"My survey"}]
}
```

<span data-ttu-id="3fab4-108">A API Web não permite solicitações anônimas, portanto, o aplicativo Web deve se autenticar usando tokens de portador OAuth 2.</span><span class="sxs-lookup"><span data-stu-id="3fab4-108">The web API does not allow anonymous requests, so the web app must authenticate itself using OAuth 2 bearer tokens.</span></span>

> [!NOTE]
> <span data-ttu-id="3fab4-109">Esse é um cenário de servidor a servidor.</span><span class="sxs-lookup"><span data-stu-id="3fab4-109">This is a server-to-server scenario.</span></span> <span data-ttu-id="3fab4-110">O aplicativo não faz chamadas AJAX à API a partir do cliente de navegador.</span><span class="sxs-lookup"><span data-stu-id="3fab4-110">The application does not make any AJAX calls to the API from the browser client.</span></span>
> 
> 

<span data-ttu-id="3fab4-111">Há duas abordagens principais que você pode usar:</span><span class="sxs-lookup"><span data-stu-id="3fab4-111">There are two main approaches you can take:</span></span>

* <span data-ttu-id="3fab4-112">Identidade de usuário delegado.</span><span class="sxs-lookup"><span data-stu-id="3fab4-112">Delegated user identity.</span></span> <span data-ttu-id="3fab4-113">O aplicativo Web autentica com a identidade do usuário.</span><span class="sxs-lookup"><span data-stu-id="3fab4-113">The web application authenticates with the user's identity.</span></span>
* <span data-ttu-id="3fab4-114">Identidade do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="3fab4-114">Application identity.</span></span> <span data-ttu-id="3fab4-115">O aplicativo Web autentica com sua ID de cliente usando o fluxo de credencial de cliente OAuth2.</span><span class="sxs-lookup"><span data-stu-id="3fab4-115">The web application authenticates with its client ID, using OAuth2 client credential flow.</span></span>

<span data-ttu-id="3fab4-116">O aplicativo Tailspin implementa a identidade de usuário delegado.</span><span class="sxs-lookup"><span data-stu-id="3fab4-116">The Tailspin application implements delegated user identity.</span></span> <span data-ttu-id="3fab4-117">Aqui estão as diferenças principais:</span><span class="sxs-lookup"><span data-stu-id="3fab4-117">Here are the main differences:</span></span>

<span data-ttu-id="3fab4-118">**Identidade de usuário delegado**</span><span class="sxs-lookup"><span data-stu-id="3fab4-118">**Delegated user identity**</span></span>

* <span data-ttu-id="3fab4-119">O token de portador enviado para a API Web contém a identidade do usuário.</span><span class="sxs-lookup"><span data-stu-id="3fab4-119">The bearer token sent to the web API contains the user identity.</span></span>
* <span data-ttu-id="3fab4-120">A API Web toma decisões de autorização baseadas na identidade do usuário.</span><span class="sxs-lookup"><span data-stu-id="3fab4-120">The web API makes authorization decisions based on the user identity.</span></span>
* <span data-ttu-id="3fab4-121">O aplicativo Web precisará lidar com erros 403 (Proibido) da API Web se o usuário não estiver autorizado a executar uma ação.</span><span class="sxs-lookup"><span data-stu-id="3fab4-121">The web application needs to handle 403 (Forbidden) errors from the web API, if the user is not authorized to perform an action.</span></span>
* <span data-ttu-id="3fab4-122">Normalmente, o aplicativo Web ainda toma algumas decisões de autorização que afetam a interface do usuário, como mostrar ou ocultar elementos da interface do usuário.</span><span class="sxs-lookup"><span data-stu-id="3fab4-122">Typically, the web application still makes some authorization decisions that affect UI, such as showing or hiding UI elements).</span></span>
* <span data-ttu-id="3fab4-123">A API Web pode vir a ser usada por clientes não confiáveis, como um aplicativo JavaScript ou um aplicativo cliente nativo.</span><span class="sxs-lookup"><span data-stu-id="3fab4-123">The web API can potentially be used by untrusted clients, such as a JavaScript application or a native client application.</span></span>

<span data-ttu-id="3fab4-124">**Identidade do aplicativo**</span><span class="sxs-lookup"><span data-stu-id="3fab4-124">**Application identity**</span></span>

* <span data-ttu-id="3fab4-125">A API Web não obtém informações sobre o usuário.</span><span class="sxs-lookup"><span data-stu-id="3fab4-125">The web API does not get information about the user.</span></span>
* <span data-ttu-id="3fab4-126">A API Web não pode executar autorizações baseadas na identidade do usuário.</span><span class="sxs-lookup"><span data-stu-id="3fab4-126">The web API cannot perform any authorization based on the user identity.</span></span> <span data-ttu-id="3fab4-127">Todas as decisões de autorização são feitas pelo aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="3fab4-127">All authorization decisions are made by the web application.</span></span>  
* <span data-ttu-id="3fab4-128">A API Web não pode ser usada por um cliente não confiável (JavaScript ou aplicativo cliente nativo).</span><span class="sxs-lookup"><span data-stu-id="3fab4-128">The web API cannot be used by an untrusted client (JavaScript or native client application).</span></span>
* <span data-ttu-id="3fab4-129">Essa abordagem pode ser um pouco mais simples de implementar já que não há lógicas de autorização na API Web.</span><span class="sxs-lookup"><span data-stu-id="3fab4-129">This approach may be somewhat simpler to implement, because there is no authorization logic in the Web API.</span></span>

<span data-ttu-id="3fab4-130">Seja qual for a abordagem, o aplicativo Web deve obter um token de acesso que é a credencial necessária para chamar a API Web.</span><span class="sxs-lookup"><span data-stu-id="3fab4-130">In either approach, the web application must get an access token, which is the credential needed to call the web API.</span></span>

* <span data-ttu-id="3fab4-131">No caso da identidade de usuário delegado, o token tem de vir do IDP, que pode emitir um token em nome do usuário.</span><span class="sxs-lookup"><span data-stu-id="3fab4-131">For delegated user identity, the token has to come from the IDP, which can issue a token on behalf of the user.</span></span>
* <span data-ttu-id="3fab4-132">No caso das credenciais do cliente, um aplicativo pode obter o token do IDP ou hospedar seu próprio servidor de tokens.</span><span class="sxs-lookup"><span data-stu-id="3fab4-132">For client credentials, an application might get the token from the IDP or host its own token server.</span></span> <span data-ttu-id="3fab4-133">(Mas não grave um servidor de token do zero; use uma estrutura bem testada, como [IdentityServer3].) Se você autenticar com o Azure AD, é altamente recomendado obter o token de acesso do Azure AD, mesmo com o fluxo de credenciais do cliente.</span><span class="sxs-lookup"><span data-stu-id="3fab4-133">(But don't write a token server from scratch; use a well-tested framework like [IdentityServer3].) If you authenticate with Azure AD, it's strongly recommended to get the access token from Azure AD, even with client credential flow.</span></span>

<span data-ttu-id="3fab4-134">O restante deste artigo pressupõe que o aplicativo esteja se autenticando com o Azure AD.</span><span class="sxs-lookup"><span data-stu-id="3fab4-134">The rest of this article assumes the application is authenticating with Azure AD.</span></span>

![Obtendo o token de acesso](./images/access-token.png)

## <a name="register-the-web-api-in-azure-ad"></a><span data-ttu-id="3fab4-136">Registrar a API Web no Azure AD</span><span class="sxs-lookup"><span data-stu-id="3fab4-136">Register the web API in Azure AD</span></span>
<span data-ttu-id="3fab4-137">Para que o Azure AD emita um token de portador para a API Web, serão necessárias algumas configurações.</span><span class="sxs-lookup"><span data-stu-id="3fab4-137">In order for Azure AD to issue a bearer token for the web API, you need to configure some things in Azure AD.</span></span>

1. <span data-ttu-id="3fab4-138">Registre a API Web no Azure AD.</span><span class="sxs-lookup"><span data-stu-id="3fab4-138">Register the web API in Azure AD.</span></span>

2. <span data-ttu-id="3fab4-139">Adicione a ID do cliente do aplicativo Web ao manifesto de aplicativo da API Web na propriedade `knownClientApplications` .</span><span class="sxs-lookup"><span data-stu-id="3fab4-139">Add the client ID of the web app to the web API application manifest, in the `knownClientApplications` property.</span></span> <span data-ttu-id="3fab4-140">Confira [Atualizar os manifestos do aplicativo].</span><span class="sxs-lookup"><span data-stu-id="3fab4-140">See [Update the application manifests].</span></span>

3. <span data-ttu-id="3fab4-141">Conceda permissão de aplicativo Web para chamar a API Web.</span><span class="sxs-lookup"><span data-stu-id="3fab4-141">Give the web application permission to call the web API.</span></span> <span data-ttu-id="3fab4-142">No Portal de Gerenciamento do Azure, você pode definir dois tipos de permissões: "Permissões de Aplicativo" para a identidade de aplicativo (fluxo da credencial de cliente) ou "Permissões Delegadas" para a identidade de usuário delegado.</span><span class="sxs-lookup"><span data-stu-id="3fab4-142">In the Azure Management Portal, you can set two types of permissions: "Application Permissions" for application identity (client credential flow), or "Delegated Permissions" for delegated user identity.</span></span>
   
   ![Permissões delegadas](./images/delegated-permissions.png)

## <a name="getting-an-access-token"></a><span data-ttu-id="3fab4-144">Obtendo um token de acesso</span><span class="sxs-lookup"><span data-stu-id="3fab4-144">Getting an access token</span></span>
<span data-ttu-id="3fab4-145">Antes de chamar a API Web, o aplicativo Web obtém um token de acesso do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="3fab4-145">Before calling the web API, the web application gets an access token from Azure AD.</span></span> <span data-ttu-id="3fab4-146">Em um aplicativo .NET, use a [Biblioteca de Autenticação do Azure AD (ADAL) para .NET][ADAL].</span><span class="sxs-lookup"><span data-stu-id="3fab4-146">In a .NET application, use the [Azure AD Authentication Library (ADAL) for .NET][ADAL].</span></span>

<span data-ttu-id="3fab4-147">No fluxo de código de autorização OAuth 2, o aplicativo troca um código de autorização por um token de acesso.</span><span class="sxs-lookup"><span data-stu-id="3fab4-147">In the OAuth 2 authorization code flow, the application exchanges an authorization code for an access token.</span></span> <span data-ttu-id="3fab4-148">O código a seguir usa ADAL para obter o token de acesso.</span><span class="sxs-lookup"><span data-stu-id="3fab4-148">The following code uses ADAL to get the access token.</span></span> <span data-ttu-id="3fab4-149">Esse código é chamado durante o evento `AuthorizationCodeReceived` .</span><span class="sxs-lookup"><span data-stu-id="3fab4-149">This code is called during the `AuthorizationCodeReceived` event.</span></span>

```csharp
// The OpenID Connect middleware sends this event when it gets the authorization code.   
public override async Task AuthorizationCodeReceived(AuthorizationCodeReceivedContext context)
{
    string authorizationCode = context.ProtocolMessage.Code;
    string authority = "https://login.microsoftonline.com/" + tenantID
    string resourceID = "https://tailspin.onmicrosoft.com/surveys.webapi" // App ID URI
    ClientCredential credential = new ClientCredential(clientId, clientSecret);

    AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
    AuthenticationResult authResult = await authContext.AcquireTokenByAuthorizationCodeAsync(
        authorizationCode, new Uri(redirectUri), credential, resourceID);

    // If successful, the token is in authResult.AccessToken
}
```

<span data-ttu-id="3fab4-150">Estes são os vários parâmetros necessários:</span><span class="sxs-lookup"><span data-stu-id="3fab4-150">Here are the various parameters that are needed:</span></span>

* <span data-ttu-id="3fab4-151">`authority`.</span><span class="sxs-lookup"><span data-stu-id="3fab4-151">`authority`.</span></span> <span data-ttu-id="3fab4-152">Derivado da ID do locatário do usuário conectado.</span><span class="sxs-lookup"><span data-stu-id="3fab4-152">Derived from the tenant ID of the signed in user.</span></span> <span data-ttu-id="3fab4-153">(Não a ID do locatário do provedor de SaaS)</span><span class="sxs-lookup"><span data-stu-id="3fab4-153">(Not the tenant ID of the SaaS provider)</span></span>  
* <span data-ttu-id="3fab4-154">`authorizationCode`.</span><span class="sxs-lookup"><span data-stu-id="3fab4-154">`authorizationCode`.</span></span> <span data-ttu-id="3fab4-155">O código de autenticação que você recebeu do IDP.</span><span class="sxs-lookup"><span data-stu-id="3fab4-155">the auth code that you got back from the IDP.</span></span>
* <span data-ttu-id="3fab4-156">`clientId`.</span><span class="sxs-lookup"><span data-stu-id="3fab4-156">`clientId`.</span></span> <span data-ttu-id="3fab4-157">A ID do cliente do aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="3fab4-157">The web application's client ID.</span></span>
* <span data-ttu-id="3fab4-158">`clientSecret`.</span><span class="sxs-lookup"><span data-stu-id="3fab4-158">`clientSecret`.</span></span> <span data-ttu-id="3fab4-159">O segredo do cliente do aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="3fab4-159">The web application's client secret.</span></span>
* <span data-ttu-id="3fab4-160">`redirectUri`.</span><span class="sxs-lookup"><span data-stu-id="3fab4-160">`redirectUri`.</span></span> <span data-ttu-id="3fab4-161">O URI de redirecionamento que você definiu para a conexão do OpenID.</span><span class="sxs-lookup"><span data-stu-id="3fab4-161">The redirect URI that you set for OpenID connect.</span></span> <span data-ttu-id="3fab4-162">Esse é o local onde o IDP retorna a chamada com o token.</span><span class="sxs-lookup"><span data-stu-id="3fab4-162">This is where the IDP calls back with the token.</span></span>
* <span data-ttu-id="3fab4-163">`resourceID`.</span><span class="sxs-lookup"><span data-stu-id="3fab4-163">`resourceID`.</span></span> <span data-ttu-id="3fab4-164">O URI da ID de aplicativo da API Web que você criou quando registrou a API Web no Azure AD</span><span class="sxs-lookup"><span data-stu-id="3fab4-164">The App ID URI of the web API, which you created when you registered the web API in Azure AD</span></span>
* <span data-ttu-id="3fab4-165">`tokenCache`.</span><span class="sxs-lookup"><span data-stu-id="3fab4-165">`tokenCache`.</span></span> <span data-ttu-id="3fab4-166">Um objeto que armazena os tokens de acesso em cache.</span><span class="sxs-lookup"><span data-stu-id="3fab4-166">An object that caches the access tokens.</span></span> <span data-ttu-id="3fab4-167">Confira [Colocação de tokens em cache].</span><span class="sxs-lookup"><span data-stu-id="3fab4-167">See [Token caching].</span></span>

<span data-ttu-id="3fab4-168">Se `AcquireTokenByAuthorizationCodeAsync` for bem-sucedido, a ADAL armazenará o token em cache.</span><span class="sxs-lookup"><span data-stu-id="3fab4-168">If `AcquireTokenByAuthorizationCodeAsync` succeeds, ADAL caches the token.</span></span> <span data-ttu-id="3fab4-169">Posteriormente, você pode obter o token do cache chamando AcquireTokenSilentAsync:</span><span class="sxs-lookup"><span data-stu-id="3fab4-169">Later, you can get the token from the cache by calling AcquireTokenSilentAsync:</span></span>

```csharp
AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
var result = await authContext.AcquireTokenSilentAsync(resourceID, credential, new UserIdentifier(userId, UserIdentifierType.UniqueId));
```

<span data-ttu-id="3fab4-170">em que `userId` é a ID de objeto do usuário, encontrada na declaração `http://schemas.microsoft.com/identity/claims/objectidentifier`.</span><span class="sxs-lookup"><span data-stu-id="3fab4-170">where `userId` is the user's object ID, which is found in the `http://schemas.microsoft.com/identity/claims/objectidentifier` claim.</span></span>

## <a name="using-the-access-token-to-call-the-web-api"></a><span data-ttu-id="3fab4-171">Usando o token de acesso para chamar a API Web</span><span class="sxs-lookup"><span data-stu-id="3fab4-171">Using the access token to call the web API</span></span>
<span data-ttu-id="3fab4-172">Quando já tiver o token, envie-o no cabeçalho Autorização das solicitações HTTP à API Web.</span><span class="sxs-lookup"><span data-stu-id="3fab4-172">Once you have the token, send it in the Authorization header of the HTTP requests to the web API.</span></span>

```
Authorization: Bearer xxxxxxxxxx
```

<span data-ttu-id="3fab4-173">O método de extensão do aplicativo Surveys a seguir define o cabeçalho Autorização em uma solicitação HTTP usando a classe **HttpClient** .</span><span class="sxs-lookup"><span data-stu-id="3fab4-173">The following extension method from the Surveys application sets the Authorization header on an HTTP request, using the **HttpClient** class.</span></span>

```csharp
public static async Task<HttpResponseMessage> SendRequestWithBearerTokenAsync(this HttpClient httpClient, HttpMethod method, string path, object requestBody, string accessToken, CancellationToken ct)
{
    var request = new HttpRequestMessage(method, path);
    if (requestBody != null)
    {
        var json = JsonConvert.SerializeObject(requestBody, Formatting.None);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        request.Content = content;
    }

    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
    request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

    var response = await httpClient.SendAsync(request, ct);
    return response;
}
```

## <a name="authenticating-in-the-web-api"></a><span data-ttu-id="3fab4-174">Autenticando na API Web</span><span class="sxs-lookup"><span data-stu-id="3fab4-174">Authenticating in the web API</span></span>
<span data-ttu-id="3fab4-175">A API Web precisa autenticar o token de portador.</span><span class="sxs-lookup"><span data-stu-id="3fab4-175">The web API has to authenticate the bearer token.</span></span> <span data-ttu-id="3fab4-176">No ASP.NET Core, você pode usar o pacote [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer].</span><span class="sxs-lookup"><span data-stu-id="3fab4-176">In ASP.NET Core, you can use the [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer] package.</span></span> <span data-ttu-id="3fab4-177">Esse pacote fornece o middleware que permite ao aplicativo receber tokens de portador do OpenID Connect.</span><span class="sxs-lookup"><span data-stu-id="3fab4-177">This package provides middleware that enables the application to receive OpenID Connect bearer tokens.</span></span>

<span data-ttu-id="3fab4-178">Registre o middleware na classe `Startup` de sua API Web.</span><span class="sxs-lookup"><span data-stu-id="3fab4-178">Register the middleware in your web API `Startup` class.</span></span>

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ApplicationDbContext dbContext, ILoggerFactory loggerFactory)
{
    // ...

    app.UseJwtBearerAuthentication(new JwtBearerOptions {
        Audience = configOptions.AzureAd.WebApiResourceId,
        Authority = Constants.AuthEndpointPrefix,
        TokenValidationParameters = new TokenValidationParameters {
            ValidateIssuer = false
        },
        Events= new SurveysJwtBearerEvents(loggerFactory.CreateLogger<SurveysJwtBearerEvents>())
    });
    
    // ...
}
```

* <span data-ttu-id="3fab4-179">**Público-alvo**.</span><span class="sxs-lookup"><span data-stu-id="3fab4-179">**Audience**.</span></span> <span data-ttu-id="3fab4-180">Defina como a URL da ID de aplicativo da API Web que você criou quando registrou a API Web no Azure AD.</span><span class="sxs-lookup"><span data-stu-id="3fab4-180">Set this to the App ID URL for the web API, which you created when you registered the web API with Azure AD.</span></span>
* <span data-ttu-id="3fab4-181">**Autoridade**.</span><span class="sxs-lookup"><span data-stu-id="3fab4-181">**Authority**.</span></span> <span data-ttu-id="3fab4-182">No caso do aplicativo de multilocatário, defina como `https://login.microsoftonline.com/common/`.</span><span class="sxs-lookup"><span data-stu-id="3fab4-182">For a multitenant application, set this to `https://login.microsoftonline.com/common/`.</span></span>
* <span data-ttu-id="3fab4-183">**TokenValidationParameters**.</span><span class="sxs-lookup"><span data-stu-id="3fab4-183">**TokenValidationParameters**.</span></span> <span data-ttu-id="3fab4-184">Para um aplicativo multilocatário, defina **ValidateIssuer** como falso.</span><span class="sxs-lookup"><span data-stu-id="3fab4-184">For a multitenant application, set **ValidateIssuer** to false.</span></span> <span data-ttu-id="3fab4-185">Isso significa que o aplicativo validará o emissor.</span><span class="sxs-lookup"><span data-stu-id="3fab4-185">That means the application will validate the issuer.</span></span>
* <span data-ttu-id="3fab4-186">**Eventos** é uma classe que deriva de **JwtBearerEvents**.</span><span class="sxs-lookup"><span data-stu-id="3fab4-186">**Events** is a class that derives from **JwtBearerEvents**.</span></span>

### <a name="issuer-validation"></a><span data-ttu-id="3fab4-187">Validação do emissor</span><span class="sxs-lookup"><span data-stu-id="3fab4-187">Issuer validation</span></span>
<span data-ttu-id="3fab4-188">Valide o emissor do token no evento **JwtBearerEvents.TokenValidated**.</span><span class="sxs-lookup"><span data-stu-id="3fab4-188">Validate the token issuer in the **JwtBearerEvents.TokenValidated** event.</span></span> <span data-ttu-id="3fab4-189">O emissor é enviado na declaração "iss".</span><span class="sxs-lookup"><span data-stu-id="3fab4-189">The issuer is sent in the "iss" claim.</span></span>

<span data-ttu-id="3fab4-190">No aplicativo Surveys, a Web API não trata da [inscrição de locatários].</span><span class="sxs-lookup"><span data-stu-id="3fab4-190">In the Surveys application, the web API doesn't handle [tenant sign-up].</span></span> <span data-ttu-id="3fab4-191">Portanto, ela apenas verifica se o emissor já está no banco de dados do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="3fab4-191">Therefore, it just checks if the issuer is already in the application database.</span></span> <span data-ttu-id="3fab4-192">Se não estiver, ela gerará uma exceção que causará uma falha na autenticação.</span><span class="sxs-lookup"><span data-stu-id="3fab4-192">If not, it throws an exception, which causes authentication to fail.</span></span>

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.Ticket.Principal;
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue);

    if (tenant == null)
    {
        // The caller was not from a trusted issuer. Throw to block the authentication flow.
        throw new SecurityTokenValidationException();
    }

    var identity = principal.Identities.First();

    // Add new claim for survey_userid
    var registeredUser = await userManager.FindByObjectIdentifier(principal.GetObjectIdentifierValue());
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyUserIdClaimType, registeredUser.Id.ToString()));
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyTenantIdClaimType, registeredUser.TenantId.ToString()));

    // Add new claim for Email
    var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
    if (!string.IsNullOrWhiteSpace(email))
    {
        identity.AddClaim(new Claim(ClaimTypes.Email, email));
    }
}
```

<span data-ttu-id="3fab4-193">Como mostra este exemplo, você também pode usar o evento **TokenValidated** para modificar as declarações.</span><span class="sxs-lookup"><span data-stu-id="3fab4-193">As this example shows, you can also use the **TokenValidated** event to modify the claims.</span></span> <span data-ttu-id="3fab4-194">Lembre-se de que as declarações são fornecidas diretamente do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="3fab4-194">Remember that the claims come directly from Azure AD.</span></span> <span data-ttu-id="3fab4-195">Se o aplicativo Web modificar as declarações que obtém, essas alterações não aparecerão no token portador que a API Web recebe.</span><span class="sxs-lookup"><span data-stu-id="3fab4-195">If the web application modifies the claims that it gets, those changes won't show up in the bearer token that the web API receives.</span></span> <span data-ttu-id="3fab4-196">Para obter mais informações, consulte [Transformações de declarações][claims-transformation].</span><span class="sxs-lookup"><span data-stu-id="3fab4-196">For more information, see [Claims transformations][claims-transformation].</span></span>

## <a name="authorization"></a><span data-ttu-id="3fab4-197">Autorização</span><span class="sxs-lookup"><span data-stu-id="3fab4-197">Authorization</span></span>
<span data-ttu-id="3fab4-198">Para obter uma discussão geral sobre autorização, consulte [Autorização baseada em função e baseada em recursos][Authorization].</span><span class="sxs-lookup"><span data-stu-id="3fab4-198">For a general discussion of authorization, see [Role-based and resource-based authorization][Authorization].</span></span> 

<span data-ttu-id="3fab4-199">O middleware JwtBearer manipula as respostas de autorização.</span><span class="sxs-lookup"><span data-stu-id="3fab4-199">The JwtBearer middleware handles the authorization responses.</span></span> <span data-ttu-id="3fab4-200">Por exemplo, para restringir uma ação do controlador para usuários autenticados, use o atributo **[Autorizar]** e especifique **JwtBearerDefaults.AuthenticationScheme** como o esquema de autenticação:</span><span class="sxs-lookup"><span data-stu-id="3fab4-200">For example, to restrict a controller action to authenticated users, use the **[Authorize]** atrribute and specify **JwtBearerDefaults.AuthenticationScheme** as the authentication scheme:</span></span>

```csharp
[Authorize(ActiveAuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
```

<span data-ttu-id="3fab4-201">retornará um código de status 401 se o usuário não estiver autenticado.</span><span class="sxs-lookup"><span data-stu-id="3fab4-201">This returns a 401 status code if the user is not authenticated.</span></span>

<span data-ttu-id="3fab4-202">Para restringir uma ação do controlador pela política de autorização, especifique o nome da política no atributo **[Autorizar]**:</span><span class="sxs-lookup"><span data-stu-id="3fab4-202">To restrict a controller action by authorizaton policy, specify the policy name in the **[Authorize]** attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
```

<span data-ttu-id="3fab4-203">retornará um código de status 401 se o usuário não estiver autenticado e 403 se o usuário estiver autenticado, mas não autorizado.</span><span class="sxs-lookup"><span data-stu-id="3fab4-203">This returns a 401 status code if the user is not authenticated, and 403 if the user is authenticated but not authorized.</span></span> <span data-ttu-id="3fab4-204">Registre a política na inicialização:</span><span class="sxs-lookup"><span data-stu-id="3fab4-204">Register the policy on startup:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthorization(options =>
    {
        options.AddPolicy(PolicyNames.RequireSurveyCreator,
            policy =>
            {
                policy.AddRequirements(new SurveyCreatorRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
        options.AddPolicy(PolicyNames.RequireSurveyAdmin,
            policy =>
            {
                policy.AddRequirements(new SurveyAdminRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
    });
    
    // ...
}
```

<span data-ttu-id="3fab4-205">[**Avançar**][token cache]</span><span class="sxs-lookup"><span data-stu-id="3fab4-205">[**Next**][token cache]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[JwtBearer]: https://www.nuget.org/packages/Microsoft.AspNet.Authentication.JwtBearer

[Tailspin Surveys]: tailspin.md
[IdentityServer3]: https://github.com/IdentityServer/IdentityServer3
[Atualizar os manifestos do aplicativo]: ./run-the-app.md#update-the-application-manifests
[Colocação de tokens em cache]: token-cache.md
[inscrição de locatários]: signup.md
[claims-transformation]: claims.md#claims-transformations
[Authorization]: authorize.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[token cache]: token-cache.md
