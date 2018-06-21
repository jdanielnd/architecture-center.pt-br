---
title: Padrões de Gerenciamento e Monitoramento
description: Os aplicativos de nuvem são executados em um data center remoto em que você não tem controle total da infraestrutura ou, em alguns casos, o sistema operacional. Isso pode tornar o gerenciamento e o monitoramento mais difíceis do que uma implantação local. Os aplicativos devem expor informações de tempo de execução que os operadores e os administradores podem usar para gerenciar e monitorar o sistema, bem como dar suporte a mudanças nos requisitos de negócios e a personalização sem a necessidade de parar ou reimplantar o aplicativo.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: faab137d59ce952e169839a71abdbbb3103ea772
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30846783"
---
# <a name="management-and-monitoring-patterns"></a>Padrões de Gerenciamento e Monitoramento

Os aplicativos de nuvem são executados em um data center remoto em que você não tem controle total da infraestrutura ou, em alguns casos, o sistema operacional. Isso pode tornar o gerenciamento e o monitoramento mais difíceis do que uma implantação local. Os aplicativos devem expor informações de tempo de execução que os operadores e os administradores podem usar para gerenciar e monitorar o sistema, bem como dar suporte a mudanças nos requisitos de negócios e a personalização sem a necessidade de parar ou reimplantar o aplicativo.


|                              Padrão                               |                                                              Resumo                                                              |
|--------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
|                   [Embaixador](../ambassador.md)                   |                 Crie serviços auxiliares que enviam solicitações de rede em nome de um consumidor de serviço ou aplicativo.                 |
|        [Camada anticorrupção](../anti-corruption-layer.md)        |                       Implemente uma camada de fachada ou adaptador entre um aplicativo moderno e um sistema herdado.                       |
| [Repositório de configuração externo](../external-configuration-store.md) |                Mova as informações de configuração para fora do pacote de implantação de aplicativo para um local centralizado.                |
|          [Agregação de Gateway](../gateway-aggregation.md)          |                          Use um gateway para agregar várias solicitações individuais em uma única solicitação.                           |
|           [Descarregamento de Gateway](../gateway-offloading.md)           |                              Descarregue a funcionalidade de serviço especializado ou compartilhado para um proxy do gateway.                              |
|              [Roteamento de Gateway](../gateway-routing.md)              |                                   Faça o roteamento de solicitações para vários serviços usando um único ponto de extremidade.                                    |
|   [Monitoramento do ponto de extremidade de integridade](../health-endpoint-monitoring.md)   |   Implemente verificações funcionais dentro de um aplicativo que ferramentas externas podem acessar por meio de pontos de extremidade expostos a intervalos regulares.    |
|                      [Sidecar](../sidecar.md)                      |         Implante os componentes de um aplicativo em um processo ou contêiner separado para fornecer isolamento e encapsulamento.          |
|                    [Estrangulador](../strangler.md)                    | Migre incrementalmente um sistema herdado substituindo gradualmente partes específicas de funcionalidade por serviços e aplicativos novos. |

