---
title: 'CAF: Redes definidas por software - somente PaaS'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Discussão sobre o modelo de PaaS somente para a nuvem com base em funcionalidade de rede
author: rotycenh
ms.openlocfilehash: 2f3f82d781ddb6544721e82e7b7d795222a2f8ff
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900290"
---
# <a name="software-defined-networks-paas-only"></a>Redes definidas por software: PaaS-only

Quando você implementa uma plataforma como um recurso de serviço (PaaS), o processo de implantação cria automaticamente uma rede subjacente assumida com um número limitado de controles na rede, incluindo balanceamento de carga, bloqueio de porta e conexões com outros serviços PaaS.

No Azure, vários tipos de recursos de PaaS podem ser [implantados](/azure/virtual-network/virtual-network-for-azure-services) ou [conectados a](/azure/virtual-network/virtual-network-service-endpoints-overview) uma rede virtual, permitindo que esses recursos integrem sua infraestrutura de rede virtual existente. No entanto, em muitos casos, uma arquitetura de rede apenas PaaS, conta somente com esses recursos de rede fornecidos originalmente pelos recursos PaaS, é suficiente para atender aos requisitos de carga de trabalho.

Se você estiver considerando uma arquitetura de rede somente de PaaS, certifique-se de que validar as suposições necessárias se alinha suas necessidades.

## <a name="paas-only-assumptions"></a>Suposições somente PaaS

Implantação de uma arquitetura de rede somente PaaS pressupõe o seguinte:

- O aplicativo que está sendo implantado é um aplicativo autônomo ou depende apenas de outros recursos de PaaS.
- As equipes de operações de TI podem atualizar suas ferramentas, treinamento e processos para dar suporte ao gerenciamento, configuração e implantação de aplicativos de PaaS autônomo.
- O aplicativo de PaaS não faz parte de um esforço de migração de nuvem mais abrangente que inclui recursos de IaaS.

Essas suposições são qualificadores mínimos alinhados ao implantar uma rede somente PaaS. Embora essa abordagem pode se alinhe com os requisitos de implantação de um único aplicativo, sua equipe de adoção de nuvem deve examinar essas perguntas de longo prazo:

- Essa implantação será expandida no escopo ou escala para exigir acesso a outros recursos sem PaaS?
- Outras implantações de PaaS estão planejadas além da solução atual?
- A organização tem planos para outras migrações de nuvem futuras?

As respostas a essas perguntas não impediriam que uma equipe escolhesse uma única opção PaaS, mas deveriam ser considerada antes de tomar uma decisão final.
