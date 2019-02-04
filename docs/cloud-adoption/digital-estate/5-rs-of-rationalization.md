---
title: 5 Rs da racionalização
titleSuffix: Enterprise Cloud Adoption
description: Descreve as opções disponíveis ao racionalizar uma propriedade digital
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 06058967e6ffcd9e3554a46c67144f72fb19078f
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908572"
---
# <a name="enterprise-cloud-adoption-the-5-rs-of-rationalization"></a>Adoção da Nuvem Empresarial: 5 Rs da racionalização

Racionalização de nuvem é o processo de avaliação de ativos para determinar a melhor abordagem para a migração ou modernização de cada ativo na nuvem. Para saber mais sobre o processo de racionalização, confira [O que é uma propriedade digital?](overview.md)

O "5 Rs da racionalização" listados aqui descrevem as opções mais comuns de racionalização.

## <a name="rehost"></a>Hospedar novamente

Também conhecido como "lift and shift", um esforço de nova hospedagem move o ativo de estado atual para o provedor de nuvem escolhido, com alterações mínimas na arquitetura geral.

Os motivadores comuns podem incluir:

* Reduzir CapEx
* Liberar espaço no datacenter
* ROI de nuvem rápido

Fatores de análise quantitativa:

* Tamanho da VM (CPU, memória, armazenamento)
* Dependências (tráfego de rede)
* Compatibilidade de ativo

Fatores de análise qualitativa:

* Tolerância à alteração
* Prioridades dos negócios
* Eventos comerciais críticos
* Dependências de processo

## <a name="refactor"></a>Refatoração

As opções de PaaS (Plataforma como serviço) podem reduzir os custos operacionais associados a muitos aplicativos. Pode ser prudente refatorar ligeiramente um aplicativo a fim de ajustá-lo a um modelo com base em PaaS.

Refatorar também se refere ao processo de desenvolvimento de aplicativo de refatoração de código, a fim de permitir que um aplicativo forneça novas oportunidades de negócios.

Os motivadores comuns podem incluir:

* Atualizações mais rápidas e curtas
* Portabilidade do código
* Maior eficiência de nuvem (recursos, velocidade e custo)

Fatores de análise quantitativa:

* Tamanho do ativo de aplicativo (CPU, memória, armazenamento)
* Dependências (tráfego de rede)
* Tráfego do usuário (exibições de página, hora na página, tempo de carregamento)
* Plataforma de desenvolvimento (linguagens, plataforma de dados, serviços de camada intermediária)

Fatores de análise qualitativa:

* Investimento contínuo nos negócios
* Opções/linhas do tempo de intermitência
* Dependências de processo comercial

## <a name="rearchitect"></a>Recriação da arquitetura

Alguns aplicativos mais velhos não são compatíveis com provedores de nuvem devido a decisões arquitetônicas tomadas durante a compilação do aplicativo. Nesses casos, o aplicativo pode precisar de uma nova arquitetura antes da transformação.

Em outros casos, os aplicativos que são compatíveis com a nuvem, mas que não conta com benefícios por serem nativos, podem melhorar o custo e as operações por meio da nova arquitetura da solução, a fim de se tornarem um aplicativo nativo de nuvem.

Os motivadores comuns podem incluir:

* Agilidade e escala do aplicativo
* Facilidade da adoção de novos recursos de nuvem
* Combinação de pilhas de tecnologia

Fatores de análise quantitativa:

* Tamanho do ativo de aplicativo (CPU, memória, armazenamento)
* Dependências (tráfego de rede)
* Tráfego do usuário (exibições de página, hora na página, tempo de carregamento)
* Plataforma de desenvolvimento (linguagens, plataforma de dados, serviços de camada intermediária)

Fatores de análise qualitativa:

* Investimentos comerciais em expansão
* Custos operacionais
* Possíveis loops de comentários e investimentos de DevOps

## <a name="rebuild"></a>Recompilação

Em alguns cenários, o delta que deve ser superado para avançar com um aplicativo pode ser muito grande para justificar o investimento. Isso é especialmente verdadeiro para aplicativos que costumavam atender às necessidades do negócio, mas que agora não têm suporte ou são não estão alinhados com o modo como os processos comerciais são executados atualmente. Nesse caso, uma nova base de código é criada de acordo com uma abordagem de [nuvem nativa](https://azure.microsoft.com/overview/cloudnative/).

Os motivadores comuns podem incluir:

* Acelerar a inovação
* Compilar aplicativos mais rápido
* Reduzir o custo operacional

Fatores de análise quantitativa:

* Tamanho do ativo de aplicativo (CPU, memória, armazenamento)
* Dependências (tráfego de rede)
* Tráfego do usuário (exibições de página, hora na página, tempo de carregamento)
* Plataforma de desenvolvimento (linguagens, plataforma de dados, serviços de camada intermediária)

Fatores de análise qualitativa:

* Satisfação do usuário final em queda
* Processos de negócio limitados pela funcionalidade
* Possíveis ganhos de custo, experiência ou receita

## <a name="replace"></a>Substitua

Normalmente, as soluções são implementadas usando a melhor tecnologia e abordagem disponíveis no momento. Em alguns casos, aplicativos SaaS (Software como serviço) podem atender a todas as funcionalidades necessárias do aplicativo hospedado. Nesses cenários, uma carga de trabalho pode ser programada para substituição futura, removendo-a efetivamente do esforço de transformação.

Os motivadores comuns podem incluir:

* Padronizar com base em práticas recomendadas do setor
* Acelerar a adoção de abordagens orientadas por processo empresarial
* Realocar investimentos em desenvolvimento para aplicativos que criam vantagens ou diferenciais com relação à concorrência

Fatores de análise quantitativa:

* Reduções de custo operacional geral
* Tamanho da VM (CPU, memória, armazenamento)
* Dependências (tráfego de rede)
* Ativos para desativação

Fatores de análise qualitativa:

* Análise de custo-benefício da arquitetura atual versus a solução de SaaS
* Mapas de processo de negócios
* Esquemas de dados
* Processos personalizados ou automatizados

## <a name="next-steps"></a>Próximas etapas

Coletivamente, esses 5 Rs da Racionalização podem ser aplicados a uma propriedade digital a fim de tomar decisões de racionalização sobre o estado futuro de cada aplicativo.

> [!div class="nextstepaction"]
> [O que é uma propriedade digital?](overview.md)
