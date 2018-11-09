---
title: Aplicativo de n camadas com SQL Server
description: Como implementar uma arquitetura multicamadas no Azure, para obter disponibilidade, segurança, escalabilidade e capacidade de gerenciamento.
author: MikeWasson
ms.date: 09/13/2018
ms.openlocfilehash: 6ddad853240b9a51ac904c06783a5c0b56dedec5
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916508"
---
# <a name="n-tier-application-with-sql-server"></a>Aplicativo de n camadas com SQL Server

Essa arquitetura de referência mostra como implantar VMs e uma rede virtual configurada para um aplicativo de N camadas usando o SQL Server no Windows para a camada de dados. [**Implante essa solução**.](#deploy-the-solution) 

![[0]][0]

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura 

A arquitetura tem os seguintes componentes:

* **Grupo de recursos.** [Grupos de recursos][resource-manager-overview] são utilizados para agrupar os recursos para que eles possam ser gerenciados pelo tempo de vida, o proprietário ou outros critérios.

* **Rede Virtual (VNet) e sub-redes.** Cada VM do Azure é implantada em uma VNet que pode ser segmentada em várias sub-redes. Sempre crie uma sub-rede separada para cada camada. 

* **Gateway de aplicativo**. [O Gateway de Aplicativo do Azure](/azure/application-gateway/) é um balanceador de carga de camada 7. Nessa arquitetura, ele roteia as solicitações HTTP para o front-end da web. Gateway de Aplicativo também fornece um [firewall do aplicativo Web](/azure/application-gateway/waf-overview) (WAF) que protege o aplicativo contra vulnerabilidades e explorações comuns. 

* **NSGs.** Use os NSGs [grupos de segurança de rede][nsg] para restringir o tráfego de rede na VNet. Por exemplo, na arquitetura de três camadas mostrada aqui, a camada de banco de dados não aceita o tráfego de front-end da Web, somente da camada comercial e da sub-rede de gerenciamento.

* **Máquinas virtuais**. Para obter recomendações sobre como configurar máquinas virtuais, consulte [Executar uma VM do Windows no Azure](./windows-vm.md) e [Executar uma VM do Linux no Azure](./linux-vm.md).

* **Conjuntos de disponibilidade.** Crie um [conjunto de disponibilidade][azure-availability-sets] para cada camada e provisione pelo menos duas VMs em cada camada. Isso torna as VMs qualificadas para um [SLA (Contrato de Nível de Serviço)][vm-sla] mais elevado. 

* **Conjunto de dimensionamento de VM** (não mostrado). Um [Conjunto de dimensionamento de VM] [ vmss] é uma alternativa ao uso de um conjunto de disponibilidade. Um conjunto de dimensionamento facilita o escalonamento horizontal de VMs em uma camada, seja manual ou automaticamente, com base em regras predefinidas.

* **Balanceadores de carga.** Use o [Azure Load Balancer] [ load-balancer] para distribuir o tráfego de rede da camada da Web para a camada comercial e da camada comercial para o SQL Server.

* **Endereço IP público**. É necessário ter um endereço IP público para que aplicativo possa receber o tráfego da Internet.

* **Jumpbox.** Também chamada de um [host bastião]. Uma VM protegida na rede que os administradores usam para se conectar às outras VMs. O jumpbox tem um NSG que permite o tráfego remoto apenas de endereços IP públicos em uma lista segura. O NSG deve permitir o tráfego de RDP (área de trabalho remota).

* **Grupo de Disponibilidade Always On do SQL Server.** Fornece alta disponibilidade na camada de dados, habilitando replicação e failover. Usa a tecnologia WSFC (Cluster de Failover do Windows Server) para o failover.

* **Servidores AD DS (Active Directory Domain Services)**. Os objetos de computação do cluster de failover e suas funções em cluster associadas são criados no Active Directory Domain Services (AD DS).

* **Testemunha de Nuvem**. Um cluster de failover requer mais da metade dos seus nós em execução, que é conhecido como ter quorum. Se o cluster tem apenas dois nós, uma partição de rede pode fazer com que cada nó pense que é o principal. Nesse caso, é necessário uma *testemunha* para desempatar e estabelecer o quorum. Testemunha é um recurso, como um disco compartilhado, que pode agir como um desempate para estabelecer o quorum. A Testemunha de Nuvem é um tipo que usa o Armazenamento de Blobs do Azure. Para saber mais sobre o conceito de quorum, consulte [Entendendo o cluster e o quorum de pool](/windows-server/storage/storage-spaces/understand-quorum). Para obter mais informações sobre a Testemunha de Nuvem, consulte [Implantar uma Testemunha de Nuvem para um Cluster de Failover](/windows-server/failover-clustering/deploy-cloud-witness). 

* **DNS do Azure**. [DNS do Azure][azure-dns] é um serviço de hospedagem para domínios DNS, que fornece resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure.

## <a name="recommendations"></a>Recomendações

Seus requisitos podem ser diferentes dos requisitos da arquitetura descrita aqui. Use essas recomendações como ponto de partida. 

### <a name="vnet--subnets"></a>VNet / Sub-redes

Quando você cria a VNet, determine quantos endereços IP seus recursos em cada sub-rede exigem. Especifique uma máscara de sub-rede e um intervalo de endereços de VNet grande o suficiente para os endereços IP necessários, usando a notação [CIDR]. Use um espaço de endereço que esteja dentro dos [blocos de endereço IP privados][private-ip-space] padrão, que são 10.0.0.0/8, 172.16.0.0/12 e 192.168.0.0/16.

Escolha um intervalo de endereços que não se sobreponha ao da rede local para caso seja necessário configurar um gateway entre a VNet e a rede local mais tarde. Depois de criar a VNet, não será possível alterar o intervalo de endereços.

Crie as sub-redes levando em conta os requisitos de funcionalidade e de segurança. Todas as VMs na mesma camada ou função devem ir para a mesma sub-rede, que pode ser um limite de segurança. Para obter mais informações sobre como criar VNets e sub-redes, consulte [Planejar e criar redes virtuais do Azure][plan-network].

### <a name="load-balancers"></a>Balanceadores de carga

Não exponha as VMs diretamente à Internet, concedendo, em vez disso, um endereço IP privado a cada VM. Os clientes se conectam usando o endereço IP público associado com o Gateway de Aplicativo.

Defina as regras do balanceador de carga para direcionar tráfego de rede para as VMs. Por exemplo, para permitir tráfego HTTP, crie uma regra que mapeie a porta 80 da configuração de front-end para a porta 80 no pool de endereços de back-end. Quando um cliente envia uma solicitação HTTP para a porta 80, o balanceador de carga seleciona um endereço IP de back-end usando um [algoritmo de hash][load-balancer-hashing] que inclui o endereço IP de origem. Dessa forma, as solicitações de cliente são distribuídas por todas as VMs.

### <a name="network-security-groups"></a>Grupos de segurança de rede

Use as regras de NSG para restringir o tráfego entre as camadas. Por exemplo, a arquitetura de três camadas mostrada acima, a camada da Web não se comunica diretamente com a camada de banco de dados. Para impor isso, a camada de banco de dados deve bloquear o tráfego de entrada da sub-rede da camada da Web.  

1. Negue todo o tráfego de entrada do VNet. (Use a marca `VIRTUAL_NETWORK` na regra.) 
2. Permita o tráfego de entrada da sub-rede de camada de negócios.  
3. Permita o tráfego de entrada da própria sub-rede de camada de dados. Essa regra permite a comunicação entre as VMs de banco de dados, que é necessária para failover e replicação de banco de dados.
4. Permita o tráfego RDP (porta 3389) da sub-rede jumpbox. Essa regra permite que os administradores se conectem à camada de banco de dados do jumpbox.

Criar regras de 2 &ndash; 4 com prioridade mais alta que a primeira regra, para que elas a substituam.


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

Não permita o acesso RDP da Internet pública para as VMs que executam a carga de trabalho do aplicativo. Em vez disso, todo o acesso RDP a essas VMs deve ocorrer por meio do jumpbox. Um administrador faz logon no jumpbox e, em seguida, faz logon na VM por meio do jumpbox. O jumpbox permite que tráfego RDP da Internet, mas apenas de endereços IP conhecidos e seguros.

O jumpbox tem requisitos de desempenho mínimo, por isso, selecione uma VM de tamanho pequeno. Crie um [endereço IP público] para o jumpbox. Coloque o jumpbox na mesma VNet que as outras VMs, mas em uma sub-rede de gerenciamento separada.

Para proteger o jumpbox, adicione uma regra NSG que permite conexões RDP somente de um conjunto seguro de endereços IP públicos. Configure os NSGs para outras sub-redes para permitir o tráfego RDP da sub-rede de gerenciamento.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

[Conjuntos de dimensionamento de VM] [ vmss] ajudam a implantar e gerenciar um conjunto de VMs idênticas. Conjuntos de dimensionamento dão suporte ao dimensionamento automático com base nas métricas de desempenho. À medida que a carga nas VMs aumenta, VMs adicionais são acrescentadas automaticamente ao balanceador de carga. Considere usar os conjuntos de dimensionamento se você precisar aumentar as VMs rapidamente ou se precisar de dimensionamento automático.

Há duas maneiras básicas de configurar as VMs implantadas em um conjunto de dimensionamento:

- Use as extensões para configurar a VM depois que ela é provisionada. Com essa abordagem, novas instâncias de VM podem levar mais tempo para ser iniciadas do que uma VM sem extensões.

- Implante um [disco gerenciado](/azure/storage/storage-managed-disks-overview) com uma imagem de disco personalizada. Essa opção pode ser mais rápida de implantar. Porém, isso requer que você mantenha a imagem atualizada.

Para ver considerações adicionais, consulte [Considerações de design para conjuntos de dimensionamento][vmss-design].

> [!TIP]
> Ao usar qualquer solução de dimensionamento automático, teste-a com cargas de trabalho no nível de produção com bastante antecedência.

Cada assinatura do Azure tem limites padrão em vigor, incluindo um número máximo de VMs por região. Você pode aumentar o limite enviando uma solicitação de suporte. Para saber mais, confira [Assinatura e limites de serviço, cotas e restrições do Azure][subscription-limits].

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Se você não estiver usando conjuntos de dimensionamento de VM, coloque as VMs na mesma camada em um conjunto de disponibilidade. Crie pelo menos duas VMs no conjunto de disponibilidade para dar suporte ao [SLA de disponibilidade para VMs do Azure][vm-sla]. Para obter mais informações, consulte [Gerenciar a disponibilidade de máquinas virtuais][availability-set]. 

O balanceador de carga utiliza [investigações de integridade][health-probes] para monitorar a disponibilidade de instâncias de VM. Se uma investigação não puder acessar uma instância dentro de um período de tempo limite, o balanceador de carga parará de enviar tráfego para essa VM. Contudo, o balanceador de carga continuará a investigar e, se a VM ficar disponível novamente, o balanceador de carga reiniciará o envio de tráfego para ela.

Aqui estão algumas recomendações sobre as investigações de integridade do balanceador de carga:

* As investigações podem testar HTTP ou TCP. Se suas VMs são executadas em um servidor HTTP, crie uma investigação HTTP. Caso contrário, crie uma investigação TCP.
* Para uma investigação HTTP, especifique o caminho para um ponto de extremidade HTTP. A investigação verifica uma resposta HTTP 200 para esse caminho. Ele pode ser o caminho raiz (“/”) ou um ponto de extremidade de monitoramento de integridade que implementa lógica personalizada para verificar a integridade do aplicativo. O ponto de extremidade deve permitir solicitações HTTP anônimas.
* A investigação é enviada de um endereço IP [conhecido][health-probe-ip], 168.63.129.16. Verifique se o tráfego de ou para esse endereço IP não é bloqueado por quaisquer políticas de firewall ou regras de NSG (Grupo de Segurança de Rede).
* Use os [logs de investigação de integridade][health-probe-log] para exibir o status das investigações de integridade. Habilite o registro em log no Portal do Azure para cada balanceador de carga. Os logs são gravados no Armazenamento de Blobs do Azure. Os logs de mostram como muitas VMs no back-end não estão recebendo o tráfego de rede devido a respostas de investigação com falha.

Se você precisar de disponibilidade mais alta do que a fornecida pelo [SLA do Azure SLA para VMs][vm-sla], replique o aplicativo em duas regiões e use o Gerenciador de Tráfego do Azure para failover. Para obter mais informações, consulte [Aplicativo de N camadas de várias regiões para alta disponibilidade][multi-dc].  

## <a name="security-considerations"></a>Considerações de segurança

Redes virtuais são um limite de isolamento de tráfego no Azure. As VMs em uma VNet não podem se comunicar diretamente com VMs em uma VNet diferente. As VMs na mesma VNet podem se comunicar, a menos que você crie NSGs ([Grupos de Segurança de Rede][nsg]) para restringir o tráfego. Para obter mais informações, consulte [Serviços em nuvem da Microsoft e segurança de rede][network-security].

Considere adicionar uma NVA (solução de virtualização de rede) para criar uma DMZ entre a Internet e a rede virtual do Azure. NVA é um termo genérico para uma solução de virtualização que pode executar tarefas relacionadas à rede, como firewall, inspeção de pacotes, auditoria e roteamento personalizado. Para obter mais informações, consulte [Implementação de uma DMZ entre o Azure e a Internet][dmz].

Criptografe dados confidenciais em repouso e use o [Azure Key Vault][azure-key-vault] para gerenciar as chaves de criptografia de banco de dados. O Key Vault pode armazenar chaves de criptografia em HSMs (módulos de segurança de hardware). Para mais informações, consulte [Configurar a Integração do Azure Key Vault para o SQL nas VMs do Azure][sql-keyvault]. Também é recomendado para armazenar segredos do aplicativo, como cadeias de caracteres de conexão de banco de dados, no cofre de chaves.

É recomendável habilitar a [Proteção contra DDoS Standard](/azure/virtual-network/ddos-protection-overview), que fornece a mitigação de DDoS adicional para os recursos em uma VNet. Embora a proteção contra DDoS básica seja habilitada automaticamente como parte da plataforma Azure, a Proteção contra DDoS Standard fornece funcionalidades de mitigação ajustadas especificamente para os recursos da Rede Virtual do Azure.  

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para essa arquitetura de referência está disponível no [GitHub][github-folder]. Observe que a implantação inteira pode levar até duas horas, que inclui a execução de scripts para configurar o AD DS, o cluster de failover do Windows Server e o grupo de disponibilidade do SQL Server.

### <a name="prerequisites"></a>Pré-requisitos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution"></a>Implantar a solução

1. Execute o seguinte comando para criar um grupo de recursos.

    ```bash
    az group create --location <location> --name <resource-group-name>
    ```

2. Execute o seguinte comando para criar uma conta de Armazenamento para a Testemunha de Nuvem.

    ```bash
    az storage account create --location <location> \
      --name <storage-account-name> \
      --resource-group <resource-group-name> \
      --sku Standard_LRS
    ```

3. Navegue para a pasta `virtual-machines\n-tier-windows` do repositório GitHub de arquiteturas de referência.

4. Abra o arquivo `n-tier-windows.json` . 

5. Procurar todas as instâncias de "witnessStorageBlobEndPoint" e substitua o texto do espaço reservado pelo nome da conta de Armazenamento da etapa 2.

    ```json
    "witnessStorageBlobEndPoint": "https://[replace-with-storageaccountname].blob.core.windows.net",
    ```

6. Execute o comando a seguir para enumerar as chaves de conta para a conta de armazenamento.

    ```bash
    az storage account keys list \
      --account-name <storage-account-name> \
      --resource-group <resource-group-name>
    ```

    A saída deve parecer com o seguinte. Copie o valor de `key1`.

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

7. No arquivo `n-tier-windows.json`, procure todas as instâncias de "witnessStorageAccountKey e cole na chave de conta.

    ```json
    "witnessStorageAccountKey": "[replace-with-storagekey]"
    ```

8. No arquivo `n-tier-windows.json`, procure todas as instâncias de `[replace-with-password]` e `[replace-with-sql-password]` substitua-as por uma senha forte. Salve o arquivo.

    > [!NOTE]
    > Se você alterar o nome de usuário do administrador, também terá que atualizar os blocos `extensions` no arquivo JSON. 

9. Execute o seguinte comando para implantar a arquitetura.

    ```bash
    azbb -s <your subscription_id> -g <resource_group_name> -l <location> -p n-tier-windows.json --deploy
    ```

Para obter mais informações sobre a implantação dessa arquitetura de referência de exemplo utilizando blocos de construção Blocos de Construção do Azure, visite o repositório [GitHub][git].


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-sql-server.md
[n-tier]: n-tier.md
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[host bastião]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[endereço IP público]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
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
[Zabbix]: https://www.zabbix.com/
[Icinga]: https://www.icinga.org/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-sql-server.png "Arquitetura de N camadas usando o Microsoft Azure"
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[load-balancer]: /azure/load-balancer/
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[subscription-limits]: /azure/azure-subscription-service-limits
[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[network-security]: /azure/best-practices-network-security
