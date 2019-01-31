---
title: Classificação de imagens para acionamento de seguro
titleSuffix: Azure Example Scenarios
description: Crie o processamento de imagens em seus aplicativos do Azure.
author: david-stanford
ms.date: 07/05/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-intelligent-apps-image-processing.png
ms.openlocfilehash: 03ef9d15ec9bf64dc743657e9d1e7a7e275c5a8e
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908317"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a>Classificação de imagens para acionamento de seguro no Azure

Esse cenário é relevante para empresas que precisam processar imagens.

Entre os possíveis usos estão a classificação de imagens para um site de moda, a análise de texto e imagens em acionamento de seguro ou a compreensão de dados de telemetria em capturas de tela de jogos. Tradicionalmente, as empresas precisariam desenvolver experiência em modelos de aprendizado de máquina, treinar os modelos e passar as imagens pelo processo personalizado para obter dados dessas imagens.

Usando serviços do Azure como a API da Pesquisa Visual Computacional e o Azure Functions, as empresas podem eliminar a necessidade de gerenciar servidores individuais, reduzindo custos e aproveitando o conhecimento que a Microsoft já desenvolveu sobre processamento de imagens com os Serviços Cognitivos. Esse cenário de exemplo trata especificamente de um caso de uso de processamento de imagem. Se você tiver necessidades diferentes para Inteligência Artificial, considere obter o conjunto completo dos [Serviços Cognitivos](/azure/#pivot=products&panel=ai).

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Outros casos de uso relevantes incluem:

- Classificar imagens em um site de moda.
- Classificar dados de telemetria de capturas de tela de jogos.

## <a name="architecture"></a>Arquitetura

![Arquitetura para a classificação de imagens][architecture]

Esse cenário aborda os componentes de back-end de um aplicativo Web ou móvel. O fluxo de dados deste cenário ocorre da seguinte forma:

1. A camada de API é criada usando o Azure Functions. Essas APIs permitem que o aplicativo carregue imagens e recupere dados do Cosmos DB.
2. Quando uma imagem é carregada por meio de uma chamada à API, ela é armazenada no Armazenamento de blobs.
3. A adição de novos arquivos no Armazenamento de Blobs dispara uma notificação de Grade de Eventos que é enviada a uma Azure Function.
4. O Azure Functions envia um link para o arquivo recém-carregado à API da Pesquisa Visual Computacional para análise.
5. Depois que os dados são retornados da API da Pesquisa Visual Computacional, o Azure Functions cria uma entrada no Cosmos DB para persistir os resultados da análise junto com os metadados da imagem.

### <a name="components"></a>Componentes

- A [API da Pesquisa Visual Computacional](/azure/cognitive-services/computer-vision/home) faz parte do conjunto dos Serviços Cognitivos e é usada para recuperar informações sobre cada imagem.
- O [Azure Functions](/azure/azure-functions/functions-overview) fornece a API de back-end para o aplicativo Web, além do processamento de eventos para imagens carregadas.
- A [Grade de Eventos](/azure/event-grid/overview) dispara um evento quando uma nova imagem é carregada no Armazenamento de Blobs. A imagem é processada com o Azure Functions.
- O [Armazenamento de Blobs](/azure/storage/blobs/storage-blobs-introduction) armazena todos os arquivos de imagem que são carregados no aplicativo Web e todos os arquivos estáticos consumidos pelo aplicativo Web.
- O [Cosmos DB](/azure/cosmos-db/introduction) armazena metadados sobre cada imagem que é carregada, incluindo os resultados do processamento pela API da Pesquisa Visual Computacional.

## <a name="alternatives"></a>Alternativas

- [Serviço de Visão Personalizada](/azure/cognitive-services/custom-vision-service/home). A API da Pesquisa Visual Computacional retorna um conjunto de [categorias baseadas em taxonomia][cv-categories]. Se você precisar processar informações que não são retornadas pela API da Pesquisa Visual Computacional, considere o Serviço de Visão Personalizada, que permite a criação de classificadores de imagem personalizados.
- [Azure Search](/azure/search/search-what-is-azure-search). Se seu caso de uso envolve a consulta dos metadados para localizar imagens que atendem a critérios específicos, considere o uso do Azure Search. Atualmente em versão prévia, a [Pesquisa Cognitiva](/azure/search/cognitive-search-concept-intro) integra esse fluxo de trabalho sem interrupções.

## <a name="considerations"></a>Considerações

### <a name="scalability"></a>Escalabilidade

A maioria dos componentes usados neste cenário de exemplo são serviços gerenciados que serão dimensionados automaticamente. Duas exceções notáveis: o Azure Functions tem um limite máximo de 200 instâncias. Se você precisar dimensionar além desse limite, considere usar várias regiões ou planos do aplicativo.

O Cosmos DB não faz o dimensionamento automático em termos de unidades de solicitação (RUs) provisionadas. Para obter orientação sobre como estimar seus requisitos, confira as [unidades de solicitação](/azure/cosmos-db/request-units) em nossa documentação. Para aproveitar integralmente a escala no Cosmos DB, entenda como as [chaves de partição](/azure/cosmos-db/partition-data) funcionam no Cosmos DB.

Os bancos de dados NoSQL costumam trocar consistência (no sentido do Teorema CAP) por disponibilidade, escalabilidade e particionamento. Neste cenário de exemplo, um modelo de dados de chave-valor é usado e a consistência de transação raramente é necessária, pois a maioria das operações é atômica por definição. Outras orientações em relação a [Escolher o armazenamento de dados correto](../../guide/technology-choices/data-store-overview.md) estão disponíveis no Centro de Arquitetura do Azure. Se sua implementação exige alta consistência, você pode [escolher seu nível de consistência](/azure/cosmos-db/consistency-levels) no Cosmos DB.

Para obter diretrizes gerais sobre como criar soluções escalonáveis, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.

### <a name="security"></a>Segurança

As [identidades de serviço gerenciadas para recursos do Azure][msi] são usadas para fornecer acesso a outros recursos internos para sua conta e, em seguida, atribuídas ao Azure Functions. Dê acesso apenas aos recursos necessários nessas identidades para fazer com que nada além seja exposto às funções (e, potencialmente, a seus clientes).

Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].

### <a name="resiliency"></a>Resiliência

Todos os componentes neste cenário são gerenciados e, portanto, em um nível regional, todos são resilientes automaticamente.

Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].

## <a name="pricing"></a>Preços

Para explorar o custo de executar esse cenário, todos os serviços são pré-configurados na calculadora de custos. Para ver como o preço seria alterado em seu caso de uso específico, altere as variáveis apropriadas de acordo com o tráfego esperado.

Fornecemos três perfis de custo de exemplo com base na quantidade de tráfego (assumimos que todas as imagens têm 100 KB de tamanho):

- [Pequeno][small-pricing]: esse exemplo de preço refere-se ao processamento de &lt; 5 mil imagens por mês.
- [Médio][medium-pricing]: esse exemplo de preço refere-se ao processamento de 500 mil imagens por mês.
- [Grande][large-pricing]: esse exemplo de preço refere-se ao processamento de 50 milhões de imagens por mês.

## <a name="related-resources"></a>Recursos relacionados

Para ver um roteiro de aprendizagem guiado, confira [Compilar um aplicativo Web sem servidor no Azure][serverless].

Antes de implantar este cenário de exemplo em um ambiente de produção, examine as práticas recomendadas para [otimizar o desempenho e a confiabilidade do Azure Functions][functions-best-practices].

<!-- links -->
[architecture]: ./media/architecture-intelligent-apps-image-processing.png
[small-pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
