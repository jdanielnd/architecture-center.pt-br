---
title: Processando arquivos CSV e JSON
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 02e684d562cfe555f9e3596ad0a2f1a00d05c7a7
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/03/2018
ms.locfileid: "30298604"
---
# <a name="working-with-csv-and-json-files-for-data-solutions"></a>Trabalhando com arquivos CSV e JSON para soluções de dados

Provavelmente, CSV e JSON são os formatos mais comuns usados para ingestão, troca e armazenamento de dados não estruturados ou semiestruturados. 

## <a name="about-csv-format"></a>Sobre o formato CSV

Os arquivos CSV (valores separados por vírgula) são geralmente usados para troca de dados de tabela entre sistemas em texto sem formatação. Eles normalmente contêm uma linha de cabeçalho que fornece nomes de coluna para os dados, mas de outra forma, são considerados semiestruturados. Isso é devido ao fato de que os CSVs não podem representar dados hierárquicos ou relacionais naturalmente. As relações de dados costumam ser manipuladas com vários arquivos CSV, em que as chaves estrangeiras são armazenadas em colunas de um ou mais arquivos, mas as relações entre esses arquivos não são expressas no próprio formato. Arquivos no formato CSV podem usar outros delimitadores além de vírgulas, como tabulações ou espaços.

Apesar de suas limitações, os arquivos CSV são uma opção popular para a troca de dados, pois são compatíveis com uma ampla gama de aplicativos de negócios, de consumidor e científicos. Por exemplo, os programas de banco de dados e planilha podem importar e exportar arquivos CSV. Da mesma forma, a maioria dos mecanismos de processamento de dados em lote e de fluxo, como o Spark e o Hadoop, dá suporte nativo à serialização e à desserialização de arquivos formatados em CSV e oferece maneiras de aplicar um esquema no momento da leitura. Isso facilita o trabalho com os dados, oferecendo opções de consulta neles e repositório das informações em um formato de dados mais eficiente para um processamento mais rápido.

## <a name="about-json-format"></a>Sobre o formato JSON

Os dados JSON (JavaScript Object Notation) são representados como pares chave-valor em um formato semiestruturado. O JSON costuma ser comparado com o XML, pois ambos podem armazenar dados em formato hierárquico, com os dados filho representados embutidos com seu pai. Ambos são autodescritivos e legíveis por humanos, mas os documentos JSON tendem a ser muito menores, resultando em seu uso popular na troca de dados online, especialmente com o advento de serviços Web baseados em REST. 

Os arquivos formatados em JSON apresentam várias vantagens em relação ao CSV:

* O JSON mantém estruturas hierárquicas, facilitando a retenção de dados relacionados em um único documento e a representação de relações complexas.
* A maioria das linguagens de programação fornece suporte nativo para desserialização de JSON em objetos ou fornece bibliotecas de serialização JSON leves.
* O JSON dá suporte a listas de objetos, ajudando a prevenir conversões confusas de listas em um modelo de dados relacional.
* O JSON é um formato de arquivo geralmente usado para bancos de dados NoSQL, como o MongoDB, Couchbase e Azure Cosmos DB.

Como muitos dados recebidos pela conexão já estão no formato JSON, a maioria das linguagens de programação baseadas na Web dá suporte ao trabalho nativo com o JSON ou por meio do uso de bibliotecas externas para serializar e desserializar dados JSON. Esse suporte universal para o JSON levou a seu uso em formatos lógicos por meio de representação da estrutura de dados, formatos de troca para dados quentes e armazenamento de dados para dados frios.

Muitos mecanismos de processamento de dados em lote e de fluxo dão suporte nativo à serialização e à desserialização de JSON. Embora os dados contidos em documentos JSON possam, em última análise, ser armazenados em formatos mais otimizados para o desempenho, como o Parquet ou o Avro, eles servem como os dados brutos para a fonte de verdade, o que é crítico para o reprocessamento dos dados, conforme necessário.

## <a name="when-to-use-csv-or-json-formats"></a>Quando usar formatos CSV ou JSON

CSVs costumam ser mais usados para exportação e importação de dados ou processamento de dados para análise e aprendizado de máquina. Os arquivos formatados em JSON trazem os mesmos benefícios, mas são mais comuns em soluções de troca de dados quentes. Os documentos JSON costumam ser enviados por dispositivos da Web e móveis que executam transações online, por dispositivos de IoT (Internet das Coisas) para e comunicação unidirecional ou bidirecional ou por aplicativos cliente que se comunicam com serviços de SaaS e PaaS ou arquiteturas sem servidor. 

Os formatos de arquivo CSV e JSON facilitam a troca de dados entre sistemas ou dispositivos diferentes. Os formatos semiestruturados permitem flexibilidade durante a transferência de praticamente qualquer tipo de dados e o suporte universal para esses formatos simplifica seu trabalho com eles. Ambos podem ser usados como a origem bruta de verdade nos casos em que os dados processados são armazenados em formatos binários para uma consulta mais eficiente. 

## <a name="working-with-csv-and-json-data-in-azure"></a>Trabalhando com os dados CSV e JSON no Azure

O Azure fornece várias soluções para trabalhar com arquivos CSV e JSON, dependendo de suas necessidades. O local principal de aterrissagem desses arquivos é o Armazenamento do Azure ou o Azure Data Lake Store. A maioria dos serviços do Azure que trabalha com esses e outros arquivos baseados em texto é integrada a um desses serviços de armazenamento de objetos. No entanto, em algumas situações, você pode optar por importar os dados diretamente para o SQL do Azure ou outro armazenamento de dados. O SQL Server tem suporte nativo para armazenar e trabalhar com documentos JSON, o que facilita [a importação e o processo desses tipos de arquivos](/sql/relational-databases/json/import-json-documents-into-sql-server). Você pode usar um utilitário como a Importação em Massa do SQL para [importar arquivos CSV](/sql/relational-databases/json/import-json-documents-into-sql-server) com facilidade.

Dependendo do cenário, você pode executar o [processamento em lotes](../big-data/batch-processing.md) ou o [processamento em tempo real](../big-data/real-time-processing.md) dos dados.

## <a name="challenges"></a>Desafios

Há alguns desafios a serem considerados ao trabalhar com esses formatos:

* Sem nenhuma restrição no modelo de dados, os arquivos CSV e JSON são propensos a dados corrompidos ("entrada e saída de lixo"). Por exemplo, não há nenhuma noção de um objeto de data/hora em nenhum desses arquivos, de modo que o formato de arquivo não impede que você insira "ABC123" em um campo de data, por exemplo.

* O uso de arquivos CSV e JSON como a solução de armazenamento frio não se adapta bem ao trabalhar com Big Data. Na maioria dos casos, eles não podem ser divididos em partições para processamento paralelo e não podem ser compactados tão bem quanto formatos binários. Isso geralmente leva ao processamento e armazenamento desses dados em formatos otimizados para leitura, como o Parquet e ORC (colunar de linha otimizado), que também fornece índices e estatísticas embutidas sobre os dados contidos.

* Talvez seja necessário aplicar um esquema nos dados semiestruturados para facilitar a consulta e análise. Normalmente, isso exige o armazenamento dos dados em outro formato que atende às necessidades do armazenamento de dados do ambiente, como em um banco de dados.

