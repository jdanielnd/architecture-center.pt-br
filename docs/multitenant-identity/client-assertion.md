---
title: Usar a declaração do cliente para obter tokens de acesso do Azure AD
description: Como usar a declaração do cliente para obter tokens de acesso do Azure AD.
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: adfs
pnp.series.next: key-vault
ms.openlocfilehash: 9fe1ee2ec5a540edc41c3a310476507f8d862f0c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540274"
---
# <a name="use-client-assertion-to-get-access-tokens-from-azure-ad"></a><span data-ttu-id="8666c-103">Usar a declaração do cliente para obter tokens de acesso do Azure AD</span><span class="sxs-lookup"><span data-stu-id="8666c-103">Use client assertion to get access tokens from Azure AD</span></span>

<span data-ttu-id="8666c-104">[Código de exemplo do ![GitHub](../_images/github.png)][sample application]</span><span class="sxs-lookup"><span data-stu-id="8666c-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

## <a name="background"></a><span data-ttu-id="8666c-105">Segundo plano</span><span class="sxs-lookup"><span data-stu-id="8666c-105">Background</span></span>
<span data-ttu-id="8666c-106">Ao usar o fluxo de código de autorização ou o fluxo híbrido no OpenID Connect, o cliente troca um código de autorização por um token de acesso.</span><span class="sxs-lookup"><span data-stu-id="8666c-106">When using authorization code flow or hybrid flow in OpenID Connect, the client exchanges an authorization code for an access token.</span></span> <span data-ttu-id="8666c-107">Durante essa etapa, o cliente deve se autenticar no servidor.</span><span class="sxs-lookup"><span data-stu-id="8666c-107">During this step, the client has to authenticate itself to the server.</span></span>

![Segredo do cliente](./images/client-secret.png)

<span data-ttu-id="8666c-109">Uma maneira de autenticar o cliente é usar um segredo do cliente.</span><span class="sxs-lookup"><span data-stu-id="8666c-109">One way to authenticate the client is by using a client secret.</span></span> <span data-ttu-id="8666c-110">É assim que o aplicativo [Tailspin Surveys][Surveys] está configurado por padrão.</span><span class="sxs-lookup"><span data-stu-id="8666c-110">That's how the [Tailspin Surveys][Surveys] application is configured by default.</span></span>

<span data-ttu-id="8666c-111">Veja um exemplo de solicitação do cliente para o IDP, solicitando um token de acesso.</span><span class="sxs-lookup"><span data-stu-id="8666c-111">Here is an example request from the client to the IDP, requesting an access token.</span></span> <span data-ttu-id="8666c-112">Observe o parâmetro `client_secret` .</span><span class="sxs-lookup"><span data-stu-id="8666c-112">Note the `client_secret` parameter.</span></span>

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_secret=i3Bf12Dn...
  &grant_type=authorization_code
  &code=PG8wJG6Y...
```

<span data-ttu-id="8666c-113">O segredo é apenas uma cadeia de caracteres, portanto, você precisa tomar cuidado para não divulgar inadvertidamente o valor.</span><span class="sxs-lookup"><span data-stu-id="8666c-113">The secret is just a string, so you have to make sure not to leak the value.</span></span> <span data-ttu-id="8666c-114">A prática recomendada é manter o segredo do cliente fora do controle de origens.</span><span class="sxs-lookup"><span data-stu-id="8666c-114">The best practice is to keep the client secret out of source control.</span></span> <span data-ttu-id="8666c-115">Quando você implantar no Azure, armazene o segredo em uma [configuração do aplicativo][configure-web-app].</span><span class="sxs-lookup"><span data-stu-id="8666c-115">When you deploy to Azure, store the secret in an [app setting][configure-web-app].</span></span>

<span data-ttu-id="8666c-116">No entanto, qualquer pessoa com acesso à assinatura do Azure pode exibir as configurações do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="8666c-116">However, anyone with access to the Azure subscription can view the app settings.</span></span> <span data-ttu-id="8666c-117">Além disso, há sempre a tentação de inserir segredos no controle de origem (por exemplo, em scripts de implantação), compartilhá-los por email e assim por diante.</span><span class="sxs-lookup"><span data-stu-id="8666c-117">Further, there is always a temptation to check secrets into source control (e.g., in deployment scripts), share them by email, and so on.</span></span>

<span data-ttu-id="8666c-118">Para obter segurança adicional, você pode usar a [declaração do cliente] em vez de um segredo do cliente.</span><span class="sxs-lookup"><span data-stu-id="8666c-118">For additional security, you can use [client assertion] instead of a client secret.</span></span> <span data-ttu-id="8666c-119">Com a declaração do cliente, o cliente usa um certificado X.509 para comprovar que a solicitação de token veio do cliente.</span><span class="sxs-lookup"><span data-stu-id="8666c-119">With client assertion, the client uses an X.509 certificate to prove the token request came from the client.</span></span> <span data-ttu-id="8666c-120">O certificado do cliente é instalado no servidor Web.</span><span class="sxs-lookup"><span data-stu-id="8666c-120">The client certificate is installed on the web server.</span></span> <span data-ttu-id="8666c-121">Em geral, é mais fácil restringir o acesso ao certificado do que garantir que ninguém revele inadvertidamente um segredo do cliente.</span><span class="sxs-lookup"><span data-stu-id="8666c-121">Generally, it will be easier to restrict access to the certificate, than to ensure that nobody inadvertently reveals a client secret.</span></span> <span data-ttu-id="8666c-122">Para saber mais sobre como configurar certificados em um aplicativo Web, veja [Uso de certificados em aplicativos de sites do Azure][using-certs-in-websites]</span><span class="sxs-lookup"><span data-stu-id="8666c-122">For more information about configuring certificates in a web app, see [Using Certificates in Azure Websites Applications][using-certs-in-websites]</span></span>

<span data-ttu-id="8666c-123">Veja uma solicitação de token usando a declaração do cliente:</span><span class="sxs-lookup"><span data-stu-id="8666c-123">Here is a token request using client assertion:</span></span>

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
  &client_assertion=eyJhbGci...
  &grant_type=authorization_code
  &code= PG8wJG6Y...
```

