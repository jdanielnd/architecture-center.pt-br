---
title: Testando aplicativos do Azure para resiliência e disponibilidade
description: Abordagens para testar a confiabilidade de aplicativos no Azure
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: cf33a859540810279b1cb85be5a6fb2ebe4500dc
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646478"
---
# <a name="testing-azure-applications-for-resiliency-and-availability"></a>Testando aplicativos do Azure para resiliência e disponibilidade

Para testar a resiliência, você deve verificar o desempenho da carga de trabalho de ponta a ponta sob condições de falha de intermitente.

Execute testes em produção usando tanto os dados de usuário sintéticos quanto reais. Teste e produção são raramente idênticos, portanto, é importante validar seu aplicativo em produção usando um [azul-verde](https://martinfowler.com/bliki/BlueGreenDeployment.html) ou [implantação canário](https://martinfowler.com/bliki/CanaryRelease.html). Dessa forma, você está testando o aplicativo em condições reais, portanto, você pode ter certeza de que ele funcionará conforme o esperado quando totalmente implantado.

Como parte do seu plano de teste, incluem:

- Pré-implantação de teste automatizado
- Teste de injeção de falha
- Teste de carga de pico
- Testes de recuperação de desastre
- Teste do serviço de terceiros

O teste é um processo iterativo. Teste o aplicativo, avalie o resultado, analise e resolva as falhas resultantes, e repita o processo.

## <a name="perform-fault-injection-testing"></a>Executar o teste de injeção de falha

Para teste de injeção de falha, verifique a resiliência do sistema durante falhas, disparando falhas reais ou simulando-as. Aqui estão algumas estratégias para induzir falhas:

- Desligue as instâncias de máquina virtual (VM).
- Falha de processos.
- expiração de certificados
- mudança de chaves de acesso
- desligamento do serviço DNS em controladores de domínio
- limitação de recursos do sistema disponíveis, como RAM ou número de threads
- desmontagem de discos
- reimplantação de uma VM

Seu plano de teste deve incorporar possíveis pontos de falha identificados durante a fase de design, além de cenários comuns de falha:

- Teste seu aplicativo em um ambiente mais próximo da produção possível.
- Falhas de teste na combinação.
- Meça o tempo de recuperação e certifique-se de que seus requisitos de negócios sejam atendidos.
- Verifique se que falhas não colocar em cascata e são tratadas de maneira isolada.

Para obter mais informações sobre cenários de falha, consulte [falha e recuperação de desastres para aplicativos do Azure](./disaster-recovery.md).

## <a name="test-under-peak-loads"></a>Testar sob cargas de pico

Teste de carga é crucial para identificar falhas que acontecem somente sob carga, como o banco de dados de back-end que está sendo sobrecarregado ou limitações de serviço. Teste de carga de pico e aumento previsto no pico de carga, usando dados de produção ou sintéticos, que é o mais próximo possível de dados produção. Sua meta é verificar como o aplicativo se comporta sob condições do mundo real.

## <a name="conduct-disaster-recovery-drills"></a>Realizar a recuperação de desastre

Começar com um plano de recuperação de desastres boa e testá-lo periodicamente para verificar se que ele funciona.

Máquinas virtuais do Azure, você pode usar [Azure Site Recovery](/azure/site-recovery/azure-to-azure-quickstart/) replicar e executar [recuperação de desastre](/azure/site-recovery/azure-to-azure-tutorial-dr-drill/) sem afetar a replicação em andamento ou aplicativos de produção.

### <a name="failover-and-failback-testing"></a>Teste de failover e failback

Testar o failover e failback para verificar que os serviços dependentes do seu aplicativo voltam ao funcionamento de forma sincronizada durante a recuperação de desastres. As alterações aos sistemas e operações podem afetar as funções de failover e failback, mas o impacto pode não ser detectado até que o sistema principal falhe ou fique sobrecarregado. Recursos de failover de teste *antes de* são necessárias para compensar um problema ao vivo. Também Certifique-se que os serviços dependentes executar failover e failback na ordem correta.

Se você estiver usando [Azure Site Recovery](/azure/site-recovery/) para replicar máquinas virtuais, execute desastres exercícios de recuperação periodicamente fazendo failovers de teste para validar sua estratégia de replicação. Um failover de teste não afeta a replicação contínua de VM ou seu ambiente de produção. Para obter mais informações, consulte [executar uma simulação de recuperação de desastres para o Azure](/azure/site-recovery/site-recovery-test-failover-to-azure).

### <a name="simulation-testing"></a>Os testes de simulação

Os testes de simulação envolvem a criação de situações de pequeno e reais. Simulações de demonstram a eficácia das soluções no plano de recuperação e realçam quaisquer problemas que não foram satisfeitos adequadamente.

Conforme você realiza testes de simulação, siga as práticas recomendadas:

- Conduza a simulações de forma que não afetem negócios reais, mas parece uma situação real.
- Certifique-se de que os cenários simulados são completamente controláveis. Se o plano de recuperação parecer estar falhando, você pode restaurar a situação voltar ao normal sem causar danos.
- Informe à gerência sobre quando e como os exercícios de simulação serão realizados. Seu plano deve detalhar o período de tempo e os recursos afetados durante a simulação.

## <a name="test-health-probes-for-load-balancers-and-traffic-managers"></a>Testar a investigações de integridade para balanceadores de carga e gerenciadores de tráfego

Configurar e testar a investigações de integridade para balanceadores de carga e gerenciadores de tráfego. Certifique-se de que seu ponto de extremidade de integridade verifica as partes críticas do sistema e responde apropriadamente.

- Para [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview/), a investigação de integridade determina se é necessário fazer failover para outra região. O ponto de extremidade de integridade deve verificar todas as dependências críticas que são implantadas na mesma região.
- Para [balanceador de carga do Azure](/azure/load-balancer/load-balancer-overview/), a investigação de integridade determina se é necessário remover uma VM da rotação. O ponto de extremidade de integridade deve relatar a integridade da VM. Não inclua outras camadas ou serviços externos. Caso contrário, uma falha que ocorra fora da VM fará com que o balanceador de carga remova a VM da rotação.

Para diretrizes sobre como implementar a integridade de monitoramento em seu aplicativo, confira [Padrão de monitoramento de ponto de extremidade de integridade](../patterns/health-endpoint-monitoring.md).

## <a name="test-monitoring-systems"></a>Testar sistemas de monitoramento

Inclua sistemas de monitoramento no plano de teste. Sistemas automatizados de failover e failback dependem do funcionamento correto da instrumentação e monitoramento. Painéis para visualizar os alertas de integridade e o operador do sistema também dependem de instrumentação e monitoramento precisos. Se esses elementos falharem, perderem informações essenciais ou relatarem dados imprecisos, um operador poderá deixar de perceber que o sistema não está íntegro ou que está falhando.

## <a name="include-third-party-services-in-test-scenarios"></a>Incluir serviços de terceiros em cenários de teste

Se seu aplicativo tiver dependências de serviços de terceiros, identifique onde e como esses serviços podem falhar e o efeito que essas falhas terá no seu aplicativo. Tenha em mente o contrato de nível de serviço (SLA) para o serviço de terceiros e o efeito que isso teria em seu plano de recuperação de desastres.

Um serviço de terceiros pode não fornecer recursos de monitoramento e diagnóstico, portanto, é importante para fazer logon sua invocações deles e correlacioná-los com a integridade do aplicativo e usando um identificador exclusivo de log de diagnóstico. Para obter mais informações sobre práticas comprovadas de monitoramento e diagnóstico, consulte [diretrizes de monitoramento e diagnóstico](../best-practices/monitoring.md).

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Implantar para resiliência e disponibilidade](./deploy.md)
