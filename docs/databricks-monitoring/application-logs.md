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
# <a name="send-azure-databricks-application-logs-to-azure-monitor"></a>Enviar logs do aplicativo do Azure Databricks para o Azure Monitor

Este artigo mostra como enviar métricas e logs de aplicativo do Azure Databricks para um [espaço de trabalho do Log Analytics](/azure/azure-monitor/platform/manage-access). Ele usa o [biblioteca de monitoramento do Azure Databricks](https://github.com/mspnp/spark-monitoring), que está disponível no GitHub.

## <a name="prerequisites"></a>Pré-requisitos

Configurar seu cluster do Databricks do Azure para usar a biblioteca de monitoramento, conforme descrito em [configurar o Azure Databricks para enviar métricas para o Azure Monitor][config-cluster].

> [!NOTE]
> A biblioteca do monitoramento de fluxos de eventos no nível do Apache Spark e Spark Structured Streaming métricas de seus trabalhos para o Azure Monitor. Você não precisa fazer nenhuma alteração ao código do aplicativo para essas métricas e eventos.

## <a name="send-application-metrics-using-dropwizard"></a>O envio de métricas de aplicativo usando Dropwizard

Spark usa um sistema de métricas configurável baseado na biblioteca Dropwizard métricas. Para obter mais informações, consulte [métricas](https://spark.apache.org/docs/latest/monitoring.html#metrics) na documentação do Spark.

Para enviar as métricas de aplicativo do código do aplicativo do Azure Databricks para o Azure Monitor, siga estas etapas:

1. Criar o **spark-ouvintes-loganalytics-1.0-snapshot. jar** arquivo JAR, conforme descrito em [criar a biblioteca de monitoramento do Azure Databricks][build-lib].

1. Criar Dropwizard [medidores ou contadores](https://metrics.dropwizard.io/4.0.0/manual/core.html) no código do aplicativo. Você pode usar o `UserMetricsSystem` definida na biblioteca do monitoramento de classe. O exemplo a seguir cria um contador denominado `counter1`.

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

    A biblioteca de monitoramento inclui um [aplicativo de exemplo] [ sample-app] que demonstra como usar o `UserMetricsSystem` classe.

## <a name="send-application-logs-using-log4j"></a>Enviar logs de aplicativo usando Log4j

Para enviar os logs de aplicativo do Azure Databricks ao Azure Log Analytics usando o [appender de Log4j](https://logging.apache.org/log4j/2.x/manual/appenders.html) na biblioteca, siga estas etapas:

1. Criar o **spark-ouvintes-loganalytics-1.0-snapshot. jar** arquivo JAR, conforme descrito em [criar a biblioteca de monitoramento do Azure Databricks][build-lib].

1. Criar uma **log4j.properties** [arquivo de configuração](https://logging.apache.org/log4j/2.x/manual/configuration.html) para seu aplicativo. Inclua as seguintes propriedades de configuração. Substitua o nome do pacote de aplicativo e o log de nível onde indicado:

    ```YAML
    log4j.appender.A1=com.microsoft.pnp.logging.loganalytics.LogAnalyticsAppender
    log4j.appender.A1.layout=com.microsoft.pnp.logging.JSONLayout
    log4j.appender.A1.layout.LocationInfo=false
    log4j.additivity.<your application package name>=false
    log4j.logger.<your application package name>=<log level>, A1
    ```

    Você pode encontrar um arquivo de configuração de exemplo [aqui][log4j.properties].

1. No código do aplicativo, inclua o **spark de ouvintes de loganalytics** do projeto e, em seguida, importe `com.microsoft.pnp.logging.Log4jconfiguration` ao código do aplicativo.

    ```Scala
    import com.microsoft.pnp.logging.Log4jConfiguration
    ```

1. Configurar Log4j usando a **log4j.properties** arquivo que você criou na etapa 3:

    ```Scala
    getClass.getResourceAsStream("<path to file in your JAR file>/log4j.properties")) {
          stream => {
            Log4jConfiguration.configure(stream)
          }
    }
    ```

1. Adicione mensagens de log do Apache Spark no nível apropriado em seu código conforme necessário. Por exemplo, use o `logDebug` método para enviar uma mensagem de log de depuração. Para obter mais informações, consulte [registro em log] [ spark-logging] na documentação do Spark.

    ```Scala
    logTrace("Trace message")
    logDebug("Debug message")
    logInfo("Info message")
    logWarning("Warning message")
    logError("Error message")
    ```

## <a name="run-the-sample-application"></a>Executar o aplicativo de exemplo

A biblioteca de monitoramento inclui um [aplicativo de exemplo] [ sample-app] que demonstra como enviar logs de aplicativo e de métricas de aplicativo para o Azure Monitor. Para executar a amostra:

1. Criar o **trabalhos do spark** do projeto na biblioteca de monitoramento, conforme descrito em [criar a biblioteca de monitoramento do Azure Databricks][build-lib].

1. Navegue até seu espaço de trabalho do Databricks e crie um novo trabalho, conforme descrito [aqui](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).

1. Na página de detalhes do trabalho, selecione **definir JAR**.

1. Carregar o arquivo JAR do `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.

1. Para **classe principal**, insira `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.

1. Selecione um cluster que já está configurado para usar a biblioteca de monitoramento. Ver [configurar o Azure Databricks para enviar métricas para o Azure Monitor][config-cluster].

Quando o trabalho é executado, você pode exibir os logs de aplicativo e as métricas em seu espaço de trabalho do Log Analytics.

Logs do aplicativo aparecem em SparkLoggingEvent_CL:

```Kusto
SparkLoggingEvent_CL | where logger_name_s contains "com.microsoft.pnp"
```

Métricas de aplicativo são exibidas em SparkMetric_CL:

```Kusto
SparkMetric_CL | where name_s contains "rowcounter" | limit 50
```

## <a name="next-steps"></a>Próximas etapas

Implante o painel de controle que acompanha esta biblioteca de código para solucionar problemas de desempenho em suas cargas de trabalho do Databricks do Azure de produção de monitoramento de desempenho.

> [!div class="nextstepaction"]
> [Usar painéis para visualizar métricas do Azure Databricks](./dashboards.md)

<!-- links -->

[build-lib]: ./configure-cluster.md##build-the-azure-databricks-monitoring-library
[config-cluster]: ./configure-cluster.md
[log4j.properties]: https://github.com/mspnp/spark-monitoring/blob/master/src/spark-jobs/src/main/resources/com/microsoft/pnp/samplejob/log4j.properties
[sample-app]: https://github.com/mspnp/spark-monitoring/tree/master/src/spark-jobs
[spark-logging]: https://spark.apache.org/docs/2.3.0/api/java/org/apache/spark/internal/Logging.html