---
title: Estilo de arquitetura de trabalho de fila da Web
titleSuffix: Azure Application Architecture Guide
description: Descreve os benefícios, os desafios e as melhores práticas para arquiteturas de trabalho de fila da Web no Azure.
author: MikeWasson
ms.date: 04/10/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 974b8b8595d6d9333552c41dfe1f3f2af848d264
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/18/2019
ms.locfileid: "59740384"
---
# <a name="web-queue-worker-architecture-style"></a><span data-ttu-id="edde7-103">Estilo de arquitetura de trabalho de fila da Web</span><span class="sxs-lookup"><span data-stu-id="edde7-103">Web-Queue-Worker architecture style</span></span>

<span data-ttu-id="edde7-104">Os componentes principais dessa arquitetura são um **front-end da Web** que atende a solicitações de cliente e um **trabalho** que executa tarefas de uso intensivo de recursos, fluxos de trabalho de longa execução ou trabalhos em lotes.</span><span class="sxs-lookup"><span data-stu-id="edde7-104">The core components of this architecture are a **web front end** that serves client requests, and a **worker** that performs resource-intensive tasks, long-running workflows, or batch jobs.</span></span>  <span data-ttu-id="edde7-105">O front-end da Web se comunica com o trabalho por meio de uma **fila de mensagens**.</span><span class="sxs-lookup"><span data-stu-id="edde7-105">The web front end communicates with the worker through a **message queue**.</span></span>

![Diagrama lógico do estilo de arquitetura de trabalho de fila da Web](./images/web-queue-worker-logical.svg)

<span data-ttu-id="edde7-107">Outros componentes que são comumente incorporados a essa arquitetura são:</span><span class="sxs-lookup"><span data-stu-id="edde7-107">Other components that are commonly incorporated into this architecture include:</span></span>

- <span data-ttu-id="edde7-108">Um ou mais bancos de dados.</span><span class="sxs-lookup"><span data-stu-id="edde7-108">One or more databases.</span></span>
- <span data-ttu-id="edde7-109">Um cache para armazenar valores do banco de dados para leituras rápidas.</span><span class="sxs-lookup"><span data-stu-id="edde7-109">A cache to store values from the database for quick reads.</span></span>
- <span data-ttu-id="edde7-110">CDN para atender a conteúdo estático</span><span class="sxs-lookup"><span data-stu-id="edde7-110">CDN to serve static content</span></span>
- <span data-ttu-id="edde7-111">Serviços remotos, como email ou serviço de SMS.</span><span class="sxs-lookup"><span data-stu-id="edde7-111">Remote services, such as email or SMS service.</span></span> <span data-ttu-id="edde7-112">Geralmente, eles são fornecidos por terceiros.</span><span class="sxs-lookup"><span data-stu-id="edde7-112">Often these are provided by third parties.</span></span>
- <span data-ttu-id="edde7-113">Provedor de identidade para autenticação.</span><span class="sxs-lookup"><span data-stu-id="edde7-113">Identity provider for authentication.</span></span>

<span data-ttu-id="edde7-114">A Web e o trabalho são sem estado.</span><span class="sxs-lookup"><span data-stu-id="edde7-114">The web and worker are both stateless.</span></span> <span data-ttu-id="edde7-115">O estado de sessão pode ser armazenado em um cache distribuído.</span><span class="sxs-lookup"><span data-stu-id="edde7-115">Session state can be stored in a distributed cache.</span></span> <span data-ttu-id="edde7-116">Qualquer trabalho de longa execução é feito assincronamente pela função de trabalho.</span><span class="sxs-lookup"><span data-stu-id="edde7-116">Any long-running work is done asynchronously by the worker.</span></span> <span data-ttu-id="edde7-117">A função de trabalho pode ser acionada por mensagens na fila ou executada em um agendamento de processamento em lotes.</span><span class="sxs-lookup"><span data-stu-id="edde7-117">The worker can be triggered by messages on the queue, or run on a schedule for batch processing.</span></span> <span data-ttu-id="edde7-118">A função de trabalho é um componente opcional.</span><span class="sxs-lookup"><span data-stu-id="edde7-118">The worker is an optional component.</span></span> <span data-ttu-id="edde7-119">Se não houver nenhuma operação de longa execução, o trabalho pode ser omitido.</span><span class="sxs-lookup"><span data-stu-id="edde7-119">If there are no long-running operations, the worker can be omitted.</span></span>

