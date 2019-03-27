---
title: Padrão de Hospedagem de Conteúdo Estático
titleSuffix: Cloud Design Patterns
description: Implante conteúdo estático em um serviço de armazenamento baseado em nuvem que pode enviá-lo diretamente para o cliente.
keywords: padrão de design
author: dragon119
ms.date: 01/04/2019
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 719f0221ecc8d52267cba3136eec20dadef30b99
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249301"
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="69e3f-104">Padrão de Hospedagem de Conteúdo Estático</span><span class="sxs-lookup"><span data-stu-id="69e3f-104">Static Content Hosting pattern</span></span>

<span data-ttu-id="69e3f-105">Implante conteúdo estático em um serviço de armazenamento baseado em nuvem que pode enviá-lo diretamente para o cliente.</span><span class="sxs-lookup"><span data-stu-id="69e3f-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="69e3f-106">Isso pode reduzir a necessidade de instâncias de computação potencialmente caras.</span><span class="sxs-lookup"><span data-stu-id="69e3f-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="69e3f-107">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="69e3f-107">Context and problem</span></span>

<span data-ttu-id="69e3f-108">Aplicativos Web geralmente incluem alguns elementos de conteúdo estático.</span><span class="sxs-lookup"><span data-stu-id="69e3f-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="69e3f-109">Este conteúdo estático pode incluir páginas HTML e outros recursos como imagens e documentos que estão disponíveis para o cliente, como parte de uma página HTML (tal como imagens embutidas, folhas de estilo e arquivos JavaScript do lado do cliente) ou como downloads separados (como documentos PDF).</span><span class="sxs-lookup"><span data-stu-id="69e3f-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="69e3f-110">Embora os servidores Web sejam otimizados para renderização dinâmica e cache de saída, eles ainda precisarão lidar com solicitações para fazer o download de conteúdo estático.</span><span class="sxs-lookup"><span data-stu-id="69e3f-110">Although web servers are optimized for dynamic rendering and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="69e3f-111">Isso consome ciclos de processamento que muitas vezes poderiam ser melhor usados em outros locais.</span><span class="sxs-lookup"><span data-stu-id="69e3f-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="69e3f-112">Solução</span><span class="sxs-lookup"><span data-stu-id="69e3f-112">Solution</span></span>

<span data-ttu-id="69e3f-113">Na maioria dos ambientes de hospedagem em nuvem, você pode colocar alguns dos recursos e páginas estáticas do aplicativo em um serviço de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="69e3f-113">In most cloud hosting environments, you can put some of an application's resources and static pages in a storage service.</span></span> <span data-ttu-id="69e3f-114">O serviço de armazenamento pode atender solicitações para esses recursos, reduzindo a carga nos recursos de computação que lidam com outras solicitações da Web.</span><span class="sxs-lookup"><span data-stu-id="69e3f-114">The storage service can serve requests for these resources, reducing load on the compute resources that handle other web requests.</span></span> <span data-ttu-id="69e3f-115">O custo do armazenamento hospedado em nuvem normalmente é muito menor que o de instâncias de computação.</span><span class="sxs-lookup"><span data-stu-id="69e3f-115">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="69e3f-116">Ao hospedar algumas partes de um aplicativo em um serviço de armazenamento, as principais considerações estão relacionadas à implantação do aplicativo e à proteção de recursos que não devem estar disponíveis para os usuários anônimos.</span><span class="sxs-lookup"><span data-stu-id="69e3f-116">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="69e3f-117">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="69e3f-117">Issues and considerations</span></span>

<span data-ttu-id="69e3f-118">Considere os seguintes pontos ao decidir como implementar esse padrão:</span><span class="sxs-lookup"><span data-stu-id="69e3f-118">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="69e3f-119">O serviço de armazenamento hospedado deve expor um ponto de extremidade HTTP que os usuários podem acessar para baixar os recursos estáticos.</span><span class="sxs-lookup"><span data-stu-id="69e3f-119">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="69e3f-120">Alguns serviços de armazenamento também dão suporte a HTTPS, portanto, é possível hospedar recursos nos serviços de armazenamento que exigem SSL.</span><span class="sxs-lookup"><span data-stu-id="69e3f-120">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="69e3f-121">Para obter o máximo de desempenho e disponibilidade, considere usar uma CDN (rede de distribuição de conteúdo) para armazenar em cache o conteúdo do contêiner de armazenamento em vários datacenters ao redor do mundo.</span><span class="sxs-lookup"><span data-stu-id="69e3f-121">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="69e3f-122">No entanto, você provavelmente precisará pagar para usar a CDN.</span><span class="sxs-lookup"><span data-stu-id="69e3f-122">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="69e3f-123">Geralmente, as contas de armazenamento são replicados geograficamente por padrão para oferecer resiliência contra eventos que podem afetar um data center.</span><span class="sxs-lookup"><span data-stu-id="69e3f-123">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="69e3f-124">Isso significa que o endereço IP pode mudar, mas a URL permanecerá a mesma.</span><span class="sxs-lookup"><span data-stu-id="69e3f-124">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="69e3f-125">Quando parte do conteúdo está localizado em uma conta de armazenamento e outras partes do conteúdo estão em uma instância de computação hospedada, é mais difícil implantar e atualizar o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="69e3f-125">When some content is located in a storage account and other content is in a hosted compute instance, it becomes more challenging to deploy and update the application.</span></span> <span data-ttu-id="69e3f-126">Pode ser necessário executar implantações separadas e controlar a versão do aplicativo e do conteúdo para gerenciá-la mais facilmente &mdash; especialmente quando o conteúdo estático inclui arquivos de script ou componentes de interface do usuário.</span><span class="sxs-lookup"><span data-stu-id="69e3f-126">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="69e3f-127">No entanto, se apenas recursos estáticos devem ser atualizados, eles podem simplesmente ser carregados para a conta de armazenamento sem precisar reimplantar o pacote de aplicativos.</span><span class="sxs-lookup"><span data-stu-id="69e3f-127">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="69e3f-128">Os serviços de armazenamento podem não dar suporte ao uso de nomes de domínio personalizados.</span><span class="sxs-lookup"><span data-stu-id="69e3f-128">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="69e3f-129">Nesse caso é necessário especificar a URL completa dos recursos nos links porque eles estarão em um domínio diferente do conteúdo gerado dinamicamente que contém os links.</span><span class="sxs-lookup"><span data-stu-id="69e3f-129">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="69e3f-130">Os contêineres de armazenamento devem ser configurados para acesso de leitura público, mas é essencial garantir que eles não estejam configurados para acesso de gravação público para impedir que os usuários possam carregar o conteúdo.</span><span class="sxs-lookup"><span data-stu-id="69e3f-130">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span>

- <span data-ttu-id="69e3f-131">Considere usar uma chave limitada ou token para controlar o acesso aos recursos que não devem estar disponíveis anonimamente.</span><span class="sxs-lookup"><span data-stu-id="69e3f-131">Consider using a valet key or token to control access to resources that shouldn't be available anonymously.</span></span> <span data-ttu-id="69e3f-132">Confira o [padrão de Chave Limitada](./valet-key.md) para ver mais informações.</span><span class="sxs-lookup"><span data-stu-id="69e3f-132">See the [Valet Key pattern](./valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="69e3f-133">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="69e3f-133">When to use this pattern</span></span>

<span data-ttu-id="69e3f-134">Esse padrão é útil para:</span><span class="sxs-lookup"><span data-stu-id="69e3f-134">This pattern is useful for:</span></span>

