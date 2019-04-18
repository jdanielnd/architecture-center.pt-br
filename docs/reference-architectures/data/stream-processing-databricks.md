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
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/18/2019
ms.locfileid: "59740452"
---
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a>Criar um pipeline de processamento de fluxo com o Azure Databricks

Essa arquitetura de referência mostra um pipeline de [processamento de fluxo](/azure/architecture/data-guide/big-data/real-time-processing) de ponta a ponta. Esse tipo de pipeline tem quatro estágios: ingerir, processar, armazenar e analisar e gerar relatórios. Para essa arquitetura de referência, o pipeline ingere dados de duas origens, realiza uma junção em registros relacionados de cada fluxo, enriquece o resultado e calcula uma média em tempo real. Os resultados são armazenados para análise posterior. [**Implantar esta solução**](#deploy-the-solution).

![Arquitetura de referência para processamento de fluxo com o Azure Databricks](./images/stream-processing-databricks.png)

**Cenário**: uma empresa de táxi coleta dados sobre cada viagem. Para esse cenário, assumimos que há dois dispositivos separados enviando dados. O táxi tem um medidor que envia informações sobre cada corrida &mdash; duração, distância e locais de embarque e desembarque de passageiros. Um dispositivo separado aceita pagamentos de clientes e envia dados sobre tarifas. Para identificar as tendências dos passageiros, a empresa de táxi deseja calcular a média de gorjeta por quilômetro percorrido em tempo real, para cada área.

## <a name="architecture"></a>Arquitetura

A arquitetura consiste nos componentes a seguir.

**Fontes de dados**. Nessa arquitetura há duas fontes de dados que geram fluxos de dados em tempo real. O primeiro fluxo contém informações da corrida, e o segundo contém informações da tarifa. A arquitetura de referência inclui um gerador de dados simulados que faz a leitura de um conjunto de arquivos estáticos e envia os dados para os Hubs de Eventos. Em um aplicativo real, as fontes de dados seriam dispositivos instalados nos táxis.

**Hubs de Eventos do Azure**. [Hubs de Eventos](/azure/event-hubs/) são um serviço de ingestão de eventos. Essa arquitetura usa duas instâncias de hub de eventos, uma para cada fonte de dados. Cada fonte de dados envia um fluxo de dados para o hub de eventos associado.

**Azure Databricks**. O [Databricks](/azure/azure-databricks/) é uma plataforma de análise baseada no Apache Spark otimizada para a plataforma de Serviços de Nuvem do Microsoft Azure. O Databricks é usado para correlacionar os dados de corrida e de tarifa do táxi e também para enriquecer os dados correlacionados com dados armazenados sobre áreas no sistema de arquivos do Databricks.

**Cosmos DB**. A saída do trabalho do Azure Databricks é uma série de registros que são gravados no [Cosmos DB](/azure/cosmos-db/) usando a API do Cassandra. A API do Cassandra é usada porque oferece suporte à modelagem de dados de série temporal.

**Azure Log Analytics**. Os dados de log do aplicativo coletados pelo [Monitor do Azure](/azure/monitoring-and-diagnostics/) são armazenados em um [espaço de trabalho do Log Analytics](/azure/log-analytics). As consultas do Log Analytics podem ser usadas para analisar e visualizar métricas e inspecionar mensagens de log para identificar problemas no aplicativo.

## <a name="data-ingestion"></a>Ingestão de dados

<!-- markdownlint-disable MD033 -->

Para simular uma fonte de dados, essa arquitetura de referência usa o conjunto de dados dos [Dados de táxi de Nova York](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797)<sup>[[1]](#note1)</sup>. Esse conjunto de dados contém dados sobre viagens de táxi em Nova York durante um período de 4 anos (2010 &ndash; 2013). Ele contém dois tipos de registro: Dados da corrida e da tarifa. Os dados de corrida incluem a duração da viagem, a distância da viagem e os locais de embarque e desembarque de passageiros. Os dados de tarifa incluem a tarifa, impostos e quantias das gorjetas. Campos comuns em ambos os tipos de registro incluem o número da licença, carteira de habilitação e ID do fornecedor. Juntos, esses três campos fazem a identificação exclusiva de um táxi e um motorista. Os dados são armazenados no formato CSV.

> [1] <span id="note1">Donovan, Brian; Work, Dan (2016): Dados de corridas de táxi na Cidade de Nova York (2010-2013). Universidade de Illinois em Urbana-Champaign. <https://doi.org/10.13012/J8PN93H8>

<!-- markdownlint-enable MD033 -->

O gerador de dados é um aplicativo .NET Core que lê os registros e os envia para os Hubs de Eventos do Azure. O gerador envia os dados de corrida em formato JSON e os dados de tarifa em formato CSV.

Os Hubs de Eventos usam [partições](/azure/event-hubs/event-hubs-features#partitions) para segmentar os dados. As partições permitem que um consumidor leia cada partição em paralelo. Ao enviar dados para os Hubs de Eventos, é possível especificar a chave de partição explicitamente. Caso contrário, os registros são atribuídos a partições no estilo round robin.

Nesse cenário, os dados de corrida e de tarifa devem ter a mesma ID de partição para um determinado táxi. Isso permite que o Databricks aplique um grau de paralelismo ao correlacionar os dois fluxos. Um registro na partição *n* dos dados de corrida corresponderá a um registro na partição *n* dos dados de tarifa.

![Diagrama de processamento de fluxo com o Azure Databricks e os Hubs de Eventos](./images/stream-processing-databricks-eh.png)

No gerador de dados, o modelo de dados comum para ambos os tipos de registro têm uma propriedade `PartitionKey` que é a concatenação de `Medallion`, `HackLicense` e `VendorId`.

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

Essa propriedade é usada para fornecer uma chave de partição explícita ao enviar para Hubs de Eventos:

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a>Hubs de Eventos

A capacidade da taxa de transferência dos Hubs de Eventos é medida em [unidades de produtividade](/azure/event-hubs/event-hubs-features#throughput-units). É possível fazer o dimensionamento automático de um hub de eventos ao permitir a [inflação automática](/azure/event-hubs/event-hubs-auto-inflate), o que dimensiona automaticamente as unidades de produtividade com base no tráfego até um número máximo configurado.

## <a name="stream-processing"></a>Processamento de fluxo

No Azure Databricks, o processamento de dados é executado por um trabalho. O trabalho é atribuído a um cluster e é executado nele. O trabalho pode ser o código personalizado escrito em Java ou um [bloco de notas](https://docs.databricks.com/user-guide/notebooks/index.html) do Spark.

Nessa arquitetura de referência, o trabalho é um arquivo morto de Java com classes escritas em Java e Scala. Ao especificar o arquivo morto de Java para um trabalho do Databricks, a classe será especificada para ser executada pelo cluster do Databricks. Aqui, o método **main** da classe **com.microsoft.pnp.TaxiCabReader** contém a lógica de processamento de dados.

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a>Ler o fluxo das duas instâncias do hub de eventos

A lógica de processamento de dados usa [o fluxo estruturado do Spark](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) para ler as duas instâncias do hub de eventos do Azure:

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

### <a name="enriching-the-data-with-the-neighborhood-information"></a>Como enriquecer os dados com informações sobre a área

Os dados da corrida incluem as coordenadas de latitude e longitude dos locais de coleta e entrega. Embora essas coordenadas sejam úteis, elas não são facilmente consumidas na análise. Portanto, esses dados são enriquecidos com dados de área que são lidos de um [shapefile](https://en.wikipedia.org/wiki/Shapefile).

O formato shapefile é binário e não é fácil analisá-lo, mas a biblioteca [GeoTools](http://geotools.org/) fornece ferramentas para dados geoespaciais que usam o formato shapefile. Essa biblioteca é usada na classe **com.microsoft.pnp.GeoFinder** para determinar o nome da área com base nas coordenadas de coleta e entrega.

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a>Adicionar os dados de corrida e de tarifa

Primeiro, os dados da corrida e da tarifa são transformados:

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

E, em seguida, os dados da corrida são associados aos dados da tarifa:

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a>Como processar dados e inseri-los no Cosmos DB

O valor da tarifa média para cada ambiente é calculado para um determinado intervalo de tempo:

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

Portanto, qual é inserido no Cosmos DB:

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a>Considerações de segurança

O acesso ao espaço de trabalho do Azure Database é controlado com o [console do administrador](https://docs.databricks.com/administration-guide/admin-settings/index.html). O console do administrador inclui recursos para adicionar usuários, gerenciar permissões de usuários e configurar o logon único. O controle de acesso para espaços de trabalho, clusters, trabalhos e tabelas também pode ser definido pelo console do administrador.

### <a name="managing-secrets"></a>Gerenciamento de segredos

As Azure Databricks inclui um [armazenamento secreto](https://docs.azuredatabricks.net/user-guide/secrets/index.html) que é usado para armazenar segredos, incluindo cadeias de conexão, chaves de acesso, nomes de usuário e senhas. Os segredos dentro do armazenamento secreto do Azure Databricks são particionados por **escopos**:

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

Os segredos são adicionados ao nível do escopo:

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> Um escopo suportado pelo Azure Key Vault pode ser usado no lugar do escopo nativo do Azure Databricks. Para obter mais informações, confira [Escopos com suporte do Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).

No código, os segredos são acessados pelos [utilitários segredos](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities) do Azure Databricks.

## <a name="monitoring-considerations"></a>Considerações de monitoramento

O Azure Databricks é baseado no Apache Spark e ambos usam [log4j](https://logging.apache.org/log4j/2.x/) como a biblioteca padrão para registros em log. Além do registro em log padrão fornecido pelo Apache Spark, essa arquitetura de referência envia logs e métricas para o [Azure Log Analytics](/azure/log-analytics/).

A classe **com.microsoft.pnp.TaxiCabReader** configura o sistema de registro em log do Apache Spark para enviar seus logs para o Azure Log Analytics usando os valores no arquivo **log4j.properties**. As mensagens do agente do Apache Spark são cadeia de caracteres mas, o Azure Log Analytics requer que as mensagens de log sejam formatadas como JSON. A classe **com.microsoft.pnp.log4j.LogAnalyticsAppender** transforma essas mensagens em JSON:

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

Como a classe **com.microsoft.pnp.TaxiCabReader** processa as mensagens de corrida e tarifa, é provável que uma delas esteja malformada e, portanto, inválida. Em um ambiente de produção, é importante analisar essas mensagens malformadas para identificar problemas com as fontes de dados para que eles possam ser corrigidos rapidamente para evitar a perda de dados. A classe **com.microsoft.pnp.TaxiCabReader** registra um Apache Spark Accumulator que controla o número de registros de tarifas e corridas malformadas:

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

O Apache Spark usa a biblioteca Dropwizard para enviar métricas. Alguns campos nativos de métricas do Dropwizard são incompatíveis com o Azure Log Analytics. Portanto, essa arquitetura de referência inclui um coletor e um repórter personalizado do Dropwizard. Ele formata as métricas no formato esperado pelo Azure Log Analytics. Quando o Apache Spark relata as métricas, as métricas personalizadas dos dados malformados de tarifa e corrida também são enviadas.

A última métrica a ser registrada no espaço de trabalho do Azure Log Analytics é o progresso cumulativo do progresso do trabalho do Fluxo Estruturado do Spark. Isso é feito usando um ouvinte personalizado do StreamingQuery implementado na classe **com.microsoft.pnp.StreamingMetricsListener**. Esta classe é registrada na sessão do Apache Spark quando o trabalho é executado:

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

Os métodos no StreamingMetricsListener são chamados pelo tempo de execução do Apache Spark sempre que um evento de fluxo estruturado ocorre, enviando mensagens de log e métricas para o espaço de trabalho do Azure Log Analytics. Você pode usar as seguintes consultas em seu espaço de trabalho para monitorar o aplicativo:

### <a name="latency-and-throughput-for-streaming-queries"></a>Latência e taxa de transferência de consultas de streaming

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a>Exceções registradas durante a execução de consulta de streaming

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a>Acumulação de tarifas malformadas e dados da corrida

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

### <a name="job-execution-to-trace-resiliency"></a>Execução do trabalho para controlar a resiliência

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

Para obter mais informações, consulte [de monitoramento do Azure Databricks](../../databricks-monitoring/index.md).

## <a name="deploy-the-solution"></a>Implantar a solução

Para a implantação e execução da implementação de referência, siga as etapas em [Leia-me do GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics).
