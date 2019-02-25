---
title: Processamento de fluxo com o Azure Stream Analytics
titleSuffix: Azure Reference Architectures
description: Crie um pipeline de processamento de fluxo de ponta a ponta no Azure.
author: MikeWasson
ms.date: 11/06/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: af56a010e90fb176b13c1110ad4000dcababe009
ms.sourcegitcommit: f4ed242dff8b204cfd8ebebb7778f356a19f5923
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/13/2019
ms.locfileid: "56224140"
---
# <a name="create-a-stream-processing-pipeline-with-azure-stream-analytics"></a><span data-ttu-id="f5254-103">Criar um pipeline de processamento de fluxo com o Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="f5254-103">Create a stream processing pipeline with Azure Stream Analytics</span></span>

<span data-ttu-id="f5254-104">Essa arquitetura de referência mostra um pipeline de [processamento de fluxo](/azure/architecture/data-guide/big-data/real-time-processing) de ponta a ponta.</span><span class="sxs-lookup"><span data-stu-id="f5254-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="f5254-105">O pipeline ingere dados de duas fontes, correlaciona os registros em dois fluxos e calcula uma média móvel em uma janela de tempo.</span><span class="sxs-lookup"><span data-stu-id="f5254-105">The pipeline ingests data from two sources, correlates records in the two streams, and calculates a rolling average across a time window.</span></span> <span data-ttu-id="f5254-106">Os resultados são armazenados para análise posterior.</span><span class="sxs-lookup"><span data-stu-id="f5254-106">The results are stored for further analysis.</span></span>

