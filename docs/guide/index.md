---
layout: LandingPage
ms.openlocfilehash: 00abbfdeac89a9006517195bd4bbc514d587fe74
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="azure-application-architecture-guide"></a>Guia de Arquitetura do Aplicativo do Azure

Este guia apresenta uma abordagem estruturada para a criação de aplicativos do Azure que são escalonáveis, flexíveis e altamente disponíveis. Ele se baseia nas práticas comprovadas aprendemos de contratos com clientes.

<img src="./images/guide-steps.svg" style="max-width:800px;"/>

## <a name="introduction"></a>Introdução

A nuvem está mudando a maneira como os aplicativos são criados. Em vez de monolitos, aplicativos são decompostos em serviços descentralizados menores. Esses serviços se comunicam por meio de APIs, ou usando o serviço de mensagens assíncrono ou eventos. Aplicativos dimensionada horizontalmente, adicionando novas instâncias como exige. 

Essas tendências trazem novos desafios. Estado do aplicativo é distribuído. Operações são feitas em paralelo e de maneira assíncrona. O sistema como um todo deve ser resiliente quando ocorrem falhas. Implantações devem ser automatizada e previsível. Monitoramento e telemetria são essenciais para obter informações sobre o sistema. O Guia de Arquitetura de Aplicativo do Azure foi projetado para ajudá-lo a navegar nessas alterações. 

<table>
<thead>
    <tr><th>Local tradicional</th><th>Nuvem moderna</th></tr>
</thead>
<tbody>
<tr><td>Monolítico, centralizado<br/>
Design para escalabilidade previsível<br/>
Banco de dados relacional<br/>
Consistência forte<br/>
Processamento serial e sincronizado<br/>
Design para evitar falhas (MTBF)<br/>
Grandes atualizações ocasionais<br/>
Gerenciamento manual<br/>
Servidores de floco de neve</td>
<td>
Decomposto, decentralizado<br/>
Design para dimensionamento elástico<br/>
Persistência Polyglot (combinação de tecnologias de armazenamento)<br/>
Consistência eventual<br/>
Processamento paralelo e assíncrono<br/>
Design de falha (MTTR)<br/>
Atualizações pequenas frequentes<br/>
Autogerenciamento automatizado<br/>
Infraestrutura imutável<br/>
</td>
</tbody>
</table>

Este guia destina-se a arquitetos de aplicativos, os desenvolvedores e equipes de operações. Não é um guia de instruções para usar serviços individuais do Azure. Depois de ler este guia, você entenderá os padrões de arquitetura e práticas recomendadas para aplicar ao criar a plataforma de nuvem do Azure.

## <a name="how-this-guide-is-structured"></a>Como este guia é estruturado

O Guia de Arquitetura de Aplicativo do Azure é organizado como uma série de etapas de arquitetura e design para implementação. Para cada etapa, há diretrizes de suporte que ajudarão você com o design de arquitetura de seu aplicativo.

**[Estilos de arquitetura][arch-styles]**. O primeiro ponto de decisão é mais importantes. Que tipo de arquitetura está compilando? Ele pode ser uma arquitetura de microsserviços, um aplicativo de N camadas mais tradicional ou uma solução big data. Nós identificamos sete estilos de arquitetura distintos. Há benefícios e desafios para cada um.

> &#10148; [Arquiteturas de Referência do Azure][ref-archs] mostra as implantações no Azure, junto com as considerações sobre escalabilidade, disponibilidade, capacidade de gerenciamento e segurança. Mais também inclui modelos implantáveis do Resource Manager.

**[Opções de Tecnologia][technology-choices]**. Duas opções de tecnologia devem ser decididas logo no início, porque elas afetam toda a sua arquitetura. Esses são a escolha das tecnologias de computação e armazenamento. O termo *computação* refere-se ao modelo de hospedagem para os recursos de computação em que seus aplicativos são executados. O armazenamento inclui bancos de dados, mas também o armazenamento de filas de mensagens, caches, dados IoT, dados de log não estruturados e qualquer outra coisa que um aplicativo possa persistir para armazenamento. 

> &#10148; [Opções de computação][compute-options] e [opções de armazenamento][storage-options] fornecer critérios de comparação detalhada para selecionar serviços de computação e armazenamento.

**[Princípios de design][design-principles]**. Durante o processo de design, tenha em mente essas dez princípios de design de alto nível. 

> &#10148; [Práticas recomendadas][best-practices] artigos fornecer diretrizes específicas em áreas como o dimensionamento automático, cache, particionamento de dados, design de API e outros.   

**[Pilares][pillars]**. Um aplicativo em nuvem com êxito se concentrará nesses cinco colunas de qualidade de software: escalabilidade, disponibilidade, resiliência, gerenciamento e segurança. 

> &#10148; Use nossas [listas de verificação de revisão de design][checklists] para examinar o design de acordo com essas colunas de qualidade. 

**[Padrões de Design na Nuvem][patterns]**. Esses padrões de design são úteis para a criação confiável e escalonável e aplicativos seguros no Azure. Cada padrão descreve um problema, um padrão que resolva o problema e um exemplo baseado no Azure.

> &#10148; Exibir completo [catálogo de padrões de design de nuvem](../patterns/index.md).


[arch-styles]: ./architecture-styles/index.md
[best-practices]: ../best-practices/index.md
[checklists]: ../checklist/index.md
[compute-options]: ./technology-choices/compute-comparison.md
[design-principles]: ./design-principles/index.md
[patterns]: ../patterns/index.md?toc=/azure/architecture/guide/toc.json
[pillars]: ./pillars.md
[ref-archs]: ../reference-architectures/index.md
[storage-options]: ./technology-choices/data-store-comparison.md
[technology-choices]: ./technology-choices/index.md

