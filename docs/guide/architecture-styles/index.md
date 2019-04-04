---
title: Estilos de arquitetura
titleSuffix: Azure Application Architecture Guide
description: Estilos comuns de arquitetura para aplicativos de nuvem.
layout: LandingPage
ms.topic: landing-page
ms.service: architecture-center
ms.subservice: reference-architecture
ms.date: 08/30/2018
ms.openlocfilehash: 6895ffc9c73dac29c27e7d8df68550c94f68b10a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58346528"
---
# <a name="architecture-styles"></a>Estilos de arquitetura

Um *estilo de arquitetura* é uma família de arquiteturas que compartilham características específicas. Por exemplo, [de N camadas][n-tier] é um estilo de arquitetura comum. Mais recentemente, as [arquiteturas de microsserviço][microservices] começaram a ganhar espaço. Estilos de arquitetura não exigem o uso de tecnologias específicas, mas algumas tecnologias são adequadas para determinadas arquiteturas. Por exemplo, os contêineres são uma opção natural para microsserviços.

Nós identificamos um conjunto de estilos de arquitetura que geralmente são encontrados em aplicativos de nuvem. O artigo para cada estilo inclui:

- Uma descrição e diagrama lógico do estilo.
- Recomendações para quando escolher o estilo.
- Benefícios, desafios e práticas recomendadas.
- Uma implantação recomendada usando os serviços do Azure relevantes.

## <a name="a-quick-tour-of-the-styles"></a>Um tour rápido dos estilos

Esta seção oferece um tour rápido dos estilos de arquitetura que nós identificamos, juntamente com algumas considerações de alto nível para seu uso. Leia mais detalhes nos tópicos com hiperlink.

### <a name="n-tier"></a>De N camadas

<!-- markdownlint-disable MD033 -->

<img src="./images/n-tier-sketch.svg" style="float:left; margin-top:6px;"/>

<!-- markdownlint-enable MD033 -->

**[De N camadas] [ n-tier]**  é uma arquitetura tradicional para aplicativos corporativos. As dependências são gerenciadas dividindo o aplicativo em *camadas* que executam funções lógicas, como o acesso a dados, lógica de negócios e apresentação. Uma camada somente pode chamar camadas que estão abaixo dela. No entanto, essa divisão em camadas horizontal pode ser um risco. Pode ser difícil introduzir alterações em uma parte do aplicativo sem tocar no resto do aplicativo. Isso torna as atualizações frequentes um desafio, limitando a rapidez com a qual novos recursos podem ser adicionados.

De N camadas é uma opção natural para migrar os aplicativos existentes que já usam uma arquitetura em camadas. Por esse motivo, de N camadas está normalmente presente em soluções de infraestrutura como serviço (IaaS), ou em aplicativos que usam uma combinação de IaaS e serviços gerenciados.

### <a name="web-queue-worker"></a>Trabalhador de fila da Web

<!-- markdownlint-disable MD033 -->

<img src="./images/web-queue-worker-sketch.svg" style="float:left; margin-top:6px;"/>

<!-- markdownlint-enable MD033 -->

Para uma solução de PaaS pura, considere uma arquitetura de **[trabalhador de fila da Web](./web-queue-worker.md)**. Neste estilo, o aplicativo possui um front-end de Web que trata as solicitações HTTP e um trabalhador de back-end que executa tarefas de uso intensivo de CPU ou de operações de longa duração. O front-end se comunica com o trabalhador por meio de uma fila de mensagens assíncronas.

O trabalhador de fila da Web é adequado para domínios relativamente simples com algumas tarefas com uso intensivo de recursos. Assim como a arquitetura de N-camadas, essa arquitetura é fácil de entender. O uso de serviços gerenciados simplifica a implantação e as operações. Mas, com domínios complexos, pode ser difícil gerenciar as dependências. O front-end e o trabalhador podem facilmente se tornar componentes grandes e monolíticos, que são difíceis de manter e atualizar. Assim como ocorre com a arquitetura de N camadas, isso pode reduzir a frequência de atualizações e limitar a inovação.

### <a name="microservices"></a>Microsserviços

<!-- markdownlint-disable MD033 -->

<img src="./images/microservices-sketch.svg" style="float:left; margin-top:6px;"/>

<!-- markdownlint-enable MD033 -->

Se o seu aplicativo tiver um domínio mais complexo, considere mudar para uma arquitetura de **[Microsserviços][microservices]**. Um aplicativo de microsserviço é composto de vários serviços pequenos e independentes. Cada serviço implementa um único recurso de negócios. Os serviços são flexíveis e se comunicam através de contratos de API.

Cada serviço pode ser criado por uma equipe pequena e dedicada. Serviços individuais podem ser implantados sem a necessidade de muita coordenação entre equipes, o que incentiva atualizações frequentes. Uma arquitetura de microsserviço é mais complexa de criar e gerenciar que as de N camadas ou de trabalhador de fila da Web. Ela requer mais experiência em desenvolvimento e cultura de DevOps. Mas feito corretamente, esse estilo pode levar a uma velocidade maior de distribuição, inovação mais rápida e arquitetura mais resiliente.

### <a name="cqrs"></a>CQRS

<!-- markdownlint-disable MD033 -->

<img src="./images/cqrs-sketch.svg" style="float:left; margin-top:6px;"/>

<!-- markdownlint-enable MD033 -->

O estilo **[CQRS](./cqrs.md)** (diferenciação de responsabilidade de comando e consulta) separa as operações de leitura e gravação em modelos separados. Isso isola as partes do sistema que atualizam dados das partes que leem os dados. Além disso, as leituras podem ser executadas em uma exibição materializada que é fisicamente separada do banco de dados de gravação. Isso permite dimensionar as cargas de trabalho de leitura e gravação de forma independente e otimizar a exibição materializada para consultas.

A CQRS faz mais sentido quando é aplicada a um subsistema de uma arquitetura maior. Em geral, você não deve impor esse estilo em todo o aplicativo, pois isso somente criará uma complexidade desnecessária. Considere-o para domínios colaborativos onde muitos usuários acessam os mesmos dados.

### <a name="event-driven-architecture"></a>Arquitetura orientada a eventos

<!-- markdownlint-disable MD033 -->

<img src="./images/event-driven-sketch.svg" style="float:left; margin-top:6px;"/>

**[Arquiteturas orientadas a eventos](./event-driven.md)** usam um modelo publicar-assinar (pub-sub), onde os produtores publicam eventos e os consumidores os assinam. Os produtores são independentes dos consumidores, e os consumidores são independentes uns dos outros.

Considere uma arquitetura orientada a eventos para aplicativos que ingerem e processam um grande volume de dados com latência muito baixa, tais como soluções de IoT. O estilo também é útil quando os diferentes subsistemas devem executar diferentes tipos de processamento nos mesmos dados de evento.

<br />

<!-- markdownlint-enable MD033 -->

### <a name="big-data-big-compute"></a>Big Data, Computação Intensa

**[Big Data](./big-data.md)** e  **[Computação Intensa](./big-compute.md)** são estilos de arquitetura especializados para cargas de trabalho que atendem a perfis específicos. Big data divide um conjunto de dados muito grande em partes, executando processamento paralelo em todo um conjunto, para análise e relatórios. Computação intensa, também chamada de computação de alto desempenho (HPC), faz cálculos paralelos em um número grande (milhares) de núcleos. Os domínios incluem renderização 3D, modelagem e simulações.

## <a name="architecture-styles-as-constraints"></a>Estilos de arquitetura como restrições

Um estilo de arquitetura impõe restrições de design, incluindo o conjunto de elementos que podem ser exibidos e as relações permitidas entre esses elementos. As restrições orientam a "forma" de uma arquitetura, restringindo o universo de opções. Quando uma arquitetura está em conformidade com as restrições de um determinado estilo, surgem determinadas propriedades desejáveis.

Por exemplo, as restrições em microsserviços incluem:

- Um serviço representa uma única responsabilidade.
- Cada serviço é independente dos outros.
- Os dados são particulares para o serviço ao qual eles pertencem. Os serviços não compartilham dados.

Ao aderir a essas restrições, o que surge é um sistema onde serviços podem ser implantados independentemente, as falhas são isoladas, atualizações frequentes são possíveis e é fácil de introduzir novas tecnologias no aplicativo.

Antes de escolher um estilo de arquitetura, certifique-se de que entendeu os princípios básicos e as restrições de estilo. Caso contrário, você pode acabar com um design que está de acordo com o estilo em um nível superficial, mas não consegue aproveitar todo o potencial do estilo. Também é importante ser pragmático. Às vezes, é melhor ceder em relação a uma restrição, em vez de insistir em uma arquitetura pura.

A tabela a seguir resume como cada estilo gerencia as dependências e os tipos de domínio que são mais adequados para cada um.

| Estilo de arquitetura | Gerenciamento de dependências | Tipo de domínio |
|--------------------|------------------------|-------------|
| De N camadas | Camadas horizontais divididas por sub-rede | Domínio de negócios tradicional. Frequência de atualizações baixa. |
| Trabalhador de fila da Web | Trabalhos de front-end e back-end, separados por mensagens assíncronas. | Domínio relativamente simple com algumas tarefas de uso intensivo de recursos. |
| Microsserviços | Serviços decompostos verticalmente (funcionalmente) que chamam uns aos outros por meio de APIs. | Domínio complicado. Atualizações frequentes. |
| CQRS | Diferenciação de leitura/gravação. Esquema e escala são otimizados separadamente. | Domínio colaborativo onde vários usuários acessam os mesmos dados. |
| Arquitetura orientada a eventos. | Produtor/consumidor. Modo de exibição independente do subsistema. | Sistemas em tempo real e IoT |
| Big data | Divida um conjunto de dados grande em partes pequenas. Processamento paralelo em conjuntos de dados locais. | Análise de dados em lotes e em tempo real. Análise preditiva usando ML. |
| Computação intensa| Alocação de dados para milhares de núcleos. | Domínios de computação intensiva tais como simulação. |

## <a name="consider-challenges-and-benefits"></a>Considere os desafios e benefícios

As restrições também criam desafios, portanto, é importante entender as vantagens e desvantagens da adoção de todos esses estilos. Os benefícios do estilo de arquitetura superam os desafios,  *para esse subdomínio e o contexto associado*.

Aqui estão alguns dos tipos de desafios a serem considerados ao selecionar um estilo de arquitetura:

- **Complexidade**. A complexidade da arquitetura justifica o seu domínio? Por outro lado, o estilo é muito simples para o seu domínio? Nesse caso, você corre o risco de acabar com um grande “[emaranhado de dados][ball-of-mud]”, porque essa arquitetura não ajuda a gerenciar dependências corretamente.

- **Sistema de mensagens assíncronas e consistência eventual**. O serviço de mensagens assíncronas pode ser usado para separar os serviços e aumentar a confiabilidade (pois as mensagens podem ser repetidas) e a escalabilidade. No entanto, isso também cria desafios, como semântica sempre uma vez (always-once) e consistência eventual.

- **Comunicação entre serviços**. Ao decompor um aplicativo em serviços separados, há um risco de que a comunicação entre serviços cause uma latência inaceitável ou crie um congestionamento de rede (por exemplo, em uma arquitetura de microsserviços).

- **Capacidade de gerenciamento**. É difícil gerenciar o aplicativo, monitorar, implantar atualizações e assim por diante?

[ball-of-mud]: https://en.wikipedia.org/wiki/Big_ball_of_mud
[microservices]: ./microservices.md
[n-tier]: ./n-tier.md
