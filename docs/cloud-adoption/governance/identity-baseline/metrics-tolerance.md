---
title: 'CAF: Métricas, indicadores e tolerância a riscos da linha de base de identidade'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Métricas, indicadores e tolerância a riscos da linha de base de identidade
author: BrianBlanchard
ms.openlocfilehash: 4722de66308f3d18885ca930925e68e0e756ec03
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242087"
---
# <a name="identity-baseline-metrics-indicators-and-risk-tolerance"></a>Métricas, indicadores e tolerância a riscos da linha de base de identidade

Este artigo tem como objetivo ajudar a quantificar a tolerância a riscos de negócios no que diz respeito à linha de base de identidade. Definir métricas e indicadores ajuda a criar um caso de negócios para investir no amadurecimento da disciplina Linha de Base de Identidade.

## <a name="metrics"></a>Métricas

A linha de base de identidade se concentra na identificação, autenticação e autorização de indivíduos, grupos de usuários ou processos automatizados e fornece a eles o devido acesso aos recursos em suas implantações de nuvem. Como parte da análise de risco, são coletados dados relacionados aos serviços de identidade para determinar qual o grau de risco enfrentado e quão importante é o investimento em governança na linha de base de identidade para suas implantações planejadas de nuvem.

Veja a seguir exemplos de métricas úteis que devem ser coletadas para ajudar a avaliar a tolerância a riscos na disciplina Linha de Base de Identidade:

- **Tamanho dos sistemas de identidade**. Quantidade total de usuários, grupos ou outros objetos gerenciados pelos sistemas de identidade.
- **Tamanho geral da infraestrutura de serviços de diretório**. Quantidade de florestas, domínios e locatários de diretório usados pela sua organização.
- **Dependência de mecanismos de autenticação herdados ou locais**. Quantidade de cargas de trabalho dependentes de mecanismos de autenticação herdados ou serviços de MFA (autenticação multifator) de terceiros.
- **Extensão dos serviços de diretório implantados na nuvem**. Quantidade de florestas, domínios e locatários de diretório implantados na nuvem.
- **Active Directories implantados na nuvem**. Quantidade de servidores do Active Directory implantados na nuvem.
- **Unidades organizacionais implantadas na nuvem**. Quantidade de unidades organizacionais do Active Directory implantadas na nuvem.
- **Extensão da federação**. Quantidade de sistemas de linha de base de identidade federados com sistemas de sua organização.  
- **Usuários elevados**. Quantidade de contas de usuário com acesso elevado para recursos ou ferramentas de gerenciamento.
- **Uso do RBAC**. A quantidade de assinaturas, grupos de recursos ou recursos individuais não são gerenciados por meio do RBAC (controle de acesso baseado em função).
- **Declarações de autenticação**. Quantidade de tentativas de autenticação de usuário bem-sucedidas e com falha.
- **Declarações de autorização**. Quantidades de tentativas bem-sucedidas e com falha de acesso a recursos feitas pelos usuários.
- **Contas comprometidas**. Quantidade de contas de usuário que foram comprometidas.

## <a name="risk-tolerance-indicators"></a>Indicadores de tolerância de risco

Os riscos relacionados à linha de base de identidade estão amplamente relacionados à complexidade da infraestrutura de identidades da organização. Se todos os usuários e grupos forem gerenciados usando um único diretório ou um provedor de identidade nativo de nuvem que utiliza a integração mínima com outros serviços, seu nível de risco provavelmente será pequeno. No entanto, como sua empresa precisa desenvolver os sistemas de linha de base de identidade, talvez seja necessário oferecer suporte a cenários mais complicados, como vários diretórios para dar suporte à organização ou federação interna com provedores de identidade externos. À medida que esses sistemas se tornam mais complexos, o risco aumenta.

Nos estágios iniciais de adoção da nuvem, trabalhe com sua equipe e segurança de TI e stakeholders de negócios para identificar [riscos de negócios](business-risks.md) relacionados à identidade e, em seguida, determine uma linha de base aceitável para a tolerância do risco de identidade. Esta seção de diretrizes de CAF fornece exemplos. No entanto, os riscos e linhas de base detalhados da empresa ou de suas implantações podem ser diferentes.

Assim que você tiver uma linha de base, estabeleça parâmetros de comparação mínimos que representem um aumento inaceitável em seus riscos identificados. Esses parâmetros de comparação atuam como gatilhos quando você precisar tomar medidas para atenuar esses riscos. Veja a seguir alguns exemplos de como métricas relacionadas à identidade, como as discutidas anteriormente, podem justificar o aumento de investimento na disciplina Linha de Base de Identidade.

- **Gatilho do número da conta de usuário**. Uma empresa com mais de X quantidade de usuários, grupos ou outros objetos gerenciados em seus sistemas de identidade poderia se beneficiar com o investimento na disciplina Linha de Base de Identidade para garantir a governança eficiente em uma grande quantidade de contas.
- **Gatilho de dependência de identidade local**. Uma empresa que planeja migrar cargas de trabalho para a nuvem que requeiram recursos de autenticação herdados ou MFA de terceiros deve investir na disciplina Linha de Base de Identidade para reduzir os riscos relacionados à refatoração ou à implantação de uma infraestrutura de nuvem adicional.
- **Gatilho de complexidade de serviços de diretório**. Uma empresa que mantém mais de X quantidade de locatários de diretório, florestas ou domínios individuais deve investir na disciplina Linha de Base de Identidade para reduzir os riscos relacionados ao gerenciamento de contas e problemas de eficiência relacionados a credenciais de vários usuários espalhadas entre vários sistemas.
- **Gatilho de serviços de diretório hospedados na nuvem**. Uma empresa que hospedada X quantidade de VMs (máquinas virtuais) de servidor do Active Directory hospedadas na nuvem ou que tem X quantidade de unidades organizacionais gerenciadas nesses servidores baseadas em nuvem pode se beneficiar com o investimento na disciplina Linha de Base de Identidade para otimizar a integração com quaisquer serviços de identidade locais ou externos.
- **Gatilho de federação**. Uma empresa que está implementando a federação de identidades com X quantidade de sistemas externos de linha de base de identidade pode se beneficiar com o investimento na disciplina Linha de Base de Identidade para garantir uma política organizacional consistente entre os membros da federação.
- **Gatilho de acesso elevado**. Uma empresa com mais de X% de usuários com permissões elevadas para recursos e ferramentas de gerenciamento deve levar em consideração o investimento na disciplina Linha de Base de Identidade para minimizar o risco de superprovisionamento inadvertido de acesso a usuários.
- **Gatilho do RBAC**. Uma empresa com em X% de recursos que usa os métodos de controle de acesso baseado em função deve levar em consideração o investimento na disciplina Linha de Base de Identidade para identificar maneiras otimizadas de atribuir o acesso do usuário aos recursos.
- **Gatilho de falha de autenticação**. Uma empresa cujas falhas de autenticação representam mais de X% das tentativas deve investir na disciplina Linha de Base de Identidade para garantir que os métodos de autenticação não fiquem sob ataque externo e que os usuários consigam usar os métodos de autenticação corretamente.
- **Gatilho de falha de autorização**. Uma empresa cujas tentativas de acesso são rejeitadas mais de X% das vezes devem investir na disciplina Linha de Base de Identidade para melhorar o aplicativo e a atualizar os controles de acesso, além de identificar tentativas de acesso possivelmente mal-intencionadas.
- **Gatilho da conta comprometida**. Uma empresa com mais de X quantidade de contas comprometidas deve investir na disciplina Linha de Base de Identidade para melhorar a força e a segurança dos mecanismos de autenticação e aprimorar os mecanismos para reduzir os riscos relacionados a contas comprometidas.

As métricas e os gatilhos exatos usados para medir a tolerância de risco e o nível de investimento na disciplina Linha de Base de Identidade serão específicos da organização. No entanto, os exemplos já apresentados devem servir como uma base útil para discussão dentro da equipe de governança de nuvem.

## <a name="next-steps"></a>Próximas etapas

Uso do [modelo de Gerenciamento de nuvem](./template.md), de métricas do documento e de indicadores de tolerância que se alinham ao atual plano de adoção da nuvem.

Usar riscos e tolerância como base e estabelecer um processo para administrar e comunicar a conformidade com a política de linha de base de identidade.

> [!div class="nextstepaction"]
> [Estabelecer os processos de conformidade com políticas](compliance-processes.md)
