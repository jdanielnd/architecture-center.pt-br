---
title: 'CAF: Empresas de grande porte - política corporativa inicial por trás da estratégia de governança'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: Empresas de grande porte - política corporativa inicial por trás da estratégia de governança.
author: BrianBlanchard
ms.openlocfilehash: d3bc31f95229a82cbc2f6ac6e684a87107783c07
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900372"
---
# <a name="large-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a>Empresas de grande porte: Política corporativa inicial por trás da estratégia de governança

A política corporativa a seguir define uma posição inicial de governança que é o ponto de partida para esse percurso. Este artigo define o estágio inicial de riscos, instruções de política inicial e processos antecipados para impor as declarações de política.

> [!NOTE]
>A política corporativa não é um documento técnico, mas conduz muitas decisões técnicas. A governança MVP descrita na [visão geral](./overview.md) deriva, por fim, essa política. Antes de implementar uma governança MVP, sua organização deve desenvolver uma política corporativa com base em seus próprios objetivos e riscos de negócios.

## <a name="cloud-governance-team"></a>Equipe de governança de nuvem

A CIO realizou recentemente uma reunião com a equipe de governança de TI para entender o histórico das políticas de missão crítica e de PII e examinar o impacto de alteração dessas políticas. Ela também discutiu o potencial geral da nuvem para IT e a empresa.

Após a reunião, os dois membros da equipe de governança de TI solicitaram permissão para pesquisar e dar suporte a iniciativas de planejamento de nuvem. Ao reconhecer a necessidade de governança e uma oportunidade para limitar a TI invisível, o Diretor de Governança de TI apoiou essa ideia. Com isso, a equipe de governança de nuvem nasceu. Nos próximos meses, ela herdará a limpeza dos muitos erros cometidos durante a exploração na nuvem de uma perspectiva de governança. Isso lhes dará o apelido de Responsáveis da Nuvem. Nas evoluções posteriores, essa jornada mostrará como suas funções mudam ao longo do tempo.

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a>Indicadores de tolerância

A tolerância a riscos atual é alta e o apetite de investir em governança de nuvem é baixo. Dessa forma, os indicadores de tolerância atuam como um sistema de avisos antecipados para disparar mais investimento de tempo e energia. Se os seguintes indicadores forem observados, você deve evoluir a estratégia de governança.

- Gerenciamento de Custos: A escala de implantação excede 1.000 ativos para a nuvem ou os gastos mensal excedem 100 milhões de dólares por mês.
- Linha de base de identidade: Inclusão de aplicativos com requisitos de autenticação multifator herdada ou de terceiros (MFA).
- Linha de Base de Segurança: Inclusão dos dados protegidos em planos de adoção definida da nuvem.
- Consistência de Recursos: Inclusão de todos os aplicativos críticos em planos de adoção definida da nuvem.

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a>Próximas etapas

Essa política corporativa prepara a equipe de governança de nuvem para implementar a governança MVP, que será a base para a adoção. A próxima etapa é implementar esse MVP.

> [!div class="nextstepaction"]
> [Melhor prática explicada](./best-practice-explained.md)