---
title: 'CAF: Ferramentas de Linha de Base de Segurança no Azure'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Explicação sobre as ferramentas que podem facilitar a melhoria da Linha de Base de Segurança no Azure
author: BrianBlanchard
ms.openlocfilehash: b316626c8ad717514f7f592abefa0f33a92afdca
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243167"
---
# <a name="security-baseline-tools-in-azure"></a>Ferramentas de Linha de Base de Segurança no Azure

A [Linha de Base de Segurança](overview.md) é uma das [Cinco Disciplinas de Governança de Nuvem](../governance-disciplines.md). Essa disciplina concentra-se em maneiras de estabelecer políticas que protejam a rede, ativos e, mais importante, os dados que residirão na solução de um Provedor de Nuvem. Dentro das cinco disciplinas de Governança de Nuvem, a Linha de Base de Segurança inclui classificação dos dados e propriedade digital. Adicionalmente, inclui documentação de riscos, tolerância comercial e estratégias de mitigação associadas à segurança dos dados, ativos e rede. De uma perspectiva técnica, inclui também o envolvimento nas decisões relativas à [criptografia](../../decision-guides/encryption/overview.md), [requisitos de rede](../../decision-guides/software-defined-network/overview.md), [estratégias de identidade híbrida](../../decision-guides/identity/overview.md) e ferramentas para [automatizar o cumprimento](../../decision-guides/policy-enforcement/overview.md) de políticas de segurança em [grupos de recursos](../../decision-guides/resource-consistency/overview.md).

A seguir, é apresentada uma lista de ferramentas do Azure que podem ajudar a consolidar políticas e processos que dão suporte à Linha de Base de Segurança.

|                                                            | [Portal do Azure](https://azure.microsoft.com/features/azure-portal/) / [Resource Manager](/azure/azure-resource-manager/resource-group-overview)  | [Cofre da Chave do Azure](/azure/key-vault)  | [Azure AD](/azure/active-directory/fundamentals/active-directory-whatis) | [Azure Policy](/azure/governance/policy/overview) | [Central de Segurança do Azure](/azure/security-center/security-center-intro) | [Azure Monitor](/azure/azure-monitor/overview) |
|------------------------------------------------------------|---------------------------------|-----------------|----------|--------------|-----------------------|---------------|
| Aplicar controles de acesso a recursos e criação de recursos   | Sim                             | Não               | Sim      | Não           | Não                    | Não             |
| Redes virtuais seguras                                    | Sim                             | Não              | Não        | Sim          | Não                    | Não             |
| Criptografar unidades virtuais                                     | Não                               | Sim             | Não       | Não           | Não                    | Não             |
| Criptografar armazenamento e bancos de dados de PaaS                         | Não                               | Sim             | Não       | Não           | Não                    | Não             |
| Gerenciar serviços de identidade híbrida                            | Não                               | Não               | Sim      | Não           | Não                    | Não             |
| Restringir tipos de recursos permitidos                         | Não                               | Não              | Não        | Sim          | Não                    | Não             |
| Impor restrições geo-regionais                          | Não                               | Não              | Não        | Sim          | Não                    | Não             |
| Monitorar a integridade da segurança de redes e recursos          | Não                               | Não              | Não       | Não            | sim                   | Sim           |
| Detectar atividade mal-intencionada                                  | Não                               | Não              | Não       | Não            | sim                   | Sim           |
| Detectar vulnerabilidades antecipadamente                        | Não                               | Não              | Não       | Não            | Sim                   | Não             |
| Configurar backup e recuperação de desastre                     | Sim                             | Não              | Não       | Não           | Não                    | Não             |

Para obter uma lista completa de ferramentas e serviços de segurança do Azure, consulte [Serviços de segurança e tecnologias disponíveis no Azure](/azure/security/azure-security-services-technologies).

Também é extremamente comum que os clientes utilizem ferramentas de terceiros para facilitar as atividades de Linha de Base de Segurança. Para obter mais informações, consulte o artigo [Integrar soluções de segurança na Central de Segurança do Azure](/azure/security-center/security-center-partner-integration).

Além das ferramentas de segurança, a [Central de Confiabilidade da Microsoft](https://www.microsoft.com/trustcenter/guidance/risk-assessment) contém diretrizes abrangentes, relatórios e documentação relacionada que podem ajudá-lo a realizar avaliações de risco como parte do processo de planejamento de migração.
