---
title: 'CAF: Definir as instruções para políticas corporativas'
description: Como você estabelece a política para refletir e atenuar os riscos?
author: BrianBlanchard
ms.date: 01/02/2019
ms.openlocfilehash: 97a2c25fea621e5418e505375eb0e80007ddb0de
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900382"
---
<!---
I understand risk and tolerance, now what do I do?
Define the policy... [aspirational statement to move towards 2/1] If you need help defining policies, each discipline includes references to common business risks and policies to mitigate the risks...
--->

# <a name="defining-corporate-policy-for-cloud-governance"></a>Definição de política corporativa para governança de nuvem

Depois ter analisado os riscos conhecidos e tolerâncias de risco relacionados para a jornada de transformação de nuvem da sua organização, a próxima etapa é estabelecer a política que explicitamente aborda esses riscos e definir as etapas necessárias para atenuá-los sempre que possível.

<!-- markdownlint-disable MD026 -->

## <a name="how-can-corporate-it-policy-become-cloud-ready"></a>Como a política de TI corporativa pode se tornar pronta para a nuvem?

No controle tradicional e governança incremental, a política corporativa cria a definição de trabalho de governança. A maioria das ações de governança de TI busca implementar a tecnologia para monitorar, impor, operar e automatizar essas políticas corporativas. A governança de nuvem é criada em conceitos semelhantes.

![Governança corporativa e disciplinas de governança](../../_images/operational-transformation-govern.png)

*Figura 1. Governança corporativa e disciplinas de governança.*

A imagem acima demonstra as interações entre o risco de negócios, política e conformidade e monitorar e impor para criar uma Estratégia de governança. Seguido por cinco disciplinas de Governança de nuvem para realizar sua estratégia.

A governança de nuvem é o produto de um esforço de adoção contínua ao longo do tempo, como uma transformação verdadeira contínua que não acontece da noite para o dia. A tentativa de entregar a governança de nuvem completa antes de abordar as alterações de política corporativa de chave usando um método rápido agressivo raramente produz os resultados desejados. Em vez disso, é recomendável uma abordagem incremental.

O que é diferente sobre uma estrutura de adoção da nuvem é o ciclo de compras e o impacto do que como esse ciclo pode habilitar o ciclo de transformação autêntica. Como não é um requisito de aquisição de despesas de capital (CapEx) grande, ps engenheiros podem começar o experimento e a adoção mais cedo. Em culturas mais corporativas, a eliminação da barreira de CapEx para a adoção pode levar a loops de comentários mais rigorosos, crescimento orgânico e execução incremental.

A mudança para a adoção da nuvem exige uma mudança na governança. Em muitas organizações, a transformação de política corporativa permite governança aprimorada e taxas mais altas de conformidade por meio de alterações incrementais de política e a aplicação automática dessas alterações, alimentado por recursos definidos recentemente que você configurar com seu provedor de serviços de nuvem.

<!-- markdownlint-enable MD026 -->

## <a name="review-existing-policies"></a>Revisão das políticas existentes

Uma vez que a sua nuvem de implantação se desenvolve e a quantidade de movimentação para o seu acervo de IT aumenta a nuvem, seus riscos e a política associada precisam também será alterados. A governança é um processo contínuo e a política deve ser revisada regularmente com a equipe de TI e stakeholders para garantir que os recursos hospedados na nuvem continuem a manter a conformidade com requisitos e objetivos corporativos. A compreensão dos novos riscos e a tolerância aceitável pode impulsionar uma [revisão de políticas existentes](what-is-a-cloud-policy-review.md) para determinar o nível necessário de governança que apropriado para sua organização.

> [!TIP]
> Se sua organização é regida por conformidade de terceiros, um dos maiores riscos de negócios a considerar pode ser um risco da aderência a [conformidade regulatória](what-is-regulatory-compliance.md). Muitas vezes, esse risco não pode ser atenuado e, em vez disso, pode exigir uma estrita aderência. Certifique-se de entender seus requisitos de conformidade de terceiros antes de iniciar uma revisão da política.

## <a name="create-cloud-policy-statements"></a>Criar instruções de política de nuvem

As políticas de TI baseadas em nuvem estabelecem requisitos, padrões e objetivos que sua equipe de TI e os sistemas automatizados precisarão dar suporte. Decisões de política são o fator principal de seu [design de arquitetura de nuvem](align-governance-journeys.md) e como você irá implementar seus [processos de conformidade de política](processes.md).

As instruções individuais da política de nuvem são diretrizes para abordar os riscos específicos identificados durante o processo de avaliação de riscos. Enquanto essas políticas podem ser integradas em sua documentação da política corporativa mais ampla, instruções de política de nuvem discutidas neste guia CAF tende a ser um resumo mais conciso dos riscos e planos para lidar com eles. Cada definição deve incluir essas duas informações:

- **Risco de negócios**. Um resumo do risco que esta política abordará.
- **Instrução da política**. Uma explicação concisa dos objetivos e requisitos da política.
- **Orientação de design ou técnica**. Recomendações viáveis, especificações ou outras orientações para dar suporte e aplicar esta política em que as equipes de TI e desenvolvedores podem usar ao projetar e criar suas implantações de nuvem.

Se você precisar de ajuda para iniciar as políticas de definição, consulte as [disciplinas de governança](../governance-disciplines.md) apresentadas na visão geral de seção de governança. Os artigos para cada uma dessas disciplinas incluem exemplos dos riscos de negócios comuns encontrados ao mover para a nuvem e políticas de exemplo usadas para atenuar esses riscos (por exemplo, consulte as definições de política de exemplo [da disciplina de Gerenciamento de Custos](../cost-management/policy-statements.md)).

## <a name="incremental-governance-and-integrating-with-existing-policy"></a>Governança incremental e integração com a política existente

Adições planejadas para o seu ambiente de nuvem sempre devem ser verificadas quanto à conformidade com a política existente e política atualizada para a conta para qualquer problema não abordado. Realize também a [revisão regular da política de nuvem](what-is-a-cloud-policy-review.md) para garantir que sua nuvem política esteja atualizada e em sincronia com qualquer nova política corporativa.

A necessidade de integrar a política de nuvem às suas políticas de TI herdadas, depende em grande parte da maturidade dos processos de governança de nuvem e do tamanho do seu acervo de nuvem. Consulte o artigo sobre [governança incremental e política de MVP](overview.md) para uma discussão mais ampla sobre como lidar com a integração de política durante sua transformação de nuvem.

## <a name="next-steps"></a>Próximas etapas

Depois de definir suas políticas, faça um rascunho de um guia de design de arquitetura para fornecer a equipe de TI e desenvolvedores de diretrizes de ações.

> [!div class="nextstepaction"]
> [Rascunho de guia de design de arquitetura](align-governance-journeys.md)
