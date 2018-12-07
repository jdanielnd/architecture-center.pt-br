---
title: Processamento de fluxo com o Azure Databricks
description: Criar um pipeline de processamento de fluxo de ponta a ponta no Azure usando o Azure Databricks
author: petertaylor9999
ms.date: 11/30/2018
ms.openlocfilehash: 0640e900c212d2b75cc9cdd5bec3a4f7c050490d
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902826"
---
# <a name="stream-processing-with-azure-databricks"></a>Processamento de fluxo com o Azure Databricks

Essa arquitetura de referência mostra um pipeline de [processamento de fluxo](/azure/architecture/data-guide/big-data/real-time-processing) de ponta a ponta. Esse tipo de pipeline tem quatro estágios: ingerir, processar, armazenar e analisar e gerar relatórios. Para essa arquitetura de referência, o pipeline ingere dados de duas origens, realiza uma junção em registros relacionados de cada fluxo, enriquece o resultado e calcula uma média em tempo real. Os resultados são armazenados para análise posterior. [**Implante essa solução**.](#deploy-the-solution)

![](./images/stream-processing-databricks.png)

**Cenário**: uma empresa de táxi coleta dados sobre cada viagem. Para esse cenário, assumimos que há dois dispositivos separados enviando dados. O táxi tem um medidor que envia informações sobre cada corrida &mdash; duração, distância e locais de embarque e desembarque de passageiros. Um dispositivo separado aceita pagamentos de clientes e envia dados sobre tarifas. Para identificar as tendências dos passageiros, a empresa de táxi deseja calcular a média de gorjeta por quilômetro percorrido em tempo real, para cada área.

## <a name="architecture"></a>Arquitetura

A arquitetura consiste nos componentes a seguir.

**Fontes de dados**. Nessa arquitetura há duas fontes de dados que geram fluxos de dados em tempo real. O primeiro fluxo contém informações da corrida, e o segundo contém informações da tarifa. A arquitetura de referência inclui um gerador de dados simulados que faz a leitura de um conjunto de arquivos estáticos e envia os dados para os Hubs de Eventos. Em um aplicativo real, as fontes de dados seriam dispositivos instalados nos táxis.

**Hubs de Eventos do Azure**. [Hubs de Eventos](/azure/event-hubs/) são um serviço de ingestão de eventos. Essa arquitetura usa duas instâncias de hub de eventos, uma para cada fonte de dados. Cada fonte de dados envia um fluxo de dados para o hub de eventos associado.

**Azure Databricks**. O [Databricks](/azure/azure-databricks/) é uma plataforma de análise baseada no Apache Spark otimizada para a plataforma de Serviços de Nuvem do Microsoft Azure. O Databricks é usado para correlacionar os dados de corrida e de tarifa do táxi e também para enriquecer os dados correlacionados com dados armazenados sobre áreas no sistema de arquivos do Databricks.

**Cosmos DB**. A saída do trabalho do Azure Databricks é uma série de registros que são gravados no [Cosmos DB](/azure/cosmos-db/) usando a API do Cassandra. A API do Cassandra é usada porque oferece suporte à modelagem de dados de série temporal.

**Azure Log Analytics**. Os dados de log do aplicativo coletados pelo [Monitor do Azure](/azure/monitoring-and-diagnostics/) são armazenados em um [espaço de trabalho do Log Analytics](/azure/log-analytics). As consultas do Log Analytics podem ser usadas para analisar e visualizar métricas e inspecionar mensagens de log para identificar problemas no aplicativo.

## <a name="data-ingestion"></a>Ingestão de dados

Para simular uma fonte de dados, essa arquitetura de referência usa o conjunto de dados dos [Dados de táxi de Nova York](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797)<sup>[[1]](#note1)</sup>. Esse conjunto de dados contém dados sobre viagens de táxi em Nova York durante um período de 4 anos (2010 &ndash; 2013). Ele contém dois tipos de registro: dados de corrida e dados de tarifa. Os dados de corrida incluem a duração da viagem, a distância da viagem e os locais de embarque e desembarque de passageiros. Os dados de tarifa incluem a tarifa, impostos e quantias das gorjetas. Campos comuns em ambos os tipos de registro incluem o número da licença, carteira de habilitação e ID do fornecedor. Juntos, esses três campos fazem a identificação exclusiva de um táxi e um motorista. Os dados são armazenados no formato CSV. 

O gerador de dados é um aplicativo .NET Core que lê os registros e os envia para os Hubs de Eventos do Azure. O gerador envia os dados de corrida em formato JSON e os dados de tarifa em formato CSV. 

Os Hubs de Eventos usam [partições](/azure/event-hubs/event-hubs-features#partitions) para segmentar os dados. As partições permitem que um consumidor leia cada partição em paralelo. Ao enviar dados para os Hubs de Eventos, é possível especificar a chave de partição explicitamente. Caso contrário, os registros são atribuídos a partições no estilo round robin. 

Nesse cenário, os dados de corrida e de tarifa devem ter a mesma ID de partição para um determinado táxi. Isso permite que o Databricks aplique um grau de paralelismo ao correlacionar os dois fluxos. Um registro na partição *n* dos dados de corrida corresponderá a um registro na partição *n* dos dados de tarifa.

![](./images/stream-processing-databricks-eh.png)

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

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para essa arquitetura de referência está disponível no [GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics). 

### <a name="prerequisites"></a>Pré-requisitos

1. Clone, crie um fork ou baixe o repositório do GitHub [Processamento por fluxo com o Azure Databricks](https://github.com/mspnp/azure-databricks-streaming-analytics).

2. Instale o [Docker](https://www.docker.com/) para executar o gerador de dados.

3. Instale a [CLI 2.0 do Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

4. Instale a [CLI do Databricks](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).

5. Em um prompt de comando, prompt do bash ou prompt do PowerShell, entre na sua conta do Azure da seguinte maneira:
    ```shell
    az login
    ```
6. Instale um Java IDE, com os seguintes recursos:
    - JDK 1.8
    - Scala SDK 2.11
    - Maven 3.5.4

### <a name="download-the-new-york-city-taxi-and-neighborhood-data-files"></a>Fazer o download dos arquivos de dados da área e do táxi de Nova York

1. Crie um diretório chamado `DataFile` na raiz do repositório Github clonado em seu sistema de arquivos local.

2. Abra um navegador da Web e acesse https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.

3. Clique no botão **Baixar** nesta página para baixar um arquivo zip com todos os dados de táxi desse ano.

4. Extraia o arquivo zip para o diretório `DataFile`.

    > [!NOTE]
    > Esse arquivo zip contém outros arquivos zip. Não extraia os arquivos zip filhos.

    A estrutura do diretório deve ser semelhante à seguinte:

    ```shell
    /DataFile
        /FOIL2013
            trip_data_1.zip
            trip_data_2.zip
            trip_data_3.zip
            ...
    ```

5. Abra um navegador da Web e acesse https://www.zillow.com/howto/api/neighborhood-boundaries.htm. 

6. Clique em **Limites de área de Nova York** para fazer o download do arquivo.

7. Copie o arquivo **ZillowNeighborhoods NY.zip** do diretório **downloads** de seu navegador para o diretório `DataFile`.

### <a name="deploy-the-azure-resources"></a>Implantar os recursos do Azure

1. Em um shell ou prompt de comando do Windows, execute o comando a seguir e siga o prompt de entrada:

    ```bash
    az login
    ```

2. Navegue até a pasta chamada `azure` no repositório do GitHub:

    ```bash
    cd azure
    ```

3. Execute os seguintes comandos para implantar os recursos do Azure:

    ```bash
    export resourceGroup='[Resource group name]'
    export resourceLocation='[Region]'
    export eventHubNamespace='[Event Hubs namespace name]'
    export databricksWorkspaceName='[Azure Databricks workspace name]'
    export cosmosDatabaseAccount='[Cosmos DB database name]'
    export logAnalyticsWorkspaceName='[Log Analytics workspace name]'
    export logAnalyticsWorkspaceRegion='[Log Analytics region]'

    # Create a resource group
    az group create --name $resourceGroup --location $resourceLocation

    # Deploy resources
    az group deployment create --resource-group $resourceGroup \
        --template-file deployresources.json --parameters \
        eventHubNamespace=$eventHubNamespace \
        databricksWorkspaceName=$databricksWorkspaceName \
        cosmosDatabaseAccount=$cosmosDatabaseAccount \
        logAnalyticsWorkspaceName=$logAnalyticsWorkspaceName \
        logAnalyticsWorkspaceRegion=$logAnalyticsWorkspaceRegion
    ```

4. A saída da implantação é gravada no console após a conclusão. Pesquise a saída JSON a seguir:

```JSON
"outputs": {
        "cosmosDb": {
          "type": "Object",
          "value": {
            "hostName": <value>,
            "secret": <value>,
            "username": <value>
          }
        },
        "eventHubs": {
          "type": "Object",
          "value": {
            "taxi-fare-eh": <value>,
            "taxi-ride-eh": <value>
          }
        },
        "logAnalytics": {
          "type": "Object",
          "value": {
            "secret": <value>,
            "workspaceId": <value>
          }
        }
},
```
Esses valores são os segredos que serão adicionados aos segredos do Databricks nas próximas seções. Mantenha-os seguros até você adicioná-los às seções.

### <a name="add-a-cassandra-table-to-the-cosmos-db-account"></a>Adicionar uma tabela do Cassandra à conta do Cosmos DB

1. No Portal do Azure, navegue até o grupo de recursos criado na seção **implantar recursos do Azure**. Clique na **conta do Azure Cosmos DB**. Crie uma tabela com a API do Cassandra.

2. Na folha **visão geral**, clique em **adicionar tabela**.

3. Quando a folha **adicionar tabela** abrir, insira `newyorktaxi` na caixa de texto **nome do Keyspace**. 

4. Na seção **insira o comando CQL para criar a tabela**, digite `neighborhoodstats` na caixa de texto ao lado de `newyorktaxi`.

5. Na caixa de texto que aparece, insira:
```shell
(neighborhood text, window_end timestamp, number_of_rides bigint,total_fare_amount double, primary key(neighborhood, window_end))
```
6. Na caixa de texto **Taxa de transferência (1,000 - 1,000,000 RU/s)**, insira o valor `4000`.

7. Clique em **OK**.

### <a name="add-the-databricks-secrets-using-the-databricks-cli"></a>Adicionar os segredos do Databricks usando a CLI do Databricks

Primeiro, insira os segredos de EventHub:

1. Com a **CLI do Azure Databricks** instalada na etapa 2 dos pré-requisitos, crie o escopo de segredo do Azure Databricks:
    ```shell
    databricks secrets create-scope --scope "azure-databricks-job"
    ```
2. Adicione o segredo do EventHub de corrida de táxi:
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
    ```
    Após a execução, esse comando abre o editor vi. Insira o valor **taxi-ride-eh** da seção de saída **eventHubs** na etapa 4 a seção *implantar recursos do Azure*. Salve e saia do vi.

3. Adicione o segredo do EventHub da tarifa de táxi:
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-fare"
    ```
    Após a execução, esse comando abre o editor vi. Insira o valor **taxi-fare-eh** da seção de saída **eventHubs** na etapa 4 a seção *implantar recursos do Azure*. Salve e saia do vi.

Em seguida, insira os segredos do Cosmos DB:

1. Abra o Portal do Azure e navegue até o grupo de recursos especificado na etapa 3 da seção **implantar recursos do Azure**. Clique na conta do Azure Cosmos DB.

2. Usando a **CLI do Azure Databricks**, adicione o segredo do nome de usuário do Cosmos DB:
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-username"
    ```
Após a execução, esse comando abre o editor vi. Insira o valor **username** da seção de saída **CosmosDb** na etapa 4 a seção *implantar recursos do Azure*. Salve e saia do vi.

3. Em seguida, adicione o segredo da senha do Cosmos DB:
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-password"
    ```

Após a execução, esse comando abre o editor vi. Insira o valor **secret** da seção de saída **CosmosDb** na etapa 4 a seção *implantar recursos do Azure*. Salve e saia do vi.

> [!NOTE]
> Se estiver usando um [escopo secreto com suporte do Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), o escopo deverá ser nomeado como **azure-databricks-job** e os segredos deverão ter exatamente os mesmos nomes que dos segredos acima.

### <a name="add-the-zillow-neighborhoods-data-file-to-the-databricks-file-system"></a>Adicionar o arquivo de dados das Áreas de Zillow ao sistema de arquivos do Databricks

1. Crie um diretório no sistema de arquivos do Databricks:
    ```bash
    dbfs mkdirs dbfs:/azure-databricks-jobs
    ```

2. Navegue até o diretório `DataFile` e insira o seguinte:
    ```bash
    dbfs cp ZillowNeighborhoods-NY.zip dbfs:/azure-databricks-jobs
    ```

### <a name="add-the-azure-log-analytics-workspace-id-and-primary-key-to-configuration-files"></a>Adicionar a ID do espaço de trabalho do Azure Log Analytics e a chave primária para arquivos de configuração

Para essa seção, você pode exigir a ID do espaço de trabalho do Log Analytics e a chave primária. A ID do espaço de trabalho é o valor **workspaceId** da seção de saída **logAnalytics** na etapa 4 da seção *implantar os recursos do Azure*. A chave primária é o **segredo** da seção de saída. 

1. Para configurar o registro em log de log4j, abra `\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties`. Edite os dois valores a seguir:
    ```shell
    log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
    log4j.appender.A1.secret=<Log Analytics primary key>
    ```

2. Para configurar o registro em log personalizado, abra `\azure\azure-databricks-monitoring\scripts\metrics.properties`. Edite os dois valores a seguir:
    ```shell
    *.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
    *.sink.loganalytics.secret=<Log Analytics primary key>
    ```

### <a name="build-the-jar-files-for-the-databricks-job-and-databricks-monitoring"></a>Compilar os arquivos .jar para o trabalho do Databricks e para o monitoramento do Databricks

1. Use o Java IDE para importar o arquivo de projeto do Maven chamado **pom.xml** localizado no diretório raiz. 

2. Execute uma compilação limpa. A saída dessa compilação são arquivos denominados **azure-databricks-trabalho-1.0-snapshot. jar** e **azure-databricks-monitoramento-0.9.jar**. 

### <a name="configure-custom-logging-for-the-databricks-job"></a>Configurar o registro em log personalizado para o trabalho do Databricks

1. Copie o arquivo **azure-databricks-monitoring-0.9.jar** para o sistema de arquivos do Databricks inserindo o seguinte comando na **CLI do Databricks**:
    ```shell
    databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
    ```

2. Copie as propriedades de registro em log personalizado do `\azure\azure-databricks-monitoring\scripts\metrics.properties` para o sistema de arquivos do Databricks inserindo o seguinte comando:
    ```shell
    databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
    ```

3. Embora você ainda não tenha escolhido um nome para o cluster do Databricks, escolha um agora. Você digitará o nome abaixo no caminho do sistema de arquivos do Databricks para seu cluster. Copie o script de inicialização do `\azure\azure-databricks-monitoring\scripts\spark.metrics` para o sistema de arquivos do Databricks inserindo o seguinte comando:
    ```
    databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
    ```

### <a name="create-a-databricks-cluster"></a>Criar um cluster Databricks

1. No espaço de trabalho do Databricks, clique em "Clusters" e em "criar cluster". Insira o nome do cluster que você criou na etapa 3 da seção **configurar o registro em log personalizado para o trabalho do Databricks**. 

2. Escolha um modo de cluster **padrão**.

3. Defina a **versão de tempo de execução do Databricks** como **4.3 (inclui o Apache Spark 2.3.1, Scala 2.11)**

4. Defina a **versão do Python** como **2**.

5. Defina o **Tipo de driver** como **Igual ao de trabalho**

6. Defina **Tipo de trabalho** como **Standard_DS3_v2**.

7. Defina **Trabalhos mín** como **2**.

8. Desmarque a opção **Habilitar o dimensionamento automático**. 

9. Abaixo da caixa de diálogo **Término Automático**, clique em **Scripts Init**. 

10. Insira **dbfs:/databricks/init/<nome-do-cluster>/spark-metrics.sh** para substituir o nome do cluster criado na etapa 1 para <nome-do-cluster>.

11. Clique no botão **Adicionar** .

12. Clique no botão **Criar Cluster**.

### <a name="create-a-databricks-job"></a>Criar um trabalho do Databricks

1. No espaço de trabalho do Databricks, clique em "Trabalhos", "criar trabalho".

2. Insira um nome de trabalho.

3. Clique em "configurar jar", isso abrirá a caixa de diálogo "Carregar JAR para executar".

4. Arraste o arquivo **azure-databricks-job-1.0-SNAPSHOT.jar** que você criou na seção **compilar o .jar para o trabalho do Databricks** para a caixa **Soltar o JAR aqui para carregar**.

5. Insira **com.microsoft.pnp.TaxiCabReader** no campo **Classe Principal**.

6. No campo de argumentos, insira o seguinte:
    ```shell
    -n jar:file:/dbfs/azure-databricks-jobs/ZillowNeighborhoods-NY.zip!/ZillowNeighborhoods-NY.shp --taxi-ride-consumer-group taxi-ride-eh-cg --taxi-fare-consumer-group taxi-fare-eh-cg --window-interval "1 minute" --cassandra-host <Cosmos DB Cassandra host name from above> 
    ``` 

7. Instale as bibliotecas dependentes seguindo estas etapas:
    
    1. Na interface do usuário do Databricks, clique no botão **doméstica**.
    
    2. Na lista suspensa **Usuários**, clique no nome de conta do usuário para abrir as configurações do espaço de trabalho da conta.
    
    3. Clique na seta suspensa ao lado do nome de sua conta, clique em **criar** e clique em **Biblioteca** para abrir a caixa de diálogo **Nova Biblioteca**.
    
    4. No controle suspenso **Origem**, escolha **Coordenada do Maven**.
    
    5. No cabeçalho **Instalar Artefatos do Maven**, insira `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` na caixa de texto **Coordenadas**. 
    
    6. Clique em **Criar biblioteca** para abrir a janela **Artefatos**.
    
    7. Em **Status dos clusters de execução**, marque a caixa de seleção **Anexar automaticamente a todos os clusters**.
    
    8. Repita as etapas de 1 a 7 para a coordenada `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` do Maven.
    
    9. Repita as etapas de 1 a 6 para a coordenada `org.geotools:gt-shapefile:19.2` do Maven.
    
    10. Clique em **Opções avançadas**.
    
    11. Insira `http://download.osgeo.org/webdav/geotools/` na caixa de texto **Repositório**. 
    
    12. Clique em **Criar biblioteca** para abrir a janela **Artefatos**. 
    
    13. Em **Status dos clusters de execução**, marque a caixa de seleção **Anexar automaticamente a todos os clusters**.

8. Adicione as bibliotecas dependentes adicionadas na etapa 7 ao trabalho criado no final da etapa 6:
    1. No espaço de trabalho do Azure Databricks, clique em **Trabalhos**.

    2. Clique no nome do trabalho criado na etapa 2 da seção **criar um trabalho do Databricks**. 
    
    3. Ao lado da seção **Bibliotecas Dependentes**, clique em **Adicionar** para abrir a caixa de diálogo **Adicionar Biblioteca Dependente**. 
    
    4. Em **Biblioteca de**, escolha **Espaço de trabalho**.
    
    5. Clique em **usuários**, nome de usuário e, em seguida, clique em `azure-eventhubs-spark_2.11:2.3.5`. 
    
    6. Clique em **OK**.
    
    7. Repita as etapas de 1 a 6 para `spark-cassandra-connector_2.11:2.3.1` e `gt-shapefile:19.2`.

9. Ao lado **Cluster:**, clique em **Editar**. Isso abrirá o diálogo **Configurar cluster**. Na lista suspensa **Tipo de Cluster**, escolha **Cluster Existente**. Na lista suspensa **Escolher Cluster**, escolha o cluster criado na seção **criar um cluster do Databricks**. Clique em **confirmar**.

10. Clique em **executar agora**.

### <a name="run-the-data-generator"></a>Execute o gerador de dados

1. Navegue até o diretório chamado `onprem` no repositório do GitHub.

2. Atualize os valores no arquivo **main.env** conforme descrito a seguir:

    ```shell
    RIDE_EVENT_HUB=[Connection string for the taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for the taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```
    A cadeia de conexão do hub de eventos da corrida de táxi é o valor **taxi-ride-eh** da seção de saída **eventHubs** na etapa 4 a seção *implantar recursos do Azure*. A cadeia de conexão do hub de eventos da tarifa de táxi é o valor **taxi-fare-eh** da seção de saída **eventHubs** na etapa 4 a seção *implantar recursos do Azure*.

3. Execute o comando a seguir para criar a imagem do Docker.

    ```bash
    docker build --no-cache -t dataloader .
    ```

4. Navegue de volta até o diretório pai.

    ```bash
    cd ..
    ```

5. Execute o comando a seguir para executar a imagem do Docker.

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

A saída deve ser semelhante ao seguinte:

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

Para verificar se o trabalho do Databricks está sendo executado corretamente, abra o portal do Azure e navegue até o banco de dados do Cosmos DB. Abra a folha **Data Explorer** e examine os dados da tabela **registros do táxi**. 

[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013). Universidade de Illinois em Urbana-Champaign. https://doi.org/10.13012/J8PN93H8
