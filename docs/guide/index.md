---
layout: LandingPage
ms.topic: landing-page
ms.service: architecture-center
ms.subservice: reference-architecture
ms.date: 08/30/2018
---

# <a name="azure-application-architecture-guide"></a>Guia de Arquitetura do Aplicativo do Azure

Este guia apresenta uma abordagem estruturada para a criação de aplicativos do Azure que são escalonáveis, flexíveis e altamente disponíveis. Ele se baseia nas práticas comprovadas que aprendemos nos contratos com clientes.

## <a name="introduction"></a>Introdução

A nuvem está mudando a maneira como os aplicativos são criados. Em vez de monolitos, aplicativos são decompostos em serviços descentralizados menores. Esses serviços se comunicam por meio de APIs, ou usando o serviço de mensagens ou eventos assíncronos. Os aplicativos são dimensionadaos horizontalmente, adicionando novas instâncias de acordo com a demanda.

Essas tendências trazem novos desafios. O estado do aplicativo é distribuído. As operações são feitas em paralelo e de maneira assíncrona. O sistema como um todo deve ser resiliente quando ocorrem falhas. As implantações devem ser automatizadas e previsíveis. Monitoramento e telemetria são essenciais para obter informações sobre o sistema. O Guia de Arquitetura do Aplicativo do Azure foi criado para ajudá-lo a entender melhor essas mudanças.

<!-- markdownlint-disable MD033 -->

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

<!-- markdownlint-enable MD033 -->

Este guia destina-se a arquitetos de aplicativos, desenvolvedores e equipes de operações. Não é um guia de instruções para usar serviços individuais do Azure. Depois de ler este guia, você entenderá os padrões de arquitetura e práticas recomendadas para aplicar ao criar a plataforma de nuvem do Azure. Também pode baixar uma [versão em livro eletrônico do guia][ebook].

## <a name="how-this-guide-is-structured"></a>Como este guia é estruturado

O Guia de Arquitetura do Aplicativo do Azure é organizado como uma série de etapas de arquitetura e design para implementação. Para cada etapa, há diretrizes de suporte que ajudarão você com o design de arquitetura de seu aplicativo.

### <a name="architecture-styles"></a>Estilos de arquitetura

O primeiro ponto de decisão é o mais importante. Que tipo de arquitetura está compilando? Pode ser uma arquitetura de microsserviços, um aplicativo de N camadas mais tradicional ou uma solução de big data. Identificamos vários estilos de arquitetura diferentes. Há benefícios e desafios para cada um.

Saiba mais:

- [Estilos de arquitetura](./architecture-styles/index.md)

### <a name="technology-choices"></a>Opções de tecnologia

Duas opções de tecnologia devem ser decididas logo no início, porque elas afetam toda a sua arquitetura. São a escolha do serviço de computação e armazenamentos de dados. *Computação* se refere ao modelo de hospedagem dos recursos de computação em que seu aplicativo é executado. *Armazenamentos de dados* incluem bancos de dados, mas também o armazenamento das filas de mensagens, caches, logs e tudo que um aplicativo pode manter no armazenamento.

Saiba mais:

- [Escolhendo um serviço de computação](./technology-choices/compute-overview.md)
- [Escolhendo um armazenamento de dados](./technology-choices/data-store-overview.md)

### <a name="design-principles"></a>Princípios de design

Identificamos dez princípios de design de alto nível que tornarão seu aplicativo mais escalonável, flexível e gerenciável. Esses princípios de design se aplicam a todos os estilos arquitetura. Durante o processo de design, mantenha esses dez princípios de design de alto nível. Em seguida, considere o conjunto de práticas recomendadas para aspectos específicos da arquitetura, como o dimensionamento automático, cache, particionamento de dados, design da API e outros.

Saiba mais:

- [Princípios de design](./design-principles/index.md)

### <a name="quality-pillars"></a>Pilares da qualidade

Um aplicativo em nuvem bem-sucedido se concentrará em cinco pilares de qualidade de software: Escalabilidade, disponibilidade, resiliência, gerenciamento e segurança. Use nossas listas de verificação do design para avaliar a arquitetura de acordo com os pilares da qualidade.

- [Pilares de qualidade](./pillars.md)

[ebook]: https://azure.microsoft.com/campaigns/cloud-application-architecture-guide/
