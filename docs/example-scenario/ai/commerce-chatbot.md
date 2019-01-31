---
title: Chatbot de conversação para reservas de hotel
titleSuffix: Azure Example Scenarios
description: Crie um chatbot de conversação para aplicativos de comércio com o Serviço de Bot do Azure.
author: iainfoulds
ms.date: 07/05/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-commerce-chatbot.png
ms.openlocfilehash: 48f85e7443bcd6149c8024d20fb50816c1a4df38
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908283"
---
# <a name="conversational-chatbot-for-hotel-reservations-on-azure"></a><span data-ttu-id="ea828-103">Chatbot de conversação para reservas de hotel no Azure</span><span class="sxs-lookup"><span data-stu-id="ea828-103">Conversational chatbot for hotel reservations on Azure</span></span>

<span data-ttu-id="ea828-104">Este cenário de exemplo é aplicável a empresas que precisam integrar um chatbot de conversação aos aplicativos.</span><span class="sxs-lookup"><span data-stu-id="ea828-104">This example scenario is applicable to businesses that need to integrate a conversational chatbot into applications.</span></span> <span data-ttu-id="ea828-105">Neste cenário, o chatbot de C# é usado para uma rede de hotéis que permite aos clientes verificar a disponibilidade e reservar acomodações através de um aplicativo Web ou móvel.</span><span class="sxs-lookup"><span data-stu-id="ea828-105">In this scenario, a C# chatbot is used for a hotel chain that allows customers to check availability and book accommodation through a web or mobile application.</span></span>

<span data-ttu-id="ea828-106">Os usos possíveis incluem fornecer aos clientes uma maneira de exibir a disponibilidade do hotel e reservar quartos, analisar um menu de restaurante que faz entrega e fazer pedidos ou pesquisar e pedir impressões de fotografias.</span><span class="sxs-lookup"><span data-stu-id="ea828-106">Potential uses include providing a way for customers to view hotel availability and book rooms, review a restaurant take-out menu and place a food order, or search for and order prints of photographs.</span></span> <span data-ttu-id="ea828-107">Tradicionalmente, as empresas precisariam contratar e treinar agentes de atendimento ao cliente para responder a essas solicitações, e os clientes teriam que esperar até que um representante estivesse disponível para prestar assistência.</span><span class="sxs-lookup"><span data-stu-id="ea828-107">Traditionally, businesses would need to hire and train customer service agents to respond to these customer requests, and customers would have to wait until a representative is available to provide assistance.</span></span>

<span data-ttu-id="ea828-108">Usando os serviços do Azure, como o Serviço de Bot e Reconhecimento vocal ou serviços da API de Reconhecimento Vocal, as empresas podem auxiliar os clientes e processar pedidos ou reservas com bots automatizados e escalonáveis.</span><span class="sxs-lookup"><span data-stu-id="ea828-108">By using Azure services such as the Bot Service and Language Understanding or Speech API services, companies can assist customers and process orders or reservations with automated, scalable bots.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="ea828-109">Casos de uso relevantes</span><span class="sxs-lookup"><span data-stu-id="ea828-109">Relevant use cases</span></span>

<span data-ttu-id="ea828-110">Outros casos de uso relevantes incluem:</span><span class="sxs-lookup"><span data-stu-id="ea828-110">Other relevant use cases include:</span></span>

- <span data-ttu-id="ea828-111">Exibir menu de entrega do restaurante e pedir comida</span><span class="sxs-lookup"><span data-stu-id="ea828-111">Viewing a restaurant take-out menu and ordering food</span></span>
- <span data-ttu-id="ea828-112">Verificar a disponibilidade do hotel e reservar um quarto</span><span class="sxs-lookup"><span data-stu-id="ea828-112">Checking hotel availability and reserving a room</span></span>
- <span data-ttu-id="ea828-113">Pesquisar fotos disponíveis e solicitar impressões</span><span class="sxs-lookup"><span data-stu-id="ea828-113">Searching available photos and ordering prints</span></span>

## <a name="architecture"></a><span data-ttu-id="ea828-114">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="ea828-114">Architecture</span></span>

![Visão geral da arquitetura dos componentes do Azure envolvidos em um chatbot de conversação][architecture]

<span data-ttu-id="ea828-116">Este cenário cobre um bot de conversação que funciona como um concierge de um hotel.</span><span class="sxs-lookup"><span data-stu-id="ea828-116">This scenario covers a conversational bot that functions as a concierge for a hotel.</span></span> <span data-ttu-id="ea828-117">O fluxo de dados neste cenário ocorre da seguinte forma:</span><span class="sxs-lookup"><span data-stu-id="ea828-117">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="ea828-118">O cliente acessa o chatbot em um dispositivo móvel ou aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="ea828-118">The customer accesses the chatbot with a mobile or web app.</span></span>
2. <span data-ttu-id="ea828-119">Usando o Azure Active Directory B2C, o usuário é autenticado.</span><span class="sxs-lookup"><span data-stu-id="ea828-119">Using Azure Active Directory B2C (Business 2 Customer), the user is authenticated.</span></span>
3. <span data-ttu-id="ea828-120">Interagindo com o Serviço de Bot, o usuário solicita informações sobre a disponibilidade do hotel.</span><span class="sxs-lookup"><span data-stu-id="ea828-120">Interacting with the Bot Service, the user requests information about hotel availability.</span></span>
4. <span data-ttu-id="ea828-121">Os Serviços Cognitivos processam a solicitação de linguagem natural para entender a comunicação do cliente.</span><span class="sxs-lookup"><span data-stu-id="ea828-121">Cognitive Services processes the natural language request to understand the customer communication.</span></span>
5. <span data-ttu-id="ea828-122">Depois que o usuário fica satisfeito com os resultados, o bot adiciona ou atualiza a reserva do cliente em um Banco de Dados SQL.</span><span class="sxs-lookup"><span data-stu-id="ea828-122">After the user is happy with the results, the bot adds or updates the customer’s reservation in a SQL Database.</span></span>
6. <span data-ttu-id="ea828-123">O Application Insights reúne telemetria do tempo de execução ao longo do processo para ajudar a equipe de DevOps em relação ao desempenho e o uso do bot.</span><span class="sxs-lookup"><span data-stu-id="ea828-123">Application Insights gathers runtime telemetry throughout the process to help the DevOps team with bot performance and usage.</span></span>

### <a name="components"></a><span data-ttu-id="ea828-124">Componentes</span><span class="sxs-lookup"><span data-stu-id="ea828-124">Components</span></span>

- <span data-ttu-id="ea828-125">O [Azure Active Directory][aad-docs] é o serviço de gerenciamento de identidades e diretório multilocatário baseado em nuvem da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="ea828-125">[Azure Active Directory][aad-docs] is Microsoft’s multi-tenant cloud-based directory and identity management service.</span></span> <span data-ttu-id="ea828-126">O Azure AD dá suporte a um conector de B2C, permitindo que você identifique pessoas que usam IDs externas, como Google, Facebook ou uma conta da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="ea828-126">Azure AD supports a B2C connector allowing you to identify individuals using external IDs such as Google, Facebook, or a Microsoft Account.</span></span>
- <span data-ttu-id="ea828-127">O [Serviço de Aplicativo][appservice-docs] permite criar e hospedar aplicativos Web na linguagem de programação de sua escolha, sem gerenciamento de infraestrutura.</span><span class="sxs-lookup"><span data-stu-id="ea828-127">[App Service][appservice-docs] enables you to build and host web applications in the programming language of your choice without managing infrastructure.</span></span>
- <span data-ttu-id="ea828-128">O [Serviço de Bot][botservice-docs] fornece ferramentas para compilar, testar, implantar e gerenciar bots inteligentes.</span><span class="sxs-lookup"><span data-stu-id="ea828-128">[Bot Service][botservice-docs] provides tools to build, test, deploy, and manage intelligent bots.</span></span>
- <span data-ttu-id="ea828-129">Os [Serviços Cognitivos][cognitive-docs] permitem usar algoritmos inteligentes para ver, ouvir, falar, entender e interpretar as necessidades do usuário por meio de métodos naturais de comunicação.</span><span class="sxs-lookup"><span data-stu-id="ea828-129">[Cognitive Services][cognitive-docs] lets you use intelligent algorithms to see, hear, speak, understand, and interpret your user needs through natural methods of communication.</span></span>
- <span data-ttu-id="ea828-130">O [Banco de Dados SQL] [ sqldatabase-docs] é um serviço de banco de dados relacional de nuvem totalmente gerenciado que oferece compatibilidade com o mecanismo do SQL Server.</span><span class="sxs-lookup"><span data-stu-id="ea828-130">[SQL Database][sqldatabase-docs] is a fully managed relational cloud database service that provides SQL Server engine compatibility.</span></span>
- <span data-ttu-id="ea828-131">O [Application Insights] [ appinsights-docs] é um serviço de APM (gerenciamento de desempenho de aplicativos) extensível que permite que você monitore o desempenho de aplicativos como o chatbot.</span><span class="sxs-lookup"><span data-stu-id="ea828-131">[Application Insights][appinsights-docs] is an extensible Application Performance Management (APM) service that lets you monitor the performance of applications, such as your chatbot.</span></span>

### <a name="alternatives"></a><span data-ttu-id="ea828-132">Alternativas</span><span class="sxs-lookup"><span data-stu-id="ea828-132">Alternatives</span></span>

- <span data-ttu-id="ea828-133">A [Speech API da Microsoft] [ speech-api] pode ser usada para alterar como os clientes interagem com o bot.</span><span class="sxs-lookup"><span data-stu-id="ea828-133">[Microsoft Speech API][speech-api] can be used to change how customers interface with your bot.</span></span>
- <span data-ttu-id="ea828-134">O [QnA Maker] [ qna-maker] pode ser usado para adicionar conhecimento ao bot rapidamente a partir de conteúdo semiestruturado, como o de perguntas frequentes.</span><span class="sxs-lookup"><span data-stu-id="ea828-134">[QnA Maker][qna-maker] can be used as to quickly add knowledge to your bot from semi-structured content like an FAQ.</span></span>
- <span data-ttu-id="ea828-135">[Tradução de Texto] [ translator] é um serviço que você pode considerar para adicionar suporte multilíngue ao bot facilmente.</span><span class="sxs-lookup"><span data-stu-id="ea828-135">[Translator Text][translator] is a service that you might consider to easily add multi-lingual support to your bot.</span></span>

## <a name="considerations"></a><span data-ttu-id="ea828-136">Considerações</span><span class="sxs-lookup"><span data-stu-id="ea828-136">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="ea828-137">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="ea828-137">Availability</span></span>

<span data-ttu-id="ea828-138">Este cenário usa o Banco de Dados SQL do Azure para armazenar as reservas dos clientes.</span><span class="sxs-lookup"><span data-stu-id="ea828-138">This scenario uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="ea828-139">O Banco de Dados SQL inclui bancos de dados com redundância de zona, grupos de failover e replicação geográfica.</span><span class="sxs-lookup"><span data-stu-id="ea828-139">SQL Database includes zone redundant databases, failover groups, and geo-replication.</span></span> <span data-ttu-id="ea828-140">Para obter mais informações, confira [Recursos de disponibilidade do Banco de Dados SQL][sqlavailability-docs].</span><span class="sxs-lookup"><span data-stu-id="ea828-140">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="ea828-141">Para ver outros tópicos sobre disponibilidade, consulte a [lista de verificação de disponibilidade][availability] no Azure Architecture Center.</span><span class="sxs-lookup"><span data-stu-id="ea828-141">For other availability topics, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="ea828-142">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="ea828-142">Scalability</span></span>

<span data-ttu-id="ea828-143">Este cenário usa o Serviço de Aplicativo do Azure.</span><span class="sxs-lookup"><span data-stu-id="ea828-143">This scenario uses Azure App Service.</span></span> <span data-ttu-id="ea828-144">Com o Serviço de Aplicativo, você pode dimensionar automaticamente o número de instâncias que executam o bot.</span><span class="sxs-lookup"><span data-stu-id="ea828-144">With App Service, you can automatically scale the number of instances that run your bot.</span></span> <span data-ttu-id="ea828-145">Essa funcionalidade permite acompanhar a demanda dos clientes em relação ao aplicativo Web e ao chatbot.</span><span class="sxs-lookup"><span data-stu-id="ea828-145">This functionality lets you keep up with customer demand for your web application and chatbot.</span></span> <span data-ttu-id="ea828-146">Para obter mais informações sobre o dimensionamento automático, confira [Melhores práticas de dimensionamento automático][autoscaling] no Centro de Arquitetura do Azure.</span><span class="sxs-lookup"><span data-stu-id="ea828-146">For more information on autoscale, see [Autoscaling best practices][autoscaling] in the Azure Architecture Center.</span></span>

<span data-ttu-id="ea828-147">Para outros tópicos de escalabilidade, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.</span><span class="sxs-lookup"><span data-stu-id="ea828-147">For other scalability topics, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="ea828-148">Segurança</span><span class="sxs-lookup"><span data-stu-id="ea828-148">Security</span></span>

<span data-ttu-id="ea828-149">Este cenário usa o Azure Active Directory B2C (Business 2 Consumer) para autenticar os usuários.</span><span class="sxs-lookup"><span data-stu-id="ea828-149">This scenario uses Azure Active Directory B2C (Business 2 Consumer) to authenticate users.</span></span> <span data-ttu-id="ea828-150">Com o AAD B2C, o chatbot não armazena informações de conta confidenciais ou credenciais de clientes.</span><span class="sxs-lookup"><span data-stu-id="ea828-150">With AAD B2C, your chatbot doesn't store any sensitive customer account information or credentials.</span></span> <span data-ttu-id="ea828-151">Para saber mais, veja [Visão geral do Azure Active Directory B2C][aadb2c-docs].</span><span class="sxs-lookup"><span data-stu-id="ea828-151">For more information, see [Azure Active Directory B2C overview][aadb2c-docs].</span></span>

<span data-ttu-id="ea828-152">As informações armazenadas no Banco de Dados SQL do Azure são criptografadas em repouso com TDE (Transparent Data Encryption).</span><span class="sxs-lookup"><span data-stu-id="ea828-152">Information stored in Azure SQL Database is encrypted at rest with transparent data encryption (TDE).</span></span> <span data-ttu-id="ea828-153">O Banco de Dados SQL também oferece o Always Encrypted, que criptografa dados durante a consulta e o processamento.</span><span class="sxs-lookup"><span data-stu-id="ea828-153">SQL Database also offers Always Encrypted which encrypts data during querying and processing.</span></span> <span data-ttu-id="ea828-154">Para obter mais informações sobre a segurança do Banco de Dados SQL, confira [Conformidade e segurança do Banco de Dados SQL][sqlsecurity-docs].</span><span class="sxs-lookup"><span data-stu-id="ea828-154">For more information on SQL Database security, see [Azure SQL Database security and compliance][sqlsecurity-docs].</span></span>

<span data-ttu-id="ea828-155">Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].</span><span class="sxs-lookup"><span data-stu-id="ea828-155">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="ea828-156">Resiliência</span><span class="sxs-lookup"><span data-stu-id="ea828-156">Resiliency</span></span>

<span data-ttu-id="ea828-157">Este cenário usa o Banco de Dados SQL do Azure para armazenar as reservas dos clientes.</span><span class="sxs-lookup"><span data-stu-id="ea828-157">This scenario uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="ea828-158">O Banco de Dados SQL inclui bancos de dados com redundância de zona, grupos de failover, replicação geográfica e backups automáticos.</span><span class="sxs-lookup"><span data-stu-id="ea828-158">SQL Database includes zone redundant databases, failover groups, geo-replication, and automatic backups.</span></span> <span data-ttu-id="ea828-159">Esses recursos permitem que seu aplicativo continue em execução caso ocorra um evento de manutenção ou interrupção.</span><span class="sxs-lookup"><span data-stu-id="ea828-159">These features allow your application to continue running if there is a maintenance event or outage.</span></span> <span data-ttu-id="ea828-160">Para obter mais informações, confira [Recursos de disponibilidade do Banco de Dados SQL][sqlavailability-docs].</span><span class="sxs-lookup"><span data-stu-id="ea828-160">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="ea828-161">Para monitorar a integridade do seu aplicativo, este cenário usa o Application Insights.</span><span class="sxs-lookup"><span data-stu-id="ea828-161">To monitor the health of your application, this scenario uses Application Insights.</span></span> <span data-ttu-id="ea828-162">Com o Application Insights, você pode gerar alertas e responder a problemas de desempenho que poderiam afetar a experiência do cliente e a disponibilidade do chatbot.</span><span class="sxs-lookup"><span data-stu-id="ea828-162">With Application Insights, you can generate alerts and respond to performance issues that would impact the customer experience and availability of the chatbot.</span></span> <span data-ttu-id="ea828-163">Para obter mais informações, confira [O que é o Application Insights?][appinsights-docs]</span><span class="sxs-lookup"><span data-stu-id="ea828-163">For more information, see [What is Application Insights?][appinsights-docs]</span></span>

<span data-ttu-id="ea828-164">Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="ea828-164">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="ea828-165">Implantar o cenário</span><span class="sxs-lookup"><span data-stu-id="ea828-165">Deploy the scenario</span></span>

<span data-ttu-id="ea828-166">Este cenário é dividido em três componentes para você explorar as áreas nas quais você quer se focar mais:</span><span class="sxs-lookup"><span data-stu-id="ea828-166">This scenario is divided into three components for you to explore areas that you are most focused on:</span></span>

- <span data-ttu-id="ea828-167">[Componentes da infraestrutura](#deploy-infrastructure-components).</span><span class="sxs-lookup"><span data-stu-id="ea828-167">[Infrastructure components](#deploy-infrastructure-components).</span></span> <span data-ttu-id="ea828-168">Use um modelo do Azure Resource Manager para implantar os componentes básicos da infraestrutura de Serviço de Aplicativo, aplicativo Web, Application Insights, conta de armazenamento, SQL Server e banco de dados.</span><span class="sxs-lookup"><span data-stu-id="ea828-168">Use an Azure Resource Manger template to deploy the core infrastructure components of an App Service, Web App, Application Insights, Storage account, and SQL Server and database.</span></span>
- <span data-ttu-id="ea828-169">[Chatbot de aplicativo Web](#deploy-web-app-chatbot).</span><span class="sxs-lookup"><span data-stu-id="ea828-169">[Web App Chatbot](#deploy-web-app-chatbot).</span></span> <span data-ttu-id="ea828-170">Use a CLI do Azure para implantar um bot com o Serviço de Bot e o aplicativo LUIS (Serviço Inteligente de Reconhecimento Vocal).</span><span class="sxs-lookup"><span data-stu-id="ea828-170">Use the Azure CLI to deploy a bot with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>
- <span data-ttu-id="ea828-171">[Exemplo de aplicativo de chatbot em C#](#deploy-chatbot-c-application-code).</span><span class="sxs-lookup"><span data-stu-id="ea828-171">[Sample C# chatbot application](#deploy-chatbot-c-application-code).</span></span> <span data-ttu-id="ea828-172">Use o Visual Studio para analisar o código do aplicativo C# de reservas de hotel de exemplo e implantar um bot no Azure.</span><span class="sxs-lookup"><span data-stu-id="ea828-172">Use Visual Studio to review the sample hotel reservation C# application code and deploy to a bot in Azure.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="ea828-173">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="ea828-173">Prerequisites</span></span>

<span data-ttu-id="ea828-174">Você deve ter uma conta do Azure já criada.</span><span class="sxs-lookup"><span data-stu-id="ea828-174">You must have an existing Azure account.</span></span> <span data-ttu-id="ea828-175">Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="ea828-175">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

### <a name="walk-through"></a><span data-ttu-id="ea828-176">Passo a passo</span><span class="sxs-lookup"><span data-stu-id="ea828-176">Walk-through</span></span>

<span data-ttu-id="ea828-177">Para implantar os componentes de infraestrutura com um modelo do Resource Manager, execute as etapas a seguir.</span><span class="sxs-lookup"><span data-stu-id="ea828-177">To deploy the infrastructure components with a Resource Manager template, perform the following steps.</span></span>

<!-- markdownlint-disable MD033 -->

1. <span data-ttu-id="ea828-178">Clique no botão **Implantar no Azure**:</span><span class="sxs-lookup"><span data-stu-id="ea828-178">Click the **Deploy to Azure** button:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="ea828-179">Aguarde até que a implantação de modelo seja aberta no portal do Azure e conclua as seguintes etapas:</span><span class="sxs-lookup"><span data-stu-id="ea828-179">Wait for the template deployment to open in the Azure portal, then complete the following steps:</span></span>
   - <span data-ttu-id="ea828-180">Escolha **criar novo** recurso de grupo e insira um nome como *myCommerceChatBotInfrastructure* na caixa de texto.</span><span class="sxs-lookup"><span data-stu-id="ea828-180">Choose to **Create new** resource group, then provide a name such as *myCommerceChatBotInfrastructure* in the text box.</span></span>
   - <span data-ttu-id="ea828-181">Selecione uma região na caixa suspensa **Local**.</span><span class="sxs-lookup"><span data-stu-id="ea828-181">Select a region from the **Location** drop-down box.</span></span>
   - <span data-ttu-id="ea828-182">Forneça um nome de usuário e uma senha segura para a conta de administrador do SQL Server.</span><span class="sxs-lookup"><span data-stu-id="ea828-182">Provide a username and secure password for the SQL Server administrator account.</span></span>
   - <span data-ttu-id="ea828-183">Analise os termos e condições e marque a opção **Concordo com os termos e condições declarados acima**.</span><span class="sxs-lookup"><span data-stu-id="ea828-183">Review the terms and conditions, then check **I agree to the terms and conditions stated above**.</span></span>
   - <span data-ttu-id="ea828-184">Selecione o botão **Comprar**.</span><span class="sxs-lookup"><span data-stu-id="ea828-184">Select the **Purchase** button.</span></span>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="ea828-185">A conclusão da implantação leva alguns minutos.</span><span class="sxs-lookup"><span data-stu-id="ea828-185">It takes a few minutes for the deployment to complete.</span></span>

### <a name="deploy-web-app-chatbot"></a><span data-ttu-id="ea828-186">Implantar o aplicativo Web de chatbot</span><span class="sxs-lookup"><span data-stu-id="ea828-186">Deploy Web App chatbot</span></span>

<span data-ttu-id="ea828-187">Para criar o chatbot, use a CLI do Azure.</span><span class="sxs-lookup"><span data-stu-id="ea828-187">To create the chatbot, use the Azure CLI.</span></span> <span data-ttu-id="ea828-188">O exemplo a seguir instala a extensão da CLI para o Serviço de Bot, cria um grupo de recursos e implanta um bot que usa o Application Insights.</span><span class="sxs-lookup"><span data-stu-id="ea828-188">The following example installs the CLI extension for Bot Service, creates a resource group, then deploys a bot that uses Application Insights.</span></span> <span data-ttu-id="ea828-189">Quando solicitado, autentique a conta da Microsoft e permita que o bot se registre no aplicativo LUIS (Serviço Inteligente de Reconhecimento Vocal) e no Serviço de Bot.</span><span class="sxs-lookup"><span data-stu-id="ea828-189">When prompted, authenticate your Microsoft account and allow the bot to register itself with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>

```azurecli-interactive
# Install the Azure CLI extension for the Bot Service
az extension add --name botservice --yes

# Create a resource group
az group create --name myCommerceChatbot --location eastus

# Create a Web App Chatbot that uses Application Insights
az bot create \
    --resource-group myCommerceChatbot \
    --name commerceChatbot \
    --location eastus \
    --kind webapp \
    --sku S1 \
    --insights eastus
```

### <a name="deploy-chatbot-c-application-code"></a><span data-ttu-id="ea828-190">Implantar o código do aplicativo de chatbot em C#</span><span class="sxs-lookup"><span data-stu-id="ea828-190">Deploy chatbot C# application code</span></span>

<span data-ttu-id="ea828-191">Um aplicativo de exemplo em C# está disponível no GitHub:</span><span class="sxs-lookup"><span data-stu-id="ea828-191">A sample C# application is available on GitHub:</span></span>

- [<span data-ttu-id="ea828-192">Exemplo de Bot de Comércio em C#</span><span class="sxs-lookup"><span data-stu-id="ea828-192">Commerce Bot C# sample</span></span>](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

<span data-ttu-id="ea828-193">O aplicativo de exemplo inclui os componentes de autenticação do Azure Active Directory e a integração com o componente do LUIS (Serviço Inteligente de Reconhecimento Vocal) dos Serviços Cognitivos.</span><span class="sxs-lookup"><span data-stu-id="ea828-193">The sample application includes the Azure Active Directory authentication components and integration with the Language Understanding and Intelligent Services (LUIS) component of Cognitive Services.</span></span> <span data-ttu-id="ea828-194">O aplicativo requer o Visual Studio para criar e implantar o cenário.</span><span class="sxs-lookup"><span data-stu-id="ea828-194">The application requires Visual Studio to build and deploy the scenario.</span></span> <span data-ttu-id="ea828-195">Outras informações sobre como configurar o AAD B2C e o aplicativo LUIS podem ser encontradas na documentação do repositório do GitHub.</span><span class="sxs-lookup"><span data-stu-id="ea828-195">Additional information on configuring AAD B2C and the LUIS app can be found in the GitHub repo documentation.</span></span>

## <a name="pricing"></a><span data-ttu-id="ea828-196">Preços</span><span class="sxs-lookup"><span data-stu-id="ea828-196">Pricing</span></span>

<span data-ttu-id="ea828-197">Para explorar o custo de executar esse cenário, todos os serviços são pré-configurados na calculadora de custos.</span><span class="sxs-lookup"><span data-stu-id="ea828-197">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="ea828-198">Para ver como o preço seria alterado em seu caso de uso específico, altere as variáveis apropriadas de acordo com o tráfego esperado.</span><span class="sxs-lookup"><span data-stu-id="ea828-198">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="ea828-199">Fornecemos três perfis de custos de exemplo com base na quantidade de mensagens que você espera que seu chatbot processe:</span><span class="sxs-lookup"><span data-stu-id="ea828-199">We have provided three sample cost profiles based on the number of messages you expect your chatbot to process:</span></span>

- <span data-ttu-id="ea828-200">[Pequeno][small-pricing]: esse exemplo de preço refere-se ao processamento de menos de 10 mil mensagens por mês.</span><span class="sxs-lookup"><span data-stu-id="ea828-200">[Small][small-pricing]: this pricing example correlates to processing < 10,000 messages per month.</span></span>
- <span data-ttu-id="ea828-201">[Médio][medium-pricing]: esse exemplo de preço refere-se ao processamento de menos de 500 mil mensagens por mês.</span><span class="sxs-lookup"><span data-stu-id="ea828-201">[Medium][medium-pricing]: this pricing example correlates to processing < 500,000 messages per month.</span></span>
- <span data-ttu-id="ea828-202">[Grande][large-pricing]: esse exemplo de preço refere-se ao processamento de < 10 milhões de mensagens por mês.</span><span class="sxs-lookup"><span data-stu-id="ea828-202">[Large][large-pricing]: this pricing example correlates to processing < 10 million messages per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="ea828-203">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="ea828-203">Related resources</span></span>

<span data-ttu-id="ea828-204">Para um conjunto de tutoriais guiados sobre o Serviço de Bot do Azure, confira a [seção de tutorial][botservice-docs] da documentação.</span><span class="sxs-lookup"><span data-stu-id="ea828-204">For a set of guided tutorials for the Azure Bot Service, see the [tutorial section][botservice-docs] of the documentation.</span></span>

<!-- links -->

[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/architecture-commerce-chatbot.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[botservice-docs]: /azure/bot-service/
[cognitive-docs]: /azure/cognitive-services/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sqlavailability-docs]: /azure/sql-database/sql-database-technical-overview#availability-capabilities
[sqldatabase-docs]: /azure/sql-database/
[sqlsecurity-docs]: /azure/sql-database/sql-database-technical-overview#advanced-security-and-compliance
[qna-maker]: /azure/cognitive-services/QnAMaker/Overview/overview
[speech-api]: /azure/cognitive-services/speech/home
[translator]: /azure/cognitive-services/translator/translator-info-overview

[small-pricing]: https://azure.com/e/dce05b6184904c50b38e1a8654f726b6
[medium-pricing]: https://azure.com/e/304d17106afc480dbc414f9726078a03
[large-pricing]: https://azure.com/e/8319dd5e5e3d4f118f9029e32a80e887