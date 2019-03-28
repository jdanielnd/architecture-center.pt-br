---
title: Arquiteturas de referência do Azure
description: Arquiteturas de referência e diretrizes de implementação para cargas de trabalho comuns no Azure.
layout: LandingPage
ms.topic: landing-page
ms.date: 03/07/2019
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="azure-reference-architectures"></a>Arquiteturas de referência do Azure

Nossas arquiteturas de referência são organizadas por cenário. Cada arquitetura inclui as práticas recomendadas, junto com as considerações sobre escalabilidade, disponibilidade, capacidade de gerenciamento e segurança. A maioria também inclui uma solução implantável ou uma implementação de referência.

Ir para: [IA](#ai-and-machine-learning) | [Big Data](#big-data-solutions) | [IoT](#internet-of-things) | [Microsserviços](#microservices) | [Sem servidor](#serverless-applications) | [Redes virtuais](#virtual-networks) | [Cargas de trabalho de VM](#vm-workloads) | [SAP](#sap) | [Active Directory](#extend-on-premises-active-directory-to-azure) | [Aplicativos Web](#web-applications)

<!-- markdownlint-disable MD033 -->

## <a name="ai-and-machine-learning"></a>IA e machine learning

<ul  class="panelContent cardsF">
<!-- Distributed training of deep learning models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/training-deep-learning.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/batch-ai.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Treinamento distribuído de modelos de aprendizado profundo</h3>
                        <p>Execute o treinamento distribuído de modelos de aprendizado profundo em clusters de VMs habilitadas para GPU.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Training of Python scikit-learn models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/training-python-models.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/python-powered-h.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Treinamento dos modelos scikit-learn de Python</h3>
                        <p>Práticas recomendadas para ajustar os hiperparâmetros de um modelo scikit-learn de Python.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Batch scoring for deep learning models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/batch-scoring-deep-learning.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/batch-ai.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Pontuação em lote para modelos de aprendizado profundo</h3>
                        <p>Automatize os trabalhos em lotes executados que aplicam a transferência neural em um vídeo.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Batch scoring of Python models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/batch-scoring-python.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/python-powered-h.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Pontuação em lote dos modelos de Python</h3>
                        <p>Pontue em lote vários modelos de Python simultaneamente em um agendamento, usando o Azure Machine Learning.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Batch scoring of Spark models on Azure Databricks -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/batch-scoring-databricks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/databricks.png" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Pontuação em lote dos modelos do Spark no Azure Databricks</h3>
                        <p>Crie uma solução escalonável para pontuar em lote um modelo de classificação do Apache Spark usando o Azure Databricks.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Real-time scoring of Python and deep learning models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/realtime-scoring-python.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/python-powered-h.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Pontuação em tempo real do Python e modelos de aprendizado profundo</h3>
                        <p>Implante os modelos em Python como serviços da web para fazer previsões em tempo real, usando os modelos regulares do Python ou os modelos de aprendizado profundo.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Real-time scoring of R machine learning models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/realtime-scoring-r.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/logo-r.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Pontuação em tempo real dos modelos do Machine Learning em R</h3>
                        <p>Implemente um serviço de previsão em tempo real em R usando o Microsoft Machine Learning Server em execução no Serviço de Kubernetes do Azure (AKS).</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Real-time recommendation API -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/real-time-recommendation.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/machine-learning.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>API de recomendação em tempo real</h3>
                        <p>Treine um modelo de recomendação usando o Azure Databricks e implante-o como uma API usando o Azure Machine Learning.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Enterprise-grade conversational bot -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/conversational-bot.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/bot-services.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Bot conversacional de nível empresarial</h3>
                        <p>Como criar um bot conversacional de nível empresarial usando o Azure Bot Framework.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="big-data-solutions"></a>Soluções de Big Data

<ul  class="panelContent cardsF">
<!-- Enterprise BI with SQL Data Warehouse -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/enterprise-bi-sqldw.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/data-guide.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Enterprise BI com o SQL Data Warehouse</h3>
                        <p>Pipeline ELT (extrair-carregar-transformar) para mover dados de um banco de dados local para o SQL Data Warehouse.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Automated enterprise BI with Azure Data Factory -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/enterprise-bi-adf.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/data-guide.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Enterprise BI automatizada com o Azure Data Factory</h3>
                        <p>Automatize um pipeline ELT para executar um carregamento incremental a partir do banco de dados local.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Stream processing with Azure Databricks -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/stream-processing-databricks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/databricks.png" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Processamento de fluxo com o Azure Databricks</h3>
                        <p>Pipeline de processamento de fluxo que une registros de dois fluxos, enriquece o resultado e calcula uma média móvel.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Stream processing with Azure Stream Analytics -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/stream-processing-stream-analytics.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/azure-analysis-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Processamento de fluxo com o Azure Stream Analytics</h3>
                        <p>Pipeline de processamento do fluxo de ponta a ponta que correlaciona os registros de dois fluxos de dados para calcular uma média móvel.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="internet-of-things"></a>Internet das coisas

<ul  class="panelContent cardsF">
<!-- Azure IoT reference architecture -->
<li style="display: flex; flex-direction: column;">
    <a href="./iot/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./iot/_images/iot.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Arquitetura de referência de IoT do Azure</h3>
                        <p>Arquitetura recomendada para aplicativos de IoT no Azure usando os componentes do PaaS (plataforma como serviço).</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="microservices"></a>Microsserviços

<ul  class="panelContent cardsF">
<!-- Microservices on Azure Kubernetes Service (AKS) -->
<li style="display: flex; flex-direction: column;">
    <a href="./microservices/aks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/aks.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Microsserviços no Serviço de Kubernetes do Azure (AKS)</h3>
                        <p>Arquitetura recomendada para implantar microsserviços no AKS.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Microservices architecture on Azure Service Fabric -->
<li style="display: flex; flex-direction: column;">
    <a href="./microservices/service-fabric.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sf.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Arquitetura de microsserviços no Azure Service Fabric</h3>
                        <p>Arquitetura recomendada de microsserviços no Service Fabric.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="serverless-applications"></a>Aplicativos sem servidor

<ul  class="panelContent cardsF">
<!-- Serverless web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./serverless/web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/functions.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicativo web sem servidor</h3>
                        <p>Um aplicativo web sem servidor que fornece conteúdo estático do Armazenamento de Blobs e implementa uma API usando o Azure Functions.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Event processing using Azure Functions -->
<li style="display: flex; flex-direction: column;">
    <a href="./serverless/event-processing.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/functions.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Processamento de eventos usando o Azure Functions</h3>
                        <p>Uma arquitetura orientada a eventos que consome um fluxo de dados e usa o Functions para processar os dados.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="virtual-networks"></a>Redes virtuais

<ul  class="panelContent cardsF">
<!-- Hybrid network using a virtual private network (VPN) -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/vpn.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vpn.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Rede híbrida usando uma rede privada virtual (VPN)</h3>
                        <p>Conecte a rede local a uma rede virtual do Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hybrid network using ExpressRoute -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/expressroute.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Rede híbrida usando o ExpressRoute</h3>
                        <p>Use uma conexão privada e dedicada para estender uma rede local para o Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hybrid network using ExpressRoute with VPN failover -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/expressroute-vpn-failover.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Rede híbrida usando o ExpressRoute com failover de VPN</h3>
                        <p>Use o ExpressRoute com uma VPN como uma conexão de failover para alta disponibilidade.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hub-spoke network topology -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/hub-spoke.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/gateway.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Topologia de rede hub-spoke</h3>
                        <p>Crie um ponto central de conectividade para sua rede local enquanto isola cargas de trabalho.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hub-spoke topology with shared services -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/shared-services.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/gateway.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Topologia hub-spoke com serviços compartilhados</h3>
                        <p>Estenda uma topologia hub-spoke incluindo serviços compartilhados como o Active Directory.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- DMZ between Azure and on-premises -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/secure-vnet-hybrid.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>DMZ entre o Azure e local</h3>
                        <p>Use soluções de virtualização de rede para criar uma rede híbrida segura.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- DMZ between Azure and the Internet -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/secure-vnet-dmz.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>DMZ entre o Azure e a Internet</h3>
                        <p>Use soluções de virtualização de rede para criar uma rede segura que aceita tráfego da Internet.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Highly available network virtual appliances -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/nva-ha.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Dispositivos virtuais de rede altamente disponíveis</h3>
                        <p>Implantar um conjunto de NVAs (soluções de virtualização de rede) para alta disponibilidade no Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="vm-workloads"></a>Cargas de trabalho de VM

<ul  class="panelContent cardsF">
<!-- N-tier application with SQL Server -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/n-tier-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicativo de n camadas com SQL Server</h3>
                        <p>Máquinas virtuais configuradas para um aplicativo de N camadas usando o SQL Server no Windows.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Multi-region N-tier application -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/multi-region-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicativo de n camadas de várias regiões</h3>
                        <p>Um aplicativo de N camadas em duas regiões para ter alta disponibilidade usando Grupos de Disponibilidade Always On do SQL Server.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- N-tier application with Cassandra -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/n-tier-cassandra.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/linux-penguin.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicativo de n camadas com Cassandra</h3>
                        <p>Máquinas virtuais configuradas para um aplicativo de N camadas usando o Apache Cassandra no Linux.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Jenkins build server -->
<li style="display: flex; flex-direction: column;">
    <a href="./jenkins/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/jenkins.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Servidor de build Jenkins</h3>
                        <p>Servidor Jenkins, escalonável, de nível corporativo, no Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- SharePoint Server 2016 farm -->
<li style="display: flex; flex-direction: column;">
    <a href="./sharepoint/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sharepoint.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Farm do SharePoint Server 2016</h3>
                        <p>Farm do SharePoint Server 2016 de alta disponibilidade no Azure com o SQL Server Always On em Grupos de Disponibilidade.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="sap"></a>SAP

<ul  class="panelContent cardsF">
<!-- SAP NetWeaver -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/sap-netweaver.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP NetWeaver</h3>
                        <p>SAP NetWeaver no Windows, em um ambiente de alta disponibilidade que oferece suporte à recuperação de desastres.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- SAP S/4HANA -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/sap-s4hana.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP S/4HANA</h3>
                        <p>SAP S/4HANA no Linux, em um ambiente de alta disponibilidade que oferece suporte à recuperação de desastres.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- SAP HANA on Azure Large Instances -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/hana-large-instances.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP HANA em Instâncias Grandes do Azure</h3>
                        <p>Instâncias grandes do HANA são implantadas em servidores físicos em regiões do Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="extend-on-premises-active-directory-to-azure"></a>Estenda o Active Directory local para o Azure

<ul  class="panelContent cardsF">
<!-- Integrate with Azure Active Directory -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/azure-ad.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/azure-active-directory.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Integre-se com o Azure Active Directory</h3>
                        <p>Integre domínios do AD local ao Azure Active Directory.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Extend an on-premises Active Directory domain to Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adds-extend-domain.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Estender um domínio do Active Directory local para o Azure</h3>
                        <p>Implante os Active Directory Domain Services (AD DS) no Azure para estender seu domínio local.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Create an AD DS forest in Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adds-forest.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Criar uma floresta do AD DS no Azure</h3>
                        <p>Crie um domínio separado do Azure AD que é confiável para a floresta do AD local.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Extend Active Directory Federation Services (AD FS) to Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adfs.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Estender o Serviços de Federação do Active Directory (AD FS) para o Azure</h3>
                        <p>Use o AD FS para autorização e autenticação federada de componentes que estão sendo executados no Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="web-applications"></a>Aplicativos Web

<ul  class="panelContent cardsF">
<!-- Basic web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/basic-web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicativo Web básico</h3>
                        <p>Um aplicativo Web com o Serviço de Aplicativo do Azure e o Banco de Dados SQL do Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Highly scalable web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/scalable-web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicativo Web altamente escalonável</h3>
                        <p>Práticas comprovadas para melhora da escalabilidade em um aplicativo Web.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Highly available web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/multi-region.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicativo Web altamente disponível</h3>
                        <p>Execute um aplicativo Web nos Serviço de Aplicativo para alcançar a alta disponibilidade.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Web application monitoring on Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/app-monitoring.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Monitoramento de aplicativo Web no Azure</h3>
                        <p>Monitore um aplicativo Web hospedado no Serviço de Aplicativo do Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>