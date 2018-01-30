---
title: "Arquiteturas de referência do Azure"
description: "Arquiteturas de referência, plantas e diretrizes de implementação prescritivas para cargas de trabalho comuns no Azure."
layout: LandingPage
ms.openlocfilehash: 0cc03f87dae39517e1a72a65d4767dcc21879d8f
ms.sourcegitcommit: 9998334bebccb86be0f715ac7dffc0c3175aea68
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/26/2018
---
# <a name="azure-reference-architectures"></a>Arquiteturas de referência do Azure

Nossas arquiteturas de referência são organizadas por cenário, com arquiteturas relacionadas agrupadas juntos. Cada arquitetura inclui as práticas recomendadas, junto com as considerações sobre escalabilidade, disponibilidade, capacidade de gerenciamento e segurança. Mais também incluem uma solução implantável.

<section class="series">
    <ul class="panelContent">
    <!--Windows VM -->
    <li>
        <a href="./virtual-machines-windows/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./virtual-machines-windows/images/n-tier.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Cargas de trabalho de máquina virtual do Windows</h3>
                            <p>Esta série começa com as práticas recomendadas para executar uma única VM do Windows e então várias VMs com balanceamento de carga e, por fim, um aplicativo de N camadas de várias regiões.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Linux VM -->
    <li>
        <a href="./virtual-machines-linux/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./virtual-machines-linux/images/n-tier.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Cargas de trabalho de VM do Linux</h3>
                            <p>Esta série começa com as práticas recomendadas para executar uma única VM do Linux e então várias VMs com balanceamento de carga e, por fim, um aplicativo de N camadas de várias regiões.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Hybrid network -->
    <li>
        <a href="./hybrid-networking/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./hybrid-networking/images/vpn.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Rede híbrida</h3>
                            <p>Esta série mostra as opções para criar uma conexão de rede entre uma rede local e o Azure.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- DMZ -->
    <li>
        <a href="./dmz/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./dmz/images/secure-vnet-dmz.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>DMZ da Rede</h3>
                            <p>Esta série mostra como criar uma rede DMZ para proteger o limite entre uma rede virtual do Azure e uma rede local ou a Internet.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Identity -->
    <li>
        <a href="./identity/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./identity/images/adds-extend-domain.svg" height="140px" >
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Gerenciamento de identidades</h3>
                            <p>Esta série mostra opções para integrar seu ambiente do Active Directory (AD) local com uma rede do Azure.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Managed web app -->
    <li>
        <a href="./app-service-web-app/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./app-service-web-app/images/scalable-web-app.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Aplicativo Web do Serviço de Aplicativo</h3>
                            <p>Esta série mostra as práticas recomendadas para aplicativos Web que usam o Serviço de Aplicativo do Azure.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    </ul>

<ul class="panelContent cardsI">
<li>
    <a href="./jenkins/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./jenkins/images/logo.svg" alt="Jenkins" height="100%" />
                    </div>
                </div>
                <div class="cardText">
                    <h3>Servidor de build Jenkins</h3>
                    <p>Implante e opere um servidor Jenkins, escalonável, de nível corporativo, no Azure.</p>
                </div>
            </div>
        </div>
    </div>
    </a>
</li>

<li>
    <a href="./sharepoint/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sharepoint/images/sharepoint.svg" alt="SharePoint Server 2016" height="100%" />
                    </div>
                </div>
                <div class="cardText">
                    <h3>Farm do SharePoint Server 2016</h3>
                    <p>Implante e execute um farm do SharePoint Server 2016 de alta disponibilidade no Azure com o SQL Server AlwaysOn em grupos de disponibilidade.</p>
                </div>
            </div>
        </div>
    </div>
    </a>
</li>

<li>
    <a href="./sap/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sap/images/sap.svg" alt="SAP NetWeaver and SAP HANA" width="100%" />
                    </div>
                </div>
                <div class="cardText">
                    <h3>SAP NetWeaver e SAP HANA</h3>
                    <p>Implante e execute o SAP NetWeaver e o SAP HANA em um ambiente de alta disponibilidade no Azure.</p>
                </div>
            </div>
        </div>
    </div>
    </a>
</li>
</ul>


</section>

