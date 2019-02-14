---
title: Padrão de repetição
titleSuffix: Cloud Design Patterns
description: Permita que um aplicativo trate falhas previstas e temporárias quando tentar se conectar a um serviço ou recurso de rede ao repetir de forma transparente uma operação que falhou anteriormente.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 8d37bc2aed17bfef4d54f29f269b23ce4a5c52c0
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897653"
---
# <a name="retry-pattern"></a>Padrão de repetição

[!INCLUDE [header](../_includes/header.md)]

Permita que um aplicativo trate falhas transitórias quando tentar se conectar a um serviço ou recurso de rede ao repetir de forma transparente uma operação com falha. Isso pode melhorar a estabilidade do aplicativo.

## <a name="context-and-problem"></a>Contexto e problema

Um aplicativo que se comunica com os elementos em execução na nuvem precisa ser suscetível às falhas transitórias que podem ocorrer nesse ambiente. As falhas incluem perda momentânea da conectividade de rede com os componentes e serviços, indisponibilidade temporária de um serviço ou tempos limite que surgem quando um serviço está ocupado.

Essas falhas geralmente são autocorretivas e, se a ação que disparou uma falha for repetida após um atraso razoável, é provável que seja bem-sucedida. Por exemplo, um serviço de banco de dados que está processando um grande número de solicitações simultâneas pode implementar uma estratégia de limitação que rejeita temporariamente solicitações adicionais até que sua carga de trabalho diminua. Um aplicativo que tentar acessar o banco de dados pode não conseguir se conectar, mas se ele tentar novamente após um atraso, talvez tenha êxito.

## <a name="solution"></a>Solução

Na nuvem, as falhas transitórias não são incomuns e um aplicativo deve ser criado para lidar com elas com elegância e transparência. Isso minimiza os efeitos que as falhas podem ter sobre as tarefas comerciais que o aplicativo está executando.

Se um aplicativo detectar uma falha ao tentar enviar uma solicitação para um serviço remoto, ele poderá lidar com falhas usando as seguintes estratégias:

- **Cancelar**. Se a falha indica não ser transitória ou provavelmente não será bem-sucedida se for repetida, o aplicativo deve cancelar a operação e relatar uma exceção. Por exemplo, uma falha de autenticação causada pelo fornecimento de credenciais inválidas provavelmente não será bem-sucedida, independentemente de quantas tentativas ocorram.

- **Tentar novamente**. Se a falha específica relatada é incomum ou rara, ela pode ter sido causada por circunstâncias incomuns, como um pacote de rede se tornar corrompido enquanto estava sendo transmitido. Nesse caso, o aplicativo pode repetir a solicitação com falha novamente imediatamente porque é improvável que a mesma falha se repita e a solicitação provavelmente será bem-sucedida.

- **Tentar novamente após atraso**. Se a falha é causada por uma ou mais falhas comuns de conectividade ou ocupado, a rede ou o serviço podem precisar de um breve período enquanto os problemas de conectividade são corrigidos ou a lista de pendências de trabalho é limpa. O aplicativo deve esperar por um tempo adequado antes de tentar executar novamente a solicitação.

Para falhas transitórias mais comuns, o período entre as repetições deve ser escolhido para distribuir solicitações de várias instâncias do aplicativo de maneira mais uniforme possível. Isso reduz a chance de um serviço ocupado continuar a ser sobrecarregado. Se muitas instâncias de um aplicativo sobrecarregam continuamente um serviço com solicitações de novas tentativas, demorará mais para o serviço se recuperar.

Se a solicitação ainda assim falhar, o aplicativo poderá esperar e fazer outra tentativa. Se necessário, esse processo pode ser repetido com atrasos maiores entre as novas tentativas, até que o número máximo de solicitações ser tentado. O atraso pode ser aumentado de forma incremental ou exponencial, dependendo do tipo de falha e da probabilidade de que ela será corrigida durante esse tempo.

O diagrama a seguir ilustra como invocar uma operação em um serviço hospedado usando esse padrão. Se a solicitação for mal-sucedida depois de um número predefinido de tentativas, o aplicativo deverá considerar a falha como uma exceção e tratá-la adequadamente.

![Figura 1 – Invocar uma operação em um serviço hospedado usando o Padrão de repetição](./_images/retry-pattern.png)

O aplicativo deve encapsular todas as tentativas de acessar um serviço remoto no código que implementa uma política de repetição correspondente a uma das estratégias listadas acima. As solicitações enviadas a serviços diferentes podem estar sujeitas a políticas diferentes. Alguns fornecedores disponibilizam bibliotecas que implementam políticas de repetição, nas quais o aplicativo pode especificar o número máximo de repetições, o tempo entre elas e demais parâmetros.

Um aplicativo deve registrar em log os detalhes das falhas e as falhas de operações. Essas informações são úteis para os operadores. Se um serviço fica indisponível ou ocupado com frequência, isso normalmente ocorre porque ele esgotou seus recursos. Você pode reduzir a frequência dessas falhas expandindo o serviço. Por exemplo, se um serviço de banco de dados fica continuamente sobrecarregado, pode ser benéfico particionar o banco de dados e distribuir a carga entre vários servidores.

> O [Microsoft Entity Framework](https://docs.microsoft.com/ef/) fornece recursos para repetir operações de banco de dados. Além disso, a maioria dos serviços do Azure e SDKs do cliente incluem um mecanismo de repetição. Para saber mais, consulte [Diretrizes de repetição para serviços específicos](/azure/architecture/best-practices/retry-service-specific).

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão.

A política de repetição deve ser ajustada para atender aos requisitos de negócios do aplicativo e a natureza da falha. Para algumas operações não críticas, é melhor falhar rapidamente do que repetir várias vezes e afetar a taxa de transferência do aplicativo. Por exemplo, em um aplicativo Web interativo que acessa um serviço remoto, é melhor falhar após um número menor de novas tentativas com um pequeno atraso entre elas e exibir uma mensagem apropriada para o usuário (por exemplo, “tente novamente mais tarde”). Para um aplicativo de lote, pode ser mais apropriado aumentar o número de tentativas de repetição com um atraso exponencialmente maior entre as tentativas.

Uma política de repetição agressiva com atraso mínimo entre as tentativas e um grande número de repetições poderia prejudicar ainda mais um serviço ocupado executado no limite da capacidade ou próximo desse limite. Essa política de repetição também poderá afetar a capacidade de resposta do aplicativo se ele tentar continuamente executar uma operação com falha.

Se uma solicitação ainda falhar após um número significativo de tentativas, é melhor para o aplicativo evitar que outras solicitações sejam enviadas para o mesmo recurso e simplesmente relatar uma falha imediatamente. Quando o período expirar, o aplicativo pode tentar permitir uma ou mais solicitações para ver se elas são bem-sucedidas. Para obter mais detalhes sobre essa estratégia, consulte o [Padrão de disjuntor](./circuit-breaker.md).

Considere se a operação é idempotente. Nesse caso, tentar novamente é inerentemente seguro. Caso contrário, as repetições podem fazer com que a operação seja executada mais de uma vez, com efeitos colaterais imprevistos. Por exemplo, um serviço pode receber a solicitação, processá-la com êxito, mas falha ao enviar uma resposta. Nesse ponto, a lógica de repetição pode reenviar a solicitação, supondo que a primeira solicitação não foi recebida.

Uma solicitação para um serviço pode falhar por várias razões e gerar exceções diferentes dependendo da natureza da falha. Algumas exceções indicam uma falha que pode ser resolvida rapidamente, enquanto outras indicam que a falha é mais duradoura. É útil para a política de repetição ajustar o tempo entre as tentativas de repetição com base no tipo de exceção.

Considere como repetir uma operação que faz parte de uma transação afeta a consistência geral da transação. Ajuste a política de repetição para operações transacionais a fim de maximizar a chance de sucesso e reduzir a necessidade de desfazer todas as etapas da transação.

Verifique se todo o código de repetição foi totalmente testado para uma variedade de condições de falha. Verifique se ele não afeta gravemente o desempenho ou a confiabilidade do aplicativo, causando excesso de carga em serviços e recursos ou gerando condições de corrida ou gargalos.

Implemente a lógica de repetição somente onde o contexto completo de uma operação com falha for compreendido. Por exemplo, se uma tarefa que contém uma política de repetição invocar outra tarefa que também contém uma política de repetição, essa camada extra de repetições pode adicionar atrasos longos para o processamento. Talvez seja melhor configurar a tarefa de nível inferior para falhar rapidamente e relatar o motivo da falha para a tarefa que a invocou. Esta tarefa de nível mais alto pode então tratar a falha com base em sua própria política.

É importante registrar todas as falhas de conectividade que causam uma tentativa para que os problemas subjacentes com o aplicativo, serviços ou recursos que possam ser identificados.

Investigue as falhas que têm mais probabilidade de ocorrer para um serviço ou um recurso para descobrir se elas provavelmente serão de longa duração ou terminal. Se elas forem, é melhor tratar a falha como uma exceção. O aplicativo pode relatar ou registrar a exceção e, em seguida, tentar continuar com a invocação de um serviço alternativo (se houver) ou oferecer funcionalidade degradada. Para obter mais informações sobre como detectar e tratar falhas de longa duração, consulte o [Padrão de disjuntor](./circuit-breaker.md).

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use esse padrão quando um aplicativo poderia apresentar falhas transitórias ao interagir com um serviço remoto ou acessar um recurso remoto. Espera-se que tais falhas sejam breves e repetir uma solicitação que falhou anteriormente pode ter êxito em uma tentativa subsequente.

Esse padrão pode não ser útil:

- Quando uma falha provavelmente é de longa duração, pois isso pode afetar a capacidade de resposta de um aplicativo. O aplicativo pode estar desperdiçando tempo e recursos tentando repetir uma solicitação que provavelmente falhará.
- Para tratar falhas não causadas por falhas transitórias, como exceções internas causadas por erros de lógica de negócios de um aplicativo.
- Como uma alternativa para tratar problemas de escalabilidade em um sistema. Se um aplicativo apresentar falhas frequentes de ocupado, isso geralmente é um sinal de que o serviço ou recurso que está sendo acessado deve ser escalado verticalmente.

## <a name="example"></a>Exemplo

Este exemplo em C# ilustra uma implementação do Padrão de repetição. O método `OperationWithBasicRetryAsync`, mostrado abaixo, invoca um serviço externo assincronamente por meio do método `TransientOperationAsync`. Os detalhes do método `TransientOperationAsync` são específicos ao serviço e serão omitidos do código de exemplo.

```csharp
private int retryCount = 3;
private readonly TimeSpan delay = TimeSpan.FromSeconds(5);

public async Task OperationWithBasicRetryAsync()
{
  int currentRetry = 0;

  for (;;)
  {
    try
    {
      // Call external service.
      await TransientOperationAsync();

      // Return or break.
      break;
    }
    catch (Exception ex)
    {
      Trace.TraceError("Operation Exception");

      currentRetry++;

      // Check if the exception thrown was a transient exception
      // based on the logic in the error detection strategy.
      // Determine whether to retry the operation, as well as how
      // long to wait, based on the retry strategy.
      if (currentRetry > this.retryCount || !IsTransient(ex))
      {
        // If this isn't a transient error or we shouldn't retry,
        // rethrow the exception.
        throw;
      }
    }

    // Wait to retry the operation.
    // Consider calculating an exponential delay here and
    // using a strategy best suited for the operation and fault.
    await Task.Delay(delay);
  }
}

// Async method that wraps a call to a remote service (details not shown).
private async Task TransientOperationAsync()
{
  ...
}
```

A instrução que chama esse método é contida em um bloco try/catch encapsulado em um loop for. O loop for existirá se a chamada para o método `TransientOperationAsync` for bem-sucedida sem gerar uma exceção. Se o método `TransientOperationAsync` falhar, o bloco catch examinará o motivo da falha. Se for considerado um erro transitório, o código esperará por um breve período antes de tentar novamente a operação.

O loop for também controla o número de vezes que a operação foi tentada e, se o código falhar três vezes, a exceção será considerada mais duradoura. Se a exceção não for transitória ou for duradoura, o manipulador catch gerará uma exceção. Essa exceção sai do loop for e deve ser capturada pelo código que invoca o método `OperationWithBasicRetryAsync`.

O método `IsTransient`, mostrado abaixo, verifica para um conjunto específico de exceções que são relevantes para o ambiente no qual o código é executado. A definição de uma exceção transitória varia de acordo com os recursos que estão sendo acessados e o ambiente no qual operação está sendo executada.

```csharp
private bool IsTransient(Exception ex)
{
  // Determine if the exception is transient.
  // In some cases this is as simple as checking the exception type, in other
  // cases it might be necessary to inspect other properties of the exception.
  if (ex is OperationTransientException)
    return true;

  var webException = ex as WebException;
  if (webException != null)
  {
    // If the web exception contains one of the following status values
    // it might be transient.
    return new[] {WebExceptionStatus.ConnectionClosed,
                  WebExceptionStatus.Timeout,
                  WebExceptionStatus.RequestCanceled }.
            Contains(webException.Status);
  }

  // Additional exception checking logic goes here.
  return false;
}
```

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados

- [Padrão de disjuntor](./circuit-breaker.md). O Padrão de repetição é útil para tratar falhas transitórias. Se espera-se que uma falha seja mais duradoura, pode ser mais apropriado implementar o Padrão de disjuntor. O Padrão de repetição também pode ser usado junto com um disjuntor para fornecer uma abordagem abrangente para tratar falhas.
- [Diretrizes de repetição para serviços específicos](/azure/architecture/best-practices/retry-service-specific)
- [Resiliência de Conexão](https://docs.microsoft.com/ef/core/miscellaneous/connection-resiliency)
