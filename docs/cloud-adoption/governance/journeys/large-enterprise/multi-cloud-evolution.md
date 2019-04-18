---
title: 'CAF: Grandes empresas – evolução de várias nuvens'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Grandes empresas – evolução de várias nuvens
author: BrianBlanchard
ms.openlocfilehash: 62a2fdd6e340c96494354f4f0cf2f78ab572c251
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59641137"
---
# <a name="large-enterprise-multi-cloud-evolution"></a>Empresas de grande porte: Evolução de várias nuvens

## <a name="evolution-of-the-narrative"></a>Evolução da narrativa

A Microsoft reconhece que os clientes estão adotando várias nuvens para fins específicos. A companhia fictícia nesse percurso não é exceção. Paralelamente ao percurso de adoção do Azure, o sucesso da empresa resultou na aquisição de uma empresa de pequeno porte, mas complementar. Essa empresa está executando todas as suas operações de TI em um provedor de nuvem diferente.

Este artigo descreve as mudanças decorrentes da integração com a nova organização. Para fins da narrativa, assumimos que essa empresa concluiu cada uma das evoluções de governança descritas neste percurso do cliente.

### <a name="evolution-of-the-current-state"></a>Evolução do estado atual

Na fase anterior dessa narrativa, a empresa havia começado a implementar controles de custo e monitoramento de custo, já que os gastos relacionados à nuvem tornaram-se parte das despesas operacionais regulares da empresa.

Desde então, ocorreram algumas mudanças que afetarão a governança:

- A identidade é controlada por uma instância local do Active Directory. A identidade híbrida é facilitada por meio da replicação para o Azure Active Directory.
- As Operações de TI ou Operações de Nuvem são amplamente gerenciadas pelo Azure Monitor e por automações relacionadas.
- A Recuperação de Desastres/Continuidade dos Negócios é controlada pelas instâncias do Azure Vault.
- A Central de Segurança do Azure é utilizada para monitorar violações de segurança e ataques.
- A Central de Segurança do Azure e o Azure Monitor são utilizados para monitorar a governança da nuvem.
- O Azure Blueprints, Azure Policy e os grupos de gerenciamento são usados para automatizar a conformidade com a política.

### <a name="evolution-of-the-future-state"></a>Evolução do estado futuro

A meta é integrar a empresa de aquisição nas operações existentes, sempre que possível.

## <a name="evolution-of-tangible-risks"></a>Evolução de riscos tangíveis

**Custo de aquisição de negócios**: A aquisição do novo negócio está prevista para ser rentável em aproximadamente cinco anos. Devido à taxa de retorno lenta, o conselho quer controlar os custos de aquisição o máximo possível. Há um risco de controle de custos e integração técnica conflitantes entre si.

Esse risco empresarial pode ser desenvolvido em alguns riscos técnicos

- Há risco de migração de nuvem produzindo custos de aquisição adicionais.
- Também há o risco do novo ambiente não ser regido adequadamente ou resultar em violações da política.

## <a name="evolution-of-the-policy-statements"></a>Evolução das declarações da política

As seguintes alterações da política ajudarão a mitigar os novos riscos e orientarão a implementação.

1. Todos os ativos em uma nuvem secundária devem ser monitorados por meio do gerenciamento operacional e das ferramentas de monitoramento de segurança existentes.
2. Todas as unidades organizacionais devem ser integradas no provedor de identidade existente.
3. O provedor de identidade primário deve reger a autenticação para ativos na nuvem secundária.

## <a name="evolution-of-the-best-practices"></a>Evolução das melhores práticas

Esta seção do artigo evoluirá o design MVP de governança para incluir novas políticas do Azure e uma implementação do Gerenciamento de Custos do Azure. Em conjunto, essas duas alterações de design atenderão às novas declarações da política corporativa.

1. Conecte as redes - Executado pela Segurança de Rede e TI, com suporte pela governança
    1. Adicionar uma conexão do provedor de linha dedicada/MPLS à nova nuvem irá integrar as redes. Adicionar tabelas de roteamento e configurações de firewall controlará o acesso e o tráfego entre os ambientes.
2. Consolidar provedores de identidade. Dependendo das cargas de trabalho que estão sendo hospedadas na nuvem secundária, haverá diversas opções para a consolidação do provedor de identidade. A seguir, estão alguns exemplos:
    1. Para aplicativos que autenticam usando OAuth 2, os usuários no Active Directory na nuvem secundária podem ser simplesmente replicados para o locatário do Azure AD existente.
    2. Por outro lado, a federação entre os dois provedores de identidade locais permitiria que os usuários dos novos domínios do Active Directory fossem replicados para o Azure.
3. Adicionar ativos ao Azure Site Recovery
    1. O Azure Site Recovery foi criado como uma ferramenta híbrida/multinuvem desde o início.
    2. As máquinas virtuais na nuvem secundária podem ser protegidas pelos mesmos processos do Azure Site Recovery usados para proteger os ativos locais.
4. Adicionar ativos ao Gerenciamento de Custos do Azure
    1. O Gerenciamento de Custos do Azure foi criado como uma ferramenta multinuvem desde o início.
    2. As máquinas virtuais na nuvem secundária podem ser compatíveis com o Gerenciamento de Custos do Azure para alguns provedores de nuvem. Custos adicionais podem ser aplicados.
5. Adicionar ativos para Azure Monitor
    1. O Azure Monitor foi desenvolvido como uma ferramenta de nuvem híbrida desde o início.
    2. As máquinas virtuais na nuvem secundária podem ser compatíveis com os agentes do Azure Monitor, permitindo que sejam incluídas no Azure Monitor para monitoramento operacional.
6. Ferramentas de imposição de governança
    1. A imposição de governança é específica da nuvem.
    2. As políticas corporativas estabelecidas no percurso de governança não. Embora a implementação possa variar de nuvem para nuvem, as declarações da política podem ser aplicadas ao provedor secundário.

Na medida em que a adoção de multinuvem cresce, a evolução do design acima continua se consolidando.

## <a name="next-steps"></a>Próximos passos

Em muitos grandes empresas, disciplinas cinco de governança de nuvem pode ser impedimentos à adoção. O próximo artigo tem algumas ideias adicionais em tornar governança esporte uma equipe para ajudar a garantir o sucesso a longo prazo na nuvem.

> [!div class="nextstepaction"]
> [Múltiplas camadas de governança](./multiple-layers-of-governance.md)
