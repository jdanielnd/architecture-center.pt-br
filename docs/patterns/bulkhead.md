---
title: Padrão de bulkhead
description: Isole os elementos de um aplicativo em pools para que, se um falhar, os outros continuarão a funcionar
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 9917870e1dcbed87aaa41e051f1622ad4950456a
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/16/2018
ms.locfileid: "31012698"
---
# <a name="bulkhead-pattern"></a><span data-ttu-id="3ab0a-103">Padrão de bulkhead</span><span class="sxs-lookup"><span data-stu-id="3ab0a-103">Bulkhead pattern</span></span>

<span data-ttu-id="3ab0a-104">Isole os elementos de um aplicativo em pools para que, se um falhar, os outros continuarão a funcionar.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-104">Isolate elements of an application into pools so that if one fails, the others will continue to function.</span></span>

<span data-ttu-id="3ab0a-105">Esse padrão é denominado *Bulkhead* (tabique) porque é parecido com as partições seccionadas do casco de um navio.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-105">This pattern is named *Bulkhead* because it resembles the sectioned partitions of a ship's hull.</span></span> <span data-ttu-id="3ab0a-106">Se o casco de um navio for comprometido, somente a seção danificada se encherá de água, impedindo que o navio afunde.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-106">If the hull of a ship is compromised, only the damaged section fills with water, which prevents the ship from sinking.</span></span> 

## <a name="context-and-problem"></a><span data-ttu-id="3ab0a-107">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="3ab0a-107">Context and problem</span></span>

<span data-ttu-id="3ab0a-108">Um aplicativo baseado em nuvem pode incluir vários serviços, cada um deles tendo um ou mais consumidores.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-108">A cloud-based application may include multiple services, with each service having one or more consumers.</span></span> <span data-ttu-id="3ab0a-109">Carga excessiva ou falha em um serviço afetará todos os consumidores do serviço.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-109">Excessive load or failure in a service will impact all consumers of the service.</span></span>

<span data-ttu-id="3ab0a-110">Além disso, um consumidor pode enviar solicitações a vários serviços ao mesmo tempo usando recursos para cada solicitação.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-110">Moreover, a consumer may send requests to multiple services simultaneously, using resources for each request.</span></span> <span data-ttu-id="3ab0a-111">Quando o consumidor envia uma solicitação para um serviço que está configurado incorretamente ou não está respondendo, os recursos usados pela solicitação do cliente não podem ser liberados de maneira oportuna.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-111">When the consumer sends a request to a service that is misconfigured or not responding, the resources used by the client's request may not be freed in a timely manner.</span></span> <span data-ttu-id="3ab0a-112">Conforme as solicitações ao serviço continuam, esses recursos podem ser esgotados.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-112">As requests to the service continue, those resources may be exhausted.</span></span> <span data-ttu-id="3ab0a-113">Por exemplo, o pool de conexão do cliente pode ser esgotado.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-113">For example, the client's connection pool may be exhausted.</span></span> <span data-ttu-id="3ab0a-114">Neste ponto, as solicitações do consumidor a outros serviços são afetadas.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-114">At that point, requests by the consumer to other services are impacted.</span></span> <span data-ttu-id="3ab0a-115">Por fim, o consumidor não poderá mais enviar solicitações a outros serviços, não apenas ao serviço que originalmente não estava respondendo.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-115">Eventually the consumer can no longer send requests to other services, not just the original unresponsive service.</span></span>

<span data-ttu-id="3ab0a-116">O mesmo problema de esgotamento de recursos afeta serviços com vários consumidores.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-116">The same issue of resource exhaustion affects services with multiple consumers.</span></span> <span data-ttu-id="3ab0a-117">Um grande número de solicitações originadas de um cliente pode esgotar os recursos disponíveis no serviço.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-117">A large number of requests originating from one client may exhaust available resources in the service.</span></span> <span data-ttu-id="3ab0a-118">Outros consumidores não são conseguem mais consumir o serviço, causando um efeito de falha em cascata.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-118">Other consumers are no longer able to consume the service, causing a cascading failure effect.</span></span>

