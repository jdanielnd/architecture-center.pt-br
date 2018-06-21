---
title: Escolhendo uma tecnologia de orquestração do pipeline de dados
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 17aeb871bc815793295ed610795e5e83de72c637
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/14/2018
ms.locfileid: "29288798"
---
# <a name="choosing-a-data-pipeline-orchestration-technology-in-azure"></a>Escolhendo uma tecnologia de orquestração do pipeline de dados no Azure

A maioria das soluções de Big Data consiste em operações repetidas de processamento de dados, encapsuladas em fluxos de trabalho. Um orquestrador de pipeline é uma ferramenta que ajuda a automatizar esses fluxos de trabalho. Um orquestrador pode agendar trabalhos, executar fluxos de trabalho e coordenar dependências entre tarefas.

## <a name="what-are-your-options-for-data-pipeline-orchestration"></a>Quais são as opções disponíveis para orquestração do pipeline de dados?

No Azure, os seguintes serviços e ferramentas atenderão aos requisitos básicos de orquestração do pipeline, fluxo de controle e movimentação de dados:

- [Azure Data Factory](/azure/data-factory/)
- [Oozie no HDInsight](/azure/hdinsight/hdinsight-use-oozie-linux-mac)
- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

Esses serviços e ferramentas podem ser usados de forma independente um do outro ou usados em conjunto para criar uma solução híbrida. Por exemplo, o IR (Integration Runtime) no Azure Data Factory V2 pode executar pacotes SSIS nativamente em um ambiente de computação gerenciado do Azure. Embora haja alguma sobreposição na funcionalidade entre esses serviços, há algumas diferenças importantes.

## <a name="key-selection-criteria"></a>Principais Critérios de Seleção

Para restringir as opções, comece respondendo a estas perguntas:

- Você precisa de funcionalidades de Big Data para mover e transformar seus dados? Geralmente, isso significa vários gigabytes a terabytes de dados. Em caso afirmativo, restrinja as opções àquelas que são mais adequadas para Big Data.

- Você precisa de um serviço gerenciado que pode operar em escala? Em caso afirmativo, selecione um dos serviços baseados em nuvem que não são limitados pelo poder de processamento local.

- Algumas das fontes de dados estão localizadas localmente? Nesse caso, procure opções que podem trabalhar com a nuvem e fontes de dados ou destinos locais.

- Os dados de origem estão armazenados no armazenamento de Blobs em um sistema de arquivos HDFS? Nesse caso, escolha uma opção que dá suporte a consultas do Hive.

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades.

### <a name="general-capabilities"></a>Funcionalidades gerais

| | Fábrica de dados do Azure | SQL Server Integration Services (SSIS) | Oozie no HDInsight
| --- | --- | --- | --- |
| Gerenciada | sim | Não  | sim |
| Baseado em nuvem | sim | Não (local) | sim |
| Pré-requisito | Assinatura do Azure | SQL Server  | Assinatura do Azure, cluster HDInsight |
| Ferramentas de gerenciamento | Portal do Azure, PowerShell, CLI, SDK do .NET | SSMS, PowerShell | Shell do Bash, API REST do Oozie, interface do usuário da Web do Oozie |
| Preços | Pagamento por uso | Licenciamento/pagamento de recursos | Nenhum custo adicional além da execução do cluster HDInsight |

### <a name="pipeline-capabilities"></a>Funcionalidades de pipeline

| | Fábrica de dados do Azure | SQL Server Integration Services (SSIS) | Oozie no HDInsight
| --- | --- | --- | --- |
| Copiar dados | sim | sim | sim |
| Transformações personalizadas | sim | sim | Sim (trabalhos do MapReduce, Pig e Hive) |
| Pontuação do Azure Machine Learning | sim | Sim (com script) | Não  |
| HDInsight sob demanda | sim | Não  | Não  |
| Lote do Azure | sim | Não  | Não  |
| Pig, Hive, MapReduce | sim | Não  | sim |
| Spark | sim | Não  | Não  |
| Executar o pacote SSIS | sim | sim | Não  |
| Controlar fluxo | sim | sim | sim |
| Acesso a dados locais | sim | sim | Não  |

### <a name="scalability-capabilities"></a>Funcionalidades de escalabilidade

| | Fábrica de dados do Azure | SQL Server Integration Services (SSIS) | Oozie no HDInsight
| --- | --- | --- | --- |
| Escalar verticalmente | sim | Não  | Não  |
| Expansão | sim | Não  | Sim (com a adição de nós de trabalho ao cluster) |
| Otimizado para Big Data | sim | Não  | sim |

