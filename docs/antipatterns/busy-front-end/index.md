---
title: Antipadrão do front-end ocupado
titleSuffix: Performance antipatterns for cloud apps
description: Um trabalho assíncrono em um grande número de threads em segundo plano pode enfraquecer outras tarefas de primeiro plano dos recursos.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
---

# <a name="busy-front-end-antipattern"></a><span data-ttu-id="fa327-103">Antipadrão do front-end ocupado</span><span class="sxs-lookup"><span data-stu-id="fa327-103">Busy Front End antipattern</span></span>

<span data-ttu-id="fa327-104">Executar um trabalho assíncrono em um grande número de threads em segundo plano pode enfraquecer outras tarefas de primeiro plano simultâneas de recursos, diminuindo os tempos de resposta a níveis inaceitáveis.</span><span class="sxs-lookup"><span data-stu-id="fa327-104">Performing asynchronous work on a large number of background threads can starve other concurrent foreground tasks of resources, decreasing response times to unacceptable levels.</span></span>

## <a name="problem-description"></a><span data-ttu-id="fa327-105">Descrição do problema</span><span class="sxs-lookup"><span data-stu-id="fa327-105">Problem description</span></span>

<span data-ttu-id="fa327-106">Tarefas de uso intensivo de recursos podem aumentar os tempos de resposta para solicitações de usuário e causar alta latência.</span><span class="sxs-lookup"><span data-stu-id="fa327-106">Resource-intensive tasks can increase the response times for user requests and cause high latency.</span></span> <span data-ttu-id="fa327-107">Uma forma de melhorar os tempos de resposta é descarregando uma tarefa de uso intensivo de recursos para um thread separado.</span><span class="sxs-lookup"><span data-stu-id="fa327-107">One way to improve response times is to offload a resource-intensive task to a separate thread.</span></span> <span data-ttu-id="fa327-108">Essa abordagem faz com que o aplicativo permaneça responsivo enquanto o processamento ocorre em segundo plano.</span><span class="sxs-lookup"><span data-stu-id="fa327-108">This approach lets the application stay responsive while processing happens in the background.</span></span> <span data-ttu-id="fa327-109">No entanto, as tarefas que são executadas em um thread em segundo plano ainda consumirão recursos.</span><span class="sxs-lookup"><span data-stu-id="fa327-109">However, tasks that run on a background thread still consume resources.</span></span> <span data-ttu-id="fa327-110">Se houver muitas delas, isso pode enfraquecer os threads que estão tratando de solicitações.</span><span class="sxs-lookup"><span data-stu-id="fa327-110">If there are too many of them, they can starve the threads that are handling requests.</span></span>

> [!NOTE]
> <span data-ttu-id="fa327-111">O termo *recurso* pode abranger muitas coisas, como utilização da CPU, ocupação da memória e E/S de rede ou disco.</span><span class="sxs-lookup"><span data-stu-id="fa327-111">The term *resource* can encompass many things, such as CPU utilization, memory occupancy, and network or disk I/O.</span></span>

<span data-ttu-id="fa327-112">Esse problema normalmente ocorre quando um aplicativo é desenvolvido como trecho de código monolítico, com toda a lógica de negócios combinada em uma única camada compartilhada com a camada de apresentação.</span><span class="sxs-lookup"><span data-stu-id="fa327-112">This problem typically occurs when an application is developed as monolithic piece of code, with all of the business logic combined into a single tier shared with the presentation layer.</span></span>

<span data-ttu-id="fa327-113">Eis aqui um exemplo usando o ASP.NET que demonstra o problema.</span><span class="sxs-lookup"><span data-stu-id="fa327-113">Here’s an example using ASP.NET that demonstrates the problem.</span></span> <span data-ttu-id="fa327-114">Você pode encontrar o exemplo completo [aqui][code-sample].</span><span class="sxs-lookup"><span data-stu-id="fa327-114">You can find the complete sample [here][code-sample].</span></span>

```csharp
public class WorkInFrontEndController : ApiController
{
    [HttpPost]
    [Route("api/workinfrontend")]
    public HttpResponseMessage Post()
    {
        new Thread(() =>
        {
            //Simulate processing
            Thread.SpinWait(Int32.MaxValue / 100);
        }).Start();

        return Request.CreateResponse(HttpStatusCode.Accepted);
    }
}

public class UserProfileController : ApiController
{
    [HttpGet]
    [Route("api/userprofile/{id}")]
    public UserProfile Get(int id)
    {
        //Simulate processing
        return new UserProfile() { FirstName = "Alton", LastName = "Hudgens" };
    }
}
```

