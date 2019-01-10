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
# <a name="security-patterns"></a>Padrões de segurança

[!INCLUDE [header](../../_includes/header.md)]

A segurança é a capacidade de um sistema impedir ações acidentais ou mal-intencionadas fora do uso projetado e evitar a divulgação ou a perda de informações. Os aplicativos de nuvem são expostos na Internet fora dos limites de locais confiáveis e geralmente são abertos ao público e podem atender a usuários não confiáveis. Os aplicativos devem ser criados e implantados de maneira a proteger contra ataques mal-intencionados, restringem o acesso a somente usuários aprovados e protegem os dados confidenciais.

|                    Padrão                     |                                                                                                         Resumo                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Identidade Federada](../federated-identity.md) |                                                                                Delegar autenticação a um provedor de identidade externa.                                                                                |
|         [Gatekeeper](../gatekeeper.md)         | Proteger aplicativos e serviços usando uma instância de host dedicado que atua como intermediário entre clientes e o aplicativo ou serviço, valida e corrige solicitações e passa solicitações e dados entre eles. |
|          [Valet Key](../valet-key.md)          |                                                        Use um token ou chave que fornece aos clientes acesso direto e restrito a um determinado recurso ou serviço.                                                        |
