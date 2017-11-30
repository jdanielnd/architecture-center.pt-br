---
title: "Padrões de Gerenciamento de Dados"
description: "O gerenciamento de dados é o elemento principal de aplicativos em nuvem e influencia a maioria dos atributos de qualidade. Os dados normalmente são hospedados em diferentes locais e em vários servidores por motivos como desempenho, escalabilidade ou a disponibilidade, e isso pode apresentar uma série de desafios. Por exemplo, deve ser mantida a consistência dos dados e dados normalmente precisam ser sincronizados em diferentes locais."
keywords: "padrão de design"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: a009a06268f114ab7be4544dd81710612dabd8f4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="data-management-patterns"></a>Padrões de Gerenciamento de Dados

[!INCLUDE [header](../../_includes/header.md)]

O gerenciamento de dados é o elemento principal de aplicativos em nuvem e influencia a maioria dos atributos de qualidade. Os dados normalmente são hospedados em diferentes locais e em vários servidores por motivos como desempenho, escalabilidade ou a disponibilidade, e isso pode apresentar uma série de desafios. Por exemplo, deve ser mantida a consistência dos dados e dados normalmente precisam ser sincronizados em diferentes locais.

| Padrão | Resumo |
| ------- | ------- |
| [Cache-Aside](../cache-aside.md) | Carregar dados sob demanda em um cache de um armazenamento de dados |
| [CQRS](../cqrs.md) | Separar as operações que leem dados de operações que atualizam dados usando interfaces separadas. |
| [Fornecimento de Evento](../event-sourcing.md) | Use um armazenamento só de acréscimo para registrar a série inteira de eventos que descrevem as ações realizadas nos dados em um domínio. |
| [Tabela de Índice](../index-table.md) | Crie índices nos campos em armazenamentos de dados que são frequentemente referenciados por consultas. |
| [Exibição Materializada](../materialized-view.md) | Gere exibições pré-preenchidas nos dados em um ou mais armazenamentos de dados quando os dados não estiverem formatados como o ideal para as operações de consulta necessárias. |
| [Fragmentação](../sharding.md) | Divida um armazenamento de dados em um conjunto de partições horizontais ou fragmentos. |
| [Hospedagem de Conteúdo Estático](../static-content-hosting.md) | Implante conteúdo estático em um serviço de armazenamento baseado em nuvem que pode enviá-lo diretamente para o cliente. |
| [Valet Key](../valet-key.md) | Use um token ou chave que fornece aos clientes acesso direto e restrito a um determinado recurso ou serviço. |