---
title: 'CAF: Métricas de Linha de base de segurança, indicadores e tolerância a risco'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Métricas de Linha de base de segurança, indicadores e tolerância a risco
author: BrianBlanchard
ms.openlocfilehash: 30deafca59b2e09c78432ad3b59d328fb27a1e2c
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900325"
---
# <a name="security-baseline-metrics-indicators-and-risk-tolerance"></a>Métricas de Linha de base de segurança, indicadores e tolerância a risco

Este artigo tem como objetivo ajudar a quantificar a tolerância a riscos de negócios no que diz respeito à Linha de base de segurança. Definir métricas e indicadores ajuda a criar um caso de negócios para investir no amadurecimento da disciplina Linha de Base de Segurança.

## <a name="metrics"></a>Métricas

Linha de base de segurança em geral se concentra na identificação de possíveis vulnerabilidades em suas implantações de nuvem. Como parte da sua análise de risco, você desejará coletar dados relacionados ao seu ambiente de segurança para determinar o grau do risco enfrentado e quão importante é o investimento na governança da linha de base de segurança para as implantações de nuvem planejadas.

Cada organização tem requisitos e ambientes de segurança diferentes e diferentes possíveis fontes de dados de segurança. Veja a seguir exemplos de métricas úteis que devem ser coletadas para ajudar a avaliar a tolerância a riscos na disciplina Linha de base de segurança:

- **Classificação de dados**. Número de dados armazenados em nuvem e serviços que não são classificados de acordo com na privacidade, conformidade ou padrões de impacto de negócios da sua organização.
- **Número de armazenamentos de dados confidenciais** - número de pontos de extremidade de armazenamento ou bancos de dados que contêm dados confidenciais e devem ser protegidos.
- **Número de armazenamentos de dados não criptografados** - número de armazenamentos de dados confidenciais que não são criptografados.
- **Superfície de ataque**. Quantas fontes de dados totais, serviços e aplicativos serão hospedados na nuvem. Qual porcentagem dessas fontes de dados ser classificada como sensível? Qual é a porcentagem desses aplicativos e serviços serem de missão crítica?
- **Padrões Cobertos**. Número de padrões de segurança definidos pela equipe de segurança.
- **Recursos Abordados**. Ativos implantados que são cobertos pelos padrões de segurança.
- **Conformidade com Padrões Gerais**. Taxa de aderência à conformidade com os padrões de segurança.
- **Ataques por Gravidade**. Quantas tentativas coordenadas para interromper os serviços hospedados na nuvem, como por meio de ataques de negação de serviço distribuído (DDoS), faz a sua experiência de infraestrutura? Qual é o tamanho e a gravidade desses ataques?
- **Proteção contra malware**. Porcentagem de máquinas virtuais implantadas (VMs) que têm todos os itens necessário anti-malware, firewall ou outro software de segurança instalado.
- **Latência de patch**. Quanto tempo passou desde que as VMs tiveram patches do sistema operacional e software aplicados.
- **Recomendações de integridade de segurança**. Número de recomendações de software de segurança para resolver os padrões de integridade para recursos implantados, organizados por gravidade.

## <a name="risk-tolerance-indicators"></a>Indicadores de tolerância de risco

Plataformas de nuvem fornecem um conjunto de recursos de linha de base que permitem que as equipes de implantação pequenas configurem as definições de segurança básicas sem planejamento adicional extensivo. Como resultado, Desenvolvimento/Teste pequeno ou primeiras cargas de trabalho experimentais que não incluem dados confidenciais representam nível de risco relativamente baixo e provavelmente não precisará de muito em termos de política formal de Linha de base de segurança. No entanto, assim que os dados importantes ou a funcionalidade de missão crítica for movida para a nuvem, o risco de segurança aumenta, enquanto a tolerância a esses riscos diminui rapidamente. Conforme mais dos seus dados e a funcionalidade são implantados na nuvem, será maior a probabilidade de você precisar de um investimento maior na disciplina de Linha de base de segurança.

Nos estágios iniciais de adoção da nuvem, trabalhe com sua equipe e segurança de TI e stakeholders de negócios para identificar os [riscos de negócios](business-risks.md) relacionados à identidade e, em seguida, determine uma linha de base aceitável para a tolerância do risco de segurança. Esta seção do CAF fornece exemplos. No entanto, os riscos e linhas de base detalhados da empresa ou de suas implantações podem ser diferentes.

Assim que você tiver uma linha de base, estabeleça parâmetros de comparação mínimos que representem um aumento inaceitável em seus riscos identificados. Esses parâmetros de comparação atuam como gatilhos quando você precisar tomar medidas para atenuar esses riscos. Veja a seguir alguns exemplos de como métricas relacionadas à segurança, como as discutidas anteriormente, podem justificar o aumento de investimento na disciplina Linha de Base de Segurança.

- **Gatilho de cargas de trabalho de missão crítica**. Uma empresa implantando cargas de trabalho de missão crítica para a nuvem deve investir na disciplina de Linha de base de segurança para impedir possíveis interrupções de serviço ou a exposição de dados confidenciais.
- **Gatilho de dados protegidos**. Uma empresa que hospeda os dados na nuvem que podem ser classificados como confidenciais, privados ou caso contrário, sujeito a preocupações com regulamentação. Precisam de uma disciplina de linha de base de segurança para garantir que esses dados não estejam sujeitos a perda, exposição ou roubo.
- **Gatilho de ataques externos**. Uma empresa que passa por sérios ataques contra sua rede de infraestrutura de rede X vez por mês pode beneficiar-se da disciplina de Linha de base de segurança.  
- **Gatilho de conformidade de padrões**. Uma empresa com mais de X% dos recursos fora de conformidade de padrões de segurança deve investir na disciplina de Linha de base de segurança para garantir que os padrões sejam aplicados de forma consistente em sua infraestrutura de TI.
- **Gatilho de tamanho do espaço de nuvem**. Uma empresa de hospedagem mais de X número de aplicativos, serviços ou fontes de dados. Implantações de nuvem grande podem aproveitar o investimento na disciplina de linha de base de segurança para garantir que a superfície de ataque geral seja devidamente protegida contra acesso não autorizado ou outras ameaças externas.
- **Gatilho de conformidade do software de segurança**. Uma empresa onde menos que X% de máquinas virtuais implantadas têm todos os softwares de segurança necessários instalados. Uma disciplina de Linha de base de segurança pode ser usada para garantir que o software seja instalado consistentemente em todos os softwares.
- **Gatilho de aplicação de patch**. Uma empresa em que máquinas virtuais implantadas ou serviços onde patches do sistema operacional ou software não foram aplicados nos últimos X número de dias. A disciplina de Linha de base de segurança pode ser usada para garantir a aplicação de patch ser mantida atualizada dentro de uma agenda necessária.
- **Segurança com foco**. Algumas empresas terão fortes requisitos de segurança e confidencialidade de dados, mesmo para teste e cargas de trabalho experimentais. Essas empresas precisarão investir na disciplina de linha de base de segurança antes de começam as implantações.

As métricas e os gatilhos exatos usados para medir a tolerância de risco e o nível de investimento na disciplina Linha de Base de Segurança serão específicos da organização. No entanto, os exemplos já apresentados devem servir como uma base útil para discussão dentro da equipe de governança de nuvem.  

## <a name="next-steps"></a>Próximas etapas

Usar o [modelo de Gerenciamento de Nuvem](./template.md), de métricas do documento e de indicadores de tolerância que se alinham ao atual plano de adoção da nuvem.

Usar riscos e tolerância como base e estabelecer um processo para administrar e comunicar a adesão com a política de Linha de base de segurança.

> [!div class="nextstepaction"]
> [Estabelecer processos de conformidade de política](compliance-processes.md)