<span data-ttu-id="edde7-120">O front-end pode consistir em uma API da Web.</span><span class="sxs-lookup"><span data-stu-id="edde7-120">The front end might consist of a web API.</span></span> <span data-ttu-id="edde7-121">Da parte do cliente, a API da Web pode ser consumida por um aplicativo de página única que faz chamadas AJAX ou por um aplicativo cliente nativo.</span><span class="sxs-lookup"><span data-stu-id="edde7-121">On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="edde7-122">Quando usar essa arquitetura</span><span class="sxs-lookup"><span data-stu-id="edde7-122">When to use this architecture</span></span>

<span data-ttu-id="edde7-123">A arquitetura de trabalho de fila da Web geralmente é implementada usando serviços de computação gerenciados, tanto o Serviço de Aplicativo do Azure ou Serviços de Nuvem do Azure.</span><span class="sxs-lookup"><span data-stu-id="edde7-123">The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.</span></span>

<span data-ttu-id="edde7-124">Considere esse estilo de arquitetura para:</span><span class="sxs-lookup"><span data-stu-id="edde7-124">Consider this architecture style for:</span></span>

- <span data-ttu-id="edde7-125">Aplicativos com um domínio relativamente simples.</span><span class="sxs-lookup"><span data-stu-id="edde7-125">Applications with a relatively simple domain.</span></span>
- <span data-ttu-id="edde7-126">Aplicativos com alguns fluxos de trabalho de longa execução ou operações em lote.</span><span class="sxs-lookup"><span data-stu-id="edde7-126">Applications with some long-running workflows or batch operations.</span></span>
- <span data-ttu-id="edde7-127">Quando você quiser usar serviços gerenciados em vez de infraestrutura como serviço (IaaS).</span><span class="sxs-lookup"><span data-stu-id="edde7-127">When you want to use managed services, rather than infrastructure as a service (IaaS).</span></span>

## <a name="benefits"></a><span data-ttu-id="edde7-128">Benefícios</span><span class="sxs-lookup"><span data-stu-id="edde7-128">Benefits</span></span>

- <span data-ttu-id="edde7-129">Arquitetura relativamente simples e fácil de entender.</span><span class="sxs-lookup"><span data-stu-id="edde7-129">Relatively simple architecture that is easy to understand.</span></span>
- <span data-ttu-id="edde7-130">Fácil de implantar e gerenciar.</span><span class="sxs-lookup"><span data-stu-id="edde7-130">Easy to deploy and manage.</span></span>
- <span data-ttu-id="edde7-131">Clara separação de preocupações.</span><span class="sxs-lookup"><span data-stu-id="edde7-131">Clear separation of concerns.</span></span>
- <span data-ttu-id="edde7-132">O front-end é separado do trabalho usando o sistema de mensagens assíncrono.</span><span class="sxs-lookup"><span data-stu-id="edde7-132">The front end is decoupled from the worker using asynchronous messaging.</span></span>
- <span data-ttu-id="edde7-133">O front-end e o trabalho podem ser dimensionado independentemente.</span><span class="sxs-lookup"><span data-stu-id="edde7-133">The front end and the worker can be scaled independently.</span></span>

## <a name="challenges"></a><span data-ttu-id="edde7-134">Desafios</span><span class="sxs-lookup"><span data-stu-id="edde7-134">Challenges</span></span>

- <span data-ttu-id="edde7-135">Sem um design cuidadoso, O front-end e o trabalho podem facilmente se tornar componentes grandes e monolíticos difíceis de serem mantidos e atualizados.</span><span class="sxs-lookup"><span data-stu-id="edde7-135">Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.</span></span>
- <span data-ttu-id="edde7-136">Pode haver dependências ocultas se o front-end e o trabalho compartilharem esquemas de dados ou módulos de código.</span><span class="sxs-lookup"><span data-stu-id="edde7-136">There may be hidden dependencies, if the front end and worker share data schemas or code modules.</span></span>

## <a name="best-practices"></a><span data-ttu-id="edde7-137">Práticas recomendadas</span><span class="sxs-lookup"><span data-stu-id="edde7-137">Best practices</span></span>

- <span data-ttu-id="edde7-138">Expor uma API bem projetada para o cliente.</span><span class="sxs-lookup"><span data-stu-id="edde7-138">Expose a well-designed API to the client.</span></span> <span data-ttu-id="edde7-139">Confira [Práticas recomendadas de design da API][api-design].</span><span class="sxs-lookup"><span data-stu-id="edde7-139">See [API design best practices][api-design].</span></span>
- <span data-ttu-id="edde7-140">Dimensionar automaticamente para controlar as alterações na carga.</span><span class="sxs-lookup"><span data-stu-id="edde7-140">Autoscale to handle changes in load.</span></span> <span data-ttu-id="edde7-141">Confira [Práticas recomendadas de dimensionamento automático][autoscaling].</span><span class="sxs-lookup"><span data-stu-id="edde7-141">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="edde7-142">Armazenar dados semiestáticos em cache.</span><span class="sxs-lookup"><span data-stu-id="edde7-142">Cache semi-static data.</span></span> <span data-ttu-id="edde7-143">Confira [Práticas recomendadas de cache][caching].</span><span class="sxs-lookup"><span data-stu-id="edde7-143">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="edde7-144">Usar uma CDN para hospedar o conteúdo estático.</span><span class="sxs-lookup"><span data-stu-id="edde7-144">Use a CDN to host static content.</span></span> <span data-ttu-id="edde7-145">Confira [Práticas recomendadas de CDN][cdn].</span><span class="sxs-lookup"><span data-stu-id="edde7-145">See [CDN best practices][cdn].</span></span>
- <span data-ttu-id="edde7-146">Usar persistência poliglota quando apropriado.</span><span class="sxs-lookup"><span data-stu-id="edde7-146">Use polyglot persistence when appropriate.</span></span> <span data-ttu-id="edde7-147">Confira [Usar o armazenamento de dados recomendado para o trabalho][polyglot].</span><span class="sxs-lookup"><span data-stu-id="edde7-147">See [Use the best data store for the job][polyglot].</span></span>
- <span data-ttu-id="edde7-148">Particionar dados para aprimorar a escalabilidade, reduzir a contenção e otimizar o desempenho.</span><span class="sxs-lookup"><span data-stu-id="edde7-148">Partition data to improve scalability, reduce contention, and optimize performance.</span></span> <span data-ttu-id="edde7-149">Confira [Práticas recomendadas de particionamento de dados][data-partition].</span><span class="sxs-lookup"><span data-stu-id="edde7-149">See [Data partitioning best practices][data-partition].</span></span>

## <a name="web-queue-worker-on-azure-app-service"></a><span data-ttu-id="edde7-150">Trabalho de fila da Web no Serviço de Aplicativo do Azure</span><span class="sxs-lookup"><span data-stu-id="edde7-150">Web-Queue-Worker on Azure App Service</span></span>

<span data-ttu-id="edde7-151">Esta seção descreve uma arquitetura recomendada de trabalho de fila da Web que usa Serviço de Aplicativo do Azure.</span><span class="sxs-lookup"><span data-stu-id="edde7-151">This section describes a recommended Web-Queue-Worker architecture that uses Azure App Service.</span></span>

![Diagrama físico do estilo de arquitetura de trabalho de fila da Web](./images/web-queue-worker-physical.png)

- <span data-ttu-id="edde7-153">O front-end é implementado como um aplicativo web do serviço de aplicativo do Azure e o trabalho é implementado como um aplicativo de funções do Azure.</span><span class="sxs-lookup"><span data-stu-id="edde7-153">The front end is implemented as an Azure App Service web app, and the worker is implemented as an Azure Functions app.</span></span> <span data-ttu-id="edde7-154">O aplicativo web e o aplicativo de funções são associadas com um plano do serviço de aplicativo que fornece as instâncias de VM.</span><span class="sxs-lookup"><span data-stu-id="edde7-154">The web app and the function app are both associated with an App Service plan that provides the VM instances.</span></span>

- <span data-ttu-id="edde7-155">Você pode usar filas do Barramento de Serviço do Microsoft Azure ou do Armazenamento do Microsoft Azure para a fila de mensagens.</span><span class="sxs-lookup"><span data-stu-id="edde7-155">You can use either Azure Service Bus or Azure Storage queues for the message queue.</span></span> <span data-ttu-id="edde7-156">(O diagrama mostra uma fila de Armazenamento do Microsoft Azure.)</span><span class="sxs-lookup"><span data-stu-id="edde7-156">(The diagram shows an Azure Storage queue.)</span></span>

- <span data-ttu-id="edde7-157">O Cache Redis do Azure armazena o estado de sessão e outros dados que precisam de acesso de baixa latência.</span><span class="sxs-lookup"><span data-stu-id="edde7-157">Azure Redis Cache stores session state and other data that needs low latency access.</span></span>

- <span data-ttu-id="edde7-158">A CDN do Azure é usada para armazenar em cache o conteúdo estático, como imagens, CSS ou HTML.</span><span class="sxs-lookup"><span data-stu-id="edde7-158">Azure CDN is used to cache static content such as images, CSS, or HTML.</span></span>

- <span data-ttu-id="edde7-159">Para o armazenamento, escolha as tecnologias de armazenamento que melhor atendem às necessidades do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="edde7-159">For storage, choose the storage technologies that best fit the needs of the application.</span></span> <span data-ttu-id="edde7-160">Você pode usar várias tecnologias de armazenamento (persistência poliglota).</span><span class="sxs-lookup"><span data-stu-id="edde7-160">You might use multiple storage technologies (polyglot persistence).</span></span> <span data-ttu-id="edde7-161">Para ilustrar essa ideia, o diagrama mostra o Banco de Dados SQL do Azure e o Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="edde7-161">To illustrate this idea, the diagram shows Azure SQL Database and Azure Cosmos DB.</span></span>

<span data-ttu-id="edde7-162">Para obter mais detalhes, confira [Arquitetura de referência do aplicativo do Serviço de Aplicativo Web][scalable-web-app].</span><span class="sxs-lookup"><span data-stu-id="edde7-162">For more details, see [App Service web application reference architecture][scalable-web-app].</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="edde7-163">Considerações adicionais</span><span class="sxs-lookup"><span data-stu-id="edde7-163">Additional considerations</span></span>

- <span data-ttu-id="edde7-164">Nem toda transação deve passar pela fila e pelo trabalho para o armazenamento.</span><span class="sxs-lookup"><span data-stu-id="edde7-164">Not every transaction has to go through the queue and worker to storage.</span></span> <span data-ttu-id="edde7-165">O front-end da Web pode executar diretamente operações simples de leitura/gravação.</span><span class="sxs-lookup"><span data-stu-id="edde7-165">The web front end can perform simple read/write operations directly.</span></span> <span data-ttu-id="edde7-166">Os trabalhos são projetados para tarefas de uso intensivo de recursos ou fluxos de trabalho de longa execução.</span><span class="sxs-lookup"><span data-stu-id="edde7-166">Workers are designed for resource-intensive tasks or long-running workflows.</span></span> <span data-ttu-id="edde7-167">Em alguns casos, talvez não seja preciso de nenhum trabalho.</span><span class="sxs-lookup"><span data-stu-id="edde7-167">In some cases, you might not need a worker at all.</span></span>

- <span data-ttu-id="edde7-168">Use o recurso interno de dimensionar automaticamente do Serviço de Aplicativo para escalar horizontalmente o número de instâncias de VM.</span><span class="sxs-lookup"><span data-stu-id="edde7-168">Use the built-in autoscale feature of App Service to scale out the number of VM instances.</span></span> <span data-ttu-id="edde7-169">Se a carga no aplicativo segue padrões previsíveis, use o dimensionamento automático baseado em agendamento.</span><span class="sxs-lookup"><span data-stu-id="edde7-169">If the load on the application follows predictable patterns, use schedule-based autoscale.</span></span> <span data-ttu-id="edde7-170">Se a carga for imprevisível, use regras de dimensionamento automático baseado em métricas.</span><span class="sxs-lookup"><span data-stu-id="edde7-170">If the load is unpredictable, use metrics-based autoscaling rules.</span></span>

- <span data-ttu-id="edde7-171">Considere colocar o aplicativo web e o aplicativo de funções em planos de serviço de aplicativo separados.</span><span class="sxs-lookup"><span data-stu-id="edde7-171">Consider putting the web app and the function app into separate App Service plans.</span></span> <span data-ttu-id="edde7-172">Dessa forma, eles podem ser dimensionados de forma independente.</span><span class="sxs-lookup"><span data-stu-id="edde7-172">That way, they can be scaled independently.</span></span>

- <span data-ttu-id="edde7-173">Use planos de Serviço de Aplicativo separados para produção e teste.</span><span class="sxs-lookup"><span data-stu-id="edde7-173">Use separate App Service plans for production and testing.</span></span> <span data-ttu-id="edde7-174">Caso contrário, se você usar o mesmo plano para produção e teste, isso significa que seus testes estão em execução em suas VMs de produção.</span><span class="sxs-lookup"><span data-stu-id="edde7-174">Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.</span></span>

- <span data-ttu-id="edde7-175">Use slots de implantação para gerenciar implantações.</span><span class="sxs-lookup"><span data-stu-id="edde7-175">Use deployment slots to manage deployments.</span></span> <span data-ttu-id="edde7-176">Isso permite implantar uma versão atualizada em um slot de preparo e depois trocar para a nova versão.</span><span class="sxs-lookup"><span data-stu-id="edde7-176">This lets you to deploy an updated version to a staging slot, then swap over to the new version.</span></span> <span data-ttu-id="edde7-177">Também permite que você troque para a versão anterior, caso tenha havido um problema com a atualização.</span><span class="sxs-lookup"><span data-stu-id="edde7-177">It also lets you swap back to the previous version, if there was a problem with the update.</span></span>

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md