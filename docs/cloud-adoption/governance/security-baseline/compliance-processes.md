---
title: 'CAF: Processos de conformidade de política de Linha de Base de Segurança'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Processos de conformidade de política de Linha de Base de Segurança
author: BrianBlanchard
ms.openlocfilehash: 8f115436d7021fe725118dcc0dfd64167c4cbfa1
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900367"
---
# <a name="security-baseline-policy-compliance-processes"></a>Processos de conformidade de política de Linha de Base de Segurança

Este artigo descreve uma abordagem para processos de adesão a políticas que regem a [Linha de Base de Segurança](./overview.md). A governança efetiva da segurança na nuvem começa com processos manuais recorrentes, projetados para detectar vulnerabilidades e impor políticas para atenuar esses riscos de segurança. Isso requer o envolvimento regular da equipe de Governança de Nuvem e dos stakeholders de negócios interessados e de TI para revisar e atualizar a política e garantir a conformidade com a política. Além disso, muitos processos de imposição e monitoramento contínuo podem ser automatizados ou complementados com ferramentas para reduzir a sobrecarga de governança e permitir uma resposta mais rápida ao desvio de políticas.

## <a name="planning-review-and-reporting-processes"></a>Planejar, revisar e relatar processos

As melhores ferramentas de linha de base de segurança na nuvem somente são tão boas quanto os processos e políticas que dão suporte a elas. A seguir, é apresentado um conjunto de exemplos de processos normalmente envolvidos na disciplina de Linha de Base de Segurança. Use esses exemplos como ponto de partida ao planejar os processos que permitirão atualizar a política de segurança com base nas alterações dos negócios e no feedback das equipes de TI e segurança encarregadas de transformar as diretrizes de governança em ações.

**Avaliação de risco inicial e planejamento**: Como parte da adoção inicial da disciplina de Linha de Base de Segurança, identifique os principais riscos e tolerâncias de negócios relacionados à segurança de nuvem. Use essas informações para discutir sobre os riscos técnicos específicos com os membros das equipes de TI e segurança e desenvolva um conjunto de linhas de base de políticas de segurança para mitigar esses riscos, objetivando estabelecer a estratégia de governança inicial.

**Planejamento de implantação**: Antes de implantar qualquer carga de trabalho ou um ativo, execute uma revisão de segurança para identificar quaisquer novos riscos e garantir que todos os requisitos de política de segurança de acesso e os dados sejam atendidos.

**Teste de implantação**: Como parte do processo de implantação para qualquer carga de trabalho ou ativo, a equipe de Governança de nuvem, em cooperação com a suas equipes de segurança corporativa, será responsável por analisar a implantação para validar a conformidade com a política de segurança.

**Planejamento anual**: Anualmente, realize uma análise de alto nível da estratégia de Linha de base de segurança. Explore as prioridades corporativas futuras e estratégias de nuvem atualizadas para identificar aumento de risco potencial e outras necessidades de segurança emergentes. Além disso, use esse tempo para analisar as melhores práticas de Linha de base de segurança e integre às suas políticas e processos de avaliação.

**Planejamento e revisão trimestral**: Trimestralmente, execute uma análise de dados de auditoria de segurança e relatórios de incidentes para identificar quaisquer alterações na política de segurança. Como parte desse processo, examine o panorama atual de segurança cibernética para prever proativamente as ameaças emergentes e atualizar a política conforme apropriado. Depois que a análise for concluída, alinhe as diretrizes de design com a política atualizada.

Esse processo de planejamento também é um momento propício para avaliar a associação atual da equipe de Governança de Nuvem, considerando as lacunas de conhecimento relacionadas a políticas novas ou em desenvolvimento e aos riscos relacionados à segurança. Convide a equipe de TI relevante a participar das análises e planejamentos como consultores temporários ou membros permanentes da sua equipe. 

**Educação e treinamento**: Bimestralmente, proporcione sessões de treinamento para certificar-se de que a equipe de TI e os desenvolvedores estejam atualizados a respeito dos requisitos de política de segurança mais recentes. Como parte desse processo, revise e atualize todas as documentações, diretrizes ou outros recursos de treinamento para garantir que estejam em sincronia com as últimas declarações da política corporativa.

**Análises de auditoria e relatórios mensais**: Mensalmente, realize uma auditoria em todas as implantações na nuvem para garantir o alinhamento contínuo com a política de segurança. Revise as atividades relacionadas à segurança com a equipe de TI e identifique quaisquer problemas de conformidade que ainda não são tratados como parte do processo de monitoramento e imposição em andamento. O resultado dessa revisão é um relatório para a equipe de Estratégia de Nuvem e cada equipe de adoção de nuvem para se comunicar com a adesão geral à política. O relatório também é armazenado para fins de auditoria e legais.

## <a name="ongoing-monitoring-processes"></a>Processos de monitoramento contínuo

Determinar se a estratégia de governança de segurança é bem-sucedida depende da visibilidade no estado atual e futuro da sua infraestrutura de nuvem. Sem a capacidade de analisar as métricas relevantes e os dados da integridade e atividade dos recursos de nuvem, você não pode identificar as mudanças nos seus riscos ou detectar violações das suas tolerâncias a riscos. Os processo de administração contínua discutidos acima exigem dados de qualidade para garantir que a política possa ser modificada para proteger melhor a sua infraestrutura contra ameaças e requisitos de segurança a alteração.

Certifique-se de que as equipes de TI e segurança implementaram sistemas de monitoramento automatizados para a infraestrutura de nuvem que captura os dados de log relevantes que você precisa para avaliar os riscos. Seja proativo no monitoramento desses sistemas para garantir a detecção e mitigação imediatas de possíveis violações da política e garanta que a sua estratégia de monitoramento esteja em linha com as suas necessidades de segurança.

## <a name="violation-triggers-and-enforcement-actions"></a>Gatilhos de violação e ações de imposição

Uma vez que a não conformidade de segurança pode levar a riscos de exposição de dados e interrupção de serviço, a equipe de Governança de nuvem deve ter visibilidade de violações graves de política. Certifique-que de que a equipe de TI tenha caminhos claros para relatar esses problemas de segurança para os membros da equipe de governança mais adequados para identificar e verificar se os problemas de políticas são atenuados.  

Quando violações forem detectadas, será necessário realizar ações para realinhar a política o mais rápido possível. A equipe de TI pode automatizar a maioria dos gatilhos de violação usando as ferramentas descritas na [cadeia de ferramentas da Linha de Base de Segurança](toolchain.md).

Os gatilhos e ações de imposição a seguir fornecem exemplos que você pode consultar ao planejar como usar os dados de monitoramento para resolver violações da política:

- Aumento em ataques detectados: Se qualquer recurso passa por um aumento de 25% na força bruta ou ataques de DDoS, discuta com a equipe de segurança de TI e com o proprietário da carga de trabalho para determinar soluções. Acompanhe o problema e diretrizes de atualização se a revisão de política for necessária para prevenir incidentes futuros.
- Dados não classificados detectados: Qualquer fonte de dados sem uma privacidade adequada, segurança ou impacto de negócios de classificação terão acesso externo negado até que a classificação seja aplicada, o proprietário dos dados e o nível apropriado de proteção de dados aplicado.
- Problema de integridade de segurança detectado: Desabilite o acesso a máquinas virtuais (VMs) que tem acesso conhecido ou vulnerabilidades de malware identificados até que os patches apropriados ou software de segurança possam ser instalados. Diretrizes de política de atualização para levar em consideração para qualquer ameaças detectadas recentemente.
- Vulnerabilidade de rede detectada: O acesso a qualquer recurso não foi explicitamente permitido pelas políticas de acesso de rede deve disparar um alerta à equipe de segurança de TI e o proprietário da carga de trabalho relevante. Acompanhe o problema e diretrizes de atualização se a revisão de política for necessária para atenuar incidentes futuros.

## <a name="next-steps"></a>Próximas etapas

Usando o [modelo de Gerenciamento de Nuvem](./template.md), documente os processos e gatilhos que alinham-se ao plano de adoção da nuvem atual.

Para obter diretrizes sobre a execução de políticas de gerenciamento de nuvem em alinhamento com planos de adoção, consulte o artigo sobre melhoria de disciplina.

> [!div class="nextstepaction"]
> [Melhoria da disciplina de Linha de Base de Segurança](./discipline-improvement.md)
