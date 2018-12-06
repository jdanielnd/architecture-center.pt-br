---
title: Escolhendo uma tecnologia de ingestão de mensagens em tempo real
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 64d6fca0a8ffac45f605e90a11cd2b3e53db287f
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/05/2018
ms.locfileid: "52901605"
---
# <a name="choosing-a-real-time-message-ingestion-technology-in-azure"></a>Escolhendo uma tecnologia de ingestão de mensagens em tempo real no Azure

O processamento em tempo real lida com fluxos de dados que são capturados em tempo real e processados com latência mínima. Muitas soluções de processamento em tempo real precisam de um repositório de ingestão de mensagens para atuar como buffer de mensagens e dar suporte a processamento de expansão, entrega confiável e outras semânticas de enfileiramento de mensagem. 

## <a name="what-are-your-options-for-real-time-message-ingestion"></a>Quais são as opções disponíveis para ingestão de mensagens em tempo real?

- [Hubs de eventos do Azure](/azure/event-hubs/)
- [Hub IoT do Azure](/azure/iot-hub/)
- [Kafka no HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started)

## <a name="azure-event-hubs"></a>Hubs de eventos do Azure

Os [Hubs de Eventos do Azure](/azure/event-hubs/) são um plataforma de streaming de dados altamente escalonável e um serviço de ingestão de dados capaz de receber e processar milhões de eventos por segundo. Os Hubs de Eventos podem processar e armazenar eventos, dados ou telemetria produzidos pelos dispositivos e software distribuídos. Os dados enviados para um Hub de Eventos podem ser transformados e armazenados usando qualquer provedor de análise em tempo real ou adaptadores de envio em lote/armazenamento. Os Hubs de Eventos oferecem funcionalidades de publicação/assinatura com baixa latência em grande escala, o que é apropriado para cenários de Big Data.

## <a name="azure-iot-hub"></a>Hub IoT do Azure

O [Hub IoT do Azure](/azure/iot-hub/) é um serviço gerenciado que permite uma comunicação bidirecional confiável e segura entre milhões de dispositivos IoT e um back-end baseado em nuvem.

O recurso de Hub IoT inclui:

* Várias opções de comunicação do dispositivo para a nuvem e da nuvem para o dispositivo. Essas opções incluem mensagens unidirecionais, transferência de arquivos e métodos de solicitação-resposta.
* Roteamento de mensagens para outros serviços do Azure.
* Repositório consultável de metadados de dispositivo e informações de estado sincronizado.
* Comunicação segura e controle de acesso usando chaves de segurança por dispositivo ou certificados X.509.
* Monitoramento de conectividade do dispositivo e eventos de gerenciamento de identidade do dispositivo.

Em termos de ingestão de mensagens, o Hub IoT é semelhante aos Hubs de Eventos. No entanto, ele foi especificamente projetado para o gerenciamento de conectividade do dispositivo IoT, não apenas para a ingestão de mensagens. Para obter mais informações, consulte [Comparação entre o Hub IoT do Azure e os Hubs de Eventos do Azure](/azure/iot-hub/iot-hub-compare-event-hubs). 

## <a name="kafka-on-hdinsight"></a>Kafka no HDInsight

O [Apache Kafka](https://kafka.apache.org/) é uma plataforma de streaming distribuída de software livre que pode ser usada para criar pipelines de dados e aplicativos de streaming em tempo real. O Kafka também fornece funcionalidade de agente de mensagem semelhante a uma fila de mensagens, onde você pode publicar e assinar os fluxos de dados nomeados. É horizontalmente escalonável, tolerante a falhas e extremamente rápido. O [Kafka no HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started) fornece um Kafka como um serviço gerenciado, altamente escalonável e altamente disponível no Azure. 

Alguns casos de uso comuns do Kafka são:

* **Mensagens**. Como ele dá suporte ao padrão de mensagem de publicação/assinatura, o Kafka geralmente é usado como um agente de mensagem.
* **Acompanhamento de atividade**. Como o Kafka fornece log na ordem de registros, ele pode ser usado para acompanhar e recriar atividades, como ações de usuário em um site da Web.
* **Agregação**. Usando o processamento de fluxo, você pode agregar informações de fluxos diferentes para combinar e centralizar as informações em dados operacionais.
* **Transformação**. Usando o processamento de fluxo, você pode combinar e enriquecer dados de vários tópicos de entrada em um ou mais tópicos de saída.

## <a name="key-selection-criteria"></a>Principais critérios de seleção

Para restringir as opções, comece respondendo a estas perguntas:

- Você precisa de uma comunicação bidirecional entre seus dispositivos IoT e o Azure? Nesse caso, escolha o Hub IoT.

- Você precisa gerenciar o acesso de dispositivos individuais e ter a capacidade de revogar o acesso a um dispositivo específico? Nesse caso, escolha o Hub IoT.

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades. 

| | Hub IoT | Hubs de Eventos | Kafka no HDInsight |
| --- | --- | --- | --- |
| Comunicações da nuvem para o dispositivo | SIM | Não | Não  |
| Upload de arquivo iniciado pelo dispositivo | SIM | Não | Não  |
| Informações de estado do dispositivo | [Dispositivos gêmeos](/azure/iot-hub/iot-hub-devguide-device-twins) | Não  | Não  |
| Suporte a protocolo | MQTT, AMQP, HTTPS <sup>1</sup> | AMQP, HTTPS | [Protocolo do Kafka](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol) |
| Segurança | Identidade por dispositivo; controle de acesso revogável. | Políticas de acesso compartilhado; revogação limitada por meio de políticas do publicador. | Autenticação com SASL; autorização conectável; integração com serviços de autenticação externa compatíveis. |

[1] Também use o [gateway de protocolo IoT do Azure](/azure/iot-hub/iot-hub-protocol-gateway) como um gateway personalizado para habilitar a adaptação de protocolo para o Hub IoT.

Para obter mais informações, consulte [Comparação entre o Hub IoT do Azure e os Hubs de Eventos do Azure](/azure/iot-hub/iot-hub-compare-event-hubs).
