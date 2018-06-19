---
title: 'Adoção do Azure: intermediário'
description: Descreve o nível intermediário de conhecimento de que uma empresa precisa para adotar o Azure
author: petertay
ms.openlocfilehash: 39b98595dd615ba1aa36921e48a0b23797bebaa0
ms.sourcegitcommit: b3d74d8a89b2224fc796ce0e89cea447af43a0d4
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/11/2018
ms.locfileid: "35290984"
---
# <a name="azure-cloud-adoption-guide-intermediate-overview"></a>Guia de adoção do Azure Cloud: visão geral intermediária

No estágio de adoção fundamental, foram introduzidos os conceitos básicos de governança de recursos do Azure. O estágio fundamental foi projetado para iniciar sua jornada de adoção do Azure, e ele conduziu você pela forma de implantar uma carga de trabalho simples com uma única equipe pequena. Na realidade, a maioria das grandes organizações tem muitas equipes que estão trabalhando em muitas cargas de trabalho diferentes ao mesmo tempo. Como você poderia esperar, um modelo de controle simples não é suficiente para gerenciar cenários organizacionais e de desenvolvimento mais complexos.

O foco do estágio intermediário de adoção do Azure é projetar seu modelo de controle para várias equipes trabalhando em várias novas cargas de trabalho de desenvolvimento do Azure.  

O público para este estágio do guia é composto pelas seguintes personas dentro de sua organização:
- *Financeiro:* proprietário do compromisso financeiro para o Azure, responsável por desenvolver políticas e procedimentos para controlar os custos de consumo de recursos, incluindo cobrança e estorno.
- *TI Central:* responsável pela governança dos recursos de nuvem da sua organização, incluindo o gerenciamento de recursos e o acesso, bem como a integridade e o monitoramento da carga de trabalho.
- *Proprietário de infraestrutura compartilhada*: funções técnicas responsáveis pela conectividade de rede do local para a nuvem.
- *Operações de segurança*: responsável pela implementação da política de segurança necessária para estender o limite de segurança local para incluir o Azure. Também pode ter a propriedade da infraestrutura de segurança no Azure para armazenar segredos.
- *Proprietário da carga de trabalho:* responsável pela publicação de uma carga de trabalho do Azure. Dependendo da estrutura de equipes de desenvolvimento de sua organização, pode ser um cliente potencial de desenvolvimento, um cliente potencial de gerenciamento do programa ou um cliente potencial de engenharia de compilação. Uma parte do processo de publicação pode incluir a implantação de recursos do Azure.
  - *Colaborador da carga de trabalho:* responsável pela colaboração na publicação de uma carga de trabalho do Azure. Pode exigir acesso de leitura aos recursos do Azure para monitoramento ou ajuste de desempenho. Não exige a permissão para criar, atualizar ou excluir os recursos.

## <a name="section-1-azure-concepts-for-multiple-workloads-and-multiple-teams"></a>Seção 1: Conceitos do Azure para várias cargas de trabalho e várias equipes

No estágio de adoção básico, você obteve algumas noções básicas sobre aspectos internos do Azure e como os recursos são criados, lidos, atualizados e excluídos. Você também aprendeu sobre identidade e que o Azure só confia no Azure Active Directory (AD) para autenticar e autorizar os usuários que precisam de acesso a esses recursos.

Você também começou a aprender como configurar as ferramentas de administração do Azure para gerenciar o uso de recursos do Azure pela organização. No estágio fundamental, examinamos como governar o acesso de uma única equipe aos recursos necessários para implantar uma carga de trabalho simples. Na realidade, sua organização vai ser composta de várias equipes trabalhando simultaneamente em várias cargas de trabalho. 

Antes de começar, vamos dar uma olhada no que o termo **carga de trabalho** realmente significa. É um termo que normalmente é entendido para definir uma unidade arbitrária de recursos, como um aplicativo ou serviço. Podemos pensar em uma carga de trabalho em termos de artefatos de código que são implantados em um servidor, bem como quaisquer outros serviços, como um banco de dados, que sejam necessários. Esta é uma definição útil para um aplicativo ou serviço local, mas na nuvem precisamos expandi-lo. 

Na nuvem, uma carga de trabalho não só abrange todos os artefatos, mas também inclui recursos de nuvem. Os recursos de nuvem são incluídos como parte de nossa definição devido a um conceito conhecido como **infraestrutura como código**. Como você aprendeu na explicação "como funciona o Azure", os recursos no Azure são implantados por um serviço orquestrador. O serviço orquestrador expõe essa funcionalidade por meio de uma API Web, e essa API Web podem ser chamadas usando várias ferramentas, como o Powershell, a interface de linha de comando (CLI) do Azure e o portal do Azure. Isso significa que podemos especificar nossos recursos em um arquivo legível por máquina que pode ser armazenado junto com os artefatos de código associados ao nosso aplicativo.

