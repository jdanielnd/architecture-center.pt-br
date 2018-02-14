---
title: "Migrar um aplicativo dos Serviços de Nuvem do Azure para o Azure Service Fabric"
description: "Como migrar um aplicativo dos Serviços de Nuvem do Azure para o Azure Service Fabric."
author: MikeWasson
ms.date: 04/27/2017
ms.openlocfilehash: 73e34c53ffd2f2eeb466d12a5f6c65dcfdaae389
ms.sourcegitcommit: 2c9a8edf3e44360d7c02e626ea8ac3b03fdfadba
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/03/2018
---
# <a name="migrate-an-azure-cloud-services-application-to-azure-service-fabric"></a>Migrar um aplicativo dos Serviços de Nuvem do Azure para o Azure Service Fabric 

[Código de exemplo do ![GitHub](../_images/github.png)][sample-code]

Este artigo descreve a migração de um aplicativo dos Serviços de Nuvem do Azure para o Azure Service Fabric. Ele se concentra nas decisões de arquitetura e práticas recomendadas. 

Para este projeto, começamos com um aplicativo dos Serviços de Nuvem chamado Surveys e movido para o Service Fabric. A meta era migrar o aplicativo com o menor número de alterações possível. Em um artigo posterior, vamos otimizar o aplicativo de Service Fabric adotando uma arquitetura de microsserviços.

Antes de ler este artigo, será útil entender os conceitos básicos de Service Fabric e arquiteturas de microsserviços em geral. Confira os seguintes artigos:

- [Visão geral do Azure Service Fabric][sf-overview]
- [Por que usar uma abordagem de microsserviço para construir aplicativos?][sf-why-microservices]


## <a name="about-the-surveys-application"></a>Sobre o aplicativo Surveys

Em 2012, o grupo de padrões e práticas criou um aplicativo chamado Surveys, para um catálogo chamado [Desenvolvimento de aplicativos multilocatários para a nuvem][tailspin-book]. O catálogo descreve uma empresa fictícia chamada Tailspin que projeta e implementa o aplicativo Surveys.

O Surveys é um aplicativo multilocatário que permite que os clientes criem pesquisas. Depois que um cliente se inscreve para o aplicativo, os membros da organização do cliente podem criar e publicar pesquisas e coletar os resultados para análise. O aplicativo inclui um site público em que as pessoas podem realizar uma pesquisa. Leia mais sobre o cenário original da Tailspin [aqui][tailspin-scenario].

Agora a Tailspin deseja mover o aplicativo de Surveys para uma arquitetura de microsserviços, usando o Service Fabric em execução no Azure. Como o aplicativo já estava implantado como aplicativo de Serviços de Nuvem, a Tailspin adota uma abordagem de várias fases:

1.  Portar os serviços de nuvem para o Service Fabric, enquanto minimiza as alterações do aplicativo.
2.  Otimizar o aplicativo de Service Fabric movendo para uma arquitetura de microsserviços.

Este artigo descreve a primeira fase. Um artigo posterior descreverá a segunda fase. Em um projeto do mundo real, é provável que os dois estágios sobreponham-se. Ao portar para o Service Fabric, você também precisaria começar a refazer a arquitetura do aplicativo em microsserviços. Mais tarde, você pode refinar a arquitetura ainda mais, talvez dividindo os serviços de alta granularidade em serviços menores.  

O código do aplicativo está disponível no [GitHub][sample-code]. Este repositório contém o aplicativo de Serviços de Nuvem e a versão do Service Fabric. 

> O serviço de nuvem é uma versão atualizada do aplicativo original do catálogo *Desenvolvimento de aplicativos multilocatários*.

## <a name="why-microservices"></a>Por que os microsserviços?

Uma discussão detalhada sobre microsserviços está além do escopo deste artigo, mas aqui estão alguns dos benefícios que a Tailspin espera obter movendo para uma arquitetura de microsserviços:

