---
title: Pipes e filtros
description: Dividir uma tarefa que executa processamento complexo em uma série de elementos separados que podem ser reutilizados.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- messaging
ms.openlocfilehash: 2c17504f594843c10fcfe221f0087f1087a73fb8
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847100"
---
# <a name="pipes-and-filters-pattern"></a><span data-ttu-id="e6ca5-104">Padrão de pipes e filtros</span><span class="sxs-lookup"><span data-stu-id="e6ca5-104">Pipes and Filters pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="e6ca5-105">Decompor uma tarefa que executa processamento complexo em uma série de elementos separados que podem ser reutilizados.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-105">Decompose a task that performs complex processing into a series of separate elements that can be reused.</span></span> <span data-ttu-id="e6ca5-106">Isso pode melhorar o desempenho, escalabilidade e reutilização, permitindo que os elementos de tarefa que executam o processamento sejam implantados e escalados de forma independente.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-106">This can improve performance, scalability, and reusability by allowing task elements that perform the processing to be deployed and scaled independently.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="e6ca5-107">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="e6ca5-107">Context and problem</span></span>

<span data-ttu-id="e6ca5-108">É necessário um aplicativo para executar uma variedade de tarefas de complexidade variável nas informações que são processadas.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-108">An application is required to perform a variety of tasks of varying complexity on the information that it processes.</span></span> <span data-ttu-id="e6ca5-109">Uma abordagem simples mas inflexível para implementar um aplicativo é realizar esse processamento como um módulo monolítico.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-109">A straightforward but inflexible approach to implementing an application is to perform this processing as a monolithic module.</span></span> <span data-ttu-id="e6ca5-110">No entanto, essa abordagem provavelmente reduzirá as oportunidades para refatorar o código, otimizá-lo ou reutilizá-lo se partes do mesmo processamento forem necessárias em outros lugares no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-110">However, this approach is likely to reduce the opportunities for refactoring the code, optimizing it, or reusing it if parts of the same processing are required elsewhere within the application.</span></span>

<span data-ttu-id="e6ca5-111">A figura ilustra os problemas com os dados de processamento usando a abordagem monolítica.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-111">The figure illustrates the issues with processing data using the monolithic approach.</span></span> <span data-ttu-id="e6ca5-112">Um aplicativo recebe e processa dados de duas fontes.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-112">An application receives and processes data from two sources.</span></span> <span data-ttu-id="e6ca5-113">Os dados de cada fonte são processados por um módulo separado que executa uma série de tarefas para transformar esses dados, antes de passar o resultado para a lógica de negócios do aplicativos.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-113">The data from each source is processed by a separate module that performs a series of tasks to transform this data, before passing the result to the business logic of the application.</span></span>

![Figura 1 - Uma solução implementada utilizando módulos monolíticos](./_images/pipes-and-filters-modules.png)

<span data-ttu-id="e6ca5-115">Algumas das tarefas que os módulos monolíticos executam são funcionalmente muito semelhantes, mas os módulos foram projetados separadamente.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-115">Some of the tasks that the monolithic modules perform are functionally very similar, but the modules have been designed separately.</span></span> <span data-ttu-id="e6ca5-116">O código que implementa as tarefas está intimamente acoplado em um módulo e foi desenvolvido com pouco ou nenhuma atenção dada para reutilização ou escalabilidade.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-116">The code that implements the tasks is closely coupled in a module, and has been developed with little or no thought given to reuse or scalability.</span></span>

<span data-ttu-id="e6ca5-117">No entanto, as tarefas de processamento executadas por cada módulo, ou os requisitos de implantação para cada tarefa, podem mudar conforme os requisitos de negócios são atualizados.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-117">However, the processing tasks performed by each module, or the deployment requirements for each task, could change as business requirements are updated.</span></span> <span data-ttu-id="e6ca5-118">Algumas tarefas podem ser de computação intensiva e podem se beneficiar com a execução de hardware poderoso, enquanto outras talvez não exijam recursos tão dispendiosos.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-118">Some tasks might be compute intensive and could benefit from running on powerful hardware, while others might not require such expensive resources.</span></span> <span data-ttu-id="e6ca5-119">Além disso, o processamento adicional pode ser exigido no futuro, ou a ordem em que as tarefas realizadas pelo processamento podem ser alteradas.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-119">Also, additional processing might be required in the future, or the order in which the tasks performed by the processing could change.</span></span> <span data-ttu-id="e6ca5-120">É necessária uma solução que aborda esses problemas e aumenta as possibilidades de reutilização de código.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-120">A solution is required that addresses these issues, and increases the possibilities for code reuse.</span></span>

## <a name="solution"></a><span data-ttu-id="e6ca5-121">Solução</span><span class="sxs-lookup"><span data-stu-id="e6ca5-121">Solution</span></span>

<span data-ttu-id="e6ca5-122">Dividir o processamento necessário para cada stream em um conjunto de componentes separados (ou filtros), cada um executando uma única tarefa.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-122">Break down the processing required for each stream into a set of separate components (or filters), each performing a single task.</span></span> <span data-ttu-id="e6ca5-123">Ao padronizar o formato dos dados que cada componente recebe e envia, esses filtros podem ser combinados em um pipeline.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-123">By standardizing the format of the data that each component receives and sends, these filters can be combined together into a pipeline.</span></span> <span data-ttu-id="e6ca5-124">Isso ajuda a evitar a duplicação de código e facilita a remoção, substituição ou integração de componentes adicionais, se os requisitos de processamento forem alterados.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-124">This helps to avoid duplicating code, and makes it easy to remove, replace, or integrate additional components if the processing requirements change.</span></span> <span data-ttu-id="e6ca5-125">A próxima figura mostra uma solução implementada usando pipes e filtros.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-125">The next figure shows a solution implemented using pipes and filters.</span></span>

![Figura 2 - Uma solução implementada usando pipes e filtros](./_images/pipes-and-filters-solution.png)


<span data-ttu-id="e6ca5-127">O tempo necessário para processar uma solicitação única depende da velocidade do filtro lento no pipeline.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-127">The time it takes to process a single request depends on the speed of the slowest filter in the pipeline.</span></span> <span data-ttu-id="e6ca5-128">Um ou mais filtros podem ser um gargalo, especialmente se um grande número de solicitações aparecer em um stream de uma determinada fonte de dados.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-128">One or more filters could be a bottleneck, especially if a large number of requests appear in a stream from a particular data source.</span></span> <span data-ttu-id="e6ca5-129">Uma vantagem chave da estrutura de pipeline é que ele oferece oportunidades para executar instâncias paralelas de filtros lentos, permitindo que o sistema espalhe a carga e melhore a taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-129">A key advantage of the pipeline structure is that it provides opportunities for running parallel instances of slow filters, enabling the system to spread the load and improve throughput.</span></span>

<span data-ttu-id="e6ca5-130">Os filtros que compõem um pipeline podem ser executados em diferentes computadores, permitindo que sejam escalados de forma independente e aproveitem a elasticidade que muitos ambientes de nuvem fornecem.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-130">The filters that make up a pipeline can run on different machines, enabling them to be scaled independently and take advantage of the elasticity that many cloud environments provide.</span></span> <span data-ttu-id="e6ca5-131">Um filtro que é computacionalmente intensivo pode ser executado em hardware de alto desempenho, enquanto outros filtros menos exigentes podem ser hospedados em hardware de mercadoria menos caro.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-131">A filter that is computationally intensive can run on high performance hardware, while other less demanding filters can be hosted on less expensive commodity hardware.</span></span> <span data-ttu-id="e6ca5-132">Os filtros não precisam estar no mesmo centro de dados ou localização geográfica, o que permite que cada elemento em um pipeline seja executado em um ambiente próximo dos recursos necessários.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-132">The filters don't even have to be in the same data center or geographical location, which allows each element in a pipeline to run in an environment that is close to the resources it requires.</span></span>  <span data-ttu-id="e6ca5-133">A próxima figura mostra um exemplo aplicado ao pipeline para os dados da Fonte 1.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-133">The next figure shows an example applied to the pipeline for the data from Source 1.</span></span>

![A Figura 3 mostra um exemplo aplicado ao pipeline para os dados da Fonte 1](./_images/pipes-and-filters-load-balancing.png)

<span data-ttu-id="e6ca5-135">Se a entrada e a saída de um filtro estiverem estruturadas como um stream, será possível executar o processamento para cada filtro em paralelo.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-135">If the input and output of a filter are structured as a stream, it's possible to perform the processing for each filter in parallel.</span></span> <span data-ttu-id="e6ca5-136">O primeiro filtro no pipeline pode iniciar seu trabalho e emitir seus resultados, que são passados diretamente para o próximo filtro na sequência antes do primeiro filtro ter concluído seu trabalho.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-136">The first filter in the pipeline can start its work and output its results, which are passed directly on to the next filter in the sequence before the first filter has completed its work.</span></span>

<span data-ttu-id="e6ca5-137">Outro benefício é a resiliência que esse modelo pode fornecer.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-137">Another benefit is the resiliency that this model can provide.</span></span> <span data-ttu-id="e6ca5-138">Se um filtro falhar ou o comutador em que ele estiver executando não estiver mais disponível, o pipeline poderá reagendar o trabalho que o filtro estava executando e direcionar esse trabalho para outra instância do componente.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-138">If a filter fails or the machine it's running on is no longer available, the pipeline can reschedule the work that the filter was performing and direct this work to another instance of the component.</span></span> <span data-ttu-id="e6ca5-139">A falha de um único filtro não resulta necessariamente na falha de todo o pipeline.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-139">Failure of a single filter doesn't necessarily result in failure of the entire pipeline.</span></span>

<span data-ttu-id="e6ca5-140">O uso do padrão de Pipes e Filtros em conjunto com o [padrão de Transação de Compensação](compensating-transaction.md) é uma abordagem alternativa para implementar transações distribuídas.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-140">Using the Pipes and Filters pattern in conjunction with the [Compensating Transaction pattern](compensating-transaction.md) is an alternative approach to implementing distributed transactions.</span></span> <span data-ttu-id="e6ca5-141">Uma transação distribuída pode ser dividida em tarefas compensáveis e separáveis, cada uma das quais pode ser implementada utilizando um filtro que também implementa o padrão de Transação de Compensação.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-141">A distributed transaction can be broken down into separate, compensable tasks, each of which can be implemented by using a filter that also implements the Compensating Transaction pattern.</span></span> <span data-ttu-id="e6ca5-142">Os filtros em um pipeline podem ser implementados como tarefas hospedadas separadas que executam perto dos dados que eles mantêm.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-142">The filters in a pipeline can be implemented as separate hosted tasks running close to the data that they maintain.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="e6ca5-143">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="e6ca5-143">Issues and considerations</span></span>

<span data-ttu-id="e6ca5-144">Os seguintes pontos devem ser considerados ao decidir como implementar esse padrão:</span><span class="sxs-lookup"><span data-stu-id="e6ca5-144">You should consider the following points when deciding how to implement this pattern:</span></span>
- <span data-ttu-id="e6ca5-145">**Complexidade**.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-145">**Complexity**.</span></span> <span data-ttu-id="e6ca5-146">A flexibilidade aumentada que esse padrão fornece também pode introduzir complexidade, especialmente se os filtros em um pipeline estiverem distribuídos em diferentes servidores.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-146">The increased flexibility that this pattern provides can also introduce complexity, especially if the filters in a pipeline are distributed across different servers.</span></span>

- <span data-ttu-id="e6ca5-147">**Confiabilidade**.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-147">**Reliability**.</span></span> <span data-ttu-id="e6ca5-148">Use uma infraestrutura que garanta que os dados que fluem entre filtros em um pipeline não sejam perdidos.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-148">Use an infrastructure that ensures that data flowing between filters in a pipeline won't be lost.</span></span>

- <span data-ttu-id="e6ca5-149">**Idempotência** .</span><span class="sxs-lookup"><span data-stu-id="e6ca5-149">**Idempotency**.</span></span> <span data-ttu-id="e6ca5-150">Se um filtro em um pipeline falhar após receber uma mensagem e o trabalho for reagendado para outra instância do filtro, parte do trabalho talvez já tenha sido concluída.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-150">If a filter in a pipeline fails after receiving a message and the work is rescheduled to another instance of the filter, part of the work might have already been completed.</span></span> <span data-ttu-id="e6ca5-151">Se esse trabalho atualizar algum aspecto do estado global (como informações armazenadas em um banco de dados), a mesma atualização poderá ser repetida.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-151">If this work updates some aspect of the global state (such as information stored in a database), the same update could be repeated.</span></span> <span data-ttu-id="e6ca5-152">Um problema semelhante pode ocorrer se um filtro falhar após a postagem dos resultados para o próximo filtro no pipeline, mas antes de indicar que seu trabalho foi concluído com êxito.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-152">A similar issue might occur if a filter fails after posting its results to the next filter in the pipeline, but before indicating that it's completed its work successfully.</span></span> <span data-ttu-id="e6ca5-153">Nesses casos, o mesmo trabalho pode ser repetido por outra instância do filtro, fazendo com que os mesmos resultados sejam postados duas vezes.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-153">In these cases, the same work could be repeated by another instance of the filter, causing the same results to be posted twice.</span></span> <span data-ttu-id="e6ca5-154">Isso pode resultar em filtros subsequentes no pipeline que processam os mesmos dados duas vezes.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-154">This could result in subsequent filters in the pipeline processing the same data twice.</span></span> <span data-ttu-id="e6ca5-155">Portanto, os filtros em um pipeline devem ser projetados para serem idempotentes.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-155">Therefore filters in a pipeline should be designed to be idempotent.</span></span> <span data-ttu-id="e6ca5-156">Para mais informações, consulte os [Padrões de Idempotência](http://blog.jonathanoliver.com/idempotency-patterns/) no blog de Jonathan Oliver.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-156">For more information see [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>

- <span data-ttu-id="e6ca5-157">**Mensagens repetidas**.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-157">**Repeated messages**.</span></span> <span data-ttu-id="e6ca5-158">Se um filtro em um pipeline falhar após a postagem de uma mensagem para o próximo estágio do pipeline, outra instância do filtro poderá ser executada, e ela postará uma cópia da mesma mensagem para o pipeline.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-158">If a filter in a pipeline fails after posting a message to the next stage of the pipeline, another instance of the filter might be run, and it'll post a copy of the same message to the pipeline.</span></span> <span data-ttu-id="e6ca5-159">Isso pode fazer com que duas instâncias da mesma mensagem sejam passadas para o próximo filtro.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-159">This could cause two instances of the same message to be passed to the next filter.</span></span> <span data-ttu-id="e6ca5-160">Para evitar isso, o pipeline deverá detectar e eliminar mensagens duplicadas.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-160">To avoid this, the pipeline should detect and eliminate duplicate messages.</span></span>

    >  <span data-ttu-id="e6ca5-161">Se você estiver implementando o pipeline utilizando filas de mensagens (como as filas do Barramento de Serviço do Microsoft Azure), a infraestrutura de enfileiramento de mensagens poderá fornecer detecção e remoção automática de mensagens duplicadas.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-161">If you're implementing the pipeline by using message queues (such as Microsoft Azure Service Bus queues), the message queuing infrastructure might provide automatic duplicate message detection and removal.</span></span>

- <span data-ttu-id="e6ca5-162">**Contexto e estado**.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-162">**Context and state**.</span></span> <span data-ttu-id="e6ca5-163">Em um pipeline, cada filtro essencialmente é executado em isolamento e não deve fazer qualquer suposição sobre como foi invocado.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-163">In a pipeline, each filter essentially runs in isolation and shouldn't make any assumptions about how it was invoked.</span></span> <span data-ttu-id="e6ca5-164">Isso significa que cada filtro deve ter um contexto suficiente para realizar seu trabalho.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-164">This means that each filter should be provided with sufficient context to perform its work.</span></span> <span data-ttu-id="e6ca5-165">Esse contexto poderia incluir uma grande quantidade de informações de estado.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-165">This context could include a large amount of state information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="e6ca5-166">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="e6ca5-166">When to use this pattern</span></span>

<span data-ttu-id="e6ca5-167">Use esse padrão quando:</span><span class="sxs-lookup"><span data-stu-id="e6ca5-167">Use this pattern when:</span></span>
- <span data-ttu-id="e6ca5-168">O processamento exigido por um aplicativo pode ser facilmente dividido em um conjunto de etapas independentes.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-168">The processing required by an application can easily be broken down into a set of independent steps.</span></span>

- <span data-ttu-id="e6ca5-169">As etapas de processamento executadas por um aplicativo têm requisitos de escalabilidade diferentes.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-169">The processing steps performed by an application have different scalability requirements.</span></span>

    >  <span data-ttu-id="e6ca5-170">É possível agrupar filtros que devem escalar no mesmo processo.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-170">It's possible to group filters that should scale together in the same process.</span></span> <span data-ttu-id="e6ca5-171">Para obter mais informações, consulte o [padrão de Consolidação de Recursos de Computação](compute-resource-consolidation.md).</span><span class="sxs-lookup"><span data-stu-id="e6ca5-171">For more information, see the [Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span>

- <span data-ttu-id="e6ca5-172">É necessária flexibilidade para permitir a reordenação das etapas de processamento executadas por um aplicativo ou a capacidade de adicionar e remover etapas.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-172">Flexibility is required to allow reordering of the processing steps performed by an application, or the capability to add and remove steps.</span></span>

- <span data-ttu-id="e6ca5-173">O sistema pode se beneficiar de distribuir o processamento para etapas em diferentes servidores.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-173">The system can benefit from distributing the processing for steps across different servers.</span></span>

- <span data-ttu-id="e6ca5-174">É necessária uma solução confiável que minimize os efeitos da falha em uma etapa, enquanto os dados estão sendo processados.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-174">A reliable solution is required that minimizes the effects of failure in a step while data is being processed.</span></span>

<span data-ttu-id="e6ca5-175">Esse padrão pode não ser útil quando:</span><span class="sxs-lookup"><span data-stu-id="e6ca5-175">This pattern might not be useful when:</span></span>
- <span data-ttu-id="e6ca5-176">As etapas de processamento executadas por um aplicativo não são independentes ou devem ser realizadas juntas como parte da mesma transação.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-176">The processing steps performed by an application aren't independent, or they have to be performed together as part of the same transaction.</span></span>

- <span data-ttu-id="e6ca5-177">A quantidade de contexto ou informações de estado exigidas por uma etapa torna esta abordagem ineficiente.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-177">The amount of context or state information required by a step makes this approach inefficient.</span></span> <span data-ttu-id="e6ca5-178">Pode ser possível persistir informações de estado em um banco de dados, mas não utilize essa estratégia se a carga adicional no banco de dados causar contenção excessiva.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-178">It might be possible to persist state information to a database instead, but don't use this strategy if the additional load on the database causes excessive contention.</span></span>

## <a name="example"></a><span data-ttu-id="e6ca5-179">Exemplo</span><span class="sxs-lookup"><span data-stu-id="e6ca5-179">Example</span></span>

<span data-ttu-id="e6ca5-180">Você pode utilizar uma sequência de filas de mensagens para fornecer a infraestrutura necessária para implementar um pipeline.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-180">You can use a sequence of message queues to provide the infrastructure required to implement a pipeline.</span></span> <span data-ttu-id="e6ca5-181">Uma fila de mensagens inicial recebe mensagens não processadas.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-181">An initial message queue receives unprocessed messages.</span></span> <span data-ttu-id="e6ca5-182">Um componente implementado como uma tarefa de filtro escuta uma mensagem nessa fila, executa seu trabalho e, em seguida, posta a mensagem transformada para a próxima fila na sequência.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-182">A component implemented as a filter task listens for a message on this queue, performs its work, and then posts the transformed message to the next queue in the sequence.</span></span> <span data-ttu-id="e6ca5-183">Outra tarefa de filtro pode escutar mensagens nessa fila, processá-las, postar os resultados em outra fila e, assim por diante, até que os dados totalmente transformados apareçam na mensagem final na fila.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-183">Another filter task can listen for messages on this queue, process them, post the results to another queue, and so on until the fully transformed data appears in the final message in the queue.</span></span> <span data-ttu-id="e6ca5-184">A próxima figura ilustra a implementação de um pipeline utilizando filas de mensagens.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-184">The next figure illustrates implementing a pipeline using message queues.</span></span>

![Figura 4 - Implementando um pipeline utilizando filas de mensagens](./_images/pipes-and-filters-message-queues.png)


<span data-ttu-id="e6ca5-186">Se estiver compilando uma solução no Azure, você poderá utilizar as filas do Barramento de Serviço para fornecer um mecanismo de enfileiramento confiável e escalonável.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-186">If you're building a solution on Azure you can use Service Bus queues to provide a reliable and scalable queuing mechanism.</span></span> <span data-ttu-id="e6ca5-187">A classe `ServiceBusPipeFilter` mostrada abaixo em C# demonstra como é possível implementar um filtro que recebe mensagens de entrada de uma fila, processa essas mensagens e posta os resultados em outra fila.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-187">The `ServiceBusPipeFilter` class shown below in C# demonstrates how you can implement a filter that receives input messages from a queue, processes these messages, and posts the results to another queue.</span></span>

>  <span data-ttu-id="e6ca5-188">A classe `ServiceBusPipeFilter` está definida no projeto PipesAndFilters.Shared disponível a partir do [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span><span class="sxs-lookup"><span data-stu-id="e6ca5-188">The `ServiceBusPipeFilter` class is defined in the PipesAndFilters.Shared project available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span></span>

```csharp
public class ServiceBusPipeFilter
{
  ...
  private readonly string inQueuePath;
  private readonly string outQueuePath;
  ...
  private QueueClient inQueue;
  private QueueClient outQueue;
  ...

  public ServiceBusPipeFilter(..., string inQueuePath, string outQueuePath = null)
  {
     ...
     this.inQueuePath = inQueuePath;
     this.outQueuePath = outQueuePath;
  }

  public void Start()
  {
    ...
    // Create the outbound filter queue if it doesn't exist.
    ...
    this.outQueue = QueueClient.CreateFromConnectionString(...);

    ...
    // Create the inbound and outbound queue clients.
    this.inQueue = QueueClient.CreateFromConnectionString(...);
  }

  public void OnPipeFilterMessageAsync(
    Func<BrokeredMessage, Task<BrokeredMessage>> asyncFilterTask, ...)
  {
    ...

    this.inQueue.OnMessageAsync(
      async (msg) =>
    {
      ...
      // Process the filter and send the output to the
      // next queue in the pipeline.
      var outMessage = await asyncFilterTask(msg);

      // Send the message from the filter processor
      // to the next queue in the pipeline.
      if (outQueue != null)
      {
        await outQueue.SendAsync(outMessage);
      }

      // Note: There's a chance that the same message could be sent twice
      // or that a message gets processed by an upstream or downstream
      // filter at the same time.
      // This would happen in a situation where processing of a message was
      // completed, it was sent to the next pipe/queue, and then failed
      // to complete when using the PeekLock method.
      // Idempotent message processing and concurrency should be considered
      // in a real-world implementation.
    },
    options);
  }

  public async Task Close(TimeSpan timespan)
  {
    // Pause the processing threads.
    this.pauseProcessingEvent.Reset();

    // There's no clean approach for waiting for the threads to complete
    // the processing. This example simply stops any new processing, waits
    // for the existing thread to complete, then closes the message pump
    // and finally returns.
    Thread.Sleep(timespan);

    this.inQueue.Close();
    ...
  }

  ...
}
```

<span data-ttu-id="e6ca5-189">O método `Start` na classe `ServiceBusPipeFilter` se conecta a um par de filas de entrada e saída e o método `Close` desconecta da fila de entrada.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-189">The `Start` method in the `ServiceBusPipeFilter` class connects to a pair of input and output queues, and the `Close` method disconnects from the input queue.</span></span> <span data-ttu-id="e6ca5-190">O método `OnPipeFilterMessageAsync` executa o processamento real das mensagens, o parâmetro `asyncFilterTask` para esse método especifica o processamento a ser executado.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-190">The `OnPipeFilterMessageAsync` method performs the actual processing of messages, the `asyncFilterTask` parameter to this method specifies the processing to be performed.</span></span> <span data-ttu-id="e6ca5-191">O método `OnPipeFilterMessageAsync` aguarda as mensagens de entrada na fila de entrada, executa o código especificado pelo parâmetro `asyncFilterTask` sobre cada mensagem à medida que chega e posta os resultados na fila de saída.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-191">The `OnPipeFilterMessageAsync` method waits for incoming messages on the input queue, runs the code specified by the `asyncFilterTask` parameter over each message as it arrives, and posts the results to the output queue.</span></span> <span data-ttu-id="e6ca5-192">As próprias filas são especificadas pelo construtor.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-192">The queues themselves are specified by the constructor.</span></span>

<span data-ttu-id="e6ca5-193">A solução de exemplo implementa filtros em um conjunto de funções de trabalho.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-193">The sample solution implements filters in a set of worker roles.</span></span> <span data-ttu-id="e6ca5-194">Cada função de trabalhador pode ser escalada de forma independente, dependendo da complexidade do processamento de negócios que executa ou dos recursos necessários para o processamento.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-194">Each worker role can be scaled independently, depending on the complexity of the business processing that it performs or the resources required for processing.</span></span> <span data-ttu-id="e6ca5-195">Além disso, várias instâncias de cada função de trabalho podem ser executadas em paralelo para melhorar a taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-195">Additionally, multiple instances of each worker role can be run in parallel to improve throughput.</span></span>

<span data-ttu-id="e6ca5-196">O código a seguir mostra uma função de trabalho do Azure chamada `PipeFilterARoleEntry`, definida no projeto PipeFilterA na solução de exemplo.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-196">The following code shows an Azure worker role named `PipeFilterARoleEntry`, defined in the PipeFilterA project in the sample solution.</span></span>

```csharp
public class PipeFilterARoleEntry : RoleEntryPoint
{
  ...
  private ServiceBusPipeFilter pipeFilterA;

  public override bool OnStart()
  {
    ...
    this.pipeFilterA = new ServiceBusPipeFilter(
      ...,
      Constants.QueueAPath,
      Constants.QueueBPath);

    this.pipeFilterA.Start();
    ...
  }

  public override void Run()
  {
    this.pipeFilterA.OnPipeFilterMessageAsync(async (msg) =>
    {
      // Clone the message and update it.
      // Properties set by the broker (Deliver count, enqueue time, ...)
      // aren't cloned and must be copied over if required.
      var newMsg = msg.Clone();

      await Task.Delay(500); // DOING WORK

      Trace.TraceInformation("Filter A processed message:{0} at {1}",
        msg.MessageId, DateTime.UtcNow);

      newMsg.Properties.Add(Constants.FilterAMessageKey, "Complete");

      return newMsg;
    });

    ...
  }

  ...
}
```

<span data-ttu-id="e6ca5-197">Esta função contém um objeto `ServiceBusPipeFilter`.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-197">This role contains a `ServiceBusPipeFilter` object.</span></span> <span data-ttu-id="e6ca5-198">O método `OnStart` na função se conecta às filas para receber mensagens de entrada e postar mensagens de saída (os nomes das filas são definidos na classe `Constants`).</span><span class="sxs-lookup"><span data-stu-id="e6ca5-198">The `OnStart` method in the role connects to the queues for receiving input messages and posting output messages (the names of the queues are defined in the `Constants` class).</span></span> <span data-ttu-id="e6ca5-199">O método `Run` invoca o método `OnPipeFilterMessagesAsync` para executar algum processamento em cada mensagem recebida (neste exemplo, o processamento é simulado esperando por um curto período de tempo).</span><span class="sxs-lookup"><span data-stu-id="e6ca5-199">The `Run` method invokes the `OnPipeFilterMessagesAsync` method to perform some processing on each message that's received (in this example, the processing is simulated by waiting for a short period of time).</span></span> <span data-ttu-id="e6ca5-200">Quando o processamento está completo, uma nova mensagem é construída contendo os resultados (nesse caso, a mensagem de entrada tem uma propriedade personalizada adicionada) e essa mensagem é postada na fila de saída.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-200">When processing is complete, a new message is constructed containing the results (in this case, the input message has a custom property added), and this message is posted to the output queue.</span></span>

<span data-ttu-id="e6ca5-201">O código de exemplo contém outra função de trabalho chamada `PipeFilterBRoleEntry` no projeto PipeFilterB.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-201">The sample code contains another worker role named `PipeFilterBRoleEntry` in the PipeFilterB project.</span></span> <span data-ttu-id="e6ca5-202">Essa função é semelhante a `PipeFilterARoleEntry`, exceto que executa processamento diferente no método `Run`.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-202">This role is similar to `PipeFilterARoleEntry` except that it performs different processing in the `Run` method.</span></span> <span data-ttu-id="e6ca5-203">Na solução de exemplo, essas duas funções são combinadas para construir um pipeline, a fila de saída para a função `PipeFilterARoleEntry` é a fila de entrada para a função `PipeFilterBRoleEntry`.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-203">In the example solution, these two roles are combined to construct a pipeline, the output queue for the `PipeFilterARoleEntry` role is the input queue for the `PipeFilterBRoleEntry` role.</span></span>

<span data-ttu-id="e6ca5-204">A solução de exemplo também fornece dois papéis adicionais denominados `InitialSenderRoleEntry` (no projeto InitialSender) e `FinalReceiverRoleEntry` (no projeto FinalReceiver).</span><span class="sxs-lookup"><span data-stu-id="e6ca5-204">The sample solution also provides two additional roles named `InitialSenderRoleEntry` (in the InitialSender project) and `FinalReceiverRoleEntry` (in the FinalReceiver project).</span></span> <span data-ttu-id="e6ca5-205">A função `InitialSenderRoleEntry` fornece a mensagem inicial no pipeline.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-205">The `InitialSenderRoleEntry` role provides the initial message in the pipeline.</span></span> <span data-ttu-id="e6ca5-206">O método `OnStart` se conecta a uma única fila e o método `Run` posta um método para esta fila.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-206">The `OnStart` method connects to a single queue and the `Run` method posts a method to this queue.</span></span> <span data-ttu-id="e6ca5-207">Essa fila é a fila de entrada utilizada pela função `PipeFilterARoleEntry`, então, para postar uma mensagem a ela faz com que a mensagem seja recebida e processada pela função `PipeFilterARoleEntry`.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-207">This queue is the input queue used by the `PipeFilterARoleEntry` role, so posting a message to it causes the message to be received and processed by the `PipeFilterARoleEntry` role.</span></span> <span data-ttu-id="e6ca5-208">A mensagem processada passa então pela função `PipeFilterBRoleEntry`.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-208">The processed message then passes through the `PipeFilterBRoleEntry` role.</span></span>

<span data-ttu-id="e6ca5-209">A fila de entrada para a função `FinalReceiveRoleEntry` é a fila de saída para a função `PipeFilterBRoleEntry`.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-209">The input queue for the `FinalReceiveRoleEntry` role is the output queue for the `PipeFilterBRoleEntry` role.</span></span> <span data-ttu-id="e6ca5-210">O `Run` método na função `FinalReceiveRoleEntry` mostrado abaixo, recebe a mensagem e executa algum processamento final.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-210">The `Run` method in the `FinalReceiveRoleEntry` role, shown below, receives the message and performs some final processing.</span></span> <span data-ttu-id="e6ca5-211">Em seguida, ele grava os valores das propriedades personalizadas adicionadas pelos filtros no pipeline ao resultado de rastreamento.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-211">Then it writes the values of the custom properties added by the filters in the pipeline to the trace output.</span></span>

```csharp
public class FinalReceiverRoleEntry : RoleEntryPoint
{
  ...
  // Final queue/pipe in the pipeline to process data from.
  private ServiceBusPipeFilter queueFinal;

  public override bool OnStart()
  {
    ...
    // Set up the queue.
    this.queueFinal = new ServiceBusPipeFilter(...,Constants.QueueFinalPath);
    this.queueFinal.Start();
    ...
  }

  public override void Run()
  {
    this.queueFinal.OnPipeFilterMessageAsync(
      async (msg) =>
      {
        await Task.Delay(500); // DOING WORK

        // The pipeline message was received.
        Trace.TraceInformation(
          "Pipeline Message Complete - FilterA:{0} FilterB:{1}",
          msg.Properties[Constants.FilterAMessageKey],
          msg.Properties[Constants.FilterBMessageKey]);

        return null;
      });
    ...
  }

  ...
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="e6ca5-212">Diretrizes e padrões relacionados</span><span class="sxs-lookup"><span data-stu-id="e6ca5-212">Related patterns and guidance</span></span>

<span data-ttu-id="e6ca5-213">Os padrões e diretrizes a seguir também podem ser relevantes ao implementar esse padrão:</span><span class="sxs-lookup"><span data-stu-id="e6ca5-213">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>
- <span data-ttu-id="e6ca5-214">Um exemplo que demonstra esse padrão está disponível em [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span><span class="sxs-lookup"><span data-stu-id="e6ca5-214">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span></span>
- <span data-ttu-id="e6ca5-215">[Padrão de consumidores concorrentes](competing-consumers.md).</span><span class="sxs-lookup"><span data-stu-id="e6ca5-215">[Competing Consumers pattern](competing-consumers.md).</span></span> <span data-ttu-id="e6ca5-216">Um pipeline pode conter várias instâncias de um ou mais filtros.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-216">A pipeline can contain multiple instances of one or more filters.</span></span> <span data-ttu-id="e6ca5-217">Essa abordagem é útil para executar instâncias paralelas de filtros lentos, permitindo que o sistema espalhe a carga e melhore a taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-217">This approach is useful for running parallel instances of slow filters, enabling the system to spread the load and improve throughput.</span></span> <span data-ttu-id="e6ca5-218">Cada instância de um filtro competirá por entrada com as outras instâncias, duas instâncias de um filtro não poderão processar os mesmos dados.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-218">Each instance of a filter will compete for input with the other instances, two instances of a filter shouldn't be able to process the same data.</span></span> <span data-ttu-id="e6ca5-219">Fornece uma explicação sobre essa abordagem.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-219">Provides an explanation of this approach.</span></span>
- <span data-ttu-id="e6ca5-220">[Padrão de consolidação de recursos de computação](compute-resource-consolidation.md).</span><span class="sxs-lookup"><span data-stu-id="e6ca5-220">[Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span> <span data-ttu-id="e6ca5-221">Pode ser possível agrupar filtros que devem ser escalados em conjunto no mesmo processo.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-221">It might be possible to group filters that should scale together into the same process.</span></span> <span data-ttu-id="e6ca5-222">Fornece mais informações sobre os benefícios e compensações dessa estratégia.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-222">Provides more information about the benefits and tradeoffs of this strategy.</span></span>
- <span data-ttu-id="e6ca5-223">[Padrão de Transação de Compensação](compensating-transaction.md).</span><span class="sxs-lookup"><span data-stu-id="e6ca5-223">[Compensating Transaction pattern](compensating-transaction.md).</span></span> <span data-ttu-id="e6ca5-224">Um filtro pode ser implementado como uma operação que pode ser revertida, ou que possui uma operação de compensação que restaura o estado a uma versão anterior em caso de falha.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-224">A filter can be implemented as an operation that can be reversed, or that has a compensating operation that restores the state to a previous version in the event of a failure.</span></span> <span data-ttu-id="e6ca5-225">Explica como isso pode ser implementado para manter ou obter uma consistência eventual.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-225">Explains how this can be implemented to maintain or achieve eventual consistency.</span></span>
- <span data-ttu-id="e6ca5-226">[Padrões de Idempotência](http://blog.jonathanoliver.com/idempotency-patterns/) no blog de Jonathan Oliver.</span><span class="sxs-lookup"><span data-stu-id="e6ca5-226">[Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>
