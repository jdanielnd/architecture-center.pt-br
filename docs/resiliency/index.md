---
title: Desenvolvimento de aplicativos resilientes para o Azure
description: Como criar aplicativos resilientes no Azure, para alta disponibilidade e recuperação de desastres.
author: MikeWasson
ms.date: 05/26/2017
ms.custom: resiliency
pnp.series.title: Design for Resiliency
ms.openlocfilehash: 9a6bd1332ea59923b32379018060403024b15e10
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/16/2018
---
# <a name="designing-resilient-applications-for-azure"></a>Desenvolvimento de aplicativos resilientes para o Azure

Em um sistema distribuído, ocorrerão falhas. O hardware pode falhar. A rede pode ter falhas transitórias. Raramente, um serviço ou uma região inteira pode sofrer uma interrupção. Mesmo assim, devemos estar preparados para isso. 

Criar um aplicativo confiável na nuvem é diferente da criação de um aplicativo confiável em uma configuração empresarial. Embora historicamente você possa ter comprado um hardware superior para escalar verticalmente, em um ambiente de nuvem, é preciso escalar horizontalmente. Os custos para ambientes em nuvem são mantidos baixos com o uso de hardware de mercadoria. Em vez de nos concentrarmos em evitar falhas e otimizar o "tempo médio entre falhas", neste novo ambiente, o foco muda para "tempo médio de restauração." O objetivo é minimizar o efeito de uma falha.

Este artigo fornece uma visão geral de como criar aplicativos resilientes no Microsoft Azure. Ele começa com uma definição do termo *resiliência* e conceitos de relacionados. Em seguida, descreve um processo para a obtenção de resiliência, usando uma abordagem estruturada sobre o tempo de vida de um aplicativo, desde o design e a implementação até a implantação e as operações.

## <a name="what-is-resiliency"></a>O que é resiliência?
**Resiliência** é a capacidade de um sistema de se recuperar de falhas e continuar funcionando. Não se trata de *evitar* falhas, mas *responder* a elas de uma maneira que evite o tempo de inatividade ou a perda de dados. A meta de resiliência é retornar o aplicativo a um estado totalmente funcional após uma falha.

Dois aspectos importantes de resiliência são alta disponibilidade e recuperação de desastres.

* **HA (Alta Disponibilidade)** é a capacidade do aplicativo de continuar em execução em um estado íntegro, sem tempo de inatividade significativo. Por “estado íntegro” queremos dizer o aplicativo está respondendo e os usuários podem se conectar ao aplicativo e interagir com ele.  
* **DR (Recuperação de Desastre)** é a capacidade de recuperação de incidentes raros, mas sérios: falhas não transitórias em larga escala, como interrupções de serviço que afetam toda a região. A recuperação de desastre inclui backup e arquivamento de dados e pode incluir a intervenção manual, como a restauração um banco de dados com base em um backup.

Uma das formas de diferenciar a HA da DR é que a última é iniciada quando o impacto de uma falha excede a capacidade do design da HA para tratá-la.  

Quando você projeta buscando resiliência, você deve entender seus requisitos de disponibilidade. Quanto tempo de inatividade é aceitável? Trata-se parcialmente de uma questão de custo. Quanto o tempo de inatividade em potencial custará para sua empresa? Quanto você deve investir para tornar o aplicativo altamente disponível? Você também precisa definir o que significa para o aplicativo estar disponível. Por exemplo, o aplicativo está "inativo" se um cliente puder enviar uma ordem, mas o sistema não puder processá-la dentro do prazo normal? Considere também a probabilidade da ocorrência de um tipo específico de interrupção e se uma estratégia de redução é econômica.

Outro termo comum é **Continuidade dos Negócios** (BC), que é a capacidade de executar funções de negócios essenciais durante e após condições adversas, como um desastre natural ou um serviço inoperante. A BC abrange toda a operação de negócios, incluindo instalações físicas, pessoas, comunicações, transporte e IT. Este artigo se concentra em aplicativos em nuvem, mas o planejamento de resiliência deve ser feito considerando-se o contexto dos requisitos gerais da BC. 

O **backup de dados** é uma parte fundamental da recuperação de desastre. Se os componentes sem monitoração de estado de um aplicativo falharem, você sempre poderá reimplantá-los. Mas se os dados forem perdidos, o sistema não poderá retornar a um estado estável. O backup dos dados deve ser feito, idealmente em uma região diferente para o caso de um desastre em toda a região. 

Backup é diferente de **replicação de dados**. Replicação de dados envolve a cópia de dados quase em tempo real, para que o sistema possa realizar failover rapidamente para uma réplica. Muitos sistemas de bancos de dados são compatíveis com replicação; Por exemplo, o SQL Server é compatível com Grupos de Disponibilidade Always On do SQL Server. A replicação de dados pode reduzir o tempo necessário para recuperar-se de uma interrupção, garantindo que uma réplica dos dados esteja sempre disponível. No entanto, a replicação de dados não protegerá contra erro humano. Se os dados forem corrompidos devido a erro humano, eles serão apenas copiados para as réplicas. Portanto, você ainda precisa incluir o backup de longo prazo em sua estratégia de DR. 

