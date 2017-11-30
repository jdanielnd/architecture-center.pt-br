---
title: "Diretrizes de dimensionamento automático"
description: "Diretrizes sobre como realizar o dimensionamento automático para alocar dinamicamente os recursos exigidos por um aplicativo."
author: dragon119
ms.date: 05/17/2017
pnp.series.title: Best Practices
ms.openlocfilehash: f2d42e9d6f4baa2da111c61fe12b48fdec785b92
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="autoscaling"></a>Dimensionamento automático
[!INCLUDE [header](../_includes/header.md)]

O dimensionamento automático é o processo de alocar recursos dinamicamente para atender aos requisitos de desempenho. À medida que o volume de trabalho aumenta, um aplicativo pode precisar de recursos adicionais para manter os níveis de desempenho desejados e atender aos contratos de nível de serviço (SLAs). Conforme a demanda diminui e os recursos adicionais não são mais necessários, é possível desalocá-los para minimizar os custos.

O Dimensionamento Automático aproveita a elasticidade de ambientes hospedados na nuvem ao mesmo tempo em que alivia a sobrecarga de gerenciamento. Ele reduz a necessidade de um operador monitorar constantemente o desempenho de um sistema e toma decisões, como adicionar ou remover recursos.

Há duas maneiras principais de um aplicativo ser dimensionado: 

* **Dimensionamento vertical**, também chamado de aumento e redução vertical, significa que é possível alterar a capacidade de um recurso. Por exemplo, você pode mover um aplicativo para um tamanho maior de VM. Geralmente, o dimensionamento vertical requer que o sistema fique temporariamente indisponível enquanto é reimplantado. Portanto, não é comum automatizar o dimensionamento vertical.
* **Dimensionamento horizontal**, também chamado de expansão e redução horizontal, significa que é possível adicionar ou remover instâncias de um recurso. O aplicativo continua em execução sem interrupções conforme os novos recursos são provisionados. Quando o processo de provisionamento é concluído, a solução é implantada com esses recursos adicionais. Se a demanda cair, os recursos adicionais poderão ser desligados e desalocados. 

Muitos sistemas baseados em nuvem, incluindo o Microsoft Azure, oferecem suporte à automação dessa forma de dimensionamento. O restante deste artigo está voltado para o dimensionamento horizontal.

> [!NOTE]
> O dimensionamento automático se aplica principalmente a recursos de computação. Embora seja possível dimensionar horizontalmente de uma fila de banco de dados ou mensagem, isso normalmente envolve [particionamento de dados][data-partitioning], que geralmente não são automatizados.
>

## <a name="overview"></a>Visão geral

Uma estratégia de dimensionamento automático geralmente envolve as seguintes partes:

* Sistemas de instrumentação e monitoramento nos níveis de aplicativos, serviços e infraestrutura. Esses sistemas capturam métricas essenciais, como tempos de resposta, comprimentos de fila, utilização da CPU e uso de memória.
* A lógica de tomada de decisões avalia essas métricas em relação a limites predefinidos ou agendas e decide se vai dimensionar.
* Componentes que dimensionam o sistema.
* Teste, monitoramento e ajuste da estratégia de dimensionamento automático para garantir que ela funcione conforme o esperado.

O Azure fornece mecanismos de dimensionamento automático internos que atendem cenários comuns. Se um determinado serviço ou tecnologia não tem a funcionalidade de dimensionamento automático interna, ou se você tiver requisitos de dimensionamento automático específicos além dos seus recursos, é possível considerar uma implementação personalizada. Uma implementação personalizada deve coletar e analisar métricas do sistema e operacionais e, em seguida, dimensionar os recursos de acordo.

## <a name="configure-autoscaling-for-an-azure-solution"></a>Configurar o dimensionamento automático para uma solução do Azure

O Azure fornece dimensionamento automático interno para a maioria das opções de computação.

* **Máquinas Virtuais** oferecem suporte ao dimensionamento automático com o uso de [Conjuntos de Dimensionamento de VM][vm-scale-sets], que são uma maneira de gerenciar um conjunto de máquinas virtuais do Azure como um grupo. Confira [Como usar o dimensionamento automático e os Conjuntos de Dimensionamento de Máquinas Virtuais][vm-scale-sets-autoscale].

* **O Service Fabric** também oferece suporte ao dimensionamento automático por meio de Conjuntos de Dimensionamento de VMs. Cada tipo de nó em um cluster do Service Fabric é configurado como um conjunto de dimensionamento de VM separado. Dessa forma, cada tipo de nó pode ser dimensionado independentemente. Confira [Como dimensionar um cluster do Service Fabric usando regras de dimensionamento automático][service-fabric-autoscale].

* **O Serviço de Aplicativo do Azure** tem o dimensionamento automático interno. As configurações de dimensionamento automático se aplicam a todos os aplicativos dentro de um Serviço de Aplicativo. Confira [Como dimensionar a contagem de instância manual ou automaticamente][app-service-autoscale].

* **Os Serviços de Nuvem do Azure** têm o dimensionamento automático interno no nível de função. Confira [Como configurar o dimensionamento automático para um Serviço de Nuvem no portal][cloud-services-autoscale].

Todas essas opções de computação usam o [Dimensionamento automático do Azure Monitor][monitoring] para fornecer um conjunto comum de funcionalidade de dimensionamento automático.

* As **Azure Functions** diferem das opções anteriores de computação porque não é necessário configurar regras de dimensionamento automático. Em vez disso, as Azure Functions alocam a capacidade de computação automaticamente quando seu código é executado, dimensionando conforme o necessário para lidar com a carga. Para obter mais informações, confira [Como escolher o plano de hospedagem correto para as Azure Functions][functions-scale].

Por fim, uma solução de dimensionamento automático personalizado, às vezes, pode ser útil. Por exemplo, você pode usar o diagnóstico do Azure e métricas baseadas no aplicativo, junto com o código personalizado para monitorar e exportar as métricas de aplicativo. Em seguida, você poderia definir regras personalizadas com base nessas métricas e usar APIs REST do Gerenciador de Recursos para acionar o dimensionamento automático. No entanto, uma solução personalizada não é simples de implementar e deve-se considerar essa possibilidade apenas se nenhuma das abordagens anteriores atendem a seus requisitos.

Use os recursos de dimensionamento automático internos da plataforma, se atenderem aos seus requisitos. Caso contrário, avalie cuidadosamente se você realmente precisa de recursos de dimensionamento mais complexos. Alguns exemplos de requisitos adicionais podem incluir mais granularidade de controle, maneiras diferentes para detectar eventos de gatilho para dimensionamento, dimensionamento entre assinaturas e dimensionamento de outros tipos de recursos.

## <a name="use-azure-monitor-autoscale"></a>Como usar o dimensionamento automático do Azure Monitor

O [Dimensionamento automático do Azure Monitor][monitoring] fornece um conjunto comum de funcionalidade de dimensionamento automático para Conjuntos de Dimensionamento de VMs, Serviço de Aplicativo do Azure e Serviço de Nuvem do Azure. O dimensionamento pode ser executado em um cronograma, ou com base em uma métrica de tempo de execução, como o uso de CPU ou memória. Exemplos:

- Expandir 10 instâncias em dias da semana e reduzir 4 instâncias no sábado e domingo. 
- Expandir uma instância se o uso médio da CPU está acima de 70%, e reduzir uma instância se o uso de CPU cai abaixo de 50%.
- Expandir uma instância se o número de mensagens em uma fila exceder um determinado limite.

Para obter uma lista das métricas internas, confira [Métricas comuns de dimensionamento automático do Azure Monitor][autoscale-metrics]. Você também pode implementar métricas personalizadas usando o Application Insights. 

