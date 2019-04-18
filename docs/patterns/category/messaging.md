---
title: Padrões de mensagens
titleSuffix: Cloud Design Patterns
description: A natureza distribuída dos aplicativos de nuvem exige uma infraestrutura de mensagens que conecta os componentes e serviços, idealmente de uma maneira flexível para maximizar a escalabilidade. O sistema de mensagens assíncronas é amplamente usado e fornece muitos benefícios, mas também traz desafios, como a ordenação de mensagens, o gerenciamento de mensagens suspeitas, a idempotência e muito mais.
keywords: padrão de design
author: dragon119
ms.date: 03/01/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f838288e10acf9af5776c93972028b6b878bd7b0
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640661"
---
# <a name="messaging-patterns"></a><span data-ttu-id="e3392-105">Padrões de mensagens</span><span class="sxs-lookup"><span data-stu-id="e3392-105">Messaging patterns</span></span>

<span data-ttu-id="e3392-106">A natureza distribuída dos aplicativos de nuvem exige uma infraestrutura de mensagens que conecta os componentes e serviços, idealmente de uma maneira flexível para maximizar a escalabilidade.</span><span class="sxs-lookup"><span data-stu-id="e3392-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="e3392-107">O sistema de mensagens assíncronas é amplamente usado e fornece muitos benefícios, mas também traz desafios, como a ordenação de mensagens, o gerenciamento de mensagens suspeitas, a idempotência e muito mais.</span><span class="sxs-lookup"><span data-stu-id="e3392-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="e3392-108">Padrão</span><span class="sxs-lookup"><span data-stu-id="e3392-108">Pattern</span></span> | <span data-ttu-id="e3392-109">Resumo</span><span class="sxs-lookup"><span data-stu-id="e3392-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="e3392-110">Verificação de declaração</span><span class="sxs-lookup"><span data-stu-id="e3392-110">Claim Check</span></span>](../claim-check.md) | <span data-ttu-id="e3392-111">Divida uma mensagem grande em uma verificação de declaração e uma carga para evitar sobrecarregar um barramento de mensagem.</span><span class="sxs-lookup"><span data-stu-id="e3392-111">Split a large message into a claim check and a payload to avoid overwhelming a message bus.</span></span> |
| [<span data-ttu-id="e3392-112">Consumidores Concorrentes</span><span class="sxs-lookup"><span data-stu-id="e3392-112">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="e3392-113">Habilite vários consumidores simultâneos para processar as mensagens recebidas no mesmo canal de mensagens.</span><span class="sxs-lookup"><span data-stu-id="e3392-113">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="e3392-114">Pipes e Filtros</span><span class="sxs-lookup"><span data-stu-id="e3392-114">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="e3392-115">Dividir uma tarefa que executa processamento complexo em uma série de elementos separados que podem ser reutilizados.</span><span class="sxs-lookup"><span data-stu-id="e3392-115">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="e3392-116">Fila de Prioridade</span><span class="sxs-lookup"><span data-stu-id="e3392-116">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="e3392-117">Priorize as solicitações enviadas a serviços para que as solicitações com uma prioridade mais alta sejam recebidas e processadas mais rapidamente do que aquelas com uma prioridade mais baixa.</span><span class="sxs-lookup"><span data-stu-id="e3392-117">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="e3392-118">Publisher-Subscriber</span><span class="sxs-lookup"><span data-stu-id="e3392-118">Publisher-Subscriber</span></span>](../publisher-subscriber.md) | <span data-ttu-id="e3392-119">Permita que um aplicativo anunciar eventos para vários consumidores interessados de forma assíncrona, sem acoplamento os remetentes aos destinatários.</span><span class="sxs-lookup"><span data-stu-id="e3392-119">Enable an application to announce events to multiple interested consumers asynchronously, without coupling the senders to the receivers.</span></span> |
| [<span data-ttu-id="e3392-120">Nivelamento de Carga Baseado em Fila</span><span class="sxs-lookup"><span data-stu-id="e3392-120">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="e3392-121">Use uma fila que funcione como um buffer entre uma tarefa e um serviço que ela invoca para simplificar cargas pesadas intermitentes.</span><span class="sxs-lookup"><span data-stu-id="e3392-121">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="e3392-122">Supervisor de Agente do Agendador</span><span class="sxs-lookup"><span data-stu-id="e3392-122">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="e3392-123">Coordene um conjunto de ações em um conjunto distribuído de serviços e outros recursos remotos.</span><span class="sxs-lookup"><span data-stu-id="e3392-123">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |
