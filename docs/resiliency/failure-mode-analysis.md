---
title: Análise do modo de falha
description: Diretrizes para executar a análise do modo de falha para soluções de nuvem baseadas no Azure.
author: MikeWasson
ms.date: 03/24/2017
ms.custom: resiliency
pnp.series.title: Design for Resiliency
ms.openlocfilehash: 95068bf8b1f5b559255e27819aaddb454d3427bc
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/07/2018
---
# <a name="failure-mode-analysis"></a>Análise do modo de falha
[!INCLUDE [header](../_includes/header.md)]

A FMA (análise do modo de falha) é um processo de criação de resiliência em um sistema por meio da identificação de possíveis pontos de falha no sistema. A FMA deve ser parte das fases de arquitetura e de design para que você possa inserir a recuperação de falha no sistema desde o início.

Veja abaixo o processo geral para realizar uma FMA:

1. Identifique todos os componentes no sistema. Inclua dependências externas, por exemplo, provedores de identidade, serviços de terceiros e assim por diante.   
2. Para cada componente, identifique possíveis falhas que possam ocorrer. Um único componente pode ter mais de um modo de falha. Por exemplo, considere a possibilidade de falhas de leitura e de gravação separadamente, pois o impacto e as possíveis mitigações serão diferentes.
3. Classifique cada modo de falha de acordo com seu risco geral. Considere estes fatores:  

   * Qual é a probabilidade da falha. Ela é relativamente comum? Extremamente rara? Não são necessários números exatos; o objetivo é ajudar a classificar a prioridade.
   * Qual é o impacto no aplicativo em termos de disponibilidade, de perda de dados, do custo monetário e da interrupção dos negócios?
4. Para cada modo de falha, determinam como o aplicativo responderá e se recuperará. Considere as compensações em relação aos custos e à complexidade do aplicativo.   

Como ponto de partida para o processo de FMA, este artigo contém um catálogo de possíveis modos de falha e suas mitigações. O catálogo é organizado por tecnologia ou serviço do Azure, além de uma categoria geral para o design em nível do aplicativo. O catálogo não é completo, mas abrange muitos dos principais serviços do Azure.

## <a name="app-service"></a>Serviço de Aplicativo
### <a name="app-service-app-shuts-down"></a>Aplicativo do Serviço de Aplicativo desliga.
**Detecção**. Possíveis causas:

* Desligamento esperado

  * Um operador desliga o aplicativo, por exemplo, usando o Portal do Azure.
  * O aplicativo foi descarregado porque estava ocioso. (Somente se a configuração `Always On` estiver desabilitada.)
* Desligamento inesperado

  * Falha no aplicativo.
  * Uma instância VM do Serviço de Aplicativo torna-se não disponível.

O log Application_End detectará o desligamento do domínio do aplicativo (falha simples de processo) e será a única maneira de detectar os desligamentos do domínio do aplicativo.

**Recuperação**

* Se o desligamento for esperado, use o evento de desligamento do aplicativo para desligá-lo normalmente. Por exemplo, no ASP.NET, use o método `Application_End`.
* Se o aplicativo for descarregado quando ocioso, ele será reiniciado automaticamente na próxima solicitação. No entanto, ocorrerá a "inicialização a frio".
* Para impedir que o aplicativo seja descarregado quando ocioso, habilite a configuração `Always On` no aplicativo Web. Veja [Configurar aplicativos Web no Serviço de Aplicativo do Azure][app-service-configure].
* Para impedir que um operador desligue o aplicativo, defina um bloqueio de recurso com o nível `ReadOnly`. Veja [Bloquear recursos com o Azure Resource Manager][rm-locks].
* Se o aplicativo falhar ou se uma VM do Serviço de Aplicativo ficar não disponível, o Serviço de Aplicativo reiniciará automaticamente o aplicativo.

**Diagnóstico**. Logs do aplicativo e logs do servidor Web. Veja [Habilitar o registro em log de diagnóstico para aplicativos Web no Serviço de Aplicativo do Azure][app-service-logging].

### <a name="a-particular-user-repeatedly-makes-bad-requests-or-overloads-the-system"></a>Um usuário específico repetidamente faz solicitações inválidas ou sobrecarrega o sistema.
**Detecção**. Autentique os usuários e inclua a ID de usuário nos logs do aplicativo.

**Recuperação**

* Use o [Gerenciamento de API do Azure][api-management] para limitar as solicitações do usuário. Veja [Limitação de solicitação avançada com o Gerenciamento de API do Azure][api-management-throttling]
* Bloqueie o usuário.

**Diagnóstico**. Registre em log todas as solicitações de autenticação.

