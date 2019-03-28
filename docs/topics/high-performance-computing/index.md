---
title: HPC (Computação de alto desempenho) no Azure
description: Um guia para compilar cargas de trabalho de HPC em execução no Azure
author: adamboeglin
ms.date: 2/4/2019
---
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD026 -->

# <a name="high-performance-computing-hpc-on-azure"></a><span data-ttu-id="45c9d-103">HPC (Computação de alto desempenho) no Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-103">High Performance Computing (HPC) on Azure</span></span>

## <a name="introduction-to-hpc"></a><span data-ttu-id="45c9d-104">Introdução à HPC</span><span class="sxs-lookup"><span data-stu-id="45c9d-104">Introduction to HPC</span></span>

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.youtube.com/embed/rKURT32faJk]

<!-- markdownlint-enable MD034 -->

<span data-ttu-id="45c9d-105">A HPC (Computação de alto desempenho), também chamada de “Computação intensa”, usa um grande número de computadores baseados em CPU ou GPU para solucionar tarefas matemáticas complexas.</span><span class="sxs-lookup"><span data-stu-id="45c9d-105">High Performance Computing (HPC), also called "Big Compute", uses a large number of CPU or GPU-based computers to solve complex mathematical tasks.</span></span>

<span data-ttu-id="45c9d-106">Vários setores usam a HPC para solucionar alguns de seus problemas mais difíceis.</span><span class="sxs-lookup"><span data-stu-id="45c9d-106">Many industries use HPC to solve some of their most difficult problems.</span></span>  <span data-ttu-id="45c9d-107">Esses problemas incluem cargas de trabalho como:</span><span class="sxs-lookup"><span data-stu-id="45c9d-107">These include workloads such as:</span></span>

- <span data-ttu-id="45c9d-108">Genomics</span><span class="sxs-lookup"><span data-stu-id="45c9d-108">Genomics</span></span>
- <span data-ttu-id="45c9d-109">Simulações de petróleo e gás</span><span class="sxs-lookup"><span data-stu-id="45c9d-109">Oil and gas simulations</span></span>
- <span data-ttu-id="45c9d-110">Finanças</span><span class="sxs-lookup"><span data-stu-id="45c9d-110">Finance</span></span>
- <span data-ttu-id="45c9d-111">Design de semicondutores</span><span class="sxs-lookup"><span data-stu-id="45c9d-111">Semiconductor design</span></span>
- <span data-ttu-id="45c9d-112">Engenharia</span><span class="sxs-lookup"><span data-stu-id="45c9d-112">Engineering</span></span>
- <span data-ttu-id="45c9d-113">Modelagem climática</span><span class="sxs-lookup"><span data-stu-id="45c9d-113">Weather modeling</span></span>

### <a name="how-is-hpc-different-on-the-cloud"></a><span data-ttu-id="45c9d-114">Qual a diferença entre a HPC e a nuvem?</span><span class="sxs-lookup"><span data-stu-id="45c9d-114">How is HPC different on the cloud?</span></span>

<span data-ttu-id="45c9d-115">Uma das principais diferenças entre o sistema de HPC local e um na nuvem é a capacidade de adicionar e remover recursos dinamicamente, conforme necessário.</span><span class="sxs-lookup"><span data-stu-id="45c9d-115">One of the primary differences between an on-premise HPC system and one in the cloud is the ability for resources to dynamically be added and removed as they're needed.</span></span>  <span data-ttu-id="45c9d-116">O escalonamento dinâmico remove o gargalo da capacidade de computação e permite que os clientes dimensionem corretamente a infraestrutura, de acordo com os requisitos de seus trabalhos.</span><span class="sxs-lookup"><span data-stu-id="45c9d-116">Dynamic scaling removes compute capacity as a bottleneck and instead allow customers to right size their infrastructure for the requirements of their jobs.</span></span>

<span data-ttu-id="45c9d-117">Os artigos a seguir fornecem mais detalhes sobre essa funcionalidade de dimensionamento dinâmico.</span><span class="sxs-lookup"><span data-stu-id="45c9d-117">The following articles provide more detail about this dynamic scaling capability.</span></span>

- [<span data-ttu-id="45c9d-118">Estilo de arquitetura de computação intensa</span><span class="sxs-lookup"><span data-stu-id="45c9d-118">Big Compute Architecture Style</span></span>](/azure/architecture/guide/architecture-styles/big-compute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="45c9d-119">Práticas recomendadas de dimensionamento automático</span><span class="sxs-lookup"><span data-stu-id="45c9d-119">Autoscaling best practices</span></span>](/azure/architecture/best-practices/auto-scaling?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="implementation-checklist"></a><span data-ttu-id="45c9d-120">Lista de verificação de implementação</span><span class="sxs-lookup"><span data-stu-id="45c9d-120">Implementation checklist</span></span>

<span data-ttu-id="45c9d-121">Se for implementar sua própria solução de HPC no Azure, consulte os seguintes tópicos:</span><span class="sxs-lookup"><span data-stu-id="45c9d-121">As you're looking to implement your own HPC solution on Azure, ensure you're reviewed the following topics:</span></span>

<!-- markdownlint-disable MD032 -->

> [!div class="checklist"]
> - <span data-ttu-id="45c9d-122">Escolher a [arquitetura](#infrastructure) apropriada com base nos requisitos</span><span class="sxs-lookup"><span data-stu-id="45c9d-122">Choose the appropriate [architecture](#infrastructure) based on your requirements</span></span>
> - <span data-ttu-id="45c9d-123">Saber quais opções de [computação](#compute) são as ideais para sua carga de trabalho</span><span class="sxs-lookup"><span data-stu-id="45c9d-123">Know which [compute](#compute) options is right for your workload</span></span>
> - <span data-ttu-id="45c9d-124">Identificar a solução de [armazenamento](#storage) ideal que atenda às suas necessidades</span><span class="sxs-lookup"><span data-stu-id="45c9d-124">Identify the right [storage](#storage) solution that meets your needs</span></span>
> - <span data-ttu-id="45c9d-125">Decidir como [gerenciará](#management) todos os seus recursos</span><span class="sxs-lookup"><span data-stu-id="45c9d-125">Decide how you're going to [manage](#management) all your resources</span></span>
> - <span data-ttu-id="45c9d-126">Otimizar seu [aplicativo](#hpc-applications) para a nuvem</span><span class="sxs-lookup"><span data-stu-id="45c9d-126">Optimize your [application](#hpc-applications) for the cloud</span></span>
> - <span data-ttu-id="45c9d-127">[Proteger](#security) sua infraestrutura</span><span class="sxs-lookup"><span data-stu-id="45c9d-127">[Secure](#security) your Infrastructure</span></span>

<!-- markdownlint-enable MD032 -->

## <a name="infrastructure"></a><span data-ttu-id="45c9d-128">Infraestrutura</span><span class="sxs-lookup"><span data-stu-id="45c9d-128">Infrastructure</span></span>

<span data-ttu-id="45c9d-129">São necessários alguns componentes de infraestrutura para compilar um sistema de HPC.</span><span class="sxs-lookup"><span data-stu-id="45c9d-129">There are a number of infrastructure components necessary to build an HPC system.</span></span>  <span data-ttu-id="45c9d-130">Computação, armazenamento e rede são os componentes básicos, independentemente de como escolha gerenciar as cargas de trabalho da HPC.</span><span class="sxs-lookup"><span data-stu-id="45c9d-130">Compute, Storage, and Networking provide the underlying components, no matter how you choose to manage your HPC workloads.</span></span>

### <a name="example-hpc-architectures"></a><span data-ttu-id="45c9d-131">Exemplo de arquiteturas de HPC</span><span class="sxs-lookup"><span data-stu-id="45c9d-131">Example HPC architectures</span></span>

<span data-ttu-id="45c9d-132">São várias as maneiras de projetar e implementar a arquitetura de HPC no Azure.</span><span class="sxs-lookup"><span data-stu-id="45c9d-132">There are a number of different ways to design and implement your HPC architecture on Azure.</span></span>  <span data-ttu-id="45c9d-133">Os aplicativos de HPC podem dimensionar até milhares de núcleos de computação, estender clusters locais ou executar solução nativa 100% em nuvem.</span><span class="sxs-lookup"><span data-stu-id="45c9d-133">HPC applications can scale to thousands of compute cores, extend on-premises clusters, or run as a 100% cloud native solution.</span></span>

<span data-ttu-id="45c9d-134">Os cenários a seguir descrevem algumas maneiras comuns de compilação das soluções de HPC.</span><span class="sxs-lookup"><span data-stu-id="45c9d-134">The following scenarios outline a few of the common ways HPC solutions are built.</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/apps/hpc-saas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/apps/media/architecture-hpc-saas.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="45c9d-135">Serviços de engenharia auxiliados por computador no Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-135">Computer-aided engineering services on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="45c9d-136">Fornece uma plataforma de software como um serviço (SaaS) para a engenharia auxiliada por computador (CAE) no Azure.</span><span class="sxs-lookup"><span data-stu-id="45c9d-136">Provide a software-as-a-service (SaaS) platform for computer-aided engineering (CAE) on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/hpc-cfd?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-hpc-cfd.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="45c9d-137">CFD (Simulações de fluidodinâmica computacional) no Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-137">Computational fluid dynamics (CFD) simulations on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="45c9d-138">Execute a dinâmica dos fluidos computacional (CFD) no Azure.</span><span class="sxs-lookup"><span data-stu-id="45c9d-138">Execute computational fluid dynamics (CFD) simulations on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/video-rendering?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-video-rendering.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="45c9d-139">Renderização de vídeo em 3D no Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-139">3D video rendering on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="45c9d-140">Executar cargas de trabalho de HPC nativas no Azure usando o serviço de Lote do Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-140">Run native HPC workloads in Azure using the Azure Batch service</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

### <a name="compute"></a><span data-ttu-id="45c9d-141">Computação</span><span class="sxs-lookup"><span data-stu-id="45c9d-141">Compute</span></span>

<span data-ttu-id="45c9d-142">O Azure oferece uma variedade de tamanhos otimizados para cargas de trabalho com uso intensivo de CPU e GPU.</span><span class="sxs-lookup"><span data-stu-id="45c9d-142">Azure offers a range of sizes that are optimized for both CPU & GPU intensive workloads.</span></span>

#### <a name="cpu-based-virtual-machines"></a><span data-ttu-id="45c9d-143">Máquinas virtuais baseadas em CPU</span><span class="sxs-lookup"><span data-stu-id="45c9d-143">CPU-based virtual machines</span></span>

- [<span data-ttu-id="45c9d-144">VMs do Linux</span><span class="sxs-lookup"><span data-stu-id="45c9d-144">Linux VMs</span></span>](/azure/virtual-machines/linux/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- <span data-ttu-id="45c9d-145">VMs das [VMs do Windows](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)</span><span class="sxs-lookup"><span data-stu-id="45c9d-145">[Windows VM's](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) VMs</span></span>
  
#### <a name="gpu-enabled-virtual-machines"></a><span data-ttu-id="45c9d-146">Máquinas virtuais habilitadas para GPU</span><span class="sxs-lookup"><span data-stu-id="45c9d-146">GPU-enabled virtual machines</span></span>

<span data-ttu-id="45c9d-147">As VMs da série N têm GPUs NVIDIA projetados para uso de computação intensa ou gráficos intensivos, incluindo visualização e aprendizagem de IA (inteligência artificial).</span><span class="sxs-lookup"><span data-stu-id="45c9d-147">N-series VMs feature NVIDIA GPUs designed for compute-intensive or graphics-intensive applications including artificial intelligence (AI) learning and visualization.</span></span>

- [<span data-ttu-id="45c9d-148">VMs do Linux</span><span class="sxs-lookup"><span data-stu-id="45c9d-148">Linux VMs</span></span>](/azure/virtual-machines/linux/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="45c9d-149">VMs do Windows</span><span class="sxs-lookup"><span data-stu-id="45c9d-149">Windows VMs</span></span>](/azure/virtual-machines/windows/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

### <a name="storage"></a><span data-ttu-id="45c9d-150">Armazenamento</span><span class="sxs-lookup"><span data-stu-id="45c9d-150">Storage</span></span>

<span data-ttu-id="45c9d-151">Cargas de trabalho do Lote e de HPC em larga escala têm exigências de armazenamento e acesso a dados que excedem os recursos de sistemas de arquivos em nuvem tradicionais.</span><span class="sxs-lookup"><span data-stu-id="45c9d-151">Large-scale Batch and HPC workloads have demands for data storage and access that exceed the capabilities of traditional cloud file systems.</span></span>  <span data-ttu-id="45c9d-152">Existem algumas soluções para gerenciar as necessidades de velocidade e capacidade dos aplicativos de HPC no Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-152">There are a number of solutions to manage both the speed and capacity needs of HPC applications on Azure</span></span>

- <span data-ttu-id="45c9d-153">[Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/), para armazenamento de dados mais rápido e acessível para computação de alto desempenho na borda</span><span class="sxs-lookup"><span data-stu-id="45c9d-153">[Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/) for faster, more accessible data storage for high-performance computing at the edge</span></span>
- [<span data-ttu-id="45c9d-154">BeeGFS</span><span class="sxs-lookup"><span data-stu-id="45c9d-154">BeeGFS</span></span>](https://azure.microsoft.com/resources/implement-glusterfs-on-azure/)
- [<span data-ttu-id="45c9d-155">Máquinas virtuais otimizadas para armazenamento</span><span class="sxs-lookup"><span data-stu-id="45c9d-155">Storage Optimized Virtual Machines</span></span>](/azure/virtual-machines/windows/sizes-storage?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="45c9d-156">Armazenamento de blobs, tabelas e filas</span><span class="sxs-lookup"><span data-stu-id="45c9d-156">Blob, table, and queue storage</span></span>](/azure/storage/storage-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="45c9d-157">Armazenamento de arquivos SMB do Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-157">Azure SMB File storage</span></span>](/azure/storage/files/storage-files-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="45c9d-158">Intel Cloud Edition Lustre</span><span class="sxs-lookup"><span data-stu-id="45c9d-158">Intel Cloud Edition Lustre</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/intel.intel-cloud-edition-gs)

<span data-ttu-id="45c9d-159">Para obter mais informações sobre a comparação de Lustre, GlusterFS e BeeGFS no Azure, consulte o [livro eletrônico sobre Sistemas de arquivos paralelos no Azure](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/).</span><span class="sxs-lookup"><span data-stu-id="45c9d-159">For more information comparing Lustre, GlusterFS, and BeeGFS on Azure, review the [Parallel Files Systems on Azure eBook](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/)</span></span>

### <a name="networking"></a><span data-ttu-id="45c9d-160">Rede</span><span class="sxs-lookup"><span data-stu-id="45c9d-160">Networking</span></span>

<span data-ttu-id="45c9d-161">VMs H16r, H16mr, A8 e A9 podem se conectar a uma rede RDMA de back-end de alta taxa de transferência.</span><span class="sxs-lookup"><span data-stu-id="45c9d-161">H16r, H16mr, A8, and A9 VMs can connect to a high throughput back-end RDMA network.</span></span> <span data-ttu-id="45c9d-162">Essa rede pode melhorar o desempenho de aplicativos paralelos firmemente acoplados em execução no Microsoft MPI ou no Intel MPI.</span><span class="sxs-lookup"><span data-stu-id="45c9d-162">This network can improve the performance of tightly coupled parallel applications running under Microsoft MPI or Intel MPI.</span></span>

- [<span data-ttu-id="45c9d-163">Instâncias compatíveis com RDMA</span><span class="sxs-lookup"><span data-stu-id="45c9d-163">RDMA Capable Instances</span></span>](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context#rdma-capable-instances)
- [<span data-ttu-id="45c9d-164">Rede Virtual</span><span class="sxs-lookup"><span data-stu-id="45c9d-164">Virtual Network</span></span>](/azure/virtual-network/virtual-networks-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="45c9d-165">ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="45c9d-165">ExpressRoute</span></span>](/azure/expressroute/expressroute-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="management"></a><span data-ttu-id="45c9d-166">Gerenciamento</span><span class="sxs-lookup"><span data-stu-id="45c9d-166">Management</span></span>

### <a name="do-it-yourself"></a><span data-ttu-id="45c9d-167">Faça você mesmo</span><span class="sxs-lookup"><span data-stu-id="45c9d-167">Do-it-yourself</span></span>

<span data-ttu-id="45c9d-168">Compilar um sistema HPC do zero no Azure oferece uma quantidade significativa de flexibilidade mas, normalmente, exige muita manutenção.</span><span class="sxs-lookup"><span data-stu-id="45c9d-168">Building an HPC system from scratch on Azure offers a significant amount of flexibility, but is often very maintenance intensive.</span></span>  

1. <span data-ttu-id="45c9d-169">Configurar seu próprio ambiente de cluster em máquinas virtuais do Azure ou [conjuntos de escala de máquina virtual](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span><span class="sxs-lookup"><span data-stu-id="45c9d-169">Set up your own cluster environment in Azure virtual machines or [virtual machine scale sets](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span></span>
2. <span data-ttu-id="45c9d-170">Use modelos do Azure Resource Manager para implantar [gerenciadores de carga de trabalho](#workload-managers), infraestrutura, e [aplicativos](#hpc-applications) iniciais.</span><span class="sxs-lookup"><span data-stu-id="45c9d-170">Use Azure Resource Manager templates to deploy leading [workload managers](#workload-managers), infrastructure, and [applications](#hpc-applications).</span></span>
3. <span data-ttu-id="45c9d-171">Escolha [tamanhos de VM de GPU e HPC](#hpc-and-gpu-sizes) que incluem conexões de rede e de hardware especializados para cargas de trabalho MPI ou GPU.</span><span class="sxs-lookup"><span data-stu-id="45c9d-171">Choose [HPC and GPU VM sizes](#hpc-and-gpu-sizes) that include specialized hardware and network connections for MPI or GPU workloads.</span></span>
4. <span data-ttu-id="45c9d-172">Adicione [armazenamento de alto desempenho](#hpc-storage) para cargas de trabalho com E/S alta.</span><span class="sxs-lookup"><span data-stu-id="45c9d-172">Add [high performance storage](#hpc-storage) for I/O-intensive workloads.</span></span>

### <a name="hybrid-and-cloud-bursting"></a><span data-ttu-id="45c9d-173">Intermitência híbrida e de nuvem</span><span class="sxs-lookup"><span data-stu-id="45c9d-173">Hybrid and cloud Bursting</span></span>

<span data-ttu-id="45c9d-174">Se você tem um sistema HPC local que deseja conectar ao Azure, há vários recursos para ajudá-lo a começar.</span><span class="sxs-lookup"><span data-stu-id="45c9d-174">If you have an existing on-premise HPC system that you'd like to connect to Azure, there are a number of resources to help get you started.</span></span>

<span data-ttu-id="45c9d-175">Primeiramente, leia o artigo [Opções de conexão de uma rede local ao Azure](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context), na documentação.</span><span class="sxs-lookup"><span data-stu-id="45c9d-175">First, review the [Options for connecting an on-premises network to Azure](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) article in the documentation.</span></span>  <span data-ttu-id="45c9d-176">A partir daí, talvez seja conveniente obter informações sobre estas opções de conectividade:</span><span class="sxs-lookup"><span data-stu-id="45c9d-176">From there, you may want information on these connectivity options:</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/vpn?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/vpn.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="45c9d-177">Conectar uma rede local ao Azure usando um Gateway de VPN</span><span class="sxs-lookup"><span data-stu-id="45c9d-177">Connect an on-premises network to Azure using a VPN gateway</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="45c9d-178">Essa arquitetura de referência mostra como estender uma rede local para o Azure, usando uma VPN (rede virtual privada) site a site.</span><span class="sxs-lookup"><span data-stu-id="45c9d-178">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="45c9d-179">Conectar uma rede local ao Azure usando o ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="45c9d-179">Connect an on-premises network to Azure using ExpressRoute</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="45c9d-180">Conexões do ExpressRoute usam uma conexão privada dedicada por meio de um provedor de conectividade de terceiros.</span><span class="sxs-lookup"><span data-stu-id="45c9d-180">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="45c9d-181">A conexão privada estende sua rede local para o Azure.</span><span class="sxs-lookup"><span data-stu-id="45c9d-181">The private connection extends your on-premises network into Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute-vpn-failover?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute-vpn-failover.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="45c9d-182">Conectar uma rede local ao Azure usando o ExpressRoute com failover de VPN</span><span class="sxs-lookup"><span data-stu-id="45c9d-182">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="45c9d-183">Implemente uma arquitetura de rede site a site segura e altamente disponível que abranja uma rede virtual do Azure e uma rede local conectada usando o ExpressRoute com failover de gateway VPN.</span><span class="sxs-lookup"><span data-stu-id="45c9d-183">Implement a highly available and secure site-to-site network architecture that spans an Azure virtual network and an on-premises network connected using ExpressRoute with VPN gateway failover.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

<span data-ttu-id="45c9d-184">Quando a conectividade de rede é estabelecida com segurança, você pode começar a usar os recursos de computação na nuvem sob demanda com as funcionalidades de intermitência do seu atual [gerenciador de carga de trabalho](#workload-manager).</span><span class="sxs-lookup"><span data-stu-id="45c9d-184">Once network connectivity is securely established, you can start using cloud compute resources on-demand with the bursting capabilities of your existing [workload manager](#workload-manager).</span></span>

### <a name="marketplace-solutions"></a><span data-ttu-id="45c9d-185">Soluções do Marketplace</span><span class="sxs-lookup"><span data-stu-id="45c9d-185">Marketplace solutions</span></span>

<span data-ttu-id="45c9d-186">O [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/) oferece alguns gerenciadores de carga de trabalho.</span><span class="sxs-lookup"><span data-stu-id="45c9d-186">There are a number of workload managers offered in the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/).</span></span>

- [<span data-ttu-id="45c9d-187">HPC baseada em CentOS RogueWave</span><span class="sxs-lookup"><span data-stu-id="45c9d-187">RogueWave CentOS-based HPC</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/RogueWave.CentOSbased73HPC?tab=Overview)
- [<span data-ttu-id="45c9d-188">SUSE Linux Enterprise Server para HPC</span><span class="sxs-lookup"><span data-stu-id="45c9d-188">SUSE Linux Enterprise Server for HPC</span></span>](https://azure.microsoft.com/marketplace/partners/suse/suselinuxenterpriseserver12optimizedforhighperformancecompute/)
- [<span data-ttu-id="45c9d-189">Mecanismo de servidor de grade TIBCO</span><span class="sxs-lookup"><span data-stu-id="45c9d-189">TIBCO Grid Server Engine</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/tibco-software.gridserverlinuxengine?tab=Overview)
- [<span data-ttu-id="45c9d-190">VM de Ciência de Dados do Azure para Windows e Linux</span><span class="sxs-lookup"><span data-stu-id="45c9d-190">Azure Data Science VM for Windows and Linux</span></span>](/azure/machine-learning/data-science-virtual-machine/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="45c9d-191">D3View</span><span class="sxs-lookup"><span data-stu-id="45c9d-191">D3View</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/xfinityinc.d3view-v5?tab=Overview)
- [<span data-ttu-id="45c9d-192">UberCloud</span><span class="sxs-lookup"><span data-stu-id="45c9d-192">UberCloud</span></span>](https://azure.microsoft.com/search/marketplace/?q=ubercloud)

### <a name="azure-batch"></a><span data-ttu-id="45c9d-193">Lote do Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-193">Azure Batch</span></span>

<span data-ttu-id="45c9d-194">O [Lote do Azure](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) é um serviço de plataforma para execução de aplicativos paralelos em grande escala e aplicativos HPC (computação de alto desempenho) com eficiência na nuvem.</span><span class="sxs-lookup"><span data-stu-id="45c9d-194">[Azure Batch](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) is a platform service for running large-scale parallel and high-performance computing (HPC) applications efficiently in the cloud.</span></span> <span data-ttu-id="45c9d-195">O Lote do Azure agenda o trabalho de computação intensiva para execução em um pool gerenciado de máquinas virtuais e pode dimensionar automaticamente os recursos de computação para atender às necessidades de seus trabalhos.</span><span class="sxs-lookup"><span data-stu-id="45c9d-195">Azure Batch schedules compute-intensive work to run on a managed pool of virtual machines, and can automatically scale compute resources to meet the needs of your jobs.</span></span>

<span data-ttu-id="45c9d-196">Os provedores ou desenvolvedores SaaS podem usar os SDKs e as ferramentas do Lote para integrar aplicativos ou cargas de trabalho de contêiner de HPC ao Azure, transferir dados para o Azure e criar pipelines de execução do trabalho.</span><span class="sxs-lookup"><span data-stu-id="45c9d-196">SaaS providers or developers can use the Batch SDKs and tools to integrate HPC applications or container workloads with Azure, stage data to Azure, and build job execution pipelines.</span></span>

### <a name="azure-cyclecloud"></a><span data-ttu-id="45c9d-197">Azure CycleCloud</span><span class="sxs-lookup"><span data-stu-id="45c9d-197">Azure CycleCloud</span></span>

<span data-ttu-id="45c9d-198">[Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) Fornece a maneira mais fácil de gerenciar cargas de trabalho de HPC usando qualquer agendador (como Slurm, Grid Engine, HPC Pack, HTCondor, LSF, PBS Pro ou Symphony), no Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-198">[Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) Provides the simplest way to manage HPC workloads using any scheduler (like Slurm, Grid Engine, HPC Pack, HTCondor, LSF, PBS Pro, or Symphony), on Azure</span></span>

<span data-ttu-id="45c9d-199">CycleCloud permite que você:</span><span class="sxs-lookup"><span data-stu-id="45c9d-199">CycleCloud allows you to:</span></span>

- <span data-ttu-id="45c9d-200">Implante clusters completos e outros recursos, incluindo agendador, VMs de computação, armazenamento, rede e cache.</span><span class="sxs-lookup"><span data-stu-id="45c9d-200">Deploy full clusters and other resources, including scheduler, compute VMs, storage, networking, and cache</span></span>
- <span data-ttu-id="45c9d-201">Organize fluxos de trabalho de dados, de nuvem e de tarefas.</span><span class="sxs-lookup"><span data-stu-id="45c9d-201">Orchestrate job, data, and cloud workflows</span></span>
- <span data-ttu-id="45c9d-202">Dê aos administradores controle total sobre quais usuários podem executar trabalhos, onde podem fazer isso e a que custo.</span><span class="sxs-lookup"><span data-stu-id="45c9d-202">Give admins full control over which users can run jobs, as well as where and at what cost</span></span>
- <span data-ttu-id="45c9d-203">Personalize e otimize clusters por meio de recursos avançados de política e governança, incluindo controles de custo, integração ao Active Directory, monitoramento e relatórios.</span><span class="sxs-lookup"><span data-stu-id="45c9d-203">Customize and optimize clusters through advanced policy and governance features, including cost controls, Active Directory integration, monitoring, and reporting</span></span>
- <span data-ttu-id="45c9d-204">Use seus aplicativos e o agendador de trabalhos atuais sem modificação.</span><span class="sxs-lookup"><span data-stu-id="45c9d-204">Use your current job scheduler and applications without modification</span></span>
- <span data-ttu-id="45c9d-205">Aproveite as arquiteturas de referência rigorosamente testadas e o dimensionamento automático interno para uma ampla variedade de setores e cargas de trabalho de HPC.</span><span class="sxs-lookup"><span data-stu-id="45c9d-205">Take advantage of built-in autoscaling and battle-tested reference architectures for a wide range of HPC workloads and industries</span></span>

### <a name="workload-managers"></a><span data-ttu-id="45c9d-206">Gerenciadores de carga de trabalho</span><span class="sxs-lookup"><span data-stu-id="45c9d-206">Workload managers</span></span>

<span data-ttu-id="45c9d-207">A seguir, os exemplos de gerenciadores de cluster e de carga de trabalho que podem ser executados na infraestrutura do Azure.</span><span class="sxs-lookup"><span data-stu-id="45c9d-207">The following are examples of cluster and workload managers that can run in Azure infrastructure.</span></span> <span data-ttu-id="45c9d-208">Crie clusters autônomos em VMs do Azure ou se espalhe para máquinas virtuais do Azure de um cluster local.</span><span class="sxs-lookup"><span data-stu-id="45c9d-208">Create stand-alone clusters in Azure VMs or burst to Azure VMs from an on-premises cluster.</span></span>

- [<span data-ttu-id="45c9d-209">Alces Flight Compute</span><span class="sxs-lookup"><span data-stu-id="45c9d-209">Alces Flight Compute</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/alces-flight-limited.alces-flight-compute-solo?tab=Overview)
- [<span data-ttu-id="45c9d-210">TIBCO DataSynapse GridServer</span><span class="sxs-lookup"><span data-stu-id="45c9d-210">TIBCO DataSynapse GridServer</span></span>](https://azure.microsoft.com/blog/tibco-datasynapse-comes-to-the-azure-marketplace/)
- [<span data-ttu-id="45c9d-211">Bright Cluster Manager</span><span class="sxs-lookup"><span data-stu-id="45c9d-211">Bright Cluster Manager</span></span>](http://www.brightcomputing.com/technology-partners/microsoft)
- [<span data-ttu-id="45c9d-212">IBM Spectrum Symphony e Symphony LSF</span><span class="sxs-lookup"><span data-stu-id="45c9d-212">IBM Spectrum Symphony and Symphony LSF</span></span>](https://azure.microsoft.com/blog/ibm-and-microsoft-azure-support-spectrum-symphony-and-spectrum-lsf/)
- [<span data-ttu-id="45c9d-213">PBS Pro</span><span class="sxs-lookup"><span data-stu-id="45c9d-213">PBS Pro</span></span>](http://pbspro.org)
- [<span data-ttu-id="45c9d-214">Altair</span><span class="sxs-lookup"><span data-stu-id="45c9d-214">Altair</span></span>](http://www.altair.com/)
- [<span data-ttu-id="45c9d-215">Rescale</span><span class="sxs-lookup"><span data-stu-id="45c9d-215">Rescale</span></span>](https://www.rescale.com/azure/)
- [<span data-ttu-id="45c9d-216">Pacote de HPC da Microsoft</span><span class="sxs-lookup"><span data-stu-id="45c9d-216">Microsoft HPC Pack</span></span>](https://technet.microsoft.com/library/mt744885.aspx)
  - [<span data-ttu-id="45c9d-217">Pacote de HPC para Windows</span><span class="sxs-lookup"><span data-stu-id="45c9d-217">HPC Pack for Windows</span></span>](/azure/virtual-machines/windows/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
  - [<span data-ttu-id="45c9d-218">Pacote de HPC para Linux</span><span class="sxs-lookup"><span data-stu-id="45c9d-218">HPC Pack for Linux</span></span>](/azure/virtual-machines/linux/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

#### <a name="containers"></a><span data-ttu-id="45c9d-219">Contêineres</span><span class="sxs-lookup"><span data-stu-id="45c9d-219">Containers</span></span>

<span data-ttu-id="45c9d-220">Contêineres também podem ser usados para gerenciar algumas cargas de trabalho de HPC.</span><span class="sxs-lookup"><span data-stu-id="45c9d-220">Containers can also be used to manage some HPC workloads.</span></span>  <span data-ttu-id="45c9d-221">Serviços como o AKS (Serviço de Kubernetes do Azure) simplificam a implantação de um cluster do Kubernetes gerenciado no Azure.</span><span class="sxs-lookup"><span data-stu-id="45c9d-221">Services like the Azure Kubernetes Service (AKS) makes it simple to deploy a managed Kubernetes cluster in Azure.</span></span>

- [<span data-ttu-id="45c9d-222">AKS (Serviço de Kubernetes do Azure)</span><span class="sxs-lookup"><span data-stu-id="45c9d-222">Azure Kubernetes Service (AKS)</span></span>](/azure/aks/intro-kubernetes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="45c9d-223">Registro de Contêiner</span><span class="sxs-lookup"><span data-stu-id="45c9d-223">Container Registry</span></span>](/azure/container-registry/container-registry-intro?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="cost-management"></a><span data-ttu-id="45c9d-224">Gerenciamento de custo</span><span class="sxs-lookup"><span data-stu-id="45c9d-224">Cost management</span></span>

<span data-ttu-id="45c9d-225">O gerenciamento de custo da HPC no Azure pode ser feito de várias maneiras.</span><span class="sxs-lookup"><span data-stu-id="45c9d-225">Managing your HPC cost on Azure can be done through a few different ways.</span></span>  <span data-ttu-id="45c9d-226">Procure ler as [Opções de compra do Azure](https://azure.microsoft.com/pricing/purchase-options/) para encontrar o método ideal para sua organização.</span><span class="sxs-lookup"><span data-stu-id="45c9d-226">Ensure you've reviewed the [Azure purchasing options](https://azure.microsoft.com/pricing/purchase-options/) to find the method that works best for your organization.</span></span>

<span data-ttu-id="45c9d-227">[VMs de baixa prioridade](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) permitem aproveitar nossa capacidade não utilizada, gerando uma economia significativa.</span><span class="sxs-lookup"><span data-stu-id="45c9d-227">[Low priority VMs](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) allow you to take advantage of our unutilized capacity at a significant cost savings.</span></span>

## <a name="security"></a><span data-ttu-id="45c9d-228">Segurança</span><span class="sxs-lookup"><span data-stu-id="45c9d-228">Security</span></span>

<span data-ttu-id="45c9d-229">Para obter uma visão geral das práticas recomendadas de segurança no Azure, leia a [documentação de segurança do Azure](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span><span class="sxs-lookup"><span data-stu-id="45c9d-229">For an overview of security best practices on Azure, review the [Azure Security Documentation](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span></span>  

<span data-ttu-id="45c9d-230">Além das configurações de rede disponíveis na seção [Intermitência da nuvem](#hybrid-and-cloud-bursting), talvez seja conveniente implementar uma configuração de hub/spoke para isolar seus recursos de computação:</span><span class="sxs-lookup"><span data-stu-id="45c9d-230">In addition to the network configurations available in the [Cloud Bursting](#hybrid-and-cloud-bursting) section, you may want to implement a hub/spoke configuration to isolate your compute resources:</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/hub-spoke.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="45c9d-231">Implementar uma topologia de rede hub-spoke no Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-231">Implement a hub-spoke network topology in Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="45c9d-232">O hub é uma VNet (rede virtual) no Azure que atua como ponto central de conectividade para sua rede local.</span><span class="sxs-lookup"><span data-stu-id="45c9d-232">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="45c9d-233">Os raios são VNets que ponto a ponto com o hub e pode ser usado para isolar as cargas de trabalho.</span><span class="sxs-lookup"><span data-stu-id="45c9d-233">The spokes are VNets that peer with the hub, and can be used to isolate workloads.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/shared-services?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/shared-services.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="45c9d-234">Implementar uma topologia de rede hub-spoke com serviços compartilhados no Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-234">Implement a hub-spoke network topology with shared services in Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="45c9d-235">Essa arquitetura de referência se baseia na arquitetura de referência hub-spoke para incluir serviços compartilhados no hub que possam ser consumidos por todos os spokes.</span><span class="sxs-lookup"><span data-stu-id="45c9d-235">This reference architecture builds on the hub-spoke reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="hpc-applications"></a><span data-ttu-id="45c9d-236">Aplicativos de HPC</span><span class="sxs-lookup"><span data-stu-id="45c9d-236">HPC applications</span></span>

<span data-ttu-id="45c9d-237">Execute aplicativos de HPC comerciais ou personalizados no Azure.</span><span class="sxs-lookup"><span data-stu-id="45c9d-237">Run custom or commercial HPC applications in Azure.</span></span> <span data-ttu-id="45c9d-238">Vários exemplos desta seção são submetidos a benchmark para dimensionar com eficiência em caso de VMs ou núcleos de computação adicionais.</span><span class="sxs-lookup"><span data-stu-id="45c9d-238">Several examples in this section are benchmarked to scale efficiently with additional VMs or compute cores.</span></span> <span data-ttu-id="45c9d-239">Visite o [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace) para obter soluções prontas para uso.</span><span class="sxs-lookup"><span data-stu-id="45c9d-239">Visit the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace) for ready-to-deploy solutions.</span></span>

> [!NOTE]
> <span data-ttu-id="45c9d-240">Verifique com o fornecedor de qualquer aplicativo comercial quanto ao licenciamento ou outras restrições de execução na nuvem.</span><span class="sxs-lookup"><span data-stu-id="45c9d-240">Check with the vendor of any commercial application for licensing or other restrictions for running in the cloud.</span></span> <span data-ttu-id="45c9d-241">Nem todos os fornecedores oferecem licenciamento pré-pago.</span><span class="sxs-lookup"><span data-stu-id="45c9d-241">Not all vendors offer pay-as-you-go licensing.</span></span> <span data-ttu-id="45c9d-242">Você talvez precise de um servidor de licenciamento na nuvem para sua solução, ou de uma conexão com um servidor de licença local.</span><span class="sxs-lookup"><span data-stu-id="45c9d-242">You might need a licensing server in the cloud for your solution, or connect to an on-premises license server.</span></span>

### <a name="engineering-applications"></a><span data-ttu-id="45c9d-243">Aplicativos de engenharia</span><span class="sxs-lookup"><span data-stu-id="45c9d-243">Engineering applications</span></span>

- [<span data-ttu-id="45c9d-244">Altair RADIOSS</span><span class="sxs-lookup"><span data-stu-id="45c9d-244">Altair RADIOSS</span></span>](https://azure.microsoft.com/blog/availability-of-altair-radioss-rdma-on-microsoft-azure/)
- [<span data-ttu-id="45c9d-245">ANSYS CFD</span><span class="sxs-lookup"><span data-stu-id="45c9d-245">ANSYS CFD</span></span>](https://azure.microsoft.com/blog/ansys-cfd-and-microsoft-azure-perform-the-best-hpc-scalability-in-the-cloud/)
- [<span data-ttu-id="45c9d-246">MATLAB Distributed Computing Server</span><span class="sxs-lookup"><span data-stu-id="45c9d-246">MATLAB Distributed Computing Server</span></span>](/azure/virtual-machines/windows/matlab-mdcs-cluster?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="45c9d-247">StarCCM +</span><span class="sxs-lookup"><span data-stu-id="45c9d-247">StarCCM+</span></span>](https://blogs.msdn.microsoft.com/azurecat/2017/07/07/run-star-ccm-in-an-azure-hpc-cluster/)
- [<span data-ttu-id="45c9d-248">OpenFOAM</span><span class="sxs-lookup"><span data-stu-id="45c9d-248">OpenFOAM</span></span>](https://simulation.azure.com/casestudies/Team-182-ABB-UC-Final.pdf)

### <a name="graphics-and-rendering"></a><span data-ttu-id="45c9d-249">Gráficos e renderização</span><span class="sxs-lookup"><span data-stu-id="45c9d-249">Graphics and rendering</span></span>

- <span data-ttu-id="45c9d-250">[Autodesk Maya, 3ds Max e Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) no Lote do Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-250">[Autodesk Maya, 3ds Max, and Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) on Azure Batch</span></span>

### <a name="ai-and-deep-learning"></a><span data-ttu-id="45c9d-251">IA e deep learning</span><span class="sxs-lookup"><span data-stu-id="45c9d-251">AI and deep learning</span></span>

- [<span data-ttu-id="45c9d-252">Microsoft Cognitive Toolkit</span><span class="sxs-lookup"><span data-stu-id="45c9d-252">Microsoft Cognitive Toolkit</span></span>](/cognitive-toolkit/cntk-on-azure)
- [<span data-ttu-id="45c9d-253">VM de deep learning</span><span class="sxs-lookup"><span data-stu-id="45c9d-253">Deep Learning VM</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning)
- [<span data-ttu-id="45c9d-254">Receitas do Shipyard do Lote para deep learning</span><span class="sxs-lookup"><span data-stu-id="45c9d-254">Batch Shipyard recipes for deep learning</span></span>](https://github.com/Azure/batch-shipyard/tree/master/recipes#deeplearning)

### <a name="mpi-providers"></a><span data-ttu-id="45c9d-255">Provedores de MPI</span><span class="sxs-lookup"><span data-stu-id="45c9d-255">MPI Providers</span></span>

- [<span data-ttu-id="45c9d-256">Microsoft MPI</span><span class="sxs-lookup"><span data-stu-id="45c9d-256">Microsoft MPI</span></span>](/message-passing-interface/microsoft-mpi)

## <a name="remote-visualization"></a><span data-ttu-id="45c9d-257">Visualização remota</span><span class="sxs-lookup"><span data-stu-id="45c9d-257">Remote visualization</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/linux-vdi-citrix?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/azure-citrix-sample-diagram.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="45c9d-258">Áreas de trabalho virtuais do Linux com o Citrix</span><span class="sxs-lookup"><span data-stu-id="45c9d-258">Linux virtual desktops with Citrix</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="45c9d-259">Crie um ambiente de VDI para áreas de trabalho do Linux usando o Citrix no Azure.</span><span class="sxs-lookup"><span data-stu-id="45c9d-259">Build a VDI environment for Linux Desktops using Citrix on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="performance-benchmarks"></a><span data-ttu-id="45c9d-260">Parâmetros de comparação de desempenho</span><span class="sxs-lookup"><span data-stu-id="45c9d-260">Performance Benchmarks</span></span>

- [<span data-ttu-id="45c9d-261">Parâmetros de comparação de computação</span><span class="sxs-lookup"><span data-stu-id="45c9d-261">Compute benchmarks</span></span>](/azure/virtual-machines/windows/compute-benchmark-scores?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="customer-stories"></a><span data-ttu-id="45c9d-262">Relatos de clientes</span><span class="sxs-lookup"><span data-stu-id="45c9d-262">Customer stories</span></span>

<span data-ttu-id="45c9d-263">Alguns clientes obtiveram muito sucesso usando o Azure nas próprias cargas de trabalho de HPC.</span><span class="sxs-lookup"><span data-stu-id="45c9d-263">There are a number of customers who have seen great success by using Azure for their HPC workloads.</span></span>  <span data-ttu-id="45c9d-264">Veja alguns desses clientes nos estudos de caso a seguir:</span><span class="sxs-lookup"><span data-stu-id="45c9d-264">You can find a few of these customer case studies below:</span></span>

- [<span data-ttu-id="45c9d-265">ANEO</span><span class="sxs-lookup"><span data-stu-id="45c9d-265">ANEO</span></span>](https://customers.microsoft.com/story/it-provider-finds-highly-scalable-cloud-based-hpc-redu)
- [<span data-ttu-id="45c9d-266">AXA Global P&C</span><span class="sxs-lookup"><span data-stu-id="45c9d-266">AXA Global P&C</span></span>](https://customers.microsoft.com/story/axa-global-p-and-c)
- [<span data-ttu-id="45c9d-267">Axioma</span><span class="sxs-lookup"><span data-stu-id="45c9d-267">Axioma</span></span>](https://customers.microsoft.com/story/axioma-delivers-fintechs-first-born-in-the-cloud-multi-asset-class-enterprise-risk-solution)
- [<span data-ttu-id="45c9d-268">d3View</span><span class="sxs-lookup"><span data-stu-id="45c9d-268">d3View</span></span>](https://customers.microsoft.com/story/big-data-solution-provider-adopts-new-cloud-gains-thou)
- [<span data-ttu-id="45c9d-269">EFS</span><span class="sxs-lookup"><span data-stu-id="45c9d-269">EFS</span></span>](https://customers.microsoft.com/story/efs-professionalservices-azure)
- [<span data-ttu-id="45c9d-270">Hymans Robertson</span><span class="sxs-lookup"><span data-stu-id="45c9d-270">Hymans Robertson</span></span>](https://customers.microsoft.com/story/hymans-robertson)
- [<span data-ttu-id="45c9d-271">MetLife</span><span class="sxs-lookup"><span data-stu-id="45c9d-271">MetLife</span></span>](https://enterprise.microsoft.com/customer-story/industries/insurance/metlife/)
- [<span data-ttu-id="45c9d-272">Microsoft Research</span><span class="sxs-lookup"><span data-stu-id="45c9d-272">Microsoft Research</span></span>](https://customers.microsoft.com/doclink/fast-lmm-and-windows-azure-put-genetics-research-on-fa)
- [<span data-ttu-id="45c9d-273">Milliman</span><span class="sxs-lookup"><span data-stu-id="45c9d-273">Milliman</span></span>](https://customers.microsoft.com/story/actuarial-firm-works-to-transform-insurance-industry-w)
- [<span data-ttu-id="45c9d-274">Mitsubishi UFJ Securities International</span><span class="sxs-lookup"><span data-stu-id="45c9d-274">Mitsubishi UFJ Securities International</span></span>](https://customers.microsoft.com/story/powering-risk-compute-grids-in-the-cloud)
- [<span data-ttu-id="45c9d-275">NeuroInitiative</span><span class="sxs-lookup"><span data-stu-id="45c9d-275">NeuroInitiative</span></span>](https://customers.microsoft.com/story/neuroinitiative-health-provider-azure)
- [<span data-ttu-id="45c9d-276">Schlumberger</span><span class="sxs-lookup"><span data-stu-id="45c9d-276">Schlumberger</span></span>](https://azure.microsoft.com/blog/big-compute-for-large-engineering-simulations)
- [<span data-ttu-id="45c9d-277">Towers Watson</span><span class="sxs-lookup"><span data-stu-id="45c9d-277">Towers Watson</span></span>](https://customers.microsoft.com/story/insurance-tech-provider-delivers-disruptive-solutions)

## <a name="other-important-information"></a><span data-ttu-id="45c9d-278">Outras informações importantes</span><span class="sxs-lookup"><span data-stu-id="45c9d-278">Other important information</span></span>

- <span data-ttu-id="45c9d-279">Verifique se a sua [cota de vCPU](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) foi aumentada antes de tentar executar cargas de trabalho de grande escala.</span><span class="sxs-lookup"><span data-stu-id="45c9d-279">Ensure your [vCPU quota](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) has been increased before attempting to run large-scale workloads.</span></span>

## <a name="next-steps"></a><span data-ttu-id="45c9d-280">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="45c9d-280">Next steps</span></span>

<span data-ttu-id="45c9d-281">Para obter os anúncios mais recentes, confira:</span><span class="sxs-lookup"><span data-stu-id="45c9d-281">For the latest announcements, see:</span></span>

- [<span data-ttu-id="45c9d-282">Blog da equipe do Microsoft HPC e do Lote</span><span class="sxs-lookup"><span data-stu-id="45c9d-282">Microsoft HPC and Batch team blog</span></span>](http://blogs.technet.com/b/windowshpc/)
- <span data-ttu-id="45c9d-283">Visite o [Blog do Azure](https://azure.microsoft.com/blog/tag/hpc/).</span><span class="sxs-lookup"><span data-stu-id="45c9d-283">Visit the [Azure blog](https://azure.microsoft.com/blog/tag/hpc/).</span></span>

### <a name="microsoft-batch-examples"></a><span data-ttu-id="45c9d-284">Exemplos de Lote do Microsoft</span><span class="sxs-lookup"><span data-stu-id="45c9d-284">Microsoft Batch Examples</span></span>

<span data-ttu-id="45c9d-285">Estes tutoriais fornecerão detalhes sobre como executar aplicativos no Microsoft Lote</span><span class="sxs-lookup"><span data-stu-id="45c9d-285">These tutorials will provide you with details on running applications on Microsoft Batch</span></span>

- [<span data-ttu-id="45c9d-286">Introdução ao desenvolvimento com o Lote</span><span class="sxs-lookup"><span data-stu-id="45c9d-286">Get started developing with Batch</span></span>](/azure/batch/quick-run-dotnet?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="45c9d-287">Usar exemplos de código do Lote do Azure</span><span class="sxs-lookup"><span data-stu-id="45c9d-287">Use Azure Batch code samples</span></span>](https://github.com/Azure/azure-batch-samples)
- [<span data-ttu-id="45c9d-288">Usar VMs de baixa prioridade com o Lote</span><span class="sxs-lookup"><span data-stu-id="45c9d-288">Use low-priority VMs with Batch</span></span>](/azure/batch/batch-low-pri-vms?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="45c9d-289">Executar cargas de trabalho de HPC em contêineres com Shipyard do Lote</span><span class="sxs-lookup"><span data-stu-id="45c9d-289">Run containerized HPC workloads with Batch Shipyard</span></span>](https://github.com/Azure/batch-shipyard)
- [<span data-ttu-id="45c9d-290">Executar cargas de trabalho paralelas R em lote</span><span class="sxs-lookup"><span data-stu-id="45c9d-290">Run parallel R workloads on Batch</span></span>](https://github.com/Azure/doAzureParallel)
- [<span data-ttu-id="45c9d-291">Executar trabalhos Spark sob demanda no Lote</span><span class="sxs-lookup"><span data-stu-id="45c9d-291">Run on-demand Spark jobs on Batch</span></span>](https://github.com/Azure/aztk)
- [<span data-ttu-id="45c9d-292">Usar VMs de computação intensa em pools do Lote</span><span class="sxs-lookup"><span data-stu-id="45c9d-292">Use compute-intensive VMs in Batch pools</span></span>](/azure/batch/batch-pool-compute-intensive-sizes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)