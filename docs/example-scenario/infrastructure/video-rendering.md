---
title: Renderização de vídeo em 3D no Azure
description: Executar cargas de trabalho de HPC nativas no Azure usando o serviço de Lote do Azure
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: e629e2ba0b9490e534057fee33f7bededa9656af
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229177"
---
# <a name="3d-video-rendering-on-azure"></a><span data-ttu-id="188bb-103">Renderização de vídeo em 3D no Azure</span><span class="sxs-lookup"><span data-stu-id="188bb-103">3D video rendering on Azure</span></span>

<span data-ttu-id="188bb-104">A renderização 3D é um processo demorado que exige uma quantidade significativa de tempo de CPU para ser concluída.</span><span class="sxs-lookup"><span data-stu-id="188bb-104">3D rendering is a time consuming process that requires a significant amount of CPU time to complete.</span></span>  <span data-ttu-id="188bb-105">Em um único computador, o processo de geração de um arquivo de vídeo de ativos estáticos pode levar horas ou mesmo dias dependendo do tamanho e da complexidade do vídeo criado.</span><span class="sxs-lookup"><span data-stu-id="188bb-105">On a single machine, the process of generating a video file from static assets can take hours or even days depending on the length and complexity of the video you are producing.</span></span>  <span data-ttu-id="188bb-106">Muitas empresas compram computadores topo de linha para executar essas tarefas ou investem em grandes farms de renderização para os quais elas podem enviar trabalhos.</span><span class="sxs-lookup"><span data-stu-id="188bb-106">Many companies will purchase either expensive high end desktop computers to perform these tasks, or invest in large render farms that they can submit jobs to.</span></span>  <span data-ttu-id="188bb-107">No entanto, ao tirar proveito do Lote do Azure, esse poder está disponível para você quando necessário e deixa de estar disponível quando você não precisa dele, tudo isso sem nenhum investimento de capital.</span><span class="sxs-lookup"><span data-stu-id="188bb-107">However, by taking advantage of Azure Batch, that power is available to you when you need it and shuts itself down when you don't, all without any capital investment.</span></span>

<span data-ttu-id="188bb-108">O Lote oferece uma experiência consistente de gerenciamento e agendamento de trabalhos, quer você opte por nós de computação do Windows Server ou do Linux.</span><span class="sxs-lookup"><span data-stu-id="188bb-108">Batch gives you a consistent management experience and job scheduling, whether you select Windows Server or Linux compute nodes.</span></span> <span data-ttu-id="188bb-109">Com o Lote, você pode usar os seus aplicativos Windows ou Linux existentes, incluindo AutoDesk Maya e Blender, para executar em trabalhos de renderização de grande escala no Azure.</span><span class="sxs-lookup"><span data-stu-id="188bb-109">With Batch, you can use your existing Windows or Linux applications, including AutoDesk Maya and Blender, to run large-scale render jobs in Azure.</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="188bb-110">Casos de uso relacionados</span><span class="sxs-lookup"><span data-stu-id="188bb-110">Related use cases</span></span>

<span data-ttu-id="188bb-111">Considere este cenário para esses casos de uso semelhantes:</span><span class="sxs-lookup"><span data-stu-id="188bb-111">Consider this scenario for these similar use cases:</span></span>

* <span data-ttu-id="188bb-112">Modelagem 3D</span><span class="sxs-lookup"><span data-stu-id="188bb-112">3D Modeling</span></span>
* <span data-ttu-id="188bb-113">Renderização visual FX (VFX)</span><span class="sxs-lookup"><span data-stu-id="188bb-113">Visual FX (VFX) Rendering</span></span>
* <span data-ttu-id="188bb-114">Transcodificação de vídeo</span><span class="sxs-lookup"><span data-stu-id="188bb-114">Video transcoding</span></span>
* <span data-ttu-id="188bb-115">Processamento de imagens, correção de cor e redimensionamento</span><span class="sxs-lookup"><span data-stu-id="188bb-115">Image processing, color correction, & resizing</span></span>

