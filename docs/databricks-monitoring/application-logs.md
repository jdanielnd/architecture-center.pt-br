---
title: Enviar logs do aplicativo do Azure Databricks para o Azure Monitor
description: Como enviar logs personalizados e métricas do Azure Databricks para o Azure Monitor
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: ea67122d7871663e8aaf42b7af0043492f63b6b1
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639172"
---
# <a name="send-azure-databricks-application-logs-to-azure-monitor"></a><span data-ttu-id="74f0a-103">Enviar logs do aplicativo do Azure Databricks para o Azure Monitor</span><span class="sxs-lookup"><span data-stu-id="74f0a-103">Send Azure Databricks application logs to Azure Monitor</span></span>

<span data-ttu-id="74f0a-104">Este artigo mostra como enviar métricas e logs de aplicativo do Azure Databricks para um [espaço de trabalho do Log Analytics](/azure/azure-monitor/platform/manage-access).</span><span class="sxs-lookup"><span data-stu-id="74f0a-104">This article shows how to send application logs and metrics from Azure Databricks to a [Log Analytics workspace](/azure/azure-monitor/platform/manage-access).</span></span> <span data-ttu-id="74f0a-105">Ele usa o [biblioteca de monitoramento do Azure Databricks](https://github.com/mspnp/spark-monitoring), que está disponível no GitHub.</span><span class="sxs-lookup"><span data-stu-id="74f0a-105">It uses the [Azure Databricks Monitoring Library](https://github.com/mspnp/spark-monitoring), which is available on GitHub.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="74f0a-106">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="74f0a-106">Prerequisites</span></span>

<span data-ttu-id="74f0a-107">Configurar seu cluster do Databricks do Azure para usar a biblioteca de monitoramento, conforme descrito em [configurar o Azure Databricks para enviar métricas para o Azure Monitor][config-cluster].</span><span class="sxs-lookup"><span data-stu-id="74f0a-107">Configure your Azure Databricks cluster to use the monitoring library, as described in [Configure Azure Databricks to send metrics to Azure Monitor][config-cluster].</span></span>

> [!NOTE]
> <span data-ttu-id="74f0a-108">A biblioteca do monitoramento de fluxos de eventos no nível do Apache Spark e Spark Structured Streaming métricas de seus trabalhos para o Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="74f0a-108">The monitoring library streams Apache Spark level events and Spark Structured Streaming metrics from your jobs to Azure Monitor.</span></span> <span data-ttu-id="74f0a-109">Você não precisa fazer nenhuma alteração ao código do aplicativo para essas métricas e eventos.</span><span class="sxs-lookup"><span data-stu-id="74f0a-109">You don't need to make any changes to your application code for these events and metrics.</span></span>

## <a name="send-application-metrics-using-dropwizard"></a><span data-ttu-id="74f0a-110">O envio de métricas de aplicativo usando Dropwizard</span><span class="sxs-lookup"><span data-stu-id="74f0a-110">Send application metrics using Dropwizard</span></span>

<span data-ttu-id="74f0a-111">Spark usa um sistema de métricas configurável baseado na biblioteca Dropwizard métricas.</span><span class="sxs-lookup"><span data-stu-id="74f0a-111">Spark uses a configurable metrics system based on the Dropwizard Metrics Library.</span></span> <span data-ttu-id="74f0a-112">Para obter mais informações, consulte [métricas](https://spark.apache.org/docs/latest/monitoring.html#metrics) na documentação do Spark.</span><span class="sxs-lookup"><span data-stu-id="74f0a-112">For more information, see [Metrics](https://spark.apache.org/docs/latest/monitoring.html#metrics) in the Spark documentation.</span></span>

<span data-ttu-id="74f0a-113">Para enviar as métricas de aplicativo do código do aplicativo do Azure Databricks para o Azure Monitor, siga estas etapas:</span><span class="sxs-lookup"><span data-stu-id="74f0a-113">To send application metrics from Azure Databricks application code to Azure Monitor, follow these steps:</span></span>

1. <span data-ttu-id="74f0a-114">Criar o **spark-ouvintes-loganalytics-1.0-snapshot. jar** arquivo JAR, conforme descrito em [criar a biblioteca de monitoramento do Azure Databricks][build-lib].</span><span class="sxs-lookup"><span data-stu-id="74f0a-114">Build the **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR file as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="74f0a-115">Criar Dropwizard [medidores ou contadores](https://metrics.dropwizard.io/4.0.0/manual/core.html) no código do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="74f0a-115">Create Dropwizard [gauges or counters](https://metrics.dropwizard.io/4.0.0/manual/core.html) in your application code.</span></span> <span data-ttu-id="74f0a-116">Você pode usar o `UserMetricsSystem` definida na biblioteca do monitoramento de classe.</span><span class="sxs-lookup"><span data-stu-id="74f0a-116">You can use the `UserMetricsSystem` class defined in the monitoring library.</span></span> <span data-ttu-id="74f0a-117">O exemplo a seguir cria um contador denominado `counter1`.</span><span class="sxs-lookup"><span data-stu-id="74f0a-117">The following example creates a counter named `counter1`.</span></span>

    ```Scala
    import org.apache.spark.metrics.UserMetricsSystems
    import org.apache.spark.sql.SparkSession

    object StreamingQueryListenerSampleJob  {

      private final val METRICS_NAMESPACE = "samplejob"
      private final val COUNTER_NAME = "counter1"

      def main(args: Array[String]): Unit = {

        val spark = SparkSession
          .builder
          .getOrCreate

        val driverMetricsSystem = UserMetricsSystems
            .getMetricSystem(METRICS_NAMESPACE, builder => {
              builder.registerCounter(COUNTER_NAME)
            })

        driverMetricsSystem.counter(COUNTER_NAME).inc(5)
      }
    }
    ```

    <span data-ttu-id="74f0a-118">A biblioteca de monitoramento inclui um [aplicativo de exemplo] [ sample-app] que demonstra como usar o `UserMetricsSystem` classe.</span><span class="sxs-lookup"><span data-stu-id="74f0a-118">The monitoring library includes a [sample application][sample-app] that demonstrates how to use the `UserMetricsSystem` class.</span></span>

## <a name="send-application-logs-using-log4j"></a><span data-ttu-id="74f0a-119">Enviar logs de aplicativo usando Log4j</span><span class="sxs-lookup"><span data-stu-id="74f0a-119">Send application logs using Log4j</span></span>

<span data-ttu-id="74f0a-120">Para enviar os logs de aplicativo do Azure Databricks ao Azure Log Analytics usando o [appender de Log4j](https://logging.apache.org/log4j/2.x/manual/appenders.html) na biblioteca, siga estas etapas:</span><span class="sxs-lookup"><span data-stu-id="74f0a-120">To send your Azure Databricks application logs to Azure Log Analytics using the [Log4j appender](https://logging.apache.org/log4j/2.x/manual/appenders.html) in the library, follow these steps:</span></span>

1. <span data-ttu-id="74f0a-121">Criar o **spark-ouvintes-loganalytics-1.0-snapshot. jar** arquivo JAR, conforme descrito em [criar a biblioteca de monitoramento do Azure Databricks][build-lib].</span><span class="sxs-lookup"><span data-stu-id="74f0a-121">Build the **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR file as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="74f0a-122">Criar uma **log4j.properties** [arquivo de configuração](https://logging.apache.org/log4j/2.x/manual/configuration.html) para seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="74f0a-122">Create a **log4j.properties** [configuration file](https://logging.apache.org/log4j/2.x/manual/configuration.html) for your application.</span></span> <span data-ttu-id="74f0a-123">Inclua as seguintes propriedades de configuração.</span><span class="sxs-lookup"><span data-stu-id="74f0a-123">Include the following configuration properties.</span></span> <span data-ttu-id="74f0a-124">Substitua o nome do pacote de aplicativo e o log de nível onde indicado:</span><span class="sxs-lookup"><span data-stu-id="74f0a-124">Substitute your application package name and log level where indicated:</span></span>

    ```YAML
    log4j.appender.A1=com.microsoft.pnp.logging.loganalytics.LogAnalyticsAppender
    log4j.appender.A1.layout=com.microsoft.pnp.logging.JSONLayout
    log4j.appender.A1.layout.LocationInfo=false
    log4j.additivity.<your application package name>=false
    log4j.logger.<your application package name>=<log level>, A1
    ```

    <span data-ttu-id="74f0a-125">Você pode encontrar um arquivo de configuração de exemplo [aqui][log4j.properties].</span><span class="sxs-lookup"><span data-stu-id="74f0a-125">You can find a sample configuration file [here][log4j.properties].</span></span>

1. <span data-ttu-id="74f0a-126">No código do aplicativo, inclua o **spark de ouvintes de loganalytics** do projeto e, em seguida, importe `com.microsoft.pnp.logging.Log4jconfiguration` ao código do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="74f0a-126">In your application code, include the **spark-listeners-loganalytics** project, and import `com.microsoft.pnp.logging.Log4jconfiguration` to your application code.</span></span>

    ```Scala
    import com.microsoft.pnp.logging.Log4jConfiguration
    ```

1. <span data-ttu-id="74f0a-127">Configurar Log4j usando a **log4j.properties** arquivo que você criou na etapa 3:</span><span class="sxs-lookup"><span data-stu-id="74f0a-127">Configure Log4j using the **log4j.properties** file you created in step 3:</span></span>

    ```Scala
    getClass.getResourceAsStream("<path to file in your JAR file>/log4j.properties")) {
          stream => {
            Log4jConfiguration.configure(stream)
          }
    }
    ```

1. <span data-ttu-id="74f0a-128">Adicione mensagens de log do Apache Spark no nível apropriado em seu código conforme necessário.</span><span class="sxs-lookup"><span data-stu-id="74f0a-128">Add Apache Spark log messages at the appropriate level in your code as required.</span></span> <span data-ttu-id="74f0a-129">Por exemplo, use o `logDebug` método para enviar uma mensagem de log de depuração.</span><span class="sxs-lookup"><span data-stu-id="74f0a-129">For example, use the `logDebug` method to send a debug log message.</span></span> <span data-ttu-id="74f0a-130">Para obter mais informações, consulte [registro em log] [ spark-logging] na documentação do Spark.</span><span class="sxs-lookup"><span data-stu-id="74f0a-130">For more information, see [Logging][spark-logging] in the Spark documentation.</span></span>

    ```Scala
    logTrace("Trace message")
    logDebug("Debug message")
    logInfo("Info message")
    logWarning("Warning message")
    logError("Error message")
    ```

## <a name="run-the-sample-application"></a><span data-ttu-id="74f0a-131">Executar o aplicativo de exemplo</span><span class="sxs-lookup"><span data-stu-id="74f0a-131">Run the sample application</span></span>

<span data-ttu-id="74f0a-132">A biblioteca de monitoramento inclui um [aplicativo de exemplo] [ sample-app] que demonstra como enviar logs de aplicativo e de métricas de aplicativo para o Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="74f0a-132">The monitoring library includes a [sample application][sample-app] that demonstrates how to send both application metrics and application logs to Azure Monitor.</span></span> <span data-ttu-id="74f0a-133">Para executar a amostra:</span><span class="sxs-lookup"><span data-stu-id="74f0a-133">To run the sample:</span></span>

1. <span data-ttu-id="74f0a-134">Criar o **trabalhos do spark** do projeto na biblioteca de monitoramento, conforme descrito em [criar a biblioteca de monitoramento do Azure Databricks][build-lib].</span><span class="sxs-lookup"><span data-stu-id="74f0a-134">Build the **spark-jobs** project in the monitoring library, as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="74f0a-135">Navegue até seu espaço de trabalho do Databricks e crie um novo trabalho, conforme descrito [aqui](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).</span><span class="sxs-lookup"><span data-stu-id="74f0a-135">Navigate to your Databricks workspace and create a new job, as described [here](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).</span></span>

1. <span data-ttu-id="74f0a-136">Na página de detalhes do trabalho, selecione **definir JAR**.</span><span class="sxs-lookup"><span data-stu-id="74f0a-136">In the job detail page, select **Set JAR**.</span></span>

1. <span data-ttu-id="74f0a-137">Carregar o arquivo JAR do `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.</span><span class="sxs-lookup"><span data-stu-id="74f0a-137">Upload the JAR file from `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.</span></span>

1. <span data-ttu-id="74f0a-138">Para **classe principal**, insira `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.</span><span class="sxs-lookup"><span data-stu-id="74f0a-138">For **Main class**, enter `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.</span></span>

1. <span data-ttu-id="74f0a-139">Selecione um cluster que já está configurado para usar a biblioteca de monitoramento.</span><span class="sxs-lookup"><span data-stu-id="74f0a-139">Select a cluster that is already configured to use the monitoring library.</span></span> <span data-ttu-id="74f0a-140">Ver [configurar o Azure Databricks para enviar métricas para o Azure Monitor][config-cluster].</span><span class="sxs-lookup"><span data-stu-id="74f0a-140">See [Configure Azure Databricks to send metrics to Azure Monitor][config-cluster].</span></span>

<span data-ttu-id="74f0a-141">Quando o trabalho é executado, você pode exibir os logs de aplicativo e as métricas em seu espaço de trabalho do Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="74f0a-141">When the job runs, you can view the application logs and metrics in your Log Analytics workspace.</span></span>

<span data-ttu-id="74f0a-142">Logs do aplicativo aparecem em SparkLoggingEvent_CL:</span><span class="sxs-lookup"><span data-stu-id="74f0a-142">Application logs appear under SparkLoggingEvent_CL:</span></span>

```Kusto
SparkLoggingEvent_CL | where logger_name_s contains "com.microsoft.pnp"
```

<span data-ttu-id="74f0a-143">Métricas de aplicativo são exibidas em SparkMetric_CL:</span><span class="sxs-lookup"><span data-stu-id="74f0a-143">Application metrics appear under SparkMetric_CL:</span></span>

```Kusto
SparkMetric_CL | where name_s contains "rowcounter" | limit 50
```

## <a name="next-steps"></a><span data-ttu-id="74f0a-144">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="74f0a-144">Next steps</span></span>

<span data-ttu-id="74f0a-145">Implante o painel de controle que acompanha esta biblioteca de código para solucionar problemas de desempenho em suas cargas de trabalho do Databricks do Azure de produção de monitoramento de desempenho.</span><span class="sxs-lookup"><span data-stu-id="74f0a-145">Deploy the performance monitoring dashboard that accompanies this code library to troubleshoot performance issues in your production Azure Databricks workloads.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="74f0a-146">Usar painéis para visualizar métricas do Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="74f0a-146">Use dashboards to visualize Azure Databricks metrics</span></span>](./dashboards.md)

<!-- links -->

[build-lib]: ./configure-cluster.md##build-the-azure-databricks-monitoring-library
[config-cluster]: ./configure-cluster.md
[log4j.properties]: https://github.com/mspnp/spark-monitoring/blob/master/src/spark-jobs/src/main/resources/com/microsoft/pnp/samplejob/log4j.properties
[sample-app]: https://github.com/mspnp/spark-monitoring/tree/master/src/spark-jobs
[spark-logging]: https://spark.apache.org/docs/2.3.0/api/java/org/apache/spark/internal/Logging.html