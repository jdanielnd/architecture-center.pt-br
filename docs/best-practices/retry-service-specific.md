---
title: Diretriz específica do serviço de repetição
description: Diretriz específica de serviço para configurar o mecanismo de repetição.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: c5a9bc99c4693f35c38dabcf07b3465add6a8cb1
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429530"
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="b9189-103">Repetir as diretrizes para serviços específicos</span><span class="sxs-lookup"><span data-stu-id="b9189-103">Retry guidance for specific services</span></span>

<span data-ttu-id="b9189-104">A maioria dos serviços do Azure e SDKs do cliente inclui um mecanismo de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="b9189-105">No entanto, eles são diferentes porque cada serviço apresenta características e requisitos diferentes, e cada mecanismo de repetição é ajustado para um serviço específico.</span><span class="sxs-lookup"><span data-stu-id="b9189-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="b9189-106">Este guia resume os recursos do mecanismo de repetição para a maioria dos serviços do Azure, além de incluir informações que ajudam a usar, a adaptar ou a estender o mecanismo de repetição para o serviço em questão.</span><span class="sxs-lookup"><span data-stu-id="b9189-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="b9189-107">Para obter orientação geral sobre o tratamento de falhas transitórias e como repetir conexões e operações em serviços e recursos, consulte [Diretriz de repetição](./transient-faults.md).</span><span class="sxs-lookup"><span data-stu-id="b9189-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="b9189-108">A tabela a seguir resume os recursos de repetição para os serviços do Azure descritos nesta diretriz.</span><span class="sxs-lookup"><span data-stu-id="b9189-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="b9189-109">**Serviço**</span><span class="sxs-lookup"><span data-stu-id="b9189-109">**Service**</span></span> | <span data-ttu-id="b9189-110">**Recursos de repetição**</span><span class="sxs-lookup"><span data-stu-id="b9189-110">**Retry capabilities**</span></span> | <span data-ttu-id="b9189-111">**Configuração de política**</span><span class="sxs-lookup"><span data-stu-id="b9189-111">**Policy configuration**</span></span> | <span data-ttu-id="b9189-112">**Escopo**</span><span class="sxs-lookup"><span data-stu-id="b9189-112">**Scope**</span></span> | <span data-ttu-id="b9189-113">**Recursos de telemetria**</span><span class="sxs-lookup"><span data-stu-id="b9189-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="b9189-114">**[Azure Active Directory](#azure-active-directory)**</span><span class="sxs-lookup"><span data-stu-id="b9189-114">**[Azure Active Directory](#azure-active-directory)**</span></span> |<span data-ttu-id="b9189-115">Nativo na biblioteca ADAL</span><span class="sxs-lookup"><span data-stu-id="b9189-115">Native in ADAL library</span></span> |<span data-ttu-id="b9189-116">Incorporado na biblioteca ADAL</span><span class="sxs-lookup"><span data-stu-id="b9189-116">Embeded into ADAL library</span></span> |<span data-ttu-id="b9189-117">Interna</span><span class="sxs-lookup"><span data-stu-id="b9189-117">Internal</span></span> |<span data-ttu-id="b9189-118">Nenhum</span><span class="sxs-lookup"><span data-stu-id="b9189-118">None</span></span> |
| <span data-ttu-id="b9189-119">**[Cosmos DB](#cosmos-db)**</span><span class="sxs-lookup"><span data-stu-id="b9189-119">**[Cosmos DB](#cosmos-db)**</span></span> |<span data-ttu-id="b9189-120">Nativo no serviço</span><span class="sxs-lookup"><span data-stu-id="b9189-120">Native in service</span></span> |<span data-ttu-id="b9189-121">Não configurável</span><span class="sxs-lookup"><span data-stu-id="b9189-121">Non-configurable</span></span> |<span data-ttu-id="b9189-122">Global</span><span class="sxs-lookup"><span data-stu-id="b9189-122">Global</span></span> |<span data-ttu-id="b9189-123">TraceSource</span><span class="sxs-lookup"><span data-stu-id="b9189-123">TraceSource</span></span> |
| <span data-ttu-id="b9189-124">**Data Lake Store**</span><span class="sxs-lookup"><span data-stu-id="b9189-124">**Data Lake Store**</span></span> |<span data-ttu-id="b9189-125">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-125">Native in client</span></span> |<span data-ttu-id="b9189-126">Não configurável</span><span class="sxs-lookup"><span data-stu-id="b9189-126">Non-configurable</span></span> |<span data-ttu-id="b9189-127">Operações individuais</span><span class="sxs-lookup"><span data-stu-id="b9189-127">Individual operations</span></span> |<span data-ttu-id="b9189-128">Nenhum</span><span class="sxs-lookup"><span data-stu-id="b9189-128">None</span></span> |
| <span data-ttu-id="b9189-129">**[Hubs de Eventos](#event-hubs)**</span><span class="sxs-lookup"><span data-stu-id="b9189-129">**[Event Hubs](#event-hubs)**</span></span> |<span data-ttu-id="b9189-130">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-130">Native in client</span></span> |<span data-ttu-id="b9189-131">Programático</span><span class="sxs-lookup"><span data-stu-id="b9189-131">Programmatic</span></span> |<span data-ttu-id="b9189-132">Cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-132">Client</span></span> |<span data-ttu-id="b9189-133">Nenhum</span><span class="sxs-lookup"><span data-stu-id="b9189-133">None</span></span> |
| <span data-ttu-id="b9189-134">**[Hub IoT](#iot-hub)**</span><span class="sxs-lookup"><span data-stu-id="b9189-134">**[IoT Hub](#iot-hub)**</span></span> |<span data-ttu-id="b9189-135">Nativo no cliente SDK</span><span class="sxs-lookup"><span data-stu-id="b9189-135">Native in client SDK</span></span> |<span data-ttu-id="b9189-136">Programático</span><span class="sxs-lookup"><span data-stu-id="b9189-136">Programmatic</span></span> |<span data-ttu-id="b9189-137">Cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-137">Client</span></span> |<span data-ttu-id="b9189-138">Nenhum</span><span class="sxs-lookup"><span data-stu-id="b9189-138">None</span></span> |
| <span data-ttu-id="b9189-139">**[Cache Redis](#azure-redis-cache)**</span><span class="sxs-lookup"><span data-stu-id="b9189-139">**[Redis Cache](#azure-redis-cache)**</span></span> |<span data-ttu-id="b9189-140">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-140">Native in client</span></span> |<span data-ttu-id="b9189-141">Programático</span><span class="sxs-lookup"><span data-stu-id="b9189-141">Programmatic</span></span> |<span data-ttu-id="b9189-142">Cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-142">Client</span></span> |<span data-ttu-id="b9189-143">TextWriter</span><span class="sxs-lookup"><span data-stu-id="b9189-143">TextWriter</span></span> |
| <span data-ttu-id="b9189-144">**[Pesquisa](#azure-search)**</span><span class="sxs-lookup"><span data-stu-id="b9189-144">**[Search](#azure-search)**</span></span> |<span data-ttu-id="b9189-145">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-145">Native in client</span></span> |<span data-ttu-id="b9189-146">Programático</span><span class="sxs-lookup"><span data-stu-id="b9189-146">Programmatic</span></span> |<span data-ttu-id="b9189-147">Cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-147">Client</span></span> |<span data-ttu-id="b9189-148">ETW ou Personalizado</span><span class="sxs-lookup"><span data-stu-id="b9189-148">ETW or Custom</span></span> |
| <span data-ttu-id="b9189-149">**[Barramento de Serviço](#service-bus)**</span><span class="sxs-lookup"><span data-stu-id="b9189-149">**[Service Bus](#service-bus)**</span></span> |<span data-ttu-id="b9189-150">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-150">Native in client</span></span> |<span data-ttu-id="b9189-151">Programático</span><span class="sxs-lookup"><span data-stu-id="b9189-151">Programmatic</span></span> |<span data-ttu-id="b9189-152">Gerenciador de Namespace, Messaging Factory e Cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-152">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="b9189-153">ETW</span><span class="sxs-lookup"><span data-stu-id="b9189-153">ETW</span></span> |
| <span data-ttu-id="b9189-154">**[Service Fabric](#service-fabric)**</span><span class="sxs-lookup"><span data-stu-id="b9189-154">**[Service Fabric](#service-fabric)**</span></span> |<span data-ttu-id="b9189-155">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-155">Native in client</span></span> |<span data-ttu-id="b9189-156">Programático</span><span class="sxs-lookup"><span data-stu-id="b9189-156">Programmatic</span></span> |<span data-ttu-id="b9189-157">Cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-157">Client</span></span> |<span data-ttu-id="b9189-158">Nenhum</span><span class="sxs-lookup"><span data-stu-id="b9189-158">None</span></span> | 
| <span data-ttu-id="b9189-159">**[Banco de dados SQL com ADO.NET](#sql-database-using-adonet)**</span><span class="sxs-lookup"><span data-stu-id="b9189-159">**[SQL Database with ADO.NET](#sql-database-using-adonet)**</span></span> |[<span data-ttu-id="b9189-160">Polly</span><span class="sxs-lookup"><span data-stu-id="b9189-160">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="b9189-161">Programático e declarativo</span><span class="sxs-lookup"><span data-stu-id="b9189-161">Declarative and programmatic</span></span> |<span data-ttu-id="b9189-162">Instruções ou blocos de código únicos</span><span class="sxs-lookup"><span data-stu-id="b9189-162">Single statements or blocks of code</span></span> |<span data-ttu-id="b9189-163">Personalizado</span><span class="sxs-lookup"><span data-stu-id="b9189-163">Custom</span></span> |
| <span data-ttu-id="b9189-164">**[Banco de dados SQL com o Entity Framework](#sql-database-using-entity-framework-6)**</span><span class="sxs-lookup"><span data-stu-id="b9189-164">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)**</span></span> |<span data-ttu-id="b9189-165">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-165">Native in client</span></span> |<span data-ttu-id="b9189-166">Programático</span><span class="sxs-lookup"><span data-stu-id="b9189-166">Programmatic</span></span> |<span data-ttu-id="b9189-167">Global por AppDomain</span><span class="sxs-lookup"><span data-stu-id="b9189-167">Global per AppDomain</span></span> |<span data-ttu-id="b9189-168">Nenhum</span><span class="sxs-lookup"><span data-stu-id="b9189-168">None</span></span> |
| <span data-ttu-id="b9189-169">**[Banco de dados SQL com Entity Framework Core](#sql-database-using-entity-framework-core)**</span><span class="sxs-lookup"><span data-stu-id="b9189-169">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)**</span></span> |<span data-ttu-id="b9189-170">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-170">Native in client</span></span> |<span data-ttu-id="b9189-171">Programático</span><span class="sxs-lookup"><span data-stu-id="b9189-171">Programmatic</span></span> |<span data-ttu-id="b9189-172">Global por AppDomain</span><span class="sxs-lookup"><span data-stu-id="b9189-172">Global per AppDomain</span></span> |<span data-ttu-id="b9189-173">Nenhum</span><span class="sxs-lookup"><span data-stu-id="b9189-173">None</span></span> |
| <span data-ttu-id="b9189-174">**[Armazenamento](#azure-storage)**</span><span class="sxs-lookup"><span data-stu-id="b9189-174">**[Storage](#azure-storage)**</span></span> |<span data-ttu-id="b9189-175">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-175">Native in client</span></span> |<span data-ttu-id="b9189-176">Programático</span><span class="sxs-lookup"><span data-stu-id="b9189-176">Programmatic</span></span> |<span data-ttu-id="b9189-177">Operações individuais e de cliente</span><span class="sxs-lookup"><span data-stu-id="b9189-177">Client and individual operations</span></span> |<span data-ttu-id="b9189-178">TraceSource</span><span class="sxs-lookup"><span data-stu-id="b9189-178">TraceSource</span></span> |

> [!NOTE]
> <span data-ttu-id="b9189-179">Para a maioria dos mecanismos de repetição internos do Azure, atualmente não há meios de aplicar uma política de repetição diferente para diversos tipos de erro ou exceção.</span><span class="sxs-lookup"><span data-stu-id="b9189-179">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception.</span></span> <span data-ttu-id="b9189-180">Você deve configurar uma política que fornece o melhor desempenho médio e disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="b9189-180">You should configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="b9189-181">Uma maneira de ajustar a política é analisar arquivos de log para determinar o tipo de falha transitória que está ocorrendo.</span><span class="sxs-lookup"><span data-stu-id="b9189-181">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> 

## <a name="azure-active-directory"></a><span data-ttu-id="b9189-182">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="b9189-182">Azure Active Directory</span></span>
<span data-ttu-id="b9189-183">O Azure Active Directory (Azure AD) é uma solução abrangente de nuvem para gerenciamento de acesso e identidade que combina serviços principais de diretório, governança avançada de identidade, segurança e gerenciamento de acesso a aplicativos.</span><span class="sxs-lookup"><span data-stu-id="b9189-183">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="b9189-184">O AD do Azure também oferece aos desenvolvedores uma plataforma de gerenciamento de identidade para fornecer controle de acesso aos respectivos aplicativos, com base nas regras e políticas centralizadas.</span><span class="sxs-lookup"><span data-stu-id="b9189-184">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

> [!NOTE]
> <span data-ttu-id="b9189-185">Para obter diretrizes sobre a repetição em pontos de extremidade de identidade de serviço gerenciado, veja [como usar uma MSI (Identidade de Serviço Gerenciada) de VM do Azure para aquisição de token](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span><span class="sxs-lookup"><span data-stu-id="b9189-185">For retry guidance on Managed Service Identity endpoints, see [How to use an Azure VM Managed Service Identity (MSI) for token acquisition](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="b9189-186">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-186">Retry mechanism</span></span>
<span data-ttu-id="b9189-187">Na Biblioteca de autenticação do Active Directory (ADAL) há um mecanismo interno de repetição para o Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="b9189-187">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="b9189-188">Para evitar bloqueios inesperados, recomendamos que o ADAL lide com as repetições e **não** permita que as bibliotecas de terceiros ou o código do aplicativo repitam as conexões com falhas.</span><span class="sxs-lookup"><span data-stu-id="b9189-188">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="b9189-189">Diretriz de uso de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-189">Retry usage guidance</span></span>
<span data-ttu-id="b9189-190">Considere as seguintes diretrizes ao usar o Azure Active Directory:</span><span class="sxs-lookup"><span data-stu-id="b9189-190">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="b9189-191">Quando possível, use a biblioteca ADAL e o suporte interno para repetições.</span><span class="sxs-lookup"><span data-stu-id="b9189-191">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="b9189-192">Se estiver a usar a API REST do Azure Active Directory, tente novamente a operação se o código de resultado for 429 (Muitos Pedidos) ou for um erro no intervalo 5xx.</span><span class="sxs-lookup"><span data-stu-id="b9189-192">If you are using the REST API for Azure Active Directory, retry the operation if the result code is 429 (Too Many Requests) or an error in the 5xx range.</span></span> <span data-ttu-id="b9189-193">Não repita para nenhum outro erro.</span><span class="sxs-lookup"><span data-stu-id="b9189-193">Do not retry for any other errors.</span></span>
* <span data-ttu-id="b9189-194">Uma política de retirada exponencial é recomendada para uso em cenários de lote com o Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="b9189-194">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="b9189-195">Considere começar com as seguintes configurações para operações de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-195">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="b9189-196">Essas são configurações de uso geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.</span><span class="sxs-lookup"><span data-stu-id="b9189-196">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="b9189-197">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="b9189-197">**Context**</span></span> | <span data-ttu-id="b9189-198">**Exemplo de latênciaE2E<br />máxima de destino**</span><span class="sxs-lookup"><span data-stu-id="b9189-198">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="b9189-199">**Estratégia de repetição**</span><span class="sxs-lookup"><span data-stu-id="b9189-199">**Retry strategy**</span></span> | <span data-ttu-id="b9189-200">**Configurações**</span><span class="sxs-lookup"><span data-stu-id="b9189-200">**Settings**</span></span> | <span data-ttu-id="b9189-201">**Valores**</span><span class="sxs-lookup"><span data-stu-id="b9189-201">**Values**</span></span> | <span data-ttu-id="b9189-202">**Como funciona**</span><span class="sxs-lookup"><span data-stu-id="b9189-202">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="b9189-203">Interativo, interface de usuário</span><span class="sxs-lookup"><span data-stu-id="b9189-203">Interactive, UI,</span></span><br /><span data-ttu-id="b9189-204">ou primeiro plano</span><span class="sxs-lookup"><span data-stu-id="b9189-204">or foreground</span></span> |<span data-ttu-id="b9189-205">2 s</span><span class="sxs-lookup"><span data-stu-id="b9189-205">2 sec</span></span> |<span data-ttu-id="b9189-206">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="b9189-206">FixedInterval</span></span> |<span data-ttu-id="b9189-207">Contagem de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-207">Retry count</span></span><br /><span data-ttu-id="b9189-208">Intervalo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-208">Retry interval</span></span><br /><span data-ttu-id="b9189-209">Primeira repetição rápida</span><span class="sxs-lookup"><span data-stu-id="b9189-209">First fast retry</span></span> |<span data-ttu-id="b9189-210">3</span><span class="sxs-lookup"><span data-stu-id="b9189-210">3</span></span><br /><span data-ttu-id="b9189-211">500 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-211">500 ms</span></span><br /><span data-ttu-id="b9189-212">verdadeiro</span><span class="sxs-lookup"><span data-stu-id="b9189-212">true</span></span> |<span data-ttu-id="b9189-213">1ª tentativa — intervalo de 0 s</span><span class="sxs-lookup"><span data-stu-id="b9189-213">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="b9189-214">2ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-214">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="b9189-215">3ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-215">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="b9189-216">Segundo plano ou</span><span class="sxs-lookup"><span data-stu-id="b9189-216">Background or</span></span><br /><span data-ttu-id="b9189-217">lote</span><span class="sxs-lookup"><span data-stu-id="b9189-217">batch</span></span> |<span data-ttu-id="b9189-218">60 s</span><span class="sxs-lookup"><span data-stu-id="b9189-218">60 sec</span></span> |<span data-ttu-id="b9189-219">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="b9189-219">ExponentialBackoff</span></span> |<span data-ttu-id="b9189-220">Contagem de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-220">Retry count</span></span><br /><span data-ttu-id="b9189-221">Retirada mín.</span><span class="sxs-lookup"><span data-stu-id="b9189-221">Min back-off</span></span><br /><span data-ttu-id="b9189-222">Retirada máx.</span><span class="sxs-lookup"><span data-stu-id="b9189-222">Max back-off</span></span><br /><span data-ttu-id="b9189-223">Retirada delta</span><span class="sxs-lookup"><span data-stu-id="b9189-223">Delta back-off</span></span><br /><span data-ttu-id="b9189-224">Primeira repetição rápida</span><span class="sxs-lookup"><span data-stu-id="b9189-224">First fast retry</span></span> |<span data-ttu-id="b9189-225">5</span><span class="sxs-lookup"><span data-stu-id="b9189-225">5</span></span><br /><span data-ttu-id="b9189-226">0 s</span><span class="sxs-lookup"><span data-stu-id="b9189-226">0 sec</span></span><br /><span data-ttu-id="b9189-227">60 s</span><span class="sxs-lookup"><span data-stu-id="b9189-227">60 sec</span></span><br /><span data-ttu-id="b9189-228">2 s</span><span class="sxs-lookup"><span data-stu-id="b9189-228">2 sec</span></span><br /><span data-ttu-id="b9189-229">falso</span><span class="sxs-lookup"><span data-stu-id="b9189-229">false</span></span> |<span data-ttu-id="b9189-230">1ª tentativa — intervalo de 0 s</span><span class="sxs-lookup"><span data-stu-id="b9189-230">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="b9189-231">2ª tentativa — intervalo de ~2 s</span><span class="sxs-lookup"><span data-stu-id="b9189-231">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="b9189-232">3ª tentativa — intervalo de ~6 s</span><span class="sxs-lookup"><span data-stu-id="b9189-232">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="b9189-233">4ª tentativa — intervalo de ~14 s</span><span class="sxs-lookup"><span data-stu-id="b9189-233">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="b9189-234">5ª tentativa — intervalo de ~30 s</span><span class="sxs-lookup"><span data-stu-id="b9189-234">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="b9189-235">Mais informações</span><span class="sxs-lookup"><span data-stu-id="b9189-235">More information</span></span>
* <span data-ttu-id="b9189-236">[Bibliotecas de autenticação do Azure Active Directory][adal]</span><span class="sxs-lookup"><span data-stu-id="b9189-236">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="cosmos-db"></a><span data-ttu-id="b9189-237">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="b9189-237">Cosmos DB</span></span>

<span data-ttu-id="b9189-238">O Cosmos DB é um banco de dados multimodelo totalmente gerenciado que oferece suporte a dados JSON sem esquema.</span><span class="sxs-lookup"><span data-stu-id="b9189-238">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="b9189-239">Ele oferece desempenho configurável e confiável, processamento transacional nativo JavaScript, além de ser criado para a nuvem com escala elástica.</span><span class="sxs-lookup"><span data-stu-id="b9189-239">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="b9189-240">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-240">Retry mechanism</span></span>
<span data-ttu-id="b9189-241">A classe `DocumentClient` repete automaticamente tentativas com falha.</span><span class="sxs-lookup"><span data-stu-id="b9189-241">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="b9189-242">Para definir o número de repetições e o tempo de espera máximo, configure [ConnectionPolicy.RetryOptions].</span><span class="sxs-lookup"><span data-stu-id="b9189-242">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="b9189-243">As exceções que o cliente gera excedem a política de repetição ou não são erros transitórios.</span><span class="sxs-lookup"><span data-stu-id="b9189-243">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="b9189-244">Se o Cosmos DB limitar o cliente, ele retornará um erro HTTP 429.</span><span class="sxs-lookup"><span data-stu-id="b9189-244">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="b9189-245">Confira o código de status no `DocumentClientException`.</span><span class="sxs-lookup"><span data-stu-id="b9189-245">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="b9189-246">Configuração de política</span><span class="sxs-lookup"><span data-stu-id="b9189-246">Policy configuration</span></span>
<span data-ttu-id="b9189-247">A tabela a seguir mostra as configurações padrão para a classe `RetryOptions`.</span><span class="sxs-lookup"><span data-stu-id="b9189-247">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="b9189-248">Configuração</span><span class="sxs-lookup"><span data-stu-id="b9189-248">Setting</span></span> | <span data-ttu-id="b9189-249">Valor padrão</span><span class="sxs-lookup"><span data-stu-id="b9189-249">Default value</span></span> | <span data-ttu-id="b9189-250">DESCRIÇÃO</span><span class="sxs-lookup"><span data-stu-id="b9189-250">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="b9189-251">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="b9189-251">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="b9189-252">9</span><span class="sxs-lookup"><span data-stu-id="b9189-252">9</span></span> |<span data-ttu-id="b9189-253">O número máximo de tentativas se a solicitação falhar porque o Cosmos DB aplicou a limitação de taxa ao cliente.</span><span class="sxs-lookup"><span data-stu-id="b9189-253">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="b9189-254">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="b9189-254">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="b9189-255">30</span><span class="sxs-lookup"><span data-stu-id="b9189-255">30</span></span> |<span data-ttu-id="b9189-256">O tempo máximo de repetição, em segundos.</span><span class="sxs-lookup"><span data-stu-id="b9189-256">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="b9189-257">Exemplo</span><span class="sxs-lookup"><span data-stu-id="b9189-257">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="b9189-258">Telemetria</span><span class="sxs-lookup"><span data-stu-id="b9189-258">Telemetry</span></span>
<span data-ttu-id="b9189-259">As tentativas de repetição são registradas como mensagens de rastreamento não estruturadas por meio de um **TraceSource**do .NET.</span><span class="sxs-lookup"><span data-stu-id="b9189-259">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="b9189-260">Você deve configurar um **TraceListener** para capturar os eventos e gravá-los em um log de destino apropriado.</span><span class="sxs-lookup"><span data-stu-id="b9189-260">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="b9189-261">Por exemplo, se você adicionar o seguinte ao arquivo App.config, rastreamentos serão gerados em um arquivo de texto no mesmo local que o executável:</span><span class="sxs-lookup"><span data-stu-id="b9189-261">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

```xml
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```

## <a name="event-hubs"></a><span data-ttu-id="b9189-262">Hubs de Eventos</span><span class="sxs-lookup"><span data-stu-id="b9189-262">Event Hubs</span></span>

<span data-ttu-id="b9189-263">Os Hubs de Eventos do Azure são um serviço de ingestão de telemetria de hiperescala que coleta, transforma e armazena milhões de eventos.</span><span class="sxs-lookup"><span data-stu-id="b9189-263">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="b9189-264">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-264">Retry mechanism</span></span>
<span data-ttu-id="b9189-265">O comportamento de repetição na biblioteca de cliente dos Hubs de Eventos do Azure é controlado pela propriedade `RetryPolicy` na classe `EventHubClient`.</span><span class="sxs-lookup"><span data-stu-id="b9189-265">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="b9189-266">As tentativas de política padrão com retirada exponencial quando o Hub de eventos do Azure retorna um `EventHubsException` transitório ou um `OperationCanceledException`.</span><span class="sxs-lookup"><span data-stu-id="b9189-266">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="b9189-267">Exemplo</span><span class="sxs-lookup"><span data-stu-id="b9189-267">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="b9189-268">Mais informações</span><span class="sxs-lookup"><span data-stu-id="b9189-268">More information</span></span>
[<span data-ttu-id="b9189-269">Biblioteca de cliente padrão .NET para Hubs de Eventos do Azure</span><span class="sxs-lookup"><span data-stu-id="b9189-269"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="iot-hub"></a><span data-ttu-id="b9189-270">Hub IoT</span><span class="sxs-lookup"><span data-stu-id="b9189-270">IoT Hub</span></span>

<span data-ttu-id="b9189-271">O Hub IoT do Azure é um serviço para conectar, monitorar e gerenciar dispositivos para desenvolver aplicativos IoT (Internet das Coisas).</span><span class="sxs-lookup"><span data-stu-id="b9189-271">Azure IoT Hub is a service for connecting, monitoring, and managing devices to develop Internet of Things (IoT) applications.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="b9189-272">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-272">Retry mechanism</span></span>

<span data-ttu-id="b9189-273">O SDK do dispositivo IoT do Azure pode detectar erros na rede, protocolo ou aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b9189-273">The Azure IoT device SDK can detect errors in the network, protocol, or application.</span></span> <span data-ttu-id="b9189-274">Com base no tipo de erro, o SDK verifica se uma nova tentativa deve ser executada.</span><span class="sxs-lookup"><span data-stu-id="b9189-274">Based on the error type, the SDK checks whether a retry needs to be performed.</span></span> <span data-ttu-id="b9189-275">Se o erro for *recuperável*, o SDK tentará novamente usando a política de repetição configurada.</span><span class="sxs-lookup"><span data-stu-id="b9189-275">If the error is *recoverable*, the SDK begins to retry using the configured retry policy.</span></span>

<span data-ttu-id="b9189-276">A política de repetição padrão é uma *retirada exponencial com variação aleatória*, mas ela pode ser configurada.</span><span class="sxs-lookup"><span data-stu-id="b9189-276">The default retry policy is *exponential back-off with random jitter*, but it can be configured.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="b9189-277">Configuração de política</span><span class="sxs-lookup"><span data-stu-id="b9189-277">Policy configuration</span></span>

<span data-ttu-id="b9189-278">A configuração de política varia segundo o idioma.</span><span class="sxs-lookup"><span data-stu-id="b9189-278">Policy configuration differs by language.</span></span> <span data-ttu-id="b9189-279">Para obter mais detalhes, consulte [Configuração da política de repetição do Hub IoT](/azure/iot-hub/iot-hub-reliability-features-in-sdks#retry-policy-apis).</span><span class="sxs-lookup"><span data-stu-id="b9189-279">For more details, see [IoT Hub retry policy configuration](/azure/iot-hub/iot-hub-reliability-features-in-sdks#retry-policy-apis).</span></span>

### <a name="more-information"></a><span data-ttu-id="b9189-280">Mais informações</span><span class="sxs-lookup"><span data-stu-id="b9189-280">More information</span></span>

* [<span data-ttu-id="b9189-281">Política de repetição do Hub IoT</span><span class="sxs-lookup"><span data-stu-id="b9189-281">IoT Hub retry policy</span></span>](/azure/iot-hub/iot-hub-reliability-features-in-sdks)
* [<span data-ttu-id="b9189-282">Solucionar problemas de desconexão do dispositivo do IoT Hub</span><span class="sxs-lookup"><span data-stu-id="b9189-282">Troubleshoot IoT Hub device disconnection</span></span>](/azure/iot-hub/iot-hub-troubleshoot-connectivity)

## <a name="azure-redis-cache"></a><span data-ttu-id="b9189-283">Cache Redis do Azure</span><span class="sxs-lookup"><span data-stu-id="b9189-283">Azure Redis Cache</span></span>
<span data-ttu-id="b9189-284">O Cache Redis do Azure é um serviço de cache de baixa latência e acesso a dados rápido com base no Cache Redis de software livre popular.</span><span class="sxs-lookup"><span data-stu-id="b9189-284">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="b9189-285">Ele é seguro, gerenciado pela Microsoft e pode ser acessado de qualquer aplicativo no Azure.</span><span class="sxs-lookup"><span data-stu-id="b9189-285">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="b9189-286">A diretriz nesta seção se baseia em como usar o cliente StackExchange.Redis para acessar o cache.</span><span class="sxs-lookup"><span data-stu-id="b9189-286">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="b9189-287">Uma lista de outros clientes adequados pode ser encontrada no [site do Redis](https://redis.io/clients), e eles podem ter mecanismos de repetição diferentes.</span><span class="sxs-lookup"><span data-stu-id="b9189-287">A list of other suitable clients can be found on the [Redis website](https://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="b9189-288">Observe que o cliente StackExchange.Redis usa multiplexação por meio de uma única conexão.</span><span class="sxs-lookup"><span data-stu-id="b9189-288">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="b9189-289">O uso recomendado é criar uma instância do cliente na inicialização do aplicativo e usar essa instância para todas as operações no cache.</span><span class="sxs-lookup"><span data-stu-id="b9189-289">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="b9189-290">Por esse motivo, a conexão com o cache é feita apenas uma vez, e toda a orientação desta seção está relacionada à política de repetição para esta conexão inicial, e não para cada operação que acessa o cache.</span><span class="sxs-lookup"><span data-stu-id="b9189-290">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="b9189-291">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-291">Retry mechanism</span></span>
<span data-ttu-id="b9189-292">O cliente StackExchange.Redis usa uma classe de gerenciador de conexões que é configurada por meio de um conjunto de opções, incluindo:</span><span class="sxs-lookup"><span data-stu-id="b9189-292">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, including:</span></span>

- <span data-ttu-id="b9189-293">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="b9189-293">**ConnectRetry**.</span></span> <span data-ttu-id="b9189-294">O número de vezes que uma conexão com falha para o cache será repetida.</span><span class="sxs-lookup"><span data-stu-id="b9189-294">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="b9189-295">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="b9189-295">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="b9189-296">Usar qual estratégia de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-296">The retry strategy to use.</span></span>
- <span data-ttu-id="b9189-297">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="b9189-297">**ConnectTimeout**.</span></span> <span data-ttu-id="b9189-298">O tempo máximo de espera em milissegundos.</span><span class="sxs-lookup"><span data-stu-id="b9189-298">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="b9189-299">Configuração de política</span><span class="sxs-lookup"><span data-stu-id="b9189-299">Policy configuration</span></span>
<span data-ttu-id="b9189-300">As políticas de repetição são configuradas de modo programático, definindo as opções para o cliente antes de se conectar ao cache.</span><span class="sxs-lookup"><span data-stu-id="b9189-300">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="b9189-301">Isso pode ser feito criando uma instância da classe **ConfigurationOptions**, populando suas propriedades e passando-a para o método **Connect**.</span><span class="sxs-lookup"><span data-stu-id="b9189-301">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="b9189-302">As classes internas suportam atraso linear (constante) e retirada exponencial com intervalos de repetição aleatórios.</span><span class="sxs-lookup"><span data-stu-id="b9189-302">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="b9189-303">Você também pode criar uma política de repetição personalizada se implementar a interface **IReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="b9189-303">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="b9189-304">O exemplo a seguir configura uma estratégia de repetição com retirada exponencial.</span><span class="sxs-lookup"><span data-stu-id="b9189-304">The following example configures a retry strategy using exponential backoff.</span></span>

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="b9189-305">Como alternativa, você pode especificar as opções como uma cadeia de caracteres e passá-la ao método **Connect**.</span><span class="sxs-lookup"><span data-stu-id="b9189-305">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="b9189-306">Observe que a propriedade **ReconnectRetryPolicy** não pode ser definida dessa forma, somente através de código.</span><span class="sxs-lookup"><span data-stu-id="b9189-306">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="b9189-307">Também é possível especificar opções diretamente ao se conectar ao cache.</span><span class="sxs-lookup"><span data-stu-id="b9189-307">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="b9189-308">Para obter mais informações, confira [Configuração do Stack Exchange Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration) na documentação do StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="b9189-308">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="b9189-309">A tabela a seguir mostra as configurações padrão da política de repetição interna.</span><span class="sxs-lookup"><span data-stu-id="b9189-309">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="b9189-310">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="b9189-310">**Context**</span></span> | <span data-ttu-id="b9189-311">**Configuração**</span><span class="sxs-lookup"><span data-stu-id="b9189-311">**Setting**</span></span> | <span data-ttu-id="b9189-312">**Valor padrão**</span><span class="sxs-lookup"><span data-stu-id="b9189-312">**Default value**</span></span><br /><span data-ttu-id="b9189-313">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="b9189-313">(v 1.2.2)</span></span> | <span data-ttu-id="b9189-314">**Significado**</span><span class="sxs-lookup"><span data-stu-id="b9189-314">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="b9189-315">ConfigurationOptions</span><span class="sxs-lookup"><span data-stu-id="b9189-315">ConfigurationOptions</span></span> |<span data-ttu-id="b9189-316">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="b9189-316">ConnectRetry</span></span><br /><br /><span data-ttu-id="b9189-317">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="b9189-317">ConnectTimeout</span></span><br /><br /><span data-ttu-id="b9189-318">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="b9189-318">SyncTimeout</span></span><br /><br /><span data-ttu-id="b9189-319">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="b9189-319">ReconnectRetryPolicy</span></span> |<span data-ttu-id="b9189-320">3</span><span class="sxs-lookup"><span data-stu-id="b9189-320">3</span></span><br /><br /><span data-ttu-id="b9189-321">Máximo de 5000 ms mais SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="b9189-321">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="b9189-322">1000</span><span class="sxs-lookup"><span data-stu-id="b9189-322">1000</span></span><br /><br /><span data-ttu-id="b9189-323">LinearRetry 5.000 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-323">LinearRetry 5000 ms</span></span> |<span data-ttu-id="b9189-324">O número de vezes para repetir tentativas de conexão durante a operação de conexão inicial.</span><span class="sxs-lookup"><span data-stu-id="b9189-324">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="b9189-325">Tempo limite (ms) para operações de conexão.</span><span class="sxs-lookup"><span data-stu-id="b9189-325">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="b9189-326">Não é um intervalo entre tentativas de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-326">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="b9189-327">Tempo (ms) para permitir operações síncronas.</span><span class="sxs-lookup"><span data-stu-id="b9189-327">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="b9189-328">Repita a cada 5.000 ms.</span><span class="sxs-lookup"><span data-stu-id="b9189-328">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="b9189-329">Em operações síncronas, `SyncTimeout` pode adicionar à latência de ponta a ponta, porém definir o valor muito baixo pode causar tempos limite excessivos.</span><span class="sxs-lookup"><span data-stu-id="b9189-329">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="b9189-330">Confira [Como solucionar problemas do Cache Redis do Azure][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="b9189-330">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="b9189-331">Em geral, evite usar operações síncronas e use operações assíncronas.</span><span class="sxs-lookup"><span data-stu-id="b9189-331">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="b9189-332">Para saber mais, consulte [Pipelines e multiplexadores](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="b9189-332">For more information see [Pipelines and Multiplexers](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="b9189-333">Diretriz de uso de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-333">Retry usage guidance</span></span>
<span data-ttu-id="b9189-334">Considere as seguintes diretrizes ao usar o Cache Redis do Azure:</span><span class="sxs-lookup"><span data-stu-id="b9189-334">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="b9189-335">O cliente StackExchange Redis gerencia suas próprias repetições, mas apenas ao estabelecer uma conexão com o cache quando o aplicativo é iniciado pela primeira vez.</span><span class="sxs-lookup"><span data-stu-id="b9189-335">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="b9189-336">Você pode configurar o tempo limite da conexão, o número de repetições e o intervalo entre as repetições para estabelecer esta conexão, porém a política de repetições não se aplica a operações contra o cache.</span><span class="sxs-lookup"><span data-stu-id="b9189-336">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="b9189-337">Em vez de usar um grande número de tentativas de repetição, considere fazer fallback acessando a fonte de dados original.</span><span class="sxs-lookup"><span data-stu-id="b9189-337">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="b9189-338">Telemetria</span><span class="sxs-lookup"><span data-stu-id="b9189-338">Telemetry</span></span>
<span data-ttu-id="b9189-339">Você pode coletar informações sobre conexões (mas não outras operações) usando um **TextWriter**.</span><span class="sxs-lookup"><span data-stu-id="b9189-339">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="b9189-340">Um exemplo da saída que isso gera é mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="b9189-340">An example of the output this generates is shown below.</span></span>

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a><span data-ttu-id="b9189-341">Exemplos</span><span class="sxs-lookup"><span data-stu-id="b9189-341">Examples</span></span>
<span data-ttu-id="b9189-342">O exemplo de código a seguir configura um atraso constante (linear) entre as repetições ao inicializar o cliente StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="b9189-342">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="b9189-343">Este exemplo mostra como definir a configuração usando uma instância de **ConfigurationOptions**.</span><span class="sxs-lookup"><span data-stu-id="b9189-343">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries

                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="b9189-344">Este exemplo mostra como definir a configuração especificando as opções como uma cadeia de caracteres.</span><span class="sxs-lookup"><span data-stu-id="b9189-344">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="b9189-345">O tempo limite da conexão é o período máximo de espera por uma conexão para o cache, não o atraso entre tentativas de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-345">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="b9189-346">Observe que a propriedade **ReconnectRetryPolicy** só pode ser definida por código.</span><span class="sxs-lookup"><span data-stu-id="b9189-346">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="b9189-347">Para obter mais exemplos, consulte [Configuração](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Configuration.md) no site do projeto.</span><span class="sxs-lookup"><span data-stu-id="b9189-347">For more examples, see [Configuration](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Configuration.md) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="b9189-348">Mais informações</span><span class="sxs-lookup"><span data-stu-id="b9189-348">More information</span></span>
* [<span data-ttu-id="b9189-349">Site do Redis</span><span class="sxs-lookup"><span data-stu-id="b9189-349">Redis website</span></span>](https://redis.io/)

## <a name="azure-search"></a><span data-ttu-id="b9189-350">Azure Search</span><span class="sxs-lookup"><span data-stu-id="b9189-350">Azure Search</span></span>
<span data-ttu-id="b9189-351">O Azure Search pode ser usada para adicionar recursos potentes e sofisticados a um site ou aplicativo, ajustar os resultados da pesquisa de maneira rápida e fácil, bem como construir modelos de classificação avançados e ajustados.</span><span class="sxs-lookup"><span data-stu-id="b9189-351">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="b9189-352">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-352">Retry mechanism</span></span>
<span data-ttu-id="b9189-353">O comportamento de repetição no SDK do Azure Search é controlado pelo método `SetRetryPolicy` nas classes [SearchServiceClient] e [SearchIndexClient].</span><span class="sxs-lookup"><span data-stu-id="b9189-353">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="b9189-354">A política padrão tenta novamente com retirada exponencial quando o Azure Search retorna uma resposta 5xx ou 408 (Tempo Limite da Solicitação).</span><span class="sxs-lookup"><span data-stu-id="b9189-354">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="b9189-355">Telemetria</span><span class="sxs-lookup"><span data-stu-id="b9189-355">Telemetry</span></span>
<span data-ttu-id="b9189-356">Rastreamento com o ETW ou por meio de registro de um provedor de rastreamento personalizado.</span><span class="sxs-lookup"><span data-stu-id="b9189-356">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="b9189-357">Para obter mais informações, confira [Documentação do AutoRest][autorest].</span><span class="sxs-lookup"><span data-stu-id="b9189-357">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="service-bus"></a><span data-ttu-id="b9189-358">Barramento de Serviço</span><span class="sxs-lookup"><span data-stu-id="b9189-358">Service Bus</span></span>
<span data-ttu-id="b9189-359">O Barramento de Serviço é uma plataforma de mensagens na nuvem que fornece troca de mensagens flexivelmente acoplada com escala e resiliência melhoradas para componentes de um aplicativo, se hospedado na nuvem ou no local.</span><span class="sxs-lookup"><span data-stu-id="b9189-359">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="b9189-360">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-360">Retry mechanism</span></span>
<span data-ttu-id="b9189-361">O Barramento de Serviço implementa repetições usando implementações da classe base [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) .</span><span class="sxs-lookup"><span data-stu-id="b9189-361">Service Bus implements retries using implementations of the [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) base class.</span></span> <span data-ttu-id="b9189-362">Todos os clientes do Barramento de Serviço expõem uma propriedade **RetryPolicy** que pode ser definida como uma das implementações da classe base **RetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="b9189-362">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="b9189-363">As implementações internas são:</span><span class="sxs-lookup"><span data-stu-id="b9189-363">The built-in implementations are:</span></span>

* <span data-ttu-id="b9189-364">A [classe RetryExponential](/dotnet/api/microsoft.servicebus.retryexponential).</span><span class="sxs-lookup"><span data-stu-id="b9189-364">The [RetryExponential Class](/dotnet/api/microsoft.servicebus.retryexponential).</span></span> <span data-ttu-id="b9189-365">Ela expõe propriedades que controlam o intervalo de retirada, a contagem de repetições e a propriedade **TerminationTimeBuffer** que é usada para limitar o tempo total para a operação ser concluída.</span><span class="sxs-lookup"><span data-stu-id="b9189-365">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="b9189-366">A [classe NoRetry](/dotnet/api/microsoft.servicebus.noretry).</span><span class="sxs-lookup"><span data-stu-id="b9189-366">The [NoRetry Class](/dotnet/api/microsoft.servicebus.noretry).</span></span> <span data-ttu-id="b9189-367">É usada quando as repetições no nível de API do Barramento de Serviço não são exigidas, como quando são gerenciadas por outro processo como parte de uma operação em lote ou de várias etapas.</span><span class="sxs-lookup"><span data-stu-id="b9189-367">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="b9189-368">As ações do Barramento de Serviço podem retornar várias exceções, conforme listado em [Exceções de mensagem do Barramento de Serviço](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span><span class="sxs-lookup"><span data-stu-id="b9189-368">Service Bus actions can return a range of exceptions, as listed in [Service Bus messaging exceptions](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span></span> <span data-ttu-id="b9189-369">A lista fornece informações que indicam se repetir a operação é adequado.</span><span class="sxs-lookup"><span data-stu-id="b9189-369">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="b9189-370">Por exemplo, um **ServerBusyException** indica que o cliente deve esperar um período de tempo, depois repetir a operação.</span><span class="sxs-lookup"><span data-stu-id="b9189-370">For example, a **ServerBusyException** indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="b9189-371">A ocorrência de um **ServerBusyException** também faz com que o Barramento de Serviço alterne para um modo diferente, no qual um intervalo extra de 10 segundos é adicionado aos intervalos de repetição computados.</span><span class="sxs-lookup"><span data-stu-id="b9189-371">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="b9189-372">Esse modo é redefinido após um curto período.</span><span class="sxs-lookup"><span data-stu-id="b9189-372">This mode is reset after a short period.</span></span>

<span data-ttu-id="b9189-373">As exceções retornadas do Barramento de Serviço expõem a propriedade **IsTransient** que indica se o cliente deve repetir a operação.</span><span class="sxs-lookup"><span data-stu-id="b9189-373">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="b9189-374">A política interna **RetryExponential** depende da propriedade **IsTransient** na classe **MessagingException**, que é a classe base para todas as exceções do Barramento de Serviço.</span><span class="sxs-lookup"><span data-stu-id="b9189-374">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="b9189-375">Se você criar implementações personalizadas da classe base **RetryPolicy**, será possível usar uma combinação do tipo de exceção e da propriedade **IsTransient** para fornecer um controle mais refinado sobre as ações de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-375">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="b9189-376">Por exemplo, é possível detectar um **QuotaExceededException** e tomar providências para diminuir a fila antes de tentar enviar novamente uma mensagem para ela.</span><span class="sxs-lookup"><span data-stu-id="b9189-376">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="b9189-377">Configuração de política</span><span class="sxs-lookup"><span data-stu-id="b9189-377">Policy configuration</span></span>
<span data-ttu-id="b9189-378">As políticas de repetição são definidas de modo programático e como uma política padrão para um **NamespaceManager** e uma **MessagingFactory** ou individualmente para cada cliente de mensagens.</span><span class="sxs-lookup"><span data-stu-id="b9189-378">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="b9189-379">Para definir a política de repetição padrão para uma sessão de mensagens, defina **RetryPolicy** do **NamespaceManager**.</span><span class="sxs-lookup"><span data-stu-id="b9189-379">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

<span data-ttu-id="b9189-380">Para definir a política de repetição padrão para todos os clientes criados de uma fábrica de mensagens, defina **RetryPolicy** da **MessagingFactory**.</span><span class="sxs-lookup"><span data-stu-id="b9189-380">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

<span data-ttu-id="b9189-381">Para definir a política de repetição para um cliente de mensagens, ou para substituir sua política padrão, defina a propriedade **RetryPolicy** usando uma instância da classe de política necessária:</span><span class="sxs-lookup"><span data-stu-id="b9189-381">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="b9189-382">A política de repetição não pode ser definida no nível de operação individual.</span><span class="sxs-lookup"><span data-stu-id="b9189-382">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="b9189-383">Ela se aplica a todas as operações do cliente de mensagens.</span><span class="sxs-lookup"><span data-stu-id="b9189-383">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="b9189-384">A tabela a seguir mostra as configurações padrão da política de repetição interna.</span><span class="sxs-lookup"><span data-stu-id="b9189-384">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="b9189-385">Configuração</span><span class="sxs-lookup"><span data-stu-id="b9189-385">Setting</span></span> | <span data-ttu-id="b9189-386">Valor padrão</span><span class="sxs-lookup"><span data-stu-id="b9189-386">Default value</span></span> | <span data-ttu-id="b9189-387">Significado</span><span class="sxs-lookup"><span data-stu-id="b9189-387">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="b9189-388">Política</span><span class="sxs-lookup"><span data-stu-id="b9189-388">Policy</span></span> | <span data-ttu-id="b9189-389">Exponencial</span><span class="sxs-lookup"><span data-stu-id="b9189-389">Exponential</span></span> | <span data-ttu-id="b9189-390">Retirada exponencial.</span><span class="sxs-lookup"><span data-stu-id="b9189-390">Exponential back-off.</span></span> |
| <span data-ttu-id="b9189-391">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="b9189-391">MinimalBackoff</span></span> | <span data-ttu-id="b9189-392">0</span><span class="sxs-lookup"><span data-stu-id="b9189-392">0</span></span> | <span data-ttu-id="b9189-393">O intervalo mínimo de retirada.</span><span class="sxs-lookup"><span data-stu-id="b9189-393">Minimum back-off interval.</span></span> <span data-ttu-id="b9189-394">Isso é adicionado ao intervalo de repetição calculados de deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="b9189-394">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="b9189-395">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="b9189-395">MaximumBackoff</span></span> | <span data-ttu-id="b9189-396">30 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-396">30 seconds</span></span> | <span data-ttu-id="b9189-397">O intervalo máximo de retirada.</span><span class="sxs-lookup"><span data-stu-id="b9189-397">Maximum back-off interval.</span></span> <span data-ttu-id="b9189-398">MaximumBackoff será usado se o intervalo de repetição calculado for maior que MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="b9189-398">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="b9189-399">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="b9189-399">DeltaBackoff</span></span> | <span data-ttu-id="b9189-400">3 Segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-400">3 seconds</span></span> | <span data-ttu-id="b9189-401">Intervalo de retirada entre repetições.</span><span class="sxs-lookup"><span data-stu-id="b9189-401">Back-off interval between retries.</span></span> <span data-ttu-id="b9189-402">Múltiplos desse período de tempo serão usados para tentativas de repetição subsequentes.</span><span class="sxs-lookup"><span data-stu-id="b9189-402">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="b9189-403">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="b9189-403">TimeBuffer</span></span> | <span data-ttu-id="b9189-404">5 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-404">5 seconds</span></span> | <span data-ttu-id="b9189-405">O buffer de tempo de encerramento associado à repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-405">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="b9189-406">Tentativas de repetição serão abandonadas se o tempo restante for menor do que o TimeBuffer.</span><span class="sxs-lookup"><span data-stu-id="b9189-406">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="b9189-407">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="b9189-407">MaxRetryCount</span></span> | <span data-ttu-id="b9189-408">10</span><span class="sxs-lookup"><span data-stu-id="b9189-408">10</span></span> | <span data-ttu-id="b9189-409">O número máximo de repetições.</span><span class="sxs-lookup"><span data-stu-id="b9189-409">The maximum number of retries.</span></span> |
| <span data-ttu-id="b9189-410">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="b9189-410">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="b9189-411">10 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-411">10 seconds</span></span> | <span data-ttu-id="b9189-412">Se a última exceção encontrada for **ServerBusyException**, esse valor será adicionado ao intervalo de repetição calculado.</span><span class="sxs-lookup"><span data-stu-id="b9189-412">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="b9189-413">Esse valor não pode ser alterado.</span><span class="sxs-lookup"><span data-stu-id="b9189-413">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="b9189-414">Diretriz de uso de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-414">Retry usage guidance</span></span>
<span data-ttu-id="b9189-415">Considere as seguintes diretrizes ao usar o Barramento de Serviço:</span><span class="sxs-lookup"><span data-stu-id="b9189-415">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="b9189-416">Ao usar a implementação **RetryExponential** interna, não implemente uma operação de fallback, pois a política reage às exceções de Servidor Ocupado e alterna automaticamente para um modo de repetição apropriado.</span><span class="sxs-lookup"><span data-stu-id="b9189-416">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="b9189-417">O Barramento de Serviço oferece suporte a um recurso chamado Namespaces Emparelhados, que implementa failover automático para uma fila de backup em um namespace separado, caso a fila no namespace primário falhe.</span><span class="sxs-lookup"><span data-stu-id="b9189-417">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="b9189-418">As mensagens da fila secundária podem ser enviadas de volta para a fila primária quando esta se recuperar.</span><span class="sxs-lookup"><span data-stu-id="b9189-418">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="b9189-419">Esse recurso ajuda a corrigir falhas transitórias.</span><span class="sxs-lookup"><span data-stu-id="b9189-419">This feature helps to address transient failures.</span></span> <span data-ttu-id="b9189-420">Para saber mais, consulte [Padrões de mensagens assíncronas e alta disponibilidade](/azure/service-bus-messaging/service-bus-async-messaging).</span><span class="sxs-lookup"><span data-stu-id="b9189-420">For more information, see [Asynchronous Messaging Patterns and High Availability](/azure/service-bus-messaging/service-bus-async-messaging).</span></span>

<span data-ttu-id="b9189-421">Considere começar com as seguintes configurações para operações de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-421">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="b9189-422">Essas são configurações de uso geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.</span><span class="sxs-lookup"><span data-stu-id="b9189-422">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="b9189-423">Contexto</span><span class="sxs-lookup"><span data-stu-id="b9189-423">Context</span></span> | <span data-ttu-id="b9189-424">Exemplo de latência máxima</span><span class="sxs-lookup"><span data-stu-id="b9189-424">Example maximum latency</span></span> | <span data-ttu-id="b9189-425">Política de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-425">Retry policy</span></span> | <span data-ttu-id="b9189-426">Configurações</span><span class="sxs-lookup"><span data-stu-id="b9189-426">Settings</span></span> | <span data-ttu-id="b9189-427">Como ele funciona</span><span class="sxs-lookup"><span data-stu-id="b9189-427">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="b9189-428">Interativo, interface do usuário ou primeiro plano</span><span class="sxs-lookup"><span data-stu-id="b9189-428">Interactive, UI, or foreground</span></span> | <span data-ttu-id="b9189-429">2 segundos\*</span><span class="sxs-lookup"><span data-stu-id="b9189-429">2 seconds\*</span></span>  | <span data-ttu-id="b9189-430">Exponencial</span><span class="sxs-lookup"><span data-stu-id="b9189-430">Exponential</span></span> | <span data-ttu-id="b9189-431">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="b9189-431">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="b9189-432">MaximumBackoff = 30 s</span><span class="sxs-lookup"><span data-stu-id="b9189-432">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="b9189-433">DeltaBackoff = 300 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-433">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="b9189-434">TimeBuffer = 300 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-434">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="b9189-435">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="b9189-435">MaxRetryCount = 2</span></span> | <span data-ttu-id="b9189-436">1ª tentativa: atraso 0 s</span><span class="sxs-lookup"><span data-stu-id="b9189-436">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="b9189-437">2ª tentativa: atraso ~300 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-437">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="b9189-438">3ª tentativa: atraso ~900 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-438">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="b9189-439">Segundo plano ou lote</span><span class="sxs-lookup"><span data-stu-id="b9189-439">Background or batch</span></span> | <span data-ttu-id="b9189-440">30 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-440">30 seconds</span></span> | <span data-ttu-id="b9189-441">Exponencial</span><span class="sxs-lookup"><span data-stu-id="b9189-441">Exponential</span></span> | <span data-ttu-id="b9189-442">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="b9189-442">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="b9189-443">MaximumBackoff = 30 s</span><span class="sxs-lookup"><span data-stu-id="b9189-443">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="b9189-444">DeltaBackoff = 1,75 seg.</span><span class="sxs-lookup"><span data-stu-id="b9189-444">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="b9189-445">TimeBuffer = 5 seg.</span><span class="sxs-lookup"><span data-stu-id="b9189-445">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="b9189-446">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="b9189-446">MaxRetryCount = 3</span></span> | <span data-ttu-id="b9189-447">1ª tentativa: atraso de ~1 s</span><span class="sxs-lookup"><span data-stu-id="b9189-447">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="b9189-448">2ª tentativa: atraso de ~3 s</span><span class="sxs-lookup"><span data-stu-id="b9189-448">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="b9189-449">3ª tentativa: atraso ~6 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-449">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="b9189-450">4ª tentativa: atraso ~13 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-450">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="b9189-451">\*Não incluindo atraso que será adicionado se uma resposta Servidor Ocupado for recebida.</span><span class="sxs-lookup"><span data-stu-id="b9189-451">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="b9189-452">Telemetria</span><span class="sxs-lookup"><span data-stu-id="b9189-452">Telemetry</span></span>
<span data-ttu-id="b9189-453">O Barramento de Serviço registra as repetições como eventos ETW usando um **EventSource**.</span><span class="sxs-lookup"><span data-stu-id="b9189-453">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="b9189-454">Você deve anexar um **EventListener** à origem do evento para capturar os eventos e exibi-los no Visualizador de Desempenho ou gravá-los em um log de destino apropriado.</span><span class="sxs-lookup"><span data-stu-id="b9189-454">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="b9189-455">Os eventos de repetição são da seguinte forma:</span><span class="sxs-lookup"><span data-stu-id="b9189-455">The retry events are of the following form:</span></span>

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a><span data-ttu-id="b9189-456">Exemplos</span><span class="sxs-lookup"><span data-stu-id="b9189-456">Examples</span></span>
<span data-ttu-id="b9189-457">O código de exemplo a seguir mostra como definir a política de repetição para:</span><span class="sxs-lookup"><span data-stu-id="b9189-457">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="b9189-458">Um gerenciador de namespaces.</span><span class="sxs-lookup"><span data-stu-id="b9189-458">A namespace manager.</span></span> <span data-ttu-id="b9189-459">A política se aplica a todas as operações nesse gerenciador e não pode ser substituída para operações individuais.</span><span class="sxs-lookup"><span data-stu-id="b9189-459">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="b9189-460">Uma fábrica de mensagens.</span><span class="sxs-lookup"><span data-stu-id="b9189-460">A messaging factory.</span></span> <span data-ttu-id="b9189-461">A política se aplica a todos os clientes criados nessa fábrica e não pode ser substituída durante a criação de clientes individuais.</span><span class="sxs-lookup"><span data-stu-id="b9189-461">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="b9189-462">Um cliente de mensagens individual.</span><span class="sxs-lookup"><span data-stu-id="b9189-462">An individual messaging client.</span></span> <span data-ttu-id="b9189-463">Depois que um cliente tiver sido criado, você poderá definir a política de repetição para esse cliente.</span><span class="sxs-lookup"><span data-stu-id="b9189-463">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="b9189-464">A política se aplica a todas as operações nesse cliente.</span><span class="sxs-lookup"><span data-stu-id="b9189-464">The policy applies to all operations on that client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }

            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }

            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="b9189-465">Mais informações</span><span class="sxs-lookup"><span data-stu-id="b9189-465">More information</span></span>
* [<span data-ttu-id="b9189-466">Padrões de mensagens assíncronas e alta disponibilidade</span><span class="sxs-lookup"><span data-stu-id="b9189-466">Asynchronous Messaging Patterns and High Availability</span></span>](/azure/service-bus-messaging/service-bus-async-messaging)

## <a name="service-fabric"></a><span data-ttu-id="b9189-467">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="b9189-467">Service Fabric</span></span>

<span data-ttu-id="b9189-468">Distribuir serviços confiáveis em um cluster do Service Fabric protege contra a maioria das falhas transitórias potenciais discutidas neste artigo.</span><span class="sxs-lookup"><span data-stu-id="b9189-468">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="b9189-469">No entanto, algumas falhas transitórias ainda podem acontecer.</span><span class="sxs-lookup"><span data-stu-id="b9189-469">Some transient faults are still possible, however.</span></span> <span data-ttu-id="b9189-470">Por exemplo, se o serviço de nomeação recebe uma solicitação no meio de uma alteração de roteamento, ele pode lançar uma exceção.</span><span class="sxs-lookup"><span data-stu-id="b9189-470">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="b9189-471">No entanto, se a mesma solicitação vier 100 milissegundos mais tarde, ela provavelmente terá êxito.</span><span class="sxs-lookup"><span data-stu-id="b9189-471">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="b9189-472">Internamente, o Service Fabric gerencia esse tipo de falha transitória.</span><span class="sxs-lookup"><span data-stu-id="b9189-472">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="b9189-473">Você pode definir algumas configurações usando a classe `OperationRetrySettings` ao configurar seus serviços.</span><span class="sxs-lookup"><span data-stu-id="b9189-473">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="b9189-474">O código a seguir mostra um exemplo.</span><span class="sxs-lookup"><span data-stu-id="b9189-474">The following code shows an example.</span></span> <span data-ttu-id="b9189-475">Na maioria dos casos, isso não deve ser necessário e as configurações padrão são suficientes.</span><span class="sxs-lookup"><span data-stu-id="b9189-475">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

```csharp
FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
{
    OperationTimeout = TimeSpan.FromSeconds(30)
};

var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
    new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
    new ServicePartitionKey(0));
```

### <a name="more-information"></a><span data-ttu-id="b9189-476">Mais informações</span><span class="sxs-lookup"><span data-stu-id="b9189-476">More information</span></span>

* [<span data-ttu-id="b9189-477">Manipulação de exceção remota</span><span class="sxs-lookup"><span data-stu-id="b9189-477">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a><span data-ttu-id="b9189-478">Banco de Dados SQL usando o ADO.NET</span><span class="sxs-lookup"><span data-stu-id="b9189-478">SQL Database using ADO.NET</span></span>
<span data-ttu-id="b9189-479">O Banco de Dados SQL é um banco de dados SQL disponível em vários tamanhos e como um serviço padrão (compartilhado) e premium (não compartilhado).</span><span class="sxs-lookup"><span data-stu-id="b9189-479">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="b9189-480">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-480">Retry mechanism</span></span>
<span data-ttu-id="b9189-481">O Banco de Dados SQL não tem suporte interno para repetições quando acessado usando o ADO.NET.</span><span class="sxs-lookup"><span data-stu-id="b9189-481">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="b9189-482">No entanto, os códigos de retorno das solicitações podem ser usados para determinar por que uma solicitação falhou.</span><span class="sxs-lookup"><span data-stu-id="b9189-482">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="b9189-483">Para obter mais informações sobre a limitação do banco de dados SQL, confira [Limites de recursos do banco de dados SQL do Azure](/azure/sql-database/sql-database-resource-limits).</span><span class="sxs-lookup"><span data-stu-id="b9189-483">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="b9189-484">Para obter uma lista dos códigos de erro relevantes, confira [códigos de erro SQL para aplicativos de cliente do banco de dados SQL](/azure/sql-database/sql-database-develop-error-messages).</span><span class="sxs-lookup"><span data-stu-id="b9189-484">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="b9189-485">Você pode usar a biblioteca Polly para implementar tentativas para o banco de dados SQL.</span><span class="sxs-lookup"><span data-stu-id="b9189-485">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="b9189-486">Confira [Falha transitória manipulando com a Polly](#transient-fault-handling-with-polly).</span><span class="sxs-lookup"><span data-stu-id="b9189-486">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="b9189-487">Diretriz de uso de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-487">Retry usage guidance</span></span>
<span data-ttu-id="b9189-488">Considere as seguintes diretrizes ao acessar o Banco de Dados SQL usando o ADO.NET:</span><span class="sxs-lookup"><span data-stu-id="b9189-488">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="b9189-489">Escolha a opção de serviço apropriada (compartilhada ou premium).</span><span class="sxs-lookup"><span data-stu-id="b9189-489">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="b9189-490">Uma instância compartilhada pode sofrer atrasos e limitação de conexão mais longos que o normal devido ao uso por outros locatários do servidor compartilhado.</span><span class="sxs-lookup"><span data-stu-id="b9189-490">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="b9189-491">Se forem necessárias mais operações confiáveis de baixa latência e desempenho previsível, considere a escolha da opção premium.</span><span class="sxs-lookup"><span data-stu-id="b9189-491">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="b9189-492">Garanta a execução de repetições no nível ou escopo apropriado para evitar operações não idempotentes que causem inconsistência nos dados.</span><span class="sxs-lookup"><span data-stu-id="b9189-492">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="b9189-493">De modo ideal, todas as operações devem ser idempotentes para que possam ser repetidas sem causar inconsistência.</span><span class="sxs-lookup"><span data-stu-id="b9189-493">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="b9189-494">Quando não for esse o caso, a repetição deverá ser realizada em um nível ou escopo que permita que todas as alterações relacionadas sejam desfeitas, caso uma operação falhe; por exemplo, em um escopo transacional.</span><span class="sxs-lookup"><span data-stu-id="b9189-494">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="b9189-495">Para saber mais, consulte [Camada de acesso a dados dos conceitos básicos do serviço de nuvem — tratamento de falhas transitórias](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span><span class="sxs-lookup"><span data-stu-id="b9189-495">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="b9189-496">Uma estratégia de intervalo fixo não é recomendada para uso com o Banco de Dados SQL do Azure, exceto em cenários interativos em que há apenas algumas repetições em intervalos muito curtos.</span><span class="sxs-lookup"><span data-stu-id="b9189-496">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="b9189-497">Em vez disso, considere usar uma estratégia de retirada exponencial para a maioria dos cenários.</span><span class="sxs-lookup"><span data-stu-id="b9189-497">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="b9189-498">Escolha um valor adequado para os tempos limite de conexão e comando ao definir conexões.</span><span class="sxs-lookup"><span data-stu-id="b9189-498">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="b9189-499">Um tempo limite muito curto pode resultar em falhas prematuras de conexões quando o banco de dados estiver ocupado.</span><span class="sxs-lookup"><span data-stu-id="b9189-499">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="b9189-500">Um tempo limite muito longo pode impedir que a lógica de repetição funcione corretamente, aguardando tempo demais para detectar uma conexão com falha.</span><span class="sxs-lookup"><span data-stu-id="b9189-500">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="b9189-501">O valor do tempo limite é um componente da latência de ponta a ponta; ele é efetivamente adicionado ao intervalo de repetição especificado na política de repetição para cada tentativa de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-501">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="b9189-502">Feche a conexão após um determinado número de repetições, mesmo ao usar uma lógica de repetição de retirada exponencial, e repita a operação em uma nova conexão.</span><span class="sxs-lookup"><span data-stu-id="b9189-502">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="b9189-503">Repetir a mesma operação várias vezes na mesma conexão pode ser um fator que contribui para problemas de conexão.</span><span class="sxs-lookup"><span data-stu-id="b9189-503">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="b9189-504">Para obter um exemplo dessa técnica, consulte [Camada de acesso a dados dos conceitos básicos do serviço de nuvem — tratamento de falhas transitórias](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span><span class="sxs-lookup"><span data-stu-id="b9189-504">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="b9189-505">Quando o pool de conexões está em uso (o padrão), é provável que a mesma conexão seja escolhida no pool, mesmo depois de fechar e reabrir uma conexão.</span><span class="sxs-lookup"><span data-stu-id="b9189-505">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="b9189-506">Se for esse o caso, uma técnica para resolver isso é chamar o método **ClearPool** da classe **SqlConnection** para marcar a conexão como não reutilizável.</span><span class="sxs-lookup"><span data-stu-id="b9189-506">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="b9189-507">No entanto, isso deve ser feito somente depois que várias tentativas de conexão tiverem falhado, e somente ao encontrar a classe específica de falhas transitórias, como tempos limite de SQL (código de erro -2), relacionada às conexões com falha.</span><span class="sxs-lookup"><span data-stu-id="b9189-507">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="b9189-508">Se o código de acesso a dados usar transações iniciadas como instâncias **TransactionScope** , a lógica de repetição deverá reabrir a conexão e iniciar um novo escopo de transação.</span><span class="sxs-lookup"><span data-stu-id="b9189-508">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="b9189-509">Por esse motivo, o bloco de código com nova tentativa deve englobar o escopo inteiro da transação.</span><span class="sxs-lookup"><span data-stu-id="b9189-509">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="b9189-510">Considere começar com as seguintes configurações para operações de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-510">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="b9189-511">Essas são configurações de uso geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.</span><span class="sxs-lookup"><span data-stu-id="b9189-511">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="b9189-512">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="b9189-512">**Context**</span></span> | <span data-ttu-id="b9189-513">**Exemplo de latênciaE2E<br />máxima de destino**</span><span class="sxs-lookup"><span data-stu-id="b9189-513">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="b9189-514">**Estratégia de repetição**</span><span class="sxs-lookup"><span data-stu-id="b9189-514">**Retry strategy**</span></span> | <span data-ttu-id="b9189-515">**Configurações**</span><span class="sxs-lookup"><span data-stu-id="b9189-515">**Settings**</span></span> | <span data-ttu-id="b9189-516">**Valores**</span><span class="sxs-lookup"><span data-stu-id="b9189-516">**Values**</span></span> | <span data-ttu-id="b9189-517">**Como funciona**</span><span class="sxs-lookup"><span data-stu-id="b9189-517">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="b9189-518">Interativo, interface de usuário</span><span class="sxs-lookup"><span data-stu-id="b9189-518">Interactive, UI,</span></span><br /><span data-ttu-id="b9189-519">ou primeiro plano</span><span class="sxs-lookup"><span data-stu-id="b9189-519">or foreground</span></span> |<span data-ttu-id="b9189-520">2 s</span><span class="sxs-lookup"><span data-stu-id="b9189-520">2 sec</span></span> |<span data-ttu-id="b9189-521">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="b9189-521">FixedInterval</span></span> |<span data-ttu-id="b9189-522">Contagem de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-522">Retry count</span></span><br /><span data-ttu-id="b9189-523">Intervalo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-523">Retry interval</span></span><br /><span data-ttu-id="b9189-524">Primeira repetição rápida</span><span class="sxs-lookup"><span data-stu-id="b9189-524">First fast retry</span></span> |<span data-ttu-id="b9189-525">3</span><span class="sxs-lookup"><span data-stu-id="b9189-525">3</span></span><br /><span data-ttu-id="b9189-526">500 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-526">500 ms</span></span><br /><span data-ttu-id="b9189-527">verdadeiro</span><span class="sxs-lookup"><span data-stu-id="b9189-527">true</span></span> |<span data-ttu-id="b9189-528">1ª tentativa — intervalo de 0 s</span><span class="sxs-lookup"><span data-stu-id="b9189-528">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="b9189-529">2ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-529">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="b9189-530">3ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-530">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="b9189-531">Segundo plano</span><span class="sxs-lookup"><span data-stu-id="b9189-531">Background</span></span><br /><span data-ttu-id="b9189-532">ou lote</span><span class="sxs-lookup"><span data-stu-id="b9189-532">or batch</span></span> |<span data-ttu-id="b9189-533">30 s</span><span class="sxs-lookup"><span data-stu-id="b9189-533">30 sec</span></span> |<span data-ttu-id="b9189-534">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="b9189-534">ExponentialBackoff</span></span> |<span data-ttu-id="b9189-535">Contagem de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-535">Retry count</span></span><br /><span data-ttu-id="b9189-536">Retirada mín.</span><span class="sxs-lookup"><span data-stu-id="b9189-536">Min back-off</span></span><br /><span data-ttu-id="b9189-537">Retirada máx.</span><span class="sxs-lookup"><span data-stu-id="b9189-537">Max back-off</span></span><br /><span data-ttu-id="b9189-538">Retirada delta</span><span class="sxs-lookup"><span data-stu-id="b9189-538">Delta back-off</span></span><br /><span data-ttu-id="b9189-539">Primeira repetição rápida</span><span class="sxs-lookup"><span data-stu-id="b9189-539">First fast retry</span></span> |<span data-ttu-id="b9189-540">5</span><span class="sxs-lookup"><span data-stu-id="b9189-540">5</span></span><br /><span data-ttu-id="b9189-541">0 s</span><span class="sxs-lookup"><span data-stu-id="b9189-541">0 sec</span></span><br /><span data-ttu-id="b9189-542">60 s</span><span class="sxs-lookup"><span data-stu-id="b9189-542">60 sec</span></span><br /><span data-ttu-id="b9189-543">2 s</span><span class="sxs-lookup"><span data-stu-id="b9189-543">2 sec</span></span><br /><span data-ttu-id="b9189-544">falso</span><span class="sxs-lookup"><span data-stu-id="b9189-544">false</span></span> |<span data-ttu-id="b9189-545">1ª tentativa — intervalo de 0 s</span><span class="sxs-lookup"><span data-stu-id="b9189-545">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="b9189-546">2ª tentativa — intervalo de ~2 s</span><span class="sxs-lookup"><span data-stu-id="b9189-546">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="b9189-547">3ª tentativa — intervalo de ~6 s</span><span class="sxs-lookup"><span data-stu-id="b9189-547">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="b9189-548">4ª tentativa — intervalo de ~14 s</span><span class="sxs-lookup"><span data-stu-id="b9189-548">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="b9189-549">5ª tentativa — intervalo de ~30 s</span><span class="sxs-lookup"><span data-stu-id="b9189-549">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="b9189-550">A latência de ponta a ponta visa o tempo limite padrão de conexões com o serviço.</span><span class="sxs-lookup"><span data-stu-id="b9189-550">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="b9189-551">Se você especificar tempos limite de conexão mais longos, a latência de ponta a ponta será estendida por esse tempo adicional para cada tentativa de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-551">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="b9189-552">Exemplos</span><span class="sxs-lookup"><span data-stu-id="b9189-552">Examples</span></span>
<span data-ttu-id="b9189-553">Esta seção mostra como você pode usar a Polly para acessar o banco de dados SQL do Azure usando um conjunto de políticas de repetição configurado na classe `Policy`.</span><span class="sxs-lookup"><span data-stu-id="b9189-553">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="b9189-554">O código a seguir mostra um método de extensão na classe `SqlCommand` que chama `ExecuteAsync` com retirada exponencial.</span><span class="sxs-lookup"><span data-stu-id="b9189-554">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) => 
        {
            // Capture some info for logging/telemetry.  
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy 

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}
```

<span data-ttu-id="b9189-555">Esse método de extensão assíncrono pode ser usado da seguinte maneira.</span><span class="sxs-lookup"><span data-stu-id="b9189-555">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="b9189-556">Mais informações</span><span class="sxs-lookup"><span data-stu-id="b9189-556">More information</span></span>
* [<span data-ttu-id="b9189-557">Camada de acesso a dados dos conceitos básicos do serviço de nuvem — tratamento de falhas transitórias</span><span class="sxs-lookup"><span data-stu-id="b9189-557">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="b9189-558">Para orientação geral sobre como obter o máximo do banco de dados SQL, confira [Desempenho de banco de dados SQL do Azure e guia de elasticidade](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span><span class="sxs-lookup"><span data-stu-id="b9189-558">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>

## <a name="sql-database-using-entity-framework-6"></a><span data-ttu-id="b9189-559">Banco de Dados SQL usando o Entity Framework 6</span><span class="sxs-lookup"><span data-stu-id="b9189-559">SQL Database using Entity Framework 6</span></span>
<span data-ttu-id="b9189-560">O Banco de Dados SQL é um banco de dados SQL disponível em vários tamanhos e como um serviço padrão (compartilhado) e premium (não compartilhado).</span><span class="sxs-lookup"><span data-stu-id="b9189-560">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="b9189-561">O Entity Framework é um mapeador relacional de objeto que permite aos desenvolvedores do .NET trabalhar com dados relacionais usando objetos específicos de domínio.</span><span class="sxs-lookup"><span data-stu-id="b9189-561">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="b9189-562">Com ele, não há a necessidade da maioria dos códigos de acesso a dados que os desenvolvedores geralmente precisam para escrever.</span><span class="sxs-lookup"><span data-stu-id="b9189-562">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="b9189-563">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-563">Retry mechanism</span></span>
<span data-ttu-id="b9189-564">O suporte à repetição é fornecido durante o acesso ao Banco de Dados SQL usando o Entity Framework 6.0 e versões posteriores por meio de um mecanismo chamado [Lógica de Repetição/Resiliência de Conexão](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span><span class="sxs-lookup"><span data-stu-id="b9189-564">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span></span> <span data-ttu-id="b9189-565">Os principais recursos do mecanismo de repetição são:</span><span class="sxs-lookup"><span data-stu-id="b9189-565">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="b9189-566">A abstração primária é a interface **IDbExecutionStrategy** .</span><span class="sxs-lookup"><span data-stu-id="b9189-566">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="b9189-567">Essa interface:</span><span class="sxs-lookup"><span data-stu-id="b9189-567">This interface:</span></span>
  * <span data-ttu-id="b9189-568">Define métodos **Execute**\* síncronos e assíncronos.</span><span class="sxs-lookup"><span data-stu-id="b9189-568">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  * <span data-ttu-id="b9189-569">Define classes que podem ser usadas diretamente ou podem ser configuradas em um contexto de banco de dados como uma estratégia padrão, mapeadas para nome do provedor ou para um nome de provedor e nome de servidor.</span><span class="sxs-lookup"><span data-stu-id="b9189-569">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="b9189-570">Quando configuradas em um contexto, as repetições ocorrem no nível de operações de banco de dados individuais, das quais pode haver várias de uma determinada operação de contexto.</span><span class="sxs-lookup"><span data-stu-id="b9189-570">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="b9189-571">Define quando repetir uma conexão com falha, e como.</span><span class="sxs-lookup"><span data-stu-id="b9189-571">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="b9189-572">Incluem várias implementações internas da interface **IDbExecutionStrategy** :</span><span class="sxs-lookup"><span data-stu-id="b9189-572">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="b9189-573">Padrão: sem repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-573">Default - no retrying.</span></span>
  * <span data-ttu-id="b9189-574">Padrão para Banco de Dados SQL(automático): sem repetição, mas inspeciona exceções e as encapsula com sugestão para usar a estratégia Banco de Dados SQL.</span><span class="sxs-lookup"><span data-stu-id="b9189-574">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="b9189-575">Padrão para Banco de Dados SQL: exponencial (herdado da classe base) mais lógica de detecção do Banco de Dados SQL.</span><span class="sxs-lookup"><span data-stu-id="b9189-575">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="b9189-576">Implementam uma estratégia de retirada exponencial que inclui aleatoriedade.</span><span class="sxs-lookup"><span data-stu-id="b9189-576">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="b9189-577">As características das classes de repetição internas são: com estado e sem thread-safe.</span><span class="sxs-lookup"><span data-stu-id="b9189-577">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="b9189-578">No entanto, elas podem ser reutilizadas depois que a operação atual for concluída.</span><span class="sxs-lookup"><span data-stu-id="b9189-578">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="b9189-579">Se a contagem de repetição especificada for excedida, os resultados serão encapsulados em uma nova exceção.</span><span class="sxs-lookup"><span data-stu-id="b9189-579">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="b9189-580">A exceção atual não é exibida.</span><span class="sxs-lookup"><span data-stu-id="b9189-580">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="b9189-581">Configuração de política</span><span class="sxs-lookup"><span data-stu-id="b9189-581">Policy configuration</span></span>
<span data-ttu-id="b9189-582">O suporte à repetição é fornecido durante o acesso ao Banco de Dados SQL usando o Entity Framework 6.0 e versões superiores.</span><span class="sxs-lookup"><span data-stu-id="b9189-582">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="b9189-583">As políticas de repetição são configuradas de modo programático.</span><span class="sxs-lookup"><span data-stu-id="b9189-583">Retry policies are configured programmatically.</span></span> <span data-ttu-id="b9189-584">A configuração não pode ser alterada por operação.</span><span class="sxs-lookup"><span data-stu-id="b9189-584">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="b9189-585">Ao configurar uma estratégia no contexto como o padrão, você especifica uma função que cria uma nova estratégia sob demanda.</span><span class="sxs-lookup"><span data-stu-id="b9189-585">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="b9189-586">O código a seguir mostra como você pode criar uma classe de configuração de repetição que estende a classe base **DbConfiguration** .</span><span class="sxs-lookup"><span data-stu-id="b9189-586">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

<span data-ttu-id="b9189-587">Depois isso pode ser especificado como a estratégia de repetição padrão para todas as operações usando o método **SetConfiguration** da instância **DbConfiguration** quando o aplicativo é iniciado.</span><span class="sxs-lookup"><span data-stu-id="b9189-587">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="b9189-588">Por padrão, o EF vai descobrir e usar automaticamente a classe de configuração.</span><span class="sxs-lookup"><span data-stu-id="b9189-588">By default, EF will automatically discover and use the configuration class.</span></span>

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

<span data-ttu-id="b9189-589">Você pode especificar a classe de configuração de repetição para um contexto anotando a classe de contexto com um atributo **DbConfigurationType** .</span><span class="sxs-lookup"><span data-stu-id="b9189-589">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="b9189-590">No entanto, se você tiver apenas uma classe de configuração, o EF a usará sem a necessidade de anotar o contexto.</span><span class="sxs-lookup"><span data-stu-id="b9189-590">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

<span data-ttu-id="b9189-591">Se precisar usar estratégias de repetição diferentes para operações específicas ou desabilitar repetições para operações específicas, você poderá criar uma classe de configuração que permita suspender ou trocar estratégias, definindo um sinalizador no **CallContext**.</span><span class="sxs-lookup"><span data-stu-id="b9189-591">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="b9189-592">A classe de configuração pode usar esse sinalizador para alternar estratégias ou desabilitar a estratégia fornecida por você e usar uma estratégia padrão.</span><span class="sxs-lookup"><span data-stu-id="b9189-592">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="b9189-593">Para saber mais, confira [Suspender a estratégia de execução](/ef/ef6/fundamentals/connection-resiliency/retry-logic#workaround-suspend-execution-strategy) (EF6 em diante).</span><span class="sxs-lookup"><span data-stu-id="b9189-593">For more information, see [Suspend Execution Strategy](/ef/ef6/fundamentals/connection-resiliency/retry-logic#workaround-suspend-execution-strategy) (EF6 onwards).</span></span>

<span data-ttu-id="b9189-594">Outra técnica para usar estratégias de repetição específicas para operações individuais é criar uma instância da classe de estratégia necessária e fornecer as configurações desejadas por meio de parâmetros.</span><span class="sxs-lookup"><span data-stu-id="b9189-594">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="b9189-595">Você então chama o método **ExecuteAsync** .</span><span class="sxs-lookup"><span data-stu-id="b9189-595">You then invoke its **ExecuteAsync** method.</span></span>

```csharp
var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
var blogs = await executionStrategy.ExecuteAsync(
    async () =>
    {
        using (var db = new BloggingContext("Blogs"))
        {
            // Acquire some values asynchronously and return them
        }
    },
    new CancellationToken()
);
```

<span data-ttu-id="b9189-596">A maneira mais simples de usar uma classe **DbConfiguration** é colocá-la no mesmo assembly da classe **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="b9189-596">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="b9189-597">No entanto, isso não é apropriado quando o mesmo contexto é necessário em cenários diferentes, como diferentes estratégias de repetição interativas e em segundo plano.</span><span class="sxs-lookup"><span data-stu-id="b9189-597">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="b9189-598">Se os diferentes contextos forem executados em AppDomains separados, você poderá usar o suporte interno para especificar classes de configuração no arquivo de configuração ou defini-las explicitamente usando código.</span><span class="sxs-lookup"><span data-stu-id="b9189-598">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="b9189-599">Se os diferentes contextos devem ser executados no mesmo AppDomain, será necessária uma solução personalizada.</span><span class="sxs-lookup"><span data-stu-id="b9189-599">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="b9189-600">Para saber mais, confira [Configuração baseada em código](/ef/ef6/fundamentals/configuring/code-based) (EF6 em diante).</span><span class="sxs-lookup"><span data-stu-id="b9189-600">For more information, see [Code-Based Configuration](/ef/ef6/fundamentals/configuring/code-based) (EF6 onwards).</span></span>

<span data-ttu-id="b9189-601">A tabela a seguir mostra as configurações padrão para a política de repetição interna ao usar o EF6.</span><span class="sxs-lookup"><span data-stu-id="b9189-601">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="b9189-602">Configuração</span><span class="sxs-lookup"><span data-stu-id="b9189-602">Setting</span></span> | <span data-ttu-id="b9189-603">Valor padrão</span><span class="sxs-lookup"><span data-stu-id="b9189-603">Default value</span></span> | <span data-ttu-id="b9189-604">Significado</span><span class="sxs-lookup"><span data-stu-id="b9189-604">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="b9189-605">Política</span><span class="sxs-lookup"><span data-stu-id="b9189-605">Policy</span></span> | <span data-ttu-id="b9189-606">Exponencial</span><span class="sxs-lookup"><span data-stu-id="b9189-606">Exponential</span></span> | <span data-ttu-id="b9189-607">Retirada exponencial.</span><span class="sxs-lookup"><span data-stu-id="b9189-607">Exponential back-off.</span></span> |
| <span data-ttu-id="b9189-608">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="b9189-608">MaxRetryCount</span></span> | <span data-ttu-id="b9189-609">5</span><span class="sxs-lookup"><span data-stu-id="b9189-609">5</span></span> | <span data-ttu-id="b9189-610">O número máximo de repetições.</span><span class="sxs-lookup"><span data-stu-id="b9189-610">The maximum number of retries.</span></span> |
| <span data-ttu-id="b9189-611">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="b9189-611">MaxDelay</span></span> | <span data-ttu-id="b9189-612">30 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-612">30 seconds</span></span> | <span data-ttu-id="b9189-613">O atraso máximo entre as repetições.</span><span class="sxs-lookup"><span data-stu-id="b9189-613">The maximum delay between retries.</span></span> <span data-ttu-id="b9189-614">Esse valor não afeta como a série de atrasos é computada.</span><span class="sxs-lookup"><span data-stu-id="b9189-614">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="b9189-615">Ele apenas define um limite superior.</span><span class="sxs-lookup"><span data-stu-id="b9189-615">It only defines an upper bound.</span></span> |
| <span data-ttu-id="b9189-616">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="b9189-616">DefaultCoefficient</span></span> | <span data-ttu-id="b9189-617">1 segundo</span><span class="sxs-lookup"><span data-stu-id="b9189-617">1 second</span></span> | <span data-ttu-id="b9189-618">O coeficiente para o cálculo de retirada exponencial.</span><span class="sxs-lookup"><span data-stu-id="b9189-618">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="b9189-619">Esse valor não pode ser alterado.</span><span class="sxs-lookup"><span data-stu-id="b9189-619">This value cannot be changed.</span></span> |
| <span data-ttu-id="b9189-620">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="b9189-620">DefaultRandomFactor</span></span> | <span data-ttu-id="b9189-621">1,1</span><span class="sxs-lookup"><span data-stu-id="b9189-621">1.1</span></span> | <span data-ttu-id="b9189-622">O multiplicador usado para adicionar um atraso aleatório a cada entrada.</span><span class="sxs-lookup"><span data-stu-id="b9189-622">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="b9189-623">Esse valor não pode ser alterado.</span><span class="sxs-lookup"><span data-stu-id="b9189-623">This value cannot be changed.</span></span> |
| <span data-ttu-id="b9189-624">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="b9189-624">DefaultExponentialBase</span></span> | <span data-ttu-id="b9189-625">2</span><span class="sxs-lookup"><span data-stu-id="b9189-625">2</span></span> | <span data-ttu-id="b9189-626">O multiplicador usado para calcular o próximo atraso.</span><span class="sxs-lookup"><span data-stu-id="b9189-626">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="b9189-627">Esse valor não pode ser alterado.</span><span class="sxs-lookup"><span data-stu-id="b9189-627">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="b9189-628">Diretriz de uso de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-628">Retry usage guidance</span></span>
<span data-ttu-id="b9189-629">Considere as seguintes diretrizes ao acessar o Banco de Dados SQL usando o EF6:</span><span class="sxs-lookup"><span data-stu-id="b9189-629">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="b9189-630">Escolha a opção de serviço apropriada (compartilhada ou premium).</span><span class="sxs-lookup"><span data-stu-id="b9189-630">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="b9189-631">Uma instância compartilhada pode sofrer atrasos e limitação de conexão mais longos que o normal devido ao uso por outros locatários do servidor compartilhado.</span><span class="sxs-lookup"><span data-stu-id="b9189-631">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="b9189-632">Se forem necessárias operações confiáveis de baixa latência e desempenho previsível, considere a escolha da opção premium.</span><span class="sxs-lookup"><span data-stu-id="b9189-632">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="b9189-633">Uma estratégia de intervalo fixo não é recomendada para ser usada com o Banco de Dados SQL do Azure.</span><span class="sxs-lookup"><span data-stu-id="b9189-633">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="b9189-634">Em vez disso, use uma estratégia de retirada exponencial, pois o serviço pode estar sobrecarregado e intervalos mais longos permitem mais tempo para a recuperação.</span><span class="sxs-lookup"><span data-stu-id="b9189-634">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="b9189-635">Escolha um valor adequado para os tempos limite de conexão e comando ao definir conexões.</span><span class="sxs-lookup"><span data-stu-id="b9189-635">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="b9189-636">Baseie o tempo limite no seu design de lógica de negócios e por meio de teste.</span><span class="sxs-lookup"><span data-stu-id="b9189-636">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="b9189-637">Talvez seja necessário modificar esse valor ao longo do tempo, uma vez que os volumes de dados ou os processos de negócios mudam.</span><span class="sxs-lookup"><span data-stu-id="b9189-637">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="b9189-638">Um tempo limite muito curto pode resultar em falhas prematuras de conexões quando o banco de dados estiver ocupado.</span><span class="sxs-lookup"><span data-stu-id="b9189-638">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="b9189-639">Um tempo limite muito longo pode impedir que a lógica de repetição funcione corretamente, aguardando tempo demais para detectar uma conexão com falha.</span><span class="sxs-lookup"><span data-stu-id="b9189-639">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="b9189-640">O valor do tempo limite é um componente da latência de ponta a ponta, embora você não possa determinar facilmente quantos comandos serão executados ao salvar o contexto.</span><span class="sxs-lookup"><span data-stu-id="b9189-640">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="b9189-641">Você pode alterar o tempo limite padrão definindo a propriedade **CommandTimeout** da instância **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="b9189-641">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="b9189-642">O Entity Framework oferece suporte às configurações de repetição definidas em arquivos de configuração.</span><span class="sxs-lookup"><span data-stu-id="b9189-642">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="b9189-643">No entanto, para máxima flexibilidade no Azure, você deve considerar a criação da configuração de modo programático dentro do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b9189-643">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="b9189-644">Os parâmetros específicos das políticas de repetição, como o número de repetições e os intervalos da repetição, podem ser armazenados no arquivo de configuração de serviço e usados no tempo de execução para criar as políticas apropriadas.</span><span class="sxs-lookup"><span data-stu-id="b9189-644">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="b9189-645">Isso permite que as configurações sejam alteradas sem precisar reiniciar o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b9189-645">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="b9189-646">Considere começar com as seguintes configurações para operações de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-646">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="b9189-647">Você não pode especificar o intervalo entre tentativas de repetição (ele é fixo e gerado como uma sequência exponencial).</span><span class="sxs-lookup"><span data-stu-id="b9189-647">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="b9189-648">Você pode especificar apenas os valores máximo, conforme mostrado aqui, a menos que você crie uma estratégia de repetição personalizada.</span><span class="sxs-lookup"><span data-stu-id="b9189-648">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="b9189-649">Essas são configurações de finalidade geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.</span><span class="sxs-lookup"><span data-stu-id="b9189-649">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="b9189-650">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="b9189-650">**Context**</span></span> | <span data-ttu-id="b9189-651">**Exemplo de latênciaE2E<br />máxima de destino**</span><span class="sxs-lookup"><span data-stu-id="b9189-651">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="b9189-652">**Política de repetição**</span><span class="sxs-lookup"><span data-stu-id="b9189-652">**Retry policy**</span></span> | <span data-ttu-id="b9189-653">**Configurações**</span><span class="sxs-lookup"><span data-stu-id="b9189-653">**Settings**</span></span> | <span data-ttu-id="b9189-654">**Valores**</span><span class="sxs-lookup"><span data-stu-id="b9189-654">**Values**</span></span> | <span data-ttu-id="b9189-655">**Como funciona**</span><span class="sxs-lookup"><span data-stu-id="b9189-655">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="b9189-656">Interativo, interface de usuário</span><span class="sxs-lookup"><span data-stu-id="b9189-656">Interactive, UI,</span></span><br /><span data-ttu-id="b9189-657">ou primeiro plano</span><span class="sxs-lookup"><span data-stu-id="b9189-657">or foreground</span></span> |<span data-ttu-id="b9189-658">2 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-658">2 seconds</span></span> |<span data-ttu-id="b9189-659">Exponencial</span><span class="sxs-lookup"><span data-stu-id="b9189-659">Exponential</span></span> |<span data-ttu-id="b9189-660">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="b9189-660">MaxRetryCount</span></span><br /><span data-ttu-id="b9189-661">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="b9189-661">MaxDelay</span></span> |<span data-ttu-id="b9189-662">3</span><span class="sxs-lookup"><span data-stu-id="b9189-662">3</span></span><br /><span data-ttu-id="b9189-663">750 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-663">750 ms</span></span> |<span data-ttu-id="b9189-664">1ª tentativa — intervalo de 0 s</span><span class="sxs-lookup"><span data-stu-id="b9189-664">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="b9189-665">2ª tentativa — intervalo de 750 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-665">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="b9189-666">3ª tentativa — intervalo de 750 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-666">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="b9189-667">Segundo plano</span><span class="sxs-lookup"><span data-stu-id="b9189-667">Background</span></span><br /> <span data-ttu-id="b9189-668">ou lote</span><span class="sxs-lookup"><span data-stu-id="b9189-668">or batch</span></span> |<span data-ttu-id="b9189-669">30 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-669">30 seconds</span></span> |<span data-ttu-id="b9189-670">Exponencial</span><span class="sxs-lookup"><span data-stu-id="b9189-670">Exponential</span></span> |<span data-ttu-id="b9189-671">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="b9189-671">MaxRetryCount</span></span><br /><span data-ttu-id="b9189-672">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="b9189-672">MaxDelay</span></span> |<span data-ttu-id="b9189-673">5</span><span class="sxs-lookup"><span data-stu-id="b9189-673">5</span></span><br /><span data-ttu-id="b9189-674">12 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-674">12 seconds</span></span> |<span data-ttu-id="b9189-675">1ª tentativa — intervalo de 0 s</span><span class="sxs-lookup"><span data-stu-id="b9189-675">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="b9189-676">2ª tentativa — intervalo de ~1 s</span><span class="sxs-lookup"><span data-stu-id="b9189-676">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="b9189-677">3ª tentativa — intervalo de ~3 s</span><span class="sxs-lookup"><span data-stu-id="b9189-677">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="b9189-678">4ª tentativa — intervalo de ~7 s</span><span class="sxs-lookup"><span data-stu-id="b9189-678">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="b9189-679">5ª tentativa — intervalo de 12 s</span><span class="sxs-lookup"><span data-stu-id="b9189-679">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="b9189-680">A latência de ponta a ponta visa o tempo limite padrão de conexões com o serviço.</span><span class="sxs-lookup"><span data-stu-id="b9189-680">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="b9189-681">Se você especificar tempos limite de conexão mais longos, a latência de ponta a ponta será estendida por esse tempo adicional para cada tentativa de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-681">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="b9189-682">Exemplos</span><span class="sxs-lookup"><span data-stu-id="b9189-682">Examples</span></span>
<span data-ttu-id="b9189-683">O exemplo de código a seguir define uma solução de acesso a dados simples que usa o Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="b9189-683">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="b9189-684">Ele estabelece uma estratégia de repetição específica definindo uma instância de uma classe chamada **BlogConfiguration** que estende **DbConfiguration**.</span><span class="sxs-lookup"><span data-stu-id="b9189-684">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

<span data-ttu-id="b9189-685">Mais exemplos de como usar o mecanismo de repetição do Entity Framework podem ser encontrados em [Lógica de Repetição/Resiliência de Conexão](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span><span class="sxs-lookup"><span data-stu-id="b9189-685">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span></span>

### <a name="more-information"></a><span data-ttu-id="b9189-686">Mais informações</span><span class="sxs-lookup"><span data-stu-id="b9189-686">More information</span></span>
* [<span data-ttu-id="b9189-687">Guia de desempenho e elasticidade do Banco de Dados SQL do Azure</span><span class="sxs-lookup"><span data-stu-id="b9189-687">Azure SQL Database Performance and Elasticity Guide</span></span>](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a><span data-ttu-id="b9189-688">Banco de Dados SQL usando o Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="b9189-688">SQL Database using Entity Framework Core</span></span>
<span data-ttu-id="b9189-689">[O Entity Framework Core](/ef/core/) é um mapeador relacional de objeto que permite aos desenvolvedores do .NET Core trabalhar com dados usando objetos específicos de domínio.</span><span class="sxs-lookup"><span data-stu-id="b9189-689">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="b9189-690">Com ele, não há a necessidade da maioria dos códigos de acesso a dados que os desenvolvedores geralmente precisam para escrever.</span><span class="sxs-lookup"><span data-stu-id="b9189-690">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="b9189-691">Esta versão do Entity Framework foi criada a partir do zero e não herda todos os recursos do EF6.x automaticamente.</span><span class="sxs-lookup"><span data-stu-id="b9189-691">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="b9189-692">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-692">Retry mechanism</span></span>
<span data-ttu-id="b9189-693">O suporte à repetição é fornecido durante o acesso ao Banco de Dados SQL usando o Entity Framework Core por meio de um mecanismo chamado [Resiliência de Conexão](/ef/core/miscellaneous/connection-resiliency).</span><span class="sxs-lookup"><span data-stu-id="b9189-693">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="b9189-694">A resiliência de conexão foi introduzida no EF Core 1.1.0.</span><span class="sxs-lookup"><span data-stu-id="b9189-694">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="b9189-695">A abstração primária é a interface `IExecutionStrategy`.</span><span class="sxs-lookup"><span data-stu-id="b9189-695">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="b9189-696">A estratégia de execução para o SQL Server, incluindo o SQL Azure, está ciente dos tipos de exceção que podem ser repetidos e tem padrões específicos para o número máximo de tentativas, atrasos entre as repetições e assim por diante.</span><span class="sxs-lookup"><span data-stu-id="b9189-696">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="b9189-697">Exemplos</span><span class="sxs-lookup"><span data-stu-id="b9189-697">Examples</span></span>

<span data-ttu-id="b9189-698">O código a seguir permite novas tentativas automáticas ao configurar o objeto DbContext, que representa uma sessão com o banco de dados.</span><span class="sxs-lookup"><span data-stu-id="b9189-698">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="b9189-699">O código a seguir mostra como executar uma transação com novas tentativas automáticas, usando uma estratégia de execução.</span><span class="sxs-lookup"><span data-stu-id="b9189-699">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="b9189-700">A transação é definida em um representante.</span><span class="sxs-lookup"><span data-stu-id="b9189-700">The transaction is defined in a delegate.</span></span> <span data-ttu-id="b9189-701">Se ocorrer uma falha transitória, a estratégia de execução invocará o representante novamente.</span><span class="sxs-lookup"><span data-stu-id="b9189-701">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "https://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "https://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

## <a name="azure-storage"></a><span data-ttu-id="b9189-702">Armazenamento do Azure</span><span class="sxs-lookup"><span data-stu-id="b9189-702">Azure Storage</span></span>
<span data-ttu-id="b9189-703">Os serviços de armazenamento do Azure incluem armazenamento de tabela e blob, arquivos e filas de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="b9189-703">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="b9189-704">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-704">Retry mechanism</span></span>
<span data-ttu-id="b9189-705">As repetições ocorrem no nível de operação REST individuais e são uma parte integrante da implementação da API do cliente.</span><span class="sxs-lookup"><span data-stu-id="b9189-705">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="b9189-706">O SDK do armazenamento do cliente usa classes que implementam a [Interface IExtendedRetryPolicy](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy).</span><span class="sxs-lookup"><span data-stu-id="b9189-706">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy).</span></span>

<span data-ttu-id="b9189-707">Há diferentes implementações da interface.</span><span class="sxs-lookup"><span data-stu-id="b9189-707">There are different implementations of the interface.</span></span> <span data-ttu-id="b9189-708">Os clientes de armazenamento podem escolher dentre políticas especificamente desenvolvidas para acesso a tabelas, blobs e filas.</span><span class="sxs-lookup"><span data-stu-id="b9189-708">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="b9189-709">Cada implementação usa uma estratégia de repetição diferente que, basicamente, define o intervalo da repetição e outros detalhes.</span><span class="sxs-lookup"><span data-stu-id="b9189-709">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="b9189-710">As classes internas fornecem suporte a intervalos lineares (constantes) e exponenciais com intervalos de repetição de aleatoriedade.</span><span class="sxs-lookup"><span data-stu-id="b9189-710">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="b9189-711">Também não há uma política de repetição a ser usada quando outro processo está tratando repetições em um nível mais alto.</span><span class="sxs-lookup"><span data-stu-id="b9189-711">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="b9189-712">No entanto, você pode implementar suas próprias classes de repetição caso haja requisitos não fornecidos pelas classes internas.</span><span class="sxs-lookup"><span data-stu-id="b9189-712">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="b9189-713">As repetições alternativas serão alternadas entre o local do serviço de armazenamento primário e secundário se você estiver usando o RA-GRS (armazenamento com redundância geográfica com acesso de leitura) e o resultado da solicitação for um erro que pode ser repetido.</span><span class="sxs-lookup"><span data-stu-id="b9189-713">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="b9189-714">Consulte [Opções de redundância de armazenamento do Azure](/azure/storage/common/storage-redundancy) para saber mais.</span><span class="sxs-lookup"><span data-stu-id="b9189-714">See [Azure Storage Redundancy Options](/azure/storage/common/storage-redundancy) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="b9189-715">Configuração de política</span><span class="sxs-lookup"><span data-stu-id="b9189-715">Policy configuration</span></span>
<span data-ttu-id="b9189-716">As políticas de repetição são configuradas de modo programático.</span><span class="sxs-lookup"><span data-stu-id="b9189-716">Retry policies are configured programmatically.</span></span> <span data-ttu-id="b9189-717">Um procedimento comum é criar e popular uma instância **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** ou **QueueRequestOptions**.</span><span class="sxs-lookup"><span data-stu-id="b9189-717">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. 
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

<span data-ttu-id="b9189-718">A instância das opções de solicitação pode ser definida no cliente e todas as operações com o cliente usarão as opções de solicitação especificadas.</span><span class="sxs-lookup"><span data-stu-id="b9189-718">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="b9189-719">É possível substituir as opções de solicitação de cliente passando uma instância populada da classe de opções de solicitação como um parâmetro para os métodos de operação.</span><span class="sxs-lookup"><span data-stu-id="b9189-719">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="b9189-720">Você usa uma instância **OperationContext** para especificar o código a ser executado quando uma repetição ocorrer e quando uma operação tiver sido concluída.</span><span class="sxs-lookup"><span data-stu-id="b9189-720">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="b9189-721">Esse código pode coletar informações sobre a operação para uso em logs e telemetria.</span><span class="sxs-lookup"><span data-stu-id="b9189-721">This code can collect information about the operation for use in logs and telemetry.</span></span>

```csharp
// Set up notifications for an operation
var context = new OperationContext();
context.ClientRequestID = "some request id";
context.Retrying += (sender, args) =>
{
    /* Collect retry information */
};
context.RequestCompleted += (sender, args) =>
{
    /* Collect operation completion information */
};
var stats = await client.GetServiceStatsAsync(null, context);
```

<span data-ttu-id="b9189-722">Além de indicar se uma falha é adequada para repetição, as políticas estendidas de repetição retornam um objeto **RetryContext** que indica o número de repetições, os resultados da última solicitação e se a próxima repetição acontecerá no local primário ou secundário (consulte a tabela abaixo para obter detalhes).</span><span class="sxs-lookup"><span data-stu-id="b9189-722">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="b9189-723">As propriedades do objeto **RetryContext** podem ser usadas para decidir se e quando tentar uma repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-723">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="b9189-724">Para obter mais detalhes, consulte [Método IExtendedRetryPolicy.Evaluate](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate).</span><span class="sxs-lookup"><span data-stu-id="b9189-724">For more details, see [IExtendedRetryPolicy.Evaluate Method](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate).</span></span>

<span data-ttu-id="b9189-725">As tabelas a seguir mostram as configurações padrão para as políticas de repetição internas.</span><span class="sxs-lookup"><span data-stu-id="b9189-725">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="b9189-726">**Opções de solicitação**</span><span class="sxs-lookup"><span data-stu-id="b9189-726">**Request options**</span></span>

| <span data-ttu-id="b9189-727">**Configuração**</span><span class="sxs-lookup"><span data-stu-id="b9189-727">**Setting**</span></span> | <span data-ttu-id="b9189-728">**Valor padrão**</span><span class="sxs-lookup"><span data-stu-id="b9189-728">**Default value**</span></span> | <span data-ttu-id="b9189-729">**Significado**</span><span class="sxs-lookup"><span data-stu-id="b9189-729">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="b9189-730">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="b9189-730">MaximumExecutionTime</span></span> | <span data-ttu-id="b9189-731">Nenhum</span><span class="sxs-lookup"><span data-stu-id="b9189-731">None</span></span> | <span data-ttu-id="b9189-732">Tempo máximo de execução para a solicitação, incluindo todas as tentativas de repetição potenciais.</span><span class="sxs-lookup"><span data-stu-id="b9189-732">Maximum execution time for the request, including all potential retry attempts.</span></span> <span data-ttu-id="b9189-733">Se não for especificada, a quantidade de tempo que uma solicitação pode levar é ilimitada.</span><span class="sxs-lookup"><span data-stu-id="b9189-733">If it is not specified, then the amount of time that a request is permitted to take is unlimited.</span></span> <span data-ttu-id="b9189-734">Em outras palavras, a solicitação pode travar.</span><span class="sxs-lookup"><span data-stu-id="b9189-734">In other words, the request might hang.</span></span> |
| <span data-ttu-id="b9189-735">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="b9189-735">ServerTimeout</span></span> | <span data-ttu-id="b9189-736">Nenhum</span><span class="sxs-lookup"><span data-stu-id="b9189-736">None</span></span> | <span data-ttu-id="b9189-737">O intervalo do tempo limite do servidor para a solicitação (o valor é arredondado para segundos).</span><span class="sxs-lookup"><span data-stu-id="b9189-737">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="b9189-738">Se não for especificado, será usado o valor padrão para todas as solicitações ao servidor.</span><span class="sxs-lookup"><span data-stu-id="b9189-738">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="b9189-739">Normalmente, a melhor opção é omitir essa configuração para que o padrão de servidor seja usado.</span><span class="sxs-lookup"><span data-stu-id="b9189-739">Usually, the best option is to omit this setting so that the server default is used.</span></span> | 
| <span data-ttu-id="b9189-740">LocationMode</span><span class="sxs-lookup"><span data-stu-id="b9189-740">LocationMode</span></span> | <span data-ttu-id="b9189-741">Nenhum</span><span class="sxs-lookup"><span data-stu-id="b9189-741">None</span></span> | <span data-ttu-id="b9189-742">Se a conta de armazenamento for criada com a opção de replicação RA-GRS (armazenamento com redundância geográfica com acesso de leitura), você poderá usar o modo de local para indicar qual local deve receber a solicitação.</span><span class="sxs-lookup"><span data-stu-id="b9189-742">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="b9189-743">Por exemplo, se **PrimaryThenSecondary** for especificado, as solicitações sempre serão enviadas para o local primário primeiro.</span><span class="sxs-lookup"><span data-stu-id="b9189-743">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="b9189-744">Se uma solicitação falhar, ela será enviada para o local secundário.</span><span class="sxs-lookup"><span data-stu-id="b9189-744">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="b9189-745">RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="b9189-745">RetryPolicy</span></span> | <span data-ttu-id="b9189-746">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="b9189-746">ExponentialPolicy</span></span> | <span data-ttu-id="b9189-747">Veja abaixo os detalhes de cada opção.</span><span class="sxs-lookup"><span data-stu-id="b9189-747">See below for details of each option.</span></span> |

<span data-ttu-id="b9189-748">**Política exponencial**</span><span class="sxs-lookup"><span data-stu-id="b9189-748">**Exponential policy**</span></span> 

| <span data-ttu-id="b9189-749">**Configuração**</span><span class="sxs-lookup"><span data-stu-id="b9189-749">**Setting**</span></span> | <span data-ttu-id="b9189-750">**Valor padrão**</span><span class="sxs-lookup"><span data-stu-id="b9189-750">**Default value**</span></span> | <span data-ttu-id="b9189-751">**Significado**</span><span class="sxs-lookup"><span data-stu-id="b9189-751">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="b9189-752">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="b9189-752">maxAttempt</span></span> | <span data-ttu-id="b9189-753">3</span><span class="sxs-lookup"><span data-stu-id="b9189-753">3</span></span> | <span data-ttu-id="b9189-754">Número de tentativas de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-754">Number of retry attempts.</span></span> |
| <span data-ttu-id="b9189-755">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="b9189-755">deltaBackoff</span></span> | <span data-ttu-id="b9189-756">4 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-756">4 seconds</span></span> | <span data-ttu-id="b9189-757">Intervalo de retirada entre repetições.</span><span class="sxs-lookup"><span data-stu-id="b9189-757">Back-off interval between retries.</span></span> <span data-ttu-id="b9189-758">Múltiplos desse período de tempo, incluindo um elemento aleatório, serão usados para tentativas de repetição subsequentes.</span><span class="sxs-lookup"><span data-stu-id="b9189-758">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="b9189-759">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="b9189-759">MinBackoff</span></span> | <span data-ttu-id="b9189-760">3 Segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-760">3 seconds</span></span> | <span data-ttu-id="b9189-761">Adicionado a todos os intervalos de repetição calculados de deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="b9189-761">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="b9189-762">Esse valor não pode ser alterado.</span><span class="sxs-lookup"><span data-stu-id="b9189-762">This value cannot be changed.</span></span>
| <span data-ttu-id="b9189-763">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="b9189-763">MaxBackoff</span></span> | <span data-ttu-id="b9189-764">120 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-764">120 seconds</span></span> | <span data-ttu-id="b9189-765">MaxBackoff será usado se o intervalo de repetição calculado for maior que MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="b9189-765">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="b9189-766">Esse valor não pode ser alterado.</span><span class="sxs-lookup"><span data-stu-id="b9189-766">This value cannot be changed.</span></span> |

<span data-ttu-id="b9189-767">**Política linear**</span><span class="sxs-lookup"><span data-stu-id="b9189-767">**Linear policy**</span></span>

| <span data-ttu-id="b9189-768">**Configuração**</span><span class="sxs-lookup"><span data-stu-id="b9189-768">**Setting**</span></span> | <span data-ttu-id="b9189-769">**Valor padrão**</span><span class="sxs-lookup"><span data-stu-id="b9189-769">**Default value**</span></span> | <span data-ttu-id="b9189-770">**Significado**</span><span class="sxs-lookup"><span data-stu-id="b9189-770">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="b9189-771">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="b9189-771">maxAttempt</span></span> | <span data-ttu-id="b9189-772">3</span><span class="sxs-lookup"><span data-stu-id="b9189-772">3</span></span> | <span data-ttu-id="b9189-773">Número de tentativas de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-773">Number of retry attempts.</span></span> |
| <span data-ttu-id="b9189-774">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="b9189-774">deltaBackoff</span></span> | <span data-ttu-id="b9189-775">30 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-775">30 seconds</span></span> | <span data-ttu-id="b9189-776">Intervalo de retirada entre repetições.</span><span class="sxs-lookup"><span data-stu-id="b9189-776">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="b9189-777">Diretriz de uso de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-777">Retry usage guidance</span></span>
<span data-ttu-id="b9189-778">Considere as seguintes diretrizes ao acessar serviços de armazenamento do Azure usando a API do cliente de armazenamento:</span><span class="sxs-lookup"><span data-stu-id="b9189-778">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="b9189-779">Use as políticas de repetição internas do namespace Microsoft.WindowsAzure.Storage.RetryPolicies onde elas forem apropriadas para suas necessidades.</span><span class="sxs-lookup"><span data-stu-id="b9189-779">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="b9189-780">Na maioria dos casos, essas políticas serão suficientes.</span><span class="sxs-lookup"><span data-stu-id="b9189-780">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="b9189-781">Use a política **ExponentialRetry** em operações em lote, tarefas em segundo plano ou cenários não interativos.</span><span class="sxs-lookup"><span data-stu-id="b9189-781">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="b9189-782">Nesses cenários, geralmente você pode permitir mais tempo para que o serviço se recupere — com uma chance consequentemente aumentada da operação ser bem-sucedida.</span><span class="sxs-lookup"><span data-stu-id="b9189-782">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="b9189-783">Considere especificar a propriedade **MaximumExecutionTime** do parâmetro **RequestOptions** para limitar o tempo de execução total, mas leve em consideração o tipo e o tamanho da operação ao escolher um valor de tempo limite.</span><span class="sxs-lookup"><span data-stu-id="b9189-783">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="b9189-784">Se você precisar implementar uma repetição personalizada, evite criar wrappers em torno das classes do cliente de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="b9189-784">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="b9189-785">Em vez disso, use os recursos para estender as políticas existentes por meio da interface **IExtendedRetryPolicy** .</span><span class="sxs-lookup"><span data-stu-id="b9189-785">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="b9189-786">Se você estiver usando RA-GRS (armazenamento com redundância geográfica com acesso de leitura), será possível usar o **LocationMode** para especificar que as tentativas de repetição acessarão a cópia secundária somente leitura do repositório caso o acesso primário falhe.</span><span class="sxs-lookup"><span data-stu-id="b9189-786">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="b9189-787">No entanto, ao usar essa opção, você deve garantir que seu aplicativo possa trabalhar perfeitamente com dados que talvez estejam obsoletos, se a replicação do repositório primário ainda não tiver sido concluída.</span><span class="sxs-lookup"><span data-stu-id="b9189-787">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="b9189-788">Considere começar com as seguintes configurações para operações de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-788">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="b9189-789">Essas são configurações de uso geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.</span><span class="sxs-lookup"><span data-stu-id="b9189-789">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="b9189-790">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="b9189-790">**Context**</span></span> | <span data-ttu-id="b9189-791">**Exemplo de latênciaE2E<br />máxima de destino**</span><span class="sxs-lookup"><span data-stu-id="b9189-791">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="b9189-792">**Política de repetição**</span><span class="sxs-lookup"><span data-stu-id="b9189-792">**Retry policy**</span></span> | <span data-ttu-id="b9189-793">**Configurações**</span><span class="sxs-lookup"><span data-stu-id="b9189-793">**Settings**</span></span> | <span data-ttu-id="b9189-794">**Valores**</span><span class="sxs-lookup"><span data-stu-id="b9189-794">**Values**</span></span> | <span data-ttu-id="b9189-795">**Como funciona**</span><span class="sxs-lookup"><span data-stu-id="b9189-795">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="b9189-796">Interativo, interface de usuário</span><span class="sxs-lookup"><span data-stu-id="b9189-796">Interactive, UI,</span></span><br /><span data-ttu-id="b9189-797">ou primeiro plano</span><span class="sxs-lookup"><span data-stu-id="b9189-797">or foreground</span></span> |<span data-ttu-id="b9189-798">2 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-798">2 seconds</span></span> |<span data-ttu-id="b9189-799">Linear</span><span class="sxs-lookup"><span data-stu-id="b9189-799">Linear</span></span> |<span data-ttu-id="b9189-800">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="b9189-800">maxAttempt</span></span><br /><span data-ttu-id="b9189-801">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="b9189-801">deltaBackoff</span></span> |<span data-ttu-id="b9189-802">3</span><span class="sxs-lookup"><span data-stu-id="b9189-802">3</span></span><br /><span data-ttu-id="b9189-803">500 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-803">500 ms</span></span> |<span data-ttu-id="b9189-804">1ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-804">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="b9189-805">2ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-805">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="b9189-806">3ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="b9189-806">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="b9189-807">Segundo plano</span><span class="sxs-lookup"><span data-stu-id="b9189-807">Background</span></span><br /><span data-ttu-id="b9189-808">ou lote</span><span class="sxs-lookup"><span data-stu-id="b9189-808">or batch</span></span> |<span data-ttu-id="b9189-809">30 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-809">30 seconds</span></span> |<span data-ttu-id="b9189-810">Exponencial</span><span class="sxs-lookup"><span data-stu-id="b9189-810">Exponential</span></span> |<span data-ttu-id="b9189-811">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="b9189-811">maxAttempt</span></span><br /><span data-ttu-id="b9189-812">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="b9189-812">deltaBackoff</span></span> |<span data-ttu-id="b9189-813">5</span><span class="sxs-lookup"><span data-stu-id="b9189-813">5</span></span><br /><span data-ttu-id="b9189-814">4 segundos</span><span class="sxs-lookup"><span data-stu-id="b9189-814">4 seconds</span></span> |<span data-ttu-id="b9189-815">1ª tentativa — intervalo de ~3 s</span><span class="sxs-lookup"><span data-stu-id="b9189-815">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="b9189-816">2ª tentativa — intervalo de ~7 s</span><span class="sxs-lookup"><span data-stu-id="b9189-816">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="b9189-817">3ª tentativa — intervalo de ~15 s</span><span class="sxs-lookup"><span data-stu-id="b9189-817">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="b9189-818">Telemetria</span><span class="sxs-lookup"><span data-stu-id="b9189-818">Telemetry</span></span>
<span data-ttu-id="b9189-819">As tentativas de repetição são registradas em um **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="b9189-819">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="b9189-820">Você deve configurar um **TraceListener** para capturar os eventos e gravá-los em um log de destino apropriado.</span><span class="sxs-lookup"><span data-stu-id="b9189-820">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="b9189-821">É possível usar o **TextWriterTraceListener** ou **XmlWriterTraceListener** para gravar os dados em um arquivo de log, o **EventLogTraceListener** para gravar no Log de Eventos do Windows ou o **EventProviderTraceListener** para gravar dados de rastreamento no subsistema ETW.</span><span class="sxs-lookup"><span data-stu-id="b9189-821">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="b9189-822">Você também pode configurar a liberação automática do buffer e o detalhamento dos eventos que serão registrados (por exemplo, Erro, Aviso, Informativo e Detalhado).</span><span class="sxs-lookup"><span data-stu-id="b9189-822">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="b9189-823">Para saber mais, consulte [Registro em log no lado do cliente com a biblioteca do cliente de armazenamento para .NET](/rest/api/storageservices/Client-side-Logging-with-the-.NET-Storage-Client-Library).</span><span class="sxs-lookup"><span data-stu-id="b9189-823">For more information, see [Client-side Logging with the .NET Storage Client Library](/rest/api/storageservices/Client-side-Logging-with-the-.NET-Storage-Client-Library).</span></span>

<span data-ttu-id="b9189-824">As operações podem receber uma instância **OperationContext**, que expõe um evento **Retrying** que pode ser usado para anexar a lógica personalizada de telemetria.</span><span class="sxs-lookup"><span data-stu-id="b9189-824">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="b9189-825">Para saber mais, consulte [Evento OperationContext.Retrying](/dotnet/api/microsoft.windowsazure.storage.operationcontext.retrying).</span><span class="sxs-lookup"><span data-stu-id="b9189-825">For more information, see [OperationContext.Retrying Event](/dotnet/api/microsoft.windowsazure.storage.operationcontext.retrying).</span></span>

### <a name="examples"></a><span data-ttu-id="b9189-826">Exemplos</span><span class="sxs-lookup"><span data-stu-id="b9189-826">Examples</span></span>
<span data-ttu-id="b9189-827">O exemplo de código a seguir mostra como criar duas instâncias **TableRequestOptions** com diferentes configurações de repetição; uma para solicitações interativas e outra para solicitações em segundo plano.</span><span class="sxs-lookup"><span data-stu-id="b9189-827">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="b9189-828">O exemplo define essas duas políticas de repetição no cliente para que elas sejam aplicadas a todas as solicitações, assim como define a estratégia interativa em uma solicitação específica de modo que ela substitua as configurações padrão aplicadas ao cliente.</span><span class="sxs-lookup"><span data-stu-id="b9189-828">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. 
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="b9189-829">Mais informações</span><span class="sxs-lookup"><span data-stu-id="b9189-829">More information</span></span>
* [<span data-ttu-id="b9189-830">Recomendações de política de repetição da biblioteca do cliente de armazenamento do Azure</span><span class="sxs-lookup"><span data-stu-id="b9189-830">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="b9189-831">Biblioteca do cliente de armazenamento 2.0 – implementando políticas de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-831">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](https://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="b9189-832">Diretrizes gerais de repetição e REST</span><span class="sxs-lookup"><span data-stu-id="b9189-832">General REST and retry guidelines</span></span>
<span data-ttu-id="b9189-833">Considere o seguinte ao acessar os serviços do Azure ou de terceiros:</span><span class="sxs-lookup"><span data-stu-id="b9189-833">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="b9189-834">Use uma abordagem sistemática para gerenciar repetições, talvez como código reutilizável, para que você possa aplicar uma metodologia consistente em todos os clientes e soluções.</span><span class="sxs-lookup"><span data-stu-id="b9189-834">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="b9189-835">Considere o uso de uma estrutura de repetição, como [Polly][polly], para gerenciar as repetições de tentativa se o cliente ou serviço de destino não tiver nenhum mecanismo de repetição interno.</span><span class="sxs-lookup"><span data-stu-id="b9189-835">Consider using a retry framework such as [Polly][polly] to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="b9189-836">Isso ajudará você a implementar um comportamento de repetição consistente, bem como pode fornecer uma estratégia de repetição padrão adequada para o serviço de destino.</span><span class="sxs-lookup"><span data-stu-id="b9189-836">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="b9189-837">No entanto, talvez seja necessário criar código de repetição personalizado para serviços que tenham comportamento não padrão, que não dependem de exceções para indicar falhas transitórias, ou se desejar, use uma resposta **Retry-Response** para gerenciar o comportamento de repetição.</span><span class="sxs-lookup"><span data-stu-id="b9189-837">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="b9189-838">A lógica de detecção transitória dependerá da API de cliente real que você usa para invocar as chamadas REST.</span><span class="sxs-lookup"><span data-stu-id="b9189-838">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="b9189-839">Alguns clientes, como a classe mais recente **HttpClient** , não lançam exceções para solicitações concluídas com um código de status HTTP sem sucesso.</span><span class="sxs-lookup"><span data-stu-id="b9189-839">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> 
* <span data-ttu-id="b9189-840">O código de status HTTP retornado do serviço pode ajudar a indicar se a falha é transitória.</span><span class="sxs-lookup"><span data-stu-id="b9189-840">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="b9189-841">Talvez seja necessário examinar as exceções geradas por um cliente ou pela estrutura de repetição para acessar o código de status ou determinar o tipo de exceção equivalente.</span><span class="sxs-lookup"><span data-stu-id="b9189-841">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="b9189-842">Os seguintes códigos HTTP geralmente indicam que uma repetição é apropriada:</span><span class="sxs-lookup"><span data-stu-id="b9189-842">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="b9189-843">408 Tempo Limite da Solicitação</span><span class="sxs-lookup"><span data-stu-id="b9189-843">408 Request Timeout</span></span>
  * <span data-ttu-id="b9189-844">429 Número excessivo de solicitações</span><span class="sxs-lookup"><span data-stu-id="b9189-844">429 Too Many Requests</span></span>
  * <span data-ttu-id="b9189-845">500 Erro Interno do Servidor</span><span class="sxs-lookup"><span data-stu-id="b9189-845">500 Internal Server Error</span></span>
  * <span data-ttu-id="b9189-846">502 Gateway Incorreto</span><span class="sxs-lookup"><span data-stu-id="b9189-846">502 Bad Gateway</span></span>
  * <span data-ttu-id="b9189-847">503 Serviço Indisponível</span><span class="sxs-lookup"><span data-stu-id="b9189-847">503 Service Unavailable</span></span>
  * <span data-ttu-id="b9189-848">504 Tempo Limite do Gateway</span><span class="sxs-lookup"><span data-stu-id="b9189-848">504 Gateway Timeout</span></span>
* <span data-ttu-id="b9189-849">Se você basear suas lógica de repetição em exceções, geralmente o que se segue indica uma falha transitória onde nenhuma conexão pode ser estabelecida:</span><span class="sxs-lookup"><span data-stu-id="b9189-849">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="b9189-850">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="b9189-850">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="b9189-851">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="b9189-851">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="b9189-852">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="b9189-852">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="b9189-853">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="b9189-853">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="b9189-854">No caso de um status de serviço indisponível, o serviço pode indicar o atraso apropriado antes de tentar a repetição no cabeçalho da resposta **Retry-After** ou em um cabeçalho personalizado diferente.</span><span class="sxs-lookup"><span data-stu-id="b9189-854">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="b9189-855">Os serviços também podem enviar informações adicionais como cabeçalhos personalizados ou inseridos no conteúdo da resposta.</span><span class="sxs-lookup"><span data-stu-id="b9189-855">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> 
* <span data-ttu-id="b9189-856">Não tente a repetição para códigos de status que representam erros de cliente (erros no intervalo 4xx), exceto para um 408 Tempo Limite de Solicitação.</span><span class="sxs-lookup"><span data-stu-id="b9189-856">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="b9189-857">Teste minuciosamente seus mecanismos e estratégias de repetição sob diversas condições, como estado diferente de rede e cargas variáveis de sistema.</span><span class="sxs-lookup"><span data-stu-id="b9189-857">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="b9189-858">Estratégias de repetição</span><span class="sxs-lookup"><span data-stu-id="b9189-858">Retry strategies</span></span>
<span data-ttu-id="b9189-859">Veja a seguir os tipos comuns de intervalo de estratégias de repetição:</span><span class="sxs-lookup"><span data-stu-id="b9189-859">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="b9189-860">**Exponencial**.</span><span class="sxs-lookup"><span data-stu-id="b9189-860">**Exponential**.</span></span> <span data-ttu-id="b9189-861">Uma política de repetição que executa um determinado número de repetições usando uma abordagem de retirada exponencial aleatória para determinar o intervalo entre as repetições.</span><span class="sxs-lookup"><span data-stu-id="b9189-861">A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="b9189-862">Por exemplo: </span><span class="sxs-lookup"><span data-stu-id="b9189-862">For example:</span></span>

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

* <span data-ttu-id="b9189-863">**Incremental**.</span><span class="sxs-lookup"><span data-stu-id="b9189-863">**Incremental**.</span></span> <span data-ttu-id="b9189-864">Uma estratégia de repetição com um número especificado de tentativas de repetição e um intervalo de tempo incremental entre entradas.</span><span class="sxs-lookup"><span data-stu-id="b9189-864">A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="b9189-865">Por exemplo: </span><span class="sxs-lookup"><span data-stu-id="b9189-865">For example:</span></span>

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

* <span data-ttu-id="b9189-866">**LinearRetry**.</span><span class="sxs-lookup"><span data-stu-id="b9189-866">**LinearRetry**.</span></span> <span data-ttu-id="b9189-867">Uma política de repetição que executa um número especificado de repetições usando um intervalo de tempo fixo especificado entre as repetições.</span><span class="sxs-lookup"><span data-stu-id="b9189-867">A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="b9189-868">Por exemplo: </span><span class="sxs-lookup"><span data-stu-id="b9189-868">For example:</span></span>

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="b9189-869">Falha transitória manipulada com a Polly</span><span class="sxs-lookup"><span data-stu-id="b9189-869">Transient fault handling with Polly</span></span>
<span data-ttu-id="b9189-870">[Polly] [ polly] é uma biblioteca para manipular programaticamente novas tentativas e estratégias de [disjuntor] [ circuit-breaker].</span><span class="sxs-lookup"><span data-stu-id="b9189-870">[Polly][polly] is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="b9189-871">O projeto Polly é um membro da [Fundação .NET][dotnet-foundation].</span><span class="sxs-lookup"><span data-stu-id="b9189-871">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="b9189-872">Para serviços em que o cliente não oferece suporte a novas tentativas, a Polly é uma alternativa válida e evita a necessidade de escrever código de repetição personalizado, que pode ser difícil de implementar corretamente.</span><span class="sxs-lookup"><span data-stu-id="b9189-872">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="b9189-873">A Polly também fornece uma maneira de rastrear erros, para que você possa fazer novas tentativas.</span><span class="sxs-lookup"><span data-stu-id="b9189-873">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>


<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx


### <a name="more-information"></a><span data-ttu-id="b9189-877">Mais informações</span><span class="sxs-lookup"><span data-stu-id="b9189-877">More information</span></span>
* [<span data-ttu-id="b9189-878">Resiliência de conexão</span><span class="sxs-lookup"><span data-stu-id="b9189-878">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="b9189-879">Pontos de dados - Core EF 1.1</span><span class="sxs-lookup"><span data-stu-id="b9189-879">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)


