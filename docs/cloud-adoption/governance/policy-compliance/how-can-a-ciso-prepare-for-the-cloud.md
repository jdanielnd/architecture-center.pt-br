---
title: 'CAF: Preparação do CISO'
description: Como um diretor de segurança da informação pode se preparar para a nuvem?
author: BrianBlanchard
ms.date: 10/03/2018
ms.openlocfilehash: a4535163990797decdacdacdcb6a33f0118366e9
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58238349"
---
# <a name="ciso-cloud-readiness-guide"></a>Guia de preparação da nuvem CISO

As Diretrizes da Microsoft, como o Azure Cloud Adoption Framework(CAF) não estão posicionadas para determinar ou guiar as restrições de segurança exclusivas de milhares de empresas com suporte nesta documentação. Ao mudar para a nuvem, o papel do Chief Information Security Officer ou do Chief Information Security Office (CISO) não é suplantado pelas tecnologias de nuvem. Muito pelo contrário, o CISO e o escritório de CISO, se tornam mais integrantes e integrados. Este guia pressupõe que o leitor esteja familiarizado com os processos CISO e está procurando modernizar esses processos para permitir a transformação de nuvem.

A adoção da nuvem permite que os serviços que não sejam considerados geralmente em ambientes de TI tradicionais. Implantações automatizadas ou serviços automatizados são normalmente executadas pelo desenvolvimento de aplicativos ou outras equipes de TI não tradicionalmente alinhado à implantação de produção. Em algumas organizações, aspectos relacionados aos negócios da mesma forma têm recursos de autoatendimento. Isso pode disparar novos requisitos de segurança que não eram necessários no mundo local. Segurança centralizada é mais desafiador, a segurança muitas vezes se torna uma responsabilidade compartilhada entre os negócios e a cultura de TI. Este artigo pode ajudar um CISO a se preparar para essa abordagem e participar de governança incremental.

## <a name="how-can-the-ciso-prepare-for-the-cloud"></a>Como um CISO, pode se preparar para a nuvem?

Como a maioria das políticas, as políticas de segurança e governança de dentro de uma organização tendem a aumentar organicamente. Quando ocorrerem incidentes de segurança, eles formam a política para informar os usuários e reduzir a probabilidade de ocorrências repetidas. Enquanto natural, essa abordagem cria inchaço de política e dependências técnicas. Percursos de transformação de nuvem Crie uma oportunidade única para modernizar e políticas de redefinição. Durante a preparação para qualquer jornada de transformação, o CISO pode criar valor imediato e mensurável, servindo como o principal interessado stakeholder em uma [revisão da política](./what-is-a-cloud-policy-review.md).

Em revisão, a função do CISO é criar um equilíbrio seguro entre as restrições de política existente/conformidade e a postura de segurança aprimorada de provedores de nuvem. Medir o progresso pode ter várias formas, geralmente é medido no número de políticas de segurança que podem ser descarregadas com segurança para o provedor de nuvem.

**Transferindo os riscos de segurança**: À medida que os serviços são movidos para modelos de hospedagem de infraestrutura como um serviço (IaaS), o negócio assume menos riscos diretos em relação ao provisionamento de hardware. O risco não é removido, em vez disso, ele é transferido para o fornecedor de nuvem. Se a abordagem de um fornecedor de nuvem ao provisionamento de hardware fornecer o mesmo nível de mitigação de risco, em um processo repetitivo seguro, o risco de execução de provisionamento de hardware será removido da política corporativa. Ela agora pode ser substituída por uma nova política para validar esses processos, mas se o risco de execução for realocado, reduzirá o risco geral de segurança.

Como soluções ainda mais movem "para cima", incorporam plataforma como serviço (PaaS) ou software como serviço (SaaS) de modelos, os riscos adicionais podem ser atenuados, transferidos e substituídos. Quando o risco com segurança é movido para um provedor de nuvem, o custo de executar, monitorar e impor políticas de segurança ou outras políticas de conformidade podem ser reduzidas com segurança também.

**Mentalidade de crescimento**: Mudança pode ser assustadora para os negócios, bem como, implementações técnicas. Quando o CISO leva a uma mudança de mentalidade de crescimento em uma organização, descobrimos que os temores naturais são substituídos por um aumento de interesse em segurança e política de conformidade. Se aproximar uma [revisão da política](./what-is-a-cloud-policy-review.md), uma Jornada de Transformação, ou simples revisões de implementação com uma mentalidade de crescimento, permite que a equipe se mova rapidamente, mas não às custas de um perfil de perfil de risco justo e manejável.

## <a name="resources-for-the-chief-information-security-officer"></a>Recursos para Chief Information Security Officer

Saber conhecimento sore a nuvem é fundamental para abordar uma [revisão da política](./what-is-a-cloud-policy-review.md) com uma mentalidade de crescimento. Os recursos a seguir podem ajudar o CISO a entender melhor a postura de segurança da plataforma do Azure da Microsoft.

Recursos de plataforma de segurança:

* [Ciclo de Desenvolvimento de Segurança](https://www.microsoft.com/sdl/), auditorias internas
* [Treinamento de segurança obrigatório, verificações em segundo plano](https://downloads.cloudsecurityalliance.org/star/self-assessment/StandardResponsetoRequestforInformationWindowsAzureSecurityPrivacy.docx)
* [Testes de penetração, Detecção de intrusão, DDoS, Auditorias e registro em log](https://www.microsoft.com/trustcenter/Security/AuditingAndLogging)
* [Centro de dados de última geração](https://www.microsoft.com/cloud-platform/global-datacenters), a segurança física, [rede segura](/azure/security/security-network-overview)
* Saiba mais em [Resposta da segurança do Microsoft Azure na nuvem (PDF)](https://aka.ms/SecurityResponsePaper)

Privacidade e cookies:

* [Gerenciar sempre seus dados](https://www.microsoft.com/trustcenter/Privacy/You-own-your-data)
* [Controlar o local dos dados](https://www.microsoft.com/trustcenter/Privacy/Where-your-data-is-located)
* [Fornecer acesso a dados em seus próprios termos](https://www.microsoft.com/trustcenter/Privacy/Who-can-access-your-data-and-on-what-terms)
* [Respondendo às autoridades competentes](https://www.microsoft.com/trustcenter/Privacy/Responding-to-govt-agency-requests-for-customer-data)
* [Padrões de privacidade rigorosos](https://www.microsoft.com/TrustCenter/Privacy/We-set-and-adhere-to-stringent-standards)

Conformidade:

* [Central de Confiabilidade](https://www.microsoft.com/trustcenter/default.aspx)
* [Hub de Controles Comuns](https://www.microsoft.com/trustcenter/Common-Controls-Hub)
* [Lista de verificação de inspeção dos Serviços de Nuvem ](https://www.microsoft.com/trustcenter/Compliance/Due-Diligence-Checklist)
* [Conformidade por serviço, local e setor](https://www.microsoft.com/trustcenter/Compliance/default.aspx)

Transparência:

* [Como a Microsoft protege os dados do cliente nos serviços do Azure](https://www.microsoft.com/trustcenter/Transparency/default.aspx)
* [Como a Microsoft gerencia o local de dados nos serviços do Azure](https://azuredatacentermap.azurewebsites.net/)
* [Quem na Microsoft pode acessar seus dados e em quais termos](https://www.microsoft.com/trustcenter/Privacy/Who-can-access-your-data-and-on-what-terms)
* [Como a Microsoft protege os dados do cliente nos serviços do Azure](https://www.microsoft.com/trustcenter/Transparency/default.aspx)
* [Revisar a certificação para serviços do Azure, hub de transparência](https://www.microsoft.com/trustcenter/Compliance/default.aspx)

## <a name="next-steps"></a>Próximas etapas

A primeira etapa para executar uma ação em qualquer estratégia de governança, é uma [revisão da política](./what-is-a-cloud-policy-review.md). [Política e conformidade](./overview.md) poderiam ser um guia útil durante sua revisão de política.

> [!div class="nextstepaction"]
> [Preparar para uma revisão de política](./what-is-a-cloud-policy-review.md)