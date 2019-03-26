---
title: Antipadrão de busca externa
titleSuffix: Performance antipatterns for cloud apps
description: Recuperar mais dados que o necessário para uma operação de negócios pode resultar em sobrecarga de E/S desnecessária e reduzir a capacidade de resposta.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
---

# <a name="extraneous-fetching-antipattern"></a>Antipadrão de busca externa

Recuperar mais dados que o necessário para uma operação de negócios pode resultar em sobrecarga de E/S desnecessária e reduzir a capacidade de resposta.

## <a name="problem-description"></a>Descrição do problema

Este antipadrão pode ocorrer se o aplicativo tentar minimizar as solicitações de E/S, recuperando todos os dados que ele *possa* precisar. Isso geralmente é um resultado de sobrecompensação para antipadrão de [E/S informais][chatty-io]. Por exemplo, um aplicativo pode buscar os detalhes de cada produto em um banco de dados. Mas o usuário pode ter apenas um subconjunto dos detalhes (alguns podem não ser relevantes para os clientes) e provavelmente não precisa ver *todos* os produtos de uma vez. Mesmo se o usuário estiver navegando pelo catálogo inteiro, faria sentido paginar os resultados &mdash; mostrando 20 ao mesmo tempo, por exemplo.

Outra fonte desse problema é seguir práticas de programação ou design ruins. Por exemplo, o código a seguir usa o Entity Framework para buscar os detalhes completos de todos os produtos. Em seguida, ele filtra os resultados para retornar apenas um subconjunto dos campos, descartando o restante. Você pode encontrar o exemplo completo [aqui][sample-app].

```csharp
public async Task<IHttpActionResult> GetAllFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Execute the query. This happens at the database.
        var products = await context.Products.ToListAsync();

        // Project fields from the query results. This happens in application memory.
        var result = products.Select(p => new ProductInfo { Id = p.ProductId, Name = p.Name });
        return Ok(result);
    }
}
```

No exemplo a seguir, o aplicativo recupera dados para executar uma agregação que pode ser feita pelo banco de dados. O aplicativo calcula o total de vendas obtendo todos os registros para todos os pedidos de venda e, em seguida, calculando a soma sobre esses registros. Você pode encontrar o exemplo completo [aqui][sample-app].

```csharp
public async Task<IHttpActionResult> AggregateOnClientAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Fetch all order totals from the database.
        var orderAmounts = await context.SalesOrderHeaders.Select(soh => soh.TotalDue).ToListAsync();

        // Sum the order totals in memory.
        var total = orderAmounts.Sum();
        return Ok(total);
    }
}
```

O exemplo a seguir mostra um problema sutil causado pelo modo como o Entity Framework utiliza o LINQ to Entities.

```csharp
var query = from p in context.Products.AsEnumerable()
            where p.SellStartDate < DateTime.Now.AddDays(-7) // AddDays cannot be mapped by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

O aplicativo está tentando localizar produtos com um `SellStartDate` com mais de uma semana. Na maioria dos casos, o LINQ to Entities converterá uma cláusula `where` para uma instrução SQL que é executada pelo banco de dados. No entanto, nesse caso, o LINQ to Entities não consegue mapear o método `AddDays` para SQL. Em vez disso, cada linha da tabela `Product` será retornada e os resultados serão filtrados na memória.

A chamada para `AsEnumerable` é uma dica de que há um problema. Este método converte os resultados para uma interface `IEnumerable`. Embora `IEnumerable` ofereça suporte à filtragem, ela é feita pelo *cliente*, e não pelo banco de dados. Por padrão, o LINQ to Entities usa `IQueryable`, que passa a responsabilidade da filtragem para a fonte de dados.

## <a name="how-to-fix-the-problem"></a>Como corrigir o problema

Evite a busca de grandes volumes de dados que rapidamente podem ficar desatualizados ou ser descartados e busque somente os dados necessários para a operação que está sendo executada.

Em vez de obter todas as colunas de uma tabela e, em seguida, filtrá-las, selecione as colunas que você precisa do banco de dados.

```csharp
public async Task<IHttpActionResult> GetRequiredFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Project fields as part of the query itself
        var result = await context.Products
            .Select(p => new ProductInfo {Id = p.ProductId, Name = p.Name})
            .ToListAsync();
        return Ok(result);
    }
}
```

Da mesma forma, execute a agregação no banco de dados e não na memória do aplicativo.

```csharp
public async Task<IHttpActionResult> AggregateOnDatabaseAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Sum the order totals as part of the database query.
        var total = await context.SalesOrderHeaders.SumAsync(soh => soh.TotalDue);
        return Ok(total);
    }
}
```

Ao usar o Entity Framework, verifique se as consultas do LINQ são resolvidas usando a interface `IQueryable` e não `IEnumerable`. Talvez seja necessário ajustar a consulta para usar apenas as funções que podem ser mapeadas para a fonte de dados. O exemplo anterior pode ser refatorado para remover o método `AddDays` da consulta, permitindo que a filtragem seja feita pelo banco de dados.

```csharp
DateTime dateSince = DateTime.Now.AddDays(-7); // AddDays has been factored out.
var query = from p in context.Products
            where p.SellStartDate < dateSince // This criterion can be passed to the database by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

## <a name="considerations"></a>Considerações

- Em alguns casos, você pode melhorar o desempenho através do particionamento horizontal de dados. Se operações diferentes acessam atributos diferentes dos dados, o particionamento horizontal pode reduzir a contenção. Geralmente, a maioria das operações são executadas em relação a um pequeno subconjunto dos dados, para que a difusão desse carregamento possa melhorar o desempenho. Confira o [Particionamento de dados][data-partitioning].

- Para operações que oferecem suporte a consultas não associadas, implemente a paginação e busque apenas um número limitado de entidades por vez. Por exemplo, se um cliente estiver navegando por um catálogo de produtos, você pode mostrar uma página de resultados de cada vez.

- Quando possível, aproveite os recursos incluídos no armazenamento de dados. Por exemplo, bancos de dados SQL normalmente fornecem funções de agregação.

- Se você estiver usando um armazenamento de dados que não oferece suporte a uma função específica, como a agregação, poderia armazenar o resultado calculado em outro lugar, atualizando o valor conforma os registros são adicionados ou atualizados, para que o aplicativo não precise recalcular o valor sempre que necessário.

- Se você observar que as solicitações estão recuperando um grande número de campos, examine o código-fonte para determinar se todos esses campos são realmente necessários. Às vezes, essas solicitações são o resultado de uma consulta `SELECT *` mal projetada.

- Da mesma forma, as solicitações que recuperam um grande número de entidades podem ser sinal de que o aplicativo não está filtrando os dados corretamente. Verifique se todas essas entidades são realmente necessárias. Se possível, use o banco de dados da filtragem, por exemplo, usando cláusulas `WHERE` no SQL.

- O processamento de descarregamento para o banco de dados nem sempre é a melhor opção. Só use essa estratégia quando o banco de dados for criado ou otimizado para fazer isso. A maioria dos sistemas de banco de dados são altamente otimizados para determinadas funções, mas não são projetados para atuar como mecanismos de uso geral do aplicativo. Para obter mais informações, confira o [Antipadrão de Banco de Dados Ocupado][BusyDatabase].

## <a name="how-to-detect-the-problem"></a>Como detectar o problema

Os sintomas de busca externa incluem latência alta e baixa taxa de transferência. Se os dados são recuperados a partir de um armazenamento de dados, uma maior contenção também é provável. É provável que os usuários finais relatem tempos de resposta estendidos ou falhas causadas por serviços atingindo o tempo limite. Essas falhas podem retornar erros HTTP 500 (Servidor Interno) ou erros HTTP 503 (Serviço Indisponível). Examine os logs de eventos para o servidor Web, que provavelmente contêm informações mais detalhadas sobre as causas e as circunstâncias dos erros.

Os sintomas desse antipadrão e alguns de telemetria obtida podem ser muito semelhantes àqueles do [Antipadrão de Persistência Monolítica][MonolithicPersistence].

Você pode executar as etapas a seguir para ajudar a identificar a causa:

1. Identifique as transações ou cargas de trabalho lentas realizando teste de carga, monitoramento de processos ou outros métodos de captura de dados de instrumentação.
2. Observe os padrões comportamentais apresentados pelo sistema. Há limites específicos em termos de transações por segundo ou volume de usuários?
3. Correlacione as instâncias de cargas de trabalho lentas com padrões comportamentais.
4. Identifique os armazenamentos de dados que estão sendo usados. Para cada fonte de dados, execute a telemetria de baixo nível para observar o comportamento das operações.
5. Identifique as consultas de execução lenta que fazem referência a essas fontes de dados.
6. Execute uma análise específica do recurso de consultas de execução lenta e determine como os dados são usados e consumidos.

Procure esses sintomas:

- Solicitações de E/S grandes e frequentes feitas para o mesmo recurso ou armazenamento de dados.
- Contenção em um armazenamento de dados ou recurso compartilhado.
- Uma operação que recebe frequentemente grandes volumes de dados através da rede.
- Aplicativos e serviços gastando tempo significativo aguardando a conclusão da E/S.

## <a name="example-diagnosis"></a>Diagnóstico de exemplo

As seções a seguir se aplicam a estas etapas para os exemplos anteriores.

### <a name="identify-slow-workloads"></a>Identificar as cargas de trabalho lentas

Este gráfico mostra os resultados de desempenho de um teste de carga que simulou até 400 usuários simultâneos executando o método `GetAllFieldsAsync` mostrado anteriormente. A taxa de transferência lenta diminui à medida que a carga aumenta. O tempo médio de resposta aumenta conforme a carga de trabalho aumenta.

![Resultados do teste de carga para o método GetAllFieldsAsync][Load-Test-Results-Client-Side1]

Um teste de carga para a operação `AggregateOnClientAsync` mostra um padrão semelhante. O volume das solicitações é razoavelmente estável. O tempo médio de resposta aumenta com a carga de trabalho, embora mais lentamente do que o gráfico anterior.

![Resultados do teste de carga para o método AggregateOnClientAsync][Load-Test-Results-Client-Side2]

### <a name="correlate-slow-workloads-with-behavioral-patterns"></a>Correlacione as cargas de trabalho lentas com os padrões comportamentais

Qualquer correlação entre períodos normais de alto uso e diminuindo o desempenho pode indicar áreas de interesse. Examine atentamente o perfil de desempenho da funcionalidade suspeita de lenta execução para determinar se ele corresponde ao teste de carga executado anteriormente.

Utilize um teste de carga com a mesma funcionalidade usando cargas de usuário com base na etapa para localizar o ponto em que o desempenho cai significativamente ou falha completamente. Se esse ponto está dentro dos limites de seu uso real esperado, examine como a funcionalidade é implementada.

Uma operação lenta não é necessariamente um problema, se ela não estiver sendo executada quando o sistema estiver sob carga excessiva, não é um tempo crítico e não afeta de forma negativa o desempenho de outras operações importantes. Por exemplo, a geração de estatísticas operacionais mensais pode ser uma operação de longa execução, mas provavelmente pode ser executada como um processo em lote e executada como um trabalho de baixa prioridade. Por outro lado, os clientes consultando o catálogo de produtos é uma operação de negócios crítica. Se concentre na telemetria gerada por essas operações críticas para ver como o desempenho varia durante os períodos de alto uso.

### <a name="identify-data-sources-in-slow-workloads"></a>Identificar as fontes de dados em cargas de trabalho lentas

Se você suspeitar que um serviço está com um desempenho ruim devido à forma como recupera os dados, investigue como o aplicativo interage com os repositórios que ele usa. Monitore o sistema ao vivo para ver quais fontes são acessadas durante períodos de baixo desempenho.

Para cada fonte de dados, instrumente o sistema para capturar o seguinte:

- A frequência com a qual cada armazenamento de dados é acessado.
- O volume de dados entrando e saindo do armazenamento de dados.
- O tempo dessas operações, especialmente a latência de solicitações.
- A natureza e a taxa de erros que ocorrem durante o acesso a cada armazenamento de dados sob carga típica.

Compare essas informações com o volume de dados que está sendo retornado pelo aplicativo para o cliente. Acompanhe a taxa do volume de dados retornado pelo armazenamento de dados contra o volume de dados retornado para o cliente. Se não houver nenhuma diferença grande, investigue para determinar se o aplicativo está buscando dados que não precisa.

Você poderá capturar esses dados ao observar o sistema ao vivo e o rastreamento de ciclo de vida de cada solicitação de usuário, ou pode modelar uma série de cargas de trabalho sintéticas e executá-las em um sistema de teste.

Os gráficos a seguir mostram a telemetria capturada usando [APM do New Relic][new-relic] durante um teste de carga do método `GetAllFieldsAsync`. Observe a diferença entre os volumes de dados recebidos do banco de dados e as respostas HTTP correspondentes.

![Telemetria para o método 'GetAllFieldsAsync'][TelemetryAllFields]

Para cada solicitação, o banco de dados retornou 80,503 bytes, mas a resposta ao cliente continha somente 19,855 bytes, cerca de 25% do tamanho da resposta de banco de dados. O tamanho dos dados retornados ao cliente pode variar dependendo do formato. Para este teste de carga, o cliente solicitou dados JSON. O teste separado usando XML (não mostrado) tinha um tamanho de resposta de 35,655 bytes ou 44% do tamanho da resposta de banco de dados.

O teste de carga para o método `AggregateOnClientAsync` mostra mais resultados extremos. Neste caso, cada teste executou uma consulta que recuperou mais de 280Kb de dados do banco de dados, mas a resposta JSON foi apenas 14 bytes. A grande diferença é porque o método calcula um resultado agregado de um grande volume de dados.

![Telemetria para o método 'AggregateOnClientAsync'][TelemetryAggregateOnClient]

### <a name="identify-and-analyze-slow-queries"></a>Identificar e analisar consultas lentas

Procure por consultas de banco de dados que consomem a maioria dos recursos e levam mais tempo para executar. Você pode adicionar a instrumentação para localizar os horários de início e conclusão para várias operações de banco de dados. Vários armazenamentos de dados também oferecem informações detalhadas sobre como as consultas são executadas e otimizadas. Por exemplo, o painel de Desempenho da Consulta no portal de gerenciamento de banco de Dados SQL do Azure permite-lhe selecionar uma consulta e exibir informações detalhadas do desempenho de tempo de execução. Aqui está a consulta gerada pela operação `GetAllFieldsAsync`:

![O painel de Detalhes da Consulta no portal de gerenciamento de banco de dados SQL do Windows Azure][QueryDetails]

## <a name="implement-the-solution-and-verify-the-result"></a>Implementar a solução e verificar o resultado

Depois de alterar o método `GetRequiredFieldsAsync` para usar uma instrução SELECT no lado do banco de dados, os testes de carga mostraram os seguintes resultados.

![Resultados do teste de carga para o método GetRequiredFieldsAsync][Load-Test-Results-Database-Side1]

Este teste de carga utilizou a mesma implantação e a mesma carga de trabalho simulada de 400 usuários simultâneos que antes. O gráfico mostra uma latência muito mais baixa. O tempo de resposta aumenta com a carga aproximadamente 1,3 segundos, comparada a 4 segundos no caso anterior. A taxa de transferência também é superior a 350 solicitações por segundo comparada a 100 anteriormente. O volume de dados recuperado do banco de dados agora intimamente corresponde ao tamanho das mensagens de resposta HTTP.

![Telemetria para o método 'GetRequiredFieldsAsync'][TelemetryRequiredFields]

Os testes de carga usando o método `AggregateOnDatabaseAsync` geram os seguintes resultados:

![Resultados do teste de carga para o método AggregateOnDatabaseAsync][Load-Test-Results-Database-Side2]

O tempo médio de resposta é agora mínimo. Esse é um aprimoramento de ordem de magnitude no desempenho, causado principalmente pela grande redução da E/S do banco de dados.

Aqui está a telemetria correspondente para o método `AggregateOnDatabaseAsync`. Foi reduzida consideravelmente a quantidade de dados recuperados do banco de dados, de 280Kb por transação a 53 bytes. Como resultado, o número máximo prolongado de solicitações por minuto foi aumentado de cerca de 2.000 para mais de 25.000.

![Telemetria para o método 'AggregateOnDatabaseAsync'][TelemetryAggregateInDatabaseAsync]

## <a name="related-resources"></a>Recursos relacionados

- [Antipadrão de Banco de Dados Ocupado][BusyDatabase]
- [Antipadrão de E/S informais][chatty-io]
- [Práticas recomendadas de particionamento de dados][data-partitioning]

[BusyDatabase]: ../busy-database/index.md
[data-partitioning]: ../../best-practices/data-partitioning.md
[new-relic]: https://newrelic.com/application-monitoring

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ExtraneousFetching

[chatty-io]: ../chatty-io/index.md
[MonolithicPersistence]: ../monolithic-persistence/index.md
[Load-Test-Results-Client-Side1]:_images/LoadTestResultsClientSide1.jpg
[Load-Test-Results-Client-Side2]:_images/LoadTestResultsClientSide2.jpg
[Load-Test-Results-Database-Side1]:_images/LoadTestResultsDatabaseSide1.jpg
[Load-Test-Results-Database-Side2]:_images/LoadTestResultsDatabaseSide2.jpg
[QueryPerformanceZoomed]: _images/QueryPerformanceZoomed.jpg
[QueryDetails]: _images/QueryDetails.jpg
[TelemetryAllFields]: _images/TelemetryAllFields.jpg
[TelemetryAggregateOnClient]: _images/TelemetryAggregateOnClient.jpg
[TelemetryRequiredFields]: _images/TelemetryRequiredFields.jpg
[TelemetryAggregateInDatabaseAsync]: _images/TelemetryAggregateInDatabase.jpg
