---
title: Recomendações de filmes no Azure
description: Use o aprendizado de máquina para automatizar recomendações de filmes, produtos ou qualquer outro item usando o aprendizado de máquina e as DSVM (Máquinas Virtuais de Ciência de Dados) do Azure para treinar um modelo no Azure.
author: njray
ms.date: 1/9/2019
ms.custom: azcat-ai
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-movie-recommender.png
ms.openlocfilehash: 9e68f38cb61d7a3255b76a662c58907704914052
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908249"
---
# <a name="movie-recommendations-on-azure"></a>Recomendações de filmes no Azure

Este cenário de exemplo mostra como uma empresa pode usar o aprendizado de máquina para automatizar as recomendações de produtos para seus clientes. Uma DSVM (Máquina Virtual de Ciência de Dados) do Azure é usada para treinar um modelo no Azure que recomenda filmes aos usuários com base nas classificações atribuídas aos filmes.

As recomendações podem ser úteis em vários setores, do varejo a notícias e mídia. Os aplicativos em potencial incluem o fornecimento de recomendações de produtos em uma loja virtual, o fornecimento de notícias ou recomendações de publicações ou o fornecimento de recomendações de músicas. Tradicionalmente, as empresas tinham que contratar e treinar assistentes para fazer recomendações personalizadas aos clientes. Hoje, podemos oferecer recomendações personalizadas em escala por meio do Azure para treinar modelos para compreender as preferências do cliente.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Considere este cenário para os casos de uso a seguir:

* Recomendações de filmes em um site.
* Recomendações de produtos de consumidor em um aplicativo móvel.
* Recomendações de notícias no streaming de mídia.

## <a name="architecture"></a>Arquitetura

![Arquitetura de um modelo de machine learning para treinamento de recomendações de filmes][architecture]

Este cenário abrange o treinamento e a avaliação do modelo de machine learning usando o algoritmo ALS ([quadrados mínimos alternativos][als]) do Spark em um conjunto de dados de classificações de filmes. As etapas para esse cenário são as seguintes:

1. O site ou serviço de aplicativo front-end coleta dados históricos de interações entre usuários e filmes, que são representados em uma tabela de tuplas de classificação numérica, de usuários e de itens.

2. Os dados históricos coletados são armazenados em um armazenamento de blobs.

3. Uma DSVM é frequentemente usada para experimentar ou produzir um modelo de recomendação do ALS do Spark. O modelo de ALS é treinado usando um conjunto de dados de treinamento, que é produzido a partir do conjunto de dados geral, aplicando a estratégia apropriada de divisão de dados. Por exemplo, o conjunto de dados pode ser dividido em conjuntos aleatoriamente, em ordem cronológica ou de forma estratificada, dependendo do requisito de negócios. Semelhante a outras tarefas de aprendizado de máquina, uma recomendação é validada usando métricas de avaliação (por exemplo, precision\@*k*, recall\@*k*, [MAP][map], [nDCG\@k][ndcg]).

4. O Serviço do Azure Machine Learning é usado para coordenar a experimentação, como a varredura de hiperparâmetro e o gerenciamento de modelo.

5. Um modelo treinado é preservado no Azure Cosmos DB, que pode ser aplicado para recomendar os principais filmes *k* para um determinado usuário.

6. O modelo é implantado em um serviço Web ou de aplicativo usando as Instâncias de Contêiner do Azure ou o Serviço de Kubernetes do Azure.

Para obter um guia detalhado para criar e dimensionar um serviço de recomendação, confira [Compilar uma API de recomendação em tempo real no Azure][ref-arch].

### <a name="components"></a>Componentes

* A [Máquina Virtual de Ciência de Dados][dsvm] (DSVM) é uma máquina virtual do Azure com estruturas e ferramentas de aprendizado profundo para aprendizado de máquina e ciência de dados. A DSVM tem um ambiente Spark autônomo que pode ser usado para executar o ALS.

* O [armazenamento de Blobs do Azure][blob] armazena o conjunto de dados para obter recomendações de filmes.

* O [Serviço do Azure Machine Learning][mls] é usado para acelerar a criação, o gerenciamento e a implantação de modelos de machine learning.

* O [Azure Cosmos DB][cosmosdb] habilita o armazenamento distribuído globalmente e o armazenamento multimodelo de banco de dados.

* As [Instâncias de Contêiner do Azure][aci] são usadas para implantar os modelos treinados em serviços Web ou de aplicativos, usando opcionalmente o [Serviço de Kubernetes do Azure][aks].

### <a name="alternatives"></a>Alternativas

O [Azure Databricks][databricks] é um cluster gerenciado do Spark em que o modelo de treinamento e avaliação é executado. Você pode configurar rapidamente um ambiente gerenciado do Spark e [dimensionar automaticamente][autoscale] para ajudar a reduzir os recursos e os custos associados ao escalonamento manual de clusters. Outra opção de economia de recursos é configurar [clusters][clusters] inativos para encerramento automático.

## <a name="considerations"></a>Considerações

### <a name="availability"></a>Disponibilidade

Os aplicativos criados pelo aprendizado de máquina são divididos em dois componentes de recursos: recursos para treinamento e recursos para fornecimento. Os recursos necessários para treinamento geralmente não precisam de alta disponibilidade já que as solicitações de produção ao vivo não atingem diretamente esses recursos. Os recursos necessários para o serviço precisam ter alta disponibilidade para atender às solicitações do cliente.

Para treinamento, a DSVM está disponível em [várias regiões][regions] de todo mundo e atende ao [Contrato de Nível de Serviço][sla] (SLA) para máquinas virtuais. Para fornecimento, o Serviço de Kubernetes do Azure fornece uma infraestrutura [altamente disponível][ha]. Os nós do agente também obedecem o [SLA][sla-aks] para máquinas virtuais.

### <a name="scalability"></a>Escalabilidade

Se você tiver um tamanho grande de dados, é possível dimensionar sua DSVM para reduzir o tempo de treinamento. Você pode dimensionar uma VM vertical ou horizontalmente alterando o [tamanho da VM][vm-size]. Escolha um tamanho de memória grande o suficiente para se ajustar ao conjunto de dados na memória e uma contagem maior de vCPU para reduzir o tempo gasto no treinamento.

### <a name="security"></a>Segurança

Esse cenário pode usar o Azure Active Directory para autenticar usuários no [acesso à DSVM][dsvm-id], que contém seu código, modelos e dados (na memória). Os dados são armazenados no Armazenamento do Azure antes de serem carregados em uma DSVM, onde são criptografados automaticamente usando a [Criptografia do Serviço de Armazenamento][storage-security]. As permissões podem ser gerenciadas pela autenticação do Azure Active Directory ou pelo controle de acesso baseado em função.

## <a name="deploy-this-scenario"></a>Implantar este cenário

**Pré-requisitos**: Você deve ter uma conta do Azure já criada. Se você não tiver uma assinatura do Azure, crie uma [conta gratuita][free] antes de começar.

O código completo para esse cenário está disponível no [repositório Microsoft Recommenders][github].

Siga estas etapas para executar o [Caderno de início rápido do ALS][notebook]:

1. [Crie uma DSVM][dsvm-ubuntu] a partir do portal do Azure.

2. Clone o repositório na pasta Blocos de Anotações:

    ```shell
    cd notebooks
    git clone https://github.com/Microsoft/Recommenders
    ```

3. Instale as dependências de conda seguindo as etapas descritas no arquivo [SETUP.md][setup].

4. Em um navegador, acesse sua VM do jupyterlab e navegue até `notebooks/00_quick_start/als_pyspark_movielens.ipynb`.

5. Execute o bloco de anotações.

## <a name="related-resources"></a>Recursos relacionados

Para obter um guia detalhado para criar e dimensionar um serviço de recomendação, confira [Compilar uma API de recomendação em tempo real no Azure][ref-arch]. Para obter tutoriais e exemplos de sistemas de recomendação, confira [repositório Microsoft Recommenders][github].

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
