---
title: Antipadrão de busca externa
description: Recuperar mais dados que o necessário para uma operação de negócios pode resultar em sobrecarga de E/S desnecessária e reduzir a capacidade de resposta.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7a72bfd3e4b2e206f3266a046fac2083224ecb4f
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30846596"
---
# <a name="extraneous-fetching-antipattern"></a><span data-ttu-id="c9efa-103">Antipadrão de busca externa</span><span class="sxs-lookup"><span data-stu-id="c9efa-103">Extraneous Fetching antipattern</span></span>

<span data-ttu-id="c9efa-104">Recuperar mais dados que o necessário para uma operação de negócios pode resultar em sobrecarga de E/S desnecessária e reduzir a capacidade de resposta.</span><span class="sxs-lookup"><span data-stu-id="c9efa-104">Retrieving more data than needed for a business operation can result in unnecessary I/O overhead and reduce responsiveness.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="c9efa-105">Descrição do problema</span><span class="sxs-lookup"><span data-stu-id="c9efa-105">Problem description</span></span>

<span data-ttu-id="c9efa-106">Este antipadrão pode ocorrer se o aplicativo tentar minimizar as solicitações de E/S, recuperando todos os dados que ele *possa* precisar.</span><span class="sxs-lookup"><span data-stu-id="c9efa-106">This antipattern can occur if the application tries to minimize I/O requests by retrieving all of the data that it *might* need.</span></span> <span data-ttu-id="c9efa-107">Isso geralmente é um resultado de sobrecompensação para antipadrão de [E/S informais][chatty-io].</span><span class="sxs-lookup"><span data-stu-id="c9efa-107">This is often a result of overcompensating for the [Chatty I/O][chatty-io] antipattern.</span></span> <span data-ttu-id="c9efa-108">Por exemplo, um aplicativo pode buscar os detalhes de cada produto em um banco de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-108">For example, an application might fetch the details for every product in a database.</span></span> <span data-ttu-id="c9efa-109">Mas o usuário pode ter apenas um subconjunto dos detalhes (alguns podem não ser relevantes para os clientes) e provavelmente não precisa ver *todos* os produtos de uma vez.</span><span class="sxs-lookup"><span data-stu-id="c9efa-109">But the user may need just a subset of the details (some may not be relevant to customers), and probably doesn't need to see *all* of the products at once.</span></span> <span data-ttu-id="c9efa-110">Mesmo se o usuário estiver navegando pelo catálogo inteiro, faria sentido paginar os resultados &mdash; mostrando 20 ao mesmo tempo, por exemplo.</span><span class="sxs-lookup"><span data-stu-id="c9efa-110">Even if the user is browsing the entire catalog, it would make sense to paginate the results &mdash; showing 20 at a time, for example.</span></span>

<span data-ttu-id="c9efa-111">Outra fonte desse problema é seguir práticas de programação ou design ruins.</span><span class="sxs-lookup"><span data-stu-id="c9efa-111">Another source of this problem is following poor programming or design practices.</span></span> <span data-ttu-id="c9efa-112">Por exemplo, o código a seguir usa o Entity Framework para buscar os detalhes completos de todos os produtos.</span><span class="sxs-lookup"><span data-stu-id="c9efa-112">For example, the following code uses Entity Framework to fetch the complete details for every product.</span></span> <span data-ttu-id="c9efa-113">Em seguida, ele filtra os resultados para retornar apenas um subconjunto dos campos, descartando o restante.</span><span class="sxs-lookup"><span data-stu-id="c9efa-113">Then it filters the results to return only a subset of the fields, discarding the rest.</span></span> <span data-ttu-id="c9efa-114">Você pode encontrar o exemplo completo [aqui][sample-app].</span><span class="sxs-lookup"><span data-stu-id="c9efa-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> GetAllFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Execute the query. This happens at the database.
        var products = await context.Products.ToListAsync();

        // Project fields from the query results. This happens in application memory.
        var result = products.Select(p => new ProductInfo { Id = p.ProductId, Name = p.Name });
        return Ok(result);
    }
}
```

<span data-ttu-id="c9efa-115">No exemplo a seguir, o aplicativo recupera dados para executar uma agregação que pode ser feita pelo banco de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-115">In the next example, the application retrieves data to perform an aggregation that could be done by the database instead.</span></span> <span data-ttu-id="c9efa-116">O aplicativo calcula o total de vendas obtendo todos os registros para todos os pedidos de venda e, em seguida, calculando a soma sobre esses registros.</span><span class="sxs-lookup"><span data-stu-id="c9efa-116">The application calculates total sales by getting every record for all orders sold, and then computing the sum over those records.</span></span> <span data-ttu-id="c9efa-117">Você pode encontrar o exemplo completo [aqui][sample-app].</span><span class="sxs-lookup"><span data-stu-id="c9efa-117">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnClientAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Fetch all order totals from the database.
        var orderAmounts = await context.SalesOrderHeaders.Select(soh => soh.TotalDue).ToListAsync();

        // Sum the order totals in memory.
        var total = orderAmounts.Sum();
        return Ok(total);
    }
}
```

