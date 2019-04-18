---
title: Criar um pipeline de CI/CD usando o Azure DevOps
titleSuffix: Azure Example Scenarios
description: Crie e libere um aplicativo .NET para aplicativos Web do Azure usando o Azure DevOps.
author: christianreddington
ms.date: 12/06/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack, seodec18
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-devops-dotnet-webapp.svg
ms.openlocfilehash: f999b2ffdf234161f668887d5b2327ecf50f0e55
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/18/2019
ms.locfileid: "59740401"
---
# <a name="design-a-cicd-pipeline-using-azure-devops"></a>Criar um pipeline de CI/CD usando o Azure DevOps

Este cenário fornece orientações de arquitetura e design para a criação de um pipeline de CI (integração contínua) e CD (implantação contínua). Neste exemplo, o pipeline de CI/CD implanta um aplicativo Web do .NET de duas camadas para o Serviço de Aplicativo do Azure.

A migração para processos de CI/CD modernos fornece muitos benefícios para compilações, implantações, teste e monitoramento de aplicativos. Ao usar o Azure DevOps juntamente com outros serviços, como o Serviço de Aplicativo, as organizações podem se concentrar no desenvolvimento de seus aplicativos, em vez do gerenciamento da infraestrutura de suporte.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Considere o Azure DevOps e os processos de CI/CD para:

- Aceleração do desenvolvimento de aplicativos e ciclos de vida de desenvolvimento
- Gerar qualidade e consistência em um processo de versão e build automatizado
- Aumentar a estabilidade e o tempo de atividade do aplicativo

## <a name="architecture"></a>Arquitetura

![Diagrama da arquitetura dos componentes do Azure envolvidos em um cenário de DevOps usando Azure DevOps e o Serviço de Aplicativo do Azure][architecture]

O fluxo de dados neste cenário ocorre da seguinte forma:

1. Um desenvolvedor altera o código-fonte do aplicativo.
2. O código do aplicativo, incluindo o arquivo web.config, é confirmado no repositório de código-fonte em Azure Repos.
3. A integração contínua dispara o build do aplicativo e os testes de unidade usando o Azure Test Plans.
4. A implantação contínua dentro do Azure Pipelines dispara uma implantação automatizada de artefatos de aplicativo *com valores de configuração específicos ao ambiente*.
5. Os artefatos são implantados no Serviço de Aplicativo do Azure.
6. O Application Insights do Azure coleta e analisa dados de integridade, de desempenho e de uso.
7. Os desenvolvedores monitoram e gerenciam a integridade, o desempenho e o uso das informações.
8. As informações da lista de pendências são usadas para priorizar novos recursos e correções de bug usando Azure Boards.

### <a name="components"></a>Componentes

- O [Azure DevOps][vsts] é um serviço para gerenciar seu ciclo de vida de desenvolvimento de ponta a ponta&mdash;, desde o planejamento e gerenciamento de projetos até o gerenciamento de código, bem como a compilação e o lançamento.

- Os [Aplicativos Web do Azure][web-apps] são um serviço de PaaS de hospedagem de aplicativos Web, APIs REST e back-ends móveis. Embora este artigo se concentre em .NET, há várias opções de plataforma de desenvolvimento adicionais com suporte.

- O [Application Insights][application-insights] é um serviço de gerenciamento de desempenho de aplicativo (APM), “first-party” extensível para desenvolvedores da Web em várias plataformas.

### <a name="alternatives"></a>Alternativas

Embora este artigo se concentre no Azure DevOps, o [Azure DevOps Server][azure-devops-server] (conhecido como Team Foundation Server) poderia ser usado como substituto local. Como alternativa, você também pode usar um conjunto de tecnologias para um pipeline de desenvolvimento de software livre usando [Jenkins][jenkins-on-azure].

De uma perspectiva de infraestrutura como código, [modelos do Resource Manager][arm-templates] foram usados como parte do projeto do Azure DevOps, mas você pode considerar outras tecnologias de gerenciamento, como [Terraform][terraform] ou [Chef][chef]. Se você prefere uma implantação baseada em IaaS (infraestrutura como serviço) e exige o gerenciamento de configuração, você poderia considerar o [State Configuration da Automação do Azure][desired-state-configuration], o [Ansible][ansible] ou o [Chef][chef].

Você pode considerar estas alternativas à hospedagem em Azure Web Apps:

- As [Máquinas Virtuais do Azure][compare-vm-hosting] lidam com cargas de trabalho que exigem um alto grau de controle, ou dependem dos componentes do sistema operacional e serviços que não são possíveis com Aplicativos Web (por exemplo, o Windows GAC ou COM).

