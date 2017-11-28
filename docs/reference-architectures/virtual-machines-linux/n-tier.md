---
title: Executar VMs do Linux para um aplicativo de N camadas no Azure
description: Como executar VMs do Linux para uma arquitetura de N camadas no Microsoft Azure.
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Linux VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 673e090e306ffc421603371658c8273b10b980d4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="run-linux-vms-for-an-n-tier-application"></a>Executar VMs do Linux para um aplicativo de N camadas

Essa arquitetura de referência mostra um conjunto de práticas comprovadas para executar VMs (máquinas virtuais) do Linux para um aplicativo de N camadas. [**Implantar esta solução**.](#deploy-the-solution)  

![[0]][0]

*Baixe um [arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura

Há muitas maneiras de implementar uma arquitetura de N camadas. O diagrama mostra um aplicativo Web de três camadas típico. Essa arquitetura se baseia em [Executar VMs com balanceamento de carga para escalabilidade e disponibilidade][multi-vm]. As camadas comerciais e da Web usam VMs com balanceamento de carga.

* **Conjuntos de disponibilidade.** Crie um [conjunto de disponibilidade][azure-availability-sets] para cada camada e provisione pelo menos duas VMs em cada camada.  Isso torna as VMs qualificadas para um [SLA (Contrato de Nível de Serviço)][vm-sla] mais elevado.
* **Sub-redes.** Sempre crie uma sub-rede separada para cada camada. Especifique o intervalo de endereços e a máscara de sub-rede usando a notação [CIDR]. 
* **Balanceadores de carga.** Use um [Balanceador de carga para a Internet][load-balancer-external] para distribuir o tráfego de entrada da Internet para a camada da Web e um [balanceador de carga interno][load-balancer-internal] para distribuir o tráfego de rede da camada da Web para a camada comercial.
* **Jumpbox.** Também chamada de um [host bastião]. Uma VM protegida na rede que os administradores usam para se conectar às outras VMs. O jumpbox tem um NSG que permite o tráfego remoto apenas de endereços IP públicos em uma lista segura. O NSG deve permitir o tráfego SSH (secure shell).
* **Monitoramento.** Softwares de monitoramento, como [Nagios], [Zabbix] ou [Icinga] podem fornecer informações sobre o tempo de resposta, tempo de atividade da VM e a integridade geral do sistema. Instale o software de monitoramento em uma VM que se encontra em uma sub-rede separada de gerenciamento.
* **NSGs.** Use os NSGs [grupos de segurança de rede][nsg] para restringir o tráfego de rede na VNet. Por exemplo, na arquitetura de três camadas mostrada aqui, a camada de banco de dados não aceita o tráfego de front-end da Web, somente da camada comercial e da sub-rede de gerenciamento.
* **Banco de dados Apache Cassandra**. Fornece alta disponibilidade na camada de dados, habilitando replicação e failover.

## <a name="recommendations"></a>Recomendações

Seus requisitos podem ser diferentes dos requisitos da arquitetura descrita aqui. Use essas recomendações como ponto de partida. 

### <a name="vnet--subnets"></a>VNet / Sub-redes

Quando você cria a VNet, determine quantos endereços IP seus recursos em cada sub-rede exigem. Especifique uma máscara de sub-rede e um intervalo de endereços de VNet grande o suficiente para os endereços IP necessários, usando a notação [CIDR]. Use um espaço de endereço que esteja dentro dos [blocos de endereço IP privados][private-ip-space] padrão, que são 10.0.0.0/8, 172.16.0.0/12 e 192.168.0.0/16.

Escolha um intervalo de endereços que não se sobreponha ao da rede local para caso seja necessário configurar um gateway entre a VNet e a rede local mais tarde. Depois de criar a VNet, não será possível alterar o intervalo de endereços.

Crie as sub-redes levando em conta os requisitos de funcionalidade e de segurança. Todas as VMs na mesma camada ou função devem ir para a mesma sub-rede, que pode ser um limite de segurança. Para obter mais informações sobre como criar VNets e sub-redes, consulte [Planejar e criar redes virtuais do Azure][plan-network].

Especifique o espaço de endereço para cada sub-rede em notação CIDR. Por exemplo, “10.0.0.0/24” cria um intervalo de 256 endereços IP. VMs podem usar 251 deles; cinco são reservados. Verifique se que os intervalos de endereços não se sobrepõem a outras sub-redes. Consulte as [Perguntas Frequentes sobre rede virtual][vnet faq].

### <a name="network-security-groups"></a>Grupos de segurança de rede

Use as regras de NSG para restringir o tráfego entre as camadas. Por exemplo, a arquitetura de três camadas mostrada acima, a camada da Web não se comunica diretamente com a camada de banco de dados. Para impor isso, a camada de banco de dados deve bloquear o tráfego de entrada da sub-rede da camada da Web.  

1. Crie um NSG e associe-o à sub-rede da camada de banco de dados.
2. Adicione uma regra que nega todo o tráfego de entrada da VNet. (Use a marca `VIRTUAL_NETWORK` na regra.) 
3. Adicione uma regra com uma prioridade mais alta que permite tráfego de entrada da sub-rede da camada comercial. Essa regra substitui a regra anterior e permite que a camada comercial se comunique com a camada de banco de dados.
4. Adicione uma regra para permitir o tráfego de entrada de dentro da própria sub-rede da camada de banco de dados. Essa regra permite a comunicação entre as VMs na camada de banco de dados, que é necessária para failover e replicação de banco de dados.
5. Adicione uma regra para permitir o tráfego SSH da sub-rede de jumpbox. Essa regra permite que os administradores se conectem à camada de banco de dados do jumpbox.
   
   > [!NOTE]
   > Um NSG tem [regras padrão][nsg-rules] que permitem qualquer tráfego de entrada de dentro da VNet. Essas regras não podem ser excluídas, mas você pode substituí-las criando regras de prioridade mais alta.
   > 
   > 

### <a name="load-balancers"></a>Balanceadores de carga

Um balanceador externo de carga que distribui o tráfego da Internet para a camada da Web. Crie um endereço IP público para esse balanceador de carga. Consulte [Criação de um balanceador de carga para a Internet][lb-external-create].

O balanceador de carga interno distribui o tráfego de rede da camada da Web para a camada comercial. Para conceder a este balanceador de carga de um endereço IP privado, crie uma configuração de IP de front-end e associe-o à sub-rede para a camada comercial. Consulte [Introdução à criação de um balanceador de carga interno][lb-internal-create].

### <a name="cassandra"></a>Cassandra

Recomendamos o [DataStax Enterprise][datastax] para uso de produção, mas essas recomendações se aplicam a qualquer edição do Cassandra. Para obter mais informações sobre como executar o DataStax no Azure, consulte [Guia de implantação do DataStax Enterprise no Azure][cassandra-in-azure]. 

Coloque as VMs para um cluster Cassandra em um conjunto de disponibilidade para garantir que as réplicas do Cassandra sejam distribuídas entre vários domínios de falha e de atualização. Para saber mais sobre domínios de falha e de atualização, consulte [Gerenciar a disponibilidade das máquinas virtuais][azure-availability-sets]. 

Configure três domínios de falha (o máximo) por conjunto de disponibilidade e 18 domínios de atualização por conjunto de disponibilidade. Isso fornece o número máximo de domínios de atualização que ainda podem ser distribuído uniformemente entre os domínios de falha.   

Configure os nós no modo de reconhecimento de rack. Mapear domínios de falha para racks no arquivo `cassandra-rackdc.properties`.

Não é necessário um balanceador de carga na frente do cluster. O cliente se conecta diretamente a um nó no cluster.

### <a name="jumpbox"></a>Jumpbox

O jumpbox terá requisitos mínimos de desempenho, portanto, selecione um tamanho de VM pequeno para o jumpbox como A1 Standard. 

Crie um [endereço IP público] para o jumpbox. Coloque o jumpbox na mesma VNet que as outras VMs, mas em uma sub-rede de gerenciamento separada.

Não permita o acesso SSH da Internet pública para as VMs que executam a carga de trabalho do aplicativo. Em vez disso, todo o acesso SSH a essas VMs deve ocorrer por meio do jumpbox. Um administrador faz logon no jumpbox e, em seguida, faz logon na VM por meio do jumpbox. O jumpbox permite que tráfego SSH da Internet, mas apenas de endereços IP conhecidos e seguros.

Para proteger o jumpbox, crie um NSG e aplique-o à sub-rede do jumpbox. Adicione uma regra NSG para permitir conexões SSH somente de um conjunto seguro de endereços IP públicos. O NSG pode ser conectado à sub-rede ou à NIC de jumpbox. Nesse caso, recomendamos anexá-lo à NIC para que o tráfego SSH seja permitido somente para o jumpbox, mesmo se você adicionar outras VMs à mesma sub-rede.

Configure os NSGs para outras sub-redes para permitir o tráfego SSH da sub-rede de gerenciamento.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Coloque cada camada ou a função VM em um conjunto de disponibilidade separado. 

Na camada de banco de dados, ter várias VMs não automaticamente significa ter um banco de dados altamente disponível. Para um banco de dados relacional, você normalmente precisa usar a replicação e failover para obter alta disponibilidade.  

Se você precisar de disponibilidade mais alta do que a fornecida pelo [SLA do Azure para VMs][vm-sla], replique o aplicativo em duas regiões e use o Gerenciador de Tráfego do Azure para failover. Para obter mais informações, consulte [Executar VMs do Linux em várias regiões para ter alta disponibilidade][multi-dc].  

## <a name="security-considerations"></a>Considerações de segurança

Considere adicionar uma NVA (solução de virtualização de rede) para criar uma DMZ entre a Internet pública e a rede virtual do Azure. NVA é um termo genérico para uma solução de virtualização que pode executar tarefas relacionadas à rede, como firewall, inspeção de pacotes, auditoria e roteamento personalizado. Para obter mais informações, consulte [Implementação de uma DMZ entre o Azure e a Internet][dmz].

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Os balanceadores de carga distribuem o tráfego de rede para as camadas comercial e da Web. Dimensione horizontalmente adicionando novas instâncias de VM. Observe que é possível dimensionar as camadas comercial e da Web independentemente, dependendo da carga. Para reduzir possíveis complicações causadas pela necessidade de manter a afinidade do cliente, as VMs na camada da Web devem ser sem estado. As VMs que hospedam a lógica de negócios também devem ser sem estado.

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Simplifique o gerenciamento de todo o sistema usando ferramentas de administração centralizada, como [Automação do Azure][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef] ou [Puppet][puppet]. Essas ferramentas podem consolidar informações de diagnóstico e de integridade capturadas de várias VMs para fornecer uma visão geral do sistema.

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para essa arquitetura está disponível no [GitHub][github-folder]. A arquitetura é implantada em três estágios. Para implantar a arquitetura, siga estas etapas: 

1. Clique no botão abaixo:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fvirtual-machines%2Fn-tier-linux%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Depois que o link for aberto no portal do Azure, insira os valores a seguir: 
   * O nome do **Grupo de recursos** já está definido no arquivo de parâmetros, portanto, selecione **Criar novo** e digite `ra-ntier-cassandra-rg` na caixa de texto.
   * Selecione a região na caixa suspensa **Local**.
   * Não edite as caixas de texto **URI da raiz do modelo** ou **URI da raiz do parâmetro**.
   * Analise os termos e condições e clique na caixa de seleção **Concordo com os termos e condições declarados acima**.
   * Clique no botão **Comprar**.
3. Verifique a notificação no Portal do Azure para ver uma mensagem indicando que a implantação foi concluída.
4. Os arquivos de parâmetro incluem nomes de usuário e senha de administrador embutidos em código e é altamente recomendável que você altere imediatamente ambos em todas as VMs. Clique em cada VM no Portal do Azure e, em seguida, clique em **Redefinir senha** na folha **Suporte + solução de problemas**. Selecione **Redefinir senha** na caixa suspensa **Modo**, selecione um novo **Nome de usuário** e **Senha**. Clique no botão **Atualizar** para manter o novo nome de usuário e senha.

<!-- links -->
[multi-dc]: multi-region-application.md
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions

[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[host bastião]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://docs.datastax.com/en/datastax_enterprise/4.5/datastax_enterprise/install/installAzure.html
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: http://www.datastax.com/products/datastax-enterprise
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[endereço IP público]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[0]: ./images/n-tier-diagram.png "Arquitetura de N camadas usando o Microsoft Azure"

