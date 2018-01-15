---
title: "Executar VMs do Linux em várias regiões do Azure para ter alta disponibilidade"
description: "Como implantar VMs em várias regiões no Azure para alta disponibilidade e resiliência."
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Linux VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 7d720a004d21edbffc0ddeba54e291aa817550e0
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/08/2018
---
# <a name="run-linux-vms-in-multiple-regions-for-high-availability"></a>Executar VMs do Linux em várias regiões para ter alta disponibilidade

Essa arquitetura de referência mostra um conjunto de práticas comprovadas para executar um aplicativo de N camadas em várias regiões do Azure, a fim de alcançar a disponibilidade e uma infraestrutura de recuperação de desastres robusta. 

![[0]][0]

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura 

Essa arquitetura baseia-se naquela mostrada em [Executar VMs do Linux para um aplicativo de N camadas](n-tier.md). 

* **Regiões primárias e secundárias**. Use duas regiões para obter alta disponibilidade. Uma é a região primária, a outra destina-se a failover.
* **DNS do Azure**. [DNS do Azure][azure-dns] é um serviço de hospedagem para domínios DNS, que fornece resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure.
* **Gerenciador de Tráfego do Microsoft Azure**. O [Gerenciador de Tráfego][traffic-manager] roteia as solicitações de entrada para uma das regiões. Durante as operações normais, ele roteia as solicitações para a região primária. Se essa região ficar indisponível, o Gerenciador de Tráfego fará failover para a região secundária. Para obter mais informações, consulte a [Configuração do Gerenciador de Tráfego](#traffic-manager-configuration).
* **Grupos de recursos**. Crie [grupos de recursos][resource groups] separados para a região primária, a região secundária e para o Gerenciador de Tráfego. Isso oferece flexibilidade para gerenciar cada região como uma única coleção de recursos. Por exemplo, você poderia reimplantar uma região sem interromper a outra. [Vincule os grupos de recursos][resource-group-links] para ser possível executar uma consulta para listar todos os recursos para o aplicativo.
* **VNets**. Crie uma VNet separada para cada região. Verifique se que os espaços de endereço não se sobrepõem.
* **Apache Cassandra**. Implante o Cassandra em data centers nas regiões do Azure para ter alta disponibilidade. Dentro de cada região, os nós são configurados no modo de reconhecimento de rack com domínios de falha e de atualização para garantir a resiliência dentro da região.

## <a name="recommendations"></a>Recomendações

Uma arquitetura de várias regiões pode fornecer uma disponibilidade maior que a implantação em uma única região. Se uma interrupção regional afetar a região primária, você poderá usar o [Gerenciador de Tráfego][traffic-manager] para fazer failover para a região secundária. Essa arquitetura também poderá ajudar a se um subsistema individual do aplicativo falhar.

Há várias abordagens gerais para alcançar alta disponibilidade em várias regiões:   

* Ativo/passivo com espera ativa. O tráfego vai para uma região, enquanto a outra aguarda em espera ativa. A espera ativa significa que as VMs na região secundária são alocadas e ficam em execução a todo momentos.
* Ativo/passivo com espera passiva. O tráfego vai para uma região, enquanto a outra aguarda em espera passiva. A espera passiva significa que as VMs na região secundária não são alocadas até serem necessárias para failover. Essa abordagem custa menos para ser executada, mas geralmente leva mais tempo para ficar online durante uma falha.
* Ativa/ativa. Ambas as regiões ficam ativas e a carga das solicitações é balanceada entre elas. Se uma região ficar indisponível, ela será retirada da rotação. 

Essa arquitetura de referência se concentra em ativo/passivo com espera ativa, usando o Gerenciador de Tráfego para failover. Observe que você poderia implantar um pequeno número de VMs para espera ativa e, em seguida, aumentar conforme necessário.


### <a name="regional-pairing"></a>Emparelhamento regional

Cada região do Azure é emparelhada com outra na mesma área geográfica. Em geral, escolha regiões do mesmo par regional (por exemplo, Leste dos EUA 2 e Centro dos EUA). Os benefícios de se fazer isso são:

* No caso de uma interrupção ampla, a recuperação de pelo menos uma região de cada par é priorizada.
* As atualizações planejadas do sistema do Azure são distribuídas em regiões emparelhadas sequencialmente para minimizar o possível tempo de inatividade.
* Os pares residem na mesma geografia para atender aos requisitos de residência de dados.

No entanto, verifique se ambas as regiões dão suporte a todos os serviços do Azure necessários para seu aplicativo (consulte [Serviços por região][services-by-region]). Para saber mais sobre pares regionais, consulte [Continuidade dos negócios e recuperação de desastres (BCDR): Regiões Emparelhadas do Azure][regional-pairs].

### <a name="traffic-manager-configuration"></a>Configuração do Gerenciador de Tráfego

Considere os seguintes pontos ao configurar o Gerenciador de Tráfego:

* **Roteamento**. O Gerenciador de Tráfego dá suporte a vários [algoritmos de roteamento][tm-routing]. Para o cenário descrito neste artigo, use roteamento *prioritário* (anteriormente chamado de roteamento de *failover*). Com essa configuração, o Gerenciador de Tráfego envia todas as solicitações para a região primária, a menos que ela fique inacessível. Nesse ponto, ele automaticamente faz failover para a região secundária. Consulte [Configurar o método de roteamento de failover][tm-configure-failover].
* **Investigação de integridade**. O Gerenciador de Tráfego usa uma [investigação][tm-monitoring] HTTP (ou HTTPS) para monitorar a disponibilidade de cada região. A investigação verifica uma resposta HTTP 200 para um caminho de URL especificado. Como uma prática recomendada, crie um ponto de extremidade que relata a integridade geral do aplicativo e use esse ponto de extremidade para a investigação de integridade. Caso contrário, a investigação pode relatar um ponto de extremidade íntegro quando partes essenciais do aplicativo estão falhando na verdade. Para obter mais informações, consulte o [Padrão de monitoramento de ponto de extremidade de integridade][health-endpoint-monitoring-pattern].

Quando o Gerenciador de Tráfego faz failover, há um período em que os clientes não podem acessar o aplicativo. A duração é afetada pelos seguintes fatores:

* A investigação de integridade precisa detectar que a região primária ficou inacessível.
* Os servidores DNS precisam atualizar os registros DNS armazenados em cache para o endereço IP, dependendo da TTL (vida útil) DNS. A TTL padrão é 300 segundos (5 minutos), mas você pode configurar esse valor ao criar o perfil do Gerenciador de Tráfego.

Para saber mais, consulte [Sobre o monitoramento do Gerenciador de Tráfego][tm-monitoring].

Em caso de falha do Gerenciador de Tráfego, é recomendável executar um failback manual em vez de implementar um failback automático. Caso contrário, você pode criar uma situação em que o aplicativo fica alternando entre as regiões. Verifique se todos os subsistemas de aplicativo estão íntegros antes de realizar o failback.

Observe que o Gerenciador de Tráfego realiza failback automaticamente por padrão. Para evitar isso, diminua manualmente a prioridade da região primária após um evento de failover. Por exemplo, suponha que a região primária tem prioridade 1 e a secundária tem prioridade 2. Após um failover, defina a região primária para a prioridade 3, para impedir o failback automático. Quando você estiver pronto retornar para ela, atualize a prioridade para 1.

O seguinte comando da [CLI do Azure][install-azure-cli] atualiza a prioridade:

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

Outra abordagem é desabilitar temporariamente o ponto de extremidade até que você esteja pronto para realizar failback:

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```    

Dependendo da causa de um failover, será necessário reimplantar os recursos dentro de uma região. Antes de fazer o failback, execute um teste de prontidão operacional. O teste deverá verificar o seguinte:

* As VMs estão configuradas corretamente. (Todo o software necessário está instalado, o IIS está em execução e assim por diante.)
* Os subsistemas de aplicativos estão íntegros.
* Testes funcionais. (Por exemplo, a camada de banco de dados pode ser acessada da camada da Web.)

### <a name="cassandra-deployment-across-multiple-regions"></a>Implantação de Cassandra em várias regiões

Data centers Cassandra são um grupo de nós de dados relacionados que são configurados juntos em um cluster para replicação e diferenciação de carga de trabalho.

Recomendamos o [DataStax Enterprise][datastax] para uso na produção. Para obter mais informações sobre como executar o DataStax no Azure, consulte [Guia de implantação do DataStax Enterprise no Azure][cassandra-in-azure]. As seguintes recomendações gerais se aplicam a qualquer edição do Cassandra: 

* Atribua um endereço IP público para cada nó. Isso permite que os clusters se comuniquem entre regiões usando a infraestrutura de backbone do Azure, fornecendo alta taxa de transferência a baixo custo.
* Proteja os nós usando o firewall e as configurações de NSG (grupo de segurança de rede) apropriados, permitindo somente o tráfego de e para hosts conhecidos, incluindo os clientes e outros nós de cluster. Observe que o Cassandra usa portas diferentes para comunicação, OpsCenter, Spark e assim por diante. Para ver mais sobre o uso de portas no Cassandra, consulte [Configurando o acesso de porta de firewall][cassandra-ports].
* Use criptografia SSL para todas as comunicações [cliente para nó][ssl-client-node] e [nó para nó][ssl-node-node].
* Dentro de uma região, siga as diretrizes das [Recomendações do Cassandra](n-tier.md#cassandra).

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Com um aplicativo complexo de N camadas, pode não ser necessário replicar o aplicativo inteiro na região secundária. Em vez disso, você pode replicar apenas um subsistema crítico que é necessário para dar suporte à continuidade de negócios.

O Gerenciador de Tráfego é um possível ponto de falha no sistema. Se o serviço do Gerenciador de Tráfego falhar, os clientes não poderão acessar seu aplicativo durante o tempo de inatividade. Examine o [SLA do Gerenciador de Tráfego][tm-sla] e determine se usar apenas o Gerenciador de Tráfego atende aos seus requisitos de negócios para alta disponibilidade. Caso contrário, considere adicionar outra solução de gerenciamento de tráfego como um failback. Se o serviço do Gerenciador de Tráfego do Azure falhar, altere os registros CNAME no DNS para apontar para outro serviço de gerenciamento de tráfego. (Esta etapa deve ser executada manualmente e seu aplicativo estará indisponível até que as alterações de DNS sejam propagadas.)

Para o cluster Cassandra, os cenários de failover a considerar dependem dos níveis de consistência usados pelo aplicativo, bem como o número de réplicas usadas. Para níveis de consistência e uso no Cassandra, consulte [Configurando a consistência dos dados][cassandra-consistency] e [Cassandra: quantos nós se comunicam com o Quorum?][cassandra-consistency-usage] A disponibilidade de dados do Cassandra é determinada pelo nível de consistência usado pelo aplicativo e pelo mecanismo de replicação. Para replicação no Cassandra, consulte [Explicação de replicação de dados em Bancos de Dados NoSQL][cassandra-replication].

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Quando você atualizar a implantação, atualize uma região de cada vez para reduzir a chance de uma falha global devido a uma configuração incorreta ou um erro no aplicativo.

Teste a resiliência do sistema a falhas. Aqui estão alguns cenários comuns de falha para testar:

* Desligamento de instâncias de VM.
* Recursos de pressão, como CPU e memória.
* Desconectar/atraso de rede.
* Falha de processos.
* Expiração de certificados.
* Simular falhas de hardware.
* Desligamento do serviço DNS nos controladores de domínio.

Meça o tempo de recuperação e verifique se ele cumpre seus requisitos de negócios. Teste também combinações dos modos de falha.


<!-- Links -->
[hybrid-vpn]: ../hybrid-networking/vpn.md
[azure-dns]: /azure/dns/dns-overview
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: http://www.planetcassandra.org/data-replication-in-nosql-databases-explained/
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2
[cassandra-ports]: https://docs.datastax.com/en/datastax_enterprise/5.0/datastax_enterprise/sec/configFirewallPorts.html
[datastax]: https://www.datastax.com/products/datastax-enterprise
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssl-client-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLClientToNode_t.html
[ssl-node-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLNodeToNode_t.html
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx
[0]: ./images/multi-region-application-diagram.png "Arquitetura de rede altamente disponível para aplicativos de N camadas do Azure"
