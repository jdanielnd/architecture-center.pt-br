---
title: Executar VMs do Windows para uma arquitetura de N camadas
description: "Como implementar uma arquitetura multicamadas no Azure, atentando-se em especial para a disponibilidade, segurança, escalabilidade e capacidade de gerenciamento da segurança."
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 0654239a5bbd966a2aa776415b7f15ae723ffd63
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/08/2018
---
# <a name="run-windows-vms-for-an-n-tier-application"></a>Executar VMs do Windows para um aplicativo de N camadas

Essa arquitetura de referência mostra um conjunto de práticas comprovadas para executar VMs (máquinas virtuais) do Windows para um aplicativo de N camadas. [**Implante essa solução**.](#deploy-the-solution) 

![[0]][0]

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura 

Há muitas maneiras de implementar uma arquitetura de N camadas. O diagrama mostra um aplicativo Web de três camadas típico. Essa arquitetura se baseia em [Executar VMs com balanceamento de carga para escalabilidade e disponibilidade][multi-vm]. As camadas comerciais e da Web usam VMs com balanceamento de carga.

* **Conjuntos de disponibilidade.** Crie um [conjunto de disponibilidade][azure-availability-sets] para cada camada e provisione pelo menos duas VMs em cada camada. Isso torna as VMs qualificadas para um [SLA (Contrato de Nível de Serviço)][vm-sla] mais elevado. É possível implantar uma VM única em um conjunto de disponibilidade, mas a VM única não se qualificará para uma garantia de SLA, a menos que a VM única esteja utilizando o Armazenamento Premium do Azure para todos discos de dados e SO.  
* **Sub-redes.** Sempre crie uma sub-rede separada para cada camada. Especifique o intervalo de endereços e a máscara de sub-rede usando a notação [CIDR]. 
* **Balanceadores de carga.** Use um [Balanceador de carga para a Internet][load-balancer-external] para distribuir o tráfego de entrada da Internet para a camada da Web e um [balanceador de carga interno][load-balancer-internal] para distribuir o tráfego de rede da camada da Web para a camada comercial.
* **Jumpbox.** Também chamada de um [host bastião]. Uma VM protegida na rede que os administradores usam para se conectar às outras VMs. O jumpbox tem um NSG que permite o tráfego remoto apenas de endereços IP públicos em uma lista segura. O NSG deve permitir o tráfego de RDP (área de trabalho remota).
* **Monitoramento.** Softwares de monitoramento, como [Nagios], [Zabbix] ou [Icinga] podem fornecer informações sobre o tempo de resposta, tempo de atividade da VM e a integridade geral do sistema. Instale o software de monitoramento em uma VM que se encontra em uma sub-rede separada de gerenciamento.
* **NSGs.** Use os NSGs [grupos de segurança de rede][nsg] para restringir o tráfego de rede na VNet. Por exemplo, na arquitetura de três camadas mostrada aqui, a camada de banco de dados não aceita o tráfego de front-end da Web, somente da camada comercial e da sub-rede de gerenciamento.
* **Grupo de Disponibilidade Always On do SQL Server.** Fornece alta disponibilidade na camada de dados, habilitando replicação e failover.
* **Servidores AD DS (Active Directory Domain Services)**. Antes do Windows Server 2016, os Grupos de Disponibilidade Always On do SQL Server precisavam ser ingressados em um domínio. Isso ocorria porque os Grupos de Disponibilidade dependem da tecnologia do WSFC (Cluster de Failover do Windows Server). O Windows Server 2016 introduziu a capacidade de criar um Cluster de Failover sem o Active Directory e nesse caso os servidores do AD DS não são necessários para essa arquitetura. Para obter mais informações, consulte [Novidades no Clustering de Failover do Windows Server 2016][wsfc-whats-new].
* **DNS do Azure**. [DNS do Azure][azure-dns] é um serviço de hospedagem para domínios DNS, que fornece resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure.

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
5. Adicione uma regra para permitir o tráfego RDP da sub-rede de jumpbox. Essa regra permite que os administradores se conectem à camada de banco de dados do jumpbox.
   
   > [!NOTE]
   > Um NSG tem regras padrão que permitem qualquer tráfego de entrada de dentro da VNet. Essas regras não podem ser excluídas, mas você pode substituí-las criando regras de prioridade mais alta.
   > 
   > 

### <a name="load-balancers"></a>Balanceadores de carga

Um balanceador externo de carga que distribui o tráfego da Internet para a camada da Web. Crie um endereço IP público para esse balanceador de carga. Consulte [Criação de um balanceador de carga para a Internet][lb-external-create].

O balanceador de carga interno distribui o tráfego de rede da camada da Web para a camada comercial. Para conceder a este balanceador de carga de um endereço IP privado, crie uma configuração de IP de front-end e associe-o à sub-rede para a camada comercial. Consulte [Introdução à criação de um balanceador de carga interno][lb-internal-create].

### <a name="sql-server-always-on-availability-groups"></a>Grupos de Disponibilidade Always On do SQL Server

Recomendamos usar os [Grupos de Disponibilidade Always On][sql-alwayson] para alta disponibilidade no SQL Server. Antes do Windows Server 2016, os Grupos de Disponibilidade Always On exigiam um controlador de domínio e todos os nós no grupo de disponibilidade precisavam estar no mesmo domínio do AD.

Outras camadas se conectam ao banco de dados por meio de um [ouvinte do grupo de disponibilidade][sql-alwayson-listeners]. O ouvinte permite que um cliente SQL se conecte sem saber o nome da instância física do SQL Server. VMs que acessam o banco de dados precisam ser ingressadas no domínio. O cliente (neste caso, outra camada) usa o DNS para resolver o nome da rede virtual do ouvinte em endereços IP.

Configure um grupo de disponibilidade Always On do SQL Server da seguinte maneira:

1. Crie um cluster WSFC (Clustering de Failover do Windows Server), um Grupo de Disponibilidade Always On do SQL Server e uma réplica primária. Para obter mais informações, consulte [Introdução aos Grupos de disponibilidade Always On][sql-alwayson-getting-started]. 
2. Crie um balanceador de carga interno com um endereço IP privado estático.
3. Crie um ouvinte do grupo de disponibilidade e mapeie o nome DNS do ouvinte para o endereço IP de um balanceador de carga interno. 
4. Crie uma regra do balanceador de carga para a porta de escuta do SQL Server (porta TCP 1433 por padrão). A regra do balanceador de carga deve habilitar *IP flutuante*, também chamado de Retorno de Servidor Direto. Isso faz com que a VM responda diretamente para o cliente, o que permite uma conexão direta com a réplica primária.
  
  > [!NOTE]
  > Quando o IP flutuante está habilitado, o número da porta de front-end deve ser igual ao número da porta de back-end na regra do balanceador de carga.
  > 
  > 

Quando um cliente SQL tenta se conectar, o balanceador de carga roteia a solicitação de conexão para a réplica primária. Se houver um failover para outra réplica, o balanceador de carga encaminhará automaticamente as solicitações subsequentes para uma nova réplica primária. Para obter mais informações, consulte [Configurar um ouvinte de ILB para Grupos de Disponibilidade Always On do SQL Server][sql-alwayson-ilb].

Durante um failover, as conexões de cliente existentes são fechadas. Após a conclusão do failover, novas conexões serão roteadas para a nova réplica primária.

Se seu aplicativo realiza muito mais leituras do que gravações, você pode descarregar algumas das consultas somente leitura para uma réplica secundária. Consulte [Usar um ouvinte para se conectar a uma réplica secundária somente leitura (roteamento somente leitura)][sql-alwayson-read-only-routing].

Teste a implantação por [forçando um failover manual][sql-alwayson-force-failover] do grupo de disponibilidade.

### <a name="jumpbox"></a>Jumpbox

O jumpbox terá requisitos mínimos de desempenho, portanto, selecione um tamanho de VM pequeno para o jumpbox como A1 Standard. 

Crie um [endereço IP público] para o jumpbox. Coloque o jumpbox na mesma VNet que as outras VMs, mas em uma sub-rede de gerenciamento separada.

Não permita o acesso RDP da Internet pública para as VMs que executam a carga de trabalho do aplicativo. Em vez disso, todo o acesso RDP a essas VMs deve ocorrer por meio do jumpbox. Um administrador faz logon no jumpbox e, em seguida, faz logon na VM por meio do jumpbox. O jumpbox permite que tráfego RDP da Internet, mas apenas de endereços IP conhecidos e seguros.

Para proteger o jumpbox, crie um NSG e aplique-o à sub-rede do jumpbox. Adicione uma regra NSG para permitir conexões RDP somente de um conjunto seguro de endereços IP públicos. O NSG pode ser conectado à sub-rede ou à NIC de jumpbox. Nesse caso, recomendamos anexá-lo à NIC para que o tráfego RDP seja permitido somente para o jumpbox, mesmo se você adicionar outras VMs à mesma sub-rede.

Configure os NSGs para outras sub-redes para permitir o tráfego RDP da sub-rede de gerenciamento.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Na camada de banco de dados, ter várias VMs não automaticamente significa ter um banco de dados altamente disponível. Para um banco de dados relacional, você normalmente precisa usar a replicação e failover para obter alta disponibilidade. Para o SQL Server, recomendamos usar os [Grupos de Disponibilidade Always On][sql-alwayson]. 

Se você precisar de disponibilidade mais alta do que a fornecida pelo [SLA do Azure para VMs][vm-sla], replique o aplicativo em duas regiões e use o Gerenciador de Tráfego do Azure para failover. Para obter mais informações, consulte [Executar VMs do Windows em várias regiões para ter alta disponibilidade][multi-dc].   

## <a name="security-considerations"></a>Considerações de segurança

Criptografe dados confidenciais em repouso e use o [Azure Key Vault][azure-key-vault] para gerenciar as chaves de criptografia de banco de dados. O Key Vault pode armazenar chaves de criptografia em HSMs (módulos de segurança de hardware). Para obter mais informações, consulte [Configurar a integração do Azure Key Vault para o SQL Server em VMs do Azure][sql-keyvault] Também é recomendado armazenar segredos do aplicativo, como cadeias de caracteres de conexão de banco de dados, no Key Vault.

Considere adicionar uma NVA (solução de virtualização de rede) para criar uma DMZ entre a Internet e a rede virtual do Azure. NVA é um termo genérico para uma solução de virtualização que pode executar tarefas relacionadas à rede, como firewall, inspeção de pacotes, auditoria e roteamento personalizado. Para obter mais informações, consulte [Implementação de uma DMZ entre o Azure e a Internet][dmz].

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Os balanceadores de carga distribuem o tráfego de rede para as camadas comercial e da Web. Dimensione horizontalmente adicionando novas instâncias de VM. Observe que é possível dimensionar as camadas comercial e da Web independentemente, dependendo da carga. Para reduzir possíveis complicações causadas pela necessidade de manter a afinidade do cliente, as VMs na camada da Web devem ser sem estado. As VMs que hospedam a lógica de negócios também devem ser sem estado.

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Simplifique o gerenciamento de todo o sistema usando ferramentas de administração centralizada, como [Automação do Azure][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef] ou [Puppet][puppet]. Essas ferramentas podem consolidar informações de diagnóstico e de integridade capturadas de várias VMs para fornecer uma visão geral do sistema.

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para essa arquitetura de referência está disponível no [GitHub][github-folder]. 

### <a name="prerequisites"></a>Pré-requisitos

Antes de implantar a arquitetura de referência para sua própria assinatura, você deve executar as etapas a seguir.

1. Clone, crie fork ou baixe o arquivo zip para as [arquiteturas de referência AzureCAT][ref-arch-repo] no repositório GitHub.

2. Verifique se a CLI do Azure 2.0 está instalada no computador. Para instalar a CLI, siga as instruções em [Instalar a CLI do Azure 2.0][azure-cli-2].

3. Instale os pacote npm dos [Blocos de construção do Azure][azbb].

  ```bash
  npm install -g @mspnp/azure-building-blocks
  ```

4. Em um prompt de comando, bash prompt ou prompt do PowerShell, faça logon na sua conta do Azure usando um dos comandos abaixo e siga os prompts.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>Implantar a solução usando azbb

Para implantar as VMs do Windows para uma arquitetura de referência de aplicativo de n camada, siga essas etapas:

1. Navegue até a pasta `virtual-machines\n-tier-windows` para o repositório que você clonou na etapa 1 dos pré-requisitos acima.

2. O arquivo de parâmetro especifica um nome de usuário e senha de administrador padrão para cada VM na implantação. Você deverá alterá-los antes de implantar a arquitetura de referência. Abra o arquivo `n-tier-windows.json` e substitua cada campo **adminUsername** e **adminPassword** com suas novas configurações.
  
  > [!NOTE]
  > Há vários scripts que são executados durante essa implantação tanto nos objetos **VirtualMachineExtension** quanto nas configurações de **extensões** para alguns dos objetos **VirtualMachine**. Alguns desses scripts requerem o nome de usuário e a senha do administrador que você acabou de alterar. É recomendável revisar esses scripts para garantir que você especificou as credenciais corretas. A implantação poderá falhar se você não especificou as credenciais corretas.
  > 
  > 

Salve o arquivo.

3. Implante a arquitetura de referência usando a ferramenta de linha de comando **azbb** conforme mostrado abaixo.

  ```bash
  azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
  ```

Para obter mais informações sobre a implantação dessa arquitetura de referência de exemplo utilizando blocos de construção Blocos de Construção do Azure, visite o repositório [GitHub][git].


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-application.md
[multi-vm]: multi-vm.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[host bastião]: https://en.wikipedia.org/wiki/Bastion_host
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
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-diagram.png "Arquitetura de N camadas usando o Microsoft Azure"