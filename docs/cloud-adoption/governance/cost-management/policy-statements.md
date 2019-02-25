---
title: 'CAF: Instruções de política de amostra de Gerenciamento de Custos'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 1/4/2019
description: Instruções de política de amostra de Gerenciamento de Custos
author: BrianBlanchard
ms.openlocfilehash: 1c8b94ae5285fa26cdf9760892beaced2487af8b
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900384"
---
# <a name="cost-management-sample-policy-statements"></a>Instruções de política de amostra de Gerenciamento de Custos

As instruções individuais da política de nuvem são diretrizes para abordar os riscos específicos identificados durante o processo de avaliação de riscos. Essas instruções oferecem um resumo conciso dos riscos e planos com os quais lidar. Cada definição de instrução deve incluir essas informações:

- Risco comercial - um resumo do risco que essa política irá abordar.
- Instrução da política - Uma explicação clara resumida dos requisitos da política.
- Opção de design - recomendações viáveis, especificações ou outras diretrizes que as equipes de TI e os desenvolvedores possam usar ao implementar a política.

As seguintes instruções de política de exemplo abordam um número de riscos comuns de negócios relacionados à configuração de endereço e são fornecidos como exemplos para fazer referência ao rascunhar instruções de política reais para atender a necessidades da sua organização. Esses exemplos não devem ser proscritivos e há potencialmente várias opções de política para lidar com qualquer risco único identificado. Trabalhe em sintonia com as equipes de TI para identificar as melhores soluções de política para os seus riscos relacionados ao custo particular.  

## <a name="future-proofing"></a>Durabilidade

**Risco de negócios**. Critérios atuais que não garantem um investimento em uma disciplina de Gerenciamento de Custos da equipe de governança. No entanto, você antecipar o investimento no futuro.

**Instrução da política**. Você deve associar todos os ativos implantados na nuvem com uma unidade de cobrança, aplicativo/carga de trabalho e atendem aos padrões de nomenclatura. Esta política garantirá que os esforços do Gerenciamento de Custos futuros entrarão em vigor.

**Opções de design**. Para obter informações sobre como estabelecer uma base à prova de obsolescência, consulte as discussões relacionadas à criação de uma governança MVP nas [guias de design acionáveis](../journeys/overview.md) incluídas como parte das diretrizes do CAF.

## <a name="budget-overruns"></a>Excedentes de orçamento

**Risco de negócios:** A implantação por autoatendimento cria um risco de excesso de gastos.

**Instrução da política:** Qualquer implantação de nuvem deve ser alocada para uma unidade de cobrança com orçamento aprovado e um mecanismo para limites orçamentários.

**Opções de design:** No Azure, o orçamento pode ser controlado com o [Gerenciamento de Custos do Azure](/azure/cost-management/manage-budgets)

## <a name="underutilization"></a>Subutilização

**Risco de negócios:** A empresa tem pré-pago por serviços de nuvem ou fez um compromisso anual gastar uma quantidade específica. Há um risco que o acordado na quantidade não será usado, resultando em um investimento perdido.

**Instrução da política:** Cada unidade de cobrança com um orçamento de nuvem alocado atenderá anualmente para definir os orçamentos trimestralmente para ajustar os orçamentos e mensal de alocar tempo para revisar os gastos reais versus planejados. Discuta desvios maiores que 20% com o líder da unidade de cobrança mensal. Para fins de acompanhamento, atribua todos os ativos para uma unidade de cobrança.

**Opções de design:**

- No Azure, gastos planejados versus reais podem ser gerenciado através do [Gerenciamento de Custos do Azure](/azure/cost-management/quick-acm-cost-analysis)
- Há várias opções para agrupar recursos por unidade de cobrança. No Azure, um [modelo de consistência do recurso](../../decision-guides/resource-consistency/overview.md) deve ser escolhido em conjunto com a equipe de governança e aplicado a todos os ativos.

## <a name="overprovisioned-assets"></a>Ativos superprovisionados

**Risco de negócios:** Em data centers tradicionais locais, é uma prática comum de implantar ativos com capacidade extra, planejamento para crescimento distante no futuro. A nuvem pode dimensionar mais rapidamente do que o equipamento tradicional. Ativos na nuvem também são cobrados com base na capacidade técnica. Há um risco da prática do local antigo artificialmente aumentando os gastos com a nuvem.

**Instrução da política:** Qualquer ativo implantado na nuvem deve ser registrado em um programa que pode monitorar a utilização e relatar qualquer capacidade mais de 50% de utilização. Qualquer ativo implantado na nuvem deve ser agrupado ou marcado de uma maneira lógica, portanto, os membros da equipe governança podem envolver o proprietário da carga de trabalho em relação a qualquer otimização de ativos sobre provisionados.

**Opções de design:**

- No Azure, o [Assistente do Azure](/azure/advisor/advisor-cost-recommendations) pode fornecer recomendações de otimização.
- Há várias opções para agrupar recursos por unidade de cobrança. No Azure, um [modelo de consistência do recurso](../../decision-guides/resource-consistency/overview.md) deve ser escolhido em conjunto com a equipe de governança e aplicado a todos os ativos.

## <a name="overoptimization"></a>Otimização excessiva

**Risco de negócios:** A partir do Gerenciamento de Custos, na verdade, pode criar novos riscos. A otimização dos gastos é o inverso para o desempenho do sistema. Ao reduzir os custos, há um risco de sobrecarregar gastos e produzindo as experiências de usuário insatisfatória.

**Instrução da política:** Qualquer ativo que afeta diretamente as experiências do cliente deve ser identificado por meio do agrupamento ou marcação. Antes de otimizar qualquer ativo que afeta a experiência do cliente, a equipe de governança de nuvem deve ajustar a otimização com base nas tendências de utilização nada menos do que 90 dias. Documente quaisquer intermitências sazonais ou orientadas por eventos consideradas ao otimizar ativos.

**Opções de design:**

- No Azure, [os recursos de insights do Azure Monitor](/azure/azure-monitor/insights/vminsights-performance) podem ajudar com a análise de utilização do sistema.
- Há várias opções para agrupar e marcar recursos baseados nas funções. No Azure, escolha um [modelo de consistência do recurso](../../decision-guides/resource-consistency/overview.md) em conjunto com a equipe de governança e aplicado a todos os ativos.

## <a name="next-steps"></a>Próximas etapas

Use os exemplos mencionados neste artigo como um ponto de partida para desenvolver políticas que abordam os riscos de negócios específicos que se alinham aos seus planos de adoção de nuvem.

Para começar a desenvolver suas próprias instruções de política personalizadas relacionadas ao Gerenciamento de Custos, realize o download do [Modelo de Gerenciamento de Custos](template.md).

Para acelerar a adoção desta disciplina, escolha o [Percurso de Governança Acionável](../journeys/overview.md) que mais se alinha ao seu ambiente. Em seguida, modifique o design para incorporar suas decisões específicas de política corporativa.

> [!div class="nextstepaction"]
> [Percursos de governança acionáveis](../journeys/overview.md)
