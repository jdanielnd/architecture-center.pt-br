---
title: Configurar o Azure Databricks para enviar métricas para o Azure Monitor
description: Uma biblioteca de scala para habilitar o monitoramento de métricas e registro em log dados no Azure Log Analytics
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: af6b6433f87964ac60c179ecf498e54129344126
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503365"
---
<!-- markdownlint-disable MD040 -->

# <a name="configure-azure-databricks-to-send-metrics-to-azure-monitor"></a>Configurar o Azure Databricks para enviar métricas para o Azure Monitor

Este artigo mostra como configurar um cluster do Databricks do Azure para enviar métricas para um [espaço de trabalho do Log Analytics](/azure/azure-monitor/platform/manage-access). Ele usa o [biblioteca de monitoramento do Azure Databricks](https://github.com/mspnp/spark-monitoring), que está disponível no GitHub. Compreensão do Java, Scala e Maven são recomendadas como prerequisistes.

## <a name="about-the-azure-databricks-monitoring-library"></a>Sobre o Azure Databricks biblioteca de monitoramento

O [repositório GitHub](https://github.com/mspnp/spark-monitoring) para a biblioteca de monitoramento do Azure Databricks tem a estrutura de diretórios a seguir:

```
/src  
    /spark-jobs  
    /spark-listeners-loganalytics  
    /spark-listeners  
    /pom.xml  
```

O **trabalhos do spark** diretório é um aplicativo Spark de exemplo que demonstra como implementar um contador de métrica do aplicativo Spark.

O **ouvintes do spark** diretório inclui a funcionalidade que permite que o Azure Databricks enviar eventos do Apache Spark no nível de serviço para um espaço de trabalho do Log Analytics do Azure. O Azure Databricks é um serviço baseado no Apache Spark, que inclui um conjunto de APIs estruturados para processamento de dados usando SQL, DataFrames e conjuntos de dados em lotes. Com o Apache Spark 2.0, foi adicionado suporte para [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), uma API baseada no processamento de APIs de lote do Spark de processamento de fluxo de dados.

O **spark de ouvintes de loganalytics** diretório inclui um coletor para ouvintes de Spark, um coletor para DropWizard e um cliente para um espaço de trabalho do Azure Log Analytics. Este diretório também inclui um Appender de log4j para os logs de aplicativo do Apache Spark.

O **spark de ouvintes de loganalytics** e **spark ouvintes** diretórios contêm o código para criar os dois arquivos JAR que são implantados no cluster do Databricks. O **spark ouvintes** diretório inclui um **scripts** diretório que contém um script de inicialização do nó de cluster para copiar o JAR de arquivos de um diretório de preparo no sistema de arquivos do Azure Databricks para nós de execução.

O **POM. XML** arquivo é o principal arquivo de build do Maven para o projeto inteiro.

## <a name="prerequisites"></a>Pré-requisitos

Para começar, implante os seguintes recursos do Azure:

- Um espaço de trabalho do Databricks do Azure. Confira [Início Rápido: Executar um trabalho do Spark no Azure Databricks usando o portal do Azure](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).
- Um workspace do Log Analytics. Ver [criar um espaço de trabalho do Log Analytics no portal do Azure](/azure/azure-monitor/learn/quick-create-workspace).

Em seguida, instale o [CLI do Azure Databricks](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli). Um token de acesso pessoal do Azure Databricks é necessário para usar a CLI. Para obter instruções, consulte [gerar um token](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).

Localize a ID do espaço de trabalho do Log Analytics e a chave. Para obter instruções, consulte [obter a ID do espaço de trabalho e a chave](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key). Quando você configura o cluster do Databricks, você precisará desses valores.

Para compilar a biblioteca de monitoramento, você precisará de um IDE de Java com os seguintes recursos:

- [Java Development Kit (JDK) versão 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [Linguagem scala 2.11 SDK](https://www.scala-lang.org/download/)
- [Apache Maven 3.5.4](http://maven.apache.org/download.cgi)

## <a name="build-the-azure-databricks-monitoring-library"></a>Compilar o Databricks do Azure, biblioteca de monitoramento

Para compilar a biblioteca de monitoramento do Azure Databricks, execute as seguintes etapas:

1. Clone, crie um fork ou baixe o [repositório GitHub](https://github.com/mspnp/spark-monitoring).

1. Importar o arquivo de modelo de objeto do projeto Maven, _POM. XML_, localizado na **/src** pasta em seu projeto. Isso vai importar três projetos:

    - trabalhos do Spark
    - ouvintes do Spark
    - Spark de ouvintes de loganalytics

1. Execute o Maven **pacote** fase de compilação no seu IDE para compilar os arquivos JAR para cada um desses projetos Java:

    |Project| Arquivo JAR|
    |-------|---------|
    |trabalhos do Spark|Spark-trabalhos-1.0-snapshot. jar|
    |ouvintes do Spark|Spark-ouvintes-1.0-snapshot. jar|
    |Spark de ouvintes de loganalytics|Spark-ouvintes-loganalytics-1.0-snapshot. jar|

1. Usar a CLI do Databricks do Azure para criar um diretório chamado **dbfs: / databricks/monitoramento-preparo**:  

    ```bash
    dbfs mkdirs dbfs:/databricks/monitoring-staging
    ```

1. Usar a CLI do Databricks do Azure para copiar **/src/spark-listeners/scripts/listeners.sh** anteriormente no diretório:

    ```bash
    dbfs cp ./src/spark-listeners/scripts/listeners.sh dbfs:/databricks/monitoring-staging/listeners.sh
    ```

1. Usar a CLI do Databricks do Azure para copiar **/src/spark-listeners/scripts/metrics.properties** para o diretório criado anteriormente:

    ```bash
    dbfs cp <local path to metrics.properties> dbfs:/databricks/monitoring-staging/metrics.properties
    ```

1. Usar a CLI do Databricks do Azure para copiar **spark-ouvintes-1.0-snapshot. jar** e **spark-ouvintes-loganalytics-1.0-snapshot. jar** para o diretório criado anteriormente:

    ```bash
    dbfs cp ./src/spark-listeners/target/spark-listeners-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-1.0-SNAPSHOT.jar
    dbfs cp ./src/spark-listeners-loganalytics/target/spark-listeners-loganalytics-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-loganalytics-1.0-SNAPSHOT.jar
    ```

## <a name="create-and-configure-an-azure-databricks-cluster"></a>Criar e configurar um cluster do Azure Databricks

Para criar e configurar um cluster do Databricks do Azure, siga estas etapas:

1. Navegue até seu espaço de trabalho do Azure Databricks no portal do Azure.
1. Na home page, clique em **novo Cluster**.
1. Insira um nome para o cluster na **nome do Cluster** caixa de texto.
1. No **versão de tempo de execução Databricks** lista suspensa, selecione **4.3** ou maior.
1. Sob **opções avançadas**, clique no **Spark** guia. Insira os seguintes pares de nome-valor na **Spark Config** caixa de texto:

    ```
    spark.extraListeners com.databricks.backend.daemon.driver.DBCEventLoggingListener,org.apache.spark.listeners.UnifiedSparkListener
    spark.unifiedListener.sink org.apache.spark.listeners.sink.loganalytics.LogAnalyticsListenerSink
    spark.unifiedListener.logBlockUpdates false
    ```

1. Enquanto ainda sob o **Spark** , insira os seguintes valores na **variáveis de ambiente** caixa de texto:

    ```
    LOG_ANALYTICS_WORKSPACE_ID=[your Azure Log Analytics workspace ID]
    LOG_ANALYTICS_WORKSPACE_KEY=[your Azure Log Analytics workspace key]
    ```

    ![Captura de tela da interface do usuário do Databricks](./_images/create-cluster1.png)

1. Enquanto ainda sob o **opções avançadas** seção, clique no **Scripts de inicialização de** guia.
1. No **destino** lista suspensa, selecione **DBFS**.
1. Sob **caminho do Script de inicialização**, insira `dbfs:/databricks/monitoring-staging/listeners.sh`.
1. Clique em **Adicionar**.

    ![Captura de tela da interface do usuário do Databricks](./_images/create-cluster2.png)

1. Clique em **criar Cluster** para criar o cluster.

## <a name="view-metrics"></a>Métricas de exibição

Depois de concluir essas etapas, o cluster Databricks transmite alguns dados de métrica sobre o cluster para o Azure Monitor. Esses dados de log estão disponíveis em seu espaço de trabalho do Log Analytics do Azure com o "Active Directory | Logs personalizados | Esquema de SparkMetric_CL". Para obter mais informações sobre os tipos de métricas que são registrados em log, consulte [métricas Core](https://metrics.dropwizard.io/4.0.0/manual/core.html) na documentação do Dropwizard. O tipo de métrica, como o medidor, contador ou histograma, é gravado para o campo de tipo.

Além disso, a biblioteca de fluxos de eventos no nível do Apache Spark e Spark Structured Streaming métricas de seus trabalhos para o Azure Monitor. Você não precisa fazer nenhuma alteração ao código do aplicativo para essas métricas e eventos. Dados de log de Streaming estruturado do Spark estão disponíveis sob o "Active Directory | Logs personalizados | Esquema de SparkListenerEvent_CL".

![Captura de tela de um espaço de trabalho do Log Analytics](./_images/workspace.png)

Para obter mais informações sobre como exibir logs, consulte [exibindo e analisando dados de log no Azure Monitor](/azure/azure-monitor/log-query/portals).

## <a name="next-steps"></a>Próximas etapas

Este artigo descreveu como configurar o cluster para enviar métricas para o Azure Monitor. O próximo artigo mostra como usar a biblioteca de monitoramento do Azure Databricks para enviar logs e métricas de aplicativo.

> [!div class="nextstepaction"]
> [Enviar logs de aplicativo para o Azure Monitor](./application-logs.md)
