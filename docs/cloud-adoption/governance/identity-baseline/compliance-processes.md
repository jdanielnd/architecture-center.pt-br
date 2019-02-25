---
title: 'CAF: Processos de conformidade de política de Linha de Base de Identidade'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Processos de conformidade de política de Linha de Base de Identidade
author: BrianBlanchard
ms.openlocfilehash: e00427662c811fccb7e0a7261a4bb3dd3437103e
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900190"
---
# <a name="identity-baseline-policy-compliance-processes"></a>Processos de conformidade de política de Linha de Base de Identidade

Este artigo descreve uma abordagem para processos de adesão a políticas que regem a [Linha de Base de Identidade](./overview.md). A governança eficaz da identidade começa com processos manuais recorrentes que orientam a adoção e revisões de políticas de identidade. Isso requer o envolvimento regular da equipe de Governança de Nuvem, TI e stakeholders para analisar e atualizar a política, visando garantir conformidade com a política. Além disso, muitos processos de imposição e monitoramento contínuo podem ser automatizados ou complementados com ferramentas para reduzir a sobrecarga de governança e permitir uma resposta mais rápida ao desvio de políticas.

## <a name="planning-review-and-reporting-processes"></a>Planejar, revisar e relatar processos

As ferramentas de gerenciamento de identidades oferecem recursos e funcionalidades que auxiliam consideravelmente o gerenciamento de usuários e controle de acesso em uma implantação na nuvem. No entanto, também exigem processos e políticas bem planejadas para dar suporte às metas da organização. A seguir, é apresentado um conjunto de exemplos de processos normalmente envolvidos na disciplina de Linha de Base de Identidade. Use esses exemplos como ponto de partida ao planejar os processos que permitirão atualizar a política de identidade com base nas alterações nos negócios e no feedback das equipes de TI encarregadas de transformar as diretrizes de governança em ações.

**Avaliação de risco inicial e planejamento**: Como parte da adoção inicial da disciplina de Linha de Base de Identidade, identifique os principais riscos e tolerâncias de negócios relacionados ao gerenciamento de identidades de nuvem. Use essas informações para discutir sobre os riscos técnicos específicos com os membros das equipes de TI responsáveis pelo gerenciamento de serviços de identidade e desenvolver um conjunto de linhas de base de políticas de segurança para mitigar esses riscos, objetivando estabelecer a estratégia de governança inicial.

**Planejamento de implantação**: Antes de qualquer implantação, revise as necessidades de acesso para qualquer carga de trabalho e desenvolva uma estratégia de controle de acesso que alinhe-se à política de identidade corporativa estabelecida. Documente quaisquer lacunas entre as necessidades e a política atual para determinar se as atualizações de políticas são necessárias e modifique a política, conforme necessário.

**Teste de implantação**: Como parte da implantação, a equipe de Governança de Nuvem, em cooperação com as equipes de TI responsáveis pelos serviços de identidade, será responsável por analisar a implantação para validar a conformidade com a política de identidade.

**Planejamento anual**: Anualmente, realize uma análise de alto nível da estratégia de gerenciamento de identidades. Explore as alterações planejadas no ambiente de serviços de identidade e as estratégias de adoção de nuvem atualizadas para identificar o aumento de risco potencial ou modificar os padrões de infraestrutura de identidade atuais. Além disso, use esse tempo para revisar as melhores práticas de gerenciamento de identidades mais recentes e integrá-las às políticas e aos processos de avaliação.

**Planejamento trimestral**: Realize trimestralmente uma análise geral dos dados de auditoria de controle de acesso e identidade e reúna-se com as equipes de adoção de nuvem para identificar novos possíveis riscos ou os requisitos operacionais que requerem atualizações na política de identidade ou mudanças na estratégia de controle de acesso.

Esse processo de planejamento também é um momento propício para avaliar a associação atual da equipe de Governança de Nuvem, considerando as lacunas de conhecimento relacionadas a políticas novas ou em desenvolvimento e aos riscos relacionados à identidade. Convide a equipe de TI relevante a participar de análises e planejamento como consultores técnicos temporários ou membros permanentes de sua equipe.  

**Educação e treinamento**: Bimestralmente, proporcione sessões de treinamento para certificar-se de que a equipe de TI e os desenvolvedores estejam atualizados sobre os requisitos política de identidade mais recentes. Como parte desse processo, revise e atualize todas as documentações, diretrizes ou outros recursos de treinamento para garantir que estejam em sincronia com as últimas declarações da política corporativa.

**Análises de auditoria e relatórios mensais**: Mensalmente, realize uma auditoria em todas as implantações na nuvem para garantir o alinhamento contínuo com a política de identidade. Use essa análise para verificar o acesso do usuário às alterações nos negócios para garantir que os usuários tenham acesso correto aos recursos da nuvem e que as estratégias de acesso, como o RBAC, estão sendo respeitadas sistematicamente. Identifique todas as contas com privilégios e documente a finalidade. Esse processo de análise gera um relatório para a equipe de Estratégia de Nuvem e para cada equipe de adoção de nuvem, detalhando a adesão global à política. Adicionalmente, o relatório é armazenado para fins de auditoria e efeitos legais.

## <a name="ongoing-monitoring-processes"></a>Processos de monitoramento contínuo

Determinar se a estratégia de governança de identidade é bem-sucedida depende da visibilidade do estado atual e anterior dos sistemas de identidade. Sem a capacidade de analisar as métricas relevantes da implantação na nuvem e os dados relacionados, não é possível identificar as alterações nos riscos nem detectar violações das tolerâncias de risco. Os processos de governança contínuos abordados acima exigem dados de qualidade para garantir que a política possa ser modificada para dar suporte às necessidades variáveis de seus negócios.

Certifique-se de que as equipes de TI implementaram sistemas de monitoramento automatizados para os serviços de identidade que capturam os logs e as informações de auditoria necessárias para avaliar o risco. Seja proativo no monitoramento desses sistemas para garantir a detecção e mitigação imediatas de possíveis violações da política e garanta que todas as alterações na infraestrutura de identidade sejam refletidas na estratégia de monitoramento.

## <a name="violation-triggers-and-enforcement-actions"></a>Gatilhos de violação e ações de imposição

As violações da política de identidade podem resultar em acesso não autorizado a dados confidenciais e resultar em graves interrupções de aplicativos e serviços críticos. Quando violações forem detectadas, será necessário realizar ações para realinhar a política o mais rápido possível. A equipe de TI pode automatizar a maioria dos gatilhos de violação, usando as ferramentas descritas na [cadeia de ferramentas da Linha de Base de Identidade](toolchain.md).

Os gatilhos e ações de imposição a seguir fornecem exemplos que você pode consultar ao planejar como usar os dados de monitoramento para resolver violações da política:

- Atividade suspeita detectada: Logons de usuários detectados a partir de endereços IP de proxy anônimos, localizações desconhecidas ou logons sucessivos de localizações geográficas impossivelmente distantes podem indicar uma potencial violação de conta ou tentativa de acesso mal-intencionado. O logon será bloqueado até que a identidade do usuário seja verificada e realizada a redefinição de senha.
- Credenciais do usuário vazadas: As contas que tiverem o nome de usuário e senha vazados para a Internet ficarão desabilitadas até que a identidade do usuário seja verificada e realizada a redefinição de senha.
- Controles de acesso insuficientes detectados: Todos os ativos protegidos cujas restrições de acesso não atendam aos requisitos de segurança terão o acesso bloqueado até que o recurso esteja em conformidade.

## <a name="next-steps"></a>Próximas etapas

Usando o [modelo de Gerenciamento de Nuvem](./template.md), documente os processos e gatilhos que alinham-se ao plano de adoção da nuvem atual.

Para obter diretrizes sobre a execução de políticas de gerenciamento de nuvem em alinhamento com planos de adoção, consulte o artigo sobre melhoria de disciplina.

> [!div class="nextstepaction"]
> [Melhoria da disciplina de Linha de Base de Identidade](./discipline-improvement.md)
