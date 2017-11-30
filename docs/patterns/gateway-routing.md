---
title: "Padrão de roteamento de gateway"
description: "Faça o roteamento de solicitações para vários serviços usando um único ponto de extremidade."
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 53239b23cfd98fad1edc38ca37c2274d5a9d7a0f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="gateway-routing-pattern"></a><span data-ttu-id="1102f-103">Padrão de roteamento de gateway</span><span class="sxs-lookup"><span data-stu-id="1102f-103">Gateway Routing pattern</span></span>

<span data-ttu-id="1102f-104">Faça o roteamento de solicitações para vários serviços usando um único ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="1102f-104">Route requests to multiple services using a single endpoint.</span></span> <span data-ttu-id="1102f-105">Esse padrão é útil quando você deseja expor vários serviços em um único ponto de extremidade e rotear para o serviço apropriado com base na solicitação.</span><span class="sxs-lookup"><span data-stu-id="1102f-105">This pattern is useful when you wish to expose multiple services on a single endpoint and route to the appropriate service based on the request.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="1102f-106">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="1102f-106">Context and problem</span></span>

<span data-ttu-id="1102f-107">Quando um cliente precisa consumir vários serviços, pode ser um desafio configurar um ponto de extremidade separado para cada serviço e precisar que o cliente gerencie cada ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="1102f-107">When a client needs to consume multiple services, setting up a separate endpoint for each service and having the client manage each endpoint can be challenging.</span></span> <span data-ttu-id="1102f-108">Por exemplo, um aplicativo de comércio eletrônico pode fornecer serviços como pesquisa, análises, carrinho de compras, check-out e histórico de pedidos.</span><span class="sxs-lookup"><span data-stu-id="1102f-108">For example, an e-commerce application might provide services such as search, reviews, cart, checkout, and order history.</span></span> <span data-ttu-id="1102f-109">Cada serviço tem uma API diferente com a qual o cliente deve interagir, e ele deve conhecer cada ponto de extremidade para se conectar aos serviços.</span><span class="sxs-lookup"><span data-stu-id="1102f-109">Each service has a different API that the client must interact with, and the client must know about each endpoint in order to connect to the services.</span></span> <span data-ttu-id="1102f-110">Se uma API muda, o cliente deverá ser atualizado também.</span><span class="sxs-lookup"><span data-stu-id="1102f-110">If an API changes, the client must be updated as well.</span></span> <span data-ttu-id="1102f-111">Se você refatorar um serviço em dois ou mais serviços separados, o código deve ser alterado no serviço e no cliente.</span><span class="sxs-lookup"><span data-stu-id="1102f-111">If you refactor a service into two or more separate services, the code must change in both the service and the client.</span></span>

## <a name="solution"></a><span data-ttu-id="1102f-112">Solução</span><span class="sxs-lookup"><span data-stu-id="1102f-112">Solution</span></span>

<span data-ttu-id="1102f-113">Coloque um gateway em frente a um conjunto de aplicativos, serviços ou implantações.</span><span class="sxs-lookup"><span data-stu-id="1102f-113">Place a gateway in front of a set of applications, services, or deployments.</span></span> <span data-ttu-id="1102f-114">Use o roteamento da camada de aplicativo 7 para rotear a solicitação para as instâncias apropriadas.</span><span class="sxs-lookup"><span data-stu-id="1102f-114">Use application Layer 7 routing to route the request to the appropriate instances.</span></span>

<span data-ttu-id="1102f-115">Com esse padrão, o aplicativo cliente só precisa conhecer e se comunicar com um único ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="1102f-115">With this pattern, the client application only needs to know about and communicate with a single endpoint.</span></span> <span data-ttu-id="1102f-116">Se um serviço for consolidado ou decomposto, o cliente não precisa necessariamente de atualização.</span><span class="sxs-lookup"><span data-stu-id="1102f-116">If a service is consolidated or decomposed, the client does not necessarily require updating.</span></span> <span data-ttu-id="1102f-117">Ele pode continuar a fazer solicitações para o gateway, e apenas o roteamento muda.</span><span class="sxs-lookup"><span data-stu-id="1102f-117">It can continue making requests to the gateway, and only the routing changes.</span></span>

<span data-ttu-id="1102f-118">Um gateway também permite abstrair os serviços de back-end dos clientes, permitindo que você mantenha as chamadas do cliente simples ao mesmo tempo que permite alterações nos serviços de back-end por trás do gateway.</span><span class="sxs-lookup"><span data-stu-id="1102f-118">A gateway also lets you abstract backend services from the clients, allowing you to keep client calls simple while enabling changes in the backend services behind the gateway.</span></span> <span data-ttu-id="1102f-119">Chamadas de cliente podem ser roteadas para qualquer serviço ou os serviços precisam controlar o comportamento esperado do cliente, permitindo que você adicione, divida e reorganize serviços por trás do gateway sem alterar o cliente.</span><span class="sxs-lookup"><span data-stu-id="1102f-119">Client calls can be routed to whatever service or services need to handle the expected client behavior, allowing you to add, split, and reorganize services behind the gateway without changing the client.</span></span>

![](./_images/gateway-routing.png)
 
<span data-ttu-id="1102f-120">Esse padrão também pode ajudar com a implantação ao permitir que você gerencie como as atualizações são distribuídas aos usuários.</span><span class="sxs-lookup"><span data-stu-id="1102f-120">This pattern can also help with deployment, by allowing you to manage how updates are rolled out to users.</span></span> <span data-ttu-id="1102f-121">Quando uma nova versão do seu serviço é implantada, ela pode ser implantada em paralelo com a versão existente.</span><span class="sxs-lookup"><span data-stu-id="1102f-121">When a new version of your service is deployed, it can be deployed in parallel with the existing version.</span></span> <span data-ttu-id="1102f-122">O roteamento permite controlar qual versão do serviço é apresentada aos clientes, dando a você a flexibilidade para usar várias estratégias de versão, seja ela incremental, paralela ou de distribuições completas das atualizações.</span><span class="sxs-lookup"><span data-stu-id="1102f-122">Routing let you control what version of the service is presented to the clients, giving you the flexibility to use various release strategies, whether incremental, parallel, or complete rollouts of updates.</span></span> <span data-ttu-id="1102f-123">Quaisquer problemas descobertos depois que o novo serviço for implantado podem ser revertidos rapidamente por meio da alteração de uma configuração no gateway e sem afetar os clientes.</span><span class="sxs-lookup"><span data-stu-id="1102f-123">Any issues discovered after the new service is deployed can be quickly reverted by making a configuration change at the gateway, without affecting clients.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="1102f-124">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="1102f-124">Issues and considerations</span></span>

- <span data-ttu-id="1102f-125">O serviço de gateway pode introduzir um ponto único de falha.</span><span class="sxs-lookup"><span data-stu-id="1102f-125">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="1102f-126">Verifique se ele está corretamente projetado para atender aos requisitos de disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="1102f-126">Ensure it is properly designed to meet your availability requirements.</span></span> <span data-ttu-id="1102f-127">Ao implementar, leve em consideração os recursos de resiliência e tolerância a falhas.</span><span class="sxs-lookup"><span data-stu-id="1102f-127">Consider resiliency and fault tolerance capabilities when implementing.</span></span>
- <span data-ttu-id="1102f-128">O serviço de gateway pode introduzir um gargalo.</span><span class="sxs-lookup"><span data-stu-id="1102f-128">The gateway service may introduce a bottleneck.</span></span> <span data-ttu-id="1102f-129">O gateway deve ter um desempenho adequado para lidar com a carga e pode facilmente ser dimensionado em linha com o crescimento das expectativas.</span><span class="sxs-lookup"><span data-stu-id="1102f-129">Ensure the gateway has adequate performance to handle load and can easily scale in line with your growth expectations.</span></span>
- <span data-ttu-id="1102f-130">Execute testes de carga no gateway para garantir que você não introduza falhas em cascata para serviços.</span><span class="sxs-lookup"><span data-stu-id="1102f-130">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="1102f-131">O roteamento de gateway é de nível 7.</span><span class="sxs-lookup"><span data-stu-id="1102f-131">Gateway routing is level 7.</span></span> <span data-ttu-id="1102f-132">Ele pode ser baseado em IP, porta, cabeçalho ou URL.</span><span class="sxs-lookup"><span data-stu-id="1102f-132">It can be based on IP, port, header, or URL.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="1102f-133">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="1102f-133">When to use this pattern</span></span>

<span data-ttu-id="1102f-134">Use esse padrão quando:</span><span class="sxs-lookup"><span data-stu-id="1102f-134">Use this pattern when:</span></span>

- <span data-ttu-id="1102f-135">Um cliente precisa consumir vários serviços que podem ser acessados por trás de um gateway.</span><span class="sxs-lookup"><span data-stu-id="1102f-135">A client needs to consume multiple services that can be accessed behind a gateway.</span></span>
- <span data-ttu-id="1102f-136">Você deseja simplificar os aplicativos cliente usando um único ponto de extremidade.</span><span class="sxs-lookup"><span data-stu-id="1102f-136">You wish to simplify client applications by using a single endpoint.</span></span>
- <span data-ttu-id="1102f-137">Você precisa rotear solicitações de pontos de extremidade endereçáveis externamente para pontos de extremidade virtuais internos, como expor as portas em uma VM para endereços IP virtuais do cluster.</span><span class="sxs-lookup"><span data-stu-id="1102f-137">You need to route requests from externally addressable endpoints to internal virtual endpoints, such as exposing ports on a VM to cluster virtual IP addresses.</span></span>

<span data-ttu-id="1102f-138">Esse padrão pode não ser adequado quando você tiver um aplicativo simples que usa apenas um ou dois serviços.</span><span class="sxs-lookup"><span data-stu-id="1102f-138">This pattern may not be suitable when you have a simple application that uses only one or two services.</span></span>

## <a name="example"></a><span data-ttu-id="1102f-139">Exemplo</span><span class="sxs-lookup"><span data-stu-id="1102f-139">Example</span></span>

<span data-ttu-id="1102f-140">Ao usar o Nginx como o roteador, a seguir há um arquivo de configuração de exemplo simples de um servidor que encaminha solicitações para aplicativos que residem em diferentes diretórios virtuais para diferentes computadores no back-end.</span><span class="sxs-lookup"><span data-stu-id="1102f-140">Using Nginx as the router, the following is a simple example configuration file for a server that routes requests for applications residing on different virtual directories to different machines at the back end.</span></span>

```
server {
    listen 80;
    server_name domain.com;

    location /app1 {
        proxy_pass http://10.0.3.10:80;
    }

    location /app2 {
        proxy_pass http://10.0.3.20:80;
    }

    location /app3 {
        proxy_pass http://10.0.3.30:80;
    }
}
```

## <a name="related-guidance"></a><span data-ttu-id="1102f-141">Diretrizes relacionadas</span><span class="sxs-lookup"><span data-stu-id="1102f-141">Related guidance</span></span>

- [<span data-ttu-id="1102f-142">Padrão de back-ends para front-ends</span><span class="sxs-lookup"><span data-stu-id="1102f-142">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="1102f-143">Padrão de agregação de gateway</span><span class="sxs-lookup"><span data-stu-id="1102f-143">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="1102f-144">Padrão de descarregamento de gateway</span><span class="sxs-lookup"><span data-stu-id="1102f-144">Gateway Offloading pattern</span></span>](./gateway-offloading.md)



