---
title: Estender a funcionalidade de modelo do Azure Resource Manager
description: Descreve dicas e truques sobre como estender a funcionalidade de modelo do Azure Resource Manager.
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 725013df3d0551060a9f504da7a97cdc370f956a
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111369"
---
# <a name="extend-azure-resource-manager-template-functionality"></a>Estender a funcionalidade de modelo do Azure Resource Manager

Em 2016, a equipe de práticas e diretrizes da Microsoft criou um conjunto de [blocos de construção de modelo](https://github.com/mspnp/template-building-blocks/wiki) do Azure Resource Manager com o objetivo de simplificar a implantação de recursos. Cada bloco de construção contém um conjunto de modelos predefinidos que implantar conjuntos de recursos especificados por arquivos de parâmetros separados.

Os modelos de bloco de construção são projetados para ser combinados para criar implantações maiores e mais complexas. Por exemplo, a implantação de uma máquina virtual no Azure requer uma rede virtual, contas de armazenamento e outros recursos. O [modelo de bloco de construção de rede virtual](https://github.com/mspnp/template-building-blocks/wiki/VNet-(v1)) implanta uma rede virtual e sub-redes. O [modelo de bloco de construção de máquina virtual](https://github.com/mspnp/template-building-blocks/wiki/Windows-and-Linux-VMs-(v1)) implanta contas de armazenamento, interfaces de rede e as VMs reais. Você pode criar um script ou um modelo para chamar os dois modelos de bloco de construção com seus arquivos de parâmetro correspondente para implantar uma arquitetura completa com uma única operação.

Durante o desenvolvimento de modelos de bloco de construção, p&p projetou vários conceitos para estender a funcionalidade de modelo do Azure Resource Manager. Nesta série, descreveremos vários desses conceitos para que você possa usá-los em seus próprios modelos.

> [!NOTE]
> Esses artigos supõem que você tenha um conhecimento avançado sobre os modelos do Azure Resource Manager.