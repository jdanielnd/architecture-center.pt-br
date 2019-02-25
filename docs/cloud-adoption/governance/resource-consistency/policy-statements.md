---
title: 'CAF: Declarações da política de amostra de Consistência de Recursos'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Declarações da política de amostra de Consistência de Recursos
author: BrianBlanchard
ms.openlocfilehash: 9217ae73b0edf5b40bedac1cdc961b7a34da87d1
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900235"
---
# <a name="resource-consistency-sample-policy-statements"></a>Declarações da política de amostra de Consistência de Recursos

Declarações da política de nuvem individuais são diretrizes para tratar de riscos específicos identificados durante o processo de avaliação de riscos. Essas declarações devem fornecer um resumo conciso dos riscos e planos para lidar com eles. Cada definição de declaração deve incluir essas informações:

- Risco técnico - Um resumo do risco que essa política abordará.
- Declaração da política - Uma explicação resumida clara dos requisitos da política.
- Opções de design - Especificações, recomendações acionáveis ou outras diretrizes que as equipes de TI e os desenvolvedores podem usar ao implementar a política.

As declarações da política de amostra a seguir abordam vários riscos empresarias relacionados à Consistência de Recursos comuns e são fornecidas como exemplos para sua referência ao redigir declarações da política para atender às necessidades da sua própria organização. Observe que esses exemplos não pretendem ser proscritivos e potencialmente há várias opções de políticas para lidar com qualquer risco específico. Trabalhe em conjunto com as equipes de negócios e de TI para identificar as melhores soluções de política para seu conjunto de riscos exclusivo.

## <a name="tagging"></a>Marcação

**Risco técnico**: Sem a marcação de metadados adequada associada aos recursos implantados, as operações de TI não podem priorizar o suporte ou a otimização de recursos com base no SLA exigido, a importância das operações de negócios ou o custo operacional. Isso pode resultar em alocação incorreta de recursos de TI e possíveis atrasos na resolução de incidentes.

**Declaração da política**: As seguintes políticas serão implementadas:

- Os recursos implantados devem ser marcados com os seguintes valores: custo, criticidade, SLA e ambiente.
- O conjunto de ferramentas de governança deve validar a marcação relacionada ao custo, criticidade, SLA, aplicativo e ambiente. Todos os valores devem ser alinhados a valores predefinidos gerenciados pela equipe de governança.

**Opções possíveis de projeto**: No Azure, as [marcas de metadados de nome-valor padrão](/azure/azure-resource-manager/resource-group-using-tags) são compatíveis com a maioria dos tipos de recursos. O [Azure Policy](/azure/governance/policy/overview) é usado para impor marcas específicas como parte da criação de recursos.

## <a name="ungoverned-subscriptions"></a>Assinaturas sem governança

**Risco técnico**: A criação arbitrária de assinaturas e grupos de gerenciamento poderá resultar em seções isoladas da propriedade de nuvem que não estejam devidamente sujeitas às políticas de governança.

**Declaração da política**: A criação de novas assinaturas ou grupos de gerenciamento para aplicativos críticos ou dados protegidos exigirá uma análise da equipe de Governança de Nuvem. As alterações aprovadas serão integradas em uma atribuição de blueprint adequada.

**Opções possíveis de projeto**: Restrinja o acesso administrativo aos [Grupos de gerenciamento do Azure](/azure/governance/management-groups/) da sua organização apenas para os membros da equipe de governança aprovados que controlarão o processo de criação de assinatura e controle de acesso.

## <a name="manage-updates-to-virtual-machines"></a>Gerenciar atualizações para máquinas virtuais

**Risco técnico**: VMs (máquinas virtuais) que não estejam atualizadas com as atualizações e correções de software mais recentes ficarão vulneráveis a problemas de segurança ou desempenho, o que poderá resultar em interrupções de serviço.

**Declaração da política**: O conjunto de ferramentas de governança deve garantir que as atualizações automáticas estejam habilitadas em todas as VMs implantadas. As violações devem ser analisadas com as equipes de gerenciamento operacional e corrigidas de acordo com as políticas de operações. Os ativos que não são atualizados automaticamente devem ser incluídos nos processos pertencentes às operações de TI.

**Opções possíveis de projeto**: Para VMs hospedadas no Azure, é possível fornecer gerenciamento de atualização consistente usando a [solução de Gerenciamento de Atualizações na Automação do Azure](/azure/automation/automation-update-management).

## <a name="deployment-compliance"></a>Conformidade de implantação

**Risco técnico**: Os scripts de implantação e as ferramentas de automação que não sejam totalmente aprovados pela equipe de Governança de Nuvem poderão resultar em implantações de recursos que violam a política.

**Declaração da política**: As seguintes políticas serão implementadas:

- As ferramentas de implantação devem ser aprovadas pela equipe de Governança de Nuvem para garantir governança contínua dos ativos implantados.
- Os scripts de implantação devem ser mantidos no repositório central, acessível pela equipe de Governança de Nuvem para análise e auditoria periódicas.

**Opções possíveis de projeto**: O uso consistente do [Azure Blueprints](/azure/governance/blueprints/) para gerenciar implantações automatizadas permite implantações consistentes de recursos do Azure que aderem aos padrões e políticas de governança da sua organização.

## <a name="monitoring"></a>Monitoramento

**Risco técnico**: O monitoramento instrumentado de forma incorreta ou inconsistente pode impedir a detecção de problemas de integridade da carga de trabalho ou outras violações de conformidade com a política.

**Declaração da política**: As seguintes políticas serão implementadas:

- As ferramentas de governança devem validar que todos os ativos relacionados a aplicativos críticos ou dados protegidos sejam incluídos no monitoramento para otimização e degradação de recursos.
- As ferramentas de governança devem validar se o nível apropriado de dados de registro em log está sendo coletado para todos os aplicativos críticos ou dados protegidos.

**Opções possíveis de projeto**: O [Azure Monitor](/azure/azure-monitor/overview) é o serviço de monitoramento padrão na plataforma do Azure e o monitoramento consistente pode ser imposto por meio do uso de [Azure Blueprints](/azure/governance/blueprints/) ao implantar recursos.

## <a name="disaster-recovery"></a>Recuperação de desastre

**Risco técnico**: Falhas, exclusões ou corrupção de recursos podem resultar na interrupção de serviços ou aplicativos críticos e na perda de dados confidenciais.

**Declaração da política**: Todos os aplicativos críticos e dados protegidos devem ter soluções de backup e recuperação implementadas para minimizar o impacto nos negócios decorrente de interrupções ou falhas do sistema.

**Opções possíveis de projeto**: O serviço [Azure Site Recovery] fornece recursos de backup, recuperação e replicação destinados a minimizar a duração da interrupção em cenários de BCDR (Continuidade dos Negócios e Recuperação de Desastres).

## <a name="next-steps"></a>Próximas etapas

Use as amostras mencionadas neste artigo como ponto de partida para desenvolver políticas que abordem os riscos empresariais específicos que alinham-se aos seus planos de adoção de nuvem.

Para começar a desenvolver suas próprias declarações da política personalizadas relacionadas à Consistência de Recursos, baixe o [modelo de Consistência de Recursos](template.md).

Para acelerar a adoção dessa disciplina, escolha o [Percurso de Governança Acionável](../journeys/overview.md) que mais alinha-se ao seu ambiente. Em seguida, modifique o projeto para incorporar as decisões específicas da política corporativa.

> [!div class="nextstepaction"]
> [Percursos de Governança Acionáveis](../journeys/overview.md)
