---
title: Detecção de fraude em tempo real
titleSuffix: Azure Example Scenarios
description: Detecte atividades fraudulentas em tempo real usando os Hubs de Eventos do Azure e o Stream Analytics.
author: alexbuckgit
ms.date: 07/05/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
social_image_url: /azure/architecture/example-scenario/data/media/architecture-fraud-detection.png
ms.openlocfilehash: b10838635cb592eb93d35ce745832c55a6daae8b
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245787"
---
# <a name="real-time-fraud-detection-on-azure"></a>Detecção de fraude em tempo real no Azure

Este cenário de exemplo é relevante para organizações que precisam analisar dados em tempo real para detectar transações fraudulentas ou outra atividade anômala.

Entre os possíveis usos estão a identificação de atividades fraudulentas em cartões de crédito ou de chamadas de celular. Os sistemas tradicionais de análise online podem levar horas para transformar e analisar os dados para identificar atividade anômala.

Usando serviços do Azure totalmente gerenciados, como os Hubs de Eventos e o Stream Analytics, as empresas podem eliminar a necessidade de gerenciar servidores individuais, reduzindo custos e aproveitando a experiência da Microsoft na ingestão de dados em escala de nuvem e na análise em tempo real. Este cenário aborda especificamente a detecção de atividade fraudulenta. Se você tiver outras necessidades de análise de dados, analise a lista de [serviços do Azure Analytics][product-category] disponíveis.

Este exemplo representa parte de uma estratégia e uma arquitetura de processamento de dados mais amplas. Outras opções para esse aspecto de uma arquitetura geral serão discutidas mais adiante neste artigo.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Outros casos de uso relevantes incluem:

- Detectar chamadas fraudulentas em cenários de telecomunicações.
- Identificar transações fraudulentas de cartão de crédito para instituições bancárias.
- Identificar compras fraudulentas em cenários de varejo ou comércio eletrônico.

## <a name="architecture"></a>Arquitetura

![Visão geral da arquitetura dos componentes do Azure de um cenário de detecção de fraudes em tempo real][architecture]

Este cenário cobre os componentes de back-end de um pipeline de análise em tempo real. O fluxo de dados deste cenário ocorre da seguinte forma:

1. Metadados de chamadas de celular são enviados do sistema de origem para uma instância dos Hubs de Eventos do Azure.
2. Um trabalho do Stream Analytics é iniciado e recebe dados da fonte do hub de eventos.
3. O trabalho do Stream Analytics executa uma consulta predefinida para transformar o fluxo de entrada e analisá-lo com base em um algoritmo de transações fraudulentas. Essa consulta usa uma janela em cascata para segmentar o fluxo em unidades temporais distintas.
4. O trabalho do Stream Analytics grava o fluxo transformado que representa chamadas fraudulentas para um coletor de saída do Armazenamentos de blobs do Azure.

### <a name="components"></a>Componentes

- Os [Hubs de Eventos do Azure][docs-event-hubs] são uma plataforma de streaming em tempo real e um serviço de ingestão de eventos capaz de receber e processar milhões de eventos por segundo. Os Hubs de Eventos podem processar e armazenar eventos, dados ou telemetria produzidos pelos dispositivos e software distribuídos. Neste cenário, os Hubs de Eventos recebem todos os metadados de chamadas telefônicas que serão analisadas em relação a atividades fraudulentas.
- O [Azure Stream Analytics][docs-stream-analytics] é um mecanismo de processamento de eventos que permite examinar grandes volumes de fluxo de dados de dispositivos e de outras fontes de dados. Ele também oferece suporte à extração de informações dos fluxos de dados para identificar padrões e relações. Esses padrões podem disparar outras ações downstream. Neste cenário, o Stream Analytics transforma o fluxo de entrada do Hub de Eventos para identificar chamadas fraudulentas.
- O [Armazenamento de Blobs](/azure/storage/blobs/storage-blobs-introduction) é usado neste cenário para armazenar os resultados do trabalho do Stream Analytics.

## <a name="considerations"></a>Considerações

### <a name="alternatives"></a>Alternativas

Muitas opções de tecnologia estão disponíveis para ingestão de mensagens em tempo real, armazenamento de dados, processamento de fluxo, armazenamento de dados analíticos, análise e relatórios. Para obter uma visão geral dessas opções, seus recursos e principais critérios de seleção, confira [Arquiteturas de Big Data: Processamento em tempo real](/azure/architecture/data-guide/technology-choices/real-time-ingestion) no Guia de Arquitetura de Dados do Azure.

