---
title: 'CAF: Monitorar e impor adesão à política'
description: Como garantir a conformidade com as políticas estabelecidas?
author: BrianBlanchard
ms.date: 1/4/2019
ms.openlocfilehash: 9066f33c8baa183476a9632e82d6eb960d03752c
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900203"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-processes-can-help-ensure-policy-adherence"></a>Quais processos podem ajudar a garantir a adesão à política?

<!---
I've defined policies, I've provided an architecture guide. Now how do I monitor adherence to policy? If there is a violation, how do I enforce the policy?
--->

Após estabelecer as declarações da política de nuvem e elaborar um guia de planejamento, será necessário criar uma estratégia para garantir que a implantação na nuvem permaneça em conformidade com os requisitos da política. Essa estratégia deverá abranger os processos de análise e comunicação contínua da equipe de Governança de Nuvem, estabelecer critérios para quando as violações da política exigirem ação e definir requisitos para sistemas de monitoramento e conformidade automatizados que detectarão violações e acionarão ações corretivas.

Consulte as seções sobre política corporativa [Percursos de Governança Acionáveis](../journeys/overview.md) para obter exemplos de como o processo de adesão à política se enquadra em um plano de governança de nuvem.

## <a name="prioritize-policy-adherence-processes"></a>Priorizar processos de adesão à política

Quanto investimento em desenvolvimento de processos é necessário para apoiar os objetivos da política? Dependendo do tamanho e da maturidade da implantação de nuvem, o esforço necessário para estabelecer processos que assegurem a conformidade e os custos associados a esse esforço podem variar amplamente.

Para pequenas implantações que consistem em recursos de desenvolvimento e teste, os requisitos da política podem ser simples e requerem direcionamento de poucos recursos dedicados. Por outro lado, uma implantação de nuvem crítica consolidada com necessidades de segurança e desempenho de alta prioridade pode exigir uma equipe de funcionários, processos internos abrangentes e ferramentas de monitoramento personalizadas para atender às metas da política.

Como primeiro etapa para definir a estratégia de adesão à política, avalie como os processos abordados abaixo podem reforçar os requisitos da política. Determine quanto esforço é viável para investir nesses processos e, em seguida, use essas informações para estabelecer planos orçamentários e de pessoal realistas para atender a essas necessidades.

## <a name="establish-cloud-governance-team-processes"></a>Estabelecer processos de equipe de Governança de Nuvem

Antes de definir os gatilhos para correção de conformidade com a política, será necessário estabelecer os processos globais que a equipe usará e como as informações serão compartilhadas e escalonadas entre a equipe de TI e a equipe de Governança de Nuvem.

### <a name="assign-cloud-governance-team-members"></a>Atribuir membros de equipe de Governança de Nuvem

Quem fornecerá as diretrizes contínuas sobre conformidade com a política e tratará dos problemas relacionados à política que surgirem durante a implantação e operação dos ativos de nuvem? O tamanho e a composição da equipe de Governança de Nuvem dependerão da complexidade dos requisitos da política e das prioridades orçamentárias e de pessoal associadas à conformidade com a política.

Escolha membros da equipe com experiência nas áreas abrangidas pelas declarações da política definidas. Para implantações de teste iniciais, isso poderá ser limitado a alguns administradores de sistema responsáveis por estabelecer os princípios básicos de governança. Na medida em que as implantações amadurecerem e as políticas se tornarem mais complexas e mais integradas com os requisitos de política corporativa mais abrangentes, será necessário alterar a equipe de Governança de Nuvem para dar suporte aos requisitos da política cada vez mais complexos.

Conforme os processos de governança amadurecerem, revise regularmente a associação da equipe de diretrizes de nuvem para assegurar que é possível atender corretamente aos requisitos da política mais recentes. Identifique os membros da equipe de TI com experiência relevante ou interesse em áreas específicas de governança e inclua-os nas equipes de forma permanente ou ad hoc, conforme necessário.

### <a name="reviews-and-policy-iteration"></a>Análises e política de iteração

À medida que recursos adicionais são implantados, a equipe de Governança de Nuvem deverá garantir que novas cargas de trabalho ou ativos estejam em conformidade com os requisitos da política. Planeje reunir-se com as equipes responsáveis pela implantação de novos recursos para discutir sobre o alinhamento dos guias de planejamento.

Na medida em que a implantação global cresce, avalie regularmente novos riscos potenciais e atualize as declarações da política e os guias de planejamento, conforme necessário. Agende periodicamente ciclos de análise de cada uma das cinco disciplinas de governança para garantir que a política esteja atualizada e sendo cumprida.

### <a name="education"></a>Educação

A conformidade com a política exige que a equipe de TI e os desenvolvedores reconheçam os requisitos da política que afetam suas áreas de responsabilidade. Planeje dedicar recursos para documentar decisões e requisitos e instruir todas as equipes relevantes sobre os guias de planejamento que dão suporte aos requisitos da política.

Conforme a política for alterada, atualize regularmente a documentação e os materiais de treinamento e garanta que os esforços educacionais comuniquem os requisitos atualizados e as diretrizes para a equipe de TI relevante.  

### <a name="establish-escalation-paths"></a>Estabelecer caminhos de escalonamento

Se um recurso tornar-se não conforme, quem será notificado? Se a equipe de TI detectar um problema de conformidade com a política, a quem contatará? Certifique-se de que o processo de escalonamento da equipe de Governança de Nuvem esteja claramente definido. Garanta que esses canais de comunicação sejam mantidos atualizados para refletir as alterações na equipe e na organização.

## <a name="violation-triggers-and-actions"></a>Gatilhos e ações de violação

Após definir a equipe de Governança de Nuvem e os respectivos processos, será necessário definir explicitamente o que qualifica-se como violações de conformidade que disparam ações e quais são essas ações.

### <a name="define-triggers"></a>Definir gatilhos

Para cada uma das declarações da política, analise os requisitos para determinar o que constitui uma violação da política. Gere os gatilhos usando as informações que você já estabeleceu como parte do processo de definição da política.

* Tolerância de risco - Crie gatilhos de violação baseado nas métricas e indicadores de risco que foram estabelecidos como parte da [análise de tolerância de risco](risk-tolerance.md).
* Requisitos da política definidos - As declarações da política podem fornecer SLA (Contrato de Nível de Serviço), BRCD (continuidade dos negócios e recuperação de desastres) ou requisitos de desempenho que devem ser utilizados como base para gatilhos de conformidade.

### <a name="define-actions"></a>Definir ações

Cada gatilho de violação deve ter uma ação correspondente. As ações disparadas deverão sempre notificar uma equipe de TI apropriada ou um membro da equipe de Governança de Nuvem quando ocorrer uma violação. Essa notificação poderá resultar em uma análise manual do problema de conformidade ou iniciar um processo de correção pré-estabelecido, dependendo do tipo e da gravidade da violação detectada.

Alguns exemplos de gatilhos e ações de violação:

| Disciplina de Governança de Nuvem | Gatilho de exemplo | Ação de exemplo |
|-----------------------------|----------------|---------------|
| Gerenciamento de Custos | Os gastos mensais com a nuvem são mais de 20% superior ao esperado. | Notifique o líder da unidade de cobrança que iniciará uma análise do uso de recursos. |
| Linha de base de segurança | Detectar atividade suspeita de logon de usuário. | Notifique a equipe de segurança de TI e desabilite a conta do usuário suspeita. |
| Consistência de recursos | A utilização da CPU para a carga de trabalho é maior que 90%. | Notifique a equipe de operações de TI e dimensione recursos adicionais para lidar com a carga. |

## <a name="monitoring-and-compliance-automation"></a>Monitoramento e automação de conformidade

Após definir os gatilhos de violação de conformidade e as ações, você poderá iniciar o planejamento de como melhor utilizar as ferramentas de relatório e registro em log e outros recursos da plataforma de nuvem para ajudar a automatizar a estratégia de monitoramento e conformidade da política.

Consulte o CAF [guia de decisão de relatório e registro em log](../../decision-guides/log-and-report/overview.md) para obter as diretrizes sobre como escolher o melhor padrão de monitoramento para a implantação.
