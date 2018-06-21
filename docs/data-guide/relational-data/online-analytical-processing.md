---
title: OLAP (processamento analítico online)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 92b71934f2081e95c3c9b0d4dc9edeb3885b12e8
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30846800"
---
# <a name="online-analytical-processing-olap"></a>OLAP (processamento analítico online)

O OLAP (processamento analítico online) é uma tecnologia que organiza bancos de dados comerciais grandes e dá suporte à análise complexa. Ele pode ser usado para executar consultas analíticas complexas sem prejudicar sistemas transacionais.

Os bancos de dados que uma empresa usa para armazenar todas as suas transações e registros são chamados bancos de dados [OLTP (processamento de transações online)](online-transaction-processing.md). Esses bancos de dados geralmente têm registros que são inseridos individualmente. Muitas vezes, eles contêm uma grande quantidade de informações que são importantes para a organização. No entanto, os bancos de dados que são usados para OLTP não foram projetados para análise. Portanto, a recuperação de respostas com base nesses bancos de dados é cara em termos de tempo e esforço. Os sistemas OLAP foram projetados para ajudar a extrair essas informações de business intelligence dos dados de uma maneira com alto desempenho. Isso ocorre porque os bancos de dados OLAP são otimizados para cargas de trabalho com leitura intensa e com pouca gravação.

![OLAP no Azure](../images/olap-data-pipeline.png) 

## <a name="semantic-modeling"></a>Modelagem semântica

Um modelo de dados semântico é um modelo conceitual que descreve o significado dos elementos de dados que ele contém. As empresas costumam ter seus próprios termos para coisas, às vezes, com sinônimos ou até mesmo significados diferentes para o mesmo termo. Por exemplo, um banco de dados de inventário pode acompanhar uma parte do equipamento com uma ID de ativo e um número de série, mas um banco de dados de vendas pode se referir ao número de série como a ID de ativo. Não há nenhuma maneira simples para relacionar esses valores sem um modelo que descreve a relação. 

A modelagem semântica fornece um nível de abstração no esquema de banco de dados, de modo que os usuários não precisem conhecer as estruturas de dados subjacentes. Isso facilita para os usuários finais consultar dados sem executar agregações e junções no esquema subjacente. Além disso, normalmente, as colunas são renomeadas com nomes mais amigáveis, de modo que o contexto e o significado dos dados sejam mais óbvios.

A modelagem semântica é predominantemente usada para cenários de leitura intensa, como análise e business intelligence (OLAP), ao contrário do processamento de dados transacionais de gravação mais intensa (OLTP). Isso é principalmente devido à natureza de uma camada semântica típica:

- Os comportamentos de agregação são definidos para que as ferramentas de relatórios os exibam corretamente.
- A lógica de negócios e os cálculos são definidos.
- Os cálculos orientados por tempo são incluídos.
- Os dados normalmente são integrados de várias fontes. 

Tradicionalmente, a camada semântica é colocada em um data warehouse por esses motivos.

![Diagrama de exemplo de uma camada semântica entre um data warehouse e uma ferramenta de relatórios](../images/semantic-modeling.png)

Há dois tipos principais de modelos semânticos:

* **Tabular**. Usa constructos de modelagem relacional (modelo, tabelas, colunas). Internamente, os metadados são herdados de constructos de modelagem OLAP (cubos, dimensões, medidas). O código e o script usam metadados OLAP.
* **Multidimensional**. Usa constructos de modelagem OLAP tradicionais (cubos, dimensões, medidas).

Serviço do Azure relevante:
- [Azure Analysis Services](https://azure.microsoft.com/services/analysis-services/)

## <a name="example-use-case"></a>Caso de uso de exemplo

Uma organização tem dados armazenados em um banco de dados grande. Ela deseja disponibilizar esses dados para usuários empresariais e clientes para criar seus próprios relatórios e fazer análises. Uma opção é apenas fornecer a esses usuários o acesso direto ao banco de dados. No entanto, há várias desvantagens ao fazer isso, incluindo o gerenciamento de segurança e controle de acesso. Além disso, o design do banco de dados, incluindo os nomes de tabelas e colunas, pode ser de difícil compreensão para um usuário. Os usuários precisam saber quais tabelas consultar, como as tabelas devem ser unidas e outra lógica de negócios que precisa ser aplicada para obter os resultados corretos. Os usuários também precisam conhecer uma linguagem de consulta como SQL, até mesmo para começar. Normalmente, isso leva ao relatório de vários usuários das mesmas métricas, mas com resultados diferentes.

Outra opção é encapsular todas as informações de que os usuários precisam em um modelo semântico. O modelo semântico pode ser consultado com mais facilidade por usuários com uma ferramenta de relatórios de sua escolha. Os dados fornecidos pelo modelo semântico são extraídos de um data warehouse, garantindo que todos os usuários vejam uma única versão da verdade. O modelo semântico também fornece nomes de tabela e coluna amigáveis, relações entre tabelas, descrições, cálculos e segurança em nível de linha.

## <a name="typical-traits-of-semantic-modeling"></a>Características comuns da modelagem semântica

A modelagem semântica e o processamento analítico tendem a ter as seguintes características:

| Requisito | DESCRIÇÃO |
| --- | --- |
| Esquema | Esquema na gravação, altamente imposto|
| Usa Transações | Não  |
| Estratégia de Bloqueio | Nenhum |
| Atualizável | Não (normalmente exige o recálculo de cubo) |
| Acrescentável | Não (normalmente exige o recálculo de cubo) |
| Carga de trabalho | Leituras intensas, somente leitura |
| Indexação | Indexação multidimensional |
| Tamanho do dado | De pequeno a médio |
| Modelo | Multidimensional |
| Forma dos dados:| Esquema de cubo ou estrela/floco de neve |
| Flexibilidade de consulta | Altamente flexível |
| Escala: | Grande (dezenas a centenas de GBs) |

## <a name="when-to-use-this-solution"></a>Quando usar esta solução

Considere o uso do OLAP nos seguintes cenários:

- Você precisa executar consultas analíticas e ad hoc complexas rapidamente, sem prejudicar os sistemas OLTP. 
- Você deseja fornecer aos usuários empresariais uma maneira simples de gerar relatórios com base em seus dados
- Você deseja fornecer várias agregações que permitirão aos usuários obter resultados consistentes e rápidos. 

O OLAP é especialmente útil para aplicação de cálculos de agregação em grandes quantidades de dados. Os sistemas OLAP são otimizados para cenários com leitura intensa, como análise e business intelligence. O OLAP permite aos usuários segmentar dados multidimensionais em fatias que podem ser exibidas em duas dimensões (como uma tabela dinâmica) ou filtrar os dados por valores específicos. Esse processo, às vezes, é chamado de "segmentar e analisar" os dados e pode ser feito independentemente se os dados são particionados em várias fontes de dados. Isso ajuda os usuários a localizar tendências, identificar padrões e explorar os dados sem precisar saber os detalhes da análise de dados tradicional.

Os modelos semânticos podem ajudar os usuários empresariais a abstrair complexidades de relações e facilitar a análise rápida dos dados.

## <a name="challenges"></a>Desafios

Para todos os benefícios oferecidos pelos sistemas OLAP, eles trazem alguns desafios:

- Enquanto os dados em sistemas OLTP são atualizados constantemente por meio de transações que fluem de várias fontes, os armazenamentos de dados OLAP geralmente são atualizados em intervalos muito mais lentos, dependendo das necessidades de negócios. Isso significa que os sistemas OLAP são mais adequados para decisões de negócios estratégicas, em vez de respostas imediatas a mudanças. Além disso, um nível de limpeza de dados e orquestração precisa ser planejado para manter os armazenamentos de dados OLAP atualizados.
- Ao contrário das tabelas relacionais e normalizadas tradicionais encontradas em sistemas OLTP, os modelos de dados OLAP tendem a ser multidimensionais. Isso dificulta ou impossibilita o mapeamento direto para modelos de relação-entidade ou orientados a objetos, em que cada atributo é mapeado para uma coluna. Em vez disso, os sistemas OLAP normalmente usam um esquema estrela ou floco de neve no lugar da normalização tradicional.

## <a name="olap-in-azure"></a>OLAP no Azure

No Azure, os dados mantidos em sistemas OLTP, como o Banco de Dados SQL do Azure, são copiados para o sistema OLAP, como o [Azure Analysis Services](/azure/analysis-services/analysis-services-overview). Ferramentas de exploração e visualização de dados como o [Power BI](https://powerbi.microsoft.com), Excel e opções de terceiros se conectam aos servidores do Analysis Services e fornecem aos usuários informações altamente interativas e visualmente ricas sobre os dados modelados. O fluxo de dados dos dados OLTP para o OLAP geralmente é orquestrado com o SQL Server Integration Services, que pode ser executado por meio do [Azure Data Factory](/azure/data-factory/concepts-integration-runtime).

No Azure, todos os seguintes armazenamentos de dados atenderão aos requisitos básicos do OLAP:

- [SQL Server com índices Columnstore](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [SQL Server Analysis Services (SSAS)](/sql/analysis-services/analysis-services)

O SSAS (SQL Server Analysis Services) oferece a funcionalidade de OLAP e mineração de dados para aplicativos de business intelligence. Você pode instalar o SSAS em servidores locais ou hospedá-lo em uma máquina virtual no Azure. O Azure Analysis Services é um serviço totalmente gerenciado que fornece os mesmos recursos principais do SSAS. O Azure Analysis Services dá suporte à conexão a [várias fontes de dados](/azure/analysis-services/analysis-services-datasource) na nuvem e locais em sua organização.

Índices Columnstore Clusterizados estão disponíveis no SQL Server 2014 e superior, bem como no Banco de Dados SQL do Azure e são ideais para cargas de trabalho OLAP. No entanto, a partir do SQL Server 2016 (incluindo o Banco de Dados SQL do Azure), você pode aproveitar o HTAP (processamento transacional/de análise híbrido) com o uso de índices columnstore não clusterizados atualizáveis. O HTAP permite a execução do processamento OLTP e OLAP na mesma plataforma, o que elimina a necessidade de armazenar várias cópias dos dados e a necessidade de diferentes sistemas OLTP e OLAP. Para obter mais informações, consulte [Introdução ao Columnstore para análise operacional em tempo real](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics).

## <a name="key-selection-criteria"></a>Principais critérios de seleção

Para restringir as opções, comece respondendo a estas perguntas:

- Você deseja ter um serviço gerenciado em vez de gerenciar seus próprios servidores?

- Você precisa de autenticação segura usando o Azure AD (Azure Active Directory)?

- Você deseja realizar a análise em tempo real? Nesse caso, restrinja as opções àquelas que dão suporte à análise em tempo real. 

    A *análise em tempo real*, neste contexto, se aplica a uma única fonte de dados, como um aplicativo ERP (planejamento de recursos empresariais), que executará uma carga de trabalho operacional e uma carga de trabalho de análise. Caso precise integrar dados de várias fontes ou de desempenho extremo de análise usando dados pré-agregados, como cubos, você ainda poderá precisar de um data warehouse separado.

- Você precisa usar dados pré-agregados, por exemplo, para fornecer modelos semânticos que tornam a análise mais amigável aos negócios? Em caso afirmativo, escolha uma opção que dá suporte a cubos multidimensionais ou modelos semânticos de tabela. 

    O fornecimento de agregações pode ajudar os usuários a calcular agregações de dados com consistência. Os dados pré-agregados também podem fornecer um grande aumento de desempenho ao lidar com várias colunas em várias linhas. Os dados podem ser pré-agregados em cubos multidimensionais ou modelos semânticos de tabela.

- Você precisa integrar dados de várias fontes, além de seu armazenamento de dados OLTP? Nesse caso, considere opções que integram várias fontes de dados com facilidade.

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades.

### <a name="general-capabilities"></a>Funcionalidades gerais

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server com Índices Columnstore | Banco de dados SQL do Azure com Índices Columnstore |
| --- | --- | --- | --- | --- |
| É um serviço gerenciado | sim | Não | Não  | sim |
| Dá suporte a cubos multidimensionais | Não  | sim | Não | Não  |
| Dá suporte a modelos semânticos de tabela | sim | sim | Não | Não  |
| É integrado com facilidade a várias fontes de dados | sim | sim | Não <sup>1</sup> | Não <sup>1</sup> |
| Dá suporte à análise em tempo real | Não  | Não  | sim | sim |
| Exige que o processo copie dados das origens | sim | sim | Não | Não  |
| Integração com o Azure AD | sim | Não  | Não <sup>2</sup> | sim |

[1] Embora o SQL Server e o Banco de Dados SQL do Azure não possa ser usado para consultar e integrar várias fontes de dados externas, você ainda poderá criar um pipeline que faz isso para você usando o [SSIS](/sql/integration-services/sql-server-integration-services) ou o [Azure Data Factory](/azure/data-factory/). O SQL Server hospedado em uma VM do Azure traz opções adicionais, como servidores vinculados e o [PolyBase](/sql/relational-databases/polybase/polybase-guide). Para obter mais informações, consulte [Orquestração de pipeline, fluxo de controle e movimentação de dados](../technology-choices/pipeline-orchestration-data-movement.md).

[2] Não há suporte para a conexão ao SQL Server em execução em uma Máquina Virtual do Azure com uma conta do Azure AD. Use um conta de domínio do Active Directory.

### <a name="scalability-capabilities"></a>Funcionalidades de Escalabilidade

|                                                  | Azure Analysis Services | SQL Server Analysis Services | SQL Server com Índices Columnstore | Banco de dados SQL do Azure com Índices Columnstore |
|--------------------------------------------------|-------------------------|------------------------------|-------------------------------------|---------------------------------------------|
| Servidores regionais redundantes para alta disponibilidade |           sim           |              Não               |                 sim                 |                     sim                     |
|             Dá suporte à expansão da consulta             |           sim           |              Não               |                 sim                 |                     Não                       |
|          Escalabilidade dinâmica (escalar verticalmente)          |           sim           |              Não               |                 sim                 |                     Não                       |

