---
title: Executar VMs com balanceamento de carga no Azure para escalabilidade e disponibilidade
description: Como executar várias VMs do Windows no Azure para ter escalabilidade e disponibilidade.
author: telmosampaio
ms.date: 11/16/2017
pnp.series.title: Windows VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: d624ba74e3173b5f4218009de3ca6019f5f18143
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>Executar VMs com balanceamento de carga para ter escalabilidade e disponibilidade

Essa arquitetura de referência mostra um conjunto de práticas comprovadas para executar várias VMs (máquinas virtuais) do Windows em um conjunto de dimensionamento atrás de um balanceador de carga para melhorar a disponibilidade e a escalabilidade. Essa arquitetura pode ser usada para qualquer carga de trabalho sem estado, como um servidor Web e é uma fundação para a implantação de aplicativos de N camadas. [**Implante essa solução**.](#deploy-the-solution)

![[0]][0]

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura

Esta arquitetura baseia-se na [Arquitetura de referência de VM única ][single-vm]. Essas recomendações também se aplicam a essa arquitetura.

Nesta arquitetura, uma carga de trabalho é distribuída em várias instâncias de VM. Há um único endereço IP público e o tráfego da Internet é distribuído para as VMs usando um balanceador de carga. Essa arquitetura pode ser usada para um aplicativo de camada única, como um aplicativo Web sem estado.

A arquitetura tem os seguintes componentes:

* **Grupo de recursos.** [Grupos de recursos][resource-manager-overview] são utilizados para agrupar os recursos para que eles possam ser gerenciados pelo tempo de vida, o proprietário ou outros critérios.
* **Rede Virtual e sub-rede.** Cada VM do Azure é implantada em uma VNet que pode ser segmentada em várias sub-redes.
* **Azure Load Balancer**. O [balanceador de carga][load-balancer] distribui as solicitações de entrada da Internet para as instâncias de VM. 
* **Endereço IP público**. É necessário ter um endereço IP público para que o balanceador de carga possa receber o tráfego da Internet.
* **DNS do Azure**. [DNS do Azure][azure-dns] é um serviço de hospedagem para domínios DNS, que fornece resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure.
* **Conjunto de dimensionamento de VM**. Um [conjunto de dimensionamento de VM][vm-scaleset] é um conjunto de VMs idênticas usado para hospedar uma carga de trabalho. Os conjuntos de dimensionamento permitem que o número de VMs seja reduzido horizontalmente dentro ou fora manualmente, ou automaticamente com base em regras predefinidas.
* **Conjunto de disponibilidade**. O [conjunto de disponibilidade][availability-set] contém as VMs, tornando-as qualificadas para um [SLA (Contrato de Nível de Serviço)][vm-sla] mais elevado. Para que o SLA mais elevado seja aplicável, o conjunto de disponibilidade deve incluir um mínimo de duas VMs. Conjuntos de disponibilidade são conjuntos de dimensionamento implícitos. Se você criar VMs fora de um conjunto de dimensionamento, será necessário criar o conjunto de disponibilidade independentemente.
* **Discos gerenciados**. O Azure Managed Disks gerenciam os arquivos VHD (disco rígido virtual) para os discos de VM. 
* **Armazenamento**. Crie uma conta de Armazenamento do Azure para conter os logs de diagnóstico para as VMs.

## <a name="recommendations"></a>Recomendações

Seus requisitos podem não se alinhar completamente com a arquitetura descrita aqui. Use essas recomendações como ponto de partida. 

### <a name="availability-and-scalability-recommendations"></a>Recomendações de disponibilidade e escalabilidade

Uma opção para ter disponibilidade e escalabilidade é usar um [conjunto de dimensionamento de máquinas virtuais][vmss]. Conjuntos de dimensionamento de VM ajudam a implantar e gerenciar um conjunto de VMs idênticas. Conjuntos de dimensionamento dão suporte ao dimensionamento automático com base nas métricas de desempenho. À medida que a carga nas VMs aumenta, VMs adicionais são acrescentadas automaticamente ao balanceador de carga. Considere usar os conjuntos de dimensionamento se você precisar aumentar as VMs rapidamente ou se precisar de dimensionamento automático.

Por padrão, os conjuntos de dimensionamento usam “excesso de provisionamento”, o que significa que o conjunto de dimensionamento inicialmente provisiona mais VMs do que você pediu, excluindo posteriormente as VMs adicionais. Isso melhora a taxa de sucesso geral ao provisionar as VMs. Se você não estiver usando [discos gerenciados](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks), recomendamos não usar mais do que 20 VMs por conta de armazenamento com excesso com provisionamento habilitado e não mais do que 40 VMs com o excesso de provisionamento desabilitado.

Há duas maneiras básicas de configurar as VMs implantadas em um conjunto de dimensionamento:

- Use as extensões para configurar a VM depois que ela é provisionada. Com essa abordagem, novas instâncias de VM podem levar mais tempo para ser iniciadas do que uma VM sem extensões.

- Implante um [disco gerenciado](/azure/storage/storage-managed-disks-overview) com uma imagem de disco personalizada. Essa opção pode ser mais rápida de implantar. Porém, isso requer que você mantenha a imagem atualizada.

Para ver considerações adicionais, consulte [Considerações de design para conjuntos de dimensionamento][vmss-design].

> [!TIP]
> Ao usar qualquer solução de dimensionamento automático, teste-a com cargas de trabalho no nível de produção com bastante antecedência.

Se você não usar um conjunto de dimensionamento, considere usar pelo menos um conjunto de disponibilidade. Crie pelo menos duas VMs no conjunto de disponibilidade para dar suporte ao [SLA de disponibilidade para VMs do Azure][vm-sla]. O Azure Load Balancer também exige que as VMs com balanceamento de carga pertençam ao mesmo conjunto de disponibilidade.

Cada assinatura do Azure tem limites padrão em vigor, incluindo um número máximo de VMs por região. Você pode aumentar o limite enviando uma solicitação de suporte. Para saber mais, confira [Assinatura e limites de serviço, cotas e restrições do Azure][subscription-limits].

### <a name="network-recommendations"></a>Recomendações de rede

Implemente as VMs dentro da mesma sub-rede. Não exponha as VMs diretamente à Internet, concedendo, em vez disso, um endereço IP privado a cada VM. Os clientes se conectam usando um endereço IP público do balanceador de carga.

Se você precisar fazer logon nas VMs atrás do balanceador de carga, considere adicionar uma VM única como jumpbox (também chamada de host de bastião) com um endereço IP público no qual você pode fazer logon. Em seguida, faça logon nas VMs por trás do balanceador de carga por meio do jumpbox. Alternativamente, é possível configurar as regras de NAT (conversão de endereços de rede) de entrada do balanceador de carga. No entanto, ter um jumpbox é a melhor solução quando você hospeda cargas de trabalho de N camadas ou várias cargas de trabalho.

### <a name="load-balancer-recommendations"></a>Recomendações de balanceador de carga

Adicione todas as VMs no conjunto de disponibilidade ao pool de endereços de back-end do balanceador de carga.

Defina as regras do balanceador de carga para direcionar tráfego de rede para as VMs. Por exemplo, para permitir tráfego HTTP, crie uma regra que mapeie a porta 80 da configuração de front-end para a porta 80 no pool de endereços de back-end. Quando um cliente envia uma solicitação HTTP para a porta 80, o balanceador de carga seleciona um endereço IP de back-end usando um [algoritmo de hash][load-balancer-hashing] que inclui o endereço IP de origem. Dessa forma, as solicitações de cliente são distribuídas por todas as VMs.

Para rotear o tráfego para uma VM específica, use as regras NAT. Por exemplo, para habilitar RDP para as máquinas virtuais, crie uma regra NAT separada para cada VM. Cada regra deve mapear um número da porta diferente para a porta 3389, a porta padrão para RDP. Por exemplo, use a porta 50001 para “VM1”, 50002 para “VM2” e assim por diante. Atribua as regras NAT às NICs nas VMs.

### <a name="storage-account-recommendations"></a>Recomendações de conta de armazenamento

Também é recomendável usar os [discos gerenciado](/azure/storage/storage-managed-disks-overview) com o [armazenamento premium][premium]. Os discos gerenciados não exigem uma conta de armazenamento. Você simplesmente especifica o tamanho e o tipo de disco e é implantado como um recurso altamente disponível.

Se você estiver utilizando discos não gerenciados, crie contas de armazenamento do Azure separadas para cada VM para manter os VHDs (discos rígidos virtuais), para evitar atingir os [limites de IOPS][vm-disk-limits] (operações de entrada/saída por segundo) para contas de armazenamento.

Crie uma conta de armazenamento para logs de diagnóstico. Esta conta de armazenamento pode ser compartilhada por todas as VMs. Ela pode ser uma conta de armazenamento não gerenciada usando discos padrão.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

O conjunto de disponibilidade torna seu aplicativo mais resiliente a eventos de manutenção planejados e não planejados.

* A *manutenção planejada* ocorre quando a Microsoft atualiza a plataforma subjacente, podendo fazer com que as VMs sejam reiniciadas. O Azure garante que as VMs em um conjunto de disponibilidade não sejam reiniciadas ao mesmo tempo. Pelo menos uma delas é mantida em execução enquanto as outras estão reiniciando.
* A *manutenção não planejada* ocorre em caso de falha de hardware. O Azure garante que as VMs em um conjunto de disponibilidade sejam provisionadas em mais de um rack do servidor. Isso ajuda a reduzir o impacto de falhas de hardware, interrupções de rede, interrupções de energia e assim por diante.

Para obter mais informações, consulte [Gerenciar a disponibilidade de máquinas virtuais][availability-set]. O vídeo a seguir também fornece uma boa visão geral dos conjuntos de disponibilidade: [Como configurar um conjunto de disponibilidade para dimensionar VMs][availability-set-ch9].

> [!WARNING]
> Verifique se o conjunto de disponibilidade está configurado ao provisionar a VM. Atualmente, não há nenhuma maneira de adicionar uma VM do Resource Manager a um conjunto de disponibilidade depois que a VM é provisionada.

O balanceador de carga utiliza [investigações de integridade][health-probes] para monitorar a disponibilidade de instâncias de VM. Se uma investigação não puder acessar uma instância dentro de um período de tempo limite, o balanceador de carga parará de enviar tráfego para essa VM. Contudo, o balanceador de carga continuará a investigar e, se a VM ficar disponível novamente, o balanceador de carga reiniciará o envio de tráfego para ela.

Aqui estão algumas recomendações sobre as investigações de integridade do balanceador de carga:

* As investigações podem testar HTTP ou TCP. Se suas VMs são executadas em um servidor HTTP, crie uma investigação HTTP. Caso contrário, crie uma investigação TCP.
* Para uma investigação HTTP, especifique o caminho para um ponto de extremidade HTTP. A investigação verifica uma resposta HTTP 200 para esse caminho. Ele pode ser o caminho raiz (“/”) ou um ponto de extremidade de monitoramento de integridade que implementa lógica personalizada para verificar a integridade do aplicativo. O ponto de extremidade deve permitir solicitações HTTP anônimas.
* A investigação é enviada de um endereço IP [conhecido][health-probe-ip], 168.63.129.16. Verifique se o tráfego de ou para esse endereço IP não é bloqueado por quaisquer políticas de firewall ou regras de NSG (Grupo de Segurança de Rede).
* Use os [logs de investigação de integridade][health-probe-log] para exibir o status das investigações de integridade. Habilite o registro em log no Portal do Azure para cada balanceador de carga. Os logs são gravados no Armazenamento de Blobs do Azure. Os logs de mostram como muitas VMs no back-end não estão recebendo o tráfego de rede devido a respostas de investigação com falha.

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Com várias VMs, é importante automatizar os processos para que eles sejam confiáveis e reproduzíveis. Você pode usar a [Automação do Azure][azure-automation] para automatizar a implantação, aplicação de patches do sistema operacional e outras tarefas. A [Automação do Azure][azure-automation] é um serviço de automação baseado no PowerShell que pode ser usado para isso. Scripts de automação de exemplo estão disponíveis a partir da [Galeria de runbook][runbook-gallery].

## <a name="security-considerations"></a>Considerações de segurança

Redes virtuais são um limite de isolamento de tráfego no Azure. As VMs em uma VNet não podem se comunicar diretamente com VMs em uma VNet diferente. As VMs na mesma VNet podem se comunicar, a menos que você crie NSGs ([Grupos de Segurança de Rede][nsg]) para restringir o tráfego. Para obter mais informações, consulte [Serviços em nuvem da Microsoft e segurança de rede][network-security].

Para tráfego de entrada da Internet, as regras do balanceador de carga definem qual tráfego pode alcançar o back-end. No entanto, as regras do balanceador de carga não dão suporte a listas de IP de confiança, por isso se você desejar adicionar determinados endereços IP públicos a uma lista de confiança, acrescente um NSG à sub-rede.

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para essa arquitetura está disponível no [GitHub][github-folder]. Ela implanta o seguinte:

  * Uma rede virtual com uma única sub-rede chamada **Web** que contém as VMs.
  * Um conjunto de dimensionamento de VM que contém VMs que executam a versão mais recente do Windows Server 2016 Datacenter Edition. O dimensionamento automático está habilitado.
  * Um balanceador de carga localizado na frente do conjunto de dimensionamento da VM.
  * Um NSG com duas regras de entrada que permite tráfego HTTP para o conjunto de dimensionamento da VM.

### <a name="prerequisites"></a>pré-requisitos

Antes de implantar a arquitetura de referência para sua própria assinatura, você deve executar as etapas a seguir.

1. Clone, crie um fork ou baixe o arquivo zip das [arquiteturas de referência][ref-arch-repo] no repositório GitHub.

2. Verifique se a CLI do Azure 2.0 está instalada no computador. Para obter instruções de instalação da CLI, consulte [Instalar a CLI 2.0 do Azure][azure-cli-2].

3. Instale os pacote npm dos [Blocos de construção do Azure][azbb].

4. Em um prompt de comando, bash prompt ou prompt do PowerShell, faça logon na sua conta do Azure usando um dos comandos abaixo e siga os prompts.

   ```bash
   az login
   ```

### <a name="deploy-the-solution-using-azbb"></a>Implantar a solução usando azbb

Para implantar a carga de trabalho de VM única de exemplo, siga estas etapas:

1. Navegue até a pasta `virtual-machines\multi-vm\parameters\windows` do repositório que você baixou na etapa de pré-requisitos acima.

2. Abra o arquivo `multi-vm-v2.json` e insira um nome de usuário e senha entre aspas, conforme mostrado abaixo e salve o arquivo.

   ```bash
   "adminUsername": "",
   "adminPassword": "",
   ```

3. Execute `azbb` para implantar as VMs conforme mostrado abaixo.

   ```bash
   azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
   ```

Para obter mais informações sobre como implantar essa arquitetura de referência de exemplo, visite nosso [Repositório GitHub][git].

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability-set-ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-automation]: /azure/automation/automation-intro
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[naming-conventions]: ../../best-practices/naming-conventions.md
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[premium]: /azure/storage/common/storage-premium-storage
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[runbook-gallery]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[single-vm]: single-vm.md
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vm-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[0]: ./images/multi-vm-diagram.png "Arquitetura de uma solução de várias VMs no Azure composta por um conjunto de disponibilidade com duas VMs e um balanceador de carga"
