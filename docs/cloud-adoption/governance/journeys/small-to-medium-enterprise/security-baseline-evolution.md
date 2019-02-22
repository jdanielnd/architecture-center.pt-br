---
title: 'CAF: Empresas de pequeno a médio porte - evolução da Linha de Base de Segurança'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Explicação - Empresas de pequeno a médio porte - Evolução da Linha de Base de Segurança
author: BrianBlanchard
ms.openlocfilehash: 5714b886ef63cc2392905250d97ea905839f6011
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900234"
---
# <a name="caf-small-to-medium-enterprise-security-baseline-evolution"></a>CAF: Empresas de pequeno a médio porte: Evolução da linha de base de segurança

Este artigo desenvolve a narrativa adicionando controles de segurança que dão suporte à transferência de dados protegidos para a nuvem.

## <a name="evolution-of-the-narrative"></a>Evolução da narrativa

A liderança de negócios e de TI estão satisfeitas com os resultados dos testes iniciais pelas equipes de TI, de Desenvolvimento de Aplicativos e de BI. Para obter valores comerciais tangíveis dessas experiências, essas equipes devem ter permissão para integrar dados protegidos a soluções. Isso desencadeia alterações à política corporativa, mas também requer uma evolução das implementações de governança de nuvem antes que os dados protegidos possam parar na nuvem.

### <a name="evolution-of-the-cloud-governance-team"></a>Evolução da equipe de Governança de Nuvem

Dado o impacto da narrativa sobre alterações e sobre suporte fornecidas até agora, a equipe de Governança de Nuvem agora é vista de forma diferente. Os dois administradores de sistema que iniciaram a equipe agora são vistos como arquitetos de nuvem experientes. Conforme esse narrativa evolui, a percepção sobre eles mudará de Custodiantes da Nuvem para uma função de Guardiões da Nuvem.

Embora a diferença seja sutil, é uma distinção importante ao criar uma cultura de TI focada em governança. Um Custodiante da Nuvem organiza a bagunça feita pelos arquitetos de nuvem inovadores. As duas funções possuem um atrito natural e objetivos opostos. Por outro lado, um Guardião da Nuvem ajuda a proteger a nuvem para que outros arquitetos de nuvem possam trabalhar mais rapidamente, com menos complicações. Além disso, um Guardião da Nuvem está envolvido na criação de modelos que aceleram a implantação e a adoção, tornando-os um acelerador de inovação, e também um defensor das cinco disciplinas da nuvem.

### <a name="evolution-of-the-current-state"></a>Evolução do estado atual

No início desse narrativa, as equipes de Desenvolvimento de Aplicativos ainda estavam trabalhando em uma funcionalidade de desenvolvimento/teste e a equipe de BI ainda estava na fase experimental. A TI trabalhava com dois ambientes de infraestrutura hospedada, chamados de Produção e Recuperação de desastre.

Desde então, ocorreram algumas mudanças que afetarão a governança:

- A equipe de Desenvolvimento de Aplicativos implementou um pipeline de CI/CD para implantar um aplicativo nativo de nuvem com uma experiência de usuário aprimorada. Esse aplicativo ainda não interage com os dados protegidos, portanto, não está pronto para produção.
- A equipe de Business Intelligence dentro da TI cuida ativamente dos dados de logística, inventário e terceiros na nuvem. Esses dados estão sendo usados para gerar novas previsões, o que poderia moldar os processos de negócios. No entanto, essas previsões e informações não são acionáveis até que os dados financeiros e do cliente possam ser integrados à plataforma de dados.
- A equipe de TI presta ajuda aos planos do Diretor Financeiro e do Diretor de TI para desativar o datacenter de recuperação de desastre. Mais de 1.000 dos 2.000 ativos no datacenter de recuperação de desastre foram desativados ou migrados.
- As políticas definidas de forma flexível em relação à PII (informações de identificação pessoal) e dados financeiros foram modernizadas. No entanto, as novas políticas corporativas dependem da implementação das políticas de segurança e governança relacionadas. As equipes ainda estão trabalhando nisso.

### <a name="evolution-of-the-future-state"></a>Evolução do estado futuro

Experimentos iniciais pelas equipes de Desenvolvimento de Aplicativos e de BI mostraram possíveis aprimoramentos nas experiências do cliente e nas decisões orientadas a dados. As duas equipes querem expandir a adoção da nuvem nos próximos 18 meses ao implantar essas soluções na produção.

Durante os seis meses restantes, a equipe de Governança de Nuvem implementará requisitos de governança e segurança para permitir que as equipes de adoção da nuvem migrem os dados protegidos para esses datacenters.

As alterações ao estado atual e futuro expõem novos riscos que exigem novas instruções de política.

