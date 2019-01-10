---
title: Padrão Valet Key
titleSuffix: Cloud Design Patterns
description: Use um token ou chave que fornece aos clientes acesso direto e restrito a um determinado recurso ou serviço.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 09173717d499d524d4d5dad2c1202c1bf361b1e5
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009858"
---
# <a name="valet-key-pattern"></a>Padrão Valet Key

[!INCLUDE [header](../_includes/header.md)]

Use um token que fornece aos clientes acesso direto e restrito a um recurso específico, para descarregar a transferência de dados do aplicativo. Isso é particularmente útil em aplicativos que utilizam filas ou sistemas de armazenamento hospedados na nuvem e podem minimizar custos e maximizar a escalabilidade e o desempenho.

## <a name="context-and-problem"></a>Contexto e problema

Programas cliente e navegadores da Web geralmente precisam ler e gravar arquivos ou fluxos de dados para e do armazenamento de um aplicativo. Normalmente, o aplicativo irá tratar o movimento dos dados &mdash;, seja buscando-o do armazenamento e transmitido-o por streaming para o cliente, ou lendo o fluxo carregado do cliente e armazenando-o no armazenamento de dados. No entanto, essa abordagem absorve recursos valiosos, como computação, memória e largura de banda.

Os repositórios de dados têm a capacidade de tratar o upload e download de dados diretamente, sem exigir que o aplicativo execute qualquer processamento para mover esses dados. Mas, isso normalmente exige que o cliente tenha acesso às credenciais de segurança do repositório. Essa pode ser uma técnica útil para minimizar os custos de transferência de dados e os requisitos para escalar horizontalmente o aplicativo e maximizar o desempenho. Isso significa, porém, que o aplicativo não é mais capaz de gerenciar a segurança dos dados. Depois que o cliente tiver uma conexão com o repositório de dados para acesso direto, o aplicativo não poderá agir como o gatekeeper. Pois já não estará mais no controle do processo e não poderá impedir de  fazer uploads ou downloads subsequentes do repositório de dados.

Essa não é uma abordagem realista em sistemas distribuídos que precisam servir clientes não confiáveis. Em vez disso, os aplicativos devem poder controlar de forma segura o acesso aos dados de forma granular, mas ainda reduzir a carga no servidor configurando essa conexão e, em seguida, permitindo que o cliente se comunique diretamente com o repositório de dados para executar as operações de leitura ou gravação necessárias .

## <a name="solution"></a>Solução

É necessário resolver o problema de controle de acesso a um repositório de dados, de maneira que o repositório não possa gerenciar a autenticação e autorização de clientes. Uma solução típica é restringir o acesso à conexão pública do repositório de dados e fornecer ao cliente uma chave ou token que possa ser validado pelo repositório de dados.

Essa chave ou token geralmente é referido como uma valet key. Ele fornece acesso com limite de tempo a recursos específicos e permite apenas operações pré-definidas, como leitura e gravação para armazenamento ou filas, ou fazer o upload ou o download em um navegador da Web. Os aplicativos podem criar e emitir keys valet para dispositivos de cliente e navegadores da Web de forma rápida e fácil, permitindo que os clientes executem as operações necessárias sem exigir que o aplicativo trate diretamente a transferência de dados. Isso remove a sobrecarga de processamento e o impacto no desempenho e escalabilidade, a partir do aplicativo e do servidor.

O cliente usa esse token para acessar um recurso específico no repositório de dados por apenas um período específico e com restrições específicas nas permissões de acesso, conforme mostrado na figura. Após o período especificado, a chave torna-se inválida e não permitirá o acesso ao recurso.

![Figura 1 - Visão geral do padrão](./_images/valet-key-pattern.png)

Também é possível configurar uma chave que tenha outras dependências, como o escopo dos dados. Por exemplo, dependendo dos recursos do repositório de dados, a chave pode especificar uma tabela completa em um repositório de dados ou apenas as linhas específicas em uma tabela. Nos sistemas de armazenamento em nuvem, a chave pode especificar um contêiner ou apenas um item específico dentro de um contêiner.

A chave também pode ser invalidada pelo aplicativo. Essa é uma abordagem útil se o cliente notificar o servidor que a operação de transferência de dados está completa. O servidor poderá invalidar essa chave para evitar mais acesso.

