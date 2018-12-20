---
title: Executar simulações da dinâmica dos fluidos computacional (CFD) no Azure
description: Execute a dinâmica dos fluidos computacional (CFD) no Azure.
author: mikewarr
ms.date: 09/20/2018
ms.custom: fasttrack
ms.openlocfilehash: 42921122d74d07bf890f55be61b04c7e9a4f4e87
ms.sourcegitcommit: a0e8d11543751d681953717f6e78173e597ae207
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/06/2018
ms.locfileid: "53004649"
---
# <a name="running-computational-fluid-dynamics-cfd-simulations-on-azure"></a>Executar simulações da dinâmica dos fluidos computacional (CFD) no Azure

As simulações da dinâmica dos fluidos computacionais (CFD) exigem um tempo significativo de computação e um hardware especializado. À medida que aumenta o uso de cluster, os tempos de simulação e de uso geral da grade crescem, gerando problemas em relação à capacidade reserva e longas filas de espera. A adição de um hardware físico pode custar caro e pode não estar em conformidade com os períodos de altas e baixas de uma empresa. Usando o Azure, muitos desses desafios podem ser superados sem gastos extras.

O Azure fornece o hardware necessário para executar seus trabalhos da CFD nas máquinas virtuais do CPU e GPU. Os tamanhos de VM habilitados para o RDMA (Acesso Remoto Direto à Memória) têm redes com base no InfiniBand do FDR, o que permite a comunicação do MPI (Message Passing Interface) de baixa latência. Combinado com o Avere vFXT, que fornece um sistema de arquivos clusterizados de escala empresarial, os clientes podem garantir a melhor taxa de transferência para operações de leitura no Azure.

Para simplificar a criação, o gerenciamento e a otimização de clusters do HPC, o Azure CycleCloud pode ser usado para provisionar os clusters e organizar os dados em cenários na nuvem e híbridos. Com o monitoramento dos trabalhos pendentes, o CycleCloud inicia automaticamente a computação sob demanda, na qual você só paga pelo que usa, conectado ao agendador de carga de trabalho de sua preferência.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Outros setores relevantes para aplicativos de CFD incluem:

* Aeronáutico
* Automotivo
* HVAC de construção
* Óleo e gás
* Ciências da vida

## <a name="architecture"></a>Arquitetura

![Diagrama da arquitetura][architecture]

Este diagrama mostra uma visão geral de alto nível de um típico design híbrido que oferece o monitoramento dos nós sob demanda no Azure:

1. Conecte-se ao servidor do Azure CycleCloud para configurar o cluster.
2. Configure e crie o nó principal do cluster, usando máquinas com o RDMA habilitado para MPI.
3. Adicione e configure o nó principal local.
4. Caso não haja recursos o suficiente, o Azure CycleCloud irá aumentar (ou reduzir) os recursos de computação no Azure. É possível definir um limite predeterminado para evitar a alocação excessiva.
5. Tarefas alocadas para os nós de execução.
6. Dados armazenados em cache no Azure do servidor NFS local.
7. Dados lidos do cache do Avere vFXT para Azure.
8. Informações de trabalho e de tarefa retransmitidas para o servidor do Azure CycleCloud.

### <a name="components"></a>Componentes

* O [Azure CycleCloud][cyclecloud] é uma ferramenta para criar, gerenciar, operar e otimizar clusters de HPC e Big Compute no Azure.
* O [Avere vFXT no Azure][avere] é usado para fornecer um sistema de arquivos clusterizados de escala empresarial desenvolvido para a nuvem.
* As [Máquinas Virtuais (VMs) do Microsoft Azure][vms] são usadas para criar um conjunto estático de instâncias de computação.
* Os [Conjuntos de Dimensionamento de Máquinas Virtuais (conjunto de dimensionamento de máquinas virtuais)][vmss] fornece um grupo de VMs idênticas que podem ser ampliadas ou reduzidas no Azure CycleCloud.
* As [contas do Armazenamento do Azure](/azure/storage/common/storage-introduction) são usadas para a sincronização e a retenção de dados.
* As [Redes Virtuais](/azure/virtual-network/virtual-networks-overview) permitem vários tipos de recursos do Azure, como Máquinas Virtuais (VMs) do Microsoft Azure, a fim de se comunicar de forma segura com a Internet, com as redes locais e com outras VMs.

### <a name="alternatives"></a>Alternativas

Os clientes também podem usar o Azure CycleCloud para criar uma grade inteiramente no Azure. Nessa configuração, o servidor do Azure CycleCloud é executado dentro da assinatura do Azure.

Para uma abordagem de aplicativo moderno, em que o gerenciamento de um agendador de carga de trabalho não é necessário, o [Lote do Azure][batch] pode ajudar. O Lote do Azure pode executar com eficiência aplicativos HPC (computação de alto desempenho) e paralelos em grande escala na nuvem. O Lote do Azure permite definir os recursos de computação do Azure para executar seus aplicativos em paralelo ou em escala sem configurar manualmente ou gerenciar a infraestrutura. O Lote do Azure agenda as tarefas que requerem muita computação e adiciona e remove dinamicamente os recursos de computação com base em seus requisitos.

