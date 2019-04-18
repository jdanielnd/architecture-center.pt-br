---
title: 'CAF: Criação de um caso de negócios de migração na nuvem'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Considerações ao criar uma justificativa comercial para a migração para a nuvem.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.openlocfilehash: a2fdb225fb978eaade39850560ca1adbbb3476e7
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640644"
---
# <a name="build-a-business-justification-for-cloud-migration"></a>Criar uma justificativa de negócios para a migração de nuvem

As migrações para a nuvem podem gerar retorno sobre o investimento (ROI) antecipado de iniciativas de transformação de nuvem. No entanto, o desenvolvimento de uma justificativa comercial clara com custos e retornos relevantes e tangíveis pode ser um processo complexo. Este artigo ajudará você a pensar sobre quais dados são necessários para criar um modelo financeiro que se alinhe com os resultados da migração para a nuvem. Primeiro, vamos eliminar alguns mitos sobre a migração para a nuvem, para que sua organização possa evitar alguns erros comuns.

## <a name="dispelling-cloud-migration-myths"></a>Dissipando mitos sobre a migração para a nuvem

**Mito: a nuvem sempre é mais barata.** É uma crença comum que operar um datacenter na nuvem é sempre mais barato do que localmente. Embora isso possa ser verdade, não é uma verdade absoluta. Em alguns casos, os custos operacionais da nuvem pode ser maiores. Geralmente, isso é causado por administração de custos ruim, desequilíbrio entre as arquiteturas de sistema, duplicação de processos, configurações de sistema incomuns ou aumento nos custos com equipes. Felizmente, muitos desses problemas podem ser reduzidos para criar um ROI antecipado. Seguir as orientações descritas em [Criar a justificativa comercial](#building-the-business-justification) pode ajudar a detectar e evitar esses desequilíbrios. Dissipar outros mitos listados aqui também pode ajudar.

**Mito: tudo deve ir para a nuvem.** Na verdade, há vários fatores comerciais que podem levá-lo a escolher uma solução híbrida. Antes de finalizar o modelo de negócios, é recomendável concluir uma primeira análise quantitativa, conforme descrito nos [artigos de Propriedade Digital](../digital-estate/5-rs-of-rationalization.md). Para obter informações adicionais sobre os fatores quantitativos individuais envolvidos na racionalização, confira [Os 5 Rs da racionalização](../digital-estate/5-rs-of-rationalization.md). As duas abordagens usarão dados de inventário obtidos facilmente e uma breve análise quantitativa para identificar as cargas de trabalho ou aplicativos que poderiam resultar em custos mais altos na nuvem. Essas abordagens também podem identificar dependências ou padrões de tráfego que exigiriam uma solução híbrida.

**Mito: o espelhamento do meu ambiente local me ajudará a economizar dinheiro na nuvem.** Durante o planejamento da Propriedade Digital, não é incomum que os clientes detectem capacidade não usada de mais de 50% do ambiente provisionado. Se os ativos são provisionados na nuvem para corresponder ao provisionamento atual, ficará difícil perceber a economia de custos. Considere reduzir o tamanho dos ativos implantados para se alinhar com os padrões de uso, não com os padrões de provisionamento.

**Mito: os custos com servidores fundamentam os casos comerciais a favor da migração para a nuvem.** Às vezes, isso é verdade. Para algumas empresas, é importante reduzir as despesas de capital contínuas relacionadas aos servidores. No entanto, isso depende de vários fatores. As empresas com um ciclo de atualização de hardware de cinco a oito anos provavelmente não verão retornos rápidos sobre sua migração para a nuvem. As empresas com ciclos de atualização padronizados ou impostos podem zerar o custo rapidamente. Em ambos os casos, outras despesas podem ser gatilhos financeiros que justificam a migração. A seguir estão alguns exemplos de custos que normalmente são ignorados quando a análise é feita somente visualizando os servidores ou as VMs:

- Os custos de software com virtualização, servidores e middleware podem ser altos. Os provedores de nuvem eliminam alguns desses custos. Dois exemplos de um provedor de nuvem que reduz custos com virtualização são os programas [Benefícios Híbridos do Azure](https://azure.microsoft.com/pricing/hybrid-benefit/#services) e [Reservas](https://azure.microsoft.com/reservations/).
- Prejuízo nos negócios devido a interrupções podem rapidamente ultrapassar os custos de hardware ou software. Se o datacenter atual está instável, trabalhe com os negócios a quantificar o efeito de interrupções em termos de custos de oportunidade ou negócios reais.
- Os custos ambientais também podem ser significativos. Para a família americana média, suas casas são o maior investimento e o custo mais alto no seu orçamento. Geralmente, o mesmo é verdadeiro para data centers. Imóveis, instalações e custos com serviços essenciais representam uma parte importante dos custos locais. Quando datacenters são desativados, essas instalações podem ser adaptadas a empresa ou potencialmente a empresa poderia ser liberada dos custos inteiramente.

**Mito: Despesas operacionais (OpEx) são melhores do que despesas de capital (CapEx).** Conforme explicado no artigo sobre [resultados fiscais](business-outcomes/fiscal-outcomes.md), OpEx pode ser uma coisa boa. No entanto, há diversos setores que veem o OpEx como algo negativo. A seguir estão alguns exemplos que poderiam exigir maior integração com os departamentos de contabilidade e comercial em relação à conversa sobre OpEx:

- Quando a empresa vê ativos de capital como um fator para a avaliação de negócios, as reduções de CapEx podem ser um resultado negativo. Embora não seja um padrão universal, esse sentimento geralmente é visto em setores de varejo, manufatura e construção.
- Aumentos de OpEx também podem ser vistos como um resultado negativo em empresas que pertencem a fundos de investimento em participações ou que buscam obter capital.
- Se a empresa tem como principal foco o aumento das margens de vendas ou a redução do custo dos produtos vendidos (COGS), o OpEx seria um resultado negativo.

O OpEx nem sempre é uma coisa ruim. As empresas, de uma forma geral, tendem a ver o OpEx como mais conveniente do que o CapEx. Por exemplo, essa abordagem pode ser bem recebida por empresas que estão tentando melhorar o fluxo de caixa, reduzir os investimentos de capital ou diminuir a posse de ativos.

Antes de fornecer uma justificativa comercial que se concentre em uma conversão de CapEx para OpEx, entenda o que é melhor para os negócios. Os departamentos de contabilidade e compras podem frequentemente ajudar a alinhar melhor a mensagem com os objetivos financeiros.

**Mito: migrar para a nuvem é como apertar um botão.** As migrações são uma transformação técnica manualmente intensa. Ao desenvolver uma justificativa comercial, especialmente justificativas com prazos apertados, considere os seguintes aspectos que podem aumentar o tempo necessário para migrar ativos:

- Limitações de largura de banda: a quantidade de largura de banda entre o datacenter atual e o provedor de nuvem determinará os cronogramas durante a migração.
- Cronogramas de teste de negócios: testar aplicativos com o negócio para garantir a preparação e o desempenho pode demorar. O alinhamento dos usuários avançados com os processos de teste é essencial.
- Cronogramas de execução de migração: a quantidade de tempo e esforço humano necessária para executar a migração pode aumentar os custos e atrasar os cronogramas. A alocação dos funcionários ou parceiros contratados também podem atrasar o processo e devem ser considerados no plano.

Impedimentos técnicos e culturais podem reduzir a adoção da nuvem. Quando o tempo é um aspecto importante da justificativa comercial, a melhor mitigação é o planejamento adequado. Há duas abordagens sugeridas durante o planejamento que podem ajudar a reduzir os riscos nos cronogramas.

- Primeiro, invista tempo e energia em entender as restrições técnicas à adoção. Embora a pressão para migrar rapidamente possa ser alta, é importante levar em conta cronogramas de execução realistas.
- Em segundo lugar, se surgirem impedimentos culturais ou pessoais, eles terão impactos mais graves que as restrições técnicas. A adoção da nuvem cria alterações, o que produz a transformação desejada. Infelizmente, às vezes as pessoas têm medo de mudanças e talvez seja necessário oferecer suporte adicional para que elas sigam o plano. Identifique as pessoas-chave da equipe que estão relutantes e envolva-as desde o início.

Para maximizar a prontidão e a mitigação de riscos no cronograma, prepare os stakeholders executivos alinhando claramente o valor comercial e os resultados comerciais. Ajude-os a entender as mudanças que virão com essa transformação. Seja claro e defina expectativas realistas desde o início. Quando pessoas ou tecnologias atrasarem o processo, será mais fácil contar com o suporte dos executivos.

## <a name="building-the-business-justification"></a>Criar a justificativa comercial

O processo a seguir define uma abordagem para desenvolver a justificativa comercial para migrações para a nuvem. Ao ler este conteúdo, se os cálculos ou termos financeiros exigirem uma explicação adicional, consulte o artigo sobre [Modelos financeiros](financial-models.md) para mais esclarecimentos.

De forma geral, a fórmula para a justificativa comercial é simples. No entanto, os pontos de dados sutis exigidos para preencher a fórmula podem ser difíceis de definir. De forma geral, a justificativa comercial enfoca o retorno sobre o investimento (ROI) associado à alteração técnica proposta. A fórmula genérica para o ROI é:

ROI = (Ganho com o investimento &minus; Investimento inicial) / Investimento inicial

Destrinchar esta fórmula cria uma exibição das fórmulas, específica da migração, que gera cada uma das variáveis de entrada no lado direito da equação. As seções restantes deste artigo oferecem algumas considerações para levar em conta.

![ROI é igual a (Ganho com o investimento - Custo do investimento) / Custo do investimento](../_images/formula-roi.png)

## <a name="migration-specific-initial-investment"></a>Investimento inicial específico da migração

- Provedores de nuvem como o Azure oferecem calculadoras para estimar os investimentos em nuvem. Um exemplo desse tipo de calculadora é a [Calculadora de preços do Azure](https://azure.microsoft.com/pricing).
- Alguns provedores de nuvem também dão suporte a calculadoras de delta do custo. Um exemplo de uma calculadora de delta de custo é a [Calculadora do Custo Total de Propriedade (TCO) do Azure](https://azure.com/tco).
- Para estruturas de custo mais refinadas, considere um exercício de [Planejamento de Propriedade Digital](../digital-estate/overview.md).
- Estime o custo da migração.
- Estime o custo das oportunidades de treinamento esperadas, se houver. O [Microsoft Learn](/learn) poderá ajudar a reduzir esses custos.
- Em algumas empresas, o tempo investido pelos membros da equipe existente pode precisar ser incluído nos custos iniciais. Consulte o departamento financeiro para obter diretrizes.
- Discuta custos adicionais ou indiretos com o departamento financeiro para validação.

## <a name="migration-specific-revenue-deltas"></a>Deltas de receita específicos da migração

Esse aspecto muitas vezes é ignorado durante a criação de uma justificativa comercial para a migração. Em algumas áreas, a nuvem pode cortar custos. No entanto, o objetivo final de qualquer transformação é produzir resultados melhores ao longo do tempo. Considere o impacto mais à frente para entender o aumento da receita a longo prazo. Quais novas tecnologias estará disponíveis para os negócios após a migração não pode ser usados hoje em dia? Quais projetos ou objetivos de negócios estão vetados pela dependência de tecnologias herdadas? Quais programas estão em espera, esperando altos custos de tecnologia em despesas de capital?

Após considerar as oportunidades viabilizadas pela nuvem, trabalhe com a empresa para calcular o aumento de receita que poderia ser atingido com essas oportunidades.

## <a name="migration-specific-cost-deltas"></a>Deltas de custo específicos da migração

Calcule todas as alterações nos custos que resultarão da migração proposta. Confira [Modelos financeiros](financial-models.md) para obter detalhes sobre os diferentes tipos de deltas de custo. Os provedores de nuvem geralmente fornecem ferramentas para calcular delta de custo. Um exemplo de uma calculadora de delta de custo é a [Calculadora do Custo Total de Propriedade (TCO) do Azure](https://azure.com/tco).

Outros exemplos de custos que podem ser reduzidos em uma migração na nuvem:

- Encerramento do Datacenter ou redução (custos ambientais)
- Redução no consumo de energia (custos ambientais)
- Encerramento de rack (recuperação de ativos físicos)
- Prevenção de atualização de hardware (redução de custo)
- Impedimento de renovação de software (redução de custo ou de redução de custos operacionais)
- Consolidação de fornecedor (redução de custos operacionais e redução de custos com software potencial)

## <a name="when-roi-results-are-surprising"></a>Quando os resultados do ROI são surpreendentes

Se o ROI da migração na nuvem não corresponder às expectativas, ele pode ser valioso para rever os mitos comuns listados no início deste artigo.

No entanto, é importante entender que um resultado de economia de custo nem sempre é possível. Há aplicativos que custam mais para funcionar na nuvem do que localmente. Esses aplicativos podem afetar significativamente os resultados em uma análise.

Quando o ROI estiver abaixo de 20%, considere um exercício de [Planejamento de Propriedade Digital](../digital-estate/overview.md), com bastante atenção à [racionalização](../digital-estate/rationalize.md). Durante a análise quantitativa, execute uma revisão de cada aplicativo para encontrar cargas de trabalho que distorcem os resultados. Pode ser aconselhável remover essas cargas de trabalho do plano. Se houver dados de uso disponíveis, considere reduzir o tamanho das VMs para corresponder ao uso.

Se o ROI ainda estiver desalinhado, procure a ajuda de seu representante de vendas da Microsoft ou [entre em contato com um parceiro experiente](https://azure.microsoft.com/migration/support).

## <a name="next-steps"></a>Próximos passos

> [!div class="nextstepaction"]
> [Criar um modelo financeiro para a transformação de nuvem](./financial-models.md)