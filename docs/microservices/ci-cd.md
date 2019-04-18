---
title: CI/CD para microsserviços
description: Integração contínua e a entrega contínua para microsserviços.
author: MikeWasson
ms.date: 03/27/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: c52ff3d0a330f564e5f7e9b0b07f0ba84c328c8b
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639765"
---
# <a name="cicd-for-microservices-architectures"></a>CI/CD para arquiteturas de microsserviços

Ciclos de lançamento mais rápidos são uma das principais vantagens de arquiteturas de microsserviços. Mas, sem um bom processo de CI/CD, você não obterá a agilidade que os microsserviços prometem. Este artigo descreve os desafios e recomenda algumas abordagens para o problema.

## <a name="what-is-cicd"></a>O que é o CI/CD?

Quando falamos sobre CI/CD, realmente falamos vários processos relacionados: Integração contínua, entrega contínua e implantação contínua.

- **Integração contínua**. Alterações de código são frequentemente mescladas à ramificação principal. Automated build e processos de teste Certifique-se de que o código na ramificação principal é sempre qualidade de produção.

- **Entrega contínua**. Nenhuma alteração de código que passam pelo processo de CI é publicadas automaticamente em um ambiente de produção. A implantação no ambiente de produção dinâmico pode exigir aprovação manual, mas caso contrário, é automatizada. A meta é que o seu código esteja sempre *pronto* para implantação na produção.

- **Implantação contínua**. Alterações de código que passe as duas etapas anteriores são automaticamente implantadas *em produção*.

Aqui estão algumas metas de um processo de CI/CD robusto para uma arquitetura de microsserviços:

- Cada equipe pode criar e implantar os serviços que ela possui independentemente, sem afetar ou interromper outras equipes.

- Antes de uma nova versão de um serviço ser implantada na produção, ela é implantada em ambientes de desenvolvimento/teste/garantia de qualidade para validação. Há restrições de qualidade impostas em cada estágio.

- Uma nova versão de um serviço pode ser implantada lado a lado com a versão anterior.

- Há políticas de controle de acesso suficientes em vigor.

- Para cargas de trabalho em contêineres, você pode confiar as imagens de contêiner que são implantadas em produção.

## <a name="why-a-robust-cicd-pipeline-matters"></a>Por que é importante de um pipeline de CI/CD robusto

Em um aplicativo monolítico tradicional, há um único pipeline de build cuja saída é o executável do aplicativo. Todo o trabalho de desenvolvimento alimenta este pipeline. Se for encontrado um bug de alta prioridade, uma correção deverá ser integrada, testada e publicada, o que poderá atrasar o lançamento dos novos recursos. Você pode atenuar esses problemas com módulos bem fatorados e usando ramificações recursos para minimizar o impacto de alterações de código. Mas conforme o aplicativo se torna mais complexo e mais recursos são adicionados, o processo de liberação para um monolito tende a se tornar mais frágil e suscetível a interrupção.

Seguindo a filosofia de microsserviços, nunca deverá haver uma versão de treinamento longa em que todas as equipes precisam se mobilizar. A equipe que cria o serviço "A" pode lançar uma atualização a qualquer momento, sem esperar que as alterações no serviço "B" sejam mescladas, testadas e implantadas.

![Diagrama de um monólito de CI/CD](./images/cicd-monolith.png)

Para obter uma versão de alta velocidade, o pipeline de lançamento deve ser automatizado e altamente confiável, para minimizar o risco. Se você liberar para produção diariamente ou várias vezes ao dia, as regressões ou interrupções de serviço devem ser muito raras. Ao mesmo tempo, se uma atualização inválida é implantada, você deve ter uma maneira confiável de reverter ou efetuar roll forward rapidamente para uma versão anterior de um serviço.

## <a name="challenges"></a>Desafios

- **Várias bases de código pequenas independentes**. Cada equipe é responsável por criar seu próprio serviço, com seu próprio pipeline de build. Em algumas organizações, as equipes podem usar repositórios de código separados. Repositórios separados podem levar a uma situação em que o conhecimento de como criar o sistema é disseminado entre equipes e ninguém na organização sabe como implantar o aplicativo inteiro. Por exemplo, o que acontecerá em um cenário de recuperação de desastre, se você precisar implantar rapidamente para um novo cluster?

    **Atenuação**: Tem um pipeline automatizado e unificado para compilar e implantar os serviços, para que esse conhecimento não estiver "oculto" dentro de cada equipe.

