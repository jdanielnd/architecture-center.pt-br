---
title: 'Datacenter Virtual do Azure: uma perspectiva de rede'
description: Aprenda a criar seu data center virtual no Azure
author: tracsman
manager: rossort
tags: azure-resource-manager
ms.service: virtual-network
ms.date: 09/24/2018
ms.author: jonor
ms.openlocfilehash: 61b8e8347fb54e95dcf1bff959e01ef00ecee189
ms.sourcegitcommit: 5d1ee2acb5beb2afea19bcc0cef34655dc70e372
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/05/2018
ms.locfileid: "48812518"
---
# <a name="azure-virtual-datacenter-a-network-perspective"></a>Datacenter Virtual do Azure: uma perspectiva de rede

## <a name="overview"></a>Visão geral
Migrar aplicativos locais para o Azure, mesmo sem quaisquer alterações significativas (uma abordagem conhecida como "lift and shift"), permite que as organizações beneficiem-se de uma infraestrutura segura e eficiente. No entanto, para aproveitar ao máximo possível a agilidade com a computação em nuvem, as empresas devem evoluir suas arquiteturas para aproveitarem os recursos do Azure. O Microsoft Azure oferece serviços e infraestrutura em larga escala, recursos e confiabilidade de nível empresarial e várias opções de conectividade híbrida. Os clientes podem optar por acessar esses serviços de nuvem pela Internet ou com o Azure ExpressRoute, que fornece conectividade de rede privada. A plataforma Microsoft Azure permite que os clientes estendam facilmente sua infra-estrutura para a nuvem e criem arquiteturas com várias camadas. Além disso, os parceiros da Microsoft oferecem recursos avançados, oferecendo serviços de segurança e soluções de virtualização otimizados para execução no Azure.

Este artigo apresenta uma visão geral de padrões e designs que podem ser usados para resolver as preocupações de segurança, desempenho e escala arquitetura que muitos clientes enfrentam quando pensam em uma transição em massa para a nuvem. Uma visão geral de como ajustar as diferentes funções de TI organizacionais ao gerenciamento e à governança do sistema também é discutida, com ênfase no requisitos de segurança e na otimização de custos.

## <a name="what-is-a-virtual-data-center"></a>O que é um Data Center Virtual?
Antigamente, as soluções de nuvem eram projetadas para hospedar aplicativos únicos relativamente isolados no espectro público. Essa abordagem funcionou bem por alguns anos. No entanto, conforme os benefícios das soluções de nuvem ficaram mais claros e várias cargas trabalho em grande escala eram hospedadas na nuvem, lidar com questões de segurança, confiabilidade, desempenho e custo de implantações em uma ou mais regiões tornou-se vital em todo o ciclo de vida do serviço de nuvem.

O diagrama de implantação em nuvem a seguir ilustra alguns exemplos de falhas de segurança (caixa vermelha) e espaço para otimizar soluções de virtualização de rede entre as cargas de trabalho (caixa amarela).

[![0]][0]

O VDC (Virtual Data Center) nasceu da necessidade de ajustar a escala para dar suporte a cargas de trabalho corporativas e lidar com os problemas introduzidos ao dar suporte a aplicativos em grande escala na nuvem pública.

Um VDC não se refere apenas às cargas de trabalho de aplicativos na nuvem, mas também à rede, à segurança, ao gerenciamento e à infraestrutura (por exemplo, DNS e Serviços de Diretório). Geralmente também fornece uma conexão privada de volta a uma rede ou data center local. Conforme cada vez mais cargas de trabalho vão para o Azure, é importante pensar na infraestrutura de suporte e nos objetos em que essas cargas de trabalho estão colocadas. Pensar cuidadosamente em como os recursos são estruturados pode evitar a proliferação de centenas de "ilhas de carga de trabalho" que devem ser gerenciadas separadamente com o fluxo de dados, modelos de segurança e desafios de conformidade independentes.

Um Data Center Virtual é essencialmente uma coleção de entidades separadas, porém relacionadas, com infraestrutura, recursos e funções de suporte em comum. Ao exibir suas cargas de trabalho como um VDC integrado, você pode aproveitar o custo reduzido devido a economias de escala, segurança otimizada por meio da centralização de componentes e fluxos de dados, além de operações, gerenciamento e auditorias de conformidade mais fáceis.

> [!NOTE]
> É importante compreender que o VDC **NÃO** é um produto separado do Azure, mas a combinação de vários recursos e capacidades para atender às suas necessidades exatas. O VDC é uma maneira de pensar sobre suas cargas de trabalho e o uso do Azure para maximizar os recursos e as habilidades na nuvem. O DC virtual é, portanto, uma abordagem modular sobre como criar serviços de TI no Azure, respeitando funções e responsabilidades organizacionais.

O VDC pode ajudar as empresas a colocarem as cargas de trabalho e os aplicativos no Azure para os seguintes cenários:

-   Hospedagem de várias cargas de trabalho relacionadas
-   Migração de cargas de trabalho de um ambiente local para o Azure
-   Implementação de segurança compartilhada ou centralizada e requisitos de acesso entre cargas de trabalho
-   Como combinar TI centralizada e DevOps apropriadamente para uma grande empresa

A chave para aproveitar as vantagens do VDC é uma topologia centralizada (hub e spokes) com uma combinação de recursos do Azure: [VNet do Azure][VNet], [NSGs][NSG], [Emparelhamento VNet][VNetPeering], [UDR (Rotas Definidas pelo Usuário)][UDR] e Identidade do Azure com [RBAC (Controle de Acesso Baseado em Função)][RBAC] e opcionalmente o [Firewall do Azure][AzFW], [DNS do Azure][DNS], [Azure Front Door][AFD] e [WAN Virtual do Azure][vWAN].

## <a name="who-needs-a-virtual-data-center"></a>Quem precisa de um Data Center Virtual?
Qualquer cliente do Azure que precise ser movido mais do que algumas cargas de trabalho no Azure pode se beneficiar de pensar sobre como usar recursos comuns. Dependendo da magnitude, até mesmo aplicativos únicos podem aproveitar o uso de padrões e componentes empregados para criar um VDC.

Se sua organização tiver uma equipe/departamento de TI, Rede, Segurança e/ou Conformidade centralizado, um VDC poderá ajudar a impor pontos da política, segregação de dever e garantir a uniformidade dos componentes comuns subjacentes enquanto dá às equipes de aplicativo o máximo de liberdade e controle apropriado para suas necessidades.

As organizações que estão considerando o DevOps podem utilizar os conceitos do VDC para fornecer grupos autorizados de recursos do Azure e garantir que tenham controle total dentro desse grupo (um grupo de recursos ou de assinaturas em uma assinatura comum), mas os limites de segurança e de rede permanecem em conformidade, como definido por uma política centralizada em um Grupo de Recursos e VNet de um hub.

## <a name="considerations-on-implementing-a-virtual-data-center"></a>Considerações sobre a implementação de um Data Center Virtual
Ao projetar um VDC, há várias questões importantes a considerar:

-   Serviços de identidade e diretório
-   Infraestrutura de segurança
-   Conectividade com a nuvem
-   Conectividade na nuvem

### <a name="identity-and-directory-service"></a>Serviço de Identidade e Diretório
Serviços de identidade e diretório são um aspecto fundamental de todos os data centers, sejam locais ou na nuvem. A identidade está relacionada a todos os aspectos de acesso e a autorização aos serviços no VDC. Para ajudar a garantir que somente usuários e processos autorizados acessem seus recursos e Conta do Azure, o Azure usa vários tipos de credenciais para autenticação. Eles incluem senhas (para acessar a conta do Azure), chaves de criptografia, assinaturas digitais e certificados. [*MFA* (Autenticação Multifator do Azure)][MFA] é uma camada adicional de segurança para acessar os serviços do Azure. O MFA do Azure oferece autenticação forte com uma variedade de opções fáceis de verificação, como chamada telefônica, mensagem de texto ou notificação por aplicativo móvel, permitindo que os clientes escolham seus métodos preferidos.

Qualquer grande empresa precisa definir um processo de gerenciamento de identidades que descreva o gerenciamento de identidades individuais, autenticação, autorização, funções e privilégios dentro ou no VDC. Os objetivos deste processo devem ser aumentar a segurança e a produtividade e diminuir o custo, o tempo de inatividade e as tarefas manuais repetitivas.

Empresas/organizações podem exigir uma combinação exigente de serviços para diferentes LOBs (linha de negócios) e os funcionários geralmente têm diferentes funções quando estão envolvido em projetos diferentes. Um VDC requer uma boa cooperação entre diferentes equipes, cada uma com definições de função específicas, para que os sistemas sejam executados com boa governança. A matriz de responsabilidades, acesso e direitos pode ser extremamente complexa. O gerenciamento de identidades no VDC é implementado por meio do [*Azure Active Directory* (Azure AD)][AAD] e do RBAC (Controle de Acesso Baseado em Função).

Um Serviço de Diretório é uma infraestrutura de informações compartilhadas para localizar, gerenciar, administrar e organizar itens cotidianos e recursos de rede. Esses recursos podem incluir volumes, pastas, arquivos, impressoras, usuários, grupos, dispositivos e outros objetos. Cada recurso na rede é considerado um objeto pelo servidor de diretório. Informações sobre um recurso são armazenadas como uma coleção de atributos associados a tal recurso ou objeto.

Todos os serviços comerciais do Microsoft Online dependem do Azure Active Directory (Azure AD) para entrada e outras necessidades de identidade. O Azure Active Directory é uma solução de nuvem de gerenciamento de acesso e identidade abrangente e altamente disponível que combina os serviços principais de diretório, governança avançada de identidades e gerenciamento do acesso de aplicativos. O Azure AD pode ser integrado ao Active Directory local para habilitar logon único para todos os aplicativos hospedados localmente e baseados em nuvem (local). Os atributos do usuário do Active Directory local podem ser sincronizados automaticamente com o Azure AD.

Um único administrador global não é necessário para atribuir todas as permissões em um VDC. Em vez disso, cada departamento específico (grupo de usuários ou serviços no Serviço de Diretório) pode ter as permissões necessárias para gerenciar seus próprios recursos em um VDC. Estruturar permissões requer balanceamento. Um número excessivo de permissões pode prejudicar a eficiência de desempenho, enquanto permissões de menos ou não exigentes podem aumentar os riscos de segurança. O RBAC (Controle de Acesso Baseado em Função) do Azure ajuda a resolver esse problema oferecendo um gerenciamento de acesso refinado para recursos VDC.

#### <a name="security-infrastructure"></a>Infraestrutura de segurança
A infraestrutura de segurança, no contexto de um VDC, está principalmente relacionada à segregação de tráfego no segmento de rede virtual específico do VDC e a como controlar os fluxos de entrada e saída em todo o VDC. O Azure se baseia na arquitetura multilocatário que impede o tráfego não autorizado e não intencional entre implantações, usando isolamento de VNet (rede virtual), ACLs (listas de controle de acesso), balanceadores de carga e filtros IP, juntamente com políticas de fluxo de tráfego. NAT (conversão de endereços de rede) separa o tráfego de rede interno do tráfego externo.

A malha do Azure aloca recursos de infraestrutura a cargas de trabalho de locatário e gerencia comunicações para e de VMs (máquinas virtuais). O hipervisor do Azure impõe a separação de memória e processo entre VMs e encaminha com segurança o tráfego de rede a locatários do sistema operacional convidado.

#### <a name="connectivity-to-the-cloud"></a>Conectividade com a nuvem
O VDC precisa de conectividade com redes externas para oferecer serviços a clientes, parceiros e/ou usuários internos. Isso geralmente significa conectividade não apenas com a Internet, mas também com data centers e redes locais.

Os clientes podem criar suas políticas de segurança para controlar o que e como os serviços hospedados do VDC específicos são acessíveis para/na Internet usando o [Firewall do Azure][AzFW] ou as Soluções de Virtualização de Rede, políticas personalizadas de roteamento e filtragem de rede ([Roteamento Definido pelo Usuário][UDR] e [Grupos de Segurança de Rede][NSG]). É recomendável que todos os recursos voltados para Internet sejam protegidos de forma adicional pela [**Proteção contra DDOS do Azure Standard**][DDOS].

As empresas geralmente precisam conectar os VDCs a data centers locais ou outros recursos. A conectividade entre o Azure e redes locais é, portanto, um aspecto essencial ao projetar uma arquitetura eficaz. As empresas têm duas maneiras diferentes de criar uma interconexão entre o VDC e o local no Azure: trânsito pela Internet e/ou por conexões diretas privadas.

Um [**VPN Site a Site do Azure**][VPN] é um serviço de interconexão pela Internet entre redes locais e o VDC, estabelecido por meio de conexões criptografadas seguras (túneis IPsec/IKE). A conexão Site a Site do Azure é flexível e rápida de criar e não requer nenhuma compra adicional, uma vez que todas as conexões são feitas pela Internet.

Para um grande número de conexões VPN, a [**WAN Virtual do Azure**][vWAN] é um serviço de rede que fornece conectividade otimizada e automatizada entre branches por meio do Azure. A WAN Virtual permite que você conecte e configure dispositivos de branch para se comunicar com o Azure. Isso pode ser feito manualmente ou usando dispositivos de provedor preferencial por meio de um parceiro de WAN Virtual. O uso de dispositivos de provedor preferencial facilitam o uso, simplificam a conectividade e permitem o gerenciamento da configuração. O painel interno de WAN do Azure fornece informações instantâneas de solução de problemas que podem ajudar você a economizar tempo e proporcionam uma maneira fácil de exibir conectividade site a site em larga escala.

[**ExpressRoute**][ExR] é um serviço de conectividade do Azure que permite criar conexões privadas entre o VDC e as redes locais. Conexões do ExpressRoute não passam pela Internet pública e oferecem maior segurança, confiabilidade e velocidades mais altas (até 10 Gbps), além de latência consistente. O ExpressRoute é muito útil para os VDCs, uma vez que os clientes do ExpressRoute podem aproveitar os benefícios das regras de conformidade associadas às conexões privadas. Com o [ExpressRoute Direct][ExRD] é possível se conectar diretamente aos roteadores da Microsoft em 100Gbps para os clientes com maiores necessidades de largura de banda.

Implantar conexões do ExpressRoute envolve normalmente a interação com um provedor de serviços do ExpressRoute. Para os clientes que precisam iniciar rapidamente, é comum usar inicialmente o VPN Site a Site para estabelecer a conectividade entre o VDC e os recursos locais, então migrar para a conexão do ExpressRoute quando a interconexão física com o provedor de serviços estiver concluída.

#### <a name="connectivity-within-the-cloud"></a>*Conectividade na nuvem*
[VNets][VNet] e [Emparelhamento VNet][VNetPeering] são serviços de conectividade de rede básicos em um VDC. Uma VNet garante um limite natural de isolamento para recursos de VDC e o emparelhamento VNet permite a intercomunicação entre diferentes VNets dentro da mesma região do Azure ou até mesmo entre regiões. Controle de tráfego em uma VNet e entre VNets precisa corresponder a um conjunto de regras de segurança especificado por meio de Listas de Controle de Acesso ([Grupo de Segurança de Rede][NSG]), [Soluções de Virtualização de Rede][NVA] e as tabelas de roteamento personalizadas ([UDR][UDR]).

## <a name="virtual-data-center-overview"></a>Visão geral do Data Center Virtual

### <a name="topology"></a>Topologia
_Hub e spokes_ são um modelo para estender um Data Center Virtual em uma única região do Azure.

[![1]][1]

O hub é a zona central que controla e inspeciona o tráfego de entrada e/ou saída entre diferentes zonas: Internet, local e os spokes. A topologia de hub e spoke dá ao departamento de TI uma maneira eficiente de impor políticas de segurança em um local central, ao mesmo tempo em que reduz a possibilidade de erros de configuração e exposição.

O hub contém os componentes de serviço comuns consumidos pelos spokes. Aqui estão alguns exemplos típicos de serviços centrais comuns:

-   A infraestrutura do Microsoft Active Directory (com o serviço ADFS relacionado) necessária para a autenticação de usuário de terceiros acessando de redes não confiáveis antes de obterem acesso às cargas de trabalho no spoke
-   Um serviço DNS para resolver nomes para a carga de trabalho nos spokes, para acessar recursos locais e na Internet se o [DNS do Azure][DNS] não for usado
-   Uma infraestrutura PKI para implementar logon único em cargas de trabalho
-   Controle de fluxo (TCP/UDP) entre os spokes e a Internet
-   Controle de fluxo entre o spoke e o local
-   Se desejar, fluxo de controle entre um spoke e outro

O VDC reduz o custo geral usando a infraestrutura de hub compartilhada entre vários spokes.

A função de cada spoke pode ser hospedar diferentes tipos de cargas de trabalho. Os spokes também podem fornecer uma abordagem modular para implantações repetíveis (por exemplo, desenvolvimento e teste, Teste de Aceitação do Usuário, pré-produção e produção) das mesmas cargas de trabalho. Os spokes também podem ser usados para separar e habilitar diferentes grupos na sua organização (por exemplo, grupos de DevOps). Dentro de um spoke, é possível implantar uma carga de trabalho básica ou cargas de trabalho complexas de várias camadas com controle de tráfego entre as camadas.

#### <a name="subscription-limits-and-multiple-hubs"></a>Limites de assinatura e vários hubs
No Azure, cada componente, seja qual for o tipo, é implantado em uma Assinatura do Azure. O isolamento dos componentes do Azure em diferentes assinaturas do Azure pode atender aos requisitos de diferentes LOBs, como configurar níveis diferenciados de acesso e autorização.

Um único VDC pode ser dimensionado em muitos spokes, embora haja limites de plataformas, como acontece com todos os sistemas de TI. A implantação do hub está associada a uma assinatura específica do Azure, que tem restrições e limites (por exemplo, um número máximo de emparelhamentos de VNet – consulte [Assinatura do Azure e limites de serviço, cotas e restrições][Limits] para obter detalhes). Em casos em que os limites possam ser um problema, a arquitetura pode ser escalada verticalmente ainda mais estendendo o modelo de um único hub-spokes para um cluster de hub e spokes. Vários hubs em uma ou mais regiões do Azure podem ser interconectados usando Emparelhamento VNET, ExpressRoute, WAN Virtual ou VPN site a site.

[![2]][2]

A introdução de vários hubs aumenta o esforço de gerenciamento e o custo do sistema e somente seria justificada pela escalabilidade (exemplos: limites ou redundância do sistema) e replicação regional (exemplos: recuperação de desastre ou desempenho do usuário final). Em cenários que exigem vários hubs, todos os hubs devem buscar oferecer o mesmo conjunto de serviços para facilidade operacional.

#### <a name="interconnection-between-spokes"></a>Interconexão entre spokes
Dentro de um único spoke, é possível implementar cargas de trabalho complexas de várias camadas. Configurações de várias camadas podem ser implementadas usando sub-redes (uma para cada camada) na mesma VNet e filtrando os fluxos usando NSGs.

Por outro lado, um arquiteto talvez queira implantar uma carga de trabalho de várias camadas em várias VNets. Usando emparelhamento VNet, spokes podem se conectar a outros spokes no mesmo hub ou em hubs diferentes. Um exemplo típico desse cenário é o caso em que os servidores de processamento do aplicativo estão em um spoke (VNet), enquanto o banco de dados está implantado em um spoke diferente (VNet). Nesse caso, é fácil interconectar os spokes ao emparelhamento VNet e, assim, evitar trânsito pelo hub. Um exame cuidadoso de arquitetura e segurança deve ser executada para garantir que ignorar o hub não ignore pontos de auditoria ou segurança importantes que podem existir somente no hub.

[![3]][3]

Os spokes também podem ser interconectados a um spoke que atue como um hub. Essa abordagem cria uma hierarquia de dois níveis: o spoke no nível mais alto (nível 0) torna-se o hub dos spokes inferiores (nível 1) da hierarquia. Os spokes do VDC precisam encaminhar o tráfego para o hub central para alcançarem a Internet ou a rede local. Uma arquitetura com dois níveis de hub apresenta roteamento complexo que remove os benefícios de uma relação de hub-spoke simples.

Embora o Azure permita topologias complexas, dois princípios mais importantes do conceito de VDC são a repetição e a simplicidade. Para minimizar o esforço de gerenciamento, o design simples de spoke em hub é a arquitetura de referência do VDC recomendada.

### <a name="components"></a>Componentes
Um Data Center Virtual é composto por quatro tipos de componente básicos: **Infraestrutura**, **Redes de Perímetro**, **Cargas de Trabalho** e **Monitoramento**.

Cada tipo de componente consiste em vários recursos e funcionalidades do Azure. Seu VDC é composto por instâncias de vários tipos de componentes e muitas variações do mesmo tipo de componente. Por exemplo, você pode ter muitas instâncias de carga de trabalho diferentes e logicamente separadas que representem diferentes aplicativos. Você usa esses diferentes tipos e instâncias de componentes para criar, por fim, o VDC.

