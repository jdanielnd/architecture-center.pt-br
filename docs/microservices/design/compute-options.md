# <a name="choosing-a-compute-option-for-microservices"></a>Escolhendo uma opção de computação para microsserviços

O termo *computação* refere-se ao modelo de hospedagem para os recursos de computação em que seu aplicativo é executado. Para uma arquitetura de microsserviços, duas abordagens são especialmente populares:

- Um orquestrador de serviços que gerencia serviços em execução em nós dedicado (VMs).
- Uma arquitetura sem servidor que funciona como um serviço (FaaS).

Embora essas não sejam as únicas opções, são abordagens comprovadas para criação de microsserviços. Um aplicativo pode incluir ambas as abordagens.

## <a name="service-orchestrators"></a>Orquestradores de serviço

Um orquestrador manipula tarefas relacionadas à implantação e ao gerenciamento de um conjunto de serviços. Essas tarefas incluem a colocação de serviços em nós, monitoramento da integridade dos serviços, reinicialização de serviços não íntegros, balanceamento de carga de tráfego de rede em instâncias de serviço, descoberta de serviço, dimensionamento do número de instâncias de um serviço e aplicação das atualizações de configuração. Entre os orquestradores populares estão Kubernetes, DC/OS, Docker Swarm e Service Fabric.

Na plataforma do Azure, considere as seguintes opções:

- O AKS ([Serviço de Kubernetes do Azure](/azure/aks/)) é um serviço Kubernetes gerenciado. O AKS provisiona Kubernetes e expõe os pontos de extremidade da API Kubernetes, mas hospeda e gerencia o plano de controle Kubernetes, realizando atualizações automatizadas, patches automatizados, dimensionamento automático e outras tarefas de gerenciamento. Pense em AKS como "API Kubernetes como um serviço".

- [Service Fabric](/azure/service-fabric/) é uma plataforma de sistemas distribuídos para empacotamento, implantação e gerenciamento de microsserviços. Os microsserviços podem ser implantados no Service Fabric como contêineres, executáveis binários ou [Reliable Services](/azure/service-fabric/service-fabric-reliable-services-introduction). Usando o modelo de programação de Reliable Services, os serviços podem usar diretamente as APIs de programação do Service Fabric para consultar o sistema, informar sobre integridade, receber notificações sobre alterações na configuração e no código e descobrir outros serviços. Uma diferencial importante do Service Fabric é o seu forte foco na criação de serviços com estado, usando [Coleções Confiáveis](/azure/service-fabric/service-fabric-reliable-services-reliable-collections).