## <a name="evolution-of-tangible-risks"></a>Evolução de riscos tangíveis

**Violação de dados**: Ao adotar uma nova plataforma de dados, há um aumento inerente nos passivos relacionados a possíveis violações de dados. Os técnicos que adotam as tecnologias de nuvem possuem maiores responsabilidades de implementar soluções que possam diminuir esse risco. Uma estratégia de governança e segurança robusta deve ser implementada para garantir que os técnicos cumpram essas responsabilidades.

Esse risco de negócios pode ser dividido em alguns riscos técnicos:

- Os aplicativos de missão crítica ou dados protegidos podem ser implantados acidentalmente.
- Dados protegidos podem ser expostos durante o armazenamento devido a más decisões de criptografia.
- Os usuários não autorizados podem acessar dados protegidos.
- Uma invasão externa pode resultar no acesso aos dados protegidos.
- Uma invasão externa ou ataques de negação de serviço podem causar uma interrupção dos negócios.
- As alterações de emprego ou da organização podem resultar em acesso não autorizado aos dados protegidos.
- Novas explorações poderiam criar novas oportunidades de invasão ou acesso.
- Os processos de implantação inconsistentes podem resultar em falhas de segurança, o que pode levar a perdas de dados ou interrupções.
- A inconsistência na configuração ou patches ausentes podem resultar em falhas de segurança não intencionais, o que pode levar a vazamentos de dados ou interrupções.

## <a name="evolution-of-the-policy-statements"></a>Evolução das instruções da política

As seguintes alterações à política ajudam a reduzir novos riscos e a orientar a implementação. A lista parece longa, mas adotar essas políticas talvez seja mais fácil do que parece.

1. Todos os ativos implantados devem ser categorizados por nível de importância e classificação de dados. As classificações devem ser revisadas pela equipe de Governança de Nuvem e pelo proprietário do aplicativo antes da implantação na nuvem.
2. Os aplicativos que armazenam ou acessam dados protegidos devem ser gerenciados de forma diferente daqueles que não o fazem. Eles devem ser, pelo menos, segmentados para evitar acesso não intencional a dados protegidos.
3. Todos os dados protegidos devem ser criptografados quando estão em repouso.
4. As permissões elevadas em qualquer segmento que contenha dados protegidos devem ser uma exceção. Tais exceções serão registradas com a equipe de Governança de Nuvem e auditadas regularmente.
5. As sub-redes de rede que contêm dados protegidos devem ser isoladas de todas as outras sub-redes. O tráfego de rede entre as sub-redes de dados protegidos será auditado regularmente.
6. Nenhuma sub-rede que contém os dados protegidos pode ser acessada diretamente pela Internet pública ou em datacenters. O acesso a essas sub-redes deve ser roteado por meio de sub-redes intermediárias. Todo o acesso a essas sub-redes deve vir por meio de uma solução de firewall que pode executar a verificação de pacotes e funções de bloqueio.
7. As ferramentas de governança devem realizar a auditoria e impor requisitos de configuração de rede definidos pela equipe de gerenciamento de segurança.
8. As ferramentas de governança devem limitar a implantação da VM somente a imagens aprovadas.
9. Sempre que possível, o gerenciamento da configuração do nó deve aplicar requisitos da política à configuração de qualquer sistema operacional convidado.
10. As ferramentas de governança devem impor que as atualizações automáticas sejam habilitadas em todos os ativos implantados. As violações devem ser revisadas com as equipes de gerenciamento operacional e corrigidas de acordo com as políticas de operações. Os ativos que não são automaticamente atualizados devem ser incluídos em processos que pertencem às operações de TI.
11. A criação de novas assinaturas ou grupos de gerenciamento para todos os aplicativos de missão crítica ou dados protegidos exigirá uma análise da equipe de Governança de Nuvem para garantir que o blueprint adequado seja atribuído.
12. Um modelo de acesso de privilégio mínimo será aplicado a qualquer grupo de gerenciamento ou assinatura que contém aplicativos de missão crítica ou dados protegidos.
13. As tendências e explorações que podem afetar as implantações de nuvem devem ser revisadas regularmente pela equipe de segurança para que sejam fornecidas atualizações às ferramentas de gerenciamento de segurança usadas na nuvem.
14. As ferramentas de implantação devem ser aprovadas pela equipe de Governança de Nuvem para garantir uma governança contínua dos ativos implantados.
15. Os scripts de implantação devem ser mantidos em um repositório central acessível pela equipe de Governança de Nuvem para revisão e auditoria periódicas.
16. Os processos de controle devem incluir auditorias no momento da implantação e em ciclos regulares para garantir a consistência em todos os ativos.
17. A implantação de qualquer aplicativo que exija a autenticação do cliente deve usar um provedor de identidade aprovado que seja compatível com o provedor de identidade principal para usuários internos.
18. Os processos de governança de nuvem devem incluir revisões trimestrais com equipes de gerenciamento de identidades. Essas revisões podem ajudar a identificar padrões de uso ou atores mal-intencionados que devem ser impedidos pela configuração de ativos de nuvem.

## <a name="evolution-of-the-best-practices"></a>Evolução das práticas recomendadas

O design de MVP de governança evoluirá para incluir novas políticas do Azure e uma implementação do Gerenciamento de Custos do Azure. Juntas, essas duas alterações de design atenderão às novas instruções da política corporativa.

1. As equipes de Rede e de Segurança de TI definirão os requisitos de rede. A equipe de Governança de Nuvem ajudará na comunicação entre as equipes.
2. As equipes de Identidade e Segurança de TI definirão requisitos de identidade e farão as alterações necessárias à implementação local do Active Directory. A equipe de Governança de Nuvem revisará as alterações.
3. Crie um repositório no Azure DevOps para armazenar e controlar a versão de todos os modelos do Azure Resource Manager relevantes e configurações com script.
4. Implementação da Central de Segurança do Azure:
    1. Configure a Central de Segurança do Azure para qualquer grupo de gerenciamento que contenha classificações de dados protegidos.
    2. Defina o provisionamento automático para ativado por padrão para garantir a conformidade da aplicação de patch.
    3. Estabeleça configurações de segurança do sistema operacional. A equipe de Segurança de TI definirá a configuração.
    4. Dê suporte à equipe de Segurança de TI no uso inicial da Central de Segurança. Faça a transição do uso da Central de Segurança para a equipe de Segurança de TI, mas mantenha o acesso para fins de aprimorar continuamente a governança.
    5. Crie um modelo do Resource Manager que reflita as alterações necessárias para a configuração da Central de Segurança dentro de uma assinatura.
5. Atualize as políticas do Azure para todas as assinaturas:
    1. Realize uma auditoria e imponha o nível de importância e a classificação de dados para todos os grupos de gerenciamento e assinaturas a fim de identificar todas as assinaturas com classificações de dados protegidos.
    2. Realize uma auditoria e imponha o uso de imagens aprovadas apenas.
6. Atualize as políticas do Azure para todas as assinaturas que contenham classificações de dados protegidos:
    1. Realize uma auditoria e imponha o uso das funções padrão do RBAC do Azure apenas.
    2. Realize uma auditoria e imponha a criptografia para todas as contas de armazenamento e arquivos em repouso em nós individuais.
    3. Realize uma auditoria e imponha a aplicação de um NSG para todas as NICs e sub-redes. As equipes de Rede e de Segurança de TI definirão o NSG.
    4. Realize uma auditoria e imponha o uso de uma sub-rede de rede aprovada e rede virtual por adaptador de rede.
    5. Realize uma auditoria e imponha a limitação das tabelas de roteamento definidas pelo usuário.
    6. Aplique as políticas internas para a configuração do convidado da seguinte maneira:
        1. Realize uma auditoria para verificar se os servidores Web do Windows estão usando protocolos de comunicação segura
        2. Realize uma auditoria para verificar se as configurações de segurança de senha estão definidas corretamente em computadores Linux e Windows
7. Configuração do firewall:
    1. Identifique uma configuração do Firewall do Azure que atenda aos requisitos de segurança necessários. Como alternativa, identifique um dispositivo de terceiros que seja compatível com o Azure.
    2. Crie um modelo do Resource Manager para implantar o firewall com as configurações necessárias.
8. Blueprint do Azure:
    1. Crie um novo blueprint chamado `protected-data`.
    2. Adicione o firewall e os modelos da Central de Segurança do Azure ao blueprint.
    3. Adicione as novas políticas para assinaturas de dados protegidos.
    4. Publique o blueprint em qualquer grupo de gerenciamento que esteja planejando hospedar dados protegidos.
    5. Aplique o novo blueprint a cada assinatura afetada, além dos blueprints existentes.

## <a name="conclusion"></a>Conclusão

Adicionar os processos e as alterações à governança do MVP acima ajudará a atenuar muitos riscos associados à governança de segurança. Juntos, eles oferecem ferramentas de monitoramento de rede, identidade e segurança necessárias para proteger os dados.

## <a name="next-steps"></a>Próximas etapas

Como a adoção da nuvem continua a se desenvolver e a tornar-se mais útil comercialmente, a governança de riscos e de nuvem também precisam evoluir. No caso da empresa fictícia desta jornada, a próxima etapa é dar suporte a cargas de trabalho críticas. Este é o ponto em que os controles de consistência de recursos se tornam necessários.

> [!div class="nextstepaction"]
> [Evolução da consistência de recursos](./resource-consistency-evolution.md)
