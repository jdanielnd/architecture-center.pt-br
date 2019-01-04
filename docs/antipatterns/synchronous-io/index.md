---
title: Antipadrão de E/S síncrona
titleSuffix: Performance antipatterns for cloud apps
description: O bloqueio do thread de chamada durante a conclusão da E/S pode reduzir o desempenho e afetar a escalabilidade vertical.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: 209295cfc911ae168bca2f1c64dc930a27a9a4ba
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009331"
---
# <a name="synchronous-io-antipattern"></a>Antipadrão de E/S síncrona

O bloqueio do thread de chamada durante a conclusão da E/S pode reduzir o desempenho e afetar a escalabilidade vertical.

## <a name="problem-description"></a>Descrição do problema

Uma operação de E/S síncrona bloqueia o thread de chamada, enquanto a E/S é concluída. O thread de chamada entra em um estado de espera e não é possível executar o trabalho útil durante esse intervalo, desperdiçando recursos de processamento.

Exemplos comuns de E/S incluem:

- Recuperar dados persistentes para um banco de dados ou qualquer tipo de armazenamento persistente.
- Enviando uma solicitação para um serviço Web.
- Envio de uma mensagem ou recuperação de mensagens de uma fila.
- Gravar ou ler de um arquivo local.

Esse antipadrão geralmente ocorre porque:

- Ele parece ser a maneira mais fácil para executar uma operação.
- O aplicativo requer uma resposta de uma solicitação.
- O aplicativo usa uma biblioteca que só fornece métodos síncronos de E/S.
- Uma biblioteca externa executa operações de E/S síncrona internamente. Uma única chamada de E/S síncrona pode bloquear uma cadeia de chamada inteira.

O código a seguir carrega um arquivo para o armazenamento de Blobs do Azure. Há dois locais em que os blocos de código aguardam a E/S síncrona, o método `CreateIfNotExists` e o método `UploadFromStream`.

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

Aqui está um exemplo de uma resposta de um serviço externo. O `GetUserProfile` método chama um serviço remoto retorna um `UserProfile`.

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

Você pode encontrar o código completo para ambos os exemplos [aqui][sample-app].

## <a name="how-to-fix-the-problem"></a>Como corrigir o problema

Substitua as operações de E/S síncrona com operações assíncronas. Isso libera o thread atual para continuar executando trabalho significativo em vez de bloquear e ajuda a melhorar a utilização dos recursos de computação. Executar E/S assíncrona é especialmente eficiente para lidar com um surto inesperado em solicitações de aplicativos cliente.

Muitas bibliotecas fornecem versões síncronas e assíncronas dos métodos. Sempre que possível, use as versões assíncronas. Aqui está a versão assíncrona do exemplo anterior que carrega um arquivo para o armazenamento de Blobs do Azure.

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

O operador `await` retorna o controle para o ambiente de chamada enquanto a operação assíncrona é executada. O código depois dessa instrução atua como uma continuação que é executada depois que a operação assíncrona é concluída.

Um serviço bem projetado também deve fornecer a operações assíncronas. Aqui está uma versão assíncrona do serviço Web que retorna os perfis de usuário. O `GetUserProfileAsync` método depende de ter uma versão assíncrona do serviço de perfil de usuário.

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

Para bibliotecas que não fornecem versões assíncronas de operações, pode ser possível criar wrappers assíncronos em torno de métodos síncronos selecionados. Siga essa abordagem com cuidado. Embora possa melhorar a capacidade de resposta no thread que invoca o wrapper assíncrono, ele realmente consome mais recursos. Um thread adicional pode ser criado e não há sobrecarga associada à sincronização do trabalho realizado por esse thread. Alguns desafios são discutidos nessa postagem no blog: [Devo expor wrappers assíncronos a métodos síncronos?][async-wrappers]

Aqui está um exemplo de um wrapper em torno de um método síncrono de assíncrono.

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

Agora, o código de chamada pode aguardar em um wrapper:

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a>Considerações

- As operações de E/S que devem ser muito curta duração e provavelmente não causar contenção podem ser mais funcional como operações síncronas. Um exemplo pode ser a leitura de pequenos arquivos em uma unidade SSD. A sobrecarga de despachar uma tarefa para outro thread e sincronizar com esse thread quando a tarefa for concluída, pode superar os benefícios de E/S assíncrona. No entanto, esses casos são relativamente raros, e a maioria das operações de E/S deve ser feita de forma assíncrona.

- Melhorando o desempenho de E/S pode causar outras partes do sistema para se tornar afunilamentos. Por exemplo, segmentos de desbloqueio podem resultar em um maior volume de solicitações simultâneas para recursos compartilhados, em vez de limitação ou privação de recursos. Se isso se tornar um problema, você precisará expandir o número de servidores Web ou dados de partição de lojas para reduzir a contenção.

## <a name="how-to-detect-the-problem"></a>Como detectar o problema

Para usuários, o aplicativo pode parecer não responder ou parar periodicamente. O aplicativo pode falhar com exceções de tempo limite. Essas falhas também poderá retornar erros HTTP 500 (servidor interno). No servidor, as solicitações de cliente podem estar bloqueadas até que um thread fique disponível, resultando em comprimentos de fila de solicitações excessivas, manifestados como erros HTTP 503 (Serviço Indisponível).

Você pode executar as etapas a seguir para ajudar a identificar o problema:

1. Monitorar o sistema de produção e determinar se bloqueado threads de trabalho são restrições de taxa de transferência.

2. Se solicitações estão sendo bloqueadas devido à falta de threads, examine o aplicativo para determinar quais operações podem ser executar E/S síncrona.

3. Execute testes de carga controlada de cada operação que está executando a E/S síncrona, para descobrir se essas operações estão afetando o desempenho do sistema.

## <a name="example-diagnosis"></a>Diagnóstico de exemplo

As seções a seguir aplicam essas etapas ao aplicativo de exemplo descrito anteriormente.

### <a name="monitor-web-server-performance"></a>Monitorar desempenho de servidor Web

Para funções Web e aplicativos Web do Azure, é importante monitorar o desempenho do servidor Web do IIS. Em particular, preste atenção para o comprimento da fila de solicitação para estabelecer solicitações estão sendo bloqueadas aguardando threads disponíveis durante períodos de alta atividade. Você pode coletar essas informações por habilitar o diagnóstico do Azure. Para obter mais informações, consulte:

- [Monitorar aplicativos no Serviço de Aplicativo do Azure][web-sites-monitor]
- [Criar e usar contadores de desempenho em um aplicativo do Azure][performance-counters]

Instrumentar o aplicativo para ver como as solicitações são tratadas quando eles tiverem aceitado. O rastreamento do fluxo de uma solicitação pode ajudar a identificar se ele está executando chamadas lentas e bloquear o thread atual. Thread de criação de perfil também pode realçar a solicitações que estão sendo bloqueadas.

### <a name="load-test-the-application"></a>Fazer teste de carga no aplicativo

O gráfico a seguir mostra o desempenho do método `GetUserProfile` assíncrono mostrado anteriormente, sob cargas diferentes de até 4.000 usuários simultâneos. O aplicativo é um aplicativo ASP.NET em execução em uma função Web do serviço de nuvem do Azure.

![Gráfico de desempenho para o aplicativo de exemplo executar operações de E/S síncrona][sync-performance]

A operação síncrona é codificada no modo de suspensão por dois segundos, para simular a E/S síncrona, portanto, o tempo de resposta mínimo é um pouco mais de dois segundos. Quando a carga atingir aproximadamente 2500 usuários simultâneos, o tempo médio de resposta atinge um limite, embora o volume de solicitações por segundo continua a aumentar. Observe que a escala para essas duas medidas é logarítmica. O número de solicitações por segundo duplicatas entre esse ponto e o fim do teste.

Isoladamente, não é necessariamente claro neste teste se a E/S síncrona é um problema. Sob carga mais pesada, o aplicativo pode alcançar o ápice onde o servidor Web não pode processar solicitações de maneira oportuna, fazendo com que os aplicativos cliente recebam exceções de tempo limite.

As solicitações de entrada são colocadas em fila pelo servidor Web do IIS e passadas para um thread em execução no pool de threads do ASP.NET. Como cada operação executa a E/S de forma síncrona, o thread é bloqueado até que a operação seja concluída. À medida que aumenta a carga de trabalho, eventualmente todos os threads ASP.NET no pool de threads são alocados e bloqueados. Nesse ponto, quaisquer solicitações de entrada adicional devem esperar na fila por um segmento disponível. À medida que aumenta o tamanho da fila, as solicitações começam a atingir o tempo limite.

### <a name="implement-the-solution-and-verify-the-result"></a>Implementar a solução e verificar o resultado

O seguinte gráfico mostra os resultados da versão assíncrona do código de teste de carga.

![Gráfico de desempenho para o aplicativo de exemplo executar operações de E/S assíncronas][async-performance]

A taxa de transferência é muito maior. Sobre a mesma duração como o teste anterior, o sistema manipula com êxito um aumento de dez vezes quase na taxa de transferência, medida em solicitações por segundo. Além disso, o tempo médio de resposta é relativamente constante e permanece aproximadamente 25 vezes menor do que o teste anterior.

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO
[async-wrappers]: https://blogs.msdn.microsoft.com/pfxteam/2012/03/24/should-i-expose-asynchronous-wrappers-for-synchronous-methods/
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg
