---
title: 'Adoção do Enterprise Cloud: design de governança no Azure para várias equipes'
description: Diretrizes para configurar controles de governança do Azure para vários ambientes de várias equipes e várias cargas de trabalho
author: petertaylor9999
ms.openlocfilehash: 2ad9fac6604d2766fed1df828f63e65c8a570888
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/31/2018
ms.locfileid: "43327466"
---
# <a name="enterprise-cloud-adoption-governance-design-for-multiple-teams"></a>Adoção do Enterprise Cloud: design de governança para várias equipes

A meta deste guia é ajudar você a aprender o processo para criar um modelo de controle de recursos no Azure para dar suporte a múltiplas equipes, múltiplas cargas de trabalho e múltiplos ambientes.  Nós irá examinar um conjunto de requisitos de governança hipotético e passar por várias implementações de exemplo que atendem a esses requisitos.

> [!NOTE]
> Para uma discussão mais detalhada sobre **ambientes** consulte o 

Esses requisitos são:
* A empresa planeja uma transição de novas funções de nuvem e responsabilidades para um conjunto de usuários e, portanto, requer gerenciamento de identidades para várias equipes com necessidades de acesso de recursos diferentes no Azure. Este sistema de gerenciamento de identidade é necessário para armazenar a identidade dos seguintes usuários:
  1. A pessoa em sua organização responsável pela propriedade de **assinaturas**.
  2. O indivíduo em sua organização responsável pela **infraestrutura de recursos compartilhados** usada para conectar sua rede local a uma rede virtual do Azure. 
  3. Duas pessoas na sua organização responsáveis pelo gerenciamento de uma **carga de trabalho**. 
* Suporte para vários **ambientes**. Um ambiente é um agrupamento lógico de recursos, como máquinas virtuais, redes virtuais e serviços de roteamento de tráfego de rede. Esses grupos de recursos têm requisitos de segurança e gerenciamento semelhantes e, normalmente, são usados para uma finalidade específica, como teste ou produção. Neste exemplo, o requisito é para três ambientes:
  1. Um **ambiente de infraestrutura compartilhada** que inclui recursos compartilhados por cargas de trabalho em outros ambientes. Por exemplo, uma rede virtual com uma sub-rede do gateway que fornece conectividade para locais.
  2. Um **ambiente de produção** com as políticas de segurança mais restritivas. Pode incluir cargas de trabalho voltadas para o ambiente interno ou externo.
  3. Um **ambiente de desenvolvimento** para o trabalho de verificação de conceito e teste. Esse ambiente tem políticas de segurança, conformidade e custo que diferem no ambiente de produção.
* Um **modelo de permissões de privilégio mínimo** no qual os usuários não têm permissões por padrão. O modelo deve oferecer suporte ao seguinte:
  * Um único usuário confiável no escopo de assinatura com permissão para atribuir direitos de acesso de recursos.
  * Por padrão, todos os proprietários de carga de trabalho têm seu acesso negado aos recursos. Os direitos de acesso do recurso são concedidos explicitamente pelo usuário único confiável no escopo da assinatura.
  * Acesso de gerenciamento para os recursos de infraestrutura compartilhada limitado ao proprietário da infraestrutura compartilhada.  
  * Acesso de gerenciamento para cada carga de trabalho restrito ao proprietário da carga de trabalho.
  * A empresa não quer ter de gerenciar funções de modo independente em cada um dos três ambientes, portanto, requer o uso restrito de [funções internas de RBAC] [ rbac-built-in-roles] somente. Se a empresa utilizar funções RBAC personalizadas, um processo adicional é necessário para sincronizar as funções personalizadas nos três ambientes. 
* Custo de controle por nome de proprietário de carga de trabalho, ambiente ou ambos. 

## <a name="identity-management"></a>Gerenciamento de identidades

Antes de podermos projetar o gerenciamento de identidades para o seu modelo de controle, é importante entender as quatro áreas principais que englobam:

* Administração: os processos e as ferramentas para criação, edição e exclusão de identidade do usuário.
* Autenticação: verificando a identidade do usuário por meio da validação de credenciais, como um nome de usuário e senha.
* Autorização: determinar os recursos que um usuário autenticado tem permissão para acesso ou as operações que eles têm permissão para executar.
* Auditoria: examinar periodicamente os logs e outras informações para descobrir problemas de segurança relacionados à identidade do usuário. Isso inclui a revisão de padrões de uso suspeito, periodicamente revisando as permissões de usuário para verificar se que elas estão corretas e outras funções.

Há apenas um serviço confiável para o Azure para identidade, que é o Azure Active Directory (Azure AD). Vamos ser adicionar usuários ao Azure AD e usá-lo para todas as funções listadas acima. Mas, antes de examinarmos como configuraremos Azure AD, é importante entender as contas com privilégios que são usadas para gerenciar o acesso a esses serviços.

Quando sua organização se inscreveu para uma conta do Azure, pelo menos um **proprietário da conta** Azure foi atribuído. Além disso, um **locatário** Azure AD foi criado, a menos que um locatário existente esteja associado com o uso da sua organização de outros serviços da Microsoft como o Office 365. Um **administrador global** com permissões totais no locatário do Azure AD estava associado quando ele foi criado. 

As identidades de usuário do proprietário da conta do Azure e do administrador global do Azure AD são armazenadas em um sistema de identidade altamente seguro que é gerenciado pela Microsoft. O proprietário da conta do Azure está autorizado a criar, atualizar e excluir assinaturas. O administrador global do Azure AD está autorizado a realizar várias ações no Azure AD, mas para este guia de design vamos nos concentrar na criação e exclusão de identidade de usuário. 

> [!NOTE]
> Sua organização talvez já tenha um locatário existente do Azure AD, se houver um existente do Office 365 ou a licença do Intune associada à sua conta.

O proprietário da conta do Azure tem permissão para criar, atualizar e excluir assinaturas: 

![Conta do Azure com o Gerente de Contas do Azure e o administrador global do Azure AD](../_images/governance-3-0.png)
*Figura 1. Uma conta do Azure com um gerente de conta e o Administrador Global do Azure AD.*

O **administrador global** do Azure AD tem permissão para criar contas de usuário:  

![Conta do Azure com o Gerente de Contas do Azure e o administrador global do Azure AD](../_images/governance-3-0a.png)
*Figura 2. O Administrador Global do Azure AD cria as contas de usuário necessárias no locatário.*

As duas primeiras contas, **proprietário de carga de trabalho do app1** e **proprietário de carga de trabalho do app2** são associadas a um indivíduo em sua organização responsável por gerenciar uma carga de trabalho. A conta **operações de rede** pertence à pessoa responsável pelos recursos de infraestrutura compartilhada. Por fim, a conta do **proprietário da assinatura** está associada à pessoa responsável pela propriedade de assinaturas.

## <a name="resource-access-permissions-model-of-least-privilege"></a>Modelo de permissões de acesso do recurso de privilégio mínimo

Agora que foram criados o seu sistema de gerenciamento de identidades e suas contas de usuário, precisamos decidir como aplicaremos o RBAC (controle de acesso baseado em função) para cada conta para dar suporte a um modelo de permissões de menos privilégios.

Há outro requisito que indica que os recursos associados a cada carga de trabalho devem ser isolados uns dos outros, de modo que nenhum proprietário de uma carga de trabalho tenha acesso de gerenciamento a qualquer outra carga de trabalho que não possuem. Há também um requisito para implementar esse modelo usando somente [funções internas para o RBAC do Azure][rbac-built-in-roles].

Cada função RBAC é aplicada em um dos três escopos no Azure: **assinatura**, **grupo de recursos**, em seguida, um **recurso** individual. As funções são herdadas em escopos inferiores. Por exemplo, se um usuário é atribuído a [função proprietário interno] [ rbac-built-in-owner] no nível de assinatura, essa função também foi atribuída a esse usuário no grupo de recursos e no nível de recurso individual, a menos que ele seja substituído.

Portanto, para criar um modelo de acesso de privilégio mínimo, é necessário decidir as ações que um tipo específico de usuário pode realizar em cada um desses três escopos. Por exemplo, o requisito é para que um proprietário de carga de trabalho tenha permissão para gerenciar o acesso a somente os recursos associados à sua carga de trabalho e nenhum outro. Se tivéssemos atribuir a [função de proprietário interna][rbac-built-in-owner] no escopo de assinatura, o proprietário de cada carga de trabalho teria acesso de gerenciamento para todas as cargas de trabalho.

Vamos dar uma olhada em dois modelos de permissão de exemplo para entender esse conceito um pouco melhor. No primeiro exemplo, o modelo confia somente no administrador de serviço para criar grupos de recursos. No segundo exemplo, o modelo atribui a função de proprietário interno para o proprietário de cada carga de trabalho no escopo da assinatura. 

Nos dois exemplos, temos um administrador de serviço de assinatura que é atribuído a [função proprietário interno][rbac-built-in-owner] no escopo de assinatura. Lembre-se de que a [função de proprietário interna][rbac-built-in-owner] concede todas as permissões, incluindo o gerenciamento de acesso aos recursos.
![administrador de serviços de assinatura com a função de proprietário](../_images/governance-2-1.png)
*Figura 3. Uma assinatura com um administrador de serviços atribuídos à função de proprietário interno.* 

1. No primeiro exemplo, temos o **proprietário da carga de trabalho A** sem permissões no escopo de assinatura — ele não tem direitos de gerenciamento de acesso de recursos por padrão. Esse usuário deseja implantar e gerenciar os recursos para sua carga de trabalho. Eles devem contatar o **administrador de serviço** para solicitar a criação de um grupo de recursos.
![criação de solicitações de proprietário de carga de trabalho do grupo de recursos A](../_images/governance-2-2.png)  

2. O **administrador de serviço** examina a solicitação e cria o **grupo de recursos A**. Neste ponto, o **proprietário da carga de trabalho A** ainda não tem permissão para fazer nada.
![administrador de serviço cria o grupo de recursos A](../_images/governance-2-3.png)

3. O **administrador de serviço** adiciona **proprietário da carga de trabalho A** para **do grupo de recursos A** e atribui o [função colaborador interna](/azure/role-based-access-control/built-in-roles#contributor). A função de colaborador concede todas as permissões no **grupo de recursos A**, exceto o gerenciamento de permissão de acesso.
![O administrador de serviços adiciona o proprietário da carga de trabalho a para o grupo de recursos a](../_images/governance-2-4.png)

4. Vamos supor que o **proprietário da carga de trabalho A** tenha um requisito para um par de membros da equipe para exibir o CPU e o tráfego de rede de dados como parte do planejamento de capacidade para a carga de trabalho de monitoramento. Como o **proprietário da carga de trabalho A** é atribuído à função de colaborador, ele não tem permissão para adicionar um usuário ao **grupo de recursos A**. Eles devem enviar esta solicitação para o **administrador de serviços**.
![o proprietário da carga de trabalho solicita que colaboradores de carga de trabalho sejam adicionados ao grupo de recursos](../_images/governance-2-5.png)

5. O **administrador de serviço** examina a solicitação e adiciona os dois **colaborador da carga de trabalho** usuários **do grupo de recursos A**. Nenhum desses dois usuários exige a permissão para gerenciar recursos, portanto, eles recebem a [função de leitor interna](/azure/role-based-access-control/built-in-roles#contributor). 
![o administrador de serviço adiciona colaboradores da carga de trabalho ao grupo de recursos A](../_images/governance-2-6.png)

6. Em seguida, o **proprietário da carga de trabalho B** também requer um grupo de recursos conter os recursos para sua carga de trabalho. Assim como acontece com **proprietário da carga de trabalho A**, **proprietário da carga de trabalho B** inicialmente não tem permissão para realizar qualquer ação no escopo de assinatura para que eles devem enviar uma solicitação para o **administrador de serviços**. 
![criação de solicitações de proprietário de carga de trabalho B do grupo de recursos B](../_images/governance-2-7.png)

7. O **administrador de serviço** examina a solicitação e cria o **grupo de recursos B**. ![Administrador de Serviços cria o grupo de recursos B](../_images/governance-2-8.png)

8. O **administrador de serviço** então adiciona o **proprietário da carga de trabalho B** ao **grupo de recursos B** e atribui a [função colaborador interna](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#contributor). 
![O administrador de serviços adiciona o proprietário da carga de trabalho B ao grupo de recursos B](../_images/governance-2-9.png)

Neste ponto, cada um dos proprietários da carga de trabalho é isolado em seu próprio grupo de recursos. Nenhum dos proprietários da carga de trabalho ou membros da equipe tem acesso de gerenciamento para os recursos em outro grupo de recursos. 

![assinatura com grupos de recursos A e B](../_images/governance-2-10.png)
*Figura 4. Uma assinatura com dois proprietários da carga de trabalho isolada com seu próprio grupo de recursos.*

Este é um modelo de privilégio mínimo - cada usuário recebe a permissão correta no escopo de gerenciamento do recurso correto.

No entanto, considere a possibilidade de que todas as tarefas neste exemplo sejam executadas pelo **administrador de serviço**. Enquanto esse é um exemplo simples e não parece ser um problema porque não havia apenas dois proprietários da carga de trabalho, é fácil imaginar que os tipos de problemas que possam resultar de uma organização de grande porte. Por exemplo, o **administrador de serviço** pode se tornar um afunilamento com uma grande lista de pendências de solicitações que resultar em atrasos.  

Vamos dar uma olhada no segundo exemplo que reduz o número de tarefas executadas pelo **administrador de serviço**. 

1. Nesse modelo, **proprietário da carga de trabalho A** é atribuído a [função proprietário interno][rbac-built-in-owner] no escopo de assinatura, permitindo que eles criem seu próprios grupo de recursos: **grupo de recursos A**. ![O administrador de serviços adiciona um proprietário da carga de trabalho A à assinatura](../_images/governance-2-11.png)

2. Quando **do grupo de recursos A** é criado, **proprietário da carga de trabalho A** é adicionado por padrão e herda o [proprietário interno] [ rbac-built-in-owner] função do escopo de assinatura.
![O proprietário da carga de trabalho A cria o grupo de recursos A](../_images/governance-2-12.png)

3. A [função interna de proprietário][rbac-built-in-owner] concede ao **proprietário da carga de trabalho A** permissão para gerenciar o acesso ao grupo de recursos. O **proprietário da carga de trabalho A** adiciona dois **colaboradores da carga de trabalho** e atribui a [função leitor interna][rbac-built-in-owner] para cada um deles. 
![Proprietário da carga de trabalho A adiciona colaboradores da carga de trabalho](../_images/governance-2-13.png)

4. O **administrador de serviço** agora adiciona o **proprietário da carga de trabalho B** à assinatura com a função de proprietário interno. 
![O administrador de serviços adiciona proprietários da carga de trabalho B à assinatura](../_images/governance-2-14.png)

5. O **proprietário da carga de trabalho B** cria o **grupo de recursos B** e é adicionado por padrão. Novamente, o **proprietário da carga de trabalho B** herda a função de proprietário interno do escopo da assinatura.
![O proprietário da carga de trabalho B cria o grupo de recursos B](../_images/governance-2-15.png)

Observe que, neste modelo, o **administrador de serviço** executou ações menos do que no primeiro exemplo, devido a delegação de acesso de gerenciamento para cada um dos proprietários da carga de trabalho individuais.

![assinatura com grupos de recursos A e B](../_images/governance-2-16.png)
*Figura 5. Uma assinatura com um administrador de serviço e dois proprietários da carga de trabalho, todos atribuídos à função de proprietário interno.*

No entanto, porque ambos **proprietário da carga de trabalho A** e **proprietário da carga de trabalho B** são atribuídos à função de proprietário interno no escopo de assinatura, eles terão cada herdadas a função de proprietário interno para cada um dos outros recursos grupo. Isso significa que não só têm acesso total aos recursos do outro, eles também são capazes de delegar acesso de gerenciamento para cada um dos outros grupos de recursos. Por exemplo, **proprietário da carga de trabalho B** tem direitos para adicionar qualquer outro usuário para **do grupo de recursos A** e pode atribuir qualquer função a eles, incluindo a função de proprietário interno.

Se compararmos cada exemplo aos requisitos, podemos ver que ambos os exemplos dão suporte a um único usuário confiável no escopo da assinatura com permissão para conceder direitos de acesso de recursos para os dois proprietários de cargas de trabalho. Cada um dos proprietários de duas cargas de trabalho não tinham acesso ao gerenciamento de recursos por padrão e exigiam que o **administrador de serviço** atribuísse explicitamente permissões a eles. No entanto, somente o primeiro exemplo suporta o requisito de que os recursos associados a cada carga de trabalho são isolados uma da outra, de modo que nenhum proprietário da carga de trabalho tem acesso aos recursos de qualquer outra carga de trabalho.

## <a name="resource-management-model"></a>Modelo de gerenciamento de recursos

Agora que criamos um modelo de permissões de privilégios mínimos, vamos dar uma olhada em algumas aplicações desses modelos de controle. Lembre-se de que os requisitos devem dar suporte aos três ambientes a seguir:
1. **Infraestrutura compartilhada:** um único grupo de recursos compartilhados por todas as cargas de trabalho. Estes são recursos como gateways de rede, firewalls e serviços de segurança.  
2. **Desenvolvimento:** vários grupos de recursos que representam várias cargas de trabalho prontas para não produção. Esses recursos são usados para verificação de conceito, teste e outras atividades de desenvolvedor. Esses recursos podem ter um modelo de governança mais flexível para permitir maior agilidade do desenvolvedor.
3. **Produção:** vários grupos de recursos que representam várias cargas de trabalho de produção. Esses recursos são usados para hospedar os artefatos de aplicativo voltados para o público e o privado. Esses recursos normalmente têm o controle mais completa e modelos de segurança para proteger os recursos, o código do aplicativo e dados contra acesso não autorizado.

Para cada um desses três ambientes, temos um requisito para controlar dados de custo por **proprietário da carga de trabalho**, **ambiente**, ou ambos. Ou seja, gostaríamos de saber o custo em andamento da **infraestrutura compartilhada**, os custos incorridos por indivíduos em ambos os ambientes de **desenvolvimento** e **produção** e, finalmente, o custo geral de **desenvolvimento** e **produção**. 

Você já aprendeu que os recursos estão no escopo em dois níveis: **assinatura** e **grupo de recursos**. Portanto, a primeira decisão é como organizar os ambientes por **assinatura**. Há apenas duas possibilidades: uma única assinatura ou várias assinaturas. 

Antes de examinarmos exemplos de cada um desses modelos, vamos examinar a estrutura de gerenciamento de assinaturas no Azure. 

Lembre-se dos requisitos de que temos um indivíduo na organização que é responsável por assinaturas, e esse usuário possui a conta do **proprietário da assinatura** no locatário do Azure AD. No entanto, essa conta não tem permissão para criar assinaturas. Somente o **Proprietário da Conta do Azure** tem permissão para fazer isso: ![](../_images/governance-3-0b.png)
*Figura 6. Um proprietário de conta do Azure cria uma assinatura.*

Quando a assinatura tiver sido criada, o **proprietário da conta do Azure** pode adicionar o **proprietário da assinatura** de conta para a assinatura com o **proprietário** função:

![](../_images/governance-3-0c.png)
*Figura 7. O proprietário da conta do Azure adiciona a conta de usuário do **proprietário da assinatura** para a assinatura com a função **proprietário**.*

O **proprietário da assinatura** agora pode criar **grupos de recursos** e delegar o gerenciamento de acesso de recursos.

Primeiro vamos examinar um modelo de gerenciamento de recursos de exemplo usando uma única assinatura. A primeira decisão é como alinhar grupos de recursos para os três ambientes. Nós temos duas opções:
1. Alinhe cada ambiente a um único grupo de recursos. Todos os recursos de infraestrutura compartilhada são implantados em um único grupo de recursos de **infraestrutura compartilhada**. Todos os recursos associados com cargas de trabalho de desenvolvimento são implantados em um único grupo de recursos de **desenvolvimento**. Todos os recursos associados com cargas de trabalho de produção são implantados em um único grupo de recursos de **produção** para o ambiente de **produção**. 
2. Crie grupos de recursos separados para cada carga de trabalho, utilizando uma convenção de nomenclatura e marcas para alinhar grupos de recursos com cada um dos três ambientes.  

Vamos começar avaliando a primeira opção. Usaremos o modelo de permissões, discutido na seção anterior, com um administrador de serviço de assinatura único que cria grupos de recursos e adiciona os usuários a eles com a função de **colaborador** interno ou de **leitor**. 

1. O primeiro grupo de recursos implantados representa o ambiente de **infraestrutura compartilhada**. O **proprietário da assinatura** cria um grupo de recursos para os recursos de infraestrutura compartilhada denominados **netops-shared-rg**. 
![](../_images/governance-3-0d.png)
2. O **proprietário da assinatura** adiciona a conta de **usuário operações de rede** ao grupo de recursos e atribui a função **colaborador**. 
![](../_images/governance-3-0e.png)
3. O **usuário de operações de rede** cria um [gateway VPN](/azure/vpn-gateway/vpn-gateway-about-vpngateways) e o configura para se conectar ao dispositivo de VPN local. O usuário **operações de rede** também se aplica a um par de [marcas](/azure/azure-resource-manager/resource-group-using-tags) para cada um dos recursos: *environment:shared* e *managedBy:netOps*. Quando o **administrador de serviço de assinatura** exportar um custo de relatório, os custos serão alinhados com cada uma dessas marcas. Isso permite que o **administrador de serviço de assinatura** dinamizar os custos usando a marca *environment* e a marca *managedBy*. Observe o contador de **limites de recursos** no lado superior direito da figura. Cada assinatura do Azure tem [limites de serviço](/azure/azure-subscription-service-limits), e para ajudá-lo a entender o efeito desses limites Seguiremos o limite de rede virtual para cada assinatura. Há um limite padrão de 50 redes virtuais por assinatura e depois que a primeira rede virtual é implantada agora há 49 disponíveis.
![](../_images/governance-3-1.png)
4. Dois ou mais grupos de recursos são implantados, o primeiro é denominado *prod rg*. Este grupo de recursos está alinhado com o ambiente de **produção**. O segundo é denominado *dev-rg* e é alinhado ao ambiente de **desenvolvimento**. Todos os recursos associados com cargas de trabalho de produção são implantados para o **produção** todos os recursos associados com cargas de trabalho de desenvolvimento e ambiente são implantados para o ambiente de **desenvolvimento**. Neste exemplo, só implantaremos duas cargas de trabalho para cada um desses dois ambientes para não encontramos qualquer limite de serviço de assinatura do Azure. No entanto, é importante considerar que cada grupo de recursos tem um limite de 800 recursos por grupo de recursos. Portanto, se pudermos continuar adicionando cargas de trabalho para cada grupo de recursos, é possível que esse limite seja atingido. 
![](../_images/governance-3-2.png)
5. A primeira **proprietário da carga de trabalho** envia uma solicitação para o **administrador de serviço de assinatura** e é adicionado a cada um dos grupos de recursos dos ambientes de **desenvolvimento** e de **produção** com a função **colaborador**. Como você aprendeu anteriormente, o **colaborador** função permite que o usuário execute qualquer operação que não seja a atribuição de uma função para outro usuário. O primeiro **proprietário da carga de trabalho** agora pode criar os recursos associados à sua carga de trabalho.
![](../_images/governance-3-3.png)
6. O primeiro **proprietário da carga de trabalho** cria uma rede virtual em cada dois grupos de recursos com um par de máquinas virtuais em cada um. O primeiro **proprietário da carga de trabalho** se aplica às marcas *environment* e *managedBy* para todos os recursos. Observe que o contador de limite do serviço do Azure está agora em 47 redes virtuais restantes.
![](../_images/governance-3-4.png)
7. Nenhuma das redes virtuais não tem conectividade com o local quando é criada. Nesse tipo de arquitetura, cada rede virtual deve ser emparelhada para a *hub-vnet* no ambiente de **infraestrutura compartilhada**. O emparelhamento de rede virtual cria uma conexão entre duas redes virtuais separadas e permite o tráfego de rede entre eles. Observe que o emparelhamento de rede virtual não é inerentemente transitiva. Um emparelhamento deve ser especificado em cada uma das duas redes virtuais que estão conectadas e se apenas uma das redes virtuais Especifica um emparelhamento a conexão está incompleta. Para ilustrar o efeito disso, o primeiro **proprietário da carga de trabalho** Especifica um emparelhamento entre **prod-vnet** e **hub-vnet**. O emparelhamento primeiro é criado, mas nenhum tráfego flui, já que o emparelhamento complementar do **hub-vnet** para **prod-vnet** ainda não foi especificado. A primeira **proprietário da carga de trabalho** contatos a **operações de rede** solicitações nesse complementares emparelhamento de conexão e usuário.
![](../_images/governance-3-5.png)
8. O usuário **operações de rede** examina a solicitação, a aprova e então especifica o emparelhamento nas configurações para o **hub-vnet**. A conexão de emparelhamento agora está concluída e o tráfego de rede flui entre as duas redes virtuais.
![](../_images/governance-3-6.png)
9. Agora, um segundo **proprietário da carga de trabalho** envia uma solicitação para o **administrador de serviço de assinatura** e é adicionado à existente **produção** e **desenvolvimento**  grupos de recurso de ambiente com o **colaborador** função. O segundo **proprietário da carga de trabalho** tem as mesmas permissões em todos os recursos como o primeiro **proprietário da carga de trabalho** em cada grupo de recursos. 
![](../_images/governance-3-7.png)
10. O segundo **proprietário da carga de trabalho** cria uma sub-rede na rede virtual **prod-vnet** e então adicionada duas máquinas virtuais. O segundo **proprietário da carga de trabalho** se aplica às marcas *environment* e *managedBy* para cada recurso.
![](../_images/governance-3-8.png) 

Esse modelo de gerenciamento de recursos de exemplo nos permite gerenciar recursos nos três ambientes necessários. Os recursos de infraestrutura compartilhada estão protegidos porque há apenas um usuário na assinatura com a permissão para acessar esses recursos. Cada um dos proprietários da carga de trabalho é capaz de utilizar os recursos de infraestrutura de compartilhamento sem ter qualquer permissão nos próprios recursos compartilhados. No entanto, esse modelo de gerenciamento falha no requisito para o isolamento de carga de trabalho — cada um dos dois **proprietários de carga de trabalho** é capaz de acessar os recursos de carga de trabalho do outro. 

Há outra consideração importante com esse modelo, que pode não ser imediatamente óbvia. No exemplo, foi o **proprietário de carga de trabalho do app1** que solicitou a conexão de rede emparelhamento com o **hub-vnet** para fornecer conectividade a locais. O usuário **operações de rede** avaliou a solicitação com base nos recursos implantados com carga de trabalho. Quando o **proprietário da assinatura** adicionou o **proprietário de carga de trabalho do app2** com a função **colaborador**, o usuário tinha direitos de acesso de gerenciamento para todos os recursos no grupo de recursos **prod-rg**. 

![](../_images/governance-3-10.png)

Isso significa que o **proprietário de carga de trabalho do app2** teve permissão para implantar sua própria sub-rede com máquinas virtuais na rede virtual **redes de produção**. Por padrão, essas máquinas virtuais agora têm acesso à rede local. O usuário **operações de rede** não está ciente dessas máquinas e não aprova sua conectividade com local.

Em seguida, vamos examinar uma única assinatura com vários grupos de recursos para diferentes ambientes e cargas de trabalho. Observe que no exemplo anterior, os recursos para cada ambiente foram facilmente identificáveis porque estavam no mesmo grupo de recursos. Agora que não temos mais esse agrupamento, precisamos contar com uma convenção de nomenclatura do grupo de recursos para fornecer essa funcionalidade. 

1. Os recursos de **infraestrutura compartilhada** ainda terão um grupo de recursos separado nesse modelo para que ele permaneça igual. Cada carga de trabalho exige dois grupos de recursos, um para cada um dos ambientes de **desenvolvimento** e de **produção**. Para a primeira carga de trabalho, o **proprietário da assinatura** cria dois grupos de recursos. A primeira é denominada **app1-prod-rg** e a segunda é denominada **app1-dev-rg**. Como discutido anteriormente, essa convenção de nomenclatura identifica os recursos como sendo associados à primeira carga de trabalho, **app1**, e o ambiente de **desenvolvimento** ou de **produção**. Novamente, o proprietário da *assinatura* adiciona o **proprietário de carga de trabalho do app1** ao grupo de recursos com a função **colaborador**.
![](../_images/governance-3-12.png)
2. Semelhante ao primeiro exemplo, **proprietário de carga de trabalho do app1** implanta uma rede virtual denominada **app1-prod-vnet** para o **produção** ambiente e outro nomeado**app1-dev-vnet** para o **desenvolvimento** ambiente. Novamente, o **proprietário de carga de trabalho do app1** envia uma solicitação para o usuário **operações de rede** para criar uma conexão de emparelhamento. Observe que o **proprietário de carga de trabalho do app1** adiciona as mesmas marcas como no primeiro exemplo e o limite de contador foi reduzido para 47 redes virtuais restantes na assinatura.
![](../_images/governance-3-13.png)
3. Agora, o **proprietário da assinatura** cria dois grupos de recursos para o **proprietário de carga de trabalho do app2**. Seguir as mesmas convenções de **proprietário de carga de trabalho do app1**, os grupos de recursos são nomeados como **app2-prod-rg** e **app2-dev-rg**. O **proprietário da assinatura** adiciona **proprietário de carga de trabalho do app2** para cada um dos grupos de recursos com a função **colaborador**.
![](../_images/governance-3-14.png)
4. O *proprietário da carga de trabalho de app2* implanta redes virtuais e máquinas virtuais para os grupos de recursos com as mesmas convenções de nomenclatura. As marcas são adicionadas e o contador de limite foi reduzido para 45 redes virtuais restantes na *assinatura*.
![](../_images/governance-3-15.png)
5. O *proprietário da carga de trabalho do app2* envia uma solicitação para o usuário *operações de rede* para o mesmo nível a *app2-prod-vnet* com o *hub vnet*. O usuário de *operações de rede* cria a conexão de emparelhamento.
![](../_images/governance-3-16.png)

O modelo de gerenciamento resultante é semelhante ao primeiro exemplo, com várias diferenças importantes:
* Cada uma das duas cargas de trabalho é isolada por carga de trabalho e pelo ambiente.
* Esse modelo exigia duas redes virtuais a mais do que o primeiro modelo de exemplo. Enquanto isso não é uma distinção importante com apenas duas cargas de trabalho, o limite teórico no número de cargas de trabalho para esse modelo é 24. 
* Os recursos não são agrupados em um único grupo de recursos para cada ambiente. O agrupamento de recursos requer uma compreensão das convenções de nomenclatura usadas para cada ambiente. 
* Cada uma das conexões de rede virtual emparelhadas foi revisada e aprovada pelo usuário *operações de rede*.

Agora vamos dar uma olhada em um modelo de gerenciamento de recursos usando várias assinaturas. Nesse modelo, cada um dos três ambientes para uma assinatura separada será alinhar: uma **serviços compartilhados** assinatura, **produção** assinatura e, finalmente, um **desenvolvimento** assinatura. As considerações para este modelo são semelhantes a um modelo usando uma única assinatura em que temos que decidir como alinhar grupos de recursos para cargas de trabalho. Nós já determinamos que criar um grupo de recursos para cada carga de trabalho satisfaz o requisito de isolamento de carga de trabalho, portanto, ficaremos com esse modelo nesse exemplo.

1. Nesse modelo, há três *assinaturas*: *infraestrutura compartilhada*, *produção* e *desenvolvimento*. Cada uma dessas três assinaturas requer um *proprietário da assinatura* e, no exemplo simples, vamos usar a mesma conta de usuário para todos os três. O *infraestrutura compartilhada* recursos são gerenciados da mesma forma para os dois primeiros exemplos acima, e a primeira carga de trabalho está associada a *app1-rg* no *produção* ambiente e o recurso de mesmo nome de grupo no *desenvolvimento* ambiente. O *proprietário de carga de trabalho do app1* é adicionado a cada grupo de recursos com a função *colaborador*. 
![](../_images/governance-3-17.png)
2. Assim como acontece com os exemplos anteriores, *proprietário de carga de trabalho do app1* cria os recursos e as solicitações de conexão de emparelhamento com o *infraestrutura compartilhada* rede virtual. O *proprietário de carga de trabalho do app1* adiciona somente a marca *managedBy* porque não há mais necessidade para a marca *environment*. Ou seja, os recursos para cada ambiente agora são agrupados na mesma *assinatura* e a marca *environment* fica redundante. O contador de limites será diminuído para 49 redes virtuais restantes.
![](../_images/governance-3-18.png)
3. Por fim, o *proprietário da assinatura* repete o processo para a carga de trabalho segundo, adicionando os grupos de recursos com o *app2 proprietário da carga de trabalho* no * função de colaborador. O contador de limite para cada uma das assinaturas do ambiente é reduzido para 48 redes virtuais restantes. 

Esse modelo de gerenciamento tem os benefícios do segundo exemplo acima. No entanto, a principal diferença é que os limites são um problema devido ao fato de que eles estão espalhados por duas *assinaturas*. A desvantagem é que os dados de custo controlados por marcas devem ser agregados em todas as três *assinaturas*. 

Portanto, você pode selecionar qualquer um desses dois modelos de gerenciamento de recursos de exemplo, dependendo da prioridade de seus requisitos. Se você antecipar que sua organização não atinja os limites de serviço para uma única assinatura, poderá usar uma única assinatura com vários grupos de recursos. Por outro lado, se sua organização prevê várias cargas de trabalho, pode ser melhor ter várias assinaturas para cada ambiente.

## <a name="implementing-the-resource-management-model"></a>Implementação do modelo de gerenciamento de recursos

Você aprendeu sobre vários modelos diferentes para controlar o acesso aos recursos do Azure. Agora vamos examinar as etapas necessárias para implementar o modelo de gerenciamento de recursos com uma assinatura para cada uma da **infraestrutura compartilhada**, **produção**, e **desenvolvimento** ambientes do guia de design. Teremos um **proprietário da assinatura** para todos os três ambientes. Cada carga de trabalho será isolada em um **grupo de recursos** com um **proprietário da carga de trabalho** adicionado com a função **colaborador**.

> [!NOTE]
> Leia as [noções básicas sobre o acesso a recursos no Azure][understand-resource-access-in-azure] para saber mais sobre a relação entre as Contas e a assinatura do Azure. 

Siga estas etapas:

1. Crie uma [conta do Azure](/azure/active-directory/sign-up-organization) se sua organização ainda não tiver uma. A pessoa que se inscreve na conta do Azure se torna o administrador da conta do Azure e liderança de sua organização deve selecionar um indivíduo para assumir essa função. Essa pessoa será responsável por:
    * Criação de assinaturas e
    * Criando e administrando locatários do [Azure Active Directory (AD)](/azure/active-directory/active-directory-whatis) que armazenam a identidade do usuário para as assinaturas.    
2. A equipe de liderança de sua organização decide quais pessoas são responsáveis por:
    * Gerenciamento de identidade do usuário. um [locatário do Azure AD](/azure/active-directory/develop/active-directory-howto-tenant) é criado por padrão quando a conta do Azure da sua organização é criada, e o administrador da conta é adicionado como o [administrador global do Azure AD](/azure/active-directory/active-directory-assign-admin-roles-azure-portal#details-about-the-global-administrator-role) por padrão. Sua organização pode escolher outro usuário para gerenciar a identidade do usuário por [atribuir a função de administrador global do Azure AD para que o usuário](/azure/active-directory/active-directory-users-assign-role-azure-portal). 
    * Assinaturas, o que significa que esses usuários:
        * Gerencie os custos associados ao uso de recursos nessa assinatura,
        * Implemente e mantenha o modelo de permissão mínima para acesso a recursos e
        * Controle os limites de serviço.
    * Infraestrutura de serviços compartilhados (se a sua organização decidir usar esse modelo), o que significa que esse usuário é responsável por:
        * Local para conectividade de rede do Azure e 
        * Propriedade de conectividade de rede no Azure por meio do emparelhamento de rede virtual.
    * Proprietários da carga de trabalho. 
3. O administrador global do Azure AD [cria novas contas de usuário](/azure/active-directory/add-users-azure-active-directory) para:
    * A pessoa que será o **proprietário da assinatura** para cada assinatura associada a cada ambiente. Observe que isso é necessário apenas se o **administrador de serviço** da assinatura não for o responsável por gerenciar o acesso a recursos para cada assinatura/ambiente.
    * A pessoa que será o **usuário de operações de rede** e
    * As pessoas que são **proprietárias da carga de trabalho**.
4. O administrador da conta do Azure cria as seguintes três assinaturas usando o [portal de conta do Azure](https://account.azure.com):
    * Uma assinatura para o ambiente de **infraestrutura compartilhada**,
    * Uma assinatura para o ambiente de **produção** e 
    * Uma assinatura para o ambiente de **desenvolvimento**. 
5. O administrador da conta do Azure [adiciona o proprietário do serviço de assinatura para cada assinatura](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-admin-for-a-subscription-in-azure-portal).
6. Crie um processo de aprovação para **proprietários da carga de trabalho** para solicitar a criação de grupos de recursos. O processo de aprovação pode ser implementado de diversas maneiras, como por email, ou você pode usar uma ferramenta de gerenciamento do processo como [fluxos de trabalho do](https://support.office.com/article/introduction-to-sharepoint-workflow-07982276-54e8-4e17-8699-5056eff4d9e3). O processo de aprovação pode seguir estas etapas:  
    * O **proprietário da carga de trabalho** prepara uma lista de materiais dos recursos do Azure necessários no **desenvolvimento** ambiente, **produção** ambiente, ou ambos e envia para o **proprietário da assinatura**.
    * O **proprietário da assinatura** examina a lista de materiais e valida os recursos solicitados para garantir que os recursos solicitados são apropriados para seu uso planejado - por exemplo, a verificação de que a solicitação [ tamanhos de máquina virtual](/azure/virtual-machines/windows/sizes) estão corretas.
    * Se a solicitação não for aprovada, o **proprietário da carga de trabalho** será notificado. Se a solicitação for aprovada, o **proprietário da assinatura** [cria o grupo de recurso solicitado](/azure/azure-resource-manager/resource-group-portal#manage-resource-groups) após sua organização [convenções de nomenclatura](/azure/architecture/best-practices/naming-conventions), [ Adiciona o **proprietário da carga de trabalho** ](/azure/role-based-access-control/role-assignments-portal#add-access) com o [ **colaborador** função](/azure/role-based-access-control/built-in-roles#contributor) e envia uma notificação para o **proprietário de carga de trabalho** que o grupo de recursos foi criado.
7. Crie um processo de aprovação para os proprietários da carga de trabalho solicitar uma emparelhamento de conexão do proprietário infraestrutura compartilhada de rede virtual. Assim como acontece com a etapa anterior, esse processo de aprovação pode ser implementado usando email ou uma ferramenta de gerenciamento de processos.

Agora que você já implementou o seu modelo de controle, poderá implantar seus serviços de infraestrutura compartilhada.

## <a name="next-steps"></a>Próximas etapas
> [!div class="nextstepaction"]
> [Saiba mais sobre como implantar uma infraestrutura básica](../infrastructure/basic-workload.md)

<!-- links -->
[understand-resource-access-in-azure]: /azure/role-based-access-control/rbac-and-directory-admin-roles

<!-- links -->

[rbac-built-in-owner]: /azure/role-based-access-control/built-in-roles#owner
[rbac-built-in-roles]: /azure/role-based-access-control/built-in-roles
