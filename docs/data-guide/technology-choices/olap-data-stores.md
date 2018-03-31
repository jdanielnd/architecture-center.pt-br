---
title: Escolhendo um armazenamento de dados OLAP
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f3041b95696c9408a2c9ab747fe1ec3041db0743
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-an-olap-data-store-in-azure"></a>Escolhendo um armazenamento de dados OLAP no Azure

O OLAP (processamento analítico online) é uma tecnologia que organiza bancos de dados comerciais grandes e dá suporte à análise complexa. Este tópico compara as opções de soluções OLAP no Azure.

> [!NOTE]
> Para obter mais informações sobre quando usar um armazenamento de dados OLAP, consulte [Processamento analítico online](../scenarios/online-analytical-processing.md).

## <a name="what-are-your-options-when-choosing-an-olap-data-store"></a>Quais são as opções disponíveis ao escolher um armazenamento de dados OLAP?

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
| É um serviço gerenciado | sim | Não  | Não  | sim |
| Dá suporte a cubos multidimensionais | Não  | sim | Não  | Não  |
| Dá suporte a modelos semânticos de tabela | sim | sim | Não  | Não  |
| É integrado com facilidade a várias fontes de dados | sim | sim | Não <sup>1</sup> | Não <sup>1</sup> |
| Dá suporte à análise em tempo real | Não  | Não  | sim | sim |
| Exige que o processo copie dados das origens | sim | sim | Não  | Não  |
| Integração com o Azure AD | sim | Não  | Não <sup>2</sup> | sim |

[1] Embora o SQL Server e o Banco de Dados SQL do Azure não possa ser usado para consultar e integrar várias fontes de dados externas, você ainda poderá criar um pipeline que faz isso para você usando o [SSIS](/sql/integration-services/sql-server-integration-services) ou o [Azure Data Factory](/azure/data-factory/). O SQL Server hospedado em uma VM do Azure traz opções adicionais, como servidores vinculados e o [PolyBase](/sql/relational-databases/polybase/polybase-guide). Para obter mais informações, consulte [Orquestração de pipeline, fluxo de controle e movimentação de dados](../technology-choices/pipeline-orchestration-data-movement.md).

[2] Não há suporte para a conexão ao SQL Server em execução em uma Máquina Virtual do Azure com uma conta do Azure AD. Use um conta de domínio do Active Directory.

### <a name="scalability-capabilities"></a>Funcionalidades de Escalabilidade

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server com Índices Columnstore | Banco de dados SQL do Azure com Índices Columnstore |
| --- | --- | --- | --- | --- |
| Servidores regionais redundantes para alta disponibilidade  | sim | Não  | sim | sim |
| Dá suporte à expansão da consulta  | sim | Não  | sim | Não  |
| Escalabilidade dinâmica (escalar verticalmente)  | sim | Não  | sim | Não  |

