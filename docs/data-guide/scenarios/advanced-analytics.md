---
title: Análise avançada
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 31ba357fe37b1de35a6eea324d2d1d6766e172e5
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/03/2018
ms.locfileid: "30298684"
---
# <a name="advanced-analytics"></a>Análise avançada

A análise avançada vai além dos relatórios históricos e da agregação de dados de BI (business intelligence) tradicional e usa técnicas de modelagem estatística, matemática e probabilística para permitir o processamento preditivo e a tomada de decisões automatizada.

Normalmente, as soluções de análise avançada envolvem as seguintes cargas de trabalho:

* Visualização e exploração de dados interativa
* Treinamento de modelo de Aprendizado de Máquina
* Processamento preditivo em tempo real ou em lote

A maioria das arquiteturas de análise avançada inclui alguns ou todos os seguintes componentes:

* **Armazenamento de dados**. As soluções de análise avançada exigem que os dados treinem modelos de aprendizado de máquina. Os cientistas de dados normalmente precisam explorar os dados para identificar seus recursos preditivos e as relações estatísticas entre eles, bem como os valores que eles preveem (conhecidos como um rótulo). O rótulo previsto pode ser um valor quantitativo, como o valor financeiro de algo no futuro ou a duração de um atraso de voo em minutos. Ou pode representar uma classe categórica, como "verdadeiro" ou "falso", "atraso de voo" ou "sem atraso de voo" ou categorias, como "baixo risco", "médio risco" ou "alto risco".

* **Processamento em lotes**. Para treinar um modelo de aprendizado de máquina, normalmente, você precisa processar um grande volume de dados de treinamento. O treinamento do modelo pode levar algum tempo (na ordem de minutos a horas). Esse treinamento pode ser realizado com scripts escritos em linguagens como o Python ou o R e pode ser dimensionado para reduzir o tempo de treinamento usando plataformas de processamento distribuído, como o Apache Spark hospedado no HDInsight ou um contêiner do Docker.

* **Ingestão de mensagens em tempo real**. Em produção, muitas análises avançadas alimentam fluxos de dados em tempo real para um modelo preditivo que foi publicado como um serviço Web. O fluxo de dados de entrada normalmente é capturado em alguma forma de fila e um mecanismo de processamento de fluxo efetua pull dessa fila e aplica a previsão aos dados de entrada quase em tempo real.  

* **Processamento de fluxo**. Depois que você tiver um modelo treinado, a previsão (ou pontuação) costuma ser uma operação muito rápida (na ordem de milissegundos) para determinado conjunto de recursos. Depois de capturar mensagens em tempo real, os valores de recursos relevantes podem ser passados para o serviço preditivo para gerar um rótulo previsto.

* **Armazenamento de dados analíticos**. Em alguns casos, os valores de rótulos previstos são gravados no armazenamento de dados analíticos para relatórios e análise futura.

* **Análise e relatórios**. Como o nome sugere, as soluções de análise avançada geralmente produzem algum tipo de relatório ou feed analítico que inclui valores de dados previstos. Geralmente, os valores de rótulo previstos são usados para popular painéis em tempo real.

* **Orquestração**. Embora a exploração e a modelagem iniciais de dados sejam executadas de maneira interativa por cientistas de dados, muitas soluções de análise avançada periodicamente treinam os modelos novamente com novos dados &mdash; refinando a precisão dos modelos de modo contínuo. Esse novo treinamento pode ser automatizado com um fluxo de trabalho orquestrado.

## <a name="machine-learning"></a>Aprendizado de máquina
O aprendizado de máquina é uma técnica de modelagem matemática usada para treinar um modelo preditivo. O princípio geral é aplicar um algoritmo estatístico a um conjunto grande de dados históricos para descobrir as relações entre os campos que ele contém.

A modelagem de aprendizado de máquina geralmente é feita por cientistas de dados, que precisam explorar por completo e preparar os dados antes de treinar um modelo. Essa exploração e preparação costumam envolver uma grande quantidade de visualização e análise de dados interativa &mdash; geralmente, usando linguagens como o Python e o R em ferramentas interativas e ambientes projetados especificamente para essa tarefa.

Em alguns casos, talvez você possa usar [modelos pré-treinados](/machine-learning-server/install/microsoftml-install-pretrained-models) que acompanham os dados de treinamento obtidos e desenvolvidos pela Microsoft. A vantagem de modelos pré-treinados é que você pode pontuar e classificar o novo conteúdo imediatamente, mesmo se não tem os dados de treinamento necessários, os recursos para gerenciar conjuntos grandes de dados ou treinar modelos complexos.

Há duas categorias amplas de aprendizado de máquina:

