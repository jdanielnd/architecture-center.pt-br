---
title: Padrão de Hospedagem de Conteúdo Estático
titleSuffix: Cloud Design Patterns
description: Implante conteúdo estático em um serviço de armazenamento baseado em nuvem que pode enviá-lo diretamente para o cliente.
keywords: padrão de design
author: dragon119
ms.date: 01/04/2019
ms.custom: seodec18
ms.openlocfilehash: cf4f65e935a01e4d84b3cc82b5779edb729bd80e
ms.sourcegitcommit: 036cd03c39f941567e0de4bae87f4e2aa8c84cf8
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/05/2019
ms.locfileid: "54058175"
---
# <a name="static-content-hosting-pattern"></a>Padrão de Hospedagem de Conteúdo Estático

Implante conteúdo estático em um serviço de armazenamento baseado em nuvem que pode enviá-lo diretamente para o cliente. Isso pode reduzir a necessidade de instâncias de computação potencialmente caras.

## <a name="context-and-problem"></a>Contexto e problema

Aplicativos Web geralmente incluem alguns elementos de conteúdo estático. Este conteúdo estático pode incluir páginas HTML e outros recursos como imagens e documentos que estão disponíveis para o cliente, como parte de uma página HTML (tal como imagens embutidas, folhas de estilo e arquivos JavaScript do lado do cliente) ou como downloads separados (como documentos PDF).

Embora os servidores Web sejam otimizados para renderização dinâmica e cache de saída, eles ainda precisarão lidar com solicitações para fazer o download de conteúdo estático. Isso consome ciclos de processamento que muitas vezes poderiam ser melhor usados em outros locais.

## <a name="solution"></a>Solução

Na maioria dos ambientes de hospedagem em nuvem, você pode colocar alguns dos recursos e páginas estáticas do aplicativo em um serviço de armazenamento. O serviço de armazenamento pode atender solicitações para esses recursos, reduzindo a carga nos recursos de computação que lidam com outras solicitações da Web. O custo do armazenamento hospedado em nuvem normalmente é muito menor que o de instâncias de computação.

Ao hospedar algumas partes de um aplicativo em um serviço de armazenamento, as principais considerações estão relacionadas à implantação do aplicativo e à proteção de recursos que não devem estar disponíveis para os usuários anônimos.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

- O serviço de armazenamento hospedado deve expor um ponto de extremidade HTTP que os usuários podem acessar para baixar os recursos estáticos. Alguns serviços de armazenamento também dão suporte a HTTPS, portanto, é possível hospedar recursos nos serviços de armazenamento que exigem SSL.

- Para obter o máximo de desempenho e disponibilidade, considere usar uma CDN (rede de distribuição de conteúdo) para armazenar em cache o conteúdo do contêiner de armazenamento em vários datacenters ao redor do mundo. No entanto, você provavelmente precisará pagar para usar a CDN.

- Geralmente, as contas de armazenamento são replicados geograficamente por padrão para oferecer resiliência contra eventos que podem afetar um data center. Isso significa que o endereço IP pode mudar, mas a URL permanecerá a mesma.

- Quando parte do conteúdo está localizado em uma conta de armazenamento e outras partes do conteúdo estão em uma instância de computação hospedada, é mais difícil implantar e atualizar o aplicativo. Pode ser necessário executar implantações separadas e controlar a versão do aplicativo e do conteúdo para gerenciá-la mais facilmente &mdash; especialmente quando o conteúdo estático inclui arquivos de script ou componentes de interface do usuário. No entanto, se apenas recursos estáticos devem ser atualizados, eles podem simplesmente ser carregados para a conta de armazenamento sem precisar reimplantar o pacote de aplicativos.

- Os serviços de armazenamento podem não dar suporte ao uso de nomes de domínio personalizados. Nesse caso é necessário especificar a URL completa dos recursos nos links porque eles estarão em um domínio diferente do conteúdo gerado dinamicamente que contém os links.

- Os contêineres de armazenamento devem ser configurados para acesso de leitura público, mas é essencial garantir que eles não estejam configurados para acesso de gravação público para impedir que os usuários possam carregar o conteúdo.

- Considere usar uma chave limitada ou token para controlar o acesso aos recursos que não devem estar disponíveis anonimamente. Confira o [padrão de Chave Limitada](./valet-key.md) para ver mais informações.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Esse padrão é útil para:

- Minimizar o custo de hospedagem de sites e aplicativos que contêm alguns recursos estáticos.

- Minimizar o custo de hospedagem de sites compostos somente por conteúdo e recursos estáticos. Dependendo das funcionalidades do sistema de armazenamento do provedor de hospedagem, seria possível hospedar inteiramente um site totalmente estático em uma conta de armazenamento.

- Expor os recursos e conteúdo estático para aplicativos em execução em outros ambientes de hospedagem ou nos servidores locais.

- Localizar conteúdo em mais de uma área geográfica usando uma rede de distribuição de conteúdo que armazena em cache o conteúdo da conta de armazenamento em vários datacenters ao redor do mundo.

- Monitorar os custos e uso de largura de banda. Usar uma conta de armazenamento separada para parte ou todo o conteúdo estático permite que os custos sejam separados mais facilmente dos custos de hospedagem e de tempo de execução.

Esse padrão pode não ser útil nas seguintes situações:

- O aplicativo precisa executar processamento no conteúdo estático antes de distribuí-lo para o cliente. Por exemplo, pode ser necessário adicionar um carimbo de data/hora a um documento.

- O volume do conteúdo estático é muito pequeno. A sobrecarga de recuperar o conteúdo do armazenamento separado pode superar o benefício de custo de separá-lo do recurso de computação.

## <a name="example"></a>Exemplo

O Armazenamento do Azure dá suporte ao fornecimento de conteúdo estático diretamente de um contêiner de armazenamento. Os arquivos são atendidos por meio de solicitações de acesso anônimo. Por padrão, os arquivos têm uma URL em um subdomínio `core.windows.net`, como `https://contoso.z4.web.core.windows.net/image.png`. Você pode configurar um nome de domínio personalizado e usar a CDN do Azure para acessar os arquivos por HTTPS. Para mais informações, confira [Hospedagem de site estático no Armazenamento do Azure](/azure/storage/blobs/storage-blob-static-website).

![Distribuição de partes estáticas de um aplicativo diretamente de um serviço de armazenamento](./_images/static-content-hosting-pattern.png)

A hospedagem de site estático torna os arquivos disponíveis para acesso anônimo. Se precisar controlar quem pode acessar os arquivos, você pode armazenar arquivos no armazenamento de blogs do Azure e, em seguida, gerar [assinaturas de acesso compartilhado](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) para limitar o acesso.

Os links nas páginas distribuídas para o cliente devem especificar a URL completa do recurso. Se o recurso estiver protegido por uma chave limitada, como uma assinatura de acesso compartilhado do Azure, essa assinatura deverá ser incluída na URL.

Um aplicativo de exemplo que demonstra o uso do armazenamento externo para recursos estáticos está disponível no [GitHub][sample-app]. Este exemplo usa arquivos de configuração que especificam a conta de armazenamento e o contêiner que retém o conteúdo estático.

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

A classe `Settings` no arquivo Settings.cs do projeto StaticContentHosting.Web contém métodos para extrair esses valores e criar um valor de cadeia de caracteres que contém a URL de contêiner da conta de armazenamento de nuvem.

```csharp
public class Settings
{
  public static string StaticContentStorageConnectionString {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue(
                              "StaticContent.StorageConnectionString");
    }
  }

  public static string StaticContentContainer
  {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue("StaticContent.Container");
    }
  }

  public static string StaticContentBaseUrl
  {
    get
    {
      var account = CloudStorageAccount.Parse(StaticContentStorageConnectionString);

      return string.Format("{0}/{1}", account.BlobEndpoint.ToString().TrimEnd('/'),
                                      StaticContentContainer.TrimStart('/'));
    }
  }
}
```

A classe `StaticContentUrlHtmlHelper` no arquivo StaticContentUrlHtmlHelper.cs expõe um método chamado `StaticContentUrl` que gera uma URL que contém o caminho para a conta de armazenamento de nuvem se a URL passada para ele começar com o caractere de caminho raiz do ASP.NET (~).

```csharp
public static class StaticContentUrlHtmlHelper
{
  public static string StaticContentUrl(this HtmlHelper helper, string contentPath)
  {
    if (contentPath.StartsWith("~"))
    {
      contentPath = contentPath.Substring(1);
    }

    contentPath = string.Format("{0}/{1}", Settings.StaticContentBaseUrl.TrimEnd('/'),
                                contentPath.TrimStart('/'));

    var url = new UrlHelper(helper.ViewContext.RequestContext);

    return url.Content(contentPath);
  }
}
```

O arquivo Index.cshtml na pasta Views\Home contém um elemento de imagem que usa o método `StaticContentUrl` para criar a URL para o seu atributo `src`.

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados

- [Exemplo de hospedagem de conteúdo estático][sample-app]. Um aplicativo de exemplo que demonstra esse padrão.
- [Padrão de Chave de Manobrista](./valet-key.md). Se os recursos de destino não estiverem disponíveis para usuários anônimos, use esse padrão para restringir o acesso direto.
- [Aplicativo Web sem servidor no Azure](../reference-architectures/serverless/web-app.md). Uma arquitetura de referência que usa a hospedagem de site estático com o Azure Functions para implementar um aplicativo Web sem servidor.

[sample-app]: https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting
