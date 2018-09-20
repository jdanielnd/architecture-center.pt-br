---
title: SAP para cargas de trabalho de desenvolvimento/teste
description: Cenário SAP para um ambiente de desenvolvimento/teste
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: d0f266e40969cf4782e69041889a686387499722
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389172"
---
# <a name="sap-for-devtest-workloads"></a><span data-ttu-id="c5ebc-103">SAP para cargas de trabalho de desenvolvimento/teste</span><span class="sxs-lookup"><span data-stu-id="c5ebc-103">SAP for dev/test workloads</span></span>

<span data-ttu-id="c5ebc-104">Este exemplo fornece orientações sobre como executar uma implementação de desenvolvimento/teste do SAP NetWeaver em um ambiente Windows ou Linux no Azure.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-104">This example provides guidance for how to run a dev/test implementation of SAP NetWeaver in a Windows or Linux environment on Azure.</span></span> <span data-ttu-id="c5ebc-105">O banco de dados usado é o AnyDB, o termo SAP para qualquer DBMS compatível (que não seja SAP HANA).</span><span class="sxs-lookup"><span data-stu-id="c5ebc-105">The database used is AnyDB, the SAP term for any supported DBMS (that isn't SAP HANA).</span></span> <span data-ttu-id="c5ebc-106">Como essa arquitetura foi projetada para ambientes que não são de produção, ele é implantado com somente uma máquina virtual (VM) e o seu tamanho pode ser alterado para acomodar as necessidades da sua organização.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-106">Because this architecture is designed for non-production environments, it's deployed with just a single virtual machine (VM) and it's size can be changed to accommodate your organization's needs.</span></span>

<span data-ttu-id="c5ebc-107">Para casos de uso de produção veja as arquiteturas SAP de referência disponíveis abaixo:</span><span class="sxs-lookup"><span data-stu-id="c5ebc-107">For production use cases review the SAP reference architectures available below:</span></span>

* <span data-ttu-id="c5ebc-108">[SAP Netweaver para AnyDB][sap-netweaver]</span><span class="sxs-lookup"><span data-stu-id="c5ebc-108">[SAP netweaver for AnyDB][sap-netweaver]</span></span>
* <span data-ttu-id="c5ebc-109">[SAP S/4Hana][sap-hana]</span><span class="sxs-lookup"><span data-stu-id="c5ebc-109">[SAP S/4Hana][sap-hana]</span></span>
* <span data-ttu-id="c5ebc-110">[SAP em instâncias grandes do Azure][sap-large]</span><span class="sxs-lookup"><span data-stu-id="c5ebc-110">[SAP on Azure large instances][sap-large]</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="c5ebc-111">Casos de uso relacionados</span><span class="sxs-lookup"><span data-stu-id="c5ebc-111">Related use cases</span></span>

<span data-ttu-id="c5ebc-112">Considere este cenário para os casos de uso a seguir:</span><span class="sxs-lookup"><span data-stu-id="c5ebc-112">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="c5ebc-113">Cargas de trabalho do SAP não críticas e que não são de produção (área restrita, desenvolvimento, teste, garantia de qualidade)</span><span class="sxs-lookup"><span data-stu-id="c5ebc-113">Non-critical SAP non-productive workloads (sandbox, development, test, quality assurance)</span></span>
* <span data-ttu-id="c5ebc-114">Cargas de trabalho não críticas do SAP Business One</span><span class="sxs-lookup"><span data-stu-id="c5ebc-114">Non-critical SAP business one workloads</span></span>

## <a name="architecture"></a><span data-ttu-id="c5ebc-115">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="c5ebc-115">Architecture</span></span>

![Diagrama](media/sap-2tier/SAP-Infra-2Tier_finalversion.png)

