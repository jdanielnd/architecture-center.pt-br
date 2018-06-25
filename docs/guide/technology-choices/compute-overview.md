---
title: Visão geral das opções de computação do Azure
description: Visão geral das opções de computação do Azure
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: ceb70f8eeff42e6cadb8a63c2f36986f26322201
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206492"
---
# <a name="overview-of-azure-compute-options"></a>Visão geral das opções de computação do Azure

O termo *computação* refere-se ao modelo de hospedagem para os recursos de computação em que seu aplicativo é executado. 

## <a name="overview"></a>Visão geral

Em uma extremidade do espectro está a **Infraestrutura como serviço (IaaS)**. Com o IaaS, você provisiona as VMs de que precisa, juntamente com componentes associados de rede e armazenamento. Depois implante os software e aplicativos que desejar nas VMs. Esse modelo é o mais próximo a um ambiente local tradicional, com a exceção de que a Microsoft gerencia a infraestrutura. Você ainda gerencia as VMs individuais.  

A **Plataforma como serviço** (PaaS) fornece um ambiente de hospedagem gerenciado, no qual você pode implantar seu aplicativo sem a necessidade de gerenciar VMs ou recursos de rede. Por exemplo, em vez de criar VMs individuais, você especifica uma contagem de instâncias, e o serviço vai provisionar, configurar e gerenciar os recursos necessários. O Serviço de Aplicativo do Azure é um exemplo de um serviço de PaaS.

Há uma variedade de IaaS para PaaS puro. Por exemplo, VMs do Azure podem dimensionar automaticamente usando Conjuntos de Dimensionamento de VMs. Esse recurso de dimensionamento automático não é estritamente de PaaS, mas é do tipo de recurso de gerenciamento que pode ser encontrado em um serviço de PaaS.

As **Funções como serviço** (FaaS) vão ainda mais além na remoção da necessidade de se preocupar sobre o ambiente de hospedagem. Em vez de criar instâncias de computação e código de implantação para essas instâncias, basta implantar seu código, e o serviço o executa automaticamente. Não é necessário administrar os recursos de computação. Esses serviços usam uma arquitetura sem servidor e perfeitamente escalam ou reduzem verticalmente para qualquer nível necessário para tratar do tráfego. As Azure Functions são um serviço FaaS.

A IaaS oferece mais controle, flexibilidade e portabilidade. As FaaS fornecem simplicidade, escala elástica e economia de custo potencial, porque você paga apenas pelo tempo em que o código está em execução. A PaaS se encontra em um ponto entre os dois. Em geral, quanto mais flexibilidade um serviço fornece, mais você é responsável por configurar e gerenciar os recursos. Serviços de FaaS gerenciam automaticamente quase todos os aspectos da execução de um aplicativo, enquanto soluções IaaS requerem que você provisione, configure e gerencie as VMs e os componentes de rede criados por você.

## <a name="azure-compute-options"></a>Opções de computação do Azure

Estas são as principais opções de computação disponíveis no Azure atualmente:

- As [Máquinas Virtuais](/azure/virtual-machines/) são um serviço IaaS e permitem que você implante e gerencie VMs dentro de uma rede virtual (VNet).
- O [Serviço de Aplicativo](/azure/app-service/app-service-value-prop-what-is) é uma oferta de PaaS gerenciada para hospedar aplicativos Web, back-ends de aplicativo móvel, APIs RESTful ou processos automatizados de negócios.
- O [Service Fabric](/azure/service-fabric/service-fabric-overview) é uma plataforma de sistemas distribuídos que pode ser executado em vários ambientes, incluindo o Azure ou no local. O Service Fabric é um orquestrador de microsserviços em um cluster de computadores. 
- O [Serviço de Contêiner do Azure](/azure/container-service/container-service-intro) permite criar, configurar e gerenciar um cluster de VMs pré-configuradas para executar aplicativos em contêineres.
- As [Instâncias de Contêiner do Azure](/azure/container-instances/container-instances-overview) oferecem a maneira mais rápida e simples para executar um contêiner no Azure, sem a necessidade de provisionar máquinas virtuais nem adotar um serviço de nível superior.
- O [Azure Functions](/azure/azure-functions/functions-overview) é um serviço gerenciado de FaaS.
- O [Lote do Azure](/azure/batch/batch-technical-overview) é um serviço gerenciado para execução de aplicativos HPC (computação de alto desempenho) paralelos e em grande escala.
- O [Serviços de Nuvem](/azure/cloud-services/cloud-services-choose-me) é um serviço gerenciado para executar aplicativos de nuvem. Ele usa um modelo de hospedagem PaaS. 

Ao selecionar uma opção de computação, estes são alguns fatores a serem considerados:

- Modelo de hospedagem. Como o serviço está hospedado? Quais requisitos e limitações são impostos por esse ambiente de hospedagem? 
- DevOps. Há suporte interno para atualizações de aplicativo? Qual é o modelo de implantação?
- Escalabilidade. Como o serviço trata da adição ou remoção de instâncias? Ele consegue dimensionar automaticamente baseado na carga e em outras métricas? 
- Disponibilidade. Qual é o serviço de SLA? 
- Custo. Além do custo do serviço em si, leve em consideração o custo de operações para gerenciar uma solução compilada nesse serviço. Por exemplo, soluções IaaS podem ter um custo de operações maior.
- Quais são as limitações gerais de cada serviço? 
- Quais tipos de arquiteturas de aplicativo são apropriados para esse serviço? 

## <a name="next-steps"></a>Próximas etapas

Para ter ajuda na hora de selecionar um serviço de computação para o seu aplicativo, use a [árvore de decisão para serviços de computação do Azure](./compute-decision-tree.md)

Para obter uma comparação mais detalhada das opções de computação no Azure, confira [Critérios para escolher um serviço de computação do Azure](./compute-comparison.md).
