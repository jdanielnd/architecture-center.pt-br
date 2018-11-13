---
title: Processamento de fluxo com o Azure Stream Analytics
description: Criar um pipeline de processamento de fluxo de ponta a ponta no Azure
author: MikeWasson
ms.date: 11/06/2018
ms.openlocfilehash: e16547ccdcb81007e154e341f09be555ac82d1a1
ms.sourcegitcommit: 02ecd259a6e780d529c853bc1db320f4fcf919da
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/07/2018
ms.locfileid: "51263755"
---
# <a name="stream-processing-with-azure-stream-analytics"></a><span data-ttu-id="7bb3e-103">Processamento de fluxo com o Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="7bb3e-103">Stream processing with Azure Stream Analytics</span></span>

<span data-ttu-id="7bb3e-104">Essa arquitetura de referência mostra um pipeline de processamento de fluxo de ponta a ponta.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-104">This reference architecture shows an end-to-end stream processing pipeline.</span></span> <span data-ttu-id="7bb3e-105">O pipeline ingere dados de duas fontes, correlaciona os registros em dois fluxos e calcula uma média móvel em uma janela de tempo.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-105">The pipeline ingests data from two sources, correlates records in the two streams, and calculates a rolling average across a time window.</span></span> <span data-ttu-id="7bb3e-106">Os resultados são armazenados para análise posterior.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-106">The results are stored for further analysis.</span></span> 

