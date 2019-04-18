---
title: 'CAF: O que é a governança dos recursos da nuvem?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.date: 02/11/2019
description: Governança de recursos de nuvem de explicação no Azure
author: petertaylor9999
ms.openlocfilehash: 0989a5aad8a6290cce07fd71771c690bbd615e0d
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59068847"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-cloud-resource-governance"></a>O que é a governança dos recursos da nuvem?

Na [como funciona o Azure?](what-is-azure.md), você aprendeu que o Azure é uma coleção de servidores e hardware que executa o software e hardware virtualizado em nome dos usuários de rede. O Azure permite o desenvolvimento de aplicativos da sua organização e aos departamentos de TI seja ágil, tornando mais fácil criar, ler, atualizar e excluir recursos conforme necessário.

No entanto, enquanto o acesso irrestrito aos recursos pode tornar os desenvolvedores muito ágil, ele também pode levar a custos inesperados. Por exemplo, uma equipe de desenvolvimento pode ser aprovada para implantar um conjunto de recursos para teste, mas esquecer de excluí-las quando o teste for concluído. Esses recursos continuarão a acumular custos, mesmo que eles não são mais necessários ou aprovado.

A solução é o controle de acesso de recursos. **Governança** é o processo contínuo de gerenciamento, monitoramento e auditoria do uso de recursos do Azure para atender aos requisitos da sua organização.

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94]

<!-- markdownlint-enable MD034 -->

Esses requisitos são exclusivos de cada organização, portanto, uma abordagem única para a governança não é útil. Em vez disso, cabe a cada organização para criar seu modelo de governança usando duas ferramentas de governança primária do Azure: **controle de acesso baseado em função (RBAC)** e **política de recurso**.

Define as funções RBAC e as funções definem os recursos de cada usuário atribuídos a essa função. Por exemplo, o **proprietário** função permite que todos os recursos (criar, ler, atualizar e excluir) para um recurso, enquanto o **leitor** função permite que apenas a capacidade de leitura. As funções podem ser definidas com um amplo escopo que se aplica a muitos tipos de recurso ou um escopo estreito que se aplica a alguns.

As políticas de recursos definem regras para a criação de recursos. Por exemplo, uma política de recurso pode limitar a SKU de uma máquina virtual com um determinado tamanho previamente aprovada. Outra política de recursos pode impor a aplicação de uma marca para um centro de custo atribuído quando a solicitação é feita para criar o recurso.

Ao configurar essas ferramentas, é importante equilibrar a governança e a agilidade organizacional. A configuração mais restritiva sua diretiva de governança, menos agile seus desenvolvedores e profissionais de TI será. Uma política de governança restritiva pode exigir mais etapas manuais, como exigir que um desenvolvedor preencher um formulário ou envie um email para um membro da equipe de governança para criar manualmente um recurso. A equipe de governança tem capacidade finita e pode se tornar um gargalo, resultando em equipes de desenvolvimento aguardando unproductively seus recursos para ser criados ou desnecessários recursos acumular custos antes de serem excluídos.

## <a name="next-steps"></a>Próximos passos

Agora que você entende o conceito de governança de recursos de nuvem, saiba mais sobre como o acesso aos recursos é gerenciado no Azure.

> [!div class="nextstepaction"]
> [Saiba mais sobre o acesso de recursos no Azure](azure-resource-access.md)
