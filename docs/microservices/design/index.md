---
title: Implementação de referência de microsserviços no Serviço de Kubernetes do Azure
description: Essa implementação de referência ilustra as práticas recomendadas de uma arquitetura de microsserviços.
author: MikeWasson
ms.date: 02/26/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 17e275e5b5f45233f7467192402cb28fce35c57b
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59068898"
---
# <a name="designing-a-microservices-architecture"></a><span data-ttu-id="96d67-103">Projetar uma arquitetura de microsserviços</span><span class="sxs-lookup"><span data-stu-id="96d67-103">Designing a microservices architecture</span></span>

<span data-ttu-id="96d67-104">Microsserviços se tornaram um estilo popular de arquitetura para criar aplicativos de nuvem resilientes, altamente escalonáveis, implantáveis independentemente e capazes de evoluir rapidamente.</span><span class="sxs-lookup"><span data-stu-id="96d67-104">Microservices have become a popular architectural style for building cloud applications that are resilient, highly scalable, independently deployable, and able to evolve quickly.</span></span> <span data-ttu-id="96d67-105">No entanto, para serem mais do que apenas uma palavra de efeito, microsserviços exigem uma abordagem diferente para o design e criação de aplicativos.</span><span class="sxs-lookup"><span data-stu-id="96d67-105">To be more than just a buzzword, however, microservices require a different approach to designing and building applications.</span></span>

<span data-ttu-id="96d67-106">Nesse conjunto de artigos, exploraremos como compilar e executar uma arquitetura de microsserviços no Azure.</span><span class="sxs-lookup"><span data-stu-id="96d67-106">In this set of articles, we explore how to build and run a microservices architecture on Azure.</span></span> <span data-ttu-id="96d67-107">Os tópicos incluem:</span><span class="sxs-lookup"><span data-stu-id="96d67-107">Topics include:</span></span>

- [<span data-ttu-id="96d67-108">Comunicação entre serviços</span><span class="sxs-lookup"><span data-stu-id="96d67-108">Interservice communication</span></span>](./interservice-communication.md)
- [<span data-ttu-id="96d67-109">Design de API</span><span class="sxs-lookup"><span data-stu-id="96d67-109">API design</span></span>](./api-design.md)
- [<span data-ttu-id="96d67-110">Gateways de API</span><span class="sxs-lookup"><span data-stu-id="96d67-110">API gateways</span></span>](./gateway.md)
- [<span data-ttu-id="96d67-111">Considerações sobre dados</span><span class="sxs-lookup"><span data-stu-id="96d67-111">Data considerations</span></span>](./data-considerations.md)
- [<span data-ttu-id="96d67-112">Padrões de design</span><span class="sxs-lookup"><span data-stu-id="96d67-112">Design patterns</span></span>](./patterns.md)

## <a name="prerequisites"></a><span data-ttu-id="96d67-113">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="96d67-113">Prerequisites</span></span>

<span data-ttu-id="96d67-114">Antes de ler estes artigos, você pode começar por:</span><span class="sxs-lookup"><span data-stu-id="96d67-114">Before reading these articles, you might start with the following:</span></span>

- <span data-ttu-id="96d67-115">[Introdução às arquiteturas de microsserviços](../introduction.md).</span><span class="sxs-lookup"><span data-stu-id="96d67-115">[Introduction to microservices architectures](../introduction.md).</span></span> <span data-ttu-id="96d67-116">Entenda os benefícios e desafios dos microsserviços e quando usar esse estilo de arquitetura.</span><span class="sxs-lookup"><span data-stu-id="96d67-116">Understand the benefits and challenges of microservices, and when to use this style of architecture.</span></span>
- <span data-ttu-id="96d67-117">[Usar análise de domínio para microsserviços de modelo](../model/domain-analysis.md).</span><span class="sxs-lookup"><span data-stu-id="96d67-117">[Using domain analysis to model microservices](../model/domain-analysis.md).</span></span> <span data-ttu-id="96d67-118">Aprenda uma abordagem baseada em domínio a microsserviços de modelagem.</span><span class="sxs-lookup"><span data-stu-id="96d67-118">Learn a domain-driven approach to modeling microservices.</span></span>

## <a name="reference-implementation"></a><span data-ttu-id="96d67-119">Implementação de referência</span><span class="sxs-lookup"><span data-stu-id="96d67-119">Reference implementation</span></span>

<span data-ttu-id="96d67-120">Para ilustrar as práticas recomendadas de uma arquitetura de microsserviços, criamos uma implementação de referência que chamamos de aplicativo de Entrega por Drone.</span><span class="sxs-lookup"><span data-stu-id="96d67-120">To illustrate best practices for a microservices architecture, we created a reference implementation that we call the Drone Delivery application.</span></span> <span data-ttu-id="96d67-121">Essa implementação é executada no Kubernetes usando o Serviço de Kubernetes do Azure (AKS).</span><span class="sxs-lookup"><span data-stu-id="96d67-121">This implementation runs on Kubernetes using Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="96d67-122">Você pode encontrar a implementação de referência no [GitHub][drone-ri].</span><span class="sxs-lookup"><span data-stu-id="96d67-122">You can find the reference implementation on [GitHub][drone-ri].</span></span>

![Arquitetura do aplicativo de Entrega por Drone](../images/drone-delivery.png)

## <a name="scenario"></a><span data-ttu-id="96d67-124">Cenário</span><span class="sxs-lookup"><span data-stu-id="96d67-124">Scenario</span></span>

<span data-ttu-id="96d67-125">A Fabrikam, Inc. está dando início a um serviço de entrega por drone.</span><span class="sxs-lookup"><span data-stu-id="96d67-125">Fabrikam, Inc. is starting a drone delivery service.</span></span> <span data-ttu-id="96d67-126">A empresa gerencia uma frota de aeronaves de tipo drone.</span><span class="sxs-lookup"><span data-stu-id="96d67-126">The company manages a fleet of drone aircraft.</span></span> <span data-ttu-id="96d67-127">Empresas registram-se no serviço e os usuários podem solicitar que um drone colete mercadorias para entrega.</span><span class="sxs-lookup"><span data-stu-id="96d67-127">Businesses register with the service, and users can request a drone to pick up goods for delivery.</span></span> <span data-ttu-id="96d67-128">Quando um cliente agenda uma coleta, um sistema de back-end atribui um drone ao usuário e notifica-o com um tempo de entrega previsto.</span><span class="sxs-lookup"><span data-stu-id="96d67-128">When a customer schedules a pickup, a backend system assigns a drone and notifies the user with an estimated delivery time.</span></span> <span data-ttu-id="96d67-129">Enquanto a entrega está em andamento, o cliente pode acompanhar a localização do drone, com um ETA atualizado continuamente.</span><span class="sxs-lookup"><span data-stu-id="96d67-129">While the delivery is in progress, the customer can track the location of the drone, with a continuously updated ETA.</span></span>

<span data-ttu-id="96d67-130">Este cenário envolve um domínio bastante complicado.</span><span class="sxs-lookup"><span data-stu-id="96d67-130">This scenario involves a fairly complicated domain.</span></span> <span data-ttu-id="96d67-131">Algumas das preocupações de negócios incluem o agendamento drones, o acompanhamento de pacotes, o gerenciamento de contas de usuário e o armazenamento e análise de dados históricos.</span><span class="sxs-lookup"><span data-stu-id="96d67-131">Some of the business concerns include scheduling drones, tracking packages, managing user accounts, and storing and analyzing historical data.</span></span> <span data-ttu-id="96d67-132">Além disso, Fabrikam deseja se lançar no mercado rapidamente e, em seguida, iterar rapidamente, adicionando novas funcionalidades e capacidades.</span><span class="sxs-lookup"><span data-stu-id="96d67-132">Moreover, Fabrikam wants to get to market quickly and then iterate quickly, adding new functionality and capabilities.</span></span> <span data-ttu-id="96d67-133">O aplicativo deve operar em escala de nuvem, com um SLO (objetivo do nível de serviço) elevado.</span><span class="sxs-lookup"><span data-stu-id="96d67-133">The application needs to operate at cloud scale, with a high service level objective (SLO).</span></span> <span data-ttu-id="96d67-134">A Fabrikam também espera que diferentes partes do sistema tenham requisitos muito diferentes de armazenamento e consulta de dados.</span><span class="sxs-lookup"><span data-stu-id="96d67-134">Fabrikam also expects that different parts of the system will have very different requirements for data storage and querying.</span></span> <span data-ttu-id="96d67-135">Todas essas considerações levam Fabrikam para escolher uma arquitetura de microsserviços para o aplicativo de Entrega por Drone.</span><span class="sxs-lookup"><span data-stu-id="96d67-135">All of these considerations lead Fabrikam to choose a microservices architecture for the Drone Delivery application.</span></span>

> [!NOTE]
> <span data-ttu-id="96d67-136">Para obter ajuda na escolha entre uma arquitetura de microsserviços e outros estilos de arquitetura, consulte o [Guia de arquitetura de aplicativo do Azure](../../guide/index.md).</span><span class="sxs-lookup"><span data-stu-id="96d67-136">For help in choosing between a microservices architecture and other architectural styles, see the [Azure Application Architecture Guide](../../guide/index.md).</span></span>

<span data-ttu-id="96d67-137">Nossa implementação de referência usa Kubernetes com o [AKS (Serviço de Kubernetes do Azure)](/azure/aks/).</span><span class="sxs-lookup"><span data-stu-id="96d67-137">Our reference implementation uses Kubernetes with [Azure Kubernetes Service](/azure/aks/) (AKS).</span></span> <span data-ttu-id="96d67-138">No entanto, muitas das decisões de arquitetura de alto nível e desafios se aplicarão a qualquer orquestrador de contêineres, incluindo o [Azure Service Fabric](/azure/service-fabric/).</span><span class="sxs-lookup"><span data-stu-id="96d67-138">However, many of the high-level architectural decisions and challenges will apply to any container orchestrator, including [Azure Service Fabric](/azure/service-fabric/).</span></span>

<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation/tree/v0.1.0-orig

## <a name="next-steps"></a><span data-ttu-id="96d67-139">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="96d67-139">Next steps</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="96d67-140">Escolher uma opção de computação</span><span class="sxs-lookup"><span data-stu-id="96d67-140">Choose a compute option</span></span>](./compute-options.md)