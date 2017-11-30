---
title: "Padrões de mensagens"
description: "A natureza distribuída dos aplicativos de nuvem exige uma infraestrutura de mensagens que conecta os componentes e serviços, idealmente de uma maneira flexível para maximizar a escalabilidade. O sistema de mensagens assíncronas é amplamente usado e fornece muitos benefícios, mas também traz desafios, como a ordenação de mensagens, o gerenciamento de mensagens suspeitas, a idempotência e muito mais."
keywords: "padrão de design"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 6151f7f76fc7b3a953988122db75bdc25b49811f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="messaging-patterns"></a><span data-ttu-id="7d573-105">Padrões de mensagens</span><span class="sxs-lookup"><span data-stu-id="7d573-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="7d573-106">A natureza distribuída dos aplicativos de nuvem exige uma infraestrutura de mensagens que conecta os componentes e serviços, idealmente de uma maneira flexível para maximizar a escalabilidade.</span><span class="sxs-lookup"><span data-stu-id="7d573-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="7d573-107">O sistema de mensagens assíncronas é amplamente usado e fornece muitos benefícios, mas também traz desafios, como a ordenação de mensagens, o gerenciamento de mensagens suspeitas, a idempotência e muito mais.</span><span class="sxs-lookup"><span data-stu-id="7d573-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="7d573-108">Padrão</span><span class="sxs-lookup"><span data-stu-id="7d573-108">Pattern</span></span> | <span data-ttu-id="7d573-109">Resumo</span><span class="sxs-lookup"><span data-stu-id="7d573-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="7d573-110">Consumidores Concorrentes</span><span class="sxs-lookup"><span data-stu-id="7d573-110">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="7d573-111">Habilite vários consumidores simultâneos para processar as mensagens recebidas no mesmo canal de mensagens.</span><span class="sxs-lookup"><span data-stu-id="7d573-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="7d573-112">Pipes e Filtros</span><span class="sxs-lookup"><span data-stu-id="7d573-112">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="7d573-113">Divida uma tarefa que executa processamento complexo em uma série de elementos separados que podem ser reutilizados.</span><span class="sxs-lookup"><span data-stu-id="7d573-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="7d573-114">Fila de Prioridade</span><span class="sxs-lookup"><span data-stu-id="7d573-114">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="7d573-115">Priorize as solicitações enviadas a serviços para que as solicitações com uma prioridade mais alta sejam recebidas e processadas mais rapidamente do que aquelas com uma prioridade mais baixa.</span><span class="sxs-lookup"><span data-stu-id="7d573-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="7d573-116">Nivelamento de Carga Baseado em Fila</span><span class="sxs-lookup"><span data-stu-id="7d573-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="7d573-117">Use uma fila que funcione como um buffer entre uma tarefa e um serviço que ela invoca para simplificar cargas pesadas intermitentes.</span><span class="sxs-lookup"><span data-stu-id="7d573-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="7d573-118">Supervisor de Agente do Agendador</span><span class="sxs-lookup"><span data-stu-id="7d573-118">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="7d573-119">Coordene um conjunto de ações em um conjunto distribuído de serviços e outros recursos remotos.</span><span class="sxs-lookup"><span data-stu-id="7d573-119">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |