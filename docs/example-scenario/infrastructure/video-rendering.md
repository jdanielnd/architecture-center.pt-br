---
title: Renderização de vídeo 3D
titleSuffix: Azure Example Scenarios
description: Execute cargas de trabalho de HPC nativas no Azure usando o serviço de Lote do Azure.
author: adamboeglin
ms.date: 07/13/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
ms.openlocfilehash: ffb400f542b94ed02d1398b2e5e909d79708248b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485699"
---
# <a name="3d-video-rendering-on-azure"></a><span data-ttu-id="fe1f5-103">Renderização de vídeo em 3D no Azure</span><span class="sxs-lookup"><span data-stu-id="fe1f5-103">3D video rendering on Azure</span></span>

<span data-ttu-id="fe1f5-104">A renderização de vídeo 3D é um processo demorado que exige uma quantidade significativa de tempo de CPU para ser concluída.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-104">3D video rendering is a time consuming process that requires a significant amount of CPU time to complete.</span></span> <span data-ttu-id="fe1f5-105">Em um único computador, o processo de geração de um arquivo de vídeo de ativos estáticos pode levar horas ou mesmo dias dependendo do tamanho e da complexidade do vídeo criado.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-105">On a single machine, the process of generating a video file from static assets can take hours or even days depending on the length and complexity of the video you are producing.</span></span> <span data-ttu-id="fe1f5-106">Muitas empresas compram computadores topo de linha para executar essas tarefas ou investem em grandes farms de renderização para os quais elas podem enviar trabalhos.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-106">Many companies will purchase either expensive high end desktop computers to perform these tasks, or invest in large render farms that they can submit jobs to.</span></span> <span data-ttu-id="fe1f5-107">No entanto, ao tirar proveito do Lote do Azure, esse poder está disponível para você quando necessário e deixa de estar disponível quando você não precisa dele, tudo isso sem nenhum investimento de capital.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-107">However, by taking advantage of Azure Batch, that power is available to you when you need it and shuts itself down when you don't, all without any capital investment.</span></span>

<span data-ttu-id="fe1f5-108">O Lote oferece uma experiência consistente de gerenciamento e agendamento de trabalhos, quer você opte por nós de computação do Windows Server ou do Linux.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-108">Batch gives you a consistent management experience and job scheduling, whether you select Windows Server or Linux compute nodes.</span></span> <span data-ttu-id="fe1f5-109">Com o Lote, você pode usar os seus aplicativos Windows ou Linux existentes, incluindo AutoDesk Maya e Blender, para executar em trabalhos de renderização de grande escala no Azure.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-109">With Batch, you can use your existing Windows or Linux applications, including AutoDesk Maya and Blender, to run large-scale render jobs in Azure.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="fe1f5-110">Casos de uso relevantes</span><span class="sxs-lookup"><span data-stu-id="fe1f5-110">Relevant use cases</span></span>

<span data-ttu-id="fe1f5-111">Outros casos de uso relevantes incluem:</span><span class="sxs-lookup"><span data-stu-id="fe1f5-111">Other relevant use cases include:</span></span>

- <span data-ttu-id="fe1f5-112">Modelagem 3D</span><span class="sxs-lookup"><span data-stu-id="fe1f5-112">3D modeling</span></span>
- <span data-ttu-id="fe1f5-113">Renderização visual FX (VFX)</span><span class="sxs-lookup"><span data-stu-id="fe1f5-113">Visual FX (VFX) rendering</span></span>
- <span data-ttu-id="fe1f5-114">Transcodificação de vídeo</span><span class="sxs-lookup"><span data-stu-id="fe1f5-114">Video transcoding</span></span>
- <span data-ttu-id="fe1f5-115">Processamento de imagens, correção de cor e redimensionamento</span><span class="sxs-lookup"><span data-stu-id="fe1f5-115">Image processing, color correction, and resizing</span></span>

## <a name="architecture"></a><span data-ttu-id="fe1f5-116">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="fe1f5-116">Architecture</span></span>

![Visão geral da arquitetura dos componentes envolvidos em uma solução de HPC nativa de nuvem usando o Lote do Azure][architecture]

<span data-ttu-id="fe1f5-118">Este cenário mostra um fluxo de trabalho que usa o Lote do Azure.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-118">This scenario shows a workflow that uses Azure Batch.</span></span> <span data-ttu-id="fe1f5-119">O fluxo de dados é o seguinte:</span><span class="sxs-lookup"><span data-stu-id="fe1f5-119">The data flows as follows:</span></span>

1. <span data-ttu-id="fe1f5-120">Carregar os arquivos de entrada e o aplicativo para processar esses arquivos em sua conta do Armazenamento do Azure.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-120">Upload input files and the applications to process those files to your Azure Storage account.</span></span>
2. <span data-ttu-id="fe1f5-121">Criar um pool de nós de computação do Lote na sua conta do Lote, um trabalho para executar a carga de trabalho no pool e tarefas no trabalho.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-121">Create a Batch pool of compute nodes in your Batch account, a job to run the workload on the pool, and tasks in the job.</span></span>
3. <span data-ttu-id="fe1f5-122">Baixe arquivos de entrada e aplicativos para o Lote.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-122">Download input files and the applications to Batch.</span></span>
4. <span data-ttu-id="fe1f5-123">Monitorar a execução da tarefa.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-123">Monitor task execution.</span></span>
5. <span data-ttu-id="fe1f5-124">Carregar a saída da tarefa.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-124">Upload task output.</span></span>
6. <span data-ttu-id="fe1f5-125">Baixar os arquivos de saída.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-125">Download output files.</span></span>

<span data-ttu-id="fe1f5-126">Para simplificar esse processo, você também pode usar os [plug-ins do Lote para Maya e 3ds Max][batch-plugins]</span><span class="sxs-lookup"><span data-stu-id="fe1f5-126">To simplify this process, you could also use the [Batch Plugins for Maya and 3ds Max][batch-plugins]</span></span>

### <a name="components"></a><span data-ttu-id="fe1f5-127">Componentes</span><span class="sxs-lookup"><span data-stu-id="fe1f5-127">Components</span></span>

<span data-ttu-id="fe1f5-128">O Lote do Azure tem como base as tecnologias do Azure a seguir:</span><span class="sxs-lookup"><span data-stu-id="fe1f5-128">Azure Batch builds upon the following Azure technologies:</span></span>

- <span data-ttu-id="fe1f5-129">[Redes Virtuais](/azure/virtual-network/virtual-networks-overview) são usadas para o nó do cabeçalho e os recursos de computação.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-129">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) are used for both the head node and the compute resources.</span></span>
- <span data-ttu-id="fe1f5-130">As [contas do Armazenamento do Azure](/azure/storage/common/storage-introduction) são usadas para a sincronização e a retenção de dados.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-130">[Azure Storage accounts](/azure/storage/common/storage-introduction) are used for synchronization and data retention.</span></span>
- <span data-ttu-id="fe1f5-131">Os [Conjuntos de Dimensionamento de Máquinas Virtuais][vmss] são usados pelo CycleCloud para recursos de computação.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-131">[Virtual Machine Scale Sets][vmss] are used by CycleCloud for compute resources.</span></span>

## <a name="considerations"></a><span data-ttu-id="fe1f5-132">Considerações</span><span class="sxs-lookup"><span data-stu-id="fe1f5-132">Considerations</span></span>

### <a name="machine-sizes-available-for-azure-batch"></a><span data-ttu-id="fe1f5-133">Tamanhos de máquina disponíveis para o Lote do Azure</span><span class="sxs-lookup"><span data-stu-id="fe1f5-133">Machine Sizes available for Azure Batch</span></span>

<span data-ttu-id="fe1f5-134">Embora a maioria dos clientes de renderização escolha recursos com alta potência de CPU, outras cargas de trabalho que usam conjuntos de dimensionamento de máquinas virtuais podem escolher as VMs de forma diferente e dependerão de vários fatores:</span><span class="sxs-lookup"><span data-stu-id="fe1f5-134">While most rendering customers will choose resources with high CPU power, other workloads using virtual machine scale sets may choose VMs differently and will depend on a number of factors:</span></span>

