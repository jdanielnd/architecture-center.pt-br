---
title: 'CAF: guia de decisão de identidade'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Saiba mais sobre identidade como um serviço principal nas migrações do Azure.
author: rotycenh
ms.openlocfilehash: addc11928a0bc72917a28b468a04880720a1fdf4
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900269"
---
# <a name="identity-decision-guide"></a>Guia de decisão de identidade

Em qualquer ambiente, local, híbrido ou somente na nuvem, o departamento de TI precisa controlar quais administradores, usuários e grupos têm acesso aos recursos. Os serviços de IAM (Gerenciamento de Identidades e Acesso) permitem que você gerencie o controle de acesso na nuvem.

![Gráfico com opções de identidade, da menos para a mais complexa, alinhada aos links de salto abaixo](../../_images/discovery-guides/discovery-guide-identity.png)

Ir para: [Determinar requisitos de integração de identidade](#determine-identity-integration-requirements) | [Nativo de nuvem](#cloud-baseline) | [Sincronização de Diretório](#directory-synchronization) | [Serviços de domínio hospedados na nuvem](#cloud-hosted-domain-services) | [Serviços de Federação do Active Directory (AD FS)](#active-directory-federation-services) | [Integração de identidade em evolução](#evolving-identity-integration)  |  [Saiba mais](#learn-more)

Há várias maneiras de gerenciar a identidade em um ambiente de nuvem, que variam em termos de custo e complexidade. Um fator essencial na estruturação dos serviços de identidade baseados em nuvem é o nível de integração exigido pela infraestrutura de identidade local existente.

As soluções de identidade de SaaS (software como serviço) baseadas em nuvem fornecem um nível básico de gerenciamento de identidades e de controle de acesso para recursos da nuvem. No entanto, se a infraestrutura do AD (Active Directory) da sua organização tiver uma estrutura de floresta complexa ou UOs (unidades organizacionais) personalizadas, as cargas de trabalho baseadas em nuvem poderão exigir a replicação de diretório na nuvem para manter um conjunto consistente de identidades, grupos e funções entre os ambientes local e de nuvem. Se a replicação de diretório é exigida para uma solução global, a complexidade pode aumentar significativamente. Além disso, o suporte a aplicativos dependentes de mecanismos de autenticação herdados pode exigir a implantação dos serviços de domínio na nuvem.

## <a name="determine-identity-integration-requirements"></a>Determinar os requisitos de integração de identidade

| Pergunta | Linha de base de nuvem | Sincronização de diretório | Serviços de domínio hospedados na nuvem | Serviços de Federação do AD |
|------|------|------|------|------|
| Você não tem um serviço de diretório local atualmente? | Sim | Não | Não | Não  |
| Suas cargas de trabalho precisam ser autenticadas em relação a algum serviço de identidade local? | Não  | Sim | Não | Não  |
| Suas cargas de trabalho dependem de mecanismos de autenticação herdados, como Kerberos ou NTLM? | Não  | Não  | Sim | Não  |
| A integração entre a nuvem e o serviços de identidade locais não é possível? | Não  | Não  | Sim | Não  |
| Você precisa de logon único entre vários provedores de identidade? | Não  | Não | Não  | Sim |

Como parte do planejamento da migração para o Azure, você precisará determinar a melhor maneira de integrar seus serviços existentes de gerenciamento de identidades e de identidade na nuvem. Abaixo temos cenários comuns de integração.

### <a name="cloud-baseline"></a>Linha de base de nuvem

As plataformas de nuvem pública fornecem um sistema de IAM nativo para conceder a usuários e grupos acesso aos recursos de gerenciamento. Se sua organização não tem uma solução de identidade local relevante e você planeja migrar cargas de trabalho por questões de compatibilidade com mecanismos de autenticação baseados em nuvem, crie sua infraestrutura de identidade usando um serviço de identidade nativo de nuvem.

**Pressuposições da linha de base de nuvem**. O uso de uma infraestrutura de identidade puramente nativa de nuvem pressupõe o seguinte:

- Os recursos baseados em nuvem não dependerão de servidores do Active Directory ou serviços de diretório locais; como alternativa, as cargas de trabalho poderão ser modificadas para remover tais dependências.
- As cargas de trabalho de aplicativo ou serviço que estão sendo migradas dão suporte a mecanismos de autenticação compatíveis com provedores de identidade de nuvem ou podem ser modificadas facilmente para dar suporte a eles. Os provedores de identidade nativos de nuvem se baseiam em mecanismos de autenticação prontos para a Internet, como SAML, OAuth e OpenID Connect. Cargas de trabalho existentes dependentes de métodos de autenticação herdados que usam protocolos como Kerberos ou NTLM talvez precisem ser refatoradas antes da migração para a nuvem.

> [!TIP]
> A maioria dos serviços de identidade nativos de nuvem não é capaz de substituir totalmente os diretórios locais tradicionais. Recursos de diretório, como gerenciamento do computador ou políticas de grupo, podem ficar disponíveis apenas com o uso de ferramentas ou serviços adicionais.

Migrar completamente os serviços de identidade para um provedor baseado em nuvem elimina a necessidade de manter sua própria infraestrutura de identidade, o que simplifica bastante o gerenciamento de TI.

### <a name="directory-synchronization"></a>Sincronização de diretório

Para organizações com uma infraestrutura de identidade existente, a sincronização de diretório costuma ser a melhor solução para preservar o gerenciamento de acesso e de usuário existente, fornecendo as funcionalidades de IAM necessárias para gerenciar recursos da nuvem. O processo acima replica continuamente as informações de diretório entre a nuvem e os ambientes locais, o que permite o SSO (logon único) para os usuários e um sistema de identidade, função e permissão consistente para toda a organização.

Observação: as organizações que adotaram o Office 365 talvez já tenham implementado a [sincronização de diretório](/office365/enterprise/set-up-directory-synchronization) entre a infraestrutura do Active Directory local e o Azure Active Directory.

**Pressuposições da sincronização de diretório**. O uso de uma solução de identidade sincronizada pressupõe o seguinte:

- Você precisa manter um conjunto comum de contas de usuário e grupos na nuvem e na infraestrutura de TI local.
- Os serviços de identidade local dão suporte à replicação com seu provedor de identidade de nuvem.
- Exigência de mecanismos de SSO para usuários que acessam os provedores de identidade locais e de nuvem.

> [!TIP]
> As cargas de trabalho baseadas em nuvem que dependem de mecanismos de autenticação herdados sem suporte pelos serviços de identidade baseados em nuvem, como o Azure AD, exigirão também conectividade com os serviços de domínio locais ou servidores virtuais no ambiente de nuvem que oferece os serviços. O uso de serviços de identidade locais também introduz dependências na conectividade entre as redes locais e de nuvem.

### <a name="cloud-hosted-domain-services"></a>Serviços de domínio hospedados na nuvem

Se você tiver cargas de trabalho que dependem de autenticação baseada em declarações usando protocolos herdados, como Kerberos ou NTLM, e essas cargas de trabalho não puderem ser refatoradas para aceitar protocolos de autenticação modernos, como SAML ou OAuth e OpenID Connect, talvez seja necessário migrar alguns dos seus serviços de domínio para a nuvem como parte da implantação de nuvem.

Esse tipo de implantação envolve a implantação de máquinas virtuais que executam o Active Directory em suas redes virtuais baseadas em nuvem para fornecer serviços de domínio aos recursos na nuvem. Todos os aplicativos e serviços existentes que estão migrando para sua rede de nuvem devem poder usar os servidores de diretório hospedados na nuvem com algumas modificações.

É provável que os serviços de domínio e diretórios existentes continuem a ser usados em seu ambiente local. Nesse cenário, é recomendável que você também use a sincronização de diretório para fornecer um conjunto comum de usuários e funções em ambientes de nuvem e locais.

**Pressuposições dos serviços de domínio hospedados na nuvem**. A execução de uma migração de diretório pressupõe o seguinte:

- Suas cargas de trabalho dependem de autenticação baseada em declarações usando protocolos como Kerberos ou NTLM.
- As máquinas virtuais da carga de trabalho precisa ter ingressado no domínio para fins de gerenciamento ou aplicação da política de grupo do Active Directory.

> [!TIP]
> Embora uma migração de diretório combinada com serviços de domínio hospedados na nuvem ofereça boa flexibilidade na hora de migrar cargas de trabalho existentes, a hospedagem de máquinas virtuais na rede virtual de nuvem para fornecer esses serviços aumenta a complexidade das tarefas de gerenciamento de TI. À medida que a experiência de migração de nuvem for progredindo, examine os requisitos de manutenção de longo prazo para hospedar os servidores. Avalie se a refatoração das cargas de trabalho existentes para permitir a compatibilidade com provedores de identidade de nuvem como o Azure Active Directory pode reduzir a necessidade de servidores hospedados na nuvem.

### <a name="active-directory-federation-services"></a>Serviços de Federação do Active Directory (AD FS)

A federação de identidades estabelece relações de confiança entre vários sistemas de gerenciamento de identidades para permitir recursos de autorização e autenticação comuns. Assim, é possível dar suporte à funcionalidade de logon único em vários domínios na organização ou nos sistemas de identidade gerenciados por seus clientes ou parceiros de negócios.

O Azure AD dá suporte à federação de domínios do Active Directory locais usando os [Serviços de Federação do Active Directory](/azure/active-directory/hybrid/how-to-connect-fed-whatis) (AD FS). Confira a arquitetura de referência [Estender o AD FS ao Azure](../../../reference-architectures/identity/adfs.md) para ver como isso pode ser implementado no Azure.

## <a name="evolving-identity-integration"></a>Integração de identidade em evolução

A integração de identidade é um processo iterativo. Talvez seja melhor começar com uma solução nativa de nuvem e um pequeno conjunto de usuários e funções correspondentes para uma implantação inicial. À medida que a migração for progredindo, pense em adotar um modelo federado ou realizar uma migração de diretório total dos serviços de identidade locais para a nuvem. Reavalie sua estratégia de identidade em cada iteração do processo de migração.

## <a name="learn-more"></a>Saiba mais

Confira os itens abaixo para obter mais informações sobre os serviços de identidade na plataforma Azure.

- [Azure AD](https://azure.microsoft.com/services/active-directory). O Azure AD fornece serviços de identidade baseados em nuvem. Ele permite que você gerencie o acesso a seus recursos do Azure e controle o gerenciamento de identidades, o registro de dispositivos, o provisionamento de usuário, o controle de acesso do aplicativo e a proteção de dados.
- [Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity). A ferramenta Azure AD Connect permite que você se conecte a instâncias do Azure AD com suas soluções de gerenciamento de identidades existente, permitindo a sincronização do diretório existente com a nuvem.
- [RBAC](/azure/role-based-access-control/overview) (Controle de Acesso Baseado em Função). O Azure AD fornece RBAC para gerenciar o acesso aos recursos no plano de gerenciamento com eficiência e segurança. Os trabalhos e as responsabilidades são organizados em funções; os usuários são atribuídos a essas funções. O RBAC permite controlar quem tem acesso a um recurso e quais ações um usuário pode executar em tal recurso.
- [Azure AD PIM](/azure/active-directory/privileged-identity-management/pim-configure) (Privileged Identity Management). O PIM reduz o tempo de exposição dos privilégios de acesso a recursos e aumenta a visibilidade em relação a seu uso por meio de relatórios e alertas. Ele limita os usuários para só poderem usar seus privilégios JIT (“Just-In-Time”) ou atribuindo privilégios por uma duração reduzida, após a qual eles são revogados automaticamente.
- [Integrar domínios do Active Directory local ao Azure Active Directory](../../../reference-architectures/identity/azure-ad.md). A presente arquitetura de referência fornece um exemplo de sincronização de diretório entre domínios do Active Directory locais e o Azure AD.
- [Estender o AD DS (Active Directory Domain Services) ao Azure.](../../../reference-architectures/identity/adds-extend-domain.md) A presente arquitetura de referência fornece um exemplo de implantação de servidores do AD DS para estender os serviços de domínio a recursos baseados em nuvem.
- [Estender o AD FS (Serviços de Federação do Active Directory) ao Azure](../../../reference-architectures/identity/adfs.md). A presente arquitetura de referência configura os Serviços de Federação do Active Directory (AD FS) para realizar a autenticação federada e a autorização com seu diretório do Azure AD.

## <a name="next-steps"></a>Próximas etapas

Saiba como implementar a imposição de política na nuvem.

> [!div class="nextstepaction"]
> [Imposição de política](../policy-enforcement/overview.md)
