---
title: 'Adoção da Nuvem Empresarial: Conceitos básicos operacionais'
description: Orientação sobre os conceitos básicos operacionais
author: petertaylor9999
ms.date: 09/20/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 26867e3ecce738f18c5a03ff41754281229851f4
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481118"
---
# <a name="establishing-an-operational-fitness-review"></a>Estabelecer uma análise de adequação operacional

Conforme sua empresa começa a operar cargas de trabalho no Azure, a próxima etapa é estabelecer um processo de **análise de adequação operacional** para enumerar, implementar e analisar de forma iterativa os requisitos **não funcionais** para essas cargas de trabalho. Os requisitos _não funcionais_ são relacionados ao comportamento operacional esperado do serviço. Há cinco categorias principais de requisitos não funcionais, conhecidas como os [pilares da qualidade do software](../../guide/pillars.md): escalabilidade, disponibilidade, resiliência (incluindo a continuidade dos negócios e recuperação de desastres), gerenciamento e segurança. A finalidade de um processo de análise de adequação operacional é garantir que suas cargas de trabalho críticas atendam às expectativas do seu negócio em relação aos pilares de qualidade.

Por esse motivo, sua empresa deve realizar um processo de análise de adequação operacional para compreender totalmente os problemas resultantes da execução da carga de trabalho em um ambiente de produção, determinar como corrigir os problemas e resolvê-los. Este artigo descreve um processo de análise de adequação operacional de alto nível que sua empresa pode usar para atingir esse objetivo.

## <a name="operational-fitness-at-microsoft"></a>Adequação operacional na Microsoft

Desde o início, o desenvolvimento da plataforma do Azure tem sido um projeto de integração e de desenvolvimento contínuo realizado por muitas equipes em toda a Microsoft. Seria muito difícil garantir a qualidade e a consistência para um projeto do tamanho e da complexidade do Azure sem um processo robusto para enumerar e implementar os requisitos não funcionais básicos regularmente.

Esses processos, seguidos pela Microsoft, formam a base para o processo descrito neste documento.

## <a name="understanding-the-problem"></a>Compreendendo o problema

Como você aprendeu no [Guia de Introdução](../../cloud-adoption/getting-started/overview.md), a primeira etapa em um processo de transformação digital da empresa é identificar os problemas empresariais que precisam ser resolvidos com a adoção do Azure. A próxima etapa é determinar uma solução de alto nível para o problema, como migrar uma carga de trabalho para a nuvem, ou adaptar um serviço local existente para incluir a funcionalidade da nuvem. Por fim, a solução é projetada e implementada.

Durante esse processo, o foco geralmente está nos _recursos_ do serviço. Ou seja, há um conjunto de requisitos _funcionais_ desejados para a realização do serviço. Por exemplo, um serviço de entrega de mercadorias exige recursos para determinar os locais de origem e de destino da mercadoria que rastreiam a mercadoria durante a entrega, as notificações do cliente, entre outros.

Por outro lado, os requisitos _não funcionais_ estão relacionados às propriedades, como a [disponibilidade](../../checklist/availability.md), [resiliência](../../resiliency/index.md) e [escalabilidade](../../checklist/scalability.md) do serviço. Essas propriedades diferem dos requisitos funcionais porque elas não afetam diretamente a função final de nenhum recurso específico no serviço. No entanto, esses requisitos não funcionais estão relacionados ao _desempenho_ e à _continuidade_ do serviço.

Alguns requisitos não funcionais podem ser especificados nos termos de um contrato de nível de serviço (SLA). Por exemplo, em relação à continuidade do serviço, um requisito de disponibilidade para o serviço pode ser expresso em forma de uma porcentagem, como **disponível 99,99% do tempo**. Outros requisitos não funcionais podem ser mais difíceis de definir e podem mudar conforme as necessidades de produção evoluem. Por exemplo, um serviço voltado para o consumidor pode começar a enfrentar requisitos imprevistos de taxa de transferência após uma onda de popularidade.

![OBSERVAÇÃO] Definir os requisitos para garantir a resiliência, incluindo explicações de RPO, RTO, SLA e conceitos relacionados, são explorados mais detalhadamente em [Projetar aplicativos resilientes para o Azure](../../resiliency/index.md#define-your-availability-requirements).

## <a name="operational-fitness-review-process"></a>Processo de análise de adequação operacional

A chave para manter o desempenho e a continuidade dos serviços de uma empresa é implementar um processo de _análise de adequação operacional_.

![Uma visão geral do processo de análise de adequação operacional](_images/ofr-flow.png)

Em um nível elevado, o processo conta com duas fases. Na fase de pré-requisitos, os requisitos são estabelecidos e mapeados para serviços de suporte. Isso ocorre com menos frequência; talvez uma vez ao ano ou quando novas operações são introduzidas. A saída da fase de pré-requisitos é usada na fase de fluxo. A fase de fluxo ocorre com mais frequência; é recomendável uma frequência mensal.

### <a name="prerequisites-phase"></a>Fase de pré-requisitos

As etapas nesta fase destinam-se a capturar os requisitos necessários para conduzir uma análise regular dos serviços importantes.

- **Identificar operações de negócios críticas**. Identificar as operações de negócio **críticas para a missão** da empresa. Operações de negócios são independentes de qualquer funcionalidade de serviço de suporte. Em outras palavras, as operações de negócios representam atividades reais que a empresa precisa realizar e são suportadas por um conjunto de serviços de TI. O termo _crítico para a missão_, ou como alternativa _comercialmente crítico_, refletirá um grave impacto nos negócios se a operação for impedida. Por exemplo, uma loja online pode oferecer operações de negócios, como “permitir que um cliente adicione um item a um carrinho de compras” ou “processar um pagamento com cartão de crédito”. Se qualquer uma dessas operações falhasse, o cliente não poderia concluir a transação e a empresa não conseguiria realizar as vendas.

- **Mapear operações para serviços**. Mapeie essas operações de negócios para os serviços que dão suporte a elas. No exemplo do carrinho de compras acima, vários serviços podem estar envolvidos: um serviço de gerenciamento de estoque de inventário, um serviço do carrinho de compras, entre outros. No exemplo do pagamento com cartão de crédito acima, um serviço de pagamento local pode interagir com um serviço de processamento de pagamento de terceiros.

- **Analisar dependências de serviços**. A maioria das operações de negócios exige um trabalho coordenado entre vários serviços de suporte. É importante entender as dependências entre os serviços e o fluxo de transações críticas para a missão por meio desses serviços. Você também deve considerar as dependências entre os serviços locais e os serviços do Azure. No exemplo do carrinho de compras, o serviço de gerenciamento de estoque de inventário pode ser hospedado no local e inserir dados recebidos pelos funcionários de um armazém físico, mas pode armazenar dados em um serviço do Azure como o [armazenamento do Azure](/azure/storage/common/storage-introduction) ou um banco de dados como o [Azure Cosmos DB](/azure/cosmos-db/introduction).

O resultado dessas atividades é um conjunto de **métricas de scorecard** para operações de serviço. As métricas são categorizadas em termos de critérios não funcionais como disponibilidade, escalabilidade e recuperação de desastre. As métricas de scorecard expressam os critérios que espera-se que o serviço atenda operacionalmente. Essas métricas podem ser expressas em qualquer nível de granularidade apropriado para a operação de serviço.

O scorecard deve ser expresso em termos simples para facilitar uma discussão relevante entre os proprietários dos negócios e a engenharia. Por exemplo, uma métrica de scorecard de escalabilidade pode ser expressa como _verde_ para a execução de critérios desejados, _amarelo_ para falha em atender os critérios desejados, mas ativamente implementando uma correção planejada, e _vermelho_ para falha ao atender os critérios desejados sem nenhum plano ou ação.

É importante enfatizar que essas métricas devem refletir diretamente a necessidades dos negócios.

### <a name="service-review-phase"></a>Fase de análise do serviço

A fase de análise do serviço é o núcleo do processo de análise de adequação operacional.

- **Medir métricas de serviço**. Usando as métricas de scorecard, os serviços devem ser monitorados para garantir o cumprimento das expectativas dos negócios. Isso significa que o serviço de monitoramento é essencial. Se não conseguir monitorar um conjunto de serviços em relação aos requisitos não funcionais, as métricas de scorecard correspondentes devem ter a indicação na cor vermelha. Nesse caso, a primeira etapa de correção é implementar o monitoramento de serviço apropriado.
Por exemplo, se a empresa espera que um serviço opere com 99,99% de disponibilidade, mas não há qualquer telemetria de produção em vigor para medir a disponibilidade, você deve presumir que não está cumprindo o requisito.

- **Planejar uma correção**. Para cada operação de serviço com métricas que ficam abaixo do limite aceitável, determine o custo de corrigir o serviço para que a operação tenha uma métrica aceitável. Se o custo de corrigir o serviço for maior que a geração de receita esperada do serviço, então considere os custos não tangíveis, como a experiência do cliente. Por exemplo, se os clientes têm dificuldade para concluir um pedido com êxito usando o serviço, eles podem escolher um concorrente.

- **Implementar a correção**. Depois que os proprietários de negócios e a engenharia concordarem em um plano, ele deverá ser implementado. O status da implementação deverá ser relatado sempre que as métricas de scorecard forem analisadas.

Esse processo for iterativo e o ideal é que sua empresa tenha uma equipe dedicada a ele. Essa equipe deve se reunir regularmente para analisar os projetos existentes de correção, iniciar a análise de conceitos básicos de novas cargas de trabalho e rastrear o scorecard geral da empresa. A equipe deve ter autoridade para garantir a responsabilidade das equipes de correção que estão atrasadas ou que não estão atendendo às métricas.

## <a name="structure-of-the-operational-fitness-review-team"></a>Estrutura da equipe de análise de adequação operacional

A equipe de análise de adequação operacional é composta pelas seguintes funções:

1. **Proprietário da empresa**. Essa função fornece conhecimento sobre a empresa para identificar e priorizar cada operação de negócios “crítica para a missão”. Essa função também compara o custo de mitigação ao impacto nos negócios e está por trás da decisão final em relação à correção.

2. **Consultor dos negócios**. Essa função é responsável por dividir as operações de negócios em partes menores e mapear essas partes para serviços e infraestrutura locais e na nuvem. A função requer um conhecimento profundo da tecnologia associada a cada operação de negócios.

3. **Responsável pela engenharia**. Essa função é responsável por implementar os serviços associados à operação de negócios. Essas pessoas podem participar de design, implementação e implantação de qualquer solução para resolver problemas não funcionais descobertos pela equipe de análise de adequação operacional.

4. **Proprietário do serviço**. Essa função é responsável por operar serviços e aplicativos de negócios. Esses indivíduos coletam dados de registro em log e de uso para esses aplicativos e serviços. Esses dados são usados para identificar problemas e verificar as correções depois de implantadas.

## <a name="operational-fitness-review-meeting"></a>Reunião de análise de adequação operacional

Recomendamos que sua equipe de análise de adequação operacional se reúna com regularidade. Por exemplo, a equipe pode reunir-se mensalmente e informar o status e as métricas à liderança sênior trimestralmente.

Os detalhes do processo e a reunião devem ser adaptados para atender às suas necessidades específicas. Como ponto de partida, recomendamos as seguintes tarefas:

1. O proprietário da empresa e o consultor de negócios enumeram e determinam os requisitos não funcionais para cada operação de negócios, com as informações fornecidas pelo responsável da engenharia e pelo proprietário de serviços. Para operações de negócios que foram identificadas anteriormente, a prioridade é analisada e verificada. Para novas operações de negócios, é atribuída uma prioridade na lista existente.

2. O responsável pela engenharia e o proprietário de serviços mapeiam o **estado atual** das operações de negócios para os serviços locais e na nuvem. O mapeamento é formado por uma lista dos componentes em cada serviço, organizada como uma árvore de dependência. Depois que a lista e a árvore de dependência são geradas, são determinados os **caminhos críticos** na árvore.

3. O responsável pela engenharia e o proprietário de serviços analisam o estado atual do log operacional e o monitoramento dos serviços listados na etapa anterior. Um registro em log e um monitoramento robustos são essenciais para identificar os componentes de serviço que contribuem para o não cumprimento de requisitos não funcionais. Se não houver um registro em log e monitoramento adequados em vigor, um plano deve ser criado e implementado para colocá-los em vigor.

4. As métricas de scorecard são criadas para novas operações de negócios. O scorecard é formado pela lista de componentes constituintes para cada serviço identificado na etapa 2, alinhados com os requisitos não funcionais e uma métrica que representa o quão bem o componente atende ao requisito.

5. Para os componentes constituintes que não atendem aos requisitos não funcionais, uma solução de alto nível é projetada e atribuída a um responsável pela engenharia. Neste ponto, o proprietário de negócios e o consultor de negócios devem estabelecer um orçamento para o trabalho de correção, com base na previsão de receita da operação de negócios.

6. Por fim, é conduzida uma análise do trabalho de correção em andamento. Cada métrica de scorecard para trabalhos em andamento é analisada em relação às métricas esperadas. Para componentes constituintes que estão atendendo a métricas, o proprietário de serviços apresenta os dados de log e monitoramento para confirmar que a métrica é atendida. Para os componentes constituintes que não estão atendendo às métricas, cada responsável pela engenharia explica os problemas que estão impedindo que as métricas sejam atingidas e quaisquer novos designs de correção.

## <a name="recommended-resources"></a>Recursos recomendados

- [Pilares de qualidade do software](../../guide/pillars.md).
Esta seção do guia de Arquitetura de aplicativo do Azure descreve os cinco pilares de qualidade do software: Escalabilidade, disponibilidade, resiliência, gerenciamento e segurança.
- [Dez princípios de design para aplicativos do Azure](../../guide/design-principles/index.md).
Esta seção do guia de Arquitetura de Aplicativo do Azure aborda um conjunto de princípios de design para tornar seu aplicativo mais escalonável, resiliente e gerenciável.
- [Projetar aplicativos resilientes para o Azure](../../resiliency/index.md).
Este guia começa com uma definição do termo resiliência e conceitos relacionados. Em seguida, descreve um processo para a obtenção de resiliência, usando uma abordagem estruturada sobre o tempo de vida de um aplicativo, desde o design e a implementação até a implantação e as operações.
- [Padrões de design na nuvem](../../patterns/index.md).
Esses padrões de design são úteis para equipes de engenharia ao criar aplicativos sobre os pilares de qualidade do software.