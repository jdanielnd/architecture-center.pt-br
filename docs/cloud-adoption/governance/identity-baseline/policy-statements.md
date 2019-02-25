---
title: 'CAF: Instruções de política de exemplo de Linha de base de identidade'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Instruções de política de exemplo de Linha de base de identidade
author: BrianBlanchard
ms.openlocfilehash: 5fad9265b9c048ee502c7e084ddd03faa0ad3e23
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900378"
---
# <a name="identity-baseline-sample-policy-statements"></a>Instruções de política de exemplo de Linha de base de identidade

As instruções individuais da política de nuvem são diretrizes para abordar os riscos específicos identificados durante o processo de avaliação de riscos. Essas instruções oferecem um resumo conciso dos riscos e planos com os quais lidar. Cada definição de instrução deve incluir essas informações:

- Risco técnico - Um resumo do risco que essa política abordará.
- Instrução da política - Uma explicação clara resumida dos requisitos da política.
- Opção de design - recomendações viáveis, especificações ou outras diretrizes que as equipes de TI e os desenvolvedores possam usar ao implementar a política.

As seguintes instruções de política de exemplo abordam um número de riscos comuns de negócios relacionados à identidade comum e são fornecidos como exemplos para fazer referência ao rascunhar instruções de política para atender a necessidades da sua organização. Esses exemplos não pretendem ser proscritivos e potencialmente há várias opções de políticas para lidar com qualquer risco específico. Trabalhe em sintonia com as equipes de TI para identificar as melhores soluções de política para o seu conjunto de riscos exclusivo.

## <a name="lack-of-access-controls"></a>Falta de controles de acesso

**Risco técnico**: Configurações de controle de acesso insuficiente ou ad hoc podem apresentar riscos de acesso não autorizado a recursos confidenciais ou críticos.

**Instrução da política**: Todos os ativos implantados na nuvem devem ser controlados usando identidades e funções aprovadas pelas políticas de governança atual.

**Opções possíveis de design**: [Acesso condicional ao Azure Active Directory](/azure/active-directory/conditional-access/overview) é o mecanismo de controle de acesso padrão no Azure.

## <a name="overprovisioned-access"></a>Acesso superprovisionados

**Risco técnico**: Usuários e grupos com controle sobre os recursos, além de suas áreas de responsabilidade podem resultar em modificações não autorizadas, levando a vulnerabilidades de segurança ou interrupções.

**Instrução da política**: As políticas a seguir serão implementadas:

- Um modelo de acesso de privilégio mínimo será aplicado a quaisquer recursos envolvidos nos aplicativos de missão crítica ou dados protegidos.
- Permissões elevadas devem ser uma exceção e qualquer exceção deve ser registrada com a equipe de governança de nuvem. As exceções serão auditadas regularmente.

**Opções possíveis de design**: Consulte as [práticas recomendadas de Gerenciamento de Identidade do Azure](/azure/security/azure-security-identity-management-best-practices) para implementar uma estratégia de controle de acesso baseado em função (RBAC) que restringe o acesso baseado na [necessidade de saber](https://wikipedia.org/wiki/Need_to_know) e [nos princípios mínimos de privilégio de segurança](https://wikipedia.org/wiki/Principle_of_least_privilege).

## <a name="lack-of-shared-management-accounts-between-on-premises-and-the-cloud"></a>Falta de contas de gerenciamento compartilhado entre local e nuvem

**Risco técnico**: A equipe administrativa ou de gerenciamento de TI com contas em seu Active Directory Domain Services local pode não ter acesso suficiente aos recursos da nuvem, pode não ser capaz de resolver problemas operacionais ou de segurança com eficiência.

**Instrução da política**: Todos os grupos que têm privilégios elevados na infraestrutura do Active Directory Domain Services local devem ser mapeados para uma função RBAC aprovada.

**Opções possíveis de design**: Implementar uma solução de identidade híbrida entre o Azure Active Directory baseado em nuvem e o seu Active Directory local, e adicionar os grupos locais necessários às funções RBAC necessárias para realizar seu trabalho.

## <a name="weak-authentication-mechanisms"></a>Mecanismos de autenticação fraca

**Risco técnico**: Sistemas de gerenciamento de identidade com métodos de autenticação de usuário insuficientemente seguro, como combinações de usuário/senha básica, podem levar a senhas comprometidas ou violadas, fornecendo um grande risco de acesso não autorizado para proteger os sistemas de nuvem.

**Instrução da política**: Todas as contas devem fazer logon em recursos protegidos usando um método de autenticação multifator (MFA).

**Opções possíveis de design**: Para o Azure Active Directory, implemente a [Autenticação Multifator do Azure](/azure/active-directory/authentication/concept-mfa-howitworks) como parte do processo de autorização de usuário.

## <a name="isolated-identity-providers"></a>Provedores de identidade isolados

**Risco técnico**: Provedores de identidade incompatíveis podem resultar na incapacidade de compartilhar recursos ou serviços com outros parceiros de negócios ou clientes.

**Instrução da política**: A implantação de qualquer aplicativo que exija a autenticação do cliente deve usar um provedor de identidade aprovado que seja compatível com o provedor de identidade principal para usuários internos.

**Opções possíveis de design**: Implemente a [Federação com o Azure Active Directory](/azure/active-directory/hybrid/whatis-fed) entre seus provedores de identidade internos e de cliente.

## <a name="identity-reviews"></a>Revisões de identidade

**Risco técnico**: Com o tempo, mudanças na empresa, adição de novas implantações de nuvem ou outras preocupações de segurança podem aumentar os riscos de acesso não autorizado a recursos seguros.

**Instrução da política**: Os processos de governança de nuvem devem incluir revisão trimestral com as equipes de gerenciamento de identidade para identificar atores mal-intencionados ou padrões de uso que devem ser impedidos pela configuração de ativo de nuvem.

**Opções possíveis de design**: Estabeleça uma reunião de revisão trimestral de segurança que inclui membros da equipe de governança e a equipe de TI responsáveis pelos serviços de identidade de gerenciamento. Examine os dados de segurança existentes e as métricas para estabelecer as lacunas na política de gerenciamento de identidade atual e ferramentas e atualiza a política para atenuar qualquer novo risco.

## <a name="next-steps"></a>Próximas etapas

Use os exemplos mencionados neste artigo como um ponto de partida para desenvolver políticas que abordam os riscos de negócios específicos que se alinham aos seus planos de adoção de nuvem.

Para começar a desenvolver suas próprias instruções de política personalizadas relacionadas à Linha de base de identidade, baixe o [Modelo de Linha de base de identidade](template.md).

Para acelerar a adoção desta disciplina, escolha o [Percurso de Governança Acionável](../journeys/overview.md) que mais se alinha ao seu ambiente. Em seguida, modifique o design para incorporar suas decisões específicas de política corporativa.

> [!div class="nextstepaction"]
> [Percursos de governança acionáveis](../journeys/overview.md)