---
title: Escolhendo uma tecnologia de aprendizado de máquina
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: a435286781281a43864561f98362675219bb1477
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486566"
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a>Escolhendo uma tecnologia de aprendizado de máquina no Azure

A ciência de dados e o aprendizado de máquina são uma carga de trabalho que normalmente é realizada por cientistas de dados. Exige ferramentas especializadas, muitas das quais foram projetadas especificamente para o tipo de tarefas de modelagem e exploração interativa de dados que devem ser executadas por um cientista de dados.

As soluções de aprendizado de máquina são criadas de forma iterativa e apresentam duas fases distintas:

- Preparação de dados e modelagem.
- Implantação e consumo de serviços preditivos.

## <a name="tools-and-services-for-data-preparation-and-modeling"></a>Ferramentas e serviços para preparação de dados e modelagem

Em geral, os cientistas de dados preferem trabalhar com os dados usando um código personalizado escrito em Python ou R. Geralmente, esse código é executado de maneira interativa, com os cientistas de dados usando-o para consultar e explorar os dados, além de gerar visualizações e estatísticas para ajudar a determinar as relações existentes entre eles. Diferentes ambientes interativos do R e Python podem ser usados pelos cientistas de dados. Um dos favoritos é o **Jupyter Notebooks** que fornece um shell baseado em navegador que permite aos cientistas de dados criar arquivos de *bloco de anotações* contendo um texto de markdown ou código do R ou Python. Essa é uma maneira eficiente de colaborar por meio de compartilhamento e documentação do código e dos resultados em um único documento.

Outras ferramentas usadas incluem:

- **Spyder**: o IDE (ambiente de desenvolvimento interativo) para o Python fornecido com a distribuição do Anaconda Python.
- **R Studio**: um IDE para a linguagem de programação R.
- **Visual Studio Code**: um ambiente de codificação leve e de multiplaforma que dá suporte ao Python e a estruturas geralmente usadas para aprendizado de máquina e desenvolvimento de IA.

Além dessas ferramentas, os cientistas de dados podem aproveitar os serviços do Azure para simplificar o gerenciamento de código e de modelos.

### <a name="azure-notebooks"></a>Azure Notebooks

O Azure Notebooks é um serviço online do Jupyter Notebooks que possibilita aos cientistas de dados criar, executar e compartilhar Jupyter Notebooks em bibliotecas baseadas em nuvem.

Principais benefícios:

- Serviço gratuito &mdash; nenhuma assinatura do Azure é necessária.
- Não há necessidade de instalar o Jupyter e as distribuições do R ou Python com suporte localmente &mdash; basta usar um navegador.
- Gerencie suas próprias bibliotecas online e acesse-as em qualquer dispositivo.
- Compartilhe seus blocos de anotações com colaboradores.

Considerações:

- Você não poderá acessar os blocos de anotações quando estiver offline.
- As funcionalidades de processamento limitadas do serviço gratuito de bloco de anotações podem não ser suficientes para treinar modelos grandes ou complexos.

### <a name="data-science-virtual-machine"></a>Máquina virtual de ciência de dados

A máquina virtual de ciência de dados é uma imagem de máquina virtual do Azure que inclui as ferramentas e estruturas usadas por cientistas de dados, incluindo o R, Python, Jupyter Notebooks, Visual Studio Code e bibliotecas para modelagem de aprendizado de máquina, como o Microsoft Cognitive Toolkit. Essas ferramentas podem ser complexas e demoradas para serem instaladas e contêm muitas interdependências que, com frequência, levam a problemas de gerenciamento de versão. Ter uma imagem pré-instalada pode reduzir o tempo gasto pelos cientistas de dados com a solução de problemas de ambiente, permitindo que eles se concentrem nas tarefas de modelagem e exploração de dados que precisam realizar.

Principais benefícios:

- Redução do tempo para instalar, gerenciar e solucionar problemas de estruturas e ferramentas de ciência de dados.
- As últimas versões de todas as ferramentas e estruturas usadas são incluídas.
- As opções de máquina virtual incluem imagens altamente escalonáveis com funcionalidades de GPU para modelagem de dados intensiva.

Considerações:

- A máquina virtual não poderá ser acessada quando você estiver offline.
- A execução de uma máquina virtual incorre em encargos do Azure e, portanto, você deve ter cuidado para que ela seja executada somente quando necessário.

### <a name="azure-machine-learning"></a>Azure Machine Learning

O Azure Machine Learning é um serviço baseado em nuvem usado para gerenciar modelos e experimentos de aprendizado de máquina. Ele inclui um serviço de experimentação que acompanha os scripts de preparação de dados e treinamento de modelagem, mantendo um histórico de todas as execuções, de modo que você possa comparar o desempenho do modelo em várias iterações. Os cientistas de dados podem criar scripts em sua ferramenta preferidas, como blocos de anotações do Jupyter ou do Visual Studio Code e, em seguida, implantar em uma variedade de [recursos de computação](/azure/machine-learning/service/how-to-set-up-training-targets) diferentes no Azure.

Os modelos podem ser implantados como um serviço Web para um contêiner do Docker, Spark no HDinsight do Azure, Microsoft Machine Learning Server ou SQL Server. Em seguida, o serviço Gerenciamento de Modelos do Azure Machine Learning permite acompanhar e gerenciar implantações de modelo na nuvem, em dispositivos de borda ou em toda a empresa.

Principais benefícios:

- Gerenciamento central de scripts e histórico de execuções, facilitando a comparação de versões de modelo.
- Transformação interativa de dados por meio de um editor visual.
- Fácil implantação e gerenciamento de modelos na nuvem ou em dispositivos de borda.

Considerações:

- Exige um pouco de familiaridade com o modelo de gerenciamento de modelos.

### <a name="azure-batch-ai"></a>IA do Lote do Azure

O IA do Lote do Azure permite que você execute experimentos de aprendizado de máquina em paralelo e faça o treinamento do modelo em escala em um cluster de máquinas virtuais com GPUs. O treinamento do IA do Lote permite que você escale horizontalmente trabalhos de aprendizado profundo em GPUs clusterizadas, usando estruturas como o Cognitive Toolkit, Caffe, Chainer e TensorFlow.

O Gerenciamento de Modelos do Azure Machine Learning pode ser usado para implantar, gerenciar e monitorar modelos do IA do Lote.

### <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

O Azure Machine Learning Studio é um ambiente de desenvolvimento visual baseado em nuvem usado para criar experimentos de dados, treinar modelos de aprendizado de máquina e publicá-los como serviços Web no Azure. Sua interface visual do tipo "arrastar e soltar" permite aos cientistas de dados e usuários avançados criar soluções de aprendizado de máquina rapidamente, enquanto dão suporte à lógica personalizada do R e do Python, a uma ampla variedade de algoritmos estatísticos estabelecidos e técnicas para tarefas de modelagem de aprendizado de máquina, bem como o suporte interno para o Jupyter Notebooks.

Principais benefícios:

- A interface visual interativa possibilita a modelagem de aprendizado de máquina com o mínimo de código.
- Jupyter Notebooks interno para exploração de dados.
- Implantação direta de modelos treinados como serviços Web do Azure.

Considerações:

- Escalabilidade limitada. O tamanho máximo de um conjunto de dados de treinamento é de 10 GB.
- Somente online. Nenhum ambiente de desenvolvimento offline.

## <a name="tools-and-services-for-deploying-machine-learning-models"></a>Ferramentas e serviços para a implantação de modelos de aprendizado de máquina

Depois que um cientista de dados criar um modelo de aprendizado de máquina, você geralmente precisará implantá-lo e consumi-lo em aplicativos ou em outros fluxos de dados. Há vários destinos potenciais de implantação para modelos de aprendizado de máquina.

### <a name="spark-on-azure-hdinsight"></a>Spark no Azure HDInsight

