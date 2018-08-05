---
title: Princípios de design para aplicativos do Azure
description: Princípios de design para aplicativos do Azure
author: MikeWasson
ms.openlocfilehash: 462896098c668c0775464ca498925266cd73c6e1
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206790"
---
# <a name="design-principles-for-azure-applications"></a>Princípios de design para aplicativos do Azure

Siga esses princípios de design para tornar seu aplicativo mais escalonável, flexível e gerenciável. 

**[Design de autorrecuperação](self-healing.md)**. Em um sistema distribuído, falhas acontecem. Projete seu aplicativo para ter autorrecuperação quando houver falhas.

**[Tornar todas as coisas redundantes](redundancy.md)**. Crie redundância no seu aplicativo, evite ter pontos únicos de falha.
 
**[Minimizar a coordenação](minimize-coordination.md)**. Minimize a coordenação entre os serviços de aplicativos para atingir a escalabilidade.
 
**[Design de expansão](scale-out.md)**. Crie seu aplicativo para que ele possa ser dimensionado horizontalmente, adicionando ou removendo novas instâncias conforme a necessidade.

**[Partição de limites](partition.md)**. Use particionamento para contornar o banco de dados, rede e limites de computação.

**[Design para operações](design-for-operations.md)**. Crie seu aplicativo para que a equipe de operações tenha as ferramentas necessárias.

**[Serviços gerenciados pelo usuário](managed-services.md)**. Quando possível, utilize a plataforma como um serviço (PaaS) em vez de infraestrutura como serviço (IaaS).

**[Use o melhor armazenamento de dados para o trabalho](use-the-best-data-store.md)**. Escolha a tecnologia de armazenamento que é o mais adequado para seus dados e como ele será usado. 
 
**[Design para evolução](design-for-evolution.md)**. Alterar todos os aplicativos com êxito ao longo do tempo. Um design evolutivo é a chave para a inovação contínua.

**[Crie de acordo com as necessidades de negócios](build-for-business.md)**. Todas as decisões de design devem ser justificado por um requisito de negócios.