<span data-ttu-id="c5ebc-117">Este cenário cobre o provisionamento de um banco de dados do sistema SAP individual e um Servidor de Aplicativos do SAP em uma única máquina virtual, o fluxo de dados ocorre da seguinte forma neste cenário:</span><span class="sxs-lookup"><span data-stu-id="c5ebc-117">This scenario covers the provision of a single SAP system database and SAP application Server on a single virtual machine, the data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="c5ebc-118">Os clientes da camada de apresentação utilizam a GUI do SAP ou outras interfaces de usuário (Internet Explorer, Excel ou outro aplicativo Web) locais para acessar o sistema SAP baseado no Azure.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-118">Customers from the Presentation Tier use their SAP GUI, or other user interfaces (Internet Explorer, Excel, or other web application) on premise to access the Azure-based SAP system.</span></span>
2. <span data-ttu-id="c5ebc-119">A conectividade é fornecida por meio do uso do Express Route estabelecido.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-119">Connectivity is provided through the use of an established Express Route.</span></span> <span data-ttu-id="c5ebc-120">A conexão do Express Route é terminada no Azure no gateway do Express Route.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-120">The Express Route connection is terminated in Azure at the Express Route Gateway.</span></span> <span data-ttu-id="c5ebc-121">O tráfego de rede é roteado através do gateway do Express Route para a sub-rede do gateway e desta para a sub-rede spoke da camada de aplicativo (consulte o padrão [hub-spoke][hub-spoke] pattern) e através de um gateway de segurança de rede até a máquina virtual do aplicativo SAP.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-121">Network traffic routes through the Express Route gateway to the Gateway Subnet and from the gateway subnet to the Application Tier Spoke subnet (see the [hub-spoke][hub-spoke] pattern) and via a Network Security Gateway to the SAP application virtual machine.</span></span>
3. <span data-ttu-id="c5ebc-122">Os servidores de gerenciamento de identidade oferecem serviços de autenticação.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-122">The identity management servers provide authentication services.</span></span>
4. <span data-ttu-id="c5ebc-123">A caixa de atalhos oferece recursos de gerenciamento local.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-123">The jump box provides local management capabilities.</span></span>

### <a name="components"></a><span data-ttu-id="c5ebc-124">Componentes</span><span class="sxs-lookup"><span data-stu-id="c5ebc-124">Components</span></span>

* <span data-ttu-id="c5ebc-125">[Grupos de recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups) são contêineres lógicos para recursos do Azure.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-125">[Resource Groups](/azure/azure-resource-manager/resource-group-overview#resource-groups) is a logical container for Azure resources.</span></span>
* <span data-ttu-id="c5ebc-126">[Redes virtuais](/azure/virtual-network/virtual-networks-overview) são a base das comunicações de rede no Azure</span><span class="sxs-lookup"><span data-stu-id="c5ebc-126">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) is the basis of network communications within Azure</span></span>
* <span data-ttu-id="c5ebc-127">[Máquinas virtuais](/azure/virtual-machines/windows/overview) no Azure oferecem uma infraestrutura sob demanda, de alta capacidade de dimensionamento, virtualizada que usa um servidor Windows ou Linux</span><span class="sxs-lookup"><span data-stu-id="c5ebc-127">[Virtual Machine](/azure/virtual-machines/windows/overview) Azure Virtual Machines provides on-demand, high-scale, secure, virtualized infrastructure using Windows or Linux Server</span></span>
* <span data-ttu-id="c5ebc-128">[Express Route](/azure/expressroute/expressroute-introduction) permite estender as redes locais na nuvem da Microsoft para uma conexão privada facilitada por um provedor de conectividade.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-128">[Express Route](/azure/expressroute/expressroute-introduction) lets you extend your on-premises networks into the Microsoft cloud over a private connection facilitated by a connectivity provider.</span></span>
* <span data-ttu-id="c5ebc-129">[Grupo de segurança de rede](/azure/virtual-network/security-overview) permite limitar o tráfego de rede para recursos em uma rede virtual.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-129">[Network Security Group](/azure/virtual-network/security-overview) lets you limit network traffic to resources in a virtual network.</span></span> <span data-ttu-id="c5ebc-130">Um grupo de segurança de rede contém uma lista de regras de segurança que permitem ou negam o tráfego de rede de entrada ou saída com base no endereço IP de origem ou destino, na porta e no protocolo.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-130">A network security group contains a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol.</span></span> 

## <a name="considerations"></a><span data-ttu-id="c5ebc-131">Considerações</span><span class="sxs-lookup"><span data-stu-id="c5ebc-131">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="c5ebc-132">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="c5ebc-132">Availability</span></span>

 <span data-ttu-id="c5ebc-133">A Microsoft oferece um contrato de nível de serviço (SLA) para instância de VM individuais.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-133">Microsoft offers a service level agreement (SLA) for single VM instances.</span></span> <span data-ttu-id="c5ebc-134">Para saber mais informações sobre o contrato de nível de serviço do Microsoft Azure para máquinas virtuais consulte [SLA para máquinas virtuais](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span><span class="sxs-lookup"><span data-stu-id="c5ebc-134">For more information on Microsoft Azure Service Level Agreement for Virtual Machines [SLA For Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span></span>

