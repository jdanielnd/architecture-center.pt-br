---
title: Design de mudança
description: Um design evolutivo é a chave para a inovação contínua.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: d05c1813dbc49f3ed8378cac4ea0c584ebdd9ff7
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252883"
---
# <a name="design-for-evolution"></a>Design para evolução

## <a name="an-evolutionary-design-is-key-for-continuous-innovation"></a>Um design evolutivo é a chave para a inovação contínua

Todos os aplicativos de sucesso mudam ao longo do tempo, seja para corrigir bugs, adicionar novos recursos, apresentar novas tecnologias ou deixar sistemas existentes mais escalonáveis e flexíveis. Se todas as partes de um aplicativo estão firmemente acopladas, fica muito difícil introduzir alterações no sistema. Uma alteração em uma parte do aplicativo pode interromper outra parte ou fazer com que as alterações sejam propagadas à toda a base de código.

Esse problema não é limitado aos aplicativos monolíticos. Um aplicativo pode ser decomposto em serviços e continuar exibindo o tipo de acoplamento firme que deixa o sistema rígido e frágil. Mas quando os serviços são projetados para serem desenvolvidos, as equipes podem inovar e entregar novos recursos continuamente. 

Os microsserviços estão se tornando uma maneira popular de obter um design evolutivo, pois abordam muitas das considerações listadas aqui.

## <a name="recommendations"></a>Recomendações

**Impor alta coesão e acoplamento flexível**. Um serviço é *coeso* se fornecer uma funcionalidade que está logicamente interligada. Os serviços têm *acoplamento flexível* se for possível alterar um serviço sem alterar outro. Alta coesão geralmente significa que as alterações feitas em uma função exigirão alterações em outras funções relacionadas. Se você descobrir que a atualização de um serviço requer atualizações coordenadas com outros serviços, isso pode ser um sinal de que os serviços não são coesos. Uma das metas do design orientado a domínio (DDD) é identificar esses limites.

**Encapsular o conhecimento de domínio**. Quando um cliente consome um serviço, a imposição das regras de negócio do domínio não deve ficar sob a responsabilidade do cliente. Em vez disso, o serviço deve encapsular todo o conhecimento de domínio que esteja sob sua responsabilidade. Caso contrário, cada cliente tem de impor as regras de negócio, e você acaba com conhecimento do domínio espalhado entre diferentes partes do aplicativo. 

**Usar mensagens assíncronas**. O sistema de mensagens assíncronas é uma forma de desacoplar o produtor de mensagem do consumidor. O produtor não depende da resposta do consumidor à mensagem ou da tomada de qualquer ação específica. Com uma arquitetura de publicação/assinatura, talvez o produtor nem saiba quem está consumindo a mensagem. Novos serviços podem facilmente consumir as mensagens sem nenhuma modificação ao produtor.

**Não compilar conhecimento de domínio em um gateway**. Gateways podem ser úteis em uma arquitetura de microsserviços para coisas como roteamento de solicitação, conversão de protocolo, balanceamento de carga ou autenticação. No entanto, o gateway deve ser restrito a esse tipo de funcionalidade de infraestrutura. Ele não deve implementar nenhum conhecimento de domínio para evitar se tornar uma dependência pesada.

**Expor interfaces abertas**. Evite criar camadas de conversão personalizada que ficam entre serviços. Em vez disso, um serviço deve expor uma API com um contrato de API bem definido. A API deve ter uma versão para que seja possível evoluir a API e manter a compatibilidade com versões anteriores. Dessa forma, é possível atualizar um serviço sem coordenar as atualizações para todos os serviços de upstream que dependem dele. Serviços voltados ao público devem expor uma API RESTful via HTTP. Serviços de back-end podem usar um protocolo de mensagens estilo RPC por motivos de desempenho. 

**Projetar e testar em comparação a contratos de serviço**. Quando serviços expõem APIs bem definidas, é possível desenvolver e testar em comparação a essas APIs. Dessa forma, você pode desenvolver e testar um serviço individual sem afetar todos os seus serviços dependentes. (É claro que ainda será realizada a integração e o teste de carga em comparação aos serviços reais.)

**Infraestrutura abstrata distante da lógica do domínio**. Não deixe a lógica do domínio se misturar a funcionalidades relacionadas à infraestrutura, como sistema de mensagens ou persistência. Caso contrário, as alterações na lógica do domínio exigirão atualizações para as camadas de infraestrutura e vice-versa. 

**Descarregar preocupações abrangentes para um serviço separado**. Por exemplo, se vários serviços precisarem autenticar solicitações, seria possível mover essa funcionalidade para seu próprio serviço. Depois seria possível evoluir o serviço de autenticação &mdash; por exemplo, adicionando um novo fluxo de autenticação &mdash; sem tocar em qualquer serviço que o utiliza.

**Implantar serviços de modo independente**. Quando a equipe de DevOps pode implantar um único serviço independentemente de outros serviços do aplicativo, as atualizações podem ocorrer com mais rapidez e segurança. Correções de bugs e novos recursos podem ser implantados em um ritmo mais regular. Projete o aplicativo e o processo de lançamento para oferecerem suporte a atualizações independentes.