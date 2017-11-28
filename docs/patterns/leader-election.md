---
title: "Eleição de Líder"
description: "Coordene as ações executadas por uma coleção de instâncias de tarefa de colaboração em um aplicativo distribuído elegendo uma instância como a líder que assume a responsabilidade por gerenciar as demais instâncias."
keywords: "padrão de design"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- resiliency
ms.openlocfilehash: ddb61097ed3229ed0ed517b94c280d3ef892c999
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="leader-election-pattern"></a>Padrão de eleição de líder

[!INCLUDE [header](../_includes/header.md)]

Coordene as ações executadas por uma coleção de instâncias de colaboração em um aplicativo distribuído elegendo uma instância como o líder que assume a responsabilidade de gerenciar as demais. Isso pode ajudar a garantir que as instâncias não entrem em conflito com outras, causar contenção de recursos compartilhados ou interferir inadvertidamente no trabalho que outras instâncias estão executando.

## <a name="context-and-problem"></a>Contexto e problema

Um aplicativo de nuvem típico tem muitas tarefas que funcionam de maneira coordenada. Essas tarefas poderiam ser instâncias que executam o mesmo código e precisam de acessar aos mesmos recursos ou podem estar trabalhando juntas em paralelo para executar as partes individuais de um cálculo complexo.

As instâncias de tarefa poderão ser executadas separadamente pela maior parte do tempo, mas também pode ser necessário coordenar as ações de cada instância para garantir que eles não entrem em conflito, causem contenção dos recursos compartilhados ou interfiram acidentalmente no trabalho que outras instâncias de tarefa estão executando.

Por exemplo:

- Em um sistema baseado em nuvem que implementa o dimensionamento horizontal, várias instâncias da mesma tarefa podem estar em execução ao mesmo tempo com cada instância atendendo a um usuário diferente. Se essas instâncias gravarem em um recurso compartilhado, será necessário coordenar suas ações para impedir que cada instância substitua as alterações feitas por outras.
- Se as tarefas estão executando elementos individuais de um cálculo complexo em paralelo, os resultados precisam ser agregados quando todos forem concluídos.

Todas as instâncias de tarefa são pares, portanto não há um líder natural que possa atuar como o coordenador ou agregador.

## <a name="solution"></a>Solução

Uma única instância de tarefa deve ser eleita para atuar como líder e essa instância deve coordenar as ações de outras instâncias de tarefa subordinada. Se todas as instâncias de tarefa executarem o mesmo código, qualquer uma delas poderá atuar como o líder. Portanto, o processo de eleição deve ser gerenciado com cuidado para evitar que duas ou mais instâncias assumam a função de líder ao mesmo tempo.

O sistema deve fornecer um mecanismo robusto para selecionar o líder. Esse método precisa lidar com eventos como interrupções da rede ou falhas de processo. Em muitas soluções, as instâncias de tarefa subordinadas monitoram o líder por meio de algum tipo de método de pulsação ou por meio de sondagem. Se o líder designado terminar inesperadamente ou se uma falha de rede deixar o líder indisponível para as instâncias de tarefa subordinadas, será necessário eleger um novo líder.

