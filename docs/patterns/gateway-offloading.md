---
title: "Padrão de descarregamento de gateway"
description: "Descarregue a funcionalidade de serviço especializado ou compartilhado para um proxy do gateway."
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 6b3e4541aae77349ca91c18c788ddb508912361d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="gateway-offloading-pattern"></a><span data-ttu-id="daef6-103">Padrão de descarregamento de gateway</span><span class="sxs-lookup"><span data-stu-id="daef6-103">Gateway Offloading pattern</span></span>

<span data-ttu-id="daef6-104">Descarregue a funcionalidade de serviço especializado ou compartilhado para um proxy do gateway.</span><span class="sxs-lookup"><span data-stu-id="daef6-104">Offload shared or specialized service functionality to a gateway proxy.</span></span> <span data-ttu-id="daef6-105">Esse padrão pode simplificar o desenvolvimento de aplicativo movendo a funcionalidade de serviços compartilhados, como o uso de certificados SSL, de outras partes do aplicativo para o gateway.</span><span class="sxs-lookup"><span data-stu-id="daef6-105">This pattern can simplify application development by moving shared service functionality, such as the use of SSL certificates, from other parts of the application into the gateway.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="daef6-106">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="daef6-106">Context and problem</span></span>

<span data-ttu-id="daef6-107">Alguns recursos são comumente usados em vários serviços e esses recursos exigem configuração, gerenciamento e manutenção.</span><span class="sxs-lookup"><span data-stu-id="daef6-107">Some features are commonly used across multiple services, and these features require configuration, management, and maintenance.</span></span> <span data-ttu-id="daef6-108">Um serviço compartilhado ou especializado que é distribuído com cada implantação de aplicativo aumenta a sobrecarga administrativa e a probabilidade de erro de implantação.</span><span class="sxs-lookup"><span data-stu-id="daef6-108">A shared or specialized service that is distributed with every application deployment increases the administrative overhead and increases the likelihood of deployment error.</span></span> <span data-ttu-id="daef6-109">Todas as atualizações para um recurso compartilhado devem ser implantadas em todos os serviços que compartilham esse recurso.</span><span class="sxs-lookup"><span data-stu-id="daef6-109">Any updates to a shared feature must be deployed across all services that share that feature.</span></span>

<span data-ttu-id="daef6-110">Tratar corretamente os problemas de segurança (validação de token, criptografia e gerenciamento de certificados SSL) e outras tarefas complexas pode requerer que os membros da equipe tenham habilidades altamente especializadas.</span><span class="sxs-lookup"><span data-stu-id="daef6-110">Properly handling security issues (token validation, encryption, SSL certificate management) and other complex tasks can require team members to have highly specialized skills.</span></span> <span data-ttu-id="daef6-111">Por exemplo, um certificado necessário para um aplicativo deve ser configurado e implantado em todas as instâncias do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="daef6-111">For example, a certificate needed by an application must be configured and deployed on all application instances.</span></span> <span data-ttu-id="daef6-112">Com cada implantação nova, o certificado deve ser gerenciado para se garantir que ele não expire.</span><span class="sxs-lookup"><span data-stu-id="daef6-112">With each new deployment, the certificate must be managed to ensure that it does not expire.</span></span> <span data-ttu-id="daef6-113">Qualquer certificado comum que esteja prestes a expirar deve ser atualizado, testado e verificado em todas as implantações de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="daef6-113">Any common certificate that is due to expire must be updated, tested, and verified on every application deployment.</span></span>

<span data-ttu-id="daef6-114">Outros serviços comuns, como autenticação, autorização, registro em log, monitoramento ou [limitação](./throttling.md) podem ser difíceis de serem implementados e gerenciado em uma grande quantidade de implantações.</span><span class="sxs-lookup"><span data-stu-id="daef6-114">Other common services such as authentication, authorization, logging, monitoring, or [throttling](./throttling.md) can be difficult to implement and manage across a large number of deployments.</span></span> <span data-ttu-id="daef6-115">Talvez seja melhor consolidar esse tipo de funcionalidade para reduzir a sobrecarga e a chance de erros.</span><span class="sxs-lookup"><span data-stu-id="daef6-115">It may be better to consolidate this type of functionality, in order to reduce overhead and the chance of errors.</span></span>

## <a name="solution"></a><span data-ttu-id="daef6-116">Solução</span><span class="sxs-lookup"><span data-stu-id="daef6-116">Solution</span></span>

<span data-ttu-id="daef6-117">Descarregue alguns recursos em um gateway de API, particularmente preocupações abrangentes como gerenciamento de certificados, autenticação, término de SSL, monitoramento, conversão de protocolo ou limitação.</span><span class="sxs-lookup"><span data-stu-id="daef6-117">Offload some features into an API gateway, particularly cross-cutting concerns such as certificate management, authentication, SSL termination, monitoring, protocol translation, or throttling.</span></span> 

<span data-ttu-id="daef6-118">O diagrama a seguir mostra um gateway de API que termina as conexões SSL de entrada.</span><span class="sxs-lookup"><span data-stu-id="daef6-118">The following diagram shows an API gateway that terminates inbound SSL connections.</span></span> <span data-ttu-id="daef6-119">Ele solicita dados em nome do solicitante original de qualquer upstream de servidor HTTP do gateway de API.</span><span class="sxs-lookup"><span data-stu-id="daef6-119">It requests data on behalf of the original requestor from any HTTP server upstream of the API gateway.</span></span>

 ![](./_images/gateway-offload.png)
 
<span data-ttu-id="daef6-120">Os benefícios desse padrão incluem:</span><span class="sxs-lookup"><span data-stu-id="daef6-120">Benefits of this pattern include:</span></span>

- <span data-ttu-id="daef6-121">Simplificar o desenvolvimento de serviços removendo a necessidade de distribuir e manter recursos de suporte, como certificados de servidor Web e a configuração de sites seguros.</span><span class="sxs-lookup"><span data-stu-id="daef6-121">Simplify the development of services by removing the need to distribute and maintain supporting resources, such as web server certificates and configuration for secure websites.</span></span> <span data-ttu-id="daef6-122">Uma configuração mais simples resulta em gerenciamento e escalabilidade mais fáceis e simplifica as atualizações de serviço.</span><span class="sxs-lookup"><span data-stu-id="daef6-122">Simpler configuration results in easier management and scalability and makes service upgrades simpler.</span></span>

- <span data-ttu-id="daef6-123">Permitir que equipes dedicadas implementem recursos que exigem competências especializadas, como a segurança.</span><span class="sxs-lookup"><span data-stu-id="daef6-123">Allow dedicated teams to implement features that require specialized expertise, such as security.</span></span> <span data-ttu-id="daef6-124">Isso permite que sua equipe principal foque a funcionalidade do aplicativo, deixando essas preocupações especializadas, mas abrangentes, para os especialistas relevantes.</span><span class="sxs-lookup"><span data-stu-id="daef6-124">This allows your core team to focus on the application functionality, leaving these specialized but cross-cutting concerns to the relevant experts.</span></span>

- <span data-ttu-id="daef6-125">Fornecer alguma consistência para registro em log e monitoramento de solicitação e de reposta.</span><span class="sxs-lookup"><span data-stu-id="daef6-125">Provide some consistency for request and response logging and monitoring.</span></span> <span data-ttu-id="daef6-126">Mesmo que um serviço não esteja instrumentado corretamente, o gateway pode ser configurado para garantir um nível mínimo de monitoramento e registro em log.</span><span class="sxs-lookup"><span data-stu-id="daef6-126">Even if a service is not correctly instrumented, the gateway can be configured to ensure a minimum level of monitoring and logging.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="daef6-127">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="daef6-127">Issues and considerations</span></span>

- <span data-ttu-id="daef6-128">Assegure que o gateway de API seja altamente disponível e resistente a falhas.</span><span class="sxs-lookup"><span data-stu-id="daef6-128">Ensure the API gateway is highly available and resilient to failure.</span></span> <span data-ttu-id="daef6-129">Evite pontos únicos de falha executando várias instâncias do seu gateway de API.</span><span class="sxs-lookup"><span data-stu-id="daef6-129">Avoid single points of failure by running multiple instances of your API gateway.</span></span> 
- <span data-ttu-id="daef6-130">Assegure que o gateway seja projetado para os requisitos de capacidade e dimensionamento de seu aplicativo e pontos de extremidade.</span><span class="sxs-lookup"><span data-stu-id="daef6-130">Ensure the gateway is designed for the capacity and scaling requirements of your application and endpoints.</span></span> <span data-ttu-id="daef6-131">Verifique se o gateway não se torna um gargalo do aplicativo e se é suficientemente escalonável.</span><span class="sxs-lookup"><span data-stu-id="daef6-131">Make sure the gateway does not become a bottleneck for the application and is sufficiently scalable.</span></span>
- <span data-ttu-id="daef6-132">Descarregue somente recursos que sejam usados pelo aplicativo inteiro, como a segurança ou transferência de dados.</span><span class="sxs-lookup"><span data-stu-id="daef6-132">Only offload features that are used by the entire application, such as security or data transfer.</span></span>
- <span data-ttu-id="daef6-133">A lógica de negócios nunca deve ser descarregada para o gateway de API.</span><span class="sxs-lookup"><span data-stu-id="daef6-133">Business logic should never be offloaded to the API gateway.</span></span> 
- <span data-ttu-id="daef6-134">Se precisar controlar transações, leve em consideração gerar IDs de correlação para fins de registro de log.</span><span class="sxs-lookup"><span data-stu-id="daef6-134">If you need to track transactions, consider generating correlation IDs for logging purposes.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="daef6-135">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="daef6-135">When to use this pattern</span></span>

<span data-ttu-id="daef6-136">Use esse padrão quando:</span><span class="sxs-lookup"><span data-stu-id="daef6-136">Use this pattern when:</span></span>

- <span data-ttu-id="daef6-137">Uma implantação de aplicativo tem uma preocupação compartilhada, como certificados SSL ou criptografia.</span><span class="sxs-lookup"><span data-stu-id="daef6-137">An application deployment has a shared concern such as SSL certificates or encryption.</span></span>
- <span data-ttu-id="daef6-138">Em um recurso que seja comum entre implantações de aplicativos que podem ter requisitos de recursos diferentes, como recursos de memória, capacidade de armazenamento ou conexões de rede.</span><span class="sxs-lookup"><span data-stu-id="daef6-138">A feature that is common across application deployments that may have different resource requirements, such as memory resources, storage capacity or network connections.</span></span>
- <span data-ttu-id="daef6-139">Você deseja mover a responsabilidade por problemas, como segurança de rede, limitação ou outras preocupações de limite de rede, para uma equipe mais especializada.</span><span class="sxs-lookup"><span data-stu-id="daef6-139">You wish to move the responsibility for issues such as network security, throttling, or other network boundary concerns to a more specialized team.</span></span>

<span data-ttu-id="daef6-140">Esse padrão pode não ser adequado caso ele introduza um acoplamento entre serviços.</span><span class="sxs-lookup"><span data-stu-id="daef6-140">This pattern may not be suitable if it introduces coupling across services.</span></span>

## <a name="example"></a><span data-ttu-id="daef6-141">Exemplo</span><span class="sxs-lookup"><span data-stu-id="daef6-141">Example</span></span>

<span data-ttu-id="daef6-142">Ao usar o Nginx como o dispositivo de descarregamento de SSL, a configuração a seguir encerra uma conexão SSL de entrada e distribui a conexão para um dos três servidores HTTP de upstream.</span><span class="sxs-lookup"><span data-stu-id="daef6-142">Using Nginx as the SSL offload appliance, the following configuration terminates an inbound SSL connection and distributes the connection to one of three upstream HTTP servers.</span></span>

```
upstream iis {
        server  10.3.0.10    max_fails=3    fail_timeout=15s;
        server  10.3.0.20    max_fails=3    fail_timeout=15s;
        server  10.3.0.30    max_fails=3    fail_timeout=15s;
}

server {
        listen 443;
        ssl on;
        ssl_certificate /etc/nginx/ssl/domain.cer;
        ssl_certificate_key /etc/nginx/ssl/domain.key;

        location / {
                set $targ iis;
                proxy_pass http://$targ;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $host;
        }
}
```

## <a name="related-guidance"></a><span data-ttu-id="daef6-143">Diretrizes relacionadas</span><span class="sxs-lookup"><span data-stu-id="daef6-143">Related guidance</span></span>

- [<span data-ttu-id="daef6-144">Padrão de back-ends para front-ends</span><span class="sxs-lookup"><span data-stu-id="daef6-144">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="daef6-145">Padrão de agregação de gateway</span><span class="sxs-lookup"><span data-stu-id="daef6-145">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="daef6-146">Padrão de roteamento de gateway</span><span class="sxs-lookup"><span data-stu-id="daef6-146">Gateway Routing pattern</span></span>](./gateway-routing.md)

