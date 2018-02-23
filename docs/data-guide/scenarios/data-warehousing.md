---
title: Data warehouse e data marts
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: eec883c68cf94637c3061814d0841c73b58d7e52
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/14/2018
---
# <a name="data-warehousing-and-data-marts"></a>Data warehouse e data marts

Um data warehouse é um repositório central, organizacional e relacional de dados integrados de uma ou mais fontes diferentes, em muitas ou todas as áreas de assunto. Data warehouses armazenam dados atuais e históricos e são usados para relatórios e análise dos dados de diferentes maneiras.

![Data warehouse no Azure](./images/data-warehousing.png)

Para mover dados para um data warehouse, eles são extraídos periodicamente de várias fontes que contêm informações comerciais importantes. Conforme os dados são movidos, eles podem ser formatados, limpos, validados, resumidos e reorganizados. Como alternativa, os dados podem ser armazenados no nível mais baixo de detalhe, com exibições agregadas fornecidas no warehouse para relatórios. Em ambos os casos, o data warehouse se torna um espaço de armazenamento permanente de dados usados para relatórios, análise e tomada de decisões comerciais importantes usando ferramentas de BI (business intelligence).

## <a name="data-marts-and-operational-data-stores"></a>Data marts e armazenamentos de dados operacionais

O gerenciamento de dados em escala é complexo e é cada vez menos comum ter um único data warehouse que representa todos os dados em toda a empresa. Em vez disso, as organizações criam data warehouses menores e mais direcionados, chamados *data marts*, que expõem os dados desejados para fins de análise. Um processo de orquestração popula os data marts com os dados mantidos em um armazenamento de dados operacionais. O armazenamento de dados operacionais atua como um intermediário entre o sistema transacional de origem e o data mart. Os dados gerenciados pelo armazenamento de dados operacionais são uma versão limpa dos dados presentes no sistema transacional de origem e, normalmente, são um subconjunto dos dados históricos mantidos pelo data warehouse ou data mart. 

## <a name="when-to-use-this-solution"></a>Quando usar esta solução

Escolha um data warehouse quando precisar transformar grandes quantidades de dados de sistemas operacionais em um formato que seja fácil de entender, atual e preciso. Os data warehouses não precisam seguir a mesma estrutura de dados concisa que pode estar sendo usada nos bancos de dados operacional/OLTP. Você pode usar nomes de coluna que fazem sentido para analistas e usuários de negócios, reestruturar o esquema para simplificar as relações de dados e consolidar várias tabelas em uma. Essas etapas ajudam a orientar os usuários que precisam criar relatórios ad hoc ou criar relatórios e analisar os dados em sistemas de BI, sem a ajuda de um DBA (administrador de banco de dados) ou desenvolvedor de dados.

Considere o uso de um data warehouse quando precisar manter dados históricos separados de sistemas de transação de origem por motivos de desempenho. Os data warehouses facilitam o acesso aos dados históricos em vários locais, fornecendo um local centralizado usando formatos, chaves, common data service e métodos de acesso comuns.

Os data warehouses são otimizados para acesso de leitura, o que resulta na geração mais rápida de relatórios comparado à execução de relatórios no sistema de transação de origem. Além disso, os data warehouses oferecem os seguintes benefícios:

* Todos os dados históricos de várias fontes podem ser armazenados e acessados em um data warehouse como a única fonte de verdade.
* Você pode melhorar a qualidade de dados limpando os dados conforme eles são importados para o data warehouse, fornecendo dados mais precisos, além de fornecer códigos e descrições consistentes.
* As ferramentas de relatórios não entram em conflito com os sistemas transacionais de origem para ciclos de processamento de consultas. Um data warehouse permite que o sistema transacional focalize predominantemente o tratamento de gravações, enquanto o data warehouse atende à maioria das solicitações de leitura.
* Um data warehouse pode ajudar a consolidar dados de diferentes softwares.
* As ferramentas de mineração de dados podem ajudá-lo a encontrar padrões ocultos usando metodologias automáticas em dados armazenados no warehouse.
* Os data warehouses facilitam o fornecimento de acesso seguro aos usuários autorizados, enquanto restringem o acesso a outras pessoas. Não é necessário conceder acesso aos usuários empresariais a dados de origem, removendo um vetor de ataque potencial contra um ou mais sistemas de transação de produção.
* Os data warehouses facilitam a criação de soluções de business intelligence com base nos dados, como [cubos OLAP](online-analytical-processing.md).

## <a name="challenges"></a>Desafios

A configuração correta de um data warehouse para atender às necessidades de sua empresa pode trazem alguns dos seguintes desafios:

* Confirmar o tempo necessário para modelar corretamente os conceitos de negócios. Essa é uma etapa importante, pois os data warehouses são controlados por informações, em que o mapeamento de conceitos impulsiona o restante do projeto. Isso envolve a padronização de termos relacionados aos negócios e formatos comuns (como moeda e datas) e a reestruturação do esquema de uma maneira que faça sentido para usuários empresariais, mas que ainda garanta a precisão das relações e agregações de dados.
* Planejar e configurar a orquestração de dados. As considerações incluem como copiar dados do sistema transacional de origem para o data warehouse e quando mover dados históricos para fora dos armazenamentos de dados operacionais e para o warehouse.
* Manter ou melhorar a qualidade de dados, limpando os dados conforme eles são importados para o warehouse.

## <a name="data-warehousing-in-azure"></a>Data warehouse no Azure

No Azure, você pode ter uma ou mais fontes de dados, seja ela de transações do cliente ou de vários aplicativos de negócios usados por vários departamentos. Tradicionalmente, esses dados são armazenados em um ou mais bancos de dados [OLTP](online-transaction-processing.md). Os dados podem ser persistidos em outros meios de armazenamento, como compartilhamentos de rede, Azure Storage Blobs ou um data lake. Os dados também podem ser armazenados pelo próprio data warehouse ou em um banco de dados relacional, como o Banco de Dados SQL do Azure. A finalidade da camada de armazenamento de dados analíticos é atender às consultas emitidas por ferramentas de relatórios ou de análise no data warehouse ou data mart. No Azure, essa funcionalidade de repositório analítico pode ser atendida com o SQL Data Warehouse do Azure ou o Azure HDInsight usando o Hive ou a Consulta Interativa. Além disso, você precisará de algum nível de orquestração para periodicamente mover ou copiar dados do armazenamento de dados para o data warehouse, que pode ser feito com o Azure Data Factory ou o Oozie no Azure HDInsight.

Serviços relacionados:

* [Banco de Dados SQL do Azure](/azure/sql-database/)
* [SQL Server em uma VM](/sql/sql-server/sql-server-technical-documentation)
* [Data Warehouse do Azure](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
* [Apache Hive no HDInsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
* [Consulta Interativa (Hive LLAP) no HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)


## <a name="technology-choices"></a>Opções de tecnologia

- [Data warehouses](../technology-choices/data-warehouses.md)
- [Orquestração de pipeline](../technology-choices/pipeline-orchestration-data-movement.md)

