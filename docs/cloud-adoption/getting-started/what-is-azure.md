---
title: 'CAF: Como funciona o Azure?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
description: Explicação sobre o funcionamento interno do Azure
author: petertaylor9999
ms.date: 02/11/2019
ms.openlocfilehash: 724d16a810865dd947a7ade34766818c8ea525a1
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245267"
---
<!-- markdownlint-disable MD026 -->

# <a name="how-does-azure-work"></a><span data-ttu-id="fb2ea-103">Como funciona o Azure?</span><span class="sxs-lookup"><span data-stu-id="fb2ea-103">How does Azure work?</span></span>

<span data-ttu-id="fb2ea-104">O Azure é a plataforma de nuvem pública da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-104">Azure is Microsoft's public cloud platform.</span></span> <span data-ttu-id="fb2ea-105">O Azure oferece uma grande coleção de serviços, incluindo PaaS (plataforma como serviço), IaaS (infraestrutura como serviço), DBaaS (banco de dados como serviço) e muitos outros.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-105">Azure offers a large collection of services including platform as a service (PaaS), infrastructure as a service (IaaS), database as a service (DBaaS), and many others.</span></span> <span data-ttu-id="fb2ea-106">Mas o que exatamente é o Azure e como ele funciona?</span><span class="sxs-lookup"><span data-stu-id="fb2ea-106">But what exactly is Azure, and how does it work?</span></span>

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ixGo]

<!-- markdownlint-enable MD034 -->

<span data-ttu-id="fb2ea-107">O Azure, como outras plataformas de nuvem, se baseia em uma tecnologia conhecida como **virtualização**.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-107">Azure, like other cloud platforms, relies on a technology known as **virtualization**.</span></span> <span data-ttu-id="fb2ea-108">A maior parte do hardware do computador pode ser emulada em software, porque a maior parte dele é apenas um conjunto de instruções codificado de forma permanente ou semipermanente em silício.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-108">Most computer hardware can be emulated in software, because most computer hardware is simply a set of instructions permanently or semi-permanently encoded in silicon.</span></span> <span data-ttu-id="fb2ea-109">Usando uma camada de emulação que mapeia instruções de software para instruções de hardware, o hardware virtualizado pode ser executado em um software como se fosse o próprio hardware real.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-109">Using an emulation layer that maps software instructions to hardware instructions, virtualized hardware can execute in software as if it were the actual hardware itself.</span></span>

<span data-ttu-id="fb2ea-110">Basicamente, a nuvem é um conjunto de servidores físicos em um ou mais datacenters que executam o hardware virtualizado em nome dos clientes.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-110">Essentially, the cloud is a set of physical servers in one or more datacenters that execute virtualized hardware on behalf of customers.</span></span> <span data-ttu-id="fb2ea-111">Então, como a nuvem cria, inicia, interrompe e exclui milhões de instâncias de hardware virtualizado para milhões de clientes simultaneamente?</span><span class="sxs-lookup"><span data-stu-id="fb2ea-111">So how does the cloud create, start, stop, and delete millions of instances of virtualized hardware for millions of customers simultaneously?</span></span>

<span data-ttu-id="fb2ea-112">Para entender isso, vamos examinar a arquitetura do hardware no datacenter.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-112">To understand this, let's look at the architecture of the hardware in the datacenter.</span></span>  <span data-ttu-id="fb2ea-113">Em cada datacenter existe uma coleção de servidores que residem em racks de servidor.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-113">Within each datacenter is a collection of servers sitting in server racks.</span></span> <span data-ttu-id="fb2ea-114">Cada rack de servidor contém muitas **folhas** de servidor, bem como um comutador de rede que fornece conectividade de rede e uma PDU (unidade de distribuição de energia) que fornece energia.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-114">Each server rack contains many server **blades** as well as a network switch providing network connectivity and a power distribution unit (PDU) providing power.</span></span> <span data-ttu-id="fb2ea-115">Às vezes, os racks são agrupados em unidades maiores conhecidas como **clusters**.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-115">Racks are sometimes grouped together in larger units known as **clusters**.</span></span>

<span data-ttu-id="fb2ea-116">Em cada rack ou cluster, a maioria dos servidores é designada para executar essas instâncias de hardware virtualizado em nome do usuário.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-116">Within each rack or cluster, most of the servers are designated to run these virtualized hardware instances on behalf of the user.</span></span> <span data-ttu-id="fb2ea-117">No entanto, vários dos servidores executam um software de gerenciamento da nuvem conhecido como um controlador de malha.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-117">However, a number of the servers run cloud management software known as a fabric controller.</span></span> <span data-ttu-id="fb2ea-118">O **controlador de malha** é um aplicativo distribuído com muitas responsabilidades.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-118">The **fabric controller** is a distributed application with many responsibilities.</span></span> <span data-ttu-id="fb2ea-119">Ele aloca serviços, monitora a integridade do servidor e dos serviços em execução nele e restaura servidores quando ocorre uma falha.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-119">It allocates services, monitors the health of the server and the services running on it, and heals servers when they fail.</span></span>

<span data-ttu-id="fb2ea-120">Cada instância do controlador de malha está conectada a outro conjunto de servidores que executam o software de orquestração da nuvem, normalmente conhecido como um **front-end**.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-120">Each instance of the fabric controller is connected to another set of servers running cloud orchestration software, typically known as a **front end**.</span></span> <span data-ttu-id="fb2ea-121">O front-end hospeda os serviços Web, as APIs RESTful e os bancos de dados internos do Azure usados para todas as funções executadas pela nuvem.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-121">The front end hosts the web services, RESTful APIs, and internal Azure databases used for all functions the cloud performs.</span></span>

<span data-ttu-id="fb2ea-122">Por exemplo, o front-end hospeda os serviços que manipulam as solicitações de cliente para alocar recursos do Azure como [redes virtuais][vnet], [máquinas virtuais][vms] e serviços como o [Cosmos DB][cosmosdb].</span><span class="sxs-lookup"><span data-stu-id="fb2ea-122">For example, the front end hosts the services that handle customer requests to allocate Azure resources such as [virtual networks][vnet], [virtual machines][vms], and services like [Cosmos DB][cosmosdb].</span></span> <span data-ttu-id="fb2ea-123">Primeiro, o front-end valida o usuário e verifica se o usuário está autorizado a alocar os recursos solicitados.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-123">First, the front end validates the user and verifies the user is authorized to allocate the requested resources.</span></span> <span data-ttu-id="fb2ea-124">Nesse caso, o front-end consulta um banco de dados para localizar um rack de servidor com capacidade suficiente e, em seguida, instrui o controlador de malha no rack a alocar o recurso.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-124">If so, the front end consults a database to locate a server rack with sufficient capacity, and then instructs the fabric controller on the rack to allocate the resource.</span></span>

<span data-ttu-id="fb2ea-125">Portanto, em termos simples, o Azure é uma grande coleção de servidores e hardware de rede, junto com um conjunto complexo de aplicativos distribuídos que orquestra a configuração e operação do software e hardware virtualizado nesses servidores.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-125">So, very simply, Azure is a huge collection of servers and networking hardware, along with a complex set of distributed applications that orchestrate the configuration and operation of the virtualized hardware and software on those servers.</span></span> <span data-ttu-id="fb2ea-126">E é essa orquestração que faz o Azure tão eficaz – os usuários deixam de ser responsáveis por manter e atualizar o hardware, pois o Azure faz tudo isso em segundo plano.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-126">And it is this orchestration that makes Azure so powerful - users are no longer responsible for maintaining and upgrading hardware, Azure does all this behind the scenes.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fb2ea-127">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="fb2ea-127">Next steps</span></span>

<span data-ttu-id="fb2ea-128">Agora que você já entende o funcionamento interno do Azure, saiba mais sobre a governança de recursos da nuvem.</span><span class="sxs-lookup"><span data-stu-id="fb2ea-128">Now that you understand the internal functioning of Azure, learn about cloud resource governance.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="fb2ea-129">Saiba mais sobre governança de recursos</span><span class="sxs-lookup"><span data-stu-id="fb2ea-129">Learn about resource governance</span></span>](what-is-governance.md)

<!-- Links -->

[cosmosdb]: /azure/cosmos-db/introduction
[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[vms]: /azure/virtual-machines/
[vnet]: /azure/virtual-network/virtual-networks-overview
