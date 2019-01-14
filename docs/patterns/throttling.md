---
title: Padrão de limitação
titleSuffix: Cloud Design Patterns
description: Controle o consumo de recursos usados por uma instância de um aplicativo, um locatário individual ou todo o serviço.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 9babe6b3c81b0846e83dfef98bbd76a89661d911
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54010655"
---
# <a name="throttling-pattern"></a>Padrão de limitação

[!INCLUDE [header](../_includes/header.md)]

Controle o consumo de recursos usados por uma instância de um aplicativo, um locatário individual ou todo o serviço. Isso pode permitir que o sistema continue a funcionar e atender aos contratos de nível de serviço mesmo quando um aumento na demanda coloca uma carga extrema nos recursos.

## <a name="context-and-problem"></a>Contexto e problema

A carga em um aplicativo de nuvem normalmente varia ao longo do tempo com base no número de usuários ativos ou nos tipos de atividades em execução. Por exemplo, provavelmente haverá mais usuários ativos durante o horário comercial ou o sistema poderá precisar executar uma análise computacionalmente dispendiosa no final de cada mês. Também pode haver picos repentinos e inesperados na atividade. Se os requisitos de processamento do sistema excederem a capacidade dos recursos disponíveis, ele sofrerá de desempenho ruim e poderá, inclusive, falhar. Se o sistema precisar atender a um nível combinado de serviço, essa falha poderá ser inaceitável.

Há muitas estratégias disponíveis para tratar a carga variável na nuvem, dependendo dos objetivos comerciais para o aplicativo. Uma estratégia é usar o dimensionamento automático para combinar os recursos provisionados com as necessidades do usuário em qualquer dado momento. Isso tem o potencial de atender consistentemente à demanda do usuário, ao mesmo tempo otimizando os custos em execução. No entanto, embora o dimensionamento automático possa disparar o provisionamento de recursos adicionais, essa configuração não é imediata. Se a demanda crescer rapidamente, poderá haver um período com déficit de recurso.

## <a name="solution"></a>Solução

Uma estratégia alternativa para dimensionamento automático é permitir que os aplicativos usem recursos até um limite e então limitá-los quando esse limite for atingido. O sistema deve monitorar como ele está usando recursos de modo que, quando o uso exceder o limite, ele possa acelerar as solicitações de um ou mais usuários. Isso permitirá que o sistema continue a funcionar e atenda a quaisquer SLAs (contratos de nível de serviço) em vigor. Para obter mais informações sobre como monitorar o uso de recursos, consulte [Orientação sobre instrumentação e telemetria](https://msdn.microsoft.com/library/dn589775.aspx).

O sistema pode implementar várias estratégias de limitação, incluindo:

- Rejeitar solicitações de um usuário individual que já tenha acessado APIs do sistema mais de n vezes por segundo durante um determinado período. Isso requer que o sistema meça o uso de recursos para cada locatário ou usuário que esteja executando um aplicativo. Para obter mais informações, consulte [Orientação de medição de serviço](https://msdn.microsoft.com/library/dn589796.aspx).

- Desabilitar ou comprometer a funcionalidade de serviços não essenciais selecionados de modo que os serviços essenciais possam ser executados sem impedimentos com recursos suficientes. Por exemplo, se o aplicativo estiver transmitindo por streaming saída de vídeo, ele poderá mudar para uma resolução mais baixa.

- Usando nivelamento para suavizar o volume de atividade (essa abordagem é abordada em mais detalhes por [Padrão de nivelamento de carga baseado em fila](./queue-based-load-leveling.md)). Em um ambiente multilocatário, essa abordagem reduzirá o desempenho de cada locatário. Se o sistema precisar dar suporte a uma combinação de locatários com diferentes SLAs, o trabalho para locatários de alto valor poderá ser executado imediatamente. Solicitações para outros locatários poderão ser retidas e manipuladas quando a lista de pendências diminuir. O padrão de Fila de Prioridade][] pode ser usado para implementar essa abordagem.

- Adiar de operações sendo executadas em nome de locatários ou de aplicativos de menor prioridade. Essas operações podem ser suspensas ou limitadas, sendo gerada uma exceção para informar o locatário de que o sistema está ocupado e a operação deverá ser repetida mais tarde.

A figura mostra um gráfico de área para o uso de recursos (uma combinação de memória, CPU, largura de banda e outros fatores) em relação ao tempo para aplicativos que estão usando três recursos. Um recurso é uma área de funcionalidade, como um componente que executa um conjunto específico de tarefas, um trecho de código que executa um cálculo complexo ou um elemento que fornece um serviço como um cache na memória. Esses recursos são rotulados como A, B e C.

![Figura 1 – Gráfico mostrando o uso de recursos em relação ao tempo para aplicativos executados em nome de três usuários](./_images/throttling-resource-utilization.png)

> A área imediatamente abaixo da linha para um recurso indica os recursos usados por aplicativos quando eles invocam esse recurso. Por exemplo, a área abaixo da linha para o Recurso A mostra os recursos usados por aplicativos que estão usando o Recurso A e a área entre as linhas para o Recurso A e o Recurso B indica os recursos usados por aplicativos que estão invocando o Recurso B. Agregar as áreas para cada recurso mostra o uso total de recursos do sistema.

A figura anterior mostra os efeitos de adiar as operações. Logo antes do horário T1, o total de recursos alocados para todos os aplicativos que usam esses recursos atinge um limite (o limite de uso de recurso). Neste ponto, os aplicativos estão em risco de esgotar os recursos disponíveis. Nesse sistema, o Recurso B é menos crítico que o Recurso A ou o Recurso C, assim, ele é temporariamente desabilitado e os recursos que ele estava usando são liberados. Entre os tempos T1 e T2, os aplicativos que usam o Recurso A e o Recurso C continuam em execução como normal. Por fim, o uso de recursos desses dois recursos diminui até um ponto em que, no tempo T2, há capacidade suficiente para habilitar o Recurso B novamente.

As abordagens de dimensionamento automático e a limitação também podem ser combinadas para ajudar a manter os aplicativos responsivo e nos SLAs. Se a expectativa for que a demanda permanecerá alta, a limitação fornecerá uma solução temporária enquanto o sistema é escalado horizontalmente. Neste ponto, a funcionalidade integral do sistema pode ser restaurada.

A figura a seguir mostra um gráfico de área do uso geral de recursos por todos os aplicativos que estão em execução em um sistema em relação ao tempo e ilustra como a limitação pode ser combinada com o dimensionamento automático.

![Figura 2 – Gráfico mostrando os efeitos de combinar limitação com dimensionamento automático](./_images/throttling-autoscaling.png)

No tempo T1, o limite especificando um limite flexível de utilização de recursos é atingido. Neste ponto, o sistema pode começar a aumentar. No entanto, se os recursos novos não ficarem disponíveis rápido o suficiente, os recursos existentes poderão ser esgotados e o sistema poderá falhar. Para evitar que isso ocorra, o sistema será temporariamente limitado, conforme descrito anteriormente. Quando o dimensionamento automático for concluído e recursos adicionais estiverem disponíveis, a limitação poderá ser reduzida.

## <a name="issues-and-considerations"></a>Problemas e considerações

Os seguintes pontos devem ser considerados ao decidir como implementar esse padrão:

- A limitação de um aplicativo e a estratégia a ser usada, é uma decisão de arquitetura que afeta todo o projeto de um sistema. A limitação deve ser considerada no início do processo de design do aplicativo, porque não é fácil adicioná-la depois que um sistema tiver sido implementado.

- A limitação deve ser executada rapidamente. O sistema deve ser capaz de detectar um aumento na atividade e reagir de acordo. O sistema também deve ser capaz de reverter para o estado original rapidamente depois que a carga diminuir. Isso requer que os dados de desempenho apropriados sejam capturados e monitorados continuamente.

- Se um serviço precisar negar temporariamente uma solicitação de usuário, ele deverá retornar um código de erro específico para que o aplicativo cliente compreenda que o motivo da recusa em executar uma operação é a limitação. O aplicativo cliente pode aguardar um período antes de tentar novamente a solicitação.

- A limitação pode ser usada como uma medida temporária enquanto o sistema é dimensionado automaticamente. Em alguns casos, é melhor simplesmente limitar, em vez de dimensionar, se um aumento na atividade for repentino e provavelmente não for durar muito, pois o dimensionamento pode aumentar consideravelmente os custos de execução.

- Se a limitação estiver sendo usada como uma medida temporária enquanto um sistema é dimensionado automaticamente e, se as demandas de recursos aumentarem muito rapidamente, o sistema poderá não conseguir continuar funcionando &mdash; mesmo ao operar em um modo limitado. Se isso não for aceitável, considere a possibilidade de manter maiores reservas de capacidade e configurar um dimensionamento automático mais agressivo.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use este padrão:

- Para garantir que um sistema continue a cumprir os contratos de nível de serviço.

- Para impedir que um único locatário monopolize os recursos fornecidos por um aplicativo.

- Para lidar com estouros de atividade.

- Para ajudar a otimizar os custos de um sistema, limitando os níveis máximos de recursos necessários para mantê-lo funcionando.

## <a name="example"></a>Exemplo

A figura final ilustra como a limitação pode ser implementada em um sistema multilocatário. Os usuários de cada uma das organizações de locatário acessam um aplicativo hospedado na nuvem em que eles preenchem e enviam as pesquisas. O aplicativo contém instrumentação que monitora a taxa à qual esses usuários estão enviando solicitações ao aplicativo.

Para evitar que os usuários de um locatário afetam a capacidade de resposta e a disponibilidade do aplicativo para todos os outros usuários, é aplicado um limite ao número de solicitações por segundo que os usuários de um locatário podem enviar. O aplicativo bloqueia solicitações que excedem esse limite.

![Figura 3 – Implementar limitação em um aplicativo multilocatário](./_images/throttling-multi-tenant.png)

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados

Os padrões e diretrizes a seguir também podem ser relevantes ao implementar esse padrão:

- [Diretrizes sobre Instrumentação e Telemetria](https://msdn.microsoft.com/library/dn589775.aspx). A limitação depende da coleta de informações sobre o quanto um serviço está sendo usado. Descreve como gerar e capturar informações de monitoramento personalizadas.
- [Diretrizes de Medição de Serviço](https://msdn.microsoft.com/library/dn589796.aspx). Descreve como medir o uso de serviços para compreender como eles são usados. Essa informação pode ser útil para determinar como limitar um serviço.
- [Diretrizes de dimensionamento automático](https://msdn.microsoft.com/library/dn589774.aspx). A limitação pode ser usada como uma medida temporária enquanto um sistema é dimensionado automaticamente ou para eliminar a necessidade de dimensionamento automático de um sistema. Contém informações sobre estratégias de dimensionamento automático.
- [Padrão de nivelamento de carga baseado em fila](./queue-based-load-leveling.md). O nivelamento de carga baseado em fila é um mecanismo comumente usado para implementar a limitação. Uma fila pode agir como um buffer que ajuda a estabilizar a taxa em que as solicitações enviadas por um aplicativo são entregues a um serviço.
- [Padrão de fila de prioridade](./priority-queue.md). Um sistema pode usar o enfileiramento prioritário como parte da sua estratégia de limitação para manter o desempenho de aplicativos críticos ou de maior valor, ao mesmo tempo reduzindo o desempenho de aplicativos menos importantes.
