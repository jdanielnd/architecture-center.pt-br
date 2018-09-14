---
title: Diretrizes de particionamento de dados
description: Instruções de como separar partições a serem gerenciadas e acessadas separadamente.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: d1d9c1b3cf07f724eb010fc260d86ceb84b789ca
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/13/2018
ms.locfileid: "29059965"
---
# <a name="data-partitioning"></a>Particionamento de dados

Em muitas soluções de grande escala, os dados são divididos em partições separadas que podem ser gerenciadas e acessadas separadamente. A estratégia de particionamento deve ser cuidadosamente escolhida para maximizar os benefícios, ao mesmo tempo que minimiza os efeitos adversos. O particionamento pode ajudar a aprimorar o dimensionamento, a reduzir a contenção e a otimizar o desempenho. Outro benefício do particionamento é que ele pode fornecer um mecanismo para dividir dados pelo padrão de uso. Por exemplo, você pode arquivar dados antigos e menos ativos (passivos) em armazenamento de dados mais barato.

## <a name="why-partition-data"></a>Por que particionar os dados?
A maioria dos serviços e aplicativos na nuvem armazena e recupera dados como parte de suas operações. O design dos repositórios de dados usado por um aplicativo pode ter uma influência significativa sobre o desempenho, taxa de transferência e escalabilidade de um sistema. Uma técnica normalmente aplicada em sistemas de grande escala é dividir os dados em partições separadas.

> Neste artigo, o termo *particionamento* significa o processo de dividir fisicamente os dados em armazenamentos de dados separados. Não é igual ao particionamento de tabela do SQL Server.

O particionamento de dados pode oferecer uma série de benefícios. Por exemplo, ele pode ser aplicado para:

* **Melhorar a escalabilidade**. Quando você escala verticalmente um sistema de banco de dados individual, em algum momento ele atingirá um limite de hardware físico. Se você dividir dados entre várias partições, cada uma das quais hospedada em um servidor separado, será possível escalar o sistema de forma horizontal quase que indefinidamente.
* **Melhorar o desempenho**. As operações de acesso a dados em cada partição ocorrem em um volume de dados menor. Desde que os dados sejam particionados de maneira adequada, o particionamento poderá tornar seu sistema mais eficiente. As operações que afetam mais de uma partição podem ser executadas paralelamente. Cada partição pode estar localizada próximo a um aplicativo que a usa para minimizar a latência de rede.
* **Melhorar a disponibilidade**. Separar dados em vários servidores evita um ponto único de falha. Se um servidor falhar, ou estiver passando por uma manutenção planejada, somente os dados nessa partição não estarão disponíveis. As operações em outras partições podem continuar. Aumentar o número de partições reduz o impacto relativo de uma falha de servidor único ao reduzir o percentual dos dados que não estarão disponíveis. Replicar cada partição pode reduzir ainda mais a probabilidade de uma falha de partição única afetar as operações. Isso também possibilita a separação de dados críticos que devem estar contínua e altamente disponíveis dos dados de valor baixo que tenham requisitos de disponibilidade mais baixos (arquivos de log, por exemplo).
* **Melhorar a segurança**. Dependendo da natureza dos dados e de como eles são particionados, é possível separar dados confidenciais e não confidenciais em partições diferentes e, portanto, em diferentes servidores ou repositórios de dados. Em seguida, a segurança pode ser especificamente otimizada para os dados confidenciais.
* **Fornecer flexibilidade operacional**. O particionamento oferece várias oportunidades para o ajuste das operações, maximizando a eficiência administrativa e minimizando os custos. Por exemplo, você pode definir diferentes estratégias para gerenciamento, monitoramento, backup, restauração e outras tarefas administrativas baseadas na importância dos dados em cada partição.
* **Fazer a correspondência do repositório de dados ao padrão de uso**. O particionamento permite que cada partição seja implantada em um tipo diferente de repositório de dados, com base no custo e nos recursos internos que o repositório de dados oferece. Por exemplo, dados binários grandes podem ser armazenados em um repositório de dados de blob, enquanto dados mais estruturados podem ser mantidos em um banco de dados de documentos. Para obter mais informações, consulte [Criando uma solução poliglota] no guia de padrões e práticas, e [Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence] no site da Microsoft.

Alguns sistemas não implementam o particionamento, pois ele é considerado um custo em vez de uma vantagem. Entre alguns dos motivos comuns para essa lógica estão:

* Muitos sistemas de armazenamento de dados não dão suporte a junções entre partições e pode ser difícil manter a integridade referencial em um sistema particionado. Frequentemente, é necessário implementar junções e verificações de integridade no código do aplicativo (na camada de particionamento), o que poderá resultar em E/S adicional e na complexidade do aplicativo.
* Manter partições nem sempre é uma tarefa comum. Em um sistema em que os dados são voláteis, talvez seja necessário rebalancear as partições periodicamente para reduzir a contenção e os pontos de acesso.
* Obviamente, algumas ferramentas comuns não funcionam com dados particionados.

## <a name="designing-partitions"></a>Criando partições
Os dados podem ser particionados de diferentes maneiras: horizontal, vertical ou funcionalmente. A estratégia escolhida depende do motivo para o particionamento dos dados e dos requisitos dos aplicativos e serviços que usarão os dados.

> [!NOTE]
> Os esquemas de particionamento descritos nestas diretrizes são explicados de forma independente da tecnologia subjacente de armazenamento de dados. Eles podem ser aplicados a vários tipos de repositórios de dados, incluindo bancos de dados relacionais e NoSQL.
>
>

### <a name="partitioning-strategies"></a>Estratégias de particionamento
As três estratégias típicas para o particionamento dos dados são:

* **Particionamento horizontal** (geralmente denominado *fragmentação*). Nessa estratégia, cada partição é um repositório de dados por si só, mas todas as partições têm o mesmo esquema. Cada partição é conhecida como um *fragmento* e contém um subconjunto específico de dados, como todos os pedidos de um conjunto específico de clientes em um aplicativo de comércio eletrônico.
* **Particionamento vertical**. Nessa estratégia, cada partição contém um subconjunto dos campos de itens no repositório de dados. Os campos são divididos de acordo com seu padrão de uso. Por exemplo, os campos acessados com frequência podem ser colocados em uma partição vertical e os campos acessados com menos frequência em outra.
* **Particionamento funcional**. Nessa estratégia, os dados são agregados de acordo com o modo como são usados por cada contexto vinculado no sistema. Por exemplo, um sistema de comércio eletrônico que implementa funções comerciais separadas para o faturamento e o gerenciamento do inventário de produtos pode armazenar dados de faturas em uma partição e dados do inventário de produtos em outra.

É importante observar que as três estratégias descritas aqui podem ser combinadas. Elas não são mutuamente exclusivas, e é recomendável considerá-las quando você projeta um esquema de particionamento. Por exemplo, você poderia dividir os dados em fragmentos e então usar o particionamento vertical para subdividir ainda mais os dados em cada fragmento. Da mesma forma, os dados em uma partição funcional podem ser divididos em fragmentos (que também podem ser verticalmente particionados).

No entanto, os diferentes requisitos de cada estratégia podem gerar uma série de problemas conflitantes. Você deve avaliar e balancear todos eles ao projetar um esquema de particionamento que atenda às metas gerais de desempenho de processamento de dados do seu sistema. As seções a seguir exploram cada uma das estratégias mais detalhadamente.

### <a name="horizontal-partitioning-sharding"></a>Particionamento horizontal (fragmentação)
A Figura 1 mostra uma visão geral do particionamento horizontal ou fragmentação. Neste exemplo, os dados do inventário de produtos são divididos em fragmentos com base na chave do produto (Product Key). Cada fragmento contém os dados para um intervalo contíguo de chaves de fragmento (A-G e H-Z), organizadas em ordem alfabética.

![Particionamento horizontal (fragmentação) de dados com base em uma chave de partição](./images/data-partitioning/DataPartitioning01.png)

*Figura 1. Particionamento horizontal (fragmentação) de dados com base em uma chave de partição*

A fragmentação ajuda você a distribuir a carga em um número maior de computadores, o que reduz a contenção e melhora o desempenho. Você pode escalar o sistema horizontalmente acrescentando mais fragmentos que são executados em servidores adicionais.

O fator mais importante ao implementar essa estratégia de particionamento é a opção de chave de fragmentação. Pode ser difícil alterar a chave depois que o sistema estiver em operação. A chave deve garantir que os dados sejam particionados para que a carga de trabalho seja a mais uniforme possível entre os fragmentos.

Observe que diferentes fragmentos não precisam conter volumes semelhantes de dados. Em vez disso, a consideração mais importante é equilibrar o número de solicitações. Alguns fragmentos podem ser muito grandes, mas cada item está sujeito a um número baixo de operações de acesso. Outros fragmentos podem ser menores, mas cada item é acessado com muito mais frequência. Também é importante garantir que um único fragmento não exceda os limites de escala (em termos de recursos de capacidade e processamento) do repositório de dados usado para hospedar esse fragmento.

Se você usar um esquema de fragmentação, evite criar pontos de acesso (ou partições mais acessadas) que possam afetar o desempenho e a disponibilidade. Por exemplo, se você usar o hash de um identificador de cliente no lugar da primeira letra do nome do cliente, isso impedirá a distribuição desbalanceada resultante de letras iniciais comuns e menos comuns. Essa é uma técnica típica que ajuda a distribuir os dados mais uniformemente nas partições.

Escolha uma chave de fragmentação que minimize os requisitos futuros de divisão de grandes fragmentos em partes menores, agrupe pequenos fragmentos em partições maiores ou altere o esquema que descreve os dados armazenados em um conjunto de partições. Essas operações podem ser muito demoradas e podem exigir que você deixe um ou mais fragmentos offline enquanto elas são executadas.

Se os fragmentos forem replicados, será possível manter algumas das réplicas online enquanto as outras são divididas, mescladas ou reconfiguradas. No entanto, talvez o sistema precise limitar as operações que podem ser realizadas nos dados nesses fragmentos enquanto a reconfiguração estiver em andamento. Por exemplo, os dados nas réplicas podem ser marcados como somente leitura para limitar o escopo de inconsistências que podem ocorrer enquanto os fragmentos estiverem sendo reestruturados.

> Para obter informações mais detalhadas e diretrizes sobre várias dessas considerações, além de técnicas de práticas recomendadas para a criação de repositórios de dados que implementam o particionamento horizontal, consulte [Sharding pattern](Padrão de fragmentação).
>
>

### <a name="vertical-partitioning"></a>Particionamento vertical
O uso mais comum do particionamento vertical é reduzir a E/S e os custos de desempenho associados à busca de itens que são acessados com mais frequência. A Figura 2 mostra um exemplo de particionamento vertical. Neste exemplo, as diferentes propriedades de cada item de dados são mantidas em diferentes partições. Uma partição mantém dados que são acessados mais frequentemente, incluindo o nome, a descrição e as informações de preço dos produtos. Outra mantém o volume em estoque e a data do último pedido.

![Particionamento vertical de dados por seu padrão de uso](./images/data-partitioning/DataPartitioning02.png)

*Figura 2. Particionamento vertical de dados por seu padrão de uso*

Neste exemplo, o aplicativo regularmente consulta o nome, a descrição e o preço do produto ao exibir os detalhes do produto para os clientes. O nível de estoque e a data mais recente em que o produto foi encomendado do fabricante são mantidos em uma partição separada, já que esses dois itens são usados juntos.

Esse esquema de particionamento tem uma vantagem adicional de que os dados relativamente lentos (nome do produto, descrição e preço) são separados dos dados mais dinâmicos (nível de estoque e data do último pedido). Um aplicativo pode considerar útil armazenar em cache na memória os dados lentos se eles forem acessados com frequência.

Outro cenário típico dessa estratégia de particionamento é maximizar a segurança dos dados confidenciais. Por exemplo, você pode fazer isso armazenando números de cartão de crédito e números de verificação de segurança do cartão correspondentes em partições separadas.

O particionamento vertical também pode reduzir a quantidade de acesso simultâneo necessário aos dados.

> O particionamento vertical funciona no nível da entidade em um armazenamento de dados, parcialmente normalizando uma entidade para dividir um item *grande* em um conjunto de itens *pequenos*. Ele é ideal para repositórios de dados orientados a colunas, como o HBase e Cassandra. Se for improvável que os dados em uma coleção de colunas serão alterados, você também poderá considerar o uso de repositórios de colunas no SQL Server.
>
>

### <a name="functional-partitioning"></a>Particionamento funcional
Para os sistemas em que seja possível identificar um contexto vinculado para cada área ou serviço comercial distinto no aplicativo, o particionamento funcional fornece uma técnica para a melhoria do isolamento e do desempenho de acesso a dados. Outro uso comum do particionamento funcional é separar dados de leitura/gravação de dados somente leitura usados para fins de relatório. A Figura 3 mostra uma visão geral de particionamento funcional em que os dados do inventário são separados dos dados do cliente.

![Particionamento funcional de dados por contexto ou subdomínio vinculado](./images/data-partitioning/DataPartitioning03.png)

*Figura 3. Particionamento funcional de dados por contexto ou subdomínio vinculado*

Essa estratégia de particionamento pode ajudar a reduzir a contenção do acesso a dados em diferentes partes de um sistema.

## <a name="designing-partitions-for-scalability"></a>Criando partições para a escalabilidade
É fundamental considerar o tamanho e a carga de trabalho de cada partição e balanceá-los para que os dados sejam distribuídos para obtenção da máxima escalabilidade. No entanto, você também deve particionar os dados para que eles não excedam os limites de dimensionamento de um único repositório de partição.

Siga estas etapas ao criar partições para escalabilidade:

1. Analise o aplicativo para entender os padrões de acesso a dados, como o tamanho do conjunto de resultados retornado por cada consulta, a frequência de acesso, a latência inerente e os requisitos de processamento de computação do lado do servidor. Em muitos casos, algumas entidades principais exigirão a maior parte dos recursos de processamento.
2. Use essa análise para determinar as metas de escalabilidade atuais e futuras, como o tamanho dos dados e carga de trabalho. Em seguida, distribua os dados nas partições para atender à meta de escalabilidade. Na estratégia de particionamento horizontal, é importante escolher a chave de fragmento adequada para garantir que a distribuição seja uniforme. Para obter mais informações, consulte [Sharding pattern](Padrão de fragmentação).
3. Verifique se os recursos disponíveis para cada partição são suficientes para lidar com os requisitos de escalabilidade em termos de tamanho de dados e taxa de transferência. Por exemplo, o nó que está hospedando uma partição pode impor um limite rígido na quantidade de espaço de armazenamento, na capacidade de processamento ou largura de banda da rede que ele fornece. Se os requisitos de armazenamento e processamento de dados tiverem a probabilidade de exceder esses limites, poderá ser necessário refinar sua estratégia de particionamento ou subdividir mais os dados. Por exemplo, uma abordagem de escalabilidade pode ser separar os dados de log dos principais recursos do aplicativo. Você pode fazer isso usando repositórios de dados separados para impedir que os requisitos de armazenamento total de dados exceda o limite de ajuste de escala do nó. Se o número total de repositórios de dados exceder o limite do nó, pode ser necessário usar nós de armazenamento separados.
4. Monitore o sistema em uso para verificar se os dados são distribuídos conforme esperado e se as partições podem manipular a carga imposta sobre elas. É possível que o uso não coincida com o uso previsto pela análise. Nesse caso, pode ser possível redistribuir as partições. Se isso não funcionar, talvez seja preciso recriar algumas partes do sistema para ganhar o balanceamento necessário.

Observe que alguns ambientes de nuvem alocam recursos em termos de limites de infraestrutura. Garanta que os limites de seu limite selecionado forneçam espaço suficiente para qualquer aumento previsto no volume de dados, em termos de armazenamento de dados, capacidade de processamento e largura de banda.

Por exemplo, se você usar o armazenamento de tabelas do Azure, um fragmento ocupado pode exigir mais recursos do que estão disponíveis para uma única partição tratar solicitações. (Há um limite para o volume de solicitações que podem ser identificadas por uma única partição em um determinado período de tempo. Consulte a página [Azure Storage Scalability and Performance Targets] no site da Microsoft para obter mais detalhes.)

 Nesse caso, o fragmento pode precisar ser reparticionado para distribuir a carga. Se o tamanho total ou a taxa de transferência dessas tabelas exceder a capacidade de uma conta de armazenamento, pode ser necessário criar mais contas de armazenamento e distribuir as tabelas entre essas contas. Se o número de contas de armazenamento exceder o número de contas disponíveis para uma assinatura, pode ser necessário usar várias assinaturas.

## <a name="designing-partitions-for-query-performance"></a>Criando partições para o desempenho da consulta
Muitas vezes, o desempenho da consulta pode ser impulsionado pelo uso de conjuntos de dados menores e pela execução de consultas paralelas. Cada partição deve conter uma pequena proporção do conjunto de dados inteiro. Essa redução no volume pode melhorar o desempenho das consultas. No entanto, o particionamento não é uma alternativa para a criação e configuração adequadas de um banco de dados. Por exemplo, verifique se você implementou os índices necessários se estiver usando um banco de dados relacional.

Siga estas etapas ao criar partições para desempenho de consulta:

1. Examine os requisitos e o desempenho do aplicativo:
   * Use os requisitos de negócios para determinar as consultas críticas que devem sempre ser executadas rapidamente.
   * Monitore o sistema para identificar todas as consultas que são executadas lentamente.
   * Estabeleça as consultas que são executadas com mais frequência. Uma única instância de cada consulta pode ter um custo mínimo, mas o consumo cumulativo de recursos pode ser significativo. Pode ser útil separar os dados recuperados por essas consultas em uma partição distinta ou até mesmo em um cache.
2. Particione os dados que estejam causando lentidão no desempenho:
   * Limitar o tamanho de cada partição para que o tempo de resposta da consulta esteja dentro da meta.
   * Crie a chave de fragmento para que o aplicativo encontre facilmente a partição, se você estiver implementando o particionamento horizontal. Isso impede que a consulta tenha que verificar cada partição.
   * Considere o local de uma partição. Se possível, tente manter os dados nas partições que estão geograficamente próximas aos aplicativos e aos usuários que os acessam.
3. Se uma entidade tiver requisitos de desempenho da consulta e de taxa de transferência, use o particionamento funcional com base nessa entidade. Se isso ainda não atender aos requisitos, aplique também o particionamento horizontal. Na maioria dos casos, uma única estratégia de particionamento será suficiente, mas em alguns casos, é mais eficiente combinar as duas estratégias.
4. Considere usar consultas assíncronas que são executadas paralelamente nas partições para melhorar o desempenho.

## <a name="designing-partitions-for-availability"></a>Criando partições para a disponibilidade
O particionamento de dados pode melhorar a disponibilidade de aplicativos, garantindo que todo o conjunto de dados não constitua um ponto único de falha e que subconjuntos individuais do conjunto de dados possam ser gerenciados independentemente. Replicar partições que contêm dados críticos também pode aumentar a disponibilidade.

Ao criar e implementar partições, considere os seguintes fatores que afetam a disponibilidade:

* **O quanto os dados são fundamentais para as operações de negócios**. Alguns dados podem incluir informações comerciais críticas, como detalhes de faturas ou transações bancárias. Outros dados podem incluir dados operacionais menos críticos, como arquivos de log, rastreamentos de desempenho e assim por diante. Depois de identificar cada tipo de dados, considere:
  * Armazenar os dados críticos em partições altamente disponíveis com um plano de backup adequado.
  * Estabelecer mecanismos ou procedimentos separados de gerenciamento e monitoramento para os diferentes níveis de importância de cada conjunto de dados. Colocar os dados com o mesmo nível de importância na mesma partição para que eles possam ser armazenados juntos em backup com uma frequência adequada. Por exemplo, as partições que contêm dados de transações bancárias podem precisar de backup com mais frequência do que as partições que contêm informações de rastreamento ou registro em log.
* **Como as partições individuais podem ser gerenciadas**. Criar partições para dar suporte ao gerenciamento e manutenção independentes oferece várias vantagens. Por exemplo: 
  * Se uma partição falhar, ela pode ser recuperada independentemente sem afetar as instâncias de aplicativos que acessam dados em outras partições.
  * O particionamento de dados por área geográfica permite que as tarefas de manutenção agendadas ocorram fora dos horários de pico de cada local. Verifique se as partições não são muito grandes para evitar a conclusão de qualquer manutenção planejada durante esse período.
* **A possibilidade de replica dados críticos entre partições**. Essa estratégia pode melhorar a disponibilidade e o desempenho, embora também possa apresentar problemas de consistência. As alterações feitas nos dados em uma partição demoram para serem sincronizadas com cada réplica. Durante esse período, diferentes partições conterão valores de dados diferentes.

## <a name="understanding-how-partitioning-affects-design-and-development"></a>Noções básicas de como o particionamento afeta o design e o desenvolvimento
Usar o particionamento acrescenta complexidade ao design e desenvolvimento do sistema. Considere o particionamento como uma parte fundamental do design do sistema, mesmo que inicialmente o sistema contenha apenas uma única partição. Se você considerar o particionamento como uma ideia posterior, quando o sistema começar a sofrer problemas de desempenho e escalabilidade, a complexidade aumentará porque você já tem um sistema dinâmico para manter.

Se você atualizar o sistema para incorporar o particionamento nesse ambiente, será necessário modificar a lógica de acesso a dados. Isso também pode envolver a migração de grandes quantidade de dados existentes para distribui-los entre partições, muitas vezes enquanto os usuários esperam poder continuar usando o sistema.

Em alguns casos, o particionamento não é considerado importante, pois o conjunto de dados inicial é pequeno e pode ser facilmente manipulado por um único servidor. Isso pode acontecer em um sistema cuja escala não deve passar do tamanho inicial, mas muitos sistemas comerciais precisam expandir conforme o número de usuários aumenta. Essa expansão normalmente é acompanhada por um crescimento no volume de dados.

Também é importante entender que o particionamento nem sempre é uma função de grandes repositórios de dados. Por exemplo, um repositório de dados pequeno pode ser muito acessado por centenas de clientes simultâneos. Particionar os dados nessa situação pode ajudar a reduzir a contenção e melhorar a taxa de transferência.

Considere os seguintes pontos ao criar um esquema de particionamento de dados:

* **Sempre que possível, mantenha os dados para as operações mais comuns do banco de dados juntos em cada partição para minimizar as operações de acesso a dados entre partições**. Consultar entre partições pode ser mais demorado do que consultar apenas em uma única partição; porém, otimizar as partições para um conjunto de consultas pode afetar outros conjuntos de consultas. Quando não for possível evitar a consulta entre partições, minimize o tempo de consulta executando consultas paralelas e agregando os resultados no aplicativo. Talvez essa abordagem não seja possível em alguns casos, por exemplo, quando é necessário obter o resultado de uma consulta e usá-lo na próxima consulta.
* **Se as consultas usarem dados de referência relativamente estáticos, como tabelas de CEP ou listas de produtos, considere replicar esses dados em todas as partições para reduzir a necessidade de operações de pesquisa separadas em outras partições**. Essa abordagem também pode reduzir a probabilidade de que os dados de referência se tornem um conjunto de dados "mais acessado" sujeito a tráfego pesado de todo o sistema. No entanto, há um custo adicional associado à sincronização das alterações que pode ocorrer nesses dados de referência.
* **Sempre que possível, minimize os requisitos de integridade referencial entre partições verticais e funcionais**. Nesses esquemas, o próprio aplicativo é responsável por manter a integridade referencial entre partições quando os dados são atualizados e consumidos. As consultas que devem unir dados em várias partições são executadas mais lentamente do que as consultas que unem dados apenas na mesma partição, pois o aplicativo normalmente precisa executar consultas consecutivas com base em uma chave e em uma chave estrangeira. Em vez disso, considere replicar ou cancelar a normalização dos dados relevantes. Para minimizar o tempo de consulta onde são necessárias junções entre partições, execute consultas paralelas nas partições e reúna os dados no aplicativo.
* **Considere o efeito que o esquema de particionamento pode ter sobre a consistência dos dados nas partições.** Avalie se a coerência forte é realmente um requisito. Em vez disso, uma abordagem comum na nuvem é implementar a consistência eventual. Os dados em cada partição são atualizados separadamente, e a lógica de aplicativo garante que a atualizações sejam concluídas com êxito. Ela também trata das inconsistências que podem surgir da consulta de dados enquanto uma operação finalmente consistente está em execução. Para saber mais sobre como implementar a consistência eventual, confira [Data consistency primer].
* **Considere como as consultas localizam a partição correta**. Se uma consulta precisar verificar todas as partições para localizar os dados necessários, haverá um impacto significativo no desempenho, mesmo quando várias consultas paralelas estiverem em execução. As consultas que são usadas com as estratégias de particionamento vertical e funcional podem obviamente especificar as partições. No entanto, o particionamento horizontal (fragmentação) pode dificultar a localização de um item, pois cada fragmento tem o mesmo esquema. Uma solução típica de fragmentação é manter um mapa que pode ser usado para procurar a localização do fragmento para itens específicos de dados. Esse mapa pode ser implementado na lógica de fragmentação do aplicativo ou mantido pelo repositório de dados se a fragmentação transparente for permitida.
* **Ao usar uma estratégia de particionamento horizontal, considere o rebalanceamento periódico dos fragmentos**. Isso ajuda a distribuir os dados uniformemente por tamanho e carga de trabalho para minimizar os pontos de acesso, maximizar o desempenho da consulta e contornar as limitações de armazenamento físico. No entanto, isso é uma tarefa complexa que geralmente requer o uso de uma ferramenta ou um processo personalizado.
* **Se você replicar cada partição, ela fornecerá proteção adicional contra falhas**. Se uma única réplica falhar, as consultas podem ser direcionadas a uma cópia funcional.
* **Se você atingir os limites físicos de uma estratégia de particionamento, talvez seja necessário estender a escalabilidade para um nível diferente**. Por exemplo, se o particionamento estiver no nível do banco de dados, você poderá precisar localizar ou replicar partições em vários bancos de dados. Se o particionamento já estiver no nível do banco de dados e as limitações físicas forem um problema, isso pode significar que você precisa localizar ou replicar partições em várias contas de hospedagem.
* **Evite as transações que acessam os dados em várias partições**. Alguns repositórios de dados implementam consistência e integridade transacionais para operações que modificam dados, mas somente quando os dados estão localizados em uma única partição. Se precisar de suporte transacional em várias partições, você provavelmente precisará implementá-lo como parte da lógica do aplicativo, pois a maioria dos sistemas de particionamento não dão suporte nativo.

Todos os repositórios de dados exigem algumas atividades operacionais de gerenciamento e monitoramento. As tarefas podem variar de carregamento de dados, backup e restauração de dados a reorganização de dados, além de garantir que o sistema esteja funcionando correta e eficientemente.

Considere os seguintes fatores que afetam o gerenciamento operacional:

* **Como implementar tarefas apropriadas de gerenciamento e operacionais quando os dados são particionados**. Essas tarefas podem incluir backup e restauração, arquivamento de dados, monitoramento do sistema e outras tarefas administrativas. Por exemplo, manter a consistência lógica durante as operações de backup e restauração pode ser um desafio.
* **Como carregar os dados em várias partições e adicionar novos dados que estão chegando de outras fontes**. Alguns utilitários e ferramentas podem não permitir operações de dados fragmentadas, como carregamento de dados na partição correta. Isso significa que talvez você tenha que criar ou obter novos utilitários e ferramentas.
* **Como arquivar e excluir os dados regularmente**. Para impedir o aumento excessivo de partições, você precisa arquivar e excluir dados regularmente (talvez mensalmente). Pode ser necessário transformar os dados para correspondência com um esquema diferente de arquivamento.
* **Como localizar problemas de integridade de dados**. Considere a execução de um processo periódico para localizar quaisquer problemas de integridade de dados, como dados em uma partição que fazem referência a informações ausentes em outra. O processo também pode tentar corrigir esses problemas automaticamente ou acionar um alerta para um operador para corrigir os problemas manualmente. Por exemplo, em um aplicativo de comércio eletrônico, as informações de pedidos podem ser mantidas em uma única partição, mas os itens de linha que constituem cada pedido podem ser mantidos em outra. O processo de fazer um pedido exige a adição de dados a outras partições. Se esse processo falhar, poderá haver itens de linha armazenados para os quais não há nenhum pedido correspondente.

Diferentes tecnologias de armazenamento de dados normalmente fornecem seus próprios recursos para dar suporte ao particionamento. As seções a seguir resumem as opções que são implementadas pelos repositórios de dados usados frequentemente pelos aplicativos do Azure. Elas também descrevem considerações para criar aplicativos que podem aproveitar melhor esses recursos.

## <a name="partitioning-strategies-for-azure-sql-database"></a>Estratégias de particionamento para o Banco de Dados SQL do Azure
O Banco de Dados SQL do Azure é um banco de dados como serviço relacional executado na nuvem. Ele se baseia no Microsoft SQL Server. Um banco de dados relacional divide as informações em tabelas e cada tabela contém informações sobre as entidades como uma série de linhas. Cada linha contém colunas que contêm os dados dos campos individuais de uma entidade. A página [O que é o Banco de Dados SQL do Azure?] no site da Microsoft fornece a documentação detalhada sobre como criar e usar bancos de dados SQL.

## <a name="horizontal-partitioning-with-elastic-database"></a>Particionamento horizontal com Banco de Dados Elástico
Um banco de dados SQL individual tem um limite para volume de dados que ele pode conter. A taxa de transferência é restrita por fatores de arquitetura e pelo número de conexões simultâneas que são permitidas. O recurso Banco de Dados Elástico do Banco de Dados SQL dá suporte ao ajuste de escala horizontal para um banco de dados SQL. Usando o Banco de Dados Elástico, você pode particionar os dados em fragmentos que são distribuídos ente vários bancos de dados SQL. Também é possível adicionar ou remover fragmentos à medida que o volume de dados com o qual você precisa lidar aumenta e diminui. Usar o Banco de Dados Elástico também pode ajudar a reduzir a contenção pela distribuição da carga nos bancos de dados.

> [!NOTE]
> O Banco de Dados Elástico é uma substituição para o recurso Federações do Banco de Dados SQL do Azure. As instalações existentes da Federação do Banco de Dados SQL podem ser migradas para o Banco de Dados Elástico usando o utilitário de migração das Federações. Como alternativa, você pode implementar seu próprio mecanismo de fragmentação se o seu cenário não for naturalmente adequado para os recursos fornecidos pelo Banco de Dados Elástico.
>
>

Cada fragmento é implementado como um banco de dados SQL. Um fragmento pode conter mais de um conjunto de dados (referido como *shardlet*). Cada banco de dados mantém metadados que descrevem os shardlets que eles contêm. Um shardlet pode ser um único item de dados ou um grupo de itens que compartilham a mesma chave de shardlet. Por exemplo, se estiver fragmentando dados em um aplicativo multilocatário, a chave de shardlet poderá ser a ID do locatário e todos os dados de um determinado locatário serão mantidos como parte do mesmo shardlet. Os dados de outros locatários seriam mantidos em shardlets diferentes.

É responsabilidade do programador associar um conjunto de dados a uma chave de shardlet. Um banco de dados SQL separado age como um gerenciador global de mapa de fragmentos. Este banco de dados contém uma lista de todos os fragmentos e shardlets no sistema. Um aplicativo cliente que acessa dados se conecta primeiro ao banco de dados do gerenciador global de mapa de fragmentos para obter uma cópia do mapa de fragmentos (listando fragmentos e shardlets), que ele armazena em cache no local.

Em seguida, o aplicativo usa essas informações para encaminhar solicitações de dados para o fragmento adequado. Essa funcionalidade está oculta atrás de uma série de APIs contidas na Biblioteca de Cliente do Banco de Dados Elástico do Banco de Dados SQL do Azure, disponível como um pacote NuGet. A página [Visão geral dos recursos do Banco de Dados Elástico] no site da Microsoft fornece uma introdução mais abrangente ao Banco de Dados Elástico.

> [!NOTE]
> Você pode replicar o banco de dados do gerenciador global de mapa de fragmentos para reduzir a latência e melhorar a disponibilidade. Se implementar o banco de dados usando um dos tipos de preço Premium, você poderá configurar a replicação geográfica ativa para copiar os dados continuamente para os bancos de dados em diferentes regiões. Crie uma cópia do banco de dados em cada região em que os usuários se baseiam. Em seguida, configure seu aplicativo para se conectar a essa cópia e obter o mapa de fragmentos.
>
> Uma abordagem alternativa é usar a Sincronização de Dados SQL do Azure ou um pipeline do Azure Data Factory para replicar o banco de dados do gerenciador de mapa de fragmentos nas regiões. Essa forma de replicação é executada periodicamente e é mais adequada se o mapa de fragmentos não for alterado com frequência. Além disso, o banco de dados do gerenciador de mapa de fragmentos não precisa ser criado usando um tipo de preço Premium.
>
>

O Banco de Dados Elástico oferece dois esquemas para mapear dados para shardlets e armazená-los em fragmentos:

* Um **mapa de fragmentos da lista** descreve uma associação entre um shardlet e uma chave única. Por exemplo, em um sistema multilocatário, os dados de cada locatário podem ser associados a uma chave exclusiva e armazenados em seu próprio shardlet. Para garantir a privacidade e o isolamento (isto é, para evitar que um locatário esgote os recursos de armazenamento de dados disponíveis para outros locatários), cada shardlet pode ser mantido em seu próprio fragmento.

![Usando um mapa de fragmentos da lista para armazenar dados de locatário em fragmentos separados](./images/data-partitioning/PointShardlet.png)

*Figura 4. Usando um mapa de fragmentos da lista para armazenar dados de locatário em fragmentos separados*

* Um **mapa de fragmentos do intervalo** descreve uma associação entre um conjunto de valores de chave contíguos e um shardlet. No exemplo de sistema multilocatário descrito anteriormente, como alternativa à implementação de shardlets dedicados, você pode agrupar os dados de um conjunto de locatários (cada um com sua própria chave) no mesmo shardlet. Esse esquema é mais barato do que o primeiro (porque os locatários compartilham os recursos de armazenamento de dados), mas ele também cria o risco de reduzir a privacidade e o isolamento dos dados.

![Usando um mapa de fragmentos do intervalo para armazenar dados de um intervalo de locatários em um fragmento](./images/data-partitioning/RangeShardlet.png)

*Figura 5. Usando um mapa de fragmentos do intervalo para armazenar dados de um intervalo de locatários em um fragmento*

Observe que um único fragmento pode conter os dados de vários shardlets. Por exemplo, você pode usar os shardlets da lista para armazenar dados de diferentes locatários não contíguos no mesmo fragmento. Você também pode combinar shardlets do intervalo e shardlets da lista no mesmo fragmento, mesmo que eles sejam abordados por meio de diferentes mapas no banco de dados do gerenciador global de mapa de fragmentos. (O banco de dados do gerenciador global de mapa de fragmentos pode conter vários mapas de fragmentos). A Figura 6 ilustra essa abordagem.

![Implementando vários mapas de fragmentos](./images/data-partitioning/MultipleShardMaps.png)

*Figura 6. Implementando vários mapas de fragmentos*

O esquema de particionamento que você implementa pode ter uma influência significativa no desempenho do sistema. Ele também pode afetar a taxa na qual os fragmentos devem ser adicionados ou removidos, ou a taxa em que os dados devem ser reparticionados entre fragmentos. Considere os seguintes pontos ao usar o Banco de Dados Elástico para particionar dados:

* Agrupe dados que são usados juntos no mesmo fragmento e evite operações que precisam acessar os dados mantidos em vários fragmentos. Lembre-se de que com o Banco de Dados Elástico, um fragmento é um banco de dados SQL por si só e o Banco de Dados SQL do Azure não dá suporte a junções entre bancos de dados (que devem ser realizadas no lado do cliente). Não se esqueça de que no Banco de Dados SQL do Azure, restrições de integridade referencial, gatilhos e procedimentos armazenados em um banco de dados não podem fazer referência a objetos em outro. Portanto, não crie um sistema que tenha dependências entre fragmentos. No entanto, um banco de dados SQL pode conter tabelas que mantêm cópias de dados de referência frequentemente usados por consultas e outras operações. Essas tabelas não precisam pertencer a algum shardlet específico. Replicar esses dados nos fragmentos pode ajudar a eliminar a necessidade de unir os dados que se espalham pelos bancos de dados. O ideal é que esses dados sejam estáticos ou lentos, para minimizar o esforço de replicação e reduzir a probabilidade de eles se tornarem obsoletos.

  > [!NOTE]
  > Embora o Banco de Dados SQL não dê suporte a junções entre bancos de dados, você pode realizar consultas entre fragmentos com a API do Banco de Dados Elástico. Essas consultas podem ser iteradas de modo transparente por meio dos dados mantidos em todos os shardlets que são referenciados por um mapa de fragmentos. A API do Banco de Dados Elástico divide as consultas entre fragmentos em uma série de consultas individuais (uma para cada banco de dados) e mescla os resultados. Para obter mais informações, visite a página [Consulta de vários fragmentos] no site da Microsoft.
  >
  >
* Os dados armazenados em shardlets que pertencem ao mesmo mapa de fragmentos devem ter o mesmo esquema. Por exemplo, não crie um mapa de fragmentos da lista que aponte para alguns shardlets que contenham dados de locatários e para outros shardlets que contenham informações de produtos. Essa regra não é imposta pelo Banco de Dados Elástico, mas o gerenciamento de dados e a consulta se tornarão muito complexos se cada shardlet tiver um esquema diferente. No exemplo que acabou de ser citado, uma boa solução é criar dois mapas de fragmentos da lista: um que faça referência a dados locatários e outro que aponte para informações do produto. Lembre-se de que os dados pertencentes a diferentes shardlets podem ser armazenados no mesmo fragmento.

  > [!NOTE]
  > A funcionalidade de consulta entre fragmentos da API do Banco de Dados Elástico depende de cada shardlet no mapa de fragmentos que contém o mesmo esquema.
  >
  >
* As operações transacionais são permitidas apenas para dados que são mantidos no mesmo fragmento, e não entre fragmentos. As transações podem abranger shardlets, desde que façam parte do mesmo fragmento. Portanto, se a sua lógica de negócios precisar executar transações, armazene os dados afetados no mesmo fragmento ou implemente a consistência eventual. Para obter mais informações, consulte [Data consistency primer](Prévia de consistência de dados).
* Coloque os fragmentos próximos aos usuários que acessam os dados nesses fragmentos (em outras palavras, localize os fragmentos geograficamente). Essa estratégia ajuda a reduzir a latência.
* Evite ter uma combinação de fragmentos altamente ativos (pontos de acesso) e fragmentos relativamente inativos. Tente distribuir a carga uniformemente entre os fragmentos. Isso pode exigir o hash das chaves de shardlet.
* Se estiver localizando os fragmentos geograficamente, verifique se as chaves em hash são mapeadas para os shardlets mantidos em fragmentos armazenados próximo aos usuários que acessam esses dados.
* Atualmente, apenas um conjunto limitado de tipos de dados SQL tem suporte como chaves shardlet: *int, bigint, varbinary* e *uniqueidentifier*. Os tipos SQL *int* e *bigint* correspondem aos tipos de dados *int* e *long* no C# e têm os mesmos intervalos. O tipo SQL *varbinary* pode ser lidado usando uma matriz de *Bytes* no C# e o tipo SQL *uniqueidentier* corresponde à classe *Guid* no .NET Framework.

Como o nome indica, o Banco de Dados Elástico torna possível que um sistema adicione e remova fragmentos conforme o volume de dados é reduzido ou aumentado. As APIs na biblioteca de cliente do Banco de Dados Elástico do Banco de Dados SQL do Azure permitem que um aplicativo crie e exclua fragmentos dinamicamente (e atualize de modo transparente o gerenciador de mapa de fragmentos). No entanto, a remoção de um fragmento é uma operação destrutiva que também requer a exclusão de todos os dados nesse fragmento.

Se um aplicativo precisar dividir um fragmento em dois fragmentos separados ou combinar fragmentos, o Banco de Dados Elástico fornecerá um serviço separado de divisão/mesclagem. Esse serviço é executado em um serviço hospedado em nuvem (que deve ser criado pelo desenvolvedor) e migra os dados com segurança entre os fragmentos. Para obter mais informações, confira o tópico [Dimensionamento usando a ferramenta de divisão/mesclagem do Banco de Dados Elástico] no site da Microsoft.

## <a name="partitioning-strategies-for-azure-storage"></a>Estratégias de particionamento para o Armazenamento do Azure
O armazenamento do Azure fornece quatro abstrações para o gerenciamento de dados:

* O Armazenamento de Blobs armazena dados de objeto não estruturados. Um blob pode ser qualquer tipo de texto ou dados binários, como um documento, um arquivo de mídia ou um instalador do aplicativo. O Armazenamento de Blobs também é chamado de armazenamento de Objeto.
* O Armazenamento de Tabelas armazena conjuntos de dados estruturados. O Armazenamento de Tabela é um repositório de dados de atributo de chave NoSQL, que permite desenvolvimento rápido e acesso rápido a grandes quantidades de dados.
* O Armazenamento de Filas fornece um sistema confiável de mensagens para processamento de fluxo de trabalho e para comunicação entre componentes dos serviços de nuvem.
* O Armazenamento de Arquivos oferece o armazenamento compartilhado para aplicativos herdados com o protocolo SMB Standard. As máquinas virtuais e os serviços de nuvem do Azure podem compartilhar dados de arquivos entre componentes de aplicativos por meio de compartilhamentos montados, e os aplicativos locais podem acessar dados de arquivos em um compartilhamento por meio da API REST do serviço de Arquivo.

Basicamente, o armazenamento de tabelas e o armazenamento de blobs são repositórios de chave-valor otimizados para armazenar dados estruturados e não estruturados, respectivamente. As filas de armazenamento fornecem um mecanismo para criar aplicativos flexíveis e escalonáveis. O armazenamento de tabelas, o armazenamento de arquivos, o armazenamento de blobs e as filas de armazenamento são criados dentro do contexto de uma conta de armazenamento do Azure. As contas de armazenamento permitem três formas de redundância:

* **Armazenamento com redundância local**, que mantém três cópias de dados em um único datacenter. Essa forma de redundância fornece proteção contra falhas de hardware, mas não contra um desastre que abrange todo o datacenter.
* **Armazenamento com redundância de zona**, que mantém três cópias de dados distribuídas em diferentes datacenters na mesma região (ou em duas regiões geograficamente próximas). Essa forma de redundância pode fornecer proteção contra desastres que ocorrem em um único datacenter, mas não pode proteger contra desconexões de rede de grande escala que afetam uma região inteira. Observe que o armazenamento com redundância de zona está disponível apenas para blobs de blocos.
* **Armazenamento com redundância geográfica**, que mantém seis cópias de dados: três cópias em uma região (sua região local) e as outras três cópias em uma região remota. Essa forma de redundância fornece o nível mais alto de proteção contra desastres.

A Microsoft publicou alvos de escalabilidade para Armazenamento do Azure. Para obter mais informações, visite a página [Azure Storage Scalability and Performance Targets] no site da Microsoft. Atualmente, a capacidade da conta de armazenamento total não pode exceder 500 TB. (Isso inclui o tamanho dos dados que são mantidos no armazenamento de tabelas, o armazenamento de arquivos e no armazenamento de blobs, bem como mensagens pendentes que são mantidas na fila de armazenamento).

A taxa de solicitação máxima para uma conta de armazenamento (supondo um tamanho de entidade, blob ou mensagem de 1 KB) é de 20.000 solicitações por segundo. Uma conta de armazenamento tem, no máximo, 1000 IOPS (8 KB de tamanho) por compartilhamento de arquivos. Se seu sistema provavelmente excede esses limites, considere o particionamento da carga entre várias contas de armazenamento. Uma única assinatura do Azure pode criar até 200 contas de armazenamento. No entanto, observe que esses limites podem ser alterados com o tempo.

## <a name="partitioning-azure-table-storage"></a>Particionamento do armazenamento de tabela do Azure
O armazenamento de tabelas do Azure é um repositório de chave/valor que foi desenvolvido em torno do particionamento. Todas as entidades são armazenadas em uma partição e as partições são gerenciadas internamente pelo armazenamento de tabela do Azure. Cada entidade armazenada em uma tabela deve fornecer uma chave de duas partes que inclui:

* **A chave de partição**. Esse é um valor de cadeia de caracteres que determina em qual partição o armazenamento de tabelas do Azure colocará a entidade. Todas as entidades com a mesma chave de partição serão armazenadas na mesma partição.
* **A chave de linha**. Este é outro valor de cadeia de caracteres que identifica a entidade na partição. Todas as entidades em uma partição são classificadas lexicalmente, em ordem crescente, por essa chave. A combinação de chave de linha/chave de partição deve ser exclusiva para cada entidade e não pode exceder 1 KB.

O restante dos dados de uma entidade consiste em campos definidos pelo aplicativo. Nenhum esquema específico é imposto e cada linha pode conter um conjunto diferente de campos definidos pelo aplicativo. A única limitação é que o tamanho máximo de uma entidade (incluindo as chaves de partição e de linha) atualmente é de 1 MB. O tamanho máximo de uma tabela é de 200 TB, embora esses números possam ser alterados no futuro. (Verifique a página [Azure Storage Scalability and Performance Targets] no site da Microsoft para obter as informações mais recentes sobre esses limites).

Se você estiver tentando armazenar entidades que excedem essa capacidade, considere dividi-las em várias tabelas. Use o particionamento vertical para dividir os campos nos grupos que têm mais probabilidade de serem acessados juntos.

A Figura 7 mostra a estrutura lógica de um exemplo de conta de armazenamento (Dados da Contoso) para um aplicativo de comércio eletrônico fictício. A conta de armazenamento contém três tabelas: Informações de clientes, Informações de produtos e Informações de pedidos. Cada tabela tem várias partições.

Na tabela Informações de clientes, os dados são particionados de acordo com a cidade em que o cliente está localizado e a chave de linha contém a ID do cliente. Na tabela Informações de produtos, os produtos são particionados por categoria de produto e a chave de linha contém o número do produto. Na tabela Informações de pedidos, os pedidos são particionados pela data de realização e a chave de linha especifica o horário em que o pedido foi recebido. Observe que todos os dados são ordenados pela chave de linha em cada partição.

![As tabelas e partições em um exemplo de conta de armazenamento](./images/data-partitioning/TableStorage.png)

*Figura 7. As tabelas e partições em um exemplo de conta de armazenamento*

> [!NOTE]
> O armazenamento de tabela do Azure também adiciona um campo de carimbo de data/hora a cada entidade. O campo de carimbo de data/hora é mantido pelo armazenamento de tabela e é atualizado sempre que a entidade é modificada e gravada de volta em uma partição. O serviço de armazenamento de tabelas usa esse campo para implementar a simultaneidade otimista. (Cada vez que um aplicativo grava uma entidade volta no armazenamento de tabelas, o serviço de armazenamento de tabelas compara o valor do carimbo de data/hora na entidade que está sendo gravada com o valor que é mantido no armazenamento de tabelas. Se os valores forem diferentes, isso significa que outro aplicativo deve ter modificado a entidade desde que ela foi recuperada pela última vez, e a operação de gravação falha. Não modifique esse campo no seu próprio código e não especifique um valor para esse campo ao criar uma nova entidade.
>
>

O armazenamento de tabela do Azure usa a chave de partição para determinar como os dados são armazenados. Se uma entidade for adicionada a uma tabela com uma chave de partição não usada anteriormente, o armazenamento de tabelas do Azure criará uma nova partição para esta entidade. Outras entidades com a mesma chave de partição serão armazenadas na mesma partição.

Esse mecanismo implementa de modo efetivo uma estratégia de escala horizontal automática. Cada partição é armazenada em um único servidor em um datacenter do Azure para ajudar a garantir que as consultas que recuperam dados de uma única partição sejam executadas rapidamente. No entanto, diferentes partições podem ser distribuídas entre vários servidores. Além disso, um único servidor pode hospedar várias partições se essas partições tiverem um tamanho limitado.

Considere os seguintes pontos ao criar entidades para o armazenamento de tabelas do Azure:

* A seleção dos valores da chave de linha e da chave de partição deve ser orientada pela maneira como os dados são acessados. Escolha uma combinação de chave de linha/chave de partição que seja compatível com a maioria de suas consultas. As consultas mais eficientes recuperam dados especificando a chave de partição e a chave de linha. As consultas que especificam uma chave de partição e um intervalo de chaves de linha podem ser concluídas por meio da verificação de uma única partição. Isso é relativamente rápido, pois os dados são mantidos na ordem da chave de linha. Se as consultas não especificarem qual partição verificar, a chave de partição poderá exigir que o armazenamento de tabelas do Azure verifique cada partição dos seus dados.

  > [!TIP]
  > Se uma entidade tiver uma chave natural, use-a como a chave de partição e especifique uma cadeia de caracteres vazia como a chave de linha. Se uma entidade tiver uma chave composta que consiste em duas propriedades, selecione a propriedade com alteração mais lenta como a chave de partição e a outra como a chave de linha. Se uma entidade tiver mais de duas propriedades de chave, use uma concatenação de propriedades para fornecer as chaves de partição e de linha.
  >
  >
* Se você executa regularmente consultas que pesquisam dados usando campos que não sejam as chaves de partição e de linha, considere a implementação do [Index Table Pattern].
* Se você gerar chaves de partição usando uma sequência monotônica crescente ou decrescente (como "0001", "0002", "0003" e assim por diante) e cada partição contiver apenas uma quantidade limitada de dados, o armazenamento de tabelas do Azure poderá agrupar fisicamente essas partições no mesmo servidor. Esse mecanismo pressupõe que o aplicativo tem mais probabilidade de executar consultas em um intervalo contíguo de partições (consultas do intervalo) e é otimizado para esse caso. No entanto, essa abordagem pode levar a pontos de acesso focados em um único servidor, pois todas as inserções das novas entidades provavelmente estão concentradas em uma ou na outra extremidade dos intervalos contíguos. Ela também pode reduzir a escalabilidade. Para distribuir a carga mais uniformemente entre os servidores, considere o hash da partição para tornar a sequência mais aleatória.
* O armazenamento de tabela do Azure dá suporte a operações transacionais para entidades que pertencem à mesma partição. Isso significa que um aplicativo pode executar várias operações de inserção, atualização, exclusão, substituição ou mesclagem como uma unidade atômica (desde que a transação não inclua mais de 100 entidades e que a carga da solicitação não exceda 4 MB). As operações que abrangem várias partições não são transacionais e podem exigir que você implemente a consistência eventual, conforme descrito em [Data consistency primer](Prévia de consistência de dados). Para obter mais informações sobre o armazenamento de tabelas e transações, visite a página [Performing entity group transactions] (Executando transações do grupo de entidades) no site da Microsoft.
* Dê atenção especial à granularidade da chave de partição pelos seguintes motivos:
  * Usar a mesma chave de partição para todas as entidades faz com que o serviço de armazenamento de tabelas crie uma única partição grande que é mantida em um servidor. Isso o impede de escalar horizontalmente e concentra a carga em um único servidor. Como resultado, essa abordagem só é adequada para sistemas que gerenciam um número pequeno de entidades. No entanto, essa abordagem garante que todas as entidades possam participar das transações do grupo de entidade.
  * Usar uma chave de partição exclusiva para todas as entidades faz com que o serviço de armazenamento de tabelas crie uma partição separada para cada entidade, possivelmente, resultando em um grande número de partições pequenas (dependendo do tamanho das entidades). Essa abordagem é mais escalonável do que usar uma chave de partição única, mas as transações de grupo da entidade não são possíveis. Além disso, as consultas que buscam mais de uma entidade podem envolver a leitura em mais de um servidor. No entanto, se o aplicativo executar consultas do intervalo, o uso de uma sequência monotônica para gerar as chaves de partição pode ajudar a otimizar essas consultas.
  * Compartilhar a chave da partição entre um subconjunto de entidades torna possível o agrupamento de entidades relacionadas na mesma partição. As operações que envolvem entidades relacionadas podem ser executadas usando transações de grupo da entidade, e as consultas que buscam um conjunto de entidades relacionadas podem ser atendidas acessando um único servidor.

Para saber mais sobre como particionar dados no armazenamento de tabelas do Azure, confira o artigo [Azure Storage Table Design Guide] no site da Microsoft.

## <a name="partitioning-azure-blob-storage"></a>Particionamento do armazenamento de blobs do Azure
O armazenamento de blobs do Azure possibilita manter objetos binários grandes; atualmente até 5 TB para blobs de blocos ou 1 TB para blobs de páginas. (Para obter as informações mais recentes, acesse a página [Azure Storage Scalability and Performance Targets] no site da Microsoft.) Use blobs de blocos em cenários como streaming em que é necessário carregar ou baixar grandes volumes de dados rapidamente. Use blobs de páginas para aplicativos que exigem o acesso aleatório em vez do acesso serial a partes dos dados.

Cada blob (de blocos ou páginas) é mantido em um contêiner em uma conta de armazenamento do Azure. Você pode usar contêineres para agrupar blobs relacionados que tenham os mesmos requisitos de segurança. Esse agrupamento é lógico em vez de físico. Dentro de um contêiner, cada blob tem um nome exclusivo.

A chave de partição de um blob é o nome da conta + nome do contêiner + nome do blob. Isso significa que cada blob poderá ter sua própria partição se a carga no blob exigir isso. BLOBs podem ser distribuídos em vários servidores para escalar horizontalmente o acesso a eles, mas um único blob só pode ser atendido por um único servidor. 

As ações de gravação de um único bloco (blob de blocos) ou uma página (blob de páginas) são atômicas, mas as operações que abrangem blobs, páginas ou blocos não. Se precisar garantir a consistência ao executar operações de gravação em blocos, páginas e blobs, remova um bloco de gravação usando uma concessão de blob.

O armazenamento de blobs do Azure visa taxas de transferência de até 60 MB por segundo ou 500 solicitações por segundo para cada blob. Se você prevê que esses limites serão excedidos e os dados do blob forem relativamente estáticos, considere replicar os blobs usando a Rede de Distribuição de Conteúdo do Azure. Para saber mais, veja a página [Rede de Distribuição de Conteúdo do Azure] no site da Microsoft. Para obter mais instruções e considerações, confira [Using Azure Content Delivery Network].

## <a name="partitioning-azure-storage-queues"></a>Particionamento de filas de armazenamento do Azure
As filas de armazenamento do Azure permitem implementar mensagens assíncronas entre processos. Uma conta de armazenamento do Azure pode conter um número ilimitado de filas e cada fila pode conter um número ilimitado de mensagens. A única limitação é o espaço disponível na conta de armazenamento. O tamanho máximo de uma mensagem individual é de 64 KB. Se precisar de mensagens com tamanhos maiores do que esse, considere usar as filas do Barramento de Serviço do Azure.

Cada fila de armazenamento tem um nome exclusivo dentro da conta de armazenamento na qual ela está contida. O Azure particiona filas com base no nome. Todas as mensagens para a mesma fila são armazenadas na mesma partição, que é controlada por um único servidor. Diferentes filas podem ser gerenciadas por diferentes servidores para ajudar a balancear a carga. A alocação de filas para servidores é transparente para aplicativos e usuários.

 Em um aplicativo de grande escala, não use a mesma fila de armazenamento para todas as instâncias do aplicativo, pois essa abordagem pode fazer com que o servidor que está hospedando a fila se torne um ponto de acesso. Em vez disso, use filas diferentes para diferentes áreas funcionais do aplicativo. As filas de armazenamento do Azure não dão suporte a transações; portanto, direcionar mensagens para diferentes filas deve ter pouco impacto na consistência das mensagens.

Uma fila de armazenamento do Azure pode manipular até 2.000 mensagens por segundo.  Se precisar processar mensagens a uma taxa maior que essa, considere criar várias filas. Por exemplo, em um aplicativo global, crie filas de armazenamento separadas em contas de armazenamento separadas para manipular instâncias do aplicativo que estão sendo executadas em cada região.

## <a name="partitioning-strategies-for-azure-service-bus"></a>Estratégias de particionamento para o Barramento de Serviço do Azure
O Barramento de Serviço do Azure usa um agente de mensagem para manipular as mensagens que são enviadas para uma fila ou um tópico do Barramento de Serviço. Por padrão, todas as mensagens que são enviadas para uma fila ou um tópico são manipuladas pelo mesmo processo de agente de mensagem. Essa arquitetura pode colocar uma limitação na taxa de transferência geral da fila de mensagens. No entanto, você também pode particionar a uma fila ou um tópico quando ele é criado. Você pode fazer isso definindo a propriedade *EnablePartitioning* da fila ou da descrição do tópico para *true*.

Uma fila ou um tópico particionado é dividido em vários fragmentos, e cada um deles tem o suporte de um repositório e agente de mensagem separados. O Barramento de Serviço assume a responsabilidade pela criação e gerenciamento desses fragmentos. Quando um aplicativo publica uma mensagem em uma fila ou tópico particionado, o Barramento de Serviço atribui a mensagem a um fragmento dessa fila ou tópico. Quando um aplicativo recebe uma mensagem de uma fila ou assinatura, o Barramento de Serviço verifica cada fragmento em relação à próxima mensagem disponível e, em seguida, transmite-a para o aplicativo para o processamento.

Essa estrutura ajuda a distribuir a carga entre agentes e repositórios de mensagem, aumentando a escalabilidade e melhorando a disponibilidade. Se o repositório ou o agente de mensagem para um fragmento estiver temporariamente indisponível, o Barramento de Serviço poderá recuperar mensagens de um dos fragmentos restantes disponíveis.

O Barramento de Serviço atribui uma mensagem a um fragmento da seguinte maneira:

* Se a mensagem pertencer a uma sessão, todas as mensagens com o mesmo valor para a propriedade *SessionId* são enviadas para o mesmo fragmento.
* Se a mensagem não pertencer a uma sessão, mas o remetente especificou um valor para a propriedade *PartitionKey*, então, todas as mensagens com o mesmo valor *PartitionKey* serão enviadas para o mesmo fragmento.

  > [!NOTE]
  > Se as propriedades *SessionId* e *PartitionKey* forem especificadas, então, deverão ser definidas para o mesmo valor ou a mensagem será rejeitada.
  >
  >
* Se as propriedades *SessionId* e *PartitionKey* de uma mensagem não forem especificadas, mas a detecção de duplicidades estiver habilitada, a propriedade *MessageId* será usada. Todas as mensagens com a mesma *MessageId* serão direcionadas para o mesmo fragmento.
* Se as mensagens não incluírem uma propriedade *SessionId, PartitionKey* ou *MessageId*, então, o Barramento de Serviço atribuirá as mensagens aos fragmentos em sequência. Se um fragmento estiver indisponível, o Barramento de Serviço passará para o próximo. Isso significa que uma falha temporária na infraestrutura das mensagens não causa uma falha na operação de envio de mensagem.

Considere os seguintes pontos ao decidir se ou como particionar uma fila ou um tópico de mensagem do Barramento de Serviço:

* Tópicos e filas do Barramento de Serviço são criados dentro do escopo de um namespace do Barramento de Serviço. O Barramento de Serviço atualmente permite até 100 filas ou tópicos particionados por namespace.
* Cada namespace do Barramento de Serviço impõe cotas nos recursos disponíveis, como o número de assinaturas por tópico, o número de solicitações simultâneas de envio e recebimento por segundo e o número máximo de conexões simultâneas que podem ser estabelecidas. Essas cotas estão documentadas no site da Microsoft, na página [Cotas do Barramento de Serviço]. Se você prevê que esses valores serão excedidos, crie namespaces adicionais com suas próprias filas e tópicos e distribua o trabalho entre esses namespaces. Por exemplo, em um aplicativo global, crie namespaces separados em cada região e configure instâncias do aplicativo para usar as filas e tópicos no namespace mais próximo.
* As mensagens enviadas como parte de uma transação devem especificar uma chave de partição. Isso pode ser uma propriedade *SessionId*, *PartitionKey* ou *MessageId*. Todas as mensagens enviadas como parte da mesma transação devem especificar a mesma chave de partição, pois elas devem ser manipuladas pelo mesmo processo de agente de mensagem. Você não pode enviar mensagens para diferentes filas ou tópicos na mesma transação.
* As filas e os tópicos particionados não podem ser configurados para serem excluídos automaticamente quando se tornam ociosos.
* Atualmente, as filas e os tópicos particionados não podem ser usados com o AMQP (Advanced Message Queuing Protocol) se você estiver criando soluções híbridas ou entre plataformas.

## <a name="partitioning-strategies-for-cosmos-db"></a>Estratégias de particionamento para o Cosmos DB

O Azure Cosmos DB é um banco de dados NoSQL pode armazenar documentos JSON usando a [API de SQL do Azure Cosmos DB][cosmosdb-sql-api]. Um documento no banco de dados do Cosmos DB é uma representação serializada pelo JSON de um objeto ou outro dado. Nenhum esquema fixo é imposto, exceto pelo fato de que cada documento deve conter uma ID exclusiva.

Os documentos são organizados em coleções. Você pode agrupar documentos relacionados em uma coleção. Por exemplo, em um sistema que mantém postagens de blog, você pode armazenar o conteúdo de cada postagem de blog como um documento em uma coleção. Também é possível criar coleções para cada tipo de assunto. Como alternativa, em um aplicativo multilocatário, como um sistema em que diferentes autores controlam e gerenciam suas postagens de blog, você pode particionar os blogs por autor e criar coleções separadas para cada um deles. O espaço de armazenamento que é alocado às coleções é flexível e pode diminuir ou aumentar conforme necessário.

O Cosmos DB suporta o particionamento automático de dados baseados numa chave de partição definida pelo aplicativo. Uma *partição lógica* é uma partição que armazena todos os dados para um valor de chave de partição única. Todos os documentos que compartilham o mesmo valor para a chave de partição são colocados na mesma partição lógica. O Cosmos DB distribui valores de acordo com o hash da chave de partição. Uma partição lógica tem um tamanho máximo de 10 GB. Portanto, a escolha da chave de partição é uma decisão importante no momento do design. Escolha uma propriedade com um amplo intervalo de valores e padrões de acesso regular. Para obter mais informações, veja [Partição e redução horizontal no Azure Cosmos DB](/azure/cosmos-db/partition-data).

> [!NOTE]
> Cada banco de dados do Cosmos DB tem um *nível de desempenho* que determina a quantidade de recursos que ele obtém. Um nível de desempenho é associado a um limite de taxa da *unidade de solicitação* (RU). O limite de taxa da RU especifica o volume de recursos que é reservado e disponibilizado para uso exclusivo dessa coleção. O custo de uma coleção depende do nível de desempenho que é selecionado para a coleção. Quanto maior o nível de desempenho (e o limite de taxa da RU), maior o custo. É possível ajustar o nível de desempenho de uma coleção usando o portal do Azure. Para obter mais informações, confira [Unidades de Solicitação no Azure Cosmos DB][cosmos-db-ru].
>
>

Se o mecanismo de particionamento que o Cosmos DB fornece não for suficiente, pode precisar de fragmentar os dados ao nível do aplicativo. As coleções de documentos fornecem um mecanismo natural para particionar dados em um banco de dados individual. A maneira mais simples de implementar a fragmentação é criar uma coleção para cada fragmento. Os contêineres são recursos lógicos e podem abranger um ou mais servidores. Contêineres de tamanho fixo têm um limite máximo de 10 GB e taxa de transferência de 10.000 RU/s. Os contêineres ilimitados não têm um tamanho de armazenamento máximo, mas têm que especificar uma chave de partição. Com a fragmentação do aplicativo, o aplicativo cliente deve direcionar as solicitações ao fragmento adequado, normalmente pela implementação do seu próprio mecanismo de mapeamento com base em alguns atributos dos dados que definem a chave de fragmento. 

Todos os bancos de dados são criados no contexto de uma conta de banco de dados do Cosmos DB. Uma única conta pode conter vários bancos de dados e especifica em qual região os bancos de dados são criados. Cada conta também impõe seu próprio controle de acesso. Você pode usar contas do Cosmos DB para localizar os fragmentos geograficamente (coleções nos bancos de dados) próximos aos usuários que precisam acessá-los e impor restrições para que somente esses usuários possam se conectar a eles.

Considere os seguintes pontos ao decidir como particionar dados com a API do SQL do Cosmos DB:

* **Os recursos disponíveis para um banco de dados do Cosmos DB estão sujeitos às limitações de cota da conta**. Cada banco de dados pode conter um número de coleções e cada coleção é associada a um nível de desempenho que rege o limite de taxa da RU (produtividade reservada) para essa coleção. Para saber mais, confira [Assinatura e limites de serviço, cotas e restrições do Azure][azure-limits].
* **Cada documento deve ter um atributo que pode ser usado para identificar exclusivamente o documento dentro da coleção na qual ele é mantido**. Esse atributo é diferente da chave de fragmento, que define qual coleção mantém o documento. Uma coleção pode conter um grande número de documentos. Teoricamente, ela é limitada apenas pelo tamanho máximo da ID do documento. A ID do documento pode ter até 255 caracteres.
* **Todas as operações em um documento são realizadas dentro do contexto de uma transação. As transações estão no escopo da coleção em que o documento está contido.** Se uma operação falhar, o trabalho realizado será revertido. Quando um documento está sujeito a uma operação, todas as alterações feitas estão sujeitas ao isolamento no nível do instantâneo. Esse mecanismo garante que se, por exemplo, houver uma falha em uma solicitação para a criação de um novo documento, outro usuário que estiver consultando o banco de dados simultaneamente não verá um documento parcial que será então removido.
* **As consultas do banco de dados também estão dentro do escopo do nível da coleção**. Uma única consulta pode recuperar dados somente de uma coleção. Se precisar recuperar dados de várias coleções, você deverá consultar cada coleção individualmente e mesclar os resultados no código do aplicativo.
* **O Cosmos DB oferece suporte aos itens programáveis que podem ser armazenados em uma coleção com os documentos**. Isso inclui procedimentos armazenados, funções definidas pelo usuário e gatilhos (escritos em JavaScript). Esses itens podem acessar qualquer documento na mesma coleção. Além disso, esses itens são executados dentro do escopo da transação de ambiente (no caso de um gatilho que é acionado como resultado de uma operação de criação, exclusão ou substituição executada em um documento) ou por iniciar uma nova transação (no caso de um procedimento armazenado executado como resultado de uma solicitação de cliente explícita). Se o código em um item programável lançar uma exceção, a transação será revertida. Você pode usar procedimentos armazenados e gatilhos para manter a integridade e a consistência entre documentos, mas esses documentos devem fazer parte da mesma coleção.
* **Provavelmente, as coleções que você pretende manter nos bancos de dados não excederão os limites de taxa de transferência definidos pelos níveis de desempenho das coleções**. Para obter mais informações, confira [Unidades de Solicitação no Azure Cosmos DB][cosmos-db-ru]. Se você prevê que esses limites serão atingidos, considere dividir as coleções em bancos de dados em diferentes contas para reduzir a carga por coleção.

## <a name="partitioning-strategies-for-azure-search"></a>Estratégias de particionamento para o Azure Search
A capacidade de pesquisar dados geralmente é o principal método de navegação e exploração fornecido por muitos aplicativos Web. Ela ajuda os usuários a encontrar recursos rapidamente (por exemplo, produtos em um aplicativo de comércio eletrônico) com base em combinações de critérios de pesquisa. O serviço de Azure Search fornece recursos de pesquisa de texto completo em relação ao conteúdo da Web e inclui recursos como consultas recomendadas de preenchimento automático com base em correspondências aproximadas e na navegação facetada. Uma descrição completa desses recursos está disponível na página [O que é o Azure Search?] no site da Microsoft.

O Azure Search armazena conteúdo pesquisável como documentos JSON em um banco de dados. Defina os índices que especificam os campos pesquisáveis nesses documentos e forneça essas definições ao Azure Search. Quando um usuário envia uma solicitação de pesquisa, o Azure Search usa os índices adequados para localizar itens correspondentes.

Para reduzir a contenção, o armazenamento usado pelo Azure Search pode ser dividido em 1, 2, 3, 4, 6 ou 12 partições, e cada partição pode ser replicada até seis vezes. O produto do número de partições multiplicado pelo número de réplicas é denominado *unidade de pesquisa* (SU). Uma única instância do Azure Search pode conter um máximo de 36 SUs (um banco de dados com 12 partições somente dá suporte a um máximo de 3 réplicas).

Você será cobrado por cada SU alocada ao seu serviço. Conforme o volume de conteúdo pesquisável ou a taxa de solicitações de pesquisa aumenta, é possível adicionar SUs a uma instância existente do Azure Search para manipular a carga extra. O Azure Search em si distribui os documentos uniformemente entre as partições. Atualmente, nenhuma estratégia de particionamento manual é compatível.

Cada partição pode conter um máximo de 15 milhões de documentos ou ocupar 300 GB de espaço de armazenamento (o que for menor). Você pode criar até 50 índices. O desempenho do serviço varia e depende da complexidade dos documentos, dos índices disponíveis e dos efeitos da latência de rede. Em média, uma única réplica (1 SU) deve ser capaz de manipular 15 QPS (consultas por segundo), embora seja recomendável avaliar o desempenho com seus próprios dados para obter uma medida mais precisa da taxa de transferência. Para saber mais, confira a página [Limites de serviço no Azure Search] no site da Microsoft.

> [!NOTE]
> Você pode armazenar um conjunto limitado de tipos de dados em documentos pesquisáveis, incluindo cadeias de caracteres, boolianos, dados numéricos, dados de data/hora e alguns dados geográficos. Para obter mais detalhes, visite a página [Tipos de Dados com Suporte (Azure Search)] no site da Microsoft.
>
>

Você tem controle limitado sobre como o Azure Search particiona os dados para cada instância do serviço. No entanto, em um ambiente global, é possível melhorar o desempenho e reduzir ainda mais a latência e contenção pelo particionamento do próprio serviço usando qualquer uma das seguintes estratégias:

* Crie uma instância do Azure Search em cada região geográfica e verifique se os aplicativos cliente são direcionados à instância disponível mais próxima. Essa estratégia exige que todas as atualizações de conteúdo pesquisável sejam replicadas pontualmente em todas as instâncias do serviço.
* Crie duas camadas de Azure Search:

  * Um serviço local em cada região que contenha os dados que são acessados com mais frequência pelos usuários nessa região. Os usuários podem direcionar solicitações para obter resultados rápidos, mas limitados.
  * Um serviço global que inclua todos os dados. Os usuários podem direcionar solicitações para obter resultados mais demorados, porém, mais completos.

Essa abordagem é mais adequada quando há uma variação regional significativa nos dados que estão sendo pesquisados.

## <a name="partitioning-strategies-for-azure-redis-cache"></a>Estratégias de particionamento para o Cache Redis do Azure
O Cache Redis do Azure fornece um serviço de cache compartilhado na nuvem que se baseia no repositório de dados de chave/valor do Redis. Como o nome sugere, o Cache Redis do Azure serve como uma solução de cache. Use-o apenas para dados manter dados transitórios, e não como um repositório de dados permanentes. Os aplicativos que utilizam o Cache Redis do Azure devem poder continuar funcionando se o cache ficar indisponível. O Cache Redis do Azure permite a replicação primária/secundária para fornecer alta disponibilidade, mas atualmente limita o tamanho máximo do cache para 53 GB. Se você precisar de mais espaço, é necessário criar caches adicionais. Para saber mais, visite a página [Cache Redis do Azure] no site da Microsoft.

O particionamento de um repositório de dados do Redis envolve a divisão dos dados entre instâncias do serviço do Redis. Cada instância constitui uma única partição. O Cache Redis do Azure abstrai os serviços do Redis por trás de uma fachada e não os expõe diretamente. A maneira mais simples de implementar o particionamento é criar várias instâncias do Cache Redis do Azure e distribuir os dados entre eles.

Você pode associar cada item de dados a um identificador (uma chave de partição) que especifica qual cache armazena o item de dados. A lógica do aplicativo cliente pode usar esse identificador para rotear solicitações para a partição adequada. Esse esquema é muito simples, mas se o esquema de particionamento for alterado (por exemplo, se instâncias adicionais do Cache Redis do Azure forem criadas), os aplicativos cliente precisarão ser reconfigurados.

O Redis nativo (não o Cache Redis do Azure) dá suporte ao particionamento do lado do servidor com base no clustering do Redis. Nessa abordagem, é possível dividir os dados uniformemente entre os servidores usando um mecanismo de hash. Cada servidor do Redis armazena metadados que descrevem o intervalo de chaves de hash contidos na partição, além de conter informações sobre quais chaves de hash estão localizadas nas partições em outros servidores.

Os aplicativos cliente simplesmente enviam solicitações para qualquer um dos servidores Redis participantes (provavelmente o mais próximo). O servidor Redis examina a solicitação do cliente. Se ela puder ser resolvida localmente, o servidor executa a operação solicitada. Caso contrário, ele encaminha a solicitação para o servidor adequado.

Esse modelo é implementado usando o clustering do Redis e é descrito mais detalhadamente na página [tutorial de cluster do Redis] no site do Redis. O clustering do Redis é transparente para os aplicativos cliente. Outros servidores do Redis podem ser adicionados ao cluster (e os dados podem ser particionados novamente) sem a necessidade de reconfigurar os clientes.

> [!IMPORTANT]
> Atualmente, o Cache Redis do Azure não dá suporte ao clustering do Redis. Se desejar implementar essa abordagem com o Azure, será necessário implementar seus próprios servidores do Redis instalando o Redis em um conjunto de máquinas virtuais do Azure e configurando-as manualmente. A página [Running Redis on a CentOS Linux VM in Azure] , no site da Microsoft, explica um exemplo que mostra como criar e configurar um nó do Redis em execução como uma VM do Azure.
>
>

A página [Partitioning: how to split data among multiple Redis instances] , no site do Redis, fornece mais informações sobre a implementação do particionamento com o Redis. Ao ler o restante desta seção, pressupomos que você esteja implementando o particionamento do lado do cliente ou assistido por proxy.

Considere os seguintes pontos ao decidir como particionar dados com o Cache Redis do Azure:

* O Cache Redis do Azure não pretende agir como um repositório de dados permanente; portanto, seja qual for o esquema de particionamento que você implemente, o código do aplicativo deverá poder recuperar dados de um local que não seja o cache.
* Os dados que são acessados juntos frequentemente devem ser mantidos na mesma partição. O Redis é um poderoso repositório de chave/valor que fornece vários mecanismos altamente otimizados para estruturação de dados. Esses mecanismos podem ser um dos seguintes:

  * Cadeias de caracteres simples (dados binários de até 512 MB)
  * Tipos de agregação, como listas (que podem atuar como filas e pilhas)
  * Conjuntos (ordenados e não ordenados)
  * Hashes (que podem agrupar campos relacionados, como os itens que representam os campos em um objeto)
* Os tipos de agregação permitem associar vários valores relacionados com a mesma chave. Uma chave do Redis identifica uma lista, um conjunto ou um hash em vez de itens de dados que ela contém. Esses tipos estão disponíveis com o Cache Redis do Azure e são descritos na página [Data types] no site do Redis. Por exemplo, em uma parte de um sistema de comércio eletrônico que rastreia os pedidos que são feitos pelos clientes, os detalhes de cada cliente podem ser armazenados em um hash do Redis que é digitado usando a ID do cliente. Cada hash pode manter uma coleção de IDs de pedido do cliente. Um conjunto separado do Redis pode manter os pedidos, novamente estruturados como hashes, e digitados com o uso da ID de pedido. A Figura 8 mostra essa estrutura. Observe que o Redis não implementa nenhuma forma de integridade referencial; portanto, é responsabilidade do desenvolvedor manter os relacionamentos entre clientes e pedidos.

![Estrutura recomendada no armazenamento do Redis para registrar pedidos de clientes e seus detalhes](./images/data-partitioning/RedisCustomersandOrders.png)

*Figura 8. Estrutura recomendada no armazenamento do Redis para registrar pedidos de clientes e seus detalhes*

> [!NOTE]
> No Redis, todas as chaves são valores de dados binários (como cadeias de caracteres do Redis) e podem conter até 512 MB de dados. Teoricamente, uma chave pode conter quase todo tipo de informação. No entanto, é recomendável adotar uma convenção de nomenclatura consistente para chaves que descrevam o tipo de dados e identifiquem a entidade, mas que não seja excessivamente longa. Uma abordagem comum é usar chaves na forma "entity_type: ID". Por exemplo, você pode usar "customer:99" para indicar a chave de um cliente com a ID 99.
>
>

* Você pode implementar o particionamento vertical armazenando informações relacionadas em diferentes agregações no mesmo banco de dados. Por exemplo, em um aplicativo de comércio eletrônico, você pode armazenar as informações frequentemente acessadas sobre produtos em um hash do Redis e as informações detalhadas usadas com menos frequência em outro.
  Os dois hashes podem usar a mesma ID de produto como parte da chave. Por exemplo, você pode usar "produto: *nn*" (onde *nn* é a ID do produto) para as informações do produto e "product_details: *nn*" para os dados detalhados. Essa estratégia pode ajudar a reduzir o volume de dados que a maioria das consultas provavelmente recuperará.
* Você pode reparticionar um repositório de dados do Redis, mas lembre-se de que é uma tarefa complexa e demorada. O clustering do Redis pode reparticionar os dados automaticamente, mas esse recurso não está disponível com o Cache Redis do Azure. Portanto, ao criar o esquema de particionamento, tente deixar espaço livre suficiente em cada partição para permitir o crescimento esperado dos dados com o tempo. No entanto, lembre-se de que o Cache Redis do Azure foi criado para armazenar os dados em cache temporariamente e que os dados mantidos no cache podem ter um tempo de vida limitado especificado como um valor de tempo de vida (TTL). Para dados relativamente voláteis, o TTL pode ser curto, mas para dados estáticos, o TTL pode ser muito mais longo. Evite armazenar grandes quantidades de dados com tempo de vida longo no cache se o volume de dados tiver a probabilidade de preencher o cache. É possível especificar uma política de remoção que faz com que o Cache Redis do Azure remova os dados se o espaço for um fator determinante.

  > [!NOTE]
  > Quando você usa o Cache Redis do Azure, especifica o tamanho máximo do cache (de 250 MB a 53 GB) selecionando o tipo de preço adequado. No entanto, depois de criar um Cache Redis do Azure, não é possível aumentar (ou diminuir) seu tamanho.
  >
  >
* Como os lotes e as transações do Redis não abrangem várias conexões, todos os dados afetados por um lote ou uma transação devem ser mantidos no mesmo banco de dados (fragmento).

  > [!NOTE]
  > Uma sequência de operações em uma transação do Redis não é necessariamente atômica. Os comandos que compõem uma transação são verificados e enfileirados antes de serem executados. Se ocorrer um erro durante essa fase, a fila inteira será descartada. No entanto, depois que a transação tiver sido enviada com êxito, os comandos na fila serão executados na sequência. Se algum comando falhar, somente a execução desse comando será interrompida. Todos os comandos anteriores e subsequentes na fila são executados. Para saber mais, visite a página [Transações] no site do Redis.
  >
  >
* O Redis permite um número limitado de operações atômicas. As únicas operações desse tipo que permitem vários valores e chaves são MGET e MSET. As operações MGET retornam uma coleção de valores para uma lista de chaves especificada e as operações MSET armazenam uma coleção de valores para uma lista de chaves especificada. Se precisar usar essas operações, os pares de chave/valor referenciados pelos comandos MSET e MGET devem ser armazenados no mesmo banco de dados.

## <a name="partitioning-strategies-for-azure-service-fabric"></a>Estratégias de particionamento para o Azure Service Fabric
O Azure Service Fabric é uma plataforma de microsserviços que fornece um tempo de execução para aplicativos distribuídos na nuvem. O Service Fabric oferece suporte a executáveis convidados do .Net, serviços com e sem monitoração de estado e contêineres. Serviços com monitoração de estado fornecem um [coleção confiável][service-fabric-reliable-collections] para armazenar dados de maneira persistente em uma coleção de chave-valor dentro do cluster do Service Fabric. Para saber mais sobre estratégias de particionamento de chaves em uma coleção confiável, veja [diretrizes e recomendações para coleções confiáveis no Azure Service Fabric].

### <a name="more-information"></a>Mais informações
* [Visão geral do Azure Service Fabric] é uma introdução ao Azure Service Fabric.
* [Particionar serviços confiáveis do Service Fabric] fornece mais informações sobre os serviços confiáveis no Azure Service Fabric.

## <a name="partitioning-strategies-for-azure-event-hubs"></a>Estratégias de particionamento para Hubs de Eventos do Azure

Os [Hubs de Eventos do Azure][event-hubs] foram projetados para streaming em grande escala e o particionamento de dados foi inserido no serviço para habilitar o escalonamento horizontal. Cada consumidor lê somente uma partição específica de fluxo de mensagens. 

O editor de eventos só está ciente da sua chave de partição, não da partição para a qual os eventos são publicados. Essa desassociação de chave e partição isenta o remetente da necessidade de saber muito sobre o processamento de downstream. (Também é possível enviar eventos diretamente para uma determinada partição, mas geralmente isso não é recomendável).  

Considere a escala de longo prazo quando você seleciona a contagem de partições. Após a criação de um hub de eventos, você não poderá alterar o número de partições. 

Para saber mais sobre como usar as partições em Hubs de Eventos, veja [O que são Hubs de Eventos?].

Para obter considerações sobre compensações entre disponibilidade e a consistência, veja [Disponibilidade e consistência nos Hubs de Eventos].

## <a name="rebalancing-partitions"></a>Rebalanceando partições
À medida que um sistema se desenvolve e você entende melhor os padrões de uso, talvez seja necessário ajustar o esquema de particionamento. Por exemplo, partições individuais podem começar a atrair um volume desproporcional de tráfego e se tornam mais acessadas, levando à contenção excessiva. Além disso, você pode ter subestimado o volume de dados em algumas partições, fazendo com que você se aproxime dos limites da capacidade de armazenamento nessas partições. Seja qual for a causa, às vezes, é necessário rebalancear as partições para distribuir a carga de maneira mais uniforme.

Em alguns casos, os sistemas de armazenamento de dados que não expõem publicamente como os dados são alocados aos servidores podem automaticamente redistribuir partições dentro dos limites dos recursos disponíveis. Em outras situações, o rebalanceamento é uma tarefa administrativa que consiste em dois estágios:

1. Determinar a nova estratégia de particionamento para apurar: 
   * Quais partições talvez precisem ser divididas (ou possivelmente combinadas).
   * Como alocar dados a essas novas partições pela criação de novas chaves de partição.
2. Migrar os dados afetados do esquema de particionamento antigo para o novo conjunto de partições.

> [!NOTE]
> O mapeamento das coleções de banco de dados para os servidores é transparente, mas você ainda pode alcançar os limites de capacidade de armazenamento e de produtividade de uma conta do Cosmos DB. Se isso acontecer, você precisará reformular o esquema de particionamento e migrar os dados.
>
>

Dependendo da tecnologia de armazenamento de dados e do design do seu sistema de armazenamento de dados, você poderá migrar os dados entre partições enquanto eles estiverem sendo usados (migração online). Se isso não for possível, talvez você precise fazer com que as partições afetadas fiquem temporariamente indisponíveis enquanto os dados são realocados (migração offline).

## <a name="offline-migration"></a>Migração offline
A migração offline é, indiscutivelmente, a abordagem mais simples, uma vez que ela reduz as chances de contenção. Não faça alterações nos dados enquanto eles estiverem sendo movidos e reestruturados.

Conceitualmente, esse processo inclui as etapas a seguir:

1. Marcar o fragmento offline.
2. Dividir/mesclar e mover os dados para os novos fragmentos.
3. Verificar os dados.
4. Ativar os novos fragmentos.
5. Remover o fragmento antigo.

Para manter um pouco de disponibilidade, é possível marcar o fragmento original como somente leitura na etapa 1 em vez de torná-lo indisponível. Isso permite que os aplicativos leiam os dados enquanto eles estão sendo movidos, mas não alterá-los.

## <a name="online-migration"></a>Migração online
A execução da migração online é mais complexa, mas causa menos interrupção para os usuários já que os dados permanecem disponíveis durante todo o procedimento. O processo é semelhante aquele usado pela migração offline, com a exceção de que o fragmento original não está marcado como offline (etapa 1). Dependendo da granularidade do processo de migração (por exemplo, se é feito item por item ou fragmento por fragmento), o código de acesso a dados nos aplicativos cliente pode ter que manipular os dados de leitura e gravação que são mantidos em dois locais (no fragmento original e no novo fragmento).

Para obter um exemplo de uma solução que permite a migração online, confira o artigo [Dimensionamento usando a ferramenta de divisão/mesclagem do Banco de Dados Elástico] no site da Microsoft.

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados
Ao considerar estratégias para implementar a consistência de dados, os seguintes padrões também podem ser relevantes para o cenário:

* A página [Data Consistency Primer] , no site da Microsoft, descreve estratégias para manter a consistência em um ambiente distribuído como a nuvem.
* A página [Data partitioning guidance] , no site da Microsoft, fornece uma visão geral da criação de partições para atender a vários critérios em uma solução distribuída.
* O [Sharding pattern] , conforme descrito no site da Microsoft, resume algumas estratégias comuns para a fragmentação de dados.
* O [Index Table Pattern] , conforme descrito no site da Microsoft, ilustra como criar índices secundários sobre os dados. Um aplicativo pode recuperar dados rapidamente com essa abordagem, usando consultas que não referenciam a chave primária de uma coleção.
* O [Materialized View Pattern] , conforme descrito no site da Microsoft, descreve como gerar exibições pré-populadas que resumem os dados para permitir operações de consulta rápidas. Essa abordagem poderá ser útil em um armazenamento de dados particionado se as partições que contêm os dados sendo resumidos forem distribuídas em vários locais.
* O artigo [Using Azure Content Delivery Network] , no site da Microsoft, fornece instruções adicionais sobre como configurar e usar a Rede de Distribuição de Conteúdo com o Azure.

## <a name="more-information"></a>Mais informações
* A página [O que é o Banco de Dados SQL do Azure?] , no site da Microsoft, fornece documentação detalhada que descreve como criar e usar bancos de dados SQL.
* A página [Visão geral dos recursos do Banco de Dados Elástico] , no site da Microsoft, fornece uma introdução abrangente ao Banco de Dados Elástico.
* A página [Dimensionamento usando a ferramenta de divisão/mesclagem do Banco de Dados Elástico] , no site da Microsoft, contém informações sobre como usar o serviço de divisão/mesclagem para gerenciar fragmentos do Banco de Dados Elástico.
* A página [Metas de desempenho e escalabilidade do Armazenamento do Azure](https://msdn.microsoft.com/library/azure/dn249410.aspx) , no site da Microsoft, documenta os limites atuais de tamanho e taxa de transferência do Armazenamento do Azure.
* A página [Performing entity group transactions] , no site da Microsoft, fornece informações detalhadas sobre como implementar operações transacionais nas entidades que são armazenadas no armazenamento de tabelas do Azure.
* O artigo [Azure Storage Table Design Guide] , no site da Microsoft, contém informações detalhadas sobre como particionar dados no armazenamento de tabelas do Azure.
* A página [Using Azure Content Delivery Network], no site da Microsoft, descreve como replicar os dados que são mantidos no armazenamento de blobs do Azure usando a Rede de Distribuição de Conteúdo do Azure.
* A página [O que é o Azure Search?] , no site da Microsoft, fornece uma descrição completa dos recursos que estão disponíveis com o Azure Search.
* A página [Limites de serviço no Azure Search] , no site da Microsoft, contém informações sobre a capacidade de cada instância do Azure Search.
* A página [Tipos de Dados com Suporte (Azure Search)] , no site da Microsoft, resume os tipos de dados que podem ser usados em documentos e índices pesquisáveis.
* A página [Cache Redis do Azure] , no site da Microsoft, fornece uma introdução ao Cache Redis do Azure.
* A página [Partitioning: how to split data among multiple Redis instances] , no site do Redis, fornece informações sobre como implementar o particionamento com o Redis.
* A página [Running Redis on a CentOS Linux VM in Azure] , no site da Microsoft, explica um exemplo que mostra como criar e configurar um nó do Redis em execução como uma VM do Azure.
* A página [Data types], no site do Redis, descreve os tipos de dados disponíveis com o Redis e o Cache Redis do Azure.

[Disponibilidade e consistência nos Hubs de Eventos]: /azure/event-hubs/event-hubs-availability-and-consistency
[azure-limits]: /azure/azure-subscription-service-limits
[Rede de Distribuição de Conteúdo do Azure]: /azure/cdn/cdn-overview
[Cache Redis do Azure]: http://azure.microsoft.com/services/cache/
[Azure Storage Scalability and Performance Targets]: /azure/storage/storage-scalability-targets
[Azure Storage Table Design Guide]: /azure/storage/storage-table-design-guide
[Criando uma solução Poliglota]: https://msdn.microsoft.com/library/dn313279.aspx
[cosmos-db-ru]: /azure/cosmos-db/request-units
[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence]: https://msdn.microsoft.com/library/dn271399.aspx
[Data consistency primer]: http://aka.ms/Data-Consistency-Primer
[Data Partitioning Guidance]: https://msdn.microsoft.com/library/dn589795.aspx
[Data Types]: http://redis.io/topics/data-types
[cosmosdb-sql-api]: /azure/cosmos-db/sql-api-introduction
[Visão geral dos recursos do Banco de Dados Elástico]: /azure/sql-database/sql-database-elastic-scale-introduction
[event-hubs]: /azure/event-hubs
[Federations Migration Utility]: https://code.msdn.microsoft.com/vstudio/Federations-Migration-ce61e9c1
[diretrizes e recomendações para Coleções Confiáveis no Azure Service Fabric]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections-guidelines
[Index Table Pattern]: http://aka.ms/Index-Table-Pattern
[Materialized View Pattern]: http://aka.ms/Materialized-View-Pattern
[Consulta de vários fragmentos]: /azure/sql-database/sql-database-elastic-scale-multishard-querying
[Visão geral do Azure Service Fabric]: /azure/service-fabric/service-fabric-overview
[Particionar serviços confiáveis do Service Fabric]: /azure/service-fabric/service-fabric-concepts-partitioning
[Partitioning: how to split data among multiple Redis instances]: http://redis.io/topics/partitioning
[Performing Entity Group Transactions]: https://msdn.microsoft.com/library/azure/dd894038.aspx
[tutorial de cluster do Redis]: http://redis.io/topics/cluster-tutorial
[Running Redis on a CentOS Linux VM in Azure]: http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx
[Dimensionamento usando a ferramenta de divisão/mesclagem do Banco de Dados Elástico]: /azure/sql-database/sql-database-elastic-scale-overview-split-and-merge
[Using Azure Content Delivery Network]: /azure/cdn/cdn-create-new-endpoint
[Cotas do Barramento de Serviço]: /azure/service-bus-messaging/service-bus-quotas
[service-fabric-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[Limites de serviço no Azure Search]:  /azure/search/search-limits-quotas-capacity
[Sharding pattern]: http://aka.ms/Sharding-Pattern
[Tipos de Dados com Suporte (Azure Search)]:  https://msdn.microsoft.com/library/azure/dn798938.aspx
[Transações]: http://redis.io/topics/transactions
[O que são Hubs de Eventos?]: /azure/event-hubs/event-hubs-what-is-event-hubs
[O que é o Azure Search?]: /azure/search/search-what-is-azure-search
[O que é o Banco de Dados SQL do Azure?]: /azure/sql-database/sql-database-technical-overview
