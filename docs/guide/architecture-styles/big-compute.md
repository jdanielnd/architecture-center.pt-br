---
title: Estilo de arquitetura de computação intensa
titleSuffix: Azure Application Architecture Guide
description: Descreve os benefícios, os desafios e as práticas recomendadas para arquiteturas de computação intensa no Azure.
author: MikeWasson
ms.date: 08/30/2018
ms.custom: seojan19
ms.openlocfilehash: 7dbd8e25a0db79e6dde4c1c7e787eaa040ffdb3b
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114055"
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="288f7-103">Estilo de arquitetura de computação intensa</span><span class="sxs-lookup"><span data-stu-id="288f7-103">Big compute architecture style</span></span>

<span data-ttu-id="288f7-104">O termo *computação intensa* descreve as cargas de trabalho em grande escala que exigem uma grande quantidade de núcleos, geralmente centenas ou milhares deles.</span><span class="sxs-lookup"><span data-stu-id="288f7-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="288f7-105">Entre os cenários, temos renderização de imagem, dinâmica de fluidos, modelagem de riscos financeiros, exploração de petróleo, concepção de medicamentos e análise de estresse de engenharia, entre outros.</span><span class="sxs-lookup"><span data-stu-id="288f7-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![Diagrama lógico do estilo de arquitetura de computação intensa](./images/big-compute-logical.png)

<span data-ttu-id="288f7-107">Estas são algumas características comuns de aplicativos de computação intensa:</span><span class="sxs-lookup"><span data-stu-id="288f7-107">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="288f7-108">O trabalho pode ser dividido em tarefas discretas, as quais podem ser executadas simultaneamente em vários núcleos.</span><span class="sxs-lookup"><span data-stu-id="288f7-108">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="288f7-109">Cada tarefa é finita.</span><span class="sxs-lookup"><span data-stu-id="288f7-109">Each task is finite.</span></span> <span data-ttu-id="288f7-110">É preciso de alguma entrada, algum processamento é executado e se produz uma saída.</span><span class="sxs-lookup"><span data-stu-id="288f7-110">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="288f7-111">O aplicativo como um todo é executado por uma quantidade finita de tempo (de minutos a dias).</span><span class="sxs-lookup"><span data-stu-id="288f7-111">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="288f7-112">Um padrão comum é provisionar uma grande quantidade de núcleos com um disparo contínuo e depois diminuir até zero assim que o aplicativo concluir.</span><span class="sxs-lookup"><span data-stu-id="288f7-112">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span>
- <span data-ttu-id="288f7-113">O aplicativo não precisa ficar ativo 24/7.</span><span class="sxs-lookup"><span data-stu-id="288f7-113">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="288f7-114">No entanto, o sistema deve tratar as falhas de nó ou panes do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="288f7-114">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="288f7-115">Para alguns aplicativos, as tarefas são independentes e podem ser executados em paralelo.</span><span class="sxs-lookup"><span data-stu-id="288f7-115">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="288f7-116">Em outros casos, as tarefas estão firmemente acopladas, o que significa que elas devem interagir ou trocar resultados intermediários.</span><span class="sxs-lookup"><span data-stu-id="288f7-116">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="288f7-117">Nesse caso, leve em consideração o uso de tecnologias de rede de alta velocidade, como InfiniBand e RDMA (acesso remoto direto à memória).</span><span class="sxs-lookup"><span data-stu-id="288f7-117">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span>
- <span data-ttu-id="288f7-118">Dependendo da sua carga de trabalho, é possível usar os tamanhos de VM de computação intensiva (H16r, H16mr e A9).</span><span class="sxs-lookup"><span data-stu-id="288f7-118">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="288f7-119">Quando usar essa arquitetura</span><span class="sxs-lookup"><span data-stu-id="288f7-119">When to use this architecture</span></span>

- <span data-ttu-id="288f7-120">Operações computacionalmente intensivas, como simulações e processamentos de números.</span><span class="sxs-lookup"><span data-stu-id="288f7-120">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="288f7-121">Simulações são computacionalmente intensivas e devem ser divididas entre CPUs de vários computadores (dezenas a milhares).</span><span class="sxs-lookup"><span data-stu-id="288f7-121">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="288f7-122">Simulações que requerem muita memória para um computador e devem ser divididas entre vários computadores.</span><span class="sxs-lookup"><span data-stu-id="288f7-122">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="288f7-123">Cálculos de longa execução que levariam muito tempo para serem concluídos em um único computador.</span><span class="sxs-lookup"><span data-stu-id="288f7-123">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="288f7-124">Computações menores que devem ser executadas centenas ou milhares de vezes, como simulações de Monte Carlo.</span><span class="sxs-lookup"><span data-stu-id="288f7-124">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="288f7-125">Benefícios</span><span class="sxs-lookup"><span data-stu-id="288f7-125">Benefits</span></span>

- <span data-ttu-id="288f7-126">Alto desempenho com processamento “[embaraçosamente paralelo][embarrassingly-parallel]”.</span><span class="sxs-lookup"><span data-stu-id="288f7-126">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="288f7-127">Pode aproveitar centenas ou milhares de núcleos de computador para solucionar grandes problemas com mais rapidez.</span><span class="sxs-lookup"><span data-stu-id="288f7-127">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="288f7-128">Acesso a hardware especializado de alto desempenho e com redes de alta velocidade InfiniBand dedicadas.</span><span class="sxs-lookup"><span data-stu-id="288f7-128">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="288f7-129">Você pode provisionar quantas VMs forem necessárias para trabalhar e depois subdividi-las.</span><span class="sxs-lookup"><span data-stu-id="288f7-129">You can provision VMs as needed to do work, and then tear them down.</span></span>

