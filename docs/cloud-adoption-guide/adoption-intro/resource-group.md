---
title: 'Diretrizes: design de grupo de recursos do Azure'
description: Diretrizes de design de grupo de recursos do Azure como parte de uma estratégia básica de adoção da nuvem
author: petertay
ms.openlocfilehash: ac6cbb03be8cdba020641d3b9034ad9d20101acf
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/09/2018
ms.locfileid: "29062094"
---
# <a name="guidance-azure-resource-group-design"></a>Diretrizes: design de grupo de recursos do Azure

No Azure, uma [grupo de recursos](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#resource-groups) é um contêiner lógico em que os recursos são agrupados. Cada recurso implantado no Azure precisa ser implantado em um único grupo de recursos.

## <a name="design-considerations"></a>Considerações sobre o design

- Todos os recursos de um grupo de recursos devem compartilhar o mesmo ciclo de vida. Ou seja, a prática deve ser implantar, atualizar e excluir recursos com o mesmo ciclo de vida como um grupo. Por exemplo, os recursos de computação de um único aplicativo Web normalmente são implantados como uma única unidade. No entanto, um banco de dados compartilhado com outros aplicativos Web provavelmente será gerenciado em um ciclo de vida diferente e deverá estar em seu próprio grupo de recursos.
- Um grupo de recursos pode conter recursos que residem em regiões diferentes.
- Todos os recursos de um único grupo de recursos devem ser associados a uma assinatura única. 
- Um recurso pode ser movido entre grupos de recursos, mas não para um grupo de recursos que contém um recurso implantado em outra assinatura.
- A atribuição de um grupo de recursos não afeta a conectividade nem a interação com recursos em outros grupos de recursos. Por exemplo, uma máquina virtual atribuída a um grupo de recursos pode se conectar a um banco de dados atribuído a outro grupo de recursos se há conectividade de rede entre eles.
- Um grupo de recursos pode ser usado para definir o escopo de controle de acesso para ações administrativas. Aplique as permissões de RBAC (controle de acesso baseado em função) no nível da assinatura ou no nível do grupo de recursos. As permissões atribuídas no nível da assinatura são herdadas no nível do grupo de recursos. Você aprenderá mais sobre as permissões de RBAC e de recursos nos estágios intermediários e avançados de adoção.

## <a name="proven-practices"></a>Práticas comprovadas

- No estágio básico, provavelmente, você gerencia apenas alguns projetos de POC (prova de conceito), cada um com um pequeno número de recursos. Como os recursos de POC geralmente compartilham o mesmo ciclo de vida, você pode criar um único grupo de recursos para cada um desses projetos.
- Durante o estágio intermediário de adoção, você gerenciará vários projetos. Diferentes tipos de projetos podem se beneficiar de outros designs de grupo de recursos. Se você pretende promover um de seus projetos de POC iniciais para produção, mova recursos para outro grupo de recursos, caso ele pertença à mesma assinatura. Portanto, neste estágio, você deverá implantar esses recursos na mesma assinatura, de modo que você possa reorganizar os recursos no futuro.

## <a name="next-steps"></a>Próximas etapas

* Agora que você aprendeu as práticas comprovadas para o estágio básico de adoção, estará apto a criar grupos de recursos e adicionar recursos a eles. Enquanto você estiver gerenciando um pequeno número de recursos neste estágio, o gerenciamento desses recursos se tornará mais complexo conforme você adicionar mais recursos. Saiba mais sobre as [convenções de nomenclatura e marcação do Azure](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json) para nomear e marcar seus recursos em preparação para o estágio intermediário de adoção.
