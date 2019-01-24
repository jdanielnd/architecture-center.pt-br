---
title: Diretrizes de particionamento de dados
titleSuffix: Best practices for cloud applications
description: Instruções de como separar partições a serem gerenciadas e acessadas separadamente.
author: dragon119
ms.date: 11/04/2018
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 561fe6e47a4cd64aa545349dde4c76260d76e78e
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54482300"
---
# <a name="horizontal-vertical-and-functional-data-partitioning"></a>Particionamento horizontal, vertical e funcional de dados

Em muitas soluções de grande escala, os dados são divididos em *partições* que podem ser gerenciadas e acessadas separadamente. O particionamento pode aprimorar a escalabilidade, reduzir a contenção e otimizar o desempenho. Ele também pode fornecer um mecanismo para dividir os dados pelo padrão de uso. Por exemplo, você pode arquivar dados antigos em armazenamento de dados mais barato.

Porém, a estratégia de particionamento deve ser cuidadosamente escolhida para maximizar os benefícios, ao mesmo tempo que minimiza os efeitos adversos.

> [!NOTE]
> Neste artigo, o termo *particionamento* significa o processo de dividir fisicamente os dados em armazenamentos de dados separados. Não é igual ao particionamento de tabela do SQL Server.

<!-- markdownlint-disable MD026 -->

## <a name="why-partition-data"></a>Por que particionar os dados?

<!-- markdownlint-enable MD026 -->

- **Melhorar a escalabilidade**. Quando você escala verticalmente um sistema de banco de dados individual, em algum momento ele atingirá um limite de hardware físico. Se você dividir os dados entre várias partições, cada uma hospedada em um servidor separado, será possível escalar o sistema de forma horizontal quase que indefinidamente.

- **Melhorar o desempenho**. As operações de acesso a dados em cada partição ocorrem em um volume de dados menor. Feito corretamente, o particionamento pode tornar seu sistema mais eficiente. As operações que afetam mais de uma partição podem ser executadas paralelamente.

- **Melhorar a segurança**. Em alguns casos, você pode separar dados confidenciais e não confidenciais em partições diferentes e aplicar controles de segurança diferentes aos dados confidenciais.

- **Fornecer flexibilidade operacional**. O particionamento oferece várias oportunidades para o ajuste das operações, maximizando a eficiência administrativa e minimizando os custos. Por exemplo, você pode definir diferentes estratégias para gerenciamento, monitoramento, backup, restauração e outras tarefas administrativas baseadas na importância dos dados em cada partição.

- **Fazer a correspondência do repositório de dados ao padrão de uso**. O particionamento permite que cada partição seja implantada em um tipo diferente de repositório de dados, com base no custo e nos recursos internos que o repositório de dados oferece. Por exemplo, dados binários grandes podem ser armazenados em um armazenamento de blob, enquanto dados mais estruturados podem ser mantidos em um banco de dados de documentos. Confira [Escolher o armazenamento de dados correto](../guide/technology-choices/data-store-overview.md).

- **Melhorar a disponibilidade**. Separar dados em vários servidores evita um ponto único de falha. Se uma instância falhar, somente os dados nessa partição não estão disponíveis. As operações em outras partições podem continuar. Para armazenamentos de dados de PaaS gerenciados, essa consideração é menos relevante porque esses serviços são projetados com redundância interna.

## <a name="designing-partitions"></a>Criando partições

Há três estratégias típicas para o particionamento dos dados:

- **Particionamento horizontal** (geralmente denominado *fragmentação*). Nessa estratégia, cada partição é um armazenamento de dados separado, mas todas as partições têm o mesmo esquema. Cada partição é conhecida como um *fragmento* e contém um subconjunto específico de dados, como todos os pedidos de um conjunto específico de clientes.

- **Particionamento vertical**. Nessa estratégia, cada partição contém um subconjunto dos campos de itens no repositório de dados. Os campos são divididos de acordo com seu padrão de uso. Por exemplo, os campos acessados com frequência podem ser colocados em uma partição vertical e os campos acessados com menos frequência em outra.

- **Particionamento funcional**. Nessa estratégia, os dados são agregados de acordo com o modo como são usados por cada contexto vinculado no sistema. Por exemplo, um sistema de comércio eletrônico pode armazenar os dados da fatura em uma partição e os dados de inventário dos produtos em outra.

