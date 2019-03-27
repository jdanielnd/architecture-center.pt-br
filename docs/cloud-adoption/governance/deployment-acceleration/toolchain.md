---
title: 'CAF: Ferramentas de Aceleração de implantação no Azure'
description: Ferramentas de Aceleração de implantação no Azure
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
author: BrianBlanchard
ms.openlocfilehash: cd00f2fa132f5f177ccc12f61be8b5342b71197d
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58247417"
---
# <a name="deployment-acceleration-tools-in-azure"></a>Ferramentas de Aceleração de implantação no Azure

[A Aceleração de implantação](overview.md) é uma das [Cinco Disciplinas de Governança de Nuvem](../governance-disciplines.md). Esta disciplina se concentra em maneiras de estabelecer políticas para controlar a configuração de ativos ou implantação. Dentro das cinco disciplinas de Governança de Nuvem, a governança de configuração inclui implantações, alinhamento de configuração e estratégias HA/DR. Isso pode ocorrer por meio de atividades manuais ou atividades de DevOps totalmente automatizadas. Em ambos os casos, as políticas permanecerão basicamente as mesmas.

Responsáveis da nuvem, guardiões da nuvem e arquitetos da nuvem com interesse na governança, cada um provavelmente irá investir muito tempo na disciplina de Aceleração de implantação, o que codifica as políticas e os requisitos entre vários esforços de adoção de nuvem. As ferramentas nessa cadeia de ferramentas são importantes para a equipe de Governança de nuvem e devem ser uma prioridade alta no caminho de aprendizado da equipe.

A seguir, é apresentada uma lista de ferramentas nativas do Azure que podem ajudar a amadurecer as políticas e os processos que dão suporte a essa disciplina de governança.

|  | Azure Policy | Grupos de Gerenciamento do Azure | Modelos do Azure Resource Manager | Azure Blueprints | Gráfico de Recursos do Azure | Gerenciamento de Custos do Azure |
|---------|---------|---------|---------|---------|---------|---------|
|Implementar políticas corporativas     |Sim |Não  |Não  |Não | Não |Não  |
|Mover políticas entre assinaturas     |Obrigatório |Sim  |Não  |Não | Não |Não  |
|Implantar recursos definidos     |Não  |Não   |Sim  |Não | Não |Não  |
|Criar ambientes em conformidade total      |Obrigatório |Obrigatório  |Obrigatório  |Sim | Não |Não  |
|Auditar políticas      |Sim |Não  |Não  |Não | Não |Não  |
|Recursos de Consulta do Azure      |Não  |Não  |Não  |Não  |Sim |Não  |
|Relatório sobre o custo dos recursos      |Não  |Não  |Não  |Não |Não  |Sim |

A seguir estão as ferramentas adicionais que podem ser necessárias para atingir os objetivos de aceleração de implantação específicos. Geralmente, essas ferramentas são usadas fora da equipe de governança, mas ainda são consideradas um aspecto de aceleração de implantação como uma disciplina.

|  |Portal do Azure  |Modelos do Azure Resource Manager  |Azure Policy  | Azure DevOps | Serviço de Backup do Azure | Azure Site Recovery |
|---------|---------|---------|---------|---------|---------|---------|
|Implantação manual (único ativo)     | Sim | sim  | Não   | Não é eficiente | Não  | Sim |
|Implantação manual (todo o ambiente)     | Não é eficiente | Sim | Não   | Não é eficiente | Não  | Sim |
|Implantação automatizada (todo o ambiente)     | Não   | Sim  | Não   | Sim  | Não  | Sim |
|Atualize a configuração de um servidor único     | Sim | Sim | Não é eficiente | Não é eficiente | Não  | Sim, durante a replicação |
|Atualizar a configuração de um ambiente completo     | Não é eficiente | Sim | sim | sim  | Não  | Sim, durante a replicação |
|Gerenciar o descompasso da configuração     | Não é eficiente | Não é eficiente | Sim  | sim  | Não  | Sim, durante a replicação |
|Criar um pipeline automatizado para implantar o código e configurar ativos (DevOps)     | Não  | Não | Não  | Sim | Não | Não  |
|Recuperar dados durante uma interrupção ou violação de SLA     | Não  | Não | Não  | sim | sim | Sim |
|Recuperar aplicativos e dados durante uma interrupção ou violação de SLA     | Não  | Não | Não  | Sim | Não  | Sim |

Além de ferramentas nativas do Azure mencionadas acima, é comum que os clientes usem ferramentas de terceiros para facilitar as implantações de Aceleração de implantação e DevOps.