- <span data-ttu-id="fa327-115">O método `Post` no controlador `WorkInFrontEnd`implementa uma operação HTTP POST.</span><span class="sxs-lookup"><span data-stu-id="fa327-115">The `Post` method in the `WorkInFrontEnd` controller implements an HTTP POST operation.</span></span> <span data-ttu-id="fa327-116">Essa operação simula uma tarefa de longo prazo com uso intensivo de CPU.</span><span class="sxs-lookup"><span data-stu-id="fa327-116">This operation simulates a long-running, CPU-intensive task.</span></span> <span data-ttu-id="fa327-117">O trabalho é executado em um thread separado, em uma tentativa de permitir que a operação POST seja concluída rapidamente.</span><span class="sxs-lookup"><span data-stu-id="fa327-117">The work is performed on a separate thread, in an attempt to enable the POST operation to complete quickly.</span></span>

- <span data-ttu-id="fa327-118">O método `Get` no controlador `UserProfile`implementa uma operação HTTP GET.</span><span class="sxs-lookup"><span data-stu-id="fa327-118">The `Get` method in the `UserProfile` controller implements an HTTP GET operation.</span></span> <span data-ttu-id="fa327-119">Esse método tem um uso muito menos intensivo da CPU.</span><span class="sxs-lookup"><span data-stu-id="fa327-119">This method is much less CPU intensive.</span></span>

<span data-ttu-id="fa327-120">A principal preocupação é com os requisitos do recurso do método `Post`.</span><span class="sxs-lookup"><span data-stu-id="fa327-120">The primary concern is the resource requirements of the `Post` method.</span></span> <span data-ttu-id="fa327-121">Embora ele coloque o trabalho em um thread em segundo plano, o trabalho ainda pode consumir muitos recursos da CPU.</span><span class="sxs-lookup"><span data-stu-id="fa327-121">Although it puts the work onto a background thread, the work can still consume considerable CPU resources.</span></span> <span data-ttu-id="fa327-122">Esses recursos são compartilhados com outras operações sendo executadas por outros usuários simultâneos.</span><span class="sxs-lookup"><span data-stu-id="fa327-122">These resources are shared with other operations being performed by other concurrent users.</span></span> <span data-ttu-id="fa327-123">Se um número moderado de usuários enviar essa solicitação ao mesmo tempo, o desempenho geral provavelmente será afetado, diminuindo todas as operações.</span><span class="sxs-lookup"><span data-stu-id="fa327-123">If a moderate number of users send this request at the same time, overall performance is likely to suffer, slowing down all operations.</span></span> <span data-ttu-id="fa327-124">Os usuários podem enfrentar latência significativa no método `Get`, por exemplo.</span><span class="sxs-lookup"><span data-stu-id="fa327-124">Users might experience significant latency in the `Get` method, for example.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="fa327-125">Como corrigir o problema</span><span class="sxs-lookup"><span data-stu-id="fa327-125">How to fix the problem</span></span>

<span data-ttu-id="fa327-126">Mova os processos que consomem recursos significativos para um back-end separado.</span><span class="sxs-lookup"><span data-stu-id="fa327-126">Move processes that consume significant resources to a separate back end.</span></span>

<span data-ttu-id="fa327-127">Com essa abordagem, o front-end coloca as tarefas de uso intensivo de recursos em uma fila de mensagens.</span><span class="sxs-lookup"><span data-stu-id="fa327-127">With this approach, the front end puts resource-intensive tasks onto a message queue.</span></span> <span data-ttu-id="fa327-128">O back-end seleciona as tarefas para processamento assíncrono.</span><span class="sxs-lookup"><span data-stu-id="fa327-128">The back end picks up the tasks for asynchronous processing.</span></span> <span data-ttu-id="fa327-129">A fila também atua como um nivelador de carga, armazenando solicitações em buffer para o back-end.</span><span class="sxs-lookup"><span data-stu-id="fa327-129">The queue also acts as a load leveler, buffering requests for the back end.</span></span> <span data-ttu-id="fa327-130">Se o comprimento da fila ficar muito longo, você pode configurar o dimensionamento automático para escalar o back-end horizontalmente.</span><span class="sxs-lookup"><span data-stu-id="fa327-130">If the queue length becomes too long, you can configure autoscaling to scale out the back end.</span></span>

