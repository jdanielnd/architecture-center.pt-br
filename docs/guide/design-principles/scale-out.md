---
title: Design a aumentar
description: Aplicativos de nuvem devem ser criados para escala horizontal.
author: MikeWasson
ms.openlocfilehash: 8207f322d4312f6a30a8b0db7328b272c1d82de0
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206516"
---
# <a name="design-to-scale-out"></a>Design a aumentar

## <a name="design-your-application-so-that-it-can-scale-horizontally"></a>Desenvolva seu aplicativo para que ele possa ser colocado em escala horizontalmente

Uma vantagem importante da nuvem é a escala elástica &mdash; a habilidade de usar tanta capacidade quanto necessário, expandir conforme a carga aumenta e reduzir a escala quando a capacidade extra não for necessária. Crie seu aplicativo para que ele possa ser dimensionado horizontalmente, adicionando ou removendo novas instâncias como exige.

## <a name="recommendations"></a>Recomendações

**Evitar a adesão de instância**. Adesão ou *afinidade de sessão*, é quando as solicitações do mesmo cliente sempre são roteadas para o mesmo servidor. A adesão limita a capacidade de aumentar o aplicativo. Por exemplo, o tráfego de um usuário de alto volume não será distribuído entre instâncias. Causas de adesão incluem armazenamento de estado de sessão na memória e usar chaves específicas de computador para a criptografia. Todas as instâncias devem conseguir lidar com uma solicitação. 

**Identificar gargalos**. Escalar horizontalmente não é uma correção mágica para todos os problemas de desempenho. Por exemplo, se seu banco de dados de back-end for o gargalo, não adiantará adicionar mais servidores Web. Primeiro identifique e resolva os gargalos do sistema antes de lançar mais instâncias no problema. Partes com estado do sistema são a causa mais provável de gargalos. 

**Decompor cargas de trabalho por requisitos de escalabilidade.**  Os aplicativos geralmente consistem em várias cargas de trabalho, com diferentes requisitos de dimensionamento. Por exemplo, um aplicativo pode ter um site voltado ao público e um site de administração separados. O site público pode apresentar surtos repentinos no tráfego, enquanto o site de administração tem uma carga menor e mais previsível. 

**Descarregar tarefas com uso intenso de recursos.** As tarefas que exigem muitos recursos de CPU ou E/S devem ser movidas para [trabalhos em segundo plano][background-jobs] quando possível para minimizar a carga no front-end que está manipulando as solicitações do usuário.

**Usar recursos de dimensionamento automático internos**. Muitos serviços de computação do Azure têm suporte interno para dimensionamento automático. Se o aplicativo tem uma carga de trabalho previsível e regular, aumente conforme um agendamento. Por exemplo, aumente durante o horário comercial. Caso contrário, se a carga de trabalho não for previsível, use métricas de desempenho, como comprimento da fila de solicitação ou CPU para disparar o dimensionamento automático. Para melhores práticas sobre dimensionamento automático, consulte [Dimensionamento automático][autoscaling].

**Considere dimensionamento automático agressivo para cargas de trabalho críticas**. Para cargas de trabalho críticas, você deseja se manter à frente de demanda. É melhor adicionar novas instâncias rapidamente sob carga pesada para tratar do tráfego adicional e, em seguida, aumentar gradualmente outra vez.

**Design para reduzir horizontalmente**.  Lembre-se de que, com a escala elástica, o aplicativo terá períodos de redução horizontal, quando as instâncias são removidas. O aplicativo deve tratar normalmente instâncias que estão sendo removidas. Aqui estão algumas maneiras de tratar a redução horizontal:

- Escutar eventos de desligamento (quando disponíveis) e desligar corretamente. 
- Clientes/consumidores de um serviço devem dar suporte a tratamento de falhas transitórias e novas tentativas. 
- Para tarefas de longa execução, considere dividir o trabalho, usando pontos de verificação ou o padrão [Pipes e Filtros][pipes-filters-pattern]. 
- Coloque os itens de trabalho em uma fila para que outra instância possa assumir o trabalho se uma instância for removida no meio do processamento. 


<!-- links -->

[autoscaling]: ../../best-practices/auto-scaling.md
[background-jobs]: ../../best-practices/background-jobs.md
[pipes-filters-pattern]: ../../patterns/pipes-and-filters.md