### <a name="scalability"></a><span data-ttu-id="c5ebc-135">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="c5ebc-135">Scalability</span></span>

<span data-ttu-id="c5ebc-136">Para obter diretrizes gerais sobre como criar soluções escalonáveis, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-136">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="c5ebc-137">Segurança</span><span class="sxs-lookup"><span data-stu-id="c5ebc-137">Security</span></span>

<span data-ttu-id="c5ebc-138">Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].</span><span class="sxs-lookup"><span data-stu-id="c5ebc-138">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="c5ebc-139">Resiliência</span><span class="sxs-lookup"><span data-stu-id="c5ebc-139">Resiliency</span></span>

<span data-ttu-id="c5ebc-140">Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="c5ebc-140">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="c5ebc-141">Preços</span><span class="sxs-lookup"><span data-stu-id="c5ebc-141">Pricing</span></span>

<span data-ttu-id="c5ebc-142">Explore o custo de executar esse cenário, todos os serviços são pré-configurados na calculadora de custos.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-142">Explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span>  <span data-ttu-id="c5ebc-143">Para ver como o preço seria alterado para o seu uso específico altere as variáveis apropriadas para que eles sejam correspondentes ao tráfego esperada.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-143">To see how the pricing would change for your particular use case change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="c5ebc-144">Fornecemos quatro exemplos de perfis de custo com base na quantidade de tráfego que você espera obter:</span><span class="sxs-lookup"><span data-stu-id="c5ebc-144">We have provided four sample cost profiles based on amount of traffic you expect to get:</span></span>

