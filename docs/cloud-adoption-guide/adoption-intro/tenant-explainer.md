---
title: 'Explicador: o que é um locatário do Azure Active Directory?'
description: Explica o funcionamento interno do Azure Active Directory para fornecer IDaaS (identidade como serviço) no Azure
author: petertay
ms.openlocfilehash: ce5a33b92047e1f360eee8fcbc7a726bcf8cd19f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-active-directory-tenant"></a>Explicador: o que é um locatário do Azure Active Directory?

No artigo do explicador [Como funciona o Azure?](azure-explainer.md), você aprendeu que o Azure é uma coleção de servidores e hardware de rede que executa software e hardware virtualizado em nome dos usuários. Você também aprendeu que alguns desses servidores executam um aplicativo distribuído de orquestração para gerenciar, criar, ler, atualizar e excluir recursos do Azure.

No entanto, como você esperaria, o Azure não permite que qualquer pessoa execute uma dessas operações em um recurso. O Azure restringe o acesso a essas operações usando um serviço de identidade digital confiável chamado **Azure AD** (Azure Active Directory). O Azure AD armazena o nome de usuário, senhas, dados de perfil e outras informações. Os usuários do Azure AD são segmentados em **locatários**. Um locatário é um constructo lógico que representa uma instância segura e dedicada do Azure AD geralmente associada a uma organização.

Para criar um locatário, o Azure exige uma **conta privilegiada**. Essa conta privilegiada é associada a uma conta do Azure ou a um Contrato Enterprise. Ambos são constructos de cobrança e não são armazenados no Azure AD &mdash; essas contas são armazenadas em um banco de dados de cobrança altamente seguro. 

Quando o locatário é criado, uma **ID do locatário** é gerada para o locatário e salva no banco de dados do Azure AD interno altamente seguro. Um proprietário da conta privilegiada pode então fazer logon em um portal do Azure e adicionar usuários ao locatário recém-criado do Azure AD. 

A maioria das empresas já tem, pelo menos, um serviço de gerenciamento de identidade, normalmente, o AD DS (Active Directory Domain Services). O Azure AD pode sincronizar ou federar a identidade do usuário por meio do AD DS, de modo que as empresas não precisem gerenciar a identidade separadamente nos dois ambientes. Isso será descrito mais detalhadamente nos artigos de estágio intermediário e avançado de adoção para identidade digital.

## <a name="next-steps"></a>Próximas etapas

* Agora que você aprendeu sobre locatários do Azure AD, a primeira etapa do estágio básico de adoção é saber [como obter um locatário do Azure Active Directory][how-to-get-aad-tenant]. Em seguida, leia as [diretrizes de design para locatários do Azure AD](tenant.md).

<!-- Links -->
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json