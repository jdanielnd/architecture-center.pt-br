---
title: Consolidação de Recursos de Computação
description: Consolidar várias tarefas ou operações em uma única unidade de computação
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
ms.openlocfilehash: 6e05a30245fbf5183a4e50a54650505f5a5f2aa8
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252917"
---
# <a name="compute-resource-consolidation-pattern"></a>Padrão de consolidação de recursos de computação

[!INCLUDE [header](../_includes/header.md)]

Consolidar várias tarefas ou operações em uma única unidade de computação. Isso pode aumentar a utilização de recursos de computação, reduzir os custos e as despesas gerais de gerenciamento associados à execução de processamento de computação em aplicativos hospedados na nuvem.

## <a name="context-and-problem"></a>Contexto e problema

Um aplicativo de nuvem geralmente implementa uma variedade de operações. Em algumas soluções, faz sentido seguir o princípio do projeto de separação de preocupações inicialmente, e dividir essas operações em unidades computacionais separadas que são hospedadas e implantadas individualmente (por exemplo, como aplicativos Web do Serviço de Aplicativo separados, Máquinas Virtuais separadas ou funções de Serviço de Nuvem separadas). No entanto, embora essa estratégia possa ajudar a simplificar o design lógico da solução, a implantação de um grande número de unidades computacionais como parte do mesmo aplicativo pode aumentar os custos de hospedagem de tempo de execução e tornar o gerenciamento do sistema mais complexo.

Como um exemplo, a figura mostra a estrutura simplificada de uma solução hospedada na nuvem que é implementada utilizando mais de uma unidade computacional. Cada unidade computacional é executada em seu próprio ambiente virtual. Cada função foi implementada como uma tarefa separada (rotulada Tarefa A através da Tarefa E) em execução em sua própria unidade computacional.

![Tarefas em execução em um ambiente de nuvem utilizando um conjunto de unidades computacionais dedicadas](./_images/compute-resource-consolidation-diagram.png)


Cada unidade computacional consome recursos passíveis de cobrança, mesmo que seja ocioso ou de uso leve. Portanto, essa não é sempre a solução mais econômica.

No Azure, essa preocupação se aplica a funções em um Serviço de Nuvem, Serviços de Aplicativos e Máquinas Virtuais. Esses itens são executados em seu próprio ambiente virtual. A execução de uma coleção de funções separadas, sites ou máquinas virtuais que são projetados para executar um conjunto de operações bem definidas, mas que precisam se comunicar e cooperar como parte de uma única solução, podem ser uma utilização de recursos ineficiente.

## <a name="solution"></a>Solução

Para ajudar a reduzir custos, aumentar a utilização, melhorar a velocidade de comunicação e reduzir o gerenciamento é possível consolidar várias tarefas ou operações em uma única unidade computacional.

As tarefas podem ser agrupadas de acordo com os critérios baseados nos recursos fornecidos pelo ambiente e os custos associados a esses recursos. Uma abordagem comum é procurar tarefas que tenham um perfil semelhante em relação aos requisitos de processamento, tempo de vida e escalabilidade. Agrupar esses juntos permite que eles escalem como uma unidade. A elasticidade fornecida por muitos ambientes de nuvem permite que instâncias adicionais de uma unidade computacional sejam iniciadas e encerradas de acordo com a carga de trabalho. Por exemplo, o Azure fornece dimensionamento automático que pode ser aplicador a funções em um Serviço de Nuvem, Serviços de Aplicativos e Máquinas Virtuais. Para obter mais informações, consulte [Diretrizes de dimensionamento automático](https://msdn.microsoft.com/library/dn589774.aspx).

Como um exemplo de contador para mostrar como a escalabilidade pode ser utilizada para determinar quais operações não devem ser agrupadas, considere as duas tarefas a seguir:

- A Tarefa 1 pesquisa mensagens infrequentes e indiferentes ao tempo enviadas para uma fila.
- A Tarefa 2 trata disparos de alto volume de tráfego de rede.

A segunda tarefa requer elasticidade que pode envolver iniciar e parar um grande número de instâncias da unidade computacional. A aplicação da mesma escala para a primeira tarefa resultaria em mais tarefas escutando mensagens infrequentes na mesma fila e um desperdício de recursos.

Em muitos ambientes de nuvem, é possível especificar os recursos disponíveis para uma unidade computacional em termos do número de núcleos de CPU, memória, espaço em disco e, assim por diante. Geralmente, quanto mais recursos forem especificados, maior será o custo. Para economizar dinheiro, é importante maximizar o trabalho que uma unidade computacional de custo elevado desempenha e não deixá-lo ficar inativo durante um período prolongado.

Se houver tarefas que exigem uma grande quantidade de energia da CPU em disparos curtos, considere consolidá-las em uma única unidade computacional que forneça a energia necessária. No entanto, é importante equilibrar essa necessidade para manter os recursos caros ocupados em relação à contenção que pode ocorrer se eles estiverem com carga excessiva. As tarefas de uso intensivo e computacional de longa duração não devem compartilhar a mesma unidade computacional, por exemplo.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao implementar esse padrão:

**Escalabilidade e elasticidade**. Muitas soluções na nuvem implementam escalabilidade e elasticidade ao nível da unidade computacional iniciando e encerrando instâncias de unidades. Evite agrupar tarefas que tenham requisitos de escalabilidade conflitantes na mesma unidade computacional.

**Tempo de vida**. A infraestrutura de nuvem recicla periodicamente o ambiente virtual que hospeda uma unidade computacional. Quando houver muitas tarefas de execução longa em uma unidade computacional, poderá ser necessário configurar a unidade para evitar que seja reciclada até que essas tarefas tenham terminado. Alternativamente, crie as tarefas utilizando uma abordagem de verificação de pontos que permitirá parar corretamente e continuar no ponto em que foram interrompidas quando a unidade computacional for reiniciada.

**Cadência de lançamento**. Se a implementação ou configuração de uma tarefa for alterada com frequência, poderá ser necessário parar a unidade computacional hospedando o código atualizado, reconfigurar e reimplantar a unidade e reiniciá-la. Esse processo também exigirá que todas as outras tarefas dentro da mesma unidade computacional sejam encerradas, redistribuídas e reiniciadas.

**Segurança**. Tarefas na mesma unidade computacional podem compartilhar o mesmo contexto de segurança e acessar os mesmos recursos. Deve haver um alto grau de confiança entre as tarefas e a confiança de que uma tarefa não irá corromper ou prejudicar outra. Além disso, aumentar o número de tarefas em execução em uma unidade computacional aumentará a superfície de ataque da unidade. Cada tarefa é tão segura quanto a que possui mais vulnerabilidades.

**Tolerância a falhas**. Se uma tarefa em uma unidade computacional falhar ou se comportar de maneira anormal, isso poderá afetar as outras tarefas em execução na mesma unidade. Por exemplo, se uma tarefa não for iniciada corretamente, isso poderá provocar falha em toda a lógica de inicialização na unidade computacional e evitar que outras tarefas na mesma unidade sejam executadas.

**Contenção**. Evite introduzir contenção entre tarefas que competem por recursos na mesma unidade computacional. Idealmente, tarefas que compartilham a mesma unidade computacional devem exibir características de utilização de recursos diferentes. Por exemplo, duas tarefas intensivas em computação provavelmente não devem residir na mesma unidade computacional e, tampouco, duas tarefas que consomem grandes quantidades de memória. No entanto, combinar uma tarefa intensiva de computação com uma tarefa que exige uma grande quantidade de memória é uma combinação viável.

> [!NOTE]
>  Considere consolidar recursos de computação apenas para um sistema que esteja em produção por um período de tempo, de modo que os operadores e desenvolvedores possam monitorar o sistema e criar um _mapa de calor_ que identifica como cada tarefa utiliza diferentes recursos. Este mapa pode ser utilizado para determinar quais tarefas são boas candidatas a compartilhar recursos de computação.

**Complexidade**. Combinar várias tarefas em uma única unidade computacional adiciona complexidade ao código na unidade, possivelmente tornando mais difícil testar, depurar e manter.

**Arquitetura lógica estável**. Desenhe e implemente o código em cada tarefa para que não precise alterar, mesmo que o ambiente físico em que a tarefa executa seja alterado.

**Outras estratégias**. A consolidação de recursos de computação é apenas uma maneira de ajudar a reduzir custos associados à execução de várias tarefas simultaneamente. Isso requer um planejamento cuidadoso e monitoramento para garantir que continue sendo uma abordagem efetiva. Outras estratégias podem ser mais apropriadas, dependendo da natureza do trabalho e onde os usuários dessas tarefas em execução estão localizados. Por exemplo, a decomposição funcional da carga de trabalho (conforme descrito pela [Diretrizes de particionamento de computação](https://msdn.microsoft.com/library/dn589773.aspx)) pode ser uma opção melhor.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Utilize esse padrão para tarefas que não são eficazes em termos de custos, caso executem em suas próprias unidades computacionais. Se uma tarefa gasta muito do seu tempo ocioso, executar essa tarefa em uma unidade dedicada poderá ser caro.

Esse padrão pode não ser adequado para tarefas que realizam operações críticas de tolerância a falhas ou tarefas que processam dados altamente sensíveis ou privados e exigem seu próprio contexto de segurança. Essas tarefas devem ser executadas em seu próprio ambiente isolado, em uma unidade computacional separada.

## <a name="example"></a>Exemplo

Ao compilar um serviço de nuvem no Azure, é possível consolidar o processamento realizado por várias tarefas em uma única função. Normalmente, essa é uma função de trabalho que executa tarefas de processamento assíncrono ou em segundo plano.

> Em alguns casos, é possível incluir tarefas de processamento assíncrono ou em seguindo plano na função web. Essa técnica ajuda a reduzir custos e simplificar a implantação, embora possa impactar a capacidade de resposta e escalabilidade da interface voltada ao público fornecida pela função web. O artigo [Combinando várias funções de trabalho do Azure em uma função web do Microsoft Azure ](http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html) contém uma descrição detalhada da implementação de tarefas de processamento assíncrono ou em segundo plano em uma função web.

A função responsável por iniciar e interromper as tarefas. Quando o controlador de malha do Azure carrega uma função, isso eleva o evento `Start` para a função. Você pode substituir o `OnStart` método da classe`WebRole` ou `WorkerRole` para lidar com esse evento, talvez para inicializar os dados e outros recursos que as tarefas desse método dependem.

Quando o método `OnStart` for concluído, a função poderá começar a responder às solicitações. É possível localizar mais informações e diretrizes sobre como usar os métodos `OnStart` e `Run` em uma função na seção [Processos de inicialização do aplicativo](https://msdn.microsoft.com/library/ff803371.aspx#sec16) nas diretrizes [Mover aplicativos para a nuvem](https://msdn.microsoft.com/library/ff728592.aspx).

> Mantenha o código no método `OnStart` o mais conciso possível. O Azure não impõe qualquer limite ao tempo necessário para que esse método seja concluído, mas a função não poderá iniciar a responder às solicitações de rede enviadas até que esse método seja concluído.

Quando o método `OnStart` for concluído, a função executará o método `Run`. Nesse ponto, o controlador de malha poderá iniciar o envio de solicitações para a função.

Coloque o código que realmente cria as tarefas no método `Run`. Observe que o método `Run` define o tempo de vida da instância de função. Quando esse método for concluído, o controlador de malha providenciará para que a função seja desligada.

Quando uma função é desligada ou é reciclada, o controlador de malha impedirá que mais solicitações recebidas sejam recebidas do balanceador de carga e eleve o evento `Stop`. É possível capturar esse evento substituindo o método `OnStop` da função e executar qualquer organização necessária antes que a função seja encerrada.

> Qualquer ação executada no método `OnStop` deverá ser concluída em cinco minutos (ou 30 segundos, se você estiver utilizando o emulador do Microsoft Azure em um computador local). Caso contrário, o controlador de malha do Azure assumirá que a função foi paralisada e irá forçá-la a interromper.

As tarefas são iniciadas pelo método `Run`, que aguarda a conclusão das tarefas. As tarefas implementam a lógica de negócios do serviço de nuvem e podem responder às mensagens postadas na função através do Azure Load Balancer. A figura mostra o ciclo de vida de tarefas e recursos em uma função em um serviço de nuvem do Azure.

![O ciclo de vida de tarefas e recursos em uma função em um serviço de nuvem do Azure](./_images/compute-resource-consolidation-lifecycle.png)


O arquivo _WorkerRole.cs_ no projeto _ComputeResourceConsolidation.Worker_ mostra um exemplo de como você pode implementar esse padrão em um serviço de nuvem do Azure.

> O projeto _ComputeResourceConsolidation.Worker_ é parte da solução _ComputeResourceConsolidation_ disponível para fazer o download a partir do [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).

Os métodos `MyWorkerTask1` e `MyWorkerTask2` ilustram como executar diferentes tarefas dentro da mesma função de trabalho. O código a seguir mostra `MyWorkerTask1`. Essa é uma tarefa simples que é suspensa por 30 segundos e, em seguida, emite uma mensagem de rastreamento. Ele repete esse processo até a tarefa ser cancelada. O código em `MyWorkerTask2` é semelhante.

```csharp
// A sample worker role task.
private static async Task MyWorkerTask1(CancellationToken ct)
{
  // Fixed interval to wake up and check for work and/or do work.
  var interval = TimeSpan.FromSeconds(30);

  try
  {
    while (!ct.IsCancellationRequested)
    {
      // Wake up and do some background processing if not canceled.
      // TASK PROCESSING CODE HERE
      Trace.TraceInformation("Doing Worker Task 1 Work");

      // Go back to sleep for a period of time unless asked to cancel.
      // Task.Delay will throw an OperationCanceledException when canceled.
      await Task.Delay(interval, ct);
    }
  }
  catch (OperationCanceledException)
  {
    // Expect this exception to be thrown in normal circumstances or check
    // the cancellation token. If the role instances are shutting down, a
    // cancellation request will be signaled.
    Trace.TraceInformation("Stopping service, cancellation requested");

    // Rethrow the exception.
    throw;
  }
}
```

> O código de exemplo mostra uma implementação comum de um processo em segundo plano. Em um aplicativo do mundo real, é possível seguir essa mesma estrutura, exceto que você deverá colocar sua própria lógica de processamento no corpo do loop que aguarda a solicitação de cancelamento.

Depois que a função de trabalho tiver inicializado os recursos que utiliza, o método `Run` iniciará as duas tarefas simultaneamente, conforme mostrado aqui.

```csharp
/// <summary>
/// The cancellation token source use to cooperatively cancel running tasks
/// </summary>
private readonly CancellationTokenSource cts = new CancellationTokenSource();

/// <summary>
/// List of running tasks on the role instance
/// </summary>
private readonly List<Task> tasks = new List<Task>();

// RoleEntry Run() is called after OnStart().
// Returning from Run() will cause a role instance to recycle.
public override void Run()
{
  // Start worker tasks and add to the task list
  tasks.Add(MyWorkerTask1(cts.Token));
  tasks.Add(MyWorkerTask2(cts.Token));

  foreach (var worker in this.workerTasks)
  {
      this.tasks.Add(worker);
  }

  Trace.TraceInformation("Worker host tasks started");
  // The assumption is that all tasks should remain running and not return,
  // similar to role entry Run() behavior.
  try
  {
    Task.WaitAll(tasks.ToArray());
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then re-throw the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }

  // If there wasn't a cancellation request, stop all tasks and return from Run()
  // An alternative to canceling and returning when a task exits would be to
  // restart the task.
  if (!cts.IsCancellationRequested)
  {
    Trace.TraceInformation("Task returned without cancellation request");
    Stop(TimeSpan.FromMinutes(5));
  }
}
...
```

Neste exemplo, o método `Run` aguarda que as tarefas sejam concluídas. Se uma tarefa for cancelada, o método `Run` assumirá que a função está sendo desligada e aguardará até que as tarefas restantes sejam canceladas antes de concluir (aguarda um máximo de cinco minutos antes de encerrar). Se uma tarefa falhar devido a uma exceção esperada, o método `Run` cancela a tarefa.

> Você poderia implementar estratégias mais abrangentes de monitoramento e tratamento de exceções no método `Run`, como reiniciar tarefas que falharam ou incluir código que permite que a função interrompa e inicie tarefas individuais.

O método `Stop` mostrado no seguinte código é chamado quando o controlador de malha desliga a instância de função (é invocado a partir do método `OnStop`). O código interrompe cada tarefa normalmente, cancelando-a. Se qualquer tarefa demorar mais de cinco minutos para concluir, o processamento de cancelamento no método `Stop` deixa de aguardar e a função é encerrada.

```csharp
// Stop running tasks and wait for tasks to complete before returning
// unless the timeout expires.
private void Stop(TimeSpan timeout)
{
  Trace.TraceInformation("Stop called. Canceling tasks.");
  // Cancel running tasks.
  cts.Cancel();

  Trace.TraceInformation("Waiting for canceled tasks to finish and return");

  // Wait for all the tasks to complete before returning. Note that the
  // emulator currently allows 30 seconds and Azure allows five
  // minutes for processing to complete.
  try
  {
    Task.WaitAll(tasks.ToArray(), timeout);
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then rethrow the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }
}
```

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados

Os padrões e diretrizes a seguir também podem ser relevantes ao implementar esse padrão:

- [Diretrizes de dimensionamento automático](https://msdn.microsoft.com/library/dn589774.aspx). O dimensionamento automático pode ser utilizado para iniciar e interromper instâncias de recursos computacionais de host de serviço, dependendo da demanda antecipada para processamento.

- [Diretrizes de particionamento de computação](https://msdn.microsoft.com/library/dn589773.aspx). Descreve como alocar os serviços e componentes em um serviço de nuvem de forma a ajudar a minimizar os custos de execução, mantendo a escalabilidade, o desempenho, a disponibilidade e a segurança do serviço.

- Esse padrão inclui um [aplicativo de exemplo](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation) baixável.
