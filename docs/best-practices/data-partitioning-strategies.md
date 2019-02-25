---
title: Estratégias de particionamento de dados
titleSuffix: Best practices for cloud applications
description: Orientação sobre como separar partições de dados para gerenciamento e acesso separados.
author: dragon119
ms.date: 11/04/2018
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 4f973a6173e882d6ae839833bd3c5bf86f8d7fb6
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898129"
---
# <a name="data-partitioning-strategies"></a>Estratégias de particionamento de dados

Este artigo descreve algumas estratégias de particionamento de dados em vários armazenamentos de dados do Azure. Para obter diretrizes gerais sobre quando particionar os dados e sobre as melhores práticas, confira [Particionamento de dados](./data-partitioning.md)

## <a name="partitioning-azure-sql-database"></a>Particionamento do Banco de Dados SQL do Azure

Um banco de dados SQL individual tem um limite para volume de dados que ele pode conter. A taxa de transferência é restrita por fatores de arquitetura e pelo número de conexões simultâneas que são permitidas.

Os [Pools elásticos](/azure/sql-database/sql-database-elastic-pool) dão suporte ao dimensionamento horizontal para um banco de dados SQL. Usando pools elásticos, você pode particionar os dados em fragmentos que são distribuídos ente vários bancos de dados SQL. Também é possível adicionar ou remover fragmentos à medida que o volume de dados com o qual você precisa lidar aumenta e diminui. Os pools elásticos também podem ajudar a reduzir a contenção pela distribuição da carga nos bancos de dados.

Cada fragmento é implementado como um banco de dados SQL. Um fragmento pode conter mais de um conjunto de dados (chamado de *shardlet*). Cada banco de dados mantém metadados que descrevem os shardlets que eles contêm. Um shardlet pode ser um único item de dados ou um grupo de itens que compartilham a mesma chave de shardlet. Por exemplo, em um aplicativo multilocatário, a chave de shardlet poderá ser a ID do locatário, e todos os dados de um locatário podem ser mantidos no mesmo shardlet.

Os aplicativos cliente são responsáveis pela associação de um conjunto de dados a uma chave de shardlet. Um banco de dados SQL separado age como um gerenciador global de mapa de fragmentos. Esse banco de dados contém uma lista de todos os fragmentos e shardlets no sistema. O aplicativo conecta-se ao banco de dados do gerenciador de mapa de fragmentos para obter uma cópia do mapa do fragmento. Ele armazena o mapa de fragmentos em cache localmente e usa o mapa para rotear as solicitações de dados ao fragmento adequado. Essa funcionalidade fica oculta atrás de uma série de APIs contidas na [Biblioteca de clientes do Banco de Dados Elástico](/azure/sql-database/sql-database-elastic-database-client-library), disponível para Java e .NET.

Para obter mais informações sobre pools elásticos, confira [Escalando horizontalmente com o Banco de Dados SQL do Azure](/azure/sql-database/sql-database-elastic-scale-introduction).

Para reduzir a latência e melhorar a disponibilidade, você pode replicar o banco de dados do gerenciador do mapa de fragmentos global. Com tipos de preço Premium, você poderá configurar a replicação geográfica ativa para copiar os dados continuamente para os bancos de dados em diferentes regiões.

Como alternativa, use a [Sincronização de Dados SQL do Azure](/azure/sql-database/sql-database-sync-data) ou o [Azure Data Factory](/azure/data-factory/) para replicar o banco de dados do gerenciador de mapa de fragmentos nas regiões. Essa forma de replicação é executada periodicamente e é mais adequada se o mapa de fragmentos não for alterado com frequência e não requerer um tipo Premium.

O Banco de Dados Elástico oferece dois esquemas para mapear dados para shardlets e armazená-los em fragmentos:

- Um **mapa de fragmentos de lista** associa uma única chave a um shardlet. Por exemplo, em um sistema multilocatário, os dados de cada locatário podem ser associados a uma chave exclusiva e armazenados em seu próprio shardlet. Para garantir o isolamento, cada shardlet pode ser mantido em seu próprio fragmento.

    ![Usando um mapa de fragmentos da lista para armazenar dados de locatário em fragmentos separados](./images/data-partitioning/PointShardlet.png)

- Um **mapa de fragmentos de intervalo** associa um conjunto contíguo de valores de chave a um shardlet. Por exemplo, você pode agrupar os dados para um conjunto de locatários (cada um com sua própria chave) dentro do mesmo shardlet. Esse esquema é mais barato que o primeiro porque os locatários compartilham o armazenamento de dados, mas apresenta menos isolamento.

    ![Usando um mapa de fragmentos do intervalo para armazenar dados de um intervalo de locatários em um fragmento](./images/data-partitioning/RangeShardlet.png)

Um único fragmento pode conter os dados de vários shardlets. Por exemplo, você pode usar os shardlets da lista para armazenar dados de diferentes locatários não contíguos no mesmo fragmento. Você também pode combinar shardlets de intervalo e de lista no mesmo fragmento, mesmo que eles sejam abordados por meio de mapas diferentes. O diagrama a seguir mostra essa abordagem:

![Implementando vários mapas de fragmentos](./images/data-partitioning/MultipleShardMaps.png)

Pools elásticos tornam possível a adição ou remoção de fragmentos conforme o volume de dados é reduzido ou aumentado. Os aplicativos cliente podem criar e excluir fragmentos dinamicamente, além de atualizar o gerenciador de mapa de fragmentos de modo transparente. No entanto, a remoção de um fragmento é uma operação destrutiva que também requer a exclusão de todos os dados nesse fragmento.

Se um aplicativo precisar dividir um fragmento em dois fragmentos separados ou combinar os fragmentos, use a [ferramenta de mesclagem/divisão](/azure/sql-database/sql-database-elastic-scale-overview-split-and-merge). Essa ferramenta é executada como um serviço Web do Azure e migra os dados entre os fragmentos com segurança.

O esquema de particionamento pode afetar significativamente o desempenho do sistema. Ele também pode afetar a taxa na qual os fragmentos devem ser adicionados ou removidos, ou esses dados devem ser reparticionados entre os fragmentos. Considere os seguintes pontos:

- Agrupe os dados que são usados juntos no mesmo fragmento e evite operações que acessem os dados a partir de vários fragmentos. Um fragmento é um banco de dados SQL em si mesmo, e as junções entre bancos de dados devem ser executadas no lado do cliente.

    Embora o Banco de Dados SQL não dê suporte a junções entre bancos de dados, você pode usar as ferramentas do Banco de Dados Elástico para realizar [consultas com vários segmentos](/azure/sql-database/sql-database-elastic-scale-multishard-querying). Uma consulta de vários fragmentos envia consultas individuais para cada banco de dados e mescla os resultados.

- Não crie um sistema que tenha dependências entre fragmentos. Restrições de integridade referencial, gatilhos e procedimentos armazenados em um banco de dados não podem fazer referência a objetos em outro.

- Se você tiver dados de referência que são usados com frequência por consultas, cogite replicar esses dados em fragmentos. Essa abordagem pode remover a necessidade de unir os dados em bancos de dados. O ideal é que esses dados sejam estáticos ou lentos para minimizar o esforço de replicação e reduzir a probabilidade de eles se tornarem obsoletos.

- Shardlets que pertencem ao mesmo mapa de fragmentos devem ter o mesmo esquema. Essa regra não é imposta pelo Banco de Dados SQL, mas o gerenciamento de dados e a consulta se tornam muito complexos se cada shardlet tiver um esquema diferente. Em vez disso, crie mapas de fragmentos separados para cada esquema. Lembre-se de que os dados pertencentes a diferentes shardlets podem ser armazenados no mesmo fragmento.

- As operações transacionais contam com suporte apenas para dados dentro de um fragmento, não entre fragmentos. As transações podem abranger shardlets, desde que façam parte do mesmo fragmento. Portanto, se a sua lógica de negócios precisar executar transações, armazene os dados no mesmo fragmento ou implemente a consistência eventual.

- Coloque os fragmentos próximos aos usuários que acessam os dados nesses fragmentos. Essa estratégia ajuda a reduzir a latência.

- Evite ter uma combinação de fragmentos altamente ativos e fragmentos relativamente inativos. Tente distribuir a carga uniformemente entre os fragmentos. Isso pode exigir o hash das chaves de fragmentação. Se estiver localizando os fragmentos geograficamente, verifique se as chaves em hash são mapeadas para os shardlets mantidos em fragmentos armazenados próximo aos usuários que acessam esses dados.

### <a name="partitioning-azure-table-storage"></a>Particionamento do armazenamento de tabela do Azure

O armazenamento de tabelas do Azure é um repositório de chave/valor que foi desenvolvido em torno do particionamento. Todas as entidades são armazenadas em uma partição e as partições são gerenciadas internamente pelo armazenamento de tabela do Azure. Cada entidade armazenada em uma tabela deve fornecer uma chave de duas partes que inclui:

- **A chave de partição**. Esse é um valor de cadeia de caracteres que determina em qual partição o armazenamento de tabelas do Azure colocará a entidade. Todas as entidades com a mesma chave de partição são armazenadas na mesma partição.

- **A chave de linha**. Trata-se de um valor de cadeia de caracteres que identifica a entidade na partição. Todas as entidades em uma partição são classificadas lexicalmente, em ordem crescente, por essa chave. A combinação de chave de linha/chave de partição deve ser exclusiva para cada entidade e não pode exceder 1 KB.

Se uma entidade for adicionada a uma tabela com uma chave de partição não usada anteriormente, o armazenamento de tabelas do Azure criará uma nova partição para esta entidade. Outras entidades com a mesma chave de partição serão armazenadas na mesma partição.

Esse mecanismo implementa de modo efetivo uma estratégia de escala horizontal automática. Cada partição é armazenada no mesmo servidor em um datacenter do Azure para ajudar a garantir que as consultas que recuperam dados de uma única partição sejam executadas rapidamente.

A Microsoft publicou [alvos de escalabilidade] para o Armazenamento do Azure. Se for provável que o sistema exceda esses limites, considere dividir as entidades em várias tabelas. Use o particionamento vertical para dividir os campos nos grupos que têm mais probabilidade de serem acessados juntos.

O diagrama a seguir mostra a estrutura lógica de um exemplo de conta de armazenamento. A conta de armazenamento contém três tabelas: Informações de clientes, Informações de produto e Informações do pedido.

![As tabelas e partições em um exemplo de conta de armazenamento](./images/data-partitioning/TableStorage.png)

Cada tabela tem várias partições.

- Na tabela Informações de clientes, os dados são particionados de acordo com a cidade em que o cliente está localizado. A chave de linha contém a ID do cliente.
- Na tabela de Informações de produtos, os produtos são particionados por categoria e a chave de linha contém o número do produto.
- Na tabela de Informações de pedidos, os pedidos são particionados por data e a chave de linha especifica o horário em que o pedido foi recebido. Observe que todos os dados são ordenados pela chave de linha em cada partição.

Considere os seguintes pontos ao criar entidades para o armazenamento de tabelas do Azure:

- Selecione uma chave de partição e chave de linha de como os dados são acessados. Escolha uma combinação de chave de linha/chave de partição que seja compatível com a maioria de suas consultas. As consultas mais eficientes recuperam dados especificando a chave de partição e a chave de linha. As consultas que especificam uma chave de partição e um intervalo de chaves de linha podem ser concluídas por meio da verificação de uma única partição. Isso é relativamente rápido, pois os dados são mantidos na ordem da chave de linha. Se as consultas não especificarem qual partição verificar, todas devem ser verificadas.

- Se uma entidade tiver uma chave natural, use-a como a chave de partição e especifique uma cadeia de caracteres vazia como a chave de linha. Se uma entidade tiver uma chave composta que consista em duas propriedades, selecione a propriedade com alteração mais lenta como a chave de partição e a outra como a chave de linha. Se uma entidade tiver mais de duas propriedades de chave, use uma concatenação de propriedades para fornecer as chaves de partição e de linha.

- Se você executa regularmente consultas que pesquisam dados usando campos que não sejam as chaves de partição e de linha, considere a implementação do [Padrão da tabela de índice](../patterns/index-table.md) ou cogite usar um armazenamento de dados diferente que dê suporte ao índice, como o Cosmos DB.

- Se você gerar chaves de partição usando uma sequência monotônica (como “0001”, “0002”, “0003”) e cada partição contiver apenas uma quantidade limitada de dados, o armazenamento de tabelas do Azure poderá agrupar fisicamente essas partições no mesmo servidor. O Armazenamento do Azure pressupõe que o aplicativo tem mais probabilidade de executar consultas em um intervalo contíguo de partições (consultas do intervalo) e é otimizado para esse caso. No entanto, essa abordagem pode levar a pontos de acesso, pois todas as inserções das novas entidades provavelmente estão concentradas em uma extremidade dos intervalos contíguos. Ela também pode reduzir a escalabilidade. Para distribuir a carga mais uniformemente, cogite fazer o hash da chave de partição.

- O armazenamento de tabela do Azure dá suporte a operações transacionais para entidades que pertencem à mesma partição. Um aplicativo pode executar várias operações de inserção, atualização, exclusão, substituição ou mesclagem como uma unidade atômica, desde que a transação não inclua mais de 100 entidades e que a carga da solicitação não exceda 4 MB. As operações que abrangem várias partições não são transacionais e podem exigir que você implemente a consistência eventual. Para obter mais informações sobre armazenamento de tabela e transações, confira [Executar transações de grupo de entidade].

- Considere a granularidade da chave de partição:

  - O uso da mesma chave de partição para todas as entidades resulta em uma única partição que é mantida em um servidor. Isso impede a escala horizontal da partição e concentra a carga em um único servidor. Como resultado, essa abordagem só é adequada para armazenar uma pequena quantidade de entidades. No entanto, ela garante que todas as entidades possam participar das transações do grupo de entidade.

  - Usar uma chave de partição exclusiva para todas as entidades faz com que o serviço de armazenamento de tabelas crie uma partição separada para cada entidade, possivelmente, resultando em uma grande quantidade de partições pequenas. Essa abordagem é mais escalonável do que usar uma chave de partição única, mas as transações de grupo da entidade não são possíveis. Além disso, as consultas que buscam mais de uma entidade podem envolver a leitura em mais de um servidor. No entanto, se o aplicativo executar consultas do intervalo, o uso de uma sequência monotônica para chaves de partição pode ajudar a otimizar essas consultas.

  - Compartilhar a chave da partição entre um subconjunto de entidades torna possível o agrupamento de entidades relacionadas na mesma partição. As operações que envolvem entidades relacionadas podem ser executadas usando transações de grupo da entidade, e as consultas que buscam um conjunto de entidades relacionadas podem ser atendidas acessando um único servidor.

Para obter mais informações, confira o [Guia de design de armazenamento do Azure].

## <a name="partitioning-azure-blob-storage"></a>Particionamento do armazenamento de blobs do Azure

O armazenamento de blobs do Azure torna possível manter objetos binários grandes. Use blobs de blocos em cenários quando for necessário fazer upload ou download de grandes volumes de dados rapidamente. Use blobs de páginas para aplicativos que exigem o acesso aleatório em vez do acesso serial a partes dos dados.

Cada blob (de blocos ou páginas) é mantido em um contêiner em uma conta de armazenamento do Azure. Você pode usar contêineres para agrupar blobs relacionados que tenham os mesmos requisitos de segurança. Esse agrupamento é lógico em vez de físico. Dentro de um contêiner, cada blob tem um nome exclusivo.

A chave de partição de um blob é o nome da conta + nome do contêiner + nome do blob. A chave de partição é usada para particionar dados em intervalos e esses intervalos têm balanceamento de carga em todo o sistema. BLOBs podem ser distribuídos em vários servidores para escalar horizontalmente o acesso a eles, mas um único blob só pode ser atendido por um único servidor.

Se o esquema de nomenclatura usa os carimbos de data/hora ou identificadores numéricos, ele pode levar ao tráfego excessivo direcionado a uma partição, não permitindo que o sistema faça o balanceamento de carga efetivamente. Por exemplo, se você tiver operações diárias que usam um objeto de blob com um carimbo de data/hora, como *aaaa-mm-dd*, todo o tráfego para essa operação iria para um único servidor de partição. Em vez disso, cogite prefixar o nome com um hash de 3 dígitos. Para obter mais informações, confira [ Convenção de nomenclatura de partição](/azure/storage/common/storage-performance-checklist#subheading47)

As ações de gravação de um único bloco ou página são atômicas, mas as operações que abrangem blocos, páginas ou blobs não são. Se precisar garantir a consistência ao executar operações de gravação em blocos, páginas e blobs, remova um bloco de gravação usando uma concessão de blob.

## <a name="partitioning-azure-storage-queues"></a>Particionamento de filas de armazenamento do Azure

As filas de armazenamento do Azure permitem implementar mensagens assíncronas entre processos. Uma conta de armazenamento do Azure pode conter um número ilimitado de filas e cada fila pode conter um número ilimitado de mensagens. A única limitação é o espaço disponível na conta de armazenamento. O tamanho máximo de uma mensagem individual é de 64 KB. Se precisar de mensagens com tamanhos maiores do que esse, considere usar as filas do Barramento de Serviço do Azure.

Cada fila de armazenamento tem um nome exclusivo dentro da conta de armazenamento na qual ela está contida. O Azure particiona filas com base no nome. Todas as mensagens para a mesma fila são armazenadas na mesma partição, que é controlada por um único servidor. Diferentes filas podem ser gerenciadas por diferentes servidores para ajudar a balancear a carga. A alocação de filas para servidores é transparente para aplicativos e usuários.

Em um aplicativo de grande escala, não use a mesma fila de armazenamento para todas as instâncias do aplicativo, pois essa abordagem pode fazer com que o servidor que está hospedando a fila se torne um ponto de acesso. Em vez disso, use filas diferentes para diferentes áreas funcionais do aplicativo. As filas de armazenamento do Azure não dão suporte a transações; portanto, direcionar mensagens para diferentes filas deve ter pouco impacto na consistência das mensagens.

Uma fila de armazenamento do Azure pode manipular até 2.000 mensagens por segundo. Se precisar processar mensagens a uma taxa maior que essa, considere criar várias filas. Por exemplo, em um aplicativo global, crie filas de armazenamento separadas em contas de armazenamento separadas para manipular instâncias do aplicativo que estão sendo executadas em cada região.

## <a name="partitioning-azure-service-bus"></a>Particionamento do Barramento de Serviço do Azure

O Barramento de Serviço do Azure usa um agente de mensagem para manipular as mensagens que são enviadas para uma fila ou um tópico do Barramento de Serviço. Por padrão, todas as mensagens que são enviadas para uma fila ou um tópico são manipuladas pelo mesmo processo de agente de mensagem. Essa arquitetura pode colocar uma limitação na taxa de transferência geral da fila de mensagens. No entanto, você também pode particionar a uma fila ou um tópico quando ele é criado. Você pode fazer isso definindo a propriedade *EnablePartitioning* da fila ou da descrição do tópico para *true*.

Uma fila ou um tópico particionado é dividido em vários fragmentos, e cada um deles tem o suporte de um repositório e agente de mensagem separados. O Barramento de Serviço assume a responsabilidade pela criação e gerenciamento desses fragmentos. Quando um aplicativo publica uma mensagem em uma fila ou tópico particionado, o Barramento de Serviço atribui a mensagem a um fragmento dessa fila ou tópico. Quando um aplicativo recebe uma mensagem de uma fila ou assinatura, o Barramento de Serviço verifica cada fragmento em relação à próxima mensagem disponível e, em seguida, transmite-a para o aplicativo para o processamento.

Essa estrutura ajuda a distribuir a carga entre agentes e repositórios de mensagem, aumentando a escalabilidade e melhorando a disponibilidade. Se o repositório ou o agente de mensagem para um fragmento estiver temporariamente indisponível, o Barramento de Serviço poderá recuperar mensagens de um dos fragmentos restantes disponíveis.

O Barramento de Serviço atribui uma mensagem a um fragmento da seguinte maneira:

- Se a mensagem pertencer a uma sessão, todas as mensagens com o mesmo valor para a propriedade *SessionId* são enviadas para o mesmo fragmento.

- Se a mensagem não pertencer a uma sessão, mas o remetente especificou um valor para a propriedade *PartitionKey*, então, todas as mensagens com o mesmo valor *PartitionKey* serão enviadas para o mesmo fragmento.

  > [!NOTE]
  > Se as propriedades *SessionId* e *PartitionKey* forem especificadas, então, deverão ser definidas para o mesmo valor ou a mensagem será rejeitada.

- Se as propriedades *SessionId* e *PartitionKey* de uma mensagem não forem especificadas, mas a detecção de duplicidades estiver habilitada, a propriedade *MessageId* será usada. Todas as mensagens com a mesma *MessageId* serão direcionadas para o mesmo fragmento.

- Se as mensagens não incluírem uma propriedade *SessionId, PartitionKey* ou *MessageId*, então, o Barramento de Serviço atribuirá as mensagens aos fragmentos em sequência. Se um fragmento estiver indisponível, o Barramento de Serviço passará para o próximo. Isso significa que uma falha temporária na infraestrutura das mensagens não causa uma falha na operação de envio de mensagem.

Considere os seguintes pontos ao decidir se ou como particionar uma fila ou um tópico de mensagem do Barramento de Serviço:

- Tópicos e filas do Barramento de Serviço são criados dentro do escopo de um namespace do Barramento de Serviço. O Barramento de Serviço atualmente permite até 100 filas ou tópicos particionados por namespace.

- Cada namespace do Barramento de Serviço impõe cotas nos recursos disponíveis, como o número de assinaturas por tópico, o número de solicitações simultâneas de envio e recebimento por segundo e o número máximo de conexões simultâneas que podem ser estabelecidas. Essas cotas são documentadas nas [cotas do Barramento de Serviço]. Se você prevê que esses valores serão excedidos, crie namespaces adicionais com suas próprias filas e tópicos e distribua o trabalho entre esses namespaces. Por exemplo, em um aplicativo global, crie namespaces separados em cada região e configure instâncias do aplicativo para usar as filas e tópicos no namespace mais próximo.

- As mensagens enviadas como parte de uma transação devem especificar uma chave de partição. Isso pode ser uma propriedade *SessionId*, *PartitionKey* ou *MessageId*. Todas as mensagens enviadas como parte da mesma transação devem especificar a mesma chave de partição, pois elas devem ser manipuladas pelo mesmo processo de agente de mensagem. Você não pode enviar mensagens para diferentes filas ou tópicos na mesma transação.

- As filas e os tópicos particionados não podem ser configurados para serem excluídos automaticamente quando se tornam ociosos.

- Atualmente, as filas e os tópicos particionados não podem ser usados com o AMQP (Advanced Message Queuing Protocol) se você estiver criando soluções híbridas ou entre plataformas.

## <a name="partitioning-cosmos-db"></a>Particionamento do Cosmos DB

O Azure Cosmos DB é um banco de dados NoSQL pode armazenar documentos JSON usando a [API de SQL do Azure Cosmos DB][cosmosdb-sql-api]. Um documento no banco de dados do Cosmos DB é uma representação serializada pelo JSON de um objeto ou outro dado. Nenhum esquema fixo é imposto, exceto pelo fato de que cada documento deve conter uma ID exclusiva.

Os documentos são organizados em coleções. Você pode agrupar documentos relacionados em uma coleção. Por exemplo, em um sistema que mantém postagens de blog, você pode armazenar o conteúdo de cada postagem de blog como um documento em uma coleção. Também é possível criar coleções para cada tipo de assunto. Como alternativa, em um aplicativo multilocatário, como um sistema em que diferentes autores controlam e gerenciam suas postagens de blog, você pode particionar os blogs por autor e criar coleções separadas para cada um deles. O espaço de armazenamento que é alocado às coleções é flexível e pode diminuir ou aumentar conforme necessário.

O Cosmos DB suporta o particionamento automático de dados baseados numa chave de partição definida pelo aplicativo. Uma *partição lógica* é uma partição que armazena todos os dados para um valor de chave de partição única. Todos os documentos que compartilham o mesmo valor para a chave de partição são colocados na mesma partição lógica. O Cosmos DB distribui valores de acordo com o hash da chave de partição. Uma partição lógica tem um tamanho máximo de 10 GB. Portanto, a escolha da chave de partição é uma decisão importante no momento do design. Escolha uma propriedade com um amplo intervalo de valores e padrões de acesso regular. Para obter mais informações, veja [Partição e redução horizontal no Azure Cosmos DB](/azure/cosmos-db/partition-data).

> [!NOTE]
> Cada banco de dados do Cosmos DB tem um *nível de desempenho* que determina a quantidade de recursos que ele obtém. Um nível de desempenho é associado a um limite de taxa da *unidade de solicitação* (RU). O limite de taxa da RU especifica o volume de recursos que é reservado e disponibilizado para uso exclusivo dessa coleção. O custo de uma coleção depende do nível de desempenho que é selecionado para a coleção. Quanto maior o nível de desempenho (e o limite de taxa da RU), maior o custo. É possível ajustar o nível de desempenho de uma coleção usando o portal do Azure. Para obter mais informações, confira [Unidades de Solicitação no Azure Cosmos DB][cosmos-db-ru].

Se o mecanismo de particionamento que o Cosmos DB fornecer não for suficiente, pode ser preciso fragmentar os dados ao nível do aplicativo. As coleções de documentos fornecem um mecanismo natural para particionar dados em um banco de dados individual. A maneira mais simples de implementar a fragmentação é criar uma coleção para cada fragmento. Os contêineres são recursos lógicos e podem abranger um ou mais servidores. Contêineres de tamanho fixo têm um limite máximo de 10 GB e taxa de transferência de 10.000 RU/s. Os contêineres ilimitados não têm um tamanho de armazenamento máximo, mas devem especificar uma chave de partição. Com a fragmentação do aplicativo, o aplicativo cliente deve direcionar as solicitações ao fragmento adequado, normalmente pela implementação do seu próprio mecanismo de mapeamento com base em alguns atributos dos dados que definem a chave de fragmento.

Todos os bancos de dados são criados no contexto de uma conta de banco de dados do Cosmos DB. Uma única conta pode conter vários bancos de dados e especifica em qual região os bancos de dados são criados. Cada conta também impõe seu próprio controle de acesso. Você pode usar contas do Cosmos DB para localizar os fragmentos geograficamente (coleções nos bancos de dados) próximos aos usuários que precisam acessá-los e impor restrições para que somente esses usuários possam se conectar a eles.

Considere os seguintes pontos ao decidir como particionar dados com a API do SQL do Cosmos DB:

- **Os recursos disponíveis para um banco de dados do Cosmos DB estão sujeitos às limitações de cota da conta**. Cada banco de dados pode conter um número de coleções e cada coleção é associada a um nível de desempenho que rege o limite de taxa da RU (produtividade reservada) para essa coleção. Para saber mais, confira [Assinatura e limites de serviço, cotas e restrições do Azure][azure-limits].

- **Cada documento deve ter um atributo que pode ser usado para identificar exclusivamente o documento dentro da coleção na qual ele é mantido**. Esse atributo é diferente da chave de fragmento, que define qual coleção mantém o documento. Uma coleção pode conter um grande número de documentos. Teoricamente, ela é limitada apenas pelo tamanho máximo da ID do documento. A ID do documento pode ter até 255 caracteres.

- **Todas as operações em um documento são realizadas dentro do contexto de uma transação. As transações estão no escopo da coleção em que o documento está contido.** Se uma operação falhar, o trabalho realizado será revertido. Quando um documento está sujeito a uma operação, todas as alterações feitas estão sujeitas ao isolamento no nível do instantâneo. Esse mecanismo garante que se, por exemplo, houver uma falha em uma solicitação para a criação de um novo documento, outro usuário que estiver consultando o banco de dados simultaneamente não verá um documento parcial que será então removido.

- **As consultas do banco de dados também estão dentro do escopo do nível da coleção**. Uma única consulta pode recuperar dados somente de uma coleção. Se precisar recuperar dados de várias coleções, você deverá consultar cada coleção individualmente e mesclar os resultados no código do aplicativo.

- **O Cosmos DB oferece suporte aos itens programáveis que podem ser armazenados em uma coleção com os documentos**. Isso inclui procedimentos armazenados, funções definidas pelo usuário e gatilhos (escritos em JavaScript). Esses itens podem acessar qualquer documento na mesma coleção. Além disso, esses itens são executados dentro do escopo da transação de ambiente (no caso de um gatilho que é acionado como resultado de uma operação de criação, exclusão ou substituição executada em um documento) ou por iniciar uma nova transação (no caso de um procedimento armazenado executado como resultado de uma solicitação de cliente explícita). Se o código em um item programável lançar uma exceção, a transação será revertida. Você pode usar procedimentos armazenados e gatilhos para manter a integridade e a consistência entre documentos, mas esses documentos devem fazer parte da mesma coleção.

- **Provavelmente, as coleções que você pretende manter nos bancos de dados não excederão os limites de taxa de transferência definidos pelos níveis de desempenho das coleções**. Para obter mais informações, confira [Unidades de Solicitação no Azure Cosmos DB][cosmos-db-ru]. Se você prevê que esses limites serão atingidos, considere dividir as coleções em bancos de dados em diferentes contas para reduzir a carga por coleção.

## <a name="partitioning-azure-search"></a>Particionamento do Azure Search

A capacidade de pesquisar dados geralmente é o principal método de navegação e exploração fornecido por muitos aplicativos Web. Ela ajuda os usuários a encontrar recursos rapidamente (por exemplo, produtos em um aplicativo de comércio eletrônico) com base em combinações de critérios de pesquisa. O serviço de Azure Search fornece recursos de pesquisa de texto completo em relação ao conteúdo da Web e inclui recursos como consultas recomendadas de preenchimento automático com base em correspondências aproximadas e na navegação facetada. Para obter mais informações, confira [O que é o Azure Search?].

O Azure Search armazena conteúdo pesquisável como documentos JSON em um banco de dados. Defina os índices que especificam os campos pesquisáveis nesses documentos e forneça essas definições ao Azure Search. Quando um usuário envia uma solicitação de pesquisa, o Azure Search usa os índices adequados para localizar itens correspondentes.

Para reduzir a contenção, o armazenamento usado pelo Azure Search pode ser dividido em 1, 2, 3, 4, 6 ou 12 partições, e cada partição pode ser replicada até seis vezes. O produto do número de partições multiplicado pelo número de réplicas é denominado *unidade de pesquisa* (SU). Uma única instância do Azure Search pode conter um máximo de 36 SUs (um banco de dados com 12 partições somente dá suporte a um máximo de 3 réplicas).

Você será cobrado por cada SU alocada ao seu serviço. Conforme o volume de conteúdo pesquisável ou a taxa de solicitações de pesquisa aumenta, é possível adicionar SUs a uma instância existente do Azure Search para manipular a carga extra. O Azure Search em si distribui os documentos uniformemente entre as partições. Atualmente, nenhuma estratégia de particionamento manual é compatível.

Cada partição pode conter um máximo de 15 milhões de documentos ou ocupar 300 GB de espaço de armazenamento (o que for menor). Você pode criar até 50 índices. O desempenho do serviço varia e depende da complexidade dos documentos, dos índices disponíveis e dos efeitos da latência de rede. Em média, uma única réplica (1 SU) deve ser capaz de manipular 15 QPS (consultas por segundo), embora seja recomendável avaliar o desempenho com seus próprios dados para obter uma medida mais precisa da taxa de transferência. Para obter mais informações, confira [Limites de serviço no Azure Search].

> [!NOTE]
> Você pode armazenar um conjunto limitado de tipos de dados em documentos pesquisáveis, incluindo cadeias de caracteres, boolianos, dados numéricos, dados de data/hora e alguns dados geográficos. Para obter mais detalhes, visite a página [Tipos de Dados com Suporte (Azure Search)] no site da Microsoft.

Você tem controle limitado sobre como o Azure Search particiona os dados para cada instância do serviço. No entanto, em um ambiente global, é possível melhorar o desempenho e reduzir ainda mais a latência e contenção pelo particionamento do próprio serviço usando qualquer uma das seguintes estratégias:

- Crie uma instância do Azure Search em cada região geográfica e verifique se os aplicativos cliente são direcionados à instância disponível mais próxima. Essa estratégia exige que todas as atualizações de conteúdo pesquisável sejam replicadas pontualmente em todas as instâncias do serviço.

- Crie duas camadas de Azure Search:

  - Um serviço local em cada região que contenha os dados que são acessados com mais frequência pelos usuários nessa região. Os usuários podem direcionar solicitações para obter resultados rápidos, mas limitados.
  - Um serviço global que inclua todos os dados. Os usuários podem direcionar solicitações para obter resultados mais demorados, porém, mais completos.

Essa abordagem é mais adequada quando há uma variação regional significativa nos dados que estão sendo pesquisados.

## <a name="partitioning-azure-redis-cache"></a>Particionamento do Cache Redis do Azure

O Cache Redis do Azure fornece um serviço de cache compartilhado na nuvem que se baseia no repositório de dados de chave/valor do Redis. Como o nome sugere, o Cache Redis do Azure serve como uma solução de cache. Use-o apenas para dados manter dados transitórios, e não como um repositório de dados permanentes. Os aplicativos que usam o Cache Redis do Azure devem poder continuar funcionando se o cache ficar indisponível. O Cache Redis do Azure permite a replicação primária/secundária para fornecer alta disponibilidade, mas atualmente limita o tamanho máximo do cache para 53 GB. Se você precisar de mais espaço, é necessário criar caches adicionais. Para obter mais informações, confira [Cache Redis do Azure].

O particionamento de um repositório de dados do Redis envolve a divisão dos dados entre instâncias do serviço do Redis. Cada instância constitui uma única partição. O Cache Redis do Azure abstrai os serviços do Redis por trás de uma fachada e não os expõe diretamente. A maneira mais simples de implementar o particionamento é criar várias instâncias do Cache Redis do Azure e distribuir os dados entre eles.

Você pode associar cada item de dados a um identificador (uma chave de partição) que especifica qual cache armazena o item de dados. A lógica do aplicativo cliente pode usar esse identificador para rotear solicitações para a partição adequada. Esse esquema é muito simples, mas se o esquema de particionamento for alterado (por exemplo, se instâncias adicionais do Cache Redis do Azure forem criadas), os aplicativos cliente precisarão ser reconfigurados.

O Redis nativo (não o Cache Redis do Azure) dá suporte ao particionamento do lado do servidor com base no clustering do Redis. Nessa abordagem, é possível dividir os dados uniformemente entre os servidores usando um mecanismo de hash. Cada servidor do Redis armazena metadados que descrevem o intervalo de chaves de hash contidos na partição, além de conter informações sobre quais chaves de hash estão localizadas nas partições em outros servidores.

Os aplicativos cliente simplesmente enviam solicitações para qualquer um dos servidores Redis participantes (provavelmente o mais próximo). O servidor Redis examina a solicitação do cliente. Se ela puder ser resolvida localmente, o servidor executa a operação solicitada. Caso contrário, ele encaminha a solicitação para o servidor adequado.

Esse modelo é implementado usando o clustering do Redis e é descrito mais detalhadamente na página [tutorial de cluster do Redis] no site do Redis. O clustering do Redis é transparente para os aplicativos cliente. Outros servidores do Redis podem ser adicionados ao cluster (e os dados podem ser particionados novamente) sem a necessidade de reconfigurar os clientes.

> [!IMPORTANT]
> O Cache Redis do Azure atualmente oferece suporte ao clustering do Redis somente na camada [premium](/azure/azure-cache-for-redis/cache-how-to-premium-clustering).

A página [Partitioning: how to split data among multiple Redis instances] , no site do Redis, fornece mais informações sobre a implementação do particionamento com o Redis. Ao ler o restante desta seção, pressupomos que você esteja implementando o particionamento do lado do cliente ou assistido por proxy.

Considere os seguintes pontos ao decidir como particionar dados com o Cache Redis do Azure:

- O Cache Redis do Azure não pretende agir como um repositório de dados permanente; portanto, seja qual for o esquema de particionamento que você implemente, o código do aplicativo deverá poder recuperar dados de um local que não seja o cache.

- Os dados que são acessados juntos frequentemente devem ser mantidos na mesma partição. O Redis é um poderoso repositório de chave/valor que fornece vários mecanismos altamente otimizados para estruturação de dados. Esses mecanismos podem ser um dos seguintes:
  - Cadeias de caracteres simples (dados binários de até 512 MB)
  - Tipos de agregação, como listas (que podem atuar como filas e pilhas)
  - Conjuntos (ordenados e não ordenados)
  - Hashes (que podem agrupar campos relacionados, como os itens que representam os campos em um objeto)

- Os tipos de agregação permitem associar vários valores relacionados com a mesma chave. Uma chave do Redis identifica uma lista, um conjunto ou um hash em vez de itens de dados que ela contém. Esses tipos estão disponíveis com o Cache Redis do Azure e são descritos na página [Data types] no site do Redis. Por exemplo, em uma parte de um sistema de comércio eletrônico que rastreia os pedidos que são feitos pelos clientes, os detalhes de cada cliente podem ser armazenados em um hash do Redis que é digitado usando a ID do cliente. Cada hash pode manter uma coleção de IDs de pedido do cliente. Um conjunto separado do Redis pode manter os pedidos, novamente estruturados como hashes, e digitados com o uso da ID de pedido. A Figura 8 mostra essa estrutura. Observe que o Redis não implementa nenhuma forma de integridade referencial; portanto, é responsabilidade do desenvolvedor manter os relacionamentos entre clientes e pedidos.

![Estrutura recomendada no armazenamento do Redis para registrar pedidos de clientes e seus detalhes](./images/data-partitioning/RedisCustomersandOrders.png)

*Figura 8. Estrutura recomendada no armazenamento do Redis para registrar pedidos de clientes e seus detalhes.*

> [!NOTE]
> No Redis, todas as chaves são valores de dados binários (como cadeias de caracteres do Redis) e podem conter até 512 MB de dados. Teoricamente, uma chave pode conter quase todo tipo de informação. No entanto, é recomendável adotar uma convenção de nomenclatura consistente para chaves que descrevam o tipo de dados e identifiquem a entidade, mas que não seja excessivamente longa. Uma abordagem comum é usar chaves na forma "entity_type: ID". Por exemplo, você pode usar "customer:99" para indicar a chave de um cliente com a ID 99.

- Você pode implementar o particionamento vertical armazenando informações relacionadas em diferentes agregações no mesmo banco de dados. Por exemplo, em um aplicativo de comércio eletrônico, você pode armazenar as informações frequentemente acessadas sobre produtos em um hash do Redis e as informações detalhadas usadas com menos frequência em outro. Os dois hashes podem usar a mesma ID de produto como parte da chave. Por exemplo, você pode usar "produto: *nn*" (onde *nn* é a ID do produto) para as informações do produto e "product_details: *nn*" para os dados detalhados. Essa estratégia pode ajudar a reduzir o volume de dados que a maioria das consultas provavelmente recuperará.

- Você pode reparticionar um repositório de dados do Redis, mas lembre-se de que é uma tarefa complexa e demorada. O clustering do Redis pode reparticionar os dados automaticamente, mas esse recurso não está disponível com o Cache Redis do Azure. Portanto, ao criar o esquema de particionamento, tente deixar espaço livre suficiente em cada partição para permitir o crescimento esperado dos dados com o tempo. No entanto, lembre-se de que o Cache Redis do Azure foi criado para armazenar os dados em cache temporariamente e que os dados mantidos no cache podem ter um tempo de vida limitado especificado como um valor de tempo de vida (TTL). Para dados relativamente voláteis, o TTL pode ser curto, mas para dados estáticos, o TTL pode ser muito mais longo. Evite armazenar grandes quantidades de dados com tempo de vida longo no cache se o volume de dados tiver a probabilidade de preencher o cache. É possível especificar uma política de remoção que faz com que o Cache Redis do Azure remova os dados se o espaço for um fator determinante.

  > [!NOTE]
  > Quando você usa o Cache Redis do Azure, especifica o tamanho máximo do cache (de 250 MB a 53 GB) selecionando o tipo de preço adequado. No entanto, depois de criar um Cache Redis do Azure, não é possível aumentar (ou diminuir) seu tamanho.

- Como os lotes e as transações do Redis não abrangem várias conexões, todos os dados afetados por um lote ou uma transação devem ser mantidos no mesmo banco de dados (fragmento).

  > [!NOTE]
  > Uma sequência de operações em uma transação do Redis não é necessariamente atômica. Os comandos que compõem uma transação são verificados e enfileirados antes de serem executados. Se ocorrer um erro durante essa fase, a fila inteira será descartada. No entanto, depois que a transação tiver sido enviada com êxito, os comandos na fila serão executados na sequência. Se algum comando falhar, somente a execução desse comando será interrompida. Todos os comandos anteriores e subsequentes na fila são executados. Para saber mais, visite a página [Transações] no site do Redis.

- O Redis permite um número limitado de operações atômicas. As únicas operações desse tipo que permitem vários valores e chaves são MGET e MSET. As operações MGET retornam uma coleção de valores para uma lista de chaves especificada e as operações MSET armazenam uma coleção de valores para uma lista de chaves especificada. Se precisar usar essas operações, os pares de chave/valor referenciados pelos comandos MSET e MGET devem ser armazenados no mesmo banco de dados.

## <a name="partitioning-azure-service-fabric"></a>Particionamento do Azure Service Fabric

O Azure Service Fabric é uma plataforma de microsserviços que fornece um tempo de execução para aplicativos distribuídos na nuvem. O Service Fabric oferece suporte a executáveis convidados do .Net, serviços com e sem monitoração de estado e contêineres. Serviços com monitoração de estado fornecem um [coleção confiável][service-fabric-reliable-collections] para armazenar dados de maneira persistente em uma coleção de chave-valor dentro do cluster do Service Fabric. Para saber mais sobre as estratégias de particionamento de chaves em uma coleção confiável, consulte [Diretrizes e recomendações para coleções confiáveis no Azure Service Fabric].

### <a name="more-information"></a>Mais informações

- [Visão geral do Azure Service Fabric] é uma introdução ao Azure Service Fabric.

- [Particionar serviços confiáveis do Service Fabric] fornece mais informações sobre os serviços confiáveis no Azure Service Fabric.

## <a name="partitioning-azure-event-hubs"></a>Particionamento dos Hubs de Eventos do Azure

Os [Hubs de Eventos do Azure][event-hubs] foram projetados para streaming em grande escala e o particionamento de dados foi inserido no serviço para habilitar o escalonamento horizontal. Cada consumidor lê somente uma partição específica de fluxo de mensagens.

O editor de eventos só está ciente da sua chave de partição, não da partição para a qual os eventos são publicados. Essa desassociação de chave e partição isenta o remetente da necessidade de saber muito sobre o processamento de downstream. (Também é possível enviar eventos diretamente para uma determinada partição, mas geralmente isso não é recomendável).  

Considere a escala de longo prazo quando você seleciona a contagem de partições. Após a criação de um hub de eventos, você não poderá alterar o número de partições.

Para saber mais sobre como usar as partições em Hubs de Eventos, veja [O que são Hubs de Eventos?].

Para obter considerações sobre compensações entre disponibilidade e a consistência, veja [Disponibilidade e consistência nos Hubs de Eventos].

[Disponibilidade e consistência nos Hubs de Eventos]: /azure/event-hubs/event-hubs-availability-and-consistency
[azure-limits]: /azure/azure-subscription-service-limits
[Azure Content Delivery Network]: /azure/cdn/cdn-overview
[Cache Redis do Azure]: https://azure.microsoft.com/services/cache/
[Azure Storage Scalability and Performance Targets]: /azure/storage/storage-scalability-targets
[Guia de design de armazenamento do Azure]: /azure/storage/storage-table-design-guide
[Building a Polyglot Solution]: https://msdn.microsoft.com/library/dn313279.aspx
[cosmos-db-ru]: /azure/cosmos-db/request-units
[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence]: https://msdn.microsoft.com/library/dn271399.aspx
[Data consistency primer]: https://aka.ms/Data-Consistency-Primer
[Data Partitioning Guidance]: https://msdn.microsoft.com/library/dn589795.aspx
[Data Types]: https://redis.io/topics/data-types
[cosmosdb-sql-api]: /azure/cosmos-db/sql-api-introduction
[Elastic Database features overview]: /azure/sql-database/sql-database-elastic-scale-introduction
[event-hubs]: /azure/event-hubs
[Federations Migration Utility]: https://code.msdn.microsoft.com/vstudio/Federations-Migration-ce61e9c1
[diretrizes e recomendações para Coleções Confiáveis no Azure Service Fabric]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections-guidelines
[Multi-shard querying]: /azure/sql-database/sql-database-elastic-scale-multishard-querying
[Visão geral do Azure Service Fabric]: /azure/service-fabric/service-fabric-overview
[Particionar serviços confiáveis do Service Fabric]: /azure/service-fabric/service-fabric-concepts-partitioning
[Partitioning: how to split data among multiple Redis instances]: https://redis.io/topics/partitioning
[Executar transações de grupo de entidade]: /rest/api/storageservices/Performing-Entity-Group-Transactions
[tutorial de cluster do Redis]: https://redis.io/topics/cluster-tutorial
[Running Redis on a CentOS Linux VM in Azure]: https://blogs.msdn.microsoft.com/tconte/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure/
[Scaling using the Elastic Database split-merge tool]: /azure/sql-database/sql-database-elastic-scale-overview-split-and-merge
[Using Azure Content Delivery Network]: /azure/cdn/cdn-create-new-endpoint
[Cotas do Barramento de Serviço]: /azure/service-bus-messaging/service-bus-quotas
[service-fabric-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[Limites de serviço no Azure Search]:  /azure/search/search-limits-quotas-capacity
[Sharding pattern]: ../patterns/sharding.md
[Tipos de Dados com Suporte (Azure Search)]:  https://msdn.microsoft.com/library/azure/dn798938.aspx
[Transações]: https://redis.io/topics/transactions
[O que são Hubs de Eventos?]: /azure/event-hubs/event-hubs-what-is-event-hubs
[O que é o Azure Search?]: /azure/search/search-what-is-azure-search
[What is Azure SQL Database?]: /azure/sql-database/sql-database-technical-overview
[alvos de escalabilidade]: /azure/storage/common/storage-scalability-targets
