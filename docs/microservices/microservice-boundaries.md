---
title: Identificando limites de microsserviço
description: Identificando limites de microsserviço
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: e4f11da9f970724c55ad99824f808a10c4558971
ms.sourcegitcommit: 744ad1381e01bbda6a1a7eff4b25e1a337385553
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/08/2018
---
# <a name="designing-microservices-identifying-microservice-boundaries"></a>Criando microsserviços: identificando limites de microsserviço

Qual é o tamanho correto de um microsserviço? Muitas vezes você ouve algo como, "não muito grande e não muito pequeno" &mdash; e, embora isso seja correto, não é muito útil, na prática. Mas, se você iniciar de um modelo de domínio cuidadosamente desenvolvido, será mais fácil pensar sobre microsserviços.

![](./images/bounded-contexts.png)

## <a name="from-domain-model-to-microservices"></a>Do modelo de domínio a microsserviços

No [capítulo anterior](./domain-analysis.md), definimos um conjunto de contextos limitados para o aplicativo de entrega por drone. Então, analisamos mais de perto um desses contextos limitados, o contexto limitado de Envio, e identificamos um conjunto de entidades, agregados e serviços de domínio para esse contexto limitado.

Agora estamos prontos para passar do modelo de domínio para o design do aplicativo. Aqui está uma abordagem que você pode usar para derivar microsserviços do modelo de domínio.

1. Comece com um contexto limitado. Em geral, a funcionalidade em um microsserviço não deve abranger mais de um contexto limitado. Por definição, um contexto limitado marca o limite de um modelo de domínio específico. Se você achar que um microsserviço combina diferentes modelos de domínio, é sinal de que precisa voltar e refinar sua análise de domínio.

2. Em seguida, examine os agregados no seu modelo de domínio. Agregados são, frequentemente, bons candidatos a microsserviços. Um agregado bem projetado exibe muitas das características de um microsserviço bem projetado, por exemplo:

    - Agregados são derivados de requisitos de negócios, e não de questões técnicas, como acesso a dados ou mensagens.  
    - Um agregado deve ter alta coesão funcional.
    - Um agregado é um limite de persistência.
    - Agregados devem ser acoplados de forma flexível. 
    
3. Serviços de domínio também são bons candidatos a microsserviços. Serviços de domínio são operações sem estado entre vários agregados. Um exemplo típico é um fluxo de trabalho que envolve vários microsserviços. Veremos um exemplo disso no aplicativo de entrega por drone.

4. Finalmente, considere os requisitos não funcionais. Observe fatores como tamanho da equipe, tipos de dados, tecnologias, requisitos de escalabilidade, requisitos de disponibilidade e requisitos de segurança. Esses fatores podem levá-lo a decompor ainda mais um microsserviço em dois ou mais serviços menores, ou fazer o contrário e combinar vários microsserviços em um. 

Depois de identificar os microsserviços em seu aplicativo, valide o design conforme os seguintes critérios:

- Cada serviço tem uma única responsabilidade.
- Não há nenhuma chamada de conversa entre os serviços. Se a divisão da funcionalidade em dois serviços gerar excesso de conversas, isso poderá ser um sintoma de que essas funções pertencem ao mesmo serviço.
- Cada serviço é pequeno o suficiente para ser criado por uma pequena equipe trabalhando de forma independente.
- Não há nenhuma interdependência que exige que dois ou mais serviços sejam implantados em sincronia. Deve ser sempre possível implantar um serviço sem redistribuir outros serviços.
- Os serviços não estão acoplados de forma firme e podem evoluir de forma independente.
- Os limites do serviço não criarão problemas de consistência ou integridade de dados. Às vezes, é importante manter a consistência dos dados colocando a funcionalidade em um único microsserviço. Dito isto, considere se você realmente precisa de consistência forte. Existem estratégias para abordar a eventual consistência em um sistema distribuído, e os benefícios dos serviços de decomposição geralmente superam os desafios de gerenciar a consistência eventual.

Acima de tudo, é importante ser pragmático e lembrar-se de que o design orientado por domínio é um processo iterativo. Em caso de dúvida, comece com microsserviços de granulação grosseira. É mais fácil dividir um microsserviço em dois serviços menores do que refatorar a funcionalidade em vários microsserviços existentes.
  
## <a name="drone-delivery-defining-the-microservices"></a>Entrega por drone: definindo a microsserviços

Lembre-se de que a equipe de desenvolvimento identificou as quatro agregações &mdash; Entrega, Pacote, Drone e Conta &mdash; e dois serviços de domínio, Agendador e o Supervisor. 

Entrega e Pacote são candidatos óbvios a microsserviços. O Agendador e o Supervisor coordenam as atividades executadas por outros microsserviços, então, faz sentido implementar esses serviços de domínio como microsserviços.  

Drone e Conta são interessantes porque pertencem a outros contextos limitados. Uma opção é Agendador chamar contextos limitados de Drone e Conta diretamente. Outra opção é criar microsserviços de Drone e Conta dentro do contexto de envio limitado. Esses microsserviços fariam a mediação entre os contextos delimitados, expondo as APIs ou os esquemas de dados mais adequados ao contexto de Envio.

Os detalhes dos contextos limitados de Drone e Conta estão além do escopo desta orientação, portanto, criamos serviços fictícios para eles em nossa implementação de referência. Mas, aqui estão alguns fatores a considerar nessa situação:

- Qual é a sobrecarga de rede ao chamar diretamente outro contexto limitado? 

- O esquema de dados para o outro contexto limitado é adequado para esse contexto ou é melhor ter um esquema adaptado a esse contexto limitado? 

- O outro contexto limitado é sistema herdado? Em caso afirmativo, crie um serviço que atue como [camada anticorrupção](../patterns/anti-corruption-layer.md) para converter entre o sistema herdado e o aplicativo moderno. 

- O que é a estrutura de equipe? É fácil se comunicar com a equipe responsável pelo outro contexto limitado? Se não for fácil, a criação de um serviço que faça a mediação entre os dois contextos pode ajudar a atenuar o custo da comunicação entre equipes.

Até agora, não consideramos nenhum requisito não funcional. Pensando nos requisitos de taxa de transferência do aplicativo, a equipe de desenvolvimento decidiu criar um microsserviço de Ingestão separado, responsável pela ingestão de solicitações do cliente. Esse microsserviço implementará o [nivelamento de carga](../patterns/queue-based-load-leveling.md), colocando as solicitações recebidas em um buffer para processamento. O Agendador fará a leitura das solicitações do buffer e executará o fluxo de trabalho. 

Requisitos não funcionais levaram a equipe a criar um serviço adicional. Todos os serviços até agora foram sobre o processo de agendamento e entrega de pacotes em tempo real. Mas o sistema também precisa armazenar o histórico de cada entrega no armazenamento de longo prazo para análise dos dados. A equipe considerou atribuir essa responsabilidade ao serviço de Entrega. No entanto, os requisitos de armazenamento de dados são muito diferentes para a análise histórica versus operações em andamento (consulte [Considerações sobre dados](./data-considerations.md)). Portanto, a equipe decidiu criar um serviço de Histórico de Entrega separado, que escutará eventos DeliveryTracking do serviço de Entrega e gravará os eventos no armazenamento de longo prazo.

O diagrama a seguir mostra o design neste ponto:
 
![](./images/microservices.png)

## <a name="choosing-a-compute-option"></a>Escolhendo um opção de computação

O termo *computação* refere-se ao modelo de hospedagem para os recursos de computação em que seu aplicativo é executado. Para uma arquitetura de microsserviços, duas abordagens são especialmente populares:

- Um orquestrador de serviços que gerencia serviços em execução em nós dedicado (VMs).
- Uma arquitetura sem servidor que funciona como um serviço (FaaS). 

Embora essas não sejam as únicas opções, são abordagens comprovadas para criação de microsserviços. Um aplicativo pode incluir ambas as abordagens.

### <a name="service-orchestrators"></a>Orquestradores de serviço

Um orquestrador manipula tarefas relacionadas à implantação e ao gerenciamento de um conjunto de serviços. Essas tarefas incluem a colocação de serviços em nós, monitoramento da integridade dos serviços, reinicialização de serviços não íntegros, balanceamento de carga de tráfego de rede em instâncias de serviço, descoberta de serviço, dimensionamento do número de instâncias de um serviço e aplicação das atualizações de configuração. Orchestrators populares incluem Kubernetes, DC/OS, Docker Swarm e Service Fabric. 

- [ACS](/azure/container-service/) (Serviço de Contêiner do Azure) é um serviço Azure que permite implantar um cluster Kubernetes, DC/OS ou Docker Swarm preparado para produção.

- [AKS (Serviço de Contêiner do Azure)](/azure/aks/) é um serviço Kubernetes gerenciado. O AKS provisiona Kubernetes e expõe os pontos de extremidade da API Kubernetes, mas hospeda e gerencia o plano de controle Kubernetes, realizando atualizações automatizadas, patches automatizados, dimensionamento automático e outras tarefas de gerenciamento. Pense em AKS como "API Kubernetes como um serviço". No momento deste artigo, o AKS ainda está em visualização. No entanto, espera-se que o AKS se torne a maneira preferida de executar Kubernetes no Azure. 

- [Service Fabric](/azure/service-fabric/) é uma plataforma de sistemas distribuídos para empacotamento, implantação e gerenciamento de microsserviços. Os microsserviços podem ser implantados no Service Fabric como contêineres, executáveis binários ou [Reliable Services](/azure/service-fabric/service-fabric-reliable-services-introduction). Usando o modelo de programação de Reliable Services, os serviços podem usar diretamente as APIs de programação do Service Fabric para consultar o sistema, informar sobre integridade, receber notificações sobre alterações na configuração e no código e descobrir outros serviços. Uma diferencial importante do Service Fabric é o seu forte foco na criação de serviços com estado, usando [Coleções Confiáveis](/azure/service-fabric/service-fabric-reliable-services-reliable-collections).

### <a name="containers"></a>Contêineres

Às vezes, as pessoas falam sobre contêineres e microsserviços como se fossem a mesma coisa. Embora isso não seja verdade &mdash; você não precisa de contêineres para criar microsserviços &mdash; os contêineres têm alguns benefícios particularmente relevantes para microsserviços, como:

- **Portabilidade**. Uma imagem de contêiner é um pacote autônomo, que é executado sem a necessidade de instalar bibliotecas ou outras dependências. Isso facilita a implantação. Contêineres podem ser iniciados e interrompidos rapidamente, portanto, você pode criar novas instâncias para lidar com mais carga ou para se recuperar de falhas de nó. 

- **Densidade**. Contêineres são leves em comparação com a execução de uma máquina virtual, porque eles compartilham os recursos do sistema operacional. Isso possibilita empacotar vários contêineres em um único nó, o que é especialmente útil quando o aplicativo é composto por muitos pequenos serviços.

- **Isolamento de recurso**. Você pode limitar a quantidade de memória e a CPU disponível para um contêiner, o que ajuda a garantir que um processo sem controle não esgote os recursos do host. Consulte o [padrão de Bulkhead](../patterns/bulkhead.md) para obter mais informações.

### <a name="serverless-functions-as-a-service"></a>Sem Servidor (Funções como um serviço)

Com uma arquitetura sem servidor, não é possível administrar VMs nem a infraestrutura de rede virtual. Em vez disso, você implanta o código, e o serviço de hospedagem coloca esse código em uma VM e o executa. Essa abordagem tende a favorecer pequenas funções granulares que sejam coordenadas usando gatilhos baseados em eventos. Por exemplo, uma mensagem colocada em uma fila pode disparar uma função que lê da fila e processa a mensagem.

[Azure Functions] [ functions] é um serviço de computação sem servidor, compatível com vários gatilhos de função, incluindo solicitações de HTTP, filas de barramento de serviço e eventos de Hubs de Eventos. Para obter uma lista completa, consulte [Gatilhos e conceitos de ligação do Azure Functions][functions-triggers]. Além disso, considere a [Grade de Eventos do Azure][event-grid], que é um serviço de roteamento de eventos gerenciados no Azure.

### <a name="orchestrator-or-serverless"></a>Orchestrator ou sem servidor?

Aqui estão alguns fatores a considerar ao escolher entre uma abordagem de orquestrador e uma abordagem sem servidor.

**Capacidade de gerenciamento** um aplicativo sem servidor é fácil de gerenciar, porque a plataforma gerencia todos os de recursos de computação por você. Um orquestrador abstrai alguns aspectos de gerenciamento e configuração de um cluster, mas ele não oculta completamente as VMs subjacentes. Com um orquestrador, você terá de pensar sobre problemas como, balanceamento de carga, uso de CPU e de memória e rede.

**Flexibilidade e controle**. Um orquestrador fornece muito controle sobre como configurar e gerenciar seus serviços e o cluster. A desvantagem é a complexidade adicional. Com uma arquitetura sem servidor, você perde um pouco do controle porque esses detalhes são abstraídos.

**Portabilidade**. Todos os orquestradores listados aqui (Kubernetes, DC/OS, Docker Swarm e Service Fabric) podem ser executados localmente ou em várias nuvens publicas. 

**Integração de aplicativos**. Pode ser um desafio criar um aplicativo complexo usando uma arquitetura sem servidor. Uma opção no Azure é usar [Aplicativos Lógicos do Azure](/azure/logic-apps/) para coordenar um conjunto do Azure Functions. Para um exemplo dessa abordagem, consulte [Criar uma função que se integre aos Aplicativos Lógicos do Azure](/azure/azure-functions/functions-twitter-email).

**Custo**. Com um orquestrador, você paga pelas VMs em execução no cluster. Com um aplicativo sem servidor, você paga apenas pelos recursos de computação real consumidos. Em ambos os casos, você precisa considerar o custo dos serviços adicionais, como armazenamento, bancos de dados e serviços de mensagens.

**Escalabilidade**. O Azure Functions é dimensionado automaticamente para atender à demanda, com base no número de eventos recebidos. Com um orquestrador, você pode expandir, aumentando o número de instâncias de serviço em execução no cluster. Você também pode dimensionar adicionando VMs ao cluster.

Nossa implementação da referência usa principalmente Kubernetes, mas nós usamos o Azure Functions para um serviço, ou seja, o serviço de histórico de entrega. O Azure Functions foi uma boa opção para esse serviço em particular, porque é uma carga de trabalho baseada em eventos. Usando um gatilho de Hubs de Eventos para chamar a função, o serviço precisou de uma quantidade mínima de código. Além disso, o serviço de Histórico de Entrega não faz parte do fluxo de trabalho principal, portanto, executá-lo fora do cluster Kubernetes não afeta a latência ponta a ponta de operações iniciadas pelo usuário. 

> [!div class="nextstepaction"]
> [Considerações sobre dados](./data-considerations.md)

<!-- links -->

[acs-engine]: https://github.com/Azure/acs-engine
[acs-faq]: /azure/container-service/dcos-swarm/container-service-faq
[event-grid]: /azure/event-grid/
[functions]: /azure/azure-functions/functions-overview
[functions-triggers]: /azure/azure-functions/functions-triggers-bindings
