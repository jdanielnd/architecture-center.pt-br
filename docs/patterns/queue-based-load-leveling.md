---
title: Nivelamento de Carga Baseado em Fila
description: Use uma fila que funcione como um buffer entre uma tarefa e um serviço que ela invoca para simplificar cargas pesadas intermitentes.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- messaging
- availability
- performance-scalability
- resiliency
ms.openlocfilehash: 99b226511fe14bffdab3cdcf65d4e6cffe89bba6
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/08/2017
ms.locfileid: "26359312"
---
# <a name="queue-based-load-leveling-pattern"></a>Padrão de nivelamento de carga baseado em fila

[!INCLUDE [header](../_includes/header.md)]

Use uma fila que funcione como um buffer entre uma tarefa e um serviço que ela invoca para simplificar sobrecargas intermitentes que podem causar falha no serviço ou a fazer a tarefa atingir o tempo limite. Isso pode ajudar a minimizar o impacto dos picos de demanda sobre a disponibilidade e a capacidade de resposta para a tarefa e o serviço.

## <a name="context-and-problem"></a>Contexto e problema

Muitas soluções na nuvem envolvem a execução de tarefas que invocam serviços. Nesse ambiente, se um serviço estiver sujeito a cargas pesadas intermitentes, isso poderá causar problemas de desempenho ou confiabilidade.

Um serviço pode ser parte da mesma solução que as tarefas que o utilizam ou pode ser um serviço de terceiros fornecendo acesso a recursos usados com frequência, como cache ou serviço de armazenamento. Se o mesmo serviço for usado por várias tarefas em execução ao mesmo tempo, poderá ser difícil prever o volume de solicitações para o serviço a qualquer momento.

Um serviço pode apresentar picos de demanda que causam a sobrecarga, não conseguindo responder às solicitações no tempo adequado. Inundar um serviço com um grande número de solicitações simultâneas também pode resultar em falha do serviço se não for possível tratar a contenção que essas solicitações causam.

## <a name="solution"></a>Solução

Refatore a solução e introduza uma fila entre a tarefa e o serviço. A tarefa e o serviço são executados de maneira assíncrona. A tarefa envia uma mensagem que contém os dados exigidos pelo serviço a uma fila. A fila atua como um buffer, armazenando a mensagem até que ela seja recuperada pelo serviço. O serviço recupera as mensagens da fila e as processa. Solicitações de várias tarefas, que podem ser geradas a uma taxa altamente variável, podem ser passadas para o serviço pela mesma fila de mensagens. Esta figura mostra o uso de uma fila para nivelar a carga em um serviço.

![Figura 1 – Como usar uma fila para nivelar a carga em um serviço](./_images/queue-based-load-leveling-pattern.png)

A fila separa as tarefas do serviço e o serviço pode manipular as mensagens em seu próprio ritmo, independentemente do volume de solicitações de tarefas simultâneas. Além disso, não haverá nenhum atraso para uma tarefa se o serviço não estiver disponível no momento em que ele envia uma mensagem à fila.

Esse padrão proporciona os seguintes benefícios:

- Pode ajudar a maximizar a disponibilidade porque atrasos decorrentes de serviços não terão um impacto direto e imediato sobre o aplicativo, que pode continuar a postar mensagens na fila mesmo quando o serviço não está disponível ou atualmente não está processando mensagens.
- Pode ajudar a maximizar a escalabilidade, pois tanto o número de filas quanto o número de serviços pode ser variado para atender à demanda.
- Pode ajudar a controlar os custos, pois o número de instâncias de serviço implantadas só precisa ser suficiente para atender a carga média, em vez da carga de pico.

    >  Alguns serviços implementam limitação quando a demanda atinge um limite além do qual o sistema poderia falhar. A limitação pode reduzir a funcionalidade disponível. Você pode implementar nivelamento de carga com esses serviços para garantir que esse limite não seja atingido.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

