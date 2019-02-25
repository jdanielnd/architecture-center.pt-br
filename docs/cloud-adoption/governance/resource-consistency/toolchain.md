---
title: 'CAF: Ferramentas de consistência de recursos no Azure'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Ferramentas de consistência de recursos no Azure
author: BrianBlanchard
ms.openlocfilehash: 68503289f60fbb3682264ff39546ca7b7700cef5
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900351"
---
# <a name="resource-consistency-tools-in-azure"></a>Ferramentas de consistência de recursos no Azure

A [Consistência de recurso](overview.md) é uma das [Cinco disciplinas de Governança de Nuvem](../governance-disciplines.md). Essa disciplina se concentra em formas de estabelecer políticas relacionadas à gestão operacional de um ambiente, aplicativo ou carga de trabalho. Nas cinco disciplinas de governança de nuvem, a Consistência de recursos inclui o monitoramento de aplicativos, a carga de trabalho e o desempenho de ativos. Ela também inclui as tarefas necessárias para atender às demandas de dimensionamento, corrigir violações de desempenho de SLA e evitar proativamente as violações de SLA por meio da correção automatizada.

A seguir, é apresentada uma lista de ferramentas nativas do Azure que podem ajudar a amadurecer as políticas e os processos que dão suporte a essa disciplina de governança.

|    | [Portal do Azure](https://azure.microsoft.com/features/azure-portal/)  | [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview)  | [Azure Blueprints](/azure/governance/blueprints/overview) | [Automação do Azure](/azure/automation/automation-intro) | [Azure AD](/azure/active-directory/fundamentals/active-directory-whatis) |
|---------|---------|---------|---------|---------|---------|
| Implantação de recursos                             | Sim | sim | sim | sim | Não   |
| Gerenciar recursos                             | Sim | sim | sim | sim | Não   |
| Implantar recursos usando modelos             | Não   | Sim | Não   | Sim | Não   |
| Implantação do ambiente orquestrada          | Não   | Não   | Sim | Não  | Não   |
| Definir grupos de recurso                       | Sim | sim | sim | Não  | Não   |
| Gerenciar os proprietários da carga de trabalho e conta           | Sim | sim | sim | Não  | Não   |
| Gerenciar acesso condicional aos recursos       | Sim | sim | sim | Não  | Não   |
| Configurar usuários do RBAC                         | Sim | Não  | Não  | Não   | Sim |
| Atribuir funções e permissões para recursos | Sim | sim | sim | Não   | Sim |
| Definir dependência entre recursos        | Não   | sim | sim | Não  | Não   |
| Aplicar controle de acesso                         | Sim | sim | sim | Não   | Sim |
| Avaliar disponibilidade e escalabilidade          | Não   | Não  | Não   | Sim | Não   |
| Aplicar marcas aos recursos                      | Sim | sim | sim | Não  | Não   |
| Atribuir regras do Azure Policy                    | Sim | sim | sim | Não  | Não   |
| Preparar recursos para recuperação de desastre         | Sim | sim | sim | Não  | Não   |
| Aplicar a correção automatizada                  | Não   | Não  | Não   | Sim | Não   |
| Gerenciar a cobrança                               | Sim | Não  | Não  | Não  | Não   |

Junto com essas ferramentas de consistência de recursos e recursos, você precisará monitorar seus recursos implantados para problemas de desempenho e integridade. O [Azure Monitor](/azure/azure-monitor/overview) é o monitoramento padrão e solução de relatórios no Azure. O Azure Monitor fornece inúmeros recursos individuais que você pode usar para monitorar seus recursos de nuvem e a seguinte lista mostra qual recurso permite que você aborde requisitos de monitoramento comuns.

|                                                    | [Portal do Azure](https://azure.microsoft.com/features/azure-portal/) | [Application Insights](/azure/application-insights/app-insights-overview) | [Log Analytics](/azure/azure-monitor/log-query/log-query-overview) | [API REST do Azure Monitor](/rest/api/monitor/) |
|----------------------------------------------------|--------------|----------------------|---------------|------------------------|
| Dados telemétricos de máquina virtual do log                 | Não            | Não                    | Sim           | Não                      |
| Dados telemétricos de rede virtual do log              | Não            | Não                    | Sim           | Não                      |
| Dados telemétricos de serviços PaaS de Log                   | Não            | Não                    | Sim           | Não                      |
| Dados telemétricos de aplicativo de log                     | Não            | Sim                  | Não            | Não                      |
| Configurar alertas e relatórios                       | Sim          | Não                   | Não             | Sim                    |
| Agendar relatórios regulares ou análise personalizada        | Não            | Não                   | Não            | Não                      |
| Visualizar e analisar dados de desempenho e log     | Sim          | Não                   | Não            | Não                      |
| Integrar a solução de monitoramento de terceiros ou locais     | Não            | Não                   | Não             | Sim                    |

Ao planejar a implantação, será necessário considerar onde os dados de registro em log serão armazenados e como os [serviços de relatório e monitoramento baseados em nuvem serão integrados](../../decision-guides/log-and-report/overview.md) com os processos e as ferramentas existentes.

> [!NOTE]
> As organizações também usam ferramentas de DevOps de terceiros para monitorar recursos e cargas de trabalho. Para obter mais informações, consulte [integrações de ferramentas do DevOps](https://azure.microsoft.com/products/devops-tool-integrations/).

# <a name="next-steps"></a>Próximas etapas

Aprenda como criar, atribuir e gerenciar [definições de políticas](/azure/governance/policy/) no Azure.
