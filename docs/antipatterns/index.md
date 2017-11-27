---
title: "Antipadrões de desempenho para aplicativos em nuvem"
description: "Práticas comuns que podem causar problemas de escalabilidade."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 423fe6533e57268610f625f523714cd1bce89546
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="performance-antipatterns-for-cloud-applications"></a>Antipadrões de desempenho para aplicativos em nuvem

Um *antipadrão desempenho* é uma prática comum que pode causar problemas de escalabilidade quando um aplicativo está sob pressão. 

Aqui está um cenário comum: um aplicativo se comporta bem durante o teste de desempenho. Ele é liberado para produção e começa a lidar com cargas de trabalho reais. Nesse ponto, ele começa a executar mal &mdash;, rejeitando solicitações do usuário, atrasando ou lançando exceções. A equipe de desenvolvimento enfrenta duas perguntas:

- Por que não esse comportamento é exibido durante o teste?
- Como podemos corrigir isso?

A resposta à primeira pergunta é simples. É muito difícil em um ambiente de teste para simular usuários reais, seus padrões de comportamento e os volumes de trabalho, que eles podem executar. A única maneira totalmente garantida de compreender como um sistema se comporta sob carga é observá-lo em produção. Para ser claro, estamos não sugerindo que você deve ignorar o teste de desempenho. Testes de desempenho são cruciais para a obtenção de métricas de desempenho de linha de base. Mas você deve estar preparado para observar e corrigir problemas de desempenho quando eles ocorrem no sistema de produção.

A resposta à segunda pergunta, como corrigir o problema, é menos direto. Qualquer número de fatores pode contribuir e, às vezes, o problema só se manifesta em determinadas circunstâncias. Instrumentação e registro em log são fundamentais para localizar a causa raiz, mas você também precisa saber o que procurar. 

Com base em nossos contratos com clientes do Microsoft Azure, identificamos alguns dos problemas de desempenho mais comuns que os clientes veem em produção. Para cada antipadrão, descrevemos por que o antipadrão normalmente ocorre, sintomas de antipadrão e técnicas para resolver o problema. Nós também fornecemos o código de exemplo que ilustra o antipadrão e uma sugestão de solução. 

Esses antipadrões podem parecer óbvios quando você lê as descrições, mas eles ocorrem com mais frequência do que você imagina. Às vezes, um aplicativo herda um design que funcionou no local, mas não escala na nuvem. Ou um aplicativo pode iniciar com um design bem simples, mas à medida que novos recursos são adicionados, um ou mais desses antipadrões podem surgir. Independentemente, este guia ajudará você a identificar e corrigir esses antipadrões.

Aqui está a lista das antipadrões que nós identificamos: 

| Antipadrão | Descrição |
|-------------|-------------|
| [Banco de Dados Ocupado][BusyDatabase] | Descarregando muito processamento para um armazenamento de dados. |
| [Front-End Ocupado][BusyFrontEnd] | Movendo tarefas de uso intensivo de recursos em threads em segundo plano. |
| [E/S com ruídos][ChattyIO] | Continuamente enviar várias solicitações de rede pequena. |
| [Busca Incorreta][ExtraneousFetching] | Recuperando dados mais que o necessário, resultando em E/S desnecessária. |
| [Instanciação Inadequada][ImproperInstantiation] | Repetidamente, criação e destruição de objetos que são projetados para serem compartilhados e reutilizados. |
| [Persistência Monolítica][MonolithicPersistence] | Usando o mesmo armazenamento de dados para dados com padrões de uso muito diferentes. |
| [Sem cache][NoCaching] | Falha ao armazenar dados em cache. |
| [E/S Síncrona][SynchronousIO] | Bloquear o thread de chamada enquanto a E/S é concluída. | 

[BusyDatabase]: ./busy-database/index.md
[BusyFrontEnd]: ./busy-front-end/index.md
[ChattyIO]: ./chatty-io/index.md
[ExtraneousFetching]: ./extraneous-fetching/index.md
[ImproperInstantiation]: ./improper-instantiation/index.md
[MonolithicPersistence]: ./monolithic-persistence/index.md
[NoCaching]: ./no-caching/index.md
[SynchronousIO]: ./synchronous-io/index.md