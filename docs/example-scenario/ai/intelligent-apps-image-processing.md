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
# <a name="image-classification-for-insurance-claims-on-azure"></a><span data-ttu-id="4f583-103">Classificação de imagens para acionamento de seguro no Azure</span><span class="sxs-lookup"><span data-stu-id="4f583-103">Image classification for insurance claims on Azure</span></span>

<span data-ttu-id="4f583-104">Este cenário de exemplo é aplicável para empresas que precisam processar imagens.</span><span class="sxs-lookup"><span data-stu-id="4f583-104">This example scenario is applicable for businesses that need to process images.</span></span>

<span data-ttu-id="4f583-105">Entre os possíveis usos estão a classificação de imagens para um site de moda, a análise de texto e imagens em acionamento de seguro ou a compreensão de dados de telemetria em capturas de tela de jogos.</span><span class="sxs-lookup"><span data-stu-id="4f583-105">Potential applications include classifying images for a fashion website, analyzing text and images for insurance claims, or understanding telemetry data from game screenshots.</span></span> <span data-ttu-id="4f583-106">Tradicionalmente, as empresas precisariam desenvolver experiência em modelos de aprendizado de máquina, treinar os modelos e passar as imagens pelo processo personalizado para obter dados dessas imagens.</span><span class="sxs-lookup"><span data-stu-id="4f583-106">Traditionally, companies would need to develop expertise in machine learning models, train the models, and finally run the images through their custom process to get the data out of the images.</span></span>

<span data-ttu-id="4f583-107">Usando serviços do Azure como a API da Pesquisa Visual Computacional e o Azure Functions, as empresas podem eliminar a necessidade de gerenciar servidores individuais, reduzindo custos e aproveitando o conhecimento que a Microsoft já desenvolveu sobre processamento de imagens com os Serviços Cognitivos.</span><span class="sxs-lookup"><span data-stu-id="4f583-107">By using Azure services such as the Computer Vision API and Azure Functions, companies can eliminate the need to manage individual servers, while reducing costs and leveraging the expertise that Microsoft has already developed around processing images with Cognitive services.</span></span> <span data-ttu-id="4f583-108">Esse cenário de exemplo trata especificamente de um caso de uso de processamento de imagem.</span><span class="sxs-lookup"><span data-stu-id="4f583-108">This example scenario specifically addresses an image-processing use case.</span></span> <span data-ttu-id="4f583-109">Se você tiver necessidades diferentes para Inteligência Artificial, considere obter o conjunto completo dos [Serviços Cognitivos][cognitive-docs].</span><span class="sxs-lookup"><span data-stu-id="4f583-109">If you have different AI needs, consider the full suite of [Cognitive Services][cognitive-docs].</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="4f583-110">Casos de uso relacionados</span><span class="sxs-lookup"><span data-stu-id="4f583-110">Related use cases</span></span>

<span data-ttu-id="4f583-111">Considere este cenário para os casos de uso a seguir:</span><span class="sxs-lookup"><span data-stu-id="4f583-111">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="4f583-112">Classificar imagens em um site de moda.</span><span class="sxs-lookup"><span data-stu-id="4f583-112">Classify images on a fashion website.</span></span>
* <span data-ttu-id="4f583-113">Classificar dados de telemetria de capturas de tela de jogos.</span><span class="sxs-lookup"><span data-stu-id="4f583-113">Classify telemetry data from screenshots of games.</span></span>

## <a name="architecture"></a><span data-ttu-id="4f583-114">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="4f583-114">Architecture</span></span>

![Arquitetura de aplicativos inteligentes – pesquisa visual computacional][architecture-computer-vision]

<span data-ttu-id="4f583-116">Esse cenário aborda os componentes de back-end de um aplicativo Web ou móvel.</span><span class="sxs-lookup"><span data-stu-id="4f583-116">This scenario covers the back-end components of a web or mobile application.</span></span> <span data-ttu-id="4f583-117">O fluxo de dados deste cenário ocorre da seguinte forma:</span><span class="sxs-lookup"><span data-stu-id="4f583-117">Data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="4f583-118">O Azure Functions atua como a camada de API.</span><span class="sxs-lookup"><span data-stu-id="4f583-118">Azure Functions acts as the API layer.</span></span> <span data-ttu-id="4f583-119">Essas APIs permitem que o aplicativo carregue imagens e recupere dados do Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="4f583-119">These APIs enable the application to upload images and retrieve data from Cosmos DB.</span></span>

2. <span data-ttu-id="4f583-120">Quando uma imagem é carregada por meio de uma chamada à API, ela é armazenada no Armazenamento de blobs.</span><span class="sxs-lookup"><span data-stu-id="4f583-120">When an image is uploaded via an API call, it's stored in Blob storage.</span></span>

3. <span data-ttu-id="4f583-121">A adição de novos arquivos no Armazenamento de blobs dispara uma notificação EventGrid que é enviada a uma função do Azure.</span><span class="sxs-lookup"><span data-stu-id="4f583-121">Adding new files to Blob storage triggers an EventGrid notification to be sent to an Azure Function.</span></span>

4. <span data-ttu-id="4f583-122">O Azure Functions envia um link para o arquivo recém-carregado à API da Pesquisa Visual Computacional para análise.</span><span class="sxs-lookup"><span data-stu-id="4f583-122">Azure Functions sends a link to the newly uploaded file to the Computer Vision API to analyze.</span></span>

5. <span data-ttu-id="4f583-123">Depois que os dados são retornados da API da Pesquisa Visual Computacional, o Azure Functions cria uma entrada no Cosmos DB para persistir os resultados da análise junto com os metadados da imagem.</span><span class="sxs-lookup"><span data-stu-id="4f583-123">Once the data has been returned from the Computer Vision API, Azure Functions makes an entry in Cosmos DB to persist the results of the analysis alongside the image metadata.</span></span>

### <a name="components"></a><span data-ttu-id="4f583-124">Componentes</span><span class="sxs-lookup"><span data-stu-id="4f583-124">Components</span></span>

* <span data-ttu-id="4f583-125">A [API da Pesquisa Visual Computacional] [ computer-vision-docs] é parte do conjunto dos Serviços Cognitivos e é usada para recuperar informações sobre cada imagem.</span><span class="sxs-lookup"><span data-stu-id="4f583-125">[Computer Vision API][computer-vision-docs] is part of the Cognitive Services suite and is used to retrieve information about each image.</span></span>

* <span data-ttu-id="4f583-126">O [Azure Functions] [ functions-docs] fornece a API de back-end para o aplicativo Web, além do processamento de eventos para imagens carregadas.</span><span class="sxs-lookup"><span data-stu-id="4f583-126">[Azure Functions][functions-docs] provides the backend API for the web application, as well as the event processing for uploaded images.</span></span>

* <span data-ttu-id="4f583-127">A [Grade de Eventos] [ eventgrid-docs] dispara um evento quando uma nova imagem é carregada no armazenamento de blobs.</span><span class="sxs-lookup"><span data-stu-id="4f583-127">[Event Grid][eventgrid-docs] triggers an event when a new image is uploaded to blob storage.</span></span> <span data-ttu-id="4f583-128">A imagem é processada com o Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="4f583-128">The image is then processed with Azure functions.</span></span>

* <span data-ttu-id="4f583-129">O [Armazenamento de Blobs] [ storage-docs] armazena todos os arquivos de imagem que são carregados no aplicativo Web e todos os arquivos estáticos consumidos pelo aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="4f583-129">[Blob Storage][storage-docs] stores all of the image files that are uploaded into the web application, as well any static files that the web application consumes.</span></span>

* <span data-ttu-id="4f583-130">O [Cosmos DB] [ cosmos-docs] armazena metadados sobre cada imagem que é carregada, incluindo os resultados do processamento pela API da Pesquisa Visual Computacional.</span><span class="sxs-lookup"><span data-stu-id="4f583-130">[Cosmos DB][cosmos-docs] stores metadata about each image that is uploaded, including the results of the processing from Computer Vision API.</span></span>

## <a name="alternatives"></a><span data-ttu-id="4f583-131">Alternativas</span><span class="sxs-lookup"><span data-stu-id="4f583-131">Alternatives</span></span>

* <span data-ttu-id="4f583-132">[Serviço de Visão Personalizada][custom-vision-docs].</span><span class="sxs-lookup"><span data-stu-id="4f583-132">[Custom Vision Service][custom-vision-docs].</span></span> <span data-ttu-id="4f583-133">A API da Pesquisa Visual Computacional retorna um conjunto de [categorias baseadas em taxonomia][cv-categories].</span><span class="sxs-lookup"><span data-stu-id="4f583-133">The Computer Vision API returns a set of [taxonomy-based categories][cv-categories].</span></span> <span data-ttu-id="4f583-134">Se você precisar processar informações que não são retornadas pela API da Pesquisa Visual Computacional, considere o Serviço de Visão Personalizada, que permite a criação de classificadores de imagem personalizados.</span><span class="sxs-lookup"><span data-stu-id="4f583-134">If you need to process information that isn't returned by the Computer Vision API, consider the Custom Vision Service, which lets you build custom image classifiers.</span></span>

* <span data-ttu-id="4f583-135">[Azure Search][azure-search-docs].</span><span class="sxs-lookup"><span data-stu-id="4f583-135">[Azure Search][azure-search-docs].</span></span> <span data-ttu-id="4f583-136">Se seu caso de uso envolve a consulta dos metadados para localizar imagens que atendem a critérios específicos, considere o uso do Azure Search.</span><span class="sxs-lookup"><span data-stu-id="4f583-136">If your use case involves querying the metadata to find images that meet specific criteria, consider using Azure Search.</span></span> <span data-ttu-id="4f583-137">Atualmente em versão prévia, a [Pesquisa Cognitiva] [ cognitive-search] integra esse fluxo de trabalho sem interrupções.</span><span class="sxs-lookup"><span data-stu-id="4f583-137">Currently in preview, [Cognitive search][cognitive-search] seamlessly integrates this workflow.</span></span>

## <a name="considerations"></a><span data-ttu-id="4f583-138">Considerações</span><span class="sxs-lookup"><span data-stu-id="4f583-138">Considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="4f583-139">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="4f583-139">Scalability</span></span>

<span data-ttu-id="4f583-140">A maioria dos componentes usados neste cenário de exemplo são serviços gerenciados que serão dimensionados automaticamente.</span><span class="sxs-lookup"><span data-stu-id="4f583-140">The majority of the components used in this example scenario are managed services that will automatically scale.</span></span> <span data-ttu-id="4f583-141">Duas exceções notáveis: o Azure Functions tem um limite de um máximo de 200 instâncias.</span><span class="sxs-lookup"><span data-stu-id="4f583-141">A couple notable exceptions: Azure Functions has a limit of a maximum of 200 instances.</span></span> <span data-ttu-id="4f583-142">Se você precisar dimensionar além desse limite, considere usar várias regiões ou planos do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="4f583-142">If you need to scale beyond this limit, consider multiple regions or app plans.</span></span>

<span data-ttu-id="4f583-143">O Cosmos DB não faz o dimensionamento automático em termos de RUs (unidades de solicitação) provisionadas.</span><span class="sxs-lookup"><span data-stu-id="4f583-143">Cosmos DB doesn’t auto-scale in terms of provisioned request units (RUs).</span></span>  <span data-ttu-id="4f583-144">Para obter orientação sobre como estimar seus requisitos, confira as [unidades de solicitação] [ request-units] em nossa documentação.</span><span class="sxs-lookup"><span data-stu-id="4f583-144">For guidance on estimating your requirements see [request units][request-units] in our documentation.</span></span> <span data-ttu-id="4f583-145">Para usufruir do escalonamento integralmente no Cosmos DB, confira as [chaves de partição][partition-key].</span><span class="sxs-lookup"><span data-stu-id="4f583-145">To fully take advantage of the scaling in Cosmos DB, explore [partition keys][partition-key].</span></span>

<span data-ttu-id="4f583-146">Os bancos de dados NoSQL costumam trocar consistência (no sentido do Teorema CAP) por disponibilidade, escalabilidade e particionamento.</span><span class="sxs-lookup"><span data-stu-id="4f583-146">NoSQL databases frequently trade consistency (in the sense of the CAP theorem) for availability, scalability, and partitioning.</span></span>  <span data-ttu-id="4f583-147">Neste cenário de exemplo, um modelo de dados de chave-valor é usado e a consistência de transação raramente é necessária, pois a maioria das operações é atômica por definição.</span><span class="sxs-lookup"><span data-stu-id="4f583-147">In this example scenario, a key-value data model is used and transaction consistency is rarely needed as most operations are by definition atomic.</span></span> <span data-ttu-id="4f583-148">Outras orientações em relação a [Escolher o armazenamento de dados correto](../../guide/technology-choices/data-store-overview.md) estão disponíveis no Centro de Arquitetura do Azure.</span><span class="sxs-lookup"><span data-stu-id="4f583-148">Additional guidance to [Choose the right data store](../../guide/technology-choices/data-store-overview.md) is available in the Azure Architecture Center.</span></span>

<span data-ttu-id="4f583-149">Para obter diretrizes gerais sobre como criar soluções escalonáveis, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.</span><span class="sxs-lookup"><span data-stu-id="4f583-149">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="4f583-150">Segurança</span><span class="sxs-lookup"><span data-stu-id="4f583-150">Security</span></span>

<span data-ttu-id="4f583-151">A [MSI] [ msi] (Identidade de Serviço Gerenciada) é usada para fornecer acesso a outros recursos internos para sua conta e, em seguida, atribuída ao seu Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="4f583-151">[Managed service identities][msi] (MSI) are used to provide access to other resources internal to your account and then assigned to your Azure Functions.</span></span> <span data-ttu-id="4f583-152">Dê acesso apenas aos recursos necessários nessas identidades para fazer com que nada além seja exposto às funções (e, potencialmente, a seus clientes).</span><span class="sxs-lookup"><span data-stu-id="4f583-152">Only allow access to the requisite resources in those identities to ensure that nothing extra is exposed to your functions (and potentially to your customers).</span></span>  

<span data-ttu-id="4f583-153">Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].</span><span class="sxs-lookup"><span data-stu-id="4f583-153">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="4f583-154">Resiliência</span><span class="sxs-lookup"><span data-stu-id="4f583-154">Resiliency</span></span>

<span data-ttu-id="4f583-155">Todos os componentes neste cenário são gerenciados e, portanto, em um nível regional, todos são resilientes automaticamente.</span><span class="sxs-lookup"><span data-stu-id="4f583-155">All of the components in this scenario are managed, so at a regional level they are all resilient automatically.</span></span>

<span data-ttu-id="4f583-156">Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="4f583-156">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="4f583-157">Preços</span><span class="sxs-lookup"><span data-stu-id="4f583-157">Pricing</span></span>

<span data-ttu-id="4f583-158">Para explorar o custo de executar esse cenário, todos os serviços são pré-configurados na calculadora de custos.</span><span class="sxs-lookup"><span data-stu-id="4f583-158">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="4f583-159">Para ver como o preço seria alterado em seu caso de uso específico, altere as variáveis apropriadas de acordo com o tráfego esperado.</span><span class="sxs-lookup"><span data-stu-id="4f583-159">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="4f583-160">Fornecemos três perfis de custo de exemplo com base na quantidade de tráfego (assumimos que todas as imagens têm 100 KB de tamanho):</span><span class="sxs-lookup"><span data-stu-id="4f583-160">We have provided three sample cost profiles based on amount of traffic (we assume all images are 100 kb in size):</span></span>

* <span data-ttu-id="4f583-161">[Pequeno][pricing]: esse exemplo de preço refere-se ao processamento de &lt; 5 mil imagens por mês.</span><span class="sxs-lookup"><span data-stu-id="4f583-161">[Small][pricing]: this pricing example correlates to processing &lt; 5000 images a month.</span></span>
* <span data-ttu-id="4f583-162">[Médio][medium-pricing]: esse exemplo de preço refere-se ao processamento de 500 mil imagens por mês.</span><span class="sxs-lookup"><span data-stu-id="4f583-162">[Medium][medium-pricing]: this pricing example correlates to processing 500,000 images a month.</span></span>
* <span data-ttu-id="4f583-163">[Grande][large-pricing]: esse exemplo de preço refere-se ao processamento de 50 milhões de imagens por mês.</span><span class="sxs-lookup"><span data-stu-id="4f583-163">[Large][large-pricing]: this pricing example correlates to processing 50 million images a month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="4f583-164">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="4f583-164">Related Resources</span></span>

<span data-ttu-id="4f583-165">Para um roteiro de aprendizagem guiado deste cenário, consulte [Compilar um aplicativo Web sem servidor no Azure][serverless].</span><span class="sxs-lookup"><span data-stu-id="4f583-165">For a guided learning path of this scenario, see [Build a serverless web app in Azure][serverless].</span></span>  

<span data-ttu-id="4f583-166">Antes implantar esse cenário de exemplo em um ambiente de produção, revise as [melhores práticas][functions-best-practices] do Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="4f583-166">Before deploying this example scenario in a production environment, review the Azure Functions [best practices][functions-best-practices].</span></span>

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