---
title: 'CAF: Métricas de Aceleração de Implantação, indicadores e tolerância a riscos'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Métricas de Aceleração de Implantação, indicadores e tolerância a riscos
author: alexbuckgit
ms.openlocfilehash: 87d6f9f67b98d5761aced560392c9d38fa682ee7
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241207"
---
# <a name="deployment-acceleration-metrics-indicators-and-risk-tolerance"></a>Métricas de Aceleração de Implantação, indicadores e tolerância a riscos

Este artigo tem como objetivo ajudar a quantificar a tolerância a riscos do negócios no que diz respeito à Aceleração de Implantação. Definir métricas e indicadores ajuda a criar um caso de negócios para fazer um investimento no amadurecimento da disciplina de Aceleração de Implantação.

## <a name="metrics"></a>Métricas

A Aceleração de Implantação se concentra na implantação, atualização e manutenção de recursos de nuvem configurados para a operação de sistemas adequados. As informações a seguir são úteis ao adotar essa disciplina de governança de nuvem:

- **Objetivo de Tempo de Recuperação (RTO)**. Tempo máximo aceitável que um aplicativo pode ficar indisponível após um incidente.
- **Objetivo de Ponto de Recuperação (RPO)**. Duração máxima de perda de dados que é aceitável durante um desastre. Por exemplo, se você armazena dados em um único banco de dados, com nenhuma replicação para outros bancos de dados, e executar backups a cada hora, poderá perder até uma hora de dados.
- **Tempo Médio de Recuperação (MTTR)**. Tempo médio necessário para restaurar um componente após uma falha.
- **Tempo médio entre falhas (MTBF)**. A duração de um componente razoável pode esperar para executar entre interrupções. Essa métrica pode ajudar você a calcular a frequência com que um serviço ficará indisponível.
- **Contratos de nível de serviço (SLAs)**. Isso pode incluir compromissos da Microsoft para atualização e conectividade dos serviços do Azure, bem como compromissos realizados pela empresa para seus clientes internos e externos.
- **Tempo de implantação**. A quantidade de tempo necessária para implantar atualizações em um sistema existente.
- **Ativos fora de conformidade**. O número ou porcentagem de recursos que estão fora de conformidade com as políticas definidas.

## <a name="risk-tolerance-indicators"></a>Indicadores de tolerância de risco

Os riscos relacionados à aceleração de implantação em grande parte são relacionados ao número e a complexidade dos sistemas baseados em nuvem implantados para sua empresa. À medida em que a sua propriedade de nuvem cresce, o número de sistemas implantados e a frequência de atualização dos seus recursos aumentarão. As dependências entre recursos ampliarão a importância de garantir a configuração adequada de recursos e a criação de sistemas para resiliência se um ou mais recurso apresentar tempo de inatividade inesperado.

<!-- "en-us" location is required for the URL below. -->

Considere adotar um DEvOps ou cultura organizacional [DevSecOps](https://www.microsoft.com/en-us/securityengineering/devsecops) no início de sua jornada de adoção da nuvem. Organizações de TI corporativas tradicionais geralmente têm isolados operações, segurança e equipes de desenvolvimento que muitas vezes não colaboram bem ou são até mesmo antagônicos ou hostis umas com as outras. Reconhecer esses desafios no início e integrar os principais stakeholders de cada uma das equipes pode ajudar a garantir a agilidade na sua adoção da nuvem permanecendo seguro e bem governado.

Trabalhe com a sua equipe DevSecOps e com os stakeholders da empresa para identificar os [riscos de negócios](business-risks.md) relacionados à configuração, em seguida, determine uma linha de base aceitável para tolerância a riscos de configuração. Esta seção de diretrizes de CAF fornece exemplos, mas os riscos e linhas de base detalhados da sua empresa ou das suas implantações podem ser diferentes.

Assim que você tiver uma linha de base, estabeleça parâmetros de comparação mínimos que representem um aumento inaceitável em seus riscos identificados. Esses parâmetros de comparação atuam como gatilhos quando você precisar tomar medidas para atenuar esses riscos. A seguir, há alguns exemplos de como métricas relacionadas à configuração, como as discutidas anteriormente, podem justificar o aumento de investimento na disciplina de Aceleração de implantação.

- **Gatilho do contrato de nível de serviço (SLA)**. Uma empresa que não pode atender seus SLAs para seus clientes externos ou parceiros internos, deve investir na disciplina de Aceleração de implantação para reduzir o tempo de inatividade do sistema.
- **Gatilhos de tempo de recuperação**. Se uma empresa exceder os limites necessários para o tempo de recuperação após uma falha do sistema, ela investir em aperfeiçoar sua disciplina de aceleração de implantação e no design do sistema para reduzir ou eliminar falhas ou o efeito do tempo de inatividade do componente individual.
- **Gatilhos de descompasso de configuração**. Uma empresa que está apresentando mudanças inesperadas na configuração dos componentes do sistema principal ou falhas na implantação ou atualizações de seus sistemas, deverá investir na disciplina de Aceleração de implantação para identificar a causa raiz e as etapas de correção.  
- **Gatilhos fora de conformidade**. Se o número de recursos fora de conformidade exceder um limite definido (seja como um número total de recursos ou uma porcentagem do total de recursos), uma empresa deve investir em melhorias na disciplina de Aceleração de implantação para garantir que cada configuração de recurso permaneça em conformidade em todo o ciclo de vida desse recurso.
- **Gatilhos de agendamento de projeto**. Se o tempo de implantação de aplicativos e recursos da empresa geralmente exceder um limite de definição, a empresa deve investir em seus processos de implantação de aceleração para introduzir ou aprimorar implantações automatizadas para consistência e previsibilidade. Tempos de implantação, medidos em dias ou até mesmo semanas geralmente indicam uma estratégia de implantação de aceleração abaixo do ideal.

## <a name="next-steps"></a>Próximas etapas

Usar o [modelo de Gerenciamento de Nuvem](./template.md), de métricas do documento e de indicadores de tolerância que se alinham ao atual plano de adoção da nuvem.

Compilar riscos e tolerância, estabelecer um processo para administrar e comunicar a aderência da política de aceleração de implantação.

> [!div class="nextstepaction"]
> [Estabelecer os processos de conformidade com as políticas](compliance-processes.md)
