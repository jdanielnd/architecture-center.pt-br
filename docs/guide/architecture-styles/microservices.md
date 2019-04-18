---
title: Estilo de arquitetura de microsserviços
titleSuffix: Azure Application Architecture Guide
description: Descreve os benefícios, desafios e melhores práticas para arquiteturas de microsserviços no Azure.
author: MikeWasson
ms.date: 11/13/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19, microservices
ms.openlocfilehash: 87a9dd31b6d935dd11a5a2a2950b6de11f337741
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639964"
---
# <a name="microservices-architecture-style"></a>Estilo de arquitetura de microsserviços

Uma arquitetura de microsserviços consiste em uma coleção de pequenos serviços autônomos. Cada serviço é independente e deve implementar uma única funcionalidade comercial.

![Diagrama lógico do estilo de arquitetura de microsserviços](./images/microservices-logical.svg)

Em alguns aspectos, microsserviços são a evolução natural das arquiteturas orientada a serviços (SOA), mas há diferenças entre microsserviços e SOA. Aqui estão algumas características que definem um microsserviço:

- Em uma arquitetura de microsserviços, os serviços são pequenos, independentes e fracamente acoplados.

- Cada serviço é uma base de código separado, que pode ser gerenciado por uma equipe de desenvolvimento pequena.

- Os serviços podem ser implantados de maneira independente. Uma equipe pode atualizar um serviço existente sem recompilar e reimplantar o aplicativo inteiro.

- Os serviços são responsáveis por manter seus próprios dados ou o estado externo. Isso é diferente do modelo tradicional, em que uma camada de dados separada lida com a persistência de dados.

- Os serviços comunicam-se entre si por meio de APIs bem definidas. Detalhes da implementação interna de cada serviço ficam ocultos de outros serviços.

- Os serviços não precisam compartilhar a mesma pilha de tecnologia, bibliotecas ou estruturas.

Além para os próprios serviços, alguns outros componentes aparecem em uma arquitetura de microsserviços típica:

**Gerenciamento**. O componente de gerenciamento é responsável por colocar serviços em nós, identificar falhas, rebalancear serviços entre nós e assim por diante.

**Descoberta de Serviço**. Mantém uma lista de serviços e em quais nós eles estão localizados. Habilita a pesquisa de serviço para localizar o ponto de extremidade para um serviço.

**Gateway de API**. O gateway de API é o ponto de entrada para os clientes. Os clientes não chamam serviços diretamente. Em vez disso, eles chamam o gateway de API, que encaminha a chamada para os serviços apropriados no back-end. O gateway de API pode agregar as respostas de vários serviços e retornar a resposta agregada.

As vantagens de usar um gateway de API incluem:

- Desacoplar os clientes dos serviços. Os serviços podem ter controle de versão ou ser refatorado sem necessidade de atualizar todos os clientes.

- Os serviços podem usar protocolos de mensagens que não sejam amigáveis à Web, como AMQP.

- O Gateway de API pode executar outras funções abrangentes, como autenticação, registro em log, terminação SSL e balanceamento de carga.

## <a name="when-to-use-this-architecture"></a>Quando usar essa arquitetura

Considere esse estilo de arquitetura para:

- Aplicativos grandes que precisem de uma alta velocidade de liberação.

- Aplicativos complexos que precisem ser altamente dimensionável.

- Aplicativos com domínios avançados ou muitos subdomínios.

- Uma organização que consista em pequenas equipes de desenvolvimento.

## <a name="benefits"></a>Benefícios

- **Implantações independentes**. Você pode atualizar um serviço sem reimplantar o aplicativo inteiro e, em seguida, reverter ou efetuar roll forward de uma atualização se algo der errado. Correções de bugs e liberações de recurso são mais fáceis de gerenciar e menos arriscadas.

- **Desenvolvimento independente**. Uma única equipe de desenvolvimento pode criar, testar e implantar um serviço. O resultado é inovação contínua e um ritmo mais rápido de liberação.

- **Equipes pequenas e focadas**. As equipes podem se concentrar em um serviço. O escopo menor de cada serviço torna a base de código mais fácil de entender e é mais fácil para novos membros da equipe fazerem a expansão.

- **Isolamento de falha**. Se um serviço falhar, ele não derrubará o aplicativo inteiro. No entanto, isso não significa que você obtém resiliência gratuitamente. Você ainda precisa seguir as melhores práticas de resiliência e padrões de design. Ver [desenvolvendo aplicativos do Azure confiáveis][resiliency-overview].

- **Pilhas de tecnologia mistas**. As equipes podem escolher a tecnologia mais adequada para seu serviço.

- **Dimensionamento granular**. Os serviços podem ser dimensionados de maneira independente. Ao mesmo tempo, a densidade mais alta de serviços por VM significa que os recursos de VM são totalmente usados. Usando restrições de posicionamento, um serviço pode corresponder a um perfil de VM (alta utilização da CPU e de memória e assim por diante).

## <a name="challenges"></a>Desafios

- **Complexidade**. Um aplicativo de microsserviços tem mais partes móveis que o aplicativo monolítico equivalente. Cada serviço é mais simples, mas o sistema como um todo é mais complexo.

- **Desenvolvimento e teste**. Desenvolver com relação a dependências de serviço exige uma abordagem diferente. As ferramentas existentes não são necessariamente projetadas para funcionar com dependências de serviço. Refatorar entre limites de serviços pode ser difícil. Também pode ser um desafio testar as dependências de serviço, especialmente quando o aplicativo está evoluindo rapidamente.

- **Falta de governança**. A abordagem descentralizada para compilar microsserviços tem vantagens, mas também pode causar problemas. Você pode acabar com muitos idiomas e estruturas diferentes que tornem difícil a manutenção do aplicativo. Pode ser útil estabelecer alguns padrões para todo o projeto, sem restringir excessivamente a flexibilidade das equipes. Isso se aplica especialmente a funcionalidades abrangentes como registro em log.

- **Latência e congestionamento de rede**. O uso de muitos serviços granulares pequenos pode resultar em mais comunicação entre serviços. Além disso, se a cadeia de dependências de serviço ficar muito longa (o serviço A chamada o B, que chama o C…) a latência adicional poderá se tornar um problema. Você precisará projetar APIs com cuidado. Evite APIs excessivamente prolixas, pense em formatos de serialização e procure locais para usar padrões de comunicação assíncrona.

- **Integridade de dados**. Cada microsserviço deve ser responsável pela própria persistência de dados. Assim, a consistência dos dados pode ser um desafio. Adote consistência eventual quando possível.

- **Gerenciamento**. Ter êxito com microsserviços requer uma cultura DevOps madura. Registro em log correlacionado entre serviços pode ser desafiador. Normalmente, o registro em log deve correlacionar várias chamadas de serviço para uma operação de um único usuário.

- **Controle de versão**. As atualizações de um serviço não devem interromper os serviços que dependerem delas. Vários serviços podem ser atualizados a qualquer momento, portanto, sem design cuidadoso, você pode ter problemas com compatibilidade com versões anteriores ou futuras.

- **Conjunto de qualificações**. Os microsserviços são sistemas altamente distribuídos. Avalie cuidadosamente se a equipe tem as habilidades e a experiência para ser bem-sucedida.

## <a name="best-practices"></a>Práticas recomendadas

- Modele os serviços em torno de domínio da empresa.

- Descentralize tudo. Equipes individuais são responsáveis por projetar e criar serviços. Evite compartilhar esquemas de dados ou códigos.

- O armazenamento de dados deve ser privado para o serviço que é o proprietário dos dados. Use o melhor armazenamento para cada serviço e tipo de dados.

- Os serviços comunicam-se por meio de APIs bem projetadas. Evite o vazamento de detalhes da implementação. As APIs devem modelar o domínio, não a implementação interna do serviço.

- Evite acoplamento entre serviços. Causas de acoplamento incluem protocolos de comunicação rígidos e esquemas de banco de dados compartilhados.

- Descarregue preocupações abrangentes, como autenticação e terminação SSL, para o gateway.

- Mantenha o conhecimento de domínio fora do gateway. O gateway deve tratar e rotear solicitações de cliente sem qualquer conhecimento das regras de negócios ou da lógica do domínio. Caso contrário, o gateway se tornará uma dependência e poderá causar um acoplamento entre serviços.

- Os serviços devem ter um acoplamento flexível e alta coesão funcional. Funções que provavelmente mudarão juntas devem ser empacotadas e implantadas juntas. Se residirem em serviços separados, esses serviços acabarão sendo fortemente acoplados, porque uma alteração em um serviço exigirá atualizar outro. Uma comunicação excessivamente prolixa entre dois serviços pode ser um sintoma de acoplamento forte e coesão baixa.

- Isole falhas. Use estratégias de resiliência para impedir que falhas em um serviço distribuam-se em cascata. Ver [padrões de resiliência] [ resiliency-patterns] e [Projetando aplicativos confiáveis][resiliency-overview].

## <a name="next-steps"></a>Próximas etapas

Para obter orientações detalhadas sobre como criar uma arquitetura de microsserviços no Azure, consulte [Projetar, criar e operar microsserviços no Azure](../../microservices/index.md).

<!-- links -->

[resiliency-overview]: ../../reliability/index.md
[resiliency-patterns]: ../../patterns/category/resiliency.md
