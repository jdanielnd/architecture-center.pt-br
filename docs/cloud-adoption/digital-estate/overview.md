---
title: 'CAF: O que é uma propriedade digital?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
description: O que é uma propriedade digital?
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: d23bdbdd98226e8b9cdb9bbb5fefa5a918a498d8
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241647"
---
<!-- markdownlint-disable MD026 -->

# <a name="caf-what-is-a-digital-estate"></a><span data-ttu-id="d8ab5-103">CAF: O que é uma propriedade digital?</span><span class="sxs-lookup"><span data-stu-id="d8ab5-103">CAF: What is a digital estate?</span></span>

<span data-ttu-id="d8ab5-104">Todas as empresas modernas têm alguma forma de propriedade digital.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-104">Every modern company has some form of digital estate.</span></span> <span data-ttu-id="d8ab5-105">Assim como uma propriedade física, uma propriedade digital é uma referência abstrata a um número de ativos de propriedade tangíveis.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-105">Much like a physical estate, a digital estate is an abstract reference to a number of tangible, owned assets.</span></span> <span data-ttu-id="d8ab5-106">Em uma propriedade digital, esses ativos são compostos por VMs (máquinas virtuais), servidores, aplicativos, dados e assim por diante.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-106">In a digital estate, those assets are comprised of virtual machines (VMs), servers, applications, data, and so on.</span></span> <span data-ttu-id="d8ab5-107">Essencialmente, uma propriedade digital é a coleção de ativos de TI que capacitam os processos de negócios e operações de suporte.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-107">Essentially, a digital estate is the collection of IT assets that power business processes and supporting operations.</span></span>

<span data-ttu-id="d8ab5-108">A importância de uma propriedade digital é mais óbvia durante o planejamento e a execução de esforços de Transformação Digital.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-108">The importance of a digital estate is most obvious during planning and execution of Digital Transformation efforts.</span></span> <span data-ttu-id="d8ab5-109">Durante jornadas de transformação, é na propriedade digital que as equipes de Estratégia de Nuvem mapeiam os resultados dos negócios para planos de lançamento e esforços técnicos.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-109">During transformation journeys, the digital estate is how Cloud Strategy teams map the business outcomes to release plans and technical efforts.</span></span> <span data-ttu-id="d8ab5-110">Tudo começa com um inventário e a medição dos ativos digitais pertencentes à organização atualmente.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-110">That all starts with an inventory and measurement of the digital assets the organization owns today.</span></span>

## <a name="how-can-a-digital-estate-be-measured"></a><span data-ttu-id="d8ab5-111">Como é possível medir uma propriedade digital?</span><span class="sxs-lookup"><span data-stu-id="d8ab5-111">How can a digital estate be measured?</span></span>

<span data-ttu-id="d8ab5-112">A medição de uma propriedade digital muda dependendo dos resultados comerciais desejados.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-112">The measurement of a digital estate changes depending on the desired business outcomes.</span></span>

- <span data-ttu-id="d8ab5-113">**Migrações de infraestrutura**.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-113">**Infrastructure migrations**.</span></span> <span data-ttu-id="d8ab5-114">Quando uma organização volta a atenção para dentro e busca otimizar o custo, os processos operacionais, a agilidade ou outros aspectos das operações, a propriedade digital se concentra em VMs, servidores e cargas de trabalho com suporte.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-114">When an organization is inward facing and seeks to optimize cost, operational processes, agility, or other aspects of optimizing operations, the digital estate focuses on VMs, servers, and supporting workloads.</span></span>

- <span data-ttu-id="d8ab5-115">**Inovação de aplicativo**.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-115">**Application innovation**.</span></span> <span data-ttu-id="d8ab5-116">Para transformações voltadas ao cliente, a perspectiva é um pouco diferente.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-116">For customer-obsessed transformations, the lens is a bit different.</span></span> <span data-ttu-id="d8ab5-117">O foco deve estar nos aplicativos, nas APIs e nos dados transacionais que dão suporte aos clientes.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-117">The focus should be placed in the applications, APIs, and transactional data that support the customers.</span></span> <span data-ttu-id="d8ab5-118">VMs e dispositivos de rede costumam receber menos foco.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-118">VMs and network appliances are often of less focus.</span></span>

- <span data-ttu-id="d8ab5-119">**Inovação controlada por dados**.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-119">**Data-driven innovation**.</span></span> <span data-ttu-id="d8ab5-120">No mercado digitalmente orientado de hoje em dia, é difícil lançar um novo produto ou serviço sem uma base sólida de dados.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-120">In today's digitally driven market, it's difficult to launch a new product or service without a strong foundation in data.</span></span> <span data-ttu-id="d8ab5-121">Durante a transformação interruptiva, o foco fica mais nos silos de dados por toda a organização.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-121">During disruptive transformation, the focus is more on the silos of data across the organization.</span></span>

<span data-ttu-id="d8ab5-122">Quando uma organização compreende a forma mais importante de transformação, o planejamento de propriedade digital fica muito mais fácil de gerenciar.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-122">Once an organization understands the most important form of transformation, digital estate planning becomes much easier to manage.</span></span>

> [!TIP]
> <span data-ttu-id="d8ab5-123">Cada tipo de transformação pode ser medido com qualquer um dos três modos de exibição.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-123">Each type of transformation could be measured with any of the three views.</span></span> <span data-ttu-id="d8ab5-124">Além disso, é comum as empresas executarem as três transformações em paralelo.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-124">Further, it is common for companies to execute all three transformations in parallel.</span></span> <span data-ttu-id="d8ab5-125">Sugerimos que a liderança e a equipe de Estratégia de Nuvem fiquem bem alinhadas com relação à transformação mais importante para o sucesso dos negócios.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-125">It is highly suggested that the leadership and Cloud Strategy team be firmly aligned regarding the transformation that is most important for business success.</span></span> <span data-ttu-id="d8ab5-126">Essa compreensão servirá como base para a linguagem e métricas comum entre várias iniciativas.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-126">That understanding will serve as the basis for common language and metrics across multiple initiatives.</span></span>

## <a name="how-can-a-financial-model-be-updated-to-reflect-the-digital-estate"></a><span data-ttu-id="d8ab5-127">Como um modelo financeiro pode ser atualizado para refletir a propriedade digital?</span><span class="sxs-lookup"><span data-stu-id="d8ab5-127">How can a financial model be updated to reflect the digital estate?</span></span>

<span data-ttu-id="d8ab5-128">Uma análise da propriedade digital determinará as atividades de adoção de nuvem.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-128">An analysis of the digital estate will drive cloud adoption activities.</span></span> <span data-ttu-id="d8ab5-129">Você também informará os modelos financeiros fornecendo modelos de custo de nuvem, o que determinará, por sua vez, o ROI (retorno sobre o investimento).</span><span class="sxs-lookup"><span data-stu-id="d8ab5-129">It will also inform financial models by providing cloud costing models, which will in turn drive the return on investment (ROI).</span></span>

<span data-ttu-id="d8ab5-130">Para concluir a análise de propriedade digital, execute as seguintes etapas:</span><span class="sxs-lookup"><span data-stu-id="d8ab5-130">To complete the digital estate analysis, perform the following steps:</span></span>

1. [<span data-ttu-id="d8ab5-131">Determine a abordagem de análise</span><span class="sxs-lookup"><span data-stu-id="d8ab5-131">Determine analysis approach</span></span>](approach.md)
1. [<span data-ttu-id="d8ab5-132">Colete o inventário do estado atual</span><span class="sxs-lookup"><span data-stu-id="d8ab5-132">Collect current state inventory</span></span>](inventory.md)
1. [<span data-ttu-id="d8ab5-133">Racionalizar os ativos na propriedade digital</span><span class="sxs-lookup"><span data-stu-id="d8ab5-133">Rationalize the assets in the digital estate</span></span>](rationalize.md)
1. [<span data-ttu-id="d8ab5-134">Alinhar ativos às ofertas de nuvem para calcular o preço</span><span class="sxs-lookup"><span data-stu-id="d8ab5-134">Align assets to cloud offerings to calculate pricing</span></span>](calculate.md)

<span data-ttu-id="d8ab5-135">Modelos financeiros e listas de pendências de migração podem ser modificadas para refletir a propriedade racionalizada e precificada.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-135">Financial models and migration backlogs can be modified to reflect the rationalized and priced estate.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d8ab5-136">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="d8ab5-136">Next steps</span></span>

<span data-ttu-id="d8ab5-137">Antes de o planejamento de propriedade digital poder começar, você deve determinar a abordagem a ser usada.</span><span class="sxs-lookup"><span data-stu-id="d8ab5-137">Before digital estate planning can begin, you must determine what approach to use.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="d8ab5-138">Abordagens para planejamento de propriedade digital</span><span class="sxs-lookup"><span data-stu-id="d8ab5-138">Approaches to digital estate planning</span></span>](approach.md)