## <a name="process-to-achieve-resiliency"></a>Processo para obter resiliência
Resiliência não é um complemento. Ela deve ser criada no sistema e colocada em prática operacional. Aqui está um modelo geral a seguir:

1. **Defina** seus requisitos de disponibilidade, com base nas necessidades de negócios.
2. **Crie** o aplicativo para garantir a resiliência. Inicie com uma arquitetura que segue práticas comprovadas e, em seguida, identifique os pontos de falha possíveis na arquitetura.
3. **Implemente** estratégias para detectar e recuperar de falhas. 
4. **Teste** a implementação simulando falhas e disparando failovers forçados. 
5. **Implante** o aplicativo em produção usando um processo confiável e reproduzível. 
6. **Monitore** o aplicativo para detectar falhas. Ao monitorar o sistema, você pode medir a integridade do aplicativo e responder a incidentes, se necessário. 
7. **Responda** se há incidentes que exijam intervenções manuais.

No restante deste artigo, vamos abordar cada uma dessas etapas mais detalhadamente.

## <a name="defining-your-resiliency-requirements"></a>Definição dos seus requisitos de resiliência
O planejamento da resiliência começa com os requisitos de negócios. A seguir estão algumas abordagens para pensarmos sobre resiliência sob esses aspectos.

### <a name="decompose-by-workload"></a>Decompor por carga de trabalho
Muitas soluções de nuvem consistem em várias cargas de trabalho de aplicativos. O termo "carga de trabalho" neste contexto significa um recurso discreto ou uma tarefa de computação, que pode ser logicamente separada de outras tarefas, em termos de requisitos de armazenamento de dados e lógica de negócios. Por exemplo, um aplicativo de comércio eletrônico pode incluir as cargas de trabalho a seguir:

* Procura e pesquisa em um catálogo de produtos.
* Criação e controle de pedidos.
* Exibição de recomendações.

Essas cargas de trabalho podem ter diferentes requisitos de disponibilidade, escalabilidade, consistência de dados, recuperação de desastres e assim por diante. Novamente, essas são decisões comerciais.

Considere também os padrões de uso. Há certos períodos críticos quando o sistema tem que estar disponível? Por exemplo, um serviço de arquivamento de imposto não pode ficar inativo antes do prazo de entrega, um serviço de streaming de vídeo tem que continuar durante um evento esportivo importante, e assim por diante. Durante os períodos críticos, talvez sejam necessárias implantações redundantes em várias regiões, para que o aplicativo possa fazer failover se uma região falhar. No entanto, uma implantação em várias regiões é mais cara, portanto, durante horários menos críticos, você pode executar o aplicativo em uma única região.

### <a name="rto-and-rpo"></a>RTO e RPO
Duas métricas importantes a considerar são o Objetivo do Tempo de Recuperação e o Objetivo do Ponto de Recuperação.

* **Objetivo do Tempo de Recuperação** (RTO) é o tempo máximo aceitável que um aplicativo pode ficar indisponível após um incidente. Se o RTO for de 90 minutos, você deverá ser capaz de restaurar o aplicativo para um estado de execução dentro de 90 minutos a partir do início de um desastre. Se você tiver um RTO muito baixo, poderá manter uma segunda implantação continuamente em execução no modo de espera, para proteger-se contra uma interrupção regional.

* **Objetivo do Ponto de Recuperação** (RPO) é a duração máxima de perda de dados aceitável durante um desastre. Por exemplo, se você armazena dados em um único banco de dados, com nenhuma replicação para outros bancos de dados, e executar backups a cada hora, poderá perder até uma hora de dados. 

RTO e RPO são os requisitos de negócios. Realizar uma avaliação de risco pode ajudá-lo a definir o RTO e o RPO do aplicativo. Outra métrica comum é **Tempo Médio para Recuperar** (MTTR), que é o tempo médio necessário para restaurar o aplicativo após uma falha. MTTR é um fato empírico sobre um sistema. Se o MTTR exceder o RTO, uma falha no sistema causará uma interrupção do negócio inaceitável, porque não é possível restaurar o sistema dentro do RTO definido. 

### <a name="slas"></a>SLAs
O [SLA (Contrato de Nível de Serviço)][sla] descreve os compromissos da Microsoft com relação ao tempo de atividade e à conectividade. Se o SLA para um serviço específico for de 99,9%, isso significa que você deve esperar que o serviço esteja disponível 99,9% do tempo.

> [!NOTE]
> O SLA do Azure também inclui provisões para se obter um crédito de serviço se o SLA não for atendido, além de definições específicas de "disponibilidade" para cada serviço. Esse aspecto do SLA atua como uma política de imposição. 
> 
> 

Você deve definir seus próprios SLAs de destino para cada carga de trabalho em sua solução. Um SLA possibilita avaliar se a arquitetura atende aos requisitos de negócios. Por exemplo, se uma carga de trabalho requer tempo de disponibilidade de 99,99%, mas depende de um serviço com um SLA de 99,9%, o serviço não poderá ser um ponto único de falha no sistema. Uma solução é ter um caminho de fallback no caso de falha do serviço ou tomar outras medidas para recuperar uma falha nesse serviço. 

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

