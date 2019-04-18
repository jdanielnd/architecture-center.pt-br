---
title: Compilar uma API de recomendação em tempo real no Azure
description: Use o aprendizado de máquina para automatizar recomendações usando o Azure Databricks e as DSVM (Máquinas Virtuais de Ciência de Dados) do Azure para treinar um modelo no Azure.
author: njray
ms.date: 12/12/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: c7e7423da11667c90d53247c2c5303a8fbd1a76a
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640151"
---
# <a name="build-a-real-time-recommendation-api-on-azure"></a><span data-ttu-id="9e524-103">Compilar uma API de recomendação em tempo real no Azure</span><span class="sxs-lookup"><span data-stu-id="9e524-103">Build a real-time recommendation API on Azure</span></span>

<span data-ttu-id="9e524-104">Esta arquitetura de referência mostra como treinar um modelo de recomendação usando o Azure Databricks e implantá-lo como uma API usando o Azure Cosmos DB, o Azure Machine Learning e o AKS (Serviço de Kubernetes do Azure).</span><span class="sxs-lookup"><span data-stu-id="9e524-104">This reference architecture shows how to train a recommendation model using Azure Databricks and deploy it as an API by using Azure Cosmos DB, Azure Machine Learning, and Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="9e524-105">Esta arquitetura pode ser adaptada para a maioria dos cenários de mecanismo de recomendação, incluindo recomendações de produtos, filmes e notícias.</span><span class="sxs-lookup"><span data-stu-id="9e524-105">This architecture can be generalized for most recommendation engine scenarios, including recommendations for products, movies, and news.</span></span>

<span data-ttu-id="9e524-106">Há uma implantação de referência para essa arquitetura de referência disponível no [GitHub][als-example].</span><span class="sxs-lookup"><span data-stu-id="9e524-106">A reference implementation for this architecture is available on [GitHub][als-example].</span></span>

![Arquitetura de um modelo de machine learning para treinamento de recomendações de filmes](./_images/recommenders-architecture.png)

<span data-ttu-id="9e524-108">**Cenário**: Uma organização de mídia deseja fornecer recomendações de vídeos ou filmes a seus usuários.</span><span class="sxs-lookup"><span data-stu-id="9e524-108">**Scenario**: A media organization wants to provide movie or video recommendations to its users.</span></span> <span data-ttu-id="9e524-109">Ao fornecer recomendações personalizadas, a organização atende a várias metas de negócios, incluindo maior taxa de cliques, maior participação no site e maior satisfação do usuário.</span><span class="sxs-lookup"><span data-stu-id="9e524-109">By providing personalized recommendations, the organization meets several business goals, including increased click-through rates, increased engagement on site, and higher user satisfaction.</span></span>

<span data-ttu-id="9e524-110">Esta arquitetura de referência é para treinar e implantar uma API de serviço de recomendação em tempo real que pode recomendar os 10 melhores filmes para cada usuário.</span><span class="sxs-lookup"><span data-stu-id="9e524-110">This reference architecture is for training and deploying a real-time recommender service API that can provide the top 10 movie recommendations for a given user.</span></span>

<span data-ttu-id="9e524-111">O fluxo de dados para o modelo de recomendação é o seguinte:</span><span class="sxs-lookup"><span data-stu-id="9e524-111">The data flow for this recommendation model is as follows:</span></span>

1. <span data-ttu-id="9e524-112">Acompanhar comportamentos do usuário.</span><span class="sxs-lookup"><span data-stu-id="9e524-112">Track user behaviors.</span></span> <span data-ttu-id="9e524-113">Por exemplo, um serviço de back-end pode registrar quando um usuário classifica um filme ou clica em um artigo de notícias ou produto.</span><span class="sxs-lookup"><span data-stu-id="9e524-113">For example, a backend service might log when a user rates a movie or clicks a product or news article.</span></span>

2. <span data-ttu-id="9e524-114">Carregar os dados no Azure Databricks de uma [fonte de dados][data-source] disponível.</span><span class="sxs-lookup"><span data-stu-id="9e524-114">Load the data into Azure Databricks from an available [data source][data-source].</span></span>

3. <span data-ttu-id="9e524-115">Preparar os dados e dividi-los em conjuntos de treinamento e teste para treinar o modelo.</span><span class="sxs-lookup"><span data-stu-id="9e524-115">Prepare the data and split it into training and testing sets to train the model.</span></span> <span data-ttu-id="9e524-116">([Este guia][guide] descreve opções para dividir os dados.)</span><span class="sxs-lookup"><span data-stu-id="9e524-116">([This guide][guide] describes options for splitting data.)</span></span>

4. <span data-ttu-id="9e524-117">Ajustar o modelo [Filtragem Colaborativa do Spark][als] aos dados.</span><span class="sxs-lookup"><span data-stu-id="9e524-117">Fit the [Spark Collaborative Filtering][als] model to the data.</span></span>

5. <span data-ttu-id="9e524-118">Avaliar a qualidade do modelo usando métricas de classificação e priorização.</span><span class="sxs-lookup"><span data-stu-id="9e524-118">Evaluate the quality of the model using rating and ranking metrics.</span></span> <span data-ttu-id="9e524-119">([Este guia][eval-guide] fornece detalhes sobre as métricas com as quais você pode avaliar suas recomendações.)</span><span class="sxs-lookup"><span data-stu-id="9e524-119">([This guide][eval-guide] provides details about the metrics you can evaluate your recommender on.)</span></span>

6. <span data-ttu-id="9e524-120">Calcular previamente as 10 principais recomendações por usuário e armazená-las como cache no Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="9e524-120">Precompute the top 10 recommendations per user and store as a cache in Azure Cosmos DB.</span></span>

7. <span data-ttu-id="9e524-121">Implantar um serviço de API no AKS usando as APIs do Azure Machine Learning para colocar a API em contêineres e implantá-la.</span><span class="sxs-lookup"><span data-stu-id="9e524-121">Deploy an API service to AKS using the Azure Machine Learning APIs to containerize and deploy the API.</span></span>

8. <span data-ttu-id="9e524-122">Quando o serviço de back-end recebe uma solicitação de um usuário, chamar a API de recomendações hospedada no AKS para obter as 10 principais recomendações e exibi-las para o usuário.</span><span class="sxs-lookup"><span data-stu-id="9e524-122">When the backend service gets a request from a user, call the recommendations API hosted in AKS to get the top 10 recommendations and display them to the user.</span></span>

## <a name="architecture"></a><span data-ttu-id="9e524-123">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="9e524-123">Architecture</span></span>

<span data-ttu-id="9e524-124">Esta arquitetura é formada pelos seguintes componentes:</span><span class="sxs-lookup"><span data-stu-id="9e524-124">This architecture consists of the following components:</span></span>

<span data-ttu-id="9e524-125">[Azure Databricks][databricks].</span><span class="sxs-lookup"><span data-stu-id="9e524-125">[Azure Databricks][databricks].</span></span> <span data-ttu-id="9e524-126">O Databricks é um ambiente de desenvolvimento usado para preparar os dados de entrada e treinar o modelo de recomendação em um cluster Spark.</span><span class="sxs-lookup"><span data-stu-id="9e524-126">Databricks is a development environment used to prepare input data and train the recommender model on a Spark cluster.</span></span> <span data-ttu-id="9e524-127">O Azure Databricks também fornece um espaço de trabalho interativo para ser executado e colaborar em notebooks em caso de tarefas de processamento de dados ou de aprendizado de máquina.</span><span class="sxs-lookup"><span data-stu-id="9e524-127">Azure Databricks also provides an interactive workspace to run and collaborate on notebooks for any data processing or machine learning tasks.</span></span>

<span data-ttu-id="9e524-128">[AKS][aks] (Serviço de Kubernetes do Azure).</span><span class="sxs-lookup"><span data-stu-id="9e524-128">[Azure Kubernetes Service][aks] (AKS).</span></span> <span data-ttu-id="9e524-129">O AKS é usado para implantar e operacionalizar uma API de serviço do modelo de machine learning em um cluster do Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="9e524-129">AKS is used to deploy and operationalize a machine learning model service API on a Kubernetes cluster.</span></span> <span data-ttu-id="9e524-130">O AKS hospeda o modelo em contêineres, fornecendo escalabilidade que atende a seus requisitos de taxa de transferência, gerenciamento de identidade e de acesso, registro em log e monitoramento de integridade.</span><span class="sxs-lookup"><span data-stu-id="9e524-130">AKS hosts the containerized model, providing scalability that meets your throughput requirements, identity and access management, and logging and health monitoring.</span></span>

<span data-ttu-id="9e524-131">[Azure Cosmos DB][cosmosdb].</span><span class="sxs-lookup"><span data-stu-id="9e524-131">[Azure Cosmos DB][cosmosdb].</span></span> <span data-ttu-id="9e524-132">O Cosmos DB é um serviço de banco de dados distribuído globalmente usado para armazenar os 10 principais filmes recomendados para cada usuário.</span><span class="sxs-lookup"><span data-stu-id="9e524-132">Cosmos DB is a globally distributed database service used to store the top 10 recommended movies for each user.</span></span> <span data-ttu-id="9e524-133">O Azure Cosmos DB é adequado para o cenário porque oferece baixa latência (10 ms no 99º percentil) para ler os primeiros itens recomendados para determinado usuário.</span><span class="sxs-lookup"><span data-stu-id="9e524-133">Azure Cosmos DB is well-suited for this scenario, because it provides low latency (10 ms at 99th percentile) to read the top recommended items for a given user.</span></span>

<span data-ttu-id="9e524-134">[Serviço do Azure Machine Learning][mls].</span><span class="sxs-lookup"><span data-stu-id="9e524-134">[Azure Machine Learning Service][mls].</span></span> <span data-ttu-id="9e524-135">Esse serviço é usado para controlar e gerenciar modelos de machine learning e, em seguida, empacotar e implantar esses modelos em um ambiente escalonável do AKS.</span><span class="sxs-lookup"><span data-stu-id="9e524-135">This service is used to track and manage machine learning models, and then package and deploy these models to a scalable AKS environment.</span></span>

<span data-ttu-id="9e524-136">[Recomendações da Microsoft][github].</span><span class="sxs-lookup"><span data-stu-id="9e524-136">[Microsoft Recommenders][github].</span></span> <span data-ttu-id="9e524-137">Esse repositório de software livre contém códigos de utilitários e exemplos para que os usuários possam começar a compilar, avaliar e operacionalizar um sistema de recomendação.</span><span class="sxs-lookup"><span data-stu-id="9e524-137">This open-source repository contains utility code and samples to help users get started in building, evaluating, and operationalizing a recommender system.</span></span>

## <a name="performance-considerations"></a><span data-ttu-id="9e524-138">Considerações sobre o desempenho</span><span class="sxs-lookup"><span data-stu-id="9e524-138">Performance considerations</span></span>

<span data-ttu-id="9e524-139">O desempenho é uma consideração essencial para obter recomendações em tempo real, porque as recomendações geralmente estão no caminho crítico da solicitação que um usuário faz em seu site.</span><span class="sxs-lookup"><span data-stu-id="9e524-139">Performance is a primary consideration for real-time recommendations, because recommendations usually fall in the critical path of the request a user makes on your site.</span></span>

<span data-ttu-id="9e524-140">A combinação do AKS com o Azure Cosmos DB permite que essa arquitetura forneça um bom ponto de partida para oferecer recomendações a uma carga de trabalho de médio porte com o mínimo de sobrecarga.</span><span class="sxs-lookup"><span data-stu-id="9e524-140">The combination of AKS and Azure Cosmos DB enables this architecture to provide a good starting point to provide recommendations for a medium-sized workload with minimal overhead.</span></span> <span data-ttu-id="9e524-141">Em um teste de carga com 200 usuários simultâneos, esta arquitetura fornece recomendações com uma média de latência de cerca de 60 ms e tem desempenho a uma taxa de transferência de 180 solicitações por segundo.</span><span class="sxs-lookup"><span data-stu-id="9e524-141">Under a load test with 200 concurrent users, this architecture provides recommendations at a median latency of about 60 ms and performs at a throughput of 180 requests per second.</span></span> <span data-ttu-id="9e524-142">O teste de carga foi executado com a configuração de implantação padrão (um cluster do AKS 3x D3 v2 com 12 vCPUs, 42 GB de memória e 11.000 [RUs (unidades de solicitação) por segundo][ru] provisionado para o Azure Cosmos DB).</span><span class="sxs-lookup"><span data-stu-id="9e524-142">The load test was run against the default deployment configuration (a 3x D3 v2 AKS cluster with 12 vCPUs, 42 GB of memory, and 11,000 [Request Units (RUs) per second][ru] provisioned for Azure Cosmos DB).</span></span>

![Gráfico do desempenho](./_images/recommenders-performance.png)

![Gráfico da taxa de transferência](./_images/recommenders-throughput.png)

<span data-ttu-id="9e524-145">O Azure Cosmos DB é recomendado por sua distribuição global turnkey e por sua utilidade em atender a todos os requisitos de banco de dados que seu aplicativo possa ter.</span><span class="sxs-lookup"><span data-stu-id="9e524-145">Azure Cosmos DB is recommended for its turnkey global distribution and usefulness in meeting any database requirements your app has.</span></span> <span data-ttu-id="9e524-146">Para uma [latência ligeiramente mais rápida][latency], considere usar o [Cache Redis do Azure][redis] em vez do Azure Cosmos DB para servir as pesquisas.</span><span class="sxs-lookup"><span data-stu-id="9e524-146">For slightly [faster latency][latency], consider using [Azure Redis Cache][redis] instead of Azure Cosmos DB to serve lookups.</span></span> <span data-ttu-id="9e524-147">O Cache Redis pode melhorar o desempenho de sistemas que dependem bastante de dados em armazenamentos de back-end.</span><span class="sxs-lookup"><span data-stu-id="9e524-147">Redis Cache can improve performance of systems that rely highly on data in back-end stores.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="9e524-148">Considerações sobre escalabilidade</span><span class="sxs-lookup"><span data-stu-id="9e524-148">Scalability considerations</span></span>

<span data-ttu-id="9e524-149">Se você não planeja usar o Spark ou tem uma carga de trabalho menor que não precisa de distribuição, considere usar a [DSVM][dsvm] (Máquina Virtual de Ciência de Dados) em vez do Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="9e524-149">If you don't plan to use Spark, or you have a smaller workload where you don't need distribution, consider using [Data Science Virtual Machine][dsvm] (DSVM) instead of Azure Databricks.</span></span> <span data-ttu-id="9e524-150">A DSVM é uma máquina virtual do Azure com estruturas e ferramentas de aprendizado profundo para aprendizado de máquina e ciência de dados.</span><span class="sxs-lookup"><span data-stu-id="9e524-150">DSVM is an Azure virtual machine with deep learning frameworks and tools for machine learning and data science.</span></span> <span data-ttu-id="9e524-151">Da mesma forma que no Azure Databricks, qualquer modelo que você criar em uma DSVM poderá ser operacionalizado como um serviço no AKS por meio do Azure Machine Learning.</span><span class="sxs-lookup"><span data-stu-id="9e524-151">As with Azure Databricks, any model you create in a DSVM can be operationalized as a service on AKS via Azure Machine Learning.</span></span>

<span data-ttu-id="9e524-152">Durante o treinamento, provisione um cluster Spark de tamanho fixo maior no Azure Databricks ou configure o [dimensionamento automático][autoscaling].</span><span class="sxs-lookup"><span data-stu-id="9e524-152">During training, provision a larger fixed-size Spark cluster in Azure Databricks or configure [autoscaling][autoscaling].</span></span> <span data-ttu-id="9e524-153">Quando o dimensionamento automático estiver habilitado, o Databricks monitorará a carga no seu cluster e escalará e reduzirá verticalmente quando necessário.</span><span class="sxs-lookup"><span data-stu-id="9e524-153">When autoscaling is enabled, Databricks monitors the load on your cluster and scales up and downs when required.</span></span> <span data-ttu-id="9e524-154">Provisione ou escale horizontalmente um cluster maior se você tiver uma quantidade grande de dados e desejar reduzir a quantidade de tempo que leva as tarefas de preparação ou modelagem de dados.</span><span class="sxs-lookup"><span data-stu-id="9e524-154">Provision or scale out a larger cluster if you have a large data size and you want to reduce the amount of time it takes for data preparation or modeling tasks.</span></span>

<span data-ttu-id="9e524-155">Dimensione o cluster do AKS para atender aos requisitos de desempenho e taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="9e524-155">Scale the AKS cluster to meet your performance and throughput requirements.</span></span> <span data-ttu-id="9e524-156">Aumente o número de [pods][scale] para utilizar totalmente o cluster e escale os [nós][nodes] do cluster para atender à demanda de seu serviço.</span><span class="sxs-lookup"><span data-stu-id="9e524-156">Take care to scale up the number of [pods][scale] to fully utilize the cluster, and to scale the [nodes][nodes] of the cluster to meet the demand of your service.</span></span> <span data-ttu-id="9e524-157">Para obter mais informações sobre como dimensionar o cluster para atender a requisitos de desempenho e taxa de transferência do serviço de recomendação, confira [Escalando clusters do Serviço de Contêiner do Azure][blog].</span><span class="sxs-lookup"><span data-stu-id="9e524-157">For more information on how to scale your cluster to meet the performance and throughput requirements of your recommender service, see [Scaling Azure Container Service Clusters][blog].</span></span>

<span data-ttu-id="9e524-158">Para gerenciar o desempenho do Azure Cosmos DB, estime o número de leituras por segundo e provisione o número necessário de [RUs por segundo][ru] (taxa de transferência).</span><span class="sxs-lookup"><span data-stu-id="9e524-158">To manage Azure Cosmos DB performance, estimate the number of reads required per second, and provision the number of [RUs per second][ru] (throughput) needed.</span></span> <span data-ttu-id="9e524-159">Use as melhores práticas de [particionamento e dimensionamento horizontal][partition-data].</span><span class="sxs-lookup"><span data-stu-id="9e524-159">Use best practices for [partitioning and horizontal scaling][partition-data].</span></span>

## <a name="cost-considerations"></a><span data-ttu-id="9e524-160">Considerações de custo</span><span class="sxs-lookup"><span data-stu-id="9e524-160">Cost considerations</span></span>

<span data-ttu-id="9e524-161">Os principais geradores de custo no cenário em questão são:</span><span class="sxs-lookup"><span data-stu-id="9e524-161">The main drivers of cost in this scenario are:</span></span>

- <span data-ttu-id="9e524-162">O tamanho do cluster do Azure Databricks necessário para treinamento.</span><span class="sxs-lookup"><span data-stu-id="9e524-162">The Azure Databricks cluster size required for training.</span></span>
- <span data-ttu-id="9e524-163">O tamanho do cluster do AKS necessário para atender às necessidades de desempenho.</span><span class="sxs-lookup"><span data-stu-id="9e524-163">The AKS cluster size required to meet your performance requirements.</span></span>
- <span data-ttu-id="9e524-164">As RUs do Azure Cosmos DB provisionadas para atender às necessidades de desempenho.</span><span class="sxs-lookup"><span data-stu-id="9e524-164">Azure Cosmos DB RUs provisioned to meet your performance requirements.</span></span>

<span data-ttu-id="9e524-165">Gerencie os custos do Azure Databricks fazendo a readaptação com menor frequência e desligando o cluster Spark quando ele não estiver em uso.</span><span class="sxs-lookup"><span data-stu-id="9e524-165">Manage the Azure Databricks costs by retraining less frequently and turning off the Spark cluster when not in use.</span></span> <span data-ttu-id="9e524-166">Os custos do AKS e do Azure Cosmos DB estão vinculados à taxa de transferência e ao desempenho exigidos pelo seu site e serão escalados e reduzidos verticalmente dependendo do volume do tráfego em seu site.</span><span class="sxs-lookup"><span data-stu-id="9e524-166">The AKS and Azure Cosmos DB costs are tied to the throughput and performance required by your site and will scale up and down depending on the volume of traffic to your site.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="9e524-167">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="9e524-167">Deploy the solution</span></span>

<span data-ttu-id="9e524-168">Para implantar essa arquitetura, siga as **Azure Databricks** instruções o [documento de instalação][setup].</span><span class="sxs-lookup"><span data-stu-id="9e524-168">To deploy this architecture, follow the **Azure Databricks** instructions in the [setup document][setup].</span></span> <span data-ttu-id="9e524-169">Em resumo, as instruções exigem que você:</span><span class="sxs-lookup"><span data-stu-id="9e524-169">Briefly, the instructions require you to:</span></span>

1. <span data-ttu-id="9e524-170">Crie um [espaço de trabalho do Azure Databricks][workspace].</span><span class="sxs-lookup"><span data-stu-id="9e524-170">Create an [Azure Databricks workspace][workspace].</span></span>