Essas estratégias podem ser combinadas, e é recomendável levá-las em consideração ao projetar um esquema de particionamento. Por exemplo, você poderia dividir os dados em fragmentos e então usar o particionamento vertical para subdividir ainda mais os dados em cada fragmento.

### <a name="horizontal-partitioning-sharding"></a>Particionamento horizontal (fragmentação)

A Figura 1 mostra um particionamento horizontal ou fragmentação. Neste exemplo, os dados do inventário de produtos são divididos em fragmentos com base na chave do produto (Product Key). Cada fragmento contém os dados para um intervalo contíguo de chaves de fragmento (A-G e H-Z), organizadas em ordem alfabética. A fragmentação distribui a carga em um número maior de computadores, o que reduz a contenção e melhora o desempenho.

![Particionamento horizontal (fragmentação) de dados com base em uma chave de partição](./images/data-partitioning/DataPartitioning01.png)

*Figura 1. Particionamento horizontal (fragmentação) de dados com base em uma chave de partição.*

O fator mais importante é a escolha de uma chave de fragmentação. Pode ser difícil alterar a chave depois que o sistema estiver em operação. A chave deve garantir que os dados sejam particionados para distribuir a carga de trabalho da maneira mais uniforme possível entre os fragmentos.

Os fragmentos não precisam ser do mesmo tamanho. É mais importante equilibrar a quantidade de solicitações. Alguns fragmentos podem ser muito grandes, mas cada item tem uma quantidade baixa de operações de acesso. Outros fragmentos podem ser menores, mas cada item é acessado com muito mais frequência. Também é importante garantir que um único fragmento não exceda os limites de escala (em termos de capacidade e recursos de processamento) do armazenamento de dados.

Evite criar partições frequentes que possam afetar o desempenho e a disponibilidade. Por exemplo, usar a primeira letra do nome de um cliente gera uma distribuição desbalanceada, pois algumas letras são mais comuns. Em vez disso, use um hash de um identificador de cliente para distribuir os dados mais uniformemente entre as partições.

Escolha uma chave de fragmentação que minimize os requisitos futuros de divisão de grandes fragmentos, agrupamento de pequenos fragmentos em partições maiores ou alteração do esquema. Essas operações podem ser muito demoradas e podem exigir que você deixe um ou mais fragmentos offline enquanto elas são executadas.

Se os fragmentos forem replicados, será possível manter algumas das réplicas online enquanto as outras são divididas, mescladas ou reconfiguradas. No entanto, talvez o sistema precise limitar as operações que podem ser realizadas durante a reconfiguração. Por exemplo, os dados nas réplicas podem ser marcados como somente leitura para evitar inconsistências de dados.

Para saber mais sobre o particionamento horizontal, confira [Padrão de fragmentação].

### <a name="vertical-partitioning"></a>Particionamento vertical

O uso mais comum do particionamento vertical é reduzir a E/S e os custos de desempenho associados à busca de itens que são acessados frequentemente. A Figura 2 mostra um exemplo de particionamento vertical. Nesse exemplo, as diferentes propriedades de um item são armazenadas em diferentes partições. Uma partição mantém dados que são acessados mais frequentemente, incluindo o nome, a descrição e o preço do produto. Outra partição mantém os dados de inventário: a contagem de estoque e a data do último pedido.

![Particionamento vertical de dados por seu padrão de uso](./images/data-partitioning/DataPartitioning02.png)

*Figura 2. Particionamento vertical de dados por seu padrão de uso.*

Neste exemplo, o aplicativo regularmente consulta o nome, a descrição e o preço do produto ao exibir os detalhes do produto para os clientes. A contagem de estoque e a data do último pedido são mantidas em uma partição separada porque, geralmente, esses dois itens são usados juntos.

Outras vantagens do particionamento vertical:

- Os dados relativamente lentos (nome do produto, descrição e preço) podem ser separados dos dados mais dinâmicos (nível de estoque e data do último pedido). Os dados lentos são bons candidatos para o armazenamento em cache de um aplicativo.

- Os dados confidenciais podem ser armazenados em uma partição separada com controles de segurança adicionais.

- O particionamento vertical pode reduzir a quantidade de acesso simultâneo necessária.

O particionamento vertical funciona no nível da entidade em um armazenamento de dados, parcialmente normalizando uma entidade para dividir um item *grande* em um conjunto de itens *pequenos*. Ele é ideal para repositórios de dados orientados a colunas, como o HBase e Cassandra. Se for improvável que os dados em uma coleção de colunas serão alterados, você também poderá considerar o uso de repositórios de colunas no SQL Server.

### <a name="functional-partitioning"></a>Particionamento funcional

Quando for possível identificar um contexto vinculado para cada área comercial distinta em um aplicativo, o particionamento funcional é uma forma de melhorar o isolamento e o desempenho de acesso a dados. Outro uso comum do particionamento funcional é separar dados de leitura/gravação de dados somente leitura. A Figura 3 mostra uma visão geral de particionamento funcional em que os dados do inventário são separados dos dados do cliente.

![Particionamento funcional de dados por contexto ou subdomínio vinculado](./images/data-partitioning/DataPartitioning03.png)

*Figura 3. Particionamento funcional de dados por contexto ou subdomínio vinculado.*

Essa estratégia de particionamento pode ajudar a reduzir a contenção do acesso a dados em diferentes partes de um sistema.

## <a name="designing-partitions-for-scalability"></a>Criando partições para a escalabilidade

É fundamental considerar o tamanho e a carga de trabalho de cada partição e balanceá-los para que os dados sejam distribuídos para obtenção da máxima escalabilidade. No entanto, você também deve particionar os dados para que eles não excedam os limites de dimensionamento de um único repositório de partição.

Siga estas etapas ao criar partições para escalabilidade:

1. Analise o aplicativo para entender os padrões de acesso a dados, como o tamanho do conjunto de resultados retornado por cada consulta, a frequência de acesso, a latência inerente e os requisitos de processamento de computação do lado do servidor. Em muitos casos, algumas entidades principais exigirão a maior parte dos recursos de processamento.
2. Use essa análise para determinar as metas de escalabilidade atuais e futuras, como o tamanho dos dados e carga de trabalho. Em seguida, distribua os dados nas partições para atender à meta de escalabilidade. Para o particionamento horizontal, é importante escolher a chave de fragmento certa para garantir que a distribuição seja uniforme. Para saber mais, confira [Padrão de fragmentação].
3. Verifique se cada partição tem recursos suficientes para lidar com os requisitos de escalabilidade em termos de tamanho de dados e taxa de transferência. Dependendo do armazenamento de dados, pode haver um limite na quantidade de espaço de armazenamento, capacidade de processamento ou largura de banda de rede por partição. Se os requisitos tiverem a probabilidade de exceder esses limites, poderá ser preciso refinar sua estratégia de particionamento ou dividir ainda mais os dados, possivelmente combinando duas ou mais estratégias.
4. Monitore o sistema para verificar se os dados são distribuídos conforme o esperado e se as partições podem manipular a carga. O uso real nem sempre corresponde à previsão de uma análise. Nesse caso, pode ser possível rebalancear as partições ou mesmo reprojetar algumas partes do sistema para obter o balanceamento necessário.

Alguns ambientes de nuvem alocam recursos em termos de limites de infraestrutura. Garanta que os limites de seu limite selecionado forneçam espaço suficiente para qualquer aumento previsto no volume de dados, em termos de armazenamento de dados, capacidade de processamento e largura de banda.

Por exemplo, caso você use o armazenamento de tabelas do Azure, há um limite para o volume de solicitações que podem ser administradas por uma única partição em um determinado período de tempo. (Confira [Metas de desempenho e escalabilidade do armazenamento do Azure].) Um fragmento ocupado pode exigir mais recursos do que uma única partição pode administrar. Nesse caso, o fragmento pode precisar ser reparticionado para distribuir a carga. Se o tamanho total ou a taxa de transferência dessas tabelas exceder a capacidade de uma conta de armazenamento, pode ser necessário criar mais contas de armazenamento e distribuir as tabelas entre essas contas.

## <a name="designing-partitions-for-query-performance"></a>Criando partições para o desempenho da consulta

Muitas vezes, o desempenho da consulta pode ser impulsionado pelo uso de conjuntos de dados menores e pela execução de consultas paralelas. Cada partição deve conter uma pequena proporção do conjunto de dados inteiro. Essa redução no volume pode melhorar o desempenho das consultas. No entanto, o particionamento não é uma alternativa para a criação e configuração adequadas de um banco de dados. Por exemplo, verifique se os índices necessários estão em vigor.

Siga estas etapas ao criar partições para desempenho de consulta:

1. Examine os requisitos e o desempenho do aplicativo:

   - Use os requisitos de negócios para determinar as consultas críticas que devem sempre ser executadas rapidamente.
   - Monitore o sistema para identificar todas as consultas que são executadas lentamente.
   - Descubra quais consultas são executadas mais frequentemente. Mesmo se uma única consulta tiver um custo mínimo, o consumo de recursos cumulativo poderia ser significativo.

2. Particione os dados que estejam causando lentidão no desempenho:
   - Limitar o tamanho de cada partição para que o tempo de resposta da consulta esteja dentro da meta.
   - Se você usar o particionamento horizontal, projete a chave compartilhada de modo que o aplicativo possa selecionar facilmente a partição correta. Isso impede que a consulta tenha que verificar cada partição.
   - Considere o local de uma partição. Se possível, tente manter os dados nas partições que estão geograficamente próximas aos aplicativos e aos usuários que os acessam.

3. Se uma entidade tiver requisitos de desempenho da consulta e de taxa de transferência, use o particionamento funcional com base nessa entidade. Se isso ainda não atender aos requisitos, aplique também o particionamento horizontal. Na maioria dos casos, uma única estratégia de particionamento será suficiente, mas em alguns casos, é mais eficiente combinar as duas estratégias.

4. Cogite executar consultas paralelamente nas partições para melhorar o desempenho.

## <a name="designing-partitions-for-availability"></a>Criando partições para a disponibilidade

O particionamento de dados pode melhorar a disponibilidade de aplicativos, garantindo que todo o conjunto de dados não constitua um ponto único de falha e que subconjuntos individuais do conjunto de dados possam ser gerenciados independentemente.

Leve os fatores a seguir em consideração, os quais afetam a disponibilidade:

**O quanto os dados são fundamentais para as operações de negócios**. Identifique quais dados são informações de negócios críticas, como transações, e quais são dados operacionais menos críticos, como arquivos de log.

- Cogite armazenar os dados críticos em partições altamente disponíveis com um plano de backup adequado.

- Estabeleça procedimentos de gerenciamento e monitoramento separados para conjunto de dados diferentes.

- Colocar os dados com o mesmo nível de importância na mesma partição para que eles possam ser armazenados juntos em backup com uma frequência adequada. Por exemplo, as partições que contêm dados de transações podem precisar de backup com mais frequência do que as partições que contêm informações de rastreamento ou registro em log.

**Como as partições individuais podem ser gerenciadas**. Criar partições para dar suporte ao gerenciamento e manutenção independentes oferece várias vantagens. Por exemplo: 

- Se uma partição falhar, ela pode ser recuperada independentemente sem aplicativos que acessem dados em outras partições.

- O particionamento de dados por área geográfica permite que as tarefas de manutenção agendadas ocorram fora dos horários de pico de cada local. Verifique se as partições não são muito grandes para evitar a conclusão de qualquer manutenção planejada durante esse período.

**A possibilidade de replica dados críticos entre partições**. Essa estratégia pode melhorar a disponibilidade e o desempenho, mas também pode apresentar problemas de consistência. A sincronização das alterações com cada réplica é demorada. Durante esse período, diferentes partições conterão valores de dados diferentes.

## <a name="application-design-considerations"></a>Considerações sobre o design de aplicativo

O particionamento acrescenta complexidade ao design e desenvolvimento do sistema. Considere o particionamento como uma parte fundamental do design do sistema, mesmo que inicialmente o sistema contenha apenas uma única partição. Se você pensar no particionamento como uma consideração posterior, o desafio será maior, porque você já tem um sistema em tempo real para manter:

- A lógica de acesso a dados precisa ser modificada.
- Pode ser que grandes quantidades de dados existentes precisem ser migradas para distribuí-las por partições
- Os usuários esperam poder continuar usando o sistema durante a migração.

Em alguns casos, o particionamento não é considerado importante, pois o conjunto de dados inicial é pequeno e pode ser facilmente manipulado por um único servidor. Isso pode ocorrer em algumas cargas de trabalho, mas muitos sistemas comerciais precisam ser expandidos conforme o número de usuários aumenta.

Além disso, não são apenas os grandes armazenamentos de dados que se beneficiam do particionamento. Por exemplo, um repositório de dados pequeno pode ser muito acessado por centenas de clientes simultâneos. Particionar os dados nessa situação pode ajudar a reduzir a contenção e melhorar a taxa de transferência.

Considere os seguintes pontos ao criar um esquema de particionamento de dados:

**Minimize as operações de acesso de dados entre partições**. Sempre que possível, mantenha os dados para as operações mais comuns do banco de dados juntos em cada partição para minimizar as operações de acesso a dados entre partições. Consultar entre partições pode ser mais demorado do que consultar uma única partição; porém, otimizar as partições para um conjunto de consultas pode afetar outros conjuntos de consultas. Se for preciso fazer consulta entre partições, minimize o tempo de consulta executando consultas paralelas e agregando os resultados no aplicativo. (Talvez essa abordagem não seja possível em alguns casos, por exemplo, quando o resultado de uma consulta é usado na próxima consulta.)

**Cogite replicar dados de referência estáticos.** Se as consultas usarem dados de referência relativamente estáticos, como tabelas de CEP ou listas de produtos, considere replicar esses dados em todas as partições para reduzir as operações de pesquisa separadas em outras partições. Essa abordagem também pode reduzir a probabilidade de que os dados de referência se tornem um conjunto de dados “frequente”, com tráfego pesado de todo o sistema. No entanto, há um custo adicional associado à sincronização de quaisquer alterações dos dados de referência.

**Minimize as junções entre partições.** Sempre que possível, minimize os requisitos de integridade referencial entre partições verticais e funcionais. Nesses esquemas, o aplicativo é responsável por manter a integridade referencial entre partições. As consultas que unem os dados em várias partições são ineficientes porque o aplicativo normalmente precisa executar consultas consecutivas com base em uma chave e em uma chave estrangeira. Em vez disso, considere replicar ou cancelar a normalização dos dados relevantes. Se as uniões entre partições forem necessárias, execute consultas paralelas nas partições e reúna os dados no aplicativo.

**Adote a consistência eventual**.  Avalie se a coerência forte é realmente um requisito. Uma abordagem comum em sistemas distribuídos é implementar a consistência eventual. Os dados em cada partição são atualizados separadamente, e a lógica de aplicativo garante que a atualizações sejam concluídas com êxito. Ela também trata das inconsistências que podem surgir da consulta de dados enquanto uma operação finalmente consistente está em execução.

**Considere como as consultas localizam a partição correta**. Se uma consulta precisar verificar todas as partições para localizar os dados necessários, haverá um impacto significativo no desempenho, mesmo quando várias consultas paralelas estiverem em execução. Com o particionamento vertical e funcional, as consultas podem especificar as partições naturalmente. O particionamento horizontal, por outro lado, pode dificultar a localização de um item, pois cada fragmento tem o mesmo esquema. Uma solução típica para manter um mapa que é usado para procurar a localização do fragmento para itens específicos. Esse mapa pode ser implementado na lógica de fragmentação do aplicativo ou mantido pelo repositório de dados se a fragmentação transparente for permitida.

**Considere o rebalanceamento periódico de fragmentos**. Com o particionamento horizontal o rebalanceamento de fragmentos pode ajudar a distribuir os dados uniformemente por tamanho e carga de trabalho para minimizar os pontos de acesso, maximizar o desempenho da consulta e contornar as limitações de armazenamento físico. No entanto, isso é uma tarefa complexa que geralmente requer o uso de uma ferramenta ou um processo personalizado.

**Replique partições.** Se você replicar cada partição, ela fornecerá proteção adicional contra falhas. Se uma única réplica falhar, as consultas podem ser direcionadas a uma cópia funcional.

**Se você atingir os limites físicos de uma estratégia de particionamento, talvez seja necessário estender a escalabilidade para um nível diferente**. Por exemplo, se o particionamento estiver no nível do banco de dados, você poderá precisar localizar ou replicar partições em vários bancos de dados. Se o particionamento já estiver no nível do banco de dados e as limitações físicas forem um problema, isso pode significar que você precisa localizar ou replicar partições em várias contas de hospedagem.

**Evite as transações que acessam os dados em várias partições**. Alguns repositórios de dados implementam consistência e integridade transacionais para operações que modificam dados, mas somente quando os dados estão localizados em uma única partição. Se precisar de suporte transacional em várias partições, você provavelmente precisará implementá-lo como parte da lógica do aplicativo, pois a maioria dos sistemas de particionamento não dão suporte nativo.

Todos os repositórios de dados exigem algumas atividades operacionais de gerenciamento e monitoramento. As tarefas podem variar de carregamento de dados, backup e restauração de dados a reorganização de dados, além de garantir que o sistema esteja funcionando correta e eficientemente.

Considere os seguintes fatores que afetam o gerenciamento operacional:

- **Como implementar tarefas apropriadas de gerenciamento e operacionais quando os dados são particionados**. Essas tarefas podem incluir backup e restauração, arquivamento de dados, monitoramento do sistema e outras tarefas administrativas. Por exemplo, manter a consistência lógica durante as operações de backup e restauração pode ser um desafio.

- **Como carregar os dados em várias partições e adicionar novos dados que estão chegando de outras fontes**. Alguns utilitários e ferramentas podem não permitir operações de dados fragmentadas, como carregamento de dados na partição correta.

- **Como arquivar e excluir os dados regularmente**. Para impedir o aumento excessivo de partições, você precisa arquivar e excluir dados regularmente (talvez mensalmente). Pode ser necessário transformar os dados para correspondência com um esquema diferente de arquivamento.

- **Como localizar problemas de integridade de dados**. Considere a execução de um processo periódico para localizar quaisquer problemas de integridade de dados, como os dados em uma partição que fazem referência a informações ausentes em outra. O processo pode tentar corrigir esses problemas de forma automática ou simplesmente gerar um relatório para revisão manual.

## <a name="rebalancing-partitions"></a>Rebalanceando partições

À medida que um sistema amadurece, talvez seja preciso ajustar o esquema de particionamento. Por exemplo, as partições individuais podem começar a obter um volume desproporcional de tráfego e se tornar mais acessadas, levando à contenção excessiva. Ou você pode ter subestimado o volume de dados em algumas partições, fazendo com que algumas partições abordem limites de capacidade.

Alguns armazenamentos de dados, como o Cosmos DB pode automaticamente rebalancear partições. Em outros casos, o rebalanceamento é uma tarefa administrativa que consiste em dois estágios:

1. Determine uma nova estratégia de particionamento.

    - Quais partições precisam ser divididas (ou possivelmente combinadas)?
    - Qual é a nova chave de partição?

2. Migre os dados do antigo esquema de particionamento para o novo conjunto de partições.

Dependendo do armazenamento de dados, você poderá migrar os dados entre as partições enquanto eles estiverem em uso. Isso é chamado de *migração online*. Se isso não for possível, talvez você precise deixar as partições indisponíveis enquanto os dados são realocados (*migração offline*).

### <a name="offline-migration"></a>Migração offline

Geralmente, a migração offline é mais simples porque ela reduz as chances de ocorrência de contenção. Conceitualmente, a migração offline funciona da seguinte maneira:

1. Marque a partição offline.
2. Divida/mescle e mova os dados para as novas partições.
3. Verificar os dados.
4. Deixe as novas partições online.
5. Remova a partição antiga.

Como opção, você pode marcar uma partição como somente leitura na etapa 1 de modo que os aplicativos ainda possam ler os dados enquanto eles estão sendo movidos.

## <a name="online-migration"></a>Migração online

A migração online é mais complexa de ser executada, mas menos interruptiva. O processo é semelhante à migração offline, com a exceção de que a partição original não está marcada como offline. Dependendo da granularidade do processo de migração (por exemplo, item por item ou fragmento por fragmento), o código de acesso a dados nos aplicativos cliente pode ter de manipular os dados de leitura e gravação que são mantidos em dois locais: a partição original e a nova partição.

## <a name="related-patterns"></a>Padrões relacionados

Os seguintes padrões de design podem ser relevantes para o seu cenário:

- O [Padrão de fragmentação](../patterns/sharding.md) descreve algumas estratégias comuns de fragmentação de dados.

- O [padrão da tabela de índice](../patterns/index-table.md) mostra como criar índices secundários sobre os dados. Um aplicativo pode recuperar dados rapidamente com essa abordagem, usando consultas que não referenciam a chave primária de uma coleção.

- O [padrão de exibição materializada](../patterns/materialized-view.md) descreve como gerar exibições pré-populadas que resumem os dados para permitir operações de consulta rápidas. Essa abordagem poderá ser útil em um armazenamento de dados particionado se as partições que contêm os dados sendo resumidos forem distribuídas em vários locais.

## <a name="next-steps"></a>Próximas etapas

- Saiba mais sobre estratégias de particionamento para serviços específicos do Azure. Confira [Estratégias de particionamento de dados](./data-partitioning-strategies.md)

[Metas de desempenho e escalabilidade do Armazenamento do Azure]: /azure/storage/storage-scalability-targets