<span data-ttu-id="fa327-131">Eis aqui uma versão revisada do código anterior.</span><span class="sxs-lookup"><span data-stu-id="fa327-131">Here is a revised version of the previous code.</span></span> <span data-ttu-id="fa327-132">Nessa versão, o método `Post` coloca uma mensagem em uma fila do Barramento de Serviço.</span><span class="sxs-lookup"><span data-stu-id="fa327-132">In this version, the `Post` method puts a message on a Service Bus queue.</span></span>

```csharp
public class WorkInBackgroundController : ApiController
{
    private static readonly QueueClient QueueClient;
    private static readonly string QueueName;
    private static readonly ServiceBusQueueHandler ServiceBusQueueHandler;

    public WorkInBackgroundController()
    {
        var serviceBusConnectionString = ...;
        QueueName = ...;
        ServiceBusQueueHandler = new ServiceBusQueueHandler(serviceBusConnectionString);
        QueueClient = ServiceBusQueueHandler.GetQueueClientAsync(QueueName).Result;
    }

    [HttpPost]
    [Route("api/workinbackground")]
    public async Task<long> Post()
    {
        return await ServiceBusQueuehandler.AddWorkLoadToQueueAsync(QueueClient, QueueName, 0);
    }
}
```

<span data-ttu-id="fa327-133">O back-end efetua o pull de mensagens da fila do Barramento de Serviço e executa o processamento.</span><span class="sxs-lookup"><span data-stu-id="fa327-133">The back end pulls messages from the Service Bus queue and does the processing.</span></span>

```csharp
public async Task RunAsync(CancellationToken cancellationToken)
{
    this._queueClient.OnMessageAsync(
        // This lambda is invoked for each message received.
        async (receivedMessage) =>
        {
            try
            {
                // Simulate processing of message
                Thread.SpinWait(Int32.Maxvalue / 1000);

                await receivedMessage.CompleteAsync();
            }
            catch
            {
                receivedMessage.Abandon();
            }
        });
}
```

## <a name="considerations"></a><span data-ttu-id="fa327-134">Considerações</span><span class="sxs-lookup"><span data-stu-id="fa327-134">Considerations</span></span>

- <span data-ttu-id="fa327-135">Essa abordagem adiciona um pouco de complexidade adicional ao aplicativo.</span><span class="sxs-lookup"><span data-stu-id="fa327-135">This approach adds some additional complexity to the application.</span></span> <span data-ttu-id="fa327-136">Você deve tratar a inserção e remoção na fila com segurança para evitar a perda de solicitações em caso de falha.</span><span class="sxs-lookup"><span data-stu-id="fa327-136">You must handle queuing and dequeuing safely to avoid losing requests in the event of a failure.</span></span>
- <span data-ttu-id="fa327-137">O aplicativo usa uma dependência em um serviço adicional para a fila de mensagens.</span><span class="sxs-lookup"><span data-stu-id="fa327-137">The application takes a dependency on an additional service for the message queue.</span></span>
- <span data-ttu-id="fa327-138">O ambiente de processamento deve ser suficientemente escalonável para tratar a carga de trabalho esperada e atender às metas de taxa de transferência necessária.</span><span class="sxs-lookup"><span data-stu-id="fa327-138">The processing environment must be sufficiently scalable to handle the expected workload and meet the required throughput targets.</span></span>
- <span data-ttu-id="fa327-139">Embora essa abordagem deva melhorar a capacidade de resposta geral, as tarefas movidas para o back-end podem levar mais tempo para serem concluídas.</span><span class="sxs-lookup"><span data-stu-id="fa327-139">While this approach should improve overall responsiveness, the tasks that are moved to the back end may take longer to complete.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="fa327-140">Como detectar o problema</span><span class="sxs-lookup"><span data-stu-id="fa327-140">How to detect the problem</span></span>

