---
title: 'CAF: Melhoria da disciplina de Linha de Base de Segurança'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Melhoria da disciplina de Linha de Base de Segurança
author: BrianBlanchard
ms.openlocfilehash: 28a971f56c9f8ada1d184bdc1cb3dbb9a17c3507
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900345"
---
# <a name="security-baseline-discipline-improvement"></a>Melhoria da disciplina de Linha de Base de Segurança

A disciplina de Linha de Base de Segurança concentra-se em maneiras de estabelecer políticas que protejam a rede, ativos e, mais importante, os dados que residirão na solução de um Provedor de Nuvem. Dentro das cinco disciplinas de Governança de Nuvem, a Linha de Base de Segurança inclui classificação dos dados e propriedade digital. Adicionalmente, inclui documentação de riscos, tolerância comercial e estratégias de mitigação associadas à segurança dos dados, ativos e rede. De uma perspectiva técnica, também inclui o envolvimento nas decisões relativas à [criptografia](../../decision-guides/encryption/overview.md), [requisitos de rede](../../decision-guides/software-defined-network/overview.md), [estratégias de identidade híbrida](../../decision-guides/identity/overview.md) e aos [processos](compliance-processes.md) utilizados para desenvolver políticas de Linha de Base de Segurança de nuvem.

Este artigo descreve algumas tarefas potenciais em que sua empresa pode se envolver para desenvolver e consolidar melhor a disciplina de Linha de Base de Segurança. Essas tarefas podem ser divididas em fases de planejamento, construção, adoção e operação da implementação de uma solução em nuvem, as quais serão iteradas ao permitir o desenvolvimento de uma [abordagem incremental para governança de nuvem](../journeys/overview.md#an-incremental-approach-to-cloud-governance).

![Quatro fases de adoção](../../_images/adoption-phases.png)

*Figura 1. Fases de adoção da abordagem incremental para governança de nuvem.*

É impossível para qualquer pessoa documentar os requisitos a serem levados em consideração de todas as empresas. Portanto, este artigo descreve as atividades de exemplo potenciais e mínimas sugeridas para cada fase do processo de consolidação de governança. O objetivo inicial dessas atividades é ajudá-lo a criar um [MVP de Política](../journeys/overview.md#an-incremental-approach-to-cloud-governance) e estabelecer uma estrutura para a evolução da política incremental. A equipe de Governança de Nuvem precisará decidir quanto investir nessas atividades para melhorar os recursos de controle de Linha de Base de Segurança.

> [!CAUTION]
> Nenhuma das atividades mínimas ou potenciais descritas neste artigo está alinhada a políticas corporativas específicas ou a requisitos de conformidade de terceiros. Essas diretrizes foram desenvolvidas para facilitar as conversas que conduzirão ao alinhamento de ambos os requisitos com um modelo de governança de nuvem.

## <a name="planning-and-readiness"></a>Planejamento e preparação

Essa fase de maturidade de governança preenche a lacuna entre os resultados de negócios e as estratégias acionáveis. Durante esse processo, a equipe de liderança define métricas específicas, mapeia essas métricas para a propriedade digital e começa a planejar o esforço geral de migração.

**Atividades mínimas sugeridas:**

- Avalie as opções da [cadeia de ferramentas de Linha de Base de Segurança](toolchain.md).
- Desenvolva um documento preliminar de Diretrizes de Arquitetura e distribua aos principais stakeholders.
- Treine e envolva as pessoas e as equipes afetadas pelo desenvolvimento de diretrizes de arquitetura.
- Adicione tarefas de segurança prioritárias à lista de pendências de migração.

**Atividades potenciais:**

- Defina um esquema de classificação de dados.
- Conduza um processo de planejamento de propriedade digital para inventariar os ativos de TI atuais, potencializando os processos empresariais e as operações de suporte.
- Realize uma [análise de política](../../governance/policy-compliance/what-is-a-cloud-policy-review.md) para iniciar o processo de modernização das políticas de segurança de TI corporativas existentes e defina políticas de MVP que abordem riscos conhecidos.
- Analise as diretrizes de segurança da sua plataforma de nuvem. Para o Azure, é possível localizá-las na [Plataforma de Confiança do Serviço da Microsoft](https://www.microsoft.com/trustcenter/stp/default.aspx).
- Determine se a política de Linha de Base de Segurança inclui um [Security Development Lifecycle](https://www.microsoft.com/securityengineering/sdl/).
- Avalie os riscos empresariais relacionados a ativos, dados e rede, tendo como base as próximas três liberações, e meça a tolerância da organização para esses riscos.
- Analise o relatório das [principais tendências em segurança cibernética](https://www.microsoft.com/security/operations/security-intelligence-report) da Microsoft para obter uma visão geral do atual cenário de segurança.
- Considere desenvolver uma função [DevOps de Segurança](https://www.microsoft.com/en-us/securityengineering/devsecops) na organização.

<!-- "en-us" location is required for the URL above. -->

## <a name="build-and-pre-deployment"></a>Construção e pré-implantação

Diversos pré-requisitos técnicos e não técnicos são necessários para migrar com êxito um ambiente. Esse processo concentra-se nas decisões, na preparação e infraestrutura principal que procede a uma migração.

**Atividades mínimas sugeridas:**

- Implemente a [cadeia de ferramentas de Linha de Base de Segurança](toolchain.md), lançando em uma fase de pré-implantação.
- Atualize o documento de Diretrizes de Arquitetura e distribua aos principais stakeholders.
- Implemente tarefas de segurança na lista de pendências de migração priorizada.
- Desenvolva materiais educacionais e documentação, comunicações de conscientização, incentivos e outros programas para ajudar a impulsionar a adoção pelos usuários.

**Atividades potenciais:**

- Determine a estratégia de [criptografia](../../decision-guides/encryption/overview.md) da organização para dados hospedados na nuvem.
- Avalie a estratégia de [identidade](../../decision-guides/identity/overview.md) da implantação em nuvem. Determine como a solução de identidade baseada em nuvem coexistirá ou se integrará aos provedores de identidade locais.
- Determine políticas de limite de rede para [SDN (Rede Definida pelo Software)](../../decision-guides/software-defined-network/overview.md) para garantir recursos de rede virtualizados seguros.
- Avalie as políticas de [acesso de privilégios mínimos](/azure/active-directory/users-groups-roles/roles-delegate-by-task) da organização e use funções baseadas em tarefas para fornecer acesso a recursos específicos.
- Aplique mecanismos de segurança e monitoramento a todos os serviços de nuvem e máquinas virtuais.
- Automatize as [políticas de segurança](../../decision-guides/policy-enforcement/overview.md) sempre que possível.
- Analise sua política de Linha de Base de Segurança e determine se é necessário modificar os planos de acordo com as diretrizes de melhores práticas, conforme descrito em [Security Development Lifecycle](https://www.microsoft.com/securityengineering/sdl/).

## <a name="adopt-and-migrate"></a>Adotar e migrar

A migração é um processo incremental que concentra-se no movimento, teste e na adoção de aplicativos ou cargas de trabalho em uma propriedade digital existente.

**Atividades mínimas sugeridas:**

- Migre sua [cadeia de ferramentas de Linha de Base de Segurança](toolchain.md) da pré-implantação para produção.
- Atualize o documento de Diretrizes de Arquitetura e distribua aos principais stakeholders.
- Desenvolver materiais educacionais e documentação, comunicações de conscientização, incentivos e outros programas para ajudar a impulsionar a adoção pelos usuários

**Atividades potenciais:**

- Revise as informações mais recentes sobre a Linha de Base de Segurança e ameaças para identificar novos riscos empresariais.
- Meça a tolerância da organização para lidar com novos riscos de segurança que possam surgir.
- Identifique os desvios da política e imponha correções.
- Ajuste a automação de controle de acesso e segurança para garantir a máxima conformidade com as políticas.  
- Valide se as melhores práticas definidas durante as fases de criação/pré-implantação são executadas corretamente.
- Revise as políticas de acesso de privilégios mínimos e ajuste os controles de acesso para maximizar a segurança.
- Teste a cadeia de ferramentas da Linha de Base de Segurança das cargas de trabalho para identificar e resolver quaisquer vulnerabilidades.

## <a name="operate-and-post-implementation"></a>Operar e pós-implementação

Depois que a transformação for concluída, a governança e as operações deverão ser mantidas para o ciclo de vida natural de um aplicativo ou uma carga de trabalho. Essa fase de maturidade de governança concentra-se nas atividades que geralmente ocorrem depois que a solução é implementada e o ciclo de transformação começa a se estabilizar.

**Atividades mínimas sugeridas:**

- Valide e/ou refine a [cadeia de ferramentas de Linha de Base de Segurança](toolchain.md).
- Personalize notificações e relatórios para alertá-lo sobre potenciais problemas de segurança.
- Refine as Diretrizes de Arquitetura para orientar futuros processos de adoção.
- Periodicamente, comunique e treine as equipes afetadas para garantir a adesão contínua às diretrizes de arquitetura.

**Atividades potenciais:**

- Descubra padrões e comportamento para as cargas de trabalho e configure as ferramentas de monitoramento e relatório para identificar e notificá-lo sobre qualquer atividade, acesso ou uso de recursos anormal.
- Atualize continuamente as políticas de monitoramento e relatório para detectar as vulnerabilidades, explorações e ataques mais recentes.
- Tenha procedimentos implantados para parar rapidamente o acesso não autorizado e desabilitar recursos que possam ter sido comprometidos por um invasor.
- Revise regularmente as melhores práticas de segurança mais recentes e aplique recomendações às políticas de segurança, automação e monitoramento de recursos, sempre que possível.

## <a name="next-steps"></a>Próximas etapas

Agora que você já entende o conceito de governança de segurança de nuvem, prossiga e saiba [quais as diretrizes de segurança e melhores práticas a Microsoft oferece](azure-security-guidance.md) para o Azure.

> [!div class="nextstepaction"]
> [Saiba mais sobre diretrizes de segurança para Azure](azure-security-guidance.md)
> [Introdução à Segurança do Azure](/azure/security/azure-security)
> [Saiba mais sobre registro em log, relatórios e monitoramento](../../decision-guides/log-and-report/overview.md)
