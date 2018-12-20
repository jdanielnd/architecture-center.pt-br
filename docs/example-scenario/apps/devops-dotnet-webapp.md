---
title: Criar um pipeline de CI/CD usando o Azure DevOps
description: Crie e libere um aplicativo .NET para aplicativos Web do Azure usando o Azure DevOps.
author: christianreddington
ms.date: 12/06/2018
ms.custom:
- fasttrack
- seodec18
ms.openlocfilehash: 23945493115522d099b6b26922f567653da0367e
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307275"
---
# <a name="design-a-cicd-pipeline-using-azure-devops"></a><span data-ttu-id="c9da9-103">Criar um pipeline de CI/CD usando o Azure DevOps</span><span class="sxs-lookup"><span data-stu-id="c9da9-103">Design a CI/CD pipeline using Azure DevOps</span></span>

<span data-ttu-id="c9da9-104">Este cenário fornece orientações de arquitetura e design para a criação de um pipeline de CI (integração contínua) e CD (implantação contínua).</span><span class="sxs-lookup"><span data-stu-id="c9da9-104">This scenario provides architecture and design guidance for building a continuous integration (CI) and continuous deployment (CD) pipeline.</span></span>  <span data-ttu-id="c9da9-105">Neste exemplo, o pipeline de CI/CD implanta um aplicativo Web do .NET de duas camadas para o Serviço de Aplicativo do Azure.</span><span class="sxs-lookup"><span data-stu-id="c9da9-105">In this example, the CI/CD pipeline deploys a two-tier .NET web application to the Azure App Service.</span></span>

<span data-ttu-id="c9da9-106">A migração para processos de CI/CD modernos fornece muitos benefícios para compilações, implantações, teste e monitoramento de aplicativos.</span><span class="sxs-lookup"><span data-stu-id="c9da9-106">Migrating to modern CI/CD processes provides many benefits for application builds, deployments, testing, and monitoring.</span></span> <span data-ttu-id="c9da9-107">Ao usar o Azure DevOps juntamente com outros serviços, como o Serviço de Aplicativo, as organizações podem se concentrar no desenvolvimento de seus aplicativos, em vez do gerenciamento da infraestrutura de suporte.</span><span class="sxs-lookup"><span data-stu-id="c9da9-107">By utilizing Azure DevOps along with other services such as App Service, organizations can focus on the development of their apps rather than the management of the supporting infrastructure.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="c9da9-108">Casos de uso relevantes</span><span class="sxs-lookup"><span data-stu-id="c9da9-108">Relevant use cases</span></span>

<span data-ttu-id="c9da9-109">Considere o Azure DevOps e os processos de CI/CD para:</span><span class="sxs-lookup"><span data-stu-id="c9da9-109">Consider Azure DevOps and CI/CD processes for:</span></span>

- <span data-ttu-id="c9da9-110">Aceleração do desenvolvimento de aplicativos e ciclos de vida de desenvolvimento</span><span class="sxs-lookup"><span data-stu-id="c9da9-110">Accelerating application development and development life cycles</span></span>
- <span data-ttu-id="c9da9-111">Gerar qualidade e consistência em um processo de versão e build automatizado</span><span class="sxs-lookup"><span data-stu-id="c9da9-111">Building quality and consistency into an automated build and release process</span></span>
- <span data-ttu-id="c9da9-112">Aumentar a estabilidade e o tempo de atividade do aplicativo</span><span class="sxs-lookup"><span data-stu-id="c9da9-112">Increasing application stability and uptime</span></span>

## <a name="architecture"></a><span data-ttu-id="c9da9-113">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="c9da9-113">Architecture</span></span>

![Diagrama da arquitetura dos componentes do Azure envolvidos em um cenário de DevOps usando Azure DevOps e o Serviço de Aplicativo do Azure][architecture]

