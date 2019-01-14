---
title: Federar com o AD FS de um cliente
description: Como federar com o AD FS de um cliente em um aplicativo multilocatário.
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: token-cache
pnp.series.next: client-assertion
ms.openlocfilehash: 27fad1aab8d359346353cc031a2e8d8746294818
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113545"
---
# <a name="federate-with-a-customers-ad-fs"></a><span data-ttu-id="539f4-103">Federar com o AD FS de um cliente</span><span class="sxs-lookup"><span data-stu-id="539f4-103">Federate with a customer's AD FS</span></span>

<span data-ttu-id="539f4-104">Este artigo descreve como um aplicativo SaaS multilocatário pode oferecer suporte à autenticação por meio do AD FS (Serviços de Federação do Active Directory) para federar com o AD FS de um cliente.</span><span class="sxs-lookup"><span data-stu-id="539f4-104">This article describes how a multi-tenant SaaS application can support authentication via Active Directory Federation Services (AD FS), in order to federate with a customer's AD FS.</span></span>

## <a name="overview"></a><span data-ttu-id="539f4-105">Visão geral</span><span class="sxs-lookup"><span data-stu-id="539f4-105">Overview</span></span>

<span data-ttu-id="539f4-106">O Azure AD (Azure Active Directory) facilita a entrada de usuários a partir de locatários do Azure AD, incluindo clientes do Office365 e do Dynamics CRM Online.</span><span class="sxs-lookup"><span data-stu-id="539f4-106">Azure Active Directory (Azure AD) makes it easy to sign in users from Azure AD tenants, including Office365 and Dynamics CRM Online customers.</span></span> <span data-ttu-id="539f4-107">Mas e quanto aos clientes que usam o Active Directory local em uma intranet corporativa?</span><span class="sxs-lookup"><span data-stu-id="539f4-107">But what about customers who use on-premise Active Directory on a corporate intranet?</span></span>

<span data-ttu-id="539f4-108">Uma opção seria esses clientes sincronizarem o AD local com o Azure AD, usando o [Azure AD Connect].</span><span class="sxs-lookup"><span data-stu-id="539f4-108">One option is for these customers to sync their on-premise AD with Azure AD, using [Azure AD Connect].</span></span> <span data-ttu-id="539f4-109">No entanto, talvez alguns clientes não consigam usar essa abordagem devido à política de TI corporativa ou por outros motivos.</span><span class="sxs-lookup"><span data-stu-id="539f4-109">However, some customers may be unable to use this approach, due to corporate IT policy or other reasons.</span></span> <span data-ttu-id="539f4-110">Nesse caso, outra opção é a federação por meio do AD FS (Serviços de Federação do Active Directory).</span><span class="sxs-lookup"><span data-stu-id="539f4-110">In that case, another option is to federate through Active Directory Federation Services (AD FS).</span></span>

<span data-ttu-id="539f4-111">Para habilitar este cenário:</span><span class="sxs-lookup"><span data-stu-id="539f4-111">To enable this scenario:</span></span>

* <span data-ttu-id="539f4-112">O cliente deve ter um farm do AD FS voltado para a Internet.</span><span class="sxs-lookup"><span data-stu-id="539f4-112">The customer must have an Internet-facing AD FS farm.</span></span>
* <span data-ttu-id="539f4-113">O provedor de SaaS implanta seu próprio farm do AD FS.</span><span class="sxs-lookup"><span data-stu-id="539f4-113">The SaaS provider deploys their own AD FS farm.</span></span>
* <span data-ttu-id="539f4-114">O cliente e o provedor de SaaS devem configurar a [confiança de federação].</span><span class="sxs-lookup"><span data-stu-id="539f4-114">The customer and the SaaS provider must set up [federation trust].</span></span> <span data-ttu-id="539f4-115">Esse é um processo manual.</span><span class="sxs-lookup"><span data-stu-id="539f4-115">This is a manual process.</span></span>

<span data-ttu-id="539f4-116">Há três funções principais na relação de confiança:</span><span class="sxs-lookup"><span data-stu-id="539f4-116">There are three main roles in the trust relation:</span></span>

* <span data-ttu-id="539f4-117">O AD FS do cliente é o [parceiro de conta], responsável por autenticar usuários do AD do cliente e criar tokens de segurança com declarações de usuário.</span><span class="sxs-lookup"><span data-stu-id="539f4-117">The customer's AD FS is the [account partner], responsible for authenticating users from the customer's AD, and creating security tokens with user claims.</span></span>
* <span data-ttu-id="539f4-118">O AD FS do provedor de SaaS é o [parceiro de recurso], que confia no parceiro de conta e recebe as declarações de usuário.</span><span class="sxs-lookup"><span data-stu-id="539f4-118">The SaaS provider's AD FS is the [resource partner], which trusts the account partner and receives the user claims.</span></span>
* <span data-ttu-id="539f4-119">O aplicativo é configurado como uma RP (terceira parte confiável) no AD FS do provedor de SaaS.</span><span class="sxs-lookup"><span data-stu-id="539f4-119">The application is configured as a relying party (RP) in the SaaS provider's AD FS.</span></span>
  
  ![confiança de federação](./images/federation-trust.png)

> [!NOTE]
> <span data-ttu-id="539f4-121">Neste artigo, supomos que o aplicativo use OpenID Connect como o protocolo de autenticação.</span><span class="sxs-lookup"><span data-stu-id="539f4-121">In this article, we assume the application uses OpenID connect as the authentication protocol.</span></span> <span data-ttu-id="539f4-122">Outra opção é usar o WS-Federation.</span><span class="sxs-lookup"><span data-stu-id="539f4-122">Another option is to use WS-Federation.</span></span>
>
> <span data-ttu-id="539f4-123">Para o OpenID Connect, o provedor de SaaS deve usar o AD FS 2016, em execução no Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="539f4-123">For OpenID Connect, the SaaS provider must use AD FS 2016, running in Windows Server 2016.</span></span> <span data-ttu-id="539f4-124">O AD FS 3.0 não oferece suporte para o OpenID Connect.</span><span class="sxs-lookup"><span data-stu-id="539f4-124">AD FS 3.0 does not support OpenID Connect.</span></span>
>
> <span data-ttu-id="539f4-125">O ASP.NET Core não inclui suporte nativo para a especificação Web Services Federation.</span><span class="sxs-lookup"><span data-stu-id="539f4-125">ASP.NET Core does not include out-of-the-box support for WS-Federation.</span></span>
>
>

<span data-ttu-id="539f4-126">Para obter um exemplo do uso da especificação Web Services Federation com ASP.NET 4, veja o [exemplo active-directory-dotnet-webapp-wsfederation][active-directory-dotnet-webapp-wsfederation].</span><span class="sxs-lookup"><span data-stu-id="539f4-126">For an example of using WS-Federation with ASP.NET 4, see the [active-directory-dotnet-webapp-wsfederation sample][active-directory-dotnet-webapp-wsfederation].</span></span>

## <a name="authentication-flow"></a><span data-ttu-id="539f4-127">Fluxo de autenticação</span><span class="sxs-lookup"><span data-stu-id="539f4-127">Authentication flow</span></span>

1. <span data-ttu-id="539f4-128">Quando o usuário clicar em "entrar", o aplicativo será redirecionado para um ponto de extremidade do OpenID Connect sobre o AD FS do provedor de SaaS.</span><span class="sxs-lookup"><span data-stu-id="539f4-128">When the user clicks "sign in", the application redirects to an OpenID Connect endpoint on the SaaS provider's AD FS.</span></span>
2. <span data-ttu-id="539f4-129">O usuário insere seu nome de usuário da organização ("`alice@corp.contoso.com`").</span><span class="sxs-lookup"><span data-stu-id="539f4-129">The user enters his or her organizational user name ("`alice@corp.contoso.com`").</span></span> <span data-ttu-id="539f4-130">O AD FS usa a descoberta de realm inicial para redirecionar o AD FS do cliente, no qual o usuário insere suas credenciais.</span><span class="sxs-lookup"><span data-stu-id="539f4-130">AD FS uses home realm discovery to redirect to the customer's AD FS, where the user enters their credentials.</span></span>
3. <span data-ttu-id="539f4-131">O AD FS do cliente envia declarações de usuário para o AD FS do provedor de SaaS, usando o WF-Federation (ou SAML).</span><span class="sxs-lookup"><span data-stu-id="539f4-131">The customer's AD FS sends user claims to the SaaS provider's AD FS, using WF-Federation (or SAML).</span></span>
4. <span data-ttu-id="539f4-132">Fluxo de declarações do AD FS para o aplicativo, usando o OpenID Connect.</span><span class="sxs-lookup"><span data-stu-id="539f4-132">Claims flow from AD FS to the app, using OpenID Connect.</span></span> <span data-ttu-id="539f4-133">Isso exige uma transição de protocolo do WS-Federation.</span><span class="sxs-lookup"><span data-stu-id="539f4-133">This requires a protocol transition from WS-Federation.</span></span>

## <a name="limitations"></a><span data-ttu-id="539f4-134">Limitações</span><span class="sxs-lookup"><span data-stu-id="539f4-134">Limitations</span></span>

<span data-ttu-id="539f4-135">Por padrão, o aplicativo de terceira parte recebe apenas um conjunto fixo de declarações disponíveis no id_token, mostrado na tabela a seguir.</span><span class="sxs-lookup"><span data-stu-id="539f4-135">By default, the relying party application receives only a fixed set of claims available in the id_token, shown in the following table.</span></span> <span data-ttu-id="539f4-136">Com o AD FS 2016, você pode personalizar o id_token em cenários de OpenID Connect.</span><span class="sxs-lookup"><span data-stu-id="539f4-136">With AD FS 2016, you can customize the id_token in OpenID Connect scenarios.</span></span> <span data-ttu-id="539f4-137">Para saber mais, veja [Tokens de ID personalizados do AD FS](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016).</span><span class="sxs-lookup"><span data-stu-id="539f4-137">For more information, see [Custom ID Tokens in AD FS](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016).</span></span>

| <span data-ttu-id="539f4-138">Declaração</span><span class="sxs-lookup"><span data-stu-id="539f4-138">Claim</span></span> | <span data-ttu-id="539f4-139">DESCRIÇÃO</span><span class="sxs-lookup"><span data-stu-id="539f4-139">Description</span></span> |
| --- | --- |
| <span data-ttu-id="539f4-140">aud</span><span class="sxs-lookup"><span data-stu-id="539f4-140">aud</span></span> |<span data-ttu-id="539f4-141">Público-alvo.</span><span class="sxs-lookup"><span data-stu-id="539f4-141">Audience.</span></span> <span data-ttu-id="539f4-142">O aplicativo para o qual as declarações foram emitidas.</span><span class="sxs-lookup"><span data-stu-id="539f4-142">The application for which the claims were issued.</span></span> |
| <span data-ttu-id="539f4-143">authenticationinstant</span><span class="sxs-lookup"><span data-stu-id="539f4-143">authenticationinstant</span></span> |<span data-ttu-id="539f4-144">[Instante da autenticação].</span><span class="sxs-lookup"><span data-stu-id="539f4-144">[Authentication instant].</span></span> <span data-ttu-id="539f4-145">A hora em que a autenticação ocorreu.</span><span class="sxs-lookup"><span data-stu-id="539f4-145">The time at which authentication occurred.</span></span> |
| <span data-ttu-id="539f4-146">c_hash</span><span class="sxs-lookup"><span data-stu-id="539f4-146">c_hash</span></span> |<span data-ttu-id="539f4-147">Valor hash do código.</span><span class="sxs-lookup"><span data-stu-id="539f4-147">Code hash value.</span></span> <span data-ttu-id="539f4-148">Este é um hash do conteúdo do token.</span><span class="sxs-lookup"><span data-stu-id="539f4-148">This is a hash of the token contents.</span></span> |
| <span data-ttu-id="539f4-149">exp</span><span class="sxs-lookup"><span data-stu-id="539f4-149">exp</span></span> |<span data-ttu-id="539f4-150">[Hora de expiração].</span><span class="sxs-lookup"><span data-stu-id="539f4-150">[Expiration time].</span></span> <span data-ttu-id="539f4-151">O tempo após o qual o token não será mais aceito.</span><span class="sxs-lookup"><span data-stu-id="539f4-151">The time after which the token will no longer be accepted.</span></span> |
| <span data-ttu-id="539f4-152">iat</span><span class="sxs-lookup"><span data-stu-id="539f4-152">iat</span></span> |<span data-ttu-id="539f4-153">Emitido em.</span><span class="sxs-lookup"><span data-stu-id="539f4-153">Issued at.</span></span> <span data-ttu-id="539f4-154">A hora em que o token foi emitido.</span><span class="sxs-lookup"><span data-stu-id="539f4-154">The time when the token was issued.</span></span> |
| <span data-ttu-id="539f4-155">iss</span><span class="sxs-lookup"><span data-stu-id="539f4-155">iss</span></span> |<span data-ttu-id="539f4-156">Emissor.</span><span class="sxs-lookup"><span data-stu-id="539f4-156">Issuer.</span></span> <span data-ttu-id="539f4-157">O valor dessa declaração sempre será o AD FS do parceiro de recurso.</span><span class="sxs-lookup"><span data-stu-id="539f4-157">The value of this claim is always the resource partner's AD FS.</span></span> |
| <span data-ttu-id="539f4-158">Nome</span><span class="sxs-lookup"><span data-stu-id="539f4-158">name</span></span> |<span data-ttu-id="539f4-159">Nome de usuário.</span><span class="sxs-lookup"><span data-stu-id="539f4-159">User name.</span></span> <span data-ttu-id="539f4-160">Exemplo: `john@corp.fabrikam.com`</span><span class="sxs-lookup"><span data-stu-id="539f4-160">Example: `john@corp.fabrikam.com`</span></span> |
| <span data-ttu-id="539f4-161">nameidentifier</span><span class="sxs-lookup"><span data-stu-id="539f4-161">nameidentifier</span></span> |<span data-ttu-id="539f4-162">[Identificador de nome].</span><span class="sxs-lookup"><span data-stu-id="539f4-162">[Name identifier].</span></span> <span data-ttu-id="539f4-163">O identificador d o nome da entidade para a qual o token foi emitido.</span><span class="sxs-lookup"><span data-stu-id="539f4-163">The identifier for the name of the entity for which the token was issued.</span></span> |
| <span data-ttu-id="539f4-164">nonce</span><span class="sxs-lookup"><span data-stu-id="539f4-164">nonce</span></span> |<span data-ttu-id="539f4-165">Nonce de sessão.</span><span class="sxs-lookup"><span data-stu-id="539f4-165">Session nonce.</span></span> <span data-ttu-id="539f4-166">Um valor exclusivo gerado pelo AD FS para ajudar a evitar ataques de repetição.</span><span class="sxs-lookup"><span data-stu-id="539f4-166">A unique value generated by AD FS to help prevent replay attacks.</span></span> |
| <span data-ttu-id="539f4-167">upn</span><span class="sxs-lookup"><span data-stu-id="539f4-167">upn</span></span> |<span data-ttu-id="539f4-168">Nome UPN.</span><span class="sxs-lookup"><span data-stu-id="539f4-168">User principal name (UPN).</span></span> <span data-ttu-id="539f4-169">Exemplo: `john@corp.fabrikam.com`</span><span class="sxs-lookup"><span data-stu-id="539f4-169">Example: `john@corp.fabrikam.com`</span></span> |
| <span data-ttu-id="539f4-170">pwd_exp</span><span class="sxs-lookup"><span data-stu-id="539f4-170">pwd_exp</span></span> |<span data-ttu-id="539f4-171">Período de expiração de senha.</span><span class="sxs-lookup"><span data-stu-id="539f4-171">Password expiration period.</span></span> <span data-ttu-id="539f4-172">O número de segundos de validade da senha do usuário ou de um segredo de autenticação semelhante, como um PIN.</span><span class="sxs-lookup"><span data-stu-id="539f4-172">The number of seconds until the user's password or a similar authentication secret, such as a PIN.</span></span> <span data-ttu-id="539f4-173">expira.</span><span class="sxs-lookup"><span data-stu-id="539f4-173">expires.</span></span> |

> [!NOTE]
> <span data-ttu-id="539f4-174">A declaração "iss" contém o AD FS do parceiro (normalmente, essa declaração identificará o provedor de SaaS como o emissor).</span><span class="sxs-lookup"><span data-stu-id="539f4-174">The "iss" claim contains the AD FS of the partner (typically, this claim will identify the SaaS provider as the issuer).</span></span> <span data-ttu-id="539f4-175">Ela não identifica AD FS do cliente.</span><span class="sxs-lookup"><span data-stu-id="539f4-175">It does not identify the customer's AD FS.</span></span> <span data-ttu-id="539f4-176">Você pode encontrar o domínio do cliente como parte do UPN.</span><span class="sxs-lookup"><span data-stu-id="539f4-176">You can find the customer's domain as part of the UPN.</span></span>

<span data-ttu-id="539f4-177">O restante deste artigo descreve como configurar a relação de confiança entre o RP (o aplicativo) e o parceiro de conta (o cliente).</span><span class="sxs-lookup"><span data-stu-id="539f4-177">The rest of this article describes how to set up the trust relationship between the RP (the app) and the account partner (the customer).</span></span>

## <a name="ad-fs-deployment"></a><span data-ttu-id="539f4-178">Implantação do AD FS</span><span class="sxs-lookup"><span data-stu-id="539f4-178">AD FS deployment</span></span>

<span data-ttu-id="539f4-179">O provedor de SaaS pode implantar o AD FS no local ou em VMs do Azure.</span><span class="sxs-lookup"><span data-stu-id="539f4-179">The SaaS provider can deploy AD FS either on-premise or on Azure VMs.</span></span> <span data-ttu-id="539f4-180">É importante seguir estas diretrizes para obter segurança e disponibilidade:</span><span class="sxs-lookup"><span data-stu-id="539f4-180">For security and availability, the following guidelines are important:</span></span>

* <span data-ttu-id="539f4-181">Implante pelo menos dois servidores do AD FS e dois servidores proxy do AD FS para atingir a melhor disponibilidade do serviço AD FS.</span><span class="sxs-lookup"><span data-stu-id="539f4-181">Deploy at least two AD FS servers and two AD FS proxy servers to achieve the best availability of the AD FS service.</span></span>
* <span data-ttu-id="539f4-182">Os controladores de domínio e servidores do AD FS nunca devem ser expostos diretamente à Internet e devem estar em uma rede virtual com acesso direto a elas.</span><span class="sxs-lookup"><span data-stu-id="539f4-182">Domain controllers and AD FS servers should never be exposed directly to the Internet and should be in a virtual network with direct access to them.</span></span>
* <span data-ttu-id="539f4-183">Os proxies do aplicativo Web (antes conhecidos como proxies do AD FS) devem ser usados para publicar servidores do AD FS na Internet.</span><span class="sxs-lookup"><span data-stu-id="539f4-183">Web application proxies (previously AD FS proxies) must be used to publish AD FS servers to the Internet.</span></span>

<span data-ttu-id="539f4-184">Para configurar uma topologia semelhante no Azure é preciso usar redes virtuais, NSGs, VMs do azure e conjuntos de disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="539f4-184">To set up a similar topology in Azure requires the use of Virtual networks, NSG’s, azure VM’s and availability sets.</span></span> <span data-ttu-id="539f4-185">Para obter mais detalhes, confira [Diretrizes para implantar o Active Directory do Windows Server em máquinas virtuais do Azure][active-directory-on-azure].</span><span class="sxs-lookup"><span data-stu-id="539f4-185">For more details, see [Guidelines for Deploying Windows Server Active Directory on Azure Virtual Machines][active-directory-on-azure].</span></span>

## <a name="configure-openid-connect-authentication-with-ad-fs"></a><span data-ttu-id="539f4-186">Configurar a autenticação do OpenID Connect com o AD FS</span><span class="sxs-lookup"><span data-stu-id="539f4-186">Configure OpenID Connect authentication with AD FS</span></span>

<span data-ttu-id="539f4-187">O provedor de SaaS deve habilitar o OpenID Connect entre o aplicativo e do AD FS.</span><span class="sxs-lookup"><span data-stu-id="539f4-187">The SaaS provider must enable OpenID Connect between the application and AD FS.</span></span> <span data-ttu-id="539f4-188">Para fazer isso, adicione um grupo de aplicativos ao AD FS.</span><span class="sxs-lookup"><span data-stu-id="539f4-188">To do so, add an application group in AD FS.</span></span>  <span data-ttu-id="539f4-189">Você pode encontrar instruções detalhadas nesta [postagem do blog], em " Setting up a Web App for OpenId Connect sign in AD FS".</span><span class="sxs-lookup"><span data-stu-id="539f4-189">You can find detailed instructions in this [blog post], under " Setting up a Web App for OpenId Connect sign in AD FS."</span></span>

<span data-ttu-id="539f4-190">Em seguida, configure o middleware OpenID Connect.</span><span class="sxs-lookup"><span data-stu-id="539f4-190">Next, configure the OpenID Connect middleware.</span></span> <span data-ttu-id="539f4-191">O ponto de extremidade de metadados é `https://domain/adfs/.well-known/openid-configuration`, no qual o domínio é o domínio do AD FS do provedor de SaaS.</span><span class="sxs-lookup"><span data-stu-id="539f4-191">The metadata endpoint is `https://domain/adfs/.well-known/openid-configuration`, where domain is the SaaS provider's AD FS domain.</span></span>

<span data-ttu-id="539f4-192">Normalmente, você pode combinar esse a outros pontos de extremidade do OpenID Connect (por exemplo, AAD).</span><span class="sxs-lookup"><span data-stu-id="539f4-192">Typically you might combine this with other OpenID Connect endpoints (such as AAD).</span></span> <span data-ttu-id="539f4-193">Você precisará de dois botões de entrada diferentes, ou alguma outra maneira para diferenciá-los, para que o usuário seja enviado ao ponto de extremidade de autenticação correto.</span><span class="sxs-lookup"><span data-stu-id="539f4-193">You'll need two different sign-in buttons or some other way to distinguish them, so that the user is sent to the correct authentication endpoint.</span></span>

## <a name="configure-the-ad-fs-resource-partner"></a><span data-ttu-id="539f4-194">Configurar o parceiro de recursos do AD FS</span><span class="sxs-lookup"><span data-stu-id="539f4-194">Configure the AD FS Resource Partner</span></span>

<span data-ttu-id="539f4-195">O provedor de SaaS deve fazer o seguinte para cada cliente que deseja se conectar via ADFS:</span><span class="sxs-lookup"><span data-stu-id="539f4-195">The SaaS provider must do the following for each customer that wants to connect via ADFS:</span></span>

1. <span data-ttu-id="539f4-196">Adicionar uma relação de confiança do provedor de declarações.</span><span class="sxs-lookup"><span data-stu-id="539f4-196">Add a claims provider trust.</span></span>
2. <span data-ttu-id="539f4-197">Adicionar regras de declarações.</span><span class="sxs-lookup"><span data-stu-id="539f4-197">Add claims rules.</span></span>
3. <span data-ttu-id="539f4-198">Habilitar a descoberta de realm inicial.</span><span class="sxs-lookup"><span data-stu-id="539f4-198">Enable home-realm discovery.</span></span>

<span data-ttu-id="539f4-199">Veja as etapas com mais detalhes.</span><span class="sxs-lookup"><span data-stu-id="539f4-199">Here are the steps in more detail.</span></span>

### <a name="add-the-claims-provider-trust"></a><span data-ttu-id="539f4-200">Adicionar a relação de confiança do provedor de declarações</span><span class="sxs-lookup"><span data-stu-id="539f4-200">Add the claims provider trust</span></span>

1. <span data-ttu-id="539f4-201">No Gerenciador do Servidor, clique em **Ferramentas** e depois selecione **Gerenciamento do AD FS**.</span><span class="sxs-lookup"><span data-stu-id="539f4-201">In Server Manager, click **Tools**, and then select **AD FS Management**.</span></span>
2. <span data-ttu-id="539f4-202">Na árvore de console, em **AD FS**, clique com botão direito do mouse em **Confianças do Provedor de Declarações**.</span><span class="sxs-lookup"><span data-stu-id="539f4-202">In the console tree, under **AD FS**, right click **Claims Provider Trusts**.</span></span> <span data-ttu-id="539f4-203">Selecione **Adicionar a Relação de Confiança do Provedor de Declarações**.</span><span class="sxs-lookup"><span data-stu-id="539f4-203">Select **Add Claims Provider Trust**.</span></span>
3. <span data-ttu-id="539f4-204">Clique em **Iniciar** para iniciar o assistente.</span><span class="sxs-lookup"><span data-stu-id="539f4-204">Click **Start** to start the wizard.</span></span>
4. <span data-ttu-id="539f4-205">Selecione os opção "Importar dados sobre o provedor de declarações publicados online ou em uma rede local".</span><span class="sxs-lookup"><span data-stu-id="539f4-205">Select the option "Import data about the claims provider published online or on a local network".</span></span> <span data-ttu-id="539f4-206">Digite o URI do ponto de extremidade de metadados de federação do cliente.</span><span class="sxs-lookup"><span data-stu-id="539f4-206">Enter the URI of the customer's federation metadata endpoint.</span></span> <span data-ttu-id="539f4-207">(Exemplo: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`). Você precisará obter isso do cliente.</span><span class="sxs-lookup"><span data-stu-id="539f4-207">(Example: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`.) You will need to get this from the customer.</span></span>
5. <span data-ttu-id="539f4-208">Conclua o assistente usando as opções padrão.</span><span class="sxs-lookup"><span data-stu-id="539f4-208">Complete the wizard using the default options.</span></span>

### <a name="edit-claims-rules"></a><span data-ttu-id="539f4-209">Editar regras de declarações</span><span class="sxs-lookup"><span data-stu-id="539f4-209">Edit claims rules</span></span>

1. <span data-ttu-id="539f4-210">Clique com o botão direito do mouse na relação de confiança de provedor de declarações recém-adicionada e selecione **Editar Regras de Declarações**.</span><span class="sxs-lookup"><span data-stu-id="539f4-210">Right-click the newly added claims provider trust, and select **Edit Claims Rules**.</span></span>
2. <span data-ttu-id="539f4-211">Clique em **Adicionar Regra**.</span><span class="sxs-lookup"><span data-stu-id="539f4-211">Click **Add Rule**.</span></span>
3. <span data-ttu-id="539f4-212">Selecione "Passar ou Filtrar uma Declaração de Entrada" e clique em **Avançar**.</span><span class="sxs-lookup"><span data-stu-id="539f4-212">Select "Pass Through or Filter an Incoming Claim" and click **Next**.</span></span>
   <span data-ttu-id="539f4-213">![Assistente para Adicionar Regra de Declaração de Transformação](./images/edit-claims-rule.png)</span><span class="sxs-lookup"><span data-stu-id="539f4-213">![Add Transform Claim Rule Wizard](./images/edit-claims-rule.png)</span></span>
4. <span data-ttu-id="539f4-214">Insira um nome para a regra.</span><span class="sxs-lookup"><span data-stu-id="539f4-214">Enter a name for the rule.</span></span>
5. <span data-ttu-id="539f4-215">Em "Tipo de declaração de entrada", selecione **UPN**.</span><span class="sxs-lookup"><span data-stu-id="539f4-215">Under "Incoming claim type", select **UPN**.</span></span>
6. <span data-ttu-id="539f4-216">Selecione "Passar todos os valores de declaração".</span><span class="sxs-lookup"><span data-stu-id="539f4-216">Select "Pass through all claim values".</span></span>
   <span data-ttu-id="539f4-217">![Assistente para Adicionar Regra de Declaração de Transformação](./images/edit-claims-rule2.png)</span><span class="sxs-lookup"><span data-stu-id="539f4-217">![Add Transform Claim Rule Wizard](./images/edit-claims-rule2.png)</span></span>
7. <span data-ttu-id="539f4-218">Clique em **Concluir**.</span><span class="sxs-lookup"><span data-stu-id="539f4-218">Click **Finish**.</span></span>
8. <span data-ttu-id="539f4-219">Repita as etapas 2 a 7 e especifique o **Tipo de Declaração de Ancoragem** do tipo de declaração de entrada.</span><span class="sxs-lookup"><span data-stu-id="539f4-219">Repeat steps 2 - 7, and specify **Anchor Claim Type** for the incoming claim type.</span></span>
9. <span data-ttu-id="539f4-220">Clique em **OK** para concluir o assistente.</span><span class="sxs-lookup"><span data-stu-id="539f4-220">Click **OK** to complete the wizard.</span></span>

### <a name="enable-home-realm-discovery"></a><span data-ttu-id="539f4-221">Habilitar a descoberta de realm inicial</span><span class="sxs-lookup"><span data-stu-id="539f4-221">Enable home-realm discovery</span></span>

<span data-ttu-id="539f4-222">Execute o seguinte script do PowerShell:</span><span class="sxs-lookup"><span data-stu-id="539f4-222">Run the following PowerShell script:</span></span>

```powershell
Set-ADFSClaimsProviderTrust -TargetName "name" -OrganizationalAccountSuffix @("suffix")
```

<span data-ttu-id="539f4-223">onde "name" é o nome amigável da relação de confiança do provedor de declarações e "suffix" é o sufixo UPN do AD do cliente (por exemplo, "corp.fabrikam.com").</span><span class="sxs-lookup"><span data-stu-id="539f4-223">where "name" is the friendly name of the claims provider trust, and "suffix" is the UPN suffix for the customer's AD (example, "corp.fabrikam.com").</span></span>

<span data-ttu-id="539f4-224">Com essa configuração, os usuários finais podem digitar sua conta institucional e o AD FS selecionará automaticamente o provedor de declarações correspondente.</span><span class="sxs-lookup"><span data-stu-id="539f4-224">With this configuration, end users can type in their organizational account, and AD FS automatically selects the corresponding claims provider.</span></span> <span data-ttu-id="539f4-225">Confira [Personalizando as páginas de entrada do AD FS], na seção "Configurar o Provedor de Identidade para usar determinados sufixos de email".</span><span class="sxs-lookup"><span data-stu-id="539f4-225">See [Customizing the AD FS Sign-in Pages], under the section "Configure Identity Provider to use certain email suffixes".</span></span>

## <a name="configure-the-ad-fs-account-partner"></a><span data-ttu-id="539f4-226">Configurar o Parceiro de Conta do AD FS</span><span class="sxs-lookup"><span data-stu-id="539f4-226">Configure the AD FS Account Partner</span></span>

<span data-ttu-id="539f4-227">Os cliente deve fazer o seguinte:</span><span class="sxs-lookup"><span data-stu-id="539f4-227">The customer must do the following:</span></span>

1. <span data-ttu-id="539f4-228">Adicionar uma relação de confiança da RP (terceira parte confiável)</span><span class="sxs-lookup"><span data-stu-id="539f4-228">Add a relying party (RP) trust.</span></span>
2. <span data-ttu-id="539f4-229">Adiciona regras de declarações.</span><span class="sxs-lookup"><span data-stu-id="539f4-229">Adds claims rules.</span></span>

### <a name="add-the-rp-trust"></a><span data-ttu-id="539f4-230">Adicionar a relação de confiança da RP</span><span class="sxs-lookup"><span data-stu-id="539f4-230">Add the RP trust</span></span>

1. <span data-ttu-id="539f4-231">No Gerenciador do Servidor, clique em **Ferramentas** e depois selecione **Gerenciamento do AD FS**.</span><span class="sxs-lookup"><span data-stu-id="539f4-231">In Server Manager, click **Tools**, and then select **AD FS Management**.</span></span>
2. <span data-ttu-id="539f4-232">Na árvore de console, em **AD FS**, clique com o botão direito do mouse em **Confianças da Terceira Parte Confiável**.</span><span class="sxs-lookup"><span data-stu-id="539f4-232">In the console tree, under **AD FS**, right click **Relying Party Trusts**.</span></span> <span data-ttu-id="539f4-233">Selecione **Adicionar Relação de Confiança de Terceira Parte Confiável**.</span><span class="sxs-lookup"><span data-stu-id="539f4-233">Select **Add Relying Party Trust**.</span></span>
3. <span data-ttu-id="539f4-234">Selecione **Com Reconhecimento de Declarações** e clique em **Iniciar**.</span><span class="sxs-lookup"><span data-stu-id="539f4-234">Select **Claims Aware** and click **Start**.</span></span>
4. <span data-ttu-id="539f4-235">Na página **Selecionar Fonte de Dados** , selecione os opção "Importar dados sobre o provedor de declarações publicados online ou em uma rede local".</span><span class="sxs-lookup"><span data-stu-id="539f4-235">On the **Select Data Source** page, select the option "Import data about the claims provider published online or on a local network".</span></span> <span data-ttu-id="539f4-236">Insira o URI do ponto de extremidade de metadados de federação do cliente.</span><span class="sxs-lookup"><span data-stu-id="539f4-236">Enter the URI of the SaaS provider's federation metadata endpoint.</span></span>
   <span data-ttu-id="539f4-237">![Assistente para Adicionar Relação de Confiança de Terceira Parte Confiável](./images/add-rp-trust.png)</span><span class="sxs-lookup"><span data-stu-id="539f4-237">![Add Relying Party Trust Wizard](./images/add-rp-trust.png)</span></span>
5. <span data-ttu-id="539f4-238">Na página **Especificar Nome para Exibição** , insira qualquer nome.</span><span class="sxs-lookup"><span data-stu-id="539f4-238">On the **Specify Display Name** page, enter any name.</span></span>
6. <span data-ttu-id="539f4-239">Na página **Escolher Política de Controle de Acesso** , escolha uma política.</span><span class="sxs-lookup"><span data-stu-id="539f4-239">On the **Choose Access Control Policy** page, choose a policy.</span></span> <span data-ttu-id="539f4-240">Você pode permitir todos na organização ou escolher um grupo de segurança específico.</span><span class="sxs-lookup"><span data-stu-id="539f4-240">You could permit everyone in the organization, or choose a specific security group.</span></span>
   <span data-ttu-id="539f4-241">![Assistente para Adicionar Relação de Confiança de Terceira Parte Confiável](./images/add-rp-trust2.png)</span><span class="sxs-lookup"><span data-stu-id="539f4-241">![Add Relying Party Trust Wizard](./images/add-rp-trust2.png)</span></span>
7. <span data-ttu-id="539f4-242">Insira todos os parâmetros necessários na caixa **Política**.</span><span class="sxs-lookup"><span data-stu-id="539f4-242">Enter any parameters required in the **Policy** box.</span></span>
8. <span data-ttu-id="539f4-243">Clique em **Avançar** para concluir o assistente.</span><span class="sxs-lookup"><span data-stu-id="539f4-243">Click **Next** to complete the wizard.</span></span>

### <a name="add-claims-rules"></a><span data-ttu-id="539f4-244">Adicionar regras de declarações</span><span class="sxs-lookup"><span data-stu-id="539f4-244">Add claims rules</span></span>

1. <span data-ttu-id="539f4-245">Clique com o botão direito do mouse na terceira parte confiável recém-adicionada e selecione **Editar Política de Emissão de Declaração**.</span><span class="sxs-lookup"><span data-stu-id="539f4-245">Right-click the newly added relying party trust, and select **Edit Claim Issuance Policy**.</span></span>
2. <span data-ttu-id="539f4-246">Clique em **Adicionar Regra**.</span><span class="sxs-lookup"><span data-stu-id="539f4-246">Click **Add Rule**.</span></span>
3. <span data-ttu-id="539f4-247">Selecione "Enviar Atributos do LDAP como Declarações" e clique em **Avançar**.</span><span class="sxs-lookup"><span data-stu-id="539f4-247">Select "Send LDAP Attributes as Claims" and click **Next**.</span></span>
4. <span data-ttu-id="539f4-248">Insira um nome para a regra, como “UPN”.</span><span class="sxs-lookup"><span data-stu-id="539f4-248">Enter a name for the rule, such as "UPN".</span></span>
5. <span data-ttu-id="539f4-249">Em **Armazenamento de atributos**, selecione **Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="539f4-249">Under **Attribute store**, select **Active Directory**.</span></span>
   <span data-ttu-id="539f4-250">![Assistente para Adicionar Regra de Declaração de Transformação](./images/add-claims-rules.png)</span><span class="sxs-lookup"><span data-stu-id="539f4-250">![Add Transform Claim Rule Wizard](./images/add-claims-rules.png)</span></span>
6. <span data-ttu-id="539f4-251">Na seção **Mapeamento de atributos LDAP** :</span><span class="sxs-lookup"><span data-stu-id="539f4-251">In the **Mapping of LDAP attributes** section:</span></span>
   * <span data-ttu-id="539f4-252">Em **Atributo LDAP**, selecione **User-Principal-Name**.</span><span class="sxs-lookup"><span data-stu-id="539f4-252">Under **LDAP Attribute**, select **User-Principal-Name**.</span></span>
   * <span data-ttu-id="539f4-253">Em **Tipo de Declaração de Saída**, selecione **UPN**.</span><span class="sxs-lookup"><span data-stu-id="539f4-253">Under **Outgoing Claim Type**, select **UPN**.</span></span>
     <span data-ttu-id="539f4-254">![Assistente para Adicionar Regra de Declaração de Transformação](./images/add-claims-rules2.png)</span><span class="sxs-lookup"><span data-stu-id="539f4-254">![Add Transform Claim Rule Wizard](./images/add-claims-rules2.png)</span></span>
7. <span data-ttu-id="539f4-255">Clique em **Concluir**.</span><span class="sxs-lookup"><span data-stu-id="539f4-255">Click **Finish**.</span></span>
8. <span data-ttu-id="539f4-256">Clique em **Adicionar regra** novamente.</span><span class="sxs-lookup"><span data-stu-id="539f4-256">Click **Add Rule** again.</span></span>
9. <span data-ttu-id="539f4-257">Selecione “Enviar Declarações Usando uma Regra Personalizada” e clique em **Avançar**.</span><span class="sxs-lookup"><span data-stu-id="539f4-257">Select "Send Claims Using a Custom Rule" and click **Next**.</span></span>
10. <span data-ttu-id="539f4-258">Insira um nome para a regra, como “Tipo de Declaração de Ancoragem”.</span><span class="sxs-lookup"><span data-stu-id="539f4-258">Enter a name for the rule, such as "Anchor Claim Type".</span></span>
11. <span data-ttu-id="539f4-259">Em **Regra personalizada**, insira o seguinte:</span><span class="sxs-lookup"><span data-stu-id="539f4-259">Under **Custom rule**, enter the following:</span></span>

    ```console
    EXISTS([Type == "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype"])=>
    issue (Type = "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype",
          Value = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn");
    ```

    <span data-ttu-id="539f4-260">Essa regra emite uma declaração de tipo `anchorclaimtype`.</span><span class="sxs-lookup"><span data-stu-id="539f4-260">This rule issues a claim of type `anchorclaimtype`.</span></span> <span data-ttu-id="539f4-261">A declaração informa a terceira parte confiável para usar o UPN como a ID imutável. do usuário.</span><span class="sxs-lookup"><span data-stu-id="539f4-261">The claim tells the relying party to use UPN as the user's immutable ID.</span></span>
12. <span data-ttu-id="539f4-262">Clique em **Concluir**.</span><span class="sxs-lookup"><span data-stu-id="539f4-262">Click **Finish**.</span></span>
13. <span data-ttu-id="539f4-263">Clique em **OK** para concluir o assistente.</span><span class="sxs-lookup"><span data-stu-id="539f4-263">Click **OK** to complete the wizard.</span></span>

<!-- links -->

[Azure AD Connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
[confiança de federação]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[federation trust]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[parceiro de conta]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[account partner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[parceiro de recurso]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[resource partner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[Instante da autenticação]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[Authentication instant]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[Hora de expiração]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[Expiration time]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[Identificador de nome]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[Name identifier]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[active-directory-on-azure]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[postagem do blog]: https://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[blog post]: https://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[Personalizando as páginas de entrada do AD FS]: https://technet.microsoft.com/library/dn280950.aspx
[Customizing the AD FS Sign-in Pages]: https://technet.microsoft.com/library/dn280950.aspx
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[client assertion]: client-assertion.md
[active-directory-dotnet-webapp-wsfederation]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation
