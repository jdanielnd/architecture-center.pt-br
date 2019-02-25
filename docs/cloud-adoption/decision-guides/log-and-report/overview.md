---
title: 'CAF: Guia de decisão de registro em log e relatório'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Saiba mais sobre registro em log, relatório e monitoramento como principais serviços nas migrações do Azure.
author: rotycenh
ms.openlocfilehash: 36552488872622ec59e2fcf4816da4184c3d4fbf
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900288"
---
# <a name="logging-and-reporting-decision-guide"></a>Guia de decisão de registro em log e relatório

Todas as organizações precisam de mecanismos para notificar as equipes de TI sobre problemas de desempenho, tempo de atividade e segurança antes que tornem-se problemas sérios. Uma estratégia de monitoramento com êxito permite reconhecer como está o desempenho dos componentes individuais que compõem as cargas de trabalho e a infraestrutura de rede. No contexto de uma migração de nuvem pública, integrar registro em log e relatório a qualquer um dos sistemas de monitoramento existentes, enquanto monitora eventos e métricas importantes para a equipe de TI apropriada, é essencial para garantir que a organização atenda às metas de tempo de atividade, segurança e conformidade com a política.

![Plotagem das opções de registro em log, relatório e monitoramento do menos ao mais complexo, alinhado com links de salto abaixo](../../_images/discovery-guides/discovery-guide-logs-and-reporting.png)

Ir para: [Planejar a infraestrutura de monitoramento](#planning-your-monitoring-infrastructure) | [Nativo de Nuvem](#cloud-native) | [Extensão local](#on-premises-extension) | [Agregação de gateway](#gateway-aggregation) | [Monitoramento híbrido (local)](#hybrid-monitoring-on-premises) | [Monitoramento híbrido (baseado em nuvem)](#hybrid-monitoring-cloud-based) | [Multinuvem](#multi-cloud) | [Saiba mais](#learn-more)

O ponto de inflexão ao determinar uma estratégia de identidade na nuvem baseia-se principalmente nos investimentos existentes feitos pela organização nos processos operacionais e, até certo ponto, nos requisitos necessários para dar suporte a uma estratégia de multinuvem.

Há várias maneiras de registrar e relatar atividades na nuvem. O registro em log nativo e centralizado em nuvem são duas opções comuns de SaaS (software como serviço) que são orientadas pelo design de assinatura e pelo número de assinaturas.

## <a name="planning-your-monitoring-infrastructure"></a>Planejar a infraestrutura de monitoramento

Ao planejar a implantação, será necessário considerar onde os dados de registro em log serão armazenados e como os serviços de relatório e monitoramento baseados em nuvem serão integrados com os processos e as ferramentas existentes.

| Pergunta | Nativo de nuvem | Extensão local | Monitoramento híbrido | Agregação de gateway |
|-----|-----|-----|-----|-----|
| Há uma infraestrutura de monitoramento local existente? | Não  | sim | sim |  Não  |
| Há requisitos para impedir o armazenamento de dados do log em locais de armazenamento externos? | Não  | Sim | Não | Não  |
| Você precisa integrar o monitoramento de nuvem com sistemas locais? | Não  | Não  | Sim | Não  |
Você precisará processar ou filtrar dados telemétricos, antes de enviá-los aos sistemas de monitoramento? | Não  | Não | Não  | Sim |

### <a name="cloud-native"></a>Nativo de nuvem

Se a organização atualmente não possuir sistemas de registros em log e relatórios estabelecidos, ou se a implantação de nuvem planejada não precisar ser integrada com sistema local existente ou outros sistemas de monitoramento externos, uma solução de SaaS nativa de nuvem será a opção mais simples.

Nesse cenário, os dados do log são registrados e armazenados no mesmo ambiente de nuvem da carga de trabalho, enquanto as ferramentas de registro em log e relatórios que processam e exibem informações para a equipe de TI são oferecidas como parte da plataforma de nuvem.

As soluções de registro em log nativas de nuvem podem ser implementadas ad hoc por assinatura ou carga de trabalho para implantações experimentais ou menores e são organizadas de maneira centralizada para monitorar dados do log em toda a propriedade de nuvem.

**Suposições sobre a nuvem nativa**. O uso de um sistema de registro em log e relatório de nuvem nativa supõe o seguinte:

- Não é necessário integrar os dados do log das cargas de trabalho de nuvem em sistemas locais existentes.
- Você não usará os sistemas de relatórios baseados em nuvem para monitorar sistemas locais.

### <a name="on-premises-extension"></a>Extensão local

Nos cenários em que for necessário integrar telemetria de nuvem com sistemas locais que não dão suporte a registro em log e relatório híbrido, ou suporte à migração de aplicativos e serviços com um valor mínimo de redesenvolvimento, será necessário implantar agentes de monitoramento em VMs que enviarão dados do log diretamente aos sistemas locais, em vez de armazená-los no ambiente de nuvem.

Para dar suporte a essa abordagem, os recursos de nuvem precisarão comunicar-se diretamente com os sistemas locais por meio de uma combinação de [rede híbrida](../software-defined-network/hybrid.md) e [serviços de domínio hospedados na nuvem](../identity/overview.md#cloud-hosted-domain-services). Dessa forma, a rede virtual na nuvem funcionará como uma extensão da rede do ambiente local. Consequentemente, as cargas de trabalho hospedadas na nuvem poderão comunicar-se diretamente com o sistema de registro em log e relatórios local.

Essa abordagem converte o investimento existente em ferramentas de monitoramento com modificações limitadas para todos os aplicativos ou serviços implantados na nuvem. Geralmente, essa é a abordagem mais rápida para dar suporte ao monitoramento durante uma migração lift-and-shift. Porém, não captura dados do log produzidos por recursos de PaaS e SaaS baseados em nuvem e omitirá todos os logs relacionados a VM gerados pela própria plataforma de nuvem, como o status da VM. Como resultado, esse padrão deverá ser uma solução temporária até que uma solução de monitoramento híbrido mais abrangente seja implementada.

Suposições sobre somente no local:

- É necessário manter os dados do log somente no ambiente local para dar suporte aos requisitos técnicos ou devido a requisitos da política ou regulatórios.
- Os sistemas locais não dão suporte a registros em log e relatórios híbridos ou soluções de agregação de gateway.
- Os aplicativos baseados em nuvem podem enviar telemetria diretamente aos sistemas de registro em log local ou os agentes de monitoramento que enviam para o local podem ser implantados em VMs de carga de trabalho.
- As cargas de trabalho não dependem de serviços de PaaS ou SaaS que exigem registros em log e relatórios baseados em nuvem.

### <a name="gateway-aggregation"></a>Agregação de gateway

Para cenários em que o valor dos dados telemétricos baseados em nuvem for muito grande ou os sistemas de monitoramento locais existentes precisarem de dados do log modificados antes de serem processados, um serviço de [agregação de gateway](../../../patterns/gateway-aggregation.md) de dados do log poderá ser necessário.

Um serviço de gateway é implantado no provedor de nuvem. Em seguida, aplicativos e serviços relevantes são configurados para enviar dados telemétricos para o gateway, em vez de um sistema de registro em log padrão. O gateway poderá, em seguida, processar os dados: agregando, combinando ou formatando-os antes de enviá-los ao serviço de monitoramento para ingestão e análise.

Além disso, um gateway poderá ser utilizado para agregar e pré-processar dados telemétricos vinculados a sistemas híbridos ou nativos de nuvem.

Suposições sobre a agregação de gateway:

- Você espera níveis muito altos de dados telemétricos dos aplicativos ou serviços baseados em nuvem.
- Você precisará formatar ou otimizar os dados telemétricos, antes de enviá-los aos sistemas de monitoramento.
- Os sistemas de monitoramento têm APIs ou outros mecanismos disponíveis para ingerir dados do log após processamento pelo gateway.

### <a name="hybrid-monitoring-on-premises"></a>Monitoramento híbrido (local)

Uma solução de monitoramento híbrido combina dados do log dos recursos locais e da nuvem para fornecer uma exibição integrada do status operacional da propriedade de TI.

Se você tiver um investimento existente em sistemas de monitoramento local cuja substituição se tornaria difícil ou de alto custo, talvez seja necessário integrar a telemetria das cargas de trabalho de nuvem em soluções de monitoramento locais preexistentes. Em um sistema de monitoramento local híbrido, os dados telemétricos locais continuam usando o sistema de monitoramento local existente. Os dados telemétricos baseados em nuvem são enviados diretamente ao sistema de monitoramento de nuvem, ou os dados são armazenados na nuvem junto com as cargas de trabalho e, em seguida, compilados e ingeridos no sistema local em intervalos regulares.

**Suposições sobre o monitoramento híbrido local**. O uso de um sistema local de registro em log e relatório para monitoramento híbrido supõe o seguinte:

- Você precisa usar sistemas de relatórios locais existentes para monitorar as cargas de trabalho na nuvem.
- Você precisa manter a propriedade dos dados do log no local.
- Os sistemas de gerenciamento locais têm APIs ou outros mecanismos disponíveis para ingerir dados do log de sistemas baseados em nuvem.

> [!TIP]
> Como parte da natureza iterativa da migração na nuvem, é provável a transição de monitoramento local e nativo de nuvem distinto para uma abordagem híbrida parcial. Certifique-se de manter as alterações na arquitetura de monitoramento alinhadas com os processos operacionais e de TI globais.

### <a name="hybrid-monitoring-cloud-based"></a>Monitoramento híbrido (baseado em nuvem)

Se você não tiver uma forte necessidade para manter um sistema de monitoramento local, ou se quiser substituir os sistemas de monitoramento locais por uma solução de SaaS, também poderá optar por integrar dados do log locais com um sistema de monitoramento baseado em nuvem centralizado.

Reproduzindo a abordagem centralizada local, nesse cenário, as cargas de trabalho de nuvem usariam o mecanismo de registro em log de nuvem padrão, e os aplicativos e serviços locais enviariam do diretório de telemetria para o sistema de registro em log baseado em nuvem, ou agregariam esses dados para ingestão no sistema de nuvem em intervalos regulares. O sistema de monitoramento baseado em nuvem serviria como o sistema primário de monitoramento e relatório para toda a área de TI.

Suposições sobre o monitoramento híbrido baseado em nuvem: O uso de sistemas de registro em log e relatório baseados em nuvem para monitoramento híbrido supõe o seguinte:

- Você não depende de sistemas de monitoramento locais existentes.
- As cargas de trabalho não possuem requisitos regulatórios ou de política para armazenar dados do log no local.
- Os sistemas de monitoramento baseados em nuvem têm APIs ou outros mecanismos disponíveis para ingerir dados do log de aplicativos e serviços locais.

### <a name="multi-cloud"></a>Várias nuvens

Integrar os recursos de registro em log e relatório em uma plataforma com várias nuvens pode ser complexo. Os serviços oferecidos entre plataformas muitas vezes não são diretamente comparáveis, e os recursos de registro em log e telemetria fornecidos por esses serviços também são diferentes.
O suporte para registro em log em várias nuvens geralmente exigirá o uso de serviços de gateway para processar dados do log em um formato comum, antes de enviar os dados para uma solução de registro em log híbrida.

## <a name="learn-more"></a>Saiba mais

O [Azure Monitor](/azure/azure-monitor/overview) é o serviço de monitoramento e relatório padrão do Azure. Ele fornece:

- Uma plataforma unificada para coletar telemetria de aplicativos, telemetria do host (como VMs), métricas de contêineres, métricas da plataforma do Azure e logs de eventos.
- Visualização, consultas, alertas e ferramentas de análise. Adicionalmente, pode fornecer informações sobre máquinas virtuais, sistemas operacionais convidados, redes virtuais e eventos de aplicativo de carga de trabalho.
- [APIs REST](/azure/monitoring-and-diagnostics/monitoring-rest-api-walkthrough) para integração com serviços externos e automação de serviços de monitoramento e alerta
- [Integração](/azure/monitoring-and-diagnostics/monitoring-partners) com muitos fornecedores de terceiros populares.
