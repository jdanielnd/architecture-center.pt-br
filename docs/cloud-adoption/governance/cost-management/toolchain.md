---
title: 'CAF: Ferramentas de Gerenciamento de Custos no Azure'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Ferramentas de Gerenciamento de Custos no Azure
author: BrianBlanchard
ms.openlocfilehash: 58dfa604863f704fd9b9fbb8d0693447cecdaf84
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58246597"
---
# <a name="cost-management-tools-in-azure"></a>Ferramentas de Gerenciamento de Custos no Azure

O [Gerenciamento de Custos](overview.md) é uma das [Cinco disciplinas de Governança de Nuvem](../governance-disciplines.md). Essa disciplina concentra-se em maneiras de estabelecer planos de gastos com a nuvem, alocação de orçamentos de nuvem, monitoramento e imposição de orçamentos de nuvem, detecção de anomalias de alto custo e ajuste do plano de governança de nuvem quando os gastos reais estiverem desalinhados.

A seguir, é apresentada uma lista de ferramentas nativas do Azure que podem ajudar a amadurecer as políticas e os processos que dão suporte a essa disciplina de governança.

|  | [Portal do Azure](https://azure.microsoft.com/features/azure-portal/)  | [Gerenciamento de Custos do Azure](/azure/cost-management/overview-cost-mgt)  | [Pacote de conteúdo do EA do Azure](/power-bi/service-connect-to-azure-enterprise)  | [Azure Policy](/azure/governance/policy/overview) |
|---------|---------|---------|---------|---------|
|O Enterprise Agreement é necessário?     | Não          | Sim (não é necessário com o [Cloudyn](/azure/cost-management/overview))         | Sim         | Não          |
|Controle de orçamento     | Não          | Sim         | Não          | Sim         |
|Monitorar gastos em um único recurso    | Sim         | sim         | sim         | Não          |
|Monitorar gastos em vários recursos    | Não          | sim        | sim         | Não          |
|Controlar gastos em um único recurso     | Sim - dimensionamento manual         | Sim         | Não          | Sim         |
|Impor gastos em vários recursos    | Não          | Sim         | Não          | Sim         |
|Monitorar e detectar tendências     | Sim - limitado         | Sim        | sim         | Não          |
|Detectar anomalias de gastos     | Não          | sim        | sim         | Não         |
|Socializar desvios     | Não         | sim        | sim        | Não         |