1. <span data-ttu-id="9e524-171">Crie um novo cluster com a seguinte configuração no Azure Databricks:</span><span class="sxs-lookup"><span data-stu-id="9e524-171">Create a new cluster with the following configuration in Azure Databricks:</span></span>

    - <span data-ttu-id="9e524-172">Modo de cluster: Standard</span><span class="sxs-lookup"><span data-stu-id="9e524-172">Cluster mode: Standard</span></span>
    - <span data-ttu-id="9e524-173">Versão do Databricks Runtime: 4.3 (inclui Apache Spark 2.3.1, Scala 2.11)</span><span class="sxs-lookup"><span data-stu-id="9e524-173">Databricks Runtime Version: 4.3 (includes Apache Spark 2.3.1, Scala 2.11)</span></span>
    - <span data-ttu-id="9e524-174">Versão do Python: 3</span><span class="sxs-lookup"><span data-stu-id="9e524-174">Python Version: 3</span></span>
    - <span data-ttu-id="9e524-175">Tipo de driver: Standard\_DS3\_v2</span><span class="sxs-lookup"><span data-stu-id="9e524-175">Driver Type: Standard\_DS3\_v2</span></span>
    - <span data-ttu-id="9e524-176">Tipo de trabalho: Standard\_DS3\_v2 (mínimo e máximo conforme necessário)</span><span class="sxs-lookup"><span data-stu-id="9e524-176">Worker Type: Standard\_DS3\_v2 (min and max as required)</span></span>
    - <span data-ttu-id="9e524-177">Término obrigatório: (conforme necessário)</span><span class="sxs-lookup"><span data-stu-id="9e524-177">Auto Termination: (as required)</span></span>
    - <span data-ttu-id="9e524-178">Configuração do Spark: (conforme necessário)</span><span class="sxs-lookup"><span data-stu-id="9e524-178">Spark Config: (as required)</span></span>
    - <span data-ttu-id="9e524-179">Variáveis de ambiente: (conforme necessário)</span><span class="sxs-lookup"><span data-stu-id="9e524-179">Environment Variables: (as required)</span></span>

1. <span data-ttu-id="9e524-180">Criar um token de acesso pessoal dentro do [espaço de trabalho do Azure Databricks][workspace].</span><span class="sxs-lookup"><span data-stu-id="9e524-180">Create a personal access token within the [Azure Databricks workspace][workspace].</span></span> <span data-ttu-id="9e524-181">Consulte a autenticação do Azure Databricks [documentação] [ adbauthentication] para obter detalhes.</span><span class="sxs-lookup"><span data-stu-id="9e524-181">See the Azure Databricks authentication [documentation][adbauthentication] for details.</span></span>

1. <span data-ttu-id="9e524-182">Clone o [Microsoft Recommenders] [ github] repositório em um ambiente em que você pode executar scripts (por exemplo, seu computador local).</span><span class="sxs-lookup"><span data-stu-id="9e524-182">Clone the [Microsoft Recommenders][github] repository into an environment where you can execute scripts (e.g. your local computer).</span></span>

1. <span data-ttu-id="9e524-183">Siga as **instalação rápida** instruções de configuração [instalar as bibliotecas relevantes] [ setup] no Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="9e524-183">Follow the **Quick install** setup instructions to [install the relevant libraries][setup] on Azure Databricks.</span></span>

1. <span data-ttu-id="9e524-184">Siga as **instalação rápida** instruções de configuração [preparar o Azure Databricks para operacionalização][setupo16n].</span><span class="sxs-lookup"><span data-stu-id="9e524-184">Follow the **Quick install** setup instructions to [prepare Azure Databricks for operationalization][setupo16n].</span></span>

