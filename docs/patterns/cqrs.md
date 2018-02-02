---
title: CQRS
description: "Separar as operações que leem dados de operações que atualizam dados usando interfaces separadas."
keywords: "padrão de design"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: 80f4a8880cf2212acf82dadb67b0181e1cbae099
ms.sourcegitcommit: a7aae13569e165d4e768ce0aaaac154ba612934f
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/30/2018
---
# <a name="command-and-query-responsibility-segregation-cqrs-pattern"></a><span data-ttu-id="2c230-104">Padrão CQRS (Segregação de Responsabilidade de Consulta e Comando)</span><span class="sxs-lookup"><span data-stu-id="2c230-104">Command and Query Responsibility Segregation (CQRS) pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="2c230-105">Separar as operações que leem dados de operações que atualizam dados usando interfaces separadas.</span><span class="sxs-lookup"><span data-stu-id="2c230-105">Segregate operations that read data from operations that update data by using separate interfaces.</span></span> <span data-ttu-id="2c230-106">Isso pode maximizar o desempenho, escalabilidade e segurança.</span><span class="sxs-lookup"><span data-stu-id="2c230-106">This can maximize performance, scalability, and security.</span></span> <span data-ttu-id="2c230-107">Suporta a evolução do sistema ao longo do tempo através de maior flexibilidade e evita que comandos de atualização provoquem conflitos de mesclagem no nível de domínio.</span><span class="sxs-lookup"><span data-stu-id="2c230-107">Supports the evolution of the system over time through higher flexibility, and prevent update commands from causing merge conflicts at the domain level.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="2c230-108">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="2c230-108">Context and problem</span></span>

<span data-ttu-id="2c230-109">Nos sistemas de gerenciamento de dados tradicionais, ambos os comandos (atualizações para os dados) e consultas (solicitações para dados) são executados no mesmo conjunto de entidades em um repositório de dados único.</span><span class="sxs-lookup"><span data-stu-id="2c230-109">In traditional data management systems, both commands (updates to the data) and queries (requests for data) are executed against the same set of entities in a single data repository.</span></span> <span data-ttu-id="2c230-110">Essas entidades podem ser um subconjunto das linhas em uma ou mais tabelas em um banco de dados relacional, como o SQL Server.</span><span class="sxs-lookup"><span data-stu-id="2c230-110">These entities can be a subset of the rows in one or more tables in a relational database such as SQL Server.</span></span>

<span data-ttu-id="2c230-111">Normalmente, nesses sistemas, todas as operações CRUD (criar, ler, atualizar e excluir) são aplicadas na mesma representação da entidade.</span><span class="sxs-lookup"><span data-stu-id="2c230-111">Typically in these systems, all create, read, update, and delete (CRUD) operations are applied to the same representation of the entity.</span></span> <span data-ttu-id="2c230-112">Por exemplo, um DTO (objeto de transferência de dados) que representa um cliente é recuperado do armazenamento de dados pela DAL (camada de acesso aos dados) e exibido na tela.</span><span class="sxs-lookup"><span data-stu-id="2c230-112">For example, a data transfer object (DTO) representing a customer is retrieved from the data store by the data access layer (DAL) and displayed on the screen.</span></span> <span data-ttu-id="2c230-113">Um usuário atualiza alguns campos do DTO (talvez através da vinculação de dados) e, em seguida, o DTO é salvo novamente no armazenamento de dados pela DAL.</span><span class="sxs-lookup"><span data-stu-id="2c230-113">A user updates some fields of the DTO (perhaps through data binding) and the DTO is then saved back in the data store by the DAL.</span></span> <span data-ttu-id="2c230-114">O mesmo DTO é utilizado para as operações de leitura e gravação.</span><span class="sxs-lookup"><span data-stu-id="2c230-114">The same DTO is used for both the read and write operations.</span></span> <span data-ttu-id="2c230-115">A figura ilustra uma arquitetura CRUD tradicional.</span><span class="sxs-lookup"><span data-stu-id="2c230-115">The figure illustrates a traditional CRUD architecture.</span></span>

![Uma arquitetura CRUD tradicional](./_images/command-and-query-responsibility-segregation-cqrs-tradition-crud.png)

<span data-ttu-id="2c230-117">Os designs CRUD tradicionais funcionam bem quando apenas uma lógica de negócios limitada é aplicada às operações de dados.</span><span class="sxs-lookup"><span data-stu-id="2c230-117">Traditional CRUD designs work well when only limited business logic is applied to the data operations.</span></span> <span data-ttu-id="2c230-118">Os mecanismos de scaffold fornecidos pelas ferramentas de desenvolvimento podem criar código de acesso aos dados muito rapidamente, o que pode ser personalizado, conforme necessário.</span><span class="sxs-lookup"><span data-stu-id="2c230-118">Scaffold mechanisms provided by development tools can create data access code very quickly, which can then be customized as required.</span></span>

<span data-ttu-id="2c230-119">No entanto, a abordagem CRUD tradicional possui algumas desvantagens:</span><span class="sxs-lookup"><span data-stu-id="2c230-119">However, the traditional CRUD approach has some disadvantages:</span></span>

- <span data-ttu-id="2c230-120">Isso significa que frequentemente há uma incompatibilidade entre as representações de leitura e gravação dos dados, como colunas ou propriedades adicionais que devem ser atualizadas corretamente mesmo que não sejam necessárias como parte de uma operação.</span><span class="sxs-lookup"><span data-stu-id="2c230-120">It often means that there's a mismatch between the read and write representations of the data, such as additional columns or properties that must be updated correctly even though they aren't required as part of an operation.</span></span>

- <span data-ttu-id="2c230-121">Isso arrisca a contenção de dados quando os registros são bloqueados no armazenamento de dados em um domínio colaborativo, onde vários atores operam em paralelo no mesmo conjunto de dados.</span><span class="sxs-lookup"><span data-stu-id="2c230-121">It risks data contention when records are locked in the data store in a collaborative domain, where multiple actors operate in parallel on the same set of data.</span></span> <span data-ttu-id="2c230-122">Ou atualizar conflitos causados por atualizações simultâneas quando o bloqueio otimista é utilizado.</span><span class="sxs-lookup"><span data-stu-id="2c230-122">Or update conflicts caused by concurrent updates when optimistic locking is used.</span></span> <span data-ttu-id="2c230-123">Esses riscos aumentam à medida que a complexidade e a produção do sistema crescem.</span><span class="sxs-lookup"><span data-stu-id="2c230-123">These risks increase as the complexity and throughput of the system grows.</span></span> <span data-ttu-id="2c230-124">Além disso, a abordagem tradicional pode ter um efeito negativo no desempenho devido à carga no armazenamento de dados e na camada de acesso aos dados, e a complexidade das consultas necessárias para recuperar informações.</span><span class="sxs-lookup"><span data-stu-id="2c230-124">In addition, the traditional approach can have a negative effect on performance due to load on the data store and data access layer, and the complexity of queries required to retrieve information.</span></span>

- <span data-ttu-id="2c230-125">Isso pode tornar o gerenciamento de segurança e permissões mais complexo porque cada entidade está sujeita tanto a operações de leitura como de gravação, o que pode expor dados no contexto errado.</span><span class="sxs-lookup"><span data-stu-id="2c230-125">It can make managing security and permissions more complex because each entity is subject to both read and write operations, which might expose data in the wrong context.</span></span>

> <span data-ttu-id="2c230-126">Para uma compreensão mais profunda dos limites da abordagem CRUD, consulte [CRUD, Somente quando você puder custear isso](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/).</span><span class="sxs-lookup"><span data-stu-id="2c230-126">For a deeper understanding of the limits of the CRUD approach see [CRUD, Only When You Can Afford It](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/).</span></span>

## <a name="solution"></a><span data-ttu-id="2c230-127">Solução</span><span class="sxs-lookup"><span data-stu-id="2c230-127">Solution</span></span>

<span data-ttu-id="2c230-128">O CQRS (Segregação de Responsabilidade de Consulta e Comando) é um padrão que segrega as operações que leem dados (consultas) das operações que atualizam dados (comandos) utilizando interfaces separadas.</span><span class="sxs-lookup"><span data-stu-id="2c230-128">Command and Query Responsibility Segregation (CQRS) is a pattern that segregates the operations that read data (queries) from the operations that update data (commands) by using separate interfaces.</span></span> <span data-ttu-id="2c230-129">Isso significa que os modelos de dados utilizados para consultas e atualizações são diferentes.</span><span class="sxs-lookup"><span data-stu-id="2c230-129">This means that the data models used for querying and updates are different.</span></span> <span data-ttu-id="2c230-130">Os modelos podem, então, serem isolados, conforme mostrado na figura a seguir, embora isso não seja um requisito absoluto.</span><span class="sxs-lookup"><span data-stu-id="2c230-130">The models can then be isolated, as shown in the following figure, although that's not an absolute requirement.</span></span>

![Uma arquitetura CQRS básica](./_images/command-and-query-responsibility-segregation-cqrs-basic.png)

<span data-ttu-id="2c230-132">Comparado com o modelo de dados único utilizado em sistemas baseados em CRUD, o uso de modelos de atualização e consulta separados para os dados em sistemas baseados em CQRS simplifica o design e a implementação.</span><span class="sxs-lookup"><span data-stu-id="2c230-132">Compared to the single data model used in CRUD-based systems, the use of separate query and update models for the data in CQRS-based systems simplifies design and implementation.</span></span> <span data-ttu-id="2c230-133">No entanto, uma desvantagem é que, ao contrário de projetos CRUD, o código CQRS não pode ser gerado automaticamente utilizando mecanismos de scaffold.</span><span class="sxs-lookup"><span data-stu-id="2c230-133">However, one disadvantage is that unlike CRUD designs, CQRS code can't automatically be generated using scaffold mechanisms.</span></span>

<span data-ttu-id="2c230-134">O modelo de consulta para leitura de dados e o modelo de atualização para gravação de dados podem acessar o mesmo repositório físico, talvez utilizando exibições SQL ou gerando projeções instantâneas.</span><span class="sxs-lookup"><span data-stu-id="2c230-134">The query model for reading data and the update model for writing data can access the same physical store, perhaps by using SQL views or by generating projections on the fly.</span></span> <span data-ttu-id="2c230-135">No entanto, é comum separar os dados em diferentes repositórios físicos para maximizar o desempenho, a escalabilidade e a segurança, conforme mostrado na próxima figura.</span><span class="sxs-lookup"><span data-stu-id="2c230-135">However, it's common to separate the data into different physical stores to maximize performance, scalability, and security, as shown in the next figure.</span></span>

![Uma arquitetura CQRS com repositórios de leitura e gravação separados](./_images/command-and-query-responsibility-segregation-cqrs-separate-stores.png)

<span data-ttu-id="2c230-137">O repositório de leitura pode ser uma réplica somente para leitura do repositório de gravação, ou repositórios de gravação e leitura podem ter uma estrutura completamente diferente.</span><span class="sxs-lookup"><span data-stu-id="2c230-137">The read store can be a read-only replica of the write store, or the read and write stores can have a different structure altogether.</span></span> <span data-ttu-id="2c230-138">O uso de múltiplas réplicas de somente para leitura do repositório de leitura pode aumentar consideravelmente o desempenho de consulta e a capacidade de resposta da Interface de Usuário do aplicativo, especialmente em cenários distribuídos onde as réplicas de somente para leitura estão localizadas próximas às instâncias do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2c230-138">Using multiple read-only replicas of the read store can greatly increase query performance and application UI responsiveness, especially in distributed scenarios where read-only replicas are located close to the application instances.</span></span> <span data-ttu-id="2c230-139">Alguns sistemas de banco de dados (SQL Server) fornecem recursos adicionais, como réplicas de failover, para maximizar a disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="2c230-139">Some database systems (SQL Server) provide additional features such as failover replicas to maximize availability.</span></span>

<span data-ttu-id="2c230-140">A separação dos repositórios de gravação e leitura também permite que cada um seja dimensionado adequadamente para corresponder à carga.</span><span class="sxs-lookup"><span data-stu-id="2c230-140">Separation of the read and write stores also allows each to be scaled appropriately to match the load.</span></span> <span data-ttu-id="2c230-141">Por exemplo, os repositórios de leitura normalmente encontram uma carga muito maior do que os repositórios de gravação.</span><span class="sxs-lookup"><span data-stu-id="2c230-141">For example, read stores typically encounter a much higher load than write stores.</span></span>

<span data-ttu-id="2c230-142">Quando o modelo leitura/consulta contém dados desnormalizados (consulte [Padrão de Exibição Materializada](materialized-view.md)), o desempenho é maximizado ao ler dados para cada uma das exibições em um aplicativo ou ao consultar os dados no sistema.</span><span class="sxs-lookup"><span data-stu-id="2c230-142">When the query/read model contains denormalized data (see [Materialized View pattern](materialized-view.md)), performance is maximized when reading data for each of the views in an application or when querying the data in the system.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="2c230-143">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="2c230-143">Issues and considerations</span></span>

<span data-ttu-id="2c230-144">Considere os seguintes pontos ao decidir como implementar esse padrão:</span><span class="sxs-lookup"><span data-stu-id="2c230-144">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="2c230-145">Dividir o armazenamento de dados em repositórios físicos separados para operações de gravação e leitura pode aumentar o desempenho e a segurança de um sistema, mas pode adicionar complexidade em termos de resiliência e coerência eventual.</span><span class="sxs-lookup"><span data-stu-id="2c230-145">Dividing the data store into separate physical stores for read and write operations can increase the performance and security of a system, but it can add complexity in terms of resiliency and eventual consistency.</span></span> <span data-ttu-id="2c230-146">O repositório de modelos de leitura deve ser atualizado para refletir as alterações no repositório de gravação e pode ser difícil de detectar quando um usuário emitiu uma solicitação baseadas em dados de leitura obsoletos, o que significa que a operação não pode ser concluída.</span><span class="sxs-lookup"><span data-stu-id="2c230-146">The read model store must be updated to reflect changes to the write model store, and it can be difficult to detect when a user has issued a request based on stale read data, which means that the operation can't be completed.</span></span>

    > <span data-ttu-id="2c230-147">Para obter uma descrição de coerência eventual, consulte [Primer de consistência de dados](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="2c230-147">For a description of eventual consistency see the [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span>

- <span data-ttu-id="2c230-148">Considere aplicar CQRS em seções limitadas do seu sistema, onde será mais valioso.</span><span class="sxs-lookup"><span data-stu-id="2c230-148">Consider applying CQRS to limited sections of your system where it will be most valuable.</span></span>

- <span data-ttu-id="2c230-149">Uma abordagem típica para a implantação de coerência eventual é usar o fornecimento de evento em conjunto com o CQRS, de modo que o modelo de gravação seja um stream de eventos somente para anexos, conduzido pela execução de comandos.</span><span class="sxs-lookup"><span data-stu-id="2c230-149">A typical approach to deploying eventual consistency is to use event sourcing in conjunction with CQRS so that the write model is an append-only stream of events driven by execution of commands.</span></span> <span data-ttu-id="2c230-150">Esses eventos são utilizados para atualizar exibições materializadas que agem como o modelo de leitura.</span><span class="sxs-lookup"><span data-stu-id="2c230-150">These events are used to update materialized views that act as the read model.</span></span> <span data-ttu-id="2c230-151">Para obter mais informações, consulte [Fornecimento de evento e CQRS](https://msdn.microsoft.com/library/dn568103.aspx#EventSourcingandCQRS).</span><span class="sxs-lookup"><span data-stu-id="2c230-151">For more information see [Event Sourcing and CQRS](https://msdn.microsoft.com/library/dn568103.aspx#EventSourcingandCQRS).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="2c230-152">Quando usar esse padrão</span><span class="sxs-lookup"><span data-stu-id="2c230-152">When to use this pattern</span></span>

<span data-ttu-id="2c230-153">Utilize esse padrão nas seguintes situações:</span><span class="sxs-lookup"><span data-stu-id="2c230-153">Use this pattern in the following situations:</span></span>

- <span data-ttu-id="2c230-154">Domínios colaborativos onde várias operações são realizadas em paralelo nos mesmos dados.</span><span class="sxs-lookup"><span data-stu-id="2c230-154">Collaborative domains where multiple operations are performed in parallel on the same data.</span></span> <span data-ttu-id="2c230-155">O CQRS permite que você defina comandos com granularidade suficiente para minimizar os conflitos de mesclagem no nível de domínio (todos os conflitos que surgem podem ser mesclados pelo comando), mesmo quando atualizar o que parece ser o mesmo tipo de dados.</span><span class="sxs-lookup"><span data-stu-id="2c230-155">CQRS allows you to define commands with enough granularity to minimize merge conflicts at the domain level (any conflicts that do arise can be merged by the command), even when updating what appears to be the same type of data.</span></span>

- <span data-ttu-id="2c230-156">Interfaces de usuário baseadas em tarefas, onde os usuários são guiados por um processo complexo como uma série de etapas ou com modelos de domínio complexos.</span><span class="sxs-lookup"><span data-stu-id="2c230-156">Task-based user interfaces where users are guided through a complex process as a series of steps or with complex domain models.</span></span> <span data-ttu-id="2c230-157">Além disso, é útil para equipes já familiarizadas com técnicas de design orientado por domínio (DDD).</span><span class="sxs-lookup"><span data-stu-id="2c230-157">Also, useful for teams already familiar with domain-driven design (DDD) techniques.</span></span> <span data-ttu-id="2c230-158">O modelo de gravação possui uma pilha de processamento de comando completa com lógica de negócios, validação de entrada e validação de negócios para garantir que tudo seja sempre consistente para cada um dos agregados (cada cluster de objetos associados tratados como uma unidade para alterações de dados) no modelo de gravação.</span><span class="sxs-lookup"><span data-stu-id="2c230-158">The write model has a full command-processing stack with business logic, input validation, and business validation to ensure that everything is always consistent for each of the aggregates (each cluster of associated objects treated as a unit for data changes) in the write model.</span></span> <span data-ttu-id="2c230-159">O modelo de leitura não possui lógica de negócios ou pilha de validação e apenas retorna um DTO para uso em um modelo de exibição.</span><span class="sxs-lookup"><span data-stu-id="2c230-159">The read model has no business logic or validation stack and just returns a DTO for use in a view model.</span></span> <span data-ttu-id="2c230-160">O modelo de leitura é, eventualmente, consistente com o modelo de gravação.</span><span class="sxs-lookup"><span data-stu-id="2c230-160">The read model is eventually consistent with the write model.</span></span>

- <span data-ttu-id="2c230-161">Cenários onde o desempenho das leituras de dados deve ter ajuste fino separadamente do desempenho das gravações de dados, especialmente quando a relação de gravação/leitura é muito alta e quando um dimensionamento horizontal é necessário.</span><span class="sxs-lookup"><span data-stu-id="2c230-161">Scenarios where performance of data reads must be fine tuned separately from performance of data writes, especially when the read/write ratio is very high, and when horizontal scaling is required.</span></span> <span data-ttu-id="2c230-162">Por exemplo, em vários sistemas, o número de operações de leitura é muitas vezes maior que o número de operações de gravação.</span><span class="sxs-lookup"><span data-stu-id="2c230-162">For example, in many systems the number of read operations is many times greater than the number of write operations.</span></span> <span data-ttu-id="2c230-163">Para acomodar isso, considere dimensionar o modelo de leitura, mas executando o modelo de gravação em apenas uma ou algumas instâncias.</span><span class="sxs-lookup"><span data-stu-id="2c230-163">To accommodate this, consider scaling out the read model, but running the write model on only one or a few instances.</span></span> <span data-ttu-id="2c230-164">Um pequeno número de instâncias de modelo de gravação também ajuda a minimizar a ocorrência de conflitos de mesclagem.</span><span class="sxs-lookup"><span data-stu-id="2c230-164">A small number of write model instances also helps to minimize the occurrence of merge conflicts.</span></span>

- <span data-ttu-id="2c230-165">Cenários onde uma equipe de desenvolvedores pode se concentrar no modelo de domínio complexo que faz parte do modelo de gravação e outra equipe pode se concentrar no modelo de leitura e nas interfaces de usuário.</span><span class="sxs-lookup"><span data-stu-id="2c230-165">Scenarios where one team of developers can focus on the complex domain model that is part of the write model, and another team can focus on the read model and the user interfaces.</span></span>

- <span data-ttu-id="2c230-166">Cenários onde o sistema deve evoluir ao longo do tempo e pode conter várias versões do modelo, ou onde as regras de negócio mudam regularmente.</span><span class="sxs-lookup"><span data-stu-id="2c230-166">Scenarios where the system is expected to evolve over time and might contain multiple versions of the model, or where business rules change regularly.</span></span>

- <span data-ttu-id="2c230-167">Integração com outros sistemas, especialmente em combinação com fornecimento de evento, onde a falha temporal de um subsistema não deve afetar a disponibilidade dos outros.</span><span class="sxs-lookup"><span data-stu-id="2c230-167">Integration with other systems, especially in combination with event sourcing, where the temporal failure of one subsystem shouldn't affect the availability of the others.</span></span>

<span data-ttu-id="2c230-168">Esse padrão não é recomendado nas seguintes situações:</span><span class="sxs-lookup"><span data-stu-id="2c230-168">This pattern isn't recommended in the following situations:</span></span>

- <span data-ttu-id="2c230-169">Onde o domínio ou as regras de negócios são simples.</span><span class="sxs-lookup"><span data-stu-id="2c230-169">Where the domain or the business rules are simple.</span></span>

- <span data-ttu-id="2c230-170">Onde uma interface de usuário com estilo de CRUD simples e as operações de acesso aos dados relacionadas são suficientes.</span><span class="sxs-lookup"><span data-stu-id="2c230-170">Where a simple CRUD-style user interface and the related data access operations are sufficient.</span></span>

- <span data-ttu-id="2c230-171">Para implementação em todo o sistema.</span><span class="sxs-lookup"><span data-stu-id="2c230-171">For implementation across the whole system.</span></span> <span data-ttu-id="2c230-172">Existem componentes específicos de um cenário global de gerenciamento de dados em que o CQRS pode ser útil, mas pode adicionar uma complexidade considerável e desnecessária quando não for necessária.</span><span class="sxs-lookup"><span data-stu-id="2c230-172">There are specific components of an overall data management scenario where CQRS can be useful, but it can add considerable and unnecessary complexity when it isn't required.</span></span>

## <a name="event-sourcing-and-cqrs"></a><span data-ttu-id="2c230-173">Fornecimento de evento e CQRS</span><span class="sxs-lookup"><span data-stu-id="2c230-173">Event Sourcing and CQRS</span></span>

<span data-ttu-id="2c230-174">O padrão CQRS é frequentemente utilizado juntamente com o padrão de Fornecimento de Evento.</span><span class="sxs-lookup"><span data-stu-id="2c230-174">The CQRS pattern is often used along with the Event Sourcing pattern.</span></span> <span data-ttu-id="2c230-175">Os sistemas baseados em CQRS utilizam modelos de dados de gravação e leitura separados, cada um adaptado a tarefas relevantes e, muitas vezes, localizado em repositórios separados fisicamente.</span><span class="sxs-lookup"><span data-stu-id="2c230-175">CQRS-based systems use separate read and write data models, each tailored to relevant tasks and often located in physically separate stores.</span></span> <span data-ttu-id="2c230-176">Quando utilizado com o padrão [Fornecimento de Evento](event-sourcing.md), o repositório de eventos é o modelo de gravação e é a fonte oficial de informações.</span><span class="sxs-lookup"><span data-stu-id="2c230-176">When used with the [Event Sourcing](event-sourcing.md) pattern, the store of events is the write model, and is the official source of information.</span></span> <span data-ttu-id="2c230-177">O modelo de leitura de um sistema baseado em CQRS fornece exibições materializadas dos dados, geralmente como exibições altamente desnormalizadas.</span><span class="sxs-lookup"><span data-stu-id="2c230-177">The read model of a CQRS-based system provides materialized views of the data, typically as highly denormalized views.</span></span> <span data-ttu-id="2c230-178">Essas exibições são adaptadas às interfaces e aos requisitos de exibição do aplicativo, o que ajuda a maximizar tanto o desempenho de consulta como exibição.</span><span class="sxs-lookup"><span data-stu-id="2c230-178">These views are tailored to the interfaces and display requirements of the application, which helps to maximize both display and query performance.</span></span>

<span data-ttu-id="2c230-179">Utilizando o stream de eventos como o repositório de gravação, em vez dos dados reais em um ponto no tempo, evita conflitos de atualização em um único agregado e maximiza o desempenho e a escalabilidade.</span><span class="sxs-lookup"><span data-stu-id="2c230-179">Using the stream of events as the write store, rather than the actual data at a point in time, avoids update conflicts on a single aggregate and maximizes performance and scalability.</span></span> <span data-ttu-id="2c230-180">Os eventos podem ser utilizados para gerar de maneira assíncrona exibições materializadas dos dados que são utilizadas para preencher o repositório de leitura.</span><span class="sxs-lookup"><span data-stu-id="2c230-180">The events can be used to asynchronously generate materialized views of the data that are used to populate the read store.</span></span>

<span data-ttu-id="2c230-181">Como o repositório de evento é a fonte oficial de informações, é possível excluir as exibições materializadas e reproduzir todos os eventos passados para criar uma nova representação do estado atual quando o sistema evoluir ou quando o modelo de leitura precisar alterar.</span><span class="sxs-lookup"><span data-stu-id="2c230-181">Because the event store is the official source of information, it is possible to delete the materialized views and replay all past events to create a new representation of the current state when the system evolves, or when the read model must change.</span></span> <span data-ttu-id="2c230-182">As exibições materializadas são efetivamente um cache somente leitura durável dos dados.</span><span class="sxs-lookup"><span data-stu-id="2c230-182">The materialized views are in effect a durable read-only cache of the data.</span></span>

<span data-ttu-id="2c230-183">Ao utilizar CQRS combinado com o padrão Fornecimento de Evento, considere o seguinte:</span><span class="sxs-lookup"><span data-stu-id="2c230-183">When using CQRS combined with the Event Sourcing pattern, consider the following:</span></span>

- <span data-ttu-id="2c230-184">Como em qualquer sistema onde os repositórios de gravação e leitura são separados, os sistemas baseados nesse padrão são eventualmente consistentes.</span><span class="sxs-lookup"><span data-stu-id="2c230-184">As with any system where the write and read stores are separate, systems based on this pattern are only eventually consistent.</span></span> <span data-ttu-id="2c230-185">Haverá algum atraso entre o evento que estiver sendo gerado e o armazenamento de dados sendo atualizado.</span><span class="sxs-lookup"><span data-stu-id="2c230-185">There will be some delay between the event being generated and the data store being updated.</span></span>

- <span data-ttu-id="2c230-186">O padrão adiciona complexidade porque o código deve ser criado para iniciar e tratar eventos e montar ou atualizar as exibições ou objetos apropriados exigidos por consultas ou um modelo de leitura.</span><span class="sxs-lookup"><span data-stu-id="2c230-186">The pattern adds complexity because code must be created to initiate and handle events, and assemble or update the appropriate views or objects required by queries or a read model.</span></span> <span data-ttu-id="2c230-187">A complexidade do padrão CQRS quando utilizado com o padrão de Fornecimento de Evento pode dificultar a implementação bem-sucedida e requer uma abordagem diferente para a concepção de sistemas.</span><span class="sxs-lookup"><span data-stu-id="2c230-187">The complexity of the CQRS pattern when used with the Event Sourcing pattern can make a successful implementation more difficult, and requires a different approach to designing systems.</span></span> <span data-ttu-id="2c230-188">No entanto, o fornecimento de evento pode tornar mais fácil para modelar o domínio e facilitar para recompilar exibições ou criar novas porque a intenção das alterações nos dados é preservada.</span><span class="sxs-lookup"><span data-stu-id="2c230-188">However, event sourcing can make it easier to model the domain, and makes it easier to rebuild views or create new ones because the intent of the changes in the data is preserved.</span></span>

- <span data-ttu-id="2c230-189">A geração de exibições materializadas para uso no modelo de leitura ou projeções dos dados, reproduzindo e manipulando os eventos para entidades específicas ou coleções de entidades pode exigir um tempo de processamento significativo e uso de recursos.</span><span class="sxs-lookup"><span data-stu-id="2c230-189">Generating materialized views for use in the read model or projections of the data by replaying and handling the events for specific entities or collections of entities can require significant processing time and resource usage.</span></span> <span data-ttu-id="2c230-190">Isto é especialmente verdadeiro se requer soma ou análise de valores em longos períodos, pois poderá ser necessário examinar todos os eventos associados.</span><span class="sxs-lookup"><span data-stu-id="2c230-190">This is especially true if it requires summation or analysis of values over long periods, because all the associated events might need to be examined.</span></span> <span data-ttu-id="2c230-191">Resolva isso implementando instantâneos dos dados em intervalos agendados, como uma contagem total do número de uma ação específica que ocorreu ou o estado atual de uma entidade.</span><span class="sxs-lookup"><span data-stu-id="2c230-191">Resolve this by implementing snapshots of the data at scheduled intervals, such as a total count of the number of a specific action that have occurred, or the current state of an entity.</span></span>

## <a name="example"></a><span data-ttu-id="2c230-192">Exemplo</span><span class="sxs-lookup"><span data-stu-id="2c230-192">Example</span></span>

<span data-ttu-id="2c230-193">O código a seguir mostra alguns extratos de um exemplo de uma implementação CQRS que utiliza diferentes definições para os modelos de leitura e gravação.</span><span class="sxs-lookup"><span data-stu-id="2c230-193">The following code shows some extracts from an example of a CQRS implementation that uses different definitions for the read and the write models.</span></span> <span data-ttu-id="2c230-194">As interfaces modelo não ditarão quaisquer recursos dos repositórios de dados subjacentes, e elas podem evoluir e terem ajustes finos de maneira independente porque essas interfaces estão separadas.</span><span class="sxs-lookup"><span data-stu-id="2c230-194">The model interfaces don't dictate any features of the underlying data stores, and they can evolve and be fine-tuned independently because these interfaces are separated.</span></span>

<span data-ttu-id="2c230-195">O código a seguir mostra a definição do modelo de leitura.</span><span class="sxs-lookup"><span data-stu-id="2c230-195">The following code shows the read model definition.</span></span>

```csharp
// Query interface
namespace ReadModel
{
  public interface ProductsDao
  {
    ProductDisplay FindById(int productId);
    ICollection<ProductDisplay> FindByName(string name);
    ICollection<ProductInventory> FindOutOfStockProducts();
    ICollection<ProductDisplay> FindRelatedProducts(int productId);
  }

  public class ProductDisplay
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal UnitPrice { get; set; }
    public bool IsOutOfStock { get; set; }
    public double UserRating { get; set; }
  }

  public class ProductInventory
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public int CurrentStock { get; set; }
  }
}
```

<span data-ttu-id="2c230-196">O sistema permite aos usuários avaliar produtos.</span><span class="sxs-lookup"><span data-stu-id="2c230-196">The system allows users to rate products.</span></span> <span data-ttu-id="2c230-197">O código do aplicativo faz isso utilizando o comando `RateProduct` mostrado no código a seguir.</span><span class="sxs-lookup"><span data-stu-id="2c230-197">The application code does this using the `RateProduct` command shown in the following code.</span></span>

```csharp
public interface ICommand
{
  Guid Id { get; }
}

public class RateProduct : ICommand
{
  public RateProduct()
  {
    this.Id = Guid.NewGuid();
  }
  public Guid Id { get; set; }
  public int ProductId { get; set; }
  public int Rating { get; set; }
  public int UserId {get; set; }
}
```

<span data-ttu-id="2c230-198">O sistema utiliza a classe `ProductsCommandHandler` para lidar com comandos enviados pelo aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2c230-198">The system uses the `ProductsCommandHandler` class to handle commands sent by the application.</span></span> <span data-ttu-id="2c230-199">Normalmente, os clientes enviam comandos para o domínio através de um sistema de mensagens, como uma fila.</span><span class="sxs-lookup"><span data-stu-id="2c230-199">Clients typically send commands to the domain through a messaging system such as a queue.</span></span> <span data-ttu-id="2c230-200">O manipulador de comando aceita esses comandos e invoca métodos da interface de domínio.</span><span class="sxs-lookup"><span data-stu-id="2c230-200">The command handler accepts these commands and invokes methods of the domain interface.</span></span> <span data-ttu-id="2c230-201">A granularidade de cada comando é projetada para reduzir a chance de solicitações conflitantes.</span><span class="sxs-lookup"><span data-stu-id="2c230-201">The granularity of each command is designed to reduce the chance of conflicting requests.</span></span> <span data-ttu-id="2c230-202">O código a seguir mostra uma estrutura de tópicos da classe `ProductsCommandHandler`.</span><span class="sxs-lookup"><span data-stu-id="2c230-202">The following code shows an outline of the `ProductsCommandHandler` class.</span></span>

```csharp
public class ProductsCommandHandler :
    ICommandHandler<AddNewProduct>,
    ICommandHandler<RateProduct>,
    ICommandHandler<AddToInventory>,
    ICommandHandler<ConfirmItemShipped>,
    ICommandHandler<UpdateStockFromInventoryRecount>
{
  private readonly IRepository<Product> repository;

  public ProductsCommandHandler (IRepository<Product> repository)
  {
    this.repository = repository;
  }

  void Handle (AddNewProduct command)
  {
    ...
  }

  void Handle (RateProduct command)
  {
    var product = repository.Find(command.ProductId);
    if (product != null)
    {
      product.RateProduct(command.UserId, command.Rating);
      repository.Save(product);
    }
  }

  void Handle (AddToInventory command)
  {
    ...
  }

  void Handle (ConfirmItemsShipped command)
  {
    ...
  }

  void Handle (UpdateStockFromInventoryRecount command)
  {
    ...
  }
}
```

<span data-ttu-id="2c230-203">O código a seguir mostra a interface `IProductsDomain` do modelo de gravação.</span><span class="sxs-lookup"><span data-stu-id="2c230-203">The following code shows the `IProductsDomain` interface from the write model.</span></span>

```csharp
public interface IProductsDomain
{
  void AddNewProduct(int id, string name, string description, decimal price);
  void RateProduct(int userId, int rating);
  void AddToInventory(int productId, int quantity);
  void ConfirmItemsShipped(int productId, int quantity);
  void UpdateStockFromInventoryRecount(int productId, int updatedQuantity);
}
```

<span data-ttu-id="2c230-204">Observe também como a `IProductsDomain` interface contém métodos que têm um significado no domínio.</span><span class="sxs-lookup"><span data-stu-id="2c230-204">Also notice how the `IProductsDomain` interface contains methods that have a meaning in the domain.</span></span> <span data-ttu-id="2c230-205">Normalmente, em um ambiente CRUD, esses métodos teriam nomes genéricos, como `Save` ou `Update`, e um DTO como o único argumento.</span><span class="sxs-lookup"><span data-stu-id="2c230-205">Typically, in a CRUD environment these methods would have generic names such as `Save` or `Update`, and have a DTO as the only argument.</span></span> <span data-ttu-id="2c230-206">A abordagem CQRS pode ser projetada para atender às necessidades dos sistemas de gerenciamento de estoque e de negócios dessa organização.</span><span class="sxs-lookup"><span data-stu-id="2c230-206">The CQRS approach can be designed to meet the needs of this organization's business and inventory management systems.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="2c230-207">Diretrizes e padrões relacionados</span><span class="sxs-lookup"><span data-stu-id="2c230-207">Related patterns and guidance</span></span>

<span data-ttu-id="2c230-208">Os seguintes padrões e diretrizes serão úteis ao implementar esse padrão:</span><span class="sxs-lookup"><span data-stu-id="2c230-208">The following patterns and guidance are useful when implementing this pattern:</span></span>

- <span data-ttu-id="2c230-209">Para uma comparação de CQRS com outros estilos arquitetônicos, consulte [Estilos de arquitetura](/azure/architecture/guide/architecture-styles/) e [Estilo de arquitetura CQRS](/azure/architecture/guide/architecture-styles/cqrs).</span><span class="sxs-lookup"><span data-stu-id="2c230-209">For a comparison of CQRS with other architectural styles, see [Architecture styles](/azure/architecture/guide/architecture-styles/) and [CQRS architecture style](/azure/architecture/guide/architecture-styles/cqrs).</span></span>

- <span data-ttu-id="2c230-210">[Primer de Consistência de Dados](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="2c230-210">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="2c230-211">Explica os problemas que normalmente são encontrados devido à coerência eventual entre os repositórios de dados de gravação e leitura ao utilizar o padrão CQRS e como esses problemas podem ser resolvidos.</span><span class="sxs-lookup"><span data-stu-id="2c230-211">Explains the issues that are typically encountered due to eventual consistency between the read and write data stores when using the CQRS pattern, and how these issues can be resolved.</span></span>

- <span data-ttu-id="2c230-212">[Diretrizes de Particionamento de Dados](https://msdn.microsoft.com/library/dn589795.aspx).</span><span class="sxs-lookup"><span data-stu-id="2c230-212">[Data Partitioning Guidance](https://msdn.microsoft.com/library/dn589795.aspx).</span></span> <span data-ttu-id="2c230-213">Descreve como os repositórios de dados de gravação e leitura usados no padrão CQRS podem ser divididos em partições que podem ser gerenciadas e acessadas separadamente para melhorar a escalabilidade, reduzir a contenção e otimizar o desempenho.</span><span class="sxs-lookup"><span data-stu-id="2c230-213">Describes how the read and write data stores used in the CQRS pattern can be divided into partitions that can be managed and accessed separately to improve scalability, reduce contention, and optimize performance.</span></span>

- <span data-ttu-id="2c230-214">[Padrão de Fornecimento de Evento](event-sourcing.md).</span><span class="sxs-lookup"><span data-stu-id="2c230-214">[Event Sourcing Pattern](event-sourcing.md).</span></span> <span data-ttu-id="2c230-215">Descreve detalhadamente como o Fornecimento de Eventos pode ser utilizado com o padrão CQRS para simplificar tarefas em domínios complexos, ao mesmo tempo em que melhora o desempenho, a escalabilidade e capacidade de resposta.</span><span class="sxs-lookup"><span data-stu-id="2c230-215">Describes in more detail how Event Sourcing can be used with the CQRS pattern to simplify tasks in complex domains while improving performance, scalability, and responsiveness.</span></span> <span data-ttu-id="2c230-216">Além disso, como fornecer consistência para dados transacionais, ao mesmo tempo que mantém trilhas de auditoria completas e histórico que podem permitir ações de compensação.</span><span class="sxs-lookup"><span data-stu-id="2c230-216">As well as how to provide consistency for transactional data while maintaining full audit trails and history that can enable compensating actions.</span></span>

- <span data-ttu-id="2c230-217">[Padrão de Exibição Materializada](materialized-view.md).</span><span class="sxs-lookup"><span data-stu-id="2c230-217">[Materialized View Pattern](materialized-view.md).</span></span> <span data-ttu-id="2c230-218">O modelo de leitura de uma implementação CQRS pode conter exibições materializadas dos dados do modelo de gravação, ou o modelo de leitura pode ser utilizado para gerar exibições materializadas.</span><span class="sxs-lookup"><span data-stu-id="2c230-218">The read model of a CQRS implementation can contain materialized views of the write model data, or the read model can be used to generate materialized views.</span></span>

- <span data-ttu-id="2c230-219">Guia de padrões e práticas [Recurso CQRS](http://aka.ms/cqrs).</span><span class="sxs-lookup"><span data-stu-id="2c230-219">The patterns & practices guide [CQRS Journey](http://aka.ms/cqrs).</span></span> <span data-ttu-id="2c230-220">Em particular, a [Introdução ao Padrão de Segregação de Responsabilidade de Consulta de Comando](https://msdn.microsoft.com/library/jj591573.aspx) explora o padrão e quando isso é útil e o [Epílogo: Lições Aprendidas](https://msdn.microsoft.com/library/jj591568.aspx) ajuda-o a entender alguns dos problemas que surgiram ao usar este padrão.</span><span class="sxs-lookup"><span data-stu-id="2c230-220">In particular, [Introducing the Command Query Responsibility Segregation Pattern](https://msdn.microsoft.com/library/jj591573.aspx) explores the pattern and when it's useful, and [Epilogue: Lessons Learned](https://msdn.microsoft.com/library/jj591568.aspx) helps you understand some of the issues that come up when using this pattern.</span></span>

- <span data-ttu-id="2c230-221">A postagem [CQRS por Martin Fowler](http://martinfowler.com/bliki/CQRS.html), que explica os conceitos básicos do padrão e links para outros recursos úteis.</span><span class="sxs-lookup"><span data-stu-id="2c230-221">The post [CQRS by Martin Fowler](http://martinfowler.com/bliki/CQRS.html), which explains the basics of the pattern and links to other useful resources.</span></span>

- <span data-ttu-id="2c230-222">[Postagens de Greg Young](http://codebetter.com/gregyoung/), que exploram muitos aspectos do padrão CQRS.</span><span class="sxs-lookup"><span data-stu-id="2c230-222">[Greg Young’s posts](http://codebetter.com/gregyoung/), which explore many aspects of the CQRS pattern.</span></span>
