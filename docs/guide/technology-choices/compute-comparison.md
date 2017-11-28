---
title: "Critérios para escolher uma opção de computação do Azure"
description: "Compare os serviços de computação do Azure entre diversos eixos."
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 640793b56c1713f63456bab75ab4b9289d22a53c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="criteria-for-choosing-an-azure-compute-option"></a>Critérios para escolher uma opção de computação do Azure

O termo *computação* refere-se ao modelo de hospedagem para os recursos de computação em que seus aplicativos são executados. As tabelas a seguir comparam os serviços de computação do Azure entre diversos eixos. Consulte estas tabelas ao selecionar uma opção de computação para seu aplicativo.

## <a name="hosting-model"></a>Modelo de hospedagem

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Contêiner do Azure | Serviços de Nuvem | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Composição de aplicativos | Independente | Aplicativos | Serviços, executáveis convidados | Funções | Contêineres | Funções | Trabalhos agendados  |
| Densidade | Independente | Vários aplicativos por instância via planos do aplicativo | Vários serviços por VM | Nenhuma instância dedicada <a href="#note1"><sup>1</sup></a> | Vários contêineres por VM | Uma instância de função por VM | Vários aplicativos por VM |
| Número mínimo de nós | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | Nenhum nó dedicado <a href="#note1"><sup>1</sup></a> | 3 | 2 | 1 <a href="#note4"><sup>4</sup></a> |
| Gerenciamento de estado | Com Estado ou Sem Estado | Sem estado | Com estado ou sem estado | Sem estado | Com Estado ou Sem Estado | Sem estado | Sem estado |
| Hospedagem na Web | Independente | Interno | Auto-hospedagem, IIS em contêineres | Não aplicável | Independente | Interno (IIS) | Não |
| SO | Windows, Linux | Windows, Linux (versão prévia)  | Windows, Linux (versão prévia) | Não aplicável | Windows, Linux | Windows | Windows, Linux |
| Pode ser implantado para VNet dedicada? | Suportado | Com suporte <a href="#note5"><sup>5</sup></a> | Suportado | Sem suporte | Suportado | Com suporte <a href="#note6"><sup>6</sup></a> | Suportado |
| Conectividade híbrida | Suportado | Com suporte <a href="#note1"><sup>7</sup></a>  | Suportado | Sem suporte | Suportado | Com suporte <a href="#note8"><sup>8</sup></a> | Suportado |

Observações

1. <span id="note1">Se estiver usando o plano de Consumo. Se estiver usando o plano de Serviço de Aplicativo, as funções serão executadas nas VMs alocadas ao seu plano de Serviço de Aplicativo. Consulte [Escolher o plano de serviço correto para o Azure Functions][function-plans].</a>
2. <span id="note2">SLA superior com duas ou mais instâncias.</a>
3. <span id="note3">Para ambientes de produção.</a>
4. <span id="note4">Pode reduzir verticalmente até zero após o trabalho ser concluído.</a>
5. <span id="note5">Requer ASE (Ambiente de Serviço de Aplicativo).</a>
6. <span id="note6">Somente VNet clássica.</a>
7. <span id="note7">Requer Conexões Híbridas do BizTalk ou ASE</a>
8. <span id="note8">VNet clássica ou VNet do Resource Manager via emparelhamento VNet</a>

## <a name="devops"></a>DevOps

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Contêiner do Azure | Serviços de Nuvem | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Depuração local | Independente | IIS Express, outros <a href="#note1b"><sup>1</sup></a> | Cluster de nó local | CLI do Azure Functions | Tempo de execução do contêiner local | Emulador local | Sem suporte |
| Modelo de programação | Independente | Aplicativo Web, WebJobs para tarefas em segundo plano | Executável convidado, modelo de Serviço, o modelo de Ator, Contêineres | Funções com gatilhos | Independente | Função Web, função de trabalho | Aplicativo de linha de comando |
| Gerenciador de Recursos | Suportado | Com suporte | Com suporte | Com suporte | Suportado | Limitado <a href="#note2b"><sup>2</sup></a> | Suportado |  
| Atualização do aplicativo | Não há suporte interno | Slots de implantação | Atualização sem interrupção (por serviço) | Não há suporte interno | Depende do orquestrador. A maioria dá suporte a atualizações sem interrupção | Troca VIP ou atualização sem interrupção | Não aplicável |