- **Atualizações de aplicativo**. Os serviços podem ser implantados independentemente, portanto você pode adotar uma abordagem incremental para atualizar um aplicativo.
- **Resiliência e isolamento a falhas**. Se um serviço falhar, outros serviços continuam sendo executados.
- **Escalabilidade**. Os serviços podem ser dimensionados de maneira independente.
- **Flexibilidade**. Os serviços são projetados em relação a cenários de negócios, não pilhas de tecnologia, tornando mais fácil migrar os serviços aos novos repositórios de dados, estruturas ou tecnologias.
- **Desenvolvimento Agile**. Serviços individuais têm menos código do que um aplicativo monolítico, tornando o código base mais fácil de entender, raciocinar sobre e testar.
- **Equipes pequenas e focadas**. Porque o aplicativo é dividido em vários serviços pequenos, podendo cada um deles ser criado por uma equipe pequena e focada.

## <a name="why-service-fabric"></a>Por que Service Fabric?
      
O Service Fabric é uma boa opção para uma arquitetura de microsserviços, porque a maioria dos recursos necessários em um sistema distribuído é incorporada ao Service Fabric, incluindo:

- **Gerenciamento de clusters**. O Service Fabric controla automaticamente o failover de nó, o monitoramento de integridade e outras funções de gerenciamento de clusters.
- **Dimensionamento em escala horizontal**. Quando você adicionar nós a um cluster do Service Fabric, o aplicativo dimensionará automaticamente, pois os serviços são distribuídos entre os novos nós.
- **Descoberta de serviço**. O Service Fabric fornece um serviço de descoberta que pode resolver o ponto de extremidade para um serviço nomeado.
- **Serviços com e sem estado**. Os serviços com estado usam [coleções confiáveis][sf-reliable-collections], que podem assumir o lugar de um cache ou fila, e podem ser particionados.
- **Gerenciamento do ciclo de vida de aplicativos**. Os serviços podem ser atualizados independentemente e sem tempo de inatividade do aplicativo.
- **Orquestração de serviço** em um cluster de computadores.
- **Maior densidade** para otimizar o consumo de recursos. Um único nó pode hospedar vários serviços.

O Service Fabric é usado por vários serviços da Microsoft, incluindo o banco de dados SQL do Azure, Cosmos DB, Hubs de Eventos do Azure, entre outros, tornando-o uma plataforma comprovada para criar aplicativos distribuídos de nuvem. 

## <a name="comparing-cloud-services-with-service-fabric"></a>Comparar os Serviços de Nuvem com o Service Fabric

A tabela a seguir resume algumas das diferenças importantes entre os Serviços de Nuvem e aplicativos do Service Fabric. Para uma discussão mais detalhada, consulte [Saiba mais sobre as diferenças entre os Serviços de Nuvem e o Service Fabric antes de migrar os aplicativos][sf-compare-cloud-services].

|        | Serviços de Nuvem | Service Fabric |
|--------|---------------|----------------|
| Composição de aplicativos | Funções| Serviços |
| Densidade |Uma instância de função por VM | Vários serviços em um único nó |
| Número mínimo de nós | 2 por função | 5 por cluster, para implantações de produção |
| Gerenciamento de estado | Sem estado | Sem estado ou com estado* |
| Hosting | As tabelas | Em nuvem ou local |
| Hospedagem na Web | IIS** | Hospedagem interna |
| Modelo de implantação | [Modelo de implantação clássico][azure-deployment-models] | [Gerenciador de Recursos][azure-deployment-models]  |
| Empacotamento | Arquivos de pacote de serviço de nuvem (.cspkg) | Aplicativo e pacotes de serviço |
| Atualização do aplicativo | Troca VIP ou atualização sem interrupção | Atualização sem interrupção |
| Dimensionamento automático | [Serviço interno][cloud-service-autoscale] | Conjuntos de Dimensionamento de VM para dimensionamento automático |
| Depurando | Emulador local | Cluster local |


\*Os serviços com estado usam [coleções confiáveis][sf-reliable-collections] para armazenar o estado entre réplicas, para que todas as leituras sejam locais aos nós no cluster. As gravações são replicadas em nós para confiabilidade. Os serviços sem estado podem ter estado externo, usando um banco de dados ou outro armazenamento externo.

** Funções de trabalho também podem hospedar automaticamente a API Web ASP.NET usando o OWIN.

## <a name="the-surveys-application-on-cloud-services"></a>O aplicativo Surveys nos Serviços de Nuvem

O diagrama a seguir mostra a arquitetura do aplicativo Surveys em execução nos Serviços de Nuvem. 

![](./images/tailspin01.png)

O aplicativo consiste em duas funções web e uma função de trabalho.

- A função web **Tailspin.Web** hospeda um site ASP.NET que clientes da Tailspin usam para criar e gerenciar pesquisas. Os clientes também usam esse site para se inscrever para o aplicativo e gerenciar suas assinaturas. Por fim, os administradores da Tailspin podem usá-lo para ver a lista de locatários e gerenciar dados de locatário. 

- A função web **Tailspin.Web.Survey.Public** hospeda um site ASP.NET no qual as pessoas podem obter as pesquisas que clientes da Tailspin publicam. 

- A função de trabalho **Tailspin.Workers.Survey** realiza processamento em segundo plano. As funções web colocam itens de trabalho em uma fila e, em seguida, a função de trabalho processa os itens. Duas tarefas em segundo plano são definidas: exportar respostas da pesquisa para o banco de dados SQL do Azure e calcular estatísticas referentes às respostas de pesquisa.

Além dos Serviços de Nuvem, o aplicativo de pesquisas usa alguns outros serviços do Azure:

- **Armazenamento do Azure** para armazenar pesquisas, respostas de pesquisas e informações de locatário.

- **Cache Redis do Azure** para armazenar em cache alguns dos dados que são armazenados no Armazenamento do Azure, para acesso de leitura mais rápido. 

- **Azure Active Directory** (Azure AD) para autenticar clientes e administradores da Tailspin.

- **Banco de dados SQL do Azure** para armazenar as respostas da pesquisa para análise. 

## <a name="moving-to-service-fabric"></a>Mover para o Service Fabric

Conforme mencionado, o objetivo dessa fase foi migrar para o Service Fabric com as mínimas alterações necessárias. Para esse fim, criamos serviços sem estado correspondentes a cada função de serviço de nuvem no aplicativo original:

![](./images/tailspin02.png)

Intencionalmente, essa arquitetura é muito semelhante ao aplicativo original. No entanto, o diagrama oculta algumas diferenças importantes. No restante deste artigo, vamos explorar essas diferenças. 


## <a name="converting-the-cloud-service-roles-to-services"></a>Converter as funções de serviço de nuvem para serviços

Conforme mencionado, migramos cada função de serviço de nuvem para um serviço do Service Fabric. Como as funções de serviço de nuvem são sem estado, para essa fase, fazia sentido criar serviços sem estado no Service Fabric. 

Para a migração, seguimos as etapas descritas em [Guia para converter Funções Web e de Trabalho em serviços sem estado do Service Fabric][sf-migration]. 

### <a name="creating-the-web-front-end-services"></a>Criar serviços de front-end da web

No Service Fabric, um serviço é executado dentro de um processo criado pelo tempo de execução do Service Fabric. Para um front-end da web, isso significa que o serviço não está em execução dentro do IIS. Em vez disso, o serviço deve hospedar um servidor web. Essa abordagem é chamada *hospedagem interna*, porque o código que é executado dentro do processo funciona como host do servidor web. 

O requisito para hospedar internamente significa que um serviço do Service Fabric não pode usar o ASP.NET MVC ou Web Forms do ASP.NET, porque essas estruturas requerem o IIS e não são compatíveis com hospedagem interna. As opções para hospedagem interna incluem:

- [ASP.NET Core][aspnet-core], hospedado internamente usando o servidor web [Kestrel][kestrel]. 
- [API Web ASP.NET][aspnet-webapi], hospedada internamente usando [OWIN][owin].
- Estruturas de terceiros, como [Nancy](http://nancyfx.org/).

O aplicativo Surveys original usa ASP.NET MVC. Como o ASP.NET MVC não pode ser hospedado internamente no Service Fabric, consideramos as seguintes opções de migração:

- As funções da web do ASP.NET Core, que podem ser hospedadas internamente.
- Converter o site da web em um aplicativo de página única (SPA) que chama uma API web implementada usando a API Web ASP.NET. Isso exigiria remodelar completamente o front-end da web.
- Manter o código existente do ASP.NET MVC e implantar o IIS em um contêiner do Windows Server para Service Fabric. Essa abordagem requer pouca ou nenhuma alteração de código. No entanto, o [suporte a contêineres][sf-containers] no Service Fabric ainda está na versão prévia.

Com base nessas considerações, selecionamos a primeira opção, portando para ASP.NET Core. Para fazer isso, seguimos as etapas descritas em [Migrar do ASP.NET MVC para ASP.NET Core MVC][aspnet-migration]. 

> [!NOTE]
> Ao usar o ASP.NET Core com Kestrel, você deve colocar um proxy reverso na frente do Kestrel para lidar com o tráfego da Internet, por motivos de segurança. Para obter mais informações, consulte [Implementação do servidor web Kestrel no ASP.NET Core][kestrel]. A seção [Implantar o aplicativo](#deploying-the-application) descreve uma implantação recomendada do Azure.

### <a name="http-listeners"></a>Ouvintes HTTP

Em Serviços de Nuvem, uma função web ou de trabalho expõe um ponto de extremidade HTTP declarando-o no [arquivo de definição de serviço][cloud-service-endpoints]. Uma função web deve ter pelo menos um ponto de extremidade.

```xml
<!-- Cloud service endpoint -->
<Endpoints>
    <InputEndpoint name="HttpIn" protocol="http" port="80" />
</Endpoints>
```

Da mesma forma, os pontos de extremidade do Service Fabric são declarados em um manifesto de serviço: 

```xml
<!-- Service Fabric endpoint -->
<Endpoints>
    <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" Port="8002" />
</Endpoints>
```

Ao contrário de uma função de serviço de nuvem, no entanto, os serviços do Service Fabric podem estar colocalizados dentro do mesmo nó. Portanto, cada serviço deve escutar em uma porta distinta. Neste artigo, discutiremos como solicitações de cliente na porta 80 ou 443 são roteadas para a porta correta para o serviço.

Um serviço deve explicitamente criar ouvintes para cada ponto de extremidade. O motivo disso é que o Service Fabric é independente quanto às pilhas de comunicação. Para obter mais informações, consulte [Compilar um serviço Web front-end para seu aplicativo usando ASP.NET Core][sf-aspnet-core].

## <a name="packaging-and-configuration"></a>Empacotamento e configuração

 Um serviço de nuvem contém os seguintes arquivos de configuração e pacote:

| Arquivo | DESCRIÇÃO |
|------|-------------|
| Definição do serviço (.csdef) | Configurações usadas pelo Azure para configurar o serviço de nuvem. Define as funções, os pontos de extremidade, as tarefas de inicialização e os nomes dos parâmetros de configuração. |
| Configuração do serviço (.cscfg) | Configurações por implantação, incluindo o número de instâncias de função, os números de porta de ponto de extremidade e os valores de parâmetros de configuração. 
| Pacote de serviço (.cspkg) | Contém o código do aplicativo, as configurações e o arquivo de definição de serviço.  |

Existe um arquivo .csdef para todo o aplicativo. Você pode ter vários arquivos. cscfg para ambientes diferentes, como local, teste ou produção. Quando o serviço está em execução, você pode atualizar o .cscfg, mas não o .csdef. Para obter mais informações, consulte [Qual é o modelo de serviço de nuvem e como empacotá-lo?][cloud-service-config]

O Service Fabric tem uma divisão semelhante entre *definição* de serviço e *configurações* de serviço, mas a estrutura é mais granular. Para entender o modelo de configuração do Service Fabric, é importante para entender como um aplicativo do Service Fabric é empacotado. Aqui está a estrutura:

```
Application package
  - Service packages
    - Code package
    - Configuration package
    - Data package (optional)
```

O pacote do aplicativo é o que você implanta. Ele contém um ou mais pacotes de serviço. Um pacote de serviço contém pacotes de código, configuração e dados. O pacote de código contém os binários para os serviços e o pacote de configuração contém as definições de configuração. Esse modelo permite que você atualize os serviços individuais sem reimplantar o aplicativo inteiro. Ele também permite atualizar apenas as definições de configuração, sem reimplantar o código ou reiniciar o serviço.

Um aplicativo do Service Fabric contém os seguintes arquivos de configuração:

| Arquivo | Local padrão | DESCRIÇÃO |
|------|----------|-------------|
| ApplicationManifest.xml | Pacote de aplicativos | Define os serviços que compõem o aplicativo. |
| ServiceManifest.xml | Pacote de serviço| Descreve um ou mais serviços. |
| Settings.xml | Pacote de configuração | Contém configurações para os serviços definidos no pacote de serviço. |

Para saber mais, confira [Modelar um aplicativo no Service Fabric][sf-application-model].

Para dar suporte a configurações diferentes para vários ambientes, use a seguinte abordagem, descrita em [Gerenciar parâmetros do aplicativo para vários ambientes][sf-multiple-environments]:

1. Defina a configuração no arquivo Setting.xml para o serviço.
2. No manifesto do aplicativo, defina uma substituição para a configuração.
3. Coloque configurações específicas do ambiente em arquivos de parâmetro do aplicativo.


## <a name="deploying-the-application"></a>Implantação do aplicativo

Enquanto os Serviços de Nuvem do Azure são um serviço gerenciado, o Service Fabric é um tempo de execução. Você pode criar clusters do Service Fabric em muitos ambientes, incluindo no Azure e no local. Neste artigo, vamos nos concentrar na implantação no Azure. 

O diagrama a seguir mostra uma implantação recomendada:

![](./images/tailspin-cluster.png)

O cluster do Service Fabric é implantado em um [conjunto de dimensionamento de VM][vm-scale-sets]. Os conjuntos de dimensionamento são um recurso de computação do Azure que podem ser usados para implantar e gerenciar um conjunto de VMs idênticas. 

Conforme mencionado, o servidor web Kestrel requer um proxy reverso por motivos de segurança. Este diagrama mostra o [Gateway de Aplicativo do Azure][application-gateway], que é um serviço do Azure que oferece vários recursos de balanceamento de carga de camada 7. Ele atua como um serviço de proxy inverso, encerrando a conexão do cliente e encaminhando solicitações aos pontos de extremidade de back-end. Você pode usar uma solução de proxy reverso diferentes, como nginx.  

### <a name="layer-7-routing"></a>Roteamento de camada 7

No [aplicativo Surveys original](https://msdn.microsoft.com/en-us/library/hh534477.aspx#sec21), uma função web que escuta a porta 80 e a outra função web escuta a porta 443. 

| Site público | Site de gerenciamento de pesquisa |
|-------------|------------------------|
| `http://tailspin.cloudapp.net` | `https://tailspin.cloudapp.net` |

Outra opção é usar o roteamento de camada 7. Nessa abordagem, diferentes caminhos de URL são roteados para números de porta diferentes no back-end. Por exemplo, o site público pode usar caminhos de URL que começam com `/public/`. 

As opções de roteamento de camada 7 incluem:

- Usar o Gateway de Aplicativo. 

- Usar uma solução de virtualização de rede (NVA), como nginx.

- Gravar um gateway personalizado como serviço sem estado.

Considere essa abordagem se você tem dois ou mais serviços com pontos de extremidade HTTP públicos, mas deseja exibi-los como um site com um único nome de domínio.

> Uma abordagem que *não* recomendamos é permitir que clientes externos enviem solicitações por meio do [proxy reverso][sf-reverse-proxy] do Service Fabric. Embora isso seja possível, o proxy reverso destina-se à comunicação entre serviços. Abri-lo para clientes externos expõe *qualquer* serviço em execução no cluster que tenha um ponto de extremidade HTTP.

### <a name="node-types-and-placement-constraints"></a>Restrições de posicionamento e tipos de nó

Na implantação mostrada acima, todos os serviços são executados em todos os nós. No entanto, você também pode agrupar serviços, para que determinados serviços sejam executados somente em nós específicos no cluster. Os motivos para usar essa abordagem incluem:

- Executar alguns serviços em diferentes tipos de VM. Por exemplo, alguns serviços podem precisar de uso intenso de computação ou exigir GPUs. Você pode ter uma mistura de tipos de VM no cluster do Service Fabric.
- Isolar serviços front-end de serviços de back-end, por motivos de segurança. Todos os serviços de front-end serão executado em um conjunto de nós e os serviços de back-end serão executados em nós diferentes no mesmo cluster.
- Diferentes requisitos de escala. Alguns serviços talvez precisem ser executados em mais nós do que outros serviços. Por exemplo, se você definir nós de front-end e nós de back-end, cada conjunto pode ser dimensionado independentemente.

O diagrama a seguir mostra um cluster que separa os serviços de front-end e de back-end:

![](././images/node-placement.png)

Para implementar essa abordagem:

1.  Quando você criar o cluster, defina dois ou mais tipos de nó. 
2.  Para cada serviço, use [restrições de posicionamento][sf-placement-constraints] para atribuir o serviço a um tipo de nó.

Quando você implanta no Azure, cada tipo de nó é implantado em um conjunto separado de dimensionamento de VM. O cluster do Service Fabric abrange todos os tipos de nó. Para obter mais informações, consulte [A relação entre os tipos de nó do Service Fabric e os conjuntos de escala da máquina virtual][sf-node-types].

> Se um cluster tem vários tipos de nó, um tipo de nó é designado como *primário*. Os serviços de tempo de execução do Service Fabric, como o serviço de gerenciamento de cluster, são executados no tipo de nó primário. Provisione pelo menos 5 nós para o tipo de nó primário em um ambiente de produção. O outro tipo de nó deve ter pelo menos 2 nós.

## <a name="configuring-and-managing-the-cluster"></a>Configurando e gerenciando o cluster

Os clusters devem ser protegidos para ajudar a impedir que usuários não autorizados se conectem ao seu cluster. É recomendável usar o Azure AD para autenticar clientes e certificados X.509 quanto à segurança entre nós. Para obter mais informações, consulte [Cenários de segurança do cluster do Service Fabric][sf-security].

Para configurar um ponto de extremidade HTTPS público, consulte [Especificar recursos em um manifesto de serviço][sf-manifest-resources].

Você pode expandir o aplicativo adicionando VMs ao cluster. Os conjuntos de dimensionamento de VM oferecem suporte ao dimensionamento automático usando regras de dimensionamento automático com base nos contadores de desempenho. Para obter mais informações, confira [Como dimensionar um cluster do Service Fabric usando regras de dimensionamento automático][sf-auto-scale].

Enquanto o cluster está em execução, você deve coletar logs de todos os nós em um local central. Para saber mais, veja [Coletar logs usando o Diagnóstico do Azure][sf-logs].   


## <a name="conclusion"></a>Conclusão

O processo de portar o aplicativo Surveys para o Service Fabric foi bastante simples. Para resumir, fizemos o seguinte:

- Convertemos as funções para serviços sem estado.
- Convertemos os front-ends da web para ASP.NET Core.
- Alteramos os arquivos de configuração e empacotamento para o modelo do Service Fabric.

Além disso, a implantação foi alterada de Serviços de Nuvem para cluster do Service Fabric em execução em um conjunto de dimensionamento de VM.

## <a name="next-steps"></a>Próximas etapas

Agora que o aplicativo Surveys foi movido com êxito, a Tailspin quer aproveitar os recursos do Service Fabric, como implantação de serviço independente e controle de versão. Saiba como a Tailspin decompôs esses serviços a uma arquitetura mais granular para aproveitar esses recursos do Service Fabric em [Refatorar um Aplicativo Azure Service Fabric migrado a partir dos Serviços de Nuvem do Azure][refactor-surveys]

<!-- links -->

[application-gateway]: /azure/application-gateway/
[aspnet-core]: /aspnet/core/
[aspnet-webapi]: https://www.asp.net/web-api
[aspnet-migration]: /aspnet/core/migration/mvc
[aspnet-hosting]: /aspnet/core/fundamentals/hosting
[aspnet-webapi]: https://www.asp.net/web-api
[azure-deployment-models]: /azure/azure-resource-manager/resource-manager-deployment-model
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale-portal
[cloud-service-config]: /azure/cloud-services/cloud-services-model-and-package
[cloud-service-endpoints]: /azure/cloud-services/cloud-services-enable-communication-role-instances#worker-roles-vs-web-roles
[kestrel]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel
[lb-probes]: /azure/load-balancer/load-balancer-custom-probe-overview
[owin]: https://www.asp.net/aspnet/overview/owin-and-katana
[refactor-surveys]: refactor-migrated-app.md
[sample-code]: https://github.com/mspnp/cloud-services-to-service-fabric
[sf-application-model]: /azure/service-fabric/service-fabric-application-model
[sf-aspnet-core]: /azure/service-fabric/service-fabric-add-a-web-frontend
[sf-auto-scale]: /azure/service-fabric/service-fabric-cluster-scale-up-down
[sf-compare-cloud-services]: /azure/service-fabric/service-fabric-cloud-services-migration-differences
[sf-connect-and-communicate]: /azure/service-fabric/service-fabric-connect-and-communicate-with-services
[sf-containers]: /azure/service-fabric/service-fabric-containers-overview
[sf-logs]: /azure/service-fabric/service-fabric-diagnostics-how-to-setup-wad
[sf-manifest-resources]: /azure/service-fabric/service-fabric-service-manifest-resources
[sf-migration]: /azure/service-fabric/service-fabric-cloud-services-migration-worker-role-stateless-service
[sf-multiple-environments]: /azure/service-fabric/service-fabric-manage-multiple-environment-app-configuration
[sf-node-types]: /azure/service-fabric/service-fabric-cluster-nodetypes
[sf-overview]: /azure/service-fabric/service-fabric-overview
[sf-placement-constraints]: /azure/service-fabric/service-fabric-cluster-resource-manager-cluster-description
[sf-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[sf-reliable-services]: /azure/service-fabric/service-fabric-reliable-services-introduction
[sf-reverse-proxy]: /azure/service-fabric/service-fabric-reverseproxy
[sf-security]: /azure/service-fabric/service-fabric-cluster-security
[sf-why-microservices]: /azure/service-fabric/service-fabric-overview-microservices
[tailspin-book]: https://msdn.microsoft.com/en-us/library/ff966499.aspx
[tailspin-scenario]: https://msdn.microsoft.com/en-us/library/hh534482.aspx
[unity]: https://msdn.microsoft.com/en-us/library/ff647202.aspx
[vm-scale-sets]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
