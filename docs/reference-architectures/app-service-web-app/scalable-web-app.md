---
title: Aplicativo Web escalonável
titleSuffix: Azure Reference Architectures
description: Melhorar a escalabilidade em um aplicativo Web em execução no Microsoft Azure.
author: MikeWasson
ms.date: 10/25/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: b015a130a74f3160c0e737b436d41e1b1ea7b5bf
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241367"
---
# <a name="improve-scalability-in-an-azure-web-application"></a>Melhorar a escalabilidade em um aplicativo Web

Essa arquitetura de referência mostra práticas comprovadas para melhorar a escalabilidade e o desempenho em um aplicativo Web do Serviço de Aplicativo do Azure.

![Aplicativo Web no Azure com escalabilidade melhorada](./images/scalable-web-app.png)

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura

Essa arquitetura baseia-se naquela mostrada em [Aplicativo Web básico][basic-web-app]. Ela inclui os seguintes componentes:

- **Grupo de recursos**. Um [grupo de recursos][resource-group] é um contêiner lógico para os recursos do Azure.
- **[Aplicativo Web][app-service-web-app]**. Um aplicativo moderno típico pode incluir um site e uma ou mais APIs Web RESTful. Uma API Web pode ser consumida pelos clientes de navegador por meio de AJAX, por aplicativos cliente nativos ou por aplicativos do lado do servidor. Para ver as considerações sobre o design de APIs Web, consulte [Diretrizes de design de API][api-guidance].
- **Aplicativo de funções**. Use [Aplicativos de funções][functions] para executar tarefas em segundo plano. As funções são invocadas por um gatilho, como um evento de temporizador ou uma mensagem sendo colocada na fila. Para tarefas com estado de longa execução, use [Funções Duráveis][durable-functions].
- **Fila**. Na arquitetura mostrada aqui, o aplicativo coloca tarefas na fila em segundo plano, posicionando uma mensagem em uma fila do [Armazenamento de Filas do Azure][queue-storage]. A mensagem dispara um aplicativo de funções. Outra opção é usar filas do Barramento de Serviço. Para comparação, consulte [Filas do Azure e filas do Barramento de Serviço – comparações e contrastes][queues-compared].
- **Cache**. Armazenar dados semi-estáticos no [Cache Redis do Azure][azure-redis].
- **CDN**. Use a CDN ([Rede de Distribuição de Conteúdo do Microsoft Azure][azure-cdn]) para armazenar em cache o conteúdo disponível publicamente para latência mais baixa e distribuição mais rápida do conteúdo.
- **Armazenamento de dados**. Use o [Banco de Dados SQL do Azure][sql-db] para dados relacionais. Para dados não relacionais, considere o [Cosmos DB][cosmosdb].
- **Azure Search**. Use o [Azure Search][azure-search] para adicionar funcionalidade de pesquisa como sugestões de pesquisa, pesquisa difusa e pesquisa específica a um idioma. O Azure Search normalmente é usado junto com outro armazenamento de dados, especialmente se o armazenamento de dados primário exigir consistência estrita. Nessa abordagem, armazene dados autoritativos em outro armazenamento de dados e o índice de pesquisa no Azure Search. O Azure Search também pode ser usado para consolidar um único índice de pesquisa de vários armazenamentos de dados.
- **DNS do Azure**. [DNS do Azure][azure-dns] é um serviço de hospedagem para domínios DNS, que fornece resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure.
- **Gateway de aplicativo**. O [Gateway de Aplicativo](/azure/application-gateway/) é um balanceador de carga da camada 7. Nessa arquitetura, ele roteia as solicitações HTTP para o front-end da web. Gateway de Aplicativo também fornece um [firewall do aplicativo Web](/azure/application-gateway/waf-overview) (WAF) que protege o aplicativo contra vulnerabilidades e explorações comuns.

## <a name="recommendations"></a>Recomendações

Seus requisitos podem ser diferentes dos requisitos da arquitetura descrita aqui. Use as recomendações nesta seção como um ponto de partida.

### <a name="app-service-apps"></a>Aplicativos do Serviço de Aplicativo

É recomendável criar o aplicativo Web e a API web como aplicativos separados do Serviço de Aplicativo. Esse design permite que você o execute em planos de Serviço de Aplicativo separados, portanto eles podem ser dimensionados de forma independente. Se você não precisar desse nível de escalabilidade inicialmente, poderá implantar os aplicativos no mesmo plano e movê-los para planos separados posteriormente, se necessário.

> [!NOTE]
> Para os planos Básico, Standard e Premium, você será cobrado pelas instâncias de VM no plano, não por aplicativo. Consulte [Preço do Serviço de Aplicativo][app-service-pricing]
>

### <a name="cache"></a>Cache

Você pode melhorar o desempenho e a escalabilidade usando o [Cache Redis do Azure][azure-redis] para alguns dados armazenados em cache. Considere usar o Cache Redis para:

- Dados da transação semi-estáticos.
- Estado de sessão.
- Saída HTML. Isso pode ser útil em aplicativos que renderizam saídas HTML complexas.

Para ver diretrizes mais detalhadas sobre como projetar uma estratégia de cache, consulte [Diretrizes de armazenamento em cache][caching-guidance].

### <a name="cdn"></a>CDN

Use a [CDN do Azure][azure-cdn] para armazenar em cache o conteúdo estático. A principal vantagem de uma CDN é reduzir a latência para os usuários, porque o conteúdo é armazenado em cache em um servidor de borda geograficamente próximo do usuário. A CDN também pode reduzir a carga no aplicativo, porque esse tráfego não está sendo tratado pelo aplicativo.

Se seu aplicativo é composto principalmente por páginas estáticas, considere usar [CDN para armazenar em cache o aplicativo inteiro][cdn-app-service]. Caso contrário, coloque o conteúdo estático, como imagens, CSS e arquivos HTML, no [Armazenamento do Azure e use a CDN para armazenar em cache esses arquivos][cdn-storage-account].

> [!NOTE]
> A CDN do Azure não pode fornecer conteúdo que exige autenticação.
>

Para ver diretrizes mais detalhadas, consulte [Diretriz de CDN (Rede de Distribuição de Conteúdo)][cdn-guidance].

### <a name="storage"></a>Armazenamento

Aplicativos modernos geralmente processam grandes volumes de dados. Para dimensionar para a nuvem, é importante escolher o tipo de armazenamento correto. Essas são algumas recomendações de linha de base.

| O que você deseja armazenar | Exemplo | Armazenamento recomendado |
| --- | --- | --- |
| Arquivos |Imagens, documentos, PDFs |Armazenamento do Blobs do Azure |
| Pares de chave/valor |Dados de perfil do usuário pesquisado por ID de usuário |Armazenamento da tabela do Azure |
| Mensagens curtas destinadas a disparar processamento adicional |Solicitações de pedidos |Armazenamento de Filas do Azure, fila do Barramento de Serviço ou tópico do Barramento de Serviço |
| Dados não relacionais com um esquema flexível que exigem consulta básica |Catálogo de produtos |Banco de dados de documentos, como o Azure Cosmos DB, MongoDB ou Apache CouchDB |
| Dados relacionais que exigem suporte mais avançado a consultas, esquema estrito ou consistência forte |Estoque de produtos |Banco de Dados SQL do Azure |

Consulte [Escolher o armazenamento de dados correto][datastore].

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Um grande benefício do Serviço de Aplicativo do Azure é a capacidade de dimensionar o aplicativo com base na carga. Aqui estão algumas considerações para ter em mente ao planejar dimensionar seu aplicativo.

### <a name="app-service-app"></a>Aplicativo de Serviço de Aplicativo

Se sua solução inclui vários aplicativos do Serviço de Aplicativo, considere implantá-los em planos de Serviço de Aplicativo separados. Essa abordagem permite redimensioná-los independentemente por serem executados em instâncias separadas.

De modo semelhante, considere colocar um aplicativo de funções em seu próprio plano para que tarefas em segundo plano não sejam executadas nas mesmas instâncias que manipulam solicitações HTTP. Se as tarefas em segundo plano forem executadas de maneira intermitente, considere o uso de um [plano de consumo][functions-consumption-plan], que é cobrado com base na quantidade de execuções e não por hora.

### <a name="sql-database"></a>Banco de dados SQL

Aumente a escalabilidade de um banco de dados SQL *fragmentando* o banco de dados. Fragmentação refere-se ao particionamento horizontal do banco de dados. A fragmentação permite aumentar o banco de dados usando as [ferramentas de Banco de Dados Elástico][sql-elastic]. As possíveis vantagens da fragmentação são:

- Melhor taxa de transferência de transações.
- Consultas podem ser executadas mais rapidamente em um subconjunto dos dados.

### <a name="azure-search"></a>Azure Search

O Azure Search remove a sobrecarga de execução de pesquisas de dados complexas de armazenamento de dados primário, podendo ser dimensionado para lidar com a carga. Consulte [Dimensionar os níveis de recursos para cargas de trabalho de consulta e indexação no Azure Search][azure-search-scaling].

## <a name="security-considerations"></a>Considerações de segurança

Esta seção lista as considerações sobre segurança específicas dos serviços do Azure descritos neste artigo. Esta não é uma lista completa de práticas recomendadas de segurança. Para obter algumas considerações de segurança adicionais, consulte [Proteger um aplicativo no Serviço de Aplicativo do Azure][app-service-security].

### <a name="cross-origin-resource-sharing-cors"></a>CORS (Compartilhamento de Recursos entre Origens)

Se você criar um site e a API Web como aplicativos separados, o site não poderá fazer chamadas AJAX no lado do cliente para a API, a menos que você habilite o CORS.

> [!NOTE]
> A segurança do navegador impede que uma página da Web envie solicitações do AJAX para outro domínio. Essa restrição se chama política da mesma origem e impede que um site mal-intencional leia dados confidenciais de outro site. O CORS é um padrão W3C que possibilita a um servidor relaxar a política de mesma origem e permitir algumas solicitações entre origens enquanto rejeita outras.
>

Os Serviços de Aplicativos têm suporte interno para o CORS, sem precisar escrever nenhum código de aplicativo. Consulte [Consumir um aplicativo de API do JavaScript usando CORS][cors]. Adicione o site à lista de origens permitidas para API.

### <a name="sql-database-encryption"></a>Criptografia do Banco de Dados SQL

Use [Transparent Data Encryption][sql-encryption] se você precisa criptografar dados em repouso no banco de dados. Esse recurso executa criptografia em tempo real e descriptografia de um banco de dados inteiro (incluindo backups e arquivos de log de transações) e não requer alterações no aplicativo. A criptografia causa certa latência, portanto, é uma prática recomendada separar os dados que devem ser protegidos em seu próprio banco de dados e habilitar a criptografia apenas para ele.

<!-- links -->

[api-guidance]: ../../best-practices/api-design.md
[app-service-security]: /azure/app-service-web/web-sites-security
[app-service-web-app]: /azure/app-service-web/app-service-web-overview
[app-service-api-app]: /azure/app-service-api/app-service-api-apps-why-best-platform
[app-service-pricing]: https://azure.microsoft.com/pricing/details/app-service/
[azure-cdn]: https://azure.microsoft.com/services/cdn/
[azure-dns]: /azure/dns/dns-overview
[azure-redis]: https://azure.microsoft.com/services/cache/
[azure-search]: /azure/search
[azure-search-scaling]: /azure/search/search-capacity-planning
[basic-web-app]: basic-web-app.md
[basic-web-app-scalability]: basic-web-app.md#scalability-considerations
[caching-guidance]: ../../best-practices/caching.md
[cdn-app-service]: /azure/app-service-web/cdn-websites-with-cdn
[cdn-storage-account]: /azure/cdn/cdn-create-a-storage-account-with-cdn
[cdn-guidance]: ../../best-practices/cdn.md
[cors]: /azure/app-service-api/app-service-api-cors-consume-javascript
[cosmosdb]: /azure/cosmos-db/
[datastore]: ../..//guide/technology-choices/data-store-overview.md
[durable-functions]: /azure/azure-functions/durable-functions-overview
[functions]: /azure/azure-functions/functions-overview
[functions-consumption-plan]: /azure/azure-functions/functions-scale#consumption-plan
[queue-storage]: /azure/storage/storage-dotnet-how-to-use-queues
[queues-compared]: /azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted
[resource-group]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[sql-db]: /azure/sql-database/
[sql-elastic]: /azure/sql-database/sql-database-elastic-scale-introduction
[sql-encryption]: https://msdn.microsoft.com/library/dn948096.aspx
[tm]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
[web-app-multi-region]: ./multi-region.md