Há várias estratégias para eleger o líder entre um conjunto de tarefas em um ambiente distribuído, incluindo:
- Selecionar a instância da tarefa com a ID de processo ou instância mais baixa.
- Corrida para adquirir um mutex compartilhado e distribuído. A primeira instância de tarefa que adquirir o mutex é o líder. No entanto, o sistema deve garantir que, se o líder for encerrado ou desconectado do restante do sistema, o mutex será liberado para permitir que outra instância de tarefa torne-se o líder.
- Implementação de um dos algoritmos comuns de eleição de líder, como o [Algoritmo Bully](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) ou [Algoritmo Ring](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html). Esses algoritmos presumem que cada candidato na eleição tem uma ID exclusiva e que ele pode se comunicar com os outros candidatos de maneira confiável.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:
- O processo de eleger um líder deve ser resistente a falhas transitórias e persistentes.
- Deve ser possível detectar quando o líder falhou ou ficou indisponível (por exemplo, devido a uma falha de comunicação). A rapidez de detecção necessária depende do sistema. Alguns sistemas podem funcionar por um curto período sem um líder, durante o qual uma falha temporária pode ser corrigida. Em outros casos, pode ser necessário detectar a falha do líder imediatamente e disparar uma nova eleição.
- Em um sistema que implementa o dimensionamento automático horizontal, o líder poderia ser terminado se o sistema for reduzido e desligar alguns dos recursos de computação.
- Usar um mutex compartilhado e distribuído apresenta uma dependência do serviço externo que fornece o mutex. O serviço constitui um ponto único de falha. Se ele ficar indisponível por qualquer motivo, o sistema não poderá eleger um líder.
- Usar um único processo dedicado como o líder é uma abordagem simples. No entanto, se o processo falhar, poderá ocorrer um atraso significativo enquanto ele é reiniciado. A latência resultante pode afetar o desempenho e os tempos de resposta de outros processos se eles estiverem esperando o líder coordenar uma operação.
- Implementar um dos algoritmos de eleição de líder manualmente oferece a maior flexibilidade possível para ajustar e otimizar o código.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use esse padrão quando as tarefas em um aplicativo distribuído, como uma solução hospedada na nuvem, precisarem de coordenação cuidadosa e não houver nenhum líder natural.

>  Evite transformar o líder em um gargalo no sistema. É a finalidade do líder coordenar o trabalho das tarefas subordinadas e ele não precisa necessariamente participar desse trabalho em si &mdash; embora deva ser capaz de fazer isso se a tarefa não for eleita como o líder.

Esse padrão pode não ser útil se:
- Há um líder natural ou processo dedicado que sempre pode atuar como o líder. Por exemplo, seria possível implementar um processo de singleton que coordena as instâncias de tarefa. Se esse processo falhar ou se tornar não íntegro, o sistema poderá desligá-lo e reiniciá-lo.
- A coordenação entre as tarefas pode ser alcançada usando um método mais simples. Por exemplo, se várias instâncias de tarefa precisam simplesmente de acesso coordenado a um recurso compartilhado, uma solução melhor é usar o bloqueio otimista ou pessimista para controlar o acesso.
- Uma solução de terceiros é mais apropriada. Por exemplo, o serviço do Microsoft Azure HDInsight (baseado em Apache Hadoop) usa os serviços fornecidos pelo Apache Zookeeper para coordenar o mapa e reduzir as tarefas que coletam e resumem os dados.

## <a name="example"></a>Exemplo

O projeto DistributedMutex na solução LeaderElection (um exemplo que demonstra esse padrão está disponível no [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)) mostra como usar uma concessão em um Azure Storage Blob para fornecer um mecanismo para implementar um mutex compartilhado e distribuído. O mutex pode ser usado para eleger um líder dentre um grupo de instâncias de função em um serviço de nuvem do Azure. A primeira instância de função a adquirir a concessão é eleita como líder e permanece dessa forma até liberar a concessão ou não for capaz de renová-la. Outras instâncias de função podem continuar a monitorar a concessão de blob caso o líder não esteja mais disponível.

>  Uma concessão de blob é um bloqueio de gravação exclusivo em um blob. Um único blob pode estar sujeito a apenas uma concessão ao mesmo tempo. Uma instância de função pode solicitar uma concessão em um blob especificado e ele receberá a concessão se nenhuma outra instância de função mantém uma concessão no mesmo blob. Caso contrário, a solicitação gerará uma exceção.

