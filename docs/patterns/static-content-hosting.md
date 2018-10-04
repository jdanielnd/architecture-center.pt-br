---
title: Hospedagem de Conteúdo Estático
description: Implante conteúdo estático em um serviço de armazenamento baseado em nuvem que pode enviá-lo diretamente para o cliente.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: 450d0c4c08098c1ba48e4c0dac3d058a46e3709b
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428203"
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="2a9c3-104">Padrão de Hospedagem de Conteúdo Estático</span><span class="sxs-lookup"><span data-stu-id="2a9c3-104">Static Content Hosting pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="2a9c3-105">Implante conteúdo estático em um serviço de armazenamento baseado em nuvem que pode enviá-lo diretamente para o cliente.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="2a9c3-106">Isso pode reduzir a necessidade de instâncias de computação potencialmente caras.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="2a9c3-107">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="2a9c3-107">Context and problem</span></span>

<span data-ttu-id="2a9c3-108">Aplicativos Web geralmente incluem alguns elementos de conteúdo estático.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="2a9c3-109">Este conteúdo estático pode incluir páginas HTML e outros recursos como imagens e documentos que estão disponíveis para o cliente, como parte de uma página HTML (tal como imagens embutidas, folhas de estilo e arquivos JavaScript do lado do cliente) ou como downloads separados (como documentos PDF).</span><span class="sxs-lookup"><span data-stu-id="2a9c3-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="2a9c3-110">Embora os servidores Web estejam bem ajustados para otimizar as solicitações por meio da execução de código de página dinâmica e cache de saída eficientes, eles ainda precisarão lidar com as solicitações para baixar conteúdo estático.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-110">Although web servers are well tuned to optimize requests through efficient dynamic page code execution and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="2a9c3-111">Isso consome ciclos de processamento que muitas vezes poderiam ser melhor usados em outros locais.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="2a9c3-112">Solução</span><span class="sxs-lookup"><span data-stu-id="2a9c3-112">Solution</span></span>

<span data-ttu-id="2a9c3-113">Na maioria dos ambientes de hospedagem em nuvem, é possível minimizar a necessidade de instâncias de computação (por exemplo, usar uma instância menor ou menos instâncias) colocando algumas das páginas estáticas e recursos de um aplicativo em um serviço de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-113">In most cloud hosting environments it's possible to minimize the need for compute instances (for example, use a smaller instance or fewer instances), by locating some of an application’s resources and static pages in a storage service.</span></span> <span data-ttu-id="2a9c3-114">O custo do armazenamento hospedado em nuvem normalmente é muito menor que o de instâncias de computação.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-114">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="2a9c3-115">Ao hospedar algumas partes de um aplicativo em um serviço de armazenamento, as principais considerações estão relacionadas à implantação do aplicativo e à proteção de recursos que não devem estar disponíveis para os usuários anônimos.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-115">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="2a9c3-116">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="2a9c3-116">Issues and considerations</span></span>

<span data-ttu-id="2a9c3-117">Considere os seguintes pontos ao decidir como implementar esse padrão:</span><span class="sxs-lookup"><span data-stu-id="2a9c3-117">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="2a9c3-118">O serviço de armazenamento hospedado deve expor um ponto de extremidade HTTP que os usuários podem acessar para baixar os recursos estáticos.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-118">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="2a9c3-119">Alguns serviços de armazenamento também dão suporte a HTTPS, portanto, é possível hospedar recursos nos serviços de armazenamento que exigem SSL.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-119">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="2a9c3-120">Para obter o máximo de desempenho e disponibilidade, considere usar uma CDN (rede de distribuição de conteúdo) para armazenar em cache o conteúdo do contêiner de armazenamento em vários datacenters ao redor do mundo.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-120">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="2a9c3-121">No entanto, você provavelmente precisará pagar para usar a CDN.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-121">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="2a9c3-122">Geralmente, as contas de armazenamento são replicados geograficamente por padrão para oferecer resiliência contra eventos que podem afetar um data center.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-122">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="2a9c3-123">Isso significa que o endereço IP pode mudar, mas a URL permanecerá a mesma.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-123">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="2a9c3-124">Quando parte do conteúdo está localizado em uma conta de armazenamento e outras partes do conteúdo estão em uma instância de computação hospedada, é mais difícil de implantar um aplicativo e atualizá-lo.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-124">When some content is located in a storage account and other content is in a hosted compute instance it becomes more challenging to deploy an application and to update it.</span></span> <span data-ttu-id="2a9c3-125">Pode ser necessário executar implantações separadas e controlar a versão do aplicativo e do conteúdo para gerenciá-la mais facilmente &mdash; especialmente quando o conteúdo estático inclui arquivos de script ou componentes de interface do usuário.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-125">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="2a9c3-126">No entanto, se apenas recursos estáticos devem ser atualizados, eles podem simplesmente ser carregados para a conta de armazenamento sem precisar reimplantar o pacote de aplicativos.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-126">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="2a9c3-127">Os serviços de armazenamento podem não dar suporte ao uso de nomes de domínio personalizados.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-127">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="2a9c3-128">Nesse caso é necessário especificar a URL completa dos recursos nos links porque eles estarão em um domínio diferente do conteúdo gerado dinamicamente que contém os links.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-128">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="2a9c3-129">Os contêineres de armazenamento devem ser configurados para acesso de leitura público, mas é essencial garantir que eles não estejam configurados para acesso de gravação público para impedir que os usuários possam carregar o conteúdo.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-129">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span> <span data-ttu-id="2a9c3-130">Considere usar uma chave de manobrista ou token para controlar o acesso aos recursos que não devem estar disponíveis anonimamente &mdash; consulte o [Padrão de Valet Key](valet-key.md) para obter mais informações.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-130">Consider using a valet key or token to control access to resources that shouldn't be available anonymously&mdash;see the [Valet Key pattern](valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="2a9c3-131">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="2a9c3-131">When to use this pattern</span></span>

<span data-ttu-id="2a9c3-132">Esse padrão é útil para:</span><span class="sxs-lookup"><span data-stu-id="2a9c3-132">This pattern is useful for:</span></span>

- <span data-ttu-id="2a9c3-133">Minimizar o custo de hospedagem de sites e aplicativos que contêm alguns recursos estáticos.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-133">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="2a9c3-134">Minimizar o custo de hospedagem de sites compostos somente por conteúdo e recursos estáticos.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-134">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="2a9c3-135">Dependendo das funcionalidades do sistema de armazenamento do provedor de hospedagem, seria possível hospedar inteiramente um site totalmente estático em uma conta de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-135">Depending on the capabilities of the hosting provider’s storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="2a9c3-136">Expor os recursos e conteúdo estático para aplicativos em execução em outros ambientes de hospedagem ou nos servidores locais.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-136">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="2a9c3-137">Localizar conteúdo em mais de uma área geográfica usando uma rede de distribuição de conteúdo que armazena em cache o conteúdo da conta de armazenamento em vários datacenters ao redor do mundo.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-137">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="2a9c3-138">Monitorar os custos e uso de largura de banda.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-138">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="2a9c3-139">Usar uma conta de armazenamento separada para parte ou todo o conteúdo estático permite que os custos sejam separados mais facilmente dos custos de hospedagem e de tempo de execução.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-139">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="2a9c3-140">Esse padrão pode não ser útil nas seguintes situações:</span><span class="sxs-lookup"><span data-stu-id="2a9c3-140">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="2a9c3-141">O aplicativo precisa executar processamento no conteúdo estático antes de distribuí-lo para o cliente.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-141">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="2a9c3-142">Por exemplo, pode ser necessário adicionar um carimbo de data/hora a um documento.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-142">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="2a9c3-143">O volume do conteúdo estático é muito pequeno.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-143">The volume of static content is very small.</span></span> <span data-ttu-id="2a9c3-144">A sobrecarga de recuperar o conteúdo do armazenamento separado pode superar o benefício de custo de separá-lo do recurso de computação.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-144">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="2a9c3-145">Exemplo</span><span class="sxs-lookup"><span data-stu-id="2a9c3-145">Example</span></span>

<span data-ttu-id="2a9c3-146">O conteúdo estático localizado no armazenamento de Blobs do Azure pode ser acessado diretamente por um navegador da Web.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-146">Static content located in Azure Blob storage can be accessed directly by a web browser.</span></span> <span data-ttu-id="2a9c3-147">O Azure fornece uma interface baseada em HTTP para o armazenamento que pode ser exposto publicamente para os clientes.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-147">Azure provides an HTTP-based interface over storage that can be publicly exposed to clients.</span></span> <span data-ttu-id="2a9c3-148">Por exemplo, o conteúdo em um contêiner de armazenamento de Blobs do Azure é exposto usando uma URL com o seguinte formato:</span><span class="sxs-lookup"><span data-stu-id="2a9c3-148">For example, content in an Azure Blob storage container is exposed using a URL with the following form:</span></span>

