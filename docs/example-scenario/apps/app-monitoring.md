---
title: Monitoramento de aplicativo Web no Azure
description: Monitore um aplicativo Web hospedado no Serviço de Aplicativo do Azure.
author: adamboeglin
ms.date: 09/12/2018
ms.openlocfilehash: ba008035c37d1d4e2d2f823463344e4941c0b4c4
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610746"
---
# <a name="web-application-monitoring-on-azure"></a>Monitoramento de aplicativo Web no Azure

Ofertas de plataforma como serviço (PaaS) no Azure gerenciam recursos de computação para você e afetam como você monitora implantações. O Azure inclui vários serviços de monitoramento, e cada um realiza uma função específica. Juntos, esses serviços oferecem uma solução abrangente para coletar, analisar e agir na telemetria do seu aplicativo e os recursos do Azure que eles consomem.

Esse cenário aborda os serviços de monitoramento que você pode usar e descreve um modelo de fluxo de dados para uso com várias fontes de dados. Quando se trata de monitoramento, várias ferramentas e serviços funcionam com implantações do Azure. Nesse cenário, escolhemos serviços prontamente disponíveis, pois eles são fáceis de usar. Outras opções de monitoramento serão discutidas mais adiante neste artigo.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Outros casos de uso relevantes incluem:

- Instrumentação de um aplicativo Web para monitoramento de telemetria.
- Coleta de telemetria de front-end e back-end para um aplicativo implantado no Azure.
- Monitoramento de métricas e cotas associadas aos serviços no Azure.

## <a name="architecture"></a>Arquitetura

![Diagrama da Arquitetura do Monitor de Aplicativo][architecture]

Esse cenário usa um ambiente do Azure gerenciado para hospedar um aplicativo e a camada de dados. O fluxo de dados neste cenário ocorre da seguinte forma:

1. Um usuário interage com o aplicativo.
2. O serviço de aplicativo e o navegador emitem a telemetria.
3. O Application Insights coleta e analisa dados de integridade, de desempenho e de uso de aplicativos.
4. Os desenvolvedores e administradores podem examinar informações de integridade, desempenho e uso.
5. O Banco de Dados SQL do Azure emite telemetria.
6. O Azure Monitor coleta e analisa cotas e métricas de infraestrutura.
7. O Log Analytics coleta e analisa logs e métricas.
8. Os desenvolvedores e administradores podem examinar informações de integridade, desempenho e uso.

### <a name="components"></a>Componentes

- O [Serviço de Aplicativo do Azure](/azure/app-service/) é um serviço de PaaS para compilar e hospedar aplicativos em máquinas virtuais gerenciadas. As infraestruturas de computação subjacentes em que os aplicativos são executados são gerenciadas para você. O Serviço de Aplicativo fornece monitoramento de cotas de uso de recursos e métricas de aplicativo, registro em log de informações de diagnóstico e alertas com base nas métricas. Melhor ainda, você pode usar o Application Insights para criar [testes de disponibilidade][availability-tests] para testar o aplicativo em diferentes regiões.
- O [Application Insights][application-insights] é um serviço de gerenciamento de desempenho de aplicativo (APM) extensível para desenvolvedores e dá suporte a várias plataformas. Ele monitora o aplicativo, detecta anomalias de aplicativo, como falhas e desempenho ruim, e envia a telemetria ao portal do Azure. O Application Insights também pode ser usado para registro em log, rastreamento distribuído e métricas de aplicativo personalizadas.
- O [Azure Monitor][azure-monitor] fornece logs e [métricas de infraestrutura de nível básico][metrics] para a maioria dos serviços do Azure. Você pode interagir com as métricas de diversas maneiras, incluindo criar gráficos delas no portal do Azure, acessá-las por meio da API REST ou consultá-las usando o PowerShell ou a CLI. O Azure Monitor também oferece seus dados diretamente no [Log Analytics e outros serviços], em que você pode consultá-los e combiná-los com dados de outras fontes no local ou na nuvem.
- O [Log Analytics][log-analytics] ajuda a correlacionar os dados de uso e desempenho coletados pelo Application Insights com dados de configuração e desempenho nos recursos do Azure que dão suporte ao aplicativo. Este cenário usa o [agente do Azure Log Analytics][Azure Log Analytics agent] para enviar por push logs de auditoria do SQL Server para o Log Analytics. Você pode escrever consultas e exibir dados na folha do Log Analytics no portal do Azure.

## <a name="considerations"></a>Considerações

Uma prática recomendada é adicionar o Application Insights ao código durante o desenvolvimento usando [SDKs do Application Insights][Application Insights SDKs] e personalizar por aplicativo. Esses SDKs de software livre estão disponíveis para a maioria das estruturas de aplicativo. Para enriquecer e controlar os dados coletados, incorpore o uso de ambos os SDKs para implantações de produção e teste em seu processo de desenvolvimento. O principal requisito é que o aplicativo tenha uma linha de visão direta ou indireta para o ponto de extremidade de ingestão do Applications Insights hospedado com um endereço voltado para a Internet. Você pode então adicionar telemetria ou enriquecer uma coleção de telemetria existente.

O monitoramento de tempo de execução é outra maneira fácil de começar. A telemetria coletada deve ser controlada por meio de arquivos de configuração. Por exemplo, você pode incluir métodos de tempo de execução que habilitam ferramentas como o [Application Insights Status Monitor][Application Insights Status Monitor] para implantar os SDKs na pasta correta e adicionar as configurações corretas para começar o monitoramento.

Como o Application Insights, o Log Analytics fornece ferramentas para [analisar dados em fontes][analyzing data across sources], criar consultas complexas e [enviar alertas proativos][sending proactive alerts] nas condições especificadas. Você também pode exibir a telemetria no [portal do Azure][the Azure portal]. O Log Analytics agrega valor aos serviços de monitoramento existentes, como o [Azure Monitor][Azure Monitor], e também pode monitorar ambientes locais.

O Application Insights e o Log Analytics usam a [Linguagem de Consulta do Azure Log Analytics][Azure Log Analytics Query Language]. Você também pode usar [consultas de recursos cruzados](https://azure.microsoft.com/blog/query-across-resources) para analisar a telemetria coletada pelo Application Insights e pelo Log Analytics em uma única consulta.

O Azure Monitor, o Application Insights e o Log Analytics enviam [alertas](/azure/monitoring-and-diagnostics/monitoring-overview-alerts). Por exemplo, o Azure Monitor alerta sobre métricas de nível de plataforma, como a utilização de CPU, enquanto o Application Insights alerta sobre métricas de nível de aplicativo, como tempo de resposta do servidor. O Azure Monitor alerta sobre novos eventos no Log de Atividades do Azure, enquanto o Log Analytics pode emitir alertas sobre dados de métricas ou eventos para os serviços configurados para usá-lo. [Alertas unificados no Azure Monitor](/azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts) é uma nova experiência de alertas unificados no Azure que usa uma taxonomia diferente.

### <a name="alternatives"></a>Alternativas

Este artigo descreve opções de monitoramento convenientemente disponíveis com recursos populares, mas você tem muitas opções, incluindo a opção de criar seus próprios mecanismos de registro em log. Uma prática recomendada é adicionar os serviços de monitoramento conforme você compila as camadas em uma solução. Aqui estão algumas possíveis extensões e alternativas:

- Consolide as métricas do Azure Monitor e do Application Insights no Grafana usando o [Azure Monitor Data Source para Grafana][Azure Monitor Data Source For Grafana].
- O [Data Dog][data-dog] tem um conector para o Azure Monitor
- Automatize funções de monitoramento usando a [Automação do Azure][Azure Automation].
- Adicione a comunicação com [soluções ITSM][ITSM solutions].
- Estenda o Log Analytics com uma [solução de gerenciamento][management solution].

### <a name="scalability-and-availability"></a>Escalabilidade e disponibilidade

Esse cenário se concentra em soluções de PaaS para o monitoramento, em grande parte porque elas lidam de forma conveniente com a disponibilidade e a escalabilidade para você e contam com SLAs (contratos de nível de serviço). Por exemplo, os Serviços de Aplicativos fornecem um [SLA][SLA] com garantia de disponibilidade.

O Application Insights tem [limites][app-insights-limits] para o número de solicitações que podem ser processadas por segundo. Se exceder o limite de solicitação, você poderá enfrentar a limitação de mensagens. Para evitar limitação, implemente [filtragem][message-filtering] ou [amostragem][message-sampling] para reduzir a taxa de dados

No entanto, considerações de alta disponibilidade para o aplicativo que você executa são responsabilidade do desenvolvedor. Para saber mais sobre a escala, por exemplo, confira a seção [Considerações sobre escalabilidade](#scalability-considerations) na arquitetura de referência do aplicativo Web básico. Depois que um aplicativo for implantado, você poderá configurar testes para [monitorar sua disponibilidade][monitor its availability] usando o Application Insights.

### <a name="security"></a>Segurança

Os requisitos de conformidade e informações confidenciais afetam a coleta, a retenção e o armazenamento de dados. Saiba mais sobre como o [Application Insights][application-insights] e o [Log Analytics][log-analytics] lidam com a telemetria.

As seguintes considerações de segurança também podem ser aplicáveis:

- Desenvolva um plano para lidar com informações pessoais, se os desenvolvedores têm permissão para coletar seus próprios dados ou enriquecer a telemetria existente.
- Considere a possibilidade de retenção de dados. Por exemplo, o Application Insights retém dados de telemetria por 90 dias. Arquive dados que você deseja acessar por períodos mais longos usando o Microsoft Power BI, a Exportação Contínua ou a API REST. As taxas de armazenamento se aplicam.
- Limite o acesso aos recursos do Azure para controlar o acesso a dados e quem pode exibir a telemetria de um aplicativo específico. Para ajudar a bloquear o acesso ao monitoramento de telemetria, confira [Recursos, funções e controle de acesso no Application Insights][Resources, roles, and access control in Application Insights].
- Considere a possibilidade de controlar o acesso de leitura/gravação no código do aplicativo para impedir que usuários adicionem marcadores de versão ou marca que limitam a ingestão de dados do aplicativo. Com o Application Insights, não há controle sobre itens de dados individuais quando eles são enviados a um recurso. Portanto, se um usuário tem acesso a alguns dados, tem acesso a todos os dados em um recurso individual.
- Adicione mecanismos de [governança](/azure/security/governance-in-azure) para impor a política ou controles de custo sobre recursos do Azure, se necessário. Por exemplo, use o Log Analytics para monitoramento de segurança, como políticas e controle de acesso baseado em função, ou use o [Azure Policy](/azure/azure-policy/azure-policy-introduction) para criar, atribuir e gerenciar definições de política.
- Para monitorar problemas de segurança potenciais e obter uma visão central do estado de segurança de seus recursos do Azure, considere o uso da [Central de Segurança do Azure](/azure/security-center/security-center-intro).

## <a name="pricing"></a>Preços

Os encargos de monitoramento podem se acumular rapidamente. Portanto, considere o preço com antecedência, entenda o que você está monitorando e verifique as taxas associadas a cada serviço. O Azure Monitor fornece [métricas básicas][basic metrics] sem custo, enquanto o monitoramento dos custos do [Application Insights][application-insights-pricing] e do [Log Analytics][log-analytics] se baseia na quantidade de dados ingeridos e no número de testes que você executa.

Para ajudá-lo a começar, use a [calculadora de preços][pricing] para estimar os custos. Para ver como o preço seria alterado para seu caso de uso específico, altere as várias opções para corresponder à sua implantação esperada.

A telemetria do Application Insights é enviada ao portal do Azure durante a depuração e depois que você publica o aplicativo. Para fins de teste e para evitar cobranças, um volume limitado de telemetria é instrumentado. Para adicionar indicadores, você pode aumentar o limite de telemetria. Para obter um controle mais granular, confira [Amostragem no Application Insights][Sampling in Application Insights].

Após a implantação, você poderá assistir a um [Live Metrics Stream][Live Metrics Stream] dos indicadores de desempenho. Esses dados não são armazenados &mdash; (você está exibindo métricas em tempo real) &mdash;, mas a telemetria pode ser coletada e analisada posteriormente. Não há custo para dados do Live Stream.

O Log Analytics é cobrado por GB (gigabyte) de dados ingeridos no serviço. Os primeiros 5 GB de dados ingeridos para o serviço Azure Log Analytics a cada mês são oferecidos gratuitamente, e os dados são retidos sem custo pelos primeiros 31 dias no espaço de trabalho do Log Analytics. 

## <a name="next-steps"></a>Próximas etapas

Confira estes recursos projetados para ajudar você a se familiarizar com sua própria solução de monitoramento:

[Arquitetura de referência de aplicativo Web básico][Basic web application reference architecture]

[Começar a monitorar o aplicativo Web ASP.NET][Start monitoring your ASP.NET Web Application]

[Coletar dados sobre as Máquinas Virtuais do Azure][Collect data about Azure Virtual Machines]

## <a name="related-resources"></a>Recursos relacionados

[Monitoramento dos aplicativos e recursos do Azure][Monitoring Azure applications and resources]

[Localizar e diagnosticar exceções de tempo de execução com o Azure Application Insights][Find and diagnose run-time exceptions with Azure Application Insights]

<!-- links -->
[architecture]: ./media/architecture-app-monitoring.png
[availability-tests]: /azure/application-insights/app-insights-monitor-web-app-availability
[application-insights]: /azure/application-insights/app-insights-overview
[azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor
[metrics]: /azure/monitoring-and-diagnostics/monitoring-supported-metrics
[Log Analytics e outros serviços]: /azure/log-analytics/log-analytics-azure-storage
[log-analytics]: /azure/log-analytics/log-analytics-overview
[Azure Log Analytics agent]: https://blogs.msdn.microsoft.com/sqlsecurity/2017/12/28/azure-log-analytics-oms-agent-now-collects-sql-server-audit-logs/
[application-insights-pricing]: https://azure.microsoft.com/pricing/details/application-insights/
[Application Insights SDKs]: /azure/application-insights/app-insights-asp-net
[Application Insights Status Monitor]: https://azure.microsoft.com/updates/application-insights-status-monitor-and-sdk-updated/
[analyzing data across sources]: /azure/log-analytics/log-analytics-dashboards
[sending proactive alerts]: /azure/log-analytics/log-analytics-alerts
[the Azure portal]: /azure/log-analytics/log-analytics-tutorial-dashboards
[Azure Log Analytics Query Language]: https://docs.loganalytics.io/docs/Learn
[cross-resource queries]: https://azure.microsoft.com/blog/query-across-resources/
[alerts]: /azure/monitoring-and-diagnostics/monitoring-overview-alerts
[Alerts (Preview)]: /azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts
[Azure Monitor Data Source For Grafana]: https://grafana.com/plugins/grafana-azure-monitor-datasource
[Azure Automation]: /azure/automation/automation-intro
[ITSM solutions]: https://azure.microsoft.com/blog/itsm-connector-for-azure-is-now-generally-available/
[management solution]: /azure/monitoring/monitoring-solutions
[SLA]: https://azure.microsoft.com/support/legal/sla/app-service/v1_4/
[monitor its availability]: /azure/application-insights/app-insights-monitor-web-app-availability
[Resources, roles, and access control in Application Insights]: /azure/application-insights/app-insights-resources-roles-access-control
[basic metrics]: /azure/monitoring-and-diagnostics/monitoring-supported-metrics
[pricing]: https://azure.microsoft.com/pricing/calculator/#log-analyticsc126d8c1-ec9c-4e5b-9b51-4db95d06a9b1
[Sampling in Application Insights]: /azure/application-insights/app-insights-sampling
[Live Metrics Stream]: /azure/application-insights/app-insights-live-stream
[Basic web application reference architecture]: /azure/architecture/reference-architectures/app-service-web-app/basic-web-app#scalability-considerations
[Start monitoring your ASP.NET Web Application]: /azure/application-insights/quick-monitor-portal
[Collect data about Azure Virtual Machines]: /azure/log-analytics/log-analytics-quick-collect-azurevm
[Monitoring Azure applications and resources]: /azure/monitoring-and-diagnostics/monitoring-overview
[Find and diagnose run-time exceptions with Azure Application Insights]: /azure/application-insights/app-insights-tutorial-runtime-exceptions
[data-dog]: https://www.datadoghq.com/blog/azure-monitoring-enhancements/
[app-insights-limits]: /azure/azure-subscription-service-limits#application-insights-limits
[message-filtering]: /azure/application-insights/app-insights-api-filtering-sampling
[message-sampling]: /azure/application-insights/app-insights-sampling
