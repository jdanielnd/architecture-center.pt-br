---
title: Desenvolver aplicativos do Azure confiáveis
description: Introdução a como tornar aplicativos do Azure confiáveis e altamente disponíveis
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: ''
ms.openlocfilehash: 16c1daeed9b1d79f33051eb69571f1574bf9be4d
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59644000"
---
# <a name="designing-reliable-azure-applications"></a>Como desenvolver aplicativos do Azure confiáveis

Criar um aplicativo confiável na nuvem é diferente do desenvolvimento de aplicativos tradicional. Embora historicamente você possa ter comprado um hardware superior para escalar verticalmente, em um ambiente de nuvem, é preciso escalar horizontalmente e não verticalmente. Em vez de tentar evitar completamente a falha, a meta é minimizar os efeitos de uma falha em um componente individual.

Aplicativos confiáveis são:

- **Resilientes** e se recuperam bem de falhas, além de continuarem a funcionar com mínimo tempo de inatividade e perda de dados antes de recuperação completa.
- **HA (altamente disponíveis)** e executam corretamente em um estado íntegro, sem tempo de inatividade significativo.

Compreender como esses elementos funcionam juntos &mdash; e como eles afetam o custo &mdash; é essencial para criar um aplicativo confiável. Isso pode ajudá-lo a determinar quanto tempo de inatividade é aceitável, o custo potencial para o seu negócio e quais funções são necessárias durante uma recuperação.

Este artigo fornece uma visão geral da criação de confiabilidade em cada etapa do processo de desenvolvimento de aplicativo do Azure. Cada seção inclui um link para um artigo detalhado sobre como integrar a confiabilidade a essa etapa específica no processo. Se você estiver procurando por considerações de confiabilidade para serviços individuais do Azure, examine a [Lista de verificação de resiliência para serviços específicos do Azure](../checklist/resiliency-per-service.md).

## <a name="build-for-reliability"></a>Criar para confiabilidade

Esta seção descreve seis etapas para criar um aplicativo do Azure confiável. Cada etapa vincula a uma seção que define mais detalhadamente o processo e os termos.

1. [**Defina os requisitos.**](#define-requirements) Desenvolva requisitos de disponibilidade e de recuperação com base em cargas de trabalho decompostas e necessidades de negócios.
1. [**Use as melhores práticas de arquitetura.**](#use-architectural-best-practices) Siga as melhores práticas, identifique os pontos de falha possíveis na arquitetura e determine como o aplicativo responderá à falha.
1. [**Teste com simulações e failovers forçados.**](#test-with-simulations-and-forced-failovers) Simule falhas, dispare failovers forçados e teste a detecção e a recuperação dessas falhas.
1. [**Implante o aplicativo de forma consistente.**](#deploy-the-application-consistently) Libere para produção usando processos confiáveis e reproduzíveis.
1. [**Monitore a integridade do aplicativo.**](#monitor-application-health) Detecte falhas, monitore os indicadores de possíveis falhas e avalie a integridade de seus aplicativos.
1. [**Responda a falhas e desastres.**](#respond-to-failures-and-disasters) Identifique quando uma falha ocorre e determine como solucioná-la com base em estratégias estabelecidas.

## <a name="define-requirements"></a>Definir requisitos

Identifique suas necessidades de negócios e compile seu plano de confiabilidade para solucioná-las. Considere o seguinte:

- **Identifique as cargas de trabalho e o uso.** Uma *carga de trabalho* é uma funcionalidade ou tarefa distinta que é logicamente separada de outras tarefas, em termos de requisitos de armazenamento de dados e lógica de negócios. Cada carga de trabalho pode ter diferentes requisitos de disponibilidade, escalabilidade, consistência de dados e recuperação de desastre.
- **Planeje quanto aos padrões de uso.** *Padrões de uso* também desempenham um papel nos requisitos. Identifique as diferenças em requisitos durante períodos críticos e não críticos. Por exemplo, um aplicativo de declaração de imposto não pode falhar durante um prazo de entrega da declaração. Para garantir o tempo de atividade, planeje a redundância entre várias regiões para o caso de uma delas falhar. Por outro lado, para minimizar os custos durante períodos não críticos, você pode executar seu aplicativo em uma única região.
- **Estabeleça métricas de disponibilidade &mdash; MTTR (*tempo médio até a recuperação*) e MTBF (*tempo médio entre falhas*).** O MTTR é o tempo médio necessário para restaurar um componente após uma falha. O MTBF é a duração que pode ser razoavelmente esperada de um componente entre interrupções. Use essas medidas para determinar onde adicionar redundância e para determinar SLAs (Contratos de Nível de Serviço) para clientes.  
- **Estabeleça métricas de recuperação &mdash; objetivo do tempo de recuperação e RPO (objetivo do ponto de recuperação).** *RTO* é o tempo máximo aceitável que um aplicativo pode ficar indisponível após um incidente. *RPO* é a duração máxima de perda de dados que é aceitável durante um desastre. Para obter esses valores, realize uma avaliação de risco e verifique se você entendeu o custo e o risco de tempo de inatividade ou perda de dados em sua organização.
    > [!NOTE]
    > Se o MTTR de *qualquer* componente crítico em uma configuração altamente disponível exceder o RTO do sistema, uma falha no sistema poderá causar uma interrupção inaceitável do negócio. Ou seja, não é possível restaurar o sistema dentro do RTO definido.
- **Determine os destinos de disponibilidade da carga de trabalho.** Para garantir que a arquitetura do aplicativo atenda aos seus requisitos de negócios, defina os SLAs de destino para cada carga de trabalho. Considere o custo e a complexidade de atender aos requisitos de disponibilidade, além das dependências do aplicativo.
- **Entenda os SLAs (Contratos de Nível de Serviço).** No Azure, o SLA descreve os compromissos da Microsoft em relação ao tempo de atividade e à conectividade. Se o SLA para um serviço específico for de 99,9%, você deverá esperar que o serviço esteja disponível 99,9% do tempo.  

    Defina seus próprios SLAs de destino para cada carga de trabalho em sua solução, para que você possa determinar se a arquitetura atende aos requisitos de negócios. Por exemplo, se uma carga de trabalho exigir tempo de atividade de 99,99%, mas depender de um serviço com um SLA de 99,9%, o serviço não poderá ser um ponto único de falha no sistema.

Para obter mais informações sobre como desenvolver os requisitos para aplicativos confiáveis, consulte [Como desenvolver requisitos para aplicativos resilientes do Azure](./requirements.md).

## <a name="use-architectural-best-practices"></a>Use as melhores práticas de arquitetura

Durante a fase de arquitetura, concentre-se na implementação de práticas que atendem aos seus requisitos de negócios, identificam pontos de falha e minimizam o escopo de falhas.

- **Execute uma FMA (Análise do modo de falha).** A FMA cria resiliência em um aplicativo no início do estágio de design. Ela ajuda você a identificar os tipos de falhas de que seu aplicativo pode enfrentar, os possíveis efeitos de cada um deles e possíveis estratégias de recuperação.
- **Crie um plano de redundância.** O nível de redundância necessário para cada carga de trabalho depende de suas necessidades de negócios e compõe o custo geral de seu aplicativo. 
- **Desenvolva buscando escalabilidade.** Um aplicativo em nuvem deve ser capaz de escalar para acomodar as alterações no uso. Comece com componentes discretos e desenvolva o aplicativo para responder automaticamente ao carregamento de alterações sempre que possível. Mantenha os limites de dimensionamento em mente durante o desenvolvimento para que você possa expandir facilmente no futuro.
- **Planeje os requisitos de assinatura e de serviço.** Você pode precisar de assinaturas adicionais para provisionar recursos suficientes para atender aos requisitos de negócios para armazenamento, conexões, taxa de transferência e muito mais. 
- **Use o balanceamento de carga para distribuir solicitações.** O balanceamento de carga distribui as solicitações do aplicativo para instâncias de serviço íntegras removendo instâncias não íntegras da rotação.
- **Implemente estratégias de resiliência.** *Resiliência* é a capacidade de um sistema de se recuperar de falhas e continuar funcionando. Sempre que possível, implemente [padrões de design de resiliência](../patterns/category/resiliency.md), tais como isolamento de recursos críticos, uso de transações de compensação e execução de operações assíncronas.
- **Crie requisitos de disponibilidade em seu design.** A *disponibilidade* é a proporção de tempo em que o sistema está funcional e em execução. Tome medidas para garantir a disponibilidade de aplicativos em conformidade com o contrato de nível de serviço. Por exemplo, evite pontos únicos de falha, decomponha as cargas de trabalho por objetivo de nível de serviço e limite os usuários de alto volume.
- **Gerencie seus dados.** O modo como você armazena, faz backup e replica os dados é essencial.

  - **Escolha os métodos de replicação para os dados do aplicativo.** Os dados do aplicativo são armazenados em diferentes armazenamentos de dados e podem ter requisitos de disponibilidade diferentes. Avalie os métodos de replicação e os locais de cada tipo de armazenamento de dados para garantir que ele atenda às suas necessidades.
  - **Documente e teste os processos de failover e failback.** Documente claramente as instruções para fazer failover para um novo armazenamento de dados. Teste-as regularmente para verificar se elas são precisas e fáceis de acompanhar.
  - **Proteja seus dados.** Faça backup dos dados e valide-os regularmente. Assegure que nenhuma conta de usuário individual tenha acesso tanto aos dados de backup quanto aos de produção.
  - **Planeje a recuperação de dados.** Verifique se sua estratégia de backup e replicação fornece tempos de recuperação de dados que atendem aos seus requisitos de nível de serviço. Contabilize todos os tipos de dados usados pelo seu aplicativo, incluindo bancos de dados e dados de referência.

Para obter mais informações sobre como arquitetar aplicativos confiáveis, consulte [Como arquitetar aplicativos do Azure para resiliência e disponibilidade](./architect.md).

## <a name="test-with-simulations-and-forced-failovers"></a>Testar com simulações e failovers forçados

O teste de confiabilidade requer medição do desempenho da carga de trabalho de ponta a ponta sob condições de falha intermitentes.

- **Teste para cenários de falha comuns disparando falhas reais ou simulando-as.** Use o teste de injeção de falha para testar cenários comuns (incluindo combinações de falhas) e o tempo de recuperação.
- **Identifique falhas que ocorrem apenas sob carga.** Teste a carga de pico usando dados de produção ou sintéticos, desde que estes se aproximem o máximo possível dos dados de produção, para ver como é o comportamento do aplicativo sob condições do mundo real.
- **Execute simulações de recuperação de desastre.** Tenha um plano para recuperação de desastre em vigor e teste-o periodicamente para verificar se ele funciona.
- **Execute teste de failover e de failback.** Verifique se os serviços que dependem do seu aplicativo fazem failover e failback na ordem correta.
- **Execute testes de simulação.** Testar cenários da vida real pode realçar problemas que precisam ser resolvidos. Cenários devem ser controláveis e sem interrupções para os negócios. Informe à gerência sobre planos de teste com simulação.
- **Teste as investigações de integridade.** Configure investigações de integridade para balanceadores de carga e gerenciadores de tráfego para verificar os componentes essenciais do sistema. Teste-as para verificar se elas respondem adequadamente.
- **Teste os sistemas de monitoramento.** Verifique se os sistemas de monitoramento estão relatando dados precisos e informações críticas de modo confiável para ajudar a identificar possíveis falhas.
- **Inclua serviços de terceiros em cenários de teste.** Teste possíveis pontos de falha devido à interrupção do serviço de terceiros, bem como a recuperação.

O teste é um processo iterativo. Teste o aplicativo, avalie o resultado, analise e resolva eventuais falhas e repita o processo.

Para obter mais informações sobre como testar a confiabilidade do aplicativo, veja [Como testar aplicativos do Azure para resiliência e disponibilidade](./testing.md).

## <a name="deploy-the-application-consistently"></a>Implantar o aplicativo de forma consistente

A *implantação* inclui etapas de provisionamento de recursos do Azure, implantação do código do aplicativo e aplicação de definições de configuração. Uma atualização pode envolver todas as três tarefas ou um subconjunto delas.

Após um aplicativo ser implantado para produção, as atualizações são uma possível fonte de erros. Minimize os erros com processos de implantação reproduzíveis e previsíveis.

- **Automatize o processo de implantação do aplicativo.** Automatize o maior número possível de processos.
- **Projete seu processo de lançamento para maximizar a disponibilidade.** Se o processo de lançamento exigir que os serviços fiquem offline durante a implantação, o aplicativo ficará não disponível até que eles fiquem online novamente. Aproveite os recursos de preparo e produção da plataforma. Usar lançamentos canário ou "blue-green" para implantar atualizações, de modo que, se ocorrer uma falha, você poderá rapidamente reverter a atualização.
- **Tenha um plano de reversão para a implantação.** Crie um processo de reversão para voltar para uma última versão válida conhecida e minimizar o tempo de inatividade em caso de falha em uma implantação.
- **Registre as implantações e audite-as.** Se você usar técnicas de implantação com preparo, haverá mais de uma versão do aplicativo em execução na produção. Implemente uma estratégia de registro em log robusta para capturar o máximo possível de informações específicas da versão.
- **Documente o processo de lançamento do aplicativo.** Defina e documente claramente seu processo de lançamento e verifique se ele está disponível para toda a equipe de operações.

Para obter mais informações sobre como testar a implantação e a confiabilidade do aplicativo, veja [Como implantar aplicativos do Azure para resiliência e disponibilidade](./deploy.md).

## <a name="monitor-application-health"></a>Monitorar a integridade do aplicativo

Implemente práticas recomendadas para monitoramento e alertas em seu aplicativo para que você possa detectar falhas e alertar um operador para corrigi-las.

- **Implemente investigações de integridade e verifique as funções.** Execute-as regularmente de fora do aplicativo para identificar a degradação do desempenho e da integridade do aplicativo.
- **Verifique os fluxos de trabalho de execução longa.** Detectar problemas antecipadamente pode minimizar a necessidade de reverter todo o fluxo de trabalho ou de executar várias transações de compensação.
- **Mantenha os logs do aplicativo.**

  - Registre em log aplicativos em produção e em limites de serviço.
  - Use o registro em log semântico e assíncrono.
  - Separe os logs do aplicativo dos logs de auditoria.

- **Meça estatísticas de chamada remota e compartilhe dados com a equipe de aplicativos.** Para fornecer à sua equipe de operações uma exibição instantânea da integridade do aplicativo, resuma métricas de chamada remota, tais como latência, taxa de transferência e erros nos percentuais 99 e 95. Execute análise estatística nas métricas para revelar os erros que ocorrem dentro de cada percentual.
- **Acompanhe as exceções transitórias e novas tentativas durante um período de tempo apropriado.** Uma tendência de aumento de exceções ao longo do tempo indica que o serviço está apresentando um problema e pode falhar.
- **Configure um sistema de avisos antecipados.** Identifique os KPIs (indicadores chave de desempenho) de integridade de um aplicativo, tais como exceções transitórias e latência de chamadas remotas e defina valores limite adequados para cada um deles. Envie um alerta para operações quando o valor limite é atingido.
- **Opere dentro dos limites de assinatura do Azure.** As assinaturas do Azure têm limites para determinados tipos de recursos, tais como o número de grupos de recursos, núcleos e contas de armazenamento. Observe seu uso dos tipos de recursos.
- **Monitore os serviços de terceiros.** Registre suas invocações e correlacione-as com a integridade e o log de diagnóstico do aplicativo usando um identificador exclusivo.
- **Treine vários operadores para monitorar o aplicativo e executar etapas de recuperação manual.** Verifique se há sempre pelo menos um operador treinado ativo.

Para obter mais informações sobre o monitoramento da confiabilidade do aplicativo, veja [Como monitorar a integridade do aplicativo do Azure](./monitoring.md).

## <a name="respond-to-failures-and-disasters"></a>Responder a falhas e desastres

Crie um plano de recuperação e verifique se ele aborda a restauração de dados, interrupções de rede, falhas de serviços dependentes e interrupções de serviço em toda a região. Considere suas VMs, armazenamento, bancos de dados e outros serviços da plataforma Azure em sua estratégia de recuperação.

- **Planeje as interações com o Suporte do Azure.** Antes da necessidade surgir, estabeleça um processo para entrar em contato com o Suporte do Azure.
- **Documente e teste seu plano de recuperação de desastre.** Escreva um plano de recuperação de desastre que reflita o impacto das falhas do aplicativo nos negócios. Automatize o processo de recuperação tanto quanto possível e documente eventuais etapas manuais. Teste regularmente seu processo de recuperação de desastres, para validar e aprimorar o plano.
- **Faça failover manualmente quando necessário.** Alguns sistemas não conseguem executar failover automaticamente e requerem um failover manual. Se um aplicativo fizer failover para uma região secundária, execute um teste de prontidão operacional. Verifique se a região principal está íntegra e pronta para receber o tráfego. antes de fazer failback. Determine o que é a funcionalidade reduzida do aplicativo e como o aplicativo informa os usuários sobre problemas temporários.
- **Prepare-se para falhas do aplicativo.** Prepare para uma variedade de falhas, inclusive as falhas que são tratadas automaticamente, aquelas que resultam em funcionalidade reduzida e aquelas que fazem com que o aplicativo fique indisponível. O aplicativo deve informar problemas temporários aos usuários.
- **Recupere dados corrompidos.** Se ocorrer falha em um armazenamento de dados, verifique se há inconsistências de dados quando o armazenamento fica disponível novamente, especialmente se os dados foram replicados. Restaure dados corrompidos por meio de um backup.
- **Recupere-se de uma interrupção da rede.** Você poderá usar dados armazenados em cache para executar localmente com funcionalidade reduzida do aplicativo. Caso contrário, considere a possibilidade de tempo de inatividade do aplicativo ou faça failover para outra região. Armazene seus dados em um local alternativo até que a conectividade seja restaurada.
- **Recupere-se de uma falha de serviço dependente.** Determine qual funcionalidade ainda está disponível e como o aplicativo deve responder.
- **Recupere-se de uma interrupção do serviço em toda a região.** Interrupções de serviço em toda a região são incomuns, mas você deve ter uma estratégia para solucioná-los, especialmente para aplicativos críticos. Você poderá reimplantar o aplicativo para outra região ou redistribuir o tráfego.

Para obter mais informações sobre como responder a falhas e recuperação de desastre, consulte [Recuperação de desastre e falha para aplicativos do Azure](./disaster-recovery.md).

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Desenvolver requisitos para aplicativos resilientes](./requirements.md)
