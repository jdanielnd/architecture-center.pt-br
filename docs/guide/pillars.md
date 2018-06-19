---
title: Pilares da qualidade de software
description: Descreve os cinco pilares de qualidade de software, escalabilidade, disponibilidade, resiliência, gerenciamento e segurança.
author: MikeWasson
ms.openlocfilehash: 117706046ca1a9b7f3203a99737347809d0c323f
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252780"
---
# <a name="pillars-of-software-quality"></a>Pilares da qualidade de software 

Um aplicativo em nuvem bem-sucedido se concentrará nestes cinco pilares da qualidade de software: escalabilidade, disponibilidade, resiliência, gerenciamento e segurança.

| Pilar | DESCRIÇÃO |
|--------|-------------|
| Escalabilidade | A capacidade de um sistema para lidar com aumentos de carga. |
| Disponibilidade | A proporção de tempo que um sistema está funcional e operando. |
| Resiliência | A capacidade de um sistema de se recuperar de falhas e continuar funcionando. |
| Gerenciamento | Processos de operações que mantêm um sistema em execução em produção. |
| Segurança | Proteger aplicativos e dados contra ameaças. |

## <a name="scalability"></a>Escalabilidade

Escalabilidade é a capacidade de um sistema de lidar com aumentos de carga. Há duas maneiras principais de um aplicativo ser dimensionado. Dimensionamento vertical (escalar *verticalmente*) significa aumentar a capacidade de um recurso, por exemplo, usando um tamanho maior de VM. Dimensionamento horizontal (escalar *horizontalmente*) é adicionar novas instâncias de um recurso, como VMs ou réplicas de banco de dados. 

O dimensionamento horizontal tem vantagens significativas sobre o dimensionamento vertical:

- Verdadeira escala de nuvem. Os aplicativos podem ser desenvolvidos para executarem em centenas ou milhares de nós, atingindo escalas que não são possíveis em um único nó.
- A escala horizontal é elástica. Você poderá adicionar mais instâncias se a carga aumentar ou removê-las durante períodos mais tranquilos.
- A escala horizontal pode ser acionada automaticamente, seja conforme uma agenda ou em resposta a mudanças na carga. 
- A escala horizontal pode ser mais barata do que a escala vertical. Executar várias VMs pequenas pode custar menos do que uma única VM grande. 
- O dimensionamento horizontal também pode melhorar a resiliência ao adicionar redundância. Se uma instância ficar inativa, o aplicativo continuará em execução.

Uma vantagem da escala vertical é que você pode fazer isso sem fazer nenhuma alteração ao aplicativo. Porém, em algum momento, você encontrará um limite em que não poderá mais escalar verticalmente. Nesse ponto, qualquer escala adicional deverá ser horizontal. 

A escala horizontal deve ser criada no sistema. Por exemplo, você pode aumentar as VMs colocando-as atrás de um balanceador de carga. Porém, cada VM no pool deve ser capaz de lidar com quaisquer solicitações do cliente, portanto, o aplicativo deve ser sem estado ou armazenar o estado externamente (digamos, em um cache distribuído). Serviços PaaS gerenciados geralmente têm escala horizontal e escala dimensionamento automático integrados. A facilidade de dimensionar esses serviços é uma grande vantagem de usar os serviços de PaaS.

Porém, apenas adicionar mais instâncias não significa que um aplicativo será dimensionado. Isso pode simplesmente empurrar o gargalo para outro lugar. Por exemplo, se você dimensionar uma front-end da Web para tratar mais solicitações de cliente, isso poderá disparar contenções de bloqueio no banco de dados. Você então precisaria considerar medidas adicionais, como particionamento de dados ou simultaneidade otimista, para habilitar mais taxa de transferência para o banco de dados.

Sempre realize teste de desempenho e de carga para localizar esses possíveis gargalos. As partes com estado de um sistema, como bancos de dados, são as causas mais comuns de gargalos e exigem design cuidadoso ao escalar horizontalmente. Resolver um gargalo pode revelar outros em outro lugar.

Use a [Lista de verificação de escalabilidade][scalability-checklist] para examinar o design de um ponto de vista de escalabilidade.

### <a name="scalability-guidance"></a>Diretriz de escalabilidade

- [Padrões de design para desempenho e escalabilidade][scalability-patterns]
- Melhores práticas: [Dimensionamento automático][autoscale], [Trabalhos em segundo plano][background-jobs], [Cache][caching], [CDN][cdn], [Particionamento de dados][data-partitioning]

## <a name="availability"></a>Disponibilidade

A disponibilidade é a proporção de tempo em que o sistema está funcional e em execução. Geralmente é medida como um percentual do tempo de atividade. Erros de aplicativo, problemas de infraestrutura e carga do sistema podem reduzir a disponibilidade. 

