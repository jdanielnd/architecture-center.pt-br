---
title: Criar de acordo com as necessidades da empresa
titleSuffix: Azure Application Architecture Guide
description: Todas as decisões de design devem ser justificado por um requisito de negócios.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 9529d1e11dc67144d2dcb641ea666de7c2205f20
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486515"
---
# <a name="build-for-the-needs-of-the-business"></a>Criar de acordo com as necessidades da empresa

## <a name="every-design-decision-must-be-justified-by-a-business-requirement"></a>Todas as decisões de design devem ser justificadas por um requisito de negócios

Esse princípio de design pode parecer óbvio, mas é crucial ter em mente ao projetar uma solução. Você prevê milhões de usuários ou alguns milhares? Uma interrupção de aplicativo de uma hora é aceitável? Você espera grandes picos de tráfego ou uma carga de trabalho muito previsível? Por fim, todas as decisões de design devem ser justificadas por um requisito de negócios.

## <a name="recommendations"></a>Recomendações

**Defina os objetivos de negócios**, incluindo o objetivo de tempo de recuperação (RTO), o objetivo de ponto de recuperação (RPO) e a interrupção máxima tolerável (MTO). Esses números devem informar decisões sobre a arquitetura. Por exemplo, para alcançar um baixo RTO, você pode implementar o failover automático para uma região secundária. Mas, se sua solução puder tolerar um RTO superior, esse nível de redundância poderá ser desnecessário.

**Documente os contratos de nível de serviço (SLA) e os objetivos de nível de serviço (SLO)**, inclusive as métricas de desempenho e disponibilidade. Você pode criar uma solução que fornece disponibilidade de 99,95%. Isso é suficiente? A resposta é uma decisão de negócios.

**Modele de aplicativo ao redor do domínio da empresa**. Comece pela análise dos requisitos de negócios. Use esses requisitos para modelar o aplicativo. Considere o uso de uma abordagem de design orientado a domínio (DDD) para criar [modelos de domínio][domain-model] que refletem os processos de negócios e os casos de uso.

**Capture os requisitos funcionais e não funcionais**. Os requisitos funcionais permitem que você avalie se o aplicativo faz o certo. Os requisitos não funcionais permitem que você avalie se o aplicativo executa *bem* essas ações. Em particular, certifique-se de que você entende seus requisitos de escalabilidade, disponibilidade e latência. Esses requisitos influenciarão as decisões de design e a opção de tecnologia.

**Decomponha por carga de trabalho**. O termo "carga de trabalho" neste contexto significa um recurso discreto ou uma tarefa de computação, que pode ser logicamente separada de outras tarefas. As cargas de trabalho diferentes podem ter diferentes requisitos de disponibilidade, escalabilidade, consistência de dados e recuperação de desastres.

**Planeje o crescimento**. Uma solução pode atender às suas necessidades atuais, em termos de número de usuários, volume de transações, armazenamento de dados e assim por diante. No entanto, um aplicativo robusto pode lidar com o crescimento sem grandes alterações de arquitetura. Veja [Design para expansão](scale-out.md) e [Partição de limites](partition.md). Além disso, considere que o modelo de negócios e os requisitos de negócios provavelmente serão alterado ao longo do tempo. Se o modelo de serviço e modelos de dados de um aplicativo forem muito rígidos, ficará difícil desenvolver o aplicativo para novos casos de uso e cenários. Veja [Design para evolução](design-for-evolution.md).

**Gerencie os custos**. Em um aplicativo local tradicional, você paga antecipadamente pelo hardware (CAPEX). Em um aplicativo de nuvem, você paga pelos recursos consumidos. Certifique-se de que você compreende o modelo de preços para os serviços que consome. O custo total incluirá o uso de largura de banda de rede, armazenamento, endereços IP, consumo do serviço e outros fatores. Para saber mais, veja [Preços do Azure][pricing]. Além disso, considere os custos operacionais. Na nuvem, você não precisa gerenciar o hardware ou outra infraestrutura, mas ainda precisa gerenciar seus aplicativos, incluindo DevOps, resposta a incidentes, recuperação de desastres e assim por diante.

[domain-model]: https://martinfowler.com/eaaCatalog/domainModel.html
[pricing]: https://azure.microsoft.com/pricing/
