---
title: Monitorando o Azure Databricks com o Azure Monitor
description: Uma biblioteca do Scala para habilitar o monitoramento do Azure Databricks no Azure Log Analytics
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 4544d3abc3264ec459a80ac1a61a912e6d30d6b2
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887685"
---
# <a name="monitoring-azure-databricks"></a>Monitorando o Azure Databricks

[O Azure Databricks](/azure/azure-databricks/) é um serviço de análise rápido e poderoso com base no [Apache Spark](https://spark.apache.org/), que torna mais fácil desenvolver e implantar rapidamente soluções de IA (inteligência artificial) e análise de Big Data. Muitos usuários aproveitam a simplicidade de notebooks em suas soluções do Azure Databricks. Para usuários que requerem opções de computação mais robustas, o Azure Databricks dá suporte à execução distribuída de código de aplicativo personalizado.

O monitoramento é uma parte crítica de qualquer solução de nível de produção. O Azure Databricks oferece uma funcionalidade avançada para monitorar métricas de aplicativo personalizadas, transmitir eventos de consulta e mensagens de log do aplicativo. O Azure Databricks pode enviar esses dados de monitoramento para diferentes serviços de registro em log.

Os artigos a seguir mostram como enviar dados de monitoramento do Azure Databricks para o [Azure Monitor](/azure/azure-monitor/overview), a plataforma de dados de monitoramento do Azure. Você deve seguir estes artigos na ordem.

1. [Configurar o Azure Databricks para enviar métricas para o Azure Monitor](./configure-cluster.md)
1. [Enviar logs do aplicativo do Azure Databricks para o Azure Monitor](./application-logs.md)
1. [Usar painéis para visualizar métricas do Azure Databricks](./dashboards.md)

A biblioteca de código que acompanha esses artigos estende a funcionalidade de monitoramento principal do Azure Databricks para enviar métricas, eventos e informações de log do Spark para o Azure Monitor.

O público-alvo para esses artigos e a biblioteca de códigos que os acompanha é composto pelos desenvolvedores de soluções do Apache Spark e do Azure Databricks. O código precisa ser incorporado a arquivos JAR (Java Archive) e, em seguida, implantado em um cluster do Azure Databricks. O código é uma combinação de [Scala](https://www.scala-lang.org/) e Java, com um conjunto correspondente de arquivos POM (modelo de objeto de projeto) do [Maven](https://maven.apache.org) para criar os arquivos JAR de saída do projeto. A compreensão do Java, Scala e Maven é recomendada como pré-requisito.

## <a name="next-steps"></a>Próximas etapas

Comece criando a biblioteca de código e implantando-a em seu cluster do Azure Databricks.

> [!div class="nextstepaction"]
> [Configurar o Azure Databricks para enviar métricas para o Azure Monitor](./configure-cluster.md)