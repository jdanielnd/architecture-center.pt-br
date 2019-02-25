---
title: 'CAF: O que é a Linha de Base de Segurança de Nuvem'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 10/10/2018
description: O que é a Linha de Base de Segurança de Nuvem?
author: BrianBlanchard
ms.openlocfilehash: cb6e3372fd76486e5c4467ef822163eac0499573
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900282"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-the-cloud-security-baseline"></a>O que é a Linha de Base de Segurança de Nuvem?

Este é um artigo introdutório sobre o tópico geral da Linha de Base de Segurança de Nuvem, que baseia-se nas [Cinco disciplinas de Governança de Nuvem](../governance-disciplines.md) para estabelecer uma estrutura de governança. Informações mais detalhadas sobre Segurança de Nuvem estão disponíveis na [Nuvem Confiável do Azure](https://azure.microsoft.com/overview/trusted-cloud/). Abordagens para melhorar as condições de segurança da organização podem ser localizadas no [Catálogo de Serviços de Segurança de Nuvem](https://www.microsoft.com/security/information-protection)

> [!NOTE]
> Este artigo não disponibiliza contexto suficiente para permitir que o leitor implemente uma estratégia de segurança. É apenas para conhecimento geral.

## <a name="cloud-security"></a>Segurança de Nuvem

A segurança de nuvem é uma extensão das práticas tradicionais de segurança da informação. A segurança de TI tradicional incluiria políticas e controles que controlam a segurança do computador, segurança de rede, proteção de dados, uso de informações, e assim por diante. Essas mesmas políticas e controles são necessários na nuvem. Durante qualquer Transformação de Nuvem, será determinante que o CISO esteja envolvido ativamente e compreenda o cenário da nuvem para garantir que as políticas de TI herdadas sejam mapeadas para níveis adequados de controle na nuvem.

No mínimo, qualquer estratégia de segurança de nuvem deve considerar os seguintes tópicos:

* Classificar dados: Classificação de dados apropriada para reconhecer todas as fontes de dados privados, protegidos ou altamente confidenciais. Isso ajudará a gerenciar risco durante o planejamento.
* Planejar um cenário de nuvem híbrida: Reconhecer como as redes locais herdadas serão conectadas à nuvem ajudará o CISO a identificar e mitigar os riscos.
* Plano para ataques e erros: Nos primeiros meses de uma transformação, erros ocorrerão conforme a equipe vai aprendendo continuamente. Inicie com implantações de baixo risco e altamente restritas para aprender com segurança.
* Priorizar a privacidade: Durante toda a transformação, toda a equipe deverá manter a privacidade do cliente e dos funcionários como prioridade. Os dados estarão seguros na nuvem, mas a equipe deverá estar ciente e ser extremamente cautelosa ao lidar com dados confidenciais.

## <a name="protecting-data-and-privacy"></a>Proteger dados e privacidade

Para organizações em todo o mundo, &mdash; se governos, organizações sem fins lucrativos ou empresas &mdash; de computação em nuvem tornarem-se uma parte fundamental da estratégia de TI contínua. Os serviços de nuvem permitem que as organizações de todos os portes acessem virtualmente o armazenamento de dados ilimitados, enquanto as desobriga da necessidade de comprar, manter e atualizar suas próprias redes e sistemas de computadores. A Microsoft e outros provedores de nuvem oferecem infraestrutura de TI, plataforma e SaaS (software como serviço), permitindo aos clientes escalar ou reduzir verticalmente de maneira muito rápida, conforme necessário, e pagar apenas pela capacidade de computação e armazenamento que utilizarem.

No entanto, como as organizações continuam beneficiando-se dos serviços de nuvem, como escolha, agilidade e flexibilidade maiores, ao mesmo tempo que aumentam a eficiência e diminuem o custo de TI, deverão considerar como a introdução de serviços de nuvem afetará as condições de privacidade, segurança e conformidade. A Microsoft tem trabalhado para tornar as ofertas de nuvem não apenas escalonáveis, confiáveis e gerenciáveis, mas também, para garantir que os dados de nossos clientes sejam protegidos e utilizados de maneira transparente.

A segurança é um elemento fundamental para garantir proteção dos dados robusta em todos os ambientes de computação online. Porém, a segurança sozinha não é suficiente. A predisposição que os consumidores e as empresas têm para utilizar um determinado produto de computação em nuvem, também depende da capacidade de confiarem que a privacidade de suas informações será protegida e que seus dados serão utilizados somente de maneira compatível com as expectativas do cliente. Saiba mais sobre a abordagem da Microsoft para [Proteção de dados e privacidade na nuvem](https://go.microsoft.com/fwlink/?LinkId=808242&clcid=0x409)

## <a name="risk-mitigation"></a>Mitigação de risco

Os dois maiores riscos em qualquer datacenter podem ser agrupados em duas origens: Sistemas obsoletos e erro humano. Proteger contra esses dois riscos é o mínimo necessário ao definir uma estratégia de segurança de TI. O mesmo aplica-se para a nuvem. A seguir, são apresentados alguns exemplos de controles que podem ser implementados para mitigar riscos e fortalecer a estratégia de Segurança de Nuvem.

* Sistemas herdados: Muitos componentes em soluções de datacenter locais consistem em software, hardware e processos que antecedem os riscos de segurança atuais. Quando possível, corrija, substitua ou desative esses sistemas durante uma Transformação de Nuvem. Evidentemente, isso nem sempre será possível ou viável. Se sistemas herdados permanecerem em produção em uma solução híbrida, é importante que esses sistemas sejam inventariados e compreendidos durante o projeto do Datacenter Virtual. Isso permitirá que a equipe de projeto elimine ou controle o acesso a esses sistemas a partir da nuvem.
* Processos e controles de segurança de TI: No mínimo, as equipes de projeto da Nuvem deverão estar atualizadas com os processos e controles de segurança de TI existentes para encaminhá-los para a nuvem. Em um cenário ideal, um membro da equipe de segurança de TI seria treinado em segurança cibernética e destinado como membro das equipes de projeto e implementação.
* Monitoramento e auditoria: Ao projetar processos e ferramentas de governança, certifique-se de que as soluções de monitoramento e auditoria incluam riscos ou violações de segurança.

> [!NOTE]
> Divulgação de dívida técnica: Este tópico não possui as próximas etapas acionáveis. Artigos adicionais desenvolverão esse tópico ao longo do tempo. Informações mais detalhadas sobre Segurança de Nuvem estão disponíveis na [Nuvem Confiável do Azure](https://azure.microsoft.com/overview/trusted-cloud/). Abordagens para melhorar as condições de segurança da organização podem ser localizadas no [Catálogo de Serviços de Segurança de Nuvem](https://www.microsoft.com/security/information-protection)