## <a name="challenges"></a><span data-ttu-id="288f7-130">Desafios</span><span class="sxs-lookup"><span data-stu-id="288f7-130">Challenges</span></span>

- <span data-ttu-id="288f7-131">Gerenciar a infraestrutura da VM.</span><span class="sxs-lookup"><span data-stu-id="288f7-131">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="288f7-132">Gerenciar o volume de processamento de números</span><span class="sxs-lookup"><span data-stu-id="288f7-132">Managing the volume of number crunching</span></span>
- <span data-ttu-id="288f7-133">Provisionar milhares de núcleos em tempo hábil.</span><span class="sxs-lookup"><span data-stu-id="288f7-133">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="288f7-134">Para tarefas firmemente acopladas, a adição de mais núcleos pode levar à diminuição de retornos.</span><span class="sxs-lookup"><span data-stu-id="288f7-134">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="288f7-135">Talvez seja preciso fazer experimentos para encontrar a quantidade ideal de núcleos.</span><span class="sxs-lookup"><span data-stu-id="288f7-135">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="288f7-136">Computação intensa usando o Lote do Azure</span><span class="sxs-lookup"><span data-stu-id="288f7-136">Big compute using Azure Batch</span></span>

<span data-ttu-id="288f7-137">O [Lote do Azure][batch] é um serviço gerenciado para execução de aplicativos HPC (computação de alto desempenho) em grande escala.</span><span class="sxs-lookup"><span data-stu-id="288f7-137">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="288f7-138">Ao usar o Lote do Azure, você configura um pool de VMs e carrega os aplicativos e os arquivos de dados.</span><span class="sxs-lookup"><span data-stu-id="288f7-138">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="288f7-139">Depois o serviço do Lote provisiona as VMs, atribui tarefas às VMs, executa as tarefas e monitora o progresso.</span><span class="sxs-lookup"><span data-stu-id="288f7-139">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="288f7-140">O Lote pode se escalar horizontal e automaticamente as VMs em resposta à carga de trabalho.</span><span class="sxs-lookup"><span data-stu-id="288f7-140">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="288f7-141">O Lote também fornece o agendamento de trabalho.</span><span class="sxs-lookup"><span data-stu-id="288f7-141">Batch also provides job scheduling.</span></span>

![Diagrama de computação intensa usando o Lote do Azure](./images/big-compute-batch.png)

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="288f7-143">Computação intensa em execução em Máquinas Virtuais</span><span class="sxs-lookup"><span data-stu-id="288f7-143">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="288f7-144">Você pode usar [Microsoft HPC Pack][hpc-pack] para administrar um cluster de VMs, além de agendar e monitorar trabalhos de HPC.</span><span class="sxs-lookup"><span data-stu-id="288f7-144">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="288f7-145">Com essa abordagem, é preciso provisionar e gerenciar a infraestrutura das VMs e da rede.</span><span class="sxs-lookup"><span data-stu-id="288f7-145">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="288f7-146">Leve essa abordagem em consideração se você tiver cargas de trabalho de HPC existentes e quiser mover algumas ou todas elas para o Azure.</span><span class="sxs-lookup"><span data-stu-id="288f7-146">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="288f7-147">É possível mover todo o cluster de HPC para o Azure ou manter o cluster de HPC local, mas usar o Azure para a capacidade de disparo contínuo.</span><span class="sxs-lookup"><span data-stu-id="288f7-147">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="288f7-148">Para obter mais informações, confira [Soluções HPC e Lote para cargas de trabalho de computação em larga escala][batch-hpc-solutions].</span><span class="sxs-lookup"><span data-stu-id="288f7-148">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="288f7-149">HPC Pack implantado no Azure</span><span class="sxs-lookup"><span data-stu-id="288f7-149">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="288f7-150">Nesse cenário, o cluster de HPC é criado inteiramente dentro do Azure.</span><span class="sxs-lookup"><span data-stu-id="288f7-150">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![Diagrama do HPC Pack implantado no Azure](./images/big-compute-iaas.png)

<span data-ttu-id="288f7-152">O nó principal fornece os serviços de gerenciamento e de agendamento de trabalho para o cluster.</span><span class="sxs-lookup"><span data-stu-id="288f7-152">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="288f7-153">Para tarefas firmemente acopladas, use uma rede de RDMA que forneça comunicação entre máquinas virtuais com largura de banda muito alta e baixa latência.</span><span class="sxs-lookup"><span data-stu-id="288f7-153">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="288f7-154">Para obter mais informações, confira [Implantar um cluster de HPC Pack 2016 no Azure][deploy-hpc-azure].</span><span class="sxs-lookup"><span data-stu-id="288f7-154">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="288f7-155">Fazer disparo contínuo de um cluster HPC para o Azure</span><span class="sxs-lookup"><span data-stu-id="288f7-155">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="288f7-156">Nesse cenário, uma organização está executando o HPC Pack local e usa as VMs do Azure para a capacidade de disparo.</span><span class="sxs-lookup"><span data-stu-id="288f7-156">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="288f7-157">O nó principal do cluster é local.</span><span class="sxs-lookup"><span data-stu-id="288f7-157">The cluster head node is on-premises.</span></span> <span data-ttu-id="288f7-158">O ExpressRoute ou Gateway de VPN conecta rede local à VNet do Azure.</span><span class="sxs-lookup"><span data-stu-id="288f7-158">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![Diagrama de um cluster de computação intensa híbrido](./images/big-compute-hybrid.png)

<!-- links -->

[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029
