---
title: Padrão de repositório de configuração externo
titleSuffix: Cloud Design Patterns
description: Mova as informações de configuração para fora do pacote de implantação de aplicativo para um local centralizado.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: fd006437aab934d951d0a0bc947d32878edbf9d8
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58244397"
---
# <a name="external-configuration-store-pattern"></a>Padrão de repositório de configuração externo

[!INCLUDE [header](../_includes/header.md)]

Mova as informações de configuração para fora do pacote de implantação de aplicativo para um local centralizado. Ele pode fornecer oportunidades para facilitar o gerenciamento e controle dos dados de configuração e para compartilhar os dados de configuração nos aplicativos e instâncias do aplicativo.

## <a name="context-and-problem"></a>Contexto e problema

A maioria dos ambientes de tempo de execução do aplicativo inclui informações de configuração que são mantidas nos arquivos implantados com o aplicativo. Em alguns casos, é possível editar esses arquivos para alterar o comportamento do aplicativo depois que ele é implantado. No entanto, as alterações na configuração requerem que o aplicativo seja reimplantado, geralmente resultando em tempo de inatividade inaceitável e outras sobrecargas administrativas.

Os arquivos de configuração locais também limitam a configuração a um único aplicativo, porém, às vezes, pode ser útil compartilhar as definições de configuração entre vários aplicativos. Alguns exemplos são as cadeias de conexão de banco de dados, informações de tema de interface do usuário ou as URLs de filas e o armazenamento usado por um conjunto de aplicativos relacionados.

É difícil para gerenciar as alterações de configurações locais em várias instâncias em execução do aplicativo, especialmente em um cenário hospedado na nuvem. Isso pode resultar em instâncias que usando definições de configuração diferentes, enquanto a atualização está sendo implantada.

Além disso, as atualizações para aplicativos e componentes podem exigir alterações nos esquemas de configuração. Muitos sistemas de configuração não dão suporte a versões diferentes de informações de configuração.

## <a name="solution"></a>Solução

Armazene as informações de configuração no armazenamento externo e forneça uma interface que pode ser usada para ler e atualizar as definições de configuração de forma rápida e eficiente. O tipo de repositório externo depende do ambiente de hospedagem e do tempo de execução do aplicativo. Em um cenário hospedado na nuvem, ele normalmente é um serviço de armazenamento baseado em nuvem, mas pode ser um banco de dados hospedado ou outro sistema.

O repositório de backup escolhido para as informações de configuração deve ter uma interface que forneça acesso consistente e de fácil utilização. Ele deve expor as informações em um formato com tipo e estrutura corretos. A implementação também pode precisar autorizar o acesso dos usuários para proteger os dados de configuração e ser flexível o suficiente para permitir o armazenamento de várias versões de configuração (como desenvolvimento, preparo ou produção, incluindo várias versões de cada uma delas).

> Muitos sistemas de configuração internos leem os dados quando o aplicativo é iniciado e os armazenar em cache na memória para fornecer acesso rápido e minimizar o impacto sobre o desempenho do aplicativo. Dependendo do tipo de repositório de backup usado e da latência dele, pode ser útil implementar um mecanismo de armazenamento em cache no repositório de configuração externo. Para saber mais, consulte [Diretrizes de caching](https://msdn.microsoft.com/library/dn589802.aspx). A figura mostra uma visão geral do Padrão de repositório de configuração externo com um cache local opcional.

![Uma visão geral do Padrão de repositório de configuração externo com um cache local opcional](./_images/external-configuration-store-overview.png)

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

Escolha um repositório de backup que ofereça desempenho aceitável, alta disponibilidade, robustez e que possa passar por backup como parte do processo de administração e manutenção do aplicativo. Em um aplicativo hospedado na nuvem, geralmente é uma boa opção usar um mecanismo de armazenamento para atender a esses requisitos.

Projete o esquema do repositório de backup para permitir flexibilidade quanto aos tipos de informações que ele pode conter. Verifique se ele fornece todos os requisitos de configuração, como dados com tipo, coleções de configurações, várias versões de configurações e outros recursos exigidos pelos aplicativos que os utilizam. Deve ser fácil estender o esquema para dar suporte a configurações adicionais à medida que os requisitos mudam.

Considere as capacidades físicas do repositório de backup, como ela se relaciona com a forma que as informações de configuração são armazenadas e os efeitos sobre o desempenho. Por exemplo, armazenar um documento XML que contém informações de configuração exigirá que a interface de configuração ou o aplicativo analise o documento para ler as configurações individuais. Será mais complicado atualizar uma configuração, porém armazená-las em cache pode ajudar a compensar o desempenho de leitura mais lento.

Considere como a interface de configuração permitirá o controle do escopo e a herança de definições de configuração. Por exemplo, isso pode ser um requisito para definições de configuração de escopo no nível da organização, do aplicativo ou do computador. Pode ser necessário dar suporte à delegação do controle de acesso a diferentes escopos e impedir ou permitir que aplicativos individuais substituam as configurações.

Verifique se a interface de configuração pode expor os dados de configuração nos formatos exigidos como valores com tipo, coleções, pares de chave/valor ou recipientes de propriedades.

Considere como a interface do repositório de configuração se comportará quando as configurações contiverem erros ou se não existirem no repositório de backup. Pode ser apropriado retornar as configurações padrão e os log de erros. Considere também aspectos como a diferenciação de maiúsculas e minúsculas nas chaves ou nomes das definições de configuração, o armazenamento e manipulação de dados binários e as maneiras como valores nulos ou vazios são manipulados.

Considere como proteger os dados de configuração para permitir o acesso somente aos usuários e aplicativos apropriados. Esse provavelmente é um recurso da interface do repositório de configuração, mas também é necessário garantir que os dados no repositório de backup não possam ser acessados diretamente sem a devida permissão. Verifique se há separação estrita entre as permissões necessárias para ler e gravar os dados de configuração. Considere também se você precisa criptografar algumas ou todas as definições de configuração e como isso será implementado na interface do repositório de configuração.

Configurações armazenadas centralmente, que alteram o comportamento do aplicativo durante o tempo de execução, são cruciais e devem ser implantados, atualizados e gerenciados usando os mesmos mecanismos da implantação de código do aplicativo. Por exemplo, as alterações que podem afetar a mais de um aplicativo precisam ser realizadas usando um teste completo e a abordagem de implantação de teste para garantir que a alteração seja apropriada para todos os aplicativos que usam essa configuração. Se um administrador editar uma configuração para atualizar um aplicativo, ele poderia afetar negativamente outros aplicativos que usam a mesma configuração.

Se um aplicativo armazenar as informações de configuração em cache, ele precisará ser alertado se as configurações forem alteradas. Pode ser possível implementar uma política de expiração de dados de configuração armazenados em cache para que essas informações sejam atualizadas automaticamente e periodicamente, e que as alterações sejam selecionadas (e as devidas ações sejam tomadas).

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Esse padrão é útil para:

- As definições de configuração compartilhadas entre vários aplicativos e instâncias do aplicativo ou nas quais uma configuração padrão precisa ser imposta em vários aplicativos e instâncias do aplicativo.

- Um sistema de configuração padrão que não dá suporte a todas as definições de configuração necessárias, como armazenamento de imagens ou tipos de dados complexos.

- Como um repositório complementar para algumas das configurações de aplicativos, talvez permitindo que os aplicativos substituam algumas ou todas as configurações armazenadas centralmente.

- Como uma maneira de simplificar a administração de vários aplicativos e, opcionalmente, para monitorar o uso de definições de configuração por meio de registro em log de todos os tipos de acesso para o repositório de configuração.

## <a name="example"></a>Exemplo

Em um aplicativo hospedado do Microsoft Azure, uma opção típica para armazenar informações de configuração externamente é usar o Armazenamento do Azure. Ele é flexível, oferece alto desempenho e é replicado três vezes com failover automático para proporcionar alta disponibilidade. O Armazenamento de Tabelas do Azure fornece um repositório de chave/valor com a capacidade de usar um esquema flexível para os valores. O Armazenamento de Blobs do Azure fornece um repositório hierárquico e baseado em contêiner que pode conter qualquer tipo de dados em blobs nomeados individualmente.

O exemplo a seguir mostra como um repositório de configuração pode ser implementado no Armazenamento de Blobs para armazenar e expor as informações de configuração. A classe `BlobSettingsStore` abstrai o Armazenamento de Blobs para manter informações de configuração e implementa a interface `ISettingsStore` mostrada no código a seguir.

> Esse código é fornecido no projeto _ExternalConfigurationStore.Cloud_ na solução _ExternalConfigurationStore_, disponível no [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

Essa interface define os métodos para recuperar e atualizar as definições de configuração mantidas no repositório de configuração e inclui um número de versão que pode ser usado para detectar se as definições de configuração foram modificadas recentemente. A classe `BlobSettingsStore` usa a propriedade `ETag` do blob para implementar o controle de versão. A propriedade `ETag` é atualizada automaticamente sempre que há uma gravação no blob.

> Por padrão, essa solução simples expõe todas as definições de configuração como valores de cadeia de caracteres em vez de valores com tipo.

A classe `ExternalConfigurationManager` fornece um wrapper em torno de um objeto `BlobSettingsStore`. Um aplicativo pode usar essa classe para armazenar e recuperar informações de configuração. Essa classe usa a biblioteca Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) para expor as alterações feitas na configuração por meio de uma implementação da interface `IObservable`. Se uma configuração for modificada chamando o método `SetAppSetting`, o evento `Changed` é acionado e todos os assinantes desse evento serão notificados.

Observe que todas as configurações também são armazenadas em cache em um objeto `Dictionary` dentro da classe `ExternalConfigurationManager` para acesso rápido. O método `GetSetting` usado para recuperar uma definição de configuração lê os dados do cache. Se a configuração não for encontrada no cache, ela será recuperada do objeto `BlobSettingsStore`.

O método `GetSettings` invoca o método `CheckForConfigurationChanges` para detectar se as informações de configuração no armazenamento de blobs foi alterado. Ele faz isso examinando o número de versão e comparando-o com o número de versão atual do objeto `ExternalConfigurationManager`. Se uma ou mais alterações ocorrerem, o evento `Changed` é acionado e as definições de configuração armazenados em cache no objeto `Dictionary` são atualizadas. Este é um aplicativo do [Padrão cache-aside](./cache-aside.md).

O exemplo de código a seguir mostra como o evento `Changed`, o método `GetSettings` e o método `CheckForConfigurationChanges` são implementados:

```csharp
public class ExternalConfigurationManager : IDisposable
{
  // An abstraction of the configuration store.
  private readonly ISettingsStore settings;
  private readonly ISubject<KeyValuePair<string, string>> changed;
  ...
  private readonly ReaderWriterLockSlim settingsCacheLock = new ReaderWriterLockSlim();
  private readonly SemaphoreSlim syncCacheSemaphore = new SemaphoreSlim(1);  
  ...
  private Dictionary<string, string> settingsCache;
  private string currentVersion;
  ...
  public ExternalConfigurationManager(ISettingsStore settings, ...)
  {
    this.settings = settings;
    ...
  }
  ...
  public IObservable<KeyValuePair<string, string>> Changed => this.changed.AsObservable();
  ...

  public string GetAppSetting(string key)
  {
    ...
    // Try to get the value from the settings cache.
    // If there's a cache miss, get the setting from the settings store and refresh the settings cache.

    string value;
    try
    {
        this.settingsCacheLock.EnterReadLock();

        this.settingsCache.TryGetValue(key, out value);
    }
    finally
    {
        this.settingsCacheLock.ExitReadLock();
    }

    return value;
  }
  ...
  private void CheckForConfigurationChanges()
  {
    try
    {
        // It is assumed that updates are infrequent.
        // To avoid race conditions in refreshing the cache, synchronize access to the in-memory cache.
        await this.syncCacheSemaphore.WaitAsync();

        var latestVersion = await this.settings.GetVersionAsync();

        // If the versions are the same, nothing has changed in the configuration.
        if (this.currentVersion == latestVersion) return;

        // Get the latest settings from the settings store and publish changes.
        var latestSettings = await this.settings.FindAllAsync();

        // Refresh the settings cache.
        try
        {
            this.settingsCacheLock.EnterWriteLock();

            if (this.settingsCache != null)
            {
                //Notify settings changed
                latestSettings.Except(this.settingsCache).ToList().ForEach(kv => this.changed.OnNext(kv));
            }
            this.settingsCache = latestSettings;
        }
        finally
        {
            this.settingsCacheLock.ExitWriteLock();
        }

        // Update the current version.
        this.currentVersion = latestVersion;
    }
    catch (Exception ex)
    {
        this.changed.OnError(ex);
    }
    finally
    {
        this.syncCacheSemaphore.Release();
    }
  }
}
```

> A classe `ExternalConfigurationManager` também fornece uma propriedade chamada `Environment`. Essa propriedade dá suporte a configurações diferentes para um aplicativo em execução em ambientes diferentes, como preparo e produção.

Um objeto `ExternalConfigurationManager` também pode consultar o objeto `BlobSettingsStore` periodicamente para quaisquer alterações. No código a seguir, o método `StartMonitor` chama `CheckForConfigurationChanges` em um intervalo para detectar possíveis alterações e aciona o evento `Changed`, conforme descrito anteriormente.

```csharp
public class ExternalConfigurationManager : IDisposable
{
  ...
  private readonly ISubject<KeyValuePair<string, string>> changed;
  private Dictionary<string, string> settingsCache;
  private readonly CancellationTokenSource cts = new CancellationTokenSource();
  private Task monitoringTask;
  private readonly TimeSpan interval;

  private readonly SemaphoreSlim timerSemaphore = new SemaphoreSlim(1);
  ...
  public ExternalConfigurationManager(string environment) : this(new BlobSettingsStore(environment), TimeSpan.FromSeconds(15), environment)
  {
  }
  
  public ExternalConfigurationManager(ISettingsStore settings, TimeSpan interval, string environment)
  {
      this.settings = settings;
      this.interval = interval;
      this.CheckForConfigurationChangesAsync().Wait();
      this.changed = new Subject<KeyValuePair<string, string>>();
      this.Environment = environment;
  }
  ...
  /// <summary>
  /// Check to see if the current instance is monitoring for changes
  /// </summary>
  public bool IsMonitoring => this.monitoringTask != null && !this.monitoringTask.IsCompleted;

  /// <summary>
  /// Start the background monitoring for configuration changes in the central store
  /// </summary>
  public void StartMonitor()
  {
      if (this.IsMonitoring)
          return;

      try
      {
          this.timerSemaphore.Wait();

          // Check again to make sure we are not already running.
          if (this.IsMonitoring)
              return;

          // Start running our task loop.
          this.monitoringTask = ConfigChangeMonitor();
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  /// <summary>
  /// Loop that monitors for configuration changes
  /// </summary>
  /// <returns></returns>
  public async Task ConfigChangeMonitor()
  {
      while (!cts.Token.IsCancellationRequested)
      {
          await this.CheckForConfigurationChangesAsync();
          await Task.Delay(this.interval, cts.Token);
      }
  }

  /// <summary>
  /// Stop monitoring for configuration changes
  /// </summary>
  public void StopMonitor()
  {
      try
      {
          this.timerSemaphore.Wait();

          // Signal the task to stop.
          this.cts.Cancel();

          // Wait for the loop to stop.
          this.monitoringTask.Wait();

          this.monitoringTask = null;
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  public void Dispose()
  {
      this.cts.Cancel();
  }
  ...
}
```

A classe `ExternalConfigurationManager` é instanciada como uma instância singleton pela classe `ExternalConfiguration` mostrada abaixo.

```csharp
public static class ExternalConfiguration
{
    private static readonly Lazy<ExternalConfigurationManager> configuredInstance = new Lazy<ExternalConfigurationManager>(
        () =>
        {
            var environment = CloudConfigurationManager.GetSetting("environment");
            return new ExternalConfigurationManager(environment);
        });

    public static ExternalConfigurationManager Instance => configuredInstance.Value;
}
```

O código a seguir é obtido da classe `WorkerRole` no projeto _ExternalConfigurationStore.Cloud_. Ele mostra como o aplicativo usa a classe `ExternalConfiguration` para ler uma configuração.

```csharp
public override void Run()
{
  // Start monitoring configuration changes.
  ExternalConfiguration.Instance.StartMonitor();

  // Get a setting.
  var setting = ExternalConfiguration.Instance.GetAppSetting("setting1");
  Trace.TraceInformation("Worker Role: Get setting1, value: " + setting);

  this.completeEvent.WaitOne();
}
```

O código a seguir, também da classe `WorkerRole`, mostra como o aplicativo assina os eventos de configuração.

```csharp
public override bool OnStart()
{
  ...
  // Subscribe to the event.
  ExternalConfiguration.Instance.Changed.Subscribe(
     m => Trace.TraceInformation("Configuration has changed. Key:{0} Value:{1}",
          m.Key, m.Value),
     ex => Trace.TraceError("Error detected: " + ex.Message));
  ...
}
```

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados

- Um exemplo que demonstra esse padrão está disponível em [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).
