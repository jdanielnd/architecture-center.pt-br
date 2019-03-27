---
title: 'CAF: Motivações e riscos de negócios que direcionam a aceleração da implantação'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Saiba mais sobre a disciplina de Aceleração de implantação como parte de uma estratégia de governança de nuvem.
author: alexbuckgit
ms.openlocfilehash: 854b1fd420de605a665922e9b207e6aecbfab2f0
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242137"
---
# <a name="deployment-acceleration-motivations-and-business-risks"></a>Motivações de aceleração de implantação e os riscos de negócios

Este artigo aborda os motivos que os clientes normalmente adotam uma disciplina de Aceleração de implantação dentro de uma estratégia de governança de nuvem. Também fornece alguns exemplos de riscos de negócios que conduzem as instruções de política.

<!-- markdownlint-disable MD026 -->

## <a name="is-deployment-acceleration-relevant"></a>A aceleração de implantação é relevante?

Em sistemas locais normalmente são implantados usando imagens de linha de base ou scripts de instalação. A configuração adicional é geralmente necessária, o que pode envolver várias etapas ou intervenção humana. Esses processos manuais são propensos a erro e geralmente resultam em "descompassos de configuração", que exigem tarefas demoradas de solução de problemas e correção.

A maioria dos recursos do Azure pode ser implantada e configurada manualmente por meio do portal do Azure. Esta abordagem pode ser suficiente para suas necessidades quando houver apenas alguns recursos para gerenciar. No entanto, à medida que cresce o seu acervo de nuvem, sua organização deve automatizar a implantação de seus recursos de nuvem para usufruir dos recursos de recuperação que o Azure fornece do dimensionamento, failover e desastres. Adotar uma abordagem de DevOps ou DevSecOps costuma ser a melhor maneira de gerenciar suas implantações.

Um plano de aceleração de implantação robusto garante que seus recursos de nuvem sejam implantados, atualizados, configurados corretamente e consistentemente e que permaneçam assim. A maturidade da sua estratégia de aceleração de implantação também pode ser um fator importante na sua [estratégia de Gerenciamento de Custos](../cost-management/overview.md). O provisionamento automatizado e a configuração de seus recursos de nuvem permitem que você reduza ou desaloque os recursos quando a demanda estiver baixa ou limite de tempo, portanto, você paga apenas pelos recursos conforme necessário.

## <a name="business-risk"></a>Riscos de negócios

A disciplina de Aceleração de implantação tenta abordar os seguintes riscos de negócios. Durante a adoção da nuvem, monitore cada uma das seguintes opções de relevância:

- **Interrupção de serviço**. Falta de processos de implantação repetíveis previsíveis ou mudanças inalteradas para configurações do sistema podem interromper as operações normais e podem resultar em perda de produtividade ou perda de negócios.
- **Saturações de custos**. Alterações inesperadas na configuração de recursos do sistema podem tornar mais difícil a identificação da causa raiz, elevar os custo de desenvolvimento, operações e manutenção.
- **Ineficiências organizacionais**. Barreiras entre o desenvolvimento, operações e as equipes de segurança podem causar vários desafios para a adoção efetiva das tecnologias de nuvem e o desenvolvimento de um modelo de governança de nuvem unificada.

## <a name="next-steps"></a>Próximas etapas

Use o [Modelo de Gerenciamento de Nuvem](./template.md), documente os riscos de negócios que podem ser introduzidos pelo plano atual de adoção de nuvem.

Quando um entendimento dos riscos de negócios realista é estabelecido, a próxima etapa é documentar a tolerância do negócio quanto a risco e os indicadores e métricas de chave para monitorar essa tolerância.

> [!div class="nextstepaction"]
> [Métricas, indicadores e tolerância a risco](./metrics-tolerance.md)
