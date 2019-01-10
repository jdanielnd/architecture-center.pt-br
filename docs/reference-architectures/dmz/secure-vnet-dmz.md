---
title: Implementar uma rede de perímetro entre o Azure e a Internet
titleSuffix: Azure Reference Architectures
description: Como implementar uma arquitetura de rede híbrida segura com acesso à Internet no Azure.
author: telmosampaio
ms.date: 10/22/2018
ms.custom: seodec18
ms.openlocfilehash: 10c8a23ab09da0555de6a51bc082deceb8c462ff
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011524"
---
# <a name="implement-a-dmz-between-azure-and-the-internet"></a><span data-ttu-id="f5d62-103">Implementar uma rede de perímetro entre o Azure e a Internet</span><span class="sxs-lookup"><span data-stu-id="f5d62-103">Implement a DMZ between Azure and the Internet</span></span>

<span data-ttu-id="f5d62-104">Essa arquitetura de referência mostra uma rede híbrido segura que estende a uma rede local para o Azure e também aceita tráfego de Internet.</span><span class="sxs-lookup"><span data-stu-id="f5d62-104">This reference architecture shows a secure hybrid network that extends an on-premises network to Azure and also accepts Internet traffic.</span></span> <span data-ttu-id="f5d62-105">[**Implantar esta solução**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="f5d62-105">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Proteger arquitetura de rede híbrida](./images/dmz-public.png)

<span data-ttu-id="f5d62-107">*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*</span><span class="sxs-lookup"><span data-stu-id="f5d62-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="f5d62-108">Essa arquitetura de referência estende a arquitetura descrita em [Implementação de uma DMZ entre o Azure e seu datacenter local][implementing-a-secure-hybrid-network-architecture].</span><span class="sxs-lookup"><span data-stu-id="f5d62-108">This reference architecture extends the architecture described in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture].</span></span> <span data-ttu-id="f5d62-109">Ela adiciona uma rede de perímetro que manipula o tráfego da Internet, além da DMZ privada que manipula o tráfego da rede local.</span><span class="sxs-lookup"><span data-stu-id="f5d62-109">It adds a public DMZ that handles Internet traffic, in addition to the private DMZ that handles traffic from the on-premises network.</span></span>

<span data-ttu-id="f5d62-110">Alguns usos típicos dessa arquitetura:</span><span class="sxs-lookup"><span data-stu-id="f5d62-110">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="f5d62-111">Aplicativos híbridos nos quais as cargas de trabalho são executadas parcialmente localmente e parcialmente no Azure.</span><span class="sxs-lookup"><span data-stu-id="f5d62-111">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
- <span data-ttu-id="f5d62-112">A infraestrutura do Azure que roteia o tráfego de entrada local e da Internet.</span><span class="sxs-lookup"><span data-stu-id="f5d62-112">Azure infrastructure that routes incoming traffic from on-premises and the Internet.</span></span>

## <a name="architecture"></a><span data-ttu-id="f5d62-113">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="f5d62-113">Architecture</span></span>

<span data-ttu-id="f5d62-114">A arquitetura consiste nos componentes a seguir.</span><span class="sxs-lookup"><span data-stu-id="f5d62-114">The architecture consists of the following components.</span></span>

- <span data-ttu-id="f5d62-115">**Endereço IP público (PIP)**.</span><span class="sxs-lookup"><span data-stu-id="f5d62-115">**Public IP address (PIP)**.</span></span> <span data-ttu-id="f5d62-116">O endereço IP do ponto de extremidade público.</span><span class="sxs-lookup"><span data-stu-id="f5d62-116">The IP address of the public endpoint.</span></span> <span data-ttu-id="f5d62-117">Usuários externos conectados à Internet podem acessar o sistema por meio desse endereço.</span><span class="sxs-lookup"><span data-stu-id="f5d62-117">External users connected to the Internet can access the system through this address.</span></span>
- <span data-ttu-id="f5d62-118">**NVA (Solução de Virtualização de Rede)**.</span><span class="sxs-lookup"><span data-stu-id="f5d62-118">**Network virtual appliance (NVA)**.</span></span> <span data-ttu-id="f5d62-119">Essa arquitetura inclui um pool separado de NVAs para tráfego originado na Internet.</span><span class="sxs-lookup"><span data-stu-id="f5d62-119">This architecture includes a separate pool of NVAs for traffic originating on the Internet.</span></span>
- <span data-ttu-id="f5d62-120">**Azure Load Balancer**.</span><span class="sxs-lookup"><span data-stu-id="f5d62-120">**Azure load balancer**.</span></span> <span data-ttu-id="f5d62-121">Todas as solicitações recebidas da Internet passam por pelo balanceador de carga e são distribuídas para NVAs na DMZ pública.</span><span class="sxs-lookup"><span data-stu-id="f5d62-121">All incoming requests from the Internet pass through the load balancer and are distributed to the NVAs in the public DMZ.</span></span>
- <span data-ttu-id="f5d62-122">**Sub-rede de entrada da DMZ pública**.</span><span class="sxs-lookup"><span data-stu-id="f5d62-122">**Public DMZ inbound subnet**.</span></span> <span data-ttu-id="f5d62-123">Essa sub-rede aceita solicitações do Azure Load Balancer.</span><span class="sxs-lookup"><span data-stu-id="f5d62-123">This subnet accepts requests from the Azure load balancer.</span></span> <span data-ttu-id="f5d62-124">Solicitações de entrada são passadas para uma das NVAs na DMZ pública.</span><span class="sxs-lookup"><span data-stu-id="f5d62-124">Incoming requests are passed to one of the NVAs in the public DMZ.</span></span>
- <span data-ttu-id="f5d62-125">**Sub-rede de saída da DMZ pública**.</span><span class="sxs-lookup"><span data-stu-id="f5d62-125">**Public DMZ outbound subnet**.</span></span> <span data-ttu-id="f5d62-126">As solicitações que foram aprovadas pela NVA passam por essa sub-rede por meio do balanceador de carga interno e vão para a camada da Web.</span><span class="sxs-lookup"><span data-stu-id="f5d62-126">Requests that are approved by the NVA pass through this subnet to the internal load balancer for the web tier.</span></span>

## <a name="recommendations"></a><span data-ttu-id="f5d62-127">Recomendações</span><span class="sxs-lookup"><span data-stu-id="f5d62-127">Recommendations</span></span>

<span data-ttu-id="f5d62-128">As seguintes recomendações aplicam-se à maioria dos cenários.</span><span class="sxs-lookup"><span data-stu-id="f5d62-128">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="f5d62-129">Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.</span><span class="sxs-lookup"><span data-stu-id="f5d62-129">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="nva-recommendations"></a><span data-ttu-id="f5d62-130">Recomendações de NVA</span><span class="sxs-lookup"><span data-stu-id="f5d62-130">NVA recommendations</span></span>

<span data-ttu-id="f5d62-131">Use um conjunto de NVAs para tráfego originado na Internet e outro para o tráfego originado localmente.</span><span class="sxs-lookup"><span data-stu-id="f5d62-131">Use one set of NVAs for traffic originating on the Internet, and another for traffic originating on-premises.</span></span> <span data-ttu-id="f5d62-132">Usar apenas um conjunto de NVAs para ambos é um risco à segurança, uma vez que ele não oferece nenhum perímetro de segurança entre os dois conjuntos de tráfego de rede.</span><span class="sxs-lookup"><span data-stu-id="f5d62-132">Using only one set of NVAs for both is a security risk, because it provides no security perimeter between the two sets of network traffic.</span></span> <span data-ttu-id="f5d62-133">Usar NVAs separadas reduz a complexidade da verificação das regras de segurança e deixa claro quais regras correspondem a quais solicitações de rede de entrada.</span><span class="sxs-lookup"><span data-stu-id="f5d62-133">Using separate NVAs reduces the complexity of checking security rules, and makes it clear which rules correspond to each incoming network request.</span></span> <span data-ttu-id="f5d62-134">Um conjunto de NVAs implementa as regras apenas para o tráfego de Internet, enquanto outro conjunto de NVAs implementa as regras apenas para tráfego local.</span><span class="sxs-lookup"><span data-stu-id="f5d62-134">One set of NVAs implements rules for Internet traffic only, while another set of NVAs implement rules for on-premises traffic only.</span></span>

