---
title: Relação de confiança descentralizada entre bancos
titleSuffix: Azure Example Scenarios
description: Estabeleça um ambiente confiável para comunicação e compartilhamento de informações sem recorrer a um banco de dados centralizado.
author: vitoc
ms.date: 09/09/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: csa-team
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-decentralized-trust.png
ms.openlocfilehash: a3c497f91b3861bf02f05981ee92e578a22a14ca
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54907972"
---
# <a name="decentralized-trust-between-banks-on-azure"></a>Relação de confiança descentralizada entre bancos no Azure

Esse cenário de exemplo é útil para bancos ou outras instituições que desejam estabelecer um ambiente confiável para o compartilhamento sem recorrer a um banco de dados centralizado de informações. Para os fins desse exemplo, descreveremos o cenário no contexto de manutenção de informações de pontuação de crédito entre bancos. Porém, a arquitetura pode ser aplicada a qualquer cenário em que um consórcio de organizações deseje compartilhar informações validadas com outro sem recorrer ao uso de um sistema central executado por uma parte única.

Tradicionalmente, os bancos em um sistema financeiro contam com fontes centralizadas, como centrais de crédito, para obter informações sobre a pontuação de crédito e o histórico de um indivíduo. Uma abordagem centralizada apresenta uma concentração de risco operacional e, às vezes, terceiros desnecessários.

Com DLTs (tecnologia de razão distribuído), um consórcio de bancos pode estabelecer um sistema descentralizado que pode ser mais eficiente e menos suscetível a ataques e servir como uma nova plataforma, em que estruturas inovadoras podem ser implementadas para resolver desafios tradicionais referentes a privacidade, velocidade e custo.

Este exemplo mostrará como serviços do Azure, como conjuntos de dimensionamento de máquina virtual, Rede Virtual, Key Vault, Armazenamento, Balanceador de Carga e Monitor, podem ser rapidamente provisionados para a implantação de um blockchain de Ethereum PoA privado e eficiente em que os bancos membros possam estabelecer seus próprios nós.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Outros casos de uso relevantes incluem:

- Movimentação de orçamentos alocados entre diferentes unidades de negócios de uma empresa multinacional
- Pagamentos internacionais
- Cenários de finanças de comércio
- Sistemas de fidelidade que envolvem diferentes empresas
- Ecossistemas de cadeia de suprimentos

## <a name="architecture"></a>Arquitetura

![Diagrama de arquitetura de confiança bancária descentralizada](./media/architecture-decentralized-trust.png)

Esse cenário aborda os componentes back-end que são necessários para criar uma rede privada, escalonável, segura e monitorada de blockchain empresarial em um consórcio de dois ou mais membros. Detalhes de como esses componentes são provisionados (ou seja, em diferentes assinaturas e grupos de recursos), bem como os requisitos de conectividade (ou seja, VPN ou ExpressRoute) são deixados para sua consideração, com base nos requisitos de política de sua organização. Veja como os dados fluem:

1. O Banco A cria/atualiza um registro de crédito de um indivíduo, enviando uma transação para a rede de blockchain por meio de JSON-RPC.
2. Os dados fluem do servidor de aplicativo privado do Banco A para o balanceador de carga do Azure e, subsequentemente, para uma VM de nó validação no conjunto de escala de máquina virtual.
3. A rede Ethereum PoA cria um bloco em um horário predefinido (dois segundos para esse cenário).
4. A transação é empacotada no bloco criado e validada em toda a rede de blockchain.
5. O Banco B pode ler o registro de crédito criado pelo banco A comunicando-se com seu próprio nó da mesma forma, por meio de JSON-RPC.

### <a name="components"></a>Componentes

- As máquinas virtuais em conjuntos de dimensionamento de máquinas virtuais fornecem o recurso de computação sob demanda para hospedar os processos de validador para o blockchain
- O Cofre de Chaves é usado como o recurso de armazenamento seguro para as chaves privadas de cada validador
- O Load Balancer distribui as solicitações de RPC, emparelhamento e Governança DApp
- Armazenamento que hospeda informações de rede persistentes e leasing de coordenação
- O Operations Management Suite (um agrupamento de alguns serviços do Azure) fornece insight sobre nós disponíveis, transações por minuto e membros do consórcio

### <a name="alternatives"></a>Alternativas

A abordagem de Ethereum PoA é escolhida para este exemplo porque é um ponto de entrada válido para um consórcio das organizações que desejam criar um ambiente em que informações possam ser facilmente trocadas e compartilhadas de maneira confiável, descentralizada e fácil de entender. Os modelos de solução do Azure disponíveis também fornecem uma maneira rápida e conveniente não apenas para um líder do consórcio iniciar um blockchain Ethereum PoA, mas também para as organizações-membro do consórcio criarem seus próprios recursos do Azure em seu próprio grupo de recursos e uma assinatura para ingressar em uma rede existente.

Para outros cenários estendidos ou diferentes, podem surgir preocupações como privacidade de transação. Por exemplo, em um cenário de transferência de valores mobiliários, os membros de um consórcio podem não querer que suas transações fiquem visíveis mesmo para outros membros. Existem alternativas para Ethereum PoA que abordam essas preocupações de sua própria maneira:

- Corda
- Quorum
- Hyperledger

## <a name="considerations"></a>Considerações

### <a name="availability"></a>Disponibilidade

O [Azure Monitor][monitor] é usado para monitorar continuamente se há problemas na rede de blockchain e garantir a disponibilidade. Um link para um painel de monitoramento personalizado com base no Azure Monitor será enviado a você após a implantação bem-sucedida do modelo de solução de blockchain usado nesse cenário. O painel mostra nós que estão relatando pulsações nos últimos 30 minutos, bem como outras estatísticas úteis.

Para ver outros tópicos sobre disponibilidade, consulte a [lista de verificação de disponibilidade][availability] no Azure Architecture Center.

### <a name="scalability"></a>Escalabilidade

Uma preocupação popular referente ao blockchain é o número de transações que ele pode incluir em um período de tempo predefinido. Esse cenário usa uma Prova de Autoridade em que tal escalabilidade pode ser gerenciada melhor do que a Prova de Trabalho. Em redes com base em Prova de Autoridade, os participantes de consenso são conhecidos e gerenciados, tornando-as mais adequadas para o blockchain privado para um consórcio de organizações que conhecem umas às outras. Os parâmetros como tempo médio de bloco, transações por minuto e consumo de recursos de computação podem ser facilmente monitorados por meio do painel personalizado. Os recursos podem ser ajustados adequadamente com base nos requisitos de dimensionamento.

Para obter diretrizes gerais sobre como criar soluções escalonáveis, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.

### <a name="security"></a>Segurança

O [Azure Key Vault][vault] é usado para armazenar e gerenciar com facilidade as chaves privadas dos validadores. A implantação padrão neste exemplo cria uma rede de blockchain que é acessível pela internet. Para um cenário de produção em que uma rede privada é desejada, os membros podem ser conectados uns aos outros por meio de conexões de gateway VPN de VNet para VNet. As etapas para configurar uma VPN são incluídas na seção de recursos relacionados abaixo.

Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].

### <a name="resiliency"></a>Resiliência

O próprio blockchain Ethereum PoA pode fornecer alguma resiliência, pois os nós de validação podem ser implantados em regiões diferentes. O Azure tem opções para implantações em mais de 54 regiões em todo o mundo. Um blockchain como o desse cenário fornece possibilidades novas e exclusivas de cooperação para aumentar a resiliência. A resiliência da rede não é fornecida apenas para por um único participante centralizado, mas por todos os membros do consórcio. Um blockchain com base em Prova de Autoridade permite que a resiliência de rede seja ainda mais planejada e deliberada.

Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].

## <a name="pricing"></a>Preços

Para explorar o custo de executar esse cenário, todos os serviços são pré-configurados na calculadora de custos. Para ver como o preço seria alterado para seu uso específico, altere as variáveis apropriadas para que sejam correspondentes a seus requisitos de desempenho e disponibilidade esperados.

Fornecemos três perfis de custo de exemplo com base no número de instâncias de VM do conjunto de dimensionamento que executam seus aplicativos (as instâncias podem residir em regiões diferentes).

- [Pequeno][small-pricing]: esse exemplo de preço refere-se a duas VMs por mês com o monitoramento desativado
- [Médio][medium-pricing]: esse exemplo de preço refere-se a sete VMs por mês com o monitoramento ativado
- [Grande][large-pricing]: esse exemplo de preço refere-se a 15 VMs por mês com o monitoramento ativado

Os preços acima são para um membro do consórcio iniciar ou ingressar em uma rede de blockchain. Normalmente, em um consórcio em que haja várias empresas ou organizações envolvidas, cada membro receberá sua própria assinatura do Azure.

## <a name="next-steps"></a>Próximas etapas

Para ver um exemplo desse cenário, implante o [aplicativo de demonstração de blockchain Ethereum PoA][deploy] no Azure. Em seguida, percorra o [LEIAME do código-fonte do cenário][source].

## <a name="related-resources"></a>Recursos relacionados

Para saber mais sobre como usar o modelo de solução de Prova de Autoridade de Ethereum para o Azure, examine este [guia de uso][guide].

<!-- links -->
[small-pricing]: https://azure.com/e/4e429d721eb54adc9a1558fae3e67990
[medium-pricing]: https://azure.com/e/bb42cd77437744be8ed7064403bfe2ef
[large-pricing]: https://azure.com/e/e205b443de3e4adfadf4e09ffee30c56
[guide]: /azure/blockchain-workbench/ethereum-poa-deployment
[deploy]: https://portal.azure.com/?pub_source=email&pub_status=success#create/microsoft-azure-blockchain.azure-blockchain-ethereumethereum-poa-consortium
[source]: https://github.com/vitoc/creditscoreblockchain
[monitor]: /azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: ../../resiliency/index.md
[security]: /azure/security/
[vault]: https://azure.microsoft.com/services/key-vault/
