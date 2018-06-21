---
title: Antipadrão de instanciação inadequada
description: Evite criar continuamente novas instâncias de um objeto que deve ser criado somente uma vez e depois compartilhado.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 4d5ef9ad9e675b46df94b51e81d7a4bd4c1b25e9
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/23/2018
ms.locfileid: "29477573"
---
# <a name="improper-instantiation-antipattern"></a><span data-ttu-id="ebdc1-103">Antipadrão de instanciação inadequada</span><span class="sxs-lookup"><span data-stu-id="ebdc1-103">Improper Instantiation antipattern</span></span>

<span data-ttu-id="ebdc1-104">Criar continuamente novas instâncias de um objeto que deve ser criado somente uma vez e depois compartilhado pode prejudicar o desempenho.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-104">It can hurt performance to continually create new instances of an object that is meant to be created once and then shared.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="ebdc1-105">Descrição do problema</span><span class="sxs-lookup"><span data-stu-id="ebdc1-105">Problem description</span></span>

<span data-ttu-id="ebdc1-106">Muitas bibliotecas fornecem abstrações de recursos externos.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-106">Many libraries provide abstractions of external resources.</span></span> <span data-ttu-id="ebdc1-107">Internamente, essas classes normalmente gerenciam suas próprias conexões ao recurso, atuando como agentes que os clientes podem usar para acessar o recurso.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-107">Internally, these classes typically manage their own connections to the resource, acting as brokers that clients can use to access the resource.</span></span> <span data-ttu-id="ebdc1-108">Aqui estão alguns exemplos de classes de agente que são relevantes para aplicativos do Azure:</span><span class="sxs-lookup"><span data-stu-id="ebdc1-108">Here are some examples of broker classes that are relevant to Azure applications:</span></span>

- <span data-ttu-id="ebdc1-109">`System.Net.Http.HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-109">`System.Net.Http.HttpClient`.</span></span> <span data-ttu-id="ebdc1-110">Se comunica com um serviço Web usando HTTP.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-110">Communicates with a web service using HTTP.</span></span>
- <span data-ttu-id="ebdc1-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span></span> <span data-ttu-id="ebdc1-112">Envia e recebe mensagens para uma fila do Barramento de Serviço.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-112">Posts and receives messages to a Service Bus queue.</span></span> 
- <span data-ttu-id="ebdc1-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span></span> <span data-ttu-id="ebdc1-114">Conecta-se a uma instância do Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="ebdc1-114">Connects to a Cosmos DB instance</span></span>
- <span data-ttu-id="ebdc1-115">`StackExchange.Redis.ConnectionMultiplexer`.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-115">`StackExchange.Redis.ConnectionMultiplexer`.</span></span> <span data-ttu-id="ebdc1-116">Conecta-se ao Redis, incluindo o Cache Redis do Azure.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-116">Connects to Redis, including Azure Redis Cache.</span></span>

<span data-ttu-id="ebdc1-117">Essas classes são destinadas a serem instanciados uma vez e reutilizadas em todo o tempo de vida de um aplicativo.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-117">These classes are intended to be instantiated once and reused throughout the lifetime of an application.</span></span> <span data-ttu-id="ebdc1-118">No entanto, é comum cometer o equívoco de achar que essas classes devem ser adquiridas somente quando necessário e lançadas rapidamente.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-118">However, it's a common misunderstanding that these classes should be acquired only as necessary and released quickly.</span></span> <span data-ttu-id="ebdc1-119">(As classes listadas são de bibliotecas .NET, mas o padrão não é exclusivo para .NET.) O ASP.NET de exemplo a seguir cria uma instância de `HttpClient` para se comunicar com um serviço remoto.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-119">(The ones listed here happen to be .NET libraries, but the pattern is not unique to .NET.) The following ASP.NET example creates an instance of `HttpClient` to communicate with a remote service.</span></span> <span data-ttu-id="ebdc1-120">Você pode encontrar o exemplo completo [aqui][sample-app].</span><span class="sxs-lookup"><span data-stu-id="ebdc1-120">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

<span data-ttu-id="ebdc1-121">Em um aplicativo Web, essa técnica não será escalonável.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-121">In a web application, this technique is not scalable.</span></span> <span data-ttu-id="ebdc1-122">Um novo objeto `HttpClient` é criado para cada solicitação de usuário.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-122">A new `HttpClient` object is created for each user request.</span></span> <span data-ttu-id="ebdc1-123">Sob carga pesada, o servidor Web pode esgotar o número de soquetes disponíveis, resultando em erros `SocketException`.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-123">Under heavy load, the web server may exhaust the number of available sockets, resulting in `SocketException` errors.</span></span>

<span data-ttu-id="ebdc1-124">Esse problema não está restrito à classe `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-124">This problem is not restricted to the `HttpClient` class.</span></span> <span data-ttu-id="ebdc1-125">Outras classes que encapsulam recursos ou que são caros de criar podem causar problemas semelhantes.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-125">Other classes that wrap resources or are expensive to create might cause similar issues.</span></span> <span data-ttu-id="ebdc1-126">O exemplo a seguir cria uma instância da classe `ExpensiveToCreateService`.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-126">The following example creates an instances of the `ExpensiveToCreateService` class.</span></span> <span data-ttu-id="ebdc1-127">Aqui o problema não é necessariamente o esgotamento de soquete, mas simplesmente quanto tempo leva para criar cada instância.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-127">Here the issue is not necessarily socket exhaustion, but simply how long it takes to create each instance.</span></span> <span data-ttu-id="ebdc1-128">Criar e destruir continuamente instâncias dessa classe pode prejudicar a escalabilidade do sistema.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-128">Continually creating and destroying instances of this class might adversely affect the scalability of the system.</span></span>

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="ebdc1-129">Como corrigir o problema</span><span class="sxs-lookup"><span data-stu-id="ebdc1-129">How to fix the problem</span></span>