<span data-ttu-id="7bb3e-107">Há uma implantação de referência para essa arquitetura de referência disponível no [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="7bb3e-107">A reference implementation for this architecture is available on [GitHub][github].</span></span> 

![](./images/stream-processing-asa/stream-processing-asa.png)

<span data-ttu-id="7bb3e-108">**Cenário**: uma empresa de táxi coleta dados sobre cada viagem.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-108">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="7bb3e-109">Para esse cenário, assumimos que há dois dispositivos separados enviando dados.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-109">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="7bb3e-110">O táxi tem um medidor que envia informações sobre cada corrida &mdash; duração, distância e locais de embarque e desembarque de passageiros.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-110">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="7bb3e-111">Um dispositivo separado aceita pagamentos de clientes e envia dados sobre tarifas.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-111">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="7bb3e-112">A empresa de táxi deseja calcular a média de gorjeta por quilômetro dirigido em tempo real para identificar tendências.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-112">The taxi company wants to calculate the average tip per mile driven, in real time, in order to spot trends.</span></span>

## <a name="architecture"></a><span data-ttu-id="7bb3e-113">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="7bb3e-113">Architecture</span></span>

<span data-ttu-id="7bb3e-114">A arquitetura consiste nos componentes a seguir.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-114">The architecture consists of the following components.</span></span>

<span data-ttu-id="7bb3e-115">**Fontes de dados**.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-115">**Data sources**.</span></span> <span data-ttu-id="7bb3e-116">Nessa arquitetura há duas fontes de dados que geram fluxos de dados em tempo real.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-116">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="7bb3e-117">O primeiro fluxo contém informações da corrida, e o segundo contém informações da tarifa.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-117">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="7bb3e-118">A arquitetura de referência inclui um gerador de dados simulados que faz a leitura de um conjunto de arquivos estáticos e envia os dados para os Hubs de Eventos.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-118">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="7bb3e-119">Em um aplicativo real, as fontes de dados seriam dispositivos instalados nos táxis.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-119">In a real application, the data sources would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="7bb3e-120">**Hubs de Eventos do Azure**.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-120">**Azure Event Hubs**.</span></span> <span data-ttu-id="7bb3e-121">[Hubs de Eventos](/azure/event-hubs/) são um serviço de ingestão de eventos.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-121">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="7bb3e-122">Essa arquitetura usa duas instâncias de hub de eventos, uma para cada fonte de dados.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-122">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="7bb3e-123">Cada fonte de dados envia um fluxo de dados para o hub de eventos associado.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-123">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="7bb3e-124">**Azure Stream Analytics**.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-124">**Azure Stream Analytics**.</span></span> <span data-ttu-id="7bb3e-125">[Stream Analytics](/azure/stream-analytics/) é um mecanismo de processamento de eventos.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-125">[Stream Analytics](/azure/stream-analytics/) is an event-processing engine.</span></span> <span data-ttu-id="7bb3e-126">Um trabalho do Stream Analytics lê os fluxos de dados dos dois hubs de eventos e executa o processamento de fluxo.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-126">A Stream Analytics job reads the data streams from the two event hubs and performs stream processing.</span></span>

<span data-ttu-id="7bb3e-127">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-127">**Cosmos DB**.</span></span> <span data-ttu-id="7bb3e-128">A saída do trabalho do Stream Analytics é uma série de registros gravados como documentos JSON para um banco de dados de documento do Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-128">The output from the Stream Analytics job is a series of records, which are written as JSON documents to a Cosmos DB document database.</span></span>

<span data-ttu-id="7bb3e-129">**Microsoft Power BI**.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-129">**Microsoft Power BI**.</span></span> <span data-ttu-id="7bb3e-130">O Power BI é um conjunto de ferramentas de análise de negócios para analisar dados a fim de obter informações comerciais.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-130">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="7bb3e-131">Nessa arquitetura, ele carrega os dados do Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-131">In this architecture, it loads the data from Cosmos DB.</span></span> <span data-ttu-id="7bb3e-132">Isso permite que os usuários analisem o conjunto completo de dados históricos que foram coletados.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-132">This allows users to analyze the complete set of historical data that's been collected.</span></span> <span data-ttu-id="7bb3e-133">Também é possível transmitir os resultados diretamente do Stream Analytics para o Power BI para obter uma exibição em tempo real dos dados.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-133">You could also stream the results directly from Stream Analytics to Power BI for a real-time view of the data.</span></span> <span data-ttu-id="7bb3e-134">Para obter mais informações, confira [Streaming em tempo real no Power BI](/power-bi/service-real-time-streaming).</span><span class="sxs-lookup"><span data-stu-id="7bb3e-134">For more information, see [Real-time streaming in Power BI](/power-bi/service-real-time-streaming).</span></span>

<span data-ttu-id="7bb3e-135">**Azure Monitor**.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-135">**Azure Monitor**.</span></span> <span data-ttu-id="7bb3e-136">O [Azure Monitor](/azure/monitoring-and-diagnostics/) coleta métricas de desempenho sobre os serviços do Azure implantados na solução.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-136">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects performance metrics about the Azure services deployed in the solution.</span></span> <span data-ttu-id="7bb3e-137">Ao visualizá-los em um painel, é possível obter informações sobre a integridade da solução.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-137">By visualizing these in a dashboard, you can get insights into the health of the solution.</span></span> 

## <a name="data-ingestion"></a><span data-ttu-id="7bb3e-138">Ingestão de dados</span><span class="sxs-lookup"><span data-stu-id="7bb3e-138">Data ingestion</span></span>

<span data-ttu-id="7bb3e-139">Para simular uma fonte de dados, essa arquitetura de referência usa o conjunto de dados dos [Dados de táxi de Nova York](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797)<sup>[[1]](#note1)</sup>.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-139">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="7bb3e-140">Esse conjunto de dados contém dados sobre viagens de táxi em Nova York durante um período de 4 anos (2010 &ndash; 2013).</span><span class="sxs-lookup"><span data-stu-id="7bb3e-140">This dataset contains data about taxi trips in New York City over a 4-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="7bb3e-141">Ele contém dois tipos de registro: dados de corrida e dados de tarifa.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-141">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="7bb3e-142">Os dados de corrida incluem a duração da viagem, a distância da viagem e os locais de embarque e desembarque de passageiros.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-142">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="7bb3e-143">Os dados de tarifa incluem a tarifa, impostos e quantias das gorjetas.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-143">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="7bb3e-144">Campos comuns em ambos os tipos de registro incluem o número da licença, carteira de habilitação e ID do fornecedor.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-144">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="7bb3e-145">Juntos, esses três campos fazem a identificação exclusiva de um táxi e um motorista.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-145">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="7bb3e-146">Os dados são armazenados no formato CSV.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-146">The data is stored in CSV format.</span></span> 

<span data-ttu-id="7bb3e-147">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span><span class="sxs-lookup"><span data-stu-id="7bb3e-147">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="7bb3e-148">Universidade de Illinois em Urbana-Champaign.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-148">University of Illinois at Urbana-Champaign.</span></span> <span data-ttu-id="7bb3e-149">https://doi.org/10.13012/J8PN93H8</span><span class="sxs-lookup"><span data-stu-id="7bb3e-149">https://doi.org/10.13012/J8PN93H8</span></span>

<span data-ttu-id="7bb3e-150">O gerador de dados é um aplicativo .NET Core que lê os registros e os envia para os Hubs de Eventos do Azure.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-150">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="7bb3e-151">O gerador envia os dados de corrida em formato JSON e os dados de tarifa em formato CSV.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-151">The generator sends ride data in JSON format and fare data in CSV format.</span></span> 

<span data-ttu-id="7bb3e-152">Os Hubs de Eventos usam [partições](/azure/event-hubs/event-hubs-features#partitions) para segmentar os dados.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-152">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="7bb3e-153">As partições permitem que um consumidor leia cada partição em paralelo.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-153">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="7bb3e-154">Ao enviar dados para os Hubs de Eventos, é possível especificar a chave de partição explicitamente.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-154">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="7bb3e-155">Caso contrário, os registros são atribuídos a partições no estilo round robin.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-155">Otherwise, records are assigned to partitions in round-robin fashion.</span></span> 

<span data-ttu-id="7bb3e-156">Nesse cenário específico, os dados de corrida e de tarifa devem ter a mesma ID de partição para um determinado táxi.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-156">In this particular scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="7bb3e-157">Isso permite que o Stream Analytics aplique um grau de paralelismo ao correlacionar os dois fluxos.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-157">This enables Stream Analytics to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="7bb3e-158">Um registro na partição *n* dos dados de corrida corresponderá a um registro na partição *n* dos dados de tarifa.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-158">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![](./images/stream-processing-asa/stream-processing-eh.png)

<span data-ttu-id="7bb3e-159">No gerador de dados, o modelo de dados comum para ambos os tipos de registro têm uma propriedade `PartitionKey` que é a concatenação de `Medallion`, `HackLicense` e `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-159">In the data generator, the common data model for both record types has a `PartitionKey` property which is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="7bb3e-160">Essa propriedade é usada para fornecer uma chave de partição explícita ao enviar para Hubs de Eventos:</span><span class="sxs-lookup"><span data-stu-id="7bb3e-160">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

## <a name="stream-processing"></a><span data-ttu-id="7bb3e-161">Processamento de fluxo</span><span class="sxs-lookup"><span data-stu-id="7bb3e-161">Stream processing</span></span>

<span data-ttu-id="7bb3e-162">O trabalho de processamento de fluxo é definido usando uma consulta SQL com várias etapas distintas.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-162">The stream processing job is defined using a SQL query with several distinct steps.</span></span> <span data-ttu-id="7bb3e-163">As duas primeiras etapas simplesmente selecionam registros dos dois fluxos de entrada.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-163">The first two steps simply select records from the two input streams.</span></span>

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

<span data-ttu-id="7bb3e-164">A próxima etapa une os dois fluxos de entrada para selecionar registros correspondentes de cada fluxo.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-164">The next step joins the two input streams to select matching records from each stream.</span></span>

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

<span data-ttu-id="7bb3e-165">Essa consulta une registros em um conjunto de campos que identificam exclusivamente os registros correspondentes (Medallion, HackLicense, VendorId e PickupTime).</span><span class="sxs-lookup"><span data-stu-id="7bb3e-165">This query joins records on a set of fields that uniquely identify matching records (Medallion, HackLicense, VendorId, and PickupTime).</span></span> <span data-ttu-id="7bb3e-166">A instrução `JOIN` também inclui a ID da partição.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-166">The `JOIN` statement also includes the partition ID.</span></span> <span data-ttu-id="7bb3e-167">Conforme mencionado, ela tira vantagem do fato de que registros de correspondência sempre têm a mesma ID de partição nesse cenário.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-167">As mentioned, this takes advantage of the fact that matching records always have the same partition ID in this scenario.</span></span>

<span data-ttu-id="7bb3e-168">No Stream Analytics, as junções são *temporais*, o que significa que os registros são unidos em uma janela específica de tempo.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-168">In Stream Analytics, joins are *temporal*, meaning records are joined within a particular window of time.</span></span> <span data-ttu-id="7bb3e-169">Caso contrário, o trabalho pode precisar aguardar indefinidamente por uma correspondência.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-169">Otherwise, the job might need to wait indefinitely for a match.</span></span> <span data-ttu-id="7bb3e-170">A função [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) especifica o quanto dois registros correspondentes podem estar separados no tempo para haver uma correspondência.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-170">The [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) function specifies how far two matching records can be separated in time for a match.</span></span> 

<span data-ttu-id="7bb3e-171">A última etapa no trabalho calcula a média de gorjeta por quilômetro, agrupada por uma janela de salto de 5 minutos.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-171">The last step in the job computes the average tip per mile, grouped by a hopping window of 5 minutes.</span></span>

```sql
SELECT System.Timestamp AS WindowTime,
       SUM(tr.TipAmount) / SUM(tr.TripDistanceInMiles) AS AverageTipPerMile
  INTO [TaxiDrain]
  FROM [Step3] tr
  GROUP BY HoppingWindow(Duration(minute, 5), Hop(minute, 1))
```

<span data-ttu-id="7bb3e-172">O Stream Analytics fornece várias [funções de janelas](/azure/stream-analytics/stream-analytics-window-functions).</span><span class="sxs-lookup"><span data-stu-id="7bb3e-172">Stream Analytics provides several [windowing functions](/azure/stream-analytics/stream-analytics-window-functions).</span></span> <span data-ttu-id="7bb3e-173">Uma janela de salto se move para a frente no tempo conforme um período fixo, nesse caso, 1 minuto por salto.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-173">A hopping window moves forward in time by a fixed period, in this case 1 minute per hop.</span></span> <span data-ttu-id="7bb3e-174">O resultado é o cálculo de uma média móvel nos últimos 5 minutos.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-174">The result is to calculate a moving average over the past 5 minutes.</span></span>

<span data-ttu-id="7bb3e-175">Na arquitetura mostrada aqui, apenas os resultados do trabalho do Stream Analytics são salvos no Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-175">In the architecture shown here, only the results of the Stream Analytics job are saved to Cosmos DB.</span></span> <span data-ttu-id="7bb3e-176">Para um cenário de Big Data, considere usar também uma [Captura de Hubs de Eventos](/azure/event-hubs/event-hubs-capture-overview) para salvar os dados brutos de evento no armazenamento de Blobs do Azure.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-176">For a big data scenario, consider also using [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) to save the raw event data into Azure Blob storage.</span></span> <span data-ttu-id="7bb3e-177">Ao manter os dados brutos, é possível executar consultas em lote de seus dados históricos em um momento posterior para derivar novos insights de dados.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-177">Keeping the raw data will allow you to run batch queries over your historical data at later time, in order to derive new insights from the data.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="7bb3e-178">Considerações sobre escalabilidade</span><span class="sxs-lookup"><span data-stu-id="7bb3e-178">Scalability considerations</span></span>

### <a name="event-hubs"></a><span data-ttu-id="7bb3e-179">Hubs de Eventos</span><span class="sxs-lookup"><span data-stu-id="7bb3e-179">Event Hubs</span></span>

<span data-ttu-id="7bb3e-180">A capacidade da taxa de transferência dos Hubs de Eventos é medida em [unidades de produtividade](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="7bb3e-180">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="7bb3e-181">É possível fazer o dimensionamento automático de um hub de eventos ao permitir a [inflação automática](/azure/event-hubs/event-hubs-auto-inflate), o que dimensiona automaticamente as unidades de produtividade com base no tráfego até um número máximo configurado.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-181">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span> 

### <a name="stream-analytics"></a><span data-ttu-id="7bb3e-182">Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="7bb3e-182">Stream Analytics</span></span>

<span data-ttu-id="7bb3e-183">Para o Stream Analytics, os recursos de computação alocados em um trabalho são medidos em unidades de streaming.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-183">For Stream Analytics, the computing resources allocated to a job are measured in Streaming Units.</span></span> <span data-ttu-id="7bb3e-184">Os trabalhos do Stream Analytics executam um dimensionamento melhor caso o trabalho possa ser paralelizado.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-184">Stream Analytics jobs scale best if the job can be parallelized.</span></span> <span data-ttu-id="7bb3e-185">Dessa forma, o Stream Analytics pode distribuir o trabalho entre vários nós de computação.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-185">That way, Stream Analytics can distribute the job across multiple compute nodes.</span></span>

<span data-ttu-id="7bb3e-186">Para a entrada de Hubs de Eventos, use a palavra-chave `PARTITION BY` para particionar o trabalho do Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-186">For Event Hubs input, use the `PARTITION BY` keyword to partition the Stream Analytics job.</span></span> <span data-ttu-id="7bb3e-187">Os dados serão divididos em subconjuntos com base nas partições dos Hubs de Eventos.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-187">The data will be divided into subsets based on the Event Hubs partitions.</span></span> 

<span data-ttu-id="7bb3e-188">Funções de janelas e uniões temporais exigem UAs adicionais.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-188">Windowing functions and temporal joins require additional SU.</span></span> <span data-ttu-id="7bb3e-189">Quando possível, use `PARTITION BY` para que cada partição seja processada separadamente.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-189">When possible, use `PARTITION BY` so that each partition is processed separately.</span></span> <span data-ttu-id="7bb3e-190">Para saber mais, confira [Compreender e ajustar unidades de streaming](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span><span class="sxs-lookup"><span data-stu-id="7bb3e-190">For more information, see [Understand and adjust Streaming Units](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span></span>

<span data-ttu-id="7bb3e-191">Se não for possível paralelizar todo o trabalho do Stream Analytics, tente dividir o trabalho em várias etapas, começando com uma ou mais etapas paralelas.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-191">If it's not possible to parallelize the entire Stream Analytics job, try to break the job into multiple steps, starting with one or more parallel steps.</span></span> <span data-ttu-id="7bb3e-192">Dessa forma, as primeiras etapas conseguem ser executadas em paralelo.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-192">That way, the first steps can run in parallel.</span></span> <span data-ttu-id="7bb3e-193">Por exemplo, nesta arquitetura de referência:</span><span class="sxs-lookup"><span data-stu-id="7bb3e-193">For example, in this reference architecture:</span></span>

- <span data-ttu-id="7bb3e-194">As etapas 1 e 2 são instruções `SELECT` simples que selecionam registros dentro de uma única partição.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-194">Steps 1 and 2 are simple `SELECT` statements that select records within a single partition.</span></span> 
- <span data-ttu-id="7bb3e-195">A etapa 3 realiza uma junção particionada em dois fluxos de entrada.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-195">Step 3 performs a partitioned join across two input streams.</span></span> <span data-ttu-id="7bb3e-196">Essa etapa tira proveito do fato de que os registros correspondentes compartilham a mesma chave de partição e, assim, é garantido que terão a mesma ID de partição em cada fluxo de entrada.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-196">This step takes advantage of the fact that matching records share the same partition key, and so are guaranteed to have the same partition ID in each input stream.</span></span>
- <span data-ttu-id="7bb3e-197">A etapa 4 agrega em todas as partições.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-197">Step 4 aggregates across all of the partitions.</span></span> <span data-ttu-id="7bb3e-198">Essa etapa não pode ser paralelizada.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-198">This step cannot be parallelized.</span></span>

<span data-ttu-id="7bb3e-199">Use o [diagrama de trabalho](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) do Stream Analytics para ver quantas partições são atribuídas a cada etapa no trabalho.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-199">Use the Stream Analytics [job diagram](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) to see how many partitions are assigned to each step in the job.</span></span> <span data-ttu-id="7bb3e-200">O diagrama a seguir mostra o diagrama de trabalho dessa arquitetura de referência:</span><span class="sxs-lookup"><span data-stu-id="7bb3e-200">The following diagram shows the job diagram for this reference architecture:</span></span>

![](./images/stream-processing-asa/job-diagram.png)

### <a name="cosmos-db"></a><span data-ttu-id="7bb3e-201">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="7bb3e-201">Cosmos DB</span></span>

<span data-ttu-id="7bb3e-202">A capacidade da taxa de transferência para o Cosmos DB é medida em [Unidades de Solicitação](/azure/cosmos-db/request-units) (RU).</span><span class="sxs-lookup"><span data-stu-id="7bb3e-202">Throughput capacity for Cosmos DB is measured in [Request Units](/azure/cosmos-db/request-units) (RU).</span></span> <span data-ttu-id="7bb3e-203">Para dimensionar um contêiner do Cosmos DB após 10.000 RU, é preciso especificar uma [chave de partição](/azure/cosmos-db/partition-data) ao criar o contêiner, além de incluir a chave de partição em todos os documentos.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-203">In order to scale a Cosmos DB container past 10,000 RU, you must specify a [partition key](/azure/cosmos-db/partition-data) when you create the container, and include the partition key in every document.</span></span> 

<span data-ttu-id="7bb3e-204">Nessa arquitetura de referência, os novos documentos são criados apenas uma vez por minuto (o intervalo da janela de salto), portanto, os requisitos de taxa de transferência são um valor muito baixo.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-204">In this reference architecture, new documents are created only once per minute (the hopping window interval), so the throughput requirements are quite low.</span></span> <span data-ttu-id="7bb3e-205">Por esse motivo, não é preciso atribuir uma chave de partição nesse cenário.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-205">For that reason, there's no need to assign a partition key in this scenario.</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="7bb3e-206">Considerações de monitoramento</span><span class="sxs-lookup"><span data-stu-id="7bb3e-206">Monitoring considerations</span></span>

<span data-ttu-id="7bb3e-207">Com qualquer solução de processamento de fluxo, é importante monitorar o desempenho e a integridade do sistema.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-207">With any stream processing solution, it's important to monitor the performance and health of the system.</span></span> <span data-ttu-id="7bb3e-208">O [Azure Monitor](/azure/monitoring-and-diagnostics/) coleta métricas e logs de diagnóstico dos serviços do Azure usados na arquitetura.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-208">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects metrics and diagnostics logs for the Azure services used in the architecture.</span></span> <span data-ttu-id="7bb3e-209">O Azure Monitor é criado na plataforma do Azure e não exige nenhum código adicional em seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-209">Azure Monitor is built into the Azure platform and does not require any additional code in your application.</span></span>

<span data-ttu-id="7bb3e-210">Qualquer um dos sinais de aviso a seguir indica que você deve escalar horizontalmente o recurso relevante do Azure:</span><span class="sxs-lookup"><span data-stu-id="7bb3e-210">Any of the following warning signals indicate that you should scale out the relevant Azure resource:</span></span>

- <span data-ttu-id="7bb3e-211">Os Hubs de Eventos limitam as solicitações ou a cota diária de mensagens está próxima.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-211">Event Hubs throttles requests or is close to the daily message quota.</span></span>
- <span data-ttu-id="7bb3e-212">O trabalho do Stream Analytics usa, consistentemente, mais de 80% das unidades de streaming alocadas (SU).</span><span class="sxs-lookup"><span data-stu-id="7bb3e-212">The Stream Analytics job consistently uses more than 80% of allocated Streaming Units (SU).</span></span>
- <span data-ttu-id="7bb3e-213">O Cosmos DB começa a limitar solicitações.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-213">Cosmos DB begins to throttle requests.</span></span>

<span data-ttu-id="7bb3e-214">A arquitetura de referência inclui um painel personalizado, que é implantado no portal do Azure.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-214">The reference architecture includes a custom dashboard, which is deployed to the Azure portal.</span></span> <span data-ttu-id="7bb3e-215">Depois de implantar a arquitetura, é possível exibir o painel. Para isso, abra o [Portal do Azure](https://portal.azure.com) e selecione `TaxiRidesDashboard` na lista de painéis.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-215">After you deploy the architecture, you can view the dashboard by opening the [Azure Portal](https://portal.azure.com) and selecting `TaxiRidesDashboard` from list of dashboards.</span></span> <span data-ttu-id="7bb3e-216">Para saber mais sobre a criação e implantação de painéis personalizados no portal do Azure, confira [Criar painéis do Azure programaticamente](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span><span class="sxs-lookup"><span data-stu-id="7bb3e-216">For more information about creating and deploying custom dashboards in the Azure portal, see [Programmatically create Azure Dashboards](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span></span>

<span data-ttu-id="7bb3e-217">A imagem a seguir mostra o painel depois que o trabalho do Stream Analytics foi executado por aproximadamente uma hora.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-217">The following image shows the dashboard after the Stream Analytics job ran for about an hour.</span></span>

![](./images/stream-processing-asa/asa-dashboard.png)

<span data-ttu-id="7bb3e-218">O painel na parte inferior esquerda mostra que o consumo de SU do trabalho do Stream Analytics aumenta durante os primeiros 15 minutos e diminui em seguida.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-218">The panel on the lower left shows that the SU consumption for the Stream Analytics job climbs during the first 15 minutes and then levels off.</span></span> <span data-ttu-id="7bb3e-219">Isso é um padrão típico conforme o trabalho atinge um estado estável.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-219">This is a typical pattern as the job reaches a steady state.</span></span> 

<span data-ttu-id="7bb3e-220">Observe que os Hubs de Eventos está limitando as solicitações, o que é mostrado no painel superior à direita.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-220">Notice that Event Hubs is throttling requests, shown in the upper right panel.</span></span> <span data-ttu-id="7bb3e-221">Uma solicitação ser limitada ocasionalmente não é um problema, porque o SDK do cliente dos Hubs de Eventos faz novas tentativas automaticamente ao receber um erro de limitação.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-221">An occasional throttled request is not a problem, because the Event Hubs client SDK automatically retries when it receives a throttling error.</span></span> <span data-ttu-id="7bb3e-222">No entanto, se houver erros de limitação consistentes, isso significa que o hub de eventos precisa de mais unidades de produtividade.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-222">However, if you see consistent throttling errors, it means the event hub needs more throughput units.</span></span> <span data-ttu-id="7bb3e-223">O gráfico a seguir mostra uma execução de teste usando o recurso de inflação automática dos Hubs de Eventos, o que dimensiona automaticamente as unidades de produtividade conforme o necessário.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-223">The following graph shows a test run using the Event Hubs auto-inflate feature, which automatically scales out the throughput units as needed.</span></span> 

![](./images/stream-processing-asa/stream-processing-eh-autoscale.png)

<span data-ttu-id="7bb3e-224">A inflação automática foi habilitada por volta da marca de 06:35.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-224">Auto-inflate was enabled at about the 06:35 mark.</span></span> <span data-ttu-id="7bb3e-225">É possível ver a queda de p em solicitações limitadas, uma vez que os Hubs de Eventos dimensionam automaticamente até 3 unidades de produtividade.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-225">You can see the p drop in throttled requests, as Event Hubs automatically scaled up to 3 throughput units.</span></span>

<span data-ttu-id="7bb3e-226">Curiosamente, isso criava o efeito colateral de aumento da utilização de SU no trabalho do Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-226">Interestingly, this had the side effect of increasing the SU utilization in the Stream Analytics job.</span></span> <span data-ttu-id="7bb3e-227">Ao fazer a limitação, os Hubs de Eventos foram reduzindo artificialmente a taxa de ingestão para o trabalho do Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-227">By throttling, Event Hubs was artificially reducing the ingestion rate for the Stream Analytics job.</span></span> <span data-ttu-id="7bb3e-228">É muito comum que a resolução de um gargalo no desempenho revele outra.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-228">It's actually common that resolving one performance bottleneck reveals another.</span></span> <span data-ttu-id="7bb3e-229">Nesse caso, a alocação de SU adicional ao trabalho do Stream Analytics resolveu o problema.</span><span class="sxs-lookup"><span data-stu-id="7bb3e-229">In this case, allocating additional SU for the Stream Analytics job resolved the issue.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="7bb3e-230">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="7bb3e-230">Deploy the solution</span></span>

<span data-ttu-id="7bb3e-231">Para a implantação e execução da implementação de referência, siga as etapas em [Leia-me do GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="7bb3e-231">To the deploy and run the reference implementation, follow the steps in the [GitHub readme][github].</span></span> 


[github]: https://github.com/mspnp/reference-architectures/tree/master/data/streaming_asa