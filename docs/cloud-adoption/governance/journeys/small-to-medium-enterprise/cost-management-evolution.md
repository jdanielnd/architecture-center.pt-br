---
title: 'CAF: Empresa de pequeno a médio porte – evolução do Gerenciamento de Custos  '
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: Explicação de empresa de pequeno a médio porte – evolução do Gerenciamento de Custos
author: BrianBlanchard
ms.openlocfilehash: ca070bfca3efbf9548faa8cf28a1adffefd4a33a
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900193"
---
# <a name="small-to-medium-enterprise-cost-management-evolution"></a>Empresas de pequeno a médio porte: Evolução do gerenciamento de custos

Este artigo desenvolve a narrativa adicionando controles de custo para a governança MVP.

## <a name="evolution-of-the-narrative"></a>Evolução da narrativa

A adoção ultrapassou o indicador de tolerância de custo definido na governança MVP. Isso é bom, pois corresponde às migrações do datacenter “DR”. Agora, o aumento em gastos justifica um investimento de tempo da equipe de governança de nuvem.

### <a name="evolution-of-the-current-state"></a>Evolução do estado atual

Na fase anterior a essa narrativa, a TI havia desativado 100% do data center de recuperação de Desastre. As equipes de BI e o desenvolvimento de aplicativos estavam prontos para o tráfego de produção.

Desde então, ocorreram algumas mudanças que afetarão a governança:

- A equipe de migração começou migrando as VMs fora do datacenter de produção.
- As equipes de desenvolvimento de aplicativo estão enviando ativamente por push a aplicativos de produção para a nuvem por meio de pipelines de CI/CD. Esses aplicativos de forma reativa podem dimensionar com demandas de usuário.
- A equipe de inteligência de negócios dentro da TI ofereceu uma série de ferramentas de análise preditiva na nuvem. os volumes de dados agregados na nuvem continuam a crescer.
- Todo esse crescimento oferece suporte aos resultados de negócios comprometidos. No entanto, os custos começaram a crescer. Orçamentos projetados estão crescendo mais rapidamente do que o esperado. O CFO precisa de abordagens aprimoradas para gerenciamento de custos.

### <a name="evolution-of-the-future-state"></a>Evolução do estado futuro

O custo de monitoramento e a emissão de relatórios deve ser adicionado à solução de nuvem. A TI ainda está atuando como uma casa de compensação de custo. Isso significa que o pagamento para serviços de nuvem continua a vir de pagamento da TI. No entanto, a criação de relatórios deve vincular as despesas operacionais diretas às funções que estão consumindo os custos de nuvem. Esse modelo é mencionado como modelo "Mostrar novamente" para a nuvem de contabilidade.

As alterações ao estado atual e futuro expõem novos riscos que exigem novas instruções de política.

## <a name="evolution-of-tangible-risks"></a>Evolução de riscos tangíveis

**Controle de orçamento**: Há um risco inerente que recursos de autoatendimento resultarão em custos excessivos e inesperados na nova plataforma. Processos de governança para monitorar os custos e minimizar os riscos de custo contínuo devem estar em vigor para garantir o alinhamento contínuo com o orçamento planejado.

Esse risco de negócios pode ser dividido em alguns riscos técnicos:

- Os custos reais podem exceder o plano.
- As condições de negócios mudam. Quando isso acontece, haverá casos quando uma função de negócios precisa consumir mais serviços de nuvem que o esperado, gerando anomalias de gastos. Há um risco de que esses gastos extras sejam considerados excedentes, em vez de um ajuste necessário para o plano.
- Sistemas poderiam ser sobreprovisionados, resultando em excesso de gastos.

## <a name="evolution-of-the-policy-statements"></a>Evolução das instruções da política

As seguintes alterações à política ajudam a reduzir novos riscos e a orientar a implementação.

1. Todos os custos de nuvem devem ser monitorados em relação ao plano semanalmente pela equipe de governança. Relatório sobre desvios entre os custos de nuvem e o plano devem compartilhados com a liderança de TI e finanças mensalmente. Todos os custos de nuvem e as atualizações de plano devem ser revisados com a liderança de TI e finanças mensalmente.
2. Todos os custos devem ser alocados para uma função de negócios para fins de responsabilidade.
3. Os ativos de nuvem devem ser monitorados continuamente para oportunidades de otimização.
4. As ferramentas de governança de nuvem devem limitar as opções de dimensionamento de ativo para uma lista aprovada de configurações. A ferramenta deve garantir que todos os ativos sejam detectáveis e controlados pelo custo de solução de monitoramento.
5. Durante o planejamento da implantação, os recursos de nuvem necessários associados à hospedagem de cargas de trabalho de produção devem ser documentados. Esta documentação ajudará a refinar os orçamentos e preparar automação adicional para evitar o uso das opções mais caras. Durante esse processo, a consideração deve ser dada a diferentes ferramentas de descontos oferecidos pelo provedor de nuvem, como instâncias reservadas ou reduções de custo de licença.
6. Todos os aplicativos proprietários são necessários para participar de treinamentos sobre práticas para otimizar cargas de trabalho para melhor controlar os custos da nuvem.

## <a name="evolution-of-the-best-practices"></a>Evolução das práticas recomendadas

Esta seção do artigo evoluirá o design MVP de governança para incluir novas políticas do Azure e uma implementação do Gerenciamento de Custos do Azure. Juntas, essas duas alterações de design atenderão às novas instruções da política corporativa.

1. Implementar o Gerenciamento de Custos do Azure
    1. Estabelecer o escopo correto de acesso para se alinhar ao padrão de assinatura e a disciplina de Consistência de recursos. Supondo que o alinhamento com a governança MVP definido nos artigos anteriores, isso exige acesso ao **Escopo de Conta de Registro** para executar a equipe de governança de nuvem nos relatórios de alto nível. Equipes adicionais fora de governança podem exigir acesso ao **Escopo do Grupo de Recursos**.
    2. Estabelecer um orçamento no Gerenciamento de Custos do Azure.
    3. Examinar e agir em recomendações iniciais. Ter um processo recorrente para oferecer suporte a relatórios.
    4. Configurar e executar Relatórios de Gerenciamento de Custos do Azure, tanto iniciais como recorrentes.
2. Atualizar o Azure Policy
    1. Auditar a marcação, grupo de gerenciamento, assinatura e valores de grupo de recursos para identificar qualquer desvio.
    2. Estabeleça as opções de tamanho de SKU para limitar as implantações para SKUs listadas na documentação de planejamento de implantação.

## <a name="conclusion"></a>Conclusão

Adicionar esses processos e alterações ao MVP de governa ajuda a eliminar muitos dos riscos associados à governança de custo. Juntos, criam a visibilidade, a responsabilidade e a otimização necessárias para controlar os custos.

## <a name="next-steps"></a>Próximas etapas

Como a adoção da nuvem continua a se desenvolver e a tornar-se mais útil comercialmente, a governança de riscos e de nuvem também precisam evoluir. Para a empresa fictícia nessa jornada, a próxima etapa é usar esse investimento de governança para gerenciar várias nuvens.

> [!div class="nextstepaction"]
> [Evolução de várias nuvens](./multi-cloud-evolution.md)
