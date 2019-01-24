---
title: Padrões de Gerenciamento de Dados
titleSuffix: Cloud Design Patterns
description: O gerenciamento de dados é o elemento principal de aplicativos em nuvem e influencia a maioria dos atributos de qualidade. Os dados normalmente são hospedados em diferentes locais e em vários servidores por motivos como desempenho, escalabilidade ou a disponibilidade, e isso pode apresentar uma série de desafios. Por exemplo, deve ser mantida a consistência dos dados e dados normalmente precisam ser sincronizados em diferentes locais.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 25571a431836656856ed3f299455dfdb94ae3477
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486974"
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