<span data-ttu-id="c9da9-115">O fluxo de dados neste cenário ocorre da seguinte forma:</span><span class="sxs-lookup"><span data-stu-id="c9da9-115">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="c9da9-116">Um desenvolvedor altera o código-fonte do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="c9da9-116">A developer changes application source code.</span></span>
2. <span data-ttu-id="c9da9-117">O código do aplicativo, incluindo o arquivo web.config, é confirmado no repositório de código-fonte em Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="c9da9-117">Application code including the web.config file is committed to the source code repository in Azure Repos.</span></span>
3. <span data-ttu-id="c9da9-118">A integração contínua dispara o build do aplicativo e os testes de unidade usando o Azure Test Plans.</span><span class="sxs-lookup"><span data-stu-id="c9da9-118">Continuous integration triggers application build and unit tests using Azure Test Plans.</span></span>
4. <span data-ttu-id="c9da9-119">A implantação contínua dentro do Azure Pipelines dispara uma implantação automatizada de artefatos de aplicativo *com valores de configuração específicos ao ambiente*.</span><span class="sxs-lookup"><span data-stu-id="c9da9-119">Continuous deployment within Azure Pipelines triggers an automated deployment of application artifacts *with environment-specific configuration values*.</span></span>
5. <span data-ttu-id="c9da9-120">Os artefatos são implantados no Serviço de Aplicativo do Azure.</span><span class="sxs-lookup"><span data-stu-id="c9da9-120">The artifacts are deployed to Azure App Service.</span></span>
6. <span data-ttu-id="c9da9-121">O Application Insights do Azure coleta e analisa dados de integridade, de desempenho e de uso.</span><span class="sxs-lookup"><span data-stu-id="c9da9-121">Azure Application Insights collects and analyzes health, performance, and usage data.</span></span>
7. <span data-ttu-id="c9da9-122">Os desenvolvedores monitoram e gerenciam a integridade, o desempenho e o uso das informações.</span><span class="sxs-lookup"><span data-stu-id="c9da9-122">Developers monitor and mange health, performance, and usage information.</span></span>
8. <span data-ttu-id="c9da9-123">As informações da lista de pendências são usadas para priorizar novos recursos e correções de bug usando Azure Boards.</span><span class="sxs-lookup"><span data-stu-id="c9da9-123">Backlog information is used to prioritize new features and bug fixes using Azure Boards.</span></span>

### <a name="components"></a><span data-ttu-id="c9da9-124">Componentes</span><span class="sxs-lookup"><span data-stu-id="c9da9-124">Components</span></span>

- <span data-ttu-id="c9da9-125">O [Azure DevOps][vsts] é um serviço para gerenciar seu ciclo de vida de desenvolvimento de ponta a ponta&mdash;, desde o planejamento e gerenciamento de projetos até o gerenciamento de código, bem como a compilação e o lançamento.</span><span class="sxs-lookup"><span data-stu-id="c9da9-125">[Azure DevOps][vsts] is a service for managing your development life cycle end-to-end &mdash; from planning and project management, to code management, and continuing to build and release.</span></span>

- <span data-ttu-id="c9da9-126">Os [Aplicativos Web do Azure][web-apps] são um serviço de PaaS de hospedagem de aplicativos Web, APIs REST e back-ends móveis.</span><span class="sxs-lookup"><span data-stu-id="c9da9-126">[Azure Web Apps][web-apps] is a PaaS service for hosting web applications, REST APIs, and mobile back ends.</span></span> <span data-ttu-id="c9da9-127">Embora este artigo se concentre em .NET, há várias opções de plataforma de desenvolvimento adicionais com suporte.</span><span class="sxs-lookup"><span data-stu-id="c9da9-127">While this article focuses on .NET, there are several additional development platform options supported.</span></span>

- <span data-ttu-id="c9da9-128">O [Application Insights][application-insights] é um serviço de gerenciamento de desempenho de aplicativo (APM), “first-party” extensível para desenvolvedores da Web em várias plataformas.</span><span class="sxs-lookup"><span data-stu-id="c9da9-128">[Application Insights][application-insights] is a first-party, extensible Application Performance Management (APM) service for web developers on multiple platforms.</span></span>

### <a name="alternatives"></a><span data-ttu-id="c9da9-129">Alternativas</span><span class="sxs-lookup"><span data-stu-id="c9da9-129">Alternatives</span></span>

<span data-ttu-id="c9da9-130">Embora este artigo se concentre no Azure DevOps, o [Azure DevOps Server][azure-devops-server] (conhecido como Team Foundation Server) poderia ser usado como substituto local.</span><span class="sxs-lookup"><span data-stu-id="c9da9-130">While this article focuses on Azure DevOps, [Azure DevOps Server][azure-devops-server] (previously known as Team Foundation Server) could be used as an on-premises substitute.</span></span> <span data-ttu-id="c9da9-131">Como alternativa, você também pode usar um conjunto de tecnologias para um pipeline de desenvolvimento de software livre usando [Jenkins][jenkins-on-azure].</span><span class="sxs-lookup"><span data-stu-id="c9da9-131">Alternatively, you could also use a set of technologies for an open-source development pipeline using [Jenkins][jenkins-on-azure].</span></span>

<span data-ttu-id="c9da9-132">De uma perspectiva de infraestrutura como código, [modelos do Resource Manager][arm-templates] foram usados como parte do projeto do Azure DevOps, mas você pode considerar outras tecnologias de gerenciamento, como [Terraform][terraform] ou [Chef][chef].</span><span class="sxs-lookup"><span data-stu-id="c9da9-132">From an infrastructure-as-code perspective, [Resource Manager templates][arm-templates] were used as part of the Azure DevOps project, but you could consider other management technologies such as [Terraform][terraform] or [Chef][chef].</span></span> <span data-ttu-id="c9da9-133">Se você prefere uma implantação baseada em IaaS (infraestrutura como serviço) e exige o gerenciamento de configuração, você poderia considerar o [State Configuration da Automação do Azure][desired-state-configuration], o [Ansible][ansible] ou o [Chef][chef].</span><span class="sxs-lookup"><span data-stu-id="c9da9-133">If you prefer an infrastructure-as-a-service (IaaS)-based deployment and require configuration management, you could consider either [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible], or [Chef][chef].</span></span>

<span data-ttu-id="c9da9-134">Você pode considerar estas alternativas à hospedagem em Azure Web Apps:</span><span class="sxs-lookup"><span data-stu-id="c9da9-134">You could consider these alternatives to hosting in Azure Web Apps:</span></span>

- <span data-ttu-id="c9da9-135">As [Máquinas Virtuais do Azure][compare-vm-hosting] lidam com cargas de trabalho que exigem um alto grau de controle, ou dependem dos componentes do sistema operacional e serviços que não são possíveis com Aplicativos Web (por exemplo, o Windows GAC ou COM).</span><span class="sxs-lookup"><span data-stu-id="c9da9-135">[Azure Virtual Machines][compare-vm-hosting] handles workloads that require a high degree of control, or depend on OS components and services that are not possible with Web Apps (for example, the Windows GAC, or COM).</span></span>

- <span data-ttu-id="c9da9-136">O [Service Fabric][service-fabric] é uma boa opção se a arquitetura da carga de trabalho for voltada para componentes distribuídos que se beneficiam da implantação e execução em um cluster com um alto grau de controle.</span><span class="sxs-lookup"><span data-stu-id="c9da9-136">[Service Fabric][service-fabric] is a good option if the workload architecture is focused around distributed components that benefit from being deployed and run across a cluster with a high degree of control.</span></span> <span data-ttu-id="c9da9-137">O Service Fabric também pode ser usado para hospedar contêineres.</span><span class="sxs-lookup"><span data-stu-id="c9da9-137">Service Fabric can also be used to host containers.</span></span>

- <span data-ttu-id="c9da9-138">O [Azure Functions][azure-functions] fornece uma abordagem eficaz sem servidor se a arquitetura de carga de trabalho é centralizada em torno de componentes distribuídos refinados, exigindo dependências mínimas, onde componentes individuais são necessários somente para execução sob demanda (não continuamente) e a orquestração dos componentes não é necessária.</span><span class="sxs-lookup"><span data-stu-id="c9da9-138">[Azure Functions][azure-functions] provides an effective serverless approach if the workload architecture is centered around fine grained distributed components, requiring minimal dependencies, where individual components are only required to run on demand (not continuously) and orchestration of components is not required.</span></span>

<span data-ttu-id="c9da9-139">Essa [árvore de decisão para serviços de computação do Azure](/azure/architecture/guide/technology-choices/compute-decision-tree) pode ajudar na escolha do caminho certo para uma migração.</span><span class="sxs-lookup"><span data-stu-id="c9da9-139">This [decision tree for Azure compute services](/azure/architecture/guide/technology-choices/compute-decision-tree) may help when choosing the right path to take for a migration.</span></span>

## <a name="management-and-security-considerations"></a><span data-ttu-id="c9da9-140">Considerações de gerenciamento e segurança</span><span class="sxs-lookup"><span data-stu-id="c9da9-140">Management and Security Considerations</span></span>

- <span data-ttu-id="c9da9-141">Considere aproveitar as [tarefas de geração de tokens][vsts-tokenization] que estão disponíveis no marketplace do VSTS.</span><span class="sxs-lookup"><span data-stu-id="c9da9-141">Consider leveraging one of the [tokenization tasks][vsts-tokenization] available in the VSTS marketplace.</span></span>

- <span data-ttu-id="c9da9-142">As tarefas do [Azure Key Vault][download-keyvault-secrets] podem baixar segredos de um Azure Key Vault para sua versão.</span><span class="sxs-lookup"><span data-stu-id="c9da9-142">[Azure Key Vault][download-keyvault-secrets] tasks can download secrets from an Azure Key Vault into your release.</span></span> <span data-ttu-id="c9da9-143">Você pode usar esses segredos como variáveis em sua definição de versão, o que evita armazená-los no controle do código-fonte.</span><span class="sxs-lookup"><span data-stu-id="c9da9-143">You can then use those secrets as variables in your release definition, which avoids storing them in source control.</span></span>

- <span data-ttu-id="c9da9-144">Use as [variáveis de versão][vsts-release-variables] em suas definições de versão para fazer alterações de configuração de seus ambientes.</span><span class="sxs-lookup"><span data-stu-id="c9da9-144">Use [release variables][vsts-release-variables] in your release definitions to drive configuration changes of your environments.</span></span> <span data-ttu-id="c9da9-145">As variáveis de versão podem ser definidas para uma versão inteira ou um determinado ambiente.</span><span class="sxs-lookup"><span data-stu-id="c9da9-145">Release variables can be scoped to an entire release or a given environment.</span></span> <span data-ttu-id="c9da9-146">Ao usar variáveis de informações secretas, selecione o ícone de cadeado.</span><span class="sxs-lookup"><span data-stu-id="c9da9-146">When using variables for secret information, ensure that you select the padlock icon.</span></span>

- <span data-ttu-id="c9da9-147">Os [portões de implantação][vsts-deployment-gates] devem ser usados em seu pipeline de lançamento.</span><span class="sxs-lookup"><span data-stu-id="c9da9-147">[Deployment gates][vsts-deployment-gates] should be used in your release pipeline.</span></span> <span data-ttu-id="c9da9-148">Isso permite que você aproveite os dados de monitoramento junto com sistemas externos (por exemplo, gerenciamento de incidentes ou outros sistemas personalizados) para determinar se uma versão deve ser promovida.</span><span class="sxs-lookup"><span data-stu-id="c9da9-148">This lets you leverage monitoring data in association with external systems (for example, incident management or additional bespoke systems) to determine whether a release should be promoted.</span></span>

- <span data-ttu-id="c9da9-149">Quando a intervenção manual em um pipeline de lançamento for necessário, use a funcionalidade [aprovações][vsts-approvals].</span><span class="sxs-lookup"><span data-stu-id="c9da9-149">Where manual intervention in a release pipeline is required, use the [approvals][vsts-approvals] functionality.</span></span>

- <span data-ttu-id="c9da9-150">Considere usar o [Application Insights][application-insights] e outras ferramentas de monitoramento o mais cedo possível no seu pipeline de lançamento.</span><span class="sxs-lookup"><span data-stu-id="c9da9-150">Consider using [Application Insights][application-insights] and additional monitoring tools as early as possible in your release pipeline.</span></span> <span data-ttu-id="c9da9-151">Muitas organizações só começam o monitoramento em seus ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="c9da9-151">Many organizations only begin monitoring in their production environment.</span></span> <span data-ttu-id="c9da9-152">Ao monitorar outros ambientes, você pode identificar bugs desde o início do processo de desenvolvimento e evitar problemas em seu ambiente de produção.</span><span class="sxs-lookup"><span data-stu-id="c9da9-152">By monitoring your other environments, you can identify bugs earlier in the development process and avoid issues in your production environment.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="c9da9-153">Implantar o cenário</span><span class="sxs-lookup"><span data-stu-id="c9da9-153">Deploy the scenario</span></span>

### <a name="prerequisites"></a><span data-ttu-id="c9da9-154">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="c9da9-154">Prerequisites</span></span>

- <span data-ttu-id="c9da9-155">Você deve ter uma conta do Azure já criada.</span><span class="sxs-lookup"><span data-stu-id="c9da9-155">You must have an existing Azure account.</span></span> <span data-ttu-id="c9da9-156">Se você não tiver uma assinatura do Azure, crie uma [conta gratuita][azure-free-account] antes de começar.</span><span class="sxs-lookup"><span data-stu-id="c9da9-156">If you don't have an Azure subscription, create a [free account][azure-free-account] before you begin.</span></span>

- <span data-ttu-id="c9da9-157">Você deve se inscrever em uma organização do Azure DevOps.</span><span class="sxs-lookup"><span data-stu-id="c9da9-157">You must sign up for an Azure DevOps organization.</span></span> <span data-ttu-id="c9da9-158">Para saber mais, consulte [Início Rápido: Criar sua organização][vsts-account-create].</span><span class="sxs-lookup"><span data-stu-id="c9da9-158">For more information, see [Quickstart: Create your organization][vsts-account-create].</span></span>

### <a name="walk-through"></a><span data-ttu-id="c9da9-159">Passo a passo</span><span class="sxs-lookup"><span data-stu-id="c9da9-159">Walk-through</span></span>

<span data-ttu-id="c9da9-160">O [projeto de Azure DevOps](/azure/devops-project/azure-devops-project-github) implantará um Plano do Serviço de Aplicativo, um Serviço de Aplicativo e um recurso do Application Insights para você, além de configurar o projeto de Azure DevOps.</span><span class="sxs-lookup"><span data-stu-id="c9da9-160">The [Azure DevOps project](/azure/devops-project/azure-devops-project-github) will deploy an App Service Plan, App Service, and an App Insights resource for you, as well as configure the Azure DevOps project for you.</span></span>

<span data-ttu-id="c9da9-161">Depois que a implantação do projeto de Azure DevOps e do build for concluída, analise as alterações de código associadas, os itens de trabalho e os resultados de teste.</span><span class="sxs-lookup"><span data-stu-id="c9da9-161">Once you've deployed the Azure DevOps project and the build is completed, review the associated code changes, work items, and test results.</span></span> <span data-ttu-id="c9da9-162">Você observará que nenhum resultado de teste é exibido, pois o código não contém nenhum teste a ser executado.</span><span class="sxs-lookup"><span data-stu-id="c9da9-162">You will notice that no test results are displayed, because the code does not contain any tests to run.</span></span>

<span data-ttu-id="c9da9-163">O projeto cria um pipeline de lançamento e uma gatilho de implantação contínua, implantando nosso aplicativo no ambiente de desenvolvimento.</span><span class="sxs-lookup"><span data-stu-id="c9da9-163">The project creates a release pipeline and continuous deployment trigger, deploying our application into the Dev environment.</span></span> <span data-ttu-id="c9da9-164">Como parte de um processo de implantação contínua, você poderá ver versões que abrangem vários ambientes.</span><span class="sxs-lookup"><span data-stu-id="c9da9-164">As part of a continuous deployment process, you may see releases that span multiple environments.</span></span> <span data-ttu-id="c9da9-165">Uma versão pode abranger infraestrutura (usando técnicas como infraestrutura como código) e também implantar os pacotes de aplicativos necessários, juntamente com quaisquer tarefas de pós-configuração.</span><span class="sxs-lookup"><span data-stu-id="c9da9-165">A release can span both infrastructure (using techniques such as infrastructure-as-code), and can also deploy the application packages required along with any post-configuration tasks.</span></span>

## <a name="pricing"></a><span data-ttu-id="c9da9-166">Preços</span><span class="sxs-lookup"><span data-stu-id="c9da9-166">Pricing</span></span>

