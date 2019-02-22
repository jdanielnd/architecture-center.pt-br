---
title: 'CAF: Melhoria da disciplina de Consistência de recursos'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Melhoria da disciplina de Consistência de recursos
author: BrianBlanchard
ms.openlocfilehash: bc81b894d46266c52291c53dba5532ab2ab6b860
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900207"
---
# <a name="resource-consistency-discipline-improvement"></a>Melhoria da disciplina de Consistência de recursos

A disciplina de Consistência de recursos se concentra em maneiras de estabelecer as políticas relacionadas ao gerenciamento operacional de um ambiente, aplicativo ou carga de trabalho. Nas cinco disciplinas de governança de nuvem, a Consistência de recursos inclui o monitoramento de aplicativos, a carga de trabalho e o desempenho de ativos. Ela também inclui as tarefas necessárias para atender às demandas de dimensionamento, corrigir violações de desempenho de SLA (Contrato de Nível de Serviço) e evitar proativamente as violações de SLA por meio da correção automatizada.

Este artigo descreve algumas tarefas possíveis que sua empresa pode realizar para desenvolver melhor e maturar a disciplina de Consistência de recursos. Essas tarefas podem ser divididas entre as fases de planejamento, criação, adoção e operacional para a implementação de uma solução de nuvem, que depois são iteradas para permitir o desenvolvimento de uma [abordagem incremental para a governança de nuvem](../journeys/overview.md#an-incremental-approach-to-cloud-governance).

![Quatro fases de adoção](../../_images/adoption-phases.png)

*Figura 1. Fases de adoção da abordagem incremental da governança de nuvem.*

É impossível para qualquer pessoa documentar os requisitos a serem levados em consideração de todas as empresas. Portanto, este artigo descreve as atividades de exemplo potenciais e mínimas sugeridas para cada fase do processo de amadurecimento da governança. O objetivo inicial dessas atividades é ajudar a criar um [MVP de política](../journeys/overview.md#an-incremental-approach-to-cloud-governance) e estabelecer uma estrutura para a evolução da política incremental. Sua equipe de governança de nuvem precisará decidir o quanto investir nessas atividades para melhorar as funcionalidades de governança da sua consistência de recursos.

> [!CAUTION]
> Nenhuma das atividades mínimas ou potenciais descritas neste artigo está alinhada a políticas corporativas específicas ou a requisitos de conformidade a terceiros. Essas diretrizes foram projetadas para ajudar a facilitar as conversas que levarão ao alinhamento de ambos os requisitos com um modelo de governança de nuvem.

## <a name="planning-and-readiness"></a>Planejamento e preparação

Esta fase de maturidade de governança une os resultados de negócios e as estratégias realizáveis. Durante esse processo, a equipe de liderança define métricas específicas, mapeia essas métricas ao estado digital e começa a planejar o esforço global de migração.

**Atividades mínimas sugeridas:**

* Avaliar suas opções de [Cadeia de ferramentas da Consistência de recursos](toolchain.md).
* Entender os requisitos de licenciamento para sua estratégia de nuvem.
* Desenvolver um documento com o esboço das Diretrizes de arquitetura e distribuí-lo aos principais stakeholders.
* Familiar-se com o gerenciador de recursos usado para implantar, gerenciar e monitorar todos os recursos da sua solução como um grupo.
* Treinar e envolver as pessoas e equipes afetadas pelo desenvolvimento das diretrizes de arquitetura.
* Adicionar as tarefas de implantação de recurso priorizadas à sua lista de pendências de migração.

**Atividades potenciais:**

* Trabalhar com os stakeholders de negócios e/ou sua equipe de estratégia de nuvem para entender a abordagem desejada de contabilidade da nuvem e as práticas recomendadas de contabilização de custos em suas unidades de negócios e na organização como um todo.
* Definir seus requisitos de [monitoramento e imposição de políticas](compliance-processes.md).
* Examine o valor de negócio e o custo da interrupção para definir a política de correção e os requisitos de SLA.
* Determinar se você implantará uma estratégia de governança de [carga de trabalho simples](./governance-simple-workload.md) ou de [várias equipes](./governance-multiple-teams.md) para seus recursos.
* Determinar requisitos de escalabilidade para suas cargas de trabalho planejadas.

## <a name="build-and-pre-deployment"></a>Compilação e pré-implantação

Diversos pré-requisitos técnicos e não técnicos são necessários para migrar um ambiente com sucesso. Esse processo se concentra nas decisões, na preparação e na infraestrutura central que ocorrem após uma migração.

**Atividades mínimas sugeridas:**

* Implementar sua [cadeia de ferramentas da Consistência de recursos](toolchain.md) fazendo o lançamento em uma fase de pré-implantação.
* Atualizar o documento com o esboço das Diretrizes de arquitetura e distribuí-lo aos principais stakeholders.
* Implementar as tarefas de implantação de recurso à sua lista de pendências de migração priorizadas.
* Desenvolver materiais e documentações educativas, comunicações de conscientização, incentivos e outros programas para ajudar a conduzir a adoção de usuários.

**Atividades potenciais:**

* Decidir sobre uma [estratégia de design de assinatura](../../decision-guides/subscriptions/overview.md), escolher os padrões de assinatura que melhor atendem às necessidades da sua organização e carga de trabalho.
* Usar uma estratégia de [Consistência de recursos](../../decision-guides/resource-consistency/overview.md) para impor diretrizes de arquitetura ao longo do tempo.
* Implementar [a nomenclatura de recursos e os padrões de marcação](../../decision-guides/resource-tagging/overview.md) em seus recursos para atender aos seus requisitos organizacionais e de contabilidade.
* Para criar uma governança proativa e pontual, use modelos de implantação e automação para impor configurações comuns e uma estrutura de agrupamento consistente durante a implantação de recursos e grupos de recursos.
* Estabelecer um modelo de permissões com menos privilégios, no qual, por padrão, os usuários não têm permissões.
* Determinar quem em sua organização possui cada carga de trabalho e a conta e quem precisará acessar para manter ou modificar esses recursos. Definir as funções de nuvem e responsabilidades que correspondem a essas necessidades e usar essas funções como a base para o controle de acesso.
* Definir dependência entre recursos.
* Implementar o dimensionamento de recursos automatizado para atender aos requisitos definidos no estágio de planejamento.
* Conduzir o desempenho de acesso para medir a qualidade dos serviços recebidos.
* Cogitar implantar a [política](/azure/governance/policy/overview) para gerenciar a imposição de SLA usando definições de configuração e regras de criação de recursos.

## <a name="adopt-and-migrate"></a>Adotar e migrar

A migração é um processo incremental que se concentra no movimento, teste e adoção de aplicativos ou cargas de trabalho em um estado digital existente.

**Atividades mínimas sugeridas:**

* Migrar sua [cadeia de ferramentas da Consistência de recursos](toolchain.md) da pré-implantação para a produção.
* Atualizar o documento com o esboço das Diretrizes de arquitetura e distribuí-lo aos principais stakeholders.
* Desenvolver materiais e documentações educativas, comunicações de conscientização, incentivos e outros programas para ajudar a conduzir a adoção de usuários.
* Migrar quaisquer scripts ou ferramentas de correção automatizada existentes para dar suporte a requisitos de SLA definidos.

**Atividades potenciais:**

* Concluir e testar o monitoramento e os dados de relatório com sua solução local, de gateway de nuvem ou híbrida escolhida.
* Determinar se precisam ser feitas alterações no SLA ou no gerenciamento de política para recursos.
* Melhorar as tarefas de operações por meio da implementação de recursos de consulta para localizar eficientemente os recursos entre seu estado de nuvem.
* Alinhar os recursos conforme as necessidades de negócios em mudança e os recursos de governança.
* Garantir que suas máquinas virtuais, redes virtuais e contas de armazenamento refletem as necessidades reais de acesso aos recursos durante cada versão e ajustar conforme o necessário.
* Verificar se o dimensionamento automático de recursos atende aos requisitos de acesso.
* Examinar o acesso de usuários a recursos, grupos de recursos e assinaturas do Azure e ajustar os controles de acesso conforme o necessário.
* Monitorar as alterações nos planos de acesso a recursos e validar com os stakeholders se forem necessárias aprovações adicionais.
* Atualizar as alterações no documento de Diretrizes de arquitetura para refletir os custos reais.
* Determinar se sua organização requer um alinhamento financeiro mais claro aos demonstrativos de perdas e ganhos para unidades de negócios.
* Para organizações globais, implementar seus requisitos de conformidade ao SLA ou de soberania.
* Para a agregação na nuvem, implantar uma solução de gateway em um provedor de nuvem.
* Para ferramentas que não permitem as opções híbrida ou de gateway, unir firmemente o monitoramento a uma ferramenta de monitoramento operacional.

## <a name="operate-and-post-implementation"></a>Operação e pós-implementação

Depois que a transformação for concluída, a governança e as operações devem ser mantidas para o ciclo de vida natural de um aplicativo ou uma carga de trabalho. Essa fase de maturidade de governança se concentra em atividades que normalmente ocorrem depois que a solução é implementada e que o ciclo de transformação começa a se estabilizar.

**Atividades mínimas sugeridas:**

* Personalizar sua [cadeia de ferramentas de Consistência de recursos](toolchain.md) com base nas atualizações das necessidades em mudança de Gerenciamento de Custos da sua organização.
* Cogitar automatizar quaisquer notificações e relatórios para refletir o uso real de recursos.
* Refinar as Diretrizes de arquitetura para orientar processos futuros de adoção.
* Treinar as equipes afetadas periodicamente para garantir a conformidade contínua às diretrizes de arquitetura.

**Atividades potenciais:**

* Ajustar planos trimestrais para refletir as alterações aos recursos reais.
* Aplicar e impor automaticamente os requisitos de governança durante futuras implantações.
* Avaliar os recursos subutilizados e determinar se vale a pena mantê-los.
* Detectar desalinhamentos e anomalias entre os usos de recursos reais e planejados.
* Auxiliar as equipes de adoção da nuvem e a equipe de estratégia de nuvem a entender e resolver essas anomalias.
* Determinar se precisam ser feitas alterações na Consistência de recursos para cobrança e SLAs.
* Avaliar as ferramentas de registro de logs e monitoramento para determinar se as soluções locais, de gateways de nuvem ou híbridas precisam de ajuste.
* Para unidades de negócios e grupos geograficamente distribuídos, determinar se sua organização deve considerar o uso de recursos adicionais de gerenciamento de nuvem (por exemplo [grupos de gerenciamento do Azure](/azure/governance/management-groups/)) para aplicar melhor a política centralizada e atender aos requisitos de SLA.

## <a name="next-steps"></a>Próximas etapas

Agora que você entende o conceito de governança de recursos de nuvem, saiba mais sobre [como o acesso aos recursos é gerenciado](azure-resource-access.md) no Azure em preparação para aprender a criar um modelo de governança para uma [carga de trabalho simples](governance-simple-workload.md) ou [múltiplas equipes](governance-multiple-teams.md).

> [!div class="nextstepaction"]
> [Saiba mais sobre o acesso aos recursos do Azure](azure-resource-access.md)
> [Saiba mais sobre SLAs do Azure](https://azure.microsoft.com/support/legal/sla/)
> [Saiba mais sobre como fazer registros em log, relatórios e monitoramentos](../../decision-guides/log-and-report/overview.md)
