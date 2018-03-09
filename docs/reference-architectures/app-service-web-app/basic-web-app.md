---
title: "Aplicativo Web básico"
description: "Arquitetura recomendada para um aplicativo Web básico executado no Microsoft Azure."
author: MikeWasson
ms.date: 12/12/2017
cardTitle: Basic web application
ms.openlocfilehash: 38b0739cc61d679742b610b99e92aaad8d3b394d
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/08/2018
---
# <a name="basic-web-application"></a>Aplicativo Web básico
[!INCLUDE [header](../../_includes/header.md)]

Essa arquitetura de referência mostra um conjunto de práticas comprovadas para um aplicativo Web que usa o [Serviço de Aplicativo do Azure][app-service] e o [Banco de Dados SQL do Azure][sql-db]. [**Implante essa solução.**](#deploy-the-solution)

![[0]][0]

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

## <a name="architecture"></a>Arquitetura 

> [!NOTE]
> Essa arquitetura não se concentra no desenvolvimento de aplicativos e não assume nenhuma estrutura de aplicativo específica. A meta é entender como vários serviços do Azure se combinam.
>
>

A arquitetura tem os seguintes componentes:

* **Grupo de recursos**. Um [grupo de recursos](/azure/azure-resource-manager/resource-group-overview) é um contêiner lógico para recursos do Azure.

* **Aplicativo do Serviço de Aplicativo**. O [Serviço de Aplicativo do Azure][app-service] é uma plataforma totalmente gerenciada para criar e implantar aplicativos de nuvem.     

* **Plano do Serviço de Aplicativo**. Um [Plano do Serviço de Aplicativo][app-service-plans] fornece as VMs (máquinas virtuais) gerenciadas que hospedam o aplicativo. Todos os aplicativos associados a um plano são executados nas mesmas instâncias de VM.

* **Slots de implantação**.  Um [slot de implantação][deployment-slots] permite preparar uma implantação e, em seguida, alterná-la para a implantação de produção. Dessa forma, evita-se implantar diretamente na produção. Consulte a seção [Capacidade de gerenciamento](#manageability-considerations) para obter recomendações específicas.

* **Endereço IP**. O aplicativo Serviço de Aplicativo tem um endereço IP público e um nome de domínio. O nome de domínio é um subdomínio de `azurewebsites.net`, como `contoso.azurewebsites.net`.  

* **DNS do Azure**. [DNS do Azure][azure-dns] é um serviço de hospedagem para domínios DNS, que fornece resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure. Para usar um nome de domínio personalizado (como `contoso.com`), crie registros DNS que mapeiem o nome de domínio personalizado até o endereço IP. Para obter mais informações, consulte [Configurar um nome de domínio personalizado no Serviço de Aplicativo do Azure][custom-domain-name].  

* **Banco de dados SQL do Azure**. O [Banco de Dados SQL][sql-db] é um banco de dados relacional como um serviço na nuvem. O Banco de Dados SQL compartilha a sua base de código com o mecanismo do banco de dados do Microsoft SQL Server. Dependendo dos requisitos de aplicativo, você também pode usar o [banco de dados do Azure para MySQL](/azure/mysql) ou o [banco de dados do Azure para PostgreSQL](/azure/postgresql). Estes são serviços de banco de dados totalmente gerenciados, baseados no código-fonte aberto MySQL Server e nos mecanismos de banco de dados do Postgres, respectivamente.

* **Servidor lógico**. No Banco de Dados SQL do Azure, um servidor lógico hospeda os seus bancos de dados. Você pode criar vários bancos de dados por servidor lógico.

* **Armazenamento do Azure**. Crie uma conta de armazenamento do Azure com um contêiner de blob para armazenar logs de diagnóstico.

* **Azure Active Directory** (Azure AD). Use o Azure AD ou outro provedor de identidade para autenticação.

## <a name="recommendations"></a>Recomendações

Seus requisitos podem ser diferentes dos requisitos da arquitetura descrita aqui. Use as recomendações nesta seção como um ponto inicial.

### <a name="app-service-plan"></a>Plano do Serviço de Aplicativo
Use as camadas Standard ou Premium porque elas dão suporte para expansão, dimensionamento automático e protocolo SSL. Cada camada dá suporte a vários *tamanhos de instância* que diferem por número de núcleos e memória. Depois de criar um plano, você pode alterar a camada ou o tamanho da instância. Para obter mais informações sobre os Planos do Serviço de Aplicativo, consulte [Preços do Serviço de Aplicativo][app-service-plans-tiers].

Você será cobrado pelas instâncias no Plano do Serviço de Aplicativo, mesmo se o aplicativo estiver parado. Lembre-se de excluir os planos que você não estiver usando (por exemplo, implantações de teste).

### <a name="sql-database"></a>Banco de dados SQL
Use a [versão V12][sql-db-v12] do Banco de Dados SQL. O Banco de Dados SQL dá suporte às [camadas de serviço][sql-db-service-tiers] Básico, Standard e Premium, com vários níveis de desempenho em cada camada que são medidos em [DTUs (Unidades de Transmissão de Dados)][sql-dtu]. Execute o planejamento de capacidade e escolha uma camada e um nível de desempenho que atendam às suas necessidades.

### <a name="region"></a>Região
Provisione o Plano do Serviço de Aplicativo e o Banco de Dados SQL na mesma região para minimizar a latência de rede. Em geral, escolha a região mais próxima aos usuários.

O grupo de recursos também tem uma região, que especifica onde armazenar os metadados de implantação. Coloque o grupo de recursos e seus recursos na mesma região. Isso pode melhorar a disponibilidade durante a implantação. 

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Um grande benefício do Serviço de Aplicativo do Azure é a capacidade de dimensionar o aplicativo com base na carga. Aqui estão algumas considerações para ter em mente ao planejar dimensionar seu aplicativo.

### <a name="scaling-the-app-service-app"></a>Dimensionando o aplicativo do Serviço de Aplicativo

Há dois modos de dimensionar um aplicativo do Serviço de Aplicativo:

* *Aumentar*, que significa alterar o tamanho da instância. O tamanho da instância determina a memória, o número de núcleos e o armazenamento em cada instância VM. Você pode aumentar manualmente, alterando o tamanho da instância ou a camada do plano.  

* *Expandir*, que significa adicionar instâncias para lidar com o aumento da carga. Cada tipo de preço tem um número máximo de instâncias. 

  Você pode expandir manualmente alterando a contagem de instâncias ou usar o [dimensionamento automático][web-app-autoscale] para que o Azure adicione ou remova instâncias automaticamente com base nas métricas de agendamento e/ou de desempenho. Cada operação de escala ocorre rapidamente, normalmente em questão de segundos. 

  Para habilitar o dimensionamento automático, crie um *perfil* de dimensionamento automático que defina o número mínimo e máximo de instâncias. Os perfis podem ser agendados. Por exemplo, você pode criar perfis separados para dias da semana e finais de semana. Opcionalmente, um perfil pode conter regras de quando adicionar ou remover instâncias. (Exemplo: adicionar duas instâncias se o uso da CPU ficar acima de 70% durante 5 minutos.)
  
Recomendações para dimensionar um aplicativo Web:

* Evite ao máximo escalar e reduzir verticalmente, pois isso pode disparar um reinício do aplicativo. Nesse caso, selecione uma camada e um tamanho que atendam aos seus requisitos de desempenho para uma carga típica e, em seguida, aumente as instâncias para lidar com as alterações no volume de tráfego.    
* Habilite o dimensionamento automático. Se seu aplicativo tiver uma carga de trabalho previsível e regular, crie perfis para agendar as contagens de instância antecipadamente. Se a carga de trabalho não for previsível, use o dimensionamento automático baseado em regras para reagir às alterações na carga conforme elas ocorrem. Você pode combinar as duas abordagens.
* Geralmente, o uso da CPU é uma boa métrica para as regras de dimensionamento automático. No entanto, você deve fazer um teste de carga do seu aplicativo, identificar gargalos potenciais e basear as regras de dimensionamento automático nesses dados.  
* As regras de dimensionamento automático incluem um tempo de *resfriamento*, que é o intervalo de espera após a conclusão de uma ação de escala antes do início de uma nova ação de escala. O período de resfriamento permite que o sistema se estabilize antes de ser escalado novamente. Defina um período de resfriamento menor para adicionar instâncias e um período de resfriamento maior para remover instâncias. Por exemplo, defina 5 minutos para adicionar uma instância, mas 60 minutos para remover uma instância. É melhor adicionar novas instâncias rapidamente quando a carga está pesada para lidar com o tráfego adicional e, quando necessário, reduzir novamente gradualmente.

### <a name="scaling-sql-database"></a>Dimensionando o Banco de Dados SQL
Caso seja necessário elevar uma camada de serviço ou um nível de desempenho do Banco de Dados SQL, você poderá aumentar os bancos de dados individuais, sem tempo de inatividade do aplicativo. Para obter mais informações, consulte [Opções e desempenho do Banco de Dados SQL: entenda o que está disponível em cada camada de serviço][sql-db-scale].

## <a name="availability-considerations"></a>Considerações sobre disponibilidade
No momento da gravação, o SLA (Contrato de Nível de Serviço) do Serviço de Aplicativo é de 99,95% e o SLA do Banco de Dados SQL é de 99,99% para as camadas Básica, Standard e Premium. 

> [!NOTE]
> O SLA do Serviço de Aplicativo aplica-se a instâncias únicas e múltiplas.  
>
>

### <a name="backups"></a>Backups
No caso de perda de dados, o Banco de Dados SQL fornece uma Recuperação Pontual e uma restauração geográfica. Esses recursos estão disponíveis em todas as camadas e são habilitados automaticamente. Você não precisa agendar ou gerenciar os backups. 

- Use a Recuperação Pontual para [recuperar após um erro humano][sql-human-error], retornando o banco de dados para um ponto anterior. 
- Use a restauração geográfica para [recuperar após uma interrupção de serviço][sql-outage-recovery] restaurando um banco de dados de um backup com redundância geográfica. 

Para obter mais informações, consulte [Continuidade dos negócios e recuperação de desastre de banco de dados em nuvem com o Banco de Dados SQL][sql-backup].

O Serviço de Aplicativo fornece um recurso de [backup e restauração][web-app-backup] para os arquivos de aplicativo. No entanto, lembre-se que os arquivos de backup incluem configurações de aplicativo em texto sem formatação e podem incluir segredos, como cadeias de conexão. Evite usar o recurso de backup do Serviço de Aplicativo para fazer backup dos bancos de dados SQL, porque ele exporta o banco de dados para um arquivo .bacpac do SQL, consumindo [DTUs][sql-dtu]. Nesse caso, use a Recuperação Pontual do banco de dados SQL descrita acima.

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento
Crie grupos de recursos separados para ambientes de produção, desenvolvimento e teste. Isso facilita o gerenciamento de implantações, a exclusão de implantações de teste e a atribuição de direitos de acesso.

Ao atribuir recursos para grupos de recursos, considere o seguinte:

* Ciclo de vida. Em geral, coloque recursos com o mesmo ciclo de vida no mesmo grupo de recursos.
* Acesso. Você pode usar o [RBAC][rbac] (controle de acesso baseado em função) para aplicar políticas de acesso aos recursos em um grupo.
* Cobrança. Você pode exibir os custos acumulados para o grupo de recursos.  

Para saber mais, consulte [Visão geral do Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview).

### <a name="deployment"></a>Implantação
A implantação envolve duas etapas:

1. Provisionamento de recursos do Azure. Recomendamos que você use [Modelos do Azure Resource Manager][arm-template] para esta etapa. Os modelos facilitam a automatização das implantações por meio do PowerShell ou da CLI (interface de linha de comando) do Azure.
2. Implantando o aplicativo (código, binários e arquivos de conteúdo). Existem várias opções, incluindo a implantação de um repositório Git local, usando o Visual Studio ou a implantação contínua usando o controle do código-fonte baseado em nuvem. Consulte [Implantar o aplicativo no Serviço de Aplicativo do Azure][deploy].  

Um aplicativo do Serviço de Aplicativo sempre tem um slot de implantação denominado `production`, que representa o site de produção. É recomendável criar um slot de preparo para implantar atualizações. Os benefícios do uso de um slot de preparo incluem:

* Você pode verificar se a implantação foi bem-sucedida, antes de alterná-la para produção.
* Implantar em um slot de preparo garante que todas as instâncias sejam aquecidas antes de serem alternadas para produção. Muitos aplicativos têm um tempo significativo de aquecimento e de resfriamento.

Também é recomendável criar um terceiro slot para manter a última implantação válida. Depois de alternar o preparo e a produção, mova a implantação de produção anterior (que agora está no processo de preparo) para o último slot válido. Dessa forma, se você descobrir um problema mais tarde, será possível reverter rapidamente para a última versão válida.

![[1]][1]

Ao reverter para uma versão anterior, verifique se todas as alterações de esquema do banco de dados são compatíveis com as versões anteriores.

Não use os slots da implantação de produção para teste, porque todos os aplicativos no mesmo Plano do Serviço de Aplicativo compartilham as mesmas instâncias de VM. Por exemplo, testes de carga podem prejudicar o site de produção dinâmica. Nesse caso, crie Planos do Serviço de Aplicativo separados para teste e produção. Colocando as implantações de teste em um plano separado, você as isola da versão de produção.

### <a name="configuration"></a>Configuração
Armazene definições de configuração como [configurações do aplicativo][app-settings]. Defina as configurações do aplicativo em seus modelos do Resource Manager ou usando o PowerShell. No tempo de execução, as configurações de aplicativo ficam disponíveis para o aplicativo como variáveis de ambiente.

Nunca verifique senhas, chaves de acesso ou cadeias de conexão no controle do código-fonte. Nesse caso, passe esses parâmetros para um script de implantação que armazena esses valores como configurações do aplicativo.

Quando você alterna um slot de implantação, as configurações do aplicativo são alternadas por padrão. Se você precisar de configurações diferentes para produção e preparo, será possível criar configurações de aplicativo que permanecem em um slot e não são alternadas.

### <a name="diagnostics-and-monitoring"></a>Diagnóstico e monitoramento
Habilite o [log de diagnósticos][diagnostic-logs], incluindo o log de aplicativo e o log do servidor Web. Configure o log para usar o Armazenamento de Blobs. Por motivos de desempenho, crie uma conta de armazenamento separada para logs de diagnóstico. Não use a mesma conta de armazenamento para logs e dados de aplicativo. Para obter uma orientação mais detalhada sobre log, consulte [Orientação para monitoramento e diagnóstico][monitoring-guidance].

Use um serviço, como [New Relic][new-relic] ou [Application Insights][app-insights] para monitorar o desempenho do aplicativo e comportamento em relação à carga. Observe os [limites de taxa de dados][app-insights-data-rate] do Application Insights.

Execute testes de carga, usando uma ferramenta como o [Visual Studio Team Services][vsts]. Para obter uma visão geral da análise de desempenho em aplicativos de nuvem, consulte [Performance Analysis Primer][perf-analysis] (Primer de análise de desempenho).

Dicas para solucionar problemas do aplicativo:

* Use a [folha Solucionar problemas][troubleshoot-blade] no portal do Azure para encontrar soluções para problemas comuns.
* Habilite o [streaming de log][web-app-log-stream] para ver as informações de log quase em tempo real.
* O [painel Kudu][kudu] tem várias ferramentas para monitorar e depurar seu aplicativo. Para obter mais informações, consulte [Azure Websites online tools you should know about][kudu] (Ferramentas online dos sites do Azure que você precisa conhecer) (postagem no blog). Você pode acessar o painel Kudu no portal do Azure. Abra a folha do aplicativo, clique em **Ferramentas** e, em seguida, clique em **Kudu**.
* Se você usa o Visual Studio, consulte o artigo [Solucionar problemas de um aplicativo Web no Serviço de Aplicativo do Azure usando o Visual Studio][troubleshoot-web-app] para obter dicas de depuração e solução de problemas.

## <a name="security-considerations"></a>Considerações de segurança
Esta seção lista as considerações sobre segurança específicas dos serviços do Azure descritos neste artigo. Esta não é uma lista completa de práticas recomendadas de segurança. Para obter algumas considerações de segurança adicionais, consulte [Proteger um aplicativo no Serviço de Aplicativo do Azure][app-service-security].

### <a name="sql-database-auditing"></a>Auditoria do Banco de Dados SQL
A auditoria pode ajudar a manter a conformidade regulamentar e obter informações sobre discrepâncias e irregularidades que podem indicar preocupações comerciais ou suspeitas de violações de segurança. Consulte [Introdução à auditoria do banco de dados SQL][sql-audit].

### <a name="deployment-slots"></a>Slots de implantação
Cada slot de implantação tem um endereço IP público. Proteja os slots que não são de produção usando o [logon do Azure Active Directory][aad-auth] para que somente os membros das equipes de desenvolvimento e de DevOps possam acessar esses pontos de extremidade.

### <a name="logging"></a>Registro em log
Os logs nunca devem registrar senhas de usuários ou outras informações que possam ser usadas para confirmação de fraude de identidade. Remova esses detalhes dos dados antes de armazená-los.   

### <a name="ssl"></a>SSL
Um aplicativo do Serviço de Aplicativo inclui um ponto de extremidade SSL em um subdomínio do `azurewebsites.net` sem custo adicional. O ponto de extremidade SSL inclui um certificado curinga para o domínio `*.azurewebsites.net`. Ao usar um nome de domínio personalizado, você deverá fornecer um certificado que corresponda ao domínio personalizado. A abordagem mais simples é comprar um certificado diretamente no portal do Azure. Você também pode importar certificados de outras autoridades de certificação. Para obter mais informações, consulte [Comprar e configurar um certificado SSL para seu Serviço de Aplicativo do Azure][ssl-cert].

Como prática recomendada de segurança, o aplicativo deve impor HTTPS redirecionando as solicitações HTTP. Você pode implementar isso dentro do aplicativo ou usar uma regra de reescrita de URL, conforme a descrição em [Habilitar HTTPS para um aplicativo no Serviço de Aplicativo do Azure][ssl-redirect].

### <a name="authentication"></a>Autenticação
Recomendamos a autenticação por meio de um IDP (provedor de identidade), como Azure AD, Facebook, Google ou Twitter. Use o OAuth 2 ou o OIDC (OpenID Connect) para o fluxo de autenticação. O Azure AD fornece a funcionalidade de gerenciar usuários e grupos, criar funções de aplicativo, integrar as identidades locais e consumir serviços de back-end, como o Office 365 e o Skype for Business.

Evite fazer com que o aplicativo gerencie as credenciais e os logons de usuário diretamente, pois isso cria uma possível superfície de ataque.  No mínimo, seria necessário ter um email de confirmação, uma recuperação de senha e uma autenticação multifator, validar a força da senha e armazenar hashes de senha com segurança. Os grandes provedores de identidade cuidam de tudo isso para você e estão sempre monitoramento e melhorando as práticas de segurança.

Considere o uso da [Autenticação do Serviço de Aplicativo][app-service-auth] para implementar o fluxo de autenticação OAuth/OIDC. Os benefícios de autenticação do Serviço de Aplicativo incluem:

* Fácil de configurar.
* Nenhum código é necessário para cenários de autenticação simples.
* Dá suporte para autorização delegada usando tokens de acesso OAuth para consumir recursos em nome do usuário.
* Fornece um cache de token interno.

Algumas limitações da autenticação do Serviço de Aplicativo:  

* Opções de personalização limitadas.
* A autorização delegada é restrita a um recurso de back-end por sessão de logon.
* Se você usar mais de um IDP, não haverá nenhum mecanismo interno para descoberta de realm inicial.
* Para cenários multilocatários, o aplicativo deve implementar a lógica para validar o emissor do token.

## <a name="deploy-the-solution"></a>Implantar a solução
Um modelo do Resource Manager de exemplo dessa arquitetura está [disponível no GitHub][paas-basic-arm-template].

Para implantar o modelo usando o PowerShell, execute os seguintes comandos:

```
New-AzureRmResourceGroup -Name <resource-group-name> -Location "West US"

$parameters = @{"appName"="<app-name>";"environment"="dev";"locationShort"="uw";"databaseName"="app-db";"administratorLogin"="<admin>";"administratorLoginPassword"="<password>"}

New-AzureRmResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile .\PaaS-Basic.json -TemplateParameterObject  $parameters
```

Para obter mais informações, consulte [Implantar recursos com modelos do Azure Resource Manager][deploy-arm-template].

<!-- links -->

[aad-auth]: /azure/app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication
[app-insights]: /azure/application-insights/app-insights-overview
[app-insights-data-rate]: /azure/application-insights/app-insights-pricing
[app-service]: https://azure.microsoft.com/documentation/services/app-service/
[app-service-auth]: /azure/app-service-api/app-service-api-authentication
[app-service-plans]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[app-service-plans-tiers]: https://azure.microsoft.com/pricing/details/app-service/
[app-service-security]: /azure/app-service-web/web-sites-security
[app-settings]: /azure/app-service-web/web-sites-configure
[arm-template]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[azure-dns]: /azure/dns/dns-overview
[custom-domain-name]: /azure/app-service-web/web-sites-custom-domain-name
[deploy]: /azure/app-service-web/web-sites-deploy
[deploy-arm-template]: /azure/resource-group-template-deploy
[deployment-slots]: /azure/app-service-web/web-sites-staged-publishing
[diagnostic-logs]: /azure/app-service-web/web-sites-enable-diagnostic-log
[kudu]: https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/
[monitoring-guidance]: ../../best-practices/monitoring.md
[new-relic]: http://newrelic.com/
[paas-basic-arm-template]: https://github.com/mspnp/reference-architectures/tree/master/managed-web-app/basic-web-app/Paas-Basic/Templates
[perf-analysis]: https://github.com/mspnp/performance-optimization/blob/master/Performance-Analysis-Primer.md
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[sla]: https://azure.microsoft.com/support/legal/sla/
[sql-audit]: /azure/sql-database/sql-database-auditing-get-started
[sql-backup]: /azure/sql-database/sql-database-business-continuity
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-db-overview]: /azure/sql-database/sql-database-technical-overview
[sql-db-scale]: /azure/sql-database/sql-database-service-tiers#scaling-up-or-scaling-down-a-single-database
[sql-db-service-tiers]: /azure/sql-database/sql-database-service-tiers
[sql-db-v12]: /azure/sql-database/sql-database-features
[sql-dtu]: /azure/sql-database/sql-database-service-tiers
[sql-human-error]: /azure/sql-database/sql-database-business-continuity#recover-a-database-after-a-user-or-application-error
[sql-outage-recovery]: /azure/sql-database/sql-database-business-continuity#recover-a-database-to-another-region-from-an-azure-regional-data-center-outage
[ssl-redirect]: /azure/app-service-web/web-sites-configure-ssl-certificate#bkmk_enforce
[sql-resource-limits]: /azure/sql-database/sql-database-resource-limits
[ssl-cert]: /azure/app-service-web/web-sites-purchase-ssl-web-site
[troubleshoot-blade]: https://azure.microsoft.com/updates/self-service-troubleshooting-for-app-service-web-apps-customers/
[troubleshoot-web-app]: /azure/app-service-web/web-sites-dotnet-troubleshoot-visual-studio
[visio-download]: https://archcenter.azureedge.net/cdn/app-service-reference-architectures.vsdx
[vsts]: https://www.visualstudio.com/features/vso-cloud-load-testing-vs.aspx
[web-app-autoscale]: /azure/app-service-web/web-sites-scale
[web-app-backup]: /azure/app-service-web/web-sites-backup
[web-app-log-stream]: /azure/app-service-web/web-sites-enable-diagnostic-log#streamlogs
[0]: ./images/basic-web-app.png "Arquitetura de um aplicativo Web do Azure básico"
[1]: ./images/paas-basic-web-app-staging-slots.png "Alternando slots para implantações de produção e preparo"
