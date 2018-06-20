---
title: Princípios de design para aplicativos do Azure
description: Princípios de design para aplicativos do Azure
author: MikeWasson
ms.openlocfilehash: 462896098c668c0775464ca498925266cd73c6e1
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206790"
---
# <a name="design-principles-for-azure-applications"></a><span data-ttu-id="4e324-103">Princípios de design para aplicativos do Azure</span><span class="sxs-lookup"><span data-stu-id="4e324-103">Design principles for Azure applications</span></span>

<span data-ttu-id="4e324-104">Siga esses princípios de design para tornar seu aplicativo mais escalonável, flexível e gerenciável.</span><span class="sxs-lookup"><span data-stu-id="4e324-104">Follow these design principles to make your application more scalable, resilient, and manageable.</span></span> 

<span data-ttu-id="4e324-105">**[Design de autorrecuperação](self-healing.md)**.</span><span class="sxs-lookup"><span data-stu-id="4e324-105">**[Design for self healing](self-healing.md)**.</span></span> <span data-ttu-id="4e324-106">Em um sistema distribuído, falhas de acontecem.</span><span class="sxs-lookup"><span data-stu-id="4e324-106">In a distributed system, failures happen.</span></span> <span data-ttu-id="4e324-107">Projete seu aplicativo para ter autorrecuperação quando houver falhas.</span><span class="sxs-lookup"><span data-stu-id="4e324-107">Design your application to be self healing when failures occur.</span></span>

<span data-ttu-id="4e324-108">**[Tornar todas as coisas redundantes](redundancy.md)**.</span><span class="sxs-lookup"><span data-stu-id="4e324-108">**[Make all things redundant](redundancy.md)**.</span></span> <span data-ttu-id="4e324-109">Crie redundância no seu aplicativo, evite ter pontos únicos de falha.</span><span class="sxs-lookup"><span data-stu-id="4e324-109">Build redundancy into your application, to avoid having single points of failure.</span></span>
 
<span data-ttu-id="4e324-110">**[Minimizar a coordenação](minimize-coordination.md)**.</span><span class="sxs-lookup"><span data-stu-id="4e324-110">**[Minimize coordination](minimize-coordination.md)**.</span></span> <span data-ttu-id="4e324-111">Minimize a coordenação entre os serviços de aplicativos para atingir a escalabilidade.</span><span class="sxs-lookup"><span data-stu-id="4e324-111">Minimize coordination between application services to achieve scalability.</span></span>
 
<span data-ttu-id="4e324-112">**[Design de expansão](scale-out.md)**. Crie seu aplicativo para que ele possa ser dimensionado horizontalmente, adicionando ou removendo novas instâncias como exige.</span><span class="sxs-lookup"><span data-stu-id="4e324-112">**[Design to scale out](scale-out.md)**. Design your application so that it can scale horizontally, adding or removing new instances as demand requires.</span></span>

<span data-ttu-id="4e324-113">**[Partição de limites](partition.md)**.</span><span class="sxs-lookup"><span data-stu-id="4e324-113">**[Partition around limits](partition.md)**.</span></span> <span data-ttu-id="4e324-114">Use particionamento para contornar o banco de dados, rede e limites de computação.</span><span class="sxs-lookup"><span data-stu-id="4e324-114">Use partitioning to work around database, network, and compute limits.</span></span>

<span data-ttu-id="4e324-115">**[Design para operações](design-for-operations.md)**.</span><span class="sxs-lookup"><span data-stu-id="4e324-115">**[Design for operations](design-for-operations.md)**.</span></span> <span data-ttu-id="4e324-116">Crie seu aplicativo para que a equipe de operações tenha as ferramentas necessárias.</span><span class="sxs-lookup"><span data-stu-id="4e324-116">Design your application so that the operations team has the tools they need.</span></span>

<span data-ttu-id="4e324-117">**[Serviços gerenciados pelo usuário](managed-services.md)**.</span><span class="sxs-lookup"><span data-stu-id="4e324-117">**[Use managed services](managed-services.md)**.</span></span> <span data-ttu-id="4e324-118">Quando possível, utilize a plataforma como um serviço (PaaS) em vez de infraestrutura como serviço (IaaS).</span><span class="sxs-lookup"><span data-stu-id="4e324-118">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS).</span></span>

<span data-ttu-id="4e324-119">**[Use o melhor armazenamento de dados para o trabalho](use-the-best-data-store.md)**.</span><span class="sxs-lookup"><span data-stu-id="4e324-119">**[Use the best data store for the job](use-the-best-data-store.md)**.</span></span> <span data-ttu-id="4e324-120">Escolha a tecnologia de armazenamento que é o mais adequado para seus dados e como ele será usado.</span><span class="sxs-lookup"><span data-stu-id="4e324-120">Pick the storage technology that is the best fit for your data and how it will be used.</span></span> 
 
<span data-ttu-id="4e324-121">**[Design para evolução](design-for-evolution.md)**.</span><span class="sxs-lookup"><span data-stu-id="4e324-121">**[Design for evolution](design-for-evolution.md)**.</span></span> <span data-ttu-id="4e324-122">Alterar todos os aplicativos com êxito ao longo do tempo.</span><span class="sxs-lookup"><span data-stu-id="4e324-122">All successful applications change over time.</span></span> <span data-ttu-id="4e324-123">Um design evolutivo é a chave para a inovação contínua.</span><span class="sxs-lookup"><span data-stu-id="4e324-123">An evolutionary design is key for continuous innovation.</span></span>

<span data-ttu-id="4e324-124">**[Crie de acordo com as necessidades de negócios](build-for-business.md)**.</span><span class="sxs-lookup"><span data-stu-id="4e324-124">**[Build for the needs of business](build-for-business.md)**.</span></span> <span data-ttu-id="4e324-125">Todas as decisões de design devem ser justificado por um requisito de negócios.</span><span class="sxs-lookup"><span data-stu-id="4e324-125">Every design decision must be justified by a business requirement.</span></span>