Um aplicativo em nuvem deve ter um SLO (objetivo de nível de serviço) que defina claramente a disponibilidade esperada e como a disponibilidade é medida. Ao definir a disponibilidade, observe o caminho crítico. O front-end Web pode ser capaz de fazer solicitações de cliente, mas se todas as transações falham porque ele não consegue se conectar ao banco de dados, o aplicativo não está disponível para os usuários. 

A disponibilidade normalmente é descrita em termos de "9s" &mdash; por exemplo, "quatro 9s" significa 99,99% de tempo de atividade. A tabela a seguir mostra o tempo de inatividade potencial cumulativo em diferentes níveis de disponibilidade.

| Tempo de atividade % | Tempo de inatividade por semana | Tempo de inatividade por mês | Tempo de inatividade por ano |
|----------|-------------------|--------------------|-------------------|
| 99% | 1,68 hora | 7,2 horas | 3,65 dias |
| 99,9% | 10 minutos | 43,2 minutos | 8,76 horas |
| 99,95% | 5 minutos | 21,6 minutos | 4,38 horas |
| 99,99% | 1 minuto | 4,32 minutos | 52,56 minutos |
| 99,999% | 6 segundos | 26 segundos | 5,26 minutos |

Observe que 99% de tempo de atividade poderia equivaler a quase 2 horas de interrupção de serviço por semana. Para muitos aplicativos, especialmente aplicativos voltados ao consumidor, esse não é um SLO aceitável. Por outro lado, cinco 9s (99,999%) significa não mais de 5 minutos de tempo de inatividade em um *ano*. É desafiador o suficiente apenas detectar uma interrupção com essa rapidez, sem falar em resolver o problema. Para obter uma disponibilidade muito alta (99,99% ou superior), você não pode depender de intervenção manual para se recuperar de falhas. O aplicativo deve fazer autodiagnóstico e autorrecuperação, e é aí que a resiliência se torna crucial.

No Azure, o SLA (Contrato de Nível de Serviço) descreve os compromissos da Microsoft com relação ao tempo de atividade e à conectividade. Se o SLA para um serviço específico for de 99,95%, isso significa que você deverá esperar que o serviço esteja disponível 99,95% do tempo.

Os aplicativos frequentemente dependem de vários serviços. Em geral, a probabilidade de qualquer serviço sofrer tempo de inatividade é independente. Por exemplo, suponha que seu aplicativo dependa de dois serviços, cada um com um SLA de 99,9%. O SLA de composição para ambos os serviços é de 99,9% &times; 99,9% &asymp; 99,8% ou um pouco menos que cada serviço por si só. 

Use a [Lista de verificação de disponibilidade][availability-checklist] para examinar o seu design de um ponto de vista de disponibilidade.

### <a name="availability-guidance"></a>Diretriz de disponibilidade

- [Padrões de design para disponibilidade][availability-patterns]
- Melhores práticas: [Dimensionamento automático][autoscale], [Trabalhos em segundo plano][background-jobs]

## <a name="resiliency"></a>Resiliência

Resiliência é a capacidade de um sistema de se recuperar de falhas e continuar funcionando. A meta da resiliência é retornar o aplicativo a um estado totalmente funcional após uma falha. Resiliência está intimamente relacionada à disponibilidade.

No desenvolvimento de aplicativos tradicionais, o foco era em como reduzir o MTBF (tempo médio entre falhas). Era feito um esforço para impedir que o sistema falhasse. Em computação em nuvem, uma mentalidade diferente é necessária devido a vários fatores:

- Sistemas distribuídos são complexos e uma falha em um ponto potencialmente pode se disseminar em cascata por todo o sistema.
- Os custos para ambientes em nuvem são mantidos baixos com o uso de hardware de mercadoria, portanto falhas ocasionais de hardware devem ser esperadas. 
- Aplicativos geralmente dependem de serviços externos, que podem ficar temporariamente indisponíveis ou limitar usuários de alto volume. 
- Os usuários atuais esperam que um aplicativo esteja disponível 24/7 sem nunca ficar offline.

Todos esses fatores significam que os aplicativos de nuvem devem ser criados para esperar falhas ocasionais e recuperarem-se delas. O Azure tem muitos recursos de resiliência já incorporados à plataforma. Por exemplo, 

- Armazenamento do Azure, Banco de Dados SQL e Cosmos DB oferecem replicação de dados interna, tanto dentro de uma região quanto entre regiões.
- Os Azure Managed Disks são colocados automaticamente em unidades de escala de armazenamento diferentes para limitar os efeitos das falhas de hardware.
- VMs em um conjunto de disponibilidade são distribuídas entre vários domínios de falha. Um domínio de falha é um grupo de VMs que compartilham uma mesma fonte de energia e um mesmo comutador de rede. Distribuir as VMs entre domínios de falha limita o impacto de falhas de hardware, indisponibilidades da rede ou interrupções de energia.

Dito isso, você ainda precisa criar resiliência em seu aplicativo. Estratégias de resiliência podem ser aplicadas em todos os níveis da arquitetura. Algumas mitigações de risco têm natureza mais tática &mdash; por exemplo, repetir uma chamada remota após uma falha transitória de rede. Outras mitigações de risco são mais estratégicas, como fazer failover de todo o aplicativo para uma região secundária. Táticas de mitigação de risco podem fazer uma grande diferença. Embora seja raro que toda a região sofra uma interrupção, problemas transitórios, como o congestionamento de rede, são mais comuns &mdash; assim, foque primeiro nisso. Também é importante ter o monitoramento e o diagnóstico certos, tanto para detectar falhas quando elas ocorrem quanto para localizar a causa raiz.

Ao criar um aplicativo para ser resiliente, você deve entender seus requisitos de disponibilidade. Quanto tempo de inatividade é aceitável? Trata-se parcialmente de uma questão de custo. Quanto o tempo de inatividade em potencial custará para sua empresa? Quanto você deve investir para tornar o aplicativo altamente disponível?

Use a [Lista de verificação de resiliência][resiliency-checklist] para examinar o design de um ponto de vista de resiliência.

### <a name="resiliency-guidance"></a>Diretrizes de resiliência

- [Projetando aplicativos resilientes para Azure][resiliency]
- [Padrões de design para resiliência][resiliency-patterns]
- Melhores práticas: [Tratamento de falhas transitórias][transient-fault-handling], [Diretriz de nova tentativa para serviços específicos][retry-service-specific]

## <a name="management-and-devops"></a>Gerenciamento e DevOps

Este pilar abrange os processos de operações que mantêm um aplicativo em execução em produção.

As implantações devem ser confiáveis e previsíveis. Elas devem ser automatizadas para reduzir a chance de erro humano. Elas devem ser um processo rápido e rotineiro para que não desacelerem a liberação de novos recursos ou correções de bugs. Igualmente importante, você deve ser capaz de rapidamente reverter ou efetuar roll forward se uma atualização tiver problemas.

Monitoramento e diagnóstico são cruciais. Os aplicativos em nuvem são executados em um data center remoto em que você não tem controle total da infraestrutura ou, em alguns casos, do sistema operacional. Em um aplicativo grande, não é prático fazer logon em VMs para solucionar um problema ou examinar arquivos de log. Com serviços PaaS, pode ainda não haver uma VM dedicada para fazer logon. Monitoramento e diagnóstico proporcionam insight sobre o sistema para que você saiba quando e em que ponto ocorrerem falhas. Todos os sistemas devem ser observáveis. Use um esquema de registro em log comum e consistente que permita correlacionar eventos entre sistemas.

O processo de monitoramento e diagnóstico tem várias fases distintas:

- Instrumentação. Gerar dados brutos, de logs de aplicativo, logs de servidor Web, diagnóstico interno à plataforma Azure e outras fontes.
- Coleta e armazenamento. Consolidar os dados em um único lugar.
- Análise e diagnóstico. Para solucionar problemas e verificar a integridade geral.
- Visualização e alertas. Usar dados de telemetria para detectar tendências ou alertar a equipe de operações.

Use a [Lista de verificação de DevOps][devops-checklist] para examinar o design de um ponto de vista de DevOps e de gerenciamento.

### <a name="management-and-devops-guidance"></a>Diretriz de gerenciamento e DevOps

- [Padrões de design para gerenciamento e monitoramento][management-patterns]
- Melhores práticas: [monitoramento e diagnóstico][monitoring]

## <a name="security"></a>Segurança

Você deve pensar sobre a segurança em todo o ciclo de vida de um aplicativo, do design e implementação até a implantação e as operações. A plataforma do Azure fornece proteções contra uma variedade de ameaças, como ataques de DDoS e invasão da rede. Porém, você ainda precisa incluir segurança em seu aplicativo e em seus processos de DevOps.

Aqui estão algumas áreas amplas de segurança a serem consideradas. 

### <a name="identity-management"></a>Gerenciamento de identidades

Considere usar o Azure AD (Azure Active Directory) para autenticar e autorizar usuários. O Azure AD é um serviço de gerenciamento de identidade e acesso totalmente gerenciado. Você pode usá-lo para criar domínios que existem somente no Azure ou integrá-lo às suas identidades do Active Directory locais. O Azure AD também se integra ao Office365, ao Dynamics CRM Online e a muitos aplicativos SaaS de terceiros. Para aplicativos voltados para o consumidor, o Azure Active Directory B2C permite aos usuários autenticar-se com suas contas sociais existentes (como Facebook, Google ou LinkedIn) ou criar uma nova conta de usuário gerenciada pelo Azure AD.

Se você quiser integrar um ambiente do Active Directory local a uma rede do Azure, várias abordagens são possíveis, dependendo dos seus requisitos. Para obter mais informações, consulte nossas arquiteturas de referência de [Gerenciamento de Identidade][identity-ref-arch].

### <a name="protecting-your-infrastructure"></a>Proteger sua infraestrutura 

Controle o acesso aos recursos do Azure que você implanta. Cada assinatura do Azure tem uma [relação de confiança][ad-subscriptions] com um locatário do Azure AD. Use RBAC ([Controle de Acesso Baseado em Função][rbac]) para conceder as permissões corretas para recursos do Azure a usuários em sua organização. Conceda o acesso atribuindo a função RBAC a usuários ou grupos em um determinado escopo. O escopo pode ser uma assinatura, um grupo de recursos ou um único recurso. Faça a [auditoria][resource-manager-auditing] de todas as alterações de infraestrutura. 

### <a name="application-security"></a>Segurança de aplicativo

Em geral, as melhores práticas de segurança para desenvolvimento de aplicativos ainda se aplicam na nuvem. Isso inclui itens como usar SSL em todos os lugares, proteger contra ataques CSRF e XSS, prevenir ataques de injeção de SQL e assim por diante. 

Aplicativos de nuvem geralmente usam serviços gerenciados que têm chaves de acesso. Nunca faça o check-in deles no controle do código-fonte. É recomendável armazenar segredos do aplicativo no Azure Key Vault.

### <a name="data-sovereignty-and-encryption"></a>Criptografia e soberania de dados

Seus dados devem permanecer na zona geopolíticas correta ao usar o Azure altamente disponível. O armazenamento com replicação geográfica do Azure usa o conceito de uma [região emparelhada][paired-region] na mesma região geopolítica. 

Use o Key Vault para proteger segredos e chaves de criptografia. Ao usar o Key Vault, você pode criptografar chaves e segredos usando as chaves protegidas por HSMs (módulos de segurança de hardware). Muitos serviços de banco de dados e armazenamento do Azure dão suporte para criptografia de dados em repouso, incluindo [Armazenamento do Azure][storage-encryption], [Banco de Dados SQL do Azure][sql-db-encryption], [SQL Data Warehouse do Azure][data-warehouse-encryption] e [Cosmos DB][cosmosdb-encryption].

### <a name="security-resources"></a>Recursos de segurança

- A [Central de Segurança do Azure][security-center] fornece monitoramento de segurança e gerenciamento de políticas integrados em suas assinaturas do Azure. 
- [Documentação de segurança do Azure][security-documentation]
- [Central de Confiabilidade da Microsoft][trust-center]



<!-- links -->

[dr-guidance]: ../resiliency/disaster-recovery-azure-applications.md
[identity-ref-arch]: ../reference-architectures/identity/index.md
[resiliency]: ../resiliency/index.md

[ad-subscriptions]: /azure/active-directory/active-directory-how-subscriptions-associated-directory
[data-warehouse-encryption]: /azure/data-lake-store/data-lake-store-security-overview#data-protection
[cosmosdb-encryption]: /azure/cosmos-db/database-security
[rbac]: /azure/active-directory/role-based-access-control-what-is
[paired-region]: /azure/best-practices-availability-paired-regions
[resource-manager-auditing]: /azure/azure-resource-manager/resource-group-audit
[security-blog]: https://azure.microsoft.com/blog/tag/security/
[security-center]: https://azure.microsoft.com/services/security-center/
[security-documentation]: /azure/security/
[sql-db-encryption]: /azure/sql-database/sql-database-always-encrypted-azure-key-vault
[storage-encryption]: /azure/storage/storage-service-encryption
[trust-center]: https://azure.microsoft.com/support/trust-center/
 

<!-- patterns -->
[availability-patterns]: ../patterns/category/availability.md
[management-patterns]: ../patterns/category/management-monitoring.md
[resiliency-patterns]: ../patterns/category/resiliency.md
[scalability-patterns]: ../patterns/category/performance-scalability.md


<!-- practices -->
[autoscale]: ../best-practices/auto-scaling.md
[background-jobs]: ../best-practices/background-jobs.md
[caching]: ../best-practices/caching.md
[cdn]: ../best-practices/cdn.md
[data-partitioning]: ../best-practices/data-partitioning.md
[monitoring]: ../best-practices/monitoring.md
[retry-service-specific]: ../best-practices/retry-service-specific.md
[transient-fault-handling]: ../best-practices/transient-faults.md


<!-- checklist -->
[availability-checklist]: ../checklist/availability.md
[devops-checklist]: ../checklist/dev-ops.md
[resiliency-checklist]: ../checklist/resiliency.md
[scalability-checklist]: ../checklist/scalability.md