## <a name="solution"></a><span data-ttu-id="3ab0a-119">Solução</span><span class="sxs-lookup"><span data-stu-id="3ab0a-119">Solution</span></span>

<span data-ttu-id="3ab0a-120">Particione instâncias de serviço em grupos diferentes conforme a carga do consumidor e nos requisitos de disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-120">Partition service instances into different groups, based on consumer load and availability requirements.</span></span> <span data-ttu-id="3ab0a-121">Esse design ajuda a isolar falhas e permite que você mantenha a funcionalidade do serviço para alguns clientes mesmo durante uma falha.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-121">This design helps to isolate failures, and allows you to sustain service functionality for some consumers, even during a failure.</span></span>

<span data-ttu-id="3ab0a-122">Um consumidor também pode dividir os recursos para garantir que os recursos usados para chamar um serviço não afetem os recursos usados para chamar outro.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-122">A consumer can also partition resources, to ensure that resources used to call one service don't affect the resources used to call another service.</span></span> <span data-ttu-id="3ab0a-123">Por exemplo, um pool de conexão para cada serviço poderá ser atribuído a um consumidor que chame vários serviços.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-123">For example, a consumer that calls multiple services may be assigned a connection pool for each service.</span></span> <span data-ttu-id="3ab0a-124">Se um serviço começar a falhar, isso afetará somente o pool de conexão atribuído a esse serviço, permitindo que o consumidor continue usando outros serviços.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-124">If a service begins to fail, it only affects the connection pool assigned for that service, allowing the consumer to continue using the other services.</span></span>

<span data-ttu-id="3ab0a-125">Os benefícios desse padrão incluem:</span><span class="sxs-lookup"><span data-stu-id="3ab0a-125">The benefits of this pattern include:</span></span>

- <span data-ttu-id="3ab0a-126">Isolar os consumidores e os serviços para que não haja falhas em cascata.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-126">Isolates consumers and services from cascading failures.</span></span> <span data-ttu-id="3ab0a-127">Um problema que afeta um consumidor ou um serviço pode ser isolado em seu próprio bulkhead, impedindo que a solução inteira falhe.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-127">An issue affecting a consumer or service can be isolated within its own bulkhead, preventing the entire solution from failing.</span></span>
- <span data-ttu-id="3ab0a-128">Permitir que você preserve algumas funcionalidades em caso de falha de serviço.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-128">Allows you to preserve some functionality in the event of a service failure.</span></span> <span data-ttu-id="3ab0a-129">Outros serviços e recursos do aplicativo continuarão a funcionar.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-129">Other services and features of the application will continue to work.</span></span>
- <span data-ttu-id="3ab0a-130">Permitir que você implante os serviços que oferecem uma qualidade de serviço diferente para aplicativos de consumo.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-130">Allows you to deploy services that offer a different quality of service for consuming applications.</span></span> <span data-ttu-id="3ab0a-131">Um pool de consumidor de alta prioridade pode ser configurado para usar serviços de alta prioridade.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-131">A high-priority consumer pool can be configured to use high-priority services.</span></span> 

<span data-ttu-id="3ab0a-132">O diagrama a seguir mostra bulkheads estruturados em torno de pools de conexão que chamam serviços individuais.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-132">The following diagram shows bulkheads structured around connection pools that call individual services.</span></span> <span data-ttu-id="3ab0a-133">Se o Serviço A falhar ou causar algum outro problema, o pool de conexão será isolado, assim, somente as cargas de trabalho que usem o pool de threads atribuído ao Serviço A serão afetadas.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-133">If Service A fails or causes some other issue, the connection pool is isolated, so only workloads using the thread pool assigned to Service A are affected.</span></span> <span data-ttu-id="3ab0a-134">Cargas de trabalho que usam os Serviços B e C não são afetadas e podem continuar trabalhando sem interrupções.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-134">Workloads that use Service B and C are not affected and can continue working without interruption.</span></span>

![](./_images/bulkhead-1.png) 

<span data-ttu-id="3ab0a-135">O diagrama seguinte mostra vários clientes chamando um único serviço.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-135">The next diagram shows multiple clients calling a single service.</span></span> <span data-ttu-id="3ab0a-136">Cada cliente é atribuído a uma instância de serviço separada.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-136">Each client is assigned a separate service instance.</span></span> <span data-ttu-id="3ab0a-137">O Cliente 1 fez solicitações demais e sobrecarregou sua instância.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-137">Client 1 has made too many requests and overwhelmed its instance.</span></span> <span data-ttu-id="3ab0a-138">Uma vez que cada instância de serviço é isolada de outras, os outros clientes podem continuar fazendo chamadas.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-138">Because each service instance is isolated from the others, the other clients can continue making calls.</span></span>

![](./_images/bulkhead-2.png)
     
## <a name="issues-and-considerations"></a><span data-ttu-id="3ab0a-139">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="3ab0a-139">Issues and considerations</span></span>

- <span data-ttu-id="3ab0a-140">Defina partições em torno de requisitos comerciais e técnicos do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-140">Define partitions around the business and technical requirements of the application.</span></span>
- <span data-ttu-id="3ab0a-141">Ao particionar serviços ou consumidores em bulkheads, considere o nível de isolamento oferecido pela tecnologia, bem como a sobrecarga em termos de custo, desempenho e capacidade de gerenciamento.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-141">When partitioning services or consumers into bulkheads, consider the level of isolation offered by the technology as well as the overhead in terms of cost, performance and manageability.</span></span>
- <span data-ttu-id="3ab0a-142">Considere combinar bulkheads com nova tentativa, interruptor de circuito e padrões de limitação para proporcionar um tratamento de falhas mais sofisticado.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-142">Consider combining bulkheads with retry, circuit breaker, and throttling patterns to provide more sophisticated fault handling.</span></span>
- <span data-ttu-id="3ab0a-143">Ao particionar os consumidores em bulkheads, considere usar processos, pools de threads e semáforos.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-143">When partitioning consumers into bulkheads, consider using processes, thread pools, and semaphores.</span></span> <span data-ttu-id="3ab0a-144">Projetos como [Netflix Hystrix][hystrix] e [Polly][polly] oferecem uma estrutura para criar bulkheads do consumidor.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-144">Projects like [Netflix Hystrix][hystrix] and [Polly][polly] offer a framework for creating consumer bulkheads.</span></span>
- <span data-ttu-id="3ab0a-145">Ao particionar serviços em bulkheads, considere implantá-los em máquinas virtuais, contêineres ou processos separados.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-145">When partitioning services into bulkheads, consider deploying them into separate virtual machines, containers, or processes.</span></span> <span data-ttu-id="3ab0a-146">Contêineres oferecem um bom equilíbrio de isolamento de recursos com uma sobrecarga razoavelmente baixa.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-146">Containers offer a good balance of resource isolation with fairly low overhead.</span></span>
- <span data-ttu-id="3ab0a-147">Serviços que se comunicam usando mensagens assíncronas podem ser isolados por meio de diferentes conjuntos de filas.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-147">Services that communicate using asynchronous messages can be isolated through different sets of queues.</span></span> <span data-ttu-id="3ab0a-148">Cada fila pode ter um conjunto dedicado de instâncias processando mensagens na fila ou um único grupo de instâncias usando um algoritmo para processamento de remoção da fila e expedição.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-148">Each queue can have a dedicated set of instances processing messages on the queue, or a single group of instances using an algorithm to dequeue and dispatch processing.</span></span>
- <span data-ttu-id="3ab0a-149">Determine o nível de granularidade dos bulkheads.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-149">Determine the level of granularity for the bulkheads.</span></span> <span data-ttu-id="3ab0a-150">Por exemplo, se você quiser distribuir locatários entre partições, poderá colocar cada locatário em uma partição separada ou colocar vários locatários em uma partição.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-150">For example, if you want to distribute tenants across partitions, you could place each tenant into a separate partition, or put several tenants into one partition.</span></span>
- <span data-ttu-id="3ab0a-151">Monitore o desempenho e o SLA de cada partição.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-151">Monitor each partition’s performance and SLA.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="3ab0a-152">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="3ab0a-152">When to use this pattern</span></span>

<span data-ttu-id="3ab0a-153">Use esse padrão para:</span><span class="sxs-lookup"><span data-stu-id="3ab0a-153">Use this pattern to:</span></span>

- <span data-ttu-id="3ab0a-154">Isolar os recursos usados para consumir um conjunto de serviços de back-end, especialmente se o aplicativo puder fornecer algum nível de funcionalidade mesmo quando um dos serviços não estiver respondendo.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-154">Isolate resources used to consume a set of backend services, especially if the application can provide some level of functionality even when one of the services is not responding.</span></span>
- <span data-ttu-id="3ab0a-155">Isole os consumidores críticos dos consumidores padrão.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-155">Isolate critical consumers from standard consumers.</span></span>
- <span data-ttu-id="3ab0a-156">Proteja o aplicativo contra falhas em cascata.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-156">Protect the application from cascading failures.</span></span>

<span data-ttu-id="3ab0a-157">Esse padrão pode não ser adequado quando:</span><span class="sxs-lookup"><span data-stu-id="3ab0a-157">This pattern may not be suitable when:</span></span>

- <span data-ttu-id="3ab0a-158">Uso menos eficiente de recursos pode não ser aceitável no projeto.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-158">Less efficient use of resources may not be acceptable in the project.</span></span>
- <span data-ttu-id="3ab0a-159">A complexidade adicional não é necessária</span><span class="sxs-lookup"><span data-stu-id="3ab0a-159">The added complexity is not necessary</span></span>

## <a name="example"></a><span data-ttu-id="3ab0a-160">Exemplo</span><span class="sxs-lookup"><span data-stu-id="3ab0a-160">Example</span></span>

<span data-ttu-id="3ab0a-161">O seguinte arquivo de configuração Kubernetes cria um contêiner isolado para executar um único serviço, com seus próprios recursos e limites de CPU e memória.</span><span class="sxs-lookup"><span data-stu-id="3ab0a-161">The following Kubernetes configuration file creates an isolated container to run a single service, with its own CPU and memory resources and limits.</span></span>

```yml
apiVersion: v1
kind: Pod
metadata:
  name: drone-management
spec:
  containers:
  - name: drone-management-container
    image: drone-service
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "1"
```

## <a name="related-guidance"></a><span data-ttu-id="3ab0a-162">Diretrizes relacionadas</span><span class="sxs-lookup"><span data-stu-id="3ab0a-162">Related guidance</span></span>

- [<span data-ttu-id="3ab0a-163">Padrão de interruptor de circuito</span><span class="sxs-lookup"><span data-stu-id="3ab0a-163">Circuit Breaker pattern</span></span>](./circuit-breaker.md)
- [<span data-ttu-id="3ab0a-164">Projeto de aplicativos resilientes do Azure</span><span class="sxs-lookup"><span data-stu-id="3ab0a-164">Designing resilient applications for Azure</span></span>](../resiliency/index.md)
- [<span data-ttu-id="3ab0a-165">Padrão de repetição</span><span class="sxs-lookup"><span data-stu-id="3ab0a-165">Retry pattern</span></span>](./retry.md)
- [<span data-ttu-id="3ab0a-166">Padrão de limitação</span><span class="sxs-lookup"><span data-stu-id="3ab0a-166">Throttling pattern</span></span>](./throttling.md)


<!-- links -->

[hystrix]: https://github.com/Netflix/Hystrix
[polly]: https://github.com/App-vNext/Polly