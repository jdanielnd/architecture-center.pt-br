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
# <a name="mass-ingestion-and-analysis-of-news-feeds-on-azure"></a><span data-ttu-id="b62a9-103">Ingestão em massa e análise de feeds de notícias no Azure</span><span class="sxs-lookup"><span data-stu-id="b62a9-103">Mass ingestion and analysis of news feeds on Azure</span></span>

<span data-ttu-id="b62a9-104">Nesse cenário de exemplo descreve um pipeline para ingestão em massa e análise em tempo real de documentos usando feeds de notícias RSS públicos quase.</span><span class="sxs-lookup"><span data-stu-id="b62a9-104">This example scenario describes a pipeline for mass ingestion and near real-time analysis of documents using public RSS news feeds.</span></span>  <span data-ttu-id="b62a9-105">Ele usa os serviços Cognitivos do Azure para oferecem informações úteis, incluindo detecção de sentimento, reconhecimento facial e tradução de texto.</span><span class="sxs-lookup"><span data-stu-id="b62a9-105">It uses Azure Cognitive Services to offer useful insights including text translation, facial recognition, and sentiment detection.</span></span>

<span data-ttu-id="b62a9-106">Este cenário contém exemplos para [inglês][english], [russo][russian], e [alemão] [ german] news feeds, mas você pode estendê-lo facilmente outros RSS feeds.</span><span class="sxs-lookup"><span data-stu-id="b62a9-106">This scenario contains examples for [English][english], [Russian][russian], and [German][german] news feeds, but you can easily extend it to other RSS feeds.</span></span> <span data-ttu-id="b62a9-107">Para facilitar a implantação, a coleta de dados, processamento e análise baseiam-se totalmente em serviços do Azure.</span><span class="sxs-lookup"><span data-stu-id="b62a9-107">For ease of deployment, the data collection, processing, and analysis are based entirely on Azure services.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="b62a9-108">Casos de uso relevantes</span><span class="sxs-lookup"><span data-stu-id="b62a9-108">Relevant use cases</span></span>

<span data-ttu-id="b62a9-109">Embora esse cenário se baseia no processamento dos RSS feeds, é relevante para qualquer documento, um site ou um artigo em que você precisa:</span><span class="sxs-lookup"><span data-stu-id="b62a9-109">While this scenario is based on processing of RSS feeds, it's relevant to any document, website, or article where you would need to:</span></span>

* <span data-ttu-id="b62a9-110">Traduza o texto para o idioma de preferência.</span><span class="sxs-lookup"><span data-stu-id="b62a9-110">Translate any text to the language of choice.</span></span>
* <span data-ttu-id="b62a9-111">Localize frases-chave, entidades e o sentimento de usuário no conteúdo digital.</span><span class="sxs-lookup"><span data-stu-id="b62a9-111">Find key phrases, entities, and user sentiment in digital content.</span></span>
* <span data-ttu-id="b62a9-112">Detecte objetos, texto e marcos em imagens associadas a um artigo digital.</span><span class="sxs-lookup"><span data-stu-id="b62a9-112">Detect objects, text, and landmarks in images associated with a digital article.</span></span>
* <span data-ttu-id="b62a9-113">Detecte pessoas por seu gênero e idade em qualquer imagem associada ao conteúdo digital.</span><span class="sxs-lookup"><span data-stu-id="b62a9-113">Detect people by their gender and age in any image associated with digital content.</span></span>

## <a name="architecture"></a><span data-ttu-id="b62a9-114">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="b62a9-114">Architecture</span></span>

![][architecture]

<span data-ttu-id="b62a9-115">Os dados fluem pela solução da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="b62a9-115">The data flows through the solution as follows:</span></span>

1. <span data-ttu-id="b62a9-116">Um RSS feed de notícias atua como o gerador que obtém dados de um documento ou um artigo.</span><span class="sxs-lookup"><span data-stu-id="b62a9-116">An RSS news feed acts as the generator that obtains data from a document or article.</span></span> <span data-ttu-id="b62a9-117">Por exemplo, com um artigo, os dados geralmente incluem um título, um resumo do corpo original do item de notícias e, às vezes, as imagens.</span><span class="sxs-lookup"><span data-stu-id="b62a9-117">For example, with an article, data typically includes a title, a summary of the original body of the news item, and sometimes images.</span></span>

2. <span data-ttu-id="b62a9-118">Um processo de ingestão ou gerador insere o artigo e todas as imagens associadas em um Azure Cosmos DB [coleta][collection].</span><span class="sxs-lookup"><span data-stu-id="b62a9-118">A generator or ingestion process inserts the article and any associated images into an Azure Cosmos DB [Collection][collection].</span></span>

3. <span data-ttu-id="b62a9-119">Uma notificação dispara uma função de ingestão no Azure Functions que armazena o texto do artigo no cosmos DB e as imagens de artigo (se houver) no armazenamento de BLOBs do Azure.</span><span class="sxs-lookup"><span data-stu-id="b62a9-119">A notification triggers an ingest function in Azure Functions that stores the article text in CosmosDB and the article images (if any) in Azure Blob Storage.</span></span>  <span data-ttu-id="b62a9-120">O artigo é então passado para a próxima fila.</span><span class="sxs-lookup"><span data-stu-id="b62a9-120">The article is then passed to the next queue.</span></span>

4. <span data-ttu-id="b62a9-121">Uma função de conversão é disparada por eventos de fila.</span><span class="sxs-lookup"><span data-stu-id="b62a9-121">A translate function is triggered by the queue event.</span></span> <span data-ttu-id="b62a9-122">Ele usa o [traduzir texto API] [ translate-text] dos serviços Cognitivos do Azure para detectar o idioma, traduzir, se necessário e coletar o sentimento, frases-chave e entidades de corpo e o título.</span><span class="sxs-lookup"><span data-stu-id="b62a9-122">It uses the [Translate Text API][translate-text] of Azure Cognitive Services to detect the language, translate if necessary, and collect the sentiment, key phrases, and entities from the body and the title.</span></span> <span data-ttu-id="b62a9-123">Em seguida, ele passa o artigo para a próxima fila.</span><span class="sxs-lookup"><span data-stu-id="b62a9-123">Then it passes the article to the next queue.</span></span>

5. <span data-ttu-id="b62a9-124">Uma função de detecção é disparada do artigo na fila.</span><span class="sxs-lookup"><span data-stu-id="b62a9-124">A detect function is triggered from the queued article.</span></span> <span data-ttu-id="b62a9-125">Ele usa o [computacional] [ vision] serviço para detectar objetos, pontos de referência e as palavras escritas na imagem associada, em seguida, passa o artigo para a próxima fila.</span><span class="sxs-lookup"><span data-stu-id="b62a9-125">It uses the [Computer Vision][vision] service to detect objects, landmarks, and written words in the associated image, then passes the article to the next queue.</span></span>

6. <span data-ttu-id="b62a9-126">Uma função de detecção facial é disparada é disparado do artigo na fila.</span><span class="sxs-lookup"><span data-stu-id="b62a9-126">A face function is triggered is triggered from the queued article.</span></span> <span data-ttu-id="b62a9-127">Ele usa o [API de detecção facial do Azure] [ face] serviço detectar faces para sexo e idade em que a imagem associada, em seguida, passa o artigo para a próxima fila.</span><span class="sxs-lookup"><span data-stu-id="b62a9-127">It uses the [Azure Face API][face] service to detect faces for gender and age in the associated image, then passes the article to the next queue.</span></span>

7. <span data-ttu-id="b62a9-128">Quando todas as funções estiverem concluídas, a função de notificação é disparada.</span><span class="sxs-lookup"><span data-stu-id="b62a9-128">When all functions are complete, the notify function is triggered.</span></span> <span data-ttu-id="b62a9-129">Ele carrega os registros processados para o artigo e verifica para ver quaisquer resultados que você deseja.</span><span class="sxs-lookup"><span data-stu-id="b62a9-129">It loads the processed records for the article and scans them for any results you want.</span></span> <span data-ttu-id="b62a9-130">Se encontrado, o conteúdo será sinalizado e uma notificação é enviada para o sistema de sua escolha.</span><span class="sxs-lookup"><span data-stu-id="b62a9-130">If found, the content is flagged and a notification is sent to the system of your choice.</span></span>

<span data-ttu-id="b62a9-131">Em cada etapa de processamento, a função grava os resultados para o Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="b62a9-131">At each processing step, the function writes the results to Azure Cosmos DB.</span></span> <span data-ttu-id="b62a9-132">Por fim, os dados podem ser usados como desejado.</span><span class="sxs-lookup"><span data-stu-id="b62a9-132">Ultimately, the data can be used as desired.</span></span> <span data-ttu-id="b62a9-133">Por exemplo, você pode usá-lo para aprimorar os processos de negócios, localizar novos clientes ou identificar problemas de satisfação do cliente.</span><span class="sxs-lookup"><span data-stu-id="b62a9-133">For example, you can use it to enhance business processes, locate new customers, or identify customer satisfaction issues.</span></span>

### <a name="components"></a><span data-ttu-id="b62a9-134">Componentes</span><span class="sxs-lookup"><span data-stu-id="b62a9-134">Components</span></span>

<span data-ttu-id="b62a9-135">A seguinte lista de componentes do Azure é usada neste exemplo.</span><span class="sxs-lookup"><span data-stu-id="b62a9-135">The following list of Azure components is used in this example.</span></span>

* <span data-ttu-id="b62a9-136">[O armazenamento do Azure] [ storage] é usado para manter a imagem bruta e arquivos de vídeo associados a um artigo.</span><span class="sxs-lookup"><span data-stu-id="b62a9-136">[Azure Storage][storage] is used to hold raw image and video files associated with an article.</span></span> <span data-ttu-id="b62a9-137">Uma conta de armazenamento secundário é criada com o serviço de aplicativo do Azure e é usada para hospedar o código de função do Azure e os logs.</span><span class="sxs-lookup"><span data-stu-id="b62a9-137">A secondary storage account is created with Azure App Service and is used to host the Azure Function code and logs.</span></span>

* <span data-ttu-id="b62a9-138">[O Azure Cosmos DB] [ cosmos-db] contém texto, imagem e informações de controle de vídeo do artigo.</span><span class="sxs-lookup"><span data-stu-id="b62a9-138">[Azure Cosmos DB][cosmos-db] holds article text, image, and video tracking information.</span></span> <span data-ttu-id="b62a9-139">Os resultados das etapas dos serviços Cognitivos também são armazenados aqui.</span><span class="sxs-lookup"><span data-stu-id="b62a9-139">The results of the Cognitive Services steps are also stored here.</span></span>

* <span data-ttu-id="b62a9-140">[O Azure Functions] [ functions] executa o código de função usado para responder a mensagens da fila e transformar o conteúdo de entrada.</span><span class="sxs-lookup"><span data-stu-id="b62a9-140">[Azure Functions][functions] executes the function code used to respond to queue messages and transform the incoming content.</span></span> <span data-ttu-id="b62a9-141">[O serviço de aplicativo do Azure] [ aas] hospeda o código de função e processa os registros em série.</span><span class="sxs-lookup"><span data-stu-id="b62a9-141">[Azure App Service][aas] hosts the function code and processes the records serially.</span></span> <span data-ttu-id="b62a9-142">Este cenário inclui cinco funções: Ingestão, transformar, detectar o objeto, Face e notificar.</span><span class="sxs-lookup"><span data-stu-id="b62a9-142">This scenario includes five functions: Ingest, Transform, Detect Object, Face, and Notify.</span></span>

* <span data-ttu-id="b62a9-143">[O Azure Service Bus] [ service-bus] hospeda as filas do barramento de serviço do Azure usadas pelas funções.</span><span class="sxs-lookup"><span data-stu-id="b62a9-143">[Azure Service Bus][service-bus] hosts the Azure Service Bus queues used by the functions.</span></span>

* <span data-ttu-id="b62a9-144">[Serviços Cognitivos do Azure] [ acs] fornece o AI para o pipeline com base em implementações dos [computacional] [ vision] serviço, [Face API][face], e [traduzir texto] [ translate-text] serviço de tradução automática.</span><span class="sxs-lookup"><span data-stu-id="b62a9-144">[Azure Cognitive Services][acs] provides the AI for the pipeline based on implementations of the [Computer Vision][vision] service, [Face API][face], and [Translate Text][translate-text] machine translation service.</span></span>

* <span data-ttu-id="b62a9-145">[O Azure Application Insights] [ aai] fornece análises para ajudá-lo a diagnosticar problemas e entender a funcionalidade do seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b62a9-145">[Azure Application Insights][aai] provides analytics to help you diagnose issues and to understand functionality of your application.</span></span>

### <a name="alternatives"></a><span data-ttu-id="b62a9-146">Alternativas</span><span class="sxs-lookup"><span data-stu-id="b62a9-146">Alternatives</span></span>

* <span data-ttu-id="b62a9-147">Em vez de usar um padrão baseado em notificação de fila e o Azure Functions, use outro padrão para esse fluxo de dados.</span><span class="sxs-lookup"><span data-stu-id="b62a9-147">Instead of using a pattern based on queue notification and Azure Functions, use another pattern for this data flow.</span></span> <span data-ttu-id="b62a9-148">Por exemplo, [tópicos do barramento de serviço do Azure] [ topics] pode ser usado para processos de várias partes do artigo em paralelo em vez do processamento em série feito neste exemplo.</span><span class="sxs-lookup"><span data-stu-id="b62a9-148">For example, [Azure Service Bus Topics][topics] can be used to processes the various parts of the article in parallel as opposed to the serial processing done in this example.</span></span> <span data-ttu-id="b62a9-149">Para obter mais informações, compare [filas e tópicos][queues-topics].</span><span class="sxs-lookup"><span data-stu-id="b62a9-149">For more information, compare [queues and topics][queues-topics].</span></span>

* <span data-ttu-id="b62a9-150">Use [aplicativo lógico do Azure] [ logic-app] para implementar o código de função e, como bloqueio em nível de registro [Redlock] [ redlock] (necessária para paralela processamento até que o Azure Cosmos DB dá suporte à [atualizações do documento parcial][partial]).</span><span class="sxs-lookup"><span data-stu-id="b62a9-150">Use [Azure Logic App][logic-app] to implement the function code and implement record-level locking such as [Redlock][redlock] (needed for parallel processing until Azure Cosmos DB supports [partial document updates][partial]).</span></span> <span data-ttu-id="b62a9-151">Para obter mais informações, [comparar funções e aplicativos lógicos][compare].</span><span class="sxs-lookup"><span data-stu-id="b62a9-151">For more information, [compare Functions and Logic Apps][compare].</span></span>

* <span data-ttu-id="b62a9-152">Implemente essa arquitetura usando componentes de inteligência Artificial personalizados em vez da existente de serviços do Azure.</span><span class="sxs-lookup"><span data-stu-id="b62a9-152">Implement this architecture using customized AI components rather than existing Azure services.</span></span> <span data-ttu-id="b62a9-153">Por exemplo, estender o pipeline usando um modelo personalizado que detecta que algumas pessoas em uma imagem em vez da contagem de pessoas genérico, sexo e classifique os dados coletados neste exemplo.</span><span class="sxs-lookup"><span data-stu-id="b62a9-153">For example, extend the pipeline using a customized model that detects certain people in an image as opposed to the generic people count, gender, and age data collected in this example.</span></span> <span data-ttu-id="b62a9-154">Para usar o aprendizado de máquina personalizados ou modelos de IA com essa arquitetura, crie os modelos como pontos de extremidade RESTful para que eles podem ser chamados a partir do Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="b62a9-154">To use customized machine learning or AI models with this architecture, build the models as RESTful endpoints so they can be called from Azure Functions.</span></span>

* <span data-ttu-id="b62a9-155">Use um mecanismo de entrada diferente em vez de RSS feeds.</span><span class="sxs-lookup"><span data-stu-id="b62a9-155">Use a different input mechanism instead of RSS feeds.</span></span> <span data-ttu-id="b62a9-156">Use vários geradores ou processos de ingestão para alimentar o Azure Cosmos DB e o armazenamento do Azure.</span><span class="sxs-lookup"><span data-stu-id="b62a9-156">Use multiple generators or ingestion processes to feed Azure Cosmos DB and Azure Storage.</span></span>

## <a name="considerations"></a><span data-ttu-id="b62a9-157">Considerações</span><span class="sxs-lookup"><span data-stu-id="b62a9-157">Considerations</span></span>

<span data-ttu-id="b62a9-158">Para simplificar, este cenário de exemplo usa apenas algumas das APIs disponíveis e serviços, serviços Cognitivos do Azure.</span><span class="sxs-lookup"><span data-stu-id="b62a9-158">For simplicity, this example scenario uses only a few of the available APIs and services from Azure Cognitive Services.</span></span> <span data-ttu-id="b62a9-159">Por exemplo, texto em imagens pode ser analisado usando as [API de análise de texto][text-analytics].</span><span class="sxs-lookup"><span data-stu-id="b62a9-159">For example, text in images can be analyzed using the [Text Analytics API][text-analytics].</span></span> <span data-ttu-id="b62a9-160">Nesse cenário, o idioma de destino deve para estar em inglês, mas você pode alterar a entrada para qualquer [idioma com suporte] [ language] de sua escolha.</span><span class="sxs-lookup"><span data-stu-id="b62a9-160">The target language in this scenario is assumed to be English, but you can change the input to any [supported language][language] of your choice.</span></span>

### <a name="scalability"></a><span data-ttu-id="b62a9-161">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="b62a9-161">Scalability</span></span>

<span data-ttu-id="b62a9-162">O Azure Functions dimensionamento depende de [plano de hospedagem] [ plan] você usar.</span><span class="sxs-lookup"><span data-stu-id="b62a9-162">Azure Functions scaling depends on the [hosting plan][plan] you use.</span></span> <span data-ttu-id="b62a9-163">Esta solução pressupõe uma [plano de consumo][plan-c], no qual computação energia é alocada automaticamente para as funções quando necessário.</span><span class="sxs-lookup"><span data-stu-id="b62a9-163">This solution assumes a [Consumption plan][plan-c], in which compute power is automatically allocated to the functions when required.</span></span> <span data-ttu-id="b62a9-164">Você paga apenas quando suas funções são executadas.</span><span class="sxs-lookup"><span data-stu-id="b62a9-164">You pay only when your functions are running.</span></span> <span data-ttu-id="b62a9-165">Outra opção é usar um [serviço de aplicativo do Azure] [ plan-aas] plano, o que permite que você dimensione entre as camadas para alocar uma quantidade diferente de recursos.</span><span class="sxs-lookup"><span data-stu-id="b62a9-165">Another option is to use an [Azure App Service][plan-aas] plan, which allows you to scale between tiers to allocate a different amount of resources.</span></span>

<span data-ttu-id="b62a9-166">Com o Azure Cosmos DB, a chave é distribuir sua carga de trabalho aproximadamente uniformemente entre um número suficientemente grande de [chaves de partição][keys].</span><span class="sxs-lookup"><span data-stu-id="b62a9-166">With Azure Cosmos DB, the key is to distribute your workload roughly evenly among a sufficiently large number of [partition keys][keys].</span></span> <span data-ttu-id="b62a9-167">Não há nenhum limite para a quantidade total de dados que um contêiner pode armazenar ou para a quantidade total de [taxa de transferência] [ throughput] que pode dar suporte a um contêiner.</span><span class="sxs-lookup"><span data-stu-id="b62a9-167">There's no limit to the total amount of data that a container can store or to the total amount of [throughput][throughput] that a container can support.</span></span>

### <a name="management-and-logging"></a><span data-ttu-id="b62a9-168">Gerenciamento e registro em log</span><span class="sxs-lookup"><span data-stu-id="b62a9-168">Management and logging</span></span>

<span data-ttu-id="b62a9-169">Esta solução usa [Application Insights] [ aai] para coletar informações de registro em log e desempenho.</span><span class="sxs-lookup"><span data-stu-id="b62a9-169">This solution uses [Application Insights][aai] to collect performance and logging information.</span></span> <span data-ttu-id="b62a9-170">Uma instância do Application Insights é criada com a implantação no mesmo grupo de recursos como os outros serviços necessários para essa implantação.</span><span class="sxs-lookup"><span data-stu-id="b62a9-170">An instance of Application Insights is created with the deployment in the same resource group as the other services needed for this deployment.</span></span>

<span data-ttu-id="b62a9-171">Para exibir os logs gerados pela solução:</span><span class="sxs-lookup"><span data-stu-id="b62a9-171">To view the logs generated by the solution:</span></span>

1. <span data-ttu-id="b62a9-172">Vá para [portal do Azure] [ portal] e navegue até o grupo de recursos criado para a implantação.</span><span class="sxs-lookup"><span data-stu-id="b62a9-172">Go to [Azure portal][portal] and navigate to the resource group created for the deployment.</span></span>

2. <span data-ttu-id="b62a9-173">Clique o **Application Insights** instância.</span><span class="sxs-lookup"><span data-stu-id="b62a9-173">Click the **Application Insights** instance.</span></span>

3. <span data-ttu-id="b62a9-174">Dos **Application Insights** seção, navegue até **investigar\\pesquisa** e pesquisar os dados.</span><span class="sxs-lookup"><span data-stu-id="b62a9-174">From the **Application Insights** section, navigate to **Investigate\\Search** and search the data.</span></span>

### <a name="security"></a><span data-ttu-id="b62a9-175">Segurança</span><span class="sxs-lookup"><span data-stu-id="b62a9-175">Security</span></span>

<span data-ttu-id="b62a9-176">O Azure Cosmos DB usa uma conexão segura e a assinatura de acesso compartilhado por meio de C\# SDK fornecida pela Microsoft.</span><span class="sxs-lookup"><span data-stu-id="b62a9-176">Azure Cosmos DB uses a secured connection and shared access signature through the C\# SDK provided by Microsoft.</span></span> <span data-ttu-id="b62a9-177">Não há nenhum outras áreas de superfície voltado para o exterior.</span><span class="sxs-lookup"><span data-stu-id="b62a9-177">There are no other externally facing surface areas.</span></span> <span data-ttu-id="b62a9-178">Saiba mais sobre segurança [práticas recomendadas] [ db-practices] para o Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="b62a9-178">Learn more about security [best practices][db-practices] for Azure Cosmos DB.</span></span>

## <a name="pricing"></a><span data-ttu-id="b62a9-179">Preços</span><span class="sxs-lookup"><span data-stu-id="b62a9-179">Pricing</span></span>

<span data-ttu-id="b62a9-180">O custo diário estimado para manter essa implantação disponível é de aproximadamente \$20 nos EUA sem dados movendo por meio do sistema.</span><span class="sxs-lookup"><span data-stu-id="b62a9-180">The estimated daily cost to keep this deployment available is approximately \$20 U.S. with no data moving through the system.</span></span>

<span data-ttu-id="b62a9-181">O Azure Cosmos DB é eficiente, mas incorre no maior [custo] [ db-cost] nessa implantação.</span><span class="sxs-lookup"><span data-stu-id="b62a9-181">Azure Cosmos DB is powerful but incurs the greatest [cost][db-cost] in this deployment.</span></span> <span data-ttu-id="b62a9-182">Você pode usar outra solução de armazenamento, refatorar o código de funções do Azure fornecido.</span><span class="sxs-lookup"><span data-stu-id="b62a9-182">You can use another storage solution by refactoring the Azure Functions code provided.</span></span>

<span data-ttu-id="b62a9-183">Preços do Azure Functions varia dependendo de [plano] [ function-plan] ele é executado no.</span><span class="sxs-lookup"><span data-stu-id="b62a9-183">Pricing for Azure Functions varies depending on the [plan][function-plan] it runs in.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="b62a9-184">Implantar o cenário</span><span class="sxs-lookup"><span data-stu-id="b62a9-184">Deploy the scenario</span></span>

> [!NOTE]
> <span data-ttu-id="b62a9-185">Você deve ter uma conta do Azure já criada.</span><span class="sxs-lookup"><span data-stu-id="b62a9-185">You must have an existing Azure account.</span></span> <span data-ttu-id="b62a9-186">Se você não tiver uma assinatura do Azure, crie uma [conta gratuita][free] antes de começar.</span><span class="sxs-lookup"><span data-stu-id="b62a9-186">If you don't have an Azure subscription, create a [free account][free] before you begin.</span></span>

<span data-ttu-id="b62a9-187">Todo o código para esse cenário está disponível na [GitHub] [ github] repositório.</span><span class="sxs-lookup"><span data-stu-id="b62a9-187">All the code for this scenario is available in the [GitHub][github] repository.</span></span> <span data-ttu-id="b62a9-188">Esse repositório contém o código-fonte usado para compilar o aplicativo gerador que alimenta o pipeline para esta demonstração.</span><span class="sxs-lookup"><span data-stu-id="b62a9-188">This repository contains the source code used to build the generator application that feeds the pipeline for this demo.</span></span>

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
