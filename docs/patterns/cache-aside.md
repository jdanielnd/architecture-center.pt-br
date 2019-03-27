---
title: Padrão Cache-Aside
titleSuffix: Cloud Design Patterns
description: Carregar dados sob demanda em um cache de um armazenamento de dados.
keywords: padrão de design
author: dragon119
ms.date: 11/01/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: c4b423b2031699210d5917f12a4c14df0f4a694c
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58244477"
---
# <a name="cache-aside-pattern"></a><span data-ttu-id="6789c-104">Padrão Cache-Aside</span><span class="sxs-lookup"><span data-stu-id="6789c-104">Cache-Aside pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="6789c-105">Carregar dados sob demanda em um cache de um armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="6789c-105">Load data on demand into a cache from a data store.</span></span> <span data-ttu-id="6789c-106">Isso pode melhorar o desempenho e também ajuda a manter a consistência entre dados mantidos no cache e os dados no armazenamento de dados subjacente.</span><span class="sxs-lookup"><span data-stu-id="6789c-106">This can improve performance and also helps to maintain consistency between data held in the cache and data in the underlying data store.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="6789c-107">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="6789c-107">Context and problem</span></span>

<span data-ttu-id="6789c-108">Os aplicativos usam um cache para melhorar o acesso repetido às informações mantidas em um armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="6789c-108">Applications use a cache to improve repeated access to information held in a data store.</span></span> <span data-ttu-id="6789c-109">No entanto, não é prático esperar que os dados armazenados em cache sejam sempre completamente consistentes com os dados no armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="6789c-109">However, it's impractical to expect that cached data will always be completely consistent with the data in the data store.</span></span> <span data-ttu-id="6789c-110">Os aplicativos devem implementar uma estratégia que ajude a garantir que os dados no cache sejam tão atualizados quanto possível, mas também possam detectar e lidar com situações que possam surgir quando os dados no cache se tornarem obsoletos.</span><span class="sxs-lookup"><span data-stu-id="6789c-110">Applications should implement a strategy that helps to ensure that the data in the cache is as up-to-date as possible, but can also detect and handle situations that arise when the data in the cache has become stale.</span></span>

## <a name="solution"></a><span data-ttu-id="6789c-111">Solução</span><span class="sxs-lookup"><span data-stu-id="6789c-111">Solution</span></span>

<span data-ttu-id="6789c-112">Muitos sistemas de armazenamento em cache comerciais fornecem operações de read-through e write-through/write-behind.</span><span class="sxs-lookup"><span data-stu-id="6789c-112">Many commercial caching systems provide read-through and write-through/write-behind operations.</span></span> <span data-ttu-id="6789c-113">Nesses sistemas, um aplicativo recupera dados referenciando o cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-113">In these systems, an application retrieves data by referencing the cache.</span></span> <span data-ttu-id="6789c-114">Se os dados não estiverem no cache, ele é recuperado do armazenamento de dados e adicionado ao cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-114">If the data isn't in the cache, it's retrieved from the data store and added to the cache.</span></span> <span data-ttu-id="6789c-115">Todas as modificações nos dados mantidos no cache são automaticamente gravadas no armazenamento de dados também.</span><span class="sxs-lookup"><span data-stu-id="6789c-115">Any modifications to data held in the cache are automatically written back to the data store as well.</span></span>

<span data-ttu-id="6789c-116">Para caches que não têm essa funcionalidade, é de responsabilidade dos aplicativos que usam o cache manter os dados.</span><span class="sxs-lookup"><span data-stu-id="6789c-116">For caches that don't provide this functionality, it's the responsibility of the applications that use the cache to maintain the data.</span></span>

<span data-ttu-id="6789c-117">Um aplicativo pode emular a funcionalidade de cache de read-through implementando a estratégia de cache-aside.</span><span class="sxs-lookup"><span data-stu-id="6789c-117">An application can emulate the functionality of read-through caching by implementing the cache-aside strategy.</span></span> <span data-ttu-id="6789c-118">Essa estratégia carrega os dados em cache sob demanda.</span><span class="sxs-lookup"><span data-stu-id="6789c-118">This strategy loads data into the cache on demand.</span></span> <span data-ttu-id="6789c-119">A figura ilustra usando o padrão de Cache-Aside para armazenar dados no cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-119">The figure illustrates using the Cache-Aside pattern to store data in the cache.</span></span>

