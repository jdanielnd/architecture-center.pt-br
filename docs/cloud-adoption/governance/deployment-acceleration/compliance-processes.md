---
title: 'CAF: Processos de conformidade de política de aceleração de implantação'
description: Processos de conformidade de política de aceleração de implantação
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
author: alexbuckgit
ms.openlocfilehash: 08046a4ae199a8714393d2aba3841dd84008d836
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900250"
---
# <a name="deployment-acceleration-policy-compliance-processes"></a>Processos de conformidade de política de aceleração de implantação

Este artigo descreve uma abordagem para processos de adesão a políticas que regem a [Aceleração de implantação](./overview.md). A governança efetiva de configuração na nuvem começa com os processos manuais recorrentes designados para detectar problemas e impor políticas para mitigar esses riscos. No entanto, você pode automatizar esses processos e complementar com ferramentas para reduzir a sobrecarga de governança e permitir respostas mais rápidas para desvio.

## <a name="planning-review-and-reporting-processes"></a>Planejar, revisar e relatar processos

As melhores ferramentas de Aceleração de implantação na nuvem somente são tão boas quanto os processos e políticas que dão suporte a elas. A seguir, é apresentado um conjunto de exemplos de processos normalmente usados como parte disciplina da Linha de Base de Segurança. Use esses exemplos como ponto de partida ao planejar os processos que permitirão atualizar a política de segurança com base nas alterações dos negócios e no feedback das equipes de TI e segurança responsáveis por transformar as diretrizes de governança em ações.

**Avaliação de risco inicial e planejamento**: Como parte da sua adoção inicial da disciplina de Aceleração de Implantação, identifique seus riscos comerciais principais e as tolerâncias relacionadas às operações e ao gerenciamento de TI. Use essas informações para discutir sobre os riscos técnicos específicos com os membros das equipes de operações e de TI e equipe de segurança, e desenvolva um conjunto de implantação e políticas de configuração para mitigar esses riscos para estabelecer sua estratégia de governança inicial.

**Planejamento de implantação**: Antes de implantar qualquer ativo, realize uma análise de segurança para identificar qualquer risco e garantir que todos os acessos e requisitos de política de segurança de dados sejam atendidos.

**Teste de implantação**: Como parte do processo de implantação para qualquer ativo, a equipe de Governança de nuvem, em cooperação com a suas equipes de segurança corporativa, será responsável por analisar a implantação para validar a conformidade com a política de segurança.

**Planejamento anual**: Conduza uma análise de alto nível anual da estratégia de Implantação de aceleração. Explore as prioridades corporativas futuras e estratégias de adoção de nuvem atualizadas para identificar o aumento de risco potencial e outras necessidades de configuração emergentes e oportunidades. Além disso, use esse tempo para analisar as melhores práticas de Aceleração de implantação e integre em suas políticas e processos de avaliação.

**Planejamento e revisão trimestral**: Trimestralmente, execute uma análise de dados de auditoria de segurança e relatórios de incidentes para identificar quaisquer alterações na política de Aceleração de implantação. Como parte desse processo, examine o panorama atual de segurança cibernética para prever proativamente as ameaças emergentes e atualizar a política conforme apropriado. Depois que a análise for concluída, alinhe as diretrizes de design e aplicativo à política atualizada.

Esse processo de planejamento também é um momento propício para avaliar a associação atual da equipe de Governança de Nuvem, considerando as lacunas de conhecimento relacionadas a políticas novas ou em desenvolvimento e aos riscos relacionados a DevOps e Aceleração de Implantação. Convide a equipe de TI relevante a participar das análises e planejamentos como consultores temporários ou membros permanentes da sua equipe. 

**Educação e treinamento**: Bimestralmente, proporcione sessões de treinamento para certificar-se de que a equipe de TI e os desenvolvedores estejam atualizados a respeito dos últimos requisitos e estratégia de Aceleração de Implantação. Como parte desse processo, revise e atualize todas as documentações, diretrizes ou outros recursos de treinamento para garantir que estejam em sincronia com as últimas declarações da política corporativa.

**Análises de auditoria e relatórios mensais**: Mensalmente, realize uma auditoria em todas as implantações na nuvem para garantir o alinhamento contínuo com a política de configuração. Revise as atividades relacionadas à segurança com a equipe de TI e identifique quaisquer problemas de conformidade que ainda não são tratados como parte do processo de monitoramento e imposição em andamento. O resultado dessa revisão é um relatório para a equipe de Estratégia de Nuvem e cada equipe de adoção de nuvem para se comunicar com a adesão geral à política. O relatório também é armazenado para fins de auditoria e legais.

## <a name="ongoing-monitoring-processes"></a>Processos de monitoramento contínuo

Determinar se a estratégia de governança de Aceleração de implantação é bem-sucedida depende da visibilidade no estado atual e futuro da sua infraestrutura de nuvem. Sem a capacidade de analisar as métricas relevantes e os dados da integridade e atividade dos recursos de nuvem, você não pode identificar as mudanças nos seus riscos ou detectar violações das suas tolerâncias a riscos. Os processo de administração contínua discutido acima exigem dados de qualidade para garantir que a política possa ser modificada para proteger a sua infraestrutura contra ameaças e riscos de recursos configurados incorretamente.

Certifique-se de que as equipes de TI e segurança implementaram sistemas de monitoramento automatizados para a infraestrutura de nuvem que captura os dados de log relevantes que você precisa para avaliar os riscos. Seja proativo no monitoramento desses sistemas para garantir a detecção e mitigação imediatas de possíveis violações da política e garanta que a sua estratégia de monitoramento esteja em linha com as necessidades de implantação e configuração.

## <a name="violation-triggers-and-enforcement-actions"></a>Gatilhos de violação e ações de imposição

Uma vez que a não conformidade com as políticas de segurança possa levar a riscos de interrupção de serviço graves, a equipe de Governança de Nuvem deverá ter visibilidade em sérias violações da política. Certifique-que de que a equipe de TI tenha caminhos de escalação clara para relatar esses problemas de conformidade de configuração para os membros da equipe de governança mais adequados para identificar e verificar se os problemas de políticas são atenuados.  

Quando violações forem detectadas, será necessário realizar ações para realinhar a política o mais rápido possível. A equipe de TI pode automatizar a maioria dos gatilhos de violação usando as ferramentas descritas na [cadeia de ferramentas de Aceleração de implantação](toolchain.md).

Os gatilhos e ações de imposição a seguir fornecem exemplos que você pode consultar ao discutir como usar os dados de monitoramento para resolver violações da política:

- **Alterações inesperadas na configuração detectada**. Se a configuração de um recurso for alterada inesperadamente, trabalhe com os proprietários de carga de trabalho e da equipe IT para identificar a causa raiz e desenvolver um plano de correção.
- **A configuração de novos recursos não adere à política.** Trabalhe com as equipes do DevOps e proprietários da carga de trabalho para examinar as políticas de aceleração de implantação durante a inicialização do projeto para que todos os envolvidos compreendam os requisitos de política relevantes.
- **Falhas na implantação ou problemas de configuração causam atrasos em cronogramas do projeto.** Trabalhe com as equipes de desenvolvimento e os proprietários da carga de trabalho para garantir que a equipe compreenda como automatizar a implantação de recursos baseados em nuvem para consistência e capacidade de repetição. Implantações totalmente automatizadas devem ser necessárias no início do ciclo de desenvolvimento &mdash; tentar realizar depois no ciclo de desenvolvimento geralmente leva a problemas inesperados e atrasos.

## <a name="next-steps"></a>Próximas etapas

Usando o [modelo de Gerenciamento de Nuvem](./template.md), documente os processos e gatilhos que alinham-se ao plano de adoção da nuvem atual.

Para obter diretrizes sobre a execução de políticas de gerenciamento de nuvem em alinhamento com planos de adoção, consulte o artigo sobre melhoria de disciplina.

> [!div class="nextstepaction"]
> [Aperfeiçoamento da disciplina de Aceleração de implantação](./discipline-improvement.md)
