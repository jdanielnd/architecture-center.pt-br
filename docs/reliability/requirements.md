---
title: Definir requisitos para aplicativos resilientes do Azure
description: Visão geral de reunir os requisitos de disponibilidade e resiliência para aplicativos do Azure
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: 2af2ce4200c285abfda1395862d21efc3c17e577
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646477"
---
# <a name="developing-requirements-for-resilient-azure-applications"></a>Desenvolvendo requisitos para aplicativos resilientes do Azure

Compilando *resiliência* (recuperar de falhas) e *disponibilidade* (em execução em um estado íntegro, sem tempo de inatividade significativo) em seus aplicativos começa com a coleta de requisitos. Por exemplo, quanto tempo de inatividade é aceitável? Quanto tempo de inatividade potencial custa seus negócios? Quais são os requisitos de disponibilidade do seu cliente? Quanto investir em tornar seu aplicativo altamente disponível? O que é o risco e o custo?

## <a name="identify-distinct-workloads"></a>Identificar as cargas de trabalho distintas

Normalmente, as soluções de nuvem consistem em um aplicativo de vários *cargas de trabalho*. Uma carga de trabalho é um recurso distinto ou uma tarefa que é logicamente separada de outras tarefas em termos de requisitos de armazenamento de dados e lógica de negócios. Por exemplo, um aplicativo de comércio eletrônico pode ter as seguintes cargas de trabalho:

- Procura e pesquisa em um catálogo de produtos.
- Criação e controle de pedidos.
- Exibição de recomendações.

Cada carga de trabalho tem diferentes requisitos de disponibilidade, escalabilidade, consistência de dados e recuperação de desastres. Tome suas decisões de negócios, equilibrando o custo versus risco para cada carga de trabalho.

Também decompor as cargas de trabalho por objetivo de nível de serviço. Se um serviço é composto de cargas de trabalho críticas e menos críticos, gerenciá-los de maneira diferente e especificar o número de instâncias necessárias para atender aos seus requisitos de disponibilidade e recursos do serviço.

## <a name="plan-for-usage-patterns"></a>Plano para padrões de uso

Identificar diferenças em requisitos durante períodos críticos e não-críticas. Há certos períodos críticos quando o sistema tem que estar disponível? Por exemplo, um aplicativo de arquivamento de imposto não pode falhar durante um prazo de entrega e um serviço de streaming de vídeo não deve ficar durante um evento ao vivo. Nessas situações, avalie o custo contra o risco.

- Para garantir o tempo de atividade e atender aos contratos de nível de serviço (SLAs) em períodos críticos, planejar a redundância entre várias regiões no caso de uma falha, mesmo que custa muito mais.
- Por outro lado, durante os períodos não críticos, execute o aplicativo em uma única região para minimizar os custos.
- Em alguns casos, você pode reduzir as despesas adicionais usando técnicas sem servidor modernas com cobrança com base no consumo.

## <a name="establish-availability-and-recovery-metrics"></a>Estabelecer métricas de disponibilidade e recuperação

Crie os números de linha de base para dois conjuntos de métricas como parte do processo de requisitos. O primeiro conjunto de ajuda a determinar onde adicionar redundância para serviços de nuvem e os SLAs para fornecer aos clientes. O segundo conjunto ajuda que você a planejar a recuperação de desastre.

### <a name="availability-metrics"></a>Métricas de disponibilidade

Use essas medidas para planejar a redundância e determinar os SLAs de cliente.

- **Tempo médio de recuperação (MTTR)** é o tempo médio que leva para restaurar um componente após uma falha.
- **Tempo médio entre falhas (MTBF)** é como long um componente razoável pode esperar ao último entre interrupções.

### <a name="recovery-metrics"></a>Métricas de recuperação

Derivar esses valores por realizar uma avaliação de risco e certifique-se de que entender o custo e o risco de tempo de inatividade e perda de dados. Esses são requisitos não-funcionais de um sistema e devem ser determinados pelos requisitos de negócios.

- **O objetivo de tempo de recuperação (RTO)** é o tempo máximo aceitável que um aplicativo não está disponível após um incidente.
- **O objetivo de ponto de recuperação (RPO)** é a duração máxima de perda de dados é aceitável durante um desastre.

Se o valor MTTR *qualquer* componente crítico em uma configuração altamente disponível excede o RTO do sistema, uma falha no sistema pode causar uma interrupção do negócio inaceitável. Ou seja, você não pode restaurar o sistema dentro do RTO definido.

## <a name="determine-workload-availability-targets"></a>Determinar as metas de disponibilidade da carga de trabalho

Defina seus próprios SLAs de destino para cada carga de trabalho em sua solução para que você possa determinar se a arquitetura atende aos requisitos de negócios.

### <a name="consider-cost-and-complexity"></a>Considere o custo e complexidade

Tudo sendo igual, maior disponibilidade é melhor. Mas como você se esforça para obter mais noves, aumentam o custo e complexidade. Um tempo de atividade de 99,99% se traduz em cerca de cinco minutos de inatividade total por mês. Vale a pena a complexidade adicional e o custo para alcançar cinco noves? A resposta depende dos requisitos de negócios.

Aqui estão algumas outras considerações ao definir um SLA:

- Para obter quatro noves (99,99%), você não pode contar intervenção manual para se recuperar de falhas. O aplicativo deve ser de autodiagnóstico e autorrecuperação.
- Além dos quatro noves, é muito difícil para detectar problemas rápido o suficiente para atender ao SLA.
- Pense no intervalo de tempo ao qual a medição do seu SLA está relacionada. Quanto menor for o intervalo, menor será a tolerância. Não faz sentido definir seu SLA em termos de tempo de atividade por hora ou diariamente.
- Considere as medidas de MTBF e MTTR. Quanto maior for seu SLA, menos frequentemente o serviço pode ficar inativo e quanto mais rápido que o serviço deve recuperar.
- Obtenha o contrato dos seus clientes para os destinos de disponibilidade de cada parte do seu aplicativo e documente-lo. Caso contrário, seu design não pode atender às expectativas dos clientes.

### <a name="identify-dependencies"></a>Identificar dependências

Execute os exercícios de mapeamento de dependência para identificar dependências internas e externas. Exemplos incluem dependências em relação à segurança ou identidade, como o Active Directory ou serviços de terceiros, como um provedor de pagamento ou o serviço de mensagens de email.

Preste atenção especial para dependências externas que podem ser um ponto único de falha ou causar gargalos. Se uma carga de trabalho requer tempo de atividade de 99,99%, mas depende de um serviço com um SLA de 99,9%, esse serviço não pode ser um ponto único de falha no sistema. Uma solução é ter um caminho de fallback em caso de falha de serviço. Como alternativa, tome outras medidas para recuperar de uma falha nesse serviço.

A tabela a seguir mostra o tempo de inatividade cumulativo em potencial para vários níveis de SLA.

| **SLA** | **Tempo de inatividade por semana** | **Tempo de inatividade por mês** | **Tempo de inatividade por ano** |
|---------|-----------------------|------------------------|-----------------------|
| 99%     | 1,68 hora            | 7,2 horas              | 3,65 dias             |
| 99,9%   | 10,1 minutos          | 43,2 minutos           | 8,76 horas            |
| 99,95%  | 5 minutos             | 21,6 minutos           | 4,38 horas            |
| 99,99%  | 1,01 minutos          | 4,32 minutos           | 52,56 minutos         |
| 99,999% | 6 segundos             | 25,9 segundos           | 5,26 minutos          |

## <a name="understand-service-level-agreements"></a>Entender os contratos de nível de serviço

No Azure, o [contrato de nível de serviço](https://azure.microsoft.com/en-us/support/legal/sla/) descreve os compromissos da Microsoft de atividade e conectividade. Se o SLA para um serviço específico for de 99,9%, você deve esperar que o serviço esteja disponível 99,9% do tempo. Serviços diferentes têm diferentes SLAs.

O SLA do Azure também inclui provisões para se obter um crédito de serviço se o SLA não for atendido, juntamente com definições específicas dos *disponibilidade* para cada serviço. Esse aspecto do SLA atua como uma política de imposição.

### <a name="composite-slas"></a>SLAs compostos

*SLAs compostos* envolver vários serviços que dão suporte a um aplicativo, cada um com diferentes níveis de disponibilidade. Por exemplo, considere um aplicativo de web do serviço de aplicativo que grava no banco de dados SQL. No momento da redação deste artigo, esses serviços do Azure têm os seguintes SLAs:

- Aplicativos web do serviço de aplicativo = 99,95%
- Banco de dados SQL = 99,99%

Qual é o tempo de inatividade máximo esperado para este aplicativo? Se o serviço falhar, o aplicativo inteiro falhará. A probabilidade de cada serviço com falha é independente, portanto, o SLA composto para este aplicativo é de 99,95% × 99,99% = % 99,94%. Que é menor do que os SLAs individuais, que não é surpreendente porque um aplicativo que depende de vários serviços possui mais pontos potenciais de falha.

Você pode melhorar o SLA composto Criando caminhos de fallback independentes. Por exemplo, se o banco de dados SQL não estiver disponível, coloque as transações em uma fila para serem processadas posteriormente.

![SLA composto](_images/composite-sla.png)

Com esse design, o aplicativo ainda estará disponível mesmo se não puder se conectar ao banco de dados. No entanto, ele falhará se o banco de dados e a fila falharem ao mesmo tempo. A porcentagem esperada de tempo de uma falha simultânea é 0,0001 × 0,001, portanto, o SLA composto para este caminho combinado é:

- Banco de dados *ou* fila = 1,0 − (0,0001 × 0,001) = 99,99999%

O SLA composto total é:

- Aplicativo Web *e* (banco de dados *ou* fila) = 99,95% × 99,99999% = \~99,95%

Há vantagens e desvantagens nessa abordagem. A lógica do aplicativo é mais complexa, você está pagando para a fila e você precisa considerar os problemas de consistência de dados.

### <a name="slas-for-multiregion-deployments"></a>SLAs para implantações multiregion

*Os SLAs para implantações multiregion* envolvem uma técnica de alta disponibilidade para implantar o aplicativo em mais de uma região e use o Gerenciador de tráfego do Azure para fazer failover, se o aplicativo falhar em uma região.

O SLA composto para uma implantação multiregion é calculado da seguinte maneira:

- *N* é o SLA composto para o aplicativo implantado em uma região.
- *R* é o número de regiões em que o aplicativo é implantado.

É a possibilidade esperada de que o aplicativo falhar em todas as regiões ao mesmo tempo ((1 − N) \^ R). Por exemplo, se o SLA de região única é de 99,95%:

- O SLA combinado para duas regiões = (1 − (1 − 0.9995) \^ 2) = % 99.999975
- O SLA combinado para quatro regiões = (1 − (1 − 0.9995) \^ 4) = % 99.999999

O [SLA do Traffic Manager](https://azure.microsoft.com/en-us/support/legal/sla/traffic-manager/v1_0/) também é um fator. O failover não é instantâneo em configurações de ativo-passivo, o que podem resultar em tempo de inatividade durante um failover. Ver [failover e monitoramento de ponto de extremidade do Gerenciador de tráfego](/azure/traffic-manager/traffic-manager-monitoring).

## <a name="next-steps"></a>Próximos passos

> [!div class="nextstepaction"]
> [Arquiteto de resiliência e disponibilidade](./architect.md)
