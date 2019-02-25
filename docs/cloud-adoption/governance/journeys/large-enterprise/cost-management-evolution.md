---
title: 'CAF: Empresa de grande porte – evolução do Gerenciamento de Custos'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Empresa de grande porte – evolução do Gerenciamento de Custos
author: BrianBlanchard
ms.openlocfilehash: 6bf63e36f6fb19dd0e5f9a16a7f66eb6e0ed54ff
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900280"
---
# <a name="large-enterprise-cost-management-evolution"></a>Empresas de grande porte: Evolução do gerenciamento de custos

Este artigo desenvolve a narrativa adicionando controles de custo a governança de produto mínimo viável (MVP).

## <a name="evolution-of-the-narrative"></a>Evolução da narrativa

A adoção ultrapassou o indicador de tolerância definido na governança MVP. Agora, o aumento em gastos justifica um investimento de tempo da equipe de Governança de nuvem para monitorar e controlar os padrões de gasto.

Como um driver claro de inovação de, a IT não é mais vista principalmente como um centro de custo. Como a organização de TI oferece mais valor, CIO e CFO concordam que é a hora de evoluir a função que a IT desempenha na empresa. Entre outras alterações, o diretor financeiro quer testar uma abordagem de pagamento direto para estatísticas para a filial canadense de uma das unidades de negócios na nuvem. Um dos dois datacenters obsoletos era exclusivamente ativos hospedados para essas operações canadenses da unidade comercial. Nesse modelo, a subsidiária canadense da unidade comercial será cobrada diretamente quanto as despesas operacionais relacionadas aos ativos hospedados. Esse modelo permite que a TI se concentre menos em gerenciar os gastos de alguém e mais em criar valor. No entanto, antes de começar essa transição, pode começar com as ferramentas do Gerenciamento de Custos necessárias no local.

### <a name="evolution-of-current-state"></a>Evolução do estado atual

Na fase anterior dessa narrativa, a equipe de TI foi ativamente movendo cargas de trabalho de produção com dados protegidos no Azure.

Desde então, ocorreram algumas mudanças que afetarão a governança:

- 5.000 ativos foram removidos de dois datacenters sinalizados para desativação. Aquisição e a segurança de TI estão agora desprovisionando os ativos físicos restantes.
- As equipes de desenvolvimento de aplicativo têm implementado pipelines de CI/CD para implantar um número de aplicativos nativos de nuvem, afetando significativamente as experiências do cliente.
- A equipe de BI criou agregação, curadoria, insight e processos de previsão, levando benefícios tangíveis para operações de negócios. Essas previsões agora estão capacitando novos produtos e serviços criativos.

### <a name="evolution-of-future-state"></a>Evolução do estado futuro

- O custo de monitoramento e a emissão de relatórios deve ser adicionado à solução de nuvem. A criação de relatórios deve vincular as despesas operacionais diretas às funções que estão consumindo os custos de nuvem. Outros tipos de relatório devem permitir que a TI monitore os gastos e forneça orientações técnicas sobre o gerenciamento de custos. Para a filial canadense, o departamento será cobrado diretamente.

## <a name="evolution-of-tangible-risks"></a>Evolução de riscos tangíveis

**Controle de orçamento**: Há um risco inerente que recursos de autoatendimento resultarão em custos excessivos e inesperados na nova plataforma. Processos de governança para monitorar os custos e minimizar os riscos de custo contínuo devem estar em vigor para garantir o alinhamento contínuo com o orçamento planejado.

Esse risco de negócios pode ser dividido em alguns riscos técnicos:

- Há um risco dos custos reais que excedem o plano.
- As condições de negócios mudam. Quando isso acontece, haverá casos quando uma função de negócios precisa consumir mais serviços de nuvem que o esperado, gerando anomalias de gastos. Há um risco que esses custos adicionais possam ser vistos como excedentes em vez de um ajuste necessário para o plano. Se for bem-sucedido, o experimento canadense deve ajudar a reduzir esse risco.
- Há um risco de sistemas que estão sendo sobre provisionados, resultando em excesso de gastos.

## <a name="evolution-of-the-policy-statements"></a>Evolução das instruções da política

As seguintes alterações à política ajudam a reduzir novos riscos e a orientar a implementação.

1. Todos os custos de nuvem devem ser monitorados em relação ao plano semanalmente pela equipe de Governança de nuvem. Relatório sobre desvios entre os custos de nuvem e o plano devem compartilhados com a liderança de TI e finanças mensalmente. Todos os custos de nuvem e as atualizações de plano devem ser revisados com a liderança de TI e finanças mensalmente.
2. Todos os custos devem ser alocados para uma função de negócios para fins de responsabilidade
3. Os ativos de nuvem devem ser monitorados continuamente para oportunidades de otimização
4. As ferramentas de governança de nuvem devem limitar as opções de dimensionamento de ativo para uma lista aprovada de configurações. A ferramenta deve garantir que todos os ativos sejam detectáveis e controlados pelo custo de solução de monitoramento.
5. Durante o planejamento da implantação, os recursos de nuvem necessários associados à hospedagem de cargas de trabalho de produção devem ser documentados. Esta documentação ajudará a refinar os orçamentos e preparar automações adicionais para evitar o uso das opções mais caras. Durante esse processo, a consideração deve ser dada a diferentes ferramentas de descontos oferecidos pelo provedor de nuvem, como Instâncias Reservadas ou reduções de custo de licença.
6. Todos os aplicativos proprietários são necessários para participar de treinamentos sobre práticas para otimizar cargas de trabalho para melhor controlar os custos da nuvem.

## <a name="evolution-of-the-best-practices"></a>Evolução das práticas recomendadas

Esta seção do artigo evoluirá o design MVP de governança para incluir novas políticas do Azure e uma implementação do Gerenciamento de Custos do Azure. Juntas, essas duas alterações de design atenderão às novas instruções da política corporativa.

1. Alterações no Azure Enterprise Portal para cobrar o administrador de departamento para a implantação canadense.
2. Implemente o Gerenciamento de Custos do Azure.
    1. Estabeleça o nível correto de acesso para se alinhar ao padrão de assinatura e ao padrão de agrupamento de recursos. Supondo que o alinhamento com a governança MVP definido nos artigos anteriores, isso exigiria acesso ao **Escopo de Conta de Registro** para executar a equipe de governança de nuvem nos relatórios de alto nível. Equipes adicionais fora da governança, como a equipe de compras canadense, exigirá acesso ao **Escopo do Grupo de Recursos**.
    2. Estabelecer um orçamento no Gerenciamento de Custos do Azure.
    3. Examinar e agir em recomendações iniciais. É recomendável ter um processo recorrente para oferecer suporte ao processo de geração de relatórios.
    4. Configurar e executar Relatórios de Gerenciamento de Custos do Azure, tanto iniciais como recorrentes.
3. Atualize o Azure Policy.
    1. Audite a marcação, grupo de gerenciamento, assinatura e valores de grupo de recursos para identificar qualquer desvio.
    2. Estabeleça as opções de tamanho de SKU para limitar as implantações para SKUs listadas na documentação de planejamento de implantação.

## <a name="conclusion"></a>Conclusão

Adicionar esses processos e alterações ao MVP de governança ajuda a eliminar muitos dos riscos associados à governança de custo. Juntos, criam a visibilidade, a responsabilidade e a otimização necessárias para controlar os custos.

## <a name="next-steps"></a>Próximas etapas

Como a adoção da nuvem continua a se desenvolver e a tornar-se mais útil comercialmente, a governança de riscos e de nuvem também precisam evoluir. Para a empresa fictícia nessa jornada, a próxima etapa é usar esse investimento de governança para gerenciar várias nuvens.

> [!div class="nextstepaction"]
> [Evolução de várias nuvens](./multi-cloud-evolution.md)