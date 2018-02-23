---
title: "Explicador – O que é um grupo de recursos do Azure?"
description: "Explica a função interna do Azure de um grupo de recursos"
author: petertay
ms.openlocfilehash: e7c7334bd88c28f57498486bd2bed3c349565222
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/09/2018
---
# <a name="what-is-an-azure-resource-group"></a>O que é um grupo de recursos do Azure?

No artigo do explicador [O que é o Azure Resource Manager?](resource-manager-explainer.md), você aprendeu que o Azure Resource Manager exige um **identificador de grupo de recursos** quando uma chamada é feita para criar, ler, atualizar ou excluir um recurso. Essa ID de grupo de recursos se refere a um **grupo de recursos**. Um grupo de recursos é simplesmente um identificador que o Azure Resource Manager aplica aos recursos para agrupá-los. Essa ID de grupo de recursos permite que o Azure Resource Manager execute operações em um grupo de recursos que compartilha essa ID.

Por exemplo, um usuário pode fazer uma chamada de **exclusão** a uma API RESTful do Azure Resource Manager especificando a ID de grupo de recursos sem incluir nenhuma ID de recurso específica. O Azure Resource Manager consulta um banco de dados interno do Azure em busca de todos os recursos com a ID do grupo de recursos especificada e chama a API RESTful para excluir cada um dos recursos.

Um grupo de recursos não pode incluir recursos de assinaturas diferentes. Isso ocorre porque há uma relação um-para-muitos entre a ID de locatário e a ID de assinatura &mdash; várias assinaturas podem confiar no mesmo locatário para fornecer autenticação e autorização, mas cada assinatura pode confiar em apenas um locatário. Também há uma relação um-para-muitos entre a ID de assinatura e a ID de grupo de recursos &mdash; vários grupos de recursos podem pertencer à mesma assinatura, mas cada grupo de recursos pode pertencer a apenas uma assinatura. Por fim, há uma relação um-para-muitos entre a ID de grupo de recursos e a ID do recurso &mdash; um único grupo de recursos pode ter vários recursos, mas cada recurso pode pertencer a apenas um único grupo de recursos.

## <a name="next-steps"></a>Próximas etapas

* Agora que você aprendeu sobre grupos de recursos do Azure, obtenha conhecimento básico [sobre como restringir o acesso a recursos](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json). Embora isso não faça parte do estágio básico de adoção, ele é importante durante o estágio intermediário de adoção. Em seguida, você pode [criar seu primeiro grupo de recursos](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json) e ler as [diretrizes de design para grupos de recursos do Azure](resource-group.md).
