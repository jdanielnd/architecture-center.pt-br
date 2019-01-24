---
title: Gateways de API
description: Gateways de API em microsserviços.
author: MikeWasson
ms.date: 10/23/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 84add394ee9ae6fa3c0df19b5a263dab66a6b140
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485869"
---
# <a name="designing-microservices-api-gateways"></a>Como criar microsserviços: Gateways de API

Em uma arquitetura de microsserviços, um cliente pode interagir com mais de um serviço front-end. Em razão disso, como um cliente sabe quais pontos de extremidade a serem chamados? O que acontece quando são introduzidos novos serviços ou serviços existentes são refatorados? Como os serviços processam terminação SSL, autenticação e outras questões? Um *gateway de API* pode ajudá-lo a enfrentar esses desafios.

![Diagrama de um gateway de API](./images/gateway.png)

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-api-gateway"></a>O que é um gateway de API?

<!-- markdownlint-enable MD026 -->

Um gateway de API fica entre clientes e serviços. Ele atua como um proxy reverso, encaminhando as solicitações de clientes para serviços. Ele também pode executar várias tarefas detalhadas, como autenticação, terminação de SSL e a limitação de taxa. Se você não implantar um gateway, os clientes deverão enviar solicitações diretamente aos serviços front-end. No entanto, há alguns possíveis problemas com a exposição de serviços diretamente aos clientes:

- Isso pode resultar em código de cliente complexo. O cliente deve manter o controle dos vários pontos de extremidade e lidar com falhas de forma resiliente.
- Ele cria um acoplamento entre o cliente e o back-end. O cliente precisa saber como os serviços individuais são decompostos. Isso torna mais difícil manter o cliente e também mais difícil refatorar serviços.
- Uma única operação pode exigir chamadas para vários serviços. Isso pode resultar em várias viagem de ida e volta na rede entre o cliente e o servidor, adicionando latência significativa.
- Cada serviço voltado ao público deve lidar com preocupações como autenticação, SSL e limite de taxa do cliente.
- Serviços devem expor um protocolo de cliente amigável, como HTTP ou WebSocket. Isso limita a escolha de [protocolos de comunicação](./interservice-communication.md).
- Serviços com pontos de extremidade públicos são uma superfície de ataque potencial e devem ser protegidos.

Um gateway ajuda a resolver esses problemas, separando clientes de serviços. Gateways podem executar várias funções diferentes, e você pode não precisar de todas elas. As funções podem ser agrupadas nos seguintes padrões de design:

[Roteamento de Gateway](../patterns/gateway-routing.md). Use o gateway como um proxy reverso para encaminhar solicitações para um ou mais serviços de back-end, usando o roteamento de 7 camadas. O gateway fornece um ponto de extremidade único para os clientes e ajuda a separar clientes de serviços.

[Agregação de Gateway](../patterns/gateway-aggregation.md). Use o gateway para agregar várias solicitações individuais em uma única solicitação. Esse padrão se aplica quando uma única operação exige chamadas para vários serviços de back-end. O cliente envia uma solicitação para o gateway. O gateway envia solicitações para os vários serviços de back-end, agrega os resultados e os envia de volta ao cliente. Isso ajuda a reduzir conversas entre o cliente e o back-end.

[Descarregamento de gateway](../patterns/gateway-offloading.md). Use o gateway para descarregar funcionalidade de serviços individuais no gateway, particularmente questões transversais. Pode ser útil consolidar essas funções em um único local, em vez de responsabilizar cada serviço por implementá-las. Isso é especialmente verdadeiro para recursos que exigem habilidades especializadas para implementação correta, como autenticação e autorização.

Aqui estão alguns exemplos de funcionalidade que pode ser transferida para um gateway:

- Terminação SSL
- Autenticação
- Lista de permissões de IP
- Limitação de taxa do cliente (limitação)
- Log e monitoramento
- Cache de resposta
- Firewall do aplicativo Web
- Compactação GZIP
- Manutenção de conteúdo estático

## <a name="choosing-a-gateway-technology"></a>Escolhendo uma tecnologia de gateway

Aqui estão algumas opções para a implementação de um gateway de API em seu aplicativo.

- **Servidor proxy reverso**. Nginx e HAProxy são servidores de proxy reverso populares compatíveis com recursos como balanceamento de carga, SSL e roteamento de 7 camadas. Os dois são produtos de software livre, com edições pagas que fornecem recursos adicionais e opções de suporte. Nginx e HAProxy são ambos produtos desenvolvidos com conjuntos de recursos avançados e alto desempenho. Você pode estendê-las com módulos de terceiros ou gravar scripts personalizados em Lua. O Nginx também é compatível um módulo de script baseado em JavaScript, chamado NginScript.

- **Controlador de entrada de malha de serviços**. Se você estiver usando uma malha de serviços como linkerd ou Istio, considere os recursos fornecidos pelo controlador de entrada dessa malha de serviços. Por exemplo, o controlador de entrada Istio é compatível com roteamento de 7 camadas, redirecionamentos de HTTP, repetições e outros recursos.

- [Gateway de Aplicativo do Azure](/azure/application-gateway/). O Gateway de Aplicativo é um serviço de balanceamento de carga gerenciado que pode executar roteamento de 7 camadas e terminação SSL. Ele também fornece um WAF (Firewall de Aplicativo Web).

- [Gerenciamento de API do Azure](/azure/api-management/). Gerenciamento de API é uma solução completa para publicar APIs para clientes externos e internos. Ela oferece recursos que são úteis para gerenciar uma API voltado ao público, incluindo limitação de taxa, permissão de IP e autenticação usando o Azure Active Directory ou outros provedores de identidade. Gerenciamento de API não executa balanceamento de carga, portanto deve ser usado em conjunto com um balanceador de carga, como um proxy reverso ou Gateway de Aplicativo. Para obter informações sobre como usar o Gerenciamento de API com o Gateway de Aplicativo, consulte [Integrar o Gerenciamento de API em uma VNET interna com o Gateway de Aplicativo](/azure/api-management/api-management-howto-integrate-internal-vnet-appgateway).

Ao escolher uma tecnologia de gateway, considere o seguinte:

**Recursos**. As opções listadas acima são compatíveis com roteamento de 7 camadas, mas o suporte a outros recursos vai variar. Dependendo dos recursos que você precisa, poderá implantar mais de um gateway.

**Implantação**. Gateway de Aplicativo do Azure e Gerenciamento de API são serviços gerenciados. Nginx e HAProxy serão executado normalmente em contêineres dentro do cluster, mas também podem ser implantados em VMs dedicadas fora do cluster. Isso isola o gateway do restante da carga de trabalho, mas resulta em maior sobrecarga de gerenciamento.

**Gerenciamento**. Talvez seja necessário atualizar as regras de roteamento de gateway quando novos serviços são atualizados ou adicionados. Considere como esse processo será gerenciado. Considerações semelhantes aplicam-se ao gerenciamento de certificados SSL, listas de permissões IP e outros aspectos da configuração.

## <a name="deploying-nginx-or-haproxy-to-kubernetes"></a>Implantando o Nginx ou HAProxy no Kubernetes

Você pode implantar Nginx ou HAProxy no Kubernetes como um [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) ou [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) que especifica a imagem de contêiner Nginx ou HAProxy. Use um ConfigMap para armazenar o arquivo de configuração no proxy e monte o ConfigMap como um volume. Crie um serviço do tipo LoadBalancer para expor o gateway por meio do Azure Load Balancer.

Uma alternativa é criar um controlador de entrada. Um controlador de entrada é um recurso Kubernetes que implanta um servidor proxy reverso ou um balanceador de carga. Existem várias implementações, incluindo Nginx e HAProxy. Um recurso separado chamado uma entrada define as configurações para o controlador de entrada, como regras de roteamento e certificados TLS. Dessa forma, você não precisa gerenciar arquivos de configuração complexos, específicos de uma tecnologia de servidor proxy.

O gateway é um gargalo potencial ou um ponto único de falha no sistema, portanto, sempre implante pelo menos duas réplicas para alta disponibilidade. Talvez seja necessário expandir as réplicas ainda mais, dependendo da carga.

Além disso, considere a execução do gateway em um conjunto dedicado de nós no cluster. Benefícios dessa abordagem:

- Isolamento. Todo o tráfego vai para um conjunto fixo de nós, que pode ser isolado de serviços back-end.

- Configuração estável. Se o gateway estiver mal configurado, todo o aplicativo poderá se tornar indisponível.

- Desempenho. Use uma configuração de VM específica ao gateway por motivos de desempenho.

> [!div class="nextstepaction"]
> [Monitoramento e registro em log](./logging-monitoring.md)
