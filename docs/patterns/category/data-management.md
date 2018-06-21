---
title: Padrões de Gerenciamento de Dados
description: O gerenciamento de dados é o elemento principal de aplicativos em nuvem e influencia a maioria dos atributos de qualidade. Os dados normalmente são hospedados em diferentes locais e em vários servidores por motivos como desempenho, escalabilidade ou a disponibilidade, e isso pode apresentar uma série de desafios. Por exemplo, deve ser mantida a consistência dos dados e dados normalmente precisam ser sincronizados em diferentes locais.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: b80c2a127af07e1e362e9078e2a476d33a26ef7c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847083"
---
# <a name="data-management-patterns"></a>Padrões de Gerenciamento de Dados

[!INCLUDE [header](../../_includes/header.md)]

O gerenciamento de dados é o elemento principal de aplicativos em nuvem e influencia a maioria dos atributos de qualidade. Os dados normalmente são hospedados em diferentes locais e em vários servidores por motivos como desempenho, escalabilidade ou a disponibilidade, e isso pode apresentar uma série de desafios. Por exemplo, deve ser mantida a consistência dos dados e dados normalmente precisam ser sincronizados em diferentes locais.


|                        Padrão                         |                                                                  Resumo                                                                  |
|--------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
|            [Cache-Aside](../cache-aside.md)            |                                            Carregar dados sob demanda em um cache de um armazenamento de dados                                             |
|                   [CQRS](../cqrs.md)                   |                    Separar as operações que leem dados de operações que atualizam dados usando interfaces separadas.                     |
|         [Fornecimento de Evento](../event-sourcing.md)         |               Use um armazenamento só de acréscimo para registrar a série inteira de eventos que descrevem as ações realizadas nos dados em um domínio.               |
|            [Tabela de Índice](../index-table.md)            |                         Crie índices nos campos em armazenamentos de dados que são frequentemente referenciados por consultas.                          |
|      [Exibição Materializada](../materialized-view.md)      | Gere exibições pré-preenchidas nos dados em um ou mais armazenamentos de dados quando os dados não estiverem formatados como o ideal para as operações de consulta necessárias. |
|               [Fragmentação](../sharding.md)               |                                    Divida um armazenamento de dados em um conjunto de partições horizontais ou fragmentos.                                     |
| [Hospedagem de Conteúdo Estático](../static-content-hosting.md) |                   Implante conteúdo estático em um serviço de armazenamento baseado em nuvem que pode enviá-lo diretamente para o cliente.                    |
|              [Valet Key](../valet-key.md)              |                 Use um token ou chave que fornece aos clientes acesso direto e restrito a um determinado recurso ou serviço.                 |

