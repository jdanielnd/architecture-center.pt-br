---
title: 'CAF: Como os riscos empresariais alteraram na nuvem?'
description: Noções básicas sobre riscos empresariais durante a migração
author: BrianBlanchard
ms.date: 10/10/2018
ms.openlocfilehash: 458474f3c94c5df4f7ffef439095adf138f33d78
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58246487"
---
<!-- markdownlint-disable MD026 -->

# <a name="how-does-business-risk-change-in-the-cloud"></a>Como os riscos empresariais alteraram na nuvem?

Compreender os riscos empresariais é essencial para qualquer transformação de nuvem. O risco impulsiona a política e influencia os requisitos de imposição e monitoramento. O risco influencia profundamente a forma como gerenciamos a propriedade digital local ou na nuvem.

<!-- markdownlint-enable MD026 -->

## <a name="relativity-of-risk"></a>Relatividade do risco

O risco é relativo. Uma pequena empresa com poucos ativos de TI, em um prédio fechado, tem pouco risco. Adicionar usuários e uma conexão de internet com acesso a esses ativos, o risco é intensificado. Quando essa pequena empresa cresce para status de classificação Fortune 500, os riscos tornam-se exponencialmente maiores. Na medida em que as receitas, os processos de negócios, as contagens de funcionários e os ativos de TI se acumulam, os riscos aumentam e se aglutinam. Os ativos de TI que auxiliam na geração de receita correm riscos reais de interromper esse fluxo de receita em caso de uma interrupção. Cada momento de inatividade equivale a perdas. Da mesma forma, na medida em que os dados se acumulam, o risco de prejudicar os clientes aumenta.

No mundo local tradicional, as equipes de governança de TI concentram-se na avaliação desses riscos. Criando processos para mitigar o risco. Implantando sistemas para garantir que medidas de mitigação sejam implementadas e bem-sucedidas. Isso equilibra os riscos necessários para operar em um ambiente de negócios moderno e conectado.

## <a name="understanding-business-risks-in-the-cloud"></a>Reconhecer riscos empresariais na nuvem

Durante uma transformação, os mesmos riscos relativos poderão ser verificados.

* Durante a experimentação inicial, alguns ativos são implantados com pouco ou nenhum dado relevante. O risco é pequeno.
* Quando a primeira carga de trabalho é implementada, o risco aumenta um pouco. Esse risco é facilmente mitigado pela escolha de um aplicativo inerentemente de baixo risco com uma pequena base de usuários.
* À medida que mais cargas de trabalho tornam-se online, os riscos alteram a cada liberação. Novos aplicativos são ativados, os riscos alteram.
* Quando uma empresa coloca os primeiros 10 a 20 aplicativos online, o perfil de risco é muito diferente quando o milésimo aplicativo entra em produção na nuvem.

Os ativos acumulados na propriedade tradicional local, provavelmente se acumularam ao longo do tempo. A maturidade das equipes de negócios e de TI provavelmente estava crescendo de maneira semelhante. Esse crescimento paralelo pode tender a criar alguma bagagem política desnecessária.

Durante uma transformação de nuvem, ambas as equipes de negócios e TI terão a oportunidade de redefinir essas políticas e criar novas com um conhecimento avançado.

<!-- markdownlint-disable MD026 -->

## <a name="what-is-a-business-risk-mvp"></a>O que é um MVP de risco empresarial?

**Produto mínimo viável** é um termo padrão do setor para definir a menor unidade de algo que possa produzir um valor tangível. Em um MVP de risco de negócios, a equipe começa com uma suposição de que alguns ativos serão implantados em um ambiente de nuvem. No momento, esses ativos são desconhecidos. Também são desconhecidos os tipos de dados que serão processados por esses ativos.

A equipe de Governança de Nuvem poderia criar o pior cenário possível e mapear todas as políticas possíveis para a nuvem. Isso não é recomendável, mas é uma opção.

Por outro lado, a equipe poderia adotar uma abordagem de MVP e definir um ponto de partida e um conjunto de suposições que seriam verdadeiros para a maioria dos ativos.
A seguir, são apresentados alguns exemplos extremamente básicos:

* Todos os ativos correm o risco de serem encerrados (por erro, falha ou manutenção)
* Todos os ativos correm o risco de gerar gastos excessivos
* Todos os ativos podem ser comprometidos por senhas fracas
* Todos os ativos com todas as portas abertas expostas à Internet correm o risco de serem comprometidos

Os exemplos acima servem para estabelecer os riscos empresariais de MVP como uma teoria. A lista atual será exclusiva para todos os ambientes.
Depois que o MVP de Risco Empresarial for estabelecido, ele poderá ser convertido em [Políticas](overview.md) para mitigar cada risco.

<!-- markdownlint-enable MD026 -->

## <a name="incremental-risk-mitigation"></a>Mitigação de risco incremental

Assumindo que um MVP de risco empresarial é o ponto de partida, a governança pode amadurecer paralelamente à implantação planejada (em vez de crescer paralelamente ao crescimento dos negócios). Esse é um modelo muito mais estável para a maturidade da governança. Em cada iteração, novos ativos são replicados e preparados. Em cada liberação, as cargas de trabalho são preparadas para promoção de produção. O risco relativo certamente pode crescer com cada ciclo.

Quando a equipe de Governança de Nuvem opera em paralelo com as equipes de adoção de nuvem, o crescimento dos riscos empresariais também pode ser tratado. Cada ativo preparado pode ser facilmente classificado de acordo com o risco. Os documentos de classificação de dados podem ser compilados ou criados em paralelo à preparação. O perfil de risco e os pontos de exposição também podem ser documentados. Ao longo do tempo, uma visão extremamente clara dos riscos empresariais se tornará evidente em toda a organização.

A cada iteração, a equipe de Governança de Nuvem poderá trabalhar com a equipe de Estratégia de Nuvem para comunicar rapidamente sobre novos riscos, estratégias de mitigação, compensações e possíveis custos. Isso possibilitará que os participantes de negócios e líderes de TI estabeleçam parcerias para tomadas de decisão consolidadas e atualizadas. Essas decisões transmitem a maturidade da política. Quando necessário, as alterações da política produzem novos itens de trabalho para a maturidade dos principais sistemas de infraestrutura. Quando alterações nos sistemas preparados forem necessárias, as equipes de adoção de nuvem terão tempo suficiente para fazer alterações, enquanto a empresa testa os sistemas preparados e desenvolve um plano de adoção do usuário.

Essa abordagem minimiza os riscos, enquanto capacita a equipe a agir rapidamente. Adicionalmente, garante que os riscos sejam prontamente identificados e resolvidos antes da implantação.

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Avaliar tolerância de risco](./risk-tolerance.md)
