---
title: Padrão de consumidores concorrentes
titleSuffix: Cloud Design Patterns
description: Habilite vários consumidores simultâneos para processar as mensagens recebidas no mesmo canal de mensagens.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 77459ff42422969acdc83e66535197547d555de1
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112100"
---
# <a name="competing-consumers-pattern"></a>Padrão de consumidores concorrentes

[!INCLUDE [header](../_includes/header.md)]

Habilite vários consumidores simultâneos para processar as mensagens recebidas no mesmo canal de mensagens. Isso permite que um sistema processe várias mensagens simultaneamente para otimizar a taxa de transferência, melhorar a escalabilidade e a disponibilidade e equilibrar a carga de trabalho.

## <a name="context-and-problem"></a>Contexto e problema

É esperado que um aplicativo em execução na nuvem lide com um grande número de solicitações. Em vez de processar cada solicitação de forma síncrona, uma técnica comum é o aplicativo passá-las por meio de um sistema de mensagens para outro serviço (um serviço consumidor) que as trata de forma assíncrona. Essa estratégia ajuda a garantir que a lógica de negócios do aplicativo não seja bloqueada enquanto as solicitações estão sendo processadas.

O número de solicitações pode variar significativamente com o passar do tempo por vários motivos. Um aumento repentino de atividade do usuário ou solicitações agregadas provenientes de vários locatários pode causar uma carga de trabalho imprevisível. Nos horário de pico de um sistema, pode ser necessário processar centenas de solicitações por segundo, enquanto em outros momentos o número pode ser muito pequeno. Além disso, a natureza do trabalho executado para lidar com essas solicitações pode ser altamente variável. Usar uma única instância do serviço consumidor pode fazer com que ela seja inundada por solicitações ou o sistema de mensagens pode ficar sobrecarregado por um fluxo de mensagens recebidas do aplicativo. Para lidar com essa carga de trabalho flutuante, o sistema pode executar várias instâncias do serviço do consumidor. No entanto, esses consumidores devem ser coordenados para garantir que cada mensagem seja entregue somente para um único consumidor. A carga de trabalho também precisa ser balanceada entre os consumidores para impedir que uma instância se torne um gargalo.

## <a name="solution"></a>Solução

Use uma fila de mensagens para implementar o canal de comunicação entre o aplicativo e as instâncias do serviço do consumidor. O aplicativo posta solicitações na forma de mensagens na fila e as instâncias de serviço do consumidor recebem mensagens da fila e as processa. Essa abordagem permite que o mesmo pool de instâncias de serviço do consumidor lide com mensagens de qualquer instância do aplicativo. A figura ilustra o uso de uma fila de mensagens para distribuir o trabalho para instâncias de um serviço.

![Usar uma fila de mensagens para distribuir o trabalho para instâncias de um serviço](./_images/competing-consumers-diagram.png)

Essa solução oferece as seguintes vantagens:

- Ele fornece um sistema para redistribuir a carga que pode lidar com grandes variações no volume de solicitações enviadas por instâncias de aplicativo. A fila atua como um buffer entre as instâncias do aplicativo e as instâncias de serviço do consumidor. Isso pode ajudar a minimizar o impacto sobre a disponibilidade e a capacidade de resposta tanto do aplicativo quanto das instâncias de serviço, conforme descrito pelo [Padrão de nivelamento de carga baseado em fila](./queue-based-load-leveling.md). Lidar com uma mensagem que requer que processamento de longa execução não impede que outras mensagens sejam tratadas simultaneamente por outras instâncias do serviço do consumidor.

- Isso aumenta a confiabilidade. Se um produtor se comunica diretamente com um consumidor em vez de usar esse padrão, mas não monitora o consumidor, há uma grande probabilidade de que as mensagens poderiam ser perdidas ou não conseguirem ser processadas se o consumidor falhar. Nesse padrão, as mensagens não são enviadas para uma instância de serviço específica. Uma instância de serviço com falha não bloqueará um produtor e as mensagens podem ser processadas por qualquer instância de serviço do trabalho.

- Ela não requer coordenação complexa entre os consumidores ou entre as instâncias do produtor e do consumidor. A fila de mensagens garante que cada mensagem seja entregue pelo menos uma vez.

- É escalonável. O sistema pode aumentar ou diminuir o número dinamicamente o número de instâncias do serviço de consumidor conforme o volume de mensagens flutua.

- Isso poderá melhorar a resiliência se a fila de mensagens fornecer operações de leitura transacionais. Se uma instância de serviço do consumidor lê e processa a mensagem como parte de uma operação transacional e a instância de serviço do consumidor falhar, esse padrão pode garantir que a mensagem seja retornada para a fila para ser coletada e tratada por outra instância e serviço do consumidor.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

- **Ordenação de mensagem**. A ordem na qual as instâncias de serviço de consumidor recebem as mensagens não é garantida e não reflete necessariamente a ordem na qual as mensagens foram criadas. Projete o sistema para garantir que o processamento de mensagens seja idempotente, pois isso ajudará a eliminar qualquer dependência em relação à ordem na qual as mensagens são manipuladas. Para obter mais informações, consulte [Padrões de idempotência](https://blog.jonathanoliver.com/idempotency-patterns/) no blog de Jonathon Oliver.

    > As Filas do Barramento de Serviço do Microsoft Azure podem implementar ordenação primeiro a entrar, primeiro a sair garantida de mensagens usando sessões de mensagens. Para obter mais informações, consulte [Padrões de mensagens usando sessões](https://msdn.microsoft.com/magazine/jj863132.aspx).

- **Projetar serviços para resiliência**. Se o sistema foi projetado para detectar e reiniciar as instâncias de serviço com falha, pode ser necessário implementar o processamento executado pelas instâncias do serviço como operações idempotentes para minimizar os efeitos de uma única mensagem sendo recuperada e processada mais de uma vez.

- **Detecção de mensagens suspeitas**. Uma mensagem malformada ou uma tarefa que exige acesso a recursos que não estão disponíveis, pode causar uma falha em uma instância de serviço. O sistema deve impedir que essas mensagens sejam retornadas para a fila, capturando-as em vez disso e armazenando os detalhes dessas mensagens em outro local para que possam ser analisados, se necessário.

- **Lidar com resultados**. A instância de serviço que lidar com uma mensagem é totalmente separada da lógica do aplicativo que gera a mensagem e pode não ser capaz de se comunicar diretamente. Se a instância de serviço gerar resultados que devem ser passados para a lógica do aplicativo, essas informações devem ser armazenadas em um local que seja acessível para ambos. Para impedir que a lógica do aplicativo recupere dados incompletos, o sistema deve indicar quando o processamento é concluído.

     > Se você estiver usando o Azure, um processo de trabalho pode retornar resultados para a lógica do aplicativo por meio de uma fila de resposta de mensagem dedicada. A lógica do aplicativo deve ser capaz de correlacionar esses resultados com a mensagem original. Esse cenário é descrito em mais detalhes no [Primer de mensagens assíncronas](https://msdn.microsoft.com/library/dn589781.aspx).

- **Dimensionando o sistema de mensagens**. Em uma solução de grande escala, uma única fila de mensagens poderia ficar sobrecarregada com o número de mensagens e se tornar um gargalo no sistema. Nessa situação, considere particionar o sistema de mensagens para enviar as mensagens de produtores específicos para determinada fila ou usar o balanceamento de carga para distribuir mensagens em várias filas de mensagens.

- **Assegurar a confiabilidade do sistema de mensagens**. Um sistema de mensagens confiável é necessário para garantir uma mensagem não seja perdida após o aplicativo enfileirá-la. Isso é essencial para assegurar que todas as mensagens sejam entregues pelo menos uma vez.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use esse padrão quando:

- A carga de trabalho de um aplicativo é dividida em tarefas que podem ser executadas de forma assíncrona.
- As tarefas são independentes e podem ser executados em paralelo.
- O volume de trabalho é altamente variável, o que exige uma solução escalonável.
- A solução deve fornecer alta disponibilidade e ser resiliente se o processamento de uma tarefa falhar.

Esse padrão pode não ser útil se:

- Não for fácil separar a carga de trabalho do aplicativo em tarefas discretas ou se há um alto grau de dependência entre as tarefas.
- As tarefas devem ser executadas de forma síncrona e a lógica do aplicativo deve esperar que uma tarefa seja concluída antes de continuar.
- As tarefas devem ser executadas em uma sequência específica.

> Alguns sistemas de mensagens dão suporte a sessões que permitem que um produtor agrupe as mensagens e verifique se elas são manipuladas pelo mesmo consumidor. Esse mecanismo pode ser usado com mensagens priorizadas (se houver suporte para elas) para implementar uma forma de ordenação de mensagens que entrega as mensagens na sequência de um produtor para um único consumidor.

## <a name="example"></a>Exemplo

O Azure fornece filas de armazenamento e filas do Barramento de Serviço que podem atuar como um mecanismo para implementar esse padrão. A lógica do aplicativo pode postar mensagens em uma fila e os consumidores implementados como tarefas em uma ou mais funções podem recuperá-las dessa fila e processá-las. Para garantir a resiliência, uma fila do Barramento de Serviço permite que um consumidor use o modo `PeekLock` quando recuperar uma mensagem da fila. Na verdade, esse modo não remove a mensagem, mas simplesmente a oculta de outros consumidores. O consumidor original poderá excluí-la quando tiver terminado de processá-la. Se o consumidor falhar, o bloqueio de inspeção atingirá o tempo limite e a mensagem se tornará visível novamente, permitindo que outro consumidor possa recuperá-la.

Para ver informações detalhadas sobre como usar filas do Barramento de Serviço do Azure, consulte [Filas, tópicos e assinaturas do Barramento de Serviço](https://msdn.microsoft.com/library/windowsazure/hh367516.aspx).

Para obter mais informações sobre como usar filas do armazenamento do Azure, consulte [Introdução ao Armazenamento de Filas do Azure usando o .NET](/azure/storage/queues/storage-dotnet-how-to-use-queues).

O código a seguir da classe `QueueManager` na solução CompetingConsumers disponível no [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) mostra como criar uma fila usando uma instância `QueueClient` no manipulador de eventos `Start` em uma função Web ou de trabalho.

```csharp
private string queueName = ...;
private string connectionString = ...;
...

public async Task Start()
{
  // Check if the queue already exists.
  var manager = NamespaceManager.CreateFromConnectionString(this.connectionString);
  if (!manager.QueueExists(this.queueName))
  {
    var queueDescription = new QueueDescription(this.queueName);

    // Set the maximum delivery count for messages in the queue. A message
    // is automatically dead-lettered after this number of deliveries. The
    // default value for dead letter count is 10.
    queueDescription.MaxDeliveryCount = 3;

    await manager.CreateQueueAsync(queueDescription);
  }
  ...

  // Create the queue client. By default the PeekLock method is used.
  this.client = QueueClient.CreateFromConnectionString(
    this.connectionString, this.queueName);
}
```

O seguinte snippet de código mostra como um aplicativo pode criar e enviar um lote de mensagens para a fila.

```csharp
public async Task SendMessagesAsync()
{
  // Simulate sending a batch of messages to the queue.
  var messages = new List<BrokeredMessage>();

  for (int i = 0; i < 10; i++)
  {
    var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
    messages.Add(message);
  }
  await this.client.SendBatchAsync(messages);
}
```

O código a seguir mostra como uma instância de serviço do consumidor pode receber mensagens da fila, seguindo uma abordagem controlada por evento. O parâmetro `processMessageTask` para o método `ReceiveMessages` é um delegado que referencia o código a ser executado quando uma mensagem é recebida. Esse código é executado de forma assíncrona.

```csharp
private ManualResetEvent pauseProcessingEvent;
...

public void ReceiveMessages(Func<BrokeredMessage, Task> processMessageTask)
{
  // Set up the options for the message pump.
  var options = new OnMessageOptions();

  // When AutoComplete is disabled it's necessary to manually
  // complete or abandon the messages and handle any errors.
  options.AutoComplete = false;
  options.MaxConcurrentCalls = 10;
  options.ExceptionReceived += this.OptionsOnExceptionReceived;

  // Use of the Service Bus OnMessage message pump.
  // The OnMessage method must be called once, otherwise an exception will occur.
  this.client.OnMessageAsync(
    async (msg) =>
    {
      // Will block the current thread if Stop is called.
      this.pauseProcessingEvent.WaitOne();

      // Execute processing task here.
      await processMessageTask(msg);
    },
    options);
}
...

private void OptionsOnExceptionReceived(object sender,
  ExceptionReceivedEventArgs exceptionReceivedEventArgs)
{
  ...
}
```

Observe que os recursos de dimensionamento automático, como aqueles disponíveis no Azure, podem ser usados para iniciar e parar instâncias de função, à medida que o tamanho da fila varia. Para saber mais, consulte as [Diretrizes de dimensionamento automático](https://msdn.microsoft.com/library/dn589774.aspx). Além disso, não é necessário manter uma correspondência um para um entre as instâncias de função e os processos de trabalho &mdash; uma única instância de função pode implementar vários processos de trabalho. Para obter mais informações, consulte [Padrão de consolidação de recursos de computação](./compute-resource-consolidation.md).

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados

Os padrões e diretrizes a seguir também podem ser relevantes ao implementar esse padrão:

- [Prévia de mensagens assíncronas](https://msdn.microsoft.com/library/dn589781.aspx). Filas de mensagens são um mecanismo de comunicação assíncrona. Se um serviço do consumidor precisa enviar uma resposta para um aplicativo, pode ser necessário implementar alguma forma de mensagens de resposta. O Primer de mensagens assíncronas fornece informações sobre como implementar mensagens de solicitação/resposta usando filas de mensagens.

- [Diretrizes de dimensionamento automático](https://msdn.microsoft.com/library/dn589774.aspx). Pode ser possível iniciar e parar instâncias de um serviço do consumidor, visto que o tamanho da fila na qual os aplicativos postam as mensagens varia. O dimensionamento automático pode ajudar a manter a taxa de transferência durante horários de pico de processamento.

- [Padrão de consolidação de recursos de computação](./compute-resource-consolidation.md). Pode ser possível consolidar várias instâncias de um serviço do consumidor em um único processo para reduzir os custos e a sobrecarga de gerenciamento. O Padrão de consolidação de recursos de computação descreve os benefícios e as vantagens e desvantagens de seguir essa abordagem.

- [Padrão de nivelamento de carga baseado em fila](./queue-based-load-leveling.md). Apresentar uma fila de mensagens pode agregar resiliência ao sistema, permitindo que as instâncias de serviço lidem com volumes variáveis de solicitações de instâncias do aplicativo. A fila de mensagens atua como um buffer que nivela a carga. O Padrão de nivelamento de carga com base em fila descreve esse cenário mais detalhadamente.

- Esse padrão tem um [aplicativo de exemplo](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) associado a ele.
