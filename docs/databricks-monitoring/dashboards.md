---
title: Use painéis para visualizar as métricas do Azure Databricks
description: Como implantar um painel do Grafana para monitorar o desempenho no Azure Databricks
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 36fcd93f6ca757e8e750d0fcbbdf0311c08560b0
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887821"
---
# <a name="use-dashboards-to-visualize-azure-databricks-metrics"></a><span data-ttu-id="4c3b5-103">Use painéis para visualizar as métricas do Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="4c3b5-103">Use dashboards to visualize Azure Databricks metrics</span></span>

<span data-ttu-id="4c3b5-104">Este artigo mostra como configurar um painel do Grafana para monitorar trabalhos do Azure Databricks para problemas de desempenho.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-104">This article shows how to set up a Grafana dashboard to monitor Azure Databricks jobs for performance issues.</span></span>

<span data-ttu-id="4c3b5-105">[O Azure Databricks](/azure/azure-databricks/) é uma rápida, poderosa e de colaboração [Apache Spark](https://spark.apache.org/)– com base em serviço de análise que torna mais fácil desenvolver e implantar soluções de inteligência artificial (AI) e análise de big data rapidamente.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-105">[Azure Databricks](/azure/azure-databricks/) is a fast, powerful, and collaborative [Apache Spark](https://spark.apache.org/)–based analytics service that makes it easy to rapidly develop and deploy big data analytics and artificial intelligence (AI) solutions.</span></span> <span data-ttu-id="4c3b5-106">O monitoramento é um componente crítico de operacionais de cargas de trabalho do Databricks do Azure em produção.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-106">Monitoring is a critical component of operating Azure Databricks workloads in production.</span></span> <span data-ttu-id="4c3b5-107">A primeira etapa é reunir métricas em um espaço de trabalho para análise.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-107">The first step is to gather metrics into a workspace for analysis.</span></span> <span data-ttu-id="4c3b5-108">No Azure, é a melhor solução para gerenciamento de dados de log [do Azure Monitor](/azure/azure-monitor/).</span><span class="sxs-lookup"><span data-stu-id="4c3b5-108">In Azure, the best solution for managing log data is [Azure Monitor](/azure/azure-monitor/).</span></span> <span data-ttu-id="4c3b5-109">O Azure Databricks não oferece suporte a enviar dados de log para o Azure monitor, mas uma [biblioteca para essa funcionalidade](https://github.com/mspnp/spark-monitoring) está disponível no [Github](https://github.com).</span><span class="sxs-lookup"><span data-stu-id="4c3b5-109">Azure Databricks does not natively support sending log data to Azure monitor, but a [library for this functionality](https://github.com/mspnp/spark-monitoring) is available in [Github](https://github.com).</span></span>

<span data-ttu-id="4c3b5-110">Essa biblioteca ativa o log de métricas de serviço do Azure Databricks, bem como a estrutura do Apache Spark streaming métricas de consulta de evento.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-110">This library enables logging of Azure Databricks service metrics as well as Apache Spark structure streaming query event metrics.</span></span> <span data-ttu-id="4c3b5-111">Depois de implantar com êxito essa biblioteca a um cluster do Azure Databricks, você pode implantar mais um conjunto de [Grafana](https://granfana.com) painéis que podem ser implantados como parte de seu ambiente de produção.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-111">Once you've successfully deployed this library to an Azure Databricks cluster, you can further deploy a set of [Grafana](https://granfana.com) dashboards that you can deploy as part of your production environment.</span></span>

![Captura de tela do painel](./_images/dashboard-screenshot.png)

## <a name="prequisites"></a><span data-ttu-id="4c3b5-113">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="4c3b5-113">Prequisites</span></span>

<span data-ttu-id="4c3b5-114">Clone o [repositório Github](https://github.com/mspnp/spark-monitoring) e [siga as instruções de implantação](./configure-cluster.md) para criar e configurar o registro em log do Azure Monitor para a biblioteca do Azure Databricks enviar logs ao seu espaço de trabalho do Log Analytics do Azure.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-114">Clone the [Github repository](https://github.com/mspnp/spark-monitoring) and [follow the deployment instructions](./configure-cluster.md) to build and configure the Azure Monitor logging for Azure Databricks library to send logs to your Azure Log Analytics workspace.</span></span>

## <a name="deploy-the-azure-log-analytics-workspace"></a><span data-ttu-id="4c3b5-115">Implantar o espaço de trabalho do Log Analytics do Azure</span><span class="sxs-lookup"><span data-stu-id="4c3b5-115">Deploy the Azure Log Analytics workspace</span></span>

<span data-ttu-id="4c3b5-116">Para implantar o espaço de trabalho do Log Analytics do Azure, siga estas etapas:</span><span class="sxs-lookup"><span data-stu-id="4c3b5-116">To deploy the Azure Log Analytics workspace, follow these steps:</span></span>

1. <span data-ttu-id="4c3b5-117">Navegue até o `/perftools/deployment/loganalytics` directory.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-117">Navigate to the `/perftools/deployment/loganalytics` directory.</span></span>
1. <span data-ttu-id="4c3b5-118">Implantar o **logAnalyticsDeploy.json** modelo do Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-118">Deploy the **logAnalyticsDeploy.json** Azure Resource Manager template.</span></span> <span data-ttu-id="4c3b5-119">Para obter mais informações sobre como implantar modelos do Resource Manager, consulte [implantar recursos com modelos do Resource Manager e a CLI do Azure][rm-cli].</span><span class="sxs-lookup"><span data-stu-id="4c3b5-119">For more information about deploying Resource Manager templates, see [Deploy resources with Resource Manager templates and Azure CLI][rm-cli].</span></span> <span data-ttu-id="4c3b5-120">O modelo tem os seguintes parâmetros:</span><span class="sxs-lookup"><span data-stu-id="4c3b5-120">The template has the following parameters:</span></span>

    * <span data-ttu-id="4c3b5-121">**local**: A região em que o espaço de trabalho do Log Analytics e os painéis são implantados.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-121">**location**: The region where the Log Analytics workspace and dashboards are deployed.</span></span>
    * <span data-ttu-id="4c3b5-122">**serviceTier**: Tipo de preço do espaço de trabalho do trocador.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-122">**serviceTier**: Rhe workspace pricing tier.</span></span> <span data-ttu-id="4c3b5-123">Ver [aqui] [ sku] para obter uma lista de valores válidos.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-123">See [here][sku] for a list of valid values.</span></span>
    * <span data-ttu-id="4c3b5-124">**dataRetention** (opcional): O número de dias de dados de log é mantido no espaço de trabalho do Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-124">**dataRetention** (optional): The number of days log data is retained in the Log Analytics workspace.</span></span> <span data-ttu-id="4c3b5-125">O valor padrão é de 30 dias.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-125">The default value is 30 days.</span></span> <span data-ttu-id="4c3b5-126">Se o tipo de preço é `Free`, a retenção de dados deve ser 7 dias.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-126">If the pricing tier is `Free`, the data retention must be 7 days.</span></span>
    * <span data-ttu-id="4c3b5-127">**WorkspaceName** (opcional): Um nome para o espaço de trabalho.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-127">**workspaceName** (optional): A name for the workspace.</span></span> <span data-ttu-id="4c3b5-128">Se não for especificado, o modelo gera um nome.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-128">If not specified, the template generates a name.</span></span>

    ```bash
    az group deployment create --resource-group <resource-group-name> --template-file logAnalyticsDeploy.json --parameters location='East US' serviceTier='Standalone'
    ```

<span data-ttu-id="4c3b5-129">Este modelo cria o espaço de trabalho e também cria um conjunto de consultas predefinidas que são usadas pelo painel.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-129">This template creates the workspace and also creates a set of predefined queries that are used by by dashboard.</span></span>

## <a name="deploy-grafana-in-a-virtual-machine"></a><span data-ttu-id="4c3b5-130">Implantar o Grafana em uma máquina virtual</span><span class="sxs-lookup"><span data-stu-id="4c3b5-130">Deploy Grafana in a virtual machine</span></span>

<span data-ttu-id="4c3b5-131">Grafana é um projeto de código-fonte aberto que pode ser implantado para visualizar as métricas de série de tempo armazenadas em seu espaço de trabalho do Log Analytics do Azure usando o plug-in do Grafana para o Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-131">Grafana is an open source project you can deploy to visualize the time series metrics stored in your Azure Log Analytics workspace using the Grafana plugin for Azure Monitor.</span></span> <span data-ttu-id="4c3b5-132">Grafana é executado em uma máquina virtual (VM) e requer uma conta de armazenamento, rede virtual e outros recursos.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-132">Grafana executes on a virtual machine (VM) and requires a storage account, virtual network, and other resources.</span></span> <span data-ttu-id="4c3b5-133">Para implantar uma máquina virtual com o bitnami certified Grafana imagem e os recursos associados, siga estas etapas:</span><span class="sxs-lookup"><span data-stu-id="4c3b5-133">To deploy a virtual machine with the bitnami certified Grafana image and associated resources, follow these steps:</span></span>

1. <span data-ttu-id="4c3b5-134">Use a CLI do Azure para aceitar os termos de imagem do Azure Marketplace para o Grafana.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-134">Use the Azure CLI to accept the Azure Marketplace image terms for Grafana.</span></span>

    ```bash
    az vm image accept-terms --publisher bitnami --offer grafana --plan default
    ```

1. <span data-ttu-id="4c3b5-135">Navegue até o `/spark-monitoring/perftools/deployment/grafana` diretório em sua cópia local do repositório GitHub.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-135">Navigate to the `/spark-monitoring/perftools/deployment/grafana` directory in your local copy of the GitHub repo.</span></span>
1. <span data-ttu-id="4c3b5-136">Implantar o **grafanaDeploy.json** modelo do Resource Manager da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="4c3b5-136">Deploy the **grafanaDeploy.json** Resource Manager template as follows:</span></span>

    ```bash
    export DATA_SOURCE="https://raw.githubusercontent.com/mspnp/spark-monitoring/master/perftools/deployment/grafana/AzureDataSource.sh"
    az group deployment create \
        --resource-group <resource-group-name> \
        --template-file grafanaDeploy.json \
        --parameters adminPass='<vm password>' dataSource=$DATA_SOURCE
    ```

<span data-ttu-id="4c3b5-137">Depois que a implantação for concluída, a imagem bitnami do Grafana é instalada na máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-137">Once the deployment is complete, the bitnami image of Grafana is installed on the virtual machine.</span></span>

## <a name="update-the-grafana-password"></a><span data-ttu-id="4c3b5-138">Atualizar a senha do Grafana</span><span class="sxs-lookup"><span data-stu-id="4c3b5-138">Update the Grafana password</span></span>

<span data-ttu-id="4c3b5-139">Como parte do processo de instalação, o script de instalação do Grafana gera uma senha temporária para o **admin** usuário.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-139">As part of the setup process, the Grafana installation script outputs a temporary password for the **admin** user.</span></span> <span data-ttu-id="4c3b5-140">Você precisa dessa senha temporária para entrar.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-140">You need this temporary password to sign in.</span></span> <span data-ttu-id="4c3b5-141">Para obter a senha temporária, siga estas etapas:</span><span class="sxs-lookup"><span data-stu-id="4c3b5-141">To obtain the temporary password, follow these steps:</span></span>  

1. <span data-ttu-id="4c3b5-142">Faça logon no Portal do Azure.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-142">Log in to the Azure portal.</span></span>  
1. <span data-ttu-id="4c3b5-143">Selecione o grupo de recursos onde os recursos foram implantados.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-143">Select the resource group where the resources were deployed.</span></span>
1. <span data-ttu-id="4c3b5-144">Selecione a VM em que o Grafana foi instalado.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-144">Select the VM where Grafana was installed.</span></span> <span data-ttu-id="4c3b5-145">Se você usou o nome do parâmetro padrão no modelo de implantação, o nome da VM seja prefixado com **sparkmonitoring-vm-grafana**.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-145">If you used the default parameter name in the deployment template, the VM name is prefaced with **sparkmonitoring-vm-grafana**.</span></span>
1. <span data-ttu-id="4c3b5-146">No **suporte + solução de problemas** seção, clique em **diagnóstico de inicialização** para abrir a página de diagnóstico de inicialização.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-146">In the **Support + troubleshooting** section, click **Boot diagnostics** to open the boot diagnostics page.</span></span>
1. <span data-ttu-id="4c3b5-147">Clique em **log Serial** na página de diagnóstico de inicialização.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-147">Click **Serial log** on the boot diagnostics page.</span></span>
1. <span data-ttu-id="4c3b5-148">Procure a cadeia de caracteres a seguir: "Definir senha de aplicativo Bitnami como".</span><span class="sxs-lookup"><span data-stu-id="4c3b5-148">Search for the following string: "Setting Bitnami application password to".</span></span>
1. <span data-ttu-id="4c3b5-149">Copie a senha para um local seguro.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-149">Copy the password to a safe location.</span></span>

<span data-ttu-id="4c3b5-150">Em seguida, altere a senha de administrador do Grafana seguindo estas etapas:</span><span class="sxs-lookup"><span data-stu-id="4c3b5-150">Next, change the Grafana administrator password by following these steps:</span></span>

1. <span data-ttu-id="4c3b5-151">No portal do Azure, selecione a VM e clique em **visão geral**.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-151">In the Azure portal, select the VM and click **Overview**.</span></span>
1. <span data-ttu-id="4c3b5-152">Copie o endereço IP público.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-152">Copy the public IP address.</span></span>
1. <span data-ttu-id="4c3b5-153">Abra um navegador da web e navegue até a seguinte URL: `http://<IP addresss>:3000`.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-153">Open a web browser and navigate to the following URL: `http://<IP addresss>:3000`.</span></span>
1. <span data-ttu-id="4c3b5-154">O log do Grafana na tela, insira **admin** para o nome de usuário e use a senha do Grafana das etapas anteriores.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-154">At the Grafana log in screen, enter **admin** for the user name, and use the Grafana password from the previous steps.</span></span>
1. <span data-ttu-id="4c3b5-155">Depois de conectado, selecione **configuração** (o ícone de engrenagem).</span><span class="sxs-lookup"><span data-stu-id="4c3b5-155">Once logged in, select **Configuration** (the gear icon).</span></span>
1. <span data-ttu-id="4c3b5-156">Selecione **administrador do servidor**.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-156">Select **Server Admin**.</span></span>
1. <span data-ttu-id="4c3b5-157">Sobre o **os usuários** guia, selecione o **admin** logon.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-157">On the **Users** tab, select the **admin** login.</span></span>
1. <span data-ttu-id="4c3b5-158">Atualize a senha.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-158">Update the password.</span></span>

## <a name="create-an-azure-monitor-data-source"></a><span data-ttu-id="4c3b5-159">Criar uma fonte de dados do Azure Monitor</span><span class="sxs-lookup"><span data-stu-id="4c3b5-159">Create an Azure Monitor data source</span></span>

1. <span data-ttu-id="4c3b5-160">Crie um serviço principal que permite que o Grafana para gerenciar o acesso ao seu espaço de trabalho do Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-160">Create a service principal that allows Grafana to manage access to your Log Analytics workspace.</span></span> <span data-ttu-id="4c3b5-161">Para obter mais informações, consulte [criar uma entidade de serviço com a CLI do Azure](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)</span><span class="sxs-lookup"><span data-stu-id="4c3b5-161">For more information, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)</span></span>

    ```bash
    az ad sp create-for-rbac --name http://<service principal name> --role "Log Analytics Reader"
    ```

1. <span data-ttu-id="4c3b5-162">Observe os valores para appId, senha e locatário na saída deste comando:</span><span class="sxs-lookup"><span data-stu-id="4c3b5-162">Note the values for appId, password, and tenant in the output from this command:</span></span>

    ```json
    {
        "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "displayName": "azure-cli-2019-03-27-00-33-39",
        "name": "http://<service principal name>",
        "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. <span data-ttu-id="4c3b5-163">Faça logon no Grafana conforme descrito anteriormente.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-163">Log into Grafana as described earlier.</span></span> <span data-ttu-id="4c3b5-164">Selecione **Configuration** (o ícone de engrenagem) e, em seguida **fontes de dados**.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-164">Select **Configuration** (the gear icon) and then **Data Sources**.</span></span>
1. <span data-ttu-id="4c3b5-165">No **fontes de dados** , clique em **adicionar fonte de dados**.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-165">In the **Data Sources** tab, click **Add data source**.</span></span>
1. <span data-ttu-id="4c3b5-166">Selecione **do Azure Monitor** como o tipo de fonte de dados.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-166">Select **Azure Monitor** as the data source type.</span></span>
1. <span data-ttu-id="4c3b5-167">No **as configurações** , digite um nome para a fonte de dados no **nome** caixa de texto.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-167">In the **Settings** section, enter a name for the data source in the **Name** textbox.</span></span>
1. <span data-ttu-id="4c3b5-168">No **detalhes de API do Azure Monitor** seção, insira as seguintes informações:</span><span class="sxs-lookup"><span data-stu-id="4c3b5-168">In the **Azure Monitor API Details** section, enter the following information:</span></span>

    * <span data-ttu-id="4c3b5-169">ID da Assinatura: Sua ID da assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-169">Subscription Id: Your Azure subscription ID.</span></span>
    * <span data-ttu-id="4c3b5-170">Id do locatário: A ID de locatário anterior.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-170">Tenant Id: The tenant ID from earlier.</span></span>
    * <span data-ttu-id="4c3b5-171">Id do cliente: O valor de "appId" anterior.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-171">Client Id: The value of "appId" from earlier.</span></span>
    * <span data-ttu-id="4c3b5-172">Segredo do cliente: O valor de "password" anterior.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-172">Client Secret: The value of "password" from earlier.</span></span>

1. <span data-ttu-id="4c3b5-173">No **detalhes de API do Azure Log Analytics** seção, verifique os **mesmo detalhes como a API do Azure Monitor** caixa de seleção.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-173">In the **Azure Log Analytics API Details** section, check the **Same Details as Azure Monitor API** checkbox.</span></span>
1. <span data-ttu-id="4c3b5-174">Clique em **salvar e testar**.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-174">Click **Save & Test**.</span></span> <span data-ttu-id="4c3b5-175">Se a fonte de dados do Log Analytics está configurada corretamente, uma mensagem de êxito será exibida.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-175">If the Log Analytics data source is correctly configured, a success message is displayed.</span></span>

## <a name="create-the-dashboard"></a><span data-ttu-id="4c3b5-176">Criar painel</span><span class="sxs-lookup"><span data-stu-id="4c3b5-176">Create the dashboard</span></span>

<span data-ttu-id="4c3b5-177">Crie os painéis no Grafana seguindo estas etapas:</span><span class="sxs-lookup"><span data-stu-id="4c3b5-177">Create the dashboards in Grafana by following these steps:</span></span>

1. <span data-ttu-id="4c3b5-178">Navegue até o `/perftools/dashboards/grafana` diretório em sua cópia local do repositório GitHub.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-178">Navigate to the `/perftools/dashboards/grafana` directory in your local copy of the GitHub repo.</span></span>
1. <span data-ttu-id="4c3b5-179">Execute o seguinte script:</span><span class="sxs-lookup"><span data-stu-id="4c3b5-179">Run the following script:</span></span>

    ```bash
    export WORKSPACE=<your Azure Log Analytics workspace ID>
    export LOGTYPE=SparkListenerEvent_CL

    sh DashGen.sh
    ```

    <span data-ttu-id="4c3b5-180">A saída do script é um arquivo chamado **SparkMonitoringDash.json**.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-180">The output from the script is a file named **SparkMonitoringDash.json**.</span></span>

1. <span data-ttu-id="4c3b5-181">Retorne ao painel do Grafana e selecione **criar** (o ícone de adição).</span><span class="sxs-lookup"><span data-stu-id="4c3b5-181">Return to the Grafana dashboard and select **Create** (the plus icon).</span></span>
1. <span data-ttu-id="4c3b5-182">Selecione **Importar**.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-182">Select **Import**.</span></span>
1. <span data-ttu-id="4c3b5-183">Clique em **carregar arquivo. JSON**.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-183">Click **Upload .json File**.</span></span>
1. <span data-ttu-id="4c3b5-184">Selecione o **SparkMonitoringDash.json** arquivo criado na etapa 2.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-184">Select the **SparkMonitoringDash.json** file created in step 2.</span></span>
1. <span data-ttu-id="4c3b5-185">No **opções** seção, no **ALA**, selecione a fonte de dados do Azure Monitor criada anteriormente.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-185">In the **Options** section, under **ALA**, select the Azure Monitor data source created earlier.</span></span>
1. <span data-ttu-id="4c3b5-186">Clique em **Importar**.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-186">Click **Import**.</span></span>

## <a name="visualizations-in-the-dashboards"></a><span data-ttu-id="4c3b5-187">Visualizações em dashboards</span><span class="sxs-lookup"><span data-stu-id="4c3b5-187">Visualizations in the dashboards</span></span>

<span data-ttu-id="4c3b5-188">O Azure Log Analytics e o Grafana painéis incluem um conjunto de visualizações de série temporal.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-188">Both the Azure Log Analytics and Grafana dashboards include a set of time-series visualizations.</span></span> <span data-ttu-id="4c3b5-189">Cada gráfico é um gráfico de série temporal de dados de métrica relacionados a um Apache Spark [trabalho](https://spark.apache.org/docs/latest/job-scheduling.html), etapas de trabalho e tarefas que compõem cada estágio.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-189">Each graph is time-series plot of metric data related to an Apache Spark [job](https://spark.apache.org/docs/latest/job-scheduling.html), stages of the job, and tasks that make up each stage.</span></span>

<span data-ttu-id="4c3b5-190">As visualizações são da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="4c3b5-190">The visualizations are as follows:</span></span>

### <a name="job-latency"></a><span data-ttu-id="4c3b5-191">Latência do trabalho</span><span class="sxs-lookup"><span data-stu-id="4c3b5-191">Job latency</span></span>

<span data-ttu-id="4c3b5-192">Esta visualização mostra uma latência de execução para um trabalho, que é um modo de exibição não refinado sobre o desempenho geral de um trabalho.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-192">This visualization shows execution latency for a job, which is a coarse view on the overall peformance of a job.</span></span> <span data-ttu-id="4c3b5-193">Exibe a duração da execução do trabalho do início até a conclusão.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-193">Displays the job execution duration from start to completion.</span></span> <span data-ttu-id="4c3b5-194">Observe que a hora de início do trabalho não é o mesmo que a hora de envio do trabalho.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-194">Note that the job start time is not the same as the job submission time.</span></span> <span data-ttu-id="4c3b5-195">Latência é representada como percentuais (10%, 30%, 50%, 90%.) de execução do trabalho indexada por ID do cluster e a ID do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-195">Latency is represented as percentiles (10%, 30%, 50%, 90%) of job execution indexed by cluster ID and application ID.</span></span>

### <a name="stage-latency"></a><span data-ttu-id="4c3b5-196">Latência de estágio</span><span class="sxs-lookup"><span data-stu-id="4c3b5-196">Stage latency</span></span>

<span data-ttu-id="4c3b5-197">A visualização mostra a latência de cada estágio por cluster, por aplicativo e por estágio individual.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-197">The visualization shows the latency of each stage per cluster, per application, and per individual stage.</span></span> <span data-ttu-id="4c3b5-198">Essa visualização é útil para identificar um estágio específico que está sendo executado lentamente.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-198">This visualization is useful for identifying a particular stage that is running slowly.</span></span>

### <a name="task-latency"></a><span data-ttu-id="4c3b5-199">Latência de tarefa</span><span class="sxs-lookup"><span data-stu-id="4c3b5-199">Task latency</span></span>

<span data-ttu-id="4c3b5-200">Esta visualização mostra uma latência de execução de tarefa.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-200">This visualization shows task execution latency.</span></span> <span data-ttu-id="4c3b5-201">Latência é representada como um percentual da execução de tarefa por cluster, o nome do estágio e o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-201">Latency is represented as a percentile of task execution per cluster, stage name, and application.</span></span>

### <a name="sum-task-execution-per-host"></a><span data-ttu-id="4c3b5-202">Execução da tarefa soma por host</span><span class="sxs-lookup"><span data-stu-id="4c3b5-202">Sum Task Execution per host</span></span>

<span data-ttu-id="4c3b5-203">Esta visualização mostra a soma da latência de execução de tarefa por execução em um cluster de host.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-203">This visualization shows the sum of task execution latency per host running on a cluster.</span></span> <span data-ttu-id="4c3b5-204">Exibindo a latência de execução de tarefa por host identifica os hosts que têm muito mais alta latência geral de tarefa que outros hosts.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-204">Viewing task execution latency per host identifies hosts that have much higher overall task latency than other hosts.</span></span> <span data-ttu-id="4c3b5-205">Isso pode significar que as tarefas foram ineficiente ou sem uniformidade distribuídas aos hosts.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-205">This may mean that tasks have been inefficiently or unevenly distributed to hosts.</span></span>

### <a name="task-metrics"></a><span data-ttu-id="4c3b5-206">Métricas de tarefa</span><span class="sxs-lookup"><span data-stu-id="4c3b5-206">Task metrics</span></span>

<span data-ttu-id="4c3b5-207">Esta visualização mostra um conjunto de métricas de execução para a execução de uma determinada tarefa.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-207">This visualization shows a set of the execution metrics for a given task's execution.</span></span> <span data-ttu-id="4c3b5-208">Essas métricas incluem o tamanho e a duração de uma ordem aleatória de dados, duração das operações de serialização e desserialização e outros.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-208">These metrics include the size and duration of a data shuffle, duration of serialization and deserialization operations, and others.</span></span> <span data-ttu-id="4c3b5-209">Para o conjunto completo de métricas, exiba a consulta do Log Analytics para o painel.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-209">For the full set of metrics, view the Log Analytics query for the panel.</span></span> <span data-ttu-id="4c3b5-210">Essa visualização é útil para entender as operações que compõem uma tarefa e o consumo de recursos de identificação de cada operação.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-210">This visualization is useful for understanding the operations that make up a task and identifying resource consumption of each operation.</span></span> <span data-ttu-id="4c3b5-211">Picos no gráfico representam operações dispendiosas que devem ser investigadas.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-211">Spikes in the graph represent costly operations that should be investigated.</span></span>

### <a name="cluster-throughput"></a><span data-ttu-id="4c3b5-212">Taxa de transferência de cluster</span><span class="sxs-lookup"><span data-stu-id="4c3b5-212">Cluster throughput</span></span>

<span data-ttu-id="4c3b5-213">Essa visualização é uma exibição de alto nível de itens de trabalho indexado pelo cluster e aplicativo para representar a quantidade de trabalho realizado por cluster e aplicativo.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-213">This visualization is a high level view of work items indexed by cluster and application to represent the amount of work done per cluster and application.</span></span> <span data-ttu-id="4c3b5-214">Ele mostra o número de trabalhos, tarefas e etapas concluídas por cluster, o aplicativo e o estágio em incrementos de um minuto.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-214">It shows the number of jobs, tasks, and stages completed per cluster, application, and stage in one minute increments.</span></span> 

### <a name="streaming-throughputlatency"></a><span data-ttu-id="4c3b5-215">Streaming de taxa de transferência/latência</span><span class="sxs-lookup"><span data-stu-id="4c3b5-215">Streaming Throughput/Latency</span></span>

<span data-ttu-id="4c3b5-216">Este visualzation está relacionado às métricas associadas a uma consulta de streaming estruturada.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-216">This visualzation is related to the metrics associated with a structured streaming query.</span></span> <span data-ttu-id="4c3b5-217">Os grafos mostra o número de linhas de entrada por segundo e o número de linhas processadas por segundo.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-217">The graphs shows the number of input rows per second and the number of rows processed per second.</span></span> <span data-ttu-id="4c3b5-218">As métricas de streaming também são representadas por aplicativo.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-218">The streaming metrics are also represented per application.</span></span> <span data-ttu-id="4c3b5-219">Essas métricas são enviadas quando o evento OnQueryProgress é gerado conforme a consulta de streaming estruturada é processada e a visualização representa streaming latência como a quantidade de tempo, em milissegundos, necessário para executar um lote de consultas.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-219">These metrics are sent when the OnQueryProgress event is generated as the structured streaming query is processed and the visualization represents streaming latency as the amount of time, in milliseconds, taken to execute a query batch.</span></span>

### <a name="resource-consumption-per-executor"></a><span data-ttu-id="4c3b5-220">Consumo de recursos por executor</span><span class="sxs-lookup"><span data-stu-id="4c3b5-220">Resource consumption per executor</span></span>

<span data-ttu-id="4c3b5-221">Em seguida, é um conjunto de visualizações para mostrar painel de determinado tipo de recurso e como eles são consumidos por executor em cada cluster.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-221">Next is a set of visualizations for the dashboard show the particular type of resource and how it is consumed per executor on each cluster.</span></span> <span data-ttu-id="4c3b5-222">Estas visualizações ajudam a identificar exceções no consumo de recursos por executor.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-222">These visualizations help identify outliers in resource consumption per executor.</span></span> <span data-ttu-id="4c3b5-223">Por exemplo, se a alocação de trabalho para um executor específico estiverem distorcida, consumo de recursos será elevado em relação a outros executores em execução no cluster.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-223">For example, if the work allocation for a particular executor is skewed, resource consumption will be elevated in relation to other executors running on the cluster.</span></span> <span data-ttu-id="4c3b5-224">Isso pode ser identificado por picos no consumo de recursos para um executor.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-224">This can be identified by spikes in the resource consumption for an executor.</span></span>

### <a name="executor-compute-time-metrics"></a><span data-ttu-id="4c3b5-225">Métricas de tempo de computação do executor</span><span class="sxs-lookup"><span data-stu-id="4c3b5-225">Executor compute time metrics</span></span>

<span data-ttu-id="4c3b5-226">Em seguida, é um conjunto de visualizações para o dashboard que mostram a proporção de executor serializar tempo, desserializar o tempo, o tempo de CPU e tempo de máquina virtual Java para o tempo de computação geral executor.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-226">Next is a set of visualizations for the dashboard that show the ratio of executor serialize time, deserialize time, CPU time, and Java virtual machine time to overall executor compute time.</span></span> <span data-ttu-id="4c3b5-227">Isso demonstra visualmente quanto cada uma dessas quatro métricas estão contribuindo para o executor geral de processamento.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-227">This demonstrates visually how much each of these four metrics are contributing to overall executor processing.</span></span>

### <a name="shuffle-metrics"></a><span data-ttu-id="4c3b5-228">Métricas de ordem aleatória</span><span class="sxs-lookup"><span data-stu-id="4c3b5-228">Shuffle metrics</span></span>

<span data-ttu-id="4c3b5-229">O conjunto final de Mostrar visualizações de dados em ordem aleatória métricas associadas a uma consulta de streaming estruturada em todos os executores.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-229">The final set of visualizations show the data shuffle metrics associated with a structured streaming query across all executors.</span></span> <span data-ttu-id="4c3b5-230">Isso inclui a ordem aleatória de bytes de leitura, em ordem aleatória de bytes gravados, embaralhar memória e uso do disco em consultas em que o sistema de arquivos é usado.</span><span class="sxs-lookup"><span data-stu-id="4c3b5-230">These include shuffle bytes read, shuffle bytes written, shuffle memory, and disk usage in queries where the file system is used.</span></span>

## <a name="next-steps"></a><span data-ttu-id="4c3b5-231">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="4c3b5-231">Next steps</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="4c3b5-232">Solucionar problemas de gargalos de desempenho</span><span class="sxs-lookup"><span data-stu-id="4c3b5-232">Troubleshoot performance bottlenecks</span></span>](./performance-troubleshooting.md)

<!-- links -->

[rm-cli]: /azure/azure-resource-manager/resource-group-template-deploy-cli
[sku]: /azure/templates/Microsoft.OperationalInsights/2015-11-01-preview/workspaces#sku-object