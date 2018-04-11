---
title: Gerenciamento de identidades
description: Explica e compara os métodos diferentes disponíveis para o gerenciamento de identidades em sistemas híbrida que abrangem o limite no local/em nuvem com o Azure.
layout: LandingPage
ms.openlocfilehash: de98ee30306f5e712759ab7140bd430cb6d4cd75
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/23/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="identity-management"></a>Gerenciamento de identidades

Essas arquiteturas mostram opções para integrar seu ambiente do Active Directory (AD) local com uma rede do Azure. <br/>[Qual devo escolher?](./considerations.md)

<section class="series">
    <ul class="panelContent">
    <!-- Integrate with Azure Active Directory -->
<li style="display: flex; flex-direction: column;">
    <a href="./azure-ad.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/azure-ad.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Integre-se com o Azure Active Directory</h3>
                        <p>Integre florestas e domínios do Active Directory local com o Azure AD.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Extend AD DS to Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./adds-extend-domain.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/adds-extend-domain.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Estender o AD DS para o Azure</h3>
                        <p>Estenda seu ambiente do Azure Active Directory usando os Serviços de Domínio Active Directory.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Create an AD DS forest in Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./adds-forest.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/adds-forest.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Criar uma floresta do AD DS no Azure</h3>
                        <p>Crie um domínio separado do Azure AD que é confiável para domínios na floresta local.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Extend AD FS to Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./adfs.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/adfs.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Estender o AD FS para o Azure</h3>
                        <p>Use os Serviços de Federação do Active Directory para autenticação federada e autorização no Azure.</p>
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