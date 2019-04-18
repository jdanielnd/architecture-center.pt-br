---
title: 'CAF: Criar consistência de nuvem híbrida'
description: Definir a abordagem para criar a consistência de nuvem híbrida
author: BrianBlanchard
ms.date: 12/27/2018
ms.openlocfilehash: d5cfc8565a97c0342b5dc200512308d4c795422a
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639911"
---
# <a name="create-hybrid-cloud-consistency"></a>Criar consistência de nuvem híbrida

Este artigo orienta você quanto às abordagens de alto nível para a criação de consistência de nuvem híbrida.

Os modelos de implantação híbrida durante a migração podem reduzir o risco e contribuir para uma transição da infraestrutura sem problemas. Em relação aos processos de negócios, as plataformas de nuvem oferecem o maior nível de flexibilidade. Muitas organizações são reticentes quanto a fazer a transição para a nuvem, preferindo manter controle total sobre os dados mais confidenciais. Infelizmente, os servidores locais não permitem ter a mesma taxa de inovação como a nuvem. Uma solução de nuvem híbrida possibilita a você o melhor dos dois cenários: a velocidade de inovação na nuvem E o conforto do gerenciamento local.

## <a name="integrate-hybrid-cloud-consistency"></a>Integrar a consistência da nuvem híbrida

O uso de uma solução de nuvem híbrida permite que as organizações dimensionem os recursos de computação. Isso também elimina a necessidade de fazer grandes investimentos de capital para lidar com picos de demanda de curto prazo. Quando alterações aos seus negócios levam à necessidade de liberar recursos locais para mais dados confidenciais ou aplicativos, o desprovisionamento de recursos de nuvem fica mais fácil, rápido e com menos custos. Você paga apenas pelos recursos que sua organização usa temporariamente em vez de precisar comprar e fazer a manutenção de recursos adicionais. Isso reduz a quantidade de equipamentos que podem permanecer inativos durante longos períodos de tempo. Nuvem híbrida é uma plataforma de "melhor de todos os mundos possíveis", fornecendo todos os benefícios da flexibilidade, escalabilidade e eficiências de custo; de computação em nuvem de computação tudo isso com o mais baixo possível risco de exposição de dados.

![Criação da consistência de nuvem híbrida entre identidades, gerenciamentos, segurança, dados, desenvolvimentos e DevOps](../../_images/hybrid-consistency.png)
*Figura 1. Criação da consistência de nuvem híbrida entre identidades, gerenciamentos, segurança, dados, desenvolvimentos e DevOps*

Uma verdadeira solução de nuvem híbrida deve fornecer quatro componentes, com cada um oferecendo benefícios significativos, incluindo:

- Identidade comum para aplicativos locais e em nuvem: melhora a produtividade do usuário ao fornecer a ele um SSO (logon único) para todos os seus aplicativos. Também é garantida a consistência conforme aplicativos e usuários ultrapassam as fronteiras da rede/nuvem.
- Gerenciamento e segurança integrados em sua nuvem híbrida: fornece uma maneira coesa para monitorar, gerenciar e proteger o ambiente, permitindo uma maior visibilidade e controle.
- Uma plataforma consistente de dados para o datacenter e a nuvem: cria a portabilidade de dados, combinada com o acesso contínuo serviços locais e de dados de nuvem para uma profunda compreensão de todas as fontes de dados.
- Desenvolvimento e DevOps unificados entre os datacenters de nuvem e locais: permite que você mova os aplicativos entre os dois ambientes conforme o necessário, melhorando a produtividade do desenvolvedor, pois agora ambos os locais têm o mesmo ambiente de desenvolvimento.
  
Os exemplos desses componentes de uma perspectiva do Azure incluem:

- o Azure AD (Azure Active Directory), que funciona com o Azure AD local para fornecer uma identidade comum a todos os usuários. O SSO entre o local e por meio da nuvem facilita o acesso seguro dos usuários aos aplicativos e ativos de que precisam. Os administradores podem gerenciar a segurança e controles de governança para que os usuários possam acessar o que eles precisam, com flexibilidade para ajustar essas permissões sem afetar a experiência do usuário.
- O Azure oferece serviços integrados de gerenciamento e segurança para as infraestruturas de nuvem e local que incluem um conjunto integrado de ferramentas para monitorar, configurar e proteger nuvens híbridas. Essa abordagem de ponta a ponta do gerenciamento aborda especificamente os desafios do mundo real que as organizações enfrentam ao levar uma solução de nuvem híbrida em consideração.
- Uma nuvem híbrida do Azure fornece ferramentas comuns que garantem acesso seguro a todos os dados, de modo perfeito e eficiente. Os serviços de dados do Azure se unem ao Microsoft SQL Server para criar uma plataforma de dados consistente. Um modelo de nuvem híbrida consistente permite que os usuários trabalhem com dados operacionais e analíticos, fornecendo os mesmos serviços locais e na nuvem para data warehouse, análise de dados e visualização de dados.
- Os serviços de nuvem do Microsoft Azure, combinados com o Microsoft Azure Stack local, fornecem desenvolvimento e DevOps unificados. A consistência entre a nuvem e o local significa que sua equipe de DevOps pode criar aplicativos que sejam executados em qualquer um desses ambientes e podem fazer facilmente a implantação ao local correto. Você também pode reutilizar os modelos em toda a solução híbrida, o que pode simplificar ainda mais os processos do DevOps.

## <a name="azure-stack-in-a-hybrid-cloud-environment"></a>Azure Stack em um ambiente de nuvem híbrida

O Microsoft Azure Stack é uma solução de nuvem híbrida que permite que organizações executem serviços consistentes do Azure em seus datacenters, fornecendo uma experiência simplificada de desenvolvimento, de gerenciamento e de segurança que seja consistente com os serviços de nuvem pública do Azure. O Azure Stack é uma extensão do Azure, o que permite a execução dos serviços do Azure a partir de seus ambientes locais e mover para a nuvem do Azure se e quando necessário.

O Azure Stack permite que você implante e opere o IaaS e o PaaS usando as mesmas ferramentas e oferecendo a mesma experiência de nuvem pública do Azure. O gerenciamento do Azure Stack, seja por meio do portal de interface do usuário da Web ou por meio do PowerShell, tem uma aparência consistente para os administradores de TI e usuários finais do Azure.

O Azure e o Azure Stack desbloqueiam novos casos de uso híbrido para aplicativos voltados ao cliente e internos de linha de negócios, incluindo:

- **Soluções desconectadas e de borda**. Os clientes podem lidar com os requisitos de latência e conectividade processando dados localmente no Azure Stack e então agregando-os ao Azure para análise adicional, com uma lógica comum do aplicativo entre ambos. Há muitos clientes interessados nesse cenário de borda em diferentes contextos, incluindo chão de fábrica, cruzeiros e minas.
- **Aplicativos de nuvem que atendem a diversos regulamentos**. Os clientes podem desenvolver e implantar aplicativos no Azure com total flexibilidade para realizar a implantar local no Azure Stack para atender aos requisitos regulatórios ou de política sem a necessidade de alterações no código. Entre os exemplos ilustrativos de aplicativos, há auditorias globais, relatórios financeiros, intercâmbios comerciais internacionais, jogos online e relatórios de despesas. Algumas vezes, os clientes querem implantar instâncias diferentes do mesmo aplicativo no Azure ou Azure Stack com base em requisitos técnicos e de negócios. Embora o Azure cumpra a maioria dos requisitos, o Azure Stack complementa a abordagem de implantação quando necessário.
- **Modelo de aplicativo de nuvem local**. Os clientes podem usar os serviços Web, os contêineres e as arquiteturas sem servidor e de microsserviço do Azure para atualizar e estender aplicativos existentes ou criar novos. É possível usar processos consistentes de DevOps entre o Azure na nuvem e o Azure Stack local. Há um crescente interesse na modernização de aplicativos, incluindo aplicativos principais de missão crítica.

O Azure Stack é oferecido com duas opções de implantação:

- **Sistemas integrados do Azure Stack**. Os sistemas integrados do Azure Stack são oferecidos por meio de uma parceria entre Microsoft e os parceiros de hardware, criando uma solução que fornece uma inovação conduzida na nuvem equilibrada com uma simplicidade no gerenciamento. Como o Azure Stack é oferecido como um sistema integrado de hardware e software, você obtém a quantidade certa de flexibilidade e controle, enquanto ainda adota a inovação a partir da nuvem. Os sistemas integrados do Azure Stack variam de 4 a 12 nós de tamanho e conta com um suporte conjunto da Microsoft e dos parceiros de hardware. Use sistemas integrados do Azure Stack para habilitar novos cenários para suas cargas de trabalho de produção.
- **Kit de Desenvolvimento do Azure Stack**. O Kit de Desenvolvimento do Microsoft Azure Stack é uma implantação de nó único do Azure Stack que você pode usar para avaliar e saber mais sobre essa extensão. Você também pode usar o kit como um ambiente de desenvolvedor, no qual é possível desenvolver usando as APIs e ferramentas consistentes com o Azure. O Kit de Desenvolvimento do Azure Stack não foi projetado para ser usado como um ambiente de produção.

## <a name="azure-stack-one-cloud-ecosystem"></a>Ecossistema de nuvem única do Azure Stack

Você pode acelerar iniciativas de pilha do Azure usando o ecossistema do Azure completo:

- O Azure garante que a maioria dos aplicativos e serviços certificados do Azure funcionará no Azure Stack. Vários ISVs &mdash; incluindo Bitnami, Docker, Kemp Technologies, Pivotal Cloud Foundry, Red Hat Enterprise Linux e SUSE Linux &mdash; estão estendendo suas soluções ao Azure Stack.
- Você pode optar por ter o Azure Stack entregue e operado como um serviço totalmente gerenciado. Vários parceiros &mdash; incluindo Tieto, Yourhosting, Revera, Pulsant e NTT &mdash; contarão com as ofertas de serviço gerenciadas no Azure e Azure Stack em breve. Esses parceiros têm fornecido serviços gerenciados do Azure por meio do programa de Provedor de soluções de nuvem (Provedores de nuvem) e agora estão ampliando suas ofertas para incluir soluções híbridas.
- Como exemplo de uma solução de nuvem híbrida completa e totalmente gerenciada, a Avanade fornece uma oferta de all-in-one que inclui serviços de transformação de nuvem, software, instalação e configuração, infraestrutura e serviços gerenciados contínuos para que os clientes possam usar o Azure Stack exatamente como fazem com o Azure atualmente.
- Os SIs (Integradores de sistemas) podem ajudar a acelerar as iniciativas de modernização de aplicativos com a criação de soluções de ponta a ponta do Azure para clientes. Eles usam conjuntos detalhados de habilidades do Azure, conhecimento sobre o setor e o domínio e experiência de processo (por exemplo, DevOps). Cada nuvem do Azure Stack é uma oportunidade para um SI criar a solução, liderar e influenciar a implantação de sistema, personalizar as funcionalidades incluídas e entregar atividades operacionais. Isso inclui SIs como Avanade, DXC, Dell EMC Services, InFront Consulting Group, HPE Pointnext e Pricewaterhouse Coopers (PwC).
