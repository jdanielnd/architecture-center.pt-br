---
title: Cache-Aside
description: Carregar dados sob demanda em um cache de um armazenamento de dados
keywords: "padrão de design"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: e0a6a91fda6ea43236f6eea552f7b8f8d31160ad
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="cache-aside-pattern"></a>Padrão Cache-Aside

[!INCLUDE [header](../_includes/header.md)]

Carregar dados sob demanda em um cache de um armazenamento de dados. Isso pode melhorar o desempenho e também ajuda a manter a consistência entre dados mantidos no cache e os dados no armazenamento de dados subjacente.

## <a name="context-and-problem"></a>Contexto e problema

Os aplicativos usam um cache para melhorar o acesso repetido às informações mantidas em um armazenamento de dados. No entanto, não é prático esperar que os dados armazenados em cache sejam sempre completamente consistentes com os dados no armazenamento de dados. Os aplicativos devem implementar uma estratégia que ajude a garantir que os dados no cache sejam tão atualizados quanto possível, mas também possam detectar e lidar com situações que possam surgir quando os dados no cache se tornarem obsoletos.

## <a name="solution"></a>Solução

Muitos sistemas de armazenamento em cache comerciais fornecem operações de read-through e write-through/write-behind. Nesses sistemas, um aplicativo recupera dados referenciando o cache. Se os dados não estiverem no cache, ele é recuperado do armazenamento de dados e adicionado ao cache. Todas as modificações nos dados mantidos no cache são automaticamente gravadas no armazenamento de dados também.

Para caches que não têm essa funcionalidade, é de responsabilidade dos aplicativos que usam o cache manter os dados.

Um aplicativo pode emular a funcionalidade de cache de read-through implementando a estratégia de cache-aside. Essa estratégia carrega os dados em cache sob demanda. A figura ilustra usando o padrão de Cache-Aside para armazenar dados no cache.

![Usar o padrão de Cache-Aside para armazenar dados no cache](./_images/cache-aside-diagram.png)


Se um aplicativo atualiza as informações, ele pode seguir a estratégia de write-through modificando o armazenamento de dados e invalidando o item correspondente no cache.

Quando o item é necessário a seguir, usar a estratégia de cache-aside fará com que os dados atualizados sejam recuperados no armazenamento de dados e adicionados novamente ao cache.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão: 

**Tempo de vida dos dados armazenados em cache**. Vários caches implementam uma política de expiração que invalida os dados e os remove do cache se ele não for acessado por um período especificado. Para o cache-aside ser efetivado, verifique se a política de expiração corresponde ao padrão de acesso para aplicativos que usam os dados. Não faça o período de validade muito curto, porque isso pode fazer com que os aplicativos recuperem continuamente os dados do armazenamento de dados e adicionem-nos ao cache. Da mesma forma, não faça o período de validade longo demais a ponto dos dados em cache se tornarem provavelmente obsoletos. Lembre-se de que o cache é mais eficiente para dados relativamente estáticos ou dados lidos com frequência.

**Removendo dados**. A maioria dos caches tem um tamanho limitado em comparação com o armazenamento de dados de onde os dados são originários, e eles removerão os dados, se necessário. A maioria dos caches adotam uma política de usados menos recentemente para selecionar itens a serem removidos, mas isso pode ser personalizável. Configure a propriedade global de expiração e outras propriedades do cache, assim como a propriedade de expiração de cada item em cache, para garantir que o cache seja econômico. Nem sempre é apropriado aplicar uma política de remoção global para cada item no cache. Por exemplo, se um item em cache for muito caro para recuperar no armazenamento de dados, pode ser benéfico manter esse item no cache e dar preferência a itens acessados com mais frequência, porém mais baratos.

**Desobstrução do cache**. Muitas soluções preenchem o cache com os dados que um aplicativo pode precisar como parte do processo de inicialização. O padrão de Cache-Aside ainda pode ser útil se alguns desses dados expirar ou for removido.

**Consistência**. Implementar o padrão de Cache-Aside não garante a consistência entre o armazenamento de dados e o cache. Um item no armazenamento de dados pode ser alterado a qualquer momento por um processo externo, e essa alteração não pode ser refletida no cache até a próxima vez que o item seja carregado. Em um sistema que replica os dados por vários armazenamentos de dados, esse problema pode se tornar sério se a sincronização ocorrer com frequência.

**Armazenamento em cache local (na memória)**. Um cache pode ser local para uma instância de aplicativo e armazenado na memória. O cache-aside pode ser útil nesse ambiente, se um aplicativo acessar os mesmos dados repetidamente. No entanto, um cache local é privado e, por isso, diferentes instâncias do aplicativo podem ter uma cópia dos mesmos dados armazenados em cache. Esses dados podem rapidamente se tornar inconsistentes entre os caches, por isso é necessário expirar os dados mantidos em um cache privado e atualizá-los com mais frequência. Nesses cenários, considere a possibilidade de investigar o uso de um mecanismo de cache compartilhado ou distribuído.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use esse padrão quando:

- Um cache não fornece read-through nativo e as operações de write-through.
- A demanda de recursos é imprevisível. Esse padrão permite que os aplicativos carreguem dados sob demanda. Ele não faz nenhuma suposição sobre quais dados um aplicativo exigirá antecipadamente.

Esse padrão pode não ser adequado:

- Quando o conjunto de dados armazenados em cache é estático. Se os dados se ajustarem ao espaço em cache disponível, desobstrua o cache com os dados na inicialização e aplique uma política que impeça os dados de expirar.
- Para armazenar em cache informações de estado de sessão em um aplicativo Web hospedado em um web farm. Nesse ambiente, você deve evitar introduzir dependências baseadas na afinidade de cliente-servidor.

## <a name="example"></a>Exemplo

No Microsoft Azure você pode usar o Cache Redis do Azure para criar um cache distribuído que pode ser compartilhado por várias instâncias de um aplicativo. 

Para se conectar a uma instância do Cache Redis do Azure, chame o método estático `Connect`e passe a cadeia de conexão. O método retorna um `ConnectionMultiplexer` que representa a conexão. Uma abordagem para compartilhar uma instância do `ConnectionMultiplexer` em seu aplicativo deve ter uma propriedade estática que retorna uma instância conectada, semelhante ao exemplo a seguir. Essa abordagem oferece uma maneira thread-safe de inicializar somente uma única instância conectada.

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

O método `GetMyEntityAsync` no exemplo de código a seguir mostra uma implementação do padrão Cache-Aside com base no Cache Redis do Azure. Esse método recupera um objeto do cache usando a abordagem read-though.

Um objeto é identificado usando uma ID inteira como chave. O método `GetMyEntityAsync` tenta recuperar um item com essa chave do cache. Se um item correspondente for encontrado, ele é retornado. Se não houver nenhuma correspondência no cache, o método `GetMyEntityAsync` recupera o objeto de um armazenamento de dados, adiciona-o ao cache e, em seguida, retorna-o. O código que realmente lê os dados do armazenamento de dados não é mostrado aqui, pois ele depende do armazenamento de dados. Observe que o item de cache é configurado para expirar, a fim de impedir que ele se torne obsoleto, caso tenha se atualizado em outro lugar.


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
    // Code has been omitted because it's data store dependent.  
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

>  Os exemplos usam a API de Cache Redis do Azure para acessar o armazenamento e recuperar as informações do cache. Para obter mais informações, confira [Como usar o Cache Redis do Microsoft Azure](https://docs.microsoft.com/en-us/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) e [Como criar um aplicativo Web com o Cache Redis](https://docs.microsoft.com/en-us/azure/redis-cache/cache-web-app-howto)

O método `UpdateEntityAsync` mostrado a seguir demonstra como invalidar um objeto no cache quando o valor é alterado pelo aplicativo. O código atualiza o armazenamento de dados original e, em seguida, remove o item do cache.

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
> A ordem das etapas é importante. Atualize o armazenamento de dados *antes* de remover o item do cache. Se você remover o item do cache primeiro, há uma pequena janela de tempo quando um cliente pode obter o item antes de atualizar o armazenamento de dados. Isso resultará na perda de cache (porque o item foi removido do cache), fazendo com que a versão anterior do item seja obtida no repositório de dados e adicionado de volta ao cache. O resultado será dados obsoletos do cache.


## <a name="related-guidance"></a>Diretrizes relacionadas 

As informações a seguir também podem ser relevantes ao implementar esse padrão:

- [Diretrizes de cache](https://docs.microsoft.com/en-us/azure/architecture/best-practices/caching). Fornece informações adicionais sobre como você pode armazenar dados em cache em uma solução de nuvem, e os problemas que você deve considerar ao implementar um cache.

- [Primer de Consistência de Dados](https://msdn.microsoft.com/library/dn589800.aspx). Os aplicativos de nuvem geralmente usam dados distribuídos entre armazenamentos de dados. Gerenciar e manter a consistência dos dados nesse ambiente são um aspecto crítico do sistema, especialmente a simultaneidade e os problemas de disponibilidade que podem surgir. Este primer descreve problemas sobre a consistência entre dados distribuídos e resume como um aplicativo pode implementar a consistência eventual para manter a disponibilidade dos dados.
