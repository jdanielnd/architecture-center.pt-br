---
title: 'Datacenter virtual do Azure: uma perspectiva de rede'
description: Aprenda a criar seu datacenter virtual no Azure
author: tracsman
manager: rossort
tags: azure-resource-manager
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: virtual-network
ms.date: 11/28/2018
ms.author: jonor
ms.openlocfilehash: a997a0f03da63bc1432f61f3299e7c6794278e5e
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908539"
---
# <a name="azure-virtual-datacenter-a-network-perspective"></a>Datacenter virtual do Azure: uma perspectiva de rede

## <a name="overview"></a>Visão geral

A migração de aplicativos locais para o Azure fornece às organizações os benefícios de uma infraestrutura segura e econômica, mesmo se os aplicativos forem migrados com o mínimo de alterações. No entanto, para aproveitar ao máximo a agilidade com a computação em nuvem, as empresas precisam evoluir suas arquiteturas para aproveitarem os recursos do Azure. 

O Microsoft Azure oferece serviços e infraestrutura de hiperescala com recursos e confiabilidade de nível empresarial. Esses serviços e infraestrutura oferecem muitas opções de conectividade híbrida para que os clientes possam escolher acessá-los pela Internet pública ou por uma conexão privada do Azure ExpressRoute. Os parceiros da Microsoft também fornecem recursos avançados, oferecendo serviços de segurança e soluções de virtualização otimizados para execução no Azure.

Os clientes podem optar por acessar esses serviços de nuvem pela Internet ou com o Azure ExpressRoute, que fornece conectividade de rede privada. Com a plataforma Microsoft Azure, os clientes podem estender facilmente sua infraestrutura para a nuvem e criar arquiteturas com várias camadas. Os parceiros da Microsoft também fornecem recursos avançados, oferecendo serviços de segurança e soluções de virtualização otimizados para execução no Azure.

## <a name="what-is-the-virtual-datacenter"></a>O que é o Datacenter Virtual?

No início, a nuvem era, essencialmente, uma plataforma de hospedagem de aplicativos voltados ao público. As empresas começaram a entender o valor da nuvem e começaram a mover aplicativos de linha de negócios internos para ela. Esses tipos de aplicativo trouxeram mais segurança, confiabilidade, desempenho e considerações de custo que exigiam maior flexibilidade na forma de prestar os serviços de nuvem. Isso abriu caminho para novos serviços de rede e infraestrutura criados para dar essa flexibilidade, mas também para novos recursos de escala, recuperação de desastre e outras questões.

## <a name="what-is-a-virtual-datacenter"></a>O que é um datacenter virtual?
As soluções de nuvem eram projetadas para hospedar aplicativos únicos relativamente isolados no espectro público. Essa abordagem funcionou bem por alguns anos. Depois, os benefícios das soluções de nuvem tornaram-se evidentes, e cargas de trabalho em grande escala passaram a ser hospedadas na nuvem. Lidar com questões relacionadas à segurança, confiabilidade, desempenho e custos com implantações se tornou fundamental durante todo o ciclo de vida do serviço de nuvem.

O diagrama de implantação de nuvem a seguir mostra um exemplo de falha de segurança na **caixa vermelha**. A **caixa amarela** mostra espaço para soluções de virtualização de rede de otimização em cargas de trabalho.

[![0]][0]

O VDC (Datacenter Virtual) é um conceito nascido da necessidade de se ajustar o dimensionamento para dar suporte a cargas de trabalho corporativas e, ao mesmo tempo, lidar com os problemas introduzidos pelo suporte a aplicativos em grande escala na nuvem pública.

Uma implementação de VDC não representa apenas as cargas de trabalho de aplicativos na nuvem, mas também a rede, a segurança, o gerenciamento e a infraestrutura (por exemplo, DNS e Serviços de Diretório). Como cada vez mais cargas de trabalho corporativas são movidas para o Azure, é importante pensar na infraestrutura de suporte e nos objetos em que essas cargas de trabalho são colocadas. Pensar cuidadosamente em como os recursos são estruturados pode evitar a proliferação de centenas de "ilhas de carga de trabalho" que devem ser gerenciadas separadamente com o fluxo de dados, modelos de segurança e desafios de conformidade independentes.

O conceito de VDC é um conjunto de recomendações e melhores práticas para implementar uma coleção de entidades separadas, porém relacionadas, com infraestrutura, recursos e funções de suporte em comum. Ao observar suas cargas de trabalho pela perspectiva do VDC, você poderá aproveitar o custo reduzido devido a economias de escala, segurança otimizada por meio da centralização de componentes e fluxos de dados, além de operações, gerenciamento e auditorias de conformidade mais fáceis.

> [!NOTE]
> É importante compreender que o VDC **NÃO** é um produto separado do Azure, mas a combinação de vários recursos e capacidades para atender às suas necessidades específicas. O VDC é uma maneira de pensar sobre suas cargas de trabalho e o uso do Azure para maximizar os recursos e as habilidades na nuvem. É uma abordagem modular de criação de serviços de TI no Azure que, ao mesmo tempo, respeita as funções organizacionais e as responsabilidades da empresa.

Uma implementação de VDC permite que as empresas coloquem cargas de trabalho e aplicativos no Azure nos seguintes cenários:

-   Hospedagem de várias cargas de trabalho relacionadas.
-   Migração de cargas de trabalho de um ambiente local para o Azure.
-   Implementação de segurança compartilhada ou centralizada e requisitos de acesso entre cargas de trabalho.
-   Como combinar TI centralizada e DevOps apropriadamente para uma grande empresa.

A chave para desbloquear as vantagens do VDC é uma topologia de rede hub-spoke centralizada com uma combinação de recursos e serviços do Azure:

* [Rede Virtual do Azure][VNet];
* [Grupos de segurança de rede][NSG];
* [Emparelhamento de rede virtual][VNetPeering];
* [UDR (Rotas Definidas pelo Usuário)][UDR] e
* Azure Identity com [RBAC (controle de acesso baseado em função)][RBAC] e, opcionalmente, [Firewall do Azure][AzFW], [DNS do Azure][DNS], [Azure Front Door][AFD] e [WAN Virtual do Azure][vWAN].

## <a name="who-should-implement-a-virtual-datacenter"></a>Quem deve implementar um Datacenter Virtual?

Qualquer cliente do Azure que tenha decidido adotar a nuvem pode se beneficiar da eficiência de configurar um conjunto de recursos para uso comum por todos os aplicativos. Dependendo da magnitude, até mesmo aplicativos únicos podem se beneficiar do uso de padrões e componentes empregados para criar uma implementação de VDC.

Se sua organização tiver uma equipe ou departamento de TI, Rede, Segurança e/ou Conformidade centralizada, a implementação de um VDC poderá ajudar a impor pontos da política, separar obrigações e garantir a uniformidade dos componentes comuns subjacentes, enquanto dá às equipes de aplicativo o máximo de liberdade e controle adequado às suas necessidades.

As organizações que estão considerando o DevOps também podem utilizar os conceitos do VDC para fornecer grupos autorizados de recursos do Azure e garantir que tenham controle total dentro desse grupo (um grupo de recursos ou de assinaturas em uma assinatura comum), mas mantendo os limites de segurança e de rede em conformidade, como definido por uma política centralizada em um Grupo de Recursos e em uma VNET de um hub.

## <a name="considerations-for-implementing-a-virtual-datacenter"></a>Considerações sobre a implementação de um Datacenter Virtual

Ao projetar uma implementação de VDC, há várias questões importantes a considerar:

### <a name="identity-and-directory-service"></a>Serviço de Identidade e Diretório

Os serviços de Identidade e Diretório são um aspecto fundamental de todos os datacenters, sejam locais ou na nuvem. A identidade está relacionada a todos os aspectos de autorização e acesso a serviços em uma implementação de VDC. Para ajudar a garantir que somente usuários e processos autorizados acessem seus recursos e Conta do Azure, o Azure usa vários tipos de credenciais para autenticação. Eles incluem senhas (para acessar a conta do Azure), chaves de criptografia, assinaturas digitais e certificados. A [MFA (Autenticação Multifator do Azure)][MFA] é uma camada adicional de segurança para acessar os serviços do Azure. O Azure MFA oferece autenticação forte com uma variedade de opções fáceis de verificação, como chamada telefônica, mensagem de texto ou notificação por aplicativo móvel, permitindo que os clientes escolham seus métodos preferidos.

As grandes empresas precisam definir um processo de gerenciamento de identidades que descreva o gerenciamento de identidades individuais, autenticação, autorização, funções e privilégios em toda a implementação de VDC. Os objetivos deste processo devem ser aumentar a segurança e a produtividade e diminuir o custo, o tempo de inatividade e as tarefas manuais repetitivas.

Empresas/organizações podem exigir uma combinação exigente de serviços para diferentes LOBs (linha de negócios) e os funcionários geralmente têm diferentes funções quando estão envolvido em projetos diferentes. O VDC requer uma boa cooperação entre diferentes equipes, cada uma com definições de função específicas, para que os sistemas sejam executados com boa governança. A matriz de responsabilidades, acesso e direitos pode ser complexa. O gerenciamento de identidades no VDC é implementado por meio do [*Azure AD* (Azure Active Directory)][AAD] e do RBAC (controle de acesso baseado em função).

Um serviço de diretório é uma infraestrutura de informações compartilhadas que localiza, gerencia, administra e organiza itens cotidianos e recursos de rede. Esses recursos podem incluir volumes, pastas, arquivos, impressoras, usuários, grupos, dispositivos e outros objetos. Cada recurso na rede é considerado um objeto pelo servidor de diretório. Informações sobre um recurso são armazenadas como uma coleção de atributos associados a tal recurso ou objeto.

Todos os serviços comerciais do Microsoft Online dependem do Azure Active Directory (Azure AD) para entrada e outras necessidades de identidade. O Azure Active Directory é uma solução de nuvem de gerenciamento de acesso e identidade abrangente e altamente disponível que combina os serviços principais de diretório, governança avançada de identidades e gerenciamento do acesso de aplicativos. O Azure AD pode ser integrado ao Active Directory local para habilitar o logon único para todos os aplicativos hospedados localmente e baseados em nuvem (local). Os atributos do usuário do Active Directory local podem ser sincronizados automaticamente com o Azure AD.

Não é necessário haver um único administrador global para atribuir todas as permissões em uma implementação de VDC. Cada departamento, grupo de usuários ou serviço específico no Serviço de Diretório pode ter as permissões necessárias para gerenciar seus próprios recursos em uma implementação de VDC. Estruturar permissões requer balanceamento. Um número excessivo de permissões pode prejudicar a eficiência de desempenho, enquanto permissões de menos ou não exigentes podem aumentar os riscos de segurança. O RBAC (controle de acesso baseado em função) do Azure ajuda a resolver esse problema oferecendo gerenciamento de acesso refinado para recursos em uma implementação de VDC.

#### <a name="security-infrastructure"></a>Infraestrutura de segurança

A infraestrutura de segurança diz respeito à segregação de tráfego em um segmento de rede virtual específico da implementação de VDC. Essa infraestrutura especifica como a entrada e a saída são controladas em uma implementação de VDC. O Azure se baseia em uma arquitetura multilocatário que impede o tráfego não autorizado e não intencional entre implantações usando isolamento de VNET (rede virtual), ACLs (listas de controle de acesso), balanceadores de carga, filtros IP e políticas de fluxo de tráfego. NAT (conversão de endereços de rede) separa o tráfego de rede interno do tráfego externo.

A malha do Azure aloca recursos de infraestrutura a cargas de trabalho de locatário e gerencia comunicações para e de VMs (máquinas virtuais). O hipervisor do Azure impõe a separação de memória e processo entre VMs e encaminha com segurança o tráfego de rede a locatários do sistema operacional convidado.

#### <a name="connectivity-to-the-cloud"></a>Conectividade com a nuvem

A implementação de VDC exige conectividade com redes externas para oferecer serviços a clientes, parceiros e/ou usuários internos. Essa necessidade de conectividade diz respeito não só à Internet, mas também a datacenters e redes locais.

Os clientes controlam a quais serviços eles têm acesso e quais ficam acessíveis pela Internet pública usando o [Firewall do Azure][AzFW] ou outros tipos de dispositivos de rede virtual (NVAs), políticas de roteamento personalizadas, usando [rotas definidas pelo usuário][UDR], assim como filtragem de rede, usando [grupos de segurança de rede][NSG]. É recomendável que todos os recursos voltados para Internet também sejam protegidos pela [Proteção contra DDoS do Azure Standard][DDOS].

As empresas podem precisar conectar suas implementações de VDC a datacenters locais ou a outros recursos. Essa conectividade entre o Azure e as redes locais é um aspecto essencial de um projeto de arquitetura eficaz. As empresas têm duas maneiras diferentes de criar essa interconexão: trânsito pela Internet e/ou por conexões diretas privadas.

[**VPN Site a Site do Azure**][VPN] é um serviço de interconexão pela Internet entre redes locais e uma implementação de VDC, estabelecido por meio de conexões criptografadas seguras (túneis IPsec/IKE). A conexão Site a Site do Azure é flexível e rápida de criar e não requer nenhuma compra adicional, uma vez que todas as conexões são feitas pela Internet.

Para um grande número de conexões VPN, a [**WAN Virtual do Azure**][vWAN] é um serviço de rede que fornece conectividade otimizada e automatizada entre branches por meio do Azure. A WAN Virtual permite que você conecte e configure dispositivos de branch para se comunicar com o Azure. A conexão e a configuração podem ser feitas manualmente ou usando os dispositivos de provedor preferidos por meio de um parceiro de WAN Virtual. O uso dos dispositivos de provedor preferidos facilitam o uso, simplificam a conectividade e permitem o gerenciamento da configuração. O painel interno de WAN do Azure fornece informações instantâneas de solução de problemas que podem ajudar você a economizar tempo e proporcionam uma maneira fácil de exibir conectividade site a site em larga escala.

O [**ExpressRoute**][ExR] é um serviço de conectividade do Azure que permite conexões privadas entre uma implementação de VDC e eventuais redes locais. Conexões do ExpressRoute não passam pela Internet pública e oferecem maior segurança, confiabilidade e velocidades mais altas (até 10 Gbps), além de latência consistente. O ExpressRoute é útil para implementações de VDC, uma vez que os clientes do ExpressRoute aproveitam os benefícios das regras de conformidade associadas às conexões privadas. Com o [ExpressRoute Direct][ExRD], é possível se conectar diretamente aos roteadores da Microsoft a 100 Gbps para clientes com maiores necessidades de largura de banda.

Implantar conexões do ExpressRoute envolve normalmente a interação com um provedor de serviços do ExpressRoute. Para clientes que precisam começar rapidamente, é comum usar inicialmente a VPN Site a Site para estabelecer a conectividade entre uma implementação de VDC e os recursos locais e, em seguida, migrar para a conexão do ExpressRoute quando a interconexão física com o provedor de serviços estiver concluída.

#### <a name="connectivity-within-the-cloud"></a>*Conectividade na nuvem*

As [VNets][VNet] e [Emparelhamento VNET][VNetPeering] são serviços de conectividade de rede básicos em uma implementação de VDC. Uma VNet garante um limite natural de isolamento para recursos de VDC e o emparelhamento VNet permite a intercomunicação entre diferentes VNets dentro da mesma região do Azure ou até mesmo entre regiões. Controle de tráfego em uma VNet e entre VNets precisa corresponder a um conjunto de regras de segurança especificado por meio de Listas de Controle de Acesso ([Grupo de Segurança de Rede][NSG]), [Soluções de Virtualização de Rede][NVA] e as tabelas de roteamento personalizadas ([UDR][UDR]).

## <a name="virtual-datacenter-overview"></a>Visão geral do datacenter virtual

### <a name="topology"></a>Topologia

O _Hub-spoke_ é um modelo para projetar a topologia de rede para uma implementação do Datacenter Virtual. 

[![1]][1]

Um hub é a zona central da rede que controla e inspeciona o tráfego de entrada ou saída entre diferentes zonas: Internet, local e os spokes. A topologia hub-spoke dá ao departamento de TI uma maneira eficiente de impor políticas de segurança em um local central. Ela também reduz a possibilidade de erros de configuração e exposição.

O hub contém os componentes de serviço comuns consumidos pelos spokes. Os exemplos a seguir são serviços centrais comuns:

-   A infraestrutura do Windows Active Directory, necessária para a autenticação de usuários de terceiros que acessam de redes não confiáveis antes de obterem acesso às cargas de trabalho no spoke. Ela inclui os Serviços de Federação do Active Directory (AD FS).
-   Um serviço DNS para resolver nomes para a carga de trabalho nos spokes, para acessar recursos locais e na Internet se o [DNS do Azure][DNS] não for usado.
-   Uma infraestrutura de chave pública (PKI) para implementar logon único em cargas de trabalho.
-   Controle de fluxo de tráfego TCP e UDP entre as zonas da rede de spoke e a Internet.
-   Controle de fluxo entre os spokes e o local.
-   Se necessário, fluxo de controle entre um spoke e outro.

O VDC reduz o custo geral usando a infraestrutura de hub compartilhada entre vários spokes.

A função de cada spoke pode ser hospedar diferentes tipos de cargas de trabalho. Os spokes também fornecem uma abordagem modular para implantações repetíveis das mesmas cargas de trabalho. Alguns exemplos são: desenvolvimento e teste, testes de aceitação do usuário, pré-produção e produção. Os spokes também podem separar e habilitar diferentes grupos na sua organização. Um exemplo são os grupos Azure DevOps. Dentro de um spoke, é possível implantar uma carga de trabalho básica ou cargas de trabalho complexas de várias camadas com controle de tráfego entre as camadas.

#### <a name="subscription-limits-and-multiple-hubs"></a>Limites de assinatura e vários hubs

No Azure, cada componente, seja qual for o tipo, é implantado em uma Assinatura do Azure. O isolamento dos componentes do Azure em diferentes assinaturas do Azure pode atender aos requisitos de diferentes LOBs, como configurar níveis diferenciados de acesso e autorização.

Uma única implementação de VDC pode escalar verticalmente para vários spokes, embora haja limites de plataformas, como acontece com todos os sistemas de TI. A implantação do hub está associada a uma assinatura específica do Azure, que tem restrições e limites (por exemplo, um número máximo de emparelhamentos de VNet – consulte [Assinatura do Azure e limites de serviço, cotas e restrições][Limits] para obter detalhes). Em casos em que os limites possam ser um problema, a arquitetura pode ser escalada verticalmente ainda mais estendendo o modelo de um único hub-spokes para um cluster de hub e spokes. Vários hubs em uma ou mais regiões do Azure podem ser interconectados usando Emparelhamento VNET, ExpressRoute, WAN Virtual ou VPN site a site.

[![2]][2]

A introdução de vários hubs aumenta o esforço de gerenciamento e o custo do sistema. Isso só seria justificado pela escalabilidade, como limites do sistema ou redundância e replicação regional, como o desempenho do usuário final ou a recuperação de desastres. Em cenários que exigem vários hubs, todos os hubs devem buscar oferecer o mesmo conjunto de serviços para facilidade operacional.

#### <a name="interconnection-between-spokes"></a>Interconexão entre spokes

Dentro de um único spoke, é possível implementar cargas de trabalho complexas de várias camadas. Configurações de várias camadas podem ser implementadas usando sub-redes (uma para cada camada) na mesma VNet e filtrando os fluxos usando NSGs.

Um arquiteto talvez queira implantar uma carga de trabalho de várias camadas em várias redes virtuais. Com um emparelhamento de redes virtuais, os spokes podem se conectar a outros spokes no mesmo hub ou em hubs diferentes. Um exemplo típico desse cenário é o caso em que os servidores de processamento do aplicativo estão em um spoke ou rede virtual. O banco de dados é implantado em um spoke ou em uma rede virtual diferente. Nesse caso, é fácil interconectar os spokes ao emparelhamento de redes virtuais e, assim, evitar trânsito pelo hub. Um exame cuidadoso de arquitetura e segurança deve ser executado para garantir que ignorar o hub não ignorará pontos de auditoria ou segurança importantes que possam existir somente no hub.

[![3]][3]

Os spokes também podem ser interconectados a um spoke que atue como um hub. Essa abordagem cria uma hierarquia de dois níveis: o spoke no nível mais alto (nível 0) torna-se o hub dos spokes inferiores (nível 1) da hierarquia. Os spokes de uma implementação de VDC são necessários para encaminhar o tráfego ao hub central para que o tráfego possa seguir até seu destino na rede local ou na Internet pública. Uma arquitetura com dois níveis de hub apresenta roteamento complexo que remove os benefícios de uma relação de hub-spoke simples.

Embora o Azure permita topologias complexas, dois princípios mais importantes do conceito de VDC são a repetição e a simplicidade. Para minimizar o esforço de gerenciamento, o design simples de hub-spoke é a arquitetura de referência do VDC que nós recomendamos.

### <a name="components"></a>Componentes

O Datacenter Virtual é composto por quatro tipos de componente básicos: **Infraestrutura**, **Redes de Perímetro**, **Cargas de Trabalho** e **Monitoramento**.

Cada tipo de componente consiste em vários recursos e funcionalidades do Azure. Sua implementação de VDC é composta por instâncias de vários tipos de componentes e muitas variações do mesmo tipo de componente. Por exemplo, você pode ter muitas instâncias de carga de trabalho diferentes e logicamente separadas que representem diferentes aplicativos. Você usa esses diferentes tipos e instâncias de componentes para criar, por fim, o VDC.

[![4]][4]

A arquitetura conceitual de alto nível do VDC anterior mostra diferentes tipos de componente usados em diferentes zonas na topologia hub-spokes. O diagrama mostra os componentes da infraestrutura em várias partes da arquitetura.

Como boa prática em geral, os privilégios e direitos de acesso devem se basear nos grupos. Lidar com grupos em vez de usuários individuais facilita a manutenção das políticas de acesso, oferecendo uma maneira consistente para gerenciá-las entre as equipes.  e ajuda a minimizar os erros de configuração. Atribuir e remover usuários de e para grupos apropriados ajuda a manter os privilégios de um usuário específico atualizados.

Cada grupo de função deve ter um prefixo exclusivo em seus nomes. Esse prefixo torna mais fácil identificar qual grupo está associado a qual carga de trabalho. Por exemplo, uma carga de trabalho que hospede um serviço de autenticação pode ter grupos chamados **AuthServiceNetOps**, **AuthServiceSecOps**, **AuthServiceDevOps** e **AuthServiceInfraOps**. Funções centralizadas ou funções não relacionadas a um serviço específico podem ser antecedidas por **Corp**. Um exemplo é **CorpNetOps**.

Muitas organizações usam uma variação dos seguintes grupos para fornecer uma divisão importante de funções:

-   O grupo de TI central **Corp** tem os direitos de propriedade para controlar os componentes de infraestrutura. Exemplos de rede e segurança. O grupo deve ter a função de colaborador na assinatura, o controle do hub e direitos de colaborador de rede nos spokes. Organizações de grande porte dividem com frequência essas responsabilidades de gerenciamento entre várias equipes. Por exemplo, o grupo **CorpNetOps** de operações de rede com foco exclusivo na rede e o grupo **CorpSecOps** de operações de segurança, responsável pela política de firewall e segurança. Nesse caso específico, os dois grupos diferentes precisam ser criados para a atribuição dessas funções personalizadas.
-   O grupo de desenvolvimento e teste **AppDevOps** tem a responsabilidade de implantar cargas de trabalho de aplicativos ou serviços. Esse grupo usa a função de colaborador de máquina virtual para implantações de IaaS e/ou uma ou mais funções de colaborador de PaaS. Confira [Funções internas para recursos do Azure][Roles]. Opcionalmente, a equipe de desenvolvimento e teste pode precisar ter visibilidade de políticas de segurança, NSGs, e políticas de roteamento, UDR, dentro do hub ou de um spoke específico. Além das funções de colaborador para cargas de trabalho, esse grupo também precisaria da função de leitor de rede.
-   O grupo de operação e manutenção, **CorpInfraOps** ou **AppInfraOps,** tem a responsabilidade de gerenciar cargas de trabalho em produção. Esse grupo deve ser um colaborador de assinatura em cargas de trabalho em qualquer assinatura de produção. Algumas organizações também podem avaliar se elas precisam de um grupo de equipe de suporte de escalonamento adicional com a função de colaborador de assinatura em produção e na assinatura do hub central. O grupo adicional corrige possíveis problemas de configuração no ambiente de produção.

O VDC é projetado de modo que os grupos criados para os grupos de TI centrais que gerenciam o hub tenham grupos correspondentes no nível da carga de trabalho. Em vez de apenas gerenciar os recursos de hub, o grupo de TI central é capaz de controlar o acesso externo e as permissões de nível superior na assinatura. Os grupos de carga de trabalho também conseguem controlar recursos e permissões da VNet de modo independente no grupo de TI Central.

O VDC é particionado para hospedar vários projetos com segurança em diferentes LOB (linhas de negócios). Todos os projetos exigem diferentes ambientes isolados (Dev, UAT, produção). Assinaturas separadas do Azure para cada um desses ambientes fornecem isolamento natural.

[![5]][5]

O diagrama anterior mostra a relação entre projetos, usuários e grupos, bem como os ambientes de uma organização em que os componentes do Azure são implantados.

Normalmente, em TI, um ambiente (ou camada) é um sistema em que vários aplicativos são implantados e executados. Grandes empresas usam um ambiente de desenvolvimento (em que as alterações são originalmente feitas e testadas) e um ambiente de produção (que os usuários finais usam). Esses ambientes são separados, geralmente com vários ambientes de preparo entre eles para permitir implantação em fases (distribuição), testes e reversão em caso de problemas. As arquiteturas de implantação variam significativamente, mas geralmente o processo básico de iniciar em desenvolvimento (DEV) e terminar em produção (PROD) ainda é seguido.

Uma arquitetura comum para esses tipos de ambientes de várias camadas consiste em ambientes de produção Azure DevOps para desenvolvimento e teste, UAT de preparo e ambientes de produção. As organizações podem aproveitar locatários do Azure AD únicos ou múltiplos para definir o acesso e os direitos para esses ambientes. O diagrama anterior mostra um caso em que dois locatários do Azure AD diferentes são usados: um para Azure DevOps e UAT e outro exclusivamente para produção.

A presença de diferentes locatários do Azure AD impõe a separação entre ambientes. O mesmo grupo de usuários, como a TI central, precisa autenticar usando um URI diferente para acessar um locatário diferente do Azure AD, a fim de modificar as funções ou permissões de ambientes do Azure DevOps ou de produção de um projeto. A presença de diferentes autenticações de usuário para acessar diferentes ambientes reduz possíveis interrupções e outros problemas causados por erros humanos.

#### <a name="component-type-infrastructure"></a>Tipo de componente: Infraestrutura

Esse tipo de componente é o local em que a maioria da infraestrutura de suporte reside. Também é o ponto em que as equipes centralizadas de TI, Segurança e/ou Conformidade passam a maior parte do tempo.

[![6]][6]

Os componentes de infraestrutura fornecem uma interconexão para vários componentes de uma implementação de VDC e estão presentes no hub e nos spokes. Normalmente, a responsabilidade por gerenciar e manter os componentes da infraestrutura é atribuída à equipe de segurança e/ou TI central.

Uma das principais tarefas da equipe de infraestrutura de TI é assegurar a consistência dos esquemas de endereço IP em toda a empresa. O espaço do endereço IP privado atribuído à implementação de VDC precisa ser consistente e NÃO se sobrepor aos endereços IP privados atribuídos em suas redes locais.

Embora NAT nos roteadores de borda locais ou em ambientes do Azure possa evitar conflitos de endereço IP, ele adiciona complicações aos componentes da sua infraestrutura. A simplicidade de gerenciamento é uma das principais metas do VDC, ou seja, usar NAT para lidar com problemas de IP não é uma solução recomendada.

Os componentes da infraestrutura contêm a seguinte funcionalidade:

-   [**Serviços de identidade e diretório**][AAD]. Acesso a todos os tipos de recurso no Azure é controlado por uma identidade armazenada em um serviço de diretório. O serviço de diretório armazena não apenas a lista de usuários, mas também os direitos de acesso aos recursos em uma assinatura específica do Azure. Esses serviços podem existir somente em nuvem ou podem ser sincronizados com identidade local armazenada no Active Directory.
-   [**Rede Virtual**][VPN]. As Redes Virtuais são um dos principais componentes do VDC e permitem criar um limite de isolamento do tráfego na plataforma Azure. Uma Rede Virtual é composta por um ou vários segmentos de rede virtual, cada um com um prefixo de rede IP específico (uma sub-rede). A rede Virtual define uma área de perímetro interna em que máquinas virtuais de IaaS e serviços de PaaS podem estabelecer uma comunicação privada. VMs (e serviços de PaaS) em uma rede virtual não podem se comunicar diretamente com VMs (e serviços de PaaS) em uma rede virtual diferente, mesmo se as duas redes virtuais forem criadas pelo mesmo cliente, na mesma assinatura. Isolamento é uma propriedade vital que garante que as VMs e as comunicações do cliente permaneçam privadas em uma rede virtual.
-   [**UDR**][UDR]. O tráfego em uma Rede Virtual é roteado por padrão com base na tabela de roteamento do sistema. Uma Rota Definida pelo Usuário é uma tabela de roteamento personalizada que os administradores de rede podem associar a uma ou mais sub-redes para substituir o comportamento da tabela de roteamento do sistema e definir um caminho de comunicação em uma rede virtual. A presença de UDRs garante que o tráfego de saída spoke transite por VMs personalizadas específicas e/ou Dispositivos de Rede Virtual e balanceadores de carga presentes no hub e nos spokes.
-   [**NSG**][NSG]. Um Grupo de Segurança de Rede é uma lista de regras de segurança que atuam como filtragem de tráfego em Fontes IP, Destino IP, Protocolos, portas de Origem IP e portas de Destino IP. O NSG pode ser aplicado a uma sub-rede, um cartão de NIC Virtual associado a uma VM do Azure ou ambos. Os NSGs são essenciais para implementar um controle de fluxo correto no hub e nos spokes. O nível de segurança proporcionada pelo NSG é uma função de quais portas você abre e para qual finalidade. Os clientes devem aplicar filtros adicionais por VM com os firewalls baseados em host, como IPtables ou o Firewall do Windows.
-   [**DNS**][DNS]. A resolução de nomes dos recursos nas VNets de uma implementação de VDC é feita pelo DNS. O Azure fornece serviços de DNS para resolução de nomes [Públicos][DNS] e [Privados][PrivateDNS]. As zonas privadas fornecem resolução de nomes em uma rede virtual e em redes virtuais. É possível ter zonas privadas que abrangem não apenas redes virtuais na mesma região, mas também regiões e assinaturas. Para resolução pública, o DNS do Azure fornece um serviço de hospedagem para domínios DNS, fornecendo resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure.
-   [**Assinatura**][SubMgmt] e [**Gerenciamento de Grupo de Recursos**][RGMgmt]. Uma assinatura define um limite natural para criar vários grupos de recursos no Azure. Recursos em uma assinatura são montados em conjunto em contêineres lógicos denominados Grupos de Recursos. O Grupo de Recursos representa um grupo lógico para organizar os recursos de uma implementação de VDC.
-   [**RBAC**][RBAC]. Por meio de RBAC, é possível mapear a função organizacional junto com direitos de acesso a recursos específicos do Azure, permitindo que você restrinja os usuários a somente um certo subconjunto de ações. Com RBAC, você pode conceder acesso ao atribuir a função apropriada a usuários, grupos e aplicativos no escopo relevante. O escopo de uma atribuição de função pode ser uma assinatura do Azure, um grupo de recursos ou um único recurso. RBAC permite a herança de permissões. Uma função atribuída a um escopo pai também concede acesso aos filhos contidos nele. Usando RBAC, você pode separar as tarefas e conceder aos usuários apenas o nível de acesso de que eles precisam para trabalhar. Por exemplo, use RBAC para permitir que um funcionário gerencie máquinas virtuais em uma assinatura, enquanto outro pode gerenciar bancos de dados SQL na mesma assinatura.
-   [**Emparelhamento VNet**][VNetPeering]. O recurso fundamental usado para criar a infraestrutura do VDC é o Emparelhamento VNET, um mecanismo que conecta duas VNETs (redes virtuais) na mesma região por meio da rede do datacenter do Azure ou usando o backbone internacional do Azure entre regiões.

#### <a name="component-type-perimeter-networks"></a>Tipo de componente: Redes de perímetro

Os componentes de [rede de perímetro][DMZ] permitem a conectividade de rede entre suas redes de datacenter físicas ou locais, além da conectividade de Internet. Também é o local em que suas equipes de segurança de rede provavelmente passarão a maior parte do tempo.

Os pacotes de entrada devem fluir por dispositivos de segurança no hub antes de alcançar os servidores back-end nos spokes. Alguns exemplos são: firewall, IDS e IPS. Antes de saírem da rede, pacotes limitados à Internet das cargas de trabalho também devem fluir pelos dispositivos de segurança na rede de perímetro. As finalidades deste fluxo são a imposição de políticas, a inspeção e a auditoria.

Os componentes de rede de perímetro fornecem os seguintes recursos:

-   [Redes virtuais][VNet], [UDRs][UDR] e [NSGs][NSG]
-   [Soluções de virtualização de rede][NVA]
-   [Azure Load Balancer][ALB]
-   [Gateway de Aplicativo Azure][AppGW] e [firewall do aplicativo Web (WAF)][WAF]
-   [IPs Públicos][PIP]
-   [Azure Front Door][AFD]
-   [Firewall do Azure][AzFW]

Normalmente, as equipes de segurança e TI centrais têm a responsabilidade de definir requisitos e a operação das redes de perímetro.

[![7]][7]

O diagrama anterior mostra a imposição de dois perímetros com acesso à Internet e uma rede local, ambos residentes nos hubs DMZ e vWAN. Em um hub DMZ, a rede de perímetro para a Internet pode ser escalada verticalmente para dar suporte a grandes números de LOBs, usando vários farms de WAFs (Firewalls de Aplicativo Web) e/ou Firewall do Azure. No hub vWAN, a conectividade altamente escalonável entre branches e do branch para o Azure é realizada por meio de VPN ou ExpressRoute, conforme a necessidade.

[**Redes virtuais**][VNet]. O hub geralmente é criado em uma rede virtual com várias sub-redes para hospedar os diferentes tipos de serviços que filtram e inspecionam o tráfego de ou para a Internet por meio de NVAs, WAFs e instâncias de Gateway de Aplicativo do Azure.

[**UDR**][UDR] Usando UDRs, os clientes podem implantar firewalls, IDS/IPS e outras soluções de virtualização, além de rotear o tráfego de rede por meio dessas soluções de segurança para imposição de política, auditoria e inspeção de limites de segurança. Os UDRs podem ser criados no hub e nos spokes para assegurar que o tráfego passe por VMs personalizadas específicas, Soluções de Virtualização da Rede e balanceadores de carga usados pela implementação de VDC. Para assegurar que o tráfego gerado de VMs residentes no spoke transite para as soluções de virtualização corretas, um UDR precisa ser definido nas sub-redes do spoke configurando o endereço IP de front-end do balanceador de carga interno como o próximo salto. O balanceador de carga interno distribui o tráfego interno para as soluções de virtualização (pool de back-end do balanceador de carga).

[**Firewall do Azure**][AzFW] é um serviço de segurança de rede gerenciado e baseado em nuvem que protege seus recursos de Rede Virtual do Azure. É um firewall como serviço totalmente com estado com alta disponibilidade interna e escalabilidade de nuvem irrestrita. É possível criar, impor e registrar centralmente políticas de conectividade de rede e de aplicativo em assinaturas e redes virtuais. O Firewall do Azure usa um endereço IP público estático para seus recursos de rede virtual. Ele permite firewalls externos para identificar o tráfego proveniente da sua rede virtual. O serviço é totalmente integrado ao Azure Monitor para registro em log e análise.

[**Soluções de virtualização de rede**][NVA]. No hub, a rede de perímetro com acesso à Internet normalmente é gerenciada por meio de uma instância do Firewall do Azure ou de um farm de firewalls ou do firewall de aplicativo Web (WAF).

Linhas de negócios diferentes normalmente usam muitos aplicativos Web. Esses aplicativos tendem a sofrer de diversas vulnerabilidades e explorações em potencial. Os firewalls de aplicativos Web são um tipo especial de produto usado para detectar ataques contra aplicativos Web (HTTP/HTTPS) em mais profundidade que um firewall genérico. Em comparação à tecnologia de firewall tradicional, WAFs têm um conjunto de recursos específicos para proteger os servidores Web internos contra ameaças.

Um Firewall do Azure ou firewall NVA usam, ambos, um plano de administração comum, com um conjunto de regras de segurança para proteger as cargas de trabalho hospedadas nos spokes e controlar o acesso a redes locais. O Firewall do Azure tem escalabilidade integrada, enquanto os firewalls do NVA podem ser dimensionados manualmente por um balanceador de carga. Normalmente, um farm de firewall tem software menos especializado comparado a um WAF, mas tem um escopo de aplicação mais amplo para filtrar e inspecionar qualquer tipo de tráfego de entrada e saída. Se for usada uma abordagem de NVA, ela pode ser localizada e implantada no Azure Marketplace.

Use um conjunto de Firewalls do Azure (ou NVAs) para o tráfego originado na Internet e outro para o tráfego originado localmente. Usar apenas um conjunto de firewalls para ambos é um risco à segurança, uma vez que ele não oferece nenhuma segurança de perímetro entre os dois conjuntos de tráfego de rede. Usar camadas de firewall separadas reduz a complexidade de verificar as regras de segurança e deixa claro quais regras correspondem a quais solicitações de rede de entrada.

É recomendável que você use um conjunto de instâncias do Firewall do Azure ou NVAs para tráfego originado na Internet. Use outro para o tráfego originado localmente. Usar apenas um conjunto de firewalls para ambos é um risco à segurança, uma vez que ele não oferece nenhuma segurança de perímetro entre os dois conjuntos de tráfego de rede. Usar camadas de firewall separadas reduz a complexidade da verificação das regras de segurança e deixa claro quais regras correspondem a quais solicitações de rede de entrada.

O [**Azure Load Balancer**][ALB] oferece um serviço de Camada 4 (TCP, UDP) de alta disponibilidade, que pode distribuir o tráfego de entrada entre instâncias de serviço definidas em um conjunto com balanceamento de carga. O tráfego enviado ao balanceador de carga de pontos de extremidade de front-end (pontos de extremidade IP públicos ou pontos de extremidade IP privados) pode ser redistribuído com ou sem conversão de endereços para um pool de endereços IP de back-end (exemplos são Soluções de Virtualização de Rede ou VMs).

O Azure Load Balancer também pode investigar a integridade de várias instâncias de servidor e quando uma investigação falha em responder, o balanceador de carga para de enviar tráfego para a instância não íntegra. No VDC, um balanceador de carga externo é implantado no hub e nos spokes. No hub, o balanceador de carga é usado para rotear o tráfego com eficiência para os serviços nos spokes e, nestes, os balanceadores de carga são usados para gerenciar o tráfego de aplicativo.

O [**AFD**][AFD] (Azure Front Door) é a Plataforma de Aceleração de Aplicativos Web, o Balanceador de Carga HTTP Global, a Proteção de Aplicativos e a Rede de Distribuição de Conteúdo altamente disponíveis e escalonáveis da Microsoft. Em execução em mais de 100 locais com a Rede Global Edge da Microsoft, o AFD permite compilar, operar e escalar horizontalmente seu aplicativo Web dinâmico e o conteúdo estático. O AFD oferece a seu aplicativo desempenho do usuário final de alto nível, automação de manutenção de carimbo/regional unificada, automação de BCDR, informações unificadas de cliente/usuário, insights de serviço e caching. A plataforma oferece SLAs de desempenho, confiabilidade e suporte, certificações de conformidade e práticas de segurança auditáveis desenvolvidas, operadas e com suporte nativo pelo Azure.

[**Gateway de Aplicativo**][AppGW] O Gateway de Aplicativo do Microsoft Azure é uma solução de virtualização dedicada que fornece o ADC (controlador de entrega de aplicativos) como um serviço, oferecendo vários recursos de balanceamento de carga de camada 7 para o aplicativo. Ele permite que você otimize a produtividade do Web farm descarregando a terminação SSL com uso intensivo de CPU para o Gateway de Aplicativo. Ele também fornece outros recursos de roteamento de camada 7, incluindo distribuição round robin do tráfego de entrada, afinidade de sessão, roteamento com base no caminho de URL e a capacidade de hospedar vários sites por trás de um único Gateway de Aplicativo baseado em cookie. Um WAF (firewall do aplicativo Web) também é fornecido como parte da SKU do WAF do gateway de aplicativo. Essa SKU oferece proteção para aplicativos Web contra explorações e vulnerabilidades comuns da Web. O Gateway de Aplicativo pode ser configurado como um gateway voltado para a Internet, um gateway apenas interno ou uma combinação de ambos. 

[**IPs Públicos**][PIP]. Com alguns recursos do Azure, você pode associar pontos de extremidade de serviço a um endereço IP público para que seu recurso possa ser acessado pela Internet. Esse ponto de extremidade usa a conversão de endereços de rede (NAT) para rotear o tráfego para a porta e o endereço internos na rede virtual do Azure. Esse caminho é a principal rota para que o tráfego externo passe para dentro da rede virtual. Você pode configurar endereços IP públicos para determinar qual tráfego é passado para dentro e como e onde ele é convertido para a rede virtual.

[**Proteção contra DDoS do Azure Standard**][DDOS] fornece funcionalidades de mitigação adicionais à camada de [serviço Básica][DDOS] ajustadas especificamente para os recursos de Rede Virtual do Azure. A Proteção contra DDoS Standard é simples de habilitar e não exige nenhuma alteração no aplicativo. Políticas de proteção são ajustadas por meio do monitoramento de tráfego dedicado e algoritmos de aprendizado de máquina. As políticas são aplicadas a endereços IP públicos associados aos recursos implantados em redes virtuais. Os exemplos são instâncias do Azure Load Balancer, Gateway de Aplicativo do Azure e Azure Service Fabric. A telemetria em tempo real está disponível por meio de exibições do Azure Monitor durante um ataque e para o histórico. Proteções de camada de aplicativo podem ser adicionadas por meio do firewall do aplicativo Web do Gateway de Aplicativo do Azure. A proteção é fornecida para endereços IP públicos IPv4 do Azure.

#### <a name="component-type-monitoring"></a>Tipo de componente: Monitoramento

Componentes de monitoramento oferecem visibilidade e alertas de todos os outros tipos de componentes. Todas as equipes devem ter acesso ao monitoramento para os componentes e serviços aos quais elas têm acesso. Se você tiver equipes de operações ou suporte técnico centralizadas, elas precisarão ter acesso integrado aos dados fornecidos por esses componentes.

O Azure oferece diferentes tipos de serviços de registro em log e monitoramento para controlar o comportamento de recursos hospedados do Azure. Governança e controle de cargas de trabalho no Azure baseiam-se não apenas na coleta de dados de log, mas também na capacidade de disparar ações com base em eventos relatados específicos.

[**Azure Monitor**][Monitor]. O Azure inclui vários serviços que individualmente executam uma função ou tarefa específica no espaço de monitoramento. Juntos, esses serviços oferecem uma solução abrangente para coletar, analisar e agir na telemetria do seu aplicativo e os recursos do Azure compatíveis com eles. Eles também podem trabalhar para monitorar recursos críticos locais para fornecer um ambiente de monitoramento híbrido. Compreender as ferramentas e os dados que estão disponíveis é a primeira etapa no desenvolvimento de uma estratégia de monitoramento completa para seu aplicativo.

Há dois tipos principais de logs no Azure:

-   O [Log de Atividades do Azure][ActLog], conhecido anteriormente como **Log Operacional**, fornece informações sobre as operações que foram executadas em recursos na assinatura do Azure. Esses logs relatam os eventos de plano de controle para suas assinaturas. Todos os recursos do Azure geram logs de auditoria.

-   Os [logs de diagnóstico do Azure Monitor][DiagLog] são logs gerados por um recurso que fornece dados avançados e frequentes sobre a operação desse recurso. O conteúdo desses logs varia de acordo com o tipo de recurso.

[![9]][9]

É importante controlar os logs de NSG, sobretudo estas informações:

-   Os [Logs de eventos][NSGLog] fornecem informações sobre quais regras de NSG são aplicadas às VMs e funções de instância com base no endereço MAC.
-   Os [Logs do contador][NSGLog] controlam quantas vezes cada regra NSG foi executada para negar ou permitir o tráfego.

Todos os logs podem ser armazenados nas contas de armazenamento do Azure para fins de auditoria, análise estática ou backup. Quando você armazena os logs em uma conta de armazenamento do Azure, os clientes podem usar diferentes tipos de estruturas para recuperar, preparar, analisar e visualizar esses dados para relatar o status e a integridade dos recursos de nuvem. 

Grandes empresas já devem ter adquirido uma estrutura padrão para monitorar sistemas locais. Elas podem estender essa estrutura para integrar logs gerados por implantações de nuvem. Usando o [Azure Log Analytics](/azure/log-analytics/log-analytics-queries), as organizações podem manter todo o registro em log na nuvem. O Log Analytics é implementado como um serviço baseado em nuvem. Portanto, você pode colocá-lo em funcionamento rapidamente com investimento mínimo nos serviços de infraestrutura. O Log Analytics também integrar-se a componentes do System Center, como o System Center Operations Manager, para estender seus investimentos atuais em gerenciamento para a nuvem. 

O Log Analytics é um serviço no Azure que ajuda a coletar, correlacionar, pesquisar e agir quanto a dados de desempenho e log gerados por sistemas operacionais, aplicativos e componentes de infraestrutura de nuvem. Ele dá aos clientes informações operacionais em tempo real usando painéis de pesquisa e personalizados integrados para analisar todos os registros em todas as cargas de trabalho em uma implementação de VDC.

O [Observador de Rede do Azure][NetWatch] fornece ferramentas para monitorar, diagnosticar, exibir métricas e ativar ou desativar os registros de recursos em uma rede virtual do Azure. É um serviço multifacetado que permite as seguintes funcionalidades e muito mais:
-    Monitorar a comunicação entre uma máquina virtual e um ponto de extremidade.
-    Exibir recursos em uma rede virtual e suas relações.
-    Diagnosticar problemas de filtragem de tráfego de erro para ou de uma VM.
-    Diagnosticar problemas de roteamento de rede de uma VM.
-    Diagnosticar conexões de saída de uma VM.
-    Capturar pacotes para e de uma VM.
-    Diagnosticar problemas com conexões e gateway de rede virtual do Azure.
-    Determinar as latências relativas entre regiões do Azure e provedores de serviços de Internet.
-    Exibir regras de segurança para uma interface de rede.
-    Exibir métricas de rede.
-    Analisar o tráfego de ou para um grupo de segurança de rede.
-    Exibir logs de diagnóstico para recursos de rede.

A solução [Monitor de Desempenho de Rede][NPM] dentro do Operations Management Suite pode fornecer informações detalhadas de rede ponta a ponta. Essas informações incluem uma exibição única de redes do Azure e redes locais. A solução tem monitores específicos para ExpressRoute e serviços públicos.

#### <a name="component-type-workloads"></a>Tipo de componente: Cargas de trabalho

Componentes de carga de trabalho são o local em que seus aplicativos reais e serviços residem. Também é o ponto em que as equipes de desenvolvimento de aplicativo passam a maior parte do tempo.

As possibilidades de carga de trabalho são infinitas. A seguir estão apenas alguns dos tipos de carga de trabalho possíveis:

**Aplicativos de LOB Internos**: Aplicativos de linha de negócios são aplicativos de computador críticos para a operação contínua de uma empresa. Aplicativos de LOB têm algumas características em comum:

-   **Interativo** por natureza. Os dados são inseridos, e os resultados ou os relatórios são retornados.
-   **Controlado por dados**&mdash;dados intensivos com acesso frequente aos bancos de dados ou outro armazenamento.
-   **Integrado**&mdash;oferecem integração a outros sistemas dentro ou fora da organização.

**Sites voltados para o cliente (voltados para a Internet ou internos)**: A maioria dos aplicativos que interagem com a Internet são sites da Web. O Azure oferece a capacidade de executar um site em uma VM IaaS ou de um site [Aplicativos Web do Azure][WebApps] (PaaS). Os Aplicativos Web do Azure dão suporte à integração com VNETs que permitem a implantação dos aplicativos Web na zona de rede de um spoke. Os sites da Web internos não precisam expor um ponto de extremidade de Internet público porque os recursos podem ser acessados por meio de endereços privados roteáveis fora da Internet a partir da rede virtual privada.

**Big Data/Análise**: quando os dados precisam ser escalados verticalmente para um volume muito grande, os bancos de dados podem não ser escalados de modo adequado. A tecnologia Hadoop oferece um sistema para executar consultas distribuídas em paralelo em um grande número de nós. Os clientes têm a opção de executar cargas de trabalho de dados em VMs de IaaS ou PaaS ([HDInsight][HDI]). O HDInsight dá suporte à implantação em uma VNet baseada em locais, podendo ser implantada em um cluster de um spoke do VDC.

**Eventos e mensagens**: [Os Hubs de Eventos do Azure][EventHubs] são um serviço de ingestão de telemetria de hiperescala que coleta, transforma e armazena milhões de eventos. Como uma plataforma de streaming distribuída, oferece baixa latência e tempo de retenção configurável, permitindo a ingestão de grandes quantidades de telemetria no Azure e a leitura de dados de vários aplicativos. Com Hubs de Evento, um único fluxo pode dar suporte a pipelines tanto em tempo real quanto em lote.

Você pode implementar um serviço entre aplicativos e serviços por meio de mensagens de nuvem altamente confiável usando o [Barramento de Serviço do Azure][ServiceBus]. Ele oferece um sistema de mensagens agenciado e assíncrono entre o cliente e o servidor, recursos de mensagens PEPS (primeiro a entrar, primeiro a sair) e de publicação e assinatura.

[![10]][10]

### <a name="making-the-vdc-highly-available-multiple-vdcs"></a>Tornando o VDC altamente disponível: vários VDCs

Até agora, este artigo concentrou-se no projeto de um único VDC, descrevendo os componentes básicos e a arquitetura que contribuem para sua resiliência. Os recursos do Azure como Azure Load Balancer, NVAs, conjuntos de disponibilidade, conjuntos de dimensionamento e outros contribuem para um sistema que permite que você crie níveis sólidos de SLA em seus serviços de produção.

No entanto, como um único VDC geralmente é implementado em uma única região, ele pode ficar vulnerável a grandes interrupções que venham a afetar toda a região. Os clientes que exigem SLAs altos precisam proteger os serviços por meio de implantações do mesmo projeto em duas (ou mais) implementações de VDC colocadas em regiões diferentes.

Além das questões de SLA, há vários cenários comuns em que a implantação de diversos VDCs faz sentido:

-   Presença regional ou global.
-   Recuperação de desastre.
-   Um mecanismo para desviar o tráfego entre datacenters.

#### <a name="regionalglobal-presence"></a>Presença regional/global

Os datacenters do Azure estão presentes em várias regiões do mundo. Ao selecionar vários datacenters do Azure, os clientes precisam considerar dois fatores relacionados: distâncias geográficas e latência. Para oferecer a melhor experiência do usuário, avalie a distância geográfica entre as implementações de VDC e a distância entre cada implementação de VDC e os usuários finais.

A região que hospeda as implementações de VDC precisa estar em conformidade com os requisitos normativos estabelecidos pelo ordenamento jurídico que regula o funcionamento de sua organização.

#### <a name="disaster-recovery"></a>Recuperação de desastre

O design de um plano de recuperação de desastre depende dos tipos de cargas de trabalho e da capacidade de sincronizar o estado dessas cargas de trabalho entre diferentes implementações de VDC. Idealmente, a maioria dos clientes quer um mecanismo de failover rápido, e isso pode exigir a sincronização de dados de aplicativo entre implantações que executam várias implementações de VDC. No entanto, durante a criação de planos de recuperação de desastre, é importante considerar que a maioria dos aplicativos são afetados pela latência que pode ser causada por essa sincronização de dados.

A sincronização e o monitoramento de pulsação dos aplicativos em diferentes implementações de VDC exigem que eles se comuniquem pela rede. Duas implementações de VDC em regiões diferentes podem ser conectadas da seguinte maneira:

-   Emparelhamento VNET - O Emparelhamento VNET pode conectar hubs entre regiões
-   Emparelhamento privado ExpressRoute quando os hubs em cada implementação de VDC estão conectados ao mesmo circuito do ExpressRoute
-   Vários circuitos do ExpressRoute conectados via backbone corporativo e suas várias implementações de VDC conectadas aos circuitos do ExpressRoute
-   Conexões VPN Site a Site entre a zona de hub de suas implementações de VDC em cada região do Azure

Normalmente, as conexões de Emparelhamento VNET ou de ExpressRoute são o tipo preferencial de conectividade de rede devido à maior largura de banda e aos níveis de latência consistentes ao transitar pelo backbone da Microsoft.

Recomendamos que os clientes executem testes de qualificação de rede para verificar a latência e a largura de banda dessas conexões e escolham a replicação de dados adequada, síncrona ou assíncrona, com base no resultado. Também é importante avaliar esses resultados tendo em vista o RTO (objetivo de tempo de recuperação) ideal.

#### <a name="disaster-recovery-diverting-traffic-from-one-region-to-another"></a>Recuperação de desastre: desviando tráfego de uma região para outra

O [Gerenciador de Tráfego do Azure][TM] verifica periodicamente a integridade do serviço de pontos de extremidade públicos em diferentes implementações de VDC e, em caso de falha desses pontos de extremidade, ele roteia automaticamente para o VDC secundário usando o DNS (Sistema de Nomes de Domínio). 

Como ele usa o DNS, o Gerenciador de Tráfego serve apenas para uso com pontos de extremidade públicos do Azure.  O serviço normalmente é usado para controlar ou desviar tráfego para VMs do Azure e aplicativos Web na instância íntegra de uma implementação de VDC. O Gerenciador de Tráfego é resiliente mesmo em caso de falha em uma região inteira do Azure e pode controlar a distribuição do tráfego de usuário para pontos de extremidade de serviço em VDCs diferentes com base em vários critérios. Por exemplo, na falha de um serviço em uma implementação específica do VDC ou ao escolher a implementação de VDC com a menor latência de rede.

### <a name="summary"></a>Resumo

O Datacenter Virtual é uma abordagem de migração do datacenter para criar uma arquitetura escalonável no Azure que maximiza o uso de recursos de nuvem, reduz custos e simplifica a governança do sistema. O VDC baseia-se em uma topologia de rede hub-spoke, fornecendo serviços compartilhados comuns no hub e permitindo aplicativos/cargas de trabalho específicos nos spokes. O VDC também corresponde à estrutura de funções da empresa, em que departamentos diferentes, como TI Central, DevOps, Operação e Manutenção, trabalham juntos durante a execução de suas funções específicas. O VDC cumpre os requisitos de uma migração "Lift and Shift", mas também oferece muitas vantagens em implantações de nuvem nativas.

## <a name="references"></a>Referências

Os seguintes recursos foram discutidos neste documento. Siga os links para saber mais.

| | | |
|-|-|-|
|Recursos de rede|Balanceamento de Carga|Conectividade|
|[Redes Virtuais do Azure][VNet]</br>[Grupos de segurança de rede][NSG]</br>[Logs do NSG][NSGLog]</br>[Roteamento Definido pelo Usuário][UDR]</br>[Soluções de Virtualização de Rede][NVA]</br>[Endereços IP Públicos][PIP]</br>[Azure DDOS][DDOS]</br>[Firewall do Azure][AzFW]</br>[DNS do Azure][DNS]|[Azure Front Door][AFD]</br>[Azure Load Balancer (L3) ][ALB]</br>[Gateway de Aplicativo (L7) ][AppGW]</br>[Firewall do Aplicativo Web][WAF]</br>[Gerenciador de Tráfego do Azure][TM]</br></br></br></br></br> |[Emparelhamento VNet][VNetPeering]</br>[Rede Privada Virtual][VPN]</br>[WAN Virtual][vWAN]</br>[ExpressRoute][ExR]</br>[ExpressRoute Direct][ExRD]</br></br></br></br></br>
|Identidade</br>|Monitoramento</br>|Práticas Recomendadas</br>|
|[Azure Active Directory][AAD]</br>[Autenticação Multifator][MFA]</br>[Controles de Acesso Baseados em Função][RBAC]</br>[Funções padrão do Azure AD][Roles]</br></br></br> |[Observador de Rede][NetWatch]</br>[Azure Monitor][Monitor]</br>[Logs de Atividade][ActLog]</br>[Logs de Diagnóstico][DiagLog]</br>[Microsoft Operations Management Suite][OMS]</br>[Monitor de Desempenho de Rede][NPM]|[Práticas Recomendadas de Redes de Perímetro][DMZ]</br>[Gerenciamento de Assinaturas][SubMgmt]</br>[Gerenciamento de Grupo de Recursos][RGMgmt]</br>[Limites de Assinatura do Azure][Limits] </br></br></br>|
|Outros serviços do Azure|
|[Aplicativos Web do Azure][WebApps]</br>[HDInsights (Hadoop) ][HDI]</br>[Hubs de Eventos][EventHubs]</br>[Barramento de Serviço][ServiceBus]|

## <a name="next-steps"></a>Próximas etapas

 - Explore o [Emparelhamento de VNet][VNetPeering], a tecnologia de base para os designs de hub e spoke do VDC
 - Implemente o [Azure AD][AAD] para começar a usar a exploração de [RBAC][RBAC]
 - Desenvolva um modelo de gerenciamento de assinaturas e recursos e um modelo de RBAC para atender à estrutura, aos requisitos e às políticas da sua organização. A atividade mais importante é o planejamento. Tanto quanto possível, planeje reorganizações, fusões, novas linhas de produto etc. <!--Image References-->
[0]: ./images/networking-redundant-equipment.png "Exemplos de sobreposição de componente" 
[1]: ./images/networking-vdc-high-level.png "Exemplo de alto nível do VDC de hub e spoke"
[2]: ./images/networking-hub-spokes-cluster.png "Cluster de hubs e spokes"
[3]: ./images/networking-spoke-to-spoke.png "Spoke-to-spoke"
[4]: ./images/networking-vdc-block-level-diagram.png "Diagrama no nível do bloco do VDC"
[5]: ./images/networking-users-groups-subsciptions.png "Usuários, grupos, assinaturas e projetos"
[6]: ./images/networking-infrastructure-high-level.png "Diagrama de alto nível de infraestrutura"
[7]: ./images/networking-highlevel-perimeter-networks.png "Diagrama de alto nível de infraestrutura"
[8]: ./images/networking-vnet-peering-perimeter-neworks.png "Redes de perímetro e de emparelhamento VNet"
[9]: ./images/networking-high-level-diagram-monitoring.png "Diagrama de alto nível para Monitoramento"
[10]: ./images/networking-high-level-workloads.png "Diagrama de alto nível para Carga de Trabalho"

<!--Link References-->
[Limits]: /azure/azure-subscription-service-limits
[Roles]: /azure/role-based-access-control/built-in-roles
[VNet]: /azure/virtual-network/virtual-networks-overview
[NSG]: /azure/virtual-network/virtual-networks-nsg
[DNS]: /azure/dns/dns-overview
[PrivateDNS]: /azure/dns/private-dns-overview
[VNetPeering]: /azure/virtual-network/virtual-network-peering-overview 
[UDR]: /azure/virtual-network/virtual-networks-udr-overview 
[RBAC]: /azure/role-based-access-control/overview
[MFA]: /azure/multi-factor-authentication/multi-factor-authentication
[AAD]: /azure/active-directory/active-directory-whatis
[VPN]: /azure/vpn-gateway/vpn-gateway-about-vpngateways 
[ExR]: /azure/expressroute/expressroute-introduction
[ExRD]: /azure/expressroute/expressroute-erdirect-about
[vWAN]: /azure/virtual-wan/virtual-wan-about
[NVA]: /azure/architecture/reference-architectures/dmz/nva-ha
[AzFW]: /azure/firewall/overview
[SubMgmt]: /azure/architecture/cloud-adoption/appendix/azure-scaffold 
[RGMgmt]: /azure/azure-resource-manager/resource-group-overview
[DMZ]: /azure/best-practices-network-security
[ALB]: /azure/load-balancer/load-balancer-overview
[DDOS]: /azure/virtual-network/ddos-protection-overview
[PIP]: /azure/virtual-network/resource-groups-networking#public-ip-address
[AFD]: /azure/frontdoor/front-door-overview
[AppGW]: /azure/application-gateway/application-gateway-introduction
[WAF]: /azure/application-gateway/application-gateway-web-application-firewall-overview
[Monitor]: /azure/monitoring-and-diagnostics/
[ActLog]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs 
[DiagLog]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs
[NSGLog]: /azure/virtual-network/virtual-network-nsg-manage-log
[OMS]: /azure/operations-management-suite/operations-management-suite-overview
[NPM]: /azure/log-analytics/log-analytics-network-performance-monitor
[NetWatch]: /azure/network-watcher/network-watcher-monitoring-overview
[WebApps]: /azure/app-service/
[HDI]: /azure/hdinsight/hdinsight-hadoop-introduction
[EventHubs]: /azure/event-hubs/event-hubs-what-is-event-hubs 
[ServiceBus]: /azure/service-bus-messaging/service-bus-messaging-overview
[TM]: /azure/traffic-manager/traffic-manager-overview
