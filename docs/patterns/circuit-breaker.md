---
title: Disjuntor
description: Trate as falhas que possam consumir uma quantidade variável de tempo para serem corrigidas ao se conectar a um serviço ou recurso remoto.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- resiliency
ms.openlocfilehash: 0f93c1ef664c8e7385895e3854835699f674ee0e
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/30/2018
---
# <a name="circuit-breaker-pattern"></a>Padrão de Disjuntor

Trate as falhas que possam consumir uma quantidade variável de tempo para serem recuperadas ao se conectar a um serviço ou recurso remoto. Isso pode melhorar a estabilidade e a resiliência de um aplicativo.

## <a name="context-and-problem"></a>Contexto e problema

Em um ambiente distribuído, as chamadas a serviços e recursos remotos podem falhar devido a falhas transitórias, como conexões de rede lentas, tempos limite esgotados ou recursos excessivamente confirmados ou temporariamente não disponíveis. Essas falhas geralmente são corrigidas automaticamente após um breve período de tempo e um aplicativo de nuvem robusto deve estar preparado para tratá-los usando uma estratégia, como o [Padrão de repetição][retry-pattern].

No entanto, também pode haver situações em que as falhas ocorrem devido a eventos inesperados e que podem demorar muito mais para serem corrigidos. Essas falhas podem variar de gravidade de uma perda parcial de conectividade até a falha completa de um serviço. Nessas situações, pode ser inútil para um aplicativo tentar repetir continuamente uma operação em que o êxito seja improvável. Em vez disso, o aplicativo deve aceitar rapidamente a falha na operação e tratá-la adequadamente.

Além disso, se um serviço estiver muito ocupado, a falha em uma parte do sistema poderá resultar em falhas em cascata. Por exemplo, uma operação que invoca um serviço pode ser configurada para implementar um tempo limite e responder com uma mensagem de falha se o serviço não responder nesse período. No entanto, essa estratégia pode causar muitas solicitações simultâneas para a mesma operação a ser bloqueada até a expiração do tempo limite. Essas solicitações bloqueadas podem conter recursos críticos do sistema, como memória, threads, conexões de banco de dados, etc. Consequentemente, esses recursos podem se esgotar, causando falha em outras partes possivelmente não relacionadas do sistema que precisam usar os mesmos recursos. Nessas situações, seria preferível que a operação falhasse imediatamente e somente tentasse invocar o serviço se o êxito fosse provável. Observe que a configuração de um tempo limite menor talvez ajude a resolver esse problema, mas o tempo limite não deve ser tão curto de modo a causar a falha da operação na maioria das vezes, mesmo se a solicitação ao serviço fosse bem-sucedida no final das contas.

## <a name="solution"></a>Solução

O padrão de Disjuntor popularizado por Michael Nygard em seu livro [Release It!](https://pragprog.com/book/mnee/release-it) (Libere!), pode impedir que um aplicativo tente executar repetidamente uma operação em que a falha é provável. Ele permite continuar sem aguardar que a falha seja corrigida ou sem desperdiçar ciclos de CPU enquanto determina-se que a falha é de longa duração. O padrão de Disjuntor também permite que um aplicativo detecte se a falha foi resolvida. Se o problema aparentar ter sido corrigido, o aplicativo poderá tentar invocar a operação.

> A finalidade do padrão de Disjuntor é diferente do padrão de Repetição. O padrão de Repetição permite que um aplicativo tente novamente uma operação na expectativa que haverá êxito. O padrão de Disjuntor impede que um aplicativo tente execute uma operação em que a falha é provável. Um aplicativo pode combinar esses dois padrões usando o padrão de Repetição para invocar uma operação por meio de um disjuntor. No entanto, a lógica de repetição deverá ser sensível às exceções retornadas pelo disjuntor e abandonar as novas tentativas se o disjuntor indicar que uma falha não é transitória.

Um disjuntor atua como um proxy para operações que podem falhar. O proxy deve monitorar o número de falhas recentes ocorridas e usar essas informações para decidir se deve permitir que a operação continue ou apenas retornar uma exceção imediatamente.

O proxy pode ser implementado como um computador de estado com os seguintes estados que imitam a funcionalidade de um disjuntor elétrico:

- **Fechado**: a solicitação do aplicativo é encaminhada para a operação. O proxy mantém a contagem do número de falhas recentes e, se a chamada à operação não for bem-sucedida, o proxy incrementará essa contagem. Se o número de falhas recentes exceder um limite especificado em um determinado período de tempo, o proxy será colocado no estado **Aberto**. Nesse momento, o proxy inicia um temporizador de tempo limite e, quando o temporizador expira, o proxy é colocado no estado **Entreaberto**.

    > A finalidade do temporizador de tempo limite é dar tempo ao sistema para corrigir o problema que causou a falha antes de permitir que o aplicativo tente executar a operação novamente.

- **Aberto**: a solicitação do aplicativo falha imediatamente e uma exceção é retornada ao aplicativo.

- **Entreaberto**: um número limitado de solicitações do aplicativo pode ser passado e invocar a operação. Se essas solicitações forem bem-sucedidas, será considerado que o problema anterior que estava causando a falha foi corrigido e o disjuntor mudará para o estado **Fechado** (o contador de falhas será reiniciado). Se alguma solicitação falhar, o disjuntor considerará que a falha ainda está presente, portanto voltará para o estado **Aberto** e reiniciará o temporizador de tempo limite para dar mais tempo ao sistema para se recuperar da falha.

    > O estado **Entreaberto** é útil para impedir que um serviço de recuperação seja inundado repentinamente com solicitações. Conforme um serviço é recuperado, ele pode dar suporte a um volume limitado de solicitações até que a recuperação seja concluída, mas enquanto a recuperação estiver em andamento, um fluxo grande de trabalho poderá fazer com que o tempo limite do serviço seja atingido ou falhe novamente.

![Estados do Disjuntor](./_images/circuit-breaker-diagram.png)

Na figura, o contador de falhas é usado no estado **Fechado** é baseado em tempo. Ele é reiniciado automaticamente em intervalos periódicos. Isso ajuda a impedir que o disjuntor entre no estado **Aberto** em caso de falhas ocasionais. O limite de falhas que aciona o disjuntor no estado **Aberto** só é atingido quando um número especificado de falhas ocorrer durante um intervalo especificado. O contador usado no estado **Entreaberto** registra o número de tentativas bem-sucedidas para invocar a operação. O disjuntor será revertido para o estado **Fechado** após um número especificado de invocações bem-sucedidas da operação ser obtido de maneira consecutiva. Se alguma invocação falhar, o disjuntor entrará no estado **Aberto** imediatamente e o contador de êxito será reiniciado na próxima vez que ele entrar no estado **Entreaberto**.

> O modo que o sistema é recuperado é tratado de maneira externa, possivelmente restaurando ou reiniciando um componente com falha ou reparando uma conexão de rede.

O padrão de Disjuntor fornece estabilidade enquanto o sistema recupera-se de uma falha e minimiza o impacto no desempenho. Ele pode ajudar a manter o tempo de resposta do sistema rejeitando rapidamente uma solicitação para uma operação em que a falha é provável, em vez de aguardar o tempo limite da operação ou nunca retornar. Se o disjuntor acionar um evento sempre que mudar de estado, essas informações poderão ser usadas para monitorar a integridade da parte do sistema protegido pelo disjuntor ou para alertar o administrador quando um disjuntor acionar o estado **Aberto**.

O padrão é personalizável e pode ser adaptado de acordo com o tipo de falha possível. Por exemplo, você pode aplicar um temporizador de tempo limite crescente para um disjuntor. Você pode colocar o disjuntor no estado **Aberto** inicialmente por alguns segundos e, se a falha não for resolvida, aumentar o tempo limite para alguns minutos e assim por diante. Em alguns casos, em vez de o estado **Aberto** retornar a falha e gerar uma exceção, pode ser útil retornar um valor padrão que seja significativo para o aplicativo.

## <a name="issues-and-considerations"></a>Problemas e considerações

Os seguintes pontos devem ser considerados ao decidir como implementar esse padrão:

**Tratamento de exceção**. Um aplicativo invocando uma operação por meio de um disjuntor deverá estar preparado para tratar as exceções geradas se a operação não estiver disponível. A maneira como são as exceções são tratadas dependerá do aplicativo. Por exemplo, um aplicativo pode sofrer uma degradação temporária de sua funcionalidade, invocar uma operação alternativa para tentar executar a mesma tarefa ou obter os mesmos dados ou relatar a exceção ao usuário e pedir a que ele tente novamente mais tarde.

**Tipos de exceções**. Uma solicitação pode falhar por vários motivos, algumas das quais podem indicar um tipo mais grave de falha que outros. Por exemplo, uma solicitação pode falhar por cause de uma falha em um serviço remoto e sua recuperação demorará vários minutos ou porque o tempo limite foi atingido, pois o serviço está temporariamente sobrecarregado. Um disjuntor poderá examinar os tipos de exceções que ocorrem e ajustar a estratégia dependendo da natureza dessas exceções. Por exemplo, ele pode exigir um número grande de exceções de tempo limite para acionar o estado **Aberto** do disjuntor em relação ao número de falhas devido ao serviço estar completamente não disponível.

**Registro em log**. Um disjuntor deve registrar em log todas as solicitações com falha (e possivelmente as solicitações bem-sucedidas) para permitir que um administrador monitore a integridade da operação.

**Capacidade de recuperação**. Você deve configurar o disjuntor para coincidir com o padrão de probabilidade de recuperação da operação sendo protegida. Por exemplo, se o disjuntor permanece no estado **Aberto** por um longo período, ele pode gerar exceções mesmo se o motivo da falha foi resolvido. De forma semelhante, um disjuntor pode flutuar e reduzir os tempos de resposta dos aplicativos se mudar do estado **Aberto** para o estado **Entreaberto** muito rapidamente.

**Teste de operações com falha**. No estado **Aberto**, em vez de usar um temporizador para determinar quando mudar para o estado **Entreaberto**, um disjuntor pode executar ping periodicamente no serviço remoto ou no recurso para determinar se ele fica disponível novamente. Esse ping pode assumir a forma de uma tentativa de invocar uma operação que falhou anteriormente ou pode usar uma operação especial fornecida pelo serviço remoto especificamente para testar a integridade do serviço, conforme descrito pelo [Padrão de monitoramento do ponto de extremidade de integridade](health-endpoint-monitoring.md).

**Substituição manual**. Em um sistema em que o tempo de recuperação de uma operação com falha é extremamente variável, é útil fornecer uma opção de reinício manual que permita ao administrador fechar um disjuntor (e reiniciar o contador de falhas). De modo semelhante, um administrador poderá forçar um disjuntor a entrar no estado **Aberto** (e reiniciar o temporizador de tempo limite) se a operação protegida pelo disjuntor estiver temporariamente não disponível.

**Simultaneidade**. O mesmo disjuntor pode ser acessado por um grande número de instâncias simultâneas de um aplicativo. A implementação não deve bloquear solicitações simultâneas nem adicionar uma sobrecarga excessiva a cada chamada a uma operação.

**Diferenciação de recurso**. Tenha cuidado ao usar um único disjuntor para um tipo de recurso se houver vários provedores independentes subjacentes. Por exemplo, em um armazenamento de dados contendo vários fragmentos, um fragmento pode estar totalmente acessível enquanto outro está enfrentando um problema temporário. Se as respostas de erro nesses cenários forem mescladas, um aplicativo poderá tentar acessar alguns fragmentos, mesmo quando a falha for altamente provável, enquanto o acesso aos outros fragmentos poderá ser bloqueado, mesmo que provavelmente tenham êxito.

**Disjuntor acelerado**. Às vezes, uma resposta de falha pode conter informações suficientes para que o disjuntor seja acionado imediatamente e permaneça assim por uma quantidade mínima de tempo. Por exemplo, a resposta de erro de um recurso compartilhado sobrecarregado pode indicar que uma nova tentativa imediata não seja recomendada e que o aplicativo, em vez disso, devesse tentar novamente em alguns minutos.

> [!NOTE]
> Um serviço poderá retornar HTTP 429 (Número excessivo de solicitações) se estiver limitando o cliente ou HTTP 503 (Serviço não disponível) se o serviço não estiver disponível no momento. A resposta pode incluir informações adicionais, como a duração prevista do atraso.

**Reprodução das solicitações com falha**. No estado **Aberto**, em vez de simplesmente falhar rapidamente, um disjuntor também poderá registrar os detalhes de cada solicitação em um diário e organizar essas solicitações para serem reproduzidas quando o serviço ou o recurso estiver disponível.

**Tempos limite inadequados em serviços externos**. Um disjuntor talvez não possa proteger totalmente os aplicativos das operações que falham em serviços externos configurados com um tempo limite longo. Se o tempo limite for muito longo, um thread executando um disjuntor poderá ser bloqueado por um longo período antes de o disjuntor indicar a falha na operação. Nesse momento, muitas outras instâncias do aplicativo também poderão tentar invocar o serviço por meio do disjuntor e associar um número significativo de threads antes que todos falhem.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use este padrão:

- Para impedir que um aplicativo tente invocar um serviço remoto ou acessar um recurso compartilhado se a operação tiver alta probabilidade de falhar.

Este padrão não é recomendável:

- Para tratar do acesso a recursos particulares locais em um aplicativo, como a estrutura de dados na memória. Nesse ambiente, usar um disjuntor poderia adicionar uma sobrecarga ao sistema.
- Como substituto para o tratamento de exceções na lógica de negócios dos seus aplicativos.

## <a name="example"></a>Exemplo

Em um aplicativo Web, muitas das páginas são populadas com os dados recuperados de um serviço externo. Se o sistema implementar um armazenamento em cache mínimo, a maioria das ocorrências dessas páginas provocará uma viagem de ida e volta para o serviço. As conexões do aplicativo Web ao serviço podem ser configuradas com um tempo limite (normalmente 60 segundos) e, se o serviço não responder nesse período, a lógica em cada página da Web considerará que o serviço não está disponível e gerará uma exceção.

No entanto, se o serviço falhar e o sistema estiver muito ocupado, os usuários poderão ser forçados a aguardar até 60 segundos antes que ocorra uma exceção. Com o tempo, recursos como memória, conexões e threads poderão ser esgotados, evitando que outros usuários se conectem ao sistema, mesmo que não estejam acessando páginas que recuperam dados do serviço.

O dimensionamento do sistema com a adição de mais servidores Web e a implementação do balanceamento de carga poderá sofrer atrasos quando os recursos acabarem, mas isso não resolverá o problema, uma vez que as solicitações do usuário ainda estarão sem resposta e todos os servidores Web ainda poderão ter seus recursos esgotados.

O encapsulamento da lógica que se conecta ao serviço e recupera os dados em um disjuntor pode ajudar a resolver esse problema e tratar a falha do serviço de modo mais elegante. As solicitações do usuário ainda falharão, mas elas falharão mais rapidamente e os recursos não serão bloqueados.

A classe `CircuitBreaker` mantém informações de estado relacionadas a um disjuntor em um objeto que implementa a interface `ICircuitBreakerStateStore` mostrada no código a seguir.

```csharp
interface ICircuitBreakerStateStore
{
  CircuitBreakerStateEnum State { get; }

  Exception LastException { get; }

  DateTime LastStateChangedDateUtc { get; }

  void Trip(Exception ex);

  void Reset();

  void HalfOpen();

  bool IsClosed { get; }
}
```

A propriedade `State` indica o estado atual do disjuntor e será **Open** (Aberto), **HalfOpen** (Entreaberto) ou **Closed** (Fechado) conforme definido pela enumeração `CircuitBreakerStateEnum`. A propriedade `IsClosed` deverá ser true se o disjuntor estiver fechado, mas false se ele estiver aberto ou entreaberto. O método `Trip` muda o estado do disjuntor para o estado aberto e registra a exceção que causou a alteração no estado, junto com a data e a hora em que a exceção ocorreu. As propriedades `LastException` e `LastStateChangedDateUtc` retornam essas informações. O método `Reset` fecha o disjuntor e o método `HalfOpen` define o disjuntor como entreaberto.

A classe `InMemoryCircuitBreakerStateStore` no exemplo contém uma implementação da interface `ICircuitBreakerStateStore`. A classe `CircuitBreaker` cria uma instância dessa classe para manter o estado do disjuntor.

O método `ExecuteAction` na classe `CircuitBreaker` encapsula uma operação, especificada como um delegado `Action`. Se o disjuntor estiver fechado, `ExecuteAction` invocará o delegado `Action`. Se a operação falhar, um manipulador de exceção chamará `TrackException`, que definirá o estado do disjuntor como aberto. O exemplo de código a seguir realça esse fluxo.

```csharp
public class CircuitBreaker
{
  private readonly ICircuitBreakerStateStore stateStore =
    CircuitBreakerStateStoreFactory.GetCircuitBreakerStateStore();

  private readonly object halfOpenSyncObject = new object ();
  ...
  public bool IsClosed { get { return stateStore.IsClosed; } }

  public bool IsOpen { get { return !IsClosed; } }

  public void ExecuteAction(Action action)
  {
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open.
      ... (see code sample below for details)
    }

    // The circuit breaker is Closed, execute the action.
    try
    {
      action();
    }
    catch (Exception ex)
    {
      // If an exception still occurs here, simply
      // retrip the breaker immediately.
      this.TrackException(ex);

      // Throw the exception so that the caller can tell
      // the type of exception that was thrown.
      throw;
    }
  }

  private void TrackException(Exception ex)
  {
    // For simplicity in this example, open the circuit breaker on the first exception.
    // In reality this would be more complex. A certain type of exception, such as one
    // that indicates a service is offline, might trip the circuit breaker immediately.
    // Alternatively it might count exceptions locally or across multiple instances and
    // use this value over time, or the exception/success ratio based on the exception
    // types, to open the circuit breaker.
    this.stateStore.Trip(ex);
  }
}
```

O exemplo a seguir mostra o código (omitido do exemplo anterior) executado se o disjuntor não estiver fechado. Ele primeiro verifica se o disjuntor esteve aberto por um período maior que o tempo especificado pelo campo `OpenToHalfOpenWaitTime` local na classe `CircuitBreaker`. Se esse for o caso, o método `ExecuteAction` definirá o disjuntor como entreaberto e, em seguida, tentará executar a operação especificada pelo delegado `Action`.

Se a operação for bem-sucedida, o disjuntor será reiniciado no estado fechado. Se a operação falhar, ele voltará ao estado aberto e a hora em que a exceção ocorreu será atualizada para que o disjuntor aguarde mais um pouco antes de tentar executar a operação novamente.

Se o disjuntor só tiver sido aberto por um curto período de tempo, menor que o valor `OpenToHalfOpenWaitTime`, o método `ExecuteAction` simplesmente gerará uma exceção `CircuitBreakerOpenException` e retornará o erro que fez com que o disjuntor fosse para o estado aberto.

Além disso, ele usará um bloqueio para impedir que o disjuntor tente realizar chamadas simultâneas à operação enquanto estiver entreaberto. Uma tentativa simultânea de invocar a operação será tratada como se o disjuntor estivesse aberto e falhará com uma exceção, conforme descrito posteriormente.

```csharp
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open. Check if the Open timeout has expired.
      // If it has, set the state to HalfOpen. Another approach might be to
      // check for the HalfOpen state that had be set by some other operation.
      if (stateStore.LastStateChangedDateUtc + OpenToHalfOpenWaitTime < DateTime.UtcNow)
      {
        // The Open timeout has expired. Allow one operation to execute. Note that, in
        // this example, the circuit breaker is set to HalfOpen after being
        // in the Open state for some period of time. An alternative would be to set
        // this using some other approach such as a timer, test method, manually, and
        // so on, and check the state here to determine how to handle execution
        // of the action.
        // Limit the number of threads to be executed when the breaker is HalfOpen.
        // An alternative would be to use a more complex approach to determine which
        // threads or how many are allowed to execute, or to execute a simple test
        // method instead.
        bool lockTaken = false;
        try
        {
          Monitor.TryEnter(halfOpenSyncObject, ref lockTaken);
          if (lockTaken)
          {
            // Set the circuit breaker state to HalfOpen.
            stateStore.HalfOpen();

            // Attempt the operation.
            action();

            // If this action succeeds, reset the state and allow other operations.
            // In reality, instead of immediately returning to the Closed state, a counter
            // here would record the number of successful operations and return the
            // circuit breaker to the Closed state only after a specified number succeed.
            this.stateStore.Reset();
            return;
          }
          catch (Exception ex)
          {
            // If there's still an exception, trip the breaker again immediately.
            this.stateStore.Trip(ex);

            // Throw the exception so that the caller knows which exception occurred.
            throw;
          }
          finally
          {
            if (lockTaken)
            {
              Monitor.Exit(halfOpenSyncObject);
            }
          }
        }
      }
      // The Open timeout hasn't yet expired. Throw a CircuitBreakerOpen exception to
      // inform the caller that the call was not actually attempted,
      // and return the most recent exception received.
      throw new CircuitBreakerOpenException(stateStore.LastException);
    }
    ...
```

Para usar um objeto `CircuitBreaker` para proteger uma operação, um aplicativo criará uma instância da classe `CircuitBreaker` e invocará o método `ExecuteAction`, especificando a operação a ser executada como o parâmetro. O aplicativo deverá estar preparado para detectar a exceção `CircuitBreakerOpenException` se a operação falhar porque o disjuntor estará aberto. O código a seguir mostra um exemplo:

```csharp
var breaker = new CircuitBreaker();

try
{
  breaker.ExecuteAction(() =>
  {
    // Operation protected by the circuit breaker.
    ...
  });
}
catch (CircuitBreakerOpenException ex)
{
  // Perform some different action when the breaker is open.
  // Last exception details are in the inner exception.
  ...
}
catch (Exception ex)
{
  ...
}
```

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados

Os seguintes padrões também serão úteis ao implementar este padrão:

- [Padrão de repetição][retry-pattern]. Descreve como um aplicativo pode tratar falhas previstas e temporárias quando tentar se conectar a um serviço ou recurso de rede repetindo de forma transparente uma operação que falhou anteriormente.

- [Padrão de monitoramento do ponto de extremidade de integridade](health-endpoint-monitoring.md). Um disjuntor pode testar a integridade de um serviço enviando uma solicitação para um ponto de extremidade exposto pelo serviço. O serviço deve retornar informações indicando seu status.


[retry-pattern]: ./retry.md
