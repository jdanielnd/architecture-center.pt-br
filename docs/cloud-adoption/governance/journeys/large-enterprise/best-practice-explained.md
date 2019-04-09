---
title: 'CAF: Grandes empresas – práticas recomendadas explicada'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Grandes empresas – práticas recomendadas explicada
author: BrianBlanchard
ms.openlocfilehash: 2d52797f1c3541fab1c97d97d0438210d2e66f79
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/08/2019
ms.locfileid: "59068983"
---
# <a name="large-enterprise-best-practice-explained"></a>Empresas de grande porte: Melhor prática explicada

A jornada de governança começa com um conjunto de [políticas corporativas](./initial-corporate-policy.md) iniciais. Essas políticas são usadas para estabelecer um produto viável mínimo (MVP) para governança que reflita as [práticas recomendadas](./overview.md).

Neste artigo, discutiremos as estratégias de alto nível que são necessárias para a criação de um MVP de governança. O ponto central do MVP de governança é a disciplina [Aceleração da Implantação](../../deployment-acceleration/overview.md). As ferramentas e os padrões aplicados neste estágio permitirão as evoluções incrementais necessárias para expandir a governança no futuro.

## <a name="governance-mvp-cloud-adoption-foundation"></a>MVP de governança (fundamentos para adoção da nuvem)

A adoção rápida da governança e a política corporativa é uma realidade graças a alguns princípios simples e ferramentas de governança baseadas em nuvem. Essas são as primeiras das três Disciplinas de governança a serem abordadas em qualquer processo de governança. Nos aprofundaremos em cada uma delas neste artigo.

Para estabelecer o ponto de partida, este artigo discutirá as estratégias de alto nível por trás da Linha de Base de Identidade, Linha de Base de Segurança e Aceleração de Implantação que são necessárias para criar um MVP de governança e que servirão como a base para a adoção como um todo.

![Exemplo de MVP de governança incremental](../../../_images/governance/governance-mvp.png)

## <a name="implementation-process"></a>Processo de implementação

A implementação do MVP de governança possui dependências de Identidade, Segurança e Rede. Depois de resolver as dependências, a equipe de Governança de Nuvem decidirá sobre alguns aspectos de governança. As decisões da equipe de Governança de Nuvem e das equipes de suporte serão implementadas por meio de um único pacote de ativos de imposição.

![Exemplo de MVP de governança incremental](../../../_images/governance/governance-mvp-implementation-flow.png)

Essa implementação também pode ser descrita usando uma lista de verificação simples:

1. Solicite decisões relacionadas às dependências básicas: Identidade, Rede e Criptografia.
2. Determine o padrão a ser usado durante a imposição da política corporativa.
3. Determine os padrões de governança apropriado para o recurso de consistência, marcação de recursos e disciplinas de registro em log e relatórios.
4. Implemente as ferramentas de governança alinhadas ao padrão de imposição de política escolhido para aplicar as decisões dependentes e as decisões de governança.

[!INCLUDE [implementation-process](../../../../../includes/cloud-adoption/governance/implementation-process.md)]

## <a name="application-of-governance-defined-patterns"></a>Aplicação de padrões definidos por governança

A equipe de Governança de Nuvem será responsável pelas seguintes decisões e implementações. Muitas equipes precisarão de informações de outras equipes, mas a equipe de Governança de Nuvem provavelmente será a responsável pela decisão e pela implementação. As seções a seguir descrevem as decisões tomadas para esse caso de uso e os detalhes de cada decisão.

### <a name="subscription-model"></a>Modelo de assinatura

O padrão **Misto** foi escolhido para assinaturas do Azure.

