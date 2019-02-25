---
title: 'CAF: Redes definidas por software - somente PaaS'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Discussão sobre o modelo de PaaS somente para a nuvem com base em funcionalidade de rede
author: rotycenh
ms.openlocfilehash: 2f3f82d781ddb6544721e82e7b7d795222a2f8ff
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900290"
---
# <a name="software-defined-networks-paas-only"></a><span data-ttu-id="1a9ce-103">Redes definidas por software: PaaS-only</span><span class="sxs-lookup"><span data-stu-id="1a9ce-103">Software Defined Networks: PaaS-only</span></span>

<span data-ttu-id="1a9ce-104">Quando você implementa uma plataforma como um recurso de serviço (PaaS), o processo de implantação cria automaticamente uma rede subjacente assumida com um número limitado de controles na rede, incluindo balanceamento de carga, bloqueio de porta e conexões com outros serviços PaaS.</span><span class="sxs-lookup"><span data-stu-id="1a9ce-104">When you implement a platform as a service (PaaS) resource, the deployment process automatically creates an assumed underlying network with a limited number of controls over that network, including load balancing, port blocking, and connections to other PaaS services.</span></span>

<span data-ttu-id="1a9ce-105">No Azure, vários tipos de recursos de PaaS podem ser [implantados](/azure/virtual-network/virtual-network-for-azure-services) ou [conectados a](/azure/virtual-network/virtual-network-service-endpoints-overview) uma rede virtual, permitindo que esses recursos integrem sua infraestrutura de rede virtual existente.</span><span class="sxs-lookup"><span data-stu-id="1a9ce-105">In Azure, several PaaS resource types can be [deployed into](/azure/virtual-network/virtual-network-for-azure-services) or [connected to](/azure/virtual-network/virtual-network-service-endpoints-overview) a virtual network, allowing these resources to integrate with your existing virtual networking infrastructure.</span></span> <span data-ttu-id="1a9ce-106">No entanto, em muitos casos, uma arquitetura de rede apenas PaaS, conta somente com esses recursos de rede fornecidos originalmente pelos recursos PaaS, é suficiente para atender aos requisitos de carga de trabalho.</span><span class="sxs-lookup"><span data-stu-id="1a9ce-106">However, in many cases a PaaS only networking architecture, relying only on these default networking capabilities natively provided by PaaS resources, is sufficient to meet workload requirements.</span></span>

<span data-ttu-id="1a9ce-107">Se você estiver considerando uma arquitetura de rede somente de PaaS, certifique-se de que validar as suposições necessárias se alinha suas necessidades.</span><span class="sxs-lookup"><span data-stu-id="1a9ce-107">If you are considering a PaaS only networking architecture, be sure you validate that the required assumptions align with your requirements.</span></span>

## <a name="paas-only-assumptions"></a><span data-ttu-id="1a9ce-108">Suposições somente PaaS</span><span class="sxs-lookup"><span data-stu-id="1a9ce-108">PaaS-only assumptions</span></span>

<span data-ttu-id="1a9ce-109">Implantação de uma arquitetura de rede somente PaaS pressupõe o seguinte:</span><span class="sxs-lookup"><span data-stu-id="1a9ce-109">Deploying a PaaS-only networking architecture assumes the following:</span></span>

- <span data-ttu-id="1a9ce-110">O aplicativo que está sendo implantado é um aplicativo autônomo ou depende apenas de outros recursos de PaaS.</span><span class="sxs-lookup"><span data-stu-id="1a9ce-110">The application being deployed is a standalone application OR is dependent on only other PaaS resources.</span></span>
- <span data-ttu-id="1a9ce-111">As equipes de operações de TI podem atualizar suas ferramentas, treinamento e processos para dar suporte ao gerenciamento, configuração e implantação de aplicativos de PaaS autônomo.</span><span class="sxs-lookup"><span data-stu-id="1a9ce-111">Your IT operations teams can update their tools, training, and processes to support management, configuration, and deployment of standalone PaaS applications.</span></span>
- <span data-ttu-id="1a9ce-112">O aplicativo de PaaS não faz parte de um esforço de migração de nuvem mais abrangente que inclui recursos de IaaS.</span><span class="sxs-lookup"><span data-stu-id="1a9ce-112">The PaaS application is not part of a broader cloud migration effort that will include IaaS resources.</span></span>

<span data-ttu-id="1a9ce-113">Essas suposições são qualificadores mínimos alinhados ao implantar uma rede somente PaaS.</span><span class="sxs-lookup"><span data-stu-id="1a9ce-113">These assumptions are minimum qualifiers aligned to deploying a PaaS-only network.</span></span> <span data-ttu-id="1a9ce-114">Embora essa abordagem pode se alinhe com os requisitos de implantação de um único aplicativo, sua equipe de adoção de nuvem deve examinar essas perguntas de longo prazo:</span><span class="sxs-lookup"><span data-stu-id="1a9ce-114">While this approach may align with the requirements of a single application deployment, your Cloud Adoption Team should examine these long-term questions:</span></span>

- <span data-ttu-id="1a9ce-115">Essa implantação será expandida no escopo ou escala para exigir acesso a outros recursos sem PaaS?</span><span class="sxs-lookup"><span data-stu-id="1a9ce-115">Will this deployment expand in scope or scale to require access to other non-PaaS resources?</span></span>
- <span data-ttu-id="1a9ce-116">Outras implantações de PaaS estão planejadas além da solução atual?</span><span class="sxs-lookup"><span data-stu-id="1a9ce-116">Are other PaaS deployments planned beyond the current solution?</span></span>
- <span data-ttu-id="1a9ce-117">A organização tem planos para outras migrações de nuvem futuras?</span><span class="sxs-lookup"><span data-stu-id="1a9ce-117">Does the organization have plans for other future cloud migrations?</span></span>

<span data-ttu-id="1a9ce-118">As respostas a essas perguntas não impediriam que uma equipe escolhesse uma única opção PaaS, mas deveriam ser considerada antes de tomar uma decisão final.</span><span class="sxs-lookup"><span data-stu-id="1a9ce-118">The answers to these questions would not preclude a team from choosing a PaaS only option but should be considered before making a final decision.</span></span>
