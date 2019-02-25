---
title: 'CAF: Empresa de grande porte – evolução da linha de base de identidade'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Empresa de grande porte – evolução da linha de base de identidade
author: BrianBlanchard
ms.openlocfilehash: efda14819bfbc70632dc9bb8b4c6c5aca96004c0
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900256"
---
# <a name="large-enterprise-identity-baseline-evolution"></a>Empresas de grande porte: Evolução da linha de base de identidade

Este artigo desenvolve a narrativa adicionando controles de Linha de Base de identidade para a governança MVP.

## <a name="evolution-of-the-narrative"></a>Evolução da narrativa

A justificativa comercial para a migração de nuvem dos dois datacenters foi aprovada pelo CFO. Durante o estudo da viabilidade técnica, várias dificuldades foram descobertas:

- Dados protegidos e aplicativos de missão crítica representam 25% das cargas de trabalho em dois data centers. Nenhum dos dois pode ser eliminado até que as políticas de controle atual em relação à PII e aplicativos de missão crítica tenham sido modernizados.
- % 7 dos ativos nos data centers não são compatíveis com a nuvem. Eles serão movidos para um datacenter alternativo antes do término do contrato de datacenter.
- 15% dos ativos no data center (750 máquinas de virtuais) têm uma dependência na autenticação herdada ou autenticação de multifator de terceiros.
- A conexão VPN que conecta os datacenters existentes e o Azure não oferece velocidades ou latência de transmissão de dados suficientes para migrar o volume dos ativos dentro da linha do tempo de dois para desativar o datacenter.

Os dois primeiros obstáculos estão sendo reduzidos em paralelo. Este artigo abordará a resolução das dificuldades do terceiro e quarto bloqueio.

### <a name="evolution-of-the-cloud-governance-team"></a>Evolução da equipe de Governança de Nuvem

A Equipe de Governança de Nuvem está aumentando. Dada a necessidade de suporte adicionais sobre o gerenciamento de identidade, um administrador de sistemas da equipe de linha de base de identidade agora participa de uma reunião semanal para manter os membros da equipe existente ciente das alterações.

### <a name="evolution-of-the-current-state"></a>Evolução do estado atual

A equipe de TI tem aprovação para prosseguir com o CIO e o CFO planeja desativar dois data centers. No entanto, a IT está preocupada que 750 (15%) dos ativos nos data centers terão de ser movidos em algum lugar diferente da nuvem.

### <a name="evolution-of-the-future-state"></a>Evolução do estado futuro

Os novos planos de estado futuro exigem uma solução de identidade da linha de base mais robusta para migrar as 750 máquinas virtuais com os requisitos de autenticação herdados. Além desses dois data centers, porcentagens semelhantes de ativos em outros data centers devem ser afetadas por esse desafio.
O estado futuro agora também requer uma conexão do provedor de nuvem à solução MPLS/baseado em linha da empresa.

As alterações ao estado atual e futuro expõem novos riscos que exigem novas instruções de política.

## <a name="evolution-of-tangible-risks"></a>Evolução de riscos tangíveis

**Interrupção dos negócios durante a migração**. A migração para a nuvem cria um risco controlado e limite de tempo que pode ser gerenciado. Mover o hardware de classificação por vencimento para outra parte do mundo é um risco muito maior. Uma estratégia de mitigação é necessária para evitar interrupções para as operações de negócios.

**Dependências de identidade existentes**. As dependências em serviços de autenticação e identidade existentes podem atrasar ou evitar a migração de algumas cargas de trabalho para a nuvem. A falha ao retornar dois datacenters na hora incorrerá em milhões de dólares em valores de concessão de datacenter.

Esse risco de negócios pode ser dividido em alguns riscos técnicos:

- A autenticação herdada pode não estar disponível na nuvem, limitando a implantação de alguns aplicativos.
- A solução MFA de terceiros atual pode não estar disponível na nuvem, limitando a implantação de alguns aplicativos.
- Reorganizar ou mover a nuvem cria resultados e adiciona custos.
- A velocidade e a estabilidade do VPN podem impedir a migração.
- O tráfego de entrada para a nuvem pode causar problemas de segurança em outras partes da rede global.

## <a name="evolution-of-the-policy-statements"></a>Evolução das instruções da política

As seguintes alterações à política ajudam a reduzir novos riscos e a orientar a implementação.

1. O provedor de nuvem escolhido deve oferecer um meio de autenticar por meio de métodos herdados.
2. O provedor de nuvem escolhido deve oferecer um meio de autenticação com a solução atual de MFA de terceiros.
3. Uma conexão privada de alta velocidade deve ser estabelecida entre o provedor de nuvem e o provedor de telecomunicações da empresa, conectando o provedor de nuvem à rede global dos datacenters.
4. Até que os requisitos de segurança suficientes sejam estabelecidos, nenhum tráfego de entrada público pode acessar os ativos da empresa hospedados na nuvem. Todas as portas são bloqueadas de qualquer fonte fora da WAN global.

## <a name="evolution-of-the-best-practices"></a>Evolução das práticas recomendadas

O design de MVP de governança evoluirá para incluir novas políticas do Azure e uma implementação do Active Directory Domain Services em uma máquina virtual. Juntas, essas duas alterações de design atenderão às novas instruções da política corporativa.

Aqui estão as novas práticas recomendadas:

1. Blueprint DMZ: O lado local do DMZ deve ser configurado para permitir comunicação entre a seguinte solução e os servidores do Active Directory Domain Services local. Essa prática recomendada requer um DMZ para habilitar o Active Directory Domain Services entre os limites de rede.
2. Modelos do Azure Resource Manager:
    1. Defina um NSG para bloquear o tráfego externo e o tráfego interno da lista de permissões.
    1. Implante duas máquinas virtuais do AD em um par com balanceamento de carga com base em uma imagem dourada. Na primeira inicialização, essa imagem executa um script do PowerShell para ingressar no domínio e registrar-se aos serviços de domínio. Para obter mais informações, consulte [Estendendo o Active Directory Domain Services (AD DS) para o Azure](../../../../reference-architectures/identity/adds-extend-domain.md).
3. Azure Policy: Aplica o NSG a todos os recursos.
4. Blueprint do Azure
    1. Criar um blueprint nomeado `active-directory-virtual-machines`.
    1. Adicione cada um dos modelos do AD e as políticas para o blueprint.
    1. Publique o plano gráfico em qualquer grupo de gerenciamento aplicáveis.
    1. Aplique o blueprint para qualquer assinatura que exija autenticação de MFA de terceiros ou herdada.
    1. A instância do AD em execução no Azure agora pode ser usada como uma extensão do local a solução do AD, permitindo que ele se integre à ferramenta MFA existente e forneçam a autenticação baseada em declarações, ambos por meio da funcionalidade existente do Active Directory Domain Services.

## <a name="conclusion"></a>Conclusão

Adicionar essas alterações para a governança MVP ajuda a atenuar muitos riscos neste artigo, permitindo que cada equipe de adoção da nuvem se mova rapidamente após esse obstáculo.

## <a name="next-steps"></a>Próximas etapas

Como a adoção da nuvem se desenvolve e entrega valor comercial, riscos e governança de nuvem também precisam evoluir. A seguir estão algumas evoluções que podem ocorrer. Para a empresa fictícia nessa jornada, o próximo gatilho é a inclusão dos dados protegidos no plano de adoção de nuvem. Essa alteração exigirá os controles de segurança adicional.

> [!div class="nextstepaction"]
> [Evolução da Linha de Base de Segurança](./security-baseline-evolution.md)
