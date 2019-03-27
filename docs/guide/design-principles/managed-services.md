---
title: Usar serviços gerenciados
titleSuffix: Azure Application Architecture Guide
description: Quando possível, utilize a plataforma como um serviço (PaaS) em vez da infraestrutura como um serviço (IaaS).
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: f08914e1eacc4f02fdb16093fe590f46249e3da3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249531"
---
# <a name="use-managed-services"></a><span data-ttu-id="81cc4-103">Usar serviços gerenciados</span><span class="sxs-lookup"><span data-stu-id="81cc4-103">Use managed services</span></span>

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a><span data-ttu-id="81cc4-104">Quando possível, utilize a plataforma como um serviço (PaaS) em vez de infraestrutura como serviço (IaaS)</span><span class="sxs-lookup"><span data-stu-id="81cc4-104">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS)</span></span>

<span data-ttu-id="81cc4-105">O IaaS é como ter uma caixa de peças.</span><span class="sxs-lookup"><span data-stu-id="81cc4-105">IaaS is like having a box of parts.</span></span> <span data-ttu-id="81cc4-106">Você pode criar qualquer coisa, mas deve montar por conta própria.</span><span class="sxs-lookup"><span data-stu-id="81cc4-106">You can build anything, but you have to assemble it yourself.</span></span> <span data-ttu-id="81cc4-107">Os serviços gerenciados são mais fáceis de configurar e de administrar.</span><span class="sxs-lookup"><span data-stu-id="81cc4-107">Managed services are easier to configure and administer.</span></span> <span data-ttu-id="81cc4-108">Você não precisa provisionar máquinas virtuais, configurar VNets, gerenciar atualizações e patches e toda a outra sobrecarga associada à execução de software em uma máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="81cc4-108">You don't need to provision VMs, set up VNets, manage patches and updates, and all of the other overhead associated with running software on a VM.</span></span>

<span data-ttu-id="81cc4-109">Por exemplo, suponha que seu aplicativo precisa de uma fila de mensagens.</span><span class="sxs-lookup"><span data-stu-id="81cc4-109">For example, suppose your application needs a message queue.</span></span> <span data-ttu-id="81cc4-110">Você pode configurar seu próprio serviço de mensagens em uma VM, algo como o RabbitMQ.</span><span class="sxs-lookup"><span data-stu-id="81cc4-110">You could set up your own messaging service on a VM, using something like RabbitMQ.</span></span> <span data-ttu-id="81cc4-111">Mas o Barramento de Serviço do Azure já fornece mensagens confiáveis como serviço, e é mais simples de configurar.</span><span class="sxs-lookup"><span data-stu-id="81cc4-111">But Azure Service Bus already provides reliable messaging as service, and it's simpler to set up.</span></span> <span data-ttu-id="81cc4-112">Basta criar um namespace do Barramento de Serviço (o que pode ser feito como parte de um script de implantação) e então chame o Barramento de Serviço usando o SDK do cliente.</span><span class="sxs-lookup"><span data-stu-id="81cc4-112">Just create a Service Bus namespace (which can be done as part of a deployment script) and then call Service Bus using the client SDK.</span></span>

<span data-ttu-id="81cc4-113">Obviamente, seu aplicativo pode ter requisitos específicos que tornam uma abordagem IaaS mais adequada.</span><span class="sxs-lookup"><span data-stu-id="81cc4-113">Of course, your application may have specific requirements that make an IaaS approach more suitable.</span></span> <span data-ttu-id="81cc4-114">No entanto, mesmo se seu aplicativo se basear em IaaS, procure locais onde pode ser natural incorporar serviços gerenciados.</span><span class="sxs-lookup"><span data-stu-id="81cc4-114">However, even if your application is based on IaaS, look for places where it may be natural to incorporate managed services.</span></span> <span data-ttu-id="81cc4-115">Isso inclui o armazenamento de dados, filas e cache.</span><span class="sxs-lookup"><span data-stu-id="81cc4-115">These include cache, queues, and data storage.</span></span>

| <span data-ttu-id="81cc4-116">Em vez de executar...</span><span class="sxs-lookup"><span data-stu-id="81cc4-116">Instead of running...</span></span> | <span data-ttu-id="81cc4-117">Considere usar...</span><span class="sxs-lookup"><span data-stu-id="81cc4-117">Consider using...</span></span> |
|-----------------------|-------------|
| <span data-ttu-id="81cc4-118">Active Directory</span><span class="sxs-lookup"><span data-stu-id="81cc4-118">Active Directory</span></span> | <span data-ttu-id="81cc4-119">Azure Active Directory Domain Services</span><span class="sxs-lookup"><span data-stu-id="81cc4-119">Azure Active Directory Domain Services</span></span> |
| <span data-ttu-id="81cc4-120">Elasticsearch</span><span class="sxs-lookup"><span data-stu-id="81cc4-120">Elasticsearch</span></span> | <span data-ttu-id="81cc4-121">Azure Search</span><span class="sxs-lookup"><span data-stu-id="81cc4-121">Azure Search</span></span> |
| <span data-ttu-id="81cc4-122">O Hadoop</span><span class="sxs-lookup"><span data-stu-id="81cc4-122">Hadoop</span></span> | <span data-ttu-id="81cc4-123">HDInsight</span><span class="sxs-lookup"><span data-stu-id="81cc4-123">HDInsight</span></span> |
| <span data-ttu-id="81cc4-124">IIS</span><span class="sxs-lookup"><span data-stu-id="81cc4-124">IIS</span></span> | <span data-ttu-id="81cc4-125">Serviço de Aplicativo</span><span class="sxs-lookup"><span data-stu-id="81cc4-125">App Service</span></span> |
| <span data-ttu-id="81cc4-126">MongoDB</span><span class="sxs-lookup"><span data-stu-id="81cc4-126">MongoDB</span></span> | <span data-ttu-id="81cc4-127">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="81cc4-127">Cosmos DB</span></span> |
| <span data-ttu-id="81cc4-128">Redis</span><span class="sxs-lookup"><span data-stu-id="81cc4-128">Redis</span></span> | <span data-ttu-id="81cc4-129">Cache Redis do Azure</span><span class="sxs-lookup"><span data-stu-id="81cc4-129">Azure Redis Cache</span></span> |
| <span data-ttu-id="81cc4-130">SQL Server</span><span class="sxs-lookup"><span data-stu-id="81cc4-130">SQL Server</span></span> | <span data-ttu-id="81cc4-131">Banco de Dados SQL do Azure</span><span class="sxs-lookup"><span data-stu-id="81cc4-131">Azure SQL Database</span></span> |
