---
title: 'CAF: Métricas de Gerenciamento de Custos, indicadores e tolerância a risco'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Explicação sobre o Gerenciamento de Custos em relação a governança de nuvem
author: BrianBlanchard
ms.openlocfilehash: 76e6b1b32dd862322f6cafd9aa63c6c4f79f4f5d
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241727"
---
# <a name="cost-management-metrics-indicators-and-risk-tolerance"></a>Métricas de Gerenciamento de Custos, indicadores e tolerância a risco

Este artigo tem como objetivo ajudar a quantificar a tolerância a riscos de negócios no que diz respeito ao Gerenciamento de Custos. Definir métricas e indicadores ajuda a criar um caso de negócios para fazer um investimento no amadurecimento da disciplina de Gerenciamento de Custos.

## <a name="metrics"></a>Métricas

O Gerenciamento de Custos em geral se concentra em métricas relacionadas aos custos. Como parte da análise de risco, você irá querer reunir dados relacionados aos seus gastos atuais e planejados em cargas de trabalho baseadas em nuvem para determinar quanto risco você enfrenta e como o investimento em governança de custos é importante para sua estratégia de adoção da nuvem.

Veja a seguir exemplos de métricas úteis que devem ser coletadas para ajudar a avaliar a tolerância a riscos na disciplina Linha de base de segurança:

- Gastos anuais: Custo anual total para serviços fornecidos por um provedor de nuvem
- Gastos mensais: Custo mensal total para serviços fornecidos por um provedor de nuvem
- Previsão versus a taxa real: A taxa comparando a previsão e os gastos reais (mensal ou anual)
- Proporção do ritmo da taxa de adoção (MOM): A porcentagem do delta em custos de nuvem de mês a mês
- Custo acumulado: Mostra o total de gastos diários acumulados, a partir do início do mês
- Tendências de gastos: Tendência de gasto em relação ao orçamento

## <a name="risk-tolerance-indicators"></a>Indicadores de tolerância de risco

Durante as implantações anteriores, como as primeiras cargas de trabalho de desenvolvimento/teste ou experimentais, o Gerenciamento de Custos é provável que seja de risco relativamente baixo. Conforme mais ativos são implantados, o risco aumenta a tolerância da empresa quanto a risco é provável de recuar. Além disso, conforme mais equipes de adoção da nuvem recebem a capacidade de configurar ou implantar ativos para a nuvem, aumenta o risco e diminui a tolerância. Por outro lado, aumentar uma disciplina de Gerenciamento de Custos levará as pessoas desde a fase de adoção da nuvem para implantar as mais novas tecnologias inovadoras.

Nos estágios iniciais de adoção da nuvem, você trabalhará com seu negócio para determinar uma linha de base de tolerância de risco. Depois de ter uma linha de base, você precisará determinar os critérios que disparam um investimento na disciplina de Gerenciamento de Custos. Esses critérios provavelmente serão diferentes para cada organização.

Depois de ter identificado [riscos de negócios](./business-risks.md), você trabalhará com seu negócio para identificar os parâmetros de comparação que você pode usar para identificar gatilhos que poderiam aumentar potencialmente esses riscos. A seguir estão alguns exemplos de como as métricas, como aquelas mencionadas acima, podem ser comparadas com sua tolerância de risco de linha de base para indicar a necessidade da sua empresa ainda mais a investir no Gerenciamento de Custos.

- Compromisso controlado (mais comum): Uma empresa que tem o compromisso de gasto $X 000, 000 neste ano em um fornecedor de nuvem. Ela precisa de uma disciplina de Gerenciamento de Custos para garantir que os negócios não excedam seus destinos de gastos em mais de 20%, e que eles usarão pelo menos 90% desse compromisso.
- Porcentagem de gatilho: Uma empresa com a nuvem de gastos que está estável para seus sistemas de produção. Se isso mudar por mais que % X, uma disciplina de Gerenciamento de Custos será um bom investimento.
- Gatilho superprovisionado: Uma empresa que acredita que suas soluções implantadas são sobreprovisionadas. O Gerenciamento de Custos é um investimento de prioridade até poder demonstrar alinhamento adequado de utilização de provisionamento e ativo.
- Gatilho de gastos mensais: Uma empresa que gasta mais de $x, 000 por mês é considerado um custo considerável. Se os gastos excederem esse valor em um determinado mês, precisará investir no Gerenciamento de Custos.
- Gatilho de gastos anuais: Uma empresa com um orçamento de TI P e D permite gastos $X, 000 por ano na experimentação de nuvem. Ela pode executar cargas de trabalho de produção na nuvem, mas ainda serão consideradas como soluções experimentais se o orçamento não exceder essa quantidade. Depois que ficar acima, será necessário tratar o orçamento, como um investimento de produção e gerenciar os gastos de perto.
- OpEx adverso (incomum): Como uma empresa, são OpEx muito adverso e precisarão de controles de Gerenciamento de Custos em vigor antes de implantar uma carga de trabalho de desenvolvimento/teste.

## <a name="next-steps"></a>Próximas etapas

Usar o [modelo de Gerenciamento de Nuvem](./template.md), de métricas do documento e de indicadores de tolerância que se alinham ao atual plano de adoção da nuvem.

Usar riscos e tolerância como base e estabelecer um processo para administrar e comunicar a conformidade com a política do Gerenciamento de Custos.

> [!div class="nextstepaction"]
> [Estabelecer processos de conformidade de política](compliance-processes.md)
