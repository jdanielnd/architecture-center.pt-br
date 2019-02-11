---
title: Armazenar em cache tokens de acesso em um aplicativo multilocatário
description: Colocação em cache de tokens de acesso usados para chamar uma API Web de back-end.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: web-api
pnp.series.next: adfs
ms.openlocfilehash: 608a244eb0842da33af32457418d837b7f0a75c2
ms.sourcegitcommit: 14226018a058e199523106199be9c07f6a3f8592
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/31/2019
ms.locfileid: "55482936"
---
# <a name="cache-access-tokens"></a><span data-ttu-id="cb15d-103">Armazenar tokens de acesso em cache</span><span class="sxs-lookup"><span data-stu-id="cb15d-103">Cache access tokens</span></span>

<span data-ttu-id="cb15d-104">[Código de exemplo do ![GitHub](../_images/github.png)][sample application]</span><span class="sxs-lookup"><span data-stu-id="cb15d-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="cb15d-105">É relativamente caro obter um token de acesso OAuth porque requer uma solicitação HTTP para o ponto de extremidade do token.</span><span class="sxs-lookup"><span data-stu-id="cb15d-105">It's relatively expensive to get an OAuth access token, because it requires an HTTP request to the token endpoint.</span></span> <span data-ttu-id="cb15d-106">Portanto, é bom armazenar tokens em cache sempre que possível.</span><span class="sxs-lookup"><span data-stu-id="cb15d-106">Therefore, it's good to cache tokens whenever possible.</span></span> <span data-ttu-id="cb15d-107">A [Biblioteca de Autenticação do Azure AD][ADAL] (ADAL) armazena em cache automaticamente tokens obtidos do Azure AD, incluindo tokens de atualização.</span><span class="sxs-lookup"><span data-stu-id="cb15d-107">The [Azure AD Authentication Library][ADAL] (ADAL)  automatically caches tokens obtained from Azure AD, including refresh tokens.</span></span>

<span data-ttu-id="cb15d-108">A ADAL fornece uma implementação de cache de token padrão.</span><span class="sxs-lookup"><span data-stu-id="cb15d-108">ADAL provides a default token cache implementation.</span></span> <span data-ttu-id="cb15d-109">No entanto, o cache de token é destinado a aplicativos cliente nativos e **não** é adequado para aplicativos Web:</span><span class="sxs-lookup"><span data-stu-id="cb15d-109">However, this token cache is intended for native client apps, and is **not** suitable for web apps:</span></span>

* <span data-ttu-id="cb15d-110">É uma instância estática e não thread-safe.</span><span class="sxs-lookup"><span data-stu-id="cb15d-110">It is a static instance, and not thread safe.</span></span>
* <span data-ttu-id="cb15d-111">Não é dimensionado para um grande número de usuários porque os tokens de todos os usuários vão para o mesmo dicionário.</span><span class="sxs-lookup"><span data-stu-id="cb15d-111">It doesn't scale to large numbers of users, because tokens from all users go into the same dictionary.</span></span>
* <span data-ttu-id="cb15d-112">Não pode ser compartilhado entre servidores Web em um farm.</span><span class="sxs-lookup"><span data-stu-id="cb15d-112">It can't be shared across web servers in a farm.</span></span>

<span data-ttu-id="cb15d-113">Em vez disso, você deve implementar um cache de token personalizado que deriva da classe `TokenCache` da ADAL, mas é adequado para um ambiente de servidor e fornece o nível desejado de isolamento entre tokens para usuários diferentes.</span><span class="sxs-lookup"><span data-stu-id="cb15d-113">Instead, you should implement a custom token cache that derives from the ADAL `TokenCache` class but is suitable for a server environment and provides the desirable level of isolation between tokens for different users.</span></span>

<span data-ttu-id="cb15d-114">A classe `TokenCache` armazena um dicionário de tokens indexado por emissor, recursos, ID de cliente e usuário.</span><span class="sxs-lookup"><span data-stu-id="cb15d-114">The `TokenCache` class stores a dictionary of tokens, indexed by issuer, resource, client ID, and user.</span></span> <span data-ttu-id="cb15d-115">Um cache de token personalizado deve gravar este dicionário em um repositório de backup, como um cache Redis.</span><span class="sxs-lookup"><span data-stu-id="cb15d-115">A custom token cache should write this dictionary to a backing store, such as a Redis cache.</span></span>

