---
title: 'CAF: Grandes empresas – várias camadas de governança em grandes empresas'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Grandes empresas – várias camadas de governança em grandes empresas
author: BrianBlanchard
ms.openlocfilehash: 1a90f8007077df0ecefa8ec5d8c0dd6bfca9ccc7
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59641205"
---
# <a name="multiple-layers-of-governance-in-large-enterprises"></a>Várias camadas de governança em empresas de grande porte

Quando empresas de grande porte necessitam de várias camadas de governança, existem níveis mais altos de complexidade que deve ser fatorados na governança MVP e posteriores evoluções de governança.

Alguns exemplos comuns de tais complexidades incluem:

- Funções de governança distribuída.
- IT Corporativa com suporte às organizações de TI da unidade comercial.
- Suporte de TI corporativa que dá suporte geograficamente distribuído em organizações de TI distribuídas.

Este artigo explora algumas maneiras de navegar por esse tipo de complexidade.

## <a name="large-enterprise-governance-is-a-team-sport"></a>A governança de empresas de grande porte é um esporte em equipe

Grandes empresas estabelecidas geralmente têm equipes ou funcionários que se concentram nas disciplinas mencionadas nessa jornada. Nessa jornada demonstra uma abordagem para tornar a governança em um esporte de equipe.

Em muitos grandes empresas, disciplinas cinco de governança de nuvem pode ser impedimentos à adoção. Desenvolver competências de nuvem na identidade, segurança, operações, implantações e configuração em toda a empresa leva tempo. Implementar holisticamente a política de governança de TI e segurança de TI pode diminuir a inovação por meses ou até mesmo anos. Equilibrar a necessidade de negócios para inovar e a necessidade de governança para proteger os recursos existentes é delicada.

Os recursos inerentes da nuvem podem remover bloqueadores de inovação, mas aumentar os riscos. Nessa jornada de governança, mostramos como a empresa de exemplo criou as grades de proteção para atenuar o risco. Em vez de lidar com cada uma das disciplinas necessárias para proteger o ambiente, a equipe de governança de nuvem leva uma abordagem baseada em riscos para determinar o que pode ser implantado, enquanto as outras equipes criam as maturidades necessárias para a nuvem. Mais importante, à medida que cada equipe atinge a maturidade de nuvem, a governança aplica suas soluções de forma global. Como cada equipe amadurece e o adiciona à solução geral, a equipe de governança de nuvem pode abrir os portões de estágio, permitindo que a inovação e adoção prosperem.

Esse modelo ilustra o crescimento de uma parceria entre a equipe de governança de nuvem e equipes empresariais existentes (segurança, governança de TI, rede, identidade e outros). A jornada começa com a governança MVP e aumenta a um estado final holístico por meio de evoluções de governança.

## <a name="requirements-to-supporting-such-a-team-sport"></a>Requisitos para dar suporte a uma equipe de esporte

É o primeiro requisito de um modelo de governança de várias camadas para entender a hierarquia de governança. Responder às perguntas a seguir ajudarão você a compreender a hierarquia de controle geral:

- Como a contabilização de nuvem (cobrança dos serviços de nuvem) é alocada entre as unidades de negócios?
- Como as responsabilidades de governança são alocadas na TI corporativa e em cada unidade de negócios?
- Que tipos de ambientes fazem cada uma dessas unidades de IT gerenciar?

## <a name="central-governance-of-a-distributed-governance-hierarchy"></a>A governança central de uma hierarquia de governança distribuída

Ferramentas, como grupos de gerenciamento, permitem que a TI corporativa crie uma estrutura de hierarquia que corresponda à hierarquia de governança. Ferramentas como o Azure Blueprints podem aplicar ativos a camadas diferentes dessa hierarquia. Especificações técnicas do Azure Blueprints podem ser atualizadas e várias versões podem ser aplicadas a grupos de gerenciamento, assinaturas ou grupos de recursos. Cada um desses conceitos é descrito mais detalhadamente nas seções a seguir.

O aspecto importante de cada uma dessas ferramentas é a capacidade de aplicar vários planos de gráficos para uma hierarquia. Isso permite que o controle seja um processo em camadas. A seguir, um exemplo dessa aplicação hierárquica de governança:

- TI Empresarial: A IT empresarial cria um conjunto de padrões e políticas que se aplicam a adoção da nuvem. Isso é materializado em um blueprint de "Linha de base". A TI corporativa possui, em seguida, a hierarquia de grupo de gerenciamento, garantindo que uma versão da linha de base é aplicada a todas as assinaturas na hierarquia.
- IT regional ou de unidade de negócios: Várias equipes de TI podem aplicar uma camada adicional de governança, criando suas próprias especificações técnicas. Esses blueprints criaram políticas e padrões aditivos. Depois que desenvolvido, a TI corporativa pode aplicar esses planos gráficos para os nós aplicáveis na hierarquia de grupo de gerenciamento.
- Equipes de Adoção da Nuvem: Decisões detalhadas e implementação de aplicativos ou cargas de trabalho podem ser feitas pelas equipes de adoção da nuvem, dentro do contexto de requisitos de governança. Às vezes, a equipe também pode solicitar outros modelos de consistência de recursos do Azure para acelerar os esforços de adoção.

Obtenha detalhes sobre a implementação da governança em cada nível exigirá a coordenação entre cada equipe. O MVP de governança e evoluções de governança descritas nessa jornada pode ajudar a alinhar essa coordenação.