- O [Service Fabric][service-fabric] é uma boa opção se a arquitetura da carga de trabalho for voltada para componentes distribuídos que se beneficiam da implantação e execução em um cluster com um alto grau de controle. O Service Fabric também pode ser usado para hospedar contêineres.

- O [Azure Functions][azure-functions] fornece uma abordagem eficaz sem servidor se a arquitetura de carga de trabalho é centralizada em torno de componentes distribuídos refinados, exigindo dependências mínimas, onde componentes individuais são necessários somente para execução sob demanda (não continuamente) e a orquestração dos componentes não é necessária.

Essa [árvore de decisão para serviços de computação do Azure](/azure/architecture/guide/technology-choices/compute-decision-tree) pode ajudar na escolha do caminho certo para uma migração.

## <a name="management-and-security-considerations"></a>Considerações de gerenciamento e segurança

- Considere aproveitar as [tarefas de geração de tokens][vsts-tokenization] que estão disponíveis no marketplace do VSTS.

- As tarefas do [Azure Key Vault][download-keyvault-secrets] podem baixar segredos de um Azure Key Vault para sua versão. Você pode usar esses segredos como variáveis em sua definição de versão, o que evita armazená-los no controle do código-fonte.

- Use as [variáveis de versão][vsts-release-variables] em suas definições de versão para fazer alterações de configuração de seus ambientes. As variáveis de versão podem ser definidas para uma versão inteira ou um determinado ambiente. Ao usar variáveis de informações secretas, selecione o ícone de cadeado.

- Os [portões de implantação][vsts-deployment-gates] devem ser usados em seu pipeline de lançamento. Isso permite que você aproveite os dados de monitoramento junto com sistemas externos (por exemplo, gerenciamento de incidentes ou outros sistemas personalizados) para determinar se uma versão deve ser promovida.

- Quando a intervenção manual em um pipeline de lançamento for necessário, use a funcionalidade [aprovações][vsts-approvals].

- Considere usar o [Application Insights][application-insights] e outras ferramentas de monitoramento o mais cedo possível no seu pipeline de lançamento. Muitas organizações só começam o monitoramento em seus ambientes de produção. Ao monitorar outros ambientes, você pode identificar bugs desde o início do processo de desenvolvimento e evitar problemas em seu ambiente de produção.

## <a name="deploy-the-scenario"></a>Implantar o cenário

### <a name="prerequisites"></a>Pré-requisitos

- Você deve ter uma conta do Azure já criada. Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.

- Você deve se inscrever em uma organização do Azure DevOps. Para saber mais, consulte [Início Rápido: Criar sua organização][vsts-account-create].

### <a name="walk-through"></a>Passo a passo

[Projetos de DevOps do Azure](/azure/devops-project/azure-devops-project-github) será implantar um plano de serviço de aplicativo, serviço de aplicativo e um recurso do Application Insights para você, bem como configurar um pipeline de Pipelines do Azure para você.

Depois que você configurou um pipeline com projetos de DevOps do Azure e a compilação for concluída, examine associado alterações de código, itens de trabalho e os resultados de teste. Você observará que nenhum resultado de teste é exibido, pois o código não contém nenhum teste a ser executado.

O pipeline cria uma definição de versão e um gatilho de implantação contínua, implantar nosso aplicativo para o ambiente de desenvolvimento. Como parte de um processo de implantação contínua, você poderá ver versões que abrangem vários ambientes. Uma versão pode abranger infraestrutura (usando técnicas como infraestrutura como código) e também implantar os pacotes de aplicativos necessários, juntamente com quaisquer tarefas de pós-configuração.

## <a name="pricing"></a>Preços

Os custos do Azure DevOps dependem do número de usuários em sua organização que precisam de acesso, juntamente com outros fatores, como o número de builds/versões simultâneos necessários e o números de usuários de teste. Para saber mais, confira [Preços de Azure DevOps][vsts-pricing-page].

Essa [Calculadora de preços][vsts-pricing-calculator] fornece uma estimativa para a execução do Azure DevOps com 20 usuários.

O Azure DevOps é cobrado por mês e por usuário. Pode haver cobranças adicionais, dependendo da necessidade de pipelines simultâneos, além de usuários de teste ou licenças de usuário básicas adicionais.

## <a name="related-resources"></a>Recursos relacionados

Examine os seguintes recursos para saber mais sobre CI/CD e o Azure DevOps:

- [O que é DevOps?][devops-whatis]
- [DevOps na Microsoft ‒ como trabalhamos com o Azure DevOps][devops-microsoft]
- [Tutoriais passo a passo: DevOps com Azure DevOps][devops-with-vsts]
- [Lista de verificação de DevOps][devops-checklist]
- [Criar um pipeline de CI/CD para .NET com projetos de DevOps do Azure][devops-project-create]

<!-- links -->

[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
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
