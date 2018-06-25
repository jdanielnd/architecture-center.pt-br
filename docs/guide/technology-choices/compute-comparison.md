---
title: Critérios para escolher um serviço de computação do Azure
description: Compare os serviços de computação do Azure entre diversos eixos
author: MikeWasson
layout: LandingPage
ms.date: 06/13/2018
ms.openlocfilehash: 29c21c44bdf3a3bfa29f17015565eecf5f86163b
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206499"
---
# <a name="criteria-for-choosing-an-azure-compute-service"></a>Critérios para escolher um serviço de computação do Azure

O termo *computação* refere-se ao modelo de hospedagem para os recursos de computação em que seus aplicativos são executados. As tabelas a seguir comparam os serviços de computação do Azure entre diversos eixos. Consulte estas tabelas ao selecionar uma opção de computação para seu aplicativo.

## <a name="hosting-model"></a>Modelo de hospedagem

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Contêiner do Azure | Instâncias de Contêiner | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Composição de aplicativos | Independente | Aplicativos, contêineres | Serviços, executáveis do convidado, contêineres | Funções | Contêineres | Contêineres | Trabalhos agendados  |
| Densidade | Independente | Vários aplicativos por instância via planos do aplicativo | Vários serviços por VM | Nenhuma instância dedicada <a href="#note1"><sup>1</sup></a> | Vários contêineres por VM |Sem instâncias dedicadas | Vários aplicativos por VM |
| Número mínimo de nós | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | Nenhum nó dedicado <a href="#note1"><sup>1</sup></a> | 3 | Sem nós dedicados | 1 <a href="#note4"><sup>4</sup></a> |
| Gerenciamento de estado | Com Estado ou Sem Estado | Sem estado | Com estado ou sem estado | Sem estado | Com Estado ou Sem Estado | Sem estado | Sem estado |
| Hospedagem na Web | Independente | Interno | Independente | Não aplicável | Independente | Independente | Não  |
| SO | Windows, Linux | Windows, Linux  | Windows, Linux | Não aplicável | Windows (visualização),  Linux | Windows, Linux | Windows, Linux |
| Pode ser implantado para VNet dedicada? | Com suporte | Com suporte <a href="#note5"><sup>5</sup></a> | Com suporte | Sem suporte | Com suporte | Sem suporte | Com suporte |
| Conectividade híbrida | Com suporte | Com suporte <a href="#note1"><sup>6</sup></a>  | Com suporte | Sem suporte | Com suporte | Sem suporte | Com suporte |

Observações

1. <span id="note1">Se estiver usando o plano de Consumo. Se estiver usando o plano de Serviço de Aplicativo, as funções serão executadas nas VMs alocadas ao seu plano de Serviço de Aplicativo. Consulte [Escolher o plano de serviço correto para o Azure Functions][function-plans].</span>
2. <span id="note2">SLA superior com duas ou mais instâncias.</span>
3. <span id="note3">Para ambientes de produção.</span>
4. <span id="note4">Pode reduzir verticalmente até zero após o trabalho ser concluído.</span>
5. <span id="note5">Requer ASE (Ambiente de Serviço de Aplicativo).</span>
6. <span id="note7">Requer Conexões Híbridas do BizTalk ou ASE</span>

## <a name="devops"></a>DevOps

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Contêiner do Azure | Instâncias de Contêiner | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Depuração local | Independente | IIS Express, outros <a href="#note1b"><sup>1</sup></a> | Cluster de nó local | CLI do Azure Functions | Tempo de execução do contêiner local | Tempo de execução do contêiner local | Sem suporte |
| Modelo de programação | Independente | Aplicativo Web, WebJobs para tarefas em segundo plano | Executável convidado, modelo de Serviço, o modelo de Ator, Contêineres | Funções com gatilhos | Independente | Independente | Aplicativo de linha de comando |
| Atualização do aplicativo | Não há suporte interno | Slots de implantação | Atualização sem interrupção (por serviço) | Não há suporte interno | Depende do orquestrador. A maioria dá suporte a atualizações sem interrupção | Atualizar imagem de contêiner | Não aplicável |

Observações

1. <span id="note1b">As opções incluem o IIS Express para ASP.NET ou node.js (iisnode); servidor Web PHP; Kit de Ferramentas do Azure para IntelliJ, Kit de Ferramentas do Azure para Eclipse. O Serviço de Aplicativo também dá suporte para depuração remota de aplicativo Web implantado.</span>
2. <span id="note2b">Consulte [Provedores, regiões, versões de API e esquemas do Resource Manager][resource-manager-supported-services].</span> 


## <a name="scalability"></a>Escalabilidade

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Contêiner do Azure | Instâncias de Contêiner | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Dimensionamento automático | Conjuntos de dimensionamento da VM | Serviço interno | Conjuntos de Dimensionamento da VM | Serviço interno | Sem suporte | Sem suporte | N/D |
| Balanceador de carga | Azure Load Balancer | Integrado | Azure Load Balancer | Integrado | Azure Load Balancer |  Não há suporte interno | Azure Load Balancer |
| Limite de escala | Imagem da plataforma: 1.000 nós por VMSS; imagem personalizada: 100 nós por VMSS | 20 instâncias, 50 com o Ambiente de Serviço de Aplicativo | 100 nós por VMSS | Infinito <a href="#note1c"><sup>1</sup></a> | 100 <a href="#note2c"><sup>2</sup></a> |20 grupos de contêiner por assinatura por padrão. Contate o atendimento ao cliente para aumento. <a href="#note3c"><sup>3</sup></a> | Limite de 20 núcleos por padrão. Contate o atendimento ao cliente para aumento. |

Observações

1. <span id="note1c">Se estiver usando o plano de Consumo. Se o plano de Serviço de Aplicativo, os limites de escala do Serviço de Aplicativo se aplicarão. Consulte [Escolher o plano de serviço correto para o Azure Functions][function-plans].</span>
2. <span id="note2c">Consulte [Expandir nós de agente em um cluster do serviço de contêiner][scale-acs]</span>.
3. <span id="note3c">Confira [Cotas e disponibilidade de região para Instâncias de Contêiner do Azure](/azure/container-instances/container-instances-quotas).</span>


## <a name="availability"></a>Disponibilidade

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Contêiner do Azure | Instâncias de Contêiner | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Contrato de Nível de Serviço | [SLA para Máquinas Virtuais][sla-vm] | [SLA para Serviço de Aplicativo][sla-app-service] | [SLA para Service Fabric][sla-sf] | [SLA para Functions][sla-functions] | [SLA para o Serviço de Contêiner do Azure][sla-acs] | [SLA para as Instâncias de Contêiner](https://azure.microsoft.com/support/legal/sla/container-instances/) | [SLA para o Lote do Azure][sla-batch] |
| Failover de várias regiões | Gerenciador de tráfego | Gerenciador de tráfego | Gerenciador de tráfego, Cluster de Várias Regiões | Sem suporte  | Gerenciador de tráfego | Sem suporte | Sem suporte |

## <a name="other"></a>Outros

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Contêiner do Azure | Instâncias de Contêiner | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | Configurado na VM | Com suporte | Com suporte  | Com suporte | Configurado na VM | Suporte com o contêiner sidecar | Com suporte |
| Custo | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [Preços do Serviço de Aplicativo][cost-app-service] | [Preços do Service Fabric][cost-service-fabric] | [Preços do Azure Functions][cost-functions] | [Preços do Serviço de Contêiner do Azure][cost-acs] | [Preço das Instâncias de Contêiner](https://azure.microsoft.com/pricing/details/container-instances/) | [Preço do Lote do Azure][cost-batch]
| Estilos de arquitetura adequados | [N camadas][n-tier], [Big compute] [ big-compute] (HPC) | [Trabalhador de fila da Web][w-q-w] | [Microsserviços][microservices], [arquitetura orientada a eventos][event-driven] | [Microsserviços][microservices], [arquitetura orientada a eventos][event-driven] | [Microsserviços][microservices], [arquitetura orientada a eventos][event-driven] | [Microsserviços][microservices], automação de tarefas, trabalhos em lotes  | [Big compute][big-compute] (HPC) |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services
[scale-acs]: /azure/container-service/kubernetes/container-service-scale#scaling-considerations

[n-tier]: ../architecture-styles/n-tier.md
[w-q-w]: ../architecture-styles/web-queue-worker.md
[microservices]: ../architecture-styles/microservices.md
[event-driven]: ../architecture-styles/event-driven.md
[big-date]: ../architecture-styles/big-data.md
[big-compute]: ../architecture-styles/big-compute.md

