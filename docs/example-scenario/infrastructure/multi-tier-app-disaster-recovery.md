---
title: Aplicativo Web de várias camadas criado para HA/DR
titleSuffix: Azure Example Scenarios
description: Crie um aplicativo Web de várias camadas projetado para alta disponibilidade e recuperação de desastres no Azure usando máquinas virtuais do Azure, conjuntos de disponibilidade, zonas de disponibilidade, Azure Site Recovery e Gerenciador de Tráfego do Azure.
author: sujayt
ms.date: 11/16/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: product-team
ms.openlocfilehash: 377c1663b8cd81f7788a1f2ce82b562a9695f3b0
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483971"
---
# <a name="multitier-web-application-built-for-high-availability-and-disaster-recovery-on-azure"></a>Aplicativo Web de várias camadas criado para alta disponibilidade e recuperação de desastres no Azure

Este cenário de exemplo é aplicável a qualquer setor que precise implantar aplicativos multicamadas resilientes criados para alta disponibilidade e recuperação de desastre. Nesse cenário, o aplicativo é formado por três camadas.

- Camada da Web: A camada superior, incluindo a interface do usuário. Essa camada analisa as interações do usuário e passa as ações para a próxima camada para processamento.
- Camada de negócios: processa as interações do usuário e toma decisões lógicas sobre as próximas etapas. Essa camada se conecta à camada da Web e à camada de dados.
- Camada de dados: Armazena os dados do aplicativo. Um banco de dados, o armazenamento de objetos ou o armazenamento de arquivos normalmente é usado.

Cenários de aplicativo comuns incluem qualquer aplicativo de missão crítica em execução no Windows ou Linux. Isso pode ser um aplicativo pronto para uso, como SAP e SharePoint, ou um aplicativo de linha de negócios personalizado.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Outros casos de uso relevantes incluem:

- Implantar aplicativos altamente resilientes, como SAP e SharePoint
- Projetar um plano de continuidade de negócios e recuperação de desastres para aplicativos de linha de negócios
- Configurar a recuperação de desastres e executar treinamentos relacionados para fins de conformidade

## <a name="architecture"></a>Arquitetura

Este cenário apresenta um aplicativo de várias camadas que usa o ASP.NET e o Microsoft SQL Server. Em [regiões do Azure que oferecem suporte a zonas de disponibilidade](/azure/availability-zones/az-overview#regions-that-support-availability-zones), é possível implantar suas máquinas virtuais (VMs) em uma região de origem nas zonas de disponibilidade e replicar as VMs na região de destino usadas para recuperação de desastres. Em regiões do Azure que não oferecem suporte a zonas de disponibilidade, é possível implantar suas máquinas virtuais (VMs) em um conjunto de disponibilidade e replicar as VMs na região de destino.

![Visão geral da arquitetura de um aplicativo Web multicamada altamente resiliente][architecture]

- Distribua as VMs em cada camada em duas zonas de disponibilidade em regiões que oferecem suporte às zonas. Em outras regiões, implante as VMs em cada camada em um conjunto de disponibilidade.
- A camada de banco de dados pode ser configurada para usar grupos de disponibilidade Always On. Com essa configuração do SQL Server, um banco de dados primário em um cluster é configurado com até oito bancos de dados secundários. Se ocorrer um problema com o banco de dados primário, o cluster faz o failover para um banco de dados secundários, permitindo que o aplicativo continue disponível. Para saber mais informações, confira [Visão geral dos grupos de disponibilidade Always On para SQL Server][docs-sql-always-on].
- Para cenários de recuperação de desastres, você pode configurar a replicação nativa assíncrona do SQL Always On para a região de destino usada na recuperação de desastre. Você também pode configurar a replicação do Azure Site Recovery para a região de destino se a taxa de alteração de dados estiver dentro dos limites compatíveis com o Azure Site Recovery.
- Os usuários acessam a camada da Web do ASP.NET front-end através do ponto de extremidade do gerenciador de tráfego.
- O gerenciador de tráfego redireciona o tráfego para o ponto de extremidade do IP público primário na região de origem primária.
- O IP público redireciona a chamada para uma das instâncias de VM de camada da Web através de um balanceador de carga público. Todas as instâncias de VM da camada da Web estão em uma sub-rede.
- A partir da VM da camada da Web, cada chamada é roteada para uma das instâncias de VM na camada de negócios através de um balanceador de carga interno para processamento. Todas as VMs da camada de negócios estão em uma sub-rede separada.
- A operação é processada na camada de negócios e o aplicativo ASP.NET conecta-se ao cluster do Microsoft SQL Server em uma camada de back-end por meio de um balanceador de carga interno do Azure. Essas instâncias do SQL Server de back-end estão em uma sub-rede separada.
- O ponto de extremidade secundário do gerenciador de tráfego é configurado como o IP público na região de destino usada na recuperação de desastre.
- No caso de uma interrupção da região primária, invoque o failover do Azure Site Recovery e o aplicativo ficará ativo na região de destino.
- O ponto de extremidade do gerenciador de tráfego redireciona automaticamente o tráfego do cliente para o IP público na região de destino.

### <a name="components"></a>Componentes

- Os [conjuntos de disponibilidade][docs-availability-sets] garantem que as VMs implantadas no Azure sejam distribuídas entre vários nós de hardware isolados em um cluster. Se ocorrer uma falha de hardware ou software no Azure, somente um subconjunto de suas VMs será afetado e toda sua solução permanecerá disponível e operacional.
- As [zonas de disponibilidade][docs-availability-zones] ajudam a proteger os aplicativos e os dados contra falhas no datacenter. As zonas de disponibilidade são locais físicos separados em uma região do Azure. Cada zona é composta por um ou mais datacenters equipados com energia, resfriamento e rede independentes.
- O [Azure Site Recovery (ASR)][docs-azure-site-recovery] permite replicar máquinas virtuais para outra região do Azure para necessidades de continuidade dos negócios e de recuperação de desastres. Você pode realizar análises periódicas de recuperação de desastres para garantir que atende às necessidades de conformidade. A máquina virtual será replicada com as configurações especificadas para a região selecionada para que você possa recuperar seus aplicativos em caso de interrupções na região de origem.
- O [Gerenciador de Tráfego do Azure][docs-traffic-manager] é um balanceador de carga de tráfego baseado em DNS que distribui o tráfego de maneira ideal para serviços em todas as regiões globais do Azure, fornecendo alta disponibilidade e capacidade de resposta.
- O [Azure Load Balancer][docs-load-balancer] distribui o tráfego de entrada de acordo com as regras e investigações de integridade. O balanceador de carga fornece baixa latência e alta taxa de transferência e pode ser escalado verticalmente em milhões de fluxos para aplicativos TCP e UDP. Um balanceador de carga público é usado neste cenário para distribuir o tráfego de clientes para a camada da Web. Um balanceador de carga interno é usado neste cenário para distribuir o tráfego da camada de negócios para o cluster do SQL Server de back-end.

### <a name="alternatives"></a>Alternativas

- O Windows pode ser substituído por outros sistemas operacionais já que nada na infraestrutura depende do sistema operacional.
- [SQL Server para Linux][docs-sql-server-linux] pode substituir o armazenamento de dados de back-end.
- O banco de dados pode ser substituído por qualquer aplicativo de banco de dados padrão disponível.

## <a name="other-considerations"></a>Outras considerações

### <a name="scalability"></a>Escalabilidade

Você pode adicionar ou remover as VMs em cada camada com base em seus requisitos de escala. Como esse cenário usa balanceadores de carga, você pode adicionar mais VMs a uma camada sem afetar o tempo de atividade do aplicativo.

Para outros tópicos de escalabilidade, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.

### <a name="security"></a>Segurança

Todo o tráfego de rede virtual para a camada de aplicativo front-end é protegido por grupos de segurança de rede. As regras limitam o fluxo de tráfego para que somente as instâncias de VM da camada de aplicativo front-end possam acessar a camada de banco de dados de back-end. Nenhum tráfego de Internet de saída é permitido da camada de banco de dados ou de negócios. Para reduzir a superfície de ataque, nenhuma porta de gerenciamento remoto direto fica aberta. Para saber mais, confira [Grupos de segurança de rede do Azure][docs-nsg].

Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].

## <a name="pricing"></a>Preços

A configuração da recuperação de desastre para VMs do Azure que usam o Azure Site Recovery incorrerá nas seguintes cobranças de maneira contínua.

- O custo do licenciamento da Recuperação de Site do Azure por VM.
- Os custos de saída de rede para replicar alterações de dados dos discos da VM de origem para outra região do Azure. O Azure Site Recovery usa a compactação interna para reduzir os requisitos de transferência de dados em cerca de 50%.
- Custos de armazenamento no site de recuperação. Isso geralmente equivale ao armazenamento da região de origem mais qualquer armazenamento adicional necessário para manter os pontos de recuperação como instantâneos para recuperação.

Fornecemos uma [calculadora de custos de exemplo][calculator] para configurar a recuperação de desastre para um aplicativo de três camadas usando seis máquinas virtuais. Todos os serviços são pré-configurados na calculadora de custos. Para ver como o preço seria alterado para seu uso específico, altere as variáveis apropriadas para estimar o custo.

<!-- links -->
[architecture]: ./media/arhitecture-disaster-recovery-multi-tier-app.png
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[availability]: ../../checklist/availability.md
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[docs-availability-zones]: /azure/availability-zones/az-overview
[docs-load-balancer]: /azure/load-balancer/load-balancer-overview
[docs-nsg]: /azure/virtual-network/security-overview
[docs-vmss]: /azure/virtual-machine-scale-sets/overview
[docs-sql-always-on]: /sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server
[docs-vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[docs-vnet]: /azure/virtual-network/virtual-networks-overview
[docs-sql-server-linux]: /sql/linux/sql-server-linux-overview?view=sql-server-linux-2017
[docs-traffic-manager]: /azure/traffic-manager/
[docs-azure-site-recovery]: /azure/site-recovery/azure-to-azure-quickstart/
[docs-availability-sets]: /azure/virtual-machines/windows/manage-availability/
[calculator]: https://azure.com/e/6835332265044d6d931d68c917979e6d/
