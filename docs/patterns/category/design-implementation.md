---
title: Padrões de design e implementação
titleSuffix: Cloud Design Patterns
description: Um bom design abrange fatores como a consistência e a coerência no design do componente e implantação, facilidade de manutenção para simplificar a administração e desenvolvimento e capacidade de reutilização para permitir que componentes e subsistemas para ser usado em outros aplicativos e em outros cenários. As decisões tomadas durante a fase de design e implementação tem um grande impacto sobre a qualidade e o custo total de propriedade de aplicativos e serviços hospedados pela nuvem.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: ff609679bed4ee4aa88daf95ff1cab4ab2d90d48
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58248911"
---
# <a name="design-and-implementation-patterns"></a>Padrões de design e implementação

Um bom design abrange fatores como a consistência e a coerência no design do componente e implantação, facilidade de manutenção para simplificar a administração e desenvolvimento e capacidade de reutilização para permitir que componentes e subsistemas para ser usado em outros aplicativos e em outros cenários. As decisões tomadas durante a fase de design e implementação tem um grande impacto sobre a qualidade e o custo total de propriedade de aplicativos e serviços hospedados pela nuvem.

|                                Padrão                                 |                                                                                                      Resumo                                                                                                       |
|------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                     [Embaixador](../ambassador.md)                     |                                                         Crie serviços auxiliares que enviam solicitações de rede em nome de um consumidor de serviço ou aplicativo.                                                          |
|          [Camada anticorrupção](../anti-corruption-layer.md)          |                                                               Implemente uma camada de fachada ou adaptador entre um aplicativo moderno e um sistema herdado.                                                                |
|         [Back-ends para Front-ends](../backends-for-frontends.md)         |                                                          Crie serviços de back-end separados a serem consumidos por aplicativos de front-end específico ou interfaces.                                                          |
|                           [CQRS](../cqrs.md)                           |                                                         Separar as operações que leem dados de operações que atualizam dados usando interfaces separadas.                                                         |
| [Consolidação de Recursos de Computação](../compute-resource-consolidation.md) |                                                                     Consolidar várias tarefas ou operações em uma única unidade de computação                                                                      |
|   [Repositório de configuração externo](../external-configuration-store.md)   |                                                        Mova as informações de configuração para fora do pacote de implantação de aplicativo para um local centralizado.                                                         |
|            [Agregação de Gateway](../gateway-aggregation.md)            |                                                                   Use um gateway para agregar várias solicitações individuais em uma única solicitação.                                                                   |
|             [Descarregamento de Gateway](../gateway-offloading.md)             |                                                                      Descarregue a funcionalidade de serviço especializado ou compartilhado para um proxy do gateway.                                                                       |
|                [Roteamento de Gateway](../gateway-routing.md)                |                                                                            Faça o roteamento de solicitações para vários serviços usando um único ponto de extremidade.                                                                            |
|                [Eleição de Líder](../leader-election.md)                | Coordene as ações executadas por uma coleção de instâncias de tarefa de colaboração em um aplicativo distribuído elegendo uma instância como a líder que assume a responsabilidade por gerenciar as demais instâncias. |
|              [Pipes e Filtros](../pipes-and-filters.md)              |                                                     Divida uma tarefa que executa processamento complexo em uma série de elementos separados que podem ser reutilizados.                                                      |
|                        [Sidecar](../sidecar.md)                        |                                                  Implante os componentes de um aplicativo em um processo ou contêiner separado para fornecer isolamento e encapsulamento.                                                  |
|         [Hospedagem de Conteúdo Estático](../static-content-hosting.md)         |                                                        Implante o conteúdo estático para um serviço de armazenamento baseado em nuvem que pode enviá-las diretamente para o cliente.                                                        |
|                      [Strangler](../strangler.md)                      |                                         Migre incrementalmente um sistema herdado substituindo gradualmente partes específicas de funcionalidade por serviços e aplicativos novos.                                          |