### <a name="a-bad-update-was-deployed"></a>Uma atualização inválida foi implantada.
**Detecção**. Monitore a integridade do aplicativo por meio do Portal do Azure (veja [Monitorar o desempenho do aplicativo Web do Azure][app-insights-web-apps]) ou implemente o [padrão de monitoramento do ponto de extremidade de integridade][health-endpoint-monitoring-pattern].

**Recuperação**. Use vários [slots de implantação][app-service-slots] e reverta para a última implantação válida conhecida. Para obter mais informações, veja [Basic web application][ra-web-apps-basic] (Aplicativo Web básico).

## <a name="azure-active-directory"></a>Azure Active Directory
### <a name="openid-connect-oidc-authentication-fails"></a>Falha na autenticação OIDC (OpenID Connect).
**Detecção**. Os possíveis modos de falha incluem:

1. O Azure AD não está disponível ou não pode ser acessado devido a um problema de rede. Falha no redirecionamento para o ponto de extremidade de autenticação e o middleware OIDC gera uma exceção.
2. O locatário do Azure AD não existe. O redirecionamento para o ponto de extremidade de autenticação retorna um código de erro HTTP e o middleware OIDC gera uma exceção.
3. O usuário não pode se conectar. Nenhuma estratégia de detecção é necessária. O Azure AD resolve as falhas de logon.

**Recuperação**

1. Detecte exceções sem tratamento do middleware.
2. Trate os eventos `AuthenticationFailed`.
3. Redirecione o usuário para uma página de erro.
4. O usuário tenta novamente.

## <a name="azure-search"></a>Azure Search
### <a name="writing-data-to-azure-search-fails"></a>Falha ao gravar dados do Azure Search.
**Detecção**. Detecte erros `Microsoft.Rest.Azure.CloudException`.

**Recuperação**

O [SDK .NET do Search][search-sdk] tenta novamente de maneira automática após falhas transitórias. As exceções geradas pelo SDK cliente devem ser tratadas como erros não transitórios.

A política de novas tentativas padrão usa retirada exponencial. Para usar uma política de novas tentativas diferente, chame `SetRetryPolicy` na classe `SearchIndexClient` ou `SearchServiceClient`. Para obter mais informações, veja [Tentativas automáticas][auto-rest-client-retry].

**Diagnóstico**. Use a [Análise de Tráfego de Pesquisa][search-analytics].

### <a name="reading-data-from-azure-search-fails"></a>Falha ao ler dados do Azure Search.
**Detecção**. Detecte erros `Microsoft.Rest.Azure.CloudException`.

**Recuperação**

O [SDK .NET do Search][search-sdk] tenta novamente de maneira automática após falhas transitórias. As exceções geradas pelo SDK cliente devem ser tratadas como erros não transitórios.

A política de novas tentativas padrão usa retirada exponencial. Para usar uma política de novas tentativas diferente, chame `SetRetryPolicy` na classe `SearchIndexClient` ou `SearchServiceClient`. Para obter mais informações, veja [Tentativas automáticas][auto-rest-client-retry].

**Diagnóstico**. Use a [Análise de Tráfego de Pesquisa][search-analytics].

## <a name="cassandra"></a>Cassandra
### <a name="reading-or-writing-to-a-node-fails"></a>Falha de leitura ou de gravação em um nó.
**Detecção**. Detecte a exceção. Para clientes .NET, normalmente será `System.Web.HttpException`. Outro cliente pode ter outros tipos de exceção.  Para obter mais informações, veja [Cassandra error handling done right](http://www.datastax.com/dev/blog/cassandra-error-handling-done-right) (Tratamento de erro do Cassandra feito da maneira correta).

**Recuperação**

* Cada [cliente Cassandra](https://wiki.apache.org/cassandra/ClientOptions) tem suas próprias políticas de novas tentativas e funcionalidades. Para obter mais informações, veja [Cassandra error handling done right][cassandra-error-handling] (Tratamento de erro do Cassandra feito da maneira correta).
* Use uma implantação com reconhecimento de rack, com nós de dados distribuídos entre os domínios de falha.
* Implante em várias regiões com consistência de quorum local. Se ocorrer uma falha não transitória, faça failover para outra região.

**Diagnóstico**. Logs de aplicativo

## <a name="cloud-service"></a>Serviço de Nuvem
### <a name="web-or-worker-roles-are-unexpectedlybeing-shut-down"></a>Funções de trabalho ou Web estão sendo desligadas inesperadamente.
**Detecção**. O evento [RoleEnvironment.Stopping][RoleEnvironment.Stopping] é disparado.

<strong>Recuperação</strong>. Substitua o método [RoleEntryPoint.OnStop][RoleEntryPoint.OnStop] para executar uma limpeza normal. Para obter mais informações, veja [The Right Way to Handle Azure OnStop Events][onstop-events] (A maneira correta de tratar eventos OnStop do Azure) (blog).

## <a name="cosmos-db"></a>Cosmos DB 
### <a name="reading-data-fails"></a>Falha na leitura de dados.
**Detecção**. Detecte `System.Net.Http.HttpRequestException` ou `Microsoft.Azure.Documents.DocumentClientException`.

**Recuperação**

* O SDK tenta novamente de maneira automática em caso de tentativas com falha. Para definir o número de novas tentativas e o tempo de espera máximo, configure `ConnectionPolicy.RetryOptions`. As exceções que o cliente gera excedem a política de repetição ou não são erros transitórios.
* Se o Cosmos DB limitar o cliente, ele retornará um erro HTTP 429. Confira o código de status no `DocumentClientException`. Se você estiver recebendo o erro 429 consistentemente, considere aumentar o valor da taxa de transferência da coleção.
    * Se você estiver usando a API do MongoDB, o serviço retornará o código de erro 16500 durante a limitação.
* Replique o banco de dados do Cosmos DB em duas ou mais regiões. Todas as réplicas são legíveis. Usando os SDKs cliente, especifique o parâmetro `PreferredLocations`. Trata-se de uma lista ordenada de regiões do Azure. Todas as leituras serão enviadas para a primeira região disponível na lista. Se a solicitação falhar, o cliente tentará outras regiões na lista, em ordem. Para obter mais informações, confira [Como configurar a distribuição global do Azure Cosmos DB usando a API do SQL][cosmosdb-multi-region].

**Diagnóstico**. Registre em log todos os erros no lado do cliente.

### <a name="writing-data-fails"></a>Falha na gravação de dados.
**Detecção**. Detecte `System.Net.Http.HttpRequestException` ou `Microsoft.Azure.Documents.DocumentClientException`.

**Recuperação**

* O SDK tenta novamente de maneira automática em caso de tentativas com falha. Para definir o número de novas tentativas e o tempo de espera máximo, configure `ConnectionPolicy.RetryOptions`. As exceções que o cliente gera excedem a política de repetição ou não são erros transitórios.
* Se o Cosmos DB limitar o cliente, ele retornará um erro HTTP 429. Confira o código de status no `DocumentClientException`. Se você estiver recebendo o erro 429 consistentemente, considere aumentar o valor da taxa de transferência da coleção.
* Replique o banco de dados do Cosmos DB em duas ou mais regiões. Se a região primária falhar, outra região será promovida para gravação. Você também pode disparar um failover manual. O SDK faz a descoberta e roteamento automáticos para que o código do aplicativo continue a funcionar após um failover. Durante o período do failover (normalmente alguns minutos), as operações de gravação terão latência mais alta, enquanto o SDK localizar a nova região de gravação.
  Para obter mais informações, confira [Como configurar a distribuição global do Azure Cosmos DB usando a API do SQL][cosmosdb-multi-region].
* Como fallback, mantenha o documento em uma fila de backup e processe a fila mais tarde.

**Diagnóstico**. Registre em log todos os erros no lado do cliente.

## <a name="elasticsearch"></a>Elasticsearch
### <a name="reading-data-from-elasticsearch-fails"></a>Falha na leitura de dados do Elasticsearch.
**Detecção**. Capture a exceção apropriada para o [cliente Elasticsearch][elasticsearch-client] específico sendo usado.

**Recuperação**

* Use um mecanismo de novas tentativas. Cada cliente tem suas próprias políticas de novas tentativas.
* Implante vários nós do Elasticsearch e use a replicação para obter alta disponibilidade.

Para obter mais informações, veja [Executar o Elasticsearch no Azure][elasticsearch-azure].

**Diagnóstico**. Você pode usar ferramentas de monitoramento para Elasticsearch ou registrar todos os erros no lado do cliente com a conteúdo. Consulte a seção "Monitoramento" em [Executar o Elasticsearch no Azure][elasticsearch-azure].

### <a name="writing-data-to-elasticsearch-fails"></a>Falha na gravação de dados no Elasticsearch.
**Detecção**. Capture a exceção apropriada para o [cliente Elasticsearch][elasticsearch-client] específico sendo usado.  

**Recuperação**

* Use um mecanismo de novas tentativas. Cada cliente tem suas próprias políticas de novas tentativas.
* Se o aplicativo puder tolerar um nível de consistência reduzido, considere gravar com a configuração `write_consistency` de `quorum`.

Para obter mais informações, veja [Executar o Elasticsearch no Azure][elasticsearch-azure].

**Diagnóstico**. Você pode usar ferramentas de monitoramento para Elasticsearch ou registrar todos os erros no lado do cliente com a conteúdo. Consulte a seção "Monitoramento" em [Executar o Elasticsearch no Azure][elasticsearch-azure].

## <a name="queue-storage"></a>Armazenamento de filas
### <a name="writing-a-message-to-azure-queue-storage-fails-consistently"></a>Falha consistente na gravação de uma mensagem do Armazenamento de Filas do Azure.
**Detecção**. Depois de *N* novas tentativas, ainda há falha na operação de gravação.

**Recuperação**

* Armazene os dados em um cache local e encaminhe as gravações para o armazenamento mais tarde, quando o serviço estiver disponível.
* Crie uma fila secundária e grave nessa fila se a fila principal não estiver disponível.

**Diagnóstico**. Use [métricas de armazenamento][storage-metrics].

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>O aplicativo não pode processar uma mensagem específica da fila.
**Detecção**. Específico ao aplicativo. Por exemplo, a mensagem contém dados inválidos ou há falha na lógica de negócios por algum motivo.

**Recuperação**

Mova a mensagem para uma fila separada. Execute um processo separado para examinar as mensagens na fila.

Considere usar filas de mensagens do Barramento de Serviço do Azure, que fornece uma funcionalidade de [fila de mensagens mortas][sb-dead-letter-queue] para essa finalidade.

> [!NOTE]
> Se você estiver usando filas de armazenamento com WebJobs, o SDK do WebJobs fornecerá tratamento interno de mensagens suspeitas. Veja [Como usar o Armazenamento de Filas do Azure com o SDK do WebJobs][sb-poison-message].

**Diagnóstico**. Use o registro em log do aplicativo.

## <a name="redis-cache"></a>Cache Redis
### <a name="reading-from-the-cache-fails"></a>Falha na leitura do cache.
**Detecção**. Detecte `StackExchange.Redis.RedisConnectionException`.

**Recuperação**

1. Tente novamente em caso de falhas transitórias. O Cache Redis do Azure é compatível com novas tentativas internas. Veja as [Diretrizes de para Cache Redis][redis-retry].
2. Trate as falhas não transitórias como uma perda no cache e faça fallback para a fonte de dados original.

**Diagnóstico**. Use o [Diagnóstico do Cache Redis][redis-monitor].

### <a name="writing-to-the-cache-fails"></a>Falha ao gravar no cache.
**Detecção**. Detecte `StackExchange.Redis.RedisConnectionException`.

**Recuperação**

1. Tente novamente em caso de falhas transitórias. O Cache Redis do Azure é compatível com novas tentativas internas. Veja as [Diretrizes de para Cache Redis][redis-retry].
2. Se o erro não for transitório, ignore-o e permita que outras transações gravem no cache mais tarde.

**Diagnóstico**. Use o [Diagnóstico do Cache Redis][redis-monitor].

## <a name="sql-database"></a>Banco de dados SQL
### <a name="cannot-connect-to-the-database-in-the-primary-region"></a>Não é possível conectar ao banco de dados na região primária.
**Detecção**. Falha na conexão.

**Recuperação**

Pré-requisito: o banco de dados deve estar configurado para replicação geográfica ativa. Veja [Replicação geográfica ativa para Banco de Dados SQL][sql-db-replication].

* Para as consultas, leia de uma réplica secundária.
* Para inserções e atualizações, faça failover manualmente para uma réplica secundária. Veja [Iniciar um failover planejado ou não planejado para o Banco de Dados SQL do Azure][sql-db-failover].

A réplica usa uma cadeia de conexão diferente, portanto será necessário atualizar a cadeia de conexão em seu aplicativo.

### <a name="client-runs-out-of-connections-in-the-connection-pool"></a>O cliente fica sem conexões no pool de conexões.
**Detecção**. Detecte erros `System.InvalidOperationException`.

**Recuperação**

* Repita a operação.
* Como plano de mitigação, isole os pools de conexões para cada caso de uso, para que um caso de uso não domine todas as conexões.
* Aumente os limites máximos dos pools de conexões.

**Diagnóstico**. Logs do aplicativo.

### <a name="database-connection-limit-is-reached"></a>O limite de conexão de banco de dados foi alcançado.
**Detecção**. O Banco de Dados SQL do Azure limita o número de trabalhos, logons e sessões simultâneos. Os limites dependem da camada de serviço. Para obter mais informações, veja [Limites de recursos do Banco de Dados SQL do Azure][sql-db-limits].

Para detectar esses erros, detecte `System.Data.SqlClient.SqlException` e verifique o valor de `SqlException.Number` para o código de erro SQL. Para obter um lista de códigos de erro relevantes, veja [Códigos de erro de SQL para aplicativos cliente do Banco de Dados SQL: erro de conexão de banco de dados e outros problemas][sql-db-errors].

**Recuperação**. Esses erros são considerados transitórios, portanto tentar novamente poderá resolver o problema. Se você encontrar consistentemente esses erros, considere dimensionar o banco de dados.

**Diagnóstico**. - A consulta [sys.event_log][sys.event_log] retorna conexões de banco de dados bem-sucedidas, falhas na conexão e deadlocks.

* Crie uma [regra de alerta][azure-alerts] para falhas em conexões.
* Habilite [Auditoria do Banco de Dados SQL][sql-db-audit] e verifique se há falha em logons.

## <a name="service-bus-messaging"></a>Mensagens do Barramento de Serviço
### <a name="reading-a-message-from-a-service-bus-queue-fails"></a>Falha na leitura de uma mensagem de uma fila do Barramento de Serviço.
**Detecção**. Detecte exceções no SDK do cliente. A classe base para exceções do Barramento de Serviço é [MessagingException][sb-messagingexception-class]. Se o erro for transitório, a propriedade `IsTransient` será true.

Para obter mais informações, veja [Exceções de mensagens do Barramento de Serviço][sb-messaging-exceptions].

**Recuperação**

1. Tente novamente em caso de falhas transitórias. Veja [Diretrizes de repetição para Barramento de Serviço][sb-retry].
2. As mensagens que não podem ser entregues a nenhum receptor são colocadas em um *fila de mensagens mortas*. Use essa fila para ver quais mensagens não puderam ser recebidas. Não há opções de limpeza automática da fila de mensagens mortas. As mensagens permanecem lá até você explicitamente recuperá-las. Veja [Visão geral das filas de mensagens mortas do Barramento de Serviço][sb-dead-letter-queue].

### <a name="writing-a-message-to-a-service-bus-queue-fails"></a>Falha na gravação de uma mensagem em uma fila do Barramento de Serviço.
**Detecção**. Detecte exceções no SDK do cliente. A classe base para exceções do Barramento de Serviço é [MessagingException][sb-messagingexception-class]. Se o erro for transitório, a propriedade `IsTransient` será true.

Para obter mais informações, veja [Exceções de mensagens do Barramento de Serviço][sb-messaging-exceptions].

**Recuperação**

1. O cliente do Barramento de Serviço tenta novamente de maneira automática após erros transitórios. Por padrão, ele usa retirada exponencial. Após a contagem máxima de novas tentativas ou do tempo limite máximo, o cliente gerará uma exceção. Para obter mais informações, veja [Diretrizes de repetição para Barramento de Serviço][sb-retry].
2. Se a cota da fila for excedida, o cliente gerará [QuotaExceededException][QuotaExceededException]. A mensagem de exceção fornece mais detalhes. Retire algumas mensagens da fila antes de tentar novamente e considere usar o padrão de Disjuntor para evitar novas tentativas contínuas enquanto a cota estiver excedida. Além disso, verifique se a propriedade [BrokeredMessage.TimeToLive] não está definida com um valor muito alto.
3. Em uma região, a resiliência pode ser melhorada usando [filas ou tópicos particionados][sb-partition]. Uma fila ou um tópico não particionado é atribuído a um repositório de mensagens. Se esse repositório de mensagens não estiver disponível, todas as operações na fila ou no tópico falharão. Uma fila ou tópico particionado é particionado em vários repositórios de mensagens.
4. Para obter resiliência adicional, crie dois namespaces do Barramento de Serviço em regiões diferentes e replique as mensagens. Você pode usar a replicação ativa ou passiva.

   * Replicação ativa: o cliente envia todas as mensagens para ambas as filas. O receptor escuta em ambas as filas. Marque mensagens com um identificador exclusivo para que o cliente possa descartar mensagens duplicadas.
   * Replicação passiva: o cliente envia a mensagem para uma fila. Se houver um erro, o cliente fará fallback para a outra fila. O receptor escuta em ambas as filas. Essa abordagem reduz o número de mensagens duplicadas enviadas. No entanto, o receptor ainda deve tratar as mensagens duplicadas.

     Para saber mais, veja [Exemplo de replicação geográfica][sb-georeplication-sample] e [Práticas recomendadas para isolar aplicativos contra interrupções e desastres do Barramento de Serviço](/azure/service-bus-messaging/service-bus-outages-disasters/).

### <a name="duplicate-message"></a>Duplique a mensagem.
**Detecção**. Examine as propriedades `MessageId` e `DeliveryCount` da mensagem.

**Recuperação**

* Se possível, crie suas próprias operações de processamento de mensagens para serem idempotentes. Caso contrário, armazene as IDs das mensagens já processadas e verifique a ID antes de processar uma mensagem.
* Habilite detecção de duplicidades criando a fila com `RequiresDuplicateDetection` definido como true. Com essa configuração, o Barramento de Serviço exclui automaticamente qualquer mensagem enviada com a mesma `MessageId` que alguma mensagem anterior.  Observe o seguinte:

  * Essa configuração impede que mensagens duplicadas sejam colocadas na fila. Ela não impede que um receptor de processar a mesma mensagem mais de uma vez.
  * A detecção de duplicidades tem uma janela de tempo. Se uma duplicata for enviada fora dessa janela, ela não será detectada.

**Diagnóstico**. Registre em log as mensagens duplicadas.

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>O aplicativo não pode processar uma mensagem específica da fila.
**Detecção**. Específico ao aplicativo. Por exemplo, a mensagem contém dados inválidos ou há falha na lógica de negócios por algum motivo.

**Recuperação**

Há dois modos de falha a serem considerados.

* O receptor detecta a falha. Nesse caso, mova a mensagem para a fila de mensagens mortas. Depois, execute um processo separado para examinar as mensagens na fila de mensagens mortas.
* O receptor encontra uma falha durante o processamento da mensagem &mdash;, por exemplo, devido a uma exceção sem tratamento. Para tratar esse caso, use o modo `PeekLock`. Nesse modo, se o bloqueio expirar, a mensagem ficará disponível para outros receptores. Se a mensagem exceder a contagem máxima de entregas ou o tempo de vida, ela será automaticamente movida para a fila de mensagens mortas.

Para obter mais informações, veja [Visão geral das filas de mensagens mortas do Barramento de Serviço][sb-dead-letter-queue].

**Diagnóstico**. Sempre que o aplicativo mover uma mensagem para a fila de mensagens mortas, grave um evento nos logs do aplicativo.

## <a name="service-fabric"></a>Service Fabric
### <a name="a-request-to-a-service-fails"></a>Falha em uma solicitação para um serviço.
**Detecção**. O serviço retorna um erro.

**Recuperação**

* Localize um proxy novamente (`ServiceProxy` ou `ActorProxy`) e chame o método do serviço/ator novamente.
* **Serviço com estado**. Encapsule operações em coleções confiáveis em uma transação. Se houver um erro, a transação será revertida. A solicitação, se extraída de uma fila, será processada novamente.
* **Serviço sem estado**. Se o serviço persistir dados em um repositório externo, todas as operações precisarão ser idempotentes.

**Diagnóstico**. Log do aplicativo

### <a name="service-fabric-node-is-shut-down"></a>O nó do Service Fabric é desligado.
**Detecção**. Um token de cancelamento é passado para o método `RunAsync` do serviço. O Service Fabric cancela a tarefa antes de desligar o nó.

**Recuperação**. Use o token de cancelamento para detectar o desligamento. Quando o Service Fabric solicitar o cancelamento, conclua qualquer trabalho e saia do  `RunAsync` assim que possível.

**Diagnóstico**. Logs de aplicativo

## <a name="storage"></a>Armazenamento
### <a name="writing-data-to-azure-storage-fails"></a>Falha na gravação de dados no Armazenamento do Azure
**Detecção**. O cliente recebe erros durante a gravação.

**Recuperação**

1. Tente a operação novamente para recuperar-se de falhas transitórias. A [política de novas tentativas][Storage.RetryPolicies] no SDK cliente trata disso automaticamente.
2. Implemente o padrão de Disjuntor para evitar armazenamento excessivo.
3. Se houver falha em N novas tentativas, execute um fallback normal. Por exemplo:

   * Armazene os dados em um cache local e encaminhe as gravações para o armazenamento mais tarde, quando o serviço estiver disponível.
   * Se a ação de gravação estiver em um escopo transacional, compense a transação.

**Diagnóstico**. Use [métricas de armazenamento][storage-metrics].

### <a name="reading-data-from-azure-storage-fails"></a>Falha na leitura de dados do Armazenamento do Azure.
**Detecção**. O cliente recebe erros durante a leitura.

**Recuperação**

1. Tente a operação novamente para recuperar-se de falhas transitórias. A [política de novas tentativas][Storage.RetryPolicies] no SDK cliente trata disso automaticamente.
2. Para o armazenamento RA-GRS, se houver falha na leitura do ponto de extremidade primário, tente ler do ponto de extremidade secundário. O SDK cliente pode tratar disso automaticamente. Veja [Replicação de Armazenamento do Azure][storage-replication].
3. Se houver falha em *N* novas tentativas, execute uma ação de fallback para degradar normalmente. Por exemplo, se uma imagem do produto não puder ser recuperada do armazenamento, mostre uma imagem de espaço reservado genérico.

**Diagnóstico**. Use [métricas de armazenamento][storage-metrics].

## <a name="virtual-machine"></a>Máquina Virtual
### <a name="connection-to-a-backend-vm-fails"></a>Falha na conexão a uma VM de back-end.
**Detecção**. Erros na conexão de rede.

**Recuperação**

* Implante pelo menos duas VMs de back-end em um conjunto de disponibilidade, atrás de um balanceador de carga.
* Se o erro na conexão for transitório, às vezes, o TCP tentará enviar a mensagem novamente com êxito.
* Implemente uma política de novas tentativas no aplicativo.
* Para erros persistentes ou não transitórios, implemente o padrão de [Disjuntor][circuit-breaker].
* Se a VM chamando exceder seu limite de saída de rede, a fila de saída será preenchida. Se a fila de saída estiver consistentemente cheia, considere a possibilidade de expansão.

**Diagnóstico**. Registre em log os eventos nos limites dos serviços.

### <a name="vm-instance-becomes-unavailable-or-unhealthy"></a>A instância VM torna-se não disponível ou não íntegra.
**Detecção**. Configure uma [investigação de integridade][lb-probe] do balanceador de carga que indique se a instância VM está íntegra. A investigação deve verificar se as funções críticas estão respondendo corretamente.

**Recuperação**. Para cada camada de aplicativo, coloque várias instâncias de VM no mesmo conjunto de disponibilidade e coloque um balanceador de carga na frente das VMs. Se houver falha na investigação de integridade, o balanceador de carga interromperá o envio de novas conexões para a instância não íntegra.

**Diagnóstico**. - Use a [análise de log][lb-monitor] do balanceador de carga.

* Configure o sistema de monitoramento para monitorar todos os pontos de extremidade de monitoramento de integridade.

### <a name="operator-accidentally-shuts-down-a-vm"></a>O operador desliga uma VM acidentalmente.
**Detecção**. N/D

**Recuperação**. Defina um bloqueio de recurso com nível `ReadOnly`. Veja [Bloquear recursos com o Azure Resource Manager][rm-locks].

**Diagnóstico**. Use os [Logs de Atividades do Azure][azure-activity-logs].

## <a name="webjobs"></a>Trabalhos Web
### <a name="continuous-job-stops-running-when-the-scm-host-is-idle"></a>A execução de trabalho contínuo é interrompida quando o host do SCM está ocioso.
**Detecção**. Passe um token de cancelamento para a função WebJob. Para obter mais informações, consulte [Desligamento normal][web-jobs-shutdown].

**Recuperação**. Habilite a configuração `Always On` no aplicativo Web. Para obter mais informações, consulte [Executar tarefas em segundo plano com o WebJobs][web-jobs].

## <a name="application-design"></a>Design do aplicativo
### <a name="application-cant-handle-a-spike-in-incoming-requests"></a>O aplicativo não pode tratar de um aumento nas solicitações de entrada.
**Detecção**. Depende do aplicativo. Sintomas comuns:

* O site é iniciado retornando códigos de erro HTTP 5xx.
* Serviços dependentes, como o banco de dados ou o armazenamento, iniciam as solicitações de limitação. Procure erros HTTP, como HTTP 429 (Número excessivo de solicitações), dependendo do serviço.
* O tamanho da Fila HTTP aumenta.

**Recuperação**

* Expanda para tratar do aumento de carga.
* Mitigue falhas para evitar que falhas em cascata interrompa o aplicativo inteiro. As estratégias de mitigação incluem:

  * Implementar o [Padrão de Limitação][throttling-pattern] para evitar sobrecarregar sistemas de back-end.
  * Usar o [nivelamento de carga baseado em fila][queue-based-load-leveling] para solicitações de buffer e processá-las em um ritmo apropriado.
  * Priorizar determinados clientes. Por exemplo, se o aplicativo tiver camadas gratuitas e pagas, limite os clientes na camada gratuita, mas não os clientes pagos. Veja [Padrão de fila de prioridade][priority-queue-pattern].

**Diagnóstico**. Use o [registro em log de diagnóstico do Serviço de Aplicativo][app-service-logging]. Use um serviço como o [Azure Log Analytics][azure-log-analytics], o [Application Insights][app-insights] ou o [New Relic][new-relic] para ajudar na compreensão dos logs de diagnóstico.

### <a name="one-of-the-operations-in-a-workflow-or-distributed-transaction-fails"></a>Falha em uma das operações em um fluxo de trabalho ou uma transação distribuída.
**Detecção**. Depois de *N* novas tentativas, ainda há falha.

**Recuperação**

* Como plano de mitigação, implemente o padrão do [Supervisor do Agente do Agendador][scheduler-agent-supervisor] para gerenciar o fluxo de trabalho inteiro.
* Não tente novamente quando o tempo limite esgotar. Há uma baixa taxa de êxito para esse erro.
* Coloque o trabalho na fila para tentar novamente mais tarde.

**Diagnóstico**. Registre em log todas as operações (bem-sucedidas e com falha), incluindo as ações de compensação. Use as IDs de correlação para que você possa acompanhar todas as operações na mesma transação.

### <a name="a-call-to-a-remote-service-fails"></a>Falha em uma chamada para um serviço remoto.
**Detecção**. Código de erro HTTP.

**Recuperação**

1. Tente novamente em caso de falhas transitórias.
2. Se a chamada falhar após *N* tentativas, execute uma ação de fallback. (Depende do aplicativo.)
3. Implemente o [padrão de Disjuntor][circuit-breaker] para evitar falhas em cascata.

**Diagnóstico**. Registre em log todas as falhas de chamada remota.

## <a name="next-steps"></a>Próximas etapas
Para obter mais informações sobre o processo FMA, veja [Resilience by design for cloud services][resilience-by-design-pdf] (Resiliência por design para serviços de nuvem) (download de PDF).

<!-- links -->

[api-management]: https://azure.microsoft.com/documentation/services/api-management/
[api-management-throttling]: /azure/api-management/api-management-sample-flexible-throttling/
[app-insights]: /azure/application-insights/app-insights-overview/
[app-insights-web-apps]: /azure/application-insights/app-insights-azure-web-apps/
[app-service-configure]: /azure/app-service-web/web-sites-configure/
[app-service-logging]: /azure/app-service-web/web-sites-enable-diagnostic-log/
[app-service-slots]: /azure/app-service-web/web-sites-staged-publishing/
[auto-rest-client-retry]: https://github.com/Azure/autorest/tree/master/docs
[azure-activity-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs/
[azure-alerts]: /azure/monitoring-and-diagnostics/insights-alerts-portal/
[azure-log-analytics]: /azure/log-analytics/log-analytics-overview/
[BrokeredMessage.TimeToLive]: https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx
[cassandra-error-handling]: http://www.datastax.com/dev/blog/cassandra-error-handling-done-right
[circuit-breaker]: https://msdn.microsoft.com/library/dn589784.aspx
[cosmosdb-multi-region]: /azure/cosmos-db/tutorial-global-distribution-sql-api
[elasticsearch-azure]: ../elasticsearch/index.md
[elasticsearch-client]: https://www.elastic.co/guide/en/elasticsearch/client/index.html
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[onstop-events]: https://azure.microsoft.com/blog/the-right-way-to-handle-azure-onstop-events/
[lb-monitor]: /azure/load-balancer/load-balancer-monitor-log/
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview/#learn-about-the-types-of-probes
[new-relic]: https://newrelic.com/
[priority-queue-pattern]: https://msdn.microsoft.com/library/dn589794.aspx
[queue-based-load-leveling]: https://msdn.microsoft.com/library/dn589783.aspx
[QuotaExceededException]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx
[ra-web-apps-basic]: ../reference-architectures/app-service-web-app/basic-web-app.md
[redis-monitor]: /azure/redis-cache/cache-how-to-monitor/
[redis-retry]: ../best-practices/retry-service-specific.md#azure-redis-cache
[resilience-by-design-pdf]: http://download.microsoft.com/download/D/8/C/D8C599A4-4E8A-49BF-80EE-FE35F49B914D/Resilience_by_Design_for_Cloud_Services_White_Paper.pdf
[RoleEntryPoint.OnStop]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[RoleEnvironment.Stopping]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.stopping.aspx
[rm-locks]: /azure/azure-resource-manager/resource-group-lock-resources/
[sb-dead-letter-queue]: /azure/service-bus-messaging/service-bus-dead-letter-queues/
[sb-georeplication-sample]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/GeoReplication
[sb-messagingexception-class]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagingexception.aspx
[sb-messaging-exceptions]: /azure/service-bus-messaging/service-bus-messaging-exceptions/
[sb-outages]: /azure/service-bus-messaging/service-bus-outages-disasters/#protecting-queues-and-topics-against-datacenter-outages-or-disasters
[sb-partition]: /azure/service-bus-messaging/service-bus-partitioning/
[sb-poison-message]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#poison
[sb-retry]: ../best-practices/retry-service-specific.md#service-bus
[search-sdk]: https://msdn.microsoft.com/library/dn951165.aspx
[scheduler-agent-supervisor]: https://msdn.microsoft.com/library/dn589780.aspx
[search-analytics]: /azure/search/search-traffic-analytics/
[sql-db-audit]: /azure/sql-database/sql-database-auditing-get-started/
[sql-db-errors]: /azure/sql-database/sql-database-develop-error-messages/#resource-governance-errors
[sql-db-failover]: /azure/sql-database/sql-database-geo-replication-failover-portal/
[sql-db-limits]: /azure/sql-database/sql-database-resource-limits/
[sql-db-replication]: /azure/sql-database/sql-database-geo-replication-overview/
[storage-metrics]: https://msdn.microsoft.com/library/dn782843.aspx
[storage-replication]: /azure/storage/storage-redundancy/
[Storage.RetryPolicies]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.aspx
[sys.event_log]: https://msdn.microsoft.com/library/dn270018.aspx
[throttling-pattern]: https://msdn.microsoft.com/library/dn589798.aspx
[web-jobs]: /azure/app-service-web/web-sites-create-web-jobs/
[web-jobs-shutdown]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#graceful
