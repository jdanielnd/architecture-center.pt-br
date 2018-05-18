---
title: Árvore de decisão para serviços de computação do Azure
description: Um fluxograma para selecionar um serviço de computação
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: 3dcfbd156d4fced863a56bcc8bb74483aa665f9f
ms.sourcegitcommit: 7ced70ebc11aa0df0dc0104092d3cc6ad5c28bd6
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/11/2018
---
# <a name="decision-tree-for-azure-compute-services"></a>Árvore de decisão para serviços de computação do Azure

O Azure oferece várias maneiras de hospedar o código do seu aplicativo. O termo *computação* refere-se ao modelo de hospedagem para os recursos de computação em que seu aplicativo é executado. O fluxograma a seguir o ajudará a escolher um serviço de computação para o aplicativo. O fluxograma orienta você por um conjunto de critérios de decisão essenciais que geram uma recomendação. 

**Trate esse fluxograma como um ponto de partida.** Cada aplicativo tem requisitos exclusivos e, portanto use a recomendação como um ponto de partida. Em seguida, execute uma avaliação mais detalhada, observando aspectos como:
 
- Conjunto de recursos
- [Limites de serviço](/azure/azure-subscription-service-limits)
- [Custo](https://azure.microsoft.com/pricing/)
- [SLA](https://azure.microsoft.com/support/legal/sla/)
- [Disponibilidade regional](https://azure.microsoft.com/global-infrastructure/services/)
- Habilidades da equipe e o ecossistema do desenvolvedor
- [Tabelas de comparação de computação](./compute-comparison.md)

Se o aplicativo tem várias cargas de trabalho, avalie cada carga de trabalho separadamente. Uma solução completa pode incorporar dois ou mais serviços de computação.

![](../images/compute-decision-tree.svg)