[![4]][4]

A arquitetura de alto nível anterior de um VDC mostra diferentes tipos de componente usados em diferentes zonas na topologia de spokes em hub. O diagrama mostra os componentes da infraestrutura em várias partes da arquitetura.

Como uma prática recomendada (para um DC ou VDC local), os direitos e os privilégios de acesso devem ser baseados em grupo. Lidar com grupos, em vez de usuários individuais, ajuda a manter as políticas de acesso de modo consistente entre equipes e a minimizar erros de configuração. Atribuir e remover usuários de e para grupos apropriados ajuda a manter os privilégios de um usuário específico atualizados.

Cada grupo de função deve ter um prefixo exclusivo em seus nomes, tornando mais fácil identificar qual grupo está associado a qual carga de trabalho. Por exemplo, uma carga de trabalho que hospede um serviço de autenticação pode ter grupos chamados *AuthServiceNetOps, AuthServiceSecOps, AuthServiceDevOps e AuthServiceInfraOps.* Da mesma forma, para funções centralizadas ou funções não relacionadas a um serviço específico, poderia ser usado o sufixo "Corp", *CorpNetOps*, por exemplo.

Muitas organizações usam uma variação dos seguintes grupos para fornecer uma divisão importante de funções:

-   O *grupo central de TI (Corp)* tem os direitos de propriedade para controlar os componentes de infraestrutura (como rede e segurança) e, portanto, precisa ter a função de colaborador na assinatura (e ter controle do hub) e direitos de colaborador de rede nos raios. Organizações de grande porte com frequência dividem essas responsabilidades de gerenciamento entre várias equipes, como: um grupo de Operações de Rede (CorpNetOps) (com foco exclusivo na rede) e um grupo de Operações de Segurança (CorpSecOps) (responsável para política de segurança e de firewall). Nesse caso específico, os dois grupos diferentes precisam ser criados para a atribuição dessas funções personalizadas.
-   O *grupo de desenvolvimento e teste (AppDevOps)* tem a responsabilidade de implantar cargas de trabalho (Aplicativos ou Serviços). Esse grupo usa a função de Colaborador de Máquina Virtual para implantações de IaaS e/ou uma ou mais funções de colaborador de PaaS (consulte [Funções internas para Controle de Acesso Baseado em Função do Azure][Roles]). Opcionalmente, a equipe de desenvolvimento e teste pode precisar ter visibilidade de políticas de segurança (NSGs) e políticas de roteamento (UDR) dentro do hub ou de um spoke específico. Portanto, além das funções de colaborador para cargas de trabalho, esse grupo também precisaria da função de Leitor de Rede.
-   O *grupo de operação e manutenção (CorpInfraOps ou AppInfraOps)* tem a responsabilidade de gerenciar cargas de trabalho em produção. Esse grupo deve ser um colaborador de assinatura em cargas de trabalho em qualquer assinatura de produção. Algumas organizações também podem avaliar se elas precisam de um grupo de equipe suporte de escalonamento adicional com a função de colaborador de assinatura em produção e na assinatura do hub central, para corrigir possíveis problemas de configuração no ambiente de produção.

Um VDC é estruturado de modo que os grupos criados para os grupos de TI centrais que gerenciam o hub tenham grupos correspondentes no nível da carga de trabalho. Além de gerenciar recursos de hub, somente os grupos de TI centrais seriam capazes de controlar o acesso externo e permissões de nível superior na assinatura. No entanto, os grupos de carga de trabalho seriam capazes de controlar recursos e permissões da VNet de modo independente na TI Central.

O VDC precisa ser particionado para hospedar vários projetos com segurança em diferentes LOBs (Linhas de Negócios). Todos os projetos exigem diferentes ambientes isolados (Dev, UAT, produção). Assinaturas separadas do Azure para cada um desses ambientes fornecem isolamento natural.

[![5]][5]

O diagrama anterior mostra a relação entre os projetos, os usuários, os grupos e os ambientes de uma organização em que os componentes do Azure são implantados.

Normalmente, em TI, um ambiente (ou camada) é um sistema em que vários aplicativos são implantados e executados. Grandes empresas usam um ambiente de desenvolvimento (em que as alterações são originalmente feitas e testadas) e um ambiente de produção (que os usuários finais usam). Esses ambientes são separados, geralmente com vários ambientes de preparo entre eles para permitir a implantação em fases (distribuição), testes e reversão em caso de problemas. As arquiteturas de implantação variam significativamente, mas geralmente o processo básico de iniciar em desenvolvimento (DEV) e terminar em produção (PROD) ainda é seguido.

Uma arquitetura comum para esses tipos de ambientes de várias camadas consiste em ambientes de produção DevOps (desenvolvimento e teste), UAT (de preparo) e ambientes de produção. As organizações podem aproveitar locatários do Azure AD únicos ou múltiplos para definir o acesso e os direitos para esses ambientes. O diagrama anterior mostra um caso em que dois locatários do Azure AD diferentes são usados: um para DevOps e UAT e outro exclusivamente para produção.

A presença de diferentes locatários do Azure AD impõe a separação entre ambientes. O mesmo grupo de usuários (por exemplo, TI Central) precisa autenticar usando um URI diferente para acessar um locatário diferente do AD, modificar as funções ou permissões de ambientes de DevOps ou de produção de um projeto. A presença de diferentes autenticações de usuário para acessar diferentes ambientes reduz possíveis interrupções e outros problemas causados por erros humanos.

#### <a name="component-type-infrastructure"></a>Tipo de componente: infraestrutura
Esse tipo de componente é o local em que a maioria da infraestrutura de suporte reside. Também é o ponto em que as equipes centralizadas de TI, Segurança e/ou Conformidade passam a maior parte do tempo.

[![6]][6]

Os componentes da infraestrutura fornecem uma interconexão entre os diferentes componentes de um VDC e estão presentes no hub e nos spokes. Normalmente, a responsabilidade por gerenciar e manter os componentes da infraestrutura é atribuída à equipe de segurança e/ou TI central.

Uma das principais tarefas da equipe de infraestrutura de TI é assegurar a consistência dos esquemas de endereço IP em toda a empresa. O espaço do endereço IP privado atribuído ao VDC precisa ser consistente e NÃO se sobrepor aos endereços IP privados atribuídos em suas redes locais.

Embora NAT nos roteadores de borda locais ou em ambientes do Azure possa evitar conflitos de endereço IP, ele adiciona complicações aos componentes da sua infraestrutura. A simplicidade de gerenciamento é uma das principais metas do VDC, assim usar a NAT para lidar com os problemas de IP não é uma solução recomendada.

Componentes da infraestrutura contêm a seguinte funcionalidade:

-   [**Serviços de identidade e diretório**][AAD]. Acesso a todos os tipos de recurso no Azure é controlado por uma identidade armazenada em um serviço de diretório. O serviço de diretório armazena não apenas a lista de usuários, mas também os direitos de acesso aos recursos em uma assinatura específica do Azure. Esses serviços podem existir somente em nuvem ou podem ser sincronizados com identidade local armazenada no Active Directory.
-   [**Rede Virtual**][VPN]. As Redes Virtuais são um dos principais componentes de um VDC e permitem criar um limite de isolamento do tráfego na plataforma do Azure. Uma Rede Virtual é composta por um ou vários segmentos de rede virtual, cada um com um prefixo de rede IP específico (uma sub-rede). A rede Virtual define uma área de perímetro interna em que máquinas virtuais de IaaS e serviços de PaaS podem estabelecer uma comunicação privada. VMs (e serviços de PaaS) em uma rede virtual não podem se comunicar diretamente com VMs (e serviços de PaaS) em uma rede virtual diferente, mesmo se as duas redes virtuais forem criadas pelo mesmo cliente, na mesma assinatura. Isolamento é uma propriedade vital que garante que as VMs e as comunicações do cliente permaneçam privadas em uma rede virtual.
-   [**UDR**][UDR]. O tráfego em uma Rede Virtual é roteado por padrão com base na tabela de roteamento do sistema. Uma Rota de Definição de Usuário é uma tabela de roteamento personalizada que administradores de rede podem associar a uma ou mais sub-redes para substituir o comportamento da tabela de roteamento do sistema e definir um caminho de comunicação em uma rede virtual. A presença de UDRs garante que o tráfego de saída spoke transite por VMs personalizadas específicas e/ou Dispositivos de Rede Virtual e balanceadores de carga presentes no hub e nos spokes.
-   [**NSG**][NSG]. Um Grupo de Segurança de Rede é uma lista de regras de segurança que atuam como filtragem de tráfego em Fontes IP, Destino IP, Protocolos, portas de Origem IP e portas de Destino IP. O NSG pode ser aplicado a uma sub-rede, um cartão de NIC Virtual associado a uma VM do Azure ou ambos. Os NSGs são essenciais para implementar um controle de fluxo correto no hub e nos spokes. O nível de segurança proporcionada pelo NSG é uma função de quais portas você abre e para qual finalidade. Os clientes devem aplicar filtros adicionais por VM com os firewalls baseados em host, como IPtables ou o Firewall do Windows.
-   [**DNS**][DNS]. A resolução de nomes dos recursos nas VNets de um VDC é fornecida por meio do DNS. O Azure fornece serviços de DNS para resolução de nomes [Públicos][DNS] e [Privados][PrivateDNS]. As zonas privadas fornecem resolução de nomes em uma rede virtual e em redes virtuais. É possível ter zonas privadas que abrangem não apenas redes virtuais na mesma região, mas também regiões e assinaturas. Para resolução pública, o DNS do Azure fornece um serviço de hospedagem para domínios DNS, fornecendo resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure.
-   [**Assinatura**][SubMgmt] e [**Gerenciamento de Grupo de Recursos**][RGMgmt]. Uma assinatura define um limite natural para criar vários grupos de recursos no Azure. Recursos em uma assinatura são montados em conjunto em contêineres lógicos denominados Grupos de Recursos. O Grupo de Recursos representa um grupo lógico para organizar os recursos de um VDC.
-   [**RBAC**][RBAC]. Por meio de RBAC, é possível mapear a função organizacional junto com direitos de acesso a recursos específicos do Azure, permitindo que você restrinja os usuários a somente um certo subconjunto de ações. Com RBAC, você pode conceder acesso ao atribuir a função apropriada a usuários, grupos e aplicativos no escopo relevante. O escopo de uma atribuição de função pode ser uma assinatura do Azure, um grupo de recursos ou um único recurso. RBAC permite a herança de permissões. Uma função atribuída a um escopo pai também concede acesso aos filhos contidos nele. Usando RBAC, você pode separar as tarefas e conceder aos usuários apenas o nível de acesso de que eles precisam para trabalhar. Por exemplo, use RBAC para permitir que um funcionário gerencie máquinas virtuais em uma assinatura, enquanto outro pode gerenciar bancos de dados SQL na mesma assinatura.
-   [**Emparelhamento VNet**][VNetPeering]. O recurso fundamental usado para criar a infraestrutura de um VDC é o Emparelhamento VNet um mecanismo que conecta duas VNets (redes virtuais) na mesma região por meio da rede do data center do Azure ou usa o backbone internacional do Azure entre as regiões.

#### <a name="component-type-perimeter-networks"></a>Tipo de componente: redes de perímetro
Componentes da [rede de perímetro][DMZ] (também conhecidos como uma rede DMZ) permitem que você forneça conectividade de rede com suas redes do data center físicas ou locais, juntamente com qualquer conectividade para e da Internet. Também é o local em que suas equipes de segurança de rede provavelmente passarão a maior parte do tempo.

Os pacotes de entrada devem fluir por dispositivos de segurança no hub, como firewall, IDS, IPS e afins, antes de alcançar os servidores back-end nos spokes. Pacotes limitados à Internet das cargas de trabalho também devem fluir pelos dispositivos de segurança na rede de perímetro para fins de imposição de política, inspeção e auditoria antes de saírem da rede.

Os componentes de rede de perímetro fornecem os seguintes recursos:

-   [Redes Virtuais][VNet], [UDR][UDR], [NSG][NSG]
-   [Solução de Virtualização de Rede][NVA]
-   [Load Balancer][ALB]
-   [Gateway de Aplicativo][AppGW] / [WAF][WAF]
-   [IPs Públicos][PIP]
-   [Azure Front Door][AFD]
-   [Firewall do Azure][AzFW]

Normalmente, as equipes de segurança e TI centrais têm a responsabilidade de definir requisitos e operações das redes de perímetro.

[![7]][7]

O diagrama anterior mostra a imposição de dois perímetros com acesso à Internet e uma rede local, ambos residentes nos hubs DMZ e vWAN. Em um hub DMZ, a rede de perímetro para a Internet pode ser escalada verticalmente para dar suporte a grandes números de LOBs, usando vários farms de WAFs (Firewalls de Aplicativo Web) e/ou Firewall do Azure. No hub vWAN, a conectividade altamente escalonável entre branches e do branch para o Azure é realizada por meio de VPN ou ExpressRoute, conforme necessário.

[**Redes Virtuais**][VNet] O hub geralmente é criado em uma VNet com várias sub-redes para hospedar os diferentes tipos de serviços que filtram e inspecionam o tráfego de ou para a Internet por meio de NVAs, WAFs e Gateways de Aplicativo do Azure.

[**UDR**][UDR] Usando UDR, os clientes podem implantar firewall, IDS/IPS e outras soluções de virtualização, além de rotear o tráfego de rede por meio dessas soluções de segurança para imposição de política, auditoria e inspeção de limite de segurança. Os UDRs podem ser criados no hub e nos spokes para assegurar que o tráfego passe por VMs personalizadas específicas, Soluções de Virtualização da Rede e balanceadores de carga usados pelo VDC. Para assegurar que o tráfego gerado de VMs residentes no spoke transite para as soluções de virtualização corretas, um UDR precisa ser definido nas sub-redes do spoke configurando o endereço IP de front-end do balanceador de carga interno como o próximo salto. O balanceador de carga interno distribui o tráfego interno para as soluções de virtualização (pool de back-end do balanceador de carga).

