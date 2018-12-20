---
title: Padrões de mensagens
description: A natureza distribuída dos aplicativos de nuvem exige uma infraestrutura de mensagens que conecta os componentes e serviços, idealmente de uma maneira flexível para maximizar a escalabilidade. O sistema de mensagens assíncronas é amplamente usado e fornece muitos benefícios, mas também traz desafios, como a ordenação de mensagens, o gerenciamento de mensagens suspeitas, a idempotência e muito mais.
keywords: padrão de design
author: dragon119
ms.date: 12/07/2018
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 754e9a7dcad20dc1c4471af00f3f142f18022d62
ms.sourcegitcommit: a0a9981e7586bed8d876a54e055dea1e392118f8
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/11/2018
ms.locfileid: "53233873"
---
# <a name="messaging-patterns"></a>Padrões de mensagens

[!INCLUDE [header](../../_includes/header.md)]

A natureza distribuída dos aplicativos de nuvem exige uma infraestrutura de mensagens que conecta os componentes e serviços, idealmente de uma maneira flexível para maximizar a escalabilidade. O sistema de mensagens assíncronas é amplamente usado e fornece muitos benefícios, mas também traz desafios, como a ordenação de mensagens, o gerenciamento de mensagens suspeitas, a idempotência e muito mais.

| Padrão | Resumo |
| ------- | ------- |
| [Consumidores Concorrentes](../competing-consumers.md) | Habilite vários consumidores simultâneos para processar as mensagens recebidas no mesmo canal de mensagens. |
| [Pipes e Filtros](../pipes-and-filters.md) | Dividir uma tarefa que executa processamento complexo em uma série de elementos separados que podem ser reutilizados. |
| [Fila de Prioridade](../priority-queue.md) | Priorize as solicitações enviadas a serviços para que as solicitações com uma prioridade mais alta sejam recebidas e processadas mais rapidamente do que aquelas com uma prioridade mais baixa. |
| [Publisher-Subscriber](../publisher-subscriber.md) | Permite a um aplicativo anunciar eventos para vários consumidores de seu interesse assincronamente, sem acoplar os remetentes aos destinatários. |
| [Nivelamento de Carga Baseado em Fila](../queue-based-load-leveling.md) | Use uma fila que funcione como um buffer entre uma tarefa e um serviço que ela invoca para simplificar cargas pesadas intermitentes. |
| [Supervisor de Agente do Agendador](../scheduler-agent-supervisor.md) | Coordene um conjunto de ações em um conjunto distribuído de serviços e outros recursos remotos. |
