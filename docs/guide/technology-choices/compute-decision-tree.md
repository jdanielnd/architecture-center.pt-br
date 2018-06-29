---
title: Árvore de decisão para serviços de computação do Azure
description: Um fluxograma para selecionar um serviço de computação
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: 689ec3f265e563273a75ad98268d03624a7b4536
ms.sourcegitcommit: ce2fa8ac2d310f7078317cade12f1b89db1ffe06
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/23/2018
ms.locfileid: "36338176"
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

Para obter mais informações sobre as opções de hospedagem de contêineres no Azure, consulte https://azure.microsoft.com/overview/containers/.

## <a name="flowchart"></a>Fluxograma

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a>Definições

- **Greenfield** descreve um projeto de software que é completamente novo e criado do zero. Ele não inclui códigos herdados. 

- **Brownfield** descreve um projeto de software que se baseia em um aplicativo existente. Ele pode herdar um código herdado ou estruturas.

- **Lift and shift** é uma estratégia de migração de uma carga de trabalho para a nuvem sem recriar o aplicativo ou fazer alterações de código. Também chamado de *nova hospedagem*. Para obter mais informações, consulte a [Central de migrações do Azure](https://azure.microsoft.com/migration/).

- **Nuvem otimizada** é uma estratégia de migração para a nuvem por meio da refatoração de um aplicativo para aproveitar os recursos e funcionalidades de nuvem nativos.

## <a name="next-steps"></a>Próximas etapas

Para conhecer critérios adicionais a serem considerados, consulte [Critérios para escolher um serviço de computação do Azure](./compute-comparison.md).
