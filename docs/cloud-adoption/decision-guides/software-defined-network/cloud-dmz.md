---
title: 'CAF: Redes definidas pelo software - DMZ de Nuvem'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Essa arquitetura de rede permite acesso limitado entre as redes locais e baseadas em nuvem
author: rotycenh
ms.openlocfilehash: a192541dcfb0f3d713f4139a2ab0541d0c7202db
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900276"
---
# <a name="software-defined-networks-cloud-dmz"></a>Redes definidas pelo software: DMZ de nuvem

A arquitetura de rede da DMZ de Nuvem permite acesso limitado entre redes locais e baseadas em nuvem, usando uma VPN (rede privada virtual) para conectar as redes. Uma DMZ é implantada na nuvem para proteger o acesso à rede local a partir de recursos baseados em nuvem.

![Proteger arquitetura de rede híbrida](../../../reference-architectures/dmz/images/dmz-private.png)

Essa arquitetura foi projetada para dar suporte a cenários em que a organização deseja iniciar a integração de cargas de trabalho baseadas em nuvem com cargas de trabalho locais, mas pode não ter políticas de segurança em nuvem totalmente desenvolvidas ou adquirido uma conexão WAN dedicada segura entre os dois ambientes. Como resultado, as redes em nuvem devem ser tratadas como uma zona desmilitarizada para garantir que os serviços locais sejam seguros.

A DMZ implanta NVAs (dispositivos virtuais de rede) para implementar a funcionalidade de segurança como firewalls e inspeção de pacotes. O tráfego que passa entre aplicativos ou serviços locais e baseados em nuvem deve passar pela DMZ onde pode ser auditado. As conexões VPN e as regras que determinam qual tráfego é permitido pela rede DMZ são estritamente controladas pelas equipes de segurança de TI.

## <a name="cloud-dmz-assumptions"></a>Suposições sobre a DMZ de Nuvem

A implantação de uma DMZ de Nuvem supõe o seguinte:

- As equipes de segurança não alinharam totalmente as exigências e políticas de segurança locais e baseadas em nuvem.
- As cargas de trabalho baseadas em nuvem exigem acesso limitado a serviços hospedados nas redes locais ou de terceiros, ou os usuários ou aplicativos no ambiente local precisam de acesso limitado a recursos hospedados em nuvem.
- A implementação de uma conexão VPN entre as redes locais e o provedor de nuvem não é impedida pela política corporativa, requisitos regulamentares ou problemas de compatibilidade técnica.
- As cargas de trabalho não exigem várias assinaturas para ignorar limites de recursos de assinatura, ou envolvem várias assinaturas mas não requerem gerenciamento central de conectividade ou serviços compartilhados usados por recursos distribuídos por várias assinaturas.

A equipe de Adoção da Nuvem deverá considerar os seguintes problemas ao analisar a implementação de uma arquitetura de rede virtual de DMZ de Nuvem:

- Conectar redes locais com redes em nuvem aumenta a complexidade dos requisitos de segurança. Embora a conexão entre as redes em nuvem e o ambiente local esteja garantida, ainda será necessário garantir que os recursos da nuvem estejam seguros.
- A arquitetura de DMZ de Nuvem normalmente é utilizada como um ponto de partida, enquanto a conectividade é mais protegida e a política de segurança é alinhada entre as redes locais e na nuvem, permitindo uma adoção mais ampla de uma arquitetura de rede híbrida em grande escala.

## <a name="learn-more"></a>Saiba mais

Consulte o seguinte para obter mais informações sobre a implementação de uma DMZ de Nuvem na plataforma do Azure.

- [Implementar uma DMZ entre o Azure e o datacenter local](../../../reference-architectures/dmz/secure-vnet-hybrid.md). Este artigo descreve como implementar uma arquitetura de rede híbrida segura no Azure.