|<span data-ttu-id="c5ebc-145">Tamanho</span><span class="sxs-lookup"><span data-stu-id="c5ebc-145">Size</span></span>|<span data-ttu-id="c5ebc-146">SAPs</span><span class="sxs-lookup"><span data-stu-id="c5ebc-146">SAPs</span></span>|<span data-ttu-id="c5ebc-147">Tipo de VM</span><span class="sxs-lookup"><span data-stu-id="c5ebc-147">VM Type</span></span>|<span data-ttu-id="c5ebc-148">Armazenamento</span><span class="sxs-lookup"><span data-stu-id="c5ebc-148">Storage</span></span>|<span data-ttu-id="c5ebc-149">Calculadora de Preços do Azure</span><span class="sxs-lookup"><span data-stu-id="c5ebc-149">Azure Pricing Calculator</span></span>|
|----|----|-------|-------|---------------|
|<span data-ttu-id="c5ebc-150">Pequena</span><span class="sxs-lookup"><span data-stu-id="c5ebc-150">Small</span></span>|<span data-ttu-id="c5ebc-151">8000</span><span class="sxs-lookup"><span data-stu-id="c5ebc-151">8000</span></span>|<span data-ttu-id="c5ebc-152">D8s_v3</span><span class="sxs-lookup"><span data-stu-id="c5ebc-152">D8s_v3</span></span>|<span data-ttu-id="c5ebc-153">2xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="c5ebc-153">2xP20, 1xP10</span></span>|[<span data-ttu-id="c5ebc-154">Pequeno</span><span class="sxs-lookup"><span data-stu-id="c5ebc-154">Small</span></span>](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|<span data-ttu-id="c5ebc-155">Média</span><span class="sxs-lookup"><span data-stu-id="c5ebc-155">Medium</span></span>|<span data-ttu-id="c5ebc-156">16000</span><span class="sxs-lookup"><span data-stu-id="c5ebc-156">16000</span></span>|<span data-ttu-id="c5ebc-157">D16s_v3</span><span class="sxs-lookup"><span data-stu-id="c5ebc-157">D16s_v3</span></span>|<span data-ttu-id="c5ebc-158">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="c5ebc-158">3xP20, 1xP10</span></span>|[<span data-ttu-id="c5ebc-159">Médio</span><span class="sxs-lookup"><span data-stu-id="c5ebc-159">Medium</span></span>](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
<span data-ttu-id="c5ebc-160">grande</span><span class="sxs-lookup"><span data-stu-id="c5ebc-160">Large</span></span>|<span data-ttu-id="c5ebc-161">32000</span><span class="sxs-lookup"><span data-stu-id="c5ebc-161">32000</span></span>|<span data-ttu-id="c5ebc-162">E32s_v3</span><span class="sxs-lookup"><span data-stu-id="c5ebc-162">E32s_v3</span></span>|<span data-ttu-id="c5ebc-163">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="c5ebc-163">3xP20, 1xP10</span></span>|[<span data-ttu-id="c5ebc-164">Grande</span><span class="sxs-lookup"><span data-stu-id="c5ebc-164">Large</span></span>](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
<span data-ttu-id="c5ebc-165">Extra grande</span><span class="sxs-lookup"><span data-stu-id="c5ebc-165">Extra Large</span></span>|<span data-ttu-id="c5ebc-166">64000</span><span class="sxs-lookup"><span data-stu-id="c5ebc-166">64000</span></span>|<span data-ttu-id="c5ebc-167">M64s</span><span class="sxs-lookup"><span data-stu-id="c5ebc-167">M64s</span></span>|<span data-ttu-id="c5ebc-168">4xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="c5ebc-168">4xP20, 1xP10</span></span>|[<span data-ttu-id="c5ebc-169">Extra grande</span><span class="sxs-lookup"><span data-stu-id="c5ebc-169">Extra Large</span></span>](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

<span data-ttu-id="c5ebc-170">Observação: o preço é um guia e indica apenas os custos de armazenamento e VMs (exclui cobranças de rede, armazenamento de backup e entrada/saída de dados).</span><span class="sxs-lookup"><span data-stu-id="c5ebc-170">Note: pricing is a guide and only indicates the VMs and storage costs (excludes, networking, backup storage, and data ingress/egress charges).</span></span>

* <span data-ttu-id="c5ebc-171">[Pequeno](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): um sistema pequeno é composto por uma VM do tipo D8s_v3 com 8x vCPUs, 32 GB de RAM e 200 GB de armazenamento temporário, além de dois discos de armazenamento premium de 512 GB e um de 128 GB.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-171">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): A small system consists of VM type D8s_v3 with 8x vCPUs, 32 GB RAM and 200 GB temp storage, additionally two 512 GB and one 128 GB premium storage disks.</span></span>
* <span data-ttu-id="c5ebc-172">[Médio](https://azure.com/e/465bd07047d148baab032b2f461550cd): um sistema médio é composto por uma VM do tipo D16s_v3 com 16x vCPUs, 64 GB de RAM e 400 GB de armazenamento temporário, além de três discos de armazenamento premium de 512 GB e um de 128 GB.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-172">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): A medium system consists of VM type D16s_v3 with 16x vCPUs, 64 GB RAM and 400 GB temp storage, additionally three 512 GB and one 128 GB premium storage disks.</span></span>
* <span data-ttu-id="c5ebc-173">[Grande](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): um sistema grande é composto por uma VM do tipo E32s_v3 com 32x vCPUs, 256 GB de RAM e 512 GB de armazenamento temporário, além de três discos de armazenamento premium de 512 GB e um de 128 GB.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-173">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): A large system consists of VM type E32s_v3 with 32x vCPUs, 256 GB RAM and 512 GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="c5ebc-174">[Extragrande](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): um sistema extragrande é composto por uma VM do tipo M64s com 64x vCPUs, 1.024 GB de RAM e 2.000 GB de armazenamento temporário, além de quatro discos de armazenamento premium de 512 GB e um de 128 GB.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-174">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): An extra large system consists of a VM type M64s with 64x vCPUs, 1024 GB RAM and 2000 GB temp storage, additionally four 512 GB and one 128 GB premium storage disks.</span></span>

## <a name="deployment"></a><span data-ttu-id="c5ebc-175">Implantação</span><span class="sxs-lookup"><span data-stu-id="c5ebc-175">Deployment</span></span>

<span data-ttu-id="c5ebc-176">Para implantar a infraestrutura subjacente semelhante ao cenário acima, use o botão de implantação</span><span class="sxs-lookup"><span data-stu-id="c5ebc-176">To deploy the underlying infrastructure similar to the scenario above, use the deploy button</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="c5ebc-177">\*O SAP não será automaticamente instalado, instale-o manualmente após a infraestrutura ser compilada.</span><span class="sxs-lookup"><span data-stu-id="c5ebc-177">\* SAP won't be automatically installed, manually install it after the infrastructure has been built.</span></span>

<!-- links -->
[reference architecture]:  /azure/architecture/reference-architectures/sap
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke