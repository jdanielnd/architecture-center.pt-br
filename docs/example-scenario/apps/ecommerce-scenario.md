---
title: Front-end de comércio eletrônico no Azure
description: Hospede um site de comércio eletrônico no Azure.
author: masonch
ms.date: 7/13/18
ms.openlocfilehash: 7baaf4d2986a00ab72b60a540bcd9d864893b109
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610814"
---
# <a name="an-e-commerce-front-end-on-azure"></a>Um front-end de comércio eletrônico no Azure

Este exemplo de cenário orienta você para uma implementação de um front-end de comércio eletrônico usando ferramentas do Azure de plataforma como um serviço (PaaS). Muitos sites de comércio eletrônico enfrentam sazonalidade e variabilidade de tráfego ao longo do tempo. Quando a demanda dos seus produtos ou serviços aumenta, seja de forma previsível ou imprevisível, usar ferramentas PaaS permitirá lidar com mais cliente mais transações automaticamente. Além disso, este cenário aproveita a economia de nuvem pagando somente a capacidade que usar.

Este documento ajudará você a aprender sobre os vários componentes de PaaS do Azure e as considerações usadas para implantar um aplicativo de comércio eletrônico de exemplo, *Relecloud Concerts*, uma plataforma de emissão de tíquetes para shows.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Outros casos de uso relevantes incluem:

* Criar um aplicativo que precisa de uma escala elástica para lidar com picos de usuários em momentos diferentes.
* Criar um aplicativo que foi projetado para operar com alta disponibilidade em diferentes regiões do Azure em todo o mundo.

## <a name="architecture"></a>Arquitetura

![Arquitetura do cenário de exemplo para um aplicativo de comércio eletrônico][architecture]

Este cenário aborda tíquetes de compras de um site de comércio eletrônico, o fluxo de dados neste ocorre da seguinte maneira:

1. O Gerenciador de Tráfego do Azure encaminha uma solicitação do usuário para o site de comércio eletrônico hospedado no Serviço de Aplicativo do Azure.
2. A CDN do Azure serve imagens estáticas e conteúdo para o usuário.
3. O usuário entra no aplicativo por meio de um locatário do Azure Active Directory B2C.
4. O usuário procura por shows usando o Azure Search.
5. O site recebe detalhes do show usando o Banco de Dados SQL do Azure. 
6. O site se refere às imagens do tíquete adquirido no Armazenamento de Blobs.
7. Os resultados da consulta ao banco de dados são armazenados em cache no Cache Redis do Azure para um melhor desempenho.
8. O usuário envia pedidos de tíquete e avaliações do show que são colocados na fila.
9. O Azure Functions processa o pagamento da ordem e as avaliações do show.
10. Os serviços cognitivos fornecem uma análise da avaliação do show para determinar o sentimento (positivo ou negativo).
11. O Application Insights fornece métricas de desempenho para monitorar a integridade do aplicativo Web.

### <a name="components"></a>Componentes

* A [CDN do Azure][docs-cdn] entrega o conteúdo estático, armazenado em cache de locais próximos aos usuários para reduzir a latência.
* O [Gerenciador de Tráfego do Azure][docs-traffic-manager] controla a distribuição de tráfego do usuário para pontos de extremidade de serviço em diferentes regiões do Azure.
* Os [Serviços de Aplicativos - Aplicativos Web][docs-webapps] hospedam os aplicativos Web, permitindo o dimensionamento automático e alta disponibilidade sem a necessidade de gerenciar a infraestrutura.
* O [Azure Active Directory - B2C][docs-b2c] é um serviço de gerenciamento de identidade que permite personalizar e controlar como os clientes se inscrevem, entram e gerenciam seus perfis em seus respectivos aplicativos.
* As [Filas de Armazenamento][docs-storage-queues] armazenam um grande número de mensagens da fila que podem ser acessados por um aplicativo.
* As [Funções][docs-functions] são opções de computação sem servidor que permitem que aplicativos sejam executados sob demanda sem a necessidade de gerenciar a infraestrutura.
* Os [Serviços Cognitivos – Análise de Sentimento][docs-sentiment-analysis] utilizam APIs de machine learning e permitem aos desenvolvedores adicionar facilmente recursos inteligentes – como emoção e detecção de vídeo; reconhecimento facial, de fala e de visão, além de compreensão de fala e de idioma – a seus aplicativos.
* O [Azure Search][docs-search] é uma solução de pesquisa como serviço na nuvem que oferece uma experiência de pesquisa avançada para conteúdo privado e heterogêneo em aplicativos Web, móveis e empresariais.
* Os [Blobs de Armazenamento][docs-storage-blobs] são otimizado para armazenar grandes quantidades de dados não estruturados, como texto ou dados binários.
* O [Cache Redis][docs-redis-cache] melhora o desempenho e escalabilidade de sistemas que dependem de armazenamentos de dados de back-end, copiando temporariamente dados acessados com frequência para armazenamento rápido localizado perto do aplicativo.
* O [Banco de Dados SQL][docs-sql-database] é um serviço gerenciado de banco de dados relacional de uso geral no Microsoft Azure que dá suporte a estruturas como XML, JSON, espacial e dados relacionais.
* O [Application Insights][docs-application-insights] foi projetado para ajudar a aprimorar continuamente o desempenho e usabilidade pela detecção automática de anomalias de desempenho por meio de ferramentas de análise integradas para ajudar a entender o que os usuários fazem com um aplicativo.

### <a name="alternatives"></a>Alternativas

Muitas outras tecnologias estão disponíveis para a criação de um aplicativo voltado para o cliente com foco em comércio eletrônico em grande escala. Essas tecnologias abrangem o front-end do aplicativo, assim como a camada de dados.

Outras opções para a camada da Web e funções incluem:

* [Service Fabric][docs-service-fabric] - Uma plataforma voltada à criação de componentes distribuídos que se beneficiam da implantação e execução em um cluster com um alto grau de controle. O Service Fabric também pode ser usado para hospedar contêineres.
* [Serviço de Kubernetes do Azure][docs-kubernetes-service] – Uma plataforma para criar e implantar soluções baseadas em contêiner que podem ser usadas como uma implementação de uma arquitetura de microsserviços. Isso permite agilidade aos diferentes componentes do aplicativo para que eles sejam capazes de dimensionar de modo independente sob demanda.
* [Instâncias de Contêiner do Azure][docs-container-instances] - Uma maneira de implantar e executar rapidamente contêineres com um ciclo de vida curto. Os contêineres aqui são geralmente implantados para executar um trabalho de processamento rápido, como processar uma mensagem ou executar um cálculo, e, depois, são desprovisionados assim que forem concluídos.
* [Barramento de Serviço] [service-bus] pode ser usado no lugar da Fila de Armazenamento.

Outras opções para a camada de dados incluem:

* [Cosmos DB](/azure/cosmos-db/introduction): banco de dados multimodelo globalmente distribuído da Microsoft. Isso fornece uma plataforma para executar outros modelos de dados, como Mongo DB, Cassandra, dados do Graph ou armazenamento de tabela simples.

## <a name="considerations"></a>Considerações

### <a name="availability"></a>Disponibilidade

* Considere a possibilidade de aproveitar os [padrões de design comuns para disponibilidade][design-patterns-availability] ao criar seu aplicativo de nuvem.
* Analise as considerações de disponibilidade na [arquitetura de referência de aplicativo Web do Serviço de Aplicativo][app-service-reference-architecture] apropriada
* Para outras considerações sobre disponibilidade, confira a [lista de verificação de disponibilidade][availability] no Centro de Arquitetura do Azure.

### <a name="scalability"></a>Escalabilidade

* Ao criar um aplicativo de nuvem esteja atento a [padrões de design comuns para escalabilidade][design-patterns-scalability].
* Analise as considerações de escalabilidade na [arquitetura de referência de aplicativo Web do Serviço de Aplicativo][app-service-reference-architecture] apropriada
* Para outros tópicos de escalabilidade, confira a [lista de verificação de escalabilidade][scalability] disponível no Centro de Arquitetura do Azure.

### <a name="security"></a>Segurança

* Considere a possibilidade de aproveitar os [padrões de design comuns de segurança][design-patterns-security] quando apropriado.
* Analise as considerações de segurança na [arquitetura de referência de aplicativo Web do Serviço de Aplicativo][app-service-reference-architecture] apropriada.
* Considere seguir um processo de [ciclo de vida de desenvolvimento seguro][secure-development] para ajudar os desenvolvedores a criar requisitos de conformidade de segurança de software e de endereço mais seguros, reduzindo o custo de desenvolvimento.
* Analise a arquitetura de blueprint para [conformidade do PCI DSS do Azure][pci-dss-blueprint].

### <a name="resiliency"></a>Resiliência

* Considere a possibilidade de aproveitar o [padrão de disjuntor][ circuit-breaker] para fornecer um tratamento de erros normal caso uma parte do aplicativo não esteja disponível.
* Analise os [padrões de design comuns para resiliência][design-patterns-resiliency] e considere a implementação deles quando apropriado.
* Você pode encontrar várias [práticas recomendadas para o Serviço de Aplicativo][resiliency-app-service] no Centro de Arquitetura do Azure.
* Considere o uso [replicação geográfica][sql-geo-replication] ativa para a camada de dados e armazenamento com [redundância geográfica][storage-geo-redudancy] para imagens e filas.
* Para uma discussão mais detalhada sobre [resiliência][resiliency], consulte o artigo relevante no Centro de Arquitetura do Azure.

## <a name="deploy-the-scenario"></a>Implantar o cenário

Para implantar este cenário, você pode seguir este [tutorial passo a passo][end-to-end-walkthrough] que demonstra como implantar manualmente cada componente. Este tutorial também fornece um aplicativo de exemplo .NET que executa um aplicativo simples de compra de tíquetes. Além disso, há um modelo do Resource Manager para automatizar a implantação da maioria dos recursos do Azure.

## <a name="pricing"></a>Preços

Explore o custo de executar esse cenário, todos os serviços são pré-configurados na calculadora de custos. Para ver como o preço seria alterado para o seu uso específico altere as variáveis apropriadas para que eles sejam correspondentes ao tráfego esperada.

Fornecemos três perfis de custo de exemplo com base na quantidade de tráfego que você espera receber:

* [Pequeno][small-pricing]: esse exemplo de preço representa os componentes necessários para criar a saída de uma instância de nível mínimo de produção. Aqui, estamos supondo uma pequena quantidade de usuários, com um total de apenas alguns milhares por mês. O aplicativo está usando uma única instância de um aplicativo Web padrão que será suficiente para habilitar o dimensionamento automático. Cada um dos outros componentes são dimensionados para uma camada básica, que permite uma quantidade mínima de custo, mas ainda garante que haja suporte para SLA e capacidade suficiente para lidar com uma carga de trabalho de nível de produção.
* [Médio][medium-pricing]: esse exemplo de preço representa os componentes indicativos de uma implantação de tamanho moderado. Aqui, podemos estimar aproximadamente 100.000 usuários usando o sistema ao longo de um mês. O tráfego esperado é tratado em uma única instância de serviço de aplicativo com uma camada standard moderada. Além disso, camadas moderadas de serviços cognitivos e de pesquisa são adicionados à calculadora.
* [Grande][large-pricing]: esse exemplo de preço representa um aplicativo de alta escala, na ordem de milhões de usuários por mês para mover terabytes de dados. Nesse nível de alto desempenho de uso, são necessários aplicativos Web da camada premium implantados em várias regiões administrados pelo gerenciador de tráfego. Os dados consistem no seguinte: armazenamento, bancos de dados e CDN, estão configurados para terabytes de dados.

## <a name="related-resources"></a>Recursos relacionados

* [Arquitetura de referência para aplicativo Web de várias regiões][multi-region-web-app]
* [eShop no exemplo de referência de contêineres][microservices-ecommerce]

<!-- links -->
[architecture]: ./media/architecture-ecommerce-scenario.png
[small-pricing]: https://azure.com/e/90fbb6a661a04888a57322985f9b34ac
[medium-pricing]: https://azure.com/e/38d5d387e3234537b6859660db1c9973
[large-pricing]: https://azure.com/e/f07f99b6c3134803a14c9b43fcba3e2f
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[availability]: /azure/architecture/checklist/availability
[circuit-breaker]: /azure/architecture/patterns/circuit-breaker
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[docs-application-insights]: /azure/application-insights/app-insights-overview
[docs-b2c]: /azure/active-directory-b2c/active-directory-b2c-overview
[docs-cdn]: /azure/cdn/cdn-overview
[docs-container-instances]: /azure/container-instances/
[docs-kubernetes-service]: /azure/aks/
[docs-functions]: /azure/azure-functions/functions-overview
[docs-redis-cache]: /azure/redis-cache/cache-overview
[docs-search]: /azure/search/search-what-is-azure-search
[docs-service-fabric]: /azure/service-fabric/
[docs-sentiment-analysis]: /azure/cognitive-services/welcome
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-storage-blobs]: /azure/storage/blobs/storage-blobs-introduction
[docs-storage-queues]: /azure/storage/queues/storage-queues-introduction
[docs-traffic-manager]: /azure/traffic-manager/traffic-manager-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[microservices-ecommerce]: https://github.com/dotnet-architecture/eShopOnContainers
[multi-region-web-app]: /azure/architecture/reference-architectures/app-service-web-app/multi-region
[pci-dss-blueprint]: /azure/security/blueprints/payment-processing-blueprint
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[secure-development]: https://www.microsoft.com/SDL/process/design.aspx
[sql-geo-replication]: /azure/sql-database/sql-database-geo-replication-overview
[storage-geo-redudancy]: /azure/storage/common/storage-redundancy-grs
