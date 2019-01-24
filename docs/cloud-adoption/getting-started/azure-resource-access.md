---
title: 'Adoção da Nuvem Empresarial: Gerenciamento de acesso aos recursos no Azure'
description: 'Explicação dos constructos de gerenciamento de acesso de recursos no Azure: Azure Resource Manager, assinaturas, grupos de recursos e recursos'
author: petertaylor9999
ms.date: 09/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 9f1d28cfeb062f44b2c58cd52c58f163d6de5b9d
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483064"
---
# <a name="enterprise-cloud-adoption-resource-access-management-in-azure"></a>Adoção da Nuvem Empresarial: Gerenciamento de acesso aos recursos no Azure

Em [O que é governança de recursos de nuvem?](what-is-governance.md), você aprendeu que governança é o processo contínuo de gerenciar, monitorar e auditar o uso de recursos do Azure para atender às metas e requisitos da sua organização. Antes de prosseguir para aprender a criar um modelo de governança, é importante entender os controles de gerenciamento de acesso a recursos no Azure. A configuração desses controles de gerenciamento de acesso a recursos constitui a base do seu modelo de governança.

Vamos começar examinando mais detalhadamente como os recursos são implantados no Azure. 

## <a name="what-is-an-azure-resource"></a>O que é um recurso do Azure?

No Azure, o termo **recurso** refere-se a uma entidade gerenciada pelo Azure. Por exemplo, máquinas virtuais, redes virtuais e contas de armazenamento são todas consideradas recursos do Azure.

![](../_images/governance-1-9.png)   
*Figura 1. Um recurso.*

## <a name="what-is-an-azure-resource-group"></a>O que é um grupo de recursos do Azure?