> Para evitar que uma instância de função com falha retenha a concessão indefinidamente, especifique um tempo de vida para a concessão. Quando ele expirar, a concessão ficará disponível. No entanto, enquanto uma instância de função contém a concessão, ela pode solicitar a renovação da concessão e ela será concedida por um período adicional. A instância de função pode repetir continuamente esse processo se desejar manter a concessão.
Para obter mais informações sobre como conceder um blob, consulte [Blob de concessão (API REST)](https://msdn.microsoft.com/library/azure/ee691972.aspx).

A classe `BlobDistributedMutex` no exemplo de C# a seguir contém o método `RunTaskWhenMutexAquired` que permite que uma instância de função tente adquirir uma concessão em um blob especificado. Os detalhes do blob (o nome, contêiner e conta de armazenamento) são passados para o construtor em um objeto `BlobSettings` quando o objeto `BlobDistributedMutex` é criado (esse objeto é uma struct simples incluído no código de exemplo). O construtor também aceita um `Task` que referencia o código que a instância de função deve executar se adquirir a concessão do blob com êxito e for eleito como líder. Observe que o código que manipula os detalhes de nível baixo da aquisição da concessão é implementado em uma classe auxiliar separada chamada `BlobLeaseManager`.

```csharp
public class BlobDistributedMutex
{
  ...
  private readonly BlobSettings blobSettings;
  private readonly Func<CancellationToken, Task> taskToRunWhenLeaseAcquired;
  ...

  public BlobDistributedMutex(BlobSettings blobSettings,
           Func<CancellationToken, Task> taskToRunWhenLeaseAquired)
  {
    this.blobSettings = blobSettings;
    this.taskToRunWhenLeaseAquired = taskToRunWhenLeaseAquired;
  }

  public async Task RunTaskWhenMutexAcquired(CancellationToken token)
  {
    var leaseManager = new BlobLeaseManager(blobSettings);
    await this.RunTaskWhenBlobLeaseAcquired(leaseManager, token);
  }
  ...
```

O método `RunTaskWhenMutexAquired` no exemplo de código acima invoca o método `RunTaskWhenBlobLeaseAcquired` mostrado no exemplo de código a seguir para, na verdade, adquirir a concessão. O método `RunTaskWhenBlobLeaseAcquired` é executado de forma assíncrona. Se a concessão for adquirida com êxito, a instância de função foi eleita como o líder. A finalidade do delegado `taskToRunWhenLeaseAcquired` é executar o trabalho que coordena as outras instâncias de função. Se a concessão não for adquirida, outra instância de função foi eleita como o líder e a instância de função atual permanece sendo uma subordinada. Observe que o `TryAcquireLeaseOrWait` é um método auxiliar que usa o objeto `BlobLeaseManager` para adquirir a concessão.

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (!token.IsCancellationRequested)
    {
      // Try to acquire the blob lease.
      // Otherwise wait for a short time before trying again.
      string leaseId = await this.TryAquireLeaseOrWait(leaseManager, token);

      if (!string.IsNullOrEmpty(leaseId))
      {
        // Create a new linked cancellation token source so that if either the
        // original token is canceled or the lease can't be renewed, the
        // leader task can be canceled.
        using (var leaseCts =
          CancellationTokenSource.CreateLinkedTokenSource(new[] { token }))
        {
          // Run the leader task.
          var leaderTask = this.taskToRunWhenLeaseAquired.Invoke(leaseCts.Token);
          ...
        }
      }
    }
    ...
  }
```

A tarefa iniciada pelo líder também é executada assincronamente. Enquanto essa tarefa está em execução, o método `RunTaskWhenBlobLeaseAquired` mostrado no exemplo de código a seguir tenta periodicamente renovar a concessão. Isso ajuda a garantir que a instância de função permaneça sendo o líder. Na solução de exemplo, o atraso entre as solicitações de renovação é menor que o tempo especificado para a duração da concessão a fim de impedir que outra instância de função seja eleita como o líder. Se a renovação falhar por algum motivo, a tarefa será cancelada.

Se a concessão não puder ser renovada ou a tarefa for cancelada (provavelmente devido ao desligamento da instância de função), a concessão será liberada. Neste ponto, essa ou outra instância de função pode ser eleita como líder. A extração de código abaixo mostra essa parte do processo.

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (...)
    {
      ...
      if (...)
      {
        ...
        using (var leaseCts = ...)
        {
          ...
          // Keep renewing the lease in regular intervals.
          // If the lease can't be renewed, then the task completes.
          var renewLeaseTask =
            this.KeepRenewingLease(leaseManager, leaseId, leaseCts.Token);

          // When any task completes (either the leader task itself or when it
          // couldn't renew the lease) then cancel the other task.
          await CancelAllWhenAnyCompletes(leaderTask, renewLeaseTask, leaseCts);
        }
      }
    }
  }
  ...
}
```

O `KeepRenewingLease` é outro método auxiliar que usa o objeto `BlobLeaseManager` para renovar a concessão. O método `CancelAllWhenAnyCompletes` cancela as tarefas especificadas como os dois primeiros parâmetros. O diagrama a seguir ilustra o uso da classe `BlobDistributedMutex` para eleger o líder e executar uma tarefa que coordena as operações.

![A Figura 1 ilustra as funções da classe BlobDistributedMutex](./_images/leader-election-diagram.png)


O exemplo de código a seguir mostra como usar a classe `BlobDistributedMutex` em uma função de trabalho. Esse código adquire uma concessão em um blob denominado `MyLeaderCoordinatorTask` no contêiner de concessão no armazenamento de desenvolvimento e especifica que o código definido no método `MyLeaderCoordinatorTask` deve ser executado se a instância de função for eleita como líder.

```csharp
var settings = new BlobSettings(CloudStorageAccount.DevelopmentStorageAccount,
  "leases", "MyLeaderCoordinatorTask");
var cts = new CancellationTokenSource();
var mutex = new BlobDistributedMutex(settings, MyLeaderCoordinatorTask);
mutex.RunTaskWhenMutexAcquired(this.cts.Token);
...

// Method that runs if the role instance is elected the leader
private static async Task MyLeaderCoordinatorTask(CancellationToken token)
{
  ...
}
```

Observe os pontos a seguir sobre a solução de exemplo:
- O blob é um ponto único de falha em potencial. Se o serviço Blob ficar indisponível ou estiver inacessível, o líder não poderá renovar a concessão e nenhuma outra instância de função poderá adquiri-la. Nesse caso, nenhuma instância de função será capaz de atuar como o líder. No entanto, o serviço Blob é projetado para ser resiliente, por isso uma falha completa do serviço Blob é considerada extremamente improvável.
- Se a tarefa que está sendo executada pelo líder parar, ele poderá continuar a renovar a concessão, impedindo que qualquer outra instância de função adquira a concessão e assuma a função de líder para coordenar tarefas. No mundo real, a integridade do líder deve ser verificada em intervalos frequentes.
- O processo de eleição é não determinístico. Você não pode fazer suposições sobre qual instância de função obterá a concessão de blob e se tornará o líder.
- O blob usado como o destino da concessão de blob não deve ser usado para nenhuma outra finalidade. Se uma instância de função tentar armazenar dados nesse blob, tais dados não poderão ser acessados, a menos que a instância de função seja o líder e possua a concessão de blob.

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados

As diretrizes a seguir também podem ser relevantes ao implementar esse padrão:
- Esse padrão tem um [aplicativos de exemplo](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election) baixável.
- [Diretrizes de dimensionamento automático](https://msdn.microsoft.com/library/dn589774.aspx). É possível iniciar e parar instâncias dos hosts de tarefa à medida que a carga do aplicativo varia. O dimensionamento automático pode ajudar a manter a taxa de transferência e o desempenho durante horários de pico de processamento.
- [Diretrizes de particionamento de computação](https://msdn.microsoft.com/library/dn589773.aspx). Este guia descreve como alocar tarefas para hosts em um serviço de nuvem de maneira a ajudar a minimizar os custos de execução e manter a escalabilidade, desempenho, disponibilidade e segurança do serviço.
- O [Padrão assíncrono baseado em tarefa](https://msdn.microsoft.com/library/hh873175.aspx).
- Um exemplo que ilustra o [Algoritmo Bully](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html).
- Um exemplo que ilustra o [Algoritmo Ring](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html).
- O artigo [Apache Zookeeper no Microsoft Azure](https://msopentech.com/opentech-projects/apache-zookeeper-on-windows-azure-2/) no site da Microsoft Open Technologies.
- O [Apache Curator](http://curator.apache.org/) é uma biblioteca de cliente para o Apache ZooKeeper.
- O artigo [Blob de Concessão (API REST)](https://msdn.microsoft.com/library/azure/ee691972.aspx) no MSDN.
