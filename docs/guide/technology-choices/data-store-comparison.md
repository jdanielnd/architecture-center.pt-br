---
title: Critérios para escolher um armazenamento de dados
description: Visão geral das opções de computação do Azure
author: MikeWasson
ms.openlocfilehash: 9cb2f77b854a38450490bc96bf0b6a2998ceb1c7
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
---
# <a name="criteria-for-choosing-a-data-store"></a><span data-ttu-id="bf9b4-103">Critérios para escolher um armazenamento de dados</span><span class="sxs-lookup"><span data-stu-id="bf9b4-103">Criteria for choosing a data store</span></span>

<span data-ttu-id="bf9b4-104">O Azure dá suporte a muitos tipos de soluções de armazenamento de dados, cada um fornecendo recursos e funcionalidades diferentes.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-104">Azure supports many types of data storage solutions, each providing different features and capabilities.</span></span> <span data-ttu-id="bf9b4-105">Este artigo descreve os critérios de comparação que você deve usar ao avaliar um armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-105">This article describes the comparison criteria you should use when evaluating a data store.</span></span> <span data-ttu-id="bf9b4-106">A meta é ajudar você a determinar quais tipos de armazenamento de dados podem atender aos requisitos da sua solução.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-106">The goal is to help you determine which data storage types can meet your solution's requirements.</span></span>

## <a name="general-considerations"></a><span data-ttu-id="bf9b4-107">Considerações gerais</span><span class="sxs-lookup"><span data-stu-id="bf9b4-107">General Considerations</span></span>

<span data-ttu-id="bf9b4-108">Para iniciar a comparação, reúna o máximo possível das informações a seguir sobre suas necessidades de dados.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-108">To start your comparison, gather as much of the following information as you can about your data needs.</span></span> <span data-ttu-id="bf9b4-109">Essas informações ajudarão a determinar quais tipos de armazenamento de dados atenda às suas necessidades.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-109">This information will help you to determine which data storage types will meet your needs.</span></span>

### <a name="functional-requirements"></a><span data-ttu-id="bf9b4-110">Requisitos funcionais</span><span class="sxs-lookup"><span data-stu-id="bf9b4-110">Functional requirements</span></span>

- <span data-ttu-id="bf9b4-111">**Formato de dados**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-111">**Data format**.</span></span> <span data-ttu-id="bf9b4-112">Quais tipos de dados você pretende armazenar?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-112">What type of data are you intending to store?</span></span> <span data-ttu-id="bf9b4-113">Tipos comuns incluem dados transacionais, objetos JSON, telemetria, índices de pesquisa ou arquivos simples.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-113">Common types include transactional data, JSON objects, telemetry, search indexes, or flat files.</span></span>
- <span data-ttu-id="bf9b4-114">**Tamanho dos dados**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-114">**Data size**.</span></span> <span data-ttu-id="bf9b4-115">Qual será o tamanho das entidades que você precisa armazenar?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-115">How large are the entities you need to store?</span></span> <span data-ttu-id="bf9b4-116">Essas entidades precisam ser mantidas como um único documento ou podem ser divididas em vários documentos, tabelas, coleções e assim por diante?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-116">Will these entities need to be maintained as a single document, or can they be split across multiple documents, tables, collections, and so forth?</span></span>
- <span data-ttu-id="bf9b4-117">**Escala e a estrutura**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-117">**Scale and structure**.</span></span> <span data-ttu-id="bf9b4-118">Qual é o volume geral da capacidade de armazenamento necessária?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-118">What is the overall amount of storage capacity you need?</span></span> <span data-ttu-id="bf9b4-119">Você pretende particionar os dados?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-119">Do you anticipate partitioning your data?</span></span> 
- <span data-ttu-id="bf9b4-120">**Relações entre os dados**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-120">**Data relationships**.</span></span> <span data-ttu-id="bf9b4-121">Seus dados precisarão dar suporte a relações um para muitos ou muitos para muitos?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-121">Will your data need to support one-to-many or many-to-many relationships?</span></span> <span data-ttu-id="bf9b4-122">São relações em si são uma parte importante dos dados?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-122">Are relationships themselves an important part of the data?</span></span> <span data-ttu-id="bf9b4-123">Você precisará ingressar ou combinar os dados de dentro do mesmo conjunto de dados ou de conjuntos de dados externos?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-123">Will you need to join or otherwise combine data from within the same dataset, or from external datasets?</span></span> 
- <span data-ttu-id="bf9b4-124">**Modelo de consistência**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-124">**Consistency model**.</span></span> <span data-ttu-id="bf9b4-125">O quão é importante que as atualizações realizadas em um nó apareçam em outros nós antes de permitir alterações adicionais?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-125">How important is it for updates made in one node to appear in other nodes, before further changes can be made?</span></span> <span data-ttu-id="bf9b4-126">É possível aceitar consistência eventual?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-126">Can you accept eventual consistency?</span></span> <span data-ttu-id="bf9b4-127">Você precisa de garantias de ACID para transações?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-127">Do you need ACID guarantees for transactions?</span></span>
- <span data-ttu-id="bf9b4-128">**Flexibilidade do esquema**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-128">**Schema flexibility**.</span></span> <span data-ttu-id="bf9b4-129">Quais tipos de esquemas serão aplicadas aos seus dados?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-129">What kind of schemas will you apply to your data?</span></span> <span data-ttu-id="bf9b4-130">Você usará um esquema fixo, uma abordagem de esquema na gravação ou de esquema na leitura?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-130">Will you use a fixed schema, a schema-on-write approach, or a schema-on-read approach?</span></span>
- <span data-ttu-id="bf9b4-131">**Simultaneidade**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-131">**Concurrency**.</span></span> <span data-ttu-id="bf9b4-132">Qual tipo de mecanismo de simultaneidade você deseja usar para atualizar e sincronizar os dados?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-132">What kind of concurrency mechanism do you want to use when updating and synchronizing data?</span></span> <span data-ttu-id="bf9b4-133">O aplicativo executará muitas atualizações que poderiam entrar em conflito.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-133">Will the application perform many updates that could potentially conflict.</span></span> <span data-ttu-id="bf9b4-134">Nesse caso, é possível exigir o bloqueio de registros e o controle de simultaneidade pessimista.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-134">If so, you may requiring record locking and pessimistic concurrency control.</span></span> <span data-ttu-id="bf9b4-135">Como alternativa, você pode dar suporte a controles de simultaneidade otimista?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-135">Alternatively, can you support optimistic concurrency controls?</span></span> <span data-ttu-id="bf9b4-136">Nesse caso, é simples o suficiente controlar a simultaneidade baseada em carimbo de data/hora ou você precisa de funcionalidade adicional de controle de simultaneidade de várias versões?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-136">If so, is simple timestamp-based concurrency control enough, or do you need the added functionality of multi-version concurrency control?</span></span>
- <span data-ttu-id="bf9b4-137">**Movimentação de dados**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-137">**Data movement**.</span></span> <span data-ttu-id="bf9b4-138">Sua solução precisará executar tarefas ETL para mover os dados para outros armazenamentos ou data warehouses?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-138">Will your solution need to perform ETL tasks to move data to other stores or data warehouses?</span></span>
- <span data-ttu-id="bf9b4-139">**Ciclo de vida dos dados**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-139">**Data lifecycle**.</span></span> <span data-ttu-id="bf9b4-140">A gravação dos ocorre uma vez, com muitas leituras?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-140">Is the data write-once, read-many?</span></span> <span data-ttu-id="bf9b4-141">Eles podem ser movidos para o armazenamento frequente ou esporádico?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-141">Can it be moved into cool or cold storage?</span></span>
- <span data-ttu-id="bf9b4-142">**Outros recursos com suporte**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-142">**Other supported features**.</span></span> <span data-ttu-id="bf9b4-143">Você precisa de algum outro recurso específico, como validação de esquema, agregação, indexação, pesquisa de texto completo, MapReduce ou outras funcionalidades de consulta?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-143">Do you need any other specific features, such as schema validation, aggregation, indexing, full-text search, MapReduce, or other query capabilities?</span></span>

