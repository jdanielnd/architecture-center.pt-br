---
title: "OLTP (processamento de transações online)"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 07e7f680c8ee5e8589ff7cd2236ff95f6ee84f4c
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/14/2018
---
# <a name="online-transaction-processing-oltp"></a>OLTP (processamento de transações online)

O gerenciamento de [dados transacionais](../concepts/transactional-data.md) com sistemas de computador é conhecido como OLTP (Processamento de Transações Online). Os sistemas OLTP registram as interações de negócios conforme elas ocorrem na operação diária da organização e dão suporte à consulta desses dados para criar inferências.

![OLTP no Azure](./images/oltp-data-pipeline.png)

## <a name="when-to-use-this-solution"></a>Quando usar esta solução

Escolha o OLTP quando precisar processar e armazenar transações comerciais com eficiência, disponibilizando-as imediatamente para aplicativos cliente de uma maneira consistente. Use essa arquitetura quando um atraso tangível no processamento tiver um impacto negativo sobre as operações diárias da empresa.

Os sistemas OLTP foram projetados para processar e armazenar transações com eficiência, bem como consultar dados transacionais. A meta de processar e armazenar transações individuais com eficiência por um sistema OLTP é parcialmente realizada pela normalização de dados &mdash; ou seja, sua divisão em partes menores que são menos redundantes. Isso dá suporte à eficiência, porque permite que o sistema OLTP processe grandes quantidades de transações de forma independente e evita o processamento extra necessário para manter a integridade dos dados na presença de dados redundantes.

## <a name="challenges"></a>Desafios
A implementação e o uso de um sistema OLTP pode apresentar alguns desafios:

- Os sistemas OLTP nem sempre são bons para manipular agregações em grandes volumes de dados, embora haja exceções, como uma solução bem planejada baseada no SQL Server. A análise dos dados, que se baseia em cálculos de agregação em milhões de transações individuais, faz uso muito intensivo de recursos de um sistema OLTP. Eles podem ser lentos para serem executados e podem causar lentidão, bloqueando outras transações no banco de dados.
- Ao realizar a análise e os relatórios sobre dados altamente normalizados, as consultas tendem a ser complexas, pois a maioria das consultas precisa desnormalizar os dados usando junções. Além disso, as convenções de nomenclatura para objetos de banco de dados em sistemas OLTP tendem a ser concisas e sucintas. O aumento da normalização, juntamente com convenções de nomenclatura concisas, dificulta a consulta dos sistemas OLTP por parte dos usuários empresariais, sem a ajuda de um DBA ou desenvolvedor de dados.
- O armazenamento do histórico de transações por tempo indeterminado e o armazenamento de excesso de dados em uma tabela qualquer podem levar à redução do desempenho da consulta, dependendo da quantidade de transações armazenadas. A solução comum é manter uma janela de tempo relevante (como o ano fiscal atual) no sistema OLTP e descarregar dados históricos em outros sistemas, como um data mart ou [data warehouse](../technology-choices/data-warehouses.md).

## <a name="oltp-in-azure"></a>OLTP no Azure

Aplicativos, como sites hospedados em [Aplicativos Web do Serviço de Aplicativo](/azure/app-service/app-service-web-overview), APIs REST em execução no Serviço de Aplicativo ou aplicativos móveis ou da área de trabalho, se comunicam com o sistema OLTP, normalmente por meio de um intermediário da API REST.

Na prática, a maioria das cargas de trabalho não é puramente OLTP. Também há uma tendência de existir um [componente analítico](../scenarios/online-analytical-processing.md). Além disso, há uma demanda cada vez maior por relatórios em tempo real, como a execução de relatórios no sistema operacional. Isso também é chamado de HTAP (Processamento Transacional e Analítico Híbrido). Para obter mais informações, consulte [Armazenamentos de dados OLAP (Processamento Analítico Online)](../technology-choices/olap-data-stores.md).

## <a name="technology-choices"></a>Opções de tecnologia

Armazenamento de dados:

- [Banco de Dados SQL do Azure](/azure/sql-database/)
- [SQL Server em uma VM do Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Banco de Dados do Azure para MySQL](/azure/mysql/)
- [Banco de Dados do Azure para PostgreSQL](/azure/postgresql/)

Para obter mais informações, consulte [Escolhendo um armazenamento de dados OLTP](../technology-choices/oltp-data-stores.md)

Fontes de dados:

- [Serviço de aplicativo](/azure/app-service/)
- [Aplicativos Móveis](/azure/app-service-mobile/)

