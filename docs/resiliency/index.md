---
title: Desenvolvimento de aplicativos resilientes para o Azure
description: Como criar aplicativos resilientes no Azure, para alta disponibilidade e recuperação de desastres.
author: MikeWasson
ms.date: 12/18/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: resiliency
ms.openlocfilehash: 4f33a3bef236ce157955f6348e7befea02a8ce80
ms.sourcegitcommit: 548374a0133f3caed3934fda6a380c76e6eaecea
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/25/2019
ms.locfileid: "58420066"
---
# <a name="designing-resilient-applications-for-azure"></a>Desenvolvimento de aplicativos resilientes para o Azure

Em um sistema distribuído, ocorrerão falhas. O hardware pode falhar. A rede pode ter falhas transitórias. Raramente, um serviço ou uma região inteira pode sofrer uma interrupção. Mesmo assim, devemos estar preparados para isso.

Criar um aplicativo confiável na nuvem é diferente da criação de um aplicativo confiável em uma configuração empresarial. Embora historicamente você possa ter comprado um hardware superior para escalar verticalmente, em um ambiente de nuvem, é preciso escalar horizontalmente. Os custos para ambientes em nuvem são mantidos baixos com o uso de hardware de mercadoria. Em vez de tentar evitar completamente a falha, a meta é minimizar os efeitos de uma falha no sistema.

Este artigo fornece uma visão geral de como criar aplicativos resilientes no Microsoft Azure. Ele começa com uma definição do termo *resiliência* e conceitos de relacionados. Em seguida, descreve um processo para a obtenção de resiliência, usando uma abordagem estruturada sobre o tempo de vida de um aplicativo, desde o design e a implementação até a implantação e as operações.

<!-- markdownlint-disable MD026 -->

## <a name="what-is-resiliency"></a>O que é resiliência?

<!-- markdownlint-enable MD026 -->

**Resiliência** é a capacidade de um sistema de se recuperar de falhas e continuar funcionando. Não se trata de *evitar* falhas, mas *responder* a elas de uma maneira que evite o tempo de inatividade ou a perda de dados. A meta de resiliência é retornar o aplicativo a um estado totalmente funcional após uma falha.

Dois aspectos importantes de resiliência são alta disponibilidade e recuperação de desastres.

- **HA (Alta Disponibilidade)** é a capacidade do aplicativo de continuar em execução em um estado íntegro, sem tempo de inatividade significativo. Por “estado íntegro” queremos dizer o aplicativo está respondendo e os usuários podem se conectar ao aplicativo e interagir com ele.  
- **DR (Recuperação de Desastre)** é a capacidade de recuperação de incidentes raros, mas sérios: falhas não transitórias em larga escala, como interrupções de serviço que afetam toda a região. A recuperação de desastre inclui backup e arquivamento de dados e pode incluir a intervenção manual, como a restauração um banco de dados com base em um backup.

Uma das formas de diferenciar a HA da DR é que a última é iniciada quando o impacto de uma falha excede a capacidade do design da HA para tratá-la.  

Quando você projeta buscando resiliência, você deve entender seus requisitos de disponibilidade. Quanto tempo de inatividade é aceitável? Trata-se parcialmente de uma questão de custo. Quanto o tempo de inatividade em potencial custará para sua empresa? Quanto você deve investir para tornar o aplicativo altamente disponível? Você também precisa definir o que significa para o aplicativo estar disponível. Por exemplo, o aplicativo está "inativo" se um cliente puder enviar uma ordem, mas o sistema não puder processá-la dentro do prazo normal? Considere também a probabilidade da ocorrência de um tipo específico de interrupção e se uma estratégia de redução é econômica.

Outro termo comum é **Continuidade dos Negócios** (BC), que é a capacidade de executar funções de negócios essenciais durante e após condições adversas, como um desastre natural ou um serviço inoperante. A BC abrange toda a operação de negócios, incluindo instalações físicas, pessoas, comunicações, transporte e IT. Este artigo se concentra em aplicativos em nuvem, mas o planejamento de resiliência deve ser feito considerando-se o contexto dos requisitos gerais da BC.

O **backup de dados** é uma parte fundamental da recuperação de desastre. Se os componentes sem monitoração de estado de um aplicativo falharem, você sempre poderá reimplantá-los. Mas se os dados forem perdidos, o sistema não poderá retornar a um estado estável. O backup dos dados deve ser feito, idealmente em uma região diferente para o caso de um desastre em toda a região.

Backup é diferente de *replicação de dados*. Replicação de dados envolve a cópia de dados quase em tempo real, para que o sistema possa realizar failover rapidamente para uma réplica. Muitos sistemas de bancos de dados são compatíveis com replicação; Por exemplo, o SQL Server é compatível com Grupos de Disponibilidade Always On do SQL Server. A replicação de dados pode reduzir o tempo necessário para recuperar-se de uma interrupção, garantindo que uma réplica dos dados esteja sempre disponível. No entanto, a replicação de dados não protegerá contra erro humano. Se os dados forem corrompidos devido a erro humano, eles serão apenas copiados para as réplicas. Portanto, você ainda precisa incluir o backup de longo prazo em sua estratégia de DR.

## <a name="process-to-achieve-resiliency"></a>Processo para obter resiliência

Resiliência não é um complemento. Ela deve ser criada no sistema e colocada em prática operacional. Aqui está um modelo geral a seguir:

1. **Defina** seus requisitos de disponibilidade, com base nas necessidades de negócios.
2. **Crie** o aplicativo para garantir a resiliência. Inicie com uma arquitetura que segue práticas comprovadas e, em seguida, identifique os pontos de falha possíveis na arquitetura.
3. **Implemente** estratégias para detectar e recuperar de falhas.
4. **Teste** a implementação simulando falhas e disparando failovers forçados.
5. **Implante** o aplicativo em produção usando um processo confiável e reproduzível.
6. **Monitore** o aplicativo para detectar falhas. Ao monitorar o sistema, você pode medir a integridade do aplicativo e responder a incidentes, se necessário.
7. **Responda** se houver falha que requer intervenções manuais.

No restante deste artigo, vamos abordar cada uma dessas etapas mais detalhadamente.

## <a name="define-your-availability-requirements"></a>Definir requisitos de disponibilidade

O planejamento da resiliência começa com os requisitos de negócios. A seguir estão algumas abordagens para pensarmos sobre resiliência sob esses aspectos.

### <a name="decompose-by-workload"></a>Decompor por carga de trabalho

Muitas soluções de nuvem consistem em várias cargas de trabalho de aplicativos. O termo "carga de trabalho" neste contexto significa um recurso discreto ou uma tarefa de computação, que pode ser logicamente separada de outras tarefas, em termos de requisitos de armazenamento de dados e lógica de negócios. Por exemplo, um aplicativo de comércio eletrônico pode incluir as cargas de trabalho a seguir:

- Procura e pesquisa em um catálogo de produtos.
- Criação e controle de pedidos.
- Exibição de recomendações.

Essas cargas de trabalho podem ter diferentes requisitos de disponibilidade, escalabilidade, consistência de dados e recuperação de desastres. Há decisões de negócios que devem ser feitas em termos de custo de balanceamento versus risco.

Considere também os padrões de uso. Há certos períodos críticos quando o sistema tem que estar disponível? Por exemplo, um serviço de arquivamento de imposto não pode ficar inativo antes do prazo de entrega, um serviço de streaming de vídeo tem que continuar durante um evento esportivo importante, e assim por diante. Durante os períodos críticos, talvez sejam necessárias implantações redundantes em várias regiões, para que o aplicativo possa fazer failover se uma região falhar. No entanto, uma implantação em várias regiões é potencialmente mais cara, por isso, durante horários menos críticos, você pode executar o aplicativo em uma única região. Em alguns casos, a despesa adicional pode ser reduzida usando técnicas sem servidor modernas, que usam a cobrança baseada em consumo, para que você não seja cobrado pelos recursos de computação pouco utilizados.

### <a name="rto-and-rpo"></a>RTO e RPO

Duas métricas importantes a considerar são o Objetivo do Tempo de Recuperação e o Objetivo do Ponto de Recuperação, pois elas pertencem à recuperação de desastres.

- **Objetivo do Tempo de Recuperação** (RTO) é o tempo máximo aceitável que um aplicativo pode ficar indisponível após um incidente. Se o RTO for de 90 minutos, você deverá ser capaz de restaurar o aplicativo para um estado de execução dentro de 90 minutos a partir do início de um desastre. Se você tiver um RTO muito baixo, poderá manter uma segunda implantação regional executando continuamente uma configuração ativa/passiva em modo de espera, para se proteger de uma interrupção regional. Em alguns casos, você pode implantar uma configuração ativa/ativa para alcançar um RTO ainda menor.

- **Objetivo do Ponto de Recuperação** (RPO) é a duração máxima de perda de dados aceitável durante um desastre. Por exemplo, se você armazena dados em um único banco de dados, com nenhuma replicação para outros bancos de dados, e executar backups a cada hora, poderá perder até uma hora de dados.

RTO e RPO são os requisitos não funcionais de um sistema e devem ser determinados pelos requisitos de negócios. Para obter esses valores, é uma boa ideia conduzir uma avaliação de risco para compreender claramente o custo do tempo de inatividade ou da perda de dados.

### <a name="mttr-and-mtbf"></a>MTTR e MTBF

Duas outras medidas comuns de disponibilidade são o tempo médio de recuperação (MTTR) e o tempo médio entre falhas (MTBF). Essas medidas são, no geral, usadas internamente pelos provedores de serviço para determinar onde adicionar redundância a serviços de nuvem e quais SLAs fornecer aos clientes.

**Tempo médio para recuperar** (MTTR) é o tempo médio que leva para restaurar um componente após uma falha. MTTR é um fato empírico sobre um componente. Com base no MTTR de cada componente, você pode estimar o MTTR de um aplicativo inteiro. A criação de aplicativos de vários componentes com valores baixos de MTTR resulta em um aplicativo com um MTTR geral baixo &mdash; que se recupera rapidamente de falhas.

**Tempo médio entre falhas** (MTBF) é o tempo de execução que um componente pode razoavelmente esperar que dure entre interrupções. Essa métrica pode ajudar você a calcular a frequência com que um serviço ficará indisponível. Um componente não confiável tem um MTBF baixo, resultando em um número baixo de SLA para esse componente. No entanto, um MTBF baixo pode ser reduzido com a implantação de várias instâncias do componente e a implementação de failover entre elas.

> [!NOTE]
> Se QUALQUER um dos valores dos componentes em uma configuração de alta disponibilidade MTTR exceder o RTO do sistema, uma falha no sistema causará uma interrupção inaceitável dos negócios. Não será possível restaurar o sistema dentro do RTO definido.

### <a name="slas"></a>SLAs

O [SLA (Contrato de Nível de Serviço)][sla] descreve os compromissos da Microsoft com relação ao tempo de atividade e à conectividade. Se o SLA para um serviço específico for de 99,9%, isso significa que você deve esperar que o serviço esteja disponível 99,9% do tempo.

> [!NOTE]
> O SLA do Azure também inclui provisões para se obter um crédito de serviço se o SLA não for atendido, além de definições específicas de "disponibilidade" para cada serviço. Esse aspecto do SLA atua como uma política de imposição.

Você deve definir seus próprios SLAs de destino para cada carga de trabalho em sua solução. Um SLA possibilita avaliar se a arquitetura atende aos requisitos de negócios. Execute um exercício de mapeamento de dependências para identificar dependências internas e externas, como o Active Directory ou serviços de terceiros, como um provedor de pagamento ou um serviço de mensagens de email. Preste atenção especificamente às dependências externas que podem ser ponto único de falha ou causar gargalos durante um evento. Por exemplo, se uma carga de trabalho requer tempo de disponibilidade de 99,99%, mas depende de um serviço com um SLA de 99,9%, o serviço não poderá ser um ponto único de falha no sistema. Uma solução é ter um caminho de fallback no caso de falha do serviço ou tomar outras medidas para recuperar uma falha nesse serviço.

A tabela a seguir mostra o tempo de inatividade cumulativo em potencial para vários níveis de SLA.

| Contrato de Nível de Serviço | Tempo de inatividade por semana | Tempo de inatividade por mês | Tempo de inatividade por ano |
| --- | --- | --- | --- |
| 99% |1,68 hora |7,2 horas |3,65 dias |
| 99,9% |10,1 minutos |43,2 minutos |8,76 horas |
| 99,95% |5 minutos |21,6 minutos |4,38 horas |
| 99,99% |1,01 minutos |4,32 minutos |52,56 minutos |
| 99,999% |6 segundos |25,9 segundos |5,26 minutos |

Obviamente, a alta disponibilidade é melhor, no caso de tudo ser igual. Mas, quando você se esforça por mais 9s, o custo e a complexidade aumentam para atingir esse nível de disponibilidade. Um tempo de disponibilidade de 99,99% se traduz em cerca de 5 minutos de inatividade total por mês. A complexidade adicional e o custo compensam valem a pena para alcançar cinco 9s? A resposta depende dos requisitos de negócios.

Aqui estão algumas outras considerações ao definir um SLA:

- Para obter quatro 9s (99,99%), você provavelmente não pode depender de intervenção manual para se recuperar de falhas. O aplicativo deve ser de autodiagnóstico e autorrecuperação.
- Além de quatro 9s, é difícil detectar problemas rápido o suficiente para atender ao SLA.
- Pense no intervalo de tempo ao qual a medição do seu SLA está relacionada. Quanto menor for o intervalo, menor será a tolerância. Provavelmente não faz sentido definir seu SLA em termos de tempo de atividade por hora ou dia.
- Considere as medidas de MTBF e MTTR. Quanto menor o SLA, menor será a frequência com que o serviço poderá ficar inativo, e mais rápida deverá ser a recuperação do serviço.

### <a name="composite-slas"></a>SLAs compostos

Considere um aplicativo Web do Serviço de Aplicativo que grava no banco de dados do SQL Azure. No momento da redação deste artigo, esses serviços do Azure têm os seguintes SLAs:

- Aplicativos Web do Serviço de Aplicativo = 99,95%
- Banco de dados SQL = 99,99%

![SLA composto](./images/sla1.png)

Qual é o tempo de inatividade máximo esperado para este aplicativo? Se o serviço falhar, o aplicativo inteiro falhará. Em geral, a probabilidade de cada serviço com falha é independente, portanto, o SLA composto para este aplicativo é de 99,95% &times; 99,99% = % 99,94%. Isso é menos do que os SLAs individuais, o que não é surpresa, porque um aplicativo que depende de vários serviços possui mais pontos de falha em potencial.

Por outro lado, você pode melhorar o SLA composto criando caminhos de fallback independentes. Por exemplo, se o banco de dados SQL não estiver disponível, coloque as transações em uma fila, para serem processadas posteriormente.

![SLA composto](./images/sla2.png)

Com esse design, o aplicativo ainda estará disponível mesmo se não puder se conectar ao banco de dados. No entanto, ele falhará se o banco de dados e a fila falharem ao mesmo tempo. A porcentagem esperada de tempo de uma falha simultânea é de 0,0001 &times; 0,001, portanto, o SLA composto para este caminho combinado é:  

- Banco de dados OU fila = 1,0 &minus; (0,0001 &times; 0,001) = 99,99999%

O SLA composto total é:

- Aplicativo Web E (banco de dados OU fila) = 99,95% &times; 99,99999% = ~99,95%

Mas existem compensações para essa abordagem. A lógica do aplicativo é mais complexa. Você está pagando pela fila e pode haver problemas de consistência de dados a serem considerados.

**SLA para implantações em várias regiões**. Outra técnica de HA é implantar o aplicativo em mais de uma região e usar o Gerenciador de Tráfego do Azure para fazer o failover se o aplicativo falhar em uma região. Para uma implantação de múltiplas regiões, o SLA composto é calculado da maneira a seguir.

Seja *N* o SLA composto para o aplicativo implantado em uma região e *R* o número de regiões em que o aplicativo é implantado. A possibilidade esperada de o aplicativo falhar em todas as regiões ao mesmo tempo é de ((1 &minus; N) ^ R).

Por exemplo, se o SLA de região única é de 99,95%,

- O SLA combinado para duas regiões = (1 &minus; (1 &minus; 0,9995) ^ 2) = 99,999975%
- O SLA combinado para quatro regiões = (1 &minus; (1 &minus; 0,9995) ^ 4) = 99,999999%

Você também deve considerar o [SLA para o Gerenciador de Tráfego][tm-sla]. No momento da redação deste artigo, o SLA para o SLA do Gerenciador de Tráfego é de 99,99%.

Além disso, o failover não é instantâneo em configurações ativas-passivas, o que pode resultar em algum tempo de inatividade durante um failover. Consulte [Monitoramento e failover de ponto de extremidade do Gerenciador de tráfego][tm-failover].

O número calculado de SLA é uma linha de base útil, mas ela não informa a história toda sobre a disponibilidade. Geralmente, um aplicativo pode degradar normalmente quando um caminho não crítico falha. Considere um aplicativo que mostra um catálogo de livros. Se o aplicativo não pode recuperar a imagem em miniatura da capa, poderá mostrar uma imagem de espaço reservado. Nesse caso, a falha ao obter a imagem não reduz o tempo de atividade do aplicativo, embora afete a experiência do usuário.  

## <a name="design-for-resiliency"></a>Design para resiliência

Durante a fase de design, você deve executar uma Análise do Modo de Falha (FMA). O objetivo de uma FMA é identificar possíveis pontos de falha e definir como o aplicativo responde a essas falhas.

- Como o aplicativo detectará esse tipo de falha?
- Como o aplicativo responderá a esse tipo de falha?
- Como você pode fazer logon e monitorar esse tipo de falha?

Para obter mais informações sobre o processo FMA, com as recomendações específicas para o Azure, confira [Orientação de resiliência do Azure: Análise do modo de falha][fma].

### <a name="example-of-identifying-failure-modes-and-detection-strategy"></a>Exemplo de identificação de modos de falha e estratégia de detecção

**Ponto de falha:** Chamada para um serviço Web externo/API.

| Modo de falha | Estratégia de detecção |
| --- | --- |
| O serviço está indisponível |HTTP 5xx |
| Limitação |HTTP 429 (Número Excessivo de Solicitações) |
| Authentication |HTTP 401 (Não Autorizado) |
| Resposta lenta |Tempo limite da solicitação atingido |

### <a name="redundancy-and-designing-for-failure"></a>Redundância e a criação pensando em falha

Falhas podem variar quanto ao escopo do seu impacto. Algumas falhas de hardware, como um disco com falha, podem afetar um único computador host. Um comutador de rede com falha pode afetar um rack de servidor inteiro. Menos comumente, vemos falhas que interrompem um data center inteiro, tais como uma queda de energia em um data center. Raramente, toda uma região pode se tornar não disponível.

Uma das principais maneiras de se tornar um aplicativo resiliente é por meio de redundância. Mas você precisa planejar essa redundância ao projetar o aplicativo. Além disso, o nível de redundância necessário depende de suas necessidades de negócios &mdash; nem todo aplicativo precisa de redundância entre regiões para se proteger contra uma interrupção regional. Em geral, há um equilíbrio entre mais redundância e confiabilidade em relação a maiores custo e complexidade.  

O Azure tem um número de recursos para tornar um aplicativo redundante em cada nível de falha, desde uma VM individual até uma região inteira.

![Recursos de resiliência do Azure](./images/redundancy.svg)

**VM individual**. O Azure fornece um [SLA de tempo de atividade](https://azure.microsoft.com/support/legal/sla/virtual-machines) para VMs únicas. (A VM deve usar o armazenamento premium para todos os discos do sistema operacional e discos de dados.) Embora você possa obter um SLA mais alto executando duas ou mais VMs, uma única VM pode ser suficientemente confiável para algumas cargas de trabalho. No entanto, para cargas de trabalho de produção, é recomendável usar duas ou mais máquinas virtuais para redundância.

**Conjuntos de disponibilidade**. Para se proteger contra falhas de hardware localizadas, como falha em um comutador de rede ou de disco, implante duas ou mais VMs em um conjunto de disponibilidade. Um conjunto de disponibilidade consiste em dois ou mais *domínios de falha* que compartilham um comutador de rede e uma fonte de energia comuns. Máquinas virtuais em um conjunto de disponibilidade são distribuídas entre os domínios de falha, portanto, se uma falha de hardware afeta um domínio de falha, o tráfego de rede ainda poderá ser roteado para as VMs nos outros domínios de falha. Para obter mais informações sobre conjuntos de disponibilidade, consulte [Gerenciar a disponibilidade das máquinas virtuais do Windows no Azure](/azure/virtual-machines/windows/manage-availability).

**Zonas de disponibilidades**.  Uma Zona de Disponibilidade é uma zona fisicamente separada em uma região do Azure. Cada zona de disponibilidade tem uma rede, resfriamento e fonte de energia distintos. A implantação de VMs em zonas de disponibilidade ajuda a proteger um aplicativo contra falhas em todo o datacenter. Nem todas as regiões oferecem suporte às Zonas de Disponibilidade. Para obter uma lista de regiões com suporte e serviços, confira [O que são Zonas de Disponibilidade no Azure?](/azure/availability-zones/az-overview).

Se você estiver planejando usar as Zonas de Disponibilidade em sua implantação, primeiro confirme se sua arquitetura de aplicativo e a base de código podem dar suporte a essa configuração. Se você estiver implantando o software commercial off-the-shelf, consulte o fornecedor de software e teste adequadamente antes de implantar na produção. Um aplicativo deve ser capaz de manter o estado e evitar a perda de dados durante uma interrupção dentro da zona configurada. O aplicativo deve oferecer suporte à execução em uma infraestrutura distribuída e elástica com nenhum componente de infraestrutura embutido em código especificado na base de código.

**Azure Site Recovery**.  Replica as máquinas virtuais do Azure para outra região do Azure para necessidades de continuidade dos negócios e de recuperação de desastres. Você pode realizar análises periódicas de recuperação de desastres para garantir que atende às necessidades de conformidade. A máquina virtual será replicada com as configurações especificadas para a região selecionada para que você possa recuperar seus aplicativos em caso de interrupções na região de origem. Para obter mais informações, confira [Replicar VMs do Azure usando o ASR][site-recovery]. Considere os números de RTO e RPO para sua solução aqui e certifique-se de que, durante o teste, o ponto de recuperação e de tempo de recuperação é apropriado para suas necessidades.

**Regiões emparelhadas**. Para proteger um aplicativo contra uma interrupção regional, você pode implantar o aplicativo em várias regiões, usando o Gerenciador de Tráfego do Azure para distribuir o tráfego de Internet para as diferentes regiões. Cada região do Azure é emparelhada com outra. Juntas, elas formam um [par regional](/azure/best-practices-availability-paired-regions). Com a exceção do Sul do Brasil, pares regionais estão localizados na mesma região geográfica para atender aos requisitos de residência de dados para fins de jurisdição de imposição fiscal e legal.

Quando você cria um aplicativo de várias regiões, leve em consideração que a latência de rede entre diferentes regiões é maior do que a obtida em uma região. Por exemplo, se você estiver replicando de um banco de dados para habilitar o failover, use a replicação síncrona de dados dentro de uma região, mas a replicação assíncrona de dados entre regiões.

Ao selecionar as regiões emparelhadas, verifique se ambas exigiram serviços do Azure. Para obter uma lista de serviços por região, consulte [Produtos disponíveis por região](https://azure.microsoft.com/global-infrastructure/services/). Também é essencial selecionar a topologia de implantação correta para recuperação de desastre, especialmente se RPO/RTO são curtos. Para garantir que a região de failover tenha capacidade suficiente para dar suporte a sua carga de trabalho, selecione uma topologia ativa/passiva (réplica completa) ou uma topologia ativa/ativa. Lembre-se de que essas topologias de implantação podem aumentar a complexidade e o custo, já que os recursos na região secundária são pré-provisionados e podem ficar ociosos. Para obter mais informações, consulte [Topologias de implantação para recuperação de desastre][deployment-topologies]

| &nbsp; | Conjunto de disponibilidade | Zona de disponibilidade | Azure Site Recovery/Região emparelhada |
|--------|------------------|-------------------|---------------|
| Escopo da falha | Rack | Datacenter | Região |
| Roteamento de solicitação | Load Balancer | Load Balancer entre zonas | Gerenciador de Tráfego |
| Latência da rede | Muito baixa | Baixo | Média a alta |
| Rede virtual  | VNET | VNET | Emparelhamento VNET entre regiões |

## <a name="implement-resiliency-strategies"></a>Implementar estratégias de resiliência

Esta seção traz uma pesquisa de algumas estratégias comuns de resiliência. A maioria delas não se limita a uma tecnologia específica. As descrições nesta seção resumem a ideia geral de cada técnica, com links para leituras adicionais.

**Repita as falhas temporárias**. As falhas transitórias podem ser causadas por perda momentânea de conectividade de rede, interrupção na conexão com o banco de dados ou tempo limite atingido quando um serviço está ocupado. Frequentemente, uma falha temporária pode ser resolvida repetindo-se a solicitação. Em muitos serviços do Azure, o SDK do cliente implementa tentativas automáticas de forma transparente para o chamador. Consulte [Orientação específica sobre repetição de serviço][retry-service-specific guidance].

Cada tentativa de repetição é adicionada à latência total. Além disso, um número excessivo de solicitações com falha pode causar afunilamento, pois as solicitações pendentes se acumulam na fila. Essas solicitações bloqueadas podem reter recursos críticos do sistema, como memória, threads, conexões de banco de dados e outros, e provocar falhas em cascata. Para evitar isso, aumente o atraso entre as tentativas de repetição e limite o número total de solicitações com falha.

![Diagrama de tentativas de repetição](./images/retry.png)

**Balanceie a carga entre as instâncias**. Para ter escalabilidade, um aplicativo em nuvem deve ser capaz de se expandir com a adição de mais instâncias. Essa abordagem também aumenta a resiliência, porque as instâncias não íntegras podem ser removidas da rotação. Por exemplo: 

- Coloque duas ou mais VMs por trás de um balanceador de carga. O balanceador de carga distribui o tráfego para todas as VMs. Consulte [Executar VMs com balanceamento de carga para escalabilidade e disponibilidade][ra-multi-vm].
- Escale um aplicativo do Serviço de Aplicativo do Azure horizontalmente para várias instâncias. O Serviço de Aplicativo equilibra automaticamente a carga entre as instâncias. Consulte [Aplicativo Web básico][ra-basic-web].
- Use o [Gerenciador de Tráfego do Azure][tm] para distribuir o tráfego em um conjunto de pontos de extremidade.

**Replique os dados**. A replicação de dados é uma estratégia geral para manipular falhas não transitórias em um armazenamento de dados. Muitas tecnologias de armazenamento fornecem replicação interna, inclusive o Armazenamento do Microsoft Azure, o Banco de Dados SQL do Azure, o Cosmos DB e o Apache Cassandra. É importante considerar os caminhos de leitura e gravação. Dependendo da tecnologia de armazenamento, você pode ter várias réplicas graváveis, ou uma só réplica gravável e várias somente leitura.

Para maximizar a disponibilidade, as réplicas podem ser colocadas em várias regiões. No entanto, isso aumenta a latência na replicação de dados. Normalmente, a replicação entre regiões é feita de forma assíncrona, o que implica um modelo de consistência eventual e a possível perda de dados, se uma réplica falhar.

Você pode suar o [Azure Site Recovery][site-recovery] para replicar as máquinas virtuais do Azure de uma região para outra. O Site Recovery replica os dados continuamente para a região de destino. Quando ocorre uma interrupção no seu site primário, você faz failover para o local secundário

**Degradar normalmente**. Se um serviço falhar e não houver caminho de failover, o aplicativo pode conseguir degradar o serviço normalmente e continuar fornecendo uma experiência de usuário aceitável. Por exemplo: 

- Colocar um item de trabalho em uma fila a ser tratada mais tarde.
- Retornar um valor estimado.
- Usar dados armazenados em cache localmente.
- Exibir uma mensagem de erro para o usuário. (Essa opção é melhor do que o aplicativo parar de responder às solicitações.)

**Limite os usuários de alto volume**. Às vezes, um pequeno número de usuários cria uma carga excessiva. Isso pode afetar os outros, reduzindo a disponibilidade geral do aplicativo.

Quando um único cliente faz um número excessivo de solicitações, o aplicativo pode limitá-lo por um determinado período. Durante o período de limitação, o aplicativo recusa algumas ou todas as solicitações desse cliente (dependendo da estratégia exata de limitação). A extensão da limitação pode depender da camada de serviço do cliente.

A limitação não significa necessariamente que o cliente estava mal-intencionado, mas apenas que ele excedeu sua cota de serviço. Pode acontecer que o consumidor exceda sua cota frequentemente, ou que seu comportamento seja, de alguma forma, incorreto. Nesse caso, você pode ser mais enérgico e bloquear esse usuário. Normalmente, isso é feito bloqueando-se uma chave de API ou um intervalo de endereços IP. Para obter mais informações, consulte [Padrão de limitação][throttling-pattern].

**Use um disjuntor**. O padrão [Disjuntor][circuit-breaker-pattern] pode impedir que um aplicativo tente repetidamente uma operação com probabilidade de falha. O disjuntor encapsula as chamadas em um serviço e controla o número de falhas recentes. Se a contagem de falhas exceder um limite, o disjuntor será iniciado retornando um código de erro sem chamar o serviço. Isso fornece ao serviço um tempo de recuperação.

**Use o nivelamento de carga para suavizar os picos no tráfego**.
Os aplicativos podem enfrentar picos repentinos no tráfego e sobrecarregar o serviços de back-end. Se um serviço de back-end não conseguir responder às solicitações rápido o suficiente, pode provocar um enfileiramento de solicitações (backup) ou fazer com que o serviço limite o aplicativo. Para evitar isso, você pode usar uma fila como buffer. Quando há um novo item de trabalho, em vez de chamar o serviço de back-end imediatamente, o aplicativo enfileira um item de trabalho para que seja executado de forma assíncrona. A fila atua como buffer e diminui os picos na carga. Para obter mais informações, consulte [Padrão de nivelamento de carga baseado em fila][load-leveling-pattern].

**Isole os recursos críticos**. Às vezes, as falhas em um subsistema podem ocorrer em cascata, causando falhas em outras partes do aplicativo. Isso pode acontecer se uma falha impedir que alguns recursos, como threads ou soquetes, sejam liberados em tempo hábil, exaurindo recursos.

Para evitar isso, você pode particionar um sistema em grupos isolados, para que a falha de uma partição não paralise todo o sistema. Essa técnica também é conhecida como [padrão de Bulkhead][bulkhead-pattern].

Exemplos:

- Particionar um banco de dados (por locatário, por exemplo) e atribuir um pool de instâncias de servidor Web a cada partição.  
- Usar pools de threads separados, para isolar as chamadas para diferentes serviços. Isso ajuda a evitar falhas em cascata, se houver falha em um dos serviços. Para ver um exemplo, consulte a [biblioteca Hystrix][hystrix] da Netflix.
- Usar [contêineres][containers] para limitar os recursos disponíveis a um determinado subsistema.

![Diagrama do padrão de Bulkhead](./images/bulkhead.png)

**Aplique transações de compensação**. Um [transação de compensação][compensating-transaction-pattern] é uma transação que desfaz os efeitos de outra transação concluída. Em um sistema distribuído, pode ser muito difícil obter uma consistência transacional sólida. As transações de compensação são uma forma de obter consistência usando uma série de transações individuais menores, que podem ser desfeitas em cada etapa.

Por exemplo, para fazer uma viagem, um cliente pode precisar reservar um voo, acomodações de hotel e um carro. Se alguma dessas etapas falhar, toda a operação falhará. Em vez de tentar usar uma única transação distribuída em toda a operação, você pode definir uma transação de compensação para cada etapa. Por exemplo, para desfazer uma reserva de carro, cancelar essa reserva. Para concluir toda a operação, um coordenador executa cada etapa. Se alguma delas falhar, o coordenador aplica transações de compensação para desfazer todas as etapas concluídas.

## <a name="test-for-resiliency"></a>Testar resiliência

Em geral, não se pode testar a resiliência da mesma maneira que a funcionalidade do aplicativo (executando testes unitários e outros). Em vez disso, deve-se testar a execução da carga de trabalho de ponta a ponta sob condições de falha intermitentes.

O teste é um processo iterativo. Teste o aplicativo, avalie o resultado, analise e resolva as falhas resultantes, e repita o processo.

**Teste de injeção de falha**. Teste a resiliência do sistema durante falhas, seja disparando falhas reais ou fazendo simulações. Aqui estão alguns cenários comuns de falha para teste:

- desligamento de instâncias de VM
- falha de processos
- expiração de certificados
- mudança de chaves de acesso
- desligamento do serviço DNS em controladores de domínio
- limitação de recursos do sistema disponíveis, como RAM ou número de threads
- desmontagem de discos
- reimplantação de uma VM

Meça o tempo de recuperação e verifique se os seus requisitos de negócios estão sendo cumpridos. Teste também combinações dos modos de falha. Certifique-se de que as falhas não se propaguem em cascata e que sejam tratadas isoladamente.

Essa é outra razão pela qual é importante analisar possíveis pontos de falha durante a fase de design. Os resultados dessa análise devem funcionar como informações para o seu plano de teste.

**Teste de carga**. O teste de carga é crucial para identificar falhas que só ocorrem sob condições de carga, como um banco de dados de back-end sobrecarregado ou limitações de serviço. Teste a carga de pico usando dados de produção ou sintéticos, desde que estes se aproximem o máximo possível dos dados de produção. O objetivo é observar como o aplicativo se comporta sob condições do mundo real.

**Análises de recuperação de desastres**. Não basta ter um bom plano de recuperação de desastres em vigor. Você precisa testá-lo periodicamente para garantir que seu plano de recuperação funcione bem quando precisar. Para as máquinas virtuais do Azure, você pode usar o [Azure Site Recovery][site-recovery] para replicar e [executar análises de DR][site-recovery-test-failover], sem afetar os aplicativos de produção ou a replicação contínua.

## <a name="deploy-using-reliable-processes"></a>Implantar usando processos confiáveis

Quando um aplicativo é implantado para produção, as atualizações são uma possível fonte de erros. Na pior das hipóteses, uma atualização corrompida pode causar inatividade. Para evitar isso, o processo de implantação deve ser repetível e previsível. A implantação inclui etapas de provisionamento de recursos do Azure, implantação do código do aplicativo e aplicação de definições de configuração. Uma atualização pode envolver todas ou parte dessas três.

O ponto fundamental é que implantações manuais são propensas a erro. Por isso, é recomendável um processo idempotente automatizado, que você pode executar sob demanda e, caso haja alguma falha, executar novamente.

- Para automatizar o provisionamento de recursos do Azure, você pode usar [Terraform][terraform], [Ansible][ansible], [Chef][chef], [Puppet][puppet], [PowerShell][powershell], [CLI][cli] ou [modelos do Azure Resource Manager][template-deployment]
- Use o [Azure Automation Desired State Configuration][dsc] (DSC) para configurar VMs. Para VMs Linux, você pode usar o [Cloud-init][cloud-init].
- Você pode automatizar a implantação de aplicativos usando o [Azure DevOps Services][azure-devops-services] ou o [Jenkins][jenkins].

Dois conceitos relacionados à implantação resiliente são: *infraestrutura como código* e *infraestrutura imutável*.

- **Infraestrutura como código** é a prática de usar código para provisionar e configurar a infraestrutura. A infraestrutura como código pode usar abordagem declarativa ou imperativa (ou mesmo uma combinação de ambas). Os modelos do Resource Manager são um exemplo de abordagem declarativa. Os scripts do PowerShell são um exemplo de abordagem imperativa.
- **Infraestrutura imutável** é o princípio segundo o qual não se deve modificar a infraestrutura após sua implantação para produção. Caso contrário, você pode entrar em um estado em que foram aplicadas alterações ad hoc, no qual é difícil saber exatamente o que mudou e chegar a uma conclusão em relação ao sistema.

Outra questão é como lançar uma atualização do aplicativo. Recomendamos técnicas como as chamadas implantações “blue-green” e versões “canário”, que enviam atualizações de forma altamente controlada, visando a minimizar os possíveis impactos de uma implantação incorreta.

- A [Implantação “blue-green”][blue-green] é uma técnica em que a atualização é implantada em um ambiente de produção à parte do aplicativo ao vivo. Após validar a implantação, alterne o roteamento de tráfego para a versão atualizada. Por exemplo, o recurso Aplicativos Web do Serviço de Aplicativo do Azure permite isso com [slots de preparo][staging-slots].
- As [versões “canário”][canary-release] são semelhantes às implantações “blue-green”. Em vez de alternar todo o tráfego para a versão atualizada, lança-se a atualização para uma pequena porcentagem de usuários encaminhando uma parte do tráfego para a nova implantação. Se houver problemas, interrompe-se o processo volta-se à implantação anterior. Se tudo correr bem, encaminha-se mais uma parte do tráfego para a nova versão, até atingir 100%.

Qualquer que seja a abordagem, certifique-se de poder reverter para a última implantação íntegra, caso a nova versão não funcione. Tenha também uma estratégia para reverter as alterações do banco de dados e quaisquer outras alterações nos serviços dependentes. Se ocorrerem erros, os logs do aplicativo deverão indicar qual versão causou o erro.

## <a name="monitor-to-detect-failures"></a>Monitorar para detectar falhas

O monitoramento é essencial para garantir a resiliência. Se algo falhar, você precisa saber o que falhou, e também ter informações sobre a causa da falha.

O monitoramento de um sistema distribuído em larga escala representa um desafio significativo. Pense em um aplicativo executado em algumas dezenas de VMs; &mdash;não é prático fazer logon em cada uma delas e examinar seus arquivos de log para tentar solucionar um problema. Além disso, o número de instâncias de VMs provavelmente não são VMs adicionadas e removidas com a expansão e retração do aplicativo e, ocasionalmente, uma instância pode falhar e precisar ser provisionada novamente. Além disso, um aplicativo de nuvem típico pode usar vários armazenamentos de dados (armazenamento do Azure, Banco de Dados SQL, Cosmos DB, cache Redis) e uma única ação do usuário pode abranger vários subsistemas.

Você pode pensar no processo de monitoramento como um pipeline com vários estágios distintos:

![SLA composto](./images/monitoring.png)

- **Instrumentação**. Os dados brutos para monitoramento são provenientes de várias fontes, incluindo [logs dos aplicativos](/azure/application-insights/app-insights-overview?toc=/azure/azure-monitor/toc.json), [métricas de desempenho de sistemas operacionais](/azure/azure-monitor/platform/agents-overview), [recursos do Azure](/azure/monitoring-and-diagnostics/monitoring-supported-metrics?toc=/azure/azure-monitor/toc.json), [assinaturas do Azure](/azure/service-health/service-health-overview) e [locatários do Azure](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics). A maioria dos serviços do Azure expõe [métricas](/azure/azure-monitor/platform/data-collection) que você pode configurar para analisar e determinar a causa dos problemas.
- **Coleta e armazenamento**. Dados de instrumentação brutos podem ser mantidos em vários locais e com vários formatos (por exemplo, logs de rastreamento de aplicativo e do IIS, contadores de desempenho). Essas fontes diferentes são coletadas, consolidadas e colocadas em armazenamentos de dados confiáveis, como Application Insights, métricas do Azure Monitor, Integridade do Serviço, contas de armazenamento e Log Analytics.
- **Análise e diagnóstico**. Após os dados serem consolidados nesses armazenamentos de dados diferentes, eles podem ser analisados para solucionar problemas e fornecer uma visão geral da integridade do aplicativo. Geralmente, você pode pesquisar pelos dados no Application Insights e no Log Analytics usando [consultas do Kusto](/azure/log-analytics/log-analytics-queries). O Assistente do Azure fornece recomendações com foco em [resiliência](/azure/advisor/advisor-high-availability-recommendations) e [otimização](/azure/advisor/advisor-performance-recommendations).
- **Visualização e alertas**. Neste estágio, os dados de telemetria são apresentados de forma que um operador pode notar rapidamente problemas ou tendências. Exemplos incluem painéis ou alertas por email. Com os [painéis do Azure](/azure/azure-portal/azure-portal-dashboards), você pode criar uma exibição de painel único dos gráficos de monitoramento originários do Application Insights, do Log Analytics, das métricas do Azure Monitor e da integridade do serviço. Com os [alertas do Azure Monitor](/azure/monitoring-and-diagnostics/monitoring-overview-alerts?toc=/azure/azure-monitor/toc.json), você pode criar alertas sobre a integridade do recurso e do serviço.

Monitoramento não é o mesmo que detecção de falha. Por exemplo, seu aplicativo pode detectar um erro transitório e tentar novamente, sem que haja qualquer inatividade. Mas ele deve registrar essa operação de repetição, para que se possa monitorar a taxa de erros e ter uma visão geral da integridade do aplicativo.

Os logs de aplicativo são uma fonte importante de dados de diagnóstico. Algumas práticas recomendadas para o log de aplicativo são:

- Obtenha logs na fase de produção. Caso contrário, você perderá informações onde mais precisa.
- Obtenha logs de eventos nos limites dos serviços. Inclua uma ID de correlação que flua nos limites dos serviços. Se uma transação fluir em vários serviços e houver uma falha, a ID de correlação ajudará a identificar o motivo da falha da transação.
- Use o log semântico, também conhecido como registro em log estruturado. Os logs não estruturados dificultam a automatização do consumo e da análise dos dados do log, necessários em escala de nuvem.
- Use logs assíncronos. Caso contrário, o próprio sistema de registro em log pode provocar falha do aplicativo com solicitações de backup, pois ocorre bloqueio durante a espera para gravar um evento de log.
- Log de aplicativo não é o mesmo que auditoria. A auditoria pode ser feita por motivos regulatórios ou de conformidade. Sendo assim, os registros de auditoria devem ser concluídos, não sendo aceitável remover nenhum deles durante o processamento de transações. Se um aplicativo exigir auditoria, ela deve ser mantida à parte do log de diagnóstico.

Para obter mais informações sobre monitoramento e diagnóstico, consulte [Diretrizes de monitoramento e diagnóstico][monitoring-guidance].

## <a name="respond-to-failures"></a>Responder a falhas

As seções anteriores se concentraram em estratégias de recuperação automatizada, que são essenciais para a alta disponibilidade. No entanto, às vezes, a intervenção manual, é necessária.

- **Alertas**. Monitore seu aplicativo, observando sinais de aviso que podem exigir intervenção proativa. Por exemplo, se você vir que o Banco de Dados SQL ou o Cosmos DB limitam repetidamente o seu aplicativo, talvez seja preciso aumentar a capacidade do banco de dados ou otimizar suas consultas. Neste exemplo, mesmo que o aplicativo consiga manipular esses erros de limitação de forma transparente, sua telemetria deve gerar um alerta, para que você possa acompanhar. É recomendável configurar alertas nas métricas de recursos do Azure e nos logs de diagnóstico em relação aos limites de serviços e de cotas. Recomendamos configurar alertas sobre métricas, pois eles são de baixa latência em comparação com os logs de diagnóstico. Além disso, o Azure pode fornecer um status de integridade pronto para uso por meio da [integridade do recurso](/azure/service-health/resource-health-checks-resource-types), o que pode ajudar a diagnosticar a limitação dos serviços do Azure.
- **Failover**. Configure uma estratégia de recuperação de desastre no seu aplicativo. A estratégia apropriada dependerá dos seus SLAs. Na maioria dos cenários, uma implementação ativa-passiva é suficiente. Para obter mais informações, consulte [Topologias de implantação para recuperação de desastre ](./disaster-recovery-azure-applications.md#deployment-topologies-for-disaster-recovery). A maioria dos serviços do Azure permite failover manual ou automatizado. Por exemplo, em um aplicativo IaaS, use o [Azure Site Recovery](/azure/site-recovery/azure-to-azure-architecture) para as camadas Web e lógica e [Grupos de Disponibilidade SQL AlwaysOn](/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-dr) para a camada do banco de dados. O [Gerenciador de Tráfego](/azure/traffic-manager/traffic-manager-overview) fornece failover automatizado entre regiões.
- **Testes de preparação operacional**. Realize um teste de prontidão operacional para failover na região secundária e failback para a região primária. Muitos serviços do Azure são compatíveis com failover manual ou failover de teste para exercícios de recuperação de desastre. Como alternativa, você pode desligar ou remover serviços para simular uma interrupção.
- **Verificação de consistência de dados**. Se houver falha em um armazenamento de dados, pode haver inconsistências de dados quando o armazenamento voltar a ser disponibilizado, especialmente se os dados tiverem sido replicados. Para serviços do Azure que fornecem replicação entre regiões, analise o RTO e o RPO para entender a perda de dados esperada em uma falha. Analise os SLAs dos serviços do Azure para saber se o failover entre regiões pode ser iniciado manualmente ou pela Microsoft. Para alguns serviços, a Microsoft decide quando executar o failover. A Microsoft pode priorizar a recuperação de dados na região primária, somente fazendo failover para uma região secundária se os dados na região primária forem considerados irrecuperáveis. Por exemplo, [armazenamento com redundância geográfica](/azure/storage/common/storage-redundancy-grs) e [Key Vault](/azure/key-vault/key-vault-disaster-recovery-guidance) seguem este modelo.
- **Restauração com base em backup**. Em alguns cenários, a restauração de backup só será possível dentro da mesma região. Esse é o caso para [Backup de VMs do Azure](/azure/backup/backup-azure-vms-first-look-arm). Outros serviços do Azure fornecem backups replicados geograficamente, como [Réplicas Geográficas do Cache Redis](/azure/redis-cache/cache-how-to-geo-replication). O objetivo dos backups é proteger contra exclusão acidental ou corrupção de dados, restaurando o aplicativo para uma versão funcional no início do tempo. Portanto, embora os backups possam servir como uma solução de recuperação de desastre em alguns casos, o inverso nem sempre é verdadeiro: A recuperação de desastres não protege você contra exclusão acidental ou corrupção de dados.  

Documente e teste seu plano de recuperação de desastres. Avalie o impacto das falhas de aplicativos sobre os negócios. Automatize o processo tanto quanto possível, e documente todas as etapas manuais, como failover manual ou restauração de dados com base em backups. Teste regularmente seu processo de recuperação de desastres, para validar e aprimorar o plano. Configure alertas para os serviços do Azure consumidos pelo seu aplicativo.

## <a name="summary"></a>Resumo

Este artigo discutiu a resiliência sob uma perspectiva holística, enfatizando alguns dos desafios exclusivos da nuvem. Eles incluem a natureza distribuída de computação em nuvem, o uso de hardware de mercadoria e a presença de falhas de rede temporárias.

Estes são os principais pontos a serem lembrados neste artigo:

- A resiliência leva à maior disponibilidade e diminui o tempo médio de recuperação de falhas.
- A resiliência na nuvem requer um conjunto de técnicas diferentes das soluções tradicionais locais.
- A resiliência não é acidental. Ela deve ser projetada e construída desde o início.
- Resiliência é algo que deve existir em todas as partes do ciclo de vida do aplicativo, do planejamento e da codificação às operações.
- Teste e monitore!

<!-- links -->

[blue-green]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[canary-release]: https://martinfowler.com/bliki/CanaryRelease.html
[circuit-breaker-pattern]: ../patterns/circuit-breaker.md
[compensating-transaction-pattern]: ../patterns/compensating-transaction.md
[containers]: https://en.wikipedia.org/wiki/Operating-system-level_virtualization
[dsc]: /azure/automation/automation-dsc-overview
[contingency-planning-guide]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-34r1.pdf
[fma]: ./failure-mode-analysis.md
[hystrix]: https://medium.com/netflix-techblog/introducing-hystrix-for-resilience-engineering-13531c1ab362
[jmeter]: https://jmeter.apache.org/
[load-leveling-pattern]: ../patterns/queue-based-load-leveling.md
[monitoring-guidance]: ../best-practices/monitoring.md
[ra-basic-web]: ../reference-architectures/app-service-web-app/basic-web-app.md
[ra-multi-vm]: ../reference-architectures/virtual-machines-windows/multi-vm.md
[checklist]: ../checklist/resiliency.md
[retry-pattern]: ../patterns/retry.md
[retry-service-specific guidance]: ../best-practices/retry-service-specific.md
[sla]: https://azure.microsoft.com/support/legal/sla/
[throttling-pattern]: ../patterns/throttling.md
[tm]: https://azure.microsoft.com/services/traffic-manager/
[tm-failover]: /azure/traffic-manager/traffic-manager-monitoring
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager
[site-recovery]: /azure/site-recovery/azure-to-azure-quickstart/
[site-recovery-test-failover]: /azure/site-recovery/azure-to-azure-tutorial-dr-drill/
[site-recovery-failover]: /azure/site-recovery/azure-to-azure-tutorial-failover-failback/
[deployment-topologies]: ./disaster-recovery-azure-applications.md#deployment-topologies-for-disaster-recovery
[bulkhead-pattern]: ../patterns/bulkhead.md
[terraform]: /azure/virtual-machines/windows/infrastructure-automation#terraform
[ansible]: /azure/virtual-machines/windows/infrastructure-automation#ansible
[chef]: /azure/virtual-machines/windows/infrastructure-automation#chef
[puppet]: /azure/virtual-machines/windows/infrastructure-automation#puppet
[template-deployment]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[cloud-init]: /azure/virtual-machines/windows/infrastructure-automation#cloud-init
[azure-devops-services]: /azure/virtual-machines/windows/infrastructure-automation#azure-devops-services
[jenkins]: /azure/virtual-machines/windows/infrastructure-automation#jenkins
[staging-slots]: /azure/app-service/deploy-staging-slots
[powershell]: /powershell/azure/overview
[cli]: /cli/azure
