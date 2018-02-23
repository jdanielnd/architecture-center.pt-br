---
title: "Antipadrão de instanciação inadequada"
description: "Evite criar continuamente novas instâncias de um objeto que deve ser criado somente uma vez e depois compartilhado."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 4d5ef9ad9e675b46df94b51e81d7a4bd4c1b25e9
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/23/2018
---
# <a name="improper-instantiation-antipattern"></a>Antipadrão de instanciação inadequada

Criar continuamente novas instâncias de um objeto que deve ser criado somente uma vez e depois compartilhado pode prejudicar o desempenho. 

## <a name="problem-description"></a>Descrição do problema

Muitas bibliotecas fornecem abstrações de recursos externos. Internamente, essas classes normalmente gerenciam suas próprias conexões ao recurso, atuando como agentes que os clientes podem usar para acessar o recurso. Aqui estão alguns exemplos de classes de agente que são relevantes para aplicativos do Azure:

- `System.Net.Http.HttpClient`. Se comunica com um serviço Web usando HTTP.
- `Microsoft.ServiceBus.Messaging.QueueClient`. Envia e recebe mensagens para uma fila do Barramento de Serviço. 
- `Microsoft.Azure.Documents.Client.DocumentClient`. Conecta-se a uma instância do Cosmos DB
- `StackExchange.Redis.ConnectionMultiplexer`. Conecta-se ao Redis, incluindo o Cache Redis do Azure.

Essas classes são destinadas a serem instanciados uma vez e reutilizadas em todo o tempo de vida de um aplicativo. No entanto, é comum cometer o equívoco de achar que essas classes devem ser adquiridas somente quando necessário e lançadas rapidamente. (As classes listadas são de bibliotecas .NET, mas o padrão não é exclusivo para .NET.) O ASP.NET de exemplo a seguir cria uma instância de `HttpClient` para se comunicar com um serviço remoto. Você pode encontrar o exemplo completo [aqui][sample-app].

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

Em um aplicativo Web, essa técnica não será escalonável. Um novo objeto `HttpClient` é criado para cada solicitação de usuário. Sob carga pesada, o servidor Web pode esgotar o número de soquetes disponíveis, resultando em erros `SocketException`.

Esse problema não está restrito à classe `HttpClient`. Outras classes que encapsulam recursos ou que são caros de criar podem causar problemas semelhantes. O exemplo a seguir cria uma instância da classe `ExpensiveToCreateService`. Aqui o problema não é necessariamente o esgotamento de soquete, mas simplesmente quanto tempo leva para criar cada instância. Criar e destruir continuamente instâncias dessa classe pode prejudicar a escalabilidade do sistema.

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a>Como corrigir o problema

Se a classe que encapsula o recurso externo for compartilhável e thread-safe, crie uma instância singleton compartilhada ou um pool de instâncias reutilizáveis da classe.

O seguinte exemplo usa uma instância `HttpClient` estática, compartilhamento, portanto, a conexão entre todas as solicitações.

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient httpClient;

    static SingleHttpClientInstanceController()
    {
        httpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a>Considerações

- O elemento chave desse antipadrão é criar e destruir repetidamente as instâncias de um objeto *compartilhável*. Se a classe não for compartilhável (não é thread-safe), então esse antipadrão não se aplica.

- O tipo de recurso compartilhado pode determinam se você deve usar um singleton ou criar um pool. A classe `HttpClient` foi projetada para ser compartilhada em vez de ser colocada em pool. Outros objetos podem oferecer suporte a pool, permitindo que o sistema distribua a carga de trabalho entre várias instâncias.

- Os objetos que você compartilha entre várias solicitações *devem* ser thread-safe. A classe `HttpClient` foi projetada para ser usada dessa maneira, mas outras classes podem não oferecer suporte a solicitações simultâneas, portanto, verifique a documentação disponível.

- Alguns tipos de recurso são escassos e não devem ser mantidos. Conexões de banco de dados são um exemplo. Manter uma conexão de banco de dados aberto que não seja necessária pode impedir que outros usuários simultâneos tenham acesso ao banco de dados.

- No .NET Framework, muitos objetos estabelecem conexões com recursos externos são criados usando os métodos de fábrica estáticos de outras classes que gerenciam essas conexões. Esses objetos de fábricas destinam-se a ser salvos e reutilizados, em vez de descartados e recriados. Por exemplo, no Barramento de Serviço do Azure, o objeto `QueueClient` é criado por meio de um objeto `MessagingFactory`. Internamente, o `MessagingFactory` gerencia as conexões. Para saber mais informações, consulte [Práticas recomendadas para melhorias de desempenho usando o Sistema de Mensagens do Barramento de Serviço][service-bus-messaging].

## <a name="how-to-detect-the-problem"></a>Como detectar o problema

Sintomas desse problema incluem uma queda na taxa de transferência ou um aumento na taxa de erro, junto com um ou mais dos sintomas a seguir: 

- Um aumento nas exceções que indica o esgotamento de recursos, como soquetes, conexões de banco de dados, identificadores de arquivos e assim por diante. 
- Um aumento no uso de memória e na coleta de lixo.
- Um aumento na atividade de banco de dados, disco ou rede.

Você pode executar as etapas a seguir para ajudar a identificar o problema:

1. Executar o monitoramento de processos do sistema de produção, para identificar os pontos onde os tempos de resposta ficam mais lentos ou onde o sistema falha devido à falta de recursos.
2. Examine os dados de telemetria capturados nesses pontos para determinar quais operações podem estar criando e destruindo os objetos que consomem recursos.
3. Faça o teste de carga de cada operação suspeita em um ambiente de teste controlado em vez de em um sistema de produção.
4. Examine o código-fonte e examine como os objetos do agente são gerenciados.

Observe os rastreamentos de pilha para operações que são lentas ou que geram exceções quando o sistema está sob carga. Essas informações podem ajudar a identificar como essas operações estão utilizando os recursos. Exceções podem ajudar a determinar se os erros são causados por recursos compartilhados sendo esgotados. 

## <a name="example-diagnosis"></a>Diagnóstico de exemplo

As seções a seguir aplicam essas etapas ao aplicativo de exemplo descrito anteriormente.

### <a name="identify-points-of-slow-down-or-failure"></a>Identificar pontos de lentidão ou falha

A imagem a seguir mostra os resultados gerados usando o [Gerenciamento de Desempenho de Aplicativos (APM) da New Relic][new-relic], mostrando também as operações que têm um tempo de resposta ruim. Nesse caso, vale a pena fazer uma investigação mais detalhada ao método `GetProductAsync` no controlador `NewHttpClientInstancePerRequest`. Observe que a taxa de erros também aumenta quando essas operações estão em execução. 

![O painel do monitor da New Relic mostra o aplicativo de exemplo criando uma nova instância de um objeto HttpClient para cada solicitação][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a>Examinar os dados de telemetria e localizar correlações

A imagem seguinte mostra os dados capturados usando a criação de perfil de thread, no mesmo período correspondente à imagem anterior. O sistema gasta um tempo significativo abrindo as conexões de soquete e ainda mais tempo fechando-as e tratando as exceções de soquete.

![O criador de perfil de thread da New Relic mostra o aplicativo de exemplo criando uma nova instância de um objeto HttpClient para cada solicitação][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a>Executar o teste de carga

Use o teste de carga para simular as operações comuns que podem ser executadas pelos usuários. Isso pode ajudar a identificar quais partes de um sistema são afetadas por um esgotamento de recursos sob cargas diferentes. Execute esses testes em um ambiente controlado em vez de um sistema de produção. O gráfico a seguir mostra a taxa de transferência de solicitações manipulada pelo controlador `NewHttpClientInstancePerRequest`, já que a carga de usuários aumenta para 100 usuários simultâneos.

![Taxa de transferência do mesmo aplicativo criando uma nova instância de um objeto HttpClient para cada solicitação][throughput-new-HTTPClient-instance]

Primeiro, o volume de solicitações tratadas por segundo aumenta conforme aumenta a carga de trabalho. Depois de aproximadamente 30 usuários, no entanto, o volume de solicitações bem sucedidas atinge um limite e o sistema começa a gerar exceções. Daí em diante, o volume de exceções aumenta gradualmente junto com a carga de usuário. 

O teste de carga relatou essas falhas como erros HTTP 500 (servidor interno). Uma revisão da telemetria mostrou que esses erros foram causados pela execução do sistema sem recursos de soquete, já que cada vez mais objetos `HttpClient` foram criados.

O gráfico seguinte mostra um teste semelhante para um controlador que cria o objeto `ExpensiveToCreateService` personalizado.

![Taxa de transferência do mesmo aplicativo criando uma nova instância do ExpensiveToCreateService para cada solicitação][throughput-new-ExpensiveToCreateService-instance]

Neste momento, o controlador não gera exceções, mas a taxa de transferência ainda atinge um limite, enquanto o tempo médio de resposta aumenta em um fator de 20. (O gráfico usa uma escala logarítmica para o tempo de resposta e a taxa de transferência). A telemetria mostrou que criar novas instâncias de `ExpensiveToCreateService` foi a causa principal do problema.

### <a name="implement-the-solution-and-verify-the-result"></a>Implementar a solução e verificar o resultado

Depois de alternar o método `GetProductAsync` para compartilhar uma única instância `HttpClient`, um segundo teste de carga mostrou um melhor desempenho. Nenhum erro foi relatado, e o sistema foi capaz de lidar com um aumento da carga de até 500 solicitações por segundo. O tempo médio de resposta diminuiu pela metade, em comparação com o teste anterior.

![Taxa de transferência do mesmo aplicativo reutilizando a mesma instância de um objeto HttpClient para cada solicitação][throughput-single-HTTPClient-instance]

Para comparação, a imagem a seguir mostra a telemetria de rastreamento de pilha. Dessa vez o sistema gasta a maior parte do tempo executando o trabalho real, em vez de abrindo e fechando soquetes.

![O criador de perfil de thread da New Relic mostra o aplicativo de exemplo criando uma única instância de um objeto HttpClient para todas as solicitações][thread-profiler-single-HTTPClient-instance]

O gráfico seguinte mostra um teste de carga semelhante usando uma instância compartilhada do objeto `ExpensiveToCreateService`. Novamente, o volume de solicitações manipuladas aumenta junto com a carga de usuário enquanto o tempo médio de resposta permanece baixo. 

![Taxa de transferência do mesmo aplicativo reutilizando a mesma instância de um objeto HttpClient para cada solicitação][throughput-single-ExpensiveToCreateService-instance]



[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