![Usar o padrão de Cache-Aside para armazenar dados no cache](./_images/cache-aside-diagram.png)

<span data-ttu-id="6789c-121">Se um aplicativo atualiza as informações, ele pode seguir a estratégia de write-through modificando o armazenamento de dados e invalidando o item correspondente no cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-121">If an application updates information, it can follow the write-through strategy by making the modification to the data store, and by invalidating the corresponding item in the cache.</span></span>

<span data-ttu-id="6789c-122">Quando o item é necessário a seguir, usar a estratégia de cache-aside fará com que os dados atualizados sejam recuperados no armazenamento de dados e adicionados novamente ao cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-122">When the item is next required, using the cache-aside strategy will cause the updated data to be retrieved from the data store and added back into the cache.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="6789c-123">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="6789c-123">Issues and considerations</span></span>

<span data-ttu-id="6789c-124">Considere os seguintes pontos ao decidir como implementar esse padrão:</span><span class="sxs-lookup"><span data-stu-id="6789c-124">Consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="6789c-125">**Tempo de vida dos dados armazenados em cache**.</span><span class="sxs-lookup"><span data-stu-id="6789c-125">**Lifetime of cached data**.</span></span> <span data-ttu-id="6789c-126">Vários caches implementam uma política de expiração que invalida os dados e os remove do cache se ele não for acessado por um período especificado.</span><span class="sxs-lookup"><span data-stu-id="6789c-126">Many caches implement an expiration policy that invalidates data and removes it from the cache if it's not accessed for a specified period.</span></span> <span data-ttu-id="6789c-127">Para o cache-aside ser efetivado, verifique se a política de expiração corresponde ao padrão de acesso para aplicativos que usam os dados.</span><span class="sxs-lookup"><span data-stu-id="6789c-127">For cache-aside to be effective, ensure that the expiration policy matches the pattern of access for applications that use the data.</span></span> <span data-ttu-id="6789c-128">Não faça o período de validade muito curto, porque isso pode fazer com que os aplicativos recuperem continuamente os dados do armazenamento de dados e adicionem-nos ao cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-128">Don't make the expiration period too short because this can cause applications to continually retrieve data from the data store and add it to the cache.</span></span> <span data-ttu-id="6789c-129">Da mesma forma, não faça o período de validade longo demais a ponto dos dados em cache se tornarem provavelmente obsoletos.</span><span class="sxs-lookup"><span data-stu-id="6789c-129">Similarly, don't make the expiration period so long that the cached data is likely to become stale.</span></span> <span data-ttu-id="6789c-130">Lembre-se de que o cache é mais eficiente para dados relativamente estáticos ou dados lidos com frequência.</span><span class="sxs-lookup"><span data-stu-id="6789c-130">Remember that caching is most effective for relatively static data, or data that is read frequently.</span></span>

<span data-ttu-id="6789c-131">**Removendo dados**.</span><span class="sxs-lookup"><span data-stu-id="6789c-131">**Evicting data**.</span></span> <span data-ttu-id="6789c-132">A maioria dos caches tem um tamanho limitado em comparação com o armazenamento de dados de onde os dados são originários, e eles removerão os dados, se necessário.</span><span class="sxs-lookup"><span data-stu-id="6789c-132">Most caches have a limited size compared to the data store where the data originates, and they'll evict data if necessary.</span></span> <span data-ttu-id="6789c-133">A maioria dos caches adotam uma política de usados menos recentemente para selecionar itens a serem removidos, mas isso pode ser personalizável.</span><span class="sxs-lookup"><span data-stu-id="6789c-133">Most caches adopt a least-recently-used policy for selecting items to evict, but this might be customizable.</span></span> <span data-ttu-id="6789c-134">Configure a propriedade global de expiração e outras propriedades do cache, assim como a propriedade de expiração de cada item em cache, para garantir que o cache seja econômico.</span><span class="sxs-lookup"><span data-stu-id="6789c-134">Configure the global expiration property and other properties of the cache, and the expiration property of each cached item, to ensure that the cache is cost effective.</span></span> <span data-ttu-id="6789c-135">Nem sempre é apropriado aplicar uma política de remoção global para cada item no cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-135">It isn't always appropriate to apply a global eviction policy to every item in the cache.</span></span> <span data-ttu-id="6789c-136">Por exemplo, se um item em cache for muito caro para recuperar no armazenamento de dados, pode ser benéfico manter esse item no cache e dar preferência a itens acessados com mais frequência, porém mais baratos.</span><span class="sxs-lookup"><span data-stu-id="6789c-136">For example, if a cached item is very expensive to retrieve from the data store, it can be beneficial to keep this item in the cache at the expense of more frequently accessed but less costly items.</span></span>

<span data-ttu-id="6789c-137">**Desobstrução do cache**.</span><span class="sxs-lookup"><span data-stu-id="6789c-137">**Priming the cache**.</span></span> <span data-ttu-id="6789c-138">Muitas soluções preenchem o cache com os dados que um aplicativo pode precisar como parte do processo de inicialização.</span><span class="sxs-lookup"><span data-stu-id="6789c-138">Many solutions prepopulate the cache with the data that an application is likely to need as part of the startup processing.</span></span> <span data-ttu-id="6789c-139">O padrão de Cache-Aside ainda pode ser útil se alguns desses dados expirar ou for removido.</span><span class="sxs-lookup"><span data-stu-id="6789c-139">The Cache-Aside pattern can still be useful if some of this data expires or is evicted.</span></span>

<span data-ttu-id="6789c-140">**Consistência**.</span><span class="sxs-lookup"><span data-stu-id="6789c-140">**Consistency**.</span></span> <span data-ttu-id="6789c-141">Implementar o padrão de Cache-Aside não garante a consistência entre o armazenamento de dados e o cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-141">Implementing the Cache-Aside pattern doesn't guarantee consistency between the data store and the cache.</span></span> <span data-ttu-id="6789c-142">Um item no armazenamento de dados pode ser alterado a qualquer momento por um processo externo, e essa alteração não pode ser refletida no cache até a próxima vez que o item seja carregado.</span><span class="sxs-lookup"><span data-stu-id="6789c-142">An item in the data store can be changed at any time by an external process, and this change might not be reflected in the cache until the next time the item is loaded.</span></span> <span data-ttu-id="6789c-143">Em um sistema que replica os dados por vários armazenamentos de dados, esse problema pode se tornar sério se a sincronização ocorrer com frequência.</span><span class="sxs-lookup"><span data-stu-id="6789c-143">In a system that replicates data across data stores, this problem can become serious if synchronization occurs frequently.</span></span>

<span data-ttu-id="6789c-144">**Armazenamento em cache local (na memória)**.</span><span class="sxs-lookup"><span data-stu-id="6789c-144">**Local (in-memory) caching**.</span></span> <span data-ttu-id="6789c-145">Um cache pode ser local para uma instância de aplicativo e armazenado na memória.</span><span class="sxs-lookup"><span data-stu-id="6789c-145">A cache could be local to an application instance and stored in-memory.</span></span> <span data-ttu-id="6789c-146">O cache-aside pode ser útil nesse ambiente, se um aplicativo acessar os mesmos dados repetidamente.</span><span class="sxs-lookup"><span data-stu-id="6789c-146">Cache-aside can be useful in this environment if an application repeatedly accesses the same data.</span></span> <span data-ttu-id="6789c-147">No entanto, um cache local é privado e, por isso, diferentes instâncias do aplicativo podem ter uma cópia dos mesmos dados armazenados em cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-147">However, a local cache is private and so different application instances could each have a copy of the same cached data.</span></span> <span data-ttu-id="6789c-148">Esses dados podem rapidamente se tornar inconsistentes entre os caches, por isso é necessário expirar os dados mantidos em um cache privado e atualizá-los com mais frequência.</span><span class="sxs-lookup"><span data-stu-id="6789c-148">This data could quickly become inconsistent between caches, so it might be necessary to expire data held in a private cache and refresh it more frequently.</span></span> <span data-ttu-id="6789c-149">Nesses cenários, considere a possibilidade de investigar o uso de um mecanismo de cache compartilhado ou distribuído.</span><span class="sxs-lookup"><span data-stu-id="6789c-149">In these scenarios, consider investigating the use of a shared or a distributed caching mechanism.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="6789c-150">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="6789c-150">When to use this pattern</span></span>

<span data-ttu-id="6789c-151">Use esse padrão quando:</span><span class="sxs-lookup"><span data-stu-id="6789c-151">Use this pattern when:</span></span>

