---
title: Antipadrão de E/S com ruídos
description: Um grande número de solicitações de E/S pode prejudicar o desempenho e a capacidade de resposta.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 17193198918cc742b2e3f30e77dfc5c3f2726ebf
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428560"
---
# <a name="chatty-io-antipattern"></a>Antipadrão de E/S com ruídos

O efeito cumulativo de um grande número de solicitações de E/S pode ter um impacto significativo no desempenho e capacidade de resposta.

## <a name="problem-description"></a>Descrição do problema

As chamadas de rede e outras operações de E/S são inerentemente lentas em comparação com as tarefas de computação. Cada solicitação de E/S geralmente tem uma sobrecarga significativa e o efeito cumulativo de várias operações de E/S pode reduzir a velocidade do sistema. Veja algumas causas comuns de E/S com ruídos.

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a>Ler e gravar registros individuais em um banco de dados como solicitações distintas

O exemplo a seguir lê em um banco de dados de produtos. Há três tabelas: `Product`, `ProductSubcategory` e `ProductPriceListHistory`. O código recupera todos os produtos em uma subcategoria, junto com as informações de preços, executando uma série de consultas:  

1. Consulte a subcategoria na tabela `ProductSubcategory`.
2. Localize todos os produtos nessa subcategoria consultando a tabela `Product`.
3. Para cada produto, consulte os dados de preço na tabela `ProductPriceListHistory`.

O aplicativo usa o [Entity Framework][ef] para consultar o banco de dados. Você pode encontrar o exemplo completo [aqui][code-sample]. 

```csharp
public async Task<IHttpActionResult> GetProductsInSubCategoryAsync(int subcategoryId)
{
    using (var context = GetContext())
    {
        // Get product subcategory.
        var productSubcategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subcategoryId)
                .FirstOrDefaultAsync();

        // Find products in that category.
        productSubcategory.Product = await context.Products
            .Where(p => subcategoryId == p.ProductSubcategoryId)
            .ToListAsync();

        // Find price history for each product.
        foreach (var prod in productSubcategory.Product)
        {
            int productId = prod.ProductId;
            var productListPriceHistory = await context.ProductListPriceHistory
                .Where(pl => pl.ProductId == productId)
                .ToListAsync();
            prod.ProductListPriceHistory = productListPriceHistory;
        }
        return Ok(productSubcategory);
    }
}
```

Esse exemplo mostra o problema explicitamente, mas, às vezes, um O/RM pode mascarar o problema se ele busca implicitamente um registro filho por vez. Isso é conhecido como "problema N+1". 

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a>Implementando uma única operação lógica como uma série de solicitações HTTP

Isso geralmente acontece quando os desenvolvedores tentam seguir um paradigma orientado a objetos e tratam os objetos remotos como se fossem objetos locais na memória. Também pode resultar em muitas viagens de ida e volta da rede. Por exemplo, a seguinte API da Web expõe as propriedades individuais dos objetos `User` por meio de métodos HTTP GET individuais. 

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}/username")]
    public HttpResponseMessage GetUserName(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/gender")]
    public HttpResponseMessage GetGender(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/dateofbirth")]
    public HttpResponseMessage GetDateOfBirth(int id)
    {
        ...
    }
}
```

Embora não haja nada tecnicamente errado com essa abordagem, a maioria dos clientes provavelmente precisará obter várias propriedades de cada `User`, resultando em um código do cliente, como a seguir. 

```csharp
HttpResponseMessage response = await client.GetAsync("users/1/username");
response.EnsureSuccessStatusCode();
var userName = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/gender");
response.EnsureSuccessStatusCode();
var gender = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/dateofbirth");
response.EnsureSuccessStatusCode();
var dob = await response.Content.ReadAsStringAsync();
```

### <a name="reading-and-writing-to-a-file-on-disk"></a>Lendo e gravando em um arquivo no disco

A E/S do arquivo envolve abrir um arquivo e ir para o ponto apropriado antes de ler ou gravar os dados. Quando a operação é concluída, o arquivo pode ser fechado para salvar os recursos do sistema operacional. Um aplicativo que lê e grava continuamente pequenas quantidades de informações em um arquivo irá gerar uma sobrecarga significativa de E/S. Pequenas solicitações de gravação também podem causar a fragmentação do arquivo, reduzindo ainda mais a velocidade das operações de E/S subsequentes. 

O exemplo a seguir usa um `FileStream` para gravar um objeto `Customer` em um arquivo. Criar o `FileStream` abre o arquivo e descartá-lo fecha o arquivo. (A instrução `using` descarta automaticamente o objeto `FileStream`.) Se o aplicativo chamar esse método repetidamente à medida que novos clientes são adicionados, a sobrecarga de E/S poderá acumular-se rapidamente.

```csharp
private async Task SaveCustomerToFileAsync(Customer cust)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        byte [] data = null;
        using (MemoryStream memStream = new MemoryStream())
        {
            formatter.Serialize(memStream, cust);
            data = memStream.ToArray();
        }
        await fileStream.WriteAsync(data, 0, data.Length);
    }
}
```

## <a name="how-to-fix-the-problem"></a>Como corrigir o problema

Reduza o número de solicitações de E/S colocando os dados em menos solicitações maiores.

Obtenha os dados em um banco de dados como uma única consulta, em vez de várias consultas menores. Veja uma versão revisada do código que recupera as informações do produto.

```csharp
public async Task<IHttpActionResult> GetProductCategoryDetailsAsync(int subCategoryId)
{
    using (var context = GetContext())
    {
        var subCategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subCategoryId)
                .Include("Product.ProductListPriceHistory")
                .FirstOrDefaultAsync();

        if (subCategory == null)
            return NotFound();

        return Ok(subCategory);
    }
}
```

Siga os princípios de design REST para as APIs da Web. Veja uma versão revisada de API da Web do exemplo anterior. Em vez de métodos GET separados para cada propriedade, há um único método GET que retorna `User`. Isso resulta em um corpo de resposta maior por solicitação, mas é provável que cada cliente faça menos chamadas à API.

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}")]
    public HttpResponseMessage GetUser(int id)
    {
        ...
    }
}

// Client code
HttpResponseMessage response = await client.GetAsync("users/1");
response.EnsureSuccessStatusCode();
var user = await response.Content.ReadAsStringAsync();
```

Para a E/S do arquivo, considere colocar em buffer os dados na memória e, em seguida, gravar os dados em buffer em um arquivo como uma única operação. Essa abordagem reduz a sobrecarga de abrir e fechar repetidamente o arquivo e ajuda a reduzir a fragmentação do arquivo no disco.

```csharp
// Save a list of customer objects to a file
private async Task SaveCustomerListToFileAsync(List<Customer> customers)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        foreach (var cust in customers)
        {
            byte[] data = null;
            using (MemoryStream memStream = new MemoryStream())
            {
                formatter.Serialize(memStream, cust);
                data = memStream.ToArray();
            }
            await fileStream.WriteAsync(data, 0, data.Length);
        }
    }
}

// In-memory buffer for customers.
List<Customer> customers = new List<Customers>();

// Create a new customer and add it to the buffer
var cust = new Customer(...);
customers.Add(cust);

// Add more customers to the list as they are created
...

// Save the contents of the list, writing all customers in a single operation
await SaveCustomerListToFileAsync(customers);
```

## <a name="considerations"></a>Considerações

- Os dois primeiros exemplos fazem *menos* chamadas de E/S, mas cada um recupera *mais* informações. Você deve considerar a compensação entre esses dois fatores. A resposta certa dependerá dos padrões de uso reais. No exemplo de API da Web, pode acontecer que os clientes normalmente precisem apenas do nome de usuário. Nesse caso, talvez faça sentido exibi-lo como uma chamada da API separada. Para obter mais informações, confira o antipadrão de [Busca Incorreta][extraneous-fetching].

- Ao ler os dados, não torne suas solicitações de E/S muito grandes. Um aplicativo deve recuperar apenas as informações que provavelmente usará. 

- Às vezes, é útil dividir as informações de um objeto em duas partes: *os dados acessados com frequência* responsáveis pela maioria das solicitações, e os *dados acessados com menos frequência* e usados raramente. Geralmente, os dados acessados com mais frequência são uma parte relativamente pequena do total de dados para um objeto, portanto, retornar apenas essa parte pode economizar uma sobrecarga de E/S significativa.

- Ao gravar os dados, evite bloquear os recursos por mais tempo que o necessário para reduzir as chances de contenção durante uma operação demorada. Se uma operação de gravação se estender por vários armazenamentos de dados, arquivos ou serviços, adote uma abordagem consistente. Confira [Orientação para a Consistência de Dados][data-consistency-guidance].

- Se você armazenar em buffer os dados na memória antes de gravá-los, eles ficarão vulneráveis se o processo falhar. Se a taxa de dados geralmente tem picos ou é relativamente esparsa, pode ser mais seguro colocar em buffer os dados em uma fila durável externa, como os [Hubs de Eventos](https://azure.microsoft.com/services/event-hubs/).

- Considere armazenar em cache os dados recuperados de um serviço ou banco de dados. Isso pode ajudar a reduzir o volume de E/S, evitando repetidas solicitações dos mesmos dados. Para obter mais informações, confira [Melhores Práticas do Cache][caching-guidance].

## <a name="how-to-detect-the-problem"></a>Como detectar o problema

Os sintomas de E/S com ruídos incluem uma alta latência e baixa taxa de transferência. Os usuários finais provavelmente irão relatar tempos de resposta maiores ou falhas causadas por serviços atingindo o tempo limite devido à contenção aumentada para os recursos de E/S.

Você pode executar as etapas a seguir para ajudar a identificar as causas dos problemas:

1. Execute o monitoramento de processos do sistema de produção para identificar as operações com tempos de resposta ruins.
2. Realize testes de carga de cada operação identificada na etapa anterior.
3. Durante os testes de carga, colete dados de telemetria sobre as solicitações de acesso a dados feitas por cada operação.
4. Colete estatísticas detalhadas para cada solicitação enviada para um armazenamento de dados.
5. Crie um perfil para o aplicativo no ambiente de teste para estabelecer onde os possíveis gargalos de E/S podem estar ocorrendo. 

Procure esses sintomas:

- Um grande número de solicitações de E/S pequenas feitas para o mesmo arquivo.
- Um grande número de pequenas solicitações de rede feitas por uma instância do aplicativo para o mesmo serviço.
- Um grande número de pequenas solicitações feitas por uma instância do aplicativo para o mesmo armazenamento de dados.
- Aplicativos e serviços tendo um limite de E/S.

## <a name="example-diagnosis"></a>Diagnóstico de exemplo

As seções a seguir aplicam essas etapas no exemplo mostrado anteriormente que consulta um banco de dados.

### <a name="load-test-the-application"></a>Fazer teste de carga no aplicativo

Este gráfico mostra os resultados do teste de carga. O tempo médio de resposta é medido em 10 s por solicitação. O gráfico mostra uma latência muito alta. Com uma carga de 1.000 usuários, um usuário terá que esperar aproximadamente um minuto para ver os resultados de uma consulta. 

![Os resultados do teste de carga dos indicadores-chave para o aplicativo de exemplo de E/S com ruídos][key-indicators-chatty-io]

> [!NOTE]
> O aplicativo foi implantado como um aplicativo Web do Serviço de Aplicativo do Azure, usando o Banco de Dados SQL do Azure. O teste de carga usou uma carga de trabalho da etapa simulada de até 1.000 usuários simultâneos. O banco de dados foi configurado com um pool de conexões com suporte de até 1.000 conexões simultâneas para reduzir a chance de que uma contenção por conexões afete os resultados. 

### <a name="monitor-the-application"></a>Monitorar o aplicativo

Você pode usar um pacote APM (monitoramento de desempenho do aplicativo) para capturar e analisar as principais métricas que podem identificar a E/S com ruídos. Quais métricas são importantes dependerá da carga de trabalho de E/S. Neste exemplo, as solicitações de E/S interessantes foram as consultas do banco de dados. 

A imagem a seguir mostra os resultados gerados usando o [APM do New Relic][new-relic]. O tempo de resposta médio do banco de dados atingiu um pico em aproximadamente 5,6 s por solicitação durante a carga de trabalho máxima. O sistema foi capaz de oferecer suporte a uma média de 410 solicitações por minuto durante o teste.

![Visão geral do tráfego acessando o banco de dados AdventureWorks2012][databasetraffic]

### <a name="gather-detailed-data-access-information"></a>Coletar informações detalhadas do acesso a dados

Uma análise mais profunda dos dados de monitoramento mostra que o aplicativo executa três instruções SQL SELECT diferentes. Elas correspondem às solicitações geradas pelo Entity Framework para obter dados nas tabelas `ProductListPriceHistory`, `Product` e `ProductSubcategory`.
Além disso, a consulta que recupera os dados na tabela `ProductListPriceHistory` é de longe a instrução SELECT mais executada, em uma ordem de importância.

![Consultas executadas pelo aplicativo de exemplo em teste][queries]

Acontece que o método `GetProductsInSubCategoryAsync`, mostrado anteriormente, executa 45 consultas SELECT. Cada consulta faz com que o aplicativo abra uma nova conexão de SQL.

![Estatísticas da consulta para o aplicativo de exemplo em teste][queries2]

> [!NOTE]
> Esta imagem mostra informações de rastreamento para a instância mais lenta da operação `GetProductsInSubCategoryAsync` no teste de carga. Em um ambiente de produção, é útil examinar os rastreamentos das instâncias mais lentas para ver se há um padrão que indica um problema. Se você examinar apenas os valores médios, poderá ignorar os problemas que serão muito piores sob carga.

A imagem seguinte mostra as instruções SQL reais que foram emitidas. A consulta de busca das informações de preço é executada para cada produto individual na subcategoria do produto. Usar uma junção reduziria muito o número de chamadas do banco de dados.

![Detalhes da consulta para o aplicativo de exemplo em teste][queries3]

Se você estiver usando um O/RM, como o Entity Framework, rastrear as consultas SQL poderá fornecer informações sobre como o O/RM converte as chamadas da programação em instruções SQL e indicará as áreas onde o acesso aos dados pode ser otimizado. 

### <a name="implement-the-solution-and-verify-the-result"></a>Implementar a solução e verificar o resultado

Reescrever a chamada para o Entity Framework gerou os seguintes resultados.

![Os resultados do teste de carga dos indicadores-chave da API robusta no aplicativo de exemplo de E/S com ruídos][key-indicators-chunky-io]

Esse teste de carga foi executado na mesma implantação, usando o mesmo perfil de carga. Desta vez, o gráfico mostra uma latência muito mais baixa. O tempo médio de solicitação com 1.000 usuários é entre 5 e 6 segundos, em quase um minuto.

Desta vez, o sistema ofereceu suporte a uma média de 3.970 solicitações por minuto, em comparação com 410 para o teste anterior.

![Visão geral da transação para a API robusta][databasetraffic2]

Rastrear a instrução SQL mostra que todos os dados são buscados em uma única instrução SELECT. Embora essa consulta seja muito mais complexa, ela é executada apenas uma vez por operação. E embora as junções complexas possam ficar caras, os sistemas do banco de dados relacional são otimizados para esse tipo de consulta.  

![Detalhes da consulta para a API robusta][queries4]

## <a name="related-resources"></a>Recursos relacionados

- [Práticas recomendadas de Design da API][api-design]
- [Práticas recomendadas do cache][caching-guidance]
- [Manual da Consistência de Dados][data-consistency-guidance]
- [Antipadrão de Busca Incorreta][extraneous-fetching]
- [Nenhum antipadrão de cache][no-cache]

[api-design]: ../../best-practices/api-design.md
[caching-guidance]: ../../best-practices/caching.md
[code-sample]:  https://github.com/mspnp/performance-optimization/tree/master/ChattyIO
[data-consistency-guidance]: https://msdn.microsoft.com/library/dn589800.aspx
[ef]: /ef/
[extraneous-fetching]: ../extraneous-fetching/index.md
[new-relic]: https://newrelic.com/application-monitoring
[no-cache]: ../no-caching/index.md

[key-indicators-chatty-io]: _images/ChattyIO.jpg
[key-indicators-chunky-io]: _images/ChunkyIO.jpg
[databasetraffic]: _images/DatabaseTraffic.jpg
[databasetraffic2]: _images/DatabaseTraffic2.jpg
[queries]: _images/DatabaseQueries.jpg
[queries2]: _images/DatabaseQueries2.jpg
[queries3]: _images/DatabaseQueries3.jpg
[queries4]: _images/DatabaseQueries4.jpg

