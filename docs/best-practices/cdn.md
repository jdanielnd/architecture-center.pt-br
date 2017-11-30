---
title: "Diretrizes da rede de distribuição de conteúdo"
description: "Diretriz na Rede de Distribuição de Conteúdo (CDN) para fornecer conteúdo de alta largura de banda hospedado no Azure."
author: dragon119
ms.date: 09/30/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 94036c803552d5e7061f99e6dd0ca9e563a32690
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="content-delivery-network"></a>Rede de Distribuição de Conteúdo
[!INCLUDE [header](../_includes/header.md)]

A CDN (Rede de Distribuição de Conteúdo) do Microsoft Azure oferece aos desenvolvedores uma solução global de fornecimento de conteúdo de alta largura de banda hospedada no Azure ou em qualquer outro local. Com a CDN, você pode armazenar em cache objetos publicamente disponíveis carregados do Armazenamento de Blobs do Azure, de um aplicativo Web, da máquina virtual, de uma pasta de aplicativo ou de outro local HTTP/HTTPS. O cache da CDN pode ser mantido em locais estratégicos para fornecer a máxima largura de banda para fornecimento de conteúdo aos usuários. A CDN normalmente é usada para fornecimento de conteúdo estático, como imagens, folhas de estilo, documentos, arquivos, scripts do lado do cliente e páginas HTML.

Você também pode usar a CDN como um cache para servir conteúdo dinâmico, como um relatório em PDF ou um gráfico com base em entradas especificadas. Se os mesmos valores de entrada forem fornecidos por usuários diferentes, o resultado deverá ser o mesmo.

As principais vantagens de usar a CDN são latência mais baixa e fornecimento mais rápido de conteúdo aos usuários independentemente da localização geográfica em relação ao datacenter em que o aplicativo está hospedado.  

![Diagrama da CDN](./images/cdn/CDN.png)

Usar a CDN também deve ajudar a reduzir a carga no aplicativo, pois ele é liberado do processamento necessário para acessar e entregar o conteúdo. Essa redução de carga pode ajudar a aumentar o desempenho e a escalabilidade do aplicativo, bem como minimizar os custos de hospedagem, reduzindo os recursos de processamento necessários para alcançar um nível específico de desempenho e disponibilidade.

## <a name="how-and-why-a-cdn-is-used"></a>Como e por que a CDN é usada
Entre os usos comuns para a CDN estão:  

* Fornecer recursos estáticos para aplicativos cliente, geralmente por meio de um site. Esses recursos podem ser imagens, folhas de estilo, documentos, arquivos, scripts do lado do cliente, páginas HTML, fragmentos HTML ou qualquer outro conteúdo que o servidor não precise modificar para cada solicitação. O aplicativo pode criar itens em tempo de execução e disponibilizá-los para o CDN (por exemplo, ao criar uma lista atual de manchetes de notícias), mas ele não faz isso para cada solicitação.
* Fornecimento de conteúdo estático e compartilhado público para dispositivos como telefones móveis e tablet PCs. O aplicativo em si é um serviço web que oferece uma API aos clientes que são executados em vários dispositivos. O CDN também pode fornecer conjuntos de dados estáticos (por meio do serviço web) para os clientes usarem, talvez para gerar a interface do usuário do cliente. Por exemplo, o CDN pode ser usado para distribuir documentos XML ou JSON.
* Servir sites inteiros que consistem em apenas conteúdo estático público para os clientes, sem a necessidade de quaisquer recursos de computação dedicados.
* Fazer o streaming de arquivos de vídeo ao cliente sob demanda. Os benefícios do vídeo de baixa latência e conectividade confiável disponível em datacenters globalmente localizados que oferecem conexões do CDN. Os Serviços de Mídia do Microsoft Azure (AMS) integram-se com o Azure CDN para entregar conteúdo diretamente ao CDN para melhor distribuição. Para saber mais, confira [Streaming endpoints overview](/azure/media-services/media-services-streaming-endpoints-overview) (Visão geral dos pontos de extremidade de streaming).
* Geralmente aprimorar a experiência dos usuários, especialmente aqueles localizados longe o datacenter que hospeda o aplicativo. Esses usuários poderiam, de outra forma, sofrer latência mais alta. Uma grande proporção do tamanho total do conteúdo em um aplicativo Web geralmente é estática e usar o CDN pode ajudar a manter o desempenho e a experiência geral do usuário eliminando a necessidade de implantar o aplicativo em vários datacenters.
* Manipular a carga crescente em aplicativos que dão suporte a soluções de IoT (Internet das coisas). O grande número desses dispositivos e aparelhos envolvidos poderia facilmente sobrecarregar o aplicativo se fosse necessário processar mensagens de difusão e gerenciar a distribuição de atualização de firmware diretamente para cada dispositivo.
* Lidar com picos e sobretensão de energia sob demanda sem exigir que o aplicativo dimensione, evitando o consequente aumento dos custos de execução. Por exemplo, quando uma atualização para um sistema operacional é liberada, para um dispositivo de hardware, como um modelo específico do roteador ou para um dispositivo do consumidor, como uma smart TV, haverá um grande pico na demanda conforme ele é baixado por milhões de usuários e dispositivos em um curto período de tempo.

A lista a seguir mostra exemplos do tempo médio para primeiro byte de vários locais geográficos. A função web de destino é implantada ao Azure no Oeste dos EUA. Há uma forte correlação entre o maior aumento devido ao CDN e a proximidade com um nó CDN. Para obter uma lista completa com as localizações dos nós do CDN disponíveis, veja [Locais de Nó da Rede de Distribuição de Conteúdo (CDN) do Azure](/azure/cdn/cdn-pop-locations/).

|  | Tempo (ms) até o Primeiro Byte (Origem) | Tempo (ms) até o Primeiro (CDN) | % de aprimoramento de tempo de CDN |
| --- | --- | --- | --- |
| \*São José, CA |47,5 |46,5 |2% |
| \*\*Dulles, VA |109 |40,5 |169% |
| Buenos Aires, AR |210 |151 |39% |
| \*Londres, Reino Unido |195 |44 |343% |
| Xangai, CN |242 |206 |17% |
| \*Cingapura |214 |74 |189% |
| \*Tóquio, JP |163 |48 |204% |
| Seul, Coreia do Sul |190 |190 |0% |

\* Tem um nó do CDN do Azure na mesma cidade.  
\*\* Tem um nó do CDN do Azure em uma cidade vizinha.  

## <a name="challenges"></a>Desafios
Há vários desafios a serem considerados ao planejar usar o CDN:  

* **Implantação**. Decida a origem da qual o CDN busca o conteúdo e se é necessário implantar o conteúdo em mais de um sistema de armazenamento (como no CDN e em um local alternativo).

  Seu mecanismo de implantação de aplicativo deve considerar o processo para a implantação de conteúdo e recursos estáticos, bem como implantar os arquivos do aplicativo, como páginas ASPX. Por exemplo, talvez seja necessário implementar uma etapa separada para carregar conteúdo no armazenamento de blobs do Azure.
* **Controle de versão e controle de cache**. Leve em consideração como você atualizará o conteúdo estático e implantará novas versões. O conteúdo da CDN pode ser [limpo](/azure/cdn/cdn-purge-endpoint/) usando o Portal do Azure quando houver novas versões de seus ativos disponíveis. Esse é um desafio semelhante a gerenciar armazenamento em cache do lado do cliente, como aquele que ocorre em um navegador da Web.
* **Testando**. Pode ser difícil executar teste local de suas configurações de CDN ao desenvolver e testar um aplicativo localmente ou em um ambiente de preparo.
* **SEO (otimização do mecanismo de pesquisa)**. Conteúdo como imagens e documentos são atendidos de um domínio diferente quando você usa CDN. Isso pode afetar a SEO para esse conteúdo.
* **Segurança de conteúdo**. Atualmente, muitos serviços de CDN, como o Azure CDN, não oferecem qualquer tipo de controle de acesso para o conteúdo.
* **Segurança do cliente**. Os clientes podem se conectar por meio de um ambiente que não permita acesso a recursos no CDN. Isso pode ser um ambiente restrito de segurança que limita o acesso a apenas um conjunto de fontes conhecidas ou que impede o carregamento de recursos por meio  de qualquer coisa que não seja a origem da página. Uma implementação de fallback é necessária para lidar com esses casos.
* **Resiliência**. O CDN é um ponto único de falha em potencial de um aplicativo. Ele tem uma menor disponibilidade de SLA que o armazenamento de blob (que pode ser usado para entregar conteúdo diretamente), então, talvez seja necessário considerar a implementação de um mecanismo de fallback para conteúdo essencial.

  Você pode monitorar a disponibilidade do conteúdo da CDN, a largura de banda, os dados transferidos, as ocorrências, a taxa de ocorrências no cache e as métricas de cache no Portal do Azure em [tempo real](/azure/cdn/cdn-real-time-stats/) e em [relatórios agregados](/azure/cdn/cdn-analyze-usage-patterns/).

