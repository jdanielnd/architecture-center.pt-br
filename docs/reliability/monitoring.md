---
title: Monitorando a integridade do aplicativo para a confiabilidade do Azure
description: Como usar o monitoramento para melhorar a confiabilidade do aplicativo no Azure
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: 46f9f3fb623a2facf4b9f9c6ec57376ae59e41dc
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646483"
---
# <a name="monitoring-application-health-for-reliability-in-azure"></a>Monitorando a integridade do aplicativo para a confiabilidade do Azure

O monitoramento e o diagnóstico são cruciais para garantir a resiliência. Se algo falhar, você precisa saber *que* ele falha, *quando* ele falha &mdash; e *por que*.

*Monitorando* não é igual a *detecção de falha*. Por exemplo, seu aplicativo pode detectar transitório erro e tente novamente, evitando o tempo de inatividade. Mas ele também deve registrar a operação de repetição para que você possa monitorar a taxa de erros para obter uma visão geral da integridade do aplicativo.

Imagine o processo de monitoramento e diagnóstico como um pipeline com a quatro estágios distintos: Instrumentação, coleta e armazenamento, análise e diagnóstico, visualização e alertas.

## <a name="instrumentation"></a>Instrumentação

Não é prático monitorar seu aplicativo diretamente, portanto, a instrumentação é a chave. Um sistema distribuído em larga escala pode ser executada em dezenas de máquinas virtuais (VMs), que são adicionadas e removidas ao longo do tempo. Da mesma forma, um aplicativo de nuvem pode usar um número de armazenamentos de dados e uma ação de usuário único pode abranger vários subsistemas.

Forneça instrumentação avançada:

- Para falhas que provavelmente mas ainda não tenham ocorrido, fornecer dados suficientes para determinar a causa, atenue a situação e certifique-se de que o sistema permaneça disponível.
- Para falhas que já tenham ocorrido, o aplicativo deve retornar uma mensagem de erro apropriado para o usuário, mas deve tentar continuar em execução, embora com funcionalidade reduzida.

Sistemas de monitoramento devem capturar detalhes abrangentes para que os aplicativos podem ser restaurados com eficiência e, se necessário, designers e desenvolvedores podem modificar o sistema para evitar a situação ocorra novamente.

Os dados brutos para monitoramento podem vir de uma variedade de fontes, incluindo:

- Logs de aplicativo, como aqueles gerados pela [Azure Application Insights](/azure/azure-monitor/app/app-insights-overview) service.
- As métricas de desempenho do sistema operacional são coletados pelo [do Azure de agentes de monitoramento](/azure/azure-monitor/platform/agents-overview).
- [Recursos do Azure](/azure/azure-monitor/platform/metrics-supported), incluindo métricas coletadas pelo Azure Monitor.
- [A integridade do serviço do Azure](/azure/service-health/service-health-overview), que oferece um painel para ajudá-lo a controlar os eventos ativos.
- [Os logs do Azure AD](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics) incorporados à plataforma do Azure.

Os serviços do Azure mais tem métricas e diagnósticos que você pode configurar para analisar e determinar a causa dos problemas. Para obter mais informações, consulte [dados de monitoramento coletados pelo Azure Monitor](/azure/azure-monitor/platform/data-collection).

## <a name="collection-and-storage"></a>Coleta e armazenamento

Dados brutos de instrumentação podem ser mantidos em diversos locais e formatos, incluindo:

- Logs de rastreamento do aplicativo
- Logs IIS
- contadores de desempenho

Essas fontes diferentes são coletadas, consolidadas e colocadas em repositórios de dados confiável no Azure, como o Application Insights, as métricas do Azure Monitor, a integridade do serviço, contas de armazenamento e Azure Log Analytics.

## <a name="analysis-and-diagnosis"></a>Análise e diagnóstico

Analise dados consolidados nesses armazenamentos de dados para solucionar problemas e obter uma visão geral da integridade do aplicativo. Em geral, você pode [pesquise e analise](/azure/azure-monitor/log-query/log-query-overview) os dados no Application Insights e Log Analytics usando consultas Kusto ou grafos de exibição pré-configurada usando [soluções de gerenciamento](/azure/azure-monitor/insights/solutions-inventory). Ou use o Azure Advisor para exibir as recomendações com foco na [resiliência](/azure/advisor/advisor-high-availability-recommendations) e [desempenho](/azure/advisor/advisor-performance-recommendations).

## <a name="visualization-and-alerts"></a>Visualização e alertas

Apresentar dados de telemetria em um formato que torna mais fácil para um operador a ser observado problemas ou tendências rapidamente, como um alerta de email ou painel.

Obter uma exibição de pilha completa do estado do aplicativo por meio [painéis do Azure](/azure/azure-portal/azure-portal-dashboards) para criar uma exibição consolidada de gráficos do Application Insights, análise de Log, métricas do Azure Monitor e a integridade do serviço de monitoramento. E usar [alertas do Azure Monitor](/azure/azure-monitor/platform/alerts-overview) criar notificações de integridade do serviço, o resource health, métricas do Azure Monitor, registra em log no Log Analytics e Application Insights.

Para obter mais informações sobre monitoramento e diagnóstico, consulte [monitoramento e diagnóstico](../best-practices/monitoring.md).

## <a name="health-probes-and-check-functions"></a>Funções de verificação e investigações de integridade

A integridade e o desempenho de um aplicativo podem diminuir ao longo do tempo e degradação pode não ser notada até que o aplicativo falhe.

Implemente investigações ou verifique funções e executá-los regularmente de fora do aplicativo. Essas verificações podem ser tão simples quanto medir o tempo de resposta para o aplicativo como um todo, as partes individuais do aplicativo, serviços específicos que o aplicativo usa ou componentes separados.

Funções de verificação podem executar processos para garantir que eles produzem resultados válidos, medem a latência e verificar a disponibilidade e extraem informações do sistema.

## <a name="long-running-workflow-failures"></a>Falhas de fluxo de trabalho de longa execução

Fluxos de trabalho de longa execução geralmente incluem várias etapas, cada um deles deve ser independente.

Acompanhe o progresso dos processos de longa execução para minimizar a probabilidade de que todo o fluxo de trabalho precisará ser revertida ou que várias transações de compensação precisará ser executado.

>[!TIP]
> Monitore e gerencie o progresso de fluxos de trabalho de longa duração implementando um padrão como [Supervisor de agente do Agendador](../patterns/scheduler-agent-supervisor.md).

## <a name="application-logs"></a>Logs de aplicativo

Os logs de aplicativo são uma fonte importante de dados de diagnóstico. Para obter informações quando precisar da maioria, siga as práticas recomendadas para o log de aplicativo.

### <a name="log-data-in-the-production-environment"></a>Dados de log no ambiente de produção

Capture dados de telemetria robustas enquanto o aplicativo está em execução no ambiente de produção, para que você tenha informações suficientes para diagnosticar a causa dos problemas no estado de produção.

### <a name="log-events-at-service-boundaries"></a>Eventos de log em limites de serviço

Inclua uma ID de correlação que flua nos limites dos serviços. Se uma transação fluir em vários serviços e um deles falhar, a correlação ID ajuda a acompanhar as solicitações em seu aplicativo e indica por que a transação falhou.

### <a name="use-semantic-structured-logging"></a>Usar o registro em log semântico (estruturado)

Com os logs estruturados, é mais fácil automatizar o consumo e a análise dos dados do log, que é especialmente importantes em escala de nuvem. Em geral, é recomendável armazenar dados de diagnóstico e métricas de recursos do Azure em um espaço de trabalho do Log Analytics em vez de em uma conta de armazenamento. Dessa forma, você pode usar consultas Kusto para obter os dados desejados rapidamente e em um formato estruturado. Você também pode usar APIs do Azure Monitor e APIs de análise de Log do Azure.

### <a name="use-asynchronous-logging"></a>Use logs assíncronos

Operações de registro em log síncrono, às vezes, bloqueiam o código do aplicativo, fazendo com que as solicitações para fazer backup de que os logs são gravados. Use logs assíncronos para preservar a disponibilidade durante o registro em log do aplicativo.

### <a name="separate-application-logging-from-auditing"></a>Log de aplicativo separada da auditoria

Registros de auditoria geralmente são mantidos para requisitos regulamentares ou de conformidade e devem ser concluídos. Para evitar transações descartadas, manter os logs de auditoria separadamente dos logs de diagnóstico.

## <a name="remote-call-statistics"></a>Estatísticas de chamada remota

Acompanhamento e relatório remoto chamar estatísticas em tempo real e fornecem uma maneira fácil de examinar essas informações, portanto, a equipe de operações tem uma exibição instantânea da integridade do seu aplicativo. Resuma métricas de chamada remota, como latência, taxa de transferência e erros nos percentuais de 99 e 95.

## <a name="transient-exceptions-and-retries"></a>Exceções transitórias e novas tentativas

Controle o número de exceções transitórias e novas tentativas ao longo do tempo para descobrir problemas ou falhas na lógica de repetição do seu aplicativo. Uma tendência de aumento de exceções ao longo do tempo pode indicar que o serviço está apresentando um problema e pode falhar. Para saber mais, confira [Diretriz específica do serviço de repetição](../best-practices/retry-service-specific.md).

## <a name="early-warning-system"></a>Sistema de avisos antecipados

Monitore seu aplicativo em busca de sinais de aviso que podem exigir intervenção proativa. Ferramentas que avaliam a integridade geral do aplicativo e suas dependências ajudarão-lo a reconhecer rapidamente quando um sistema ou seus componentes se tornam indisponíveis repentinamente. Usá-los para implementar um sistema de avisos antecipados.

1. Identifique os indicadores chave de desempenho de integridade do seu aplicativo, como exceções transitórias e latência de chamada remota.
1. Definir limites em níveis que identifiquem problemas antes que eles se tornem críticos e exigem uma resposta de recuperação.
1. Envie um alerta para operações quando o valor limite é atingido.

Considere [Microsoft System Center 2016](https://www.microsoft.com/cloud-platform/system-center) ou ferramentas de terceiros para fornecer recursos de monitoramento. A maioria das soluções de monitoramento monitora os principais contadores de desempenho e a disponibilidade do serviço. [O Azure resource health](/azure/service-health/resource-health-checks-resource-types) fornece alguns integridade interna verificações de status, que podem ajudar a diagnosticar a limitação de serviços do Azure.

## <a name="subscription-and-service-limitations"></a>Limitações de serviço e assinatura

As assinaturas do Azure têm limites em determinados tipos de recurso, como o número de grupos de recursos, núcleos e contas de armazenamento. Para garantir que seu aplicativo não é executado em relação a limites de assinatura do Azure, crie alertas que fazem votação para serviços perto de seus limites e cotas.

Os seguintes limites de assinatura com alertas de endereço.

### <a name="individual-services"></a>Serviços individuais

Serviços do Azure individuais têm limites de consumo de armazenamento, taxa de transferência, número de conexões, solicitações por segundo e outras métricas. O aplicativo falhará se ele tentar usar recursos além desses limites, resultando em tempo de inatividade possíveis e limitação do serviço.

Dependendo do serviço específico e seus requisitos de aplicativo, você geralmente pode permanecer abaixo desses limites, escalar verticalmente (escolhendo outro tipo de preço, por exemplo) ou expandindo (adicionando novas instâncias).

### <a name="azure-storage-scalability-and-performance-targets"></a>Metas de desempenho e escalabilidade do armazenamento do Azure

Azure permite que um número máximo de contas de armazenamento por assinatura. Se seu aplicativo exigir mais contas de armazenamento que estão atualmente disponíveis na sua assinatura, crie uma nova assinatura com contas de armazenamento adicionais. Para saber mais, confira [Assinatura e limites de serviço, cotas e restrições do Azure](/azure/azure-subscription-service-limits/#storage-limits).

Se você exceder as metas de desempenho e escalabilidade do armazenamento do Azure, seu aplicativo sofrerá uma limitação de armazenamento. Para saber mais, confira [Metas de desempenho e escalabilidade do Armazenamento do Azure](/azure/storage/storage-scalability-targets/).

### <a name="scalability-targets-for-virtual-machine-disks"></a>Metas de escalabilidade para discos de máquina virtual

Uma infraestrutura como serviço (IaaS) VM do Azure dá suporte à anexação de um número de discos de dados, dependendo de vários fatores, incluindo o tamanho da VM e o tipo de conta de armazenamento. Se o aplicativo exceder as metas de escalabilidade para discos de máquina virtual, provisione contas de armazenamento adicionais e crie os discos da máquina virtual nelas. Para saber mais, confira [Metas de desempenho e escalabilidade do Armazenamento do Azure](/azure/storage/storage-scalability-targets/#scalability-targets-for-virtual-machine-disks).

### <a name="vm-size"></a>Tamanho da VM

Se a CPU, memória, disco e e/s de suas VMs real aproximam dos limites de tamanho da VM, seu aplicativo pode apresentar problemas de capacidade. Para corrigir os problemas, aumente o tamanho da VM. Tamanhos de VM são descritos em [tamanhos para máquinas virtuais no Azure](/azure/virtual-machines/virtual-machines-windows-sizes/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Se sua carga de trabalho varia ao longo do tempo, considere usar o dimensionamento de máquinas virtuais define como para dimensionar automaticamente o número de instâncias de VM. Caso contrário, você precisará aumentar ou diminuir o número de VMs manualmente. Para obter mais informações, consulte o [visão geral dos conjuntos de dimensionamento de máquinas virtuais](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/).

### <a name="azure-sql-database"></a>Banco de Dados SQL do Azure

Se sua camada de banco de dados SQL não é adequada para lidar com requisitos de unidade de transação de banco de dados (DTU) do seu aplicativo, o uso de dados será limitado. Para obter mais informações sobre como selecionar o plano de serviço correto, consulte [modelos de compra do Azure SQL Database](/azure/sql-database/sql-database-service-tiers/).

## <a name="third-party-services"></a>Serviços de terceiros

Se seu aplicativo tiver dependências de serviços de terceiros, identifique como esses serviços podem falhar e o que terá falhas em vigor no seu aplicativo.

Um serviço de terceiros pode não incluir monitoramento e diagnóstico. Faça chamadas para esses serviços e correlacioná-los com a integridade e o log de diagnóstico usando um identificador exclusivo do seu aplicativo. Para obter mais informações sobre práticas comprovadas de monitoramento e diagnóstico, consulte [diretrizes de monitoramento e diagnóstico](../best-practices/monitoring.md).

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Plano de recuperação de desastres](./disaster-recovery.md)
