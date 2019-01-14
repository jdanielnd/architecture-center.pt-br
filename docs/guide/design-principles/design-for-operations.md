---
title: Design para operações
titleSuffix: Azure Application Architecture Guide
description: Crie um aplicativo para que a equipe de operações tenha as ferramentas necessárias.
author: MikeWasson
ms.date: 08/30/2018
ms.custom: seojan19
ms.openlocfilehash: c14f3d14330633e7a8c88eedd38f1844291195b1
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54110400"
---
# <a name="design-for-operations"></a>Design para operações

## <a name="design-an-application-so-that-the-operations-team-has-the-tools-they-need"></a>Projetar um aplicativo para que a equipe de operações tenha as ferramentas necessárias

A nuvem mudou significativamente a função da equipe de operações. Eles não são mais responsáveis por gerenciar o hardware e a infraestrutura que hospeda o aplicativo.  Dito isso, as operações ainda são uma parte crítica da execução bem-sucedida de um aplicativo em nuvem. Algumas das funções importantes da equipe de operações incluem:

- Implantação
- Monitoramento
- Escalonamento
- Resposta a incidentes
- Auditoria de segurança

O rastreamento e o registro robustos são particularmente importantes em aplicativos de nuvem. Envolva a equipe de operações no design e no planejamento, para garantir que o aplicativo ofereça os dados e as informações que precisam ser bem-sucedidas.  <!-- to do: Link to DevOps checklist -->

## <a name="recommendations"></a>Recomendações

**Torne todas as coisas observáveis**. Depois que uma solução é implantado e está em execução, os logs e os rastreamentos são suas informações primárias do sistema. *Rastreamento* registra um caminho através do sistema e é útil para identificar afunilamentos, problemas de desempenho e pontos de falha. O *registro em log* captura eventos individuais, como alterações de estado, erros e exceções do aplicativo. Faça logon em produção, caso contrário, você perderá informações nos momentos em que mais precisará delas.

**Instrumente para monitoramento**. O monitoramento oferece informações sobre o desempenho do aplicativo, bom ou ruim, em termos de disponibilidade, desempenho e integridade do sistema. Por exemplo, o monitoramento informa se você está atendendo ao seu SLA. O monitoramento ocorre durante a operação normal do sistema. Ele deve ser o mais próximo possível ao tempo real para que a equipe de operações possa reagir rapidamente aos problemas. Idealmente, o monitoramento pode ajudar a impedir problemas que antes levavam a uma falha crítica. Para saber mais, confira [Monitoramento e diagnóstico][monitoring].

**Instrumente para análise da causa raiz**. A análise da causa raiz é o processo de encontrar a causa de falhas. Isso ocorre quando a falha já aconteceu.

**Use o rastreamento distribuído**. Use um sistema de rastreamento distribuído projetado para simultaneidade, assincronia e escala de nuvem. Os rastreamentos devem incluir uma ID de correlação que flua entre os limites dos serviços. Uma única operação pode envolver a chamadas para vários serviços de aplicativo. Se uma operação falhar, a ID de correlação ajuda a identificar a causa da falha.

**Padronize logs e métricas**. A equipe de operações precisará agregar logs entre vários serviços em sua solução. Se cada serviço usar seu próprio formato de log, fica difícil ou impossível obter informações úteis deles. Defina um esquema comum que inclua campos como a ID de correlação, o nome do evento, o endereço IP do remetente e assim por diante. Os serviços individuais podem derivar esquemas personalizados que herdam o esquema base e contêm campos adicionais.

**Automatize tarefas de gerenciamento**, incluindo provisionamento, implantação e monitoramento. A automação de uma tarefa a torna repetível e menos propensa a erros humanos.

**Trate a configuração como código**. Verifique os arquivos de configuração em um sistema de controle de versão, para que você possa rastrear e controlar a versão das alterações e reverter, se necessário.

<!-- links -->

[monitoring]: ../../best-practices/monitoring.md
