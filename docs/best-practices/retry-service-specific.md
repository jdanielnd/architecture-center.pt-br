---
title: Diretriz específica do serviço de repetição
description: Diretriz específica de serviço para configurar o mecanismo de repetição.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 65206c5f39a74d228c7eaa0fea0c5b1b0710b22f
ms.sourcegitcommit: bb348bd3a8a4e27ef61e8eee74b54b07b65dbf98
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/21/2018
ms.locfileid: "34423010"
---
# <a name="retry-guidance-for-specific-services"></a>Repetir as diretrizes para serviços específicos

A maioria dos serviços do Azure e SDKs do cliente inclui um mecanismo de repetição. No entanto, eles são diferentes porque cada serviço apresenta características e requisitos diferentes, e cada mecanismo de repetição é ajustado para um serviço específico. Este guia resume os recursos do mecanismo de repetição para a maioria dos serviços do Azure, além de incluir informações que ajudam a usar, a adaptar ou a estender o mecanismo de repetição para o serviço em questão.

Para obter orientação geral sobre o tratamento de falhas transitórias e como repetir conexões e operações em serviços e recursos, consulte [Diretriz de repetição](./transient-faults.md).

A tabela a seguir resume os recursos de repetição para os serviços do Azure descritos nesta diretriz.

| **Serviço** | **Recursos de repetição** | **Configuração de política** | **Escopo** | **Recursos de telemetria** |
| --- | --- | --- | --- | --- |
| **[Azure Active Directory](#azure-active-directory)** |Nativo na biblioteca ADAL |Incorporado na biblioteca ADAL |Interna |Nenhum |
| **[Cosmos DB](#cosmos-db)** |Nativo no serviço |Não configurável |Global |TraceSource |
| **[Hubs de Eventos](#azure-event-hubs)** |Nativo no cliente |Programático |Cliente |Nenhum |
| **[Cache Redis](#azure-redis-cache)** |Nativo no cliente |Programático |Cliente |TextWriter |
| **[Pesquisa](#azure-search)** |Nativo no cliente |Programático |Cliente |ETW ou Personalizado |
| **[Barramento de Serviço](#service-bus)** |Nativo no cliente |Programático |Gerenciador de Namespace, Messaging Factory e Cliente |ETW |
| **[Service Fabric](#service-fabric)** |Nativo no cliente |Programático |Cliente |Nenhum | 
| **[Banco de dados SQL com ADO.NET](#sql-database-using-adonet)** |[Polly](#transient-fault-handling-with-polly) |Programático e declarativo |Instruções ou blocos de código únicos |Personalizado |
| **[Banco de dados SQL com o Entity Framework](#sql-database-using-entity-framework-6)** |Nativo no cliente |Programático |Global por AppDomain |Nenhum |
| **[Banco de dados SQL com Entity Framework Core](#sql-database-using-entity-framework-core)** |Nativo no cliente |Programático |Global por AppDomain |Nenhum |
| **[Armazenamento](#azure-storage)** |Nativo no cliente |Programático |Operações individuais e de cliente |TraceSource |

> [!NOTE]
> Para a maioria dos mecanismos de repetição internos do Azure, atualmente não há meios de aplicar uma política de repetição diferente para diversos tipos de erro ou exceção. Você deve configurar uma política que fornece o melhor desempenho médio e disponibilidade. Uma maneira de ajustar a política é analisar arquivos de log para determinar o tipo de falha transitória que está ocorrendo. 

## <a name="azure-active-directory"></a>Azure Active Directory
O Azure Active Directory (Azure AD) é uma solução abrangente de nuvem para gerenciamento de acesso e identidade que combina serviços principais de diretório, governança avançada de identidade, segurança e gerenciamento de acesso a aplicativos. O AD do Azure também oferece aos desenvolvedores uma plataforma de gerenciamento de identidade para fornecer controle de acesso aos respectivos aplicativos, com base nas regras e políticas centralizadas.

### <a name="retry-mechanism"></a>Mecanismo de repetição
Na Biblioteca de autenticação do Active Directory (ADAL) há um mecanismo interno de repetição para o Azure Active Directory. Para evitar bloqueios inesperados, recomendamos que o ADAL lide com as repetições e **não** permita que as bibliotecas de terceiros ou o código do aplicativo repitam as conexões com falhas. 

### <a name="retry-usage-guidance"></a>Diretriz de uso de repetição
Considere as seguintes diretrizes ao usar o Azure Active Directory:

* Quando possível, use a biblioteca ADAL e o suporte interno para repetições.
* Se estiver a usar a API REST do Azure Active Directory, tente novamente a operação se o código de resultado for 429 (Muitos Pedidos) ou for um erro no intervalo 5xx. Não repita para nenhum outro erro.
* Uma política de retirada exponencial é recomendada para uso em cenários de lote com o Azure Active Directory.

Considere começar com as seguintes configurações para operações de repetição. Essas são configurações de uso geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.

| **Contexto** | **Exemplo de latênciaE2E<br />máxima de destino** | **Estratégia de repetição** | **Configurações** | **Valores** | **Como funciona** |
| --- | --- | --- | --- | --- | --- |
| Interativo, interface de usuário<br />ou primeiro plano |2 s |FixedInterval |Contagem de repetição<br />Intervalo de repetição<br />Primeira repetição rápida |3<br />500 ms<br />verdadeiro |1ª tentativa — intervalo de 0 s<br />2ª tentativa — intervalo de 500 ms<br />3ª tentativa — intervalo de 500 ms |
| Segundo plano ou<br />lote |60 s |ExponentialBackoff |Contagem de repetição<br />Retirada mín.<br />Retirada máx.<br />Retirada delta<br />Primeira repetição rápida |5<br />0 s<br />60 s<br />2 s<br />falso |1ª tentativa — intervalo de 0 s<br />2ª tentativa — intervalo de ~2 s<br />3ª tentativa — intervalo de ~6 s<br />4ª tentativa — intervalo de ~14 s<br />5ª tentativa — intervalo de ~30 s |

### <a name="more-information"></a>Mais informações
* [Bibliotecas de autenticação do Azure Active Directory][adal]

## <a name="cosmos-db"></a>Cosmos DB

O Cosmos DB é um banco de dados multimodelo totalmente gerenciado que oferece suporte a dados JSON sem esquema. Ele oferece desempenho configurável e confiável, processamento transacional nativo JavaScript, além de ser criado para a nuvem com escala elástica.

### <a name="retry-mechanism"></a>Mecanismo de repetição
A classe `DocumentClient` repete automaticamente tentativas com falha. Para definir o número de repetições e o tempo de espera máximo, configure [ConnectionPolicy.RetryOptions]. As exceções que o cliente gera excedem a política de repetição ou não são erros transitórios.

Se o Cosmos DB limitar o cliente, ele retornará um erro HTTP 429. Confira o código de status no `DocumentClientException`.

### <a name="policy-configuration"></a>Configuração de política
A tabela a seguir mostra as configurações padrão para a classe `RetryOptions`.

| Configuração | Valor padrão | DESCRIÇÃO |
| --- | --- | --- |
| MaxRetryAttemptsOnThrottledRequests |9 |O número máximo de tentativas se a solicitação falhar porque o Cosmos DB aplicou a limitação de taxa ao cliente. |
| MaxRetryWaitTimeInSeconds |30 |O tempo máximo de repetição, em segundos. |

### <a name="example"></a>Exemplo
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a>Telemetria
As tentativas de repetição são registradas como mensagens de rastreamento não estruturadas por meio de um **TraceSource**do .NET. Você deve configurar um **TraceListener** para capturar os eventos e gravá-los em um log de destino apropriado.

Por exemplo, se você adicionar o seguinte ao arquivo App.config, rastreamentos serão gerados em um arquivo de texto no mesmo local que o executável:

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

## <a name="event-hubs"></a>Hubs de Eventos

Os Hubs de Eventos do Azure são um serviço de ingestão de telemetria de hiperescala que coleta, transforma e armazena milhões de eventos.

### <a name="retry-mechanism"></a>Mecanismo de repetição
O comportamento de repetição na biblioteca de cliente dos Hubs de Eventos do Azure é controlado pela propriedade `RetryPolicy` na classe `EventHubClient`. As tentativas de política padrão com retirada exponencial quando o Hub de eventos do Azure retorna um `EventHubsException` transitório ou um `OperationCanceledException`.

### <a name="example"></a>Exemplo
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a>Mais informações
[Biblioteca de cliente padrão .NET para Hubs de Eventos do Azure](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="azure-redis-cache"></a>Cache Redis do Azure
O Cache Redis do Azure é um serviço de cache de baixa latência e acesso a dados rápido com base no Cache Redis de software livre popular. Ele é seguro, gerenciado pela Microsoft e pode ser acessado de qualquer aplicativo no Azure.

A diretriz nesta seção se baseia em como usar o cliente StackExchange.Redis para acessar o cache. Uma lista de outros clientes adequados pode ser encontrada no [site do Redis](http://redis.io/clients), e eles podem ter mecanismos de repetição diferentes.

Observe que o cliente StackExchange.Redis usa multiplexação por meio de uma única conexão. O uso recomendado é criar uma instância do cliente na inicialização do aplicativo e usar essa instância para todas as operações no cache. Por esse motivo, a conexão com o cache é feita apenas uma vez, e toda a orientação desta seção está relacionada à política de repetição para esta conexão inicial, e não para cada operação que acessa o cache.

### <a name="retry-mechanism"></a>Mecanismo de repetição
O cliente StackExchange.Redis usa uma classe de gerenciador de conexões que é configurada por meio de um conjunto de opções, incluindo:

- **ConnectRetry**. O número de vezes que uma conexão com falha para o cache será repetida.
- **ReconnectRetryPolicy**. Usar qual estratégia de repetição.
- **ConnectTimeout**. O tempo máximo de espera em milissegundos.

### <a name="policy-configuration"></a>Configuração de política
As políticas de repetição são configuradas de modo programático, definindo as opções para o cliente antes de se conectar ao cache. Isso pode ser feito criando uma instância da classe **ConfigurationOptions**, populando suas propriedades e passando-a para o método **Connect**.

As classes internas suportam atraso linear (constante) e retirada exponencial com intervalos de repetição aleatórios. Você também pode criar uma política de repetição personalizada se implementar a interface **IReconnectRetryPolicy**.

O exemplo a seguir configura uma estratégia de repetição com retirada exponencial.

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

Como alternativa, você pode especificar as opções como uma cadeia de caracteres e passá-la ao método **Connect**. Observe que a propriedade **ReconnectRetryPolicy** não pode ser definida dessa forma, somente através de código.

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Também é possível especificar opções diretamente ao se conectar ao cache.

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

Para obter mais informações, confira [Configuração do Stack Exchange Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration) na documentação do StackExchange.Redis.

A tabela a seguir mostra as configurações padrão da política de repetição interna.

| **Contexto** | **Configuração** | **Valor padrão**<br />(v 1.2.2) | **Significado** |
| --- | --- | --- | --- |
| ConfigurationOptions |ConnectRetry<br /><br />ConnectTimeout<br /><br />SyncTimeout<br /><br />ReconnectRetryPolicy |3<br /><br />Máximo de 5000 ms mais SyncTimeout<br />1000<br /><br />LinearRetry 5.000 ms |O número de vezes para repetir tentativas de conexão durante a operação de conexão inicial.<br />Tempo limite (ms) para operações de conexão. Não é um intervalo entre tentativas de repetição.<br />Tempo (ms) para permitir operações síncronas.<br /><br />Repita a cada 5.000 ms.|

> [!NOTE]
> Em operações síncronas, `SyncTimeout` pode adicionar à latência de ponta a ponta, porém definir o valor muito baixo pode causar tempos limite excessivos. Confira [Como solucionar problemas do Cache Redis do Azure][redis-cache-troubleshoot]. Em geral, evite usar operações síncronas e use operações assíncronas. Para saber mais, consulte [Pipelines e multiplexadores](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).
>
>

### <a name="retry-usage-guidance"></a>Diretriz de uso de repetição
Considere as seguintes diretrizes ao usar o Cache Redis do Azure:

* O cliente StackExchange Redis gerencia suas próprias repetições, mas apenas ao estabelecer uma conexão com o cache quando o aplicativo é iniciado pela primeira vez. Você pode configurar o tempo limite da conexão, o número de repetições e o intervalo entre as repetições para estabelecer esta conexão, porém a política de repetições não se aplica a operações contra o cache.
* Em vez de usar um grande número de tentativas de repetição, considere fazer fallback acessando a fonte de dados original.

### <a name="telemetry"></a>Telemetria
Você pode coletar informações sobre conexões (mas não outras operações) usando um **TextWriter**.

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Um exemplo da saída que isso gera é mostrado abaixo.

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

### <a name="examples"></a>Exemplos
O exemplo de código a seguir configura um atraso constante (linear) entre as repetições ao inicializar o cliente StackExchange.Redis. Este exemplo mostra como definir a configuração usando uma instância de **ConfigurationOptions**.

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

Este exemplo mostra como definir a configuração especificando as opções como uma cadeia de caracteres. O tempo limite da conexão é o período máximo de espera por uma conexão para o cache, não o atraso entre tentativas de repetição. Observe que a propriedade **ReconnectRetryPolicy** só pode ser definida por código.

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

Para obter mais exemplos, consulte [Configuração](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) no site do projeto.

### <a name="more-information"></a>Mais informações
* [Site do Redis](http://redis.io/)

## <a name="azure-search"></a>Azure Search
O Azure Search pode ser usada para adicionar recursos potentes e sofisticados a um site ou aplicativo, ajustar os resultados da pesquisa de maneira rápida e fácil, bem como construir modelos de classificação avançados e ajustados.

### <a name="retry-mechanism"></a>Mecanismo de repetição
O comportamento de repetição no SDK do Azure Search é controlado pelo método `SetRetryPolicy` nas classes [SearchServiceClient] e [SearchIndexClient]. A política padrão tenta novamente com retirada exponencial quando o Azure Search retorna uma resposta 5xx ou 408 (Tempo Limite da Solicitação).

### <a name="telemetry"></a>Telemetria
Rastreamento com o ETW ou por meio de registro de um provedor de rastreamento personalizado. Para obter mais informações, confira [Documentação do AutoRest][autorest].

## <a name="service-bus"></a>Barramento de Serviço
O Barramento de Serviço é uma plataforma de mensagens na nuvem que fornece troca de mensagens flexivelmente acoplada com escala e resiliência melhoradas para componentes de um aplicativo, se hospedado na nuvem ou no local.

### <a name="retry-mechanism"></a>Mecanismo de repetição
O Barramento de Serviço implementa repetições usando implementações da classe base [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) . Todos os clientes do Barramento de Serviço expõem uma propriedade **RetryPolicy** que pode ser definida como uma das implementações da classe base **RetryPolicy**. As implementações internas são:

* A [classe RetryExponential](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx). Ela expõe propriedades que controlam o intervalo de retirada, a contagem de repetições e a propriedade **TerminationTimeBuffer** que é usada para limitar o tempo total para a operação ser concluída.
* A [classe NoRetry](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx). É usada quando as repetições no nível de API do Barramento de Serviço não são exigidas, como quando são gerenciadas por outro processo como parte de uma operação em lote ou de várias etapas.

As ações do Barramento de Serviço podem retornar uma gama de exceções, conforme listado no [Apêndice: exceções de mensagens](http://msdn.microsoft.com/library/hh418082.aspx). A lista fornece informações que indicam se repetir a operação é adequado. Por exemplo, um [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) indica que o cliente deve esperar um período de tempo, depois repetir a operação. A ocorrência de um **ServerBusyException** também faz com que o Barramento de Serviço alterne para um modo diferente, no qual um intervalo extra de 10 segundos é adicionado aos intervalos de repetição computados. Esse modo é redefinido após um curto período.

As exceções retornadas do Barramento de Serviço expõem a propriedade **IsTransient** que indica se o cliente deve repetir a operação. A política interna **RetryExponential** depende da propriedade **IsTransient** na classe **MessagingException**, que é a classe base para todas as exceções do Barramento de Serviço. Se você criar implementações personalizadas da classe base **RetryPolicy**, será possível usar uma combinação do tipo de exceção e da propriedade **IsTransient** para fornecer um controle mais refinado sobre as ações de repetição. Por exemplo, é possível detectar um **QuotaExceededException** e tomar providências para diminuir a fila antes de tentar enviar novamente uma mensagem para ela.

### <a name="policy-configuration"></a>Configuração de política
As políticas de repetição são definidas de modo programático e como uma política padrão para um **NamespaceManager** e uma **MessagingFactory** ou individualmente para cada cliente de mensagens. Para definir a política de repetição padrão para uma sessão de mensagens, defina **RetryPolicy** do **NamespaceManager**.

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

Para definir a política de repetição padrão para todos os clientes criados de uma fábrica de mensagens, defina **RetryPolicy** da **MessagingFactory**.

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

Para definir a política de repetição para um cliente de mensagens, ou para substituir sua política padrão, defina a propriedade **RetryPolicy** usando uma instância da classe de política necessária:

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

A política de repetição não pode ser definida no nível de operação individual. Ela se aplica a todas as operações do cliente de mensagens.
A tabela a seguir mostra as configurações padrão da política de repetição interna.

| Configuração | Valor padrão | Significado |
|---------|---------------|---------|
| Política | Exponencial | Retirada exponencial. |
| MinimalBackoff | 0 | O intervalo mínimo de retirada. Isso é adicionado ao intervalo de repetição calculados de deltaBackoff. |
| MaximumBackoff | 30 segundos | O intervalo máximo de retirada. MaximumBackoff será usado se o intervalo de repetição calculado for maior que MaxBackoff. |
| DeltaBackoff | 3 Segundos | Intervalo de retirada entre repetições. Múltiplos desse período de tempo serão usados para tentativas de repetição subsequentes. |
| TimeBuffer | 5 segundos | O buffer de tempo de encerramento associado à repetição. Tentativas de repetição serão abandonadas se o tempo restante for menor do que o TimeBuffer. |
| MaxRetryCount | 10 | O número máximo de repetições. |
| ServerBusyBaseSleepTime | 10 segundos | Se a última exceção encontrada for **ServerBusyException**, esse valor será adicionado ao intervalo de repetição calculado. Esse valor não pode ser alterado. |

### <a name="retry-usage-guidance"></a>Diretriz de uso de repetição
Considere as seguintes diretrizes ao usar o Barramento de Serviço:

* Ao usar a implementação **RetryExponential** interna, não implemente uma operação de fallback, pois a política reage às exceções de Servidor Ocupado e alterna automaticamente para um modo de repetição apropriado.
* O Barramento de Serviço oferece suporte a um recurso chamado Namespaces Emparelhados, que implementa failover automático para uma fila de backup em um namespace separado, caso a fila no namespace primário falhe. As mensagens da fila secundária podem ser enviadas de volta para a fila primária quando esta se recuperar. Esse recurso ajuda a corrigir falhas transitórias. Para saber mais, consulte [Padrões de mensagens assíncronas e alta disponibilidade](http://msdn.microsoft.com/library/azure/dn292562.aspx).

Considere começar com as seguintes configurações para operações de repetição. Essas são configurações de uso geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.

| Contexto | Exemplo de latência máxima | Política de repetição | Configurações | Como ele funciona |
|---------|---------|---------|---------|---------|
| Interativo, interface do usuário ou primeiro plano | 2 segundos*  | Exponencial | MinimumBackoff = 0 <br/> MaximumBackoff = 30 s <br/> DeltaBackoff = 300 ms <br/> TimeBuffer = 300 ms <br/> MaxRetryCount = 2 | 1ª tentativa: atraso 0 s <br/> 2ª tentativa: atraso ~300 ms <br/> 3ª tentativa: atraso ~900 ms |
| Segundo plano ou lote | 30 segundos | Exponencial | MinimumBackoff = 1 <br/> MaximumBackoff = 30 s <br/> DeltaBackoff = 1,75 seg. <br/> TimeBuffer = 5 seg. <br/> MaxRetryCount = 3 | 1ª tentativa: atraso de ~1 s <br/> 2ª tentativa: atraso de ~3 s <br/> 3ª tentativa: atraso ~6 ms <br/> 4ª tentativa: atraso ~13 ms |

\*Não incluindo atraso que será adicionado se uma resposta Servidor Ocupado for recebida.

### <a name="telemetry"></a>Telemetria
O Barramento de Serviço registra as repetições como eventos ETW usando um **EventSource**. Você deve anexar um **EventListener** à origem do evento para capturar os eventos e exibi-los no Visualizador de Desempenho ou gravá-los em um log de destino apropriado. Você pode usar o [Bloco de Aplicativos para Registro Semântico](http://msdn.microsoft.com/library/dn775006.aspx) para fazer isso. Os eventos de repetição são da seguinte forma:

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

### <a name="examples"></a>Exemplos
O código de exemplo a seguir mostra como definir a política de repetição para:

* Um gerenciador de namespaces. A política se aplica a todas as operações nesse gerenciador e não pode ser substituída para operações individuais.
* Uma fábrica de mensagens. A política se aplica a todos os clientes criados nessa fábrica e não pode ser substituída durante a criação de clientes individuais.
* Um cliente de mensagens individual. Depois que um cliente tiver sido criado, você poderá definir a política de repetição para esse cliente. A política se aplica a todas as operações nesse cliente.

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

### <a name="more-information"></a>Mais informações
* [Padrões de mensagens assíncronas e alta disponibilidade](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="service-fabric"></a>Service Fabric

Distribuir serviços confiáveis em um cluster do Service Fabric protege contra a maioria das falhas transitórias potenciais discutidas neste artigo. No entanto, algumas falhas transitórias ainda podem acontecer. Por exemplo, se o serviço de nomeação recebe uma solicitação no meio de uma alteração de roteamento, ele pode lançar uma exceção. No entanto, se a mesma solicitação vier 100 milissegundos mais tarde, ela provavelmente terá êxito.

Internamente, o Service Fabric gerencia esse tipo de falha transitória. Você pode definir algumas configurações usando a classe `OperationRetrySettings` ao configurar seus serviços.  O código a seguir mostra um exemplo. Na maioria dos casos, isso não deve ser necessário e as configurações padrão são suficientes.

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

### <a name="more-information"></a>Mais informações

* [Manipulação de exceção remota](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a>Banco de Dados SQL usando o ADO.NET
O Banco de Dados SQL é um banco de dados SQL disponível em vários tamanhos e como um serviço padrão (compartilhado) e premium (não compartilhado).

### <a name="retry-mechanism"></a>Mecanismo de repetição
O Banco de Dados SQL não tem suporte interno para repetições quando acessado usando o ADO.NET. No entanto, os códigos de retorno das solicitações podem ser usados para determinar por que uma solicitação falhou. Para obter mais informações sobre a limitação do banco de dados SQL, confira [Limites de recursos do banco de dados SQL do Azure](/azure/sql-database/sql-database-resource-limits). Para obter uma lista dos códigos de erro relevantes, confira [códigos de erro SQL para aplicativos de cliente do banco de dados SQL](/azure/sql-database/sql-database-develop-error-messages).

Você pode usar a biblioteca Polly para implementar tentativas para o banco de dados SQL. Confira [Falha transitória manipulando com a Polly](#transient-fault-handling-with-polly).

### <a name="retry-usage-guidance"></a>Diretriz de uso de repetição
Considere as seguintes diretrizes ao acessar o Banco de Dados SQL usando o ADO.NET:

* Escolha a opção de serviço apropriada (compartilhada ou premium). Uma instância compartilhada pode sofrer atrasos e limitação de conexão mais longos que o normal devido ao uso por outros locatários do servidor compartilhado. Se forem necessárias mais operações confiáveis de baixa latência e desempenho previsível, considere a escolha da opção premium.
* Garanta a execução de repetições no nível ou escopo apropriado para evitar operações não idempotentes que causem inconsistência nos dados. De modo ideal, todas as operações devem ser idempotentes para que possam ser repetidas sem causar inconsistência. Quando não for esse o caso, a repetição deverá ser realizada em um nível ou escopo que permita que todas as alterações relacionadas sejam desfeitas, caso uma operação falhe; por exemplo, em um escopo transacional. Para saber mais, consulte [Camada de acesso a dados dos conceitos básicos do serviço de nuvem — tratamento de falhas transitórias](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).
* Uma estratégia de intervalo fixo não é recomendada para uso com o Banco de Dados SQL do Azure, exceto em cenários interativos em que há apenas algumas repetições em intervalos muito curtos. Em vez disso, considere usar uma estratégia de retirada exponencial para a maioria dos cenários.
* Escolha um valor adequado para os tempos limite de conexão e comando ao definir conexões. Um tempo limite muito curto pode resultar em falhas prematuras de conexões quando o banco de dados estiver ocupado. Um tempo limite muito longo pode impedir que a lógica de repetição funcione corretamente, aguardando tempo demais para detectar uma conexão com falha. O valor do tempo limite é um componente da latência de ponta a ponta; ele é efetivamente adicionado ao intervalo de repetição especificado na política de repetição para cada tentativa de repetição.
* Feche a conexão após um determinado número de repetições, mesmo ao usar uma lógica de repetição de retirada exponencial, e repita a operação em uma nova conexão. Repetir a mesma operação várias vezes na mesma conexão pode ser um fator que contribui para problemas de conexão. Para obter um exemplo dessa técnica, consulte [Camada de acesso a dados dos conceitos básicos do serviço de nuvem — tratamento de falhas transitórias](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).
* Quando o pool de conexões está em uso (o padrão), é provável que a mesma conexão seja escolhida no pool, mesmo depois de fechar e reabrir uma conexão. Se for esse o caso, uma técnica para resolver isso é chamar o método **ClearPool** da classe **SqlConnection** para marcar a conexão como não reutilizável. No entanto, isso deve ser feito somente depois que várias tentativas de conexão tiverem falhado, e somente ao encontrar a classe específica de falhas transitórias, como tempos limite de SQL (código de erro -2), relacionada às conexões com falha.
* Se o código de acesso a dados usar transações iniciadas como instâncias **TransactionScope** , a lógica de repetição deverá reabrir a conexão e iniciar um novo escopo de transação. Por esse motivo, o bloco de código com nova tentativa deve englobar o escopo inteiro da transação.

Considere começar com as seguintes configurações para operações de repetição. Essas são configurações de uso geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.

| **Contexto** | **Exemplo de latênciaE2E<br />máxima de destino** | **Estratégia de repetição** | **Configurações** | **Valores** | **Como funciona** |
| --- | --- | --- | --- | --- | --- |
| Interativo, interface de usuário<br />ou primeiro plano |2 s |FixedInterval |Contagem de repetição<br />Intervalo de repetição<br />Primeira repetição rápida |3<br />500 ms<br />verdadeiro |1ª tentativa — intervalo de 0 s<br />2ª tentativa — intervalo de 500 ms<br />3ª tentativa — intervalo de 500 ms |
| Segundo plano<br />ou lote |30 s |ExponentialBackoff |Contagem de repetição<br />Retirada mín.<br />Retirada máx.<br />Retirada delta<br />Primeira repetição rápida |5<br />0 s<br />60 s<br />2 s<br />falso |1ª tentativa — intervalo de 0 s<br />2ª tentativa — intervalo de ~2 s<br />3ª tentativa — intervalo de ~6 s<br />4ª tentativa — intervalo de ~14 s<br />5ª tentativa — intervalo de ~30 s |

> [!NOTE]
> A latência de ponta a ponta visa o tempo limite padrão de conexões com o serviço. Se você especificar tempos limite de conexão mais longos, a latência de ponta a ponta será estendida por esse tempo adicional para cada tentativa de repetição.
>
>

### <a name="examples"></a>Exemplos
Esta seção mostra como você pode usar a Polly para acessar o banco de dados SQL do Azure usando um conjunto de políticas de repetição configurado na classe `Policy`.

O código a seguir mostra um método de extensão na classe `SqlCommand` que chama `ExecuteAsync` com retirada exponencial.

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

Esse método de extensão assíncrono pode ser usado da seguinte maneira.

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a>Mais informações
* [Camada de acesso a dados dos conceitos básicos do serviço de nuvem — tratamento de falhas transitórias](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

Para orientação geral sobre como obter o máximo do banco de dados SQL, confira [Desempenho de banco de dados SQL do Azure e guia de elasticidade](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).

## <a name="sql-database-using-entity-framework-6"></a>Banco de Dados SQL usando o Entity Framework 6
O Banco de Dados SQL é um banco de dados SQL disponível em vários tamanhos e como um serviço padrão (compartilhado) e premium (não compartilhado). O Entity Framework é um mapeador relacional de objeto que permite aos desenvolvedores do .NET trabalhar com dados relacionais usando objetos específicos de domínio. Com ele, não há a necessidade da maioria dos códigos de acesso a dados que os desenvolvedores geralmente precisam para escrever.

### <a name="retry-mechanism"></a>Mecanismo de repetição
O suporte à repetição é fornecido durante o acesso ao Banco de Dados SQL usando o Entity Framework 6.0 e versões posteriores por meio de um mecanismo chamado [Lógica de Repetição/Resiliência de Conexão](http://msdn.microsoft.com/data/dn456835.aspx). Os principais recursos do mecanismo de repetição são:

* A abstração primária é a interface **IDbExecutionStrategy** . Essa interface:
  * Define métodos **Execute*** síncronos e assíncronos.
  * Define classes que podem ser usadas diretamente ou podem ser configuradas em um contexto de banco de dados como uma estratégia padrão, mapeadas para nome do provedor ou para um nome de provedor e nome de servidor. Quando configuradas em um contexto, as repetições ocorrem no nível de operações de banco de dados individuais, das quais pode haver várias de uma determinada operação de contexto.
  * Define quando repetir uma conexão com falha, e como.
* Incluem várias implementações internas da interface **IDbExecutionStrategy** :
  * Padrão: sem repetição.
  * Padrão para Banco de Dados SQL(automático): sem repetição, mas inspeciona exceções e as encapsula com sugestão para usar a estratégia Banco de Dados SQL.
  * Padrão para Banco de Dados SQL: exponencial (herdado da classe base) mais lógica de detecção do Banco de Dados SQL.
* Implementam uma estratégia de retirada exponencial que inclui aleatoriedade.
* As características das classes de repetição internas são: com estado e sem thread-safe. No entanto, elas podem ser reutilizadas depois que a operação atual for concluída.
* Se a contagem de repetição especificada for excedida, os resultados serão encapsulados em uma nova exceção. A exceção atual não é exibida.

### <a name="policy-configuration"></a>Configuração de política
O suporte à repetição é fornecido durante o acesso ao Banco de Dados SQL usando o Entity Framework 6.0 e versões superiores. As políticas de repetição são configuradas de modo programático. A configuração não pode ser alterada por operação.

Ao configurar uma estratégia no contexto como o padrão, você especifica uma função que cria uma nova estratégia sob demanda. O código a seguir mostra como você pode criar uma classe de configuração de repetição que estende a classe base **DbConfiguration** .

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

Depois isso pode ser especificado como a estratégia de repetição padrão para todas as operações usando o método **SetConfiguration** da instância **DbConfiguration** quando o aplicativo é iniciado. Por padrão, o EF vai descobrir e usar automaticamente a classe de configuração.

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

Você pode especificar a classe de configuração de repetição para um contexto anotando a classe de contexto com um atributo **DbConfigurationType** . No entanto, se você tiver apenas uma classe de configuração, o EF a usará sem a necessidade de anotar o contexto.

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

Se precisar usar estratégias de repetição diferentes para operações específicas ou desabilitar repetições para operações específicas, você poderá criar uma classe de configuração que permita suspender ou trocar estratégias, definindo um sinalizador no **CallContext**. A classe de configuração pode usar esse sinalizador para alternar estratégias ou desabilitar a estratégia fornecida por você e usar uma estratégia padrão. Para saber mais, consulte [Suspender estratégia de execução](http://msdn.microsoft.com/dn307226#transactions_workarounds) na página Limitações com estratégias de execução de repetição (EF6 em diante).

Outra técnica para usar estratégias de repetição específicas para operações individuais é criar uma instância da classe de estratégia necessária e fornecer as configurações desejadas por meio de parâmetros. Você então chama o método **ExecuteAsync** .

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

A maneira mais simples de usar uma classe **DbConfiguration** é colocá-la no mesmo assembly da classe **DbContext**. No entanto, isso não é apropriado quando o mesmo contexto é necessário em cenários diferentes, como diferentes estratégias de repetição interativas e em segundo plano. Se os diferentes contextos forem executados em AppDomains separados, você poderá usar o suporte interno para especificar classes de configuração no arquivo de configuração ou defini-las explicitamente usando código. Se os diferentes contextos devem ser executados no mesmo AppDomain, será necessária uma solução personalizada.

Para saber mais, consulte [Configuração baseada em código (EF6 em diante)](http://msdn.microsoft.com/data/jj680699.aspx).

A tabela a seguir mostra as configurações padrão para a política de repetição interna ao usar o EF6.

| Configuração | Valor padrão | Significado |
|---------|---------------|---------|
| Política | Exponencial | Retirada exponencial. |
| MaxRetryCount | 5 | O número máximo de repetições. |
| MaxDelay | 30 segundos | O atraso máximo entre as repetições. Esse valor não afeta como a série de atrasos é computada. Ele apenas define um limite superior. |
| DefaultCoefficient | 1 segundo | O coeficiente para o cálculo de retirada exponencial. Esse valor não pode ser alterado. |
| DefaultRandomFactor | 1,1 | O multiplicador usado para adicionar um atraso aleatório a cada entrada. Esse valor não pode ser alterado. |
| DefaultExponentialBase | 2 | O multiplicador usado para calcular o próximo atraso. Esse valor não pode ser alterado. |

### <a name="retry-usage-guidance"></a>Diretriz de uso de repetição
Considere as seguintes diretrizes ao acessar o Banco de Dados SQL usando o EF6:

* Escolha a opção de serviço apropriada (compartilhada ou premium). Uma instância compartilhada pode sofrer atrasos e limitação de conexão mais longos que o normal devido ao uso por outros locatários do servidor compartilhado. Se forem necessárias operações confiáveis de baixa latência e desempenho previsível, considere a escolha da opção premium.
* Uma estratégia de intervalo fixo não é recomendada para ser usada com o Banco de Dados SQL do Azure. Em vez disso, use uma estratégia de retirada exponencial, pois o serviço pode estar sobrecarregado e intervalos mais longos permitem mais tempo para a recuperação.
* Escolha um valor adequado para os tempos limite de conexão e comando ao definir conexões. Baseie o tempo limite no seu design de lógica de negócios e por meio de teste. Talvez seja necessário modificar esse valor ao longo do tempo, uma vez que os volumes de dados ou os processos de negócios mudam. Um tempo limite muito curto pode resultar em falhas prematuras de conexões quando o banco de dados estiver ocupado. Um tempo limite muito longo pode impedir que a lógica de repetição funcione corretamente, aguardando tempo demais para detectar uma conexão com falha. O valor do tempo limite é um componente da latência de ponta a ponta, embora você não possa determinar facilmente quantos comandos serão executados ao salvar o contexto. Você pode alterar o tempo limite padrão definindo a propriedade **CommandTimeout** da instância **DbContext**.
* O Entity Framework oferece suporte às configurações de repetição definidas em arquivos de configuração. No entanto, para máxima flexibilidade no Azure, você deve considerar a criação da configuração de modo programático dentro do aplicativo. Os parâmetros específicos das políticas de repetição, como o número de repetições e os intervalos da repetição, podem ser armazenados no arquivo de configuração de serviço e usados no tempo de execução para criar as políticas apropriadas. Isso permite que as configurações sejam alteradas sem precisar reiniciar o aplicativo.

Considere começar com as seguintes configurações para operações de repetição. Você não pode especificar o intervalo entre tentativas de repetição (ele é fixo e gerado como uma sequência exponencial). Você pode especificar apenas os valores máximo, conforme mostrado aqui, a menos que você crie uma estratégia de repetição personalizada. Essas são configurações de finalidade geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.

| **Contexto** | **Exemplo de latênciaE2E<br />máxima de destino** | **Política de repetição** | **Configurações** | **Valores** | **Como funciona** |
| --- | --- | --- | --- | --- | --- |
| Interativo, interface de usuário<br />ou primeiro plano |2 segundos |Exponencial |MaxRetryCount<br />MaxDelay |3<br />750 ms |1ª tentativa — intervalo de 0 s<br />2ª tentativa — intervalo de 750 ms<br />3ª tentativa — intervalo de 750 ms |
| Segundo plano<br /> ou lote |30 segundos |Exponencial |MaxRetryCount<br />MaxDelay |5<br />12 segundos |1ª tentativa — intervalo de 0 s<br />2ª tentativa — intervalo de ~1 s<br />3ª tentativa — intervalo de ~3 s<br />4ª tentativa — intervalo de ~7 s<br />5ª tentativa — intervalo de 12 s |

> [!NOTE]
> A latência de ponta a ponta visa o tempo limite padrão de conexões com o serviço. Se você especificar tempos limite de conexão mais longos, a latência de ponta a ponta será estendida por esse tempo adicional para cada tentativa de repetição.
>
>

### <a name="examples"></a>Exemplos
O exemplo de código a seguir define uma solução de acesso a dados simples que usa o Entity Framework. Ele estabelece uma estratégia de repetição específica definindo uma instância de uma classe chamada **BlogConfiguration** que estende **DbConfiguration**.

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

Mais exemplos de como usar o mecanismo de repetição do Entity Framework podem ser encontrados em [Lógica de Repetição/Resiliência de Conexão](http://msdn.microsoft.com/data/dn456835.aspx).

### <a name="more-information"></a>Mais informações
* [Guia de desempenho e elasticidade do Banco de Dados SQL do Azure](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a>Banco de Dados SQL usando o Entity Framework Core
[O Entity Framework Core](/ef/core/) é um mapeador relacional de objeto que permite aos desenvolvedores do .NET Core trabalhar com dados usando objetos específicos de domínio. Com ele, não há a necessidade da maioria dos códigos de acesso a dados que os desenvolvedores geralmente precisam para escrever. Esta versão do Entity Framework foi criada a partir do zero e não herda todos os recursos do EF6.x automaticamente.

### <a name="retry-mechanism"></a>Mecanismo de repetição
O suporte à repetição é fornecido durante o acesso ao Banco de Dados SQL usando o Entity Framework Core por meio de um mecanismo chamado [Resiliência de Conexão](/ef/core/miscellaneous/connection-resiliency). A resiliência de conexão foi introduzida no EF Core 1.1.0.

A abstração primária é a interface `IExecutionStrategy`. A estratégia de execução para o SQL Server, incluindo o SQL Azure, está ciente dos tipos de exceção que podem ser repetidos e tem padrões específicos para o número máximo de tentativas, atrasos entre as repetições e assim por diante.

### <a name="examples"></a>Exemplos

O código a seguir permite novas tentativas automáticas ao configurar o objeto DbContext, que representa uma sessão com o banco de dados. 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

O código a seguir mostra como executar uma transação com novas tentativas automáticas, usando uma estratégia de execução. A transação é definida em um representante. Se ocorrer uma falha transitória, a estratégia de execução invocará o representante novamente.

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

## <a name="azure-storage"></a>Armazenamento do Azure
Os serviços de armazenamento do Azure incluem armazenamento de tabela e blob, arquivos e filas de armazenamento.

### <a name="retry-mechanism"></a>Mecanismo de repetição
As repetições ocorrem no nível de operação REST individuais e são uma parte integrante da implementação da API do cliente. O SDK do armazenamento do cliente usa classes que implementam a [Interface IExtendedRetryPolicy](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).

Há diferentes implementações da interface. Os clientes de armazenamento podem escolher dentre políticas especificamente desenvolvidas para acesso a tabelas, blobs e filas. Cada implementação usa uma estratégia de repetição diferente que, basicamente, define o intervalo da repetição e outros detalhes.

As classes internas fornecem suporte a intervalos lineares (constantes) e exponenciais com intervalos de repetição de aleatoriedade. Também não há uma política de repetição a ser usada quando outro processo está tratando repetições em um nível mais alto. No entanto, você pode implementar suas próprias classes de repetição caso haja requisitos não fornecidos pelas classes internas.

As repetições alternativas serão alternadas entre o local do serviço de armazenamento primário e secundário se você estiver usando o RA-GRS (armazenamento com redundância geográfica com acesso de leitura) e o resultado da solicitação for um erro que pode ser repetido. Consulte [Opções de redundância de armazenamento do Azure](http://msdn.microsoft.com/library/azure/dn727290.aspx) para saber mais.

### <a name="policy-configuration"></a>Configuração de política
As políticas de repetição são configuradas de modo programático. Um procedimento comum é criar e popular uma instância **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** ou **QueueRequestOptions**.

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

A instância das opções de solicitação pode ser definida no cliente e todas as operações com o cliente usarão as opções de solicitação especificadas.

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

É possível substituir as opções de solicitação de cliente passando uma instância populada da classe de opções de solicitação como um parâmetro para os métodos de operação.

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

Você usa uma instância **OperationContext** para especificar o código a ser executado quando uma repetição ocorrer e quando uma operação tiver sido concluída. Esse código pode coletar informações sobre a operação para uso em logs e telemetria.

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

Além de indicar se uma falha é adequada para repetição, as políticas estendidas de repetição retornam um objeto **RetryContext** que indica o número de repetições, os resultados da última solicitação e se a próxima repetição acontecerá no local primário ou secundário (consulte a tabela abaixo para obter detalhes). As propriedades do objeto **RetryContext** podem ser usadas para decidir se e quando tentar uma repetição. Para obter mais detalhes, consulte [Método IExtendedRetryPolicy.Evaluate](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).

As tabelas a seguir mostram as configurações padrão para as políticas de repetição internas.

**Opções de solicitação**

| **Configuração** | **Valor padrão** | **Significado** |
| --- | --- | --- |
| MaximumExecutionTime | Nenhum | Tempo máximo de execução para a solicitação, incluindo todas as tentativas de repetição potenciais. Se não for especificada, a quantidade de tempo que uma solicitação pode levar é ilimitada. Em outras palavras, a solicitação pode travar. |
| ServerTimeout | Nenhum | O intervalo do tempo limite do servidor para a solicitação (o valor é arredondado para segundos). Se não for especificado, será usado o valor padrão para todas as solicitações ao servidor. Normalmente, a melhor opção é omitir essa configuração para que o padrão de servidor seja usado. | 
| LocationMode | Nenhum | Se a conta de armazenamento for criada com a opção de replicação RA-GRS (armazenamento com redundância geográfica com acesso de leitura), você poderá usar o modo de local para indicar qual local deve receber a solicitação. Por exemplo, se **PrimaryThenSecondary** for especificado, as solicitações sempre serão enviadas para o local primário primeiro. Se uma solicitação falhar, ela será enviada para o local secundário. |
| RetryPolicy | ExponentialPolicy | Veja abaixo os detalhes de cada opção. |

**Política exponencial** 

| **Configuração** | **Valor padrão** | **Significado** |
| --- | --- | --- |
| maxAttempt | 3 | Número de tentativas de repetição. |
| deltaBackoff | 4 segundos | Intervalo de retirada entre repetições. Múltiplos desse período de tempo, incluindo um elemento aleatório, serão usados para tentativas de repetição subsequentes. |
| MinBackoff | 3 Segundos | Adicionado a todos os intervalos de repetição calculados de deltaBackoff. Esse valor não pode ser alterado.
| MaxBackoff | 120 segundos | MaxBackoff será usado se o intervalo de repetição calculado for maior que MaxBackoff. Esse valor não pode ser alterado. |

**Política linear**

| **Configuração** | **Valor padrão** | **Significado** |
| --- | --- | --- |
| maxAttempt | 3 | Número de tentativas de repetição. |
| deltaBackoff | 30 segundos | Intervalo de retirada entre repetições. |

### <a name="retry-usage-guidance"></a>Diretriz de uso de repetição
Considere as seguintes diretrizes ao acessar serviços de armazenamento do Azure usando a API do cliente de armazenamento:

* Use as políticas de repetição internas do namespace Microsoft.WindowsAzure.Storage.RetryPolicies onde elas forem apropriadas para suas necessidades. Na maioria dos casos, essas políticas serão suficientes.
* Use a política **ExponentialRetry** em operações em lote, tarefas em segundo plano ou cenários não interativos. Nesses cenários, geralmente você pode permitir mais tempo para que o serviço se recupere — com uma chance consequentemente aumentada da operação ser bem-sucedida.
* Considere especificar a propriedade **MaximumExecutionTime** do parâmetro **RequestOptions** para limitar o tempo de execução total, mas leve em consideração o tipo e o tamanho da operação ao escolher um valor de tempo limite.
* Se você precisar implementar uma repetição personalizada, evite criar wrappers em torno das classes do cliente de armazenamento. Em vez disso, use os recursos para estender as políticas existentes por meio da interface **IExtendedRetryPolicy** .
* Se você estiver usando RA-GRS (armazenamento com redundância geográfica com acesso de leitura), será possível usar o **LocationMode** para especificar que as tentativas de repetição acessarão a cópia secundária somente leitura do repositório caso o acesso primário falhe. No entanto, ao usar essa opção, você deve garantir que seu aplicativo possa trabalhar perfeitamente com dados que talvez estejam obsoletos, se a replicação do repositório primário ainda não tiver sido concluída.

Considere começar com as seguintes configurações para operações de repetição. Essas são configurações de uso geral, por isso, você deve monitorar as operações e ajustar os valores para que atendam ao seu cenário.  

| **Contexto** | **Exemplo de latênciaE2E<br />máxima de destino** | **Política de repetição** | **Configurações** | **Valores** | **Como funciona** |
| --- | --- | --- | --- | --- | --- |
| Interativo, interface de usuário<br />ou primeiro plano |2 segundos |Linear |maxAttempt<br />deltaBackoff |3<br />500 ms |1ª tentativa — intervalo de 500 ms<br />2ª tentativa — intervalo de 500 ms<br />3ª tentativa — intervalo de 500 ms |
| Segundo plano<br />ou lote |30 segundos |Exponencial |maxAttempt<br />deltaBackoff |5<br />4 segundos |1ª tentativa — intervalo de ~3 s<br />2ª tentativa — intervalo de ~7 s<br />3ª tentativa — intervalo de ~15 s |

### <a name="telemetry"></a>Telemetria
As tentativas de repetição são registradas em um **TraceSource**. Você deve configurar um **TraceListener** para capturar os eventos e gravá-los em um log de destino apropriado. É possível usar o **TextWriterTraceListener** ou **XmlWriterTraceListener** para gravar os dados em um arquivo de log, o **EventLogTraceListener** para gravar no Log de Eventos do Windows ou o **EventProviderTraceListener** para gravar dados de rastreamento no subsistema ETW. Você também pode configurar a liberação automática do buffer e o detalhamento dos eventos que serão registrados (por exemplo, Erro, Aviso, Informativo e Detalhado). Para saber mais, consulte [Registro em log no lado do cliente com a biblioteca do cliente de armazenamento para .NET](http://msdn.microsoft.com/library/azure/dn782839.aspx).

As operações podem receber uma instância **OperationContext**, que expõe um evento **Retrying** que pode ser usado para anexar a lógica personalizada de telemetria. Para saber mais, consulte [Evento OperationContext.Retrying](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).

### <a name="examples"></a>Exemplos
O exemplo de código a seguir mostra como criar duas instâncias **TableRequestOptions** com diferentes configurações de repetição; uma para solicitações interativas e outra para solicitações em segundo plano. O exemplo define essas duas políticas de repetição no cliente para que elas sejam aplicadas a todas as solicitações, assim como define a estratégia interativa em uma solicitação específica de modo que ela substitua as configurações padrão aplicadas ao cliente.

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

### <a name="more-information"></a>Mais informações
* [Recomendações de política de repetição da biblioteca do cliente de armazenamento do Azure](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [Biblioteca do cliente de armazenamento 2.0 – implementando políticas de repetição](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a>Diretrizes gerais de repetição e REST
Considere o seguinte ao acessar os serviços do Azure ou de terceiros:

* Use uma abordagem sistemática para gerenciar repetições, talvez como código reutilizável, para que você possa aplicar uma metodologia consistente em todos os clientes e soluções.
* Considere usar uma estrutura de repetição, como o Bloco de Aplicativos para Tratamento de Falhas Transitórias, para gerenciar repetições se o serviço ou cliente de destino não tiver algum mecanismo de repetição interno. Isso ajudará você a implementar um comportamento de repetição consistente, bem como pode fornecer uma estratégia de repetição padrão adequada para o serviço de destino. No entanto, talvez seja necessário criar código de repetição personalizado para serviços que tenham comportamento não padrão, que não dependem de exceções para indicar falhas transitórias, ou se desejar, use uma resposta **Retry-Response** para gerenciar o comportamento de repetição.
* A lógica de detecção transitória dependerá da API de cliente real que você usa para invocar as chamadas REST. Alguns clientes, como a classe mais recente **HttpClient** , não lançam exceções para solicitações concluídas com um código de status HTTP sem sucesso. Isso melhora o desempenho, mas impede o uso do Bloco de Aplicativos para Tratamento de Falhas Transitórias. Nesse caso, você pode encapsular a chamada à API REST com o código que gera exceções para códigos de status HTTP sem sucesso, que pode então ser processada pelo bloco. Como alternativa, é possível usar um mecanismo diferente para orientar as repetições.
* O código de status HTTP retornado do serviço pode ajudar a indicar se a falha é transitória. Talvez seja necessário examinar as exceções geradas por um cliente ou pela estrutura de repetição para acessar o código de status ou determinar o tipo de exceção equivalente. Os seguintes códigos HTTP geralmente indicam que uma repetição é apropriada:
  * 408 Tempo Limite da Solicitação
  * 429 Número excessivo de solicitações
  * 500 Erro Interno do Servidor
  * 502 Gateway Incorreto
  * 503 Serviço Indisponível
  * 504 Tempo Limite do Gateway
* Se você basear suas lógica de repetição em exceções, geralmente o que se segue indica uma falha transitória onde nenhuma conexão pode ser estabelecida:
  * WebExceptionStatus.ConnectionClosed
  * WebExceptionStatus.ConnectFailure
  * WebExceptionStatus.Timeout
  * WebExceptionStatus.RequestCanceled
* No caso de um status de serviço indisponível, o serviço pode indicar o atraso apropriado antes de tentar a repetição no cabeçalho da resposta **Retry-After** ou em um cabeçalho personalizado diferente. Os serviços também podem enviar informações adicionais como cabeçalhos personalizados ou inseridos no conteúdo da resposta. O Bloco de Aplicativos para Tratamento de Falhas Transitórias não pode usar os cabeçalhos “retry-after” padrão ou personalizados.
* Não tente a repetição para códigos de status que representam erros de cliente (erros no intervalo 4xx), exceto para um 408 Tempo Limite de Solicitação.
* Teste minuciosamente seus mecanismos e estratégias de repetição sob diversas condições, como estado diferente de rede e cargas variáveis de sistema.

### <a name="retry-strategies"></a>Estratégias de repetição
Veja a seguir os tipos comuns de intervalo de estratégias de repetição:

* **Exponencial**. Uma política de repetição que executa um determinado número de repetições usando uma abordagem de retirada exponencial aleatória para determinar o intervalo entre as repetições. Por exemplo: 

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

* **Incremental**. Uma estratégia de repetição com um número especificado de tentativas de repetição e um intervalo de tempo incremental entre entradas. Por exemplo: 

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

* **LinearRetry**. Uma política de repetição que executa um número especificado de repetições usando um intervalo de tempo fixo especificado entre as repetições. Por exemplo: 

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a>Falha transitória manipulada com a Polly
[Polly] [ polly] é uma biblioteca para manipular programaticamente novas tentativas e estratégias de [disjuntor] [ circuit-breaker]. O projeto Polly é um membro da [Fundação .NET][dotnet-foundation]. Para serviços em que o cliente não oferece suporte a novas tentativas, a Polly é uma alternativa válida e evita a necessidade de escrever código de repetição personalizado, que pode ser difícil de implementar corretamente. A Polly também fornece uma maneira de rastrear erros, para que você possa fazer novas tentativas.


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


### <a name="more-information"></a>Mais informações
* [Resiliência de conexão](/ef/core/miscellaneous/connection-resiliency)
* [Pontos de dados - Core EF 1.1](https://msdn.microsoft.com/magazine/mt745093.aspx)


