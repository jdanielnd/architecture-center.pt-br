---
title: "Padrão do Estrangulador"
description: "Migre incrementalmente um sistema herdado substituindo gradualmente partes específicas de funcionalidade por serviços e aplicativos novos."
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: d03e8a1ef9077b6e00ea5a17423bf7e09b68111a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="strangler-pattern"></a><span data-ttu-id="c533a-103">Padrão do Estrangulador</span><span class="sxs-lookup"><span data-stu-id="c533a-103">Strangler pattern</span></span>

<span data-ttu-id="c533a-104">Migre incrementalmente um sistema herdado substituindo gradualmente partes específicas de funcionalidade por serviços e aplicativos novos.</span><span class="sxs-lookup"><span data-stu-id="c533a-104">Incrementally migrate a legacy system by gradually replacing specific pieces of functionality with new applications and services.</span></span> <span data-ttu-id="c533a-105">Como os recursos do sistema herdado são substituídos, o novo sistema acaba substituindo todos os recursos do sistema antigo, estrangulando o sistema antigo e permitindo que você o encerre.</span><span class="sxs-lookup"><span data-stu-id="c533a-105">As features from the legacy system are replaced, the new system eventually replaces all of the old system's features, strangling the old system and allowing you to decommission it.</span></span> 

## <a name="context-and-problem"></a><span data-ttu-id="c533a-106">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="c533a-106">Context and problem</span></span>

<span data-ttu-id="c533a-107">Conforme o sistema envelhece, as ferramentas de desenvolvimento, a tecnologia de hospedagem e até mesmo as arquiteturas do sistema em que elas criadas podem se tornar cada vez mais obsoletas.</span><span class="sxs-lookup"><span data-stu-id="c533a-107">As systems age, the development tools, hosting technology, and even system architectures they were built on can become increasingly obsolete.</span></span> <span data-ttu-id="c533a-108">Conforme são adicionados novos recursos e funcionalidades, a complexidade desses aplicativos pode aumentar consideravelmente, tornando mais difícil de manter ou adicionar novos recursos.</span><span class="sxs-lookup"><span data-stu-id="c533a-108">As new features and functionality are added, the complexity of these applications can increase dramatically, making them harder to maintain or add new features to.</span></span>

<span data-ttu-id="c533a-109">Substituir completamente um sistema complexo pode ser uma tarefa muito grande.</span><span class="sxs-lookup"><span data-stu-id="c533a-109">Completely replacing a complex system can be a huge undertaking.</span></span> <span data-ttu-id="c533a-110">Geralmente, você precisará de uma migração gradual para um novo sistema, mantendo o sistema antigo para manipular recursos que ainda não tenham sido migrados.</span><span class="sxs-lookup"><span data-stu-id="c533a-110">Often, you will need a gradual migration to a new system, while keeping the old system to handle features that haven't been migrated yet.</span></span> <span data-ttu-id="c533a-111">No entanto, executar duas versões diferentes de um aplicativo significa que os clientes precisam saber o local em que determinados recursos estão localizados.</span><span class="sxs-lookup"><span data-stu-id="c533a-111">However, running two separate versions of an application means that clients have to know where particular features are located.</span></span> <span data-ttu-id="c533a-112">Sempre que um recurso ou serviço for migrado, os clientes precisarão ser atualizados para apontar para o novo local.</span><span class="sxs-lookup"><span data-stu-id="c533a-112">Every time a feature or service is migrated, clients need to be updated to point to the new location.</span></span>

## <a name="solution"></a><span data-ttu-id="c533a-113">Solução</span><span class="sxs-lookup"><span data-stu-id="c533a-113">Solution</span></span>

<span data-ttu-id="c533a-114">Substitua incrementalmente partes específicas da funcionalidade por novos serviços e aplicativos.</span><span class="sxs-lookup"><span data-stu-id="c533a-114">Incrementally replace specific pieces of functionality with new applications and services.</span></span> <span data-ttu-id="c533a-115">Crie uma fachada que intercepte solicitações para o sistema herdado de back-end.</span><span class="sxs-lookup"><span data-stu-id="c533a-115">Create a façade that intercepts requests going to the backend legacy system.</span></span> <span data-ttu-id="c533a-116">A fachada encaminha essas solicitações para o aplicativo herdado ou para os novos serviços.</span><span class="sxs-lookup"><span data-stu-id="c533a-116">The façade routes these requests either to the legacy application or the new services.</span></span> <span data-ttu-id="c533a-117">Os recursos existentes podem ser migrados para o novo sistema gradualmente e os consumidores podem continuar usando a mesma interface, sem que estejam cientes de que qualquer migração ocorreu.</span><span class="sxs-lookup"><span data-stu-id="c533a-117">Existing features can be migrated to the new system gradually, and consumers can continue using the same interface, unaware that any migration has taken place.</span></span>

![](./_images/strangler.png)  

<span data-ttu-id="c533a-118">Esse padrão ajuda a minimizar o risco da migração e distribui o esforço de desenvolvimento ao longo do tempo.</span><span class="sxs-lookup"><span data-stu-id="c533a-118">This pattern helps to minimize risk from the migration, and spread the development effort over time.</span></span> <span data-ttu-id="c533a-119">Com a fachada roteando com segurança os usuários para o aplicativo correto, você pode adicionar funcionalidade ao novo sistema em qualquer velocidade que desejar, enquanto garante que o aplicativo herdado continua a funcionar.</span><span class="sxs-lookup"><span data-stu-id="c533a-119">With the façade safely routing users to the correct application, you can add functionality to the new system at whatever pace you like, while ensuring the legacy application continues to function.</span></span> <span data-ttu-id="c533a-120">Ao longo do tempo, conforme os recursos são migrados para o novo sistema, o sistema herdado acaba sendo “estrangulado" e não é mais necessário.</span><span class="sxs-lookup"><span data-stu-id="c533a-120">Over time, as features are migrated to the new system, the legacy system is eventually "strangled" and is no longer necessary.</span></span> <span data-ttu-id="c533a-121">Quando esse processo for concluído, o sistema herdado com segurança poderá ser descontinuado.</span><span class="sxs-lookup"><span data-stu-id="c533a-121">Once this process is complete, the legacy system can safely be retired.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="c533a-122">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="c533a-122">Issues and considerations</span></span>

- <span data-ttu-id="c533a-123">Considere como lidar com serviços e armazenamentos de dados potencialmente usados por sistemas herdados e novos.</span><span class="sxs-lookup"><span data-stu-id="c533a-123">Consider how to handle services and data stores that are potentially used by both new and legacy systems.</span></span> <span data-ttu-id="c533a-124">Verifique se ambos podem acessar os recursos lado a lado.</span><span class="sxs-lookup"><span data-stu-id="c533a-124">Make sure both can access these resources side-by-side.</span></span>
- <span data-ttu-id="c533a-125">Estruture novos aplicativos e serviços de maneira que possam facilmente ser interceptados e substituídos em futuras migrações com estrangulamento.</span><span class="sxs-lookup"><span data-stu-id="c533a-125">Structure new applications and services in a way that they can easily be intercepted and replaced in future strangler migrations.</span></span>
- <span data-ttu-id="c533a-126">Em algum momento, quando a migração for concluída, fachada de estrangulamento desaparecerá ou se transformará em um adaptador para clientes herdados.</span><span class="sxs-lookup"><span data-stu-id="c533a-126">At some point, when the migration is complete, the strangler façade will either go away or evolve into an adaptor for legacy clients.</span></span>
- <span data-ttu-id="c533a-127">Verifique se a fachada acompanha a migração.</span><span class="sxs-lookup"><span data-stu-id="c533a-127">Make sure the façade keeps up with the migration.</span></span>
- <span data-ttu-id="c533a-128">Verifique se a fachada não se torna um ponto único de falha ou um gargalo de desempenho.</span><span class="sxs-lookup"><span data-stu-id="c533a-128">Make sure the façade doesn't become a single point of failure or a performance bottleneck.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="c533a-129">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="c533a-129">When to use this pattern</span></span>

<span data-ttu-id="c533a-130">Use esse padrão ao migrar gradualmente um aplicativo de back-end para uma nova arquitetura.</span><span class="sxs-lookup"><span data-stu-id="c533a-130">Use this pattern when gradually migrating a back-end application to a new architecture.</span></span>

<span data-ttu-id="c533a-131">Esse padrão pode não ser adequado:</span><span class="sxs-lookup"><span data-stu-id="c533a-131">This pattern may not be suitable:</span></span>

- <span data-ttu-id="c533a-132">Quando as solicitações ao sistema de back-end não podem ser interceptadas.</span><span class="sxs-lookup"><span data-stu-id="c533a-132">When requests to the back-end system cannot be intercepted.</span></span>
- <span data-ttu-id="c533a-133">Para sistemas menores em que a complexidade de toda a substituição é baixa.</span><span class="sxs-lookup"><span data-stu-id="c533a-133">For smaller systems where the complexity of wholesale replacement is low.</span></span>

## <a name="related-guidance"></a><span data-ttu-id="c533a-134">Diretrizes relacionadas</span><span class="sxs-lookup"><span data-stu-id="c533a-134">Related guidance</span></span>

- [<span data-ttu-id="c533a-135">Padrão de camada anticorrupção</span><span class="sxs-lookup"><span data-stu-id="c533a-135">Anti-Corruption Layer pattern</span></span>](./anti-corruption-layer.md)
- [<span data-ttu-id="c533a-136">Padrão de roteamento de gateway</span><span class="sxs-lookup"><span data-stu-id="c533a-136">Gateway Routing pattern</span></span>](./gateway-routing.md)


 

