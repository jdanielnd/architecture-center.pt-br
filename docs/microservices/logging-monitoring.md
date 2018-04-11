---
title: Registro em log e monitoramento em microsserviços
description: Registro em log e monitoramento em microsserviços
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 1da67047daa9ae87cda5dd7dd581d6081183c428
ms.sourcegitcommit: 786bafefc731245414c3c1510fc21027afe303dc
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/12/2017
---
# <a name="designing-microservices-logging-and-monitoring"></a>Projetando microsserviços: registro em log e monitoramento

Em qualquer aplicativo complexo, em algum momento, algo dará errado. Em um aplicativo de microsserviço, você precisa controlar o que está acontecendo em dúzias ou até mesmo centenas de serviços. O monitoramento e o registro em log são extremamente importantes para fornecer uma visão holística do sistema. 

![](./images/monitoring.png)

Em uma arquitetura de microsserviços, descobrir a causa exata de erros ou gargalos de desempenho pode ser algo especialmente desafiador. Uma única operação de usuário pode abranger vários serviços. Serviços podem atingir os limites de E/S de rede dentro do cluster. Uma cadeia de chamadas entre serviços pode causar pressão de retorno no sistema, resultando em latência alta ou em falhas em cascata. Além disso, geralmente você não sabe em qual nó um contêiner específico será executado. Contêineres colocados no mesmo nó podem estar competindo por CPU ou memória limitada. 

Para entender o que está acontecendo, o aplicativo deve emitir eventos de telemetria. Você pode categorizar essas métricas e logs baseados em texto. 

*Métricas* são valores numéricos que podem ser analisados. Você pode usá-los para observar o sistema em tempo real (ou quase em tempo real) ou então para analisar as tendências de desempenho ao longo do tempo. As métricas incluem:

- Métricas do sistema no nível de nó, incluindo CPU, memória, rede, disco e uso do sistema de arquivos. As métricas do sistema ajudam a compreender a alocação de recurso para cada nó no cluster e exceções de solução de problemas.
 
- Métricas de Kubernetes. Já que os serviços são executados em contêineres, você precisa coletar métricas no nível do contêiner, não apenas no nível de VM. No Kubernetes, o cAdvisor (Assistente de Contêiner) é o agente que coleta estatísticas sobre a CPU, memória, sistema de arquivos e recursos de rede usados por cada contêiner. O daemon kubelet coleta estatísticas de recursos do cAdvisor e as expõe por meio de uma API REST.
   
- Métricas de aplicativo. Isso inclui as métricas que são relevantes para entender o comportamento de um serviço. Exemplos incluem o número de solicitações HTTP de entrada na fila, latência de solicitação, comprimento da fila de mensagens ou número de transações processadas por segundo.

- Métricas de serviço dependentes. Serviços de cluster podem chamar serviços externos que estão fora do cluster, tais como serviços de PaaS gerenciados. Você pode monitorar os Serviços do Azure usando o [Azure Monitor](/azure/monitoring-and-diagnostics/monitoring-overview). Os serviços de terceiros podem ou não fornecer alguma métrica. Se não fornecerem, você dependerá de suas próprias métricas de aplicativo para acompanhar as estatísticas de latência e taxa de erros.

*Logs* são registros de eventos que ocorrem durante a execução do aplicativo. Eles incluem itens como logs de aplicativos (instruções de rastreamento) ou logs do servidor Web. Os logs são úteis principalmente para análise da causa raiz e análise forense. 

## <a name="considerations"></a>Considerações

O artigo [Monitoramento e diagnóstico](../best-practices/monitoring.md) descreve práticas recomendadas gerais para monitorar um aplicativo. Aqui estão algumas coisas específicas para se pensar no contexto de uma arquitetura de microsserviços.

**Configuração e gerenciamento**. Você usará um serviço gerenciado para monitoramento e registro em log ou implantará os componentes de registro em log e monitoramento como contêineres dentro do cluster? Para mais informações sobre essas opções, consulte a seção [Opções de tecnologia](#technology-options) abaixo.

**Taxa de ingestão**. Qual é a taxa de transferência na qual o sistema pode incluir eventos de telemetria? O que acontece se essa taxa é excedida? Por exemplo, o sistema pode limitar os clientes, caso em que os dados de telemetria são perdidos, ou pode reduzir a amostra de dados. Às vezes, você pode atenuar esse problema reduzindo a quantidade de dados coletados:

  - Agregue métricas calculando estatísticas como média e desvio padrão e envie esses dados estatísticos para o sistema de monitoramento.  

  - Reduza a amostra de dados &mdash; ou seja, processe apenas um percentual dos eventos.

  - Envie os dados em lotes para reduzir o número de chamadas de rede para o serviço de monitoramento.

**Custo**. O custo de ingerir e armazenar dados de telemetria pode ser alto, especialmente em grandes volumes. Em alguns casos, ele pode até mesmo exceder o custo de executar o aplicativo. Nesse caso, convém reduzir o volume de telemetria agregando os dados, reduzindo sua resolução ou enviando-os em lotes, conforme descrito acima. 
        
**Fidelidade de dados**. Qual o nível de precisão das métricas? Médias podem ocultar exceções, especialmente em escala. Além disso, se a taxa de amostragem é muito baixa, ela pode suavizar flutuações nos dados. Pode parecer que tem todas as solicitações têm aproximadamente a mesma latência de ponta a ponta, quando na verdade uma parte significativa das solicitações estão demorando muito mais. 

**Latência**. Para habilitar alertas e monitoramento em tempo real, os dados de telemetria devem estar disponíveis rapidamente. Quanto a disponibilização dos dados que aparecem no painel de monitoramento realmente se aproxima de "tempo real"? Com alguns segundos de atraso? Mais de um minuto?

**Armazenamento.** Para logs, pode ser mais eficiente para gravar os eventos de log em armazenamento efêmero no cluster e configurar um agente para enviar os arquivos de log para armazenamento mais persistente.  Dados devem ser movidos eventualmente para o armazenamento de longo prazo para que ele esteja disponível para análise retrospectiva. Uma arquitetura de microsserviços pode gerar um grande volume de dados de telemetria, portanto, o custo de armazenar esses dados é uma consideração importante. Além disso, considere como você consultará os dados. 

**Painel e visualização.** Você tem uma visão holística do sistema, em todos os serviços, tanto de dentro do cluster quanto dos serviços externos? Se você estiver gravando dados de telemetria e logs em mais de um local, o painel poderá mostrar todos eles e correlacioná-los? O painel de monitoramento deve mostrar pelo menos as seguintes informações:

- Alocação de recurso geral para crescimento e capacidade. Isso inclui o número de contêineres, as métricas do sistema de arquivos, rede e alocação de núcleos.
- Métricas de contêiner correlacionadas no nível de serviço.
- As métricas do sistema correlacionadas a contêineres.
- Erros de serviço e exceções.
    

## <a name="distributed-tracing"></a>Rastreamento distribuído

Conforme mencionado, um desafio em microsserviços é entender o fluxo de eventos entre os serviços. Uma única operação ou transação pode envolver a chamadas para vários serviços. Para reconstruir a toda sequência de etapas, cada serviço deve propagar uma *ID de correlação* que atua como um identificador exclusivo para essa operação. A ID de correlação habilita o [rastreamento distribuído](http://microservices.io/patterns/observability/distributed-tracing.html) entre serviços.

O primeiro serviço que recebe uma solicitação de cliente deve gerar a ID de correlação. Se o serviço faz uma chamada HTTP para outro serviço, ele coloca a ID de correlação em um cabeçalho de solicitação. Da mesma forma, se o serviço envia uma mensagem assíncrona, ele coloca a ID de correlação nessa mensagem. Serviços de downstream continuam a propagar a ID de correlação, de modo que ela flui através de todo o sistema. Além disso, todo o código que grava eventos de log ou métricas do aplicativo deve incluir a ID de correlação.

Quando as chamadas de serviço são correlacionadas, você pode calcular métricas operacionais, como a latência de ponta a ponta para uma transação completa, o número de transações bem-sucedidas por segundo e o percentual de transações com falha. Incluir IDs de correlação nos logs de aplicativo possibilita realizar a análise da causa raiz. Se uma operação falhar, você poderá encontrar as instruções de log para todas as chamadas de serviço que fizerem parte da mesma operação. 

Aqui estão algumas considerações ao implementar o rastreamento distribuído:

- Atualmente, não há nenhum cabeçalho HTTP padrão para IDs de correlação. Sua equipe deve padronizar em um valor de cabeçalho personalizado. Essa escolha pode ser decidida por sua estrutura de monitoramento/registro em log ou sua escolha de malha de serviço.

- Para mensagens assíncronas, se sua infraestrutura de mensagens é compatível com a adição de metadados a mensagens, você deve incluir a ID de correlação como metadados. Caso contrário, inclua-a como parte do esquema de mensagem.

- Em vez de um único identificador opaco, você pode enviar um *contexto de correlação* que inclui informações mais avançadas, tais como relações chamador/receptor. 

- O SDK do Azure Application Insights injeta automaticamente o contexto de correlação em cabeçalhos HTTP e inclui a ID de correlação nos logs do Application Insights. Se você decidir usar recursos de correlação do Application Insights, alguns serviços ainda precisarão propagar os cabeçalhos de correlação explicitamente, dependendo de bibliotecas que estiverem sendo usadas. Para obter mais informações, consulte [Correlação de telemetria no Application Insights](/azure/application-insights/application-insights-correlation).
   
- Se você estiver usando Istio ou linkerd como uma malha de serviço, essas tecnologias gerarão cabeçalhos de correlação automaticamente quando chamadas HTTP forem roteadas por meio dos proxies de malha de serviço. Os serviços devem encaminhar os cabeçalhos pertinentes. 

    - Istio: [rastreamento de solicitações distribuído](https://istio-releases.github.io/v0.1/docs/tasks/zipkin-tracing.html)
    
    - linkerd: [cabeçalhos de contexto](https://linkerd.io/config/1.3.0/linkerd/index.html#http-headers)
    
- Considere o modo como você agregará os logs. Você talvez queira que o modo como as IDs de correlação são incluídas nos logs seja padronizado entre as equipes. Use um formato estruturado ou semiestruturado, tal como JSON, e defina um campo comum para conter a ID de correlação.

## <a name="technology-options"></a>Opções de tecnologia

**Application Insights** é um serviço gerenciado do Azure que ingere e armazena dados de telemetria e fornece ferramentas para pesquisar por dados e analisá-los. Para usar o Application Insights, você pode instalar um pacote de instrumentação em seu aplicativo. Esse pacote monitora o aplicativo e envia dados de telemetria para o serviço do Application Insights. Ele também pode efetuar pull de dados de telemetria do ambiente de host. O Application Insights fornece correlação e acompanhamento de dependência internos. Ele permite controlar as métricas do sistema, de aplicativo e de serviço do Azure, em um só lugar.

Lembre-se de que o Application Insights será limitado se a taxa de dados exceder um limite máximo; para obter detalhes, consulte [Limites do Application Insights](/azure/azure-subscription-service-limits#application-insights-limits). Uma única operação pode gerar vários eventos de telemetria, portanto, se o aplicativo passar por um alto volume de tráfego, ele provavelmente será limitado. Para atenuar esse problema, você pode executar amostragem para reduzir o tráfego de telemetria. A desvantagem é que suas métricas serão menos precisas. Para obter mais informações, consulte [Amostragem no Application Insights](/azure/application-insights/app-insights-sampling). Você também pode reduzir o volume de dados pré-agregando métricas &mdash; ou seja, calculando valores estatísticos como média e desvio padrão e enviando esses valores em lugar da telemetria bruta. A postagem no blog a seguir descreve uma abordagem para uso do Application Insights em escala: [Monitoramento do Azure e análise em escala](https://blogs.msdn.microsoft.com/azurecat/2017/05/11/azure-monitoring-and-analytics-at-scale/).

Além disso, verifique se você compreende o modelo de preços para o Application Insights, porque a cobrança é feita com base no volume de dados. Para obter mais informações, consulte [Gerenciar o preço e o volume de dados no Application Insights](/azure/application-insights/app-insights-pricing). Se seu aplicativo gera um grande volume de telemetria e você não deseja realizar amostragem nem agregação dos dados, o Application Insights pode não ser a escolha apropriada. 

Se o Application Insights não atende às suas necessidades, aqui estão algumas abordagens sugeridas que usam tecnologias de software livre populares.

Para métricas de sistema e de contêiner, considere exportar métricas para um banco de dados de série temporal como **Prometheus** ou **InfluxDB** em execução no cluster.

- InfluxDB é um sistema baseado em push. É necessário que um agente envie as métricas por push. Você pode usar o [Heapster][heapster], que é um serviço que coleta métricas de todo o cluster do kubelet, agrega os dados e envia-os por push para InfluxDB ou outra solução de armazenamento de série temporal. O Serviço de Contêiner do Azure implanta o Heapster como parte da configuração do cluster. Outra opção é o [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/), que é um agente para coletar e relatar métricas. 

- O Prometheus é um sistema baseado em pull. Ele extrai periodicamente as métricas de locais configurados. O Prometheus pode extrair métricas geradas por cAdvisor ou métricas de estado kube. [métricas de estado kube][kube-state-metrics] é um serviço que coleta métricas do servidor de API do Kubernetes e as disponibiliza para o Prometheus (ou um extrator que seja compatível com um ponto de extremidade de cliente do Prometheus). Embora o Heapster agregue as métricas geradas pelo Kubernetes e encaminhe-as para um coletor, o serviço métricas de estado kube gera suas próprias métricas e disponibiliza-as por meio de um ponto de extremidade para extração. Para métricas do sistema, use o [Exportador de nó](https://github.com/prometheus/node_exporter), que é um exportador Prometheus para métricas do sistema. O Prometheus é compatível com os dados de ponto flutuante, mas não com os dados de cadeia de caracteres, portanto, ele é apropriado para as métricas do sistema, mas não para logs.

- Usar uma ferramenta de painel, tal como **Kibana** ou **Grafana**, para visualizar e monitorar os dados. O serviço de painel também pode ser executado dentro de um contêiner no cluster.

Para logs de aplicativo, considere o uso de **Fluentd** e **Elasticsearch**. Fluentd é um coletor de dados de software livre e Elasticsearch é um banco de dados de documento que é otimizado para atuar como um mecanismo de pesquisa. Usando essa abordagem, cada serviço envia logs para `stdout` e `stderr` e o Kubernetes grava esses fluxos para o sistema de arquivos local. O Fluentd coleta os logs, opcionalmente enriquece-os com metadados adicionais do Kubernetes e envia os logs para o Elasticsearch. Use o Kibana, o Grafana ou uma ferramenta semelhante para criar um painel para o Elasticsearch. O Fluentd é executado como um conjunto de daemons no cluster, o que garante que um pod do Fluentd é atribuído a cada nó. Você pode configurar o Fluentd para coletar logs do kubelet, bem como os logs de contêiner. Em grandes volumes, gravar logs no sistema de arquivos local pode se tornar um gargalo de desempenho, especialmente quando vários serviços estão em execução no mesmo nó. Monitore a latência do disco e a utilização do sistema de arquivos na produção.

Uma vantagem de usar Fluentd com Elasticsearch para logs é que os serviços não exigem nenhuma dependência de biblioteca adicional. Cada serviço grava apenas para `stdout` e `stderr` e o Fluentd fica responsável pela exportação dos logs para o Elasticsearch. Além disso, as equipes que escrevem código de serviços não precisam entender como configurar a infraestrutura de log. Um desafio é configurar o cluster Elasticsearch para uma implantação de produção de modo que ele seja dimensionado para lidar com o tráfego. 

Outra opção é enviar logs para o Log Analytics do OMS (Operations Management Suite). O serviço [Log Analytics][log-analytics] coleta dados de log em um repositório central e também pode consolidar dados de outros serviços do Azure usados pelo seu aplicativo. Para obter mais informações, consulte [Monitorar um cluster do Serviço de Contêiner do Azure com o Microsoft Operations Management Suite (OMS)][k8s-to-oms].

## <a name="example-logging-with-correlation-ids"></a>Exemplo: registro em log com IDs de correlação

Para ilustrar alguns dos pontos abordados neste capítulo, aqui está um exemplo estendido de como o serviço Package implementa o registro em log. O serviço Package foi escrito em TypeScript e usa a estrutura [Koa](http://koajs.com/) da Web para Node.js. Há várias bibliotecas de registro em log do Node.js à sua escolha. Escolhemos [Winston](https://github.com/winstonjs/winston), uma biblioteca de log popular que atendeu a nossos requisitos de desempenho quando testada.

Para encapsular os detalhes de implementação, definimos uma interface `ILogger` abstrata:

```ts
export interface ILogger {
    log(level: string, msg: string, meta?: any)
    debug(msg: string, meta?: any)
    info(msg: string, meta?: any)
    warn(msg: string, meta?: any)
    error(msg: string, meta?: any)
}
```

Aqui está uma implementação de `ILogger` que encapsula a biblioteca Winston. Ela usa a ID de correlação como um parâmetro de construtor e injeta a ID em todas as mensagens de log. 

```ts
class WinstonLogger implements ILogger {
    constructor(private correlationId: string) {}
    log(level: string, msg: string, payload?: any) {
        var meta : any = {};
        if (payload) { meta.payload = payload };
        if (this.correlationId) { meta.correlationId = this.correlationId }
        winston.log(level, msg, meta)
    }
  
    info(msg: string, payload?: any) {
        this.log('info', msg, payload);
    }
    debug(msg: string, payload?: any) {
        this.log('debug', msg, payload);
    }
    warn(msg: string, payload?: any) {
        this.log('warn', msg, payload);
    }
    error(msg: string, payload?: any) {
        this.log('error', msg, payload);
    }
}
```

O serviço Package precisa extrair a ID de correlação da solicitação HTTP. Por exemplo, se você estiver usando linkerd, a ID de correlação será encontrada no cabeçalho `l5d-ctx-trace`. No Koa, a solicitação HTTP é armazenada em um objeto Context que é passado por meio da pipeline de processamento de solicitação. Podemos definir uma função de middleware para obter a ID de correlação do Context e inicializar o agente. (Uma função de middleware em Koa é simplesmente uma função que é executada para cada solicitação.)

```ts
export type CorrelationIdFn = (ctx: Context) => string;

export function logger(level: string, getCorrelationId: CorrelationIdFn) {
    winston.configure({ 
        level: level,
        transports: [new (winston.transports.Console)()]
        });
    return async function(ctx: any, next: any) {
        ctx.state.logger = new WinstonLogger(getCorrelationId(ctx));
        await next();
    }
}
```

Esse middleware chama uma função definida pelo chamador, `getCorrelationId`, para obter a ID de correlação. Em seguida, ele cria uma instância do agente e armazena-a dentro de `ctx.state`, que é um dicionário de chave-valor usado em Koa para passar informações por meio do pipeline. 

O middleware do agente é adicionado ao pipeline na inicialização:

```ts
app.use(logger(Settings.logLevel(), function (ctx) {
    return ctx.headers[Settings.correlationHeader()];  
}));
```

Depois que tudo está configurado, é fácil adicionar instruções de registro em log ao código. Por exemplo, aqui está o método que pesquisa um pacote. Ele faz duas chamadas para o método `ILogger.info`.

```ts
async getById(ctx: any, next: any) {
  var logger : ILogger = ctx.state.logger;
  var packageId = ctx.params.packageId;
  logger.info('Entering getById, packageId = %s', packageId);

  await next();

  let pkg = await this.repository.findPackage(ctx.params.packageId)

  if (pkg == null) {
    logger.info(`getById: %s not found`, packageId);
    ctx.response.status= 404;
    return;
  }

  ctx.response.status = 200;
  ctx.response.body = this.mapPackageDbToApi(pkg);
}
```

Não precisamos incluir a ID de correlação em instruções de registro em log, porque isso é feito automaticamente pela função de middleware. Isso torna o código de log mais limpo e reduz a chance de que um desenvolvedor esqueça de incluir a ID de correlação. Já que todas as instruções de registro em log usam a interface `ILogger` abstrata, seria fácil substituir a implementação do agente posteriormente.

> [!div class="nextstepaction"]
> [Integração e entrega contínuas](./ci-cd.md)

<!-- links -->

[app-insights]: /azure/application-insights/app-insights-overview
[heapster]: https://github.com/kubernetes/heapster
[kube-state-metrics]: https://github.com/kubernetes/kube-state-metrics
[k8s-to-oms]: /azure/container-service/kubernetes/container-service-kubernetes-oms
[log-analytics]: /azure/log-analytics/