<span data-ttu-id="cb15d-116">No aplicativo Tailspin Surveys, a classe `DistributedTokenCache` implementa o cache de token.</span><span class="sxs-lookup"><span data-stu-id="cb15d-116">In the Tailspin Surveys application, the `DistributedTokenCache` class implements the token cache.</span></span> <span data-ttu-id="cb15d-117">Essa implementação usa a abstração [IDistributedCache][distributed-cache] do ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="cb15d-117">This implementation uses the [IDistributedCache][distributed-cache] abstraction from ASP.NET Core.</span></span> <span data-ttu-id="cb15d-118">Dessa forma, qualquer implementação de `IDistributedCache` pode ser usada como um repositório de backup.</span><span class="sxs-lookup"><span data-stu-id="cb15d-118">That way, any `IDistributedCache` implementation can be used as a backing store.</span></span>

* <span data-ttu-id="cb15d-119">Por padrão, o aplicativo Surveys usa um cache Redis.</span><span class="sxs-lookup"><span data-stu-id="cb15d-119">By default, the Surveys app uses a Redis cache.</span></span>
* <span data-ttu-id="cb15d-120">Para um servidor Web de instância única, você pode usar o [cache na memória][in-memory-cache] do ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="cb15d-120">For a single-instance web server, you could use the ASP.NET Core [in-memory cache][in-memory-cache].</span></span> <span data-ttu-id="cb15d-121">(Isso também é uma boa opção para executar o aplicativo localmente durante o desenvolvimento.)</span><span class="sxs-lookup"><span data-stu-id="cb15d-121">(This is also a good option for running the app locally during development.)</span></span>

<span data-ttu-id="cb15d-122">`DistributedTokenCache` armazena os dados do cache como pares chave/valor no repositório de backup.</span><span class="sxs-lookup"><span data-stu-id="cb15d-122">`DistributedTokenCache` stores the cache data as key/value pairs in the backing store.</span></span> <span data-ttu-id="cb15d-123">A chave é a ID do cliente mais a ID de usuário para que o repositório de backup armazene dados de cache separados para cada combinação exclusiva de usuário/cliente.</span><span class="sxs-lookup"><span data-stu-id="cb15d-123">The key is the user ID plus client ID, so the backing store holds separate cache data for each unique combination of user/client.</span></span>

![Cache de token](./images/token-cache.png)

<span data-ttu-id="cb15d-125">O repositório de backup é particionado pelo usuário.</span><span class="sxs-lookup"><span data-stu-id="cb15d-125">The backing store is partitioned by user.</span></span> <span data-ttu-id="cb15d-126">Para cada solicitação HTTP, os tokens de usuário são lidos no armazenamento de backup e carregados no dicionário `TokenCache` .</span><span class="sxs-lookup"><span data-stu-id="cb15d-126">For each HTTP request, the tokens for that user are read from the backing store and loaded into the `TokenCache` dictionary.</span></span> <span data-ttu-id="cb15d-127">Se o Redis é usado como repositório de backup, cada instância de servidor em um farm de servidores lê e grava no mesmo cache e essa abordagem pode ser expandida para muitos usuários.</span><span class="sxs-lookup"><span data-stu-id="cb15d-127">If Redis is used as the backing store, every server instance in a server farm reads/writes to the same cache, and this approach scales to many users.</span></span>

## <a name="encrypting-cached-tokens"></a><span data-ttu-id="cb15d-128">Criptografar tokens em cache</span><span class="sxs-lookup"><span data-stu-id="cb15d-128">Encrypting cached tokens</span></span>

<span data-ttu-id="cb15d-129">Tokens são dados confidenciais, pois concedem acesso aos recursos do usuário.</span><span class="sxs-lookup"><span data-stu-id="cb15d-129">Tokens are sensitive data, because they grant access to a user's resources.</span></span> <span data-ttu-id="cb15d-130">(Além disso, ao contrário de uma senha de usuário, você não pode apenas armazenar um hash do token.) Portanto, é essencial proteger os tokens para que não sejam comprometidos.</span><span class="sxs-lookup"><span data-stu-id="cb15d-130">(Moreover, unlike a user's password, you can't just store a hash of the token.) Therefore, it's critical to protect tokens from being compromised.</span></span> <span data-ttu-id="cb15d-131">O cache Redis com suporte do Redis é protegido por senha, mas se alguém obtiver a senha, poderá obter todos os tokens de acesso armazenados em cache.</span><span class="sxs-lookup"><span data-stu-id="cb15d-131">The Redis-backed cache is protected by a password, but if someone obtains the password, they could get all of the cached access tokens.</span></span> <span data-ttu-id="cb15d-132">Por esse motivo, o `DistributedTokenCache` criptografa tudo o que grava no repositório de backup.</span><span class="sxs-lookup"><span data-stu-id="cb15d-132">For that reason, the `DistributedTokenCache` encrypts everything that it writes to the backing store.</span></span> <span data-ttu-id="cb15d-133">A criptografia é feita usando as APIs de [proteção de dados][data-protection] do ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="cb15d-133">Encryption is done using the ASP.NET Core [data protection][data-protection] APIs.</span></span>

