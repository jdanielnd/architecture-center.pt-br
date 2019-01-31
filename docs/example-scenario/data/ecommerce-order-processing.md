---
title: Processamento de pedidos escalonável
titleSuffix: Azure Example Scenarios
description: Crie um pipeline de processamento de pedidos altamente escalonável usando o Azure Cosmos DB.
author: alexbuckgit
ms.date: 07/10/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
social_image_url: /azure/architecture/example-scenario/data/media/architecture-ecommerce-order-processing.png
ms.openlocfilehash: 3bb6e0998f2832bfdb20ba8b7bbf099cf6ac0423
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908385"
---
# <a name="scalable-order-processing-on-azure"></a>Processamento de pedidos escalonável no Azure

Esse cenário de exemplo é relevante para organizações que precisam de uma arquitetura altamente escalonável e resiliente para processamento de pedidos online. Os aplicativos potenciais incluem comércio eletrônico e ponto de venda de varejo, preenchimento de pedidos e reserva e acompanhamento de inventário.

Esse cenário possui uma abordagem de fornecimento de eventos, usando um modelo de programação funcional implementado por meio de microsserviços. Cada microsserviço é tratado como um processador de fluxo, e toda a lógica de negócios é implementada por meio de microsserviços. Essa abordagem permite alta disponibilidade e resiliência, replicação geográfica e desempenho rápido.

Usar serviços gerenciados do Azure, como o Cosmos DB e o HDInsight pode ajudar a reduzir os custos ao aproveitar a experiência da Microsoft em armazenamento e recuperação de dados distribuído globalmente na nuvem. Esse cenário trata especificamente de um cenário de varejo ou de comércio eletrônico; se você tiver outras necessidades de serviços de dados, você deve analisar a lista de [serviços de banco de dados inteligente totalmente gerenciados no Azure][product-category] disponíveis.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Outros casos de uso relevantes incluem:

- Sistemas de back-end de comércio eletrônico ou de ponto de venda de varejo.
- Sistemas de gerenciamento de inventário.
- Sistemas de atendimento de pedidos.
- Outros cenários de integração relevantes para um pipeline de processamento de pedidos.

## <a name="architecture"></a>Arquitetura

![Exemplo de arquitetura para um pipeline de processamento de pedidos escalonável][architecture]

Essa arquitetura fornece detalhes sobre os principais componentes de um pipeline de processamento de pedidos. O fluxo de dados neste cenário ocorre da seguinte forma:

1. As mensagens de evento entram no sistema por meio de aplicativos voltados ao cliente (de forma síncrona sobre HTTP) e vários sistemas de back-end (de forma assíncrona por meio do Apache Kafka). Essas mensagens são passadas para um pipeline de processamento do comando.
2. Cada mensagem de evento é ingerida e mapeada para um comando específico de um conjunto definido de comandos por um microsserviço do processador de comandos. O processador de comandos recupera qualquer estado atual relevante para executar o comando de um banco de dados de instantâneo de fluxo de eventos. O comando é então executado e a saída do comando é emitida como um novo evento.
3. Cada evento emitido como a saída de um comando está confirmado em um banco de dados de fluxo de eventos usando o Cosmos DB.
4. Para cada inserção de banco de dados ou atualização confirmada no banco de dados de fluxo de eventos, um evento é gerado pelo feed de alterações do Cosmos DB. Os sistemas downstream podem assinar qualquer tópico de evento que seja relevantes para esse sistema.
5. Todos os eventos do feed de alterações do Cosmos DB também são enviados a um microsserviço de fluxo de evento de instantâneo, que calcula as alterações de estado causadas por eventos que ocorreram. O novo estado é então confirmado no banco de dados de instantâneo de fluxo de eventos armazenado no Cosmos DB. O banco de dados de instantâneo fornece uma fonte de dados distribuída globalmente e com baixa latência para o estado atual de todos os elementos de dados. O banco de dados de fluxo de eventos fornece um registro completo de todas as mensagens de evento que passaram por meio da arquitetura, que permite cenários de recuperação de desastre, solução de problemas e testes robustos.

### <a name="components"></a>Componentes

- O [Cosmos DB](/azure/cosmos-db/introduction) é um serviço de banco de dados de vários modelos distribuído globalmente da Microsoft, que permite dimensionar soluções de forma elástica e independente a taxa de transferência e armazenamento em qualquer número de regiões geográficas. Ele oferece garantias de taxa de transferência, disponibilidade, latência e consistência com contratos de nível de serviço (SLAs) abrangentes. Esse cenário usa o Cosmos DB para os armazenamentos do fluxo de eventos e de instantâneos, e aproveita os [recursos do Feed de Alterações do Cosmos DB][docs-cosmos-db-change-feed] para fornecer consistência de dados e recuperação de falhas.
- [Apache Kafka no HDInsight](/azure/hdinsight/kafka/apache-kafka-introduction) é uma implementação de serviço gerenciado do Apache Kafka, uma plataforma de streaming distribuída de software livre para criar aplicativos e pipelines de dados de streaming em tempo real. O Kafka também fornece funcionalidade de agente de mensagem semelhante a uma fila de mensagens, para publicar e assinar os fluxos de dados nomeados. Esse cenário usa Kafka para processar a entrada, bem como eventos downstream no pipeline de processamento de pedidos.

## <a name="considerations"></a>Considerações

Muitas opções de tecnologia estão disponíveis para ingestão de mensagens em tempo real, armazenamento de dados, processamento de fluxo, armazenamento de dados analíticos, análise e relatórios. Para obter uma visão geral dessas opções, seus recursos e principais critérios de seleção, confira [Arquiteturas de Big Data: processamento em tempo real](/azure/architecture/data-guide/technology-choices/real-time-ingestion) no [Guia de Arquitetura de Dados do Azure](/azure/architecture/data-guide).

Microsserviços se tornaram um estilo popular de arquitetura para criar aplicativos de nuvem resilientes, altamente escalonáveis, implantáveis independentemente e capazes de evoluir rapidamente. Os microsserviços exigem uma abordagem diferente para o design e criação de aplicativos. Ferramentas como o Docker, Kubernetes, Azure Service Fabric e Nomad permitem o desenvolvimento de arquiteturas baseadas em microsserviços. Para obter orientação sobre a criação e execução de uma arquitetura baseada em microsserviços, consulte [Criando microsserviços no Azure](/azure/architecture/microservices) no Azure Architecture Center.

### <a name="availability"></a>Disponibilidade

A abordagem de fornecimento de eventos deste cenário permite que os componentes do sistema sejam livremente acoplados e implantados independentemente um do outro. O Cosmos DB oferece [alta disponibilidade][docs-cosmos-db-regional-failover] e ajuda a organização a gerenciar as compensações associadas à consistência, disponibilidade e desempenho, tudo com [garantias correspondentes][docs-cosmos-db-guarantees]. O Apache Kafka no HDInsight também foi projetado para [alta disponibilidade][docs-kafka-high-availability].

O Azure Monitor fornece interfaces de usuário unificadas para monitoramento entre os diferentes serviços do Azure. Para obter mais informações, confira [Monitoramento no Microsoft Azure](/azure/monitoring-and-diagnostics/monitoring-overview). Os Hubs de Eventos e o Stream Analytics estão integrados ao Azure Monitor.

Para obter outras considerações sobre disponibilidade, confira a [lista de verificação de disponibilidade][availability].

### <a name="scalability"></a>Escalabilidade

O Kafka no HDInsight permite a [configuração de armazenamento e a escalabilidade](/azure/hdinsight/kafka/apache-kafka-scalability) para clusters do Kafka. O Cosmos DB fornece um desempenho rápido e previsível e é [dimensionado de forma perfeita](/azure/cosmos-db/partition-data) à medida que o aplicativo cresce.
A arquitetura baseada em microsserviços de fornecimento de eventos desse cenário também facilita o dimensionamento do seu sistema e a expansão da sua funcionalidade.

Para outros considerações sobre escalabilidade, confira a [lista de verificação de escalabilidade][scalability] disponível no Architecture Center.

### <a name="security"></a>Segurança

O [modelo de segurança do Cosmos DB](/azure/cosmos-db/secure-access-to-data) autentica os usuários e fornece acesso a dados e recursos. Para obter mais informações, consulte a [segurança de banco de dados do Cosmos DB](/azure/cosmos-db/database-security).

Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].

### <a name="resiliency"></a>Resiliência

A arquitetura de fornecimento de eventos e as tecnologias associadas no cenário de exemplo tornam esse cenário altamente resiliente quando ocorrem falhas. Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].

## <a name="pricing"></a>Preços

Para explorar o custo da execução dessa cenário, todos os serviços são pré-configurados na calculadora de custos. Para ver como o preço seria alterado em seu cenário específico, altere as variáveis apropriadas de acordo com o volume de dados esperado. Para este cenário, o preço de exemplo inclui apenas o Cosmos DB e um cluster do Kafka para o processamento de eventos gerados a partir do feed de alterações do Cosmos DB. Processadores de evento e microsserviços para outros sistemas de downstream e sistemas de origem não são incluídos e seu custo é altamente dependente da quantidade e da escala desses serviços, bem como das tecnologias escolhidas para implementá-los.

A moeda do Azure Cosmos DB é a unidade de solicitação (RU). Com unidades de solicitação, você não precisa reservar capacidade de leitura/gravação ou provisão da CPU, memória e IOPS. Banco de dados do Azure Cosmos dá suporte a várias APIs que têm diferentes operações, desde simples lê e grava a consultas complexas de gráfico. Porque nem todas as solicitações forem iguais, as solicitações são atribuídas a uma quantidade normalizada de unidades de solicitação com base na quantidade de computação necessária para atender à solicitação. O número de unidades de solicitação necessários para sua solução depende do tamanho do elemento de dados e do número de banco de dados das operações de leitura e gravação por segundo. Para obter mais informações, confira [Unidades de solicitação no Azure Cosmos DB](/azure/cosmos-db/request-units). Esses preços estimados se baseiam no Cosmos DB em execução em duas regiões do Azure.

Fornecemos três perfis de custo de exemplo com base na quantidade de tráfego esperado:

- [Pequeno][small-pricing]: esse exemplo de preço refere-se a cinco RUs reservadas com um armazenamento de dados de 1 TB no Cosmos DB e um cluster Kafka pequeno (D3 v2).
- [Médio][medium-pricing]: esse exemplo de preço refere-se a 50 RUs reservadas com um armazenamento de dados de 10 TB no Cosmos DB e um cluster Kafka médio (D4 v2).
- [Grande][large-pricing]: esse exemplo de preço refere-se a 500 RUs reservadas com um armazenamento de dados de 30 TB no Cosmos DB e um cluster Kafka grande (D5 v2).

## <a name="related-resources"></a>Recursos relacionados

Este cenário de exemplo se baseia em uma versão mais extensa dessa arquitetura criada no [Jet.com](https://jet.com) para seu pipeline de processamento de pedidos de ponta a ponta. Para obter mais informações, consulte o [perfil de cliente técnico jet.com][source-document] e [apresentação do jet.com no Build 2018][source-presentation].

Outros recursos relacionados incluem:

- *[Designing Data-Intensive Applications](https://dataintensive.net)* por Martin Kleppmann (O'Reilly Media, 2017).
- *[Domain Modeling Made Functional: Tackle Software Complexity with Domain-Driven Design and F#](https://pragprog.com/book/swdddf/domain-modeling-made-functional)* por Scott Wlaschin (Pragmatic Programmers LLC, 2018).
- Outros [casos de uso do Cosmos DB][docs-cosmos-db-use-cases]
- [Arquitetura de processamento em tempo real](/azure/architecture/data-guide/big-data/real-time-processing), no [Guia de arquitetura de dados do Azure](/azure/architecture/data-guide).

<!-- links -->

[architecture]: ./media/architecture-ecommerce-order-processing.png
[product-category]: https://azure.microsoft.com/product-categories/databases/
[source-document]: https://customers.microsoft.com/story/jet-com-powers-innovative-e-commerce-engine-on-azure-in-less-than-12-months
[source-presentation]: https://channel9.msdn.com/events/Build/2018/BRK3602
[small-pricing]: https://azure.com/e/3d43949ffbb945a88cc0a126dc3a0e6e
[medium-pricing]: https://azure.com/e/1f1e7bf2a6ad4f7799581211f4369b9b
[large-pricing]: https://azure.com/e/75207172ece94cf6b5fb354a2252b333
[docs-cosmos-db-change-feed]: /azure/cosmos-db/change-feed
[docs-cosmos-db-regional-failover]: /azure/cosmos-db/regional-failover
[docs-cosmos-db-guarantees]: /azure/cosmos-db/distribute-data-globally#AvailabilityGuarantees
[docs-cosmos-db-use-cases]: /azure/cosmos-db/use-cases
[docs-kafka-high-availability]: /azure/hdinsight/kafka/apache-kafka-high-availability
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: /azure/architecture/patterns/category/resiliency/
[security]: /azure/security/
