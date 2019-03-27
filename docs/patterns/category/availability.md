---
title: Padrões de disponibilidade
titleSuffix: Cloud Design Patterns
description: A disponibilidade define a proporção de tempo em que o sistema está funcionando. Ele será afetado por erros de sistema, problemas de infraestrutura, ataques mal-intencionados e carga do sistema. Ele geralmente é medido como uma porcentagem do tempo de atividade. Os aplicativos de nuvem normalmente fornecem aos usuários um contrato de nível de serviço (SLA), o que significa que os aplicativos devem ser criados e implementados para maximizar a disponibilidade.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 897bc3c87beb23770220ef0fc3c5c4394d192f2a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243037"
---
# <a name="availability-patterns"></a>Padrões de disponibilidade

[!INCLUDE [header](../../_includes/header.md)]

A disponibilidade define a proporção de tempo em que o sistema está funcionando. Ele será afetado por erros de sistema, problemas de infraestrutura, ataques mal-intencionados e carga do sistema. Ele geralmente é medido como uma porcentagem do tempo de atividade. Os aplicativos de nuvem normalmente fornecem aos usuários um contrato de nível de serviço (SLA), o que significa que os aplicativos devem ser criados e implementados para maximizar a disponibilidade.

|                            Padrão                             |                                                           Resumo                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [Monitoramento do Ponto de Extremidade de Integridade](../health-endpoint-monitoring.md) | Implemente verificações funcionais dentro de um aplicativo cujas ferramentas externas podem acessar por meio de pontos de extremidade expostos em intervalos regulares. |
|  [Nivelamento de Carga Baseado em Fila](../queue-based-load-leveling.md)  | Use uma fila que funcione como um buffer entre uma tarefa e um serviço que ela invoca para simplificar cargas pesadas intermitentes.  |
|                 [Limitação](../throttling.md)                 |   Controle o consumo de recursos usados por uma instância de um aplicativo, um locatário individual ou todo o serviço.    |
