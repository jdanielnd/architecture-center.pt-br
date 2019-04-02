---
title: Ingestão em massa e análise de feeds de notícias no Azure
description: Crie um pipeline para a ingestão e análise de texto, imagens, sentimento e outros dados de feeds de notícias RSS usando apenas os serviços do Azure, incluindo o Azure Cosmos DB e os Serviços Cognitivos do Azure.
author: njray
ms.date: 2/1/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/mass-ingestion-newsfeeds-architecture.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: e1bc636103753d474b545a6e3e9118b2ef302b34
ms.sourcegitcommit: ea97ac004c38c6b456794c1a8eef29f8d2b77d50
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/26/2019
ms.locfileid: "58489247"
---
# <a name="mass-ingestion-and-analysis-of-news-feeds-on-azure"></a>Ingestão em massa e análise de feeds de notícias no Azure

Nesse cenário de exemplo descreve um pipeline para ingestão em massa e análise em tempo real de documentos usando feeds de notícias RSS públicos quase.  Ele usa os serviços Cognitivos do Azure para oferecem informações úteis, incluindo detecção de sentimento, reconhecimento facial e tradução de texto.

Este cenário contém exemplos para [inglês][english], [russo][russian], e [alemão] [ german] news feeds, mas você pode estendê-lo facilmente outros RSS feeds. Para facilitar a implantação, a coleta de dados, processamento e análise baseiam-se totalmente em serviços do Azure.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Embora esse cenário se baseia no processamento dos RSS feeds, é relevante para qualquer documento, um site ou um artigo em que você precisa:

* Traduza o texto para o idioma de preferência.
* Localize frases-chave, entidades e o sentimento de usuário no conteúdo digital.
* Detecte objetos, texto e marcos em imagens associadas a um artigo digital.
* Detecte pessoas por seu gênero e idade em qualquer imagem associada ao conteúdo digital.

## <a name="architecture"></a>Arquitetura

![][architecture]

Os dados fluem pela solução da seguinte maneira:

1. Um RSS feed de notícias atua como o gerador que obtém dados de um documento ou um artigo. Por exemplo, com um artigo, os dados geralmente incluem um título, um resumo do corpo original do item de notícias e, às vezes, as imagens.

2. Um processo de ingestão ou gerador insere o artigo e todas as imagens associadas em um Azure Cosmos DB [coleta][collection].

3. Uma notificação dispara uma função de ingestão no Azure Functions que armazena o texto do artigo no cosmos DB e as imagens de artigo (se houver) no armazenamento de BLOBs do Azure.  O artigo é então passado para a próxima fila.

4. Uma função de conversão é disparada por eventos de fila. Ele usa o [traduzir texto API] [ translate-text] dos serviços Cognitivos do Azure para detectar o idioma, traduzir, se necessário e coletar o sentimento, frases-chave e entidades de corpo e o título. Em seguida, ele passa o artigo para a próxima fila.

5. Uma função de detecção é disparada do artigo na fila. Ele usa o [computacional] [ vision] serviço para detectar objetos, pontos de referência e as palavras escritas na imagem associada, em seguida, passa o artigo para a próxima fila.

6. Uma função de detecção facial é disparada é disparado do artigo na fila. Ele usa o [API de detecção facial do Azure] [ face] serviço detectar faces para sexo e idade em que a imagem associada, em seguida, passa o artigo para a próxima fila.

7. Quando todas as funções estiverem concluídas, a função de notificação é disparada. Ele carrega os registros processados para o artigo e verifica para ver quaisquer resultados que você deseja. Se encontrado, o conteúdo será sinalizado e uma notificação é enviada para o sistema de sua escolha.

Em cada etapa de processamento, a função grava os resultados para o Azure Cosmos DB. Por fim, os dados podem ser usados como desejado. Por exemplo, você pode usá-lo para aprimorar os processos de negócios, localizar novos clientes ou identificar problemas de satisfação do cliente.

### <a name="components"></a>Componentes

A seguinte lista de componentes do Azure é usada neste exemplo.

* [O armazenamento do Azure] [ storage] é usado para manter a imagem bruta e arquivos de vídeo associados a um artigo. Uma conta de armazenamento secundário é criada com o serviço de aplicativo do Azure e é usada para hospedar o código de função do Azure e os logs.

* [O Azure Cosmos DB] [ cosmos-db] contém texto, imagem e informações de controle de vídeo do artigo. Os resultados das etapas dos serviços Cognitivos também são armazenados aqui.

* [O Azure Functions] [ functions] executa o código de função usado para responder a mensagens da fila e transformar o conteúdo de entrada. [O serviço de aplicativo do Azure] [ aas] hospeda o código de função e processa os registros em série. Este cenário inclui cinco funções: Ingestão, transformar, detectar o objeto, Face e notificar.

* [O Azure Service Bus] [ service-bus] hospeda as filas do barramento de serviço do Azure usadas pelas funções.

* [Serviços Cognitivos do Azure] [ acs] fornece o AI para o pipeline com base em implementações dos [computacional] [ vision] serviço, [Face API][face], e [traduzir texto] [ translate-text] serviço de tradução automática.

* [O Azure Application Insights] [ aai] fornece análises para ajudá-lo a diagnosticar problemas e entender a funcionalidade do seu aplicativo.

### <a name="alternatives"></a>Alternativas

* Em vez de usar um padrão baseado em notificação de fila e o Azure Functions, use outro padrão para esse fluxo de dados. Por exemplo, [tópicos do barramento de serviço do Azure] [ topics] pode ser usado para processos de várias partes do artigo em paralelo em vez do processamento em série feito neste exemplo. Para obter mais informações, compare [filas e tópicos][queues-topics].

* Use [aplicativo lógico do Azure] [ logic-app] para implementar o código de função e, como bloqueio em nível de registro [Redlock] [ redlock] (necessária para paralela processamento até que o Azure Cosmos DB dá suporte à [atualizações do documento parcial][partial]). Para obter mais informações, [comparar funções e aplicativos lógicos][compare].

* Implemente essa arquitetura usando componentes de inteligência Artificial personalizados em vez da existente de serviços do Azure. Por exemplo, estender o pipeline usando um modelo personalizado que detecta que algumas pessoas em uma imagem em vez da contagem de pessoas genérico, sexo e classifique os dados coletados neste exemplo. Para usar o aprendizado de máquina personalizados ou modelos de IA com essa arquitetura, crie os modelos como pontos de extremidade RESTful para que eles podem ser chamados a partir do Azure Functions.

* Use um mecanismo de entrada diferente em vez de RSS feeds. Use vários geradores ou processos de ingestão para alimentar o Azure Cosmos DB e o armazenamento do Azure.

## <a name="considerations"></a>Considerações

Para simplificar, este cenário de exemplo usa apenas algumas das APIs disponíveis e serviços, serviços Cognitivos do Azure. Por exemplo, texto em imagens pode ser analisado usando as [API de análise de texto][text-analytics]. Nesse cenário, o idioma de destino deve para estar em inglês, mas você pode alterar a entrada para qualquer [idioma com suporte] [ language] de sua escolha.

### <a name="scalability"></a>Escalabilidade

O Azure Functions dimensionamento depende de [plano de hospedagem] [ plan] você usar. Esta solução pressupõe uma [plano de consumo][plan-c], no qual computação energia é alocada automaticamente para as funções quando necessário. Você paga apenas quando suas funções são executadas. Outra opção é usar um [serviço de aplicativo do Azure] [ plan-aas] plano, o que permite que você dimensione entre as camadas para alocar uma quantidade diferente de recursos.

Com o Azure Cosmos DB, a chave é distribuir sua carga de trabalho aproximadamente uniformemente entre um número suficientemente grande de [chaves de partição][keys]. Não há nenhum limite para a quantidade total de dados que um contêiner pode armazenar ou para a quantidade total de [taxa de transferência] [ throughput] que pode dar suporte a um contêiner.

### <a name="management-and-logging"></a>Gerenciamento e registro em log

Esta solução usa [Application Insights] [ aai] para coletar informações de registro em log e desempenho. Uma instância do Application Insights é criada com a implantação no mesmo grupo de recursos como os outros serviços necessários para essa implantação.

Para exibir os logs gerados pela solução:

1. Vá para [portal do Azure] [ portal] e navegue até o grupo de recursos criado para a implantação.

2. Clique o **Application Insights** instância.

3. Dos **Application Insights** seção, navegue até **investigar\\pesquisa** e pesquisar os dados.

### <a name="security"></a>Segurança

O Azure Cosmos DB usa uma conexão segura e a assinatura de acesso compartilhado por meio de C\# SDK fornecida pela Microsoft. Não há nenhum outras áreas de superfície voltado para o exterior. Saiba mais sobre segurança [práticas recomendadas] [ db-practices] para o Azure Cosmos DB.

## <a name="pricing"></a>Preços

O custo diário estimado para manter essa implantação disponível é de aproximadamente \$20 nos EUA sem dados movendo por meio do sistema.

O Azure Cosmos DB é eficiente, mas incorre no maior [custo] [ db-cost] nessa implantação. Você pode usar outra solução de armazenamento, refatorar o código de funções do Azure fornecido.

Preços do Azure Functions varia dependendo de [plano] [ function-plan] ele é executado no.

## <a name="deploy-the-scenario"></a>Implantar o cenário

> [!NOTE]
> Você deve ter uma conta do Azure já criada. Se você não tiver uma assinatura do Azure, crie uma [conta gratuita][free] antes de começar.

Todo o código para esse cenário está disponível na [GitHub] [ github] repositório. Esse repositório contém o código-fonte usado para compilar o aplicativo gerador que alimenta o pipeline para esta demonstração.

[architecture]: ./media/mass-ingestion-newsfeeds-architecture.png
[aai]: /azure/azure-monitor/app/app-insights-overview
[aas]: https://azure.microsoft.com/try/app-service/
[acs]: https://azure.microsoft.com/services/cognitive-services/directory/
[collection]: /rest/api/cosmos-db/collections
[compare]: /azure/azure-functions/functions-compare-logic-apps-ms-flow-webjobs#compare-azure-functions-and-azure-logic-apps
[cosmos-db]: /azure/cosmos-db/introduction
[db-cost]: https://azure.microsoft.com/pricing/details/cosmos-db/
[db-practices]: /azure/cosmos-db/database-security
[db-collection]: /azure/cosmos-db/databases-containers-items
[english]: https://www.nasa.gov/rss/dyn/breaking_news.rss
[face]: /azure/cognitive-services/face/overview
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[functions]: /azure/azure-functions/functions-overview
[function-plan]: /azure/azure-functions/functions-scale
[german]: http://www.bamf.de/SiteGlobals/Functions/RSS/DE/Feed/RSSNewsfeed_Meldungen
[github]: https://github.com/Azure/cognitive-services
[keys]: /azure/cosmos-db/partition-data
[language]: /azure/cognitive-services/translator/reference/v3-0-languages
[logic-app]: /azure/logic-apps/logic-apps-overview
[queues-topics]: /azure/service-bus-messaging/service-bus-queues-topics-subscriptions
[partial]: https://feedback.azure.com/forums/263030-azure-cosmos-db/suggestions/6693091-be-able-to-do-partial-updates-on-document
[plan]: /azure/azure-functions/functions-scale
[plan-aas]: /azure/azure-functions/functions-scale#app-service-plan
[plan-c]: /azure/azure-functions/functions-scale#consumption-plan
[portal]: http://portal.azure.com
[redlock]: https://redis.io/topics/distlock
[russian]: http://government.ru/all/rss/
[service-bus]: /azure/service-bus-messaging/
[storage]: /azure/storage/common/storage-account-overview 
[throughput]: /azure/cosmos-db/scaling-throughput
[topics]: /azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions
[text-analytics]: /azure/cognitive-services/text-analytics/
[translate-text]: /azure/cognitive-services/translator/translator-info-overview
[vision]: /azure/cognitive-services/computer-vision/home
