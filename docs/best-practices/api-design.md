---
title: Diretrizes de design de API
description: "Orientação sobre como criar uma API bem projetada."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 3ffadce1b0c4a4da808e52d61cff0b7f0b27de11
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="api-design"></a>Design de API
[!INCLUDE [header](../_includes/header.md)]

Muitas soluções modernas baseadas na Web utilizam serviços Web, hospedados por servidores Web, para fornecer funcionalidades para aplicativos de cliente remoto. As operações que um serviço Web expõe constituem uma API da Web. Uma API da Web bem projetada deve buscar apoiar:

* **Independência de plataforma**. Aplicativos cliente devem ser capazes de utilizar a API que fornece o serviço Web, sem exigências sobre o modo como os dados ou operações expostos pela API são implementados fisicamente. Isso requer que a API opere em conformidade com padrões comuns que permitem que um serviço Web e aplicativos de cliente concordem sobre quais formatos de dados utilizar e sobre a estrutura dos dados que são trocados entre aplicativos cliente e o serviço Web.
* **Evolução do serviço**. O serviço Web deve ser capaz de evoluir e adicionar (ou remover) funcionalidade independentemente de aplicativos cliente. Os aplicativos existentes do cliente devem ser capazes de continuar a operar sem alterações conforme os recursos fornecidos pelo serviço Web são modificados. Todas as funcionalidades também devem ser detectáveis, para que os aplicativos cliente possam utilizá-las plenamente.

A finalidade destas diretrizes é descrever as questões que você deve considerar ao criar uma API da Web.

## <a name="introduction-to-representational-state-transfer-rest"></a>Introdução ao REST (Representational State Transfer)
Em sua dissertação em 2000, Roy Fielding propôs uma abordagem de arquitetura alternativa para estruturar as operações expostas por serviços Web: REST. REST é um estilo arquitetural para a criação de sistemas distribuídos com base em hipermídia. Uma vantagem principal do modelo REST é que ele é baseado em padrões abertos e não vincula a implementação do modelo ou os aplicativos cliente que o acessam a nenhuma implementação específica. Por exemplo, um serviço Web REST poderia ser implementado usando a API Web ASP.NET da Microsoft, e aplicativos cliente poderiam ser desenvolvidos usando qualquer linguagem e conjunto de ferramentas capazes de gerar solicitações HTTP e analisar as respostas HTTP.

> [!NOTE]
> O REST é, na verdade, independente de qualquer protocolo subjacente e não está necessariamente ligado a HTTP. No entanto, as implementações mais comuns dos sistemas baseados em REST utilizam HTTP como o protocolo de aplicativo para enviar e receber solicitações. Este documento concentra-se no mapeamento de princípios REST para sistemas projetados para operar usando HTTP.
>
>

O modelo de REST usa um esquema de navegação para representar objetos e serviços em uma rede (mencionados como *funcionalidades*). Muitos sistemas que implementam REST geralmente usam o protocolo HTTP para transmitir solicitações para acessar esses recursos. Nesses sistemas, um aplicativo cliente envia uma solicitação na forma de um URI que identifica um recurso, além de um método HTTP (os mais comuns são GET, POST, PUT e DELETE) que indica a operação a ser executada nesse recurso.  O corpo da solicitação HTTP contém os dados necessários para executar a operação. O ponto que é importante entender é que o REST define um modelo de solicitação sem monitoração de estado. Solicitações HTTP devem ser independentes e podem ocorrer em qualquer ordem, portanto, a tentativa de reter informações de estado transitório entre as solicitações não é viável.  O único local onde as informações são armazenadas é nos próprios recursos, e cada solicitação deve ser uma operação atômica. Na verdade, um modelo REST implementa uma máquina de estado finito onde uma solicitação faz, em um recurso, a transição de um estado bem definido não transitório para outro.

> [!NOTE]
> A natureza sem monitoração de estado de solicitações individuais no modelo REST permite que um sistema construído seguindo esses princípios seja altamente dimensionável. Não é necessário manter qualquer afinidade entre um aplicativo cliente fazendo uma série de solicitações e os servidores Web específicos tratando dessas solicitações.
>
>

Outro ponto crucial na implementação de um modelo eficaz do REST é entender as relações entre os vários recursos aos quais o modelo fornece acesso. Esses recursos normalmente são organizados como coleções e relacionamentos. Por exemplo, suponha que uma rápida análise de um sistema de comércio eletrônico mostre que há duas coleções pelas quais aplicativos cliente provavelmente se interessam: pedidos e clientes. Cada pedido e cliente deve ter sua própria chave exclusiva para fins de identificação. O URI para acessar a coleção de pedidos pode ser algo tão simples quanto */orders* e, do mesmo modo, o URI para recuperar todos os clientes pode ser */customers*. Emitir uma solicitação HTTP GET para o URI */orders* deve retornar uma lista que representa todos os pedidos na coleção, codificada como uma resposta HTTP:

```HTTP
GET http://adventure-works.com/orders HTTP/1.1
...
```

A resposta mostrada abaixo codifica os pedidos como uma estrutura de lista JSON.

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
[{"orderId":1,"orderValue":99.90,"productId":1,"quantity":1},{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2},{"orderId":3,"orderValue":16.60,"productId":2,"quantity":4},{"orderId":4,"orderValue":25.90,"productId":3,"quantity":1},{"orderId":5,"orderValue":99.90,"productId":1,"quantity":1}]
```
Buscar um pedido individual requer a especificação do identificador para o pedido por meio do recurso *orders*, como */orders/2*:

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
```

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2}
```

> [!NOTE]
> Para simplificar, nestes exemplos mostram as informações nas respostas sendo retornadas como dados de texto JSON. No entanto, não há nenhuma razão pela qual recursos não devam conter qualquer outro tipo de dados para o qual HTTP oferece suporte, como informações binárias ou criptografadas; o tipo de conteúdo na resposta HTTP deve especificar o tipo. Além disso, um modelo REST pode ser capaz de retornar os mesmos dados em diferentes formatos, como XML ou JSON. Nesse caso, o serviço Web deve ser capaz de realizar a negociação de conteúdo com o cliente que está fazendo a solicitação. A solicitação pode incluir um cabeçalho *Accept* que especifica o formato preferencial que o cliente gostaria de receber, sendo que o serviço Web deve tentar honrar esse formato se possível.
>
>

Observe que a resposta de uma solicitação REST faz uso dos códigos de status HTTP padrão. Por exemplo, uma solicitação que retorna dados válidos deve incluir o código de resposta HTTP 200 (OK), enquanto uma solicitação que não consegue localizar ou excluir um recurso especificado deve retornar uma resposta que inclui o código de status HTTP 404 (Não Encontrado).

## <a name="design-and-structure-of-a-restful-web-api"></a>Design e a estrutura de uma API da Web RESTful
As chaves para a criação de uma API da Web bem-sucedida são simplicidade e consistência. Uma API da Web que exibe esses dois fatores facilita a criação de aplicativos cliente que precisam consumir a API.

Uma API da Web RESTful concentra-se na exposição de um conjunto de recursos conectados e no fornecimento das principais operações que permitem que um aplicativo manipule esses recursos e navegue facilmente entre eles. Por esse motivo, os URIs que constituem uma API da Web RESTful típica deve ser orientados para os dados que ela expõe, além de usar os recursos fornecidos pelo HTTP para realizar operações com esses dados. Essa abordagem exige uma mentalidade diferente daquela empregada normalmente durante a criação de um conjunto de classes em uma API orientada a objeto, que tende a ser mais motivada pelo comportamento de objetos e classes. Além disso, uma API da Web RESTful deve ser sem monitoração de estado e não deve depender de operações serem invocadas em uma sequência específica. As seções a seguir resumem os pontos que você deve considerar ao criar uma API da Web RESTful.

### <a name="organizing-the-web-api-around-resources"></a>Organizando a API da Web em torno de recursos
> [!TIP]
> Os URIs expostos por um serviço Web REST deve se basear em substantivos (os dados aos quais a API da Web fornece acesso) e não em verbos (o que um aplicativo pode fazer com os dados).
>
>

Concentre-se nas entidades comerciais que a API da Web expõe. Por exemplo, em uma API da Web projetada para oferecer suporte ao sistema de comércio eletrônico descrito anteriormente, as entidades principais são clientes e pedidos. Processos como o ato de fazer um pedido podem ser realizados pelo fornecimento de uma operação HTTP POST, que coleta as informações do pedido e adiciona-as à lista de pedidos para o cliente. Internamente, essa operação POST pode executar tarefas como verificação dos níveis de estoque e cobrança do cliente. A resposta HTTP pode indicar se o pedido foi feito com êxito ou não. Observe também que um recurso não precisa se basear em um único item de dados físico. Por exemplo, um recurso de pedido pode ser implementado internamente usando informações agregadas de muitas linhas distribuídas por várias tabelas em um banco de dados relacional, mas apresentadas ao cliente como uma única entidade.

> [!TIP]
> Evite a criação de uma interface REST que espelhe ou dependa da estrutura interna dos dados que ele expõe. REST tem maior foco na implementação de operações CRUD (Create, Retrieve, Update, Delete - criar, recuperar, atualizar, excluir) simples em tabelas separadas em um banco de dados relacional. A finalidade do REST é mapear entidades comerciais e as operações que um aplicativo pode executar nessas entidades para a implementação física dessas entidades, mas um cliente não deve ser exposto a esses detalhes físicos.
>
>

Entidades comerciais individuais raramente existem em isolamento (embora possam existir alguns objetos singleton), mas em vez disso, tendem a ser agrupadas em coleções. Em termos de REST, cada entidade e cada coleção são recursos. Em uma API da Web RESTful, cada coleção tem seu próprio URI contido no serviço Web, e executar uma solicitação HTTP GET por um URI para uma coleção recupera uma lista dos itens nessa coleção. Cada item individual também tem seu próprio URI, e um aplicativo pode enviar outra solicitação HTTP GET usando esse URI para obter os detalhes desse item. Você deve organizar os URIs para coleções e itens de maneira hierárquica. No sistema de comércio eletrônico, o URI */customers* denota a coleção de clientes, e */customers/5* recupera os detalhes para o único cliente com ID 5 desta coleção. Essa abordagem ajuda a manter a API da Web intuitiva.

> [!TIP]
> Adote uma convenção de nomenclatura consistente nos URIs; em geral, ajuda usar substantivos plurais para URIs que fazem referência a coleções.
>
>

Você também precisa considerar as relações entre diferentes tipos de recursos e como você pode expor essas associações. Por exemplo, os clientes podem fazer zero ou mais pedidos. Uma maneira natural de representar essa relação seria por meio de um URI como */clientes/5/orders* , para localizar todos os pedidos do cliente 5. Você também pode considerar a possibilidade de representar a associação de um pedido a um cliente específico, por meio de um URI como */orders/99/customer* para localizar o cliente para o pedido 99, mas estender muito esse modelo pode torná-lo difícil de implementar. Uma solução melhor é fornecer links navegáveis para recursos associados, como o cliente, no corpo da mensagem de resposta HTTP retornada quando se consulta o pedido. Esse mecanismo é descrito mais detalhadamente na seção Usando a Abordagem de HATEOAS Para Habilitar a Navegação Para Recursos Relacionados, posteriormente nestas diretrizes.

Em sistemas mais complexos pode haver muito mais tipos de entidade, e pode ser tentador fornecer URIs que permitem a um aplicativo cliente navegar por vários níveis de relações, como */customers/1/orders/99/products* para obter a lista de produtos no pedido 99 feito pelo cliente 1. No entanto, esse nível de complexidade pode ser difícil de manter e é inflexível no caso de as relações entre os recursos mudarem no futuro. Em vez disso, você deve tentar manter os URIs relativamente simples. Tenha em mente que uma vez que um aplicativo tem uma referência a um recurso, deve ser possível usar essa referência para localizar itens relacionados a esse recurso. A consulta anterior pode ser substituída pelo URI */customers/1/orders* para localizar todos os pedidos do cliente 1 e, em seguida, consultar o URI */orders/99/products* para encontrar os produtos nesse pedido (supondo que o pedido 99 tenha sido feito pelo cliente 1).

> [!TIP]
> Evite exigir URIs de recurso mais complexos do que *collection/item/collection*.
>
>

Outro ponto a considerar é que todas as solicitações da Web impõem uma carga ao servidor Web; quanto maior o número de solicitações, maior será a carga. Você deve tentar definir seus recursos para evitar APIs da Web "verborrágicas", que expõem um grande número de recursos pequenos. Uma API desse tipo pode exigir que um aplicativo cliente envie várias solicitações para localizar todos os dados que ele necessita. Pode ser benéfico desnormalizar dados e combinar informações relacionadas em recursos maiores, que podem ser recuperados por meio de uma única solicitação. No entanto, você precisa equilibrar essa abordagem em relação à sobrecarga causada pela coleta de dados que podem, frequentemente, não ser solicitados pelo cliente. Recuperar objetos grandes pode aumentar a latência de uma solicitação e incorre em custos de largura de banda adicional, o que traz pouca vantagem se os dados adicionais não são usados com frequência.

Evite introduzir as dependências entre a API da Web na estrutura, tipo ou local das fontes de dados subjacentes. Por exemplo, se seus dados estão localizados em um banco de dados relacional, a API da Web não precisa expor cada tabela como uma coleção de recursos. Considere a API da Web como uma abstração do banco de dados e, se necessário, introduza uma camada de mapeamento entre o banco de dados e a API da Web. Desse modo, se a implementação ou o design do banco de dados for alterado (por exemplo, você deixa de usar um banco de dados relacional que contém uma coleção de tabelas normalizadas e passa a usar um sistema de armazenamento NoSQL desnormalizado, como um banco de dados de documentos), aplicativos cliente serão protegidos contra essas alterações.

> [!TIP]
> A fonte de dados que serve de base para uma API da Web não precisa ser um armazenamento de dados; pode ser outro serviço ou aplicativo de linha de negócios, ou até mesmo um aplicativo herdado em execução local, dentro de uma organização.
>
>

Por fim, pode não ser possível mapear toda operação implementada por uma API da Web para um recurso específico. Você pode tratar desses cenários *sem recursos* por meio de solicitações HTTP GET, que invocam uma funcionalidade e retornam os resultados como uma mensagem de resposta HTTP. Uma API da Web que implementa operações simples estilo calculadora, como adicionar e subtrair, pode fornecer URIs que exponham essas operações como pseudorrecursos, além de utilizar a cadeia de consulta para especificar os parâmetros necessários. Por exemplo, uma solicitação GET para o URI */add?operand1=99&operand2=1* pode retornar uma mensagem de resposta com o corpo contendo o valor 100, enquanto a solicitação GET para o URI */subtract?operand1=50&operand2=20* pode retornar uma mensagem de resposta com o corpo contendo o valor 30. No entanto, use essas formas de URIs apenas com moderação.

### <a name="defining-operations-in-terms-of-http-methods"></a>Definindo operações em termos de métodos HTTP
O protocolo HTTP define vários métodos que atribuem significado semântico a uma solicitação. Os métodos HTTP comuns usados pelas APIs da Web mais RESTful são:

* **GET**, para obter uma cópia do recurso no URI especificado. O corpo da mensagem de resposta contém os detalhes do recurso solicitado.
* **POST**, para criar um novo recurso no URI especificado. O corpo da mensagem de solicitação fornece os detalhes do novo recurso. Observe que POST também pode ser usado para disparar operações que, na verdade, não criam recursos.
* **PUT**, para substituir ou atualizar o recurso no URI especificado. O corpo da mensagem de solicitação especifica o recurso a ser modificado e os valores a serem aplicados.
* **DELETE**, para remover o recurso no URI especificado.

> [!NOTE]
> O protocolo HTTP também define outros métodos menos usados como PATCH, que é usado para solicitar atualizações seletivas a um recurso; HEAD, que é usado para solicitar uma descrição de um recurso; OPTIONS, que permite a um cliente obter informações sobre as opções de comunicação para as quais há suporte no servidor; por fim, TRACE, que permite que um cliente solicite informações que pode usar para fins de teste e diagnóstico.
>
>

O efeito de uma solicitação específica deve depender de o recurso ao qual ela é aplicada tratar-se de uma coleção ou um item individual. A tabela a seguir resume as convenções comuns adotadas pelas implementações mais RESTful, usando o exemplo de comércio eletrônico. Observe que nem todas essas solicitações podem ser implementadas; isso depende do cenário específico.

| **Recurso** | **POST** | **GET** | **PUT** | **DELETE** |
| --- | --- | --- | --- | --- |
| /clientes |Criar um novo cliente |Obter todos os clientes |Atualização em massa de clientes (*se implementado*) |Remover todos os clientes |
| /clientes/1 |Erro |Obter os detalhes do cliente 1 |Atualizar os detalhes do cliente 1 se ele existir; caso contrário, retornar um erro |Remover cliente 1 |
| /clientes/1/pedidos |Criar um novo pedido para o cliente 1 |Obter todos os pedidos do cliente 1 |Atualização em massa de pedidos do cliente 1 (*se implementado*) |Remover todos os pedidos do cliente 1 (*se implementado*) |

A finalidade de solicitações GET e DELETE são relativamente simples, mas há escopo suficiente para confusão no que se refere à finalidade e aos efeitos das solicitações POST e PUT.

Uma solicitação POST deve criar um novo recurso com dados fornecidos no corpo da solicitação. No modelo de REST, você aplica frequentemente solicitações POST a recursos que são coleções; o novo recurso é adicionado à coleção.

> [!NOTE]
> Você também pode definir as solicitações POST que disparam alguma funcionalidade (e que não necessariamente retornam dados), e esses tipos de solicitação podem ser aplicados a coleções. Por exemplo, você poderia usar uma solicitação POST para transmitir um quadro de horários para um serviço de processamento de folha de pagamento e obter os impostos calculados como uma resposta.
>
>

Uma solicitação PUT destina-se a modificar um recurso existente. Se o recurso especificado não existir, a solicitação PUT poderia retornar um erro (em alguns casos ela pode até, na verdade, criar o recurso). Solicitações PUT são aplicadas com mais frequência aos recursos que são itens individuais (como um cliente ou pedido específico), embora elas possam ser aplicadas a coleções - o que, todavia, é implementado menos frequentemente. Observe que as solicitações PUT são idempotentes, enquanto as solicitações POST não são; se um aplicativo envia a mesma solicitação PUT várias vezes, os resultados devem ser sempre os mesmos (o mesmo recurso será modificado com os mesmos valores), mas se um aplicativo repete a mesma solicitação POST, o resultado será a criação de vários recursos.

> [!NOTE]
> A rigor, uma solicitação HTTP PUT substitui um recurso existente pelo recurso especificado no corpo da solicitação. Se a intenção é modificar uma seleção de propriedades em um recurso mantendo as demais propriedades inalteradas, isso deve ser implementado usando uma solicitação HTTP PATCH. No entanto, muitas implementações RESTful flexibilizam essa regra e usam PUT para ambas as situações.
>
>

### <a name="processing-http-requests"></a>Processando solicitações HTTP
Os dados incluídos por um aplicativo cliente em muitas solicitações HTTP e as mensagens de resposta correspondentes no servidor Web podem ser apresentados em uma variedade de formatos (ou tipos de mídia). Por exemplo, os dados que especificam os detalhes de um cliente ou pedido podem ser fornecidos como XML, JSON ou algum outro formato codificado e compactado. Uma API da Web RESTful deve oferecer suporte a diferentes tipos de mídia, conforme solicitado pelo aplicativo cliente que envia uma solicitação.

Quando um aplicativo cliente envia uma solicitação que retorna dados no corpo de uma mensagem ele pode especificar, no cabeçalho Accept da solicitação, os tipos de mídia com os quais pode lidar. O código a seguir ilustra uma solicitação HTTP GET que obtém os detalhes do pedido 2 e solicita que os resultados sejam retornados como JSON (o cliente ainda deve examinar o tipo de mídia dos dados na resposta, para verificar o formato dos dados retornados):

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
Accept: application/json
...
```

Se o servidor Web oferece suporte a esse tipo de mídia, ele pode enviar uma resposta que inclui o cabeçalho Content-Type, que especifica o formato dos dados no corpo da mensagem:

> [!NOTE]
> Para interoperabilidade máxima, a tipos referenciados nos cabeçalhos Accept e Content-Type devem ser tipos MIME reconhecidos, em vez de algum tipo personalizado de mídia.
>
>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

Se o servidor Web não oferece suporte ao tipo de mídia solicitado, ele pode enviar os dados em um formato diferente. Em todos os casos, ele deve especificar o tipo de mídia (como *application/json*) no cabeçalho Content-Type. É responsabilidade do aplicativo cliente analisar a mensagem de resposta e interpretar adequadamente os resultados no corpo da mensagem.

Observe que neste exemplo, o servidor Web recupera com êxito os dados solicitados e indica tal êxito, passando de volta um código de status 200 no cabeçalho de resposta. Se nenhum dado correspondente for encontrado, ele deverá retornar um código de status de 404 (não encontrado) e o corpo da mensagem de resposta pode conter informações adicionais. O formato dessas informações é especificado pelo cabeçalho Content-Type, conforme mostrado no exemplo a seguir:

```HTTP
GET http://adventure-works.com/orders/222 HTTP/1.1
...
Accept: application/json
...
```

O pedido 222 não existe, portanto, a mensagem de resposta tem essa aparência:

```HTTP
HTTP/1.1 404 Not Found
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"message":"No such order"}
```

Quando um aplicativo envia uma solicitação HTTP PUT para atualizar um recurso, ele especifica o URI do recurso e fornece os dados a serem modificados no corpo da mensagem de solicitação. Ele também deve especificar o formato desses dados por meio do cabeçalho Content-Type. Um formato comum usado para informações baseadas em texto é *application/x-www-form-urlencoded*, que inclui um conjunto de pares nome/valor separados pelo caractere &. O exemplo a seguir mostra uma solicitação HTTP PUT que modifica as informações no pedido 1:

```HTTP
PUT http://adventure-works.com/orders/1 HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
ProductID=3&Quantity=5&OrderValue=250
```

Se a modificação for bem-sucedida, ele deve idealmente responder com um código de status HTTP 204, indicando que o processo foi tratado com êxito, mas sem nenhuma informação adicional no corpo da resposta. O cabeçalho Location na resposta contém o URI do recurso recém-atualizado:

```HTTP
HTTP/1.1 204 No Content
...
Location: http://adventure-works.com/orders/1
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

> [!TIP]
> Se os dados em uma mensagem de solicitação HTTP PUT incluem informações de data e hora, certifique-se de que o serviço Web aceita datas e horários formatados segundo o padrão ISO 8601.
>
>

Se o recurso a ser atualizado não existe, o servidor Web pode responder com uma resposta “Não Encontrado”, conforme descrito anteriormente. Alternativamente, se o próprio servidor efetivamente cria o objeto, ele pode retornar os códigos de status HTTP 200 (OK) ou HTTP 201 (Criado) e o corpo da resposta pode conter os dados para o novo recurso. Se o cabeçalho Content-Type da solicitação especifica um formato de dados que o servidor Web não pode processar, ele deve responder com o código de status HTTP 415 (Tipo de Mídia Sem Suporte).

> [!TIP]
> Considere a possibilidade de implementar operações HTTP PUT em massa, que podem realizar atualizações em lote para vários recursos em uma coleção. A solicitação PUT deve especificar o URI da coleção, enquanto o corpo da solicitação deve especificar os detalhes dos recursos a serem modificados. Essa abordagem pode ajudar a reduzir a verborragia e a melhorar o desempenho.
>
>

O formato das solicitações HTTP POST que criam novos recursos é semelhante ao das solicitações PUT; o corpo da mensagem contém os detalhes do novo recurso a ser adicionado. No entanto, o URI normalmente especifica a coleção à qual o recurso deve ser adicionado. O exemplo a seguir cria um novo pedido e adiciona-o à coleção de pedidos:

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
productID=5&quantity=15&orderValue=400
```

Se a solicitação for bem-sucedida, o servidor Web deve responder com um código de mensagem com o código de status HTTP 201 (Criado). O cabeçalho Location deve conter o URI do recurso recém-criado, e o corpo da resposta deve conter uma cópia do novo recurso; o cabeçalho Content-Type especifica o formato desses dados:

```HTTP
HTTP/1.1 201 Created
...
Content-Type: application/json; charset=utf-8
Location: http://adventure-works.com/orders/99
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":99,"productID":5,"quantity":15,"orderValue":400}
```

> [!TIP]
> Se os dados fornecidos por uma solicitação PUT ou POST forem inválidos, o servidor Web deve responder com uma mensagem com código de status HTTP 400 (Solicitação Incorreta). O corpo da mensagem pode conter informações adicionais sobre o problema com a solicitação e os formatos esperados, ou pode conter um link para uma URL que fornece mais detalhes.
>
>

Para remover um recurso, uma solicitação HTTP DELETE simplesmente fornece o URI do recurso a ser excluído. O exemplo a seguir tenta remover o pedido 99:

```HTTP
DELETE http://adventure-works.com/orders/99 HTTP/1.1
...
```

Se a operação de exclusão for bem-sucedida, o servidor Web deve responder com o código de status HTTP 204, indicando que o processo ocorreu com êxito mas o corpo da resposta não contém nenhuma informação adicional (trata-se da mesma resposta retornada por uma operação PUT bem-sucedida, porém sem um cabeçalho Location, já que o recurso não existe mais). Também é possível que uma solicitação de exclusão retorne o código de status HTTP 200 (OK) ou 202 (Aceito) se a exclusão for executada de modo assíncrono.

```HTTP
HTTP/1.1 204 No Content
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

Se o recurso não for encontrado, o servidor Web deverá retornar,  em vez disso, uma mensagem 404 (Não Encontrado).

> [!TIP]
> Se todos os recursos em uma coleção precisarem ser excluídos, permita que uma solicitação HTTP DELETE seja especificada para o URI da coleção, em vez de forçar um aplicativo a remover um recurso da coleção de cada vez.
>
>

### <a name="filtering-and-paginating-data"></a>Filtragem e paginação de dados
Você deve se esforçar para manter os URIs simples e intuitivos. Expor uma coleção de recursos por meio de um único URI ajuda nesse sentido, mas pode levar a aplicativos que buscam grandes quantidades de dados, quando apenas um subconjunto dessas informações é necessário. Gerar um grande volume de tráfego afeta não apenas o desempenho e a escalabilidade do servidor Web, mas também a capacidade de resposta dos aplicativos cliente que solicitam os dados.

Por exemplo, se pedidos contêm o preço pago pelo pedido, um aplicativo cliente que precisa recuperar todos os pedidos que têm um custo por um valor específico talvez precise recuperar todos os pedidos do URI */orders* e, em seguida, filtrar esses pedidos localmente. Fica claro que esse processo é muito ineficiente; desperdiça energia tanto de processamento quanto de largura de banda de rede no servidor que hospeda a API da Web.

Uma solução pode ser fornecer um esquema de URI, como */orders/ordervalue_greater_than_n*, onde *n* é o preço do pedido; porém, para todos os preços exceto um número limitado, essa abordagem não é prática. Além disso, se você precisar consultar pedidos com base em outros critérios, você pode terminar sendo obrigado a fornecer uma longa lista de URIs com nomes possivelmente não intuitivos.

Uma estratégia melhor para filtragem de dados é fornecer os critérios de filtro na cadeia de consulta que é passada para a API da Web, como */pedidos?ordervaluethreshold=n*. Neste exemplo, a operação correspondente na API da Web é responsável pela análise e processamento do parâmetro `ordervaluethreshold` na cadeia de consulta e por retornar os resultados filtrados na resposta HTTP.

Algumas solicitações HTTP GET simples sobre os recursos de coleção teriam potencial de retornar um grande número de itens. Para combater a possibilidade dessa ocorrência, você deve projetar a API da Web para limitar a quantidade de dados retornados por qualquer solicitação única. Você pode obter isso oferecendo suporte a cadeias de consulta que permitam ao usuário especificar o número máximo de itens a serem recuperados (que poderia, por sua vez, estar sujeito a um limite máximo para ajudar a evitar ataques de Negação de Serviço) e um deslocamento inicial na coleção. Por exemplo, a cadeia de consulta no URI */orders?limit=25&offset=50* deve recuperar 25 pedidos, começando a partir do 50º pedido encontrado na coleção pedidos. Assim como ocorre com a filtragem de dados, a operação que implementa a solicitação GET na API da Web é responsável pela análise e processamento dos parâmetros `limit` e `offset` na cadeia de consulta. Para ajudar aplicativos cliente, solicitações GET que retornam dados paginados também devem incluir algum tipo de metadados que indiquem o número total de recursos disponíveis na coleção. Você também pode considerar outras estratégias de paginação inteligente; para obter mais informações, consulte [Observações Sobre Design de API: Paginação Inteligente](http://bizcoder.com/api-design-notes-smart-paging)

Você pode seguir uma estratégia semelhante para classificar os dados conforme eles são encontrados; você pode fornecer um parâmetro de classificação que use um nome de campo como o valor, tal como */orders?sort=ProductID*. No entanto, observe que essa abordagem pode ter um efeito deletério no cache (parâmetros de cadeia de consulta fazem parte do identificador de recurso usado por muitas implementações de cache como a chave para dados armazenados em cache).

Você pode estender esse método para limitar (projetar) os campos retornados se um único item de recurso contém uma grande quantidade de dados. Por exemplo, você poderia usar um parâmetro de cadeia de consulta que aceita uma lista de campos delimitada por vírgulas, como */orders?fields=ProductID,Quantity*.

> [!TIP]
> Forneça padrões significativos a todos os parâmetros opcionais nas cadeias de consulta. Por exemplo, defina o parâmetro `limit` como 10 e o parâmetro `offset` como 0 se você implementar a paginação, defina o parâmetro sort para a chave do recurso se você implementar ordenação e, por fim, defina o parâmetro `fields` para todos os campos no recurso se você oferecer suporte a projeções.
>
>

### <a name="handling-large-binary-resources"></a>Manipulação de recursos binários grandes
Um único recurso pode conter campos binários grandes, como arquivos ou imagens. Para superar os problemas de transmissão causados por conexões intermitentes e não confiáveis e para melhorar os tempos de resposta, considere a possibilidade de fornecer operações que habilitem a recuperação desses recursos em partes pelo aplicativo cliente. Para fazer isso, a API da Web deve oferecer suporte ao cabeçalho Accept-Ranges para solicitações GET para grandes recursos e, idealmente, implementar solicitações HTTP HEAD para esses recursos. O cabeçalho Accept-Ranges indica que a operação GET oferece suporte a resultados parciais, e que um aplicativo cliente pode enviar solicitações GET que retornam um subconjunto de um recurso especificado como um intervalo de bytes. Uma solicitação HEAD é semelhante a uma solicitação GET, porém retorna apenas um cabeçalho que descreve o recurso e um corpo de mensagem vazio. Um aplicativo cliente pode emitir uma solicitação HEAD para determinar se deve ou não buscar um recurso pelo uso de solicitações GET parciais. O exemplo a seguir mostra uma solicitação HEAD que obtém informações sobre uma imagem do produto:

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
...
```

A mensagem de resposta contém um cabeçalho que inclui o tamanho do recurso (4.580 bytes) e o cabeçalho Accept-Ranges, que indica que a operação GET correspondente oferece suporte a resultados parciais:

```HTTP
HTTP/1.1 200 OK
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
...
```

O aplicativo cliente pode usar essas informações para construir uma série de operações GET para recuperar a imagem em partes menores. A primeira solicitação busca os primeiros 2.500 bytes usando o cabeçalho Range:

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
...
```

A mensagem de resposta indica, ao retornar o código de status HTTP 206, que esta é uma resposta parcial. O cabeçalho Content-Length especifica o número real de bytes retornados no corpo da mensagem (não o tamanho do recurso), enquanto o cabeçalho Content-Range indica que parte do recurso isso é (0-2.499 bytes de 4.580):

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580
...
_{binary data not shown}_
```

Uma solicitação subsequente do aplicativo cliente pode recuperar o restante do recurso usando um cabeçalho Range apropriado:

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=2500-
...
```

A mensagem de resultado correspondente deve ter esta aparência:

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2080
Content-Range: bytes 2500-4580/4580
...
```

## <a name="using-the-hateoas-approach-to-enable-navigation-to-related-resources"></a>Usando a abordagem HATEOAS para habilitar a navegação para recursos relacionados
Uma das principais motivações por trás de REST é a que deve ser possível navegar por todo o conjunto de recursos sem exigir conhecimento prévio do esquema de URI. Cada solicitação HTTP GET deve retornar as informações necessárias para localizar os recursos relacionados diretamente ao objeto solicitado por hiperlinks incluídos na resposta, e também deve ser provida de informações descrevendo as operações disponíveis em cada um desses recursos. Esse conceito é conhecido como HATEOAS, ou Hipertexto como o Mecanismo de Estado do Aplicativo. O sistema é efetivamente uma máquina de estado finito, e a resposta para cada solicitação contém as informações necessárias para ir de um estado para outro; nenhuma outra informação deve ser necessária.

> [!NOTE]
> Atualmente não há padrões nem especificações que definam como  modelar o princípio HATEOAS. Os exemplos mostrados nesta seção ilustram uma possível solução.
>
>

Por exemplo, para lidar com o relacionamento entre clientes e pedidos, os dados retornados na resposta para um pedido específico devem conter URIs na forma de um hiperlink que identifica o cliente que fez o pedido e as operações que podem ser executadas nesse cliente.

```HTTP
GET http://adventure-works.com/orders/3 HTTP/1.1
Accept: application/json
...
```

O corpo da mensagem de resposta contém uma matriz `links` (realçada no código de exemplo) que especifica a natureza da relação (*Customer*), o URI do cliente (*http://adventure-works.com/customers/3*), como obter os detalhes desse cliente (*GET*), e os tipos MIME para os quais há suporte no servidor Web usados para recuperar essas informações (*text/xml* e *application/json*). Essas são todas as informações de que um aplicativo cliente precisa para ser capaz de obter os detalhes do cliente. Além disso, a matriz Links também inclui links para as operações que podem ser executadas, como PUT (para modificar o cliente, junto com o formato que o servidor Web espera que o cliente forneça) e DELETE.

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[(some links omitted){"rel":"customer","href":" http://adventure-works.com/customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":"
customer","href":" http://adventure-works.com /customers/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"customer","href":" http://adventure-works.com /customers/3","action":"DELETE","types":[]}]}
```

Para ser completa, a matriz de Links também deve incluir informações de autorreferência pertencentes ao recurso que foi recuperado. Esses links foram omitidos do exemplo anterior, mas são destacados no código a seguir. Observe que nesses links, a relação *self* foi usada para indicar que se trata de uma referência ao recurso que está sendo retornado pela operação:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[{"rel":"self","href":" http://adventure-works.com/orders/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" self","href":" http://adventure-works.com /orders/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"self","href":" http://adventure-works.com /orders/3", "action":"DELETE","types":[]},{"rel":"customer",
"href":" http://adventure-works.com /customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" customer" (customer links omitted)}]}
```

Para que essa abordagem seja eficaz, aplicativos cliente devem estar preparados para recuperar e analisar essas informações adicionais.

## <a name="versioning-a-restful-web-api"></a>Controlando a versão de uma API da Web RESTful
É muito improvável em todas as situações, exceto as mais simples, que uma API da Web permanecerá estática. Conforme os requisitos de negócios mudam, novas coleções de recursos podem ser adicionados, as relações entre os recursos podem mudar e a estrutura dos dados nos recursos pode ser alterada. Enquanto a atualização de uma API da Web para lidar com requisitos novos ou diferentes é um processo relativamente simples, você deve considerar os efeitos que essas mudanças terão em aplicativos cliente que consomem a API da Web. O problema é que, embora o desenvolvedor que projeta e implementa uma API da Web tenha controle total sobre essa API, o desenvolvedor não tem o mesmo grau de controle sobre os aplicativos cliente, que podem ser criados por organizações terceirizadas operando remotamente. A principal prioridade é habilitar aplicativos cliente existentes a continuar funcionando inalterados, permitindo simultaneamente que novos aplicativos cliente tirem proveito das novas funções e recursos.

O controle de versão permite que uma API da Web indique os recursos e as funções que ela expõe, e um aplicativo cliente pode enviar solicitações direcionadas a uma versão específica de uma função ou recurso. As seções a seguir descrevem várias abordagens diferentes, cada qual com suas próprias vantagens e desvantagens.

### <a name="no-versioning"></a>Sem controle de versão
Essa é a abordagem mais simples e pode ser aceitável para algumas APIs internas. Grandes alterações poderiam ser representadas como novos recursos ou novos links.  Adicionar conteúdo aos recursos existentes não deve representar uma alteração significativa, já que aplicativos cliente que não esperavam ver esse conteúdo vão simplesmente ignorá-lo.

Por exemplo, uma solicitação para o URI *http://adventure-works.com/customers/3* deve retornar os detalhes de um único cliente contendo os campos `id`, `name` e `address` esperados pelo aplicativo cliente:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> Para simplicidade e maior clareza, os exemplos de resposta mostrados nesta seção não incluem links HATEOAS.
>
>

Se o campo `DateCreated` é adicionado ao esquema do recurso de cliente, a resposta terá essa aparência:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

Aplicativos cliente existentes podem continuar funcionando corretamente se forem capazes de ignorar campos não reconhecidos, enquanto os novos aplicativos cliente podem ser projetados para lidar com esse novo campo. No entanto, se ocorrerem alterações mais radicais ao esquema de recursos (por exemplo, remover ou renomear campos) ou se as relações entre os recursos mudarem, estas poderão constituir alterações significativas que impedem que aplicativos cliente existentes funcionem corretamente. Nessas situações, você deve considerar uma das abordagens a seguir.

### <a name="uri-versioning"></a>Controle de versão de URI
Cada vez que você modifica a API da Web ou altera o esquema de recursos, você adiciona um número de versão ao URI para cada recurso. Os URIs previamente existentes devem continuar a funcionar como antes, recuperando recursos que estão em conformidade com seu esquema original.

Estendendo o exemplo anterior, se o campo `address` é reestruturado em subcampos contendo cada parte constituinte do endereço (como `streetAddress`, `city`, `state` e `zipCode`), esta versão do recurso pode ser exposta por meio de um URI contendo um número de versão, como http://adventure-works.com/v2/customers/3:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Esse mecanismo de controle de versão é muito simples, mas depende do servidor realizar o roteamento da solicitação para o ponto de extremidade apropriado. No entanto, ele pode se tornar inviável conforme a API da Web amadurece ao passar por várias iterações e o servidor tem que oferecer suporte a um número de versões diferentes. Além disso, de um ponto de vista purista, em todos os casos os aplicativos cliente estão buscando os mesmos dados (cliente 3), portanto o URI não deve ser diferente, independentemente de qual for a versão. Esse esquema também complica a implementação de HATEOAS, já que todos os links precisarão incluir o número da versão em seus URIs.

### <a name="query-string-versioning"></a>Controle de versão de cadeia de consulta
Em vez de fornecer vários URIs, você pode especificar a versão do recurso usando um parâmetro de cadeia de consulta acrescentada à solicitação HTTP, como *http://adventure-works.com/customers/3?version=2*. O parâmetro de versão, caso seja omitido por aplicativos cliente mais antigos, deve passar a usar um valor padrão significativo, como 1.

Essa abordagem tem a vantagem de semântica que o mesmo recurso é sempre recuperado do mesmo URI, mas para isso, é necessário que o código que processa a solicitação analise a cadeia de consulta e envie de volta a resposta HTTP apropriada. Essa abordagem também tem as mesmas complicações para implementar HATEOAS como o mecanismo de controle de versão do URI.

> [!NOTE]
> Alguns navegadores e proxies da Web mais antigos não armazenarão em cache respostas para solicitações que incluam, na URL, uma cadeia de consulta. Isso pode ter um impacto negativo no desempenho de aplicativos Web que usam uma API da Web e que são executados de um navegador da Web desse tipo.
>
>

### <a name="header-versioning"></a>Controle de versão de cabeçalho
Em vez de acrescentar o número de versão como um parâmetro de cadeia de consulta, você pode implementar um cabeçalho personalizado que indica a versão do recurso. Essa abordagem requer que o aplicativo cliente adicione o cabeçalho apropriado a quaisquer solicitações, embora o código que processa a solicitação do cliente possa usar um valor padrão (versão 1) se o cabeçalho da versão for omitido. Os exemplos a seguir utilizam um cabeçalho personalizado chamado *Custom-Header*. O valor desse cabeçalho indica a versão da API da Web.

Versão 1:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=1
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Versão 2:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=2
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Observe que assim como nas duas abordagens anteriores, implementar HATEOAS requer a inclusão do cabeçalho personalizado apropriado em quaisquer eventuais links.

### <a name="media-type-versioning"></a>Controle de versão do tipo de mídia
Quando um aplicativo cliente envia uma solicitação HTTP GET para um servidor Web, ele deve estipular o formato do conteúdo que pode manipular usando um cabeçalho Accept, conforme descrito anteriormente nestas diretrizes. Frequentemente, a finalidade do cabeçalho *Accept* é permitir que o aplicativo cliente especifique se o corpo da resposta deve ser XML, JSON ou algum outro formato comum que o cliente possa analisar. No entanto, é possível definir tipos de mídia personalizados que incluem informações, permitindo que o aplicativo cliente indique qual versão de um recurso esse aplicativo está esperando. O exemplo a seguir mostra uma solicitação que especifica um cabeçalho *Accept* com o valor *application/vnd.adventure-works.v1+json*. O elemento *vnd.adventure-works.v1* indica ao servidor Web que ele deve retornar a versão 1 do recurso, enquanto o elemento *json* especifica que o formato do corpo da resposta deve ser JSON:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Accept: application/vnd.adventure-works.v1+json
...
```

O código que processa a solicitação é responsável por processar o cabeçalho *Accept* e honrá-lo o máximo possível (o aplicativo cliente pode especificar vários formatos no cabeçalho *Accept*; nesse caso, o servidor Web pode escolher o formato mais apropriado para o corpo de resposta). O servidor Web confirma o formato dos dados no corpo da resposta usando o cabeçalho Content-Type:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Se o cabeçalho Accept não especificar nenhum tipo de mídia conhecido, o servidor Web pode gerar uma mensagem de resposta HTTP 406 (Não Aceitável) ou retornar uma mensagem com um tipo de mídia padrão.

Essa abordagem é possivelmente o mais puro dos mecanismos de controle de versão e aplica-se naturalmente a HATEOAS, que pode incluir o tipo MIME dos dados relacionados em links de recursos.

> [!NOTE]
> Quando você seleciona uma estratégia de controle de versão, você também deve considerar as implicações de desempenho, especialmente no armazenamento em cache no servidor Web. O controle de versão do URI e esquemas de controle de versão de cadeia de consulta facilitam o armazenamento em cache na medida em que, a cada vez, a mesma combinação de URI/cadeia de consulta refere-se aos mesmos dados.
>
> Os mecanismos de controle de versão do cabeçalho e do tipo de mídia, normalmente, exigem lógica adicional para examinar os valores no cabeçalho personalizado ou no cabeçalho Accept. Em um ambiente de grande escala, muitos clientes usando versões diferentes de uma API da Web podem resultar em uma quantidade significativa de dados duplicados em um cache do lado do servidor. Esse problema pode se tornar importante se um aplicativo cliente se comunica com um servidor Web através de um proxy que implementa caching, e que encaminha uma solicitação ao servidor Web somente se ele não mantém atualmente uma cópia dos dados solicitados em seu cache.
>
>

## <a name="open-api-initiative"></a>Abrir Iniciativa de API
A [Open API Initiative](https://www.openapis.org/) foi criada por um consórcio do setor para padronizar descrições de API REST entre fornecedores. Como parte dessa iniciativa, a especificação de Swagger 2.0 foi renomeada como OAS (OpenAPI Specification) e colocada sob a iniciativa de API aberta.

Talvez você queira adotar a OpenAPI para suas APIs Web. Considere o seguinte:

- A OpenAPI Specification vem com um conjunto de diretrizes teimosas sobre como uma API REST deve ser criada. Isso apresenta vantagens para a interoperabilidade, mas requer mais cuidado durante o design de sua API para ficar de acordo com a especificação.
- A OpenAPI promove uma abordagem de primeiro contrato, em vez de uma primeira abordagem de implementação. Primeiro contato significa que você projeta o contrato da API (a interface) primeiro e então escreve o código que implementa o contrato. 
- Ferramentas como o Swagger podem gerar bibliotecas de cliente ou a documentação de contratos de API. Por exemplo, veja [Páginas da Ajuda da API Web ASP.NET usando o Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).

## <a name="more-information"></a>Mais informações
* As [Diretrizes da API REST da Microsoft](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md) contêm recomendações detalhadas para a criação de APIs REST públicas.
* O [Guia RESTful](http://restcookbook.com/) contém uma introdução à criação de APIs RESTful.
* A [Lista de Verificação de API Web](https://mathieu.fenniak.net/the-api-checklist/) contém uma lista útil de itens a serem considerados ao projetar e implementar uma API Web.
* O site [Open API Initiative](https://www.openapis.org/) contém toda a documentação relacionada e os detalhes da implementação da Open API.
