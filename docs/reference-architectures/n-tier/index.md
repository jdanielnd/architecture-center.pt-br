---
title: Arquiteturas de referência do aplicativo de n camadas
description: Explica algumas arquiteturas comuns para a implantação de máquinas virtuais que hospedam aplicativos de grande porte no Azure.
layout: LandingPage
ms.openlocfilehash: 288acc36e7c310e70240caa3ed9f2095bbb8bc58
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 05/10/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="n-tier-application-reference-architectures"></a>Arquiteturas de referência do aplicativo de n camadas

<section class="series">
    <ul class="panelContent">

<!-- N-tier Windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/n-tier-sql-server.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicativo de n camadas com SQL Server</h3>
                        <p>Implante VMs e uma rede virtual configurada para um aplicativo de n camadas usando o SQL Server no Windows.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- Multi-region Windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./multi-region-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/multi-region-application.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicativo de n camadas de várias regiões com SQL Server</h3>
                        <p>Implante um aplicativo de n camadas em duas regiões para ter alta disponibilidade, usando Grupos de Disponibilidade Always On do SQL Server.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- N-tier Linux -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier-cassandra.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/n-tier-cassandra.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aplicativo de n camadas com Cassandra</h3>
                        <p>Implante VMs Linux e uma rede virtual configurada para um aplicativo de n camadas usando o Apache Cassandra.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<ul class="panelContent cardsI">
<li style="display: flex; flex-direction: column;">
    <a href="./windows-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/Windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>VM Windows</h3>
                        <p>Recomendações da linha de base para executar uma MV Windows no Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<li style="display: flex; flex-direction: column;">
    <a href="./linux-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/LinuxPenguin.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>VM Linux</h3>
                        <p>Recomendações da linha de base para executar uma MV Linux no Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

</ul>