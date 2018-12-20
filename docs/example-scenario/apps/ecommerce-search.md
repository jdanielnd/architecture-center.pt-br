---
title: Mecanismo de pesquisa de produto inteligente para comércio eletrônico
description: Fornece uma experiência de pesquisa de nível internacional em um aplicativo de comércio eletrônico.
author: jelledruyts
ms.date: 09/14/2018
ms.custom: fasttrack
ms.openlocfilehash: 5eabdb94b9345e73b21526681e0dbd6ae859d7be
ms.sourcegitcommit: a0e8d11543751d681953717f6e78173e597ae207
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/06/2018
ms.locfileid: "53004900"
---
# <a name="intelligent-product-search-engine-for-e-commerce"></a>Mecanismo de pesquisa de produto inteligente para comércio eletrônico

Esse cenário de exemplo mostra como o uso de um serviço de pesquisa dedicado pode aumentar drasticamente a relevância dos resultados da pesquisa para seus clientes de comércio eletrônico.

A pesquisa é o mecanismo principal por meio do qual os clientes encontram e, em última análise, compram produtos. Portanto, é essencial que os resultados da pesquisa sejam relevantes para a _intenção_ de consulta de pesquisa e que a experiência de pesquisa de ponta a ponta corresponda à dos gigantes de pesquisa, fornecendo resultados quase instantâneos, análise linguística, correspondência de localização geográfica, filtragem, facetas, preenchimento automático, realce de ocorrências, etc.

Imagine um aplicativo Web de comércio eletrônico típico, com dados de produtos armazenados em um banco de dados relacional como o SQL Server ou o Banco de Dados SQL. As consultas de pesquisa normalmente são tratadas no banco de dados usando consultas do `LIKE` ou recursos de [Pesquisa de Texto Completo][docs-sql-fts]. Usando o [Azure Search][docs-search] em vez disso, você libera seu banco de dados operacional do processamento de consultas e pode facilmente começar a tirar proveito de recursos de implementação difícil que fornecem aos clientes a melhor experiência de pesquisa possível. Além disso, como o Azure Search é um componente de PaaS (plataforma como serviço), você não precisa se preocupar em gerenciar a infraestrutura ou se tornar um especialista em pesquisa.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Outros casos de uso relevantes incluem:

* Localizar listagens de imóveis ou lojas perto da localização física do usuário.
* Pesquisar artigos em um site de notícias ou procurar resultados esportivos, com preferência mais elevada para obter informações mais _recentes_.
* Pesquisar em grandes repositórios de organizações _voltadas em documentos_ como criadores de políticas e notários.

Em última análise, _qualquer_ aplicativo que tenha alguma funcionalidade de pesquisa poderá se beneficiar de um serviço de pesquisa dedicado.

## <a name="architecture"></a>Arquitetura

![Visão geral da arquitetura dos componentes envolvidos em um mecanismo de pesquisa de produto inteligente para comércio eletrônico do Azure][architecture]

Este cenário aborda uma solução de comércio eletrônico em que os clientes podem pesquisar em um catálogo de produtos.
1. Os clientes navegam até o **aplicativo Web de comércio eletrônico** por meio de qualquer dispositivo.
2. O catálogo de produtos é mantido em um **Banco de Dados SQL** para processamento transacional.
3. O Azure Search usa um **indexador de pesquisa** para manter seu índice de pesquisa atualizado automaticamente por meio do controle de alterações integrado.
4. As consultas de pesquisa do cliente são descarregadas para o serviço **Azure Search**, que processa a consulta e retorna os resultados mais relevantes.
5. Como alternativa a uma experiência de pesquisa baseada na Web, os clientes também podem usar um **bot de conversa** em mídias sociais ou diretamente de assistentes digitais para pesquisar produtos e refinar a consulta de pesquisa e os resultados de forma incremental.
6. Opcionalmente, o recurso de **Pesquisa Cognitiva** pode ser usado para aplicar inteligência artificial e tornar o processamento ainda mais inteligente.

### <a name="components"></a>Componentes

* Os [Serviços de Aplicativos - Aplicativos Web][docs-webapps] hospedam os aplicativos Web, permitindo o dimensionamento automático e alta disponibilidade sem a necessidade de gerenciar a infraestrutura.
* O [Banco de Dados SQL][docs-sql-database] é um serviço gerenciado de banco de dados relacional de uso geral no Microsoft Azure que dá suporte a estruturas como XML, JSON, espacial e dados relacionais.
* O [Azure Search][docs-search] é uma solução de pesquisa como serviço na nuvem que oferece uma experiência de pesquisa avançada para conteúdo privado e heterogêneo em aplicativos Web, móveis e empresariais.
* O [Serviço de Bot][docs-botservice] fornece ferramentas para compilar, testar, implantar e gerenciar bots inteligentes.
* Os [Serviços Cognitivos][docs-cognitive] permitem usar algoritmos inteligentes para ver, ouvir, falar, entender e interpretar as necessidades do usuário por meio de métodos naturais de comunicação.

### <a name="alternatives"></a>Alternativas

* Você pode usar recursos de **pesquisa no banco de dados**, por exemplo, por meio da pesquisa de texto completo do SQL Server. Porém, nesse caso, o repositório transacional também processa consultas (aumentando a necessidade de capacidade de processamento) e os recursos de pesquisa no banco de dados são mais limitados.
* Você pode hospedar o software livre [Apache Lucene][apache-lucene] (no qual o Azure Search se baseia) em Máquinas Virtuais do Azure, mas, dessa forma, volta a gerenciar IaaS (infraestrutura como Serviço) e não se beneficia dos muitos recursos adicionais que o Azure Search oferece em comparação com o Lucene.
* Você também pode considerar a implantação de [Elastic Search][elastic-marketplace] do Azure Marketplace, que é um produto de pesquisa alternativo e eficaz de um fornecedor de terceiros, mas, também nesse caso, executa uma carga de trabalho de IaaS.

Outras opções para a camada de dados incluem:

* [Cosmos DB](/azure/cosmos-db/introduction) ‒ banco de dados multimodelo globalmente distribuído da Microsoft. O Cosmos DB fornece uma plataforma para executar outros modelos de dados, como Mongo DB, Cassandra, dados do gráfico ou armazenamento de tabela simples. O Azure Search também dá suporte à indexação de dados diretamente do Cosmos DB.

## <a name="considerations"></a>Considerações

### <a name="scalability"></a>Escalabilidade

A [camada de preço][search-tier] do serviço Azure Search não determina os recursos disponíveis, mas é usada principalmente para [planejamento de capacidade][search-capacity], pois define o armazenamento máximo que você obtém e quantas partições e réplicas pode provisionar. **Partições** permitem que você indexe mais documentos e obtenha taxas de transferência de gravação mais altas, enquanto **réplicas** fornecem mais QPS (Consultas por Segundo) e Alta Disponibilidade.

Você pode alterar dinamicamente o número de partições e réplicas, mas não é possível alterar a camada de preço. Portanto, você deve considerar cuidadosamente a camada certa para sua carga de trabalho de destino. Se precisar alterar a camada mesmo assim, você precisará provisionar um novo serviço lado a lado e recarregar os índices, podendo então apontar seus aplicativos no novo serviço.

### <a name="availability"></a>Disponibilidade

O Azure Search fornece um [SLA de 99,9% de disponibilidade][search-sla] para _leituras_ (ou seja, consultas) se você tem pelo menos duas réplicas e para _atualizações_(ou seja, atualização dos índices de pesquisa) se você tem pelo menos três réplicas. Portanto, você deverá provisionar pelo menos duas réplicas se quiser que seus clientes possam _pesquisar_ de maneira confiável e três se _alterações reais no índice_ também precisarem ser consideradas operações de alta disponibilidade.

Se houver necessidade de fazer alterações significativas no índice sem tempo de inatividade (por exemplo, alterar tipos de dados, excluir ou renomear campos), o índice precisará ser recriado. Assim como ocorre com a alteração da camada de serviço, isso significa criar um novo índice, populá-lo novamente com os dados e atualizando seus aplicativos para apontar para o novo índice.

### <a name="security"></a>Segurança

O Azure Search está em conformidade com muitas [normas de privacidade de dados e segurança][search-security], o que permite que seja usado na maioria dos setores.

Para proteger o acesso ao serviço, o Azure Search usa dois tipos de chaves: **chaves de administração**, que permitem que você execute _qualquer_ tarefa em relação ao serviço, e **chaves de consulta**, que só podem ser usadas para operações somente leitura, como consultas. Normalmente, o aplicativo que executa a pesquisa não atualiza o índice. Portanto, ele deve ser configurado apenas com uma chave de consulta, não com uma chave de administração (especialmente se a pesquisa é realizada de um dispositivo de usuário final, como um script em execução em um navegador da Web).

### <a name="search-relevance"></a>Relevância de pesquisa

O êxito de seu aplicativo de comércio eletrônico depende em grande parte da relevância dos resultados da pesquisa para seus clientes. Ajuste cuidadosamente o serviço de pesquisa para fornecer resultados ideais com base na pesquisa de usuário ou recorra a recursos internos, como [análise de tráfego de pesquisa][search-analysis], para entender os padrões de pesquisa do cliente e tomar decisões com base nos dados.

Maneiras comuns de ajustar o serviço de pesquisa:

* Usar [perfis de pontuação][search-scoring] para influenciar a relevância dos resultados da pesquisa, por exemplo, com base em qual campo corresponde à consulta, até que ponto os dados são recentes, a distância geográfica até o usuário etc.
* Usar [analisadores de idioma fornecidos pela Microsoft][search-languages], que usam uma pilha de NLP (Processamento de Linguagem Natural) avançada para interpretar melhor as consultas
* Usar [analisadores personalizados][search-analyzers] garantir que seus produtos sejam encontrados corretamente, sobretudo se você deseja pesquisar informações não baseadas em linguagem, como marca e modelo de um produto.

## <a name="deploy-this-scenario"></a>Implantar este cenário

Para implantar uma versão de comércio eletrônico mais completa deste cenário, você pode seguir este [tutorial passo a passo][end-to-end-walkthrough], que fornece um aplicativo .NET de exemplo que executa um aplicativo simples de compra de tíquetes. Ele também inclui o Azure Search e usa muitos dos recursos discutidos. Além disso, há um modelo do Resource Manager para automatizar a implantação da maioria dos recursos do Azure.

## <a name="pricing"></a>Preços

Para explorar o custo de executar esse cenário, todos os serviços mencionados acima são pré-configurados na calculadora de custos. Para ver como o preço seria alterado para o seu uso específico, altere as variáveis apropriadas para que sejam correspondentes ao uso esperado.

Fornecemos três perfis de custo de exemplo com base na quantidade de tráfego que você espera receber:

* [Pequeno][small-pricing]: neste perfil, usamos um único aplicativo Web `Standard S1` para hospedar o site, a camada gratuita do serviço de Bot do Azure, um único serviço do Azure Search `Basic` e um Banco de Dados SQL `Standard S2`.
* [Médio][medium-pricing]: dimensionamos o aplicativo Web para duas instâncias da camada `Standard S3`, atualizando o Serviço de Pesquisa para uma camada `Standard S1` e usando um Banco de Dados SQL `Standard S6`.
* [Grande][large-pricing]: no maior perfil, usamos quatro instâncias de um aplicativo Web `Premium P2V2`, atualizamos o serviço de Bot do Azure para a camada `Standard S1` (com 1.000.000 mensagens nos canais Premium), usamos duas unidades do serviço Azure Search `Standard S3` e um Banco de Dados SQL `Premium P6`.

## <a name="related-resources"></a>Recursos relacionados

Para saber mais sobre o Azure Search, acesse o [centro de documentação][docs-search], confira os [exemplos][search-samples] ou veja um [site de demonstração][search-demo] completo em ação.

<!-- links -->
[architecture]: ./media/architecture-ecommerce-search.png
[docs-sql-fts]: /sql/relational-databases/search/query-with-full-text-search
[docs-search]: /azure/search/search-what-is-azure-search
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[docs-botservice]: /azure/bot-service/
[docs-cognitive]: /azure/cognitive-services/
[apache-lucene]: https://lucene.apache.org/
[elastic-marketplace]: https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[search-sla]: https://go.microsoft.com/fwlink/?LinkId=716855
[search-tier]: /azure/search/search-sku-tier
[search-capacity]: /azure/search/search-capacity-planning
[search-security]: /azure/search/search-security-overview
[search-analysis]: /azure/search/search-traffic-analytics
[search-languages]: /rest/api/searchservice/language-support
[search-analyzers]: /rest/api/searchservice/custom-analyzers-in-azure-search
[search-scoring]: /rest/api/searchservice/add-scoring-profiles-to-a-search-index
[search-samples]: https://azure.microsoft.com/resources/samples/?service=search&sort=0
[search-demo]: https://azjobsdemo.azurewebsites.net/
[small-pricing]: https://azure.com/e/db2672a55b6b4d768ef0060a8d9759bd
[medium-pricing]: https://azure.com/e/a5ad0706c9e74add811e83ef83766a1c
[large-pricing]: https://azure.com/e/57f95a898daa487795bd305599973ee6
