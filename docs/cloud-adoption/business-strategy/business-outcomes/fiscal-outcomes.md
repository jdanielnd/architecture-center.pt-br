---
title: 'CAF: Exemplos de resultados fiscais'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Exemplos de resultados fiscais
author: BrianBlanchard
ms.date: 10/10/2018
ms.topic: guide
ms.openlocfilehash: 5b28e22fca294bd1311d29ee5f70ca45e70fae3e
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640219"
---
# <a name="examples-of-fiscal-outcomes"></a>Exemplos de resultados fiscais

No nível mais alto, as conversas fiscais são compostas por três conceitos básicos:

* Receita: Mais dinheiro entrará para a empresa como resultado das vendas de mercadorias ou serviços.
* Custo: Menos dinheiro será gasto na criação, marketing, vendas ou entrega de mercadorias ou serviços.
* Lucro: Embora seja raro, algumas transformações podem aumentar a receita e reduzir os custos. Esse é um resultado de lucro.

O restante deste artigo explicará esses resultados fiscais no contexto de uma transformação de nuvem.

> [!NOTE]
> Os exemplos a seguir são hipotéticos e não devem ser vistos como uma garantia de retorno ao adotar qualquer estratégia de nuvem.

## <a name="revenue-outcomes"></a>Resultados da receita

### <a name="new-revenue-streams"></a>Novos fluxos de receita

A nuvem permite oportunidades de oferecer novos produtos aos clientes ou oferecer produtos existentes de uma nova maneira. Novos Fluxos de Receita são inovadores, empresariais e empolgantes para muitas pessoas do mundo dos negócios. Os novos fluxos de receita também estão sujeitos a falhas, e são vistos em muitas empresas como algo de alto risco. Quando são propostos pelo departamento de TI, a probabilidade de recusa é muito alta. Para adicionar credibilidade a esses resultados, firme uma parceria com um líder de negócios comprovadamente inovador. A validação do fluxo de receita no início do processo ajudará a evitar os obstáculos.

* Exemplo: Uma empresa vende livros há mais de cem anos. Um funcionário da empresa percebe que o conteúdo pode ser entregue eletronicamente e cria um dispositivo que pode ser vendido na livraria, o qual permite que os mesmos livros sejam baixados diretamente, gerando $X em novas vendas de livros.

### <a name="revenue-increases"></a>A receita aumenta

Com escala global e alcance digital, a nuvem permite que as empresas aumentem a receita de fluxos de receita existentes. Muitas vezes, esse tipo de resultado acontece graças a um alinhamento com a liderança de marketing ou de vendas.

* Exemplo: Uma empresa que vende widgets poderia vender mais widgets se o pessoal de vendas tivesse a capacidade de acessar com segurança o catálogo digital e os níveis de estoque da empresa. Infelizmente, os dados estão apenas no sistema ERP da empresa, que só pode ser acessado por meio de um dispositivo conectado à rede. A criação de uma fachada do serviço para interação com o ERP, expondo o catálogo e os níveis de estoque não confidenciais a um aplicativo na nuvem, permitiria que o pessoal de vendas acessasse os dados necessários durante uma reunião com um cliente. Estender o AD usando o Azure AD e integrar o acesso baseado em função ao aplicativo permitiria que a empresa garantisse a segurança dos dados. Esse projeto simples pode afetar a receita de uma linha de produtos existente em X%.

### <a name="profit-increases"></a>Aumento do lucro

Raramente, um único esforço aumenta a receita e reduz os custos simultaneamente. No entanto, quando isso acontece, alinhe o demonstrativo do resultado de um ou mais resultados de receita com um ou mais resultados de custo para comunicar o resultado desejado.

## <a name="cost-outcomes"></a>Resultados de custo

### <a name="cost-reduction"></a>Redução de custo

Nuvem de computação pode reduzir as despesas de capital (CapEx) relacionadas a compra de hardware e software, a configuração de data centers, a execução de datacenters locais, etc... Os racks de servidores, a eletricidade permanente para proporcionar energia e resfriamento e especialistas de TI para gerenciar a infraestrutura aumentam rapidamente. Desligando um data center pode reduzir CapEx compromissos. Isso é às vezes denominado "tirando proveito de negócios datacenter". Redução de custos geralmente é medida em dólares no orçamento atual, que pode abranger um a cinco anos, dependendo de como o CFO gerencia Finanças.

* Exemplo 1: Contas de datacenter da empresa para um grande percentual do orçamento de TI anual. IT opta por executar uma transformação operacionais um migra os ativos naquele Datacenter para infraestrutura como uma solução de serviço (IaaS), criando uma redução de custos de três anos.
* Exemplo 2: Uma empresa holding adquiriu recentemente uma nova empresa. A aquisição, os termos ditam que a nova entidade seja removido do atuais data centers dentro de seis meses. Caso contrário, a holding receberia uma multa de $1 milhão/mês. A movimentação dos ativos digitais para a nuvem em uma Transformação Operacional permitiria um rápido descomissionamento dos ativos antigos.
* Exemplo 3: Uma empresa de imposto de renda que atende consumidores recebe 70% da receita anual durante os três primeiros meses do ano. No restante do ano, seu grande investimento em TI fica relativamente inativo. Uma Transformação Operacional permitiria que a TI implantasse a capacidade de computação/hospedagem necessária para esses três meses. Durante os nove meses restantes, os custos de IaaS poderiam ser reduzidos significativamente por meio da redução do volume de computação.

### <a name="coverdell"></a>Coverdell

A Coverdell moderniza sua infraestrutura para gerar recordes de economias com o Azure. A decisão da Coverdell em investir no Azure, e unificar sua rede de sites, aplicativos, dados e infraestrutura dentro desse ambiente, gerou mais economia de custo do que a empresa esperava. A migração para um ambiente somente no Azure eliminada 54.000 dólares em custos mensais dos serviços de colocação. Com a infra-estrutura da empresa novo, Unidos sozinha, Coverdell espera economizar uma estimado US $1 M USD nos próximos dois ou três anos.
"Que têm acesso para a pilha de tecnologia do Azure abre as portas para alguns escalável, fácil de implementar e soluções altamente disponíveis que são econômicas. Isso permite que nossa arquitetos ser muito mais criativo com as soluções que eles fornecem."
Ryan Sorensen, diretor de desenvolvimento de aplicativos e Coverdell de arquitetura empresarial

### <a name="cost-avoidance"></a>Prevenção de custos

Encerrar datacenters também pode resultar em redução de custos, impedindo que os ciclos de atualização futura. Um ciclo de atualização é o processo de aquisição de novos hardwares e softwares, a fim de substituir sistemas locais antigos. No Azure, o hardware e o sistema operacional são rotineiramente mantidos, corrigidos e atualizados sem custo adicional aos clientes. Isso permite que um diretor financeiro remova gastos futuros planejado das previsões financeiras de longo prazo. A prevenção do custo é medida em dólares. Ela difere de Redução de Custos no sentido de que se concentra, geralmente, em um orçamento futuro que ainda não foi totalmente aprovado.

* Exemplo: Datacenter da empresa está próxima da renovação de uma concessão em seis meses. Nesse Data Center está em serviço por oito anos. Há quatro anos, todos os servidores foram atualizados e virtualizado custando a empresa $ milhões. No próximo ano, a empresa pretende atualizar novamente hardware e software. Migrando os ativos naquele Datacenter, como parte de uma transformação operacionais, criaria a redução de custos, removendo a atualização planejada do orçamento previstos do próximo ano. Ela também poderia produzir uma redução de custos ao diminuir ou eliminar os custos de concessão de imóveis.

### <a name="capex-versus-opex"></a>CapEx versus OpEx

Antes de discutir os Resultados de Custo, é importante entender as duas opções principais de custo: CapEx (Despesas de capital) e OpEx (Despesas operacionais).

Os termos a seguir servem para criar uma compreensão das diferenças entre CapEx e OpEx ao se comunicar com a empresa sobre sua jornada de Transformação.

* **Capital** é o dinheiro ou ativos pertencentes a uma empresa para contribuir com uma finalidade específica, como aumentar a capacidade do servidor ou criar um aplicativo.
* **Maiuscula despesas (CapEx)** é uma despesa que gera benefícios por um longo período. Normalmente, uma despesa como essa não é recorrente e resulta na aquisição de ativos permanentes. A criação de um aplicativo poderia ser qualificada como uma despesa de capital.
* **OpEx (Despesa operacional)** é uma despesa que representa um custo contínuo dos negócios. O consumo de serviços de nuvem em um modelo pré-pago pode ser qualificado como uma despesa operacional.
* Ativo é um recurso econômico que pode ser próprio ou controlado a fim de produzir um valor. Servidores, Data Lakes e aplicativos poderiam ser considerados ativos.
* **Depreciação** é a diminuição do valor de um ativo ao longo do tempo. Um aspecto mais relevante à conversa sobre CapEx/OpEx é como os custos de um ativo são alocados nos períodos em que são usados. Por exemplo, se você criar um aplicativo deste ano, mas ele deve ter uma vida útil média de cinco anos (como aplicativos comerciais mais), em seguida, o custo da equipe de desenvolvimento e as ferramentas necessárias necessárias para criar e implantar a base de código seria ser depreciado uniformemente por cinco anos.
* **Avaliação** é o processo de estimativa de valor de uma empresa. Na maioria dos setores, a avaliação tem base na capacidade da empresa de gerar receitas e lucros, respeitando simultaneamente os custos operacionais necessários para criar os bens que fornecem essa receita. Em alguns setores, como varejo, ou em alguns tipos de transação, como capital privado, os ativos e a depreciação podem ter um papel importante na avaliação da empresa.

Geralmente, é uma aposta segura de vários executivos, inclusive do diretor de informação, debater o melhor uso do capital para expandir a empresa na direção desejada. Fornecer ao diretor de informação meios de converter conversas sobre CapEx altamente competitivo em uma contabilidade clara de OpEx pode ser um resultado interessante por si só. Em muitos setores, os diretores financeiros buscam maneiras de associar melhor a responsabilidade fiscal ao custo das mercadorias vendidas.

No entanto, antes de associar qualquer Jornada de Transformação a esse tipo de conversão de CapEx em OpEx, é aconselhável se encontrar com membros das equipes do diretor financeiro ou do diretor de informações para verificar se a empresa prefere estruturas de custo CapEx ou OpEx. Em algumas organizações, a ideia de reduzir o CapEx em favor de OpEx, é, na verdade, um resultado muito ruim. Conforme mencionado acima, isso é visto em empresas de varejo, holding e de capital privado que valorizam mais os modelos de contabilidade de ativo tradicionais, o que coloca pouco valor sobre IP. Isso também pode ser visto em organizações que tiveram experiências negativas ao terceirizar a equipe de TI ou outras funções.

Se OpEx for desejável, o exemplo a seguir poderá ser um resultado comercial viável:

* Exemplo: O datacenter da empresa no momento é depreciação em $X por ano para os próximos três anos. Espera-se exigir mais $Y para atualizar o hardware nos próximos anos. Podemos converter todos esse CapEx em um modelo de OpEx com uma taxa uniforme de $Z/mês, permitindo melhor gerenciamento e prestação de contas dos custos operacionais com tecnologia.
