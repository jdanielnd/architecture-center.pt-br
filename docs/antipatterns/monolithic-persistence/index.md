---
title: Antipadrão de persistência monolítica
titleSuffix: Performance antipatterns for cloud apps
description: Colocar todos os dados de um aplicativo em um único armazenamento de dados pode prejudicar o desempenho.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: e3d6fbb789e422af22ce54b0defd6bb73099f15a
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487263"
---
# <a name="monolithic-persistence-antipattern"></a>Antipadrão de persistência monolítica

Colocar todos os dados de um aplicativo em um único armazenamento de dados pode prejudicar o desempenho, pois leva à contenção de recursos ou porque o armazenamento de dados não é uma boa opção para alguns dos dados.

## <a name="problem-description"></a>Descrição do problema

Historicamente, os aplicativos geralmente usavam um único armazenamento de dados, independentemente dos diferentes tipos de dados que o aplicativo precisasse armazenar. Geralmente, isso era feito para simplificar o design do aplicativo, ou para coincidir com o conjunto de habilidades existentes da equipe de desenvolvimento.

Sistemas baseados em nuvem modernos geralmente possuem requisitos funcionais e não funcionais adicionais e precisam armazenar muitos tipos heterogêneos de dados, como documentos, imagens, dados armazenados em cache, mensagens em fila, os logs de aplicativo e telemetria. Seguir a abordagem tradicional e colocar todas essas informações no mesmo armazenamento de dados pode prejudicar o desempenho, por dois motivos principais:

- Armazenar e recuperar grandes quantidades de dados não relacionados no mesmo armazenamento de dados pode causar contenção, que por sua vez, leva a tempos de resposta mais lentos e falhas de conexão.
- Independentemente do armazenamento de dados escolhido, talvez ele não seja o mais adequado para todos os diferentes tipos de dados ou ele pode não ser otimizado para as operações o aplicativo executa.

O exemplo a seguir mostra um controlador ASP.NET Web API que adiciona um novo registro em um banco de dados e também registra o resultado em um log. O log é mantido no mesmo banco de dados que os dados de negócios. Você pode encontrar o exemplo completo [aqui][sample-app].

```csharp
public class MonoController : ApiController
{
    private static readonly string ProductionDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        await DataAccess.LogAsync(ProductionDb, LogTableName);
        return Ok();
    }
}
```

A taxa na qual os registros de log são gerados provavelmente afetará o desempenho das operações de negócios. E se outro componente, como um monitor de processo do aplicativo, lê e processa os dados de log regularmente, isso também pode afetar as operações de negócios.

## <a name="how-to-fix-the-problem"></a>Como corrigir o problema

Dados separados de acordo com o uso. Para cada conjunto de dados, selecione um armazenamento de dados que melhor descreva como o conjunto de dados será usado. No exemplo anterior, o aplicativo deve fazer o registro em log em um armazenamento separado do banco de dados que contém dados de negócios:

```csharp
public class PolyController : ApiController
{
    private static readonly string ProductionDb = ...;
    private static readonly string LogDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        // Log to a different data store.
        await DataAccess.LogAsync(LogDb, LogTableName);
        return Ok();
    }
}
```

## <a name="considerations"></a>Considerações

- Separar dados conforma a maneira que eles são usados e acessados. Por exemplo, não armazene informações de log e dados de negócios no mesmo armazenamento de dados. Esses tipos de dados possuem requisitos e padrões de acesso diferentes. Os registros de log são inerentemente sequenciais, enquanto os dados de negócios mais provavelmente exigirão acesso aleatório e geralmente são relacionais.

- Considere o padrão de acesso a dados para cada tipo de dados. Por exemplo, armazene relatórios e documentos formatados em um banco de dados do documento como [Cosmos DB][CosmosDB], mas use o [Cache Redis do Azure][Azure-cache] para armazenar em cache dados temporários.

- Se você seguir essas diretrizes e, mesmo assim, alcançar os limites do banco de dados, você precisará escalar verticalmente o banco de dados. Também considere a possibilidade de escalar horizontalmente e particionar a carga entre os servidores de banco de dados. No entanto, pode ser que o particionamento exija um novo design do aplicativo. Para saber mais, consulte [Particionamento de dados][DataPartitioningGuidance].

## <a name="how-to-detect-the-problem"></a>Como detectar o problema

O sistema provavelmente ficará bem mais lento e eventualmente falhará, já que o sistema fica sem recursos, como conexões de banco de dados, por exemplo.

Você pode executar as etapas a seguir para ajudar a identificar a causa.

1. Instrumentar o sistema para registrar as principais estatísticas de desempenho. Capture informações de tempo para cada operação, bem como os pontos onde o aplicativo lê e grava dados.
2. Se possível, monitore o sistema em execução por alguns dias em um ambiente de produção para obter uma visão real de como o sistema é usado. Se isso não for possível, execute testes de carga com script com um volume realista de usuários virtuais executando uma série normal de operações.
3. Use os dados de telemetria para identificar os períodos de baixo desempenho.
4. Identifique quais armazenamentos de dados foram acessados durante esses períodos.
5. Identifique os recursos de armazenamento de dados que podem estar com problemas de contenção.

## <a name="example-diagnosis"></a>Diagnóstico de exemplo

As seções a seguir aplicam essas etapas ao aplicativo de exemplo descrito anteriormente.

### <a name="instrument-and-monitor-the-system"></a>Instrumentar e monitorar o sistema

O gráfico a seguir mostra os resultados de teste de carga para o aplicativo de exemplo descrito anteriormente. O teste usou uma carga de até 1.000 usuários simultâneos.

![Resultados de desempenho do teste de carga para o controlador baseado em SQL][MonolithicScenarioLoadTest]

À medida que a carga aumenta para 700 usuários, também aumenta a taxa de transferência. Mas nesse ponto, os níveis de taxa de transferência ficam extremamente elevados e o sistema parecem estar em execução em sua capacidade máxima. A média de resposta aumenta gradualmente junto com a carga de usuário, mostrando que o sistema não pode atender à demanda.

### <a name="identify-periods-of-poor-performance"></a>Identificar períodos de baixo desempenho

Se você estiver monitorando o sistema de produção, você poderá observar padrões. Por exemplo, os tempos de resposta podem diminuir significativamente na mesma hora todos os dias. Isso pode ser causado por uma carga de trabalho normal ou um trabalho em lotes agendado ou simplesmente porque o sistema tem mais usuários em determinados momentos. Você deve se concentrar nos dados de telemetria para esses eventos.

Procure as correlações entre tempos de resposta maiores e maior atividade de banco de dados ou E/S para recursos compartilhados. Se houver correlações, isso significa que o banco de dados pode ser um gargalo.

### <a name="identify-which-data-stores-are-accessed-during-those-periods"></a>Identifique quais armazenamentos de dados são acessados durante esses períodos

O gráfico a seguir mostra a utilização de unidades da taxa de transferência do banco de dados (DTU) durante o teste de carga. (Uma DTU é uma medida da capacidade disponível e é uma combinação de utilização da CPU, alocação de memória e a taxa de E/S). A utilização de DTUs rapidamente atingiu 100%. Esse é aproximadamente o ponto de pico da taxa de transferência no gráfico anterior. A utilização do banco de dados permanece muito alta, até que o teste seja concluído. Há uma pequena queda perto do final, que pode ser causada pela limitação, concorrência por conexões de banco de dados ou outros fatores.

![O monitor de banco de dados no portal clássico do Azure mostrando a utilização de recursos do banco de dados][MonolithicDatabaseUtilization]

### <a name="examine-the-telemetry-for-the-data-stores"></a>Examinar a telemetria para os armazenamentos de dados

Instrumente os armazenamentos de dados para capturar os detalhes de baixo nível da atividade. No aplicativo de exemplo, as estatísticas de acesso de dados mostraram um alto volume de operações de inserção executadas na tabela `PurchaseOrderHeader` e na tabela `MonoLog`.

![As estatísticas de acesso a dados para o aplicativo de exemplo][MonolithicDataAccessStats]

### <a name="identify-resource-contention"></a>Identificar contenção de recurso

Neste ponto, você pode examinar o código-fonte, concentrando-se nos pontos onde os recursos em contenção são acessados pelo aplicativo. Procure situações, como:

- Dados que são logicamente separados sendo gravados no mesmo armazenamento. Dados como mensagens em fila, relatórios e logs não devem ser mantidos no mesmo banco de dados que as informações de negócios.
- Uma incompatibilidade entre a opção de armazenamento de dados e o tipo de dados, como blobs grandes ou documentos XML em um banco de dados relacional.
- Dados com padrões de uso significativamente diferentes que compartilham o mesmo armazenamento, como dados de gravação alta/leitura baixa sendo armazenados com dados de gravação baixa/leitura alta.

### <a name="implement-the-solution-and-verify-the-result"></a>Implementar a solução e verificar o resultado

O aplicativo foi alterado para gravar logs em um armazenamento de dados separado. Aqui estão os resultados do teste de carga:

![Resultados de desempenho de teste de carga usando o controlador Polyglot][PolyglotScenarioLoadTest]

O padrão de taxa de transferência é semelhante ao gráfico anterior, mas o ponto de pico do desempenho é de aproximadamente 500 solicitações por segundo mais alto. O tempo médio de resposta é um pouco menor. No entanto, essas estatísticas não mostram tudo. A telemetria para o banco de dados de negócios mostra que a utilização de DTU atinge o pico em torno de 75%, em vez de 100%.

![O monitor do banco de dados no portal clássico do Azure mostrando a utilização de recursos do banco de dados no cenário do Polyglot][PolyglotDatabaseUtilization]

Da mesma forma, a utilização máxima de DTU do banco de dados de log atinge somente cerca de 70%. Os bancos de dados não são mais o fator limitante no desempenho do sistema.

![O monitor do banco de dados no portal clássico do Azure mostrando a utilização de recursos do banco de dados de log no cenário do Polyglot][LogDatabaseUtilization]

## <a name="related-resources"></a>Recursos relacionados

- [Escolha o armazenamento de dados correto][data-store-overview]
- [Critérios para escolher um armazenamento de dados][data-store-comparison]
- [Acesso a dados para soluções altamente escalonáveis: Usando SQL, NoSQL e persistência poliglota][Data-Access-Guide]
- [Particionamento de dados][DataPartitioningGuidance]

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/MonolithicPersistence
[CosmosDB]: https://azure.microsoft.com/services/cosmos-db/
[Azure-cache]: /azure/redis-cache/
[Data-Access-Guide]: https://msdn.microsoft.com/library/dn271399.aspx
[DataPartitioningGuidance]: ../../best-practices/data-partitioning.md
[data-store-overview]: ../../guide/technology-choices/data-store-overview.md
[data-store-comparison]: ../../guide/technology-choices/data-store-comparison.md

[MonolithicScenarioLoadTest]: _images/MonolithicScenarioLoadTest.jpg
[MonolithicDatabaseUtilization]: _images/MonolithicDatabaseUtilization.jpg
[MonolithicDataAccessStats]: _images/MonolithicDataAccessStats.jpg
[PolyglotScenarioLoadTest]: _images/PolyglotScenarioLoadTest.jpg
[PolyglotDatabaseUtilization]: _images/PolyglotDatabaseUtilization.jpg
[LogDatabaseUtilization]: _images/LogDatabaseUtilization.jpg