- **Múltiplas linguagens de programação e estruturas**. Com cada equipe usando seu próprio conjunto de tecnologias, pode ser difícil criar um processo de build único que funcione em toda a organização. O processo de build deve ser flexível o suficiente para que cada equipe possa adaptá-lo para sua linguagem de programação ou estrutura de preferência.

    **Atenuação**: Colocar em contêiner o processo de compilação para cada serviço. Dessa forma, o sistema de compilação apenas precisa ser capaz de executar os contêineres.

- **Integração e teste de carga**. Com as equipes de liberação de atualizações em seu próprio ritmo, pode ser difícil projetar testes de ponta a ponta robustos, especialmente quando os serviços têm dependências em outros serviços. Além disso, a execução de um cluster de produção completo pode ser cara, portanto, é improvável que todas as equipes executará seu próprio cluster completo em escalas de produção, apenas para teste.

- **Gerenciamento de versão**. Cada equipe deve ser capaz de implantar uma atualização para a produção. Isso não significa que cada membro da equipe tem permissões para fazer isso. Mas ter uma função de Gerenciador de Versão centralizada pode reduzir a velocidade das implantações.

    **Atenuação**: Quanto mais o processo de CI/CD for automatizado e confiável, menos deverá haver necessidade de uma autoridade central. Dito isso, você pode ter diferentes políticas para liberar atualizações dos principais recursos e correções de bugs secundários. Ser descentralizado não significa zero governança.

- **Atualizações de serviço**. Quando você atualizar um serviço para uma nova versão, ele não deverá interromper outros serviços que dependem dele.

    **Atenuação**: Use técnicas de implantação, como lançamento canário ou azul verde para alterações sem interrupções. Para alterações da API, implante a nova versão lado a lado com a versão anterior. Dessa forma, os serviços que consomem a API anterior seja atualizados e testados para a nova API. Ver [serviços de atualização](#updating-services), abaixo.

## <a name="monorepo-vs-multi-repo"></a>Vários repositórios do Monorepo vs

Antes de criar um fluxo de trabalho de CI/CD, você precisa saber como a base de código será estruturada e gerenciada.

- As equipes funcionam em repositórios separados ou em um monorepo (único repositório)?
- Qual é sua estratégia de ramificação?
- Quem pode efetuar push do código para a produção? Existe uma função de gerente de versão?

A abordagem de repositório único tem tido mais aceitação, mas há vantagens e desvantagens em ambas.

| &nbsp; | Repositório único | Vários repositórios |
|--------|----------|----------------|
| **Vantagens** | Compartilhamento de código<br/>Maior facilidade em padronizar o código e as ferramentas<br/>Maior facilidade em refatorar o código<br/>Detectabilidade – modo de exibição único do código<br/> | Propriedade clara por equipe<br/>Possivelmente menos conflitos de mesclagem<br/>Ajuda a impor o desacoplamento de microsserviços |
| **Desafios** | As alterações no código compartilhado podem afetar vários microsserviços<br/>Maior potencial de conflitos de mesclagem<br/>As ferramentas devem ser dimensionadas para uma base de código grande<br/>Controle de acesso<br/>Processo de implantação mais complexo | Mais difícil de compartilhar o código<br/>Mais difícil de impor padrões de codificação<br/>Gerenciamento de dependências<br/>Base de código difusa, baixa detectabilidade<br/>Falta de infraestrutura compartilhada

## <a name="updating-services"></a>Atualizando serviços

Há várias estratégias para atualizar um serviço que já está em produção. Aqui, abordamos três opções comuns: Atualização sem interrupção, implantação "blue-green" e versão canário.

### <a name="rolling-updates"></a>Atualizações sem interrupção

Em uma atualização sem interrupção, você implanta novas instâncias de um serviço e as novas instâncias começam a receber solicitações imediatamente. À medida que as novas instâncias chegam, as anteriores são removidas.

**Exemplo:** No Kubernetes, atualizações sem interrupção são o comportamento padrão quando você atualiza a especificação de pod para uma implantação. O controlador de implantação cria um novo ReplicaSet para os pods atualizados. Em seguida, ele expande o novo ReplicaSet e reduz simultaneamente o antigo, para manter a contagem de réplicas desejada. Ela não exclui os pods antigos até que os novos estejam prontos. Kubernetes mantém um histórico da atualização, portanto, você pode reverter uma atualização se necessário.

Um desafio de reverter atualizações é que durante o processo de atualização uma mistura das versões antiga e nova estão em execução e recebendo tráfego. Durante esse período, qualquer solicitação poderia ser roteada para qualquer uma das duas versões.

Para alterações da API, uma prática recomendada é dar suporte a ambas as versões lado a lado, até que todos os clientes da versão anterior são atualizados. Ver [controle de versão de API](./design/api-design.md#api-versioning).

### <a name="blue-green-deployment"></a>Implantação "blue-green"

Em uma implantação "blue-green", você deve implantar a nova versão juntamente com a versão anterior. Depois de validar a nova versão, você pode mudar todo o tráfego da versão anterior para a nova versão, de uma só vez. Após a troca, você deve monitorar o aplicativo para quaisquer problemas. Se algo der errado, você poderá retornar à versão antiga. Se nenhum problema ocorrer, você poderá excluir a versão antiga.

Com um aplicativo monolítico ou de N camadas mais tradicional, a implantação "blue-green" geralmente significa provisionar dois ambientes idênticos. Você implantaria a nova versão em um ambiente de preparo e então redirecionaria o tráfego de cliente para o ambiente de preparo &mdash; por exemplo, alternando VIPs. Em uma arquitetura de microsserviços, as atualizações ocorrem no nível do microsserviço, portanto, você normalmente seria implantar a atualização no mesmo ambiente e usar um mecanismo de descoberta de serviço para trocar.

**Exemplo**. No Kubernetes, você não precisa provisionar um cluster separado para fazer implantações "blue-green". Em vez disso, você pode tirar proveito de seletores. Crie um novo recurso de implantação com uma nova especificação de pod e um conjunto diferente de rótulos. Crie essa implantação sem excluir a implantação anterior nem modificar o serviço que aponta para ela. Quando os novo pods estiverem em execução, você poderá atualizar o seletor do serviço para corresponder à nova implantação.

Uma desvantagem de implantação "blue-green" é que durante a atualização, você está executando dobro pods para o serviço (atual e a próxima). Se os pods exigirem muitos recursos de CPU ou de memória, talvez você precisará expandir o cluster temporariamente para dar conta do consumo de recursos.

### <a name="canary-release"></a>Versão canário

Em uma versão canário, você distribui uma versão atualizada para um número pequeno de clientes. Em seguida, você monitora o comportamento do novo serviço antes de implantá-lo em todos os clientes. Isso lhe permite fazer uma distribuição lenta de forma controlada, observar dados reais e identificar problemas antes que todos os clientes sejam afetados.

Uma versão canário é mais complexa de gerenciar do que a atualização sem interrupção ou "blue-green", porque você deve rotear solicitações dinamicamente para diferentes versões do serviço.

**Exemplo**. No Kubernetes, você pode configurar um serviço para abranger dois conjuntos de réplicas (um para cada versão) e ajustar as contagens de réplicas manualmente. No entanto, essa abordagem tem uma granularidade bastante alta devido ao modo como o Kubernetes balanceia a carga entre os pods. Por exemplo, se você tiver um total de 10 réplicas, você só poderá realizar deslocamentos tráfego em incrementos de 10%. Se você estiver usando uma malha de serviço, poderá usar as regras de roteamento de malha do serviço para implementar uma estratégia de versão canário mais sofisticada.

## <a name="next-steps"></a>Próximas etapas

Aprenda as práticas específicas de CI/CD para microsserviços em execução no Kubernetes.

- [CI/CD para microsserviços no Kubernetes](./ci-cd-kubernetes.md)