---
title: Compilar microsserviços no Azure
description: Projetar, compilar e operar arquiteturas de microsserviços no Azure
ms.date: 03/07/2019
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: e74d6f6098eb68c8bbd737cf3a047ce5de04327d
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58346137"
---
# <a name="building-microservices-on-azure"></a>Compilar microsserviços no Azure

<!-- markdownlint-disable MD033 -->

<img src="../_images/microservices.svg" style="float:left; margin-top:8px; margin-right:8px; max-width: 80px; max-height: 80px;"/>

Microsserviços são um estilo popular de arquitetura para compilar aplicativos resilientes, altamente escalonáveis, implantáveis independentemente e capazes de evoluir rapidamente. Mas, uma arquitetura de microsserviços bem-sucedida exige uma abordagem diferente ao projeto e à compilação de aplicativos.

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./introduction.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>O que são microsserviços?</h3>
                        <p>Qual a diferença dos microsserviços em relação a outras arquiteturas e quando se deve usá-los?</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../guide/architecture-styles/microservices.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Estilo de arquitetura de microsserviços</h3>
                        <p>Visão geral de alto nível do estilo de arquitetura de microsserviços</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="examples-of-microservices-architectures"></a>Exemplos de arquiteturas de microsserviços

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="../example-scenario/infrastructure/service-fabric-microservices.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Usar o Service Fabric para decompor aplicativos monolíticos</h3>
                        <p>Uma abordagem iterativa à decomposição de um site do ASP.NET em microsserviços.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../example-scenario/data/ecommerce-order-processing.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Processamento de pedidos escalonável no Azure</h3>
                        <p>Processamento de pedidos usando um modelo de programação funcional implementado por meio de microsserviços.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="build-a-microservices-application"></a>Compilar um aplicativo de microsserviços

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./model/domain-analysis.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Usar análise de domínio para microsserviços de modelo</h3>
                        <p>Para evitar algumas armadilhas comuns ao projetar microsserviços, use a análise de domínio para definir os limites do microsserviço.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../reference-architectures/microservices/aks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Arquitetura de referência do Serviço de Kubernetes do Azure (AKS)</h3>
                        <p>Essa arquitetura de referência mostra uma configuração básica do AKS que pode ser o ponto de partida da maioria das implantações.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../reference-architectures/microservices/service-fabric.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Arquitetura de referência do Azure Service Fabric</h3>
                        <p>Essa arquitetura de referência mostra a configuração recomendada que pode ser o ponto de partida da maioria das implantações.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./design/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Projetar uma arquitetura de microsserviços</h3>
                        <p>Esses artigos aprofundam seus conhecimentos sobre como compilar um aplicativo de microsserviços, com base em uma implementação de referência que usa o Serviço de Kubernetes do Azure (AKS).</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./design/patterns.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Padrões de design</h3>
                        <p>Um conjunto de padrões de design úteis dos microsserviços.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="operate-microservices-in-production"></a>Operar microsserviços na produção

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./logging-monitoring.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Log e monitoramento</h3>
                        <p>A natureza distribuída das arquiteturas de microsserviços torna o log e o monitoramento especialmente críticos.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./ci-cd.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Integração e implantação contínuas</h3>
                        <p>CI/CD (integração contínua e entrega contínua) são fundamentais para se obter sucesso com os microsserviços.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>
