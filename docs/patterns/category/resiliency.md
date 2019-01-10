---
title: Padrões de resiliência
titleSuffix: Cloud Design Patterns
description: A resiliência é a capacidade de um sistema tratar normalmente e recuperar-se de falhas. A natureza da hospedagem em nuvem, em que os aplicativos costumam ser multilocatário, usam serviços de plataforma compartilhada, competem por recursos e largura de banda, comunicam-se pela Internet e são executados no hardware de mercadoria significa que há uma maior probabilidade de que ocorram falhas tanto transitórias quanto mais permanentes. A detecção de falhas e a recuperação rápida e eficaz são necessárias para manter a resiliência.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: d478c0fb42e89c6bb5d84b4d077259ff4bf35f16
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009705"
---
# <a name="resiliency-patterns"></a>Padrões de resiliência

A resiliência é a capacidade de um sistema tratar normalmente e recuperar-se de falhas. A natureza da hospedagem em nuvem, em que os aplicativos costumam ser multilocatário, usam serviços de plataforma compartilhada, competem por recursos e largura de banda, comunicam-se pela Internet e são executados no hardware de mercadoria significa que há uma maior probabilidade de que ocorram falhas tanto transitórias quanto mais permanentes. A detecção de falhas e a recuperação rápida e eficaz são necessárias para manter a resiliência.

|                            Padrão                             |                                                                                                      Resumo                                                                                                       |
|----------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                   [Bulkhead](../bulkhead.md)                   |                                                     Isole os elementos de um aplicativo em pools para que, se um falhar, os outros continuarão a funcionar.                                                      |
|            [Interruptor de Circuito](../circuit-breaker.md)            |                                                  Trate as falhas que possam consumir uma quantidade variável de tempo para serem corrigidas ao se conectar a um serviço ou recurso remoto.                                                   |
|   [Transação de Compensação](../compensating-transaction.md)   |                                                      Desfaça o trabalho executado por uma série de etapas que, juntas, definem uma operação que acabe sendo consistente.                                                       |
| [Monitoramento do Ponto de Extremidade de Integridade](../health-endpoint-monitoring.md) |                                            Implemente verificações funcionais dentro de um aplicativo cujas ferramentas externas podem acessar por meio de pontos de extremidade expostos em intervalos regulares.                                            |
|            [Eleição de Líder](../leader-election.md)            | Coordene as ações executadas por uma coleção de instâncias de tarefa de colaboração em um aplicativo distribuído elegendo uma instância como a líder que assume a responsabilidade por gerenciar as demais instâncias. |
|  [Nivelamento de Carga Baseado em Fila](../queue-based-load-leveling.md)  |                                            Use uma fila que funcione como um buffer entre uma tarefa e um serviço que ela invoca para simplificar cargas pesadas intermitentes.                                             |
|                      [Tentar Novamente](../retry.md)                      |             Permita que um aplicativo trate falhas previstas e temporárias quando tentar se conectar a um serviço ou recurso de rede ao repetir de forma transparente uma operação que falhou anteriormente.             |
| [Supervisor de Agente do Agendador](../scheduler-agent-supervisor.md) |                                                            Coordene um conjunto de ações em um conjunto distribuído de serviços e outros recursos remotos.                                                            |