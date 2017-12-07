---
title: Fornecimento do evento
description: "Use um repositório somente de acréscimo para registrar a série inteira de eventos que descrevem as ações realizadas nos dados em um domínio."
keywords: "padrão de design"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: d5d4e99a6ff49cb823f592c83590471c0d68bfd1
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="event-sourcing-pattern"></a>Padrão de fornecimento do evento

[!INCLUDE [header](../_includes/header.md)]

Em vez de armazenar apenas o estado atual dos dados em um domínio, use um armazenamento somente de acréscimo para registrar a série completa de ações executadas nesses dados.
O repositório atua como o sistema de registro e pode ser usado para materializar os objetos de domínio. Isso pode simplificar tarefas em domínios complexos, evitando a necessidade de sincronizar o modelo de dados e o domínio da empresa, melhorando o desempenho, a escalabilidade e a capacidade de resposta. Isso também pode fornecer a consistência de dados transacionais e manter histórico e logs de auditoria completos, que podem permitir ações de compensação.

## <a name="context-and-problem"></a>Contexto e problema

A maioria dos aplicativos trabalham usando dados e a abordagem típica é o aplicativo manter o estado atual dos dados atualizando-os conforme os usuários trabalham com eles. Por exemplo, no modelo CRUD (criar, ler, atualizar e excluir) tradicional, um processo de dados típico é ler dados do repositório, fazer algumas modificações neles e atualizar o estado atual dos dados com os novos valores&mdash;geralmente usando transações que bloqueiam os dados.

A abordagem CRUD tem algumas limitações:

- Sistemas CRUD executam operações de atualização diretamente em um armazenamento de dados, o que pode diminuir o desempenho e capacidade de resposta e limitar a escalabilidade, devido à sobrecarga de processamento que isso requer.

- Em um domínio de colaboração com muitos usuários simultâneos, conflitos de atualização de dados são mais prováveis porque as operações de atualização ocorrem em um único item de dados.

- A menos que haja um mecanismo de auditoria adicional que registre os detalhes de cada operação em um log separado, o histórico será perdido.

> Para obter uma compreensão mais profunda dos limites da abordagem CRUD, consulte [CRUD, apenas quando viável](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/).

## <a name="solution"></a>Solução

O padrão de fornecimento do evento define uma abordagem para lidar com operações em dados que são controladas por uma sequência de eventos, cada um dos quais é registrado em um repositório somente de acréscimo. O código do aplicativo envia uma série de eventos que descrevem imperativamente cada ação ocorrida nos dados para o repositório de eventos, no qual eles são persistidos. Cada evento representa um conjunto de alterações nos dados (assim como `AddedItemToOrder`).

Os eventos são persistidos em um repositório de eventos que atua como o sistema de registro (fonte de dados autoritativa) sobre o estado atual dos dados. O repositório de eventos geralmente publica esses eventos de modo que os consumidores possam ser notificados e os manipulem se necessário. Os consumidores podem, por exemplo, iniciar tarefas que aplicam as operações nos eventos a outros sistemas ou então executar qualquer ação associada necessária para a conclusão da operação. Observe que o código do aplicativo que gera os eventos é separado dos sistemas que assinam os eventos.

Usos típicos dos eventos publicados pelo repositório de eventos são manter exibições materializadas de entidades conforme elas são alteradas por ações no aplicativo, bem como para integração com sistemas externos. Por exemplo, um sistema pode manter uma exibição materializada de todas as ordens de cliente que é usada para popular partes da interface do usuário. Conforme o aplicativo adiciona novas ordens, adiciona ou remove itens na ordem e adiciona informações de envio, os eventos que descrevem essas alterações podem ser manipulados e usados para atualizar a [exibição materializada](materialized-view.md).

Além disso, os aplicativos podem ler a qualquer momento o histórico de eventos e usá-lo para materializar o estado atual de uma entidade, reproduzindo e consumindo todos os eventos relacionados a essa entidade. Isso pode ocorrer sob demanda para materializar um objeto de domínio ao lidar com uma solicitação ou por meio de uma tarefa agendada para que o estado da entidade possa ser armazenado como uma exibição materializada para dar suporte à camada de apresentação.

A figura mostra uma visão geral do padrão, incluindo algumas das opções para usar o fluxo de eventos, como criar uma exibição materializada, integrar eventos com sistemas e aplicativos externos e reproduzir eventos para criar projeções do estado atual do entidades específicas.

![Uma visão geral e um exemplo do padrão de fornecimento do evento](./_images/event-sourcing-overview.png)


O padrão de fornecimento do evento fornece as seguintes vantagens:

Eventos são imutáveis e podem ser armazenados usando uma operação somente de acréscimo. A interface do usuário, um fluxo de trabalho ou um processo que iniciou um evento pode continuar e tarefas que manipulam os eventos podem ser executadas em segundo plano. Isso, combinado com o fato de que não há nenhuma contenção durante o processamento de transações, pode melhorar muito o desempenho e a escalabilidade de aplicativos, especialmente para a interface do usuário ou o nível de apresentação.

Os eventos são objetos simples que descrevem uma ação que ocorreu, juntamente com quaisquer dados associados necessários para descrever a ação representada pelo evento. Eventos não atualizam um armazenamento de dados diretamente. Eles simplesmente são registrados para manipulação no momento apropriado. Isso pode simplificar a implementação e o gerenciamento.

Eventos geralmente têm significado para um especialista de domínio, enquanto a [incompatibilidade de impedância relacional de objeto](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch) pode tornar as tabelas de banco de dados complexas difíceis de entender. As tabelas são construções artificiais que representam o estado atual do sistema, não os eventos que ocorreram.

O fornecimento do evento pode ajudar a evitar que atualizações simultâneas causem conflitos porque ele evita a necessidade de atualizar diretamente os objetos no armazenamento de dados. No entanto, o modelo de domínio ainda deve ser criado para se proteger de solicitações que podem resultar em um estado inconsistente.

O armazenamento de eventos somente de acréscimo oferece um log de auditoria que pode ser usado para monitorar as ações executadas em um armazenamento de dados, gerar o estado atual como exibições materializadas ou projeções reproduzindo eventos a qualquer momento e ajudar a testar e a depurar o sistema. Além disso, o requisito para usar os eventos de compensação para cancelar as alterações fornece um histórico das alterações que foram revertidas, o que não ocorreria se o modelo simplesmente armazenasse o estado atual. A lista de eventos também pode ser usada para analisar o desempenho do aplicativo e detectar tendências de comportamento do usuário ou então para obter outras informações de negócios úteis.

O repositório de eventos aciona eventos e tarefas executam operações em resposta a esses eventos. Desassociar as tarefas dos eventos desse modo fornece flexibilidade e extensibilidade. As tarefas sabem sobre o tipo de evento e os dados de evento, mas não sobre a operação que disparou o evento. Além disso, múltiplas tarefas podem manipular cada evento. Isso permite uma integração fácil com outros serviços e sistemas que escutam somente novos eventos acionados pelo repositório de eventos. No entanto, os eventos de fornecimento do evento tendem a ser de nível muito baixo e pode ser necessário gerar eventos de integração específicos em vez deles.

> O fornecimento do evento é geralmente combinado com o padrão CQRS executando-se as tarefas de gerenciamento de dados em resposta aos eventos e materializando exibições dos eventos armazenados.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

O sistema só será eventualmente consistente durante a criação de exibições materializadas ou a geração de projeções de dados por meio da reprodução de eventos. Há algum atraso entre um aplicativo adicionando eventos para o repositório de eventos como resultado da manipulação de uma solicitação, da publicação de eventos e da manipulação dos eventos pelos respectivos consumidores. Durante esse período, novos eventos que descrevem ainda mais as alterações nas entidades podem ter chegado ao repositório de eventos.

> [!NOTE]
> Veja o [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) para obter informações sobre consistência eventual.

O repositório de eventos é a origem permanente das informações e, desse modo, os dados de evento nunca devem ser atualizados. A única maneira de atualizar uma entidade para desfazer uma alteração é adicionar um evento de compensação ao repositório de eventos. Se o formato (em vez dos dados) dos eventos persistentes precisarem ser alterados, talvez durante uma migração, poderá ser difícil combinar eventos existentes no repositório com a nova versão. Talvez seja necessário iterar por todos os eventos fazendo alterações de modo que eles fiquem em conformidade com o novo formato ou então adicionar novos eventos que usam o novo formato. Considere usar um carimbo de versão em cada versão do esquema do evento para manter o formato de evento antigo e o novo.

Aplicativos com multithread e várias instâncias de aplicativos podem estar armazenando eventos no repositório de eventos. A consistência dos eventos no repositório de eventos é essencial, assim como a ordem de eventos que afetam uma entidade específica (a ordem em que ocorrem alterações a uma entidade afeta seu estado atual). Adicionar um carimbo de data/hora a cada evento pode ajudar a evitar problemas. Outra prática comum é anotar cada evento resultante de uma solicitação com um identificador incremental. Se duas ações tentam adicionar eventos à mesma entidade simultaneamente, o repositório de eventos pode rejeitar um evento que corresponde um identificador de entidade e um identificador de evento existentes.

Não há nenhuma abordagem padrão, tampouco mecanismos existentes como consultas SQL, para ler os eventos a fim de obter informações. Os únicos dados que podem ser extraídos são um fluxo de eventos usando um identificador de evento como o critério. Normalmente, a ID do evento é mapeada para entidades individuais. O estado atual de uma entidade pode ser determinado somente reproduzindo-se todos os eventos relacionados a ela mediante o estado original da entidade.

O tamanho de cada fluxo de eventos afeta o gerenciamento e a atualização do sistema. Se os fluxos forem grandes, considere a criação de instantâneos em intervalos específicos como um número de eventos especificado. O estado atual da entidade pode ser obtido do instantâneo e repetindo-se todos os eventos que ocorreram depois desse ponto no tempo. Para obter mais informações sobre a criação de instantâneos de dados, consulte [Instantâneo no site de arquitetura de aplicativo empresarial de Martin Fowler](http://martinfowler.com/eaaDev/Snapshot.html) e [Replicação de instantâneo mestre-subordinado](https://msdn.microsoft.com/library/ff650012.aspx).

Embora o fornecimento do evento minimize a oportunidade de atualizações conflitantes para os dados, o aplicativo ainda deve ser capaz de lidar com inconsistências que resultam de consistência eventual e da falta de transações. Por exemplo, um evento que indica uma redução no inventário de estoque pode chegar no armazenamento de dados enquanto uma ordem para esse item está sendo efetuada, resultando em um requisito para reconciliar as duas operações, seja informando o cliente ou criando uma ordem pendente.

A publicação do evento pode ser "pelo menos uma vez", assim os consumidores de eventos devem ser idempotentes. Eles não devem aplicar a atualização descrita em um evento se o evento é manipulado mais de uma vez. Por exemplo, se várias instâncias de um consumidor mantêm e agregam uma propriedade da entidade, tal como o número total de ordens efetuadas, somente uma deve ter êxito em incrementar a agregação quando ocorre um evento de ordem efetuada. Embora isso não seja uma característica-chave do fornecimento do evento, é a decisão de implementação mais comum.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use esse padrão nos cenários a seguir:

- Quando você deseja capturar a intenção, o objetivo ou o motivo nos dados. Por exemplo, as alterações para uma entidade cliente poderão ser capturadas como uma série de tipos de evento específicos como _Mudança de residência_, _Conta fechada_ ou _Falecimento_.

- Quando é minimizar ou evitar completamente a ocorrência de atualizações conflitantes aos dados.

- Quando você deseja registrar eventos que ocorrem e poder reproduzi-los para restaurar o estado de um sistema, reverter as alterações ou manter um histórico e log de auditoria. Por exemplo, quando uma tarefa envolve várias etapas você precisa executar ações para reverter as atualizações e, em seguida, repetir algumas etapas para trazer os dados de volta para um estado consistente.

- Quando usar eventos é um recurso natural da operação do aplicativo e requer pouco desenvolvimento ou esforço de implementação adicional.

- Quando você precisa desassociar o processo de inserção ou atualização de dados das tarefas necessárias para aplicar essas ações. Isso pode ser para melhorar o desempenho de interface do usuário ou para distribuir eventos em outros ouvintes que entram em ação quando os eventos ocorrem. Por exemplo, integrar um sistema de folha de pagamento com um site de envio de despesas de modo que os eventos acionados pelo repositório de eventos em resposta a atualizações aos dados feitas no site sejam consumidos tanto pelo site quanto pelo sistema de folha de pagamento.

- Quando você quiser flexibilidade para poder alterar o formato dos modelos materializados e dados de entidade se os requisitos sofrerem alterações, ou&mdash;quando usado em conjunto com CQRS&mdash;será necessário adaptar um modelo de leitura ou os modos de exibição que expõem os dados.

- Quando usado em conjunto com CQRS e consistência eventual é aceitável enquanto um modelo de leitura é atualizado ou então quando o impacto no desempenho causado pela reidratação de entidades e dados de um fluxo de eventos é aceitável.

Esse padrão pode não ser útil nas seguintes situações:

- Domínios pequenos ou simples, sistemas que têm pouca ou nenhuma lógica de negócios ou sistemas que não são de domínio que funcionam naturalmente bem com mecanismos tradicionais de gerenciamento de dados CRUD.

- Sistemas em que a consistência e atualizações em tempo real para as exibições dos dados são necessárias.

- Sistemas em que logs de auditoria, histórico e recursos para reverter e repetir ações não são necessários.

- Sistemas em que há somente uma ocorrência muito baixa de atualizações conflitantes nos dados subjacentes. Por exemplo, os sistemas que predominantemente adicionam dados em vez de atualizá-los.

## <a name="example"></a>Exemplo

Um sistema de gerenciamento de conferência precisa acompanhar o número de reservas concluídas para uma conferência para que ele possa verificar se ainda há lugares disponíveis quando um potencial participante tenta fazer uma reserva. O sistema poderia armazenar o número total de reservas para uma conferência de pelo menos duas maneiras:

- O sistema pode armazenar as informações sobre o número total de reservas como uma entidade separada em um banco de dados que contém as informações de reserva. Conforme as reservas são feitas ou canceladas, o sistema pode incrementar ou decrementar esse número conforme apropriado. Essa abordagem é simples em teoria, mas pode causar problemas de escalabilidade se um grande número de participantes está tentando reservar lugares durante um curto período de tempo. Por exemplo, no último dia antes do encerramento do período de reservas.

- O sistema poderia armazenar informações sobre reservas e cancelamentos como eventos mantidos em um repositório de eventos. Ele poderia então calcular o número de lugares disponíveis reproduzindo esses eventos. Essa abordagem pode ser mais escalonável devido à imutabilidade dos eventos. O sistema só precisa ser capaz de ler dados do repositório de eventos ou acrescentar dados ao repositório de eventos. As informações de evento sobre reservas e cancelamentos nunca são modificadas.

O diagrama a seguir ilustra como o subsistema de reserva de lugares do sistema de gerenciamento da conferência pode ser implementado usando o fornecimento do evento.

![Usando o fornecimento do evento para capturar informações sobre reservas de lugares em um sistema de gerenciamento de conferência](./_images/event-sourcing-bounded-context.png)


A sequência de ações para reservar dois lugares é conforme descrito a seguir:

1. A interface do usuário emite um comando para reservar lugares para dois participantes. O comando é manipulado por um manipulador de comandos separado. Uma parte da lógica que é separada da interface do usuário e é responsável por gerenciar solicitações lançadas como comandos.

2. Uma agregação que contém informações sobre todas as reservas para a conferência é construída consultando os eventos que descrevem as reservas e cancelamentos. Essa agregação é chamada `SeatAvailability` e está contida em um modelo de domínio que expõe métodos para consultar e modificar os dados na agregação.

    > Algumas otimizações a considerar estão usando instantâneos (de modo que você não precisa consultar e reproduzir a lista completa de eventos para obter o estado atual da agregação) e mantendo uma cópia armazenada em cache da agregação na memória.

3. O manipulador de comandos invoca um método exposto pelo modelo de domínio para fazer as reservas.

4. A agregação `SeatAvailability` registra um evento que contém o número de lugares que foram reservados. Na próxima vez em que a agregação aplicar eventos, todas as reservas serão usadas para calcular quantos lugares restam.

5. O sistema acrescenta o novo evento à lista de eventos no repositório de eventos.

Se um usuário cancela a reserva de um lugar, o sistema segue um processo semelhante, exceto pelo fato de que o manipulador de comandos emite um comando que gera um evento de cancelamento de lugar e o anexa ao repositório de eventos.

Além de fornecer mais escopo para escalabilidade, usar um repositório de eventos também fornece um histórico completo ou log de auditoria das reservas e cancelamentos para uma conferência. Os eventos no repositório de eventos são o registro preciso. Não é necessário persistir agregações de qualquer outra forma, porque o sistema pode facilmente reproduzir os eventos e restaurar o estado para qualquer ponto no tempo.

> Você pode encontrar mais informações sobre esse exemplo em [Apresentando o fornecimento do evento](https://msdn.microsoft.com/library/jj591559.aspx).

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados

Os padrões e diretrizes a seguir também podem ser relevantes ao implementar esse padrão:

- [Padrão de CQRS (comando e segregação de responsabilidade de consulta)](cqrs.md). O repositório de gravação que fornece a origem permanente das informações de uma implementação de CQRS é normalmente baseado em uma implementação do padrão do fornecimento do evento. Descreve como separar as operações que leem dados em um aplicativo das operações que atualizam dados usando interfaces separadas.

- [Padrão de Exibição Materializada](materialized-view.md). O armazenamento de dados usado em um sistema baseado em evento de fornecimento normalmente não é adequado para consultar de modo eficiente. Em vez disso, uma abordagem comum é gerar previamente exibições dos dados em intervalos regulares ou quando os dados são alterados. Mostra como isso pode ser feito.

- [Padrão de Transação de Compensação](compensating-transaction.md). Os dados existentes em um repositório de fornecimento do evento não são atualizados, em vez disso, são adicionadas novas entradas que fazem a transição de estado das entidades para os novos valores. Para reverter uma alteração, as entradas de compensação são usadas porque não é possível simplesmente reverter a alteração anterior. Descreve como desfazer o trabalho realizado por uma operação anterior.

- [Primer de Consistência de Dados](https://msdn.microsoft.com/library/dn589800.aspx). Ao usar fornecimento do evento com um repositório de leitura separado ou exibições materializadas, os dados de leitura não serão imediatamente consistentes, mas em vez disso serão apenas eventualmente consistentes. Resume os problemas que envolvem a manutenção da consistência em dados distribuídos.

- [Diretrizes de particionamento de dados](https://msdn.microsoft.com/library/dn589795.aspx). Dados muitas vezes são particionados ao usar o fornecimento do evento para melhorar a escalabilidade, reduzir a contenção e otimizar o desempenho. Descreve como dividir dados em partições discretas e os problemas que podem surgir.

- Post de Greg Young, [Por que usar o fornecimento do evento?](http://codebetter.com/gregyoung/2010/02/20/why-use-event-sourcing/).
