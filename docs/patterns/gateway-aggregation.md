---
title: Padrão de agregação de gateway
description: Use um gateway para agregar várias solicitações individuais em uma única solicitação.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: f59c8b8b02c6db28024d13621b782997e63a4e9e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541266"
---
# <a name="gateway-aggregation-pattern"></a><span data-ttu-id="a2060-103">Padrão de agregação de gateway</span><span class="sxs-lookup"><span data-stu-id="a2060-103">Gateway Aggregation pattern</span></span>

<span data-ttu-id="a2060-104">Use um gateway para agregar várias solicitações individuais em uma única solicitação.</span><span class="sxs-lookup"><span data-stu-id="a2060-104">Use a gateway to aggregate multiple individual requests into a single request.</span></span> <span data-ttu-id="a2060-105">Esse padrão é útil quando um cliente deve fazer várias chamadas para diferentes sistemas de back-end ao executar uma operação.</span><span class="sxs-lookup"><span data-stu-id="a2060-105">This pattern is useful when a client must make multiple calls to different backend systems to perform an operation.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="a2060-106">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="a2060-106">Context and problem</span></span>

<span data-ttu-id="a2060-107">Para executar uma única tarefa, um cliente talvez precise fazer várias chamadas a vários serviços de back-end.</span><span class="sxs-lookup"><span data-stu-id="a2060-107">To perform a single task, a client may have to make multiple calls to various backend services.</span></span> <span data-ttu-id="a2060-108">Um aplicativo que depende de muitos serviços para executar uma tarefa deve gastar recursos em cada solicitação.</span><span class="sxs-lookup"><span data-stu-id="a2060-108">An application that relies on many services to perform a task must expend resources on each request.</span></span> <span data-ttu-id="a2060-109">Quando qualquer novo serviço ou recurso é adicionado ao aplicativo, solicitações adicionais são necessárias, além de aumentar os requisitos de recursos e chamadas de rede.</span><span class="sxs-lookup"><span data-stu-id="a2060-109">When any new feature or service is added to the application, additional requests are needed, further increasing resource requirements and network calls.</span></span> <span data-ttu-id="a2060-110">Esse excesso de conversa entre um cliente e um back-end pode afetar negativamente o desempenho e a escala do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="a2060-110">This chattiness between a client and a backend can adversely impact the performance and scale of the application.</span></span>  <span data-ttu-id="a2060-111">Arquiteturas de microsserviço tornaram esse problema mais comum, uma vez que aplicativos baseados em torno de muitos serviços menores naturalmente têm uma quantidade maior de chamadas entre serviços.</span><span class="sxs-lookup"><span data-stu-id="a2060-111">Microservice architectures have made this problem more common, as applications built around many smaller services naturally have a higher amount of cross-service calls.</span></span> 

<span data-ttu-id="a2060-112">No diagrama a seguir, o cliente envia solicitações a cada serviço (1,2,3).</span><span class="sxs-lookup"><span data-stu-id="a2060-112">In the following diagram, the client sends requests to each service (1,2,3).</span></span> <span data-ttu-id="a2060-113">Cada serviço processa a solicitação e envia a resposta de volta ao aplicativo (4,5,6).</span><span class="sxs-lookup"><span data-stu-id="a2060-113">Each service processes the request and sends the response back to the application (4,5,6).</span></span> <span data-ttu-id="a2060-114">Em uma rede celular com latência normalmente alta, usar solicitações individuais dessa maneira é ineficiente e pode resultar em interrupção da conectividade ou em solicitações incompletas.</span><span class="sxs-lookup"><span data-stu-id="a2060-114">Over a cellular network with typically high latency, using individual requests in this manner is inefficient and could result in broken connectivity or incomplete requests.</span></span> <span data-ttu-id="a2060-115">Embora cada solicitação possa ser executada em paralelo, o aplicativo deve enviar, aguardar e processar dados para cada solicitação, tudo em conexões separadas, aumentando a possibilidade de falha.</span><span class="sxs-lookup"><span data-stu-id="a2060-115">While each request may be done in parallel, the application must send, wait, and process data for each request, all on separate connections, increasing the chance of failure.</span></span>

![](./_images/gateway-aggregation-problem.png) 

## <a name="solution"></a><span data-ttu-id="a2060-116">Solução</span><span class="sxs-lookup"><span data-stu-id="a2060-116">Solution</span></span>

<span data-ttu-id="a2060-117">Use um gateway para reduzir a conversa entre o cliente e os serviços.</span><span class="sxs-lookup"><span data-stu-id="a2060-117">Use a gateway to reduce chattiness between the client and the services.</span></span> <span data-ttu-id="a2060-118">O gateway recebe solicitações de cliente, envia solicitações para os vários sistemas de back-end e, em seguida, agrega os resultados e envia-os de volta ao cliente solicitante.</span><span class="sxs-lookup"><span data-stu-id="a2060-118">The gateway receives client requests, dispatches requests to the various backend systems, and then aggregates the results and sends them back to the requesting client.</span></span>

<span data-ttu-id="a2060-119">Esse padrão pode reduzir o número de solicitações que o aplicativo faz a serviços back-end e melhorar o desempenho do aplicativo em redes de alta latência.</span><span class="sxs-lookup"><span data-stu-id="a2060-119">This pattern can reduce the number of requests that the application makes to backend services, and improve application performance over high-latency networks.</span></span>

<span data-ttu-id="a2060-120">No diagrama a seguir, o aplicativo envia uma solicitação para o gateway (1).</span><span class="sxs-lookup"><span data-stu-id="a2060-120">In the following diagram, the application sends a request to the gateway (1).</span></span> <span data-ttu-id="a2060-121">A solicitação contém um pacote de solicitações adicionais.</span><span class="sxs-lookup"><span data-stu-id="a2060-121">The request contains a package of additional requests.</span></span> <span data-ttu-id="a2060-122">O gateway as decompõe e processa cada solicitação enviando-a ao serviço em questão (2).</span><span class="sxs-lookup"><span data-stu-id="a2060-122">The gateway decomposes these and processes each request by sending it to the relevant service (2).</span></span> <span data-ttu-id="a2060-123">Cada serviço retorna uma resposta ao gateway (3).</span><span class="sxs-lookup"><span data-stu-id="a2060-123">Each service returns a response to the gateway (3).</span></span> <span data-ttu-id="a2060-124">O gateway combina as respostas de cada serviço e envia a resposta ao aplicativo (4).</span><span class="sxs-lookup"><span data-stu-id="a2060-124">The gateway combines the responses from each service and sends the response to the application (4).</span></span> <span data-ttu-id="a2060-125">O aplicativo faz uma única solicitação e recebe apenas uma única resposta do gateway.</span><span class="sxs-lookup"><span data-stu-id="a2060-125">The application makes a single request and receives only a single response from the gateway.</span></span>

![](./_images/gateway-aggregation.png)

## <a name="issues-and-considerations"></a><span data-ttu-id="a2060-126">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="a2060-126">Issues and considerations</span></span>

