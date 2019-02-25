---
title: 'CAF: Guias de decisão de arquitetura'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Saiba mais sobre guias de decisão de arquitetura no Cloud Adoption Framework.
author: rotycenh
ms.openlocfilehash: b43047b5fc5b636bc84b9f28ec24846730e63672
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900359"
---
# <a name="architectural-decision-guides"></a>Guias de decisão de arquitetura

Guias de decisão de arquitetura em que a estrutura de adoção da nuvem descrevem padrões e modelos que ajudam durante a criação de diretrizes de design de governança de nuvem. Cada guia de decisão se concentra em um componente de infraestrutura de núcleo de implantações de nuvem e lista potenciais padrões ou modelos que se destinam a dar suporte a cenários de implantação de nuvem específica.

Quando você começa a estabelecer a governança de nuvem para sua organização, percursos de governança acionáveis fornecem um roteiro de linha de base. No entanto, esses percursos fazem suposições sobre os requisitos e as prioridades que podem não refletir àquelas de sua organização.
Esses guias de decisão complementam os percursos de governança de exemplo, fornecendo padrões alternativos e modelos que ajudam você a alinhar as escolhas de design arquitetônico feitas nas diretrizes de design de exemplo pelos seus próprios requisitos.

## <a name="design-guidance-categories"></a>Categorias de diretrizes de design

Cada uma das categorias a seguir representa uma tecnologia de base de todas as implantações de nuvem. Para cada opção nos percursos do design de exemplo que não correspondem aos seus requisitos, examine as opções na seção relevante abaixo para escolher um padrão ou o modelo melhor adequado às suas necessidades.

[Assinaturas](./subscriptions/overview.md): Planeje a estrutura de design e conta de assinatura da sua implantação para corresponder com a propriedade da sua organização, faturamento e capacidades de gerenciamento.

[Identidade](./identity/overview.md): Integre os serviços de identidade baseado em nuvem aos seus recursos de identidade existentes para dar suporte ao controle de acesso em seu ambiente de TI.

[Imposição de Política](./policy-enforcement/overview.md): Definir e impor regras de política organizacional de recursos e cargas de trabalho que você implanta na nuvem.

[Consistência de Recursos](./resource-consistency/overview.md): Certifique-se que implantação e a organização dos seus recursos baseados em nuvem se alinham para impor requisitos de gerenciamento e a política de recursos.

[Marcação de Recursos](./resource-tagging/overview.md): Organize seus recursos baseados em nuvem para dar suporte a modelos de cobrança, abordagens de contabilidade da nuvem, gerenciamento, controle de acesso e eficiência operacional. A marcação de recursos exige um esquema de nomenclatura e de metadados consistente e bem organizado.

[Redes definidas por software](./software-defined-network/overview.md): Implante cargas de trabalho seguras para a nuvem usando a implantação rápida e modificação de recursos de rede virtualizados. Redes definidas por software (SDNs) podem dar suporte a fluxos de trabalho ágeis, isolar os recursos e integrar sistemas baseados em nuvem com sua infraestrutura de TI existente.

[Criptografia](./encryption/overview.md): Proteja seus dados confidenciais usando a criptografia, um aspecto importante da segurança dentro de uma implantação de nuvem.

[Logs e relatórios](./log-and-report/overview.md): Monitorar dados de log gerados pelos recursos baseados em nuvem. A análise de dados fornece insights relacionados à integridade nas operações, a manutenção e o status de imposição da política de cargas de trabalho.

## <a name="next-steps"></a>Próximas etapas

Saiba como as assinaturas e contas servem como a base de uma implantação de nuvem.

> [!div class="nextstepaction"]
> [Design de assinaturas](subscriptions/overview.md)