### <a name="scalability-and-security"></a>Escalabilidade e Segurança

O dimensionamento dos nós de execução no Azure CycleCloud pode ser feito manualmente ou usando o dimensionamento automático. Para saber mais, veja [Dimensionamento Automático do CycleCloud][cycle-scale].

Confira orientações gerais sobre como criar soluções seguras na [Documentação de segurança do Azure][security].

## <a name="deploy-this-scenario"></a>Implantar este cenário

Alguns pré-requisitos são necessários antes da implantação no Azure. Siga estas etapas antes de implantar o modelo do Resource Manager:
1. Crie uma [entidade de serviço][cycle-svcprin] para recuperar o appId, displayName, nome, senha e locatário.
2. Gere um [par de chaves SSH][cycle-ssh] para entrar com segurança no servidor CycleCloud.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

3. [Faça logon no servidor do CycleCloud][cycle-login] para configurar e criar um novo cluster.
4. [Crie um cluster][cycle-create].

O Cache do Avere é uma solução opcional que pode aumentar drasticamente a taxa de transferência de leitura dos dados do trabalho do aplicativo. O Avere vFXT para Azure resolve o problema da execução desses aplicativos de HPC na nuvem e aproveita os dados armazenados no local ou no Armazenamento de Blobs do Azure.

Para organizações que pretendem usar uma infraestrutura híbrida com computação na nuvem e armazenamento no local, os aplicativos de HPC podem “disparar” para o Azure usando os dados armazenados nos dispositivos NAS e processar CPUs virtuais conforme necessário. O conjunto de dados nunca é completamente movido para a nuvem. Os bytes solicitados são armazenados temporariamente em cache usando um cluster do Avere durante o processamento.

Para definir e configurar uma instalação do Avere vFXT, siga o [Guia de Instalação e Configuração do Avere][avere].

## <a name="pricing"></a>Preços

O custo de executar uma implantação de HPC usando o servidor do CycleCloud varia de acordo com uma série de fatores. Por exemplo, o CycleCloud é cobrado pelo tempo de computação que é usado, com o servidor do CycleCloud e Master normalmente sendo constantemente alocado e em execução. O custo para executar os nós de execução depende do tempo de operação deles, bem como do tamanho que é usado. Também se aplicam os encargos normais de armazenamento e rede do Azure.

Este cenário mostra como os aplicativos de CFD podem ser executados no Azure, as máquinas exigirão a funcionalidade de RDMA, que só está disponível em tamanhos específicos de máquina virtual. Veja a seguir exemplos de custos gerados para um conjunto de dimensionamento alocado continuamente oito horas por dia durante um mês, com saída de dados de 1 TB. Isso inclui também o preço do servidor do Azure CycleCloud e da instalação do Avere vFXT para Azure:

* Região: Norte da Europa
* Servidor Azure CycleCloud: 1 x Standard D3 (4 x CPUs, 14 GB de memória, Standard HDD 32 GB)
* Servidor mestre Azure CycleCloud: 1 x Standard D12 v (4 x CPUs, 28 GB de memória, Standard HDD 32 GB)
* Matriz de Nó do Azure CycleCloud: 10 x Standard H16r (16 x CPUs, 112 GB de memória)
* Avere vFXT no Azure Cluster: 3 x D16s v3 (Sistema operacional 200 GB, disco de dados Premium SSD 1-TB)
* Saída de Dados: 1 TB

Analise esta [estimativa de preço][pricing] do hardware listado acima.

## <a name="next-steps"></a>Próximas etapas

Depois de implantar o exemplo, saiba mais sobre o [Azure CycleCloud][cyclecloud].

## <a name="related-resources"></a>Recursos relacionados

* [Instâncias de Máquina Compatíveis com RDMA][rdma]
* [Personalizar uma Máquina Virtual da Instância do RDMA][rdma-custom]

<!-- links -->
[architecture]: ./media/architecture-hpc-cfd.png
[calculator]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[cyclecloud]: /azure/cyclecloud/
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[gpu]: /azure/virtual-machines/windows/sizes-gpu
[hpcsizes]: /azure/virtual-machines/windows/sizes-hpc
[vms]: /azure/virtual-machines/
[low-pri]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority
[batch]: /azure/batch/
[avere]: https://github.com/Azure/Avere/blob/master/README.md
[cycle-prereq]: /azure/cyclecloud/quickstart-install-cyclecloud#prerequisites
[cycle-svcprin]: /azure/cyclecloud/quickstart-install-cyclecloud#service-principal
[cycle-ssh]: /azure/cyclecloud/quickstart-install-cyclecloud#ssh-keypair
[cycle-login]: /azure/cyclecloud/quickstart-install-cyclecloud#log-into-the-cyclecloud-application-server
[cycle-create]: /azure/cyclecloud/quickstart-create-and-run-cluster
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[rdma-custom]: /azure/virtual-machines/linux/classic/rdma-cluster#customize-the-vm
[pricing]: https://azure.com/e/53030a04a2ab47a289156e2377a4247a
[cycle-scale]: /azure/cyclecloud/autoscale
