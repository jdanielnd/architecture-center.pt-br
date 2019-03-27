---
title: 'CAF: Como funciona o Azure?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
description: Explicação sobre o funcionamento interno do Azure
author: petertaylor9999
ms.date: 02/11/2019
ms.openlocfilehash: 724d16a810865dd947a7ade34766818c8ea525a1
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245267"
---
<!-- markdownlint-disable MD026 -->

# <a name="how-does-azure-work"></a>Como funciona o Azure?

O Azure é a plataforma de nuvem pública da Microsoft. O Azure oferece uma grande coleção de serviços, incluindo PaaS (plataforma como serviço), IaaS (infraestrutura como serviço), DBaaS (banco de dados como serviço) e muitos outros. Mas o que exatamente é o Azure e como ele funciona?

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ixGo]

<!-- markdownlint-enable MD034 -->

O Azure, como outras plataformas de nuvem, se baseia em uma tecnologia conhecida como **virtualização**. A maior parte do hardware do computador pode ser emulada em software, porque a maior parte dele é apenas um conjunto de instruções codificado de forma permanente ou semipermanente em silício. Usando uma camada de emulação que mapeia instruções de software para instruções de hardware, o hardware virtualizado pode ser executado em um software como se fosse o próprio hardware real.

Basicamente, a nuvem é um conjunto de servidores físicos em um ou mais datacenters que executam o hardware virtualizado em nome dos clientes. Então, como a nuvem cria, inicia, interrompe e exclui milhões de instâncias de hardware virtualizado para milhões de clientes simultaneamente?

Para entender isso, vamos examinar a arquitetura do hardware no datacenter.  Em cada datacenter existe uma coleção de servidores que residem em racks de servidor. Cada rack de servidor contém muitas **folhas** de servidor, bem como um comutador de rede que fornece conectividade de rede e uma PDU (unidade de distribuição de energia) que fornece energia. Às vezes, os racks são agrupados em unidades maiores conhecidas como **clusters**.

Em cada rack ou cluster, a maioria dos servidores é designada para executar essas instâncias de hardware virtualizado em nome do usuário. No entanto, vários dos servidores executam um software de gerenciamento da nuvem conhecido como um controlador de malha. O **controlador de malha** é um aplicativo distribuído com muitas responsabilidades. Ele aloca serviços, monitora a integridade do servidor e dos serviços em execução nele e restaura servidores quando ocorre uma falha.

Cada instância do controlador de malha está conectada a outro conjunto de servidores que executam o software de orquestração da nuvem, normalmente conhecido como um **front-end**. O front-end hospeda os serviços Web, as APIs RESTful e os bancos de dados internos do Azure usados para todas as funções executadas pela nuvem.

Por exemplo, o front-end hospeda os serviços que manipulam as solicitações de cliente para alocar recursos do Azure como [redes virtuais][vnet], [máquinas virtuais][vms] e serviços como o [Cosmos DB][cosmosdb]. Primeiro, o front-end valida o usuário e verifica se o usuário está autorizado a alocar os recursos solicitados. Nesse caso, o front-end consulta um banco de dados para localizar um rack de servidor com capacidade suficiente e, em seguida, instrui o controlador de malha no rack a alocar o recurso.

Portanto, em termos simples, o Azure é uma grande coleção de servidores e hardware de rede, junto com um conjunto complexo de aplicativos distribuídos que orquestra a configuração e operação do software e hardware virtualizado nesses servidores. E é essa orquestração que faz o Azure tão eficaz – os usuários deixam de ser responsáveis por manter e atualizar o hardware, pois o Azure faz tudo isso em segundo plano.

## <a name="next-steps"></a>Próximas etapas

Agora que você já entende o funcionamento interno do Azure, saiba mais sobre a governança de recursos da nuvem.

> [!div class="nextstepaction"]
> [Saiba mais sobre governança de recursos](what-is-governance.md)

<!-- Links -->

[cosmosdb]: /azure/cosmos-db/introduction
[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[vms]: /azure/virtual-machines/
[vnet]: /azure/virtual-network/virtual-networks-overview
