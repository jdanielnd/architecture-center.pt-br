---
title: Padrões de disponibilidade
description: A disponibilidade define a proporção de tempo em que o sistema está funcionando. Ele será afetado por erros de sistema, problemas de infraestrutura, ataques mal-intencionados e carga do sistema. Ele geralmente é medido como uma porcentagem do tempo de atividade. Os aplicativos de nuvem normalmente fornecem aos usuários um contrato de nível de serviço (SLA), o que significa que os aplicativos devem ser criados e implementados para maximizar a disponibilidade.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: eac1d192fe83f8501d11dce02d9bec16e939ba55
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847270"
---
# <a name="availability-patterns"></a>Padrões de disponibilidade

[!INCLUDE [header](../../_includes/header.md)]

A disponibilidade define a proporção de tempo em que o sistema está funcionando. Ele será afetado por erros de sistema, problemas de infraestrutura, ataques mal-intencionados e carga do sistema. Ele geralmente é medido como uma porcentagem do tempo de atividade. Os aplicativos de nuvem normalmente fornecem aos usuários um contrato de nível de serviço (SLA), o que significa que os aplicativos devem ser criados e implementados para maximizar a disponibilidade.


|                            Padrão                             |                                                           Resumo                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [Monitoramento do Ponto de Extremidade de Integridade](../health-endpoint-monitoring.md) | Implemente verificações funcionais dentro de um aplicativo cujas ferramentas externas podem acessar por meio de pontos de extremidade expostos em intervalos regulares. |
|  [Nivelamento de Carga Baseado em Fila](../queue-based-load-leveling.md)  | Use uma fila que funcione como um buffer entre uma tarefa e um serviço que ela invoca para simplificar cargas pesadas intermitentes.  |
|                 [Limitação](../throttling.md)                 |   Controle o consumo de recursos usados por uma instância de um aplicativo, um locatário individual ou todo o serviço.    |