> [!NOTE]
> <span data-ttu-id="cb15d-134">Se você implantar em Sites do Azure, será feito um backup das chaves de criptografia no armazenamento de rede e sincronizadas em todos os computadores (consulte [Gerenciamento e tempo de vida da chave][key-management]).</span><span class="sxs-lookup"><span data-stu-id="cb15d-134">If you deploy to Azure Web Sites, the encryption keys are backed up to network storage and synchronized across all machines (see [Key management and lifetime][key-management]).</span></span> <span data-ttu-id="cb15d-135">Por padrão, as chaves não são criptografadas durante a execução em sites do Azure Web, mas você pode [habilitar a criptografia usando um certificado X.509][x509-cert-encryption].</span><span class="sxs-lookup"><span data-stu-id="cb15d-135">By default, keys are not encrypted when running in Azure Web Sites, but you can [enable encryption using an X.509 certificate][x509-cert-encryption].</span></span>

## <a name="distributedtokencache-implementation"></a><span data-ttu-id="cb15d-136">Implementação do DistributedTokenCache</span><span class="sxs-lookup"><span data-stu-id="cb15d-136">DistributedTokenCache implementation</span></span>

<span data-ttu-id="cb15d-137">A classe `DistributedTokenCache` deriva da classe [TokenCache][tokencache-class] ADAL.</span><span class="sxs-lookup"><span data-stu-id="cb15d-137">The `DistributedTokenCache` class derives from the ADAL [TokenCache][tokencache-class] class.</span></span>

<span data-ttu-id="cb15d-138">No construtor, a classe `DistributedTokenCache` cria uma chave para o usuário atual e carrega o cache a partir do armazenamento de backup:</span><span class="sxs-lookup"><span data-stu-id="cb15d-138">In the constructor, the `DistributedTokenCache` class creates a key for the current user and loads the cache from the backing store:</span></span>

```csharp
public DistributedTokenCache(
    ClaimsPrincipal claimsPrincipal,
    IDistributedCache distributedCache,
    ILoggerFactory loggerFactory,
    IDataProtectionProvider dataProtectionProvider)
    : base()
{
    _claimsPrincipal = claimsPrincipal;
    _cacheKey = BuildCacheKey(_claimsPrincipal);
    _distributedCache = distributedCache;
    _logger = loggerFactory.CreateLogger<DistributedTokenCache>();
    _protector = dataProtectionProvider.CreateProtector(typeof(DistributedTokenCache).FullName);
    AfterAccess = AfterAccessNotification;
    LoadFromCache();
}
```

<span data-ttu-id="cb15d-139">A chave é criada concatenando a ID de usuário e a ID do cliente.</span><span class="sxs-lookup"><span data-stu-id="cb15d-139">The key is created by concatenating the user ID and client ID.</span></span> <span data-ttu-id="cb15d-140">Ambos são tirados das declarações encontradas na `ClaimsPrincipal`do usuário:</span><span class="sxs-lookup"><span data-stu-id="cb15d-140">Both of these are taken from claims found in the user's `ClaimsPrincipal`:</span></span>

```csharp
private static string BuildCacheKey(ClaimsPrincipal claimsPrincipal)
{
    string clientId = claimsPrincipal.FindFirstValue("aud", true);
    return string.Format(
        "UserId:{0}::ClientId:{1}",
        claimsPrincipal.GetObjectIdentifierValue(),
        clientId);
}
```

<span data-ttu-id="cb15d-141">Para carregar os dados do cache, leia o blob serializado do repositório de backup e chame `TokenCache.Deserialize` para converter o blob em dados de cache.</span><span class="sxs-lookup"><span data-stu-id="cb15d-141">To load the cache data, read the serialized blob from the backing store, and call `TokenCache.Deserialize` to convert the blob into cache data.</span></span>

```csharp
private void LoadFromCache()
{
    byte[] cacheData = _distributedCache.Get(_cacheKey);
    if (cacheData != null)
    {
        this.Deserialize(_protector.Unprotect(cacheData));
    }
}
```

<span data-ttu-id="cb15d-142">Sempre que a ADAL acessa o cache, ela dispara um evento `AfterAccess` .</span><span class="sxs-lookup"><span data-stu-id="cb15d-142">Whenever ADAL access the cache, it fires an `AfterAccess` event.</span></span> <span data-ttu-id="cb15d-143">Se os dados do cache forem alterados, a propriedade `HasStateChanged` será true.</span><span class="sxs-lookup"><span data-stu-id="cb15d-143">If the cache data has changed, the `HasStateChanged` property is true.</span></span> <span data-ttu-id="cb15d-144">Nesse caso, atualize o repositório de backup para refletir a alteração e defina `HasStateChanged` como false.</span><span class="sxs-lookup"><span data-stu-id="cb15d-144">In that case, update the backing store to reflect the change, and then set `HasStateChanged` to false.</span></span>

