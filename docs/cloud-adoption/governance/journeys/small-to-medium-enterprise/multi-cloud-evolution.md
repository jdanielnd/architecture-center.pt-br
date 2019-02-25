---
title: 'CAF: Empresa de pequeno a médio porte – evolução de várias nuvens'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Exeplicação de empresa de pequeno a médio porte – evolução de várias nuvens
author: BrianBlanchard
ms.openlocfilehash: a5b09c92acc4e165590b5a35827b88b0ca099bed
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900383"
---
# <a name="small-to-medium-enterprise-multi-cloud-evolution"></a>Empresas de pequeno a médio porte: Evolução de várias nuvens

Este artigo desenvolve a narrativa adicionando controles de custo para a adoção de várias nuvens.

## <a name="evolution-of-the-narrative"></a>Evolução da narrativa

A Microsoft reconhece que os clientes estão adotando várias nuvens para fins específicos. O cliente fictício nesse percurso não é exceção. Em paralelo ao percurso de adoção do Azure, o sucesso nos negócios levou à aquisição de uma empresa pequena, mas complementar. Esse negócio que está sendo executado para todas as suas operações de TI em um provedor de nuvem diferente.

Este artigo descreve como as coisas mudam ao integrar a nova organização. Para fins de narrativa, presumimos que essa empresa concluiu cada uma das evoluções de governança delimitadas nessa jornada do cliente.

### <a name="evolution-of-the-current-state"></a>Evolução do estado atual

Na fase anterior dessa narrativa, a empresa começou a enviar ativamente os aplicativos de produção para a nuvem por meio de pipelines de CI/CD.

Desde então, ocorreram algumas mudanças que afetarão a governança:

- A identidade é controlada por uma instância local do Active Directory Domain Services. A identidade híbrida é facilitada por meio da replicação para o Azure Active Directory.
- Operações de TI ou operações na nuvem em grande parte são gerenciadas pelo Azure Monitor e relacionados automações.
- Recuperação de desastre / continuidade de negócios é controlada pelas instâncias do Azure Vault.
- A Central de Segurança do Azure é usada para monitorar ataques e violações de segurança.
- A Central de Segurança do Azure e o Azure Monitor são usados para monitorar a governança de nuvem.
- O Azure Blueprints, Azure Policy e os grupos de gerenciamento do Azure são usados para automatizar a conformidade com a política.

### <a name="evolution-of-the-future-state"></a>Evolução do estado futuro

O objetivo é integrar a empresa de aquisição em operações existentes sempre que possível.

## <a name="evolution-of-tangible-risks"></a>Evolução de riscos tangíveis

**Custo de aquisição de negócios**: A aquisição de um novo negócio está prevista para ser lucrativa em aproximadamente cinco anos. Devido à taxa de retorno lenta, o conselho quer controlar os custos de aquisição o máximo possível. Há um risco de controle de custos e integração técnica conflitantes entre si.

Esse risco de negócios pode ser dividido em alguns riscos técnicos:

- A migração para a nuvem pode produzir os custos de aquisição adicionais
- O novo ambiente pode não ser devidamente controlado, o que pode resultar em violações de política.

## <a name="evolution-of-the-policy-statements"></a>Evolução das instruções da política

As seguintes alterações à política ajudam a reduzir novos riscos e a orientar a implementação.

1. Todos os ativos em uma nuvem secundária devem ser monitorados por meio do gerenciamento operacional existente e as ferramentas de monitoramento de segurança
2. Todas as Unidades de Organização devem ser integradas ao provedor de identidade existente
3. O provedor de identidade primária deve reger a autenticação aos ativos na nuvem secundária

## <a name="evolution-of-the-best-practices"></a>Evolução das práticas recomendadas

Esta seção do artigo evoluirá o design MVP de governança para incluir novas políticas do Azure e uma implementação do Gerenciamento de Custos do Azure. Juntas, essas duas alterações de design atenderão às novas instruções da política corporativa.

1. Conectar as redes. Esta etapa é executada pelas equipes de rede e segurança de TI e suporte da equipe de governança de nuvem. Adicionar uma conexão do provedor de MPLS/baseado em linha à nova nuvem irá integrar as redes. Adicionar tabelas de roteamento e configurações de firewall irá controlar o acesso e o tráfego entre os ambientes.
2. Consolidar provedores de identidade. Dependendo das cargas de trabalho que estão sendo hospedadas na nuvem secundária, há uma variedade de opções para a consolidação do provedor de identidade. Seguem alguns exemplos:
    1. Para aplicativos que se autenticam usando o OAuth 2, os usuários do Active Directory na nuvem secundária simplesmente podem ser replicados para o locatário existente do Active Directory Domain Services. Isso garante que todos os usuários podem ser autenticados no locatário.
    2. Por outro lado, a federação permite UOs fluam para o Active Directory local, em seguida, para a instância do Active Directory Domain Services.
3. Adicionar ativos ao Azure Site Recovery.
    1. O Azure Site Recovery foi projetado desde o início como uma ferramenta híbrida/de várias nuvens.
    2. As VMs na nuvem secundária poderão ser protegidas pelos mesmos processos do Azure Site Recovery usados para proteger ativos locais.
4. Adicionar ativos ao Gerenciamento de Custos do Azure
    1. O Gerenciamento de Custos do Azure foi projetado desde o início como uma ferramenta híbrida/de várias nuvens.
    2. Máquinas virtuais na nuvem secundária podem ser compatíveis com o Gerenciamento de Custos do Azure para alguns provedores de nuvem. Custos adicionais podem ser aplicados.
5. Adicione ativos para o Azure Monitor.
    1. O Azure Monitor foi projetado como uma ferramenta de nuvem híbrida desde o início.
    2. Máquinas virtuais na nuvem secundária podem ser compatíveis com os agentes do Azure Monitor, permitindo que sejam incluídos no Azure Monitor para monitoramento operacional.
6. Ferramentas de imposição de governança:
    1. A imposição de governança é específico da nuvem.
    2. As políticas corporativas estabelecidas no percurso da governança não são específicas da nuvem. Embora a implementação possa variar da nuvem para nuvem, as políticas podem ser aplicadas ao provedor secundário.

À medida que aumenta a adoção de várias nuvens, a evolução do design acima continuará a amadurecer.

## <a name="conclusion"></a>Conclusão

Esta série de artigos descreveu a evolução das práticas recomendadas de governança, alinhadas às experiências desta empresa fictícia. Iniciando pequena, mas com a base certa, a empresa pode se mover rapidamente e ainda assim se aplica a quantidade certa de governança no momento certo. O MVP por si só não protege o cliente. Em vez disso, criou a base para atenuar o risco e adicionou proteções. A partir daí, as camadas de governança foram aplicadas para reduzir os riscos tangíveis. O percurso exato apresentado aqui não se alinha 100% com as experiências de qualquer leitor. Em vez disso, serve como um padrão de governança incremental. O leitor é aconselhado a fim de adaptar essas práticas recomendadas de acordo com suas próprias restrições exclusivas e requisitos de governança.