1. <span data-ttu-id="9e524-185">Importar o [bloco de anotações de operacionalização de filme ALS] [ als-example] em seu espaço de trabalho.</span><span class="sxs-lookup"><span data-stu-id="9e524-185">Import the [ALS Movie Operationalization notebook][als-example] into your workspace.</span></span> <span data-ttu-id="9e524-186">Após fazer logon no seu espaço de trabalho do Databricks do Azure, faça o seguinte:</span><span class="sxs-lookup"><span data-stu-id="9e524-186">After logging into your Azure Databricks Workspace, do the following:</span></span>

    <span data-ttu-id="9e524-187"> a.</span><span class="sxs-lookup"><span data-stu-id="9e524-187">a.</span></span> <span data-ttu-id="9e524-188">Clique em **Home** no lado esquerdo do espaço de trabalho.</span><span class="sxs-lookup"><span data-stu-id="9e524-188">Click **Home** on the left side of the workspace.</span></span>

    <span data-ttu-id="9e524-189">b.</span><span class="sxs-lookup"><span data-stu-id="9e524-189">b.</span></span> <span data-ttu-id="9e524-190">Clique com botão direito no espaço em branco em seu diretório base.</span><span class="sxs-lookup"><span data-stu-id="9e524-190">Right-click on white space in your home directory.</span></span> <span data-ttu-id="9e524-191">Selecione **Importar**.</span><span class="sxs-lookup"><span data-stu-id="9e524-191">Select **Import**.</span></span>

    <span data-ttu-id="9e524-192">c.</span><span class="sxs-lookup"><span data-stu-id="9e524-192">c.</span></span> <span data-ttu-id="9e524-193">Selecione **URL**e cole o seguinte no campo de texto: `https://github.com/Microsoft/Recommenders/blob/master/notebooks/05_operationalize/als_movie_o16n.ipynb`</span><span class="sxs-lookup"><span data-stu-id="9e524-193">Select **URL**, and paste the following into the text field: `https://github.com/Microsoft/Recommenders/blob/master/notebooks/05_operationalize/als_movie_o16n.ipynb`</span></span>

    <span data-ttu-id="9e524-194">d.</span><span class="sxs-lookup"><span data-stu-id="9e524-194">d.</span></span> <span data-ttu-id="9e524-195">Clique em **Importar**.</span><span class="sxs-lookup"><span data-stu-id="9e524-195">Click **Import**.</span></span>

1. <span data-ttu-id="9e524-196">Abra o bloco de notas no Databricks do Azure e anexe o cluster configurado.</span><span class="sxs-lookup"><span data-stu-id="9e524-196">Open the notebook within Azure Databricks and attach the configured cluster.</span></span>

1. <span data-ttu-id="9e524-197">Execute o bloco de anotações para criar os recursos do Azure necessários para criar uma API de recomendação que fornece as recomendações de filmes de 10 principais para um determinado usuário.</span><span class="sxs-lookup"><span data-stu-id="9e524-197">Run the notebook to create the Azure resources required to create a recommendation API that provides the top-10 movie recommendations for a given user.</span></span>

## <a name="related-architectures"></a><span data-ttu-id="9e524-198">Arquiteturas relacionadas</span><span class="sxs-lookup"><span data-stu-id="9e524-198">Related architectures</span></span>

<span data-ttu-id="9e524-199">Também criamos uma arquitetura de referência que usa o Spark e Azure Databricks para executar os [processos de pontuação em lote agendados][batch-scoring].</span><span class="sxs-lookup"><span data-stu-id="9e524-199">We have also built a reference architecture that uses Spark and Azure Databricks to execute scheduled [batch-scoring processes][batch-scoring].</span></span> <span data-ttu-id="9e524-200">Consulte essa arquitetura de referência para entender uma abordagem recomendada para gerar novas recomendações periodicamente.</span><span class="sxs-lookup"><span data-stu-id="9e524-200">See that reference architecture to understand a recommended approach for generating new recommendations routinely.</span></span>

<!-- links -->
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[adbauthentication]: https://docs.azuredatabricks.net/api/latest/authentication.html#generate-a-token
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[als-example]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/05_operationalize/als_movie_o16n.ipynb
[autoscaling]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[availability]: /azure/architecture/checklist/availability
[batch-scoring]: /azure/architecture/reference-architectures/ai/batch-scoring-databricks
[blob]: /azure/storage/blobs/storage-blobs-introduction
[blog]: https://blogs.technet.microsoft.com/machinelearning/2018/03/20/scaling-azure-container-service-cluster/
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[data-source]: https://docs.azuredatabricks.net/spark/latest/data-sources/index.html
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[dsvm-ubuntu]: /azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro
[eval-guide]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/03_evaluate/evaluation.ipynb
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[guide]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/01_prepare_data/data_split.ipynb
[latency]: https://github.com/jessebenson/azure-performance
[mls]: /azure/machine-learning/service/
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[nodes]: /azure/aks/scale-cluster
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[partition-data]: /azure/cosmos-db/partition-data
[redis]: /azure/redis-cache/cache-overview
[regions]: https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[ru]: /azure/cosmos-db/request-units
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md#repository-installation
[setupo16n]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md#prepare-azure-databricks-for-operationalization
[scale]: /azure/aks/tutorial-kubernetes-scale
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[workspace]: https://docs.azuredatabricks.net/getting-started/index.html
