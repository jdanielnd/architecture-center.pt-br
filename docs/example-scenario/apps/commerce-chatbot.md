---
title: Formato de conversação chatbot do Azure para reservas de hotel
description: Solução comprovada para a criação de um chatbot de conversação para aplicativos comerciais com o Serviço de Bot do Azure, os Serviços Cognitivos e o LUIS, o Banco de Dados SQL do Azure e o Application Insights.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: 85bdc3194961bbbd8d89db34e5c56e4baa8d8599
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891254"
---
# <a name="conversational-azure-chatbot-for-hotel-reservations"></a>Formato de conversação chatbot do Azure para reservas de hotel

Este cenário de exemplo é aplicável a empresas que precisam integrar um chatbot de conversação aos aplicativos. Nesta solução, um chatbot C# é usado por uma cadeia de hotel que permite que os clientes verifiquem a disponibilidade e reservem quartos por meio de um aplicativo Web ou móvel.

Entre os cenários de exemplo estão a possibilidade de os clientes exibirem a disponibilidade do hotel e reservarem quartos, analisarem um menu de restaurante que faz entrega e fazerem pedido ou pesquisarem e pedirem impressões de fotografias. Tradicionalmente, as empresas precisariam contratar e treinar agentes de atendimento ao cliente para responder a essas solicitações, e os clientes teriam que esperar até que um representante estivesse disponível para prestar assistência.

Usando os serviços do Azure, como o Serviço de Bot e Reconhecimento vocal ou serviços da API de Reconhecimento Vocal, as empresas podem auxiliar os clientes e processar pedidos ou reservas com bots automatizados e escalonáveis.

## <a name="potential-use-cases"></a>Possíveis casos de uso

Considere usar a solução nos casos de uso abaixo:

* Exibir menu de entrega do restaurante e pedir comida
* Verificar a disponibilidade do hotel e reservar um quarto
* Pesquisar fotos disponíveis e solicitar impressões

## <a name="architecture"></a>Arquitetura

![Visão geral da arquitetura dos componentes do Azure envolvidos em um chatbot de conversação][architecture]

Esta solução aborda um bot de conversação que funciona como um concierge de um hotel. Os dados fluem pela solução da seguinte maneira:

1. O cliente acessa o chatbot em um dispositivo móvel ou aplicativo Web.
2. Usando o Azure Active Directory B2C, o usuário é autenticado.
3. Interagindo com o Serviço de Bot, o usuário solicita informações sobre a disponibilidade do hotel.
4. Os Serviços Cognitivos processam a solicitação de linguagem natural para entender a comunicação do cliente.
5. Depois que o usuário fica satisfeito com os resultados, o bot adiciona ou atualiza a reserva do cliente em um Banco de Dados SQL.
6. O Application Insights reúne telemetria do tempo de execução ao longo do processo para ajudar a equipe de DevOps em relação ao desempenho e o uso do bot.

### <a name="components"></a>Componentes

* O [Azure Active Directory][aad-docs] é o serviço de gerenciamento de identidades e diretório multilocatário baseado em nuvem da Microsoft. O Azure AD dá suporte a um conector de B2C, permitindo que você identifique pessoas que usam IDs externas, como Google, Facebook ou uma conta da Microsoft.
* O [Serviço de Aplicativo][appservice-docs] permite criar e hospedar aplicativos Web na linguagem de programação de sua escolha, sem gerenciamento de infraestrutura.
* O [Serviço de Bot][botservice-docs] fornece ferramentas para compilar, testar, implantar e gerenciar bots inteligentes.
* Os [Serviços Cognitivos][cognitive-docs] permitem usar algoritmos para ver, ouvir, falar, entender e interpretar as necessidades do usuário por meio de métodos naturais de comunicação.
* O [Banco de Dados SQL] [ sqldatabase-docs] é um serviço de banco de dados relacional de nuvem totalmente gerenciado que oferece compatibilidade com o mecanismo do SQL Server.
* O [Application Insights] [ appinsights-docs] é um serviço de APM (gerenciamento de desempenho de aplicativos) extensível que permite que você monitore o desempenho de aplicativos como o chatbot.

### <a name="alternatives"></a>Alternativas

* A [Speech API da Microsoft] [ speech-api] pode ser usada para alterar como os clientes interagem com o bot.
* O [QnA Maker] [ qna-maker] pode ser usado para adicionar conhecimento ao bot rapidamente a partir de conteúdo semiestruturado, como o de perguntas frequentes.
* [Tradução de Texto] [ translator] é um serviço que você pode considerar para adicionar suporte multilíngue ao bot facilmente.

## <a name="considerations"></a>Considerações

### <a name="availability"></a>Disponibilidade

Essa solução usa o Banco de Dados SQL para armazenar as reservas de clientes. O Banco de Dados SQL inclui bancos de dados com redundância de zona, grupos de failover e replicação geográfica. Para obter mais informações, confira [Recursos de disponibilidade do Banco de Dados SQL][sqlavailability-docs].

Para outros tópicos de escalabilidade, confira a [lista de verificação de disponibilidade] [ availability] no Azure Architecture Center.

### <a name="scalability"></a>Escalabilidade

Esta solução usa o Serviço de Aplicativo do Azure. Com o Serviço de Aplicativo, você pode dimensionar automaticamente o número de instâncias que executam o bot. Essa funcionalidade permite acompanhar a demanda dos clientes em relação ao aplicativo Web e ao chatbot. Para obter mais informações sobre o dimensionamento automático, confira [Melhores práticas de dimensionamento automático] [ autoscaling] no Architecture Center.

Para outros tópicos de escalabilidade, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.

### <a name="security"></a>Segurança

Esta solução usa o Azure Active Directory B2C para autenticar usuários. Com o AAD B2C, o chatbot não armazena informações de conta confidenciais ou credenciais de clientes. Para saber mais, veja [Visão geral do Azure Active Directory B2C][aadb2c-docs].

As informações armazenadas no Banco de Dados SQL do Azure são criptografadas em repouso com TDE (Transparent Data Encryption). O Banco de Dados SQL também oferece o Always Encrypted, que criptografa dados durante a consulta e o processamento. Para obter mais informações sobre a segurança do Banco de Dados SQL, confira [Conformidade e segurança do Banco de Dados SQL][sqlsecurity-docs].

Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].

### <a name="resiliency"></a>Resiliência

Essa solução usa o Banco de Dados SQL para armazenar as reservas de clientes. O Banco de Dados SQL inclui bancos de dados com redundância de zona, grupos de failover, replicação geográfica e backups automáticos. Esses recursos permitem que seu aplicativo continue em execução caso ocorra um evento de manutenção ou interrupção. Para obter mais informações, confira [Recursos de disponibilidade do Banco de Dados SQL][sqlavailability-docs].

Para monitorar a integridade do aplicativo, esta solução usa o Application Insights. Com o Application Insights, você pode gerar alertas e responder a problemas de desempenho que poderiam afetar a experiência do cliente e a disponibilidade do chatbot. Para obter mais informações, confira [O que é o Application Insights?][appinsights-docs]

Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].

## <a name="deploy-the-solution"></a>Implantar a solução

A solução é dividida em três componentes para você explorar as áreas de maior relevância:

* [Componentes da infraestrutura](#deploy-infrastructure-components). Use um modelo do Azure Resource Manager para implantar os componentes básicos da infraestrutura de Serviço de Aplicativo, aplicativo Web, Application Insights, conta de armazenamento, SQL Server e banco de dados.
* [Chatbot de aplicativo Web](#deploy-web-app-chatbot). Use a CLI do Azure para implantar um bot com o Serviço de Bot e o aplicativo LUIS (Serviço Inteligente de Reconhecimento Vocal).
* [Exemplo de aplicativo de chatbot em C#](#deploy-chatbot-c-application-code). Use o Visual Studio para analisar o código do aplicativo C# de reservas de hotel de exemplo e implantar um bot no Azure.

**Pré-requisitos.** Você deve ter uma conta do Azure já criada. Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.

### <a name="deploy-infrastructure-components"></a>Implantar componentes de infraestrutura

Para implantar os componentes de infraestrutura com um modelo do Azure Resource Manager, execute as etapas a seguir.

1. Clique no botão **Implantar no Azure**:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Aguarde até que a implantação de modelo seja aberta no portal do Azure e conclua as seguintes etapas:
   * Escolha **criar novo** recurso de grupo e insira um nome como *myCommerceChatBotInfrastructure* na caixa de texto.
   * Selecione uma região na caixa suspensa **Local**.
   * Forneça um nome de usuário e uma senha segura para a conta de administrador do SQL Server.
   * Analise os termos e condições e marque a opção **Concordo com os termos e condições declarados acima**.
   * Selecione o botão **Comprar**.

A conclusão da implantação leva alguns minutos.

### <a name="deploy-web-app-chatbot"></a>Implantar o aplicativo Web de chatbot

Para criar o chatbot, use a CLI do Azure. O exemplo a seguir instala a extensão da CLI para o Serviço de Bot, cria um grupo de recursos e implanta um bot que usa o Application Insights. Quando solicitado, autentique a conta da Microsoft e permita que o bot se registre no aplicativo LUIS (Serviço Inteligente de Reconhecimento Vocal) e no Serviço de Bot.

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

### <a name="deploy-chatbot-c-application-code"></a>Implantar o código do aplicativo de chatbot em C#

Um aplicativo de exemplo em C# está disponível no GitHub: 

* [Exemplo de Bot de Comércio em C#](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

O aplicativo de exemplo inclui os componentes de autenticação do Azure Active Directory e a integração com o componente do LUIS (Serviço Inteligente de Reconhecimento Vocal) dos Serviços Cognitivos. O aplicativo requer que o Visual Studio compile e implante a solução. Outras informações sobre como configurar o AAD B2C e o aplicativo LUIS podem ser encontradas na documentação do repositório do GitHub.

## <a name="pricing"></a>Preços

Para explorar o custo da execução dessa solução, todos os serviços estarão pré-configurados na calculadora de custos. Para ver como o preço seria alterado em seu caso de uso específico, altere as variáveis apropriadas de acordo com o tráfego esperado.

Fornecemos três perfis de custo de exemplo com base na quantidade de mensagens que o chatbot deverá processar:

* [Pequeno][small-pricing]: referente ao processamento de < 10 mil mensagens por mês.
* [Médio][medium-pricing]: referente ao processamento de < 500 mil mensagens por mês.
* [Grande][large-pricing]: referente ao processamento de < 10 milhões de mensagens por mês.

## <a name="related-resources"></a>Recursos relacionados

Para um conjunto de tutoriais guiados sobre como aproveitar o Serviço de Bot do Azure, confira o [nó de tutorial] [ botservice-docs] da documentação.

<!-- links -->
[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/commerce-chatbot/architecture-commerce-chatbot.png
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