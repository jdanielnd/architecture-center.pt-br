---
title: Aplicativo Web de várias regiões
description: Arquitetura recomendada para aplicativo Web com alta disponibilidade em execução no Microsoft Azure.
author: MikeWasson
ms.date: 10/25/2018
ms.openlocfilehash: 7780a3d2f39f398d2776c783fbc3922c3353e44c
ms.sourcegitcommit: 877777094b554559dc9cb1f0d9214d6d38197439
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/11/2018
ms.locfileid: "51527653"
---
# <a name="run-a-web-application-in-multiple-azure-regions"></a>Executar um aplicativo Web em várias regiões do Azure

Essa arquitetura de referência mostra como executar um aplicativo do Serviço de Aplicativo do Azure em várias regiões para obter alta disponibilidade. 

![Arquitetura de referência: aplicativo Web com alta disponibilidade](./images/multi-region-web-app-diagram.png) 

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura 

Essa arquitetura baseia-se naquela mostrada em [Melhorar a escalabilidade em um aplicativo Web][guidance-web-apps-scalability]. As principais diferenças são:

* **Regiões primárias e secundárias**. Essa arquitetura usa duas regiões para obter alta disponibilidade. O aplicativo está implantado em cada região. Durante as operações normais, o tráfego de rede é roteado para a região primária. Se a região primária ficar indisponível, o tráfego será roteado para a região secundária. 
* **DNS do Azure**. [DNS do Azure][azure-dns] é um serviço de hospedagem para domínios DNS, que fornece resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure.
* **Gerenciador de Tráfego do Microsoft Azure**. O [Gerenciador de Tráfego][traffic-manager] roteia as solicitações de entrada para a região primária. Se o aplicativo em execução nessa região ficar indisponível, o Gerenciador de Tráfego fará failover para a região secundária.
* **Replicação geográfica** do Banco de Dados SQL e Cosmos DB. 

Uma arquitetura de várias regiões pode fornecer uma disponibilidade maior que a implantação em uma única região. Se uma interrupção regional afetar a região primária, você poderá usar o [Gerenciador de Tráfego][traffic-manager] para fazer failover para a região secundária. Essa arquitetura também poderá ajudar a se um subsistema individual do aplicativo falhar.

Há várias abordagens gerais para alcançar alta disponibilidade em várias regiões: 

* Ativo/passivo com espera ativa. O tráfego vai para uma região, enquanto a outra aguarda em espera ativa. A espera ativa significa que as VMs na região secundária são alocadas e ficam em execução a todo momentos.
* Ativo/passivo com espera passiva. O tráfego vai para uma região, enquanto a outra aguarda em espera passiva. A espera passiva significa que as VMs na região secundária não são alocadas até serem necessárias para failover. Essa abordagem custa menos para ser executada, mas geralmente leva mais tempo para ficar online durante uma falha.
* Ativa/ativa. Ambas as regiões ficam ativas e a carga das solicitações é balanceada entre elas. Se uma região ficar indisponível, ela será retirada da rotação. 

Essa arquitetura de referência se concentra em ativo/passivo com espera ativa, usando o Gerenciador de Tráfego para o failover. 


## <a name="recommendations"></a>Recomendações

Seus requisitos podem ser diferentes dos requisitos da arquitetura descrita aqui. Use as recomendações nesta seção como um ponto inicial.

### <a name="regional-pairing"></a>Emparelhamento regional
Cada região do Azure é emparelhada com outra na mesma área geográfica. Em geral, escolha regiões do mesmo par regional (por exemplo, Leste dos EUA 2 e EUA Central). Os benefícios de se fazer isso são:

* No caso de uma interrupção ampla, a recuperação de pelo menos uma região de cada par é priorizada.
* As atualizações planejadas do sistema do Azure são distribuídas em regiões emparelhadas sequencialmente para minimizar o possível tempo de inatividade.
* Na maioria dos casos, pares regionais residem na mesma geografia para atender aos requisitos de residência de dados.

No entanto, verifique se ambas as regiões dão suporte a todos os serviços do Azure necessários para seu aplicativo. Consulte [Serviços por região][services-by-region]. Para saber mais sobre pares regionais, consulte [Continuidade dos negócios e recuperação de desastres (BCDR): Regiões Emparelhadas do Azure][regional-pairs].

### <a name="resource-groups"></a>Grupos de recursos
Considere a possibilidade de colocar a região primária, a região secundária e o Gerenciador de Tráfego em [grupos de recursos][resource groups] separados. Isso permite que você gerencie os recursos implantados para cada região como uma única coleção.

### <a name="traffic-manager-configuration"></a>Configuração do Gerenciador de Tráfego 