<span data-ttu-id="fa327-141">Entre os sintomas de um front-end ocupado, temos a alta latência quando tarefas de uso intensivo de recursos estão sendo executadas.</span><span class="sxs-lookup"><span data-stu-id="fa327-141">Symptoms of a busy front end include high latency when resource-intensive tasks are being performed.</span></span> <span data-ttu-id="fa327-142">É provável que os usuários finais relatem tempos de resposta estendidos ou falhas causadas por serviços atingindo o tempo limite. Essas falhas também podem retornar erros HTTP 500 (Servidor Interno) ou erros HTTP 503 (Serviço Indisponível).</span><span class="sxs-lookup"><span data-stu-id="fa327-142">End users are likely to report extended response times or failures caused by services timing out. These failures could also return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="fa327-143">Examine os logs de eventos para o servidor Web, que provavelmente contêm informações mais detalhadas sobre as causas e as circunstâncias dos erros.</span><span class="sxs-lookup"><span data-stu-id="fa327-143">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="fa327-144">Você pode executar as etapas a seguir para ajudar a identificar o problema:</span><span class="sxs-lookup"><span data-stu-id="fa327-144">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="fa327-145">Executar o monitoramento de processos do sistema de produção para identificar pontos quando os tempos de resposta ficam lentos.</span><span class="sxs-lookup"><span data-stu-id="fa327-145">Perform process monitoring of the production system, to identify points when response times slow down.</span></span>
2. <span data-ttu-id="fa327-146">Examinar os dados de telemetria capturados nesses pontos para determinar a combinação de operações sendo executadas e recursos sendo usados.</span><span class="sxs-lookup"><span data-stu-id="fa327-146">Examine the telemetry data captured at these points to determine the mix of operations being performed and the resources being used.</span></span>
3. <span data-ttu-id="fa327-147">Encontrar quaisquer correlações entre os tempos de resposta ruins e os volumes e combinações de operações que estavam acontecendo nesses momentos.</span><span class="sxs-lookup"><span data-stu-id="fa327-147">Find any correlations between poor response times and the volumes and combinations of operations that were happening at those times.</span></span>
4. <span data-ttu-id="fa327-148">Fazer um teste de carga em cada operação suspeita para identificar quais operações estão consumindo recursos e enfraquecendo outras operações.</span><span class="sxs-lookup"><span data-stu-id="fa327-148">Load test each suspected operation to identify which operations are consuming resources and starving other operations.</span></span>
5. <span data-ttu-id="fa327-149">Examinar o código-fonte dessas operações para determinar por que eles podem causar consumo excessivo de recursos.</span><span class="sxs-lookup"><span data-stu-id="fa327-149">Review the source code for those operations to determine why they might cause excessive resource consumption.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="fa327-150">Diagnóstico de exemplo</span><span class="sxs-lookup"><span data-stu-id="fa327-150">Example diagnosis</span></span>

<span data-ttu-id="fa327-151">As seções a seguir aplicam essas etapas ao aplicativo de exemplo descrito anteriormente.</span><span class="sxs-lookup"><span data-stu-id="fa327-151">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slowdown"></a><span data-ttu-id="fa327-152">Identificar pontos de lentidão</span><span class="sxs-lookup"><span data-stu-id="fa327-152">Identify points of slowdown</span></span>

<span data-ttu-id="fa327-153">Instrumente cada método para acompanhar a duração e os recursos consumidos por cada solicitação.</span><span class="sxs-lookup"><span data-stu-id="fa327-153">Instrument each method to track the duration and resources consumed by each request.</span></span> <span data-ttu-id="fa327-154">Depois monitore o aplicativo em produção.</span><span class="sxs-lookup"><span data-stu-id="fa327-154">Then monitor the application in production.</span></span> <span data-ttu-id="fa327-155">Isso pode fornecer uma visão geral de como solicitações concorrem umas com as outras.</span><span class="sxs-lookup"><span data-stu-id="fa327-155">This can provide an overall view of how requests compete with each other.</span></span> <span data-ttu-id="fa327-156">Durante períodos de estresse, solicitações lentas que consomem muitos recursos provavelmente afetarão outras operações. Esse comportamento pode ser observado fazendo o monitoramento do sistema e observando a queda no desempenho.</span><span class="sxs-lookup"><span data-stu-id="fa327-156">During periods of stress, slow-running resource-hungry requests will likely affect other operations, and this behavior can be observed by monitoring the system and noting the drop off in performance.</span></span>

