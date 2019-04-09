---
title: 'CAF: Governança no CAF Microsoft Azure'
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Governança no CAF Microsoft Azure
author: BrianBlanchard
ms.openlocfilehash: ce407de0daa590e767382346692c80e0a113bb3c
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/08/2019
ms.locfileid: "59068830"
---
# <a name="governance-in-the-microsoft-caf-for-azure"></a>Governança no CAF Microsoft Azure

A nuvem cria novos paradigmas sobre as tecnologias que dão suporte ao negócio. Esses novos paradigmas também causam turnos na maneira como essas tecnologias são adotadas, gerenciadas e controladas. Quando datacenters inteiros podem ser destruídos e recriados com uma linha de código executado a partir de um processo autônomo, é preciso repensar as abordagens tradicionais. Isso é igualmente verdadeiro quando se trata de governança.

Para organizações com as políticas existentes que regem ambientes de TI locais, governança de nuvem deve completar essas políticas. No entanto, o nível de integração de política corporativa entre os locais e a nuvem irão variar, dependendo do amadurecimento da governança de nuvem e propriedade digital na nuvem. À medida que o estado da nuvem evolui ao longo do tempo, o mesmo ocorrerá com os processos e políticas de governança da nuvem.

As orientações nesta seção de CAF foram projetadas para duas finalidades:

* Fornecer percursos de informações valiosas dos clientes que representam as experiências comuns que os clientes encontram normalmente. Cada um desses riscos comercias encapsulados, políticas corporativas para mitigação de risco e diretriz de design para implementar soluções técnicas. Por necessidade, as diretrizes de design são específicas para o Azure. Todas as outras diretrizes nesses percursos puderam ser aplicadas como parte de uma abordagem de nuvem independente ou com várias nuvens.
* Ajudar os leitores a criar soluções personalizadas de governança que podem atender a uma variedade de necessidades comerciais, incluindo a governança de várias nuvens públicas, por meio de obter orientações detalhadas sobre o desenvolvimento de políticas corporativas, processos e as ferramentas.

Este conteúdo destina-se a equipe de Governança de Nuvem. Também é relevante para os arquitetos de nuvem que precisam desenvolver uma base sólida de governança de nuvem.

## <a name="audience"></a>Público

O conteúdo no CAF afeta os negócios, tecnologia e cultura das empresas. Esta seção de CAF irá interagir intensamente com as equipes de Segurança de TI,Governança de TI, finanças, líderes de linha de negócios, rede, identidade e adoção de nuvem. Há várias co-dependências nessas personas que exigem uma abordagem facilitadora pelos arquitetos de nuvem usando essa diretriz. A facilitação com essas equipes pode ser um esforço único, mas em alguns casos, isso resultará em interações recorrentes com outras personas.

O arquiteto de nuvem serve como o líder de ideias e facilitador para reunir esses públicos. O conteúdo nesta coleção de guias foi projetado para ajudar o arquiteto de nuvem a facilitar a conversa certa, com o público correto, para orientar decisões necessárias. A transformação de negócios que é capacitada pela nuvem depende da função do arquiteto de nuvem para ajudar a guiar as decisões em toda a empresa e IT.

**Especialização de arquiteto de nuvem nesta seção:** Cada seção de CAF representa outra especialização ou variante da função do Arquiteto de Nuvem. Esta seção do CAF destina-se de arquitetos da nuvem com uma paixão por redução ou reduzindo os riscos de técnicos. Muitos provedores de nuvem se referem a esses especialistas como responsáveis da nuvem. Preferimos guardiões de nuvem ou, coletivamente, a equipe de Governança de nuvem. Em cada jornada do cliente acionável, os artigos mostram como a composição e a função da equipe de governança de nuvem podem ser alteradas ao longo do tempo.

## <a name="using-this-guide"></a>Usando este guia

Para leitores que desejam seguir este guia do início ao fim, esse conteúdo ajudará a desenvolver uma estratégia de governança de nuvem robusta em paralelo à implementação de nuvem. A diretriz orienta o leitor na teoria e implementação de uma estratégia.

Para um curso intensivo na teoria e acesso rápido a implementação do Azure, comece com o [visão geral dos percursos de governança acionáveis](./journeys/overview.md). Por meio deste guia, o leitor pode começar pequeno e evoluir suas necessidades de governança em paralelo aos seus esforços de adoção de nuvem.

## <a name="next-steps"></a>Próximas etapas

Examine os percursos de governança acionáveis.

> [!div class="nextstepaction"]
> [Percursos de governança acionáveis](./journeys/overview.md)
