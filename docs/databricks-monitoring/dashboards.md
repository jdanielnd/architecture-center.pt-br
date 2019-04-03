---
title: Use painéis para visualizar as métricas do Azure Databricks
description: Como implantar um painel do Grafana para monitorar o desempenho no Azure Databricks
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 36fcd93f6ca757e8e750d0fcbbdf0311c08560b0
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887821"
---
# <a name="use-dashboards-to-visualize-azure-databricks-metrics"></a>Use painéis para visualizar as métricas do Azure Databricks

Este artigo mostra como configurar um painel do Grafana para monitorar trabalhos do Azure Databricks para problemas de desempenho.

[O Azure Databricks](/azure/azure-databricks/) é uma rápida, poderosa e de colaboração [Apache Spark](https://spark.apache.org/)– com base em serviço de análise que torna mais fácil desenvolver e implantar soluções de inteligência artificial (AI) e análise de big data rapidamente. O monitoramento é um componente crítico de operacionais de cargas de trabalho do Databricks do Azure em produção. A primeira etapa é reunir métricas em um espaço de trabalho para análise. No Azure, é a melhor solução para gerenciamento de dados de log [do Azure Monitor](/azure/azure-monitor/). O Azure Databricks não oferece suporte a enviar dados de log para o Azure monitor, mas uma [biblioteca para essa funcionalidade](https://github.com/mspnp/spark-monitoring) está disponível no [Github](https://github.com).

Essa biblioteca ativa o log de métricas de serviço do Azure Databricks, bem como a estrutura do Apache Spark streaming métricas de consulta de evento. Depois de implantar com êxito essa biblioteca a um cluster do Azure Databricks, você pode implantar mais um conjunto de [Grafana](https://granfana.com) painéis que podem ser implantados como parte de seu ambiente de produção.

![Captura de tela do painel](./_images/dashboard-screenshot.png)

## <a name="prequisites"></a>Pré-requisitos

Clone o [repositório Github](https://github.com/mspnp/spark-monitoring) e [siga as instruções de implantação](./configure-cluster.md) para criar e configurar o registro em log do Azure Monitor para a biblioteca do Azure Databricks enviar logs ao seu espaço de trabalho do Log Analytics do Azure.

## <a name="deploy-the-azure-log-analytics-workspace"></a>Implantar o espaço de trabalho do Log Analytics do Azure

Para implantar o espaço de trabalho do Log Analytics do Azure, siga estas etapas:

1. Navegue até o `/perftools/deployment/loganalytics` directory.
1. Implantar o **logAnalyticsDeploy.json** modelo do Resource Manager. Para obter mais informações sobre como implantar modelos do Resource Manager, consulte [implantar recursos com modelos do Resource Manager e a CLI do Azure][rm-cli]. O modelo tem os seguintes parâmetros:

    * **local**: A região em que o espaço de trabalho do Log Analytics e os painéis são implantados.
    * **serviceTier**: Tipo de preço do espaço de trabalho do trocador. Ver [aqui] [ sku] para obter uma lista de valores válidos.
    * **dataRetention** (opcional): O número de dias de dados de log é mantido no espaço de trabalho do Log Analytics. O valor padrão é de 30 dias. Se o tipo de preço é `Free`, a retenção de dados deve ser 7 dias.
    * **WorkspaceName** (opcional): Um nome para o espaço de trabalho. Se não for especificado, o modelo gera um nome.

    ```bash
    az group deployment create --resource-group <resource-group-name> --template-file logAnalyticsDeploy.json --parameters location='East US' serviceTier='Standalone'
    ```

Este modelo cria o espaço de trabalho e também cria um conjunto de consultas predefinidas que são usadas pelo painel.

## <a name="deploy-grafana-in-a-virtual-machine"></a>Implantar o Grafana em uma máquina virtual

Grafana é um projeto de código-fonte aberto que pode ser implantado para visualizar as métricas de série de tempo armazenadas em seu espaço de trabalho do Log Analytics do Azure usando o plug-in do Grafana para o Azure Monitor. Grafana é executado em uma máquina virtual (VM) e requer uma conta de armazenamento, rede virtual e outros recursos. Para implantar uma máquina virtual com o bitnami certified Grafana imagem e os recursos associados, siga estas etapas:

1. Use a CLI do Azure para aceitar os termos de imagem do Azure Marketplace para o Grafana.

    ```bash
    az vm image accept-terms --publisher bitnami --offer grafana --plan default
    ```

1. Navegue até o `/spark-monitoring/perftools/deployment/grafana` diretório em sua cópia local do repositório GitHub.
1. Implantar o **grafanaDeploy.json** modelo do Resource Manager da seguinte maneira:

    ```bash
    export DATA_SOURCE="https://raw.githubusercontent.com/mspnp/spark-monitoring/master/perftools/deployment/grafana/AzureDataSource.sh"
    az group deployment create \
        --resource-group <resource-group-name> \
        --template-file grafanaDeploy.json \
        --parameters adminPass='<vm password>' dataSource=$DATA_SOURCE
    ```

Depois que a implantação for concluída, a imagem bitnami do Grafana é instalada na máquina virtual.

## <a name="update-the-grafana-password"></a>Atualizar a senha do Grafana

Como parte do processo de instalação, o script de instalação do Grafana gera uma senha temporária para o **admin** usuário. Você precisa dessa senha temporária para entrar. Para obter a senha temporária, siga estas etapas:  

1. Faça logon no Portal do Azure.  
1. Selecione o grupo de recursos onde os recursos foram implantados.
1. Selecione a VM em que o Grafana foi instalado. Se você usou o nome do parâmetro padrão no modelo de implantação, o nome da VM seja prefixado com **sparkmonitoring-vm-grafana**.
1. No **suporte + solução de problemas** seção, clique em **diagnóstico de inicialização** para abrir a página de diagnóstico de inicialização.
1. Clique em **log Serial** na página de diagnóstico de inicialização.
1. Procure a cadeia de caracteres a seguir: "Definir senha de aplicativo Bitnami como".
1. Copie a senha para um local seguro.

Em seguida, altere a senha de administrador do Grafana seguindo estas etapas:

1. No portal do Azure, selecione a VM e clique em **visão geral**.
1. Copie o endereço IP público.
1. Abra um navegador da web e navegue até a seguinte URL: `http://<IP addresss>:3000`.
1. O log do Grafana na tela, insira **admin** para o nome de usuário e use a senha do Grafana das etapas anteriores.
1. Depois de conectado, selecione **configuração** (o ícone de engrenagem).
1. Selecione **administrador do servidor**.
1. Sobre o **os usuários** guia, selecione o **admin** logon.
1. Atualize a senha.

## <a name="create-an-azure-monitor-data-source"></a>Criar uma fonte de dados do Azure Monitor

1. Crie um serviço principal que permite que o Grafana para gerenciar o acesso ao seu espaço de trabalho do Log Analytics. Para obter mais informações, consulte [criar uma entidade de serviço com a CLI do Azure](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)

    ```bash
    az ad sp create-for-rbac --name http://<service principal name> --role "Log Analytics Reader"
    ```

1. Observe os valores para appId, senha e locatário na saída deste comando:

    ```json
    {
        "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "displayName": "azure-cli-2019-03-27-00-33-39",
        "name": "http://<service principal name>",
        "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. Faça logon no Grafana conforme descrito anteriormente. Selecione **Configuration** (o ícone de engrenagem) e, em seguida **fontes de dados**.
1. No **fontes de dados** , clique em **adicionar fonte de dados**.
1. Selecione **do Azure Monitor** como o tipo de fonte de dados.
1. No **as configurações** , digite um nome para a fonte de dados no **nome** caixa de texto.
1. No **detalhes de API do Azure Monitor** seção, insira as seguintes informações:

    * ID da Assinatura: Sua ID da assinatura do Azure.
    * Id do locatário: A ID de locatário anterior.
    * Id do cliente: O valor de "appId" anterior.
    * Segredo do cliente: O valor de "password" anterior.

1. No **detalhes de API do Azure Log Analytics** seção, verifique os **mesmo detalhes como a API do Azure Monitor** caixa de seleção.
1. Clique em **salvar e testar**. Se a fonte de dados do Log Analytics está configurada corretamente, uma mensagem de êxito será exibida.

## <a name="create-the-dashboard"></a>Criar painel

Crie os painéis no Grafana seguindo estas etapas:

1. Navegue até o `/perftools/dashboards/grafana` diretório em sua cópia local do repositório GitHub.
1. Execute o seguinte script:

    ```bash
    export WORKSPACE=<your Azure Log Analytics workspace ID>
    export LOGTYPE=SparkListenerEvent_CL

    sh DashGen.sh
    ```

    A saída do script é um arquivo chamado **SparkMonitoringDash.json**.

1. Retorne ao painel do Grafana e selecione **criar** (o ícone de adição).
1. Selecione **Importar**.
1. Clique em **carregar arquivo. JSON**.
1. Selecione o **SparkMonitoringDash.json** arquivo criado na etapa 2.
1. No **opções** seção, no **ALA**, selecione a fonte de dados do Azure Monitor criada anteriormente.
1. Clique em **Importar**.

## <a name="visualizations-in-the-dashboards"></a>Visualizações em dashboards

O Azure Log Analytics e o Grafana painéis incluem um conjunto de visualizações de série temporal. Cada gráfico é um gráfico de série temporal de dados de métrica relacionados a um Apache Spark [trabalho](https://spark.apache.org/docs/latest/job-scheduling.html), etapas de trabalho e tarefas que compõem cada estágio.

As visualizações são da seguinte maneira:

### <a name="job-latency"></a>Latência do trabalho

Esta visualização mostra uma latência de execução para um trabalho, que é um modo de exibição não refinado sobre o desempenho geral de um trabalho. Exibe a duração da execução do trabalho do início até a conclusão. Observe que a hora de início do trabalho não é o mesmo que a hora de envio do trabalho. Latência é representada como percentuais (10%, 30%, 50%, 90%.) de execução do trabalho indexada por ID do cluster e a ID do aplicativo.

### <a name="stage-latency"></a>Latência de estágio

A visualização mostra a latência de cada estágio por cluster, por aplicativo e por estágio individual. Essa visualização é útil para identificar um estágio específico que está sendo executado lentamente.

### <a name="task-latency"></a>Latência de tarefa

Esta visualização mostra uma latência de execução de tarefa. Latência é representada como um percentual da execução de tarefa por cluster, o nome do estágio e o aplicativo.

### <a name="sum-task-execution-per-host"></a>Execução da tarefa soma por host

Esta visualização mostra a soma da latência de execução de tarefa por execução em um cluster de host. Exibindo a latência de execução de tarefa por host identifica os hosts que têm muito mais alta latência geral de tarefa que outros hosts. Isso pode significar que as tarefas foram ineficiente ou sem uniformidade distribuídas aos hosts.

### <a name="task-metrics"></a>Métricas de tarefa

Esta visualização mostra um conjunto de métricas de execução para a execução de uma determinada tarefa. Essas métricas incluem o tamanho e a duração de uma ordem aleatória de dados, duração das operações de serialização e desserialização e outros. Para o conjunto completo de métricas, exiba a consulta do Log Analytics para o painel. Essa visualização é útil para entender as operações que compõem uma tarefa e o consumo de recursos de identificação de cada operação. Picos no gráfico representam operações dispendiosas que devem ser investigadas.

### <a name="cluster-throughput"></a>Taxa de transferência de cluster

Essa visualização é uma exibição de alto nível de itens de trabalho indexado pelo cluster e aplicativo para representar a quantidade de trabalho realizado por cluster e aplicativo. Ele mostra o número de trabalhos, tarefas e etapas concluídas por cluster, o aplicativo e o estágio em incrementos de um minuto. 

### <a name="streaming-throughputlatency"></a>Streaming de taxa de transferência/latência

Este visualzation está relacionado às métricas associadas a uma consulta de streaming estruturada. Os grafos mostra o número de linhas de entrada por segundo e o número de linhas processadas por segundo. As métricas de streaming também são representadas por aplicativo. Essas métricas são enviadas quando o evento OnQueryProgress é gerado conforme a consulta de streaming estruturada é processada e a visualização representa streaming latência como a quantidade de tempo, em milissegundos, necessário para executar um lote de consultas.

### <a name="resource-consumption-per-executor"></a>Consumo de recursos por executor

Em seguida, é um conjunto de visualizações para mostrar painel de determinado tipo de recurso e como eles são consumidos por executor em cada cluster. Estas visualizações ajudam a identificar exceções no consumo de recursos por executor. Por exemplo, se a alocação de trabalho para um executor específico estiverem distorcida, consumo de recursos será elevado em relação a outros executores em execução no cluster. Isso pode ser identificado por picos no consumo de recursos para um executor.

### <a name="executor-compute-time-metrics"></a>Métricas de tempo de computação do executor

Em seguida, é um conjunto de visualizações para o dashboard que mostram a proporção de executor serializar tempo, desserializar o tempo, o tempo de CPU e tempo de máquina virtual Java para o tempo de computação geral executor. Isso demonstra visualmente quanto cada uma dessas quatro métricas estão contribuindo para o executor geral de processamento.

### <a name="shuffle-metrics"></a>Métricas de ordem aleatória

O conjunto final de Mostrar visualizações de dados em ordem aleatória métricas associadas a uma consulta de streaming estruturada em todos os executores. Isso inclui a ordem aleatória de bytes de leitura, em ordem aleatória de bytes gravados, embaralhar memória e uso do disco em consultas em que o sistema de arquivos é usado.

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Solucionar problemas de gargalos de desempenho](./performance-troubleshooting.md)

<!-- links -->

[rm-cli]: /azure/azure-resource-manager/resource-group-template-deploy-cli
[sku]: /azure/templates/Microsoft.OperationalInsights/2015-11-01-preview/workspaces#sku-object