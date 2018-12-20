---
title: Racionalizar a propriedade digital
titleSuffix: Enterprise Cloud Adoption
description: Um processo para avaliar os ativos digitais e encontrar a melhor maneira de hospedá-los na nuvem.
author: BrianBlanchard
ms.date: 12/10/2018
ms.openlocfilehash: 05509bdd047b93b4a3b41907836022c837f9c7b4
ms.sourcegitcommit: e7f8676bbffe500fc4d6deb603b7c0b7ba1884a6
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/10/2018
ms.locfileid: "53179594"
---
# <a name="enterprise-cloud-adoption-rationalize-the-digital-estate"></a>Adoção do Enterprise Cloud: Racionalizar a propriedade digital

A racionalização de nuvem é o processo de avaliação de ativos para determinar a melhor abordagem para hospedá-los na nuvem. Quando uma [abordagem](approach.md) é estabelecida e um [inventário](inventory.md) é levantado, a racionalização de nuvem pode ser realizada. Os [5 Rs de racionalização](5-rs-of-rationalization.md) discutem as opções mais comuns de racionalização.

## <a name="traditional-view-of-rationalization"></a>Visão tradicional de racionalização

É fácil entender a racionalização ao visualizar seu processo tradicional como uma árvore de decisão complexa. Cada ativo na propriedade digital é inserido por meio de um processo que resulta em uma de cinco respostas (os 5 Rs). Para propriedades pequenas, esse processo funciona bem. Para propriedades maiores, ele não é muito eficiente e pode resultar em atrasos significativos. Vamos examinar o processo para ver o motivo. Em seguida, apresentaremos um modelo mais eficiente.

**Inventário**. Um inventário completo dos ativos, incluindo aplicativos, software, hardware, sistemas operacionais e métricas de desempenho do sistema, é necessário para concluir uma racionalização completa usando modelos tradicionais.

**Análise quantitativa**. Na árvore de decisão, perguntas quantitativas geram a primeira camada de decisões. Perguntas comuns: O ativo está sendo usado atualmente? Nesse caso, ele está otimizado e dimensionado corretamente? Quais dependências existem entre os ativos? Essas perguntas são vitais para a classificação do inventário.

**Análise qualitativa**. O próximo conjunto de decisões exige inteligência humana na forma de análise qualitativa. Muitas vezes, essas perguntas são exclusivas para a solução e só podem ser respondidas pelos participantes da empresa e por usuários avançados. Essas decisões é que costumam atrasar o processo, atrasando tudo consideravelmente. Essa análise geralmente consome 40&ndash;80 FTE por aplicativo. Para obter diretrizes sobre como criar uma lista de perguntas de análise qualitativa, consulte [Abordagens para o planejamento de propriedade digital](approach.md).

**Decisão de racionalização**. Nas mãos de uma equipe experiente de racionalização, os dados qualitativos e quantitativos criam decisões claras. Infelizmente, as equipes com alto grau de experiência em racionalização são caras de se contratar ou precisam de meses de treinamento.

## <a name="rationalization-at-enterprise-scale"></a>Racionalização em escala empresarial

Se essa iniciativa é demorada e assustadora para uma propriedade digital de 50 VMs, imagine o esforço necessário para a transformação do negócio em um ambiente com milhares de VMs e centenas de aplicativos. O esforço humano necessário pode ultrapassar facilmente 1.500 FTE e nove meses de planejamento.

Embora a racionalização completa seja o estado final e uma ótima meta, ela raramente produz um alto ROI em relação ao tempo e à energia exigidos.

Quando a racionalização é essencial para decisões financeiras, vale a pena considerar contratar uma organização de serviços profissionais especializada em racionalização de nuvem para acelerar o processo. Mesmo assim, a racionalização completa pode ser um esforço caro e demorado que poderia atrasar os resultados de negócios ou a transformação.

O restante deste artigo descreve uma abordagem alternativa, conhecida como racionalização incremental.

## <a name="incremental-rationalization"></a>Racionalização incremental

A racionalização completa de uma grande propriedade digital está propensa a riscos e pode sofrer atrasos associados à complexidade. A suposição por trás da abordagem incremental é a de que decisões adiadas podem balancear a carga sobre o negócio para reduzir o risco de obstáculos. Ao longo do tempo, essa abordagem cria um modelo orgânico para desenvolver os processos e a experiência necessários para a tomada de decisões de racionalização qualificadas e fazer isso com mais eficiência.

### <a name="inventory-reduce-discovery-data-points"></a>Inventário: Reduzir pontos de dados de descoberta

Pouquíssimas organizações investem o tempo, a energia e as despesas necessárias para manter um inventário preciso e em tempo real de toda a propriedade digital. Perda, roubo, ciclos de atualização e integração de funcionários geralmente justificam o acompanhamento detalhado de ativos de dispositivos do usuário final. No entanto, o retorno sobre o investimento (ROI) de manter um inventário de servidores e aplicativos preciso em um datacenter local tradicional geralmente é baixo. A maioria das organizações de TI têm outros problemas mais urgentes do que acompanhar o uso de ativos fixos em um data center.

Em uma Transformação de Nuvem, o inventário se correlaciona diretamente com os custos operacionais. São necessários dados de inventário precisos para o planejamento adequado. Infelizmente, as opções de verificação do ambiente atuais podem atrasar as decisões por semanas ou meses, para verificar e catalogar todo o inventário. Felizmente, existem alguns truques para acelerar a coleta de dados.

A verificação baseada em agente é o atraso citado com mais frequência. Os dados robustos necessários para uma racionalização tradicional geralmente dependem de dados que só podem ser coletados com um agente em execução em cada ativo. Essa dependência dos agentes geralmente reduz o progresso, já que ela pode exigir interações com funções de administração, operações e segurança.

Em um processo de racionalização incremental, uma solução sem agente pode ser usada para uma descoberta inicial para acelerar as decisões iniciais. Dependendo do nível de complexidade do ambiente, ainda poderá ser necessário usar uma solução baseada em agente, mas ele pode ser removido do caminho crítico da alteração comercial.

### <a name="quantitative-analysis-streamline-decisions"></a>Análise quantitativa: Decisões simplificadas

Independentemente da abordagem para a descoberta de inventário, a análise quantitativa pode gerar diversas decisões e suposições iniciais. Isso é verdade principalmente ao tentar identificar a primeira carga de trabalho ou quando o objetivo da racionalização é uma comparação geral de custos. Em um processo de racionalização incremental, as equipes de Estratégia de Nuvem e Adoção de Nuvem limitam os [5 Rs de racionalização](5-rs-of-rationalization.md) a duas decisões concisas e aplicam somente esses fatores quantitativos, simplificando a análise e reduzindo a quantidade de dados iniciais necessários para gerar mudanças.

Por exemplo, se uma organização estiver no meio de uma migração de IaaS para a nuvem, é possível pressupor que a maioria das cargas de trabalho serão desativadas ou hospedadas novamente.

### <a name="qualitative-analysis-temporary-assumptions"></a>Análise qualitativa: Suposições temporárias

Ao reduzir o número de resultados possíveis, fica mais fácil chegar a uma decisão inicial sobre o estado futuro de um ativo. Quando as opções são reduzidas, o número de perguntas feitas sobre o negócio nesse estágio inicial também é reduzido.

Continuando com o exemplo acima, se as opções são limitadas a hospedar novamente ou desativar, há, de fato, apenas uma pergunta a fazer à empresa durante a racionalização inicial &mdash;, ou seja, se ela quer desativar ou não.

“A análise sugere que não há nenhum usuário aproveitando esse ativo. Isso está correto ou deixamos passar alguma coisa?” Uma pergunta binária é geralmente muito mais fácil de usar em uma análise qualitativa.

Essa abordagem simplificada produz linhas de base, planos financeiros, estratégia e orientações. Em atividades posteriores, cada ativo passaria por mais racionalização e análise qualitativa para avaliar outras opções. Todas as suposições feitas nessa racionalização inicial devem ser testadas antes da implementação.

## <a name="challenging-assumptions"></a>Desafiar suposições

O resultado da seção anterior é uma racionalização aproximada cheia de suposições. A seguir, vamos desafiar algumas dessas suposições.

### <a name="retiring-assets"></a>Desativar ativos

Em um ambiente local tradicional, hospedar ativos pequenos e não usados raramente causa um impacto significativo nos custos anuais. Com poucas exceções, a economia de custos associada à remoção e à desativação desses ativos é superada pelo esforço de FTE exigido para analisar e desativar o ativo em si.

No entanto, ao mudar para um modelo de contabilidade na nuvem, a desativação de ativos pode gerar uma economia significativa de custos operacionais anuais e dos esforços de migração iniciais.

Não é incomum as organizações desativarem 20% ou mais de sua propriedade digital depois da conclusão de uma análise quantitativa. É recomendável fazer uma análise qualitativa antes de decidir sobre essa ação. Depois de confirmada, a desativação desses ativos pode produzir a primeiro vitória do ROI da migração para a nuvem. Em muitos casos, esse é um dos maiores fatores de economia de custos. Como tal, é aconselhável que a equipe de Estratégia de Nuvem supervisione a validação e a desativação de ativos, paralelamente à fase de criação do processo de migração, para permitir uma vitória financeira antecipada.

### <a name="program-adjustments"></a>Ajustes do programa

Uma empresa raramente embarca em apenas uma jornada de transformação. A escolha entre redução de custos, crescimento no mercado e novos fluxos de receita raramente é binária. Como tal, é aconselhável que a equipe de Estratégia de Nuvem trabalhe com o departamento de TI para identificar ativos em iniciativas de transformação paralelas que estejam fora do escopo da Jornada de Transformação principal.

No exemplo de Migração do IaaS usado neste artigo:

- Peça à equipe de DevOps para identificar os ativos que já fazem parte de uma automação da implantação e remova-as do plano de migração principal.

- Peça às equipes de dados e de pesquisa e desenvolvimento para identificar ativos que estão viabilizando novos fluxos de receita e remova-os do plano de migração principal.

Essa análise qualitativa voltada para o programa pode ser executada rapidamente e criará alinhamento entre várias listas de pendências de migração.

Alguns ativos ainda podem precisar ser tratados como ativos de nova hospedagem por algum tempo, entrando na racionalização posteriormente, após a migração inicial.

## <a name="selecting-the-first-workload"></a>Selecionar a primeira carga de trabalho

A implementação da primeira carga de trabalho é a chave para testes e aprendizado. É a primeira oportunidade de demonstrar e criar uma mentalidade de crescimento.

### <a name="business-criteria"></a>Critérios comerciais

Identifique uma carga de trabalho que esteja atribuída a um membro da unidade de negócios da equipe de Estratégia de Nuvem para garantir a transparência dos negócios. Preferencialmente, escolha uma na qual a equipe tenha grande interesse e forte motivação para migrar para a nuvem.

### <a name="technical-criteria"></a>Critérios técnicos

Selecione uma carga de trabalho que tenha dependências mínimas e que possa ser migrada como um pequeno grupo de ativos. É recomendável que uma carga de trabalho com um caminho de teste definido seja escolhida para facilitar a validação.

A primeira carga de trabalho é normalmente implantada em um ambiente experimental sem capacidade de governança ou operacional. É muito importante escolher uma carga de trabalho que não interaja com dados seguros.

### <a name="qualitative-analysis"></a>Análise qualitativa

As equipes de Adoção de Nuvem e Estratégia de Nuvem podem trabalhar juntas para analisar essa pequena carga de trabalho. Isso cria uma oportunidade controlada para criar e testar os critérios de análise qualitativa. A população menor cria a oportunidade de fazer uma pesquisa com os usuários afetados, para concluir uma análise qualitativa detalhada em uma semana ou menos. Para ver os fatores de análise qualitativa comuns, consulte o destino de racionalização específico nos [5 Rs da racionalização](5-rs-of-rationalization.md).

### <a name="migration"></a>Migração

Paralelamente à racionalização contínua, a equipe de Adoção de Nuvem pode começar a migrar a pequena carga de trabalho para expandir o aprendizado nas seguintes áreas principais:

- Melhorar habilidades com a plataforma do provedor de nuvem.
- Definir os serviços principais (e padrões do Azure) necessários para a visão a longo prazo.
- Entender melhor como as operações possam ter que evoluir mais tarde na transformação.
- Entender os riscos inerentes aos negócios e a tolerância dos negócios em relação a esses riscos.
- Estabelecer uma linha de base ou produto minimamente viável (MVP) para governança com base na tolerância dos negócios a riscos

## <a name="release-planning"></a>Planejamento de lançamento

Enquanto a equipe de Adoção de Nuvem está executando a migração ou a implementação da primeira carga de trabalho, a equipe de Estratégia de Nuvem pode começar a priorizar os aplicativos/cargas de trabalho restantes.

### <a name="power-of-ten"></a>Poder dos Dez

O método tradicional de racionalização tenta aquecer um oceano inteiro. Felizmente, nem sempre é necessário ter um plano para todos os aplicativos para iniciar uma jornada de transformação. Em um modelo incremental, o Poder dos Dez fornece um bom ponto de partida. Nesse modelo, a equipe de estratégia de nuvem seleciona os dez primeiros aplicativos a serem migrados. Essas dez cargas de trabalho devem conter uma mistura de cargas de trabalho simples e complexas.

### <a name="building-the-first-backlogs"></a>Criar as primeiras listas de pendências

As equipes de Adoção de Nuvem e Estratégia de Nuvem podem trabalhar juntas na análise qualitativa nas dez primeiras cargas de trabalho. Isso cria a primeira lista de pendências de migração priorizada e a primeira lista de pendências de versão priorizada. Essa abordagem permite que as equipes iterem sobre a abordagem e fornece tempo suficiente para criar um processo adequado para a análise qualitativa.

### <a name="maturing-the-process"></a>Amadurecer o processo

Depois que as duas equipes chegarem a um acordo em relação aos critérios de análise qualitativa, a avaliação poderá se tornar uma tarefa dentro de cada iteração. Um consenso sobre critérios de avaliação geralmente exige 2&ndash;3 versões.

Depois que a avaliação é transferida para os processos de execução incremental da migração, a equipe de Adoção de Nuvem pode iterar sobre a arquitetura e a avaliação com mais rapidez. Nesse estágio, a equipe de Estratégia de Nuvem também é ignorada, reduzindo o consumo de seu tempo. Isso também permite que a equipe de Estratégia de Nuvem se concentre em priorizar os aplicativos que ainda não estão em uma versão específica, garantindo assim um forte alinhamento com as mudanças nas condições de mercado.

Nem todos os aplicativos priorizados estarão prontos para a migração. O sequenciamento possivelmente vai mudar, conforme a equipe vai fazendo a análise qualitativa de forma mais detalhada e descobrindo eventos comerciais e dependências que vão exigindo uma nova priorização da lista de pendências. Algumas versões podem agrupar um pequeno número de cargas de trabalho. Outras podem conter apenas uma única carga de trabalho.

A equipe de Adoção de Nuvem provavelmente executará iterações que não produzem uma migração completa da carga de trabalho. Quanto menor for a carga de trabalho e as dependências, maior a probabilidade de uma carga de trabalho se encaixar em um único sprint ou iteração. Por esse motivo, é aconselhável que os primeiros aplicativos na lista de pendências de versão sejam pequenos e contenham algumas dependências externas.

## <a name="end-state"></a>Estado final

Ao longo do tempo, a combinação das equipes de Adoção de Nuvem e de Estratégia de Nuvem concluirão uma racionalização completa do inventário. No entanto, essa abordagem incremental permite que as equipes fiquem cada vez mais rápidas no processo de racionalização. Ela também permite que a Jornada de Transformação produza resultados comerciais tangíveis mais cedo, sem a necessidade de um grande esforço de análise inicial.

Em alguns casos, o modelo financeiro pode ser muito rígido para gerar uma decisão sem mais racionalização. Nesses casos, pode ser necessária uma abordagem mais tradicional da racionalização.

## <a name="next-steps"></a>Próximas etapas

A saída de um esforço de racionalização é uma lista de pendências priorizada de todos os ativos afetados pela transformação escolhida. Essa lista de pendências agora está pronta para servir como base para modelos de custo dos serviços de nuvem.

> [!div class="nextstepaction"]
> [Alinhar modelos de custo com a propriedade digital](calculate.md)