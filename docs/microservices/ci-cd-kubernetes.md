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

# <a name="building-a-cicd-pipeline-for-microservices-on-kubernetes"></a>Criando um pipeline de CI/CD para microsserviços no Kubernetes

Ele pode ser um desafio para criar um processo de CI/CD confiável para uma arquitetura de microsserviços. As equipes individuais devem ser capazes de liberar o serviços de forma rápida e confiável, sem interromper outras equipes ou desestabilizar o aplicativo como um todo.

Este artigo descreve um exemplo de pipeline de CI/CD para implantar microsserviços para o serviço de Kubernetes do Azure (AKS). Cada equipe e projeto são diferente, tome neste artigo como um conjunto de regras rígidas e rápidas. Em vez disso, ele foi projetado para ser um ponto de partida para a criação de seu próprio processo de CI/CD.

O pipeline de exemplo descrito aqui foi criado para uma implementação de referência de microsserviços chamada o aplicativo de entrega por Drone, que pode ser encontrado na [GitHub][ri]. O cenário de aplicativo é descrito [aqui](./design/index.md#reference-implementation).

As metas do pipeline podem ser resumidas da seguinte maneira:

- As equipes podem criar e implantar seus serviços de forma independente.
- Alterações de código que passam pelo processo de CI são implantadas automaticamente em um ambiente de produção.
- Restrições de qualidade são impostas em cada estágio do pipeline.
- Uma nova versão de um serviço pode ser implantada lado a lado com a versão anterior.

Para obter mais informações, consulte [CI/CD para arquiteturas de microsserviços](./ci-cd.md).

## <a name="assumptions"></a>Suposições

Para fins deste exemplo, aqui estão algumas suposições sobre a equipe de desenvolvimento e o código base:

- O repositório de código é um monorepo, com pastas organizadas por microsserviço.
- A estratégia de ramificação da equipe se baseia no [desenvolvimento com base em troncos](https://trunkbaseddevelopment.com/).
- A equipe usa [branches de lançamento](/azure/devops/repos/git/git-branching-guidance?view=azure-devops#manage-releases) para gerenciar as liberações. Versões separadas são criadas para cada microsserviço.
- Usa o processo de CI/CD [Pipelines do Azure](/azure/devops/pipelines/?view=azure-devops) para compilar, testar e implantar microsserviços no AKS.
- As imagens de contêiner para cada microsserviço são armazenadas no [registro de contêiner do Azure](/azure/container-registry/).
- A equipe usa gráficos Helm para empacotar cada microsserviço.

Essas suposições unidade muitos dos detalhes específicos do pipeline de CI/CD. No entanto, a abordagem básica descrita aqui ser adaptado para outros processos, ferramentas e serviços, como Jenkins ou Hub do Docker.

## <a name="validation-builds"></a>Compilações de validação

Suponha que um desenvolvedor está trabalhando em um microsserviço chamado serviço de entrega. Ao desenvolver um novo recurso, o desenvolvedor coloca código em um branch de recursos. Por convenção, os branches de recursos recebem o nome de `feature/*`.

![Fluxo de trabalho de CI/CD](./images/aks-cicd-1.png)

O arquivo de definição de compilação inclui um gatilho que filtra por nome da branch e o caminho de origem:

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

&#11162;Consulte a [arquivo de origem](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-validation.yml).

Usando essa abordagem, cada equipe pode ter seu próprio pipeline de build. Somente o código que é verificado no `/src/shipping/delivery` pasta dispara uma compilação ao serviço de entrega. Push confirmações para uma ramificação que corresponde ao filtro dispara um build de CI. Agora, no fluxo de trabalho, a compilação de CI executa uma verificação mínima no código:

1. Compile o código.
1. Execute testes de unidade.

O objetivo é manter os tempos de build curto, para que o desenvolvedor possa obter comentários rápidos. Depois que o recurso está pronto para mesclar no mestre, o desenvolvedor abre uma PR. Isso dispara outra compilação de CI que executa verificações adicionais:

1. Compile o código.
1. Execute testes de unidade.
1. Compile a imagem de contêiner de tempo de execução.
1. Execute verificações de vulnerabilidade na imagem.

![Fluxo de trabalho de CI/CD](./images/aks-cicd-2.png)

> [!NOTE]
> Em repositórios de DevOps do Azure, você pode definir [políticas](/azure/devops/repos/git/branch-policies) para proteger branches. Por exemplo, a política poderia exigir uma compilação de CI bem-sucedida e a aprovação de uma pessoa pertinente para fazer a mesclagem com o mestre.

## <a name="full-cicd-build"></a>Compilação de CI/CD completa

Em algum momento, a equipe estará pronta para implantar uma nova versão do Serviço de Entrega. O gerente de versão cria uma ramificação do mestre com esse padrão de nomenclatura: `release/<microservice name>/<semver>`. Por exemplo, `release/delivery/v1.0.2`.

![Fluxo de trabalho de CI/CD](./images/aks-cicd-3.png)

A criação dessa ramificação dispara uma compilação CI completa que executa todas as etapas anteriores adição:

1. Enviar por push a imagem de contêiner ao registro de contêiner do Azure. A imagem é marcada com o número de versão obtido do nome do branch.
2. Executar `helm package` para empacotar o gráfico do Helm para o serviço. O gráfico também é marcado com um número de versão.
3. Enviar por push o pacote do Helm para registro de contêiner.

Supondo que essa compilação for bem-sucedida, ele dispara um processo de implantação (CD) usando um Pipelines do Azure [pipeline de lançamento](/azure/devops/pipelines/release/what-is-release-management). Esse pipeline tem as seguintes etapas:

1. Implante o gráfico do Helm em um ambiente de controle de qualidade.
1. Um aprovador confirma a aprovação antes que o pacote seja movido para a produção. Confira [Controle de implantação de versão usando aprovações](/azure/devops/pipelines/release/approvals/approvals).
1. Remover marcas de formatação a imagem do Docker para o namespace de produção no registro de contêiner do Azure. Por exemplo, se a marca atual for `myrepo.azurecr.io/delivery:v1.0.2`, a marca de produção será `myrepo.azurecr.io/prod/delivery:v1.0.2`.
1. Implante o gráfico do Helm para o ambiente de produção.

Mesmo em um monorepo, essas tarefas podem ser definidas para os microsserviços individuais, para que as equipes podem implantar com alta velocidade. O processo tem algumas etapas manuais: Aprovar solicitações de pull, criar branches de versão e aprovar implantações no cluster de produção. Essas etapas são manuais pela política &mdash; poderiam ser automatizadas se prefere a organização.

## <a name="isolation-of-environments"></a>Isolamento de ambientes

Você terá vários ambientes com serviços implantados, incluindo ambientes de desenvolvimento, smoke test, testes de integração, testes de carga e, finalmente, produção. Esses ambientes precisam de algum nível de isolamento. No Kubernetes, você pode escolher entre o isolamento físico e o isolamento lógico. Isolamento físico significa implantar em clusters separados. O isolamento lógico faz uso de namespaces e as políticas, conforme descrito anteriormente.

Nossa recomendação é criar um cluster de produção dedicado juntamente com um cluster separado para seus ambientes de desenvolvimento/teste. Use o isolamento lógico para separar ambientes dentro do cluster de desenvolvimento/teste. Os serviços implantados no cluster de desenvolvimento/teste nunca devem ter acesso a armazenamentos de dados que contêm dados de negócios.

## <a name="build-process"></a>Processo de compilação

Quando possível, o processo de compilação do pacote em um contêiner do Docker. Que permite que você crie seus artefatos de código usando o Docker, sem a necessidade de configurar o ambiente de compilação em cada computador de compilação. Um processo de compilação em contêineres torna mais fácil para escalar horizontalmente o pipeline de CI, adicionando novos agentes de compilação. Além disso, qualquer desenvolvedor da equipe pode compilar o código simplesmente executando o contêiner de build.

Ao usar builds de vários estágios no Docker, você pode definir o ambiente de compilação e a imagem de tempo de execução em um único Dockerfile. Por exemplo, aqui está um Dockerfile que cria um aplicativo ASP.NET Core:

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

&#11162;Consulte a [arquivo de origem](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/workflow/Dockerfile).

Este Dockerfile define vários estágios de compilação. Observe que o estágio denominado `base` usa o tempo de execução do ASP.NET Core, durante o estágio chamado `build` usa o SDK do ASP.NET Core completo. O `build` estágio é usado para compilar o projeto do ASP.NET Core. Mas o contêiner de tempo de execução final é criado a partir `base`, que contém apenas o tempo de execução e é significativamente menor do que a imagem completa do SDK.

### <a name="building-a-test-runner"></a>Criação de um executor de teste

Outra prática recomendada é executar testes de unidade no contêiner. Por exemplo, aqui está parte de um arquivo do Docker que cria um executor de teste:

```
FROM build AS testrunner
WORKDIR /src/tests

COPY Fabrikam.Workflow.Service.Tests/*.csproj .
RUN dotnet restore Fabrikam.Workflow.Service.Tests.csproj

COPY Fabrikam.Workflow.Service.Tests/. .
ENTRYPOINT ["dotnet", "test", "--logger:trx"]
```

Um desenvolvedor pode usar esse arquivo do Docker para executar os testes localmente:

```bash
docker build . -t delivery-test:1 --target=testrunner
docker run -p 8080:8080 delivery-test:1
```

O pipeline de CI também deve executar os testes como parte da etapa de verificação de compilação.

Observe que este arquivo usa o Docker `ENTRYPOINT` comando para executar os testes, não o Docker `RUN` comando.

- Se você usar o `RUN` de comando, os testes executados sempre que você criar a imagem. Usando `ENTRYPOINT`, os testes são opt-in. Eles executam somente quando você direcionar explicitamente o `testrunner` estágio.
- Um teste com falha não faz com que o Docker `build` comando falhe. Dessa forma, você pode distinguir o contêiner de falhas de falhas de teste da compilação.
- Resultados de teste podem ser salvos em um volume montado.

### <a name="container-best-practices"></a>Práticas recomendadas do contêiner

Aqui estão algumas práticas recomendadas a serem consideradas para contêineres:

- Defina as convenções de toda a organização para marcações de contêiner, controle de versão e convenções de nomenclatura de recursos implantados para o cluster (pods, serviços e assim por diante). Isso pode facilitar o diagnóstico de problemas de implantação.

- Durante o ciclo de desenvolvimento e teste, o processo de CI/CD cria muitas imagens de contêiner. Somente algumas dessas imagens são candidatos a lançamento e, em seguida, apenas alguns desses candidatos serão promovidos para a produção. Ter uma estratégia de controle de versão clara, para que você saiba quais imagens estão implantadas atualmente para produção e podem reverter para uma versão anterior se necessário.

- Sempre implante marcas de versão de contêiner específico, não `latest`.

- Use [namespaces](/azure/container-registry/container-registry-best-practices#repository-namespaces) no registro de contêiner do Azure para isolar as imagens que foram aprovadas para a produção de imagens que ainda estão sendo testados. Não mova uma imagem para o namespace de produção até que você está pronto para implantá-lo em produção. Se você combinar essa prática com controle de versão semântico de imagens de contêiner, isso poderá reduzir a chance de acidentalmente implantar uma versão não aprovada para lançamento.

- Siga o princípio de privilégios mínimos por contêineres em execução como um usuário sem privilégios. No Kubernetes, você pode criar uma política de segurança pod que impede que a execução como contêineres *raiz*. Ver [impedir a execução com privilégios de raiz de Pods](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-cluster-psp/).

## <a name="helm-charts"></a>Gráficos Helm

Considere usar o Helm para gerenciar, criar e implantar serviços. Aqui estão alguns dos recursos que ajudam com CI/CD do Helm:

- Geralmente, um único microsserviço é definido por vários objetos de Kubernetes. Helm permite que esses objetos para serem empacotados em um único gráfico do Helm.
- Um gráfico pode ser implantado com um único comando de Helm, em vez de uma série de comandos kubectl.
- Os gráficos são explicitamente com controle de versão. Use o Helm para uma versão de lançamento, exibir versões e reverter para uma versão anterior. Acompanhar atualizações e revisões, usando controle de versão semântico, além da capacidade de reverter para uma versão anterior.
- Gráficos do Helm usam modelos para evitar a duplicação de informações, como rótulos e seletores, em vários arquivos.
- Helm pode gerenciar as dependências entre gráficos.
- Gráficos podem ser armazenados em um repositório do Helm, como o registro de contêiner do Azure e pode ser integrados ao pipeline de build.

Para obter mais informações sobre como usar o Registro de Contêiner como um repositório do Helm, confira [Usar o Registro de Contêiner do Azure como um repositório do Helm para os gráficos do aplicativo](/azure/container-registry/container-registry-helm-repos).

Um único microsserviço pode envolver vários arquivos de configuração do Kubernetes. Atualização de um serviço pode significar a tocar em todos esses arquivos para atualizar seletores, rótulos e marcas de imagem. Helm os trata como um único pacote chamado de gráfico e permite que você atualize facilmente os arquivos YAML usando variáveis. Helm usa uma linguagem de modelo (com base em modelos de Go) para permitir que você grave arquivos de configuração com parâmetros YAML.

Por exemplo, aqui está parte de um arquivo YAML que define uma implantação:

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

&#11162;Consulte a [arquivo de origem](https://github.com/mspnp/microservices-reference-implementation/blob/master/charts/package/templates/package-deploy.yaml).

Você pode ver que o nome da implantação, rótulos e contêiner da especificação todos os parâmetros de modelo de uso, que são fornecidos no momento da implantação. Por exemplo, na linha de comando:

```bash
helm install $HELM_CHARTS/package/ \
     --set image.tag=0.1.0 \
     --set image.repository=package \
     --set dockerregistry=$ACR_SERVER \
     --namespace backend \
     --name package-v0.1.0
```

Embora seu pipeline de CI/CD poderia instalar um gráfico diretamente no Kubernetes, é recomendável criar um arquivo de gráfico (arquivo. tgz) e enviar por push o gráfico a um repositório do Helm, como o registro de contêiner do Azure. Para obter mais informações, consulte [baseadas em Docker do pacote de aplicativos nos gráficos do Helm em Pipelines do Azure](/azure/devops/pipelines/languages/helm?view=azure-devops).

Considere implantar Helm para seu próprio namespace e usando o controle de acesso baseado em função (RBAC) para restringir quais namespaces pode implantar. Para obter mais informações, consulte [controle de acesso baseado em função](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control) na documentação do Helm.

### <a name="revisions"></a>Revisões

Gráficos do Helm sempre têm um número de versão, que deve ser usado [controle de versão semântico](https://semver.org/). Um gráfico também pode ter um `appVersion`. Este campo é opcional e não precisa estar relacionado para a versão do gráfico. Algumas equipes talvez queira versões do aplicativo separadamente das atualizações para os gráficos. Mas uma abordagem mais simples é usar um número de versão, portanto, não há uma relação de 1:1 entre a versão de gráfico e a versão do aplicativo. Dessa forma, você pode armazenar um gráfico por versão e implantar facilmente a versão desejada:

```bash
helm install <package-chart-name> --version <desiredVersion>
```

Outra prática recomendada é fornecer uma anotação de causa de alterações no modelo de implantação:

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

Isso permite que você exiba o campo de causa de alteração para cada revisão, usando o `kubectl rollout history` comando. No exemplo anterior, a causa de alterações é fornecida como um parâmetro de gráfico do Helm.

```bash
$ kubectl rollout history deployments/delivery-v010 -n backend
deployment.extensions/delivery-v010
REVISION  CHANGE-CAUSE
1         Initial deployment
```

Você também pode usar o `helm list` comando para exibir o histórico de revisão:

```bash
~$ helm list
NAME            REVISION    UPDATED                     STATUS        CHART            APP VERSION     NAMESPACE
delivery-v0.1.0 1           Sun Apr  7 00:25:30 2019    DEPLOYED      delivery-v0.1.0  v0.1.0          backend
```

> [!TIP]
> Use o `--history-max` sinalizar ao inicializar o Helm. Essa configuração limita o número de revisões Tiller salva no seu histórico. Tiller armazena o histórico de revisão em configmaps. Se você estiver liberando atualizações com frequência, o configmaps pode ficar muito grande, a menos que você limite o tamanho do histórico.

## <a name="azure-devops-pipeline"></a>Pipeline de DevOps do Azure

Em Pipelines do Azure, os pipelines são divididos em *compilar pipelines* e *liberar pipelines*. O pipeline de build executa o processo de CI e cria os artefatos de compilação. Para uma arquitetura de microsserviços no Kubernetes, esses artefatos são as imagens de contêiner e os gráficos do Helm que definem cada microsserviço. O pipeline de lançamento executa esse processo de CD que implanta um microsserviço em um cluster.

Com base no fluxo de CI, descrito anteriormente neste artigo, um pipeline de compilação pode conter as seguintes tarefas:

1. Crie o contêiner de executor de teste.

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        arguments: '--pull --target testrunner'
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        imageName: '$(imageName)-test'
    ```

1. Execute os testes, invocando o docker executar contra o contêiner de executor de teste.

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

1. Publica os resultados do teste. Ver [integrar o build e testar tarefas](/azure/devops/pipelines/languages/docker?view=azure-devops&tabs=yaml#integrate-build-and-test-tasks).

    ```yaml
    - task: PublishTestResults@2
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: 'TestResults/*.trx'
        searchFolder: '$(System.DefaultWorkingDirectory)'
        publishRunAttachments: true
    ```

1. Crie o contêiner de tempo de execução.

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        includeLatestTag: false
        imageName: '$(imageName)'
    ```

1. Enviar por push para o contêiner ao registro de contêiner do Azure (ou outro registro de contêiner).

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        command: 'Push an image'
        imageName: '$(imageName)'
        includeSourceTags: false
    ```

1. O gráfico do Helm do pacote.

    ```yaml
    - task: HelmDeploy@0
      inputs:
        command: package
        chartPath: $(chartPath)
        chartVersion: $(Build.SourceBranchName)
        arguments: '--app-version $(Build.SourceBranchName)'
    ```

1. Enviar por push o pacote do Helm para o registro de contêiner do Azure (ou outro repositório do Helm).

    ```yaml
    task: AzureCLI@1
      inputs:
        azureSubscription: $(AzureSubscription)
        scriptLocation: inlineScript
        inlineScript: |
        az acr helm push $(System.ArtifactsDirectory)/$(repositoryName)-$(Build.SourceBranchName).tgz --name $(AzureContainerRegistry);
    ```

&#11162;Consulte a [arquivo de origem](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-ci.yml).

A saída do pipeline de CI é uma imagem de contêiner pronto para produção e um gráfico Helm atualizado para o microsserviço. Neste ponto, o pipeline de lançamento pode assumir. Ele executa as seguintes etapas:

- Implante em ambientes de preparo/de controle de qualidade de desenvolvimento.
- Aguarde até que um aprovador aprovar ou rejeitar a implantação.
- Remover marcas de formatação na imagem de contêiner para lançamento
- Enviar por push a marca de versão para o registro de contêiner.
- Atualize o gráfico do Helm no cluster de produção.

Para obter mais informações sobre como criar um pipeline de lançamento, consulte [pipelines de lançamento, versões de rascunho e opções de versão](/azure/devops/pipelines/release/?view=azure-devops).

O diagrama a seguir mostra o processo de CI/CD de ponta a ponta descrito neste artigo:

![Pipeline de CD/CD](./images/aks-cicd-flow.png)

## <a name="next-steps"></a>Próximos passos

Este artigo foi com base em uma implementação de referência que você pode encontrar na [GitHub][ri].

[ri]: https://github.com/mspnp/microservices-reference-implementation