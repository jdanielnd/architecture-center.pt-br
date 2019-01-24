---
title: Práticas recomendadas para empresas que estão migrando para o Azure
description: Descreve um andaime que as empresas podem usar para garantir um ambiente seguro e gerenciável.
author: rdendtler
ms.author: rodend
ms.date: 9/22/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 883f32b1533261977aa274f64c78762c9e7b13f3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54484407"
---
# <a name="azure-enterprise-scaffold-prescriptive-subscription-governance"></a>Andaime do Azure Enterprise: Governança de assinatura prescritiva

As empresas estão adotando cada vez mais a nuvem pública em busca de agilidade e flexibilidade. Elas utilizam os pontos fortes da nuvem para gerar receita e otimizar o uso de recursos para os negócios. O Microsoft Azure fornece diversos serviços e recursos que as empresas montam como blocos de construção para atender a uma ampla gama de aplicativos e cargas de trabalho.

Decidir usar o Microsoft Azure é apenas a primeira etapa para obter o benefício da nuvem. A segunda etapa é entender como a empresa pode usar o Azure efetivamente e identificar os recursos de linha de base que precisam estar em vigor para responder a perguntas como:

* “Estou preocupado com o domínio de dados; como posso garantir que meus dados e sistemas atendam aos nossos requisitos regulatórios?”
* "Como saber o que cada recurso aceita para que eu possa responsabilizá-lo e cobrá-lo de volta precisamente?"
* “Eu quero ter certeza de que tudo o que podemos implantar ou fazer na nuvem pública comece com a ideia da segurança em primeiro lugar; como ajudar a facilitar isso?"

A possibilidade de uma assinatura vazia sem recursos de segurança é assustadora. Esse espaço em branco pode atrasar sua migração para o Azure.

Este artigo fornece um ponto de partida para que profissionais técnicos atendam à necessidade da governança e a equilibrem com a necessidade da agilidade. Aqui é apresentado o conceito de um andaime empresarial que orienta as organizações na implementação e no gerenciamento de seus ambientes do Azure de uma maneira segura. O artigo fornece a estrutura para desenvolver controles eficazes e eficientes.

## <a name="need-for-governance"></a>Necessidade de governança

Ao migrar para o Azure, você deve abordar o tópico de governança antecipadamente para garantir o uso bem-sucedido da nuvem dentro da empresa. Infelizmente, o tempo e a burocracia necessários para criar um sistema abrangente de governança significam que alguns grupos de negócios chegam diretamente aos fornecedores sem se envolverem com a TI empresarial. Essa abordagem pode comprometer a empresa se os recursos não forem gerenciados corretamente. As características da nuvem pública — agilidade, flexibilidade e preço com base no consumo — são importantes para os grupos de negócios que precisam atender rapidamente às demandas dos clientes (internos e externos). Porém, a TI empresarial precisa garantir que sistemas e dados sejam protegidos eficazmente.

Ao construir um prédio, o andaime é usado para criar a base de uma estrutura. O andaime guia o plano geral, além de fornecer pontos de ancoragem para que mais sistemas permanentes sejam montados. Um andaime empresarial é igual: um conjunto de controles flexíveis e recursos do Azure que fornecem estrutura ao ambiente, bem como âncoras para serviços criados com base na nuvem pública. Ele fornece aos criadores (TI e grupos de negócios) uma base para criar e agregar novos serviços, mantendo a agilidade na entrega em mente.

Um andaime se baseia em práticas que coletamos dos muitos compromissos com clientes de vários tamanhos. Esses clientes variam de pequenas organizações que desenvolvem soluções na nuvem até grandes empresas multinacionais, bem como fornecedores de software independentes que estão migrando cargas de trabalho e desenvolvendo soluções nativas na nuvem. O andaime empresarial foi “criado especificamente” para ser flexível e aceitar cargas de TI tradicionais e cargas de trabalho ágeis; por exemplo, desenvolvedores que criam aplicativos SaaS (software como serviço) baseados em recursos da plataforma do Azure.

O andaime empresarial destina-se a ser a base de cada nova assinatura no Azure. Ele permite aos administradores garantir que as cargas de trabalho atendam aos requisitos mínimos de governança de uma organização sem impedir que grupos de negócios e desenvolvedores cumpram rapidamente suas próprias metas. Nossa experiência mostra que isso acelera bastante o crescimento da nuvem pública, em vez de impedi-lo.

> [!NOTE]
> A Microsoft lançou a versão prévia de um novo recurso chamado [Azure Blueprints](/azure/governance/blueprints/overview), que permitirá a você empacotar, gerenciar e implantar imagens, modelos, políticas e scripts comuns em assinaturas e grupos de gerenciamento. Esse recurso é a ponte entre a finalidade do andaime como modelo de referência e a implantação desse modelo na organização.
>
A imagem a seguir mostra os componentes do andaime. A base depende de um plano sólido para a hierarquia de gerenciamento e para as assinaturas. Os pilares consistem em políticas do Resource Manager e sólidos padrões de nomenclatura. O restante do andaime é composto pelos principais recursos e funcionalidades do Azure, que possibilitam e conectam um ambiente seguro e gerenciável.

![andaime empresarial](./_images/scaffoldv2.png)

## <a name="define-your-hierarchy"></a>Definir sua hierarquia

A base do andaime é a hierarquia e o relacionamento do Registro empresarial do Azure para assinaturas e grupos de recursos. O Registro empresarial define a forma e o uso dos serviços do Azure em uma empresa de um ponto de vista contratual. No contrato empresarial, os clientes podem subdividir o ambiente em departamentos, contas e, por fim, em assinaturas e grupos de recursos correspondentes à estrutura de suas respectivas organizações.

![hierarquia](./_images/agreement.png)

Uma assinatura do Azure é a unidade básica onde todos os recursos estão contidos. Ela também define vários limites dentro do Azure, como o número de núcleos, de redes virtuais e outros recursos. Os Grupos de Recursos do Azure são usados para refinar o modelo de assinatura ainda mais e permitir um agrupamento mais natural de recursos.

Cada empresa é diferente e a hierarquia na imagem acima permite flexibilidade significativa em como o Azure é organizado dentro da sua empresa. A primeira &mdash; e mais importante &mdash; decisão que você pode tomar ao começar a usar a nuvem pública é modelar sua hierarquia para refletir as necessidades da sua empresa para cobrança, gerenciamento e acesso a recursos.

### <a name="departments-and-accounts"></a>Departamentos e contas

Os três padrões comuns para inscrições do Azure são:

* O padrão **funcional**

    ![funcional](./_images/functional.png)
* O padrão da **unidade de negócios**

    ![negócios](./_images/business.png)
* O padrão **geográfico**

    ![geográfico](./_images/geographic.png)

Embora cada um desses padrões tenha o seu lugar, o padrão de **unidade de negócios** está sendo cada vez mais adotado devido à sua flexibilidade na criação de um modelo de custo da organização e porque ele reflete o alcance de controle. O grupo de Engenharia e Operações Fundamentais da Microsoft criou um subconjunto do padrão de **unidade de negócios** que é muito eficaz, modelado como **Federal**, **Estadual** e **Local**. (Para obter mais informações, consulte [Organizar assinaturas e grupos de recursos na empresa](https://azure.microsoft.com/blog/organizing-subscriptions-and-resource-groups-within-the-enterprise/).)

### <a name="management-groups"></a>Grupos de Gerenciamento

A Microsoft lançou recentemente uma nova maneira de modelar sua hierarquia: [Grupos de gerenciamento do Azure](/azure/azure-resource-manager/management-groups-overview). Os grupos de gerenciamento são muito mais flexíveis do que os departamentos e contas e podem ser aninhados em até seis níveis. Os grupos de gerenciamento permitem que você crie uma hierarquia separada da sua hierarquia de cobrança, exclusivamente para o gerenciamento eficiente de recursos. Os grupos de gerenciamento podem espelhar sua hierarquia de cobrança; e as empresas começam desse modo muitas vezes. No entanto, o poder dos grupos de gerenciamento surge quando você os utiliza para modelar a organização de modo que as assinaturas relacionadas &mdash; independentemente de onde elas estejam na hierarquia de cobrança, &mdash; sejam agrupadas e precisem de funções comuns atribuídas, assim como iniciativas e políticas. Alguns exemplos:

* **Produção/não produção**. Algumas empresas criam grupos de gerenciamento para identificar suas assinaturas de produção e não produção. Os grupos de gerenciamento permitem que esses clientes gerenciem mais facilmente funções e políticas, por exemplo: uma assinatura que não é de produção pode permitir aos desenvolvedores o acesso de “colaborador”, mas em uma de produção, eles têm apenas acesso de “leitor”.
* **Serviços internos/externos**. Assim como ocorre com o par produção/não produção, as empresas geralmente têm requisitos, políticas e funções diferentes para serviços internos e externos (voltados para o cliente).

Grupos de gerenciamento bem pensados são, juntamente com o Azure Policy e as Iniciativas do Azure, a espinha dorsal de uma governança eficiente do Azure.

### <a name="subscriptions"></a>Assinaturas

Ao decidir sobre seus departamentos e contas (ou grupos de gerenciamento), você está examinando principalmente como dividir seu ambiente do Azure para corresponder à sua organização. As assinaturas, no entanto, são onde o trabalho realmente acontece, e suas decisões aqui impactam a segurança, escalabilidade e cobrança.  Muitas organizações examinam os seguintes padrões como guias:

* **Aplicativo/serviço**: as assinaturas representam um aplicativo ou um serviço (portfólio de aplicativos)
* **Ciclo de vida**: as assinaturas representam um ciclo de vida de um serviço, como desenvolvimento ou produção.
* **Departamento**: as assinaturas representam departamentos na organização.

Os dois primeiros padrões são mais comumente utilizados, e ambos são altamente recomendáveis. A abordagem de Ciclo de vida é apropriada para a maioria das organizações. Nesse caso, a recomendação geral é usar duas assinaturas de base. “Produção” e “Não produção”, depois use grupos de recursos para subdividir ainda mais os ambientes.

### <a name="resource-groups"></a>Grupos de recursos

O Azure Resource Manager permite colocar recursos em grupos significativos para gerenciamento, cobrança ou afinidade natural. Os grupos de recursos são contêineres de recursos que têm um ciclo de vida comum ou compartilham um atributo, como "todos os servidores SQL" ou "Aplicativo A".

Os grupos de recursos não podem ser aninhados e os recursos podem pertencer apenas a um grupo de recursos. Algumas ações podem agir em todos os recursos em um grupo de recursos. Por exemplo, a exclusão de um grupo de recursos remove todos os recursos do grupo de recursos. Assim como as assinaturas, existem padrões comuns durante a criação de grupos de recursos e eles vão variar de cargas de trabalho de “TI tradicional” a cargas de trabalho de “TI Agile”:

* As cargas de trabalho de “TI Tradicional” são mais frequentemente agrupadas por itens no mesmo ciclo de vida, como um aplicativo. O agrupamento por aplicativo permite o gerenciamento individual de aplicativo.
* As cargas de trabalho "TI da Agile" tendem a se concentrar nos aplicativos de nuvem voltados para o cliente. Os grupos de recursos muitas vezes refletem as camadas da implantação (como Camada da Web, Camada de Aplicativo) e o gerenciamento.

> [!NOTE]
> Noções básicas sobre sua carga de trabalho ajudam a desenvolver uma estratégia de grupo de recursos. Esses padrões podem ser misturados e combinados. Por exemplo, um grupo de recursos de serviços compartilhados na mesma assinatura como grupos de recursos “Agile”.

## <a name="naming-standards"></a>Padrões de nomenclatura

O primeiro pilar do andaime é um padrão consistente de nomenclatura. Os padrões de nomenclatura bem definidos permitem identificar recursos no portal, em uma cobrança e dentro de scripts. Você provavelmente já tem padrões de nomenclatura para a infraestrutura local. Ao adicionar o Azure ao seu ambiente, você deve estender esses padrões de nomenclatura para os recursos do Azure.

> [!TIP]
> Para convenções de nomenclatura:
> * Analise e adote onde possível as [Orientações de padrões e práticas](https://docs.microsoft.com/en-us/azure/architecture/best-practices/naming-conventions). Essas diretrizes ajudam a decidir sobre um padrão de nomenclatura significativo e fornece exemplos abrangentes.
> * Usar políticas do Resource Manager para ajudar a impor padrões de nomenclatura
>
>Lembre-se de que é difícil alterar nomes mais tarde, portanto, alguns minutos agora evitarão problemas mais tarde.

Concentre-se nos padrões de nomenclatura desses recursos que são mais comumente usados e pesquisados.  Por exemplo, todos os grupos de recursos devem seguir um padrão bem definido para maior clareza.

### <a name="resource-tags"></a>Marcações de recursos

As marcas de recurso estão alinhadas diretamente com padrões de nomenclatura. Conforme recursos são adicionados às assinaturas, torna-se cada vez mais importante categorizá-los logicamente para fins operacionais, de gerenciamento e de cobrança. Para obter mais informações, veja [Usar marcas para organizar os recursos do Azure](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags).

> [!IMPORTANT]
> As marcas podem conter informações pessoais e podem estar enquadradas nos regulamentos do RGPD. Planeje cuidadosamente o gerenciamento de suas marcas. Se você estiver buscando informações gerais sobre o GDPR, confira a seção sobre GDPR do [Portal de Confiança do Serviço](https://servicetrust.microsoft.com/ViewPage/GDPRGetStarted).

As marcas são usadas de várias maneiras, além de gerenciamento e cobrança. Geralmente, eles são usados como parte da automação (consulte a seção posterior). Isso pode causar conflitos se não for considerado de antemão. A prática recomendada é identificar todas as marcas comuns no nível corporativo (por exemplo, ApplicationOwner, CostCenter) e aplicá-las de forma consistente ao implantar recursos usando a automação.

## <a name="azure-policy-and-initiatives"></a>Azure Policy e Iniciativas do Azure

O segundo pilar do andaime envolve o uso de [Azure Policy e Iniciativas do Azure](/azure/azure-policy/azure-policy-introduction) para gerenciar o risco pela imposição de regras (com efeitos) sobre os recursos e serviços em suas assinaturas. Iniciativas do Azure são coleções de políticas que são projetadas para alcançar uma única meta. O Azure Policy e as Iniciativas do Azure são atribuídos a um escopo de recursos para começar a imposição das diretivas especificadas.

O Azure Policy e as Iniciativas do Azure são ainda mais eficientes quando usadas com os grupos de gerenciamento mencionados anteriormente. Os grupos de gerenciamento permitem a atribuição de uma iniciativa ou política para um conjunto inteiro de assinaturas.

### <a name="common-uses-of-resource-manager-policies"></a>Usos comuns das políticas do Resource Manager

As políticas e iniciativas do Azure são uma poderosa ferramenta do kit de ferramentas do Azure. As políticas permitem às empresas fornecer controles para cargas de trabalho de “TI tradicional” que permitem a estabilidade necessária para aplicativos de LOB, permitindo também cargas de trabalho “Agile”; por exemplo, desenvolver aplicativos de cliente sem expor a empresa a riscos adicionais. Os padrões mais comuns que vemos para políticas são:

* **Domínio geográfico de conformidade/dados**. O Azure tem uma lista crescente de regiões em todo o mundo. As empresas geralmente precisam garantir que recursos em um determinado escopo permaneçam em uma região geográfica para atender aos requisitos regulatórios.
* **Evite expor publicamente os servidores**. O Azure Policy pode impedir a implantação de certos tipos de recursos. Um uso comum é criar uma política para negar a criação de um IP público dentro de um determinado escopo, evitando a exposição não intencional de um servidor à Internet.
* **Gerenciamento de Custos e Metadados**. As marcas de recurso são geralmente usadas para adicionar dados de cobrança importantes a recursos e grupos de recursos, como o CostCenter, Proprietário e muito mais. Essas marcas são imprescindíveis para uma cobrança e gerenciamento precisos dos recursos. As políticas podem impor a aplicação de marcas de recursos a todos os recursos implantados, facilitando o gerenciamento destes.

### <a name="common-uses-of-initiatives"></a>Usos comuns de iniciativas

A introdução de iniciativas ofereceu às empresas uma maneira de agrupar as políticas de lógicas e controlá-las como um todo. Iniciativas também oferecem suporte à empresa para atender às necessidades das cargas de trabalho “Agile” e “tradicionais”. Temos visto usos muito criativos de iniciativas, mas geralmente vemos:

* **Habilitar o monitoramento na Central de Segurança do Azure**. Essa é uma iniciativa de padrão no Azure Policy e um ótimo exemplo sobre o que são as iniciativas. Ela permite políticas que identificam os bancos de dados SQL não criptografados, vulnerabilidades relacionadas à VM e necessidades de segurança mais comuns.
* **Iniciativa específica regulatória**. As empresas geralmente agrupam políticas comuns em um requisito regulatório (como HIPAA) para que os controles e a conformidade a esses controles sejam acompanhados com eficiência.
* **Tipos de recurso e SKUs**. Criar uma iniciativa que restrinja os tipos de recursos que podem ser implantados, bem como as SKUs que podem ser implantadas pode ajudar a controlar os custos e garantir que sua organização esteja implantando somente recursos para os quais sua equipe possui o conjunto de qualificações e procedimentos para dar suporte.

> [!TIP]
> É recomendável que você use sempre definições de iniciativa em vez de definições de política. Depois de atribuir uma iniciativa a um escopo, como assinatura ou grupo de gerenciamento, você pode adicionar facilmente outra política à iniciativa sem precisar alterar as atribuições. Isso facilita bastante entender o que é aplicado e o acompanhamento da conformidade.

### <a name="policy-and-initiative-assignments"></a>Atribuições de Iniciativa e Política

Após criar políticas e agrupá-las em iniciativas lógicas você deve atribuir a política a um escopo, se ela um grupo de gerenciamento, uma assinatura ou até mesmo um grupo de recursos. As atribuições permitem que você também exclua um subescopo da atribuição de uma política. Por exemplo, se você negar a criação de IPs públicos em uma assinatura, você pode criar uma atribuição com uma exclusão para um grupo de recursos conectado à sua rede de perímetro protegida.

Você encontrará vários exemplos de política que mostram como iniciativas e políticas podem ser aplicadas a vários recursos no Azure neste repositório [GitHub](https://github.com/Azure/azure-policy).

## <a name="identity-and-access-management"></a>Gerenciamento de identidade e de acesso

Uma das primeira e mais importantes perguntas que você se faz ao começar a usar a nuvem pública é “quem devem ter acesso aos recursos?” e "como controlar esse acesso?" Permitir ou negar acesso ao portal do Azure, bem como controlar o acesso aos recursos no portal é essencial para o sucesso a longo prazo e para a segurança dos seus ativos na nuvem.

Para executar a tarefa de proteger o acesso aos seus recursos você primeiro configura seu provedor de identidade e, em seguida, configura o acesso e as funções. O Azure Active Directory (Azure AD), conectado ao seu Active Directory local, é a base da identidade do Azure Identity. Dito isto, o Azure AD *não* é o Active Directory e é importante entender o que é um locatário do Azure AD e como ele se relaciona ao seu registro no Azure.  Examine as [informações](../getting-started/azure-resource-access.md) disponíveis para ter uma base sólida sobre o Azure AD e o AD. Para conectar e sincronizar seu Active Directory ao Azure AD, instale e configure a [ferramenta AD Connect](/azure/active-directory/connect/active-directory-aadconnect) local.

![arch.png](./_images/arch.png)

Quando o Azure foi inicialmente lançado, os controles de acesso para uma assinatura eram básicos: Administrador ou Coadministrador. O acesso a uma assinatura no modelo Clássico implicava acesso a todos os recursos no portal. Essa falta de um controle refinado levou à proliferação de assinaturas para fornecer um nível de controle de acesso razoável para uma Inscrição no Azure. Essa proliferação de assinaturas não é mais necessária. Com o controle de acesso baseado em função (RBAC), você pode atribuir usuários às funções padrão que fornecem acesso comum, como “proprietário”, “colaborador” ou “leitor” ou até mesmo criar suas próprias funções

Ao implementar acesso baseado em função, o código a seguir é altamente recomendável:

* Controle o Administrador/Coadministrador de uma assinatura já que essas funções têm permissões abrangentes. Você só precisa adicionar o Proprietário da Assinatura como um Coadministrador caso ele precise gerenciar implantações clássicas do Azure.

* Use Grupos de Gerenciamento para atribuir [funções](/azure/azure-resource-manager/management-groups-overview#management-group-access) entre várias assinaturas e reduzir a carga de gerenciá-las no nível da assinatura.
* Adicione usuários do Azure a um grupo (por exemplo, Proprietários do Aplicativo X) no Active Directory. Use o grupo sincronizado para fornecer aos membros do grupo os direitos apropriados para gerenciar o grupo de recursos que contém o aplicativo.
* Siga o princípio de conceder o **privilégio mínimo** exigido para realizar o trabalho esperado.

> [!IMPORTANT]
>Considere o uso dos recursos de [Azure AD Privileged Identity Management](/azure/active-directory/privileged-identity-management/pim-configure), [Autenticação Multifator](/azure/active-directory/authentication/howto-mfa-getstarted) do Azure e [Acesso Condicional](/azure/active-directory/active-directory-conditional-access-azure-portal) para fornecer uma melhor segurança e mais visibilidade para ações administrativas em suas assinaturas do Azure. Esses recursos são provenientes de uma licença válida do Azure AD Premium (dependendo do recurso) para proteger ainda mais e gerenciar sua identidade. O Azure AD PIM habilita a acesso administrativo “Just In Time” com o fluxo de trabalho de aprovação, bem como uma auditoria completa de atividades e ativações do administrador. A MFA do Azure é outro recurso crítico e permite a verificação em duas etapas para fazer logon no portal do Azure. Combinados com Controles de Acesso Condicional você pode gerenciar com eficiência o risco de ameaça à segurança.

Planejar e preparar a sua identidade e os controles de acesso e seguir a prática recomendada do Azure Identity Management ([link](/azure/security/azure-security-identity-management-best-practices)) é uma das melhores estratégias de mitigação de risco que você pode empregar e deve ser considerada obrigatória para cada implantação.

## <a name="security"></a>Segurança

Talvez um dos maiores impedimentos à adoção da nuvem de forma tradicional tenham sido as preocupações com a segurança. Os gerentes de risco em TI e os departamentos de segurança precisam garantir que os recursos no Azure estejam protegidos e seguros por padrão. O Azure fornece uma série de recursos que você pode aproveitar para proteger os recursos e detectar/impedir ameaças em relação a esses recursos.

### <a name="azure-security-center"></a>Central de Segurança do Azure

A [Central de Segurança do Azure](/azure/security-center/security-center-intro) oferece uma exibição unificada do status de segurança dos recursos em seu ambiente, além de proteção avançada contra ameaças. A Central de Segurança do Azure é uma plataforma aberta que permite aos parceiros da Microsoft criar software que se conecte à Central de Segurança do Azure e aprimorar seus recursos. Os recursos de linha de base da Central de Segurança do Azure (camada gratuita) oferecem avaliação e recomendações que aprimorarão sua postura em relação à segurança. Suas camadas pagas habilitam recursos adicionais e valiosos, como acesso de administração Just In Time e controles de aplicativo adaptáveis (lista de permissões).

> [!TIP]
>A Central de Segurança do Azure é uma ferramenta muito poderosa que é constantemente aprimorada e incorpora sempre novos recursos que você pode aproveitar para detectar ameaças e proteger sua empresa. É altamente recomendável sempre habilitar ASC.

### <a name="azure-resource-locks"></a>Bloqueios de recursos do Azure

À medida que sua organização adiciona serviços principais à assinatura, torna-se cada vez mais importante evitar interrupção nos negócios. Um tipo de interrupção que vemos com frequência é consequências não intencionais de scripts e ferramentas trabalhando contra uma assinatura do Azure, excluindo recursos por engano. Os [Bloqueios de recursos](/azure/azure-resource-manager/resource-group-lock-resources) permitem restringir operações nos recursos de alto valor, onde sua modificação ou exclusão teria um impacto significativo. Os bloqueios são aplicados a uma assinatura, grupo de recursos ou até mesmo recursos individuais. O caso de uso comum é aplicar bloqueios a recursos fundamentais, como redes virtuais, gateways, grupos de segurança de rede e contas de armazenamento de chaves.

### <a name="secure-devops-toolkit"></a>Kit de ferramentas de DevOps seguro

O “Kit de Segurança do DevOps para Azure” (AzSK) é uma coleção de scripts, ferramentas, extensões, automações, etc. originalmente criado pela equipe de TI da própria Microsoft e lançado como software livre através do Github ([link](https://github.com/azsk/DevOpsKit-docs)). O AzSK cuida das necessidades de segurança de ponta a ponta e a segurança dos recursos e da assinatura do Azure para equipes que usam automação extensiva e integra perfeitamente a segurança em fluxos de trabalho do DevOps nativo ajudando a proteger o DevOps essas áreas de 6 enfoque:

* Proteger a assinatura
* Permitir o desenvolvimento seguro
* Integrar segurança para CI/CD
* Garantia contínua
* Alertas e monitoramento
* Governança de risco na nuvem

![Kit de ferramentas do Azure DevOps](_images/Secure_DevOps_Kit_Azure.png)

O AzSK é um conjunto avançado de ferramentas, scripts e informações que são uma parte importante de um plano de governança completo do Azure e incorporar isso no seu andaime é crucial para dar suporte a suas metas de gerenciamento de riscos de organizações

### <a name="azure-update-management"></a>Gerenciamento de Atualizações do Azure

Uma das principais tarefas que podem ser feitas para proteger o ambiente é garantir que os servidores tenham as atualizações mais recentes. Embora haja várias ferramentas para fazer isso, o Azure fornece a solução de [Gerenciamento de Atualizações do Azure](/azure/automation/automation-update-management) para tratar da identificação e da distribuição de patches críticos do sistema operacional.  Ele faz uso da Automação do Azure (que é abordado na seção [Automatizar](#automate) mais adiante neste guia.

## <a name="monitor-and-alerts"></a>Monitoramento e alertas

Coletar e analisar a telemetria que fornece uma linha de visão sobre as atividades, métricas de desempenho, integridade e disponibilidade dos serviços que você está usando em todas as suas assinaturas do Azure é essencial para gerenciar proativamente seus aplicativos e infraestrutura e é uma necessidade fundamental de cada assinatura do Azure. Cada serviço do Azure emite a telemetria na forma de Logs de atividades, Métricas e Logs de diagnóstico.

* Os **Log de atividades** descrevem todas as operações executadas nos recursos em suas assinaturas
* As **Métricas** são informações numéricas emitidas de dentro de um recurso que descrevem o desempenho e a integridade de um recurso
* Os **Logs de diagnóstico** são emitidos por um serviço do Azure e fornecem dados avançados e frequentes sobre a operação deste serviço.

Essas informações podem ser exibidas e tratadas em vários níveis e estão sendo continuamente aperfeiçoadas. O Azure oferece recursos de monitoramento **compartilhado**, **principal** e **profundo** dos recursos do Azure por meio dos serviços descritos no diagrama a seguir.
![monitoramento](./_images/monitoring.png)

### <a name="shared-capabilities"></a>Funcionalidades compartilhadas

* **Alertas**: você pode coletar cada log, eventos e métricas de recursos do Azure, mas sem a capacidade de ser notificado sobre condições críticas e de como atuar. Esses dados serão úteis somente para fins de histórico e análise forense. O Azure Alerts notifica proativamente sobre condições que você define para todos os seus aplicativos e infraestrutura. Você pode criar regras de alerta nos logs, eventos e métricas que usam grupos de ações para notificar os conjuntos de destinatários. Os grupos de ação também fornecem a capacidade de automatizar a correção usando ações externas, como webhooks para executar runbooks de Automação do Azure e o Azure Functions.

* **Painéis**: os painéis permitem que você agregue exibições de monitoramento e combine dados de recursos e assinaturas para dar a você uma visão de toda a empresa sobre a telemetria dos recursos do Azure. Você pode criar e configurar seus próprios modos de exibição e compartilhá-los com outras pessoas. Por exemplo, você poderia criar um painel composto por vários blocos para que os DBAs possam fornecer informações em todos os serviços de banco de dados do Azure, incluindo o BD SQL do Azure, banco de dados do Azure para PostgreSQL e o banco de dados do Azure para MySQL.

* **Metrics Explorer**: métricas são valores numéricos gerados pelos recursos do Azure (por exemplo, porcentagem de CPU, E/S de disco), que fornecem informações sobre a operação e o desempenho dos seus recursos. Usando o Metrics Explorer, você pode definir e enviar as métricas no qual você está interessado ao Log Analytics para agregação e análise.

### <a name="core-monitoring"></a>Monitoramento principal

* **Azure Monitor**: é o serviço de plataforma principal que fornece uma única fonte para monitorar os recursos do Azure. A interface do Portal do Azure do Azure Monitor fornece um ponto de partida centralizado para todos os recursos de monitoramento no Azure, incluindo os recursos de monitoramento profundo do Application Insights, Log Analytics, Monitor de Rede, Soluções de Gerenciamento e Mapas do Serviço. Com o Azure Monitor, é possível visualizar, consultar, rotear, arquivar e executar ações nas métricas e nos logs provenientes dos recursos do Azure em toda a sua nuvem. Além do portal, você pode recuperar dados por meio de cmdlets do PowerShell do Monitor, da CLI de plataforma cruzada ou das APIs REST do Azure Monitor.

* **Assistente do Azure**: o Assistente do Azure monitora a telemetria em suas assinaturas e ambientes constantemente e fornece recomendações sobre as melhores práticas sobre como otimizar seus recursos do Azure para economizar dinheiro e melhorar o desempenho, segurança e disponibilidade dos recursos que compõem seus aplicativos.

* **Integridade do Serviço**: a Integridade do Serviço do Azure identifica os problemas com os Serviços do Azure que podem afetar seus aplicativos, além de ajudar no planejamento de janelas de manutenção agendada.

* **Log de atividades**: o Log de atividades descreve todas as operações executadas nos recursos em suas assinaturas. Ele fornece uma trilha de auditoria para determinar “o que”, “quem” e “quando” de qualquer operação de criação, atualização e exclusão de recursos. Os eventos do Log de Atividades são armazenados na plataforma e ficam disponíveis para consulta por 90 dias. Você pode ingerir os Logs de atividade para o Log Analytics para maiores períodos de retenção e consultas e análises mais profundas em vários recursos.

### <a name="deep-application-monitoring"></a>Monitoramento profundo de aplicativos

* **Application Insights**: o Application Insights permite que você colete telemetria específica do aplicativo e monitore o desempenho, a disponibilidade e o uso de aplicativos na nuvem ou locais. Ao instrumentar seu aplicativo com os SDKs com suporte para várias linguagens, incluindo .NET, JavaScript, JAVA, Node.js, Ruby e Python. Os eventos do Application Insights são ingeridos no mesmo armazenamento de dados do Log Analytics que dá suporte à infraestrutura e monitoramento de segurança para que você possa correlacionar e agregar eventos ao longo do tempo por meio de uma linguagem de consulta avançada.

### <a name="deep-infrastructure-monitoring"></a>Monitoramento profundo de infraestrutura

* **Log Analytics**: o Log Analytics desempenha um papel central no monitoramento do Azure ao coletar a telemetria e outros dados de diversas fontes e fornecer um mecanismo de linguagem de consulta e de análise que fornece informações sobre a operação de seus aplicativos e recursos. Você pode interagir diretamente com os dados do Log Analytics por meio de exibições e pesquisas de log de alto desempenho ou usar as ferramentas de análise em outros serviços do Azure que armazenam seus dados no Log Analytics, como o Application Insights e a Central de Segurança do Azure.

* **Monitoramento de rede**: os serviços de monitoramento de rede do Azure permitem que você obtenha informações sobre o fluxo de tráfego de rede, desempenho, segurança, conectividade e gargalos. Um design de rede bem planejado deve incluir a configuração de serviços de monitoramento de rede do Azure, como o Observador de Rede e o Monitor do ExpressRoute.

* **Soluções de gerenciamento**: as soluções de gerenciamento são conjuntos empacotados de lógica, insights e consultas do Log Analytics predefinidas para um aplicativo ou serviço. Elas dependem do Log Analytics como base para armazenar e analisar dados de evento. Os exemplos de soluções de gerenciamento incluem o monitoramento de contêineres e análise de Banco de Dados SQL do Azure.

* **Mapa do Serviço**: o Mapa do Serviço fornece uma exibição gráfica em seus componentes de infraestrutura, seus processos e interdependências em outros computadores e processos externos. Ele integra soluções de gerenciamento, eventos e dados de desempenho ao Log Analytics.

> [!TIP]
> Antes de criar alertas individuais, crie e mantenha um conjunto compartilhado de Grupos de Ação que pode ser usado para os Alertas do Azure. Isso permitirá que você mantenha centralmente o ciclo de vida de listas de destinatários, métodos de entrega de notificação (email, números de telefone para SMS) e webhooks para ações externas (runbooks de Automação do Azure, Azure Functions / Aplicativos Lógicos, ITSM).

## <a name="cost-management"></a>Gerenciamento de custo

Uma das principais alterações que você enfrentará mudar da nuvem local para a nuvem pública é a mudança de gastos de capital (comprar hardware) para gastos operacionais (pagar por serviço conforme você o utiliza). Essa mudança de CAPEX para OPEX também traz a necessidade de gerenciar com mais cuidado seus custos. O benefício da nuvem é que você pode afetar essencial e positivamente o custo de um serviço usado apenas desativando-o (ou redimensionando-o) quando ele não for necessário. Deliberadamente gerenciar seus custos na nuvem é uma prática recomendada e que os clientes maduros fazem diariamente.

A Microsoft fornece várias ferramentas para você poder visualizar, controlar e gerenciar seus custos. Nós também fornecemos um conjunto completo de APIs para que você possa personalizar e integrar o gerenciamento de custos em suas próprias ferramentas e painéis. Essas ferramentas são agrupadas livremente em: Recursos do Portal do Azure e recursos externos

### <a name="azure-portal-capabilities"></a>Recursos do Portal do Azure

Essas são ferramentas para fornecer informações instantâneas sobre custo, bem como a capacidade de executar ações

* **Custo do Recurso de Assinatura**: localizada no Portal, a exibição da [Análise de Custo do Azure](/azure/cost-management/overview) fornece uma visão geral de seus custos e informações sobre os gastos diários por recurso ou grupo de recursos.
* **Gerenciamento de Custos do Azure**: este produto é o resultado da compra da Cloudyn pela Microsoft e permite que você gerencie e analise seus gastos com o Azure e também o que você gasta em outros provedores de nuvem pública. Há uma camada gratuita e outra paga, com uma grande variedade de recursos conforme visto na [visão geral](/azure/cost-management/overview).
* **Grupos de Ações e Azure Orçamentos**: saber quanto algo custa e fazer algo sobre isso, até recentemente, era mais um trabalho manual. Com a introdução do Azure Orçamentos e suas APIs, agora é possível criar ações (como visto [neste](https://channel9.msdn.com/Shows/Azure-Friday/Managing-costs-with-the-Azure-Budgets-API-and-Action-Groups) exemplo) quando os custos atingem um limite. Por exemplo, desligar um grupo de recursos de “teste” quando este atinge 100% de seu orçamento ou [outro exemplo].
* **Assistente do Azure**: saber quanto algo custa é apenas a metade; a outra metade é saber o que fazer com essas informações. O [Assistente do Azure](/azure/advisor/advisor-overview) fornece recomendações sobre as ações necessárias para economizar dinheiro, melhorar a confiabilidade ou até mesmo aumentar a segurança.

### <a name="external-cost-management-tools"></a>Ferramentas de gerenciamento de custos externas

* **Azure Consumption Insights do Power BI**. Você deseja criar suas próprias visualizações para sua organização? Nesse caso, o pacote de conteúdo do Azure Consumption Insights para PowerBI é sua ferramenta de escolha. Usando este pacote de conteúdo e o PowerBI, você pode criar visualizações personalizadas para representar sua organização, fazer uma análise mais profunda sobre os custos e adicionar outras fontes de dados para aprimoramento adicional.

* **API de consumo**. As [APIs de consumo](/rest/api/consumption/) oferecem acesso programático aos dados de uso e custo, além de informações sobre orçamentos, instâncias reservadas e cobranças do marketplace. Essas APIs são acessíveis somente para Registros Enterprise e algumas assinaturas Web Direct, porém oferecem a capacidade de integrar seus dados de custo em suas próprias ferramentas e data warehouses. Você também pode acessar essas APIs usando a CLI do Azure, visto [aqui](/cli/azure/consumption?view=azure-cli-latest).

Quando olhamos entre os clientes que usaram a nuvem por muito tempo e são “maduros” em termos de uso, podemos ver várias práticas altamente recomendadas

* **Monitorar ativamente os custos**. As organizações que são usuários maduros do Azure monitoram constantemente os custos e executam ações quando necessário. Algumas organizações até possuem pessoas dedicadas a fazer a análise e sugerir alterações para uso, e essas pessoas mais que compensam o preço dos seus serviços na primeira vez que encontram um cluster HDInsight não utilizado que estava em execução durante meses.
* **Usar instâncias reservadas**. Outra prática essencial para o gerenciamento de custos na nuvem é usar a ferramenta certa para o trabalho. Se você tiver uma VM IaaS que deve permanece ligada 24x7, então usar uma Instância Reservada economizará uma quantidade significativa de dinheiro. Encontrar o equilíbrio certo entre automatizar o desligamento de máquinas virtuais e usar RIs exige experiência e análise.
* **Usar a automação de forma eficaz**: muitas cargas de trabalho não precisam estar em execução todos os dias. Até mesmo desligar uma VM por um período de 4 horas todos os dias pode economizar 15% do seu custo. A automação se pagará rapidamente.
* **Use as marcas de recurso para visibilidade**: conforme mencionado em outro lugar neste documento, usar as marcas de recurso permitirá uma melhor análise de custos.

Gerenciamento de custos é uma disciplina essencial para uma execução eficaz e eficiente de uma nuvem pública. As empresas que alcançam o sucesso serão capazes de controlar os custos e correspondê-los à sua demanda real em vez de gastar demais e esperar que a demanda chegue.

## <a name="automate"></a>Automatizar

Um dos muitos recursos que diferencia a maturidade das organizações que usam provedores de nuvem é o nível de automação que elas têm incorporado.  A automação é um processo contínuo e conforme sua organização se move para a nuvem é uma área na qual você precisa investir tempo e recursos.  A automação serve para muitas finalidades, incluindo distribuição consistente de recursos (ligada diretamente a outro conceito essencial do andaime, modelos e DevOps) para a correção de problemas.  A automação é o “tecido conjuntivo” do andaime do Azure e vincula cada área.

Há várias ferramentas disponíveis à medida que você cria esse recurso, desde ferramentas próprias da Microsoft, como a Automação do Azure, Grade de Eventos e das ferramentas de CLI do Azure, até uma grande quantidade de ferramentas de terceiros, como Terraform, Jenkins, Chef e Puppet (para citar alguns exemplos). Essenciais que sua equipe de operações possa automatizar são a Automação do Azure, Grade de Eventos e o Azure Cloud Shell:

* **Automação do Azure**: é um recurso baseado em nuvem que permite a criação de Runbooks (no PowerShell ou Python) e permite automatizar processos, configurar recursos e até aplicar patches.  A [Automação do Azure](/azure/automation/automation-intro) tem um conjunto abrangente de recursos de plataforma cruzada que fazem parte da sua implantação, mas são muito extensos para serem abordado em detalhes aqui.
* **Grade de Eventos**: este [serviço](/azure/event-grid) é um sistema de roteamento de eventos totalmente gerenciado permite reagir a eventos dentro de seu ambiente do Azure. Assim como a Automação é o tecido conjuntivo das organizações de nuvem maduras, a Grade de Eventos é o tecido conjuntivo da boa automação. Usando a Grade de Eventos, você pode criar uma ação simples, sem servidor, para enviar um email ao administrador sempre que um novo recurso é criado e registrar este recurso em um banco de dados. Essa mesma Grade de Eventos pode notificar quando um recurso é excluído e remover o item do banco de dados.
* **Azure Cloud Shell**: é um [shell](/azure/cloud-shell/overview) interativo, baseado em navegador para o gerenciamento de recursos do Azure. Ele fornece um ambiente completo para o PowerShell ou Bash que é iniciado conforme necessário (e mantido por você) para que você tenha um ambiente consistente para executar seus scripts. O Azure Cloud Shell fornece acesso a ferramentas essenciais adicionais - já instaladas - para automatizar seu ambiente, incluindo [CLI do Azure](/cli/azure/get-started-with-azure-cli?view=azure-cli-latest), [Terraform](/azure/virtual-machines/linux/terraform-install-configure) e uma lista crescente de [ferramentas](https://azure.microsoft.com/updates/cloud-shell-new-cli-tools-and-font-size-selection/) adicionais para gerenciar contêineres, bancos de dados (sqlcmd) e muito mais.

A automação é um trabalho em tempo integral e ele rapidamente se tornará uma das mais importantes tarefas operacionais dentro de sua equipe de nuvem. As organizações que usam a abordagem de “primeiro automatizar” tem maior sucesso usando o Azure:

* Gerenciamento de custos: buscar oportunidades de automação ativamente e criar automação para redimensionar recursos, escalar verticalmente/reduzir verticalmente e desativar recursos não utilizados.
* Flexibilidade operacional: através do uso de automação (juntamente com modelos e DevOps) você obtém um nível de capacidade de repetição que aumenta a disponibilidade e a segurança e permite que sua equipe se concentre na solução de problemas de negócios.

## <a name="templates-and-devops"></a>Modelos e DevOps

Conforme realçado na seção Automação, sua meta como uma organização deve ser provisionar recursos por meio de scripts e modelos controlados por código-fonte e para minimizar a configuração interativa de seus ambientes. Essa abordagem de “infraestrutura como código”, juntamente com um processo de DevOps disciplinado para implantação contínua pode garantir a consistência e reduzir desvios em seus ambientes. Quase todos os recursos do Azure podem ser implantados por meio de [modelos JSON do Azure Resource Manager (ARM)](/azure/azure-resource-manager/resource-group-template-deploy) em conjunto com o PowerShell ou com a CLI da plataforma Azure e ferramentas como a Terraform da Hashicorp (que tem um suporte de primeira classe e é integrado ao Azure Cloud Shell).

Artigos como [este](https://blogs.msdn.microsoft.com/mvpawardprogram/2018/05/01/azure-resource-manager/) oferecem uma ótima discussão sobre as melhores práticas e lições aprendidas na aplicação de uma abordagem de DevOps para modelos do ARM com a cadeia de ferramentas do [Azure DevOps](/azure/devops/user-guide/?view=vsts). Demore o tempo e faça o esforço necessários para desenvolver um conjunto principal de modelos específicos para os requisitos da sua organização e para desenvolver pipelines de entrega contínua com cadeias de ferramentas do DevOps (Azure DevOps, Jenkins, Bamboo, Teamcity, Concourse), especialmente para ambientes de produção e de QA. Há uma grande biblioteca de [modelos de início rápido do Azure](https://github.com/Azure/azure-quickstart-templates) no GitHub que você pode usar como ponto de partida para modelos, e você pode criar rapidamente os pipelines de entrega baseado em nuvem com o Azure DevOps.

Como uma prática recomendada para assinaturas de produção ou de grupos de recursos, sua meta deve ser utilizar segurança RBAC para impedir usuários interativos por padrão, e utilizar os pipelines de entrega contínua automatizados com base em entidades de serviço para provisionar todos os recursos e fornecer todo o código do aplicativo. Nenhum administrador ou desenvolvedor deve tocar o Portal do Azure para configurar os recursos de forma interativa. Esse nível de DevOps exige um esforço concentrado e utiliza todos os conceitos de andaime do Azure e fornece um ambiente consistente e mais seguro que atenda às suas organizações para aumentar a escala.

> [!TIP]
> Ao projetar e desenvolver modelos complexos do ARM, use [modelos vinculados](/azure/azure-resource-manager/resource-group-linked-templates) para organizar e refatorar relacionamentos complexos de arquivos JSON monolíticos. Isso permitirá que você gerencie recursos individualmente e faça seus modelos mais legíveis, testáveis e reutilizáveis.

O Azure é um provedor de nuvem de grande escala e, ao mudar sua organização do mundo de servidores locais para a nuvem, utilizar os mesmos conceitos que provedores de nuvem e aplicativos SaaS utilizam fornecerá a sua organização a capacidade de reagir às necessidades dos negócios de uma maneira muito mais eficiente.

## <a name="core-network"></a>Rede principal

O componente final do modelo de referência de andaime do Azure é essencial para como sua organização acessa o Azure, de forma segura. O acesso a recursos pode ser interno (dentro da rede da empresa) ou externo (por meio da Internet). É fácil para os usuários em sua organização colocar recursos inadvertidamente no ponto errado e potencialmente abri-los para o acesso mal-intencionado. Assim como acontece com dispositivos locais, as empresas devem adicionar os controles adequados para garantir que os usuários do Azure tomam as decisões certas. Para governança da assinatura, identificamos recursos principais que fornecem controle básico de acesso. Os principais recursos consistem em:

* **Redes virtuais** são objetos de contêiner para sub-redes. Embora não sejam estritamente necessárias, geralmente elas são usadas ao conectar aplicativos aos recursos corporativos internos.
* **Rotas definidas pelo usuário** permitem que você manipule a tabela de rotas em uma sub-rede, permitindo o envio de tráfego por meio de uma solução de virtualização de rede ou para um gateway remoto em uma rede virtual emparelhada.
* **Emparelhamento de rede virtual** permite que você conecte duas ou mais redes virtuais do Azure sem nenhum problema, criando designs de hub-spoke ou redes de serviços compartilhados mais complexos.
* **Pontos de extremidade de serviço**. No passado, os serviços de PaaS contavam com métodos diferentes para proteger o acesso aos recursos de suas redes virtuais. Pontos de extremidade de serviço permitem que você proteja o acesso aos serviços de PaaS habilitados APENAS de pontos de extremidade conectados, aumentando a segurança como um todo.
* **Grupos de segurança** são um amplo conjunto de regras que fornecem a você a capacidade de permitir ou negar o tráfego de entrada e saída para/de recursos do Azure. [Grupos de Segurança](/azure/virtual-network/security-overview) consistem em regras de segurança, que podem ser aumentadas com **Marcas de Serviço** (que definem os serviços do Azure comuns como AzureKeyVault, Sql e outros) e **Grupos de Aplicativos** (que definem a estrutura do aplicativo, como servidores Web, AppServers e assim por diante)

> [!TIP]
> Usar marcas de serviço e grupos de aplicativos em seus grupos de segurança de rede não apenas melhoram a legibilidade das regras, que é essencial para entender o impacto, mas também possibilita uma microssegmentação eficaz dentro de uma sub-rede maior, reduzindo a expansão e aumentando a flexibilidade.

### <a name="virtual-data-center"></a>Data Center Virtual

O Azure fornece recursos internos e recursos de terceiros da nossa extensa rede de parceiros que permitem que você tenha uma postura eficaz em relação à segurança. Mais importante, a Microsoft fornece as melhores práticas e orientações com o [Azure Data Center Virtual](/azure/architecture/vdc/networking-virtual-datacenter). Ao mudar de uma única carga de trabalho para várias cargas de trabalho que utilizam recursos híbridos, as diretrizes de VDC fornecerão “receita” para possibilitar uma rede flexível, que aumentará conforme as suas cargas de trabalho no Azure crescem.  

## <a name="next-steps"></a>Próximas etapas

A governança é essencial para o sucesso do Azure. Este artigo foca na implementação técnica de um andaime empresarial, mas toca apenas no processo mais amplo e nas relações entre os componentes. A governança da política flui de cima para baixo e é determinada por aquilo que a empresa quer alcançar. Naturalmente, a criação de um modelo de governança para o Azure inclui representantes da TI, mas o mais importante é ter uma forte representação dos líderes do grupo de negócios, além de gerenciamento de segurança e risco. No fim, um andaime empresarial é sobre reduzir o risco aos negócios para facilitar a missão e os objetivos de uma organização

Agora que você aprendeu sobre governança de assinatura, é hora de ver essas recomendações na prática. Veja [Exemplos de implementação da governança de assinatura do Azure](azure-scaffold-examples.md).
