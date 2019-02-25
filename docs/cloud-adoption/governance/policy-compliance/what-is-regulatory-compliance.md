---
title: 'CAF: Introdução à conformidade regulatória'
description: O que é conformidade regulatória
author: BrianBlanchard
ms.date: 2/8/2019
ms.openlocfilehash: 5ff7b591d7cab647a99cee223e6271928e185a2f
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900381"
---
# <a name="introduction-to-regulatory-compliance"></a>Introdução à conformidade regulatória

Este é um artigo introdutório sobre conformidade regulatória, portanto, não destina-se a implementar uma estratégia de conformidade. É apenas para conhecimento geral. Informações mais detalhadas sobre as [Ofertas de conformidade do Azure](https://aka.ms/allcompliance) estão disponíveis na [Central de Confiabilidade da Microsoft]. Além disso, toda a documentação disponível para download está disponível para clientes do Azure sob um contrato de confidencialidade do [Portal de Confiança do Serviço da Microsoft](https://servicetrust.microsoft.com/).

Conformidade regulatória refere-se à disciplina e ao processo para garantir que uma empresa cumpra as leis estabelecidas por órgãos diretivos em sua geografia ou setor. Para conformidade regulatória de TI, pessoas e/ou processos monitoram os sistemas corporativos em um esforço para detectar e evitar violações de políticas e procedimentos estabelecidos pelas regulamentações e legislações vigentes. Isso, por sua vez, aplica-se a uma área muito ampla de processos de monitoramento e cumprimento. Dependendo do setor e da geografia, esses processos podem se tornar bastante longos e complexos.

Para organizações multinacionais (particularmente aquelas em setores altamente regulamentados, como serviços financeiros e de saúde), a conformidade pode ser muito exigente. Normas e regulamentações existem em grande número e, naturalmente, elas mudam com frequência, tornando difícil para as empresas manterem-se informadas sobre a evolução das leis internacionais que regem o tratamento de dados eletrônicos.

Assim como nos controles de segurança, as organizações devem compreender a divisão de responsabilidades relacionadas à conformidade regulatória na nuvem. Os provedores de nuvem estão empenhados para garantir que as plataformas e serviços estejam em conformidade. Mas, as organizações também precisam assegurar que seus próprios aplicativos, e aqueles fornecidos por terceiros, são compatíveis. Da mesma forma, aplicativos em setores regulamentados que usam serviços de nuvem podem exigir certificação do provedor de nuvem.

A seguir, estão as descrições de regulamentos de conformidade em vários setores e geografias:

## <a name="hipaa"></a>HIPAA

Um aplicativo para serviços de saúde que processa PHI (informações de saúde protegidas) está sujeito tanto à Regra de Privacidade e como à Regra de Segurança englobadas na Lei HIPAA (Lei de Portabilidade e Responsabilidade de Seguros de Saúde). No mínimo, a HIPAA provavelmente exigirá que uma empresa de serviços de saúde receba garantias por escrito do provedor de nuvem de que protegerá qualquer PHI recebida ou criada.

## <a name="pci"></a>PCI

PCI DSS (Padrão de Segurança de Dados da Indústria de Pagamento com Cartão) é um padrão de segurança de informações patenteado para organizações que lidam com cartões de crédito de marca dos principais sistemas de cartões, incluindo Visa, MasterCard, American Express, Discover e JCB. O padrão PCI é exigido pelas marcas de cartões e administrado pelo Conselho de Normas de Segurança da Indústria de Meios de Pagamento. O padrão foi criado para aumentar os controles sobre os dados do titular do cartão para reduzir fraudes com cartões de crédito. A validação da conformidade é realizada anualmente, seja por um QSA (Assessor de Segurança Qualificado) externo ou por um ISA (Assessor de Segurança Interno) específico da empresa que cria um ROC (Relatório de Conformidade) para organizações que lidam com grandes volumes de transações ou por um SAQ (Questionário de Autoavaliação) para empresas.

## <a name="pii"></a>PII

PII (Informações de Identificação Pessoal) são todos os pontos de dados que podem ser utilizados para identificar um consumidor, funcionário, parceiro ou qualquer outra entidade legal ou viva. Muitas leis emergentes, particularmente aquelas que lidam com privacidade e PII individual, exigem que as próprias empresas cumpram e reportem a conformidade e quaisquer violações que possam ocorrer.

## <a name="gdpr"></a>GDPR

Um dos desenvolvimentos mais importantes nesta área é a recente promulgação pela Comissão Europeia do GDPR (Regulamento Geral sobre a Proteção de Dados), concebido para reforçar a proteção de dados para as pessoas dentro da União Europeia. O GDPR exige que dados pessoais como "um nome, endereço residencial, uma foto, endereço de email, dados bancários, postagens em sites de redes sociais, informações médicas ou endereço IP de um computador", sejam mantidos nos servidores dentro da UE e não transferidos para fora dela. Também exige que as empresas notifiquem as pessoas sobre quaisquer violações de dados e determina que as empresas tenham um Responsável pela Proteção de Dados. Outros países têm ou estão desenvolvendo tipos similares de regulamentos.

## <a name="compliant-foundation-in-azure"></a>Base compatível no Azure

Para ajudar os clientes a cumprirem suas próprias obrigações de conformidade em setores e mercados regulamentados em todo o mundo, o Azure mantém o maior portfólio de conformidade do setor &mdash; em amplitude (número total de ofertas), bem como profundidade (número de serviços voltados ao cliente no escopo da avaliação). As ofertas de conformidade do Azure são agrupadas em quatro segmentos: aplicável globalmente, US Government, específico do setor e específico do região/país.

As ofertas de conformidade do Azure são baseadas em vários tipos de garantias, incluindo certificações formais, atestados, validações, autorizações e avaliações produzidas por empresas de auditoria de terceiros independentes, bem como alterações contratuais, autoavaliações e documentos de diretrizes para o cliente produzidos pela Microsoft. Cada descrição de oferta neste documento fornece uma declaração de escopo atualizada indicando quais serviços voltados ao cliente do Azure estão no escopo da avaliação, assim como links para recursos baixáveis para auxiliar os clientes com suas próprias obrigações de conformidade.

Informações mais detalhadas sobre as ofertas de conformidade do Azure estão disponíveis na [Central de Confiabilidade da Microsoft](/trustcenter/compliance/complianceofferings). Adicionalmente, toda a documentação baixável está disponível para clientes do Azure sob um contrato de confidencialidade no [Portal de Confiança do Serviço](https://servicetrust.microsoft.com) nas seções a seguir:

* Relatórios de auditoria: Inclui as seções de relatórios FedRAMP, avaliação de GRC, ISO, PCI DSS e SOC
* Recursos de proteção de dados: Inclui guias de conformidade, perguntas frequentes, white papers e seções de avaliação de segurança e teste de penetração
