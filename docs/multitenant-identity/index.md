---
title: Gerenciamento de Identidades para Aplicativos Multilocatários
description: Práticas recomendadas para autenticação, autorização e gerenciamento de identidades em aplicativos multilocatários.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: be906106fb12c381d57ad40ae22e748dcff9722f
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58346069"
---
# <a name="manage-identity-in-multitenant-applications"></a>Gerenciar a identidade em aplicativos multilocatários

Esta série de artigos descreve as práticas recomendadas para multilocação, ao usar o Azure AD para autenticação e gerenciamento de identidades.

[Código de exemplo do ![GitHub](../_images/github.png)][sample-application]

Quando estiver compilando um aplicativo multilocatário, um dos primeiros desafios é gerenciar identidades de usuário, porque agora cada usuário pertence a um locatário. Por exemplo: 

- Os usuários entram com suas credenciais organizacionais.
- Eles devem ter acesso aos dados de sua organização, mas não aos dados que pertencem a outros locatários.
- Uma organização pode se inscrever para obter o aplicativo e, em seguida, atribuir funções do aplicativo a seus membros.

O Azure AD (Azure Active Directory) tem alguns excelentes recursos que dão suporte a todos esses cenários.

Para acompanhar esta série de artigos, criamos uma [implementação de ponta a ponta][sample-application] completa de um aplicativo multilocatário. Os artigos refletem o que aprendemos no processo de compilação do aplicativo. Para começar a usar o aplicativo, veja [Executar o aplicativo Surveys][running-the-app].

## <a name="introduction"></a>Introdução

Digamos que você esteja gravando um aplicativo SaaS corporativo para ser hospedado na nuvem. É óbvio que o aplicativo terá usuários:

![Usuários](./images/users.png)

Mas esses usuários pertencem a organizações:

![Usuários da organização](./images/org-users.png)

Exemplo: A Tailspin vende assinaturas para seu aplicativo SaaS. A Contoso e a Fabrikam inscrevem-se no aplicativo. Quando Alice (`alice@contoso`) entra, o aplicativo deve saber que Alice faz parte da Contoso.

- Alice *deve* ter acesso aos dados da Contoso.
- Alice *não deve* ter acesso aos dados da Fabrikam.

Essa diretriz mostra como gerenciar as identidades dos usuários em um aplicativo multilocatário, usando o [Azure Active Directory (Azure AD)](/azure/active-directory) para lidar com a entrada e a autenticação.

<!-- markdownlint-disable MD026 -->

## <a name="what-is-multitenancy"></a>O que é a multilocação?

<!-- markdownlint-enable MD026 -->

Um *locatário* é um grupo de usuários. Em um aplicativo SaaS, o locatário é um assinante ou o cliente do aplicativo. *Multilocação* é uma arquitetura onde vários locatários compartilham a mesma instância física do aplicativo. Embora locatários compartilhem recursos físicos (como VMs ou armazenamento), cada locatário obtém sua própria instância lógica do aplicativo.

Normalmente, os dados de aplicativo são compartilhados entre os usuários dentro de um locatário, mas não com outros locatários.

![Multilocatário](./images/multitenant.png)

Compare essa arquitetura com uma arquitetura de locatário único, onde cada locatário tem uma instância física dedicada. Em uma arquitetura de locatário único, você pode adicionar locatários gerando novas instâncias do aplicativo.

![Locatário único](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a>Multilocação e dimensionamento horizontal

É comum adicionar mais instâncias para dimensionar na nuvem. Isso é conhecido como *escalonamento horizontal* ou *expansão*. Considere um aplicativo Web. Para lidar com mais tráfego, você pode adicionar mais VMs de servidor e colocá-las atrás de um balanceador de carga. Cada VM executa uma instância física separada do aplicativo Web.

![Balancear a carga de um site](./images/load-balancing.png)

Qualquer solicitação pode ser roteada para qualquer instância. Em conjunto, o sistema funciona como uma única instância lógica. Você pode subdividir uma VM ou criar uma nova VM sem afetar os usuários. Nesta arquitetura, cada instância física é multilocatário e você pode dimensionar adicionando mais instâncias. Se uma instância falhar, não deverá afetar nenhum locatário.

## <a name="identity-in-a-multitenant-app"></a>Identidade em um aplicativo multilocatário

Em um aplicativo multilocatário, você deve considerar os usuários no contexto de locatários.

### <a name="authentication"></a>Authentication

- Os usuários entram no aplicativo com suas credenciais da organização. Eles não precisam criar novos perfis de usuário para o aplicativo.
- Os usuários dentro da mesma organização fazem parte do mesmo locatário.
- Quando um usuário faz logon, o aplicativo sabe a qual locatário o usuário pertence.

### <a name="authorization"></a>Autorização

- Ao autorizar ações de um usuário (digamos, exibição de um recurso), o aplicativo deve levar em conta o locatário do usuário.
- Os usuários podem ser atribuídos a funções dentro do aplicativo, como "Admin" ou "Usuário Padrão". As atribuições de função devem ser gerenciadas pelo cliente e não pelo provedor de SaaS.

**Exemplo:** Alice, uma funcionária da Contoso, navega para o aplicativo em seu navegador e clica no botão "Entrar". Ela é redirecionada para uma tela de logon onde insere suas credenciais corporativas (nome de usuário e senha). Neste ponto, ele está conectada ao aplicativo como `alice@contoso.com`. O aplicativo também sabe que Alice é um usuário administrador deste aplicativo. Por ser administrador, ela pode ver uma lista de todos os recursos que pertencem à Contoso. No entanto, ela não poderá exibir recursos da Fabrikam pois é admin somente dentro de seu locatário.

Neste guia, vamos examinar especificamente o uso do Azure AD para gerenciamento de identidades.

- Supomos que o cliente armazene seus perfis de usuário no Azure AD (incluindo locatários do Office 365 e Dynamics CRM)
- Os clientes com Active Directory local podem usar o [Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity) para sincronizar o Active Directory local com o Azure AD. Se um cliente com o Active Directory local não conseguir usar o Azure AD Connect (devido à política de TI corporativa ou por outros motivos), o provedor de SaaS poderá federar com o diretório do cliente por meio dos Serviços de Federação do Active Directory (AD FS). Essa opção é descrita em [Federar com o AD FS de um cliente](adfs.md).

Este guia não considera outros aspectos da multilocação, como particionamento de dados, configuração por locatário e assim por diante.

[**Avançar**](./tailspin.md)

<!-- links -->

[sample-application]: https://github.com/mspnp/multitenant-saas-guidance
[running-the-app]: ./run-the-app.md
