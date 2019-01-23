---
title: Recomendações de filmes no Azure
description: Use o aprendizado de máquina para automatizar recomendações de filmes, produtos ou qualquer outro item usando o aprendizado de máquina e as DSVM (Máquinas Virtuais de Ciência de Dados) do Azure para treinar um modelo no Azure.
author: njray
ms.date: 1/9/2019
ms.custom: azcat-ai
ms.openlocfilehash: 38e883bac032596d4c14b230fa3aa102fde45837
ms.sourcegitcommit: d5ea427c25f9f7799cc859b99f328739ca2d8c1c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/15/2019
ms.locfileid: "54307735"
---
# <a name="movie-recommendations-on-azure"></a><span data-ttu-id="d9819-103">Recomendações de filmes no Azure</span><span class="sxs-lookup"><span data-stu-id="d9819-103">Movie recommendations on Azure</span></span>

<span data-ttu-id="d9819-104">Este cenário de exemplo mostra como uma empresa pode usar o aprendizado de máquina para automatizar as recomendações de produtos para seus clientes.</span><span class="sxs-lookup"><span data-stu-id="d9819-104">This example scenario shows how a business can use machine learning to automate product recommendations for their customers.</span></span> <span data-ttu-id="d9819-105">Uma DSVM (Máquina Virtual de Ciência de Dados) do Azure é usada para treinar um modelo no Azure que recomenda filmes aos usuários com base nas classificações atribuídas aos filmes.</span><span class="sxs-lookup"><span data-stu-id="d9819-105">An Azure Data Science Virtual Machine (DSVM) is used to train a model on Azure that recommends movies to users based on ratings that have been given to movies.</span></span>

<span data-ttu-id="d9819-106">As recomendações podem ser úteis em vários setores, do varejo a notícias e mídia.</span><span class="sxs-lookup"><span data-stu-id="d9819-106">Recommendations can be useful in various industries from retail to news to media.</span></span> <span data-ttu-id="d9819-107">Os aplicativos em potencial incluem o fornecimento de recomendações de produtos em uma loja virtual, o fornecimento de notícias ou recomendações de publicações ou o fornecimento de recomendações de músicas.</span><span class="sxs-lookup"><span data-stu-id="d9819-107">Potential applications include providing  product recommendations in a virtual store, providing news or post recommendations, or providing music recommendations.</span></span> <span data-ttu-id="d9819-108">Tradicionalmente, as empresas tinham que contratar e treinar assistentes para fazer recomendações personalizadas aos clientes.</span><span class="sxs-lookup"><span data-stu-id="d9819-108">Traditionally, businesses had to hire and train assistants to make personalized recommendations to customers.</span></span> <span data-ttu-id="d9819-109">Hoje, podemos oferecer recomendações personalizadas em escala por meio do Azure para treinar modelos para compreender as preferências do cliente.</span><span class="sxs-lookup"><span data-stu-id="d9819-109">Today, we can  provide customized recommendations at scale by utilizing Azure to train models to understand customer preferences.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="d9819-110">Casos de uso relevantes</span><span class="sxs-lookup"><span data-stu-id="d9819-110">Relevant use cases</span></span>

<span data-ttu-id="d9819-111">Considere este cenário para os casos de uso a seguir:</span><span class="sxs-lookup"><span data-stu-id="d9819-111">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="d9819-112">Recomendações de filmes em um site.</span><span class="sxs-lookup"><span data-stu-id="d9819-112">Movie recommendations on a website.</span></span>
* <span data-ttu-id="d9819-113">Recomendações de produtos de consumidor em um aplicativo móvel.</span><span class="sxs-lookup"><span data-stu-id="d9819-113">Consumer product recommendations in a mobile app.</span></span>
* <span data-ttu-id="d9819-114">Recomendações de notícias no streaming de mídia.</span><span class="sxs-lookup"><span data-stu-id="d9819-114">News recommendations on streaming media.</span></span>

## <a name="architecture"></a><span data-ttu-id="d9819-115">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="d9819-115">Architecture</span></span>

![Arquitetura de um modelo de machine learning para treinamento de recomendações de filmes][architecture]

<span data-ttu-id="d9819-117">Este cenário abrange o treinamento e a avaliação do modelo de machine learning usando o algoritmo ALS ([quadrados mínimos alternativos][als]) do Spark em um conjunto de dados de classificações de filmes.</span><span class="sxs-lookup"><span data-stu-id="d9819-117">This scenario covers the training and evaluating of the machine learning model using the Spark [alternating least squares][als] (ALS) algorithm on a dataset of movie ratings.</span></span> <span data-ttu-id="d9819-118">As etapas para esse cenário são as seguintes:</span><span class="sxs-lookup"><span data-stu-id="d9819-118">The steps for this scenario are as following:</span></span>

1. <span data-ttu-id="d9819-119">O site ou serviço de aplicativo front-end coleta dados históricos de interações entre usuários e filmes, que são representados em uma tabela de tuplas de classificação numérica, de usuários e de itens.</span><span class="sxs-lookup"><span data-stu-id="d9819-119">The front-end website or app service collects historical data of user-movie interactions, which are represented in a table of user, item, and numerical rating tuples.</span></span>

2. <span data-ttu-id="d9819-120">Os dados históricos coletados são armazenados em um armazenamento de blobs.</span><span class="sxs-lookup"><span data-stu-id="d9819-120">The collected historical data is stored in a blob storage.</span></span>

3. <span data-ttu-id="d9819-121">Uma DSVM é frequentemente usada para experimentar ou produzir um modelo de recomendação do ALS do Spark.</span><span class="sxs-lookup"><span data-stu-id="d9819-121">A DSVM is often used to experiment with or productize a Spark ALS recommender model.</span></span> <span data-ttu-id="d9819-122">O modelo de ALS é treinado usando um conjunto de dados de treinamento, que é produzido a partir do conjunto de dados geral, aplicando a estratégia apropriada de divisão de dados.</span><span class="sxs-lookup"><span data-stu-id="d9819-122">The ALS model is trained using a training dataset, which is produced from the overall dataset by applying the appropriate data splitting strategy.</span></span> <span data-ttu-id="d9819-123">Por exemplo, o conjunto de dados pode ser dividido em conjuntos aleatoriamente, em ordem cronológica ou de forma estratificada, dependendo do requisito de negócios.</span><span class="sxs-lookup"><span data-stu-id="d9819-123">For example, the dataset can be split into sets randomly, chronologically, or stratified, depending on the business requirement.</span></span> <span data-ttu-id="d9819-124">Semelhante a outras tarefas de aprendizado de máquina, uma recomendação é validada usando métricas de avaliação (por exemplo, precision\@*k*, recall\@*k*, [MAP][map], [nDCG\@k][ndcg]).</span><span class="sxs-lookup"><span data-stu-id="d9819-124">Similar to other machine learning tasks, a recommender is validated by using evaluation metrics (for example, precision\@*k*, recall\@*k*, [MAP][map], [nDCG\@k][ndcg]).</span></span>

4. <span data-ttu-id="d9819-125">O Serviço do Azure Machine Learning é usado para coordenar a experimentação, como a varredura de hiperparâmetro e o gerenciamento de modelo.</span><span class="sxs-lookup"><span data-stu-id="d9819-125">Azure Machine Learning service is used for coordinating the experimentation, such as hyperparameter sweeping and model management.</span></span>

5. <span data-ttu-id="d9819-126">Um modelo treinado é preservado no Azure Cosmos DB, que pode ser aplicado para recomendar os principais filmes *k* para um determinado usuário.</span><span class="sxs-lookup"><span data-stu-id="d9819-126">A trained model is preserved on Azure Cosmos DB, which can then be applied for recommending the top *k* movies for a given user.</span></span>

6. <span data-ttu-id="d9819-127">O modelo é implantado em um serviço Web ou de aplicativo usando as Instâncias de Contêiner do Azure ou o Serviço de Kubernetes do Azure.</span><span class="sxs-lookup"><span data-stu-id="d9819-127">The model is then deployed onto a web or app service by using Azure Container Instances or Azure Kubernetes Service.</span></span>

<span data-ttu-id="d9819-128">Para obter um guia detalhado para criar e dimensionar um serviço de recomendação, confira [Compilar uma API de recomendação em tempo real no Azure][ref-arch].</span><span class="sxs-lookup"><span data-stu-id="d9819-128">For an in-depth guide to building and scaling a recommender service, see [Build a real-time recommendation API on Azure][ref-arch].</span></span>

### <a name="components"></a><span data-ttu-id="d9819-129">Componentes</span><span class="sxs-lookup"><span data-stu-id="d9819-129">Components</span></span>

* <span data-ttu-id="d9819-130">A [Máquina Virtual de Ciência de Dados][dsvm] (DSVM) é uma máquina virtual do Azure com estruturas e ferramentas de aprendizado profundo para aprendizado de máquina e ciência de dados.</span><span class="sxs-lookup"><span data-stu-id="d9819-130">[Data Science Virtual Machine][dsvm] (DSVM) is an Azure virtual machine with deep learning frameworks and tools for machine learning and data science.</span></span> <span data-ttu-id="d9819-131">A DSVM tem um ambiente Spark autônomo que pode ser usado para executar o ALS.</span><span class="sxs-lookup"><span data-stu-id="d9819-131">The DSVM has a standalone Spark environment that can be used to run ALS.</span></span>

* <span data-ttu-id="d9819-132">O [armazenamento de Blobs do Azure][blob] armazena o conjunto de dados para obter recomendações de filmes.</span><span class="sxs-lookup"><span data-stu-id="d9819-132">[Azure Blob storage][blob] stores the dataset for movie recommendations.</span></span>

* <span data-ttu-id="d9819-133">O [Serviço do Azure Machine Learning][mls] é usado para acelerar a criação, o gerenciamento e a implantação de modelos de machine learning.</span><span class="sxs-lookup"><span data-stu-id="d9819-133">[Azure Machine Learning service][mls] is used to accelerate the building, managing, and deploying of machine learning models.</span></span>

* <span data-ttu-id="d9819-134">O [Azure Cosmos DB][cosmosdb] habilita o armazenamento distribuído globalmente e o armazenamento multimodelo de banco de dados.</span><span class="sxs-lookup"><span data-stu-id="d9819-134">[Azure Cosmos DB][cosmosdb] enables globally distributed and multi-model database storage.</span></span>

* <span data-ttu-id="d9819-135">As [Instâncias de Contêiner do Azure][aci] são usadas para implantar os modelos treinados em serviços Web ou de aplicativos, usando opcionalmente o [Serviço de Kubernetes do Azure][aks].</span><span class="sxs-lookup"><span data-stu-id="d9819-135">[Azure Container Instances][aci] is used to deploy the trained models to web or app services, optionally using [Azure Kubernetes Service][aks].</span></span>

### <a name="alternatives"></a><span data-ttu-id="d9819-136">Alternativas</span><span class="sxs-lookup"><span data-stu-id="d9819-136">Alternatives</span></span>

<span data-ttu-id="d9819-137">O [Azure Databricks][databricks] é um cluster gerenciado do Spark em que o modelo de treinamento e avaliação é executado.</span><span class="sxs-lookup"><span data-stu-id="d9819-137">[Azure Databricks][databricks] is a managed Spark cluster where model training and evaluating is performed.</span></span> <span data-ttu-id="d9819-138">Você pode configurar rapidamente um ambiente gerenciado do Spark e [dimensionar automaticamente][autoscale] para ajudar a reduzir os recursos e os custos associados ao escalonamento manual de clusters.</span><span class="sxs-lookup"><span data-stu-id="d9819-138">You can set up a managed Spark environment in minutes, and [autoscale][autoscale] up and down to help reduce the resources and costs associated with scaling clusters manually.</span></span> <span data-ttu-id="d9819-139">Outra opção de economia de recursos é configurar [clusters][clusters] inativos para encerramento automático.</span><span class="sxs-lookup"><span data-stu-id="d9819-139">Another resource-saving option is to configure inactive [clusters][clusters] to terminate automatically.</span></span>

## <a name="considerations"></a><span data-ttu-id="d9819-140">Considerações</span><span class="sxs-lookup"><span data-stu-id="d9819-140">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="d9819-141">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="d9819-141">Availability</span></span>

<span data-ttu-id="d9819-142">Os aplicativos criados pelo aprendizado de máquina são divididos em dois componentes de recursos: recursos para treinamento e recursos para fornecimento.</span><span class="sxs-lookup"><span data-stu-id="d9819-142">Machine-learning-built apps are split into two resource components: resources for training, and resources for serving.</span></span> <span data-ttu-id="d9819-143">Os recursos necessários para treinamento geralmente não precisam de alta disponibilidade já que as solicitações de produção ao vivo não atingem diretamente esses recursos.</span><span class="sxs-lookup"><span data-stu-id="d9819-143">Resources required for training generally do not need  high availability, as live production requests do not directly hit these resources.</span></span> <span data-ttu-id="d9819-144">Os recursos necessários para o serviço precisam ter alta disponibilidade para atender às solicitações do cliente.</span><span class="sxs-lookup"><span data-stu-id="d9819-144">Resources required for serving need to have high availability to serve customer requests.</span></span>

<span data-ttu-id="d9819-145">Para treinamento, a DSVM está disponível em [várias regiões][regions] de todo mundo e atende ao [Contrato de Nível de Serviço][sla] (SLA) para máquinas virtuais.</span><span class="sxs-lookup"><span data-stu-id="d9819-145">For training, the DSVM is available in [multiple regions][regions] around the globe and meets the [service level agreement][sla] (SLA) for virtual machines.</span></span> <span data-ttu-id="d9819-146">Para fornecimento, o Serviço de Kubernetes do Azure fornece uma infraestrutura [altamente disponível][ha].</span><span class="sxs-lookup"><span data-stu-id="d9819-146">For serving, Azure Kubernetes Service provides a [highly available][ha] infrastructure.</span></span> <span data-ttu-id="d9819-147">Os nós do agente também obedecem o [SLA][sla-aks] para máquinas virtuais.</span><span class="sxs-lookup"><span data-stu-id="d9819-147">Agent nodes also follow the [SLA][sla-aks] for virtual machines.</span></span>

### <a name="scalability"></a><span data-ttu-id="d9819-148">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="d9819-148">Scalability</span></span>

<span data-ttu-id="d9819-149">Se você tiver um tamanho grande de dados, é possível dimensionar sua DSVM para reduzir o tempo de treinamento.</span><span class="sxs-lookup"><span data-stu-id="d9819-149">If you have a large data size, you can scale your DSVM to shorten training time.</span></span> <span data-ttu-id="d9819-150">Você pode dimensionar uma VM vertical ou horizontalmente alterando o [tamanho da VM][vm-size].</span><span class="sxs-lookup"><span data-stu-id="d9819-150">You can scale a VM up or down by changing the [VM size][vm-size].</span></span> <span data-ttu-id="d9819-151">Escolha um tamanho de memória grande o suficiente para se ajustar ao conjunto de dados na memória e uma contagem maior de vCPU para reduzir o tempo gasto no treinamento.</span><span class="sxs-lookup"><span data-stu-id="d9819-151">Choose a memory size large enough to fit your dataset in-memory and a higher vCPU count in order to decrease the amount of time that training takes.</span></span>

### <a name="security"></a><span data-ttu-id="d9819-152">Segurança</span><span class="sxs-lookup"><span data-stu-id="d9819-152">Security</span></span>

<span data-ttu-id="d9819-153">Esse cenário pode usar o Azure Active Directory para autenticar usuários no [acesso à DSVM][dsvm-id], que contém seu código, modelos e dados (na memória).</span><span class="sxs-lookup"><span data-stu-id="d9819-153">This scenario can use Azure Active Directory to authenticate users for [access to the DSVM][dsvm-id], which contains your code, models, and (in-memory) data.</span></span> <span data-ttu-id="d9819-154">Os dados são armazenados no Armazenamento do Azure antes de serem carregados em uma DSVM, onde são criptografados automaticamente usando a [Criptografia do Serviço de Armazenamento][storage-security].</span><span class="sxs-lookup"><span data-stu-id="d9819-154">Data is stored in Azure Storage prior to being loaded on a DSVM, where it is automatically encrypted using [Storage Service Encryption][storage-security].</span></span> <span data-ttu-id="d9819-155">As permissões podem ser gerenciadas pela autenticação do Azure Active Directory ou pelo controle de acesso baseado em função.</span><span class="sxs-lookup"><span data-stu-id="d9819-155">Permissions can be managed via Azure Active Directory authentication or role-based access control.</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="d9819-156">Implantar este cenário</span><span class="sxs-lookup"><span data-stu-id="d9819-156">Deploy this scenario</span></span>

