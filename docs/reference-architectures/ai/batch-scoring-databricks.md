---
title: Pontuação em lote dos modelos do Spark no Azure Databricks
description: Crie uma solução escalonável para pontuar em lote um modelo de classificação do Apache Spark usando o Azure Databricks.
author: njray
ms.date: 02/07/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: cba8d272ddbdbf2c2da94f68b288e9fb79be7de2
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/18/2019
ms.locfileid: "59740350"
---
# <a name="batch-scoring-of-spark-machine-learning-models-on-azure-databricks"></a>Modelos de pontuação do lote do machine learning do Spark no Azure Databricks

Essa arquitetura de referência mostra como compilar uma solução escalonável para pontuação em lote no modelo de classificação do Apache Spark em um cronograma usando o Azure Databricks, uma plataforma analítica baseada no Apache Spark otimizada para o Azure. A solução pode ser usada como um modelo que pode ser generalizado para outros cenários.

Há uma implantação de referência para essa arquitetura disponível no  [GitHub][github].

![Pontuação em lote dos modelos do Spark no Azure Databricks](./_images/batch-scoring-spark.png)

**Cenário**: Uma empresa em um setor de ativos pesados deseja minimizar os custos e o tempo de inatividade associado a falhas mecânicas inesperadas. Usando dados de IoT coletadas dos seus computadores, podem criar um modelo de manutenção preditiva. Esse modelo permite que os negócios mantenham componentes de forma proativa e repara-os antes de falharem. Ao maximizar o uso de componente mecânico, eles podem controlar os custos e reduzir o tempo de inatividade.

Um modelo de manutenção preditiva coleta dados das máquinas e retém os exemplos de histórico de falhas de componente. O modelo, em seguida, pode ser usado para monitorar o estado atual dos componentes e prever se um determinado componente falhará em um futuro próximo. Para casos de uso comum e abordagens de modelação, consulte [Guia de IA do Azure para soluções de manutenção preditiva][ai-guide].

Essa arquitetura de referência foi projetada para cargas de trabalho que são disparadas pela presença de novos dados de máquinas de componente. O processamento envolve as seguintes etapas:

1. A ingestão de dados de armazenamento de dados externos no armazenamento de dados do Azure Databricks.

2. Treine um modelo de aprendizado de máquina transformando um conjunto de dados de treinamento, em seguida, crie um modelo do Spark MLlib. O MLlib consiste em algoritmos de aprendizado de máquina e utilitários otimizados para usufruir os recursos de escalabilidade de dados do Spark mais comuns.

3. Aplique o modelo treinado para prever (classificar) as falhas de componente, transformando os dados em um conjunto de dados de pontuação. Pontue os dados com o modelo do Spark MLLib.

4. Armazene resultados no armazenamento de dados de Databricks para consumo de pós-processamento.

Notebooks são fornecidos no [GitHub] [ github] para executar cada uma dessas tarefas.

## <a name="architecture"></a>Arquitetura

A arquitetura define um fluxo de dados que está inteiramente contido dentro do [Azure Databricks][databricks] com base em um conjunto de [notebooks][notebooks] executados sequencialmente. Isso consiste nos seguintes componentes:

**[Arquivos de dados][github]**. A implementação de referência usa um conjunto de dados simulados contido em cinco arquivos de dados estáticos.

**[Ingestão][notebooks]**. O notebook de ingestão de dados baixa os arquivos de dados de entrada em uma coleção de conjuntos de dados do Databricks. Em um cenário do mundo real, dados de dispositivos IoT iriam para o armazenamento acessível de Databricks como Azure SQL Server ou Armazenamento de Blobs do Azure. O Databricks dá suporte a várias [fontes de dados][data-sources].

**Pipeline de treinamento**. Este notebook executa o notebook de engenharia de recursos para criar um conjunto de dados de análise de dados ingeridos. Depois, executará um notebook de criação de modelo que treina o modelo de machine learning usando a biblioteca de aprendizado de máquina executa do [Apache Spark MLlib][mllib].

**Pipeline de pontuação**. Este notebook executa o notebook de engenharia de recursos para criar o conjunto de dados de pontuação dos dados ingeridos e executa notebook de pontuação. O notebook de pontuação usa o modelo [Spark MLlib][mllib-spark] para gerar previsões para as observações no conjunto de dados de pontuação. As previsões são armazenadas no repositório de resultados, um novo conjunto de dados no repositório de dados do Databricks.

**Agendador**. Um trabalho do Databricks [agendado][job] lida com pontuação em lote com o modelo Spark. O trabalho executa o notebook de pipeline de pontuação, passando argumentos variáveis por meio de parâmetros de bloco de anotações para especificar os detalhes para a construção de conjunto de dados de pontuação e onde armazenar o conjunto de dados de resultados.

O cenário é criado como um fluxo de pipeline. Cada notebook é otimizado para executar em uma configuração de lote para cada uma das operações: ingestão, engenharia de recursos, criação de modelo e pontuações do modelo. Para fazer isso, o notebook de engenharia de recursos foi projetado para gerar um conjunto de dados geral para qualquer operação de treinamento, calibração teste ou pontuação. Nesse cenário, usamos uma estratégia de divisão temporal para essas operações, portanto, os parâmetros de notebooks são usados para definir a filtragem do intervalo de datas.

Como o cenário cria um pipeline de lote, fornecemos um conjunto de notebooks de exame opcionais para explorar a saída dos notebooks de pipeline. Você pode encontrar esses no repositório do GitHub:

- `1a_raw-data_exploring`
- `2a_feature_exploration`
- `2b_model_testing`
- `3b_model_scoring_evaluation`

## <a name="recommendations"></a>Recomendações

O Databricks está configurado para que você possa carregar e implantar seus modelos treinados para fazer previsões com novos dados. Usamos o Databricks para esse cenário porque fornece essas vantagens adicionais:

- Suporte a logon único usando as credenciais do Azure Active Directory.
- Agendador de trabalhos para executar os trabalhos para os pipelines de produção.
- O notebook completamente interativo com colaboração, painéis e APIS de REST.
- Clusters ilimitados que podem ser dimensionados para qualquer tamanho.
- Segurança avançada, controles de acesso baseado em função e os logs de auditoria.

Para interagir com o serviço do Azure Databricks, use o Databricks [Espaço de trabalho][workspace] interface em um navegador da web ou a [interface de linha de comando][cli] (CLI). Acesse a CLI do Databricks de qualquer plataforma que dá suporte ao Python 2.7.9 a 3.6.

A implementação de referência usa os [notebooks][notebooks] para executar tarefas em sequência. Cada notebook armazena artefatos de dados intermediários (treinamento, teste, de pontuação ou conjuntos de resultados de dados) para o mesmo armazenamento de dados como dados de entrada. O objetivo é tornar mais fácil para que você possa usá-lo conforme necessário no seu caso de uso específico. Na prática, se conectar a fonte de dados à instância do Azure Databricks aos notebooks para ler e gravar diretamente de volta no seu armazenamento.

Você pode monitorar a execução do trabalho por meio da interface do usuário Databricks, o armazenamento de dados ou a [CLI][cli] do Databricks conforme necessário. Monitorar o cluster usando o [log de eventos][log] e outras [métricas][ metrics] que o Databricks fornece.

## <a name="performance-considerations"></a>Considerações sobre o desempenho

Um cluster do Azure Databricks habilita o dimensionamento automático por padrão, de modo que, durante o tempo de execução, o Databricks realoca dinamicamente os trabalhadores para levar em conta as características do seu trabalho. Certas partes do seu pipeline podem ser mais exigentes que outras. O Databricks adiciona trabalhadores adicionais durante essas fases do seu trabalho (e remove-os quando não forem mais necessários). Dimensionamento automático torna mais fácil de obter alta [utilização do cluster][cluster], porque você não precisa provisionar o cluster de acordo com uma carga de trabalho.

