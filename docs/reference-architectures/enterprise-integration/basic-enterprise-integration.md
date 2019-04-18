---
title: Integração empresarial básica usando o Azure
titleSuffix: Azure Reference Architectures
description: Arquitetura recomendada para implementação de um padrão de integração empresarial usando Aplicativos Lógicos do Azure e Gerenciamento de API do Azure.
services: logic-apps
author: mattfarm
ms.reviewer: jonfan, estfan, LADocs
ms.topic: reference-architecture
ms.date: 12/03/2018
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: integration-services
ms.openlocfilehash: 16fce3a85cbc0a94dd93277d942fae51ae0e4c04
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640831"
---
# <a name="basic-enterprise-integration-on-azure"></a>Integração empresarial básica no Azure

Essa arquitetura de referência usa os [Azure Integration Services][integration-services] para orquestrar chamadas para sistemas de back-end empresariais. Os sistemas de back-end podem incluir sistemas SaaS (software como serviço), serviços do Azure e serviços da Web existentes em sua empresa.

Os Azure Integration Services são uma coleção de serviços para a integração de aplicativos e dados. Essa arquitetura usa dois desses serviços: [Aplicativos Lógicos][logic-apps] para orquestrar fluxos de trabalho, e [Gerenciamento de API][apim] para criar catálogos de APIs. Essa arquitetura é suficiente para cenários de integração básica nos quais o fluxo de trabalho é disparado por chamadas síncronas para serviços de back-end. Uma arquitetura mais sofisticada que usa [filas e eventos](./queues-events.md) aproveita essa arquitetura básica.

![Diagrama de arquitetura - integração empresarial simples](./_images/simple-enterprise-integration.png)

## <a name="architecture"></a>Arquitetura

A arquitetura tem os seguintes componentes:

- **Sistemas de back-end**. No lado direito do diagrama estão os vários sistemas de back-end que a empresa implantou ou dos quais ela depende. Podem incluir sistemas de SaaS, outros serviços do Azure ou serviços da Web que expõem pontos de extremidade REST ou SOAP.

- **Aplicativos Lógicos do Azure**. Os [Aplicativos Lógicos][logic-apps] são uma plataforma sem servidor para compilar fluxos de trabalho corporativos que integram aplicativos, dados e serviços. Nessa arquitetura, os aplicativos lógicos são disparados por solicitações HTTP. Você também pode aninhar fluxos de trabalho para uma orquestração mais complexa. Os Aplicativos Lógicos usam [conectores][logic-apps-connectors] para integração com serviços usados normalmente. Os Aplicativos Lógicos oferecem centenas de conectores, e você também pode criar conectores personalizados.

- **Gerenciamento de API do Azure**. O [Gerenciamento de API][apim] é um serviço gerenciado para publicação de catálogos de APIs HTTP, para promover a reutilização e a capacidade de descoberta. O Gerenciamento de API é formado por dois componentes relacionados:

  - **Gateway de API**. O gateway de API aceita chamadas HTTP e as encaminha para o back-end.

  - **Portal do desenvolvedor**. Cada instância do Gerenciamento de API do Azure fornece acesso a um [portal do desenvolvedor][apim-dev-portal]. Esse portal dá aos seus desenvolvedores acesso à documentação e a exemplos de códigos para chamar as APIs. Também é possível testar as APIs no portal do desenvolvedor.

  Nessa arquitetura, as APIs de composição são compiladas pela [importação de aplicativos lógicos][apim-logic-app] como APIs. Você também pode importar os serviços Web existentes [importando especificações da OpenAPI][apim-openapi] (Swagger) ou [importando APIs do SOAP][apim-soap] a partir das especificações de WSDL.

  O gateway de API ajuda a separar clientes front-end do back-end. Por exemplo, ele pode reescrever URLs ou transformar as solicitações antes que elas atinjam o back-end. Ele também lida com muitas questões transversais, como autenticação, suporte a CORS (compartilhamento de recurso entre origens) e armazenamento em cache da resposta.

- **DNS do Azure**. O [DNS do Azure][dns] é um serviço de hospedagem para domínios DNS. O DNS do Azure fornece resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Microsoft Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure. Para usar um nome de domínio personalizado, como contoso.com, crie registros DNS que mapeiem o nome de domínio personalizado para o endereço IP. Para saber mais, confira [Configurar um nome de domínio personalizado no Gerenciamento de API][apim-domain].

- **Azure Active Directory (Azure AD)**. Use o [Azure AD][aad] para autenticar clientes que chamam o gateway de API. O Azure AD oferece suporte ao protocolo OIDC (OpenID Connect). Os clientes obtêm um token de acesso do Azure AD e o Gateway de API [valida o token][apim-jwt] para autorizar a solicitação. Ao usar o tipo Standard ou Premium do Gerenciamento de API, o Azure AD também pode proteger o acesso ao portal do desenvolvedor.

## <a name="recommendations"></a>Recomendações

Seus requisitos específicos podem ser diferentes da arquitetura genérica exibida aqui. Use as recomendações nesta seção como um ponto inicial.

### <a name="api-management"></a>Gerenciamento de API

Use as camadas de gerenciamento de API básica, Standard ou Premium. Essas camadas oferecem um SLA (Contrato de Nível de Serviço) de produção e são compatíveis com escala horizontal dentro da região do Azure. A capacidade de taxa de transferência para o Gerenciamento de API é medida em *unidades*. Cada tipo de preço tem uma expansão máxima. A camada Premium também dá suporte para escalar horizontalmente em várias regiões do Azure. Escolha sua camada com base no conjunto de recursos e o nível de taxa de transferência necessário. Para saber mais, confira [Preços do Gerenciamento de API][apim-pricing] e [Capacidade de uma instância de Gerenciamento de API do Azure][apim-capacity].

Cada instância de Gerenciamento de API do Azure tem um nome de domínio padrão, que é um subdomínio do `azure-api.net` &mdash, por exemplo, `contoso.azure-api.net`. Considere a configuração de um [domínio personalizado][apim-domain] para a sua organização.

### <a name="logic-apps"></a>Aplicativos Lógicos

Os Aplicativos Lógicos funcionam melhor em cenários que não exigem baixa latência para uma resposta, por exemplo, chamadas à API de execução semi-longa ou assíncronas. Se for necessário ter baixa latência, por exemplo, em uma chamada que bloqueia uma interface do usuário, use uma tecnologia diferente. Por exemplo, use o Azure Functions ou uma API Web implantada no Serviço de Aplicativo do Azure. Use o Gerenciamento de API para administrar a API para seus consumidores de API.

### <a name="region"></a>Região

Para minimizar a latência de rede, coloque o Gerenciamento de API e os Aplicativos Lógicos na mesma região. Em geral, escolha a região mais próxima dos usuários (ou mais próxima de seus serviços de back-end).

O grupo de recursos também tem uma região. Essa região especifica onde os metadados de implantação são armazenados e onde o modelo de implantação é executado. Para melhorar a disponibilidade durante a implantação, coloque o grupo de recursos e os respectivos recursos na mesma região.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Para elevar a escalabilidade do Gerenciamento de API, adicione [políticas de cache][apim-caching] conforme apropriado. Armazenamento em cache também ajuda a reduzir a carga nos serviços de back-end.

Para oferecer maior capacidade, aumente as camadas Básica, Standard e Premium do Gerenciamento de API em uma região do Azure. Para analisar o uso para seu serviço, no menu **Métrica**, selecione a opção **Métricas de Capacidade** e aumente ou reduza, conforme apropriado. O processo de atualização ou escala pode levar de 15 a 45 minutos para ser aplicado.

Recomendações para colocação em escala de um serviço de Gerenciamento de API:

- Considere os padrões de tráfego ao dimensionar. Os clientes com padrões de tráfego mais voláteis precisam de mais capacidade.

- Capacidade consistente acima de 66% pode indicar a necessidade de expansão.

- Capacidade consistente abaixo de 20% pode indicar uma oportunidade para reduzir verticalmente.

- Antes de habilitar a carga em produção, sempre realize teste de carga no seu serviço do Gerenciamento de API com uma carga representativa.

Com o tipo Premium, você pode dimensionar uma instância do Gerenciamento de API em várias regiões do Azure. Isso qualifica o Gerenciamento de API para um SLA mais alto e permite que você provisione serviços próximos aos usuários em várias regiões.

Em modelos sem servidor dos Aplicativos Lógicos, os administradores não precisam planejar a escalabilidade do serviço. O serviço é dimensionado automaticamente para atender à demanda.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Examine o SLA para cada serviço:

- [SLA de Gerenciamento de API][apim-sla]
- [SLA de Aplicativos Lógicos][logic-apps-sla]

Se você implantar o Gerenciamento de API em duas ou mais regiões com o tipo Premium, ele se tornará qualificado para um SLA mais alto. Confira [Preços de Gerenciamento de API][apim-pricing]

### <a name="backups"></a>Backups

[Faça backups][apim-backup] regularmente da configuração do seu Gerenciamento de API. Armazene seus arquivos de backup em um local ou região do Azure diferente da região onde o serviço está implantado. Com base em seu [RTO][rto], escolha uma estratégia de recuperação de desastres:

- No caso de um evento de recuperação de desastre, provisione uma nova instância de Gerenciamento de API, restaure o backup para a nova instância e reordene os registros de DNS.

- Mantenha uma instância passiva do serviço de Gerenciamento de API em outra região do Azure. Restaure regularmente backups nessa instância, para mantê-la em sincronia com o serviço ativo. Para restaurar o serviço durante um evento de recuperação de desastre, será preciso apenas reordenar os registros DNS. Essa abordagem incorre em custos adicionais, pois paga pela instância passiva, mas reduz o tempo de recuperação.

Para aplicativos lógicos, recomendamos uma abordagem de configuração como código para fazer backup e restauração. Como os aplicativos lógicos são sem servidor, você pode recriá-los rapidamente em modelos do Azure Resource Manager. Salve os modelos no controle do código-fonte, integre os modelos ao seu processo de CI/CD (integração/implantação contínua). No caso de um evento de recuperação de desastres, implante o modelo em uma nova região.

Se você implantar um aplicativo lógico em uma região diferente, atualize a configuração no Gerenciamento de API. É possível atualizar a propriedade de **Back-end** da API usando um script básico do PowerShell.

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Crie grupos de recursos separados para ambientes de produção, desenvolvimento e teste. Ter grupos de recursos separados facilita o gerenciamento de implantações, a exclusão de implantações de teste a atribuição de direitos de acesso.

Ao atribuir recursos a grupos de recursos, considere os seguintes fatores:

- **Ciclo de vida**. Em geral, coloque recursos com o mesmo ciclo de vida no mesmo grupo de recursos.

- **Acesso**. Para aplicar políticas de acesso aos recursos em um grupo, use o [controle de acesso baseado em função][rbac] (RBAC).

- **Cobrança**. Você pode exibir os custos acumulados para o grupo de recursos.

- **Tipo de preço para o gerenciamento de API**. Use o tipo Desenvolvedor para ambientes de desenvolvimento e teste. Para minimizar os custos durante a pré-produção, implante uma réplica do seu ambiente de produção, execute os testes e desligue.

### <a name="deployment"></a>Implantação

Use [modelos do Azure Resource Manager][arm] para implantar recursos do Azure. Os modelos facilitam a automatização das implantações usando o PowerShell ou a CLI do Azure.

Coloque o Gerenciamento de API e os aplicativos lógicos individuais em seus próprios modelos do Resource Manager separados. Usando modelos separados, é possível armazenar os recursos em sistemas de controle do código-fonte. Você pode implantar os modelos juntos ou individualmente como parte de um processo de CI/CD.

### <a name="versions"></a>Versões

Sempre que você fizer uma alteração na configuração do aplicativo lógico ou implantar uma atualização por meio do modelo do Resource Manager, o Azure manterá uma cópia dessa versão, e todas as versões que tiverem uma execução de histórico serão mantidas. Você pode usar essas versões para controlar as alterações históricas ou promover uma versão como a configuração atual do aplicativo lógico. Por exemplo, você pode reverter um aplicativo lógico para uma versão anterior.

O Gerenciamento de API oferece suporte a dois conceitos de versão distintos, mas complementares:

- *Versões* permitem que os consumidores escolham uma versão de API com base nas necessidades, por exemplo, v1, v2, beta ou produção.

- *Revisões* permitem que os administradores de API façam alterações sem interrupções em uma API e implantem essas alterações, juntamente com um log de alterações para informar aos consumidores da API sobre as alterações.

É possível fazer uma revisão em um ambiente de desenvolvimento e implantar essa alteração em outros ambientes usando os modelos do Resource Manager. Para saber mais, confira [Publicar várias versões de sua API][apim-versions]

Você também pode usar as revisões para testar uma API antes de tornar as alterações atuais e acessíveis aos usuários. No entanto, esse método não é recomendado para teste de carga ou teste de integração. Use ambientes de pré-produção ou teste separados.

## <a name="diagnostics-and-monitoring"></a>Diagnóstico e monitoramento

Use o [Azure Monitor][monitor] para monitoramento operacional tanto no Gerenciamento de API quanto nos Aplicativos Lógicos. O Azure Monitor fornece informações com base nas métricas que configuradas para cada serviço e é habilitado por padrão. Para obter mais informações, consulte:

- [Monitorar APIs publicadas][apim-monitor]
- [Monitorar o status, configurar o log de diagnósticos e ativar alertas para os Aplicativos Lógicos do Azure][logic-apps-monitor]

Cada serviço também tem essas opções:

- Para análise e painéis mais aprofundados, envie os logs dos Aplicativos Lógicos para o [Azure Log Analytics][logic-apps-log-analytics].

- Para monitoramento de DevOps, configure o Azure Application Insights para Gerenciamento de API.

- O Gerenciamento de API é compatível com o [modelo de solução do Power BI para análise de APIs personalizadas][apim-pbi]. Use esse modelo de solução para criar sua própria solução de análise. Para usuários empresariais, o Power BI disponibiliza relatórios.

## <a name="security-considerations"></a>Considerações de segurança

Embora essa lista não descreva completamente todas as práticas recomendadas de segurança, vejas essas considerações de segurança que se aplicam especificamente a esta arquitetura:

- O serviço de Gerenciamento de API do Azure tem um endereço IP público fixo. Restrinja o acesso para chamar pontos de extremidade de Aplicativos Lógicos apenas para o endereço IP do Gerenciamento de API. Para saber mais, confira [Restringir endereços IP de entrada][logic-apps-restrict-ip].

- Para garantir que os usuários tenham níveis de acesso apropriados, use o RBAC (controle de acesso baseado em função).

- Proteja os pontos de extremidade de API públicos no Gerenciamento de API usando OAuth ou OpenID Connect. Para proteger os pontos de extremidade de API públicos, configure um provedor de identidade e adicione uma política de validação de JWT (JSON Web Token). Para saber mais, confira [Proteger uma API usando OAuth 2.0 com o Azure Active Directory e o Gerenciamento de API][apim-oauth].

- Conectar os serviços back-end do Gerenciamento de API usando certificados mútuos

- Impor HTTPS sobre as APIs de Gerenciamento de API.

### <a name="storing-secrets"></a>Armazenar segredos

Nunca verifique senhas, chaves de acesso ou cadeias de conexão no controle do código-fonte. Se tais valores forem necessários, proteja e implante-os usando as técnicas apropriadas.

Se um aplicativo lógico exigir qualquer valor confidencial que não é possível ser criado dentro de um conector, armazene tais valores no Azure Key Vault e faça referência a eles em um modelo do Resource Manager. Use os parâmetros de modelo de implantação e arquivos de parâmetros para cada ambiente. Para saber mais, confira [Proteger parâmetros e entradas em um fluxo de trabalho][logic-apps-secure].

O Gerenciamento de API administra os segredos usando objetos chamados *valores* ou *propriedades nomeadas*. Esses objetos armazenam com segurança valores que podem ser acessados por meio das políticas de Gerenciamento de API. Para saber mais, confira [Como usar Valores Nomeados nas políticas de Gerenciamento de API do Azure][apim-properties].

## <a name="cost-considerations"></a>Considerações de custo

Você será cobrado por todas as instâncias de Gerenciamento de API quando estiverem em execução. Se você tiver escalado verticalmente e não precisar desse nível de desempenho o tempo todo, reduza verticalmente de forma manual ou configure o [dimensionamento automático][apim-autoscale].

Os Aplicativos lógicos usam um modelo [sem servidor](/azure/logic-apps/logic-apps-serverless-overview). A cobrança é calculada com base na ação e execução do conector. Para obter mais informações, consulte [Preços de Aplicativos Lógicos](https://azure.microsoft.com/pricing/details/logic-apps/). Atualmente, não há considerações de camada para Aplicativos Lógicos.

## <a name="next-steps"></a>Próximos passos

Para obter mais confiabilidade e escalabilidade, use filas de mensagens e eventos para separar os sistemas de back-end. Esse padrão aparece na próxima arquitetura de referência nesta série: [Integração empresarial usando filas de mensagens e eventos](./queues-events.md).

<!-- links -->

[aad]: /azure/active-directory
[apim]: /azure/api-management
[apim-autoscale]: /azure/api-management/api-management-howto-autoscale
[apim-backup]: /azure/api-management/api-management-howto-disaster-recovery-backup-restore
[apim-caching]: /azure/api-management/api-management-howto-cache
[apim-capacity]: /azure/api-management/api-management-capacity
[apim-dev-portal]: /azure/api-management/api-management-key-concepts#a-namedeveloper-portal-a-developer-portal
[apim-domain]: /azure/api-management/configure-custom-domain
[apim-jwt]: /azure/api-management/policies/authorize-request-based-on-jwt-claims
[apim-logic-app]: /azure/api-management/import-logic-app-as-api
[apim-monitor]: /azure/api-management/api-management-howto-use-azure-monitor
[apim-oauth]: /azure/api-management/api-management-howto-protect-backend-with-aad
[apim-openapi]: /azure/api-management/import-api-from-oas
[apim-pbi]: https://aka.ms/apimpbi
[apim-pricing]: https://azure.microsoft.com/pricing/details/api-management/
[apim-properties]: /azure/api-management/api-management-howto-properties
[apim-sla]: https://azure.microsoft.com/support/legal/sla/api-management/
[apim-soap]: /azure/api-management/import-soap-api
[apim-versions]: /azure/api-management/api-management-get-started-publish-versions
[arm]: /azure/azure-resource-manager/resource-group-authoring-templates
[dns]: /azure/dns/
[integration-services]: https://azure.microsoft.com/product-categories/integration/
[logic-apps]: /azure/logic-apps/logic-apps-overview
[logic-apps-connectors]: /azure/connectors/apis-list
[logic-apps-log-analytics]: /azure/logic-apps/logic-apps-monitor-your-logic-apps-oms
[logic-apps-monitor]: /azure/logic-apps/logic-apps-monitor-your-logic-apps
[logic-apps-restrict-ip]: /azure/logic-apps/logic-apps-securing-a-logic-app#restrict-incoming-ip-addresses
[logic-apps-secure]: /azure/logic-apps/logic-apps-securing-a-logic-app#secure-parameters-and-inputs-within-a-workflow
[logic-apps-sla]: https://azure.microsoft.com/support/legal/sla/logic-apps
[monitor]: /azure/azure-monitor/overview
[rbac]: /azure/role-based-access-control/overview
[rto]: ../../reliability/requirements.md#recovery-metrics