`https://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


<span data-ttu-id="2a9c3-149">Ao carregar o conteúdo, é necessário criar um ou mais contêineres de blob para conter os arquivos e documentos.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-149">When uploading the content it's necessary to create one or more blob containers to hold the files and documents.</span></span> <span data-ttu-id="2a9c3-150">Observe que a permissão padrão para um novo contêiner é Privado e você deve alterar isso para Público para permitir que os clientes acessem o conteúdo.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-150">Note that the default permission for a new container is Private, and you must change this to Public to allow clients to access the contents.</span></span> <span data-ttu-id="2a9c3-151">Se for necessário proteger o conteúdo do acesso anônimo, você poderá implementar o [padrão de Chave de Manobrista](valet-key.md) para que os usuários precisem apresentar um token válido para baixar os recursos.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-151">If it's necessary to protect the content from anonymous access, you can implement the [Valet Key pattern](valet-key.md) so users must present a valid token to download the resources.</span></span>

> <span data-ttu-id="2a9c3-152">Os [Conceitos do Serviço Blob](https://msdn.microsoft.com/library/azure/dd179376.aspx) trazem informações sobre o armazenamento de blobs e as maneiras para acessá-los e usá-los.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-152">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) has information about blob storage, and the ways that you can access and use it.</span></span>

<span data-ttu-id="2a9c3-153">Os links em cada página especificam a URL do recurso e o cliente irá acessá-la diretamente do serviço de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-153">The links in each page will specify the URL of the resource and the client will access it directly from the storage service.</span></span> <span data-ttu-id="2a9c3-154">A figura ilustra a distribuição de partes estáticas de um aplicativo diretamente de um serviço de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-154">The figure illustrates delivering static parts of an application directly from a storage service.</span></span>

![Figura 1 – Distribuição de partes estáticas de um aplicativo diretamente de um serviço de armazenamento](./_images/static-content-hosting-pattern.png)


<span data-ttu-id="2a9c3-156">Os links nas páginas distribuídas para o cliente devem especificar a URL completa do contêiner e recursos de blobs.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-156">The links in the pages delivered to the client must specify the full URL of the blob container and resource.</span></span> <span data-ttu-id="2a9c3-157">Por exemplo, uma página que contém um link para uma imagem em um contêiner público pode conter o seguinte HTML.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-157">For example, a page that contains a link to an image in a public container might contain the following HTML.</span></span>

```html
<img src="https://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> <span data-ttu-id="2a9c3-158">Se os recursos forem protegidos usando uma chave de manobrista, como uma assinatura de acesso compartilhado do Azure, essa assinatura deverá ser incluída nas URLs nos links.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-158">If the resources are protected by using a valet key, such as an Azure shared access signature, this signature must be included in the URLs in the links.</span></span>

<span data-ttu-id="2a9c3-159">Uma solução chamada StaticContentHosting que demonstra o uso do armazenamento externo para recursos estáticos está disponível no [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span><span class="sxs-lookup"><span data-stu-id="2a9c3-159">A solution named StaticContentHosting that demonstrates using external storage for static resources is available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span> <span data-ttu-id="2a9c3-160">O projeto StaticContentHosting.Cloud contém arquivos de configuração que especificam a conta de armazenamento e o contêiner que retém o conteúdo estático.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-160">The StaticContentHosting.Cloud project contains configuration files that specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="2a9c3-161">A classe `Settings` no arquivo Settings.cs do projeto StaticContentHosting.Web contém métodos para extrair esses valores e criar um valor de cadeia de caracteres que contém a URL de contêiner da conta de armazenamento de nuvem.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-161">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

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

<span data-ttu-id="2a9c3-162">A classe `StaticContentUrlHtmlHelper` no arquivo StaticContentUrlHtmlHelper.cs expõe um método chamado `StaticContentUrl` que gera uma URL que contém o caminho para a conta de armazenamento de nuvem se a URL passada para ele começar com o caractere de caminho raiz do ASP.NET (~).</span><span class="sxs-lookup"><span data-stu-id="2a9c3-162">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

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

<span data-ttu-id="2a9c3-163">O arquivo Index.cshtml na pasta Views\Home contém um elemento de imagem que usa o método `StaticContentUrl` para criar a URL para o seu atributo `src`.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-163">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="2a9c3-164">Diretrizes e padrões relacionados</span><span class="sxs-lookup"><span data-stu-id="2a9c3-164">Related patterns and guidance</span></span>

- <span data-ttu-id="2a9c3-165">Um exemplo que demonstra esse padrão está disponível em [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span><span class="sxs-lookup"><span data-stu-id="2a9c3-165">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span>
- <span data-ttu-id="2a9c3-166">[Padrão de Chave de Manobrista](valet-key.md).</span><span class="sxs-lookup"><span data-stu-id="2a9c3-166">[Valet Key pattern](valet-key.md).</span></span> <span data-ttu-id="2a9c3-167">Se os recursos de destino não devem estar disponíveis para usuários anônimos, é necessário implementar segurança no repositório que contém o conteúdo estático.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-167">If the target resources aren't supposed to be available to anonymous users it's necessary to implement security over the store that holds the static content.</span></span> <span data-ttu-id="2a9c3-168">Descreve como usar um token ou chave que fornece aos clientes acesso direto e restrito a determinado recurso ou serviço, tal como o serviço de armazenamento hospedado na nuvem.</span><span class="sxs-lookup"><span data-stu-id="2a9c3-168">Describes how to use a token or key that provides clients with restricted direct access to a specific resource or service such as a cloud-hosted storage service.</span></span>
- [<span data-ttu-id="2a9c3-169">Conceitos do Serviço Blob</span><span class="sxs-lookup"><span data-stu-id="2a9c3-169">Blob Service Concepts</span></span>](https://msdn.microsoft.com/library/azure/dd179376.aspx)
