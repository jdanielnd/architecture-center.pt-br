---
title: Análise de dados e de IoT no setor da construção
description: Use dispositivos de IoT e de análise de dados para fornecer gerenciamento e operação abrangentes de projetos de construção.
author: alexbuckgit
ms.date: 08/29/2018
ms.openlocfilehash: 7ab0de50b0eba1ab420e450f3408fe5dc45f04ac
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/05/2018
ms.locfileid: "48818489"
---
# <a name="iot-and-data-analytics-in-the-construction-industry"></a>Análise de dados e de IoT no setor da construção

Esse cenário de exemplo é relevante para organizações que desenvolvem soluções que integram dados de vários dispositivos de IoT em uma arquitetura de análise de dados abrangente para melhorar e automatizar a tomada de decisões. Alguns aplicativos potenciais são mineração, construção, fabricação ou outras soluções do setor que envolvem grandes volumes de dados de muitas entradas de dados com base em IoT.

Nesse cenário, um fabricante de equipamento de construção constrói veículos, medidores e drones que usam tecnologias de IoT e GPS para emitir dados de telemetria. A empresa deseja modernizar sua arquitetura de dados para monitorar melhor a integridade do equipamento e as condições de operação. A substituição da solução herdada da empresa usando a infraestrutura local seria demorada e trabalhosa, e não seria possível realizar um dimensionamento suficiente para lidar com o volume de dados antecipado.

A empresa quer criar uma solução baseada em nuvem de "construção inteligente". Ela deve reunir um conjunto abrangente de dados para um site de construção e automatizar a operação e a manutenção dos vários elementos do site. As metas da empresa incluem:

* Integrar e analisar todos o equipamento e os dados do site de construção para minimizar o tempo de inatividade do equipamento e reduzir os roubos.
* Controlar de forma remota e automática o equipamento de construção para mitigar os efeitos da falta de mão de obra e, em última análise, exigir menos trabalhadores e permitir que trabalhadores menos capacitados tenham êxito.
* Minimizar os requisitos de mão de obra e os custos operacionais para a infraestrutura de suporte e, ao mesmo tempo, aumentar a produtividade e a segurança.
* Dimensionar a infraestrutura com facilidade para dar suporte a aumentos nos dados de telemetria.
* Manter a conformidade com todos os requisitos legais relevantes por meio do provisionamento de recursos no país, sem comprometer a disponibilidade do sistema.
* Usar software livre para maximizar o investimento nas habilidades atuais dos trabalhadores.

O uso de serviços gerenciados do Azure, como o Hub IoT e o HDInsight, permitirá que o cliente crie e implante rapidamente uma solução abrangente com um custo operacional menor. Se tiver necessidades de análise de dados adicionais, você deverá examinar a lista de [serviços de análise de dados totalmente gerenciados no Azure][product-category] disponíveis.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Considere usar a solução nos casos de uso abaixo:

* Cenários de construção, mineração ou produção de equipamento
* Coleta em grande escala de dados de dispositivos para armazenamento e análise
* Ingestão e análise de grandes conjuntos de dados

## <a name="architecture"></a>Arquitetura

![Arquitetura para análise de dados e IoT no setor da construção][architecture]

Os dados fluem pela solução da seguinte maneira:

1. O equipamento de construção coleta dados de sensores e envia os dados de resultados de construção a intervalos regulares para serviços Web com balanceamento de carga hospedados em um cluster de máquinas virtuais do Azure.
2. Os serviços Web personalizados ingerem os dados de resultados de construção e os armazenam em um cluster do Apache Cassandra também em execução em máquinas virtuais do Azure.
3. Outro conjunto de dados é coletado pelos sensores de IoT em vários equipamentos de construção e enviado ao Hub IoT.
4. Os dados brutos coletados são enviados diretamente do Hub IoT para o Armazenamento de Blobs do Azure e são disponibilizados imediatamente para exibição e análise.
5. Os dados coletados por meio do Hub IoT são processados em tempo real por um trabalho do Azure Stream Analytics e armazenados em um banco de dados SQL do Azure.
6. O aplicativo Web Smart Construction Cloud está disponível para que analistas e usuários finais exibam e analisem dados de sensores e imagens. 
7. Trabalhos em lote são iniciados sob demanda por usuários do aplicativo Web. O trabalho em lote é executado no Apache Spark no HDInsight e analisa novos dados armazenados no cluster Cassandra. 

### <a name="components"></a>Componentes

* O [Hub IoT](/azure/iot-hub/about-iot-hub) atua como um hub de mensagens central para comunicação bidirecional segura com identidade por dispositivo entre a plataforma de nuvem e o equipamento de construção e outros elementos do site. O Hub IoT pode coletar dados rapidamente para cada dispositivo para a ingestão no pipeline de análise de dados. 
* O [Azure Stream Analytics](/azure/stream-analytics/stream-analytics-introduction) é um mecanismo de processamento de eventos que permite examinar grandes volumes de fluxo de dados de dispositivos e de outras fontes de dados. Ele também oferece suporte à extração de informações dos fluxos de dados para identificar padrões e relações. Nesse cenário, o Stream Analytics ingere e analisa dados de dispositivos IoT e armazena os resultados no Banco de Dados SQL do Azure. 
* O [Banco de Dados SQL do Azure](/azure/sql-database/sql-database-technical-overview) contém os resultados dos dados analisados de dispositivos IoT e medidores, que podem ser exibidos por analistas e usuários por meio de um aplicativo Web baseado no Azure. 
* O [Armazenamento de Blobs](/azure/storage/blobs/storage-blobs-introduction) armazena dados de imagem coletados dos dispositivos de hub IoT. Os dados de imagem podem ser exibidos por meio do aplicativo Web.
* O [Gerenciador de Tráfego](/azure/traffic-manager/traffic-manager-overview) controla a distribuição de tráfego do usuário para pontos de extremidade de serviço em diferentes regiões do Azure.
* O [Load Balancer](/azure/load-balancer/load-balancer-overview) distribui os envios de dados de dispositivos de equipamento de construção entre os serviços Web baseados em VM para fornecer alta disponibilidade.
* [Máquinas Virtuais do Azure](/azure/virtual-machines) hospedam os serviços Web que recebem e ingerem os dados de resultados de construção no banco de dados Apache Cassandra.
* O [Apache Cassandra](https://cassandra.apache.org) é um banco de dados NoSQL distribuído usado para armazenar dados de construção para processamento posterior pelo Apache Spark.
* [Aplicativos Web](/azure/app-service/app-service-web-overview) hospedam o aplicativo Web do usuário final, que pode ser usado para consultar e exibir imagens e dados de origem. Os usuários também podem iniciar trabalhos em lote no Apache Spark por meio do aplicativo.
* O [Apache Spark no HDInsight](/azure/hdinsight/spark/apache-spark-overview) dá suporte ao processamento na memória para melhorar o desempenho de aplicativos de análise de Big Data. Nesse cenário, o Spark é usado para executar algoritmos complexos em relação aos dados armazenados no Apache Cassandra.


### <a name="alternatives"></a>Alternativas

* O [Cosmos DB](/azure/cosmos-db/introduction) é uma tecnologia de banco de dados NoSQL alternativo. O Cosmos DB fornece [suporte de vários mestres em escala global](/azure/cosmos-db/multi-region-writers) com [vários níveis de consistência bem definidos](/azure/cosmos-db/consistency-levels) para atender a vários requisitos do cliente. Ele também dá suporte à [API do Cassandra](/azure/cosmos-db/cassandra-introduction). 
* O [Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) é uma plataforma de análise com base no Apache Spark otimizada para o Azure. Ele é integrado ao Azure para fornecer configuração com um clique, fluxos de trabalho simplificados e um espaço de trabalho colaborativo interativo.
* O [Data Lake Storage](/azure/storage/data-lake-storage) é uma alternativa ao Armazenamento de Blobs. Para este cenário, o Data Lake Storage não estava disponível na região de destino.
* [Aplicativos Web](/azure/app-service) também podem ser usados para hospedar os serviços Web para ingerir dados de resultados de construção.
* Muitas opções de tecnologia estão disponíveis para ingestão de mensagens em tempo real, armazenamento de dados, processamento de fluxo, armazenamento de dados analíticos, análise e relatórios. Para obter uma visão geral dessas opções, seus recursos e os principais critérios de seleção, confira [Arquiteturas de Big Data: processamento em tempo real](/azure/architecture/data-guide/technology-choices/real-time-ingestion) no [Guia de Arquitetura de Dados do Azure](/azure/architecture/data-guide).

## <a name="considerations"></a>Considerações

A ampla disponibilidade de regiões do Azure é um fator importante para esse cenário. Ter mais de uma região em um único país pode fornecer recuperação de desastre, além de habilitar a conformidade com requisitos de imposição da lei e obrigações contratuais. A comunicação de alta velocidade do Azure entre regiões também é um fator importante nesse cenário.

O suporte do Azure para tecnologias de software livre permitiu que o cliente aproveitasse as habilidades existentes da força de trabalho. O cliente também pode acelerar a adoção de novas tecnologias com custos mais baixos e cargas de trabalho operacionais, em comparação com uma solução local. 

## <a name="pricing"></a>Preços

As considerações a seguir vão orientar uma parte significativa dos custos para esta solução.

* Os custos da máquina virtual do Azure aumentarão de forma linear conforme novas instâncias forem provisionadas. As máquinas virtuais desalocadas só geram custos de armazenamento, não de computação. Essas máquinas desalocadas poderão ser realocadas posteriormente, quando houver alta demanda.
* Os custos do [Hub IoT](https://azure.microsoft.com/pricing/details/iot-hub) são gerados pelo número de unidades de IoT provisionadas, bem como pela camada de serviço escolhida, que determina o número de mensagens por dia permitido por unidade. 
* O [Stream Analytics](https://azure.microsoft.com/pricing/details/stream-analytics) é cobrado por número de unidades de streaming exigidas para processar os dados no serviço.

## <a name="related-resources"></a>Recursos relacionados

Para ver uma implementação de uma arquitetura semelhante, leia a [história do cliente Komatsu][customer-story].

Diretrizes para arquiteturas de big data estão disponíveis no [Guia de Arquitetura de Dados do Azure](/azure/architecture/data-guide).

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[customer-site]: https://home.komatsu/en/
[customer-story]: https://customers.microsoft.com/story/komatsu-manufacturing-azure-iot-hub-japan
[architecture]: ./media/architecture-big-data-with-iot.png