[**Firewall do Azure**][AzFW] é um serviço de segurança de rede gerenciado e baseado em nuvem que protege seus recursos de Rede Virtual do Azure. É um firewall totalmente com estado como serviço, com alta disponibilidade interna e escalabilidade de nuvem sem restrições. É possível criar, impor e registrar centralmente políticas de conectividade de rede e de aplicativo em assinaturas e redes virtuais. O Firewall do Azure usa um endereço IP público estático para seus recursos de rede virtual, permitindo que firewalls externos identifiquem o tráfego originário de sua rede virtual. O serviço é totalmente integrado ao Azure Monitor para registro em log e análise.

[**Soluções de Virtualização de Rede**][NVA] No hub, a rede de perímetro com acesso à Internet normalmente é gerenciado por meio do Firewall do Azure ou de um farm de firewalls e/ou WAFs (Firewalls de Aplicativo Web).

LOBs diferentes geralmente usam muitos aplicativos Web e esses aplicativos tendem a sofrer de diversas vulnerabilidades e explorações em potencial. Os Firewalls de Aplicativos Web são uma categoria especial de produto usada para detectar ataques contra aplicativos Web (HTTP/HTTPS) em mais profundidade que um firewall genérico. Em comparação à tecnologia de firewall tradicional, WAFs têm um conjunto de recursos específicos para proteger os servidores Web internos contra ameaças.

Um Firewall do Azure ou farm de firewall do NVA usam um plano de administração comum, com um conjunto de regras de segurança para proteger as cargas de trabalho hospedadas em spokes e controlar o acesso a redes locais. O Firewall do Azure tem escalabilidade integrada, enquanto os firewalls do NVA podem ser dimensionados manualmente por um balanceador de carga. Normalmente, um farm de firewall tem software menos especializado comparado a um WAF, mas tem um escopo de aplicação mais amplo para filtrar e inspecionar qualquer tipo de tráfego de entrada e saída. Se for usada uma abordagem de NVA, ela pode ser localizada e implantada no Azure Marketplace.

É recomendável usar um conjunto de Firewall do Azure (ou NVAs) para tráfego originado na Internet e outro para o tráfego originado localmente. Usar apenas um conjunto de firewalls para ambos é um risco à segurança, uma vez que ele não oferece nenhuma segurança de perímetro entre os dois conjuntos de tráfego de rede. Usar camadas de firewall separadas reduz a complexidade de verificar as regras de segurança e deixa claro quais regras correspondem a quais solicitações de rede de entrada.

A maioria das grandes empresas gerencia vários domínios. Um [**DNS do Azure**][DNS] pode ser usado para hospedar os registros DNS para um domínio específico. Como exemplo, o VIP (endereço IP virtual) do balanceador externo de carga do Azure (ou WAFs) pode ser registrado no registro A de um registro de DNS do Azure. O [**DNS privado**][PrivateDNS] também está disponível para gerenciar os espaços de endereço privado dentro de VNets.

[**Azure Load Balancer**][ALB] O Azure Load Balancer oferece um serviço de Camada 4 (TCP, UDP) alta disponibilidade, que pode distribuir o tráfego de entrada entre instâncias de serviço definidas em um conjunto com balanceamento de carga. O tráfego enviado ao balanceador de carga de pontos de extremidade de front-end (pontos de extremidade IP públicos ou pontos de extremidade IP privados) pode ser redistribuído com ou sem conversão de endereços para um pool de endereços IP de back-end (exemplos são Soluções de Virtualização de Rede ou VMs).

O Azure Load Balancer também pode investigar a integridade de várias instâncias de servidor e quando uma investigação falha em responder, o balanceador de carga para de enviar tráfego para a instância não íntegra. Em um VDC, temos a presença de um balanceador de carga externo no hub (por exemplo, balancear o tráfego para NVAs) e nos spokes (para realizar tarefas como o balanceamento do tráfego entre diferentes VMs de um aplicativo com várias camadas).

O [**Azure Front Door**][AFD] (AFD) é a Plataforma de Aceleração de Aplicativos da Web altamente disponível e escalonável da Microsoft, o Balanceador de Carga HTTP Global, a Proteção de Aplicativos e a Rede de Distribuição de Conteúdo. Em execução em mais de 100 locais com a Rede Global Edge da Microsoft, o AFD permite compilar, operar e escalar horizontalmente seu aplicativo da web dinâmico e o conteúdo estático. O AFD fornece seu aplicativo com o desempenho do usuário final de alto nível, automação de manutenção regional/carimbo unificada, automação de BCDR, informações unificadas de cliente/usuário, insights de serviço e caching. A plataforma oferece SLAs de desempenho, confiabilidade e suporte, certificações de conformidade e práticas de segurança auditáveis desenvolvidas, operadas e com suporte nativo pelo Azure.

[**Gateway de Aplicativo**][AppGW] O Gateway de Aplicativo do Microsoft Azure é uma solução de virtualização dedicada que fornece o ADC (controlador de entrega de aplicativos) como um serviço, oferecendo vários recursos de balanceamento de carga de camada 7 para o seu aplicativo. Ele permite que você otimize a produtividade do Web farm descarregando a terminação SSL com uso intensivo de CPU para o Gateway de Aplicativo. Ele também fornece outros recursos de roteamento de camada 7, incluindo distribuição round robin do tráfego de entrada, afinidade de sessão, roteamento com base no caminho de URL e a capacidade de hospedar vários sites por trás de um único Gateway de Aplicativo baseado em cookie. Um WAF (firewall do aplicativo Web) também é fornecido como parte da SKU do WAF do gateway de aplicativo. Essa SKU oferece proteção para aplicativos Web contra explorações e vulnerabilidades comuns da Web. O Gateway de Aplicativo pode ser configurado como um gateway voltado para a Internet, um gateway apenas interno ou uma combinação de ambos. 

[**IPs públicos**][PIP] Alguns recursos do Azure permitem associar pontos de extremidade de serviço a um endereço IP público que permite que seu recurso seja acessado pela Internet. Esse ponto de extremidade usa NAT (Conversão de Endereços de Rede) para rotear o tráfego para a porta e endereço internos na rede virtual do Azure. Esse caminho é a principal rota para que o tráfego externo passe para dentro da rede virtual. Os endereços IP Públicos podem ser configurados pelo usuário para determinar qual tráfego é passado para dentro e como e onde ele é convertido para a rede virtual.

[**Proteção contra DDoS do Azure Standard**][DDOS] fornece funcionalidades de mitigação adicionais à camada de [serviço Básica][DDOS] ajustadas especificamente para os recursos de Rede Virtual do Azure. A Proteção contra DDoS Standard é simples de habilitar e não exige nenhuma alteração no aplicativo. Políticas de proteção são ajustadas por meio do monitoramento de tráfego dedicado e algoritmos de aprendizado de máquina. As políticas aplicadas a Endereços IP Públicos associados aos recursos implantados em redes virtuais, como instâncias do Azure Load Balancer, Gateway de Aplicativo do Azure e Azure Service Fabric. A telemetria em tempo real está disponível por meio de exibições do Azure Monitor durante um ataque e para fins de histórico. Proteções de camada de aplicativo podem ser adicionadas por meio do Firewall do Aplicativo Web do Gateway de Aplicativo do Azure. A proteção é fornecida para endereços IP públicos IPv4 do Azure.

#### <a name="component-type-monitoring"></a>Tipo de componente: monitoramento
Componentes de monitoramento oferecem visibilidade e alertas de todos os outros tipos de componentes. Todas as equipes devem ter acesso ao monitoramento para os componentes e serviços aos quais elas têm acesso. Se você tiver um equipes de operações ou suporte técnico centralizada, ela precisará ter acesso integrado aos dados fornecidos por esses componentes.

O Azure oferece diferentes tipos de serviços de registro em log e monitoramento para controlar o comportamento de recursos hospedados do Azure. Governança e controle de cargas de trabalho no Azure baseiam-se não apenas na coleta de dados de log, mas também na capacidade de disparar ações com base em eventos relatados específicos.

[**Azure Monitor**][Monitor] - O Azure inclui vários serviços que executam individualmente uma função ou tarefa específica no espaço de monitoramento. Juntos, esses serviços oferecem uma solução abrangente para coletar, analisar e agir na telemetria do seu aplicativo e os recursos do Azure compatíveis com eles. Eles também podem trabalhar para monitorar recursos críticos locais para fornecer um ambiente de monitoramento híbrido. Compreender as ferramentas e os dados que estão disponíveis é a primeira etapa no desenvolvimento de uma estratégia de monitoramento completa para seu aplicativo.

Há dois tipos principais de logs no Azure:

-   [**Logs de Atividade**][ActLog] (conhecidos também como "Log Operacional") fornecem informações sobre as operações que foram executadas em recursos na assinatura do Azure. Esses logs relatam os eventos de plano de controle para suas assinaturas. Todos os recursos do Azure geram logs de auditoria.

-   [**Logs de Diagnóstico do Azure**][DiagLog] são logs gerados por um recurso que fornecem dados avançados e frequentes sobre a operação desse recurso. O conteúdo desses logs varia de acordo com o tipo de recurso.

[![9]][9]

Em um VDC, é extremamente importante controlar os logs de NSGs, sobretudo estas informações:

-   [**Logs de Eventos**][NSGLog]: fornecem informações sobre quais regras de NSG são aplicadas às VMs e funções de instância com base no endereço MAC.
-   [**Logs do Contador**][NSGLog]: controlam quantas vezes cada regra NSG foi executada para negar ou permitir o tráfego.

Todos os logs podem ser armazenados nas Contas de Armazenamento do Azure para fins de auditoria, análise estática ou backup. Quando os logs são armazenados em uma conta de armazenamento do Azure, os clientes podem usar diferentes tipos de estruturas para recuperar, preparar, analisar e visualizar esses dados para relatar o status e a integridade dos recursos de nuvem.

Grandes empresas já devem ter adquirido uma estrutura padrão para monitorar sistemas locais e podem estender essa estrutura para integrar logs gerados por implantações de nuvem. Para organizações que desejam manter todo o registro na nuvem, o[Log Analytics][../log-analytics/log-analytics-overview .md] é uma ótima escolha. Como o Log Analytics é implementado como um serviço baseado em nuvem, é possível colocá-lo em funcionamento com investimentos mínimos em serviços de infraestrutura. O Log Analytics também pode integrar-se a componentes do System Center, como System Center Operations Manager, para estender seus investimentos atuais em gerenciamento para a nuvem.

O Log Analytics é um serviço no Azure que ajuda a coletar, correlacionar, pesquisar e agir quanto a dados de desempenho e log gerados por sistemas operacionais, aplicativos e componentes de infraestrutura de nuvem. Ele dá aos clientes informações operacionais em tempo real usando uma pesquisa integrada e painéis personalizados para analisar todos os registros em todas as cargas de trabalho em um VDC.

[Observador de Rede do Azure][NetWatch] fornece ferramentas para monitorar, diagnosticar, exibir métricas e ativar ou desativar os registros de recursos em uma rede virtual do Azure. É um serviço multifacetado que permite as seguintes funcionalidades e muito mais:
-    Monitorar a comunicação entre uma máquina virtual e um ponto de extremidade
-    Exibir recursos em uma rede virtual e suas relações
-    Diagnosticar problemas de filtragem de tráfego de erro para ou de uma VM
-    Diagnosticar problemas de roteamento de rede de uma VM
-    Diagnosticar de conexão de saída de uma máquina virtual
-    Capturar pacotes para e de uma máquina virtual
-    Diagnosticar problemas com conexões e gateway de rede Virtual do Azure
-    Determinar as latências relativas entre regiões do Microsoft Azure e provedores de serviços de internet
-    Exibir regras de segurança para um adaptador de rede
-    Exibir métricas de rede
-    Analisar o tráfego de ou para um grupo de segurança de rede
-    Exibir logs de diagnóstico para recursos de rede

A solução [NPM (Monitor de Desempenho de Rede)][NPM] dentro do OMS pode fornecer informações de rede detalhadas de ponta a ponta, incluindo uma exibição única das redes do Azure e das redes locais. Com monitores específicos para ExpressRoute e serviços públicos.

#### <a name="component-type-workloads"></a>Tipo de componente: cargas de trabalho
Componentes de carga de trabalho são o local em que seus aplicativos reais e serviços residem. Também é o ponto em que as equipes de desenvolvimento de aplicativo passam a maior parte do tempo.

As possibilidades de carga de trabalho são realmente infinitas. A seguir estão apenas alguns dos tipos de carga de trabalho possíveis:

**Aplicativos de LOB Internos**. Aplicativos de linha de negócios são aplicativos de computador críticos para a operação contínua de uma empresa. Aplicativos de LOB têm algumas características em comum:

-   **Interativos**. Aplicativos de LOB são interativos por natureza: os dados são inseridos e o resultado/relatórios são retornados.
-   **Conduzidos por dados**. Aplicativos de LOB fazem uso intenso de dados, com acesso frequente aos bancos de dados ou outro armazenamento.
-   **Integrados**. Aplicativos de LOB oferecem integração a outros sistemas dentro ou fora da organização.

**Clientes para sites da Web (voltados para a Internet ou internos)**. A maioria dos aplicativos que interagem com a Internet são sites da Web. O Azure oferece a capacidade de executar um site em uma VM IaaS ou de um site [Aplicativos Web do Azure][WebApps] (PaaS). Os Aplicativos Web do Azure dão suporte à integração com VNets que permitem a implantação dos Aplicativos Web no spoke de um VDC. Ao analisar sites internos, com a integração VNET, não é necessário expor um ponto de extremidade da Internet para os aplicativos, mas é possível usar os recursos através de endereços privados roteáveis não Internet a partir da sua rede privada virtual.

**Big Data/Análise de Dados** Quando os dados precisam ser escalados verticalmente para um volume muito grande, talvez os bancos de dados não sejam escalados verticalmente de modo adequado. A tecnologia Hadoop oferece um sistema para executar consultas distribuídas em paralelo em um grande número de nós. Os clientes têm a opção de executar cargas de trabalho de dados em VMs de IaaS ou PaaS ([HDInsight][HDI]). O HDInsight dá suporte à implantação em uma VNet baseada em locais, podendo ser implantada em um cluster de um spoke do VDC.

**Eventos e Mensagens**. [Os Hubs de Eventos do Azure][EventHubs] são um serviço de ingestão de telemetria de hiperescala que coleta, transforma e armazena milhões de eventos. Como uma plataforma de streaming distribuída, oferece baixa latência e tempo de retenção configurável, permitindo a ingestão de grandes quantidades de telemetria no Azure e a leitura de dados de vários aplicativos. Com Hubs de Evento, um único fluxo pode dar suporte a pipelines tanto em tempo real quanto em lote.

Um serviço de mensagem em nuvem altamente confiável entre aplicativos e serviços pode ser implementado por meio do [Barramento de Serviço do Azure][ServiceBus] que oferece sistema de mensagens agenciado assíncrono entre cliente e servidor, junto com recursos de publicar/assinar e mensagens PEPS (primeiro a entrar, primeiro a sair).

[![10]][10]

### <a name="multiple-vdc"></a>Vários VDCs
Até agora, este artigo concentrou-se em um único VDC, descrevendo os componentes básicos e a arquitetura que contribuem para um VDC resiliente. Recursos do Azure, como Azure Load Balancer, NVAs, conjuntos de disponibilidade, conjuntos de dimensionamento, junto a outros mecanismos, contribuem para um sistema que permite que você crie níveis sólidos de SLA em seus serviços de produção.

No entanto, um único VDC é hospedado em uma única região e está vulnerável a interrupções importantes que podem afetar toda uma região. Os clientes que desejam obter SLAs altos precisam proteger os serviços por meio de implantações do mesmo projeto em dois (ou mais) VDCs, colocados em regiões diferentes.

Além das questões de SLA, há vários cenários comuns em que a implantação de diversos VDCs faz sentido:

-   Presença regional/global
-   Recuperação de desastre
-   Mecanismo para desviar o tráfego entre o DC

#### <a name="regionalglobal-presence"></a>Presença regional/global
Data Centers do Azure estão presentes em várias regiões no mundo todo. Ao selecionar vários data centers do Azure, os clientes precisam considerar dois fatores relacionados: distâncias geográficas e latência. Os clientes precisam avaliar a distância geográfica entre os VDCs e a distância entre o VDC e os usuários finais para oferecer a melhor experiência de usuário.

A Região do Azure que hospeda os VDCs também precisa estar em conformidade com os requisitos normativos estabelecidos por qualquer jurisdição legal sob a qual sua organização opera.

#### <a name="disaster-recovery"></a>Recuperação de desastre
A implementação de um plano de recuperação de desastre está fortemente relacionada ao tipo de carga de trabalho em questão e à capacidade de sincronizar o estado da carga de trabalho entre diferentes VDCs. O ideal é que a maioria dos clientes queira sincronizar os dados do aplicativo entre as implantações em execução em dois VDCs diferentes para implementar um mecanismo de failover rápido. A maioria dos aplicativos é sensível à latência e isso pode causar possíveis tempos limites e atrasos na sincronização de dados.

A sincronização ou o monitoramento de pulsação de aplicativos em diferentes VDCs exige a comunicação entre eles. Dois VDCs em regiões diferentes podem ser conectados por meio de:

-   Emparelhamento VNET - O Emparelhamento VNET pode conectar hubs entre regiões
-   emparelhamento privado do ExpressRoute quando os hubs de VDC estão conectados ao mesmo circuito do ExpressRoute
-   vários circuitos do ExpressRoute conectados via backbone corporativo e sua malha VDC conectada aos circuitos do ExpressRoute
-   conexões de VPN Site a Site entre seus hubs de VDC em cada região do Azure

Geralmente, as conexões do ExpressRoute ou o Emparelhamento VNET são os mecanismos preferenciais devido à maior largura de banda e latência consistente ao transitar pelo backbone da Microsoft.

Não há nenhuma receita mágica para validar um aplicativo distribuído entre dois (ou mais) VDCs diferentes localizados em regiões diferentes. Os clientes devem executar testes de qualificação de rede para verificar a latência e a largura de banda das conexões e definir se replicação síncrona ou assíncrona de dados é apropriada e qual pode ser o RTO (objetivo de tempo de recuperação) para suas cargas de trabalho.

#### <a name="mechanism-to-divert-traffic-between-dc"></a>Mecanismo para desviar o tráfego entre o DC
Uma técnica eficaz para desviar o tráfego que entra em um DC para outro é baseada em DNS. O [Gerenciador de Tráfego do Azure][TM] usa o mecanismo de DNS (Sistema de Nomes de Domínio) para direcionar o tráfego do usuário final para o ponto de extremidade público mais apropriado em um VDC específico. Por meio de testes, o Gerenciador de Tráfego verifica periodicamente a integridade do serviço de pontos de extremidade públicos em diferentes VDCs e em caso de falha desses pontos, faz automaticamente o roteamento para o VDC secundário.

O Gerenciador de Tráfego funciona em pontos de extremidade públicos do Azure e pode ser usado, por exemplo, para controlar/desviar o tráfego para VMs do Azure e Aplicativos Web no VDC apropriado. O Gerenciador de Tráfego é resiliente mesmo em caso de falha de toda a região do Azure e pode controlar a distribuição do tráfego do usuário para pontos de extremidade do serviço em diferentes VDCs com base em diversos critérios (por exemplo, falha de um serviço em um VDC específico ou selecionando o VDC com a menor latência de rede para o cliente).

### <a name="conclusion"></a>Conclusão
O Data Center Virtual é uma abordagem à migração do data center para a nuvem que usa uma combinação de recursos e funcionalidades para criar uma arquitetura escalonável no Azure que maximiza o uso de recursos de nuvem, reduzindo os custos e simplificando a governança do sistema. O conceito de VDC baseia-se em uma topologia de spokes em hub, fornecendo serviços compartilhados comuns no hub e permitindo aplicativos/cargas de trabalho específicos nos spokes. Um VDC corresponde à estrutura de funções da empresa, em que departamentos diferentes (TI Central, DevOps, operação e manutenção) trabalham em conjunto, cada um com uma lista específica de funções e tarefas. Um VDC cumpre os requisitos para uma migração "Lift and Shift", mas também oferece muitas vantagens em implantações de nuvem nativas.

## <a name="references"></a>Referências
Os seguintes recursos foram discutidos neste documento. Clique nos links para sabe mais.

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
 - Desenvolva um modelo de gerenciamento de Assinatura e Recurso e um modelo de RBAC para atender à estrutura, aos requisitos e às políticas da sua organização. A atividade mais importante é o planejamento. Tanto quanto possível, planeje reorganizações, fusões, novas linhas de produto etc.

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

