---
title: 'CAF: Empresas de pequeno a médio porte - detalhes técnicos adicionais sobre o MVP de governança'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Explicação - Empresas de pequeno a médio porte - Detalhes técnicos adicionais sobre o MVP de governança
author: BrianBlanchard
ms.openlocfilehash: e726213459c8bee63e3cc77ab54868fe7196b3ac
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900253"
---
# <a name="small-to-medium-enterprise-best-practice-explained"></a>Empresas de pequeno a médio porte: Melhor prática explicada

A jornada de governança começa com um conjunto de [políticas corporativas](./initial-corporate-policy.md) iniciais. Essas políticas são usadas para estabelecer um MVP de governança que reflita as [práticas recomendadas](./overview.md).

Neste artigo, discutiremos as estratégias de alto nível que são necessárias para a criação de um MVP de governança. O ponto central do MVP de governança é a disciplina [Aceleração da Implantação](../../deployment-acceleration/overview.md). As ferramentas e os padrões aplicados neste estágio permitirão as evoluções incrementais necessárias para expandir a governança no futuro.

## <a name="governance-mvp-cloud-adoption-foundation"></a>MVP de governança (fundamentos para adoção da nuvem)

A adoção rápida da governança e a política corporativa é uma realidade graças a alguns princípios simples e ferramentas de governança baseadas em nuvem. Essas são as primeiras das três Disciplinas de governança de nuvem a serem abordadas em qualquer processo de governança. Nos aprofundaremos em cada uma delas neste artigo.

Para estabelecer o ponto de partida, este artigo discutirá as estratégias de alto nível por trás da Linha de Base de Identidade, Linha de Base de Segurança e Aceleração de Implantação que são necessárias para criar um MVP de governança e que servirão como a base para a adoção como um todo.

![Exemplo de MVP de governança incremental](../../../_images/governance/governance-mvp.png)

## <a name="implementation-process"></a>Processo de implementação

A implementação do MVP de governança possui dependências de Identidade, Segurança e Rede. Depois de resolver as dependências, a equipe de Governança de Nuvem decidirá sobre alguns aspectos de governança. As decisões da equipe de Governança de Nuvem e das equipes de suporte serão implementadas por meio de um único pacote de ativos de imposição.

![Exemplo de MVP de governança incremental](../../../_images/governance/governance-mvp-implementation-flow.png)

Essa implementação também pode ser descrita usando uma lista de verificação simples:

1. Solicite decisões relacionadas às dependências básicas: Identidade, Rede e Criptografia.
2. Determine o padrão a ser usado durante a imposição da política corporativa.
3. Determine os padrões de governança apropriados para as disciplinas Consistência de Recursos, Marcação de Recursos e Registro em Log e Relatórios.
4. Implemente as ferramentas de governança alinhadas ao padrão de imposição de política escolhido para aplicar as decisões dependentes e as decisões de governança.

[!INCLUDE [implementation-process](../../../../../includes/cloud-adoption/governance/implementation-process.md)]

## <a name="application-of-governance-defined-patterns"></a>Aplicação de padrões definidos por governança

A equipe de Governança de Nuvem é responsável pelas seguintes decisões e implementações. Muitas equipes precisam de informações de outras equipes, mas a equipe de Governança de Nuvem provavelmente será a responsável pela decisão e pela implementação. As seções a seguir descrevem as decisões tomadas para esse caso de uso e os detalhes de cada decisão.

### <a name="subscription-model"></a>Modelo de assinatura

O padrão **Categoria do aplicativo** foi escolhido para assinaturas do Azure.

- Um arquétipo de aplicativo é uma maneira de agrupar aplicativos com necessidades semelhantes. Exemplos comuns incluem: Aplicativos com dados protegidos, aplicativos regulamentados (como HIPAA ou FedRAMP), aplicativos de baixo risco, aplicativos com dependências locais, SAP ou outros mainframes no Azure ou aplicativos que estendem o SAP ou mainframes locais. Esses arquétipos são exclusivos por organização, baseados nas classificações de dados e nos tipos de aplicativos que potencializam os negócios. O mapeamento de dependência do espaço digital pode ajudar a definir arquétipos de aplicativo em uma organização.
- Os departamentos provavelmente não serão necessários, dado o foco atual. As implantações devem ser limitadas a uma única unidade de cobrança. No estágio de adoção, pode até não haver um Contrato Enterprise para centralizar a cobrança. É provável que esse nível de adoção seja gerenciado por uma única assinatura pré-paga do Azure.
- Independentemente do uso do Portal do EA ou da existência de um Contrato Enterprise, um modelo de assinatura ainda deve ser definido e aceito para minimizar sobrecargas administrativas além de apenas cobrança.
- No padrão **Categoria de aplicativo**, as assinaturas são criadas para cada arquétipo de aplicativo. Cada assinatura pertence a uma conta para cada ambiente (Desenvolvimento, Teste e Produção).
- Uma convenção de nomenclatura comum deve ser aceita como parte do design da assinatura com base nos dois pontos anteriores.

### <a name="resource-consistency"></a>Consistência de recursos

O padrão **Consistência de implantação** foi escolhido como uma Consistência de recursos.

- Grupos de recursos são criados para cada aplicativo. Grupos de gerenciamento são criados para cada arquétipo de aplicativo. O Azure Policy deve ser aplicado a todas as assinaturas do grupo de gerenciamento associado.
- Como parte do processo de implantação, os modelos de Consistência de recursos do Azure do grupo de recursos devem ser armazenados no controle do código-fonte.
- Cada grupo de recursos está associado a um aplicativo ou carga de trabalho específica.
- Os grupos de gerenciamento do Azure permitem a atualização de designs de governança conforme a política corporativa amadurece.
- Uma ampla implementação do Azure Policy pode exceder os compromissos de tempo da equipe e pode não ser de grande utilidade no momento. No entanto, uma política padrão simples deve ser criada e aplicada a cada grupo de gerenciamento para impor o pequeno número de instruções de políticas de governança de nuvem atual. Essa política definirá a implementação de requisitos de governança específicos. Essas implementações poderão então ser aplicadas em todos os ativos implantados.

### <a name="resource-tagging"></a>Marcação de recursos

O padrão **Classificação** para a marcação foi escolhido como um modelo para marcação de recursos.

- Os ativos implantados devem ser marcados com os seguintes valores: Classificação de dados, Nível de importância, SLA e Ambiente.
- Esses quatro valores guiarão as decisões relativas à governança, operações e segurança.
- Se essa jornada de governança estiver sendo implementada para uma unidade de negócios ou equipe dentro de uma empresa maior, a marcação também deverá incluir metadados para a unidade de cobrança.

### <a name="logging-and-reporting"></a>Registro em log e relatórios

Neste ponto, um padrão **Nativo de nuvem** para o registro em log e relatório é sugerido, mas não é exigido de nenhuma equipe de desenvolvimento.

- Nenhum requisito de governança foi definido em relação aos dados a serem coletados para fins de registro em log ou relatório.
- Uma análise adicional será necessária antes de liberar quaisquer dados protegidos ou cargas de trabalho críticas.

## <a name="evolution-of-governance-processes"></a>Evolução dos processos de governança

À medida que a governança evolui, algumas declarações de política não podem ou não devem ser controladas por ferramentas automatizadas. Outras políticas resultarão em esforços por parte da equipe de Segurança de TI e da equipe local de Gerenciamento de Identidades ao longo do tempo. Para ajudar a atenuar novos riscos conforme eles surgirem, a equipe de Governança de Nuvem supervisionará os processos a seguir.

**Aceleração da adoção**: A equipe de Governança de Nuvem tem examinado scripts de implantação em várias equipes. Ela mantém um conjunto de scripts que atuam como modelos de implantação. Esses modelos são usados na adoção da nuvem e pelas equipes de DevOps para definir implantações com mais rapidez. Cada um desses scripts contém os requisitos necessários para impor diversas políticas de governança, sem esforço adicional dos engenheiros de adoção da nuvem. Como curadores desses scripts, a equipe de Governança de Nuvem pode implementar mais rapidamente as alterações de política. Como resultado da curadoria do script, a equipe de Governança de Nuvem é vista como uma fonte de aceleração da adoção. Isso cria consistência entre implantações, sem forçar estritamente a aderência.

**Treinamento de engenharia**: A equipe de Governança de Nuvem oferece sessões de treinamento bimestrais além de ter criado dois vídeos para engenheiros. Esses materiais ajudam os engenheiros a aprender rapidamente a cultura de governança e como as coisas são feitas durante as implantações. A equipe está adicionando ativos de treinamento que mostram a diferença entre implantações em produção e de não produção para que os engenheiros compreendam como as novas políticas afetarão a adoção. Isso cria consistência entre implantações, sem forçar estritamente a aderência.

**Planejamento de implantação**: Antes de implantar qualquer ativo que contenha dados protegidos, a equipe de Governança de Nuvem revisará os scripts de implantação para validar o alinhamento da governança. Equipes existentes com implantações aprovadas anteriormente serão auditadas usando ferramentas programáticas.

**Auditoria mensal e relatórios**: A cada mês, a equipe de Governança de Nuvem executa uma auditoria de todas as implantações na nuvem para validar o alinhamento contínuo à política. Quando são encontrados desvios, eles são documentados e compartilhados com as equipes de adoção da nuvem. Quando a imposição não representa um risco de interrupção dos negócios ou de perda de dados, as políticas serão automaticamente aplicadas. No final da auditoria, a equipe de Governança de Nuvem compila um relatório para a equipe de Estratégia de Nuvem e para cada equipe de adoção da nuvem para comunicar a adesão, de forma geral, à política. O relatório também é armazenado para fins de auditoria e legais.

**Revisão trimestral da política**: A cada trimestre, a equipe de Governança de Nuvem e a equipe de Estratégia de Nuvem revisará os resultados de auditoria e sugerirá alterações à política corporativa. Muitas dessas sugestões são o resultado de melhorias contínuas e da observação dos padrões de uso. As alterações de política aprovadas são integradas às ferramentas de governança durante ciclos posteriores de auditoria.

## <a name="alternative-patterns"></a>Padrões alternativos

Se qualquer um dos padrões escolhidos nessa jornada de governança não se alinhar aos requisitos do leitor, há alternativas disponíveis para cada padrão:

- [Padrões de criptografia](../../../decision-guides/encryption/overview.md)
- [Padrões de identidade](../../../decision-guides/identity/overview.md)
- [Padrões de registro em log e relatórios](../../../decision-guides/log-and-report/overview.md)
- [Padrões de imposição de política](../../../decision-guides/policy-enforcement/overview.md)
- [Padrões de consistência de recursos](../../../decision-guides/resource-consistency/overview.md)
- [Padrões de marcação de recursos](../../../decision-guides/resource-tagging/overview.md)
- [Padrões de rede definida por software](../../../decision-guides/software-defined-network/overview.md)
- [Padrões de design de assinatura](../../../decision-guides/subscriptions/overview.md)

## <a name="next-steps"></a>Próximas etapas

Depois que este guia for implementado, cada equipe de adoção da nuvem poderá continuar o trabalho com uma base sólida sobre governança. A equipe de Governança de Nuvem trabalhará em paralelo para atualizar continuamente as políticas corporativas e disciplinas de governança.

As duas equipes usarão os indicadores de tolerância para identificar a próxima evolução necessária para continuar a dar suporte à adoção da nuvem. Para a empresa fictícia nessa jornada, a próxima etapa é desenvolver a Linha de Base de Segurança para dar suporte à transferência de dados protegidos para a nuvem.

> [!div class="nextstepaction"]
> [Evolução da Linha de Base de Segurança](./security-baseline-evolution.md)