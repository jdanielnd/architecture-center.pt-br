---
title: Antipadrão de E/S com ruídos
description: Um grande número de solicitações de E/S pode prejudicar o desempenho e a capacidade de resposta.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 17193198918cc742b2e3f30e77dfc5c3f2726ebf
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428560"
---
# <a name="chatty-io-antipattern"></a><span data-ttu-id="263f2-103">Antipadrão de E/S com ruídos</span><span class="sxs-lookup"><span data-stu-id="263f2-103">Chatty I/O antipattern</span></span>

<span data-ttu-id="263f2-104">O efeito cumulativo de um grande número de solicitações de E/S pode ter um impacto significativo no desempenho e capacidade de resposta.</span><span class="sxs-lookup"><span data-stu-id="263f2-104">The cumulative effect of a large number of I/O requests can have a significant impact on performance and responsiveness.</span></span>

## <a name="problem-description"></a><span data-ttu-id="263f2-105">Descrição do problema</span><span class="sxs-lookup"><span data-stu-id="263f2-105">Problem description</span></span>

<span data-ttu-id="263f2-106">As chamadas de rede e outras operações de E/S são inerentemente lentas em comparação com as tarefas de computação.</span><span class="sxs-lookup"><span data-stu-id="263f2-106">Network calls and other I/O operations are inherently slow compared to compute tasks.</span></span> <span data-ttu-id="263f2-107">Cada solicitação de E/S geralmente tem uma sobrecarga significativa e o efeito cumulativo de várias operações de E/S pode reduzir a velocidade do sistema.</span><span class="sxs-lookup"><span data-stu-id="263f2-107">Each I/O request typically has significant overhead, and the cumulative effect of numerous I/O operations can slow down the system.</span></span> <span data-ttu-id="263f2-108">Veja algumas causas comuns de E/S com ruídos.</span><span class="sxs-lookup"><span data-stu-id="263f2-108">Here are some common causes of chatty I/O.</span></span>

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a><span data-ttu-id="263f2-109">Ler e gravar registros individuais em um banco de dados como solicitações distintas</span><span class="sxs-lookup"><span data-stu-id="263f2-109">Reading and writing individual records to a database as distinct requests</span></span>

<span data-ttu-id="263f2-110">O exemplo a seguir lê em um banco de dados de produtos.</span><span class="sxs-lookup"><span data-stu-id="263f2-110">The following example reads from a database of products.</span></span> <span data-ttu-id="263f2-111">Há três tabelas: `Product`, `ProductSubcategory` e `ProductPriceListHistory`.</span><span class="sxs-lookup"><span data-stu-id="263f2-111">There are three tables, `Product`, `ProductSubcategory`, and `ProductPriceListHistory`.</span></span> <span data-ttu-id="263f2-112">O código recupera todos os produtos em uma subcategoria, junto com as informações de preços, executando uma série de consultas:</span><span class="sxs-lookup"><span data-stu-id="263f2-112">The code retrieves all of the products in a subcategory, along with the pricing information, by executing a series of queries:</span></span>  

1. <span data-ttu-id="263f2-113">Consulte a subcategoria na tabela `ProductSubcategory`.</span><span class="sxs-lookup"><span data-stu-id="263f2-113">Query the subcategory from the `ProductSubcategory` table.</span></span>
2. <span data-ttu-id="263f2-114">Localize todos os produtos nessa subcategoria consultando a tabela `Product`.</span><span class="sxs-lookup"><span data-stu-id="263f2-114">Find all products in that subcategory by querying the `Product` table.</span></span>
3. <span data-ttu-id="263f2-115">Para cada produto, consulte os dados de preço na tabela `ProductPriceListHistory`.</span><span class="sxs-lookup"><span data-stu-id="263f2-115">For each product, query the pricing data from the `ProductPriceListHistory` table.</span></span>

<span data-ttu-id="263f2-116">O aplicativo usa o [Entity Framework][ef] para consultar o banco de dados.</span><span class="sxs-lookup"><span data-stu-id="263f2-116">The application uses [Entity Framework][ef] to query the database.</span></span> <span data-ttu-id="263f2-117">Você pode encontrar o exemplo completo [aqui][code-sample].</span><span class="sxs-lookup"><span data-stu-id="263f2-117">You can find the complete sample [here][code-sample].</span></span> 

```csharp
public async Task<IHttpActionResult> GetProductsInSubCategoryAsync(int subcategoryId)
{
    using (var context = GetContext())
    {
        // Get product subcategory.
        var productSubcategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subcategoryId)
                .FirstOrDefaultAsync();

        // Find products in that category.
        productSubcategory.Product = await context.Products
            .Where(p => subcategoryId == p.ProductSubcategoryId)
            .ToListAsync();

        // Find price history for each product.
        foreach (var prod in productSubcategory.Product)
        {
            int productId = prod.ProductId;
            var productListPriceHistory = await context.ProductListPriceHistory
                .Where(pl => pl.ProductId == productId)
                .ToListAsync();
            prod.ProductListPriceHistory = productListPriceHistory;
        }
        return Ok(productSubcategory);
    }
}
```