O Apache Spark inclui o Spark MLlib, uma estrutura e biblioteca de modelos de aprendizado de máquina. O MMLSpark (biblioteca do Microsoft Machine Learning para Spark) também fornece suporte a algoritmos de aprendizado profundo em modelos preditivos no Spark.

Principais benefícios:

- O Spark é uma plataforma distribuída que oferece alta escalabilidade para processos de aprendizado de máquina de alto volume.
- Implante modelos diretamente no Spark no HDinsight e gerencie-os usando o serviço Gerenciamento de Modelos do Azure Machine Learning.

Considerações:

- O Spark é executado em um cluster HDInsight que resulta em encargos durante todo o tempo em que estiver em execução. Se o serviço de aprendizado de máquina for destinado a ser usado apenas ocasionalmente, isso poderá resultar em custos desnecessários.

### <a name="azure-databricks"></a>Azure Databricks

O [Azure Databricks](/azure/azure-databricks/) é uma plataforma de análise baseada no Apache Spark. Você pode imaginá-la como o “Spark como um serviço”. É a maneira mais fácil de usar o Spark na plataforma do Azure. Para aprendizado de máquina, você pode usar [MLFlow](https://www.mlflow.org/), [ML do Databricks Runtime](https://docs.azuredatabricks.net/user-guide/clusters/mlruntime.html), MLlib do Apache Spark e outros. Para mais informações, confira [Azure Databricks: Machine Learning](https://docs.azuredatabricks.net/spark/latest/mllib/index.html).

### <a name="web-service-in-a-container"></a>Serviço Web em um contêiner

Implante um modelo de aprendizado de máquina como um serviço Web do Python em um contêiner do Docker. Implante o modelo no Azure ou em um dispositivo de borda, em que ele pode ser usado localmente com os dados nos quais ele opera.

Principais benefícios:

- Os contêineres são uma maneira leve e geralmente econômica de empacotar e implantar serviços.
- A capacidade de implantar em um dispositivo de borda permite aproximar a lógica preditiva dos dados.

Considerações:

- Esse modelo de implantação se baseia em contêineres do Docker; portanto, você deve estar familiarizado com essa tecnologia antes de implantar um serviço Web dessa maneira.

### <a name="microsoft-machine-learning-server"></a>Microsoft Machine Learning Server

O Machine Learning Server (anteriormente Microsoft R Server) é uma plataforma escalonável para o código do R e do Python, projetada especificamente para cenários de aprendizado de máquina.

Principais benefícios:

- Alta escalabilidade.

Considerações:

- Você precisa implantar e gerenciar o Machine Learning Server em sua empresa.

### <a name="microsoft-sql-server"></a>Microsoft SQL Server

O Microsoft SQL Server dá suporte ao R e ao Python nativamente, permitindo encapsular modelos de aprendizado de máquina criados nessas linguagens como funções Transact-SQL em um banco de dados.

Principais benefícios:

- Encapsule a lógica preditiva em uma função de banco de dados, facilitando sua inclusão na lógica da camada de dados.

Considerações:

- Pressupõe um banco de dados do SQL Server como a camada de dados para o aplicativo.

### <a name="azure-machine-learning-web-service"></a>Serviço Web de Azure Machine Learning

Quando você cria um modelo de aprendizado de máquina com o Azure Machine Learning Studio, pode implantá-lo como um serviço Web. Em seguida, isso pode ser consumido por meio de uma interface REST em qualquer aplicativo cliente com capacidade de se comunicar por HTTP.

Principais benefícios:

- Facilidade de desenvolvimento e implantação.
- Portal de gerenciamento de serviço Web com métricas básicas de monitoramento.
- Suporte interno para chamada de serviços Web do Azure Machine Learning no Azure Data Lake Analytics, Azure Data Factory e Azure Stream Analytics.

Considerações:

- Disponível somente para modelos criados com o Azure Machine Learning Studio.
- Modelos treinados somente com acesso baseado na Web não podem ser executados localmente ou offline.