- <span data-ttu-id="69e3f-135">Minimizar o custo de hospedagem de sites e aplicativos que contêm alguns recursos estáticos.</span><span class="sxs-lookup"><span data-stu-id="69e3f-135">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="69e3f-136">Minimizar o custo de hospedagem de sites compostos somente por conteúdo e recursos estáticos.</span><span class="sxs-lookup"><span data-stu-id="69e3f-136">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="69e3f-137">Dependendo das funcionalidades do sistema de armazenamento do provedor de hospedagem, seria possível hospedar inteiramente um site totalmente estático em uma conta de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="69e3f-137">Depending on the capabilities of the hosting provider's storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="69e3f-138">Expor os recursos e conteúdo estático para aplicativos em execução em outros ambientes de hospedagem ou nos servidores locais.</span><span class="sxs-lookup"><span data-stu-id="69e3f-138">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="69e3f-139">Localizar conteúdo em mais de uma área geográfica usando uma rede de distribuição de conteúdo que armazena em cache o conteúdo da conta de armazenamento em vários datacenters ao redor do mundo.</span><span class="sxs-lookup"><span data-stu-id="69e3f-139">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="69e3f-140">Monitorar os custos e uso de largura de banda.</span><span class="sxs-lookup"><span data-stu-id="69e3f-140">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="69e3f-141">Usar uma conta de armazenamento separada para parte ou todo o conteúdo estático permite que os custos sejam separados mais facilmente dos custos de hospedagem e de tempo de execução.</span><span class="sxs-lookup"><span data-stu-id="69e3f-141">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="69e3f-142">Esse padrão pode não ser útil nas seguintes situações:</span><span class="sxs-lookup"><span data-stu-id="69e3f-142">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="69e3f-143">O aplicativo precisa executar processamento no conteúdo estático antes de distribuí-lo para o cliente.</span><span class="sxs-lookup"><span data-stu-id="69e3f-143">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="69e3f-144">Por exemplo, pode ser necessário adicionar um carimbo de data/hora a um documento.</span><span class="sxs-lookup"><span data-stu-id="69e3f-144">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="69e3f-145">O volume do conteúdo estático é muito pequeno.</span><span class="sxs-lookup"><span data-stu-id="69e3f-145">The volume of static content is very small.</span></span> <span data-ttu-id="69e3f-146">A sobrecarga de recuperar o conteúdo do armazenamento separado pode superar o benefício de custo de separá-lo do recurso de computação.</span><span class="sxs-lookup"><span data-stu-id="69e3f-146">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="69e3f-147">Exemplo</span><span class="sxs-lookup"><span data-stu-id="69e3f-147">Example</span></span>

<span data-ttu-id="69e3f-148">O Armazenamento do Azure dá suporte ao fornecimento de conteúdo estático diretamente de um contêiner de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="69e3f-148">Azure Storage supports serving static content directly from a storage container.</span></span> <span data-ttu-id="69e3f-149">Os arquivos são atendidos por meio de solicitações de acesso anônimo.</span><span class="sxs-lookup"><span data-stu-id="69e3f-149">Files are served through anonymous access requests.</span></span> <span data-ttu-id="69e3f-150">Por padrão, os arquivos têm uma URL em um subdomínio `core.windows.net`, como `https://contoso.z4.web.core.windows.net/image.png`.</span><span class="sxs-lookup"><span data-stu-id="69e3f-150">By default, files have a URL in a subdomain of `core.windows.net`, such as `https://contoso.z4.web.core.windows.net/image.png`.</span></span> <span data-ttu-id="69e3f-151">Você pode configurar um nome de domínio personalizado e usar a CDN do Azure para acessar os arquivos por HTTPS.</span><span class="sxs-lookup"><span data-stu-id="69e3f-151">You can configure a custom domain name, and use Azure CDN to access the files over HTTPS.</span></span> <span data-ttu-id="69e3f-152">Para mais informações, confira [Hospedagem de site estático no Armazenamento do Azure](/azure/storage/blobs/storage-blob-static-website).</span><span class="sxs-lookup"><span data-stu-id="69e3f-152">For more information, see [Static website hosting in Azure Storage](/azure/storage/blobs/storage-blob-static-website).</span></span>

![Distribuição de partes estáticas de um aplicativo diretamente de um serviço de armazenamento](./_images/static-content-hosting-pattern.png)

<span data-ttu-id="69e3f-154">A hospedagem de site estático torna os arquivos disponíveis para acesso anônimo.</span><span class="sxs-lookup"><span data-stu-id="69e3f-154">Static website hosting makes the files available for anonymous access.</span></span> <span data-ttu-id="69e3f-155">Se precisar controlar quem pode acessar os arquivos, você pode armazenar arquivos no armazenamento de blogs do Azure e, em seguida, gerar [assinaturas de acesso compartilhado](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) para limitar o acesso.</span><span class="sxs-lookup"><span data-stu-id="69e3f-155">If you need to control who can access the files, you can store files in Azure blob storage and then generate [shared access signatures](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) to limit access.</span></span>

