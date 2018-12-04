---
title: 'Datacenter virtual do Azure: uma perspectiva de rede'
description: Aprenda a criar seu datacenter virtual no Azure
author: tracsman
manager: rossort
tags: azure-resource-manager
ms.service: virtual-network
ms.date: 09/24/2018
ms.author: jonor
ms.openlocfilehash: b10930ac6d6458872d8b626825d21bd0bed2b807
ms.sourcegitcommit: 16bc6a91b6b9565ca3bcc72d6eb27c2c4ae935e4
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/28/2018
ms.locfileid: "52550454"
---
# <a name="azure-virtual-datacenter-a-network-perspective"></a>Datacenter virtual do Azure: uma perspectiva de rede

## <a name="overview"></a>Visão geral
Migrar aplicativos locais para o Azure, mesmo sem quaisquer alterações significativas, permite que as organizações se beneficiem de uma infraestrutura segura e econômica. Essa abordagem é conhecida como **lift-and-shift**. Para aproveitar ao máximo a agilidade possível com a computação em nuvem, as empresas devem evoluir suas arquiteturas para aproveitarem os recursos do Azure. O Microsoft Azure oferece serviços e infraestrutura em hiperescala, recursos e confiabilidade de nível empresarial e várias opções de conectividade híbrida. 

Os clientes podem optar por acessar esses serviços de nuvem pela Internet ou com o Azure ExpressRoute, que fornece conectividade de rede privada. Com a plataforma Microsoft Azure, os clientes podem estender facilmente sua infraestrutura para a nuvem e criar arquiteturas com várias camadas. Os parceiros da Microsoft também fornecem recursos avançados, oferecendo serviços de segurança e soluções de virtualização otimizados para execução no Azure.

Este artigo apresenta uma visão geral de padrões e designs para resolver as preocupações de segurança, desempenho e escala de arquitetura que muitos clientes enfrentam quando pensam em uma transição em massa para a nuvem. Uma visão geral de como ajustar as diferentes funções de TI organizacionais ao gerenciamento e à governança do sistema também é discutida. A ênfase é dada a requisitos de segurança e à otimização de custos.

## <a name="what-is-a-virtual-datacenter"></a>O que é um datacenter virtual?
As soluções de nuvem eram projetadas para hospedar aplicativos únicos relativamente isolados no espectro público. Essa abordagem funcionou bem por alguns anos. Depois, os benefícios das soluções de nuvem tornaram-se evidentes, e cargas de trabalho em grande escala passaram a ser hospedadas na nuvem. Lidar com questões relacionadas à segurança, confiabilidade, desempenho e custos com implantações se tornou fundamental durante todo o ciclo de vida do serviço de nuvem.

O diagrama de implantação de nuvem a seguir mostra um exemplo de falha de segurança na **caixa vermelha**. A **caixa amarela** mostra espaço para soluções de virtualização de rede de otimização em cargas de trabalho.

[![0]][0]

O Datacenter virtual (VDC) nasceu da necessidade de escalonar o suporte a cargas de trabalho empresariais. Ele também lida com os problemas resultantes do suporte a aplicativos em larga escala na nuvem pública.

O VDC não é composto apenas pelas cargas de trabalho de aplicativo na nuvem. Ele também engloba rede, segurança, gerenciamento e infraestrutura. DNS e serviços de diretório são bons exemplos de VDC. Geralmente, ele fornece uma conexão privada de volta para uma rede ou datacenter local. Como cada vez mais cargas de trabalho são movidas para o Azure, é importante pensar na infraestrutura de suporte e nos objetos em que essas cargas de trabalho são colocadas. Analise atentamente o modo como os recursos devem ser estruturados para evitar a proliferação de centenas de **ilhas de carga de trabalho** que devem ser gerenciadas separadamente, com fluxo de dados, modelos de segurança e desafios de conformidade independentes.

O datacenter virtual é uma coleção de entidades separadas, porém relacionadas, com infraestrutura, recursos e funções de suporte em comum. Ao exibir suas cargas de trabalho como um VDC integrado, você pode reduzir os custos com economias de escala e com otimização de segurança por meio de centralização de componentes e do fluxo de dados. Você também obtém operações, gerenciamento e auditorias de conformidade com mais facilidade.

> [!NOTE]  
> É importante reconhecer que o VDC **não** é um produto separado do Azure. É a combinação de vários recursos e capacidades para atender às suas necessidades específicas. O VDC é uma maneira de pensar sobre suas cargas de trabalho e o uso do Azure para maximizar os recursos e as habilidades na nuvem. O datacenter virtual é, portanto, uma abordagem modular sobre como criar serviços de TI no Azure, respeitando funções e responsabilidades organizacionais.

O VDC pode ajudar as empresas a colocarem as cargas de trabalho e os aplicativos no Azure para os seguintes cenários:

-   Hospedagem de várias cargas de trabalho relacionadas.
-   Migração de cargas de trabalho de um ambiente local para o Azure.
-   Implementação de segurança compartilhada ou centralizada e requisitos de acesso entre cargas de trabalho.
-   Como combinar TI centralizada e DevOps apropriadamente para uma grande empresa.

O segredo para aproveitar as vantagens do VDC é uma topologia centralizada, hub-spoke, com uma mistura de recursos do Azure: 

- [Rede Virtual do Azure][VNet]. 
- [Grupos de segurança de rede (NSGs)][NSG].
- [Emparelhamento de rede virtual][VNetPeering]. 
- [UDRs (Rotas Definidas pelo Usuário)][UDR].
- Serviços de identificação do Azure com o [controle de acesso baseado em função (RBAC)][RBAC]. 
- Opcionalmente, [Firewall do Azure][AzFW], [DNS do Azure][DNS], [Azure Front Door] [ AFD] e [WAN Virtual do Azure][vWAN].

## <a name="who-should-implement-a-virtual-datacenter"></a>Quem deve implementar um datacenter virtual?
Qualquer cliente do Azure que precise mover mais do que algumas cargas de trabalho no Azure pode se beneficiar do uso de recursos comuns. Dependendo do tamanho, até mesmo aplicativos únicos podem aproveitar o uso de padrões e componentes empregados para criar o VDC.

Se sua organização tiver TI, rede e segurança centralizadas ou um departamento ou equipe de conformidade, o VDC poderá ajudar a impor pontos da política e a separação de responsabilidades. Ele também garante a uniformidade dos componentes comuns subjacentes e concede às equipes de aplicativo a liberdade e o controle apropriados às suas necessidades.

As organizações que procuram o Azure DevOps podem utilizar os conceitos do VDC para fornecer grupos autorizados de recursos do Azure e garantir que eles tenham controle total dentro desse grupo. Os grupos são assinaturas ou grupos de recursos em uma assinatura comum. Mas os limites de segurança e de rede mantêm a conformidade definida por uma política centralizada em uma rede virtual e em um grupo de recursos de hub.

## <a name="considerations-when-you-implement-a-virtual-datacenter"></a>Considerações ao implementar um datacenter virtual
Quando você projeta seu VDC, há várias questões importantes a considerar:

-   Serviços de identidade e diretório.
-   Infraestrutura de segurança.
-   Conectividade com a nuvem.
-   Conectividade na nuvem.

### <a name="identity-and-directory-services"></a>Serviços de identidade e diretório
Serviços de identidade e diretório são um aspecto fundamental de todos os datacenters, sejam locais ou na nuvem. A identidade está relacionada a todos os aspectos de acesso e a autorização aos serviços no VDC. Para garantir que somente usuários e processos autorizados acessem seus recursos e a conta do Azure, o Azure usa vários tipos de credenciais para autenticação. Eles incluem senhas para acessar a conta do Azure, chaves de criptografia, assinaturas digitais e certificados. 

[Autenticação Multifator do Azure][MFA] é uma camada adicional de segurança para acessar os serviços do Azure. Ela fornece autenticação forte com uma variedade de opções de verificação fácil. Os clientes podem escolher o método preferido. As opções incluem uma chamada telefônica, mensagem de texto ou notificação de aplicativo móvel. 

Qualquer grande empresa precisa definir um processo de gerenciamento de identidades que descreva o gerenciamento de identidades individuais e suas respectivas autenticação, autorização, funções e privilégios dentro ou no VDC. Os objetivos deste processo são aumentar a segurança e a produtividade e diminuir o custo, o tempo de inatividade e as tarefas manuais repetitivas.

As empresas e organizações podem exigir um conjunto exigente de serviços para diferentes linhas de negócios. E os funcionários geralmente têm diferentes funções quando envolvidos em projetos distintos. O VDC requer uma boa cooperação entre diferentes equipes, cada uma com definições de função específicas, para que os sistemas sejam executados com boa governança. A matriz de responsabilidades, acesso e direitos pode ser complexa. O gerenciamento de identidades no VDC é implementado por meio do [Azure Active Directory (Azure AD)][AAD] e do RBAC (Controle de acesso baseado em função).

Um serviço de diretório é uma infraestrutura de informações compartilhadas que localiza, gerencia, administra e organiza itens cotidianos e recursos de rede. Esses recursos podem incluir volumes, pastas, arquivos, impressoras, usuários, grupos, dispositivos e outros objetos. Cada recurso na rede é considerado um objeto pelo servidor de diretório. Informações sobre um recurso são armazenadas como uma coleção de atributos associados a tal recurso ou objeto.

Todos os serviços comerciais do Microsoft Online dependem do Azure Active Directory (Azure AD) para entrada e outras necessidades de identidade. O Azure Active Directory é uma solução de nuvem de gerenciamento de acesso e identidade abrangente e altamente disponível que combina os serviços principais de diretório, governança avançada de identidades e gerenciamento do acesso de aplicativos. O Azure AD pode ser integrado ao Active Directory local para habilitar o logon único para todos os aplicativos hospedados localmente e baseados em nuvem (local). Os atributos do usuário do Active Directory local podem ser sincronizados automaticamente com o Azure AD.

Não é necessário haver um único administrador global para atribuir todas as permissões na implementação de um VDC. Em vez disso, cada departamento específico ou grupo de usuários ou serviços no serviço de diretório pode ter as permissões necessárias para gerenciar seus próprios recursos no VDC. Estruturar permissões requer balanceamento. Um número excessivo de permissões impede o desempenho eficiente. Por outro lado, um número muito pequeno de permissões ou permissões flexíveis aumentam os riscos de segurança. O RBAC (Controle de acesso baseado em função) do Azure ajuda a resolver esse problema oferecendo um gerenciamento de acesso refinado para recursos VDC.

#### <a name="security-infrastructure"></a>Infraestrutura de segurança
Esta infraestrutura de segurança refere-se à segregação de tráfego no segmento de rede virtual específico do VDC e a como controlar os fluxos de entrada e saída em todo o VDC. O Azure baseia-se na arquitetura multilocatário que impede o tráfego não autorizado e não intencional entre as implantações. Ele usa isolamento de rede virtual, listas de controle de acesso (ACLs), políticas de fluxo de tráfego, filtros IP e balanceadores de carga. NAT (conversão de endereços de rede) separa o tráfego de rede interno do tráfego externo.

A malha do Azure aloca recursos de infraestrutura a cargas de trabalho de locatário e gerencia comunicações para e de VMs (máquinas virtuais). O hipervisor do Azure impõe a separação de memória e processo entre VMs e encaminha com segurança o tráfego de rede a locatários do sistema operacional convidado.

#### <a name="connectivity-to-the-cloud"></a>Conectividade com a nuvem
A implementação de um VDC precisa de conectividade com redes externas para oferecer serviços a clientes, parceiros e usuários internos. Isso geralmente necessita de conectividade não apenas com a Internet, mas também com datacenters e redes locais.

Os clientes controlam a quais serviços eles têm acesso e quais ficam acessíveis pela Internet pública usando o [Firewall do Azure][AzFW] ou outros tipos de dispositivos de rede virtual (NVAs), políticas de roteamento personalizadas, usando [rotas definidas pelo usuário][UDR], assim como filtragem de rede, usando [grupos de segurança de rede][NSG]. É recomendável que todos os recursos voltados para Internet também sejam protegidos pela [Proteção contra DDoS do Azure Standard][DDOS].

As empresas geralmente precisam conectar suas implementações de VDC a datacenters locais ou outros recursos. A conectividade entre o Azure e redes locais é, portanto, um aspecto essencial ao projetar uma arquitetura eficaz. As empresas têm duas maneiras diferentes de criar interconexões entre o VDC e o local no Azure: trânsito pela Internet ou por conexões diretas privadas.

Uma [VPN site a site do Azure][ VPN] conecta a rede local de uma empresa à sua implementação de VDC pela Internet pública. O tráfego via VPN é criptografado com túnel IPSec e IKE. Uma conexão site a site do Azure é flexível e rápida de criar. Esta VPN não requer nenhuma compra adicional, pois todas as conexões são feitas pela Internet.

Para um grande número de conexões VPN, a [WAN Virtual do Azure][vWAN] é um serviço de rede que fornece conectividade otimizada e automatizada entre branches por meio do Azure. Com a WAN Virtual, você pode conectar e configurar dispositivos de branch para se comunicar com o Azure. Essa conexão pode ser feita manualmente ou usando dispositivos de provedor preferencial por meio de um parceiro de WAN Virtual. Os dispositivos de provedor preferencial facilitam o uso, simplificam a conectividade e o gerenciamento da configuração. O painel interno da WAN do Azure fornece informações imediatas para solução de problemas que podem economizar tempo. E o painel fornece uma maneira fácil de exibir a conectividade de site a site em larga escala.

O [ExpressRoute][ExR], um serviço de conectividade do Azure, permite conexões privadas entre a rede local da empresa e sua implementação de VDC. Ele oferece maior segurança, confiabilidade e velocidades mais altas, de até 10 Gbps, além de latência consistente. O ExpressRoute é útil para as implementações de VDCs, uma vez que os clientes do ExpressRoute aproveitam os benefícios das regras de conformidade associadas às conexões privadas. O [ExpressRoute Direct][ExRD] conecta você diretamente aos roteadores da Microsoft a 100 Gbps para os clientes com maiores necessidades de largura de banda.

Implantar conexões do ExpressRoute envolve normalmente a interação com um provedor de serviços do ExpressRoute. Para os clientes que precisam iniciar rapidamente, é comum usar inicialmente o VPN site a site para estabelecer uma conectividade entre o VDC e os recursos locais. Em seguida, é feita a migração para a conexão do ExpressRoute quando a interconexão física com seu provedor de serviço é concluída.

#### <a name="connectivity-within-the-cloud"></a>Conectividade na nuvem
[Redes virtuais][VNet] e [emparelhamento de redes virtuais][VNetPeering] são os serviços básicos de conectividade de rede dentro do VDC. Uma rede virtual garante um limite natural de isolamento para os recursos do VDC. E o emparelhamento de rede virtual permite a intercomunicação entre diferentes redes virtuais na mesma região do Azure ou até mesmo entre regiões distintas. O controle de tráfego dentro de uma rede virtual e entre redes virtuais precisa cumprir um conjunto de regras de segurança especificado por meio de listas de controle de acesso. Confira as informações sobre [grupos de segurança de rede][NSG], [(NVAs) soluções de virtualização de rede][NVA] e [tabelas de rotas personalizadas definidas pelo usuário][UDR].

## <a name="virtual-datacenter-overview"></a>Visão geral do datacenter virtual

### <a name="topology"></a>Topologia
**Hub e spoke** é um modelo usado para estender um datacenter virtual em uma única região do Azure.

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
No Azure, cada componente, seja qual for o tipo, é implantado em uma assinatura do Azure. O isolamento dos componentes do Azure em diferentes assinaturas do Azure pode atender aos requisitos de diferentes linhas de negócios. Um exemplo é a configuração de níveis diferenciados de acesso e autorização.

Uma única implementação de VDC pode escalar verticalmente para um grande número de spokes. Mas, assim como acontece com todos os sistemas de TI, há limites de plataformas. A implantação de hub está limitada a uma assinatura específica do Azure, que tem restrições e limites. Um exemplo é um número máximo de emparelhamentos de rede virtual. Para obter informações detalhadas, consulte [Assinatura do Azure e limites de serviço, cotas e restrições][Limits]. Em casos em que os limites podem ser um problema, a arquitetura pode aumentar ainda mais. Estenda o modelo de spokes com hub único para um cluster de hubs e spokes. Interconecte vários hubs em uma ou mais regiões do Azure usando emparelhamento de rede virtual, ExpressRoute, WAN Virtual ou VPN site a site.

[![2]][2]

A introdução de vários hubs aumenta o esforço de gerenciamento e o custo do sistema. Isso só seria justificado pela escalabilidade, como limites do sistema ou redundância e replicação regional, como o desempenho do usuário final ou a recuperação de desastres. Em cenários que exigem vários hubs, todos os hubs devem buscar oferecer o mesmo conjunto de serviços para facilidade operacional.

#### <a name="interconnection-between-spokes"></a>Interconexão entre spokes
Dentro de um único spoke, é possível implementar cargas de trabalho complexas de várias camadas. Você pode implementar configurações de várias camadas usando uma sub-rede para cada camada na mesma rede virtual. Filtre os fluxos usando NSGs.

Um arquiteto talvez queira implantar uma carga de trabalho de várias camadas em várias redes virtuais. Com um emparelhamento de redes virtuais, os spokes podem se conectar a outros spokes no mesmo hub ou em hubs diferentes. Um exemplo típico desse cenário é o caso em que os servidores de processamento do aplicativo estão em um spoke ou rede virtual. O banco de dados é implantado em um spoke ou em uma rede virtual diferente. Nesse caso, é fácil interconectar os spokes ao emparelhamento de redes virtuais e, assim, evitar trânsito pelo hub. Um exame cuidadoso de arquitetura e segurança deve ser executado para garantir que ignorar o hub não ignorará pontos de auditoria ou segurança importantes que possam existir somente no hub.

[![3]][3]

Os spokes também podem ser interconectados a um spoke que atue como um hub. Essa abordagem cria uma hierarquia de dois níveis: o spoke no nível mais alto (nível 0) torna-se o hub dos spokes inferiores (nível 1) da hierarquia. Os spokes da implementação de um VDC precisam encaminhar o tráfego para o hub central para alcançarem a Internet ou a rede local. Uma arquitetura com dois níveis de hub apresenta roteamento complexo que remove os benefícios de uma relação de hub-spoke simples.

Embora o Azure permita topologias complexas, dois princípios mais importantes do conceito de VDC são a repetição e a simplicidade. Para minimizar o esforço de gerenciamento, o design simples de hub-spoke é a arquitetura de referência do VDC que nós recomendamos.

### <a name="components"></a>Componentes
A implementação de um datacenter virtual é composta por quatro tipos de componente básicos: **Infraestrutura**, **Redes de Perímetro**, **Cargas de Trabalho** e **Monitoramento**.

Cada tipo de componente consiste em vários recursos e funcionalidades do Azure. A implementação de um VDC de empresa é composta por instâncias de vários tipos de componentes e muitas variações do mesmo tipo de componente. Por exemplo, você pode ter muitas instâncias de carga de trabalho diferentes e logicamente separadas que representem diferentes aplicativos. Você usa esses diferentes tipos e instâncias de componentes para criar, por fim, o VDC.

[![4]][4]

A arquitetura de alto nível anterior da implementação de um VDC mostra diferentes tipos de componentes usados em diferentes zonas na topologia hub-spoke. O diagrama mostra os componentes da infraestrutura em várias partes da arquitetura.

Como uma prática recomendada para a implementação de um datacenter ou VDC local, os direitos e os privilégios de acesso devem ser baseados em grupo. Lide com grupos, em vez de usuários individuais, para manter as políticas de acesso de modo consistente entre equipes e minimizar erros de configuração. Atribua e remova usuários de e para grupos apropriados para manter os privilégios de um usuário específico atualizados.

Cada grupo de função deve ter um prefixo exclusivo em seus nomes. Esse prefixo torna mais fácil identificar qual grupo está associado a qual carga de trabalho. Por exemplo, uma carga de trabalho que hospede um serviço de autenticação pode ter grupos chamados **AuthServiceNetOps**, **AuthServiceSecOps**, **AuthServiceDevOps** e **AuthServiceInfraOps**. Funções centralizadas ou funções não relacionadas a um serviço específico podem ser antecedidas por **Corp**. Um exemplo é **CorpNetOps**.

Muitas organizações usam uma variação dos seguintes grupos para fornecer uma divisão importante de funções:

-   O grupo de TI central **Corp** tem os direitos de propriedade para controlar os componentes de infraestrutura. Exemplos de rede e segurança. O grupo deve ter a função de colaborador na assinatura, o controle do hub e direitos de colaborador de rede nos spokes. Organizações de grande porte dividem com frequência essas responsabilidades de gerenciamento entre várias equipes. Por exemplo, o grupo **CorpNetOps** de operações de rede com foco exclusivo na rede e o grupo **CorpSecOps** de operações de segurança, responsável pela política de firewall e segurança. Nesse caso específico, os dois grupos diferentes precisam ser criados para a atribuição dessas funções personalizadas.
-   O grupo de desenvolvimento e teste **AppDevOps** tem a responsabilidade de implantar cargas de trabalho de aplicativos ou serviços. Esse grupo usa a função de colaborador de máquina virtual para implantações de IaaS e/ou uma ou mais funções de colaborador de PaaS. Confira [Funções internas para recursos do Azure][Roles]. Opcionalmente, a equipe de desenvolvimento e teste pode precisar ter visibilidade de políticas de segurança, NSGs, e políticas de roteamento, UDR, dentro do hub ou de um spoke específico. Além das funções de colaborador para cargas de trabalho, esse grupo também precisaria da função de leitor de rede.
-   O grupo de operação e manutenção, **CorpInfraOps** ou **AppInfraOps,** tem a responsabilidade de gerenciar cargas de trabalho em produção. Esse grupo deve ser um colaborador de assinatura em cargas de trabalho em qualquer assinatura de produção. Algumas organizações também podem avaliar se elas precisam de um grupo de equipe de suporte de escalonamento adicional com a função de colaborador de assinatura em produção e na assinatura do hub central. O grupo adicional corrige possíveis problemas de configuração no ambiente de produção.

O VDC é estruturado de modo que os grupos criados para os grupos de TI centrais que gerenciam o hub tenham grupos correspondentes no nível da carga de trabalho. Além de gerenciar recursos de hub, somente os grupos de TI centrais poderiam controlar o acesso externo e permissões de nível superior na assinatura. No entanto, os grupos de carga de trabalho poderiam controlar recursos e permissões da rede virtual de modo independente na TI Central.

Particione o VDC para hospedar vários projetos com segurança em diferentes linhas de negócios. Todos os projetos exigem diferentes ambientes isolados, como dev, UAT e produção. Assinaturas separadas do Azure para cada um desses ambientes fornecem isolamento natural.

[![5]][5]

O diagrama anterior mostra a relação entre projetos, usuários e grupos, bem como os ambientes de uma organização em que os componentes do Azure são implantados.

Normalmente, em TI, um ambiente (ou camada) é um sistema em que vários aplicativos são implantados e executados. Grandes empresas usam um ambiente de desenvolvimento para fazer alterações e testá-las e um ambiente de produção que os usuários finais usam. Esses ambientes são separados, geralmente com vários ambientes de preparo entre eles para permitir a implantação em fases (distribuição), testes e reversão em caso de problemas. As arquiteturas de implantação variam significativamente, mas geralmente ainda seguem o processo básico de iniciar em desenvolvimento, **DEV**, e terminar em produção, **PROD**.

Uma arquitetura comum para esses tipos de ambientes de várias camadas consiste em ambientes de produção Azure DevOps para desenvolvimento e teste, UAT de preparo e ambientes de produção. As organizações podem aproveitar locatários do Azure AD únicos ou múltiplos para definir o acesso e os direitos para esses ambientes. O diagrama anterior mostra um caso em que dois locatários do Azure AD diferentes são usados: um para Azure DevOps e UAT e outro exclusivamente para produção.

A presença de diferentes locatários do Azure AD impõe a separação entre ambientes. O mesmo grupo de usuários, como a TI central, precisa autenticar usando um URI diferente para acessar um locatário diferente do Azure AD, a fim de modificar as funções ou permissões de ambientes do Azure DevOps ou de produção de um projeto. A presença de diferentes autenticações de usuário para acessar diferentes ambientes reduz possíveis interrupções e outros problemas causados por erros humanos.

#### <a name="component-type-infrastructure"></a>Tipo de componente: infraestrutura
Esse tipo de componente é o local em que a maioria da infraestrutura de suporte reside. Também é o ponto em que as equipes centralizadas de TI, segurança e conformidade passam a maior parte do tempo.

[![6]][6]

Os componentes da infraestrutura fornecem uma interconexão entre os diferentes componentes do VDC. Eles estão presentes no hub e nos spokes. Normalmente, a responsabilidade por gerenciar e manter os componentes da infraestrutura é atribuída à equipe de segurança ou TI central.

Uma das principais tarefas da equipe de infraestrutura de TI é assegurar a consistência dos esquemas de endereço IP em toda a empresa. O espaço de endereço IP privado atribuído ao VDC precisa ser consistente. Ele não pode sobrepor endereços IP privados atribuídos em suas redes locais.

O NAT nos roteadores de borda locais ou em ambientes do Azure pode evitar conflitos de endereços IP. Mas ele adiciona complicações aos componentes da sua infraestrutura. Simplicidade de gerenciamento é uma das principais metas do VDC. Portanto, usar o NAT para lidar com preocupações com o IP não é uma solução que recomendamos.

Os componentes da infraestrutura contêm a seguinte funcionalidade:

-   [**Serviços de identidade e diretório**][AAD]. Acesso a todos os tipos de recurso no Azure é controlado por uma identidade armazenada em um serviço de diretório. O serviço de diretório armazena a lista de usuários e os direitos de acesso aos recursos em uma assinatura específica do Azure. Esses serviços podem existir na nuvem apenas  ou podem ser sincronizados com identidade local armazenada no Azure Active Directory.
-   [**Rede virtual**][VPN]. Redes virtuais são um dos principais componentes do VDC. Ao usar redes virtuais, você pode criar um limite de isolamento de tráfego na plataforma do Azure. Uma rede virtual é composta de um ou vários segmentos de rede virtual. Cada um tem um prefixo de rede IP específico, uma sub-rede. A rede virtual define uma área de perímetro interna em que máquinas virtuais de IaaS e serviços de PaaS podem estabelecer uma comunicação privada. Os serviços de PaaS e de VMs em uma rede virtual não podem se comunicar diretamente com serviços de VMs e PaaS em uma rede virtual diferente. Esse isolamento ocorrerá mesmo se o cliente criar as duas redes virtuais na mesma assinatura. O isolamento é uma propriedade vital que garante que as VMs e as comunicações do cliente permaneçam privadas em uma rede virtual.
-   [**UDR**][UDR]. O tráfego em uma rede virtual é roteado por padrão com base na tabela de roteamento do sistema. Uma rota definida pelo usuário é uma tabela de roteamento personalizada que os administradores de rede podem associar a uma ou mais sub-redes. Essa associação substitui o comportamento da tabela de roteamento do sistema e define um caminho de comunicação em uma rede virtual. A presença de UDRs garante que o tráfego de saída spoke transite por VMs personalizadas específicas ou dispositivos de rede virtual e balanceadores de carga presentes no hub e nos spokes.
-   [**NSG**][NSG]. Um grupo de segurança de rede é uma lista de regras de segurança que atuam como filtragem de tráfego em fontes IP, destinos IP, protocolos, portas de origem IP e portas de destino IP. O NSG pode ser aplicado a uma sub-rede, um cartão de NIC virtual associado a uma VM do Azure ou ambos. Os NSGs são essenciais para implementar um controle de fluxo correto no hub e nos spokes. O nível de segurança proporcionado pelo NSG é uma função de quais portas você abre e para qual finalidade. Os clientes devem aplicar filtros adicionais por VM com os firewalls baseados em host, como IPtables ou o Firewall do Windows.
-   [**DNS**][DNS]. A resolução de nomes dos recursos nas redes virtuais do VDC é fornecida por meio do DNS. O Azure fornece serviços de DNS para resolução de nomes [públicos][DNS] e [privados][PrivateDNS]. As zonas privadas fornecem resolução de nomes em uma rede virtual e em redes virtuais. É possível ter zonas privadas que abrangem redes virtuais na mesma região e também em regiões e assinaturas. Para resolução pública, o DNS do Azure fornece um serviço de hospedagem para domínios DNS. Ele fornece resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e faturamento que os outros serviços do Azure.
-   [**Assinatura**][SubMgmt] e [**gerenciamento de grupo de recursos**][RGMgmt]. Uma assinatura define um limite natural para criar vários grupos de recursos no Azure. Recursos em uma assinatura são mostrados em conjunto em contêineres lógicos denominados **grupos de recursos**. O grupo de recursos representa um grupo lógico para organizar os recursos do VDC.
-   [**RBAC**][RBAC]. Por meio de RBAC, é possível mapear as funções organizacionais com direitos de acesso a recursos específicos do Azure, o que permite que você restrinja os usuários a somente um certo subconjunto de ações. Com RBAC, você pode conceder acesso ao atribuir a função apropriada a usuários, grupos e aplicativos no escopo relevante. O escopo de uma atribuição de função pode ser uma assinatura do Azure, um grupo de recursos ou um único recurso. RBAC permite a herança de permissões. Uma função atribuída a um escopo pai também concede acesso aos filhos nele contidos. Com o RBAC, você pode separar as tarefas e conceder aos usuários apenas o nível de acesso de que eles precisam para trabalhar. Por exemplo, use o RBAC para que um funcionário possa gerenciar máquinas virtuais   e outro possa gerenciar bancos de dados SQL, ambos com a mesma assinatura.
-   [**Emparelhamento de rede virtual**][VNetPeering]. O recurso fundamental usado para criar a infraestrutura do VDC é o emparelhamento de rede virtual. É um mecanismo que conecta duas redes virtuais na mesma região por meio da rede de datacenter do Azure ou usando o backbone mundial do Azure nas diferentes regiões.

#### <a name="component-type-perimeter-networks"></a>Tipo de componente: redes de perímetro
Com os componentes da [rede de perímetro][DMZ], ou uma rede de perímetro, você pode fornecer conectividade de rede com suas redes do datacenter físicas ou locais, e com qualquer conectividade para e da Internet. Também é o local em que suas equipes de segurança de rede provavelmente passarão a maior parte do tempo.

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

O diagrama anterior mostra a imposição de dois perímetros com acesso à Internet e uma rede local. Ambos são residentes nos hubs de rede de perímetro e vWAN. Em um hub DMZ, a rede de perímetro para a Internet pode ser escalada verticalmente para dar suporte a um grande número de linhas de negócios, usando vários farms de firewalls de aplicativo web (WAF) ou instâncias do Firewall do Azure. No hub vWAN, a conectividade altamente escalonável entre branches e do branch para o Azure é realizada por meio de uma VPN ou ExpressRoute, conforme necessário.

[**Redes virtuais**][VNet]. O hub geralmente é criado em uma rede virtual com várias sub-redes para hospedar os diferentes tipos de serviços que filtram e inspecionam o tráfego de ou para a Internet por meio de NVAs, WAFs e instâncias de Gateway de Aplicativo do Azure.

[**UDRs**][UDR]. Com UDRs, os clientes podem implantar firewalls, IDSs ou IPSs e outras soluções de virtualização. Eles podem rotear o tráfego de rede por meio dessas soluções de segurança para a aplicação, auditoria e inspeção das políticas de limites de segurança. Você pode criar UDRs no hub e nos spokes. Eles asseguram que o tráfego passe por VMs personalizadas específicas, soluções de virtualização da rede e balanceadores de carga usados pelo VDC. Para garantir que o tráfego gerado de VMs residentes no spoke transite para as soluções de virtualização corretas, defina um UDR nas sub-redes do spoke. Defina o endereço IP de front-end do balanceador de carga interno como o próximo salto. O balanceador de carga interno distribui o tráfego interno para as soluções de virtualização ou back-end do balanceador de carga.

[**Firewall do Azure**][AzFW] é um serviço de segurança de rede gerenciado e baseado em nuvem que protege seus recursos de Rede Virtual do Azure. É um firewall como serviço totalmente com estado com alta disponibilidade interna e escalabilidade de nuvem irrestrita. É possível criar, impor e registrar centralmente políticas de conectividade de rede e de aplicativo em assinaturas e redes virtuais. O Firewall do Azure usa um endereço IP público estático para seus recursos de rede virtual. Ele permite firewalls externos para identificar o tráfego proveniente da sua rede virtual. O serviço é totalmente integrado ao Azure Monitor para registro em log e análise.

[**Soluções de virtualização de rede**][NVA]. No hub, a rede de perímetro com acesso à Internet normalmente é gerenciada por meio de uma instância do Firewall do Azure ou de um farm de firewalls ou do firewall de aplicativo Web (WAF).

Linhas de negócios diferentes normalmente usam muitos aplicativos Web. Esses aplicativos tendem a sofrer de diversas vulnerabilidades e explorações em potencial. Os firewalls de aplicativos Web são um tipo especial de produto usado para detectar ataques contra aplicativos Web (HTTP/HTTPS) em mais profundidade que um firewall genérico. Em comparação à tecnologia de firewall tradicional, WAFs têm um conjunto de recursos específicos para proteger os servidores Web internos contra ameaças.

No entanto, uma única implementação do VDC precisa ser hospedada em uma única região, pois os requisitos de assinatura não permitem a expansão de regiões. Esse requisito faz com que uma única implementação do VDC fique vulnerável a interrupções regionais.

Uma instância do Firewall do Azure ou farms de firewalls do NVA usam um plano de administração comum. Um conjunto de regras de segurança protege as cargas de trabalho hospedadas nos spokes e controla o acesso a redes locais. O Firewall do Azure tem escalabilidade integrada, enquanto os firewalls do NVA podem ser dimensionados manualmente por um balanceador de carga. Em geral, um farm de firewall tem menos software especializado em comparação com um WAF. Porém, ele tem um escopo de aplicativo mais amplo para filtrar e inspecionar qualquer tipo de tráfego de entrada e saída. Se for usada uma abordagem de NVA, ela pode ser localizada e implantada no Azure Marketplace.

É recomendável que você use um conjunto de instâncias do Firewall do Azure ou NVAs para tráfego originado na Internet. Use outro para o tráfego originado localmente. Usar apenas um conjunto de firewalls para ambos é um risco à segurança, uma vez que ele não oferece nenhuma segurança de perímetro entre os dois conjuntos de tráfego de rede. Usar camadas de firewall separadas reduz a complexidade da verificação das regras de segurança e deixa claro quais regras correspondem a quais solicitações de rede de entrada.

A maioria das grandes empresas gerencia vários domínios. Um [DNS do Azure][DNS] pode ser usado para hospedar os registros DNS para um domínio específico. Como exemplo, o VIP (endereço IP virtual) do balanceador externo de carga do Azure, ou WAF, pode ser registrado no registro **A** de um registro de DNS do Azure. Os [**DNSs privados**][PrivateDNS] também estão disponíveis para gerenciar os espaços de endereço privados dentro de redes virtuais.

O [**Azure Load Balancer** ] [ ALB] oferece um serviço de camada 4, TCP ou UDP, de alta disponibilidade. Ele pode distribuir o tráfego de entrada entre instâncias de serviço definidas em um conjunto com balanceamento de carga. O tráfego enviado ao balanceador de carga de pontos de extremidade de front-end, IP públicos ou privados, pode ser redistribuído com ou sem conversão de endereços em um pool de endereços IP de back-end. Os exemplos são soluções de virtualização de rede ou máquinas virtuais.

O Azure Load Balancer também pode testar a integridade de várias instâncias de servidor. Quando uma investigação não responde, o balanceador de carga interrompe o envio de tráfego para a instância não íntegra. No VDC, um balanceador externo de carga no hub equilibra o tráfego para NVAs, por exemplo. Nos spokes, ele realiza tarefas como balanceamento de tráfego entre diferentes VMs de um aplicativo de várias camadas.

O [**Azure Front Door**][AFD] é a plataforma de aceleração de aplicativos da Web altamente disponível e escalonável da Microsoft, o balanceador de carga HTTP global, a proteção de aplicativos e a rede de distribuição de conteúdo. O Front Door é executado em mais de 100 locais na borda da rede global da Microsoft. Usando o Front Door, você pode compilar, operar e expandir seus aplicativos Web dinâmicos e o conteúdo estático. O Front Door oferece os seguintes benefícios ao seu aplicativo: 
* Desempenho de classe mundial do usuário final.
* Automação unificada de manutenção regional ou por selo.
* Automação de BCDR.
* Informações unificadas de clientes ou usuários. 
* Armazenamento em cache. 
* Informações de serviço. A plataforma oferece SLAs de desempenho, confiabilidade e suporte, certificações de conformidade e práticas de segurança auditáveis desenvolvidas, operadas e com suporte nativo pelo Azure.

O [**Gateway de Aplicativo**][AppGW] é uma solução de virtualização dedicada que fornece o ADC (controlador de entrega de aplicativos) como serviço, oferecendo várias funcionalidades de balanceamento de carga de camada 7 para o aplicativo. Você pode otimizar a produtividade do Web farm descarregando a terminação SSL com uso intensivo de CPU para a instância do Gateway de Aplicativo. Ele também fornece outros recursos de roteamento de camada 7, incluindo os seguintes exemplos: 
* Distribuição round robin do tráfego de entrada. 
* Afinidade de sessão baseada em cookie. 
* Visão geral do roteamento baseado em caminho de URL. 
* A capacidade de hospedar vários sites subjacentes em uma única instância do Gateway de Aplicativo. Um firewall do aplicativo Web (WAF) também é fornecido como parte da SKU do WAF do Gateway de Aplicativo. Essa SKU oferece proteção para aplicativos Web contra explorações e vulnerabilidades comuns da Web. O Gateway de Aplicativo pode ser configurado como um gateway voltado para a Internet, um gateway apenas interno ou uma combinação de ambos. 

[**IPs Públicos**][PIP]. Com alguns recursos do Azure, você pode associar pontos de extremidade de serviço a um endereço IP público para que seu recurso possa ser acessado pela Internet. Esse ponto de extremidade usa a conversão de endereços de rede (NAT) para rotear o tráfego para a porta e o endereço internos na rede virtual do Azure. Esse caminho é a principal rota para que o tráfego externo passe para dentro da rede virtual. Você pode configurar endereços IP públicos para determinar qual tráfego é passado para dentro e como e onde ele é convertido para a rede virtual.

[**Proteção contra DDoS do Azure Standard**][DDOS] fornece funcionalidades de mitigação adicionais à camada de [serviço Básica][DDOS] ajustadas especificamente para os recursos de Rede Virtual do Azure. A Proteção contra DDoS Standard é simples de habilitar e não exige nenhuma alteração no aplicativo. Políticas de proteção são ajustadas por meio do monitoramento de tráfego dedicado e algoritmos de aprendizado de máquina. As políticas são aplicadas a endereços IP públicos associados aos recursos implantados em redes virtuais. Os exemplos são instâncias do Azure Load Balancer, Gateway de Aplicativo do Azure e Azure Service Fabric. A telemetria em tempo real está disponível por meio de exibições do Azure Monitor durante um ataque e para o histórico. Proteções de camada de aplicativo podem ser adicionadas por meio do firewall do aplicativo Web do Gateway de Aplicativo do Azure. A proteção é fornecida para endereços IP públicos IPv4 do Azure.

#### <a name="component-type-monitoring"></a>Tipo de componente: monitoramento
Componentes de monitoramento oferecem visibilidade e alertas de todos os outros tipos de componentes. Todas as equipes devem ter acesso ao monitoramento para os componentes e serviços aos quais elas têm acesso. Se você tiver equipes de operações ou suporte técnico centralizadas, elas precisarão ter acesso integrado aos dados fornecidos por esses componentes.

O Azure oferece diferentes tipos de serviços de registro em log e monitoramento para controlar o comportamento de recursos hospedados do Azure. Governança e controle de cargas de trabalho no Azure baseiam-se não apenas na coleta de dados de log, mas também na capacidade de disparar ações com base em eventos relatados específicos.

[**Azure Monitor**][Monitor]. O Azure inclui vários serviços que individualmente executam uma função ou tarefa específica no espaço de monitoramento. Juntos, esses serviços oferecem uma solução abrangente para coletar, analisar e agir na telemetria do seu aplicativo e os recursos do Azure compatíveis com eles. Eles também podem trabalhar para monitorar recursos críticos locais para fornecer um ambiente de monitoramento híbrido. Compreender as ferramentas e os dados que estão disponíveis é a primeira etapa no desenvolvimento de uma estratégia de monitoramento completa para seu aplicativo.

Há dois tipos principais de logs no Azure:

-   O [Log de Atividades do Azure][ActLog], conhecido anteriormente como **Log Operacional**, fornece informações sobre as operações que foram executadas em recursos na assinatura do Azure. Esses logs relatam os eventos de plano de controle para suas assinaturas. Todos os recursos do Azure geram logs de auditoria.

-   Os [logs de diagnóstico do Azure Monitor][DiagLog] são logs gerados por um recurso que fornece dados avançados e frequentes sobre a operação desse recurso. O conteúdo desses logs varia de acordo com o tipo de recurso.

[![9]][9]

No VDC, é importante controlar os logs de NSG, sobretudo estas informações:

-   Os [Logs de eventos][NSGLog] fornecem informações sobre quais regras de NSG são aplicadas às VMs e funções de instância com base no endereço MAC.
-   Os [Logs do contador][NSGLog] controlam quantas vezes cada regra NSG foi executada para negar ou permitir o tráfego.

Todos os logs podem ser armazenados nas contas de armazenamento do Azure para fins de auditoria, análise estática ou backup. Quando você armazena os logs em uma conta de armazenamento do Azure, os clientes podem usar diferentes tipos de estruturas para recuperar, preparar, analisar e visualizar esses dados para relatar o status e a integridade dos recursos de nuvem. 

Grandes empresas já devem ter adquirido uma estrutura padrão para monitorar sistemas locais. Elas podem estender essa estrutura para integrar logs gerados por implantações de nuvem. Usando o [Azure Log Analytics] [https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-queries], as organizações podem manter todo o registro em log na nuvem. O Log Analytics é implementado como um serviço baseado em nuvem. Portanto, você pode colocá-lo em funcionamento rapidamente com investimento mínimo nos serviços de infraestrutura. O Log Analytics também integrar-se a componentes do System Center, como o System Center Operations Manager, para estender seus investimentos atuais em gerenciamento para a nuvem. 

O Log Analytics é um serviço no Azure que ajuda a coletar, correlacionar, pesquisar e agir quanto a dados de desempenho e log gerados por sistemas operacionais, aplicativos e componentes de infraestrutura de nuvem. Ele dá aos clientes informações operacionais em tempo real usando uma pesquisa integrada e painéis personalizados para analisar todos os registros em todas as cargas de trabalho no VDC.

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

#### <a name="component-type-workloads"></a>Tipo de componente: cargas de trabalho
Componentes de carga de trabalho são o local em que seus aplicativos reais e serviços residem. Também é o ponto em que as equipes de desenvolvimento de aplicativo passam a maior parte do tempo.

As possibilidades de carga de trabalho são infinitas. A seguir estão apenas alguns dos tipos de carga de trabalho possíveis:

Os **aplicativos de linha de negócios** são aplicativos de computador críticos para a operação contínua de uma empresa. Os aplicativos de linha de negócios têm algumas características em comum:

-   **Interativo** por natureza. Os dados são inseridos, e os resultados ou os relatórios são retornados.
-   **Controlado por dados**&mdash;dados intensivos com acesso frequente aos bancos de dados ou outro armazenamento.
-   **Integrado**&mdash;oferecem integração a outros sistemas dentro ou fora da organização.

**Clientes para sites da Web (voltados para a Internet ou internos)**. A maioria dos aplicativos que interagem com a Internet são sites da Web. O Azure oferece a capacidade de executar um site em uma VM IaaS ou de um [recurso Aplicativos Web do site (PaaS) do Serviço de Aplicativo do Azure] [ WebApps]. Os Aplicativos Web dão suporte à integração com redes virtuais que permitem a implantação dos Aplicativos Web no spoke do VDC. Ao analisar sites internos, com a integração de rede virtual, você não precisa expor um ponto de extremidade de Internet para seus aplicativos. Mas, em vez disso, você pode usar os recursos por meio de endereços roteáveis privados de fora da Internet de sua rede virtual privada.

**Big Data e análise**. Quando os dados precisam ser escalados verticalmente para um volume muito grande, talvez os bancos de dados não sejam escalados de modo adequado. A tecnologia Apache Hadoop oferece um sistema para executar consultas distribuídas em paralelo em um grande número de nós. Os clientes podem executar cargas de trabalho de dados em VMs de IaaS ou PaaS [Azure HDInsight][HDI]. O HDInsight dá suporte à implantação em uma rede virtual com base no local. Ela pode ser implantada em um cluster em um spoke do VDC.

**Eventos e mensagens**. Os [Hubs de Eventos do Azure][EventHubs] são um serviço de ingestão de telemetria de hiperescala que coleta, transforma e armazena milhões de eventos. Como uma plataforma de streaming distribuída, ela oferece baixa latência e retenção de tempo configurável. Portanto, você pode ingerir grandes quantidades de telemetria no Azure e ler esses dados de vários aplicativos. Com Hubs de Evento, um único fluxo pode dar suporte a pipelines tanto em tempo real quanto em lote.

Você pode implementar um serviço entre aplicativos e serviços por meio de mensagens de nuvem altamente confiável usando o [Barramento de Serviço do Azure][ServiceBus]. Ele oferece um sistema de mensagens agenciado e assíncrono entre o cliente e o servidor, recursos de mensagens PEPS (primeiro a entrar, primeiro a sair) e de publicação e assinatura.

[![10]][10]

### <a name="multiple-vdc-implementations"></a>Implementações de vários VDCs


No entanto, um único VDC é hospedado em uma única região. Ele fica vulnerável a uma grande interrupção que pode afetar toda uma região. Os clientes que desejam obter SLAs altos precisam proteger os serviços por meio de implantações do mesmo projeto em implementações de dois ou mais VDCs, colocados em regiões diferentes.

Além das questões de SLA, há vários cenários comuns em que a implantação de diversos VDCs faz sentido:

-   Presença regional ou global.
-   Recuperação de desastre.
-   Um mecanismo para desviar o tráfego entre datacenters.

#### <a name="regional-and-global-presence"></a>Presença regional ou global
Os datacenters do Azure estão presentes em várias regiões no mundo todo. Ao selecionar vários datacenters do Azure, os clientes precisam considerar dois fatores relacionados: distâncias geográficas e latência. Para oferecer a melhor experiência aos usuários, os clientes precisam avaliar a distância geográfica entre as implementações de VDCs e a distância entre elas e os usuários finais.

A região do Azure que hospeda as implementações de VDCs também precisa estar em conformidade com os requisitos normativos estabelecidos por qualquer jurisdição legal sob a qual sua organização opera.

#### <a name="disaster-recovery"></a>Recuperação de desastre
A implementação de um plano de recuperação de desastre está relacionada ao tipo de carga de trabalho em questão e à capacidade de sincronizar o estado da carga de trabalho entre diferentes implementações de VDCs. O ideal é que a maioria dos clientes queira sincronizar os dados do aplicativo entre as implantações em execução em implementações de dois VDCs diferentes para implementar um mecanismo de failover rápido. A maioria dos aplicativos é sensível à latência e isso pode causar possíveis tempos limites e atrasos na sincronização de dados.

A sincronização ou o monitoramento de pulsação de aplicativos em implementações de diferentes VDCs exige a comunicação entre eles. As implementações de dois VDCs em regiões diferentes podem ser conectadas da seguinte maneira:

-   O emparelhamento de redes virtuais pode conectar hubs entre regiões.
-   O emparelhamento privado do ExpressRoute quando os hubs de VDC estão conectados ao mesmo circuito do ExpressRoute.
-   Vários circuitos do ExpressRoute conectados via backbone corporativo e sua malha VDC conectada aos circuitos do ExpressRoute.
-   Conexões de VPN Site a Site entre seus hubs de VDC em cada região do Azure.

Geralmente, as conexões do ExpressRoute ou o emparelhamento de redes virtuais são os mecanismos preferenciais devido à maior largura de banda e latência consistente ao transitar pelo backbone da Microsoft.

Não há nenhuma receita mágica para validar um aplicativo distribuído entre duas ou mais implementações de VDCs diferentes localizados em regiões diferentes. Os clientes devem executar testes de qualificação de rede para verificar a latência e largura de banda das conexões. Em seguida, defina se a replicação de dados síncrona ou assíncrona é apropriada e qual é o objetivo de tempo de recuperação ideal (RTO) para suas cargas de trabalho.

#### <a name="mechanism-to-divert-traffic-between-datacenters"></a>Mecanismo para desviar o tráfego entre datacenters
Uma técnica eficaz para desviar o tráfego que entra em um datacenter para outro é baseada no Sistema de Nomes de Domínio (DNS). O [Gerenciador de Tráfego do Azure][TM] usa o mecanismo de DNS para direcionar o tráfego do usuário final para o ponto de extremidade público mais apropriado em um VDC específico. Por meio de testes, o Gerenciador de Tráfego verifica periodicamente a integridade do serviço de pontos de extremidade públicos em diferentes implementações de VDC. Se os pontos de extremidade falharem, será feito o roteamento automático para o VDC secundário.

O Gerenciador de Tráfego funciona em pontos de extremidade públicos do Azure. Por exemplo, ele pode controlar ou desviar o tráfego para VMs do Azure e aplicativos Web no VDC apropriado. O Gerenciador de Tráfego é resiliente mesmo em caso de falha de toda a região do Azure. Ele pode controlar a distribuição de tráfego do usuário para pontos de extremidade de serviço em diferentes implementações de VDC com base em vários critérios. Alguns exemplos são a falha de serviço em um VDC específico ou a seleção do VDC com a menor latência de rede para o cliente.

### <a name="conclusion"></a>Conclusão
O datacenter virtual é uma abordagem à migração do data center para a nuvem que usa uma combinação de recursos e funcionalidades para criar uma arquitetura escalonável no Azure. Ele maximiza o uso de recursos de nuvem, reduz custos e simplifica a governança do sistema. O conceito de VDC baseia-se em uma topologia hub-spoke que fornece serviços compartilhados comuns no hub. Ele permite aplicativos específicos ou cargas de trabalho nos spokes. O VDC atende à estrutura das funções da empresa, em que diferentes departamentos funcionam juntos. Os exemplos são TI central, Azure DevOps, operações e manutenção. Cada departamento tem uma lista específica de funções e responsabilidades. O conceito de VDC cumpre os requisitos para uma migração **lift-and-shift**. Mas ele também oferece muitas vantagens em implantações de nuvem nativa.

## <a name="references"></a>Referências
Saiba mais sobre os seguintes recursos discutidos neste artigo:

| | | |
|-|-|-|
|Recursos de rede|Balanceamento de carga|Conectividade|
|[Rede Virtual do Azure][VNet]</br>[Grupos de segurança de rede][NSG]</br>[Logs do NSG][NSGLog]</br>[Rotas definidas pelo usuário][UDR]</br>[Soluções de virtualização de rede][NVA]</br>[Endereços IP Públicos][PIP]</br>[Proteção contra DDoS do Azure][DDOS]</br>[Firewall do Azure][AzFW]</br>[DNS do Azure][DNS]|[Azure Front Door][AFD]</br>[Azure Load Balancer (L3)][ALB]</br>[Gateway de Aplicativo (L7)][AppGW]</br>[Firewall do aplicativo Web][WAF]</br>[Gerenciador de Tráfego do Azure][TM]</br></br></br></br></br> |[Emparelhamento de rede virtual][VNetPeering]</br>[Rede virtual privada][VPN]</br>[WAN Virtual][vWAN]</br>[ExpressRoute][ExR]</br>[ExpressRoute Direct][ExRD]</br></br></br></br></br>
|Identidade</br>|Monitoramento</br>|Práticas recomendadas</br>|
|[Azure Active Directory][AAD]</br>[Autenticação Multifator][MFA]</br>[Controle de acesso baseado em função][RBAC]</br>[Funções padrão do Azure AD][Roles]</br></br></br> |[Observador de Rede][NetWatch]</br>[Azure Monitor][Monitor]</br>[Log de atividades][ActLog]</br>[Logs de diagnóstico do Monitor][DiagLog]</br>[Microsoft Operations Management Suite][OMS]</br>[Monitor de Desempenho de Rede][NPM]|[Práticas recomendadas de redes de perímetro][DMZ]</br>[Gerenciamento de assinaturas][SubMgmt]</br>[Gerenciamento de grupos de recursos][RGMgmt]</br>[Limites de assinatura do Azure][Limits] </br></br></br>|
|Outros serviços do Azure|
|[Aplicativos Web][WebApps]</br>[HDInsight (Hadoop)][HDI]</br>[Hubs de Eventos][EventHubs]</br>[Barramento de Serviço][ServiceBus]|

## <a name="next-steps"></a>Próximas etapas
 - Explore o [emparelhamento de redes virtuais][VNetPeering], a tecnologia de base para os designs de hub e spoke do VDC.
 - Implemente o [Azure AD][AAD] para começar a usar a exploração de [RBAC][RBAC].
 - Desenvolva um modelo de gerenciamento de assinatura e recurso e um modelo de RBAC para atender à estrutura, aos requisitos e às políticas da sua organização. A atividade mais importante é o planejamento. Tanto quanto possível, planeje reorganizações, fusões, novas linhas de produto e outras possibilidades.

<!--Image References-->
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
[ExRD]: https://docs.microsoft.com/en-us/azure/expressroute/expressroute-erdirect-about
[vWAN]: /azure/virtual-wan/virtual-wan-about
[NVA]: /azure/architecture/reference-architectures/dmz/nva-ha
[AzFW]: /azure/firewall/overview
[SubMgmt]: /azure/architecture/cloud-adoption/appendix/azure-scaffold 
[RGMgmt]: /azure/azure-resource-manager/resource-group-overview
[DMZ]: /azure/best-practices-network-security
[ALB]: /azure/load-balancer/load-balancer-overview
[DDOS]: /azure/virtual-network/ddos-protection-overview
[PIP]: /azure/virtual-network/resource-groups-networking#public-ip-address
[AFD]: https://docs.microsoft.com/en-us/azure/frontdoor/front-door-overview
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

