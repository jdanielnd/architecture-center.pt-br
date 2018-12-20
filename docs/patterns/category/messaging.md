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
# <a name="messaging-patterns"></a><span data-ttu-id="b5200-105">Padrões de mensagens</span><span class="sxs-lookup"><span data-stu-id="b5200-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="b5200-106">A natureza distribuída dos aplicativos de nuvem exige uma infraestrutura de mensagens que conecta os componentes e serviços, idealmente de uma maneira flexível para maximizar a escalabilidade.</span><span class="sxs-lookup"><span data-stu-id="b5200-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="b5200-107">O sistema de mensagens assíncronas é amplamente usado e fornece muitos benefícios, mas também traz desafios, como a ordenação de mensagens, o gerenciamento de mensagens suspeitas, a idempotência e muito mais.</span><span class="sxs-lookup"><span data-stu-id="b5200-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="b5200-108">Padrão</span><span class="sxs-lookup"><span data-stu-id="b5200-108">Pattern</span></span> | <span data-ttu-id="b5200-109">Resumo</span><span class="sxs-lookup"><span data-stu-id="b5200-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="b5200-110">Consumidores Concorrentes</span><span class="sxs-lookup"><span data-stu-id="b5200-110">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="b5200-111">Habilite vários consumidores simultâneos para processar as mensagens recebidas no mesmo canal de mensagens.</span><span class="sxs-lookup"><span data-stu-id="b5200-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="b5200-112">Pipes e Filtros</span><span class="sxs-lookup"><span data-stu-id="b5200-112">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="b5200-113">Dividir uma tarefa que executa processamento complexo em uma série de elementos separados que podem ser reutilizados.</span><span class="sxs-lookup"><span data-stu-id="b5200-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="b5200-114">Fila de Prioridade</span><span class="sxs-lookup"><span data-stu-id="b5200-114">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="b5200-115">Priorize as solicitações enviadas a serviços para que as solicitações com uma prioridade mais alta sejam recebidas e processadas mais rapidamente do que aquelas com uma prioridade mais baixa.</span><span class="sxs-lookup"><span data-stu-id="b5200-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="b5200-116">Publisher-Subscriber</span><span class="sxs-lookup"><span data-stu-id="b5200-116">Publisher-Subscriber</span></span>](../publisher-subscriber.md) | <span data-ttu-id="b5200-117">Permite a um aplicativo anunciar eventos para vários consumidores de seu interesse assincronamente, sem acoplar os remetentes aos destinatários.</span><span class="sxs-lookup"><span data-stu-id="b5200-117">Enable an application to announce events to multiple interested consumers aynchronously, without coupling the senders to the receivers.</span></span> |
| [<span data-ttu-id="b5200-118">Nivelamento de Carga Baseado em Fila</span><span class="sxs-lookup"><span data-stu-id="b5200-118">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="b5200-119">Use uma fila que funcione como um buffer entre uma tarefa e um serviço que ela invoca para simplificar cargas pesadas intermitentes.</span><span class="sxs-lookup"><span data-stu-id="b5200-119">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="b5200-120">Supervisor de Agente do Agendador</span><span class="sxs-lookup"><span data-stu-id="b5200-120">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="b5200-121">Coordene um conjunto de ações em um conjunto distribuído de serviços e outros recursos remotos.</span><span class="sxs-lookup"><span data-stu-id="b5200-121">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |
