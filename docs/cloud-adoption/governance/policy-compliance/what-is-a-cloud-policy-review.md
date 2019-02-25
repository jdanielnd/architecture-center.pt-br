---
title: 'CAF: O que é uma revisão de política de nuvem?'
description: O que é revisão de política de nuvem?
author: BrianBlanchard
ms.date: 2/8/2019
ms.openlocfilehash: b879f261e16ffc72180417e2e0f533e2eaa435ba
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900324"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-a-cloud-policy-review"></a>O que é uma revisão de política de nuvem?

Um **revisão da política de nuvem** é a primeira etapa em direção à [maturidade de governança](../overview.md) na nuvem. O objetivo desse processo é modernizar as políticas corporativas de TI existentes. Quando concluído, as políticas atualizadas fornecem um nível equivalente de mitigação de risco para recursos baseados em nuvem. Este artigo explica o processo de revisão da política de nuvem e sua importância.

## <a name="why-perform-a-cloud-policy-review"></a>Por que realizar uma revisão de política de nuvem?

A maioria das empresas gerenciam a TI através da execução de processos em alinhamento às políticas em vigor. Em empresas de pequeno porta, essas políticas podem ser anedóticas e os processos vagamente definidos. À medida que as empresas crescem para grandes empresas, as políticas e processos tendem a ser documentados com mais clareza e executadas de forma consistente.

Conforme as empresas amadurecem as políticas corporativas de TI, as dependências nas últimas decisões técnicas têm uma tendência a se infiltrar nas políticas em vigor. Por exemplo, é comum ver que os processos de recuperação de desastres incluem a política que exige backups em fitas externas. Essa inclusão pressupõe uma dependência em um tipo de tecnologia (backups em fita), que pode não ser mais a solução mais relevante.

Transformações em nuvem criam um ponto de inflexão natural para reconsiderar as decisões de política herdada do passado. Recursos técnicos e processos padrão consideravelmente alteram na nuvem, como fazem os riscos herdados. Aproveitando o exemplo anterior, a política de backup em fita ramificada do risco de um ponto único de falha, mantendo os dados em um único local e a empresa precisar minimizar o perfil de risco, reduzindo esse risco. Em uma implantação de nuvem, há várias opções que fornecem a mesma mitigação de riscos, com muito muitos objetivos de tempo de recuperação inferiores (RTO). Por exemplo:

- Uma solução nativa da nuvem pode habilitar a replicação geográfica do Banco de Dados do SQL Azure
- Uma solução híbrida pode aproveitar o Azure Site Recovery (ASR) para replicar uma carga de trabalho de IaaS para vários datacenters

Ao executar uma transformação de nuvem, as políticas geralmente controlam muitas das ferramentas, serviços e processos disponíveis para as equipes de adoção da nuvem. Se essas políticas se baseiam em tecnologias herdadas, elas podem atrapalhar os esforços da equipe para conduzir mudanças. Na pior das hipóteses, políticas importantes são inteiramente ignoradas pela equipe de migração para habilitar soluções alternativas. Nenhum deles é um resultado aceitável.

## <a name="the-cloud-policy-review-process"></a>Processo de revisão da política de nuvem

Revisões da política de nuvem alinham a governança de TI existente e políticas de segurança às [cinco disciplinas de Governança de Nuvem](../overview.md): [Gerenciamento de Custos](../cost-management/overview.md), [Linhas de base de segurança](../security-baseline/overview.md), [Linha de base de identidade](../identity-baseline/overview.md), [Consistência de recursos](../resource-consistency/overview.md), e [Aceleração de Implantação](../deployment-acceleration/overview.md).

Para cada uma dessas disciplinas, o processo de revisão segue essas etapas:

1. Políticas de revisão local existentes relacionadas à disciplina específica, procurando dois pontos de dados principais: dependências de legado e riscos comerciais identificados.
2. Avalie cada risco aos negócios fazendo uma pergunta simples: "O risco de negócios ainda existe em um modelo de nuvem?"
3. Se ainda existe o risco, grave novamente a política ao documentar a atenuação necessária e não a solução técnica.
4. Examine a política atualizada com as equipes de adoção da nuvem para entender as soluções possíveis para a atenuação necessária.

## <a name="example-of-a-policy-review-for-a-legacy-policy"></a>Exemplo de uma revisão de política para uma política de legado

Para fornecer um exemplo do processo, vamos usar novamente a política de backup em fita da seção anterior:

- Uma política corporativa que rege a maneira de fazer backups em fita para todos os sistemas de produção. Nessa política, você pode ver dois pontos de dados de interesse:
  - Dependência herdada em uma solução de backup em fita
  - Um risco de negócios assumido associado ao armazenamento de backups no mesmo local físico que o equipamento de produção.
- O risco ainda existe? Sim. Até mesmo na nuvem, uma dependência em um único recurso cria algum risco. Há uma probabilidade menor desse risco afetar o negócio que estava presente na solução no local, mas ainda existe o risco.
- Regravação da política. Em caso de desastre de todo o datacenter, deve existir uma maneira de restaurar os sistemas de produção dentro de 24 horas de interrupção em um datacenter diferente e um local geográfico diferente.
- Examine com as equipes de adoção da nuvem. Dependendo da solução que está sendo implementada, há vários meios de aderir a esta política de Consistência de Recursos.

## <a name="tools-to-help-create-modern-policies"></a>Ferramentas para ajudar a criar políticas modernas

Para ajudar a acelerar a criação de políticas modernas, um conjunto de políticas de exemplo está disponível em cada uma das cinco disciplinas de governança de nuvem. Essas políticas de exemplo serão iniciadas com uma das três suposições de design:

- **Nativo de nuvem**: A solução que está sendo implantada é nativa de nuvem e pode aproveitar a soluções de padrão encontradas no Azure, com configuração mínima.
- **Enterprise**: A solução que está sendo implantada é complexa e exige um modelo de implantação de nuvem híbrida. Isso necessita de implementações mais complexas de determinadas disciplinas de governança.
- **Princípio de design (CDP) em conformidade de nuvem**: A solução que está sendo implantada deve aderir aos eixos de arquitetura definidos no CDP, exigindo um grau muito maior de controle.  

Para cada disciplina, uma política de exemplo precisa ser criada em cada um desses níveis. Cada exemplo destina-se a disparar pensamentos e conversas dentro do ambiente corporativo. Observe que esses exemplos não se destinam a ser usado como uma alternativa para uma política de TI corporativa construída de maneira adequada.