- <span data-ttu-id="fe1f5-135">O aplicativo que está sendo executado tem um limite de memória?</span><span class="sxs-lookup"><span data-stu-id="fe1f5-135">Is the application being run memory bound?</span></span>
- <span data-ttu-id="fe1f5-136">O aplicativo precisa usar GPUs?</span><span class="sxs-lookup"><span data-stu-id="fe1f5-136">Does the application need to use GPUs?</span></span>
- <span data-ttu-id="fe1f5-137">Os tipos de trabalho são paralelos ou precisam de conectividade do Infiniband para trabalhos firmemente acoplados?</span><span class="sxs-lookup"><span data-stu-id="fe1f5-137">Are the job types embarrassingly parallel or require infiniband connectivity for tightly coupled jobs?</span></span>
- <span data-ttu-id="fe1f5-138">Exige E/S rápida para acessar o armazenamento em nós de computação.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-138">Require fast I/O to access storage on the compute Nodes.</span></span>

<span data-ttu-id="fe1f5-139">O Azure tem uma grande variedade de tamanhos de VM que podem atender todos os requisitos de aplicativo acima. Alguns são específicos para HPC, mas até mesmo os tamanhos menores podem ser utilizados para fornecer uma implementação de grade efetiva:</span><span class="sxs-lookup"><span data-stu-id="fe1f5-139">Azure has a wide range of VM sizes that can address each and every one of the above application requirements, some are specific to HPC, but even the smallest sizes can be utilized to provide an effective grid implementation:</span></span>

- <span data-ttu-id="fe1f5-140">[Tamanhos de VM de HPC][compute-hpc] Devido à limitação natural de renderização da CPU, a Microsoft normalmente sugere VMs da série H do Azure.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-140">[HPC VM sizes][compute-hpc] Due to the CPU bound nature of rendering, Microsoft typically suggests the Azure H-Series VMs.</span></span> <span data-ttu-id="fe1f5-141">Esse tipo de VM é criado especificamente para necessidades de computação de alto nível, está disponível em tamanhos de 8 e 16 núcleos de vCPU, memória DDR4, armazenamento temporário de SSD e tecnologia Haswell E5 da Intel.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-141">This type of VM is built specifically for high end computational needs, they have 8 and 16 core vCPU sizes available, and features DDR4 memory, SSD temporary storage, and Haswell E5 Intel technology.</span></span>
- <span data-ttu-id="fe1f5-142">[Tamanhos de VM de GPU][compute-gpu] Os tamanhos de VM otimizados para GPU são máquinas virtuais especializadas disponíveis com um ou vários GPUs NVIDIA.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-142">[GPU VM sizes][compute-gpu] GPU optimized VM sizes are specialized virtual machines available with single or multiple NVIDIA GPUs.</span></span> <span data-ttu-id="fe1f5-143">Esses tamanhos são projetados para cargas de trabalho de visualização e com muita computação e muitos gráficos.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-143">These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.</span></span>
- <span data-ttu-id="fe1f5-144">Os tamanhos NC, NCv2, NCv3 e ND são otimizados para aplicativos de rede e computação intensiva e algoritmos, incluindo aplicativos baseados em CUDA e OpenCL e simulações, AI e Aprendizagem Profunda.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-144">NC, NCv2, NCv3, and ND sizes are optimized for compute-intensive and network-intensive applications and algorithms, including CUDA and OpenCL-based applications and simulations, AI, and Deep Learning.</span></span> <span data-ttu-id="fe1f5-145">Os tamanhos NV são otimizados e projetados para cenários de visualização remota, streaming, jogos, codificação e VDI usando estruturas como OpenGL e DirectX.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-145">NV sizes are optimized and designed for remote visualization, streaming, gaming, encoding, and VDI scenarios utilizing frameworks such as OpenGL and DirectX.</span></span>
- <span data-ttu-id="fe1f5-146">[Tamanhos de VM otimizados para memória][compute-memory] Quando for necessária mais memória, os tamanhos de VM otimizados para memória oferecem uma maior proporção de memória para CPU.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-146">[Memory optimized VM sizes][compute-memory] When more memory is required, the memory optimized VM sizes offer a higher memory-to-CPU ratio.</span></span>
- <span data-ttu-id="fe1f5-147">[Tamanhos de VM para uso geral][compute-general] Tamanhos de VM para uso geral também estão disponíveis e oferecem uma proporção balanceada de CPU e memória.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-147">[General purposes VM sizes][compute-general] General-purpose VM sizes are also available and provide balanced CPU-to-memory ratio.</span></span>

### <a name="alternatives"></a><span data-ttu-id="fe1f5-148">Alternativas</span><span class="sxs-lookup"><span data-stu-id="fe1f5-148">Alternatives</span></span>

<span data-ttu-id="fe1f5-149">Se você precisar de mais controle sobre seu ambiente de renderização no Azure ou precisar de uma implementação híbrida, então a computação CycleCloud pode ajudar a coordenar uma grade de IaaS na nuvem.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-149">If you require more control over your rendering environment in Azure or need a hybrid implementation, then CycleCloud computing can help orchestrate an IaaS grid in the cloud.</span></span> <span data-ttu-id="fe1f5-150">Usando as mesmas tecnologias subjacentes do Azure como Lote do Azure, ele torna a criação e manutenção de uma grade de IaaS um processo eficiente.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-150">Using the same underlying Azure technologies as Azure Batch, it makes building and maintaining an IaaS grid an efficient process.</span></span> <span data-ttu-id="fe1f5-151">Para obter mais informações e saber mais sobre os princípios de design, use o seguinte link:</span><span class="sxs-lookup"><span data-stu-id="fe1f5-151">To find out more and learn about the design principles use the following link:</span></span>

<span data-ttu-id="fe1f5-152">Para obter uma visão geral completa de todas as soluções de HPC que estão disponíveis para você no Azure, consulte o artigo [Soluções de HPC, Lote e Big Compute usando VMs do Azure][hpc-alt-solutions]</span><span class="sxs-lookup"><span data-stu-id="fe1f5-152">For a complete overview of all the HPC solutions that are available to you in Azure, see the article [HPC, Batch, and Big Compute solutions using Azure VMs][hpc-alt-solutions]</span></span>

### <a name="availability"></a><span data-ttu-id="fe1f5-153">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="fe1f5-153">Availability</span></span>

<span data-ttu-id="fe1f5-154">O monitoramento dos componentes do Lote do Azure está disponível por meio de vários serviços, ferramentas e APIs.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-154">Monitoring of the Azure Batch components is available through a range of services, tools, and APIs.</span></span> <span data-ttu-id="fe1f5-155">Isso é discutido mais detalhadamente no artigo [Monitorar soluções do Lote][batch-monitor].</span><span class="sxs-lookup"><span data-stu-id="fe1f5-155">Monitoring is discussed further in the [Monitor Batch solutions][batch-monitor] article.</span></span>

### <a name="scalability"></a><span data-ttu-id="fe1f5-156">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="fe1f5-156">Scalability</span></span>

<span data-ttu-id="fe1f5-157">Pools dentro de uma conta do Lote do Azure podem ser dimensionados por meio de intervenção manual ou, usando uma fórmula com base em métricas do Lote do Azure, podem ser dimensionados automaticamente.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-157">Pools within an Azure Batch account can either scale through manual intervention or, by using a formula based on Azure Batch metrics, be scaled automatically.</span></span> <span data-ttu-id="fe1f5-158">Para obter mais informações sobre escalabilidade, consulte o artigo [Criar uma fórmula de dimensionamento automático para dimensionar nós em um pool do Lote][batch-scaling].</span><span class="sxs-lookup"><span data-stu-id="fe1f5-158">For more information on scalability, see the article [Create an automatic scaling formula for scaling nodes in a Batch pool][batch-scaling].</span></span>

### <a name="security"></a><span data-ttu-id="fe1f5-159">Segurança</span><span class="sxs-lookup"><span data-stu-id="fe1f5-159">Security</span></span>

<span data-ttu-id="fe1f5-160">Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].</span><span class="sxs-lookup"><span data-stu-id="fe1f5-160">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="fe1f5-161">Resiliência</span><span class="sxs-lookup"><span data-stu-id="fe1f5-161">Resiliency</span></span>

<span data-ttu-id="fe1f5-162">Embora não exista atualmente nenhuma funcionalidade de failover no Lote do Azure, é recomendável usar as seguintes etapas para garantir a disponibilidade em caso de interrupção não planejada:</span><span class="sxs-lookup"><span data-stu-id="fe1f5-162">While there is currently no failover capability in Azure Batch, we recommend using the following steps to ensure availability if there is an unplanned outage:</span></span>

- <span data-ttu-id="fe1f5-163">Criar uma conta do Lote do Azure em um local alternativo do Azure com uma conta de armazenamento alternativa</span><span class="sxs-lookup"><span data-stu-id="fe1f5-163">Create an Azure Batch account in an alternate Azure location with an alternate Storage Account</span></span>
- <span data-ttu-id="fe1f5-164">Criar os mesmos pools de nós com o mesmo nome, com nenhum nó alocado</span><span class="sxs-lookup"><span data-stu-id="fe1f5-164">Create the same node pools with the same name, with zero nodes allocated</span></span>
- <span data-ttu-id="fe1f5-165">Certificar-se de que os aplicativos são criados e atualizados para a conta de armazenamento alternativa</span><span class="sxs-lookup"><span data-stu-id="fe1f5-165">Ensure Applications are created and updated to the alternate Storage Account</span></span>
- <span data-ttu-id="fe1f5-166">Carregar arquivos de entrada e enviar trabalhos para a conta do Lote do Azure alternativa</span><span class="sxs-lookup"><span data-stu-id="fe1f5-166">Upload input files and submit jobs to the alternate Azure Batch account</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="fe1f5-167">Implantar o cenário</span><span class="sxs-lookup"><span data-stu-id="fe1f5-167">Deploy the scenario</span></span>

### <a name="create-an-azure-batch-account-and-pools-manually"></a><span data-ttu-id="fe1f5-168">Criar pools e uma conta do Lote do Azure manualmente</span><span class="sxs-lookup"><span data-stu-id="fe1f5-168">Create an Azure Batch account and pools manually</span></span>

<span data-ttu-id="fe1f5-169">Este cenário demonstra como o Lote do Azure funciona durante a apresentação de Laboratórios do Lote do Azure como um exemplo de solução de SaaS que pode ser desenvolvida para seus próprios clientes:</span><span class="sxs-lookup"><span data-stu-id="fe1f5-169">This scenario demonstrates how Azure Batch works while showcasing Azure Batch Labs as an example SaaS solution that can be developed for your own customers:</span></span>

<span data-ttu-id="fe1f5-170">[Masterclass do Lote do Azure][batch-labs-masterclass]</span><span class="sxs-lookup"><span data-stu-id="fe1f5-170">[Azure Batch Masterclass][batch-labs-masterclass]</span></span>

### <a name="deploy-the-components"></a><span data-ttu-id="fe1f5-171">Implantar os componentes</span><span class="sxs-lookup"><span data-stu-id="fe1f5-171">Deploy the components</span></span>

<span data-ttu-id="fe1f5-172">O modelo será implantado:</span><span class="sxs-lookup"><span data-stu-id="fe1f5-172">The template will deploy:</span></span>

- <span data-ttu-id="fe1f5-173">Uma nova conta do Lote do Azure</span><span class="sxs-lookup"><span data-stu-id="fe1f5-173">A new Azure Batch account</span></span>
- <span data-ttu-id="fe1f5-174">Uma conta de armazenamento</span><span class="sxs-lookup"><span data-stu-id="fe1f5-174">A storage account</span></span>
- <span data-ttu-id="fe1f5-175">Um pool de nós associado à conta do lote</span><span class="sxs-lookup"><span data-stu-id="fe1f5-175">A node pool associated with the batch account</span></span>
- <span data-ttu-id="fe1f5-176">O pool de nós será configurado para usar VMs de A2 v2 com imagens do Ubuntu da Canonical</span><span class="sxs-lookup"><span data-stu-id="fe1f5-176">The node pool will be configured to use A2 v2 VMs with Canonical Ubuntu images</span></span>
- <span data-ttu-id="fe1f5-177">O pool de nós conterá zero VMs inicialmente e exigirá dimensionando manual para adicionar máquinas virtuais</span><span class="sxs-lookup"><span data-stu-id="fe1f5-177">The node pool will contain zero VMs initially and will require you to manually scale to add VMs</span></span>

<!-- markdownlint-disable MD033 -->

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>
<!-- markdownlint-enable MD033 -->

<span data-ttu-id="fe1f5-178">[Saiba mais sobre modelos do Resource Manager][azure-arm-templates]</span><span class="sxs-lookup"><span data-stu-id="fe1f5-178">[Learn more about Resource Manager templates][azure-arm-templates]</span></span>

## <a name="pricing"></a><span data-ttu-id="fe1f5-179">Preços</span><span class="sxs-lookup"><span data-stu-id="fe1f5-179">Pricing</span></span>

<span data-ttu-id="fe1f5-180">O custo de usar o Lote do Azure dependerá dos tamanhos de VM que são usados para os pools e quanto tempo essas VMs ficam alocadas e em execução, não há nenhum custo associado a uma criação de conta do Lote do Azure.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-180">The cost of using Azure Batch will depend on the VM sizes that are used for the pools and how long these VMs are allocated and running, there is no cost associated with an Azure Batch account creation.</span></span> <span data-ttu-id="fe1f5-181">A saída de dados e o armazenamento devem ser considerados já que implicarão em custos adicionais.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-181">Storage and data egress should be taken into account as these will apply additional costs.</span></span>

<span data-ttu-id="fe1f5-182">A seguir estão exemplos de custos que poderiam ser cobrados para um trabalho que termina em 8 horas usando um número diferente de servidores:</span><span class="sxs-lookup"><span data-stu-id="fe1f5-182">The following are examples of costs that could be incurred for a job that completes in 8 hours using a different number of servers:</span></span>

- <span data-ttu-id="fe1f5-183">100 VMs com CPU de alto desempenho: [Estimativa de custo][hpc-est-high]</span><span class="sxs-lookup"><span data-stu-id="fe1f5-183">100 High-Performance CPU VMs: [Cost Estimate][hpc-est-high]</span></span>

  <span data-ttu-id="fe1f5-184">100 x H16m (16 núcleos, 225 GB de RAM, Armazenamento Premium de 512 GB), Armazenamento de Blobs de 2 TB, saída de 1 TB</span><span class="sxs-lookup"><span data-stu-id="fe1f5-184">100 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

- <span data-ttu-id="fe1f5-185">50 VMs com CPU de alto desempenho: [Estimativa de custo][hpc-est-med]</span><span class="sxs-lookup"><span data-stu-id="fe1f5-185">50 High-Performance CPU VMs: [Cost Estimate][hpc-est-med]</span></span>

  <span data-ttu-id="fe1f5-186">50 x H16m (16 núcleos, 225 GB de RAM, Armazenamento Premium de 512 GB), Armazenamento de Blobs de 2 TB, saída de 1 TB</span><span class="sxs-lookup"><span data-stu-id="fe1f5-186">50 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

- <span data-ttu-id="fe1f5-187">10 VMs com CPU de alto desempenho: [Estimativa de custo][hpc-est-low]</span><span class="sxs-lookup"><span data-stu-id="fe1f5-187">10 High-Performance CPU VMs: [Cost Estimate][hpc-est-low]</span></span>

  <span data-ttu-id="fe1f5-188">10 x H16m (16 núcleos, 225 GB de RAM, Armazenamento Premium de 512 GB), Armazenamento de Blobs de 2 TB, saída de 1 TB</span><span class="sxs-lookup"><span data-stu-id="fe1f5-188">10 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

### <a name="pricing-for-low-priority-vms"></a><span data-ttu-id="fe1f5-189">Preços de VMs de baixa prioridade</span><span class="sxs-lookup"><span data-stu-id="fe1f5-189">Pricing for low-priority VMs</span></span>

<span data-ttu-id="fe1f5-190">O Lote do Azure também dá suporte ao uso de VMs de baixa prioridade nos pools de nó, que potencialmente podem fornecer uma economia significativa de custos.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-190">Azure Batch also supports the use of low-priority VMs in the node pools, which can potentially provide a substantial cost saving.</span></span> <span data-ttu-id="fe1f5-191">Para saber mais, incluindo uma comparação de preços entre as VMs padrão e as VMs de baixa prioridade, confira [Preços do Lote do Azure][batch-pricing].</span><span class="sxs-lookup"><span data-stu-id="fe1f5-191">For more information, including a price comparison between standard VMs and low-priority VMs, see [Azure Batch Pricing][batch-pricing].</span></span>

> [!NOTE]
> <span data-ttu-id="fe1f5-192">VMs de baixa prioridade só são adequadas para determinados aplicativos e cargas de trabalho.</span><span class="sxs-lookup"><span data-stu-id="fe1f5-192">Low-priority VMs are only suitable for certain applications and workloads.</span></span>

## <a name="related-resources"></a><span data-ttu-id="fe1f5-193">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="fe1f5-193">Related resources</span></span>

<span data-ttu-id="fe1f5-194">[Visão Geral do Lote do Azure][batch-overview]</span><span class="sxs-lookup"><span data-stu-id="fe1f5-194">[Azure Batch Overview][batch-overview]</span></span>

<span data-ttu-id="fe1f5-195">[Documentação do Lote do Azure][batch-doc]</span><span class="sxs-lookup"><span data-stu-id="fe1f5-195">[Azure Batch Documentation][batch-doc]</span></span>

<span data-ttu-id="fe1f5-196">[Usar contêineres no Lote do Azure][batch-containers]</span><span class="sxs-lookup"><span data-stu-id="fe1f5-196">[Using containers on Azure Batch][batch-containers]</span></span>

<!-- links -->
[architecture]: ./media/architecture-video-rendering.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[storage]: https://azure.microsoft.com/services/storage/
[batch]: https://azure.microsoft.com/services/batch/
[batch-arch]: https://azure.microsoft.com/solutions/architecture/big-compute-with-azure-batch/
[compute-hpc]: /azure/virtual-machines/windows/sizes-hpc
[compute-gpu]: /azure/virtual-machines/windows/sizes-gpu
[compute-compute]: /azure/virtual-machines/windows/sizes-compute
[compute-memory]: /azure/virtual-machines/windows/sizes-memory
[compute-general]: /azure/virtual-machines/windows/sizes-general
[compute-storage]: /azure/virtual-machines/windows/sizes-storage
[compute-acu]: /azure/virtual-machines/windows/acu
[compute=benchmark]: /azure/virtual-machines/windows/compute-benchmark-scores
[hpc-est-high]: https://azure.com/e/9ac25baf44ef49c3a6b156935ee9544c
[hpc-est-med]: https://azure.com/e/0286f1d6f6784310af4dcda5aec8c893
[hpc-est-low]: https://azure.com/e/e39afab4e71949f9bbabed99b428ba4a
[batch-labs-masterclass]: https://github.com/azurebigcompute/BigComputeLabs/tree/master/Azure%20Batch%20Masterclass%20Labs
[batch-scaling]: /azure/batch/batch-automatic-scaling
[hpc-alt-solutions]: /azure/virtual-machines/linux/high-performance-computing?toc=%2fazure%2fbatch%2ftoc.json
[batch-monitor]: /azure/batch/monitoring-overview
[batch-pricing]: https://azure.microsoft.com/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job