```csharp
public void AfterAccessNotification(TokenCacheNotificationArgs args)
{
    if (this.HasStateChanged)
    {
        try
        {
            if (this.Count > 0)
            {
                _distributedCache.Set(_cacheKey, _protector.Protect(this.Serialize()));
            }
            else
            {
                // There are no tokens for this user/client, so remove the item from the cache.
                _distributedCache.Remove(_cacheKey);
            }
            this.HasStateChanged = false;
        }
        catch (Exception exp)
        {
            _logger.WriteToCacheFailed(exp);
            throw;
        }
    }
}
```

<span data-ttu-id="cb15d-145">O TokenCache envia dois outros eventos:</span><span class="sxs-lookup"><span data-stu-id="cb15d-145">TokenCache sends two other events:</span></span>

* <span data-ttu-id="cb15d-146">`BeforeWrite`.</span><span class="sxs-lookup"><span data-stu-id="cb15d-146">`BeforeWrite`.</span></span> <span data-ttu-id="cb15d-147">Chamado imediatamente antes da ADAL gravar no cache.</span><span class="sxs-lookup"><span data-stu-id="cb15d-147">Called immediately before ADAL writes to the cache.</span></span> <span data-ttu-id="cb15d-148">Você pode usar isso para implementar uma estratégia de simultaneidade</span><span class="sxs-lookup"><span data-stu-id="cb15d-148">You can use this to implement a concurrency strategy</span></span>
* <span data-ttu-id="cb15d-149">`BeforeAccess`.</span><span class="sxs-lookup"><span data-stu-id="cb15d-149">`BeforeAccess`.</span></span> <span data-ttu-id="cb15d-150">Chamado imediatamente antes da ADAL ler do cache.</span><span class="sxs-lookup"><span data-stu-id="cb15d-150">Called immediately before ADAL reads from the cache.</span></span> <span data-ttu-id="cb15d-151">Aqui você pode recarregar o cache para obter a versão mais recente.</span><span class="sxs-lookup"><span data-stu-id="cb15d-151">Here you can reload the cache to get the latest version.</span></span>

<span data-ttu-id="cb15d-152">No nosso caso, decidimos não manipular esses dois eventos.</span><span class="sxs-lookup"><span data-stu-id="cb15d-152">In our case, we decided not to handle these two events.</span></span>

* <span data-ttu-id="cb15d-153">No caso de simultaneidade, a última gravação prevalece.</span><span class="sxs-lookup"><span data-stu-id="cb15d-153">For concurrency, last write wins.</span></span> <span data-ttu-id="cb15d-154">Não há problema, pois os tokens são armazenados de forma independente para cada usuário + cliente, portanto um conflito só acontece se o mesmo usuário tiver duas sessões simultâneas de logon.</span><span class="sxs-lookup"><span data-stu-id="cb15d-154">That's OK, because tokens are stored independently for each user + client, so a conflict would only happen if the same user had two concurrent login sessions.</span></span>
* <span data-ttu-id="cb15d-155">Para leitura, carregamos o cache em cada solicitação.</span><span class="sxs-lookup"><span data-stu-id="cb15d-155">For reading, we load the cache on every request.</span></span> <span data-ttu-id="cb15d-156">As solicitações têm vida curta.</span><span class="sxs-lookup"><span data-stu-id="cb15d-156">Requests are short lived.</span></span> <span data-ttu-id="cb15d-157">Se o cache for modificado nesse momento, a próxima solicitação selecionará o novo valor.</span><span class="sxs-lookup"><span data-stu-id="cb15d-157">If the cache gets modified in that time, the next request will pick up the new value.</span></span>

<span data-ttu-id="cb15d-158">[**Avançar**][client-assertion]</span><span class="sxs-lookup"><span data-stu-id="cb15d-158">[**Next**][client-assertion]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[client-assertion]: ./client-assertion.md
[data-protection]: /aspnet/core/security/data-protection/
[distributed-cache]: /aspnet/core/performance/caching/distributed
[key-management]: /aspnet/core/security/data-protection/configuration/default-settings
[in-memory-cache]: /aspnet/core/performance/caching/memory
[tokencache-class]: https://msdn.microsoft.com/library/azure/microsoft.identitymodel.clients.activedirectory.tokencache.aspx
[x509-cert-encryption]: /aspnet/core/security/data-protection/implementation/key-encryption-at-rest#x509-certificate
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
