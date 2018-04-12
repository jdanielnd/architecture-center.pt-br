---
title: CQRS
description: Separar as operações que leem dados de operações que atualizam dados usando interfaces separadas.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: ce8d20ae82ae7d5ba00b4bc264a5c4d90fc383bd
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/17/2018
---
# <a name="command-and-query-responsibility-segregation-cqrs-pattern"></a>Padrão CQRS (Segregação de Responsabilidade de Consulta e Comando)

[!INCLUDE [header](../_includes/header.md)]

Separar as operações que leem dados de operações que atualizam dados usando interfaces separadas. Isso pode maximizar o desempenho, escalabilidade e segurança. Suporta a evolução do sistema ao longo do tempo através de maior flexibilidade e evita que comandos de atualização provoquem conflitos de mesclagem no nível de domínio.

## <a name="context-and-problem"></a>Contexto e problema

Nos sistemas de gerenciamento de dados tradicionais, ambos os comandos (atualizações para os dados) e consultas (solicitações para dados) são executados no mesmo conjunto de entidades em um repositório de dados único. Essas entidades podem ser um subconjunto das linhas em uma ou mais tabelas em um banco de dados relacional, como o SQL Server.

Normalmente, nesses sistemas, todas as operações CRUD (criar, ler, atualizar e excluir) são aplicadas na mesma representação da entidade. Por exemplo, um DTO (objeto de transferência de dados) que representa um cliente é recuperado do armazenamento de dados pela DAL (camada de acesso aos dados) e exibido na tela. Um usuário atualiza alguns campos do DTO (talvez através da vinculação de dados) e, em seguida, o DTO é salvo novamente no armazenamento de dados pela DAL. O mesmo DTO é utilizado para as operações de leitura e gravação. A figura ilustra uma arquitetura CRUD tradicional.

![Uma arquitetura CRUD tradicional](./_images/command-and-query-responsibility-segregation-cqrs-tradition-crud.png)

Os designs CRUD tradicionais funcionam bem quando apenas uma lógica de negócios limitada é aplicada às operações de dados. Os mecanismos de scaffold fornecidos pelas ferramentas de desenvolvimento podem criar código de acesso aos dados muito rapidamente, o que pode ser personalizado, conforme necessário.

No entanto, a abordagem CRUD tradicional possui algumas desvantagens:

- Isso significa que frequentemente há uma incompatibilidade entre as representações de leitura e gravação dos dados, como colunas ou propriedades adicionais que devem ser atualizadas corretamente mesmo que não sejam necessárias como parte de uma operação.

- Isso arrisca a contenção de dados quando os registros são bloqueados no armazenamento de dados em um domínio colaborativo, onde vários atores operam em paralelo no mesmo conjunto de dados. Ou atualizar conflitos causados por atualizações simultâneas quando o bloqueio otimista é utilizado. Esses riscos aumentam à medida que a complexidade e a produção do sistema crescem. Além disso, a abordagem tradicional pode ter um efeito negativo no desempenho devido à carga no armazenamento de dados e na camada de acesso aos dados, e a complexidade das consultas necessárias para recuperar informações.

- Isso pode tornar o gerenciamento de segurança e permissões mais complexo porque cada entidade está sujeita tanto a operações de leitura como de gravação, o que pode expor dados no contexto errado.

> Para uma compreensão mais profunda dos limites da abordagem CRUD, consulte [CRUD, Somente quando você puder custear isso](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/).

## <a name="solution"></a>Solução

O CQRS (Segregação de Responsabilidade de Consulta e Comando) é um padrão que segrega as operações que leem dados (consultas) das operações que atualizam dados (comandos) utilizando interfaces separadas. Isso significa que os modelos de dados utilizados para consultas e atualizações são diferentes. Os modelos podem, então, serem isolados, conforme mostrado na figura a seguir, embora isso não seja um requisito absoluto.

![Uma arquitetura CQRS básica](./_images/command-and-query-responsibility-segregation-cqrs-basic.png)

Comparado com o modelo de dados único utilizado em sistemas baseados em CRUD, o uso de modelos de atualização e consulta separados para os dados em sistemas baseados em CQRS simplifica o design e a implementação. No entanto, uma desvantagem é que, ao contrário de projetos CRUD, o código CQRS não pode ser gerado automaticamente utilizando mecanismos de scaffold.

O modelo de consulta para leitura de dados e o modelo de atualização para gravação de dados podem acessar o mesmo repositório físico, talvez utilizando exibições SQL ou gerando projeções instantâneas. No entanto, é comum separar os dados em diferentes repositórios físicos para maximizar o desempenho, a escalabilidade e a segurança, conforme mostrado na próxima figura.

![Uma arquitetura CQRS com repositórios de leitura e gravação separados](./_images/command-and-query-responsibility-segregation-cqrs-separate-stores.png)

O repositório de leitura pode ser uma réplica somente para leitura do repositório de gravação, ou repositórios de gravação e leitura podem ter uma estrutura completamente diferente. O uso de múltiplas réplicas de somente para leitura do repositório de leitura pode aumentar consideravelmente o desempenho de consulta e a capacidade de resposta da Interface de Usuário do aplicativo, especialmente em cenários distribuídos onde as réplicas de somente para leitura estão localizadas próximas às instâncias do aplicativo. Alguns sistemas de banco de dados (SQL Server) fornecem recursos adicionais, como réplicas de failover, para maximizar a disponibilidade.

A separação dos repositórios de gravação e leitura também permite que cada um seja dimensionado adequadamente para corresponder à carga. Por exemplo, os repositórios de leitura normalmente encontram uma carga muito maior do que os repositórios de gravação.

Quando o modelo leitura/consulta contém dados desnormalizados (consulte [Padrão de Exibição Materializada](materialized-view.md)), o desempenho é maximizado ao ler dados para cada uma das exibições em um aplicativo ou ao consultar os dados no sistema.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

- Dividir o armazenamento de dados em repositórios físicos separados para operações de gravação e leitura pode aumentar o desempenho e a segurança de um sistema, mas pode adicionar complexidade em termos de resiliência e coerência eventual. O repositório de modelos de leitura deve ser atualizado para refletir as alterações no repositório de gravação e pode ser difícil de detectar quando um usuário emitiu uma solicitação baseadas em dados de leitura obsoletos, o que significa que a operação não pode ser concluída.

    > Para obter uma descrição de coerência eventual, consulte [Primer de consistência de dados](https://msdn.microsoft.com/library/dn589800.aspx).

- Considere aplicar CQRS em seções limitadas do seu sistema, onde será mais valioso.

- Uma abordagem típica para a implantação de coerência eventual é usar o fornecimento de evento em conjunto com o CQRS, de modo que o modelo de gravação seja um stream de eventos somente para anexos, conduzido pela execução de comandos. Esses eventos são utilizados para atualizar exibições materializadas que agem como o modelo de leitura. Para obter mais informações, consulte [Fornecimento de evento e CQRS](/azure/architecture/patterns/cqrs#event-sourcing-and-cqrs).

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Utilize esse padrão nas seguintes situações:

- Domínios colaborativos onde várias operações são realizadas em paralelo nos mesmos dados. O CQRS permite que você defina comandos com granularidade suficiente para minimizar os conflitos de mesclagem no nível de domínio (todos os conflitos que surgem podem ser mesclados pelo comando), mesmo quando atualizar o que parece ser o mesmo tipo de dados.

- Interfaces de usuário baseadas em tarefas, onde os usuários são guiados por um processo complexo como uma série de etapas ou com modelos de domínio complexos. Além disso, é útil para equipes já familiarizadas com técnicas de design orientado por domínio (DDD). O modelo de gravação possui uma pilha de processamento de comando completa com lógica de negócios, validação de entrada e validação de negócios para garantir que tudo seja sempre consistente para cada um dos agregados (cada cluster de objetos associados tratados como uma unidade para alterações de dados) no modelo de gravação. O modelo de leitura não possui lógica de negócios ou pilha de validação e apenas retorna um DTO para uso em um modelo de exibição. O modelo de leitura é, eventualmente, consistente com o modelo de gravação.

- Cenários onde o desempenho das leituras de dados deve ter ajuste fino separadamente do desempenho das gravações de dados, especialmente quando a relação de gravação/leitura é muito alta e quando um dimensionamento horizontal é necessário. Por exemplo, em vários sistemas, o número de operações de leitura é muitas vezes maior que o número de operações de gravação. Para acomodar isso, considere dimensionar o modelo de leitura, mas executando o modelo de gravação em apenas uma ou algumas instâncias. Um pequeno número de instâncias de modelo de gravação também ajuda a minimizar a ocorrência de conflitos de mesclagem.

- Cenários onde uma equipe de desenvolvedores pode se concentrar no modelo de domínio complexo que faz parte do modelo de gravação e outra equipe pode se concentrar no modelo de leitura e nas interfaces de usuário.

- Cenários onde o sistema deve evoluir ao longo do tempo e pode conter várias versões do modelo, ou onde as regras de negócio mudam regularmente.

- Integração com outros sistemas, especialmente em combinação com fornecimento de evento, onde a falha temporal de um subsistema não deve afetar a disponibilidade dos outros.

Esse padrão não é recomendado nas seguintes situações:

- Onde o domínio ou as regras de negócios são simples.

- Onde uma interface de usuário com estilo de CRUD simples e as operações de acesso aos dados relacionadas são suficientes.

- Para implementação em todo o sistema. Existem componentes específicos de um cenário global de gerenciamento de dados em que o CQRS pode ser útil, mas pode adicionar uma complexidade considerável e desnecessária quando não for necessária.

## <a name="event-sourcing-and-cqrs"></a>Fornecimento de evento e CQRS

O padrão CQRS é frequentemente utilizado juntamente com o padrão de Fornecimento de Evento. Os sistemas baseados em CQRS utilizam modelos de dados de gravação e leitura separados, cada um adaptado a tarefas relevantes e, muitas vezes, localizado em repositórios separados fisicamente. Quando utilizado com o padrão [Fornecimento de Evento](event-sourcing.md), o repositório de eventos é o modelo de gravação e é a fonte oficial de informações. O modelo de leitura de um sistema baseado em CQRS fornece exibições materializadas dos dados, geralmente como exibições altamente desnormalizadas. Essas exibições são adaptadas às interfaces e aos requisitos de exibição do aplicativo, o que ajuda a maximizar tanto o desempenho de consulta como exibição.

Utilizando o stream de eventos como o repositório de gravação, em vez dos dados reais em um ponto no tempo, evita conflitos de atualização em um único agregado e maximiza o desempenho e a escalabilidade. Os eventos podem ser utilizados para gerar de maneira assíncrona exibições materializadas dos dados que são utilizadas para preencher o repositório de leitura.

Como o repositório de evento é a fonte oficial de informações, é possível excluir as exibições materializadas e reproduzir todos os eventos passados para criar uma nova representação do estado atual quando o sistema evoluir ou quando o modelo de leitura precisar alterar. As exibições materializadas são efetivamente um cache somente leitura durável dos dados.

Ao utilizar CQRS combinado com o padrão Fornecimento de Evento, considere o seguinte:

- Como em qualquer sistema onde os repositórios de gravação e leitura são separados, os sistemas baseados nesse padrão são eventualmente consistentes. Haverá algum atraso entre o evento que estiver sendo gerado e o armazenamento de dados sendo atualizado.

- O padrão adiciona complexidade porque o código deve ser criado para iniciar e tratar eventos e montar ou atualizar as exibições ou objetos apropriados exigidos por consultas ou um modelo de leitura. A complexidade do padrão CQRS quando utilizado com o padrão de Fornecimento de Evento pode dificultar a implementação bem-sucedida e requer uma abordagem diferente para a concepção de sistemas. No entanto, o fornecimento de evento pode tornar mais fácil para modelar o domínio e facilitar para recompilar exibições ou criar novas porque a intenção das alterações nos dados é preservada.

- A geração de exibições materializadas para uso no modelo de leitura ou projeções dos dados, reproduzindo e manipulando os eventos para entidades específicas ou coleções de entidades pode exigir um tempo de processamento significativo e uso de recursos. Isto é especialmente verdadeiro se requer soma ou análise de valores em longos períodos, pois poderá ser necessário examinar todos os eventos associados. Resolva isso implementando instantâneos dos dados em intervalos agendados, como uma contagem total do número de uma ação específica que ocorreu ou o estado atual de uma entidade.

## <a name="example"></a>Exemplo

O código a seguir mostra alguns extratos de um exemplo de uma implementação CQRS que utiliza diferentes definições para os modelos de leitura e gravação. As interfaces modelo não ditarão quaisquer recursos dos repositórios de dados subjacentes, e elas podem evoluir e terem ajustes finos de maneira independente porque essas interfaces estão separadas.

O código a seguir mostra a definição do modelo de leitura.

```csharp
// Query interface
namespace ReadModel
{
  public interface ProductsDao
  {
    ProductDisplay FindById(int productId);
    ICollection<ProductDisplay> FindByName(string name);
    ICollection<ProductInventory> FindOutOfStockProducts();
    ICollection<ProductDisplay> FindRelatedProducts(int productId);
  }

  public class ProductDisplay
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal UnitPrice { get; set; }
    public bool IsOutOfStock { get; set; }
    public double UserRating { get; set; }
  }

  public class ProductInventory
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public int CurrentStock { get; set; }
  }
}
```

O sistema permite aos usuários avaliar produtos. O código do aplicativo faz isso utilizando o comando `RateProduct` mostrado no código a seguir.

```csharp
public interface ICommand
{
  Guid Id { get; }
}

public class RateProduct : ICommand
{
  public RateProduct()
  {
    this.Id = Guid.NewGuid();
  }
  public Guid Id { get; set; }
  public int ProductId { get; set; }
  public int Rating { get; set; }
  public int UserId {get; set; }
}
```

O sistema utiliza a classe `ProductsCommandHandler` para lidar com comandos enviados pelo aplicativo. Normalmente, os clientes enviam comandos para o domínio através de um sistema de mensagens, como uma fila. O manipulador de comando aceita esses comandos e invoca métodos da interface de domínio. A granularidade de cada comando é projetada para reduzir a chance de solicitações conflitantes. O código a seguir mostra uma estrutura de tópicos da classe `ProductsCommandHandler`.

```csharp
public class ProductsCommandHandler :
    ICommandHandler<AddNewProduct>,
    ICommandHandler<RateProduct>,
    ICommandHandler<AddToInventory>,
    ICommandHandler<ConfirmItemShipped>,
    ICommandHandler<UpdateStockFromInventoryRecount>
{
  private readonly IRepository<Product> repository;

  public ProductsCommandHandler (IRepository<Product> repository)
  {
    this.repository = repository;
  }

  void Handle (AddNewProduct command)
  {
    ...
  }

  void Handle (RateProduct command)
  {
    var product = repository.Find(command.ProductId);
    if (product != null)
    {
      product.RateProduct(command.UserId, command.Rating);
      repository.Save(product);
    }
  }

  void Handle (AddToInventory command)
  {
    ...
  }

  void Handle (ConfirmItemsShipped command)
  {
    ...
  }

  void Handle (UpdateStockFromInventoryRecount command)
  {
    ...
  }
}
```

O código a seguir mostra a interface `IProductsDomain` do modelo de gravação.

```csharp
public interface IProductsDomain
{
  void AddNewProduct(int id, string name, string description, decimal price);
  void RateProduct(int userId, int rating);
  void AddToInventory(int productId, int quantity);
  void ConfirmItemsShipped(int productId, int quantity);
  void UpdateStockFromInventoryRecount(int productId, int updatedQuantity);
}
```

Observe também como a `IProductsDomain` interface contém métodos que têm um significado no domínio. Normalmente, em um ambiente CRUD, esses métodos teriam nomes genéricos, como `Save` ou `Update`, e um DTO como o único argumento. A abordagem CQRS pode ser projetada para atender às necessidades dos sistemas de gerenciamento de estoque e de negócios dessa organização.

## <a name="related-patterns-and-guidance"></a>Diretrizes e padrões relacionados

Os seguintes padrões e diretrizes serão úteis ao implementar esse padrão:

- Para uma comparação de CQRS com outros estilos arquitetônicos, consulte [Estilos de arquitetura](/azure/architecture/guide/architecture-styles/) e [Estilo de arquitetura CQRS](/azure/architecture/guide/architecture-styles/cqrs).

- [Primer de Consistência de Dados](https://msdn.microsoft.com/library/dn589800.aspx). Explica os problemas que normalmente são encontrados devido à coerência eventual entre os repositórios de dados de gravação e leitura ao utilizar o padrão CQRS e como esses problemas podem ser resolvidos.

- [Diretrizes de Particionamento de Dados](https://msdn.microsoft.com/library/dn589795.aspx). Descreve como os repositórios de dados de gravação e leitura usados no padrão CQRS podem ser divididos em partições que podem ser gerenciadas e acessadas separadamente para melhorar a escalabilidade, reduzir a contenção e otimizar o desempenho.

- [Padrão de Fornecimento de Evento](event-sourcing.md). Descreve detalhadamente como o Fornecimento de Eventos pode ser utilizado com o padrão CQRS para simplificar tarefas em domínios complexos, ao mesmo tempo em que melhora o desempenho, a escalabilidade e capacidade de resposta. Além disso, como fornecer consistência para dados transacionais, ao mesmo tempo que mantém trilhas de auditoria completas e histórico que podem permitir ações de compensação.

- [Padrão de Exibição Materializada](materialized-view.md). O modelo de leitura de uma implementação CQRS pode conter exibições materializadas dos dados do modelo de gravação, ou o modelo de leitura pode ser utilizado para gerar exibições materializadas.

- Guia de padrões e práticas [Recurso CQRS](http://aka.ms/cqrs). Em particular, a [Introdução ao Padrão de Segregação de Responsabilidade de Consulta de Comando](https://msdn.microsoft.com/library/jj591573.aspx) explora o padrão e quando isso é útil e o [Epílogo: Lições Aprendidas](https://msdn.microsoft.com/library/jj591568.aspx) ajuda-o a entender alguns dos problemas que surgiram ao usar este padrão.

- A postagem [CQRS por Martin Fowler](http://martinfowler.com/bliki/CQRS.html), que explica os conceitos básicos do padrão e links para outros recursos úteis.

- [Postagens de Greg Young](http://codebetter.com/gregyoung/), que exploram muitos aspectos do padrão CQRS.
