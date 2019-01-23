---
title: Antipadrão de Banco de Dados Ocupado
titleSuffix: Performance antipatterns for cloud apps
description: Descarregamento de processamento em um servidor de banco de dados pode causar problemas de desempenho e escalabilidade.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 7d3fe47407eff7168dfd227a1dd1bd5917c7d431
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487025"
---
# <a name="busy-database-antipattern"></a>Antipadrão de Banco de Dados Ocupado

O processamento de descarregamento para um servidor de banco de dados pode fazer com que ele gaste uma proporção significativa de tempo de execução do código, em vez de responder às solicitações para armazenar e recuperar dados.

## <a name="problem-description"></a>Descrição do problema

Muitos sistemas de banco de dados podem executar o código. Exemplos incluem procedimentos armazenados e gatilhos. Geralmente, é mais eficiente para executar esse processamento aproximado para os dados, em vez de transmitir os dados para um aplicativo cliente para processamento. No entanto, uso excessivo esses recursos pode prejudicar o desempenho, por vários motivos:

- O servidor de banco de dados pode levar muito tempo de processamento, em vez de aceitar novas solicitações de cliente e buscar dados.
- Um banco de dados geralmente é um recurso compartilhado, ele pode se tornar um afunilamento durante períodos de alto uso.
- Os custos de tempo de execução podem ser excessivos se o armazenamento de dados é monitorado. Que é especialmente verdadeiro para serviços de banco de dados gerenciados. Por exemplo, o Banco de Dados SQL do Azure cobra por [Unidades de Transação do Banco de Dados][dtu] (DTUs).
- Bancos de dados têm capacidade finita de expansão e não é comum para dimensionar um banco de dados horizontalmente. Portanto, talvez seja melhor mover o processamento em um recurso de computação, como um aplicativo do Serviço de Aplicativo ou de VM, que pode expandir facilmente.

Esse antipadrão geralmente ocorre porque:

- O banco de dados é exibido como um serviço em vez de um armazenamento. Um aplicativo pode usar o servidor de banco de dados para formato de dados (por exemplo, a conversão em XML), manipular dados de cadeia de caracteres ou executar cálculos complexos.
- Os desenvolvedores tentam escrever consultas cujos resultados podem ser exibidos diretamente aos usuários. Por exemplo, uma consulta pode combinar campos ou formatar datas, horas e moedas de acordo com a localidade.
- Os desenvolvedores estão tentando corrigir o antipadrão [Busca Incorreta][ExtraneousFetching] durante o envio por push de cálculos para o banco de dados.
- Os procedimentos armazenados são usados para encapsular a lógica de negócios, talvez porque eles são considerados mais fáceis de manter e atualizar.

O exemplo a seguir recupera os 20 pedidos mais valiosos para uma região de vendas especificada e formata os resultados como XML.
Ele usa funções Transact-SQL para analisar os dados e converter os resultados em XML. Você pode encontrar o exemplo completo [aqui][sample-app].

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

Claramente, essa é uma consulta complexa. Como você verá mais tarde, na verdade, para usar os recursos de processamento significativo no servidor de banco de dados.

## <a name="how-to-fix-the-problem"></a>Como corrigir o problema

Mova o processamento do servidor de banco de dados em outros níveis de aplicativo. Idealmente, você deve limitar o banco de dados para executar operações de acesso de dados, usando apenas os recursos para os quais o banco de dados é otimizado, como a agregação em um RDBMS.

Por exemplo, o código Transact-SQL anterior pode ser substituído por uma instrução que simplesmente recupera os dados a serem processados.

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

Em seguida, o aplicativo usa o .NET Framework `System.Xml.Linq` APIs para formatar os resultados como XML.

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
> Esse código é um pouco complexo. Para um novo aplicativo, talvez você prefira usar uma biblioteca de serialização. No entanto, a suposição aqui é que a equipe de desenvolvimento esteja refatorando um aplicativo existente e, portanto, o método deve retornar o mesmo formato exato do que o código original.

## <a name="considerations"></a>Considerações

- Muitos sistemas de banco de dados são altamente otimizados para executar determinados tipos de processamento de dados, como calcular valores de agregação ao longo de grandes conjuntos de dados. Não mova esses tipos de processamento fora do banco de dados.

- Não realoque o processamento se isso fizer o banco de dados transferir mais dados pela rede. Veja o [Antipadrão de Busca incorreta][ExtraneousFetching].

- Se você mover o processamento para uma camada de aplicativo, talvez seja necessário expandir a camada para lidar com o trabalho adicional.

## <a name="how-to-detect-the-problem"></a>Como detectar o problema

Os sintomas de um banco de dados ocupado incluem um declínio desproporcional de tempos de resposta e a taxa de transferência em operações que acessam o banco de dados.

Você pode executar as etapas a seguir para ajudar a identificar o problema:

1. Use o monitoramento de desempenho para identificar o tempo que o sistema de produção passa executando a atividade de banco de dados.

2. Examine o trabalho executado pelo banco de dados durante esses períodos.

3. Se você suspeitar de que determinadas operações podem causar muita atividade de banco de dados, execute em um ambiente controlado de teste de carga. Cada teste deve ser executado uma combinação das operações suspeitas com uma carga de usuário de variável. Examine a telemetria dos testes de carga para observar como o banco de dados é usado.

4. Se a atividade de banco de dados revela o processamento significativo, mas pouco tráfego de dados, examine o código-fonte para determinar se o processamento pode ser mais bem executado em outro lugar.

Se o volume de atividade do banco de dados é baixo ou tempos de resposta são relativamente rápidos, um banco de dados ocupado é improvável de ser um problema de desempenho.

## <a name="example-diagnosis"></a>Diagnóstico de exemplo

As seções a seguir aplicam essas etapas ao aplicativo de exemplo descrito anteriormente.

### <a name="monitor-the-volume-of-database-activity"></a>Monitorar o volume de atividade do banco de dados

O gráfico a seguir mostra os resultados da execução de um teste de carga no aplicativo de exemplo, usando uma carga de etapa de até 50 usuários simultâneos. O volume de solicitações atinge um limite e permanece nesse nível, enquanto o tempo médio de resposta aumenta rapidamente. Observe que uma escala logarítmica é usada para essas duas métricas.

![Resultados de teste de carga para executar o processamento no banco de dados][ProcessingInDatabaseLoadTest]

O gráfico de Avançar mostra a utilização da CPU e DTUs como um percentual da cota de serviço. DTUs fornece uma medida de quanto o banco de dados executa o processamento. O gráfico mostra que a utilização da CPU e DTU ambos os rapidamente atingido 100%.

![Monitor do Banco de Dados SQL do Azure mostrando o desempenho do banco de dados ao executar o processamento][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a>Examine o trabalho executado pelo banco de dados

É possível que as tarefas executadas pelo banco de dados sejam operações de acesso de dados original, em vez de processamento e, portanto, é importante entender as instruções SQL que estão sendo executadas enquanto o banco de dados está ocupado. Monitore o sistema para capturar o tráfego SQL e correlacionar as operações de SQL com solicitações do aplicativo.

Se as operações de banco de dados são puramente acesso operações de dados, sem muito processamento, em seguida, o problema pode ser [Busca Incorreta][ExtraneousFetching].

### <a name="implement-the-solution-and-verify-the-result"></a>Implementar a solução e verificar o resultado

O gráfico a seguir mostra um teste de carga usando o código atualizado. Taxa de transferência é significativamente maior, mais de 400 solicitações por segundo versus 12 anteriormente. Tempo médio de resposta também é muito menor, logo acima 0,1 segundos em comparação comparados mais de 4 segundos.

![Resultados de teste de carga para executar o processamento no banco de dados][ProcessingInClientApplicationLoadTest]

A utilização da CPU e DTU mostra que o sistema levou mais tempo para alcançar a saturação, apesar da maior taxa de transferência.

![O monitor de Banco de Dados SQL do Azure mostrando o desempenho do banco de dados ao executar o processamento no aplicativo cliente][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a>Recursos relacionados

- [Antipadrão de Busca Incorreta][ExtraneousFetching]

[dtu]: /azure/sql-database/sql-database-service-tiers-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
