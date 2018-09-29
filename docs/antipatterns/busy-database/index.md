---
title: Antipadrão de Banco de Dados Ocupado
description: Descarregamento de processamento em um servidor de banco de dados pode causar problemas de desempenho e escalabilidade.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: a14a350aefc1801ae08cb4a8d0eb3d5b248c92bf
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428900"
---
# <a name="busy-database-antipattern"></a><span data-ttu-id="96d0f-103">Antipadrão de Banco de Dados Ocupado</span><span class="sxs-lookup"><span data-stu-id="96d0f-103">Busy Database antipattern</span></span>

<span data-ttu-id="96d0f-104">O processamento de descarregamento para um servidor de banco de dados pode fazer com que ele gaste uma proporção significativa de tempo de execução do código, em vez de responder às solicitações para armazenar e recuperar dados.</span><span class="sxs-lookup"><span data-stu-id="96d0f-104">Offloading processing to a database server can cause it to spend a significant proportion of time running code, rather than responding to requests to store and retrieve data.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="96d0f-105">Descrição do problema</span><span class="sxs-lookup"><span data-stu-id="96d0f-105">Problem description</span></span>

<span data-ttu-id="96d0f-106">Muitos sistemas de banco de dados podem executar o código.</span><span class="sxs-lookup"><span data-stu-id="96d0f-106">Many database systems can run code.</span></span> <span data-ttu-id="96d0f-107">Exemplos incluem procedimentos armazenados e gatilhos.</span><span class="sxs-lookup"><span data-stu-id="96d0f-107">Examples include stored procedures and triggers.</span></span> <span data-ttu-id="96d0f-108">Geralmente, é mais eficiente para executar esse processamento aproximado para os dados, em vez de transmitir os dados para um aplicativo cliente para processamento.</span><span class="sxs-lookup"><span data-stu-id="96d0f-108">Often, it's more efficient to perform this processing close to the data, rather than transmitting the data to a client application for processing.</span></span> <span data-ttu-id="96d0f-109">No entanto, uso excessivo esses recursos pode prejudicar o desempenho, por vários motivos:</span><span class="sxs-lookup"><span data-stu-id="96d0f-109">However, overusing these features can hurt performance, for several reasons:</span></span>

- <span data-ttu-id="96d0f-110">O servidor de banco de dados pode levar muito tempo de processamento, em vez de aceitar novas solicitações de cliente e buscar dados.</span><span class="sxs-lookup"><span data-stu-id="96d0f-110">The database server may spend too much time processing, rather than accepting new client requests and fetching data.</span></span>
- <span data-ttu-id="96d0f-111">Um banco de dados geralmente é um recurso compartilhado, ele pode se tornar um afunilamento durante períodos de alto uso.</span><span class="sxs-lookup"><span data-stu-id="96d0f-111">A database is usually a shared resource, so it can become a bottleneck during periods of high use.</span></span>
- <span data-ttu-id="96d0f-112">Os custos de tempo de execução podem ser excessivos se o armazenamento de dados é monitorado.</span><span class="sxs-lookup"><span data-stu-id="96d0f-112">Runtime costs may be excessive if the data store is metered.</span></span> <span data-ttu-id="96d0f-113">Que é especialmente verdadeiro para serviços de banco de dados gerenciados.</span><span class="sxs-lookup"><span data-stu-id="96d0f-113">That's particularly true of managed database services.</span></span> <span data-ttu-id="96d0f-114">Por exemplo, o Banco de Dados SQL do Azure cobra por [Unidades de Transação do Banco de Dados][dtu] (DTUs).</span><span class="sxs-lookup"><span data-stu-id="96d0f-114">For example, Azure SQL Database charges for [Database Transaction Units][dtu] (DTUs).</span></span>
- <span data-ttu-id="96d0f-115">Bancos de dados têm capacidade finita de expansão e não é comum para dimensionar um banco de dados horizontalmente.</span><span class="sxs-lookup"><span data-stu-id="96d0f-115">Databases have finite capacity to scale up, and it's not trivial to scale a database horizontally.</span></span> <span data-ttu-id="96d0f-116">Portanto, talvez seja melhor mover o processamento em um recurso de computação, como um aplicativo do Serviço de Aplicativo ou de VM, que pode expandir facilmente.</span><span class="sxs-lookup"><span data-stu-id="96d0f-116">Therefore, it may be better to move processing into a compute resource, such as a VM or App Service app, that can easily scale out.</span></span>