<span data-ttu-id="69e3f-156">Os links nas páginas distribuídas para o cliente devem especificar a URL completa do recurso.</span><span class="sxs-lookup"><span data-stu-id="69e3f-156">The links in the pages delivered to the client must specify the full URL of the resource.</span></span> <span data-ttu-id="69e3f-157">Se o recurso estiver protegido por uma chave limitada, como uma assinatura de acesso compartilhado do Azure, essa assinatura deverá ser incluída na URL.</span><span class="sxs-lookup"><span data-stu-id="69e3f-157">If the resource is protected with a valet key, such as a shared access signature, this signature must be included in the URL.</span></span>

<span data-ttu-id="69e3f-158">Um aplicativo de exemplo que demonstra o uso do armazenamento externo para recursos estáticos está disponível no [GitHub][sample-app].</span><span class="sxs-lookup"><span data-stu-id="69e3f-158">A sample application that demonstrates using external storage for static resources is available on [GitHub][sample-app].</span></span> <span data-ttu-id="69e3f-159">Este exemplo usa arquivos de configuração que especificam a conta de armazenamento e o contêiner que retém o conteúdo estático.</span><span class="sxs-lookup"><span data-stu-id="69e3f-159">This sample uses configuration files to specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="69e3f-160">A classe `Settings` no arquivo Settings.cs do projeto StaticContentHosting.Web contém métodos para extrair esses valores e criar um valor de cadeia de caracteres que contém a URL de contêiner da conta de armazenamento de nuvem.</span><span class="sxs-lookup"><span data-stu-id="69e3f-160">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

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

<span data-ttu-id="69e3f-161">A classe `StaticContentUrlHtmlHelper` no arquivo StaticContentUrlHtmlHelper.cs expõe um método chamado `StaticContentUrl` que gera uma URL que contém o caminho para a conta de armazenamento de nuvem se a URL passada para ele começar com o caractere de caminho raiz do ASP.NET (~).</span><span class="sxs-lookup"><span data-stu-id="69e3f-161">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

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

<span data-ttu-id="69e3f-162">O arquivo Index.cshtml na pasta Views\Home contém um elemento de imagem que usa o método `StaticContentUrl` para criar a URL para o seu atributo `src`.</span><span class="sxs-lookup"><span data-stu-id="69e3f-162">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="69e3f-163">Diretrizes e padrões relacionados</span><span class="sxs-lookup"><span data-stu-id="69e3f-163">Related patterns and guidance</span></span>

- <span data-ttu-id="69e3f-164">[Exemplo de hospedagem de conteúdo estático][sample-app].</span><span class="sxs-lookup"><span data-stu-id="69e3f-164">[Static Content Hosting sample][sample-app].</span></span> <span data-ttu-id="69e3f-165">Um aplicativo de exemplo que demonstra esse padrão.</span><span class="sxs-lookup"><span data-stu-id="69e3f-165">A sample application that demonstrates this pattern.</span></span>
- <span data-ttu-id="69e3f-166">[Padrão de Chave de Manobrista](./valet-key.md).</span><span class="sxs-lookup"><span data-stu-id="69e3f-166">[Valet Key pattern](./valet-key.md).</span></span> <span data-ttu-id="69e3f-167">Se os recursos de destino não estiverem disponíveis para usuários anônimos, use esse padrão para restringir o acesso direto.</span><span class="sxs-lookup"><span data-stu-id="69e3f-167">If the target resources aren't supposed to be available to anonymous users, use this pattern to restrict direct access.</span></span>
- <span data-ttu-id="69e3f-168">[Aplicativo Web sem servidor no Azure](../reference-architectures/serverless/web-app.md).</span><span class="sxs-lookup"><span data-stu-id="69e3f-168">[Serverless web application on Azure](../reference-architectures/serverless/web-app.md).</span></span> <span data-ttu-id="69e3f-169">Uma arquitetura de referência que usa a hospedagem de site estático com o Azure Functions para implementar um aplicativo Web sem servidor.</span><span class="sxs-lookup"><span data-stu-id="69e3f-169">A reference architecture that uses static website hosting with Azure Functions to implement a serverless web app.</span></span>

[sample-app]: https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting
