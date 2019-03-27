---
title: 'CAF: Política de linha de base de segurança nativa da nuvem'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Política de linha de base de segurança nativa da nuvem
author: BrianBlanchard
ms.openlocfilehash: fc161009f6ce7aa2b775fe6b3b53ff1e1d62a848
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242427"
---
# <a name="cloud-native-security-baseline-policy"></a>Política de linha de base de segurança nativa da nuvem

A [Linha de base de segurança](overview.md) é uma das [cinco disciplinas de Governança de nuvem](../governance-disciplines.md). Essa disciplina se concentra em tópicos de segurança geral, incluindo a proteção de rede, ativos digitais, dados, etc. Conforme descrito na [guia de política de revisão](../policy-compliance/what-is-a-cloud-policy-review.md), a CAF inclui três níveis de **política de exemplo**: Nativos de nuvem, Enterprise e Conformidade com o princípio de design da nuvem para cada uma das cinco disciplinas. Este artigo discute a Política de exemplo nativo de nuvem para a Disciplina da linha de base de segurança.

> [!NOTE]
> A Microsoft não está em nenhuma posição de ditar política corporativa ou de TI. Este artigo destina-se a ajudá-lo a se preparar para uma revisão de política interna. Supomos que essa política de exemplo será estendida, validada e testada com a sua política corporativa antes de usá-la. Qualquer uso dessa política de exemplo, como está, não é recomendado.

## <a name="policy-alignment"></a>Alinhamento de políticas

Essa política de exemplo sintetiza um cenário de nuvem nativa, o que significa que as ferramentas e plataformas fornecidas pelo Azure são suficientes para reduzir os riscos de negócios sobre uma implantação. Nesse cenário, supõe-se que uma configuração simples dos serviços padrão do Azure forneçam proteção suficiente do ativo.

## <a name="cloud-security-and-compliance"></a>Segurança e conformidade de nuvem

A segurança é integrada em cada aspecto do Azure, oferecendo vantagens de segurança exclusivas derivadas da inteligência de segurança global, controles sofisticados voltados para o cliente e infraestrutura de proteção segura. Essa poderosa combinação ajuda a proteger seus aplicativos e dados, dar suporte a seus esforços de conformidade e fornecer segurança econômica para organizações de todos os tamanhos. Essa abordagem cria uma forte posição inicial para qualquer política de segurança, mas não nega a necessidade de práticas de segurança igualmente fortes relacionadas aos serviços de segurança sendo usados.

### <a name="built-in-security-controls"></a>Controles de segurança interna

É difícil de manter uma infraestrutura de segurança forte quando os controles de segurança não são intuitivos e precisam ser configurados separadamente. O Azure inclui controles de segurança integrados em uma variedade de serviços que ajudam você a proteger os dados e cargas de trabalho rapidamente e a gerenciar risco entre ambientes híbridos. Soluções integradas de parceiros também permitem fazer uma transição fácil das proteções existentes para a nuvem.

### <a name="cloud-native-identity-policies"></a>Políticas de identidade Nativos de Nuvem

A identidade está se tornando o novo plano de controle de limite para segurança, assumindo a função antes exercida pela perspectiva centrada em rede tradicional. Os perímetros de rede têm se tornado cada vez mais porosos e a defesa do perímetro não pode ser tão eficiente quanto era antes da evolução de trazer seu próprio dispositivo (BYOD) e aplicativos de nuvem. O gerenciamento de identidade do Azure e o controle de acesso permitem acesso contínuo e seguro a todos os seus aplicativos.

Uma política de nativo de nuvem de exemplo para identidade entre diretórios locais e de nuvem, pode incluir requisitos semelhante ao seguinte:

* Acesso autorizado a recursos com controle de acesso baseado em função (RBAC), autenticação multifator (MFA) e logon único (SSO)
* Mitigação rápida de identidades de usuário suspeito de comprometimento
* Just-in-time (JIT), acesso suficiente concedido em uma base por tarefa para limitar a exposição de credenciais de administrador com muitos privilégios
* Identidade do usuário estendida e acesso às políticas em vários ambientes do Azure Active Directory

Embora seja importante entender a [Linha de Base de Identidade](../identity-baseline/overview.md) no contexto de linha de base de segurança, as [Cinco disciplinas de governança de nuvem](../overview.md) chamam a [Linha de base de identidade](../identity-baseline/overview.md) como sua própria disciplina, separada da Linha de base de segurança.

### <a name="network-access-policies"></a>Políticas de acesso de rede

O controle de rede inclui configuração, gerenciamento e proteção dos elementos de rede como redes virtuais, balanceamento de carga, DNS e gateways. Os controles fornecem um meio para os serviços se comunicarem e interoperarem. O Azure inclui uma infraestrutura de rede robusta e segura para dar suporte a seus requisitos de conectividade de aplicativo e de serviço. A conectividade de rede é possível entre os recursos localizados no Azure, entre os recursos locais e aqueles hospedados no Azure e de origem e destino à Internet e ao Azure.

Uma política de nativo de nuvem para controles de rede pode incluir requisitos semelhantes ao seguinte:

* Conexões híbridas para recursos locais (embora seja tecnicamente possível no Azure), podem não ser permitidas em uma política de nuvem nativa. Se uma conexão híbrida se provar necessária, um exemplo de política de segurança corporativa mais robusto seria uma referência mais relevante.
* Os usuários podem estabelecer conexões seguras para e dentro do Azure usando redes virtuais e grupos de segurança de rede.
* O Firewall do Azure protege os hosts do tráfego de rede mal-intencionados por acesso à porta limitado. Um bom exemplo dessa política seria o requisito para bloquear (ou não habilitar) tráfego diretamente para uma VM por RDP - TCP/UDP porta 3389.
* Serviços como o Firewall do Aplicativo Web e a Proteção contra DDoS do Azure protegem aplicativos e garantem a disponibilidade para máquinas virtuais em execução no Azure. Esses recursos não devem ser desabilitados ou mal utilizados.

### <a name="data-protection"></a>Proteção de dados

Uma das chaves de proteção de dados na nuvem é responsável por possíveis estados em que os dados podem ocorrer e quais controles estão disponíveis para esse estado. Em relação às práticas recomendadas de segurança de dados e criptografia do Azure, as recomendações se concentrarão nos estados de dados a seguir:

* Controles de criptografia de dados são criados nos serviços integrados de máquinas virtuais para armazenamento e banco de Dados SQL.
* Como os dados se movem entre nuvens e clientes, isso pode ser protegido usando protocolos de criptografia padrão do setor.
* O Azure Key Vault permite aos usuários do Azure proteger e controlar chaves de criptografia e outros segredos usados por aplicativos e serviços de nuvem.
* A Proteção de Informações do Azure ajudará a classificar, rotular e proteger seus dados confidenciais em aplicativos.

Embora esses recursos sejam criados no Azure, cada um dos itens acima requer configuração e pode aumentar os custos. Alinhamento de cada recurso Nativo de Nuvem com um [estratégia de classificação de dados](../policy-compliance/what-is-data-classification.md) é altamente recomendável.

### <a name="security-monitoring"></a>Monitoramento de segurança

O monitoramento de segurança é uma estratégia proativa que audita os recursos para identificar sistemas que não seguem os padrões ou as melhores práticas organizacionais. A Central de Segurança do Azure fornece um gerenciamento de segurança unificado e proteção avançada contra ameaças nas cargas de trabalho de nuvem híbrida. Com a Central de Segurança, é possível aplicar políticas de segurança em suas cargas de trabalho, limitar a exposição a ameaças e detectar e responder a ataques, incluindo:

* Exibição unificada sobre a segurança em todas as suas cargas de trabalho locais e na nuvem com a Central de Segurança do Azure
* Avaliações contínuas de monitoramento e segurança para garantir a conformidade e corrigir quaisquer vulnerabilidades
* Ferramentas interativas e inteligência contextual contra ameaças
* Registro em log extensivo e integração com as informações de segurança existentes

### <a name="extending-cloud-native-policies"></a>Estender as políticas de nuvem nativa

Usar a nuvem pode reduzir essa carga de segurança. A Microsoft fornece a segurança física para datacenters do Azure e ajuda a proteger a plataforma de nuvem contra ameaças de infraestrutura como um ataque de DDoS. Considerando que a Microsoft tem milhares de especialistas de segurança cibernética trabalhando em segurança todos os dias, os recursos para atenuar ataques cibernéticos são consideráveis. Na verdade, enquanto as organizações estavam preocupadas se a nuvem era segura, a maioria agora compreende que considerando o nível de investimento que fornecedores como a Microsoft fazem em pessoas e infra-estrutura especializada, a nuvem é realmente mais segura do que a maioria dos datacenters locais.

Mesmo com esse investimento na linha de base de segurança de nuvem nativa, é recomendável que qualquer política de linha de base de segurança se estenda para as políticas nativas de nuvem padrão. A seguir estão exemplos de políticas estendidas que devem ser considerados, mesmo em um ambiente nativo de nuvem:

* **Proteger VMs**. A segurança deve ser a prioridade mais alta de cada organização e fazê-lo com eficiência requer várias coisas. Você deve avaliar o estado da segurança, proteger contra ameaças à segurança e, em seguida, detectar e responder rapidamente às ameaças que ocorrem.
* **Proteger o conteúdo da VM**. Configurar os backups automatizados regulares é essencial para proteger contra erros do usuário. No entanto, isso não é suficiente. Você também deve verificar se os backups estão protegidos contra ataques cibernéticos e estão disponíveis quando você precisar deles.
* **Monitorar as VMs e aplicativos**. Esse padrão abrange várias tarefas, incluindo obter informações sobre a integridade de suas VMs, interações de compreensão entre eles e o estabelecimento de maneiras de monitorar os aplicativos que executam essas VMs. Todas essas tarefas são essenciais em manter seus aplicativos em execução o tempo todo.

## <a name="next-steps"></a>Próximas etapas

Agora que você analisou a política de exemplo de linha de base de segurança para soluções de nuvem nativa, retornar para o [guia de política de revisão](../policy-compliance/what-is-a-cloud-policy-review.md) para começar a criar essa amostra para criar suas próprias políticas para a adoção da nuvem.

> [!div class="nextstepaction"]
> [Crie suas próprias políticas usando o guia da política de revisão](../policy-compliance/what-is-a-cloud-policy-review.md)
