---
title: Nenhum antipadrão de cache
titleSuffix: Performance antipatterns for cloud apps
description: Buscar repetidamente os mesmos dados pode reduzir o desempenho e a escalabilidade.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 5c3062c0a17de708ada83ba81dcb111e6b1f8440
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58343842"
---
# <a name="no-caching-antipattern"></a>Nenhum antipadrão de cache

Em um aplicativo de nuvem que trata muitas solicitações simultâneas, buscar repetidamente os mesmos dados pode reduzir o desempenho e a escalabilidade.

## <a name="problem-description"></a>Descrição do problema

Quando os dados não estão armazenados em cache, isso pode causar vários comportamentos indesejados, incluindo:

- Buscando repetidamente as mesmas informações de um recurso que é caro para o acesso, em termos de latência ou sobrecarga de E/S.
- Construindo repetidamente os mesmos objetos ou estruturas de dados para várias solicitações.
- Fazendo chamadas excessivas para um serviço remoto que tem uma cota de serviço e limita os clientes após um determinado limite.

Por sua vez, esses problemas podem resultar em tempos de resposta baixos, mais contenção no armazenamento de dados e baixa escalabilidade.

O exemplo a seguir usa o Entity Framework para se conectar a um banco de dados. Cada solicitação de cliente resulta em uma chamada para o banco de dados, mesmo se várias solicitações estejam buscando exatamente os mesmos dados. O custo de solicitações repetidas, em termos de sobrecarga de E/S e encargos de acesso de dados, pode se acumular rapidamente.

```csharp
public class PersonRepository : IPersonRepository
{
    public async Task<Person> GetAsync(int id)
    {
        using (var context = new AdventureWorksContext())
        {
            return await context.People
                .Where(p => p.Id == id)
                .FirstOrDefaultAsync()
                .ConfigureAwait(false);
        }
    }
}
```

Você pode encontrar o exemplo completo [aqui][sample-app].

Esse antipadrão geralmente ocorre porque:

- Não usar um cache é mais simples de implementar, e funciona bem em cargas baixas. O cache torna o código mais complicado.
- As vantagens e desvantagens do uso de um cache não são claramente compreendidas.
- Há uma preocupação quanto à sobrecarga de manter a precisão e a atualização de dados armazenados em cache.
- Um aplicativo foi migrado de um sistema local, em que a latência de rede não era um problema e o sistema foi executado em hardwares mais caros de alto desempenho, para que o cache não fosse considerado no design original.
- Os desenvolvedores não estão cientes de que o cache é uma possibilidade em um determinado cenário. Por exemplo, os desenvolvedores podem não pensar em usar ETags ao implementar uma API da Web.

## <a name="how-to-fix-the-problem"></a>Como corrigir o problema

A estratégia de cache mais popular é a estratégia *sob demanda* ou *cache-aside*.

- Na leitura, o aplicativo tenta ler os dados do cache. Se os dados não estiverem no cache, o aplicativo recupera-o da fonte de dados e o adiciona ao cache.
- Na gravação, o aplicativo grava a alteração diretamente na fonte de dados e remove o valor antigo do cache. Ele será recuperado e adicionado ao cache da próxima vez que for necessário.

Essa abordagem é adequada para dados que são alterados com frequência. Aqui está o exemplo anterior atualizado para usar o padrão [Cache-Aside][cache-aside-pattern].

```csharp
public class CachedPersonRepository : IPersonRepository
{
    private readonly PersonRepository _innerRepository;

    public CachedPersonRepository(PersonRepository innerRepository)
    {
        _innerRepository = innerRepository;
    }

    public async Task<Person> GetAsync(int id)
    {
        return await CacheService.GetAsync<Person>("p:" + id, () => _innerRepository.GetAsync(id)).ConfigureAwait(false);
    }
}

public class CacheService
{
    private static ConnectionMultiplexer _connection;

    public static async Task<T> GetAsync<T>(string key, Func<Task<T>> loadCache, double expirationTimeInMinutes)
    {
        IDatabase cache = Connection.GetDatabase();
        T value = await GetAsync<T>(cache, key).ConfigureAwait(false);
        if (value == null)
        {
            // Value was not found in the cache. Call the lambda to get the value from the database.
            value = await loadCache().ConfigureAwait(false);
            if (value != null)
            {
                // Add the value to the cache.
                await SetAsync(cache, key, value, expirationTimeInMinutes).ConfigureAwait(false);
            }
        }
        return value;
    }
}
```

Observe que o método `GetAsync` agora chama a classe `CacheService`, em vez de chamar diretamente o banco de dados. Primeiro, a classe `CacheService` tenta obter o item do Cache Redis do Azure. Se o valor não for encontrado no Cache Redis, o `CacheService` invoca uma função lambda que foi passada pelo chamador. A função lambda é responsável por buscar os dados do banco de dados. Essa implementação separa o repositório da solução de cache específica e separa o `CacheService` do banco de dados.

## <a name="considerations"></a>Considerações

- Se o cache estiver indisponível, talvez devido a uma falha transitória, não retorne um erro para o cliente. Em vez disso, busque os dados da fonte de dados original. Mas, lembre-se: durante a recuperação do cache, o armazenamento de dados original pode ser inundado com solicitações, resultando em tempos limite e conexões com falha. (Afinal, isso é uma das motivações para usar um cache em primeiro lugar.) Use uma técnica, como o [Padrão de Disjuntor][circuit-breaker] para evitar sobrecarregar a fonte de dados.

- Aplicativos que armazenam em cache dados não estáticos devem ser criados para oferecer suporte à consistência eventual.

- Para APIs da Web, você pode oferecer suporte ao cache do cliente, incluindo um cabeçalho Cache-Control nas mensagens de solicitação e resposta e usar ETags para identificar as versões de objetos. Para obter mais informações, confira [Implementação da API][api-implementation].

- Você não precisa colocar em cache as entidades completas. Se grande parte de uma entidade for estática, mas somente uma pequena parte for alterada com frequência, coloque em cache os elementos estáticos e recupere os elementos dinâmicos da fonte de dados. Essa abordagem pode ajudar a reduzir o volume de E/S que está sendo executado em relação à fonte de dados.

- Em alguns casos, se os dados voláteis forem de curta duração, pode ser útil para colocar em cache. Por exemplo, considere um dispositivo que envia continuamente as atualizações de status. Pode fazer sentido armazenar em cache essas informações conforme elas chegam e não gravá-las em um repositório persistente.

- Para impedir que os dados se tornem obsoletos, muitas soluções de cache oferecem suporte a períodos de validade configurável, para que os dados sejam removidos do cache automaticamente após um intervalo especificado. Talvez seja necessário ajustar o tempo de expiração para seu cenário. Os dados que são altamente estáticos podem permanecer no cache por períodos mais longos que os dados voláteis que podem se tornar obsoletos rapidamente.

- Se a solução de cache não fornecer expiração interna, você precisará implementar um processo em segundo plano que ocasionalmente varre o cache, para impedir que ele cresça sem limites.

- Além de armazenar em cache os dados de uma fonte de dados externa, você pode usar o cache para salvar os resultados de cálculos complexos. Antes de fazer isso, no entanto, instrumente o aplicativo para determinar se o aplicativo está realmente associado à CPU.

- Pode ser útil preparar o cache quando o aplicativo for iniciado. Popule o cache com os dados que são mais prováveis de serem utilizados.

- Inclua sempre instrumentação que detecta ocorrências no cache e perdas no cache. Use essas informações para ajustar as políticas de cache, tais como: quais dados armazenar em cache e por quanto tempo manter os dados no cache antes de expirar.

- Se a falta de armazenamento em cache for um gargalo, adicionar o cache pode aumentar o volume de solicitações de tal forma que o front-end da Web fique sobrecarregado. Os clientes podem começar a receber erros HTTP 503 (Serviço Indisponível). Essas são indicações de que você deve escalar horizontalmente o front-end.

## <a name="how-to-detect-the-problem"></a>Como detectar o problema

Você pode executar as etapas a seguir para ajudar a identificar se a falta de cache está causando problemas de desempenho:

1. Examine o design do aplicativo. Faça um inventário de todos os repositórios de dados que o aplicativo utiliza. Para cada um, determine se o aplicativo está usando um cache. Se possível, determine a frequência com a qual os dados são alterados. Bons candidatos inicias para o cache incluem dados que são alterados lentamente e dados de referência estática lidos com frequência.

2. Instrumente o aplicativo e monitore o sistema ao vivo para descobrir a frequência com a qual o aplicativo recupera dados ou calcula informações.

3. Crie um perfil para o aplicativo em um ambiente de teste para capturar métricas de baixo nível sobre a sobrecarga associada com operações de acesso de dados ou outros cálculos frequentemente executados.

4. Execute o teste de carga em um ambiente de teste para identificar como o sistema responde sob uma carga de trabalho normal e sob uma carga pesada. O teste de carga deve simular o padrão de acesso a dados observado no ambiente de produção usando cargas de trabalho reais.

5. Examine as estatísticas de acesso a dados para os armazenamentos de dados subjacentes e examine quantas vezes as mesmas solicitações de dados são repetidas.

## <a name="example-diagnosis"></a>Diagnóstico de exemplo

As seções a seguir aplicam essas etapas ao aplicativo de exemplo descrito anteriormente.

### <a name="instrument-the-application-and-monitor-the-live-system"></a>Instrumentar o aplicativo e monitorar o sistema ao vivo

Instrumente o aplicativo e monitore-o para obter informações sobre as solicitações específicas feitas pelos usuários enquanto o aplicativo está em produção.

A imagem a seguir mostra o monitoramento dados capturados por [New Relic][NewRelic] durante um teste de carga. Nesse caso, a única operação HTTP GET executada é `Person/GetAsync`. Mas, em um ambiente de produção ao vivo, saber a frequência relativa com a qual cada solicitação é executada pode fornecer informações sobre os recursos que devem ser armazenados em cache.

![New Relic mostrando as solicitações do servidor para o aplicativo CachingDemo][NewRelic-server-requests]

Se precisar de uma análise mais profunda, você pode usar um criador de perfil para capturar dados de desempenho de baixo nível em um ambiente de teste (não no sistema de produção). Examine as métricas, como taxas de solicitação de E/S, o uso da memória e a utilização da CPU. Essas métricas podem mostrar um grande número de solicitações para um armazenamento de dados ou serviço, ou processamento repetido que realiza o mesmo cálculo.

### <a name="load-test-the-application"></a>Fazer teste de carga no aplicativo

O gráfico a seguir mostra os resultados do teste de carga para o aplicativo de exemplo. O teste de carga simula uma carga de etapa de até 800 usuários executando uma série típica de operações.

![Resultados do teste de carga de desempenho para o cenário sem cache][Performance-Load-Test-Results-Uncached]

O número de testes bem-sucedidos executados a cada segundo atinge um limite e, como resultado, as solicitações adicionais são desaceleradas. O tempo médio de teste é incrementado continuamente com a carga de trabalho. Os níveis do tempo de resposta são desativados quando o usuário realiza picos de carga.

### <a name="examine-data-access-statistics"></a>Examinar as estatísticas de acesso de dados

As estatísticas de acesso a dados e outras informações fornecidas por um armazenamento de dados pode fornecer informações úteis, tais como: quais consultas são repetidas com mais frequência. Por exemplo, no Microsoft SQL Server, a exibição de gerenciamento `sys.dm_exec_query_stats` tem informações estatísticas para consultas executadas recentemente. O texto de cada consulta está disponível na exibição `sys.dm_exec-query_plan`. Você pode usar uma ferramenta como o SQL Server Management Studio para executar a seguinte consulta SQL e determinar a frequência com a qual as consultas são executadas.

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

A coluna `UseCount` nos resultados indica a frequência com a qual cada consulta é executada. A imagem a seguir mostra que a terceira consulta foi executada mais de 250.000 vezes, significativamente mais do que qualquer outra consulta.

![Resultados da consulta das exibições de gerenciamento dinâmico no Servidor de Gerenciamento do SQL Server][Dynamic-Management-Views]

Aqui está a consulta SQL que está causando tantas solicitações de banco de dados:

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

Esta é a consulta que o Entity Framework gera no método `GetByIdAsync` mostrado anteriormente.

### <a name="implement-the-solution-and-verify-the-result"></a>Implementar a solução e verificar o resultado

Depois que você incorporar um cache, repita os testes de carga e compare os resultados para os testes de carga anteriores sem um cache. Aqui estão os resultados do teste de carga após a adição de um cache ao aplicativo de exemplo.

![Resultados do teste de carga de desempenho para o cenário com cache][Performance-Load-Test-Results-Cached]

O volume de testes bem-sucedido ainda atinge um limite, mas em uma carga de usuário maior. A taxa de solicitação com essa carga é significativamente maior do que a anterior. O tempo médio de teste ainda aumenta com a carga, mas o tempo de resposta máximo é 0,05 ms, comparados com 1 ms anterior a um aperfeiçoamento &mdash; 20&times;.

## <a name="related-resources"></a>Recursos relacionados

- [Práticas recomendadas de implementação de API][api-implementation]
- [Padrão Cache-Aside][cache-aside-pattern]
- [Práticas recomendadas do cache][caching-guidance]
- [Padrão de Disjuntor][circuit-breaker]

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/NoCaching
[cache-aside-pattern]: /azure/architecture/patterns/cache-aside
[caching-guidance]: ../../best-practices/caching.md
[circuit-breaker]: ../../patterns/circuit-breaker.md
[api-implementation]: ../../best-practices/api-implementation.md#optimizing-client-side-data-access
[NewRelic]: https://newrelic.com/partner/azure
[NewRelic-server-requests]: _images/New-Relic.jpg
[Performance-Load-Test-Results-Uncached]:_images/InitialLoadTestResults.jpg
[Dynamic-Management-Views]: _images/SQLServerManagementStudio.jpg
[Performance-Load-Test-Results-Cached]: _images/CachedLoadTestResults.jpg
