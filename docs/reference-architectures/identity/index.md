---
title: Integrar Active Directory local ao Azure
titleSuffix: Azure Reference Architectures
description: Comparar arquiteturas de referência para a integração do Active Directory local ao Azure.
ms.date: 07/02/2018
ms.custom: seodec18
ms.openlocfilehash: 905dedda6de1a107f55b2f7651441780a685aea7
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/08/2018
ms.locfileid: "53119856"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a><span data-ttu-id="b75c1-103">Escolha uma solução para a integração do Active Directory local ao Azure</span><span class="sxs-lookup"><span data-stu-id="b75c1-103">Choose a solution for integrating on-premises Active Directory with Azure</span></span>

<span data-ttu-id="b75c1-104">Esta série mostra opções para integrar seu ambiente do AD (Active Directory) local a uma rede do Azure.</span><span class="sxs-lookup"><span data-stu-id="b75c1-104">This article compares options for integrating your on-premises Active Directory (AD) environment with an Azure network.</span></span> <span data-ttu-id="b75c1-105">Para cada opção, existe uma arquitetura de referência mais detalhada disponível.</span><span class="sxs-lookup"><span data-stu-id="b75c1-105">For each option, a more detailed reference architecture is available.</span></span>

<span data-ttu-id="b75c1-106">Muitas organizações usam AD DS (Active Directory Domain Services) para autenticar identidades associadas a usuários, computadores, aplicativos ou outros recursos incluídos em um limite de segurança.</span><span class="sxs-lookup"><span data-stu-id="b75c1-106">Many organizations use Active Directory Domain Services (AD DS) to authenticate identities associated with users, computers, applications, or other resources that are included in a security boundary.</span></span> <span data-ttu-id="b75c1-107">Serviços de identidade e diretório geralmente são hospedados localmente, mas se o aplicativo estiver hospedado parcialmente localmente e parcialmente no Azure, poderá haver latência ao enviar solicitações de autenticação do Azure de volta para o local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-107">Directory and identity services are typically hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, there may be latency sending authentication requests from Azure back to on-premises.</span></span> <span data-ttu-id="b75c1-108">A implementação de serviços de identidade e diretório no Azure pode reduzir essa latência.</span><span class="sxs-lookup"><span data-stu-id="b75c1-108">Implementing directory and identity services in Azure can reduce this latency.</span></span>

<span data-ttu-id="b75c1-109">O Azure oferece duas soluções para a implementação de serviços de identidade e diretório no Azure:</span><span class="sxs-lookup"><span data-stu-id="b75c1-109">Azure provides two solutions for implementing directory and identity services in Azure:</span></span>

- <span data-ttu-id="b75c1-110">Use [Azure AD][azure-active-directory] para criar um domínio do Active Directory na nuvem e conectá-lo ao seu domínio do Active Directory local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-110">Use [Azure AD][azure-active-directory] to create an Active Directory domain in the cloud and connect it to your on-premises Active Directory domain.</span></span> <span data-ttu-id="b75c1-111">O [Azure AD Connect][azure-ad-connect] integra seus diretórios locais ao Azure AD.</span><span class="sxs-lookup"><span data-stu-id="b75c1-111">[Azure AD Connect][azure-ad-connect] integrates your on-premises directories with Azure AD.</span></span>

- <span data-ttu-id="b75c1-112">Estenda sua infraestrutura local existente do Active Directory para o Azure implantando uma VM no Azure que execute o AD DS como um controlador de domínio.</span><span class="sxs-lookup"><span data-stu-id="b75c1-112">Extend your existing on-premises Active Directory infrastructure to Azure, by deploying a VM in Azure that runs AD DS as a domain controller.</span></span> <span data-ttu-id="b75c1-113">Essa arquitetura geralmente é usada quando a rede local e a VNet (rede virtual) do Azure estão conectadas por uma conexão VPN ou ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="b75c1-113">This architecture is more common when the on-premises network and the Azure virtual network (VNet) are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="b75c1-114">Diversas variações dessa arquitetura são possíveis:</span><span class="sxs-lookup"><span data-stu-id="b75c1-114">Several variations of this architecture are possible:</span></span>

  - <span data-ttu-id="b75c1-115">Crie um domínio no Azure e associe-o à sua floresta do AD local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-115">Create a domain in Azure and join it to your on-premises AD forest.</span></span>
  - <span data-ttu-id="b75c1-116">Crie uma floresta separada no Azure que seja confiável para domínios na sua floresta local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-116">Create a separate forest in Azure that is trusted by domains in your on-premises forest.</span></span>
  - <span data-ttu-id="b75c1-117">Replique uma implantação do Serviços de Federação do Active Directory (AD FS) no Azure.</span><span class="sxs-lookup"><span data-stu-id="b75c1-117">Replicate an Active Directory Federation Services (AD FS) deployment to Azure.</span></span>

<span data-ttu-id="b75c1-118">As seções a seguir descrevem cada uma dessas opções em mais detalhes.</span><span class="sxs-lookup"><span data-stu-id="b75c1-118">The next sections describe each of these options in more detail.</span></span>

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a><span data-ttu-id="b75c1-119">Integre seus domínios locais ao Azure AD</span><span class="sxs-lookup"><span data-stu-id="b75c1-119">Integrate your on-premises domains with Azure AD</span></span>

<span data-ttu-id="b75c1-120">Use o Azure AD (Azure Active Directory) para criar um domínio no Azure e vinculá-lo a um domínio do AD local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-120">Use Azure Active Directory (Azure AD) to create a domain in Azure and link it to an on-premises AD domain.</span></span>

<span data-ttu-id="b75c1-121">O diretório do Azure AD não é uma extensão de um diretório local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-121">The Azure AD directory is not an extension of an on-premises directory.</span></span> <span data-ttu-id="b75c1-122">Em vez disso, é uma cópia que contém os mesmos objetos e identidades.</span><span class="sxs-lookup"><span data-stu-id="b75c1-122">Rather, it's a copy that contains the same objects and identities.</span></span> <span data-ttu-id="b75c1-123">As alterações feitas a esses itens localmente são copiadas para o Azure AD, mas as alterações feitas ao Azure AD não são replicadas de volta para o domínio local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-123">Changes made to these items on-premises are copied to Azure AD, but changes made in Azure AD are not replicated back to the on-premises domain.</span></span>

<span data-ttu-id="b75c1-124">Você também pode usar o Azure AD sem usar um diretório local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-124">You can also use Azure AD without using an on-premises directory.</span></span> <span data-ttu-id="b75c1-125">Nesse caso, o Azure AD atua como a origem primária de todas as informações de identidade, em vez de contendo dados replicados de um diretório local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-125">In this case, Azure AD acts as the primary source of all identity information, rather than containing data replicated from an on-premises directory.</span></span>

<span data-ttu-id="b75c1-126">**Benefícios**</span><span class="sxs-lookup"><span data-stu-id="b75c1-126">**Benefits**</span></span>

- <span data-ttu-id="b75c1-127">Você não precisa manter uma infraestrutura do AD na nuvem.</span><span class="sxs-lookup"><span data-stu-id="b75c1-127">You don't need to maintain an AD infrastructure in the cloud.</span></span> <span data-ttu-id="b75c1-128">O Azure AD é totalmente gerenciado e mantido pela Microsoft.</span><span class="sxs-lookup"><span data-stu-id="b75c1-128">Azure AD is entirely managed and maintained by Microsoft.</span></span>
- <span data-ttu-id="b75c1-129">O Azure AD fornece as mesmas informações de identidade disponíveis localmente.</span><span class="sxs-lookup"><span data-stu-id="b75c1-129">Azure AD provides the same identity information that is available on-premises.</span></span>
- <span data-ttu-id="b75c1-130">A autenticação pode acontecer no Azure, reduzindo a necessidade de aplicativos e usuários externos entrarem em contato com o domínio local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-130">Authentication can happen in Azure, reducing the need for external applications and users to contact the on-premises domain.</span></span>

<span data-ttu-id="b75c1-131">**Desafios**</span><span class="sxs-lookup"><span data-stu-id="b75c1-131">**Challenges**</span></span>

- <span data-ttu-id="b75c1-132">Serviços de identidade são limitados a usuários e grupos.</span><span class="sxs-lookup"><span data-stu-id="b75c1-132">Identity services are limited to users and groups.</span></span> <span data-ttu-id="b75c1-133">Não é possível autenticar contas de serviço e computador.</span><span class="sxs-lookup"><span data-stu-id="b75c1-133">There is no ability to authenticate service and computer accounts.</span></span>
- <span data-ttu-id="b75c1-134">Você deve configurar a conectividade com o seu domínio local para manter o diretório do Azure AD sincronizado.</span><span class="sxs-lookup"><span data-stu-id="b75c1-134">You must configure connectivity with your on-premises domain to keep the Azure AD directory synchronized.</span></span> 
- <span data-ttu-id="b75c1-135">Os aplicativos talvez precisem ser reescritos para habilitar a autenticação por meio do Azure AD.</span><span class="sxs-lookup"><span data-stu-id="b75c1-135">Applications may need to be rewritten to enable authentication through Azure AD.</span></span>

<span data-ttu-id="b75c1-136">**Arquitetura de referência**</span><span class="sxs-lookup"><span data-stu-id="b75c1-136">**Reference architecture**</span></span>

- <span data-ttu-id="b75c1-137">[Integrar domínios do Active Directory local ao Azure Active Directory][aad]</span><span class="sxs-lookup"><span data-stu-id="b75c1-137">[Integrate on-premises Active Directory domains with Azure Active Directory][aad]</span></span>

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a><span data-ttu-id="b75c1-138">AD DS no Azure unido a uma floresta local</span><span class="sxs-lookup"><span data-stu-id="b75c1-138">AD DS in Azure joined to an on-premises forest</span></span>

<span data-ttu-id="b75c1-139">Implantar servidores do AD DS (Active Directory Domain Services) para o Azure.</span><span class="sxs-lookup"><span data-stu-id="b75c1-139">Deploy AD Domain Services (AD DS) servers to Azure.</span></span> <span data-ttu-id="b75c1-140">Crie um domínio no Azure e associe-o à sua floresta do AD local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-140">Create a domain in Azure and join it to your on-premises AD forest.</span></span> 

<span data-ttu-id="b75c1-141">Considere essa opção se você precisar usar recursos do AD DS que não estejam atualmente implementados pelo Azure AD.</span><span class="sxs-lookup"><span data-stu-id="b75c1-141">Consider this option if you need to use AD DS features that are not currently implemented by Azure AD.</span></span> 

<span data-ttu-id="b75c1-142">**Benefícios**</span><span class="sxs-lookup"><span data-stu-id="b75c1-142">**Benefits**</span></span>

- <span data-ttu-id="b75c1-143">Fornece acesso às mesmas informações de identidade disponíveis localmente.</span><span class="sxs-lookup"><span data-stu-id="b75c1-143">Provides access to the same identity information that is available on-premises.</span></span>
- <span data-ttu-id="b75c1-144">Você pode autenticar contas de usuário, serviço e computador localmente e no Azure.</span><span class="sxs-lookup"><span data-stu-id="b75c1-144">You can authenticate user, service, and computer accounts on-premises and in Azure.</span></span>
- <span data-ttu-id="b75c1-145">Você não precisa gerenciar uma floresta do AD separada.</span><span class="sxs-lookup"><span data-stu-id="b75c1-145">You don't need to manage a separate AD forest.</span></span> <span data-ttu-id="b75c1-146">O domínio no Azure pode pertencer à floresta local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-146">The domain in Azure can belong to the on-premises forest.</span></span>
- <span data-ttu-id="b75c1-147">Você pode aplicar a política de grupo definida por Objetos de Política de Grupo local ao domínio no Azure.</span><span class="sxs-lookup"><span data-stu-id="b75c1-147">You can apply group policy defined by on-premises Group Policy Objects to the domain in Azure.</span></span>

<span data-ttu-id="b75c1-148">**Desafios**</span><span class="sxs-lookup"><span data-stu-id="b75c1-148">**Challenges**</span></span>

- <span data-ttu-id="b75c1-149">Você deve implantar e gerenciar seus próprios servidores do AD DS e o domínio na nuvem.</span><span class="sxs-lookup"><span data-stu-id="b75c1-149">You must deploy and manage your own AD DS servers and domain in the cloud.</span></span>
- <span data-ttu-id="b75c1-150">Pode haver alguma latência de sincronização entre os servidores de domínio na nuvem e os servidores em execução localmente.</span><span class="sxs-lookup"><span data-stu-id="b75c1-150">There may be some synchronization latency between the domain servers in the cloud and the servers running on-premises.</span></span>

<span data-ttu-id="b75c1-151">**Arquitetura de referência**</span><span class="sxs-lookup"><span data-stu-id="b75c1-151">**Reference architecture**</span></span>

- <span data-ttu-id="b75c1-152">[Estender o AD DS (Active Directory Domain Services) para o Azure][ad-ds]</span><span class="sxs-lookup"><span data-stu-id="b75c1-152">[Extend Active Directory Domain Services (AD DS) to Azure][ad-ds]</span></span>

## <a name="ad-ds-in-azure-with-a-separate-forest"></a><span data-ttu-id="b75c1-153">AD DS no Azure com uma floresta separada</span><span class="sxs-lookup"><span data-stu-id="b75c1-153">AD DS in Azure with a separate forest</span></span>

<span data-ttu-id="b75c1-154">Implante os servidores do AD DS (Serviços de Domínio do AD) no Azure, mas crie uma [floresta][ad-forest-defn] do Active Directory separada da floresta local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-154">Deploy AD Domain Services (AD DS) servers to Azure, but create a separate Active Directory [forest][ad-forest-defn] that is separate from the on-premises forest.</span></span> <span data-ttu-id="b75c1-155">Essa floresta terá a confiança de domínios na floresta local.</span><span class="sxs-lookup"><span data-stu-id="b75c1-155">This forest is trusted by domains in your on-premises forest.</span></span>

<span data-ttu-id="b75c1-156">Usos típicos para essa arquitetura incluem manter uma separação segurança para objetos e identidades mantidos na nuvem e migrar os domínios individuais do local para a nuvem.</span><span class="sxs-lookup"><span data-stu-id="b75c1-156">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="b75c1-157">**Benefícios**</span><span class="sxs-lookup"><span data-stu-id="b75c1-157">**Benefits**</span></span>

- <span data-ttu-id="b75c1-158">Você pode implementar identidades locais e separar identidades somente do Azure.</span><span class="sxs-lookup"><span data-stu-id="b75c1-158">You can implement on-premises identities and separate Azure-only identities.</span></span>
- <span data-ttu-id="b75c1-159">Você não precisa replicar da floresta do AD local para o Azure.</span><span class="sxs-lookup"><span data-stu-id="b75c1-159">You don't need to replicate from the on-premises AD forest to Azure.</span></span>

<span data-ttu-id="b75c1-160">**Desafios**</span><span class="sxs-lookup"><span data-stu-id="b75c1-160">**Challenges**</span></span>

- <span data-ttu-id="b75c1-161">A autenticação no Azure para identidades locais requer saltos de rede extras para os servidores AD locais.</span><span class="sxs-lookup"><span data-stu-id="b75c1-161">Authentication within Azure for on-premises identities requires extra network hops to the on-premises AD servers.</span></span>
- <span data-ttu-id="b75c1-162">Você deve implantar seus próprios servidores e floresta AD DS na nuvem e estabelecer as relações de confiança apropriadas entre as florestas.</span><span class="sxs-lookup"><span data-stu-id="b75c1-162">You must deploy your own AD DS servers and forest in the cloud, and establish the appropriate trust relationships between forests.</span></span>

<span data-ttu-id="b75c1-163">**Arquitetura de referência**</span><span class="sxs-lookup"><span data-stu-id="b75c1-163">**Reference architecture**</span></span>

- <span data-ttu-id="b75c1-164">[Criar uma floresta de recursos do AD DS (Active Directory Domain Services) no Azure][ad-ds-forest]</span><span class="sxs-lookup"><span data-stu-id="b75c1-164">[Create an Active Directory Domain Services (AD DS) resource forest in Azure][ad-ds-forest]</span></span>

## <a name="extend-ad-fs-to-azure"></a><span data-ttu-id="b75c1-165">Estender o AD FS para o Azure</span><span class="sxs-lookup"><span data-stu-id="b75c1-165">Extend AD FS to Azure</span></span>

<span data-ttu-id="b75c1-166">Replique uma implantação de Serviços de Federação do Active Directory (AD FS) no Azure para executar autenticação federada e autorização para componentes em execução no Azure.</span><span class="sxs-lookup"><span data-stu-id="b75c1-166">Replicate an Active Directory Federation Services (AD FS) deployment to Azure, to perform federated authentication and authorization for components running in Azure.</span></span> 

<span data-ttu-id="b75c1-167">Usos típicos para essa arquitetura:</span><span class="sxs-lookup"><span data-stu-id="b75c1-167">Typical uses for this architecture:</span></span>

- <span data-ttu-id="b75c1-168">Autenticar e autorizar usuários de organizações parceiras.</span><span class="sxs-lookup"><span data-stu-id="b75c1-168">Authenticate and authorize users from partner organizations.</span></span>
- <span data-ttu-id="b75c1-169">Permitir que os usuários façam a autenticação usando navegadores Web em execução fora do firewall organizacional.</span><span class="sxs-lookup"><span data-stu-id="b75c1-169">Allow users to authenticate from web browsers running outside of the organizational firewall.</span></span>
- <span data-ttu-id="b75c1-170">Permitir que os usuários se conectem de dispositivos externos autorizados, como dispositivos móveis.</span><span class="sxs-lookup"><span data-stu-id="b75c1-170">Allow users to connect from authorized external devices such as mobile devices.</span></span> 

<span data-ttu-id="b75c1-171">**Benefícios**</span><span class="sxs-lookup"><span data-stu-id="b75c1-171">**Benefits**</span></span>

- <span data-ttu-id="b75c1-172">Você pode aproveitar aplicativos com reconhecimento de declarações.</span><span class="sxs-lookup"><span data-stu-id="b75c1-172">You can leverage claims-aware applications.</span></span>
- <span data-ttu-id="b75c1-173">Fornece a capacidade de confiar em parceiros externos para autenticação.</span><span class="sxs-lookup"><span data-stu-id="b75c1-173">Provides the ability to trust external partners for authentication.</span></span>
- <span data-ttu-id="b75c1-174">Compatibilidade com um grande conjunto de protocolos de autenticação.</span><span class="sxs-lookup"><span data-stu-id="b75c1-174">Compatibility with large set of authentication protocols.</span></span>

<span data-ttu-id="b75c1-175">**Desafios**</span><span class="sxs-lookup"><span data-stu-id="b75c1-175">**Challenges**</span></span>

- <span data-ttu-id="b75c1-176">Você deve implantar seus próprios servidores AD DS, AD FS e Proxy de Aplicativo Web do AD FS no Azure.</span><span class="sxs-lookup"><span data-stu-id="b75c1-176">You must deploy your own AD DS, AD FS, and AD FS Web Application Proxy servers in Azure.</span></span>
- <span data-ttu-id="b75c1-177">A configuração dessa arquitetura pode ser complexa.</span><span class="sxs-lookup"><span data-stu-id="b75c1-177">This architecture can be complex to configure.</span></span>

<span data-ttu-id="b75c1-178">**Arquitetura de referência**</span><span class="sxs-lookup"><span data-stu-id="b75c1-178">**Reference architecture**</span></span>

- <span data-ttu-id="b75c1-179">[Estender o Serviços de Federação do Active Directory (AD FS) para o Azure][adfs]</span><span class="sxs-lookup"><span data-stu-id="b75c1-179">[Extend Active Directory Federation Services (AD FS) to Azure][adfs]</span></span>

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: /windows/desktop/AD/forests
[adfs]: ./adfs.md

[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
