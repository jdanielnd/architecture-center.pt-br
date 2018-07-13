---
title: Escolha uma solução para a integração do Active Directory local ao Azure.
description: Compara arquiteturas de referência para a integração do Active Directory local ao Azure.
ms.date: 07/02/2018
ms.openlocfilehash: 7e89998c59bccf4d37cebca5ddd4ea7ecba58cd5
ms.sourcegitcommit: 776b8c1efc662d42273a33de3b82ec69e3cd80c5
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/12/2018
ms.locfileid: "38987522"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a>Escolha uma solução para a integração do Active Directory local ao Azure

Esta série mostra opções para integrar seu ambiente do AD (Active Directory) local a uma rede do Azure. Para cada opção, existe uma arquitetura de referência mais detalhada disponível.

Muitas organizações usam AD DS (Active Directory Domain Services) para autenticar identidades associadas a usuários, computadores, aplicativos ou outros recursos incluídos em um limite de segurança. Serviços de identidade e diretório geralmente são hospedados localmente, mas se o aplicativo estiver hospedado parcialmente localmente e parcialmente no Azure, poderá haver latência ao enviar solicitações de autenticação do Azure de volta para o local. A implementação de serviços de identidade e diretório no Azure pode reduzir essa latência.

O Azure oferece duas soluções para a implementação de serviços de identidade e diretório no Azure: 

* Use [Azure AD][azure-active-directory] para criar um domínio do Active Directory na nuvem e conectá-lo ao seu domínio do Active Directory local. O [Azure AD Connect][azure-ad-connect] integra seus diretórios locais ao Azure AD.

* Estenda sua infraestrutura local existente do Active Directory para o Azure implantando uma VM no Azure que execute o AD DS como um controlador de domínio. Essa arquitetura geralmente é usada quando a rede local e a VNet (rede virtual) do Azure estão conectadas por uma conexão VPN ou ExpressRoute. Diversas variações dessa arquitetura são possíveis: 

    - Crie um domínio no Azure e associe-o à sua floresta do AD local.
    - Crie uma floresta separada no Azure que seja confiável para domínios na sua floresta local.
    - Replique uma implantação do Serviços de Federação do Active Directory (AD FS) no Azure. 

As seções a seguir descrevem cada uma dessas opções em mais detalhes.

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a>Integre seus domínios locais ao Azure AD

Use o Azure AD (Azure Active Directory) para criar um domínio no Azure e vinculá-lo a um domínio do AD local. 

O diretório do Azure AD não é uma extensão de um diretório local. Em vez disso, é uma cópia que contém os mesmos objetos e identidades. As alterações feitas a esses itens localmente são copiadas para o Azure AD, mas as alterações feitas ao Azure AD não são replicadas de volta para o domínio local.

Você também pode usar o Azure AD sem usar um diretório local. Nesse caso, o Azure AD atua como a origem primária de todas as informações de identidade, em vez de contendo dados replicados de um diretório local.

**Benefícios**

* Você não precisa manter uma infraestrutura do AD na nuvem. O Azure AD é totalmente gerenciado e mantido pela Microsoft.
* O Azure AD fornece as mesmas informações de identidade disponíveis localmente.
* A autenticação pode acontecer no Azure, reduzindo a necessidade de aplicativos e usuários externos entrarem em contato com o domínio local.

**Desafios**

* Serviços de identidade são limitados a usuários e grupos. Não é possível autenticar contas de serviço e computador.
* Você deve configurar a conectividade com o seu domínio local para manter o diretório do Azure AD sincronizado. 
* Os aplicativos talvez precisem ser reescritos para habilitar a autenticação por meio do Azure AD.

**Arquitetura de referência**

- [Integrar domínios do Active Directory local ao Azure Active Directory][aad]

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a>AD DS no Azure unido a uma floresta local

Implantar servidores do AD DS (Active Directory Domain Services) para o Azure. Crie um domínio no Azure e associe-o à sua floresta do AD local. 

Considere essa opção se você precisar usar recursos do AD DS que não estejam atualmente implementados pelo Azure AD. 

**Benefícios**

* Fornece acesso às mesmas informações de identidade disponíveis localmente.
* Você pode autenticar contas de usuário, serviço e computador localmente e no Azure.
* Você não precisa gerenciar uma floresta do AD separada. O domínio no Azure pode pertencer à floresta local.
* Você pode aplicar a política de grupo definida por Objetos de Política de Grupo local ao domínio no Azure.

**Desafios**

* Você deve implantar e gerenciar seus próprios servidores do AD DS e o domínio na nuvem.
* Pode haver alguma latência de sincronização entre os servidores de domínio na nuvem e os servidores em execução localmente.

**Arquitetura de referência**

- [Estender o AD DS (Active Directory Domain Services) para o Azure][ad-ds]

## <a name="ad-ds-in-azure-with-a-separate-forest"></a>AD DS no Azure com uma floresta separada

Implante os servidores do AD DS (Serviços de Domínio do AD) no Azure, mas crie uma [floresta][ad-forest-defn] do Active Directory separada da floresta local. Essa floresta terá a confiança de domínios na floresta local.

Usos típicos para essa arquitetura incluem manter uma separação segurança para objetos e identidades mantidos na nuvem e migrar os domínios individuais do local para a nuvem.

**Benefícios**

* Você pode implementar identidades locais e separar identidades somente do Azure.
* Você não precisa replicar da floresta do AD local para o Azure.

**Desafios**

* A autenticação no Azure para identidades locais requer saltos de rede extras para os servidores AD locais.
* Você deve implantar seus próprios servidores e floresta AD DS na nuvem e estabelecer as relações de confiança apropriadas entre as florestas.

**Arquitetura de referência**

- [Criar uma floresta de recursos do AD DS (Active Directory Domain Services) no Azure][ad-ds-forest]

## <a name="extend-ad-fs-to-azure"></a>Estender o AD FS para o Azure

Replique uma implantação de Serviços de Federação do Active Directory (AD FS) no Azure para executar autenticação federada e autorização para componentes em execução no Azure. 

Usos típicos para essa arquitetura:

* Autenticar e autorizar usuários de organizações parceiras.
* Permitir que os usuários façam a autenticação usando navegadores Web em execução fora do firewall organizacional.
* Permitir que os usuários se conectem de dispositivos externos autorizados, como dispositivos móveis. 

**Benefícios**

* Você pode aproveitar aplicativos com reconhecimento de declarações.
* Fornece a capacidade de confiar em parceiros externos para autenticação.
* Compatibilidade com um grande conjunto de protocolos de autenticação.

**Desafios**

* Você deve implantar seus próprios servidores AD DS, AD FS e Proxy de Aplicativo Web do AD FS no Azure.
* A configuração dessa arquitetura pode ser complexa.

**Arquitetura de referência**

- [Estender o Serviços de Federação do Active Directory (AD FS) para o Azure][adfs]

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: /windows/desktop/AD/forests
[adfs]: ./adfs.md

[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
