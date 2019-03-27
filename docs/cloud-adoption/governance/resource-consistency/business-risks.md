---
title: 'CAF: Motivações de Consistência de Recursos e os riscos de negócios'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Motivações de Consistência de Recursos e os riscos de negócios
author: alexbuckgit
ms.openlocfilehash: 19e0d761e4afa3473099bde2edc960c8b9eadb79
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242457"
---
# <a name="resource-consistency-motivations-and-business-risks"></a>Motivações de Consistência de Recursos e os riscos de negócios

Este artigo aborda os motivos que os clientes normalmente adotam uma disciplina de Consistência de Recursos dentro de uma estratégia de governança de nuvem. Também fornece alguns exemplos de riscos de negócios potenciais que podem conduzir as instruções de política.

<!-- markdownlint-disable MD026 -->

## <a name="is-resource-consistency-relevant"></a>A Consistência de Recursos é relevante?

Quando se trata de implantação de recursos e cargas de trabalho, a nuvem oferece maior agilidade e flexibilidade em datacenters mais tradicionais no local. No entanto, essas vantagens potenciais baseadas em nuvem também são fornecidas emparelhadas com possíveis desvantagens de gerenciamento que possam prejudicar seriamente o sucesso da sua adoção da nuvem. Quais ativos você implantou? Quais equipes possuem quais ativos? Você tem recursos suficientes que dão suporte a uma carga de trabalho? Como saber se as cargas de trabalho estão íntegras?

A Consistência de Recursos é crucial para garantir que os recursos sejam implantados, atualizados e configurados de forma consistente e repetidamente, e que interrupções de serviço sejam minimizadas e corrigidas em menor tempo possível.

A disciplina de Consistência de Recursos está preocupada com a identificação e mitigação de riscos de negócios relacionados aos aspectos operacionais da sua implantação de nuvem. A Consistência de Recursos inclui monitoramento de aplicativos, cargas de trabalho e desempenho de ativos. Ela também inclui as tarefas necessárias para atender às demandas de dimensionamento, corrigir violações de desempenho de SLA (Contrato de Nível de Serviço) e evitar proativamente as violações de SLA de desempenho por meio da correção automatizada.

Implantações de teste inicial podem não exigir muito além de adotar alguns padrões de nomenclatura e marcação para dar suporte às necessidades de Consistência de Recursos. À medida que sua adoção da nuvem amadurece e você implanta mais ativos críticos e complicados, a necessidade de investir na disciplina de consistência de recurso aumenta rapidamente.

## <a name="business-risk"></a>Riscos de negócios

A disciplina de Consistência de Recursos tenta riscos de negócios operacionais de núcleo do endereço. Trabalhar com seus negócios e equipes de TI para identificar esses riscos e monitorar cada um deles para relevância como planejar e implementar suas implantações de nuvem.

Os riscos serão diferentes entre as organizações, mas a seguir servem como riscos comuns que você pode usar como ponto de partida para discussões dentro de sua equipe de governança de nuvem:

- **Custo operacional desnecessário**. Recursos obsoletos ou não utilizados ou recursos que são sobreprovisionados durante horários de baixa demanda, adicione custos operacionais desnecessários.
- **Recursos não provisionados**. Recursos que experimentam uma demanda maior que a antecipada podem resultar em interrupção dos negócios como recursos de nuvem sobrecarregados por demanda.
- **Ineficiências de gerenciamento**. Falta de nomenclatura consistente e metadados associados aos recursos de marcação pode levar a equipe de TI a ter dificuldades para localizar recursos para tarefas de gerenciamento ou de identificação de propriedade e informações de estatísticas relacionadas a ativos. Isso resulta em ineficiências de gerenciamento que podem aumentar o custo e a capacidade de resposta lenta da TI a interrupção do serviço ou outros problemas operacionais.
- **Interrupção dos negócios**. Interrupções de serviço que resultam em violações dos Contratos e Nível de Serviço (SLAs) estabelecidos da sua organização podem resultar em perda de negócios ou outros impactos financeiros a sua empresa.

## <a name="next-steps"></a>Próximas etapas

Use o [Modelo de Gerenciamento de Nuvem](./template.md), documente os riscos de negócios que podem ser introduzidos pelo plano atual de adoção de nuvem.

Quando um entendimento dos riscos de negócios realista é estabelecido, a próxima etapa é documentar a tolerância do negócio quanto a risco e os indicadores e métricas de chave para monitorar essa tolerância.

> [!div class="nextstepaction"]
> [Entenda os indicadores, métricas e tolerância a risco](./metrics-tolerance.md)
