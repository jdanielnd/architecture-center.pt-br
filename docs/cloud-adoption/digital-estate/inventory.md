---
title: 'CAF: Coletar dados de inventário para uma propriedade digital'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
description: Como coletar dados de inventário para um estado digital.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 0c68ff1e5ff51395698d37fb9b59c7a76c9479b7
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242547"
---
# <a name="gather-inventory-data-for-a-digital-estate"></a>Coletar dados de inventário para uma propriedade digital

Desenvolver um inventário é a primeira etapa no [Planejamento de propriedade digital](overview.md). Nesse processo, uma lista de ativos de TI que dão suporte a funções de negócios específicas seria coletada para análise e racionalização posterior. Este artigo pressupõe que uma abordagem de baixo para cima à análise é mais apropriada para necessidades de planejamento. Para saber mais, confira [Abordagens para planejamento de propriedade digital](./approach.md).

## <a name="take-inventory-of-a-digital-estate"></a>Faça o inventário de um espaço digital

O inventário que dá suporte a uma propriedade digital muda dependendo da Transformação Digital desejada e a Jornada de Transformação correspondente.

- **Transformação operacional**. Durante uma transformação operacional, geralmente aconselhamos a coleta do inventário de ferramentas de exame que possam criar uma lista centralizada de todas as VMs e servidores. Algumas ferramentas também podem criar mapeamentos de rede e dependências, o que ajudará a definir o alinhamento da carga de trabalho.

- **Transformação incremental.** O inventário para uma transformação incremental começa com o cliente. O mapeamento da experiência do cliente do início ao fim é um bom lugar para começar. O alinhamento mapeado para aplicativos, APIs, dados e outros ativos criará um inventário detalhado para análise.

- **Transformação Interruptiva.** A transformação interruptiva se concentra no produto ou serviço. A partir daí, um inventário incluiria um mapeamento das oportunidades de interrupção do mercado e dos recursos necessários.

## <a name="accuracy-and-completeness-of-an-inventory"></a>Precisão e integridade de um inventário

Raramente, um inventário fica totalmente concluído em sua primeira iteração. É altamente recomendável que vários membros da equipe de Estratégia de Nuvem se alinhem aos participantes e usuários avançados para validar o inventário. Quando possível, outras ferramentas, como a análise de rede e de dependência, podem ser usadas para identificar os ativos que estão recebendo tráfego, mas que não estão no inventário.

## <a name="next-steps"></a>Próximas etapas

Após a compilação e validação de um inventário, ele pode ser racionalizado. Racionalização de Inventário é a próxima etapa para planejamento de propriedade digital.

> [!div class="nextstepaction"]
> [Racionalizar a propriedade digital](rationalize.md)