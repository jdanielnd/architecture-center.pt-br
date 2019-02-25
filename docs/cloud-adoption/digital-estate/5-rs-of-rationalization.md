---
title: 'CAF: 5 Rs da racionalização'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Examine as opções disponíveis para racionalizar um espaço digital.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.openlocfilehash: ee196487e6f59b1e1b3c63bab9496cbbf805affd
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897466"
---
# <a name="the-5-rs-of-rationalization"></a><span data-ttu-id="ac261-103">5 Rs da racionalização</span><span class="sxs-lookup"><span data-stu-id="ac261-103">The 5 Rs of rationalization</span></span>

<span data-ttu-id="ac261-104">Racionalização de nuvem é o processo de avaliação de ativos para determinar a melhor abordagem para a migração ou modernização de cada ativo na nuvem.</span><span class="sxs-lookup"><span data-stu-id="ac261-104">Cloud Rationalization is the process of evaluating assets to determine the best approach to migrating or modernizing each asset in the cloud.</span></span> <span data-ttu-id="ac261-105">Para saber mais sobre o processo de racionalização, confira [O que é uma propriedade digital?](overview.md)</span><span class="sxs-lookup"><span data-stu-id="ac261-105">For more information about the process of rationalization, see [What is a digital estate?](overview.md)</span></span>

<span data-ttu-id="ac261-106">O "5 Rs da racionalização" listados aqui descrevem as opções mais comuns de racionalização.</span><span class="sxs-lookup"><span data-stu-id="ac261-106">The "5 Rs of rationalization" listed here describe the most common options for rationalization.</span></span>

## <a name="rehost"></a><span data-ttu-id="ac261-107">Hospedar novamente</span><span class="sxs-lookup"><span data-stu-id="ac261-107">Rehost</span></span>

<span data-ttu-id="ac261-108">Também conhecido como "lift and shift", um esforço de nova hospedagem move o ativo de estado atual para o provedor de nuvem escolhido, com alterações mínimas na arquitetura geral.</span><span class="sxs-lookup"><span data-stu-id="ac261-108">Also known as "lift and shift," a rehost effort moves the current state asset to the chosen cloud provider, with minimal change to overall architecture.</span></span>

<span data-ttu-id="ac261-109">Os motivadores comuns podem incluir:</span><span class="sxs-lookup"><span data-stu-id="ac261-109">Common drivers could include:</span></span>

* <span data-ttu-id="ac261-110">Reduzir CapEx</span><span class="sxs-lookup"><span data-stu-id="ac261-110">Reduce CapEx</span></span>
* <span data-ttu-id="ac261-111">Liberar espaço no datacenter</span><span class="sxs-lookup"><span data-stu-id="ac261-111">Free up datacenter space</span></span>
* <span data-ttu-id="ac261-112">ROI de nuvem rápido</span><span class="sxs-lookup"><span data-stu-id="ac261-112">Quick cloud ROI</span></span>

<span data-ttu-id="ac261-113">Fatores de análise quantitativa:</span><span class="sxs-lookup"><span data-stu-id="ac261-113">Quantitative analysis factors:</span></span>

* <span data-ttu-id="ac261-114">Tamanho da VM (CPU, memória, armazenamento)</span><span class="sxs-lookup"><span data-stu-id="ac261-114">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="ac261-115">Dependências (tráfego de rede)</span><span class="sxs-lookup"><span data-stu-id="ac261-115">Dependencies (network traffic)</span></span>
* <span data-ttu-id="ac261-116">Compatibilidade de ativo</span><span class="sxs-lookup"><span data-stu-id="ac261-116">Asset compatibility</span></span>

<span data-ttu-id="ac261-117">Fatores de análise qualitativa:</span><span class="sxs-lookup"><span data-stu-id="ac261-117">Qualitative analysis factors:</span></span>

* <span data-ttu-id="ac261-118">Tolerância à alteração</span><span class="sxs-lookup"><span data-stu-id="ac261-118">Tolerance for change</span></span>
* <span data-ttu-id="ac261-119">Prioridades dos negócios</span><span class="sxs-lookup"><span data-stu-id="ac261-119">Business priorities</span></span>
* <span data-ttu-id="ac261-120">Eventos comerciais críticos</span><span class="sxs-lookup"><span data-stu-id="ac261-120">Critical business events</span></span>
* <span data-ttu-id="ac261-121">Dependências de processo</span><span class="sxs-lookup"><span data-stu-id="ac261-121">Process dependencies</span></span>

## <a name="refactor"></a><span data-ttu-id="ac261-122">Refatoração</span><span class="sxs-lookup"><span data-stu-id="ac261-122">Refactor</span></span>

<span data-ttu-id="ac261-123">As opções de PaaS (Plataforma como serviço) podem reduzir os custos operacionais associados a muitos aplicativos.</span><span class="sxs-lookup"><span data-stu-id="ac261-123">Platform as a Service (PaaS) options can reduce operational costs associated with many applications.</span></span> <span data-ttu-id="ac261-124">Pode ser prudente refatorar ligeiramente um aplicativo a fim de ajustá-lo a um modelo com base em PaaS.</span><span class="sxs-lookup"><span data-stu-id="ac261-124">It can be prudent to slightly refactor an application to fit a PaaS based model.</span></span>

<span data-ttu-id="ac261-125">Refatorar também se refere ao processo de desenvolvimento de aplicativo de refatoração de código, a fim de permitir que um aplicativo forneça novas oportunidades de negócios.</span><span class="sxs-lookup"><span data-stu-id="ac261-125">Refactor also refers to the application development process of refactoring code to allow an application to deliver on new business opportunities.</span></span>

<span data-ttu-id="ac261-126">Os motivadores comuns podem incluir:</span><span class="sxs-lookup"><span data-stu-id="ac261-126">Common drivers could include:</span></span>

* <span data-ttu-id="ac261-127">Atualizações mais rápidas e curtas</span><span class="sxs-lookup"><span data-stu-id="ac261-127">Faster, shorter, updates</span></span>
* <span data-ttu-id="ac261-128">Portabilidade do código</span><span class="sxs-lookup"><span data-stu-id="ac261-128">Code portability</span></span>
* <span data-ttu-id="ac261-129">Maior eficiência de nuvem (recursos, velocidade e custo)</span><span class="sxs-lookup"><span data-stu-id="ac261-129">Greater cloud efficiency (resources, speed, cost)</span></span>

<span data-ttu-id="ac261-130">Fatores de análise quantitativa:</span><span class="sxs-lookup"><span data-stu-id="ac261-130">Quantitative analysis factors:</span></span>

* <span data-ttu-id="ac261-131">Tamanho do ativo de aplicativo (CPU, memória, armazenamento)</span><span class="sxs-lookup"><span data-stu-id="ac261-131">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="ac261-132">Dependências (tráfego de rede)</span><span class="sxs-lookup"><span data-stu-id="ac261-132">Dependencies (network traffic)</span></span>
* <span data-ttu-id="ac261-133">Tráfego do usuário (exibições de página, hora na página, tempo de carregamento)</span><span class="sxs-lookup"><span data-stu-id="ac261-133">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="ac261-134">Plataforma de desenvolvimento (linguagens, plataforma de dados, serviços de camada intermediária)</span><span class="sxs-lookup"><span data-stu-id="ac261-134">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="ac261-135">Fatores de análise qualitativa:</span><span class="sxs-lookup"><span data-stu-id="ac261-135">Qualitative analysis factors:</span></span>

* <span data-ttu-id="ac261-136">Investimento contínuo nos negócios</span><span class="sxs-lookup"><span data-stu-id="ac261-136">Continued business investments</span></span>
* <span data-ttu-id="ac261-137">Opções/linhas do tempo de intermitência</span><span class="sxs-lookup"><span data-stu-id="ac261-137">Bursting options/timelines</span></span>
* <span data-ttu-id="ac261-138">Dependências de processo comercial</span><span class="sxs-lookup"><span data-stu-id="ac261-138">Business process dependencies</span></span>

## <a name="rearchitect"></a><span data-ttu-id="ac261-139">Recriação da arquitetura</span><span class="sxs-lookup"><span data-stu-id="ac261-139">Rearchitect</span></span>

<span data-ttu-id="ac261-140">Alguns aplicativos mais velhos não são compatíveis com provedores de nuvem devido a decisões arquitetônicas tomadas durante a compilação do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="ac261-140">Some aging applications aren't compatible with cloud providers because of the architectural decisions made when the application was built.</span></span> <span data-ttu-id="ac261-141">Nesses casos, o aplicativo pode precisar de uma nova arquitetura antes da transformação.</span><span class="sxs-lookup"><span data-stu-id="ac261-141">In these cases, the application may need to be rearchitected prior to transformation.</span></span>

<span data-ttu-id="ac261-142">Em outros casos, os aplicativos que são compatíveis com a nuvem, mas que não conta com benefícios por serem nativos, podem melhorar o custo e as operações por meio da nova arquitetura da solução, a fim de se tornarem um aplicativo nativo de nuvem.</span><span class="sxs-lookup"><span data-stu-id="ac261-142">In other cases, applications that are cloud compatible, but not cloud native benefits, may produce cost efficiencies and operational efficiencies by rearchitecting the solution to be a cloud native application.</span></span>

<span data-ttu-id="ac261-143">Os motivadores comuns podem incluir:</span><span class="sxs-lookup"><span data-stu-id="ac261-143">Common drivers could include:</span></span>

* <span data-ttu-id="ac261-144">Agilidade e escala do aplicativo</span><span class="sxs-lookup"><span data-stu-id="ac261-144">Application scale and agility</span></span>
* <span data-ttu-id="ac261-145">Facilidade da adoção de novos recursos de nuvem</span><span class="sxs-lookup"><span data-stu-id="ac261-145">Easier adoption of new cloud capabilities</span></span>
* <span data-ttu-id="ac261-146">Combinação de pilhas de tecnologia</span><span class="sxs-lookup"><span data-stu-id="ac261-146">Mix of technology stacks</span></span>

<span data-ttu-id="ac261-147">Fatores de análise quantitativa:</span><span class="sxs-lookup"><span data-stu-id="ac261-147">Quantitative analysis factors:</span></span>

* <span data-ttu-id="ac261-148">Tamanho do ativo de aplicativo (CPU, memória, armazenamento)</span><span class="sxs-lookup"><span data-stu-id="ac261-148">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="ac261-149">Dependências (tráfego de rede)</span><span class="sxs-lookup"><span data-stu-id="ac261-149">Dependencies (network traffic)</span></span>
* <span data-ttu-id="ac261-150">Tráfego do usuário (exibições de página, hora na página, tempo de carregamento)</span><span class="sxs-lookup"><span data-stu-id="ac261-150">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="ac261-151">Plataforma de desenvolvimento (linguagens, plataforma de dados, serviços de camada intermediária)</span><span class="sxs-lookup"><span data-stu-id="ac261-151">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="ac261-152">Fatores de análise qualitativa:</span><span class="sxs-lookup"><span data-stu-id="ac261-152">Qualitative analysis factors:</span></span>

* <span data-ttu-id="ac261-153">Investimentos comerciais em expansão</span><span class="sxs-lookup"><span data-stu-id="ac261-153">Growing business investments</span></span>
* <span data-ttu-id="ac261-154">Custos operacionais</span><span class="sxs-lookup"><span data-stu-id="ac261-154">Operational costs</span></span>
* <span data-ttu-id="ac261-155">Possíveis loops de comentários e investimentos de DevOps</span><span class="sxs-lookup"><span data-stu-id="ac261-155">Potential feedback loops and DevOps investments</span></span>

## <a name="rebuild"></a><span data-ttu-id="ac261-156">Recompilação</span><span class="sxs-lookup"><span data-stu-id="ac261-156">Rebuild</span></span>

<span data-ttu-id="ac261-157">Em alguns cenários, o delta que deve ser superado para avançar com um aplicativo pode ser muito grande para justificar o investimento.</span><span class="sxs-lookup"><span data-stu-id="ac261-157">In some scenarios, the delta that must be overcome to carry forward an application can be too large to justify further investment.</span></span> <span data-ttu-id="ac261-158">Isso é especialmente verdadeiro para aplicativos que costumavam atender às necessidades do negócio, mas que agora não têm suporte ou são não estão alinhados com o modo como os processos comerciais são executados atualmente.</span><span class="sxs-lookup"><span data-stu-id="ac261-158">This is especially true for applications that used to meet the needs of the business, but are now unsupported or misaligned with how the business processes are executed today.</span></span> <span data-ttu-id="ac261-159">Nesse caso, uma nova base de código é criada de acordo com uma abordagem de [nuvem nativa](https://azure.microsoft.com/overview/cloudnative/).</span><span class="sxs-lookup"><span data-stu-id="ac261-159">In this case, a new code base is created to align with a [cloud native](https://azure.microsoft.com/overview/cloudnative/) approach.</span></span>

<span data-ttu-id="ac261-160">Os motivadores comuns podem incluir:</span><span class="sxs-lookup"><span data-stu-id="ac261-160">Common drivers could include:</span></span>

* <span data-ttu-id="ac261-161">Acelerar a inovação</span><span class="sxs-lookup"><span data-stu-id="ac261-161">Accelerate innovation</span></span>
* <span data-ttu-id="ac261-162">Compilar aplicativos mais rápido</span><span class="sxs-lookup"><span data-stu-id="ac261-162">Build apps faster</span></span>
* <span data-ttu-id="ac261-163">Reduzir o custo operacional</span><span class="sxs-lookup"><span data-stu-id="ac261-163">Reduce operational cost</span></span>

<span data-ttu-id="ac261-164">Fatores de análise quantitativa:</span><span class="sxs-lookup"><span data-stu-id="ac261-164">Quantitative analysis factors:</span></span>

* <span data-ttu-id="ac261-165">Tamanho do ativo de aplicativo (CPU, memória, armazenamento)</span><span class="sxs-lookup"><span data-stu-id="ac261-165">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="ac261-166">Dependências (tráfego de rede)</span><span class="sxs-lookup"><span data-stu-id="ac261-166">Dependencies (network traffic)</span></span>
* <span data-ttu-id="ac261-167">Tráfego do usuário (exibições de página, hora na página, tempo de carregamento)</span><span class="sxs-lookup"><span data-stu-id="ac261-167">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="ac261-168">Plataforma de desenvolvimento (linguagens, plataforma de dados, serviços de camada intermediária)</span><span class="sxs-lookup"><span data-stu-id="ac261-168">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="ac261-169">Fatores de análise qualitativa:</span><span class="sxs-lookup"><span data-stu-id="ac261-169">Qualitative analysis Factors:</span></span>

* <span data-ttu-id="ac261-170">Satisfação do usuário final em queda</span><span class="sxs-lookup"><span data-stu-id="ac261-170">Declining end user satisfaction</span></span>
* <span data-ttu-id="ac261-171">Processos de negócio limitados pela funcionalidade</span><span class="sxs-lookup"><span data-stu-id="ac261-171">Business processes limited by functionality</span></span>
* <span data-ttu-id="ac261-172">Possíveis ganhos de custo, experiência ou receita</span><span class="sxs-lookup"><span data-stu-id="ac261-172">Potential cost, experience, or revenue gains</span></span>

## <a name="replace"></a><span data-ttu-id="ac261-173">Substitua</span><span class="sxs-lookup"><span data-stu-id="ac261-173">Replace</span></span>

<span data-ttu-id="ac261-174">Normalmente, as soluções são implementadas usando a melhor tecnologia e abordagem disponíveis no momento.</span><span class="sxs-lookup"><span data-stu-id="ac261-174">Solutions are generally implemented using the best technology and approach available at the time.</span></span> <span data-ttu-id="ac261-175">Em alguns casos, aplicativos SaaS (Software como serviço) podem atender a todas as funcionalidades necessárias do aplicativo hospedado.</span><span class="sxs-lookup"><span data-stu-id="ac261-175">In some cases, Software as a Service (SaaS) applications can meet all of the functionality required of the hosted application.</span></span> <span data-ttu-id="ac261-176">Nesses cenários, uma carga de trabalho pode ser programada para substituição futura, removendo-a efetivamente do esforço de transformação.</span><span class="sxs-lookup"><span data-stu-id="ac261-176">In these scenarios, a workload could be slated for future replacement, effectively removing it from the transformation effort.</span></span>

<span data-ttu-id="ac261-177">Os motivadores comuns podem incluir:</span><span class="sxs-lookup"><span data-stu-id="ac261-177">Common drivers could include:</span></span>

* <span data-ttu-id="ac261-178">Padronizar com base em práticas recomendadas do setor</span><span class="sxs-lookup"><span data-stu-id="ac261-178">Standardize around industry-best practices</span></span>
* <span data-ttu-id="ac261-179">Acelerar a adoção de abordagens orientadas por processo empresarial</span><span class="sxs-lookup"><span data-stu-id="ac261-179">Accelerate adoption of business process driven approaches</span></span>
* <span data-ttu-id="ac261-180">Realocar investimentos em desenvolvimento para aplicativos que criam vantagens ou diferenciais com relação à concorrência</span><span class="sxs-lookup"><span data-stu-id="ac261-180">Reallocate development investments into applications that create competitive differentiation or advantages</span></span>

<span data-ttu-id="ac261-181">Fatores de análise quantitativa:</span><span class="sxs-lookup"><span data-stu-id="ac261-181">Quantitative analysis factors:</span></span>

* <span data-ttu-id="ac261-182">Reduções de custo operacional geral</span><span class="sxs-lookup"><span data-stu-id="ac261-182">General operating cost reductions</span></span>
* <span data-ttu-id="ac261-183">Tamanho da VM (CPU, memória, armazenamento)</span><span class="sxs-lookup"><span data-stu-id="ac261-183">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="ac261-184">Dependências (tráfego de rede)</span><span class="sxs-lookup"><span data-stu-id="ac261-184">Dependencies (network traffic)</span></span>
* <span data-ttu-id="ac261-185">Ativos para desativação</span><span class="sxs-lookup"><span data-stu-id="ac261-185">Assets to be retired</span></span>

<span data-ttu-id="ac261-186">Fatores de análise qualitativa:</span><span class="sxs-lookup"><span data-stu-id="ac261-186">Qualitative analysis factors:</span></span>

* <span data-ttu-id="ac261-187">Análise de custo-benefício da arquitetura atual versus a solução de SaaS</span><span class="sxs-lookup"><span data-stu-id="ac261-187">Cost benefit analysis of the current architecture versus a SaaS solution</span></span>
* <span data-ttu-id="ac261-188">Mapas de processo de negócios</span><span class="sxs-lookup"><span data-stu-id="ac261-188">Business process maps</span></span>
* <span data-ttu-id="ac261-189">Esquemas de dados</span><span class="sxs-lookup"><span data-stu-id="ac261-189">Data schemas</span></span>
* <span data-ttu-id="ac261-190">Processos personalizados ou automatizados</span><span class="sxs-lookup"><span data-stu-id="ac261-190">Custom or automated processes</span></span>

## <a name="next-steps"></a><span data-ttu-id="ac261-191">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="ac261-191">Next steps</span></span>

<span data-ttu-id="ac261-192">Coletivamente, esses 5 Rs da Racionalização podem ser aplicados a uma propriedade digital a fim de tomar decisões de racionalização sobre o estado futuro de cada aplicativo.</span><span class="sxs-lookup"><span data-stu-id="ac261-192">Collectively, these 5 Rs of Rationalization can be applied to a digital estate to make rationalization decisions regarding the future state of each application.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="ac261-193">O que é uma propriedade digital?</span><span class="sxs-lookup"><span data-stu-id="ac261-193">What is a digital estate?</span></span>](overview.md)
