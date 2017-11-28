---
title: "Padrão sidecar"
description: "Implante os componentes de um aplicativo em um processo ou contêiner separado para fornecer isolamento e encapsulamento."
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: ec168009aa99f412c3f1222a1c404ea4ea5cb669
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="sidecar-pattern"></a>Padrão sidecar

Implante os componentes de um aplicativo em um processo ou contêiner separado para fornecer isolamento e encapsulamento. Esse padrão também pode habilitar aplicativos para serem compostos por tecnologias e componentes heterogêneos.

Esse padrão é denominado *Sidecar* porque é semelhante a um sidecar anexado a uma moto. No padrão, o sidecar está anexado a um aplicativo pai e fornece recursos de suporte para o aplicativo. O sidecar também compartilha o mesmo ciclo de vida do aplicativo pai, sendo criado e desativado junto com ele. O padrão sidecar às vezes é conhecido como o padrão de sidekick e é um padrão de decomposição.

## <a name="context-and-problem"></a>Contexto e problema

Aplicativos e serviços geralmente requerem funcionalidades relacionadas, como monitoramento, registro em log, configuração e serviços de rede. Essas tarefas periféricas podem ser implementadas como componentes ou serviços separados. 

Se elas estiverem intimamente integrados ao aplicativo, poderão ser executadas no mesmo processo que o aplicativo, usando com eficiência os recursos compartilhados. No entanto, isso também significa que eles não são bem isolados e uma interrupção em um desses componentes poderia afetar os demais componentes ou todo o aplicativo. Além disso, eles geralmente precisam ser implementados usando o mesmo idioma que o aplicativo pai. Como resultado, o componente e o aplicativo têm uma interdependência íntima entre si.

Se o aplicativo for decomposto em serviços, cada serviço poderá ser compilado usando diferentes linguagens e tecnologias. Embora isso proporcione maior flexibilidade, também significa que cada componente tem suas próprias dependências e requer bibliotecas específicas a um idioma para acessar a plataforma subjacente e qualquer recurso compartilhados com o aplicativo pai. Além disso, implantar esses recursos como serviços separados pode acrescentar latência ao aplicativo. Gerenciar o código e as dependências para essas interfaces específicas a um idioma também pode agregar considerável complexidade, especialmente para hospedagem, implantação e gerenciamento.

## <a name="solution"></a>Solução

Posicione um conjunto consistente de tarefas no aplicativo principal, porém coloque-os dentro de seu próprio processo ou contêiner, fornecendo uma interface homogênea para serviços de plataforma entre linguagens. 

![](./_images/sidecar.png)

Um serviço sidecar não faz necessariamente parte do aplicativo, mas está conectado a ele. Ele irá sempre onde o aplicativo pai for. Sidecars são processos ou serviços de apoio que são implantados com o aplicativo principal. Em um moto, o sidecar é anexado a uma moto e cada moto pode ter seu próprio sidecar. Da mesma forma, um serviço sidecar compartilha a sina de seu aplicativo pai. Para cada instância do aplicativo, uma instância do sidecar é implantada e hospedada junto com ele. 

Vantagens de usar um padrão de sidecar:

- Um sidecar é independente do seu aplicativo principal em termos de ambiente de tempo de execução e de linguagem de programação, portanto você não precisa desenvolver um sidecar por linguagem. 

- O sidecar pode acessar os mesmos recursos que o aplicativo principal. Por exemplo, um sidecar pode monitorar os recursos do sistema usados tanto por ele mesmo quanto pelo aplicativo principal. 

- Por causa de sua proximidade do aplicativo principal, não há latência significativa ao se comunicar entre eles.

- Mesmo para aplicativos que não fornecem um mecanismo de extensibilidade, você pode usar um sidecar para estender a funcionalidade anexando-o como o próprio processo no mesmo host ou subcontêiner como o aplicativo principal.

O padrão de sidecar geralmente é usado com contêineres e conhecido como um contêiner de sidecar ou sidekick. 

## <a name="issues-and-considerations"></a>Problemas e considerações

- Considere a implantação e o formato de empacotamento que você usará para implantar serviços, processos ou contêineres. Contêineres são especialmente adequados para o padrão de sidecar.
- Ao projetar um serviço sidecar, escolha cuidadosamente o mecanismo de comunicação entre processos. Experimente usar tecnologias independente de linguagem ou estrutura, a menos que os requisitos de desempenho tornem isso inviável.
- Antes de colocar a funcionalidade em um sidecar, considere se ela funcionaria melhor como um serviço separado ou um daemon mais tradicional.
- Considere também se a funcionalidade poderia ser implementada como uma biblioteca ou usando um mecanismo de extensão tradicional. Bibliotecas específicas a um idioma podem ter um nível mais profundo de integração e menos sobrecarga de rede.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use esse padrão quando:

- O aplicativo principal usa um conjunto heterogêneo de linguagens e estruturas. Um componente localizado em um serviço secundários pode ser consumido por aplicativos escritos em linguagens diferentes usando estruturas diferentes.
- Um componente pertence a uma equipe remota ou a uma organização diferente.
- Um componente ou recurso deve ser localizado no mesmo host que o aplicativo
- Você precisa de um serviço que compartilha o ciclo de vida geral do aplicativo principal, mas pode ser atualizado independentemente.
- Você precisa de controle refinado sobre os limites do recurso para determinado recurso ou componente. Por exemplo, você talvez queira restringir a quantidade de memória que um componente específico usa. Você pode implantar o componente como um sidecar e gerenciar o uso de memória, independentemente do aplicativo principal.

Esse padrão pode não ser adequado:

- Quando a comunicação entre processos precisa ser otimizada. A comunicação entre um aplicativo pai e os serviços de sidecar incluem certa sobrecarga e latência notável nas chamadas. Essa pode não ser uma compensação aceitável para interfaces verborrágicas.
- Para pequenos aplicativos em que o custo do recurso da implantação de um serviço de sidecar para cada instância não compensa a vantagem de isolamento.
- Quando o serviço precisa ser dimensionado de forma diferente ou independente dos aplicativos principais. Nesse caso, é melhor implantar o recurso como um serviço separado.

## <a name="example"></a>Exemplo

O padrão sidecar se aplica a muitos cenários. Alguns exemplos comuns:

- API de Infraestrutura. A equipe de desenvolvimento de infraestrutura cria um serviço implantado ao lado de cada aplicativo, em vez de uma biblioteca de cliente específica a um idioma para acessar a infraestrutura. O serviço é carregado como um sidecar e fornece uma camada comum para serviços de infraestrutura, incluindo registro em log, dados de ambiente, repositório de configuração, descoberta, verificações de integridade e serviços de watchdog. O secundário também monitora o ambiente de host do aplicativo pai e o processo (ou contêiner) e registra as informações em um serviço centralizado.
- Gerenciar NGINX/HAProxy. Implante NGINX com um serviço sidecar que monitora o estado do ambiente, em seguida, atualize o arquivo de configuração NGINX e recicle o processo quando uma alteração de estado é necessária.
- Sidecar embaixador. Implantar um serviço [embaixador][ambassador] como um sidecar. O aplicativo chama por meio do embaixador, que lida com o registro em log de solicitações, roteamento, quebra de circuito e demais recursos relacionados à conectividade.
- Proxy de descarregamento. Coloque um proxy NGINX na frente de uma instância de serviço do node.js, para manipular a distribuição do conteúdo de arquivo estático para o serviço.


## <a name="related-guidance"></a>Diretrizes relacionadas

- [Padrão embaixador][ambassador]


[ambassador]: ./ambassador.md

