---
title: 'CAF: O que é a governança dos recursos da nuvem?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.date: 02/11/2019
description: Governança de recursos de nuvem de explicação no Azure
author: petertaylor9999
ms.openlocfilehash: 602ac81f2b2201c77746df971d282582ceee23f7
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503189"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-cloud-resource-governance"></a>O que é a governança dos recursos da nuvem?

Em [Como funciona o Azure?](what-is-azure.md), você aprendeu que o Azure é uma coleção de servidores e hardware de rede que executa software e hardware virtualizado em nome dos usuários. Azure permite o desenvolvimento de sua organização e os departamentos de TI para ser ágil, tornando mais fácil criar, ler, atualizar e excluir recursos conforme necessário.

No entanto, enquanto irrestrito fornecendo acesso a recursos para desenvolvedores pode torná-los muito ágil, também poderá causar consequências não intencionais de custo. Por exemplo, uma equipe de desenvolvimento pode ser aprovada para implantar um conjunto de recursos para teste, mas esquecer de excluí-las quando o teste for concluído. Esses recursos continuarão a gerar custos, embora seu uso não seja mais necessário ou aprovado.

A solução para esse problema é a **governança** de acesso a recursos. Governança refere-se ao processo contínuo de gerenciamento, monitoramento e auditoria, o uso de recursos do Azure para atender as metas e requisitos da sua organização.

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94]

<!-- markdownlint-enable MD034 -->

Essas metas e requisitos são exclusivos de cada organização e, portanto, não é possível ter uma abordagem única para governança. Em vez disso, o Azure implementa duas ferramentas de governança primária, **RBAC (controle de acesso baseado em função)** e **política de recursos**, e depende de cada organização criar seu modelo de governança usando essas ferramentas.

O RBAC define funções, e as funções definem os recursos para um usuário atribuído à função. Por exemplo, a função **proprietário** habilita todos os recursos (criar, ler, atualizar e excluir) para um recurso, enquanto as funções **leitor** habilita somente a capacidade de leitura. As funções podem ser definidas com um escopo amplo que se aplique a vários tipos de recursos ou um escopo estreito que se aplique a alguns.

As políticas de recursos definem regras para a criação de recursos. Por exemplo, uma política de recursos pode limitar o SKU de uma VM para um tamanho pré-aprovado específico. Ou então, uma política de recursos pode impor a adição de uma marca com um centro de custo quando a solicitação é feita para criar o recurso.

Ao configurar essas ferramentas, uma consideração importante é o balanceamento da governança versus agilidade organizacional. Ou seja, quanto mais restritiva a política de governança, menos ágeis se tornarão seus desenvolvedores e profissionais de TI. Isso ocorre porque uma política de governança restritiva pode exigir mais etapas manuais, como exigir que um desenvolvedor preencher um formulário ou envie um email para uma pessoa da equipe de governança para criar manualmente um recurso. A equipe de governança tem recursos finitos e pode ficar acumulada, resultando em equipes de desenvolvimento improdutivos aguardando seus recursos para ser criados e desnecessários recursos acumular custos enquanto esperam a ser excluído.

## <a name="next-steps"></a>Próximas etapas

Agora que você entende o conceito de governança de recursos de nuvem, saiba mais sobre como o acesso aos recursos é gerenciado no Azure.

> [!div class="nextstepaction"]
> [Saiba mais sobre o acesso de recursos no Azure](azure-resource-access.md)
