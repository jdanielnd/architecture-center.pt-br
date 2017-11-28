---
title: "Hospedagem de Conteúdo Estático"
description: "Implante conteúdo estático em um serviço de armazenamento baseado em nuvem que pode enviá-lo diretamente para o cliente."
keywords: "padrão de design"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: deb15001bea2598d56a2793be78bbc3e7473bdf3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="static-content-hosting-pattern"></a>Padrão de Hospedagem de Conteúdo Estático

[!INCLUDE [header](../_includes/header.md)]

Implante conteúdo estático em um serviço de armazenamento baseado em nuvem que pode enviá-lo diretamente para o cliente. Isso pode reduzir a necessidade de instâncias de computação potencialmente caras.

## <a name="context-and-problem"></a>Contexto e problema

Aplicativos Web geralmente incluem alguns elementos de conteúdo estático. Este conteúdo estático pode incluir páginas HTML e outros recursos como imagens e documentos que estão disponíveis para o cliente, como parte de uma página HTML (tal como imagens embutidas, folhas de estilo e arquivos JavaScript do lado do cliente) ou como downloads separados (como documentos PDF).

Embora os servidores Web estejam bem ajustados para otimizar as solicitações por meio da execução de código de página dinâmica e cache de saída eficientes, eles ainda precisarão lidar com as solicitações para baixar conteúdo estático. Isso consome ciclos de processamento que muitas vezes poderiam ser melhor usados em outros locais.

## <a name="solution"></a>Solução

Na maioria dos ambientes de hospedagem em nuvem, é possível minimizar a necessidade de instâncias de computação (por exemplo, usar uma instância menor ou menos instâncias) colocando algumas das páginas estáticas e recursos de um aplicativo em um serviço de armazenamento. O custo do armazenamento hospedado em nuvem normalmente é muito menor que o de instâncias de computação.

Ao hospedar algumas partes de um aplicativo em um serviço de armazenamento, as principais considerações estão relacionadas à implantação do aplicativo e à proteção de recursos que não devem estar disponíveis para os usuários anônimos.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

- O serviço de armazenamento hospedado deve expor um ponto de extremidade HTTP que os usuários podem acessar para baixar os recursos estáticos. Alguns serviços de armazenamento também dão suporte a HTTPS, portanto, é possível hospedar recursos nos serviços de armazenamento que exigem SSL.

- Para obter o máximo de desempenho e disponibilidade, considere usar uma CDN (rede de distribuição de conteúdo) para armazenar em cache o conteúdo do contêiner de armazenamento em vários datacenters ao redor do mundo. No entanto, você provavelmente precisará pagar para usar a CDN.

- Geralmente, as contas de armazenamento são replicados geograficamente por padrão para oferecer resiliência contra eventos que podem afetar um data center. Isso significa que o endereço IP pode mudar, mas a URL permanecerá a mesma.

- Quando parte do conteúdo está localizado em uma conta de armazenamento e outras partes do conteúdo estão em uma instância de computação hospedada, é mais difícil de implantar um aplicativo e atualizá-lo. Pode ser necessário executar implantações separadas e controlar a versão do aplicativo e do conteúdo para gerenciá-la mais facilmente &mdash; especialmente quando o conteúdo estático inclui arquivos de script ou componentes de interface do usuário. No entanto, se apenas recursos estáticos devem ser atualizados, eles podem simplesmente ser carregados para a conta de armazenamento sem precisar reimplantar o pacote de aplicativos.

- Os serviços de armazenamento podem não dar suporte ao uso de nomes de domínio personalizados. Nesse caso é necessário especificar a URL completa dos recursos nos links porque eles estarão em um domínio diferente do conteúdo gerado dinamicamente que contém os links.

- Os contêineres de armazenamento devem ser configurados para acesso de leitura público, mas é essencial garantir que eles não estejam configurados para acesso de gravação público para impedir que os usuários possam carregar o conteúdo. Considere usar uma chave de manobrista ou token para controlar o acesso aos recursos que não devem estar disponíveis anonimamente &mdash; consulte o [Padrão de Valet Key](valet-key.md) para obter mais informações.

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

O conteúdo estático localizado no armazenamento de Blobs do Azure pode ser acessado diretamente por um navegador da Web. O Azure fornece uma interface baseada em HTTP para o armazenamento que pode ser exposto publicamente para os clientes. Por exemplo, o conteúdo em um contêiner de armazenamento de Blobs do Azure é exposto usando uma URL com o seguinte formato:

`http://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


Ao carregar o conteúdo, é necessário criar um ou mais contêineres de blob para conter os arquivos e documentos. Observe que a permissão padrão para um novo contêiner é Privado e você deve alterar isso para Público para permitir que os clientes acessem o conteúdo. Se for necessário proteger o conteúdo do acesso anônimo, você poderá implementar o [padrão de Chave de Manobrista](valet-key.md) para que os usuários precisem apresentar um token válido para baixar os recursos.

> Os [Conceitos do Serviço Blob](https://msdn.microsoft.com/library/azure/dd179376.aspx) trazem informações sobre o armazenamento de blobs e as maneiras para acessá-los e usá-los.

Os links em cada página especificam a URL do recurso e o cliente irá acessá-la diretamente do serviço de armazenamento. A figura ilustra a distribuição de partes estáticas de um aplicativo diretamente de um serviço de armazenamento.

![Figura 1 – Distribuição de partes estáticas de um aplicativo diretamente de um serviço de armazenamento](./_images/static-content-hosting-pattern.png)


Os links nas páginas distribuídas para o cliente devem especificar a URL completa do contêiner e recursos de blobs. Por exemplo, uma página que contém um link para uma imagem em um contêiner público pode conter o seguinte HTML.

```html
<img src="http://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> Se os recursos forem protegidos usando uma chave de manobrista, como uma assinatura de acesso compartilhado do Azure, essa assinatura deverá ser incluída nas URLs nos links.

Uma solução chamada StaticContentHosting que demonstra o uso do armazenamento externo para recursos estáticos está disponível no [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting). O projeto StaticContentHosting.Cloud contém arquivos de configuração que especificam a conta de armazenamento e o contêiner que retém o conteúdo estático.

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

- Um exemplo que demonstra esse padrão está disponível em [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).
- [Padrão de Chave de Manobrista](valet-key.md). Se os recursos de destino não devem estar disponíveis para usuários anônimos, é necessário implementar segurança no repositório que contém o conteúdo estático. Descreve como usar um token ou chave que fornece aos clientes acesso direto e restrito a determinado recurso ou serviço, tal como o serviço de armazenamento hospedado na nuvem.
- [Uma maneira eficiente de implantar um site da Web estático no Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html) no blog da Infosys.
- [Conceitos do Serviço Blob](https://msdn.microsoft.com/library/azure/dd179376.aspx)
