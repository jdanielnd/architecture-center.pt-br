---
title: Processamento de evento sem servidor usando o Azure Functions
description: Arquitetura de referência que mostra o processamento e ingestão de eventos sem servidor
author: MikeWasson
ms.date: 10/16/2018
ms.openlocfilehash: 2bb7600fbed95e4b9368cf342c0bc6a75c5f8755
ms.sourcegitcommit: 113a7248b9793c670b0f2d4278d30ad8616abe6c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/16/2018
ms.locfileid: "49349931"
---
# <a name="serverless-event-processing-using-azure-functions"></a><span data-ttu-id="b096c-103">Processamento de evento sem servidor usando o Azure Functions</span><span class="sxs-lookup"><span data-stu-id="b096c-103">Serverless event processing using Azure Functions</span></span>

<span data-ttu-id="b096c-104">Essa arquitetura de referência mostra uma arquitetura sem servidor controlada por evento que ingere um fluxo de dados, processa os dados e grava os resultados em um banco de dados back-end.</span><span class="sxs-lookup"><span data-stu-id="b096c-104">This reference architecture shows a serverless, event-driven architecture that ingests a stream of data, processes the data, and writes the results to a back-end database.</span></span> <span data-ttu-id="b096c-105">Há uma implantação de referência para essa arquitetura de referência disponível no [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="b096c-105">A reference implementation for this architecture is available on [GitHub][github].</span></span>

![](./_images/serverless-event-processing.png)

## <a name="architecture"></a><span data-ttu-id="b096c-106">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="b096c-106">Architecture</span></span>

<span data-ttu-id="b096c-107">Os **Hubs de Eventos** ingerem o fluxo de dados.</span><span class="sxs-lookup"><span data-stu-id="b096c-107">**Event Hubs** ingests the data stream.</span></span> <span data-ttu-id="b096c-108">Os [Hubs de Eventos][eh] foram projetados para cenários de streaming de dados com alta taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="b096c-108">[Event Hubs][eh] is designed for high-throughput data streaming scenarios.</span></span>

> [!NOTE]
> <span data-ttu-id="b096c-109">Para cenários de IoT, recomendamos o Hub IoT.</span><span class="sxs-lookup"><span data-stu-id="b096c-109">For IoT scenarios, we recommend IoT Hub.</span></span> <span data-ttu-id="b096c-110">O Hub IoT tem um ponto de extremidade interno compatível com a API de Hubs de Eventos do Azure, para que você possa usar um dos serviços nessa arquitetura sem grandes mudanças no processamento de back-end.</span><span class="sxs-lookup"><span data-stu-id="b096c-110">IoT Hub has a built-in endpoint that’s compatible with the Azure Event Hubs API, so you can use either service in this architecture with no major changes in the backend processing.</span></span> <span data-ttu-id="b096c-111">Para saber mais, confira [Conectar dispositivos IoT ao Azure: Hub IoT e Hubs de Eventos][iot].</span><span class="sxs-lookup"><span data-stu-id="b096c-111">For more information, see [Connecting IoT Devices to Azure: IoT Hub and Event Hubs][iot].</span></span>

<span data-ttu-id="b096c-112">**Aplicativo de funções**.</span><span class="sxs-lookup"><span data-stu-id="b096c-112">**Function App**.</span></span> <span data-ttu-id="b096c-113">O [Azure Functions][functions] é uma opção de computação sem servidor.</span><span class="sxs-lookup"><span data-stu-id="b096c-113">[Azure Functions][functions] is a serverless compute option.</span></span> <span data-ttu-id="b096c-114">Ele usa um modelo controlado por evento, em que um trecho de código (uma "função") é invocado por um gatilho.</span><span class="sxs-lookup"><span data-stu-id="b096c-114">It uses an event-driven model, where a piece of code (a “function”) is invoked by a trigger.</span></span> <span data-ttu-id="b096c-115">Nessa arquitetura, quando os eventos chegam aos Hubs de Eventos, eles disparam uma função que processa os eventos e grava os resultados no armazenamento.</span><span class="sxs-lookup"><span data-stu-id="b096c-115">In this architecture, when events arrive at Event Hubs, they trigger a function that processes the events and writes the results to storage.</span></span>

<span data-ttu-id="b096c-116">Os Aplicativos de Funções são adequados para processar registros individuais dos Hubs de Eventos.</span><span class="sxs-lookup"><span data-stu-id="b096c-116">Function Apps are suitable for processing individual records from Event Hubs.</span></span> <span data-ttu-id="b096c-117">Para cenários de processamento de fluxo mais complexos, considere o Apache Spark com o Azure Databricks, ou o Azure Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="b096c-117">For more complex stream processing scenarios, consider Apache Spark using Azure Databricks, or Azure Stream Analytics.</span></span>

<span data-ttu-id="b096c-118">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="b096c-118">**Cosmos DB**.</span></span> <span data-ttu-id="b096c-119">O [Cosmos DB][cosmosdb] é um serviço de banco de dados multimodelo.</span><span class="sxs-lookup"><span data-stu-id="b096c-119">[Cosmos DB][cosmosdb] is a multi-model database service.</span></span> <span data-ttu-id="b096c-120">Para este cenário, a função de processamento de eventos armazena registros JSON, usando a [API do SQL][cosmosdb-sql] do Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="b096c-120">For this scenario, the event-processing function stores JSON records, using the Cosmos DB [SQL API][cosmosdb-sql].</span></span>

<span data-ttu-id="b096c-121">**Armazenamento de filas**.</span><span class="sxs-lookup"><span data-stu-id="b096c-121">**Queue storage**.</span></span> <span data-ttu-id="b096c-122">O [Armazenamento de filas][queue] é usado para mensagens mortas.</span><span class="sxs-lookup"><span data-stu-id="b096c-122">[Queue storage][queue] is used for dead letter messages.</span></span> <span data-ttu-id="b096c-123">Se ocorrer um erro ao processar um evento, a função armazenará os dados do evento em uma fila de mensagens mortas para processamento posterior.</span><span class="sxs-lookup"><span data-stu-id="b096c-123">If an error occurs while processing an event, the function stores the event data in a dead letter queue for later processing.</span></span> <span data-ttu-id="b096c-124">Para saber mais, confira [Considerações de resiliência](#resiliency-considerations).</span><span class="sxs-lookup"><span data-stu-id="b096c-124">For more information, see [Resiliency Considerations](#resiliency-considerations).</span></span>

<span data-ttu-id="b096c-125">**Azure Monitor**.</span><span class="sxs-lookup"><span data-stu-id="b096c-125">**Azure Monitor**.</span></span> <span data-ttu-id="b096c-126">O [Monitor][monitor] coleta métricas de desempenho sobre os serviços do Azure implantados na solução.</span><span class="sxs-lookup"><span data-stu-id="b096c-126">[Monitor][monitor] collects performance metrics about the Azure services deployed in the solution.</span></span> <span data-ttu-id="b096c-127">Ao visualizá-las em um painel, você obtém informações sobre a integridade da solução.</span><span class="sxs-lookup"><span data-stu-id="b096c-127">By visualizing these in a dashboard, you can get visibility  into the health of the solution.</span></span>

<span data-ttu-id="b096c-128">**Azure Pipelines**.</span><span class="sxs-lookup"><span data-stu-id="b096c-128">**Azure Pipelines**.</span></span> <span data-ttu-id="b096c-129">O [Pipelines][pipelines] é um serviço de CI (integração contínua) e CD (entrega contínua) que compila, testa e implanta o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b096c-129">[Pipelines][pipelines] is a continuous integration (CI) and continuous delivery (CD) service that builds, tests, and deploys the application.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="b096c-130">Considerações sobre escalabilidade</span><span class="sxs-lookup"><span data-stu-id="b096c-130">Scalability considerations</span></span>

### <a name="event-hubs"></a><span data-ttu-id="b096c-131">Hubs de Eventos</span><span class="sxs-lookup"><span data-stu-id="b096c-131">Event Hubs</span></span>

<span data-ttu-id="b096c-132">A capacidade da taxa de transferência dos Hubs de Eventos é medida em [unidades de produtividade][eh-throughput].</span><span class="sxs-lookup"><span data-stu-id="b096c-132">The throughput capacity of Event Hubs is measured in [throughput units][eh-throughput].</span></span> <span data-ttu-id="b096c-133">É possível fazer o dimensionamento automático de um hub de eventos ao permitir a [inflação automática][eh-autoscale], o que dimensiona automaticamente as unidades de produtividade com base no tráfego até um número máximo configurado.</span><span class="sxs-lookup"><span data-stu-id="b096c-133">You can autoscale an event hub by enabling [auto-inflate][eh-autoscale], which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

<span data-ttu-id="b096c-134">O [gatilho do Hub de Eventos][eh-trigger] no aplicativo de funções é dimensionado de acordo com o número de partições no hub de eventos.</span><span class="sxs-lookup"><span data-stu-id="b096c-134">The [Event Hub trigger][eh-trigger] in the function app scales according to the number of partitions in the event hub.</span></span> <span data-ttu-id="b096c-135">Cada partição recebe uma instância de função por vez.</span><span class="sxs-lookup"><span data-stu-id="b096c-135">Each partition is assigned one function instance at a time.</span></span> <span data-ttu-id="b096c-136">Para maximizar a taxa de transferência, receba os eventos em lote, em vez de um de cada vez.</span><span class="sxs-lookup"><span data-stu-id="b096c-136">To maximize throughput, receive the events in a batch, instead of one at a time.</span></span>

### <a name="cosmos-db"></a><span data-ttu-id="b096c-137">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="b096c-137">Cosmos DB</span></span>

<span data-ttu-id="b096c-138">A capacidade da taxa de transferência para o Cosmos DB é medida em [Unidades de Solicitação][ru] (RU).</span><span class="sxs-lookup"><span data-stu-id="b096c-138">Throughput capacity for Cosmos DB is measured in [Request Units][ru] (RU).</span></span> <span data-ttu-id="b096c-139">Para dimensionar um contêiner do Cosmos DB acima de 10.000 RU, é preciso especificar uma [chave de partição][partition-key] ao criar o contêiner e incluir a chave de partição em todos os documentos criados por você.</span><span class="sxs-lookup"><span data-stu-id="b096c-139">In order to scale a Cosmos DB container past 10,000 RU, you must specify a [partition key][partition-key] when you create the container, and include the partition key in every document that you create.</span></span>

<span data-ttu-id="b096c-140">Estas são algumas características de uma boa chave de partição:</span><span class="sxs-lookup"><span data-stu-id="b096c-140">Here are some characteristics of a good partition key:</span></span>

- <span data-ttu-id="b096c-141">O espaço para o valor de chave é grande.</span><span class="sxs-lookup"><span data-stu-id="b096c-141">The key value space is large.</span></span> 
- <span data-ttu-id="b096c-142">Haverá uma distribuição uniforme de leituras/gravações por valor de chave, evitando teclas de atalho.</span><span class="sxs-lookup"><span data-stu-id="b096c-142">There will be an even distribution of reads/writes per key value, avoiding hot keys.</span></span>
- <span data-ttu-id="b096c-143">O máximo de dados armazenado para qualquer valor de chave única não excederá o tamanho máximo da partição física (10 GB).</span><span class="sxs-lookup"><span data-stu-id="b096c-143">The maximum data stored for any single key value will not exceed the maximum physical partition size (10 GB).</span></span> 
- <span data-ttu-id="b096c-144">A chave de partição de um documento não será alterada.</span><span class="sxs-lookup"><span data-stu-id="b096c-144">The partition key for a document won't change.</span></span> <span data-ttu-id="b096c-145">Não é possível atualizar a chave de partição em um documento existente.</span><span class="sxs-lookup"><span data-stu-id="b096c-145">You can't update the partition key on an existing document.</span></span> 

<span data-ttu-id="b096c-146">No cenário dessa arquitetura de referência, a função armazena exatamente um documento por dispositivo que está enviando dados.</span><span class="sxs-lookup"><span data-stu-id="b096c-146">In the scenario for this reference architecture, the function stores exactly one document per device that is sending data.</span></span> <span data-ttu-id="b096c-147">A função atualiza continuamente os documentos com o status mais recente do dispositivo, usando uma operação upsert.</span><span class="sxs-lookup"><span data-stu-id="b096c-147">The function continually updates the documents with latest device status, using an upsert operation.</span></span> <span data-ttu-id="b096c-148">A ID do dispositivo é uma boa chave de partição para esse cenário, pois as gravações serão distribuídas uniformemente entre as chaves e o tamanho de cada partição será estritamente limitado, pois há um único documento para cada valor de chave.</span><span class="sxs-lookup"><span data-stu-id="b096c-148">Device ID is a good partition key for this scenario, because writes will be evenly distributed across the keys, and the size of each partition will be strictly bounded, because there is a single document for each key value.</span></span> <span data-ttu-id="b096c-149">Para saber mais sobre chaves de particionamento, confira [Particionar e dimensionar no Azure Cosmos DB][cosmosdb-scale].</span><span class="sxs-lookup"><span data-stu-id="b096c-149">For more information about partition keys, see [Partition and scale in Azure Cosmos DB][cosmosdb-scale].</span></span>

## <a name="resiliency-considerations"></a><span data-ttu-id="b096c-150">Considerações de resiliência</span><span class="sxs-lookup"><span data-stu-id="b096c-150">Resiliency considerations</span></span>

<span data-ttu-id="b096c-151">Ao usar o gatilho dos Hubs de Eventos com o Functions, capture exceções dentro de seu loop de processamento.</span><span class="sxs-lookup"><span data-stu-id="b096c-151">When using the Event Hubs trigger with Functions, catch exceptions within your processing loop.</span></span> <span data-ttu-id="b096c-152">Se ocorrer uma exceção sem tratamento, o tempo de execução do Functions não tentará novamente as mensagens.</span><span class="sxs-lookup"><span data-stu-id="b096c-152">If an unhandled exception occurs, the Functions runtime does not retry the messages.</span></span> <span data-ttu-id="b096c-153">Se não for possível processar uma mensagem, coloque-a em uma fila de mensagens mortas.</span><span class="sxs-lookup"><span data-stu-id="b096c-153">If a message cannot be processed, put the message into a dead letter queue.</span></span> <span data-ttu-id="b096c-154">Use um processo fora de banda para examinar as mensagens e determinar a ação corretiva.</span><span class="sxs-lookup"><span data-stu-id="b096c-154">Use an out-of-band process to examine the messages and determine corrective action.</span></span> 

<span data-ttu-id="b096c-155">O código a seguir mostra como a função de ingestão captura exceções e coloca as mensagens não processadas em uma fila de mensagens mortas.</span><span class="sxs-lookup"><span data-stu-id="b096c-155">The following code shows how the ingestion function catches exceptions and puts unprocessed messages onto a dead letter queue.</span></span>

```csharp
[FunctionName("RawTelemetryFunction")]
[StorageAccount("DeadLetterStorage")]
public static async Task RunAsync(
    [EventHubTrigger("%EventHubName%", Connection = "EventHubConnection", ConsumerGroup ="%EventHubConsumerGroup%")]EventData[] messages,
    [Queue("deadletterqueue")] IAsyncCollector<DeadLetterMessage> deadLetterMessages,
    ILogger logger)
{
    foreach (var message in messages)
    {
        DeviceState deviceState = null;

        try
        {
            deviceState = telemetryProcessor.Deserialize(message.Body.Array, logger);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error deserializing message", message.SystemProperties.PartitionKey, message.SystemProperties.SequenceNumber);
            await deadLetterMessages.AddAsync(new DeadLetterMessage { Issue = ex.Message, EventData = message });
        }

        try
        {
            await stateChangeProcessor.UpdateState(deviceState, logger);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error updating status document", deviceState);
            await deadLetterMessages.AddAsync(new DeadLetterMessage { Issue = ex.Message, EventData = message, DeviceState = deviceState });
        }
    }
}
```

<span data-ttu-id="b096c-156">Observe que a função usa a [associação de saída do armazenamento de filas][queue-binding] para colocar itens na fila.</span><span class="sxs-lookup"><span data-stu-id="b096c-156">Notice that the function uses the [Queue storage output binding][queue-binding] to put items in the queue.</span></span>

<span data-ttu-id="b096c-157">O código exibido acima também registra exceções em log para o Application Insights.</span><span class="sxs-lookup"><span data-stu-id="b096c-157">The code shown above also logs exceptions to Application Insights.</span></span> <span data-ttu-id="b096c-158">Use o número de sequência e a chave de partição para correlacionar mensagens mortas com as exceções nos logs.</span><span class="sxs-lookup"><span data-stu-id="b096c-158">You can use the partition key and sequence number to correlate dead letter messages with the exceptions in the logs.</span></span> 

<span data-ttu-id="b096c-159">As mensagens na fila de mensagens mortas devem ter informações suficientes para você entender o contexto do erro.</span><span class="sxs-lookup"><span data-stu-id="b096c-159">Messages in the dead letter queue should have enough information so that you can understand the context of error.</span></span> <span data-ttu-id="b096c-160">Neste exemplo, a classe `DeadLetterMessage` contém a mensagem de exceção, os dados do evento original e a mensagem de evento desserializada (se estiver disponível).</span><span class="sxs-lookup"><span data-stu-id="b096c-160">In this example, the `DeadLetterMessage` class contains the exception message, the original event data, and the deserialized event message (if available).</span></span> 

```csharp
public class DeadLetterMessage
{
    public string Issue { get; set; }
    public EventData EventData { get; set; }
    public DeviceState DeviceState { get; set; }
}
```

<span data-ttu-id="b096c-161">Use o [Azure Monitor][monitor] para monitorar o hub de eventos.</span><span class="sxs-lookup"><span data-stu-id="b096c-161">Use [Azure Monitor][monitor] to monitor the event hub.</span></span> <span data-ttu-id="b096c-162">Se você vir que há entrada, mas nenhuma saída, as mensagens não estarão sendo processadas.</span><span class="sxs-lookup"><span data-stu-id="b096c-162">If you see there is input but no output, it means that messages are not being processed.</span></span> <span data-ttu-id="b096c-163">Nesse caso, acesse o [Log Analytics][log-analytics] e procure exceções ou outros erros.</span><span class="sxs-lookup"><span data-stu-id="b096c-163">In that case, go into [Log Analytics][log-analytics] and look for exceptions or other errors.</span></span>

## <a name="disaster-recovery-considerations"></a><span data-ttu-id="b096c-164">Considerações de recuperação de desastres</span><span class="sxs-lookup"><span data-stu-id="b096c-164">Disaster recovery considerations</span></span>

<span data-ttu-id="b096c-165">A implantação exibida aqui reside em uma única região do Azure.</span><span class="sxs-lookup"><span data-stu-id="b096c-165">The deployment shown here resides in a single Azure region.</span></span> <span data-ttu-id="b096c-166">Para uma abordagem mais resiliente à recuperação de desastres, aproveite os recursos de distribuição geográfica de vários serviços:</span><span class="sxs-lookup"><span data-stu-id="b096c-166">For a more resilient approach to disaster-recovery, take advantage of geo-distribution features in the various services:</span></span>

- <span data-ttu-id="b096c-167">**Hubs de Evento**.</span><span class="sxs-lookup"><span data-stu-id="b096c-167">**Event Hubs**.</span></span> <span data-ttu-id="b096c-168">Crie dois namespaces de Hubs de Eventos, um namespace primário (ativo) e um namespace secundário (passivo).</span><span class="sxs-lookup"><span data-stu-id="b096c-168">Create two Event Hubs namespaces, a primary (active) namespace and a secondary (passive) namespace.</span></span> <span data-ttu-id="b096c-169">As mensagens são roteadas automaticamente para o namespace ativo, a menos que você faça o failover para o namespace secundário.</span><span class="sxs-lookup"><span data-stu-id="b096c-169">Messages are automatically routed to the active namespace unless you fail over to the secondary namespace.</span></span> <span data-ttu-id="b096c-170">Para saber mais, confira [Recuperação de desastre geográfico dos Hubs de Eventos do Azure][eh-dr].</span><span class="sxs-lookup"><span data-stu-id="b096c-170">For more information, see [Azure Event Hubs Geo-disaster recovery][eh-dr].</span></span>

- <span data-ttu-id="b096c-171">**Aplicativo de funções**.</span><span class="sxs-lookup"><span data-stu-id="b096c-171">**Function App**.</span></span> <span data-ttu-id="b096c-172">Implante um segundo aplicativo de função que está aguardando leitura do namespace secundário de Hubs de Eventos.</span><span class="sxs-lookup"><span data-stu-id="b096c-172">Deploy a second function app that is waiting to read from the secondary Event Hubs namespace.</span></span> <span data-ttu-id="b096c-173">Essa função grava em uma conta de armazenamento secundário para a fila de mensagens mortas.</span><span class="sxs-lookup"><span data-stu-id="b096c-173">This function writes to a secondary storage account for dead letter queue.</span></span>

- <span data-ttu-id="b096c-174">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="b096c-174">**Cosmos DB**.</span></span> <span data-ttu-id="b096c-175">O Cosmos DB dá suporte a [várias regiões mestres][cosmosdb-geo], o que permite a gravação em qualquer região adicionada à sua conta do Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="b096c-175">Cosmos DB supports [multiple master regions][cosmosdb-geo], which enables writes to any region that you add to your Cosmos DB account.</span></span> <span data-ttu-id="b096c-176">Se você não habilitar vários mestres, ainda poderá fazer o failover da região de gravação primária.</span><span class="sxs-lookup"><span data-stu-id="b096c-176">If you don’t enable multi-master, you can still fail over the primary write region.</span></span> <span data-ttu-id="b096c-177">Os SDKs de cliente do Cosmos DB e as associações do Azure Functions tratam automaticamente o failover, assim você não precisa atualizar as definições de configuração do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b096c-177">The Cosmos DB client SDKs and the Azure Function bindings automatically handle the failover, so you don’t need to update any application configuration settings.</span></span>

- <span data-ttu-id="b096c-178">**Armazenamento do Azure**.</span><span class="sxs-lookup"><span data-stu-id="b096c-178">**Azure Storage**.</span></span> <span data-ttu-id="b096c-179">Use o armazenamento [RA-GRS][ra-grs] para a fila de mensagens mortas.</span><span class="sxs-lookup"><span data-stu-id="b096c-179">Use [RA-GRS][ra-grs] storage for the dead letter queue.</span></span> <span data-ttu-id="b096c-180">Isso cria uma réplica somente leitura em outra região.</span><span class="sxs-lookup"><span data-stu-id="b096c-180">This creates a read-only replica in another region.</span></span> <span data-ttu-id="b096c-181">Se a região primária ficar indisponível, você poderá ler os itens atualmente na fila.</span><span class="sxs-lookup"><span data-stu-id="b096c-181">If the primary region becomes unavailable, you can read the items currently in the queue.</span></span> <span data-ttu-id="b096c-182">Além disso, provisione outra conta de armazenamento na região secundária para gravação da função após um failover.</span><span class="sxs-lookup"><span data-stu-id="b096c-182">In addition, provision another storage account in the secondary region that the function can write to after a fail-over.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="b096c-183">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="b096c-183">Deploy the solution</span></span>

<span data-ttu-id="b096c-184">Para implantar essa arquitetura de referência, confira o [Leiame do GitHub][readme].</span><span class="sxs-lookup"><span data-stu-id="b096c-184">To deploy this reference architecture, view the [GitHub readme][readme].</span></span> 

<!-- links -->

[cosmosdb]: /azure/cosmos-db/introduction
[cosmosdb-geo]: /azure/cosmos-db/distribute-data-globally
[cosmosdb-scale]: /azure/cosmos-db/partition-data
[cosmosdb-sql]: /azure/cosmos-db/sql-api-introduction
[eh]: /azure/event-hubs/
[eh-autoscale]: /azure/event-hubs/event-hubs-auto-inflate
[eh-dr]: /azure/event-hubs/event-hubs-geo-dr
[eh-throughput]: /azure/event-hubs/event-hubs-features#throughput-units
[eh-trigger]: /azure/azure-functions/functions-bindings-event-hubs
[functions]: /azure/azure-functions/functions-overview
[iot]: /azure/iot-hub/iot-hub-compare-event-hubs
[log-analytics]: /azure/log-analytics/log-analytics-queries
[monitor]: /azure/azure-monitor/overview
[partition-key]: /azure/cosmos-db/partition-data
[pipelines]: /azure/devops/pipelines/index
[queue]: /azure/storage/queues/storage-queues-introduction
[queue-binding]: /azure/azure-functions/functions-bindings-storage-queue#output
[ra-grs]: /azure/storage/common/storage-redundancy-grs
[ru]: /azure/cosmos-db/request-units

[github]: https://github.com/mspnp/serverless-reference-implementation
[readme]: https://github.com/mspnp/serverless-reference-implementation/blob/master/README.md