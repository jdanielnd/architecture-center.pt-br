---
title: Aplicativo de N camadas com Apache Cassandra
titleSuffix: Azure Reference Architectures
description: Execute máquinas virtuais com Linux para uma arquitetura de N camadas com Apache Cassandra no Microsoft Azure.
author: MikeWasson
ms.date: 11/12/2018
ms.custom: seodec18
ms.openlocfilehash: bbd1029fe17b5d88d54246127c5d8983a573b012
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120152"
---
# <a name="linux-n-tier-application-in-azure-with-apache-cassandra"></a>Aplicativo Linux de N camadas no Azure com Apache Cassandra

Essa arquitetura de referência mostra como implantar VMs (máquinas virtuais) e uma rede virtual configurada para um aplicativo de N camadas usando o Apache Cassandra no Linux para a camada de dados. [**Implantar esta solução**](#deploy-the-solution).

![Arquitetura de N camadas usando o Microsoft Azure](./images/n-tier-cassandra.png)

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura

A arquitetura tem os seguintes componentes:

- **Grupo de recursos**. [Grupos de recursos][resource-manager-overview] são utilizados para agrupar os recursos para que eles possam ser gerenciados pelo tempo de vida, o proprietário ou outros critérios.

- **VNet (Rede Virtual) e sub-redes**. Cada VM do Azure é implantada em uma VNet que pode ser segmentada em sub-redes. Sempre crie uma sub-rede separada para cada camada.

- **NSGs**. Use os NSGs [grupos de segurança de rede][nsg] para restringir o tráfego de rede na VNet. Por exemplo, na arquitetura de três camadas mostrada aqui, a camada de banco de dados aceita o tráfego da camada comercial e da sub-rede de gerenciamento, mas não do front-end da Web.

- **Proteção contra DDoS**. Embora a plataforma do Azure forneça proteção básica em relação a ataques de negação de serviço distribuído (DDoS), é recomendável usar [Proteção contra DDoS Standard][ddos], que melhorou os recursos de mitigação de DDoS. Confira [Considerações de segurança](#security-considerations).

- **Máquinas virtuais**. Para obter recomendações sobre como configurar máquinas virtuais, consulte [Executar uma VM do Windows no Azure](./windows-vm.md) e [Executar uma VM do Linux no Azure](./linux-vm.md).

- **Conjuntos de disponibilidade**. Crie um [conjunto de disponibilidade][azure-availability-sets] para cada camada e provisione pelo menos duas VMs em cada camada, o que torna as VMs qualificadas para um [contrato de nível de serviço (SLA)][vm-sla] maior.

- **Balanceadores de carga do Azure**. Os [balanceadores de carga] [ load-balancer] distribuem solicitações de entrada da Internet para as instâncias de VM. Use um [balanceador de carga público][load-balancer-external] para distribuir o tráfego de entrada da Internet para a camada da Web e um [balanceador de carga interno][load-balancer-internal] para distribuir o tráfego de rede da camada da Web para a camada comercial.

- **Endereço IP público**. É necessário ter um endereço IP para que o balanceador de carga possa receber o tráfego da Internet.

- **Jumpbox**. Também chamada de um [host bastião]. Uma VM protegida na rede que os administradores usam para se conectar às outras VMs. O jumpbox tem um NSG que permite o tráfego remoto apenas de endereços IP públicos em uma lista segura. O NSG deve permitir o tráfego SSH.

- **Banco de dados Apache Cassandra**. Fornece alta disponibilidade na camada de dados, habilitando replicação e failover.

- **DNS do Azure**. O [DNS do Azure][azure-dns] é um serviço de hospedagem para domínios DNS. Ele fornece resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure.

## <a name="recommendations"></a>Recomendações

Seus requisitos podem ser diferentes dos requisitos da arquitetura descrita aqui. Use essas recomendações como ponto de partida.

### <a name="vnet--subnets"></a>VNet / Sub-redes

Quando você cria a VNet, determine quantos endereços IP seus recursos em cada sub-rede exigem. Especifique uma máscara de sub-rede e um intervalo de endereços de VNet grande o suficiente para os endereços IP necessários, usando a notação [CIDR]. Use um espaço de endereço que esteja dentro dos [blocos de endereço IP privados][private-ip-space] padrão, que são 10.0.0.0/8, 172.16.0.0/12 e 192.168.0.0/16.

Escolha um intervalo de endereços que não se sobreponha ao da rede local, caso seja necessário configurar um gateway entre a VNet e a rede local mais tarde. Depois de criar a VNet, não será possível alterar o intervalo de endereços.

Crie as sub-redes levando em conta os requisitos de funcionalidade e de segurança. Todas as VMs na mesma camada ou função devem ir para a mesma sub-rede, que pode ser um limite de segurança. Para obter mais informações sobre como criar VNets e sub-redes, consulte [Planejar e criar redes virtuais do Azure][plan-network].

### <a name="load-balancers"></a>Balanceadores de carga

Não exponha as VMs diretamente à Internet. Em vez disso, dê um endereço IP privado a cada VM. Os clientes se conectam usando o endereço IP do balanceador de carga público.

Defina as regras do balanceador de carga para direcionar tráfego de rede para as VMs. Por exemplo, para permitir tráfego HTTP, crie uma regra que mapeie a porta 80 da configuração de front-end para a porta 80 no pool de endereços de back-end. Quando um cliente envia uma solicitação HTTP para a porta 80, o balanceador de carga seleciona um endereço IP de back-end usando um [algoritmo de hash][load-balancer-hashing] que inclui o endereço IP de origem. As solicitações de cliente são distribuídas por todas as VMs.

### <a name="network-security-groups"></a>Grupos de segurança de rede

Use as regras de NSG para restringir o tráfego entre as camadas. Por exemplo, na arquitetura de três camadas mostrada acima, a camada da Web não se comunica diretamente com a camada de banco de dados. Para impor isso, a camada de banco de dados deve bloquear o tráfego de entrada da sub-rede da camada da Web.

1. Negue todo o tráfego de entrada do VNet. (Use a marca `VIRTUAL_NETWORK` na regra.)
2. Permita o tráfego de entrada da sub-rede de camada de negócios.
3. Permita o tráfego de entrada da própria sub-rede de camada de dados. Essa regra permite a comunicação entre as VMs de banco de dados, que é necessária para failover e replicação de banco de dados.
4. Permita o tráfego SSH (porta 22) da sub-rede jumpbox. Essa regra permite que os administradores se conectem à camada de banco de dados do jumpbox.

Criar regras de 2 &ndash; 4 com prioridade mais alta que a primeira regra, para que elas a substituam.

### <a name="cassandra"></a>Cassandra

Recomendamos o [DataStax Enterprise][datastax] para uso de produção, mas essas recomendações se aplicam a qualquer edição do Cassandra. Para obter mais informações sobre como executar o DataStax no Azure, consulte [Guia de implantação do DataStax Enterprise no Azure][cassandra-in-azure].

Coloque as VMs para um cluster Cassandra em um conjunto de disponibilidade para garantir que as réplicas do Cassandra sejam distribuídas entre vários domínios de falha e de atualização. Para saber mais sobre domínios de falha e de atualização, consulte [Gerenciar a disponibilidade das máquinas virtuais][azure-availability-sets].

Configure três domínios de falha (o máximo) por conjunto de disponibilidade e 18 domínios de atualização por conjunto de disponibilidade. Isso fornece o número máximo de domínios de atualização que ainda podem ser distribuído uniformemente entre os domínios de falha.

Configure os nós no modo de reconhecimento de rack. Mapear domínios de falha para racks no arquivo `cassandra-rackdc.properties`.

Não é necessário um balanceador de carga na frente do cluster. O cliente se conecta diretamente a um nó no cluster.

Para alta disponibilidade, implante o Cassandra em mais de uma região do Azure. Os nós de cada região são configurados no modo de reconhecimento de rack com domínios de falha e de atualização para garantir a resiliência dentro da região.

### <a name="jumpbox"></a>Jumpbox

Não permita o acesso SSH da Internet pública às VMs que executam a carga de trabalho do aplicativo. Em vez disso, todo o acesso SSH a essas VMs deve ocorrer por meio do jumpbox. Um administrador faz logon no jumpbox e, em seguida, faz logon na VM por meio do jumpbox. O jumpbox permite o tráfego SSH da Internet, mas apenas de endereços IP conhecidos e seguros.

O jumpbox tem requisitos de desempenho mínimo, por isso, selecione uma VM de tamanho pequeno. Crie um [endereço IP público] para o jumpbox. Coloque o jumpbox na mesma VNet que as outras VMs, mas em uma sub-rede de gerenciamento separada.

Para proteger o jumpbox, adicione uma regra NSG que permite conexões SSH somente de um conjunto seguro de endereços IP públicos. Configure os NSGs para outras sub-redes para permitir o tráfego SSH da sub-rede de gerenciamento.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Para as camadas Web e business, considere o uso de [conjuntos de dimensionamento de máquinas virtuais][vmss], em vez de implantar VMs separadas em um conjunto de disponibilidade. Um conjunto de dimensionamento facilita a implantação e gerenciamento de um conjunto de VMs idênticas e o dimensionamento automático de VMs com base em métricas de desempenho. À medida que a carga nas VMs aumenta, VMs adicionais são acrescentadas automaticamente ao balanceador de carga. Considere usar os conjuntos de dimensionamento se você precisar aumentar as VMs rapidamente ou se precisar de dimensionamento automático.

Há duas maneiras básicas de configurar as VMs implantadas em um conjunto de dimensionamento:

- Use as extensões para configurar a VM depois que ela é implantada. Com essa abordagem, novas instâncias de VM podem levar mais tempo para ser iniciadas do que uma VM sem extensões.

- Implante um [disco gerenciado](/azure/storage/storage-managed-disks-overview) com uma imagem de disco personalizada. Essa opção pode ser mais rápida de implantar. Porém, isso requer que você mantenha a imagem atualizada.

Para obter mais informações, confira [Considerações de design para conjuntos de dimensionamento][vmss-design].

> [!TIP]
> Ao usar qualquer solução de dimensionamento automático, teste-a com cargas de trabalho no nível de produção com bastante antecedência.

Cada assinatura do Azure tem limites padrão em vigor, incluindo um número máximo de VMs por região. Você pode aumentar o limite enviando uma solicitação de suporte. Para saber mais, confira [Assinatura e limites de serviço, cotas e restrições do Azure][subscription-limits].

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Se você não usar conjuntos de dimensionamento de máquinas virtuais, coloque as VMs para a mesma camada em um conjunto de disponibilidade. Crie pelo menos duas VMs no conjunto de disponibilidade para dar suporte ao [SLA de disponibilidade para VMs do Azure][vm-sla]. Para obter mais informações, consulte [Gerenciar a disponibilidade de máquinas virtuais][availability-set]. Os conjuntos de dimensionamento automaticamente usam *grupos de posicionamento*, que funcionam como um conjunto de disponibilidade implícito.

O balanceador de carga utiliza [investigações de integridade][health-probes] para monitorar a disponibilidade de instâncias de VM. Se uma investigação não puder acessar uma instância dentro de um período de tempo limite, o balanceador de carga parará de enviar tráfego para essa VM. O balanceador de carga continuará a investigar e, se a VM ficar disponível novamente, o balanceador de carga reiniciará o envio de tráfego para ela.

Aqui estão algumas recomendações sobre as investigações de integridade do balanceador de carga:

- As investigações podem testar HTTP ou TCP. Se suas VMs são executadas em um servidor HTTP, crie uma investigação HTTP. Caso contrário, crie uma investigação TCP.
- Para uma investigação HTTP, especifique o caminho para um ponto de extremidade HTTP. A investigação verifica uma resposta HTTP 200 para esse caminho. Ele pode ser o caminho raiz (“/”) ou um ponto de extremidade de monitoramento de integridade que implementa lógica personalizada para verificar a integridade do aplicativo. O ponto de extremidade deve permitir solicitações HTTP anônimas.
- A investigação é enviada de um endereço IP [conhecido][health-probe-ip], 168.63.129.16. Verifique se o tráfego de ou para esse endereço IP não é bloqueado por quaisquer políticas de firewall ou regras de NSG.
- Use os [logs de investigação de integridade][health-probe-log] para exibir o status das investigações de integridade. Habilite o registro em log no Portal do Azure para cada balanceador de carga. Os logs são gravados no Armazenamento de Blobs do Azure. Os logs mostram quantas VMs não estão recebendo o tráfego de rede devido a respostas de investigação com falha.

Para o cluster Cassandra, os cenários de failover dependem dos níveis de consistência usados pelo aplicativo e do número de réplicas. Para níveis de consistência e uso no Cassandra, consulte [Configurando a consistência dos dados][cassandra-consistency] e [Cassandra: quantos nós se comunicam com o Quorum?][cassandra-consistency-usage] A disponibilidade de dados do Cassandra é determinada pelo nível de consistência usado pelo aplicativo e pelo mecanismo de replicação. Para replicação no Cassandra, consulte [Explicação de replicação de dados em Bancos de Dados NoSQL][cassandra-replication].

## <a name="security-considerations"></a>Considerações de segurança

Redes virtuais são um limite de isolamento de tráfego no Azure. As VMs em uma VNet não podem se comunicar diretamente com VMs em uma VNet diferente. As VMs na mesma VNet podem se comunicar, a menos que você crie NSGs ([Grupos de Segurança de Rede][nsg]) para restringir o tráfego. Para obter mais informações, consulte [Serviços em nuvem da Microsoft e segurança de rede][network-security].

Para tráfego de entrada da Internet, as regras do balanceador de carga definem qual tráfego pode alcançar o back-end. No entanto, as regras do balanceador de carga não dão suporte a listas de IP de confiança, por isso se você desejar adicionar determinados endereços IP públicos a uma lista de confiança, acrescente um NSG à sub-rede.

**Rede de Perímetro**. Considere adicionar uma NVA (solução de virtualização de rede) para criar uma DMZ entre a Internet e a rede virtual do Azure. NVA é um termo genérico para uma solução de virtualização que pode executar tarefas relacionadas à rede, como firewall, inspeção de pacotes, auditoria e roteamento personalizado. Para obter mais informações, consulte [Implementação de uma DMZ entre o Azure e a Internet][dmz].

**Criptografia**. Criptografe dados confidenciais em repouso e use o [Azure Key Vault][azure-key-vault] para gerenciar as chaves de criptografia de banco de dados. O Key Vault pode armazenar chaves de criptografia em HSMs (módulos de segurança de hardware). Também é recomendado para armazenar segredos do aplicativo, como cadeias de caracteres de conexão de banco de dados, no cofre de chaves.

**Proteção contra DDoS**. A plataforma do Azure fornece a proteção contra DDoS básica por padrão. Essa proteção básica se destina a proteger a infraestrutura do Azure como um todo. Embora a proteção contra DDoS básica esteja habilitada automaticamente, é recomendável usar a [Proteção contra DDoS Standard][ddos]. A proteção Standard utiliza ajuste adaptável, com base nos padrões de tráfego de rede do seu aplicativo, para detectar ameaças. Isso permite aplicar mitigações de risco contra ataques de DDoS que podem não ser detectadas pelas políticas de DDoS de toda a infraestrutura. A proteção Standard também fornece alertas, telemetria e análise por meio do Azure Monitor. Para saber mais, confira [Proteção contra DDoS do Azure: Melhores práticas e arquiteturas de referência][ddos-best-practices].

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para essa arquitetura de referência está disponível no [GitHub][github-folder].

### <a name="prerequisites"></a>Pré-requisitos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution-using-azbb"></a>Implantar a solução usando azbb

Para implantar as VMs do Linux para uma arquitetura de referência de aplicativo de n camada, siga essas etapas:

1. Navegue até a pasta `virtual-machines\n-tier-linux` para o repositório que você clonou na etapa 1 dos pré-requisitos acima.

2. O arquivo de parâmetro especifica um nome de usuário e senha de administrador padrão para cada VM na implantação. Altere-os antes de implantar a arquitetura de referência. Abra o arquivo `n-tier-linux.json` e substitua cada campo **adminUsername** e **adminPassword** com suas novas configurações.   Salve o arquivo.

3. Implante a arquitetura de referência usando a ferramenta **azbb** conforme mostrado abaixo.

   ```azurecli
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
   ```

Para obter mais informações sobre a implantação dessa arquitetura de referência de exemplo utilizando blocos de construção Blocos de Construção do Azure, visite o repositório [GitHub][git].

<!-- links -->

[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault

[host bastião]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: https://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: https://academy.datastax.com/planet-cassandra/data-replication-in-nosql-databases-explained
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2

[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[datastax]: https://www.datastax.com/products/datastax-enterprise
[ddos]: /azure/virtual-network/ddos-protection-overview
[ddos-best-practices]: /azure/security/azure-ddos-best-practices
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[endereço IP público]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx

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
