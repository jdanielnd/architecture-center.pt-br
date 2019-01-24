---
title: 'Adoção da Nuvem Empresarial: Introdução'
description: Descreve uma visão geral da primeira fase da transformação digital de uma empresa ao adotar tecnologias de nuvem do Azure
author: petertaylor9999
ms.date: 09/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 027757c76008da092e0d7ab65b072259a04b3cad
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488079"
---
# <a name="enterprise-cloud-adoption-getting-started"></a>Adoção da Nuvem Empresarial: Introdução 

A **transformação digital** para a computação em nuvem representa uma mudança da operação local para a operação na nuvem. Essa mudança inclui novas maneiras de fazer negócios — por exemplo, a transformação digital muda de despesas de capital com software e hardware do data center para despesas operacionais com o uso dos recursos de nuvem. 

## <a name="digital-transformation-process"></a>Transformação digital: processo

Para obter êxito na adoção da nuvem, uma empresa deve preparar a organização, o pessoal e os processos para a transformação digital. Toda estrutura organizacional empresarial é diferente, portanto, não há uma abordagem única para a preparação da organização. Este documento descreve as etapas de alto nível que sua empresa pode seguir para se preparar. Sua organização terá de passar um tempo desenvolvendo um plano detalhado para realizar cada uma das etapas listadas.

O processo de alto nível para a transformação digital inclui:

1. Criar uma equipe de estratégia de nuvem. Essa equipe é responsável por comandar a transformação digital. Nesse estágio, também é importante formar uma equipe de governança e uma equipe de segurança para a transformação digital.
2. Os membros da equipe de estratégia de nuvem aprendem o que há de novo e diferente sobre tecnologias de nuvem.  
3. A equipe de estratégia de nuvem prepara a empresa, criando o caso comercial para a transformação digital — enumera todas as lacunas atuais na estratégia de negócios e determina as soluções de alto nível para eliminá-las.
4. Alinhar as soluções de alto nível com grupos de negócios. Em cada grupo comercial, identifique os stakeholders que serão responsáveis pelo design e implementação para cada solução.
5. Modifique as funções, habilidades e processos existentes para incluir funções, habilidades e processos de nuvem.  
<!--6. Develop processes for operating in the cloud to make solutions more robust in terms of availability, resiliency, and security. 
7. Optimize solutions for performance, scalability, and cost efficiency.-->

## <a name="step-1-create-a-cloud-strategy-team"></a>Etapa 1: criar uma equipe de estratégia de nuvem

A primeira etapa no processo de transformação digital da sua empresa é envolver líderes de negócios de toda a organização para criar uma CST (equipe de estratégia de nuvem). Essa equipe é composta por líderes de negócios do departamento financeiro, da infraestrutura de TI e de grupos de aplicativos. Essas equipes podem ajudar na análise de nuvem e na fase de experimentação.

Por exemplo, uma equipe de estratégia de nuvem pode ser conduzida pelo Diretor de Tecnologia (CTO) e ser composta por membros da equipe de arquitetura empresarial, do departamento financeiro de TI, de tecnólogos seniores de vários grupos de aplicativos de TI (RH, finanças e assim por diante) e por líderes das equipes de infraestrutura, segurança, e rede.  

Também é importante formar duas outras equipes de alto nível: uma equipe de governança e uma equipe de segurança. Essas equipes são responsáveis pelo design, implementação e manutenção de auditorias das políticas de segurança e governança da empresa. A equipe de governança requer membros que tenham trabalhado com proteção de ativos, gerenciamento de custos, política de grupo e tópicos relacionados. A equipe de segurança requer membros que estejam familiarizados com os padrões de segurança atuais do setor, assim como com os requisitos de segurança da empresa.

![Equipe de estratégia de nuvem, com as equipes de segurança e governança](../_images/getting-started-overview-1.png)

A equipe de governança é responsável por projetar e implementar o modelo de governança da empresa na nuvem, bem como implantar e manter os ativos de infraestrutura compartilhada que fazem parte da transformação digital. Esses ativos incluem o hardware, software e recursos da nuvem necessários para conectar a rede local à rede virtual na nuvem.

A equipe de segurança é responsável por projetar e implementar a política de segurança da empresa na nuvem, trabalhando de perto com a equipe de governança. A equipe de segurança é responsável pela extensão do limite de segurança da rede local, que inclui a rede virtual na nuvem. Isso pode significar a responsabilidade e manutenção dos firewalls de entrada e saída da rede virtual da nuvem, além de garantir que as ferramentas e políticas evitem a implantação de recursos não autorizados.

## <a name="step-2-learn-whats-new-in-the-cloud"></a>Etapa 2: saber o que há de novo na nuvem
 
Na próxima etapa do processo de transformação digital da sua empresa, os membros da equipe de estratégia de nuvem deverão aprender como a tecnologia de nuvem mudará a maneira como a empresa faz negócios. Trata-se da preparação e do planejamento para as mudanças na empresa, no pessoal e na tecnologia. Isso é importante para que os membros da equipe de estratégia de nuvem reconheçam o que há de novo e de diferente na nuvem em comparação às infraestruturas locais.

![As equipes de segurança, governança e estratégia de nuvem aprendem as melhores práticas para operar na nuvem.](../_images/getting-started-overview-2.png)

O ponto de partida para entender a nuvem é aprender [como o Azure funciona](what-is-azure.md) em um alto nível. Em seguida, saiba mais sobre os conceitos básicos da [governança no Azure](what-is-governance.md) na preparação para [entender o gerenciamento de acesso de recurso](azure-resource-access.md).

Para o aprendizado avançado, a equipe de governança deve revisar os conceitos e guias de design na seção de governança da tabela de conteúdos. As seções de infraestrutura e cargas de trabalho são úteis para aprender mais sobre arquiteturas típicas e cargas de trabalho na nuvem.

## <a name="step-3-identify-gaps-in-business-strategy"></a>Etapa 3: identificar lacunas na estratégia de negócios

A próxima etapa é a especificação dos problemas de negócios que exigem uma solução de transformação digital, pela equipe de estratégia de nuvem. Por exemplo, uma empresa pode ter um data center local existente com hardware com fim de vida útil e que exige substituição. Em outro exemplo, uma empresa pode estar enfrentando dificuldades com o tempo de lançamento no mercado para novos recursos e serviços e pode estar atrasada em relação à concorrência. Essas lacunas representam as *metas* da transformação digital da sua empresa.

As lacunas na estratégia de negócios podem ser classificadas nas seguintes categorias:

|Categoria|DESCRIÇÃO|
|-----|-----|
|Gerenciamento de custo|Representa uma lacuna na maneira que a empresa paga pela tecnologia.|
|Governança|Representa uma lacuna nos processos usados pela empresa para proteger seus ativos contra o uso indevido que pode resultar em custos excessivos, problemas de segurança ou problemas de conformidade. | 
|Conformidade|Representa uma lacuna na maneira com que a empresa adere aos seus próprios processos internos e políticas, bem como leis externas, regulamentações e padrões. |
|Segurança|Representa uma lacuna na maneira com que a empresa protege seus ativos de tecnologia e os dados contra ameaças externas. |
|Governança de dados|Representa uma lacuna na maneira com que uma empresa gerencia seus dados, especialmente os dados do cliente. Por exemplo, o novo Regulamento Geral sobre a Proteção de Dados (RGPD) na União Europeia tem requisitos rigorosos para a proteção de dados do cliente que podem exigir novos hardware e software.|    

Depois de sua empresa classificar todas as lacunas de estratégia de negócios nessas categorias, a próxima etapa é determinar uma solução de alto nível para cada problema.

A tabela a seguir ilustra diversos exemplos:

|Lacuna de estratégia de negócios|Categoria &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|Solução &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
|-----|-----|-----|
| Os serviços atualmente hospedados no local apresentam problemas com disponibilidade, resiliência e escalabilidade durante o tempo de pico de demanda, que é aproximadamente dez por cento do uso. Os servidores no data center local apresentam fim de vida útil. A TI empresarial recomenda adquirir novo hardware local para o data center com as especificações para lidar com os picos de demanda.| Gerenciamento de custo | Migre as cargas de trabalho locais afetadas para recursos escalonáveis na nuvem, pagando somente pelo uso. |
| As leis e os regulamentos de gerenciamento de dados externos exigem que a empresa adira a um conjunto de controles padrão que requer criptografia de dados em repouso, exigindo novo hardware e software. | Governança de dados | Mova dados para a criptografia do serviço de armazenamento do Azure para dados em repouso. |
| Os serviços hospedados no data center local sofreram ataques de negação de serviço distribuído (DDoS) em serviços públicos. Os ataques são difíceis de atenuar e exigem que novas equipes de hardware, software e segurança os resolvam com eficiência. | Segurança | Migre os serviços para o Azure e tire proveito da proteção contra DDoS do Azure.|

Quando todas as lacunas na estratégia de negócios tiverem sido enumeradas e soluções de alto nível tiverem sido determinadas, priorize a lista. A lista pode ser priorizada alinhando as lacunas de estratégia de negócios com as metas de curto e longo prazo da empresa em cada categoria. Por exemplo, se a empresa tem uma meta de curto prazo para reduzir custos de TI nos próximos dois trimestres fiscais, as lacunas de negócios na categoria de *gerenciamento de custos* podem ser priorizadas pela economia de custos projetada associada a cada uma delas.

A saída desse processo é uma lista com classificação da pilha com soluções de alto nível alinhadas com as categorias de negócios. 

## <a name="step-4-align-high-level-solutions-with-business-groups-to-design-solutions"></a>Etapa 4: alinhar as soluções de alto nível com grupos de negócios para criar soluções 

Agora que as metas da transformação digital foram enumeradas, priorizadas, e soluções de alto nível propostas, a próxima etapa é para a equipe de estratégia de nuvem alinhar cada uma das soluções de alto nível com as equipes de design e implementação em cada um dos grupos de negócios. 

As equipes usam as listas priorizadas e trabalham em cada solução de alto nível para planejar cada solução. O processo de planejamento envolve a especificação de nova infraestrutura e novas cargas de trabalho. Também podem ocorrer alterações nas funções das pessoas e nos processos que elas seguem. Nesse estágio, também é extremamente importante que cada uma das equipes de design inclua as equipes de governança e segurança para revisão de cada projeto. Cada design deve ficar dentro das políticas e dos procedimentos definidos pelas equipes de segurança e governança, e essas equipes devem ser incluídas na aprovação de cada design.

![A equipe de estratégia de nuvem entrega soluções de alto nível para as equipes de design e implementação.](../_images/getting-started-overview-3.png)

O design de cada solução é uma tarefa não trivial e, à medida que os designs são criados, eles devem ser considerados no contexto com outros designs de solução de outras equipes. Por exemplo, se vários dos designs resultarem em uma migração de serviços de aplicativos existentes do local para a nuvem, pode ser mais eficiente agrupá-los e projetar uma estratégia geral de migração. Em outro exemplo, pode não ser possível migrar alguns aplicativos locais e serviços existentes, e a solução pode ser substituí-los por novos serviços de desenvolvimento ou de terceiros. Nesse caso, pode ser mais eficiente agrupar esses juntos e determinar a sobreposição entre eles para determinar se um serviço de terceiros pode ser usado para mais de uma solução.

Depois que o design da solução for concluído, a equipe passa para a fase de implementação de cada design. A fase de implementação de cada design de solução pode ser executada usando processos de gerenciamento do design padrão.

## <a name="step-5-translate-existing-roles-skills-and-process-for-the-cloud"></a>Etapa 5: modificar as funções, habilidades e processos existentes para a nuvem

Em cada fase evolucionária da história no setor de TI, as alterações mais notórias geralmente são marcadas por mudanças nas funções da equipe. Durante a transição dos mainframes para o modelo de cliente/servidor, a função do operador de computador desapareceu em grande parte, substituída pelo administrador do sistema. Quando a era da virtualização chegou, a necessidade de indivíduos que trabalham com servidores físicos diminuiu, substituída por uma necessidade de especialistas em virtualização. Da mesma forma, à medida que as instituições mudarem para computação em nuvem, as funções provavelmente mudarão novamente. Por exemplo, os especialistas de data center podem ser substituídos por analistas financeiros de nuvem. Mesmo nos casos em que os cargos de trabalho de TI não foram alterados, as funções de trabalho diário evoluíram significativamente. 

Os membros da equipe de TI podem se sentir ansiosos sobre suas funções e posições, pois percebem que é necessário um conjunto diferente de habilidades para o suporte de soluções em nuvem. Mas funcionários ágeis que exploram e aprendem novas tecnologias de nuvem não precisam ter esse medo. Eles podem liderar a adoção de serviços de nuvem e ajudar a organização a entender e adotar as mudanças associadas. 

### <a name="capturing-concerns"></a>Capturando preocupações

Durante a transformação digital, cada equipe deve capturar quaisquer preocupações dos funcionários conforme elas surjam. Durante a captura de preocupações, identifique o seguinte: 
* O tipo de preocupação. Por exemplo, os trabalhadores podem ser resistentes às alterações em seus trabalhos que acompanham a transformação digital.
* O impacto da preocupação se não for direcionado. Por exemplo, a resistência à transformação digital pode resultar em trabalhadores executando lentamente as mudanças necessárias. 
* A área equipada para tratar da preocupação. Por exemplo, se os funcionários no departamento de TI estão relutantes em adquirir novas habilidades, a área de stakeholders da TI é a mais bem equipada para lidar com essa questão. Identificar a área pode ficar claro para algumas questões e, nesses casos, talvez seja necessário escalonar para liderança executiva. 

### <a name="identify-gaps"></a>Identificar lacunas

Outro aspecto para lidar com os problemas da transformação digital da sua empresa é identificar **lacunas**. Uma lacuna é uma função, habilidade ou processo necessário para a transformação digital que atualmente não existe na sua empresa. 

Comece enumerando as novas responsabilidades que acompanham a transformação digital, com ênfase em novas responsabilidades e responsabilidades atuais a serem desativadas. Identifique a área que está alinhada com cada responsabilidade. Para novas responsabilidades, determine o quanto está alinhada com a área. Algumas responsabilidades podem abranger várias áreas, e isso representa uma oportunidade para um melhor alinhamento que deve ser capturado como uma preocupação. Caso nenhuma área seja identificada como responsável, use isso como uma lacuna.

Em seguida, identifique as habilidades necessárias para dar suporte à responsabilidade. Determine se sua empresa possui recursos existentes com essas habilidades. Se não houver nenhum recurso existente, determine quais programas de treinamento ou aquisição de talentos são necessários. Determine o período pelo qual a responsabilidade deve ser suportada para manter sua transformação digital no caminho certo.

Por fim, identifique as funções que executarão essas habilidades. Parte da sua força de trabalho atual assumirá partes da função e, em outros casos, poderá ser necessária uma função inteiramente nova. 

### <a name="partner-across-teams"></a>Parceria entre as equipes

As habilidades necessárias para preencher as lacunas na transformação digital de sua organização normalmente não se limitarão a uma única função ou a até mesmo a um único departamento. Habilidades terão relações e dependências que podem abranger uma única ou várias funções, e essas funções podem existir em vários departamentos. Por exemplo, um proprietário de carga de trabalho pode exigir que alguém em uma função de TI provisione recursos principais, como assinaturas e grupos de recursos.

Essas dependências representam novos processos implementados pela organização para gerenciar o fluxo de trabalho entre funções. No exemplo acima, há vários tipos diferentes de processo que podem dar suporte à relação entre o proprietário da carga de trabalho e a função de TI. Por exemplo, uma ferramenta de fluxo de trabalho pode ser criada para gerenciar o processo ou um modelo de email simples pode ser utilizado.

Acompanhe essas dependências e anote os processos que oferecerão suporte a eles e se o processo existe ou não atualmente. Para processos que exigem ferramentas, verifique se a linha do tempo para a implantação de qualquer ferramenta está alinhada ao cronograma de transformação digital geral.

## <a name="next-steps"></a>Próximas etapas

A transformação digital é um processo iterativo e, a cada iteração, as equipes envolvidas ficarão mais eficientes. 

> [!div class="nextstepaction"]
> [Entenda como o Azure funciona](what-is-azure.md)