<span data-ttu-id="f5254-107">Há uma implantação de referência para essa arquitetura de referência disponível no [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="f5254-107">A reference implementation for this architecture is available on [GitHub][github].</span></span>

![Arquitetura de referência para a criação de um pipeline de processamento de fluxo com o Azure Stream Analytics](./images/stream-processing-asa/stream-processing-asa.png)

<span data-ttu-id="f5254-109">**Cenário**: uma empresa de táxi coleta dados sobre cada viagem.</span><span class="sxs-lookup"><span data-stu-id="f5254-109">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="f5254-110">Para esse cenário, assumimos que há dois dispositivos separados enviando dados.</span><span class="sxs-lookup"><span data-stu-id="f5254-110">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="f5254-111">O táxi tem um medidor que envia informações sobre cada corrida &mdash; duração, distância e locais de embarque e desembarque de passageiros.</span><span class="sxs-lookup"><span data-stu-id="f5254-111">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="f5254-112">Um dispositivo separado aceita pagamentos de clientes e envia dados sobre tarifas.</span><span class="sxs-lookup"><span data-stu-id="f5254-112">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="f5254-113">A empresa de táxi deseja calcular a média de gorjeta por quilômetro dirigido em tempo real para identificar tendências.</span><span class="sxs-lookup"><span data-stu-id="f5254-113">The taxi company wants to calculate the average tip per mile driven, in real time, in order to spot trends.</span></span>

## <a name="architecture"></a><span data-ttu-id="f5254-114">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="f5254-114">Architecture</span></span>

<span data-ttu-id="f5254-115">A arquitetura consiste nos componentes a seguir.</span><span class="sxs-lookup"><span data-stu-id="f5254-115">The architecture consists of the following components.</span></span>

<span data-ttu-id="f5254-116">**Fontes de dados**.</span><span class="sxs-lookup"><span data-stu-id="f5254-116">**Data sources**.</span></span> <span data-ttu-id="f5254-117">Nessa arquitetura há duas fontes de dados que geram fluxos de dados em tempo real.</span><span class="sxs-lookup"><span data-stu-id="f5254-117">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="f5254-118">O primeiro fluxo contém informações da corrida, e o segundo contém informações da tarifa.</span><span class="sxs-lookup"><span data-stu-id="f5254-118">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="f5254-119">A arquitetura de referência inclui um gerador de dados simulados que faz a leitura de um conjunto de arquivos estáticos e envia os dados para os Hubs de Eventos.</span><span class="sxs-lookup"><span data-stu-id="f5254-119">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="f5254-120">Em um aplicativo real, as fontes de dados seriam dispositivos instalados nos táxis.</span><span class="sxs-lookup"><span data-stu-id="f5254-120">In a real application, the data sources would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="f5254-121">**Hubs de Eventos do Azure**.</span><span class="sxs-lookup"><span data-stu-id="f5254-121">**Azure Event Hubs**.</span></span> <span data-ttu-id="f5254-122">[Hubs de Eventos](/azure/event-hubs/) são um serviço de ingestão de eventos.</span><span class="sxs-lookup"><span data-stu-id="f5254-122">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="f5254-123">Essa arquitetura usa duas instâncias de hub de eventos, uma para cada fonte de dados.</span><span class="sxs-lookup"><span data-stu-id="f5254-123">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="f5254-124">Cada fonte de dados envia um fluxo de dados para o hub de eventos associado.</span><span class="sxs-lookup"><span data-stu-id="f5254-124">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="f5254-125">**Azure Stream Analytics**.</span><span class="sxs-lookup"><span data-stu-id="f5254-125">**Azure Stream Analytics**.</span></span> <span data-ttu-id="f5254-126">[Stream Analytics](/azure/stream-analytics/) é um mecanismo de processamento de eventos.</span><span class="sxs-lookup"><span data-stu-id="f5254-126">[Stream Analytics](/azure/stream-analytics/) is an event-processing engine.</span></span> <span data-ttu-id="f5254-127">Um trabalho do Stream Analytics lê os fluxos de dados dos dois hubs de eventos e executa o processamento de fluxo.</span><span class="sxs-lookup"><span data-stu-id="f5254-127">A Stream Analytics job reads the data streams from the two event hubs and performs stream processing.</span></span>

<span data-ttu-id="f5254-128">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="f5254-128">**Cosmos DB**.</span></span> <span data-ttu-id="f5254-129">A saída do trabalho do Stream Analytics é uma série de registros gravados como documentos JSON para um banco de dados de documento do Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="f5254-129">The output from the Stream Analytics job is a series of records, which are written as JSON documents to a Cosmos DB document database.</span></span>

<span data-ttu-id="f5254-130">**Microsoft Power BI**.</span><span class="sxs-lookup"><span data-stu-id="f5254-130">**Microsoft Power BI**.</span></span> <span data-ttu-id="f5254-131">O Power BI é um conjunto de ferramentas de análise de negócios para analisar dados a fim de obter informações comerciais.</span><span class="sxs-lookup"><span data-stu-id="f5254-131">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="f5254-132">Nessa arquitetura, ele carrega os dados do Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="f5254-132">In this architecture, it loads the data from Cosmos DB.</span></span> <span data-ttu-id="f5254-133">Isso permite que os usuários analisem o conjunto completo de dados históricos que foram coletados.</span><span class="sxs-lookup"><span data-stu-id="f5254-133">This allows users to analyze the complete set of historical data that's been collected.</span></span> <span data-ttu-id="f5254-134">Também é possível transmitir os resultados diretamente do Stream Analytics para o Power BI para obter uma exibição em tempo real dos dados.</span><span class="sxs-lookup"><span data-stu-id="f5254-134">You could also stream the results directly from Stream Analytics to Power BI for a real-time view of the data.</span></span> <span data-ttu-id="f5254-135">Para obter mais informações, confira [Streaming em tempo real no Power BI](/power-bi/service-real-time-streaming).</span><span class="sxs-lookup"><span data-stu-id="f5254-135">For more information, see [Real-time streaming in Power BI](/power-bi/service-real-time-streaming).</span></span>

<span data-ttu-id="f5254-136">**Azure Monitor**.</span><span class="sxs-lookup"><span data-stu-id="f5254-136">**Azure Monitor**.</span></span> <span data-ttu-id="f5254-137">O [Azure Monitor](/azure/monitoring-and-diagnostics/) coleta métricas de desempenho sobre os serviços do Azure implantados na solução.</span><span class="sxs-lookup"><span data-stu-id="f5254-137">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects performance metrics about the Azure services deployed in the solution.</span></span> <span data-ttu-id="f5254-138">Ao visualizá-los em um painel, é possível obter informações sobre a integridade da solução.</span><span class="sxs-lookup"><span data-stu-id="f5254-138">By visualizing these in a dashboard, you can get insights into the health of the solution.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="f5254-139">Ingestão de dados</span><span class="sxs-lookup"><span data-stu-id="f5254-139">Data ingestion</span></span>

<!-- markdownlint-disable MD033 -->

<span data-ttu-id="f5254-140">Para simular uma fonte de dados, essa arquitetura de referência usa o conjunto de dados dos [Dados de táxi de Nova York](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797)<sup>[[1]](#note1)</sup>.</span><span class="sxs-lookup"><span data-stu-id="f5254-140">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="f5254-141">Esse conjunto de dados contém dados sobre viagens de táxi em Nova York durante um período de 4 anos (2010 &ndash; 2013).</span><span class="sxs-lookup"><span data-stu-id="f5254-141">This dataset contains data about taxi trips in New York City over a 4-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="f5254-142">Ele contém dois tipos de registro: Dados da corrida e da tarifa.</span><span class="sxs-lookup"><span data-stu-id="f5254-142">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="f5254-143">Os dados de corrida incluem a duração da viagem, a distância da viagem e os locais de embarque e desembarque de passageiros.</span><span class="sxs-lookup"><span data-stu-id="f5254-143">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="f5254-144">Os dados de tarifa incluem a tarifa, impostos e quantias das gorjetas.</span><span class="sxs-lookup"><span data-stu-id="f5254-144">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="f5254-145">Campos comuns em ambos os tipos de registro incluem o número da licença, carteira de habilitação e ID do fornecedor.</span><span class="sxs-lookup"><span data-stu-id="f5254-145">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="f5254-146">Juntos, esses três campos fazem a identificação exclusiva de um táxi e um motorista.</span><span class="sxs-lookup"><span data-stu-id="f5254-146">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="f5254-147">Os dados são armazenados no formato CSV.</span><span class="sxs-lookup"><span data-stu-id="f5254-147">The data is stored in CSV format.</span></span>

<span data-ttu-id="f5254-148"><span id="note1">[1]</span> Donovan, Brian; Work, Dan (2016): Dados de corridas de táxi na Cidade de Nova York (2010-2013).</span><span class="sxs-lookup"><span data-stu-id="f5254-148"><span id="note1">[1]</span> Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="f5254-149">Universidade de Illinois em Urbana-Champaign.</span><span class="sxs-lookup"><span data-stu-id="f5254-149">University of Illinois at Urbana-Champaign.</span></span> <span data-ttu-id="f5254-150">https://doi.org/10.13012/J8PN93H8</span><span class="sxs-lookup"><span data-stu-id="f5254-150">https://doi.org/10.13012/J8PN93H8</span></span>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="f5254-151">O gerador de dados é um aplicativo .NET Core que lê os registros e os envia para os Hubs de Eventos do Azure.</span><span class="sxs-lookup"><span data-stu-id="f5254-151">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="f5254-152">O gerador envia os dados de corrida em formato JSON e os dados de tarifa em formato CSV.</span><span class="sxs-lookup"><span data-stu-id="f5254-152">The generator sends ride data in JSON format and fare data in CSV format.</span></span>

<span data-ttu-id="f5254-153">Os Hubs de Eventos usam [partições](/azure/event-hubs/event-hubs-features#partitions) para segmentar os dados.</span><span class="sxs-lookup"><span data-stu-id="f5254-153">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="f5254-154">As partições permitem que um consumidor leia cada partição em paralelo.</span><span class="sxs-lookup"><span data-stu-id="f5254-154">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="f5254-155">Ao enviar dados para os Hubs de Eventos, é possível especificar a chave de partição explicitamente.</span><span class="sxs-lookup"><span data-stu-id="f5254-155">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="f5254-156">Caso contrário, os registros são atribuídos a partições no estilo round robin.</span><span class="sxs-lookup"><span data-stu-id="f5254-156">Otherwise, records are assigned to partitions in round-robin fashion.</span></span>

<span data-ttu-id="f5254-157">Nesse cenário específico, os dados de corrida e de tarifa devem ter a mesma ID de partição para um determinado táxi.</span><span class="sxs-lookup"><span data-stu-id="f5254-157">In this particular scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="f5254-158">Isso permite que o Stream Analytics aplique um grau de paralelismo ao correlacionar os dois fluxos.</span><span class="sxs-lookup"><span data-stu-id="f5254-158">This enables Stream Analytics to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="f5254-159">Um registro na partição *n* dos dados de corrida corresponderá a um registro na partição *n* dos dados de tarifa.</span><span class="sxs-lookup"><span data-stu-id="f5254-159">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![Diagrama de processamento de fluxo com o Azure Stream Analytics e os Hubs de Eventos](./images/stream-processing-asa/stream-processing-eh.png)

<span data-ttu-id="f5254-161">No gerador de dados, o modelo de dados comum para ambos os tipos de registro têm uma propriedade `PartitionKey` que é a concatenação de `Medallion`, `HackLicense` e `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="f5254-161">In the data generator, the common data model for both record types has a `PartitionKey` property which is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

```csharp
public abstract class TaxiData
{
    public TaxiData()
    {
    }

    [JsonProperty]
    public long Medallion { get; set; }

    [JsonProperty]
    public long HackLicense { get; set; }

    [JsonProperty]
    public string VendorId { get; set; }

    [JsonProperty]
    public DateTimeOffset PickupTime { get; set; }

    [JsonIgnore]
    public string PartitionKey
    {
        get => $"{Medallion}_{HackLicense}_{VendorId}";
    }
```

<span data-ttu-id="f5254-162">Essa propriedade é usada para fornecer uma chave de partição explícita ao enviar para Hubs de Eventos:</span><span class="sxs-lookup"><span data-stu-id="f5254-162">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

## <a name="stream-processing"></a><span data-ttu-id="f5254-163">Processamento de fluxo</span><span class="sxs-lookup"><span data-stu-id="f5254-163">Stream processing</span></span>

<span data-ttu-id="f5254-164">O trabalho de processamento de fluxo é definido usando uma consulta SQL com várias etapas distintas.</span><span class="sxs-lookup"><span data-stu-id="f5254-164">The stream processing job is defined using a SQL query with several distinct steps.</span></span> <span data-ttu-id="f5254-165">As duas primeiras etapas simplesmente selecionam registros dos dois fluxos de entrada.</span><span class="sxs-lookup"><span data-stu-id="f5254-165">The first two steps simply select records from the two input streams.</span></span>

```sql
WITH
Step1 AS (
    SELECT PartitionId,
           TRY_CAST(Medallion AS nvarchar(max)) AS Medallion,
           TRY_CAST(HackLicense AS nvarchar(max)) AS HackLicense,
           VendorId,
           TRY_CAST(PickupTime AS datetime) AS PickupTime,
           TripDistanceInMiles
    FROM [TaxiRide] PARTITION BY PartitionId
),
Step2 AS (
    SELECT PartitionId,
           medallion AS Medallion,
           hack_license AS HackLicense,
           vendor_id AS VendorId,
           TRY_CAST(pickup_datetime AS datetime) AS PickupTime,
           tip_amount AS TipAmount
    FROM [TaxiFare] PARTITION BY PartitionId
),
```

<span data-ttu-id="f5254-166">A próxima etapa une os dois fluxos de entrada para selecionar registros correspondentes de cada fluxo.</span><span class="sxs-lookup"><span data-stu-id="f5254-166">The next step joins the two input streams to select matching records from each stream.</span></span>

```sql
Step3 AS (
  SELECT
         tr.Medallion,
         tr.HackLicense,
         tr.VendorId,
         tr.PickupTime,
         tr.TripDistanceInMiles,
         tf.TipAmount
    FROM [Step1] tr
    PARTITION BY PartitionId
    JOIN [Step2] tf PARTITION BY PartitionId
      ON tr.Medallion = tf.Medallion
     AND tr.HackLicense = tf.HackLicense
     AND tr.VendorId = tf.VendorId
     AND tr.PickupTime = tf.PickupTime
     AND tr.PartitionId = tf.PartitionId
     AND DATEDIFF(minute, tr, tf) BETWEEN 0 AND 15
)
```

<span data-ttu-id="f5254-167">Essa consulta une registros em um conjunto de campos que identificam exclusivamente os registros correspondentes (Medallion, HackLicense, VendorId e PickupTime).</span><span class="sxs-lookup"><span data-stu-id="f5254-167">This query joins records on a set of fields that uniquely identify matching records (Medallion, HackLicense, VendorId, and PickupTime).</span></span> <span data-ttu-id="f5254-168">A instrução `JOIN` também inclui a ID da partição.</span><span class="sxs-lookup"><span data-stu-id="f5254-168">The `JOIN` statement also includes the partition ID.</span></span> <span data-ttu-id="f5254-169">Conforme mencionado, ela tira vantagem do fato de que registros de correspondência sempre têm a mesma ID de partição nesse cenário.</span><span class="sxs-lookup"><span data-stu-id="f5254-169">As mentioned, this takes advantage of the fact that matching records always have the same partition ID in this scenario.</span></span>

<span data-ttu-id="f5254-170">No Stream Analytics, as junções são *temporais*, o que significa que os registros são unidos em uma janela específica de tempo.</span><span class="sxs-lookup"><span data-stu-id="f5254-170">In Stream Analytics, joins are *temporal*, meaning records are joined within a particular window of time.</span></span> <span data-ttu-id="f5254-171">Caso contrário, o trabalho pode precisar aguardar indefinidamente por uma correspondência.</span><span class="sxs-lookup"><span data-stu-id="f5254-171">Otherwise, the job might need to wait indefinitely for a match.</span></span> <span data-ttu-id="f5254-172">A função [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) especifica o quanto dois registros correspondentes podem estar separados no tempo para haver uma correspondência.</span><span class="sxs-lookup"><span data-stu-id="f5254-172">The [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) function specifies how far two matching records can be separated in time for a match.</span></span>

<span data-ttu-id="f5254-173">A última etapa no trabalho calcula a média de gorjeta por quilômetro, agrupada por uma janela de salto de 5 minutos.</span><span class="sxs-lookup"><span data-stu-id="f5254-173">The last step in the job computes the average tip per mile, grouped by a hopping window of 5 minutes.</span></span>

```sql
SELECT System.Timestamp AS WindowTime,
       SUM(tr.TipAmount) / SUM(tr.TripDistanceInMiles) AS AverageTipPerMile
  INTO [TaxiDrain]
  FROM [Step3] tr
  GROUP BY HoppingWindow(Duration(minute, 5), Hop(minute, 1))
```

<span data-ttu-id="f5254-174">O Stream Analytics fornece várias [funções de janelas](/azure/stream-analytics/stream-analytics-window-functions).</span><span class="sxs-lookup"><span data-stu-id="f5254-174">Stream Analytics provides several [windowing functions](/azure/stream-analytics/stream-analytics-window-functions).</span></span> <span data-ttu-id="f5254-175">Uma janela de salto se move para a frente no tempo conforme um período fixo, nesse caso, 1 minuto por salto.</span><span class="sxs-lookup"><span data-stu-id="f5254-175">A hopping window moves forward in time by a fixed period, in this case 1 minute per hop.</span></span> <span data-ttu-id="f5254-176">O resultado é o cálculo de uma média móvel nos últimos 5 minutos.</span><span class="sxs-lookup"><span data-stu-id="f5254-176">The result is to calculate a moving average over the past 5 minutes.</span></span>

<span data-ttu-id="f5254-177">Na arquitetura mostrada aqui, apenas os resultados do trabalho do Stream Analytics são salvos no Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="f5254-177">In the architecture shown here, only the results of the Stream Analytics job are saved to Cosmos DB.</span></span> <span data-ttu-id="f5254-178">Para um cenário de Big Data, considere usar também uma [Captura de Hubs de Eventos](/azure/event-hubs/event-hubs-capture-overview) para salvar os dados brutos de evento no armazenamento de Blobs do Azure.</span><span class="sxs-lookup"><span data-stu-id="f5254-178">For a big data scenario, consider also using [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) to save the raw event data into Azure Blob storage.</span></span> <span data-ttu-id="f5254-179">Ao manter os dados brutos, é possível executar consultas em lote de seus dados históricos em um momento posterior para derivar novos insights de dados.</span><span class="sxs-lookup"><span data-stu-id="f5254-179">Keeping the raw data will allow you to run batch queries over your historical data at later time, in order to derive new insights from the data.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="f5254-180">Considerações sobre escalabilidade</span><span class="sxs-lookup"><span data-stu-id="f5254-180">Scalability considerations</span></span>

### <a name="event-hubs"></a><span data-ttu-id="f5254-181">Hubs de Eventos</span><span class="sxs-lookup"><span data-stu-id="f5254-181">Event Hubs</span></span>

<span data-ttu-id="f5254-182">A capacidade da taxa de transferência dos Hubs de Eventos é medida em [unidades de produtividade](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="f5254-182">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="f5254-183">É possível fazer o dimensionamento automático de um hub de eventos ao permitir a [inflação automática](/azure/event-hubs/event-hubs-auto-inflate), o que dimensiona automaticamente as unidades de produtividade com base no tráfego até um número máximo configurado.</span><span class="sxs-lookup"><span data-stu-id="f5254-183">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

### <a name="stream-analytics"></a><span data-ttu-id="f5254-184">Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="f5254-184">Stream Analytics</span></span>

<span data-ttu-id="f5254-185">Para o Stream Analytics, os recursos de computação alocados em um trabalho são medidos em unidades de streaming.</span><span class="sxs-lookup"><span data-stu-id="f5254-185">For Stream Analytics, the computing resources allocated to a job are measured in Streaming Units.</span></span> <span data-ttu-id="f5254-186">Os trabalhos do Stream Analytics executam um dimensionamento melhor caso o trabalho possa ser paralelizado.</span><span class="sxs-lookup"><span data-stu-id="f5254-186">Stream Analytics jobs scale best if the job can be parallelized.</span></span> <span data-ttu-id="f5254-187">Dessa forma, o Stream Analytics pode distribuir o trabalho entre vários nós de computação.</span><span class="sxs-lookup"><span data-stu-id="f5254-187">That way, Stream Analytics can distribute the job across multiple compute nodes.</span></span>

<span data-ttu-id="f5254-188">Para a entrada de Hubs de Eventos, use a palavra-chave `PARTITION BY` para particionar o trabalho do Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="f5254-188">For Event Hubs input, use the `PARTITION BY` keyword to partition the Stream Analytics job.</span></span> <span data-ttu-id="f5254-189">Os dados serão divididos em subconjuntos com base nas partições dos Hubs de Eventos.</span><span class="sxs-lookup"><span data-stu-id="f5254-189">The data will be divided into subsets based on the Event Hubs partitions.</span></span>

<span data-ttu-id="f5254-190">Funções de janelas e uniões temporais exigem UAs adicionais.</span><span class="sxs-lookup"><span data-stu-id="f5254-190">Windowing functions and temporal joins require additional SU.</span></span> <span data-ttu-id="f5254-191">Quando possível, use `PARTITION BY` para que cada partição seja processada separadamente.</span><span class="sxs-lookup"><span data-stu-id="f5254-191">When possible, use `PARTITION BY` so that each partition is processed separately.</span></span> <span data-ttu-id="f5254-192">Para saber mais, confira [Compreender e ajustar unidades de streaming](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span><span class="sxs-lookup"><span data-stu-id="f5254-192">For more information, see [Understand and adjust Streaming Units](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span></span>

<span data-ttu-id="f5254-193">Se não for possível paralelizar todo o trabalho do Stream Analytics, tente dividir o trabalho em várias etapas, começando com uma ou mais etapas paralelas.</span><span class="sxs-lookup"><span data-stu-id="f5254-193">If it's not possible to parallelize the entire Stream Analytics job, try to break the job into multiple steps, starting with one or more parallel steps.</span></span> <span data-ttu-id="f5254-194">Dessa forma, as primeiras etapas conseguem ser executadas em paralelo.</span><span class="sxs-lookup"><span data-stu-id="f5254-194">That way, the first steps can run in parallel.</span></span> <span data-ttu-id="f5254-195">Por exemplo, nesta arquitetura de referência:</span><span class="sxs-lookup"><span data-stu-id="f5254-195">For example, in this reference architecture:</span></span>

- <span data-ttu-id="f5254-196">As etapas 1 e 2 são instruções `SELECT` simples que selecionam registros dentro de uma única partição.</span><span class="sxs-lookup"><span data-stu-id="f5254-196">Steps 1 and 2 are simple `SELECT` statements that select records within a single partition.</span></span>
- <span data-ttu-id="f5254-197">A etapa 3 realiza uma junção particionada em dois fluxos de entrada.</span><span class="sxs-lookup"><span data-stu-id="f5254-197">Step 3 performs a partitioned join across two input streams.</span></span> <span data-ttu-id="f5254-198">Essa etapa tira proveito do fato de que os registros correspondentes compartilham a mesma chave de partição e, assim, é garantido que terão a mesma ID de partição em cada fluxo de entrada.</span><span class="sxs-lookup"><span data-stu-id="f5254-198">This step takes advantage of the fact that matching records share the same partition key, and so are guaranteed to have the same partition ID in each input stream.</span></span>
- <span data-ttu-id="f5254-199">A etapa 4 agrega em todas as partições.</span><span class="sxs-lookup"><span data-stu-id="f5254-199">Step 4 aggregates across all of the partitions.</span></span> <span data-ttu-id="f5254-200">Essa etapa não pode ser paralelizada.</span><span class="sxs-lookup"><span data-stu-id="f5254-200">This step cannot be parallelized.</span></span>

<span data-ttu-id="f5254-201">Use o [diagrama de trabalho](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) do Stream Analytics para ver quantas partições são atribuídas a cada etapa no trabalho.</span><span class="sxs-lookup"><span data-stu-id="f5254-201">Use the Stream Analytics [job diagram](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) to see how many partitions are assigned to each step in the job.</span></span> <span data-ttu-id="f5254-202">O diagrama a seguir mostra o diagrama de trabalho dessa arquitetura de referência:</span><span class="sxs-lookup"><span data-stu-id="f5254-202">The following diagram shows the job diagram for this reference architecture:</span></span>

![Diagrama do trabalho](./images/stream-processing-asa/job-diagram.png)

### <a name="cosmos-db"></a><span data-ttu-id="f5254-204">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="f5254-204">Cosmos DB</span></span>

<span data-ttu-id="f5254-205">A capacidade da taxa de transferência para o Cosmos DB é medida em [Unidades de Solicitação](/azure/cosmos-db/request-units) (RU).</span><span class="sxs-lookup"><span data-stu-id="f5254-205">Throughput capacity for Cosmos DB is measured in [Request Units](/azure/cosmos-db/request-units) (RU).</span></span> <span data-ttu-id="f5254-206">Para dimensionar um contêiner do Cosmos DB após 10.000 RU, é preciso especificar uma [chave de partição](/azure/cosmos-db/partition-data) ao criar o contêiner, além de incluir a chave de partição em todos os documentos.</span><span class="sxs-lookup"><span data-stu-id="f5254-206">In order to scale a Cosmos DB container past 10,000 RU, you must specify a [partition key](/azure/cosmos-db/partition-data) when you create the container, and include the partition key in every document.</span></span>

<span data-ttu-id="f5254-207">Nessa arquitetura de referência, os novos documentos são criados apenas uma vez por minuto (o intervalo da janela de salto), portanto, os requisitos de taxa de transferência são um valor muito baixo.</span><span class="sxs-lookup"><span data-stu-id="f5254-207">In this reference architecture, new documents are created only once per minute (the hopping window interval), so the throughput requirements are quite low.</span></span> <span data-ttu-id="f5254-208">Por esse motivo, não é preciso atribuir uma chave de partição nesse cenário.</span><span class="sxs-lookup"><span data-stu-id="f5254-208">For that reason, there's no need to assign a partition key in this scenario.</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="f5254-209">Considerações de monitoramento</span><span class="sxs-lookup"><span data-stu-id="f5254-209">Monitoring considerations</span></span>

<span data-ttu-id="f5254-210">Com qualquer solução de processamento de fluxo, é importante monitorar o desempenho e a integridade do sistema.</span><span class="sxs-lookup"><span data-stu-id="f5254-210">With any stream processing solution, it's important to monitor the performance and health of the system.</span></span> <span data-ttu-id="f5254-211">O [Azure Monitor](/azure/monitoring-and-diagnostics/) coleta métricas e logs de diagnóstico dos serviços do Azure usados na arquitetura.</span><span class="sxs-lookup"><span data-stu-id="f5254-211">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects metrics and diagnostics logs for the Azure services used in the architecture.</span></span> <span data-ttu-id="f5254-212">O Azure Monitor é criado na plataforma do Azure e não exige nenhum código adicional em seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="f5254-212">Azure Monitor is built into the Azure platform and does not require any additional code in your application.</span></span>

<span data-ttu-id="f5254-213">Qualquer um dos sinais de aviso a seguir indica que você deve escalar horizontalmente o recurso relevante do Azure:</span><span class="sxs-lookup"><span data-stu-id="f5254-213">Any of the following warning signals indicate that you should scale out the relevant Azure resource:</span></span>

- <span data-ttu-id="f5254-214">Os Hubs de Eventos limitam as solicitações ou a cota diária de mensagens está próxima.</span><span class="sxs-lookup"><span data-stu-id="f5254-214">Event Hubs throttles requests or is close to the daily message quota.</span></span>
- <span data-ttu-id="f5254-215">O trabalho do Stream Analytics usa, consistentemente, mais de 80% das unidades de streaming alocadas (SU).</span><span class="sxs-lookup"><span data-stu-id="f5254-215">The Stream Analytics job consistently uses more than 80% of allocated Streaming Units (SU).</span></span>
- <span data-ttu-id="f5254-216">O Cosmos DB começa a limitar solicitações.</span><span class="sxs-lookup"><span data-stu-id="f5254-216">Cosmos DB begins to throttle requests.</span></span>

<span data-ttu-id="f5254-217">A arquitetura de referência inclui um painel personalizado, que é implantado no portal do Azure.</span><span class="sxs-lookup"><span data-stu-id="f5254-217">The reference architecture includes a custom dashboard, which is deployed to the Azure portal.</span></span> <span data-ttu-id="f5254-218">Depois de implantar a arquitetura, é possível exibir o painel. Para isso, abra o [Portal do Azure](https://portal.azure.com) e selecione `TaxiRidesDashboard` na lista de painéis.</span><span class="sxs-lookup"><span data-stu-id="f5254-218">After you deploy the architecture, you can view the dashboard by opening the [Azure Portal](https://portal.azure.com) and selecting `TaxiRidesDashboard` from list of dashboards.</span></span> <span data-ttu-id="f5254-219">Para saber mais sobre a criação e implantação de painéis personalizados no portal do Azure, confira [Criar painéis do Azure programaticamente](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span><span class="sxs-lookup"><span data-stu-id="f5254-219">For more information about creating and deploying custom dashboards in the Azure portal, see [Programmatically create Azure Dashboards](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span></span>

<span data-ttu-id="f5254-220">A imagem a seguir mostra o painel depois que o trabalho do Stream Analytics foi executado por aproximadamente uma hora.</span><span class="sxs-lookup"><span data-stu-id="f5254-220">The following image shows the dashboard after the Stream Analytics job ran for about an hour.</span></span>

![Captura de tela do dashboard Corridas de táxi](./images/stream-processing-asa/asa-dashboard.png)

<span data-ttu-id="f5254-222">O painel na parte inferior esquerda mostra que o consumo de SU do trabalho do Stream Analytics aumenta durante os primeiros 15 minutos e diminui em seguida.</span><span class="sxs-lookup"><span data-stu-id="f5254-222">The panel on the lower left shows that the SU consumption for the Stream Analytics job climbs during the first 15 minutes and then levels off.</span></span> <span data-ttu-id="f5254-223">Isso é um padrão típico conforme o trabalho atinge um estado estável.</span><span class="sxs-lookup"><span data-stu-id="f5254-223">This is a typical pattern as the job reaches a steady state.</span></span>

<span data-ttu-id="f5254-224">Observe que os Hubs de Eventos está limitando as solicitações, o que é mostrado no painel superior à direita.</span><span class="sxs-lookup"><span data-stu-id="f5254-224">Notice that Event Hubs is throttling requests, shown in the upper right panel.</span></span> <span data-ttu-id="f5254-225">Uma solicitação ser limitada ocasionalmente não é um problema, porque o SDK do cliente dos Hubs de Eventos faz novas tentativas automaticamente ao receber um erro de limitação.</span><span class="sxs-lookup"><span data-stu-id="f5254-225">An occasional throttled request is not a problem, because the Event Hubs client SDK automatically retries when it receives a throttling error.</span></span> <span data-ttu-id="f5254-226">No entanto, se houver erros de limitação consistentes, isso significa que o hub de eventos precisa de mais unidades de produtividade.</span><span class="sxs-lookup"><span data-stu-id="f5254-226">However, if you see consistent throttling errors, it means the event hub needs more throughput units.</span></span> <span data-ttu-id="f5254-227">O gráfico a seguir mostra uma execução de teste usando o recurso de inflação automática dos Hubs de Eventos, o que dimensiona automaticamente as unidades de produtividade conforme o necessário.</span><span class="sxs-lookup"><span data-stu-id="f5254-227">The following graph shows a test run using the Event Hubs auto-inflate feature, which automatically scales out the throughput units as needed.</span></span>

![Captura de tela de dimensionamento automático de Hubs de Eventos](./images/stream-processing-asa/stream-processing-eh-autoscale.png)

<span data-ttu-id="f5254-229">A inflação automática foi habilitada por volta da marca de 06:35.</span><span class="sxs-lookup"><span data-stu-id="f5254-229">Auto-inflate was enabled at about the 06:35 mark.</span></span> <span data-ttu-id="f5254-230">É possível ver a queda de p em solicitações limitadas, uma vez que os Hubs de Eventos dimensionam automaticamente até 3 unidades de produtividade.</span><span class="sxs-lookup"><span data-stu-id="f5254-230">You can see the p drop in throttled requests, as Event Hubs automatically scaled up to 3 throughput units.</span></span>

<span data-ttu-id="f5254-231">Curiosamente, isso criava o efeito colateral de aumento da utilização de SU no trabalho do Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="f5254-231">Interestingly, this had the side effect of increasing the SU utilization in the Stream Analytics job.</span></span> <span data-ttu-id="f5254-232">Ao fazer a limitação, os Hubs de Eventos foram reduzindo artificialmente a taxa de ingestão para o trabalho do Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="f5254-232">By throttling, Event Hubs was artificially reducing the ingestion rate for the Stream Analytics job.</span></span> <span data-ttu-id="f5254-233">É muito comum que a resolução de um gargalo no desempenho revele outra.</span><span class="sxs-lookup"><span data-stu-id="f5254-233">It's actually common that resolving one performance bottleneck reveals another.</span></span> <span data-ttu-id="f5254-234">Nesse caso, a alocação de SU adicional ao trabalho do Stream Analytics resolveu o problema.</span><span class="sxs-lookup"><span data-stu-id="f5254-234">In this case, allocating additional SU for the Stream Analytics job resolved the issue.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="f5254-235">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="f5254-235">Deploy the solution</span></span>

<span data-ttu-id="f5254-236">Para a implantação e execução da implementação de referência, siga as etapas em [Leia-me do GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="f5254-236">To the deploy and run the reference implementation, follow the steps in the [GitHub readme][github].</span></span>

## <a name="related-resources"></a><span data-ttu-id="f5254-237">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="f5254-237">Related resources</span></span>

<span data-ttu-id="f5254-238">Talvez seja melhor examinar os seguintes [cenários de exemplo do Azure](/azure/architecture/example-scenario), que demonstram soluções específicas usando algumas das mesmas tecnologias:</span><span class="sxs-lookup"><span data-stu-id="f5254-238">You may wish to review the following [Azure example scenarios](/azure/architecture/example-scenario) that demonstrate specific solutions using some of the same technologies:</span></span>

- [<span data-ttu-id="f5254-239">Análise de dados e de IoT no setor de construção civil</span><span class="sxs-lookup"><span data-stu-id="f5254-239">IoT and data analytics in the construction industry</span></span>](/azure/architecture/example-scenario/data/big-data-with-iot)
- [<span data-ttu-id="f5254-240">Detecção de fraude em tempo real</span><span class="sxs-lookup"><span data-stu-id="f5254-240">Real-time fraud detection</span></span>](/azure/architecture/example-scenario/data/fraud-detection)

<!-- links -->

[github]: https://github.com/mspnp/azure-stream-analytics-data-pipeline

