---
title: 'CAF: Empresa de pequeno a médio porte - narrativa inicial por trás da estratégia de governança'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Esta narrativa estabelece um caso de uso para um percurso de governança de empresa de pequeno a médio porte.
author: BrianBlanchard
ms.openlocfilehash: 6173b01f310169017761110d6641acfa51d45df8
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900422"
---
# <a name="small-to-medium-enterprise-the-narrative-behind-the-governance-strategy"></a>Empresas de pequeno a médio porte: A narrativa por trás da estratégia de governança

A narrativa a seguir descreve o caso de uso para o [percurso de governança de empresa de pequeno a médio porte](./overview.md). Antes de implementar a jornada, é importante entender as suposições e o raciocínio que são refletidos nessa narrativa. Em seguida, você pode melhorar o alinhamento da estratégia de governança para a jornada da sua própria organização.

## <a name="back-story"></a>Histórico

O conselho de diretores começou o ano com planos para acionar os negócios de várias maneiras. Eles estão enviando por push liderança para melhorar as experiências de cliente para obter a participação no mercado. Eles também estão enviando por push para novos produtos e serviços que irão posicionar a empresa como um líder de ideias no setor. Também iniciaram um esforço paralelo para reduzir o desperdício e cortar custos desnecessários. Embora intimidantes, as ações da placa e liderança mostram que esse esforço está concentrando o maior número de capital possível em crescimento futuro.

No passado, CIO da empresa foi excluído dessas conversas estratégicas. No entanto, como a visão futura intrinsecamente está vinculada ao crescimento técnico, a TI tem uma vaga para ajudar a orientar essas grandes planos. A IT agora é esperada para entregar de novas maneiras. A equipe não está realmente preparada para essas alterações e é provável que tenha dificuldades com a curva de aprendizado.

## <a name="business-characteristics"></a>Características de negócios

A empresa tem o seguinte perfil de negócios:

- Todas as operações e vendas residem em um único país, com uma porcentagem baixa de clientes globais.
- A empresa opera como uma única unidade de negócios, com um orçamento alinhado a funções, incluindo vendas, marketing, operações e TI.
- A empresa vê a maior parte da TI como um dreno de capital ou centro de custo.

## <a name="current-state"></a>Estado atual

Veja o estado atual da TI da empresa e das operações de nuvem:

- A TI opera com dois ambientes de infraestrutura hospedada. Um ambiente contém ativos de produção. O segundo ambiente contém alguns ativos de desenvolvimento/teste e recuperação de desastres. Esses ambientes são hospedados por dois provedores diferentes. A TI se refere a esses dois datacenters como produção e recuperação de Desastre, respectivamente.
- A TI inseriu a nuvem migrando todas as contas de email de usuário final para o Office 365. Essa migração foi concluída há seis meses. Apenas alguns outros ativos de TI foram implantados na nuvem.
- As equipes de desenvolvimento de aplicativo estão trabalhando em uma capacidade de desenvolvimento/teste para saber mais sobre os recursos nativos da nuvem.
- A equipe do business intelligence (BI) está experimentando big data na nuvem e curadoria dos dados em novas plataformas.
- A empresa tem uma política vagamente definida informando as informações de identificação pessoal (PII) desse cliente e os dados financeiros não podem ser hospedados na nuvem, o que limita a aplicativos de missão crítica nas implantações atuais.
- Os investimentos de TI são controlados por despesas de capital (CapEx). Esses investimentos são planejados anualmente. Nos últimos anos, investimentos incluíram pouco requisitos de manutenção mais básicos.

## <a name="future-state"></a>Estado futuro

As seguintes alterações estão previstas nos próximos vários anos:

- O CIO está examinando a política de PII e dados financeiros para permitir as metas de estado futuro.
- As equipes de BI e o desenvolvimento de aplicativos desejam liberar soluções com base em nuvem para produção nos próximos 24 meses com base na visão do compromisso com o cliente e novos produtos.
- Este ano, a equipe de TI concluirá a retirada das cargas de trabalho de recuperação de desastre do datacenter de recuperação de desastre, migrando 2.000 VMs para a nuvem. Isso é esperado para produzir uma economia de custo estimado de 25 milhões de dólares nos próximos cinco anos.
    ![Os custos em comparação aos custos do Azure demonstram um retorno de 25 milhões de dólares nos próximos cinco anos](../../../_images/governance/calculator-small-to-medium-enterprise.png)
- Os planos da empresa para alterar como fazer os investimentos de TI reposicionando o CapEx compromissado com uma despesa operacional (OpEx) dentro da TI. Essa alteração fornecerá maior controle de custo e habilitará a TI para acelerar outros esforços planejados.

## <a name="next-steps"></a>Próximas etapas

A empresa desenvolveu uma política corporativa para formar a implementação de governança. A política corporativa conduz muitas das decisões técnicas.

> [!div class="nextstepaction"]
> [Examinar a política corporativa inicial](./initial-corporate-policy.md)