Usar esse padrão pode simplificar o gerenciamento de acesso aos recursos, porque não há necessidade de criar e autenticar um usuário, conceder permissões e, posteriormente, remover o usuário novamente. Isso também facilita limitar a localização, a permissão e do período de validade&mdash;, simplesmente gerando uma chave no tempo de execução. Os fatores importantes são para limitar o período de validade e especialmente a localização do recurso, tão rigorosamente o quanto possível para que o destinatário possa usá-la apenas para o propósito pretendido.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

**Gerenciar o status de validade e o período da chave**. Se vazada ou comprometida, a chave efetivamente desbloqueará o item de destino e o disponibilizará para uso malicioso durante o período de validade. Uma chave geralmente pode ser revogada ou desabilitada, dependendo de como foi emitida. As políticas do servidor podem ser alteradas ou a chave do servidor, com a qual foi assinada, pode ser invalidada. Especifique um período de validade curto para minimizar o risco de permitir que operações não autorizadas ocorram contra o repositório de dados. No entanto, se o período de validade for muito curto, o cliente não poderá concluir a operação antes que a chave expire. Permita que os usuários autorizados renovem a chave antes do período de validade expirar, caso sejam necessários múltiplos acessos ao recurso protegido.

**Controle o nível de acesso que a chave fornecerá**. Normalmente, a chave deve permitir ao usuário executar apenas as ações necessárias para concluir a operação, como o acesso somente leitura se o cliente não puder fazer o upload de dados no repositório de dados. Para carregamentos de arquivos é comum especificar uma chave que forneça permissão somente gravação, assim como a localização e o período de validade. É fundamental especificar com precisão o recurso ou o conjunto de recursos ao qual a chave se aplica.

**Considere como controlar o comportamento dos usuários**. A implementação desse padrão significa uma perda de controle sobre os recursos que os usuários possuem acesso. O nível de controle que pode ser exercido é limitado pelos recursos das políticas e permissões disponíveis para o serviço ou o repositório de dados de destino. Por exemplo, geralmente não é possível criar uma chave que limita o tamanho dos dados a serem gravados no armazenamento, ou o número de vezes que a chave pode ser usada para acessar um arquivo. Isso pode resultar em enormes custos inesperados para a transferência de dados, mesmo quando usado pelo cliente pretendido e podem ser causados por um erro no código que causa upload ou download repetido. Para limitar o número de vezes que um arquivo pode ser carregado, sempre que possível, force o cliente a notificar o aplicativo quando uma operação for concluída. Por exemplo, alguns repositórios de dados aumentam os eventos que o código do aplicativo pode usar para monitorar as operações e controlar o comportamento do usuário. No entanto, é difícil exigir cotas para usuários individuais em um cenário multilocatário, onde a mesma chave é utilizada por todos os usuários de um locatário.

**Validar, e opcionalmente limpar, todos os dados carregados**. Um usuário mal-intencionado que obtenha acesso à chave poderá carregar dados projetados para comprometer o sistema. Alternativamente, usuários autorizados podem carregar dados inválidos e, quando processados, poderão resultar em um erro ou falha no sistema. Para proteger contra isso, assegure-se de que todos os dados carregados sejam validados e verificados quanto a conteúdo malicioso antes de serem utilizados.

**Auditar todas as operações**. Muitos mecanismos baseados em chave podem registrar operações como uploads, downloads e falhas. Esses registros geralmente podem ser incorporados em um processo de auditoria e, também, utilizados para cobrança se o usuário for cobrado com base no tamanho do arquivo ou no volume de dados. Use os registros para detectar falhas de autenticação que podem ser causadas por problemas com o provedor de chave ou remoção acidental de uma política de acesso armazenada.

**Entregar a chave com segurança**. É possível ser inserida em uma URL que o usuário ativa em uma página da Web, ou pode ser utilizada em uma operação de redirecionamento do servidor para que o download ocorra automaticamente. Sempre use HTTPS para entregar a chave por um canal seguro.

**Proteger dados confidenciais em trânsito**. Os dados confiáveis entregues através do aplicativo geralmente ocorrem usando SSL ou TLS e isso deve ser aplicado para clientes que acessem o repositório de dados diretamente.

Outras questões a serem reconhecidas ao implementar esse padrão são:

- Se o cliente não notificar o servidor da conclusão da operação, ou não puder fazê-la, e o único limite for o período de validade da chave, o aplicativo não poderá realizar operações de auditoria, tais como contar o número de uploads ou downloads, ou impedir múltiplos carregamentos ou downloads.

- A flexibilidade das políticas de chave que podem ser geradas pode ser limitada. Por exemplo, alguns mecanismos somente permitem o uso de um período de validade cronometrado. Outros não são capazes de especificar uma granularidade suficiente de permissões de leitura/gravação.

- Se a hora de início para o período de validade de chave ou token for especificada, assegure-se de que seja um pouco mais cedo do que o tempo atual do servidor para permitir que os relógios do cliente possam estar ligeiramente fora da sincronização. O padrão, se não especificado, geralmente é o tempo atual do servidor.

- A URL que contém a chave será gravada em arquivos de log do servidor. Embora a chave normalmente tenha expirado antes que os arquivos de log sejam utilizados para análise, assegure-se de limitar o acesso. Se os dados de log forem transmitidos para um sistema de monitoramento ou armazenados em outro local, considere implementar um atraso para evitar vazamentos de chaves até que o período de validade tenha expirado.

- Se o código do cliente for executado em um navegador da Web, talvez o navegador precise suportar o CORS (Compartilhamento de Recursos entre Origens) para habilitar o código que executa dentro do navegador da Web para acessar dados em um domínio diferente daquele que atendia a página. Alguns navegadores mais antigos e alguns repositórios de dados não suportam o CORS e, o código que executa nesses navegadores pode usar uma valet key para fornecer o acesso para dados em um domínio diferente, como uma conta de armazenamento em nuvem.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Esse padrão não é útil para as seguintes situações:

- Minimizar o carregamento de recursos e maximizar o desempenho e a escalabilidade. O uso de uma valet key não requer que o recurso seja bloqueado, nenhuma chamada remota do servidor é necessária, não há limite no número de valet keys que possam ser emitidas e evita um ponto único de falha resultante da execução da transferência de dados através do código do aplicativo. Criar uma valet key é tipicamente uma operação criptográfica simples de autenticar uma cadeia de caracteres com uma chave.

- Minimizar o custo operacional. Permitir acesso direto a repositórios e filas é um recurso economicamente eficiente, podendo resultar em menos viagens de ida e volta da rede e permitir uma redução no número de recursos de computação necessários.

- Quando os clientes fazem o upload ou download de dados regularmente, especialmente quando há um grande volume ou quando cada operação envolve arquivos grandes.

- Quando o aplicativo possui recursos de computação limitados disponíveis, devido a limitações de hospedagem ou considerações de custo. Nesse cenário, o padrão é ainda mais útil se houver muitos uploads ou downloads simultâneos de dados, pois alivia o aplicativo de tratar com a transferência de dados.

- Quando os dados são armazenados em um repositório de dados remoto ou em um datacenter diferente. Se o aplicativo fosse necessário para agir como um gatekeeper, poderia haver uma cobrança pela largura de banda adicional da transferência de dados entre datacenters, ou entre redes públicas ou privadas entre o cliente e o aplicativo e, depois, entre o aplicativo e o repositório de dados.

Esse padrão pode não ser útil nas seguintes situações:

- Se o aplicativo tiver que executar alguma tarefa nos dados antes de ser armazenado ou antes de ser enviado ao cliente. Por exemplo, se o aplicativo precisar executar a validação, acessar o log com êxito ou executar uma transformação nos dados. No entanto, alguns repositórios de dados e clientes são capazes de negociar e realizar transformações simples, como compactação e descompactação (por exemplo, um navegador da Web geralmente pode tratar formatos GZip).

- Se o design de um aplicativo existente dificulta a incorporação do padrão. O uso deste padrão normalmente requer uma abordagem arquitetônica diferente para entregar e receber dados.

- Se for necessário manter trilhas de auditoria ou controlar o número de vezes que uma operação de transferência de dados é executada e o mecanismo de valet key em uso não fornece suporte para notificações que o servidor pode usar para gerenciar essas operações.

- Se for necessário limitar o tamanho dos dados, especialmente durante as operações de upload. A única solução para isso é que o aplicativo verifique o tamanho dos dados após a conclusão da operação ou verifique o tamanho dos uploads após um período especificado ou com base em um agendamento.

## <a name="example"></a>Exemplo

O Azure fornece suporte a assinaturas de acesso compartilhado no Armazenamento do Microsoft Azure para o controle de acesso granular para dados em blobs, tabelas e filas e para filas e tópicos do Barramento de Serviço. Um token de assinatura de acesso compartilhado pode ser configurado para fornecer direitos de acesso específicos, como ler, escrever, atualizar e excluir em uma tabela específica; um intervalo chaves dentro de uma tabela; uma fila; um blob; ou um contêiner de blob. A validade pode ser um período de tempo especificado ou sem limite de tempo.

As assinaturas de acesso compartilhado do Azure também fornecem suporte a políticas de acesso armazenadas em servidor que podem ser associadas a um recurso específico, como uma tabela ou blob. Esse recurso fornece controle e flexibilidade adicionais em comparação com tokens de assinatura de acesso compartilhado gerado por aplicativos e deve ser usado sempre que possível. As configurações definidas em uma política armazenada em servidor podem ser alteradas e são refletidas no token sem requerer que um novo token seja emitido, mas, as configurações definidas no token não podem ser alteradas sem emitir um novo token. Essa abordagem também permite revogar um token de assinatura de acesso compartilhado válido antes de expirar.

> Para obter mais informações, consulte [Introdução à SAS (Assinatura de Acesso Compartilhado) de tabela, SAS de fila e atualização para SAS de Blob](https://blogs.msdn.microsoft.com/windowsazurestorage/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas/) e [Uso de assinaturas de acesso compartilhado](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) no MSDN.

O código a seguir mostra como criar um token de assinatura de acesso compartilhado válido por cinco minutos. O método `GetSharedAccessReferenceForUpload` retorna um token de assinatura de acesso compartilhado que pode ser usado para carregar um arquivo para o Armazenamento de Blobs do Azure.

```csharp
public class ValuesController : ApiController
{
  private readonly CloudStorageAccount account;
  private readonly string blobContainer;
  ...
  /// <summary>
  /// Return a limited access key that allows the caller to upload a file
  /// to this specific destination for a defined period of time.
  /// </summary>
  private StorageEntitySas GetSharedAccessReferenceForUpload(string blobName)
  {
    var blobClient = this.account.CreateCloudBlobClient();
    var container = blobClient.GetContainerReference(this.blobContainer);

    var blob = container.GetBlockBlobReference(blobName);

    var policy = new SharedAccessBlobPolicy
    {
      Permissions = SharedAccessBlobPermissions.Write,

      // Specify a start time five minutes earlier to allow for client clock skew.
      SharedAccessStartTime = DateTime.UtcNow.AddMinutes(-5),

      // Specify a validity period of five minutes starting from now.
      SharedAccessExpiryTime = DateTime.UtcNow.AddMinutes(5)
    };

    // Create the signature.
    var sas = blob.GetSharedAccessSignature(policy);

    return new StorageEntitySas
    {
      BlobUri = blob.Uri,
      Credentials = sas,
      Name = blobName
    };
  }

  public struct StorageEntitySas
  {
    public string Credentials;
    public Uri BlobUri;
    public string Name;
  }
}
```

> A amostra completa está disponível na solução ValetKey disponível para fazer o download a partir do [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/valet-key). O projeto ValetKey.Web nessa solução contém um aplicativo da Web que inclui a classe `ValuesController` mostrada acima. Um aplicativo cliente de amostra que utiliza esse aplicativo Web para recuperar uma chave de assinaturas de acesso compartilhado e carregar um arquivo para o armazenamento de Blobs está disponível no projeto ValetKey.Client.

## <a name="next-steps"></a>Próximas etapas

Os padrões e diretrizes a seguir também podem ser relevantes ao implementar esse padrão:

- Um exemplo que demonstra esse padrão está disponível em [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/valet-key).
- [Padrão de gatekeeper](./gatekeeper.md). Este padrão pode ser utilizado em conjunto com o padrão Valet Key para proteger aplicativos e serviços, utilizando uma instância de host dedicada que age como agente entre clientes e o aplicativo ou serviço. O gatekeeper valida e limpa as solicitações, e transmite as solicitações e os dados entre o cliente e o aplicativo. Isso pode fornecer uma camada adicional de segurança e reduzir a superfície de ataque do sistema.
- [Padrão de hospedagem de conteúdo estático](./static-content-hosting.md). Descreve como implantar recursos estáticos em um serviço de armazenamento baseado em nuvem que pode entregar esses recursos diretamente ao cliente para reduzir o requisito de instâncias de computação dispendiosas. Onde os recursos não se destinam a ser publicamente disponíveis, o padrão Valet Key poderá ser usado para protegê-los.
- [Introdução à SAS (Assinatura de Acesso Compartilhado) de tabela, SAS de fila e atualização para SAS de Blob](https://blogs.msdn.microsoft.com/windowsazurestorage/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas/)
- [Uso de assinaturas de acesso compartilhado](/azure/storage/common/storage-dotnet-shared-access-signature-part-1)
- [Autenticação de assinatura de acesso compartilhado com Barramento de serviço](/azure/service-bus-messaging/service-bus-sas)
