---
title: Critérios para escolher um armazenamento de dados
titleSuffix: Azure Application Architecture Guide
description: Visão geral das opções de computação do Azure.
author: MikeWasson
ms.date: 06/01/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 3fd3badd66edbe561bea88576bb80d9fc3e0bb79
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59068915"
---
# <a name="criteria-for-choosing-a-data-store"></a>Critérios para escolher um armazenamento de dados

O Azure dá suporte a muitos tipos de soluções de armazenamento de dados, cada um fornecendo recursos e funcionalidades diferentes. Este artigo descreve os critérios de comparação que você deve usar ao avaliar um armazenamento de dados. A meta é ajudar você a determinar quais tipos de armazenamento de dados podem atender aos requisitos da sua solução.

## <a name="general-considerations"></a>Considerações gerais

Para iniciar a comparação, reúna o máximo possível das informações a seguir sobre suas necessidades de dados. Essas informações ajudarão a determinar quais tipos de armazenamento de dados atenda às suas necessidades.

### <a name="functional-requirements"></a>Requisitos funcionais

- **Formato de dados**. Quais tipos de dados você pretende armazenar? Tipos comuns incluem dados transacionais, objetos JSON, telemetria, índices de pesquisa ou arquivos simples.

- **Tamanho dos dados**. Qual será o tamanho das entidades que você precisa armazenar? Essas entidades precisam ser mantidas como um único documento ou podem ser divididas em vários documentos, tabelas, coleções e assim por diante?

- **Escala e a estrutura**. Qual é o volume geral da capacidade de armazenamento necessária? Você pretende particionar os dados?

- **Relações entre os dados**. Seus dados precisarão dar suporte a relações um para muitos ou muitos para muitos? São relações em si são uma parte importante dos dados? Você precisará ingressar ou combinar os dados de dentro do mesmo conjunto de dados ou de conjuntos de dados externos?

- **Modelo de consistência**. O quão é importante que as atualizações realizadas em um nó apareçam em outros nós antes de permitir alterações adicionais? É possível aceitar consistência eventual? Você precisa de garantias de ACID para transações?

- **Flexibilidade do esquema**. Quais tipos de esquemas serão aplicadas aos seus dados? Você usará um esquema fixo, uma abordagem de esquema na gravação ou de esquema na leitura?

- **Simultaneidade**. Qual tipo de mecanismo de simultaneidade você deseja usar para atualizar e sincronizar os dados? O aplicativo executará muitas atualizações que poderiam entrar em conflito. Nesse caso, é possível exigir o bloqueio de registros e o controle de simultaneidade pessimista. Como alternativa, você pode dar suporte a controles de simultaneidade otimista? Nesse caso, é simples o suficiente controlar a simultaneidade baseada em carimbo de data/hora ou você precisa de funcionalidade adicional de controle de simultaneidade de várias versões?

- **Movimentação de dados**. Sua solução precisará executar tarefas ETL para mover os dados para outros armazenamentos ou data warehouses?

- **Ciclo de vida dos dados**. A gravação dos ocorre uma vez, com muitas leituras? Eles podem ser movidos para o armazenamento frequente ou esporádico?

- **Outros recursos com suporte**. Você precisa de algum outro recurso específico, como validação de esquema, agregação, indexação, pesquisa de texto completo, MapReduce ou outras funcionalidades de consulta?

### <a name="non-functional-requirements"></a>Requisitos não funcionais

- **Desempenho e escalabilidade**. Quais são seus requisitos de desempenho de dados? Você tem requisitos específicos para as taxas de ingestão de dados e taxas de processamento de dados? Quais são os tempos de resposta aceitáveis para consulta e agregação dos dados depois de ingeridos? Até que tamanho você precisará aumentar o armazenamento de dados? Sua carga de trabalho tem leituras ou gravações mais frequentes?

- **Confiabilidade**. A qual SLA geral você precisa dar suporte? Qual nível de tolerância a falhas você precisa fornecer para os consumidores dos dados? De quais tipos de funcionalidades de backup e restauração você precisa?

- **Replicação**. Os dados precisarão ser distribuído entre várias réplicas ou regiões? De quais tipos de funcionalidades de replicação de dados você precisa?

- **Limites**. Os limites de um armazenamento de dados específico dará suporte aos seus requisitos de escala, número de conexões e taxa de transferência?

### <a name="management-and-cost"></a>Gerenciamento e o custo

- **Serviço gerenciado**. Quando possível, use um serviço de dados gerenciados, a menos que você precise de funcionalidades específicas encontradas somente em um armazenamento de dados hospedado por IaaS.

- **Disponibilidade de região**. Para serviços gerenciais, o serviço está disponível em todas as regiões do Azure? Sua solução precisa ser hospedada em determinadas regiões do Azure?

- **Portabilidade**. Seus dados precisarão ser migrados para o local, datacenters externos ou outro ambientes de hospedagem em nuvem?

- **Licenciamento**. Você tem preferência pelo tipo de licença proprietária em comparação a OSS? Há outras restrições externas sobre o tipo de licença que você pode usar?

- **Custo geral**. Qual é o custo geral de usar o serviço na sua solução? Quantas instâncias você precisará executar para dar suporte aos seus requisitos de tempo de atividade e taxa de transferência? Considere os custos das operações neste cálculo. Um motivo para preferir serviços gerenciados é a redução dos custos operacionais.

- **Economia de custos**. Você pode particionar seus dados para armazená-los de forma mais econômica? Por exemplo, você pode mover grandes objetos para fora de um banco de dados relacional caro e para um repositório de objetos?

### <a name="security"></a>Segurança

- **Segurança**. Qual tipo de criptografia é necessária? Você precisa de criptografia em repouso? Qual mecanismo de autenticação você deseja usar para se conectar aos seus dados?

- **Auditoria**. Qual tipo de log de auditoria você precisa gerar?

- **Requisitos de rede**. Você precisa restringir ou gerenciar o acesso aos dados de outros recursos de rede? Os dados precisam ser acessíveis somente de dentro do ambiente do Azure? Os dados precisam ser acessíveis de endereços IP ou sub-redes específicas? Eles precisam ser acessíveis de aplicativos ou serviços hospedados localmente ou de outros datacenters externos?

### <a name="devops"></a>DevOps

- **Conjunto de qualificações**. Há determinadas linguagens de programação, sistemas operacionais ou outras tecnologias que sua equipe tem experiência específica em usar? Há outras com as quais seria difícil sua equipe trabalhar?

- **Clientes** Há suporte de cliente adequado para suas linguagens de desenvolvimento?

As seções a seguir comparam vários modelos de armazenamento de dados com relação ao perfil de carga de trabalho, tipos de dados e casos de uso de exemplo.

## <a name="relational-database-management-systems-rdbms"></a>RDBMS

<!-- markdownlint-disable MD033 -->

<table>
<tr><td><strong>Carga de trabalho</strong></td>
    <td>
        <ul>
            <li>Tanto a criação de novos registros quanto atualizações em dados existentes ocorrem regularmente.</li>
            <li>Várias operações precisam ser concluídas em uma única transação.</li>
            <li>Requer funções de agregação para executar tabulação cruzada.</li>
            <li>Forte integração com ferramentas de relatório é necessária.</li>
            <li>As relações são impostas usando restrições de banco de dados.</li>
            <li>Os índices são usados para otimizar o desempenho da consulta.</li>
            <li>Permite acesso a subconjuntos específicos de dados.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de dados</strong></td>
    <td>
        <ul>
            <li>Os dados são altamente normalizados.</li>
            <li>Esquemas de banco de dados são necessários e impostos.</li>
            <li>Relações muitos para muitos entre as entidades de dados no banco de dados.</li>
            <li>As restrições são definidas no esquema e impostas nos dados no banco de dados.</li>
            <li>Os dados exigem alta integridade. Os índices e as relações precisam ser mantidos com precisão.</li>
            <li>Os dados requerem consistência forte. As transações operam de forma a garantir que todos os dados sejam 100% consistentes para todos os usuários e processos.</li>
            <li>O tamanho das entradas de dados individuais deve ser de pequeno a médio porte.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemplos</strong></td>
    <td>
        <ul>
            <li>Linha de negócios (gerenciamento de capital humano, gerenciamento de relacionamento com o cliente, planejamento de recursos da empresa)</li>
            <li>Gerenciamento de estoque</li>
            <li>Banco de dados de relatórios</li>
            <li>Contabilidade</li>
            <li>Gerenciamento de ativos</li>
            <li>Gerenciamento de fundos</li>
            <li>Gerenciamento de pedidos</li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a>Bancos de dados de documentos

<table>
<tr><td><strong>Carga de trabalho</strong></td>
    <td>
        <ul>
            <li>Uso geral.</li>
            <li>As operações de inserção e atualização são comuns. Tanto a criação de novos registros quanto atualizações em dados existentes ocorrem regularmente.</li>
            <li>Não há incompatibilidade de impedância relacional de objeto. Os documentos correspondem melhor às estruturas de objeto usadas no código do aplicativo.</li>
            <li>A simultaneidade otimista é usada com mais frequência.</li>
            <li>Os dados precisam ser modificados e processados pelo aplicativo de consumo.</li>
            <li>Os dados requerem um índice de vários campos.</li>
            <li>Os documentos individuais são recuperados e gravados como um único bloco.</li>
    </td>
</tr>
<tr><td><strong>Tipo de dados</strong></td>
    <td>
        <ul>
            <li>Os dados podem ser gerenciados de forma desordenada.</li>
            <li>O tamanho dos dados do documento individual é relativamente pequeno.</li>
            <li>Cada tipo de documento pode usar seu próprio esquema.</li>
            <li>Os documentos podem incluir campos opcionais.</li>
            <li>Os dados de documento são semi-estruturados, o que significa que os tipos de dados de cada campo não são estritamente definidos.</li>
            <li>Há suporte para a agregação de dados.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemplos</strong></td>
    <td>
        <ul>
            <li>Catálogo de produtos</li>
            <li>Contas de usuário</li>
            <li>Lista de materiais</li>
            <li>Personalização</li>
            <li>Gerenciamento de conteúdo</li>
            <li>Dados de operações</li>
            <li>Gerenciamento de estoque</li>
            <li>Dados do histórico de transação</li>
            <li>Exibição materializada de outros repositórios NoSQL. Substitui a indexação de arquivo/BLOB.</li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a>Armazenamentos de valor/chave

<table>
<tr><td><strong>Carga de trabalho</strong></td>
    <td>
        <ul>
            <li>Os dados são identificados e acessados usando uma única chave de ID, como um dicionário.</li>
            <li>Amplamente escalonável.</li>
            <li>Associações, bloqueios ou uniões não são necessários.</li>
            <li>Mecanismos de agregação não são usados.</li>
            <li>Os índices secundários geralmente não são usados.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de dados</strong></td>
    <td>
        <ul>
            <li>O tamanho dos dados tende a ser grande.</li>
            <li>Cada chave é associada a um único valor, que é um BLOB de dados não gerenciado.</li>
            <li>Não há imposição do esquema.</li>
            <li>Não há relações entre entidades.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemplos</strong></td>
    <td>
        <ul>
            <li>Armazenamento de dados em cache</li>
            <li>Gerenciamento da sessão</li>
            <li>Preferência do usuário e gerenciamento de perfil</li>
            <li>Recomendação de produtos e veiculação de anúncios</li>
            <li>Dicionários</li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a>Bancos de dados de grafo

<table>
<tr><td><strong>Carga de trabalho</strong></td>
    <td>
        <ul>
            <li>As relações entre os itens de dados são muito complexas, o que envolve vários saltos entre os itens de dados relacionados.</li>
            <li>A relação entre os itens de dados são dinâmicas e mudam com o passar do tempo.</li>
            <li>As relações entre objetos são de primeira classe, sem precisar de chaves estrangeiras e ingressos para serem percorridas.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de dados</strong></td>
    <td>
        <ul>
            <li>Os dados são compostos por nós e relações.</li>
            <li>Nós são semelhantes às linhas da tabela ou documentos JSON.</li>
            <li>As relações são tão importantes quanto os nós e são expostas diretamente na linguagem de consulta.</li>
            <li>Objetos de composição, como uma pessoa com vários números de telefone, tendem a ser divididos em nós menores combinado com relações navegáveis </li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemplos</strong></td>
    <td>
        <ul>
            <li>Quadros da organização</li>
            <li>Gráficos sociais</li>
            <li>Detecção de fraude</li>
            <li>Análise</li>
            <li>Mecanismos de recomendação</li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a>Bancos de dados de família de coluna

<table>
<tr><td><strong>Carga de trabalho</strong></td>
    <td>
        <ul>
            <li>A maioria dos bancos de dados de famílias de colunas executam operações de gravação muito rapidamente.</li>
            <li>As operações de atualização e exclusão são raras.</li>
            <li>Projetado para fornecer acesso com alta taxa de transferência e baixa latência.</li>
            <li>Dá suporte ao acesso de consulta fácil para determinado conjunto de campos dentro de um registro muito maior.</li>
            <li>Amplamente escalonável.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de dados</strong></td>
    <td>
        <ul>
            <li>Os dados são armazenados em tabelas compostas por uma coluna de chave e uma ou mais famílias de colunas.</li>
            <li>As colunas específicas podem variar conforme as linhas individuais.</li>
            <li>As células individuais são acessadas por meio de comandos get e put</li>
            <li>Várias linhas são retornadas usando um comando de verificação.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemplos</strong></td>
    <td>
        <ul>
            <li>Recomendações</li>
            <li>Personalização</li>
            <li>Dados de sensor</li>
            <li>Telemetria</li>
            <li>Mensagens</li>
            <li>Análise de mídia social</li>
            <li>Análise da Web</li>
            <li>Monitorando de atividades</li>
            <li>Previsão do tempo e outros dados de série temporal</li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a>Bancos de dados de mecanismo de pesquisa

<table>
<tr><td><strong>Carga de trabalho</strong></td>
    <td>
        <ul>
            <li>Indexação de dados de várias fontes e serviços.</li>
            <li>Consultas são ad hoc e podem ser complexas.</li>
            <li>Requer agregação.</li>
            <li>Requer pesquisa de texto completo.</li>
            <li>Requer consulta ad hoc de autoatendimento.</li>
            <li>Requer análise de dados com índice em todos os campos.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de dados</strong></td>
    <td>
        <ul>
            <li>Dados semi-estruturados ou não estruturados</li>
            <li>Texto</li>
            <li>Texto com referência a dados estruturados</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemplos</strong></td>
    <td>
        <ul>
            <li>Catálogos de produtos</li>
            <li>Pesquisa de site</li>
            <li>Registro em log</li>
            <li>Análise</li>
            <li>Sites de compra</li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a>Data warehouse

<table>
<tr><td><strong>Carga de trabalho</strong></td>
    <td>
        <ul>
            <li>Análise de dados</li>
            <li>Enterprise BI</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de dados</strong></td>
    <td>
        <ul>
            <li>Dados históricos de várias fontes.</li>
            <li>Geralmente desnormalizado em um esquema de &quot;estrela&quot; ou &quot;floco de neve&quot;, consistindo em tabelas de dimensões e fatos.</li>
            <li>Geralmente é carregado com novos dados de forma programada.</li>
            <li>As tabelas de dimensões geralmente incluem várias versões históricas de uma entidade, conhecida como uma <em>dimensão de alteração lenta</em>.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemplos</strong></td>
    <td>Um data warehouse corporativo que fornece dados para os modelos, relatórios e painéis analíticos.
    </td>
</tr>
</table>

## <a name="time-series-databases"></a>Bancos de dados de séries temporais

<table>
<tr><td><strong>Carga de trabalho</strong></td>
    <td>
        <ul>
            <li>Um proporção predominante das operações (95% a 99%) é composta por gravações.</li>
            <li>Os registros são geralmente anexados sequencialmente em ordem cronológica.</li>
            <li>As atualizações são raras.</li>
            <li>As exclusões ocorrem em massa e são realizadas em blocos ou registros contíguos.</li>
            <li>As solicitações de leitura podem ser maiores que a memória disponível.</li>
            <li>É comum que várias leituras ocorram simultaneamente.</li>
            <li>Os dados são lidos em sequência em ordem crescente ou decrescente cronológica.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de dados</strong></td>
    <td>
        <ul>
            <li>Um carimbo de data/hora que é usado a chave primária e o mecanismo de classificação.</li>
            <li>Medidas da entrada ou descrições do que a entrada representa.</li>
            <li>Marcas que definem informações adicionais sobre o tipo, origem e outras informações sobre a entrada.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemplos</strong></td>
    <td>
        <ul>
            <li>Monitoramento e telemetria de evento.</li>
            <li>Sensor ou outros dados de IoT.</li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a>Armazenamento de objetos

<table>
<tr><td><strong>Carga de trabalho</strong></td>
    <td>
        <ul>
            <li>Identificado pela chave.</li>
            <li>Os objetos podem estar acessíveis de forma pública ou privada.</li>
            <li>O conteúdo normalmente é um ativo, como uma planilha, uma imagem ou um arquivo de vídeo.</li>
            <li>O conteúdo deve ser durável (persistente) e externos a qualquer camada de aplicativo ou máquina virtual.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de dados</strong></td>
    <td>
        <ul>
            <li>O tamanho dos dados é grande.</li>
            <li>Dados de blob.</li>
            <li>O valor é opaco.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemplos</strong></td>
    <td>
        <ul>
            <li>Imagens, vídeos, documentos do office e PDFs</li>
            <li>CSS, Scripts, CSV</li>
            <li>HTML estático, JSON</li>
            <li>Arquivos de log e auditoria</li>
            <li>Backups de banco de dados</li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a>Arquivos compartilhados

<table>
<tr><td><strong>Carga de trabalho</strong></td>
    <td>
        <ul>
            <li>Migração de aplicativos existentes que interagem com o sistema de arquivos.</li>
            <li>Requer a interface SMB.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de dados</strong></td>
    <td>
        <ul>
            <li>Arquivos em um conjunto hierárquico de pastas.</li>
            <li>Acessível com bibliotecas de E/S padrão.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemplos</strong></td>
    <td>
        <ul>
            <li>Arquivos herdados</li>
            <li>O conteúdo compartilhado pode ser acessado por certo número de VMs ou instâncias de aplicativo</li>
        </ul>
    </td>
</tr>
</table>

<!-- markdownlint-enable MD033 -->