<span data-ttu-id="8666c-124">Observe que o parâmetro `client_secret` não é mais usado.</span><span class="sxs-lookup"><span data-stu-id="8666c-124">Notice that the `client_secret` parameter is no longer used.</span></span> <span data-ttu-id="8666c-125">Em vez disso, o parâmetro `client_assertion` contém um token JWT que foi assinado usando o certificado do cliente.</span><span class="sxs-lookup"><span data-stu-id="8666c-125">Instead, the `client_assertion` parameter contains a JWT token that was signed using the client certificate.</span></span> <span data-ttu-id="8666c-126">O parâmetro `client_assertion_type` especifica o tipo de declaração &mdash;, nesse caso, o token JWT.</span><span class="sxs-lookup"><span data-stu-id="8666c-126">The `client_assertion_type` parameter specifies the type of assertion &mdash; in this case, JWT token.</span></span> <span data-ttu-id="8666c-127">O servidor valida o token JWT.</span><span class="sxs-lookup"><span data-stu-id="8666c-127">The server validates the JWT token.</span></span> <span data-ttu-id="8666c-128">Se o token JWT for inválido, a solicitação de token retornará um erro.</span><span class="sxs-lookup"><span data-stu-id="8666c-128">If the JWT token is invalid, the token request returns an error.</span></span>

> [!NOTE]
> <span data-ttu-id="8666c-129">Os certificados x. 509 não são a única forma de asserção de cliente; nosso foco nele aqui é porque ele tem suporte do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="8666c-129">X.509 certificates are not the only form of client assertion; we focus on it here because it is supported by Azure AD.</span></span>
> 
> 

<span data-ttu-id="8666c-130">No tempo de execução, o aplicativo Web lê o certificado do repositório de certificados.</span><span class="sxs-lookup"><span data-stu-id="8666c-130">At run time, the web application reads the certificate from the certificate store.</span></span> <span data-ttu-id="8666c-131">O certificado deve ser instalado no mesmo computador que o aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="8666c-131">The certificate must be installed on the same machine as the web app.</span></span>

<span data-ttu-id="8666c-132">O aplicativo Surveys inclui uma classe auxiliar que cria um [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) que você pode passar para o método [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) para adquirir um token do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="8666c-132">The Surveys application includes a helper class that creates a [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) that you can pass to the [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) method to acquire a token from Azure AD.</span></span>

```csharp
public class CertificateCredentialService : ICredentialService
{
    private Lazy<Task<AdalCredential>> _credential;

    public CertificateCredentialService(IOptions<ConfigurationOptions> options)
    {
        var aadOptions = options.Value?.AzureAd;
        _credential = new Lazy<Task<AdalCredential>>(() =>
        {
            X509Certificate2 cert = CertificateUtility.FindCertificateByThumbprint(
                aadOptions.Asymmetric.StoreName,
                aadOptions.Asymmetric.StoreLocation,
                aadOptions.Asymmetric.CertificateThumbprint,
                aadOptions.Asymmetric.ValidationRequired);
            string password = null;
            var certBytes = CertificateUtility.ExportCertificateWithPrivateKey(cert, out password);
            return Task.FromResult(new AdalCredential(new ClientAssertionCertificate(aadOptions.ClientId, new X509Certificate2(certBytes, password))));
        });
    }

    public async Task<AdalCredential> GetCredentialsAsync()
    {
        return await _credential.Value;
    }
}
```

<span data-ttu-id="8666c-133">Para saber mais sobre como configurar a asserção de cliente no aplicativo Surveys, veja [Usar o Azure Key Vault para proteger segredos do aplicativo ] [ key vault].</span><span class="sxs-lookup"><span data-stu-id="8666c-133">For information about setting up client assertion in the Surveys application, see [Use Azure Key Vault to protect application secrets ][key vault].</span></span>

<span data-ttu-id="8666c-134">[**Avançar**][key vault]</span><span class="sxs-lookup"><span data-stu-id="8666c-134">[**Next**][key vault]</span></span>

<!-- Links -->
[configure-web-app]: /azure/app-service-web/web-sites-configure/
[azure-management-portal]: https://portal.azure.com
[declaração do cliente]: https://tools.ietf.org/html/rfc7521
[client assertion]: https://tools.ietf.org/html/rfc7521
[key vault]: key-vault.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[using-certs-in-websites]: https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/

[sample application]: https://github.com/mspnp/multitenant-saas-guidance
