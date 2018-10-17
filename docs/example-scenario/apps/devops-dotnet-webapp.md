---
title: Pipeline de CI/CD com o Azure DevOps
description: Crie e libere um aplicativo .NET para aplicativos Web do Azure usando o Azure DevOps.
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: 80890784d4c97aac418cef4e49f9075dbef10b8a
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/05/2018
ms.locfileid: "48818931"
---
# <a name="cicd-pipeline-with-azure-devops"></a>Pipeline de CI/CD com o Azure DevOps

DevOps é a integração do desenvolvimento, da garantia de qualidade e das operações de TI. O DevOps requer uma cultura unificada e um conjunto robusto de processos para entrega de software.

Este cenário de exemplo demonstra como as equipes de desenvolvimento podem usar Azure DevOps para implantar um aplicativo Web de duas camadas.NET no Serviço de Aplicativo do Azure. O aplicativo Web depende dos serviços de PaaS (plataforma como serviço) do Azure downstream. Este documento também destaca algumas considerações que você deve fazer ao projetar um cenário como esse usando PaaS do Azure.

Adotar uma abordagem moderna para o desenvolvimento de aplicativos usando Integração Contínua e Implantação Contínua (CI/CD), ajuda a acelerar a entrega de valor para seus usuários por meio de um serviço de compilação, teste, implantação e monitoramento robusto. Usando uma plataforma como o Azure DevOps, juntamente com serviços do Azure como o Serviço de Aplicativo, as organizações podem se concentrar no desenvolvimento de seu cenário, em vez do gerenciamento da infraestrutura de suporte.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Considere usar o DevOps nos casos de uso abaixo:

* Aceleração do desenvolvimento de aplicativos e ciclos de vida de desenvolvimento
* Gerar qualidade e consistência em um processo de versão e build automatizado

## <a name="architecture"></a>Arquitetura

![Visão geral da arquitetura dos componentes do Azure envolvidos em um cenário de DevOps usando Azure DevOps e o Serviço de Aplicativo do Azure][architecture]

Este cenário aborda um pipeline de CI/CD para um aplicativo Web do .NET usando o Azure DevOps. O fluxo de dados neste cenário ocorre da seguinte forma:

1. Altere o código-fonte do aplicativo.
2. Confirme o código do aplicativo e o arquivo web.config de Aplicativos Web.
3. A integração contínua dispara o build do aplicativo e os testes de unidade.
4. O gatilho de implantação contínua orquestra a implantação de artefatos de aplicativo com *valores de configuração com parâmetros específicos do ambiente*.
5. Implantação do Serviço de Aplicativo do Azure.
6. O Application Insights do Azure coleta e analisa dados de integridade, de desempenho e de uso.
7. Examine as informações sobre integridade, desempenho e uso.

### <a name="components"></a>Componentes

* O [Azure DevOps][vsts] é um serviço para gerenciar seu ciclo de vida de desenvolvimento de ponta a ponta&mdash;, desde o planejamento e gerenciamento de projetos até o gerenciamento de código, bem como a compilação e o lançamento.
* Os [Aplicativos Web do Azure][web-apps] são um serviço de PaaS de hospedagem de aplicativos Web, APIs REST e back-ends móveis. Embora este artigo se concentre em .NET, há várias opções de plataforma de desenvolvimento adicionais com suporte.
* O [Application Insights][application-insights] é um serviço de gerenciamento de desempenho de aplicativo (APM), “first-party” extensível para desenvolvedores da Web em várias plataformas.

### <a name="alternative-devops-tooling-options"></a>Opções alternativas de ferramentas do DevOps

Embora este artigo se concentre no Azure DevOps, o [Team Foundation Server][team-foundation-server] poderia ser usado como substituto local. Como alternativa, você também pode usar um conjunto de tecnologias para um pipeline de desenvolvimento de software livre usando [Jenkins][jenkins-on-azure].

De uma perspectiva de infraestrutura como código, [modelos do Azure Resource Manager][arm-templates] são incluídos como parte do projeto do Azure DevOps, mas você pode considerar [Terraform][terraform] ou [Chef][chef]. Se você prefere uma implantação baseada em IaaS (infraestrutura como serviço) e exige o gerenciamento de configuração, você poderia considerar o [State Configuration da Automação do Azure][desired-state-configuration], o [Ansible][ansible] ou o [Chef][chef].

### <a name="alternatives-to-azure-web-apps"></a>Alternativas para aplicativos Web do Azure

Você pode considerar estas alternativas à hospedagem em Azure Web Apps:

* [Máquinas Virtuais do Azure][compare-vm-hosting] &mdash; Para cargas de trabalho que exigem um alto grau de controle, ou dependem dos componentes do sistema operacional e serviços que não são possíveis com Aplicativos Web (por exemplo, o Windows GAC ou COM).
* [Service Fabric][service-fabric] &mdash; uma boa opção se a arquitetura da carga de trabalho for voltada para componentes distribuídos que se beneficiam da implantação e execução em um cluster com um alto grau de controle. O Service Fabric também pode ser usado para hospedar contêineres.
* [Azure Functions][azure-functions] - uma abordagem eficaz sem servidor se a arquitetura de carga de trabalho é centralizada em torno de componentes distribuídos refinados, exigindo dependências mínimas, onde componentes individuais são necessários somente para execução sob demanda (não continuamente) e a orquestração dos componentes não é necessária.

Essa [árvore de decisão](/azure/architecture/guide/technology-choices/compute-decision-tree) pode ajudar na escolha do caminho certo para uma migração.

### <a name="devops"></a>DevOps

A **[Integração Contínua (CI)][continuous-integration]** mantém uma compilação estável, com vários desenvolvedores regularmente confirmando alterações pequenas e frequentes na base de código compartilhado. Como parte de seu pipeline de integração contínua, você deve:
* Confirme com frequência as alterações de código secundárias. Evite reunir em lote alterações maiores ou mais complexas que podem ser mais difíceis de mesclar com êxito.
* Realize testes de unidade dos componentes do aplicativo com cobertura de código suficiente, incluindo testes de caminhos com falha.
* Garanta que a compilação seja executada em relação ao branch mestre (ou tronco) compartilhado. Essa ramificação deve ser estável e mantida como “pronta para implantação”. As alterações incompletas ou em andamento devem ser isoladas em um branch separado com mesclagens frequentes de "integração para uso futuro" para evitar conflitos mais tarde.

A **[Entrega contínua (CD)][continuous-delivery]** demonstra não apenas uma compilação estável, mas também uma implantação estável. Isso torna a concretização da CD um pouco mais difícil, exigindo uma configuração específica para o ambiente, bem como um mecanismo para definir esses valores corretamente. Outras considerações de CD incluem o seguinte:
* A cobertura de teste de integração suficiente é necessária para validar que os vários componentes estão configurados e funcionam corretamente de ponta a ponta.
* A CD também pode exigir configuração e redefinição dos dados específicos do ambiente e o gerenciamento de versões de esquema de banco de dados.
* A entrega contínua também deve ser estendida para teste de carga e ambientes de teste de aceitação do usuário.
* A entrega contínua beneficia-se de monitoramento contínuo, idealmente em todos os ambientes.
* A consistência e a confiabilidade de implantações e teste de integração entre ambientes são facilitadas com scripts para a criação e a configuração da infraestrutura de hospedagem. Isso é consideravelmente mais fácil para cargas de trabalho baseadas em nuvem. Para saber mais, veja [Infraestrutura como código][infra-as-code].
* Inicie a entrega contínua o mais cedo possível no ciclo de vida do projeto. Quanto mais tarde você começar, mais difícil será incorporar.
* Integração e testes de unidade devem receber a mesma prioridade que os recursos de aplicativo.
* Use pacotes de implantação não relacionados ao ambiente e gerencie a configuração específica do ambiente por meio do processo de liberação.
* Proteja a configuração confidencial usando as ferramentas de gerenciamento de versão ou ao chamar um HSM, ou [Azure Key Vault][azure-key-vault], durante o processo de liberação. Não armazene configurações confidenciais dentro do controle do código-fonte.

**Aprendizado contínuo**. O monitoramento mais eficiente de um ambiente de CD é fornecido por ferramentas de APM (monitoramento do desempenho de aplicativo), como o [Application Insights][application-insights]. Uma capacidade de monitoramento detalhada o suficiente para uma carga de trabalho do aplicativo é fundamental para entender bugs ou o desempenho sob carga. O Application Insights pode ser integrado ao VSTS para habilitar o [monitoramento contínuo do pipeline de CD][app-insights-cd-monitoring]. Isso pode ser usado para habilitar a progressão automática para o próximo estágio, sem intervenção humana, ou a reversão se um alerta for detectado.

## <a name="considerations"></a>Considerações

### <a name="availability"></a>Disponibilidade

Considere a possibilidade de aproveitar os [padrões de design comuns para disponibilidade][design-patterns-availability] ao criar seu aplicativo de nuvem.

Analise as considerações de disponibilidade na [arquitetura de referência de aplicativo Web do Serviço de Aplicativo][app-service-reference-architecture] apropriada

Para ver outros tópicos sobre disponibilidade, consulte a [lista de verificação de disponibilidade][availability] no Azure Architecture Center.

### <a name="scalability"></a>Escalabilidade

Ao criar um aplicativo de nuvem esteja atento a [padrões de design comuns para escalabilidade][design-patterns-scalability].

Analise as considerações de escalabilidade na [arquitetura de referência de aplicativo Web do Serviço de Aplicativo][app-service-reference-architecture] apropriada

Para outros tópicos de escalabilidade, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.

### <a name="security"></a>Segurança

Considere a possibilidade de aproveitar os [padrões de design comuns de segurança][design-patterns-security] quando apropriado.

Analise as considerações de segurança na [arquitetura de referência de aplicativo Web do Serviço de Aplicativo][app-service-reference-architecture] apropriada.

Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].

### <a name="resiliency"></a>Resiliência

Analise os [padrões de design comuns para resiliência][design-patterns-resiliency] e considere a implementação deles quando apropriado.

Você pode encontrar várias [práticas recomendadas para o Serviço de Aplicativo][resiliency-app-service] no Centro de Arquitetura do Azure.

Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].

## <a name="deploy-the-scenario"></a>Implantar o cenário

### <a name="prerequisites"></a>Pré-requisitos

* Você deve ter uma conta do Azure já criada. Se você não tiver uma assinatura do Azure, crie uma [conta gratuita][azure-free-account] antes de começar.
* Você deve se inscrever em uma organização do Azure DevOps. Para saber mais, confira [Início Rápido: criar sua organização][vsts-account-create].

### <a name="walk-through"></a>Passo a passo

Neste cenário, você usará o projeto do Azure DevOps para criar seu pipeline de CI/CD.

O projeto de Azure DevOps implantará um Plano do Serviço de Aplicativo, um Serviço de Aplicativo e um recurso do Application Insights para você, além de configurar o projeto de Azure DevOps.

Depois que a implantação do projeto de Azure DevOps e do build for concluída, analise as alterações de código associadas, os itens de trabalho e os resultados de teste. Você observará que nenhum resultado de teste é exibido, pois o código não contém nenhum teste a ser executado.

Analise as definições de versão. Observe que um pipeline de lançamento foi configurado, liberando o nosso aplicativo para o ambiente de desenvolvimento. Observe que há um **gatilho de implantação contínua** definido no artefato de lançamento **Drop** com lançamentos automáticos em ambientes de desenvolvimento. Como parte de um processo de implantação contínua, você poderá ver versões que abrangem vários ambientes. Uma versão pode abranger infraestrutura (usando técnicas como infraestrutura como código) e também implantar os pacotes de aplicativos necessários, juntamente com quaisquer tarefas de pós-configuração.

## <a name="additional-considerations"></a>Considerações adicionais

* Considere aproveitar as [tarefas de geração de tokens][vsts-tokenization] que estão disponíveis no marketplace do VSTS.
* Considere usar a tarefa do VSTS [Implantar: Azure Key Vault][download-keyvault-secrets] para baixar segredos de um Azure Key Vault para sua versão. Você pode usar esses segredos como variáveis em sua definição de versão, para que possa evitar armazená-los no controle de origem.
* Considere usar as [variáveis de versão][vsts-release-variables] em suas definições de versão para fazer alterações de configuração de seus ambientes. As variáveis de versão podem ser definidas para uma versão inteira ou um determinado ambiente. Ao usar variáveis de informações secretas, selecione o ícone de cadeado.
* Considere o uso de [portões de implantação][vsts-deployment-gates] em seu pipeline de lançamento. Isso permite que você aproveite os dados de monitoramento junto com sistemas externos (por exemplo, gerenciamento de incidentes ou outros sistemas personalizados) para determinar se uma versão deve ser promovida.
* Quando a intervenção manual em um pipeline de lançamento for necessário, considere usar a funcionalidade de [aprovações][vsts-approvals].
* Considere usar o [Application Insights][application-insights] e outras ferramentas de monitoramento o mais cedo possível no seu pipeline de lançamento. Muitas organizações começa a monitorar apenas seu ambiente de produção. Monitorando seus outros ambientes, você pode identificar bugs desde o início no processo de desenvolvimento e evitar problemas em seu ambiente de produção.

## <a name="pricing"></a>Preços

Os custos do Azure DevOps dependem do número de usuários em sua organização que precisam de acesso, juntamente com outros fatores, como o número de builds/versões simultâneos necessários e o números de usuários de teste. Para saber mais, confira [Preços de Azure DevOps][vsts-pricing-page].

* [Azure DevOps][vsts-pricing-calculator] é um serviço que permite que você gerencie o ciclo de vida de desenvolvimento. Ele é pago por usuário e por mês. Pode haver cobranças adicionais, dependendo da necessidade de pipelines simultâneos, além de usuários de teste ou licenças de usuário básicas adicionais.

## <a name="related-resources"></a>Recursos relacionados

* [O que é DevOps?][devops-whatis]
* [DevOps na Microsoft ‒ como trabalhamos com o Azure DevOps][devops-microsoft]
* [Tutoriais passo a passo: DevOps com Azure DevOps][devops-with-vsts]
* [Lista de verificação de DevOps][devops-checklist]
* [Criar um pipeline de CI/CD para .NET com o Projeto de DevOps do Azure][devops-project-create]

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
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
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[terraform]: /azure/terraform/
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
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]: /azure/service-fabric/
[azure-functions]: /azure/azure-functions/
[azure-containers]: https://azure.microsoft.com/overview/containers/
[compare-vm-hosting]: /azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]: /azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]: /azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[security]: /azure/security/