- Conforme surgem novas solicitações para recursos do Azure, um "Departamento" deve ser estabelecido para cada unidade de negócios importantes em cada geografia operacional. Dentro de cada um dos Departamentos, "Assinaturas" devem ser criadas para cada arquétipo do aplicativo.
- Um arquétipo de aplicativo é uma maneira de agrupar aplicativos com necessidades semelhantes. Exemplos comuns incluem: Aplicativos com dados protegidos, aplicativos regulamentados (como HIPAA ou FedRAMP), aplicativos de baixo risco, aplicativos com dependências locais, SAP ou outros mainframes no Azure ou aplicativos que estendem o SAP ou mainframes locais. Cada organização tem necessidades únicas baseadas nas classificações de dados e nos tipos de aplicativos que são compatíveis com os negócios. O mapeamento de dependência do estado digital pode ajudar a definir arquétipos de aplicativo em uma organização.
- Uma convenção de nomenclatura comum deve ser aceita como parte do design da assinatura com base nos dois marcadores anteriores.

### <a name="resource-consistency"></a>Consistência de recursos

A **Consistência de hierárquica** foi escolhida como um padrão de Consistência de Recursos.

- Grupos de recursos devem ser criados para cada aplicativo. Grupos de gerenciamento são criados para cada arquétipo de aplicativo. O Azure Policy deve ser aplicado a todas as assinaturas no grupo de gerenciamento associado.
- Como parte do processo de implantação, os modelos de Consistência de recursos para todos os ativos devem ser armazenados o controle de origem.
- Cada grupo de recursos deve associar a um aplicativo ou carga de trabalho específica.
- A hierarquia do grupo de gerenciamento do Azure definida deve representar a responsabilidade de cobrança e a propriedade de aplicativo usando grupos aninhados.
- Uma ampla implementação do Azure Policy pode exceder os compromissos de tempo da equipe e pode não fornecer muito valor neste momento. No entanto, uma política padrão simples deve ser criada e aplicada a cada grupo de para impor as primeiras poucas declarações de política de governança de nuvem. Isso serve para definir a implementação de requisitos de governança específicos. Essas implementações poderão então ser aplicadas em todos os ativos implantados.

### <a name="resource-tagging"></a>Marcação de recursos

O padrão de **Contabilidade** foi escolhido para marcação de recursos.

- Os ativos implantados devem ser marcados com valores para: Unidade de cobrança/departamento, geografia, classificação de dados, nível de importância, SLA, ambiente, arquétipo do aplicativo, aplicativo e proprietário do aplicativo.
- Esses valores junto com o grupo de gerenciamento do Azure e a assinatura associada a um ativo implantado impulsionarão as decisões de governança, operações e segurança.

### <a name="logging-and-reporting"></a>Registro em log e relatórios

Neste ponto, um padrão **Híbrido** para o registro em log e relatório é sugerido, mas não é exigido de nenhuma equipe de desenvolvimento.

- Nenhum requisito de governança está definido atualmente em relação aos dados a serem coletados para fins de registro em log ou relatório. Isso é específico para esse narrativa fictícia e deve ser considerado um antipadrão. Padrões de registro em log devem ser determinados e impostos assim que possível.
- Uma análise adicional será necessária antes da liberação de dados protegidos ou cargas de trabalho críticas.
- Antes de dar suporte a cargas de trabalho de missão crítica ou dados protegidos, a solução de monitoramento operacional local deve conceder acesso ao espaço de trabalho usado para registro em log. Aplicativos são necessários para atender a requisitos de segurança e registro em log com o uso do locatário, se o aplicativo for compatível com um SLA definido.

## <a name="evolution-of-governance-processes"></a>Evolução dos processos de governança

Algumas das instruções de política não podem ou não devem ser controladas por ferramentas automatizadas. Outras políticas exigirão esforço periódico de segurança de TI e as equipes de linha de base de identidade local. A equipe de governança de nuvem será necessária para supervisionar os seguintes processos para implementar as últimas oito instruções de política:

**Alterações de política corporativa**: A equipe de governança de nuvem fará alterações de design MVP para adotar as novas políticas de governança. O valor de governança de MVP é que ele permite a aplicação automática de novas políticas.

**Aceleração da adoção**: A equipe de Governança de Nuvem tem examinado scripts de implantação em várias equipes. Ela mantém um conjunto de scripts que atuam como modelos de implantação. Esses modelos são usados pelas equipes de adoção de nuvem e equipes DevOps para definir implantações mais rapidamente. Cada script contém requisitos para impor políticas de controle e não é necessário esforço adicional dos engenheiros de adoção da nuvem. Como curadores desses scripts, eles implementam mais rapidamente as alterações de política. Além disso, são exibidos como aceleradores de adoção. Isso garante consistência entre implantações, sem forçar estritamente a aderência.

**Treinamento de engenharia**: A equipe de Governança de Nuvem oferece sessões de treinamento bimestrais além de ter criado dois vídeos para engenheiros. Ambos os recursos ajudam os engenheiros a acelerar rapidamente sobre a cultura de governança e as implantações executadas. A equipe está adicionando ativos de treinamento que mostram a diferença entre implantações em produção e de não produção, o que ajuda os engenheiros a entender como as novas políticas afetam a adoção. Isso garante consistência entre implantações, sem forçar estritamente a aderência.

**Planejamento de implantação**: Antes de implantar qualquer ativo que contém dados protegidos, a equipe de governança de nuvem será responsável por examinar os scripts de implantação para validar o alinhamento de governança. Equipes existentes com implantações aprovadas anteriormente serão auditadas usando ferramentas programáticas.

**Auditoria mensal e relatórios**: A cada mês, a equipe de Governança de Nuvem executa uma auditoria de todas as implantações na nuvem para validar o alinhamento contínuo à política. Quando são encontrados desvios, eles são documentados e compartilhados com as equipes de adoção da nuvem. Quando a imposição não representa um risco de interrupção dos negócios ou de perda de dados, as políticas serão automaticamente aplicadas. No final da auditoria, a equipe de Governança de Nuvem compila um relatório para a equipe de Estratégia de Nuvem e para cada equipe de adoção da nuvem para comunicar a adesão, de forma geral, à política. O relatório também é armazenado para fins de auditoria e legais.

**Revisão trimestral da política**: A cada trimestre, a equipe de Governança de Nuvem e a equipe de Estratégia de Nuvem revisará os resultados de auditoria e sugerirá alterações à política corporativa. Muitas dessas sugestões são o resultado de melhorias contínuas e da observação dos padrões de uso. As alterações de política aprovadas são integradas às ferramentas de governança durante ciclos posteriores de auditoria.

## <a name="alternative-patterns"></a>Padrões alternativos

Se qualquer um dos padrões escolhidos nessa jornada de governança não se alinhar aos requisitos do leitor, há alternativas disponíveis para cada padrão:

- [Padrões de criptografia](../../../decision-guides/encryption/overview.md)
- [Padrões de identidade](../../../decision-guides/identity/overview.md)
- [Registro em log e padrões de emissão de relatórios](../../../decision-guides/log-and-report/overview.md)
- [Padrões de imposição de política](../../../decision-guides/policy-enforcement/overview.md)
- [Padrões de consistência de recursos](../../../decision-guides/resource-consistency/overview.md)
- [Padrões de marcação de recursos](../../../decision-guides/resource-tagging/overview.md)
- [Padrões de rede definida pelo software](../../../decision-guides/software-defined-network/overview.md)
- [Padrões de Design de assinatura](../../../decision-guides/subscriptions/overview.md)

## <a name="next-steps"></a>Próximas etapas

Depois que este guia for implementado, cada equipe de adoção da nuvem poderá continuar com uma base sólida sobre governança. A equipe de Governança de Nuvem trabalhará em paralelo para atualizar continuamente as políticas corporativas e disciplinas de governança.

As duas equipes usarão os indicadores de tolerância para identificar a próxima evolução necessária para continuar a dar suporte à adoção da nuvem. A próxima etapa para a empresa nessa jornada está evoluindo sua linha de base de governança para dar suporte a aplicativos com requisitos de autenticação de multifator herdados ou de terceiros (MFA).

> [!div class="nextstepaction"]
> [Evolução da linha de base de identidade](./identity-baseline-evolution.md)