- <span data-ttu-id="6789c-152">Um cache não fornece read-through nativo e as operações de write-through.</span><span class="sxs-lookup"><span data-stu-id="6789c-152">A cache doesn't provide native read-through and write-through operations.</span></span>
- <span data-ttu-id="6789c-153">A demanda de recursos é imprevisível.</span><span class="sxs-lookup"><span data-stu-id="6789c-153">Resource demand is unpredictable.</span></span> <span data-ttu-id="6789c-154">Esse padrão permite que os aplicativos carreguem dados sob demanda.</span><span class="sxs-lookup"><span data-stu-id="6789c-154">This pattern enables applications to load data on demand.</span></span> <span data-ttu-id="6789c-155">Ele não faz nenhuma suposição sobre quais dados um aplicativo exigirá antecipadamente.</span><span class="sxs-lookup"><span data-stu-id="6789c-155">It makes no assumptions about which data an application will require in advance.</span></span>

<span data-ttu-id="6789c-156">Esse padrão pode não ser adequado:</span><span class="sxs-lookup"><span data-stu-id="6789c-156">This pattern might not be suitable:</span></span>

- <span data-ttu-id="6789c-157">Quando o conjunto de dados armazenados em cache é estático.</span><span class="sxs-lookup"><span data-stu-id="6789c-157">When the cached data set is static.</span></span> <span data-ttu-id="6789c-158">Se os dados se ajustarem ao espaço em cache disponível, desobstrua o cache com os dados na inicialização e aplique uma política que impeça os dados de expirar.</span><span class="sxs-lookup"><span data-stu-id="6789c-158">If the data will fit into the available cache space, prime the cache with the data on startup and apply a policy that prevents the data from expiring.</span></span>
- <span data-ttu-id="6789c-159">Para armazenar em cache informações de estado de sessão em um aplicativo Web hospedado em um web farm.</span><span class="sxs-lookup"><span data-stu-id="6789c-159">For caching session state information in a web application hosted in a web farm.</span></span> <span data-ttu-id="6789c-160">Nesse ambiente, você deve evitar introduzir dependências baseadas na afinidade de cliente-servidor.</span><span class="sxs-lookup"><span data-stu-id="6789c-160">In this environment, you should avoid introducing dependencies based on client-server affinity.</span></span>

## <a name="example"></a><span data-ttu-id="6789c-161">Exemplo</span><span class="sxs-lookup"><span data-stu-id="6789c-161">Example</span></span>

<span data-ttu-id="6789c-162">No Microsoft Azure você pode usar o Cache Redis do Azure para criar um cache distribuído que pode ser compartilhado por várias instâncias de um aplicativo.</span><span class="sxs-lookup"><span data-stu-id="6789c-162">In Microsoft Azure you can use Azure Redis Cache to create a distributed cache that can be shared by multiple instances of an application.</span></span>

<span data-ttu-id="6789c-163">Esses exemplos de código a seguir usam o cliente [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis), que é uma biblioteca de cliente Redis gravada para o .NET.</span><span class="sxs-lookup"><span data-stu-id="6789c-163">This following code examples use the [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) client, which is a Redis client library written for .NET.</span></span> <span data-ttu-id="6789c-164">Para se conectar a uma instância do Cache Redis do Azure, chame o método estático `ConnectionMultiplexer.Connect`e passe a cadeia de conexão.</span><span class="sxs-lookup"><span data-stu-id="6789c-164">To connect to an Azure Redis Cache instance, call the static `ConnectionMultiplexer.Connect` method and pass in the connection string.</span></span> <span data-ttu-id="6789c-165">O método retorna um `ConnectionMultiplexer` que representa a conexão.</span><span class="sxs-lookup"><span data-stu-id="6789c-165">The method returns a `ConnectionMultiplexer` that represents the connection.</span></span> <span data-ttu-id="6789c-166">Uma abordagem para compartilhar uma instância do `ConnectionMultiplexer` em seu aplicativo deve ter uma propriedade estática que retorna uma instância conectada, semelhante ao exemplo a seguir.</span><span class="sxs-lookup"><span data-stu-id="6789c-166">One approach to sharing a `ConnectionMultiplexer` instance in your application is to have a static property that returns a connected instance, similar to the following example.</span></span> <span data-ttu-id="6789c-167">Essa abordagem oferece uma maneira thread-safe de inicializar somente uma única instância conectada.</span><span class="sxs-lookup"><span data-stu-id="6789c-167">This approach provides a thread-safe way to initialize only a single connected instance.</span></span>

