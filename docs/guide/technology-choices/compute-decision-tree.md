---
title: Árvore de decisão para serviços de computação do Azure
titleSuffix: Azure Application Architecture Guide
description: Um fluxograma para selecionar um serviço de computação.
author: MikeWasson
ms.date: 11/03/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: e3df1cbdd049e8c40597a85eca29899d8d0d1bc3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54484639"
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

Para obter mais informações sobre as opções de hospedagem de contêineres no Azure, confira [Contêineres do Azure](https://azure.microsoft.com/overview/containers/).

## <a name="flowchart"></a>Fluxograma

![Árvore de decisão para serviços de computação do Azure](../images/compute-decision-tree.svg)

## <a name="definitions"></a>Definições

- **Lift and shift** é uma estratégia de migração de uma carga de trabalho para a nuvem sem recriar o aplicativo ou fazer alterações de código. Também chamado de *nova hospedagem*. Para obter mais informações, consulte a [Central de migrações do Azure](https://azure.microsoft.com/migration/).

- **Nuvem otimizada** é uma estratégia de migração para a nuvem por meio da refatoração de um aplicativo para aproveitar os recursos e funcionalidades de nuvem nativos.

## <a name="next-steps"></a>Próximas etapas

Para conhecer critérios adicionais a serem considerados, consulte [Critérios para escolher um serviço de computação do Azure](./compute-comparison.md).
