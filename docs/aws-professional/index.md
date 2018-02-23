---
title: Azure para profissionais do AWS
description: "Entenda os conceitos básicos de contas, plataforma e serviços do Microsoft Azure. Conheça também as principais semelhanças e diferenças entre as plataformas AWS e Azure. Aproveite sua experiência com o AWS no Azure."
keywords: "Especialistas em AWS, comparação com o Azure, comparação com o AWS, diferença entre azure e aws, azure e aws"
author: lbrader
ms.date: 03/24/2017
pnp.series.title: Azure for AWS Professionals
ms.openlocfilehash: b8698675efa42bb3fae73cefe7b078942549b412
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/23/2018
---
# <a name="azure-for-aws-professionals"></a>Azure para profissionais do AWS

Este artigo ajuda os especialistas em AWS (Amazon Web Services) a compreender os conceitos básicos de contas, plataforma e serviços do Microsoft Azure. Aborda também as principais semelhanças e diferenças entre as plataformas AWS e Azure.

O que você aprenderá:

* Como contas e recursos são organizados no Azure.
* Como as soluções disponíveis são estruturadas no Azure.
* As principais diferenças entre os serviços Azure e AWS.

O Azure e o AWS construíram seus recursos independentemente, ao longo do tempo, e têm diferenças importantes de design e implementação.

## <a name="overview"></a>Visão geral

Tal como o AWS, o Microsoft Azure foi criado para prestar um conjunto central de serviços de computação, armazenamento, banco de dados e rede. Em muitos casos, ambas as plataformas têm uma equivalência básica entre os produtos e serviços que oferecem. O AWS e o Azure permitem criar soluções altamente disponíveis com base em hosts do Windows ou do Linux. Portanto, se você estiver acostumado ao desenvolvimento com tecnologia OSS e Linux, as duas plataformas são eficazes.

Embora elas tenham funcionalidades semelhantes, os recursos que as fornecem são, em geral, organizados de forma diferente. As relações um-para-um entre os serviços necessários para compilar uma solução nem sempre são claras. Também há casos em que um serviço específico pode ser oferecido em uma plataforma, mas não na outra. Confira os [gráficos comparativos de serviços do Azure e do AWS](services.md).

## <a name="accounts-and-subscriptions"></a>Contas e assinaturas

Os serviços do Azure podem ser adquiridos em várias opções de preços, dependendo do tamanho e das necessidades da sua organização. Confira os detalhes na página de [visão geral de preços](https://azure.microsoft.com/pricing/).

As [assinaturas do Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/) são um agrupamento de recursos com um proprietário atribuído, responsável por gerenciar cobranças e permissões. Diferentemente do AWS, onde todos os recursos criados sob a conta AWS são vinculados a essa conta, as assinaturas existem independentemente das contas do proprietário e podem ser reatribuídas a novos proprietários conforme necessário.

![Comparação entre a estrutura e a propriedade de contas do AWS e assinaturas do Azure](./images/azure-aws-account-compare.png "Comparação entre a estrutura e a propriedade de contas do AWS e assinaturas do Azure")
<br/>*Comparação entre a estrutura e a propriedade de contas do AWS e assinaturas do Azure*
<br/><br/>

As assinaturas são atribuídas a três tipos de contas de administrador:

-   **Administrador da Conta** - É o proprietário da assinatura, e também a conta responsável pelo pagamento dos recursos usados na assinatura. O administrador da conta só pode ser alterado transferindo-se a propriedade da assinatura.

-   **Administrador de Serviços** - Esta conta tem direitos para criar e gerenciar recursos na assinatura, mas não responde pelas questões de cobrança. Por padrão, o administrador da conta e o administrador de serviços são atribuídos à mesma conta. O administrador da conta pode atribuir um usuário separado à conta de administrador de serviços para gerenciar os aspectos técnicos e operacionais de uma assinatura. Há somente um administrador de serviços por assinatura.

-   **Coadministrador** - Pode haver várias contas de coadministrador atribuídas a uma assinatura. Os coadministradores não podem mudar o administrador de serviços, mas, quanto ao mais, têm controle total sobre os recursos de assinatura e os usuários.

Abaixo do nível de assinatura, as permissões de funções de usuário e individuais também podem ser atribuídas a recursos específicos, tal como ocorre com a concessão de permissões a usuários e grupos do IAM no AWS. No Azure, todas as contas de usuário são associadas a uma Conta da Microsoft ou Conta Organizacional (gerenciada por meio de um Active Directory do Azure).

Tal como as contas do AWS, as assinaturas têm cotas e limites de serviço padrão. Para obter uma lista completa desses limites, confira [Limites, cotas e restrições em assinaturas e serviços do Azure](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/).
Esses limites podem ser aumentados até o máximo [preenchendo-se uma solicitação de suporte no portal de gerenciamento](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/).

### <a name="see-also"></a>Consulte também

-   [Como adicionar ou alterar funções de administrador do Azure](https://azure.microsoft.com/documentation/articles/billing-add-change-azure-subscription-administrator/)

-   [Como baixar sua fatura de cobrança e dados de uso diário do Azure](https://azure.microsoft.com/documentation/articles/billing-download-azure-invoice-daily-usage-date/)

## <a name="resource-management"></a>Gerenciamento de recursos

O termo "recurso" no Azure é usado tal como no AWS, significando qualquer instância de computação, objeto de armazenamento, dispositivo de rede ou outra entidade que você pode criar ou configurar na plataforma.

Os recursos do Azure são implantados e gerenciados com um destes dois modelos: [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview) ou o antigo [modelo de implantação clássico](/azure/azure-resource-manager/resource-manager-deployment-model) do Azure.
Todos os recursos novos são criados com o modelo Resource Manager.

### <a name="resource-groups"></a>Grupos de recursos

O Azure e o AWS têm entidades chamadas "grupos de recursos", que organizam recursos como VMs, armazenamento e dispositivos de rede virtuais. No entanto, os [grupos de recursos do Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/) não se comparam diretamente aos do AWS.

O AWS permite que um recurso seja marcado em vários grupos de recursos; já no Azure, ele está sempre associado um grupo de recursos. Um recurso criado em um grupo de recursos pode ser movido para outro, mas só pode estar em um por vez. Os grupos de recursos são o agrupamento fundamental usado pelo Azure Resource Manager.

Os recursos também podem ser organizados usando-se [marcas](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/).
Marcas são pares chave-valor que permitem agrupar recursos na assinatura, independentemente da associação ao grupo de recursos.

### <a name="management-interfaces"></a>Interfaces de gerenciamento

O Azure oferece várias maneiras de gerenciar seus recursos:

-   [Interface da Web](https://azure.microsoft.com/documentation/articles/resource-group-portal/).
    Como o painel do AWS, o portal do Azure fornece uma interface totalmente baseada na Web para o gerenciamento dos recursos do Azure.

-   [API REST](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/).
    A REST API do Azure Resource Manager fornece acesso programático à maioria dos recursos disponíveis no portal do Azure.

-   [Linha de Comando](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/).
    A ferramenta Azure CLI 2.0 fornece uma interface de linha de comando capaz de criar e gerenciar recursos do Azure. A CLI do Azure está disponível para [Windows, Linux e Mac OS](https://aka.ms/azurecli2).

-   [PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/).
    Os módulos do Azure para PowerShell permitem executar tarefas de gerenciamento automatizado usando um script. O PowerShell está disponível para [Windows, Linux e Mac OS](https://github.com/PowerShell/PowerShell).

-   [Modelos](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/).
    Os modelos do Azure Resource Manager fornecem recursos JSON de gerenciamento baseado em modelo semelhantes aos do serviço CloudFormation do AWS.

Em cada uma dessas interfaces, o grupo de recursos é fundamental para o modo como os recursos do Azure são criados, implantados ou modificados. Isso é semelhante à função da "pilha" no agrupamento de recursos do AWS durante as implantações de CloudFormation.

A sintaxe e a estrutura dessas interfaces diferem das de seus equivalentes do AWS, mas os recursos são comparáveis. Além disso, muitas ferramentas de gerenciamento de terceiros usadas no AWS, como [Terraform do Hashicorp](https://www.terraform.io/docs/providers/azurerm/) e [Netflix Spinnaker](http://www.spinnaker.io/), também estão disponíveis no Azure.

### <a name="see-also"></a>Consulte também

-   [Diretrizes de grupos de recursos do Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)

## <a name="regions-and-zones-high-availability"></a>Regiões e zonas (alta disponibilidade)

Falhas podem variar quanto ao escopo do seu impacto. Algumas falhas de hardware, como um disco com falha, podem afetar um único computador host. Um comutador de rede com falha pode afetar um rack de servidor inteiro. Menos comuns são falhas que interrompem um data center inteiro, tais como uma perda de energia em um data center. Raramente, toda uma região pode se tornar não disponível.

Uma das principais maneiras de se tornar um aplicativo resiliente é por meio de redundância. Mas você precisa planejar essa redundância ao projetar o aplicativo. Além disso, o nível de redundância necessário depende de suas necessidades de negócios &mdash; nem todo aplicativo precisa de redundância entre regiões para se proteger contra uma interrupção regional. Em geral, há um equilíbrio entre mais redundância e confiabilidade em relação a maiores custo e complexidade.  

No AWS, uma região é dividida em duas ou mais Zonas de Disponibilidade. Uma Zona de Disponibilidade corresponde a um datacenter fisicamente isolado na região geográfica. O Azure tem vários recursos para tornar um aplicativo redundante em cada nível de falha, incluindo **conjuntos de disponibilidade**, **zonas de disponibilidade** e **zonas emparelhadas**. 

![](../resiliency/images/redundancy.svg)

A tabela a seguir resume cada opção.

| &nbsp; | Conjunto de disponibilidade | Zona de disponibilidade | Região emparelhada |
|--------|------------------|-------------------|---------------|
| Escopo da falha | Rack | Datacenter | Região |
| Roteamento de solicitação | Balanceador de carga | Load Balancer entre zonas | Gerenciador de Tráfego |
| Latência da rede | Muito baixa | Baixo | Média a alta |
| Rede Virtual  | VNET | VNET | Emparelhamento VNET entre regiões (versão prévia) |

### <a name="availability-sets"></a>Conjuntos de disponibilidade 

Para se proteger contra falhas de hardware localizadas, como falha em um comutador de rede ou de disco, implante duas ou mais VMs em um conjunto de disponibilidade. Um conjunto de disponibilidade consiste em dois ou mais *domínios de falha* que compartilham um comutador de rede e uma fonte de energia comuns. Máquinas virtuais em um conjunto de disponibilidade são distribuídas entre os domínios de falha, portanto, se uma falha de hardware afeta um domínio de falha, o tráfego de rede ainda poderá ser roteado para as VMs nos outros domínios de falha. Para obter mais informações sobre conjuntos de disponibilidade, consulte [Gerenciar a disponibilidade das máquinas virtuais do Windows no Azure](/azure/virtual-machines/windows/manage-availability).

Quando instâncias de VM são adicionadas a conjuntos de disponibilidade, elas também recebem um [domínio de atualização](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/). Um domínio de atualização é um grupo de VMs definidas para eventos de manutenção planejada ao mesmo tempo. A distribuição de VMs em vários domínios de atualização garante que os eventos de atualização planejada e de aplicação de patch afetem apenas um subconjunto dessas VMs em determinado momento.

Os conjuntos de disponibilidade devem ser organizados pela função da instância no seu aplicativo, para garantir que uma instância de cada função esteja operacional. Por exemplo, em um aplicativo web de três camadas, crie conjuntos de disponibilidade separados para as camadas de front-end, aplicativos e camadas de dados.

![Conjuntos de disponibilidade do Azure para cada função de aplicativo](./images/three-tier-example.png "Conjuntos de disponibilidade para cada função de aplicativo")

### <a name="availability-zones-preview"></a>Zonas de disponibilidade (versão prévia)

Uma [Zona de Disponibilidade](/azure/availability-zones/az-overview) é uma zona fisicamente separada em uma região do Azure. Cada zona de disponibilidade tem uma rede, resfriamento e fonte de energia distintos. A implantação de VMs em zonas de disponibilidade ajuda a proteger um aplicativo contra falhas em todo o datacenter. 

### <a name="paired-regions"></a>Regiões emparelhadas

Para proteger um aplicativo contra uma interrupção regional, você pode implantar o aplicativo em várias regiões, usando o [Gerenciador de Tráfego do Azure][traffic-manager] para distribuir o tráfego de Internet para as diferentes regiões. Cada região do Azure é emparelhada com outra. Juntas, elas formam um [par regional][paired-regions]. Com a exceção do Sul do Brasil, pares regionais estão localizados na mesma região geográfica para atender aos requisitos de residência de dados para fins de jurisdição de imposição fiscal e legal.

Diferentemente das Zonas de Disponibilidade, que, embora sejam datacenters fisicamente separados, podem estar em áreas geográficas relativamente próximas, as regiões emparelhadas ficam, em geral, distantes no mínimo 300 milhas (cerca de 480 km). Isso visa a garantir que desastres em maior escala afetem somente uma das regiões do par. Os pares vizinhos podem ser definidos para sincronizar dados de serviços de armazenamento e de bancos de dados, e são configurados para que as atualizações de plataforma sejam distribuídas em apenas uma região do par de cada vez.

O backup do [armazenamento com redundância geográfica](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage) do Azure é feito automaticamente na região emparelhada apropriada. Em todos os outros recursos, criar uma solução totalmente redundante usando regiões emparelhadas significa gerar uma cópia completa da sua solução em ambas as regiões.


### <a name="see-also"></a>Consulte também

-   [Regiões e disponibilidade para máquinas virtuais no Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-regions-and-availability/)

-   [Alta disponibilidade para os aplicativos do Azure](../resiliency/high-availability-azure-applications.md)

-   [Recuperação de desastre para aplicativos do Azure](../resiliency/disaster-recovery-azure-applications.md)

-   [Manutenção planejada para máquinas virtuais Linux no Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-planned-maintenance/)

## <a name="services"></a>Serviços

Confira a [matriz de comparação completa de serviços AWS e Azure](https://aka.ms/azure4aws-services) para obter uma lista completa de como todos os serviços são mapeados entre as plataformas.

Nem todos os produtos e serviços do Azure estão disponíveis em todas as regiões. Confira a página [Produtos por região](https://azure.microsoft.com/regions/services/) para obter detalhes. Você pode encontrar as garantias de tempo de atividade e políticas de crédito de tempo de inatividade para cada produto ou serviço do Azure na página [Contratos de nível de serviço](https://azure.microsoft.com/support/legal/sla/).

As seções a seguir fornecem uma breve explicação das diferenças entre os serviços e recursos comumente usados nas plataformas AWS e Azure.

### <a name="compute-services"></a>Serviços de computação

#### <a name="ec2-instances-and-azure-virtual-machines"></a>Instâncias do EC2 e máquinas virtuais do Azure

Embora os tipos de instâncias do AWS e os tamanhos de máquinas virtuais do Azure se dividam de maneira semelhante, há diferenças nos recursos de RAM, CPU e armazenamento.

-   [Tipos de instâncias do Amazon EC2](https://aws.amazon.com/ec2/instance-types/)

-   [Tamanhos das máquinas virtuais no Azure (Windows)](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/)

-   [Tamanhos das máquinas virtuais no Azure (Linux)](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-sizes/)

Diferentemente da cobrança por segundo do AWS, as VMs sob demanda do Azure são cobradas por minuto.

O Azure não tem equivalentes das instâncias especiais ou dos hosts dedicados do EC2.

#### <a name="ebs-and-azure-storage-for-vm-disks"></a>EBS e Armazenamento do Azure para discos de VM

O armazenamento de dados durável para VMs do Azure é fornecido por [discos de dados](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/) que residem no armazenamento de blobs. Isso é semelhante à forma como as instâncias do EC2 armazenam volumes de disco em Elastic Block Store (EBS). O [Armazenamento temporário do Azure](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/) também fornece às VMs o mesmo armazenamento temporário de leitura-gravação com baixa latência do Instance Storage do EC2 (também chamado de armazenamento efêmero).

A E/S em disco de desempenho superior é suportada com o [armazenamento premium do Azure](https://docs.microsoft.com/azure/storage/storage-premium-storage).
Isso é semelhante às opções de armazenamento provisionado IOPS fornecidas pelo AWS.

#### <a name="lambda-azure-functions-azure-web-jobs-and-azure-logic-apps"></a>Lambda, Azure Functions, Azure Web-Jobs e Aplicativos Lógicos do Azure

O [Azure Functions](https://azure.microsoft.com/services/functions/) é o equivalente principal do Lambda do AWS no fornecimento de código sem servidor sob demanda.
No entanto, a funcionalidade do Lambda também se sobrepõe a outros serviços do Azure:

-   [WebJobs](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/) - Permite criar tarefas em segundo plano agendadas ou de execução contínua.

-   [Aplicativos Lógicos](https://azure.microsoft.com/services/logic-apps/) - Fornece serviços de comunicações, integração e gerenciamento de regras de negócios.

#### <a name="autoscaling-azure-vm-scaling-and-azure-app-service-autoscale"></a>Dimensionamento automático, escala de VM do Azure e Dimensionamento Automático de Serviço de Aplicativo do Azure

O dimensionamento automático no Azure é tratado por dois serviços:

-   [Conjuntos de escala de VM](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/) - Permitem implantar e gerenciar um conjunto idêntico de VMs. O número de instâncias pode ser dimensionado automaticamente com base nas necessidades de desempenho.

-   [Dimensionamento Automático de Serviço de Aplicativo](https://azure.microsoft.com/documentation/articles/web-sites-scale/) - Permite o dimensionamento automático de soluções do Serviço de Aplicativo do Azure.


#### <a name="container-service"></a>Serviço de Contêiner
O [Serviço de Contêiner do Azure](https://docs.microsoft.com/azure/container-service/container-service-intro) oferece suporte a contêineres do Docker gerenciados por meio de Docker Swarm, Kubernetes ou DC/OS.

#### <a name="other-compute-services"></a>Outros serviços de computação 


O Azure oferece vários serviços de computação sem equivalente direto no AWS:

-   [Lote do Azure](https://azure.microsoft.com/documentation/articles/batch-technical-overview/) - Permite gerenciar trabalhos de computação intensiva em um conjunto dimensionável de máquinas virtuais.

-   [Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/) - Plataforma para desenvolvimento e hospedagem de soluções escalonáveis de [microsserviços](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/).

#### <a name="see-also"></a>Consulte também

-   [Criar uma VM do Linux no Azure usando o Portal](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-quick-create-portal/)

-   [Arquitetura de referência no Azure: processamento de uma VM Linux no Azure](https://azure.microsoft.com/documentation/articles/guidance-compute-single-vm-linux/)

-   [Get started with Node.js web apps in Azure App Service (Introdução aos aplicativos Web do Node.js no Serviço de Aplicativo do Azure)](https://azure.microsoft.com/documentation/articles/app-service-web-nodejs-get-started/)

-   [Arquitetura de referência do Azure: aplicativo Web básico](https://azure.microsoft.com/documentation/articles/guidance-web-apps-basic/)

-   [Criar sua primeira Função do Azure](https://azure.microsoft.com/documentation/articles/functions-create-first-azure-function/)

### <a name="storage"></a>Armazenamento

#### <a name="s3ebsefs-and-azure-storage"></a>S3/EBS/EFS e Armazenamento do Azure

Na plataforma AWS, o armazenamento em nuvem divide-se principalmente em três serviços:

-   **Simple Storage Service (S3)** - Armazenamento básico de objetos. Disponibiliza os dados por meio de uma API acessível da Internet.

-   **Elastic Block Storage (EBS)** - Armazenamento em nível de bloco, destinado ao acesso por uma única VM.

-   **Elastic File System (EFS)** - Armazenamento de arquivos para uso compartilhado, comportando milhares de instâncias de EC2.

No Armazenamento do Azure, as [contas de armazenamento](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) associadas à assinatura permitem criar e gerenciar os serviços de armazenamento a seguir:

-   [Armazenamento de blobs](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) - Armazena qualquer tipo de texto ou de dados binários, como documentos, arquivos de mídia ou instaladores de aplicativos. Você pode definir o armazenamento de Blobs para acesso privado ou para compartilhar conteúdo público na Internet. O armazenamento de blobs tem a mesma finalidade dos serviços S3 e EBS do AWS.
-   [Armazenamento de tabelas](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/) - Armazena conjuntos de dados estruturados. O Armazenamento de tabelas é um armazenamento de dados de atributos de chave NoSQL que acelera o desenvolvimento e o acesso a grandes quantidades de dados. Semelhante aos serviços SimpleDB e DynamoDB do AWS.

-   [Armazenamento de filas](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - Fornece um sistema de mensagens para processamento de fluxos de trabalho e comunicação entre componentes de serviços de nuvem.

-   [Armazenamento de arquivos](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/) - Oferece armazenamento compartilhado para aplicativos herdados que usam o protocolo padrão de bloco de mensagens de servidor (SMB). O armazenamento de arquivo é usado de maneira semelhante ao EFS da plataforma AWS.
 
#### <a name="glacier-and-azure-storage"></a>Glacier e Armazenamento do Azure 

O [Armazenamento de Blobs de Arquivo Morto do Azure](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier) é comparável ao serviço de armazenamento AWS Glacier. Ele se destina a dados acessados raramente que ficam armazenados por pelo menos 180 dias e podem tolerar várias horas de latência de recuperação. 

Para dados que são acessados com pouca frequência, mas devem estar disponíveis imediatamente quando acessados, a [Camada de Armazenamento de Blobs Fria do Azure](/azure/storage/blobs/storage-blob-storage-tiers#cool-access-tier) fornece armazenamento mais barato do que o armazenamento de blobs padrão. Esta camada de armazenamento é comparável ao AWS S3 – serviço de armazenamento de Acesso Não Frequente.

#### <a name="see-also"></a>Consulte também

-   [Lista de verificação de desempenho e escalabilidade do Armazenamento do Microsoft Azure](https://azure.microsoft.com/documentation/articles/storage-performance-checklist/)

-   [Guia de segurança do Armazenamento do Azure](https://azure.microsoft.com/documentation/articles/storage-security-guide/)

-   [Padrões e práticas: diretriz da Rede de Distribuição de Conteúdo (CDN)](https://azure.microsoft.com/documentation/articles/best-practices-cdn/)

### <a name="networking"></a>Rede

#### <a name="elastic-load-balancing-azure-load-balancer-and-azure-application-gateway"></a>Elastic Load Balancing, Azure Load Balancer e Gateway de Aplicativo do Azure

Os equivalentes do Azure aos dois serviços Elastic Load Balancing são:

-   [Load Balancer](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) - Fornece os mesmos recursos que o Classic Load Balancer do AWS, permitindo distribuir o tráfego por várias VMs no nível da rede. Ele também fornece recursos de failover.

-   [Gateway de Aplicativo](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) - Oferece roteamento com base em regra no nível de aplicativo comparável ao Application Load Balancer do AWS.

#### <a name="route-53-azure-dns-and-azure-traffic-manager"></a>Route 53, DNS do Azure e Gerenciador de Tráfego do Azure

No AWS, o Route 53 fornece serviços de gerenciamento de nomes DNS e roteamento de tráfego em nível de DNS e failover. No Azure, isso é feito por dois serviços:

-   O [DNS do Azure](https://azure.microsoft.com/documentation/services/dns/) fornece gerenciamento de domínio e de DNS.

-   O [Gerenciador de Tráfego][traffic-manager] fornece recursos de roteamento de DNS em nível de tráfego, balanceamento de carga e failover.

#### <a name="direct-connect-and-azure-expressroute"></a>Conexão direta e Azure ExpressRoute

O Azure fornece conexões dedicadas site a site semelhantes por meio do seu serviço [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/). O ExpressRoute permite conectar a rede local diretamente a recursos do Azure usando uma conexão de rede privada dedicada. O Azure também oferece [conexões VPN site a site](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/) mais convencionais, com menor custo.

#### <a name="see-also"></a>Consulte também

-   [Criar uma rede virtual usando o portal do Azure](https://azure.microsoft.com/documentation/articles/virtual-networks-create-vnet-arm-pportal/)

-   [Planejar e projetar redes virtuais do Azure](https://azure.microsoft.com/documentation/articles/virtual-network-vnet-plan-design-arm/)

-   [Melhores práticas de segurança de rede do Azure](https://azure.microsoft.com/documentation/articles/azure-security-network-security-best-practices/)

### <a name="database-services"></a>Serviços de banco de dados

#### <a name="rds-and-azure-relational-database-services"></a>Serviços de banco de dados relacional do Azure e RDS

O Azure fornece vários serviços de outro banco de dados relacional que serão o equivalente do Serviço de banco de dados relacional (RDS) do AWS.

-   [Banco de Dados SQL](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)
-   [Banco de Dados do Azure para MySQL](https://docs.microsoft.com/azure/mysql/overview)
-   [Banco de Dados do Azure para PostgreSQL](https://docs.microsoft.com/azure/postgresql/overview)

Outros mecanismos de bancos de dados, como [SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/), [Oracle](https://azure.microsoft.com/campaigns/oracle/) e [MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/) podem ser implantados com a utilização de Instâncias de VM do Azure.

Os custos de RDS do AWS são determinados pela quantidade de recursos de hardware usados pela instância, como CPU, RAM, armazenamento e largura de banda de rede. Nos serviços de banco de dados do Azure, o custo depende do tamanho do seu banco de dados, das conexões simultâneas e dos níveis de produtividade.

#### <a name="see-also"></a>Consulte também

-   [Tutoriais do Banco de Dados SQL do Azure](https://azure.microsoft.com/documentation/articles/sql-database-explore-tutorials/)

-   [Configurar a replicação geográfica para o Banco de Dados SQL do Azure com o portal do Azure](https://azure.microsoft.com/documentation/articles/sql-database-geo-replication-portal/)

-   [Introdução ao Cosmos DB: um banco de dados JSON NoSQL](/azure/cosmos-db/sql-api-introduction)

-   [Como usar o armazenamento de Tabela do Azure com Node.js](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/)

### <a name="security-and-identity"></a>Segurança e identidade

#### <a name="directory-service-and-azure-active-directory"></a>Serviço de diretório e Azure Active Directory

O Azure divide os serviços de diretório em:

-   [Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) - Serviço de gerenciamento de identidades e diretórios baseado em nuvem.

-   [Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) - Habilita o acesso aos seus aplicativos corporativos com identidades gerenciadas pelo parceiro.

-   [Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) - Serviço que oferece suporte ao gerenciamento de logon e usuário único para aplicativos destinados ao consumidor.

-   [Azure Active Directory Domain Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) - Serviço de controlador de domínio hospedado que permite o gerenciamento da junção de domínios compatíveis e usuários.

#### <a name="web-application-firewall"></a>Firewall do aplicativo Web

Além do [Firewall do Aplicativo Web do Gateway de Aplicativo](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/), você também pode [usar firewalls de aplicativos Web](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/) fornecidos por terceiros, como o [Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/).

#### <a name="see-also"></a>Consulte também

-   [Introdução à segurança do Microsoft Azure](https://azure.microsoft.com/documentation/articles/azure-security-getting-started/)

-   [Práticas recomendadas de Gerenciamento de Identidade do Azure e segurança de controle de acesso](https://azure.microsoft.com/documentation/articles/azure-security-identity-management-best-practices/)

### <a name="application-and-messaging-services"></a>Serviços de aplicativos e de mensagens

#### <a name="simple-email-service"></a>Simple Email Service

O Simple Email Service (SES) do AWS envia emails de notificação, transacionais ou de marketing. No Azure, soluções de terceiros, como o [Sendgrid](https://sendgrid.com/partners/azure/), fornecem serviços de email.

#### <a name="simple-queueing-service"></a>Simple Queueing Service

O Simple Queueing Service (SQS) do AWS fornece um sistema de mensagens para conectar aplicativos, serviços e dispositivos dentro da plataforma AWS. O Azure tem dois serviços com funcionalidade semelhante:

-   [Armazenamento de Filas](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - Um serviço de mensagens em nuvem que permite a comunicação entre componentes de aplicativos na plataforma Azure.

-   [Barramento de Serviço](https://azure.microsoft.com/services/service-bus/) - Um sistema de mensagens mais robusto, para conectar aplicativos, serviços e dispositivos. Usando a [retransmissão do Barramento de Serviço](https://docs.microsoft.com/azure/service-bus-relay/relay-what-is-it) relacionada, o Barramento de Serviço também pode se conectar a aplicativos e serviços hospedados remotamente.

#### <a name="device-farm"></a>Device Farm

O Device Farm do AWS fornece serviços de teste de vários dispositivos. No Azure, o [Xamarin Test Cloud](https://www.xamarin.com/test-cloud) fornece testes de front-end semelhantes entre dispositivos para dispositivos móveis.

Além dos testes de front-end, o [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab/) fornece recursos de teste de back-end para ambientes Linux e Windows.

#### <a name="see-also"></a>Consulte também

-   [Como usar o Armazenamento de Fila do Node.js](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/)

-   [Como usar filas do Barramento de Serviço](https://azure.microsoft.com/documentation/articles/service-bus-nodejs-how-to-use-queues/)

### <a name="analytics-and-big-data"></a>Análises e big data

[O Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/) é o pacote de produtos e serviços do Azure projetados para capturar, organizar, analisar e visualizar grandes quantidades de dados. O pacote Cortana consiste nos seguintes serviços:

-   [HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - Distribuição gerenciada do Apache, que inclui Hadoop, Spark, Storm ou HBase.

-   [Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) - Fornece funcionalidade de orquestração e pipeline de dados.

-   [SQL Data Warehouse](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) - Armazenamento de dados relacionais em grande escala.

-   [Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/) - Armazenamento em larga escala otimizado para cargas de trabalho de análise de big data.

-   [Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/) - Usado para criar e aplicar a análise preditiva em dados.

-   [Stream Analytics](https://azure.microsoft.com/documentation/services/stream-analytics/) - Análise de dados em tempo real.

-   [Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) -Serviço de análise de larga escala, otimizado para trabalhar com o Data Lake Store

-   [PowerBI](https://powerbi.microsoft.com/) - Usado para visualização de dados de energia.

#### <a name="see-also"></a>Consulte também

-   [Galeria do Cortana Intelligence](https://gallery.cortanaintelligence.com/)

-   [Noções básicas sobre soluções de big data da Microsoft](https://msdn.microsoft.com/library/dn749804.aspx)

-   [Blog do Azure Data Lake e do Azure HDInsight](https://blogs.msdn.microsoft.com/azuredatalake/)

### <a name="internet-of-things"></a>Internet das coisas

#### <a name="see-also"></a>Consulte também

-   [Introdução ao Hub IoT do Azure](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)

-   [Comparação entre Hub IoT e Hubs de Eventos](https://azure.microsoft.com/documentation/articles/iot-hub-compare-event-hubs/)

### <a name="mobile-services"></a>Serviços móveis

#### <a name="notifications"></a>Notificações

Os Hubs de Notificação não dão suporte ao envio de SMS ou mensagens de email; por isso, são necessários serviços de terceiros para esses tipos de entrega.

#### <a name="see-also"></a>Consulte também

-   [Criar um aplicativo Android](https://azure.microsoft.com/documentation/articles/app-service-mobile-android-get-started/)

-   [Autenticação e Autorização nos Aplicativos Móveis do Azure](https://azure.microsoft.com/documentation/articles/app-service-mobile-auth/)

-   [Como enviar notificações por push com Hubs de Notificação do Azure](https://azure.microsoft.com/documentation/articles/notification-hubs-android-push-notification-google-fcm-get-started/)

### <a name="management-and-monitoring"></a>Gerenciamento e monitoramento

#### <a name="see-also"></a>Consulte também
-   [Diretrizes de monitoramento e diagnóstico](https://azure.microsoft.com/documentation/articles/best-practices-monitoring/)

-   [Práticas recomendadas para criação de modelos do Azure Resource Manager](https://azure.microsoft.com/documentation/articles/resource-manager-template-best-practices/)

-   [Modelos de início rápido do Azure Resource Manager](https://azure.microsoft.com/documentation/templates/)


## <a name="next-steps"></a>Próximas etapas

-   [Matriz de comparação completa de serviços AWS e Azure](https://aka.ms/azure4aws-services)

-   [Visão geral interativa da plataforma Azure](http://azureplatform.azurewebsites.net/)

-   [Introdução ao Azure](https://azure.microsoft.com/get-started/)

-   [Arquiteturas de solução do Azure](https://azure.microsoft.com/solutions/architecture/)

-   [Arquiteturas de referência do Azure](https://azure.microsoft.com/documentation/articles/guidance-architecture/)

-   [Padrões e práticas: orientação sobre o Azure](https://azure.microsoft.com/documentation/articles/guidance/)

-   [Curso online gratuito: Microsoft Azure para especialistas em AWS](http://aka.ms/azureforaws)


<!-- links -->

[paired-regions]: https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/
[traffic-manager]: /azure/traffic-manager/