### <a name="non-functional-requirements"></a><span data-ttu-id="bf9b4-144">Requisitos não funcionais</span><span class="sxs-lookup"><span data-stu-id="bf9b4-144">Non-functional requirements</span></span>

- <span data-ttu-id="bf9b4-145">**Desempenho e escalabilidade**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-145">**Performance and scalability**.</span></span> <span data-ttu-id="bf9b4-146">Quais são seus requisitos de desempenho de dados?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-146">What are your data performance requirements?</span></span> <span data-ttu-id="bf9b4-147">Você tem requisitos específicos para as taxas de ingestão de dados e taxas de processamento de dados?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-147">Do you have specific requirements for data ingestion rates and data processing rates?</span></span> <span data-ttu-id="bf9b4-148">Quais são os tempos de resposta aceitáveis para consulta e agregação dos dados depois de ingeridos?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-148">What are the acceptable response times for querying and aggregation of data once ingested?</span></span> <span data-ttu-id="bf9b4-149">Até que tamanho você precisará aumentar o armazenamento de dados?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-149">How large will you need the data store to scale up?</span></span> <span data-ttu-id="bf9b4-150">Sua carga de trabalho tem leituras ou gravações mais frequentes?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-150">Is your workload more read-heavy or write-heavy?</span></span>
- <span data-ttu-id="bf9b4-151">**Confiabilidade**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-151">**Reliability**.</span></span> <span data-ttu-id="bf9b4-152">A qual SLA geral você precisa dar suporte?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-152">What overall SLA do you need to support?</span></span> <span data-ttu-id="bf9b4-153">Qual nível de tolerância a falhas você precisa fornecer para os consumidores dos dados?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-153">What level of fault-tolerance do you need to provide for data consumers?</span></span> <span data-ttu-id="bf9b4-154">De quais tipos de funcionalidades de backup e restauração você precisa?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-154">What kind of backup and restore capabilities do you need?</span></span> 
- <span data-ttu-id="bf9b4-155">**Replicação**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-155">**Replication**.</span></span> <span data-ttu-id="bf9b4-156">Os dados precisarão ser distribuído entre várias réplicas ou regiões?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-156">Will your data need to be distributed among multiple replicas or regions?</span></span> <span data-ttu-id="bf9b4-157">De quais tipos de funcionalidades de replicação de dados você precisa?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-157">What kind of data replication capabilities do you require?</span></span> 
- <span data-ttu-id="bf9b4-158">**Limites**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-158">**Limits**.</span></span> <span data-ttu-id="bf9b4-159">Os limites de um armazenamento de dados específico dará suporte aos seus requisitos de escala, número de conexões e taxa de transferência?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-159">Will the limits of a particular data store support your requirements for scale, number of connections, and throughput?</span></span> 

### <a name="management-and-cost"></a><span data-ttu-id="bf9b4-160">Gerenciamento e o custo</span><span class="sxs-lookup"><span data-stu-id="bf9b4-160">Management and cost</span></span>

- <span data-ttu-id="bf9b4-161">**Serviço gerenciado**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-161">**Managed service**.</span></span> <span data-ttu-id="bf9b4-162">Quando possível, use um serviço de dados gerenciados, a menos que você precise de funcionalidades específicas encontradas somente em um armazenamento de dados hospedado por IaaS.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-162">When possible, use a managed data service, unless you require specific capabilities that can only be found in an IaaS-hosted data store.</span></span>
- <span data-ttu-id="bf9b4-163">**Disponibilidade de região**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-163">**Region availability**.</span></span> <span data-ttu-id="bf9b4-164">Para serviços gerenciais, o serviço está disponível em todas as regiões do Azure?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-164">For managed services, is the service available in all Azure regions?</span></span> <span data-ttu-id="bf9b4-165">Sua solução precisa ser hospedada em determinadas regiões do Azure?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-165">Does your solution need to be hosted in certain Azure regions?</span></span>
- <span data-ttu-id="bf9b4-166">**Portabilidade**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-166">**Portability**.</span></span> <span data-ttu-id="bf9b4-167">Seus dados precisarão ser migrados para o local, datacenters externos ou outros ambientes de hospedagem em nuvem?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-167">Will your data need to migrated to on-premises, external datacenters, or other cloud hosting environments?</span></span>
- <span data-ttu-id="bf9b4-168">**Licenciamento**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-168">**Licensing**.</span></span> <span data-ttu-id="bf9b4-169">Você tem preferência pelo tipo de licença proprietária em comparação a OSS?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-169">Do you have a preference of a proprietary versus OSS license type?</span></span> <span data-ttu-id="bf9b4-170">Há outras restrições externas sobre o tipo de licença que você pode usar?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-170">Are there any other external restrictions on what type of license you can use?</span></span>
- <span data-ttu-id="bf9b4-171">**Custo geral**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-171">**Overall cost**.</span></span> <span data-ttu-id="bf9b4-172">Qual é o custo geral de usar o serviço na sua solução?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-172">What is the overall cost of using the service within your solution?</span></span> <span data-ttu-id="bf9b4-173">Quantas instâncias você precisará executar para dar suporte aos seus requisitos de tempo de atividade e taxa de transferência?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-173">How many instances will need to run, to support your uptime and throughput requirements?</span></span> <span data-ttu-id="bf9b4-174">Considere os custos das operações neste cálculo.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-174">Consider operations costs in this calculation.</span></span> <span data-ttu-id="bf9b4-175">Um motivo para preferir serviços gerenciados é a redução dos custos operacionais.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-175">One reason to prefer managed services is the reduced operational cost.</span></span>
- <span data-ttu-id="bf9b4-176">**Economia de custos**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-176">**Cost effectiveness**.</span></span> <span data-ttu-id="bf9b4-177">Você pode particionar seus dados para armazená-los de forma mais econômica?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-177">Can you partition your data, to store it more cost effectively?</span></span> <span data-ttu-id="bf9b4-178">Por exemplo, você pode mover grandes objetos para fora de um banco de dados relacional caro e para um repositório de objetos?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-178">For example, can you move large objects out of an expensive relational database into an object store?</span></span>

### <a name="security"></a><span data-ttu-id="bf9b4-179">Segurança</span><span class="sxs-lookup"><span data-stu-id="bf9b4-179">Security</span></span>

- <span data-ttu-id="bf9b4-180">**Segurança**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-180">**Security**.</span></span> <span data-ttu-id="bf9b4-181">Qual tipo de criptografia é necessária?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-181">What type of encryption do you require?</span></span> <span data-ttu-id="bf9b4-182">Você precisa de criptografia em repouso?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-182">Do you need encryption at rest?</span></span> <span data-ttu-id="bf9b4-183">Qual mecanismo de autenticação você deseja usar para se conectar aos seus dados?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-183">What authentication mechanism do you want to use to connect to your data?</span></span>
- <span data-ttu-id="bf9b4-184">**Auditoria**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-184">**Auditing**.</span></span> <span data-ttu-id="bf9b4-185">Qual tipo de log de auditoria você precisa gerar?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-185">What kind of audit log do you need to generate?</span></span>
- <span data-ttu-id="bf9b4-186">**Requisitos de rede**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-186">**Networking requirements**.</span></span> <span data-ttu-id="bf9b4-187">Você precisa restringir ou gerenciar o acesso aos dados de outros recursos de rede?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-187">Do you need to restrict or otherwise manage access to your data from other network resources?</span></span> <span data-ttu-id="bf9b4-188">Os dados precisam ser acessíveis somente de dentro do ambiente do Azure?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-188">Does data need to be accessible only from inside the Azure environment?</span></span> <span data-ttu-id="bf9b4-189">Os dados precisam ser acessíveis de endereços IP ou sub-redes específicas?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-189">Does the data need to be accessible from specific IP addresses or subnets?</span></span> <span data-ttu-id="bf9b4-190">Eles precisam ser acessíveis de aplicativos ou serviços hospedados localmente ou de outros datacenters externos?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-190">Does it need to be accessible from applications or services hosted on-premises or in other external datacenters?</span></span>

### <a name="devops"></a><span data-ttu-id="bf9b4-191">DevOps</span><span class="sxs-lookup"><span data-stu-id="bf9b4-191">DevOps</span></span>

- <span data-ttu-id="bf9b4-192">**Conjunto de qualificações**.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-192">**Skill set**.</span></span> <span data-ttu-id="bf9b4-193">Há determinadas linguagens de programação, sistemas operacionais ou outras tecnologias que sua equipe tem experiência específica em usar?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-193">Are there particular programming languages, operating systems, or other technology that your team is particularly adept at using?</span></span> <span data-ttu-id="bf9b4-194">Há outras com as quais seria difícil sua equipe trabalhar?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-194">Are there others that would be difficult for your team to work with?</span></span>
- <span data-ttu-id="bf9b4-195">**Clientes** Há suporte de cliente adequado para suas linguagens de desenvolvimento?</span><span class="sxs-lookup"><span data-stu-id="bf9b4-195">**Clients** Is there good client support for your development languages?</span></span>

<span data-ttu-id="bf9b4-196">As seções a seguir comparam vários modelos de armazenamento de dados com relação ao perfil de carga de trabalho, tipos de dados e casos de uso de exemplo.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-196">The following sections compare various data store models in terms of workload profile, data types, and example use cases.</span></span>

## <a name="relational-database-management-systems-rdbms"></a><span data-ttu-id="bf9b4-197">RDBMS</span><span class="sxs-lookup"><span data-stu-id="bf9b4-197">Relational database management systems (RDBMS)</span></span>

<table>
<tr><td><span data-ttu-id="bf9b4-198"><strong>Carga de trabalho</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-198"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-199">Tanto a criação de novos registros quanto atualizações em dados existentes ocorrem regularmente.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-199">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="bf9b4-200">Várias operações precisam ser concluídas em uma única transação.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-200">Multiple operations have to be completed in a single transaction.</span></span></li>
            <li><span data-ttu-id="bf9b4-201">Requer funções de agregação para executar tabulação cruzada.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-201">Requires aggregation functions to perform cross-tabulation.</span></span></li>
            <li><span data-ttu-id="bf9b4-202">Forte integração com ferramentas de relatório é necessária.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-202">Strong integration with reporting tools is required.</span></span></li>
            <li><span data-ttu-id="bf9b4-203">As relações são impostas usando restrições de banco de dados.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-203">Relationships are enforced using database constraints.</span></span></li>
            <li><span data-ttu-id="bf9b4-204">Os índices são usados para otimizar o desempenho da consulta.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-204">Indexes are used to optimize query performance.</span></span></li>
            <li><span data-ttu-id="bf9b4-205">Permite acesso a subconjuntos específicos de dados.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-205">Allows access to specific subsets of data.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-206"><strong>Tipo de dados</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-206"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-207">Os dados são altamente normalizados.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-207">Data is highly normalized.</span></span></li>
            <li><span data-ttu-id="bf9b4-208">Esquemas de banco de dados são necessários e impostos.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-208">Database schemas are required and enforced.</span></span></li>
            <li><span data-ttu-id="bf9b4-209">Relações muitos para muitos entre as entidades de dados no banco de dados.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-209">Many-to-many relationships between data entities in the database.</span></span></li>
            <li><span data-ttu-id="bf9b4-210">As restrições são definidas no esquema e impostas nos dados no banco de dados.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-210">Constraints are defined in the schema and imposed on any data in the database.</span></span></li>
            <li><span data-ttu-id="bf9b4-211">Os dados exigem alta integridade.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-211">Data requires high integrity.</span></span> <span data-ttu-id="bf9b4-212">Os índices e as relações precisam ser mantidos com precisão.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-212">Indexes and relationships need to be maintained accurately.</span></span></li>
            <li><span data-ttu-id="bf9b4-213">Os dados requerem consistência forte.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-213">Data requires strong consistency.</span></span> <span data-ttu-id="bf9b4-214">As transações operam de forma a garantir que todos os dados sejam 100% consistentes para todos os usuários e processos.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-214">Transactions operate in a way that ensures all data are 100% consistent for all users and processes.</span></span></li>
            <li><span data-ttu-id="bf9b4-215">O tamanho das entradas de dados individuais deve ser de pequeno a médio porte.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-215">Size of individual data entries is intended to be small to medium-sized.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-216"><strong>Exemplos</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-216"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-217">Linha de negócios (gerenciamento de capital humano, gerenciamento de relacionamento com o cliente, planejamento de recursos da empresa)</span><span class="sxs-lookup"><span data-stu-id="bf9b4-217">Line of business  (human capital management, customer relationship management, enterprise resource planning)</span></span></li>
            <li><span data-ttu-id="bf9b4-218">Gerenciamento de estoque</span><span class="sxs-lookup"><span data-stu-id="bf9b4-218">Inventory management</span></span></li>
            <li><span data-ttu-id="bf9b4-219">Banco de dados de relatórios</span><span class="sxs-lookup"><span data-stu-id="bf9b4-219">Reporting database</span></span></li>
            <li><span data-ttu-id="bf9b4-220">Contabilidade</span><span class="sxs-lookup"><span data-stu-id="bf9b4-220">Accounting</span></span></li>
            <li><span data-ttu-id="bf9b4-221">Gerenciamento de ativos</span><span class="sxs-lookup"><span data-stu-id="bf9b4-221">Asset management</span></span></li>
            <li><span data-ttu-id="bf9b4-222">Gerenciamento de fundos</span><span class="sxs-lookup"><span data-stu-id="bf9b4-222">Fund management</span></span></li>
            <li><span data-ttu-id="bf9b4-223">Gerenciamento de pedidos</span><span class="sxs-lookup"><span data-stu-id="bf9b4-223">Order management</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a><span data-ttu-id="bf9b4-224">Bancos de dados de documentos</span><span class="sxs-lookup"><span data-stu-id="bf9b4-224">Document databases</span></span>

<table>
<tr><td><span data-ttu-id="bf9b4-225"><strong>Carga de trabalho</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-225"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-226">Uso geral.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-226">General purpose.</span></span></li>
            <li><span data-ttu-id="bf9b4-227">As operações de inserção e atualização são comuns.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-227">Insert and update operations are common.</span></span> <span data-ttu-id="bf9b4-228">Tanto a criação de novos registros quanto atualizações em dados existentes ocorrem regularmente.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-228">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="bf9b4-229">Não há incompatibilidade de impedância relacional de objeto.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-229">No object-relational impedance mismatch.</span></span> <span data-ttu-id="bf9b4-230">Os documentos correspondem melhor às estruturas de objeto usadas no código do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-230">Documents can better match the object structures used in application code.</span></span></li>
            <li><span data-ttu-id="bf9b4-231">A simultaneidade otimista é usada com mais frequência.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-231">Optimistic concurrency is more commonly used.</span></span></li>
            <li><span data-ttu-id="bf9b4-232">Os dados precisam ser modificados e processados pelo aplicativo de consumo.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-232">Data must be modified and processed by consuming application.</span></span></li>
            <li><span data-ttu-id="bf9b4-233">Os dados requerem um índice de vários campos.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-233">Data requires index on multiple fields.</span></span></li>
            <li><span data-ttu-id="bf9b4-234">Os documentos individuais são recuperados e gravados como um único bloco.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-234">Individual documents are retrieved and written as a single block.</span></span></li>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-235"><strong>Tipo de dados</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-235"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-236">Os dados podem ser gerenciados de forma desordenada.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-236">Data can be managed in de-normalized way.</span></span></li>
            <li><span data-ttu-id="bf9b4-237">O tamanho dos dados do documento individual é relativamente pequeno.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-237">Size of individual document data is relatively small.</span></span></li>
            <li><span data-ttu-id="bf9b4-238">Cada tipo de documento pode usar seu próprio esquema.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-238">Each document type can use its own schema.</span></span></li>
            <li><span data-ttu-id="bf9b4-239">Os documentos podem incluir campos opcionais.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-239">Documents can include optional fields.</span></span></li>
            <li><span data-ttu-id="bf9b4-240">Os dados de documento são semi-estruturados, o que significa que os tipos de dados de cada campo não são estritamente definidos.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-240">Document data is semi-structured, meaning that data types of each field are not strictly defined.</span></span></li>
            <li><span data-ttu-id="bf9b4-241">Há suporte para a agregação de dados.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-241">Data aggregation is supported.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-242"><strong>Exemplos</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-242"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-243">Catálogo de produtos</span><span class="sxs-lookup"><span data-stu-id="bf9b4-243">Product catalog</span></span></li>
            <li><span data-ttu-id="bf9b4-244">Contas de usuário</span><span class="sxs-lookup"><span data-stu-id="bf9b4-244">User accounts</span></span></li>
            <li><span data-ttu-id="bf9b4-245">Lista de materiais</span><span class="sxs-lookup"><span data-stu-id="bf9b4-245">Bill of materials</span></span></li>
            <li><span data-ttu-id="bf9b4-246">Personalização</span><span class="sxs-lookup"><span data-stu-id="bf9b4-246">Personalization</span></span></li>
            <li><span data-ttu-id="bf9b4-247">Gerenciamento de conteúdo</span><span class="sxs-lookup"><span data-stu-id="bf9b4-247">Content management</span></span></li>
            <li><span data-ttu-id="bf9b4-248">Dados de operações</span><span class="sxs-lookup"><span data-stu-id="bf9b4-248">Operations data</span></span></li>
            <li><span data-ttu-id="bf9b4-249">Gerenciamento de estoque</span><span class="sxs-lookup"><span data-stu-id="bf9b4-249">Inventory management</span></span></li>
            <li><span data-ttu-id="bf9b4-250">Dados do histórico de transação</span><span class="sxs-lookup"><span data-stu-id="bf9b4-250">Transaction history data</span></span></li>
            <li><span data-ttu-id="bf9b4-251">Exibição materializada de outros repositórios NoSQL.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-251">Materialized view of other NoSQL stores.</span></span> <span data-ttu-id="bf9b4-252">Substitui a indexação de arquivo/BLOB.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-252">Replaces file/BLOB indexing.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a><span data-ttu-id="bf9b4-253">Armazenamentos de valor/chave</span><span class="sxs-lookup"><span data-stu-id="bf9b4-253">Key/value stores</span></span>

<table>
<tr><td><span data-ttu-id="bf9b4-254"><strong>Carga de trabalho</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-254"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-255">Os dados são identificados e acessados usando uma única chave de ID, como um dicionário.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-255">Data is identified and accessed using a single ID key, like a dictionary.</span></span></li>
            <li><span data-ttu-id="bf9b4-256">Amplamente escalonável.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-256">Massively scalable.</span></span></li>
            <li><span data-ttu-id="bf9b4-257">Associações, bloqueios ou uniões não são necessários.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-257">No joins, lock, or unions are required.</span></span></li>
            <li><span data-ttu-id="bf9b4-258">Mecanismos de agregação não são usados.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-258">No aggregation mechanisms are used.</span></span></li>
            <li><span data-ttu-id="bf9b4-259">Os índices secundários geralmente não são usados.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-259">Secondary indexes are generally not used.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-260"><strong>Tipo de dados</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-260"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-261">O tamanho dos dados tende a ser grande.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-261">Data size tends to be large.</span></span></li>
            <li><span data-ttu-id="bf9b4-262">Cada chave é associada a um único valor, que é um BLOB de dados não gerenciado.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-262">Each key is associated with a single value, which is an unmanaged data BLOB.</span></span></li>
            <li><span data-ttu-id="bf9b4-263">Não há imposição do esquema.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-263">There is no schema enforcement.</span></span></li>
            <li><span data-ttu-id="bf9b4-264">Não há relações entre entidades.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-264">No relationships between entities.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-265"><strong>Exemplos</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-265"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-266">Armazenamento de dados em cache</span><span class="sxs-lookup"><span data-stu-id="bf9b4-266">Data caching</span></span></li>
            <li><span data-ttu-id="bf9b4-267">Gerenciamento da sessão</span><span class="sxs-lookup"><span data-stu-id="bf9b4-267">Session management</span></span></li>
            <li><span data-ttu-id="bf9b4-268">Preferência do usuário e gerenciamento de perfil</span><span class="sxs-lookup"><span data-stu-id="bf9b4-268">User preference and profile management</span></span></li>
            <li><span data-ttu-id="bf9b4-269">Recomendação de produtos e veiculação de anúncios</span><span class="sxs-lookup"><span data-stu-id="bf9b4-269">Product recommendation and ad serving</span></span></li>
            <li><span data-ttu-id="bf9b4-270">Dicionários</span><span class="sxs-lookup"><span data-stu-id="bf9b4-270">Dictionaries</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a><span data-ttu-id="bf9b4-271">Bancos de dados de grafo</span><span class="sxs-lookup"><span data-stu-id="bf9b4-271">Graph databases</span></span>

<table>
<tr><td><span data-ttu-id="bf9b4-272"><strong>Carga de trabalho</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-272"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-273">As relações entre os itens de dados são muito complexas, o que envolve vários saltos entre os itens de dados relacionados.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-273">The relationships between data items are very complex, involving many hops between related data items.</span></span></li>
            <li><span data-ttu-id="bf9b4-274">A relação entre os itens de dados são dinâmicas e mudam com o passar do tempo.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-274">The relationship between data items are dynamic and change over time.</span></span></li>
            <li><span data-ttu-id="bf9b4-275">As relações entre objetos são de primeira classe, sem precisar de chaves estrangeiras e ingressos para serem percorridas.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-275">Relationships between objects are first-class citizens, without requiring foreign-keys and joins to traverse.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-276"><strong>Tipo de dados</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-276"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-277">Os dados são compostos por nós e relações.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-277">Data is comprised of nodes and relationships.</span></span></li>
            <li><span data-ttu-id="bf9b4-278">Nós são semelhantes às linhas da tabela ou documentos JSON.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-278">Nodes are similar to table rows or JSON documents.</span></span></li>
            <li><span data-ttu-id="bf9b4-279">As relações são tão importantes quanto os nós e são expostas diretamente na linguagem de consulta.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-279">Relationships are just as important as nodes, and are exposed directly in the query language.</span></span></li>
            <li><span data-ttu-id="bf9b4-280">Objetos de composição, como uma pessoa com vários números de telefone, tendem a ser divididos em nós menores combinado com relações navegáveis</span><span class="sxs-lookup"><span data-stu-id="bf9b4-280">Composite objects, such as a person with multiple phone numbers, tend to be broken into separate, smaller nodes, combined with traversable relationships</span></span> </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-281"><strong>Exemplos</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-281"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-282">Quadros da organização</span><span class="sxs-lookup"><span data-stu-id="bf9b4-282">Organization charts</span></span></li>
            <li><span data-ttu-id="bf9b4-283">Gráficos sociais</span><span class="sxs-lookup"><span data-stu-id="bf9b4-283">Social graphs</span></span></li>
            <li><span data-ttu-id="bf9b4-284">Detecção de fraude</span><span class="sxs-lookup"><span data-stu-id="bf9b4-284">Fraud detection</span></span></li>
            <li><span data-ttu-id="bf9b4-285">Análise</span><span class="sxs-lookup"><span data-stu-id="bf9b4-285">Analytics</span></span></li>
            <li><span data-ttu-id="bf9b4-286">Mecanismos de recomendação</span><span class="sxs-lookup"><span data-stu-id="bf9b4-286">Recommendation engines</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a><span data-ttu-id="bf9b4-287">Bancos de dados de família de coluna</span><span class="sxs-lookup"><span data-stu-id="bf9b4-287">Column-family databases</span></span>

<table>
<tr><td><span data-ttu-id="bf9b4-288"><strong>Carga de trabalho</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-288"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-289">A maioria dos bancos de dados de famílias de colunas executam operações de gravação muito rapidamente.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-289">Most column-family databases perform write operations extremely quickly.</span></span></li>
            <li><span data-ttu-id="bf9b4-290">As operações de atualização e exclusão são raras.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-290">Update and delete operations are rare.</span></span></li>
            <li><span data-ttu-id="bf9b4-291">Projetado para fornecer acesso com alta taxa de transferência e baixa latência.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-291">Designed to provide high throughput and low-latency access.</span></span></li>
            <li><span data-ttu-id="bf9b4-292">Dá suporte ao acesso de consulta fácil para determinado conjunto de campos dentro de um registro muito maior.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-292">Supports easy query access to a particular set of fields within a much larger record.</span></span></li>
            <li><span data-ttu-id="bf9b4-293">Amplamente escalonável.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-293">Massively scalable.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-294"><strong>Tipo de dados</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-294"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-295">Os dados são armazenados em tabelas compostas por uma coluna de chave e uma ou mais famílias de colunas.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-295">Data is stored in tables consisting of a key column and one or more column families.</span></span></li>
            <li><span data-ttu-id="bf9b4-296">As colunas específicas podem variar conforme as linhas individuais.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-296">Specific columns can vary by individual rows.</span></span></li>
            <li><span data-ttu-id="bf9b4-297">As células individuais são acessadas por meio de comandos get e put</span><span class="sxs-lookup"><span data-stu-id="bf9b4-297">Individual cells are accessed via get and put commands</span></span></li>
            <li><span data-ttu-id="bf9b4-298">Várias linhas são retornadas usando um comando de verificação.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-298">Multiple rows are returned using a scan command.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-299"><strong>Exemplos</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-299"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-300">Recomendações</span><span class="sxs-lookup"><span data-stu-id="bf9b4-300">Recommendations</span></span></li>
            <li><span data-ttu-id="bf9b4-301">Personalização</span><span class="sxs-lookup"><span data-stu-id="bf9b4-301">Personalization</span></span></li>
            <li><span data-ttu-id="bf9b4-302">Dados de sensor</span><span class="sxs-lookup"><span data-stu-id="bf9b4-302">Sensor data</span></span></li>
            <li><span data-ttu-id="bf9b4-303">Telemetria</span><span class="sxs-lookup"><span data-stu-id="bf9b4-303">Telemetry</span></span></li>
            <li><span data-ttu-id="bf9b4-304">Mensagens</span><span class="sxs-lookup"><span data-stu-id="bf9b4-304">Messaging</span></span></li>
            <li><span data-ttu-id="bf9b4-305">Análise de mídia social</span><span class="sxs-lookup"><span data-stu-id="bf9b4-305">Social media analytics</span></span></li>
            <li><span data-ttu-id="bf9b4-306">Análise da Web</span><span class="sxs-lookup"><span data-stu-id="bf9b4-306">Web analytics</span></span></li>
            <li><span data-ttu-id="bf9b4-307">Monitorando de atividades</span><span class="sxs-lookup"><span data-stu-id="bf9b4-307">Activity monitoring</span></span></li>
            <li><span data-ttu-id="bf9b4-308">Previsão do tempo e outros dados de série temporal</span><span class="sxs-lookup"><span data-stu-id="bf9b4-308">Weather and other time-series data</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a><span data-ttu-id="bf9b4-309">Bancos de dados de mecanismo de pesquisa</span><span class="sxs-lookup"><span data-stu-id="bf9b4-309">Search engine databases</span></span>

