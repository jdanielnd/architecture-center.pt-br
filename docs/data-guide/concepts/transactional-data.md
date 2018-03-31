---
title: Dados transacionais
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b7fdbb403d2a438ebc59e40ef58ed8067489dddc
ms.sourcegitcommit: 943e671a8d522cef5ddc8c6e04848134b03c2de4
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/05/2018
---
# <a name="transactional-data"></a>Dados transacionais

Dados transacionais são informações que acompanham as interações relacionadas às atividades de uma organização. Normalmente, essas interações são transações comerciais, como pagamentos recebidos de clientes, pagamentos feitos a fornecedores e movimentação de produtos pelo inventário, pedidos feitos ou serviços entregues. Eventos transacionais, que representam as transações em si, normalmente contêm uma dimensão temporal, alguns valores numéricos e referências a outros dados. 

As transações normalmente precisam ser *atômicas* e *consistentes*. Atomicidade significa que uma transação inteira sempre tem êxito ou falha como uma unidade de trabalho e nunca é deixada em um estado concluído incompleto. Se uma transação não pode ser concluída, o sistema de banco de dados precisa reverter todas as etapas que já foram realizadas como parte dessa transação. Em um RDBMS tradicional, essa reversão ocorre automaticamente se uma transação não pode ser concluída. Consistência significa que as transações sempre deixam os dados em um estado válido. (Essas são descrições muito informais de atomicidade e consistência. Há definições mais formais dessas propriedades, como [ACID](https://en.wikipedia.org/wiki/ACID).)

Bancos de dados transacionais podem dar suporte à coerência forte em transações que usam várias estratégias de bloqueio, como bloqueio pessimista, a fim de garantir que todos os dados sejam altamente consistentes dentro do contexto da empresa, para todos os usuários e processos. 

A arquitetura de implantação mais comum que usa dados transacionais é a camada de armazenamento de dados em uma arquitetura de três camadas. Uma arquitetura de três camadas normalmente consiste em uma camada de apresentação, uma camada de lógica de negócios e uma camada de armazenamento de dados. Uma arquitetura de implantação relacionada é a arquitetura [de N camadas](/azure/architecture/guide/architecture-styles/n-tier), que pode ter várias camadas intermediárias que manipulam a lógica de negócios.

![Exemplo de um aplicativo de três camadas](./images/three-tier-application.png)

## <a name="typical-traits-of-transactional-data"></a>Características comuns de dados transacionais

Os dados transacionais tendem a ter as seguintes características:

| Requisito | DESCRIÇÃO |
| --- | --- |
| Normalização | Altamente normalizado |
| Esquema | Esquema na gravação, altamente imposto|
| Consistência | Coerência forte, garantia de ACID |
| Integridade | Alta integridade |
| Usa transações | sim |
| Estratégia de bloqueio | Pessimista ou otimista|
| Atualizável | sim |
| Acrescentável | sim |
| Carga de trabalho | Gravações intensas, leituras moderadas |
| Indexação | Índices primários e secundários |
| Tamanho do dado | De pequeno a médio |
| Modelo | Relacional |
| Forma dos dados | Tabular |
| Flexibilidade de consulta | Altamente flexível |
| Escala | Pequeno (MBs) a grande (alguns TBs) | 

## <a name="see-also"></a>Veja também

[Processamento de Transações Online](../scenarios/online-transaction-processing.md)