Observações

1. <span id="note1b">As opções incluem o IIS Express para ASP.NET ou node.js (iisnode); servidor Web PHP; Kit de Ferramentas do Azure para IntelliJ, Kit de Ferramentas do Azure para Eclipse. O Serviço de Aplicativo também dá suporte para depuração remota de aplicativo Web implantado.</a>
2. <span id="note2b">Consulte [Provedores, regiões, versões de API e esquemas do Resource Manager][resource-manager-supported-services]. 


## <a name="scalability"></a>Escalabilidade

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Contêiner do Azure | Serviços de Nuvem | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Dimensionamento automático | Conjuntos de dimensionamento da VM | Serviço interno | Conjuntos de Dimensionamento da VM | Serviço interno | Sem suporte | Serviço interno | N/D |
| Balanceador de carga | Azure Load Balancer | Integrado | Azure Load Balancer | Integrado | Azure Load Balancer | Integrado | Azure Load Balancer |
| Limite de escala | Imagem da plataforma: 1.000 nós por VMSS; imagem personalizada: 100 nós por VMSS | 20 instâncias, 50 com o Ambiente de Serviço de Aplicativo | 100 nós por VMSS | Infinito <a href="#note1c"><sup>1</sup></a> | 100 | Não há limite definido, um máximo de 200 é recomendado | Limite de 20 núcleos por padrão. Contate o atendimento ao cliente para aumento. |

Observações

1. <span id="note1c">Se estiver usando o plano de Consumo. Se o plano de Serviço de Aplicativo, os limites de escala do Serviço de Aplicativo se aplicarão. Consulte [Escolher o plano de serviço correto para o Azure Functions][function-plans].</a>

## <a name="availability"></a>Disponibilidade

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Contêiner do Azure | Serviços de Nuvem | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Contrato de Nível de Serviço | [SLA para Máquinas Virtuais][sla-vm] | [SLA para Serviço de Aplicativo][sla-app-service] | [SLA para Service Fabric][sla-sf] | [SLA para Functions][sla-functions] | [SLA para o Serviço de Contêiner do Azure][sla-acs] | [SLA para Serviços de Nuvem][sla-cloud-service] | [SLA para o Lote do Azure][sla-batch] |
| Failover de várias regiões | Gerenciador de tráfego | Gerenciador de tráfego | Gerenciador de tráfego, Cluster de Várias Regiões | Sem suporte  | Gerenciador de tráfego | Gerenciador de tráfego | Sem suporte |

## <a name="security"></a>Segurança

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Contêiner do Azure | Serviços de Nuvem | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | Configurado na VM | Suportado | Com suporte  | Suportado | Configurado na VM | Suportado | Suportado |
| RBAC | Suportado | Com suporte | Com suporte | Com suporte | Suportado | Sem suporte | Suportado |

## <a name="other"></a>outro

| Critérios | Máquinas Virtuais | Serviço de Aplicativo | Service Fabric | Funções do Azure | Serviço de Contêiner do Azure | Serviços de Nuvem | Lote do Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Custo | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [Preços do Serviço de Aplicativo][cost-app-service] | [Preços do Service Fabric][cost-service-fabric] | [Preços do Azure Functions][cost-functions] | [Preços do Serviço de Contêiner do Azure][cost-acs] | [Preços dos Serviços de Nuvem][cost-cloud-services] | [Preço do Lote do Azure][cost-batch]
| Estilos de arquitetura adequados | N camadas, Big compute (HPC) | Trabalhador de fila da Web | Microsserviços, EDA (arquitetura orientada a eventos) | Microsserviços, EDA | Microsserviços, EDA | Trabalhador de fila da Web | Big Compute |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-cloud-services]: https://azure.microsoft.com/pricing/details/cloud-services/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-cloud-service]: https://azure.microsoft.com/support/legal/sla/cloud-services/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services