**Roteamento**. O Gerenciador de Tráfego dá suporte a vários [algoritmos de roteamento][tm-routing]. Para o cenário descrito neste artigo, use roteamento *prioritário* (anteriormente chamado de roteamento de *failover*). Com essa configuração, o Gerenciador de Tráfego envia todas as solicitações para a região primária, a menos que o ponto de extremidade para essa região fique inacessível. Nesse ponto, ele automaticamente faz failover para a região secundária. Consulte [Configurar o método de roteamento de failover][tm-configure-failover].

**Investigação de integridade**. O Gerenciador de Tráfego usa uma investigação HTTP (ou HTTPS) para monitorar a disponibilidade de cada ponto de extremidade. A investigação fornece ao Gerenciador de Tráfego um teste aprovado/reprovado para realizar o failover para a região secundária. Ele funciona com o envio de uma solicitação para um caminho de URL especificado. Se ele obtiver uma resposta não 200 dentro de um período de tempo limite, o teste falhará. Após quatro solicitações com falha, o Gerenciador de Tráfego marca o ponto de extremidade como degradado e faz failover para outro ponto de extremidade. Para ver mais detalhes, consulte [Monitoramento e failover do ponto de extremidade do Gerenciador de Tráfego][tm-monitoring].

Como uma prática recomendada, crie um ponto de extremidade de investigação de integridade que relata a integridade geral do aplicativo e use esse ponto de extremidade para a investigação de integridade. O ponto de extremidade deve verificar dependências críticas, como aplicativos do Serviço de Aplicativo, a fila de armazenamento e o Banco de Dados SQL. Caso contrário, a investigação pode relatar um ponto de extremidade íntegro quando partes essenciais do aplicativo estão falhando na verdade.

Por outro lado, não use a investigação de integridade para verificar os serviços de baixa prioridade. Por exemplo, se um serviço de email ficar inativo, o aplicativo poderá mudar para um segundo provedor ou apenas enviar emails mais tarde. Essa não é uma prioridade alta o suficiente para fazer failover no aplicativo. Para obter mais informações, consulte o [Padrão de monitoramento de ponto de extremidade de integridade][health-endpoint-monitoring-pattern].
 
### <a name="sql-database"></a>Banco de dados SQL
Use a [Replicação Geográfica Ativa][sql-replication] para criar uma réplica secundária legível em uma região diferente. Você pode ter até quatro réplicas secundárias legíveis. Faça failover para um banco de dados secundário se o banco de dados primário falhar ou precisar ser deixado offline. A replicação geográfica ativa pode ser configurada para qualquer banco de dados em qualquer pool de banco de dados elástico.

### <a name="cosmos-db"></a>Cosmos DB
O Cosmos DB dá suporte à replicação geográfica entre regiões com vários mestres (várias regiões de gravação). Como alternativa, você pode designar uma região como a região gravável e outras como réplicas somente leitura. Se houver uma interrupção regional, será possível fazer failover selecionando outra região para ser a região de gravação. O SDK do cliente envia automaticamente solicitações de gravação para a região de gravação atual, portanto você não precisa atualizar a configuração do cliente após um failover. Para obter mais informações, confira [Distribuição de dados global com o Azure Cosmos DB][cosmosdb-geo].

> [!NOTE]
> Todas as réplicas de pertencem ao mesmo grupo de recursos.
>
>

### <a name="storage"></a>Armazenamento
Para o Armazenamento do Azure, use RA-GRS [armazenamento com redundância geográfica com acesso de leitura][ra-grs]. Com o armazenamento de RA-GRS, os dados são replicados para uma região secundária. Você tem acesso somente leitura aos dados na região secundária por meio de um ponto de extremidade separado. Se houver uma interrupção ou desastre regional, a equipe do Armazenamento do Azure pode decidir executar um failover geográfico para a região secundária. Nenhuma ação do cliente é necessária para este failover.

Para o Armazenamento de Filas, crie uma fila de backup na região secundária. Durante o failover, o aplicativo pode usar a fila de backup até que a região principal fique disponível novamente. Dessa forma, o aplicativo ainda poderá processar novas solicitações.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade


### <a name="traffic-manager"></a>Gerenciador de Tráfego

O Gerenciador de Tráfego fará failover automaticamente se a região principal ficar indisponível. Quando o Gerenciador de Tráfego faz failover, há um período em que os clientes não podem acessar o aplicativo. A duração é afetada pelos seguintes fatores:

* A investigação de integridade precisa detectar que o data center primário ficou inacessível.
* Os servidores DNS (Serviço de Nomes de Domínio) precisam atualizar os registros DNS armazenados em cache para o endereço IP, dependendo da TTL (vida útil) DNS. A TTL padrão é 300 segundos (5 minutos), mas você pode configurar esse valor ao criar o perfil do Gerenciador de Tráfego.

Para saber mais, consulte [Sobre o monitoramento do Gerenciador de Tráfego][tm-monitoring].

O Gerenciador de Tráfego é um possível ponto de falha no sistema. Se o serviço falhar, os clientes não poderão acessar seu aplicativo durante o tempo de inatividade. Examine o [SLA (Contrato de Nível de Serviço) do Gerenciador de Tráfego][tm-sla] e determine se usar apenas o Gerenciador de Tráfego atende aos seus requisitos de negócios para alta disponibilidade. Caso contrário, considere adicionar outra solução de gerenciamento de tráfego como um fallback. Se o serviço do Gerenciador de Tráfego do Azure falhar, altere os registros CNAME (nome canônico) no DNS para apontar para outro serviço de gerenciamento de tráfego. Esta etapa deve ser executada manualmente e seu aplicativo estará indisponível até que as alterações de DNS sejam propagadas.

### <a name="sql-database"></a>Banco de dados SQL
O RPO (objetivo de ponto de recuperação) e o ERT (tempo de recuperação estimado) para o Banco de Dados SQL estão documentados na [Visão geral de continuidade de negócios com o Banco de Dados SQL do Microsoft Azure][sql-rpo]. 

### <a name="storage"></a>Armazenamento
O armazenamento de RA-GRS fornece armazenamento durável, mas é importante entender o que pode ocorrer durante uma interrupção:

* Se ocorrer uma interrupção de armazenamento, haverá um período em que você não terá acesso de gravação aos dados. Você ainda poderá ler do ponto de extremidade secundário durante a interrupção.
* Se uma interrupção ou desastre regional afetar o local primário e os dados não puderem ser recuperados, a equipe do Armazenamento do Azure poderá optar por executar um failover geográfico para a região secundária.
* A replicação de dados para a região secundária é executada de forma assíncrona. Portanto, se um failover geográfico for executado, é possível que haja certa perda de dados se eles não podem ser recuperados da região primária.
* Falhas transitórias, como uma interrupção da rede, não disparam um failover de armazenamento. Projete seu aplicativo para ser resiliente a falhas transitórias. Possíveis mitigações:
  
  * Ler de uma região secundária.
  * Alternar temporariamente para outra conta de armazenamento para novas operações de gravação (por exemplo, para colocar mensagens na fila).
  * Copiar dados da região secundária para outra conta de armazenamento.
  * Fornecer funcionalidade reduzida até a conclusão do failback do sistema.

Para obter mais informações, consulte [O que fazer se ocorrer uma interrupção do Armazenamento do Azure][storage-outage].

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

### <a name="traffic-manager"></a>Gerenciador de Tráfego

Em caso de falha do Gerenciador de Tráfego, é recomendável executar um failback manual em vez de implementar um failback automático. Caso contrário, você pode criar uma situação em que o aplicativo fica alternando entre as regiões. Verifique se todos os subsistemas de aplicativo estão íntegros antes de realizar o failback.

Observe que o Gerenciador de Tráfego realiza failback automaticamente por padrão. Para evitar isso, diminua manualmente a prioridade da região primária após um evento de failover. Por exemplo, suponha que a região primária tem prioridade 1 e a secundária tem prioridade 2. Após um failover, defina a região primária para a prioridade 3, para impedir o failback automático. Quando você estiver pronto retornar para ela, atualize a prioridade para 1.

Os comandos a seguir atualizam a prioridade.

**PowerShell**

```bat
$endpoint = Get-AzureRmTrafficManagerEndpoint -Name <endpoint> -ProfileName <profile> -ResourceGroupName <resource-group> -Type AzureEndpoints
$endpoint.Priority = 3
Set-AzureRmTrafficManagerEndpoint -TrafficManagerEndpoint $endpoint
```

Para obter mais informações, consulte [Cmdlets do Gerenciador de Tráfego do Microsoft Azure][tm-ps].

**CLI do Azure**

```bat
az network traffic-manager endpoint update --resource-group <resource-group> --profile-name <profile> \
    --name <endpoint-name> --type azureEndpoints --priority 3
```    

### <a name="sql-database"></a>Banco de dados SQL
Se o banco de dados primário falhar, realize um failover manual para o banco de dados secundário. Consulte [Restaurar um Banco de Dados SQL do Azure ou fazer failover para um secundário][sql-failover]. O banco de dados secundário permanece somente leitura até você fazer o failover.


<!-- links -->

[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[azure-dns]: /azure/dns/dns-overview
[cosmosdb-geo]: /azure/cosmos-db/distribute-data-globally
[guidance-web-apps-scalability]: ./scalable-web-app.md
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[ra-grs]: /azure/storage/storage-redundancy#read-access-geo-redundant-storage
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-failover]: /azure/sql-database/sql-database-disaster-recovery
[sql-replication]: /azure/sql-database/sql-database-geo-replication-overview
[sql-rpo]: /azure/sql-database/sql-database-business-continuity#sql-database-features-that-you-can-use-to-provide-business-continuity
[storage-outage]: /azure/storage/storage-disaster-recovery-guidance
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-ps]: /powershell/module/azurerm.trafficmanager
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
