---
title: Pipeline de CI/CD com VSTS
description: Um exemplo de criação e lançamento de um aplicativo .NET para Aplicativos Web do Azure
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: aea757087f4a505a8c52658abe1841c5455977cc
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389257"
---
# <a name="cicd-pipeline-with-vsts"></a>Pipeline de CI/CD com VSTS

DevOps é a integração do desenvolvimento, da garantia de qualidade e das operações de TI. O DevOps requer uma cultura unificada e um conjunto robusto de processos para entrega de software.

Este cenário de exemplo demonstra como as equipes de desenvolvimento podem usar o Visual Studio Team Services para implantar um aplicativo Web de duas camadas.NET no Serviço de Aplicativo do Azure. O aplicativo Web depende dos serviços de PaaS (plataforma como serviço) do Azure downstream. Este documento também destaca algumas considerações que você deve fazer ao projetar um cenário como esse usando a plataforma como um serviço (PaaS) do Azure.

Adotar uma abordagem moderna para o desenvolvimento de aplicativos usando CI (integração contínua) e CD (implantação contínua), ajuda a acelerar a entrega de valor para seus usuários por meio de um serviço de compilação, teste, implantação e monitoramento robusto. Usando uma plataforma como o Visual Studio Team Services, além dos serviços do Azure, como o Serviço de Aplicativo, as organizações podem garantir que elas permaneçam concentradas no desenvolvimento de seu cenário, em vez do gerenciamento da infraestrutura para habilitá-lo.

## <a name="related-use-cases"></a>Casos de uso relacionados

Considere usar o DevOps nos casos de uso abaixo:

* Acelerar o desenvolvimento de aplicativos e ciclos de vida de desenvolvimento
* Gerar qualidade e consistência em um processo de versão e build automatizado

## <a name="architecture"></a>Arquitetura

![Visão geral da arquitetura dos componentes do Azure envolvidos em um cenário de DevOps usando o Visual Studio Team Services e o Serviço de Aplicativo do Azure][architecture]

Este cenário aborda um pipeline de DevOps para um aplicativo Web .NET usando o Visual Studio Team Services (VSTS). O fluxo de dados neste cenário ocorre da seguinte forma:

1. Altere o código-fonte do aplicativo.
2. Confirme o código do aplicativo e o arquivo web.config de Aplicativos Web.
3. A integração contínua dispara o build do aplicativo e os testes de unidade.
4. O gatilho de implantação contínua orquestra a implantação de artefatos de aplicativo com *valores de configuração com parâmetros específicos do ambiente*.
5. Implantação do Serviço de Aplicativo do Azure.
6. O Application Insights do Azure coleta e analisa dados de integridade, de desempenho e de uso.
7. Examine as informações sobre integridade, desempenho e uso.

### <a name="components"></a>Componentes

* [Grupos de recursos][resource-groups] são um contêiner lógico para recursos do Azure e também fornecem um limite de controle de acesso para o plano de gerenciamento - pense em um grupo de recursos como a representação de uma “unidade de implantação”.
* [Visual Studio Team Services (VSTS)][vsts] é um serviço que permite que você gerencie o seu ciclo de vida de desenvolvimento de ponta a ponta; do planejamento e gerenciamento de projeto, ao gerenciamento de código, por meio de build e versão.
* [Aplicativos Web do Azure][web-apps] é uma PaaS (plataforma como serviço) para hospedar aplicativos Web, APIs REST e back-ends móveis. Embora este artigo se concentre em .NET, há várias opções de plataforma de desenvolvimento adicionais com suporte.
* O [Application Insights][application-insights] é um serviço de gerenciamento de desempenho de aplicativo (APM), “first-party” extensível para desenvolvedores da Web em várias plataformas.

### <a name="alternative-devops-tooling-options"></a>Opções alternativas de ferramentas do DevOps

Embora este artigo se concentre no Visual Studio Team Services, o [Team Foundation Server][team-foundation-server] poderia ser usado como um substituto local. Como alternativa, você também pode encontrar uma coleção de tecnologias que estão sendo usadas em conjunto para um pipeline de desenvolvimento de software livre usando [Jenkins][jenkins-on-azure].

De uma perspectiva de infraestrutura como código, [modelos do Azure Resource Manager][arm-templates] são incluídos como parte do projeto do Azure DevOps, mas você pode considerar [Terraform][terraform] ou [Chef][chef] se tiver investimentos aqui. Se você prefere uma implantação baseada em IaaS (infraestrutura como serviço) e exige o gerenciamento de configuração, você poderia considerar o [State Configuration da Automação do Azure][desired-state-configuration], o [Ansible][ansible] ou o [Chef][chef].

### <a name="alternatives-to-web-app-hosting"></a>Alternativas à hospedagem do aplicativo Web

Alternativas à hospedagem nos Aplicativos Web do Azure:

* [VM][compare-vm-hosting] – Para cargas de trabalho que exigem um alto grau de controle, ou dependem dos componentes do sistema operacional/serviços que não são possíveis com Aplicativos Web (por exemplo, o Windows GAC ou COM)
* [Hospedagem de contêiner][azure-containers] - Onde há dependências de sistema operacional e a portabilidade de hospedagem ou densidade de hospedagem, também são requisitos.
* [Service Fabric][service-fabric] - Uma boa opção se a arquitetura da carga de trabalho for voltada para componentes distribuídos que se beneficiam da implantação e execução em um cluster com um alto grau de controle. O Service Fabric também pode ser usado para hospedar contêineres.
* [Azure Functions sem servidor][azure-functions] - Uma boa opção se a arquitetura de carga de trabalho é centralizada em torno de componentes distribuídos refinados, exigindo dependências mínimas, onde componentes individuais são necessários somente para execução sob demanda (não continuamente) e a orquestração dos componentes não é necessária.

### <a name="devops"></a>DevOps

**[Integração contínua (CI)][continuous-integration]** deve ter como objetivo demonstrar uma compilação estável, com mais de um desenvolvedor ou equipe individual ou continuamente confirmando alterações pequenas e frequentes no código-fonte compartilhado.
Como parte de seu pipeline de integração contínua, você deve;

* Verificar pequenas quantidades de código com frequência (evite o envio em lote de alterações maiores ou mais complexas já que elas podem ser mais difíceis de serem mescladas com êxito)
* Faça o teste de unidade dos componentes do seu aplicativo com cobertura de código suficiente (incluindo os caminhos de falha)
* Garantindo que a compilação é executada em relação ao branch mestre (ou tronco) compartilhado. Essa ramificação deve ser estável e mantida como “pronta para implantação”. Alterações incompletas ou em andamento devem ser isoladas em um branch separado com mesclagens frequentes de “integração para uso futuro” para evitar conflitos mais tarde.

**[Entrega contínua (CD)][continuous-delivery]** deve ter como objetivo demonstrar não apenas uma compilação estável, mas uma implementação estável. Isso torna a concretização da CD um pouco mais difícil, uma configuração específica para o ambiente é necessária, bem como um mecanismo para definir esses valores corretamente.

Além disso, uma cobertura suficiente de testes de integração é necessária para garantir que os vários componentes estejam configurados e funcionando corretamente de ponta a ponta.

Isso também pode exigir configuração e redefinição dos dados específicos do ambiente e o gerenciamento de versões de esquema de banco de dados.

A entrega contínua também pode ser estendida para teste de carga e ambientes de teste de aceitação do usuário.

A entrega contínua beneficia-se de monitoramento contínuo, idealmente em todos os ambientes.
A consistência e a confiabilidade de implantações e teste de integração entre ambientes são facilitadas com um script para criação e configuração ou para a infraestrutura de hospedagem (algo que é consideravelmente mais fácil para cargas de trabalho baseadas em nuvem, veja a infraestrutura como código do Azure) – isso também é conhecido como [“infraestrutura como código”][infra-as-code].

* Inicie a entrega contínua o mais cedo possível no ciclo de vida do projeto. Quanto mais tarde você o inicia, mais difícil será.
* Integração e testes de unidade devem receber a mesma prioridade que os recursos de projeto
* Use pacotes de implantação não relacionados ao ambiente e gerencie a configuração específica do ambiente por meio do processo de liberação.
* Proteja a configuração confidencial dentro das ferramentas de gerenciamento de versão ou ao chamar um HSM, ou [Key Vault][azure-key-vault], durante o processo de liberação. Não armazene configurações confidenciais dentro do controle do código-fonte.

**Aprendizado contínuo** - O monitoramento mais eficiente de um ambiente de CD é fornecido pelas ferramentas de monitoramento de desempenho de aplicativos (APM de forma abreviada), por exemplo, o [Application Insights][application-insights] da Microsoft. Uma capacidade de monitoramento detalhada o suficiente para uma carga de trabalho do aplicativo é fundamental para entender bugs e o desempenho sob carga. [O App Insights pode ser integrado no VSTS para habilitar o monitoramento contínuo do pipeline de CD][app-insights-cd-monitoring]. Isso pode ser usado para habilitar a progressão automática para o próximo estágio, sem intervenção humana, ou a reversão se um alerta for detectado.

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
* Você deve ter uma conta existente do Visual Studio Team Services (VSTS). Saiba mais detalhes sobre [criar uma conta do Visual Studio Team Services (VSTS)][vsts-account-create].

### <a name="walk-through"></a>Passo a passo

Nesse cenário, você usará o projeto do Azure DevOps para criar seu pipeline de CI/CD.

O projeto de DevOps implantará um Plano de Serviço de Aplicativo, Serviço de Aplicativo e um recurso do App Insights para você, assim como configurar o projeto do Visual Studio Team Services para você.

Depois que a implantação do projeto de DevOps e do build for concluída, analise as alterações de código associadas, os itens de trabalho e os resultados de teste. Você observará que nenhum resultado de teste é exibido, pois o código não contém nenhum teste a ser executado.

Analise as Definições de versão. Observe que um pipeline de lançamento foi configurado, liberando o nosso aplicativo para desenvolvimento. Observe que há um **gatilho de implantação contínua** definido no artefato de lançamento **Drop** com lançamentos automáticos em ambientes de desenvolvimento. Como parte de um processo de implantação contínua, você poderá ver as versões em vários ambientes. Uma versão pode abranger infraestrutura (usando técnicas como infraestrutura como código) e também implantar os pacotes de aplicativos necessários, bem como quaisquer tarefas de pós-configuração.

**Considerações adicionais.**

* Considere aproveitar as [tarefas de geração de tokens][vsts-tokenization] que estão disponíveis no marketplace do VSTS.
* Considere usar a tarefa do VSTS [Implantar: Azure Key Vault] [ download-keyvault-secrets] para baixar segredos de um Azure Key Vault para sua versão. Você pode usar esses segredos como variáveis como parte de sua definição de versão e não deve armazená-los no controle do código-fonte.
* Considere usar as [variáveis de versão][vsts-release-variables] em suas definições de versão para fazer alterações de configuração de seus ambientes. As variáveis de versão podem ser definidas para uma versão inteira ou um determinado ambiente. Se usar variáveis de informações secretas, certifique-se de selecionar o ícone de cadeado.
* Considere o uso de [portões de implantação][vsts-deployment-gates] em seu pipeline de lançamento. Isso permite que você aproveite os dados de monitoramento junto com sistemas externos (por exemplo, gerenciamento de incidentes ou outros sistemas personalizados) para determinar se uma versão deve ser promovida.
* Quando a intervenção manual em um pipeline de lançamento for necessário, considere usar a funcionalidade de [aprovações][vsts-approvals].
* Considere usar o [Application Insights][application-insights] e outras ferramentas de monitoramento o mais cedo possível no seu pipeline de lançamento. A maioria das organizações inicia o monitoramento em seu ambiente de produção, embora você possa identificar possíveis bugs mais cedo no processo e impedir o impacto para os usuários em produção.

## <a name="pricing"></a>Preços

O custo do Visual Studio Team Services varia de acordo com o número de usuários em sua organização que exigem acesso, além de fatores como o número de build/versões simultâneas necessárias e o número de usuários de teste. Isso é detalhado mais adiante na [página de preços do VSTS][vsts-pricing-page].

* [Visual Studio Team Services (VSTS)][vsts-pricing-calculator] é um serviço que permite que você gerencie o ciclo de vida de desenvolvimento e é pago por cada usuário, por mês. Podem haver cobranças adicionais dependendo da necessidade de pipelines simultâneos, além de quaisquer usuários de teste, ou licenças de usuário básicas adicionais.

## <a name="related-resources"></a>Recursos relacionados

* [O que é DevOps?][devops-whatis]
* [DevOps na Microsoft - Como trabalhamos com o Visual Studio Team Services][devops-microsoft]
* [Tutoriais passo a passo: DevOps com o Visual Studio Team Services][devops-with-vsts]
* [Criar um pipeline de CI/CD para .NET com o Projeto de DevOps do Azure][devops-project-create]

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: /azure/architecture/reference-architectures/app-service-web-app/
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/devops-dotnet-webapp/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[application-insights]: https://azure.microsoft.com/en-gb/services/application-insights/
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
[vsts-account-create]: /vsts/organizations/accounts/create-account-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/en-us/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]:/azure/service-fabric/
[azure-functions]:/azure/azure-functions/
[azure-containers]:https://azure.microsoft.com/en-us/overview/containers/
[compare-vm-hosting]:/azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]:/azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]:/azure/best-practices-availability-paired-regions
[devops-project-create]: /vsts/pipelines/apps/cd/azure/azure-devops-project-aspnetcore?view=vsts
[security]: /azure/security/