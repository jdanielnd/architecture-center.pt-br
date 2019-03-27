---
title: Escolhendo uma tecnologia de serviços cognitivos
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: AI
ms.openlocfilehash: e8bbdf4874d5fc669e1c08add8cf212217d176a4
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58246207"
---
# <a name="choosing-a-microsoft-cognitive-services-technology"></a>Escolhendo uma tecnologia de serviços cognitivos da Microsoft

Os serviços cognitivos da Microsoft são APIs baseadas em nuvem que você pode usar em aplicativos e fluxos de dados de IA (inteligência artificial). Eles fornecem modelos pré-treinados prontos para uso em seu aplicativo, que não exigem dados nem treinamento de modelo de sua parte. Os serviços cognitivos são desenvolvidos pela equipe de IA e Pesquisa da Microsoft e utilizam os últimos algoritmos de aprendizado profundo. Eles são consumidos em interfaces HTTP REST. Além disso, os SDKs estão disponíveis para muitas estruturas comuns de desenvolvimento de aplicativos.

Os serviços cognitivos incluem:

- Análise de texto
- Visual computacional
- Análise de vídeos
- Geração e reconhecimento de fala
- Compreensão de idioma natural
- Pesquisa inteligente

Principais benefícios:

- Esforço de desenvolvimento mínimo para serviços modernos de IA.
- Fácil integração com aplicativos por meio de interfaces HTTP REST.
- Suporte interno para consumo de serviços cognitivos no Azure Data Lake Analytics.

Considerações:

- Disponível somente na Web. A conectividade com a Internet é geralmente necessária. Uma exceção é o Serviço de Visão Personalizada, cujo modelo treinado pode ser exportado para previsão em dispositivos e no IoT Edge.

- Embora haja suporte para uma personalização considerável, os serviços disponíveis podem não atender a todos os requisitos de análise preditiva.

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-when-choosing-amongst-the-cognitive-services"></a>Quais são as opções disponíveis ao escolher entre os serviços cognitivos?

<!-- markdownlint-disable MD026 -->

No Azure, há dezenas de Serviços Cognitivos disponíveis. A lista atual deles está disponível em um diretório categorizado pela área funcional para a qual eles dão suporte:

- [Visão](https://azure.microsoft.com/services/cognitive-services/directory/vision/)
- [Fala](https://azure.microsoft.com/services/cognitive-services/directory/speech/)
- [Conhecimento](https://azure.microsoft.com/services/cognitive-services/directory/know/)
- [Search](https://azure.microsoft.com/services/cognitive-services/directory/search/)
- [Linguagem](https://azure.microsoft.com/services/cognitive-services/directory/lang/)

## <a name="key-selection-criteria"></a>Principais Critérios de Seleção

Para restringir as opções, comece respondendo a estas perguntas:

- Com qual tipo de dados você está lidando? Restrinja suas opções com base no tipo de dados de entrada com o qual você está trabalhando. Por exemplo, se a entrada for texto, selecione um serviço que tem um tipo de entrada de texto.

- Você tem os dados para treinar um modelo? Em caso afirmativo, considere os serviços personalizados que permitem que você treine seus modelos subjacentes com os dados fornecidos, para melhor precisão e desempenho.

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades.

### <a name="uses-prebuilt-models"></a>Usa modelos predefinidos

|                                                   |             Tipo de entrada              |                                                                                Principal benefício                                                                                |
|---------------------------------------------------|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                API de Análise de Texto                 |                Texto                 |                                                       Avalie sentimento e tópicos para entender o que os usuários querem.                                                        |
|                API de Vinculação de Entidade                 |                Texto                 |                                               Fortaleça os links de dados do aplicativo com desambiguidade e reconhecimento de entidade nomeada.                                               |
| Serviço Inteligente de Reconhecimento Vocal (LUIS) |                Texto                 |                                                          Ensine seus aplicativos a entenderem os comandos dos usuários.                                                          |
|                 Serviço QnA Maker                 |                Texto                 |                                             Extraia informações formatadas em perguntas frequentes em respostas com formato de conversação e de fácil navegação.                                              |
|              API de Análise Linguística              |                Texto                 |                                                            Simplifique conceitos de linguagem complexos e analise o texto.                                                             |
|           Serviço de Exploração de Conhecimento           |                Texto                 |                                          Habilite experiências de pesquisa interativa em dados estruturados por meio de entradas de idioma natural.                                          |
|              API de Modelo de Linguagem da Web               |                Texto                 |                                                         Use os modelos de linguagem preditiva treinados em dados na escala da Web.                                                         |
|              API de Conhecimento Acadêmico               |                Texto                 |                                        Alcance a riqueza do conteúdo acadêmico no Microsoft Academic Graph populado pelo Bing.                                         |
|               API de Sugestão Automática do Bing                |                Texto                 |                                                        Dê ao seu aplicativo opções inteligentes de sugestão automática para pesquisas.                                                        |
|               API de Verificação Ortográfica do Bing                |                Texto                 |                                                             Detecte e corrija erros de ortografia em seu aplicativo.                                                             |
|                API de Tradução de Texto                |                Texto                 |                                                                           Tradução automática.                                                                            |
|                API de Recomendações                |                Texto                 |                                                             Preveja e recomende itens que seus clientes querem.                                                              |
|              API de Pesquisa de Entidade do Bing               |       Texto (consulta da pesquisa na Web)       |                                                           Identifique e aumente as informações de entidade da Web.                                                           |
|               API de Pesquisa de Imagem do Bing               |       Texto (consulta da pesquisa na Web)       |                                                                            Pesquise imagens.                                                                             |
|               API de Pesquisa de Notícias do Bing                |       Texto (consulta da pesquisa na Web)       |                                                                             Pesquise notícias.                                                                              |
|               API de Pesquisa de Vídeo do Bing               |       Texto (consulta da pesquisa na Web)       |                                                                            Pesquise vídeos.                                                                             |
|                API de Pesquisa na Web do Bing                |       Texto (consulta da pesquisa na Web)       |                                                        Obtenha detalhes de pesquisa aprimorados de bilhões de documentos da Web.                                                        |
|                  API de Fala do Bing                  |           Texto ou fala            |                                                                  Converta fala em texto e vice-versa.                                                                   |
|              API de Reconhecimento do Locutor              |               Fala                |                                                       Use a fala para identificar e autenticar locutores individuais.                                                        |
|               API de Tradução de Fala               |               Fala                |                                                                   Realize a tradução de fala em tempo real.                                                                   |
|                API da Pesquisa Visual Computacional                |    Imagens (ou quadros de vídeo)    | Extraia informações acionáveis de imagens, crie descrição de fotos automaticamente, obtenha marcas, reconheça celebridades, extraia texto e crie miniaturas precisas. |
|                 Content Moderator                 |        Texto, imagens ou vídeo        |                                                               Moderação automatizada de imagem, texto e vídeo.                                                                |
|                    API de Detecção de Emoções                    | Imagens (fotos com sujeitos humanos) |                                                              Identifique a variação de emoções de sujeitos humanos.                                                               |
|                     API de Detecção Facial                      | Imagens (fotos com sujeitos humanos) |                                                       Detecte, identifique, analise, organize e marque rostos em fotos.                                                       |
|                   Video Indexer                   |                Vídeo                |                        Insights de vídeos como sentimento, transcrição de fala, tradução de fala, reconhecimento de rostos e emoções e extração de palavras-chave.                         |

### <a name="trained-with-custom-data-you-provide"></a>Treinado com os dados personalizados fornecidos

| | Tipo de entrada | Principal benefício |
| --- | --- | --- |
| Serviço de Visão Personalizada | Imagens (ou quadros de vídeo) | Personalize seus próprios modelos de visão computacional. |
| Serviço de Fala Personalizado | Fala | Supere as barreiras para o reconhecimento de fala como estilo de fala, ruído de fundo e vocabulário. |
| Serviço Personalizado de Decisão | Conteúdo da Web (por exemplo, RSS feed) | Use o aprendizado de máquina para selecionar automaticamente o conteúdo apropriado para a home page |
| API de Pesquisa Personalizada do Bing | Texto (consulta da pesquisa na Web) | Ferramenta de pesquisa de nível comercial. |