<span data-ttu-id="ebdc1-130">Se a classe que encapsula o recurso externo for compartilhável e thread-safe, crie uma instância singleton compartilhada ou um pool de instâncias reutilizáveis da classe.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-130">If the class that wraps the external resource is shareable and thread-safe, create a shared singleton instance or a pool of reusable instances of the class.</span></span>

<span data-ttu-id="ebdc1-131">O seguinte exemplo usa uma instância `HttpClient` estática, compartilhamento, portanto, a conexão entre todas as solicitações.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-131">The following example uses a static `HttpClient` instance, thus sharing the connection across all requests.</span></span>

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient httpClient;

    static SingleHttpClientInstanceController()
    {
        httpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a><span data-ttu-id="ebdc1-132">Considerações</span><span class="sxs-lookup"><span data-stu-id="ebdc1-132">Considerations</span></span>

- <span data-ttu-id="ebdc1-133">O elemento chave desse antipadrão é criar e destruir repetidamente as instâncias de um objeto *compartilhável*.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-133">The key element of this antipattern is repeatedly creating and destroying instances of a *shareable* object.</span></span> <span data-ttu-id="ebdc1-134">Se a classe não for compartilhável (não é thread-safe), então esse antipadrão não se aplica.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-134">If a class is not shareable (not thread-safe), then this antipattern does not apply.</span></span>

- <span data-ttu-id="ebdc1-135">O tipo de recurso compartilhado pode determinam se você deve usar um singleton ou criar um pool.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-135">The type of shared resource might dictate whether you should use a singleton or create a pool.</span></span> <span data-ttu-id="ebdc1-136">A classe `HttpClient` foi projetada para ser compartilhada em vez de ser colocada em pool.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-136">The `HttpClient` class is designed to be shared rather than pooled.</span></span> <span data-ttu-id="ebdc1-137">Outros objetos podem oferecer suporte a pool, permitindo que o sistema distribua a carga de trabalho entre várias instâncias.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-137">Other objects might support pooling, enabling the system to spread the workload across multiple instances.</span></span>

- <span data-ttu-id="ebdc1-138">Os objetos que você compartilha entre várias solicitações *devem* ser thread-safe.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-138">Objects that you share across multiple requests *must* be thread-safe.</span></span> <span data-ttu-id="ebdc1-139">A classe `HttpClient` foi projetada para ser usada dessa maneira, mas outras classes podem não oferecer suporte a solicitações simultâneas, portanto, verifique a documentação disponível.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-139">The `HttpClient` class is designed to be used in this manner, but other classes might not support concurrent requests, so check the available documentation.</span></span>

- <span data-ttu-id="ebdc1-140">Alguns tipos de recurso são escassos e não devem ser mantidos.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-140">Some resource types are scarce and should not be held onto.</span></span> <span data-ttu-id="ebdc1-141">Conexões de banco de dados são um exemplo.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-141">Database connections are an example.</span></span> <span data-ttu-id="ebdc1-142">Manter uma conexão de banco de dados aberto que não seja necessária pode impedir que outros usuários simultâneos tenham acesso ao banco de dados.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-142">Holding an open database connection that is not required may prevent other concurrent users from gaining access to the database.</span></span>

- <span data-ttu-id="ebdc1-143">No .NET Framework, muitos objetos estabelecem conexões com recursos externos são criados usando os métodos de fábrica estáticos de outras classes que gerenciam essas conexões.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-143">In the .NET Framework, many objects that establish connections to external resources are created by using static factory methods of other classes that manage these connections.</span></span> <span data-ttu-id="ebdc1-144">Esses objetos de fábricas destinam-se a ser salvos e reutilizados, em vez de descartados e recriados.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-144">These factories  objects are intended to be saved and reused, rather than disposed and recreated.</span></span> <span data-ttu-id="ebdc1-145">Por exemplo, no Barramento de Serviço do Azure, o objeto `QueueClient` é criado por meio de um objeto `MessagingFactory`.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-145">For example, in Azure Service Bus, the `QueueClient` object is created through a `MessagingFactory` object.</span></span> <span data-ttu-id="ebdc1-146">Internamente, o `MessagingFactory` gerencia as conexões.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-146">Internally, the `MessagingFactory` manages connections.</span></span> <span data-ttu-id="ebdc1-147">Para saber mais informações, consulte [Práticas recomendadas para melhorias de desempenho usando o Sistema de Mensagens do Barramento de Serviço][service-bus-messaging].</span><span class="sxs-lookup"><span data-stu-id="ebdc1-147">For more information, see [Best Practices for performance improvements using Service Bus Messaging][service-bus-messaging].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="ebdc1-148">Como detectar o problema</span><span class="sxs-lookup"><span data-stu-id="ebdc1-148">How to detect the problem</span></span>

<span data-ttu-id="ebdc1-149">Sintomas desse problema incluem uma queda na taxa de transferência ou um aumento na taxa de erro, junto com um ou mais dos sintomas a seguir:</span><span class="sxs-lookup"><span data-stu-id="ebdc1-149">Symptoms of this problem include a drop in throughput or an increased error rate, along with one or more of the following:</span></span> 

- <span data-ttu-id="ebdc1-150">Um aumento nas exceções que indica o esgotamento de recursos, como soquetes, conexões de banco de dados, identificadores de arquivos e assim por diante.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-150">An increase in exceptions that indicate exhaustion of resources such as sockets, database connections, file handles, and so on.</span></span> 
- <span data-ttu-id="ebdc1-151">Um aumento no uso de memória e na coleta de lixo.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-151">Increased memory use and garbage collection.</span></span>
- <span data-ttu-id="ebdc1-152">Um aumento na atividade de banco de dados, disco ou rede.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-152">An increase in network, disk, or database activity.</span></span>

<span data-ttu-id="ebdc1-153">Você pode executar as etapas a seguir para ajudar a identificar o problema:</span><span class="sxs-lookup"><span data-stu-id="ebdc1-153">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="ebdc1-154">Executar o monitoramento de processos do sistema de produção, para identificar os pontos onde os tempos de resposta ficam mais lentos ou onde o sistema falha devido à falta de recursos.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-154">Performing process monitoring of the production system, to identify points when response times slow down or the system fails due to lack of resources.</span></span>
2. <span data-ttu-id="ebdc1-155">Examine os dados de telemetria capturados nesses pontos para determinar quais operações podem estar criando e destruindo os objetos que consomem recursos.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-155">Examine the telemetry data captured at these points to determine which operations might be creating and destroying resource-consuming objects.</span></span>
3. <span data-ttu-id="ebdc1-156">Faça o teste de carga de cada operação suspeita em um ambiente de teste controlado em vez de em um sistema de produção.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-156">Load test each suspected operation, in a controlled test environment rather than the production system.</span></span>
4. <span data-ttu-id="ebdc1-157">Examine o código-fonte e examine como os objetos do agente são gerenciados.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-157">Review the source code and examine the how broker objects are managed.</span></span>

<span data-ttu-id="ebdc1-158">Observe os rastreamentos de pilha para operações que são lentas ou que geram exceções quando o sistema está sob carga.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-158">Look at stack traces for operations that are slow-running or that generate exceptions when the system is under load.</span></span> <span data-ttu-id="ebdc1-159">Essas informações podem ajudar a identificar como essas operações estão utilizando os recursos.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-159">This information can help to identify how these operations are utilizing resources.</span></span> <span data-ttu-id="ebdc1-160">Exceções podem ajudar a determinar se os erros são causados por recursos compartilhados sendo esgotados.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-160">Exceptions can help to determine whether errors are caused by shared resources being exhausted.</span></span> 

## <a name="example-diagnosis"></a><span data-ttu-id="ebdc1-161">Diagnóstico de exemplo</span><span class="sxs-lookup"><span data-stu-id="ebdc1-161">Example diagnosis</span></span>

<span data-ttu-id="ebdc1-162">As seções a seguir aplicam essas etapas ao aplicativo de exemplo descrito anteriormente.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-162">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slow-down-or-failure"></a><span data-ttu-id="ebdc1-163">Identificar pontos de lentidão ou falha</span><span class="sxs-lookup"><span data-stu-id="ebdc1-163">Identify points of slow down or failure</span></span>

<span data-ttu-id="ebdc1-164">A imagem a seguir mostra os resultados gerados usando o [Gerenciamento de Desempenho de Aplicativos (APM) da New Relic][new-relic], mostrando também as operações que têm um tempo de resposta ruim.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-164">The following image shows results generated using [New Relic APM][new-relic], showing operations that have a poor response time.</span></span> <span data-ttu-id="ebdc1-165">Nesse caso, vale a pena fazer uma investigação mais detalhada ao método `GetProductAsync` no controlador `NewHttpClientInstancePerRequest`.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-165">In this case, the `GetProductAsync` method in the `NewHttpClientInstancePerRequest` controller is worth investigating further.</span></span> <span data-ttu-id="ebdc1-166">Observe que a taxa de erros também aumenta quando essas operações estão em execução.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-166">Notice that the error rate also increases when these operations are running.</span></span> 

![O painel do monitor da New Relic mostra o aplicativo de exemplo criando uma nova instância de um objeto HttpClient para cada solicitação][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="ebdc1-168">Examinar os dados de telemetria e localizar correlações</span><span class="sxs-lookup"><span data-stu-id="ebdc1-168">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="ebdc1-169">A imagem seguinte mostra os dados capturados usando a criação de perfil de thread, no mesmo período correspondente à imagem anterior.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-169">The next image shows data captured using thread profiling, over the same period corresponding as the previous image.</span></span> <span data-ttu-id="ebdc1-170">O sistema gasta um tempo significativo abrindo as conexões de soquete e ainda mais tempo fechando-as e tratando as exceções de soquete.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-170">The system spends a significant time opening socket connections, and even more time closing them and handling socket exceptions.</span></span>

![O criador de perfil de thread da New Relic mostra o aplicativo de exemplo criando uma nova instância de um objeto HttpClient para cada solicitação][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a><span data-ttu-id="ebdc1-172">Executar o teste de carga</span><span class="sxs-lookup"><span data-stu-id="ebdc1-172">Performing load testing</span></span>

<span data-ttu-id="ebdc1-173">Use o teste de carga para simular as operações comuns que podem ser executadas pelos usuários.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-173">Use load testing to simulate the typical operations that users might perform.</span></span> <span data-ttu-id="ebdc1-174">Isso pode ajudar a identificar quais partes de um sistema são afetadas por um esgotamento de recursos sob cargas diferentes.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-174">This can help to identify which parts of a system suffer from resource exhaustion under varying loads.</span></span> <span data-ttu-id="ebdc1-175">Execute esses testes em um ambiente controlado em vez de um sistema de produção.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-175">Perform these tests in a controlled environment rather than the production system.</span></span> <span data-ttu-id="ebdc1-176">O gráfico a seguir mostra a taxa de transferência de solicitações manipulada pelo controlador `NewHttpClientInstancePerRequest`, já que a carga de usuários aumenta para 100 usuários simultâneos.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-176">The following graph shows the throughput of requests handled by the `NewHttpClientInstancePerRequest` controller as the user load increases to 100 concurrent users.</span></span>

![Taxa de transferência do mesmo aplicativo criando uma nova instância de um objeto HttpClient para cada solicitação][throughput-new-HTTPClient-instance]

<span data-ttu-id="ebdc1-178">Primeiro, o volume de solicitações tratadas por segundo aumenta conforme aumenta a carga de trabalho.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-178">At first, the volume of requests handled per second increases as the workload increases.</span></span> <span data-ttu-id="ebdc1-179">Depois de aproximadamente 30 usuários, no entanto, o volume de solicitações bem sucedidas atinge um limite e o sistema começa a gerar exceções.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-179">At about 30 users, however, the volume of successful requests reaches a limit, and the system starts to generate exceptions.</span></span> <span data-ttu-id="ebdc1-180">Daí em diante, o volume de exceções aumenta gradualmente junto com a carga de usuário.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-180">From then on, the volume of exceptions gradually increases with the user load.</span></span> 

<span data-ttu-id="ebdc1-181">O teste de carga relatou essas falhas como erros HTTP 500 (servidor interno).</span><span class="sxs-lookup"><span data-stu-id="ebdc1-181">The load test reported these failures as HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="ebdc1-182">Uma revisão da telemetria mostrou que esses erros foram causados pela execução do sistema sem recursos de soquete, já que cada vez mais objetos `HttpClient` foram criados.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-182">Reviewing the telemetry showed that these errors were caused by the system running out of socket resources, as more and more `HttpClient` objects were created.</span></span>

<span data-ttu-id="ebdc1-183">O gráfico seguinte mostra um teste semelhante para um controlador que cria o objeto `ExpensiveToCreateService` personalizado.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-183">The next graph shows a similar test for a controller that creates the custom `ExpensiveToCreateService` object.</span></span>

![Taxa de transferência do mesmo aplicativo criando uma nova instância do ExpensiveToCreateService para cada solicitação][throughput-new-ExpensiveToCreateService-instance]

<span data-ttu-id="ebdc1-185">Neste momento, o controlador não gera exceções, mas a taxa de transferência ainda atinge um limite, enquanto o tempo médio de resposta aumenta em um fator de 20.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-185">This time, the controller does not generate any exceptions, but throughput still reaches a plateau, while the average response time increases by a factor of 20.</span></span> <span data-ttu-id="ebdc1-186">(O gráfico usa uma escala logarítmica para o tempo de resposta e a taxa de transferência). A telemetria mostrou que criar novas instâncias de `ExpensiveToCreateService` foi a causa principal do problema.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-186">(The graph uses a logarithmic scale for response time and throughput.) Telemetry showed that creating new instances of the `ExpensiveToCreateService` was the main cause of the problem.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="ebdc1-187">Implementar a solução e verificar o resultado</span><span class="sxs-lookup"><span data-stu-id="ebdc1-187">Implement the solution and verify the result</span></span>

<span data-ttu-id="ebdc1-188">Depois de alternar o método `GetProductAsync` para compartilhar uma única instância `HttpClient`, um segundo teste de carga mostrou um melhor desempenho.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-188">After switching the `GetProductAsync` method to share a single `HttpClient` instance, a second load test showed improved performance.</span></span> <span data-ttu-id="ebdc1-189">Nenhum erro foi relatado, e o sistema foi capaz de lidar com um aumento da carga de até 500 solicitações por segundo.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-189">No errors were reported, and the system was able to handle an increasing load of up to 500 requests per second.</span></span> <span data-ttu-id="ebdc1-190">O tempo médio de resposta diminuiu pela metade, em comparação com o teste anterior.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-190">The average response time was cut in half, compared with the previous test.</span></span>

![Taxa de transferência do mesmo aplicativo reutilizando a mesma instância de um objeto HttpClient para cada solicitação][throughput-single-HTTPClient-instance]

<span data-ttu-id="ebdc1-192">Para comparação, a imagem a seguir mostra a telemetria de rastreamento de pilha.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-192">For comparison, the following image shows the stack trace telemetry.</span></span> <span data-ttu-id="ebdc1-193">Dessa vez o sistema gasta a maior parte do tempo executando o trabalho real, em vez de abrindo e fechando soquetes.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-193">This time, the system spends most of its time performing real work, rather than opening and closing sockets.</span></span>

![O criador de perfil de thread da New Relic mostra o aplicativo de exemplo criando uma única instância de um objeto HttpClient para todas as solicitações][thread-profiler-single-HTTPClient-instance]

<span data-ttu-id="ebdc1-195">O gráfico seguinte mostra um teste de carga semelhante usando uma instância compartilhada do objeto `ExpensiveToCreateService`.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-195">The next graph shows a similar load test using a shared instance of the `ExpensiveToCreateService` object.</span></span> <span data-ttu-id="ebdc1-196">Novamente, o volume de solicitações manipuladas aumenta junto com a carga de usuário enquanto o tempo médio de resposta permanece baixo.</span><span class="sxs-lookup"><span data-stu-id="ebdc1-196">Again, the volume of handled requests increases in line with the user load, while the average response time remains low.</span></span> 

![Taxa de transferência do mesmo aplicativo reutilizando a mesma instância de um objeto HttpClient para cada solicitação][throughput-single-ExpensiveToCreateService-instance]



[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
