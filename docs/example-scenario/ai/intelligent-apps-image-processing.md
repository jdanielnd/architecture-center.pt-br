---
title: Classificação de imagens para acionamento de seguro no Azure
description: Cenário comprovado para a criação de processamento de imagens em seus aplicativos do Azure.
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: 0ca0b46e83219afc5e22c2ac6467bf4be945c97a
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389155"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a>Classificação de imagens para acionamento de seguro no Azure

Este cenário de exemplo é aplicável para empresas que precisam processar imagens.

Entre os possíveis usos estão a classificação de imagens para um site de moda, a análise de texto e imagens em acionamento de seguro ou a compreensão de dados de telemetria em capturas de tela de jogos. Tradicionalmente, as empresas precisariam desenvolver experiência em modelos de aprendizado de máquina, treinar os modelos e passar as imagens pelo processo personalizado para obter dados dessas imagens.

Usando serviços do Azure como a API da Pesquisa Visual Computacional e o Azure Functions, as empresas podem eliminar a necessidade de gerenciar servidores individuais, reduzindo custos e aproveitando o conhecimento que a Microsoft já desenvolveu sobre processamento de imagens com os Serviços Cognitivos. Esse cenário de exemplo trata especificamente de um caso de uso de processamento de imagem. Se você tiver necessidades diferentes para Inteligência Artificial, considere obter o conjunto completo dos [Serviços Cognitivos][cognitive-docs].

## <a name="related-use-cases"></a>Casos de uso relacionados

Considere este cenário para os casos de uso a seguir:

* Classificar imagens em um site de moda.
* Classificar dados de telemetria de capturas de tela de jogos.

## <a name="architecture"></a>Arquitetura

![Arquitetura de aplicativos inteligentes – pesquisa visual computacional][architecture-computer-vision]

Esse cenário aborda os componentes de back-end de um aplicativo Web ou móvel. O fluxo de dados deste cenário ocorre da seguinte forma:

1. O Azure Functions atua como a camada de API. Essas APIs permitem que o aplicativo carregue imagens e recupere dados do Cosmos DB.

2. Quando uma imagem é carregada por meio de uma chamada à API, ela é armazenada no Armazenamento de blobs.

3. A adição de novos arquivos no Armazenamento de blobs dispara uma notificação EventGrid que é enviada a uma função do Azure.

4. O Azure Functions envia um link para o arquivo recém-carregado à API da Pesquisa Visual Computacional para análise.

5. Depois que os dados são retornados da API da Pesquisa Visual Computacional, o Azure Functions cria uma entrada no Cosmos DB para persistir os resultados da análise junto com os metadados da imagem.

### <a name="components"></a>Componentes

* A [API da Pesquisa Visual Computacional] [ computer-vision-docs] é parte do conjunto dos Serviços Cognitivos e é usada para recuperar informações sobre cada imagem.

* O [Azure Functions] [ functions-docs] fornece a API de back-end para o aplicativo Web, além do processamento de eventos para imagens carregadas.

* A [Grade de Eventos] [ eventgrid-docs] dispara um evento quando uma nova imagem é carregada no armazenamento de blobs. A imagem é processada com o Azure Functions.

* O [Armazenamento de Blobs] [ storage-docs] armazena todos os arquivos de imagem que são carregados no aplicativo Web e todos os arquivos estáticos consumidos pelo aplicativo Web.

* O [Cosmos DB] [ cosmos-docs] armazena metadados sobre cada imagem que é carregada, incluindo os resultados do processamento pela API da Pesquisa Visual Computacional.

## <a name="alternatives"></a>Alternativas

* [Serviço de Visão Personalizada][custom-vision-docs]. A API da Pesquisa Visual Computacional retorna um conjunto de [categorias baseadas em taxonomia][cv-categories]. Se você precisar processar informações que não são retornadas pela API da Pesquisa Visual Computacional, considere o Serviço de Visão Personalizada, que permite a criação de classificadores de imagem personalizados.

* [Azure Search][azure-search-docs]. Se seu caso de uso envolve a consulta dos metadados para localizar imagens que atendem a critérios específicos, considere o uso do Azure Search. Atualmente em versão prévia, a [Pesquisa Cognitiva] [ cognitive-search] integra esse fluxo de trabalho sem interrupções.

## <a name="considerations"></a>Considerações

### <a name="scalability"></a>Escalabilidade

A maioria dos componentes usados neste cenário de exemplo são serviços gerenciados que serão dimensionados automaticamente. Duas exceções notáveis: o Azure Functions tem um limite de um máximo de 200 instâncias. Se você precisar dimensionar além desse limite, considere usar várias regiões ou planos do aplicativo.

O Cosmos DB não faz o dimensionamento automático em termos de RUs (unidades de solicitação) provisionadas.  Para obter orientação sobre como estimar seus requisitos, confira as [unidades de solicitação] [ request-units] em nossa documentação. Para usufruir do escalonamento integralmente no Cosmos DB, confira as [chaves de partição][partition-key].

Os bancos de dados NoSQL costumam trocar consistência (no sentido do Teorema CAP) por disponibilidade, escalabilidade e particionamento.  Neste cenário de exemplo, um modelo de dados de chave-valor é usado e a consistência de transação raramente é necessária, pois a maioria das operações é atômica por definição. Outras orientações em relação a [Escolher o armazenamento de dados correto](../../guide/technology-choices/data-store-overview.md) estão disponíveis no Centro de Arquitetura do Azure.

Para obter diretrizes gerais sobre como criar soluções escalonáveis, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.

### <a name="security"></a>Segurança

A [MSI] [ msi] (Identidade de Serviço Gerenciada) é usada para fornecer acesso a outros recursos internos para sua conta e, em seguida, atribuída ao seu Azure Functions. Dê acesso apenas aos recursos necessários nessas identidades para fazer com que nada além seja exposto às funções (e, potencialmente, a seus clientes).  

Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].

### <a name="resiliency"></a>Resiliência

Todos os componentes neste cenário são gerenciados e, portanto, em um nível regional, todos são resilientes automaticamente.

Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].

## <a name="pricing"></a>Preços

Para explorar o custo de executar esse cenário, todos os serviços são pré-configurados na calculadora de custos. Para ver como o preço seria alterado em seu caso de uso específico, altere as variáveis apropriadas de acordo com o tráfego esperado.

Fornecemos três perfis de custo de exemplo com base na quantidade de tráfego (assumimos que todas as imagens têm 100 KB de tamanho):

* [Pequeno][pricing]: esse exemplo de preço refere-se ao processamento de &lt; 5 mil imagens por mês.
* [Médio][medium-pricing]: esse exemplo de preço refere-se ao processamento de 500 mil imagens por mês.
* [Grande][large-pricing]: esse exemplo de preço refere-se ao processamento de 50 milhões de imagens por mês.

## <a name="related-resources"></a>Recursos relacionados

Para um roteiro de aprendizagem guiado deste cenário, consulte [Compilar um aplicativo Web sem servidor no Azure][serverless].  

Antes implantar esse cenário de exemplo em um ambiente de produção, revise as [melhores práticas][functions-best-practices] do Azure Functions.

<!-- links -->
[pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[functions-docs]: /azure/azure-functions/
[computer-vision-docs]: /azure/cognitive-services/computer-vision/home
[storage-docs]: /azure/storage/
[azure-search-docs]: /azure/search/
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[architecture-computer-vision]: ./media/architecture-computer-vision.png
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cosmos-docs]: /azure/cosmos-db/
[eventgrid-docs]: /azure/event-grid/
[cognitive-docs]: /azure/#pivot=products&panel=ai
[custom-vision-docs]: /azure/cognitive-services/Custom-Vision-Service/home
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
[request-units]: /azure/cosmos-db/request-units
[partition-key]: /azure/cosmos-db/partition-data