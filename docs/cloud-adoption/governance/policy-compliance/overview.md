---
title: 'CAF: Como a política de TI corporativa pode se tornar pronta para a nuvem?'
description: Explicação sobre o conceito de política corporativa em relação a governança de nuvem
author: BrianBlanchard
ms.date: 2/8/2019
ms.openlocfilehash: e15f4ef4e6fa9a64ea2bb78cd1df6c5867528383
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900416"
---
<!-- markdownlint-disable MD026 -->

# <a name="caf-how-can-corporate-it-policy-become-cloud-ready"></a>CAF: Como a política de TI corporativa pode se tornar pronta para a nuvem?

A governança de nuvem é o produto de um esforço de adoção contínua ao longo do tempo, como uma transformação verdadeira contínua que não acontece da noite para o dia. A tentativa de entregar a governança de nuvem completa antes de abordar as alterações de política corporativa de chave usando um método rápido agressivo raramente produz os resultados desejados. Em vez disso, é recomendável uma abordagem incremental.

O que é diferente sobre nossa estrutura de adoção da nuvem é o ciclo de compras e o impacto do que como esse ciclo pode habilitar o ciclo de transformação autêntica. Como não é um requisito de aquisição de despesas de capital (CapEx) grande, ps engenheiros podem começar o experimento e a adoção mais cedo. Em culturas mais corporativas, a eliminação da barreira de CapEx para a adoção pode levar a loops de comentários mais rigorosos, crescimento orgânico e execução incremental.

A mudança para a adoção da nuvem exige uma mudança na governança. Em muitas organizações, a transformação de política corporativa permite governança aprimorada e taxas mais altas de conformidade por meio de alterações incrementais de política e a aplicação automática dessas alterações, alimentado por recursos definidos recentemente que você configurar com seu provedor de serviços de nuvem.

Este artigo descreve as principais atividades que podem ajudar você a moldar as políticas da empresa para habilitar um modelo de governança expandido.

## <a name="define-corporate-policy-to-mature-cloud-governance"></a>Defina a política corporativa para governança de nuvem madura

No controle tradicional e governança incremental, a política corporativa cria a definição de trabalho de governança. A maioria das ações de governança de TI busca implementar a tecnologia para monitorar, impor, operar e automatizar essas políticas corporativas. A governança de nuvem é criada em conceitos semelhantes.

![Governança corporativa e disciplinas de governança](../../_images/operational-transformation-govern.png)

*Figura 1. Governança corporativa e disciplinas de governança.*

A imagem acima demonstra as interações entre o risco de negócios, política e conformidade e monitorar e impor para criar uma estratégia de governança. Seguido por cinco disciplinas de Governança de nuvem para realizar sua estratégia.

## <a name="review-existing-policies"></a>Revisão das políticas existentes

Na imagem acima, a estratégia de governança (risco, política e conformidade, monitorar e impor) começa com o reconhecimento dos riscos de negócios. Noções básicas sobre como as alterações do [risco aos negócios](understanding-business-risk.md) é a primeira etapa para criar uma estratégia de governança de nuvem duradoura. Trabalhar com as suas unidades comerciais para obter um medidor [preciso da tolerância do negócio quando a risco](risk-tolerance.md), ajuda você a entender qual nível de riscos precisam ser mitigados. A compreensão dos novos riscos e a tolerância aceitável pode impulsionar uma [revisão de políticas existentes](what-is-a-cloud-policy-review.md) para determinar o nível necessário de governança que apropriado para sua organização.

> [!TIP]
> Se sua organização é regida por conformidade de terceiros, um dos maiores riscos de negócios a considerar pode ser um risco da aderência a [conformidade regulatória](what-is-regulatory-compliance.md). Muitas vezes, esse risco não pode ser atenuado e, em vez disso, pode exigir uma estrita aderência. Certifique-se de entender seus requisitos de conformidade de terceiros antes de iniciar uma revisão da política.

## <a name="an-incremental-approach-to-cloud-governance"></a>Uma abordagem incremental para a governança de nuvem

Uma abordagem incremental para a governança de nuvem pressupõe que é inaceitável para exceder a [tolerância de riscos da empresa](risk-tolerance.md). Em vez disso, pressupõe que a função de governança é acelerar as mudanças dos negócios, ajudar os engenheiros a entender as diretrizes de arquitetura e certificar-se de que os [riscos da empresa](understanding-business-risk.md) são comunicados e mitigados regularmente. Como alternativa, a função tradicional de governança pode se tornar uma barreira para a adoção pelos engenheiros ou pela empresa como um todo.

Com uma abordagem incremental para governança de nuvem, às vezes, há um atrito natural entre as equipes de criação de novas soluções de negócios e proteger a empresa contra riscos. No entanto, neste modelo essas duas equipes podem se tornar pares trabalhando em incrementos ou sprints. Como pares, a equipe de Governança de nuvem e as equipes de adoção de nuvem começam a trabalhar juntar para expor, avaliar e mitigar os riscos comerciais. Esse esforço pode criar uma maneira natural de reduzir o atrito e a criação de colaboração entre equipes.

## <a name="minimum-viable-product-mvp-for-policy"></a>Produto mínimo viável (MVP) para a política

A primeira etapa em uma parceria emergente entre suas equipes de governança e a adoção da nuvem é um acordo sobre a política de MVP. Seu MVP para governança de nuvem deve confirmar que os riscos de negócios são pequenos no início, mas provavelmente aumentarão conforme sua empresa adota mais serviço de nuvem ao longo do temp.

Por exemplo:  Para uma empresa que implanta 5 VMs que não contem nenhum dado e Impacto alto nos negócios (HBI), o risco comercial é pequeno. E vários incrementos posteriormente, quando o número atingir 1.000 VMs e a empresa estiver começando a mover dados HBI, o risco comercial aumenta.

A política MVP é uma tentativa de definir uma base obrigatória para as políticas necessárias implantarem as primeiras VMs “x”’ ou o primeiro x número de aplicativos. Onde x é uma quantidade pequena, porém com impacto das unidades sendo adotado. Este conjunto de políticas requer algumas restrições, mas poderá conter os aspectos fundamentais necessários para crescer rapidamente a partir de um incremento de trabalho para o próximo. Durante o desenvolvimento incremental de política, essa estratégia de governança tende a aumentar ao longo do tempo. Por meio de turnos sutis lentos, MVP de política tende a aumentar em paridade de recursos com as saídas do exercício de análise de política.

## <a name="incremental-policy-growth"></a>Crescimento incremental de política

O crescimento incremental de política é o mecanismo principal para aumentar a política e a governança de nuvem cada vez mais. Também é o principal requisito para a adoção de um modelo incremental para governança. Para este modelo funcionar bem, a equipe de governança deve ser confirmada para uma alocação em andamento de tempo em cada sprint, para avaliar e implementar a alteração das disciplinas de governança.

**Requisitos de tempo de Sprint:** No início de cada iteração, cada equipe de adoção da nuvem cria uma lista de ativos a serem migrados ou adotados no incremento atual. A equipe de governança de nuvem deve permitir tempo suficiente para revisar a lista, validar as classificações para ativos de dados, avaliar quaisquer novos riscos associados a cada ativo, diretrizes de arquitetura de atualização e treinar a equipe a respeito das alterações. Esses compromissos normalmente exigem 10 a 30 horas por sprint. Também é esperado para esse nível de envolvimento, exigir pelo menos um funcionário dedicado gerenciar o controle em um esforço de adoção de nuvem grande.

**Requisitos de tempo de lançamento:** No início de cada versão, as equipes de adoção da nuvem e a equipe de estratégia de nuvem devem priorizar uma lista de aplicativos ou cargas de trabalho a serem migrados na iteração atual, juntamente com qualquer atividade de mudança comercial. Esses pontos de dados permitem que a equipe de governança de nuvem entenda os novos riscos de negócios antecipadamente. Isso permite o tempo para se alinhar aos negócios e medir a tolerância do negócio de riscos.