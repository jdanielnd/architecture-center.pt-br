---
title: 'CAF: O que é classificação de dados?'
description: O que é classificação de dados?
author: BrianBlanchard
ms.date: 2/11/2019
ms.openlocfilehash: 07268e7242d92ac2581bf28b378a3c43d166620c
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900267"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-data-classification"></a>O que é classificação de dados?

Este é um artigo introdutório sobre o tópico geral da classificação de dados. A classificação de dados é um ponto de partida muito comum para toda a governança.

## <a name="business-risks-and-governance"></a>Governança e riscos de negócios

Na maioria das organizações, os principais motivos para investir em governança podem ser reduzidos para três riscos de negócios:

* Responsabilidade associada a violações de dados
* Interrupção para os negócios contra interrupções
* Gastos inesperados ou não planejados

Há muitas variantes desses três riscos de negócios. No entanto, a tendência é ser mais comum.

## <a name="understand-then-mitigate"></a>Entender e, em seguida, atenuar

Antes de qualquer risco poder ser atenuado, ele deve ser compreendido. No caso de responsabilidade de violação de dados, esse entendimento começa com a classificação de dados. A classificação de dados é o processo de associar uma característica de dados de metadados para todos os ativos em um estado digital, que identifica o tipo de dados associados a esse ativo.

A Microsoft sugere que qualquer ativo que for identificado como um candidato potencial para a migração ou implantação para a nuvem deve ter documentado metadados para registrar a classificação de dados, nível de importância de negócios e responsabilidade de cobrança. Esses três pontos de classificação podem ir muito longe para compreensão e minimização de riscos.

## <a name="microsofts-data-classification"></a>Classificação de dados da Microsoft

A seguir, uma lista de classificações de usos da Microsoft. Dependendo do seu setor ou requisitos de segurança existentes, os padrões de classificações de dados já podem existir dentro da sua organização. Se nenhum padrão existir, damos as boas-vindas para usar essa classificação de exemplo, para ajudá-lo a entender melhor o seu perfil de risco e espaço digital.  

* **Não comerciais:** Dados de sua vida pessoal que não pertencem à Microsoft
* **Público:** Dados corporativos preparados e aprovados especificamente para consumo público
* **Geral:** Dados comerciais que não se destinam a um público em geral
* **Confidencial:** Dados comerciais que possam causar danos à Microsoft se compartilhados em excesso
* **Altamente Confidencial:** Dados comerciais que podem causar danos à Microsoft se compartilhados em excesso

## <a name="tagging-data-classification-in-azure"></a>Marcação de classificação de dados no Azure

Cada provedor de nuvem deve oferecer um mecanismo para gravar metadados sobre qualquer ativo. Os metadados são vitais para o gerenciamento de ativos na nuvem. No caso do Azure, as marcas de recurso são a abordagem sugerida para o armazenamento de metadados. Para saber mais sobre marcação de recursos no Azure, consulte o artigo em [Uso de marcas para organizar os recursos do Azure](/azure/azure-resource-manager/resource-group-using-tags).

## <a name="next-steps"></a>Próximas etapas

Aplicar classificações de dados durante um dos percursos a governança acionáveis.

> [!div class="nextstepaction"]
> [Iniciar Percursos de governança acionáveis](../journeys/overview.md)
