---
title: "Diretriz específica do serviço de repetição"
description: "Diretriz específica de serviço para configurar o mecanismo de repetição."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 6aba60dc3a60e96e59e2034d4a1e380e0f1c996a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="30594-103">Repetir as diretrizes para serviços específicos</span><span class="sxs-lookup"><span data-stu-id="30594-103">Retry guidance for specific services</span></span>

<span data-ttu-id="30594-104">A maioria dos serviços do Azure e SDKs do cliente inclui um mecanismo de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="30594-105">No entanto, eles são diferentes porque cada serviço apresenta características e requisitos diferentes, e cada mecanismo de repetição é ajustado para um serviço específico.</span><span class="sxs-lookup"><span data-stu-id="30594-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="30594-106">Este guia resume os recursos do mecanismo de repetição para a maioria dos serviços do Azure, além de incluir informações que ajudam a usar, a adaptar ou a estender o mecanismo de repetição para o serviço em questão.</span><span class="sxs-lookup"><span data-stu-id="30594-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="30594-107">Para obter orientação geral sobre o tratamento de falhas transitórias e como repetir conexões e operações em serviços e recursos, consulte [Diretriz de repetição](./transient-faults.md).</span><span class="sxs-lookup"><span data-stu-id="30594-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="30594-108">A tabela a seguir resume os recursos de repetição para os serviços do Azure descritos nesta diretriz.</span><span class="sxs-lookup"><span data-stu-id="30594-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="30594-109">**Serviço**</span><span class="sxs-lookup"><span data-stu-id="30594-109">**Service**</span></span> | <span data-ttu-id="30594-110">**Recursos de repetição**</span><span class="sxs-lookup"><span data-stu-id="30594-110">**Retry capabilities**</span></span> | <span data-ttu-id="30594-111">**Configuração de política**</span><span class="sxs-lookup"><span data-stu-id="30594-111">**Policy configuration**</span></span> | <span data-ttu-id="30594-112">**Escopo**</span><span class="sxs-lookup"><span data-stu-id="30594-112">**Scope**</span></span> | <span data-ttu-id="30594-113">**Recursos de telemetria**</span><span class="sxs-lookup"><span data-stu-id="30594-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="30594-114">**[Armazenamento do Azure](#azure-storage-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="30594-114">**[Azure Storage](#azure-storage-retry-guidelines)**</span></span> |<span data-ttu-id="30594-115">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="30594-115">Native in client</span></span> |<span data-ttu-id="30594-116">Programático</span><span class="sxs-lookup"><span data-stu-id="30594-116">Programmatic</span></span> |<span data-ttu-id="30594-117">Operações individuais e de cliente</span><span class="sxs-lookup"><span data-stu-id="30594-117">Client and individual operations</span></span> |<span data-ttu-id="30594-118">TraceSource</span><span class="sxs-lookup"><span data-stu-id="30594-118">TraceSource</span></span> |
| <span data-ttu-id="30594-119">**[Banco de dados SQL com o Entity Framework](#sql-database-using-entity-framework-6-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="30594-119">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6-retry-guidelines)**</span></span> |<span data-ttu-id="30594-120">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="30594-120">Native in client</span></span> |<span data-ttu-id="30594-121">Programático</span><span class="sxs-lookup"><span data-stu-id="30594-121">Programmatic</span></span> |<span data-ttu-id="30594-122">Global por AppDomain</span><span class="sxs-lookup"><span data-stu-id="30594-122">Global per AppDomain</span></span> |<span data-ttu-id="30594-123">Nenhum</span><span class="sxs-lookup"><span data-stu-id="30594-123">None</span></span> |
| <span data-ttu-id="30594-124">**[Banco de dados SQL com Entity Framework Core](#sql-database-using-entity-framework-core-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="30594-124">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core-retry-guidelines)**</span></span> |<span data-ttu-id="30594-125">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="30594-125">Native in client</span></span> |<span data-ttu-id="30594-126">Programático</span><span class="sxs-lookup"><span data-stu-id="30594-126">Programmatic</span></span> |<span data-ttu-id="30594-127">Global por AppDomain</span><span class="sxs-lookup"><span data-stu-id="30594-127">Global per AppDomain</span></span> |<span data-ttu-id="30594-128">Nenhum</span><span class="sxs-lookup"><span data-stu-id="30594-128">None</span></span> |
| <span data-ttu-id="30594-129">**[Banco de dados SQL com ADO.NET](#sql-database-using-adonet-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="30594-129">**[SQL Database with ADO.NET](#sql-database-using-adonet-retry-guidelines)**</span></span> |[<span data-ttu-id="30594-130">Polly</span><span class="sxs-lookup"><span data-stu-id="30594-130">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="30594-131">Programático e declarativo</span><span class="sxs-lookup"><span data-stu-id="30594-131">Declarative and programmatic</span></span> |<span data-ttu-id="30594-132">Instruções ou blocos de código únicos</span><span class="sxs-lookup"><span data-stu-id="30594-132">Single statements or blocks of code</span></span> |<span data-ttu-id="30594-133">Personalizado</span><span class="sxs-lookup"><span data-stu-id="30594-133">Custom</span></span> |
| <span data-ttu-id="30594-134">**[Barramento de Serviço](#service-bus-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="30594-134">**[Service Bus](#service-bus-retry-guidelines)**</span></span> |<span data-ttu-id="30594-135">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="30594-135">Native in client</span></span> |<span data-ttu-id="30594-136">Programático</span><span class="sxs-lookup"><span data-stu-id="30594-136">Programmatic</span></span> |<span data-ttu-id="30594-137">Gerenciador de Namespace, Messaging Factory e Cliente</span><span class="sxs-lookup"><span data-stu-id="30594-137">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="30594-138">ETW</span><span class="sxs-lookup"><span data-stu-id="30594-138">ETW</span></span> |
| <span data-ttu-id="30594-139">**[Cache Redis do Azure](#azure-redis-cache-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="30594-139">**[Azure Redis Cache](#azure-redis-cache-retry-guidelines)**</span></span> |<span data-ttu-id="30594-140">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="30594-140">Native in client</span></span> |<span data-ttu-id="30594-141">Programático</span><span class="sxs-lookup"><span data-stu-id="30594-141">Programmatic</span></span> |<span data-ttu-id="30594-142">Cliente</span><span class="sxs-lookup"><span data-stu-id="30594-142">Client</span></span> |<span data-ttu-id="30594-143">TextWriter</span><span class="sxs-lookup"><span data-stu-id="30594-143">TextWriter</span></span> |
| <span data-ttu-id="30594-144">**[API do DocumentDB](#documentdb-api-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="30594-144">**[DocumentDB API](#documentdb-api-retry-guidelines)**</span></span> |<span data-ttu-id="30594-145">Nativo no serviço</span><span class="sxs-lookup"><span data-stu-id="30594-145">Native in service</span></span> |<span data-ttu-id="30594-146">Não configurável</span><span class="sxs-lookup"><span data-stu-id="30594-146">Non-configurable</span></span> |<span data-ttu-id="30594-147">Global</span><span class="sxs-lookup"><span data-stu-id="30594-147">Global</span></span> |<span data-ttu-id="30594-148">TraceSource</span><span class="sxs-lookup"><span data-stu-id="30594-148">TraceSource</span></span> |
| <span data-ttu-id="30594-149">**[Azure Search](#azure-storage-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="30594-149">**[Azure Search](#azure-storage-retry-guidelines)**</span></span> |<span data-ttu-id="30594-150">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="30594-150">Native in client</span></span> |<span data-ttu-id="30594-151">Programático</span><span class="sxs-lookup"><span data-stu-id="30594-151">Programmatic</span></span> |<span data-ttu-id="30594-152">Cliente</span><span class="sxs-lookup"><span data-stu-id="30594-152">Client</span></span> |<span data-ttu-id="30594-153">ETW ou Personalizado</span><span class="sxs-lookup"><span data-stu-id="30594-153">ETW or Custom</span></span> |
| <span data-ttu-id="30594-154">**[Azure Active Directory](#azure-active-directory-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="30594-154">**[Azure Active Directory](#azure-active-directory-retry-guidelines)**</span></span> |<span data-ttu-id="30594-155">Nativo na biblioteca ADAL</span><span class="sxs-lookup"><span data-stu-id="30594-155">Native in ADAL library</span></span> |<span data-ttu-id="30594-156">Incorporado na biblioteca ADAL</span><span class="sxs-lookup"><span data-stu-id="30594-156">Embeded into ADAL library</span></span> |<span data-ttu-id="30594-157">Interna</span><span class="sxs-lookup"><span data-stu-id="30594-157">Internal</span></span> |<span data-ttu-id="30594-158">Nenhum</span><span class="sxs-lookup"><span data-stu-id="30594-158">None</span></span> |
| <span data-ttu-id="30594-159">**[Service Fabric](#service-fabric-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="30594-159">**[Service Fabric](#service-fabric-retry-guidelines)**</span></span> |<span data-ttu-id="30594-160">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="30594-160">Native in client</span></span> |<span data-ttu-id="30594-161">Programático</span><span class="sxs-lookup"><span data-stu-id="30594-161">Programmatic</span></span> |<span data-ttu-id="30594-162">Cliente</span><span class="sxs-lookup"><span data-stu-id="30594-162">Client</span></span> |<span data-ttu-id="30594-163">Nenhum</span><span class="sxs-lookup"><span data-stu-id="30594-163">None</span></span> | 
| <span data-ttu-id="30594-164">**[Hubs de eventos do Azure](#azure-event-hubs-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="30594-164">**[Azure Event Hubs](#azure-event-hubs-retry-guidelines)**</span></span> |<span data-ttu-id="30594-165">Nativo no cliente</span><span class="sxs-lookup"><span data-stu-id="30594-165">Native in client</span></span> |<span data-ttu-id="30594-166">Programático</span><span class="sxs-lookup"><span data-stu-id="30594-166">Programmatic</span></span> |<span data-ttu-id="30594-167">Cliente</span><span class="sxs-lookup"><span data-stu-id="30594-167">Client</span></span> |<span data-ttu-id="30594-168">Nenhum</span><span class="sxs-lookup"><span data-stu-id="30594-168">None</span></span> |

> [!NOTE]
> <span data-ttu-id="30594-169">Para a maioria dos mecanismos de repetição internos do Azure, atualmente não há meios de aplicar uma política de repetição diferente para diferentes tipos de erro ou exceção além da funcionalidade incluída na política de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-169">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception beyond the functionality include in the retry policy.</span></span> <span data-ttu-id="30594-170">Portanto, a melhor diretriz disponível neste momento em que este documento está sendo escrito é configurar uma política que forneça uma média ideal de desempenho e disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="30594-170">Therefore, the best guidance available at the time of writing is to configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="30594-171">Uma maneira de ajustar a política é analisar arquivos de log para determinar o tipo de falha transitória que está ocorrendo.</span><span class="sxs-lookup"><span data-stu-id="30594-171">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> <span data-ttu-id="30594-172">Por exemplo, se a maioria dos erros estiver relacionada aos problemas de conectividade de rede, você poderá tentar uma repetição imediata em vez de aguardar um longo período para a primeira repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-172">For example, if the majority of errors are related to network connectivity issues, you might attempt an immediate retry rather than wait a long time for the first retry.</span></span>
>
>

## <a name="azure-storage-retry-guidelines"></a><span data-ttu-id="30594-173">Diretrizes de repetição para o Armazenamento do Azure</span><span class="sxs-lookup"><span data-stu-id="30594-173">Azure Storage retry guidelines</span></span>
<span data-ttu-id="30594-174">Os serviços de armazenamento do Azure incluem armazenamento de tabela e blob, arquivos e filas de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="30594-174">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="30594-175">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-175">Retry mechanism</span></span>
<span data-ttu-id="30594-176">As repetições ocorrem no nível de operação REST individuais e são uma parte integrante da implementação da API do cliente.</span><span class="sxs-lookup"><span data-stu-id="30594-176">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="30594-177">O SDK do armazenamento do cliente usa classes que implementam a [Interface IExtendedRetryPolicy](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-177">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span></span>

<span data-ttu-id="30594-178">Há diferentes implementações da interface.</span><span class="sxs-lookup"><span data-stu-id="30594-178">There are different implementations of the interface.</span></span> <span data-ttu-id="30594-179">Os clientes de armazenamento podem escolher dentre políticas especificamente desenvolvidas para acesso a tabelas, blobs e filas.</span><span class="sxs-lookup"><span data-stu-id="30594-179">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="30594-180">Cada implementação usa uma estratégia de repetição diferente que, basicamente, define o intervalo da repetição e outros detalhes.</span><span class="sxs-lookup"><span data-stu-id="30594-180">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="30594-181">As classes internas fornecem suporte a intervalos lineares (constantes) e exponenciais com intervalos de repetição de aleatoriedade.</span><span class="sxs-lookup"><span data-stu-id="30594-181">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="30594-182">Também não há uma política de repetição a ser usada quando outro processo está tratando repetições em um nível mais alto.</span><span class="sxs-lookup"><span data-stu-id="30594-182">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="30594-183">No entanto, você pode implementar suas próprias classes de repetição caso haja requisitos não fornecidos pelas classes internas.</span><span class="sxs-lookup"><span data-stu-id="30594-183">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="30594-184">As repetições alternativas serão alternadas entre o local do serviço de armazenamento primário e secundário se você estiver usando o RA-GRS (armazenamento com redundância geográfica com acesso de leitura) e o resultado da solicitação for um erro que pode ser repetido.</span><span class="sxs-lookup"><span data-stu-id="30594-184">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="30594-185">Consulte [Opções de redundância de armazenamento do Azure](http://msdn.microsoft.com/library/azure/dn727290.aspx) para saber mais.</span><span class="sxs-lookup"><span data-stu-id="30594-185">See [Azure Storage Redundancy Options](http://msdn.microsoft.com/library/azure/dn727290.aspx) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="30594-186">Configuração de política</span><span class="sxs-lookup"><span data-stu-id="30594-186">Policy configuration</span></span>
<span data-ttu-id="30594-187">As políticas de repetição são configuradas de modo programático.</span><span class="sxs-lookup"><span data-stu-id="30594-187">Retry policies are configured programmatically.</span></span> <span data-ttu-id="30594-188">Um procedimento comum é criar e popular uma instância **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** ou **QueueRequestOptions**.</span><span class="sxs-lookup"><span data-stu-id="30594-188">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. Maximum value up to 10 seconds.
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

<span data-ttu-id="30594-189">A instância das opções de solicitação pode ser definida no cliente e todas as operações com o cliente usarão as opções de solicitação especificadas.</span><span class="sxs-lookup"><span data-stu-id="30594-189">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="30594-190">É possível substituir as opções de solicitação de cliente passando uma instância populada da classe de opções de solicitação como um parâmetro para os métodos de operação.</span><span class="sxs-lookup"><span data-stu-id="30594-190">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="30594-191">Você usa uma instância **OperationContext** para especificar o código a ser executado quando uma repetição ocorrer e quando uma operação tiver sido concluída.</span><span class="sxs-lookup"><span data-stu-id="30594-191">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="30594-192">Esse código pode coletar informações sobre a operação para uso em logs e telemetria.</span><span class="sxs-lookup"><span data-stu-id="30594-192">This code can collect information about the operation for use in logs and telemetry.</span></span>

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

<span data-ttu-id="30594-193">Além de indicar se uma falha é adequada para repetição, as políticas estendidas de repetição retornam um objeto **RetryContext** que indica o número de repetições, os resultados da última solicitação e se a próxima repetição acontecerá no local primário ou secundário (consulte a tabela abaixo para obter detalhes).</span><span class="sxs-lookup"><span data-stu-id="30594-193">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="30594-194">As propriedades do objeto **RetryContext** podem ser usadas para decidir se e quando tentar uma repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-194">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="30594-195">Para obter mais detalhes, consulte [Método IExtendedRetryPolicy.Evaluate](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-195">For more details, see [IExtendedRetryPolicy.Evaluate Method](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span></span>

<span data-ttu-id="30594-196">A tabela a seguir mostra as configurações padrão para as políticas de repetição internas.</span><span class="sxs-lookup"><span data-stu-id="30594-196">The following table shows the default settings for the built-in retry policies.</span></span>

| <span data-ttu-id="30594-197">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="30594-197">**Context**</span></span> | <span data-ttu-id="30594-198">**Configuração**</span><span class="sxs-lookup"><span data-stu-id="30594-198">**Setting**</span></span> | <span data-ttu-id="30594-199">**Valor padrão**</span><span class="sxs-lookup"><span data-stu-id="30594-199">**Default value**</span></span> | <span data-ttu-id="30594-200">**Significado**</span><span class="sxs-lookup"><span data-stu-id="30594-200">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="30594-201">QueueRequestOptions</span><span class="sxs-lookup"><span data-stu-id="30594-201">Table / Blob / File</span></span><br /><span data-ttu-id="30594-202">QueueRequestOptions</span><span class="sxs-lookup"><span data-stu-id="30594-202">QueueRequestOptions</span></span> |<span data-ttu-id="30594-203">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="30594-203">MaximumExecutionTime</span></span><br /><br /><span data-ttu-id="30594-204">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="30594-204">ServerTimeout</span></span><br /><br /><br /><br /><br /><span data-ttu-id="30594-205">LocationMode</span><span class="sxs-lookup"><span data-stu-id="30594-205">LocationMode</span></span><br /><br /><br /><br /><br /><br /><br /><span data-ttu-id="30594-206">RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="30594-206">RetryPolicy</span></span> |<span data-ttu-id="30594-207">120 segundos</span><span class="sxs-lookup"><span data-stu-id="30594-207">120 seconds</span></span><br /><br /><span data-ttu-id="30594-208">Nenhum</span><span class="sxs-lookup"><span data-stu-id="30594-208">None</span></span><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><span data-ttu-id="30594-209">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="30594-209">ExponentialPolicy</span></span> |<span data-ttu-id="30594-210">Tempo máximo de execução para a solicitação, incluindo todas as tentativas de repetição potenciais.</span><span class="sxs-lookup"><span data-stu-id="30594-210">Maximum execution time for the request, including all potential retry attempts.</span></span><br /><span data-ttu-id="30594-211">O intervalo do tempo limite do servidor para a solicitação (o valor é arredondado para segundos).</span><span class="sxs-lookup"><span data-stu-id="30594-211">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="30594-212">Se não for especificado, será usado o valor padrão para todas as solicitações ao servidor.</span><span class="sxs-lookup"><span data-stu-id="30594-212">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="30594-213">Normalmente, a melhor opção é omitir essa configuração para que o padrão de servidor seja usado.</span><span class="sxs-lookup"><span data-stu-id="30594-213">Usually, the best option is to omit this setting so that the server default is used.</span></span><br /><span data-ttu-id="30594-214">Se a conta de armazenamento for criada com a opção de replicação RA-GRS (armazenamento com redundância geográfica com acesso de leitura), você poderá usar o modo de local para indicar qual local deve receber a solicitação.</span><span class="sxs-lookup"><span data-stu-id="30594-214">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="30594-215">Por exemplo, se **PrimaryThenSecondary** for especificado, as solicitações sempre serão enviadas para o local primário primeiro.</span><span class="sxs-lookup"><span data-stu-id="30594-215">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="30594-216">Se uma solicitação falhar, ela será enviada para o local secundário.</span><span class="sxs-lookup"><span data-stu-id="30594-216">If a request fails, it is sent to the secondary location.</span></span><br /><span data-ttu-id="30594-217">Veja abaixo os detalhes de cada opção.</span><span class="sxs-lookup"><span data-stu-id="30594-217">See below for details of each option.</span></span> |
| <span data-ttu-id="30594-218">Política exponencial</span><span class="sxs-lookup"><span data-stu-id="30594-218">Exponential policy</span></span> |<span data-ttu-id="30594-219">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="30594-219">maxAttempt</span></span><br /><span data-ttu-id="30594-220">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="30594-220">deltaBackoff</span></span><br /><br /><br /><span data-ttu-id="30594-221">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="30594-221">MinBackoff</span></span><br /><br /><span data-ttu-id="30594-222">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="30594-222">MaxBackoff</span></span> |<span data-ttu-id="30594-223">3</span><span class="sxs-lookup"><span data-stu-id="30594-223">3</span></span><br /><span data-ttu-id="30594-224">4 segundos</span><span class="sxs-lookup"><span data-stu-id="30594-224">4 seconds</span></span><br /><br /><br /><span data-ttu-id="30594-225">3 Segundos</span><span class="sxs-lookup"><span data-stu-id="30594-225">3 seconds</span></span><br /><br /><span data-ttu-id="30594-226">120 segundos</span><span class="sxs-lookup"><span data-stu-id="30594-226">120 seconds</span></span> |<span data-ttu-id="30594-227">Número de tentativas de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-227">Number of retry attempts.</span></span><br /><span data-ttu-id="30594-228">Intervalo de retirada entre repetições.</span><span class="sxs-lookup"><span data-stu-id="30594-228">Back-off interval between retries.</span></span> <span data-ttu-id="30594-229">Múltiplos desse período de tempo, incluindo um elemento aleatório, serão usados para tentativas de repetição subsequentes.</span><span class="sxs-lookup"><span data-stu-id="30594-229">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span><br /><span data-ttu-id="30594-230">Adicionado a todos os intervalos de repetição calculados de deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="30594-230">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="30594-231">Esse valor não pode ser alterado.</span><span class="sxs-lookup"><span data-stu-id="30594-231">This value cannot be changed.</span></span><br /><span data-ttu-id="30594-232">MaxBackoff será usado se o intervalo de repetição calculado for maior que MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="30594-232">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="30594-233">Esse valor não pode ser alterado.</span><span class="sxs-lookup"><span data-stu-id="30594-233">This value cannot be changed.</span></span> |
| <span data-ttu-id="30594-234">Política linear</span><span class="sxs-lookup"><span data-stu-id="30594-234">Linear policy</span></span> |<span data-ttu-id="30594-235">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="30594-235">maxAttempt</span></span><br /><span data-ttu-id="30594-236">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="30594-236">deltaBackoff</span></span> |<span data-ttu-id="30594-237">3</span><span class="sxs-lookup"><span data-stu-id="30594-237">3</span></span><br /><span data-ttu-id="30594-238">30 segundos</span><span class="sxs-lookup"><span data-stu-id="30594-238">30 seconds</span></span> |<span data-ttu-id="30594-239">Número de tentativas de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-239">Number of retry attempts.</span></span><br /><span data-ttu-id="30594-240">Intervalo de retirada entre repetições.</span><span class="sxs-lookup"><span data-stu-id="30594-240">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="30594-241">Diretriz de uso de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-241">Retry usage guidance</span></span>
<span data-ttu-id="30594-242">Considere as seguintes diretrizes ao acessar serviços de armazenamento do Azure usando a API do cliente de armazenamento:</span><span class="sxs-lookup"><span data-stu-id="30594-242">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="30594-243">Use as políticas de repetição internas do namespace Microsoft.WindowsAzure.Storage.RetryPolicies onde elas forem apropriadas para suas necessidades.</span><span class="sxs-lookup"><span data-stu-id="30594-243">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="30594-244">Na maioria dos casos, essas políticas serão suficientes.</span><span class="sxs-lookup"><span data-stu-id="30594-244">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="30594-245">Use a política **ExponentialRetry** em operações em lote, tarefas em segundo plano ou cenários não interativos.</span><span class="sxs-lookup"><span data-stu-id="30594-245">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="30594-246">Nesses cenários, geralmente você pode permitir mais tempo para que o serviço se recupere — com uma chance consequentemente aumentada da operação ser bem-sucedida.</span><span class="sxs-lookup"><span data-stu-id="30594-246">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="30594-247">Considere especificar a propriedade **MaximumExecutionTime** do parâmetro **RequestOptions** para limitar o tempo de execução total, mas leve em consideração o tipo e o tamanho da operação ao escolher um valor de tempo limite.</span><span class="sxs-lookup"><span data-stu-id="30594-247">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="30594-248">Se você precisar implementar uma repetição personalizada, evite criar wrappers em torno das classes do cliente de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="30594-248">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="30594-249">Em vez disso, use os recursos para estender as políticas existentes por meio da interface **IExtendedRetryPolicy** .</span><span class="sxs-lookup"><span data-stu-id="30594-249">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="30594-250">Se você estiver usando RA-GRS (armazenamento com redundância geográfica com acesso de leitura), será possível usar o **LocationMode** para especificar que as tentativas de repetição acessarão a cópia secundária somente leitura do repositório caso o acesso primário falhe.</span><span class="sxs-lookup"><span data-stu-id="30594-250">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="30594-251">No entanto, ao usar essa opção, você deve garantir que seu aplicativo possa trabalhar perfeitamente com dados que talvez estejam obsoletos, se a replicação do repositório primário ainda não tiver sido concluída.</span><span class="sxs-lookup"><span data-stu-id="30594-251">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="30594-252">Considere começar com as seguintes configurações para operações de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-252">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="30594-253">Essas são configurações de finalidade geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.</span><span class="sxs-lookup"><span data-stu-id="30594-253">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="30594-254">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="30594-254">**Context**</span></span> | <span data-ttu-id="30594-255">**Exemplo de latênciaE2E<br />máxima de destino**</span><span class="sxs-lookup"><span data-stu-id="30594-255">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="30594-256">**Política de repetição**</span><span class="sxs-lookup"><span data-stu-id="30594-256">**Retry policy**</span></span> | <span data-ttu-id="30594-257">**Configurações**</span><span class="sxs-lookup"><span data-stu-id="30594-257">**Settings**</span></span> | <span data-ttu-id="30594-258">**Valores**</span><span class="sxs-lookup"><span data-stu-id="30594-258">**Values**</span></span> | <span data-ttu-id="30594-259">**Como funciona**</span><span class="sxs-lookup"><span data-stu-id="30594-259">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="30594-260">Interativo, interface de usuário</span><span class="sxs-lookup"><span data-stu-id="30594-260">Interactive, UI,</span></span><br /><span data-ttu-id="30594-261">ou primeiro plano</span><span class="sxs-lookup"><span data-stu-id="30594-261">or foreground</span></span> |<span data-ttu-id="30594-262">2 segundos</span><span class="sxs-lookup"><span data-stu-id="30594-262">2 seconds</span></span> |<span data-ttu-id="30594-263">Linear</span><span class="sxs-lookup"><span data-stu-id="30594-263">Linear</span></span> |<span data-ttu-id="30594-264">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="30594-264">maxAttempt</span></span><br /><span data-ttu-id="30594-265">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="30594-265">deltaBackoff</span></span> |<span data-ttu-id="30594-266">3</span><span class="sxs-lookup"><span data-stu-id="30594-266">3</span></span><br /><span data-ttu-id="30594-267">500 ms</span><span class="sxs-lookup"><span data-stu-id="30594-267">500 ms</span></span> |<span data-ttu-id="30594-268">1ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="30594-268">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="30594-269">2ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="30594-269">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="30594-270">3ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="30594-270">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="30594-271">Segundo plano</span><span class="sxs-lookup"><span data-stu-id="30594-271">Background</span></span><br /><span data-ttu-id="30594-272">ou lote</span><span class="sxs-lookup"><span data-stu-id="30594-272">or batch</span></span> |<span data-ttu-id="30594-273">30 segundos</span><span class="sxs-lookup"><span data-stu-id="30594-273">30 seconds</span></span> |<span data-ttu-id="30594-274">Exponencial</span><span class="sxs-lookup"><span data-stu-id="30594-274">Exponential</span></span> |<span data-ttu-id="30594-275">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="30594-275">maxAttempt</span></span><br /><span data-ttu-id="30594-276">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="30594-276">deltaBackoff</span></span> |<span data-ttu-id="30594-277">5</span><span class="sxs-lookup"><span data-stu-id="30594-277">5</span></span><br /><span data-ttu-id="30594-278">4 segundos</span><span class="sxs-lookup"><span data-stu-id="30594-278">4 seconds</span></span> |<span data-ttu-id="30594-279">1ª tentativa — intervalo de ~3 s</span><span class="sxs-lookup"><span data-stu-id="30594-279">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="30594-280">2ª tentativa — intervalo de ~7 s</span><span class="sxs-lookup"><span data-stu-id="30594-280">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="30594-281">3ª tentativa — intervalo de ~15 s</span><span class="sxs-lookup"><span data-stu-id="30594-281">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="30594-282">Telemetria</span><span class="sxs-lookup"><span data-stu-id="30594-282">Telemetry</span></span>
<span data-ttu-id="30594-283">As tentativas de repetição são registradas em um **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="30594-283">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="30594-284">Você deve configurar um **TraceListener** para capturar os eventos e gravá-los em um log de destino apropriado.</span><span class="sxs-lookup"><span data-stu-id="30594-284">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="30594-285">É possível usar o **TextWriterTraceListener** ou **XmlWriterTraceListener** para gravar os dados em um arquivo de log, o **EventLogTraceListener** para gravar no Log de Eventos do Windows ou o **EventProviderTraceListener** para gravar dados de rastreamento no subsistema ETW.</span><span class="sxs-lookup"><span data-stu-id="30594-285">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="30594-286">Você também pode configurar a liberação automática do buffer e o detalhamento dos eventos que serão registrados (por exemplo, Erro, Aviso, Informativo e Detalhado).</span><span class="sxs-lookup"><span data-stu-id="30594-286">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="30594-287">Para saber mais, consulte [Registro em log no lado do cliente com a biblioteca do cliente de armazenamento para .NET](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-287">For more information, see [Client-side Logging with the .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span></span>

<span data-ttu-id="30594-288">As operações podem receber uma instância **OperationContext**, que expõe um evento **Retrying** que pode ser usado para anexar a lógica personalizada de telemetria.</span><span class="sxs-lookup"><span data-stu-id="30594-288">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="30594-289">Para saber mais, consulte [Evento OperationContext.Retrying](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-289">For more information, see [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span></span>

### <a name="examples"></a><span data-ttu-id="30594-290">Exemplos</span><span class="sxs-lookup"><span data-stu-id="30594-290">Examples</span></span>
<span data-ttu-id="30594-291">O exemplo de código a seguir mostra como criar duas instâncias **TableRequestOptions** com diferentes configurações de repetição; uma para solicitações interativas e outra para solicitações em segundo plano.</span><span class="sxs-lookup"><span data-stu-id="30594-291">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="30594-292">O exemplo define essas duas políticas de repetição no cliente para que elas sejam aplicadas a todas as solicitações, assim como define a estratégia interativa em uma solicitação específica de modo que ela substitua as configurações padrão aplicadas ao cliente.</span><span class="sxs-lookup"><span data-stu-id="30594-292">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

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
                // Maximum execution time based on the business use case. Maximum value up to 10 seconds.
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

### <a name="more-information"></a><span data-ttu-id="30594-293">Mais informações</span><span class="sxs-lookup"><span data-stu-id="30594-293">More information</span></span>
* [<span data-ttu-id="30594-294">Recomendações de política de repetição da biblioteca do cliente de armazenamento do Azure</span><span class="sxs-lookup"><span data-stu-id="30594-294">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="30594-295">Biblioteca do cliente de armazenamento 2.0 – implementando políticas de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-295">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="sql-database-using-entity-framework-6-retry-guidelines"></a><span data-ttu-id="30594-296">Diretrizes de repetição para Banco de Dados SQL usando o Entity Framework 6</span><span class="sxs-lookup"><span data-stu-id="30594-296">SQL Database using Entity Framework 6 retry guidelines</span></span>
<span data-ttu-id="30594-297">O Banco de Dados SQL é um banco de dados SQL disponível em vários tamanhos e como um serviço padrão (compartilhado) e premium (não compartilhado).</span><span class="sxs-lookup"><span data-stu-id="30594-297">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="30594-298">O Entity Framework é um mapeador relacional de objeto que permite aos desenvolvedores do .NET trabalhar com dados relacionais usando objetos específicos de domínio.</span><span class="sxs-lookup"><span data-stu-id="30594-298">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="30594-299">Com ele, não há a necessidade da maioria dos códigos de acesso a dados que os desenvolvedores geralmente precisam para escrever.</span><span class="sxs-lookup"><span data-stu-id="30594-299">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="30594-300">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-300">Retry mechanism</span></span>
<span data-ttu-id="30594-301">O suporte à repetição é fornecido durante o acesso ao Banco de Dados SQL usando o Entity Framework 6.0 e versões posteriores por meio de um mecanismo chamado [Lógica de Repetição/Resiliência de Conexão](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-301">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span> <span data-ttu-id="30594-302">Os principais recursos do mecanismo de repetição são:</span><span class="sxs-lookup"><span data-stu-id="30594-302">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="30594-303">A abstração primária é a interface **IDbExecutionStrategy** .</span><span class="sxs-lookup"><span data-stu-id="30594-303">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="30594-304">Essa interface:</span><span class="sxs-lookup"><span data-stu-id="30594-304">This interface:</span></span>
  * <span data-ttu-id="30594-305">Define métodos **Execute*** síncronos e assíncronos.</span><span class="sxs-lookup"><span data-stu-id="30594-305">Defines synchronous and asynchronous **Execute*** methods.</span></span>
  * <span data-ttu-id="30594-306">Define classes que podem ser usadas diretamente ou podem ser configuradas em um contexto de banco de dados como uma estratégia padrão, mapeadas para nome do provedor ou para um nome de provedor e nome de servidor.</span><span class="sxs-lookup"><span data-stu-id="30594-306">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="30594-307">Quando configuradas em um contexto, as repetições ocorrem no nível de operações de banco de dados individuais, das quais pode haver várias de uma determinada operação de contexto.</span><span class="sxs-lookup"><span data-stu-id="30594-307">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="30594-308">Define quando repetir uma conexão com falha, e como.</span><span class="sxs-lookup"><span data-stu-id="30594-308">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="30594-309">Incluem várias implementações internas da interface **IDbExecutionStrategy** :</span><span class="sxs-lookup"><span data-stu-id="30594-309">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="30594-310">Padrão: sem repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-310">Default - no retrying.</span></span>
  * <span data-ttu-id="30594-311">Padrão para Banco de Dados SQL(automático): sem repetição, mas inspeciona exceções e as encapsula com sugestão para usar a estratégia Banco de Dados SQL.</span><span class="sxs-lookup"><span data-stu-id="30594-311">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="30594-312">Padrão para Banco de Dados SQL: exponencial (herdado da classe base) mais lógica de detecção do Banco de Dados SQL.</span><span class="sxs-lookup"><span data-stu-id="30594-312">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="30594-313">Implementam uma estratégia de retirada exponencial que inclui aleatoriedade.</span><span class="sxs-lookup"><span data-stu-id="30594-313">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="30594-314">As características das classes de repetição internas são: com estado e sem thread-safe.</span><span class="sxs-lookup"><span data-stu-id="30594-314">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="30594-315">No entanto, elas podem ser reutilizadas depois que a operação atual for concluída.</span><span class="sxs-lookup"><span data-stu-id="30594-315">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="30594-316">Se a contagem de repetição especificada for excedida, os resultados serão encapsulados em uma nova exceção.</span><span class="sxs-lookup"><span data-stu-id="30594-316">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="30594-317">A exceção atual não é exibida.</span><span class="sxs-lookup"><span data-stu-id="30594-317">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="30594-318">Configuração de política</span><span class="sxs-lookup"><span data-stu-id="30594-318">Policy configuration</span></span>
<span data-ttu-id="30594-319">O suporte à repetição é fornecido durante o acesso ao Banco de Dados SQL usando o Entity Framework 6.0 e versões superiores.</span><span class="sxs-lookup"><span data-stu-id="30594-319">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="30594-320">As políticas de repetição são configuradas de modo programático.</span><span class="sxs-lookup"><span data-stu-id="30594-320">Retry policies are configured programmatically.</span></span> <span data-ttu-id="30594-321">A configuração não pode ser alterada por operação.</span><span class="sxs-lookup"><span data-stu-id="30594-321">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="30594-322">Ao configurar uma estratégia no contexto como o padrão, você especifica uma função que cria uma nova estratégia sob demanda.</span><span class="sxs-lookup"><span data-stu-id="30594-322">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="30594-323">O código a seguir mostra como você pode criar uma classe de configuração de repetição que estende a classe base **DbConfiguration** .</span><span class="sxs-lookup"><span data-stu-id="30594-323">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

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

<span data-ttu-id="30594-324">Depois isso pode ser especificado como a estratégia de repetição padrão para todas as operações usando o método **SetConfiguration** da instância **DbConfiguration** quando o aplicativo é iniciado.</span><span class="sxs-lookup"><span data-stu-id="30594-324">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="30594-325">Por padrão, o EF vai descobrir e usar automaticamente a classe de configuração.</span><span class="sxs-lookup"><span data-stu-id="30594-325">By default, EF will automatically discover and use the configuration class.</span></span>

    DbConfiguration.SetConfiguration(new BloggingContextConfiguration());

<span data-ttu-id="30594-326">Você pode especificar a classe de configuração de repetição para um contexto anotando a classe de contexto com um atributo **DbConfigurationType** .</span><span class="sxs-lookup"><span data-stu-id="30594-326">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="30594-327">No entanto, se você tiver apenas uma classe de configuração, o EF a usará sem a necessidade de anotar o contexto.</span><span class="sxs-lookup"><span data-stu-id="30594-327">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

    [DbConfigurationType(typeof(BloggingContextConfiguration))]
    public class BloggingContext : DbContext
    { ...

<span data-ttu-id="30594-328">Se precisar usar estratégias de repetição diferentes para operações específicas ou desabilitar repetições para operações específicas, você poderá criar uma classe de configuração que permita suspender ou trocar estratégias, definindo um sinalizador no **CallContext**.</span><span class="sxs-lookup"><span data-stu-id="30594-328">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="30594-329">A classe de configuração pode usar esse sinalizador para alternar estratégias ou desabilitar a estratégia fornecida por você e usar uma estratégia padrão.</span><span class="sxs-lookup"><span data-stu-id="30594-329">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="30594-330">Para saber mais, consulte [Suspender estratégia de execução](http://msdn.microsoft.com/dn307226#transactions_workarounds) na página Limitações com estratégias de execução de repetição (EF6 em diante).</span><span class="sxs-lookup"><span data-stu-id="30594-330">For more information, see [Suspend Execution Strategy](http://msdn.microsoft.com/dn307226#transactions_workarounds) in the page Limitations with Retrying Execution Strategies (EF6 onwards).</span></span>

<span data-ttu-id="30594-331">Outra técnica para usar estratégias de repetição específicas para operações individuais é criar uma instância da classe de estratégia necessária e fornecer as configurações desejadas por meio de parâmetros.</span><span class="sxs-lookup"><span data-stu-id="30594-331">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="30594-332">Você então chama o método **ExecuteAsync** .</span><span class="sxs-lookup"><span data-stu-id="30594-332">You then invoke its **ExecuteAsync** method.</span></span>

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

<span data-ttu-id="30594-333">A maneira mais simples de usar uma classe **DbConfiguration** é colocá-la no mesmo assembly da classe **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="30594-333">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="30594-334">No entanto, isso não é apropriado quando o mesmo contexto é necessário em cenários diferentes, como diferentes estratégias de repetição interativas e em segundo plano.</span><span class="sxs-lookup"><span data-stu-id="30594-334">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="30594-335">Se os diferentes contextos forem executados em AppDomains separados, você poderá usar o suporte interno para especificar classes de configuração no arquivo de configuração ou defini-las explicitamente usando código.</span><span class="sxs-lookup"><span data-stu-id="30594-335">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="30594-336">Se os diferentes contextos devem ser executados no mesmo AppDomain, será necessária uma solução personalizada.</span><span class="sxs-lookup"><span data-stu-id="30594-336">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="30594-337">Para saber mais, consulte [Configuração baseada em código (EF6 em diante)](http://msdn.microsoft.com/data/jj680699.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-337">For more information, see [Code-Based Configuration (EF6 onwards)](http://msdn.microsoft.com/data/jj680699.aspx).</span></span>

<span data-ttu-id="30594-338">A tabela a seguir mostra as configurações padrão para a política de repetição interna ao usar o EF6.</span><span class="sxs-lookup"><span data-stu-id="30594-338">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

![Tente a tabela de orientação novamente](./images/retry-service-specific/RetryServiceSpecificGuidanceTable4.png)

### <a name="retry-usage-guidance"></a><span data-ttu-id="30594-340">Diretriz de uso de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-340">Retry usage guidance</span></span>
<span data-ttu-id="30594-341">Considere as seguintes diretrizes ao acessar o Banco de Dados SQL usando o EF6:</span><span class="sxs-lookup"><span data-stu-id="30594-341">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="30594-342">Escolha a opção de serviço apropriada (compartilhada ou premium).</span><span class="sxs-lookup"><span data-stu-id="30594-342">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="30594-343">Uma instância compartilhada pode sofrer atrasos e limitação de conexão mais longos que o normal devido ao uso por outros locatários do servidor compartilhado.</span><span class="sxs-lookup"><span data-stu-id="30594-343">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="30594-344">Se forem necessárias operações confiáveis de baixa latência e desempenho previsível, considere a escolha da opção premium.</span><span class="sxs-lookup"><span data-stu-id="30594-344">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="30594-345">Uma estratégia de intervalo fixo não é recomendada para ser usada com o Banco de Dados SQL do Azure.</span><span class="sxs-lookup"><span data-stu-id="30594-345">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="30594-346">Em vez disso, use uma estratégia de retirada exponencial, pois o serviço pode estar sobrecarregado e intervalos mais longos permitem mais tempo para a recuperação.</span><span class="sxs-lookup"><span data-stu-id="30594-346">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="30594-347">Escolha um valor adequado para os tempos limite de conexão e comando ao definir conexões.</span><span class="sxs-lookup"><span data-stu-id="30594-347">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="30594-348">Baseie o tempo limite no seu design de lógica de negócios e por meio de teste.</span><span class="sxs-lookup"><span data-stu-id="30594-348">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="30594-349">Talvez seja necessário modificar esse valor ao longo do tempo, uma vez que os volumes de dados ou os processos de negócios mudam.</span><span class="sxs-lookup"><span data-stu-id="30594-349">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="30594-350">Um tempo limite muito curto pode resultar em falhas prematuras de conexões quando o banco de dados estiver ocupado.</span><span class="sxs-lookup"><span data-stu-id="30594-350">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="30594-351">Um tempo limite muito longo pode impedir que a lógica de repetição funcione corretamente, aguardando tempo demais para detectar uma conexão com falha.</span><span class="sxs-lookup"><span data-stu-id="30594-351">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="30594-352">O valor do tempo limite é um componente da latência de ponta a ponta, embora você não possa determinar facilmente quantos comandos serão executados ao salvar o contexto.</span><span class="sxs-lookup"><span data-stu-id="30594-352">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="30594-353">Você pode alterar o tempo limite padrão definindo a propriedade **CommandTimeout** da instância **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="30594-353">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="30594-354">O Entity Framework oferece suporte às configurações de repetição definidas em arquivos de configuração.</span><span class="sxs-lookup"><span data-stu-id="30594-354">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="30594-355">No entanto, para máxima flexibilidade no Azure, você deve considerar a criação da configuração de modo programático dentro do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="30594-355">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="30594-356">Os parâmetros específicos das políticas de repetição, como o número de repetições e os intervalos da repetição, podem ser armazenados no arquivo de configuração de serviço e usados no tempo de execução para criar as políticas apropriadas.</span><span class="sxs-lookup"><span data-stu-id="30594-356">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="30594-357">Isso permite que as configurações sejam alteradas sem precisar reiniciar o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="30594-357">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="30594-358">Considere começar com as seguintes configurações para operações de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-358">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="30594-359">Você não pode especificar o intervalo entre tentativas de repetição (ele é fixo e gerado como uma sequência exponencial).</span><span class="sxs-lookup"><span data-stu-id="30594-359">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="30594-360">Você pode especificar apenas os valores máximo, conforme mostrado aqui, a menos que você crie uma estratégia de repetição personalizada.</span><span class="sxs-lookup"><span data-stu-id="30594-360">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="30594-361">Essas são configurações de finalidade geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.</span><span class="sxs-lookup"><span data-stu-id="30594-361">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="30594-362">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="30594-362">**Context**</span></span> | <span data-ttu-id="30594-363">**Exemplo de latênciaE2E<br />máxima de destino**</span><span class="sxs-lookup"><span data-stu-id="30594-363">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="30594-364">**Política de repetição**</span><span class="sxs-lookup"><span data-stu-id="30594-364">**Retry policy**</span></span> | <span data-ttu-id="30594-365">**Configurações**</span><span class="sxs-lookup"><span data-stu-id="30594-365">**Settings**</span></span> | <span data-ttu-id="30594-366">**Valores**</span><span class="sxs-lookup"><span data-stu-id="30594-366">**Values**</span></span> | <span data-ttu-id="30594-367">**Como funciona**</span><span class="sxs-lookup"><span data-stu-id="30594-367">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="30594-368">Interativo, interface de usuário</span><span class="sxs-lookup"><span data-stu-id="30594-368">Interactive, UI,</span></span><br /><span data-ttu-id="30594-369">ou primeiro plano</span><span class="sxs-lookup"><span data-stu-id="30594-369">or foreground</span></span> |<span data-ttu-id="30594-370">2 segundos</span><span class="sxs-lookup"><span data-stu-id="30594-370">2 seconds</span></span> |<span data-ttu-id="30594-371">Exponencial</span><span class="sxs-lookup"><span data-stu-id="30594-371">Exponential</span></span> |<span data-ttu-id="30594-372">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="30594-372">MaxRetryCount</span></span><br /><span data-ttu-id="30594-373">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="30594-373">MaxDelay</span></span> |<span data-ttu-id="30594-374">3</span><span class="sxs-lookup"><span data-stu-id="30594-374">3</span></span><br /><span data-ttu-id="30594-375">750 ms</span><span class="sxs-lookup"><span data-stu-id="30594-375">750 ms</span></span> |<span data-ttu-id="30594-376">1ª tentativa — intervalo de 0 s</span><span class="sxs-lookup"><span data-stu-id="30594-376">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="30594-377">2ª tentativa — intervalo de 750 ms</span><span class="sxs-lookup"><span data-stu-id="30594-377">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="30594-378">3ª tentativa — intervalo de 750 ms</span><span class="sxs-lookup"><span data-stu-id="30594-378">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="30594-379">Segundo plano</span><span class="sxs-lookup"><span data-stu-id="30594-379">Background</span></span><br /> <span data-ttu-id="30594-380">ou lote</span><span class="sxs-lookup"><span data-stu-id="30594-380">or batch</span></span> |<span data-ttu-id="30594-381">30 segundos</span><span class="sxs-lookup"><span data-stu-id="30594-381">30 seconds</span></span> |<span data-ttu-id="30594-382">Exponencial</span><span class="sxs-lookup"><span data-stu-id="30594-382">Exponential</span></span> |<span data-ttu-id="30594-383">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="30594-383">MaxRetryCount</span></span><br /><span data-ttu-id="30594-384">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="30594-384">MaxDelay</span></span> |<span data-ttu-id="30594-385">5</span><span class="sxs-lookup"><span data-stu-id="30594-385">5</span></span><br /><span data-ttu-id="30594-386">12 segundos</span><span class="sxs-lookup"><span data-stu-id="30594-386">12 seconds</span></span> |<span data-ttu-id="30594-387">1ª tentativa — intervalo de 0 s</span><span class="sxs-lookup"><span data-stu-id="30594-387">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="30594-388">2ª tentativa — intervalo de ~1 s</span><span class="sxs-lookup"><span data-stu-id="30594-388">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="30594-389">3ª tentativa — intervalo de ~3 s</span><span class="sxs-lookup"><span data-stu-id="30594-389">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="30594-390">4ª tentativa — intervalo de ~7 s</span><span class="sxs-lookup"><span data-stu-id="30594-390">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="30594-391">5ª tentativa — intervalo de 12 s</span><span class="sxs-lookup"><span data-stu-id="30594-391">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="30594-392">A latência de ponta a ponta visa o tempo limite padrão de conexões com o serviço.</span><span class="sxs-lookup"><span data-stu-id="30594-392">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="30594-393">Se você especificar tempos limite de conexão mais longos, a latência de ponta a ponta será estendida por esse tempo adicional para cada tentativa de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-393">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="30594-394">Exemplos</span><span class="sxs-lookup"><span data-stu-id="30594-394">Examples</span></span>
<span data-ttu-id="30594-395">O exemplo de código a seguir define uma solução de acesso a dados simples que usa o Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="30594-395">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="30594-396">Ele estabelece uma estratégia de repetição específica definindo uma instância de uma classe chamada **BlogConfiguration** que estende **DbConfiguration**.</span><span class="sxs-lookup"><span data-stu-id="30594-396">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

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

<span data-ttu-id="30594-397">Mais exemplos de como usar o mecanismo de repetição do Entity Framework podem ser encontrados em [Lógica de Repetição/Resiliência de Conexão](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-397">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span>

### <a name="more-information"></a><span data-ttu-id="30594-398">Mais informações</span><span class="sxs-lookup"><span data-stu-id="30594-398">More information</span></span>
* [<span data-ttu-id="30594-399">Guia de desempenho e elasticidade do Banco de Dados SQL do Azure</span><span class="sxs-lookup"><span data-stu-id="30594-399">Azure SQL Database Performance and Elasticity Guide</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core-retry-guidelines"></a><span data-ttu-id="30594-400">As diretrizes de repetição para Banco de Dados SQL usando o Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="30594-400">SQL Database using Entity Framework Core retry guidelines</span></span>
<span data-ttu-id="30594-401">[O Entity Framework Core](/ef/core/) é um mapeador relacional de objeto que permite aos desenvolvedores do .NET Core trabalhar com dados usando objetos específicos de domínio.</span><span class="sxs-lookup"><span data-stu-id="30594-401">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="30594-402">Com ele, não há a necessidade da maioria dos códigos de acesso a dados que os desenvolvedores geralmente precisam para escrever.</span><span class="sxs-lookup"><span data-stu-id="30594-402">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="30594-403">Esta versão do Entity Framework foi criada a partir do zero e não herda todos os recursos do EF6.x automaticamente.</span><span class="sxs-lookup"><span data-stu-id="30594-403">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="30594-404">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-404">Retry mechanism</span></span>
<span data-ttu-id="30594-405">O suporte à repetição é fornecido durante o acesso ao Banco de Dados SQL usando o Entity Framework Core por meio de um mecanismo chamado [Resiliência de Conexão](/ef/core/miscellaneous/connection-resiliency).</span><span class="sxs-lookup"><span data-stu-id="30594-405">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="30594-406">A resiliência de conexão foi introduzida no EF Core 1.1.0.</span><span class="sxs-lookup"><span data-stu-id="30594-406">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="30594-407">A abstração primária é a interface `IExecutionStrategy`.</span><span class="sxs-lookup"><span data-stu-id="30594-407">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="30594-408">A estratégia de execução para o SQL Server, incluindo o SQL Azure, está ciente dos tipos de exceção que podem ser repetidos e tem padrões específicos para o número máximo de tentativas, atrasos entre as repetições e assim por diante.</span><span class="sxs-lookup"><span data-stu-id="30594-408">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="30594-409">Exemplos</span><span class="sxs-lookup"><span data-stu-id="30594-409">Examples</span></span>

<span data-ttu-id="30594-410">O código a seguir permite novas tentativas automáticas ao configurar o objeto DbContext, que representa uma sessão com o banco de dados.</span><span class="sxs-lookup"><span data-stu-id="30594-410">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="30594-411">O código a seguir mostra como executar uma transação com novas tentativas automáticas, usando uma estratégia de execução.</span><span class="sxs-lookup"><span data-stu-id="30594-411">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="30594-412">A transação é definida em um representante.</span><span class="sxs-lookup"><span data-stu-id="30594-412">The transaction is defined in a delegate.</span></span> <span data-ttu-id="30594-413">Se ocorrer uma falha transitória, a estratégia de execução invocará o representante novamente.</span><span class="sxs-lookup"><span data-stu-id="30594-413">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

### <a name="more-information"></a><span data-ttu-id="30594-414">Mais informações</span><span class="sxs-lookup"><span data-stu-id="30594-414">More information</span></span>
* [<span data-ttu-id="30594-415">Resiliência de conexão</span><span class="sxs-lookup"><span data-stu-id="30594-415">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="30594-416">Pontos de dados - Core EF 1.1</span><span class="sxs-lookup"><span data-stu-id="30594-416">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/en-us/magazine/mt745093.aspx)

## <a name="sql-database-using-adonet-retry-guidelines"></a><span data-ttu-id="30594-417">Diretrizes de repetição para Banco de Dados SQL usando o ADO.NET</span><span class="sxs-lookup"><span data-stu-id="30594-417">SQL Database using ADO.NET retry guidelines</span></span>
<span data-ttu-id="30594-418">O Banco de Dados SQL é um banco de dados SQL disponível em vários tamanhos e como um serviço padrão (compartilhado) e premium (não compartilhado).</span><span class="sxs-lookup"><span data-stu-id="30594-418">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="30594-419">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-419">Retry mechanism</span></span>
<span data-ttu-id="30594-420">O Banco de Dados SQL não tem suporte interno para repetições quando acessado usando o ADO.NET.</span><span class="sxs-lookup"><span data-stu-id="30594-420">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="30594-421">No entanto, os códigos de retorno das solicitações podem ser usados para determinar por que uma solicitação falhou.</span><span class="sxs-lookup"><span data-stu-id="30594-421">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="30594-422">Para obter mais informações sobre a limitação do banco de dados SQL, confira [Limites de recursos do banco de dados SQL do Azure](/azure/sql-database/sql-database-resource-limits).</span><span class="sxs-lookup"><span data-stu-id="30594-422">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="30594-423">Para obter uma lista dos códigos de erro relevantes, confira [códigos de erro SQL para aplicativos de cliente do banco de dados SQL](/azure/sql-database/sql-database-develop-error-messages).</span><span class="sxs-lookup"><span data-stu-id="30594-423">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="30594-424">Você pode usar a biblioteca Polly para implementar tentativas para o banco de dados SQL.</span><span class="sxs-lookup"><span data-stu-id="30594-424">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="30594-425">Confira [Falha transitória manipulando com a Polly](#transient-fault-handling-with-polly).</span><span class="sxs-lookup"><span data-stu-id="30594-425">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="30594-426">Diretriz de uso de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-426">Retry usage guidance</span></span>
<span data-ttu-id="30594-427">Considere as seguintes diretrizes ao acessar o Banco de Dados SQL usando o ADO.NET:</span><span class="sxs-lookup"><span data-stu-id="30594-427">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="30594-428">Escolha a opção de serviço apropriada (compartilhada ou premium).</span><span class="sxs-lookup"><span data-stu-id="30594-428">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="30594-429">Uma instância compartilhada pode sofrer atrasos e limitação de conexão mais longos que o normal devido ao uso por outros locatários do servidor compartilhado.</span><span class="sxs-lookup"><span data-stu-id="30594-429">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="30594-430">Se forem necessárias mais operações confiáveis de baixa latência e desempenho previsível, considere a escolha da opção premium.</span><span class="sxs-lookup"><span data-stu-id="30594-430">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="30594-431">Garanta a execução de repetições no nível ou escopo apropriado para evitar operações não idempotentes que causem inconsistência nos dados.</span><span class="sxs-lookup"><span data-stu-id="30594-431">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="30594-432">De modo ideal, todas as operações devem ser idempotentes para que possam ser repetidas sem causar inconsistência.</span><span class="sxs-lookup"><span data-stu-id="30594-432">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="30594-433">Quando não for esse o caso, a repetição deverá ser realizada em um nível ou escopo que permita que todas as alterações relacionadas sejam desfeitas, caso uma operação falhe; por exemplo, em um escopo transacional.</span><span class="sxs-lookup"><span data-stu-id="30594-433">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="30594-434">Para saber mais, consulte [Camada de acesso a dados dos conceitos básicos do serviço de nuvem — tratamento de falhas transitórias](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span><span class="sxs-lookup"><span data-stu-id="30594-434">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="30594-435">Uma estratégia de intervalo fixo não é recomendada para uso com o Banco de Dados SQL do Azure, exceto em cenários interativos em que há apenas algumas repetições em intervalos muito curtos.</span><span class="sxs-lookup"><span data-stu-id="30594-435">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="30594-436">Em vez disso, considere usar uma estratégia de retirada exponencial para a maioria dos cenários.</span><span class="sxs-lookup"><span data-stu-id="30594-436">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="30594-437">Escolha um valor adequado para os tempos limite de conexão e comando ao definir conexões.</span><span class="sxs-lookup"><span data-stu-id="30594-437">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="30594-438">Um tempo limite muito curto pode resultar em falhas prematuras de conexões quando o banco de dados estiver ocupado.</span><span class="sxs-lookup"><span data-stu-id="30594-438">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="30594-439">Um tempo limite muito longo pode impedir que a lógica de repetição funcione corretamente, aguardando tempo demais para detectar uma conexão com falha.</span><span class="sxs-lookup"><span data-stu-id="30594-439">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="30594-440">O valor do tempo limite é um componente da latência de ponta a ponta; ele é efetivamente adicionado ao intervalo de repetição especificado na política de repetição para cada tentativa de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-440">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="30594-441">Feche a conexão após um determinado número de repetições, mesmo ao usar uma lógica de repetição de retirada exponencial, e repita a operação em uma nova conexão.</span><span class="sxs-lookup"><span data-stu-id="30594-441">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="30594-442">Repetir a mesma operação várias vezes na mesma conexão pode ser um fator que contribui para problemas de conexão.</span><span class="sxs-lookup"><span data-stu-id="30594-442">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="30594-443">Para obter um exemplo dessa técnica, consulte [Camada de acesso a dados dos conceitos básicos do serviço de nuvem — tratamento de falhas transitórias](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-443">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="30594-444">Quando o pool de conexões está em uso (o padrão), é provável que a mesma conexão seja escolhida no pool, mesmo depois de fechar e reabrir uma conexão.</span><span class="sxs-lookup"><span data-stu-id="30594-444">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="30594-445">Se for esse o caso, uma técnica para resolver isso é chamar o método **ClearPool** da classe **SqlConnection** para marcar a conexão como não reutilizável.</span><span class="sxs-lookup"><span data-stu-id="30594-445">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="30594-446">No entanto, isso deve ser feito somente depois que várias tentativas de conexão tiverem falhado, e somente ao encontrar a classe específica de falhas transitórias, como tempos limite de SQL (código de erro -2), relacionada às conexões com falha.</span><span class="sxs-lookup"><span data-stu-id="30594-446">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="30594-447">Se o código de acesso a dados usar transações iniciadas como instâncias **TransactionScope** , a lógica de repetição deverá reabrir a conexão e iniciar um novo escopo de transação.</span><span class="sxs-lookup"><span data-stu-id="30594-447">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="30594-448">Por esse motivo, o bloco de código com nova tentativa deve englobar o escopo inteiro da transação.</span><span class="sxs-lookup"><span data-stu-id="30594-448">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="30594-449">Considere começar com as seguintes configurações para operações de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-449">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="30594-450">Essas são configurações de finalidade geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.</span><span class="sxs-lookup"><span data-stu-id="30594-450">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="30594-451">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="30594-451">**Context**</span></span> | <span data-ttu-id="30594-452">**Exemplo de latênciaE2E<br />máxima de destino**</span><span class="sxs-lookup"><span data-stu-id="30594-452">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="30594-453">**Estratégia de repetição**</span><span class="sxs-lookup"><span data-stu-id="30594-453">**Retry strategy**</span></span> | <span data-ttu-id="30594-454">**Configurações**</span><span class="sxs-lookup"><span data-stu-id="30594-454">**Settings**</span></span> | <span data-ttu-id="30594-455">**Valores**</span><span class="sxs-lookup"><span data-stu-id="30594-455">**Values**</span></span> | <span data-ttu-id="30594-456">**Como funciona**</span><span class="sxs-lookup"><span data-stu-id="30594-456">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="30594-457">Interativo, interface de usuário</span><span class="sxs-lookup"><span data-stu-id="30594-457">Interactive, UI,</span></span><br /><span data-ttu-id="30594-458">ou primeiro plano</span><span class="sxs-lookup"><span data-stu-id="30594-458">or foreground</span></span> |<span data-ttu-id="30594-459">2 s</span><span class="sxs-lookup"><span data-stu-id="30594-459">2 sec</span></span> |<span data-ttu-id="30594-460">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="30594-460">FixedInterval</span></span> |<span data-ttu-id="30594-461">Contagem de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-461">Retry count</span></span><br /><span data-ttu-id="30594-462">Intervalo de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-462">Retry interval</span></span><br /><span data-ttu-id="30594-463">Primeira repetição rápida</span><span class="sxs-lookup"><span data-stu-id="30594-463">First fast retry</span></span> |<span data-ttu-id="30594-464">3</span><span class="sxs-lookup"><span data-stu-id="30594-464">3</span></span><br /><span data-ttu-id="30594-465">500 ms</span><span class="sxs-lookup"><span data-stu-id="30594-465">500 ms</span></span><br /><span data-ttu-id="30594-466">verdadeiro</span><span class="sxs-lookup"><span data-stu-id="30594-466">true</span></span> |<span data-ttu-id="30594-467">1ª tentativa — intervalo de 0 s</span><span class="sxs-lookup"><span data-stu-id="30594-467">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="30594-468">2ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="30594-468">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="30594-469">3ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="30594-469">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="30594-470">Segundo plano</span><span class="sxs-lookup"><span data-stu-id="30594-470">Background</span></span><br /><span data-ttu-id="30594-471">ou lote</span><span class="sxs-lookup"><span data-stu-id="30594-471">or batch</span></span> |<span data-ttu-id="30594-472">30 s</span><span class="sxs-lookup"><span data-stu-id="30594-472">30 sec</span></span> |<span data-ttu-id="30594-473">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="30594-473">ExponentialBackoff</span></span> |<span data-ttu-id="30594-474">Contagem de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-474">Retry count</span></span><br /><span data-ttu-id="30594-475">Retirada mín.</span><span class="sxs-lookup"><span data-stu-id="30594-475">Min back-off</span></span><br /><span data-ttu-id="30594-476">Retirada máx.</span><span class="sxs-lookup"><span data-stu-id="30594-476">Max back-off</span></span><br /><span data-ttu-id="30594-477">Retirada delta</span><span class="sxs-lookup"><span data-stu-id="30594-477">Delta back-off</span></span><br /><span data-ttu-id="30594-478">Primeira repetição rápida</span><span class="sxs-lookup"><span data-stu-id="30594-478">First fast retry</span></span> |<span data-ttu-id="30594-479">5</span><span class="sxs-lookup"><span data-stu-id="30594-479">5</span></span><br /><span data-ttu-id="30594-480">0 s</span><span class="sxs-lookup"><span data-stu-id="30594-480">0 sec</span></span><br /><span data-ttu-id="30594-481">60 s</span><span class="sxs-lookup"><span data-stu-id="30594-481">60 sec</span></span><br /><span data-ttu-id="30594-482">2 s</span><span class="sxs-lookup"><span data-stu-id="30594-482">2 sec</span></span><br /><span data-ttu-id="30594-483">falso</span><span class="sxs-lookup"><span data-stu-id="30594-483">false</span></span> |<span data-ttu-id="30594-484">1ª tentativa — intervalo de 0 s</span><span class="sxs-lookup"><span data-stu-id="30594-484">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="30594-485">2ª tentativa — intervalo de ~2 s</span><span class="sxs-lookup"><span data-stu-id="30594-485">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="30594-486">3ª tentativa — intervalo de ~6 s</span><span class="sxs-lookup"><span data-stu-id="30594-486">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="30594-487">4ª tentativa — intervalo de ~14 s</span><span class="sxs-lookup"><span data-stu-id="30594-487">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="30594-488">5ª tentativa — intervalo de ~30 s</span><span class="sxs-lookup"><span data-stu-id="30594-488">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="30594-489">A latência de ponta a ponta visa o tempo limite padrão de conexões com o serviço.</span><span class="sxs-lookup"><span data-stu-id="30594-489">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="30594-490">Se você especificar tempos limite de conexão mais longos, a latência de ponta a ponta será estendida por esse tempo adicional para cada tentativa de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-490">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="30594-491">Exemplos</span><span class="sxs-lookup"><span data-stu-id="30594-491">Examples</span></span>
<span data-ttu-id="30594-492">Esta seção mostra como você pode usar a Polly para acessar o banco de dados SQL do Azure usando um conjunto de políticas de repetição configurado na classe `Policy`.</span><span class="sxs-lookup"><span data-stu-id="30594-492">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="30594-493">O código a seguir mostra um método de extensão na classe `SqlCommand` que chama `ExecuteAsync` com retirada exponencial.</span><span class="sxs-lookup"><span data-stu-id="30594-493">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

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

<span data-ttu-id="30594-494">Esse método de extensão assíncrono pode ser usado da seguinte maneira.</span><span class="sxs-lookup"><span data-stu-id="30594-494">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="30594-495">Mais informações</span><span class="sxs-lookup"><span data-stu-id="30594-495">More information</span></span>
* [<span data-ttu-id="30594-496">Camada de acesso a dados dos conceitos básicos do serviço de nuvem — tratamento de falhas transitórias</span><span class="sxs-lookup"><span data-stu-id="30594-496">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="30594-497">Para orientação geral sobre como obter o máximo do banco de dados SQL, confira [Desempenho de banco de dados SQL do Azure e guia de elasticidade](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-497">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>


## <a name="service-bus-retry-guidelines"></a><span data-ttu-id="30594-498">Diretrizes de repetição para Barramento de Serviço</span><span class="sxs-lookup"><span data-stu-id="30594-498">Service Bus retry guidelines</span></span>
<span data-ttu-id="30594-499">O Barramento de Serviço é uma plataforma de mensagens na nuvem que fornece troca de mensagens flexivelmente acoplada com escala e resiliência melhoradas para componentes de um aplicativo, se hospedado na nuvem ou no local.</span><span class="sxs-lookup"><span data-stu-id="30594-499">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="30594-500">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-500">Retry mechanism</span></span>
<span data-ttu-id="30594-501">O Barramento de Serviço implementa repetições usando implementações da classe base [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) .</span><span class="sxs-lookup"><span data-stu-id="30594-501">Service Bus implements retries using implementations of the [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) base class.</span></span> <span data-ttu-id="30594-502">Todos os clientes do Barramento de Serviço expõem uma propriedade **RetryPolicy** que pode ser definida como uma das implementações da classe base **RetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="30594-502">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="30594-503">As implementações internas são:</span><span class="sxs-lookup"><span data-stu-id="30594-503">The built-in implementations are:</span></span>

* <span data-ttu-id="30594-504">A [classe RetryExponential](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-504">The [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span></span> <span data-ttu-id="30594-505">Ela expõe propriedades que controlam o intervalo de retirada, a contagem de repetições e a propriedade **TerminationTimeBuffer** que é usada para limitar o tempo total para a operação ser concluída.</span><span class="sxs-lookup"><span data-stu-id="30594-505">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="30594-506">A [classe NoRetry](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-506">The [NoRetry Class](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span></span> <span data-ttu-id="30594-507">É usada quando as repetições no nível de API do Barramento de Serviço não são exigidas, como quando são gerenciadas por outro processo como parte de uma operação em lote ou de várias etapas.</span><span class="sxs-lookup"><span data-stu-id="30594-507">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="30594-508">As ações do Barramento de Serviço podem retornar uma gama de exceções, conforme listado no [Apêndice: exceções de mensagens](http://msdn.microsoft.com/library/hh418082.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-508">Service Bus actions can return a range of exceptions, as listed in [Appendix: Messaging Exceptions](http://msdn.microsoft.com/library/hh418082.aspx).</span></span> <span data-ttu-id="30594-509">A lista fornece informações que indicam se repetir a operação é adequado.</span><span class="sxs-lookup"><span data-stu-id="30594-509">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="30594-510">Por exemplo, um [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) indica que o cliente deve esperar um período de tempo, depois repetir a operação.</span><span class="sxs-lookup"><span data-stu-id="30594-510">For example, a [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="30594-511">A ocorrência de um **ServerBusyException** também faz com que o Barramento de Serviço alterne para um modo diferente, no qual um intervalo extra de 10 segundos é adicionado aos intervalos de repetição computados.</span><span class="sxs-lookup"><span data-stu-id="30594-511">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="30594-512">Esse modo é redefinido após um curto período.</span><span class="sxs-lookup"><span data-stu-id="30594-512">This mode is reset after a short period.</span></span>

<span data-ttu-id="30594-513">As exceções retornadas do Barramento de Serviço expõem a propriedade **IsTransient** que indica se o cliente deve repetir a operação.</span><span class="sxs-lookup"><span data-stu-id="30594-513">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="30594-514">A política interna **RetryExponential** depende da propriedade **IsTransient** na classe **MessagingException**, que é a classe base para todas as exceções do Barramento de Serviço.</span><span class="sxs-lookup"><span data-stu-id="30594-514">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="30594-515">Se você criar implementações personalizadas da classe base **RetryPolicy**, será possível usar uma combinação do tipo de exceção e da propriedade **IsTransient** para fornecer um controle mais refinado sobre as ações de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-515">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="30594-516">Por exemplo, é possível detectar um **QuotaExceededException** e tomar providências para diminuir a fila antes de tentar enviar novamente uma mensagem para ela.</span><span class="sxs-lookup"><span data-stu-id="30594-516">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="30594-517">Configuração de política</span><span class="sxs-lookup"><span data-stu-id="30594-517">Policy configuration</span></span>
<span data-ttu-id="30594-518">As políticas de repetição são definidas de modo programático e como uma política padrão para um **NamespaceManager** e uma **MessagingFactory** ou individualmente para cada cliente de mensagens.</span><span class="sxs-lookup"><span data-stu-id="30594-518">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="30594-519">Para definir a política de repetição padrão para uma sessão de mensagens, defina **RetryPolicy** do **NamespaceManager**.</span><span class="sxs-lookup"><span data-stu-id="30594-519">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

    namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                 maxBackoff: TimeSpan.FromSeconds(30),
                                                                 maxRetryCount: 3);

<span data-ttu-id="30594-520">Para definir a política de repetição padrão para todos os clientes criados de uma fábrica de mensagens, defina **RetryPolicy** da **MessagingFactory**.</span><span class="sxs-lookup"><span data-stu-id="30594-520">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

    messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                        maxBackoff: TimeSpan.FromSeconds(30),
                                                        maxRetryCount: 3);

<span data-ttu-id="30594-521">Para definir a política de repetição para um cliente de mensagens, ou para substituir sua política padrão, defina a propriedade **RetryPolicy** usando uma instância da classe de política necessária:</span><span class="sxs-lookup"><span data-stu-id="30594-521">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="30594-522">A política de repetição não pode ser definida no nível de operação individual.</span><span class="sxs-lookup"><span data-stu-id="30594-522">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="30594-523">Ela se aplica a todas as operações do cliente de mensagens.</span><span class="sxs-lookup"><span data-stu-id="30594-523">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="30594-524">A tabela a seguir mostra as configurações padrão da política de repetição interna.</span><span class="sxs-lookup"><span data-stu-id="30594-524">The following table shows the default settings for the built-in retry policy.</span></span>

![Tente a tabela de orientação novamente](./images/retry-service-specific/RetryServiceSpecificGuidanceTable7.png)

### <a name="retry-usage-guidance"></a><span data-ttu-id="30594-526">Diretriz de uso de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-526">Retry usage guidance</span></span>
<span data-ttu-id="30594-527">Considere as seguintes diretrizes ao usar o Barramento de Serviço:</span><span class="sxs-lookup"><span data-stu-id="30594-527">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="30594-528">Ao usar a implementação **RetryExponential** interna, não implemente uma operação de fallback, pois a política reage às exceções de Servidor Ocupado e alterna automaticamente para um modo de repetição apropriado.</span><span class="sxs-lookup"><span data-stu-id="30594-528">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="30594-529">O Barramento de Serviço oferece suporte a um recurso chamado Namespaces Emparelhados, que implementa failover automático para uma fila de backup em um namespace separado, caso a fila no namespace primário falhe.</span><span class="sxs-lookup"><span data-stu-id="30594-529">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="30594-530">As mensagens da fila secundária podem ser enviadas de volta para a fila primária quando esta se recuperar.</span><span class="sxs-lookup"><span data-stu-id="30594-530">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="30594-531">Esse recurso ajuda a corrigir falhas transitórias.</span><span class="sxs-lookup"><span data-stu-id="30594-531">This feature helps to address transient failures.</span></span> <span data-ttu-id="30594-532">Para saber mais, consulte [Padrões de mensagens assíncronas e alta disponibilidade](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span><span class="sxs-lookup"><span data-stu-id="30594-532">For more information, see [Asynchronous Messaging Patterns and High Availability](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span></span>

<span data-ttu-id="30594-533">Considere começar com as seguintes configurações para operações de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-533">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="30594-534">Essas são configurações de finalidade geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.</span><span class="sxs-lookup"><span data-stu-id="30594-534">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

![Tente a tabela de orientação novamente](./images/retry-service-specific/RetryServiceSpecificGuidanceTable8.png)

### <a name="telemetry"></a><span data-ttu-id="30594-536">Telemetria</span><span class="sxs-lookup"><span data-stu-id="30594-536">Telemetry</span></span>
<span data-ttu-id="30594-537">O Barramento de Serviço registra as repetições como eventos ETW usando um **EventSource**.</span><span class="sxs-lookup"><span data-stu-id="30594-537">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="30594-538">Você deve anexar um **EventListener** à origem do evento para capturar os eventos e exibi-los no Visualizador de Desempenho ou gravá-los em um log de destino apropriado.</span><span class="sxs-lookup"><span data-stu-id="30594-538">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="30594-539">Você pode usar o [Bloco de Aplicativos para Registro Semântico](http://msdn.microsoft.com/library/dn775006.aspx) para fazer isso.</span><span class="sxs-lookup"><span data-stu-id="30594-539">You could use the [Semantic Logging Application Block](http://msdn.microsoft.com/library/dn775006.aspx) to do this.</span></span> <span data-ttu-id="30594-540">Os eventos de repetição são da seguinte forma:</span><span class="sxs-lookup"><span data-stu-id="30594-540">The retry events are of the following form:</span></span>

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

### <a name="examples"></a><span data-ttu-id="30594-541">Exemplos</span><span class="sxs-lookup"><span data-stu-id="30594-541">Examples</span></span>
<span data-ttu-id="30594-542">O código de exemplo a seguir mostra como definir a política de repetição para:</span><span class="sxs-lookup"><span data-stu-id="30594-542">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="30594-543">Um gerenciador de namespaces.</span><span class="sxs-lookup"><span data-stu-id="30594-543">A namespace manager.</span></span> <span data-ttu-id="30594-544">A política se aplica a todas as operações nesse gerenciador e não pode ser substituída para operações individuais.</span><span class="sxs-lookup"><span data-stu-id="30594-544">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="30594-545">Uma fábrica de mensagens.</span><span class="sxs-lookup"><span data-stu-id="30594-545">A messaging factory.</span></span> <span data-ttu-id="30594-546">A política se aplica a todos os clientes criados nessa fábrica e não pode ser substituída durante a criação de clientes individuais.</span><span class="sxs-lookup"><span data-stu-id="30594-546">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="30594-547">Um cliente de mensagens individual.</span><span class="sxs-lookup"><span data-stu-id="30594-547">An individual messaging client.</span></span> <span data-ttu-id="30594-548">Depois que um cliente tiver sido criado, você poderá definir a política de repetição para esse cliente.</span><span class="sxs-lookup"><span data-stu-id="30594-548">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="30594-549">A política se aplica a todas as operações nesse cliente.</span><span class="sxs-lookup"><span data-stu-id="30594-549">The policy applies to all operations on that client.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="30594-550">Mais informações</span><span class="sxs-lookup"><span data-stu-id="30594-550">More information</span></span>
* [<span data-ttu-id="30594-551">Padrões de mensagens assíncronas e alta disponibilidade</span><span class="sxs-lookup"><span data-stu-id="30594-551">Asynchronous Messaging Patterns and High Availability</span></span>](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="azure-redis-cache-retry-guidelines"></a><span data-ttu-id="30594-552">Diretrizes de repetição para Cache Redis do Azure</span><span class="sxs-lookup"><span data-stu-id="30594-552">Azure Redis Cache retry guidelines</span></span>
<span data-ttu-id="30594-553">O Cache Redis do Azure é um serviço de cache de baixa latência e acesso a dados rápido com base no Cache Redis de software livre popular.</span><span class="sxs-lookup"><span data-stu-id="30594-553">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="30594-554">Ele é seguro, gerenciado pela Microsoft e pode ser acessado de qualquer aplicativo no Azure.</span><span class="sxs-lookup"><span data-stu-id="30594-554">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="30594-555">A diretriz nesta seção se baseia em como usar o cliente StackExchange.Redis para acessar o cache.</span><span class="sxs-lookup"><span data-stu-id="30594-555">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="30594-556">Uma lista de outros clientes adequados pode ser encontrada no [site do Redis](http://redis.io/clients), e eles podem ter mecanismos de repetição diferentes.</span><span class="sxs-lookup"><span data-stu-id="30594-556">A list of other suitable clients can be found on the [Redis website](http://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="30594-557">Observe que o cliente StackExchange.Redis usa multiplexação por meio de uma única conexão.</span><span class="sxs-lookup"><span data-stu-id="30594-557">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="30594-558">O uso recomendado é criar uma instância do cliente na inicialização do aplicativo e usar essa instância para todas as operações no cache.</span><span class="sxs-lookup"><span data-stu-id="30594-558">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="30594-559">Por esse motivo, a conexão com o cache é feita apenas uma vez, e toda a orientação desta seção está relacionada à política de repetição para esta conexão inicial, e não para cada operação que acessa o cache.</span><span class="sxs-lookup"><span data-stu-id="30594-559">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="30594-560">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-560">Retry mechanism</span></span>
<span data-ttu-id="30594-561">O cliente StackExchange.Redis usa uma classe de gerenciador de conexões que é configurada por meio de um conjunto de opções, incluindo:</span><span class="sxs-lookup"><span data-stu-id="30594-561">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, incuding:</span></span>

- <span data-ttu-id="30594-562">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="30594-562">**ConnectRetry**.</span></span> <span data-ttu-id="30594-563">O número de vezes que uma conexão com falha para o cache será repetida.</span><span class="sxs-lookup"><span data-stu-id="30594-563">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="30594-564">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="30594-564">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="30594-565">Usar qual estratégia de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-565">The retry strategy to use.</span></span>
- <span data-ttu-id="30594-566">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="30594-566">**ConnectTimeout**.</span></span> <span data-ttu-id="30594-567">O tempo máximo de espera em milissegundos.</span><span class="sxs-lookup"><span data-stu-id="30594-567">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="30594-568">Configuração de política</span><span class="sxs-lookup"><span data-stu-id="30594-568">Policy configuration</span></span>
<span data-ttu-id="30594-569">As políticas de repetição são configuradas de modo programático, definindo as opções para o cliente antes de se conectar ao cache.</span><span class="sxs-lookup"><span data-stu-id="30594-569">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="30594-570">Isso pode ser feito criando uma instância da classe **ConfigurationOptions**, populando suas propriedades e passando-a para o método **Connect**.</span><span class="sxs-lookup"><span data-stu-id="30594-570">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="30594-571">As classes internas suportam atraso linear (constante) e retirada exponencial com intervalos de repetição aleatórios.</span><span class="sxs-lookup"><span data-stu-id="30594-571">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="30594-572">Você também pode criar uma política de repetição personalizada se implementar a interface **IReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="30594-572">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="30594-573">O exemplo a seguir configura uma estratégia de repetição com retirada exponencial.</span><span class="sxs-lookup"><span data-stu-id="30594-573">The following example configures a retry strategy using exponential backoff.</span></span>

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

<span data-ttu-id="30594-574">Como alternativa, você pode especificar as opções como uma cadeia de caracteres e passá-la ao método **Connect** .</span><span class="sxs-lookup"><span data-stu-id="30594-574">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="30594-575">Observe que a propriedade **ReconnectRetryPolicy** não pode ser definida dessa forma, somente através de código.</span><span class="sxs-lookup"><span data-stu-id="30594-575">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
    var options = "localhost,connectRetry=3,connectTimeout=2000";
    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="30594-576">Também é possível especificar opções diretamente ao se conectar ao cache.</span><span class="sxs-lookup"><span data-stu-id="30594-576">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="30594-577">Para obter mais informações, confira [Configuração do Stack Exchange Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration) na documentação do StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="30594-577">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="30594-578">A tabela a seguir mostra as configurações padrão da política de repetição interna.</span><span class="sxs-lookup"><span data-stu-id="30594-578">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="30594-579">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="30594-579">**Context**</span></span> | <span data-ttu-id="30594-580">**Configuração**</span><span class="sxs-lookup"><span data-stu-id="30594-580">**Setting**</span></span> | <span data-ttu-id="30594-581">**Valor padrão**</span><span class="sxs-lookup"><span data-stu-id="30594-581">**Default value**</span></span><br /><span data-ttu-id="30594-582">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="30594-582">(v 1.2.2)</span></span> | <span data-ttu-id="30594-583">**Significado**</span><span class="sxs-lookup"><span data-stu-id="30594-583">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="30594-584">ConfigurationOptions</span><span class="sxs-lookup"><span data-stu-id="30594-584">ConfigurationOptions</span></span> |<span data-ttu-id="30594-585">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="30594-585">ConnectRetry</span></span><br /><br /><span data-ttu-id="30594-586">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="30594-586">ConnectTimeout</span></span><br /><br /><span data-ttu-id="30594-587">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="30594-587">SyncTimeout</span></span><br /><br /><span data-ttu-id="30594-588">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="30594-588">ReconnectRetryPolicy</span></span> |<span data-ttu-id="30594-589">3</span><span class="sxs-lookup"><span data-stu-id="30594-589">3</span></span><br /><br /><span data-ttu-id="30594-590">Máximo de 5000 ms mais SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="30594-590">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="30594-591">1000</span><span class="sxs-lookup"><span data-stu-id="30594-591">1000</span></span><br /><br /><span data-ttu-id="30594-592">LinearRetry 5.000 ms</span><span class="sxs-lookup"><span data-stu-id="30594-592">LinearRetry 5000 ms</span></span> |<span data-ttu-id="30594-593">O número de vezes para repetir tentativas de conexão durante a operação de conexão inicial.</span><span class="sxs-lookup"><span data-stu-id="30594-593">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="30594-594">Tempo limite (ms) para operações de conexão.</span><span class="sxs-lookup"><span data-stu-id="30594-594">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="30594-595">Não é um intervalo entre tentativas de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-595">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="30594-596">Tempo (ms) para permitir operações síncronas.</span><span class="sxs-lookup"><span data-stu-id="30594-596">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="30594-597">Repita a cada 5.000 ms.</span><span class="sxs-lookup"><span data-stu-id="30594-597">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="30594-598">Em operações síncronas, `SyncTimeout` pode adicionar à latência de ponta a ponta, porém definir o valor muito baixo pode causar tempos limite excessivos.</span><span class="sxs-lookup"><span data-stu-id="30594-598">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="30594-599">Confira [Como solucionar problemas do Cache Redis do Azure][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="30594-599">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="30594-600">Em geral, evite usar operações síncronas e use operações assíncronas.</span><span class="sxs-lookup"><span data-stu-id="30594-600">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="30594-601">Para saber mais, consulte [Pipelines e multiplexadores](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="30594-601">For more information see [Pipelines and Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="30594-602">Diretriz de uso de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-602">Retry usage guidance</span></span>
<span data-ttu-id="30594-603">Considere as seguintes diretrizes ao usar o Cache Redis do Azure:</span><span class="sxs-lookup"><span data-stu-id="30594-603">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="30594-604">O cliente StackExchange Redis gerencia suas próprias repetições, mas apenas ao estabelecer uma conexão com o cache quando o aplicativo é iniciado pela primeira vez.</span><span class="sxs-lookup"><span data-stu-id="30594-604">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="30594-605">Você pode configurar o tempo limite da conexão, o número de repetições e o intervalo entre as repetições para estabelecer esta conexão, porém a política de repetições não se aplica a operações contra o cache.</span><span class="sxs-lookup"><span data-stu-id="30594-605">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="30594-606">Em vez de usar um grande número de tentativas de repetição, considere fazer fallback acessando a fonte de dados original.</span><span class="sxs-lookup"><span data-stu-id="30594-606">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="30594-607">Telemetria</span><span class="sxs-lookup"><span data-stu-id="30594-607">Telemetry</span></span>
<span data-ttu-id="30594-608">Você pode coletar informações sobre conexões (mas não outras operações) usando um **TextWriter**.</span><span class="sxs-lookup"><span data-stu-id="30594-608">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
...
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="30594-609">Um exemplo da saída que isso gera é mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="30594-609">An example of the output this generates is shown below.</span></span>

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

### <a name="examples"></a><span data-ttu-id="30594-610">Exemplos</span><span class="sxs-lookup"><span data-stu-id="30594-610">Examples</span></span>
<span data-ttu-id="30594-611">O exemplo de código a seguir configura um atraso constante (linear) entre as repetições ao inicializar o cliente StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="30594-611">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="30594-612">Este exemplo mostra como definir a configuração usando uma instância de **ConfigurationOptions**.</span><span class="sxs-lookup"><span data-stu-id="30594-612">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

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

<span data-ttu-id="30594-613">Este exemplo mostra como definir a configuração especificando as opções como uma cadeia de caracteres.</span><span class="sxs-lookup"><span data-stu-id="30594-613">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="30594-614">O tempo limite da conexão é o período máximo de espera por uma conexão para o cache, não o atraso entre tentativas de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-614">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="30594-615">Observe que a propriedade **ReconnectRetryPolicy** só pode ser definida por código.</span><span class="sxs-lookup"><span data-stu-id="30594-615">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

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

<span data-ttu-id="30594-616">Para obter mais exemplos, consulte [Configuração](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) no site do projeto.</span><span class="sxs-lookup"><span data-stu-id="30594-616">For more examples, see [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="30594-617">Mais informações</span><span class="sxs-lookup"><span data-stu-id="30594-617">More information</span></span>
* [<span data-ttu-id="30594-618">Site do Redis</span><span class="sxs-lookup"><span data-stu-id="30594-618">Redis website</span></span>](http://redis.io/)

## <a name="documentdb-api-retry-guidelines"></a><span data-ttu-id="30594-619">Diretrizes de repetição da API do DocumentDB</span><span class="sxs-lookup"><span data-stu-id="30594-619">DocumentDB API retry guidelines</span></span>

<span data-ttu-id="30594-620">O Cosmos DB é um banco de dados multimodelo totalmente gerenciado que oferece suporte a dados JSON sem esquema usando a [API do DocumentDB][documentdb-api].</span><span class="sxs-lookup"><span data-stu-id="30594-620">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data by using the [DocumentDB API][documentdb-api].</span></span> <span data-ttu-id="30594-621">Ele oferece desempenho configurável e confiável, processamento transacional nativo JavaScript, além de ser criado para a nuvem com escala elástica.</span><span class="sxs-lookup"><span data-stu-id="30594-621">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="30594-622">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-622">Retry mechanism</span></span>
<span data-ttu-id="30594-623">A classe `DocumentClient` repete automaticamente tentativas com falha.</span><span class="sxs-lookup"><span data-stu-id="30594-623">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="30594-624">Para definir o número de repetições e o tempo de espera máximo, configure [ConnectionPolicy.RetryOptions].</span><span class="sxs-lookup"><span data-stu-id="30594-624">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="30594-625">As exceções que o cliente gera excedem a política de repetição ou não são erros transitórios.</span><span class="sxs-lookup"><span data-stu-id="30594-625">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="30594-626">Se o Cosmos DB limitar o cliente, ele retornará um erro HTTP 429.</span><span class="sxs-lookup"><span data-stu-id="30594-626">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="30594-627">Confira o código de status no `DocumentClientException`.</span><span class="sxs-lookup"><span data-stu-id="30594-627">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="30594-628">Configuração de política</span><span class="sxs-lookup"><span data-stu-id="30594-628">Policy configuration</span></span>
<span data-ttu-id="30594-629">A tabela a seguir mostra as configurações padrão para a classe `RetryOptions`.</span><span class="sxs-lookup"><span data-stu-id="30594-629">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="30594-630">Configuração</span><span class="sxs-lookup"><span data-stu-id="30594-630">Setting</span></span> | <span data-ttu-id="30594-631">Valor padrão</span><span class="sxs-lookup"><span data-stu-id="30594-631">Default value</span></span> | <span data-ttu-id="30594-632">Descrição</span><span class="sxs-lookup"><span data-stu-id="30594-632">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="30594-633">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="30594-633">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="30594-634">9</span><span class="sxs-lookup"><span data-stu-id="30594-634">9</span></span> |<span data-ttu-id="30594-635">O número máximo de tentativas se a solicitação falhar porque o Cosmos DB aplicou a limitação de taxa ao cliente.</span><span class="sxs-lookup"><span data-stu-id="30594-635">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="30594-636">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="30594-636">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="30594-637">30</span><span class="sxs-lookup"><span data-stu-id="30594-637">30</span></span> |<span data-ttu-id="30594-638">O tempo máximo de repetição, em segundos.</span><span class="sxs-lookup"><span data-stu-id="30594-638">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="30594-639">Exemplo</span><span class="sxs-lookup"><span data-stu-id="30594-639">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="30594-640">Telemetria</span><span class="sxs-lookup"><span data-stu-id="30594-640">Telemetry</span></span>
<span data-ttu-id="30594-641">As tentativas de repetição são registradas como mensagens de rastreamento não estruturadas por meio de um **TraceSource**do .NET.</span><span class="sxs-lookup"><span data-stu-id="30594-641">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="30594-642">Você deve configurar um **TraceListener** para capturar os eventos e gravá-los em um log de destino apropriado.</span><span class="sxs-lookup"><span data-stu-id="30594-642">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="30594-643">Por exemplo, se você adicionar o seguinte ao arquivo App.config, rastreamentos serão gerados em um arquivo de texto no mesmo local que o executável:</span><span class="sxs-lookup"><span data-stu-id="30594-643">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

```
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="DocumentDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```


## <a name="azure-search-retry-guidelines"></a><span data-ttu-id="30594-644">Diretrizes de repetição para Azure Search</span><span class="sxs-lookup"><span data-stu-id="30594-644">Azure Search retry guidelines</span></span>
<span data-ttu-id="30594-645">O Azure Search pode ser usada para adicionar recursos potentes e sofisticados a um site ou aplicativo, ajustar os resultados da pesquisa de maneira rápida e fácil, bem como construir modelos de classificação avançados e ajustados.</span><span class="sxs-lookup"><span data-stu-id="30594-645">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="30594-646">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-646">Retry mechanism</span></span>
<span data-ttu-id="30594-647">O comportamento de repetição no SDK do Azure Search é controlado pelo método `SetRetryPolicy` nas classes [SearchServiceClient] e [SearchIndexClient].</span><span class="sxs-lookup"><span data-stu-id="30594-647">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="30594-648">A política padrão tenta novamente com retirada exponencial quando o Azure Search retorna uma resposta 5xx ou 408 (Tempo Limite da Solicitação).</span><span class="sxs-lookup"><span data-stu-id="30594-648">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="30594-649">Telemetria</span><span class="sxs-lookup"><span data-stu-id="30594-649">Telemetry</span></span>
<span data-ttu-id="30594-650">Rastreamento com o ETW ou por meio de registro de um provedor de rastreamento personalizado.</span><span class="sxs-lookup"><span data-stu-id="30594-650">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="30594-651">Para obter mais informações, confira [Documentação do AutoRest][autorest].</span><span class="sxs-lookup"><span data-stu-id="30594-651">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="azure-active-directory-retry-guidelines"></a><span data-ttu-id="30594-652">Diretrizes de repetição para o Active Directory do Azure</span><span class="sxs-lookup"><span data-stu-id="30594-652">Azure Active Directory retry guidelines</span></span>
<span data-ttu-id="30594-653">O Azure Active Directory (Azure AD) é uma solução abrangente de nuvem para gerenciamento de acesso e identidade que combina serviços principais de diretório, governança avançada de identidade, segurança e gerenciamento de acesso a aplicativos.</span><span class="sxs-lookup"><span data-stu-id="30594-653">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="30594-654">O AD do Azure também oferece aos desenvolvedores uma plataforma de gerenciamento de identidade para fornecer controle de acesso aos respectivos aplicativos, com base nas regras e políticas centralizadas.</span><span class="sxs-lookup"><span data-stu-id="30594-654">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="30594-655">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-655">Retry mechanism</span></span>
<span data-ttu-id="30594-656">Na Biblioteca de autenticação do Active Directory (ADAL) há um mecanismo interno de repetição para o Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="30594-656">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="30594-657">Para evitar bloqueios inesperados, recomendamos que o ADAL lide com as repetições e *não* permita que as bibliotecas de terceiros ou o código do aplicativo repitam as conexões com falhas.</span><span class="sxs-lookup"><span data-stu-id="30594-657">To avoid unexpected lockouts, we recommend that third party libraries and application code do *not* retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="30594-658">Diretriz de uso de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-658">Retry usage guidance</span></span>
<span data-ttu-id="30594-659">Considere as seguintes diretrizes ao usar o Active Directory do Azure:</span><span class="sxs-lookup"><span data-stu-id="30594-659">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="30594-660">Quando possível, use a biblioteca ADAL e o suporte interno para repetições.</span><span class="sxs-lookup"><span data-stu-id="30594-660">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="30594-661">Se estiver usando a API REST do Active Directory do Azure, você deverá repetir a operação somente se o resultado for um erro no intervalo 5xx (como 500 Erro de Servidor Interno, 502 Gateway Incorreto, 503 Serviço Indisponível e 504 Tempo Limite do Gateway).</span><span class="sxs-lookup"><span data-stu-id="30594-661">If you are using the REST API for Azure Active Directory, you should retry the operation only if the result is an error in the 5xx range (such as 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, and 504 Gateway Timeout).</span></span> <span data-ttu-id="30594-662">Não repita para nenhum outro erro.</span><span class="sxs-lookup"><span data-stu-id="30594-662">Do not retry for any other errors.</span></span>
* <span data-ttu-id="30594-663">Uma política de retirada exponencial é recomendada para uso em cenários de lote com o Active Directory do Azure.</span><span class="sxs-lookup"><span data-stu-id="30594-663">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="30594-664">Considere começar com as seguintes configurações para operações de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-664">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="30594-665">Essas são configurações de finalidade geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.</span><span class="sxs-lookup"><span data-stu-id="30594-665">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="30594-666">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="30594-666">**Context**</span></span> | <span data-ttu-id="30594-667">**Exemplo de latênciaE2E<br />máxima de destino**</span><span class="sxs-lookup"><span data-stu-id="30594-667">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="30594-668">**Estratégia de repetição**</span><span class="sxs-lookup"><span data-stu-id="30594-668">**Retry strategy**</span></span> | <span data-ttu-id="30594-669">**Configurações**</span><span class="sxs-lookup"><span data-stu-id="30594-669">**Settings**</span></span> | <span data-ttu-id="30594-670">**Valores**</span><span class="sxs-lookup"><span data-stu-id="30594-670">**Values**</span></span> | <span data-ttu-id="30594-671">**Como funciona**</span><span class="sxs-lookup"><span data-stu-id="30594-671">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="30594-672">Interativo, interface de usuário</span><span class="sxs-lookup"><span data-stu-id="30594-672">Interactive, UI,</span></span><br /><span data-ttu-id="30594-673">ou primeiro plano</span><span class="sxs-lookup"><span data-stu-id="30594-673">or foreground</span></span> |<span data-ttu-id="30594-674">2 s</span><span class="sxs-lookup"><span data-stu-id="30594-674">2 sec</span></span> |<span data-ttu-id="30594-675">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="30594-675">FixedInterval</span></span> |<span data-ttu-id="30594-676">Contagem de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-676">Retry count</span></span><br /><span data-ttu-id="30594-677">Intervalo de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-677">Retry interval</span></span><br /><span data-ttu-id="30594-678">Primeira repetição rápida</span><span class="sxs-lookup"><span data-stu-id="30594-678">First fast retry</span></span> |<span data-ttu-id="30594-679">3</span><span class="sxs-lookup"><span data-stu-id="30594-679">3</span></span><br /><span data-ttu-id="30594-680">500 ms</span><span class="sxs-lookup"><span data-stu-id="30594-680">500 ms</span></span><br /><span data-ttu-id="30594-681">verdadeiro</span><span class="sxs-lookup"><span data-stu-id="30594-681">true</span></span> |<span data-ttu-id="30594-682">1ª tentativa — intervalo de 0 s</span><span class="sxs-lookup"><span data-stu-id="30594-682">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="30594-683">2ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="30594-683">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="30594-684">3ª tentativa — intervalo de 500 ms</span><span class="sxs-lookup"><span data-stu-id="30594-684">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="30594-685">Segundo plano ou</span><span class="sxs-lookup"><span data-stu-id="30594-685">Background or</span></span><br /><span data-ttu-id="30594-686">lote</span><span class="sxs-lookup"><span data-stu-id="30594-686">batch</span></span> |<span data-ttu-id="30594-687">60 s</span><span class="sxs-lookup"><span data-stu-id="30594-687">60 sec</span></span> |<span data-ttu-id="30594-688">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="30594-688">ExponentialBackoff</span></span> |<span data-ttu-id="30594-689">Contagem de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-689">Retry count</span></span><br /><span data-ttu-id="30594-690">Retirada mín.</span><span class="sxs-lookup"><span data-stu-id="30594-690">Min back-off</span></span><br /><span data-ttu-id="30594-691">Retirada máx.</span><span class="sxs-lookup"><span data-stu-id="30594-691">Max back-off</span></span><br /><span data-ttu-id="30594-692">Retirada delta</span><span class="sxs-lookup"><span data-stu-id="30594-692">Delta back-off</span></span><br /><span data-ttu-id="30594-693">Primeira repetição rápida</span><span class="sxs-lookup"><span data-stu-id="30594-693">First fast retry</span></span> |<span data-ttu-id="30594-694">5</span><span class="sxs-lookup"><span data-stu-id="30594-694">5</span></span><br /><span data-ttu-id="30594-695">0 s</span><span class="sxs-lookup"><span data-stu-id="30594-695">0 sec</span></span><br /><span data-ttu-id="30594-696">60 s</span><span class="sxs-lookup"><span data-stu-id="30594-696">60 sec</span></span><br /><span data-ttu-id="30594-697">2 s</span><span class="sxs-lookup"><span data-stu-id="30594-697">2 sec</span></span><br /><span data-ttu-id="30594-698">falso</span><span class="sxs-lookup"><span data-stu-id="30594-698">false</span></span> |<span data-ttu-id="30594-699">1ª tentativa — intervalo de 0 s</span><span class="sxs-lookup"><span data-stu-id="30594-699">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="30594-700">2ª tentativa — intervalo de ~2 s</span><span class="sxs-lookup"><span data-stu-id="30594-700">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="30594-701">3ª tentativa — intervalo de ~6 s</span><span class="sxs-lookup"><span data-stu-id="30594-701">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="30594-702">4ª tentativa — intervalo de ~14 s</span><span class="sxs-lookup"><span data-stu-id="30594-702">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="30594-703">5ª tentativa — intervalo de ~30 s</span><span class="sxs-lookup"><span data-stu-id="30594-703">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="30594-704">Mais informações</span><span class="sxs-lookup"><span data-stu-id="30594-704">More information</span></span>
* <span data-ttu-id="30594-705">[Bibliotecas de autenticação do Azure Active Directory][adal]</span><span class="sxs-lookup"><span data-stu-id="30594-705">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="service-fabric-retry-guidelines"></a><span data-ttu-id="30594-706">Diretrizes para repetição do Service Fabric</span><span class="sxs-lookup"><span data-stu-id="30594-706">Service Fabric retry guidelines</span></span>

<span data-ttu-id="30594-707">Distribuir serviços confiáveis em um cluster do Service Fabric protege contra a maioria das falhas transitórias potenciais discutidas neste artigo.</span><span class="sxs-lookup"><span data-stu-id="30594-707">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="30594-708">No entanto, algumas falhas transitórias ainda podem acontecer.</span><span class="sxs-lookup"><span data-stu-id="30594-708">Some transient faults are still possible, however.</span></span> <span data-ttu-id="30594-709">Por exemplo, se o serviço de nomeação recebe uma solicitação no meio de uma alteração de roteamento, ele pode lançar uma exceção.</span><span class="sxs-lookup"><span data-stu-id="30594-709">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="30594-710">No entanto, se a mesma solicitação vier 100 milissegundos mais tarde, ela provavelmente terá êxito.</span><span class="sxs-lookup"><span data-stu-id="30594-710">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="30594-711">Internamente, o Service Fabric gerencia esse tipo de falha transitória.</span><span class="sxs-lookup"><span data-stu-id="30594-711">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="30594-712">Você pode definir algumas configurações usando a classe `OperationRetrySettings` ao configurar seus serviços.</span><span class="sxs-lookup"><span data-stu-id="30594-712">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="30594-713">O código a seguir mostra um exemplo.</span><span class="sxs-lookup"><span data-stu-id="30594-713">The following code shows an example.</span></span> <span data-ttu-id="30594-714">Na maioria dos casos, isso não deve ser necessário e as configurações padrão são suficientes.</span><span class="sxs-lookup"><span data-stu-id="30594-714">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

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

## <a name="more-information"></a><span data-ttu-id="30594-715">Mais informações</span><span class="sxs-lookup"><span data-stu-id="30594-715">More information</span></span>

* [<span data-ttu-id="30594-716">Manipulação de exceção remota</span><span class="sxs-lookup"><span data-stu-id="30594-716">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)


## <a name="azure-event-hubs-retry-guidelines"></a><span data-ttu-id="30594-717">Diretrizes sobre repetição de hubs de eventos do Azure</span><span class="sxs-lookup"><span data-stu-id="30594-717">Azure Event Hubs retry guidelines</span></span>

<span data-ttu-id="30594-718">Os hubs de eventos do Azure são um serviço de ingestão de telemetria de hiperescala que coleta, transforma e armazena milhões de eventos.</span><span class="sxs-lookup"><span data-stu-id="30594-718">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="30594-719">Mecanismo de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-719">Retry mechanism</span></span>
<span data-ttu-id="30594-720">O comportamento de repetição na biblioteca de cliente dos hubs de eventos do Azure é controlado pela propriedade `RetryPolicy` na classe `EventHubClient`.</span><span class="sxs-lookup"><span data-stu-id="30594-720">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="30594-721">As tentativas de política padrão com retirada exponencial quando o Hub de eventos do Azure retorna um `EventHubsException` transitório ou um `OperationCanceledException`.</span><span class="sxs-lookup"><span data-stu-id="30594-721">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="30594-722">Exemplo</span><span class="sxs-lookup"><span data-stu-id="30594-722">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="30594-723">Mais informações</span><span class="sxs-lookup"><span data-stu-id="30594-723">More information</span></span>
[<span data-ttu-id="30594-724">Biblioteca de cliente padrão .NET para hubs de eventos do Azure</span><span class="sxs-lookup"><span data-stu-id="30594-724"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="30594-725">Diretrizes gerais de repetição e REST</span><span class="sxs-lookup"><span data-stu-id="30594-725">General REST and retry guidelines</span></span>
<span data-ttu-id="30594-726">Considere o seguinte ao acessar os serviços do Azure ou de terceiros:</span><span class="sxs-lookup"><span data-stu-id="30594-726">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="30594-727">Use uma abordagem sistemática para gerenciar repetições, talvez como código reutilizável, para que você possa aplicar uma metodologia consistente em todos os clientes e soluções.</span><span class="sxs-lookup"><span data-stu-id="30594-727">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="30594-728">Considere usar uma estrutura de repetição, como o Bloco de Aplicativos para Tratamento de Falhas Transitórias, para gerenciar repetições se o serviço ou cliente de destino não tiver algum mecanismo de repetição interno.</span><span class="sxs-lookup"><span data-stu-id="30594-728">Consider using a retry framework such as the Transient Fault Handling Application Block to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="30594-729">Isso ajudará você a implementar um comportamento de repetição consistente, bem como pode fornecer uma estratégia de repetição padrão adequada para o serviço de destino.</span><span class="sxs-lookup"><span data-stu-id="30594-729">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="30594-730">No entanto, talvez seja necessário criar código de repetição personalizado para serviços que tenham comportamento não padrão, que não dependem de exceções para indicar falhas transitórias, ou se desejar, use uma resposta **Retry-Response** para gerenciar o comportamento de repetição.</span><span class="sxs-lookup"><span data-stu-id="30594-730">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="30594-731">A lógica de detecção transitória dependerá da API de cliente real que você usa para invocar as chamadas REST.</span><span class="sxs-lookup"><span data-stu-id="30594-731">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="30594-732">Alguns clientes, como a classe mais recente **HttpClient** , não lançam exceções para solicitações concluídas com um código de status HTTP sem sucesso.</span><span class="sxs-lookup"><span data-stu-id="30594-732">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> <span data-ttu-id="30594-733">Isso melhora o desempenho, mas impede o uso do Bloco de Aplicativos para Tratamento de Falhas Transitórias.</span><span class="sxs-lookup"><span data-stu-id="30594-733">This improves performance but prevents the use of the Transient Fault Handling Application Block.</span></span> <span data-ttu-id="30594-734">Nesse caso, você pode encapsular a chamada à API REST com o código que gera exceções para códigos de status HTTP sem sucesso, que pode então ser processada pelo bloco.</span><span class="sxs-lookup"><span data-stu-id="30594-734">In this case you could wrap the call to the REST API with code that produces exceptions for non-success HTTP status codes, which can then be processed by the block.</span></span> <span data-ttu-id="30594-735">Como alternativa, é possível usar um mecanismo diferente para orientar as repetições.</span><span class="sxs-lookup"><span data-stu-id="30594-735">Alternatively, you can use a different mechanism to drive the retries.</span></span>
* <span data-ttu-id="30594-736">O código de status HTTP retornado do serviço pode ajudar a indicar se a falha é transitória.</span><span class="sxs-lookup"><span data-stu-id="30594-736">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="30594-737">Talvez seja necessário examinar as exceções geradas por um cliente ou pela estrutura de repetição para acessar o código de status ou determinar o tipo de exceção equivalente.</span><span class="sxs-lookup"><span data-stu-id="30594-737">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="30594-738">Os seguintes códigos HTTP geralmente indicam que uma repetição é apropriada:</span><span class="sxs-lookup"><span data-stu-id="30594-738">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="30594-739">408 Tempo Limite da Solicitação</span><span class="sxs-lookup"><span data-stu-id="30594-739">408 Request Timeout</span></span>
  * <span data-ttu-id="30594-740">500 Erro Interno do Servidor</span><span class="sxs-lookup"><span data-stu-id="30594-740">500 Internal Server Error</span></span>
  * <span data-ttu-id="30594-741">502 Gateway Incorreto</span><span class="sxs-lookup"><span data-stu-id="30594-741">502 Bad Gateway</span></span>
  * <span data-ttu-id="30594-742">503 Serviço Indisponível</span><span class="sxs-lookup"><span data-stu-id="30594-742">503 Service Unavailable</span></span>
  * <span data-ttu-id="30594-743">504 Tempo Limite do Gateway</span><span class="sxs-lookup"><span data-stu-id="30594-743">504 Gateway Timeout</span></span>
* <span data-ttu-id="30594-744">Se você basear suas lógica de repetição em exceções, geralmente o que se segue indica uma falha transitória onde nenhuma conexão pode ser estabelecida:</span><span class="sxs-lookup"><span data-stu-id="30594-744">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="30594-745">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="30594-745">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="30594-746">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="30594-746">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="30594-747">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="30594-747">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="30594-748">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="30594-748">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="30594-749">No caso de um status de serviço indisponível, o serviço pode indicar o atraso apropriado antes de tentar a repetição no cabeçalho da resposta **Retry-After** ou em um cabeçalho personalizado diferente.</span><span class="sxs-lookup"><span data-stu-id="30594-749">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="30594-750">Os serviços também podem enviar informações adicionais como cabeçalhos personalizados ou inseridos no conteúdo da resposta.</span><span class="sxs-lookup"><span data-stu-id="30594-750">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> <span data-ttu-id="30594-751">O Bloco de Aplicativos para Tratamento de Falhas Transitórias não pode usar os cabeçalhos “retry-after” padrão ou personalizados.</span><span class="sxs-lookup"><span data-stu-id="30594-751">The Transient Fault Handling Application Block cannot use the standard or any custom “retry-after” headers.</span></span>
* <span data-ttu-id="30594-752">Não tente a repetição para códigos de status que representam erros de cliente (erros no intervalo 4xx), exceto para um 408 Tempo Limite de Solicitação.</span><span class="sxs-lookup"><span data-stu-id="30594-752">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="30594-753">Teste minuciosamente seus mecanismos e estratégias de repetição sob diversas condições, como estado diferente de rede e cargas variáveis de sistema.</span><span class="sxs-lookup"><span data-stu-id="30594-753">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="30594-754">Estratégias de repetição</span><span class="sxs-lookup"><span data-stu-id="30594-754">Retry strategies</span></span>
<span data-ttu-id="30594-755">Veja a seguir os tipos comuns de intervalo de estratégias de repetição:</span><span class="sxs-lookup"><span data-stu-id="30594-755">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="30594-756">**Exponencial**: uma política de repetição que executa um determinado número de repetições usando uma abordagem de retirada exponencial aleatória para determinar o intervalo entre as repetições.</span><span class="sxs-lookup"><span data-stu-id="30594-756">**Exponential**: A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="30594-757">Por exemplo:</span><span class="sxs-lookup"><span data-stu-id="30594-757">For example:</span></span>

        var random = new Random();

        var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                    random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                    (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
        var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                       this.maxBackoff.TotalMilliseconds);
        retryInterval = TimeSpan.FromMilliseconds(interval);
* <span data-ttu-id="30594-758">**Incremental**: uma estratégia de repetição com um número especificado de tentativas de repetição e um intervalo de tempo incremental entre entradas.</span><span class="sxs-lookup"><span data-stu-id="30594-758">**Incremental**: A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="30594-759">Por exemplo:</span><span class="sxs-lookup"><span data-stu-id="30594-759">For example:</span></span>

        retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                       (this.increment.TotalMilliseconds * currentRetryCount));
* <span data-ttu-id="30594-760">**LinearRetry**: uma política de repetição que executa um número especificado de repetições usando um intervalo de tempo fixo especificado entre as repetições.</span><span class="sxs-lookup"><span data-stu-id="30594-760">**LinearRetry**: A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="30594-761">Por exemplo:</span><span class="sxs-lookup"><span data-stu-id="30594-761">For example:</span></span>

        retryInterval = this.deltaBackoff;

### <a name="more-information"></a><span data-ttu-id="30594-762">Mais informações</span><span class="sxs-lookup"><span data-stu-id="30594-762">More information</span></span>
* [<span data-ttu-id="30594-763">Estratégias de disjuntor</span><span class="sxs-lookup"><span data-stu-id="30594-763">Circuit breaker strategies</span></span>](http://msdn.microsoft.com/library/dn589784.aspx)

## <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="30594-764">Falha transitória manipulada com a Polly</span><span class="sxs-lookup"><span data-stu-id="30594-764">Transient fault handling with Polly</span></span>
<span data-ttu-id="30594-765">[Polly](http://www.thepollyproject.org) é uma biblioteca para manipular programaticamente novas tentativas e estratégias de [disjuntor] [ circuit-breaker].</span><span class="sxs-lookup"><span data-stu-id="30594-765">[Polly](http://www.thepollyproject.org) is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="30594-766">O projeto Polly é um membro da [Fundação .NET][dotnet-foundation].</span><span class="sxs-lookup"><span data-stu-id="30594-766">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="30594-767">Para serviços em que o cliente não oferece suporte a novas tentativas, a Polly é uma alternativa válida e evita a necessidade de escrever código de repetição personalizado, que pode ser difícil de implementar corretamente.</span><span class="sxs-lookup"><span data-stu-id="30594-767">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="30594-768">A Polly também fornece uma maneira de rastrear erros, para que você possa fazer novas tentativas.</span><span class="sxs-lookup"><span data-stu-id="30594-768">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>

<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[documentdb-api]: /azure/documentdb/documentdb-introduction
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx
