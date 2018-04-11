---
title: 'Diretrizes: design de locatário do Azure AD'
description: Diretrizes de design de locatário do Azure como parte de uma estratégia básica de adoção da nuvem
author: telmosampaio
ms.openlocfilehash: 9ac52e9fd44bd8b9c777625002d5960f4f269be2
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/27/2018
---
# <a name="guidance-azure-ad-tenant-design"></a>Diretrizes: design de locatário do Azure AD

Um locatário do Azure AD fornece serviços de identidade digital e namespaces usados para uma ou mais [assinaturas do Azure](subscription-explainer.md). Se estiver seguindo a descrição básica de adoção, você já aprendeu [a obter um locatário do Azure AD][how-to-get-aad-tenant]. 

## <a name="design-considerations"></a>Considerações sobre o design

- No estágio básico de adoção, você pode começar com um único locatário do Azure AD. Caso sua organização tenha uma assinatura do Office 365 ou uma assinatura do Azure, você já tem um locatário do Azure AD que poderá usar. Caso não tenha nenhuma dessas, saiba mais sobre [como obter um locatário do Azure AD][how-to-get-aad-tenant]. 
- Nos estágios intermediário e avançado de adoção, você aprenderá a sincronizar ou federar diretórios locais com o Azure AD. Isso permitirá que você use a identidade digital local no Azure AD. No entanto, no estágio básico, você adicionará novos usuários que têm identidade apenas no locatário único do Azure AD. Você será responsável por gerenciar essas identidades. Por exemplo, você precisará integrar novos usuários do Azure AD, transferir os usuários do Azure AD que você não deseja mais que tenham acesso aos recursos do Azure e outras alterações nas permissões de usuário.

## <a name="next-steps"></a>Próximas etapas

* Agora que você tem um locatário do Azure AD, saiba [como adicionar um usuário][azure-ad-add-user]. Depois de adicionar um ou mais novos usuários ao locatário do Azure AD, a próxima etapa é aprender sobre [assinaturas do Azure](subscription-explainer.md).

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