<span data-ttu-id="263f2-118">Esse exemplo mostra o problema explicitamente, mas, às vezes, um O/RM pode mascarar o problema se ele busca implicitamente um registro filho por vez.</span><span class="sxs-lookup"><span data-stu-id="263f2-118">This example shows the problem explicitly, but sometimes an O/RM can mask the problem, if it implicitly fetches child records one at a time.</span></span> <span data-ttu-id="263f2-119">Isso é conhecido como "problema N+1".</span><span class="sxs-lookup"><span data-stu-id="263f2-119">This is known as the "N+1 problem".</span></span> 

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a><span data-ttu-id="263f2-120">Implementando uma única operação lógica como uma série de solicitações HTTP</span><span class="sxs-lookup"><span data-stu-id="263f2-120">Implementing a single logical operation as a series of HTTP requests</span></span>

<span data-ttu-id="263f2-121">Isso geralmente acontece quando os desenvolvedores tentam seguir um paradigma orientado a objetos e tratam os objetos remotos como se fossem objetos locais na memória.</span><span class="sxs-lookup"><span data-stu-id="263f2-121">This often happens when developers try to follow an object-oriented paradigm, and treat remote objects as if they were local objects in memory.</span></span> <span data-ttu-id="263f2-122">Também pode resultar em muitas viagens de ida e volta da rede.</span><span class="sxs-lookup"><span data-stu-id="263f2-122">This can result in too many network round trips.</span></span> <span data-ttu-id="263f2-123">Por exemplo, a seguinte API da Web expõe as propriedades individuais dos objetos `User` por meio de métodos HTTP GET individuais.</span><span class="sxs-lookup"><span data-stu-id="263f2-123">For example, the following web API exposes the individual properties of `User` objects through individual HTTP GET methods.</span></span> 

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}/username")]
    public HttpResponseMessage GetUserName(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/gender")]
    public HttpResponseMessage GetGender(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/dateofbirth")]
    public HttpResponseMessage GetDateOfBirth(int id)
    {
        ...
    }
}
```

<span data-ttu-id="263f2-124">Embora não haja nada tecnicamente errado com essa abordagem, a maioria dos clientes provavelmente precisará obter várias propriedades de cada `User`, resultando em um código do cliente, como a seguir.</span><span class="sxs-lookup"><span data-stu-id="263f2-124">While there's nothing technically wrong with this approach, most clients will probably need to get several properties for each `User`, resulting in client code like the following.</span></span> 

```csharp
HttpResponseMessage response = await client.GetAsync("users/1/username");
response.EnsureSuccessStatusCode();
var userName = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/gender");
response.EnsureSuccessStatusCode();
var gender = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/dateofbirth");
response.EnsureSuccessStatusCode();
var dob = await response.Content.ReadAsStringAsync();
```

### <a name="reading-and-writing-to-a-file-on-disk"></a><span data-ttu-id="263f2-125">Lendo e gravando em um arquivo no disco</span><span class="sxs-lookup"><span data-stu-id="263f2-125">Reading and writing to a file on disk</span></span>

<span data-ttu-id="263f2-126">A E/S do arquivo envolve abrir um arquivo e ir para o ponto apropriado antes de ler ou gravar os dados.</span><span class="sxs-lookup"><span data-stu-id="263f2-126">File I/O involves opening a file and moving to the appropriate point before reading or writing data.</span></span> <span data-ttu-id="263f2-127">Quando a operação é concluída, o arquivo pode ser fechado para salvar os recursos do sistema operacional.</span><span class="sxs-lookup"><span data-stu-id="263f2-127">When the operation is complete, the file might be closed to save operating system resources.</span></span> <span data-ttu-id="263f2-128">Um aplicativo que lê e grava continuamente pequenas quantidades de informações em um arquivo irá gerar uma sobrecarga significativa de E/S.</span><span class="sxs-lookup"><span data-stu-id="263f2-128">An application that continually reads and writes small amounts of information to a file will generate significant I/O overhead.</span></span> <span data-ttu-id="263f2-129">Pequenas solicitações de gravação também podem causar a fragmentação do arquivo, reduzindo ainda mais a velocidade das operações de E/S subsequentes.</span><span class="sxs-lookup"><span data-stu-id="263f2-129">Small write requests can also lead to file fragmentation, slowing subsequent I/O operations still further.</span></span> 

<span data-ttu-id="263f2-130">O exemplo a seguir usa um `FileStream` para gravar um objeto `Customer` em um arquivo.</span><span class="sxs-lookup"><span data-stu-id="263f2-130">The following example uses a `FileStream` to write a `Customer` object to a file.</span></span> <span data-ttu-id="263f2-131">Criar o `FileStream` abre o arquivo e descartá-lo fecha o arquivo.</span><span class="sxs-lookup"><span data-stu-id="263f2-131">Creating the `FileStream` opens the file, and disposing it closes the file.</span></span> <span data-ttu-id="263f2-132">(A instrução `using` descarta automaticamente o objeto `FileStream`.) Se o aplicativo chamar esse método repetidamente à medida que novos clientes são adicionados, a sobrecarga de E/S poderá acumular-se rapidamente.</span><span class="sxs-lookup"><span data-stu-id="263f2-132">(The `using` statement automatically disposes the `FileStream` object.) If the application calls this method repeatedly as new customers are added, the I/O overhead can accumulate quickly.</span></span>

```csharp
private async Task SaveCustomerToFileAsync(Customer cust)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        byte [] data = null;
        using (MemoryStream memStream = new MemoryStream())
        {
            formatter.Serialize(memStream, cust);
            data = memStream.ToArray();
        }
        await fileStream.WriteAsync(data, 0, data.Length);
    }
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="263f2-133">Como corrigir o problema</span><span class="sxs-lookup"><span data-stu-id="263f2-133">How to fix the problem</span></span>

<span data-ttu-id="263f2-134">Reduza o número de solicitações de E/S colocando os dados em menos solicitações maiores.</span><span class="sxs-lookup"><span data-stu-id="263f2-134">Reduce the number of I/O requests by packaging the data into larger, fewer requests.</span></span>

<span data-ttu-id="263f2-135">Obtenha os dados em um banco de dados como uma única consulta, em vez de várias consultas menores.</span><span class="sxs-lookup"><span data-stu-id="263f2-135">Fetch data from a database as a single query, instead of several smaller queries.</span></span> <span data-ttu-id="263f2-136">Veja uma versão revisada do código que recupera as informações do produto.</span><span class="sxs-lookup"><span data-stu-id="263f2-136">Here's a revised version of the code that retrieves product information.</span></span>

```csharp
public async Task<IHttpActionResult> GetProductCategoryDetailsAsync(int subCategoryId)
{
    using (var context = GetContext())
    {
        var subCategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subCategoryId)
                .Include("Product.ProductListPriceHistory")
                .FirstOrDefaultAsync();

        if (subCategory == null)
            return NotFound();

        return Ok(subCategory);
    }
}
```

<span data-ttu-id="263f2-137">Siga os princípios de design REST para as APIs da Web.</span><span class="sxs-lookup"><span data-stu-id="263f2-137">Follow REST design principles for web APIs.</span></span> <span data-ttu-id="263f2-138">Veja uma versão revisada de API da Web do exemplo anterior.</span><span class="sxs-lookup"><span data-stu-id="263f2-138">Here's a revised version of the web API from the earlier example.</span></span> <span data-ttu-id="263f2-139">Em vez de métodos GET separados para cada propriedade, há um único método GET que retorna `User`.</span><span class="sxs-lookup"><span data-stu-id="263f2-139">Instead of separate GET methods for each property, there is a single GET method that returns the `User`.</span></span> <span data-ttu-id="263f2-140">Isso resulta em um corpo de resposta maior por solicitação, mas é provável que cada cliente faça menos chamadas à API.</span><span class="sxs-lookup"><span data-stu-id="263f2-140">This results in a larger response body per request, but each client is likely to make fewer API calls.</span></span>

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}")]
    public HttpResponseMessage GetUser(int id)
    {
        ...
    }
}

// Client code
HttpResponseMessage response = await client.GetAsync("users/1");
response.EnsureSuccessStatusCode();
var user = await response.Content.ReadAsStringAsync();
```

<span data-ttu-id="263f2-141">Para a E/S do arquivo, considere colocar em buffer os dados na memória e, em seguida, gravar os dados em buffer em um arquivo como uma única operação.</span><span class="sxs-lookup"><span data-stu-id="263f2-141">For file I/O, consider buffering data in memory and then writing the buffered data to a file as a single operation.</span></span> <span data-ttu-id="263f2-142">Essa abordagem reduz a sobrecarga de abrir e fechar repetidamente o arquivo e ajuda a reduzir a fragmentação do arquivo no disco.</span><span class="sxs-lookup"><span data-stu-id="263f2-142">This approach reduces the overhead from repeatedly opening and closing the file, and helps to reduce fragmentation of the file on disk.</span></span>

```csharp
// Save a list of customer objects to a file
private async Task SaveCustomerListToFileAsync(List<Customer> customers)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        foreach (var cust in customers)
        {
            byte[] data = null;
            using (MemoryStream memStream = new MemoryStream())
            {
                formatter.Serialize(memStream, cust);
                data = memStream.ToArray();
            }
            await fileStream.WriteAsync(data, 0, data.Length);
        }
    }
}

// In-memory buffer for customers.
List<Customer> customers = new List<Customers>();

// Create a new customer and add it to the buffer
var cust = new Customer(...);
customers.Add(cust);

// Add more customers to the list as they are created
...

// Save the contents of the list, writing all customers in a single operation
await SaveCustomerListToFileAsync(customers);
```

## <a name="considerations"></a><span data-ttu-id="263f2-143">Considerações</span><span class="sxs-lookup"><span data-stu-id="263f2-143">Considerations</span></span>

- <span data-ttu-id="263f2-144">Os dois primeiros exemplos fazem *menos* chamadas de E/S, mas cada um recupera *mais* informações.</span><span class="sxs-lookup"><span data-stu-id="263f2-144">The first two examples make *fewer* I/O calls, but each one retrieves *more* information.</span></span> <span data-ttu-id="263f2-145">Você deve considerar a compensação entre esses dois fatores.</span><span class="sxs-lookup"><span data-stu-id="263f2-145">You must consider the tradeoff between these two factors.</span></span> <span data-ttu-id="263f2-146">A resposta certa dependerá dos padrões de uso reais.</span><span class="sxs-lookup"><span data-stu-id="263f2-146">The right answer will depend on the actual usage patterns.</span></span> <span data-ttu-id="263f2-147">No exemplo de API da Web, pode acontecer que os clientes normalmente precisem apenas do nome de usuário.</span><span class="sxs-lookup"><span data-stu-id="263f2-147">For example, in the web API example, it might turn out that clients often need just the user name.</span></span> <span data-ttu-id="263f2-148">Nesse caso, talvez faça sentido exibi-lo como uma chamada da API separada.</span><span class="sxs-lookup"><span data-stu-id="263f2-148">In that case, it might make sense to expose it as a separate API call.</span></span> <span data-ttu-id="263f2-149">Para obter mais informações, confira o antipadrão de [Busca Incorreta][extraneous-fetching].</span><span class="sxs-lookup"><span data-stu-id="263f2-149">For more information, see the [Extraneous Fetching][extraneous-fetching] antipattern.</span></span>

- <span data-ttu-id="263f2-150">Ao ler os dados, não torne suas solicitações de E/S muito grandes.</span><span class="sxs-lookup"><span data-stu-id="263f2-150">When reading data, do not make your I/O requests too large.</span></span> <span data-ttu-id="263f2-151">Um aplicativo deve recuperar apenas as informações que provavelmente usará.</span><span class="sxs-lookup"><span data-stu-id="263f2-151">An application should only retrieve the information that it is likely to use.</span></span> 

- <span data-ttu-id="263f2-152">Às vezes, é útil dividir as informações de um objeto em duas partes: *os dados acessados com frequência* responsáveis pela maioria das solicitações, e os *dados acessados com menos frequência* e usados raramente.</span><span class="sxs-lookup"><span data-stu-id="263f2-152">Sometimes it helps to partition the information for an object into two chunks, *frequently accessed data* that accounts for most requests, and *less frequently accessed data* that is used rarely.</span></span> <span data-ttu-id="263f2-153">Geralmente, os dados acessados com mais frequência são uma parte relativamente pequena do total de dados para um objeto, portanto, retornar apenas essa parte pode economizar uma sobrecarga de E/S significativa.</span><span class="sxs-lookup"><span data-stu-id="263f2-153">Often the most frequently accessed data is a relatively small portion of the total data for an object, so returning just that portion can save significant I/O overhead.</span></span>

- <span data-ttu-id="263f2-154">Ao gravar os dados, evite bloquear os recursos por mais tempo que o necessário para reduzir as chances de contenção durante uma operação demorada.</span><span class="sxs-lookup"><span data-stu-id="263f2-154">When writing data, avoid locking resources for longer than necessary, to reduce the chances of contention during a lengthy operation.</span></span> <span data-ttu-id="263f2-155">Se uma operação de gravação se estender por vários armazenamentos de dados, arquivos ou serviços, adote uma abordagem consistente.</span><span class="sxs-lookup"><span data-stu-id="263f2-155">If a write operation spans multiple data stores, files, or services, then adopt an eventually consistent approach.</span></span> <span data-ttu-id="263f2-156">Confira [Orientação para a Consistência de Dados][data-consistency-guidance].</span><span class="sxs-lookup"><span data-stu-id="263f2-156">See [Data Consistency guidance][data-consistency-guidance].</span></span>

- <span data-ttu-id="263f2-157">Se você armazenar em buffer os dados na memória antes de gravá-los, eles ficarão vulneráveis se o processo falhar.</span><span class="sxs-lookup"><span data-stu-id="263f2-157">If you buffer data in memory before writing it, the data is vulnerable if the process crashes.</span></span> <span data-ttu-id="263f2-158">Se a taxa de dados geralmente tem picos ou é relativamente esparsa, pode ser mais seguro colocar em buffer os dados em uma fila durável externa, como os [Hubs de Eventos](https://azure.microsoft.com/services/event-hubs/).</span><span class="sxs-lookup"><span data-stu-id="263f2-158">If the data rate typically has bursts or is relatively sparse, it may be safer to buffer the data in an external durable queue such as [Event Hubs](https://azure.microsoft.com/services/event-hubs/).</span></span>

- <span data-ttu-id="263f2-159">Considere armazenar em cache os dados recuperados de um serviço ou banco de dados.</span><span class="sxs-lookup"><span data-stu-id="263f2-159">Consider caching data that you retrieve from a service or a database.</span></span> <span data-ttu-id="263f2-160">Isso pode ajudar a reduzir o volume de E/S, evitando repetidas solicitações dos mesmos dados.</span><span class="sxs-lookup"><span data-stu-id="263f2-160">This can help to reduce the volume of I/O by avoiding repeated requests for the same data.</span></span> <span data-ttu-id="263f2-161">Para obter mais informações, confira [Melhores Práticas do Cache][caching-guidance].</span><span class="sxs-lookup"><span data-stu-id="263f2-161">For more information, see [Caching best practices][caching-guidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="263f2-162">Como detectar o problema</span><span class="sxs-lookup"><span data-stu-id="263f2-162">How to detect the problem</span></span>

<span data-ttu-id="263f2-163">Os sintomas de E/S com ruídos incluem uma alta latência e baixa taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="263f2-163">Symptoms of chatty I/O include high latency and low throughput.</span></span> <span data-ttu-id="263f2-164">Os usuários finais provavelmente irão relatar tempos de resposta maiores ou falhas causadas por serviços atingindo o tempo limite devido à contenção aumentada para os recursos de E/S.</span><span class="sxs-lookup"><span data-stu-id="263f2-164">End users are likely to report extended response times or failures caused by services timing out, due to increased contention for I/O resources.</span></span>

<span data-ttu-id="263f2-165">Você pode executar as etapas a seguir para ajudar a identificar as causas dos problemas:</span><span class="sxs-lookup"><span data-stu-id="263f2-165">You can perform the following steps to help identify the causes of any problems:</span></span>

1. <span data-ttu-id="263f2-166">Execute o monitoramento de processos do sistema de produção para identificar as operações com tempos de resposta ruins.</span><span class="sxs-lookup"><span data-stu-id="263f2-166">Perform process monitoring of the production system to identify operations with poor response times.</span></span>
2. <span data-ttu-id="263f2-167">Realize testes de carga de cada operação identificada na etapa anterior.</span><span class="sxs-lookup"><span data-stu-id="263f2-167">Perform load testing of each operation identified in the previous step.</span></span>
3. <span data-ttu-id="263f2-168">Durante os testes de carga, colete dados de telemetria sobre as solicitações de acesso a dados feitas por cada operação.</span><span class="sxs-lookup"><span data-stu-id="263f2-168">During the load tests, gather telemetry data about the data access requests made by each operation.</span></span>
4. <span data-ttu-id="263f2-169">Colete estatísticas detalhadas para cada solicitação enviada para um armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="263f2-169">Gather detailed statistics for each request sent to a data store.</span></span>
5. <span data-ttu-id="263f2-170">Crie um perfil para o aplicativo no ambiente de teste para estabelecer onde os possíveis gargalos de E/S podem estar ocorrendo.</span><span class="sxs-lookup"><span data-stu-id="263f2-170">Profile the application in the test environment to establish where possible I/O bottlenecks might be occurring.</span></span> 

<span data-ttu-id="263f2-171">Procure esses sintomas:</span><span class="sxs-lookup"><span data-stu-id="263f2-171">Look for any of these symptoms:</span></span>

- <span data-ttu-id="263f2-172">Um grande número de solicitações de E/S pequenas feitas para o mesmo arquivo.</span><span class="sxs-lookup"><span data-stu-id="263f2-172">A large number of small I/O requests made to the same file.</span></span>
- <span data-ttu-id="263f2-173">Um grande número de pequenas solicitações de rede feitas por uma instância do aplicativo para o mesmo serviço.</span><span class="sxs-lookup"><span data-stu-id="263f2-173">A large number of small network requests made by an application instance to the same service.</span></span>
- <span data-ttu-id="263f2-174">Um grande número de pequenas solicitações feitas por uma instância do aplicativo para o mesmo armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="263f2-174">A large number of small requests made by an application instance to the same data store.</span></span>
- <span data-ttu-id="263f2-175">Aplicativos e serviços tendo um limite de E/S.</span><span class="sxs-lookup"><span data-stu-id="263f2-175">Applications and services becoming I/O bound.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="263f2-176">Diagnóstico de exemplo</span><span class="sxs-lookup"><span data-stu-id="263f2-176">Example diagnosis</span></span>

<span data-ttu-id="263f2-177">As seções a seguir aplicam essas etapas no exemplo mostrado anteriormente que consulta um banco de dados.</span><span class="sxs-lookup"><span data-stu-id="263f2-177">The following sections apply these steps to the example shown earlier that queries a database.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="263f2-178">Fazer teste de carga no aplicativo</span><span class="sxs-lookup"><span data-stu-id="263f2-178">Load test the application</span></span>

<span data-ttu-id="263f2-179">Este gráfico mostra os resultados do teste de carga.</span><span class="sxs-lookup"><span data-stu-id="263f2-179">This graph shows the results of load testing.</span></span> <span data-ttu-id="263f2-180">O tempo médio de resposta é medido em 10 s por solicitação.</span><span class="sxs-lookup"><span data-stu-id="263f2-180">Median response time is measured in 10s of seconds per request.</span></span> <span data-ttu-id="263f2-181">O gráfico mostra uma latência muito alta.</span><span class="sxs-lookup"><span data-stu-id="263f2-181">The graph shows very high latency.</span></span> <span data-ttu-id="263f2-182">Com uma carga de 1.000 usuários, um usuário terá que esperar aproximadamente um minuto para ver os resultados de uma consulta.</span><span class="sxs-lookup"><span data-stu-id="263f2-182">With a load of 1000 users, a user might have to wait for nearly a minute to see the results of a query.</span></span> 

![Os resultados do teste de carga dos indicadores-chave para o aplicativo de exemplo de E/S com ruídos][key-indicators-chatty-io]

> [!NOTE]
> <span data-ttu-id="263f2-184">O aplicativo foi implantado como um aplicativo Web do Serviço de Aplicativo do Azure, usando o Banco de Dados SQL do Azure.</span><span class="sxs-lookup"><span data-stu-id="263f2-184">The application was deployed as an Azure App Service web app, using Azure SQL Database.</span></span> <span data-ttu-id="263f2-185">O teste de carga usou uma carga de trabalho da etapa simulada de até 1.000 usuários simultâneos.</span><span class="sxs-lookup"><span data-stu-id="263f2-185">The load test used a simulated step workload of up to 1000 concurrent users.</span></span> <span data-ttu-id="263f2-186">O banco de dados foi configurado com um pool de conexões com suporte de até 1.000 conexões simultâneas para reduzir a chance de que uma contenção por conexões afete os resultados.</span><span class="sxs-lookup"><span data-stu-id="263f2-186">The database was configured with a connection pool supporting up to 1000 concurrent connections, to reduce the chance that contention for connections would affect the results.</span></span> 

### <a name="monitor-the-application"></a><span data-ttu-id="263f2-187">Monitorar o aplicativo</span><span class="sxs-lookup"><span data-stu-id="263f2-187">Monitor the application</span></span>

<span data-ttu-id="263f2-188">Você pode usar um pacote APM (monitoramento de desempenho do aplicativo) para capturar e analisar as principais métricas que podem identificar a E/S com ruídos.</span><span class="sxs-lookup"><span data-stu-id="263f2-188">You can use an application performance monitoring (APM) package to capture and analyze the key metrics that might identify chatty I/O.</span></span> <span data-ttu-id="263f2-189">Quais métricas são importantes dependerá da carga de trabalho de E/S.</span><span class="sxs-lookup"><span data-stu-id="263f2-189">Which metrics are important will depend on the I/O workload.</span></span> <span data-ttu-id="263f2-190">Neste exemplo, as solicitações de E/S interessantes foram as consultas do banco de dados.</span><span class="sxs-lookup"><span data-stu-id="263f2-190">For this example, the interesting I/O requests were the database queries.</span></span> 

<span data-ttu-id="263f2-191">A imagem a seguir mostra os resultados gerados usando o [APM do New Relic][new-relic].</span><span class="sxs-lookup"><span data-stu-id="263f2-191">The following image shows results generated using [New Relic APM][new-relic].</span></span> <span data-ttu-id="263f2-192">O tempo de resposta médio do banco de dados atingiu um pico em aproximadamente 5,6 s por solicitação durante a carga de trabalho máxima.</span><span class="sxs-lookup"><span data-stu-id="263f2-192">The average database response time peaked at approximately 5.6 seconds per request during the maximum workload.</span></span> <span data-ttu-id="263f2-193">O sistema foi capaz de oferecer suporte a uma média de 410 solicitações por minuto durante o teste.</span><span class="sxs-lookup"><span data-stu-id="263f2-193">The system was able to support an average of 410 requests per minute throughout the test.</span></span>

![Visão geral do tráfego acessando o banco de dados AdventureWorks2012][databasetraffic]

### <a name="gather-detailed-data-access-information"></a><span data-ttu-id="263f2-195">Coletar informações detalhadas do acesso a dados</span><span class="sxs-lookup"><span data-stu-id="263f2-195">Gather detailed data access information</span></span>

<span data-ttu-id="263f2-196">Uma análise mais profunda dos dados de monitoramento mostra que o aplicativo executa três instruções SQL SELECT diferentes.</span><span class="sxs-lookup"><span data-stu-id="263f2-196">Digging deeper into the monitoring data shows the application executes three different SQL SELECT statements.</span></span> <span data-ttu-id="263f2-197">Elas correspondem às solicitações geradas pelo Entity Framework para obter dados nas tabelas `ProductListPriceHistory`, `Product` e `ProductSubcategory`.</span><span class="sxs-lookup"><span data-stu-id="263f2-197">These correspond to the requests generated by Entity Framework to fetch data from the `ProductListPriceHistory`, `Product`, and `ProductSubcategory` tables.</span></span>
<span data-ttu-id="263f2-198">Além disso, a consulta que recupera os dados na tabela `ProductListPriceHistory` é de longe a instrução SELECT mais executada, em uma ordem de importância.</span><span class="sxs-lookup"><span data-stu-id="263f2-198">Furthermore, the query that retrieves data from the `ProductListPriceHistory` table is by far the most frequently executed SELECT statement, by an order of magnitude.</span></span>

![Consultas executadas pelo aplicativo de exemplo em teste][queries]

<span data-ttu-id="263f2-200">Acontece que o método `GetProductsInSubCategoryAsync`, mostrado anteriormente, executa 45 consultas SELECT.</span><span class="sxs-lookup"><span data-stu-id="263f2-200">It turns out that the `GetProductsInSubCategoryAsync` method, shown earlier, performs 45 SELECT queries.</span></span> <span data-ttu-id="263f2-201">Cada consulta faz com que o aplicativo abra uma nova conexão de SQL.</span><span class="sxs-lookup"><span data-stu-id="263f2-201">Each query causes the application to open a new SQL connection.</span></span>

![Estatísticas da consulta para o aplicativo de exemplo em teste][queries2]

> [!NOTE]
> <span data-ttu-id="263f2-203">Esta imagem mostra informações de rastreamento para a instância mais lenta da operação `GetProductsInSubCategoryAsync` no teste de carga.</span><span class="sxs-lookup"><span data-stu-id="263f2-203">This image shows trace information for the slowest instance of the `GetProductsInSubCategoryAsync` operation in the load test.</span></span> <span data-ttu-id="263f2-204">Em um ambiente de produção, é útil examinar os rastreamentos das instâncias mais lentas para ver se há um padrão que indica um problema.</span><span class="sxs-lookup"><span data-stu-id="263f2-204">In a production environment, it's useful to examine traces of the slowest instances, to see if there is a pattern that suggests a problem.</span></span> <span data-ttu-id="263f2-205">Se você examinar apenas os valores médios, poderá ignorar os problemas que serão muito piores sob carga.</span><span class="sxs-lookup"><span data-stu-id="263f2-205">If you just look at the average values, you might overlook problems that will get dramatically worse under load.</span></span>

<span data-ttu-id="263f2-206">A imagem seguinte mostra as instruções SQL reais que foram emitidas.</span><span class="sxs-lookup"><span data-stu-id="263f2-206">The next image shows the actual SQL statements that were issued.</span></span> <span data-ttu-id="263f2-207">A consulta de busca das informações de preço é executada para cada produto individual na subcategoria do produto.</span><span class="sxs-lookup"><span data-stu-id="263f2-207">The query that fetches price information is run for each individual product in the product subcategory.</span></span> <span data-ttu-id="263f2-208">Usar uma junção reduziria muito o número de chamadas do banco de dados.</span><span class="sxs-lookup"><span data-stu-id="263f2-208">Using a join would considerably reduce the number of database calls.</span></span>

![Detalhes da consulta para o aplicativo de exemplo em teste][queries3]

<span data-ttu-id="263f2-210">Se você estiver usando um O/RM, como o Entity Framework, rastrear as consultas SQL poderá fornecer informações sobre como o O/RM converte as chamadas da programação em instruções SQL e indicará as áreas onde o acesso aos dados pode ser otimizado.</span><span class="sxs-lookup"><span data-stu-id="263f2-210">If you are using an O/RM, such as Entity Framework, tracing the SQL queries can provide insight into how the O/RM translates programmatic calls into SQL statements, and indicate areas where data access might be optimized.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="263f2-211">Implementar a solução e verificar o resultado</span><span class="sxs-lookup"><span data-stu-id="263f2-211">Implement the solution and verify the result</span></span>

<span data-ttu-id="263f2-212">Reescrever a chamada para o Entity Framework gerou os seguintes resultados.</span><span class="sxs-lookup"><span data-stu-id="263f2-212">Rewriting the call to Entity Framework produced the following results.</span></span>

![Os resultados do teste de carga dos indicadores-chave da API robusta no aplicativo de exemplo de E/S com ruídos][key-indicators-chunky-io]

<span data-ttu-id="263f2-214">Esse teste de carga foi executado na mesma implantação, usando o mesmo perfil de carga.</span><span class="sxs-lookup"><span data-stu-id="263f2-214">This load test was performed on the same deployment, using the same load profile.</span></span> <span data-ttu-id="263f2-215">Desta vez, o gráfico mostra uma latência muito mais baixa.</span><span class="sxs-lookup"><span data-stu-id="263f2-215">This time the graph shows much lower latency.</span></span> <span data-ttu-id="263f2-216">O tempo médio de solicitação com 1.000 usuários é entre 5 e 6 segundos, em quase um minuto.</span><span class="sxs-lookup"><span data-stu-id="263f2-216">The average request time at 1000 users is between 5 and 6 seconds, down from nearly a minute.</span></span>

<span data-ttu-id="263f2-217">Desta vez, o sistema ofereceu suporte a uma média de 3.970 solicitações por minuto, em comparação com 410 para o teste anterior.</span><span class="sxs-lookup"><span data-stu-id="263f2-217">This time the system supported an average of 3,970 requests per minute, compared to 410 for the earlier test.</span></span>

![Visão geral da transação para a API robusta][databasetraffic2]

<span data-ttu-id="263f2-219">Rastrear a instrução SQL mostra que todos os dados são buscados em uma única instrução SELECT.</span><span class="sxs-lookup"><span data-stu-id="263f2-219">Tracing the SQL statement shows that all the data is fetched in a single SELECT statement.</span></span> <span data-ttu-id="263f2-220">Embora essa consulta seja muito mais complexa, ela é executada apenas uma vez por operação.</span><span class="sxs-lookup"><span data-stu-id="263f2-220">Although this query is considerably more complex, it is performed only once per operation.</span></span> <span data-ttu-id="263f2-221">E embora as junções complexas possam ficar caras, os sistemas do banco de dados relacional são otimizados para esse tipo de consulta.</span><span class="sxs-lookup"><span data-stu-id="263f2-221">And while complex joins can become expensive, relational database systems are optimized for this type of query.</span></span>  

![Detalhes da consulta para a API robusta][queries4]

## <a name="related-resources"></a><span data-ttu-id="263f2-223">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="263f2-223">Related resources</span></span>

- <span data-ttu-id="263f2-224">[Práticas recomendadas de Design da API][api-design]</span><span class="sxs-lookup"><span data-stu-id="263f2-224">[API Design best practices][api-design]</span></span>
- <span data-ttu-id="263f2-225">[Práticas recomendadas do cache][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="263f2-225">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="263f2-226">[Manual da Consistência de Dados][data-consistency-guidance]</span><span class="sxs-lookup"><span data-stu-id="263f2-226">[Data Consistency Primer][data-consistency-guidance]</span></span>
- <span data-ttu-id="263f2-227">[Antipadrão de Busca Incorreta][extraneous-fetching]</span><span class="sxs-lookup"><span data-stu-id="263f2-227">[Extraneous Fetching antipattern][extraneous-fetching]</span></span>
- <span data-ttu-id="263f2-228">[Nenhum antipadrão de cache][no-cache]</span><span class="sxs-lookup"><span data-stu-id="263f2-228">[No Caching antipattern][no-cache]</span></span>

[api-design]: ../../best-practices/api-design.md
[caching-guidance]: ../../best-practices/caching.md
[code-sample]:  https://github.com/mspnp/performance-optimization/tree/master/ChattyIO
[data-consistency-guidance]: https://msdn.microsoft.com/library/dn589800.aspx
[ef]: /ef/
[extraneous-fetching]: ../extraneous-fetching/index.md
[new-relic]: https://newrelic.com/application-monitoring
[no-cache]: ../no-caching/index.md

[key-indicators-chatty-io]: _images/ChattyIO.jpg
[key-indicators-chunky-io]: _images/ChunkyIO.jpg
[databasetraffic]: _images/DatabaseTraffic.jpg
[databasetraffic2]: _images/DatabaseTraffic2.jpg
[queries]: _images/DatabaseQueries.jpg
[queries2]: _images/DatabaseQueries2.jpg
[queries3]: _images/DatabaseQueries3.jpg
[queries4]: _images/DatabaseQueries4.jpg

