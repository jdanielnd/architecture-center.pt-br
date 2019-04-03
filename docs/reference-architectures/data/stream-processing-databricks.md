---
title: Processamento de fluxo com o Azure Databricks
titleSuffix: Azure Reference Architectures
description: Criar um pipeline de processamento de fluxo de ponta a ponta no Azure usando o Azure Databricks.
author: petertaylor9999
ms.date: 11/30/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: 3d109cb830b7dfc8c3d4de0e654f9d8667acf101
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887787"
---
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a><span data-ttu-id="88679-103">Criar um pipeline de processamento de fluxo com o Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="88679-103">Create a stream processing pipeline with Azure Databricks</span></span>

<span data-ttu-id="88679-104">Essa arquitetura de referência mostra um pipeline de [processamento de fluxo](/azure/architecture/data-guide/big-data/real-time-processing) de ponta a ponta.</span><span class="sxs-lookup"><span data-stu-id="88679-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="88679-105">Esse tipo de pipeline tem quatro estágios: ingerir, processar, armazenar e analisar e gerar relatórios.</span><span class="sxs-lookup"><span data-stu-id="88679-105">This type of pipeline has four stages: ingest, process, store, and analysis and reporting.</span></span> <span data-ttu-id="88679-106">Para essa arquitetura de referência, o pipeline ingere dados de duas origens, realiza uma junção em registros relacionados de cada fluxo, enriquece o resultado e calcula uma média em tempo real.</span><span class="sxs-lookup"><span data-stu-id="88679-106">For this reference architecture, the pipeline ingests data from two sources, performs a join on related records from each stream, enriches the result, and calculates an average in real time.</span></span> <span data-ttu-id="88679-107">Os resultados são armazenados para análise posterior.</span><span class="sxs-lookup"><span data-stu-id="88679-107">The results are stored for further analysis.</span></span> <span data-ttu-id="88679-108">[**Implantar esta solução**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="88679-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Arquitetura de referência para processamento de fluxo com o Azure Databricks](./images/stream-processing-databricks.png)

<span data-ttu-id="88679-110">**Cenário**: uma empresa de táxi coleta dados sobre cada viagem.</span><span class="sxs-lookup"><span data-stu-id="88679-110">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="88679-111">Para esse cenário, assumimos que há dois dispositivos separados enviando dados.</span><span class="sxs-lookup"><span data-stu-id="88679-111">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="88679-112">O táxi tem um medidor que envia informações sobre cada corrida &mdash; duração, distância e locais de embarque e desembarque de passageiros.</span><span class="sxs-lookup"><span data-stu-id="88679-112">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="88679-113">Um dispositivo separado aceita pagamentos de clientes e envia dados sobre tarifas.</span><span class="sxs-lookup"><span data-stu-id="88679-113">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="88679-114">Para identificar as tendências dos passageiros, a empresa de táxi deseja calcular a média de gorjeta por quilômetro percorrido em tempo real, para cada área.</span><span class="sxs-lookup"><span data-stu-id="88679-114">To spot ridership trends, the taxi company wants to calculate the average tip per mile driven, in real time, for each neighborhood.</span></span>

## <a name="architecture"></a><span data-ttu-id="88679-115">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="88679-115">Architecture</span></span>

<span data-ttu-id="88679-116">A arquitetura consiste nos componentes a seguir.</span><span class="sxs-lookup"><span data-stu-id="88679-116">The architecture consists of the following components.</span></span>

<span data-ttu-id="88679-117">**Fontes de dados**.</span><span class="sxs-lookup"><span data-stu-id="88679-117">**Data sources**.</span></span> <span data-ttu-id="88679-118">Nessa arquitetura há duas fontes de dados que geram fluxos de dados em tempo real.</span><span class="sxs-lookup"><span data-stu-id="88679-118">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="88679-119">O primeiro fluxo contém informações da corrida, e o segundo contém informações da tarifa.</span><span class="sxs-lookup"><span data-stu-id="88679-119">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="88679-120">A arquitetura de referência inclui um gerador de dados simulados que faz a leitura de um conjunto de arquivos estáticos e envia os dados para os Hubs de Eventos.</span><span class="sxs-lookup"><span data-stu-id="88679-120">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="88679-121">Em um aplicativo real, as fontes de dados seriam dispositivos instalados nos táxis.</span><span class="sxs-lookup"><span data-stu-id="88679-121">The data sources in a real application would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="88679-122">**Hubs de Eventos do Azure**.</span><span class="sxs-lookup"><span data-stu-id="88679-122">**Azure Event Hubs**.</span></span> <span data-ttu-id="88679-123">[Hubs de Eventos](/azure/event-hubs/) são um serviço de ingestão de eventos.</span><span class="sxs-lookup"><span data-stu-id="88679-123">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="88679-124">Essa arquitetura usa duas instâncias de hub de eventos, uma para cada fonte de dados.</span><span class="sxs-lookup"><span data-stu-id="88679-124">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="88679-125">Cada fonte de dados envia um fluxo de dados para o hub de eventos associado.</span><span class="sxs-lookup"><span data-stu-id="88679-125">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="88679-126">**Azure Databricks**.</span><span class="sxs-lookup"><span data-stu-id="88679-126">**Azure Databricks**.</span></span> <span data-ttu-id="88679-127">O [Databricks](/azure/azure-databricks/) é uma plataforma de análise baseada no Apache Spark otimizada para a plataforma de Serviços de Nuvem do Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="88679-127">[Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform optimized for the Microsoft Azure cloud services platform.</span></span> <span data-ttu-id="88679-128">O Databricks é usado para correlacionar os dados de corrida e de tarifa do táxi e também para enriquecer os dados correlacionados com dados armazenados sobre áreas no sistema de arquivos do Databricks.</span><span class="sxs-lookup"><span data-stu-id="88679-128">Databricks is used to correlate of the taxi ride and fare data, and also to enrich the correlated data with neighborhood data stored in the Databricks file system.</span></span>

<span data-ttu-id="88679-129">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="88679-129">**Cosmos DB**.</span></span> <span data-ttu-id="88679-130">A saída do trabalho do Azure Databricks é uma série de registros que são gravados no [Cosmos DB](/azure/cosmos-db/) usando a API do Cassandra.</span><span class="sxs-lookup"><span data-stu-id="88679-130">The output from Azure Databricks job is a series of records, which are written to [Cosmos DB](/azure/cosmos-db/) using the Cassandra API.</span></span> <span data-ttu-id="88679-131">A API do Cassandra é usada porque oferece suporte à modelagem de dados de série temporal.</span><span class="sxs-lookup"><span data-stu-id="88679-131">The Cassandra API is used because it supports time series data modeling.</span></span>

<span data-ttu-id="88679-132">**Azure Log Analytics**.</span><span class="sxs-lookup"><span data-stu-id="88679-132">**Azure Log Analytics**.</span></span> <span data-ttu-id="88679-133">Os dados de log do aplicativo coletados pelo [Monitor do Azure](/azure/monitoring-and-diagnostics/) são armazenados em um [espaço de trabalho do Log Analytics](/azure/log-analytics).</span><span class="sxs-lookup"><span data-stu-id="88679-133">Application log data collected by [Azure Monitor](/azure/monitoring-and-diagnostics/) is stored in a [Log Analytics workspace](/azure/log-analytics).</span></span> <span data-ttu-id="88679-134">As consultas do Log Analytics podem ser usadas para analisar e visualizar métricas e inspecionar mensagens de log para identificar problemas no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="88679-134">Log Analytics queries can be used to analyze and visualize metrics and inspect log messages to identify issues within the application.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="88679-135">Ingestão de dados</span><span class="sxs-lookup"><span data-stu-id="88679-135">Data ingestion</span></span>

<!-- markdownlint-disable MD033 -->

<span data-ttu-id="88679-136">Para simular uma fonte de dados, essa arquitetura de referência usa o conjunto de dados dos [Dados de táxi de Nova York](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797)<sup>[[1]](#note1)</sup>.</span><span class="sxs-lookup"><span data-stu-id="88679-136">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="88679-137">Esse conjunto de dados contém dados sobre viagens de táxi em Nova York durante um período de 4 anos (2010 &ndash; 2013).</span><span class="sxs-lookup"><span data-stu-id="88679-137">This dataset contains data about taxi trips in New York City over a four-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="88679-138">Ele contém dois tipos de registro: Dados da corrida e da tarifa.</span><span class="sxs-lookup"><span data-stu-id="88679-138">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="88679-139">Os dados de corrida incluem a duração da viagem, a distância da viagem e os locais de embarque e desembarque de passageiros.</span><span class="sxs-lookup"><span data-stu-id="88679-139">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="88679-140">Os dados de tarifa incluem a tarifa, impostos e quantias das gorjetas.</span><span class="sxs-lookup"><span data-stu-id="88679-140">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="88679-141">Campos comuns em ambos os tipos de registro incluem o número da licença, carteira de habilitação e ID do fornecedor.</span><span class="sxs-lookup"><span data-stu-id="88679-141">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="88679-142">Juntos, esses três campos fazem a identificação exclusiva de um táxi e um motorista.</span><span class="sxs-lookup"><span data-stu-id="88679-142">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="88679-143">Os dados são armazenados no formato CSV.</span><span class="sxs-lookup"><span data-stu-id="88679-143">The data is stored in CSV format.</span></span>

> <span data-ttu-id="88679-144">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): Dados de corridas de táxi na Cidade de Nova York (2010-2013).</span><span class="sxs-lookup"><span data-stu-id="88679-144">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="88679-145">Universidade de Illinois em Urbana-Champaign.</span><span class="sxs-lookup"><span data-stu-id="88679-145">University of Illinois at Urbana-Champaign.</span></span> <https://doi.org/10.13012/J8PN93H8>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="88679-146">O gerador de dados é um aplicativo .NET Core que lê os registros e os envia para os Hubs de Eventos do Azure.</span><span class="sxs-lookup"><span data-stu-id="88679-146">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="88679-147">O gerador envia os dados de corrida em formato JSON e os dados de tarifa em formato CSV.</span><span class="sxs-lookup"><span data-stu-id="88679-147">The generator sends ride data in JSON format and fare data in CSV format.</span></span>

<span data-ttu-id="88679-148">Os Hubs de Eventos usam [partições](/azure/event-hubs/event-hubs-features#partitions) para segmentar os dados.</span><span class="sxs-lookup"><span data-stu-id="88679-148">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="88679-149">As partições permitem que um consumidor leia cada partição em paralelo.</span><span class="sxs-lookup"><span data-stu-id="88679-149">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="88679-150">Ao enviar dados para os Hubs de Eventos, é possível especificar a chave de partição explicitamente.</span><span class="sxs-lookup"><span data-stu-id="88679-150">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="88679-151">Caso contrário, os registros são atribuídos a partições no estilo round robin.</span><span class="sxs-lookup"><span data-stu-id="88679-151">Otherwise, records are assigned to partitions in round-robin fashion.</span></span>

<span data-ttu-id="88679-152">Nesse cenário, os dados de corrida e de tarifa devem ter a mesma ID de partição para um determinado táxi.</span><span class="sxs-lookup"><span data-stu-id="88679-152">In this scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="88679-153">Isso permite que o Databricks aplique um grau de paralelismo ao correlacionar os dois fluxos.</span><span class="sxs-lookup"><span data-stu-id="88679-153">This enables Databricks to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="88679-154">Um registro na partição *n* dos dados de corrida corresponderá a um registro na partição *n* dos dados de tarifa.</span><span class="sxs-lookup"><span data-stu-id="88679-154">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![Diagrama de processamento de fluxo com o Azure Databricks e os Hubs de Eventos](./images/stream-processing-databricks-eh.png)

<span data-ttu-id="88679-156">No gerador de dados, o modelo de dados comum para ambos os tipos de registro têm uma propriedade `PartitionKey` que é a concatenação de `Medallion`, `HackLicense` e `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="88679-156">In the data generator, the common data model for both record types has a `PartitionKey` property that is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="88679-157">Essa propriedade é usada para fornecer uma chave de partição explícita ao enviar para Hubs de Eventos:</span><span class="sxs-lookup"><span data-stu-id="88679-157">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a><span data-ttu-id="88679-158">Hubs de Eventos</span><span class="sxs-lookup"><span data-stu-id="88679-158">Event Hubs</span></span>

<span data-ttu-id="88679-159">A capacidade da taxa de transferência dos Hubs de Eventos é medida em [unidades de produtividade](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="88679-159">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="88679-160">É possível fazer o dimensionamento automático de um hub de eventos ao permitir a [inflação automática](/azure/event-hubs/event-hubs-auto-inflate), o que dimensiona automaticamente as unidades de produtividade com base no tráfego até um número máximo configurado.</span><span class="sxs-lookup"><span data-stu-id="88679-160">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

## <a name="stream-processing"></a><span data-ttu-id="88679-161">Processamento de fluxo</span><span class="sxs-lookup"><span data-stu-id="88679-161">Stream processing</span></span>

<span data-ttu-id="88679-162">No Azure Databricks, o processamento de dados é executado por um trabalho.</span><span class="sxs-lookup"><span data-stu-id="88679-162">In Azure Databricks, data processing is performed by a job.</span></span> <span data-ttu-id="88679-163">O trabalho é atribuído a um cluster e é executado nele.</span><span class="sxs-lookup"><span data-stu-id="88679-163">The job is assigned to and runs on a cluster.</span></span> <span data-ttu-id="88679-164">O trabalho pode ser o código personalizado escrito em Java ou um [bloco de notas](https://docs.databricks.com/user-guide/notebooks/index.html) do Spark.</span><span class="sxs-lookup"><span data-stu-id="88679-164">The job can either be custom code written in Java, or a Spark [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span></span>

<span data-ttu-id="88679-165">Nessa arquitetura de referência, o trabalho é um arquivo morto de Java com classes escritas em Java e Scala.</span><span class="sxs-lookup"><span data-stu-id="88679-165">In this reference architecture, the job is a Java archive with classes written in both Java and Scala.</span></span> <span data-ttu-id="88679-166">Ao especificar o arquivo morto de Java para um trabalho do Databricks, a classe será especificada para ser executada pelo cluster do Databricks.</span><span class="sxs-lookup"><span data-stu-id="88679-166">When specifying the Java archive for a Databricks job, the class is specified for execution by the Databricks cluster.</span></span> <span data-ttu-id="88679-167">Aqui, o método **main** da classe **com.microsoft.pnp.TaxiCabReader** contém a lógica de processamento de dados.</span><span class="sxs-lookup"><span data-stu-id="88679-167">Here, the **main** method of the **com.microsoft.pnp.TaxiCabReader** class contains the data processing logic.</span></span>

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a><span data-ttu-id="88679-168">Ler o fluxo das duas instâncias do hub de eventos</span><span class="sxs-lookup"><span data-stu-id="88679-168">Reading the stream from the two event hub instances</span></span>

<span data-ttu-id="88679-169">A lógica de processamento de dados usa [o fluxo estruturado do Spark](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) para ler as duas instâncias do hub de eventos do Azure:</span><span class="sxs-lookup"><span data-stu-id="88679-169">The data processing logic uses [Spark structured streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) to read from the two Azure event hub instances:</span></span>

```scala
val rideEventHubOptions = EventHubsConf(rideEventHubConnectionString)
      .setConsumerGroup(conf.taxiRideConsumerGroup())
      .setStartingPosition(EventPosition.fromStartOfStream)
    val rideEvents = spark.readStream
      .format("eventhubs")
      .options(rideEventHubOptions.toMap)
      .load

    val fareEventHubOptions = EventHubsConf(fareEventHubConnectionString)
      .setConsumerGroup(conf.taxiFareConsumerGroup())
      .setStartingPosition(EventPosition.fromStartOfStream)
    val fareEvents = spark.readStream
      .format("eventhubs")
      .options(fareEventHubOptions.toMap)
      .load
```

### <a name="enriching-the-data-with-the-neighborhood-information"></a><span data-ttu-id="88679-170">Como enriquecer os dados com informações sobre a área</span><span class="sxs-lookup"><span data-stu-id="88679-170">Enriching the data with the neighborhood information</span></span>

<span data-ttu-id="88679-171">Os dados da corrida incluem as coordenadas de latitude e longitude dos locais de coleta e entrega.</span><span class="sxs-lookup"><span data-stu-id="88679-171">The ride data includes the latitude and longitude coordinates of the pick up and drop off locations.</span></span> <span data-ttu-id="88679-172">Embora essas coordenadas sejam úteis, elas não são facilmente consumidas na análise.</span><span class="sxs-lookup"><span data-stu-id="88679-172">While these coordinates are useful, they are not easily consumed for analysis.</span></span> <span data-ttu-id="88679-173">Portanto, esses dados são enriquecidos com dados de área que são lidos de um [shapefile](https://en.wikipedia.org/wiki/Shapefile).</span><span class="sxs-lookup"><span data-stu-id="88679-173">Therefore, this data is enriched with neighborhood data that is read from a [shapefile](https://en.wikipedia.org/wiki/Shapefile).</span></span>

<span data-ttu-id="88679-174">O formato shapefile é binário e não é fácil analisá-lo, mas a biblioteca [GeoTools](http://geotools.org/) fornece ferramentas para dados geoespaciais que usam o formato shapefile.</span><span class="sxs-lookup"><span data-stu-id="88679-174">The shapefile format is binary and not easily parsed, but the [GeoTools](http://geotools.org/) library provides tools for geospatial data that use the shapefile format.</span></span> <span data-ttu-id="88679-175">Essa biblioteca é usada na classe **com.microsoft.pnp.GeoFinder** para determinar o nome da área com base nas coordenadas de coleta e entrega.</span><span class="sxs-lookup"><span data-stu-id="88679-175">This library is used in the **com.microsoft.pnp.GeoFinder** class to determine the neighborhood name based on the pick up and drop off coordinates.</span></span>

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a><span data-ttu-id="88679-176">Adicionar os dados de corrida e de tarifa</span><span class="sxs-lookup"><span data-stu-id="88679-176">Joining the ride and fare data</span></span>

<span data-ttu-id="88679-177">Primeiro, os dados da corrida e da tarifa são transformados:</span><span class="sxs-lookup"><span data-stu-id="88679-177">First the ride and fare data is transformed:</span></span>

```scala
    val rides = transformedRides
      .filter(r => {
        if (r.isNullAt(r.fieldIndex("errorMessage"))) {
          true
        }
        else {
          malformedRides.add(1)
          false
        }
      })
      .select(
        $"ride.*",
        to_neighborhood($"ride.pickupLon", $"ride.pickupLat")
          .as("pickupNeighborhood"),
        to_neighborhood($"ride.dropoffLon", $"ride.dropoffLat")
          .as("dropoffNeighborhood")
      )
      .withWatermark("pickupTime", conf.taxiRideWatermarkInterval())

    val fares = transformedFares
      .filter(r => {
        if (r.isNullAt(r.fieldIndex("errorMessage"))) {
          true
        }
        else {
          malformedFares.add(1)
          false
        }
      })
      .select(
        $"fare.*",
        $"pickupTime"
      )
      .withWatermark("pickupTime", conf.taxiFareWatermarkInterval())
```

<span data-ttu-id="88679-178">E, em seguida, os dados da corrida são associados aos dados da tarifa:</span><span class="sxs-lookup"><span data-stu-id="88679-178">And then the ride data is joined with the fare data:</span></span>

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a><span data-ttu-id="88679-179">Como processar dados e inseri-los no Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="88679-179">Processing the data and inserting into Cosmos DB</span></span>

<span data-ttu-id="88679-180">O valor da tarifa média para cada ambiente é calculado para um determinado intervalo de tempo:</span><span class="sxs-lookup"><span data-stu-id="88679-180">The average fare amount for each neighborhood is calculated for a given time interval:</span></span>

```scala
val maxAvgFarePerNeighborhood = mergedTaxiTrip.selectExpr("medallion", "hackLicense", "vendorId", "pickupTime", "rateCode", "storeAndForwardFlag", "dropoffTime", "passengerCount", "tripTimeInSeconds", "tripDistanceInMiles", "pickupLon", "pickupLat", "dropoffLon", "dropoffLat", "paymentType", "fareAmount", "surcharge", "mtaTax", "tipAmount", "tollsAmount", "totalAmount", "pickupNeighborhood", "dropoffNeighborhood")
      .groupBy(window($"pickupTime", conf.windowInterval()), $"pickupNeighborhood")
      .agg(
        count("*").as("rideCount"),
        sum($"fareAmount").as("totalFareAmount"),
        sum($"tipAmount").as("totalTipAmount")
      )
      .select($"window.start", $"window.end", $"pickupNeighborhood", $"rideCount", $"totalFareAmount", $"totalTipAmount")
```

<span data-ttu-id="88679-181">Portanto, qual é inserido no Cosmos DB:</span><span class="sxs-lookup"><span data-stu-id="88679-181">Which is then inserted into Cosmos DB:</span></span>

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a><span data-ttu-id="88679-182">Considerações de segurança</span><span class="sxs-lookup"><span data-stu-id="88679-182">Security considerations</span></span>

<span data-ttu-id="88679-183">O acesso ao espaço de trabalho do Azure Database é controlado com o [console do administrador](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span><span class="sxs-lookup"><span data-stu-id="88679-183">Access to the Azure Database workspace is controlled using the [administrator console](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span></span> <span data-ttu-id="88679-184">O console do administrador inclui recursos para adicionar usuários, gerenciar permissões de usuários e configurar o logon único.</span><span class="sxs-lookup"><span data-stu-id="88679-184">The administrator console includes functionality to add users, manage user permissions, and set up single sign-on.</span></span> <span data-ttu-id="88679-185">O controle de acesso para espaços de trabalho, clusters, trabalhos e tabelas também pode ser definido pelo console do administrador.</span><span class="sxs-lookup"><span data-stu-id="88679-185">Access control for workspaces, clusters, jobs, and tables can also be set through the administrator console.</span></span>

### <a name="managing-secrets"></a><span data-ttu-id="88679-186">Gerenciamento de segredos</span><span class="sxs-lookup"><span data-stu-id="88679-186">Managing secrets</span></span>

<span data-ttu-id="88679-187">As Azure Databricks inclui um [armazenamento secreto](https://docs.azuredatabricks.net/user-guide/secrets/index.html) que é usado para armazenar segredos, incluindo cadeias de conexão, chaves de acesso, nomes de usuário e senhas.</span><span class="sxs-lookup"><span data-stu-id="88679-187">Azure Databricks includes a [secret store](https://docs.azuredatabricks.net/user-guide/secrets/index.html) that is used to store secrets, including connection strings, access keys, user names, and passwords.</span></span> <span data-ttu-id="88679-188">Os segredos dentro do armazenamento secreto do Azure Databricks são particionados por **escopos**:</span><span class="sxs-lookup"><span data-stu-id="88679-188">Secrets within the Azure Databricks secret store are partitioned by **scopes**:</span></span>

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

<span data-ttu-id="88679-189">Os segredos são adicionados ao nível do escopo:</span><span class="sxs-lookup"><span data-stu-id="88679-189">Secrets are added at the scope level:</span></span>

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> <span data-ttu-id="88679-190">Um escopo suportado pelo Azure Key Vault pode ser usado no lugar do escopo nativo do Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="88679-190">An Azure Key Vault-backed scope can be used instead of the native Azure Databricks scope.</span></span> <span data-ttu-id="88679-191">Para obter mais informações, confira [Escopos com suporte do Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span><span class="sxs-lookup"><span data-stu-id="88679-191">To learn more, see [Azure Key Vault-backed scopes](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span></span>

<span data-ttu-id="88679-192">No código, os segredos são acessados pelos [utilitários segredos](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities) do Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="88679-192">In code, secrets are accessed via the Azure Databricks [secrets utilities](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="88679-193">Considerações de monitoramento</span><span class="sxs-lookup"><span data-stu-id="88679-193">Monitoring considerations</span></span>

<span data-ttu-id="88679-194">O Azure Databricks é baseado no Apache Spark e ambos usam [log4j](https://logging.apache.org/log4j/2.x/) como a biblioteca padrão para registros em log.</span><span class="sxs-lookup"><span data-stu-id="88679-194">Azure Databricks is based on Apache Spark, and both use [log4j](https://logging.apache.org/log4j/2.x/) as the standard library for logging.</span></span> <span data-ttu-id="88679-195">Além do registro em log padrão fornecido pelo Apache Spark, essa arquitetura de referência envia logs e métricas para o [Azure Log Analytics](/azure/log-analytics/).</span><span class="sxs-lookup"><span data-stu-id="88679-195">In addition to the default logging provided by Apache Spark, this reference architecture sends logs and metrics to [Azure Log Analytics](/azure/log-analytics/).</span></span>

<span data-ttu-id="88679-196">A classe **com.microsoft.pnp.TaxiCabReader** configura o sistema de registro em log do Apache Spark para enviar seus logs para o Azure Log Analytics usando os valores no arquivo **log4j.properties**.</span><span class="sxs-lookup"><span data-stu-id="88679-196">The **com.microsoft.pnp.TaxiCabReader** class configures the Apache Spark logging system to send its logs to Azure Log Analytics using the values in the **log4j.properties** file.</span></span> <span data-ttu-id="88679-197">As mensagens do agente do Apache Spark são cadeia de caracteres mas, o Azure Log Analytics requer que as mensagens de log sejam formatadas como JSON.</span><span class="sxs-lookup"><span data-stu-id="88679-197">While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON.</span></span> <span data-ttu-id="88679-198">A classe **com.microsoft.pnp.log4j.LogAnalyticsAppender** transforma essas mensagens em JSON:</span><span class="sxs-lookup"><span data-stu-id="88679-198">The **com.microsoft.pnp.log4j.LogAnalyticsAppender** class transforms these messages to JSON:</span></span>

```scala

    @Override
    protected void append(LoggingEvent loggingEvent) {
        if (this.layout == null) {
            this.setLayout(new JSONLayout());
        }

        String json = this.getLayout().format(loggingEvent);
        try {
            this.client.send(json, this.logType);
        } catch(IOException ioe) {
            LogLog.warn("Error sending LoggingEvent to Log Analytics", ioe);
        }
    }

```

<span data-ttu-id="88679-199">Como a classe **com.microsoft.pnp.TaxiCabReader** processa as mensagens de corrida e tarifa, é provável que uma delas esteja malformada e, portanto, inválida.</span><span class="sxs-lookup"><span data-stu-id="88679-199">As the **com.microsoft.pnp.TaxiCabReader** class processes ride and fare messages, it's possible that either one may be malformed and therefore not valid.</span></span> <span data-ttu-id="88679-200">Em um ambiente de produção, é importante analisar essas mensagens malformadas para identificar problemas com as fontes de dados para que eles possam ser corrigidos rapidamente para evitar a perda de dados.</span><span class="sxs-lookup"><span data-stu-id="88679-200">In a production environment, it's important to analyze these malformed messages to identify a problem with the data sources so it can be fixed quickly to prevent data loss.</span></span> <span data-ttu-id="88679-201">A classe **com.microsoft.pnp.TaxiCabReader** registra um Apache Spark Accumulator que controla o número de registros de tarifas e corridas malformadas:</span><span class="sxs-lookup"><span data-stu-id="88679-201">The **com.microsoft.pnp.TaxiCabReader** class registers an Apache Spark Accumulator that keeps track of the number of malformed fare and ride records:</span></span>

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

<span data-ttu-id="88679-202">O Apache Spark usa a biblioteca Dropwizard para enviar métricas. Alguns campos nativos de métricas do Dropwizard são incompatíveis com o Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="88679-202">Apache Spark uses the Dropwizard library to send metrics, and some of the native Dropwizard metrics fields are incompatible with Azure Log Analytics.</span></span> <span data-ttu-id="88679-203">Portanto, essa arquitetura de referência inclui um coletor e um repórter personalizado do Dropwizard.</span><span class="sxs-lookup"><span data-stu-id="88679-203">Therefore, this reference architecture includes a custom Dropwizard sink and reporter.</span></span> <span data-ttu-id="88679-204">Ele formata as métricas no formato esperado pelo Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="88679-204">It formats the metrics in the format expected by Azure Log Analytics.</span></span> <span data-ttu-id="88679-205">Quando o Apache Spark relata as métricas, as métricas personalizadas dos dados malformados de tarifa e corrida também são enviadas.</span><span class="sxs-lookup"><span data-stu-id="88679-205">When Apache Spark reports metrics, the custom metrics for the malformed ride and fare data are also sent.</span></span>

<span data-ttu-id="88679-206">A última métrica a ser registrada no espaço de trabalho do Azure Log Analytics é o progresso cumulativo do progresso do trabalho do Fluxo Estruturado do Spark.</span><span class="sxs-lookup"><span data-stu-id="88679-206">The last metric to be logged to the Azure Log Analytics workspace is the cumulative progress of the Spark Structured Streaming job progress.</span></span> <span data-ttu-id="88679-207">Isso é feito usando um ouvinte personalizado do StreamingQuery implementado na classe **com.microsoft.pnp.StreamingMetricsListener**.</span><span class="sxs-lookup"><span data-stu-id="88679-207">This is done using a custom StreamingQuery listener implemented in the **com.microsoft.pnp.StreamingMetricsListener** class.</span></span> <span data-ttu-id="88679-208">Esta classe é registrada na sessão do Apache Spark quando o trabalho é executado:</span><span class="sxs-lookup"><span data-stu-id="88679-208">This class is registered to the Apache Spark Session when the job runs:</span></span>

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

<span data-ttu-id="88679-209">Os métodos no StreamingMetricsListener são chamados pelo tempo de execução do Apache Spark sempre que um evento de fluxo estruturado ocorre, enviando mensagens de log e métricas para o espaço de trabalho do Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="88679-209">The methods in the StreamingMetricsListener are called by the Apache Spark runtime whenever a structured steaming event occurs, sending log messages and metrics to the Azure Log Analytics workspace.</span></span> <span data-ttu-id="88679-210">Você pode usar as seguintes consultas em seu espaço de trabalho para monitorar o aplicativo:</span><span class="sxs-lookup"><span data-stu-id="88679-210">You can use the following queries in your workspace to monitor the application:</span></span>

### <a name="latency-and-throughput-for-streaming-queries"></a><span data-ttu-id="88679-211">Latência e taxa de transferência de consultas de streaming</span><span class="sxs-lookup"><span data-stu-id="88679-211">Latency and throughput for streaming queries</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a><span data-ttu-id="88679-212">Exceções registradas durante a execução de consulta de streaming</span><span class="sxs-lookup"><span data-stu-id="88679-212">Exceptions logged during stream query execution</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a><span data-ttu-id="88679-213">Acumulação de tarifas malformadas e dados da corrida</span><span class="sxs-lookup"><span data-stu-id="88679-213">Accumulation of malformed fare and ride data</span></span>

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "metrics.malformedrides"

SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "metrics.malformedfares"
```

### <a name="job-execution-to-trace-resiliency"></a><span data-ttu-id="88679-214">Execução do trabalho para controlar a resiliência</span><span class="sxs-lookup"><span data-stu-id="88679-214">Job execution to trace resiliency</span></span>

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

<span data-ttu-id="88679-215">Para obter mais informações, consulte [de monitoramento do Azure Databricks](../../databricks-monitoring/index.md).</span><span class="sxs-lookup"><span data-stu-id="88679-215">For more information, see [Monitoring Azure Databricks](../../databricks-monitoring/index.md).</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="88679-216">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="88679-216">Deploy the solution</span></span>

<span data-ttu-id="88679-217">Para a implantação e execução da implementação de referência, siga as etapas em [Leia-me do GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics).</span><span class="sxs-lookup"><span data-stu-id="88679-217">To the deploy and run the reference implementation, follow the steps in the [GitHub readme](https://github.com/mspnp/azure-databricks-streaming-analytics).</span></span>
