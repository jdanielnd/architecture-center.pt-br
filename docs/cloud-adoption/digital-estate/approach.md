---
title: Abordagens para planejamento de propriedade digital
titleSuffix: Enterprise Cloud Adoption
description: Descreve algumas abordagens ao planejamento de propriedade digital
author: BrianBlanchard
ms.date: 12/10/2018
ms.openlocfilehash: 5803447ff87e733aa5a9c24ac626bba665b8394a
ms.sourcegitcommit: e7f8676bbffe500fc4d6deb603b7c0b7ba1884a6
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/10/2018
ms.locfileid: "53179588"
---
# <a name="enterprise-cloud-adoption-approaches-to-digital-estate-planning"></a>Adoção da Nuvem Empresarial: Abordagens para planejamento de propriedade digital

O planejamento de propriedade digital pode assumir diversas formas, dependendo dos resultados desejados e do tamanho da propriedade existente. Também há um várias opções relacionadas à abordagem usada. É importante definir as expectativas com relação à abordagem logo no início dos ciclos de planejamento. Expectativas incertas geralmente resultam em atrasos associados a exercícios adicionais de coleta de inventário. Este artigo descreve três abordagens para análise.

## <a name="workload-driven-approach"></a>Abordagem orientada por carga de trabalho

A abordagem de avaliação de cima para baixo avalia os aspectos de segurança, como a categorização de dados (alto, médio ou baixo impacto nos negócios), conformidade, soberania e requisitos de risco de segurança. Em seguida, essa abordagem avalia a complexidade da arquitetura de alto nível, avaliando aspectos como autenticação, estrutura de dados, requisitos de latência, dependências e expectativa de vida do aplicativo. Em seguida, a abordagem de cima para baixo mede os requisitos operacionais do aplicativo, como os níveis de serviço, integração, janelas de manutenção, monitoramento e análise. Após a análise e consideração de todos esses aspectos, o resultado é uma pontuação que reflete a dificuldade relativa para migrar esse aplicativo para cada uma das plataformas de nuvem: IaaS, PaaS e SaaS.

Em segundo lugar, a avaliação de cima para baixo avalia os benefícios financeiros do aplicativo, como a eficiência operacional, TCO, retorno sobre investimento ou quaisquer outras métricas financeiras apropriadas. Além disso, a avaliação também examina a periodicidade do aplicativo (se há períodos do ano com picos de demanda) e a carga de computação geral. Além disso, ele examina os tipos de usuários aos quais dá suporte (casual/especialista, sempre/ocasionalmente conectado), e, consequentemente, a escalabilidade e a elasticidade necessárias. Por fim, a avaliação termina examinando a continuidade dos negócios e os requisitos de resiliência do aplicativo, bem como as dependências de execução do aplicativo, se ocorrer uma interrupção do serviço.

> [!TIP]
> Essa abordagem exige entrevistas e comentários informais dos participantes técnicos e empresariais. A disponibilidade de indivíduos chave é o maior risco à pontualidade. A natureza informal das fontes de dados dificulta ainda mais a produção de estimativas de custo ou de tempo. Planeje com antecedência o agendamento e valide todos os dados coletados.

## <a name="asset-driven-approach"></a>Abordagem orientada por ativos

A abordagem orientada por ativos fornece um plano com base nos ativos que dão suporte a um aplicativo para migrar. Nessa abordagem, os dados estatísticos de uso são extraídos de um CMDB (Banco de dados de gerenciamento de configuração) ou de outras ferramentas de avaliação de infraestrutura. Geralmente, essa abordagem pressupõe um modelo de implantação IaaS como linha de base. Nesse processo, a análise avalia os atributos de cada ativo: memória, número de processadores (núcleos de CPU), espaço de armazenamento do sistema operacional, unidades de dados, NICs (adaptadores de rede), IPv6, balanceamento de carga de rede, clustering, versão do sistema operacional, versão do banco de dados (se precisar), os domínios com suporte e os componentes de terceiros ou pacotes de software, entre outros. Depois, os ativos em inventário nessa abordagem são alinhados com as cargas de trabalho ou aplicativos para fins de mapeamento de dependência e agrupamento.

> [!TIP]
> Essa abordagem exige uma fonte rica de dados de uso em estatística. O tempo para verificar o inventário e coletar dados é o maior risco à pontualidade. As fontes de dados de baixo nível podem perder dependências entre os aplicativos ou ativos. Planeje durante pelo menos um mês a verificação do inventário. Valide as dependências antes da implantação.

## <a name="incremental-approach"></a>Abordagem incremental

Assim como grande parte da estrutura de Adoção de Nuvem Empresarial, sugerimos uma abordagem incremental. No caso do planejamento de propriedade digital, isso é igual a um processo com várias fases, como mostramos a seguir:

- Análise de custo inicial: Se a validação financeira for necessária, comece com uma abordagem controlada por ativos, descrita acima, para obter um cálculo de custo inicial de toda a propriedade digital, sem racionalização. Isso estabelece um parâmetro de comparação de pior cenário possível.

- Planejamento de migração: Após a atribuição de equipe de Estratégia de Nuvem, crie uma lista de pendências de migração inicial usando uma abordagem orientada por carga de trabalho, com base apenas no conhecimento coletivo e nas entrevistas limitadas dos interessados. Essa abordagem cria rapidamente uma avaliação de carga de trabalho leve para promoção da colaboração.

- Planejamento de lançamento: Em cada lançamento, a lista de pendências de migração é removida e priorizada novamente para se concentrar no impacto comercial mais relevante. Durante esse processo, as 5&ndash;10 cargas de trabalho seguintes seriam selecionadas como versões priorizadas. Neste ponto, a equipe de Estratégia de Nuvem investiria tempo na conclusão de uma abordagem completa orientada por carga de trabalho. Atrasar essa avaliação até o alinhamento da versão respeita o tempo das partes interessadas. Também atrasa o investimento em análise completa até que a empresa comece a ver resultados dos esforços anteriores.

- Análise de execução: Antes da migração, modernização ou replicação de qualquer ativo, o ativo deve ser avaliado individualmente e como parte de uma versão coletiva. Neste ponto, os dados da abordagem inicial orientada por ativos podem ser examinados a fim de garantir o dimensionamento preciso e restrições operacionais.

> [!TIP]
> Essa abordagem incremental permite o planejamento simplificado e resultados mais rápidos. É muito importante que todas as partes envolvidas entendam a abordagem à tomada de decisão atrasada. É igualmente importante que as suposições feitas em cada estágio sejam documentadas para evitar a perda de detalhes.

## <a name="next-steps"></a>Próximas etapas

Após a seleção de uma abordagem, o inventário pode ser coletado.

> [!div class="nextstepaction"]
> [Coletar dados de inventário](inventory.md)