* **Aprendizado supervisionado**. O aprendizado supervisionado é a abordagem mais comum usada pelo aprendizado de máquina. Em um modelo de aprendizado supervisionado, os dados de origem consistem em um conjunto de campos de dados de *recurso* que têm uma relação matemática com um ou mais campos de dados de *rótulo*. Durante a fase de treinamento do processo de aprendizado de máquina, o conjunto de dados inclui recursos e rótulos conhecidos e um algoritmo é aplicado de acordo com uma função que opera nos recursos para calcular as previsões de rótulo correspondentes. Normalmente, um subconjunto do conjunto de dados de treinamento é retido e usado para validar o desempenho do modelo treinado. Depois que o modelo foi treinado, ele pode ser implantado em produção e ser usado para prever valores desconhecidos. 

* **Aprendizado não supervisionado**. Em um modelo de aprendizado não supervisionado, os dados de treinamento não incluem valores de rótulo conhecidos. Em vez disso, o algoritmo faz suas previsões com base em sua primeira exposição aos dados. A forma mais comum de aprendizado não supervisado é o *clustering*, em que o algoritmo determina a melhor maneira de dividir os dados em um número especificado de clusters com base nas semelhanças estatísticas dos recursos. No clustering, o resultado previsto é o número do cluster ao qual pertencem os recursos de entrada. Embora, às vezes, eles possam ser usados diretamente para gerar previsões úteis, como o uso do clustering para identificar grupos de usuários em um banco de dados de clientes, abordagens de aprendizado não supervisado são usados com mais frequência para identificar quais dados são mais úteis de serem fornecidos a um algoritmo de aprendizado supervisionado no treinamento de um modelo.

Serviços do Azure relevantes:

- [Azure Machine Learning](/azure/machine-learning/)
- [Machine Learning Server (R Server) no HDInsight](/azure/hdinsight/r-server/r-server-overview)

## <a name="deep-learning"></a>Aprendizado

Os modelos de aprendizado de máquina baseados em técnicas matemáticas como regressão linear ou logística estão disponíveis por algum tempo. Mais recentemente, houve um aumento do uso de técnicas de *aprendizado profundo* com base em redes neurais. Isso é impulsionado em parte pela disponibilidade de sistemas de processamento altamente escalonáveis que reduzem o tempo que leva para treinar modelos complexos. Além disso, o aumento da prevalência de Big Data facilita o treinamento de modelos de aprendizado profundo em uma variedade de domínios.

Ao criar uma arquitetura de nuvem para análise avançada, você deve considerar a necessidade de processamento em larga escala de modelos de aprendizado profundo. Eles podem ser fornecidos por meio de plataformas de processamento distribuído como o Apache Spark e a última geração de máquinas virtuais que incluem o acesso ao hardware da GPU.

Serviços do Azure relevantes:

- [Máquina Virtual de Aprendizado Profundo](/azure/machine-learning/data-science-virtual-machine/deep-learning-dsvm-overview)
- [Apache Spark no HDInsight](/azure/hdinsight/spark/apache-spark-overview)

## <a name="artificial-intelligence"></a>Inteligência artificial

A IA (inteligência artificial) refere-se a cenários em que um computador simula as funções cognitivas associadas às mentes humanas, como o aprendizado e a solução de problemas. Como o IA utiliza algoritmos de aprendizado de máquina, ele é considerado um termo abrangente. A maioria das soluções de IA conta com uma combinação de serviços preditivos, geralmente implementados como serviços Web e interfaces de idioma natural, como chatbots que interagem por meio de texto ou fala, apresentados por aplicativos de IA em execução em dispositivos móveis ou outros clientes. Em alguns casos, o modelo de aprendizado de máquina é inserido com o aplicativo de IA. 

## <a name="model-deployment"></a>Implantação de modelo

Os serviços preditivos que dão suporte à aplicativos de IA podem utilizar modelos de aprendizado de máquina personalizados ou serviços cognitivos prontos para uso que fornecem acesso a modelos pré-treinados. O processo de implantação de modelos personalizados em produção é conhecido como operacionalização, em que os mesmos modelos de IA treinados e testados no ambiente de processamento são serializados e disponibilizados para aplicativos e serviços externos para previsões em lote ou de autoatendimento. Para usar a funcionalidade preditiva do modelo, ele é desserializado e carregado usando a mesma biblioteca de aprendizado de máquina que contém o algoritmo que foi usado para treinar o modelo em primeiro lugar. Essa biblioteca fornece funções preditivas (geralmente chamadas de pontuação ou previsão) que usam o modelo e os recursos como entrada e retornam a previsão. Essa lógica é então encapsulada em uma função que um aplicativo pode chamar diretamente ou pode ser exposta como um serviço Web. 

Serviços do Azure relevantes:

- [Azure Machine Learning](/azure/machine-learning/)
- [Machine Learning Server (R Server) no HDInsight](/azure/hdinsight/r-server/r-server-overview)


## <a name="see-also"></a>Consulte também

- [Escolhendo uma tecnologia de serviços cognitivos](../technology-choices/cognitive-services.md)
- [Escolhendo uma tecnologia de aprendizado de máquina](../technology-choices/data-science-and-machine-learning.md)
