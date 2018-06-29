---
title: Aplicativo de n camadas com SQL Server
description: Como implementar uma arquitetura multicamadas no Azure, para obter disponibilidade, segurança, escalabilidade e capacidade de gerenciamento.
author: MikeWasson
ms.date: 06/23/2018
ms.openlocfilehash: 050ea9b3104a2dc9af4cdaad3b4540cd75434e9d
ms.sourcegitcommit: 767c8570d7ab85551c2686c095b39a56d813664b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/24/2018
ms.locfileid: "36746665"
---
# <a name="n-tier-application-with-sql-server"></a><span data-ttu-id="0b641-103">Aplicativo de n camadas com SQL Server</span><span class="sxs-lookup"><span data-stu-id="0b641-103">N-tier application with SQL Server</span></span>

<span data-ttu-id="0b641-104">Essa arquitetura de referência mostra como implantar VMs e uma rede virtual configurada para um aplicativo de N camadas usando o SQL Server no Windows para a camada de dados.</span><span class="sxs-lookup"><span data-stu-id="0b641-104">This reference architecture shows how to deploy VMs and a virtual network configured for an N-tier application, using SQL Server on Windows for the data tier.</span></span> [<span data-ttu-id="0b641-105">**Implante essa solução**.</span><span class="sxs-lookup"><span data-stu-id="0b641-105">**Deploy this solution**.</span></span>](#deploy-the-solution) 

<span data-ttu-id="0b641-106">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="0b641-106">![[0]][0]</span></span>

<span data-ttu-id="0b641-107">*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*</span><span class="sxs-lookup"><span data-stu-id="0b641-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="0b641-108">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="0b641-108">Architecture</span></span> 

<span data-ttu-id="0b641-109">A arquitetura tem os seguintes componentes:</span><span class="sxs-lookup"><span data-stu-id="0b641-109">The architecture has the following components:</span></span>

* <span data-ttu-id="0b641-110">**Grupo de recursos.**</span><span class="sxs-lookup"><span data-stu-id="0b641-110">**Resource group.**</span></span> <span data-ttu-id="0b641-111">[Grupos de recursos][resource-manager-overview] são utilizados para agrupar os recursos para que eles possam ser gerenciados pelo tempo de vida, o proprietário ou outros critérios.</span><span class="sxs-lookup"><span data-stu-id="0b641-111">[Resource groups][resource-manager-overview] are used to group resources so they can be managed by lifetime, owner, or other criteria.</span></span>

* <span data-ttu-id="0b641-112">**Rede Virtual (VNet) e sub-redes.**</span><span class="sxs-lookup"><span data-stu-id="0b641-112">**Virtual network (VNet) and subnets.**</span></span> <span data-ttu-id="0b641-113">Cada VM do Azure é implantada em uma VNet que pode ser segmentada em várias sub-redes.</span><span class="sxs-lookup"><span data-stu-id="0b641-113">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span> <span data-ttu-id="0b641-114">Sempre crie uma sub-rede separada para cada camada.</span><span class="sxs-lookup"><span data-stu-id="0b641-114">Create a separate subnet for each tier.</span></span> 

* <span data-ttu-id="0b641-115">**NSGs.**</span><span class="sxs-lookup"><span data-stu-id="0b641-115">**NSGs.**</span></span> <span data-ttu-id="0b641-116">Use os NSGs [grupos de segurança de rede][nsg] para restringir o tráfego de rede na VNet.</span><span class="sxs-lookup"><span data-stu-id="0b641-116">Use [network security groups][nsg] (NSGs) to restrict network traffic within the VNet.</span></span> <span data-ttu-id="0b641-117">Por exemplo, na arquitetura de três camadas mostrada aqui, a camada de banco de dados não aceita o tráfego de front-end da Web, somente da camada comercial e da sub-rede de gerenciamento.</span><span class="sxs-lookup"><span data-stu-id="0b641-117">For example, in the 3-tier architecture shown here, the database tier does not accept traffic from the web front end, only from the business tier and the management subnet.</span></span>

* <span data-ttu-id="0b641-118">**Máquinas virtuais**.</span><span class="sxs-lookup"><span data-stu-id="0b641-118">**Virtual machines**.</span></span> <span data-ttu-id="0b641-119">Para obter recomendações sobre como configurar máquinas virtuais, consulte [Executar uma VM do Windows no Azure](./windows-vm.md) e [Executar uma VM do Linux no Azure](./linux-vm.md).</span><span class="sxs-lookup"><span data-stu-id="0b641-119">For recommendations on configuring VMs, see [Run a Windows VM on Azure](./windows-vm.md) and [Run a Linux VM on Azure](./linux-vm.md).</span></span>

* <span data-ttu-id="0b641-120">**Conjuntos de disponibilidade.**</span><span class="sxs-lookup"><span data-stu-id="0b641-120">**Availability sets.**</span></span> <span data-ttu-id="0b641-121">Crie um [conjunto de disponibilidade][azure-availability-sets] para cada camada e provisione pelo menos duas VMs em cada camada.</span><span class="sxs-lookup"><span data-stu-id="0b641-121">Create an [availability set][azure-availability-sets] for each tier, and provision at least two VMs in each tier.</span></span> <span data-ttu-id="0b641-122">Isso torna as VMs qualificadas para um [SLA (Contrato de Nível de Serviço)][vm-sla] mais elevado.</span><span class="sxs-lookup"><span data-stu-id="0b641-122">This makes the VMs eligible for a higher [service level agreement (SLA)][vm-sla] for VMs.</span></span> 

* <span data-ttu-id="0b641-123">**Conjunto de dimensionamento de VM** (não mostrado).</span><span class="sxs-lookup"><span data-stu-id="0b641-123">**VM scale set** (not shown).</span></span> <span data-ttu-id="0b641-124">Um [Conjunto de dimensionamento de VM] [ vmss] é uma alternativa ao uso de um conjunto de disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="0b641-124">A [VM scale set][vmss] is an alternative to using an availability set.</span></span> <span data-ttu-id="0b641-125">Um conjunto de dimensionamento facilita o escalonamento horizontal de VMs em uma camada, seja manual ou automaticamente, com base em regras predefinidas.</span><span class="sxs-lookup"><span data-stu-id="0b641-125">A scale sets makes it easy to scale out the VMs in a tier, either manually or automatically based on predefined rules.</span></span>

* <span data-ttu-id="0b641-126">**Balanceadores de carga do Azure.**</span><span class="sxs-lookup"><span data-stu-id="0b641-126">**Azure Load balancers.**</span></span> <span data-ttu-id="0b641-127">Os [balanceadores de carga] [ load-balancer] distribuem solicitações de entrada da Internet para as instâncias de VM.</span><span class="sxs-lookup"><span data-stu-id="0b641-127">The [load balancers][load-balancer] distribute incoming Internet requests to the VM instances.</span></span> <span data-ttu-id="0b641-128">Use um [balanceador de carga público][load-balancer-external] para distribuir o tráfego de entrada da Internet para a camada da Web e um [balanceador de carga interno][load-balancer-internal] para distribuir o tráfego de rede da camada da Web para a camada comercial.</span><span class="sxs-lookup"><span data-stu-id="0b641-128">Use a [public load balancer][load-balancer-external] to distribute incoming Internet traffic to the web tier, and an [internal load balancer][load-balancer-internal] to distribute network traffic from the web tier to the business tier.</span></span>

* <span data-ttu-id="0b641-129">**Endereço IP público**.</span><span class="sxs-lookup"><span data-stu-id="0b641-129">**Public IP address**.</span></span> <span data-ttu-id="0b641-130">É necessário ter um endereço IP para que o balanceador de carga possa receber o tráfego da Internet.</span><span class="sxs-lookup"><span data-stu-id="0b641-130">A public IP address is needed for the public load balancer to receive Internet traffic.</span></span>

* <span data-ttu-id="0b641-131">**Jumpbox.**</span><span class="sxs-lookup"><span data-stu-id="0b641-131">**Jumpbox.**</span></span> <span data-ttu-id="0b641-132">Também chamada de um [host bastião].</span><span class="sxs-lookup"><span data-stu-id="0b641-132">Also called a [bastion host].</span></span> <span data-ttu-id="0b641-133">Uma VM protegida na rede que os administradores usam para se conectar às outras VMs.</span><span class="sxs-lookup"><span data-stu-id="0b641-133">A secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="0b641-134">O jumpbox tem um NSG que permite o tráfego remoto apenas de endereços IP públicos em uma lista segura.</span><span class="sxs-lookup"><span data-stu-id="0b641-134">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="0b641-135">O NSG deve permitir o tráfego de RDP (área de trabalho remota).</span><span class="sxs-lookup"><span data-stu-id="0b641-135">The NSG should permit remote desktop (RDP) traffic.</span></span>

* <span data-ttu-id="0b641-136">**Grupo de Disponibilidade Always On do SQL Server.**</span><span class="sxs-lookup"><span data-stu-id="0b641-136">**SQL Server Always On Availability Group.**</span></span> <span data-ttu-id="0b641-137">Fornece alta disponibilidade na camada de dados, habilitando replicação e failover.</span><span class="sxs-lookup"><span data-stu-id="0b641-137">Provides high availability at the data tier, by enabling replication and failover.</span></span> <span data-ttu-id="0b641-138">Usa a tecnologia WSFC (Cluster de Failover do Windows Server) para o failover.</span><span class="sxs-lookup"><span data-stu-id="0b641-138">It uses Windows Server Failover Cluster (WSFC) technology for failover.</span></span>

* <span data-ttu-id="0b641-139">**Servidores AD DS (Active Directory Domain Services)**.</span><span class="sxs-lookup"><span data-stu-id="0b641-139">**Active Directory Domain Services (AD DS) Servers**.</span></span> <span data-ttu-id="0b641-140">Os objetos de computação do cluster de failover e suas funções em cluster associadas são criados no Active Directory Domain Services (AD DS).</span><span class="sxs-lookup"><span data-stu-id="0b641-140">The computer objects for the failover cluster and its associated clustered roles are created in Active Directory Domain Services (AD DS).</span></span>

* <span data-ttu-id="0b641-141">**Testemunha de Nuvem**.</span><span class="sxs-lookup"><span data-stu-id="0b641-141">**Cloud Witness**.</span></span> <span data-ttu-id="0b641-142">Um cluster de failover requer mais da metade dos seus nós em execução, que é conhecido como ter quorum.</span><span class="sxs-lookup"><span data-stu-id="0b641-142">A failover cluster requires more than half of its nodes to be running, which is known as having quorum.</span></span> <span data-ttu-id="0b641-143">Se o cluster tem apenas dois nós, uma partição de rede pode fazer com que cada nó pense que é o principal.</span><span class="sxs-lookup"><span data-stu-id="0b641-143">If the cluster has just two nodes, a network partition could cause each node to think it's the master node.</span></span> <span data-ttu-id="0b641-144">Nesse caso, é necessário uma *testemunha* para desempatar e estabelecer o quorum.</span><span class="sxs-lookup"><span data-stu-id="0b641-144">In that case, you need a *witness* to break ties and establish quorum.</span></span> <span data-ttu-id="0b641-145">Testemunha é um recurso, como um disco compartilhado, que pode agir como um desempate para estabelecer o quorum.</span><span class="sxs-lookup"><span data-stu-id="0b641-145">A witness is a resource such as a shared disk that can act as a tie breaker to establish quorum.</span></span> <span data-ttu-id="0b641-146">A Testemunha de Nuvem é um tipo que usa o Armazenamento de Blobs do Azure.</span><span class="sxs-lookup"><span data-stu-id="0b641-146">Cloud Witness is a type of witness that uses Azure Blob Storage.</span></span> <span data-ttu-id="0b641-147">Para saber mais sobre o conceito de quorum, consulte [Entendendo o cluster e o quorum de pool](/windows-server/storage/storage-spaces/understand-quorum).</span><span class="sxs-lookup"><span data-stu-id="0b641-147">To learn more about the concept of quorum, see [Understanding cluster and pool quorum](/windows-server/storage/storage-spaces/understand-quorum).</span></span> <span data-ttu-id="0b641-148">Para obter mais informações sobre a Testemunha de Nuvem, consulte [Implantar uma Testemunha de Nuvem para um Cluster de Failover](/windows-server/failover-clustering/deploy-cloud-witness).</span><span class="sxs-lookup"><span data-stu-id="0b641-148">For more information about Cloud Witness, see [Deploy a Cloud Witness for a Failover Cluster](/windows-server/failover-clustering/deploy-cloud-witness).</span></span> 

* <span data-ttu-id="0b641-149">**DNS do Azure**.</span><span class="sxs-lookup"><span data-stu-id="0b641-149">**Azure DNS**.</span></span> <span data-ttu-id="0b641-150">[DNS do Azure][azure-dns] é um serviço de hospedagem para domínios DNS, que fornece resolução de nomes usando a infraestrutura do Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="0b641-150">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="0b641-151">Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure.</span><span class="sxs-lookup"><span data-stu-id="0b641-151">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

## <a name="recommendations"></a><span data-ttu-id="0b641-152">Recomendações</span><span class="sxs-lookup"><span data-stu-id="0b641-152">Recommendations</span></span>

<span data-ttu-id="0b641-153">Seus requisitos podem ser diferentes dos requisitos da arquitetura descrita aqui.</span><span class="sxs-lookup"><span data-stu-id="0b641-153">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="0b641-154">Use essas recomendações como ponto de partida.</span><span class="sxs-lookup"><span data-stu-id="0b641-154">Use these recommendations as a starting point.</span></span> 

### <a name="vnet--subnets"></a><span data-ttu-id="0b641-155">VNet / Sub-redes</span><span class="sxs-lookup"><span data-stu-id="0b641-155">VNet / Subnets</span></span>

<span data-ttu-id="0b641-156">Quando você cria a VNet, determine quantos endereços IP seus recursos em cada sub-rede exigem.</span><span class="sxs-lookup"><span data-stu-id="0b641-156">When you create the VNet, determine how many IP addresses your resources in each subnet require.</span></span> <span data-ttu-id="0b641-157">Especifique uma máscara de sub-rede e um intervalo de endereços de VNet grande o suficiente para os endereços IP necessários, usando a notação [CIDR].</span><span class="sxs-lookup"><span data-stu-id="0b641-157">Specify a subnet mask and a VNet address range large enough for the required IP addresses, using [CIDR] notation.</span></span> <span data-ttu-id="0b641-158">Use um espaço de endereço que esteja dentro dos [blocos de endereço IP privados][private-ip-space] padrão, que são 10.0.0.0/8, 172.16.0.0/12 e 192.168.0.0/16.</span><span class="sxs-lookup"><span data-stu-id="0b641-158">Use an address space that falls within the standard [private IP address blocks][private-ip-space], which are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.</span></span>

<span data-ttu-id="0b641-159">Escolha um intervalo de endereços que não se sobreponha ao da rede local para caso seja necessário configurar um gateway entre a VNet e a rede local mais tarde.</span><span class="sxs-lookup"><span data-stu-id="0b641-159">Choose an address range that does not overlap with your on-premises network, in case you need to set up a gateway between the VNet and your on-premise network later.</span></span> <span data-ttu-id="0b641-160">Depois de criar a VNet, não será possível alterar o intervalo de endereços.</span><span class="sxs-lookup"><span data-stu-id="0b641-160">Once you create the VNet, you can't change the address range.</span></span>

<span data-ttu-id="0b641-161">Crie as sub-redes levando em conta os requisitos de funcionalidade e de segurança.</span><span class="sxs-lookup"><span data-stu-id="0b641-161">Design subnets with functionality and security requirements in mind.</span></span> <span data-ttu-id="0b641-162">Todas as VMs na mesma camada ou função devem ir para a mesma sub-rede, que pode ser um limite de segurança.</span><span class="sxs-lookup"><span data-stu-id="0b641-162">All VMs within the same tier or role should go into the same subnet, which can be a security boundary.</span></span> <span data-ttu-id="0b641-163">Para obter mais informações sobre como criar VNets e sub-redes, consulte [Planejar e criar redes virtuais do Azure][plan-network].</span><span class="sxs-lookup"><span data-stu-id="0b641-163">For more information about designing VNets and subnets, see [Plan and design Azure Virtual Networks][plan-network].</span></span>

### <a name="load-balancers"></a><span data-ttu-id="0b641-164">Balanceadores de carga</span><span class="sxs-lookup"><span data-stu-id="0b641-164">Load balancers</span></span>

<span data-ttu-id="0b641-165">Não exponha as VMs diretamente à Internet, concedendo, em vez disso, um endereço IP privado a cada VM.</span><span class="sxs-lookup"><span data-stu-id="0b641-165">Do not expose the VMs directly to the Internet, but instead give each VM a private IP address.</span></span> <span data-ttu-id="0b641-166">Os clientes se conectam usando o endereço IP do balanceador de carga público.</span><span class="sxs-lookup"><span data-stu-id="0b641-166">Clients connect using the IP address of the public load balancer.</span></span>

<span data-ttu-id="0b641-167">Defina as regras do balanceador de carga para direcionar tráfego de rede para as VMs.</span><span class="sxs-lookup"><span data-stu-id="0b641-167">Define load balancer rules to direct network traffic to the VMs.</span></span> <span data-ttu-id="0b641-168">Por exemplo, para permitir tráfego HTTP, crie uma regra que mapeie a porta 80 da configuração de front-end para a porta 80 no pool de endereços de back-end.</span><span class="sxs-lookup"><span data-stu-id="0b641-168">For example, to enable HTTP traffic, create a rule that maps port 80 from the front-end configuration to port 80 on the back-end address pool.</span></span> <span data-ttu-id="0b641-169">Quando um cliente envia uma solicitação HTTP para a porta 80, o balanceador de carga seleciona um endereço IP de back-end usando um [algoritmo de hash][load-balancer-hashing] que inclui o endereço IP de origem.</span><span class="sxs-lookup"><span data-stu-id="0b641-169">When a client sends an HTTP request to port 80, the load balancer selects a back-end IP address by using a [hashing algorithm][load-balancer-hashing] that includes the source IP address.</span></span> <span data-ttu-id="0b641-170">Dessa forma, as solicitações de cliente são distribuídas por todas as VMs.</span><span class="sxs-lookup"><span data-stu-id="0b641-170">In that way, client requests are distributed across all the VMs.</span></span>

### <a name="network-security-groups"></a><span data-ttu-id="0b641-171">Grupos de segurança de rede</span><span class="sxs-lookup"><span data-stu-id="0b641-171">Network security groups</span></span>

<span data-ttu-id="0b641-172">Use as regras de NSG para restringir o tráfego entre as camadas.</span><span class="sxs-lookup"><span data-stu-id="0b641-172">Use NSG rules to restrict traffic between tiers.</span></span> <span data-ttu-id="0b641-173">Por exemplo, a arquitetura de três camadas mostrada acima, a camada da Web não se comunica diretamente com a camada de banco de dados.</span><span class="sxs-lookup"><span data-stu-id="0b641-173">For example, in the 3-tier architecture shown above, the web tier does not communicate directly with the database tier.</span></span> <span data-ttu-id="0b641-174">Para impor isso, a camada de banco de dados deve bloquear o tráfego de entrada da sub-rede da camada da Web.</span><span class="sxs-lookup"><span data-stu-id="0b641-174">To enforce this, the database tier should block incoming traffic from the web tier subnet.</span></span>  

1. <span data-ttu-id="0b641-175">Negue todo o tráfego de entrada do VNet.</span><span class="sxs-lookup"><span data-stu-id="0b641-175">Deny all inbound traffic from the VNet.</span></span> <span data-ttu-id="0b641-176">(Use a marca `VIRTUAL_NETWORK` na regra.)</span><span class="sxs-lookup"><span data-stu-id="0b641-176">(Use the `VIRTUAL_NETWORK` tag in the rule.)</span></span> 
2. <span data-ttu-id="0b641-177">Permita o tráfego de entrada da sub-rede de camada de negócios.</span><span class="sxs-lookup"><span data-stu-id="0b641-177">Allow inbound traffic from the business tier subnet.</span></span>  
3. <span data-ttu-id="0b641-178">Permita o tráfego de entrada da própria sub-rede de camada de dados.</span><span class="sxs-lookup"><span data-stu-id="0b641-178">Allow inbound traffic from the database tier subnet itself.</span></span> <span data-ttu-id="0b641-179">Essa regra permite a comunicação entre as VMs de banco de dados, que é necessária para failover e replicação de banco de dados.</span><span class="sxs-lookup"><span data-stu-id="0b641-179">This rule allows communication between the database VMs, which is needed for database replication and failover.</span></span>
4. <span data-ttu-id="0b641-180">Permita o tráfego RDP (porta 3389) da sub-rede jumpbox.</span><span class="sxs-lookup"><span data-stu-id="0b641-180">Allow RDP traffic (port 3389) from the jumpbox subnet.</span></span> <span data-ttu-id="0b641-181">Essa regra permite que os administradores se conectem à camada de banco de dados do jumpbox.</span><span class="sxs-lookup"><span data-stu-id="0b641-181">This rule lets administrators connect to the database tier from the jumpbox.</span></span>

<span data-ttu-id="0b641-182">Criar regras de 2 &ndash; 4 com prioridade mais alta que a primeira regra, para que elas a substituam.</span><span class="sxs-lookup"><span data-stu-id="0b641-182">Create rules 2 &ndash; 4 with higher priority than the first rule, so they override it.</span></span>


### <a name="sql-server-always-on-availability-groups"></a><span data-ttu-id="0b641-183">Grupos de Disponibilidade Always On do SQL Server</span><span class="sxs-lookup"><span data-stu-id="0b641-183">SQL Server Always On Availability Groups</span></span>

<span data-ttu-id="0b641-184">Recomendamos usar os [Grupos de Disponibilidade Always On][sql-alwayson] para alta disponibilidade no SQL Server.</span><span class="sxs-lookup"><span data-stu-id="0b641-184">We recommend [Always On Availability Groups][sql-alwayson] for SQL Server high availability.</span></span> <span data-ttu-id="0b641-185">Antes do Windows Server 2016, os Grupos de Disponibilidade Always On exigiam um controlador de domínio e todos os nós no grupo de disponibilidade precisavam estar no mesmo domínio do AD.</span><span class="sxs-lookup"><span data-stu-id="0b641-185">Prior to Windows Server 2016, Always On Availability Groups require a domain controller, and all nodes in the availability group must be in the same AD domain.</span></span>

<span data-ttu-id="0b641-186">Outras camadas se conectam ao banco de dados por meio de um [ouvinte do grupo de disponibilidade][sql-alwayson-listeners].</span><span class="sxs-lookup"><span data-stu-id="0b641-186">Other tiers connect to the database through an [availability group listener][sql-alwayson-listeners].</span></span> <span data-ttu-id="0b641-187">O ouvinte permite que um cliente SQL se conecte sem saber o nome da instância física do SQL Server.</span><span class="sxs-lookup"><span data-stu-id="0b641-187">The listener enables a SQL client to connect without knowing the name of the physical instance of SQL Server.</span></span> <span data-ttu-id="0b641-188">VMs que acessam o banco de dados precisam ser ingressadas no domínio.</span><span class="sxs-lookup"><span data-stu-id="0b641-188">VMs that access the database must be joined to the domain.</span></span> <span data-ttu-id="0b641-189">O cliente (neste caso, outra camada) usa o DNS para resolver o nome da rede virtual do ouvinte em endereços IP.</span><span class="sxs-lookup"><span data-stu-id="0b641-189">The client (in this case, another tier) uses DNS to resolve the listener's virtual network name into IP addresses.</span></span>

<span data-ttu-id="0b641-190">Configure um grupo de disponibilidade Always On do SQL Server da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="0b641-190">Configure the SQL Server Always On Availability Group as follows:</span></span>

1. <span data-ttu-id="0b641-191">Crie um cluster WSFC (Clustering de Failover do Windows Server), um Grupo de Disponibilidade Always On do SQL Server e uma réplica primária.</span><span class="sxs-lookup"><span data-stu-id="0b641-191">Create a Windows Server Failover Clustering (WSFC) cluster, a SQL Server Always On Availability Group, and a primary replica.</span></span> <span data-ttu-id="0b641-192">Para obter mais informações, consulte [Introdução aos Grupos de disponibilidade Always On][sql-alwayson-getting-started].</span><span class="sxs-lookup"><span data-stu-id="0b641-192">For more information, see [Getting Started with Always On Availability Groups][sql-alwayson-getting-started].</span></span> 
2. <span data-ttu-id="0b641-193">Crie um balanceador de carga interno com um endereço IP privado estático.</span><span class="sxs-lookup"><span data-stu-id="0b641-193">Create an internal load balancer with a static private IP address.</span></span>
3. <span data-ttu-id="0b641-194">Crie um ouvinte do grupo de disponibilidade e mapeie o nome DNS do ouvinte para o endereço IP de um balanceador de carga interno.</span><span class="sxs-lookup"><span data-stu-id="0b641-194">Create an availability group listener, and map the listener's DNS name to the IP address of an internal load balancer.</span></span> 
4. <span data-ttu-id="0b641-195">Crie uma regra do balanceador de carga para a porta de escuta do SQL Server (porta TCP 1433 por padrão).</span><span class="sxs-lookup"><span data-stu-id="0b641-195">Create a load balancer rule for the SQL Server listening port (TCP port 1433 by default).</span></span> <span data-ttu-id="0b641-196">A regra do balanceador de carga deve habilitar *IP flutuante*, também chamado de Retorno de Servidor Direto.</span><span class="sxs-lookup"><span data-stu-id="0b641-196">The load balancer rule must enable *floating IP*, also called Direct Server Return.</span></span> <span data-ttu-id="0b641-197">Isso faz com que a VM responda diretamente para o cliente, o que permite uma conexão direta com a réplica primária.</span><span class="sxs-lookup"><span data-stu-id="0b641-197">This causes the VM to reply directly to the client, which enables a direct connection to the primary replica.</span></span>
  
   > [!NOTE]
   > <span data-ttu-id="0b641-198">Quando o IP flutuante está habilitado, o número da porta de front-end deve ser igual ao número da porta de back-end na regra do balanceador de carga.</span><span class="sxs-lookup"><span data-stu-id="0b641-198">When floating IP is enabled, the front-end port number must be the same as the back-end port number in the load balancer rule.</span></span>
   > 
   > 

<span data-ttu-id="0b641-199">Quando um cliente SQL tenta se conectar, o balanceador de carga roteia a solicitação de conexão para a réplica primária.</span><span class="sxs-lookup"><span data-stu-id="0b641-199">When a SQL client tries to connect, the load balancer routes the connection request to the primary replica.</span></span> <span data-ttu-id="0b641-200">Se houver um failover para outra réplica, o balanceador de carga encaminhará automaticamente as solicitações subsequentes para uma nova réplica primária.</span><span class="sxs-lookup"><span data-stu-id="0b641-200">If there is a failover to another replica, the load balancer automatically routes subsequent requests to a new primary replica.</span></span> <span data-ttu-id="0b641-201">Para obter mais informações, consulte [Configurar um ouvinte de ILB para Grupos de Disponibilidade Always On do SQL Server][sql-alwayson-ilb].</span><span class="sxs-lookup"><span data-stu-id="0b641-201">For more information, see [Configure an ILB listener for SQL Server Always On Availability Groups][sql-alwayson-ilb].</span></span>

<span data-ttu-id="0b641-202">Durante um failover, as conexões de cliente existentes são fechadas.</span><span class="sxs-lookup"><span data-stu-id="0b641-202">During a failover, existing client connections are closed.</span></span> <span data-ttu-id="0b641-203">Após a conclusão do failover, novas conexões serão roteadas para a nova réplica primária.</span><span class="sxs-lookup"><span data-stu-id="0b641-203">After the failover completes, new connections will be routed to the new primary replica.</span></span>

<span data-ttu-id="0b641-204">Se seu aplicativo realiza muito mais leituras do que gravações, você pode descarregar algumas das consultas somente leitura para uma réplica secundária.</span><span class="sxs-lookup"><span data-stu-id="0b641-204">If your application makes significantly more reads than writes, you can offload some of the read-only queries to a secondary replica.</span></span> <span data-ttu-id="0b641-205">Consulte [Usar um ouvinte para se conectar a uma réplica secundária somente leitura (roteamento somente leitura)][sql-alwayson-read-only-routing].</span><span class="sxs-lookup"><span data-stu-id="0b641-205">See [Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing].</span></span>

<span data-ttu-id="0b641-206">Teste a implantação por [forçando um failover manual][sql-alwayson-force-failover] do grupo de disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="0b641-206">Test your deployment by [forcing a manual failover][sql-alwayson-force-failover] of the availability group.</span></span>

### <a name="jumpbox"></a><span data-ttu-id="0b641-207">Jumpbox</span><span class="sxs-lookup"><span data-stu-id="0b641-207">Jumpbox</span></span>

<span data-ttu-id="0b641-208">Não permita o acesso RDP da Internet pública para as VMs que executam a carga de trabalho do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="0b641-208">Do not allow RDP access from the public Internet to the VMs that run the application workload.</span></span> <span data-ttu-id="0b641-209">Em vez disso, todo o acesso RDP a essas VMs deve ocorrer por meio do jumpbox.</span><span class="sxs-lookup"><span data-stu-id="0b641-209">Instead, all RDP access to these VMs must come through the jumpbox.</span></span> <span data-ttu-id="0b641-210">Um administrador faz logon no jumpbox e, em seguida, faz logon na VM por meio do jumpbox.</span><span class="sxs-lookup"><span data-stu-id="0b641-210">An administrator logs into the jumpbox, and then logs into the other VM from the jumpbox.</span></span> <span data-ttu-id="0b641-211">O jumpbox permite que tráfego RDP da Internet, mas apenas de endereços IP conhecidos e seguros.</span><span class="sxs-lookup"><span data-stu-id="0b641-211">The jumpbox allows RDP traffic from the Internet, but only from known, safe IP addresses.</span></span>

<span data-ttu-id="0b641-212">O jumpbox tem requisitos de desempenho mínimo, por isso, selecione uma VM de tamanho pequeno.</span><span class="sxs-lookup"><span data-stu-id="0b641-212">The jumpbox has minimal performance requirements, so select a small VM size.</span></span> <span data-ttu-id="0b641-213">Crie um [endereço IP público] para o jumpbox.</span><span class="sxs-lookup"><span data-stu-id="0b641-213">Create a [public IP address] for the jumpbox.</span></span> <span data-ttu-id="0b641-214">Coloque o jumpbox na mesma VNet que as outras VMs, mas em uma sub-rede de gerenciamento separada.</span><span class="sxs-lookup"><span data-stu-id="0b641-214">Place the jumpbox in the same VNet as the other VMs, but in a separate management subnet.</span></span>

<span data-ttu-id="0b641-215">Para proteger o jumpbox, adicione uma regra NSG que permite conexões RDP somente de um conjunto seguro de endereços IP públicos.</span><span class="sxs-lookup"><span data-stu-id="0b641-215">To secure the jumpbox, add an NSG rule that allows RDP connections only from a safe set of public IP addresses.</span></span> <span data-ttu-id="0b641-216">Configure os NSGs para outras sub-redes para permitir o tráfego RDP da sub-rede de gerenciamento.</span><span class="sxs-lookup"><span data-stu-id="0b641-216">Configure the NSGs for the other subnets to allow RDP traffic from the management subnet.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="0b641-217">Considerações sobre escalabilidade</span><span class="sxs-lookup"><span data-stu-id="0b641-217">Scalability considerations</span></span>

<span data-ttu-id="0b641-218">[Conjuntos de dimensionamento de VM] [ vmss] ajudam a implantar e gerenciar um conjunto de VMs idênticas.</span><span class="sxs-lookup"><span data-stu-id="0b641-218">[VM scale sets][vmss] help you to deploy and manage a set of identical VMs.</span></span> <span data-ttu-id="0b641-219">Conjuntos de dimensionamento dão suporte ao dimensionamento automático com base nas métricas de desempenho.</span><span class="sxs-lookup"><span data-stu-id="0b641-219">Scale sets support autoscaling based on performance metrics.</span></span> <span data-ttu-id="0b641-220">À medida que a carga nas VMs aumenta, VMs adicionais são acrescentadas automaticamente ao balanceador de carga.</span><span class="sxs-lookup"><span data-stu-id="0b641-220">As the load on the VMs increases, additional VMs are automatically added to the load balancer.</span></span> <span data-ttu-id="0b641-221">Considere usar os conjuntos de dimensionamento se você precisar aumentar as VMs rapidamente ou se precisar de dimensionamento automático.</span><span class="sxs-lookup"><span data-stu-id="0b641-221">Consider scale sets if you need to quickly scale out VMs, or need to autoscale.</span></span>

<span data-ttu-id="0b641-222">Há duas maneiras básicas de configurar as VMs implantadas em um conjunto de dimensionamento:</span><span class="sxs-lookup"><span data-stu-id="0b641-222">There are two basic ways to configure VMs deployed in a scale set:</span></span>

- <span data-ttu-id="0b641-223">Use as extensões para configurar a VM depois que ela é provisionada.</span><span class="sxs-lookup"><span data-stu-id="0b641-223">Use extensions to configure the VM after it is provisioned.</span></span> <span data-ttu-id="0b641-224">Com essa abordagem, novas instâncias de VM podem levar mais tempo para ser iniciadas do que uma VM sem extensões.</span><span class="sxs-lookup"><span data-stu-id="0b641-224">With this approach, new VM instances may take longer to start up than a VM with no extensions.</span></span>

- <span data-ttu-id="0b641-225">Implante um [disco gerenciado](/azure/storage/storage-managed-disks-overview) com uma imagem de disco personalizada.</span><span class="sxs-lookup"><span data-stu-id="0b641-225">Deploy a [managed disk](/azure/storage/storage-managed-disks-overview) with a custom disk image.</span></span> <span data-ttu-id="0b641-226">Essa opção pode ser mais rápida de implantar.</span><span class="sxs-lookup"><span data-stu-id="0b641-226">This option may be quicker to deploy.</span></span> <span data-ttu-id="0b641-227">Porém, isso requer que você mantenha a imagem atualizada.</span><span class="sxs-lookup"><span data-stu-id="0b641-227">However, it requires you to keep the image up to date.</span></span>

<span data-ttu-id="0b641-228">Para ver considerações adicionais, consulte [Considerações de design para conjuntos de dimensionamento][vmss-design].</span><span class="sxs-lookup"><span data-stu-id="0b641-228">For additional considerations, see [Design considerations for scale sets][vmss-design].</span></span>

> [!TIP]
> <span data-ttu-id="0b641-229">Ao usar qualquer solução de dimensionamento automático, teste-a com cargas de trabalho no nível de produção com bastante antecedência.</span><span class="sxs-lookup"><span data-stu-id="0b641-229">When using any autoscale solution, test it with production-level workloads well in advance.</span></span>

<span data-ttu-id="0b641-230">Cada assinatura do Azure tem limites padrão em vigor, incluindo um número máximo de VMs por região.</span><span class="sxs-lookup"><span data-stu-id="0b641-230">Each Azure subscription has default limits in place, including a maximum number of VMs per region.</span></span> <span data-ttu-id="0b641-231">Você pode aumentar o limite enviando uma solicitação de suporte.</span><span class="sxs-lookup"><span data-stu-id="0b641-231">You can increase the limit by filing a support request.</span></span> <span data-ttu-id="0b641-232">Para saber mais, confira [Assinatura e limites de serviço, cotas e restrições do Azure][subscription-limits].</span><span class="sxs-lookup"><span data-stu-id="0b641-232">For more information, see [Azure subscription and service limits, quotas, and constraints][subscription-limits].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="0b641-233">Considerações sobre disponibilidade</span><span class="sxs-lookup"><span data-stu-id="0b641-233">Availability considerations</span></span>

<span data-ttu-id="0b641-234">Se você não estiver usando conjuntos de dimensionamento de VM, coloque as VMs na mesma camada em um conjunto de disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="0b641-234">If you are not using VM scale sets, put VMs in the same tier into an availability set.</span></span> <span data-ttu-id="0b641-235">Crie pelo menos duas VMs no conjunto de disponibilidade para dar suporte ao [SLA de disponibilidade para VMs do Azure][vm-sla].</span><span class="sxs-lookup"><span data-stu-id="0b641-235">Create at least two VMs in the availability set to support the [availability SLA for Azure VMs][vm-sla].</span></span> <span data-ttu-id="0b641-236">Para obter mais informações, consulte [Gerenciar a disponibilidade de máquinas virtuais][availability-set].</span><span class="sxs-lookup"><span data-stu-id="0b641-236">For more information, see [Manage the availability of virtual machines][availability-set].</span></span> 

<span data-ttu-id="0b641-237">O balanceador de carga utiliza [investigações de integridade][health-probes] para monitorar a disponibilidade de instâncias de VM.</span><span class="sxs-lookup"><span data-stu-id="0b641-237">The load balancer uses [health probes][health-probes] to monitor the availability of VM instances.</span></span> <span data-ttu-id="0b641-238">Se uma investigação não puder acessar uma instância dentro de um período de tempo limite, o balanceador de carga parará de enviar tráfego para essa VM.</span><span class="sxs-lookup"><span data-stu-id="0b641-238">If a probe cannot reach an instance within a timeout period, the load balancer stops sending traffic to that VM.</span></span> <span data-ttu-id="0b641-239">Contudo, o balanceador de carga continuará a investigar e, se a VM ficar disponível novamente, o balanceador de carga reiniciará o envio de tráfego para ela.</span><span class="sxs-lookup"><span data-stu-id="0b641-239">However, the load balancer will continue to probe, and if the VM becomes available again, the load balancer resumes sending traffic to that VM.</span></span>

<span data-ttu-id="0b641-240">Aqui estão algumas recomendações sobre as investigações de integridade do balanceador de carga:</span><span class="sxs-lookup"><span data-stu-id="0b641-240">Here are some recommendations on load balancer health probes:</span></span>

* <span data-ttu-id="0b641-241">As investigações podem testar HTTP ou TCP.</span><span class="sxs-lookup"><span data-stu-id="0b641-241">Probes can test either HTTP or TCP.</span></span> <span data-ttu-id="0b641-242">Se suas VMs são executadas em um servidor HTTP, crie uma investigação HTTP.</span><span class="sxs-lookup"><span data-stu-id="0b641-242">If your VMs run an HTTP server, create an HTTP probe.</span></span> <span data-ttu-id="0b641-243">Caso contrário, crie uma investigação TCP.</span><span class="sxs-lookup"><span data-stu-id="0b641-243">Otherwise create a TCP probe.</span></span>
* <span data-ttu-id="0b641-244">Para uma investigação HTTP, especifique o caminho para um ponto de extremidade HTTP.</span><span class="sxs-lookup"><span data-stu-id="0b641-244">For an HTTP probe, specify the path to an HTTP endpoint.</span></span> <span data-ttu-id="0b641-245">A investigação verifica uma resposta HTTP 200 para esse caminho.</span><span class="sxs-lookup"><span data-stu-id="0b641-245">The probe checks for an HTTP 200 response from this path.</span></span> <span data-ttu-id="0b641-246">Ele pode ser o caminho raiz (“/”) ou um ponto de extremidade de monitoramento de integridade que implementa lógica personalizada para verificar a integridade do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="0b641-246">This can be the root path ("/"), or a health-monitoring endpoint that implements some custom logic to check the health of the application.</span></span> <span data-ttu-id="0b641-247">O ponto de extremidade deve permitir solicitações HTTP anônimas.</span><span class="sxs-lookup"><span data-stu-id="0b641-247">The endpoint must allow anonymous HTTP requests.</span></span>
* <span data-ttu-id="0b641-248">A investigação é enviada de um endereço IP [conhecido][health-probe-ip], 168.63.129.16.</span><span class="sxs-lookup"><span data-stu-id="0b641-248">The probe is sent from a [known IP address][health-probe-ip], 168.63.129.16.</span></span> <span data-ttu-id="0b641-249">Verifique se o tráfego de ou para esse endereço IP não é bloqueado por quaisquer políticas de firewall ou regras de NSG (Grupo de Segurança de Rede).</span><span class="sxs-lookup"><span data-stu-id="0b641-249">Make sure you don't block traffic to or from this IP address in any firewall policies or network security group (NSG) rules.</span></span>
* <span data-ttu-id="0b641-250">Use os [logs de investigação de integridade][health-probe-log] para exibir o status das investigações de integridade.</span><span class="sxs-lookup"><span data-stu-id="0b641-250">Use [health probe logs][health-probe-log] to view the status of the health probes.</span></span> <span data-ttu-id="0b641-251">Habilite o registro em log no Portal do Azure para cada balanceador de carga.</span><span class="sxs-lookup"><span data-stu-id="0b641-251">Enable logging in the Azure portal for each load balancer.</span></span> <span data-ttu-id="0b641-252">Os logs são gravados no Armazenamento de Blobs do Azure.</span><span class="sxs-lookup"><span data-stu-id="0b641-252">Logs are written to Azure Blob storage.</span></span> <span data-ttu-id="0b641-253">Os logs de mostram como muitas VMs no back-end não estão recebendo o tráfego de rede devido a respostas de investigação com falha.</span><span class="sxs-lookup"><span data-stu-id="0b641-253">The logs show how many VMs on the back end are not receiving network traffic due to failed probe responses.</span></span>

<span data-ttu-id="0b641-254">Se você precisar de disponibilidade mais alta do que a fornecida pelo [SLA do Azure SLA para VMs][vm-sla], replique o aplicativo em duas regiões e use o Gerenciador de Tráfego do Azure para failover.</span><span class="sxs-lookup"><span data-stu-id="0b641-254">If you need higher availability than the [Azure SLA for VMs][vm-sla] provides, consider replication the application across two regions, using Azure Traffic Manager for failover.</span></span> <span data-ttu-id="0b641-255">Para obter mais informações, consulte [Aplicativo de N camadas de várias regiões para alta disponibilidade][multi-dc].</span><span class="sxs-lookup"><span data-stu-id="0b641-255">For more information, see [Multi-region N-tier application for high availability][multi-dc].</span></span>  

## <a name="security-considerations"></a><span data-ttu-id="0b641-256">Considerações de segurança</span><span class="sxs-lookup"><span data-stu-id="0b641-256">Security considerations</span></span>

<span data-ttu-id="0b641-257">Redes virtuais são um limite de isolamento de tráfego no Azure.</span><span class="sxs-lookup"><span data-stu-id="0b641-257">Virtual networks are a traffic isolation boundary in Azure.</span></span> <span data-ttu-id="0b641-258">As VMs em uma VNet não podem se comunicar diretamente com VMs em uma VNet diferente.</span><span class="sxs-lookup"><span data-stu-id="0b641-258">VMs in one VNet cannot communicate directly with VMs in a different VNet.</span></span> <span data-ttu-id="0b641-259">As VMs na mesma VNet podem se comunicar, a menos que você crie NSGs ([Grupos de Segurança de Rede][nsg]) para restringir o tráfego.</span><span class="sxs-lookup"><span data-stu-id="0b641-259">VMs within the same VNet can communicate, unless you create [network security groups][nsg] (NSGs) to restrict traffic.</span></span> <span data-ttu-id="0b641-260">Para obter mais informações, consulte [Serviços em nuvem da Microsoft e segurança de rede][network-security].</span><span class="sxs-lookup"><span data-stu-id="0b641-260">For more information, see [Microsoft cloud services and network security][network-security].</span></span>

<span data-ttu-id="0b641-261">Para tráfego de entrada da Internet, as regras do balanceador de carga definem qual tráfego pode alcançar o back-end.</span><span class="sxs-lookup"><span data-stu-id="0b641-261">For incoming Internet traffic, the load balancer rules define which traffic can reach the back end.</span></span> <span data-ttu-id="0b641-262">No entanto, as regras do balanceador de carga não dão suporte a listas de IP de confiança, por isso se você desejar adicionar determinados endereços IP públicos a uma lista de confiança, acrescente um NSG à sub-rede.</span><span class="sxs-lookup"><span data-stu-id="0b641-262">However, load balancer rules don't support IP safe lists, so if you want to add certain public IP addresses to a safe list, add an NSG to the subnet.</span></span>

<span data-ttu-id="0b641-263">Considere adicionar uma NVA (solução de virtualização de rede) para criar uma DMZ entre a Internet e a rede virtual do Azure.</span><span class="sxs-lookup"><span data-stu-id="0b641-263">Consider adding a network virtual appliance (NVA) to create a DMZ between the Internet and the Azure virtual network.</span></span> <span data-ttu-id="0b641-264">NVA é um termo genérico para uma solução de virtualização que pode executar tarefas relacionadas à rede, como firewall, inspeção de pacotes, auditoria e roteamento personalizado.</span><span class="sxs-lookup"><span data-stu-id="0b641-264">NVA is a generic term for a virtual appliance that can perform network-related tasks, such as firewall, packet inspection, auditing, and custom routing.</span></span> <span data-ttu-id="0b641-265">Para obter mais informações, consulte [Implementação de uma DMZ entre o Azure e a Internet][dmz].</span><span class="sxs-lookup"><span data-stu-id="0b641-265">For more information, see [Implementing a DMZ between Azure and the Internet][dmz].</span></span>

<span data-ttu-id="0b641-266">Criptografe dados confidenciais em repouso e use o [Azure Key Vault][azure-key-vault] para gerenciar as chaves de criptografia de banco de dados.</span><span class="sxs-lookup"><span data-stu-id="0b641-266">Encrypt sensitive data at rest and use [Azure Key Vault][azure-key-vault] to manage the database encryption keys.</span></span> <span data-ttu-id="0b641-267">O Key Vault pode armazenar chaves de criptografia em HSMs (módulos de segurança de hardware).</span><span class="sxs-lookup"><span data-stu-id="0b641-267">Key Vault can store encryption keys in hardware security modules (HSMs).</span></span> <span data-ttu-id="0b641-268">Para mais informações, consulte [Configurar a Integração do Azure Key Vault para o SQL nas VMs do Azure][sql-keyvault].</span><span class="sxs-lookup"><span data-stu-id="0b641-268">For more information, see [Configure Azure Key Vault Integration for SQL Server on Azure VMs][sql-keyvault].</span></span> <span data-ttu-id="0b641-269">Também é recomendado para armazenar segredos do aplicativo, como cadeias de caracteres de conexão de banco de dados, no cofre de chaves.</span><span class="sxs-lookup"><span data-stu-id="0b641-269">It's also recommended to store application secrets, such as database connection strings, in Key Vault.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="0b641-270">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="0b641-270">Deploy the solution</span></span>

<span data-ttu-id="0b641-271">Uma implantação para essa arquitetura de referência está disponível no [GitHub][github-folder].</span><span class="sxs-lookup"><span data-stu-id="0b641-271">A deployment for this reference architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="0b641-272">Observe que a implantação inteira pode levar até duas horas, que inclui a execução de scripts para configurar o AD DS, o cluster de failover do Windows Server e o grupo de disponibilidade do SQL Server.</span><span class="sxs-lookup"><span data-stu-id="0b641-272">Note that the entire deployment can take up to two hours, which includes running the scripts to configure AD DS, the Windows Server failover cluster, and the SQL Server availability group.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="0b641-273">pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="0b641-273">Prerequisites</span></span>

1. <span data-ttu-id="0b641-274">Clone, crie um fork ou baixe o arquivo zip das [arquiteturas de referência][ref-arch-repo] no repositório GitHub.</span><span class="sxs-lookup"><span data-stu-id="0b641-274">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="0b641-275">Instale a [CLI do Azure 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="0b641-275">Install [Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="0b641-276">Instale os pacote npm dos [Blocos de construção do Azure][azbb].</span><span class="sxs-lookup"><span data-stu-id="0b641-276">Install the [Azure building blocks][azbb] npm package.</span></span>

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. <span data-ttu-id="0b641-277">Em um prompt de comando, prompt do bash ou prompt do PowerShell, faça logon na sua conta do Azure usando o comando abaixo.</span><span class="sxs-lookup"><span data-stu-id="0b641-277">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-solution"></a><span data-ttu-id="0b641-278">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="0b641-278">Deploy the solution</span></span> 

1. <span data-ttu-id="0b641-279">Execute o seguinte comando para criar um grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="0b641-279">Run the following command to create a resource group.</span></span>

    ```bash
    az group create --location <location> --name <resource-group-name>
    ```

2. <span data-ttu-id="0b641-280">Execute o seguinte comando para criar uma conta de Armazenamento para a Testemunha de Nuvem.</span><span class="sxs-lookup"><span data-stu-id="0b641-280">Run the following command to create a Storage account for the Cloud Witness.</span></span>

    ```bash
    az storage account create --location <location> \
      --name <storage-account-name> \
      --resource-group <resource-group-name> \
      --sku Standard_LRS
    ```

3. <span data-ttu-id="0b641-281">Navegue para a pasta `virtual-machines\n-tier-windows` do repositório GitHub de arquiteturas de referência.</span><span class="sxs-lookup"><span data-stu-id="0b641-281">Navigate to the `virtual-machines\n-tier-windows` folder of the reference architectures GitHub repository.</span></span>

4. <span data-ttu-id="0b641-282">Abra o arquivo `n-tier-windows.json` .</span><span class="sxs-lookup"><span data-stu-id="0b641-282">Open the `n-tier-windows.json` file.</span></span> 

5. <span data-ttu-id="0b641-283">Procurar todas as instâncias de "witnessStorageBlobEndPoint" e substitua o texto do espaço reservado pelo nome da conta de Armazenamento da etapa 2.</span><span class="sxs-lookup"><span data-stu-id="0b641-283">Search for all instances of "witnessStorageBlobEndPoint" and replace the placeholder text with the name of the Storage account from step 2.</span></span>

    ```json
    "witnessStorageBlobEndPoint": "https://[replace-with-storageaccountname].blob.core.windows.net",
    ```

6. <span data-ttu-id="0b641-284">Execute o comando a seguir para enumerar as chaves de conta para a conta de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="0b641-284">Run the following command to list the account keys for the storage account.</span></span>

    ```bash
    az storage account keys list \
      --account-name <storage-account-name> \
      --resource-group <resource-group-name>
    ```

    <span data-ttu-id="0b641-285">A saída deve parecer com o seguinte.</span><span class="sxs-lookup"><span data-stu-id="0b641-285">The output should look like the following.</span></span> <span data-ttu-id="0b641-286">Copie o valor de `key1`.</span><span class="sxs-lookup"><span data-stu-id="0b641-286">Copy the value of `key1`.</span></span>

    ```json
    [
    {
        "keyName": "key1",
        "permissions": "Full",
        "value": "..."
    },
    {
        "keyName": "key2",
        "permissions": "Full",
        "value": "..."
    }
    ]
    ```

7. <span data-ttu-id="0b641-287">No arquivo `n-tier-windows.json`, procure todas as instâncias de "witnessStorageAccountKey e cole na chave de conta.</span><span class="sxs-lookup"><span data-stu-id="0b641-287">In the `n-tier-windows.json` file, search for all instances of "witnessStorageAccountKey" and paste in the account key.</span></span>

    ```json
    "witnessStorageAccountKey": "[replace-with-storagekey]"
    ```

8. <span data-ttu-id="0b641-288">No arquivo `n-tier-windows.json`, procure todas as instâncias `testPassw0rd!23`, `test$!Passw0rd111` e `AweS0me@SQLServicePW`.</span><span class="sxs-lookup"><span data-stu-id="0b641-288">In the `n-tier-windows.json` file, search for all instances `testPassw0rd!23`, `test$!Passw0rd111`, and `AweS0me@SQLServicePW`.</span></span> <span data-ttu-id="0b641-289">Substitua-as por suas próprias senhas e salve o arquivo.</span><span class="sxs-lookup"><span data-stu-id="0b641-289">Replace them with your own passwords and save the file.</span></span>

    > [!NOTE]
    > <span data-ttu-id="0b641-290">Se você alterar o nome de usuário do administrador, também terá que atualizar os blocos `extensions` no arquivo JSON.</span><span class="sxs-lookup"><span data-stu-id="0b641-290">If you change the adminstrator user name, you must also update the `extensions` blocks in the JSON file.</span></span> 

9. <span data-ttu-id="0b641-291">Execute o seguinte comando para implantar a arquitetura.</span><span class="sxs-lookup"><span data-stu-id="0b641-291">Run the following command to deploy the architecture.</span></span>

    ```bash
    azbb -s <your subscription_id> -g <resource_group_name> -l <location> -p n-tier-windows.json --deploy
    ```

<span data-ttu-id="0b641-292">Para obter mais informações sobre a implantação dessa arquitetura de referência de exemplo utilizando blocos de construção Blocos de Construção do Azure, visite o repositório [GitHub][git].</span><span class="sxs-lookup"><span data-stu-id="0b641-292">For more information on deploying this sample reference architecture using Azure Building Blocks, visit the [GitHub repository][git].</span></span>


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-sql-server.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[host bastião]: https://en.wikipedia.org/wiki/Bastion_host
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[endereço IP público]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[public IP address]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-sql-server.png "Arquitetura de N camadas usando o Microsoft Azure"
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[subscription-limits]: /azure/azure-subscription-service-limits
[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[network-security]: /azure/best-practices-network-security