Os cenários onde o CDN pode ser menos útil incluem:  

* Se o conteúdo tiver uma baixa taxa de ocorrências, ele pode ser acessado apenas algumas vezes enquanto for válido (determinado pela sua configuração de tempo de vida). Na primeira vez que um item é baixado, você incorre em duas taxas de transação (da origem para o CDN e então do CDN para o cliente).
* Se os dados forem particulares, como para empresas de grande porte ou ecossistemas de cadeia de suprimentos.

## <a name="general-guidelines-and-good-practices"></a>Diretrizes gerais e práticas recomendadas
Usar o CDN é uma boa maneira de reduzir a carga no seu aplicativo e maximizar a disponibilidade e o desempenho. Leve em consideração a adoção dessa estratégia para todos os recursos e conteúdos apropriados que o seu aplicativo usa. Considere os pontos nas seções a seguir ao projetar a sua estratégia para usar o CDN:  

### <a name="origin"></a>Origem
Para implantar o conteúdo por meio da CDN, basta especificar um ponto de extremidade HTTP e/ou HTTPS que o serviço CDN usará para acessar e armazenar o conteúdo em cache.

O ponto de extremidade pode especificar um contêiner de armazenamento de blobs do Azure com o conteúdo estático que você deseja enviar por meio da CDN. O contêiner deve ser marcado como público. Somente os blobs em um contêiner público que tem acesso público de leitura estão disponíveis por meio da CDN.

O ponto de extremidade pode especificar uma pasta chamada **cdn** na raiz de uma das camadas de computação do aplicativo (por exemplo, uma função web ou uma máquina virtual). Os resultados das solicitações de recursos, incluindo recursos dinâmicos, como páginas ASPX, serão armazenados em cache no CDN. O período mínimo de cache é de 300 segundos. Qualquer período mais curto impede o conteúdo de ser implantado na CDN (para obter mais informações, confira o título *Controle de cache* a seguir).

Se você estiver usando os Aplicativos Web do Azure, o ponto de extremidade será definido como a pasta raiz do site por meio da seleção do site durante a criação da instância da CDN. Todo o conteúdo do site estará disponível por meio do CDN.

Na maioria dos casos, apontar o seu ponto de extremidade da CDN em uma pasta dentro de uma das camadas de computação do seu aplicativo oferece mais flexibilidade e controle. Por exemplo, ele facilita o gerenciamento das necessidades atuais e futuras de roteamento e gera dinamicamente o conteúdo estático como miniaturas de imagem.

Você pode usar as [cadeias de caracteres de consulta](/azure/cdn/cdn-query-string/) para diferenciar objetos no cache quando o conteúdo for entregue por meio de origens dinâmicas, como páginas ASPX. No entanto, esse comportamento pode ser desabilitado por uma configuração no Portal do Azure quando você especifica o ponto de extremidade da CDN. Ao fornecer conteúdo do armazenamento de blob, as cadeias de caracteres de consulta são tratadas como literais de cadeia de caracteres para que dois itens que têm o mesmo nome, mas cadeias de caracteres de consulta diferentes, sejam armazenados como itens separados na CDN.

Você pode utilizar a regravação de URL para recursos como scripts e outros conteúdos que evitem mover os arquivos para a pasta de origem da CDN.

Ao usar os blobs de armazenamento do Azure para manter o conteúdo para a CDN, a URL dos recursos em blobs diferencia maiúsculas de minúsculas para o nome do contêiner e do blob.

Ao usar origens personalizadas ou Aplicativos Web do Azure, você especifica o caminho para a instância da CDN nos links para recursos. Por exemplo, a seguir está especificado um arquivo de imagem na pasta **Imagens** do site que será entregue por meio do CDN:

```XML
<img src="http://[your-cdn-endpoint].azureedge.net/Images/image.jpg" />
```

### <a name="deployment"></a>Implantação
O conteúdo estático pode precisar ser provisionado e implantado independentemente do aplicativo se você não incluí-lo no pacote ou processo de implantação do aplicativo. Considere como isso afetará a abordagem do controle de versão usada para gerenciar os componentes do aplicativo e o conteúdo de recursos estáticos.

Considere como agrupamento (combinar vários arquivos em um arquivo) e minificação (remover caracteres desnecessários, como espaço em branco, caracteres de nova linha, comentários e outros caracteres) para arquivos de script e CSS serão tratados. Essas são técnicas normalmente usadas que podem reduzir tempos de carregamento para clientes e são compatíveis com o fornecimento de conteúdo por meio do CDN. Para obter mais informações, consulte [Agrupamento e minificação](http://www.asp.net/mvc/tutorials/mvc-4/bundling-and-minification).

Se você precisar implantar o conteúdo em um local adicional, isso será uma etapa extra no processo de implantação. Se o aplicativo atualizar o conteúdo para o CDN, talvez em intervalos regulares ou em resposta a um evento, ele deve armazenar o conteúdo atualizado em todos os locais adicionais, bem como o ponto de extremidade para o CDN.

Você não pode configurar um ponto de extremidade da CDN para um aplicativo no emulador do Azure local no Visual Studio. Essa restrição afetará o teste de unidade, os testes funcionais e os testes de pré-implantação final. Você deve permitir isso implementando um mecanismo alternativo. Por exemplo, você pode implantar previamente o conteúdo ao CDN usando um utilitário ou um aplicativo personalizado e realizar testes durante o período em que ele é armazenado em cache. Como alternativa, use diretivas de compilação ou constantes globais para controlar de onde o aplicativo carrega os recursos. Por exemplo, quando em execução no modo de depuração, ele poderia carregar recursos como pacotes de script do lado do cliente e outros tipos de conteúdo de uma pasta local e usar o CDN ao ser executado no modo de versão.

Considere para qual abordagem de compactação você deseja que o CDN tenha suporte:

* Você pode [habilitar a compactação](/azure/cdn/cdn-improve-performance/) no servidor de origem e, nesse caso, o CDN oferecerá suporte à compactação por padrão e entregará conteúdo compactado aos clientes em um formato como zip ou gzip. Ao usar uma pasta de aplicativo como o ponto de extremidade do CDN, o servidor pode compactar algum conteúdo de modo automático da mesma maneira que ao fornecê-lo diretamente em um navegador da Web ou outro tipo de cliente. O formato depende do valor do cabeçalho **Accept-Encoding** na solicitação enviada pelo cliente. No Azure, o mecanismo padrão é compactar automaticamente o conteúdo quando a utilização da CPU for inferior a 50%. Se você estiver usando um serviço de nuvem para hospedar o aplicativo, alterar as configurações pode exigir o uso de uma tarefa de inicialização para ativar a compactação de saída dinâmica no IIS. Consulte [Habilitar a compactação gzip com o Microsoft Azure CDN por meio de uma função web](http://blogs.msdn.com/b/avkashchauhan/archive/2012/03/05/enableing-gzip-compression-with-windows-azure-cdn-through-web-role.aspx) para obter mais informações.
* Você pode habilitar a compactação diretamente nos servidores de borda da CDN e, nesse caso, a CDN compactará os arquivos e os servirá para os usuários finais. Para saber mais, veja [Compactação do CDN do Azure](/azure/cdn/cdn-improve-performance/).

### <a name="routing-and-versioning"></a>Roteamento e controle de versão
Você talvez precise usar diferentes instâncias do CDN em vários momentos. Por exemplo, ao implantar uma nova versão do aplicativo, você pode querer usar um novo CDN e reter o CDN antigo (mantendo o conteúdo em um formato antigo) para versões anteriores. Se você usar o armazenamento de blob do Azure como a origem de conteúdo, pode criar uma conta de armazenamento separada ou um contêiner separado e direcionar o ponto de extremidade da CDN a ele. Se você usar a pasta raiz cdn dentro do aplicativo como o ponto de extremidade da CDN, é possível usar técnicas de regravação de URL para direcionar solicitações para uma pasta diferente.

Não use a cadeia de caracteres de consulta para denotar versões diferentes do aplicativo nos links para recursos na CDN, pois, ao recuperar o conteúdo do armazenamento de blobs do Azure, a cadeia de caracteres de consulta faz parte do nome do recurso (o nome de blob). Essa abordagem também pode afetar como o cliente armazena os recursos em cache.

Implantar novas versões do conteúdo estático quando você atualiza um aplicativo pode ser um desafio se os recursos anteriores forem armazenados em cache no CDN. Para saber mais, veja a seção *controle de cache*.

Considere restringir o acesso ao conteúdo CDN por país. O CDN do Azure permite filtrar solicitações com base no país de origem e restringir o conteúdo fornecido. Para saber mais, veja [Restringir o acesso ao seu conteúdo por país](/azure/cdn/cdn-restrict-access-by-country/).

### <a name="cache-control"></a>controle de cache
Considere como gerenciar o caching dentro do sistema. Por exemplo, ao usar uma pasta como a origem do CDN, você pode especificar a capacidade de cache de páginas que geram o conteúdo e o momento de expiração do conteúdo para todos os recursos em uma pasta específica. Você também pode especificar propriedades de cache para o CDN e para o cliente usando os cabeçalhos HTTP padrão. Embora você já deva estar gerenciando armazenamento em cache no servidor e no cliente, usar o CDN o ajudará a se tornar mais consciente de como o conteúdo é armazenado em cache e onde isso é feito.

Para impedir que os objetos estejam disponíveis no CDN, você pode excluí-los da origem (pasta raiz *cdn* do aplicativo ou contêiner de blob), remover ou excluir o ponto de extremidade do CDN, ou, no caso do armazenamento de blobs, tornar o contêiner ou blob particular. No entanto, os itens serão removidos do CDN somente quando sua vida útil expirar. Se nenhum período de expiração do cache for especificado (por exemplo, quando o conteúdo for carregado do armazenamento de blobs), ele é armazenado em cache na CDN por até sete dias.  Você pode também [limpar manualmente um ponto de extremidade da CDN](/azure/cdn/cdn-purge-endpoint/).

Em um aplicativo Web, você pode definir o caching e a expiração de todo o conteúdo usando o elemento *clientCache* na seção *system.webServer/staticContent* de um arquivo web.config. Lembre-se de que, ao colocar um arquivo web.config em uma pasta, ele afetará os arquivos naquela pasta e em todas as subpastas.

Se você criar o conteúdo para o CDN dinamicamente (no código do aplicativo, por exemplo), verifique se você especificou a propriedade *cache. SetExpires* em cada página. O CDN não armazenará em cache a saída de páginas que usam a configuração de capacidade de cache *pública*padrão.  Defina o período de expiração do cache com um valor apropriado para garantir que o conteúdo não seja descartado e recarregado do aplicativo em intervalos muito curtos.  

### <a name="security"></a>Segurança
A CDN pode entregar conteúdo pelo HTTPS (SSL) usando o certificado fornecido pela CDN, e também pelo HTTP padrão. Para evitar avisos de navegador sobre o conteúdo misto, talvez seja necessário usar HTTPS para solicitar o conteúdo estático que é exibido nas páginas carregadas por meio de HTTPS.

Se você entregar ativos estáticos (como arquivos de fonte) usando a CDN, talvez encontre problemas de política de mesma origem se usar uma chamada *XMLHttpRequest* para solicitar esses recursos de um domínio diferente. Muitos navegadores da web impedem o compartilhamento de recursos de origens cruzadas (CORS), a menos que o servidor Web esteja configurado para definir os cabeçalhos de resposta apropriada. É possível configurar a CDN para dar suporte ao CORS usando um dos seguintes métodos:

* Use o mecanismo de regras da CDN para adicionar cabeçalhos do CORS para as respostas. Esse método geralmente é o melhor porque há suporte para curingas e para várias origens de permissão específica. Para obter mais informações, confira [Usando a CDN do Azure com o CORS](https://docs.microsoft.com/en-us/azure/cdn/cdn-cors). 
* Adicione uma *CorsRule* para as propriedades do serviço. É possível usar esse método se a origem da qual você está entregando conteúdo for o armazenamento de blobs do Azure. A regra pode especificar as origens permitidas para solicitações de CORS, os métodos permitidos, como GET, e a duração máxima em segundos para a regra (o período em que o cliente deve solicitar os recursos vinculados depois de carregar o conteúdo original). Ao definir CORS no Armazenamento para usar com a CDN, há suporte somente para o curinga ‘*’ para a lista de origens permitidas. Para obter mais informações, consulte [Suporte de Compartilhamento de Recursos de Origens Cruzadas (CORS) para os serviços de armazenamento do Azure](http://msdn.microsoft.com/library/azure/dn535601.aspx).
* Configure regras de saída no arquivo de configuração de aplicativo para definir um cabeçalho *Access-Control-Allow-Origin* em todas as respostas. É possível usar esse método se o servidor de origem estiver executando o IIS. Ao usar esse método com a CDN, há suporte somente para o curinga ‘*’ para a lista de origens permitidas. Para obter mais informações sobre como usar regras de reescrita, consulte [Módulo de Reescrita de URL](http://www.iis.net/learn/extensions/url-rewrite-module).

### <a name="custom-domains"></a>Domínios personalizados
A CDN do Azure permite especificar um [nome de domínio personalizado](/azure/cdn/cdn-map-content-to-custom-domain/) e usá-lo para acessar recursos por meio da CDN. Você também pode configurar um nome de subdomínio personalizado usando um registro *CNAME* no seu DNS. Usar essa abordagem pode fornecer uma camada adicional de abstração e controle.

Se você usar um *CNAME*, não será possível usar SSL, pois o CDN usa o próprio certificado SSL único, e esse certificado não corresponderá aos seus nomes de domínio/subdomínio personalizados.

### <a name="cdn-fallback"></a>Fallback de CDN
Leve em consideração como seu aplicativo lidará com uma falha ou indisponibilidade temporária da CDN. Os aplicativos cliente poderão usar cópias dos recursos que foram armazenados em cache localmente (no cliente) durante solicitações anteriores, ou eles podem incluir o código que detecta falhas e solicita, em vez disso, os recursos de origem (a pasta de aplicativo ou o contêiner de blob do Azure que contém os recursos) se o CDN não estiver disponível.

O exemplo a seguir mostra os mecanismos de fallback usando [Auxiliares de marcação](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro) em um modo de exibição do Razor.

```HTML
...
<link rel="stylesheet" href="https://[your-cdn-endpoint].azureedge.net/lib/bootstrap/dist/css/bootstrap.min.css"
      asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
      asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute"/>
<link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true"/>
...
<script src="https://[your-cdn-endpoint].azureedge.net/lib/jquery/dist/jquery-2.2.0.min.js"
        asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
        asp-fallback-test="window.jQuery">
</script>
<script src="https://[your-cdn-endpoint].azureedge.net/lib/bootstrap/dist/js/bootstrap.min.js"
        asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.min.js"
        asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal">
</script>
...
```

### <a name="search-engine-optimization"></a>Otimização do mecanismo de pesquisa
Se o SEO for uma consideração importante em seu aplicativo, execute as seguintes tarefas:

* Incluir um cabeçalho *Rel* canonical em cada página ou recurso.
* Use um registro de subdomínio *CNAME* e acesse os recursos usando esse nome.
* Considere o impacto do fato de que o endereço IP do CDN pode ser um país ou região diferente daquele do aplicativo em si.
* Ao usar o armazenamento de blob do Azure como a origem, mantenha a mesma estrutura de arquivo para recursos no CDN como nas pastas de aplicativo.

### <a name="monitoring-and-logging"></a>Monitoramento e registro em log
Incluem o CDN como parte da sua estratégia para detectar e medir as falhas de monitoramento de aplicativos ou as ocorrências de latência estendidas.  O monitoramento está disponível no gerenciador de perfis da CDN localizado no site do portal do Azure.

Habilitar o registro em log para o CDN e monitorar esse log como parte das operações diárias.

Considere analisar o tráfego do CDN quanto a padrões de uso. O portal do Azure fornece ferramentas que permitem monitorar:

* Largura de banda,
* Dados Transferidos,
* Ocorrências (códigos de status),
* Status do Cache,
* Taxa de Ocorrências do Cache e
* Taxa de solicitações de IPV4/IPV6.

Para saber mais, confira [Analisar padrões de uso da CDN](/azure/cdn/cdn-analyze-usage-patterns/).

### <a name="cost-implications"></a>Implicações de custo
Você será cobrado por transferências de dados de saída da CDN.  Além disso, se você estiver usando o armazenamento de blobs para hospedar seus ativos, será cobrado pelas transações de armazenamento quando a CDN carregar os dados de seu aplicativo. Defina períodos realistas de expiração de cache para o conteúdo a fim de garantir a atualização, mas não tão curtos a ponto de causar a repetição do carregamento de conteúdo do aplicativo ou do armazenamento de blob para o CDN.

Os itens que raramente são baixados incorrerão em encargos de duas transações sem fornecer qualquer redução significativa na carga do servidor.

### <a name="bundling-and-minification"></a>Agrupamento e minificação
Use agrupamento e minificação para reduzir o tamanho dos recursos, como código JavaScript e páginas HTML armazenadas no CDN. Essa estratégia pode ajudar a reduzir o tempo necessário para baixar esses itens para o cliente.

O agrupamento e a minimização podem ser manipulados pelo ASP.NET. Em um projeto do MVC, você define os seus pacotes em *BundleConfig.cs*. É criada uma referência ao conjunto de script minificado chamando o método *Script.Render* , normalmente em código na classe de exibição. Esta referência contém uma cadeia de caracteres de consulta que inclui um hash, que se baseia no conteúdo do pacote. Se o conteúdo do pacote for alterado, o hash gerado também será alterado.  

Por padrão, as instâncias do Azure CDN têm a configuração *Status de cadeia de caracteres de consulta* desabilitada. Para os pacotes de script atualizado serem tratados corretamente pelo CDN, você deve habilitar a configuração *Status de cadeia de caracteres de consulta* para a instância CDN. Observe que pode levar uma hora ou mais antes de a configuração entrar em vigor.

### <a name="features"></a>Recursos

O Azure tem vários produtos da CDN. Ao selecionar uma CDN, considere os recursos aos quais cada produto dá suporte. Confira [Recursos da CDN do Azure][cdn-features] para obter detalhes. Entre os recursos premium que você talvez queira considerar, temos:

- **[Mecanismo de regras](/azure/cdn/cdn-rules-engine)**. O mecanismo de regras permite personalizar a forma como as solicitações HTTP são manipuladas, como o bloqueio de entrega de alguns tipos de conteúdo, definição de uma política de caching e modificação dos cabeçalhos HTTP. 

- **[Estatísticas em tempo real](/azure/cdn/cdn-real-time-stats)**. Monitora dados em tempo real, como largura de banda, status de cache e conexões simultâneas ao seu perfil da CDN e recebe [alertas em tempo real](/azure/cdn/cdn-real-time-alerts). 


## <a name="rules-engine-url-rewriting-example"></a>Exemplo de regravação de URL do mecanismo de regras

O diagrama a seguir demonstra como executar [Regravação de URL](https://technet.microsoft.com/library/ee215194.aspx) ao usar a CDN. As solicitações do CDN para o conteúdo que está armazenado em cache serão redirecionadas para pastas específicas dentro da raiz do aplicativo com base no tipo de recurso (como scripts e imagens).  

![Diagrama do mecanismo de regras](./images/cdn/rules-engine.png)

Essas regras de regravação executam os redirecionamentos a seguir:

* A primeira regra permite que você insira uma versão no nome do arquivo de um recurso, o que, em seguida, é ignorado. Por exemplo, *Filename_v123.jpg* é regravado como *Filename.jpg*.
* As próximas quatro regras mostram como redirecionar as solicitações se você não desejar armazenar os recursos em uma pasta chamada *cdn** na raiz da função web. As regras mapeiam as URLs *cdn/Images*, *cdn/Content*, *cdn/Scripts* e *cdn/bundles* até suas respectivas pastas raiz na função web.

Observe que usar a regravação de URL exige fazer algumas alterações ao agrupamento de recursos.     

## <a name="more-information"></a>Mais informações
* [CDN do Azure](https://azure.microsoft.com/services/cdn/)
* [Documentação da CDN (rede de distribuição de conteúdo) do Azure](https://azure.microsoft.com/documentation/services/cdn/)
* [Usando o Azure CDN](/azure/cdn/cdn-create-new-endpoint/)
* [Integrando um serviço de nuvem com a CDN do Azure](/azure/cdn/cdn-cloud-service-with-cdn/) (https://azure.microsoft.com/blog/2011/03/18/best-practices-for-the-windows-azure-content-delivery-network/)


<!-- links -->

[cdn-features]: /azure/cdn/cdn-overview#azure-cdn-features
