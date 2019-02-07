---
title: Implementar uma rede de perímetro entre o Azure e a Internet
titleSuffix: Azure Reference Architectures
description: Como implementar uma arquitetura de rede híbrida segura com acesso à Internet no Azure.
author: telmosampaio
ms.date: 10/22/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, networking
ms.openlocfilehash: 45ae8de1138b738fdfb42bdf57402711e1be6ebb
ms.sourcegitcommit: 14226018a058e199523106199be9c07f6a3f8592
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/31/2019
ms.locfileid: "55482902"
---
# <a name="implement-a-dmz-between-azure-and-the-internet"></a>Implementar uma rede de perímetro entre o Azure e a Internet

Essa arquitetura de referência mostra uma rede híbrido segura que estende a uma rede local para o Azure e também aceita tráfego de Internet. [**Implantar esta solução**](#deploy-the-solution).

> [!NOTE]
> Este cenário também pode ser obtido usando o [Firewall do Azure](/azure/firewall/), um serviço de segurança de rede baseado em nuvem.

![Proteger arquitetura de rede híbrida](./images/dmz-public.png)

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

Essa arquitetura de referência estende a arquitetura descrita em [Implementação de uma DMZ entre o Azure e seu datacenter local][implementing-a-secure-hybrid-network-architecture]. Ela adiciona uma rede de perímetro que manipula o tráfego da Internet, além da DMZ privada que manipula o tráfego da rede local.

Alguns usos típicos dessa arquitetura:

- Aplicativos híbridos nos quais as cargas de trabalho são executadas parcialmente localmente e parcialmente no Azure.
- A infraestrutura do Azure que roteia o tráfego de entrada local e da Internet.

## <a name="architecture"></a>Arquitetura

A arquitetura consiste nos componentes a seguir.

- **Endereço IP público (PIP)**. O endereço IP do ponto de extremidade público. Usuários externos conectados à Internet podem acessar o sistema por meio desse endereço.
- **NVA (Solução de Virtualização de Rede)**. Essa arquitetura inclui um pool separado de NVAs para tráfego originado na Internet.
- **Azure Load Balancer**. Todas as solicitações recebidas da Internet passam por pelo balanceador de carga e são distribuídas para NVAs na DMZ pública.
- **Sub-rede de entrada da DMZ pública**. Essa sub-rede aceita solicitações do Azure Load Balancer. Solicitações de entrada são passadas para uma das NVAs na DMZ pública.
- **Sub-rede de saída da DMZ pública**. As solicitações que foram aprovadas pela NVA passam por essa sub-rede por meio do balanceador de carga interno e vão para a camada da Web.

## <a name="recommendations"></a>Recomendações

As seguintes recomendações aplicam-se à maioria dos cenários. Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.

### <a name="nva-recommendations"></a>Recomendações de NVA

Use um conjunto de NVAs para tráfego originado na Internet e outro para o tráfego originado localmente. Usar apenas um conjunto de NVAs para ambos é um risco à segurança, uma vez que ele não oferece nenhum perímetro de segurança entre os dois conjuntos de tráfego de rede. Usar NVAs separadas reduz a complexidade da verificação das regras de segurança e deixa claro quais regras correspondem a quais solicitações de rede de entrada. Um conjunto de NVAs implementa as regras apenas para o tráfego de Internet, enquanto outro conjunto de NVAs implementa as regras apenas para tráfego local.

Inclua uma NVA da camada 7 para terminar as conexões de aplicativo no nível de NVA e manter a compatibilidade com as camadas de back-end. Isso garante a conectividade simétrica na qual o tráfego de resposta de camadas de back-end retorna por meio da NVA.

### <a name="public-load-balancer-recommendations"></a>Recomendações de balanceador de carga público

Para obter escalabilidade e disponibilidade, implante as NVAs da DMZ pública em um [conjunto de disponibilidade][availability-set] e use um [balanceador de carga para a Internet][load-balancer] para distribuir solicitações da Internet para as NVAs no conjunto de disponibilidade.

Configure o balanceador de carga para aceitar somente as solicitações nas portas necessárias para o tráfego de Internet. Por exemplo, restrinja as solicitações HTTP de entrada à porta 80 e as solicitações HTTPS de entrada à porta 443.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Mesmo se a arquitetura inicialmente exigir uma única NVA na DMZ pública, é recomendável colocar um balanceador de carga na frente da DMZ pública desde o início. Isso facilitará o dimensionamento para várias NVAs no futuro, se necessário.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

O balanceador de carga para a Internet requer que cada NVA na sub-rede de entrada da DMZ pública implemente uma [investigação de integridade][lb-probe]. Uma investigação de integridade que não responder a esse ponto de extremidade é considerado indisponível e o balanceador de carga direcionará as solicitações para outras NVAs no mesmo conjunto de disponibilidade. Observe que, se nenhum NVA responder o aplicativo falhará, portanto, é importante que o monitoramento esteja configurado para alertar o DevOps quando o número de instâncias íntegras de NVA ficar abaixo do limite definido.

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Todos os monitoramentos e gerenciamentos para NVAs na rede de perímetro pública devem ser realizados pelo jumpbox na sub-rede de gerenciamento. Conforme discutido em [Implementação de uma DMZ entre o Azure e seu datacenter local][implementing-a-secure-hybrid-network-architecture], defina uma única rota de rede da rede local por meio do gateway para o jumpbox, a fim de restringir o acesso.

Se a conectividade de gateway da rede local para o Azure estiver inativa, você ainda poderá acessar o jumpbox implantando um endereço IP público, adicionando-o ao jumpbox e fazendo logon da Internet.

## <a name="security-considerations"></a>Considerações de segurança

Essa arquitetura de referência implementa vários níveis de segurança:

- O balanceador de carga para a Internet direciona as solicitações para as NVAs na sub-rede DMZ pública de entrada e apenas nas portas necessárias para o aplicativo.
- As regras do NSG para as sub-redes da DMZ pública de entrada e saída impedem que as NVAs sejam comprometidas, bloqueando as solicitações que estão fora das regras do NSG.
- A configuração de roteamento de NAT para as NVAs direciona solicitações de entrada na porta 80 e 443 para o balanceador de carga da camada da Web, mas ignora as solicitações em todas as demais portas.

Você deve registrar todas as solicitações de entrada em todas as portas. Audite os logs regularmente, prestando atenção às solicitações que estão fora dos parâmetros esperados, pois isso pode indicar tentativas de invasão.

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para uma arquitetura de referência que implementa essas recomendações está disponível no [GitHub][github-folder].

### <a name="prerequisites"></a>Pré-requisitos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a>Implantação de recursos

1. Navegue para a pasta `/dmz/secure-vnet-dmz` do repositório GitHub de arquiteturas de referência.

2. Execute o comando a seguir:

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. Navegue para a pasta `/dmz/ssecure-vnet-hybrid` do repositório GitHub de arquiteturas de referência.

4. Execute o comando a seguir:

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a>Conectar os gateways do Azure e locais

Nesta etapa, você conectará os dois gateways de rede locais.

1. No portal do Azure, navegue até o grupo de recursos que você criou.

2. Localize o recurso denominado `ra-vpn-vgw-pip` e copie o endereço IP mostrado na folha **Visão geral**.

3. Encontre o recurso denominado `onprem-vpn-lgw`.

4. Clique na folha **Configuração**. No **Endereço IP**, cole o endereço IP da etapa 2.

    ![Captura de tela do campo Endereço IP](./images/local-net-gw.png)

5. Clique em **Salvar** e aguarde a conclusão da operação. Isso pode levar cerca de 5 minutos.

6. Encontre o recurso denominado `onprem-vpn-gateway1-pip`. Copie o endereço IP mostrado na folha **Visão geral**.

7. Encontre o recurso denominado `ra-vpn-lgw`.

8. Clique na folha **Configuração**. No **Endereço IP**, cole o endereço IP da etapa 6.

9. Clique em **Salvar** e aguarde a conclusão da operação.

10. Para verificar a conexão, vá até a folha **Conexões** de cada gateway. O status deve ser **Conectado**.

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a>Verificar se o tráfego de rede atinge a camada da Web

1. No portal do Azure, navegue até o grupo de recursos que você criou.

2. Encontre o recurso denominado `pub-dmz-lb`, que é o balanceador de carga na frente da DMZ pública.

3. Copie o endereço IP público da folha **Visão geral** e abra-o em um navegador da Web. Você deve ver a home page padrão do servidor Apache2.

4. Encontre o recurso denominado `int-dmz-lb`, que é o balanceador de carga na frente da DMZ privada. Copie o endereço IP privado da folha **Visão geral**.

5. Encontre a VM denominada `jb-vm1`. Clique em **Conectar** e use a Área de Trabalho Remota para se conectar à VM. O nome de usuário e a senha são especificadas no arquivo onprem.json.

6. Na sessão de Área de Trabalho Remota, abra um navegador da Web e navegue até o endereço IP da etapa 4. Você deve ver a home page padrão do servidor Apache2.

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