Além disso, os algoritmos mais complexos para detecção de fraude podem ser produzidos por vários serviços de aprendizado de máquina no Azure. Para obter uma visão geral dessas opções, confira [Opções de tecnologia para aprendizado de máquina](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning) no [Guia de Arquitetura de Dados do Azure](../../data-guide/index.md).

### <a name="availability"></a>Disponibilidade

O Azure Monitor fornece interfaces de usuário unificadas para monitoramento entre os diferentes serviços do Azure. Para obter mais informações, confira [Monitoramento no Microsoft Azure](/azure/monitoring-and-diagnostics/monitoring-overview). Os Hubs de Eventos e o Stream Analytics estão integrados ao Azure Monitor.

Para outras considerações sobre disponibilidade, confira a [lista de verificação de disponibilidade][availability] no Azure Architecture Center.

### <a name="scalability"></a>Escalabilidade

Os componentes deste cenários foram concebidos para ingestão de grande escala e análise em tempo real paralela em massa. Os Hubs de Eventos do Azure são altamente escalonáveis, capazes de receber e processar milhões de eventos por segundo com baixa latência. Os Hubs de Eventos [escalam verticalmente](/azure/event-hubs/event-hubs-auto-inflate) o número de unidades de taxa de transferência automaticamente para atender às necessidades de uso. O Azure Stream Analytics é capaz de analisar grandes volumes de dados de streaming de várias fontes. Você pode escalar verticalmente o Stream Analytics, aumentando o número de [unidades de streaming](/azure/stream-analytics/stream-analytics-streaming-unit-consumption) alocado para executar o trabalho de streaming.

Para obter diretrizes gerais sobre como criar soluções escalonáveis, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.

### <a name="security"></a>Segurança

Os Hubs de Eventos do Azure protegem dados por meio de um [modelo de autenticação e segurança][docs-event-hubs-security-model] com base em uma combinação de tokens SAS (Assinatura de Acesso Compartilhado) e editores de eventos. Um editor de eventos define um ponto de extremidade virtual para um hub de eventos. O editor só pode ser usado para enviar mensagens a um hub de eventos. Não é possível receber mensagens de um editor.

Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].

### <a name="resiliency"></a>Resiliência

Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].

## <a name="deploy-the-scenario"></a>Implantar o cenário

Para implantar este cenário, você pode seguir este [tutorial passo a passo][tutorial] que demonstra como implantar manualmente cada componente do cenário. O tutorial também fornece um aplicativo de cliente .NET para gerar metadados de chamada telefônica de exemplo e enviar esses dados para uma instância de hub de eventos.

## <a name="pricing"></a>Preços

Para explorar o custo de executar esse cenário, todos os serviços são pré-configurados na calculadora de custos. Para ver como o preço seria alterado em seu caso de uso específico, altere as variáveis apropriadas de acordo com o volume de dados esperado.

Fornecemos três perfis de custo de exemplo com base na quantidade de tráfego que você espera receber:

- [Pequeno][small-pricing]: processa um milhão de eventos por meio de uma unidade de streaming padrão por mês.
- [Médio][medium-pricing]: processa 100 milhões de eventos por meio de cinco unidades de streaming padrão por mês.
- [Grande][large-pricing]: processa 999 milhões de eventos por meio de 20 unidades de streaming padrão por mês.

## <a name="related-resources"></a>Recursos relacionados

Cenários de detecção de fraude mais complexos podem se beneficiar de um modelo de aprendizado de máquina. Para cenários criados usando o Machine Learning Server, confira [Detecção de fraude usando o Machine Learning Server][r-server-fraud-detection]. Para outros modelos de solução usando o Machine Learning Server, confira [Cenários de ciência de dados e modelos de solução][docs-r-server-sample-solutions]. Para uma solução de exemplo usando o Azure Data Lake Analytics, confira [Usando Azure Data Lake e R para Detecção de Fraude][technet-fraud-detection].

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[tutorial]: /azure/stream-analytics/stream-analytics-real-time-fraud-detection
[small-pricing]: https://azure.com/e/74149ec312c049ccba79bfb3cfa67606
[medium-pricing]: https://azure.com/e/4fc94f7376de484d8ae67a6958cae60a
[large-pricing]: https://azure.com/e/7da8804396f9428a984578700003ba42
[architecture]: ./media/architecture-fraud-detection.png
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-event-hubs-security-model]: /azure/event-hubs/event-hubs-authentication-and-security-model-overview
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[docs-r-server-sample-solutions]: /machine-learning-server/r/sample-solutions
[r-server-fraud-detection]: https://microsoft.github.io/r-server-fraud-detection/
[technet-fraud-detection]: https://blogs.technet.microsoft.com/machinelearning/2017/06/28/using-azure-data-lake-and-r-for-fraud-detection/
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: ../../resiliency/index.md
[security]: /azure/security/
