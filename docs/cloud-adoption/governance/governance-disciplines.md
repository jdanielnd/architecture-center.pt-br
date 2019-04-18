---
title: 'CAF: As cinco disciplinas da governança de nuvem'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Visão geral de conteúdo de governança em CAF
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: ddd7f5e4b7e79ebe2ddf87be7421b6c4691aced1
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639862"
---
# <a name="the-five-disciplines-of-cloud-governance"></a>As cinco disciplinas da governança de nuvem

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsI">
<li style="display: flex; flex-direction: column;">
    <div class="cardSize">
        <div class="cardPadding" style="padding-bottom:10px;">
            <div class="card" style="padding-bottom:10px;">
                <div class="cardText" style="padding-left:0px;">
Qualquer alteração em processos de negócios ou plataformas de tecnologia apresenta. Equipes de governança de nuvem, cujos membros muitas vezes são conhecidos como responsáveis da nuvem, ficam encarregadas de atenuar esses riscos com interrupção mínima para os esforços de adoção ou inovação.<br/><br/>O modelo de governança CAF orienta essas decisões (independentemente da plataforma de nuvem escolhido), concentrando-se no desenvolvimento de políticas corporativas e <a href="#disciplines-of-cloud-governance">disciplinas de governança de nuvem</a>. <a href="./journeys/overview.md">Percursos acionáveis</a> demonstrar esse modelo usando os serviços do Azure. Este artigo serve como uma página de aterrissagem para as cinco disciplinas do modelo de governança CAF.
                </div>
            </div>
        </div>
    </div>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../_images/operational-transformation-govern-highres.png" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize">
            <div class="cardPadding" style="padding-bottom:10px;">
                <div class="card" style="padding-bottom:10px;">
                    <div class="cardText" style="padding-left:0px;">
<img src="../_images/operational-transformation-govern-highres.png" alt="Diagram of the CAF governance model: Corporate policy and governance disciplines">
<br>
<i>Figura 1. Diagrama de política corporativa e disciplinas cinco de governança de nuvem</i>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="disciplines-of-cloud-governance"></a>Cinco disciplinas da governança de nuvem

Em cada provedor de nuvem, há disciplinas de governança comum que podem servir como um guia para ajudar a informar as políticas e alinhar cadeias de ferramentas. Esses disciplinas orientam as decisões sobre o nível adequado de automação e a imposição de política corporativa entre provedores na nuvem.

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsA">
<li style="display: flex; flex-direction: column;">
    <a href="./cost-management/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/cost-management.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Gerenciamento de Custos</h3>
                        <p>Custo é uma preocupação principal para usuários na nuvem. Desenvolva políticas de controle de custos para todas as plataformas de nuvem.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./security-baseline/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/security-baseline.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Linha de base de segurança</h3>
                        <p>A segurança é um tópico complexo e pessoal, exclusivo para cada empresa. Depois que os requisitos de segurança são estabelecidos, as políticas de controle de nuvem e a imposição aplicam esses requisitos em configurações de rede, dados e ativos.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./identity-baseline/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/identity-baseline.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Linha de base de identidade</h3>
                        <p>Inconsistências na aplicação de requisitos de identidade podem aumentar o risco de violação. A disciplina de linha de base de identidade se concentra em maneiras de garantir que a identidade seja consistentemente aplicada através de esforços de adoção de nuvem.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./resource-consistency/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/resource-consistency.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Consistência de recursos</h3>
                        <p>Operações na nuvem dependem de consistência na configuração de recurso. Por meio de ferramentas de governança, recursos consistentemente podem ser configurados para reduzir os riscos relacionados à integração, desvio, descoberta e recuperação.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./deployment-acceleration/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/deployment-acceleration.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Aceleração de implantação</h3>
                        <p>A centralização, padronização e a consistência em abordagens de implantação e configuração melhoram as práticas de governança. Quando disponibilizado por meio de ferramentas de controle baseado em nuvem, eles criam um fator de nuvem que pode acelerar a atividades de implantação.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->