* Para obter quatro 9s (99,99%), você provavelmente não pode depender de intervenção manual para se recuperar de falhas. O aplicativo deve ser de autodiagnóstico e autorrecuperação. 
* Além de quatro 9s, é difícil detectar problemas rápido o suficiente para atender ao SLA.
* Pense no intervalo de tempo ao qual a medição do seu SLA está relacionada. Quanto menor for o intervalo, menor será a tolerância. Provavelmente não faz sentido definir seu SLA em termos de tempo de atividade por hora ou dia. 

### <a name="composite-slas"></a>SLAs compostos
Considere um aplicativo Web do Serviço de Aplicativo que grava no banco de dados do SQL Azure. No momento da redação deste artigo, esses serviços do Azure têm os seguintes SLAs:

* Aplicativos Web do Serviço de Aplicativo = 99,95%
* Banco de dados SQL = 99,99%

![SLA composto](./images/sla1.png)

Qual é o tempo de inatividade máximo esperado para este aplicativo? Se o serviço falhar, o aplicativo inteiro falhará. Em geral, a probabilidade de cada serviço com falha é independente, portanto, o SLA composto para este aplicativo é de 99,95% &times; 99,99% = % 99,94%. Isso é menos do que os SLAs individuais, o que não é surpresa, porque um aplicativo que depende de vários serviços possui mais pontos de falha em potencial. 

Por outro lado, você pode melhorar o SLA composto criando caminhos de fallback independentes. Por exemplo, se o banco de dados SQL não estiver disponível, coloque as transações em uma fila, para serem processadas posteriormente.

![SLA composto](./images/sla2.png)

Com esse design, o aplicativo ainda estará disponível mesmo se não puder se conectar ao banco de dados. No entanto, ele falhará se o banco de dados e a fila falharem ao mesmo tempo. A porcentagem esperada de tempo de uma falha simultânea é de 0,0001 &times; 0,001, portanto, o SLA composto para este caminho combinado é:  

* Banco de dados OU fila = 1,0 &minus; (0,0001 &times; 0,001) = 99,99999%

O SLA composto total é:

* Aplicativo Web E (banco de dados OU fila) = 99,95% &times; 99,99999% = ~99,95%

Mas existem compensações para essa abordagem. A lógica do aplicativo é mais complexa. Você está pagando pela fila e pode haver problemas de consistência de dados a serem considerados.

**SLA para implantações em várias regiões**. Outra técnica de HA é implantar o aplicativo em mais de uma região e usar o Gerenciador de Tráfego do Azure para fazer o failover se o aplicativo falhar em uma região. Para uma implantação de duas regiões, o SLA composto é calculado da maneira a seguir. 

Permita que *N* seja o SLA composto para o aplicativo implantado em uma região. A possibilidade esperada de que o aplicativo falhará em ambas as regiões ao mesmo tempo será de (1 &minus; N) &times; (1 &minus; N). Portanto:

* SLA combinado para ambas as regiões = 1 &minus; (1 &minus; N)(1 &minus; N) = N + (1 &minus; N)N

Por fim, você deve considerar o [SLA para o Gerenciador de Tráfego][tm-sla]. No momento da redação deste artigo, o SLA para o SLA do Gerenciador de Tráfego é de 99,99%.

* SLA composto = 99,99% &times; (SLA combinado para ambas as regiões)

Além disso, o failover não é instantâneo e pode resultar em algum tempo de inatividade durante um failover. Consulte [Monitoramento e failover de ponto de extremidade do Gerenciador de tráfego][tm-failover].

O número calculado de SLA é uma linha de base útil, mas ela não informa a história toda sobre a disponibilidade. Geralmente, um aplicativo pode degradar normalmente quando um caminho não crítico falha. Considere um aplicativo que mostra um catálogo de livros. Se o aplicativo não pode recuperar a imagem em miniatura da capa, poderá mostrar uma imagem de espaço reservado. Nesse caso, a falha ao obter a imagem não reduz o tempo de atividade do aplicativo, embora afete a experiência do usuário.  

## <a name="redundancy-and-designing-for-failure"></a>Redundância e a criação pensando em falha

Falhas podem variar quanto ao escopo do seu impacto. Algumas falhas de hardware, como um disco com falha, podem afetar um único computador host. Um comutador de rede com falha pode afetar um rack de servidor inteiro. Menos comuns são falhas que interrompem um data center inteiro, tais como uma perda de energia em um data center. Raramente, toda uma região pode se tornar não disponível.

Uma das principais maneiras de se tornar um aplicativo resiliente é por meio de redundância. Mas você precisa planejar essa redundância ao projetar o aplicativo. Além disso, o nível de redundância necessário depende de suas necessidades de negócios &mdash; nem todo aplicativo precisa de redundância entre regiões para se proteger contra uma interrupção regional. Em geral, há um equilíbrio entre mais redundância e confiabilidade em relação a maiores custo e complexidade.  

O Azure tem um número de recursos para tornar um aplicativo redundante em cada nível de falha, desde uma VM individual até uma região inteira. 

![](./images/redundancy.svg)

**VM individual**. O Azure fornece um SLA de tempo de atividade para VMs individuais. Embora você possa obter um SLA mais alto executando duas ou mais VMs, uma única VM pode ser suficientemente confiável para algumas cargas de trabalho. Para cargas de trabalho de produção, é recomendável usar duas ou mais máquinas virtuais para redundância. 

**Conjuntos de disponibilidade**. Para se proteger contra falhas de hardware localizadas, como falha em um comutador de rede ou de disco, implante duas ou mais VMs em um conjunto de disponibilidade. Um conjunto de disponibilidade consiste em dois ou mais *domínios de falha* que compartilham um comutador de rede e uma fonte de energia comuns. Máquinas virtuais em um conjunto de disponibilidade são distribuídas entre os domínios de falha, portanto, se uma falha de hardware afeta um domínio de falha, o tráfego de rede ainda poderá ser roteado para as VMs nos outros domínios de falha. Para obter mais informações sobre conjuntos de disponibilidade, consulte [Gerenciar a disponibilidade das máquinas virtuais do Windows no Azure](/azure/virtual-machines/windows/manage-availability).

**Zonas de disponibilidades**.  Uma Zona de Disponibilidade é uma zona fisicamente separada em uma região do Azure. Cada zona de disponibilidade tem uma rede, resfriamento e fonte de energia distintos. A implantação de VMs em zonas de disponibilidade ajuda a proteger um aplicativo contra falhas em todo o datacenter. 

**Regiões emparelhadas**. Para proteger um aplicativo contra uma interrupção regional, você pode implantar o aplicativo em várias regiões, usando o Gerenciador de Tráfego do Azure para distribuir o tráfego de Internet para as diferentes regiões. Cada região do Azure é emparelhada com outra. Juntas, elas formam um [par regional](/azure/best-practices-availability-paired-regions). Com a exceção do Sul do Brasil, pares regionais estão localizados na mesma região geográfica para atender aos requisitos de residência de dados para fins de jurisdição de imposição fiscal e legal.

Quando você cria um aplicativo de várias regiões, leve em consideração que a latência de rede entre diferentes regiões é maior do que a obtida em uma região. Por exemplo, se você estiver replicando de um banco de dados para habilitar o failover, use a replicação síncrona de dados dentro de uma região, mas a replicação assíncrona de dados entre regiões.

| &nbsp; | Conjunto de disponibilidade | Zona de disponibilidade | Região emparelhada |
|--------|------------------|-------------------|---------------|
| Escopo da falha | Rack | Datacenter | Região |
| Roteamento de solicitação | Balanceador de carga | Load Balancer entre zonas | Gerenciador de Tráfego |
| Latência da rede | Muito baixa | Baixo | Média a alta |
| Rede virtual  | VNET | VNET | Emparelhamento VNET entre regiões |

## <a name="designing-for-resiliency"></a>Design para resiliência
Durante a fase de design, você deve executar uma Análise do Modo de Falha (FMA). O objetivo de uma FMA é identificar possíveis pontos de falha e definir como o aplicativo responde a essas falhas.

* Como o aplicativo detectará esse tipo de falha?
* Como o aplicativo responderá a esse tipo de falha?
* Como você pode fazer logon e monitorar esse tipo de falha? 

Para obter mais informações sobre o processo FMA, com as recomendações específicas para o Azure, consulte [Orientação de resiliência do Azure: Análise de Falha de Modo][fma].

### <a name="example-of-identifying-failure-modes-and-detection-strategy"></a>Exemplo de identificação de modos de falha e estratégia de detecção
**Ponto de falha:** chamada para um serviço Web externo/API.

| Modo de falha | Estratégia de detecção |
| --- | --- |
| O serviço está indisponível |HTTP 5xx |
| Limitação |HTTP 429 (Número Excessivo de Solicitações) |
| Autenticação |HTTP 401 (Não Autorizado) |
| Resposta lenta |Tempo limite da solicitação atingido |

## <a name="resiliency-strategies"></a>Estratégias de resiliência
Esta seção traz uma pesquisa de algumas estratégias comuns de resiliência. A maioria delas não se limita a uma tecnologia específica. As descrições nesta seção resumem a ideia geral de cada técnica, com links para leituras adicionais.

### <a name="retry-transient-failures"></a>Repetição de falhas transitórias
As falhas transitórias podem ser causadas por perda momentânea de conectividade de rede, interrupção na conexão com o banco de dados ou tempo limite atingido quando um serviço está ocupado. Frequentemente, uma falha temporária pode ser resolvida repetindo-se a solicitação. Em muitos serviços do Azure, o SDK do cliente implementa tentativas automáticas de forma transparente para o chamador. Consulte [Orientação específica sobre repetição de serviço][retry-service-specific guidance].

Cada tentativa de repetição é adicionada à latência total. Além disso, um número excessivo de solicitações com falha pode causar afunilamento, pois as solicitações pendentes se acumulam na fila. Essas solicitações bloqueadas podem reter recursos críticos do sistema, como memória, threads, conexões de banco de dados e outros, e provocar falhas em cascata. Para evitar isso, aumente o atraso entre as tentativas de repetição e limite o número total de solicitações com falha.

![SLA composto](./images/retry.png)

Para obter mais informações, consulte [Padrão de repetição][retry-pattern].

### <a name="load-balance-across-instances"></a>Balanceamento de carga entre instâncias
Para ter escalabilidade, um aplicativo em nuvem deve ser capaz de se expandir com a adição de mais instâncias. Essa abordagem também aumenta a resiliência, porque as instâncias não íntegras podem ser removidas da rotação.  

Por exemplo: 

* Coloque duas ou mais VMs por trás de um balanceador de carga. O balanceador de carga distribui o tráfego para todas as VMs. Consulte [Executar VMs com balanceamento de carga para escalabilidade e disponibilidade][ra-multi-vm].
* Escale um aplicativo do Serviço de Aplicativo do Azure horizontalmente para várias instâncias. O Serviço de Aplicativo equilibra automaticamente a carga entre as instâncias. Consulte [Aplicativo Web básico][ra-basic-web].
* Use o [Gerenciador de Tráfego do Azure][tm] para distribuir o tráfego em um conjunto de pontos de extremidade.

### <a name="replicate-data"></a>Replicar dados
A replicação de dados é uma estratégia geral para manipular falhas não transitórias em um armazenamento de dados. Muitas tecnologias de armazenamento fornecem replicação interna, inclusive o Banco de Dados SQL do Azure, Cosmos DB e Apache Cassandra.  

É importante considerar os caminhos de leitura e gravação. Dependendo da tecnologia de armazenamento, você pode ter várias réplicas graváveis, ou uma só réplica gravável e várias somente leitura. 

Para maximizar a disponibilidade, as réplicas podem ser colocadas em várias regiões. No entanto, isso aumenta a latência na replicação de dados. Normalmente, a replicação entre regiões é feita de forma assíncrona, o que implica um modelo de consistência eventual e a possível perda de dados, se uma réplica falhar. 

### <a name="degrade-gracefully"></a>Degradar normalmente
Se um serviço falhar e não houver caminho de failover, o aplicativo pode conseguir degradar o serviço normalmente e continuar fornecendo uma experiência de usuário aceitável. Por exemplo: 

* Colocar um item de trabalho em uma fila a ser tratada mais tarde. 
* Retornar um valor estimado.
* Usar dados armazenados em cache localmente. 
* Exibir uma mensagem de erro para o usuário. (Essa opção é melhor do que o aplicativo parar de responder às solicitações.)

### <a name="throttle-high-volume-users"></a>Limitar os usuários de alto volume
Às vezes, um pequeno número de usuários cria uma carga excessiva. Isso pode afetar os outros, reduzindo a disponibilidade geral do aplicativo.

Quando um único cliente faz um número excessivo de solicitações, o aplicativo pode limitá-lo por um determinado período. Durante o período de limitação, o aplicativo recusa algumas ou todas as solicitações desse cliente (dependendo da estratégia exata de limitação). A extensão da limitação pode depender da camada de serviço do cliente. 

A limitação não significa necessariamente que o cliente estava mal-intencionado, mas apenas que ele excedeu sua cota de serviço. Pode acontecer que o consumidor exceda sua cota frequentemente, ou que seu comportamento seja, de alguma forma, incorreto. Nesse caso, você pode ser mais enérgico e bloquear esse usuário. Normalmente, isso é feito bloqueando-se uma chave de API ou um intervalo de endereços IP.

Para obter mais informações, consulte [Padrão de limitação][throttling-pattern].

### <a name="use-a-circuit-breaker"></a>Uso do disjuntor
O uso do padrão de Disjuntor pode impedir que um aplicativo tente várias vezes repetir uma operação que provavelmente falhará. O princípio é semelhante ao do disjuntor físico, ou seja, um comutador que interrompe o fluxo de corrente quando um circuito está sobrecarregado.

O disjuntor encapsula chamadas para um serviço. Ele tem três estados:

* **Fechado**. Este é o estado normal. O disjuntor envia solicitações ao serviço e um contador acompanha o número de falhas recentes. Se essa contagem exceder um limite em determinado período, o disjuntor alterna para o estado Aberto. 
* **Aberto**. Nesse estado, o disjuntor cancela imediatamente todas as solicitações, sem chamar o serviço. O aplicativo deve usar um caminho de mitigação, como ler dados em uma réplica ou, simplesmente, retornar um erro para o usuário. Quando o disjuntor muda para o estado Aberto, aciona um temporizador. Quando esse temporizador expira, o disjuntor alterna para o estado Entreaberto.
* **Entreaberto**. Nesse estado, o disjuntor permite a transmissão de um número limitado de solicitações ao serviço. Se elas forem bem-sucedidas, o serviço deverá ser recuperado e o disjuntor mudará para o estado Fechado. Se isso não ocorrer, ele voltará ao estado Aberto. O estado Entreaberto impede que um serviço de recuperação receba uma sobrecarga repentina de solicitações.

Para obter mais informações, consulte [Padrão de disjuntor][circuit-breaker-pattern].

### <a name="use-load-leveling-to-smooth-out-spikes-in-traffic"></a>Usar o nivelamento de carga para suavizar picos no tráfego
Os aplicativos podem enfrentar picos repentinos no tráfego e sobrecarregar o serviços de back-end. Se um serviço de back-end não conseguir responder às solicitações rápido o suficiente, pode provocar um enfileiramento de solicitações (backup) ou fazer com que o serviço limite o aplicativo.

Para evitar isso, você pode usar uma fila como buffer. Quando há um novo item de trabalho, em vez de chamar o serviço de back-end imediatamente, o aplicativo enfileira um item de trabalho para que seja executado de forma assíncrona. A fila atua como buffer e diminui os picos na carga. 

Para obter mais informações, consulte [Padrão de nivelamento de carga baseado em fila][load-leveling-pattern].

### <a name="isolate-critical-resources"></a>Isolar recursos críticos
Às vezes, as falhas em um subsistema podem ocorrer em cascata, causando falhas em outras partes do aplicativo. Isso pode acontecer se uma falha impedir que alguns recursos, como threads ou soquetes, sejam liberados em tempo hábil, exaurindo recursos. 

Para evitar isso, você pode particionar um sistema em grupos isolados, para que a falha de uma partição não paralise todo o sistema. Essa técnica também é conhecida como padrão de Bulkhead.

Exemplos:

* Particionar um banco de dados (por locatário, por exemplo) e atribuir um pool de instâncias de servidor Web a cada partição.  
* Usar pools de threads separados, para isolar as chamadas para diferentes serviços. Isso ajuda a evitar falhas em cascata, se houver falha em um dos serviços. Para ver um exemplo, consulte a [biblioteca Hystrix][hystrix] da Netflix.
* Usar [contêineres][containers] para limitar os recursos disponíveis a um determinado subsistema. 

![SLA composto](./images/bulkhead.png)

### <a name="apply-compensating-transactions"></a>Aplicar transações de compensação
Uma transação de compensação é uma operação que desfaz os efeitos de outra transação concluída.

Em um sistema distribuído, pode ser muito difícil obter uma consistência transacional sólida. As transações de compensação são uma forma de obter consistência usando uma série de transações individuais menores, que podem ser desfeitas em cada etapa.

Por exemplo, para fazer uma viagem, um cliente pode precisar reservar um voo, acomodações de hotel e um carro. Se alguma dessas etapas falhar, toda a operação falhará. Em vez de tentar usar uma única transação distribuída em toda a operação, você pode definir uma transação de compensação para cada etapa. Por exemplo, para desfazer uma reserva de carro, cancelar essa reserva. Para concluir toda a operação, um coordenador executa cada etapa. Se alguma delas falhar, o coordenador aplica transações de compensação para desfazer todas as etapas concluídas. 

Para obter mais informações, consulte [Padrão de transação de compensação][compensating-transaction-pattern]. 


## <a name="testing-for-resiliency"></a>Teste de resiliência
Em geral, não se pode testar a resiliência da mesma maneira que a funcionalidade do aplicativo (executando testes unitários e outros). Em vez disso, deve-se testar a execução da carga de trabalho de ponta a ponta sob condições de falha intermitentes.

O teste é um processo iterativo. Teste o aplicativo, avalie o resultado, analise e resolva as falhas resultantes, e repita o processo.

**Teste de injeção de falha**. Teste a resiliência do sistema durante falhas, seja disparando falhas reais ou fazendo simulações. Aqui estão alguns cenários comuns de falha para teste:

* desligamento de instâncias de VM
* falha de processos
* expiração de certificados
* mudança de chaves de acesso
* desligamento do serviço DNS em controladores de domínio
* limitação de recursos do sistema disponíveis, como RAM ou número de threads
* desmontagem de discos
* reimplantação de uma VM

Meça o tempo de recuperação e verifique se os seus requisitos de negócios estão sendo cumpridos. Teste também combinações dos modos de falha. Certifique-se de que as falhas não se propaguem em cascata e que sejam tratadas isoladamente.

Essa é outra razão pela qual é importante analisar possíveis pontos de falha durante a fase de design. Os resultados dessa análise devem funcionar como informações para o seu plano de teste.

**Teste de carga**. Faça testes de carga no aplicativo usando uma ferramenta como o [Visual Studio Team Services][vsts] ou o [Apache JMeter][jmeter]. O teste de carga é crucial para identificar falhas que só ocorrem sob condições de carga, como um banco de dados de back-end sobrecarregado ou limitações de serviço. Teste a carga de pico usando dados de produção ou sintéticos, desde que estes se aproximem o máximo possível dos dados de produção. O objetivo é observar como o aplicativo se comporta sob condições do mundo real.   

## <a name="resilient-deployment"></a>Implantação resiliente
Quando um aplicativo é implantado para produção, as atualizações são uma possível fonte de erros. Na pior das hipóteses, uma atualização corrompida pode causar inatividade. Para evitar isso, o processo de implantação deve ser repetível e previsível. A implantação inclui etapas de provisionamento de recursos do Azure, implantação do código do aplicativo e aplicação de definições de configuração. Uma atualização pode envolver todas ou parte dessas três. 

O ponto fundamental é que implantações manuais são propensas a erro. Por isso, é recomendável um processo idempotente automatizado, que você pode executar sob demanda e, caso haja alguma falha, executar novamente. 

* Use modelos do Resource Manager para automatizar o provisionamento de recursos do Azure.
* Use o [Azure Automation Desired State Configuration][dsc] (DSC) para configurar VMs.
* Use um processo de implantação automática para o código do aplicativo.

Dois conceitos relacionados à implantação resiliente são: *infraestrutura como código* e *infraestrutura imutável*.

* **Infraestrutura como código** é a prática de usar código para provisionar e configurar a infraestrutura. A infraestrutura como código pode usar abordagem declarativa ou imperativa (ou mesmo uma combinação de ambas). Os modelos do Resource Manager são um exemplo de abordagem declarativa. Os scripts do PowerShell são um exemplo de abordagem imperativa.
* **Infraestrutura imutável** é o princípio segundo o qual não se deve modificar a infraestrutura após sua implantação para produção. Caso contrário, você pode entrar em um estado em que foram aplicadas alterações ad hoc, no qual é difícil saber exatamente o que mudou e chegar a uma conclusão em relação ao sistema. 

Outra questão é como lançar uma atualização do aplicativo. Recomendamos técnicas como as chamadas implantações “blue-green” e versões “canário”, que enviam atualizações de forma altamente controlada, visando a minimizar os possíveis impactos de uma implantação incorreta.

* A [Implantação “blue-green”][blue-green] é uma técnica em que a atualização é implantada em um ambiente de produção à parte do aplicativo ao vivo. Após validar a implantação, alterne o roteamento de tráfego para a versão atualizada. Por exemplo, o recurso Aplicativos Web do Serviço de Aplicativo do Azure permite isso com slots de preparo.
* As [versões “canário”][canary-release] são semelhantes às implantações “blue-green”. Em vez de alternar todo o tráfego para a versão atualizada, lança-se a atualização para uma pequena porcentagem de usuários encaminhando uma parte do tráfego para a nova implantação. Se houver problemas, interrompe-se o processo volta-se à implantação anterior. Se tudo correr bem, encaminha-se mais uma parte do tráfego para a nova versão, até atingir 100%.

Qualquer que seja a abordagem, certifique-se de poder reverter para a última implantação íntegra, caso a nova versão não funcione. Além disso, se houver erros, os logs de aplicativo devem indicar a versão causadora do erro. 

## <a name="monitoring-and-diagnostics"></a>Monitoramento e diagnóstico
O monitoramento e o diagnóstico são cruciais para garantir a resiliência. Se algo falhar, você precisa saber o que falhou, e também ter informações sobre a causa da falha. 

O monitoramento de um sistema distribuído em larga escala representa um desafio significativo. Pense em um aplicativo executado em algumas dezenas de VMs; &mdash;não é prático fazer logon em cada uma delas e examinar seus arquivos de log para tentar solucionar um problema. Além disso, o número de instâncias de VM provavelmente não é estático. As VMs são adicionadas e removidas com a expansão e retração do aplicativo, e, ocasionalmente, uma instância pode falhar e precisar ser provisionada novamente. Além disso, um aplicativo de nuvem típico pode usar vários armazenamentos de dados (armazenamento do Azure, Banco de Dados SQL, Cosmos DB, cache Redis) e uma única ação do usuário pode abranger vários subsistemas. 

Pode-se considerar o processo de monitoramento e diagnóstico como um pipeline com várias etapas distintas:

![SLA composto](./images/monitoring.png)

* **Instrumentação**. Os dados brutos para monitoramento e diagnóstico vêm de várias fontes, inclusive logs de aplicativo e de servidor Web, contadores de desempenho do sistema operacional, logs de banco de dados e diagnósticos incorporados à plataforma Azure. A maioria dos serviços do Azure tem um recurso de diagnóstico que pode ser usado para determinar a causa dos problemas.
* **Coleta e armazenamento**. Dados de instrumentação brutos podem ser mantidos em vários locais e com vários formatos (por exemplo, logs de rastreamento de aplicativo e do IIS, contadores de desempenho). Essas fontes diferentes são coletadas, consolidadas e colocadas em um armazenamento confiável.
* **Análise e diagnóstico**. Após os dados serem consolidados, eles podem ser analisados para solucionar problemas e fornecer uma visão geral da integridade do aplicativo.
* **Visualização e alertas**. Neste estágio, os dados de telemetria são apresentados de forma que um operador pode notar rapidamente problemas ou tendências. Exemplo: inclusão de painéis ou alertas por email.  

Monitoramento não é o mesmo que detecção de falha. Por exemplo, seu aplicativo pode detectar um erro transitório e tentar novamente, sem que haja qualquer inatividade. Mas ele deve registrar essa operação de repetição, para que se possa monitorar a taxa de erros e ter uma visão geral da integridade do aplicativo. 

Os logs de aplicativo são uma fonte importante de dados de diagnóstico. Algumas práticas recomendadas para o log de aplicativo são:

* Obtenha logs na fase de produção. Caso contrário, você perderá informações onde mais precisa.
* Obtenha logs de eventos nos limites dos serviços. Inclua uma ID de correlação que flua nos limites dos serviços. Se uma transação fluir em vários serviços e houver uma falha, a ID de correlação ajudará a identificar o motivo da falha da transação.
* Use o log semântico, também conhecido como registro em log estruturado. Os logs não estruturados dificultam a automatização do consumo e da análise dos dados do log, necessários em escala de nuvem.
* Use logs assíncronos. Caso contrário, o próprio sistema de registro em log pode provocar falha do aplicativo com solicitações de backup, pois ocorre bloqueio durante a espera para gravar um evento de log.
* Log de aplicativo não é o mesmo que auditoria. A auditoria pode ser feita por motivos regulatórios ou de conformidade. Sendo assim, os registros de auditoria devem ser concluídos, não sendo aceitável remover nenhum deles durante o processamento de transações. Se um aplicativo exigir auditoria, ela deve ser mantida à parte do log de diagnóstico. 

Para obter mais informações sobre monitoramento e diagnóstico, consulte [Diretrizes de monitoramento e diagnóstico][monitoring-guidance].

## <a name="manual-failure-responses"></a>Respostas de falha manual
As seções anteriores se concentraram em estratégias de recuperação automatizada, que são essenciais para a alta disponibilidade. No entanto, às vezes, a intervenção manual, é necessária.

* **Alertas**. Monitore seu aplicativo, observando sinais de aviso que podem exigir intervenção proativa. Por exemplo, se você vir que o Banco de Dados SQL ou o Cosmos DB limitam repetidamente o seu aplicativo, talvez seja preciso aumentar a capacidade do banco de dados ou otimizar suas consultas. Neste exemplo, mesmo que o aplicativo consiga manipular esses erros de limitação de forma transparente, sua telemetria deve gerar um alerta, para que você possa acompanhar.  
* **Failover manual**. Alguns sistemas não conseguem executar failover automaticamente, e ele deve ser manual. 
* **Testes de preparação operacional**. Se o seu aplicativo executar failover para uma região secundária, você deve executar um teste de preparação operacional antes do failback para a região principal. Esse teste deve verificar se a região principal está íntegra e pronta para receber o tráfego.
* **Verificação de consistência de dados**. Se houver falha em um armazenamento de dados, pode haver inconsistências de dados quando o armazenamento voltar a ser disponibilizado, especialmente se os dados tiverem sido replicados. 
* **Restauração com base em backup**. Por exemplo, se o Banco de Dados SQL sofrer uma interrupção regional, você pode fazer uma restauração geográfica do banco de dados com base no backup mais recente.

Documente e teste seu plano de recuperação de desastres. Avalie o impacto das falhas de aplicativos sobre os negócios. Automatize o processo tanto quanto possível, e documente todas as etapas manuais, como failover manual ou restauração de dados com base em backups. Teste regularmente seu processo de recuperação de desastres, para validar e aprimorar o plano. 

## <a name="summary"></a>Resumo
Este artigo discutiu a resiliência sob uma perspectiva holística, enfatizando alguns dos desafios exclusivos da nuvem. Eles incluem a natureza distribuída de computação em nuvem, o uso de hardware de mercadoria e a presença de falhas de rede temporárias.

Estes são os principais pontos a serem lembrados neste artigo:

* A resiliência leva à maior disponibilidade e diminui o tempo médio de recuperação de falhas. 
* A resiliência na nuvem requer um conjunto de técnicas diferentes das soluções tradicionais locais. 
* A resiliência não é acidental. Ela deve ser projetada e construída desde o início.
* Resiliência é algo que deve existir em todas as partes do ciclo de vida do aplicativo, do planejamento e da codificação às operações.
* Teste e monitore!


<!-- links -->

[blue-green]: http://martinfowler.com/bliki/BlueGreenDeployment.html
[canary-release]: http://martinfowler.com/bliki/CanaryRelease.html
[circuit-breaker-pattern]: https://msdn.microsoft.com/library/dn589784.aspx
[compensating-transaction-pattern]: https://msdn.microsoft.com/library/dn589804.aspx
[containers]: https://en.wikipedia.org/wiki/Operating-system-level_virtualization
[dsc]: /azure/automation/automation-dsc-overview
[contingency-planning-guide]: http://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-34r1.pdf
[fma]: failure-mode-analysis.md
[hystrix]: http://techblog.netflix.com/2012/11/hystrix.html
[jmeter]: http://jmeter.apache.org/
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
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[vsts]: https://www.visualstudio.com/features/vso-cloud-load-testing-vs.aspx