- <span data-ttu-id="a2060-127">O gateway não deve introduzir acoplamento de serviço entre os serviços de back-end.</span><span class="sxs-lookup"><span data-stu-id="a2060-127">The gateway should not introduce service coupling across the backend services.</span></span>
- <span data-ttu-id="a2060-128">O gateway deve estar localizado próximo aos serviços de back-end para reduzir a latência tanto quanto possível.</span><span class="sxs-lookup"><span data-stu-id="a2060-128">The gateway should be located near the backend services to reduce latency as much as possible.</span></span>
- <span data-ttu-id="a2060-129">O serviço de gateway pode introduzir um ponto único de falha.</span><span class="sxs-lookup"><span data-stu-id="a2060-129">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="a2060-130">Verifique se o gateway está corretamente projetado para atender aos requisitos de disponibilidade do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="a2060-130">Ensure the gateway is properly designed to meet your application's availability requirements.</span></span>
- <span data-ttu-id="a2060-131">O gateway pode introduzir um gargalo.</span><span class="sxs-lookup"><span data-stu-id="a2060-131">The gateway may introduce a bottleneck.</span></span> <span data-ttu-id="a2060-132">O gateway deve ter um desempenho adequado para lidar com a carga e pode ser dimensionado para atender o seu crescimento previsto.</span><span class="sxs-lookup"><span data-stu-id="a2060-132">Ensure the gateway has adequate performance to handle load and can be scaled to meet your anticipated growth.</span></span>
- <span data-ttu-id="a2060-133">Execute testes de carga no gateway para garantir que você não introduza falhas em cascata para serviços.</span><span class="sxs-lookup"><span data-stu-id="a2060-133">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="a2060-134">Implemente um design resiliente usando técnicas como [bulkheads][bulkhead], [interrupção de circuito][circuit-breaker], [nova tentativa][retry] e tempos limite.</span><span class="sxs-lookup"><span data-stu-id="a2060-134">Implement a resilient design, using techniques such as [bulkheads][bulkhead], [circuit breaking][circuit-breaker], [retry][retry], and timeouts.</span></span>
- <span data-ttu-id="a2060-135">Se uma ou mais chamadas de serviço levarem tempo demais, pode ser aceitável atingir o tempo limite e retornar um conjunto parcial de dados.</span><span class="sxs-lookup"><span data-stu-id="a2060-135">If one or more service calls takes too long, it may be acceptable to timeout and return a partial set of data.</span></span> <span data-ttu-id="a2060-136">Considere como o aplicativo tratará esse cenário.</span><span class="sxs-lookup"><span data-stu-id="a2060-136">Consider how your application will handle this scenario.</span></span>
- <span data-ttu-id="a2060-137">Use E/S assíncrona para garantir que um atraso no back-end não cause problemas de desempenho no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="a2060-137">Use asynchronous I/O to ensure that a delay at the backend doesn't cause performance issues in the application.</span></span>
- <span data-ttu-id="a2060-138">Implemente rastreamento distribuído usando IDs de correlação para rastrear cada chamada individual.</span><span class="sxs-lookup"><span data-stu-id="a2060-138">Implement distributed tracing using correlation IDs to track each individual call.</span></span>
- <span data-ttu-id="a2060-139">Monitore métricas de solicitação e tamanhos de resposta.</span><span class="sxs-lookup"><span data-stu-id="a2060-139">Monitor request metrics and response sizes.</span></span>
- <span data-ttu-id="a2060-140">Considere retornar dados em cache como uma estratégia de failover para lidar com falhas.</span><span class="sxs-lookup"><span data-stu-id="a2060-140">Consider returning cached data as a failover strategy to handle failures.</span></span>
- <span data-ttu-id="a2060-141">Em vez de criar a agregação para o gateway, considere colocar um serviço de agregação atrás do gateway.</span><span class="sxs-lookup"><span data-stu-id="a2060-141">Instead of building aggregation into the gateway, consider placing an aggregation service behind the gateway.</span></span> <span data-ttu-id="a2060-142">Agregação de solicitação provavelmente terá requisitos de recursos diferentes de outros serviços no gateway e poderá afetar a funcionalidade de roteamento e descarregamento do gateway.</span><span class="sxs-lookup"><span data-stu-id="a2060-142">Request aggregation will likely have different resource requirements than other services in the gateway and may impact the gateway's routing and offloading functionality.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="a2060-143">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="a2060-143">When to use this pattern</span></span>

<span data-ttu-id="a2060-144">Use esse padrão quando:</span><span class="sxs-lookup"><span data-stu-id="a2060-144">Use this pattern when:</span></span>

- <span data-ttu-id="a2060-145">Um cliente precisa se comunicar com vários serviços de back-end para executar uma operação.</span><span class="sxs-lookup"><span data-stu-id="a2060-145">A client needs to communicate with multiple backend services to perform an operation.</span></span>
- <span data-ttu-id="a2060-146">O cliente puder usar redes com latência significativa, como redes de celulares.</span><span class="sxs-lookup"><span data-stu-id="a2060-146">The client may use networks with significant latency, such as cellular networks.</span></span>

<span data-ttu-id="a2060-147">Esse padrão pode não ser adequado quando:</span><span class="sxs-lookup"><span data-stu-id="a2060-147">This pattern may not be suitable when:</span></span>

- <span data-ttu-id="a2060-148">Você quiser reduzir o número de chamadas entre um cliente e um único serviço entre várias operações.</span><span class="sxs-lookup"><span data-stu-id="a2060-148">You want to reduce the number of calls between a client and a single service across multiple operations.</span></span> <span data-ttu-id="a2060-149">Nesse cenário, talvez seja melhor adicionar uma operação em lote ao serviço.</span><span class="sxs-lookup"><span data-stu-id="a2060-149">In that scenario, it may be better to add a batch operation to the service.</span></span>
- <span data-ttu-id="a2060-150">O cliente ou aplicativo está localizado próximo dos serviços de back-end e a latência não é um fator significativo.</span><span class="sxs-lookup"><span data-stu-id="a2060-150">The client or application is located near the backend services and latency is not a significant factor.</span></span>

## <a name="example"></a><span data-ttu-id="a2060-151">Exemplo</span><span class="sxs-lookup"><span data-stu-id="a2060-151">Example</span></span>

<span data-ttu-id="a2060-152">O exemplo a seguir ilustra como criar um serviço NGINX de agregação do gateway simples usando Lua.</span><span class="sxs-lookup"><span data-stu-id="a2060-152">The following example illustrates how to create a simple a gateway aggregation NGINX service using Lua.</span></span>

```lua
worker_processes  4;

events {
  worker_connections 1024;
}

http {
  server {
    listen 80;

    location = /batch {
      content_by_lua '
        ngx.req.read_body()

        -- read json body content
        local cjson = require "cjson"
        local batch = cjson.decode(ngx.req.get_body_data())["batch"]

        -- create capture_multi table
        local requests = {}
        for i, item in ipairs(batch) do
          table.insert(requests, {item.relative_url, { method = ngx.HTTP_GET}})
        end

        -- execute batch requests in parallel
        local results = {}
        local resps = { ngx.location.capture_multi(requests) }
        for i, res in ipairs(resps) do
          table.insert(results, {status = res.status, body = cjson.decode(res.body), header = res.header})
        end

        ngx.say(cjson.encode({results = results}))
      ';
    }

    location = /service1 {
      default_type application/json;
      echo '{"attr1":"val1"}';
    }

    location = /service2 {
      default_type application/json;
      echo '{"attr2":"val2"}';
    }
  }
}
```

## <a name="related-guidance"></a><span data-ttu-id="a2060-153">Diretrizes relacionadas</span><span class="sxs-lookup"><span data-stu-id="a2060-153">Related guidance</span></span>

- [<span data-ttu-id="a2060-154">Padrão de back-ends para front-ends</span><span class="sxs-lookup"><span data-stu-id="a2060-154">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="a2060-155">Padrão de descarregamento de gateway</span><span class="sxs-lookup"><span data-stu-id="a2060-155">Gateway Offloading pattern</span></span>](./gateway-offloading.md)
- [<span data-ttu-id="a2060-156">Padrão de roteamento de gateway</span><span class="sxs-lookup"><span data-stu-id="a2060-156">Gateway Routing pattern</span></span>](./gateway-routing.md)

[bulkhead]: ./bulkhead.md
[circuit-breaker]: ./circuit-breaker.md
[retry]: ./retry.md