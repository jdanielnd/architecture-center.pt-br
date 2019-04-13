---
title: Criando um pipeline de CI/CD para microsserviços no Kubernetes
description: Descreve um exemplo de pipeline de CI/CD para implantar microsserviços para o serviço de Kubernetes do Azure (AKS).
author: MikeWasson
ms.date: 04/11/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: e7da8a1b4111fb7856b919b40d033833a4e475e0
ms.sourcegitcommit: d58e6b2b891c9c99e951c59f15fce71addcb96b1
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/12/2019
ms.locfileid: "59533149"
---
<!-- markdownlint-disable MD040 -->

# <a name="building-a-cicd-pipeline-for-microservices-on-kubernetes"></a><span data-ttu-id="762c2-103">Criando um pipeline de CI/CD para microsserviços no Kubernetes</span><span class="sxs-lookup"><span data-stu-id="762c2-103">Building a CI/CD pipeline for microservices on Kubernetes</span></span>

<span data-ttu-id="762c2-104">Ele pode ser um desafio para criar um processo de CI/CD confiável para uma arquitetura de microsserviços.</span><span class="sxs-lookup"><span data-stu-id="762c2-104">It can be challenging to create a reliable CI/CD process for a microservices architecture.</span></span> <span data-ttu-id="762c2-105">As equipes individuais devem ser capazes de liberar o serviços de forma rápida e confiável, sem interromper outras equipes ou desestabilizar o aplicativo como um todo.</span><span class="sxs-lookup"><span data-stu-id="762c2-105">Individual teams must be able to release services quickly and reliably, without disrupting other teams or destabilizing the application as a whole.</span></span>

<span data-ttu-id="762c2-106">Este artigo descreve um exemplo de pipeline de CI/CD para implantar microsserviços para o serviço de Kubernetes do Azure (AKS).</span><span class="sxs-lookup"><span data-stu-id="762c2-106">This article describes an example CI/CD pipeline for deploying microservices to Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="762c2-107">Cada equipe e projeto são diferente, tome neste artigo como um conjunto de regras rígidas e rápidas.</span><span class="sxs-lookup"><span data-stu-id="762c2-107">Every team and project is different, so don't take this article as a set of hard-and-fast rules.</span></span> <span data-ttu-id="762c2-108">Em vez disso, ele foi projetado para ser um ponto de partida para a criação de seu próprio processo de CI/CD.</span><span class="sxs-lookup"><span data-stu-id="762c2-108">Instead, it's meant to be a starting point for designing your own CI/CD process.</span></span>

<span data-ttu-id="762c2-109">O pipeline de exemplo descrito aqui foi criado para uma implementação de referência de microsserviços chamada o aplicativo de entrega por Drone, que pode ser encontrado na [GitHub][ri].</span><span class="sxs-lookup"><span data-stu-id="762c2-109">The example pipeline described here was created for a microservices reference implementation called the Drone Delivery application, which you can find on [GitHub][ri].</span></span> <span data-ttu-id="762c2-110">O cenário de aplicativo é descrito [aqui](./design/index.md#reference-implementation).</span><span class="sxs-lookup"><span data-stu-id="762c2-110">The application scenario is described [here](./design/index.md#reference-implementation).</span></span>

<span data-ttu-id="762c2-111">As metas do pipeline podem ser resumidas da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="762c2-111">The goals of the pipeline can be summarized as follows:</span></span>

- <span data-ttu-id="762c2-112">As equipes podem criar e implantar seus serviços de forma independente.</span><span class="sxs-lookup"><span data-stu-id="762c2-112">Teams can build and deploy their services independently.</span></span>
- <span data-ttu-id="762c2-113">Alterações de código que passam pelo processo de CI são implantadas automaticamente em um ambiente de produção.</span><span class="sxs-lookup"><span data-stu-id="762c2-113">Code changes that pass the CI process are automatically deployed to a production-like environment.</span></span>
- <span data-ttu-id="762c2-114">Restrições de qualidade são impostas em cada estágio do pipeline.</span><span class="sxs-lookup"><span data-stu-id="762c2-114">Quality gates are enforced at each stage of the pipeline.</span></span>
- <span data-ttu-id="762c2-115">Uma nova versão de um serviço pode ser implantada lado a lado com a versão anterior.</span><span class="sxs-lookup"><span data-stu-id="762c2-115">A new version of a service can be deployed side by side with the previous version.</span></span>

<span data-ttu-id="762c2-116">Para obter mais informações, consulte [CI/CD para arquiteturas de microsserviços](./ci-cd.md).</span><span class="sxs-lookup"><span data-stu-id="762c2-116">For more background, see [CI/CD for microservices architectures](./ci-cd.md).</span></span>

## <a name="assumptions"></a><span data-ttu-id="762c2-117">Suposições</span><span class="sxs-lookup"><span data-stu-id="762c2-117">Assumptions</span></span>

<span data-ttu-id="762c2-118">Para fins deste exemplo, aqui estão algumas suposições sobre a equipe de desenvolvimento e o código base:</span><span class="sxs-lookup"><span data-stu-id="762c2-118">For purposes of this example, here are some assumptions about the development team and the code base:</span></span>

- <span data-ttu-id="762c2-119">O repositório de código é um monorepo, com pastas organizadas por microsserviço.</span><span class="sxs-lookup"><span data-stu-id="762c2-119">The code repository is a monorepo, with folders organized by microservice.</span></span>
- <span data-ttu-id="762c2-120">A estratégia de ramificação da equipe se baseia no [desenvolvimento com base em troncos](https://trunkbaseddevelopment.com/).</span><span class="sxs-lookup"><span data-stu-id="762c2-120">The team's branching strategy is based on [trunk-based development](https://trunkbaseddevelopment.com/).</span></span>
- <span data-ttu-id="762c2-121">A equipe usa [branches de lançamento](/azure/devops/repos/git/git-branching-guidance?view=azure-devops#manage-releases) para gerenciar as liberações.</span><span class="sxs-lookup"><span data-stu-id="762c2-121">The team uses [release branches](/azure/devops/repos/git/git-branching-guidance?view=azure-devops#manage-releases) to manage releases.</span></span> <span data-ttu-id="762c2-122">Versões separadas são criadas para cada microsserviço.</span><span class="sxs-lookup"><span data-stu-id="762c2-122">Separate releases are created for each microservice.</span></span>
- <span data-ttu-id="762c2-123">Usa o processo de CI/CD [Pipelines do Azure](/azure/devops/pipelines/?view=azure-devops) para compilar, testar e implantar microsserviços no AKS.</span><span class="sxs-lookup"><span data-stu-id="762c2-123">The CI/CD process uses [Azure Pipelines](/azure/devops/pipelines/?view=azure-devops) to build, test, and deploy the microservices to AKS.</span></span>
- <span data-ttu-id="762c2-124">As imagens de contêiner para cada microsserviço são armazenadas no [registro de contêiner do Azure](/azure/container-registry/).</span><span class="sxs-lookup"><span data-stu-id="762c2-124">The container images for each microservice are stored in [Azure Container Registry](/azure/container-registry/).</span></span>
- <span data-ttu-id="762c2-125">A equipe usa gráficos Helm para empacotar cada microsserviço.</span><span class="sxs-lookup"><span data-stu-id="762c2-125">The team uses Helm charts to package each microservice.</span></span>

<span data-ttu-id="762c2-126">Essas suposições unidade muitos dos detalhes específicos do pipeline de CI/CD.</span><span class="sxs-lookup"><span data-stu-id="762c2-126">These assumptions drive many of the specific details of the CI/CD pipeline.</span></span> <span data-ttu-id="762c2-127">No entanto, a abordagem básica descrita aqui ser adaptado para outros processos, ferramentas e serviços, como Jenkins ou Hub do Docker.</span><span class="sxs-lookup"><span data-stu-id="762c2-127">However, the basic approach described here be adapted for other processes, tools, and services, such as Jenkins or Docker Hub.</span></span>

## <a name="validation-builds"></a><span data-ttu-id="762c2-128">Compilações de validação</span><span class="sxs-lookup"><span data-stu-id="762c2-128">Validation builds</span></span>

<span data-ttu-id="762c2-129">Suponha que um desenvolvedor está trabalhando em um microsserviço chamado serviço de entrega.</span><span class="sxs-lookup"><span data-stu-id="762c2-129">Suppose that a developer is working on a microservice called the Delivery Service.</span></span> <span data-ttu-id="762c2-130">Ao desenvolver um novo recurso, o desenvolvedor coloca código em um branch de recursos.</span><span class="sxs-lookup"><span data-stu-id="762c2-130">While developing a new feature, the developer checks code into a feature branch.</span></span> <span data-ttu-id="762c2-131">Por convenção, os branches de recursos recebem o nome de `feature/*`.</span><span class="sxs-lookup"><span data-stu-id="762c2-131">By convention, feature branches are named `feature/*`.</span></span>

![Fluxo de trabalho de CI/CD](./images/aks-cicd-1.png)

<span data-ttu-id="762c2-133">O arquivo de definição de compilação inclui um gatilho que filtra por nome da branch e o caminho de origem:</span><span class="sxs-lookup"><span data-stu-id="762c2-133">The build definition file includes a trigger that filters by the branch name and the source path:</span></span>

```yaml
trigger:
  batch: true
  branches:
    include:
    - master
    - feature/*
    - topic/*

    exclude:
    - feature/experimental/*
    - topic/experimental/*

  paths:
     include:
     - /src/shipping/delivery/
```

<span data-ttu-id="762c2-134">&#11162;Consulte a [arquivo de origem](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-validation.yml).</span><span class="sxs-lookup"><span data-stu-id="762c2-134">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-validation.yml).</span></span>

<span data-ttu-id="762c2-135">Usando essa abordagem, cada equipe pode ter seu próprio pipeline de build.</span><span class="sxs-lookup"><span data-stu-id="762c2-135">Using this approach, each team can have its own build pipeline.</span></span> <span data-ttu-id="762c2-136">Somente o código que é verificado no `/src/shipping/delivery` pasta dispara uma compilação ao serviço de entrega.</span><span class="sxs-lookup"><span data-stu-id="762c2-136">Only code that is checked into the `/src/shipping/delivery` folder triggers a build of the Delivery Service.</span></span> <span data-ttu-id="762c2-137">Push confirmações para uma ramificação que corresponde ao filtro dispara um build de CI.</span><span class="sxs-lookup"><span data-stu-id="762c2-137">Pushing commits to a branch that matches the filter triggers a CI build.</span></span> <span data-ttu-id="762c2-138">Agora, no fluxo de trabalho, a compilação de CI executa uma verificação mínima no código:</span><span class="sxs-lookup"><span data-stu-id="762c2-138">At this point in the workflow, the CI build runs some minimal code verification:</span></span>

1. <span data-ttu-id="762c2-139">Compile o código.</span><span class="sxs-lookup"><span data-stu-id="762c2-139">Build the code.</span></span>
1. <span data-ttu-id="762c2-140">Execute testes de unidade.</span><span class="sxs-lookup"><span data-stu-id="762c2-140">Run unit tests.</span></span>

<span data-ttu-id="762c2-141">O objetivo é manter os tempos de build curto, para que o desenvolvedor possa obter comentários rápidos.</span><span class="sxs-lookup"><span data-stu-id="762c2-141">The goal is to keep build times short, so the developer can get quick feedback.</span></span> <span data-ttu-id="762c2-142">Depois que o recurso está pronto para mesclar no mestre, o desenvolvedor abre uma PR.</span><span class="sxs-lookup"><span data-stu-id="762c2-142">Once the feature is ready to merge into master, the developer opens a PR.</span></span> <span data-ttu-id="762c2-143">Isso dispara outra compilação de CI que executa verificações adicionais:</span><span class="sxs-lookup"><span data-stu-id="762c2-143">This triggers another CI build that performs some additional checks:</span></span>

1. <span data-ttu-id="762c2-144">Compile o código.</span><span class="sxs-lookup"><span data-stu-id="762c2-144">Build the code.</span></span>
1. <span data-ttu-id="762c2-145">Execute testes de unidade.</span><span class="sxs-lookup"><span data-stu-id="762c2-145">Run unit tests.</span></span>
1. <span data-ttu-id="762c2-146">Compile a imagem de contêiner de tempo de execução.</span><span class="sxs-lookup"><span data-stu-id="762c2-146">Build the runtime container image.</span></span>
1. <span data-ttu-id="762c2-147">Execute verificações de vulnerabilidade na imagem.</span><span class="sxs-lookup"><span data-stu-id="762c2-147">Run vulnerability scans on the image.</span></span>

![Fluxo de trabalho de CI/CD](./images/aks-cicd-2.png)

> [!NOTE]
> <span data-ttu-id="762c2-149">Em repositórios de DevOps do Azure, você pode definir [políticas](/azure/devops/repos/git/branch-policies) para proteger branches.</span><span class="sxs-lookup"><span data-stu-id="762c2-149">In Azure DevOps Repos, you can define [policies](/azure/devops/repos/git/branch-policies) to protect branches.</span></span> <span data-ttu-id="762c2-150">Por exemplo, a política poderia exigir uma compilação de CI bem-sucedida e a aprovação de uma pessoa pertinente para fazer a mesclagem com o mestre.</span><span class="sxs-lookup"><span data-stu-id="762c2-150">For example, the policy could require a successful CI build plus a sign-off from an approver in order to merge into master.</span></span>

## <a name="full-cicd-build"></a><span data-ttu-id="762c2-151">Compilação de CI/CD completa</span><span class="sxs-lookup"><span data-stu-id="762c2-151">Full CI/CD build</span></span>

<span data-ttu-id="762c2-152">Em algum momento, a equipe estará pronta para implantar uma nova versão do Serviço de Entrega.</span><span class="sxs-lookup"><span data-stu-id="762c2-152">At some point, the team is ready to deploy a new version of the Delivery service.</span></span> <span data-ttu-id="762c2-153">O gerente de versão cria uma ramificação do mestre com esse padrão de nomenclatura: `release/<microservice name>/<semver>`.</span><span class="sxs-lookup"><span data-stu-id="762c2-153">The release manager creates a branch from master with this naming pattern: `release/<microservice name>/<semver>`.</span></span> <span data-ttu-id="762c2-154">Por exemplo, `release/delivery/v1.0.2`.</span><span class="sxs-lookup"><span data-stu-id="762c2-154">For example, `release/delivery/v1.0.2`.</span></span>

![Fluxo de trabalho de CI/CD](./images/aks-cicd-3.png)

<span data-ttu-id="762c2-156">A criação dessa ramificação dispara uma compilação CI completa que executa todas as etapas anteriores adição:</span><span class="sxs-lookup"><span data-stu-id="762c2-156">Creation of this branch triggers a full CI build that runs all of the previous steps plus:</span></span>

1. <span data-ttu-id="762c2-157">Enviar por push a imagem de contêiner ao registro de contêiner do Azure.</span><span class="sxs-lookup"><span data-stu-id="762c2-157">Push the container image to Azure Container Registry.</span></span> <span data-ttu-id="762c2-158">A imagem é marcada com o número de versão obtido do nome do branch.</span><span class="sxs-lookup"><span data-stu-id="762c2-158">The image is tagged with the version number taken from the branch name.</span></span>
2. <span data-ttu-id="762c2-159">Executar `helm package` para empacotar o gráfico do Helm para o serviço.</span><span class="sxs-lookup"><span data-stu-id="762c2-159">Run `helm package` to package the Helm chart for the service.</span></span> <span data-ttu-id="762c2-160">O gráfico também é marcado com um número de versão.</span><span class="sxs-lookup"><span data-stu-id="762c2-160">The chart is also tagged with a version number.</span></span>
3. <span data-ttu-id="762c2-161">Enviar por push o pacote do Helm para registro de contêiner.</span><span class="sxs-lookup"><span data-stu-id="762c2-161">Push the Helm package to Container Registry.</span></span>

<span data-ttu-id="762c2-162">Supondo que essa compilação for bem-sucedida, ele dispara um processo de implantação (CD) usando um Pipelines do Azure [pipeline de lançamento](/azure/devops/pipelines/release/what-is-release-management).</span><span class="sxs-lookup"><span data-stu-id="762c2-162">Assuming this build succeeds, it triggers a deployment (CD) process using an Azure Pipelines [release pipeline](/azure/devops/pipelines/release/what-is-release-management).</span></span> <span data-ttu-id="762c2-163">Esse pipeline tem as seguintes etapas:</span><span class="sxs-lookup"><span data-stu-id="762c2-163">This pipeline has the following steps:</span></span>

1. <span data-ttu-id="762c2-164">Implante o gráfico do Helm em um ambiente de controle de qualidade.</span><span class="sxs-lookup"><span data-stu-id="762c2-164">Deploy the Helm chart to a QA environment.</span></span>
1. <span data-ttu-id="762c2-165">Um aprovador confirma a aprovação antes que o pacote seja movido para a produção.</span><span class="sxs-lookup"><span data-stu-id="762c2-165">An approver signs off before the package moves to production.</span></span> <span data-ttu-id="762c2-166">Confira [Controle de implantação de versão usando aprovações](/azure/devops/pipelines/release/approvals/approvals).</span><span class="sxs-lookup"><span data-stu-id="762c2-166">See [Release deployment control using approvals](/azure/devops/pipelines/release/approvals/approvals).</span></span>
1. <span data-ttu-id="762c2-167">Remover marcas de formatação a imagem do Docker para o namespace de produção no registro de contêiner do Azure.</span><span class="sxs-lookup"><span data-stu-id="762c2-167">Retag the Docker image for the production namespace in Azure Container Registry.</span></span> <span data-ttu-id="762c2-168">Por exemplo, se a marca atual for `myrepo.azurecr.io/delivery:v1.0.2`, a marca de produção será `myrepo.azurecr.io/prod/delivery:v1.0.2`.</span><span class="sxs-lookup"><span data-stu-id="762c2-168">For example, if the current tag is `myrepo.azurecr.io/delivery:v1.0.2`, the production tag is `myrepo.azurecr.io/prod/delivery:v1.0.2`.</span></span>
1. <span data-ttu-id="762c2-169">Implante o gráfico do Helm para o ambiente de produção.</span><span class="sxs-lookup"><span data-stu-id="762c2-169">Deploy the Helm chart to the production environment.</span></span>

<span data-ttu-id="762c2-170">Mesmo em um monorepo, essas tarefas podem ser definidas para os microsserviços individuais, para que as equipes podem implantar com alta velocidade.</span><span class="sxs-lookup"><span data-stu-id="762c2-170">Even in a monorepo, these tasks can be scoped to individual microservices, so that teams can deploy with high velocity.</span></span> <span data-ttu-id="762c2-171">O processo tem algumas etapas manuais: Aprovar solicitações de pull, criar branches de versão e aprovar implantações no cluster de produção.</span><span class="sxs-lookup"><span data-stu-id="762c2-171">The process has some manual steps: Approving PRs, creating release branches, and approving deployments into the production cluster.</span></span> <span data-ttu-id="762c2-172">Essas etapas são manuais pela política &mdash; poderiam ser automatizadas se prefere a organização.</span><span class="sxs-lookup"><span data-stu-id="762c2-172">These steps are manual by policy &mdash; they could be automated if the organization prefers.</span></span>

## <a name="isolation-of-environments"></a><span data-ttu-id="762c2-173">Isolamento de ambientes</span><span class="sxs-lookup"><span data-stu-id="762c2-173">Isolation of environments</span></span>

<span data-ttu-id="762c2-174">Você terá vários ambientes com serviços implantados, incluindo ambientes de desenvolvimento, smoke test, testes de integração, testes de carga e, finalmente, produção.</span><span class="sxs-lookup"><span data-stu-id="762c2-174">You will have multiple environments where you deploy services, including environments for development, smoke testing, integration testing, load testing, and finally production.</span></span> <span data-ttu-id="762c2-175">Esses ambientes precisam de algum nível de isolamento.</span><span class="sxs-lookup"><span data-stu-id="762c2-175">These environments need some level of isolation.</span></span> <span data-ttu-id="762c2-176">No Kubernetes, você pode escolher entre o isolamento físico e o isolamento lógico.</span><span class="sxs-lookup"><span data-stu-id="762c2-176">In Kubernetes, you have a choice between physical isolation and logical isolation.</span></span> <span data-ttu-id="762c2-177">Isolamento físico significa implantar em clusters separados.</span><span class="sxs-lookup"><span data-stu-id="762c2-177">Physical isolation means deploying to separate clusters.</span></span> <span data-ttu-id="762c2-178">O isolamento lógico faz uso de namespaces e as políticas, conforme descrito anteriormente.</span><span class="sxs-lookup"><span data-stu-id="762c2-178">Logical isolation makes use of namespaces and policies, as described earlier.</span></span>

<span data-ttu-id="762c2-179">Nossa recomendação é criar um cluster de produção dedicado juntamente com um cluster separado para seus ambientes de desenvolvimento/teste.</span><span class="sxs-lookup"><span data-stu-id="762c2-179">Our recommendation is to create a dedicated production cluster along with a separate cluster for your dev/test environments.</span></span> <span data-ttu-id="762c2-180">Use o isolamento lógico para separar ambientes dentro do cluster de desenvolvimento/teste.</span><span class="sxs-lookup"><span data-stu-id="762c2-180">Use logical isolation to separate environments within the dev/test cluster.</span></span> <span data-ttu-id="762c2-181">Os serviços implantados no cluster de desenvolvimento/teste nunca devem ter acesso a armazenamentos de dados que contêm dados de negócios.</span><span class="sxs-lookup"><span data-stu-id="762c2-181">Services deployed to the dev/test cluster should never have access to data stores that hold business data.</span></span>

## <a name="build-process"></a><span data-ttu-id="762c2-182">Processo de compilação</span><span class="sxs-lookup"><span data-stu-id="762c2-182">Build process</span></span>

<span data-ttu-id="762c2-183">Quando possível, o processo de compilação do pacote em um contêiner do Docker.</span><span class="sxs-lookup"><span data-stu-id="762c2-183">When possible, package your build process into a Docker container.</span></span> <span data-ttu-id="762c2-184">Que permite que você crie seus artefatos de código usando o Docker, sem a necessidade de configurar o ambiente de compilação em cada computador de compilação.</span><span class="sxs-lookup"><span data-stu-id="762c2-184">That allows you to build your code artifacts using Docker, without needing to configure the build environment on each build machine.</span></span> <span data-ttu-id="762c2-185">Um processo de compilação em contêineres torna mais fácil para escalar horizontalmente o pipeline de CI, adicionando novos agentes de compilação.</span><span class="sxs-lookup"><span data-stu-id="762c2-185">A containerized build process makes it easy to scale out the CI pipeline by adding new build agents.</span></span> <span data-ttu-id="762c2-186">Além disso, qualquer desenvolvedor da equipe pode compilar o código simplesmente executando o contêiner de build.</span><span class="sxs-lookup"><span data-stu-id="762c2-186">Also, any developer on the team can build the code simply by running the build container.</span></span>

<span data-ttu-id="762c2-187">Ao usar builds de vários estágios no Docker, você pode definir o ambiente de compilação e a imagem de tempo de execução em um único Dockerfile.</span><span class="sxs-lookup"><span data-stu-id="762c2-187">By using multi-stage builds in Docker, you can define the build environment and the runtime image in a single Dockerfile.</span></span> <span data-ttu-id="762c2-188">Por exemplo, aqui está um Dockerfile que cria um aplicativo ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="762c2-188">For example, here's a Dockerfile that builds an ASP.NET Core application:</span></span>

```
FROM microsoft/dotnet:2.2-runtime AS base
WORKDIR /app

FROM microsoft/dotnet:2.2-sdk AS build
WORKDIR /src/Fabrikam.Workflow.Service

COPY Fabrikam.Workflow.Service/Fabrikam.Workflow.Service.csproj .
RUN dotnet restore Fabrikam.Workflow.Service.csproj

COPY Fabrikam.Workflow.Service/. .
RUN dotnet build Fabrikam.Workflow.Service.csproj -c Release -o /app

FROM build AS publish
RUN dotnet publish Fabrikam.Workflow.Service.csproj -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "Fabrikam.Workflow.Service.dll"]
```

<span data-ttu-id="762c2-189">&#11162;Consulte a [arquivo de origem](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/workflow/Dockerfile).</span><span class="sxs-lookup"><span data-stu-id="762c2-189">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/workflow/Dockerfile).</span></span>

<span data-ttu-id="762c2-190">Este Dockerfile define vários estágios de compilação.</span><span class="sxs-lookup"><span data-stu-id="762c2-190">This Dockerfile defines several build stages.</span></span> <span data-ttu-id="762c2-191">Observe que o estágio denominado `base` usa o tempo de execução do ASP.NET Core, durante o estágio chamado `build` usa o SDK do ASP.NET Core completo.</span><span class="sxs-lookup"><span data-stu-id="762c2-191">Notice that the stage named `base` uses the ASP.NET Core runtime, while the stage named `build` uses the full ASP.NET Core SDK.</span></span> <span data-ttu-id="762c2-192">O `build` estágio é usado para compilar o projeto do ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="762c2-192">The `build` stage is used to build the ASP.NET Core project.</span></span> <span data-ttu-id="762c2-193">Mas o contêiner de tempo de execução final é criado a partir `base`, que contém apenas o tempo de execução e é significativamente menor do que a imagem completa do SDK.</span><span class="sxs-lookup"><span data-stu-id="762c2-193">But the final runtime container is built from `base`, which contains just the runtime and is significantly smaller than the full SDK image.</span></span>

### <a name="building-a-test-runner"></a><span data-ttu-id="762c2-194">Criação de um executor de teste</span><span class="sxs-lookup"><span data-stu-id="762c2-194">Building a test runner</span></span>

<span data-ttu-id="762c2-195">Outra prática recomendada é executar testes de unidade no contêiner.</span><span class="sxs-lookup"><span data-stu-id="762c2-195">Another good practice is to run unit tests in the container.</span></span> <span data-ttu-id="762c2-196">Por exemplo, aqui está parte de um arquivo do Docker que cria um executor de teste:</span><span class="sxs-lookup"><span data-stu-id="762c2-196">For example, here is part of a Docker file that builds a test runner:</span></span>

```
FROM build AS testrunner
WORKDIR /src/tests

COPY Fabrikam.Workflow.Service.Tests/*.csproj .
RUN dotnet restore Fabrikam.Workflow.Service.Tests.csproj

COPY Fabrikam.Workflow.Service.Tests/. .
ENTRYPOINT ["dotnet", "test", "--logger:trx"]
```

<span data-ttu-id="762c2-197">Um desenvolvedor pode usar esse arquivo do Docker para executar os testes localmente:</span><span class="sxs-lookup"><span data-stu-id="762c2-197">A developer can use this Docker file to run the tests locally:</span></span>

```bash
docker build . -t delivery-test:1 --target=testrunner
docker run -p 8080:8080 delivery-test:1
```

<span data-ttu-id="762c2-198">O pipeline de CI também deve executar os testes como parte da etapa de verificação de compilação.</span><span class="sxs-lookup"><span data-stu-id="762c2-198">The CI pipeline should also run the tests as part of the build verification step.</span></span>

<span data-ttu-id="762c2-199">Observe que este arquivo usa o Docker `ENTRYPOINT` comando para executar os testes, não o Docker `RUN` comando.</span><span class="sxs-lookup"><span data-stu-id="762c2-199">Note that this file uses the Docker `ENTRYPOINT` command to run the tests, not the Docker `RUN` command.</span></span>

- <span data-ttu-id="762c2-200">Se você usar o `RUN` de comando, os testes executados sempre que você criar a imagem.</span><span class="sxs-lookup"><span data-stu-id="762c2-200">If you use the `RUN` command, the tests run every time you build the image.</span></span> <span data-ttu-id="762c2-201">Usando `ENTRYPOINT`, os testes são opt-in.</span><span class="sxs-lookup"><span data-stu-id="762c2-201">By using `ENTRYPOINT`, the tests are opt-in.</span></span> <span data-ttu-id="762c2-202">Eles executam somente quando você direcionar explicitamente o `testrunner` estágio.</span><span class="sxs-lookup"><span data-stu-id="762c2-202">They run only when you explicitly target the `testrunner` stage.</span></span>
- <span data-ttu-id="762c2-203">Um teste com falha não faz com que o Docker `build` comando falhe.</span><span class="sxs-lookup"><span data-stu-id="762c2-203">A failing test doesn't cause the Docker `build` command to fail.</span></span> <span data-ttu-id="762c2-204">Dessa forma, você pode distinguir o contêiner de falhas de falhas de teste da compilação.</span><span class="sxs-lookup"><span data-stu-id="762c2-204">That way you can distinguish container build failures from test failures.</span></span>
- <span data-ttu-id="762c2-205">Resultados de teste podem ser salvos em um volume montado.</span><span class="sxs-lookup"><span data-stu-id="762c2-205">Test results can be saved to a mounted volume.</span></span>

### <a name="container-best-practices"></a><span data-ttu-id="762c2-206">Práticas recomendadas do contêiner</span><span class="sxs-lookup"><span data-stu-id="762c2-206">Container best practices</span></span>

<span data-ttu-id="762c2-207">Aqui estão algumas práticas recomendadas a serem consideradas para contêineres:</span><span class="sxs-lookup"><span data-stu-id="762c2-207">Here are some other best practices to consider for containers:</span></span>

- <span data-ttu-id="762c2-208">Defina as convenções de toda a organização para marcações de contêiner, controle de versão e convenções de nomenclatura de recursos implantados para o cluster (pods, serviços e assim por diante).</span><span class="sxs-lookup"><span data-stu-id="762c2-208">Define organization-wide conventions for container tags, versioning, and naming conventions for resources deployed to the cluster (pods, services, and so on).</span></span> <span data-ttu-id="762c2-209">Isso pode facilitar o diagnóstico de problemas de implantação.</span><span class="sxs-lookup"><span data-stu-id="762c2-209">That can make it easier to diagnose deployment issues.</span></span>

- <span data-ttu-id="762c2-210">Durante o ciclo de desenvolvimento e teste, o processo de CI/CD cria muitas imagens de contêiner.</span><span class="sxs-lookup"><span data-stu-id="762c2-210">During the development and test cycle, the CI/CD process will build many container images.</span></span> <span data-ttu-id="762c2-211">Somente algumas dessas imagens são candidatos a lançamento e, em seguida, apenas alguns desses candidatos serão promovidos para a produção.</span><span class="sxs-lookup"><span data-stu-id="762c2-211">Only some of those images are candidates for release, and then only some of those release candidates will get promoted to production.</span></span> <span data-ttu-id="762c2-212">Ter uma estratégia de controle de versão clara, para que você saiba quais imagens estão implantadas atualmente para produção e podem reverter para uma versão anterior se necessário.</span><span class="sxs-lookup"><span data-stu-id="762c2-212">Have a clear versioning strategy, so that you know which images are currently deployed to production, and can roll back to a previous version if necessary.</span></span>

- <span data-ttu-id="762c2-213">Sempre implante marcas de versão de contêiner específico, não `latest`.</span><span class="sxs-lookup"><span data-stu-id="762c2-213">Always deploy specific container version tags, not `latest`.</span></span>

- <span data-ttu-id="762c2-214">Use [namespaces](/azure/container-registry/container-registry-best-practices#repository-namespaces) no registro de contêiner do Azure para isolar as imagens que foram aprovadas para a produção de imagens que ainda estão sendo testados.</span><span class="sxs-lookup"><span data-stu-id="762c2-214">Use [namespaces](/azure/container-registry/container-registry-best-practices#repository-namespaces) in Azure Container Registry to isolate images that are approved for production from images that are still being tested.</span></span> <span data-ttu-id="762c2-215">Não mova uma imagem para o namespace de produção até que você está pronto para implantá-lo em produção.</span><span class="sxs-lookup"><span data-stu-id="762c2-215">Don't move an image into the production namespace until you're ready to deploy it into production.</span></span> <span data-ttu-id="762c2-216">Se você combinar essa prática com controle de versão semântico de imagens de contêiner, isso poderá reduzir a chance de acidentalmente implantar uma versão não aprovada para lançamento.</span><span class="sxs-lookup"><span data-stu-id="762c2-216">If you combine this practice with semantic versioning of container images, it can reduce the chance of accidentally deploying a version that wasn't approved for release.</span></span>

- <span data-ttu-id="762c2-217">Siga o princípio de privilégios mínimos por contêineres em execução como um usuário sem privilégios.</span><span class="sxs-lookup"><span data-stu-id="762c2-217">Follow the principle of least privilege by running containers as a nonprivileged user.</span></span> <span data-ttu-id="762c2-218">No Kubernetes, você pode criar uma política de segurança pod que impede que a execução como contêineres *raiz*.</span><span class="sxs-lookup"><span data-stu-id="762c2-218">In Kubernetes, you can create a pod security policy that prevents containers from running as *root*.</span></span> <span data-ttu-id="762c2-219">Ver [impedir a execução com privilégios de raiz de Pods](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-cluster-psp/).</span><span class="sxs-lookup"><span data-stu-id="762c2-219">See [Prevent Pods From Running With Root Privileges](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-cluster-psp/).</span></span>

## <a name="helm-charts"></a><span data-ttu-id="762c2-220">Gráficos Helm</span><span class="sxs-lookup"><span data-stu-id="762c2-220">Helm charts</span></span>

<span data-ttu-id="762c2-221">Considere usar o Helm para gerenciar, criar e implantar serviços.</span><span class="sxs-lookup"><span data-stu-id="762c2-221">Consider using Helm to manage building and deploying services.</span></span> <span data-ttu-id="762c2-222">Aqui estão alguns dos recursos que ajudam com CI/CD do Helm:</span><span class="sxs-lookup"><span data-stu-id="762c2-222">Here are some of the features of Helm that help with CI/CD:</span></span>

- <span data-ttu-id="762c2-223">Geralmente, um único microsserviço é definido por vários objetos de Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="762c2-223">Often a single microservice is defined by multiple Kubernetes objects.</span></span> <span data-ttu-id="762c2-224">Helm permite que esses objetos para serem empacotados em um único gráfico do Helm.</span><span class="sxs-lookup"><span data-stu-id="762c2-224">Helm allows these objects to be packaged into a single Helm chart.</span></span>
- <span data-ttu-id="762c2-225">Um gráfico pode ser implantado com um único comando de Helm, em vez de uma série de comandos kubectl.</span><span class="sxs-lookup"><span data-stu-id="762c2-225">A chart can be deployed with a single Helm command, rather than a series of kubectl commands.</span></span>
- <span data-ttu-id="762c2-226">Os gráficos são explicitamente com controle de versão.</span><span class="sxs-lookup"><span data-stu-id="762c2-226">Charts are explicitly versioned.</span></span> <span data-ttu-id="762c2-227">Use o Helm para uma versão de lançamento, exibir versões e reverter para uma versão anterior.</span><span class="sxs-lookup"><span data-stu-id="762c2-227">Use Helm to release a version, view releases, and roll back to a previous version.</span></span> <span data-ttu-id="762c2-228">Acompanhar atualizações e revisões, usando controle de versão semântico, além da capacidade de reverter para uma versão anterior.</span><span class="sxs-lookup"><span data-stu-id="762c2-228">Tracking updates and revisions, using semantic versioning, along with the ability to roll back to a previous version.</span></span>
- <span data-ttu-id="762c2-229">Gráficos do Helm usam modelos para evitar a duplicação de informações, como rótulos e seletores, em vários arquivos.</span><span class="sxs-lookup"><span data-stu-id="762c2-229">Helm charts use templates to avoid duplicating information, such as labels and selectors, across many files.</span></span>
- <span data-ttu-id="762c2-230">Helm pode gerenciar as dependências entre gráficos.</span><span class="sxs-lookup"><span data-stu-id="762c2-230">Helm can manage dependencies between charts.</span></span>
- <span data-ttu-id="762c2-231">Gráficos podem ser armazenados em um repositório do Helm, como o registro de contêiner do Azure e pode ser integrados ao pipeline de build.</span><span class="sxs-lookup"><span data-stu-id="762c2-231">Charts can be stored in a Helm repository, such as Azure Container Registry, and integrated into the build pipeline.</span></span>

<span data-ttu-id="762c2-232">Para obter mais informações sobre como usar o Registro de Contêiner como um repositório do Helm, confira [Usar o Registro de Contêiner do Azure como um repositório do Helm para os gráficos do aplicativo](/azure/container-registry/container-registry-helm-repos).</span><span class="sxs-lookup"><span data-stu-id="762c2-232">For more information about using Container Registry as a Helm repository, see [Use Azure Container Registry as a Helm repository for your application charts](/azure/container-registry/container-registry-helm-repos).</span></span>

<span data-ttu-id="762c2-233">Um único microsserviço pode envolver vários arquivos de configuração do Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="762c2-233">A single microservice may involve multiple Kubernetes configuration files.</span></span> <span data-ttu-id="762c2-234">Atualização de um serviço pode significar a tocar em todos esses arquivos para atualizar seletores, rótulos e marcas de imagem.</span><span class="sxs-lookup"><span data-stu-id="762c2-234">Updating a service can mean touching all of these files to update selectors, labels, and image tags.</span></span> <span data-ttu-id="762c2-235">Helm os trata como um único pacote chamado de gráfico e permite que você atualize facilmente os arquivos YAML usando variáveis.</span><span class="sxs-lookup"><span data-stu-id="762c2-235">Helm treats these as a single package called a chart and allows you to easily update the YAML files by using variables.</span></span> <span data-ttu-id="762c2-236">Helm usa uma linguagem de modelo (com base em modelos de Go) para permitir que você grave arquivos de configuração com parâmetros YAML.</span><span class="sxs-lookup"><span data-stu-id="762c2-236">Helm uses a template language (based on Go templates) to let you write parameterized YAML configuration files.</span></span>

<span data-ttu-id="762c2-237">Por exemplo, aqui está parte de um arquivo YAML que define uma implantação:</span><span class="sxs-lookup"><span data-stu-id="762c2-237">For example, here's part of a YAML file that defines a deployment:</span></span>

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ include "package.fullname" . | replace "." "" }}
  labels:
    app.kubernetes.io/name: {{ include "package.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
  annotations:
    kubernetes.io/change-cause: {{ .Values.reason }}

...

  spec:
      containers:
      - name: &package-container_name fabrikam-package
        image: {{ .Values.dockerregistry }}/{{ .Values.image.repository }}:{{ .Values.image.tag }}
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        env:
        - name: LOG_LEVEL
          value: {{ .Values.log.level }}
```

<span data-ttu-id="762c2-238">&#11162;Consulte a [arquivo de origem](https://github.com/mspnp/microservices-reference-implementation/blob/master/charts/package/templates/package-deploy.yaml).</span><span class="sxs-lookup"><span data-stu-id="762c2-238">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/charts/package/templates/package-deploy.yaml).</span></span>

<span data-ttu-id="762c2-239">Você pode ver que o nome da implantação, rótulos e contêiner da especificação todos os parâmetros de modelo de uso, que são fornecidos no momento da implantação.</span><span class="sxs-lookup"><span data-stu-id="762c2-239">You can see that the deployment name, labels, and container spec all use template parameters, which are provided at deployment time.</span></span> <span data-ttu-id="762c2-240">Por exemplo, na linha de comando:</span><span class="sxs-lookup"><span data-stu-id="762c2-240">For example, from the command line:</span></span>

```bash
helm install $HELM_CHARTS/package/ \
     --set image.tag=0.1.0 \
     --set image.repository=package \
     --set dockerregistry=$ACR_SERVER \
     --namespace backend \
     --name package-v0.1.0
```

<span data-ttu-id="762c2-241">Embora seu pipeline de CI/CD poderia instalar um gráfico diretamente no Kubernetes, é recomendável criar um arquivo de gráfico (arquivo. tgz) e enviar por push o gráfico a um repositório do Helm, como o registro de contêiner do Azure.</span><span class="sxs-lookup"><span data-stu-id="762c2-241">Although your CI/CD pipeline could install a chart directly to Kubernetes, we recommend creating a chart archive (.tgz file) and pushing the chart to a Helm repository such as Azure Container Registry.</span></span> <span data-ttu-id="762c2-242">Para obter mais informações, consulte [baseadas em Docker do pacote de aplicativos nos gráficos do Helm em Pipelines do Azure](/azure/devops/pipelines/languages/helm?view=azure-devops).</span><span class="sxs-lookup"><span data-stu-id="762c2-242">For more information, see [Package Docker-based apps in Helm charts in Azure Pipelines](/azure/devops/pipelines/languages/helm?view=azure-devops).</span></span>

<span data-ttu-id="762c2-243">Considere implantar Helm para seu próprio namespace e usando o controle de acesso baseado em função (RBAC) para restringir quais namespaces pode implantar.</span><span class="sxs-lookup"><span data-stu-id="762c2-243">Consider deploying Helm to its own namespace and using role-based access control (RBAC) to restrict which namespaces it can deploy to.</span></span> <span data-ttu-id="762c2-244">Para obter mais informações, consulte [controle de acesso baseado em função](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control) na documentação do Helm.</span><span class="sxs-lookup"><span data-stu-id="762c2-244">For more information, see [Role-based Access Control](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control) in the Helm documentation.</span></span>

### <a name="revisions"></a><span data-ttu-id="762c2-245">Revisões</span><span class="sxs-lookup"><span data-stu-id="762c2-245">Revisions</span></span>

<span data-ttu-id="762c2-246">Gráficos do Helm sempre têm um número de versão, que deve ser usado [controle de versão semântico](https://semver.org/).</span><span class="sxs-lookup"><span data-stu-id="762c2-246">Helm charts always have a version number, which must use [semantic versioning](https://semver.org/).</span></span> <span data-ttu-id="762c2-247">Um gráfico também pode ter um `appVersion`.</span><span class="sxs-lookup"><span data-stu-id="762c2-247">A chart can also have an `appVersion`.</span></span> <span data-ttu-id="762c2-248">Este campo é opcional e não precisa estar relacionado para a versão do gráfico.</span><span class="sxs-lookup"><span data-stu-id="762c2-248">This field is optional, and doesn't have to be related to the chart version.</span></span> <span data-ttu-id="762c2-249">Algumas equipes talvez queira versões do aplicativo separadamente das atualizações para os gráficos.</span><span class="sxs-lookup"><span data-stu-id="762c2-249">Some teams might want to application versions separately from updates to the charts.</span></span> <span data-ttu-id="762c2-250">Mas uma abordagem mais simples é usar um número de versão, portanto, não há uma relação de 1:1 entre a versão de gráfico e a versão do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="762c2-250">But a simpler approach is to use one version number, so there's a 1:1 relation between chart version and application version.</span></span> <span data-ttu-id="762c2-251">Dessa forma, você pode armazenar um gráfico por versão e implantar facilmente a versão desejada:</span><span class="sxs-lookup"><span data-stu-id="762c2-251">That way, you can store one chart per release and easily deploy the desired release:</span></span>

```bash
helm install <package-chart-name> --version <desiredVersion>
```

<span data-ttu-id="762c2-252">Outra prática recomendada é fornecer uma anotação de causa de alterações no modelo de implantação:</span><span class="sxs-lookup"><span data-stu-id="762c2-252">Another good practice is to provide a change-cause annotation in the deployment template:</span></span>

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ include "delivery.fullname" . | replace "." "" }}
  labels:
     ...
  annotations:
    kubernetes.io/change-cause: {{ .Values.reason }}
```

<span data-ttu-id="762c2-253">Isso permite que você exiba o campo de causa de alteração para cada revisão, usando o `kubectl rollout history` comando.</span><span class="sxs-lookup"><span data-stu-id="762c2-253">This lets you view the change-cause field for each revision, using the `kubectl rollout history` command.</span></span> <span data-ttu-id="762c2-254">No exemplo anterior, a causa de alterações é fornecida como um parâmetro de gráfico do Helm.</span><span class="sxs-lookup"><span data-stu-id="762c2-254">In the previous example, the change-cause is provided as a Helm chart parameter.</span></span>

```bash
$ kubectl rollout history deployments/delivery-v010 -n backend
deployment.extensions/delivery-v010
REVISION  CHANGE-CAUSE
1         Initial deployment
```

<span data-ttu-id="762c2-255">Você também pode usar o `helm list` comando para exibir o histórico de revisão:</span><span class="sxs-lookup"><span data-stu-id="762c2-255">You can also use the `helm list` command to view the revision history:</span></span>

```bash
~$ helm list
NAME            REVISION    UPDATED                     STATUS        CHART            APP VERSION     NAMESPACE
delivery-v0.1.0 1           Sun Apr  7 00:25:30 2019    DEPLOYED      delivery-v0.1.0  v0.1.0          backend
```

> [!TIP]
> <span data-ttu-id="762c2-256">Use o `--history-max` sinalizar ao inicializar o Helm.</span><span class="sxs-lookup"><span data-stu-id="762c2-256">Use the `--history-max` flag when initializing Helm.</span></span> <span data-ttu-id="762c2-257">Essa configuração limita o número de revisões Tiller salva no seu histórico.</span><span class="sxs-lookup"><span data-stu-id="762c2-257">This setting limits the number of revisions that Tiller saves in its history.</span></span> <span data-ttu-id="762c2-258">Tiller armazena o histórico de revisão em configmaps.</span><span class="sxs-lookup"><span data-stu-id="762c2-258">Tiller stores revision history in configmaps.</span></span> <span data-ttu-id="762c2-259">Se você estiver liberando atualizações com frequência, o configmaps pode ficar muito grande, a menos que você limite o tamanho do histórico.</span><span class="sxs-lookup"><span data-stu-id="762c2-259">If you're releasing updates frequently, the configmaps can grow very large unless you limit the history size.</span></span>

## <a name="azure-devops-pipeline"></a><span data-ttu-id="762c2-260">Pipeline de DevOps do Azure</span><span class="sxs-lookup"><span data-stu-id="762c2-260">Azure DevOps Pipeline</span></span>

<span data-ttu-id="762c2-261">Em Pipelines do Azure, os pipelines são divididos em *compilar pipelines* e *liberar pipelines*.</span><span class="sxs-lookup"><span data-stu-id="762c2-261">In Azure Pipelines, pipelines are divided into *build pipelines* and *release pipelines*.</span></span> <span data-ttu-id="762c2-262">O pipeline de build executa o processo de CI e cria os artefatos de compilação.</span><span class="sxs-lookup"><span data-stu-id="762c2-262">The build pipeline runs the CI process and creates build artifacts.</span></span> <span data-ttu-id="762c2-263">Para uma arquitetura de microsserviços no Kubernetes, esses artefatos são as imagens de contêiner e os gráficos do Helm que definem cada microsserviço.</span><span class="sxs-lookup"><span data-stu-id="762c2-263">For a microservices architecture on Kubernetes, these artifacts are the container images and Helm charts that define each microservice.</span></span> <span data-ttu-id="762c2-264">O pipeline de lançamento executa esse processo de CD que implanta um microsserviço em um cluster.</span><span class="sxs-lookup"><span data-stu-id="762c2-264">The release pipeline runs that CD process that deploys a microservice into a cluster.</span></span>

<span data-ttu-id="762c2-265">Com base no fluxo de CI, descrito anteriormente neste artigo, um pipeline de compilação pode conter as seguintes tarefas:</span><span class="sxs-lookup"><span data-stu-id="762c2-265">Based on the CI flow described earlier in this article, a build pipeline might consist of the following tasks:</span></span>

1. <span data-ttu-id="762c2-266">Crie o contêiner de executor de teste.</span><span class="sxs-lookup"><span data-stu-id="762c2-266">Build the test runner container.</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        arguments: '--pull --target testrunner'
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        imageName: '$(imageName)-test'
    ```

1. <span data-ttu-id="762c2-267">Execute os testes, invocando o docker executar contra o contêiner de executor de teste.</span><span class="sxs-lookup"><span data-stu-id="762c2-267">Run the tests, by invoking docker run against the test runner container.</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        command: 'run'
        containerName: testrunner
        volumes: '$(System.DefaultWorkingDirectory)/TestResults:/app/tests/TestResults'
        imageName: '$(imageName)-test'
        runInBackground: false
    ```

1. <span data-ttu-id="762c2-268">Publica os resultados do teste.</span><span class="sxs-lookup"><span data-stu-id="762c2-268">Publish the test results.</span></span> <span data-ttu-id="762c2-269">Ver [integrar o build e testar tarefas](/azure/devops/pipelines/languages/docker?view=azure-devops&tabs=yaml#integrate-build-and-test-tasks).</span><span class="sxs-lookup"><span data-stu-id="762c2-269">See [Integrate build and test tasks](/azure/devops/pipelines/languages/docker?view=azure-devops&tabs=yaml#integrate-build-and-test-tasks).</span></span>

    ```yaml
    - task: PublishTestResults@2
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: 'TestResults/*.trx'
        searchFolder: '$(System.DefaultWorkingDirectory)'
        publishRunAttachments: true
    ```

1. <span data-ttu-id="762c2-270">Crie o contêiner de tempo de execução.</span><span class="sxs-lookup"><span data-stu-id="762c2-270">Build the runtime container.</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        includeLatestTag: false
        imageName: '$(imageName)'
    ```

1. <span data-ttu-id="762c2-271">Enviar por push para o contêiner ao registro de contêiner do Azure (ou outro registro de contêiner).</span><span class="sxs-lookup"><span data-stu-id="762c2-271">Push to the container to Azure Container Registry (or other container registry).</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        command: 'Push an image'
        imageName: '$(imageName)'
        includeSourceTags: false
    ```

1. <span data-ttu-id="762c2-272">O gráfico do Helm do pacote.</span><span class="sxs-lookup"><span data-stu-id="762c2-272">Package the Helm chart.</span></span>

    ```yaml
    - task: HelmDeploy@0
      inputs:
        command: package
        chartPath: $(chartPath)
        chartVersion: $(Build.SourceBranchName)
        arguments: '--app-version $(Build.SourceBranchName)'
    ```

1. <span data-ttu-id="762c2-273">Enviar por push o pacote do Helm para o registro de contêiner do Azure (ou outro repositório do Helm).</span><span class="sxs-lookup"><span data-stu-id="762c2-273">Push the Helm package to Azure Container Registry (or other Helm repository).</span></span>

    ```yaml
    task: AzureCLI@1
      inputs:
        azureSubscription: $(AzureSubscription)
        scriptLocation: inlineScript
        inlineScript: |
        az acr helm push $(System.ArtifactsDirectory)/$(repositoryName)-$(Build.SourceBranchName).tgz --name $(AzureContainerRegistry);
    ```

<span data-ttu-id="762c2-274">&#11162;Consulte a [arquivo de origem](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-ci.yml).</span><span class="sxs-lookup"><span data-stu-id="762c2-274">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-ci.yml).</span></span>

<span data-ttu-id="762c2-275">A saída do pipeline de CI é uma imagem de contêiner pronto para produção e um gráfico Helm atualizado para o microsserviço.</span><span class="sxs-lookup"><span data-stu-id="762c2-275">The output from the CI pipeline is a production-ready container image and an updated Helm chart for the microservice.</span></span> <span data-ttu-id="762c2-276">Neste ponto, o pipeline de lançamento pode assumir.</span><span class="sxs-lookup"><span data-stu-id="762c2-276">At this point, the release pipeline can take over.</span></span> <span data-ttu-id="762c2-277">Ele executa as seguintes etapas:</span><span class="sxs-lookup"><span data-stu-id="762c2-277">It performs the following steps:</span></span>

- <span data-ttu-id="762c2-278">Implante em ambientes de preparo/de controle de qualidade de desenvolvimento.</span><span class="sxs-lookup"><span data-stu-id="762c2-278">Deploy to dev/QA/staging environments.</span></span>
- <span data-ttu-id="762c2-279">Aguarde até que um aprovador aprovar ou rejeitar a implantação.</span><span class="sxs-lookup"><span data-stu-id="762c2-279">Wait for an approver to approve or reject the deployment.</span></span>
- <span data-ttu-id="762c2-280">Remover marcas de formatação na imagem de contêiner para lançamento</span><span class="sxs-lookup"><span data-stu-id="762c2-280">Retag the container image for release</span></span>
- <span data-ttu-id="762c2-281">Enviar por push a marca de versão para o registro de contêiner.</span><span class="sxs-lookup"><span data-stu-id="762c2-281">Push the release tag to the container registry.</span></span>
- <span data-ttu-id="762c2-282">Atualize o gráfico do Helm no cluster de produção.</span><span class="sxs-lookup"><span data-stu-id="762c2-282">Upgrade the Helm chart in the production cluster.</span></span>

<span data-ttu-id="762c2-283">Para obter mais informações sobre como criar um pipeline de lançamento, consulte [pipelines de lançamento, versões de rascunho e opções de versão](/azure/devops/pipelines/release/?view=azure-devops).</span><span class="sxs-lookup"><span data-stu-id="762c2-283">For more information about creating a release pipeline, see [Release pipelines, draft releases, and release options](/azure/devops/pipelines/release/?view=azure-devops).</span></span>

<span data-ttu-id="762c2-284">O diagrama a seguir mostra o processo de CI/CD de ponta a ponta descrito neste artigo:</span><span class="sxs-lookup"><span data-stu-id="762c2-284">The following diagram shows the end-to-end CI/CD process described in this article:</span></span>

![Pipeline de CD/CD](./images/aks-cicd-flow.png)

## <a name="next-steps"></a><span data-ttu-id="762c2-286">Próximos passos</span><span class="sxs-lookup"><span data-stu-id="762c2-286">Next steps</span></span>

<span data-ttu-id="762c2-287">Este artigo foi com base em uma implementação de referência que você pode encontrar na [GitHub][ri].</span><span class="sxs-lookup"><span data-stu-id="762c2-287">This article was based on a reference implementation that you can find on [GitHub][ri].</span></span>

[ri]: https://github.com/mspnp/microservices-reference-implementation