- Outras opções, como o Docker Enterprise Edition e o Mesosphere DC/SO podem executar em um ambiente de IaaS no Azure. Você pode encontrar modelos de implantação na [do Azure Marketplace](https://azuremarketplace.microsoft.com).

## <a name="containers"></a>Contêineres

Às vezes, as pessoas falam sobre contêineres e microsserviços como se fossem a mesma coisa. Embora isso não seja verdade &mdash; você não precisa de contêineres para criar microsserviços &mdash; os contêineres têm alguns benefícios particularmente relevantes para microsserviços, como:

- **Portabilidade**. Uma imagem de contêiner é um pacote autônomo, que é executado sem a necessidade de instalar bibliotecas ou outras dependências. Isso facilita a implantação. Contêineres podem ser iniciados e interrompidos rapidamente, portanto, você pode criar novas instâncias para lidar com mais carga ou para se recuperar de falhas de nó.

- **Densidade**. Contêineres são leves em comparação com a execução de uma máquina virtual, porque eles compartilham os recursos do sistema operacional. Isso possibilita empacotar vários contêineres em um único nó, o que é especialmente útil quando o aplicativo é composto por muitos pequenos serviços.

- **Isolamento de recurso**. Você pode limitar a quantidade de memória e a CPU disponível para um contêiner, o que ajuda a garantir que um processo sem controle não esgote os recursos do host. Confira o [padrão de Bulkhead](../../patterns/bulkhead.md) para obter mais informações.

## <a name="serverless-functions-as-a-service"></a>Sem Servidor (Funções como um serviço)

Com uma arquitetura [sem servidor](https://azure.microsoft.com/solutions/serverless/), não é possível administrar VMs nem a infraestrutura de rede virtual. Em vez disso, você implanta o código, e o serviço de hospedagem coloca esse código em uma VM e o executa. Essa abordagem tende a favorecer pequenas funções granulares que sejam coordenadas usando gatilhos baseados em eventos. Por exemplo, uma mensagem colocada em uma fila pode disparar uma função que lê da fila e processa a mensagem.

[O Azure Functions](/azure/azure-functions/) é um serviço de computação sem servidor que oferece suporte a vários gatilhos de função, incluindo solicitações HTTP, as filas do barramento de serviço e eventos de Hubs de eventos. Para obter uma lista completa, consulte [conceitos de associações e gatilhos do Azure Functions](/azure/azure-functions/functions-triggers-bindings). Também considere [grade de eventos do Azure](/azure/event-grid/), que é um serviço de roteamento de evento gerenciado no Azure.

<!-- markdownlint-disable MD026 -->

## <a name="orchestrator-or-serverless"></a>Orchestrator ou sem servidor?

<!-- markdownlint-enable MD026 -->

Aqui estão alguns fatores a considerar ao escolher entre uma abordagem de orquestrador e uma abordagem sem servidor.

**Capacidade de gerenciamento** um aplicativo sem servidor é fácil de gerenciar, porque a plataforma gerencia todos os de recursos de computação por você. Um orquestrador abstrai alguns aspectos de gerenciamento e configuração de um cluster, mas ele não oculta completamente as VMs subjacentes. Com um orquestrador, você terá de pensar sobre problemas como, balanceamento de carga, uso de CPU e de memória e rede.

**Flexibilidade e controle**. Um orquestrador fornece muito controle sobre como configurar e gerenciar seus serviços e o cluster. A desvantagem é a complexidade adicional. Com uma arquitetura sem servidor, você perde um pouco do controle porque esses detalhes são abstraídos.

**Portabilidade**. Todos os orquestradores listados aqui (Kubernetes, DC/OS, Docker Swarm e Service Fabric) podem ser executados localmente ou em várias nuvens publicas.

**Integração de aplicativos**. Pode ser um desafio criar um aplicativo complexo usando uma arquitetura sem servidor. Uma opção no Azure é usar [Aplicativos Lógicos do Azure](/azure/logic-apps/) para coordenar um conjunto do Azure Functions. Para um exemplo dessa abordagem, consulte [Criar uma função que se integre aos Aplicativos Lógicos do Azure](/azure/azure-functions/functions-twitter-email).

**Custo**. Com um orquestrador, você paga pelas VMs em execução no cluster. Com um aplicativo sem servidor, você paga apenas pelos recursos de computação real consumidos. Em ambos os casos, você precisa considerar o custo dos serviços adicionais, como armazenamento, bancos de dados e serviços de mensagens.

**Escalabilidade**. O Azure Functions é dimensionado automaticamente para atender à demanda, com base no número de eventos recebidos. Com um orquestrador, você pode expandir, aumentando o número de instâncias de serviço em execução no cluster. Você também pode dimensionar adicionando VMs ao cluster.

Nossa implementação da referência usa principalmente Kubernetes, mas nós usamos o Azure Functions para um serviço, ou seja, o serviço de histórico de entrega. O Azure Functions foi uma boa opção para esse serviço em particular, porque é uma carga de trabalho baseada em eventos. Usando um gatilho de Hubs de Eventos para chamar a função, o serviço precisou de uma quantidade mínima de código. Além disso, o serviço de Histórico de Entrega não faz parte do fluxo de trabalho principal, portanto, executá-lo fora do cluster Kubernetes não afeta a latência ponta a ponta de operações iniciadas pelo usuário.

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Comunicação entre serviços](./interservice-communication.md)
