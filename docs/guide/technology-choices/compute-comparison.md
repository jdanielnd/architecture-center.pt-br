---
title: Critérios para escolher um serviço de computação do Azure
titleSuffix: Azure Application Architecture Guide
description: Compare os serviços de computação do Azure entre diversos eixos.
author: MikeWasson
ms.date: 08/08/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 2b6b9b941bf7a3c0136b71ecb65bfe4b4a59e07b
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245597"
---
# <a name="criteria-for-choosing-an-azure-compute-service"></a>Critérios para escolher um serviço de computação do Azure

O termo *computação* refere-se ao modelo de hospedagem para os recursos de computação em que seus aplicativos são executados. As tabelas a seguir comparam os serviços de computação do Azure entre diversos eixos. Consulte estas tabelas ao selecionar uma opção de computação para seu aplicativo.

## <a name="hosting-model"></a>Modelo de hospedagem

<!-- markdownlint-disable MD033 -->

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Kubernetes do Azure | Instâncias de Contêiner | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Composição de aplicativos | Independente | Aplicativos, contêineres | Serviços, executáveis do convidado, contêineres | Funções | Contêineres | Contêineres | Trabalhos agendados  |
| Densidade | Independente | Vários aplicativos por instância via planos do serviço de aplicativo | Vários serviços por VM | Sem servidor <a href="#note1"><sup>1</sup></a> | Vários contêineres por nó |Sem instâncias dedicadas | Vários aplicativos por VM |
| Número mínimo de nós | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | Sem servidor <a href="#note1"><sup>1</sup></a> | 3 <a href="#note3"><sup>3</sup></a> | Sem nós dedicados | 1 <a href="#note4"><sup>4</sup></a> |
| Gerenciamento de estado | Com Estado ou Sem Estado | Sem estado | Com estado ou sem estado | Sem estado | Com Estado ou Sem Estado | Sem estado | Sem estado |
| Hospedagem na Web | Independente | Interno | Independente | Não aplicável | Independente | Independente | Não  |
| Pode ser implantado para VNet dedicada? | Com suporte | Com suporte<a href="#note5"><sup>5</sup></a> | Com suporte | Com suporte <a href="#note5"><sup>5</sup></a> | [Com suporte](/azure/aks/networking-overview) | Sem suporte | Com suporte |
| Conectividade híbrida | Com suporte | Com suporte <a href="#note6"><sup>6</sup></a>  | Com suporte | Com suporte <a href="#node7"><sup>7</sup></a> | Com suporte | Sem suporte | Com suporte |

Observações

1. <span id="note1">Se estiver usando o plano de Consumo. Se estiver usando o plano de Serviço de Aplicativo, as funções serão executadas nas VMs alocadas ao seu plano de Serviço de Aplicativo. Consulte [Escolher o plano de serviço correto para o Azure Functions][function-plans].</span>
2. <span id="note2">SLA superior com duas ou mais instâncias.</span>
3. <span id="note3">Recomendado para ambientes de produção.</span>
4. <span id="note4">Pode reduzir verticalmente até zero após o trabalho ser concluído.</span>
5. <span id="note5">Requer ASE (Ambiente de Serviço de Aplicativo).</span>
6. <span id="note6">Use [Conexões Híbridas do Serviço de Aplicativo do Azure][app-service-hybrid].</span>
7. <span id="note7">Requer o Plano do Serviço de Aplicativo.</span>

## <a name="devops"></a>DevOps

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Kubernetes do Azure | Instâncias de Contêiner | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Depuração local | Independente | IIS Express, outros <a href="#note1b"><sup>1</sup></a> | Cluster de nó local | Visual Studio ou CLI do Azure Functions | Minikube, outros | Tempo de execução do contêiner local | Sem suporte |
| Modelo de programação | Independente | Aplicativos da Web e da API, Trabalhos da Web para tarefas em segundo plano | Executável convidado, modelo de Serviço, o modelo de Ator, Contêineres | Funções com gatilhos | Independente | Independente | Aplicativo de linha de comando |
| Atualização do aplicativo | Não há suporte interno | Slots de implantação | Atualização sem interrupção (por serviço) | Slots de implantação | Atualização sem interrupção | Não aplicável |

Observações

1. <span id="note1b">As opções incluem o IIS Express para ASP.NET ou node.js (iisnode); servidor Web PHP; Kit de Ferramentas do Azure para IntelliJ, Kit de Ferramentas do Azure para Eclipse. O Serviço de Aplicativo também dá suporte para depuração remota de aplicativo Web implantado.</span>
2. <span id="note2b">Consulte [Provedores, regiões, versões de API e esquemas do Resource Manager][resource-manager-supported-services].</span>

## <a name="scalability"></a>Escalabilidade

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Kubernetes do Azure | Instâncias de Contêiner | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Dimensionamento automático | Conjuntos de dimensionamento da VM | Serviço interno | Conjuntos de Dimensionamento da VM | Serviço interno | Sem suporte | Sem suporte | N/D |
| Balanceador de carga | Azure Load Balancer | Integrado | Azure Load Balancer | Integrado | Integrado |  Não há suporte interno | Azure Load Balancer |
| Limite de escala<a href="#note1c"><sup>1</sup></a> | Imagem da plataforma: 1.000 nós por VMSS; imagem personalizada: 100 nós por VMSS | 20 instâncias, 100 com Ambiente do Serviço de Aplicativo | 100 nós por VMSS | 200 instâncias por aplicativo de Função | 100 nós por cluster (limite padrão) |20 grupos de contêiner por assinatura (limite padrão). | Limite de 20 núcleos (limite padrão). |

Observações

1. <span id="note1c">Consulte [Assinatura do Azure e limites de serviço, cotas e restrições](/azure/azure-subscription-service-limits)</span>.

## <a name="availability"></a>Disponibilidade

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Kubernetes do Azure | Instâncias de Contêiner | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Contrato de Nível de Serviço | [SLA para Máquinas Virtuais][sla-vm] | [SLA para Serviço de Aplicativo][sla-app-service] | [SLA para Service Fabric][sla-sf] | [SLA para Functions][sla-functions] | [SLA para AKS][sla-acs] | [SLA para as Instâncias de Contêiner](https://azure.microsoft.com/support/legal/sla/container-instances/) | [SLA para o Lote do Azure][sla-batch] |
| Failover de várias regiões | Gerenciador de tráfego | Gerenciador de tráfego | Gerenciador de tráfego, Cluster de Várias Regiões | Sem suporte | Gerenciador de tráfego | Sem suporte | Sem suporte |

## <a name="other"></a>Outros

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Kubernetes do Azure | Instâncias de Contêiner | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | Configurado na VM | Com suporte | Com suporte  | Com suporte | [Controlador de entrada](/azure/aks/ingress) | Usar contêiner [sidecar](../../patterns/sidecar.md) | Com suporte |
| Custo | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [Preços do Serviço de Aplicativo][cost-app-service] | [Preços do Service Fabric][cost-service-fabric] | [Preços do Azure Functions][cost-functions] | [Preços do AKS][cost-acs] | [Preço das Instâncias de Contêiner](https://azure.microsoft.com/pricing/details/container-instances/) | [Preço do Lote do Azure][cost-batch]
| Estilos de arquitetura adequados | [N camadas][n-tier], [Big compute] [ big-compute] (HPC) | [Trabalhado de Fila da Web][w-q-w], [N Camadas][n-tier] | [Microsserviços][microservices], [arquitetura orientada a eventos][event-driven] | [Microsserviços][microservices], [arquitetura orientada a eventos][event-driven] | [Microsserviços][microservices], [arquitetura orientada a eventos][event-driven] | [Microsserviços][microservices], automação de tarefas, trabalhos em lotes  | [Big compute][big-compute] (HPC) |

<!-- markdownlint-enable MD033 -->

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/kubernetes-service/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/kubernetes-service
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

[app-service-hybrid]: /azure/app-service/app-service-hybrid-connections
