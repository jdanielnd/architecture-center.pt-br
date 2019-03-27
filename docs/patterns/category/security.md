---
title: Padrões de segurança
titleSuffix: Cloud Design Patterns
description: A segurança é a capacidade de um sistema impedir ações acidentais ou mal-intencionadas fora do uso projetado e evitar a divulgação ou a perda de informações. Os aplicativos de nuvem são expostos na Internet fora dos limites de locais confiáveis e geralmente são abertos ao público e podem atender a usuários não confiáveis. Os aplicativos devem ser criados e implantados de maneira a proteger contra ataques mal-intencionados, restringem o acesso a somente usuários aprovados e protegem os dados confidenciais.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: c18255dccdbd8659705884a4141f5ecd099d9ece
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58248337"
---
# <a name="security-patterns"></a><span data-ttu-id="fe459-106">Padrões de segurança</span><span class="sxs-lookup"><span data-stu-id="fe459-106">Security patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="fe459-107">A segurança é a capacidade de um sistema impedir ações acidentais ou mal-intencionadas fora do uso projetado e evitar a divulgação ou a perda de informações.</span><span class="sxs-lookup"><span data-stu-id="fe459-107">Security is the capability of a system to prevent malicious or accidental actions outside of the designed usage, and to prevent disclosure or loss of information.</span></span> <span data-ttu-id="fe459-108">Os aplicativos de nuvem são expostos na Internet fora dos limites de locais confiáveis e geralmente são abertos ao público e podem atender a usuários não confiáveis.</span><span class="sxs-lookup"><span data-stu-id="fe459-108">Cloud applications are exposed on the Internet outside trusted on-premises boundaries, are often open to the public, and may serve untrusted users.</span></span> <span data-ttu-id="fe459-109">Os aplicativos devem ser criados e implantados de maneira a proteger contra ataques mal-intencionados, restringem o acesso a somente usuários aprovados e protegem os dados confidenciais.</span><span class="sxs-lookup"><span data-stu-id="fe459-109">Applications must be designed and deployed in a way that protects them from malicious attacks, restricts access to only approved users, and protects sensitive data.</span></span>

|                    <span data-ttu-id="fe459-110">Padrão</span><span class="sxs-lookup"><span data-stu-id="fe459-110">Pattern</span></span>                     |                                                                                                         <span data-ttu-id="fe459-111">Resumo</span><span class="sxs-lookup"><span data-stu-id="fe459-111">Summary</span></span>                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="fe459-112">Identidade Federada</span><span class="sxs-lookup"><span data-stu-id="fe459-112">Federated Identity</span></span>](../federated-identity.md) |                                                                                <span data-ttu-id="fe459-113">Delegar autenticação a um provedor de identidade externa.</span><span class="sxs-lookup"><span data-stu-id="fe459-113">Delegate authentication to an external identity provider.</span></span>                                                                                |
|         [<span data-ttu-id="fe459-114">Gatekeeper</span><span class="sxs-lookup"><span data-stu-id="fe459-114">Gatekeeper</span></span>](../gatekeeper.md)         | <span data-ttu-id="fe459-115">Proteger aplicativos e serviços usando uma instância de host dedicado que atua como intermediário entre clientes e o aplicativo ou serviço, valida e corrige solicitações e passa solicitações e dados entre eles.</span><span class="sxs-lookup"><span data-stu-id="fe459-115">Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service, validates and sanitizes requests, and passes requests and data between them.</span></span> |
|          [<span data-ttu-id="fe459-116">Valet Key</span><span class="sxs-lookup"><span data-stu-id="fe459-116">Valet Key</span></span>](../valet-key.md)          |                                                        <span data-ttu-id="fe459-117">Use um token ou chave que fornece aos clientes acesso direto e restrito a um determinado recurso ou serviço.</span><span class="sxs-lookup"><span data-stu-id="fe459-117">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span>                                                        |
