---
title: 'Adoção do Enterprise Cloud: implantar uma carga de trabalho básica'
description: Descreve como implantar uma carga de trabalho básica para o Azure
author: petertaylor9999
ms.date: 09/10/2018
ms.openlocfilehash: 363e7e6f394389fb6c1577e2cbaeffeddcf2de1a
ms.sourcegitcommit: b38ba378c9d6110da2dfd50b4233fadd94604bb0
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/25/2018
ms.locfileid: "47167346"
---
# <a name="enterprise-cloud-adoption-deploy-a-basic-workload"></a>Adoção da Nuvem na Empresa: implantar uma carga de trabalho básica

O termo **carga de trabalho** é normalmente entendido como a definição de uma unidade arbitrária de recursos, como um aplicativo ou serviço. Pensamos em uma carga de trabalho em termos de artefatos de código que são implantados em um servidor, mas também em termos de quaisquer outros serviços que sejam necessários. Esta é uma definição útil para um aplicativo ou serviço local, mas na nuvem precisamos expandi-lo.

Na nuvem, uma carga de trabalho não só abrange todos os artefatos, mas também inclui recursos de nuvem. Os recursos de nuvem são incluídos como parte de nossa definição devido a um conceito conhecido como infraestrutura como código. Como você aprendeu em [Como funciona o Azure?](../getting-started/what-is-azure.md), os recursos no Azure são implantados por um serviço orquestrador. O serviço orquestrador expõe essa funcionalidade por meio de uma API Web, e essa API Web podem ser chamadas usando várias ferramentas, como o Powershell, a interface de linha de comando (CLI) do Azure e o portal do Azure. Isso significa que podemos especificar nossos recursos em um arquivo legível por máquina que pode ser armazenado junto com os artefatos de código associados ao nosso aplicativo.

Isso nos permite definir uma carga de trabalho em termos de artefatos de código e os recursos de nuvem necessários e isso nos permite isolar mais nossas cargas de trabalho. As cargas de trabalho podem ser isoladas pela forma como os recursos são organizados por topologia de rede ou por outros atributos. O objetivo do isolamento de carga de trabalho é associar recursos específicos de uma carga de trabalho para uma equipe para a equipe de forma independente pode gerenciar todos os aspectos desses recursos. Isso permite que várias equipes compartilharem serviços de gerenciamento de recursos no Azure, evitando a exclusão não intencional ou modificação de uns dos outros recursos.

Esse isolamento também permite outro conceito conhecido como DevOps. DevOps inclui as práticas de desenvolvimento de software que incluem o desenvolvimento de software e as operações de TI acima, mas adiciona o uso de automação tanto quanto possível. Um dos princípios de DevOps é conhecido como integração contínua e entrega contínua (CI/CD). A integração contínua refere-se aos processos de compilação automatizados que são executados sempre que um desenvolvedor confirmar uma alteração de código; e o fornecimento contínuo refere-se aos processos automatizados que implantam esse código em vários ambientes como um ambiente de desenvolvimento para teste ou um ambiente de produção para a implantação final.

## <a name="basic-workload"></a>Carga de trabalho básica

Uma **carga de trabalho básica** normalmente é definida como um aplicativo Web único ou uma VNet (rede virtual) com VM (máquina virtual). 

> [!NOTE]
> Este guia não abrange o desenvolvimento de aplicativos. Para obter mais informações sobre como desenvolver aplicativos no Azure, consulte o [Guia de arquitetura de aplicativo do Azure](/azure/architecture/guide/).

Independentemente se a carga de trabalho é um aplicativo Web ou uma VM, cada uma dessas implantações requer um **grupo de recursos**. Um usuário com permissão para criar um grupo de recursos deve fazer isso antes de seguir as etapas abaixo.

## <a name="basic-web-application-paas"></a>Aplicativo Web básico (PaaS)

Para um aplicativo Web básico, selecione um dos guias de início rápido de 5 minutos da [documentação de aplicativos Web](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json) e siga as etapas. 

> [!NOTE]
> Alguns dos guias de início rápido implantarão um grupo de recursos por padrão. Nesse caso, não é necessário criar um grupo de recursos explicitamente. Caso contrário, implante o aplicativo Web para o grupo de recursos criado acima.

Após a implantação de sua carga de trabalho simples, você pode aprender mais sobre as práticas comprovadas para a implantação de um [aplicativo Web básico](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) no Azure.

## <a name="single-windows-or-linux-vm-iaas"></a>VM única Windows ou Linux (IaaS)

Para uma carga de trabalho simples que é executada em uma máquina virtual, a primeira etapa é implantar uma rede virtual. Todos os recursos de IaaS no Azure como máquinas virtuais, balanceadores de carga e gateways exigem uma rede virtual. Saiba mais sobre [redes virtuais do Azure](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json) e, em seguida, siga as etapas para [implantar uma rede virtual no Azure usando o portal](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Quando você especifica as configurações para a rede virtual no portal do Azure, especifique o nome do grupo de recursos criado acima.

A próxima etapa é decidir se deseja implantar uma VM única Windows ou Linux. Para uma VM Windows, siga as etapas para [implantar uma VM Windows no Azure com o portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Novamente, quando você especifica as configurações para a máquina virtual no portal do Azure, especifique o nome do grupo de recursos criado acima.

Depois que você tiver seguido as etapas e implantado a máquina virtual, você pode aprender sobre as [práticas comprovadas para executar uma VM Windows no Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json). Para uma VM Linux, siga as etapas para [implantar uma VM Linux no Azure com o portal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Você também pode aprender mais sobre [práticas comprovadas para executar uma VM Linux no Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).