```csharp
private static ConnectionMultiplexer Connection;

// Redis Connection string info
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
    return ConnectionMultiplexer.Connect(cacheConnection);
});

public static ConnectionMultiplexer Connection => lazyConnection.Value;
```

<span data-ttu-id="6789c-168">O método `GetMyEntityAsync` no exemplo de código a seguir mostra uma implementação do padrão Cache-Aside.</span><span class="sxs-lookup"><span data-stu-id="6789c-168">The `GetMyEntityAsync` method in the following code example shows an implementation of the Cache-Aside pattern.</span></span> <span data-ttu-id="6789c-169">Esse método recupera um objeto do cache usando a abordagem read-through.</span><span class="sxs-lookup"><span data-stu-id="6789c-169">This method retrieves an object from the cache using the read-through approach.</span></span>

<span data-ttu-id="6789c-170">Um objeto é identificado usando uma ID inteira como chave.</span><span class="sxs-lookup"><span data-stu-id="6789c-170">An object is identified by using an integer ID as the key.</span></span> <span data-ttu-id="6789c-171">O método `GetMyEntityAsync` tenta recuperar um item com essa chave do cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-171">The `GetMyEntityAsync` method tries to retrieve an item with this key from the cache.</span></span> <span data-ttu-id="6789c-172">Se um item correspondente for encontrado, ele é retornado.</span><span class="sxs-lookup"><span data-stu-id="6789c-172">If a matching item is found, it's returned.</span></span> <span data-ttu-id="6789c-173">Se não houver nenhuma correspondência no cache, o método `GetMyEntityAsync` recupera o objeto de um armazenamento de dados, adiciona-o ao cache e, em seguida, retorna-o.</span><span class="sxs-lookup"><span data-stu-id="6789c-173">If there's no match in the cache, the `GetMyEntityAsync` method retrieves the object from a data store, adds it to the cache, and then returns it.</span></span> <span data-ttu-id="6789c-174">O código que realmente lê os dados do armazenamento de dados não é mostrado aqui, pois ele depende do armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="6789c-174">The code that actually reads the data from the data store is not shown here, because it depends on the data store.</span></span> <span data-ttu-id="6789c-175">Observe que o item de cache é configurado para expirar, a fim de impedir que ele se torne obsoleto, caso tenha se atualizado em outro lugar.</span><span class="sxs-lookup"><span data-stu-id="6789c-175">Note that the cached item is configured to expire to prevent it from becoming stale if it's updated elsewhere.</span></span>

```csharp
// Set five minute expiration as a default
private const double DefaultExpirationTimeInMinutes = 5.0;

public async Task<MyEntity> GetMyEntityAsync(int id)
{
  // Define a unique key for this method and its parameters.
  var key = $"MyEntity:{id}";
  var cache = Connection.GetDatabase();

  // Try to get the entity from the cache.
  var json = await cache.StringGetAsync(key).ConfigureAwait(false);
  var value = string.IsNullOrWhiteSpace(json)
                ? default(MyEntity)
                : JsonConvert.DeserializeObject<MyEntity>(json);

  if (value == null) // Cache miss
  {
    // If there's a cache miss, get the entity from the original store and cache it.
    // Code has been omitted because it is data store dependent.
    value = ...;

    // Avoid caching a null value.
    if (value != null)
    {
      // Put the item in the cache with a custom expiration time that
      // depends on how critical it is to have stale data.
      await cache.StringSetAsync(key, JsonConvert.SerializeObject(value)).ConfigureAwait(false);
      await cache.KeyExpireAsync(key, TimeSpan.FromMinutes(DefaultExpirationTimeInMinutes)).ConfigureAwait(false);
    }
  }

  return value;
}
```

> <span data-ttu-id="6789c-176">Os exemplos usam o Cache Redis para acessar o armazenamento e recuperar as informações do cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-176">The examples use Redis Cache to access the store and retrieve information from the cache.</span></span> <span data-ttu-id="6789c-177">Para obter mais informações, confira [Como usar o Cache Redis do Microsoft Azure](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) e [Como criar um aplicativo Web com o Cache Redis](/azure/redis-cache/cache-web-app-howto)</span><span class="sxs-lookup"><span data-stu-id="6789c-177">For more information, see [Using Microsoft Azure Redis Cache](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) and [How to create a Web App with Redis Cache](/azure/redis-cache/cache-web-app-howto)</span></span>

<span data-ttu-id="6789c-178">O método `UpdateEntityAsync` mostrado a seguir demonstra como invalidar um objeto no cache quando o valor é alterado pelo aplicativo.</span><span class="sxs-lookup"><span data-stu-id="6789c-178">The `UpdateEntityAsync` method shown below demonstrates how to invalidate an object in the cache when the value is changed by the application.</span></span> <span data-ttu-id="6789c-179">O código atualiza o armazenamento de dados original e, em seguida, remove o item do cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-179">The code updates the original data store and then removes the cached item from the cache.</span></span>

```csharp
public async Task UpdateEntityAsync(MyEntity entity)
{
    // Update the object in the original data store.
    await this.store.UpdateEntityAsync(entity).ConfigureAwait(false);

    // Invalidate the current cache object.
    var cache = Connection.GetDatabase();
    var id = entity.Id;
    var key = $"MyEntity:{id}"; // The key for the cached object.
    await cache.KeyDeleteAsync(key).ConfigureAwait(false); // Delete this key from the cache.
}
```

> [!NOTE]
> <span data-ttu-id="6789c-180">A ordem das etapas é importante.</span><span class="sxs-lookup"><span data-stu-id="6789c-180">The order of the steps is important.</span></span> <span data-ttu-id="6789c-181">Atualize o armazenamento de dados *antes* de remover o item do cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-181">Update the data store *before* removing the item from the cache.</span></span> <span data-ttu-id="6789c-182">Se você remover o item do cache primeiro, há uma pequena janela de tempo quando um cliente pode obter o item antes de atualizar o armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="6789c-182">If you remove the cached item first, there is a small window of time when a client might fetch the item before the data store is updated.</span></span> <span data-ttu-id="6789c-183">Isso resultará na perda de cache (porque o item foi removido do cache), fazendo com que a versão anterior do item seja obtida no repositório de dados e adicionado de volta ao cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-183">That will result in a cache miss (because the item was removed from the cache), causing the earlier version of the item to be fetched from the data store and added back into the cache.</span></span> <span data-ttu-id="6789c-184">O resultado será dados obsoletos do cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-184">The result will be stale cache data.</span></span>

## <a name="related-guidance"></a><span data-ttu-id="6789c-185">Diretrizes relacionadas</span><span class="sxs-lookup"><span data-stu-id="6789c-185">Related guidance</span></span>

<span data-ttu-id="6789c-186">As informações a seguir também podem ser relevantes ao implementar esse padrão:</span><span class="sxs-lookup"><span data-stu-id="6789c-186">The following information may be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="6789c-187">[Diretrizes de cache](/azure/architecture/best-practices/caching).</span><span class="sxs-lookup"><span data-stu-id="6789c-187">[Caching Guidance](/azure/architecture/best-practices/caching).</span></span> <span data-ttu-id="6789c-188">Fornece informações adicionais sobre como você pode armazenar dados em cache em uma solução de nuvem, e os problemas que você deve considerar ao implementar um cache.</span><span class="sxs-lookup"><span data-stu-id="6789c-188">Provides additional information on how you can cache data in a cloud solution, and the issues that you should consider when you implement a cache.</span></span>

- <span data-ttu-id="6789c-189">[Primer de Consistência de Dados](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="6789c-189">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="6789c-190">Os aplicativos de nuvem geralmente usam dados distribuídos entre armazenamentos de dados.</span><span class="sxs-lookup"><span data-stu-id="6789c-190">Cloud applications typically use data that's spread across data stores.</span></span> <span data-ttu-id="6789c-191">Gerenciar e manter a consistência dos dados nesse ambiente são um aspecto crítico do sistema, especialmente a simultaneidade e os problemas de disponibilidade que podem surgir.</span><span class="sxs-lookup"><span data-stu-id="6789c-191">Managing and maintaining data consistency in this environment is a critical aspect of the system, particularly the concurrency and availability issues that can arise.</span></span> <span data-ttu-id="6789c-192">Este primer descreve problemas sobre a consistência entre dados distribuídos e resume como um aplicativo pode implementar a consistência eventual para manter a disponibilidade dos dados.</span><span class="sxs-lookup"><span data-stu-id="6789c-192">This primer describes issues about consistency across distributed data, and summarizes how an application can implement eventual consistency to maintain the availability of data.</span></span>
