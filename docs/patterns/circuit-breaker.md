---
title: Disjuntor
description: Trate as falhas que possam consumir uma quantidade variável de tempo para serem corrigidas ao se conectar a um serviço ou recurso remoto.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- resiliency
ms.openlocfilehash: 5a9c8254bf62488b46517ee3582c2323e206df8a
ms.sourcegitcommit: e9d9e214529edd0dc78df5bda29615b8fafd0e56
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/28/2018
ms.locfileid: "37090944"
---
# <a name="circuit-breaker-pattern"></a><span data-ttu-id="ea9ab-104">Padrão de Disjuntor</span><span class="sxs-lookup"><span data-stu-id="ea9ab-104">Circuit Breaker pattern</span></span>

<span data-ttu-id="ea9ab-105">Trate as falhas que possam consumir uma quantidade variável de tempo para serem recuperadas ao se conectar a um serviço ou recurso remoto.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-105">Handle faults that might take a variable amount of time to recover from, when connecting to a remote service or resource.</span></span> <span data-ttu-id="ea9ab-106">Isso pode melhorar a estabilidade e a resiliência de um aplicativo.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-106">This can improve the stability and resiliency of an application.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="ea9ab-107">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="ea9ab-107">Context and problem</span></span>

<span data-ttu-id="ea9ab-108">Em um ambiente distribuído, as chamadas a serviços e recursos remotos podem falhar devido a falhas transitórias, como conexões de rede lentas, tempos limite esgotados ou recursos excessivamente confirmados ou temporariamente não disponíveis.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-108">In a distributed environment, calls to remote resources and services can fail due to transient faults, such as slow network connections, timeouts, or the resources being overcommitted or temporarily unavailable.</span></span> <span data-ttu-id="ea9ab-109">Essas falhas geralmente são corrigidas automaticamente após um breve período de tempo e um aplicativo de nuvem robusto deve estar preparado para tratá-los usando uma estratégia, como o [Padrão de repetição][retry-pattern].</span><span class="sxs-lookup"><span data-stu-id="ea9ab-109">These faults typically correct themselves after a short period of time, and a robust cloud application should be prepared to handle them by using a strategy such as the [Retry pattern][retry-pattern].</span></span>

<span data-ttu-id="ea9ab-110">No entanto, também pode haver situações em que as falhas ocorrem devido a eventos inesperados e que podem demorar muito mais para serem corrigidos.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-110">However, there can also be situations where faults are due to unanticipated events, and that might take much longer to fix.</span></span> <span data-ttu-id="ea9ab-111">Essas falhas podem variar de gravidade de uma perda parcial de conectividade até a falha completa de um serviço.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-111">These faults can range in severity from a partial loss of connectivity to the complete failure of a service.</span></span> <span data-ttu-id="ea9ab-112">Nessas situações, pode ser inútil para um aplicativo tentar repetir continuamente uma operação em que o êxito seja improvável. Em vez disso, o aplicativo deve aceitar rapidamente a falha na operação e tratá-la adequadamente.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-112">In these situations it might be pointless for an application to continually retry an operation that is unlikely to succeed, and instead the application should quickly accept that the operation has failed and handle this failure accordingly.</span></span>

<span data-ttu-id="ea9ab-113">Além disso, se um serviço estiver muito ocupado, a falha em uma parte do sistema poderá resultar em falhas em cascata.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-113">Additionally, if a service is very busy, failure in one part of the system might lead to cascading failures.</span></span> <span data-ttu-id="ea9ab-114">Por exemplo, uma operação que invoca um serviço pode ser configurada para implementar um tempo limite e responder com uma mensagem de falha se o serviço não responder nesse período.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-114">For example, an operation that invokes a service could be configured to implement a timeout, and reply with a failure message if the service fails to respond within this period.</span></span> <span data-ttu-id="ea9ab-115">No entanto, essa estratégia pode causar muitas solicitações simultâneas para a mesma operação a ser bloqueada até a expiração do tempo limite.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-115">However, this strategy could cause many concurrent requests to the same operation to be blocked until the timeout period expires.</span></span> <span data-ttu-id="ea9ab-116">Essas solicitações bloqueadas podem conter recursos críticos do sistema, como memória, threads, conexões de banco de dados, etc.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-116">These blocked requests might hold critical system resources such as memory, threads, database connections, and so on.</span></span> <span data-ttu-id="ea9ab-117">Consequentemente, esses recursos podem se esgotar, causando falha em outras partes possivelmente não relacionadas do sistema que precisam usar os mesmos recursos.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-117">Consequently, these resources could become exhausted, causing failure of other possibly unrelated parts of the system that need to use the same resources.</span></span> <span data-ttu-id="ea9ab-118">Nessas situações, seria preferível que a operação falhasse imediatamente e somente tentasse invocar o serviço se o êxito fosse provável.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-118">In these situations, it would be preferable for the operation to fail immediately, and only attempt to invoke the service if it's likely to succeed.</span></span> <span data-ttu-id="ea9ab-119">Observe que a configuração de um tempo limite menor talvez ajude a resolver esse problema, mas o tempo limite não deve ser tão curto de modo a causar a falha da operação na maioria das vezes, mesmo se a solicitação ao serviço fosse bem-sucedida no final das contas.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-119">Note that setting a shorter timeout might help to resolve this problem, but the timeout shouldn't be so short that the operation fails most of the time, even if the request to the service would eventually succeed.</span></span>

## <a name="solution"></a><span data-ttu-id="ea9ab-120">Solução</span><span class="sxs-lookup"><span data-stu-id="ea9ab-120">Solution</span></span>

<span data-ttu-id="ea9ab-121">O padrão de Disjuntor popularizado por Michael Nygard em seu livro [Release It!](https://pragprog.com/book/mnee/release-it) (Libere!), pode impedir que um aplicativo tente executar repetidamente uma operação em que a falha é provável.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-121">The Circuit Breaker pattern, popularized by Michael Nygard in his book, [Release It!](https://pragprog.com/book/mnee/release-it), can prevent an application from repeatedly trying to execute an operation that's likely to fail.</span></span> <span data-ttu-id="ea9ab-122">Ele permite continuar sem aguardar que a falha seja corrigida ou sem desperdiçar ciclos de CPU enquanto determina-se que a falha é de longa duração.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-122">Allowing it to continue without waiting for the fault to be fixed or wasting CPU cycles while it determines that the fault is long lasting.</span></span> <span data-ttu-id="ea9ab-123">O padrão de Disjuntor também permite que um aplicativo detecte se a falha foi resolvida.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-123">The Circuit Breaker pattern also enables an application to detect whether the fault has been resolved.</span></span> <span data-ttu-id="ea9ab-124">Se o problema aparentar ter sido corrigido, o aplicativo poderá tentar invocar a operação.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-124">If the problem appears to have been fixed, the application can try to invoke the operation.</span></span>

> <span data-ttu-id="ea9ab-125">A finalidade do padrão de Disjuntor é diferente do padrão de Repetição.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-125">The purpose of the Circuit Breaker pattern is different than the Retry pattern.</span></span> <span data-ttu-id="ea9ab-126">O padrão de Repetição permite que um aplicativo tente novamente uma operação na expectativa que haverá êxito.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-126">The Retry pattern enables an application to retry an operation in the expectation that it'll succeed.</span></span> <span data-ttu-id="ea9ab-127">O padrão de Disjuntor impede que um aplicativo tente execute uma operação em que a falha é provável.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-127">The Circuit Breaker pattern prevents an application from performing an operation that is likely to fail.</span></span> <span data-ttu-id="ea9ab-128">Um aplicativo pode combinar esses dois padrões usando o padrão de Repetição para invocar uma operação por meio de um disjuntor.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-128">An application can combine these two patterns by using the Retry pattern to invoke an operation through a circuit breaker.</span></span> <span data-ttu-id="ea9ab-129">No entanto, a lógica de repetição deverá ser sensível às exceções retornadas pelo disjuntor e abandonar as novas tentativas se o disjuntor indicar que uma falha não é transitória.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-129">However, the retry logic should be sensitive to any exceptions returned by the circuit breaker and abandon retry attempts if the circuit breaker indicates that a fault is not transient.</span></span>

<span data-ttu-id="ea9ab-130">Um disjuntor atua como um proxy para operações que podem falhar.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-130">A circuit breaker acts as a proxy for operations that might fail.</span></span> <span data-ttu-id="ea9ab-131">O proxy deve monitorar o número de falhas recentes ocorridas e usar essas informações para decidir se deve permitir que a operação continue ou apenas retornar uma exceção imediatamente.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-131">The proxy should monitor the number of recent failures that have occurred, and use this information to decide whether to allow the operation to proceed, or simply return an exception immediately.</span></span>

<span data-ttu-id="ea9ab-132">O proxy pode ser implementado como um computador de estado com os seguintes estados que imitam a funcionalidade de um disjuntor elétrico:</span><span class="sxs-lookup"><span data-stu-id="ea9ab-132">The proxy can be implemented as a state machine with the following states that mimic the functionality of an electrical circuit breaker:</span></span>

- <span data-ttu-id="ea9ab-133">**Fechado**: a solicitação do aplicativo é encaminhada para a operação.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-133">**Closed**: The request from the application is routed to the operation.</span></span> <span data-ttu-id="ea9ab-134">O proxy mantém a contagem do número de falhas recentes e, se a chamada à operação não for bem-sucedida, o proxy incrementará essa contagem.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-134">The proxy maintains a count of the number of recent failures, and if the call to the operation is unsuccessful the proxy increments this count.</span></span> <span data-ttu-id="ea9ab-135">Se o número de falhas recentes exceder um limite especificado em um determinado período de tempo, o proxy será colocado no estado **Aberto**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-135">If the number of recent failures exceeds a specified threshold within a given time period, the proxy is placed into the **Open** state.</span></span> <span data-ttu-id="ea9ab-136">Nesse momento, o proxy inicia um temporizador de tempo limite e, quando o temporizador expira, o proxy é colocado no estado **Entreaberto**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-136">At this point the proxy starts a timeout timer, and when this timer expires the proxy is placed into the **Half-Open** state.</span></span>

    > <span data-ttu-id="ea9ab-137">A finalidade do temporizador de tempo limite é dar tempo ao sistema para corrigir o problema que causou a falha antes de permitir que o aplicativo tente executar a operação novamente.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-137">The purpose of the timeout timer is to give the system time to fix the problem that caused the failure before allowing the application to try to perform the operation again.</span></span>

- <span data-ttu-id="ea9ab-138">**Aberto**: a solicitação do aplicativo falha imediatamente e uma exceção é retornada ao aplicativo.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-138">**Open**: The request from the application fails immediately and an exception is returned to the application.</span></span>

- <span data-ttu-id="ea9ab-139">**Entreaberto**: um número limitado de solicitações do aplicativo pode ser passado e invocar a operação.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-139">**Half-Open**: A limited number of requests from the application are allowed to pass through and invoke the operation.</span></span> <span data-ttu-id="ea9ab-140">Se essas solicitações forem bem-sucedidas, será considerado que o problema anterior que estava causando a falha foi corrigido e o disjuntor mudará para o estado **Fechado** (o contador de falhas será reiniciado).</span><span class="sxs-lookup"><span data-stu-id="ea9ab-140">If these requests are successful, it's assumed that the fault that was previously causing the failure has been fixed and the circuit breaker switches to the **Closed** state (the failure counter is reset).</span></span> <span data-ttu-id="ea9ab-141">Se alguma solicitação falhar, o disjuntor considerará que a falha ainda está presente, portanto voltará para o estado **Aberto** e reiniciará o temporizador de tempo limite para dar mais tempo ao sistema para se recuperar da falha.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-141">If any request fails, the circuit breaker assumes that the fault is still present so it reverts back to the **Open** state and restarts the timeout timer to give the system a further period of time to recover from the failure.</span></span>

    > <span data-ttu-id="ea9ab-142">O estado **Entreaberto** é útil para impedir que um serviço de recuperação seja inundado repentinamente com solicitações.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-142">The **Half-Open** state is useful to prevent a recovering service from suddenly being flooded with requests.</span></span> <span data-ttu-id="ea9ab-143">Conforme um serviço é recuperado, ele pode dar suporte a um volume limitado de solicitações até que a recuperação seja concluída, mas enquanto a recuperação estiver em andamento, um fluxo grande de trabalho poderá fazer com que o tempo limite do serviço seja atingido ou falhe novamente.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-143">As a service recovers, it might be able to support a limited volume of requests until the recovery is complete, but while recovery is in progress a flood of work can cause the service to time out or fail again.</span></span>

![Estados do Disjuntor](./_images/circuit-breaker-diagram.png)

<span data-ttu-id="ea9ab-145">Na figura, o contador de falhas é usado no estado **Fechado** é baseado em tempo.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-145">In the figure, the failure counter used by the **Closed** state is time based.</span></span> <span data-ttu-id="ea9ab-146">Ele é reiniciado automaticamente em intervalos periódicos.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-146">It's automatically reset at periodic intervals.</span></span> <span data-ttu-id="ea9ab-147">Isso ajuda a impedir que o disjuntor entre no estado **Aberto** em caso de falhas ocasionais.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-147">This helps to prevent the circuit breaker from entering the **Open** state if it experiences occasional failures.</span></span> <span data-ttu-id="ea9ab-148">O limite de falhas que aciona o disjuntor no estado **Aberto** só é atingido quando um número especificado de falhas ocorrer durante um intervalo especificado.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-148">The failure threshold that trips the circuit breaker into the **Open** state is only reached when a specified number of failures have occurred during a specified interval.</span></span> <span data-ttu-id="ea9ab-149">O contador usado no estado **Entreaberto** registra o número de tentativas bem-sucedidas para invocar a operação.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-149">The counter used by the **Half-Open** state records the number of successful attempts to invoke the operation.</span></span> <span data-ttu-id="ea9ab-150">O disjuntor será revertido para o estado **Fechado** após um número especificado de invocações bem-sucedidas da operação ser obtido de maneira consecutiva.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-150">The circuit breaker reverts to the **Closed** state after a specified number of consecutive operation invocations have been successful.</span></span> <span data-ttu-id="ea9ab-151">Se alguma invocação falhar, o disjuntor entrará no estado **Aberto** imediatamente e o contador de êxito será reiniciado na próxima vez que ele entrar no estado **Entreaberto**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-151">If any invocation fails, the circuit breaker enters the **Open** state immediately and the success counter will be reset the next time it enters the **Half-Open** state.</span></span>

> <span data-ttu-id="ea9ab-152">O modo que o sistema é recuperado é tratado de maneira externa, possivelmente restaurando ou reiniciando um componente com falha ou reparando uma conexão de rede.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-152">How the system recovers is handled externally, possibly by restoring or restarting a failed component or repairing a network connection.</span></span>

<span data-ttu-id="ea9ab-153">O padrão de Disjuntor fornece estabilidade enquanto o sistema recupera-se de uma falha e minimiza o impacto no desempenho.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-153">The Circuit Breaker pattern provides stability while the system recovers from a failure and minimizes the impact on performance.</span></span> <span data-ttu-id="ea9ab-154">Ele pode ajudar a manter o tempo de resposta do sistema rejeitando rapidamente uma solicitação para uma operação em que a falha é provável, em vez de aguardar o tempo limite da operação ou nunca retornar.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-154">It can help to maintain the response time of the system by quickly rejecting a request for an operation that's likely to fail, rather than waiting for the operation to time out, or never return.</span></span> <span data-ttu-id="ea9ab-155">Se o disjuntor acionar um evento sempre que mudar de estado, essas informações poderão ser usadas para monitorar a integridade da parte do sistema protegido pelo disjuntor ou para alertar o administrador quando um disjuntor acionar o estado **Aberto**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-155">If the circuit breaker raises an event each time it changes state, this information can be used to monitor the health of the part of the system protected by the circuit breaker, or to alert an administrator when a circuit breaker trips to the **Open** state.</span></span>

<span data-ttu-id="ea9ab-156">O padrão é personalizável e pode ser adaptado de acordo com o tipo de falha possível.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-156">The pattern is customizable and can be adapted according to the type of the possible failure.</span></span> <span data-ttu-id="ea9ab-157">Por exemplo, você pode aplicar um temporizador de tempo limite crescente para um disjuntor.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-157">For example, you can apply an increasing timeout timer to a circuit breaker.</span></span> <span data-ttu-id="ea9ab-158">Você pode colocar o disjuntor no estado **Aberto** inicialmente por alguns segundos e, se a falha não for resolvida, aumentar o tempo limite para alguns minutos e assim por diante.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-158">You could place the circuit breaker in the **Open** state for a few seconds initially, and then if the failure hasn't been resolved increase the timeout to a few minutes, and so on.</span></span> <span data-ttu-id="ea9ab-159">Em alguns casos, em vez de o estado **Aberto** retornar a falha e gerar uma exceção, pode ser útil retornar um valor padrão que seja significativo para o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-159">In some cases, rather than the **Open** state returning failure and raising an exception, it could be useful to return a default value that is meaningful to the application.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="ea9ab-160">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="ea9ab-160">Issues and considerations</span></span>

<span data-ttu-id="ea9ab-161">Os seguintes pontos devem ser considerados ao decidir como implementar esse padrão:</span><span class="sxs-lookup"><span data-stu-id="ea9ab-161">You should consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="ea9ab-162">**Tratamento de exceção**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-162">**Exception Handling**.</span></span> <span data-ttu-id="ea9ab-163">Um aplicativo invocando uma operação por meio de um disjuntor deverá estar preparado para tratar as exceções geradas se a operação não estiver disponível.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-163">An application invoking an operation through a circuit breaker must be prepared to handle the exceptions raised if the operation is unavailable.</span></span> <span data-ttu-id="ea9ab-164">A maneira como são as exceções são tratadas dependerá do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-164">The way exceptions are handled will be application specific.</span></span> <span data-ttu-id="ea9ab-165">Por exemplo, um aplicativo pode sofrer uma degradação temporária de sua funcionalidade, invocar uma operação alternativa para tentar executar a mesma tarefa ou obter os mesmos dados ou relatar a exceção ao usuário e pedir a que ele tente novamente mais tarde.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-165">For example, an application could temporarily degrade its functionality, invoke an alternative operation to try to perform the same task or obtain the same data, or report the exception to the user and ask them to try again later.</span></span>

<span data-ttu-id="ea9ab-166">**Tipos de exceções**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-166">**Types of Exceptions**.</span></span> <span data-ttu-id="ea9ab-167">Uma solicitação pode falhar por vários motivos, algumas das quais podem indicar um tipo mais grave de falha que outros.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-167">A request might fail for many reasons, some of which might indicate a more severe type of failure than others.</span></span> <span data-ttu-id="ea9ab-168">Por exemplo, uma solicitação pode falhar por cause de uma falha em um serviço remoto e sua recuperação demorará vários minutos ou porque o tempo limite foi atingido, pois o serviço está temporariamente sobrecarregado.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-168">For example, a request might fail because a remote service has crashed and will take several minutes to recover, or because of a timeout due to the service being temporarily overloaded.</span></span> <span data-ttu-id="ea9ab-169">Um disjuntor poderá examinar os tipos de exceções que ocorrem e ajustar a estratégia dependendo da natureza dessas exceções.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-169">A circuit breaker might be able to examine the types of exceptions that occur and adjust its strategy depending on the nature of these exceptions.</span></span> <span data-ttu-id="ea9ab-170">Por exemplo, ele pode exigir um número grande de exceções de tempo limite para acionar o estado **Aberto** do disjuntor em relação ao número de falhas devido ao serviço estar completamente não disponível.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-170">For example, it might require a larger number of timeout exceptions to trip the circuit breaker to the **Open** state compared to the number of failures due to the service being completely unavailable.</span></span>

<span data-ttu-id="ea9ab-171">**Registro em log**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-171">**Logging**.</span></span> <span data-ttu-id="ea9ab-172">Um disjuntor deve registrar em log todas as solicitações com falha (e possivelmente as solicitações bem-sucedidas) para permitir que um administrador monitore a integridade da operação.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-172">A circuit breaker should log all failed requests (and possibly successful requests) to enable an administrator to monitor the health of the operation.</span></span>

<span data-ttu-id="ea9ab-173">**Capacidade de recuperação**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-173">**Recoverability**.</span></span> <span data-ttu-id="ea9ab-174">Você deve configurar o disjuntor para coincidir com o padrão de probabilidade de recuperação da operação sendo protegida.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-174">You should configure the circuit breaker to match the likely recovery pattern of the operation it's protecting.</span></span> <span data-ttu-id="ea9ab-175">Por exemplo, se o disjuntor permanece no estado **Aberto** por um longo período, ele pode gerar exceções mesmo se o motivo da falha foi resolvido.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-175">For example, if the circuit breaker remains in the **Open** state for a long period, it could raise exceptions even if the reason for the failure has been resolved.</span></span> <span data-ttu-id="ea9ab-176">De forma semelhante, um disjuntor pode flutuar e reduzir os tempos de resposta dos aplicativos se mudar do estado **Aberto** para o estado **Entreaberto** muito rapidamente.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-176">Similarly, a circuit breaker could fluctuate and reduce the response times of applications if it switches from the **Open** state to the **Half-Open** state too quickly.</span></span>

<span data-ttu-id="ea9ab-177">**Teste de operações com falha**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-177">**Testing Failed Operations**.</span></span> <span data-ttu-id="ea9ab-178">No estado **Aberto**, em vez de usar um temporizador para determinar quando mudar para o estado **Entreaberto**, um disjuntor pode executar ping periodicamente no serviço remoto ou no recurso para determinar se ele fica disponível novamente.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-178">In the **Open** state, rather than using a timer to determine when to switch to the **Half-Open** state, a circuit breaker can instead periodically ping the remote service or resource to determine whether it's become available again.</span></span> <span data-ttu-id="ea9ab-179">Esse ping pode assumir a forma de uma tentativa de invocar uma operação que falhou anteriormente ou pode usar uma operação especial fornecida pelo serviço remoto especificamente para testar a integridade do serviço, conforme descrito pelo [Padrão de monitoramento do ponto de extremidade de integridade](health-endpoint-monitoring.md).</span><span class="sxs-lookup"><span data-stu-id="ea9ab-179">This ping could take the form of an attempt to invoke an operation that had previously failed, or it could use a special operation provided by the remote service specifically for testing the health of the service, as described by the [Health Endpoint Monitoring pattern](health-endpoint-monitoring.md).</span></span>

<span data-ttu-id="ea9ab-180">**Substituição manual**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-180">**Manual Override**.</span></span> <span data-ttu-id="ea9ab-181">Em um sistema em que o tempo de recuperação de uma operação com falha é extremamente variável, é útil fornecer uma opção de reinício manual que permita ao administrador fechar um disjuntor (e reiniciar o contador de falhas).</span><span class="sxs-lookup"><span data-stu-id="ea9ab-181">In a system where the recovery time for a failing operation is extremely variable, it's beneficial to provide a manual reset option that enables an administrator to close a circuit breaker (and reset the failure counter).</span></span> <span data-ttu-id="ea9ab-182">De modo semelhante, um administrador poderá forçar um disjuntor a entrar no estado **Aberto** (e reiniciar o temporizador de tempo limite) se a operação protegida pelo disjuntor estiver temporariamente não disponível.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-182">Similarly, an administrator could force a circuit breaker into the **Open** state (and restart the timeout timer) if the operation protected by the circuit breaker is temporarily unavailable.</span></span>

<span data-ttu-id="ea9ab-183">**Simultaneidade**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-183">**Concurrency**.</span></span> <span data-ttu-id="ea9ab-184">O mesmo disjuntor pode ser acessado por um grande número de instâncias simultâneas de um aplicativo.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-184">The same circuit breaker could be accessed by a large number of concurrent instances of an application.</span></span> <span data-ttu-id="ea9ab-185">A implementação não deve bloquear solicitações simultâneas nem adicionar uma sobrecarga excessiva a cada chamada a uma operação.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-185">The implementation shouldn't block concurrent requests or add excessive overhead to each call to an operation.</span></span>

<span data-ttu-id="ea9ab-186">**Diferenciação de recurso**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-186">**Resource Differentiation**.</span></span> <span data-ttu-id="ea9ab-187">Tenha cuidado ao usar um único disjuntor para um tipo de recurso se houver vários provedores independentes subjacentes.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-187">Be careful when using a single circuit breaker for one type of resource if there might be multiple underlying independent providers.</span></span> <span data-ttu-id="ea9ab-188">Por exemplo, em um armazenamento de dados contendo vários fragmentos, um fragmento pode estar totalmente acessível enquanto outro está enfrentando um problema temporário.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-188">For example, in a data store that contains multiple shards, one shard might be fully accessible while another is experiencing a temporary issue.</span></span> <span data-ttu-id="ea9ab-189">Se as respostas de erro nesses cenários forem mescladas, um aplicativo poderá tentar acessar alguns fragmentos, mesmo quando a falha for altamente provável, enquanto o acesso aos outros fragmentos poderá ser bloqueado, mesmo que provavelmente tenham êxito.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-189">If the error responses in these scenarios are merged, an application might try to access some shards even when failure is highly likely, while access to other shards might be blocked even though it's likely to succeed.</span></span>

<span data-ttu-id="ea9ab-190">**Disjuntor acelerado**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-190">**Accelerated Circuit Breaking**.</span></span> <span data-ttu-id="ea9ab-191">Às vezes, uma resposta de falha pode conter informações suficientes para que o disjuntor seja acionado imediatamente e permaneça assim por uma quantidade mínima de tempo.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-191">Sometimes a failure response can contain enough information for the circuit breaker to trip immediately and stay tripped for a minimum amount of time.</span></span> <span data-ttu-id="ea9ab-192">Por exemplo, a resposta de erro de um recurso compartilhado sobrecarregado pode indicar que uma nova tentativa imediata não seja recomendada e que o aplicativo, em vez disso, devesse tentar novamente em alguns minutos.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-192">For example, the error response from a shared resource that's overloaded could indicate that an immediate retry isn't recommended and that the application should instead try again in a few minutes.</span></span>

> [!NOTE]
> <span data-ttu-id="ea9ab-193">Um serviço poderá retornar HTTP 429 (Número excessivo de solicitações) se estiver limitando o cliente ou HTTP 503 (Serviço não disponível) se o serviço não estiver disponível no momento.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-193">A service can return HTTP 429 (Too Many Requests) if it is throttling the client, or HTTP 503 (Service Unavailable) if the service is not currently available.</span></span> <span data-ttu-id="ea9ab-194">A resposta pode incluir informações adicionais, como a duração prevista do atraso.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-194">The response can include additional information, such as the anticipated duration of the delay.</span></span>

<span data-ttu-id="ea9ab-195">**Reprodução das solicitações com falha**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-195">**Replaying Failed Requests**.</span></span> <span data-ttu-id="ea9ab-196">No estado **Aberto**, em vez de simplesmente falhar rapidamente, um disjuntor também poderá registrar os detalhes de cada solicitação em um diário e organizar essas solicitações para serem reproduzidas quando o serviço ou o recurso estiver disponível.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-196">In the **Open** state, rather than simply failing quickly, a circuit breaker could also record the details of each request to a journal and arrange for these requests to be replayed when the remote resource or service becomes available.</span></span>

<span data-ttu-id="ea9ab-197">**Tempos limite inadequados em serviços externos**.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-197">**Inappropriate Timeouts on External Services**.</span></span> <span data-ttu-id="ea9ab-198">Um disjuntor talvez não possa proteger totalmente os aplicativos das operações que falham em serviços externos configurados com um tempo limite longo.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-198">A circuit breaker might not be able to fully protect applications from operations that fail in external services that are configured with a lengthy timeout period.</span></span> <span data-ttu-id="ea9ab-199">Se o tempo limite for muito longo, um thread executando um disjuntor poderá ser bloqueado por um longo período antes de o disjuntor indicar a falha na operação.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-199">If the timeout is too long, a thread running a circuit breaker might be blocked for an extended period before the circuit breaker indicates that the operation has failed.</span></span> <span data-ttu-id="ea9ab-200">Nesse momento, muitas outras instâncias do aplicativo também poderão tentar invocar o serviço por meio do disjuntor e associar um número significativo de threads antes que todos falhem.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-200">In this time, many other application instances might also try to invoke the service through the circuit breaker and tie up a significant number of threads before they all fail.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="ea9ab-201">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="ea9ab-201">When to use this pattern</span></span>

<span data-ttu-id="ea9ab-202">Use este padrão:</span><span class="sxs-lookup"><span data-stu-id="ea9ab-202">Use this pattern:</span></span>

- <span data-ttu-id="ea9ab-203">Para impedir que um aplicativo tente invocar um serviço remoto ou acessar um recurso compartilhado se a operação tiver alta probabilidade de falhar.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-203">To prevent an application from trying to invoke a remote service or access a shared resource if this operation is highly likely to fail.</span></span>

<span data-ttu-id="ea9ab-204">Este padrão não é recomendável:</span><span class="sxs-lookup"><span data-stu-id="ea9ab-204">This pattern isn't recommended:</span></span>

- <span data-ttu-id="ea9ab-205">Para tratar do acesso a recursos particulares locais em um aplicativo, como a estrutura de dados na memória.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-205">For handling access to local private resources in an application, such as in-memory data structure.</span></span> <span data-ttu-id="ea9ab-206">Nesse ambiente, usar um disjuntor poderia adicionar uma sobrecarga ao sistema.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-206">In this environment, using a circuit breaker would add overhead to your system.</span></span>
- <span data-ttu-id="ea9ab-207">Como substituto para o tratamento de exceções na lógica de negócios dos seus aplicativos.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-207">As a substitute for handling exceptions in the business logic of your applications.</span></span>

## <a name="example"></a><span data-ttu-id="ea9ab-208">Exemplo</span><span class="sxs-lookup"><span data-stu-id="ea9ab-208">Example</span></span>

<span data-ttu-id="ea9ab-209">Em um aplicativo Web, muitas das páginas são populadas com os dados recuperados de um serviço externo.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-209">In a web application, several of the pages are populated with data retrieved from an external service.</span></span> <span data-ttu-id="ea9ab-210">Se o sistema implementar um armazenamento em cache mínimo, a maioria das ocorrências dessas páginas provocará uma viagem de ida e volta para o serviço.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-210">If the system implements minimal caching, most hits to these pages will cause a round trip to the service.</span></span> <span data-ttu-id="ea9ab-211">As conexões do aplicativo Web ao serviço podem ser configuradas com um tempo limite (normalmente 60 segundos) e, se o serviço não responder nesse período, a lógica em cada página da Web considerará que o serviço não está disponível e gerará uma exceção.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-211">Connections from the web application to the service could be configured with a timeout period (typically 60 seconds), and if the service doesn't respond in this time the logic in each web page will assume that the service is unavailable and throw an exception.</span></span>

<span data-ttu-id="ea9ab-212">No entanto, se o serviço falhar e o sistema estiver muito ocupado, os usuários poderão ser forçados a aguardar até 60 segundos antes que ocorra uma exceção.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-212">However, if the service fails and the system is very busy, users could be forced to wait for up to 60 seconds before an exception occurs.</span></span> <span data-ttu-id="ea9ab-213">Com o tempo, recursos como memória, conexões e threads poderão ser esgotados, evitando que outros usuários se conectem ao sistema, mesmo que não estejam acessando páginas que recuperam dados do serviço.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-213">Eventually resources such as memory, connections, and threads could be exhausted, preventing other users from connecting to the system, even if they aren't accessing pages that retrieve data from the service.</span></span>

<span data-ttu-id="ea9ab-214">O dimensionamento do sistema com a adição de mais servidores Web e a implementação do balanceamento de carga poderá sofrer atrasos quando os recursos acabarem, mas isso não resolverá o problema, uma vez que as solicitações do usuário ainda estarão sem resposta e todos os servidores Web ainda poderão ter seus recursos esgotados.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-214">Scaling the system by adding further web servers and implementing load balancing might delay when resources become exhausted, but it won't resolve the issue because user requests will still be unresponsive and all web servers could still eventually run out of resources.</span></span>

<span data-ttu-id="ea9ab-215">O encapsulamento da lógica que se conecta ao serviço e recupera os dados em um disjuntor pode ajudar a resolver esse problema e tratar a falha do serviço de modo mais elegante.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-215">Wrapping the logic that connects to the service and retrieves the data in a circuit breaker could help to solve this problem and handle the service failure more elegantly.</span></span> <span data-ttu-id="ea9ab-216">As solicitações do usuário ainda falharão, mas elas falharão mais rapidamente e os recursos não serão bloqueados.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-216">User requests will still fail, but they'll fail more quickly and the resources won't be blocked.</span></span>

<span data-ttu-id="ea9ab-217">A classe `CircuitBreaker` mantém informações de estado relacionadas a um disjuntor em um objeto que implementa a interface `ICircuitBreakerStateStore` mostrada no código a seguir.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-217">The `CircuitBreaker` class maintains state information about a circuit breaker in an object that implements the `ICircuitBreakerStateStore` interface shown in the following code.</span></span>

```csharp
interface ICircuitBreakerStateStore
{
  CircuitBreakerStateEnum State { get; }

  Exception LastException { get; }

  DateTime LastStateChangedDateUtc { get; }

  void Trip(Exception ex);

  void Reset();

  void HalfOpen();

  bool IsClosed { get; }
}
```

<span data-ttu-id="ea9ab-218">A propriedade `State` indica o estado atual do disjuntor e será **Open** (Aberto), **HalfOpen** (Entreaberto) ou **Closed** (Fechado) conforme definido pela enumeração `CircuitBreakerStateEnum`.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-218">The `State` property indicates the current state of the circuit breaker, and will be either **Open**, **HalfOpen**, or **Closed** as defined by the `CircuitBreakerStateEnum` enumeration.</span></span> <span data-ttu-id="ea9ab-219">A propriedade `IsClosed` deverá ser true se o disjuntor estiver fechado, mas false se ele estiver aberto ou entreaberto.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-219">The `IsClosed` property should be true if the circuit breaker is closed, but false if it's open or half open.</span></span> <span data-ttu-id="ea9ab-220">O método `Trip` muda o estado do disjuntor para o estado aberto e registra a exceção que causou a alteração no estado, junto com a data e a hora em que a exceção ocorreu.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-220">The `Trip` method switches the state of the circuit breaker to the open state and records the exception that caused the change in state, together with the date and time that the exception occurred.</span></span> <span data-ttu-id="ea9ab-221">As propriedades `LastException` e `LastStateChangedDateUtc` retornam essas informações.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-221">The `LastException` and the `LastStateChangedDateUtc` properties return this information.</span></span> <span data-ttu-id="ea9ab-222">O método `Reset` fecha o disjuntor e o método `HalfOpen` define o disjuntor como entreaberto.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-222">The `Reset` method closes the circuit breaker, and the `HalfOpen` method sets the circuit breaker to half open.</span></span>

<span data-ttu-id="ea9ab-223">A classe `InMemoryCircuitBreakerStateStore` no exemplo contém uma implementação da interface `ICircuitBreakerStateStore`.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-223">The `InMemoryCircuitBreakerStateStore` class in the example contains an implementation of the `ICircuitBreakerStateStore` interface.</span></span> <span data-ttu-id="ea9ab-224">A classe `CircuitBreaker` cria uma instância dessa classe para manter o estado do disjuntor.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-224">The `CircuitBreaker` class creates an instance of this class to hold the state of the circuit breaker.</span></span>

<span data-ttu-id="ea9ab-225">O método `ExecuteAction` na classe `CircuitBreaker` encapsula uma operação, especificada como um delegado `Action`.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-225">The `ExecuteAction` method in the `CircuitBreaker` class wraps an operation, specified as an `Action` delegate.</span></span> <span data-ttu-id="ea9ab-226">Se o disjuntor estiver fechado, `ExecuteAction` invocará o delegado `Action`.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-226">If the circuit breaker is closed, `ExecuteAction` invokes the `Action` delegate.</span></span> <span data-ttu-id="ea9ab-227">Se a operação falhar, um manipulador de exceção chamará `TrackException`, que definirá o estado do disjuntor como aberto.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-227">If the operation fails, an exception handler calls `TrackException`, which sets the circuit breaker state to open.</span></span> <span data-ttu-id="ea9ab-228">O exemplo de código a seguir realça esse fluxo.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-228">The following code example highlights this flow.</span></span>

```csharp
public class CircuitBreaker
{
  private readonly ICircuitBreakerStateStore stateStore =
    CircuitBreakerStateStoreFactory.GetCircuitBreakerStateStore();

  private readonly object halfOpenSyncObject = new object ();
  ...
  public bool IsClosed { get { return stateStore.IsClosed; } }

  public bool IsOpen { get { return !IsClosed; } }

  public void ExecuteAction(Action action)
  {
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open.
      ... (see code sample below for details)
    }

    // The circuit breaker is Closed, execute the action.
    try
    {
      action();
    }
    catch (Exception ex)
    {
      // If an exception still occurs here, simply
      // retrip the breaker immediately.
      this.TrackException(ex);

      // Throw the exception so that the caller can tell
      // the type of exception that was thrown.
      throw;
    }
  }

  private void TrackException(Exception ex)
  {
    // For simplicity in this example, open the circuit breaker on the first exception.
    // In reality this would be more complex. A certain type of exception, such as one
    // that indicates a service is offline, might trip the circuit breaker immediately.
    // Alternatively it might count exceptions locally or across multiple instances and
    // use this value over time, or the exception/success ratio based on the exception
    // types, to open the circuit breaker.
    this.stateStore.Trip(ex);
  }
}
```

<span data-ttu-id="ea9ab-229">O exemplo a seguir mostra o código (omitido do exemplo anterior) executado se o disjuntor não estiver fechado.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-229">The following example shows the code (omitted from the previous example) that is executed if the circuit breaker isn't closed.</span></span> <span data-ttu-id="ea9ab-230">Ele primeiro verifica se o disjuntor esteve aberto por um período maior que o tempo especificado pelo campo `OpenToHalfOpenWaitTime` local na classe `CircuitBreaker`.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-230">It first checks if the circuit breaker has been open for a period longer than the time specified by the local `OpenToHalfOpenWaitTime` field in the `CircuitBreaker` class.</span></span> <span data-ttu-id="ea9ab-231">Se esse for o caso, o método `ExecuteAction` definirá o disjuntor como entreaberto e, em seguida, tentará executar a operação especificada pelo delegado `Action`.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-231">If this is the case, the `ExecuteAction` method sets the circuit breaker to half open, then tries to perform the operation specified by the `Action` delegate.</span></span>

<span data-ttu-id="ea9ab-232">Se a operação for bem-sucedida, o disjuntor será reiniciado no estado fechado.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-232">If the operation is successful, the circuit breaker is reset to the closed state.</span></span> <span data-ttu-id="ea9ab-233">Se a operação falhar, ele voltará ao estado aberto e a hora em que a exceção ocorreu será atualizada para que o disjuntor aguarde mais um pouco antes de tentar executar a operação novamente.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-233">If the operation fails, it is tripped back to the open state and the time the exception occurred is updated so that the circuit breaker will wait for a further period before trying to perform the operation again.</span></span>

<span data-ttu-id="ea9ab-234">Se o disjuntor só tiver sido aberto por um curto período de tempo, menor que o valor `OpenToHalfOpenWaitTime`, o método `ExecuteAction` simplesmente gerará uma exceção `CircuitBreakerOpenException` e retornará o erro que fez com que o disjuntor fosse para o estado aberto.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-234">If the circuit breaker has only been open for a short time, less than the `OpenToHalfOpenWaitTime` value, the `ExecuteAction` method simply throws a `CircuitBreakerOpenException` exception and returns the error that caused the circuit breaker to transition to the open state.</span></span>

<span data-ttu-id="ea9ab-235">Além disso, ele usará um bloqueio para impedir que o disjuntor tente realizar chamadas simultâneas à operação enquanto estiver entreaberto.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-235">Additionally, it uses a lock to prevent the circuit breaker from trying to perform concurrent calls to the operation while it's half open.</span></span> <span data-ttu-id="ea9ab-236">Uma tentativa simultânea de invocar a operação será tratada como se o disjuntor estivesse aberto e falhará com uma exceção, conforme descrito posteriormente.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-236">A concurrent attempt to invoke the operation will be handled as if the circuit breaker was open, and it'll fail with an exception as described later.</span></span>

```csharp
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open. Check if the Open timeout has expired.
      // If it has, set the state to HalfOpen. Another approach might be to
      // check for the HalfOpen state that had be set by some other operation.
      if (stateStore.LastStateChangedDateUtc + OpenToHalfOpenWaitTime < DateTime.UtcNow)
      {
        // The Open timeout has expired. Allow one operation to execute. Note that, in
        // this example, the circuit breaker is set to HalfOpen after being
        // in the Open state for some period of time. An alternative would be to set
        // this using some other approach such as a timer, test method, manually, and
        // so on, and check the state here to determine how to handle execution
        // of the action.
        // Limit the number of threads to be executed when the breaker is HalfOpen.
        // An alternative would be to use a more complex approach to determine which
        // threads or how many are allowed to execute, or to execute a simple test
        // method instead.
        bool lockTaken = false;
        try
        {
          Monitor.TryEnter(halfOpenSyncObject, ref lockTaken);
          if (lockTaken)
          {
            // Set the circuit breaker state to HalfOpen.
            stateStore.HalfOpen();

            // Attempt the operation.
            action();

            // If this action succeeds, reset the state and allow other operations.
            // In reality, instead of immediately returning to the Closed state, a counter
            // here would record the number of successful operations and return the
            // circuit breaker to the Closed state only after a specified number succeed.
            this.stateStore.Reset();
            return;
          }
        }
        catch (Exception ex)
        {
          // If there's still an exception, trip the breaker again immediately.
          this.stateStore.Trip(ex);

          // Throw the exception so that the caller knows which exception occurred.
          throw;
        }
        finally
        {
          if (lockTaken)
          {
            Monitor.Exit(halfOpenSyncObject);
          }
        }
      }
      // The Open timeout hasn't yet expired. Throw a CircuitBreakerOpen exception to
      // inform the caller that the call was not actually attempted,
      // and return the most recent exception received.
      throw new CircuitBreakerOpenException(stateStore.LastException);
    }
    ...
```

<span data-ttu-id="ea9ab-237">Para usar um objeto `CircuitBreaker` para proteger uma operação, um aplicativo criará uma instância da classe `CircuitBreaker` e invocará o método `ExecuteAction`, especificando a operação a ser executada como o parâmetro.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-237">To use a `CircuitBreaker` object to protect an operation, an application creates an instance of the `CircuitBreaker` class and invokes the `ExecuteAction` method, specifying the operation to be performed as the parameter.</span></span> <span data-ttu-id="ea9ab-238">O aplicativo deverá estar preparado para detectar a exceção `CircuitBreakerOpenException` se a operação falhar porque o disjuntor estará aberto.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-238">The application should be prepared to catch the `CircuitBreakerOpenException` exception if the operation fails because the circuit breaker is open.</span></span> <span data-ttu-id="ea9ab-239">O código a seguir mostra um exemplo:</span><span class="sxs-lookup"><span data-stu-id="ea9ab-239">The following code shows an example:</span></span>

```csharp
var breaker = new CircuitBreaker();

try
{
  breaker.ExecuteAction(() =>
  {
    // Operation protected by the circuit breaker.
    ...
  });
}
catch (CircuitBreakerOpenException ex)
{
  // Perform some different action when the breaker is open.
  // Last exception details are in the inner exception.
  ...
}
catch (Exception ex)
{
  ...
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="ea9ab-240">Diretrizes e padrões relacionados</span><span class="sxs-lookup"><span data-stu-id="ea9ab-240">Related patterns and guidance</span></span>

<span data-ttu-id="ea9ab-241">Os seguintes padrões também serão úteis ao implementar este padrão:</span><span class="sxs-lookup"><span data-stu-id="ea9ab-241">The following patterns might also be useful when implementing this pattern:</span></span>

- <span data-ttu-id="ea9ab-242">[Padrão de repetição][retry-pattern].</span><span class="sxs-lookup"><span data-stu-id="ea9ab-242">[Retry Pattern][retry-pattern].</span></span> <span data-ttu-id="ea9ab-243">Descreve como um aplicativo pode tratar falhas previstas e temporárias quando tentar se conectar a um serviço ou recurso de rede repetindo de forma transparente uma operação que falhou anteriormente.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-243">Describes how an application can handle anticipated temporary failures when it tries to connect to a service or network resource by transparently retrying an operation that has previously failed.</span></span>

- <span data-ttu-id="ea9ab-244">[Padrão de monitoramento do ponto de extremidade de integridade](health-endpoint-monitoring.md).</span><span class="sxs-lookup"><span data-stu-id="ea9ab-244">[Health Endpoint Monitoring Pattern](health-endpoint-monitoring.md).</span></span> <span data-ttu-id="ea9ab-245">Um disjuntor pode testar a integridade de um serviço enviando uma solicitação para um ponto de extremidade exposto pelo serviço.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-245">A circuit breaker might be able to test the health of a service by sending a request to an endpoint exposed by the service.</span></span> <span data-ttu-id="ea9ab-246">O serviço deve retornar informações indicando seu status.</span><span class="sxs-lookup"><span data-stu-id="ea9ab-246">The service should return information indicating its status.</span></span>


[retry-pattern]: ./retry.md
