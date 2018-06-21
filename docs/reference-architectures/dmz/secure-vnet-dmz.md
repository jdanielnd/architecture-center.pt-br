---
title: Implementação de uma DMZ entre o Azure e a Internet
description: Como implementar uma arquitetura de rede híbrida segura com acesso à Internet no Azure.
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: c88545b1fcae49b413e7e2b6ac5bd92d3fd3456d
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/30/2018
ms.locfileid: "30270385"
---
# <a name="dmz-between-azure-and-the-internet"></a>DMZ entre o Azure e a Internet

Essa arquitetura de referência mostra uma rede híbrido segura que estende a uma rede local para o Azure e também aceita tráfego de Internet. 

[![0]][0] 

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

Essa arquitetura de referência estende a arquitetura descrita em [Implementação de uma DMZ entre o Azure e seu datacenter local][implementing-a-secure-hybrid-network-architecture]. Ela adiciona uma DMZ que manipula o tráfego da Internet, além da DMZ privada que manipula o tráfego da rede local 

Alguns usos típicos para essa arquitetura:

* Aplicativos híbridos nos quais as cargas de trabalho são executadas parcialmente localmente e parcialmente no Azure.
* A infraestrutura do Azure que roteia o tráfego de entrada local e da Internet.

## <a name="architecture"></a>Arquitetura

A arquitetura consiste nos componentes a seguir.

* **Endereço IP público (PIP)**. O endereço IP do ponto de extremidade público. Usuários externos conectados à Internet podem acessar o sistema por meio desse endereço.
* **NVA (Solução de Virtualização de Rede)**. Essa arquitetura inclui um pool separado de NVAs para tráfego originado na Internet.
* **Azure Load Balancer**. Todas as solicitações recebidas da Internet passam por pelo balanceador de carga e são distribuídas para NVAs na DMZ pública.
* **Sub-rede de entrada da DMZ pública**. Essa sub-rede aceita solicitações do Azure Load Balancer. Solicitações de entrada são passadas para uma das NVAs na DMZ pública.
* **Sub-rede de saída da DMZ pública**. As solicitações que foram aprovadas pela NVA passam por essa sub-rede por meio do balanceador de carga interno e vão para a camada da Web.

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

* O balanceador de carga para a Internet direciona as solicitações para as NVAs na sub-rede DMZ pública de entrada e apenas nas portas necessárias para o aplicativo.
* As regras do NSG para as sub-redes da DMZ pública de entrada e saída impedem que as NVAs sejam comprometidas, bloqueando as solicitações que estão fora das regras do NSG.
* A configuração de roteamento de NAT para as NVAs direciona solicitações de entrada na porta 80 e 443 para o balanceador de carga da camada da Web, mas ignora as solicitações em todas as demais portas.

Você deve registrar todas as solicitações de entrada em todas as portas. Audite os logs regularmente, prestando atenção às solicitações que estão fora dos parâmetros esperados, pois isso pode indicar tentativas de invasão.

## <a name="solution-deployment"></a>Implantação da solução

Uma implantação para uma arquitetura de referência que implementa essas recomendações está disponível no [GitHub][github-folder]. A arquitetura de referência pode ser implantada com VMs do Windows ou Linux seguindo as instruções abaixo:

1. Clique no botão abaixo:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Depois que o link for aberto no portal do Azure, você deve inserir valores para algumas das configurações:
   * O nome do **Grupo de recursos** já está definido no arquivo de parâmetros, portanto, selecione **Criar novo** e digite `ra-public-dmz-network-rg` na caixa de texto.
   * Selecione a região na caixa suspensa **Local**.
   * Não edite as caixas de texto **URI da raiz do modelo** ou **URI da raiz do parâmetro**.
   * Selecione o **Tipo de SO** na caixa suspensa **windows** ou **linux**.
   * Analise os termos e condições e clique na caixa de seleção **Concordo com os termos e condições declarados acima**.
   * Clique no botão **Comprar**.
3. Aguarde até que a implantação seja concluída.
4. Clique no botão abaixo:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Depois que o link for aberto no portal do Azure, você deve inserir valores para algumas das configurações:
   * O nome do **Grupo de recursos** já está definido no arquivo de parâmetros, portanto, selecione **Criar novo** e digite `ra-public-dmz-wl-rg` na caixa de texto.
   * Selecione a região na caixa suspensa **Local**.
   * Não edite as caixas de texto **URI da raiz do modelo** ou **URI da raiz do parâmetro**.
   * Analise os termos e condições e clique na caixa de seleção **Concordo com os termos e condições declarados acima**.
   * Clique no botão **Comprar**.
6. Aguarde até que a implantação seja concluída.
7. Clique no botão abaixo:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. Depois que o link for aberto no portal do Azure, você deve inserir valores para algumas das configurações:
   * O nome do **Grupo de recursos** já está definido no arquivo de parâmetros, portanto, selecione **Usar Existente** e digite `ra-public-dmz-network-rg` na caixa de texto.
   * Selecione a região na caixa suspensa **Local**.
   * Não edite as caixas de texto **URI da raiz do modelo** ou **URI da raiz do parâmetro**.
   * Analise os termos e condições e clique na caixa de seleção **Concordo com os termos e condições declarados acima**.
   * Clique no botão **Comprar**.
9. Aguarde até que a implantação seja concluída.
10. Os arquivos de parâmetro incluem um nome e uma senha de usuário administrador embutidos em código para todas as VMs e é altamente recomendável alterá-los imediatamente. Clique em cada VM na implantação, selecione-a no Portal do Azure e, em seguida, clique em **Redefinir senha** na folha **Suporte + solução de problemas**. Selecione **Redefinir senha** na caixa suspensa **Modo** e escolha um novo **Nome de usuário** e **Senha**. Clique no botão **Atualizar** para salvar.


[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "Arquitetura de rede híbrida protegida"
