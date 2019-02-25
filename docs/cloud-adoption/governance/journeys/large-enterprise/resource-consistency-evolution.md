---
title: Empresas de grande porte – Evolução da consistência de recursos
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Empresas de grande porte – Evolução da consistência de recursos
author: BrianBlanchard
ms.openlocfilehash: 9fc6ca015310c02338463d3c8aa53ddfc74699df
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900268"
---
# <a name="large-enterprise-resource-consistency-evolution"></a>Empresas de grande porte: Evolução da consistência de recursos

Este artigo desenvolve a narrativa adicionando controles de consistência de recurso para a governança MVP para dar suporte a aplicativos de missão crítica.

## <a name="evolution-of-the-narrative"></a>Evolução da narrativa

As equipes de adoção de nuvem têm atendido todos os requisitos para mover os dados protegidos. Com esses aplicativos vêm compromissos de SLA para os negócios e a necessidade de suporte das operações de TI. Por trás da equipe de migração de dois datacenters, várias equipes de BI e de desenvolvimento múltiplo estão prontas para começar a iniciar novas soluções na produção. As operações de TI são novas para a ideia de operações na nuvem e precisa de uma maneira para integrar rapidamente os processos operacionais existentes.

### <a name="evolution-of-current-state"></a>Evolução do estado atual

- A TI está movendo ativamente as cargas de trabalho de produção com dados protegidos no Azure. Um número de cargas de trabalho de baixa prioridade está fornecendo o tráfego de produção. Pode ser cortado mais, assim que as operações de TI estiverem prontas para dar suporte às cargas de trabalho.
- As equipes de desenvolvimento de aplicativo estão prontas para o tráfego de produção.
- A equipe do BI está pronta para integrar as previsões e ideias sobre os sistemas que executam operações para as três unidades de negócios.

### <a name="evolution-of-the-future-state"></a>Evolução do estado futuro

- As operações de TI são novas para a ideia de operações na nuvem e precisa de uma maneira para integrar rapidamente os processos operacionais existentes.

As alterações ao estado atual e futuro expõem novos riscos que exigem novas instruções de política.

## <a name="evolution-of-tangible-risks"></a>Evolução de riscos tangíveis

**Interrupção dos negócios**: Há um risco inerente da nova plataforma causar interrupções para processos de negócios de missão crítica. A equipe de operações de TI e as equipes que executam em várias adoções de nuvem são relativamente inexperientes com operações na nuvem. Isso aumenta o risco de interrupção e deve ser reduzido e controlado.

Esse risco de negócios pode ser dividido em alguns riscos técnicos:

- Processos operacionais desalinhados podem levar a interrupções que não podem ser detectadas ou corrigidas rapidamente.
- Uma invasão externa ou ataques de negação de serviço podem causar uma interrupção dos negócios
- Ativos críticos não estão corretamente descobertos e, portanto, não operados corretamente.
- Ativos não descobertos ou rotulados incorretamente podem não ser suportados pelos processos de gerenciamento operacionais existentes.
- Configuração de ativos implantados pode não atender às expectativas de desempenho.
- Registro em log pode não ser corretamente registrado e centralizado para permitir a correção dos problemas de desempenho.
- As políticas de recuperação podem falhar ou levar mais tempo do que o esperado.
- Os processos de implantação podem resultar em falhas de segurança, o que pode levar a perdas de dados ou interrupções.
- A inconsistência na configuração ou patches ausentes podem resultar em falhas de segurança não intencionais, o que pode levar a vazamentos de dados ou interrupções.
- Configuração não pode impor os requisitos de SLAs ou requisitos de recuperação confirmados.
- Sistemas operacionais implantados ou aplicativos poderão não atender requisitos de proteção de aplicativo e de sistema operacional.
- Há um risco de inconsistência devido a várias equipes trabalharem na nuvem.

## <a name="evolution-of-the-policy-statements"></a>Evolução das instruções da política

As seguintes alterações à política ajudam a reduzir novos riscos e a orientar a implementação. A lista parece longa, mas a adoção dessas políticas pode ser mais fácil do que parece.

1. Todos os ativos implantados devem ser categorizados por nível de importância e classificação de dados. As classificações devem ser revisadas pela equipe de Governança de Nuvem e pelo proprietário do aplicativo antes da implantação na nuvem.
2. Sub-redes que contêm aplicativos de missão crítica devem ser protegidas por uma solução de firewall capaz de detecção de invasões e responder a ataques.
3. As ferramentas de governança devem realizar auditoria e impor requisitos de configuração de rede definidos pela equipe de Linha de Base de segurança.
4. Ferramentas de governança devem validar que todos os ativos relacionados a aplicativos de missão crítica ou dados protegidos são incluídos no monitoramento para otimização e esgotamento de recursos.
5. Ferramentas de governança devem validar que o nível apropriado de dados de log está sendo coletado para todos os aplicativos de missão crítica ou dados protegidos.
6. O processo de governança deve validar o backup, recuperação e o cumprimento de SLA são implementados corretamente para aplicativos de missão crítica e dados protegidos.
7. As ferramentas de governança devem limitar a implantação da VM somente a imagens aprovadas.
8. Ferramentas de governança devem impor que as atualizações automáticas estejam **impedidas** em todos os ativos implantados que dão suporte a aplicativos de missão crítica. As violações devem ser revisadas com as equipes de gerenciamento operacional e corrigidas de acordo com as políticas de operações. Os ativos que não são automaticamente atualizados devem ser incluídos em processos que pertencem às operações de TI.
9. Ferramentas de governança devem validar a marcação relacionadas à classificação de custo, nível de importância, SLA, aplicativos e dados. Todos os valores devem estar alinhados aos valores predefinidos gerenciados pela equipe de governança de nuvem.
10. Os processos de controle devem incluir auditorias no momento da implantação e em ciclos regulares para garantir a consistência em todos os ativos.
11. As tendências e explorações que podem afetar as implantações de nuvem devem ser revisadas regularmente pela equipe de segurança para que sejam fornecidas atualizações às ferramentas de Linha de base de segurança usadas na nuvem.
12. Antes do lançamento para a produção, todos os aplicativos de missão crítica e dados protegidos devem ser adicionados à solução de monitoramento operacional designada. Ativos que não podem ser descobertos pelas ferramentas de operações de TI escolhidas, não podem ser liberados para uso em produção. Quaisquer alterações necessárias para tornar os ativos detectáveis devem ser feitas aos processos de implantação relevantes para garantir que os ativos sejam detectáveis em implantações futuras.
13. Após a descoberta, o dimensionamento do ativo é ser validado por equipes de gerenciamento operacional para validar que o ativo atenda aos requisitos de desempenho.
14. As ferramentas de implantação devem ser aprovadas pela equipe de Governança de Nuvem para garantir uma governança contínua dos ativos implantados.
15. Os scripts de implantação devem ser mantidos em um repositório central acessível pela equipe de Governança de Nuvem para revisão e auditoria periódicas.
16. Processos de revisão de governança devem validar que os ativos implantados estão configurados corretamente alinhados aos requisitos de SLA e recuperação.

## <a name="evolution-of-the-best-practices"></a>Evolução das práticas recomendadas

Esta seção do artigo evoluirá o design MVP de governança para incluir novas políticas do Azure e uma implementação do Gerenciamento de Custos do Azure. Juntas, essas duas alterações de design atenderão às novas instruções da política corporativa.

Seguindo a experiência deste exemplo fictício, supõe-se que já aconteceu a evolução dos dados protegidos. Aproveitando essa prática recomendada, o seguinte adicionará requisitos operacionais de monitoramento, preparando uma assinatura para aplicativos de missão crítica.

**Assinatura de TI corporativa**: Adicione o seguinte para a assinatura de TI corporativa, que atua como um hub.

1. Como uma dependência externa, a equipe de operações de nuvem será necessária para definir as ferramentas de monitoramento operacional, ferramentas de recuperação de desastres/continuidade de negócios (BCDR) e a correção automatizada de ferramentas. O equipe de Governança de nuvem pode, em seguida, dar suporte aos processos de descoberta necessários.
    1. Nesse caso, a equipe de operações de nuvem escolheu o Azure Monitor como a principal ferramenta para monitoramento de aplicativos de missão crítica.
    2. A equipe também escolheu o Azure Site Recovery como as principais ferramentas de BCDR.
2. Implementação do Azure Site Recovery
    1. Definir e implantar o Azure Vault para backup e processos de recuperação
    2. Criar um modelo de Gerenciamento de Recursos do Azure para criação de um cofre em cada assinatura
3. Implementação do Azure Monitor
    1. Depois que uma assinatura de missão crítica for identificada, um espaço de trabalho do Log Analytics pode ser criado usando o PowerShell. Esse é um processo de pré-implantação.

**Assinatura de adoção de nuvem individual**: O exemplo a seguir garantirá que cada assinatura seja detectável pela solução de monitoramento e pronta para ser incluída nas práticas BCDR.

1. Azure Policy para nós de missão crítica
    1. Realize uma auditoria e imponha o uso das funções apenas.
    2. Realize uma auditoria e imponha o aplicativo de criptografia para todas as contas de armazenamento.
    3. Realize uma auditoria e imponha o uso de uma sub-rede de rede aprovada e rede virtual por adaptador de rede.
    4. Realize uma auditoria e imponha a limitação das tabelas de roteamento definidas pelo usuário.
    5. Realize uma auditoria e imponha a implantação dos agentes do Log Analytics para máquinas virtuais Windows ou Linux.
2. Blueprint do Azure
    1. Criar um blueprint nomeado `mission-critical-workloads-and-protected-data`. Este blueprint aplicará ativos, além do blueprint dos dados protegidos.
    2. Adicione as novas políticas do Azure para o blueprint.
    3. Aplique o blueprint para qualquer assinatura que é esperada para hospedar um aplicativo de missão crítica.

## <a name="conclusion"></a>Conclusão

Adicione esses processos e alterações ao MVP de governa ajuda a eliminar muitos dos riscos associados à governança de recursos. Juntos, aumentam a recuperação, dimensionamento e controles de monitoramento necessários para capacitar as operações de reconhecimento de nuvem.

## <a name="next-steps"></a>Próximas etapas

Como a adoção da nuvem continua a se desenvolver e a tornar-se mais útil comercialmente, a governança de riscos e de nuvem também precisam evoluir. Para a empresa fictícia nessa jornada, o próximo gatilho é quando a escala de implantação excede 1.000 ativos para a nuvem ou de gastos mensais excede 10 mil dólares por mês. Neste ponto, a equipe de governança de nuvem adiciona controles de Gerenciamento de Custos.

> [!div class="nextstepaction"]
> [Evolução do Gerenciamento de Custos](./cost-management-evolution.md)