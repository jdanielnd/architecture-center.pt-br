---
title: Antipadrão de E/S síncrona
description: O bloqueio do thread de chamada durante a conclusão da E/S pode reduzir o desempenho e afetar a escalabilidade vertical.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: d5b3635565c6b71ef7716f54ee8cccc76093c3a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="synchronous-io-antipattern"></a><span data-ttu-id="cfa6e-103">Antipadrão de E/S síncrona</span><span class="sxs-lookup"><span data-stu-id="cfa6e-103">Synchronous I/O antipattern</span></span>

<span data-ttu-id="cfa6e-104">O bloqueio do thread de chamada durante a conclusão da E/S pode reduzir o desempenho e afetar a escalabilidade vertical.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-104">Blocking the calling thread while I/O completes can reduce performance and affect vertical scalability.</span></span>

## <a name="problem-description"></a><span data-ttu-id="cfa6e-105">Descrição do problema</span><span class="sxs-lookup"><span data-stu-id="cfa6e-105">Problem description</span></span>

<span data-ttu-id="cfa6e-106">Uma operação de E/S síncrona bloqueia o thread de chamada, enquanto a E/S é concluída.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-106">A synchronous I/O operation blocks the calling thread while the I/O completes.</span></span> <span data-ttu-id="cfa6e-107">O thread de chamada entra em um estado de espera e não é possível executar o trabalho útil durante esse intervalo, desperdiçando recursos de processamento.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-107">The calling thread enters a wait state and is unable to perform useful work during this interval, wasting processing resources.</span></span>

<span data-ttu-id="cfa6e-108">Exemplos comuns de E/S incluem:</span><span class="sxs-lookup"><span data-stu-id="cfa6e-108">Common examples of I/O include:</span></span>

- <span data-ttu-id="cfa6e-109">Recuperar dados persistentes para um banco de dados ou qualquer tipo de armazenamento persistente.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-109">Retrieving or persisting data to a database or any type of persistent storage.</span></span>
- <span data-ttu-id="cfa6e-110">Enviando uma solicitação para um serviço Web.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-110">Sending a request to a web service.</span></span>
- <span data-ttu-id="cfa6e-111">Envio de uma mensagem ou recuperação de mensagens de uma fila.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-111">Posting a message or retrieving a message from a queue.</span></span>
- <span data-ttu-id="cfa6e-112">Gravar ou ler de um arquivo local.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-112">Writing to or reading from a local file.</span></span>

<span data-ttu-id="cfa6e-113">Esse antipadrão geralmente ocorre porque:</span><span class="sxs-lookup"><span data-stu-id="cfa6e-113">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="cfa6e-114">Ele parece ser a maneira mais fácil para executar uma operação.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-114">It appears to be the most intuitive way to perform an operation.</span></span> 
- <span data-ttu-id="cfa6e-115">O aplicativo requer uma resposta de uma solicitação.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-115">The application requires a response from a request.</span></span>
- <span data-ttu-id="cfa6e-116">O aplicativo usa uma biblioteca que só fornece métodos síncronos de E/S.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-116">The application uses a library that only provides synchronous methods for I/O.</span></span> 
- <span data-ttu-id="cfa6e-117">Uma biblioteca externa executa operações de E/S síncrona internamente.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-117">An external library performs synchronous I/O operations internally.</span></span> <span data-ttu-id="cfa6e-118">Uma única chamada de E/S síncrona pode bloquear uma cadeia de chamada inteira.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-118">A single synchronous I/O call can block an entire call chain.</span></span>

<span data-ttu-id="cfa6e-119">O código a seguir carrega um arquivo para o armazenamento de Blobs do Azure.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-119">The following code uploads a file to Azure blob storage.</span></span> <span data-ttu-id="cfa6e-120">Há dois locais em que os blocos de código aguardam a E/S síncrona, o método `CreateIfNotExists` e o método `UploadFromStream`.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-120">There are two places where the code blocks waiting for synchronous I/O, the `CreateIfNotExists` method and the `UploadFromStream` method.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

<span data-ttu-id="cfa6e-121">Aqui está um exemplo de uma resposta de um serviço externo.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-121">Here's an example of waiting for a response from an external service.</span></span> <span data-ttu-id="cfa6e-122">O `GetUserProfile` método chama um serviço remoto retorna um `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-122">The `GetUserProfile` method calls a remote service that returns a `UserProfile`.</span></span>

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

