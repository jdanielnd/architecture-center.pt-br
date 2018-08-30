---
title: Guia de design da governança do Azure
description: Orientação para configurar os controles de governança do Azure para habilitar um usuário e implantar uma carga de trabalho simples
author: petertay
ms.openlocfilehash: 88cd24a46a6bdaca4e0dd18706af1f7e3fe3bce2
ms.sourcegitcommit: c4106b58ad08f490e170e461009a4693578294ea
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/10/2018
ms.locfileid: "43016025"
---
# <a name="azure-governance-design-guide"></a>Guia de design da governança do Azure

A audiência para este guia de design é a persona da *TI central* em sua organização. A *TI central* é responsável por projetar e implementar a arquitetura de governança de nuvem da sua organização. Como você aprendeu no explicador [o que é governança de recursos de nuvem?](governance-explainer.md), governança é o processo contínuo de gerenciar, monitorar e auditar o uso de recursos do Azure para atender as metas e requisitos da sua organização.

O objetivo deste guia é ajudar você a aprender o processo de criação da arquitetura de governança de sua organização, observando um conjunto de requisitos e objetivos de governança hipotético. Em seguida, discutiremos como configurar as ferramentas de governança do Azure para atendê-las. 

No estágio básico de adoção, nossa meta é implantar uma carga de trabalho simples no Azure. Isso resulta nos seguintes requisitos:
* Gerenciamento de identidade para um único **proprietário de carga de trabalho**, que é responsável por implantar e manter a carga de trabalho simples. O proprietário da carga de trabalho exige permissão para criar, ler, atualizar e excluir recursos, bem como permissão para delegar esses direitos a outros usuários no sistema de gerenciamento de identidade.
* Gerencie todos os recursos para a carga de trabalho simples como uma unidade única de gerenciamento.

## <a name="licensing-azure"></a>Licenciamento do Azure

Antes de começarmos a criação de nosso modelo de governança, é importante entender como o Azure é licenciado. Isso ocorre porque as contas administrativas associadas à sua licença do Azure têm o maior nível de acesso para todos os seus recursos do Azure. Essas contas administrativas formam a base do seu modelo de governança.  