<span data-ttu-id="d9819-157">**Pré-requisitos**: Você deve ter uma conta do Azure já criada.</span><span class="sxs-lookup"><span data-stu-id="d9819-157">**Prerequisites**: You must have an existing Azure account.</span></span> <span data-ttu-id="d9819-158">Se você não tiver uma assinatura do Azure, crie uma [conta gratuita][free] antes de começar.</span><span class="sxs-lookup"><span data-stu-id="d9819-158">If you don't have an Azure subscription, create a [free account][free] before you begin.</span></span>

<span data-ttu-id="d9819-159">O código completo para esse cenário está disponível no [repositório Microsoft Recommenders][github].</span><span class="sxs-lookup"><span data-stu-id="d9819-159">All the code for this scenario is available in the [Microsoft Recommenders repository][github].</span></span>

<span data-ttu-id="d9819-160">Siga estas etapas para executar o [Caderno de início rápido do ALS][notebook]:</span><span class="sxs-lookup"><span data-stu-id="d9819-160">Follow these steps to run the [ALS quickstart notebook][notebook]:</span></span>

1. <span data-ttu-id="d9819-161">[Crie uma DSVM][dsvm-ubuntu] a partir do portal do Azure.</span><span class="sxs-lookup"><span data-stu-id="d9819-161">[Create a DSVM][dsvm-ubuntu] from the Azure portal.</span></span>

2. <span data-ttu-id="d9819-162">Clone o repositório na pasta Blocos de Anotações:</span><span class="sxs-lookup"><span data-stu-id="d9819-162">Clone the repo in the Notebooks folder:</span></span>

    ```shell
    cd notebooks
    git clone https://github.com/Microsoft/Recommenders
    ```

3. <span data-ttu-id="d9819-163">Instale as dependências de conda seguindo as etapas descritas no arquivo [SETUP.md][setup].</span><span class="sxs-lookup"><span data-stu-id="d9819-163">Install the conda dependencies following the steps described in the [SETUP.md][setup] file.</span></span>

4. <span data-ttu-id="d9819-164">Em um navegador, acesse sua VM do jupyterlab e navegue até `notebooks/00_quick_start/als_pyspark_movielens.ipynb`.</span><span class="sxs-lookup"><span data-stu-id="d9819-164">In a browser, go to your jupyterlab VM and navigate to `notebooks/00_quick_start/als_pyspark_movielens.ipynb`.</span></span>

5. <span data-ttu-id="d9819-165">Execute o bloco de anotações.</span><span class="sxs-lookup"><span data-stu-id="d9819-165">Execute the notebook.</span></span>

## <a name="related-resources"></a><span data-ttu-id="d9819-166">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="d9819-166">Related resources</span></span>

<span data-ttu-id="d9819-167">Para obter um guia detalhado para criar e dimensionar um serviço de recomendação, confira [Compilar uma API de recomendação em tempo real no Azure][ref-arch].</span><span class="sxs-lookup"><span data-stu-id="d9819-167">For an in-depth guide to building and scaling a recommender service, see [Build a real-time recommendation API on Azure][ref-arch].</span></span> <span data-ttu-id="d9819-168">Para obter tutoriais e exemplos de sistemas de recomendação, confira [repositório Microsoft Recommenders][github].</span><span class="sxs-lookup"><span data-stu-id="d9819-168">For tutorials and examples of recommendation systems, see [Microsoft Recommenders repository][github].</span></span>

[architecture]: ./media/architecture-movie-recommender.png
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[blob]: /azure/storage/blobs/storage-blobs-introduction
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[dsvm-id]: /azure/machine-learning/data-science-virtual-machine/dsvm-common-identity
[dsvm-ubuntu]: /azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[ha]: /azure/aks/container-service-quotas
[map]: https://en.wikipedia.org/wiki/Evaluation_measures_(information_retrieval)
[mls]: /azure/machine-learning/service/
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[ref-arch]: /azure/architecture/reference-architectures/ai/real-time-recommendation
[regions]: https://azure.microsoft.com/en-us/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md%60
[sla]: https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_8/
[sla-aks]: https://azure.microsoft.com/en-us/support/legal/sla/kubernetes-service/v1_0/
[storage-security]: /azure/storage/common/storage-service-encryption
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