<span data-ttu-id="c9efa-118">O exemplo a seguir mostra um problema sutil causado pelo modo como o Entity Framework utiliza o LINQ to Entities.</span><span class="sxs-lookup"><span data-stu-id="c9efa-118">The next example shows a subtle problem caused by the way Entity Framework uses LINQ to Entities.</span></span> 

```csharp
var query = from p in context.Products.AsEnumerable()
            where p.SellStartDate < DateTime.Now.AddDays(-7) // AddDays cannot be mapped by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

<span data-ttu-id="c9efa-119">O aplicativo está tentando localizar produtos com um `SellStartDate` com mais de uma semana.</span><span class="sxs-lookup"><span data-stu-id="c9efa-119">The application is trying to find products with a `SellStartDate` more than a week old.</span></span> <span data-ttu-id="c9efa-120">Na maioria dos casos, o LINQ to Entities converterá uma cláusula `where` para uma instrução SQL que é executada pelo banco de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-120">In most cases, LINQ to Entities would translate a `where` clause to a SQL statement that is executed by the database.</span></span> <span data-ttu-id="c9efa-121">No entanto, nesse caso, o LINQ to Entities não consegue mapear o método `AddDays` para SQL.</span><span class="sxs-lookup"><span data-stu-id="c9efa-121">In this case, however, LINQ to Entities cannot map the `AddDays` method to SQL.</span></span> <span data-ttu-id="c9efa-122">Em vez disso, cada linha da tabela `Product` será retornada e os resultados serão filtrados na memória.</span><span class="sxs-lookup"><span data-stu-id="c9efa-122">Instead, every row from the `Product` table is returned, and the results are filtered in memory.</span></span> 

<span data-ttu-id="c9efa-123">A chamada para `AsEnumerable` é uma dica de que há um problema.</span><span class="sxs-lookup"><span data-stu-id="c9efa-123">The call to `AsEnumerable` is a hint that there is a problem.</span></span> <span data-ttu-id="c9efa-124">Este método converte os resultados para uma interface `IEnumerable`.</span><span class="sxs-lookup"><span data-stu-id="c9efa-124">This method converts the results to an `IEnumerable` interface.</span></span> <span data-ttu-id="c9efa-125">Embora `IEnumerable` ofereça suporte à filtragem, ela é feita pelo *cliente*, e não pelo banco de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-125">Although `IEnumerable` supports filtering, the filtering is done on the *client* side, not the database.</span></span> <span data-ttu-id="c9efa-126">Por padrão, o LINQ to Entities usa `IQueryable`, que passa a responsabilidade da filtragem para a fonte de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-126">By default, LINQ to Entities uses `IQueryable`, which passes the responsibility for filtering to the data source.</span></span> 

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="c9efa-127">Como corrigir o problema</span><span class="sxs-lookup"><span data-stu-id="c9efa-127">How to fix the problem</span></span>

<span data-ttu-id="c9efa-128">Evite a busca de grandes volumes de dados que rapidamente podem ficar desatualizados ou ser descartados e busque somente os dados necessários para a operação que está sendo executada.</span><span class="sxs-lookup"><span data-stu-id="c9efa-128">Avoid fetching large volumes of data that may quickly become outdated or might be discarded, and only fetch the data needed for the operation being performed.</span></span> 

<span data-ttu-id="c9efa-129">Em vez de obter todas as colunas de uma tabela e, em seguida, filtrá-las, selecione as colunas que você precisa do banco de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-129">Instead of getting every column from a table and then filtering them, select the columns that you need from the database.</span></span>

```csharp
public async Task<IHttpActionResult> GetRequiredFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Project fields as part of the query itself
        var result = await context.Products
            .Select(p => new ProductInfo {Id = p.ProductId, Name = p.Name})
            .ToListAsync();
        return Ok(result);
    }
}
```

<span data-ttu-id="c9efa-130">Da mesma forma, execute a agregação no banco de dados e não na memória do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="c9efa-130">Similarly, perform aggregation in the database and not in application memory.</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnDatabaseAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Sum the order totals as part of the database query.
        var total = await context.SalesOrderHeaders.SumAsync(soh => soh.TotalDue);
        return Ok(total);
    }
}
```

<span data-ttu-id="c9efa-131">Ao usar o Entity Framework, verifique se as consultas do LINQ são resolvidas usando a interface `IQueryable` e não `IEnumerable`.</span><span class="sxs-lookup"><span data-stu-id="c9efa-131">When using Entity Framework, ensure that LINQ queries are resolved using the `IQueryable`interface and not `IEnumerable`.</span></span> <span data-ttu-id="c9efa-132">Talvez seja necessário ajustar a consulta para usar apenas as funções que podem ser mapeadas para a fonte de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-132">You may need to adjust the query to use only functions that can be mapped to the data source.</span></span> <span data-ttu-id="c9efa-133">O exemplo anterior pode ser refatorado para remover o método `AddDays` da consulta, permitindo que a filtragem seja feita pelo banco de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-133">The earlier example can be refactored to remove the `AddDays` method from the query, allowing filtering to be done by the database.</span></span>

