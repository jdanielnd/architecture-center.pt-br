---
title: Padrões de desempenho e Escalabilidade
titleSuffix: Cloud Design Patterns
description: O desempenho é uma indicação de que a capacidade de resposta de um sistema para executar qualquer ação dentro de um determinado intervalo, enquanto a escalabilidade é a capacidade de um sistema para lidar com o aumento de carga sem impacto no desempenho ou para os recursos disponíveis ser prontamente aumentado. Aplicativos de nuvem normalmente encontram cargas de trabalho variável e picos na atividade. Prever esses, especialmente em um cenário de multilocatário, é quase impossível. Em vez disso, os aplicativos devem ser capazes de expansão dentro dos limites para atender aos picos de demanda e a escala em quando a demanda diminui. Os problemas de escalabilidade de computação não apenas instâncias, mas outros elementos, como o armazenamento de dados, infraestrutura de mensagens e muito mais.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: a57194f70218a8294507bd389ea9526660e3041b
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58248901"
---
# <a name="performance-and-scalability-patterns"></a>Padrões de desempenho e Escalabilidade

[!INCLUDE [header](../../_includes/header.md)]

O desempenho é uma indicação de que a capacidade de resposta de um sistema para executar qualquer ação dentro de um determinado intervalo, enquanto a escalabilidade é a capacidade de um sistema para lidar com o aumento de carga sem impacto no desempenho ou para os recursos disponíveis ser prontamente aumentado. Aplicativos de nuvem normalmente encontram cargas de trabalho variável e picos na atividade. Prever esses, especialmente em um cenário de multilocatário, é quase impossível. Em vez disso, os aplicativos devem ser capazes de expansão dentro dos limites para atender aos picos de demanda e a escala em quando a demanda diminui. Os problemas de escalabilidade de computação não apenas instâncias, mas outros elementos, como o armazenamento de dados, infraestrutura de mensagens e muito mais.

|                           Padrão                            |                                                                        Resumo                                                                         |
|--------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
|               [Cache-Aside](../cache-aside.md)               |                                                   Carregar dados sob demanda em um cache de um armazenamento de dados                                                   |
|                      [CQRS](../cqrs.md)                      |                           Separar as operações que leem dados de operações que atualizam dados usando interfaces separadas.                           |
|            [Fornecimento de Evento](../event-sourcing.md)            |                     Use um armazenamento só de acréscimo para registrar a série inteira de eventos que descrevem as ações realizadas nos dados em um domínio.                      |
|               [Tabela de Índice](../index-table.md)               |                                Crie índices nos campos em armazenamentos de dados que são frequentemente referenciados por consultas.                                |
|         [Exibição Materializada](../materialized-view.md)         |       Gere exibições pré-preenchidas nos dados em um ou mais armazenamentos de dados quando os dados não estiverem formatados como o ideal para as operações de consulta necessárias.        |
|            [Fila de Prioridade](../priority-queue.md)            | Priorize as solicitações enviadas a serviços para que as solicitações com uma prioridade mais alta sejam recebidas e processadas mais rapidamente do que aquelas com uma prioridade mais baixa. |
| [Nivelamento de Carga Baseado em Fila](../queue-based-load-leveling.md) |              Use uma fila que funcione como um buffer entre uma tarefa e um serviço que ela invoca para simplificar cargas pesadas intermitentes.               |
|                  [Fragmentação](../sharding.md)                  |                                           Divida um armazenamento de dados em um conjunto de partições horizontais ou fragmentos.                                           |
|    [Hospedagem de Conteúdo Estático](../static-content-hosting.md)    |                          Implante conteúdo estático em um serviço de armazenamento baseado em nuvem que pode enviá-lo diretamente para o cliente.                          |
|                [Limitação](../throttling.md)                |                Controle o consumo de recursos usados por uma instância de um aplicativo, um locatário individual ou todo o serviço.                 |