Isso nos permite definir uma carga de trabalho em termos de artefatos de código e os recursos de nuvem necessários e isso permite mais nos **isolar** nossas cargas de trabalho. As cargas de trabalho podem ser isoladas pela forma como os recursos são organizados por topologia de rede ou por outros atributos. O objetivo do isolamento de carga de trabalho é associar recursos específicos de uma carga de trabalho para uma equipe para a equipe de forma independente pode gerenciar todos os aspectos desses recursos. Isso permite que várias equipes compartilharem serviços de gerenciamento de recursos no Azure, evitando a exclusão não intencional ou modificação de uns dos outros recursos.

Esse isolamento também permite que o outro conceito conhecido como [DevOps](https://azure.microsoft.com/solutions/devops/). DevOps inclui as práticas de desenvolvimento de software que incluem o desenvolvimento de software e as operações de TI acima, mas adiciona o uso de automação tanto quanto possível. Um dos princípios de DevOps é conhecido como integração contínua e entrega contínua (CI/CD). Integração contínua refere-se aos processos de compilação automatizado que são executados sempre que um desenvolvedor confirmar uma alteração de código e fornecimento contínuo refere-se aos processos automatizados que implanta esse código em vários **ambientes** como um **ambiente de desenvolvimento** para teste ou um **ambiente de produção** para a implantação final.

## <a name="section-2-governance-design-for-multiple-teams-and-multiple-workloads"></a>Seção 2: Design de governança para várias equipes e cargas de trabalho

No [estágio fundamental](/azure/architecture/cloud-adoption-guide/adoption-intro/overview) do guia de adoção da nuvem do Azure, você foi apresentado ao conceito de governança de nuvem. Você aprendeu a criar um modelo de controle simples para uma única equipe trabalhando em uma única carga de trabalho. 

No estágio de intermediários, de [guia de design de governança](governance-design-guide.md) expande os conceitos fundamentais para adicionar várias equipes, várias cargas de trabalho e vários ambientes. Depois de seguir os exemplos do documento, você pode aplicar os princípios de design para projetar e implementar o modelo de governança da sua organização.

## <a name="section-3-implementing-a-resource-management-model"></a>Seção 3: Implementação de um modelo de gerenciamento de recursos

O modelo de controle de nuvem da organização representa a interseção entre as ferramentas de gerenciamento de acesso de recursos do Azure, seus funcionários e as regras de gerenciamento de acesso que você definiu. No guia de design de governança, você aprendeu sobre vários modelos diferentes para controlar o acesso aos recursos do Azure. Agora vamos examinar as etapas necessárias para implementar o modelo de gerenciamento de recursos com uma assinatura para cada uma da **infraestrutura compartilhada**, **produção**, e **desenvolvimento** ambientes do guia de design. Teremos um **proprietário da assinatura** para todos os três ambientes. Cada carga de trabalho será isolada em um **grupo de recursos** com um **proprietário da carga de trabalho** adicionado com a função **colaborador**.

> [!NOTE]
> Leia as [noções básicas sobre o acesso a recursos no Azure][understand-resource-access-in-azure] para saber mais sobre a relação entre as Contas e a assinatura do Azure. 

Siga estas etapas:

1. Crie uma [conta do Azure](/azure/active-directory/sign-up-organization) se sua organização ainda não tiver uma. A pessoa que se inscreve na conta do Azure se torna o administrador da conta do Azure e liderança de sua organização deve selecionar um indivíduo para assumir essa função. Essa pessoa será responsável por:
  * criação de assinaturas e
  * criando e administrando locatários do [Azure Active Directory (AD)](/azure/active-directory/active-directory-whatis) que armazenam a identidade do usuário para as assinaturas.    
2. A equipe de liderança de sua organização decide quais pessoas são responsáveis por:
  * Gerenciamento de identidade do usuário. um [locatário do Azure AD](/azure/active-directory/develop/active-directory-howto-tenant) é criado por padrão quando a conta do Azure da sua organização é criada, e o administrador da conta é adicionado como o [administrador global do Azure AD](/azure/active-directory/active-directory-assign-admin-roles-azure-portal#details-about-the-global-administrator-role) por padrão. Sua organização pode escolher outro usuário para gerenciar a identidade do usuário por [atribuir a função de administrador global do Azure AD para que o usuário](/azure/active-directory/active-directory-users-assign-role-azure-portal). 
  * Assinaturas, o que significa que esses usuários:
    * gerencie os custos associados ao uso de recursos nessa assinatura,
    * implementar e manter o modelo de permissão mínima para acesso a recursos e
    * controlar os limites de serviço.
  * Infraestrutura de serviços compartilhados (se a sua organização decidir usar esse modelo), o que significa que esse usuário é responsável por:
    * local para conectividade de rede do Azure e 
    * propriedade de conectividade de rede no Azure por meio do emparelhamento de rede virtual.
  * Proprietários da carga de trabalho. 
3. O administrador global do Azure AD [cria novas contas de usuário](/azure/active-directory/add-users-azure-active-directory) para:
  * a pessoa que será o **proprietário da assinatura** para cada assinatura associada a cada ambiente. Observe que isso é necessário apenas se o **administrador de serviço** da assinatura não for o responsável por gerenciar o acesso a recursos para cada assinatura/ambiente.
  * a pessoa que será o **usuário de operações de rede** e
  * as pessoas que são **proprietários da carga de trabalho**.
4. O administrador da conta do Azure cria as seguintes três assinaturas usando o [portal de conta do Azure](https://account.azure.com):
  * uma assinatura para o ambiente de **infraestrutura compartilhada**,
  * uma assinatura para o ambiente de **produção** e 
  * uma assinatura para o ambiente de **desenvolvimento**. 
5. O administrador da conta do Azure [adiciona o proprietário do serviço de assinatura para cada assinatura](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-admin-for-a-subscription-in-azure-portal).
6. Crie um processo de aprovação para **proprietários da carga de trabalho** para solicitar a criação de grupos de recursos. O processo de aprovação pode ser implementado de diversas maneiras, como por email, ou você pode usar uma ferramenta de gerenciamento do processo como [fluxos de trabalho do](https://support.office.com/article/introduction-to-sharepoint-workflow-07982276-54e8-4e17-8699-5056eff4d9e3). O processo de aprovação pode seguir estas etapas:
  1. O **proprietário da carga de trabalho** prepara uma lista de materiais dos recursos do Azure necessários no **desenvolvimento** ambiente, **produção** ambiente, ou ambos e envia para o **proprietário da assinatura**.
  2. O **proprietário da assinatura** examina a lista de materiais e valida os recursos solicitados para garantir que os recursos solicitados são apropriados para seu uso planejado - por exemplo, a verificação de que a solicitação [ tamanhos de máquina virtual](/azure/virtual-machines/windows/sizes) estão corretas.
  3. Se a solicitação não for aprovada, o **proprietário da carga de trabalho** será notificado. Se a solicitação for aprovada, o **proprietário da assinatura** [cria o grupo de recurso solicitado](/azure/azure-resource-manager/resource-group-portal#manage-resource-groups) após sua organização [convenções de nomenclatura](/azure/architecture/best-practices/naming-conventions), [ Adiciona o **proprietário da carga de trabalho** ](/azure/role-based-access-control/role-assignments-portal#add-access) com o [ **colaborador** função](/azure/role-based-access-control/built-in-roles#contributor) e envia uma notificação para o **proprietário de carga de trabalho** que o grupo de recursos foi criado.
7. Crie um processo de aprovação para os proprietários da carga de trabalho solicitar uma emparelhamento de conexão do proprietário infraestrutura compartilhada de rede virtual. Assim como acontece com a etapa anterior, esse processo de aprovação pode ser implementado usando email ou uma ferramenta de gerenciamento de processos.

Agora que você já implementou o seu modelo de controle, poderá implantar seus serviços de infraestrutura compartilhada.

## <a name="section-4-deploy-shared-infrastructure-services"></a>Seção 4: implantar serviços de infraestrutura compartilhada

Há várias [arquiteturas de rede híbrida](/azure/architecture/reference-architectures/hybrid-networking/) que sua organização pode usar para se conectar a sua rede local ao Azure. Cada essas arquiteturas de referência inclui uma implantação que requer um identificador de assinatura. Durante a implantação, especifique o identificador de assinatura para a assinatura associada ao seu ambiente **compartilhado da infraestrutura**. Você também precisa editar os arquivos de modelo para especificar o grupo de recursos é gerenciado pelo seu **operações de rede** usuário ou, você pode usar os grupos de recursos padrão na implantação e adicionar o **operações de rede**  usuário com o **colaborador** função a eles.

<!-- links -->
[understand-resource-access-in-azure]: /azure/role-based-access-control/rbac-and-directory-admin-roles