```csharp
DateTime dateSince = DateTime.Now.AddDays(-7); // AddDays has been factored out. 
var query = from p in context.Products
            where p.SellStartDate < dateSince // This criterion can be passed to the database by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

## <a name="considerations"></a><span data-ttu-id="c9efa-134">Considerações</span><span class="sxs-lookup"><span data-stu-id="c9efa-134">Considerations</span></span>

- <span data-ttu-id="c9efa-135">Em alguns casos, você pode melhorar o desempenho através do particionamento horizontal de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-135">In some cases, you can improve performance by partitioning data horizontally.</span></span> <span data-ttu-id="c9efa-136">Se operações diferentes acessam atributos diferentes dos dados, o particionamento horizontal pode reduzir a contenção.</span><span class="sxs-lookup"><span data-stu-id="c9efa-136">If different operations access different attributes of the data, horizontal partitioning may reduce contention.</span></span> <span data-ttu-id="c9efa-137">Geralmente, a maioria das operações são executadas em relação a um pequeno subconjunto dos dados, para que a difusão desse carregamento possa melhorar o desempenho.</span><span class="sxs-lookup"><span data-stu-id="c9efa-137">Often, most operations are run against a small subset of the data, so spreading this load may improve performance.</span></span> <span data-ttu-id="c9efa-138">Confira o [Particionamento de dados][data-partitioning].</span><span class="sxs-lookup"><span data-stu-id="c9efa-138">See [Data partitioning][data-partitioning].</span></span>

- <span data-ttu-id="c9efa-139">Para operações que oferecem suporte a consultas não associadas, implemente a paginação e busque apenas um número limitado de entidades por vez.</span><span class="sxs-lookup"><span data-stu-id="c9efa-139">For operations that have to support unbounded queries, implement pagination and only fetch a limited number of entities at a time.</span></span> <span data-ttu-id="c9efa-140">Por exemplo, se um cliente estiver navegando por um catálogo de produtos, você pode mostrar uma página de resultados de cada vez.</span><span class="sxs-lookup"><span data-stu-id="c9efa-140">For example, if a customer is browsing a product catalog, you can show one page of results at a time.</span></span>

- <span data-ttu-id="c9efa-141">Quando possível, aproveite os recursos incluídos no armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-141">When possible, take advantage of features built into the data store.</span></span> <span data-ttu-id="c9efa-142">Por exemplo, bancos de dados SQL normalmente fornecem funções de agregação.</span><span class="sxs-lookup"><span data-stu-id="c9efa-142">For example, SQL databases typically provide aggregate functions.</span></span> 

- <span data-ttu-id="c9efa-143">Se você estiver usando um armazenamento de dados que não oferece suporte a uma função específica, como a agregação, poderia armazenar o resultado calculado em outro lugar, atualizando o valor conforma os registros são adicionados ou atualizados, para que o aplicativo não precise recalcular o valor sempre que necessário.</span><span class="sxs-lookup"><span data-stu-id="c9efa-143">If you're using a data store that doesn't support a particular function, such as aggregration, you could store the calculated result elsewhere, updating the value as records are added or updated, so the application doesn't have to recalculate the value each time it's needed.</span></span>

- <span data-ttu-id="c9efa-144">Se você observar que as solicitações estão recuperando um grande número de campos, examine o código-fonte para determinar se todos esses campos são realmente necessários.</span><span class="sxs-lookup"><span data-stu-id="c9efa-144">If you see that requests are retrieving a large number of fields, examine the source code to determine whether all of these fields are actually necessary.</span></span> <span data-ttu-id="c9efa-145">Às vezes, essas solicitações são o resultado de uma consulta `SELECT *` mal projetada.</span><span class="sxs-lookup"><span data-stu-id="c9efa-145">Sometimes these requests are the result of poorly designed `SELECT *` query.</span></span> 

- <span data-ttu-id="c9efa-146">Da mesma forma, as solicitações que recuperam um grande número de entidades podem ser sinal de que o aplicativo não está filtrando os dados corretamente.</span><span class="sxs-lookup"><span data-stu-id="c9efa-146">Similarly, requests that retrieve a large number of entities may be sign that the application is not filtering data correctly.</span></span> <span data-ttu-id="c9efa-147">Verifique se todas essas entidades são realmente necessárias.</span><span class="sxs-lookup"><span data-stu-id="c9efa-147">Verify that all of these entities are actually needed.</span></span> <span data-ttu-id="c9efa-148">Se possível, use o banco de dados da filtragem, por exemplo, usando cláusulas `WHERE` no SQL.</span><span class="sxs-lookup"><span data-stu-id="c9efa-148">Use database-side filtering if possible, for example, by using `WHERE` clauses in SQL.</span></span> 

- <span data-ttu-id="c9efa-149">O processamento de descarregamento para o banco de dados nem sempre é a melhor opção.</span><span class="sxs-lookup"><span data-stu-id="c9efa-149">Offloading processing to the database is not always the best option.</span></span> <span data-ttu-id="c9efa-150">Só use essa estratégia quando o banco de dados for criado ou otimizado para fazer isso.</span><span class="sxs-lookup"><span data-stu-id="c9efa-150">Only use this strategy when the database is designed or optimized to do so.</span></span> <span data-ttu-id="c9efa-151">A maioria dos sistemas de banco de dados são altamente otimizados para determinadas funções, mas não são projetados para atuar como mecanismos de uso geral do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="c9efa-151">Most database systems are highly optimized for certain functions, but are not designed to act as general-purpose application engines.</span></span> <span data-ttu-id="c9efa-152">Para obter mais informações, confira o [Antipadrão de Banco de Dados Ocupado][BusyDatabase].</span><span class="sxs-lookup"><span data-stu-id="c9efa-152">For more information, see the [Busy Database antipattern][BusyDatabase].</span></span>


## <a name="how-to-detect-the-problem"></a><span data-ttu-id="c9efa-153">Como detectar o problema</span><span class="sxs-lookup"><span data-stu-id="c9efa-153">How to detect the problem</span></span>

<span data-ttu-id="c9efa-154">Os sintomas de busca externa incluem latência alta e baixa taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="c9efa-154">Symptoms of extraneous fetching include high latency and low throughput.</span></span> <span data-ttu-id="c9efa-155">Se os dados são recuperados a partir de um armazenamento de dados, uma maior contenção também é provável.</span><span class="sxs-lookup"><span data-stu-id="c9efa-155">If the data is retrieved from a data store, increased contention is also probable.</span></span> <span data-ttu-id="c9efa-156">É provável que os usuários finais relatem tempos de resposta estendidos ou falhas causadas por serviços atingindo o tempo limite. Essas falhas podem retornar erros HTTP 500 (Servidor Interno) ou erros HTTP 503 (Serviço Indisponível).</span><span class="sxs-lookup"><span data-stu-id="c9efa-156">End users are likely to report extended response times or failures caused by services timing out. These failures could return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="c9efa-157">Examine os logs de eventos para o servidor Web, que provavelmente contêm informações mais detalhadas sobre as causas e as circunstâncias dos erros.</span><span class="sxs-lookup"><span data-stu-id="c9efa-157">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="c9efa-158">Os sintomas desse antipadrão e alguns de telemetria obtida podem ser muito semelhantes àqueles do [Antipadrão de Persistência Monolítica][MonolithicPersistence].</span><span class="sxs-lookup"><span data-stu-id="c9efa-158">The symptoms of this antipattern and some of the telemetry obtained might be very similar to those of the [Monolithic Persistence antipattern][MonolithicPersistence].</span></span> 

<span data-ttu-id="c9efa-159">Você pode executar as etapas a seguir para ajudar a identificar a causa:</span><span class="sxs-lookup"><span data-stu-id="c9efa-159">You can perform the following steps to help identify the cause:</span></span>

1. <span data-ttu-id="c9efa-160">Identifique as transações ou cargas de trabalho lentas realizando teste de carga, monitoramento de processos ou outros métodos de captura de dados de instrumentação.</span><span class="sxs-lookup"><span data-stu-id="c9efa-160">Identify slow workloads or transactions by performing load-testing, process monitoring, or other methods of capturing instrumentation data.</span></span>
2. <span data-ttu-id="c9efa-161">Observe os padrões comportamentais apresentados pelo sistema.</span><span class="sxs-lookup"><span data-stu-id="c9efa-161">Observe any behavioral patterns exhibited by the system.</span></span> <span data-ttu-id="c9efa-162">Há limites específicos em termos de transações por segundo ou volume de usuários?</span><span class="sxs-lookup"><span data-stu-id="c9efa-162">Are there particular limits in terms of transactions per second or volume of users?</span></span>
3. <span data-ttu-id="c9efa-163">Correlacione as instâncias de cargas de trabalho lentas com padrões comportamentais.</span><span class="sxs-lookup"><span data-stu-id="c9efa-163">Correlate the instances of slow workloads with behavioral patterns.</span></span>
4. <span data-ttu-id="c9efa-164">Identifique os armazenamentos de dados que estão sendo usados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-164">Identify the data stores being used.</span></span> <span data-ttu-id="c9efa-165">Para cada fonte de dados, execute a telemetria de baixo nível para observar o comportamento das operações.</span><span class="sxs-lookup"><span data-stu-id="c9efa-165">For each data source, run lower level telemetry to observe the behavior of operations.</span></span>
6. <span data-ttu-id="c9efa-166">Identifique as consultas de execução lenta que fazem referência a essas fontes de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-166">Identify any slow-running queries that reference these data sources.</span></span>
7. <span data-ttu-id="c9efa-167">Execute uma análise específica do recurso de consultas de execução lenta e determine como os dados são usados e consumidos.</span><span class="sxs-lookup"><span data-stu-id="c9efa-167">Perform a resource-specific analysis of the slow-running queries and ascertain how the data is used and consumed.</span></span>

<span data-ttu-id="c9efa-168">Procure esses sintomas:</span><span class="sxs-lookup"><span data-stu-id="c9efa-168">Look for any of these symptoms:</span></span>

- <span data-ttu-id="c9efa-169">Solicitações de E/S grandes e frequentes feitas para o mesmo recurso ou armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-169">Frequent, large I/O requests made to the same resource or data store.</span></span>
- <span data-ttu-id="c9efa-170">Contenção em um armazenamento de dados ou recurso compartilhado.</span><span class="sxs-lookup"><span data-stu-id="c9efa-170">Contention in a shared resource or data store.</span></span>
- <span data-ttu-id="c9efa-171">Uma operação que recebe frequentemente grandes volumes de dados através da rede.</span><span class="sxs-lookup"><span data-stu-id="c9efa-171">An operation that frequently receives large volumes of data over the network.</span></span>
- <span data-ttu-id="c9efa-172">Aplicativos e serviços gastando tempo significativo aguardando a conclusão da E/S.</span><span class="sxs-lookup"><span data-stu-id="c9efa-172">Applications and services spending significant time waiting for I/O to complete.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="c9efa-173">Diagnóstico de exemplo</span><span class="sxs-lookup"><span data-stu-id="c9efa-173">Example diagnosis</span></span>    

<span data-ttu-id="c9efa-174">As seções a seguir se aplicam a estas etapas para os exemplos anteriores.</span><span class="sxs-lookup"><span data-stu-id="c9efa-174">The following sections apply these steps to the previous examples.</span></span>

### <a name="identify-slow-workloads"></a><span data-ttu-id="c9efa-175">Identificar as cargas de trabalho lentas</span><span class="sxs-lookup"><span data-stu-id="c9efa-175">Identify slow workloads</span></span>

<span data-ttu-id="c9efa-176">Este gráfico mostra os resultados de desempenho de um teste de carga que simulou até 400 usuários simultâneos executando o método `GetAllFieldsAsync` mostrado anteriormente.</span><span class="sxs-lookup"><span data-stu-id="c9efa-176">This graph shows performance results from a load test that simulated up to 400 concurrent users running the `GetAllFieldsAsync` method shown earlier.</span></span> <span data-ttu-id="c9efa-177">A taxa de transferência lenta diminui à medida que a carga aumenta.</span><span class="sxs-lookup"><span data-stu-id="c9efa-177">Throughput diminishes slowly as the load increases.</span></span> <span data-ttu-id="c9efa-178">O tempo médio de resposta aumenta conforme a carga de trabalho aumenta.</span><span class="sxs-lookup"><span data-stu-id="c9efa-178">Average response time goes up as the workload increases.</span></span> 

![Resultados do teste de carga para o método GetAllFieldsAsync][Load-Test-Results-Client-Side1]

<span data-ttu-id="c9efa-180">Um teste de carga para a operação `AggregateOnClientAsync` mostra um padrão semelhante.</span><span class="sxs-lookup"><span data-stu-id="c9efa-180">A load test for the `AggregateOnClientAsync` operation shows a similar pattern.</span></span> <span data-ttu-id="c9efa-181">O volume das solicitações é razoavelmente estável.</span><span class="sxs-lookup"><span data-stu-id="c9efa-181">The volume of requests is reasonably stable.</span></span> <span data-ttu-id="c9efa-182">O tempo médio de resposta aumenta com a carga de trabalho, embora mais lentamente do que o gráfico anterior.</span><span class="sxs-lookup"><span data-stu-id="c9efa-182">The average response time increases with the workload, although more slowly than the previous graph.</span></span>

![Resultados do teste de carga para o método AggregateOnClientAsync][Load-Test-Results-Client-Side2]

### <a name="correlate-slow-workloads-with-behavioral-patterns"></a><span data-ttu-id="c9efa-184">Correlacione as cargas de trabalho lentas com os padrões comportamentais</span><span class="sxs-lookup"><span data-stu-id="c9efa-184">Correlate slow workloads with behavioral patterns</span></span>

<span data-ttu-id="c9efa-185">Qualquer correlação entre períodos normais de alto uso e diminuindo o desempenho pode indicar áreas de interesse.</span><span class="sxs-lookup"><span data-stu-id="c9efa-185">Any correlation between regular periods of high usage and slowing performance can indicate areas of concern.</span></span> <span data-ttu-id="c9efa-186">Examine atentamente o perfil de desempenho da funcionalidade suspeita de lenta execução para determinar se ele corresponde ao teste de carga executado anteriormente.</span><span class="sxs-lookup"><span data-stu-id="c9efa-186">Closely examine the performance profile of functionality that is suspected to be slow running, to determine whether it matches the load testing performed earlier.</span></span>

<span data-ttu-id="c9efa-187">Utilize um teste de carga com a mesma funcionalidade usando cargas de usuário com base na etapa para localizar o ponto em que o desempenho cai significativamente ou falha completamente.</span><span class="sxs-lookup"><span data-stu-id="c9efa-187">Load test the same functionality using step-based user loads, to find the point where performance drops significantly or fails completely.</span></span> <span data-ttu-id="c9efa-188">Se esse ponto está dentro dos limites de seu uso real esperado, examine como a funcionalidade é implementada.</span><span class="sxs-lookup"><span data-stu-id="c9efa-188">If that point falls within the bounds of your expected real-world usage, examine how the functionality is implemented.</span></span>

<span data-ttu-id="c9efa-189">Uma operação lenta não é necessariamente um problema, se ela não estiver sendo executada quando o sistema estiver sob carga excessiva, não é um tempo crítico e não afeta de forma negativa o desempenho de outras operações importantes.</span><span class="sxs-lookup"><span data-stu-id="c9efa-189">A slow operation is not necessarily a problem, if it is not being performed when the system is under stress, is not time critical, and does not negatively affect the performance of other important operations.</span></span> <span data-ttu-id="c9efa-190">Por exemplo, a geração de estatísticas operacionais mensais pode ser uma operação de longa execução, mas provavelmente pode ser executada como um processo em lote e executada como um trabalho de baixa prioridade.</span><span class="sxs-lookup"><span data-stu-id="c9efa-190">For example, generating monthly operational statistics might be a long-running operation, but it can probably be performed as a batch process and run as a low priority job.</span></span> <span data-ttu-id="c9efa-191">Por outro lado, os clientes consultando o catálogo de produtos é uma operação de negócios crítica.</span><span class="sxs-lookup"><span data-stu-id="c9efa-191">On the other hand, customers querying the product catalog is a critical business operation.</span></span> <span data-ttu-id="c9efa-192">Se concentre na telemetria gerada por essas operações críticas para ver como o desempenho varia durante os períodos de alto uso.</span><span class="sxs-lookup"><span data-stu-id="c9efa-192">Focus on the telemetry generated by these critical operations to see how the performance varies during periods of high usage.</span></span>

### <a name="identify-data-sources-in-slow-workloads"></a><span data-ttu-id="c9efa-193">Identificar as fontes de dados em cargas de trabalho lentas</span><span class="sxs-lookup"><span data-stu-id="c9efa-193">Identify data sources in slow workloads</span></span>

<span data-ttu-id="c9efa-194">Se você suspeitar que um serviço está com um desempenho ruim devido à forma como recupera os dados, investigue como o aplicativo interage com os repositórios que ele usa.</span><span class="sxs-lookup"><span data-stu-id="c9efa-194">If you suspect that a service is performing poorly because of the way it retrieves data, investigate how the application interacts with the repositories it uses.</span></span> <span data-ttu-id="c9efa-195">Monitore o sistema ao vivo para ver quais fontes são acessadas durante períodos de baixo desempenho.</span><span class="sxs-lookup"><span data-stu-id="c9efa-195">Monitor the live system to see which sources are accessed during periods of poor performance.</span></span> 

<span data-ttu-id="c9efa-196">Para cada fonte de dados, instrumente o sistema para capturar o seguinte:</span><span class="sxs-lookup"><span data-stu-id="c9efa-196">For each data source, instrument the system to capture the following:</span></span>

- <span data-ttu-id="c9efa-197">A frequência com a qual cada armazenamento de dados é acessado.</span><span class="sxs-lookup"><span data-stu-id="c9efa-197">The frequency that each data store is accessed.</span></span>
- <span data-ttu-id="c9efa-198">O volume de dados entrando e saindo do armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-198">The volume of data entering and exiting the data store.</span></span>
- <span data-ttu-id="c9efa-199">O tempo dessas operações, especialmente a latência de solicitações.</span><span class="sxs-lookup"><span data-stu-id="c9efa-199">The timing of these operations, especially the latency of requests.</span></span>
- <span data-ttu-id="c9efa-200">A natureza e a taxa de erros que ocorrem durante o acesso a cada armazenamento de dados sob carga típica.</span><span class="sxs-lookup"><span data-stu-id="c9efa-200">The nature and rate of any errors that occur while accessing each data store under typical load.</span></span>

<span data-ttu-id="c9efa-201">Compare essas informações com o volume de dados que está sendo retornado pelo aplicativo para o cliente.</span><span class="sxs-lookup"><span data-stu-id="c9efa-201">Compare this information against the volume of data being returned by the application to the client.</span></span> <span data-ttu-id="c9efa-202">Acompanhe a taxa do volume de dados retornado pelo armazenamento de dados contra o volume de dados retornado para o cliente.</span><span class="sxs-lookup"><span data-stu-id="c9efa-202">Track the ratio of the volume of data returned by the data store against the volume of data returned to the client.</span></span> <span data-ttu-id="c9efa-203">Se não houver nenhuma diferença grande, investigue para determinar se o aplicativo está buscando dados que não precisa.</span><span class="sxs-lookup"><span data-stu-id="c9efa-203">If there is any large disparity, investigate to determine whether the application is fetching data that it doesn't need.</span></span>

<span data-ttu-id="c9efa-204">Você poderá capturar esses dados ao observar o sistema ao vivo e o rastreamento de ciclo de vida de cada solicitação de usuário, ou pode modelar uma série de cargas de trabalho sintéticas e executá-las em um sistema de teste.</span><span class="sxs-lookup"><span data-stu-id="c9efa-204">You may be able to capture this data by observing the live system and tracing the lifecycle of each user request, or you can model a series of synthetic workloads and run them against a test system.</span></span>

<span data-ttu-id="c9efa-205">Os gráficos a seguir mostram a telemetria capturada usando [APM do New Relic][new-relic] durante um teste de carga do método `GetAllFieldsAsync`.</span><span class="sxs-lookup"><span data-stu-id="c9efa-205">The following graphs show telemetry captured using [New Relic APM][new-relic] during a load test of the `GetAllFieldsAsync` method.</span></span> <span data-ttu-id="c9efa-206">Observe a diferença entre os volumes de dados recebidos do banco de dados e as respostas HTTP correspondentes.</span><span class="sxs-lookup"><span data-stu-id="c9efa-206">Note the difference between the volumes of data received from the database and the corresponding HTTP responses.</span></span>

![Telemetria para o método 'GetAllFieldsAsync'][TelemetryAllFields]

<span data-ttu-id="c9efa-208">Para cada solicitação, o banco de dados retornou 80,503 bytes, mas a resposta ao cliente continha somente 19,855 bytes, cerca de 25% do tamanho da resposta de banco de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-208">For each request, the database returned 80,503 bytes, but the response to the client only contained 19,855 bytes, about 25% of the size of the database response.</span></span> <span data-ttu-id="c9efa-209">O tamanho dos dados retornados ao cliente pode variar dependendo do formato.</span><span class="sxs-lookup"><span data-stu-id="c9efa-209">The size of the data returned to the client can vary depending on the format.</span></span> <span data-ttu-id="c9efa-210">Para este teste de carga, o cliente solicitou dados JSON.</span><span class="sxs-lookup"><span data-stu-id="c9efa-210">For this load test, the client requested JSON data.</span></span> <span data-ttu-id="c9efa-211">O teste separado usando XML (não mostrado) tinha um tamanho de resposta de 35,655 bytes ou 44% do tamanho da resposta de banco de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-211">Separate testing using XML (not shown) had a response size of 35,655 bytes, or 44% of the size of the database response.</span></span>

<span data-ttu-id="c9efa-212">O teste de carga para o método `AggregateOnClientAsync` mostra mais resultados extremos.</span><span class="sxs-lookup"><span data-stu-id="c9efa-212">The load test for the `AggregateOnClientAsync` method shows more extreme results.</span></span> <span data-ttu-id="c9efa-213">Neste caso, cada teste executou uma consulta que recuperou mais de 280Kb de dados do banco de dados, mas a resposta JSON foi apenas 14 bytes.</span><span class="sxs-lookup"><span data-stu-id="c9efa-213">In this case, each test performed a query that retrieved over 280Kb of data from the database, but the JSON response was a mere 14 bytes.</span></span> <span data-ttu-id="c9efa-214">A grande diferença é porque o método calcula um resultado agregado de um grande volume de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-214">The wide disparity is because the method calculates an aggregated result from a large volume of data.</span></span>

![Telemetria para o método 'AggregateOnClientAsync'][TelemetryAggregateOnClient]

### <a name="identify-and-analyze-slow-queries"></a><span data-ttu-id="c9efa-216">Identificar e analisar consultas lentas</span><span class="sxs-lookup"><span data-stu-id="c9efa-216">Identify and analyze slow queries</span></span>

<span data-ttu-id="c9efa-217">Procure por consultas de banco de dados que consomem a maioria dos recursos e levam mais tempo para executar.</span><span class="sxs-lookup"><span data-stu-id="c9efa-217">Look for database queries that consume the most resources and take the most time to execute.</span></span> <span data-ttu-id="c9efa-218">Você pode adicionar a instrumentação para localizar os horários de início e conclusão para várias operações de banco de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-218">You can add instrumentation to find the start and completion times for many database operations.</span></span> <span data-ttu-id="c9efa-219">Vários armazenamentos de dados também oferecem informações detalhadas sobre como as consultas são executadas e otimizadas.</span><span class="sxs-lookup"><span data-stu-id="c9efa-219">Many data stores also provide in-depth information on how queries are performed and optimized.</span></span> <span data-ttu-id="c9efa-220">Por exemplo, o painel de Desempenho da Consulta no portal de gerenciamento de banco de Dados SQL do Azure permite-lhe selecionar uma consulta e exibir informações detalhadas do desempenho de tempo de execução.</span><span class="sxs-lookup"><span data-stu-id="c9efa-220">For example, the Query Performance pane in the Azure SQL Database management portal lets you select a query and view detailed runtime performance information.</span></span> <span data-ttu-id="c9efa-221">Aqui está a consulta gerada pela operação `GetAllFieldsAsync`:</span><span class="sxs-lookup"><span data-stu-id="c9efa-221">Here is the query generated by the `GetAllFieldsAsync` operation:</span></span>

![O painel de Detalhes da Consulta no portal de gerenciamento de banco de dados SQL do Windows Azure][QueryDetails]


## <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="c9efa-223">Implementar a solução e verificar o resultado</span><span class="sxs-lookup"><span data-stu-id="c9efa-223">Implement the solution and verify the result</span></span>

<span data-ttu-id="c9efa-224">Depois de alterar o método `GetRequiredFieldsAsync` para usar uma instrução SELECT no lado do banco de dados, os testes de carga mostraram os seguintes resultados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-224">After changing the `GetRequiredFieldsAsync` method to use a SELECT statement on the database side, load testing showed the following results.</span></span>

![Resultados do teste de carga para o método GetRequiredFieldsAsync][Load-Test-Results-Database-Side1]

<span data-ttu-id="c9efa-226">Este teste de carga utilizou a mesma implantação e a mesma carga de trabalho simulada de 400 usuários simultâneos que antes.</span><span class="sxs-lookup"><span data-stu-id="c9efa-226">This load test used the same deployment and the same simulated workload of 400 concurrent users as before.</span></span> <span data-ttu-id="c9efa-227">O gráfico mostra uma latência muito mais baixa.</span><span class="sxs-lookup"><span data-stu-id="c9efa-227">The graph shows much lower latency.</span></span> <span data-ttu-id="c9efa-228">O tempo de resposta aumenta com a carga aproximadamente 1,3 segundos, comparada a 4 segundos no caso anterior.</span><span class="sxs-lookup"><span data-stu-id="c9efa-228">Response time rises with load to approximately 1.3 seconds, compared to 4 seconds in the previous case.</span></span> <span data-ttu-id="c9efa-229">A taxa de transferência também é superior a 350 solicitações por segundo comparada a 100 anteriormente.</span><span class="sxs-lookup"><span data-stu-id="c9efa-229">The throughput is also higher at 350 requests per second compared to 100 earlier.</span></span> <span data-ttu-id="c9efa-230">O volume de dados recuperado do banco de dados agora intimamente corresponde ao tamanho das mensagens de resposta HTTP.</span><span class="sxs-lookup"><span data-stu-id="c9efa-230">The volume of data retrieved from the database now closely matches the size of the HTTP response messages.</span></span>

![Telemetria para o método 'GetRequiredFieldsAsync'][TelemetryRequiredFields]

<span data-ttu-id="c9efa-232">Os testes de carga usando o método `AggregateOnDatabaseAsync` geram os seguintes resultados:</span><span class="sxs-lookup"><span data-stu-id="c9efa-232">Load testing using the `AggregateOnDatabaseAsync` method generates the following results:</span></span>

![Resultados do teste de carga para o método AggregateOnDatabaseAsync][Load-Test-Results-Database-Side2]

<span data-ttu-id="c9efa-234">O tempo médio de resposta é agora mínimo.</span><span class="sxs-lookup"><span data-stu-id="c9efa-234">The average response time is now minimal.</span></span> <span data-ttu-id="c9efa-235">Esse é um aprimoramento de ordem de magnitude no desempenho, causado principalmente pela grande redução da E/S do banco de dados.</span><span class="sxs-lookup"><span data-stu-id="c9efa-235">This is an order of magnitude improvement in performance, caused primarily by the large reduction in I/O from the database.</span></span> 

<span data-ttu-id="c9efa-236">Aqui está a telemetria correspondente para o método `AggregateOnDatabaseAsync`.</span><span class="sxs-lookup"><span data-stu-id="c9efa-236">Here is the corresponding telemetry for the `AggregateOnDatabaseAsync` method.</span></span> <span data-ttu-id="c9efa-237">Foi reduzida consideravelmente a quantidade de dados recuperados do banco de dados, de 280Kb por transação a 53 bytes.</span><span class="sxs-lookup"><span data-stu-id="c9efa-237">The amount of data retrieved from the database was vastly reduced, from over 280Kb per transaction to 53 bytes.</span></span> <span data-ttu-id="c9efa-238">Como resultado, o número máximo prolongado de solicitações por minuto foi aumentado de cerca de 2.000 para mais de 25.000.</span><span class="sxs-lookup"><span data-stu-id="c9efa-238">As a result, the maximum sustained number of requests per minute was raised from around 2,000 to over 25,000.</span></span>

![Telemetria para o método 'AggregateOnDatabaseAsync'][TelemetryAggregateInDatabaseAsync]


## <a name="related-resources"></a><span data-ttu-id="c9efa-240">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="c9efa-240">Related resources</span></span>

- <span data-ttu-id="c9efa-241">[Antipadrão de Banco de Dados Ocupado][BusyDatabase]</span><span class="sxs-lookup"><span data-stu-id="c9efa-241">[Busy Database antipattern][BusyDatabase]</span></span>
- <span data-ttu-id="c9efa-242">[Antipadrão de E/S informais][chatty-io]</span><span class="sxs-lookup"><span data-stu-id="c9efa-242">[Chatty I/O antipattern][chatty-io]</span></span>
- <span data-ttu-id="c9efa-243">[Práticas recomendadas de particionamento de dados][data-partitioning]</span><span class="sxs-lookup"><span data-stu-id="c9efa-243">[Data partitioning best practices][data-partitioning]</span></span>


[BusyDatabase]: ../busy-database/index.md
[data-partitioning]: ../../best-practices/data-partitioning.md
[new-relic]: https://newrelic.com/application-monitoring

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ExtraneousFetching

[chatty-io]: ../chatty-io/index.md
[MonolithicPersistence]: ../monolithic-persistence/index.md
[Load-Test-Results-Client-Side1]:_images/LoadTestResultsClientSide1.jpg
[Load-Test-Results-Client-Side2]:_images/LoadTestResultsClientSide2.jpg
[Load-Test-Results-Database-Side1]:_images/LoadTestResultsDatabaseSide1.jpg
[Load-Test-Results-Database-Side2]:_images/LoadTestResultsDatabaseSide2.jpg
[QueryPerformanceZoomed]: _images/QueryPerformanceZoomed.jpg
[QueryDetails]: _images/QueryDetails.jpg
[TelemetryAllFields]: _images/TelemetryAllFields.jpg
[TelemetryAggregateOnClient]: _images/TelemetryAggregateOnClient.jpg
[TelemetryRequiredFields]: _images/TelemetryRequiredFields.jpg
[TelemetryAggregateInDatabaseAsync]: _images/TelemetryAggregateInDatabase.jpg
