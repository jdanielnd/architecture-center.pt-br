---
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 530844a0d3b1256cec807e7bad509a40dca304f6
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
---
# <a name="azure-application-architecture-guide"></a>Guia de Arquitetura do Aplicativo do Azure

Este guia apresenta uma abordagem estruturada para a criação de aplicativos do Azure que são escalonáveis, flexíveis e altamente disponíveis. Ele se baseia nas práticas comprovadas que aprendemos nos contratos com clientes.

<img src="./images/guide-steps.svg" style="max-width:800px;"/>

## <a name="introduction"></a>Introdução

A nuvem está mudando a maneira como os aplicativos são criados. Em vez de monolitos, aplicativos são decompostos em serviços descentralizados menores. Esses serviços se comunicam por meio de APIs, ou usando o serviço de mensagens ou eventos assíncronos. Os aplicativos são dimensionadaos horizontalmente, adicionando novas instâncias de acordo com a demanda. 

Essas tendências trazem novos desafios. O estado do aplicativo é distribuído. As operações são feitas em paralelo e de maneira assíncrona. O sistema como um todo deve ser resiliente quando ocorrem falhas. As implantações devem ser automatizadas e previsíveis. Monitoramento e telemetria são essenciais para obter informações sobre o sistema. O Guia de Arquitetura do Aplicativo do Azure foi criado para ajudá-lo a entender melhor essas mudanças. 

<table>
<thead>
    <tr><th>Tradicional fornecido no local</th><th>Nuvem moderna</th></tr>
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
Servidores floco de neve</td>
<td>
Decomposto, decentralizado<br/>
Design para dimensionamento elástico<br/>
Persistência poliglota<br/>
Consistência eventual<br/>
Processamento paralelo e assíncrono<br/>
Design para falhas (MTTR)<br/>
Atualizações pequenas frequentes<br/>
Autogerenciamento automatizado<br/>
Infraestrutura imutável<br/>
</td>
</tbody>
</table>

Este guia destina-se a arquitetos de aplicativos, desenvolvedores e equipes de operações. Não é um guia de instruções para usar serviços individuais do Azure. Depois de ler este guia, você entenderá os padrões de arquitetura e práticas recomendadas para aplicar ao criar a plataforma de nuvem do Azure. Também pode baixar uma [versão em livro eletrônico do guia][ebook].

## <a name="how-this-guide-is-structured"></a>Como este guia é estruturado

O Guia de Arquitetura do Aplicativo do Azure é organizado como uma série de etapas de arquitetura e design para implementação. Para cada etapa, há diretrizes de suporte que ajudarão você com o design de arquitetura de seu aplicativo.

**[Estilos de arquitetura][arch-styles]**. O primeiro ponto de decisão é o mais importante. Que tipo de arquitetura está compilando? Pode ser uma arquitetura de microsserviços, um aplicativo de N camadas mais tradicional ou uma solução de big data. Nós identificamos sete estilos de arquitetura distintos. Há benefícios e desafios para cada um.

> &#10148; [Arquiteturas de Referência do Azure][ref-archs] mostra as implantações no Azure, junto com as considerações sobre escalabilidade, disponibilidade, capacidade de gerenciamento e segurança. A maioria inclui também modelos implantáveis do Resource Manager.

**[Opções de Tecnologia][technology-choices]**. Duas opções de tecnologia devem ser decididas logo no início, porque elas afetam toda a sua arquitetura. Essas são as opções de computação e de tecnologias armazenamento. O termo *computação* refere-se ao modelo de hospedagem para os recursos de computação em que seus aplicativos são executados. O armazenamento inclui bancos de dados, assim como o armazenamento de filas de mensagens, caches, dados da IoT, dados de log não estruturados e tudo mais que um aplicativo possa encaminhar para o armazenamento. 

> &#10148; [Opções de computação][compute-options] e [opções de armazenamento][storage-options] fornecem critérios de comparação detalhados para a seleção de serviços de computação e armazenamento.

**[Princípios de design][design-principles]**. Durante o processo de design, mantenha esses dez princípios de design de alto nível. 

> &#10148; [Práticas recomendadas][best-practices] fornecem diretrizes específicas em áreas como o dimensionamento automático, cache, particionamento de dados, design de API e outras.   

**[Pilares][pillars]**. Um aplicativo em nuvem bem-sucedido deverá focar nestes cinco pilares de qualidade do software: escalabilidade, disponibilidade, resiliência, gerenciamento e segurança. 

> &#10148; Use nossas [listas de verificação para a análise do design][checklists] para avaliar o design de acordo com esses pilares de qualidade. 

**[Padrões de Design na Nuvem][patterns]**. Esses padrões de design são úteis para a criação de aplicativos confiáveis, escalonáveis e seguros no Azure. Cada padrão descreve um problema, um padrão que resolva o problema e um exemplo baseado no Azure.

> &#10148; Exibir o [Catálogo completo de padrões de design na nuvem](../patterns/index.md).


[arch-styles]: ./architecture-styles/index.md
[best-practices]: ../best-practices/index.md
[checklists]: ../checklist/index.md
[compute-options]: ./technology-choices/compute-comparison.md
[design-principles]: ./design-principles/index.md
[ebook]: https://azure.microsoft.com/campaigns/cloud-application-architecture-guide/
[patterns]: ../patterns/index.md?toc=/azure/architecture/guide/toc.json
[pillars]: ./pillars.md
[ref-archs]: ../reference-architectures/index.md
[storage-options]: ./technology-choices/data-store-comparison.md
[technology-choices]: ./technology-choices/index.md