> [!NOTE]
> Se sua organização tiver um [Microsoft Enterprise Agreement](https://www.microsoft.com/en-us/licensing/licensing-programs/enterprise.aspx) existente que não inclui o Azure, este pode ser adicionado mediante uma confirmação de pagamento adiantado. Consulte [licenciamento do Azure para a empresa](https://azure.microsoft.com/pricing/enterprise-agreement/) para obter mais informações. 

Quando o Azure é adicionado ao Enterprise Agreement de sua organização, ela deve criar uma **conta do Azure**. Durante o processo de criação da conta, um **proprietário da conta do Azure** foi criado, bem como um locatário do Azure Active Directory (Azure AD) com uma conta de **administrador global**. Um locatário do Azure AD é um constructo lógico que representa uma instância segura e dedicada do Azure AD.

![Conta do Azure com o Gerente de Contas do Azure e o administrador global do Azure AD](../_images/governance-3-0.png)
*Figura 1. Uma conta do Azure com um gerente de conta e o Administrador Global do Azure AD.*

## <a name="identity-management"></a>Gerenciamento de identidades

Somente o Azure confia no [Azure AD](/azure/active-directory) para autenticar usuários e autorizar o acesso de usuários a recursos, por isso, o Azure AD é o nosso sistema de gerenciamento de identidade. O administrador global do Azure AD tem o nível mais alto de permissões e pode executar todas as ações relacionadas à identidade, incluindo a criação de usuários e a atribuição de permissões. 

Nosso requisito é o gerenciamento de identidade para um único **proprietário de carga de trabalho**, que é responsável por implantar e manter a carga de trabalho simples. O proprietário da carga de trabalho exige permissão para criar, ler, atualizar e excluir recursos, bem como permissão para delegar esses direitos a outros usuários no sistema de gerenciamento de identidade.

Nosso administrador global do Azure Active Directory criará o **proprietário da carga de trabalho** da conta para o **proprietário da carga de trabalho**:

![O administrador global do Azure Active Directory cria a conta de proprietário da carga de trabalho](../_images/governance-1-2.png)
*Figura 2. O administrador global do Azure Active Directory cria a conta de usuário da carga de trabalho*

Não conseguimos atribuir permissão de acesso ao recurso até que esse usuário seja adicionado a uma **assinatura**, portanto, faremos isso nas próximas duas seções. 

## <a name="resource-management-scope"></a>Escopo do gerenciamento de recursos

À medida que o número de recursos implantados por sua organização cresce, a complexidade de governar esses recursos também aumenta. O Azure implementa uma hierarquia de contêiner lógico para permitir que sua organização gerencie seus recursos em grupos em vários níveis de granularidade, o que também é conhecido como **escopo**. 

O nível superior do escopo de gerenciamento de recursos é o nível de **assinatura**. Uma assinatura é criada pelo **proprietário da conta** do Azure, que estabelece o compromisso financeiro e é responsável pelo pagamento de todos os recursos do Azure associado à assinatura:

![O proprietário da conta do Azure cria uma assinatura](../_images/governance-1-3.png)
*Figura 3. O proprietário da conta do Azure cria uma assinatura.*

Quando a assinatura é criada, o **proprietário da conta** do Azure associa um locatário do Azure Active Directory com a assinatura e este locatário do Azure AD é usado para autenticar e autorizar usuários:

![O proprietário da conta do Azure associa o locatário do Azure AD com a assinatura](../_images/governance-1-4.png)
*Figura 4. O proprietário da conta do Azure associa o locatário do Azure AD com a assinatura.*

Você talvez tenha observado que não há, no momento, nenhum usuário associado à assinatura, o que significa que ninguém tem permissão para gerenciar recursos. Na realidade, o **proprietário da conta** é o proprietário da assinatura e tem permissão para executar qualquer ação em um recurso na assinatura. No entanto, em termos práticos, o **proprietário da conta** é muito provavelmente uma pessoa do departamento financeiro de sua organização e não é responsável por criar, ler, atualizar e excluir recursos - essas tarefas serão executadas pelo **proprietário da carga de trabalho**. Portanto, precisamos adicionar o **proprietário da carga de trabalho** à assinatura e atribuir permissões.

Uma vez que o **proprietário da conta** atualmente é o único usuário com permissão para adicionar o **proprietário da carga de trabalho** à assinatura, eles adicionam o **proprietário da carga de trabalho** à assinatura:

![O proprietário da conta do Azure adiciona o **proprietário da carga de trabalho** à assinatura](../_images/governance-1-5.png)
*Figura 5. O proprietário da conta do Azure adiciona o proprietário da carga de trabalho à assinatura.*

O **proprietário da conta** do Azure concede permissões ao **proprietário da carga de trabalho** atribuindo uma função de [controle de acesso baseado em função (RBAC)](/azure/role-based-access-control/). A função RBAC especifica um conjunto de permissões que o **proprietário da carga de trabalho** tem para um tipo de recurso individual ou um conjunto de tipos de recursos.

Observe que neste exemplo, o **proprietário da conta** atribuiu a [função de **proprietário** interna](/azure/role-based-access-control/built-in-roles#owner): 

![O **proprietário da carga de trabalho** foi atribuído à função de proprietário interna](../_images/governance-1-6.png)
*Figura 6. O proprietário da carga de trabalho foi atribuído à função de proprietário interna.*

A função de **proprietário** interna concede todas as permissões ao **proprietário da carga de trabalho** no escopo da assinatura. 

> [!IMPORTANT]
> O **proprietário de conta** do Azure é responsável pelo compromisso financeiro associado à assinatura, mas o **proprietário da carga de trabalho** tem as mesmas permissões. O **proprietário da conta** deve confiar no **proprietário da carga de trabalho** para implantar recursos que estão dentro do orçamento da assinatura.

O próximo nível do escopo de gerenciamento é o nível do **grupo de recursos**. Um grupo de recursos é um contêiner lógico para recursos. Operações aplicadas no nível do grupo de recursos se aplicam a todos os recursos em um grupo. Além disso, é importante observar que as permissões para cada usuário são herdadas do próximo nível para cima, a menos que elas sejam alteradas explicitamente nesse escopo. 

Para ilustrar isso, vamos examinar o que acontece quando o **proprietário da carga de trabalho** cria um grupo de recursos:

![O **proprietário da carga de trabalho** cria um grupo de recursos](../_images/governance-1-7.png)
*Figura 7. O proprietário da carga de trabalho cria um grupo de recursos e herda a função de proprietário interna no escopo do grupo de recursos.*

Novamente, a função de **proprietário** interna concede todas as permissões ao **proprietário da carga de trabalho** no escopo do grupo de recursos. Conforme discutido anteriormente, essa função é herdada do nível da assinatura. Se uma função diferente for atribuída a esse usuário nesse escopo, ela se aplicará somente a esse escopo.

O menor nível do escopo de gerenciamento é o nível de **recurso**. Operações aplicadas ao nível de recurso se aplicam somente ao próprio recurso. E, mais uma vez, as permissões no nível de recurso são herdadas do escopo do grupo de recursos. Por exemplo, vejamos o que acontece se o **proprietário da carga de trabalho** implantar uma [rede virtual](/azure/virtual-network/virtual-networks-overview) no grupo de recursos:

![O **proprietário da carga de trabalho** cria um recurso](../_images/governance-1-8.png)
*Figura 8. O proprietário da carga de trabalho cria um recurso e herda a função de proprietário interna no escopo do recurso.*

O **proprietário da carga de trabalho** herda a função de proprietário no escopo do recurso, o que significa que o proprietário da carga de trabalho tem todas as permissões para a rede virtual. 

## <a name="summary"></a>Resumo

Neste artigo, você aprendeu que:

* Somente o Azure confia no Azure Active Directory para o gerenciamento de identidades.
* Uma assinatura tem o escopo mais alto de gerenciamento de recursos, e cada assinatura é associada a um locatário do Azure Active Directory. Somente os usuários no locatário do Azure Active Directory associado podem acessar recursos na assinatura.
* Há três níveis de escopo de gerenciamento de recursos: assinatura, grupo de recursos e recurso. As permissões são atribuídas em cada escopo usando funções RBAC. As funções RBAC são herdadas de um escopo superior para reduzir o escopo.

## <a name="next-steps"></a>Próximas etapas

Volte para a [visão geral do estágio básico de adoção](overview.md) para saber como implementar esse modelo governança. Em seguida, selecione um tipo de carga de trabalho e saiba como implantá-lo.