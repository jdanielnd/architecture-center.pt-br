---
title: 'Explicador: o que é o Azure Resource Manager?'
description: Explica o funcionamento interno do Azure Resource Manager
author: petertay
ms.openlocfilehash: 60f09901bdc4b292abd73335b78c7d56a76f27a6
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/09/2018
ms.locfileid: "29062024"
---
# <a name="explainer-what-is-azure-resource-manager"></a>Explicador: o que é o Azure Resource Manager?

No explicador [Como funciona o Azure?](azure-explainer.md), você aprendeu sobre a arquitetura interna do Azure. Essa arquitetura inclui um front-end que hospeda os aplicativos distribuídos que gerenciam os serviços internos do Azure.

O front-end do Azure inclui um serviço chamado Azure Resource Manager. O Azure Resource Manager é responsável pelo ciclo de vida dos recursos hospedados no Azure, desde a criação até a exclusão. Há várias maneiras de interagir com o Azure Resource Manager &mdash; usando o PowerShell, a interface de linha de comando do Azure, SDKs &mdash;, mas cada uma dessas ferramentas é apenas um wrapper em uma API RESTful hospedada pelo Azure Resource Manager.

A API RESTful fornecida pelo Azure Resource Manager é uma interface consistente em um conjunto de **provedores de recursos**. Os provedores de recursos são simplesmente serviços do Azure que criam, leem, atualizam e excluem recursos do Azure. Na verdade, a API RESTful inclui métodos para cada uma dessas funções. 

A API RESTful exige um token de acesso para o usuário, uma **ID da assinatura** e algo novo – uma **ID de grupo de recursos**. Vamos abordar os grupos de recursos no artigo do [explicador do grupo de recursos](resource-group-explainer.md). O Azure Resource Manager também exige a **ID de locatário**, que é codificada como parte do token de acesso. 

Quando uma chamada à API válida para criar um recurso é recebida, o Azure Resource Manager encontra a capacidade na região especificada e copia todos os arquivos necessários para um local de preparo. A solicitação é então enviada para o controlador de malha no rack e o controlador de malha aloca os recursos. O controlador de malha responde à solicitação com uma notificação de êxito ou falha, juntamente com uma **ID do recurso** para o recurso recém-criado. Essas quatro IDs são armazenadas internamente no Azure e, em conjunto, atuam como um identificador exclusivo de um recurso implantado.

## <a name="next-steps"></a>Próximas etapas

* Agora que você entendeu o funcionamento interno do Azure Resource Manager, saiba mais [sobre os grupos de recursos](resource-group-explainer.md) para ajudá-lo a criar seu primeiro grupo de recursos.
