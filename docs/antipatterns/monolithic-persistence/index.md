---
title: "Antipadrão de persistência monolítica"
description: "Colocar todos os dados de um aplicativo em um único armazenamento de dados pode prejudicar o desempenho."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7f04b9f0805c281068b6b2edaf040683773e6f6e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="monolithic-persistence-antipattern"></a><span data-ttu-id="be974-103">Antipadrão de persistência monolítica</span><span class="sxs-lookup"><span data-stu-id="be974-103">Monolithic Persistence antipattern</span></span>

<span data-ttu-id="be974-104">Colocar todos os dados de um aplicativo em um único armazenamento de dados pode prejudicar o desempenho, pois leva à contenção de recursos ou porque o armazenamento de dados não é uma boa opção para alguns dos dados.</span><span class="sxs-lookup"><span data-stu-id="be974-104">Putting all of an application's data into a single data store can hurt performance, either because it leads to resource contention, or because the data store is not a good fit for some of the data.</span></span>

## <a name="problem-description"></a><span data-ttu-id="be974-105">Descrição do problema</span><span class="sxs-lookup"><span data-stu-id="be974-105">Problem description</span></span>

<span data-ttu-id="be974-106">Historicamente, os aplicativos geralmente usavam um único armazenamento de dados, independentemente dos diferentes tipos de dados que o aplicativo precisasse armazenar.</span><span class="sxs-lookup"><span data-stu-id="be974-106">Historically, applications have often used a single data store, regardless of the different types of data that the application might need to store.</span></span> <span data-ttu-id="be974-107">Geralmente, isso era feito para simplificar o design do aplicativo, ou para coincidir com o conjunto de habilidades existentes da equipe de desenvolvimento.</span><span class="sxs-lookup"><span data-stu-id="be974-107">Usually this was done to simplify the application design, or else to match the existing skill set of the development team.</span></span> 

<span data-ttu-id="be974-108">Sistemas baseados em nuvem modernos geralmente possuem requisitos funcionais e não funcionais adicionais e precisam armazenar muitos tipos heterogêneos de dados, como documentos, imagens, dados armazenados em cache, mensagens em fila, os logs de aplicativo e telemetria.</span><span class="sxs-lookup"><span data-stu-id="be974-108">Modern cloud-based systems often have additional functional and nonfunctional requirements, and need to store many heterogenous types of data, such as documents, images, cached data, queued messages, application logs, and telemetry.</span></span> <span data-ttu-id="be974-109">Seguir a abordagem tradicional e colocar todas essas informações no mesmo armazenamento de dados pode prejudicar o desempenho, por dois motivos principais:</span><span class="sxs-lookup"><span data-stu-id="be974-109">Following the traditional approach and putting all of this information into the same data store can hurt performance, for two main reasons:</span></span>

- <span data-ttu-id="be974-110">Armazenar e recuperar grandes quantidades de dados não relacionados no mesmo armazenamento de dados pode causar contenção, que por sua vez, leva a tempos de resposta mais lentos e falhas de conexão.</span><span class="sxs-lookup"><span data-stu-id="be974-110">Storing and retrieving large amounts of unrelated data in the same data store can cause contention, which in turn leads to slow response times and connection failures.</span></span>
- <span data-ttu-id="be974-111">Independentemente do armazenamento de dados escolhido, talvez ele não seja o mais adequado para todos os diferentes tipos de dados ou ele pode não ser otimizado para as operações o aplicativo executa.</span><span class="sxs-lookup"><span data-stu-id="be974-111">Whichever data store is chosen, it might not be the best fit for all of the different types of data, or it might not be optimized for the operations that the application performs.</span></span> 

<span data-ttu-id="be974-112">O exemplo a seguir mostra um controlador API Web ASP.NET que adiciona um novo registro em um banco de dados e também registra o resultado em um log.</span><span class="sxs-lookup"><span data-stu-id="be974-112">The following example shows an ASP.NET Web API controller that adds a new record to a database and also records the result to a log.</span></span> <span data-ttu-id="be974-113">O log é mantido no mesmo banco de dados que os dados de negócios.</span><span class="sxs-lookup"><span data-stu-id="be974-113">The log is held in the same database as the business data.</span></span> <span data-ttu-id="be974-114">Você pode encontrar o exemplo completo [aqui][sample-app].</span><span class="sxs-lookup"><span data-stu-id="be974-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class MonoController : ApiController
{
    private static readonly string ProductionDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        await DataAccess.LogAsync(ProductionDb, LogTableName);
        return Ok();
    }
}
```

<span data-ttu-id="be974-115">A taxa na qual os registros de log são gerados provavelmente afetará o desempenho das operações de negócios.</span><span class="sxs-lookup"><span data-stu-id="be974-115">The rate at which log records are generated will probably affect the performance of the business operations.</span></span> <span data-ttu-id="be974-116">E se outro componente, como um monitor de processo do aplicativo, lê e processa os dados de log regularmente, isso também pode afetar as operações de negócios.</span><span class="sxs-lookup"><span data-stu-id="be974-116">And if another component, such as an application process monitor, regularly reads and processes the log data, that can also affect the business operations.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="be974-117">Como corrigir o problema</span><span class="sxs-lookup"><span data-stu-id="be974-117">How to fix the problem</span></span>

<span data-ttu-id="be974-118">Dados separados de acordo com o uso.</span><span class="sxs-lookup"><span data-stu-id="be974-118">Separate data according to its use.</span></span> <span data-ttu-id="be974-119">Para cada conjunto de dados, selecione um armazenamento de dados que melhor descreva como o conjunto de dados será usado.</span><span class="sxs-lookup"><span data-stu-id="be974-119">For each data set, select a data store that best matches how that data set will be used.</span></span> <span data-ttu-id="be974-120">No exemplo anterior, o aplicativo deve fazer o registro em log em um armazenamento separado do banco de dados que contém dados de negócios:</span><span class="sxs-lookup"><span data-stu-id="be974-120">In the previous example, the application should be logging to a separate store from the database that holds business data:</span></span> 

```csharp
public class PolyController : ApiController
{
    private static readonly string ProductionDb = ...;
    private static readonly string LogDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        // Log to a different data store.
        await DataAccess.LogAsync(LogDb, LogTableName);
        return Ok();
    }
}
```

## <a name="considerations"></a><span data-ttu-id="be974-121">Considerações</span><span class="sxs-lookup"><span data-stu-id="be974-121">Considerations</span></span>

- <span data-ttu-id="be974-122">Separar dados conforma a maneira que eles são usados e acessados.</span><span class="sxs-lookup"><span data-stu-id="be974-122">Separate data by the way it is used and how it is accessed.</span></span> <span data-ttu-id="be974-123">Por exemplo, não armazene informações de log e dados de negócios no mesmo armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="be974-123">For example, don't store log information and business data in the same data store.</span></span> <span data-ttu-id="be974-124">Esses tipos de dados possuem requisitos e padrões de acesso diferentes.</span><span class="sxs-lookup"><span data-stu-id="be974-124">These types of data have significantly different requirements and patterns of access.</span></span> <span data-ttu-id="be974-125">Os registros de log são inerentemente sequenciais, enquanto os dados de negócios mais provavelmente exigirão acesso aleatório e geralmente são relacionais.</span><span class="sxs-lookup"><span data-stu-id="be974-125">Log records are inherently sequential, while business data is more likely to require random access, and is often relational.</span></span>

- <span data-ttu-id="be974-126">Considere o padrão de acesso a dados para cada tipo de dados.</span><span class="sxs-lookup"><span data-stu-id="be974-126">Consider the data access pattern for each type of data.</span></span> <span data-ttu-id="be974-127">Por exemplo, armazene relatórios e documentos formatados em um banco de dados do documento como [Cosmos DB][CosmosDB], mas use o [Cache Redis do Azure][Azure-cache] para armazenar em cache dados temporários.</span><span class="sxs-lookup"><span data-stu-id="be974-127">For example, store formatted reports and documents in a document database such as [Cosmos DB][CosmosDB], but use [Azure Redis Cache][Azure-cache] to cache temporary data.</span></span>

- <span data-ttu-id="be974-128">Se você seguir essas diretrizes e, mesmo assim, alcançar os limites do banco de dados, você precisará escalar verticalmente o banco de dados.</span><span class="sxs-lookup"><span data-stu-id="be974-128">If you follow this guidance but still reach the limits of the database, you may need to scale up the database.</span></span> <span data-ttu-id="be974-129">Também considere a possibilidade de escalar horizontalmente e particionar a carga entre os servidores de banco de dados.</span><span class="sxs-lookup"><span data-stu-id="be974-129">Also consider scaling horizontally and partitioning the load across database servers.</span></span> <span data-ttu-id="be974-130">No entanto, pode ser que o particionamento exija um novo design do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="be974-130">However, partitioning may require redesigning the application.</span></span> <span data-ttu-id="be974-131">Para saber mais, consulte [Particionamento de dados][DataPartitioningGuidance].</span><span class="sxs-lookup"><span data-stu-id="be974-131">For more information, see [Data partitioning][DataPartitioningGuidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="be974-132">Como detectar o problema</span><span class="sxs-lookup"><span data-stu-id="be974-132">How to detect the problem</span></span>

<span data-ttu-id="be974-133">O sistema provavelmente ficará bem mais lento e eventualmente falhará, já que o sistema fica sem recursos, como conexões de banco de dados, por exemplo.</span><span class="sxs-lookup"><span data-stu-id="be974-133">The system will likely slow down dramatically and eventually fail, as the system runs out of resources such as database connections.</span></span>

<span data-ttu-id="be974-134">Você pode executar as etapas a seguir para ajudar a identificar a causa.</span><span class="sxs-lookup"><span data-stu-id="be974-134">You can perform the following steps to help identify the cause.</span></span>

1. <span data-ttu-id="be974-135">Instrumentar o sistema para registrar as principais estatísticas de desempenho.</span><span class="sxs-lookup"><span data-stu-id="be974-135">Instrument the system to record the key performance statistics.</span></span> <span data-ttu-id="be974-136">Capture informações de tempo para cada operação, bem como os pontos onde o aplicativo lê e grava dados.</span><span class="sxs-lookup"><span data-stu-id="be974-136">Capture timing information for each operation, as well as the points where the application reads and writes data.</span></span>
1. <span data-ttu-id="be974-137">Se possível, monitore o sistema em execução por alguns dias em um ambiente de produção para obter uma visão real de como o sistema é usado.</span><span class="sxs-lookup"><span data-stu-id="be974-137">If possible, monitor the system running for a few days in a production environment to get a real-world view of how the system is used.</span></span> <span data-ttu-id="be974-138">Se isso não for possível, execute testes de carga com script com um volume realista de usuários virtuais executando uma série normal de operações.</span><span class="sxs-lookup"><span data-stu-id="be974-138">If this is not possible, run scripted load tests with a realistic volume of virtual users performing a typical series of operations.</span></span>
2. <span data-ttu-id="be974-139">Use os dados de telemetria para identificar os períodos de baixo desempenho.</span><span class="sxs-lookup"><span data-stu-id="be974-139">Use the telemetry data to identify periods of poor performance.</span></span>
3. <span data-ttu-id="be974-140">Identifique quais armazenamentos de dados foram acessados durante esses períodos.</span><span class="sxs-lookup"><span data-stu-id="be974-140">Identify which data stores were accessed during those periods.</span></span>
4. <span data-ttu-id="be974-141">Identifique os recursos de armazenamento de dados que podem estar com problemas de contenção.</span><span class="sxs-lookup"><span data-stu-id="be974-141">Identify data storage resources that might be experiencing contention.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="be974-142">Diagnóstico de exemplo</span><span class="sxs-lookup"><span data-stu-id="be974-142">Example diagnosis</span></span>

<span data-ttu-id="be974-143">As seções a seguir aplicam essas etapas ao aplicativo de exemplo descrito anteriormente.</span><span class="sxs-lookup"><span data-stu-id="be974-143">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-and-monitor-the-system"></a><span data-ttu-id="be974-144">Instrumentar e monitorar o sistema</span><span class="sxs-lookup"><span data-stu-id="be974-144">Instrument and monitor the system</span></span>

<span data-ttu-id="be974-145">O gráfico a seguir mostra os resultados de teste de carga para o aplicativo de exemplo descrito anteriormente.</span><span class="sxs-lookup"><span data-stu-id="be974-145">The following graph shows the results of load testing the sample application described earlier.</span></span> <span data-ttu-id="be974-146">O teste usou uma carga de até 1.000 usuários simultâneos.</span><span class="sxs-lookup"><span data-stu-id="be974-146">The test used a step load of up to 1000 concurrent users.</span></span>

![Resultados de desempenho do teste de carga para o controlador baseado em SQL][MonolithicScenarioLoadTest]

<span data-ttu-id="be974-148">À medida que a carga aumenta para 700 usuários, também aumenta a taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="be974-148">As the load increases to 700 users, so does the throughput.</span></span> <span data-ttu-id="be974-149">Mas nesse ponto, os níveis de taxa de transferência ficam extremamente elevados e o sistema parecem estar em execução em sua capacidade máxima.</span><span class="sxs-lookup"><span data-stu-id="be974-149">But at that point, throughput levels off, and the system appears to be running at its maximum capacity.</span></span> <span data-ttu-id="be974-150">A média de resposta aumenta gradualmente junto com a carga de usuário, mostrando que o sistema não pode atender à demanda.</span><span class="sxs-lookup"><span data-stu-id="be974-150">The average response gradually increases with user load, showing that the system can't keep up with demand.</span></span>

### <a name="identify-periods-of-poor-performance"></a><span data-ttu-id="be974-151">Identificar períodos de baixo desempenho</span><span class="sxs-lookup"><span data-stu-id="be974-151">Identify periods of poor performance</span></span>

<span data-ttu-id="be974-152">Se você estiver monitorando o sistema de produção, você poderá observar padrões.</span><span class="sxs-lookup"><span data-stu-id="be974-152">If you are monitoring the production system, you might notice patterns.</span></span> <span data-ttu-id="be974-153">Por exemplo, os tempos de resposta podem diminuir significativamente na mesma hora todos os dias.</span><span class="sxs-lookup"><span data-stu-id="be974-153">For example, response times might drop off significantly at the same time each day.</span></span> <span data-ttu-id="be974-154">Isso pode ser causado por uma carga de trabalho normal ou um trabalho em lotes agendado ou simplesmente porque o sistema tem mais usuários em determinados momentos.</span><span class="sxs-lookup"><span data-stu-id="be974-154">This could be caused by a regular workload or scheduled batch job, or just because the system has more users at certain times.</span></span> <span data-ttu-id="be974-155">Você deve se concentrar nos dados de telemetria para esses eventos.</span><span class="sxs-lookup"><span data-stu-id="be974-155">You should focus on the telemetry data for these events.</span></span>

<span data-ttu-id="be974-156">Procure as correlações entre tempos de resposta maiores e maior atividade de banco de dados ou E/S para recursos compartilhados.</span><span class="sxs-lookup"><span data-stu-id="be974-156">Look for correlations between increased response times and increased database activity or I/O to shared resources.</span></span> <span data-ttu-id="be974-157">Se houver correlações, isso significa que o banco de dados pode ser um gargalo.</span><span class="sxs-lookup"><span data-stu-id="be974-157">If there are correlations, it means the database might be a bottleneck.</span></span>

### <a name="identify-which-data-stores-are-accessed-during-those-periods"></a><span data-ttu-id="be974-158">Identifique quais armazenamentos de dados são acessados durante esses períodos</span><span class="sxs-lookup"><span data-stu-id="be974-158">Identify which data stores are accessed during those periods</span></span>

<span data-ttu-id="be974-159">O gráfico a seguir mostra a utilização de unidades da taxa de transferência do banco de dados (DTU) durante o teste de carga.</span><span class="sxs-lookup"><span data-stu-id="be974-159">The next graph shows the utilization of database throughput units (DTU) during the load test.</span></span> <span data-ttu-id="be974-160">(Uma DTU é uma medida da capacidade disponível e é uma combinação de utilização da CPU, alocação de memória e a taxa de E/S). A utilização de DTUs rapidamente atingiu 100%.</span><span class="sxs-lookup"><span data-stu-id="be974-160">(A DTU is a measure of available capacity, and is a combination of CPU utilization, memory allocation, I/O rate.) Utilization of DTUs quickly reached 100%.</span></span> <span data-ttu-id="be974-161">Esse é aproximadamente o ponto de pico da taxa de transferência no gráfico anterior.</span><span class="sxs-lookup"><span data-stu-id="be974-161">This is roughly the point where throughput peaked in the previous graph.</span></span> <span data-ttu-id="be974-162">A utilização do banco de dados permanece muito alta, até que o teste seja concluído.</span><span class="sxs-lookup"><span data-stu-id="be974-162">Database utilization remained very high until the test finished.</span></span> <span data-ttu-id="be974-163">Há uma pequena queda perto do final, que pode ser causada pela limitação, concorrência por conexões de banco de dados ou outros fatores.</span><span class="sxs-lookup"><span data-stu-id="be974-163">There is a slight drop toward the end, which could be caused by throttling, competition for database connections, or other factors.</span></span>

![O monitor de banco de dados no portal clássico do Azure mostrando a utilização de recursos do banco de dados][MonolithicDatabaseUtilization]

### <a name="examine-the-telemetry-for-the-data-stores"></a><span data-ttu-id="be974-165">Examinar a telemetria para os armazenamentos de dados</span><span class="sxs-lookup"><span data-stu-id="be974-165">Examine the telemetry for the data stores</span></span>

<span data-ttu-id="be974-166">Instrumente os armazenamentos de dados para capturar os detalhes de baixo nível da atividade.</span><span class="sxs-lookup"><span data-stu-id="be974-166">Instrument the data stores to capture the low-level details of the activity.</span></span> <span data-ttu-id="be974-167">No aplicativo de exemplo, as estatísticas de acesso de dados mostraram um alto volume de operações de inserção executadas na tabela `PurchaseOrderHeader` e na tabela `MonoLog`.</span><span class="sxs-lookup"><span data-stu-id="be974-167">In the sample application, the data access statistics showed a high volume of insert operations performed against both the `PurchaseOrderHeader` table and the `MonoLog` table.</span></span> 

![As estatísticas de acesso a dados para o aplicativo de exemplo][MonolithicDataAccessStats]

### <a name="identify-resource-contention"></a><span data-ttu-id="be974-169">Identificar contenção de recurso</span><span class="sxs-lookup"><span data-stu-id="be974-169">Identify resource contention</span></span>

<span data-ttu-id="be974-170">Neste ponto, você pode examinar o código-fonte, concentrando-se nos pontos onde os recursos em contenção são acessados pelo aplicativo.</span><span class="sxs-lookup"><span data-stu-id="be974-170">At this point, you can review the source code, focusing on the points where contended resources are accessed by the application.</span></span> <span data-ttu-id="be974-171">Procure situações, como:</span><span class="sxs-lookup"><span data-stu-id="be974-171">Look for situations such as:</span></span>

- <span data-ttu-id="be974-172">Dados que são logicamente separados sendo gravados no mesmo armazenamento.</span><span class="sxs-lookup"><span data-stu-id="be974-172">Data that is logically separate being written to the same store.</span></span> <span data-ttu-id="be974-173">Dados como mensagens em fila, relatórios e logs não devem ser mantidos no mesmo banco de dados que as informações de negócios.</span><span class="sxs-lookup"><span data-stu-id="be974-173">Data such as logs, reports, and queued messages should not be held in the same database as business information.</span></span>
- <span data-ttu-id="be974-174">Uma incompatibilidade entre a opção de armazenamento de dados e o tipo de dados, como blobs grandes ou documentos XML em um banco de dados relacional.</span><span class="sxs-lookup"><span data-stu-id="be974-174">A mismatch between the choice of data store and the type of data, such as large blobs or XML documents in a relational database.</span></span>
- <span data-ttu-id="be974-175">Dados com padrões de uso significativamente diferentes que compartilham o mesmo armazenamento, como dados de gravação alta/leitura baixa sendo armazenados com dados de gravação baixa/leitura alta.</span><span class="sxs-lookup"><span data-stu-id="be974-175">Data with significantly different usage patterns that share the same store, such as high-write/low-read data being stored with low-write/high-read data.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="be974-176">Implementar a solução e verificar o resultado</span><span class="sxs-lookup"><span data-stu-id="be974-176">Implement the solution and verify the result</span></span>

<span data-ttu-id="be974-177">O aplicativo foi alterado para gravar logs em um armazenamento de dados separado.</span><span class="sxs-lookup"><span data-stu-id="be974-177">The application was changed to write logs to a separate data store.</span></span> <span data-ttu-id="be974-178">Aqui estão os resultados do teste de carga:</span><span class="sxs-lookup"><span data-stu-id="be974-178">Here are the load test results:</span></span>

![Resultados de desempenho de teste de carga usando o controlador Polyglot][PolyglotScenarioLoadTest]

<span data-ttu-id="be974-180">O padrão de taxa de transferência é semelhante ao gráfico anterior, mas o ponto de pico do desempenho é de aproximadamente 500 solicitações por segundo mais alto.</span><span class="sxs-lookup"><span data-stu-id="be974-180">The pattern of throughput is similar to the earlier graph, but the point at which performance peaks is approximately 500 requests per second higher.</span></span> <span data-ttu-id="be974-181">O tempo médio de resposta é um pouco menor.</span><span class="sxs-lookup"><span data-stu-id="be974-181">The average response time is marginally lower.</span></span> <span data-ttu-id="be974-182">No entanto, essas estatísticas não mostram tudo.</span><span class="sxs-lookup"><span data-stu-id="be974-182">However, these statistics don't tell the full story.</span></span> <span data-ttu-id="be974-183">A telemetria para o banco de dados de negócios mostra que a utilização de DTU atinge o pico em torno de 75%, em vez de 100%.</span><span class="sxs-lookup"><span data-stu-id="be974-183">Telemetry for the business database shows that DTU utilization peaks at around 75%, rather than 100%.</span></span>

![O monitor do banco de dados no portal clássico do Azure mostrando a utilização de recursos do banco de dados no cenário do Polyglot][PolyglotDatabaseUtilization]

<span data-ttu-id="be974-185">Da mesma forma, a utilização máxima de DTU do banco de dados de log atinge somente cerca de 70%.</span><span class="sxs-lookup"><span data-stu-id="be974-185">Similarly, the maximum DTU utilization of the log database only reaches about 70%.</span></span> <span data-ttu-id="be974-186">Os bancos de dados não são mais o fator limitante no desempenho do sistema.</span><span class="sxs-lookup"><span data-stu-id="be974-186">The databases are no longer the limiting factor in the performance of the system.</span></span>

![O monitor do banco de dados no portal clássico do Azure mostrando a utilização de recursos do banco de dados de log no cenário do Polyglot][LogDatabaseUtilization]


## <a name="related-resources"></a><span data-ttu-id="be974-188">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="be974-188">Related resources</span></span>

- <span data-ttu-id="be974-189">[Escolha o armazenamento de dados correto][data-store-overview]</span><span class="sxs-lookup"><span data-stu-id="be974-189">[Choose the right data store][data-store-overview]</span></span>
- <span data-ttu-id="be974-190">[Critérios para escolher um armazenamento de dados][data-store-comparison]</span><span class="sxs-lookup"><span data-stu-id="be974-190">[Criteria for choosing a data store][data-store-comparison]</span></span>
- <span data-ttu-id="be974-191">[Acesso de dados para soluções altamente escalonáveis: Usando persistência no SQL, NoSQL e Polyglot][Data-Access-Guide]</span><span class="sxs-lookup"><span data-stu-id="be974-191">[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence][Data-Access-Guide]</span></span>
- <span data-ttu-id="be974-192">[Particionamento de dados][DataPartitioningGuidance]</span><span class="sxs-lookup"><span data-stu-id="be974-192">[Data partitioning][DataPartitioningGuidance]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/MonolithicPersistence
[CosmosDB]: http://azure.microsoft.com/services/cosmos-db/
[Azure-cache]: /azure/redis-cache/
[Data-Access-Guide]: https://msdn.microsoft.com/library/dn271399.aspx
[DataPartitioningGuidance]: ../../best-practices/data-partitioning.md
[data-store-overview]: ../../guide/technology-choices/data-store-overview.md
[data-store-comparison]: ../../guide/technology-choices/data-store-comparison.md

[MonolithicScenarioLoadTest]: _images/MonolithicScenarioLoadTest.jpg
[MonolithicDatabaseUtilization]: _images/MonolithicDatabaseUtilization.jpg
[MonolithicDataAccessStats]: _images/MonolithicDataAccessStats.jpg
[PolyglotScenarioLoadTest]: _images/PolyglotScenarioLoadTest.jpg
[PolyglotDatabaseUtilization]: _images/PolyglotDatabaseUtilization.jpg
[LogDatabaseUtilization]: _images/LogDatabaseUtilization.jpg
