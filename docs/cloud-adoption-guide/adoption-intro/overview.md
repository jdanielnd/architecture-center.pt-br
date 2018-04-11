---
title: 'Adotando o Azure: conceitos básicos'
description: Descreve o nível de linha de base de conhecimento que uma empresa precisa para adotar o Azure
author: petertay
ms.openlocfilehash: e9421b610e4eb07a3ed37bca56e513b0689484ef
ms.sourcegitcommit: 9ba82cf84cee06ccba398ec04c51dab0e1ca8974
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/13/2018
---
# <a name="adopting-azure-foundational"></a>Adotando o Azure: conceitos básicos

A adoção do Azure é o primeiro estágio de maturidade organizacional de uma empresa. Ao final deste estágio, as pessoas de sua organização poderão implantar cargas de trabalho simples no Azure.

A lista abaixo inclui as tarefas para concluir o estágio básico de adoção. A lista é progressiva e, portanto, conclua cada tarefa na ordem. Se você concluiu a tarefa anteriormente, passe para a próxima tarefa na lista. 

1. Conheça mais sobre os recursos internos do Azure:
    - **Explicador:** [como funciona o Azure?](azure-explainer.md)
2. Conheça mais sobre a identidade digital da empresa no Azure:
    - **Explicador:** [o que é um locatário do Azure Active Directory?](tenant-explainer.md)
    - **Como** [obter um locatário do Azure Active Directory](/azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Diretrizes:** [diretrizes de design de locatário do Azure AD](tenant.md)
    - **Como** [adicionar novos usuários ao Azure Active Directory](/azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json)    
3. Conheça mais sobre as assinaturas no Azure:
    - **Explicador:** [o que é uma assinatura do Azure?](subscription-explainer.md)
    - **Diretrizes:** [design de assinatura do Azure](subscription.md)
4. Conheça mais sobre o gerenciamento de recursos no Azure: 
    - **Explicador:** [o que é o Azure Resource Manager?](resource-manager-explainer.md)
    - **Explicador:** [o que é um grupo de recursos do Azure?](resource-group-explainer.md)
    - **Explicador:** [noções básicas sobre o acesso aos recursos no Azure](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Como** [criar um grupo de recursos do Azure usando o portal do Azure](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Diretrizes:** [diretrizes de design de grupo de recursos do Azure](resource-group.md)
    - **Diretrizes:** [convenções de nomenclatura para recursos do Azure](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)
5. Implante uma arquitetura básica do Azure:
    - Saiba mais sobre os diferentes tipos de opções de computação do Azure, como IaaS (infraestrutura como serviço) e PaaS (plataforma como serviço) em [visão geral das opções de computação do Azure](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json).
    - Agora que você entendeu os diferentes tipos de opções de computação do Azure, escolha um aplicativo Web de PaaS ou uma máquina virtual de IaaS como seu primeiro recurso no Azure:
    - PaaS: introdução à Plataforma como Serviço:
        - **Como** [implantar um aplicativo Web básico no Azure](/azure/app-service/app-service-web-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Diretrizes:** melhores práticas comprovadas para implantar um [aplicativo Web básico](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) no Azure
    - IaaS: introdução à Rede Virtual:
        - **Explicador:** [rede virtual do Azure](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Como** [implantar uma Rede Virtual no Azure usando o portal](/azure/virtual-network/virtual-networks-create-vnet-arm-pportal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - IasS: implantar uma carga de trabalho de VM (máquina virtual) única (Windows e Linux):
        - **Como** [implantar uma VM Windows no Azure com o portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Diretrizes:** [práticas comprovadas para executar uma VM Windows no Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Como** [implantar uma VM Linux no Azure com o portal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Diretrizes:** [práticas comprovadas para executar uma VM Linux no Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
