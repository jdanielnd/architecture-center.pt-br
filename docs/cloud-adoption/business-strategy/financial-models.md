---
title: 'CAF: Criar um modelo financeiro para a transformação de nuvem'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Como criar um modelo financeiro para a transformação de nuvem.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.openlocfilehash: 4fe9b178962bf2cd6a79233278c73085237772f0
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898197"
---
# <a name="create-a-financial-model-for-cloud-transformation"></a>Criar um modelo financeiro para a transformação de nuvem

A criação de um modelo financeiro que representa com precisão o valor comercial total de uma transformação de nuvem pode ser complicado. Modelos financeiros e justificativas de negócios tendem a variar de uma organização para outra. Este artigo estabelece algumas fórmulas e aponta alguns aspectos que não costumam ser percebidos durante a criação de um modelo financeiro.

## <a name="return-on-investment-roi"></a>ROI (retorno sobre o investimento)

ROI (retorno sobre o investimento) geralmente é um critério importante para os diretores ou o conselho de administração. ROI é usado para comparar as diferentes maneiras de se investir recursos de capital limitados. A fórmula do ROI é bastante simples. Os detalhes necessários para criar cada valor para a fórmula podem não ser tão simples. Essencialmente, ROI é o valor de retorno produzido por um investimento inicial. Normalmente, ele é representado como uma porcentagem:

![ROI (retorno sobre o investimento) é igual a (ganho com o investimento - custos do investimento)/custo do investimento](../_images/formula-roi.png)

<!-- markdownlint-disable MD036 -->
*ROI = (ganho com o investimento &minus; investimento inicial) / investimento inicial*
<!-- markdownlint-enable MD036 -->

Nas próximas seções, vamos percorrer os dados necessários para calcular o investimento inicial e o ganho com o investimento (ganhos).

## <a name="calculating-initial-investment"></a>Calculando o investimento inicial

Investimento inicial é as CapEx (despesas de capital) e as OpEx (despesas operacionais) necessárias para concluir uma transformação. A classificação dos custos pode variar dependendo dos modelos de contabilidade e da preferência do diretor financeiro. No entanto, essa categoria inclui coisas como: Serviços profissionais para a transformação, licenças de software que são usadas somente durante a transformação, o custo dos serviços de nuvem durante a transformação e, potencialmente, o custo dos funcionários assalariados durante a transformação.

Some esses custos para criar uma estimativa do investimento inicial.

## <a name="calculating-the-gain-from-investment"></a>Calculando o ganho com o investimento

O ganho com o investimento geralmente requer uma segunda fórmula de cálculo, que é muito específica para as alterações técnicas associadas e os resultados de negócios. Ganhos não são tão simples como calcular a redução nos custos.

Para calcular os ganhos, são necessárias duas variáveis:

![Ganho com o investimento é igual a Deltas de Receita + Deltas de Custo](../_images/formula-gain-from-investment.png)

<!-- markdownlint-disable MD036 -->
*Ganho com o investimento = Deltas de Receita + Deltas de Custo*
<!-- markdownlint-enable MD036 -->

Cada um deles está descrito abaixo.

## <a name="revenue-delta"></a>Delta da receita

O delta da receita deve ser previsto em parceria com o negócio. Depois que os stakeholders chegam a um acordo em relação ao impacto sobre a receita, isso que pode ser usado para melhorar a posição de ganho.

## <a name="cost-deltas"></a>Deltas de custo

Deltas de custo são a quantidade de aumento ou redução que resultará da transformação. Há uma série de variáveis independentes que podem afetar os deltas de custo. Os ganhos baseiam-se principalmente em custos tangíveis, como reduções de despesas de capital, provisões, reduções de custos operacionais e reduções de depreciação. As seções a seguir são exemplos de deltas de custo a serem considerados.

### <a name="depreciation-reductions-or-acceleration"></a>Aceleração ou reduções de depreciação

Para obter orientação sobre depreciação, fale com o diretor financeiro ou com a equipe financeira. O exemplo a seguir serve como uma referência geral sobre o tópico de depreciação.

Quando o capital é investida na aquisição de um ativo, esse investimento poderia ser usado para fins financeiros ou tributários a fim de produzir benefícios contínuos ao longo do tempo de vida esperado do ativo. Algumas empresas veem depreciação como uma vantagem tributária positiva. Outras pessoas a veem como uma despesa contínua e confirmada, semelhante a outras despesas recorrentes atribuídas ao orçamento de TI anual.

Fale com o departamento financeiro para ver se a eliminação da depreciação é possível e se isso seria bom para os deltas de custo.

### <a name="physical-asset-recovery"></a>Recuperação de ativos físicos

Em alguns casos, os ativos desativados podem ser vendidos como uma fonte de receita. Geralmente, essa venda é jogada em redução de custos para manter a simplicidade. No entanto, ela representa realmente um aumento de receita e pode ser tributada como tal. Fale com o departamento financeiro para compreender a viabilidade dessa opção e como levar em conta a receita resultante.

### <a name="operational-cost-reductions"></a>Reduções de custo operacional

As despesas recorrentes necessárias para operar os negócios são geralmente denominadas OpEx (despesas operacionais). OpEx é uma categoria muito ampla. Na maioria dos modelos de contabilidade, isso incluiria o licenciamento de software, as despesas de hospedagem, as contas de energia elétrica, os aluguéis de imóveis, as despesas de resfriamento, as equipes temporárias necessárias para operações, os aluguéis de equipamento, as peças de reposição, os contratos de manutenção e serviços de reparo, os serviços de BC/DR (Recuperação de Desastre/Continuidade dos Negócios) e várias outras despesas que não exigem a aprovação de despesas de capital.

Esta categoria é uma das maiores áreas de ganhos ao se considerar uma Jornada de Transformação Operacional. Não vale gastar tempo colocando todas as hipóteses disso em uma lista exaustiva. Pergunte ao diretor financeiro ou ao departamento financeiro para ter certeza de que todos os custos operacionais foram levados em conta.

### <a name="cost-avoidance"></a>Prevenção de custos

Quando uma OpEx (despesa operacional) é esperada, mas ainda não está em um orçamento aprovado, ela não cabe em uma categoria de redução de custo. Por exemplo, se as licenças do VMWare e da Microsoft precisam ser renegociadas e pagas no próximo ano, elas ainda não estão totalmente qualificadas como custos. Reduções nesses custos esperados seriam tratados como custos operacionais para fins de cálculos de delta do custo. Informalmente, no entanto, eles devem ser referidas como "provisões" até que a negociação e a aprovação do orçamento sejam concluídas.

### <a name="soft-cost-reductions"></a>Reduções de custos variáveis

Em algumas empresas, os custos intangíveis, como reduções da complexidade operacional ou redução da equipe em tempo integral para operar um datacenter, também poderiam ser incluídos. No entanto, pode não ser recomendável incluir os custos intangíveis. A inclusão dos custos intangíveis insere uma suposição não documentada de que a redução nos custos equivalerá a uma economia de custo tangível. Os projetos de tecnologia raramente resultam em uma recuperação real de custos com software.

### <a name="headcount-reductions"></a>Reduções do número de funcionários

A economia de tempo da equipe geralmente é incluída em redução de custos intangíveis. Quando essas economias de tempo equivalem a reduções reais de salários ou de equipe de TI, elas podem ser calculadas separadamente como uma redução do número de funcionários.

Dito isso, as habilidades necessárias no local geralmente equivalem a um conjunto de habilidades semelhante (ou de nível mais alto) necessário na nuvem. Isso significa que as pessoas geralmente não são demitidas após uma migração para a nuvem.

Uma exceção é quando capacidade operacional é fornecida por um provedor de serviços gerenciados (MSP) ou de terceiros. Se os sistemas de TI são gerenciados por um terceiro, os custos de operação podem ser substituídos por uma solução de nuvem nativa ou um MSP de nuvem nativo. Um MSP nativo de nuvem provavelmente opera com mais eficiência e a um custo possivelmente mais baixo. Se esse for o caso, as reduções de custo operacional pertencem aos cálculos de custos tangíveis.

### <a name="capital-expense-reductions-or-avoidance"></a>Reduções de despesas de capital ou provisão

As CapEx (despesas de capita) são um pouco diferentes das despesas operacionais. Em geral, essa categoria é orientada pela expansão do datacenter ou pelos ciclos de atualização. Um exemplo de uma expansão de datacenter seria um novo cluster de alto desempenho hospedar uma solução de Big Data ou um data warehouse, e geralmente cabe em uma categoria de CapEx. Os ciclos de atualização básicos são mais comuns. Algumas empresas têm ciclos de atualização de hardware rígidos, o que significa que ativos são desativados e substituídos em um ciclo regular (normalmente a cada 3, 5 ou 8 anos). Esses ciclos geralmente coincidem com os ciclos de arrendamento do ativo ou do ciclo de vida previsto do equipamento. Quando um ciclo de atualização é concluído, o departamento de TI desembolsa CapEx para adquirir novos equipamentos.

Se um ciclo de atualização é aprovado e está no orçamento, a Transformação de Nuvem pode ajudar a eliminar esse custo. Se um ciclo de atualização é planejado, mas ainda não aprovado, a Transformação de Nuvem pode criar uma provisão de custo de CapEx. Ambos os cenários seriam adicionados ao delta de custo.

## <a name="next-steps"></a>Próximas etapas

Ler alguns exemplos de resultados fiscais no contexto de uma transformação de nuvem.

> [!div class="nextstepaction"]
> [Exemplos de resultados fiscais](./business-outcomes/fiscal-outcomes.md)