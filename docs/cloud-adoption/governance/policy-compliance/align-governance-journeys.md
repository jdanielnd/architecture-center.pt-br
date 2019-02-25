---
title: 'CAF: Alinhar os guias de design com a política.'
description: Como alinhar os guias de design com a política?
author: BrianBlanchard
ms.date: 01/04/2019
ms.openlocfilehash: 77a35597585e5f58967ea79d3c7b0fa17b6ab80e
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900196"
---
<!---
I've established policies. How to help developers adopt these policies?
Draft an architecture design guide.

[Aspirational statement] If you're using Azure, you can use one of ours as a starting point. The choose one of the following 6 as a starting point and mold it to fit your policies.
--->

<!-- markdownlint-disable MD026 -->

# <a name="how-do-you-align-design-guides-with-policy"></a>Como alinhar os guias de design com a política?

Depois de ter [definido as políticas de nuvem](define-policy.md) com base nos seus [riscos identificados](understanding-business-risk.md), você precisará gerar orientação prática que se alinha a essas políticas para sua equipe de TI e desenvolvedores se refiram. Esboçar um guia de design de governança de nuvem permite que você especifique escolhas específicas, estruturais, tecnológicas e processuais com base nas instruções de política que você gerou para cada uma das cinco disciplinas de governança.

Um guia de design de governança de nuvem deve estabelecer as escolhas de arquitetura e padrões de design para cada um dos componentes básicos da infraestrutura de implantações de nuvem que melhor atendem às suas necessidades de política. De forma paralela, forneça uma explicação de alto nível a respeito das ferramentas de tecnologia e processos que darão suporte a essas decisões de design.

Embora suas instruções de análise e a política de risco possam, até certo ponto, ser independente de plataforma de nuvem, o guia de design deve fornecer os detalhes de implementação específicos da plataforma da sua equipe de TI e desenvolvimento. Concentre-se na arquitetura, ferramentas e recursos da sua plataforma escolhidos ao tomar a decisão de design e fornecimento de diretrizes.

Enquanto os guias de design de nuvem devem levar em consideração alguns dos detalhes técnicos associados a cada componente de infraestrutura, eles não devem ser abranger documentos técnicos ou especificações. Certifique-se de que suas guias abordem todas as instruções de política e informem claramente as decisões de design em formato fácil para a equipe entender e referenciar.

<!-- markdownlint-enable MD033 -->

## <a name="using-the-actionable-governance-journeys"></a>Usar os percursos de governança acionáveis

Se você estiver planejando usar a plataforma do Azure para a adoção da nuvem, o CAF fornece [percursos de governança](../journeys/overview.md) que ilustram a abordagem incremental do modelo de governança do CAF. Esses percursos de narrativa abrangem uma variedade de cenários comuns de adoção, incluindo os riscos de negócios, requisitos de tolerância e instruções de políticas que se tornaram a criação de um produto viável mínimo de governança (MVP). Esses percursos representam uma síntese de experiência de clientes do mundo real do processo de adoção de nuvem no Azure.

Enquanto cada adoção de nuvem tiver metas exclusivas, prioridades e desafios, estes exemplos devem fornecer um bom modelo para converter sua política em orientação. Escolha o cenário mais próximo à sua situação como ponto de partida e molde-o para atender às suas necessidades de política específica.

## <a name="next-steps"></a>Próximas etapas

Com a diretriz de design no local, estabeleça processos de adesão da política para garantir a conformidade de política.

> [!div class="nextstepaction"]
> [Processos de conformidade de política](processes.md)