Cada recurso no Azure deve pertencer a um [grupo de recurso](/azure/azure-resource-manager/resource-group-overview#resource-groups). Um grupo de recursos é simplesmente um constructo lógico que agrupa vários recursos para que eles possam ser gerenciados como uma única entidade. Por exemplo, os recursos que compartilham um ciclo de vida semelhante, como os recursos para um [aplicativo de n camadas](/azure/architecture/guide/architecture-styles/n-tier) podem ser criados ou excluídos como um grupo. 

![](../_images/governance-1-10.png)   
*Figura 2. Um grupo de recursos contém um recurso.* 

Os grupos de recursos e os recursos que eles contêm são associados a uma **assinatura** do Azure. 

## <a name="what-is-an-azure-subscription"></a>O que é uma assinatura do Azure?

Uma assinatura do Azure é semelhante a um grupo de recursos em que ele é um constructo lógico que agrupa os grupos de recursos e seus recursos. No entanto, uma assinatura do Azure também está associada aos controles usados pelo Azure Resource Manager. O que isso significa? Vamos examinar mais detalhadamente o Azure Resource Manager para saber mais sobre a relação entre ele e uma assinatura do Azure.

![](../_images/governance-1-11.png)   
*Figura 3. Uma assinatura do Azure.*

## <a name="what-is-azure-resource-manager"></a>O que é o Azure Resource Manager?

Em [Como funciona o Azure?](what-is-azure.md) você aprendeu que o Azure inclui um "front-end" com muitos serviços que orquestram todas as funções do Azure. Um desses serviços é o [Azure Resource Manager](/azure/azure-resource-manager/), que hospeda a API RESTful usada pelos clientes para gerenciar recursos. 

![](../_images/governance-1-12.png)   
*Figura 4. Azure Resource Manager.*

A figura a seguir mostra três clientes: [PowerShell](/powershell/azure/overview), [o portal do Azure](https://portal.azure.com) e a [interface de linha de comando (CLI) do Azure](/cli/azure):

![](../_images/governance-1-13.png)   
*Figura 5. Os clientes do Azure conectam-se à API RESTful do Azure Resource Manager.*

Embora esses clientes se conectem ao Azure Resource Manager usando a API RESTful, o Azure Resource Manager não inclui a funcionalidade para gerenciar diretamente os recursos. Em vez disso, a maioria dos tipos de recursos no Azure têm seus próprios [ **provedores de recursos**](/azure/azure-resource-manager/resource-group-overview#terminology). 

![](../_images/governance-1-14.png)   
*Figura 6. Provedores de recursos do Azure.*

Quando um cliente faz uma solicitação para gerenciar um recurso específico, o Azure Resource Manager se conecta ao provedor de recursos para esse tipo de recurso concluir a solicitação. Por exemplo, se um cliente fizer uma solicitação para gerenciar um recurso de máquina virtual, o Azure Resource Manager se conecta ao provedor de recursos **Microsoft.compute**. 

![](../_images/governance-1-15.png)   
*Figura 7. O Azure Resource Manager se conecta ao provedor de recursos **Microsoft.compute** para gerenciar o recurso especificado na solicitação do cliente.*

O Azure Resource Manager exige que o cliente especifique um identificador para a assinatura e o grupo de recursos para gerenciar o recurso de máquina virtual. 

Agora que você tem uma compreensão de como funciona o Azure Resource Manager, vamos voltar para nossa discussão de como uma assinatura do Azure está associada aos controles usados pelo Azure Resource Manager. Antes que qualquer solicitação de gerenciamento de recursos possa ser executada pelo Azure Resource Manager, um conjunto de controles será verificado. 

O primeiro controle é que a solicitação deve ser feita por um usuário validado e o Azure Resource Manager tem uma relação confiável com [Azure Active Directory (Azure AD)](/azure/active-directory/) para fornecer a funcionalidade de identidade do usuário.

![](../_images/governance-1-16.png)   
*Figura 8. Azure Active Directory*

No Azure AD, os usuários são segmentados em **locatários**. Um locatário é um constructo lógico que representa uma instância segura e dedicada do Azure AD geralmente associada a uma organização. Cada assinatura está associada a um locatário do Azure Active Directory.

![](../_images/governance-1-17.png)   
*Figura 9. Um locatário do Microsoft Azure Active Directory associado a uma assinatura.*

Cada solicitação de cliente para gerenciar um recurso em uma assinatura específica exige que o usuário tenha uma conta associada ao locatário do Azure AD. 

O próximo controle é uma verificação de que o usuário tem permissão suficiente para fazer a solicitação. As permissões são atribuídas aos usuários que usam o [controle de acesso baseado em função (RBAC)](/azure/role-based-access-control/).

![](../_images/governance-1-18.png)   
*Figura 10. Cada usuário no locatário é atribuído a uma ou mais funções RBAC.*

Uma função RBAC especifica um conjunto de permissões que um usuário pode executar em um recurso específico. Quando a função é atribuída ao usuário, essas permissões são aplicadas. Por exemplo, a [função de **proprietário** interna](/azure/role-based-access-control/built-in-roles#owner) permite que um usuário realize qualquer ação em um recurso.

O próximo controle é uma verificação de que a solicitação será permitida sob as configurações especificadas da [política de recursos do Azure](/azure/governance/policy/). As políticas de recursos do Azure especificam as operações permitidas para um recurso específico. Por exemplo, uma política de recursos do Azure pode especificar que os usuários só têm permissão para implantar um tipo específico de máquina virtual.

![](../_images/governance-1-19.png)   
*Figura 11. Políticas de recursos do Azure.*

O próximo controle é uma verificação de que a solicitação não excede um [limite de assinatura do Azure](/azure/azure-subscription-service-limits). Por exemplo, cada assinatura tem um limite de 980 grupos de recursos por assinatura. Se uma solicitação é recebida para implantar um grupo de recursos adicionais depois que o limite for atingido, ela será negada.

![](../_images/governance-1-20.png)   
*Figura 12. Limites de recursos do Azure.* 

O controle final é uma verificação de que a solicitação está dentro do compromisso financeiro associado à assinatura. Por exemplo, se a solicitação é para implantar uma máquina virtual, o Azure Resource Manager verifica se a assinatura tem informações suficientes de pagamento.

![](../_images/governance-1-21.png)   
*Figura 13. Um compromisso financeiro é associado a uma assinatura.*

## <a name="summary"></a>Resumo

Neste artigo, você aprendeu sobre como o acesso aos recursos é gerenciado no Azure usando o Azure Resource Manager.

## <a name="next-steps"></a>Próximas etapas

Agora que você sabe como o acesso a recursos é gerenciado no Azure, saiba como criar um modelo de governança [para uma única equipe](../governance/governance-single-team.md) ou [várias equipes](../governance/governance-multiple-teams.md) usando esses serviços.

> [!div class="nextstepaction"]
> [Uma visão geral de governança](../governance/overview.md)
