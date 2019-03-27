---
title: 'CAF: Métricas, indicadores e tolerância a riscos da Consistência de Recursos'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Métricas, indicadores e tolerância a riscos da Consistência de Recursos
author: BrianBlanchard
ms.openlocfilehash: 3a2561a6d1d81a6395bb256e921a7a26898b45a0
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241987"
---
# <a name="resource-consistency-metrics-indicators-and-risk-tolerance"></a>Métricas, indicadores e tolerância a riscos da Consistência de Recursos

Este artigo tem como objetivo ajudar a quantificar a tolerância a riscos de negócios no que diz respeito à Consistência de Recursos. Definir métricas e indicadores ajuda a criar um caso de negócios para investir no amadurecimento da disciplina Consistência de Recursos.

## <a name="metrics"></a>Métricas

A disciplina de Consistência de Recursos se concentra em lidar com os riscos relacionados ao gerenciamento operacional de suas implantações de nuvem. Como parte da análise de risco, você irá querer reunir os dados relacionados às suas operações de TI para determinar quanto risco você enfrenta e como é importante o investimento na governança de Consistência de Recursos em suas implantações de nuvem planejadas.

Cada organização tem cenários operacionais diferentes, mas os itens a seguir representam exemplos úteis das métricas que você deve obter ao avaliar a tolerância de risco na disciplina de Consistência de Recursos:

- **Ativos de nuvem**. Número total de recursos de implantação de nuvem.
- **Recursos não marcados**. Número de recursos sem marcas de contabilização, impacto comercial ou organizacionais.
- **Ativos subutilizados**. Número de recursos em que recursos de memória, CPU ou rede são subutilizados de forma consistente.
- **Esgotamento de recursos**. Número de recursos em que a memória, a CPU ou os recursos de rede estão esgotados por carga.
- **Idade do recurso**. Tempo desde que o recurso foi implantado ou modificado.
- **Disponibilidade do serviço**. Porcentagem de tempo de atividade real hospedado na nuvem cargas de trabalho comparada ao tempo de atividade esperado.
- **VMs em condição crítica**. Número de máquinas virtuais onde um ou mais problemas críticos são detectados que precisam ser abordados para restaurar a funcionalidade normal.
- **Alertas por Gravidade**. Número total de alertas em um ativo de implantado, divididos por gravidade.
- **Links de sub-rede não íntegros**. Número de recursos com problemas de conectividade de rede.
- **Pontos de extremidade de serviço não íntegro**. Número de problemas com pontos de extremidade de rede externa.
- **Incidentes de integridade do serviço do Provedor de nuvem**. Número de interrupções ou incidentes de desempenho causados pelo provedor de nuvem.
- **Backup de integridade**. Número de backups que estão sendo sincronizados ativamente.
- **Integridade da recuperação**. Número de operações de recuperação executada com êxito.

## <a name="risk-tolerance-indicators"></a>Indicadores de tolerância de risco

Plataformas de nuvem que oferecem um conjunto de recursos que permite que a implantação de equipes gerencie com eficácia pequenas implantações sem planejamento adicional amplo ou processos. Como resultado, Desenvolvimento/Teste pequeno ou primeiras cargas de trabalho experimentais que incluem uma quantidade relativamente pequena de ativos baseados em nuvem, representam baixo nível de risco e provavelmente não serão necessário muito em relação a uma política de Consistência de Recursos formal.

No entanto, à medida que o tamanho do seu acervo de nuvem aumenta a complexidade de gerenciar seus ativos, torna-se significativamente mais difícil. Com mais ativos na nuvem, a capacidade de identificar a propriedade dos recursos e recursos de controle úteis se torna essencial para minimizar os riscos. Conforme mais cargas de trabalho de missão crítica são implantadas para a nuvem, o tempo de atividade do serviço se torna mais crítico e a tolerância para custo potencial de interrupção diminui rapidamente.

Nos estágios iniciais de adoção da nuvem, trabalhe com a equipe de operações de TI e stakeholders de negócios para identificar os [riscos de negócios](business-risks.md) relacionados a consistência de recursos, em seguida, determine uma linha de base aceitável para tolerância a riscos. Esta seção do CAF fornece exemplos. No entanto, os riscos e linhas de base detalhados da empresa ou de suas implantações podem ser diferentes.

Assim que você tiver uma linha de base, estabeleça parâmetros de comparação mínimos que representem um aumento inaceitável em seus riscos identificados. Esses parâmetros de comparação atuam como gatilhos quando você precisar tomar medidas para atenuar esses riscos. Veja a seguir alguns exemplos de como métricas operacionais, como as discutidas anteriormente, podem justificar o aumento de investimento na disciplina de Consistência de Recursos.

- **Marcação e gatilho de nomenclatura**. Uma empresa com mais X recursos faltando precisava de informações de marcação ou não obedecer os padrões de nomenclatura deve considerar investir na disciplina de Consistência de Recursos para ajudar a refinar esses padrões e garantir a aplicação consistente deles aos ativos implantados na nuvem.
- **Gatilho de recursos sobreprovisionados**. Se uma empresa tiver mais de X% de ativos usando regularmente muitas pequenas quantidades de seus recursos de memória, CPU ou rede disponíveis, o investimento na disciplina de Consistência de Recursos é sugerido para ajudar a otimizar os uso dos recursos para esses itens.
- **Gatilho de recursos não provisionados**. Se uma empresa tiver mais de X% de ativos usando exaustivamente a maioria de seus recursos de memória, CPU ou rede disponíveis, o investimento na disciplina de Consistência de Recursos é sugerido para ajudar a garantir que esses ativos tenham os recursos necessários para evitar as interrupções do serviço.
- **Gatilho de idade de recurso**. Uma empresa com mais de X recursos que não foram atualizados nos últimos X meses, poderia se beneficiar do investimento na disciplina de Consistência de Recursos planejada para garantir que os recursos ativos estejam corrigidos e íntegros, enquanto tenta novamente ativos obsoletos ou não de outra forma não utilizados.  
- **Gatilho de disponibilidade do serviço**. Uma empresa que tenha passado X % de tempo de atividade para serviços de missão crítica, deve investir na disciplina de consistência de recursos para melhorar a confiabilidade do serviço.
- **Gatilho de integridade da VM**. Uma empresa que tem mais de X % das VMs enfrentando um problema de integridade crítico, deve investir na disciplina de Consistência de Recursos para identificar problemas e melhorar a estabilidade da VM.
- **Gatilho de integridade de rede**. Uma empresa que tem mais de X % de pontos de extremidade com problemas de conectividade ou sub-redes da rede deve investir na disciplina de consistência de recursos para identificar e resolver problemas de rede.
- **Gatilho de cobertura de backup**. Uma empresa com X % de ativos de missão crítica sem backups atualizados no local pode se beneficiar de um maior investimento na disciplina de consistência de recurso para garantir uma estratégia de backup consistente.
- **Gatilho de integridade do backup**. Uma empresa com mais de X % falha das operações de restauração deve investir na disciplina de Consistência de Recursos para identificar problemas com o backup e garantir que os recursos importantes sejam protegidos.

As métricas e os gatilhos exatos usados para medir a tolerância de risco e o nível de investimento na disciplina Consistência de Recursos serão específicos da organização. No entanto, os exemplos já apresentados devem servir como uma base útil para discussão dentro da equipe de governança de nuvem.  

## <a name="next-steps"></a>Próximas etapas

Usar o [modelo de Gerenciamento de Nuvem](./template.md), de métricas do documento e de indicadores de tolerância que se alinham ao atual plano de adoção da nuvem.

Compilar riscos e tolerância como base e estabelecer um processo para administrar e comunicar a adesão à política de Consistência de Recursos.

> [!div class="nextstepaction"]
> [Estabelecer processos de conformidade de política](compliance-processes.md)
