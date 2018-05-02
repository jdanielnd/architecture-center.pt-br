---
title: Padrão de Camada Anticorrupção
description: Implemente uma camada de fachada ou adaptador entre um aplicativo moderno e um sistema herdado.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: ac898519c9aa0a0aa2301da9f48756db0eb2af7c
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/16/2018
---
# <a name="anti-corruption-layer-pattern"></a>Padrão de Camada Anticorrupção

Implemente uma camada de fachada ou de adaptador entre diferentes subsistemas que não compartilham a mesma semântica. Essa camada move solicitações feitas por um subsistema para o outro subsistema. Use esse padrão para garantir que o design do aplicativo não seja limitado por dependências em subsistemas externos. Esse padrão foi descrito pela primeira vez por Eric Evans em *Design orientado ao domínio*.

## <a name="context-and-problem"></a>Contexto e problema

A maioria dos aplicativos depende de outros sistemas para alguns dados ou funcionalidade. Por exemplo, quando um aplicativo herdado é migrado para um sistema moderno, talvez ainda precise de recursos herdados existentes. Os novos recursos devem ser capazes de chamar o sistema herdado. Isso é especialmente verdadeiro para migrações graduais, onde os diferentes recursos de um aplicativo maior são movidos para um sistema moderno ao longo do tempo.

Geralmente, esses sistemas herdados sofrem problemas de qualidade, como esquemas de dados diferentes ou APIs obsoletas. Os recursos e tecnologias usados em sistemas herdados podem variar amplamente em sistemas mais modernos. Para interagir com o sistema herdado, pode ser necessário que o novo aplicativo dê suporte a infraestrutura, protocolos, modelos de dados, APIs ou outros recursos desatualizados que você não colocaria em um aplicativo moderno.

Manter o acesso entre sistemas herdados e novos pode forçar o novo sistema a aderir a pelo menos algumas das APIs do sistema herdado ou outras semânticas. Quando esses recursos herdados tiverem problemas de qualidade, o suporte a eles "corrompe" o que, caso contrário, seria um aplicativo moderno projetado corretamente. 

Problemas semelhantes podem surgir com qualquer sistema externo que sua equipe de desenvolvimento não controle, não apenas com sistemas herdados. 

## <a name="solution"></a>Solução

Isole os diferentes subsistemas colocando uma camada anticorrupção entre eles. Essa camada move as comunicações entre os dois sistemas, permitindo que um sistema permaneça inalterado, enquanto o outro pode evitar comprometer seu design e sua abordagem tecnológica.

![](./_images/anti-corruption-layer.png) 

O diagrama acima mostra um aplicativo com dois subsistemas. O Subsistema A chama para o Subsistema B por meio de uma camada anticorrupção. A comunicação entre o Subsistema A e a camada anticorrupção sempre usa o modelo de dados e a arquitetura do subsistema A. Chamadas da camada anticorrupção para o Subsistema B estão em conformidade com o modelo de dados ou os métodos desse subsistema. A camada anticorrupção contém toda a lógica necessária para traduzir entre os dois sistemas. A camada pode ser implementada como um componente do aplicativo ou como um serviço independente.

## <a name="issues-and-considerations"></a>Problemas e considerações

- A camada anticorrupção pode adicionar latência para chamadas feitas entre os dois sistemas.
- A camada anticorrupção adiciona outro serviço que deve ser gerenciado e mantido.
- Considere como a camada anticorrupção será dimensionada.
- Considere se você precisa de mais de uma camada anticorrupção. Talvez você queira decompor a funcionalidade em vários serviços usando tecnologias ou linguagens diferentes, ou pode haver outros motivos para particionar a camada anticorrupção.
- Considere como a camada anticorrupção será gerenciada em relação com outros aplicativos ou serviços. Como ele será integrado aos processos de monitoramento, versão e configuração?
- Certifique-se de que a transação e a consistência de dados sejam mantidas e que possam ser monitoradas.
- Leve em consideração se a camada anticorrupção precisa lidar com toda a comunicação entre diferentes subsistemas ou apenas com um subconjunto de recursos. 
- Se a camada anticorrupção fizer parte de uma estratégia de migração, leve em consideração se ela será permanente ou se será desativada depois que todas as funcionalidades herdadas forem migradas.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use esse padrão quando:

- Haja um plano de migração que acontecerá em vários estágios, mas a integração entre os sistemas novo e herdado precisará ser mantida.
- Dois ou mais subsistemas têm semânticas diferentes, mas ainda precisam se comunicar. 

Esse padrão poderá não ser adequado se não houver nenhuma diferença significativa semântica entre os sistemas herdados e novos. 

## <a name="related-guidance"></a>Diretrizes relacionadas

- [Padrão do estrangulador](./strangler.md)