<span data-ttu-id="f5d62-135">Inclua uma NVA da camada 7 para terminar as conexões de aplicativo no nível de NVA e manter a compatibilidade com as camadas de back-end.</span><span class="sxs-lookup"><span data-stu-id="f5d62-135">Include a layer-7 NVA to terminate application connections at the NVA level and maintain compatibility with the backend tiers.</span></span> <span data-ttu-id="f5d62-136">Isso garante a conectividade simétrica na qual o tráfego de resposta de camadas de back-end retorna por meio da NVA.</span><span class="sxs-lookup"><span data-stu-id="f5d62-136">This guarantees symmetric connectivity where response traffic from the backend tiers returns through the NVA.</span></span>

### <a name="public-load-balancer-recommendations"></a><span data-ttu-id="f5d62-137">Recomendações de balanceador de carga público</span><span class="sxs-lookup"><span data-stu-id="f5d62-137">Public load balancer recommendations</span></span>

<span data-ttu-id="f5d62-138">Para obter escalabilidade e disponibilidade, implante as NVAs da DMZ pública em um [conjunto de disponibilidade][availability-set] e use um [balanceador de carga para a Internet][load-balancer] para distribuir solicitações da Internet para as NVAs no conjunto de disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="f5d62-138">For scalability and availability, deploy the public DMZ NVAs in an [availability set][availability-set] and use an [Internet facing load balancer][load-balancer] to distribute Internet requests across the NVAs in the availability set.</span></span>

<span data-ttu-id="f5d62-139">Configure o balanceador de carga para aceitar somente as solicitações nas portas necessárias para o tráfego de Internet.</span><span class="sxs-lookup"><span data-stu-id="f5d62-139">Configure the load balancer to accept requests only on the ports necessary for Internet traffic.</span></span> <span data-ttu-id="f5d62-140">Por exemplo, restrinja as solicitações HTTP de entrada à porta 80 e as solicitações HTTPS de entrada à porta 443.</span><span class="sxs-lookup"><span data-stu-id="f5d62-140">For example, restrict inbound HTTP requests to port 80 and inbound HTTPS requests to port 443.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="f5d62-141">Considerações sobre escalabilidade</span><span class="sxs-lookup"><span data-stu-id="f5d62-141">Scalability considerations</span></span>

<span data-ttu-id="f5d62-142">Mesmo se a arquitetura inicialmente exigir uma única NVA na DMZ pública, é recomendável colocar um balanceador de carga na frente da DMZ pública desde o início.</span><span class="sxs-lookup"><span data-stu-id="f5d62-142">Even if your architecture initially requires a single NVA in the public DMZ, we recommend putting a load balancer in front of the public DMZ from the beginning.</span></span> <span data-ttu-id="f5d62-143">Isso facilitará o dimensionamento para várias NVAs no futuro, se necessário.</span><span class="sxs-lookup"><span data-stu-id="f5d62-143">That will make it easier to scale to multiple NVAs in the future, if needed.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="f5d62-144">Considerações sobre disponibilidade</span><span class="sxs-lookup"><span data-stu-id="f5d62-144">Availability considerations</span></span>

<span data-ttu-id="f5d62-145">O balanceador de carga para a Internet requer que cada NVA na sub-rede de entrada da DMZ pública implemente uma [investigação de integridade][lb-probe].</span><span class="sxs-lookup"><span data-stu-id="f5d62-145">The Internet facing load balancer requires each NVA in the public DMZ inbound subnet to implement a [health probe][lb-probe].</span></span> <span data-ttu-id="f5d62-146">Uma investigação de integridade que não responder a esse ponto de extremidade é considerado indisponível e o balanceador de carga direcionará as solicitações para outras NVAs no mesmo conjunto de disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="f5d62-146">A health probe that fails to respond on this endpoint is considered to be unavailable, and the load balancer will direct requests to other NVAs in the same availability set.</span></span> <span data-ttu-id="f5d62-147">Observe que, se nenhum NVA responder o aplicativo falhará, portanto, é importante que o monitoramento esteja configurado para alertar o DevOps quando o número de instâncias íntegras de NVA ficar abaixo do limite definido.</span><span class="sxs-lookup"><span data-stu-id="f5d62-147">Note that if all NVAs fail to respond, your application will fail, so it's important to have monitoring configured to alert DevOps when the number of healthy NVA instances falls below a defined threshold.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="f5d62-148">Considerações sobre capacidade de gerenciamento</span><span class="sxs-lookup"><span data-stu-id="f5d62-148">Manageability considerations</span></span>