<table>
<tr><td><span data-ttu-id="bf9b4-310"><strong>Carga de trabalho</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-310"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-311">Indexação de dados de várias fontes e serviços.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-311">Indexing data from multiple sources and services.</span></span></li>
            <li><span data-ttu-id="bf9b4-312">Consultas são ad hoc e podem ser complexas.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-312">Queries are ad-hoc and can be complex.</span></span></li>
            <li><span data-ttu-id="bf9b4-313">Requer agregação.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-313">Requires aggregation.</span></span></li>
            <li><span data-ttu-id="bf9b4-314">Requer pesquisa de texto completo.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-314">Full text search is required.</span></span></li>
            <li><span data-ttu-id="bf9b4-315">Requer consulta ad hoc de autoatendimento.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-315">Ad hoc self-service query is required.</span></span></li>
            <li><span data-ttu-id="bf9b4-316">Requer análise de dados com índice em todos os campos.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-316">Data analysis with index on all fields is required.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-317"><strong>Tipo de dados</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-317"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-318">Dados semi-estruturados ou não estruturados</span><span class="sxs-lookup"><span data-stu-id="bf9b4-318">Semi-structured or unstructured</span></span></li>
            <li><span data-ttu-id="bf9b4-319">Texto</span><span class="sxs-lookup"><span data-stu-id="bf9b4-319">Text</span></span></li>
            <li><span data-ttu-id="bf9b4-320">Texto com referência a dados estruturados</span><span class="sxs-lookup"><span data-stu-id="bf9b4-320">Text with reference to structured data</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-321"><strong>Exemplos</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-321"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-322">Catálogos de produtos</span><span class="sxs-lookup"><span data-stu-id="bf9b4-322">Product catalogs</span></span></li>
            <li><span data-ttu-id="bf9b4-323">Pesquisa de site</span><span class="sxs-lookup"><span data-stu-id="bf9b4-323">Site search</span></span></li>
            <li><span data-ttu-id="bf9b4-324">Registro em log</span><span class="sxs-lookup"><span data-stu-id="bf9b4-324">Logging</span></span></li>
            <li><span data-ttu-id="bf9b4-325">Análise</span><span class="sxs-lookup"><span data-stu-id="bf9b4-325">Analytics</span></span></li>
            <li><span data-ttu-id="bf9b4-326">Sites de compra</span><span class="sxs-lookup"><span data-stu-id="bf9b4-326">Shopping sites</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a><span data-ttu-id="bf9b4-327">Data warehouse</span><span class="sxs-lookup"><span data-stu-id="bf9b4-327">Data warehouse</span></span>

<table>
<tr><td><span data-ttu-id="bf9b4-328"><strong>Carga de trabalho</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-328"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-329">Análise de dados</span><span class="sxs-lookup"><span data-stu-id="bf9b4-329">Data analytics</span></span></li>
            <li><span data-ttu-id="bf9b4-330">Enterprise BI</span><span class="sxs-lookup"><span data-stu-id="bf9b4-330">Enterprise BI</span></span>   </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-331"><strong>Tipo de dados</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-331"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-332">Dados históricos de várias fontes.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-332">Historical data from multiple sources.</span></span></li>
            <li><span data-ttu-id="bf9b4-333">Geralmente desnormalizado em um esquema de &quot;estrela&quot; ou &quot;floco de neve&quot;, consistindo em tabelas de dimensões e fatos.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-333">Usually denormalized in a &quot;star&quot; or &quot;snowflake&quot; schema, consisting of fact and dimension tables.</span></span></li>
            <li><span data-ttu-id="bf9b4-334">Geralmente é carregado com novos dados de forma programada.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-334">Usually loaded with new data on a scheduled basis.</span></span></li>
            <li><span data-ttu-id="bf9b4-335">As tabelas de dimensões geralmente incluem várias versões históricas de uma entidade, conhecida como uma <em>dimensão de alteração lenta</em>.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-335">Dimension tables often include multiple historic versions of an entity, referred to as a <em>slowly changing dimension</em>.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-336"><strong>Exemplos</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-336"><strong>Examples</strong></span></span></td>
    <td><span data-ttu-id="bf9b4-337">Um data warehouse corporativo que fornece dados para os modelos, relatórios e painéis analíticos.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-337">An enterprise data warehouse that provides data for analytical models, reports, and dashboards.</span></span>
    </td>
</tr>
</table>


## <a name="time-series-databases"></a><span data-ttu-id="bf9b4-338">Bancos de dados de séries temporais</span><span class="sxs-lookup"><span data-stu-id="bf9b4-338">Time series databases</span></span>

<table>
<tr><td><span data-ttu-id="bf9b4-339"><strong>Carga de trabalho</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-339"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-340">Um proporção predominante das operações (95% a 99%) são gravações.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-340">An overwhelmingly proportion of operations (95-99%) are writes.</span></span></li>
            <li><span data-ttu-id="bf9b4-341">Os registros são geralmente anexados sequencialmente em ordem cronológica.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-341">Records are generally appended sequentially in time order.</span></span></li>
            <li><span data-ttu-id="bf9b4-342">As atualizações são raras.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-342">Updates are rare.</span></span></li>
            <li><span data-ttu-id="bf9b4-343">As exclusões ocorrem em massa e são realizadas em blocos ou registros contíguos.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-343">Deletes occur in bulk, and are made to contiguous blocks or records.</span></span></li>
            <li><span data-ttu-id="bf9b4-344">As solicitações de leitura podem ser maiores que a memória disponível.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-344">Read requests can be larger than available memory.</span></span></li>
            <li><span data-ttu-id="bf9b4-345">É comum que várias leituras ocorram simultaneamente.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-345">It&#39;s common for multiple reads to occur simultaneously.</span></span></li>
            <li><span data-ttu-id="bf9b4-346">Os dados são lidos em sequência em ordem crescente ou decrescente cronológica.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-346">Data is read sequentially in either ascending or descending time order.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-347"><strong>Tipo de dados</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-347"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-348">Um carimbo de data/hora que é usado a chave primária e o mecanismo de classificação.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-348">A time stamp that is used as the primary key and sorting mechanism.</span></span></li>
            <li><span data-ttu-id="bf9b4-349">Medidas da entrada ou descrições do que a entrada representa.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-349">Measurements from the entry or descriptions of what the entry represents.</span></span></li>
            <li><span data-ttu-id="bf9b4-350">Marcas que definem informações adicionais sobre o tipo, origem e outras informações sobre a entrada.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-350">Tags that define additional information about the type, origin, and other information about the entry.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-351"><strong>Exemplos</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-351"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-352">Monitoramento e telemetria de evento.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-352">Monitoring and event telemetry.</span></span></li>
            <li><span data-ttu-id="bf9b4-353">Sensor ou outros dados de IoT.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-353">Sensor or other IoT data.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a><span data-ttu-id="bf9b4-354">Armazenamento de objetos</span><span class="sxs-lookup"><span data-stu-id="bf9b4-354">Object storage</span></span>

<table>
<tr><td><span data-ttu-id="bf9b4-355"><strong>Carga de trabalho</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-355"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-356">Identificado pela chave.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-356">Identified by key.</span></span></li>
            <li><span data-ttu-id="bf9b4-357">Os objetos podem estar acessíveis de forma pública ou privada.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-357">Objects may be publicly or privately accessible.</span></span></li>
            <li><span data-ttu-id="bf9b4-358">O conteúdo normalmente é um ativo, como uma planilha, uma imagem ou um arquivo de vídeo.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-358">Content is typically an asset such as a spreadsheet, image, or video file.</span></span></li>
            <li><span data-ttu-id="bf9b4-359">O conteúdo deve ser durável (persistente) e externos a qualquer camada de aplicativo ou máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-359">Content must be durable (persistent), and external to any application tier or virtual machine.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-360"><strong>Tipo de dados</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-360"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-361">O tamanho dos dados é grande.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-361">Data size is large.</span></span></li>
            <li><span data-ttu-id="bf9b4-362">Dados de blob.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-362">Blob data.</span></span></li>
            <li><span data-ttu-id="bf9b4-363">O valor é opaco.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-363">Value is opaque.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-364"><strong>Exemplos</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-364"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-365">Imagens, vídeos, documentos do office e PDFs</span><span class="sxs-lookup"><span data-stu-id="bf9b4-365">Images, videos, office documents, PDFs</span></span></li>
            <li><span data-ttu-id="bf9b4-366">CSS, Scripts, CSV</span><span class="sxs-lookup"><span data-stu-id="bf9b4-366">CSS, Scripts, CSV</span></span></li>
            <li><span data-ttu-id="bf9b4-367">HTML estático, JSON</span><span class="sxs-lookup"><span data-stu-id="bf9b4-367">Static HTML, JSON</span></span></li>
            <li><span data-ttu-id="bf9b4-368">Arquivos de log e auditoria</span><span class="sxs-lookup"><span data-stu-id="bf9b4-368">Log and audit files</span></span></li>
            <li><span data-ttu-id="bf9b4-369">Backups de banco de dados</span><span class="sxs-lookup"><span data-stu-id="bf9b4-369">Database backups</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a><span data-ttu-id="bf9b4-370">Arquivos compartilhados</span><span class="sxs-lookup"><span data-stu-id="bf9b4-370">Shared files</span></span>

<table>
<tr><td><span data-ttu-id="bf9b4-371"><strong>Carga de trabalho</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-371"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-372">Migração de aplicativos existentes que interagem com o sistema de arquivos.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-372">Migration from existing apps that interact with the file system.</span></span></li>
            <li><span data-ttu-id="bf9b4-373">Requer a interface SMB.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-373">Requires SMB interface.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-374"><strong>Tipo de dados</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-374"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-375">Arquivos em um conjunto hierárquico de pastas.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-375">Files in a hierarchical set of folders.</span></span></li>
            <li><span data-ttu-id="bf9b4-376">Acessível com bibliotecas de E/S padrão.</span><span class="sxs-lookup"><span data-stu-id="bf9b4-376">Accessible with standard I/O libraries.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="bf9b4-377"><strong>Exemplos</strong></span><span class="sxs-lookup"><span data-stu-id="bf9b4-377"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="bf9b4-378">Arquivos herdados</span><span class="sxs-lookup"><span data-stu-id="bf9b4-378">Legacy files</span></span></li>
            <li><span data-ttu-id="bf9b4-379">O conteúdo compartilhado pode ser acessado por certo número de VMs ou instâncias de aplicativo</span><span class="sxs-lookup"><span data-stu-id="bf9b4-379">Shared content accessible among a number of VMs or app instances</span></span></li>
        </ul>
    </td>
</tr>
</table>
