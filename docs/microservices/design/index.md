---
title: Implementação de referência de microsserviços no Serviço de Kubernetes do Azure
description: Essa implementação de referência ilustra as práticas recomendadas de uma arquitetura de microsserviços.
author: MikeWasson
ms.date: 02/26/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 15e9aa16c0e2cfccecbfb84d217c275cc99a66fd
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58344403"
---
# <a name="designing-a-microservices-architecture"></a>Projetar uma arquitetura de microsserviços

Microsserviços se tornaram um estilo popular de arquitetura para criar aplicativos de nuvem resilientes, altamente escalonáveis, implantáveis independentemente e capazes de evoluir rapidamente. No entanto, para serem mais do que apenas uma palavra de efeito, microsserviços exigem uma abordagem diferente para o design e criação de aplicativos.

Nesse conjunto de artigos, exploraremos como compilar e executar uma arquitetura de microsserviços no Azure. Os tópicos incluem:

- [Comunicação entre serviços](./interservice-communication.md)
- [Design de API](./api-design.md)
- [Gateways de API](./gateway.md)
- [Considerações sobre dados](./data-considerations.md)
- [Padrões de design](./patterns.md)

## <a name="prerequisites"></a>Pré-requisitos

Antes de ler estes artigos, você pode começar por:

- [Introdução às arquiteturas de microsserviços](../introduction.md). Entenda os benefícios e desafios dos microsserviços e quando usar esse estilo de arquitetura.
- [Usar análise de domínio para microsserviços de modelo](../model/domain-analysis.md). Aprenda uma abordagem baseada em domínio a microsserviços de modelagem.

## <a name="reference-implementation"></a>Implementação de referência

Para ilustrar as práticas recomendadas de uma arquitetura de microsserviços, criamos uma implementação de referência que chamamos de aplicativo de Entrega por Drone. Essa implementação é executada no Kubernetes usando o Serviço de Kubernetes do Azure (AKS). Você pode encontrar a implementação de referência no [GitHub][drone-ri].

![Arquitetura do aplicativo de Entrega por Drone](../images/drone-delivery.png)

## <a name="scenario"></a>Cenário

A Fabrikam, Inc. está dando início a um serviço de entrega por drone. A empresa gerencia uma frota de aeronaves de tipo drone. Empresas registram-se no serviço e os usuários podem solicitar que um drone colete mercadorias para entrega. Quando um cliente agenda uma coleta, um sistema de back-end atribui um drone ao usuário e notifica-o com um tempo de entrega previsto. Enquanto a entrega está em andamento, o cliente pode acompanhar a localização do drone, com um ETA atualizado continuamente.

Este cenário envolve um domínio bastante complicado. Algumas das preocupações de negócios incluem o agendamento drones, o acompanhamento de pacotes, o gerenciamento de contas de usuário e o armazenamento e análise de dados históricos. Além disso, Fabrikam deseja se lançar no mercado rapidamente e, em seguida, iterar rapidamente, adicionando novas funcionalidades e capacidades. O aplicativo deve operar em escala de nuvem, com um SLO (objetivo do nível de serviço) elevado. A Fabrikam também espera que diferentes partes do sistema tenham requisitos muito diferentes de armazenamento e consulta de dados. Todas essas considerações levam Fabrikam para escolher uma arquitetura de microsserviços para o aplicativo de Entrega por Drone.

> [!NOTE]
> Para obter ajuda na escolha entre uma arquitetura de microsserviços e outros estilos de arquitetura, consulte o [Guia de arquitetura de aplicativo do Azure](../../guide/index.md).

Nossa implementação de referência usa Kubernetes com o [AKS (Serviço de Kubernetes do Azure)](/azure/aks/). No entanto, muitas das decisões de arquitetura de alto nível e desafios se aplicarão a qualquer orquestrador de contêineres, incluindo o [Azure Service Fabric](/azure/service-fabric/).

<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Escolher uma opção de computação](./compute-options.md)