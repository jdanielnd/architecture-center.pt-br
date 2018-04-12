---
title: Padrões de disponibilidade
description: A disponibilidade define a proporção de tempo em que o sistema está funcionando. Ele será afetado por erros de sistema, problemas de infraestrutura, ataques mal-intencionados e carga do sistema. Ele geralmente é medido como uma porcentagem do tempo de atividade. Os aplicativos de nuvem normalmente fornecem aos usuários um contrato de nível de serviço (SLA), o que significa que os aplicativos devem ser criados e implementados para maximizar a disponibilidade.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: eac1d192fe83f8501d11dce02d9bec16e939ba55
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
---
# <a name="availability-patterns"></a><span data-ttu-id="16a4e-107">Padrões de disponibilidade</span><span class="sxs-lookup"><span data-stu-id="16a4e-107">Availability patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="16a4e-108">A disponibilidade define a proporção de tempo em que o sistema está funcionando.</span><span class="sxs-lookup"><span data-stu-id="16a4e-108">Availability defines the proportion of time that the system is functional and working.</span></span> <span data-ttu-id="16a4e-109">Ele será afetado por erros de sistema, problemas de infraestrutura, ataques mal-intencionados e carga do sistema.</span><span class="sxs-lookup"><span data-stu-id="16a4e-109">It will be affected by system errors, infrastructure problems, malicious attacks, and system load.</span></span> <span data-ttu-id="16a4e-110">Ele geralmente é medido como uma porcentagem do tempo de atividade.</span><span class="sxs-lookup"><span data-stu-id="16a4e-110">It is usually measured as a percentage of uptime.</span></span> <span data-ttu-id="16a4e-111">Os aplicativos de nuvem normalmente fornecem aos usuários um contrato de nível de serviço (SLA), o que significa que os aplicativos devem ser criados e implementados para maximizar a disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="16a4e-111">Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.</span></span>


|                            <span data-ttu-id="16a4e-112">Padrão</span><span class="sxs-lookup"><span data-stu-id="16a4e-112">Pattern</span></span>                             |                                                           <span data-ttu-id="16a4e-113">Resumo</span><span class="sxs-lookup"><span data-stu-id="16a4e-113">Summary</span></span>                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="16a4e-114">Monitoramento do Ponto de Extremidade de Integridade</span><span class="sxs-lookup"><span data-stu-id="16a4e-114">Health Endpoint Monitoring</span></span>](../health-endpoint-monitoring.md) | <span data-ttu-id="16a4e-115">Implemente verificações funcionais dentro de um aplicativo cujas ferramentas externas podem acessar por meio de pontos de extremidade expostos em intervalos regulares.</span><span class="sxs-lookup"><span data-stu-id="16a4e-115">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> |
|  [<span data-ttu-id="16a4e-116">Nivelamento de Carga Baseado em Fila</span><span class="sxs-lookup"><span data-stu-id="16a4e-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md)  | <span data-ttu-id="16a4e-117">Use uma fila que funcione como um buffer entre uma tarefa e um serviço que ela invoca para simplificar cargas pesadas intermitentes.</span><span class="sxs-lookup"><span data-stu-id="16a4e-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span>  |
|                 [<span data-ttu-id="16a4e-118">Limitação</span><span class="sxs-lookup"><span data-stu-id="16a4e-118">Throttling</span></span>](../throttling.md)                 |   <span data-ttu-id="16a4e-119">Controle o consumo de recursos usados por uma instância de um aplicativo, um locatário individual ou todo o serviço.</span><span class="sxs-lookup"><span data-stu-id="16a4e-119">Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.</span></span>    |

