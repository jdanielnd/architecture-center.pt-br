---
title: Escolhendo uma tecnologia de processamento em lotes
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 0117798af82f2caa6704dc86e88be57f09c381ea
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>Escolhendo uma tecnologia de processamento em lotes no Azure

Com frequência, as soluções de Big Data usam trabalhos em lotes de execução longa para filtrar, agregar e, de outro modo, preparar os dados para análise. Normalmente, esses trabalhos envolvem a leitura de arquivos de origem de um armazenamento escalonável (como o HDFS, Azure Data Lake Store e Armazenamento do Azure), seu processamento e gravação da saída em novos arquivos no armazenamento escalonável. 

O principal requisito desses mecanismos de processamento em lotes é a capacidade de expandir cálculos, para manipular um grande volume de dados. No entanto, ao contrário do processamento em tempo real, o processamento em lotes deve ter latências (o tempo entre a ingestão de dados e o cálculo de um resultado) que são medidas em minutos a horas.

## <a name="what-are-your-options-when-choosing-a-batch-processing-technology"></a>Quais são as opções disponíveis ao escolher uma tecnologia de processamento em lotes?

No Azure, todos os seguintes armazenamentos de dados atenderão aos requisitos básicos do processamento em lotes:

- [Azure Data Lake Analytics](/azure/data-lake-analytics/)
- [SQL Data Warehouse do Azure](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [HDInsight com Spark](/azure/hdinsight/spark/apache-spark-overview)
- [HDInsight com Hive](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [HDInsight com Hive LLAP](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

## <a name="key-selection-criteria"></a>Principais critérios de seleção

Para restringir as opções, comece respondendo a estas perguntas:

- Você deseja ter um serviço gerenciado em vez de gerenciar seus próprios servidores?

- Você deseja criar a lógica do processamento em lotes de forma declarativa ou imperativa?

- Você executará o processamento em lotes em intermitências? Em caso afirmativo, considere opções que permitem que você pause o cluster ou cujo modelo de preço se baseia no trabalho em lotes.

- Você precisa consultar armazenamentos de dados relacionais junto com o processamento em lotes, por exemplo, para pesquisar dados de referência? Nesse caso, considere as opções que permitem consultar relational stores externos.

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades. 

### <a name="general-capabilities"></a>Funcionalidades gerais

| | Análise Azure Data Lake | SQL Data Warehouse do Azure | HDInsight com Spark | HDInsight com Hive | HDInsight com Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| É um serviço gerenciado | sim | sim | Sim <sup>1</sup> | Sim <sup>1</sup> | Sim <sup>1</sup> |
| Dá suporte à pausa de computação | Não  | sim | Não | Não  | Não  |
| Armazenamento de dados relacionais | sim | sim | Não | Não  | Não  |
| Programação | U-SQL | T-SQL | Python, Scala, Java, R | HiveQL | HiveQL |
| Paradigma de programação | Combinação de declarativo e imperativo  | Declarativo | Combinação de declarativo e imperativo | Declarativo | Declarativo | 
| Modelo de preços | Por trabalho em lotes | Por hora de cluster | Por hora de cluster | Por hora de cluster | Por hora de cluster |  

[1] Com configuração manual e dimensionamento.

### <a name="integration-capabilities"></a>Funcionalidades de integração

| | Análise Azure Data Lake | SQL Data Warehouse | HDInsight com Spark | HDInsight com Hive | HDInsight com Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Acesso por meio do Azure Data Lake Store | sim | sim | sim | sim | sim |
| Consulta por meio do Armazenamento do Azure | sim | sim | sim | sim | sim |
| Consulta por meio de relational stores externos | sim | Não  | sim | Não | Não  |

### <a name="scalability-capabilities"></a>Funcionalidades de escalabilidade

| | Análise Azure Data Lake | SQL Data Warehouse | HDInsight com Spark | HDInsight com Hive | HDInsight com Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Granularidade de expansão  | Por trabalho | Por cluster | Por cluster | Por cluster | Por cluster |
| Rápida expansão (menos de 1 minuto) | sim | sim | Não | Não  | Não  |
| Cache em memória de dados | Não  | sim | sim | Não  | sim | 

### <a name="security-capabilities"></a>Funcionalidades de segurança

| | Análise Azure Data Lake | SQL Data Warehouse | HDInsight com Spark | Apache Hive no HDInsight | Hive LLAP no HDInsight |
| --- | --- | --- | --- | --- | --- |
| Autenticação  | Active Directory do Azure (Azure AD) | SQL/Azure AD | Não  | local/Azure AD <sup>1</sup> | local/Azure AD <sup>1</sup> |
| Autorização  | sim | sim| Não  | Sim <sup>1</sup> | Sim <sup>1</sup> |
| Auditoria  | sim | sim | Não  | Sim <sup>1</sup> | Sim <sup>1</sup> |
| Criptografia de dados em repouso | sim| Sim <sup>2</sup> | sim | sim | sim |
| Segurança em nível de linha | Não  | sim | Não  | Sim <sup>1</sup> | Sim <sup>1</sup> |
| Dá suporte a firewalls | sim | sim | sim | Sim <sup>3</sup> | Sim <sup>3</sup> |
| Mascaramento de dados dinâmicos | Não  | Não  | Não  | Sim <sup>1</sup> | Sim <sup>1</sup> |

[1] Exige o uso de um [cluster HDInsight ingressado no domínio](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Exige o uso de TDE (Transparent Data Encryption) para criptografar e descriptografar os dados em repouso.

[3] Compatível quando [usado em uma Rede Virtual do Azure](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