<span data-ttu-id="f5d62-149">Todos os monitoramentos e gerenciamentos para NVAs na rede de perímetro pública devem ser realizados pelo jumpbox na sub-rede de gerenciamento.</span><span class="sxs-lookup"><span data-stu-id="f5d62-149">All monitoring and management for the NVAs in the public DMZ should be performed by the jumpbox in the management subnet.</span></span> <span data-ttu-id="f5d62-150">Conforme discutido em [Implementação de uma DMZ entre o Azure e seu datacenter local][implementing-a-secure-hybrid-network-architecture], defina uma única rota de rede da rede local por meio do gateway para o jumpbox, a fim de restringir o acesso.</span><span class="sxs-lookup"><span data-stu-id="f5d62-150">As discussed in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture], define a single network route from the on-premises network through the gateway to the jumpbox, in order to restrict access.</span></span>

<span data-ttu-id="f5d62-151">Se a conectividade de gateway da rede local para o Azure estiver inativa, você ainda poderá acessar o jumpbox implantando um endereço IP público, adicionando-o ao jumpbox e fazendo logon da Internet.</span><span class="sxs-lookup"><span data-stu-id="f5d62-151">If gateway connectivity from your on-premises network to Azure is down, you can still reach the jumpbox by deploying a public IP address, adding it to the jumpbox, and logging in from the Internet.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="f5d62-152">Considerações de segurança</span><span class="sxs-lookup"><span data-stu-id="f5d62-152">Security considerations</span></span>

<span data-ttu-id="f5d62-153">Essa arquitetura de referência implementa vários níveis de segurança:</span><span class="sxs-lookup"><span data-stu-id="f5d62-153">This reference architecture implements multiple levels of security:</span></span>

- <span data-ttu-id="f5d62-154">O balanceador de carga para a Internet direciona as solicitações para as NVAs na sub-rede DMZ pública de entrada e apenas nas portas necessárias para o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="f5d62-154">The Internet facing load balancer directs requests to the NVAs in the inbound public DMZ subnet, and only on the ports necessary for the application.</span></span>
- <span data-ttu-id="f5d62-155">As regras do NSG para as sub-redes da DMZ pública de entrada e saída impedem que as NVAs sejam comprometidas, bloqueando as solicitações que estão fora das regras do NSG.</span><span class="sxs-lookup"><span data-stu-id="f5d62-155">The NSG rules for the inbound and outbound public DMZ subnets prevent the NVAs from being compromised, by blocking requests that fall outside of the NSG rules.</span></span>
- <span data-ttu-id="f5d62-156">A configuração de roteamento de NAT para as NVAs direciona solicitações de entrada na porta 80 e 443 para o balanceador de carga da camada da Web, mas ignora as solicitações em todas as demais portas.</span><span class="sxs-lookup"><span data-stu-id="f5d62-156">The NAT routing configuration for the NVAs directs incoming requests on port 80 and port 443 to the web tier load balancer, but ignores requests on all other ports.</span></span>

<span data-ttu-id="f5d62-157">Você deve registrar todas as solicitações de entrada em todas as portas.</span><span class="sxs-lookup"><span data-stu-id="f5d62-157">You should log all incoming requests on all ports.</span></span> <span data-ttu-id="f5d62-158">Audite os logs regularmente, prestando atenção às solicitações que estão fora dos parâmetros esperados, pois isso pode indicar tentativas de invasão.</span><span class="sxs-lookup"><span data-stu-id="f5d62-158">Regularly audit the logs, paying attention to requests that fall outside of expected parameters, as these may indicate intrusion attempts.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="f5d62-159">Implantar a solução</span><span class="sxs-lookup"><span data-stu-id="f5d62-159">Deploy the solution</span></span>