<span data-ttu-id="c9da9-167">Os custos do Azure DevOps dependem do número de usuários em sua organização que precisam de acesso, juntamente com outros fatores, como o número de builds/versões simultâneos necessários e o números de usuários de teste.</span><span class="sxs-lookup"><span data-stu-id="c9da9-167">Azure DevOps costs depend on the number of users in your organization that require access, along with other factors like the number of concurrent build/releases required and number of test users.</span></span> <span data-ttu-id="c9da9-168">Para saber mais, confira [Preços de Azure DevOps][vsts-pricing-page].</span><span class="sxs-lookup"><span data-stu-id="c9da9-168">For more information, see [Azure DevOps pricing][vsts-pricing-page].</span></span>

<span data-ttu-id="c9da9-169">Essa [Calculadora de preços][vsts-pricing-calculator] fornece uma estimativa para a execução do Azure DevOps com 20 usuários.</span><span class="sxs-lookup"><span data-stu-id="c9da9-169">This [pricing calculator][vsts-pricing-calculator] provides an estimate for running Azure DevOps with 20 users.</span></span>

<span data-ttu-id="c9da9-170">O Azure DevOps é cobrado por mês e por usuário.</span><span class="sxs-lookup"><span data-stu-id="c9da9-170">Azure DevOps is billed on a per-user per-month basis.</span></span> <span data-ttu-id="c9da9-171">Pode haver cobranças adicionais, dependendo da necessidade de pipelines simultâneos, além de usuários de teste ou licenças de usuário básicas adicionais.</span><span class="sxs-lookup"><span data-stu-id="c9da9-171">There may be additional charges dependent upon concurrent pipelines needed, in addition to any additional test users or user basic licenses.</span></span>

## <a name="related-resources"></a><span data-ttu-id="c9da9-172">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="c9da9-172">Related resources</span></span>

<span data-ttu-id="c9da9-173">Examine os seguintes recursos para saber mais sobre CI/CD e o Azure DevOps:</span><span class="sxs-lookup"><span data-stu-id="c9da9-173">Review the following resources to learn more about CI/CD and Azure DevOps:</span></span>

- <span data-ttu-id="c9da9-174">[O que é DevOps?][devops-whatis]</span><span class="sxs-lookup"><span data-stu-id="c9da9-174">[What is DevOps?][devops-whatis]</span></span>
- <span data-ttu-id="c9da9-175">[DevOps na Microsoft ‒ como trabalhamos com o Azure DevOps][devops-microsoft]</span><span class="sxs-lookup"><span data-stu-id="c9da9-175">[DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft]</span></span>
- <span data-ttu-id="c9da9-176">[Tutoriais passo a passo: DevOps com Azure DevOps][devops-with-vsts]</span><span class="sxs-lookup"><span data-stu-id="c9da9-176">[Step-by-step Tutorials: DevOps with Azure DevOps][devops-with-vsts]</span></span>
- <span data-ttu-id="c9da9-177">[Lista de verificação de DevOps][devops-checklist]</span><span class="sxs-lookup"><span data-stu-id="c9da9-177">[Devops Checklist][devops-checklist]</span></span>
- <span data-ttu-id="c9da9-178">[Criar um pipeline de CI/CD para .NET com o Projeto de DevOps do Azure][devops-project-create]</span><span class="sxs-lookup"><span data-stu-id="c9da9-178">[Create a CI/CD pipeline for .NET with the Azure DevOps project][devops-project-create]</span></span>

<!-- links -->

[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/architecture-devops-dotnet-webapp.svg
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[devops-checklist]: /azure/architecture/checklist/dev-ops
[application-insights]: https://azure.microsoft.com/services/application-insights/
[cloud-based-load-testing]: https://visualstudio.microsoft.com/team-services/cloud-load-testing/
[cloud-based-load-testing-on-premises]: /vsts/test/load-test/clt-with-private-machines?view=vsts
[jenkins-on-azure]: /azure/jenkins/
[devops-whatis]: /azure/devops/what-is-devops
[download-keyvault-secrets]: /vsts/pipelines/tasks/deploy/azure-key-vault?view=vsts
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[vsts-account-create]: /azure/devops/organizations/accounts/create-organization-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[azure-devops-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]: /azure/service-fabric/
[azure-functions]: /azure/azure-functions/
[azure-containers]: https://azure.microsoft.com/overview/containers/
[compare-vm-hosting]: /azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]: /azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]: /azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[terraform]: /azure/terraform/
