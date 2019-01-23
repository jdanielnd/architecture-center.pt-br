---
title: Projetar, criar e operar microsserviços no Azure com Kubernetes
description: Projetar, criar e operar microsserviços no Azure.
author: MikeWasson
ms.date: 10/23/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: ffd42679d0e04c2283cd78b8b03d7b5c695578be
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481942"
---
# <a name="designing-building-and-operating-microservices-on-azure"></a>Projetar, criar e operar microsserviços no Azure

![Diagrama de um serviço de entrega por drone](./images/drone.svg)

Microsserviços se tornaram um estilo popular de arquitetura para criar aplicativos de nuvem resilientes, altamente escalonáveis, implantáveis independentemente e capazes de evoluir rapidamente. No entanto, para serem mais do que apenas uma palavra de efeito, microsserviços exigem uma abordagem diferente para o design e criação de aplicativos.

Nesse conjunto de artigos, exploraremos como compilar e executar uma arquitetura de microsserviços no Azure. Os tópicos incluem:

- Usar o DDD (Design Controlado por Domínio) para criar uma arquitetura de microsserviços.
- Escolher as tecnologias do Azure certas para computação, armazenamento, mensagens e outros elementos de design.
- Noções básicas sobre padrões de design de microsserviços.
- Criar visando desempenho, escalabilidade e resiliência.
- Criar um pipeline de CI/CD.

Durante todo esse processo, nos concentraremos em um cenário de ponta a ponta: um serviço de entrega por drones que permite que os clientes agendem pacotes a serem selecionados e fornecidos por meio de drones. Você pode encontrar o código para nossa implementação de referência no GitHub

[Implementação de referência do ![GitHub](../_images/github.png)][drone-ri]

Mas primeiro, vamos começar com os conceitos básicos. O que são microsserviços e quais são os benefícios de adotar uma arquitetura de microsserviços?

<!-- markdownlint-disable MD026 -->

## <a name="why-build-microservices"></a>Por que criar microsserviços?

<!-- markdownlint-enable MD026 -->

Em um aplicativo de microsserviços, o aplicativo é composto de vários serviços pequenos e independentes. Aqui estão algumas características que definem microsserviços:

- Cada microsserviço implementa um único recurso de negócios.
- Um microsserviço é suficientemente pequeno para que uma única equipe pequena de desenvolvedores possa escrever o código dele e mantê-lo.
- Microsserviços são executados em processos separados, comunicando-se por meio de padrões de mensagens ou APIs bem definidas.
- Microsserviços não compartilham armazenamentos de dados nem esquemas de dados. Cada microsserviço é responsável por gerenciar seus próprios dados.
- Microsserviços têm bases de código separadas e não compartilham código-fonte. No entanto, eles podem usar bibliotecas de utilitários comuns.
- Cada microsserviço pode ser implantado e atualizado de forma independente de outros serviços.

Se utilizados corretamente, microsserviços podem fornecer vários benefícios úteis:

- **Agilidade.** Já que os microsserviços são implantados de forma independente, é mais fácil gerenciar correções de bugs e lançamentos de recursos. Você poderá atualizar um serviço sem reimplantar o aplicativo inteiro e, em seguida, reverter uma atualização se algo der errado. Em muitos aplicativos tradicionais, se um bug for encontrado em uma parte do aplicativo, ele poderá bloquear todo o processo de lançamento. Como resultado, novos recursos poderão ser mantidos em espera até uma correção de bug ser integrada, testada e publicada.

- **Código pequeno, equipes pequenas.** Um microsserviço deve ser pequeno o suficiente para que possa ser criado, testado e implantado por uma única equipe de recurso. Bases de código pequenas são mais fáceis de entender. Em um aplicativo monolítico grande, há uma tendência de que, ao longo do tempo, certas dependências de código fiquem entrelaçadas, fazendo com que, para adicionar um novo recurso, seja necessário alterar o código em vários lugares. Não compartilhando código nem armazenamentos de dados, uma arquitetura de microsserviços minimiza as dependências, o que torna mais fácil adicionar novos recursos. Tamanhos de equipe pequenos também promovem maior agilidade. A regra de "duas pizzas" diz que uma equipe deve ser suficientemente pequena para que duas pizzas possam alimentar a equipe. É claro que não se trata de uma métrica exata e que depende dos apetites dos membros da equipe! Mas o ponto é que grandes grupos costumam ser menos produtivos, porque a comunicação é mais lenta, a sobrecarga de gerenciamento aumenta e a agilidade diminui.

- **Combinação de tecnologias**. Equipes podem escolher a tecnologia mais adequada para seu serviço, usando uma combinação de pilhas de tecnologia conforme apropriado.

- **Resiliência**. Se um microsserviço individual se tornar não disponível, ele não interromperá o aplicativo inteiro, desde determinados microsserviços upstream sejam projetados para lidar com falhas corretamente (por exemplo, implementando a interrupção do circuito).

- **Escalabilidade**. Uma arquitetura de microsserviços permite que cada microsserviço seja dimensionado independentemente dos outros. Isso permite que você expanda subsistemas que exigem mais recursos, sem necessidade de expandir o aplicativo inteiro. Se você implantar serviços dentro de contêineres, você também poderá empacotar uma maior densidade de microsserviços em um único host, o que permitirá uma utilização de recursos mais eficiente.

- **Isolamento de dados**. É muito mais fácil executar atualizações de esquema, porque apenas um único microsserviço é afetado. Em um aplicativo monolítico, atualizações de esquema podem se tornar muito difíceis porque todas as diferentes partes do aplicativo podem tocar os mesmos dados, o que torna a realização de qualquer alteração no esquema algo arriscado.

## <a name="no-free-lunch"></a>Nenhum bônus

Esses benefícios têm um ônus. Esta série de artigos foi criada para tratar de alguns dos desafios na criação de microsserviços flexíveis, escalonáveis e gerenciáveis.

- **Limites de serviço**. Quando você compila microsserviços, você precisa pensar cuidadosamente sobre o local certo no qual desenhar os limites entre os serviços. Depois que os serviços são compilados e implantados em produção, pode ser difícil refatorar entre esses limites. Escolher os limites de serviços certos é um dos maiores desafios durante a criação de uma arquitetura de microsserviços. Qual deve ser o tamanho de cada serviço? Quando a funcionalidade deve ser incluída em vários serviços e quando ela deve ficar dentro do mesmo serviço? Este guia descreve uma abordagem que usa o design controlado por domínio para encontrar os limites de serviço. Ele começa com [análise de domínio](./domain-analysis.md) para localizar os contextos limitados e, em seguida, aplica um conjunto de [padrões de DDD tático](./microservice-boundaries.md) com base em requisitos funcionais e não funcionais.

- **Integridade e consistência de dados**. Um princípio básico dos microsserviços é que cada serviço gerencia seus próprios dados. Isso mantém os serviços separados, mas pode levar a desafios relacionados à integridade de dados ou à redundância. Podemos explorar alguns desses problemas nas [Considerações de dados](./data-considerations.md).

- **Latência e congestionamento de rede**. O uso de muitos serviços granulares pequenos pode resultar em mais comunicação entre serviços e maior latência de ponta a ponta. O capítulo [Comunicação entre serviços](./interservice-communication.md) descreve as considerações para mensagens entre serviços. Tanto a comunicação síncrona quanto a assíncrona têm um papel nas arquiteturas de microsserviços. Um bom [design de API](./api-design.md) é importante para que os serviços permaneçam acoplados e possam ser independentemente implantados e atualizados.

- **Complexidade**. Um aplicativo de microsserviço tem mais partes móveis. Cada serviço pode ser simples, mas os serviços precisam trabalhar juntos como um todo. Uma única operação de usuário pode envolver vários serviços. No capítulo [Ingestão e fluxo de trabalho](./ingestion-workflow.md), podemos examinar algumas questões sobre ingestão de solicitações com alta taxa de transferência, a coordenação de um fluxo de trabalho e o tratamento de falhas.

- **Comunicação entre clientes e o aplicativo.**  Quando você decompõe um aplicativo em vários serviços pequenos, como os clientes devem se comunicar com esses serviços? Um cliente deve chamar cada serviço diretamente ou rotear solicitações por meio de um [gateway de API](./gateway.md)?

- **Monitoramento**. Monitorar um aplicativo distribuído pode ser muito mais difícil que um aplicativo monolítico, porque você precisa correlacionar a telemetria de vários serviços. O capítulo [Monitoramento e registro em log](./logging-monitoring.md) aborda essas preocupações.

- **CI/CD (integração e entrega contínuas)**. Uma das principais metas dos microsserviços é a agilidade. Para fazer isso, você deve ter [CI/CD](./ci-cd.md) automatizada e robusta, para que você possa implantar serviços individuais em ambientes de teste e produção de forma rápida e confiável.

## <a name="the-drone-delivery-application"></a>O aplicativo de entrega por drone

Para explorar esses problemas e ilustrar algumas das práticas recomendadas para uma arquitetura de microsserviços, criamos uma implementação de referência que chamamos de aplicativo de Entrega por Drone. Você pode encontrar a implementação de referência no [GitHub][drone-ri].

A Fabrikam, Inc. está dando início a um serviço de entrega por drone. A empresa gerencia uma frota de aeronaves de tipo drone. Empresas registram-se no serviço e os usuários podem solicitar que um drone colete mercadorias para entrega. Quando um cliente agenda uma coleta, um sistema de back-end atribui um drone ao usuário e notifica-o com um tempo de entrega previsto. Enquanto a entrega está em andamento, o cliente pode acompanhar a localização do drone, com um ETA atualizado continuamente.

Este cenário envolve um domínio bastante complicado. Algumas das preocupações de negócios incluem o agendamento drones, o acompanhamento de pacotes, o gerenciamento de contas de usuário e o armazenamento e análise de dados históricos. Além disso, Fabrikam deseja se lançar no mercado rapidamente e, em seguida, iterar rapidamente, adicionando novas funcionalidades e capacidades. O aplicativo deve operar em escala de nuvem, com um SLO (objetivo do nível de serviço) elevado. A Fabrikam também espera que diferentes partes do sistema tenham requisitos muito diferentes de armazenamento e consulta de dados. Todas essas considerações levam Fabrikam para escolher uma arquitetura de microsserviços para o aplicativo de Entrega por Drone.

> [!NOTE]
> Para obter ajuda na escolha entre uma arquitetura de microsserviços e outros estilos de arquitetura, consulte o [Guia de arquitetura de aplicativo do Azure](../guide/index.md).

Nossa implementação de referência usa Kubernetes com o [AKS (Serviço de Kubernetes do Azure)](/azure/aks/). No entanto, muitas das decisões de arquitetura de alto nível e desafios se aplicarão a qualquer orquestrador de contêineres, incluindo o [Azure Service Fabric](/azure/service-fabric/).

> [!div class="nextstepaction"]
> [Análise de domínio](./domain-analysis.md)

<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation
