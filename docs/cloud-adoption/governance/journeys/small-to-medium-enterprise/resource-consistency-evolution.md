---
title: 'CAF: Empresas de pequeno a médio porte – Evolução da consistência de recursos '
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Explicação de empresas de pequeno a médio porte – Evolução da consistência de recursos
author: BrianBlanchard
ms.openlocfilehash: 402bb3ff4182123cdc8825522f965f7cf8637980
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900292"
---
# <a name="small-to-medium-enterprise-resource-consistency-evolution"></a>Empresas de pequeno a médio porte: Evolução da consistência de recursos

Este artigo desenvolve a narrativa adicionando controles de Consistência de Recursos para dar suporte a aplicativos de missão crítica.

## <a name="evolution-of-the-narrative"></a>Evolução da narrativa

Novas experiências do cliente, novas ferramentas de previsão e infraestrutura migrada continuam a evoluir. A empresa está pronta para começar a usar esses ativos em uma capacidade de produção.

### <a name="evolution-of-the-current-state"></a>Evolução do estado atual

Na fase anterior desse narrativa, as equipes de BI e o desenvolvimento de aplicativos estavam quase prontos para integrar clientes e dados financeiros em cargas de trabalho de produção. A equipe de TI estava no processo de desativação do datacenter de recuperação de desastre.

Desde então, ocorreram algumas mudanças que afetarão a governança:

- A IT desativou 100% do datacenter de recuperação de desastres, à frente do cronograma. No processo, um número de ativos no datacenter de produção foi identificado como candidatos à migração de nuvem.
- As equipes de desenvolvimento de aplicativo estão agora prontas para o tráfego de produção.
- A equipe de BI está pronta para alimentar previsões e informações de volta para os sistemas operacionais no data center de produção.

### <a name="evolution-of-the-future-state"></a>Evolução do estado futuro

Antes de usar as implantações do Azure em processos de negócios de produção, as operações de nuvem devem amadurecer. Em conjunto, uma evolução de governança adicional é necessária para garantir que ativos possam ser operados corretamente.

As alterações ao estado atual e futuro expõem novos riscos que exigem novas instruções de política.

## <a name="evolution-of-tangible-risks"></a>Evolução de riscos tangíveis

**Interrupção dos negócios**: Há um risco inerente da nova plataforma causar interrupções para processos de negócios de missão crítica. A equipe de operações de TI e as equipes que executam em várias adoções de nuvem são relativamente inexperientes com operações na nuvem. Isso aumenta o risco de interrupção e deve ser reduzido e controlado.

Esse risco de negócios pode ser dividido em alguns riscos técnicos:

- Uma invasão externa ou ataques de negação de serviço podem causar uma interrupção dos negócios
- Ativos críticos não são corretamente descobertos e, portanto, não operados corretamente
- Ativos não descobertos ou rotulados incorretamente podem não ser suportados pelos processos de gerenciamento operacionais existentes.
- A configuração de ativos implantados pode não atender às expectativas de desempenho
- Registro em log pode não ser corretamente registrado e centralizado para permitir a correção dos problemas de desempenho.
- As políticas de recuperação podem falhar ou levar mais tempo do que o esperado.
- Os processos de implantação podem resultar em falhas de segurança, o que pode levar a perdas de dados ou interrupções.
- A inconsistência na configuração ou patches ausentes podem resultar em falhas de segurança não intencionais, o que pode levar a vazamentos de dados ou interrupções.
- Configuração não pode impor os requisitos de SLAs ou requisitos de recuperação confirmados.
- Sistemas operacionais implantados ou aplicativos poderão falhar em atender aos requisitos de proteção
- Com muitas equipes que trabalham na nuvem, há um risco de inconsistência.

## <a name="evolution-of-the-policy-statements"></a>Evolução das instruções da política

As seguintes alterações à política ajudam a reduzir novos riscos e a orientar a implementação. A lista parece longa, mas adotar essas políticas talvez seja mais fácil do que parece.

1. Todos os ativos implantados devem ser categorizados por nível de importância e classificação de dados. As classificações devem ser revisadas pela equipe de Governança de Nuvem e pelo proprietário do aplicativo antes da implantação na nuvem
2. Sub-redes que contêm aplicativos de missão crítica devem ser protegidas por uma solução de firewall capaz de detecção de invasões e responder a ataques.
3. As ferramentas de governança devem realizar a auditoria e impor requisitos de configuração de rede definidos pela equipe de gerenciamento de segurança
4. Ferramentas de governança devem validar que todos os ativos relacionados a aplicativos de missão crítica ou dados protegidos são incluídos no monitoramento para otimização e esgotamento de recursos.
5. Ferramentas de governança devem validar que o nível apropriado de dados de log está sendo coletado para todos os aplicativos de missão crítica ou dados protegidos.
6. O processo de governança deve validar o backup, recuperação e o cumprimento de SLA são implementados corretamente para aplicativos de missão crítica e dados protegidos.
7. As ferramentas de governança devem limitar as implantações da VM somente a imagens aprovadas.
8. Ferramentas de governança devem impor que as atualizações automáticas estejam impedidas em todos os ativos implantados que dão suporte a aplicativos de missão crítica. As violações devem ser revisadas com as equipes de gerenciamento operacional e corrigidas de acordo com as políticas de operações. Os ativos que não são automaticamente atualizados devem ser incluídos em processos que pertencem às operações de TI.
9. Ferramentas de governança devem validar a marcação relacionadas à classificação de custo, nível de importância, SLA, aplicativos e dados. Todos os valores devem estar alinhados aos valores predefinidos gerenciados pela equipe de governança.
10. Os processos de controle devem incluir auditorias no momento da implantação e em ciclos regulares para garantir a consistência em todos os ativos.
11. As tendências e explorações que podem afetar as implantações de nuvem devem ser revisadas regularmente pela equipe de segurança para que sejam fornecidas atualizações às ferramentas de gerenciamento de segurança usadas na nuvem.
12. Antes do lançamento para a produção, todos os aplicativos de missão crítica e dados protegidos devem ser adicionados à solução de monitoramento operacional designada. Ativos que não podem ser descobertos pelas ferramentas de operações de TI escolhidas, não podem ser liberados para uso em produção. Quaisquer alterações necessárias para tornar os ativos detectáveis devem ser feitas aos processos de implantação relevantes para garantir que os ativos sejam detectáveis em implantações futuras.
13. Após a descoberta, as equipes de gerenciamento operacional irão dimensionar os ativos para garantir que os ativos atendam aos requisitos de desempenho
14. As ferramentas de implantação devem ser aprovadas pela equipe de Governança de Nuvem para garantir uma governança contínua dos ativos implantados.
15. Os scripts de implantação devem ser mantidos em um repositório central acessível pela equipe de Governança de Nuvem para revisão e auditoria periódicas.
16. Processos de revisão de governança devem validar que os ativos implantados estão configurados corretamente alinhados aos requisitos de SLA e recuperação.

## <a name="evolution-of-the-best-practices"></a>Evolução das práticas recomendadas

Esta seção do artigo evoluirá o design MVP de governança para incluir novas políticas do Azure e uma implementação do Gerenciamento de Custos do Azure. Juntas, essas duas alterações de design atenderão às novas instruções da política corporativa.

1. A equipe de operações de nuvem definirá ferramentas de monitoramento operacional e automatizado de ferramentas de correção. A equipe de Governança de Nuvem ajudará no suporte a esses processos de descoberta. Nesse caso, a equipe de operações de nuvem escolheu o Azure Monitor como a principal ferramenta para monitoramento de aplicativos de missão crítica.
2. Crie um repositório no Azure DevOps para armazenar e controlar a versão de todos os modelos do Gerenciador de Recursos relevantes e configurações com script.
3. Implementação do Azure Vault:
    1. Definir e implantar o Azure Vault para backup e processos de recuperação.
    2. Criar um modelo do Gerenciador de Recursos para criação de um cofre em cada assinatura.
4. Atualizar o Azure Policy para todas as assinaturas:
    1. Realize auditoria e imponha o nível de importância e classificação de dados em todas as assinaturas para identificar todas as assinaturas com ativos de missão crítica.
    2. Realize uma auditoria e imponha o uso de imagens aprovadas apenas.
5. Implementação do Azure Monitor:
    1. Depois que uma assinatura de missão crítica for identificada, crie um espaço de trabalho no Azure Monitor usando o PowerShell. Esse é um processo de pré-implantação.
    2. Durante o teste de implantação, a equipe de operações na nuvem implanta a descoberta de agentes e os testes necessários.
6. Atualize o Azure Policy para todas as assinaturas que contêm aplicativos de missão crítica.
    1. Realize uma auditoria e imponha a aplicação de um NSG para todas as NICs e sub-redes. Rede e de Segurança de TI definirão o NSG.
    2. Realize uma auditoria e imponha o uso de sub-redes e VNets para cada interface de rede.
    3. Realize uma auditoria e imponha a limitação das tabelas de roteamento definidas pelo usuário.
    4. Audite e imponha a implantação de agentes do Azure Monitor para todas as máquinas virtuais.
    5. Audite e imponha que o Azure Vault existe na assinatura.
7. Configuração do firewall:
    1. Identifique uma configuração do Firewall do Azure que atenda aos requisitos de segurança. Como alternativa, identifique um dispositivo de terceiros que seja compatível com o Azure.
    2. Crie um modelo do Resource Manager para implantar o firewall com as configurações necessárias.
8. Blueprint do Azure:
    1. Crie um novo blueprint do Azure chamado `protected-data`.
    2. Adicione o firewall e os modelos Azure Vault ao blueprint.
    3. Adicione as novas políticas para assinaturas de dados protegidos.
    4. Publique blueprint em qualquer grupo de gerenciamento que se destina a hospedar aplicativos de missão crítica.
    5. Aplique o novo blueprint a cada assinatura afetada, bem como os blueprints existentes.

## <a name="conclusion"></a>Conclusão

Esses processos e alterações adicionais à governança MVP ajudam a eliminar muitos dos riscos associados à governança de recursos. Juntos, aumentam a recuperação, dimensionamento e controles de monitoramento que capacitam as operações de recuperação de nuvem.

## <a name="next-steps"></a>Próximas etapas

Como a adoção da nuvem continua a se desenvolver e a tornar-se mais útil comercialmente, a governança de riscos e de nuvem também precisam evoluir. Para a empresa fictícia nessa jornada, o próximo gatilho é quando a escala de implantação excede 100 ativos para a nuvem ou de gastos mensais excede 1 mil dólares por mês. Neste ponto, a equipe de governança de nuvem adiciona controles de Gerenciamento de Custos.

> [!div class="nextstepaction"]
> [Evolução do gerenciamento de custos](./cost-management-evolution.md)