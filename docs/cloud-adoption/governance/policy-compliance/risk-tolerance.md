---
title: 'CAF: Quais riscos empresariais estão associados a uma Transformação de Nuvem?'
description: Explicação sobre os riscos empresariais associados a uma Transformação de Nuvem.
author: BrianBlanchard
ms.date: 10/10/2018
ms.openlocfilehash: bfd91da42d20a85004debc6b767a1482ba3e158d
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900390"
---
# <a name="evaluating-risk-tolerance"></a>Avaliar a tolerância de risco

Cada decisão empresarial gera novos riscos. Quaisquer tipos de investimentos criam riscos de perdas. Novos produtos ou serviços criam riscos de falha de mercado. Alterações nos produtos ou serviços atuais podem reduzir a participação de mercado. Uma Transformação de Nuvem não fornece uma solução mágica para os riscos empresariais cotidianos. Pelo contrário, soluções conectadas (nuvem ou local) introduzem novos riscos. A implantação de ativos em qualquer instalação conectada à rede também amplia o perfil de ameaça potencial, expondo as vulnerabilidades de segurança a uma comunidade global muito mais ampla. Felizmente, os Provedores de Nuvem estão atentos às mudanças, ao crescimento e à adição de riscos. Eles investem fortemente para mitigar esses riscos em prol de seu clientes.

Este artigo não objetiva discutir sobre os riscos de nuvem. Em vez disso, aborda os riscos empresariais associados a várias formas de Transformação de Nuvem. No decorrer do artigo, a discussão mudará o foco para analisar as maneiras de reconhecer a tolerância aos riscos empresariais.

<!-- markdownlint-disable MD026 -->

## <a name="what-business-risks-are-associated-with-a-cloud-transformation"></a>Quais riscos empresariais estão associados a uma Transformação de Nuvem?

Lamentavelmente, os verdadeiros riscos empresariais serão baseados no que e no como por trás de transformações específicas. Felizmente, há vários riscos muito comuns que podem ser utilizados como uma discussão iniciada para reconhecer os riscos específicos e personalizados.

** Antes de ler o seguinte texto, atente-se para o fato de que cada um desses riscos pode ser corrigido. O objetivo deste artigo é informar e preparar os leitores para que as conversas de mitigação possam ser mais eficazes na mitigação de riscos. **

* Risco de proteção de dados: O risco número um associado a qualquer transformação é a proteção de dados. Na atual era digital dos negócios, os dados são o novo petróleo. Eles alimentam a economia, aquecem o escritório e fascinam os clientes. No entanto, quando vazam, o resultado é igualmente destrutivo. Quaisquer alterações na maneira como os dados são armazenados, processados ou usados geram riscos. As Transformações de Nuvem criam um alto grau de mudança em relação ao gerenciamento de dados, portanto, o risco não deve ser ignorado. A [Linha de Base de Segurança](../security-baseline/overview.md), [Classificação de Dados](./what-is-data-classification.md), e [Racionalização Incremental](../../digital-estate/rationalize.md#incremental-rationalization) podem ajudar a mitigar esse risco.

* Risco de operações e experiência do cliente: As operações de negócios e as experiências do cliente dependem fortemente de operações técnicas. As Transformações de Nuvem criarão mudanças nas operações técnicas (TechOps). Em algumas organizações, essa mudança é pequena e facilmente ajustada. Em outras organizações, as mudanças de TechOps podem exigir reorganização, requalificação ou novas abordagens para suporte. Quanto maior a mudança, maior o impacto potencial nas Operações Comerciais e Experiência do Cliente. A mitigação desse risco será decorrente do envolvimento do negócio no planejamento de transformação. O planejamento de liberação e a seleção da Primeira carga de trabalho no artigo [Racionalização Incremental](../../digital-estate/rationalize.md#incremental-rationalization) discutem maneiras de escolher cargas de trabalho para projetos de transformação. A função corporativa nessa atividade é comunicar o risco de Operações Comerciais das alterações nas cargas de trabalho priorizadas. Ajudar a TI a escolher cargas de trabalho que teriam um impacto reduzido nas operações reduzirá o risco geral.

* Risco de Custos: Os modelos de custo alteram na Nuvem. Essa alteração pode criar riscos associados a custos excessivos ou aumentos no CPV (Custo dos Produtos Vendidos), especialmente as despesas operacionais atribuídas diretamente. Quando o corporativo colabora estreitamente com a TI, é possível criar transparência em relação aos custos e serviços consumidos por várias unidades de negócios, programas, projetos, etc. O [Gerenciamento de Custos] fornece exemplos de maneiras pelas quais Corporativo e TI podem estabelecer parceria neste tópico.

Esses riscos apresentados são alguns dos mais comuns mencionados pelos clientes. A equipe de Governança de Nuvem e as equipes de adoção de Nuvem poderão começar a desenvolver um perfil de risco, na medida em que as cargas de trabalho forem migradas e preparadas para liberação de produção. Prepare-se para participar de conversas para definir, refinar e mitigar riscos com base nos resultados dos negócios desejados e no esforço de transformação.

## <a name="understanding-risk-tolerance"></a>Reconhecer a tolerância de risco

Identificar o risco é um processo bastante direto. Os riscos são bastante comuns em todos os setores. No entanto, a tolerância de risco é extremamente personalizável. Esse é o ponto em que as conversas entre corporativo e TI tendem a ficar suspensas. Basicamente, cada lado da conversa tem conhecimentos e pontos de vistas distintos. As comparações e perguntas a seguir são desenvolvidas para iniciar conversas que ajudam cada parte a melhor reconhecer e calcular a tolerância de risco.

## <a name="simple-use-case-for-comparison"></a>Caso de uso simples para comparação

Para ajudar a reconhecer a tolerância de risco, examinaremos os dados do cliente. Se uma empresa de qualquer setor publica dados de clientes em um servidor desprotegido, o risco de que os dados sejam comprometidos ou roubados é relativamente o mesmo. No entanto, a natureza dos dados e a tolerância das empresas para esse risco mudam muito.

* As empresas do setor de saúde e finanças nos Estados Unidos são regidas por rigorosos requisitos de conformidade de terceiros. Presume-se que PII (Informações de identificação pessoal) ou dados relacionados à saúde sejam extremamente confidenciais. Haverá graves consequências para esses tipos de empresas, caso estejam envolvidas no cenário de riscos acima mencionado. A tolerância será extremamente baixa. Todos os dados de clientes publicados dentro ou fora da rede precisarão ser regidos por essas políticas de conformidade de terceiros.
* Uma empresa de jogos, cujos dados do cliente são limitados a um nome do usuário, tempos de jogo e recordes, não terá a mesma probabilidade de sofrer consequências significativas, caso se envolva no comportamento de risco acima. Algum dado desprotegido está em risco, o impacto desse risco é pequeno. Portanto, nesse caso a tolerância de risco é alta.
* Uma empresa de porte médio que fornece serviços de limpeza de carpetes para milhares de clientes se enquadraria entre esses dois extremos de tolerância. Os dados do cliente podem ser mais robustos, contendo detalhes como endereço ou número de telefone. Ambos podem ser considerados PII e devem ser protegidos. No entanto, pode não haver requisitos de governança específicos exigindo a proteção dos dados. De uma perspectiva de TI, a resposta é simples, proteja os dados. De uma perspectiva corporativa, pode não ser tão simples. O corporativo precisaria de mais detalhes antes de determinar um nível de tolerância para esse risco.

A próxima seção compartilha algumas perguntas que podem ajudar a empresa a determinar um nível de tolerância de risco para o caso de uso acima, ou outros.

## <a name="risk-tolerance-questions"></a>Perguntas sobre tolerância de risco

Esta seção lista as perguntas que estimulam conversas em três categorias: Impacto da Perda, Probabilidade de Perda e Custos de Mitigação. Quando o corporativo e os parceiros de TI abordam cada uma dessas áreas, a decisão de mitigar riscos e a tolerância geral a um risco específico pode ser facilmente determinada.

**Impacto da Perda**: Perguntas para determinar o impacto de um risco. Essas perguntas podem ser difíceis (às vezes impossíveis) de responder. Quantificar o impacto é o ideal, porém, algumas vezes a própria conversa é suficiente para reconhecer a tolerância. Os intervalos também são aceitáveis, especialmente se incluem suposições que determinam esses intervalos.

* Esse risco viola os requisitos de conformidade de terceiros?
* Esse risco viola as políticas corporativas internas?
* Esse risco poderia prejudicar clientes ou participação de mercado? Se sim, esse impacto pode ser quantificado?
* Esse risco poderia criar experiências negativas para o cliente? Essas experiências podem impactar na execução de vendas ou receita?
* Este risco poderia criar uma nova responsabilidade jurídica? Em caso afirmativo, há uma precedência para compensação por danos nesses tipos de casos?
* Esse risco poderia interromper as operações de negócios? Em caso afirmativo, por quanto tempo as operações ficariam inativas?
* Isso poderia resultar em operações de negócios lentas? Em caso afirmativo, o quão lenta e por quanto tempo?
* Nesse estágio da transformação, esse é um risco único ou será repetido?
* O risco aumenta ou diminui em frequência, na medida em que a transformação avança?
* O risco aumenta ou diminui em probabilidade ao longo do tempo?
* O risco é por natureza sensível ao tempo? O risco vai passar ou agravar, se não for resolvido?

Essas perguntas básicas conduzirão a muitas outras. Após explorar um diálogo produtivo, é recomendável que os riscos relevantes sejam registrados e, quando possível, quantificados.

**Custos de mitigação**: Perguntas para determinar o custo de mitigação ou remoção do risco. Essas perguntas podem ser bastante diretas, especialmente quando representadas em faixa.

* Há uma solução clara? Qual é o custo?
* Há opções para resolver ou mitigar esse risco? Qual é a faixa de custo para essas soluções?
* O que é necessário do corporativo para escolher a solução clara e ideal?
* O que é necessário do corporativo para validar os custos?
* Quais outros benefícios podem resultar da solução que mitigaria esse risco?

Essas perguntas simplificam as soluções técnicas necessárias para mitigar os riscos. No entanto, essas questões comunicam essas soluções de maneira que a empresa possa integrar-se rapidamente a um processo de decisão.

**Probabilidade de Perda**: Perguntas para determinar a probabilidade do risco tornar-se realidade. Essa é a área mais difícil de quantificar. Em vez disso, é recomendável que a equipe de Governança de Nuvem crie categorias para a probabilidade de comunicação, com base nos dados de suporte. As perguntas a seguir podem ajudar a criar categorias que sejam significativas para a equipe.

* Alguma pesquisa foi realizada acerca da probabilidade desse risco ser realizado?
* O fornecedor pode fornecer referências ou estatísticas sobre a probabilidade do impacto?
* Há outras empresas do setor relevante ou vertical que foram atingidas por esse risco?
* Pesquise mais, há outras empresas em geral que foram atingidas por esse risco?
* Esse risco é exclusivo relacionado a algo que a empresa tem feito inadequadamente?

Após responder a essas perguntas e a outras mais, conforme determinado pela equipe de Governança de Nuvem, provavelmente surgirão agrupamentos de probabilidade. A seguir, estão alguns exemplos de agrupamento para ajudar a começar:

* Nenhuma indicação: Não há pesquisas concluídas suficientes para determinar a probabilidade
* Baixo risco: Pesquisas atuais sugerem que não há probabilidade do risco ser realizado
* Risco futuro: A probabilidade atual é de Baixo Risco. No entanto, a adoção continuada dispararia uma análise atualizada
* Risco Médio: É provável que o risco tenha impacto nos negócios
* Alto Risco: Ao longo do tempo, é cada vez menos provável que o negócio evite o impacto desse risco
* Risco em declínio: O risco é Médio a Alto. No entanto, ações em TI ou negócios estão reduzindo a probabilidade de um impacto.

**Determinar tolerância:**

Os três conjuntos de perguntas acima devem fornecer dados suficientes para determinar as tolerâncias iniciais. Quando o risco e a probabilidade forem baixos e o custo de mitigação for alto, será improvável que o corporativo invista em correção. Quando o risco e a probabilidade forem altos, será provável que o corporativo considere um investimento, desde que os custos não excedam os riscos potenciais.

## <a name="next-steps"></a>Próximas etapas

Esse tipo de conversa pode ajudar o corporativo e a TI a avaliarem a tolerância de maneira mais eficaz. Essas conversas podem ser usadas durante a criação de políticas de MVP e durante análises de políticas incrementais.

> [!div class="nextstepaction"]
> [Definir política corporativa](./define-policy.md)
