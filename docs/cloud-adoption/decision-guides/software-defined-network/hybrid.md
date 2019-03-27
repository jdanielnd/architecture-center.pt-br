---
title: 'CAF: Redes Definidas pelo Software - Rede híbrida'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Discussão sobre como as redes híbridas permitem que as redes virtuais em nuvem conectem-se a recursos locais
author: rotycenh
ms.openlocfilehash: 02d181db0ae9baef3b453b8623d212b624f6b16a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243357"
---
# <a name="software-defined-networks-hybrid-network"></a>Redes Definidas pelo Software: Rede híbrida

A arquitetura de rede em nuvem híbrida permite que as redes virtuais acessem os recursos e serviços locais e vice-versa, usando uma Conexão WAN dedicada como o ExpressRoute ou outro método de conexão para conectar diretamente as redes.

![Rede híbrida](../../../reference-architectures/hybrid-networking/images/expressroute.png)

Com base na arquitetura de rede virtual nativa de nuvem, uma rede virtual híbrida é isolada quando criada inicialmente. Adicionar conectividade ao ambiente local concede acesso de e para a rede local, embora todos os outros recursos de direcionamento de tráfego de entrada na rede virtual precisem ser explicitamente permitidos. É possível proteger a conexão usando dispositivos de firewall virtuais e regras de roteamento para limitar o acesso, especificar exatamente quais serviços podem ser acessados entre as duas redes usando recurso de roteamento nativo de nuvem, ou implantando NVAs (dispositivos virtuais de rede) para gerenciar o tráfego.

Embora a arquitetura de rede híbrida dê suporte a conexões VPN, as Conexões WAN dedicadas, como o ExpressRoute, geralmente são preferidas devido ao desempenho mais alto e maior segurança.

## <a name="hybrid-assumptions"></a>Suposições sobre a rede híbrida

Implantar uma rede virtual híbrida supõe o seguinte:

- As equipes de segurança de TI alinham a política de segurança de rede local e baseada em nuvem para garantir que as redes virtuais baseadas em nuvem sejam confiáveis para a comunicação direta com os sistemas locais.
- As cargas de trabalho baseadas em nuvem exigem acesso a armazenamento, aplicativos e serviços hospedados nas redes locais ou de terceiros, ou os usuários ou aplicativos locais precisam de acesso a recursos hospedados em nuvem.
- Você precisa migrar aplicativos e serviços existentes que dependem de recursos locais, mas não quer gastar os recursos em redesenvolvimento para remover essas dependências.
- A implementação de uma VPN ou Conexão WAN dedicada entre as redes locais e o provedor de nuvem não é impedida pela política corporativa, pelos requisitos regulamentares ou problemas de compatibilidade técnica.
- As cargas de trabalho não exigem várias assinaturas para ignorar os limites de recursos de assinatura, OU as cargas de trabalho envolvem várias assinaturas, mas não requerem gerenciamento central de conectividade ou serviços compartilhados usados por recursos distribuídos por várias assinaturas.

A equipe de Adoção de Nuvem deverá considerar os seguintes problemas ao analisar a implementação de uma arquitetura de rede virtual híbrida:

- Conectar redes locais com redes em nuvem aumenta a complexidade dos requisitos de segurança. Ambas as redes precisam ser protegidas contra vulnerabilidades externas e acesso não autorizado de ambos os lados do ambiente híbrido.
- O dimensionamento do número e tamanho das cargas de trabalho em um ambiente de nuvem híbrida pode adicionar uma complexidade significativa ao gerenciamento de tráfego e roteamento.
- Será necessário desenvolver políticas de gerenciamento e controle de acesso compatíveis para manter uma governança consistente em toda a organização.

## <a name="learn-more"></a>Saiba mais

Consulte o seguinte para obter mais informações sobre a rede híbrida na plataforma do Azure.

- [Arquitetura de referência de rede híbrida](../../../reference-architectures/hybrid-networking/expressroute.md). As redes virtuais híbridas do Azure usam um circuito de ExpressRoute ou uma VPN do Azure para conectar a rede virtual aos ativos de TI hospedados não Azure existentes da organização. Este artigo aborda as opções para criar uma rede híbrida no Azure.