Além disso, os pipelines agendados mais complexos podem ser desenvolvidos usando o [Azure Data Factory][adf] com o Azure Databricks.

## <a name="storage-considerations"></a>Considerações de armazenamento

Nessas implementações de referência, os dados são armazenados diretamente no armazenamento do Databricks para manter a simplicidade. Em um ambiente de produção, no entanto, os dados podem ser armazenados no armazenamento de dados de nuvem, como [Armazenamento de Blob do Azure][blob]. [Databricks][databricks-connect] também dá suporte ao Azure Data Lake Store, Azure SQL Data Warehouse, Azure Cosmos DB, Apache Kafka e Hadoop.

## <a name="cost-considerations"></a>Considerações de custo

O Azure Databricks é um Spark premium oferecendo com um custo associado. Além disso, [tipos de preço][pricing] de Databricks standard e premium.

Para este cenário, o tipo de preço standard é suficiente. No entanto, se seu aplicativo específico requer que o dimensionamento automático de clusters lide com cargas de trabalho maiores ou painéis interativos do Databricks, o nível premium pode aumentar ainda mais os custos.

Os notebooks de solução podem ser executados em qualquer plataforma com base no Spark com edições mínimas para remover os pacotes específicos do Databricks. Consulte as seguintes soluções semelhantes para várias plataformas do Azure:

- [Python no Azure Machine Learning Studio][python-aml]
- [Serviços de R do SQL Server][sql-r]
- [PySpark em uma Máquina Virtual de Ciência de Dados do Azure][py-dvsm]

## <a name="deploy-the-solution"></a>Implantar a solução

Para implantar essa arquitetura de referência, siga as etapas descritas no repositório  [GitHub][github] para criar uma solução escalonável para modelos de pontuação Spark em lote no Azure Databricks.

## <a name="related-architectures"></a>Arquiteturas relacionadas

Também criamos uma arquitetura de referência que usa o Spark para construir [sistemas de recomendação em tempo real] [ recommendation] com pontuações offline, pré-computadas. Esses sistemas de recomendação são cenários comuns em que as pontuações são processadas em lote.

[adf]: https://azure.microsoft.com/blog/operationalize-azure-databricks-notebooks-using-data-factory/
[ai-guide]: /azure/machine-learning/team-data-science-process/cortana-analytics-playbook-predictive-maintenance
[blob]: https://docs.databricks.com/spark/latest/data-sources/azure/azure-storage.html
[cli]: https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html
[cluster]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html
[databricks]: /azure/azure-databricks/
[databricks-connect]: /azure/azure-databricks/databricks-connect-to-data-sources
[data-sources]: https://docs.databricks.com/spark/latest/data-sources/index.html
[github]: https://github.com/Azure/BatchSparkScoringPredictiveMaintenance
[job]: https://docs.databricks.com/user-guide/jobs.html
[log]: https://docs.databricks.com/user-guide/clusters/event-log.html
[metrics]: https://docs.databricks.com/user-guide/clusters/metrics.html
[mllib]: https://docs.databricks.com/spark/latest/mllib/index.html
[mllib-spark]: https://docs.databricks.com/spark/latest/mllib/index.html#apache-spark-mllib
[notebooks]: https://docs.databricks.com/user-guide/notebooks/index.html
[pricing]: https://azure.microsoft.com/en-us/pricing/details/databricks/
[python-aml]: https://gallery.azure.ai/Notebook/Predictive-Maintenance-Modelling-Guide-Python-Notebook-1
[py-dvsm]: https://gallery.azure.ai/Tutorial/Predictive-Maintenance-using-PySpark
[recommendation]: /azure/architecture/reference-architectures/ai/real-time-recommendation
[sql-r]: https://gallery.azure.ai/Tutorial/Predictive-Maintenance-Modeling-Guide-using-SQL-R-Services-1
[workspace]: https://docs.databricks.com/user-guide/workspace.html