<span data-ttu-id="cfa6e-123">Você pode encontrar o código completo para ambos os exemplos [aqui][sample-app].</span><span class="sxs-lookup"><span data-stu-id="cfa6e-123">You can find the complete code for both of these examples [here][sample-app].</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="cfa6e-124">Como corrigir o problema</span><span class="sxs-lookup"><span data-stu-id="cfa6e-124">How to fix the problem</span></span>

<span data-ttu-id="cfa6e-125">Substitua as operações de E/S síncrona com operações assíncronas.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-125">Replace synchronous I/O operations with asynchronous operations.</span></span> <span data-ttu-id="cfa6e-126">Isso libera o thread atual para continuar executando trabalho significativo em vez de bloquear e ajuda a melhorar a utilização dos recursos de computação.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-126">This frees the current thread to continue performing meaningful work rather than blocking, and helps improve the utilization of compute resources.</span></span> <span data-ttu-id="cfa6e-127">Executar E/S assíncrona é especialmente eficiente para lidar com um surto inesperado em solicitações de aplicativos cliente.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-127">Performing I/O asynchronously is particularly efficient for handling an unexpected surge in requests from client applications.</span></span> 

<span data-ttu-id="cfa6e-128">Muitas bibliotecas fornecem versões síncronas e assíncronas dos métodos.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-128">Many libraries provide both synchronous and asynchronous versions of methods.</span></span> <span data-ttu-id="cfa6e-129">Sempre que possível, use as versões assíncronas.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-129">Whenever possible, use the asynchronous versions.</span></span> <span data-ttu-id="cfa6e-130">Aqui está a versão assíncrona do exemplo anterior que carrega um arquivo para o armazenamento de Blobs do Azure.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-130">Here is the asynchronous version of the previous example that uploads a file to Azure blob storage.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

<span data-ttu-id="cfa6e-131">O operador `await` retorna o controle para o ambiente de chamada enquanto a operação assíncrona é executada.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-131">The `await` operator returns control to the calling environment while the asynchronous operation is performed.</span></span> <span data-ttu-id="cfa6e-132">O código depois dessa instrução atua como uma continuação que é executada depois que a operação assíncrona é concluída.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-132">The code after this statement acts as a continuation that runs when the asynchronous operation has completed.</span></span>

<span data-ttu-id="cfa6e-133">Um serviço bem projetado também deve fornecer a operações assíncronas.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-133">A well designed service should also provide asynchronous operations.</span></span> <span data-ttu-id="cfa6e-134">Aqui está uma versão assíncrona do serviço Web que retorna os perfis de usuário.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-134">Here is an asynchronous version of the web service that returns user profiles.</span></span> <span data-ttu-id="cfa6e-135">O `GetUserProfileAsync` método depende de ter uma versão assíncrona do serviço de perfil de usuário.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-135">The `GetUserProfileAsync` method depends on having an asynchronous version of the User Profile service.</span></span>

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

<span data-ttu-id="cfa6e-136">Para bibliotecas que não fornecem versões assíncronas de operações, pode ser possível criar wrappers assíncronos em torno de métodos síncronos selecionados.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-136">For libraries that don't provide asynchronous versions of operations, it may be possible to create asynchronous wrappers around selected synchronous methods.</span></span> <span data-ttu-id="cfa6e-137">Siga essa abordagem com cuidado.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-137">Follow this approach with caution.</span></span> <span data-ttu-id="cfa6e-138">Embora possa melhorar a capacidade de resposta no thread que invoca o wrapper assíncrono, ele realmente consome mais recursos.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-138">While it may improve responsiveness on the thread that invokes the asynchronous wrapper, it actually consumes more resources.</span></span> <span data-ttu-id="cfa6e-139">Um thread adicional pode ser criado e não há sobrecarga associada à sincronização do trabalho realizado por esse thread.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-139">An extra thread may be created, and there is overhead associated with synchronizing the work done by this thread.</span></span> <span data-ttu-id="cfa6e-140">Alguns desafios são discutidos nesta postagem de blog: [Devo divulgar wrappers assíncronos para métodos síncronos?][async-wrappers]</span><span class="sxs-lookup"><span data-stu-id="cfa6e-140">Some tradeoffs are discussed in this blog post: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]</span></span>

<span data-ttu-id="cfa6e-141">Aqui está um exemplo de um wrapper em torno de um método síncrono de assíncrono.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-141">Here is an example of an asynchronous wrapper around a synchronous method.</span></span>

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

<span data-ttu-id="cfa6e-142">Agora, o código de chamada pode aguardar em um wrapper:</span><span class="sxs-lookup"><span data-stu-id="cfa6e-142">Now the calling code can await on the wrapper:</span></span>

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a><span data-ttu-id="cfa6e-143">Considerações</span><span class="sxs-lookup"><span data-stu-id="cfa6e-143">Considerations</span></span>

- <span data-ttu-id="cfa6e-144">As operações de E/S que devem ser muito curta duração e provavelmente não causar contenção podem ser mais funcional como operações síncronas.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-144">I/O operations that are expected to be very short lived and are unlikely to cause contention might be more performant as synchronous operations.</span></span> <span data-ttu-id="cfa6e-145">Um exemplo pode ser a leitura de pequenos arquivos em uma unidade SSD.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-145">An example might be reading small files on an SSD drive.</span></span> <span data-ttu-id="cfa6e-146">A sobrecarga de despachar uma tarefa para outro thread e sincronizar com esse thread quando a tarefa for concluída, pode superar os benefícios de E/S assíncrona.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-146">The overhead of dispatching a task to another thread, and synchronizing with that thread when the task completes, might outweigh the benefits of asynchronous I/O.</span></span> <span data-ttu-id="cfa6e-147">No entanto, esses casos são relativamente raros, e a maioria das operações de E/S deve ser feita de forma assíncrona.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-147">However, these cases are relatively rare, and most I/O operations should be done asynchronously.</span></span>

- <span data-ttu-id="cfa6e-148">Melhorando o desempenho de E/S pode causar outras partes do sistema para se tornar afunilamentos.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-148">Improving I/O performance may cause other parts of the system to become bottlenecks.</span></span> <span data-ttu-id="cfa6e-149">Por exemplo, segmentos de desbloqueio podem resultar em um maior volume de solicitações simultâneas para recursos compartilhados, em vez de limitação ou privação de recursos.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-149">For example, unblocking threads might result in a higher volume of concurrent requests to shared resources, leading in turn to resource starvation or throttling.</span></span> <span data-ttu-id="cfa6e-150">Se isso se tornar um problema, você precisará expandir o número de servidores Web ou dados de partição de lojas para reduzir a contenção.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-150">If that becomes a problem, you might need to scale out the number of web servers or partition data stores to reduce contention.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="cfa6e-151">Como detectar o problema</span><span class="sxs-lookup"><span data-stu-id="cfa6e-151">How to detect the problem</span></span>

<span data-ttu-id="cfa6e-152">Para usuários, o aplicativo pode parecer não responder ou parar periodicamente.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-152">For users, the application may seem unresponsive or appear to hang periodically.</span></span> <span data-ttu-id="cfa6e-153">O aplicativo pode falhar com exceções de tempo limite.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-153">The application might fail with timeout exceptions.</span></span> <span data-ttu-id="cfa6e-154">Essas falhas também poderá retornar erros HTTP 500 (servidor interno).</span><span class="sxs-lookup"><span data-stu-id="cfa6e-154">These failures could also return HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="cfa6e-155">No servidor, as solicitações de cliente podem estar bloqueadas até que um thread fique disponível, resultando em comprimentos de fila de solicitações excessivas, manifestados como erros HTTP 503 (Serviço Indisponível).</span><span class="sxs-lookup"><span data-stu-id="cfa6e-155">On the server, incoming client requests might be blocked until a thread becomes available, resulting in excessive request queue lengths, manifested as HTTP 503 (Service Unavailable) errors.</span></span>

<span data-ttu-id="cfa6e-156">Você pode executar as etapas a seguir para ajudar a identificar o problema:</span><span class="sxs-lookup"><span data-stu-id="cfa6e-156">You can perform the following steps to help identify the problem:</span></span>

1. <span data-ttu-id="cfa6e-157">Monitorar o sistema de produção e determinar se bloqueado threads de trabalho são restrições de taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-157">Monitor the production system and determine whether blocked worker threads are constraining throughput.</span></span>

2. <span data-ttu-id="cfa6e-158">Se solicitações estão sendo bloqueadas devido à falta de threads, examine o aplicativo para determinar quais operações podem ser executar E/S síncrona.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-158">If requests are being blocked due to lack of threads, review the application to determine which operations may be performing I/O synchronously.</span></span>

3. <span data-ttu-id="cfa6e-159">Execute testes de carga controlada de cada operação que está executando a E/S síncrona, para descobrir se essas operações estão afetando o desempenho do sistema.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-159">Perform controlled load testing of each operation that is performing synchronous I/O, to find out whether those operations are affecting system performance.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="cfa6e-160">Diagnóstico de exemplo</span><span class="sxs-lookup"><span data-stu-id="cfa6e-160">Example diagnosis</span></span>

<span data-ttu-id="cfa6e-161">As seções a seguir aplicam essas etapas ao aplicativo de exemplo descrito anteriormente.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-161">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-web-server-performance"></a><span data-ttu-id="cfa6e-162">Monitorar desempenho de servidor Web</span><span class="sxs-lookup"><span data-stu-id="cfa6e-162">Monitor web server performance</span></span>

<span data-ttu-id="cfa6e-163">Para funções Web e aplicativos Web do Azure, é importante monitorar o desempenho do servidor Web do IIS.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-163">For Azure web applications and web roles, it's worth monitoring the performance of the IIS web server.</span></span> <span data-ttu-id="cfa6e-164">Em particular, preste atenção para o comprimento da fila de solicitação para estabelecer solicitações estão sendo bloqueadas aguardando threads disponíveis durante períodos de alta atividade.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-164">In particular, pay attention to the request queue length to establish whether requests are being blocked waiting for available threads during periods of high activity.</span></span> <span data-ttu-id="cfa6e-165">Você pode coletar essas informações por habilitar o diagnóstico do Azure.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-165">You can gather this information by enabling Azure diagnostics.</span></span> <span data-ttu-id="cfa6e-166">Para obter mais informações, consulte:</span><span class="sxs-lookup"><span data-stu-id="cfa6e-166">For more information, see:</span></span>

- <span data-ttu-id="cfa6e-167">[Monitorar aplicativos no Serviço de Aplicativo do Azure][web-sites-monitor]</span><span class="sxs-lookup"><span data-stu-id="cfa6e-167">[Monitor Apps in Azure App Service][web-sites-monitor]</span></span>
- <span data-ttu-id="cfa6e-168">[Criar e usar contadores de desempenho em um aplicativo do Azure][performance-counters]</span><span class="sxs-lookup"><span data-stu-id="cfa6e-168">[Create and use performance counters in an Azure application][performance-counters]</span></span>

<span data-ttu-id="cfa6e-169">Instrumentar o aplicativo para ver como as solicitações são tratadas quando eles tiverem aceitado.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-169">Instrument the application to see how requests are handled once they have been accepted.</span></span> <span data-ttu-id="cfa6e-170">O rastreamento do fluxo de uma solicitação pode ajudar a identificar se ele está executando chamadas lentas e bloquear o thread atual.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-170">Tracing the flow of a request can help to identify whether it is performing slow-running calls and blocking the current thread.</span></span> <span data-ttu-id="cfa6e-171">Thread de criação de perfil também pode realçar a solicitações que estão sendo bloqueadas.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-171">Thread profiling can also highlight requests that are being blocked.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="cfa6e-172">Fazer teste de carga no aplicativo</span><span class="sxs-lookup"><span data-stu-id="cfa6e-172">Load test the application</span></span>

<span data-ttu-id="cfa6e-173">O gráfico a seguir mostra o desempenho do método `GetUserProfile` assíncrono mostrado anteriormente, sob cargas diferentes de até 4.000 usuários simultâneos.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-173">The following graph shows the performance of the synchronous `GetUserProfile` method shown earlier, under varying loads of up to 4000 concurrent users.</span></span> <span data-ttu-id="cfa6e-174">O aplicativo é um aplicativo ASP.NET em execução em uma função Web do serviço de nuvem do Azure.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-174">The application is an ASP.NET application running in an Azure Cloud Service web role.</span></span>

![Gráfico de desempenho para o aplicativo de exemplo executar operações de E/S síncrona][sync-performance]

<span data-ttu-id="cfa6e-176">A operação síncrona é codificada no modo de suspensão por dois segundos, para simular a E/S síncrona, portanto, o tempo de resposta mínimo é um pouco mais de dois segundos.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-176">The synchronous operation is hard-coded to sleep for 2 seconds, to simulate synchronous I/O, so the minimum response time is slightly over 2 seconds.</span></span> <span data-ttu-id="cfa6e-177">Quando a carga atingir aproximadamente 2500 usuários simultâneos, o tempo médio de resposta atinge um limite, embora o volume de solicitações por segundo continua a aumentar.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-177">When the load reaches approximately 2500 concurrent users, the average response time reaches a plateau, although the volume of requests per second continues to increase.</span></span> <span data-ttu-id="cfa6e-178">Observe que a escala para essas duas medidas é logarítmica.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-178">Note that the scale for these two measures is logarithmic.</span></span> <span data-ttu-id="cfa6e-179">O número de solicitações por segundo duplicatas entre esse ponto e o fim do teste.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-179">The number of requests per second doubles between this point and the end of the test.</span></span>

<span data-ttu-id="cfa6e-180">Isoladamente, não é necessariamente claro neste teste se a E/S síncrona é um problema.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-180">In isolation, it's not necessarily clear from this test whether the synchronous I/O is a problem.</span></span> <span data-ttu-id="cfa6e-181">Sob carga mais pesada, o aplicativo pode alcançar o ápice onde o servidor Web não pode processar solicitações de maneira oportuna, fazendo com que os aplicativos cliente recebam exceções de tempo limite.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-181">Under heavier load, the application may reach a tipping point where the web server can no longer process requests in a timely manner, causing client applications to receive time-out exceptions.</span></span>

<span data-ttu-id="cfa6e-182">As solicitações de entrada são colocadas em fila pelo servidor Web do IIS e passadas para um thread em execução no pool de threads do ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-182">Incoming requests are queued by the IIS web server and handed to a thread running in the ASP.NET thread pool.</span></span> <span data-ttu-id="cfa6e-183">Como cada operação executa a E/S de forma síncrona, o thread é bloqueado até que a operação seja concluída.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-183">Because each operation performs I/O synchronously, the thread is blocked until the operation completes.</span></span> <span data-ttu-id="cfa6e-184">À medida que aumenta a carga de trabalho, eventualmente todos os threads ASP.NET no pool de threads são alocados e bloqueados.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-184">As the workload increases, eventually all of the ASP.NET threads in the thread pool are allocated and blocked.</span></span> <span data-ttu-id="cfa6e-185">Nesse ponto, quaisquer solicitações de entrada adicional devem esperar na fila por um segmento disponível.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-185">At that point, any further incoming requests must wait in the queue for an available thread.</span></span> <span data-ttu-id="cfa6e-186">À medida que aumenta o tamanho da fila, as solicitações começam a atingir o tempo limite.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-186">As the queue length grows, requests start to time out.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="cfa6e-187">Implementar a solução e verificar o resultado</span><span class="sxs-lookup"><span data-stu-id="cfa6e-187">Implement the solution and verify the result</span></span>

<span data-ttu-id="cfa6e-188">O seguinte gráfico mostra os resultados da versão assíncrona do código de teste de carga.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-188">The next graph shows the results from load testing the asynchronous version of the code.</span></span>

![Gráfico de desempenho para o aplicativo de exemplo executar operações de E/S assíncronas][async-performance]

<span data-ttu-id="cfa6e-190">A taxa de transferência é muito maior.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-190">Throughput is far higher.</span></span> <span data-ttu-id="cfa6e-191">Sobre a mesma duração como o teste anterior, o sistema manipula com êxito um aumento de dez vezes quase na taxa de transferência, medida em solicitações por segundo.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-191">Over the same duration as the previous test, the system successfully handles a nearly tenfold increase in throughput, as measured in requests per second.</span></span> <span data-ttu-id="cfa6e-192">Além disso, o tempo médio de resposta é relativamente constante e permanece aproximadamente 25 vezes menor do que o teste anterior.</span><span class="sxs-lookup"><span data-stu-id="cfa6e-192">Moreover, the average response time is relatively constant and remains approximately 25 times smaller than the previous test.</span></span>


[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO


[async-wrappers]: http://blogs.msdn.com/b/pfxteam/archive/2012/03/24/10287244.aspx
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg



