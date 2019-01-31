---
title: Ambientes de desenvolvimento/teste para cargas de trabalho do SAP
titleSuffix: Azure Example Scenarios
description: Crie um ambiente de desenvolvimento/teste para cargas de trabalho do SAP.
author: AndrewDibbins
ms.date: 07/11/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack, SAP
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-sap-dev-test.png
ms.openlocfilehash: cd01639ab46fbe3c25c5a770ead13759da9a8c47
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908168"
---
# <a name="devtest-environments-for-sap-workloads-on-azure"></a><span data-ttu-id="c315f-103">Ambientes de desenvolvimento/teste para cargas de trabalho do SAP no Azure</span><span class="sxs-lookup"><span data-stu-id="c315f-103">Dev/test environments for SAP workloads on Azure</span></span>

<span data-ttu-id="c315f-104">Este exemplo mostra como estabelecer um ambiente de desenvolvimento/teste do SAP NetWeaver em um ambiente Windows ou Linux no Azure.</span><span class="sxs-lookup"><span data-stu-id="c315f-104">This example shows how to establish a dev/test environment for SAP NetWeaver in a Windows or Linux environment on Azure.</span></span> <span data-ttu-id="c315f-105">O banco de dados usado é o AnyDB, o termo SAP para qualquer DBMS compatível (que não seja SAP HANA).</span><span class="sxs-lookup"><span data-stu-id="c315f-105">The database used is AnyDB, the SAP term for any supported DBMS (that isn't SAP HANA).</span></span> <span data-ttu-id="c315f-106">Como essa arquitetura foi projetada para ambientes que não são de produção, ele é implantado com somente uma máquina virtual (VM) e o seu tamanho pode ser alterado para acomodar as necessidades da sua organização.</span><span class="sxs-lookup"><span data-stu-id="c315f-106">Because this architecture is designed for non-production environments, it's deployed with just a single virtual machine (VM) and it's size can be changed to accommodate your organization's needs.</span></span>

<span data-ttu-id="c315f-107">Para casos de uso de produção veja as arquiteturas SAP de referência disponíveis abaixo:</span><span class="sxs-lookup"><span data-stu-id="c315f-107">For production use cases review the SAP reference architectures available below:</span></span>

- <span data-ttu-id="c315f-108">[SAP NetWeaver para AnyDB][sap-netweaver]</span><span class="sxs-lookup"><span data-stu-id="c315f-108">[SAP NetWeaver for AnyDB][sap-netweaver]</span></span>
- <span data-ttu-id="c315f-109">[SAP S/4HANA][sap-hana]</span><span class="sxs-lookup"><span data-stu-id="c315f-109">[SAP S/4HANA][sap-hana]</span></span>
- <span data-ttu-id="c315f-110">[SAP em instâncias grandes do Azure][sap-large]</span><span class="sxs-lookup"><span data-stu-id="c315f-110">[SAP on Azure large instances][sap-large]</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="c315f-111">Casos de uso relevantes</span><span class="sxs-lookup"><span data-stu-id="c315f-111">Relevant use cases</span></span>

<span data-ttu-id="c315f-112">Outros casos de uso relevantes incluem:</span><span class="sxs-lookup"><span data-stu-id="c315f-112">Other relevant use cases include:</span></span>

- <span data-ttu-id="c315f-113">Cargas de trabalho do SAP não críticas e que não são de produção (área restrita, desenvolvimento, teste, garantia de qualidade)</span><span class="sxs-lookup"><span data-stu-id="c315f-113">Non-critical SAP non-productive workloads (sandbox, development, test, quality assurance)</span></span>
- <span data-ttu-id="c315f-114">Cargas de trabalho de negócios não críticas do SAP</span><span class="sxs-lookup"><span data-stu-id="c315f-114">Non-critical SAP business workloads</span></span>

## <a name="architecture"></a><span data-ttu-id="c315f-115">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="c315f-115">Architecture</span></span>

![Diagrama de arquitetura para ambientes de desenvolvimento/teste para cargas de trabalho do SAP](./media/architecture-sap-dev-test.png)

<span data-ttu-id="c315f-117">Este cenário demonstra o provisionamento de um único banco de dados do sistema do SAP e o servidor de aplicativos SAP em uma única máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="c315f-117">This scenario demonstrates provisioning a single SAP system database and SAP application server on a single virtual machine.</span></span> <span data-ttu-id="c315f-118">O fluxo de dados neste cenário ocorre da seguinte forma:</span><span class="sxs-lookup"><span data-stu-id="c315f-118">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="c315f-119">Os clientes usam a interface do usuário do SAP ou outras ferramentas de cliente (Excel, um navegador da Web ou outro aplicativo Web) para acessar o sistema SAP com base no Azure.</span><span class="sxs-lookup"><span data-stu-id="c315f-119">Customers use the SAP user interface or other client tools (Excel, a web browser, or other web application) to access the Azure-based SAP system.</span></span>
2. <span data-ttu-id="c315f-120">A conectividade é fornecida por meio do uso do ExpressRoute estabelecido.</span><span class="sxs-lookup"><span data-stu-id="c315f-120">Connectivity is provided through the use of an established ExpressRoute.</span></span> <span data-ttu-id="c315f-121">A conexão do ExpressRoute é terminada no Azure no gateway do ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="c315f-121">The ExpressRoute connection is terminated in Azure at the ExpressRoute gateway.</span></span> <span data-ttu-id="c315f-122">O tráfego de rede é roteado pelo gateway do ExpressRoute para a sub-rede do gateway e desta para a sub-rede spoke da camada de aplicativo (confira a [topologia de rede hub-spoke][hub-spoke]) e por um Gateway de Segurança de Rede até a máquina virtual do aplicativo SAP.</span><span class="sxs-lookup"><span data-stu-id="c315f-122">Network traffic routes through the ExpressRoute gateway to the gateway subnet, and from the gateway subnet to the application-tier spoke subnet (see the [hub-spoke network topology][hub-spoke]) and via a Network Security Gateway to the SAP application virtual machine.</span></span>
3. <span data-ttu-id="c315f-123">Os servidores de gerenciamento de identidade oferecem serviços de autenticação.</span><span class="sxs-lookup"><span data-stu-id="c315f-123">The identity management servers provide authentication services.</span></span>
4. <span data-ttu-id="c315f-124">A caixa de atalhos oferece recursos de gerenciamento local.</span><span class="sxs-lookup"><span data-stu-id="c315f-124">The jump box provides local management capabilities.</span></span>

### <a name="components"></a><span data-ttu-id="c315f-125">Componentes</span><span class="sxs-lookup"><span data-stu-id="c315f-125">Components</span></span>

- <span data-ttu-id="c315f-126">As [Redes Virtuais](/azure/virtual-network/virtual-networks-overview) são a base das comunicações de rede no Azure.</span><span class="sxs-lookup"><span data-stu-id="c315f-126">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) are the basis of network communication within Azure.</span></span>
- <span data-ttu-id="c315f-127">As [Máquinas Virtuais](/azure/virtual-machines/windows/overview) do Azure oferecem uma infraestrutura sob demanda, de alta capacidade de dimensionamento, virtualizada que usa um servidor Windows ou Linux.</span><span class="sxs-lookup"><span data-stu-id="c315f-127">[Virtual Machine](/azure/virtual-machines/windows/overview) Azure Virtual Machines provides on-demand, high-scale, secure, virtualized infrastructure using Windows or Linux Server.</span></span>
- <span data-ttu-id="c315f-128">O [ExpressRoute](/azure/expressroute/expressroute-introduction) permite que você estenda suas redes locais até a nuvem da Microsoft por meio de conexão privada facilitada por um provedor de conectividade.</span><span class="sxs-lookup"><span data-stu-id="c315f-128">[ExpressRoute](/azure/expressroute/expressroute-introduction) lets you extend your on-premises networks into the Microsoft cloud over a private connection facilitated by a connectivity provider.</span></span>
- <span data-ttu-id="c315f-129">[Grupo de segurança de rede](/azure/virtual-network/security-overview) permite limitar o tráfego de rede para recursos em uma rede virtual.</span><span class="sxs-lookup"><span data-stu-id="c315f-129">[Network Security Group](/azure/virtual-network/security-overview) lets you limit network traffic to resources in a virtual network.</span></span> <span data-ttu-id="c315f-130">Um grupo de segurança de rede contém uma lista de regras de segurança que permitem ou negam o tráfego de rede de entrada ou saída com base no endereço IP de origem ou destino, na porta e no protocolo.</span><span class="sxs-lookup"><span data-stu-id="c315f-130">A network security group contains a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol.</span></span>
- <span data-ttu-id="c315f-131">Os [Grupos de Recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups) atuam como contêineres lógicos para recursos do Azure.</span><span class="sxs-lookup"><span data-stu-id="c315f-131">[Resource Groups](/azure/azure-resource-manager/resource-group-overview#resource-groups) act as logical containers for Azure resources.</span></span>

## <a name="considerations"></a><span data-ttu-id="c315f-132">Considerações</span><span class="sxs-lookup"><span data-stu-id="c315f-132">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="c315f-133">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="c315f-133">Availability</span></span>

<span data-ttu-id="c315f-134">A Microsoft oferece um contrato de nível de serviço (SLA) para instância de VM individuais.</span><span class="sxs-lookup"><span data-stu-id="c315f-134">Microsoft offers a service level agreement (SLA) for single VM instances.</span></span> <span data-ttu-id="c315f-135">Para saber mais informações sobre o contrato de nível de serviço do Microsoft Azure para máquinas virtuais consulte [SLA para máquinas virtuais](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span><span class="sxs-lookup"><span data-stu-id="c315f-135">For more information on Microsoft Azure Service Level Agreement for Virtual Machines [SLA For Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span></span>

### <a name="scalability"></a><span data-ttu-id="c315f-136">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="c315f-136">Scalability</span></span>

<span data-ttu-id="c315f-137">Para obter diretrizes gerais sobre como criar soluções escalonáveis, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.</span><span class="sxs-lookup"><span data-stu-id="c315f-137">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="c315f-138">Segurança</span><span class="sxs-lookup"><span data-stu-id="c315f-138">Security</span></span>

<span data-ttu-id="c315f-139">Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].</span><span class="sxs-lookup"><span data-stu-id="c315f-139">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="c315f-140">Resiliência</span><span class="sxs-lookup"><span data-stu-id="c315f-140">Resiliency</span></span>

<span data-ttu-id="c315f-141">Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="c315f-141">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="c315f-142">Preços</span><span class="sxs-lookup"><span data-stu-id="c315f-142">Pricing</span></span>

<span data-ttu-id="c315f-143">Para ajudá-lo a explorar o custo da execução nesse cenário, todos os serviços são pré-configurados nos exemplos de calculadora de custos abaixo.</span><span class="sxs-lookup"><span data-stu-id="c315f-143">To help you explore the cost of running this scenario, all of the services are pre-configured in the cost calculator examples below.</span></span> <span data-ttu-id="c315f-144">Para ver como o preço seria alterado em seu caso de uso específico, altere as variáveis apropriadas de acordo com o tráfego esperado.</span><span class="sxs-lookup"><span data-stu-id="c315f-144">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="c315f-145">Fornecemos quatro exemplos de perfis de custo com base na quantidade de tráfego que você espera receber:</span><span class="sxs-lookup"><span data-stu-id="c315f-145">We have provided four sample cost profiles based on amount of traffic you expect to receive:</span></span>

|<span data-ttu-id="c315f-146">Tamanho</span><span class="sxs-lookup"><span data-stu-id="c315f-146">Size</span></span>|<span data-ttu-id="c315f-147">SAPs</span><span class="sxs-lookup"><span data-stu-id="c315f-147">SAPs</span></span>|<span data-ttu-id="c315f-148">Tipo de VM</span><span class="sxs-lookup"><span data-stu-id="c315f-148">VM Type</span></span>|<span data-ttu-id="c315f-149">Armazenamento</span><span class="sxs-lookup"><span data-stu-id="c315f-149">Storage</span></span>|<span data-ttu-id="c315f-150">Calculadora de Preços do Azure</span><span class="sxs-lookup"><span data-stu-id="c315f-150">Azure Pricing Calculator</span></span>|
|----|----|-------|-------|---------------|
|<span data-ttu-id="c315f-151">Pequena</span><span class="sxs-lookup"><span data-stu-id="c315f-151">Small</span></span>|<span data-ttu-id="c315f-152">8000</span><span class="sxs-lookup"><span data-stu-id="c315f-152">8000</span></span>|<span data-ttu-id="c315f-153">D8s_v3</span><span class="sxs-lookup"><span data-stu-id="c315f-153">D8s_v3</span></span>|<span data-ttu-id="c315f-154">2xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="c315f-154">2xP20, 1xP10</span></span>|[<span data-ttu-id="c315f-155">Pequeno</span><span class="sxs-lookup"><span data-stu-id="c315f-155">Small</span></span>](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|<span data-ttu-id="c315f-156">Média</span><span class="sxs-lookup"><span data-stu-id="c315f-156">Medium</span></span>|<span data-ttu-id="c315f-157">16000</span><span class="sxs-lookup"><span data-stu-id="c315f-157">16000</span></span>|<span data-ttu-id="c315f-158">D16s_v3</span><span class="sxs-lookup"><span data-stu-id="c315f-158">D16s_v3</span></span>|<span data-ttu-id="c315f-159">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="c315f-159">3xP20, 1xP10</span></span>|[<span data-ttu-id="c315f-160">Médio</span><span class="sxs-lookup"><span data-stu-id="c315f-160">Medium</span></span>](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
<span data-ttu-id="c315f-161">grande</span><span class="sxs-lookup"><span data-stu-id="c315f-161">Large</span></span>|<span data-ttu-id="c315f-162">32000</span><span class="sxs-lookup"><span data-stu-id="c315f-162">32000</span></span>|<span data-ttu-id="c315f-163">E32s_v3</span><span class="sxs-lookup"><span data-stu-id="c315f-163">E32s_v3</span></span>|<span data-ttu-id="c315f-164">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="c315f-164">3xP20, 1xP10</span></span>|[<span data-ttu-id="c315f-165">Grande</span><span class="sxs-lookup"><span data-stu-id="c315f-165">Large</span></span>](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
<span data-ttu-id="c315f-166">Extra grande</span><span class="sxs-lookup"><span data-stu-id="c315f-166">Extra Large</span></span>|<span data-ttu-id="c315f-167">64000</span><span class="sxs-lookup"><span data-stu-id="c315f-167">64000</span></span>|<span data-ttu-id="c315f-168">M64s</span><span class="sxs-lookup"><span data-stu-id="c315f-168">M64s</span></span>|<span data-ttu-id="c315f-169">4xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="c315f-169">4xP20, 1xP10</span></span>|[<span data-ttu-id="c315f-170">Extra grande</span><span class="sxs-lookup"><span data-stu-id="c315f-170">Extra Large</span></span>](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

> [!NOTE]
> <span data-ttu-id="c315f-171">Esse preço é um guia que indica somente as VMs e os custos de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="c315f-171">This pricing is a guide that only indicates the VMs and storage costs.</span></span> <span data-ttu-id="c315f-172">Isso exclui rede, armazenamento de backup e encargos de entrada/saída de dados.</span><span class="sxs-lookup"><span data-stu-id="c315f-172">It excludes networking, backup storage, and data ingress/egress charges.</span></span>

- <span data-ttu-id="c315f-173">[Pequeno](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): um sistema pequeno é composto por uma VM do tipo D8s_v3 com 8x vCPUs, 32 GB de RAM e 200 GB de armazenamento temporário, além de dois discos de armazenamento premium de 512 GB e um de 128 GB.</span><span class="sxs-lookup"><span data-stu-id="c315f-173">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): A small system consists of VM type D8s_v3 with 8x vCPUs, 32 GB RAM and 200 GB temp storage, additionally two 512 GB and one 128 GB premium storage disks.</span></span>
- <span data-ttu-id="c315f-174">[Médio](https://azure.com/e/465bd07047d148baab032b2f461550cd): um sistema médio é composto por uma VM do tipo D16s_v3 com 16x vCPUs, 64 GB de RAM e 400 GB de armazenamento temporário, além de três discos de armazenamento premium de 512 GB e um de 128 GB.</span><span class="sxs-lookup"><span data-stu-id="c315f-174">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): A medium system consists of VM type D16s_v3 with 16x vCPUs, 64 GB RAM and 400 GB temp storage, additionally three 512 GB and one 128 GB premium storage disks.</span></span>
- <span data-ttu-id="c315f-175">[Grande](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): um sistema grande é composto por uma VM do tipo E32s_v3 com 32x vCPUs, 256 GB de RAM e 512 GB de armazenamento temporário, além de três discos de armazenamento premium de 512 GB e um de 128 GB.</span><span class="sxs-lookup"><span data-stu-id="c315f-175">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): A large system consists of VM type E32s_v3 with 32x vCPUs, 256 GB RAM and 512 GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
- <span data-ttu-id="c315f-176">[Extra grande](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): um sistema extragrande é composto por uma VM do tipo M64s com 64x vCPUs, 1024 GB de RAM e 2000 GB de armazenamento temporário, além de quatro discos de armazenamento premium de 512 GB e um de 128 GB.</span><span class="sxs-lookup"><span data-stu-id="c315f-176">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): An extra large system consists of a VM type M64s with 64x vCPUs, 1024 GB RAM and 2000 GB temp storage, additionally four 512 GB and one 128 GB premium storage disks.</span></span>

## <a name="deployment"></a><span data-ttu-id="c315f-177">Implantação</span><span class="sxs-lookup"><span data-stu-id="c315f-177">Deployment</span></span>

<span data-ttu-id="c315f-178">Clique aqui para implantar a infraestrutura subjacente para esse cenário.</span><span class="sxs-lookup"><span data-stu-id="c315f-178">Click here to deploy the underlying infrastructure for this scenario.</span></span>

<!-- markdownlint-disable MD033 -->

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

<!-- markdownlint-enable MD033 -->

> [!NOTE]
> <span data-ttu-id="c315f-179">SAP e Oracle não são instalados durante essa implantação.</span><span class="sxs-lookup"><span data-stu-id="c315f-179">SAP and Oracle are not installed during this deployment.</span></span> <span data-ttu-id="c315f-180">Você deve implantar esses componentes separadamente.</span><span class="sxs-lookup"><span data-stu-id="c315f-180">You will need to deploy these components separately.</span></span>

<!-- links -->
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke
