---
title: Cargas de trabalho de VM do Linux
description: "Explica algumas arquiteturas comuns para a implantação de máquinas virtuais que hospedam aplicativos de grande porte no Azure."
layout: LandingPage
ms.openlocfilehash: ef06fb93355f4676f44954930bcfeb2c2d012d43
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/23/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="linux-vm-workloads"></a>Cargas de trabalho de VM do Linux

Essas arquiteturas mostram práticas comprovadas para executar VMs do Linux no Azure.

<section class="series">
    <ul class="panelContent">
    <!-- Single VM -->
<li style="display: flex; flex-direction: column;">
    <a href="./single-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/single-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>VM única</h3>
                        <p>Recomendações de linha de base para executar qualquer VM do Linux no Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Load balanced VMs -->
<li style="display: flex; flex-direction: column;">
    <a href="./multi-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/multi-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>VMs com balanceamento de carga</h3>
                        <p>Várias VMs colocadas por trás de um balanceador de carga para escalabilidade e disponibilidade.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- N-tier application -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/n-tier.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicativo de N camadas</h3>
                        <p>Máquinas virtuais configuradas para um aplicativo de N camadas com Apache Cassandra.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Multi-region application -->
<li style="display: flex; flex-direction: column;">
    <a href="./multi-region-application.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/multi-region-application.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicativo de várias regiões</h3>
                        <p>Aplicativo de N camadas implantado em duas regiões para alta disponibilidade.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
</ul>