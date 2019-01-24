---
title: ETL Híbrido com SSIS local existente e Azure Data Factory
titleSuffix: Azure Example Scenarios
description: ETL Híbrido com implantações SSIS (SQL Server Integration Services) locais existentes e Azure Data Factory.
author: alhieng
ms.date: 09/20/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: tsp-team
ms.openlocfilehash: 2fc5a94391b2e7e24209e884a790fe72070c4fc2
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485104"
---
# <a name="hybrid-etl-with-existing-on-premises-ssis-and-azure-data-factory"></a>ETL Híbrido com SSIS local existente e Azure Data Factory

As organizações que migram seus bancos de dados do SQL Server para a nuvem podem obter uma enorme economia de custos, ganhos de desempenho, flexibilidade adicional e maior escalabilidade. No entanto, a reformulação de processos existentes de ETL (extrair, transformar e carregar) criados com o SSIS (SQL Server Integration Services) pode ser um obstáculo de migração. Em outros casos, o processo de carregamento de dados requer lógica complexa e/ou componentes específicos da ferramenta de dados que ainda não têm suporte pelo Azure Data Factory v2. Os recursos do SSIS normalmente usados incluem transformações de Pesquisa Difusa e Agrupamento Difuso, Change Data Capture (CDC), Dimensões de Alteração Lenta (SCD) e Data Quality Services (DQS).

Uma abordagem de ETL híbrida pode ser a opção mais adequada para facilitar uma migração lift-and-shift de um banco de dados SQL existente. Uma abordagem híbrida usa o Data Factory como o principal mecanismo de orquestração, mas continua a aproveitar os pacotes SSIS existentes para limpar dados e trabalhar com recursos locais. Essa abordagem usa o IR (Tempo de Execução de Integração) do SQL Server do Data Factory para habilitar um lift-and-shift dos bancos de dados existentes para a nuvem enquanto usa o código existente e os pacotes do SSIS.

Este cenário de exemplo é relevante para organizações que estão movendo bancos de dados para a nuvem e considerando o uso do Data Factory como seu principal mecanismo ETL baseado na nuvem, ao mesmo tempo em que incorporam pacotes do SSIS existentes ao novo fluxo de trabalho de dados na nuvem. Muitas organizações investiram significativamente no desenvolvimento de pacotes SSIS ETL para tarefas de dados específicas. Regravar esses pacotes pode ser uma tarefa desestimulante. Além disso, muitos pacotes de código existentes têm dependências de recursos locais, impedindo a migração para a nuvem.

O Data Factory permite que os clientes aproveitem seus pacotes ETL existentes, limitando ainda mais o investimento no desenvolvimento de ETL local. Este exemplo discute possíveis casos de uso para aproveitar pacotes SSIS existentes como parte de um novo fluxo de trabalho de dados em nuvem usando o Azure Data Factory v2.

## <a name="potential-use-cases"></a>Possíveis casos de uso

Tradicionalmente, o SSIS tem sido a ferramenta de ETL preferida por muitos profissionais de dados do SQL Server para transformação e carregamento de dados. Às vezes, recursos SSIS específicos ou componentes de conexão de terceiros foram usados para acelerar o esforço de desenvolvimento. A substituição ou o novo desenvolvimento desses pacotes pode não ser uma opção, o que impede que os clientes migrem seus bancos de dados para a nuvem. Os clientes procuram abordagens de baixo impacto para migrar seus bancos de dados existentes para a nuvem e aproveitar os pacotes existentes do SSIS.

Veja abaixo vários possíveis casos de uso locais:

- Carregar logs do roteador de rede para um banco de dados de análise.
- Preparar dados de emprego de recursos humanos para relatórios analíticos.
- Carregar dados de produtos e vendas em um data warehouse para previsão de vendas.
- Automatizar o carregamento ou armazenamento de dados operacionais ou data warehouses para finanças e contabilidade.

## <a name="architecture"></a>Arquitetura

![Visão geral da arquitetura de um processo de ETL híbrido usando o Azure Data Factory][architecture-diagram]

1. Os dados são originados do armazenamento de Blobs no Data Factory.
2. O pipeline do Data Factory chama um procedimento armazenado para executar um trabalho do SSIS hospedado no local pelo Tempo de Execução de Integração.
3. As tarefas de limpeza de dados são executadas para preparar os dados para o consumo de downstream.
4. Depois que a tarefa de limpeza de dados for concluída com êxito, uma tarefa de cópia será executada para carregar os dados limpos no Azure.
5. Em seguida, os dados limpos são carregados em tabelas no SQL Data Warehouse.

### <a name="components"></a>Componentes

- O [armazenamento de blobs][docs-blob-storage] é usado para armazenar arquivos e também atua como uma fonte do Data Factory para recuperar dados.
- O [SQL Server Integration Services][docs-ssis] contém os pacotes de ETL locais usados para executar cargas de trabalho específicas da tarefa.
- O [Azure Data Factory][docs-data-factory] é o mecanismo de orquestração da nuvem que usa dados de várias fontes e combina, orquestra e carrega os dados em um data warehouse.
- O [SQL Data Warehouse][docs-sql-data-warehouse] centraliza os dados na nuvem para facilitar o acesso usando consultas SQL ANSI padrão.

### <a name="alternatives"></a>Alternativas

O Data Factory poderia invocar procedimentos de limpeza de dados implementados usando outras tecnologias, como um bloco de notas do Databricks, um script do Python ou uma instância do SSIS em execução em uma máquina virtual. [Instalar componentes personalizados pagos ou licenciados para o tempo de execução de integração do Azure-SSIS](/azure/data-factory/how-to-develop-azure-ssis-ir-licensed-components) pode ser uma alternativa viável à abordagem híbrida.

## <a name="considerations"></a>Considerações

O Tempo de Execução de Integração (IR) oferece suporte a dois modelos: IR auto-hospedado ou IR hospedado pelo Azure. Você primeiro deve decidir entre essas duas opções. A hospedagem interna é mais econômica. No entanto, tem mais sobrecarga de manutenção e gerenciamento. Para obter mais informações, confira [IR Auto-hospedado](/azure/data-factory/concepts-integration-runtime#self-hosted-integration-runtime). Se você precisar de ajuda para determinar qual IR usar, confira [Como determinar qual IR usar](/azure/data-factory/concepts-integration-runtime#determining-which-ir-to-use).

Na abordagem hospedada no Azure, você deve decidir quanta energia é necessária para processar seus dados. A configuração hospedada no Azure permite escolher o tamanho da VM como parte das etapas de configuração. Para saber mais sobre como escolher tamanhos de VM, confira [considerações sobre o desempenho de VMs](/azure/cloud-services/cloud-services-sizes-specs#performance-considerations).

A decisão é muito mais fácil quando você já tem pacotes SSIS existentes que possuem dependências locais, como fontes de dados ou arquivos que não podem ser acessados pelo Azure. Nesse cenário, sua única opção é o IR auto-hospedado. Essa abordagem fornece a maior flexibilidade para aproveitar a nuvem como o mecanismo de orquestração, sem precisar reescrever os pacotes existentes.

Em última análise, a intenção é mover os dados processados para a nuvem para serem ainda mais refinados ou combiná-los com outros dados armazenados na nuvem. Como parte do processo de design, acompanhe o número de atividades usadas nos pipelines do Data Factory. Para saber mais, confira [Pipelines e atividades no Azure Data Factory](/azure/data-factory/concepts-pipelines-activities).

## <a name="pricing"></a>Preços

O Data Factory é uma maneira econômica de orquestrar a movimentação de dados na nuvem. O custo é baseado em vários fatores.

- Número de execuções do pipeline
- Número de entidades/atividades usadas dentro do pipeline
- Número de operações de monitoramento
- Número de Execuções da Integração (IR hospedado no Azure ou IR auto-hospedado)

O Data Factory faz cobrança com base no consumo. Portanto, o custo só é incorrido durante as execuções e o monitoramento do pipeline. A execução de um pipeline básico custaria apenas 50 centavos e o monitoramento apenas 25 centavos. A [calculadora de custos do Azure](https://azure.microsoft.com/pricing/calculator/) pode ser usada para criar uma estimativa mais precisa com base em sua carga de trabalho específica.

Ao executar uma carga de trabalho de ETL híbrida, você deve levar em consideração o custo da máquina virtual usada para hospedar seus pacotes do SSIS. Esse custo é baseado no tamanho da VM que varia de um D1v2 (1 núcleo, 3,5 GB de RAM, disco de 50 GB) a um E64V3 (64 núcleos, 432 GB de RAM, disco de 1600 GB). Se você precisar de orientação adicional sobre a escolha do tamanho apropriado da VM, confira as [Considerações sobre o desempenho da VM](/azure/cloud-services/cloud-services-sizes-specs#performance-considerations).

## <a name="next-steps"></a>Próximas etapas

- Saiba mais sobre o [Azure Data Factory](https://azure.microsoft.com/services/data-factory/).
- Introdução ao Azure Data Factory seguindo o [tutorial passo a passo](/azure/data-factory/#step-by-step-tutorials).
- [Provisionar o Azure-SSIS Integration Runtime no Azure Data Factory](/azure/data-factory/tutorial-deploy-ssis-packages-azure).

<!-- links -->
[architecture-diagram]: ./media/architecture-diagram-hybrid-etl-with-adf.png
[small-pricing]: https://azure.com/e/
[medium-pricing]: https://azure.com/e/
[large-pricing]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[docs-blob-storage]: /azure/storage/blobs/
[docs-data-factory]: /azure/data-factory/introduction
[docs-resource-groups]: /azure/azure-resource-manager/resource-group-overview
[docs-ssis]: /sql/integration-services/sql-server-integration-services
[docs-sql-data-warehouse]: /azure/sql-data-warehouse/sql-data-warehouse-overview-what-is