<span data-ttu-id="fa327-157">A imagem a seguir mostra um painel de monitoramento.</span><span class="sxs-lookup"><span data-stu-id="fa327-157">The following image shows a monitoring dashboard.</span></span> <span data-ttu-id="fa327-158">(Usamos [AppDynamics] para nossos testes.) Inicialmente, o sistema tem carga leve.</span><span class="sxs-lookup"><span data-stu-id="fa327-158">(We used [AppDynamics] for our tests.) Initially, the system has light load.</span></span> <span data-ttu-id="fa327-159">Em seguida, os usuários começam a solicitar o método GET `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="fa327-159">Then users start requesting the `UserProfile` GET method.</span></span> <span data-ttu-id="fa327-160">O desempenho é razoavelmente bom até que outros usuários comecem a emitir solicitações para o método POST `WorkInFrontEnd`.</span><span class="sxs-lookup"><span data-stu-id="fa327-160">The performance is reasonably good until other users start issuing requests to the `WorkInFrontEnd` POST method.</span></span> <span data-ttu-id="fa327-161">Nesse ponto, os tempos de resposta aumentam consideravelmente (primeira seta).</span><span class="sxs-lookup"><span data-stu-id="fa327-161">At that point, response times increase dramatically (first arrow).</span></span> <span data-ttu-id="fa327-162">Os tempos de resposta só melhoram após a diminuição do volume de solicitações para o controlador `WorkInFrontEnd` (segunda seta).</span><span class="sxs-lookup"><span data-stu-id="fa327-162">Response times only improve after the volume of requests to the `WorkInFrontEnd` controller diminishes (second arrow).</span></span>

![Painel de transações comerciais do AppDynamics mostrando os efeitos dos tempos de resposta de todas as solicitações quando o controlador WorkInFrontEnd é usado][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="fa327-164">Examinar os dados de telemetria e localizar correlações</span><span class="sxs-lookup"><span data-stu-id="fa327-164">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="fa327-165">A imagem a seguir mostra algumas das métricas coletadas para monitorar a utilização de recursos durante o mesmo intervalo.</span><span class="sxs-lookup"><span data-stu-id="fa327-165">The next image shows some of the metrics gathered to monitor resource utilization during the same interval.</span></span> <span data-ttu-id="fa327-166">No primeiro momento, poucos usuários estão acessando o sistema.</span><span class="sxs-lookup"><span data-stu-id="fa327-166">At first, few users are accessing the system.</span></span> <span data-ttu-id="fa327-167">À medida que mais usuários se conectam, a utilização da CPU fica muito alta (100%).</span><span class="sxs-lookup"><span data-stu-id="fa327-167">As more users connect, CPU utilization becomes very high (100%).</span></span> <span data-ttu-id="fa327-168">Também observe que a taxa de E/S de rede inicialmente sobe à medida que o uso da CPU aumenta.</span><span class="sxs-lookup"><span data-stu-id="fa327-168">Also notice that the network I/O rate initially goes up as CPU usage rises.</span></span> <span data-ttu-id="fa327-169">Mas uma vez que o uso da CPU fica em pico, a E/S de rede diminui.</span><span class="sxs-lookup"><span data-stu-id="fa327-169">But once CPU usage peaks, network I/O actually goes down.</span></span> <span data-ttu-id="fa327-170">Isso se deve ao fato de o sistema só conseguir lidar com um número relativamente pequeno de solicitações depois que a CPU está na capacidade máxima.</span><span class="sxs-lookup"><span data-stu-id="fa327-170">That's because the system can only handle a relatively small number of requests once the CPU is at capacity.</span></span> <span data-ttu-id="fa327-171">À medida que os usuários se desconectam, a carga da CPU diminui.</span><span class="sxs-lookup"><span data-stu-id="fa327-171">As users disconnect, the CPU load tails off.</span></span>

![Métricas do AppDynamics mostrando a utilização de CPU e de rede][AppDynamics-Metrics-Front-End-Requests]

<span data-ttu-id="fa327-173">Nesse ponto, parece que o método `Post` no controlador `WorkInFrontEnd` é um candidato perfeito para uma análise mais detalhada.</span><span class="sxs-lookup"><span data-stu-id="fa327-173">At this point, it appears the `Post` method in the `WorkInFrontEnd` controller is a prime candidate for closer examination.</span></span> <span data-ttu-id="fa327-174">É preciso um trabalho mais profundo em um ambiente controlado para confirmar a hipótese.</span><span class="sxs-lookup"><span data-stu-id="fa327-174">Further work in a controlled environment is needed to confirm the hypothesis.</span></span>

### <a name="perform-load-testing"></a><span data-ttu-id="fa327-175">Realizar testes de carga</span><span class="sxs-lookup"><span data-stu-id="fa327-175">Perform load testing</span></span>

<span data-ttu-id="fa327-176">A próxima etapa é executar testes em um ambiente controlado.</span><span class="sxs-lookup"><span data-stu-id="fa327-176">The next step is to perform tests in a controlled environment.</span></span> <span data-ttu-id="fa327-177">Por exemplo, executar uma série de testes de carga que incluam e depois omitam cada solicitação para ver os efeitos.</span><span class="sxs-lookup"><span data-stu-id="fa327-177">For example, run a series of load tests that include and then omit each request in turn to see the effects.</span></span>

<span data-ttu-id="fa327-178">O gráfico abaixo mostra os resultados de um teste de carga executado em uma implantação idêntica do serviço em nuvem usado nos testes anteriores.</span><span class="sxs-lookup"><span data-stu-id="fa327-178">The graph below shows the results of a load test performed against an identical deployment of the cloud service used in the previous tests.</span></span> <span data-ttu-id="fa327-179">O teste usou uma carga constante de 500 usuários executando a operação `Get` no controlador `UserProfile`, junto com uma carga por etapa de usuários executando a operação `Post` no controlador `WorkInFrontEnd`.</span><span class="sxs-lookup"><span data-stu-id="fa327-179">The test used a constant load of 500 users performing the `Get` operation in the `UserProfile` controller, along with a step load of users performing the `Post` operation in the `WorkInFrontEnd` controller.</span></span>

![Resultados de teste de carga inicial do controlador WorkInFrontEnd][Initial-Load-Test-Results-Front-End]

<span data-ttu-id="fa327-181">Inicialmente, a carga por etapa é 0, de modo que somente os usuários ativos estão executando as solicitações `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="fa327-181">Initially, the step load is 0, so the only active users are performing the `UserProfile` requests.</span></span> <span data-ttu-id="fa327-182">O sistema é capaz de responder a aproximadamente 500 solicitações por segundo.</span><span class="sxs-lookup"><span data-stu-id="fa327-182">The system is able to respond to approximately 500 requests per second.</span></span> <span data-ttu-id="fa327-183">Depois de 60 segundos, uma carga de 100 usuários adicionais começa a enviar solicitações POST para o controlador `WorkInFrontEnd`.</span><span class="sxs-lookup"><span data-stu-id="fa327-183">After 60 seconds, a load of 100 additional users starts sending POST requests to the `WorkInFrontEnd` controller.</span></span> <span data-ttu-id="fa327-184">Quase imediatamente, a carga de trabalho enviada para o controlador `UserProfile` cai para cerca de 150 solicitações por segundo.</span><span class="sxs-lookup"><span data-stu-id="fa327-184">Almost immediately, the workload sent to the `UserProfile` controller drops to about 150 requests per second.</span></span> <span data-ttu-id="fa327-185">Isso se deve à maneira como o executor de teste de carga funciona.</span><span class="sxs-lookup"><span data-stu-id="fa327-185">This is due to the way the load-test runner functions.</span></span> <span data-ttu-id="fa327-186">Ele espera por uma resposta antes de enviar a próxima solicitação, portanto, quanto mais tempo ele leva para receber uma resposta, menor a taxa de solicitação.</span><span class="sxs-lookup"><span data-stu-id="fa327-186">It waits for a response before sending the next request, so the longer it takes to receive a response, the lower the request rate.</span></span>

<span data-ttu-id="fa327-187">À medida que mais usuários enviam solicitações POST para o controlador `WorkInFrontEnd`, a taxa de resposta do controlador `UserProfile` continua a cair.</span><span class="sxs-lookup"><span data-stu-id="fa327-187">As more users send POST requests to the `WorkInFrontEnd` controller, the response rate of the `UserProfile` controller continues to drop.</span></span> <span data-ttu-id="fa327-188">Mas observe que o volume de solicitações tratadas pelo controlador `WorkInFrontEnd` permanece relativamente constante.</span><span class="sxs-lookup"><span data-stu-id="fa327-188">But note that the volume of requests handled by the `WorkInFrontEnd`controller remains relatively constant.</span></span> <span data-ttu-id="fa327-189">A saturação do sistema torna-se aparente à medida que a taxa geral de ambas as solicitações tende a um limite estável, mas baixo.</span><span class="sxs-lookup"><span data-stu-id="fa327-189">The saturation of the system becomes apparent as the overall rate of both requests tends towards a steady but low limit.</span></span>

### <a name="review-the-source-code"></a><span data-ttu-id="fa327-190">Examinar o código-fonte</span><span class="sxs-lookup"><span data-stu-id="fa327-190">Review the source code</span></span>

<span data-ttu-id="fa327-191">A etapa final é examinar o código-fonte.</span><span class="sxs-lookup"><span data-stu-id="fa327-191">The final step is to look at the source code.</span></span> <span data-ttu-id="fa327-192">A equipe de desenvolvimento estava ciente de que o método `Post` poderia demorar um tempo considerável, por isso que a implementação original usou um thread separado.</span><span class="sxs-lookup"><span data-stu-id="fa327-192">The development team was aware that the `Post` method could take a considerable amount of time, which is why the original implementation used a separate thread.</span></span> <span data-ttu-id="fa327-193">Isso solucionou o problema imediato porque o método `Post` não foi bloqueado ao aguardar uma tarefa de execução longa ser concluída.</span><span class="sxs-lookup"><span data-stu-id="fa327-193">That solved the immediate problem, because the `Post` method did not block waiting for a long-running task to complete.</span></span>

<span data-ttu-id="fa327-194">No entanto, o trabalho executado por esse método ainda consome CPU, memória e outros recursos.</span><span class="sxs-lookup"><span data-stu-id="fa327-194">However, the work performed by this method still consumes CPU, memory, and other resources.</span></span> <span data-ttu-id="fa327-195">Habilitar esse processo para ser executado de forma assíncrona pode até afetar o desempenho, uma vez que os usuários podem acionar um grande número dessas operações simultaneamente e de maneira descontrolada.</span><span class="sxs-lookup"><span data-stu-id="fa327-195">Enabling this process to run asynchronously might actually damage performance, as users can trigger a large number of these operations simultaneously, in an uncontrolled manner.</span></span> <span data-ttu-id="fa327-196">Há um limite para o número de threads que podem ser executados por um servidor.</span><span class="sxs-lookup"><span data-stu-id="fa327-196">There is a limit to the number of threads that a server can run.</span></span> <span data-ttu-id="fa327-197">Ao passar desse limite, é provável que o aplicativo receba uma exceção ao tentar iniciar um novo thread.</span><span class="sxs-lookup"><span data-stu-id="fa327-197">Past this limit, the application is likely to get an exception when it tries to start a new thread.</span></span>

> [!NOTE]
> <span data-ttu-id="fa327-198">Isso não significa que você deve evitar operações assíncronas.</span><span class="sxs-lookup"><span data-stu-id="fa327-198">This doesn't mean you should avoid asynchronous operations.</span></span> <span data-ttu-id="fa327-199">Executar uma espera assíncrona em uma chamada de rede é uma prática recomendada.</span><span class="sxs-lookup"><span data-stu-id="fa327-199">Performing an asynchronous await on a network call is a recommended practice.</span></span> <span data-ttu-id="fa327-200">(Consulte o antipadrão [E/S síncronas][sync-io].) O problema aqui é que o trabalho com uso intensivo de CPU foi gerado em outro thread.</span><span class="sxs-lookup"><span data-stu-id="fa327-200">(See the [Synchronous I/O][sync-io] antipattern.) The problem here is that CPU-intensive work was spawned on another thread.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="fa327-201">Implementar a solução e verificar o resultado</span><span class="sxs-lookup"><span data-stu-id="fa327-201">Implement the solution and verify the result</span></span>

<span data-ttu-id="fa327-202">A imagem a seguir mostra o monitoramento de desempenho depois de a solução ter sido implementada.</span><span class="sxs-lookup"><span data-stu-id="fa327-202">The following image shows performance monitoring after the solution was implemented.</span></span> <span data-ttu-id="fa327-203">A carga foi semelhante à mostrada anteriormente, mas os tempos de resposta para o controlador `UserProfile` agora estão muito mais rápidos.</span><span class="sxs-lookup"><span data-stu-id="fa327-203">The load was similar to that shown earlier, but the response times for the `UserProfile` controller are now much faster.</span></span> <span data-ttu-id="fa327-204">O volume de solicitações aumentou com a mesma duração: de 2.759 para 23.565.</span><span class="sxs-lookup"><span data-stu-id="fa327-204">The volume of requests increased over the same duration, from 2,759 to 23,565.</span></span>

![Painel de transações comerciais do AppDynamics mostrando os efeitos dos tempos de resposta de todas as solicitações quando o controlador WorkInBackground é usado][AppDynamics-Transactions-Background-Requests]

<span data-ttu-id="fa327-206">Observe que o controlador `WorkInBackground` também tratou de um volume muito maior de solicitações.</span><span class="sxs-lookup"><span data-stu-id="fa327-206">Note that the `WorkInBackground` controller also handled a much larger volume of requests.</span></span> <span data-ttu-id="fa327-207">No entanto, você não pode fazer uma comparação direta nesse caso porque o trabalho sendo executado nesse controlador é muito diferente do código original.</span><span class="sxs-lookup"><span data-stu-id="fa327-207">However, you can't make a direct comparison in this case, because the work being performed in this controller is very different from the original code.</span></span> <span data-ttu-id="fa327-208">A nova versão simplesmente coloca uma solicitação na fila, em vez de executar um cálculo demorado.</span><span class="sxs-lookup"><span data-stu-id="fa327-208">The new version simply queues a request, rather than performing a time consuming calculation.</span></span> <span data-ttu-id="fa327-209">O ponto principal é que esse método não mais arrasta para baixo todo o sistema sob carga.</span><span class="sxs-lookup"><span data-stu-id="fa327-209">The main point is that this method no longer drags down the entire system under load.</span></span>

<span data-ttu-id="fa327-210">O uso de CPU e de rede também mostra o desempenho aprimorado.</span><span class="sxs-lookup"><span data-stu-id="fa327-210">CPU and network utilization also show the improved performance.</span></span> <span data-ttu-id="fa327-211">O uso da CPU nunca atingiu 100%, e o volume de solicitações de rede tratadas era muito maior do que o anterior e não diminuiu até a carga de trabalho diminuir.</span><span class="sxs-lookup"><span data-stu-id="fa327-211">The CPU utilization never reached 100%, and the volume of handled network requests was far greater than earlier, and did not tail off until the workload dropped.</span></span>

![Métricas do AppDynamics mostrando o uso de CPU e de rede para o controlador WorkInBackground][AppDynamics-Metrics-Background-Requests]

<span data-ttu-id="fa327-213">O gráfico a seguir mostra os resultados de um teste de carga.</span><span class="sxs-lookup"><span data-stu-id="fa327-213">The following graph shows the results of a load test.</span></span> <span data-ttu-id="fa327-214">O volume total de solicitações atendidas é significativamente maior em comparação com os testes anteriores.</span><span class="sxs-lookup"><span data-stu-id="fa327-214">The overall volume of requests serviced is greatly improved compared to the the earlier tests.</span></span>

![Resultados de teste de carga para o controlador BackgroundImageProcessing][Load-Test-Results-Background]

## <a name="related-guidance"></a><span data-ttu-id="fa327-216">Diretrizes relacionadas</span><span class="sxs-lookup"><span data-stu-id="fa327-216">Related guidance</span></span>

- <span data-ttu-id="fa327-217">[Práticas recomendadas de dimensionamento automático][autoscaling]</span><span class="sxs-lookup"><span data-stu-id="fa327-217">[Autoscaling best practices][autoscaling]</span></span>
- <span data-ttu-id="fa327-218">[Práticas recomendadas de trabalhos em segundo plano][background-jobs]</span><span class="sxs-lookup"><span data-stu-id="fa327-218">[Background jobs best practices][background-jobs]</span></span>
- <span data-ttu-id="fa327-219">[Padrão de nivelamento de carga baseado em fila][load-leveling]</span><span class="sxs-lookup"><span data-stu-id="fa327-219">[Queue-Based Load Leveling pattern][load-leveling]</span></span>
- <span data-ttu-id="fa327-220">[Estilo de arquitetura de trabalho em fila da Web][web-queue-worker]</span><span class="sxs-lookup"><span data-stu-id="fa327-220">[Web Queue Worker architecture style][web-queue-worker]</span></span>

[AppDyanamics]: https://www.appdynamics.com/
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[background-jobs]: /azure/architecture/best-practices/background-jobs
[code-sample]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[fullDemonstrationOfSolution]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[load-leveling]: /azure/architecture/patterns/queue-based-load-leveling
[sync-io]: ../synchronous-io/index.md
[web-queue-worker]: /azure/architecture/guide/architecture-styles/web-queue-worker

[WebJobs]: https://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx
[ComputePartitioning]: https://msdn.microsoft.com/library/dn589773.aspx
[ServiceBusQueues]: https://msdn.microsoft.com/library/azure/hh367516.aspx
[AppDynamics-Transactions-Front-End-Requests]: ./_images/AppDynamicsPerformanceStats.jpg
[AppDynamics-Metrics-Front-End-Requests]: ./_images/AppDynamicsFrontEndMetrics.jpg
[Initial-Load-Test-Results-Front-End]: ./_images/InitialLoadTestResultsFrontEnd.jpg
[AppDynamics-Transactions-Background-Requests]: ./_images/AppDynamicsBackgroundPerformanceStats.jpg
[AppDynamics-Metrics-Background-Requests]: ./_images/AppDynamicsBackgroundMetrics.jpg
[Load-Test-Results-Background]: ./_images/LoadTestResultsBackground.jpg