<span data-ttu-id="96d0f-117">Esse antipadrão geralmente ocorre porque:</span><span class="sxs-lookup"><span data-stu-id="96d0f-117">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="96d0f-118">O banco de dados é exibido como um serviço em vez de um armazenamento.</span><span class="sxs-lookup"><span data-stu-id="96d0f-118">The database is viewed as a service rather than a repository.</span></span> <span data-ttu-id="96d0f-119">Um aplicativo pode usar o servidor de banco de dados para formato de dados (por exemplo, a conversão em XML), manipular dados de cadeia de caracteres ou executar cálculos complexos.</span><span class="sxs-lookup"><span data-stu-id="96d0f-119">An application might use the database server to format data (for example, converting to XML), manipulate string data, or perform complex calculations.</span></span>
- <span data-ttu-id="96d0f-120">Os desenvolvedores tentam escrever consultas cujos resultados podem ser exibidos diretamente aos usuários.</span><span class="sxs-lookup"><span data-stu-id="96d0f-120">Developers try to write queries whose results can be displayed directly to users.</span></span> <span data-ttu-id="96d0f-121">Por exemplo, uma consulta pode combinar campos ou formatar datas, horas e moedas de acordo com a localidade.</span><span class="sxs-lookup"><span data-stu-id="96d0f-121">For example a query might combine fields, or format dates, times, and currency according to locale.</span></span>
- <span data-ttu-id="96d0f-122">Os desenvolvedores estão tentando corrigir o antipadrão [Busca Incorreta][ExtraneousFetching] durante o envio por push de cálculos para o banco de dados.</span><span class="sxs-lookup"><span data-stu-id="96d0f-122">Developers are trying to correct the [Extraneous Fetching][ExtraneousFetching] antipattern by pushing computations to the database.</span></span>
- <span data-ttu-id="96d0f-123">Os procedimentos armazenados são usados para encapsular a lógica de negócios, talvez porque eles são considerados mais fáceis de manter e atualizar.</span><span class="sxs-lookup"><span data-stu-id="96d0f-123">Stored procedures are used to encapsulate business logic, perhaps because they are considered easier to maintain and update.</span></span>

<span data-ttu-id="96d0f-124">O exemplo a seguir recupera os 20 pedidos mais valiosos para uma região de vendas especificada e formata os resultados como XML.</span><span class="sxs-lookup"><span data-stu-id="96d0f-124">The following example retrieves the 20 most valuable orders for a specified sales territory and formats the results as XML.</span></span>
<span data-ttu-id="96d0f-125">Ele usa funções Transact-SQL para analisar os dados e converter os resultados em XML.</span><span class="sxs-lookup"><span data-stu-id="96d0f-125">It uses Transact-SQL functions to parse the data and convert the results to XML.</span></span> <span data-ttu-id="96d0f-126">Você pode encontrar o exemplo completo [aqui][sample-app].</span><span class="sxs-lookup"><span data-stu-id="96d0f-126">You can find the complete sample [here][sample-app].</span></span>

```SQL
SELECT TOP 20
  soh.[SalesOrderNumber]  AS '@OrderNumber',
  soh.[Status]            AS '@Status',
  soh.[ShipDate]          AS '@ShipDate',
  YEAR(soh.[OrderDate])   AS '@OrderDateYear',
  MONTH(soh.[OrderDate])  AS '@OrderDateMonth',
  soh.[DueDate]           AS '@DueDate',
  FORMAT(ROUND(soh.[SubTotal],2),'C')
                          AS '@SubTotal',
  FORMAT(ROUND(soh.[TaxAmt],2),'C')
                          AS '@TaxAmt',
  FORMAT(ROUND(soh.[TotalDue],2),'C')
                          AS '@TotalDue',
  CASE WHEN soh.[TotalDue] > 5000 THEN 'Y' ELSE 'N' END
                          AS '@ReviewRequired',
  (
  SELECT
    c.[AccountNumber]     AS '@AccountNumber',
    UPPER(LTRIM(RTRIM(REPLACE(
    CONCAT( p.[Title], ' ', p.[FirstName], ' ', p.[MiddleName], ' ', p.[LastName], ' ', p.[Suffix]),
    '  ', ' '))))         AS '@FullName'
  FROM [Sales].[Customer] c
    INNER JOIN [Person].[Person] p
  ON c.[PersonID] = p.[BusinessEntityID]
  WHERE c.[CustomerID] = soh.[CustomerID]
  FOR XML PATH ('Customer'), TYPE
  ),

  (
  SELECT
    sod.[OrderQty]      AS '@Quantity',
    FORMAT(sod.[UnitPrice],'C')
                        AS '@UnitPrice',
    FORMAT(ROUND(sod.[LineTotal],2),'C')
                        AS '@LineTotal',
    sod.[ProductID]     AS '@ProductId',
    CASE WHEN (sod.[ProductID] >= 710) AND (sod.[ProductID] <= 720) AND (sod.[OrderQty] >= 5) THEN 'Y' ELSE 'N' END
                        AS '@InventoryCheckRequired'

  FROM [Sales].[SalesOrderDetail] sod
  WHERE sod.[SalesOrderID] = soh.[SalesOrderID]
  ORDER BY sod.[SalesOrderDetailID]
  FOR XML PATH ('LineItem'), TYPE, ROOT('OrderLineItems')
  )

FROM [Sales].[SalesOrderHeader] soh
WHERE soh.[TerritoryId] = @TerritoryId
ORDER BY soh.[TotalDue] DESC
FOR XML PATH ('Order'), ROOT('Orders')
```

<span data-ttu-id="96d0f-127">Claramente, essa é uma consulta complexa.</span><span class="sxs-lookup"><span data-stu-id="96d0f-127">Clearly, this is complex query.</span></span> <span data-ttu-id="96d0f-128">Como você verá mais tarde, na verdade, para usar os recursos de processamento significativo no servidor de banco de dados.</span><span class="sxs-lookup"><span data-stu-id="96d0f-128">As we'll see later, it turns out to use significant processing resources on the database server.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="96d0f-129">Como corrigir o problema</span><span class="sxs-lookup"><span data-stu-id="96d0f-129">How to fix the problem</span></span>

<span data-ttu-id="96d0f-130">Mova o processamento do servidor de banco de dados em outros níveis de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="96d0f-130">Move processing from the database server into other application tiers.</span></span> <span data-ttu-id="96d0f-131">Idealmente, você deve limitar o banco de dados para executar operações de acesso de dados, usando apenas os recursos para os quais o banco de dados é otimizado, como a agregação em um RDBMS.</span><span class="sxs-lookup"><span data-stu-id="96d0f-131">Ideally, you should limit the database to performing data access operations, using only the capabilities that the database is optimized for, such as aggregation in an RDBMS.</span></span>

<span data-ttu-id="96d0f-132">Por exemplo, o código Transact-SQL anterior pode ser substituído por uma instrução que simplesmente recupera os dados a serem processados.</span><span class="sxs-lookup"><span data-stu-id="96d0f-132">For example, the previous Transact-SQL code can be replaced with a statement that simply retrieves the data to be processed.</span></span>

```SQL
SELECT
soh.[SalesOrderNumber]  AS [OrderNumber],
soh.[Status]            AS [Status],
soh.[OrderDate]         AS [OrderDate],
soh.[DueDate]           AS [DueDate],
soh.[ShipDate]          AS [ShipDate],
soh.[SubTotal]          AS [SubTotal],
soh.[TaxAmt]            AS [TaxAmt],
soh.[TotalDue]          AS [TotalDue],
c.[AccountNumber]       AS [AccountNumber],
p.[Title]               AS [CustomerTitle],
p.[FirstName]           AS [CustomerFirstName],
p.[MiddleName]          AS [CustomerMiddleName],
p.[LastName]            AS [CustomerLastName],
p.[Suffix]              AS [CustomerSuffix],
sod.[OrderQty]          AS [Quantity],
sod.[UnitPrice]         AS [UnitPrice],
sod.[LineTotal]         AS [LineTotal],
sod.[ProductID]         AS [ProductId]
FROM [Sales].[SalesOrderHeader] soh
INNER JOIN [Sales].[Customer] c ON soh.[CustomerID] = c.[CustomerID]
INNER JOIN [Person].[Person] p ON c.[PersonID] = p.[BusinessEntityID]
INNER JOIN [Sales].[SalesOrderDetail] sod ON soh.[SalesOrderID] = sod.[SalesOrderID]
WHERE soh.[TerritoryId] = @TerritoryId
AND soh.[SalesOrderId] IN (
    SELECT TOP 20 SalesOrderId
    FROM [Sales].[SalesOrderHeader] soh
    WHERE soh.[TerritoryId] = @TerritoryId
    ORDER BY soh.[TotalDue] DESC)
ORDER BY soh.[TotalDue] DESC, sod.[SalesOrderDetailID]
```

<span data-ttu-id="96d0f-133">Em seguida, o aplicativo usa o .NET Framework `System.Xml.Linq` APIs para formatar os resultados como XML.</span><span class="sxs-lookup"><span data-stu-id="96d0f-133">The application then uses the .NET Framework `System.Xml.Linq` APIs to format the results as XML.</span></span>

```csharp
// Create a new SqlCommand to run the Transact-SQL query
using (var command = new SqlCommand(...))
{
    command.Parameters.AddWithValue("@TerritoryId", id);

    // Run the query and create the initial XML document
    using (var reader = await command.ExecuteReaderAsync())
    {
        var lastOrderNumber = string.Empty;
        var doc = new XDocument();
        var orders = new XElement("Orders");
        doc.Add(orders);

        XElement lineItems = null;
        // Fetch each row in turn, format the results as XML, and add them to the XML document
        while (await reader.ReadAsync())
        {
            var orderNumber = reader["OrderNumber"].ToString();
            if (orderNumber != lastOrderNumber)
            {
                lastOrderNumber = orderNumber;

                var order = new XElement("Order");
                orders.Add(order);
                var customer = new XElement("Customer");
                lineItems = new XElement("OrderLineItems");
                order.Add(customer, lineItems);

                var orderDate = (DateTime)reader["OrderDate"];
                var totalDue = (Decimal)reader["TotalDue"];
                var reviewRequired = totalDue > 5000 ? 'Y' : 'N';

                order.Add(
                    new XAttribute("OrderNumber", orderNumber),
                    new XAttribute("Status", reader["Status"]),
                    new XAttribute("ShipDate", reader["ShipDate"]),
                    ... // More attributes, not shown.

                    var fullName = string.Join(" ",
                        reader["CustomerTitle"],
                        reader["CustomerFirstName"],
                        reader["CustomerMiddleName"],
                        reader["CustomerLastName"],
                        reader["CustomerSuffix"]
                    )
                   .Replace("  ", " ") //remove double spaces
                   .Trim()
                   .ToUpper();

               customer.Add(
                    new XAttribute("AccountNumber", reader["AccountNumber"]),
                    new XAttribute("FullName", fullName));
            }

            var productId = (int)reader["ProductID"];
            var quantity = (short)reader["Quantity"];
            var inventoryCheckRequired = (productId >= 710 && productId <= 720 && quantity >= 5) ? 'Y' : 'N';

            lineItems.Add(
                new XElement("LineItem",
                    new XAttribute("Quantity", quantity),
                    new XAttribute("UnitPrice", ((Decimal)reader["UnitPrice"]).ToString("C")),
                    new XAttribute("LineTotal", RoundAndFormat(reader["LineTotal"])),
                    new XAttribute("ProductId", productId),
                    new XAttribute("InventoryCheckRequired", inventoryCheckRequired)
                ));
        }
        // Match the exact formatting of the XML returned from SQL
        var xml = doc
            .ToString(SaveOptions.DisableFormatting)
            .Replace(" />", "/>");
    }
}
```

> [!NOTE]
> <span data-ttu-id="96d0f-134">Esse código é um pouco complexo.</span><span class="sxs-lookup"><span data-stu-id="96d0f-134">This code is somewhat complex.</span></span> <span data-ttu-id="96d0f-135">Para um novo aplicativo, talvez você prefira usar uma biblioteca de serialização.</span><span class="sxs-lookup"><span data-stu-id="96d0f-135">For a new application, you might prefer to use a serialization library.</span></span> <span data-ttu-id="96d0f-136">No entanto, a suposição aqui é que a equipe de desenvolvimento esteja refatorando um aplicativo existente e, portanto, o método deve retornar o mesmo formato exato do que o código original.</span><span class="sxs-lookup"><span data-stu-id="96d0f-136">However, the assumption here is that the development team is refactoring an existing application, so the method needs to return the exact same format as the original code.</span></span>

## <a name="considerations"></a><span data-ttu-id="96d0f-137">Considerações</span><span class="sxs-lookup"><span data-stu-id="96d0f-137">Considerations</span></span>

- <span data-ttu-id="96d0f-138">Muitos sistemas de banco de dados são altamente otimizados para executar determinados tipos de processamento de dados, como calcular valores de agregação ao longo de grandes conjuntos de dados.</span><span class="sxs-lookup"><span data-stu-id="96d0f-138">Many database systems are highly optimized to perform certain types of data processing, such as calculating aggregate values over large datasets.</span></span> <span data-ttu-id="96d0f-139">Não mova esses tipos de processamento fora do banco de dados.</span><span class="sxs-lookup"><span data-stu-id="96d0f-139">Don't move those types of processing out of the database.</span></span>

- <span data-ttu-id="96d0f-140">Não realoque o processamento se isso fizer o banco de dados transferir mais dados pela rede.</span><span class="sxs-lookup"><span data-stu-id="96d0f-140">Do not relocate processing if doing so causes the database to transfer far more data over the network.</span></span> <span data-ttu-id="96d0f-141">Veja o [Antipadrão de Busca incorreta][ExtraneousFetching].</span><span class="sxs-lookup"><span data-stu-id="96d0f-141">See the [Extraneous Fetching antipattern][ExtraneousFetching].</span></span>

- <span data-ttu-id="96d0f-142">Se você mover o processamento para uma camada de aplicativo, talvez seja necessário expandir a camada para lidar com o trabalho adicional.</span><span class="sxs-lookup"><span data-stu-id="96d0f-142">If you move processing to an application tier, that tier may need to scale out to handle the additional work.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="96d0f-143">Como detectar o problema</span><span class="sxs-lookup"><span data-stu-id="96d0f-143">How to detect the problem</span></span>

<span data-ttu-id="96d0f-144">Os sintomas de um banco de dados ocupado incluem um declínio desproporcional de tempos de resposta e a taxa de transferência em operações que acessam o banco de dados.</span><span class="sxs-lookup"><span data-stu-id="96d0f-144">Symptoms of a busy database include a disproportionate decline in throughput and response times in operations that access the database.</span></span> 

<span data-ttu-id="96d0f-145">Você pode executar as etapas a seguir para ajudar a identificar o problema:</span><span class="sxs-lookup"><span data-stu-id="96d0f-145">You can perform the following steps to help identify this problem:</span></span> 

1. <span data-ttu-id="96d0f-146">Use o monitoramento de desempenho para identificar o tempo que o sistema de produção passa executando a atividade de banco de dados.</span><span class="sxs-lookup"><span data-stu-id="96d0f-146">Use performance monitoring to identify how much time the production system spends performing database activity.</span></span>

2. <span data-ttu-id="96d0f-147">Examine o trabalho executado pelo banco de dados durante esses períodos.</span><span class="sxs-lookup"><span data-stu-id="96d0f-147">Examine the work performed by the database during these periods.</span></span>

3. <span data-ttu-id="96d0f-148">Se você suspeitar de que determinadas operações podem causar muita atividade de banco de dados, execute em um ambiente controlado de teste de carga.</span><span class="sxs-lookup"><span data-stu-id="96d0f-148">If you suspect that particular operations might cause too much database activity, perform load testing in a controlled environment.</span></span> <span data-ttu-id="96d0f-149">Cada teste deve ser executado uma combinação das operações suspeitas com uma carga de usuário de variável.</span><span class="sxs-lookup"><span data-stu-id="96d0f-149">Each test should run a mixture of the suspect operations with a variable user load.</span></span> <span data-ttu-id="96d0f-150">Examine a telemetria dos testes de carga para observar como o banco de dados é usado.</span><span class="sxs-lookup"><span data-stu-id="96d0f-150">Examine the telemetry from the load tests to observe how the database is used.</span></span>

4. <span data-ttu-id="96d0f-151">Se a atividade de banco de dados revela o processamento significativo, mas pouco tráfego de dados, examine o código-fonte para determinar se o processamento pode ser mais bem executado em outro lugar.</span><span class="sxs-lookup"><span data-stu-id="96d0f-151">If the database activity reveals significant processing but little data traffic, review the source code to determine whether the processing can better be performed elsewhere.</span></span>

<span data-ttu-id="96d0f-152">Se o volume de atividade do banco de dados é baixo ou tempos de resposta são relativamente rápidos, um banco de dados ocupado é improvável de ser um problema de desempenho.</span><span class="sxs-lookup"><span data-stu-id="96d0f-152">If the volume of database activity is low or response times are relatively fast, then a busy database is unlikely to be a performance problem.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="96d0f-153">Diagnóstico de exemplo</span><span class="sxs-lookup"><span data-stu-id="96d0f-153">Example diagnosis</span></span>

<span data-ttu-id="96d0f-154">As seções a seguir aplicam essas etapas ao aplicativo de exemplo descrito anteriormente.</span><span class="sxs-lookup"><span data-stu-id="96d0f-154">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-the-volume-of-database-activity"></a><span data-ttu-id="96d0f-155">Monitorar o volume de atividade do banco de dados</span><span class="sxs-lookup"><span data-stu-id="96d0f-155">Monitor the volume of database activity</span></span>

<span data-ttu-id="96d0f-156">O gráfico a seguir mostra os resultados da execução de um teste de carga no aplicativo de exemplo, usando uma carga de etapa de até 50 usuários simultâneos.</span><span class="sxs-lookup"><span data-stu-id="96d0f-156">The following graph shows the results of running a load test against the sample application, using a step load of up to 50 concurrent users.</span></span> <span data-ttu-id="96d0f-157">O volume de solicitações atinge um limite e permanece nesse nível, enquanto o tempo médio de resposta aumenta rapidamente.</span><span class="sxs-lookup"><span data-stu-id="96d0f-157">The volume of requests quickly reaches a limit and stays at that level, while the average response time steadily increases.</span></span> <span data-ttu-id="96d0f-158">Observe que uma escala logarítmica é usada para essas duas métricas.</span><span class="sxs-lookup"><span data-stu-id="96d0f-158">Note that a logarithmic scale is used for those two metrics.</span></span>

![Resultados de teste de carga para executar o processamento no banco de dados][ProcessingInDatabaseLoadTest]

<span data-ttu-id="96d0f-160">O gráfico de Avançar mostra a utilização da CPU e DTUs como um percentual da cota de serviço.</span><span class="sxs-lookup"><span data-stu-id="96d0f-160">The next graph shows CPU utilization and DTUs as a percentage of service quota.</span></span> <span data-ttu-id="96d0f-161">DTUs fornece uma medida de quanto o banco de dados executa o processamento.</span><span class="sxs-lookup"><span data-stu-id="96d0f-161">DTUs provides a measure of how much processing the database performs.</span></span> <span data-ttu-id="96d0f-162">O gráfico mostra que a utilização da CPU e DTU ambos os rapidamente atingido 100%.</span><span class="sxs-lookup"><span data-stu-id="96d0f-162">The graph shows that CPU and DTU utilization both quickly reached 100%.</span></span>

![Monitor do Banco de Dados SQL do Azure mostrando o desempenho do banco de dados ao executar o processamento][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a><span data-ttu-id="96d0f-164">Examine o trabalho executado pelo banco de dados</span><span class="sxs-lookup"><span data-stu-id="96d0f-164">Examine the work performed by the database</span></span>

<span data-ttu-id="96d0f-165">É possível que as tarefas executadas pelo banco de dados sejam operações de acesso de dados original, em vez de processamento e, portanto, é importante entender as instruções SQL que estão sendo executadas enquanto o banco de dados está ocupado.</span><span class="sxs-lookup"><span data-stu-id="96d0f-165">It could be that the tasks performed by the database are genuine data access operations, rather than processing, so it is important to understand the SQL statements being run while the database is busy.</span></span> <span data-ttu-id="96d0f-166">Monitore o sistema para capturar o tráfego SQL e correlacionar as operações de SQL com solicitações do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="96d0f-166">Monitor the system to capture the SQL traffic and correlate the SQL operations with application requests.</span></span>

<span data-ttu-id="96d0f-167">Se as operações de banco de dados são puramente acesso operações de dados, sem muito processamento, em seguida, o problema pode ser [Busca Incorreta][ExtraneousFetching].</span><span class="sxs-lookup"><span data-stu-id="96d0f-167">If the database operations are purely data access operations, without a lot of processing, then the problem might be [Extraneous Fetching][ExtraneousFetching].</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="96d0f-168">Implementar a solução e verificar o resultado</span><span class="sxs-lookup"><span data-stu-id="96d0f-168">Implement the solution and verify the result</span></span>

<span data-ttu-id="96d0f-169">O gráfico a seguir mostra um teste de carga usando o código atualizado.</span><span class="sxs-lookup"><span data-stu-id="96d0f-169">The following graph shows a load test using the updated code.</span></span> <span data-ttu-id="96d0f-170">Taxa de transferência é significativamente maior, mais de 400 solicitações por segundo versus 12 anteriormente.</span><span class="sxs-lookup"><span data-stu-id="96d0f-170">Throughput is significantly higher, over 400 requests per second versus 12 earlier.</span></span> <span data-ttu-id="96d0f-171">Tempo médio de resposta também é muito menor, logo acima 0,1 segundos em comparação comparados mais de 4 segundos.</span><span class="sxs-lookup"><span data-stu-id="96d0f-171">The average response time is also much lower, just above 0.1 seconds compared to over 4 seconds.</span></span>

![Resultados de teste de carga para executar o processamento no banco de dados][ProcessingInClientApplicationLoadTest]

<span data-ttu-id="96d0f-173">A utilização da CPU e DTU mostra que o sistema levou mais tempo para alcançar a saturação, apesar da maior taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="96d0f-173">CPU and DTU utilization shows that the system took longer to reach saturation, despite the increased throughput.</span></span>

![O monitor de Banco de Dados SQL do Azure mostrando o desempenho do banco de dados ao executar o processamento no aplicativo cliente][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a><span data-ttu-id="96d0f-175">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="96d0f-175">Related resources</span></span> 

- <span data-ttu-id="96d0f-176">[Antipadrão de Busca Incorreta][ExtraneousFetching]</span><span class="sxs-lookup"><span data-stu-id="96d0f-176">[Extraneous Fetching antipattern][ExtraneousFetching]</span></span>


[dtu]: /azure/sql-database/sql-database-service-tiers-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
