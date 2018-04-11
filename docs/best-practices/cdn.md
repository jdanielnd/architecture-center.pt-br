---
title: Diretrizes da Rede de Distribuição de Conteúdo
description: Diretriz na Rede de Distribuição de Conteúdo (CDN) para fornecer conteúdo de alta largura de banda hospedado no Azure.
author: dragon119
ms.date: 02/02/2018
pnp.series.title: Best Practices
ms.openlocfilehash: 42b73db08ecef858f5279ea292cf8c0df77b847c
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/27/2018
---
# <a name="best-practices-for-using-content-delivery-networks-cdns"></a>Melhores práticas para uso das CDNs (redes de distribuição de conteúdo)

Uma CDN (rede de distribuição de conteúdo) é uma rede distribuída de servidores que pode fornecer conteúdo da Web para os usuários com eficiência. As CDNs armazenam conteúdo armazenado em cache em servidores de borda que estão próximos aos usuários finais, a fim de minimizar a latência. 

As CDNs normalmente são usadas para fornecimento de conteúdo estático, como imagens, folhas de estilo, documentos, scripts do lado do cliente e páginas HTML. As principais vantagens de usar uma CDN são latência mais baixa e fornecimento mais rápido de conteúdo aos usuários, independentemente de sua localização geográfica em relação ao datacenter no qual o aplicativo está hospedado. As CDNs também podem ajudar a reduzir a carga em um aplicativo Web, porque o aplicativo não precisa atender a solicitações para o conteúdo que está hospedado na CDN.
 
![Diagrama da CDN](./images/cdn/CDN.png)

No Azure, a [Rede de Distribuição de Conteúdo do Azure](/azure/cdn/cdn-overview) é uma solução global de CDN de fornecimento de conteúdo de alta largura de banda hospedada no Azure ou em qualquer outro local. Usando a CDN do Azure, você pode armazenar em cache objetos disponíveis publicamente carregados de um armazenamento de blobs do Azure, um aplicativo Web, uma máquina virtual e qualquer servidor Web acessível publicamente. 

Este tópico descreve algumas das melhores práticas gerais e considerações ao usar uma CDN. Para saber mais sobre como usar a CDN do Azure, consulte [Documentação da CDN](/azure/cdn/).

## <a name="how-and-why-a-cdn-is-used"></a>Como e por que a CDN é usada

Entre os usos comuns para a CDN estão:  

* Fornecer recursos estáticos para aplicativos cliente, geralmente por meio de um site. Esses recursos podem ser imagens, folhas de estilo, documentos, arquivos, scripts do lado do cliente, páginas HTML, fragmentos HTML ou qualquer outro conteúdo que o servidor não precise modificar para cada solicitação. O aplicativo pode criar itens em tempo de execução e disponibilizá-los para o CDN (por exemplo, ao criar uma lista atual de manchetes de notícias), mas ele não faz isso para cada solicitação.
* Fornecimento de conteúdo estático e compartilhado público para dispositivos como telefones móveis e tablet PCs. O aplicativo em si é um serviço web que oferece uma API aos clientes que são executados em vários dispositivos. O CDN também pode fornecer conjuntos de dados estáticos (por meio do serviço web) para os clientes usarem, talvez para gerar a interface do usuário do cliente. Por exemplo, o CDN pode ser usado para distribuir documentos XML ou JSON.
* Servir sites inteiros que consistem em apenas conteúdo estático público para os clientes, sem a necessidade de quaisquer recursos de computação dedicados.
* Fazer o streaming de arquivos de vídeo ao cliente sob demanda. Os benefícios do vídeo de baixa latência e conectividade confiável disponível em datacenters globalmente localizados que oferecem conexões do CDN. Os Serviços de Mídia do Microsoft Azure (AMS) integram-se com o Azure CDN para entregar conteúdo diretamente ao CDN para melhor distribuição. Para saber mais, confira [Streaming endpoints overview](/azure/media-services/media-services-streaming-endpoints-overview) (Visão geral dos pontos de extremidade de streaming).
* Geralmente aprimorar a experiência dos usuários, especialmente aqueles localizados longe o datacenter que hospeda o aplicativo. Esses usuários poderiam, de outra forma, sofrer latência mais alta. Uma grande proporção do tamanho total do conteúdo em um aplicativo Web geralmente é estática e usar o CDN pode ajudar a manter o desempenho e a experiência geral do usuário eliminando a necessidade de implantar o aplicativo em vários datacenters. Para ver uma lista de locais de nó da CDN do Azure, consulte [Locais POP da CDN do Azure](/azure/cdn/cdn-pop-locations/).
* Dando suporte a soluções de IoT (Internet das Coisas). O grande número de dispositivos e aplicativos envolvidos em uma solução de IoT pode sobrecarregar um aplicativo com facilidade, se ele precisasse distribuir atualizações de firmware diretamente para cada dispositivo.
* Lidar com picos e sobretensão de energia sob demanda sem exigir que o aplicativo dimensione, evitando o consequente aumento dos custos de execução. Por exemplo, quando uma atualização para um sistema operacional é liberada, para um dispositivo de hardware, como um modelo específico do roteador ou para um dispositivo do consumidor, como uma smart TV, haverá um grande pico na demanda conforme ele é baixado por milhões de usuários e dispositivos em um curto período de tempo.

## <a name="challenges"></a>Desafios

Há vários desafios a serem considerados ao planejar o uso de uma CDN.  

* **Implantação**. Decida a origem da qual a CDN busca o conteúdo e se é necessário implantar o conteúdo em mais de um sistema de armazenamento. Leve em consideração o processo de implantação de recursos e conteúdo estático. Por exemplo, talvez seja necessário implementar uma etapa separada para carregar conteúdo no armazenamento de blobs do Azure.
* **Controle de versão e controle de cache**. Leve em consideração como você atualizará o conteúdo estático e implantará novas versões. Entenda como a CDN executa o cache e a TTL (vida útil). Para a CDN do Azure, consulte [Como funciona o cache](/azure/cdn/cdn-how-caching-works).
* **Testando**. Pode ser difícil executar teste local de suas configurações de CDN ao desenvolver e testar um aplicativo localmente ou em um ambiente de preparo.
* **SEO (otimização do mecanismo de pesquisa)**. Conteúdo como imagens e documentos são atendidos de um domínio diferente quando você usa CDN. Isso pode afetar a SEO para esse conteúdo.
* **Segurança de conteúdo**. Nem todas as CDNs oferecem uma forma de controle de acesso para o conteúdo. Alguns serviços da CDN, incluindo a CDN do Azure, dão suporte à autenticação baseada em token para proteger o conteúdo da CDN. Para obter mais informações, consulte [Protegendo ativos da Rede de Distribuição de Conteúdo do Azure com autenticação de token](/azure/cdn/cdn-token-auth).
* **Segurança do cliente**. Os clientes podem se conectar por meio de um ambiente que não permita acesso a recursos no CDN. Isso pode ser um ambiente restrito de segurança que limita o acesso a apenas um conjunto de fontes conhecidas ou que impede o carregamento de recursos por meio  de qualquer coisa que não seja a origem da página. Uma implementação de fallback é necessária para lidar com esses casos.
* **Resiliência**. O CDN é um ponto único de falha em potencial de um aplicativo. 

Os cenários onde o CDN pode ser menos útil incluem:  

* Se o conteúdo tiver uma baixa taxa de ocorrências, ele pode ser acessado apenas algumas vezes enquanto for válido (determinado pela sua configuração de tempo de vida). 
* Se os dados forem particulares, como para empresas de grande porte ou ecossistemas de cadeia de suprimentos.

## <a name="general-guidelines-and-good-practices"></a>Diretrizes gerais e práticas recomendadas

O uso de uma CDN é uma boa maneira de minimizar a carga no aplicativo e maximizar a disponibilidade e o desempenho. Leve em consideração a adoção dessa estratégia para todos os recursos e conteúdos apropriados que o seu aplicativo usa. Considere os pontos nas seções a seguir ao projetar sua estratégia para usar uma CDN.

### <a name="deployment"></a>Implantação
O conteúdo estático pode precisar ser provisionado e implantado independentemente do aplicativo se você não incluí-lo no pacote ou processo de implantação do aplicativo. Considere como isso afetará a abordagem do controle de versão usada para gerenciar os componentes do aplicativo e o conteúdo de recursos estáticos.

Considere o uso de técnicas de agrupamento e minificação para reduzir os tempos de carregamento para clientes. O agrupamento combina vários arquivos em um único arquivo. A minificação remove caracteres desnecessários de scripts e arquivos CSS sem alterar a funcionalidade.

Se você precisar implantar o conteúdo em um local adicional, isso será uma etapa extra no processo de implantação. Se o aplicativo atualizar o conteúdo para o CDN, talvez em intervalos regulares ou em resposta a um evento, ele deve armazenar o conteúdo atualizado em todos os locais adicionais, bem como o ponto de extremidade para o CDN.

Considere como você manipulará o desenvolvimento e teste local quando um conteúdo estático precisar ser fornecido por meio de uma CDN. Por exemplo, você pode pré-implantar o conteúdo na CDN como parte do script de build. Como alternativa, use as diretivas de compilação ou sinalizadores para controlar como o aplicativo carrega os recursos. Por exemplo, no modo de depuração, o aplicativo pôde carregar recursos estáticos de uma pasta local. No modo de versão, o aplicativo usará a CDN.

Considere as opções de compactação de arquivo, como gzip (GNU zip). A compactação pode ser executada no servidor de origem pelo aplicativo Web host ou diretamente nos servidores de borda pela CDN. Para obter mais informações, consulte [Melhorar o desempenho compactando arquivos na CDN do Azure](/azure/cdn/cdn-improve-performance).


### <a name="routing-and-versioning"></a>Roteamento e controle de versão
Você talvez precise usar diferentes instâncias do CDN em vários momentos. Por exemplo, ao implantar uma nova versão do aplicativo, você pode querer usar um novo CDN e reter o CDN antigo (mantendo o conteúdo em um formato antigo) para versões anteriores. Se você usar o armazenamento de blob do Azure como a origem de conteúdo, pode criar uma conta de armazenamento separada ou um contêiner separado e direcionar o ponto de extremidade da CDN a ele. 

Não use a cadeia de caracteres de consulta para denotar versões diferentes do aplicativo nos links para recursos na CDN, pois, ao recuperar o conteúdo do armazenamento de blobs do Azure, a cadeia de caracteres de consulta faz parte do nome do recurso (o nome de blob). Essa abordagem também pode afetar como o cliente armazena os recursos em cache.

Implantar novas versões do conteúdo estático quando você atualiza um aplicativo pode ser um desafio se os recursos anteriores forem armazenados em cache no CDN. Para obter mais informações, consulte a seção sobre controle de cache abaixo.

Considere restringir o acesso ao conteúdo CDN por país. O CDN do Azure permite filtrar solicitações com base no país de origem e restringir o conteúdo fornecido. Para saber mais, veja [Restringir o acesso ao seu conteúdo por país](/azure/cdn/cdn-restrict-access-by-country/).

### <a name="cache-control"></a>controle de cache
Considere como gerenciar o caching dentro do sistema. Por exemplo, na CDN do Azure, você pode definir regras de cache global e, em seguida, definir um cache personalizado para pontos de extremidade de origem específicos. Você também pode controlar como o cache é executado em uma CDN pelo envio de cabeçalhos de diretiva de cache na origem. 

Para obter mais informações, consulte [Como funciona o cache](/azure/cdn/cdn-how-caching-works).

Para impedir que objetos estejam disponíveis na CDN, exclua-os da origem remova ou exclua o ponto de extremidade da CDN ou, no caso do armazenamento de blobs, torne o contêiner ou blob particular. No entanto, os itens não são removidos até que a vida útil expire. Também limpe manualmente um ponto de extremidade da CDN.

### <a name="security"></a>Segurança

A CDN pode entregar conteúdo pelo HTTPS (SSL) usando o certificado fornecido pela CDN, e também pelo HTTP padrão. Para evitar avisos de navegador sobre o conteúdo misto, talvez seja necessário usar HTTPS para solicitar o conteúdo estático que é exibido nas páginas carregadas por meio de HTTPS.

Se você entregar ativos estáticos (como arquivos de fonte) usando a CDN, talvez encontre problemas de política de mesma origem se usar uma chamada *XMLHttpRequest* para solicitar esses recursos de um domínio diferente. Muitos navegadores da web impedem o compartilhamento de recursos de origens cruzadas (CORS), a menos que o servidor Web esteja configurado para definir os cabeçalhos de resposta apropriada. É possível configurar a CDN para dar suporte ao CORS usando um dos seguintes métodos:

* Configure a CDN para adicionar cabeçalhos CORS às respostas. Para obter mais informações, confira [Usando a CDN do Azure com o CORS](/azure/cdn/cdn-cors). 
* Se a origem é o armazenamento de blobs do Azure, adicione regras do CORS ao ponto de extremidade de armazenamento. Para obter mais informações, consulte [Suporte de Compartilhamento de Recursos de Origens Cruzadas (CORS) para os serviços de armazenamento do Azure](http://msdn.microsoft.com/library/azure/dn535601.aspx).
* Configure o aplicativo para definir os cabeçalhos CORS. Por exemplo, confira [Habilitando as Solicitações Entre Origens (CORS)](/aspnet/core/security/cors) na documentação do ASP.NET Core.

### <a name="cdn-fallback"></a>Fallback de CDN
Leve em consideração como seu aplicativo lidará com uma falha ou indisponibilidade temporária da CDN. Os aplicativos cliente poderão usar cópias dos recursos que foram armazenados em cache localmente (no cliente) durante solicitações anteriores, ou eles podem incluir o código que detecta falhas e solicita, em vez disso, os recursos de origem (a pasta de aplicativo ou o contêiner de blob do Azure que contém os recursos) se o CDN não estiver disponível.
