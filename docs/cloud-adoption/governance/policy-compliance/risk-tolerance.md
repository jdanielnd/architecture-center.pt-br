---
title: 'CAF: Quais riscos empresariais estão associados a uma Transformação de Nuvem?'
description: Explicação sobre os riscos empresariais associados a uma Transformação de Nuvem.
author: BrianBlanchard
ms.date: 10/10/2018
ms.openlocfilehash: 774a6703b353a3c670b3764505185aecb794784f
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59068932"
---
# <a name="evaluating-risk-tolerance"></a>Avaliar a tolerância de risco

Cada decisão empresarial gera novos riscos. Quaisquer tipos de investimentos criam riscos de perdas. Novos produtos ou serviços criam riscos de falha de mercado. Alterações nos produtos ou serviços atuais podem reduzir a participação de mercado. Uma Transformação de Nuvem não fornece uma solução mágica para os riscos empresariais cotidianos. Pelo contrário, soluções conectadas (nuvem ou local) introduzem novos riscos. A implantação de ativos em qualquer instalação conectada à rede também amplia o perfil de ameaça potencial, expondo as vulnerabilidades de segurança a uma comunidade global muito mais ampla. Felizmente, os Provedores de Nuvem estão atentos às mudanças, ao crescimento e à adição de riscos. Eles investem intensamente para reduzir e gerenciar esses riscos em nome de seus clientes.

Este artigo não objetiva discutir sobre os riscos de nuvem. Em vez disso, aborda os riscos empresariais associados a várias formas de Transformação de Nuvem. No decorrer do artigo, a discussão mudará o foco para analisar as maneiras de reconhecer a tolerância aos riscos empresariais.

<!-- markdownlint-disable MD026 -->

## <a name="what-business-risks-are-associated-with-a-cloud-transformation"></a>Quais riscos empresariais estão associados a uma Transformação de Nuvem?

Os riscos de negócios True se baseará o que e como por trás de transformações específicas. Felizmente, há vários riscos muito comuns que podem ser utilizados como uma discussão iniciada para reconhecer os riscos específicos e personalizados.

* * Antes de ler o seguinte, lembre-se de que cada um desses riscos pode ser gerenciada. O objetivo deste artigo é informar e preparar os leitores, para que a discussão sobre gerenciamento de risco pode ser mais eficiente. **

- **Violação de dados**: O risco número um associado a qualquer transformação é a proteção de dados. Vazamentos de dados pode causar danos significativos à sua empresa, levando à perda de clientes, diminuição business ou responsabilidade legal mesmo. Quaisquer alterações na maneira como os dados são armazenados, processados ou usados geram riscos. As Transformações de Nuvem criam um alto grau de mudança em relação ao gerenciamento de dados, portanto, o risco não deve ser ignorado. [Linha de base de segurança](../security-baseline/overview.md), [classificação de dados](./what-is-data-classification.md), e [racionalização Incremental](../../digital-estate/rationalize.md#incremental-rationalization) pode ajudar a gerenciar esse risco.

- **Interrupção de serviço**: Operações de negócios e experiências do cliente dependem muito operações técnicas. As Transformações de Nuvem criarão mudanças nas operações técnicas (TechOps). Em algumas organizações, que a alteração é pequeno e facilmente ajustado. Em outras organizações, as mudanças de TechOps podem exigir reorganização, requalificação ou novas abordagens para suporte. Quanto maior a alteração, quanto maior o efeito potencial sobre operações de negócios e experiência do cliente. Gerenciar esse risco exigirá o envolvimento dos negócios no planejamento de transformação. O planejamento de liberação e a seleção da Primeira carga de trabalho no artigo [Racionalização Incremental](../../digital-estate/rationalize.md#incremental-rationalization) discutem maneiras de escolher cargas de trabalho para projetos de transformação. Função da empresa em que a atividade é comunicar-se o risco de operações de negócios de alterar prioridade cargas de trabalho. Ajudando IT escolha as cargas de trabalho que têm um menor impacto nas operações reduzirá o risco geral.

- **Controle de orçamento**: Os modelos de custo alteram na Nuvem. Essa alteração pode criar riscos associados a custos excessivos ou aumentos no CPV (Custo dos Produtos Vendidos), especialmente as despesas operacionais atribuídas diretamente. Quando os negócios trabalha junto com o IT, é possível criar a transparência sobre os custos e os serviços consumidos por várias unidades de negócios, programas, projetos, etc... [Gerenciamento de custos](../cost-management/overview.md) fornece exemplos de maneiras de negócios e TI pode parceiro sobre esse tópico.

Esses riscos apresentados são alguns dos mais comuns mencionados pelos clientes. A equipe de Governança de Nuvem e as equipes de adoção de Nuvem poderão começar a desenvolver um perfil de risco, na medida em que as cargas de trabalho forem migradas e preparadas para liberação de produção. Esteja preparado para conversas definir, refinar e gerenciar riscos com base no esforço de transformação e os resultados comerciais desejados.

## <a name="understanding-risk-tolerance"></a>Reconhecer a tolerância de risco

Identificar o risco é um processo bastante direto. Riscos relacionados a TI são geralmente padrão em vários setores. No entanto, a tolerância para esses riscos é específica para cada organização. Esse é o ponto em que as conversas entre corporativo e TI tendem a ficar suspensas. Basicamente, cada lado da conversa tem conhecimentos e pontos de vistas distintos. As comparações e perguntas a seguir são desenvolvidas para iniciar conversas que ajudam cada parte a melhor reconhecer e calcular a tolerância de risco.

## <a name="simple-use-case-for-comparison"></a>Caso de uso simples para comparação

Para ajudar a entender a tolerância a riscos, vamos examinar os dados do cliente. Se uma empresa em qualquer setor postar os dados do cliente em um servidor desprotegido, o risco técnico de que os dados que está sendo comprometida ou roubada é aproximadamente o mesmo. No entanto, a tolerância da empresa para esse risco irá variar amplamente com base no valor da natureza e o potencial dos dados.

- As empresas do setor de saúde e finanças nos Estados Unidos são regidas por rigorosos requisitos de conformidade de terceiros. Presume-se que PII (Informações de identificação pessoal) ou dados relacionados à saúde sejam extremamente confidenciais. Haverá graves consequências para esses tipos de empresas, caso estejam envolvidas no cenário de riscos acima mencionado. A tolerância será extremamente baixa. Todos os dados de clientes publicados dentro ou fora da rede precisarão ser regidos por essas políticas de conformidade de terceiros.
- Uma empresa de jogos cujos dados de cliente são limitados a um nome de usuário, os tempos de execução e pontuações mais altas não é mais provável de sofrer consequências qualquer significativas, se envolva-se no comportamento arriscado acima. Enquanto todos os dados não seguros estão em risco, o impacto desse risco é pequeno. Portanto, a tolerância a riscos nesse caso é alta.
- Uma empresa de médio porte que fornece o tapete de limpeza de serviços para milhares de clientes deve se enquadrar entre esses extremos de tolerância de dois. Lá, os dados do cliente podem ser mais robustos, que contém detalhes como o endereço ou número de telefone. Ambos podem ser considerados PII e devem ser protegidos. No entanto, pode não haver requisitos de governança específicos exigindo a proteção dos dados. De uma perspectiva de TI, a resposta é simples, proteja os dados. De uma perspectiva corporativa, pode não ser tão simples. O corporativo precisaria de mais detalhes antes de determinar um nível de tolerância para esse risco.

A próxima seção compartilha alguns exemplos de perguntas que podem ajudar os negócios determinar um nível de tolerância de risco para o caso de uso acima ou outras pessoas.

## <a name="risk-tolerance-questions"></a>Perguntas sobre tolerância de risco

Esta seção lista conversa provocar perguntas em três categorias: impacto de perda, a probabilidade de perda e custos de remediação. Quando o parceiro de TI para resolver cada uma dessas áreas, a decisão de fazer o esforço no gerenciamento de riscos e a tolerância geral de um risco específico e de negócios podem facilmente ser determinado.

**Impacto da perda**: Perguntas para determinar o impacto de um risco. Essas perguntas podem ser difíceis (às vezes impossíveis) de responder. Quantificar o impacto é o ideal, porém, algumas vezes a própria conversa é suficiente para reconhecer a tolerância. Os intervalos também são aceitáveis, especialmente se incluem suposições que determinam esses intervalos.

- Esse risco viola os requisitos de conformidade de terceiros?
- Esse risco viola as políticas corporativas internas?
- Esse risco poderia prejudicar clientes ou participação de mercado? Se assim, esse custo pode ser quantificado?
- Esse risco poderia criar experiências negativas para o cliente? Essas experiências são provavelmente afetará de vendas ou realização de receita?
- Este risco poderia criar uma nova responsabilidade jurídica? Em caso afirmativo, há uma precedência para compensação por danos nesses tipos de casos?
- Esse risco poderia interromper as operações de negócios? Em caso afirmativo, por quanto tempo as operações ficariam inativas?
- Isso poderia resultar em operações de negócios lentas? Nesse caso, como lento e por quanto tempo?
- Nesse estágio da transformação, esse é um risco único ou será repetido?
- O risco aumenta ou diminui em frequência, na medida em que a transformação avança?
- O risco aumenta ou diminui em probabilidade ao longo do tempo?
- O risco é por natureza sensível ao tempo? O risco vai passar ou agravar, se não for resolvido?

Essas perguntas básicas conduzirão a muitas outras. Após explorar um diálogo produtivo, é recomendável que os riscos relevantes sejam registrados e, quando possível, quantificados.

**Os custos de correção de risco**: Perguntas para determinar o custo de remoção ou caso contrário, minimizando o risco. Essas perguntas podem ser bastante diretas, especialmente quando representadas em faixa.

- Há uma solução clara? Qual é o custo?
- Há opções para impedir ou minimizar esse risco? Qual é a faixa de custo para essas soluções?
- O que é necessário do corporativo para escolher a solução clara e ideal?
- O que é necessário do corporativo para validar os custos?
- O que outros benefícios podem vir da solução que deseja remover esse risco?

Essas perguntas sobre simplificam as soluções técnicas necessárias para gerenciar ou remover os riscos. No entanto, essas questões comunicam essas soluções de maneira que a empresa possa integrar-se rapidamente a um processo de decisão.

**Probabilidade de Perda**: Perguntas para determinar a probabilidade do risco tornar-se realidade. Essa é a área mais difícil de quantificar. Em vez disso, é recomendável que a equipe de Governança de Nuvem crie categorias para a probabilidade de comunicação, com base nos dados de suporte. As perguntas a seguir podem ajudar a criar categorias que sejam significativas para a equipe.

- Alguma pesquisa foi realizada acerca da probabilidade desse risco ser realizado?
- O fornecedor pode fornecer referências ou estatísticas sobre a probabilidade do impacto?
- Há outras empresas do setor relevante ou vertical que foram atingidas por esse risco?
- Pesquise mais, há outras empresas em geral que foram atingidas por esse risco?
- Esse risco é exclusivo relacionado a algo que a empresa tem feito inadequadamente?

Depois de responder a essas perguntas, juntamente com perguntas, conforme determinado pela equipe de governança de nuvem, agrupamentos de probabilidade aparecerem. A seguir, estão alguns exemplos de agrupamento para ajudar a começar:

- Nenhuma indicação: Não há pesquisa foi concluída para determinar a probabilidade.
- Baixo risco: Pesquisa atual sugere que o risco não é provável que a serem obtidos.
- Risco futuro: A probabilidade atual é de Baixo Risco. No entanto, a adoção contínua dispararia uma nova análise.
- Risco Médio: É provável que o risco terá impacto sobre os negócios.
- Alto Risco: Com o tempo, ele está aumentando menos provável que os negócios evitará o impacto desse risco.
- Risco em declínio: O risco é Médio a Alto. No entanto, ações em TI ou negócios estão reduzindo a probabilidade de um impacto.

**Determinar tolerância:**

Os três conjuntos de perguntas acima devem fornecer dados suficientes para determinar as tolerâncias iniciais. Quando o risco e probabilidade são baixos e custos de correção de risco são altos, a empresa é improvável que investir na correção. Quando o risco e probabilidade são altas, a empresa provavelmente considerar um investimento, desde que os custos não excedam os riscos potenciais.

## <a name="next-steps"></a>Próximas etapas

Esse tipo de conversa pode ajudar o corporativo e a TI a avaliarem a tolerância de maneira mais eficaz. Essas conversas podem ser usadas durante a criação de políticas de MVP e durante análises de políticas incrementais.

> [!div class="nextstepaction"]
> [Definir política corporativa](./define-policy.md)