Você pode configurar o dimensionamento automático usando o PowerShell, a CLI do Azure, um modelo do Azure Resource Manager ou o portal do Azure. Para obter mais controle, use a [API REST do Azure Resource Manager](https://msdn.microsoft.com//library/azure/dn790568.aspx). A [Biblioteca de Gerenciamento dos Serviços de Monitoramento do Azure](http://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring) e a [Biblioteca do Microsoft Insights](https://www.nuget.org/packages/Microsoft.Azure.Insights/) (em visualização) são SDKs que permitem a coleta de métricas de recursos diferentes e realizam o dimensionamento automático usando a API REST. Para recursos em que o suporte do Azure Resource Manager não está disponível, ou se você estiver usando os Serviços de Nuvem do Azure, a API REST do Gerenciamento de Serviços pode ser usada para dimensionamento automático. Nos outros casos, use o Azure Resource Manager.

Considere os seguintes pontos ao usar o dimensionamento automático do Azure:

* Avalie se é possível prever a carga no aplicativo com precisão suficiente para usar dimensionamento automático agendado, adicionando e removendo instâncias para atender a picos de demanda previstos. Se isso não for possível, use dimensionamento automático reativo com base nas métricas de tempo de execução para lidar com alterações imprevisíveis na demanda. Normalmente, você pode combinar essas abordagens. Por exemplo, crie uma estratégia que adiciona recursos com agendamento previsto para os momentos de maior demanda do aplicativo. Isso ajuda a garantir que a capacidade esteja disponível quando necessário, sem a demora enfrentada ao iniciar novas instâncias. Além disso, para cada regra agendada, defina métricas que permitam o dimensionamento automático reativo durante esse período para garantir que o aplicativo possa dar suporte a picos de demanda prolongados, mas repentinos.
* Muitas vezes é difícil entender a relação entre os requisitos de capacidade e métricas, especialmente quando um aplicativo é inicialmente implantado. Prefira provisionar um pouco de capacidade extra no início e, então, monitorar e ajustar as regras de dimensionamento automático para aproximar a capacidade da carga real.
* Configure as regras de dimensionamento automático e monitore o desempenho do seu aplicativo ao longo do tempo. Use os resultados desse monitoramento para ajustar a maneira na qual o sistema pode ser dimensionado, se necessário. No entanto, tenha em mente que o dimensionamento automático não é um processo instantâneo. Leva tempo para reagir a uma métrica como a média de uso da CPU que excede (ou fica abaixo) de um limite especificado.
* As regras de dimensionamento automático que usam um mecanismo de detecção com base em um atributo de gatilho de medida (por exemplo, o comprimento da fila ou a utilização de CPU) usam, para disparar uma ação de dimensionamento automático, um valor agregado ao longo do tempo em vez de valores instantâneos. Por padrão, a agregação é uma média dos valores. Isso impede que o sistema reaja rápido demais ou cause oscilação rápida. Ela também dá tempo para que novas instâncias iniciadas automaticamente adequem-se ao modo de execução, impedindo que ações de dimensionamento automático adicionais ocorram enquanto as novas instâncias estão inicializando. Para Serviços de Nuvem e Máquinas Virtuais do Azure, o período padrão para a agregação é 45 minutos, então esse é o período de tempo máximo que poderá levar para a métrica disparar o dimensionamento automático em resposta a picos de demanda. Você pode alterar o período de agregação usando o SDK, mas lembre-se de que os períodos inferiores a 25 minutos podem causar resultados imprevisíveis (para saber mais, confira [Dimensionamento Automático de Serviços de Nuvem segundo o percentual de CPU com a Biblioteca de Gerenciamento de Serviços de Monitoramento do Azure](http://rickrainey.com/2013/12/15/auto-scaling-cloud-services-on-cpu-percentage-with-the-windows-azure-monitoring-services-management-library/)). Para aplicativos Web, o período de cálculo da média é muito menor, permitindo que novas instâncias estejam disponíveis cerca de cinco minutos após uma alteração à medida de gatilho segundo a média.
* Se configurar o dimensionamento automático usando o SDK em vez do portal, você poderá especificar uma agenda mais detalhada durante a qual as regras estão ativas. Você também pode criar suas próprias métricas e usá-las, com ou sem as métricas existentes, em suas regras de dimensionamento automático. Por exemplo, você poderá usar contadores alternativos como o número de solicitações por segundo ou a disponibilidade média de memória, ou ainda usar contadores personalizados que medem processos comerciais específicos.
* Ao realizar um dimensionamento automático no Service Fabric, os tipos de nós no cluster são compostos de conjuntos de dimensionamento de VM no back-end. Então, você precisa configurar as regras de dimensionamento automático para cada tipo de nó. Leve em conta o número de nós que você precisa ter antes de configurar o dimensionamento automático. O número mínimo de nós necessários para o tipo de nó primário é controlado pelo nível de confiabilidade escolhido. Para obter mais informações, confira [Como dimensionar um cluster do Service Fabric usando regras de dimensionamento automático](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-cluster-scale-up-down).
* Você pode usar o portal para vincular recursos como instâncias do Banco de Dados SQL e filas a uma instância do Serviço de Nuvem. Isso permite que você acesse mais facilmente as opções de configuração de dimensionamento manual e automático separadas, para cada um dos recursos vinculados. Para obter mais informações, confira [Como vincular um recurso a um serviço de nuvem](/azure/cloud-services/cloud-services-how-to-manage).
* Ao configurar várias políticas e regras, elas poderão entrar em conflito entre si. O Dimensionamento Automático usa as regras de resolução de conflitos a seguir para garantir que haja sempre um número suficiente de instâncias em execução:
  * As operações de escalamento horizontal sempre têm precedência sobre operações de redução horizontal.
  * Quando há conflito entre operações de escalamento horizontal, a regra que inicia o maior aumento no número de instâncias tem precedência.
  * Quando há conflito entre operações de redução horizontal, a regra que inicia a menor redução no número de instâncias tem precedência.
* Em um Ambiente do Serviço de Aplicativo, qualquer métrica de pool de trabalho ou de front-end pode ser usada para definir regras de dimensionamento automático. Para obter mais informações, confira [Dimensionamento automático e Ambiente do Serviço de Aplicativo](/azure/app-service/app-service-environment-auto-scale).

## <a name="application-design-considerations"></a>Considerações sobre o design de aplicativo
O dimensionamento automático não é uma solução imediata. Apenas adicionar recursos a um sistema ou executar mais instâncias de um processo não garante que o desempenho do sistema vai melhorar. Considere os seguintes pontos ao criar uma estratégia de dimensionamento automático:

* O sistema deve ser projetado para ser escalonável horizontalmente. Evite fazer suposições sobre afinidade de instância; não crie soluções que exijam que o código esteja sempre sendo executado em uma instância específica de um processo. Ao dimensionar um site ou serviço de nuvem horizontalmente, não presuma que uma série de solicitações da mesma fonte será sempre roteada para a mesma instância. Projete os serviços, por essa mesma razão, como sem monitoração de estado, evitando assim a exigência de que uma série de solicitações de um aplicativo sejam sempre roteadas para a mesma instância de um serviço. Ao criar um serviço que lê as mensagens de uma fila e as processa, não faça suposições sobre qual instância do serviço lidará com qual mensagem. O Dimensionamento Automático pode iniciar instâncias adicionais de um serviço conforme o comprimento da fila cresce. O [Padrão de Consumidores Concorrentes][competing-consumers] descreve como administrar esse cenário.
* Se a solução implementa uma tarefa de execução longa, projete-a para oferecer suporte tanto a escalar quanto a reduzir horizontalmente. Sem o devido cuidado, essa tarefa pode impedir que uma instância de um processo seja desligada corretamente quando o sistema é reduzido horizontalmente, ou ela poderá perder dados se o processo for encerrado. Idealmente, refatore uma tarefa de execução longa e divida o processamento que ela executa em blocos menores e distintos. O [Padrão de Filtros e Tubos][pipes-and-filters] fornece um exemplo de como atingir esse resultado.
* Como alternativa, você pode implementar um mecanismo de ponto de verificação que registre informações de estado da tarefa em intervalos regulares, e salve esse estado em armazenamento durável que possa ser acessado por qualquer instância do processo executando a tarefa. Desse modo, se o processo é encerrado, o trabalho que ele estava executando poderá ser retomado do último ponto de verificação, por meio do uso de outra instância.
* Quando tarefas em segundo plano são executadas em instâncias de computação separadas, como em funções de trabalho de um aplicativo hospedado em Serviços de Nuvem, talvez seja necessário dimensionar diferentes partes do aplicativo usando diferentes políticas de dimensionamento. Por exemplo, talvez seja necessário implantar instâncias de computação de IU (interface do usuário) adicionais sem aumentar o número de instâncias de computação em segundo plano, ou fazer o oposto disso. Se você oferecer níveis diferentes de serviço (como pacotes de serviço básicos e premium), talvez seja necessário escalar horizontalmente os recursos de computação para pacotes de serviço premium de modo mais agressivo do que aqueles para pacotes de serviço básico, para assim atender aos SLAs.
* Considere usar o comprimento da fila, pela qual a interface do usuário e as instâncias de computação em segundo plano se comunicam, como um critério da sua estratégia de dimensionamento automático. Esse é o melhor indicador de um desequilíbrio ou a diferença entre a carga atual e a capacidade de processamento da tarefa em segundo plano.
* Se sua estratégia de dimensionamento automático em contadores que medem processos de negócios, como o número de pedidos feitos por hora ou o tempo médio de execução de uma transação complexa, certifique-se de entender completamente a relação entre os resultados desses tipos de contadores e os reais requisitos de capacidade de computação. Pode ser necessário dimensionar mais de um componente ou unidade de computação em resposta a alterações nos contadores de processo empresarial.  
* Para evitar que o sistema tente escalar horizontalmente de modo excessivo e para evitar os custos associados à execução de vários milhares de instâncias, considere a possibilidade de limitar o número máximo de instâncias que podem ser adicionadas automaticamente. A maioria dos mecanismos de dimensionamento automático permitem que você especifique os números mínimo e máximo de instâncias para uma regra. Além disso, considere degradar normalmente a funcionalidade que o sistema oferece se o número máximo de instâncias tiverem sido implantadas e, ainda assim, o sistema estiver sobrecarregado.
* Mantenha em mente que dimensionamento automático pode não ser o mecanismo mais apropriado para lidar com um aumento repentino na carga de trabalho. Leva tempo para provisionar e iniciar novas instâncias de um serviço ou adicionar recursos a um sistema, e o pico da demanda pode já ter passado quando esses recursos adicionais forem disponibilizados. Nesse cenário, talvez seja melhor limitar o serviço. Para obter mais informações, confira [Padrão de Limitação][throttling].
* Por outro lado, se você precisa da capacidade de processar todas as solicitações quando o volume flutua rapidamente e o custo não é um fator principal, considere usar uma estratégia de dimensionamento automático agressiva que inicia as instâncias adicionais mais rapidamente. Você também pode usar uma política programada que inicie um número suficiente de instâncias para atender à carga máxima antes que ela seja esperada.
* O mecanismo de dimensionamento automático deve monitorar o processo de dimensionamento automático e registrar os detalhes de cada evento desse dimensionamento (o que o disparou, quais recursos foram adicionados ou removidos e quando isso ocorreu). Se você criar um mecanismo de dimensionamento automático personalizado, certifique-se de que ele incorpora essa funcionalidade. Analise as informações para ajudar a medir a eficiência da estratégia de dimensionamento automático e ajustá-la se necessário. Você pode ajustar tanto a curto prazo, conforme os padrões de uso tornam-se mais óbvios, quanto a longo prazo, conforme os negócios se expandem ou os requisitos do aplicativo evoluem. Se um aplicativo atinge o limite superior definido para o dimensionamento automático, o mecanismo também pode alertar um operador, que pode iniciar recursos adicionais manualmente se necessário. Observe que, sob essas circunstâncias, o operador também pode ser responsável por remover manualmente esses recursos após redução na carga de trabalho.

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados
Os padrões e diretrizes a seguir também podem ser relevantes a esse cenário ao implementar o dimensionamento automático:

* [Padrão de Limitação][throttling]. Esse padrão descreve como um aplicativo pode continuar a funcionar e cumprir os contratos de nível de serviço quando um aumento na demanda coloca uma carga extrema sobre os recursos. É possível usar limitação com o dimensionamento automático para evitar que, ao escalar horizontalmente o sistema, este seja sobrecarregado.
* [Padrão de Consumidores Concorrentes][competing-consumers]. Esse padrão descreve como implementar um conjunto de instâncias de serviço que pode lidar com mensagens de qualquer instância do aplicativo. O dimensionamento automático pode ser usado para iniciar e parar as instâncias de serviço de acordo com a carga de trabalho prevista. Essa abordagem permite que um sistema processe várias mensagens simultaneamente para otimizar o resultado, melhorar a escalabilidade e disponibilidade e equilibrar a carga de trabalho.
* [Monitoramento e diagnósticos](./monitoring.md). Instrumentação e telemetria são essenciais para reunir as informações que podem orientar o processo de dimensionamento automático.


<!-- links -->

[monitoring]: /azure/monitoring-and-diagnostics/monitoring-overview-autoscale
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale?toc=%2fazure%2fapp-service-web%2ftoc.json#scaling-based-on-a-pre-set-metric
[app-service-plan]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[autoscale-metrics]: /azure/monitoring-and-diagnostics/insights-autoscale-common-metrics
[cloud-services-autoscale]: /azure/cloud-services/cloud-services-how-to-scale-portal
[competing-consumers]: ../patterns/competing-consumers.md
[data-partitioning]: ./data-partitioning.md
[functions-scale]: /azure/azure-functions/functions-scale
[link-resource-to-cloud-service]: /azure/cloud-services/cloud-services-how-to-manage#how-to-link-a-resource-to-a-cloud-service
[pipes-and-filters]: ../patterns/pipes-and-filters.md
[service-fabric-autoscale]: /azure/service-fabric/service-fabric-cluster-scale-up-down
[throttling]: ../patterns/throttling.md
[vm-scale-sets]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vm-scale-sets-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
