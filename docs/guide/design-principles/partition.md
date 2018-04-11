---
title: Partição de limites
description: Usar o particionamento para contornar os limites de banco de dados, de rede e de computação
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 4371490385b24230551bf17db0075052f320b574
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="partition-around-limits"></a>Partição de limites

## <a name="use-partitioning-to-work-around-database-network-and-compute-limits"></a>Usar o particionamento para contornar os limites de banco de dados, de rede e de computação

Na nuvem, todos os serviços têm limites em sua capacidade de escalar verticalmente. Os limites de serviço do Azure estão documentados na [Assinatura e limites de serviço, cotas e restrições do Azure][azure-limits]. Os limites incluem o número de núcleos, o tamanho do banco de dados, a taxa de transferência de consulta e a taxa de transferência de rede. Se o sistema ficar suficientemente grande, você poderá atingir um ou mais desses limites. Use o particionamento para solucionar esses limites.

Há várias maneiras de particionar um sistema, como:

- Partição de um banco de dados para evitar o limite de tamanho do banco de dados, E/S de dados ou número de sessões simultâneas.

- Partição de uma fila ou mensagem de barramento para evitar limites sobre o número de solicitações ou o número de conexões simultâneas.

- Partição de um aplicativo Web do Serviço de Aplicativo para evitar limites no número de instâncias por plano do Serviço de Aplicativo. 

Um banco de dados pode ser particionado *horizontalmente*, *verticalmente* ou *funcionalmente*.

- No particionamento horizontal, também chamado de fragmentação, cada partição contém dados para um subconjunto do conjunto de dados total. As partições compartilham o mesmo esquema de dados. Por exemplo, os clientes cujos nomes começam com A&ndash;M entram em uma partição, N&ndash;Z em outra partição.

- Com o particionamento vertical, cada partição contém um subconjunto dos campos para os itens no armazenamento de dados. Por exemplo, coloque os campos acessados com frequência em uma partição vertical e os campos acessados com menos frequência em outra.

- No particionamento funcional, os dados são particionados de acordo com a forma como eles são usados por cada contexto limitado no sistema. Por exemplo, armazene dados da fatura em uma partição e os dados de estoque de produtos em outra. Os esquemas são independentes.

Para obter diretrizes mais detalhadas, veja [Particionamento de dados][data-partitioning-guidance].

## <a name="recommendations"></a>Recomendações

**Particione partes diferentes do aplicativo**. Os bancos de dados são candidatos óbvios ao particionamento, mas também considere as instâncias de armazenamento, cache, filas e computação.

**Projete a chave de partição para evitar pontos de acesso**. Se você particionar um banco de dados, mas se um fragmento ainda tiver a maioria das solicitações, você ainda não terá solucionado o problema. Idealmente, a carga é distribuída uniformemente entre todas as partições. Por exemplo, o hash por ID de cliente e não a primeira letra do nome do cliente, pois algumas letras são mais frequentes. O mesmo princípio se aplica no particionamento de uma fila de mensagens. Selecione uma chave de partição que leve a uma distribuição uniforme de mensagens em um conjunto de filas. Para saber mais, veja [Fragmentação][sharding].

**Particione em torno da assinatura do Azure e limites de serviço**. Os componentes e os serviços individuais têm limites, mas também há limites para assinaturas e grupos de recursos. Para aplicativos muito grandes, talvez seja necessário particionar em torno desses limites.  

**Particione em níveis diferentes**. Considere um servidor de banco de dados implantado em uma máquina virtual. A máquina virtual tem um VHD cujo backup é feito pelo Armazenamento do Azure. A conta de armazenamento pertence a uma assinatura do Azure. Observe que cada etapa na hierarquia tem limites. O servidor de banco de dados pode ter um limite de pool de conexão. As máquinas virtuais têm limites de CPU e de rede. O Armazenamento tem limites de IOPS. A assinatura tem limites no número de núcleos de VM. Geralmente, é mais fácil particionar a parte inferior na hierarquia. Somente os aplicativos grandes devem precisar de particionamento no nível da assinatura. 

<!-- links -->

[azure-limits]: /azure/azure-subscription-service-limits
[data-partitioning-guidance]: ../../best-practices/data-partitioning.md
[sharding]: ../../patterns/sharding.md

 