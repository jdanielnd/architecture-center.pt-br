---
title: Usar serviços gerenciados
titleSuffix: Azure Application Architecture Guide
description: Quando possível, utilize a plataforma como um serviço (PaaS) em vez da infraestrutura como um serviço (IaaS).
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: f08914e1eacc4f02fdb16093fe590f46249e3da3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249531"
---
# <a name="use-managed-services"></a>Usar serviços gerenciados

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a>Quando possível, utilize a plataforma como um serviço (PaaS) em vez de infraestrutura como serviço (IaaS)

O IaaS é como ter uma caixa de peças. Você pode criar qualquer coisa, mas deve montar por conta própria. Os serviços gerenciados são mais fáceis de configurar e de administrar. Você não precisa provisionar máquinas virtuais, configurar VNets, gerenciar atualizações e patches e toda a outra sobrecarga associada à execução de software em uma máquina virtual.

Por exemplo, suponha que seu aplicativo precisa de uma fila de mensagens. Você pode configurar seu próprio serviço de mensagens em uma VM, algo como o RabbitMQ. Mas o Barramento de Serviço do Azure já fornece mensagens confiáveis como serviço, e é mais simples de configurar. Basta criar um namespace do Barramento de Serviço (o que pode ser feito como parte de um script de implantação) e então chame o Barramento de Serviço usando o SDK do cliente.

Obviamente, seu aplicativo pode ter requisitos específicos que tornam uma abordagem IaaS mais adequada. No entanto, mesmo se seu aplicativo se basear em IaaS, procure locais onde pode ser natural incorporar serviços gerenciados. Isso inclui o armazenamento de dados, filas e cache.

| Em vez de executar... | Considere usar... |
|-----------------------|-------------|
| Active Directory | Azure Active Directory Domain Services |
| Elasticsearch | Azure Search |
| O Hadoop | HDInsight |
| IIS | Serviço de Aplicativo |
| MongoDB | Cosmos DB |
| Redis | Cache Redis do Azure |
| SQL Server | Banco de Dados SQL do Azure |