- É necessário implementar lógica de aplicativo que controle a taxa à qual os serviços processam as mensagens para evitar sobrecarregar o recurso de destino. Evite passar picos de demanda para o próximo estágio do sistema. Teste o sistema sob carga para garantir que ele ofereça o nivelamento necessário e ajuste o número de filas e o número de instâncias de serviço que manipulam mensagens para conseguir isso.
- Filas de mensagens são um mecanismo de comunicação unidirecional. Se uma tarefa espera uma resposta de um serviço, pode ser necessário implementar um mecanismo que o serviço pode usar para enviar uma resposta. Para obter mais informações, consulte o [Primer de mensagens assíncronas](https://msdn.microsoft.com/library/dn589781.aspx).
- Tenha cuidado ao aplicar o dimensionamento automático a serviços que estão ouvindo solicitações na fila. Isso pode resultar em maior contenção para quaisquer recursos que esses serviços compartilhem e reduzir a eficiência do uso da fila para nivelar a carga.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Esse padrão é útil para qualquer aplicativo que use serviços sujeitos a sobrecarga.

Esse padrão não é útil se o aplicativo esperar uma resposta do serviço com latência mínima.

## <a name="example"></a>Exemplo

Uma função Web do Microsoft Azure armazena dados usando um serviço de armazenamento separado. Se um grande número de instâncias da função web é executado simultaneamente, é possível que o serviço de armazenamento não consiga responder às solicitações com rapidez suficiente para impedir que essas solicitações atinjam o tempo limite ou falhem. Esta figura realça um serviço que está sendo sobrecarregado com um grande número de solicitações simultâneas de instâncias de uma função web.

![Figura 2 – Um serviço que está sendo sobrecarregado com um grande número de solicitações simultâneas de instâncias de uma função web](./_images/queue-based-load-leveling-overwhelmed.png)


Para resolver esse problema, use uma fila para nivelar a carga entre as instâncias de função web e o serviço de armazenamento. No entanto, o serviço de armazenamento é projetado para aceitar solicitações síncronas e não pode ser facilmente modificado para ler mensagens e gerenciar a taxa de transferência. Você pode introduzir uma função de trabalho para atuar como um serviço de proxy que recebe as solicitações da fila e as encaminha ao serviço de armazenamento. A lógica do aplicativo na função de trabalho pode controlar a taxa em que ele transmite solicitações ao serviço de armazenamento para impedir que o serviço de armazenamento seja sobrecarregado. Esta figura ilustra o uso de uma fila e uma função de trabalho para nivelar a carga entre as instâncias de função web e o serviço.

![Figura 3 – Usar uma fila e uma função de trabalho para redistribuir a carga entre as instâncias da função web e o serviço](./_images/queue-based-load-leveling-worker-role.png)

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados

Os padrões e diretrizes a seguir também podem ser relevantes ao implementar esse padrão:

- [Prévia de mensagens assíncronas](https://msdn.microsoft.com/library/dn589781.aspx). Filas de mensagens são inerentemente assíncronas. Talvez seja necessário recriar a lógica do aplicativo em uma tarefa se ela tiver sido adaptada de se comunicar diretamente com um serviço para usar uma fila de mensagens. Da mesma forma, talvez seja necessário refatorar um serviço para aceitar solicitações de uma fila de mensagens. Como alternativa, talvez seja possível implementar um serviço de proxy, conforme descrito no exemplo.
- [Padrão de consumidores concorrentes](competing-consumers.md). Talvez seja possível executar várias instâncias de um serviço, cada uma agindo como um consumidor de mensagem da fila nivelamento de carga. Você pode usar essa abordagem para ajustar a taxa à qual as mensagens são recebidas e passadas para um serviço.
- [Padrão de limitação](throttling.md). Uma maneira simples de implementar a limitação com um serviço é usar o nivelamento de carga baseado em fila e encaminhar todas as solicitações para um serviço por meio de uma fila de mensagens. O serviço pode processar solicitações a uma taxa que garanta que os recursos exigidos pelo serviço não sejam esgotados e para reduzir a quantidade de contenção que poderia ocorrer.
- [Conceitos do serviço Fila](https://msdn.microsoft.com/library/azure/dd179353.aspx). Informações sobre como escolher um mecanismo de mensagens e enfileiramento em aplicativos do Azure.
