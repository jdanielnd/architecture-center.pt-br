---
title: Design de autorrecuperação
titleSuffix: Azure Application Architecture Guide
description: Aplicativos resilientes podem recuperar-se de falhas sem intervenção manual.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 5e5af0be41fa892e490d556ef4286d5367144fd9
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249501"
---
# <a name="design-for-self-healing"></a>Design de autorrecuperação

## <a name="design-your-application-to-be-self-healing-when-failures-occur"></a>Desenvolva seu aplicativo para realizar autorrecuperação quando houver falhas

Em um sistema distribuído, falhas de acontecem. O hardware pode falhar. A rede pode ter falhas transitórias. Raramente, um serviço ou uma região inteira pode sofrer uma interrupção. Mesmo assim, devemos estar preparados para isso.

Portanto, desenvolva um aplicativo para realizar autorrecuperação quando houver falhas. Isso requer uma abordagem em três fases:

- Detectar falhas.
- Responder a falhas normalmente.
- Registrar em log e monitorar falhas para fornecer insights operacionais.

Como você responde a um determinado tipo de falha pode depender dos requisitos de disponibilidade do seu aplicativo. Por exemplo, se você precisar de disponibilidade muito alta, poderá automaticamente fazer failover para uma região secundária durante uma interrupção regional. No entanto, isso acarretará um custo maior do que uma implantação de região única.

Além disso, não considere apenas grandes eventos, como interrupções regionais, que costumam ser raros. Você deve focar tanto ou mais no tratamento de falhas locais de curta duração, como falhas de conectividade de rede ou falhas em conexões com o banco de dados.

## <a name="recommendations"></a>Recomendações

**Repita as operações com falha**. As falhas transitórias podem ocorrer por perda momentânea de conectividade de rede, interrupção na conexão de banco de dados ou tempo limite atingido quando um serviço está ocupado. Crie lógica de novas tentativas em seu aplicativo para lidar com falhas transitórias. Para muitos serviços do Azure, o SDK do cliente implementa novas tentativas automáticas. Para saber mais, confira [Transient fault handling][transient-fault-handling] (Tratamento de falha transitória) e [Retry pattern][retry] (Padrão de nova tentativa).

**Proteger os serviços remotos com falha (interruptor de circuito)**. É bom tentar novamente após uma falha temporária, mas se a falha persistir, você poderá acabar com muitos chamadores insistindo em um serviço com falha. Isso pode levar a falhas em cascata, conforme as solicitações se acumulam. Use o [Padrão de Interruptor de Circuito][circuit-breaker] para falhar rapidamente (sem fazer a chamada remota) quando uma operação provavelmente for falhar.

**Isolar recursos críticos (bulkhead)**. Falhas em um subsistema às vezes podem formar uma cascata. Isso pode acontecer se uma falha impedir que alguns recursos, como threads ou soquetes, sejam liberados em tempo hábil, exaurindo recursos. Para evitar isso, particione um sistema em grupos isolados para que a falha em uma partição não deixe todo o sistema inoperante.

**Executar o nivelamento de carga**. Os aplicativos podem enfrentar picos repentinos no tráfego, que podem sobrecarregar o serviços no back-end. Para evitar isso, use o [Padrão de Nivelamento de Carga Baseado em Fila][load-level] para enfileirar itens de trabalho para que sejam executados de maneira assíncrona. A fila atua como buffer e diminui os picos na carga.

**Failover**. Se não for possível alcançar uma instância, faça o failover para outra instância. Para elementos sem estado, como um servidor Web, coloque várias instâncias atrás de um balanceador de carga ou gerenciador de tráfego. Para elementos que armazenam estado, como um banco de dados, use réplicas e failover. Dependendo do armazenamento de dados e como ele replica, isso pode exigir que o aplicativo lide com possível consistência.

**Compensar transações com falha**. Em geral, evite transações distribuídas, pois elas exigem a coordenação entre serviços e recursos. Em vez disso, componha uma operação usando transações individuais menores. Se a operação falhar no meio, use [Como compensar transações][compensating-transactions] para desfazer qualquer etapa já concluída.

**Executar ponto de verificação de transações de execução longa**. Pontos de verificação podem oferecer resiliência quando uma operação de execução longa falha. Quando a operação é reiniciada (por exemplo, é capturada por outra VM), ela pode ser retomada do último ponto de verificação.

**Degradar normalmente**. Às vezes, você não pode solucionar um problema, mas pode fornecer funcionalidade reduzida que ainda seja útil. Considere um aplicativo que mostra um catálogo de livros. Se o aplicativo não puder recuperar a imagem em miniatura da capa, poderá mostrar uma imagem de espaço reservado. Subsistemas inteiros podem não ser críticos para o aplicativo. Por exemplo, em um site de comércio eletrônico, mostrar as recomendações de produto é provavelmente menos crítico que processar pedidos.

**Limitar os clientes**. Às vezes, um pequeno número de usuários cria carga excessiva, o que pode reduzir a disponibilidade do aplicativo para outros usuários. Nessa situação, limite o cliente a um determinado período. Confira o [Padrão de limitação][throttle].

**Bloquear atores ruins**. Só porque você limita um cliente, isso não significa cliente estava agindo maliciosamente. Significa apenas que o cliente excedeu sua cota de serviço. Porém, se um cliente consistentemente exceder sua cota ou comporta-se incorretamente de outra maneira, você poderá bloqueá-lo. Defina um processo fora de banda para o usuário solicitar seu desbloqueio.

**Usar eleição de líder**. Quando você precisar coordenar uma tarefa, use [Eleição de Líder][leader-election] para selecionar um coordenador. Dessa forma, o coordenador não será um ponto único de falha. Se o coordenador falhar, um novo será selecionado. Em vez de implementar um algoritmo de eleição líder do zero, considere uma solução comercial, como Zookeeper.

**Teste com injeção de falha**. Com muita frequência, o caminho de sucesso é bem testado, mas não o caminho de falha. Um sistema pode executar em produção por um longo tempo antes que um caminho de falha seja utilizado. Use a injeção de teste para testar a resiliência do sistema a falhas, seja disparando falhas reais ou simulando-as.

**Adotar a engenharia caos**. A engenharia do caos estende a noção de injeção de falha injetando aleatoriamente falhas ou condições anormais em instâncias de produção.

Para uma abordagem estruturada para tornar seus aplicativos aptos a realizar autorrecuperação, consulte [Desenvolvimento de aplicativos resilientes para o Azure][resiliency-overview].

<!-- links -->

[circuit-breaker]: ../../patterns/circuit-breaker.md
[compensating-transactions]: ../../patterns/compensating-transaction.md
[leader-election]: ../../patterns/leader-election.md
[load-level]: ../../patterns/queue-based-load-leveling.md
[resiliency-overview]: ../../resiliency/index.md
[retry]: ../../patterns/retry.md
[throttle]: ../../patterns/throttling.md
[transient-fault-handling]: ../../best-practices/transient-faults.md
