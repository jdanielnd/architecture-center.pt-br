---
title: Escolhendo uma tecnologia de aprendizado de máquina
description: Comparar as opções para criar, implantar e gerenciar seus modelos de aprendizado de máquina. Decida quais produtos da Microsoft escolher para sua solução.
author: MikeWasson
ms.date: 03/06/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 1020e938a04c6a82e6cc831e6620ec17c9a10cc7
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503223"
---
# <a name="what-are-the-machine-learning-products-at-microsoft"></a>Quais são os produtos de aprendizado de máquina na Microsoft?

O Machine Learning é uma técnica da ciência de dados que permite que os computadores usem os dados existentes para prever tendências, resultados e comportamentos futuros. Usando o aprendizado de máquina, os computadores aprendem sem serem explicitamente programados.

Soluções de aprendizado de máquina são criadas de forma iterativa e tem fases distintas:

- Preparando dados
- Testando e treinamento de modelos
- Implantando modelos treinados
- Gerenciando modelos de implantados

A Microsoft fornece uma variedade de opções de produto para preparar, compilar, implantar e gerenciar seu modelos de aprendizado de máquina. Compare esses produtos e escolha o que você precisa para desenvolver com mais eficiência soluções de aprendizado de máquina.

## <a name="cloud-based-options"></a>Opções baseadas em nuvem

As seguintes opções estão disponíveis para o aprendizado de máquina na nuvem do Azure.

| Opções&nbsp;de nuvem | O que é | O que você pode fazer com ele |
|-|-|-|
| [Serviço do Azure Machine Learning](#azure-machine-learning-service) | Serviço de nuvem gerenciado para o machine learning  | Treinar, implantar e gerenciar modelos no Azure usando o Python e a CLI |
| [Azure Machine Learning Studio](#azure-machine-learning-studio) | Arraste&ndash;e&ndash;soltar a interface visual para o machine learning | Criar, experimentar e implantar modelos usando algoritmos pré-configurados |

Se você quiser usar modelos de aprendizado de máquina e IA pré-criados [serviços Cognitivos do Azure](#azure-cognitive-services) permite que você adicione facilmente recursos inteligentes aos seus aplicativos.

## <a name="on-premises-options"></a>Opções de local

As opções a seguir estão disponíveis para aprendizado de máquina local. Os servidores locais também podem ser executados em uma máquina virtual na nuvem.

| Opções&nbsp;locais | O que é | O que você pode fazer com ele |
|-|-|-|
| [Serviços de Machine Learning do SQL Server](#sql-server-machine-learning-services) | Mecanismo de análise inserido em SQL | Compilar e implantar modelos dentro do SQL Server |
| [Microsoft Machine Learning Server](#microsoft-machine-learning-server) | Servidor empresarial autônomo para análise de previsão | Criar e implantar modelos de dados pré-processados |

## <a name="development-platforms-and-tools"></a>Ferramentas e plataformas de desenvolvimento

As seguintes plataformas de desenvolvimento e as ferramentas estão disponíveis para o aprendizado de máquina.

| Plataformas/tools | O que é | O que você pode fazer com ele |
|-|-|-|
| [Máquina Virtual de Ciência de Dados do Azure](#azure-data-science-virtual-machine) | Máquina virtual com as ferramentas de ciência de dados previamente instaladas | Desenvolver soluções de aprendizado de máquina em um ambiente de pré-configurado |
| [Azure Databricks](#azure-databricks) | Plataforma de análise com base no Spark | Crie e implante modelos e fluxos de trabalho de dados |
| [ML.NET](#mlnet) | SDK de aprendizado de máquina de software livre, plataforma cruzada | Desenvolver soluções de aprendizado de máquina para aplicativos .NET |
| [Windows ML](#windows-ml) | Plataforma de aprendizado de máquina do Windows 10 | Avaliar modelos treinados em um dispositivo Windows 10 |

## <a name="azure-machine-learning-service"></a>Serviço do Azure Machine Learning

[O serviço do Azure Machine Learning](/azure/machine-learning/service/overview-what-is-azure-ml.md) é um serviço de nuvem totalmente gerenciado usado para treinar, implantar e gerenciar modelos de aprendizado de máquina em escala. Ele dá suporte total a tecnologias de software livre, portanto, você pode usar dezenas de milhares de pacotes do Python de software livre, como TensorFlow, PyTorch e scikit-learn. Ferramentas avançadas também estão disponíveis, como [Azure Notebooks](https://notebooks.azure.com/), [Jupyter Notebooks](http://jupyter.org) ou a extensão do [Azure Machine Learning para Visual Studio Code](https://aka.ms/vscodetoolsforai) para facilitar a exploração e a transformação de dados e, posteriormente, o treinamento e a implantação de modelos. Os serviços do Azure Machine Learning incluem recursos que automatizam a geração e o ajuste de modelo com facilidade, eficiência e precisão.

Use o serviço Azure Machine Learning para treinar, implantar e gerenciar modelos de aprendizado de máquina usando o Python e a CLI em escala de nuvem.

Experimente a [versão gratuita ou paga do Serviço do Azure Machine Learning](http://aka.ms/AMLFree).

|||
|-|-|
|**Tipo**                   |Solução de aprendizado de máquina baseados em nuvem|
|**Idiomas com suporte**    |Python|
|**Fases de aprendizado de máquina**|Preparação dos dados<br>Treinamento do modelo<br>Implantação<br>Gerenciamento|
|**Principais benefícios**           |Gerenciamento central de scripts e histórico de execuções, facilitando a comparação de versões de modelo.<br/><br/>Fácil implantação e gerenciamento de modelos na nuvem ou em dispositivos de borda.|
|**Considerações**         |Exige um pouco de familiaridade com o modelo de gerenciamento de modelos.|

## <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

[O Azure Machine Learning Studio](/azure/machine-learning/studio/) oferece um workspace visual e interativo que você pode usar para compilar, testar e implantar modelos usando algoritmos de aprendizado de máquina predefinidos com rapidez e facilidade. O Machine Learning Studio publica modelos como serviços Web que podem ser facilmente consumidos por aplicativos personalizados ou ferramentas de BI como o Excel.
Nenhuma programação é necessária: você constrói seu modelo de machine learning conectando conjuntos de dados e módulos de análise em uma tela interativa e, em seguida, implanta-o com alguns cliques.

Use o Machine Learning Studio quando quiser desenvolver e implantar modelos sem exigir nenhum código.

Experimente o [Azure Machine Learning Studio](https://studio.azureml.net/?selectAccess=true&o=2&target=_blank), disponível nas opções paga ou gratuita.

|||
|-|-|
|**Tipo**                   |Solução de aprendizado de máquina baseados em nuvem, arrastar e soltar|
|**Idiomas com suporte**    |Python, R|
|**Fases de aprendizado de máquina**|Preparação dos dados<br>Treinamento do modelo<br>Implantação<br>Gerenciamento|
|**Principais benefícios**           |A interface visual interativa possibilita a modelagem de aprendizado de máquina com o mínimo de código.<br/><br/>Jupyter Notebooks interno para exploração de dados.<br/><br/>Implantação direta de modelos treinados como serviços Web do Azure.|
|**Considerações**         |Escalabilidade limitada. O tamanho máximo de um conjunto de dados de treinamento é de 10 GB.<br/><br/>Somente online. Nenhum ambiente de desenvolvimento offline.|

## <a name="azure-cognitive-services"></a>Serviços Cognitivos do Azure

Os [Serviços Cognitivos do Azure](/azure/cognitive-services/welcome) são um conjunto de APIs que permitem criar aplicativos que usam métodos naturais de comunicação. Essas APIs permitem que seus aplicativos vejam, ouçam, falem, entendam e interpretem as necessidades dos usuários com apenas algumas linhas de código. Adicione facilmente recursos inteligentes para seus aplicativos, tais como:

- Detecção de emoção e sentimento
- Visão e reconhecimento de fala
- LUIS (Reconhecimento vocal)
- Conhecimento e pesquisa

Use Serviços Cognitivos para desenvolver aplicativos entre dispositivos e plataformas. As APIs estão em constante aperfeiçoamento e são fáceis de configurar.

|||
|-|-|
|**Tipo**                   |APIs para criação de aplicativos inteligentes|
|**Idiomas com suporte**    |muitas opções, dependendo do serviço|
|**Fases de aprendizado de máquina**|Implantação|
|**Principais benefícios**           |Incorporando recursos de aprendizado de máquina em aplicativos que usam modelos previamente treinados.<br/><br/>Variedade de modelos para os métodos de comunicação natural com a visão e fala.|
|**Considerações**         |Modelos previamente treinados e não são personalizáveis.|

## <a name="sql-server-machine-learning-services"></a>Serviços de Machine Learning do SQL Server

O [Serviço do Microsoft Machine Learning para SQL Server](https://docs.microsoft.com/sql/advanced-analytics/r/r-services) adiciona análise estatística, visualização de dados e análise preditiva em R e Python para dados relacionais em bancos de dados do SQL Server. Bibliotecas de R e Python da Microsoft incluem modelos avançados e algoritmos de aprendizado de máquina, que podem ser executados em paralelo e em escala, no SQL Server.

Use os Serviços de Machine Learning do SQL Server quando precisar de IA integrada e análise preditiva em dados relacionais no SQL Server.

|||
|-|-|
|**Tipo**                   |Análise preditiva para dados relacionais locais|
|**Idiomas com suporte**    |Python, R|
|**Fases de aprendizado de máquina**|Preparação dos dados<br>Treinamento do modelo<br>Implantação|
|**Principais benefícios**           |Encapsule a lógica preditiva em uma função de banco de dados, facilitando sua inclusão na lógica da camada de dados.|
|**Considerações**         |Pressupõe um banco de dados do SQL Server como a camada de dados para o aplicativo.|

## <a name="microsoft-machine-learning-server"></a>Microsoft Machine Learning Server

[O Microsoft Machine Learning Server](https://docs.microsoft.com/machine-learning-server/what-is-machine-learning-server) é um servidor empresarial para hospedagem e gerenciamento de cargas de trabalho paralelas e distribuídas de processos do R e do Python. O Microsoft Machine Learning Server é executado em Windows, Linux, Hadoop e Apache Spark e também está disponível no [HDInsight](https://azure.microsoft.com/services/hdinsight/r-server/). Ele fornece um mecanismo de execução para soluções criadas usando [pacotes RevoScaleR](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/revoscaler), [revoscalepy](https://docs.microsoft.com/machine-learning-server/python-reference/revoscalepy/revoscalepy-package) e [MicrosoftML](https://docs.microsoft.com/r-server/r/concept-what-is-the-microsoftml-package), e estende R e Python de software livre com suporte para análise de alto desempenho, análise estatística, aprendizado de máquina e conjuntos de dados realmente grandes. Essa funcionalidade agregada é fornecida por meio de pacotes de proprietários que são instalados com o servidor. Para desenvolvimento, você pode usar IDEs como [Ferramentas do R para Visual Studio](https://www.visualstudio.com/vs/rtvs/) e [Ferramentas Python para Visual Studio](https://www.visualstudio.com/vs/python/).

Use o Microsoft Machine Learning Server quando precisar compilar e operacionalizar modelos criados com R e Python em um servidor ou distribuir o treinamento de R e Python em grande escala em um cluster Hadoop ou Spark.

|||
|-|-|
|**Tipo**                   |Local enterprise server para análise preditiva|
|**Idiomas com suporte**    |Python, R|
|**Fases de aprendizado de máquina**|Treinamento do modelo<br>Implantação|
|**Principais benefícios**           |Alta escalabilidade.|
|**Considerações**         |Você precisa implantar e gerenciar o Machine Learning Server em sua empresa.|

## <a name="azure-data-science-virtual-machine"></a>Máquina Virtual da Ciência de Dados do Azure

A [Máquina Virtual de Ciência de Dados do Azure](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/overview) é um ambiente de máquina virtual personalizada na nuvem do Microsoft Azure, especificamente criada para ciência de dados. Ela tem muitas ferramentas conhecidas de ciência de dados, entre outras, pré-instaladas e pré-configuradas que ajudam a começar a criar rapidamente aplicativos inteligentes para análise avançada.

Há suporte para a Máquina Virtual de Ciência de Dados como um destino para os serviços do Azure Machine Learning.
Está disponível nas versões para Windows e Linux Ubuntu (não há suporte para os serviços do Azure Machine Learning no Linux CentOS).
Para obter informações de versão específica e uma lista do que foi incluído, veja [Introdução à Máquina Virtual de Ciência de Dados do Azure](/azure/machine-learning/data-science-virtual-machine/overview.md).

Use a VM de Ciência de Dados quando você precisar executar ou hospedar seus trabalhos em um único nó. Ou então, se você precisar expandir remotamente o processamento em um único computador.

|||
|-|-|
|**Tipo**                   |Ambiente de máquina virtual personalizada para ciência de dados|
|**Principais benefícios**           |Redução do tempo para instalar, gerenciar e solucionar problemas de estruturas e ferramentas de ciência de dados.<br/><br/>As últimas versões de todas as ferramentas e estruturas usadas são incluídas.<br/><br/>As opções de máquina virtual incluem imagens altamente escalonáveis com funcionalidades de GPU para modelagem de dados intensiva.|
|**Considerações**         |A máquina virtual não poderá ser acessada quando você estiver offline.<br/><br/>A execução de uma máquina virtual incorre em encargos do Azure e, portanto, você deve ter cuidado para que ela seja executada somente quando necessário.|

## <a name="azure-databricks"></a>Azure Databricks

O [Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) é uma plataforma de análise baseada no Apache Spark otimizada para a plataforma de Serviços de Nuvem do Microsoft Azure. O Databricks é integrado com o Azure para fornecer instalação com um clique, fluxos de trabalho simplificados e um workspace interativo que permite a colaboração entre os cientistas de dados, os engenheiros de dados e os analistas empresariais.
Use código R, Python, Scala e SQL em blocos de anotações baseado na Web para consultar, visualizar e modelar dados.

Use o Databricks quando quiser colaborar na criação de soluções de aprendizado de máquina no Apache Spark.

|||
|-|-|
|**Tipo**                   |Plataforma de análise com base no Apache Spark|
|**Idiomas com suporte**    |Python, R, Scala, SQL|
|**Fases de aprendizado de máquina**|Consulta de dados<br>Treinamento do modelo|

## <a name="mlnet"></a>ML.NET

[ML.NET](https://docs.microsoft.com/dotnet/machine-learning/) é uma estrutura de aprendizado de máquina gratuita, de software livre e multiplataforma que permite que você crie soluções de aprendizado de máquina personalizadas e integre-as em seus aplicativos .NET.

Use ML.NET quando quiser integrar soluções de aprendizado de máquina a seus aplicativos .NET.

|||
|-|-|
|**Tipo**                   |Estrutura do código-fonte aberto para o desenvolvimento de aplicativos de aprendizado de máquina personalizada|
|**Idiomas com suporte**    |.NET|

## <a name="windows-ml"></a>Windows ML

[Windows ML](https://docs.microsoft.com/windows/uwp/machine-learning/) mecanismo de inferência de tipos permite que você use modelos em seus aplicativos de aprendizado, avaliar modelos treinados localmente em dispositivos Windows 10 de máquina treinado.

Use o Windows ML quando quiser usar modelos de aprendizado de máquina treinados em seus aplicativos do Windows.

|||
|-|-|
|**Tipo**                   |Mecanismo de inferência para modelos treinados de dispositivos do Windows|
|**Idiomas com suporte**    |C#/C++, JavaScript|

## <a name="next-steps"></a>Próximas etapas

- Para saber mais sobre todos os produtos de desenvolvimento de IA (Inteligência Artificial) disponíveis na Microsoft, veja [Plataforma de IA da Microsoft](https://www.microsoft.com/ai)
- Para obter treinamento sobre como desenvolver soluções de IA, veja a [Microsoft AI School](https://aischool.microsoft.com/learning-paths)
