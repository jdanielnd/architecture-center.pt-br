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

# <a name="configure-azure-databricks-to-send-metrics-to-azure-monitor"></a><span data-ttu-id="f60cc-103">Configurar o Azure Databricks para enviar métricas para o Azure Monitor</span><span class="sxs-lookup"><span data-stu-id="f60cc-103">Configure Azure Databricks to send metrics to Azure Monitor</span></span>

<span data-ttu-id="f60cc-104">Este artigo mostra como configurar um cluster do Databricks do Azure para enviar métricas para um [espaço de trabalho do Log Analytics](/azure/azure-monitor/platform/manage-access).</span><span class="sxs-lookup"><span data-stu-id="f60cc-104">This article shows how to configure an Azure Databricks cluster to send metrics to a [Log Analytics workspace](/azure/azure-monitor/platform/manage-access).</span></span> <span data-ttu-id="f60cc-105">Ele usa o [biblioteca de monitoramento do Azure Databricks](https://github.com/mspnp/spark-monitoring), que está disponível no GitHub.</span><span class="sxs-lookup"><span data-stu-id="f60cc-105">It uses the [Azure Databricks Monitoring Library](https://github.com/mspnp/spark-monitoring), which is available on GitHub.</span></span> <span data-ttu-id="f60cc-106">Compreensão do Java, Scala e Maven são recomendadas como prerequisistes.</span><span class="sxs-lookup"><span data-stu-id="f60cc-106">Understanding of Java, Scala, and Maven are recommended as prerequisistes.</span></span>

## <a name="about-the-azure-databricks-monitoring-library"></a><span data-ttu-id="f60cc-107">Sobre o Azure Databricks biblioteca de monitoramento</span><span class="sxs-lookup"><span data-stu-id="f60cc-107">About the Azure Databricks Monitoring Library</span></span>

<span data-ttu-id="f60cc-108">O [repositório GitHub](https://github.com/mspnp/spark-monitoring) para a biblioteca de monitoramento do Azure Databricks tem a estrutura de diretórios a seguir:</span><span class="sxs-lookup"><span data-stu-id="f60cc-108">The [GitHub repo](https://github.com/mspnp/spark-monitoring) for the Azure Databricks Monitoring Library has following directory structure:</span></span>

```
/src  
    /spark-jobs  
    /spark-listeners-loganalytics  
    /spark-listeners  
    /pom.xml  
```

<span data-ttu-id="f60cc-109">O **trabalhos do spark** diretório é um aplicativo Spark de exemplo que demonstra como implementar um contador de métrica do aplicativo Spark.</span><span class="sxs-lookup"><span data-stu-id="f60cc-109">The **spark-jobs** directory is a sample Spark application demonstrating how to implement a Spark application metric counter.</span></span>

<span data-ttu-id="f60cc-110">O **ouvintes do spark** diretório inclui a funcionalidade que permite que o Azure Databricks enviar eventos do Apache Spark no nível de serviço para um espaço de trabalho do Log Analytics do Azure.</span><span class="sxs-lookup"><span data-stu-id="f60cc-110">The **spark-listeners** directory includes functionality that enables Azure Databricks to send Apache Spark events at the service level to an Azure Log Analytics workspace.</span></span> <span data-ttu-id="f60cc-111">O Azure Databricks é um serviço baseado no Apache Spark, que inclui um conjunto de APIs estruturados para processamento de dados usando SQL, DataFrames e conjuntos de dados em lotes.</span><span class="sxs-lookup"><span data-stu-id="f60cc-111">Azure Databricks is a service based on Apache Spark, which includes a set of structured APIs for batch processing data using Datasets, DataFrames, and SQL.</span></span> <span data-ttu-id="f60cc-112">Com o Apache Spark 2.0, foi adicionado suporte para [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), uma API baseada no processamento de APIs de lote do Spark de processamento de fluxo de dados.</span><span class="sxs-lookup"><span data-stu-id="f60cc-112">With Apache Spark 2.0, support was added for [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), a data stream processing API built on Spark's batch processing APIs.</span></span>

<span data-ttu-id="f60cc-113">O **spark de ouvintes de loganalytics** diretório inclui um coletor para ouvintes de Spark, um coletor para DropWizard e um cliente para um espaço de trabalho do Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="f60cc-113">The **spark-listeners-loganalytics** directory includes a sink for Spark listeners, a sink for DropWizard, and a client for an Azure Log Analytics Workspace.</span></span> <span data-ttu-id="f60cc-114">Este diretório também inclui um Appender de log4j para os logs de aplicativo do Apache Spark.</span><span class="sxs-lookup"><span data-stu-id="f60cc-114">This directory also includes a log4j Appender for your Apache Spark application logs.</span></span>

<span data-ttu-id="f60cc-115">O **spark de ouvintes de loganalytics** e **spark ouvintes** diretórios contêm o código para criar os dois arquivos JAR que são implantados no cluster do Databricks.</span><span class="sxs-lookup"><span data-stu-id="f60cc-115">The **spark-listeners-loganalytics** and **spark-listeners** directories contain the code for building the two JAR files that are deployed to the Databricks cluster.</span></span> <span data-ttu-id="f60cc-116">O **spark ouvintes** diretório inclui um **scripts** diretório que contém um script de inicialização do nó de cluster para copiar o JAR de arquivos de um diretório de preparo no sistema de arquivos do Azure Databricks para nós de execução.</span><span class="sxs-lookup"><span data-stu-id="f60cc-116">The **spark-listeners** directory includes a **scripts** directory that contains a cluster node initialization script to copy the JAR files from a staging directory in the Azure Databricks file system to execution nodes.</span></span>

<span data-ttu-id="f60cc-117">O **POM. XML** arquivo é o principal arquivo de build do Maven para o projeto inteiro.</span><span class="sxs-lookup"><span data-stu-id="f60cc-117">The **pom.xml** file is the main Maven build file for the entire project.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="f60cc-118">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="f60cc-118">Prerequisites</span></span>

<span data-ttu-id="f60cc-119">Para começar, implante os seguintes recursos do Azure:</span><span class="sxs-lookup"><span data-stu-id="f60cc-119">To get started, deploy the following Azure resources:</span></span>

- <span data-ttu-id="f60cc-120">Um espaço de trabalho do Databricks do Azure.</span><span class="sxs-lookup"><span data-stu-id="f60cc-120">An Azure Databricks workspace.</span></span> <span data-ttu-id="f60cc-121">Confira [Início Rápido: Executar um trabalho do Spark no Azure Databricks usando o portal do Azure](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).</span><span class="sxs-lookup"><span data-stu-id="f60cc-121">See [Quickstart: Run a Spark job on Azure Databricks using the Azure portal](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).</span></span>
- <span data-ttu-id="f60cc-122">Um workspace do Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="f60cc-122">A Log Analytics workspace.</span></span> <span data-ttu-id="f60cc-123">Ver [criar um espaço de trabalho do Log Analytics no portal do Azure](/azure/azure-monitor/learn/quick-create-workspace).</span><span class="sxs-lookup"><span data-stu-id="f60cc-123">See [Create a Log Analytics workspace in the Azure portal](/azure/azure-monitor/learn/quick-create-workspace).</span></span>

<span data-ttu-id="f60cc-124">Em seguida, instale o [CLI do Azure Databricks](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli).</span><span class="sxs-lookup"><span data-stu-id="f60cc-124">Next, install the [Azure Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli).</span></span> <span data-ttu-id="f60cc-125">Um token de acesso pessoal do Azure Databricks é necessário para usar a CLI.</span><span class="sxs-lookup"><span data-stu-id="f60cc-125">An Azure Databricks personal access token is required to use the CLI.</span></span> <span data-ttu-id="f60cc-126">Para obter instruções, consulte [gerar um token](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).</span><span class="sxs-lookup"><span data-stu-id="f60cc-126">For instructions, see [Generate a token](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).</span></span>

<span data-ttu-id="f60cc-127">Localize a ID do espaço de trabalho do Log Analytics e a chave.</span><span class="sxs-lookup"><span data-stu-id="f60cc-127">Find your Log Analytics workspace ID and key.</span></span> <span data-ttu-id="f60cc-128">Para obter instruções, consulte [obter a ID do espaço de trabalho e a chave](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key).</span><span class="sxs-lookup"><span data-stu-id="f60cc-128">For instructions, see [Obtain workspace ID and key](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key).</span></span> <span data-ttu-id="f60cc-129">Quando você configura o cluster do Databricks, você precisará desses valores.</span><span class="sxs-lookup"><span data-stu-id="f60cc-129">You will need these values when you configure the Databricks cluster.</span></span>

<span data-ttu-id="f60cc-130">Para compilar a biblioteca de monitoramento, você precisará de um IDE de Java com os seguintes recursos:</span><span class="sxs-lookup"><span data-stu-id="f60cc-130">To build the monitoring library, you will need a Java IDE with the following resources:</span></span>

- [<span data-ttu-id="f60cc-131">Java Development Kit (JDK) versão 1.8</span><span class="sxs-lookup"><span data-stu-id="f60cc-131">Java Development Kit (JDK) version 1.8</span></span>](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [<span data-ttu-id="f60cc-132">Linguagem scala 2.11 SDK</span><span class="sxs-lookup"><span data-stu-id="f60cc-132">Scala language SDK 2.11</span></span>](https://www.scala-lang.org/download/)
- [<span data-ttu-id="f60cc-133">Apache Maven 3.5.4</span><span class="sxs-lookup"><span data-stu-id="f60cc-133">Apache Maven 3.5.4</span></span>](http://maven.apache.org/download.cgi)

## <a name="build-the-azure-databricks-monitoring-library"></a><span data-ttu-id="f60cc-134">Compilar o Databricks do Azure, biblioteca de monitoramento</span><span class="sxs-lookup"><span data-stu-id="f60cc-134">Build the Azure Databricks Monitoring Library</span></span>

<span data-ttu-id="f60cc-135">Para compilar a biblioteca de monitoramento do Azure Databricks, execute as seguintes etapas:</span><span class="sxs-lookup"><span data-stu-id="f60cc-135">To build the Azure Databricks Monitoring Library, perform the following steps:</span></span>

1. <span data-ttu-id="f60cc-136">Clone, crie um fork ou baixe o [repositório GitHub](https://github.com/mspnp/spark-monitoring).</span><span class="sxs-lookup"><span data-stu-id="f60cc-136">Clone, fork, or download the [GitHub repository](https://github.com/mspnp/spark-monitoring).</span></span>

1. <span data-ttu-id="f60cc-137">Importar o arquivo de modelo de objeto do projeto Maven, _POM. XML_, localizado na **/src** pasta em seu projeto.</span><span class="sxs-lookup"><span data-stu-id="f60cc-137">Import the Maven project object model file, _pom.xml_, located in the **/src** folder into your project.</span></span> <span data-ttu-id="f60cc-138">Isso vai importar três projetos:</span><span class="sxs-lookup"><span data-stu-id="f60cc-138">This will import three projects:</span></span>

    - <span data-ttu-id="f60cc-139">trabalhos do Spark</span><span class="sxs-lookup"><span data-stu-id="f60cc-139">spark-jobs</span></span>
    - <span data-ttu-id="f60cc-140">ouvintes do Spark</span><span class="sxs-lookup"><span data-stu-id="f60cc-140">spark-listeners</span></span>
    - <span data-ttu-id="f60cc-141">Spark de ouvintes de loganalytics</span><span class="sxs-lookup"><span data-stu-id="f60cc-141">spark-listeners-loganalytics</span></span>

1. <span data-ttu-id="f60cc-142">Execute o Maven **pacote** fase de compilação no seu IDE para compilar os arquivos JAR para cada um desses projetos Java:</span><span class="sxs-lookup"><span data-stu-id="f60cc-142">Execute the Maven **package** build phase in your Java IDE to build the JAR files for each of these projects:</span></span>

    |<span data-ttu-id="f60cc-143">Project</span><span class="sxs-lookup"><span data-stu-id="f60cc-143">Project</span></span>| <span data-ttu-id="f60cc-144">Arquivo JAR</span><span class="sxs-lookup"><span data-stu-id="f60cc-144">JAR file</span></span>|
    |-------|---------|
    |<span data-ttu-id="f60cc-145">trabalhos do Spark</span><span class="sxs-lookup"><span data-stu-id="f60cc-145">spark-jobs</span></span>|<span data-ttu-id="f60cc-146">Spark-trabalhos-1.0-snapshot. jar</span><span class="sxs-lookup"><span data-stu-id="f60cc-146">spark-jobs-1.0-SNAPSHOT.jar</span></span>|
    |<span data-ttu-id="f60cc-147">ouvintes do Spark</span><span class="sxs-lookup"><span data-stu-id="f60cc-147">spark-listeners</span></span>|<span data-ttu-id="f60cc-148">Spark-ouvintes-1.0-snapshot. jar</span><span class="sxs-lookup"><span data-stu-id="f60cc-148">spark-listeners-1.0-SNAPSHOT.jar</span></span>|
    |<span data-ttu-id="f60cc-149">Spark de ouvintes de loganalytics</span><span class="sxs-lookup"><span data-stu-id="f60cc-149">spark-listeners-loganalytics</span></span>|<span data-ttu-id="f60cc-150">Spark-ouvintes-loganalytics-1.0-snapshot. jar</span><span class="sxs-lookup"><span data-stu-id="f60cc-150">spark-listeners-loganalytics-1.0-SNAPSHOT.jar</span></span>|

1. <span data-ttu-id="f60cc-151">Usar a CLI do Databricks do Azure para criar um diretório chamado **dbfs: / databricks/monitoramento-preparo**:</span><span class="sxs-lookup"><span data-stu-id="f60cc-151">Use the Azure Databricks CLI to create a directory named **dbfs:/databricks/monitoring-staging**:</span></span>  

    ```bash
    dbfs mkdirs dbfs:/databricks/monitoring-staging
    ```

1. <span data-ttu-id="f60cc-152">Usar a CLI do Databricks do Azure para copiar **/src/spark-listeners/scripts/listeners.sh** anteriormente no diretório:</span><span class="sxs-lookup"><span data-stu-id="f60cc-152">Use the Azure Databricks CLI to copy **/src/spark-listeners/scripts/listeners.sh** to the directory previously:</span></span>

    ```bash
    dbfs cp ./src/spark-listeners/scripts/listeners.sh dbfs:/databricks/monitoring-staging/listeners.sh
    ```

1. <span data-ttu-id="f60cc-153">Usar a CLI do Databricks do Azure para copiar **/src/spark-listeners/scripts/metrics.properties** para o diretório criado anteriormente:</span><span class="sxs-lookup"><span data-stu-id="f60cc-153">Use the Azure Databricks CLI to copy **/src/spark-listeners/scripts/metrics.properties** to the directory created previously:</span></span>

    ```bash
    dbfs cp <local path to metrics.properties> dbfs:/databricks/monitoring-staging/metrics.properties
    ```

1. <span data-ttu-id="f60cc-154">Usar a CLI do Databricks do Azure para copiar **spark-ouvintes-1.0-snapshot. jar** e **spark-ouvintes-loganalytics-1.0-snapshot. jar** para o diretório criado anteriormente:</span><span class="sxs-lookup"><span data-stu-id="f60cc-154">Use the Azure Databricks CLI to copy **spark-listeners-1.0-SNAPSHOT.jar** and **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** to the directory created previously:</span></span>

    ```bash
    dbfs cp ./src/spark-listeners/target/spark-listeners-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-1.0-SNAPSHOT.jar
    dbfs cp ./src/spark-listeners-loganalytics/target/spark-listeners-loganalytics-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-loganalytics-1.0-SNAPSHOT.jar
    ```

## <a name="create-and-configure-an-azure-databricks-cluster"></a><span data-ttu-id="f60cc-155">Criar e configurar um cluster do Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="f60cc-155">Create and configure an Azure Databricks cluster</span></span>

<span data-ttu-id="f60cc-156">Para criar e configurar um cluster do Databricks do Azure, siga estas etapas:</span><span class="sxs-lookup"><span data-stu-id="f60cc-156">To create and configure an Azure Databricks cluster, follow these steps:</span></span>

1. <span data-ttu-id="f60cc-157">Navegue até seu espaço de trabalho do Azure Databricks no portal do Azure.</span><span class="sxs-lookup"><span data-stu-id="f60cc-157">Navigate to your Azure Databricks workspace in the Azure portal.</span></span>
1. <span data-ttu-id="f60cc-158">Na home page, clique em **novo Cluster**.</span><span class="sxs-lookup"><span data-stu-id="f60cc-158">On the home page, click **New Cluster**.</span></span>
1. <span data-ttu-id="f60cc-159">Insira um nome para o cluster na **nome do Cluster** caixa de texto.</span><span class="sxs-lookup"><span data-stu-id="f60cc-159">Enter a name for your cluster in the **Cluster Name** text box.</span></span>
1. <span data-ttu-id="f60cc-160">No **versão de tempo de execução Databricks** lista suspensa, selecione **4.3** ou maior.</span><span class="sxs-lookup"><span data-stu-id="f60cc-160">In the **Databricks Runtime Version** dropdown, select **4.3** or greater.</span></span>
1. <span data-ttu-id="f60cc-161">Sob **opções avançadas**, clique no **Spark** guia. Insira os seguintes pares de nome-valor na **Spark Config** caixa de texto:</span><span class="sxs-lookup"><span data-stu-id="f60cc-161">Under **Advanced Options**, click on the **Spark** tab. Enter the following name-value pairs in the **Spark Config** text box:</span></span>

    ```
    spark.extraListeners com.databricks.backend.daemon.driver.DBCEventLoggingListener,org.apache.spark.listeners.UnifiedSparkListener
    spark.unifiedListener.sink org.apache.spark.listeners.sink.loganalytics.LogAnalyticsListenerSink
    spark.unifiedListener.logBlockUpdates false
    ```

1. <span data-ttu-id="f60cc-162">Enquanto ainda sob o **Spark** , insira os seguintes valores na **variáveis de ambiente** caixa de texto:</span><span class="sxs-lookup"><span data-stu-id="f60cc-162">While still under the **Spark** tab, enter the following values in the **Environment Variables** text box:</span></span>

    ```
    LOG_ANALYTICS_WORKSPACE_ID=[your Azure Log Analytics workspace ID]
    LOG_ANALYTICS_WORKSPACE_KEY=[your Azure Log Analytics workspace key]
    ```

    ![Captura de tela da interface do usuário do Databricks](./_images/create-cluster1.png)

1. <span data-ttu-id="f60cc-164">Enquanto ainda sob o **opções avançadas** seção, clique no **Scripts de inicialização de** guia.</span><span class="sxs-lookup"><span data-stu-id="f60cc-164">While still under the **Advanced Options** section, click on the **Init Scripts** tab.</span></span>
1. <span data-ttu-id="f60cc-165">No **destino** lista suspensa, selecione **DBFS**.</span><span class="sxs-lookup"><span data-stu-id="f60cc-165">In the **Destination** dropdown, select **DBFS**.</span></span>
1. <span data-ttu-id="f60cc-166">Sob **caminho do Script de inicialização**, insira `dbfs:/databricks/monitoring-staging/listeners.sh`.</span><span class="sxs-lookup"><span data-stu-id="f60cc-166">Under **Init Script Path**, enter `dbfs:/databricks/monitoring-staging/listeners.sh`.</span></span>
1. <span data-ttu-id="f60cc-167">Clique em **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="f60cc-167">Click **Add**.</span></span>

    ![Captura de tela da interface do usuário do Databricks](./_images/create-cluster2.png)

1. <span data-ttu-id="f60cc-169">Clique em **criar Cluster** para criar o cluster.</span><span class="sxs-lookup"><span data-stu-id="f60cc-169">Click **Create Cluster** to create the cluster.</span></span>

## <a name="view-metrics"></a><span data-ttu-id="f60cc-170">Métricas de exibição</span><span class="sxs-lookup"><span data-stu-id="f60cc-170">View metrics</span></span>

<span data-ttu-id="f60cc-171">Depois de concluir essas etapas, o cluster Databricks transmite alguns dados de métrica sobre o cluster para o Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="f60cc-171">After you complete these steps, your Databricks cluster streams some metric data about the cluster itself to Azure Monitor.</span></span> <span data-ttu-id="f60cc-172">Esses dados de log estão disponíveis em seu espaço de trabalho do Log Analytics do Azure com o "Active Directory | Logs personalizados | Esquema de SparkMetric_CL".</span><span class="sxs-lookup"><span data-stu-id="f60cc-172">This log data is available in your Azure Log Analytics workspace under the "Active | Custom Logs | SparkMetric_CL" schema.</span></span> <span data-ttu-id="f60cc-173">Para obter mais informações sobre os tipos de métricas que são registrados em log, consulte [métricas Core](https://metrics.dropwizard.io/4.0.0/manual/core.html) na documentação do Dropwizard.</span><span class="sxs-lookup"><span data-stu-id="f60cc-173">For more information about the types of metrics that are logged, see [Metrics Core](https://metrics.dropwizard.io/4.0.0/manual/core.html) in the Dropwizard documentation.</span></span> <span data-ttu-id="f60cc-174">O tipo de métrica, como o medidor, contador ou histograma, é gravado para o campo de tipo.</span><span class="sxs-lookup"><span data-stu-id="f60cc-174">The metric type, such as gauge, counter, or histogram, is written to the Type field.</span></span>

<span data-ttu-id="f60cc-175">Além disso, a biblioteca de fluxos de eventos no nível do Apache Spark e Spark Structured Streaming métricas de seus trabalhos para o Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="f60cc-175">In addition, the library streams Apache Spark level events and Spark Structured Streaming metrics from your jobs to Azure Monitor.</span></span> <span data-ttu-id="f60cc-176">Você não precisa fazer nenhuma alteração ao código do aplicativo para essas métricas e eventos.</span><span class="sxs-lookup"><span data-stu-id="f60cc-176">You don't need to make any changes to your application code for these events and metrics.</span></span> <span data-ttu-id="f60cc-177">Dados de log de Streaming estruturado do Spark estão disponíveis sob o "Active Directory | Logs personalizados | Esquema de SparkListenerEvent_CL".</span><span class="sxs-lookup"><span data-stu-id="f60cc-177">Spark Structured Streaming log data is available under the "Active | Custom Logs | SparkListenerEvent_CL" schema.</span></span>

![Captura de tela de um espaço de trabalho do Log Analytics](./_images/workspace.png)

<span data-ttu-id="f60cc-179">Para obter mais informações sobre como exibir logs, consulte [exibindo e analisando dados de log no Azure Monitor](/azure/azure-monitor/log-query/portals).</span><span class="sxs-lookup"><span data-stu-id="f60cc-179">For more information about viewing logs, see [Viewing and analyzing log data in Azure Monitor](/azure/azure-monitor/log-query/portals).</span></span>

## <a name="next-steps"></a><span data-ttu-id="f60cc-180">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="f60cc-180">Next steps</span></span>

<span data-ttu-id="f60cc-181">Este artigo descreveu como configurar o cluster para enviar métricas para o Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="f60cc-181">This article described how to configure your cluster to send metrics to Azure Monitor.</span></span> <span data-ttu-id="f60cc-182">O próximo artigo mostra como usar a biblioteca de monitoramento do Azure Databricks para enviar logs e métricas de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="f60cc-182">The next article shows how to use the Azure Databricks Monitoring Library to send application metrics and logs.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="f60cc-183">Enviar logs de aplicativo para o Azure Monitor</span><span class="sxs-lookup"><span data-stu-id="f60cc-183">Send application logs to Azure Monitor</span></span>](./application-logs.md)