## <a name="architecture"></a><span data-ttu-id="188bb-116">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="188bb-116">Architecture</span></span>

![Visão geral da arquitetura dos componentes envolvidos em uma solução de HPC nativa de nuvem usando o Lote do Azure][architecture]

<span data-ttu-id="188bb-118">Este exemplo de cenário aborda o fluxo de trabalho ao usar o Lote do Azure, o fluxo de dados ocorre da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="188bb-118">This sample scenario covers the workflow when using Azure Batch, the data flows as follows:</span></span>

1. <span data-ttu-id="188bb-119">Carregar os arquivos de entrada e o aplicativo para processar esses arquivos em sua conta do Armazenamento do Azure</span><span class="sxs-lookup"><span data-stu-id="188bb-119">Upload input files and the applications to process those files to your Azure Storage account</span></span>
2. <span data-ttu-id="188bb-120">Criar um pool de nós de computação do Lote na sua conta do Lote, um trabalho para executar a carga de trabalho no pool e tarefas no trabalho.</span><span class="sxs-lookup"><span data-stu-id="188bb-120">Create a Batch pool of compute nodes in your Batch account, a job to run the workload on the pool, and tasks in the job.</span></span>
3. <span data-ttu-id="188bb-121">Baixe arquivos de entrada e aplicativos para o Lote</span><span class="sxs-lookup"><span data-stu-id="188bb-121">Download input files and the applications to Batch</span></span>
4. <span data-ttu-id="188bb-122">Monitorar a execução de tarefas</span><span class="sxs-lookup"><span data-stu-id="188bb-122">Monitor task execution</span></span>
5. <span data-ttu-id="188bb-123">Carregar a saída da tarefa</span><span class="sxs-lookup"><span data-stu-id="188bb-123">Upload task output</span></span>
6. <span data-ttu-id="188bb-124">Baixar os arquivos de saída</span><span class="sxs-lookup"><span data-stu-id="188bb-124">Download output files</span></span>

<span data-ttu-id="188bb-125">Para simplificar esse processo, você também pode usar os [plug-ins do Lote para Maya & 3ds Max][batch-plugins]</span><span class="sxs-lookup"><span data-stu-id="188bb-125">To simplify this process, you could also use the [Batch Plugins for Maya & 3ds Max][batch-plugins]</span></span>

### <a name="components"></a><span data-ttu-id="188bb-126">Componentes</span><span class="sxs-lookup"><span data-stu-id="188bb-126">Components</span></span>

<span data-ttu-id="188bb-127">O Lote do Azure tem como base as tecnologias do Azure a seguir:</span><span class="sxs-lookup"><span data-stu-id="188bb-127">Azure Batch builds upon the following Azure technologies:</span></span>

* <span data-ttu-id="188bb-128">[Grupos de Recursos][resource-groups] são um contêiner lógico para recursos do Azure.</span><span class="sxs-lookup"><span data-stu-id="188bb-128">[Resource Groups][resource-groups] is a logical container for Azure resources.</span></span>
* <span data-ttu-id="188bb-129">[Redes Virtuais][vnet] são usadas para o nó do cabeçalho e os recursos de computação</span><span class="sxs-lookup"><span data-stu-id="188bb-129">[Virtual Networks][vnet] are used to for both the Head Node and Compute resources</span></span>
* <span data-ttu-id="188bb-130">Contas de [Armazenamento][storage] são usadas para a retenção de dados e sincronização</span><span class="sxs-lookup"><span data-stu-id="188bb-130">[Storage][storage] accounts are used for the synchronisation and data retention</span></span>
* <span data-ttu-id="188bb-131">[Conjuntos de Dimensionamento de Máquinas Virtuais][vmss] são usadas pelo CycleCloud para recursos de computação</span><span class="sxs-lookup"><span data-stu-id="188bb-131">[Virtual Machine Scale Sets][vmss] are utilised by CycleCloud for compute resources</span></span>

## <a name="considerations"></a><span data-ttu-id="188bb-132">Considerações</span><span class="sxs-lookup"><span data-stu-id="188bb-132">Considerations</span></span>

### <a name="machine-sizes-available-for-azure-batch"></a><span data-ttu-id="188bb-133">Tamanhos de máquina disponíveis para o Lote do Azure</span><span class="sxs-lookup"><span data-stu-id="188bb-133">Machine Sizes available for Azure Batch</span></span>
<span data-ttu-id="188bb-134">Embora a maioria dos clientes escolha recursos com alta potência de CPU, outras cargas de trabalho que usam conjuntos de dimensionamento de VM podem escolher as VM de forma diferente e dependerão de vários fatores:</span><span class="sxs-lookup"><span data-stu-id="188bb-134">While most rendering customers whill choose resources with high CPU power, other workloads using VM Scale Sets may choose VM's differently and will depend on a number of factors:</span></span>
  - <span data-ttu-id="188bb-135">O aplicativo que está sendo executado possui um limite de memória?</span><span class="sxs-lookup"><span data-stu-id="188bb-135">Is the Application being run memory bound?</span></span>
  - <span data-ttu-id="188bb-136">O aplicativo precisa usar GPU?</span><span class="sxs-lookup"><span data-stu-id="188bb-136">Does the Application need to use GPU's?</span></span> 
  - <span data-ttu-id="188bb-137">Os tipos de trabalho paralelo são paralelos ou precisam de conectividade do Infiniband para trabalhos firmemente acoplados?</span><span class="sxs-lookup"><span data-stu-id="188bb-137">Are the job types embarassingly parallel or require Infiniband connectivity for tightly coupled jobs?</span></span>
  - <span data-ttu-id="188bb-138">Exige E/S rápida para o armazenamento em nós de computação</span><span class="sxs-lookup"><span data-stu-id="188bb-138">Require fast I/O to Storage on the Compute Nodes</span></span>

<span data-ttu-id="188bb-139">O Azure tem uma grande variedade de tamanhos de VM que podem atender a todos e cada um dos requisitos de aplicativo acima, alguns são específicos para HPC, mas até mesmo os tamanhos menores podem ser utilizados para fornecer uma implementação de grade efetiva:</span><span class="sxs-lookup"><span data-stu-id="188bb-139">Azure has a wide range of VM sizes that can address each and every one of the above application requirements, some are specific to HPC, but even the smallest sizes can be utilised to provide an effective grid implementation:</span></span>

  - <span data-ttu-id="188bb-140">[Tamanhos de VM do HPC][compute-hpc] devido à limitação natural de renderização da CPU, a Microsoft normalmente sugere VM’s da série H do Azure.</span><span class="sxs-lookup"><span data-stu-id="188bb-140">[HPC VM sizes][compute-hpc] Due to the CPU bound nature of rendering, Microsoft typically suggests the Azure H-Series VM's.</span></span>  <span data-ttu-id="188bb-141">Elas são criados especificamente disponíveis para necessidades de computação de alto nível, possuem tamanhos de 8 e 16 núcleos de vCPU disponíveis, e memória DDR4, armazenamento temporário de SSD e tecnologia Haswell E5 da Intel.</span><span class="sxs-lookup"><span data-stu-id="188bb-141">These are built specifically available for high end computational needs, have 8 and 16 core vCPU sizes available, and feature DDR4 memory, SSD temporary storage and Haswell E5 Intel technology.</span></span>
  - <span data-ttu-id="188bb-142">[Tamanhos de VM de GPU][compute-gpu] Os tamanhos de VM otimizados para GPU são máquinas virtuais especializadas disponíveis com um ou vários GPUs NVIDIA.</span><span class="sxs-lookup"><span data-stu-id="188bb-142">[GPU VM sizes][compute-gpu] GPU optimized VM sizes are specialized virtual machines available with single or multiple NVIDIA GPUs.</span></span> <span data-ttu-id="188bb-143">Esses tamanhos são projetados para cargas de trabalho de visualização e com muita computação e muitos gráficos.</span><span class="sxs-lookup"><span data-stu-id="188bb-143">These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.</span></span>
    - <span data-ttu-id="188bb-144">Os tamanhos NC, NCv2, NCv3 e ND são otimizados para aplicativos de rede e computação intensiva e algoritmos, incluindo aplicativos baseados em CUDA e OpenCL e simulações, AI e Aprendizagem Profunda.</span><span class="sxs-lookup"><span data-stu-id="188bb-144">NC, NCv2, NCv3, and ND sizes are optimized for compute-intensive and network-intensive applications and algorithms, including CUDA and OpenCL-based applications and simulations, AI, and Deep Learning.</span></span> <span data-ttu-id="188bb-145">Os tamanhos NV são otimizados e projetados para cenários de visualização remota, streaming, jogos, codificação e VDI usando estruturas como OpenGL e DirectX.</span><span class="sxs-lookup"><span data-stu-id="188bb-145">NV sizes are optimized and designed for remote visualization, streaming, gaming, encoding, and VDI scenarios utilizing frameworks such as OpenGL and DirectX.</span></span>
  - <span data-ttu-id="188bb-146">[Tamanhos de VM otimizados para memória][compute-memory] Quando for necessária mais memória, os tamanhos de VM otimizados para memória oferecem uma maior taxa de memória para CPU.</span><span class="sxs-lookup"><span data-stu-id="188bb-146">[Memory optmised VM sizes][compute-memory] When more memory is required, the memory optimized VM sizes offer a higher memory-to-CPU ratio.</span></span>
  - <span data-ttu-id="188bb-147">[Tamanhos de VM para uso geral][compute-general] Tamanhos de VM para uso geral também estão disponíveis e oferecem uma proporção balanceada de CPU e memória.</span><span class="sxs-lookup"><span data-stu-id="188bb-147">[General purposes VM sizes][compute-general] General purpose VM sizes are also available and provide balanced CPU-to-memory ratio.</span></span>

### <a name="alternatives"></a><span data-ttu-id="188bb-148">Alternativas</span><span class="sxs-lookup"><span data-stu-id="188bb-148">Alternatives</span></span>

<span data-ttu-id="188bb-149">Se você precisar de mais controle sobre seu ambiente de renderização no Azure ou precisar de uma implementação híbrida, então a computação CycleCloud pode ajudar a coordenar uma grade de IaaS na nuvem.</span><span class="sxs-lookup"><span data-stu-id="188bb-149">If you require more control over your rendering environment in Azure or need a hybrid implementation, then CycleCloud computing can help orchestrate an IaaS grid in the cloud.</span></span> <span data-ttu-id="188bb-150">Usando as mesmas tecnologias subjacentes do Azure como Lote do Azure, ele torna a criação e manutenção de uma grade de IaaS um processo eficiente.</span><span class="sxs-lookup"><span data-stu-id="188bb-150">Using the same underlying Azure technologies as Azure Batch, it makes building and maintaining an IaaS grid an efficient process.</span></span> <span data-ttu-id="188bb-151">Para obter mais informações e saber mais sobre os princípios de design, use o seguinte link:</span><span class="sxs-lookup"><span data-stu-id="188bb-151">To find out more and learn about the design principles, please use the following link:</span></span>

<span data-ttu-id="188bb-152">Para obter uma visão geral completa de todas as soluções HPC que estão disponíveis para você no Azure, consulte o artigo [Soluções de HPC, Lote e Big Compute usando VMs do Azure][hpc-alt-solutions]</span><span class="sxs-lookup"><span data-stu-id="188bb-152">For a complete overview of all the HPC solutions that are available to you in Azure, please see the article [HPC, Batch, and Big Compute solutions using Azure VMs][hpc-alt-solutions]</span></span>

### <a name="availability"></a><span data-ttu-id="188bb-153">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="188bb-153">Availability</span></span>

<span data-ttu-id="188bb-154">O monitoramento dos componentes do Lote do Azure está disponível por meio de vários serviços, ferramentas e APIs.</span><span class="sxs-lookup"><span data-stu-id="188bb-154">Monitoring of the Azure Batch components is available through a range of services, tools and APIs.</span></span> <span data-ttu-id="188bb-155">Isso é discutido mais detalhadamente no artigo [Monitorar soluções do Lote][batch-monitor].</span><span class="sxs-lookup"><span data-stu-id="188bb-155">This is discussed further in the [Monitor Batch solutions][batch-monitor] article.</span></span>

### <a name="scalability"></a><span data-ttu-id="188bb-156">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="188bb-156">Scalability</span></span>

<span data-ttu-id="188bb-157">Pools dentro de uma conta do Lote do Azure podem ser dimensionadas por meio de intervenção manual ou, usando uma fórmula com base em métricas do Lote do Azure, serem dimensionadas automaticamente.</span><span class="sxs-lookup"><span data-stu-id="188bb-157">Pools within an Azure Batch account can either scale through manual intervention or, by using an formula based on Azure Batch metrics, be scaled automatically.</span></span> <span data-ttu-id="188bb-158">Para obter mais informações sobre isso, consulte o artigo [Criar uma fórmula de dimensionamento automático para dimensionar nós em um pool do Lote][batch-scaling].</span><span class="sxs-lookup"><span data-stu-id="188bb-158">For more information on this, please see the article [Create an automatic scaling formula for scaling nodes in a Batch pool][batch-scaling].</span></span>

### <a name="security"></a><span data-ttu-id="188bb-159">Segurança</span><span class="sxs-lookup"><span data-stu-id="188bb-159">Security</span></span>

<span data-ttu-id="188bb-160">Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].</span><span class="sxs-lookup"><span data-stu-id="188bb-160">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="188bb-161">Resiliência</span><span class="sxs-lookup"><span data-stu-id="188bb-161">Resiliency</span></span>

<span data-ttu-id="188bb-162">Embora não exista atualmente nenhuma funcionalidade de failover no Lote do Azure, é recomendável usar as seguintes etapas para garantir a disponibilidade em caso de interrupção não planejada:</span><span class="sxs-lookup"><span data-stu-id="188bb-162">While there is currently no failover capability in Azure Batch, we recommend using the following steps to ensure availbility in case of an unplanned outage:</span></span>

* <span data-ttu-id="188bb-163">Criar uma conta do Lote do Azure em um local alternativo do Azure com uma conta de armazenamento alternativa</span><span class="sxs-lookup"><span data-stu-id="188bb-163">Create a Azure Batch account in an alternate Azure location with an alternate Storage Account</span></span>
* <span data-ttu-id="188bb-164">Criar os mesmos pools de nós com o mesmo nome, com 0 nós alocados</span><span class="sxs-lookup"><span data-stu-id="188bb-164">Create the same node pools with the same name, with 0 nodes allocated</span></span>
* <span data-ttu-id="188bb-165">Certificar-se de que os aplicativos são criados e atualizados para a conta de armazenamento alternativa</span><span class="sxs-lookup"><span data-stu-id="188bb-165">Ensure Applications are created and updated to the alternate Storage Account</span></span>
* <span data-ttu-id="188bb-166">Carregar arquivos de entrada e enviar trabalhos para a conta do Lote do Azure alternativa</span><span class="sxs-lookup"><span data-stu-id="188bb-166">Upload input files and submit jobs to the alternate Azure Batch account</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="188bb-167">Implantar este cenário</span><span class="sxs-lookup"><span data-stu-id="188bb-167">Deploy this scenario</span></span>

### <a name="creating-an-azure-batch-account-and-pools-manually"></a><span data-ttu-id="188bb-168">Criar uma conta do Lote do Azure e pools manualmente</span><span class="sxs-lookup"><span data-stu-id="188bb-168">Creating an Azure Batch account and pools manually</span></span>

<span data-ttu-id="188bb-169">Este exemplo de cenário ajudará a aprender como o Lote do Azure funciona durante a apresentação de Laboratórios do Lote do Azure como um exemplo de solução de SaaS que pode ser desenvolvido para seus próprios clientes:</span><span class="sxs-lookup"><span data-stu-id="188bb-169">This sample scenario will provide help in learning how Azure Batch works while showcasing Azure Batch Labs as an example SaaS solution that can be developed for your own customers:</span></span>

<span data-ttu-id="188bb-170">[Masterclass do Lote do Azure][batch-labs-masterclass]</span><span class="sxs-lookup"><span data-stu-id="188bb-170">[Azure Batch Masterclass][batch-labs-masterclass]</span></span>

### <a name="deploying-the-sample-scenario-using-an-azure-resource-manager-arm-template"></a><span data-ttu-id="188bb-171">Implantar o cenário de exemplo usando um modelo do Azure Resource Manager (ARM)</span><span class="sxs-lookup"><span data-stu-id="188bb-171">Deploying the sample scenario using an Azure Resource Manager (ARM) template</span></span>

<span data-ttu-id="188bb-172">O modelo será implantado:</span><span class="sxs-lookup"><span data-stu-id="188bb-172">The template will deploy:</span></span>
  - <span data-ttu-id="188bb-173">Uma nova conta do Lote do Azure</span><span class="sxs-lookup"><span data-stu-id="188bb-173">A new Azure Batch account</span></span>
  - <span data-ttu-id="188bb-174">Uma conta de armazenamento</span><span class="sxs-lookup"><span data-stu-id="188bb-174">A storage account</span></span>
  - <span data-ttu-id="188bb-175">Um pool de nós associado à conta do lote</span><span class="sxs-lookup"><span data-stu-id="188bb-175">A node pool associated with the batch account</span></span>
  - <span data-ttu-id="188bb-176">O pool de nós será configurado para usar VMs de A2 v2 com imagens do Ubuntu da Canonical</span><span class="sxs-lookup"><span data-stu-id="188bb-176">The node pool will be configured to use A2 v2 VMs with Canonical Ubuntu images</span></span>
  - <span data-ttu-id="188bb-177">O pool de nós conterá 0 VMs inicialmente e exigirá dimensionando manual para adicionar máquinas virtuais</span><span class="sxs-lookup"><span data-stu-id="188bb-177">The node pool will contain 0 VMs initially and will require scaling manually to add VMs</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="188bb-178">[Saiba mais sobre modelos de ARM][azure-arm-templates]</span><span class="sxs-lookup"><span data-stu-id="188bb-178">[Learn more about ARM templates][azure-arm-templates]</span></span>

## <a name="pricing"></a><span data-ttu-id="188bb-179">Preços</span><span class="sxs-lookup"><span data-stu-id="188bb-179">Pricing</span></span>

<span data-ttu-id="188bb-180">O custo de usar o Lote do Azure dependerá dos tamanhos de VM que são usados para os pools e quanto tempo eles ficam alocados e em execução, não há nenhum custo associado a uma criação de conta do Lote do Azure.</span><span class="sxs-lookup"><span data-stu-id="188bb-180">The cost of using Azure Batch will depend on the VM sizes that are used for the pools and how long these are allocated and running, there is no cost associated with an Azure Batch account creation.</span></span> <span data-ttu-id="188bb-181">A saída de dados e armazenamento também deve ser levada em conta já elas implicarão em custos adicionais.</span><span class="sxs-lookup"><span data-stu-id="188bb-181">Storage and data egress should also be taken into account as these will apply additional costs.</span></span>

<span data-ttu-id="188bb-182">A seguir estão exemplos de custos que poderiam ser cobrados para um trabalho que termina em 8 horas usando um número diferente de servidores:</span><span class="sxs-lookup"><span data-stu-id="188bb-182">The following are examples of costs that could be incurred for a job that completes in 8 hours using a different number of servers:</span></span>


- <span data-ttu-id="188bb-183">100 VMs de CPU de alto desempenho: [Estimativa de custo][hpc-est-high]</span><span class="sxs-lookup"><span data-stu-id="188bb-183">100 High Performance CPU VMs: [Cost Estimate][hpc-est-high]</span></span>

  <span data-ttu-id="188bb-184">100 x H16m (16 núcleos, 225 GB de RAM, armazenamento Premium de 512 GB), armazenamento de Blobs de 2 TB, saída de 1 TB</span><span class="sxs-lookup"><span data-stu-id="188bb-184">100 x H16m (16 cores, 225GB RAM, Premium Storage 512GB), 2 TB Blob Storage, 1 TB egress</span></span>

- <span data-ttu-id="188bb-185">50 VMs de CPU de alto desempenho: [Estimativa de custo][hpc-est-med]</span><span class="sxs-lookup"><span data-stu-id="188bb-185">50 High Performance CPU VMs: [Cost Estimate][hpc-est-med]</span></span>

  <span data-ttu-id="188bb-186">50 x H16m (16 núcleos, 225 GB de RAM, armazenamento Premium de 512 GB), armazenamento de Blobs de 2 TB, saída de 1 TB</span><span class="sxs-lookup"><span data-stu-id="188bb-186">50 x H16m (16 cores, 225GB RAM, Premium Storage 512GB), 2 TB Blob Storage, 1 TB egress</span></span>

- <span data-ttu-id="188bb-187">10 VMs de CPU de alto desempenho: [Estimativa de custo][hpc-est-low]</span><span class="sxs-lookup"><span data-stu-id="188bb-187">10 High Performance CPU VMs: [Cost Estimate][hpc-est-low]</span></span>
  
  <span data-ttu-id="188bb-188">10 x H16m (16 núcleos, 225 GB de RAM, armazenamento Premium de 512 GB), armazenamento de Blobs de 2 TB, saída de 1 TB</span><span class="sxs-lookup"><span data-stu-id="188bb-188">10 x H16m (16 cores, 225GB RAM, Premium Storage 512GB), 2 TB Blob Storage, 1 TB egress</span></span>

### <a name="low-priority-vm-pricing"></a><span data-ttu-id="188bb-189">Preços de VM de baixa prioridade</span><span class="sxs-lookup"><span data-stu-id="188bb-189">Low Priority VM Pricing</span></span>

<span data-ttu-id="188bb-190">O Lote do Azure também oferece suporte ao uso de VMs de baixa prioridade\* nos pools de nó, que potencialmente podem fornecer uma economia significativa de custos.</span><span class="sxs-lookup"><span data-stu-id="188bb-190">Azure Batch also supports the use of Low Priority VMs\* in the node pools, which can potentially provide a substantial cost saving.</span></span> <span data-ttu-id="188bb-191">Para obter uma comparação de preços entre as VMs padrão e as VMs de baixa prioridade e para obter mais informações sobre as VMs de baixa prioridade, consulte [Preços do Lote][batch-pricing].</span><span class="sxs-lookup"><span data-stu-id="188bb-191">For a price comparison between standard VMs and Low Priority VMs, and to find out more about Low Priority VMs, please see [Batch Pricing][batch-pricing].</span></span>

<span data-ttu-id="188bb-192">\* Observe que somente determinados aplicativos e cargas de trabalho serão adequados para execução em VMs de baixa prioridade.</span><span class="sxs-lookup"><span data-stu-id="188bb-192">\* Please note that only certain applications and workloads will be suitable to run on Low Priority VMs.</span></span>

## <a name="related-resources"></a><span data-ttu-id="188bb-193">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="188bb-193">Related Resources</span></span>

<span data-ttu-id="188bb-194">[Visão Geral do Lote do Azure][batch-overview]</span><span class="sxs-lookup"><span data-stu-id="188bb-194">[Azure Batch Overview][batch-overview]</span></span>

<span data-ttu-id="188bb-195">[Documentação do Lote do Azure][batch-doc]</span><span class="sxs-lookup"><span data-stu-id="188bb-195">[Azure Batch Documentation][batch-doc]</span></span>

<span data-ttu-id="188bb-196">[Usar contêineres no Lote do Azure][batch-containers]</span><span class="sxs-lookup"><span data-stu-id="188bb-196">[Using containers on Azure Batch][batch-containers]</span></span>

<!-- links -->
[architecture]: ./media/native-hpc-ref-arch.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[vnet]: /azure/virtual-network/virtual-networks-overview
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
[batch-pricing]: https://azure.microsoft.com/en-gb/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job
