---
title: 5 Rs da racionalização
titleSuffix: Enterprise Cloud Adoption
description: Descreve as opções disponíveis ao racionalizar uma propriedade digital
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 06058967e6ffcd9e3554a46c67144f72fb19078f
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908572"
---
# <a name="enterprise-cloud-adoption-the-5-rs-of-rationalization"></a><span data-ttu-id="ec395-103">Adoção da Nuvem Empresarial: 5 Rs da racionalização</span><span class="sxs-lookup"><span data-stu-id="ec395-103">Enterprise Cloud Adoption: The 5 Rs of rationalization</span></span>

<span data-ttu-id="ec395-104">Racionalização de nuvem é o processo de avaliação de ativos para determinar a melhor abordagem para a migração ou modernização de cada ativo na nuvem.</span><span class="sxs-lookup"><span data-stu-id="ec395-104">Cloud Rationalization is the process of evaluating assets to determine the best approach to migrating or modernizing each asset in the cloud.</span></span> <span data-ttu-id="ec395-105">Para saber mais sobre o processo de racionalização, confira [O que é uma propriedade digital?](overview.md)</span><span class="sxs-lookup"><span data-stu-id="ec395-105">For more information about the process of rationalization, see [What is a digital estate?](overview.md)</span></span>

<span data-ttu-id="ec395-106">O "5 Rs da racionalização" listados aqui descrevem as opções mais comuns de racionalização.</span><span class="sxs-lookup"><span data-stu-id="ec395-106">The "5 Rs of rationalization" listed here describe the most common options for rationalization.</span></span>

## <a name="rehost"></a><span data-ttu-id="ec395-107">Hospedar novamente</span><span class="sxs-lookup"><span data-stu-id="ec395-107">Rehost</span></span>

<span data-ttu-id="ec395-108">Também conhecido como "lift and shift", um esforço de nova hospedagem move o ativo de estado atual para o provedor de nuvem escolhido, com alterações mínimas na arquitetura geral.</span><span class="sxs-lookup"><span data-stu-id="ec395-108">Also known as "lift and shift," a rehost effort moves the current state asset to the chosen cloud provider, with minimal change to overall architecture.</span></span>

<span data-ttu-id="ec395-109">Os motivadores comuns podem incluir:</span><span class="sxs-lookup"><span data-stu-id="ec395-109">Common drivers could include:</span></span>

* <span data-ttu-id="ec395-110">Reduzir CapEx</span><span class="sxs-lookup"><span data-stu-id="ec395-110">Reduce CapEx</span></span>
* <span data-ttu-id="ec395-111">Liberar espaço no datacenter</span><span class="sxs-lookup"><span data-stu-id="ec395-111">Free up datacenter space</span></span>
* <span data-ttu-id="ec395-112">ROI de nuvem rápido</span><span class="sxs-lookup"><span data-stu-id="ec395-112">Quick cloud ROI</span></span>

<span data-ttu-id="ec395-113">Fatores de análise quantitativa:</span><span class="sxs-lookup"><span data-stu-id="ec395-113">Quantitative analysis factors:</span></span>

* <span data-ttu-id="ec395-114">Tamanho da VM (CPU, memória, armazenamento)</span><span class="sxs-lookup"><span data-stu-id="ec395-114">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="ec395-115">Dependências (tráfego de rede)</span><span class="sxs-lookup"><span data-stu-id="ec395-115">Dependencies (network traffic)</span></span>
* <span data-ttu-id="ec395-116">Compatibilidade de ativo</span><span class="sxs-lookup"><span data-stu-id="ec395-116">Asset compatibility</span></span>

<span data-ttu-id="ec395-117">Fatores de análise qualitativa:</span><span class="sxs-lookup"><span data-stu-id="ec395-117">Qualitative analysis factors:</span></span>

* <span data-ttu-id="ec395-118">Tolerância à alteração</span><span class="sxs-lookup"><span data-stu-id="ec395-118">Tolerance for change</span></span>
* <span data-ttu-id="ec395-119">Prioridades dos negócios</span><span class="sxs-lookup"><span data-stu-id="ec395-119">Business priorities</span></span>
* <span data-ttu-id="ec395-120">Eventos comerciais críticos</span><span class="sxs-lookup"><span data-stu-id="ec395-120">Critical business events</span></span>
* <span data-ttu-id="ec395-121">Dependências de processo</span><span class="sxs-lookup"><span data-stu-id="ec395-121">Process dependencies</span></span>

## <a name="refactor"></a><span data-ttu-id="ec395-122">Refatoração</span><span class="sxs-lookup"><span data-stu-id="ec395-122">Refactor</span></span>

<span data-ttu-id="ec395-123">As opções de PaaS (Plataforma como serviço) podem reduzir os custos operacionais associados a muitos aplicativos.</span><span class="sxs-lookup"><span data-stu-id="ec395-123">Platform as a Service (PaaS) options can reduce operational costs associated with many applications.</span></span> <span data-ttu-id="ec395-124">Pode ser prudente refatorar ligeiramente um aplicativo a fim de ajustá-lo a um modelo com base em PaaS.</span><span class="sxs-lookup"><span data-stu-id="ec395-124">It can be prudent to slightly refactor an application to fit a PaaS based model.</span></span>

<span data-ttu-id="ec395-125">Refatorar também se refere ao processo de desenvolvimento de aplicativo de refatoração de código, a fim de permitir que um aplicativo forneça novas oportunidades de negócios.</span><span class="sxs-lookup"><span data-stu-id="ec395-125">Refactor also refers to the application development process of refactoring code to allow an application to deliver on new business opportunities.</span></span>

<span data-ttu-id="ec395-126">Os motivadores comuns podem incluir:</span><span class="sxs-lookup"><span data-stu-id="ec395-126">Common drivers could include:</span></span>

* <span data-ttu-id="ec395-127">Atualizações mais rápidas e curtas</span><span class="sxs-lookup"><span data-stu-id="ec395-127">Faster, shorter, updates</span></span>
* <span data-ttu-id="ec395-128">Portabilidade do código</span><span class="sxs-lookup"><span data-stu-id="ec395-128">Code portability</span></span>
* <span data-ttu-id="ec395-129">Maior eficiência de nuvem (recursos, velocidade e custo)</span><span class="sxs-lookup"><span data-stu-id="ec395-129">Greater cloud efficiency (resources, speed, cost)</span></span>

<span data-ttu-id="ec395-130">Fatores de análise quantitativa:</span><span class="sxs-lookup"><span data-stu-id="ec395-130">Quantitative analysis factors:</span></span>

* <span data-ttu-id="ec395-131">Tamanho do ativo de aplicativo (CPU, memória, armazenamento)</span><span class="sxs-lookup"><span data-stu-id="ec395-131">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="ec395-132">Dependências (tráfego de rede)</span><span class="sxs-lookup"><span data-stu-id="ec395-132">Dependencies (network traffic)</span></span>
* <span data-ttu-id="ec395-133">Tráfego do usuário (exibições de página, hora na página, tempo de carregamento)</span><span class="sxs-lookup"><span data-stu-id="ec395-133">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="ec395-134">Plataforma de desenvolvimento (linguagens, plataforma de dados, serviços de camada intermediária)</span><span class="sxs-lookup"><span data-stu-id="ec395-134">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="ec395-135">Fatores de análise qualitativa:</span><span class="sxs-lookup"><span data-stu-id="ec395-135">Qualitative analysis factors:</span></span>

* <span data-ttu-id="ec395-136">Investimento contínuo nos negócios</span><span class="sxs-lookup"><span data-stu-id="ec395-136">Continued business investments</span></span>
* <span data-ttu-id="ec395-137">Opções/linhas do tempo de intermitência</span><span class="sxs-lookup"><span data-stu-id="ec395-137">Bursting options/timelines</span></span>
* <span data-ttu-id="ec395-138">Dependências de processo comercial</span><span class="sxs-lookup"><span data-stu-id="ec395-138">Business process dependencies</span></span>

## <a name="rearchitect"></a><span data-ttu-id="ec395-139">Recriação da arquitetura</span><span class="sxs-lookup"><span data-stu-id="ec395-139">Rearchitect</span></span>

<span data-ttu-id="ec395-140">Alguns aplicativos mais velhos não são compatíveis com provedores de nuvem devido a decisões arquitetônicas tomadas durante a compilação do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="ec395-140">Some aging applications aren't compatible with cloud providers because of the architectural decisions made when the application was built.</span></span> <span data-ttu-id="ec395-141">Nesses casos, o aplicativo pode precisar de uma nova arquitetura antes da transformação.</span><span class="sxs-lookup"><span data-stu-id="ec395-141">In these cases, the application may need to be rearchitected prior to transformation.</span></span>

<span data-ttu-id="ec395-142">Em outros casos, os aplicativos que são compatíveis com a nuvem, mas que não conta com benefícios por serem nativos, podem melhorar o custo e as operações por meio da nova arquitetura da solução, a fim de se tornarem um aplicativo nativo de nuvem.</span><span class="sxs-lookup"><span data-stu-id="ec395-142">In other cases, applications that are cloud compatible, but not cloud native benefits, may produce cost efficiencies and operational efficiencies by rearchitecting the solution to be a cloud native application.</span></span>

<span data-ttu-id="ec395-143">Os motivadores comuns podem incluir:</span><span class="sxs-lookup"><span data-stu-id="ec395-143">Common drivers could include:</span></span>

* <span data-ttu-id="ec395-144">Agilidade e escala do aplicativo</span><span class="sxs-lookup"><span data-stu-id="ec395-144">Application scale and agility</span></span>
* <span data-ttu-id="ec395-145">Facilidade da adoção de novos recursos de nuvem</span><span class="sxs-lookup"><span data-stu-id="ec395-145">Easier adoption of new cloud capabilities</span></span>
* <span data-ttu-id="ec395-146">Combinação de pilhas de tecnologia</span><span class="sxs-lookup"><span data-stu-id="ec395-146">Mix of technology stacks</span></span>

<span data-ttu-id="ec395-147">Fatores de análise quantitativa:</span><span class="sxs-lookup"><span data-stu-id="ec395-147">Quantitative analysis factors:</span></span>

* <span data-ttu-id="ec395-148">Tamanho do ativo de aplicativo (CPU, memória, armazenamento)</span><span class="sxs-lookup"><span data-stu-id="ec395-148">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="ec395-149">Dependências (tráfego de rede)</span><span class="sxs-lookup"><span data-stu-id="ec395-149">Dependencies (network traffic)</span></span>
* <span data-ttu-id="ec395-150">Tráfego do usuário (exibições de página, hora na página, tempo de carregamento)</span><span class="sxs-lookup"><span data-stu-id="ec395-150">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="ec395-151">Plataforma de desenvolvimento (linguagens, plataforma de dados, serviços de camada intermediária)</span><span class="sxs-lookup"><span data-stu-id="ec395-151">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="ec395-152">Fatores de análise qualitativa:</span><span class="sxs-lookup"><span data-stu-id="ec395-152">Qualitative analysis factors:</span></span>

* <span data-ttu-id="ec395-153">Investimentos comerciais em expansão</span><span class="sxs-lookup"><span data-stu-id="ec395-153">Growing business investments</span></span>
* <span data-ttu-id="ec395-154">Custos operacionais</span><span class="sxs-lookup"><span data-stu-id="ec395-154">Operational costs</span></span>
* <span data-ttu-id="ec395-155">Possíveis loops de comentários e investimentos de DevOps</span><span class="sxs-lookup"><span data-stu-id="ec395-155">Potential feedback loops and DevOps investments</span></span>

## <a name="rebuild"></a><span data-ttu-id="ec395-156">Recompilação</span><span class="sxs-lookup"><span data-stu-id="ec395-156">Rebuild</span></span>

<span data-ttu-id="ec395-157">Em alguns cenários, o delta que deve ser superado para avançar com um aplicativo pode ser muito grande para justificar o investimento.</span><span class="sxs-lookup"><span data-stu-id="ec395-157">In some scenarios, the delta that must be overcome to carry forward an application can be too large to justify further investment.</span></span> <span data-ttu-id="ec395-158">Isso é especialmente verdadeiro para aplicativos que costumavam atender às necessidades do negócio, mas que agora não têm suporte ou são não estão alinhados com o modo como os processos comerciais são executados atualmente.</span><span class="sxs-lookup"><span data-stu-id="ec395-158">This is especially true for applications that used to meet the needs of the business, but are now unsupported or misaligned with how the business processes are executed today.</span></span> <span data-ttu-id="ec395-159">Nesse caso, uma nova base de código é criada de acordo com uma abordagem de [nuvem nativa](https://azure.microsoft.com/overview/cloudnative/).</span><span class="sxs-lookup"><span data-stu-id="ec395-159">In this case, a new code base is created to align with a [cloud native](https://azure.microsoft.com/overview/cloudnative/) approach.</span></span>

<span data-ttu-id="ec395-160">Os motivadores comuns podem incluir:</span><span class="sxs-lookup"><span data-stu-id="ec395-160">Common drivers could include:</span></span>

* <span data-ttu-id="ec395-161">Acelerar a inovação</span><span class="sxs-lookup"><span data-stu-id="ec395-161">Accelerate innovation</span></span>
* <span data-ttu-id="ec395-162">Compilar aplicativos mais rápido</span><span class="sxs-lookup"><span data-stu-id="ec395-162">Build apps faster</span></span>
* <span data-ttu-id="ec395-163">Reduzir o custo operacional</span><span class="sxs-lookup"><span data-stu-id="ec395-163">Reduce operational cost</span></span>

<span data-ttu-id="ec395-164">Fatores de análise quantitativa:</span><span class="sxs-lookup"><span data-stu-id="ec395-164">Quantitative analysis factors:</span></span>

* <span data-ttu-id="ec395-165">Tamanho do ativo de aplicativo (CPU, memória, armazenamento)</span><span class="sxs-lookup"><span data-stu-id="ec395-165">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="ec395-166">Dependências (tráfego de rede)</span><span class="sxs-lookup"><span data-stu-id="ec395-166">Dependencies (network traffic)</span></span>
* <span data-ttu-id="ec395-167">Tráfego do usuário (exibições de página, hora na página, tempo de carregamento)</span><span class="sxs-lookup"><span data-stu-id="ec395-167">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="ec395-168">Plataforma de desenvolvimento (linguagens, plataforma de dados, serviços de camada intermediária)</span><span class="sxs-lookup"><span data-stu-id="ec395-168">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="ec395-169">Fatores de análise qualitativa:</span><span class="sxs-lookup"><span data-stu-id="ec395-169">Qualitative analysis Factors:</span></span>

* <span data-ttu-id="ec395-170">Satisfação do usuário final em queda</span><span class="sxs-lookup"><span data-stu-id="ec395-170">Declining end user satisfaction</span></span>
* <span data-ttu-id="ec395-171">Processos de negócio limitados pela funcionalidade</span><span class="sxs-lookup"><span data-stu-id="ec395-171">Business processes limited by functionality</span></span>
* <span data-ttu-id="ec395-172">Possíveis ganhos de custo, experiência ou receita</span><span class="sxs-lookup"><span data-stu-id="ec395-172">Potential cost, experience, or revenue gains</span></span>

## <a name="replace"></a><span data-ttu-id="ec395-173">Substitua</span><span class="sxs-lookup"><span data-stu-id="ec395-173">Replace</span></span>

<span data-ttu-id="ec395-174">Normalmente, as soluções são implementadas usando a melhor tecnologia e abordagem disponíveis no momento.</span><span class="sxs-lookup"><span data-stu-id="ec395-174">Solutions are generally implemented using the best technology and approach available at the time.</span></span> <span data-ttu-id="ec395-175">Em alguns casos, aplicativos SaaS (Software como serviço) podem atender a todas as funcionalidades necessárias do aplicativo hospedado.</span><span class="sxs-lookup"><span data-stu-id="ec395-175">In some cases, Software as a Service (SaaS) applications can meet all of the functionality required of the hosted application.</span></span> <span data-ttu-id="ec395-176">Nesses cenários, uma carga de trabalho pode ser programada para substituição futura, removendo-a efetivamente do esforço de transformação.</span><span class="sxs-lookup"><span data-stu-id="ec395-176">In these scenarios, a workload could be slated for future replacement, effectively removing it from the transformation effort.</span></span>

<span data-ttu-id="ec395-177">Os motivadores comuns podem incluir:</span><span class="sxs-lookup"><span data-stu-id="ec395-177">Common drivers could include:</span></span>

* <span data-ttu-id="ec395-178">Padronizar com base em práticas recomendadas do setor</span><span class="sxs-lookup"><span data-stu-id="ec395-178">Standardize around industry-best practices</span></span>
* <span data-ttu-id="ec395-179">Acelerar a adoção de abordagens orientadas por processo empresarial</span><span class="sxs-lookup"><span data-stu-id="ec395-179">Accelerate adoption of business process driven approaches</span></span>
* <span data-ttu-id="ec395-180">Realocar investimentos em desenvolvimento para aplicativos que criam vantagens ou diferenciais com relação à concorrência</span><span class="sxs-lookup"><span data-stu-id="ec395-180">Reallocate development investments into applications that create competitive differentiation or advantages</span></span>

<span data-ttu-id="ec395-181">Fatores de análise quantitativa:</span><span class="sxs-lookup"><span data-stu-id="ec395-181">Quantitative analysis factors:</span></span>

* <span data-ttu-id="ec395-182">Reduções de custo operacional geral</span><span class="sxs-lookup"><span data-stu-id="ec395-182">General operating cost reductions</span></span>
* <span data-ttu-id="ec395-183">Tamanho da VM (CPU, memória, armazenamento)</span><span class="sxs-lookup"><span data-stu-id="ec395-183">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="ec395-184">Dependências (tráfego de rede)</span><span class="sxs-lookup"><span data-stu-id="ec395-184">Dependencies (network traffic)</span></span>
* <span data-ttu-id="ec395-185">Ativos para desativação</span><span class="sxs-lookup"><span data-stu-id="ec395-185">Assets to be retired</span></span>

<span data-ttu-id="ec395-186">Fatores de análise qualitativa:</span><span class="sxs-lookup"><span data-stu-id="ec395-186">Qualitative analysis factors:</span></span>

* <span data-ttu-id="ec395-187">Análise de custo-benefício da arquitetura atual versus a solução de SaaS</span><span class="sxs-lookup"><span data-stu-id="ec395-187">Cost benefit analysis of current architecture vs SaaS solution</span></span>
* <span data-ttu-id="ec395-188">Mapas de processo de negócios</span><span class="sxs-lookup"><span data-stu-id="ec395-188">Business process maps</span></span>
* <span data-ttu-id="ec395-189">Esquemas de dados</span><span class="sxs-lookup"><span data-stu-id="ec395-189">Data schemas</span></span>
* <span data-ttu-id="ec395-190">Processos personalizados ou automatizados</span><span class="sxs-lookup"><span data-stu-id="ec395-190">Custom or automated processes</span></span>

## <a name="next-steps"></a><span data-ttu-id="ec395-191">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="ec395-191">Next steps</span></span>

<span data-ttu-id="ec395-192">Coletivamente, esses 5 Rs da Racionalização podem ser aplicados a uma propriedade digital a fim de tomar decisões de racionalização sobre o estado futuro de cada aplicativo.</span><span class="sxs-lookup"><span data-stu-id="ec395-192">Collectively, these 5 Rs of Rationalization can be applied to a digital estate to make rationalization decisions regarding the future state of each application.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="ec395-193">O que é uma propriedade digital?</span><span class="sxs-lookup"><span data-stu-id="ec395-193">What is a digital estate?</span></span>](overview.md)
