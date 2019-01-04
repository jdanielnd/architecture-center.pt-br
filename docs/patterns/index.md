---
title: Padrões de design na nuvem
titleSuffix: Azure Architecture Center
description: Padrões de design na nuvem para a criação de aplicativos confiáveis, escalonáveis e seguros na nuvem
keywords: Azure
author: dragon119
ms.date: 12/10/2018
ms.custom: seodec18
ms.openlocfilehash: 003bef866b0cd873122cfb8d4730b95ba49d3d7f
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011286"
---
# <a name="cloud-design-patterns"></a>Padrões de design na nuvem

Esses padrões de design são úteis para a criação de aplicativos confiáveis, dimensionáveis e seguros na nuvem.

Cada padrão descreve o problema ao qual o padrão se destina, as considerações para a aplicação do padrão e um exemplo com base no Microsoft Azure. A maioria dos padrões inclui exemplos de código ou snippets de código que mostram como implementar o padrão no Azure. No entanto, a maioria dos padrões é relevante para qualquer sistema distribuído, se hospedados no Azure ou em outras plataformas de nuvem.

## <a name="challenges-in-cloud-development"></a>Desafios de desenvolvimento em nuvem

<!-- markdownlint-disable MD033 -->
<table>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/availability.md"><img src="_images/category/availability.svg" alt="Availability" /></a></td>
    <td>
        <h3><a href="./category/availability.md">Disponibilidade</a></h3>
        <p>A disponibilidade é a proporção de tempo que o sistema está funcionando, geralmente medido como uma porcentagem do tempo de atividade. Ele pode ser afetado por erros de sistema, problemas de infraestrutura, ataques maliciosos e carga do sistema.  Os aplicativos de nuvem normalmente fornecem aos usuários um contrato de nível de serviço (SLA) e, portanto, os aplicativos devem ser criados para maximizar a disponibilidade.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/data-management.md"><img src="_images/category/data-management.svg" alt="Data Management" /></a></td>
    <td>
        <h3><a href="./category/data-management.md">Gerenciamento de dados</a></h3>
        <p>O gerenciamento de dados é o elemento principal de aplicativos em nuvem e influencia a maioria dos atributos de qualidade. Os dados normalmente são hospedados em diferentes locais e em vários servidores por motivos como desempenho, escalabilidade ou a disponibilidade, e isso pode apresentar uma série de desafios. Por exemplo, deve ser mantida a consistência dos dados e dados normalmente precisam ser sincronizados em diferentes locais.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/design-implementation.md"><img src="_images/category/design-implementation.svg" alt="Design and Implementation" /></a></td>
    <td>
        <h3><a href="./category/design-implementation.md">Design e implementação</a></h3>
        <p>Um bom design abrange fatores como a consistência e a coerência no design do componente e implantação, facilidade de manutenção para simplificar a administração e desenvolvimento e capacidade de reutilização para permitir que componentes e subsistemas para ser usado em outros aplicativos e em outros cenários. As decisões tomadas durante a fase de design e implementação tem um grande impacto sobre a qualidade e o custo total de propriedade de aplicativos e serviços hospedados pela nuvem.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/messaging.md"><img src="_images/category/messaging.svg" alt="Messaging" /></a></td>
    <td>
        <h3><a href="./category/messaging.md">Mensagens</a></h3>
        <p>A natureza distribuída dos aplicativos de nuvem exige uma infraestrutura de mensagens que conecta os componentes e serviços, idealmente de uma maneira flexível para maximizar a escalabilidade. O sistema de mensagens assíncronas é amplamente usado e fornece muitos benefícios, mas também traz desafios, como a ordenação de mensagens, o gerenciamento de mensagens suspeitas, a idempotência e muito mais</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/management-monitoring.md"><img src="_images/category/management-monitoring.svg" alt="Management and Monitoring" /></a></td>
    <td>
        <h3><a href="./category/management-monitoring.md">Gerenciamento e Monitoramento</a></h3>
        <p>Os aplicativos de nuvem executados em um data center remoto em que você não tem controle total da infraestrutura ou, em alguns casos, o sistema operacional. Isso pode tornar o gerenciamento e o monitoramento mais difíceis do que uma implantação local. Os aplicativos devem expor informações de tempo de execução que operadores e administradores podem usar para gerenciar e monitorar o sistema, bem como suporte para alteração requisitos de negócios e personalização sem a necessidade do aplicativo a ser interrompido ou reimplantado.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/performance-scalability.md"><img src="_images/category/performance-scalability.svg" alt="Performance and Scalability" /></a></td>
    <td>
        <h3><a href="./category/performance-scalability.md">Desempenho e Escalabilidade</a></h3>
        <p>O desempenho é uma indicação de que a capacidade de resposta de um sistema para executar qualquer ação dentro de um determinado intervalo, enquanto a escalabilidade é a capacidade de um sistema para lidar com o aumento de carga sem impacto no desempenho ou para os recursos disponíveis ser prontamente aumentado. Aplicativos de nuvem normalmente encontram cargas de trabalho variável e picos na atividade. Prever esses, especialmente em um cenário de multilocatário, é quase impossível. Em vez disso, os aplicativos devem ser capazes de expansão dentro dos limites para atender aos picos de demanda e a escala em quando a demanda diminui. Os problemas de escalabilidade de computação não apenas instâncias, mas outros elementos, como o armazenamento de dados, infraestrutura de mensagens e muito mais.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/resiliency.md"><img src="_images/category/resiliency.svg" alt="Resiliency" /></a></td>
    <td>
        <h3><a href="./category/resiliency.md">Resiliência</a></h3>
        <p>A resiliência é a capacidade de um sistema para tratar normalmente e recuperar de falhas. A natureza da hospedagem em nuvem, em que os aplicativos costumam ser multilocatário, usam serviços de plataforma compartilhada, competem por recursos e largura de banda, comunicam-se pela Internet e são executados no hardware de mercadoria significa que há uma maior probabilidade de que ocorram falhas tanto transitórias quanto mais permanentes. A detecção de falhas e a recuperação rápida e eficaz são necessárias para manter a resiliência.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/security.md"><img src="_images/category/security.svg" alt="Security" /></a></td>
    <td>
        <h3><a href="./category/security.md">Segurança</a></h3>
        <p>A segurança é a capacidade de um sistema impedir ações acidentais ou mal-intencionadas fora do uso projetado e evitar a divulgação ou a perda de informações. Os aplicativos de nuvem são expostos na Internet fora dos limites de locais confiáveis e geralmente são abertos ao público e podem atender a usuários não confiáveis. Os aplicativos devem ser criados e implantados de maneira que protege contra ataques mal-intencionados, restringe o acesso a somente usuários aprovados e protege dados confidenciais.</p>
    </td>
</tr>
</table>
<!-- markdownlint-disable MD033 -->

## <a name="catalog-of-patterns"></a>Catálogo de padrões

|                                Padrão                                |                                                                                                         Resumo                                                                                                         |
|-----------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                     [Embaixador](./ambassador.md)                     |                                                            Crie serviços auxiliares que enviam solicitações de rede em nome de um consumidor de serviço ou aplicativo.                                                            |
|          [Camada anticorrupção](./anti-corruption-layer.md)          |                                                                  Implemente uma camada de fachada ou adaptador entre um aplicativo moderno e um sistema herdado.                                                                  |
|         [Back-ends para Front-ends](./backends-for-frontends.md)         |                                                            Crie serviços de back-end separados a serem consumidos por aplicativos de front-end específico ou interfaces.                                                             |
|                       [Bulkhead](./bulkhead.md)                       |                                                        Isole os elementos de um aplicativo em pools para que, se um falhar, os outros continuarão a funcionar.                                                        |
|                    [Cache-Aside](./cache-aside.md)                    |                                                                                   Carregar dados sob demanda em um cache de um armazenamento de dados                                                                                    |
|                [Disjuntor](./circuit-breaker.md)                |                                                     Trate as falhas que possam consumir uma quantidade variável de tempo em sua correção ao se conectar a um serviço ou recurso remoto.                                                     |
|                           [CQRS](./cqrs.md)                           |                                                           Separar as operações que leem dados de operações que atualizam dados usando interfaces separadas.                                                            |
|       [Transação de Compensação](./compensating-transaction.md)       |                                                         Desfaça o trabalho executado por uma série de etapas, que juntas definem uma operação finalmente consistente.                                                         |
|            [Consumidores Concorrentes](./competing-consumers.md)            |                                                            Habilite vários consumidores simultâneos processar as mensagens recebidas no mesmo canal de mensagens.                                                             |
| [Consolidação de Recursos de Computação](./compute-resource-consolidation.md) |                                                                        Consolidar várias tarefas ou operações em uma única unidade de computação                                                                        |
|                 [Fornecimento de Evento](./event-sourcing.md)                 |                                                      Use um repositório somente de acréscimo para registrar a série inteira de eventos que descrevem as ações realizadas nos dados em um domínio.                                                      |
|   [Armazenamento de Configuração Externa](./external-configuration-store.md)   |                                                           Mova as informações de configuração para fora do pacote de implantação de aplicativo para um local centralizado.                                                           |
|             [Identidade Federada](./federated-identity.md)             |                                                                                Delegar autenticação a um provedor de identidade externa.                                                                                |
|                     [Gatekeeper](./gatekeeper.md)                     | Proteger aplicativos e serviços usando uma instância de host dedicado que atua como intermediário entre clientes e o aplicativo ou serviço, valida e corrige solicitações e passa solicitações e dados entre eles. |
|            [Agregação de Gateway](./gateway-aggregation.md)            |                                                                     Use um gateway para agregar várias solicitações individuais em uma única solicitação.                                                                      |
|             [Descarregamento de Gateway](./gateway-offloading.md)             |                                                                         Descarregue a funcionalidade de serviço especializado ou compartilhado para um proxy do gateway.                                                                         |
|                [Roteamento de Gateway](./gateway-routing.md)                |                                                                              Faça o roteamento de solicitações para vários serviços usando um único ponto de extremidade.                                                                               |
|     [Monitoramento do ponto de extremidade de integridade](./health-endpoint-monitoring.md)     |                                              Implemente verificações funcionais dentro de um aplicativo que ferramentas externas podem acessar por meio de pontos de extremidade expostos em intervalos regulares.                                               |
|                    [Tabela de Índice](./index-table.md)                    |                                                                Crie índices nos campos em armazenamentos de dados que são frequentemente referenciados por consultas.                                                                 |
|                [Eleição de Líder](./leader-election.md)                |   Coordene as ações executadas por uma coleção de instâncias de tarefa de colaboração em um aplicativo distribuído ao escolher uma instância como o líder que assume a responsabilidade de gerenciamento de outras instâncias.    |
|              [Exibição Materializada](./materialized-view.md)              |                                        Gere exibições pré-preenchidas nos dados em um ou mais armazenamentos de dados quando os dados não estiverem formatados como o ideal para as operações de consulta necessárias.                                        |
|              [Pipes e Filtros](./pipes-and-filters.md)              |                                                        Dividir uma tarefa que executa processamento complexo em uma série de elementos separados que podem ser reutilizados.                                                        |
|                 [Fila de Prioridade](./priority-queue.md)                 |                                 Priorize as solicitações enviadas a serviços para que as solicitações com uma prioridade mais alta sejam recebidas e processadas mais rapidamente do que aquelas com uma prioridade mais baixa.                                  |
| [Publicador/Assinante](./publisher-subscriber.md) | Permite a um aplicativo anunciar eventos para vários consumidores de seu interesse assincronamente, sem acoplar os remetentes aos destinatários. |
|      [Nivelamento de Carga Baseado em Fila](./queue-based-load-leveling.md)      |                                               Use uma fila que funcione como um buffer entre uma tarefa e um serviço que ela invoca para simplificar cargas pesadas intermitentes.                                               |
|                          [Tentar Novamente](./retry.md)                          |               Permita que um aplicativo trate falhas previstas e temporárias quando tentar se conectar a um serviço ou recurso de rede ao repetir de forma transparente uma operação que falhou anteriormente.                |
|     [Supervisor de Agente do Agendador](./scheduler-agent-supervisor.md)     |                                                              Coordene um conjunto de ações em um conjunto distribuído de serviços e outros recursos remotos.                                                               |
|                       [Fragmentação](./sharding.md)                       |                                                                           Divida um armazenamento de dados em um conjunto de partições horizontais ou fragmentos.                                                                            |
|                        [Sidecar](./sidecar.md)                        |                                                    Implante os componentes de um aplicativo em um processo ou contêiner separado para fornecer isolamento e encapsulamento.                                                     |
|         [Hospedagem de Conteúdo Estático](./static-content-hosting.md)         |                                                          Implante o conteúdo estático para um serviço de armazenamento baseado em nuvem que pode enviá-las diretamente para o cliente.                                                           |
|                      [Strangler](./strangler.md)                      |                                            Migre incrementalmente um sistema herdado substituindo gradualmente partes específicas de funcionalidade por serviços e aplicativos novos.                                            |
|                     [Limitação](./throttling.md)                     |                                                 Controle o consumo de recursos usados por uma instância de um aplicativo, um locatário individual ou todo o serviço.                                                 |
|                      [Valet Key](./valet-key.md)                      |                                                        Use um token ou chave que fornece aos clientes acesso direto e restrito a um determinado recurso ou serviço.                                                        |
