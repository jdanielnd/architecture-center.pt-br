---
title: 'CAF: Redes definidas por software - nuvem nativa'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Discussão dos serviços de rede virtual nativa da nuvem
author: rotycenh
ms.openlocfilehash: c6200491bc9ba35a9f00e0003e51716b58628980
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900182"
---
# <a name="software-defined-networks-cloud-native"></a>Redes definidas por software: Nativo de nuvem

Uma rede virtual nativa de nuvem é necessária ao implantar recursos de IaaS, como máquinas virtuais em uma plataforma de nuvem. Acesso a redes virtuais de fontes externas, semelhantes à web, precisa ser provisionado explicitamente. Esses tipos de redes virtuais dão suporte a criação de sub-redes, regras de roteamento e firewall virtual e dispositivos de gerenciamento de tráfego.

Uma rede virtual nativa de nuvem não tem dependências no local da sua organização ou outros recursos que não são nuvem para dar suporte as cargas de trabalho hospedadas na nuvem. Todos os recursos necessários são provisionados na rede virtual em si ou por meio de ofertas de PaaS gerenciadas.

## <a name="cloud-native-assumptions"></a>Suposições sobre a nuvem nativa

Implantar uma rede virtual nativa de nuvem supõe o seguinte:

- As cargas de trabalho que você implantar a rede virtual não têm dependências em aplicativos ou serviços que são acessíveis somente de dentro de sua rede local. A menos que forneçam pontos de extremidade acessíveis pela internet pública, aplicativos e serviços hospedados internamente em locais não são utilizáveis por recursos hospedados em uma plataforma de nuvem.
- Seu gerenciamento de identidade de carga de trabalho e controle de acesso depende dos serviços de identidade da plataforma ou servidores IaaS hospedados no seu ambiente de nuvem. Você não precisará se conectar diretamente aos serviços de identidade hospedados no local ou em locais externos.
- Os serviços de identidade não precisam dar suporte a logon único (SSO) com diretórios locais.

Redes virtuais nativas de nuvem não têm nenhuma dependência externa. Isso os torna simples para implantar e configurar, e como o resultado dessa arquitetura é geralmente a melhor escolha para experimentos ou outras implantações menores independentes ou iteradas rapidamente.

Problemas adicionais que sua equipe de adoção de nuvem deve considerar ao discutir uma arquitetura de rede virtual nativa da nuvem incluem:

- Cargas de trabalho existentes, projetadas para serem executadas em um datacenter local, talvez seja necessário uma extensa modificação para tirar proveito da funcionalidade baseada em nuvem, como os serviços de armazenamento ou autenticação.
- Redes em nuvem nativas são gerenciadas exclusivamente por meio das ferramentas de gerenciamento da plataforma de nuvem e, portanto, podem levar a divergência de política e o gerenciamento de seus padrões de TI existentes com o tempo.

## <a name="learn-more"></a>Saiba mais

Consulte o seguinte para obter mais informações sobre a rede virtual nativa de nuvem na plataforma do Azure.

- [Rede Virtual do Azure: Guias de instruções](/azure/virtual-network/virtual-network-vnet-plan-design-arm). Redes Virtuais do Azure recentemente criadas são nativas de nuvem por padrão. Use esses guias para ajudar a planejar a criação e implantação de suas redes virtuais.
- [Limites de assinatura: Rede](/azure/azure-subscription-service-limits?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits). Qualquer rede virtual única e os recursos conectados só podem existir em uma única assinatura e são associados por limites de assinatura.
