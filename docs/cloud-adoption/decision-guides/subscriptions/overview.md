---
title: 'CAF: guia de decisão da assinatura'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Saiba mais sobre assinaturas de plataforma de nuvem como um serviço principal nas migrações do Azure.
author: rotycenh
ms.openlocfilehash: c0781f6af25150d359395b1b80506dd0cfee8e3c
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900219"
---
# <a name="subscription-decision-guide"></a>Guia de decisão da assinatura

Todas as plataformas de nuvem se baseiam em um modelo de propriedade principal que fornece às organizações várias opções de cobrança e de gerenciamento de recursos. A estrutura que o Azure usa é diferente de outros provedores de nuvem porque ela inclui várias opções de suporte à hierarquia organizacional e à propriedade de assinatura agrupada. Independentemente disso, normalmente há um indivíduo responsável pela cobrança e outro que recebe a atribuição de proprietário de nível superior para fins de gerenciamento de recursos.

![Gráfico com opções de assinatura, da menos para a mais complexa, alinhada aos links de salto abaixo](../../_images/discovery-guides/discovery-guide-subscriptions.png)

Ir para: [Design de assinaturas e Contratos Enterprise do Azure](#subscriptions-design-and-azure-enterprise-agreements) | [Padrões de design de assinatura](#subscription-design-patterns) | [Grupos de gerenciamento](#management-groups)  |  [Organização no nível da assinatura](#organization-at-the-subscription-level)

O design de assinatura é uma das estratégias mais comuns que as empresas usam para estabelecer uma estrutura ou organizar ativos durante a adoção da nuvem.

**Hierarquia de assinatura**: uma *assinatura* é uma coleção lógica de serviços do Azure (como máquinas virtuais, Bancos de Dados SQL, Serviços de Aplicativos ou contêineres). Cada ativo no Azure é implantado em uma assinatura única. Cada assinatura pertence a uma *conta*. A conta é uma conta de usuário (ou, de preferência, uma conta de serviço) que fornece acessos administrativo e de cobrança em uma assinatura. Para clientes que tiverem assumido o compromisso de usar um valor específico do Azure por meio de um EA (Contrato Enterprise), outro nível de controle, chamado *departamento*, é adicionado. No portal do EA, assinaturas, contas e departamentos podem ser usados a fim de criar uma hierarquia para fins de cobrança e gerenciamento.

A complexidade dos designs de assinatura varia. Decisões relativas a uma estratégia de design têm pontos de inflexão exclusivos, pois normalmente envolvem restrições tanto comerciais quanto de TI. Antes de tomar decisões técnicas, os arquitetos de TI e os tomadores de decisão devem trabalhar com os stakeholders do negócio e com a equipe de estratégia para entender a abordagem contábil para a nuvem desejada, as práticas contábeis nas unidades de negócios e as necessidades de mercado globais da organização.

**Ponto de inflexão**: a linha pontilhada na imagem acima faz referência a um ponto de inflexão entre padrões simples e padrões mais complexos de design de assinatura. Outros pontos de decisões técnicas com base no tamanho da propriedade digital comparados com os limites da assinatura do Azure, as políticas de isolamento e de diferenciação e as divisões operacionais de TI geralmente afetam de forma substancial o design de assinatura.

**Outras considerações**: Uma coisa importante a se observar ao selecionar um design de assinatura é que as assinaturas não são a única maneira de agrupar recursos ou implantações. As assinaturas foram criadas nos primórdios do Azure e, como tal, elas têm limitações relacionadas a soluções do Azure anteriores, como o Azure Service Manager.

A estrutura de implantação, a automação e novas abordagens ao agrupamento de recursos podem afetar o design de assinatura estruturado. Antes de finalizar um design de assinatura, considere como as decisões de [consistência de recursos](../resource-consistency/overview.md) podem influenciar as escolhas de design. Por exemplo, uma grande empresa multinacional pode inicialmente levar em conta um padrão complexo para o gerenciamento da assinatura. No entanto, a mesma empresa pode ter melhores benefícios com um padrão de unidade de negócios mais simples adicionando uma hierarquia do grupo de gerenciamento.

## <a name="subscriptions-design-and-azure-enterprise-agreements"></a>Design de assinaturas e Contratos Enterprise do Azure

Todas as assinaturas do Azure estão associadas uma conta, que está conectada ao controle de acesso de cobrança e de nível superior para cada assinatura. Uma mesma conta pode ter várias assinaturas e fornecer um nível base à organização das assinaturas.

Para pequenas implantações do Azure, uma assinatura única ou pequena coleção de assinaturas pode compor o acervo de nuvem completo. No entanto, grandes implantações do Azure provavelmente precisam abranger várias assinaturas para dar suporte a sua estrutura organizacional e ignorar [cotas e limites de assinatura](/azure/azure-subscription-service-limits).

Cada Contrato Enterprise do Azure permite maior organização de assinaturas e contas em hierarquias que refletem suas prioridades organizacionais. O registro empresarial de sua organização define a forma e o uso dos serviços do Azure na empresa sob uma perspectiva contratual. É possível, em cada contrato Enterprise, subdividir o ambiente em departamentos, contas e assinaturas correspondentes à estrutura da organização.

![hierarquia](../../_images/infra-subscriptions/agreement.png)

## <a name="subscription-design-patterns"></a>Padrões de design de assinatura

Cada empresa é diferente. Portanto, a hierarquia de departamento/conta/assinatura habilitada por meio de um Contrato Enterprise do Azure permite grande flexibilidade em relação a como o Azure é organizado. A primeira e mais importante decisão que você pode tomar ao começar a usar a nuvem pública é modelar a hierarquia da organização para refletir as necessidades da empresa em termos de cobrança, gerenciamento e acesso a recursos.

Os seguintes padrões de assinatura refletem um aumento geral na complexidade do design de assinatura para dar suporte a possíveis prioridades organizacionais:

### <a name="single-subscription"></a>Assinatura única

Uma assinatura única por conta pode ser suficiente para organizações que precisam implantar um pequeno número de ativos hospedados na nuvem. Ele geralmente é o primeiro padrão de assinatura implementado no início de um processo de adoção de nuvem, permitindo que implantações de prova de conceito ou experimentais em pequena escala explorem as funcionalidades de uma plataforma de nuvem.

No entanto, pode haver limitações técnicas ao número de recursos com suporte por uma assinatura única. À medida que aumenta o tamanho do seu acervo de nuvem, você provavelmente precisará também dar suporte à organização dos recursos para organizar melhor políticas e controle de acesso de uma maneira que não tem suporte pela assinatura única.

### <a name="application-category-pattern"></a>Padrão de categoria do aplicativo

Conforme o volume de uma organização vai crescendo na nuvem, cresce também a probabilidade de se ter que usar várias assinaturas. Para isso, as assinaturas geralmente são criadas para dar suporte a aplicativos com diferenças fundamentais em relação ao grau de importância no negócio, aos requisitos de conformidade, aos controles de acesso ou às necessidades de proteção de dados. As assinaturas e contas que dão suporte a essas categorias de aplicativo são organizadas em um único departamento que pertence e é administrado pela equipe central de operações de TI.

Cada organização escolherá categorizar os aplicativos de forma diferente, geralmente separando as assinaturas com base em serviços ou aplicativos específicos, ou de acordo com arquétipos de aplicativo. Algumas cargas de trabalho que podem justificar uma assinatura separada de acordo com o padrão em questão incluem:

- Aplicativos experimentais ou de baixo risco
- Aplicativos com dados protegidos
- Cargas de trabalho críticas
- Aplicativos sujeitos a requisitos regulatórios (como HIPAA ou FedRAMP)
- Cargas de trabalho em lotes
- Cargas de trabalho de Big Data, como o Hadoop
- Cargas de trabalho em contêineres que usam orquestradores de implantação, como o Kubernetes
- Cargas de trabalho de análise

O padrão mencionado dá suporte a vários proprietários de contas responsáveis por cargas de trabalho específicas. Já que ele não possui uma estrutura mais complexa no nível de departamento da hierarquia do Contrato Enterprise, o padrão não requer um Contrato Enterprise do Azure para ser implementado.

![Padrão de categoria do aplicativo](../../_images/infra-subscriptions/application.png)

### <a name="functional-pattern"></a>Padrão funcional

É um padrão que organiza assinaturas e contas conforme linhas funcionais, como financeiro, vendas ou suporte de TI, usando a hierarquia de Empresa/Departamento/Conta/Assinatura fornecida aos clientes do Contrato Enterprise do Azure.

![Padrão funcional](../../_images/infra-subscriptions/functional.png)

### <a name="business-unit-pattern"></a>Padrão da unidade de negócios

É um padrão que agrupa assinaturas e contas com base na categoria de lucros e prejuízos, unidade de negócios, divisão, centro de lucro ou estrutura de negócios semelhante usando a hierarquia do Contrato Enterprise do Azure.

![Padrão da unidade de negócios](../../_images/infra-subscriptions/business.png)

### <a name="geographic-pattern"></a>Padrão geográfico

Para organizações com operações globais, é um padrão que agrupa assinaturas e contas com base em regiões geográficas usando a hierarquia do Contrato Enterprise do Azure.

![Padrão geográfico](../../_images/infra-subscriptions/geographic.png)

### <a name="mixed-patterns"></a>Padrões mistos

Hierarquia de empresa/departamento/conta/assinaturas. No entanto, você pode combinar os padrões, como região geográfica e unidade de negócios, para refletir estruturas de cobrança e organizacionais mais complexas em sua empresa. Além disso, o [design de consistência de recursos](../resource-consistency/overview.md) poderá estender ainda mais a governança e a estrutura organizacional de seu design de assinatura.

Os grupos de gerenciamento, conforme discutido na seção a seguir, podem ajudar a dar suporte a estruturas organizacionais mais complicadas.

Os grupos de gerenciamento, discutidos na seção a seguir, podem ajudar a dar suporte a estruturas organizacionais mais complicadas.

## <a name="management-groups"></a>Grupos de gerenciamento

Além da estrutura de departamento e organização fornecida pelos Contratos Enterprise, os [grupos de gerenciamento do Azure](/azure/governance/management-groups/index) oferecem maior flexibilidade para organizar controle de acesso, política e conformidade entre várias assinaturas. Grupos de gerenciamento podem ser aninhados em até seis níveis, permitindo que você crie uma hierarquia separada da hierarquia de cobrança. Isso pode ocorrer exclusivamente para o gerenciamento eficiente dos recursos.

Os grupos de gerenciamento podem espelhar a hierarquia de cobrança; é comum as empresas começarem assim. No entanto, o poder dos grupos de gerenciamento surge quando você os utiliza para modelar a organização de modo que as assinaturas relacionadas &mdash; independentemente de onde elas estejam na hierarquia de cobrança, &mdash; sejam agrupadas e precisem de funções comuns atribuídas, junto com iniciativas e políticas.

Os exemplos incluem:

- Produção/não produção: Algumas empresas criam grupos de gerenciamento para identificar suas assinaturas de produção e não produção. Os grupos de gerenciamento permitem que esses clientes gerenciem mais facilmente funções e políticas, por exemplo: uma assinatura que não é de produção pode permitir aos desenvolvedores o acesso de “colaborador”, mas em uma de produção, eles têm apenas acesso de “leitor”.
- Serviços internos/externos: Assim como ocorre com o par produção/não produção, as empresas geralmente têm requisitos, políticas e funções diferentes para serviços internos e externos (voltados para o cliente).

## <a name="organization-at-the-subscription-level"></a>Organização no nível da assinatura

Ao determinar seus departamentos e contas (ou grupos de gerenciamento), você precisará inicialmente decidir como vai dividir o ambiente do Azure para corresponder à organização. No entanto, as assinaturas são onde o trabalho realmente acontece, e as decisões envolvidas afetarão a segurança, a escalabilidade e a cobrança.

Considere os seguintes padrões como guias:

- **Aplicativo/serviço**: as assinaturas representam um aplicativo ou um serviço (portfólio de aplicativos).

- **Ciclo de vida**: as assinaturas representam um ciclo de vida de um serviço, como desenvolvimento ou produção.

- **Departamento**: as assinaturas representam departamentos na organização.

Os dois primeiros padrões são mais comumente utilizados, e são ambos altamente recomendáveis. A abordagem de ciclo de vida é adequada à maioria das organizações. Aqui, a recomendação geral é usar duas assinaturas base, produção e não produção, e usar grupos de recursos para detalhar ainda mais os ambientes.

Para obter uma descrição geral de como os grupos de recursos e as assinaturas do Azure são usados, confira [Gerenciamento de acesso a recursos no Azure](../../getting-started/azure-resource-access.md).

## <a name="next-steps"></a>Próximas etapas

Saiba como os serviços de identidade são usados no gerenciamento e no controle de acesso na nuvem.

> [!div class="nextstepaction"]
> [Identidade](../identity/overview.md)