<span data-ttu-id="f5d62-160">Uma implantação para uma arquitetura de referência que implementa essas recomendações está disponível no [GitHub][github-folder].</span><span class="sxs-lookup"><span data-stu-id="f5d62-160">A deployment for a reference architecture that implements these recommendations is available on [GitHub][github-folder].</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f5d62-161">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="f5d62-161">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a><span data-ttu-id="f5d62-162">Implantação de recursos</span><span class="sxs-lookup"><span data-stu-id="f5d62-162">Deploy resources</span></span>

1. <span data-ttu-id="f5d62-163">Navegue para a pasta `/dmz/secure-vnet-dmz` do repositório GitHub de arquiteturas de referência.</span><span class="sxs-lookup"><span data-stu-id="f5d62-163">Navigate to the `/dmz/secure-vnet-dmz` folder of the reference architectures GitHub repository.</span></span>

2. <span data-ttu-id="f5d62-164">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="f5d62-164">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. <span data-ttu-id="f5d62-165">Execute o comando a seguir:</span><span class="sxs-lookup"><span data-stu-id="f5d62-165">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a><span data-ttu-id="f5d62-166">Conectar os gateways do Azure e locais</span><span class="sxs-lookup"><span data-stu-id="f5d62-166">Connect the on-premises and Azure gateways</span></span>

<span data-ttu-id="f5d62-167">Nesta etapa, você conectará os dois gateways de rede locais.</span><span class="sxs-lookup"><span data-stu-id="f5d62-167">In this step, you will connect the two local network gateways.</span></span>

1. <span data-ttu-id="f5d62-168">No portal do Azure, navegue até o grupo de recursos que você criou.</span><span class="sxs-lookup"><span data-stu-id="f5d62-168">In the Azure Portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="f5d62-169">Localize o recurso denominado `ra-vpn-vgw-pip` e copie o endereço IP mostrado na folha **Visão geral**.</span><span class="sxs-lookup"><span data-stu-id="f5d62-169">Find the resource named `ra-vpn-vgw-pip` and copy the IP address shown in the **Overview** blade.</span></span>

3. <span data-ttu-id="f5d62-170">Encontre o recurso denominado `onprem-vpn-lgw`.</span><span class="sxs-lookup"><span data-stu-id="f5d62-170">Find the resource named `onprem-vpn-lgw`.</span></span>

4. <span data-ttu-id="f5d62-171">Clique na folha **Configuração**.</span><span class="sxs-lookup"><span data-stu-id="f5d62-171">Click the **Configuration** blade.</span></span> <span data-ttu-id="f5d62-172">No **Endereço IP**, cole o endereço IP da etapa 2.</span><span class="sxs-lookup"><span data-stu-id="f5d62-172">Under **IP address**, paste in the IP address from step 2.</span></span>

    ![Captura de tela do campo Endereço IP](./images/local-net-gw.png)

5. <span data-ttu-id="f5d62-174">Clique em **Salvar** e aguarde a conclusão da operação.</span><span class="sxs-lookup"><span data-stu-id="f5d62-174">Click **Save** and wait for the operation to complete.</span></span> <span data-ttu-id="f5d62-175">Isso pode levar cerca de 5 minutos.</span><span class="sxs-lookup"><span data-stu-id="f5d62-175">It can take about 5 minutes.</span></span>

6. <span data-ttu-id="f5d62-176">Encontre o recurso denominado `onprem-vpn-gateway1-pip`.</span><span class="sxs-lookup"><span data-stu-id="f5d62-176">Find the resource named `onprem-vpn-gateway1-pip`.</span></span> <span data-ttu-id="f5d62-177">Copie o endereço IP mostrado na folha **Visão geral**.</span><span class="sxs-lookup"><span data-stu-id="f5d62-177">Copy the IP address shown in the **Overview** blade.</span></span>

7. <span data-ttu-id="f5d62-178">Encontre o recurso denominado `ra-vpn-lgw`.</span><span class="sxs-lookup"><span data-stu-id="f5d62-178">Find the resource named `ra-vpn-lgw`.</span></span>

8. <span data-ttu-id="f5d62-179">Clique na folha **Configuração**.</span><span class="sxs-lookup"><span data-stu-id="f5d62-179">Click the **Configuration** blade.</span></span> <span data-ttu-id="f5d62-180">No **Endereço IP**, cole o endereço IP da etapa 6.</span><span class="sxs-lookup"><span data-stu-id="f5d62-180">Under **IP address**, paste in the IP address from step 6.</span></span>

9. <span data-ttu-id="f5d62-181">Clique em **Salvar** e aguarde a conclusão da operação.</span><span class="sxs-lookup"><span data-stu-id="f5d62-181">Click **Save** and wait for the operation to complete.</span></span>

10. <span data-ttu-id="f5d62-182">Para verificar a conexão, vá até a folha **Conexões** de cada gateway.</span><span class="sxs-lookup"><span data-stu-id="f5d62-182">To verify the connection, go to the **Connections** blade for each gateway.</span></span> <span data-ttu-id="f5d62-183">O status deve ser **Conectado**.</span><span class="sxs-lookup"><span data-stu-id="f5d62-183">The status should be **Connected**.</span></span>

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a><span data-ttu-id="f5d62-184">Verificar se o tráfego de rede atinge a camada da Web</span><span class="sxs-lookup"><span data-stu-id="f5d62-184">Verify that network traffic reaches the web tier</span></span>

1. <span data-ttu-id="f5d62-185">No portal do Azure, navegue até o grupo de recursos que você criou.</span><span class="sxs-lookup"><span data-stu-id="f5d62-185">In the Azure Portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="f5d62-186">Encontre o recurso denominado `pub-dmz-lb`, que é o balanceador de carga na frente da DMZ pública.</span><span class="sxs-lookup"><span data-stu-id="f5d62-186">Find the resource named `pub-dmz-lb`, which is the load balancer in front of the public DMZ.</span></span>

3. <span data-ttu-id="f5d62-187">Copie o endereço IP público da folha **Visão geral** e abra-o em um navegador da Web.</span><span class="sxs-lookup"><span data-stu-id="f5d62-187">Copy the public IP addess from the **Overview** blade and open this address in a web browser.</span></span> <span data-ttu-id="f5d62-188">Você deve ver a home page padrão do servidor Apache2.</span><span class="sxs-lookup"><span data-stu-id="f5d62-188">You should see the default Apache2 server home page.</span></span>

4. <span data-ttu-id="f5d62-189">Encontre o recurso denominado `int-dmz-lb`, que é o balanceador de carga na frente da DMZ privada.</span><span class="sxs-lookup"><span data-stu-id="f5d62-189">Find the resource named `int-dmz-lb`, which is the load balancer in front of the private DMZ.</span></span> <span data-ttu-id="f5d62-190">Copie o endereço IP privado da folha **Visão geral**.</span><span class="sxs-lookup"><span data-stu-id="f5d62-190">Copy the private IP address from the **Overview** blade.</span></span>

5. <span data-ttu-id="f5d62-191">Encontre a VM denominada `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="f5d62-191">Find the VM named `jb-vm1`.</span></span> <span data-ttu-id="f5d62-192">Clique em **Conectar** e use a Área de Trabalho Remota para se conectar à VM.</span><span class="sxs-lookup"><span data-stu-id="f5d62-192">Click **Connect** and use Remote Desktop to connect to the VM.</span></span> <span data-ttu-id="f5d62-193">O nome de usuário e a senha são especificadas no arquivo onprem.json.</span><span class="sxs-lookup"><span data-stu-id="f5d62-193">The user name and password are specified in the onprem.json file.</span></span>

6. <span data-ttu-id="f5d62-194">Na sessão de Área de Trabalho Remota, abra um navegador da Web e navegue até o endereço IP da etapa 4.</span><span class="sxs-lookup"><span data-stu-id="f5d62-194">From the Remote Desktop Session, open a web browser and navigate to the IP address from step 4.</span></span> <span data-ttu-id="f5d62-195">Você deve ver a home page padrão do servidor Apache2.</span><span class="sxs-lookup"><span data-stu-id="f5d62-195">You should see the default Apache2 server home page.</span></span>

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
