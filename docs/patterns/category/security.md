---
title: Padrões de segurança
titleSuffix: Cloud Design Patterns
description: A segurança é a capacidade de um sistema impedir ações acidentais ou mal-intencionadas fora do uso projetado e evitar a divulgação ou a perda de informações. Os aplicativos de nuvem são expostos na Internet fora dos limites de locais confiáveis e geralmente são abertos ao público e podem atender a usuários não confiáveis. Os aplicativos devem ser criados e implantados de maneira a proteger contra ataques mal-intencionados, restringem o acesso a somente usuários aprovados e protegem os dados confidenciais.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 259d4bd5178ca8ab85df0cd03e5268c5a7845a3a
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009433"
---
# <a name="security-patterns"></a><span data-ttu-id="f8c97-106">Padrões de segurança</span><span class="sxs-lookup"><span data-stu-id="f8c97-106">Security patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="f8c97-107">A segurança é a capacidade de um sistema impedir ações acidentais ou mal-intencionadas fora do uso projetado e evitar a divulgação ou a perda de informações.</span><span class="sxs-lookup"><span data-stu-id="f8c97-107">Security is the capability of a system to prevent malicious or accidental actions outside of the designed usage, and to prevent disclosure or loss of information.</span></span> <span data-ttu-id="f8c97-108">Os aplicativos de nuvem são expostos na Internet fora dos limites de locais confiáveis e geralmente são abertos ao público e podem atender a usuários não confiáveis.</span><span class="sxs-lookup"><span data-stu-id="f8c97-108">Cloud applications are exposed on the Internet outside trusted on-premises boundaries, are often open to the public, and may serve untrusted users.</span></span> <span data-ttu-id="f8c97-109">Os aplicativos devem ser criados e implantados de maneira a proteger contra ataques mal-intencionados, restringem o acesso a somente usuários aprovados e protegem os dados confidenciais.</span><span class="sxs-lookup"><span data-stu-id="f8c97-109">Applications must be designed and deployed in a way that protects them from malicious attacks, restricts access to only approved users, and protects sensitive data.</span></span>

|                    <span data-ttu-id="f8c97-110">Padrão</span><span class="sxs-lookup"><span data-stu-id="f8c97-110">Pattern</span></span>                     |                                                                                                         <span data-ttu-id="f8c97-111">Resumo</span><span class="sxs-lookup"><span data-stu-id="f8c97-111">Summary</span></span>                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="f8c97-112">Identidade Federada</span><span class="sxs-lookup"><span data-stu-id="f8c97-112">Federated Identity</span></span>](../federated-identity.md) |                                                                                <span data-ttu-id="f8c97-113">Delegar autenticação a um provedor de identidade externa.</span><span class="sxs-lookup"><span data-stu-id="f8c97-113">Delegate authentication to an external identity provider.</span></span>                                                                                |
|         [<span data-ttu-id="f8c97-114">Gatekeeper</span><span class="sxs-lookup"><span data-stu-id="f8c97-114">Gatekeeper</span></span>](../gatekeeper.md)         | <span data-ttu-id="f8c97-115">Proteger aplicativos e serviços usando uma instância de host dedicado que atua como intermediário entre clientes e o aplicativo ou serviço, valida e corrige solicitações e passa solicitações e dados entre eles.</span><span class="sxs-lookup"><span data-stu-id="f8c97-115">Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service, validates and sanitizes requests, and passes requests and data between them.</span></span> |
|          [<span data-ttu-id="f8c97-116">Valet Key</span><span class="sxs-lookup"><span data-stu-id="f8c97-116">Valet Key</span></span>](../valet-key.md)          |                                                        <span data-ttu-id="f8c97-117">Use um token ou chave que fornece aos clientes acesso direto e restrito a um determinado recurso ou serviço.</span><span class="sxs-lookup"><span data-stu-id="f8c97-117">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span>                                                        |
