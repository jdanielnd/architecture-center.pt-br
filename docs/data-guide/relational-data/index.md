---
title: Dados relacionais
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: d68bddcb75e5c8f786a7739e85de2645a2c3d641
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54484917"
---
# <a name="traditional-relational-database-solutions"></a>Soluções tradicionais de banco de dados relacional

Dados relacionais são dados modelados com o modelo relacional. Nesse modelo, os dados são expressos como tuplas. Uma *tupla* é um conjunto de pares de atributo/valor. Por exemplo, uma tupla pode ser (itemid = 5, orderid = 1, item = "Chair", amount = 200.00). Um conjunto de tuplas que compartilham os mesmos atributos é chamado de uma *relação*.

As relações são naturalmente representadas como tabelas, em que cada tupla é exposta como uma linha na tabela. No entanto, as linhas têm uma ordenação explícita, ao contrário das tuplas. O esquema de banco de dados define as colunas (cabeçalhos) de cada tabela. Cada coluna é definida com um nome e um tipo de dados para todos os valores armazenados nessa coluna em todas as linhas na tabela.

![Exemplo que mostra os dados usando um banco de dados relacional](../images/example-relational.png)

Um armazenamento de dados que organiza dados usando o modelo relacional é chamado de um banco de dados relacional. As chaves primárias identificam com exclusividade as linhas dentro de uma tabela. Os campos de chave estrangeira são usados em uma tabela para se referir a uma linha em outra tabela referenciando a chave primária da outra tabela. Chaves estrangeiras são usadas para manter a integridade referencial, garantindo que as linhas referenciadas não sejam alteradas nem excluídas enquanto a linha de referência depender dela.

![Exemplo que mostra os dados usando um banco de dados relacional](../images/example-relational2.png)

Os bancos de dados relacionais dão suporte a vários tipos de restrições que ajudam a garantir a integridade dos dados:

- As restrições exclusivas garantem que todos os valores em uma coluna sejam exclusivos.

- As restrições de chave estrangeira impõem um vínculo entre os dados nas duas tabelas. Uma chave estrangeira referencia a chave primária ou outra chave exclusiva de outra tabela. Uma restrição de chave estrangeira impõe a integridade referencial, não permitindo alterações que gerem valores inválidos de chave estrangeira.

- As restrições de verificação, também conhecidas como restrições de integridade de entidade, limitam os valores que podem ser armazenados em uma única coluna ou em relação aos valores em outras colunas da mesma linha.

A maioria dos bancos de dados relacionais usa a linguagem SQL que permite uma abordagem declarativa para a consulta. A consulta descreve o resultado desejado, mas não as etapas para executar a consulta. O mecanismo então decide a melhor maneira de executar a consulta. Isso é diferente de uma abordagem de procedimento, em que o programa de consulta especifica as etapas de processamento de forma explícita. No entanto, os bancos de dados relacionais podem armazenar rotinas de código executável na forma de procedimentos armazenados e funções, o que permite uma combinação de abordagens declarativas e de procedimento.

Para melhorar o desempenho da consulta, os bancos de dados relacionais usam *índices*. Os índices primários, que são usados pela chave primária, definem a ordem dos dados quando eles são recebidos no disco. Os índices secundários fornecem uma combinação alternativa de campos, de modo que as linhas desejadas possam ser consultadas com eficiência, sem a necessidade de classificar novamente todos os dados no disco.

Como os bancos de dados relacionais impõem a integridade referencial, o dimensionamento de um banco de dados relacional pode se tornar um desafio. Isso ocorre porque qualquer operação de consulta ou inserção pode abranger qualquer número de tabelas. Você pode escalar horizontalmente um banco de dados relacional com a *fragmentação* dos dados, mas isso exige um design cuidadoso do esquema. Para obter mais informações, consulte [Padrão de fragmentação](../../patterns/sharding.md).

Se os dados não são relacionais ou têm requisitos que não são adequados para um banco de dados relacional, considere o uso de um armazenamento de dados [não relacional ou NoSQL](../big-data/non-relational-data.md).
