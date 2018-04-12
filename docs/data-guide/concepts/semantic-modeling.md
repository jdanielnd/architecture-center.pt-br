---
title: Modelagem semântica
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 343d17af0d933d515c724a062237c8d5df3a9e31
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/31/2018
---
# <a name="semantic-modeling"></a>Modelagem semântica

Um modelo de dados semântico é um modelo conceitual que descreve o significado dos elementos de dados que ele contém. As empresas costumam ter seus próprios termos para coisas, às vezes, com sinônimos ou até mesmo significados diferentes para o mesmo termo. Por exemplo, um banco de dados de inventário pode acompanhar uma parte do equipamento com uma ID de ativo e um número de série, mas um banco de dados de vendas pode se referir ao número de série como a ID de ativo. Não há nenhuma maneira simples para relacionar esses valores sem um modelo que descreve a relação. 

A modelagem semântica fornece um nível de abstração no esquema de banco de dados, de modo que os usuários não precisem conhecer as estruturas de dados subjacentes. Isso facilita para os usuários finais consultar dados sem executar agregações e junções no esquema subjacente. Além disso, normalmente, as colunas são renomeadas com nomes mais amigáveis, de modo que o contexto e o significado dos dados sejam mais óbvios.

A modelagem semântica é predominantemente usada para cenários de leitura intensa, como análise e business intelligence (OLAP), ao contrário do processamento de dados transacionais de gravação mais intensa (OLTP). Isso é principalmente devido à natureza de uma camada semântica típica:

- Os comportamentos de agregação são definidos para que as ferramentas de relatórios os exibam corretamente.
- A lógica de negócios e os cálculos são definidos.
- Os cálculos orientados por tempo são incluídos.
- Os dados normalmente são integrados de várias fontes. 

Tradicionalmente, a camada semântica é colocada em um data warehouse por esses motivos.

![Diagrama de exemplo de uma camada semântica entre um data warehouse e uma ferramenta de relatórios](./images/semantic-modeling.png)

Há dois tipos principais de modelos semânticos:

* **Tabular**. Usa constructos de modelagem relacional (modelo, tabelas, colunas). Internamente, os metadados são herdados de constructos de modelagem OLAP (cubos, dimensões, medidas). O código e o script usam metadados OLAP.
* **Multidimensional**. Usa constructos de modelagem OLAP tradicionais (cubos, dimensões, medidas).

Serviço do Azure relevante:
- [Azure Analysis Services](https://azure.microsoft.com/services/analysis-services/)

## <a name="example-use-case"></a>Caso de uso de exemplo

Uma organização tem dados armazenados em um banco de dados grande. Ela deseja disponibilizar esses dados para usuários empresariais e clientes para criar seus próprios relatórios e fazer análises. Uma opção é apenas fornecer a esses usuários o acesso direto ao banco de dados. No entanto, há várias desvantagens ao fazer isso, incluindo o gerenciamento de segurança e controle de acesso. Além disso, o design do banco de dados, incluindo os nomes de tabelas e colunas, pode ser de difícil compreensão para um usuário. Os usuários precisam saber quais tabelas consultar, como as tabelas devem ser unidas e outra lógica de negócios que precisa ser aplicada para obter os resultados corretos. Os usuários também precisam conhecer uma linguagem de consulta como SQL, até mesmo para começar. Normalmente, isso leva ao relatório de vários usuários das mesmas métricas, mas com resultados diferentes.

Outra opção é encapsular todas as informações de que os usuários precisam em um modelo semântico. O modelo semântico pode ser consultado com mais facilidade por usuários com uma ferramenta de relatórios de sua escolha. Os dados fornecidos pelo modelo semântico são extraídos de um data warehouse, garantindo que todos os usuários vejam uma única versão da verdade. O modelo semântico também fornece nomes de tabela e coluna amigáveis, relações entre tabelas, descrições, cálculos e segurança em nível de linha.

## <a name="typical-traits-of-semantic-modeling"></a>Características comuns da modelagem semântica

A modelagem semântica e o processamento analítico tendem a ter as seguintes características:

| Requisito | DESCRIÇÃO |
| --- | --- |
| Esquema | Esquema na gravação, altamente imposto|
| Usa Transações | Não  |
| Estratégia de Bloqueio | Nenhum |
| Atualizável | Não (normalmente exige o recálculo de cubo) |
| Acrescentável | Não (normalmente exige o recálculo de cubo) |
| Carga de trabalho | Leituras intensas, somente leitura |
| Indexação | Indexação multidimensional |
| Tamanho do dado | De pequeno a médio |
| Modelo | Multidimensional |
| Forma dos dados:| Esquema de cubo ou estrela/floco de neve |
| Flexibilidade de consulta | Altamente flexível |
| Escala: | Grande (dezenas a centenas de GBs) |

## <a name="see-also"></a>Consulte também

- [Data warehouse](../scenarios/data-warehousing.md)
- [OLAP (processamento analítico online)](../scenarios/online-analytical-processing.md)
