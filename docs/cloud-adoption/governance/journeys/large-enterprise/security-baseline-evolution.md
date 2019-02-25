---
title: 'CAF: Empresa de grande porte – evolução da linha de base de segurança '
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Empresa de grande porte – evolução da linha de base de segurança
author: BrianBlanchard
ms.openlocfilehash: 59fb3655f1ff2a5f0a30abc760c27c77b8f802af
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900391"
---
# <a name="large-enterprise-security-baseline-evolution"></a>Empresas de grande porte: Evolução da linha de base de segurança

Este artigo desenvolve a narrativa adicionando controles de segurança que dão suporte à transferência de dados protegidos para a nuvem.

## <a name="evolution-of-the-narrative"></a>Evolução da narrativa

O CIO passou meses colaborando com colegas e a equipe jurídica da empresa. Um consultor de gerenciamento com experiência em segurança cibernética foi envolvido para ajudar a segurança de TI existente e as equipes de governança de TI a esboçar uma nova política em relação a dados protegidos. O grupo foi capaz de promover o suporte de quadro para substituir a política existente, permitindo PII e dados financeiros para serem armazenados pelos provedores de nuvem aprovados. Isso exigia a adoção de um conjunto de requisitos de segurança e um processo de controle para verificar e documentar a adesão a essas políticas.

Nos últimos 12 meses, as equipes de adoção de nuvem limparam a maioria dos ativos de 5.000 datacenters que serão desativados. 350 ativos incompatíveis foram movidos para um datacenter alternativo. Somente 1.250 máquinas virtuais que contêm dados protegidos.

### <a name="evolution-of-the-cloud-governance-team"></a>Evolução da equipe de Governança de Nuvem

A equipe de governança de nuvem continua a evoluir junto com a narrativa. Os dois membros fundadores da equipe agora estão entre os arquitetos de nuvem mais respeitados na empresa. A coleção de scripts de configuração cresceu à medida que novas equipes lidam com novas implantações inovadoras. A Equipe de Governança de Nuvem também cresceu. Mais recentemente, os membros da equipe de operações de TI ingressaram as atividades da equipe de Governança de nuvem para se preparar para operações na nuvem. Os arquitetos de nuvem que ajudam a criar essa comunidade são vistos como guardiões de nuvem e aceleradores de nuvem.

Embora a diferença seja sutil, é uma distinção importante ao criar uma cultura de TI focada em governança. Um guardião de nuvem limpa a bagunça feita pelos arquitetos de nuvem inovadores e as duas funções têm atrito natural e objetivos opostos. Por outro lado, um Guardião da Nuvem ajuda a proteger a nuvem para que outros arquitetos de nuvem possam trabalhar mais rapidamente, com menos complicações. Um acelerador de nuvem executa ambas as funções, mas também está envolvido na criação de modelos para acelerar a implantação e a adoção, tornando-se um acelerador de inovação, bem como um defensor cinco disciplinas de nuvem.

### <a name="evolution-of-the-current-state"></a>Evolução do estado atual

Na fase anterior desse narrativa, a empresa começou o processo de desativação de dois data centers. Esse esforço contínuo inclui a migração de alguns aplicativos com requisitos de autenticação herdados, que exigiam uma evolução da linha de base de identidade, descrito na [artigo anterior](identity-baseline-evolution.md).

Desde então, ocorreram algumas mudanças que afetarão a governança:

- Milhares de TI e ativos de negócios foram implantados para a nuvem.
- A equipe de desenvolvimento de aplicativos implementou uma integração contínua e um pipeline de integração contínua (CI/CD) para implantar uma aplicação nativa de nuvem com uma experiência de usuário aprimorada. Esse aplicativo ainda não interage com os dados protegidos, portanto, não está pronto para produção.
- A equipe de Business Intelligence dentro da TI cuida ativamente dos dados de logística, inventário e terceiros na nuvem. Esses dados estão sendo usados para gerar novas previsões, o que poderia moldar os processos de negócios. No entanto, essas previsões e informações não são acionáveis até que os dados financeiros e do cliente possam ser integrados à plataforma de dados.
- A equipe de TI está em andamento nos planos do CIO e CFO para desativar dois datacenters. Quase 3.500 ativos no datacenter de recuperação de desastre foram desativados ou migrados.
- As políticas em relação à PII e dados financeiros foram modernizadas. No entanto, as novas políticas corporativas dependem da implementação das políticas de segurança e governança relacionadas. As equipes ainda estão trabalhando nisso.

### <a name="evolution-of-the-future-state"></a>Evolução do estado futuro

- Experimentos iniciais pelas equipes de desenvolvimento de aplicativos e de BI mostraram possíveis aprimoramentos nas experiências do cliente e nas decisões orientadas a dados. As duas equipes querem expandir a adoção da nuvem nos próximos 18 meses ao implantar essas soluções na produção.
- A TI desenvolveu uma justificativa de negócios para migrar cinco mais datacenters para Azure, que irá diminuir os custos de TI e fornecer maior agilidade de negócios. Embora menores em escala, a desativação dos data centers é esperada para dobrar as economias de custo total.
- Despesas de capital e orçamentos de despesas operacionais aprovaram para implementar as políticas de segurança e governança necessárias, ferramentas e processos. As economias de custo esperadas da desativação do datacenter são mais do que suficiente para pagar por essa nova iniciativa. TI e liderança de negócios estão confiantes que esse investimento acelerará a realização de retornos em outras áreas. A equipe de governança de nuvem cultivada tornou-se uma equipe reconhecida com liderança dedicada e pessoal.
- Coletivamente, a equipes de adoção de nuvem, a equipe de governança de nuvem, a equipe de segurança de TI e a equipe de governança de TI irá implementar requisitos de governança e segurança para permitir as equipes de adoção para migrar dados protegidos para a nuvem.

## <a name="evolution-of-tangible-risks"></a>Evolução de riscos tangíveis

**Violação de dados**: Há um aumento inerente nos passivos relacionados a possíveis violações de dados ao adotar qualquer nova plataforma de dados. Os técnicos que adotam as tecnologias de nuvem possuem maiores responsabilidades de implementar soluções que possam diminuir esse risco. Uma estratégia de governança e segurança robusta deve ser implementada para garantir que os técnicos cumpram essas responsabilidades.

Esse risco de negócios pode ser dividido em alguns riscos técnicos:

- Os aplicativos de missão crítica ou dados protegidos podem ser implantados acidentalmente.
- Dados protegidos podem ser expostos durante o armazenamento devido a más decisões de criptografia.
- Os usuários não autorizados podem acessar dados protegidos.
- Uma invasão externa pode resultar no acesso aos dados protegidos.
- Uma invasão externa ou ataques de negação de serviço pode causar uma interrupção dos negócios.
- As alterações de emprego ou da organização podem resultar em acesso não autorizado aos dados protegidos.
- Novas explorações podem criar oportunidades de invasão ou acesso não autorizado.
- Os processos de implantação podem resultar em falhas de segurança, o que pode levar a perdas de dados ou interrupções.
- A inconsistência na configuração ou patches ausentes podem resultar em falhas de segurança não intencionais, o que pode levar a vazamentos de dados ou interrupções.
- Dispositivos de borda diferentes podem aumentar os custos das operações de rede.
- Configurações de dispositivo diferentes podem levar a descuidos na configuração e comprometimentos de segurança.
- A equipe de segurança cibernética insiste que há um risco de bloqueio de fornecedor gerando chaves de criptografia na plataforma de um único provedor de nuvem. Enquanto essa declaração é infundada, foi aceita pela equipe por enquanto.

## <a name="evolution-of-the-policy-statements"></a>Evolução das instruções da política

As seguintes alterações à política ajudam a reduzir novos riscos e a orientar a implementação. A lista parece longa, mas a adoção dessas políticas pode ser mais fácil do que parece.

1. Todos os ativos implantados devem ser categorizados por nível de importância e classificação de dados. As classificações devem ser revisadas pela equipe de Governança de Nuvem e o aplicativo antes da implantação na nuvem.
2. Os aplicativos que armazenam ou acessam dados protegidos devem ser gerenciados de forma diferente daqueles que não o fazem. Eles devem ser, pelo menos, segmentados para evitar acesso não intencional a dados protegidos.
3. Todos os dados protegidos devem ser criptografados quando estão em repouso.
4. As permissões elevadas em qualquer segmento que contenha dados protegidos devem ser uma exceção. Tais exceções serão registradas com a equipe de Governança de Nuvem e auditadas regularmente.
5. As sub-redes de rede que contêm dados protegidos devem ser isoladas de todas as outras sub-redes. O tráfego de rede entre as sub-redes de dados protegidos será auditado regularmente.
6. Nenhuma sub-rede que contém os dados protegidos pode ser acessada diretamente pela Internet pública ou em datacenters. O acesso a essas sub-redes deve ser roteado por meio de sub-redes intermediárias. Todo o acesso a essas sub-redes deve vir por meio de uma solução de firewall que pode executar a verificação de pacotes e funções de bloqueio.
7. As ferramentas de governança devem realizar a auditoria e impor requisitos de configuração de rede definidos pela equipe de gerenciamento de segurança.
8. As ferramentas de governança devem limitar a implantação da VM somente a imagens aprovadas.
9. Sempre que possível, o gerenciamento da configuração do nó deve aplicar requisitos da política à configuração de qualquer sistema operacional convidado. O gerenciamento de configuração de nó deve respeitar o investimento existente no Objeto de Política de Grupo (GPO) para a configuração de recurso.
10. As ferramentas de governança devem impor que as atualizações automáticas sejam habilitadas em todos os ativos implantados. Quando possível, as atualizações automáticas serão impostas. Quando não forem impostas por ferramentas, as violações devem ser revisadas com as equipes de gerenciamento operacional e corrigidas de acordo com as políticas de operações. Os ativos que não são automaticamente atualizados devem ser incluídos em processos que pertencem às operações de TI.
11. A criação de novas assinaturas ou grupos de gerenciamento para todos os aplicativos de missão crítica ou dados protegidos exigirá uma análise da equipe de Governança de Nuvem para garantir que a atribuição adequada de blueprint.
12. Um modelo de acesso de privilégio mínimo será aplicado a qualquer assinatura que contém aplicativos de missão crítica ou dados protegidos.
13. O fornecedor de nuvem deve ser capaz de integrar chaves de criptografia gerenciadas pela solução no local existente.
14. O fornecedor de nuvem deve ser capaz de dar suporte a solução existente de dispositivo de borda e todas as configurações necessárias para proteger qualquer limite de rede publicamente exposto.
15. O fornecedor de nuvem deve ser capaz de dar suporte a uma conexão compartilhada à WAN global, com a transmissão de dados roteados por meio da solução de dispositivo de borda existente.
16. As tendências e explorações que podem afetar as implantações de nuvem devem ser revisadas regularmente pela equipe de segurança para que sejam fornecidas atualizações às ferramentas de Linha de base de segurança usadas na nuvem.
17. As ferramentas de implantação devem ser aprovadas pela equipe de Governança de Nuvem para garantir uma governança contínua dos ativos implantados.
18. Os scripts de implantação devem ser mantidos em um repositório central acessível pela equipe de Governança de Nuvem para revisão e auditoria periódicas.
19. Os processos de controle devem incluir auditorias no momento da implantação e em ciclos regulares para garantir a consistência em todos os ativos.
20. A implantação de qualquer aplicativo que exija a autenticação do cliente deve usar um provedor de identidade aprovado que seja compatível com o provedor de identidade principal para usuários internos.
21. Os processos de governança de nuvem devem incluir revisão trimestral com as equipes de Linha de Base de Identidade para identidade atores mal-intencionados ou padrões de uso que devem ser evitados pela configuração de ativos de nuvem.

## <a name="evolution-of-the-best-practices"></a>Evolução das práticas recomendadas

Esta seção do artigo evoluirá o design MVP de governança para incluir novas políticas do Azure e uma implementação do Gerenciamento de Custos do Azure. Juntas, essas duas alterações de design atenderão às novas instruções da política corporativa.

Novas práticas recomendadas se enquadram em duas categorias: TI Corporativa (Hub) e (Spoke) de adoção da nuvem.

**Estabelecimento de uma assinatura de hub/spoke IT corporativa para centralizar a linha de base de segurança**: Nesta prática recomendada, a capacidade de governança existente é encapsulada por uma [Topologia de Spoke com serviços compartilhados do Hub][shared-services], com algumas adições principais da equipe de governança de nuvem.

1. Repositório de Azure DevOps. Crie um repositório no Azure DevOps para armazenar e controlar a versão de todos os modelos do Azure Resource Manager relevantes e configurações com script
2. Modelo de Hub-Spoke.
    1. As diretrizes no [Hub-Spoke com arquitetura de referência de serviços compartilhados] [ shared-services] podem ser usadas para gerar modelos do Resource Manager para os ativos necessários em um hub de TI corporativo.
    2. Ao usar esses modelos, essa estrutura pode ser feita repetidamente, como parte de uma estratégia de governança central.
    3. Além da arquitetura de referência atual, é recomendável que um modelo de grupo de segurança de rede (NSG) seja criando capturando quaisquer requisitos de porta de bloqueio ou a lista de permissões para a rede virtual hospedar o firewall. Este NSG será diferente de NSGs anteriores, pois será o primeiro NSG para permitir o tráfego público em uma rede virtual.
3. Criar políticas do Azure. Crie uma política chamada `Hub NSG Enforcement` para impor a configuração do NSG atribuído a qualquer rede virtual criado nesta assinatura. Aplique as políticas internas para a configuração do convidado da seguinte maneira:
    1. Realize uma auditoria para verificar se os servidores Web do Windows estão usando protocolos de comunicação segura.
    2. Realize uma auditoria para verificar se as configurações de segurança de senha estão definidas corretamente em computadores Linux e Windows.
4. Blueprint de TI corporativo
    1. Crie um novo blueprint do Azure chamado `corporate-it-subscription`.
    2. Adicione os modelos de hub/spoke e a política de NSG de Hub.
5. Expandindo na hierarquia do grupo de gerenciamento inicial.
    1. Para cada grupo de gerenciamento que solicitou suporte para dados protegidos, o `corporate-it-subscription-blueprint` blueprint fornece uma solução acelerada de hub.
    2. Como os grupos de gerenciamento neste exemplo fictício incluem uma hierarquia regional, além de uma hierarquia de unidade de negócios, este projeto será implantado em cada região.
    3. Para cada região na hierarquia de grupo de gerenciamento, crie uma assinatura denominada `Corporate IT Subscription`.
    4. Aplique o `corporate-it-subscription-blueprint` plano gráfico para cada instância regional.
    5. Isso estabelecerá um hub para cada unidade de negócios em cada região. Observação: Ainda mais economia de custos pode ser obtida, mas hubs de compartilhamento entre unidades de negócios em cada região.
6. Integre objetos de política de grupo (GPO) por meio de Configuração de estado desejado (DSC):
    1. Converter o GPO para DSC – o [projeto de Gerenciamento de Linha de base da Microsoft](https://github.com/Microsoft/BaselineManagement) no Github pode acelerar esse esforço. * Certifique-se de armazenar o DSC no repositório em paralelo aos modelos do Gerenciador de Recursos.
    2. Implante a configuração de estado de Automação do Azure a todas as instâncias da assinatura de TI corporativa. A Automação do Azure pode ser usada para aplicar a DSC para máquinas virtuais implantadas em assinaturas com suporte no grupo de gerenciamento.
    3. Os planos de roteiro atuais habilitam as políticas de configuração personalizada do convidado. Quando esse recurso é liberado, o uso da Automação do Azure nessa recomendação não será necessário.

**Aplicar governança adicional para uma Assinatura de Adoção da Nuvem (Spoke)**: Compilar no `Corporate IT Subscription`, pequenas alterações para a governança aplicada a cada assinatura dedicada ao suporte de arquétipos de aplicativo do MVP podem produzir rápida evolução.

No evoluções anteriores da prática recomendada, os NSGs foram definidos, bloqueando o tráfego público e o tráfego interno na lista de permissões. Além disso, o blueprint do Azure temporariamente criou os recursos de DMZ e do Active Directory Domain Services. Nessa evolução ajustaremos esses ativos um pouco, criando uma nova versão de especificação técnica do Azure.

1. Modelo de emparelhamento de rede. Este modelo irá emparelhar a VNet em cada assinatura com a VNet de Hub na assinatura de TI corporativa.
    1. As diretrizes da seção anterior, [Hub-Spoke com arquitetura de referência de serviços compartilhados] [ shared-services] gerou um modelo do Resource Manager para habilitar o emparelhamento de VNet.
    2. Esse modelo pode ser usado como um guia para modificar o modelo DMZ antes da evolução de governança.
    3. Essencialmente, agora estamos adicionando emparelhamento VNet para VNet DMZ que tiver sido conectado anteriormente para o dispositivo de borda locais pela VPN.
    4. Também é aconselhável que o VPN seja removido com base neste modelo também para garantir que nenhum tráfego seja roteado diretamente para o datacenter local, sem passar pela assinatura corporativa de TI e solução de Firewall.
    5. [Configuração de rede](/azure/automation/automation-dsc-overview#network-planning) adicional será necessária pela Automação do Azure para aplicar DSC às VMs hospedadas.
2. Modifique o NSG. Bloquear todo o tráfego público e direto no local no NSG. Somente o tráfego de entrada deve estar vindo por meio do par de rede virtual na assinatura da TI corporativa.
    1. Na evolução anterior, um NSG foi criado bloqueando todo o tráfego público e lista de permissões de todo o tráfego interno. Agora, desejamos deslocar este NSG um pouco.
    2. A nova configuração de NSG, deve bloquear todo o tráfego público e todo o tráfego do datacenter local.
    3. Inserir essa rede virtual de tráfego deve vir apenas de rede virtual no outro lado do par de rede virtual.
3. Implementação da Central de Segurança do Azure
    1. Configure a Central de Segurança do Azure para qualquer grupo de gerenciamento que contenha classificações de dados protegidos.
    2. Defina o provisionamento automático para ativado por padrão para garantir a conformidade da aplicação de patch.
    3. Estabeleça configurações de segurança do sistema operacional. A Segurança de TI definirá a configuração.
    4. Segurança da TI de suporte no uso inicial da Central de Segurança do Azure. Fazer a transição de uso da Central de segurança para a segurança de TI, mas manter o acesso para fins de aprimoramento contínuo de governança
    5. Crie um modelo do Gerenciador de Recursos que reflita as alterações necessárias para a configuração da Central de Segurança dentro de uma assinatura.
4. Atualizar o Azure Policy para todas as assinaturas.
    1. Realize uma auditoria e imponha o nível de importância e a classificação de dados para todos os grupos de gerenciamento e assinaturas a fim de identificar todas as assinaturas com classificações de dados protegidos.
    2. Realize uma auditoria e imponha o uso de imagens do SO aprovadas apenas.
    3. Faça uma auditoria e imponha as configurações de convidado com base nos requisitos de segurança para cada nó.
5. Atualize as políticas do Azure para todas as assinaturas que contenham classificações de dados protegidos.
    1. Realize uma auditoria e imponha o uso das funções apenas
    2. Realize uma auditoria e imponha o aplicativo de criptografia para todas as contas de armazenamento e arquivos em repouso nos nós individuais.
    3. Realize uma auditoria e imponha o aplicativo da nova versão do DMZ NSG.
    4. Realize uma auditoria e imponha o uso de uma sub-rede de rede aprovada e rede virtual por adaptador de rede.
    5. Realize uma auditoria e imponha a limitação das tabelas de roteamento definidas pelo usuário.
6. Blueprint do Azure:
    1. Crie um novo blueprint do Azure chamado `protected-data`.
    2. Adicione os modelos do par de VNet, NSG, e Central de Segurança do Azure para o blueprint.
    3. Certifique-se de que o modelo para o Active Directory Domain Services da evolução anterior não está incluído o blueprint. Todas as dependências no Active Directory Domain Services serão fornecidas pela assinatura da IT corporativa.
    4. Encerrar qualquer VMs do Active Directory Domain Services existente implantado na evolução anterior.
    5. Adicione as novas políticas para assinaturas de dados protegidos.
    6. Publique blueprint em qualquer grupo de gerenciamento que se destina a hospedar os dados protegidos.
    7. Aplique o novo blueprint a cada assinatura afetada, bem como os blueprints existentes.

## <a name="conclusion"></a>Conclusão

Adicione esses processos e alterações ao MVP de governa ajuda a eliminar muitos dos riscos associados à governança de segurança. Juntos, eles oferecem ferramentas de monitoramento de rede, identidade e segurança necessárias para proteger os dados.

## <a name="next-steps"></a>Próximas etapas

Como a adoção da nuvem continua a se desenvolver e a tornar-se mais útil comercialmente, a governança de riscos e de nuvem também precisam evoluir. No caso da empresa fictícia desta jornada, a próxima etapa é dar suporte a cargas de trabalho críticas. Este é o ponto em que os controles de consistência de recursos se tornam necessários.

> [!div class="nextstepaction"]
> [Evolução da consistência de recursos](./resource-consistency-evolution.md)

<!-- links -->

[shared-services]: ../../../../reference-architectures/hybrid-networking/shared-services.md