---
title: Escolhendo uma tecnologia de análise de dados e relatórios
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 05e33a3da0933036a604d2bc4cc5a20ae70fe772
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428305"
---
# <a name="choosing-a-data-analytics-technology-in-azure"></a>Escolhendo uma tecnologia de análise de dados no Azure

A meta da maioria das soluções de Big Data é gerar insights sobre os dados por meio de análise e relatórios. Isso pode incluir visualizações e relatórios pré-configurados ou a exploração interativa de dados. 

## <a name="what-are-your-options-when-choosing-a-data-analytics-technology"></a>Quais são as opções disponíveis ao escolher uma tecnologia de análise de dados?

Há várias opções para análise, visualizações e relatórios no Azure, dependendo de suas necessidades:

- [Power BI](/power-bi/)
- [Jupyter Notebooks](https://jupyter.readthedocs.io/en/latest/index.html)
- [Zeppelin Notebooks](https://zeppelin.apache.org/)
- [Microsoft Azure Notebooks](https://notebooks.azure.com/)

### <a name="power-bi"></a>Power BI

O [Power BI](/power-bi/) é um pacote de ferramentas de análise de negócios. Ele pode se conectar a centenas de fontes de dados e pode ser usado para análise ad hoc. Veja [esta lista](/power-bi/desktop-data-sources) das fontes de dados disponíveis no momento. Use o [Power BI Embedded](https://azure.microsoft.com/services/power-bi-embedded/) para integrar o Power BI a seus próprios aplicativos, sem a necessidade de licença adicional.

As organizações podem usar o Power BI para gerar relatórios e publicá-los na organização. Todos podem criar dashboards personalizados, com governança e [segurança internas](/power-bi/service-admin-power-bi-security). O Power BI usa o [Azure AD](/azure/active-directory/) (Azure Active Directory) para autenticar os usuários que fazem logon no serviço do Power BI e usa as credenciais de logon do Power BI sempre que um usuário tenta acessar recursos que exigem autenticação.

### <a name="jupyter-notebooks"></a>Notebooks Jupyter 

O [Jupyter Notebooks](https://jupyter.readthedocs.io/en/latest/index.html) fornece um shell baseado em navegador que possibilita aos cientistas de dados criar arquivos de *bloco de anotações* que contêm o código e texto markdown do Python, Scala ou R, tornando-o uma maneira eficiente para colaborar com o compartilhamento e a documentação de código e resultados em um único documento.

A maioria das variedades de clusters HDInsight, como o Spark ou Hadoop, vem [pré-configurada com blocos de anotações do Jupyter](/azure/hdinsight/spark/apache-spark-jupyter-notebook-kernels) para interagir com os dados e enviar trabalhos para processamento. Dependendo do tipo de cluster HDInsight que está sendo usado, um ou mais kernels serão fornecidos para interpretar e executar o código. Por exemplo, os clusters Spark no HDInsight fornecem kernels relacionados ao Spark que você pode selecionar para executar o código do Python ou Scala usando o mecanismo do Spark.

Os blocos de anotações do Jupyter fornecem um ambiente excelente para análise, visualização e processamento dos dados antes da criação de visualizações mais avançadas com uma ferramenta de relatórios/BI como o Power BI.

### <a name="zeppelin-notebooks"></a>Zeppelin Notebooks

O [Zeppelin Notebooks](https://zeppelin.apache.org/) é outra opção de shell baseado em navegador, semelhante ao Jupyter em funcionalidade. Alguns clusters HDInsight vêm [pré-configurados com blocos de anotações do Zeppelin](/azure/hdinsight/spark/apache-spark-zeppelin-notebook). No entanto, caso você esteja usando um cluster de [Consulta Interativa do HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started) (Hive LLAP), atualmente, o [Zeppelin](/azure/hdinsight/hdinsight-connect-hive-zeppelin) será a única opção de bloco de anotações que você poderá usar para executar consultas interativas do Hive. Além disso, se você estiver usando um [cluster HDInsight ingressado no domínio](/azure/hdinsight/domain-joined/apache-domain-joined-introduction), os blocos de anotações do Zeppelin serão o único tipo que permite atribuir logons de usuário diferentes para controlar o acesso aos blocos de anotações e às tabelas subjacentes do Hive.

### <a name="microsoft-azure-notebooks"></a>Microsoft Azure Notebooks

O [Azure Notebooks](https://notebooks.azure.com/) é um serviço online baseado no Jupyter Notebooks que possibilita aos cientistas de dados criar, executar e compartilhar Jupyter Notebooks em bibliotecas baseadas em nuvem. O Azure Notebooks fornece ambientes de execução para o Python 2, Python 3, F# e R, além de fornecer várias bibliotecas de gráficos para visualização dos dados, como ggplot, matplotlib, bokeh e seaborn.

Ao contrário dos blocos de anotações do Jupyter em execução em um cluster HDInsight, que estão conectados à conta de armazenamento padrão do cluster, o Azure Notebooks não fornece dados. É necessário [carregar os dados](https://notebooks.azure.com/Microsoft/libraries/samples/html/Getting%20to%20your%20Data%20in%20Azure%20Notebooks.ipynb) de várias maneiras, como baixar os dados em uma fonte online, interagir com o Armazenamento de Blobs ou de Tabelas do Azure, conectar-se a um banco de dados SQL ou carregar os dados com o Assistente de Cópia do Azure Data Factory.

Principais benefícios:

* Serviço gratuito &mdash; nenhuma assinatura do Azure é necessária.
* Não há necessidade de instalar o Jupyter e as distribuições do R ou Python com suporte localmente &mdash; basta usar um navegador.
* Gerencie suas próprias bibliotecas online e acesse-as em qualquer dispositivo.
* Compartilhe seus blocos de anotações com colaboradores.

Considerações:

* Você não poderá acessar os blocos de anotações quando estiver offline.
* As funcionalidades de processamento limitadas do serviço gratuito de bloco de anotações podem não ser suficientes para treinar modelos grandes ou complexos.

## <a name="key-selection-criteria"></a>Principais critérios de seleção

Para restringir as opções, comece respondendo a estas perguntas:

- Você precisa se conectar a várias fontes de dados, fornecendo um local centralizado para criar relatórios de dados distribuídos em todo o domínio? Nesse caso, escolha uma opção que permite que você se conecte a centenas de fontes de dados.

- Você deseja inserir visualizações dinâmicas em um site ou aplicativo externo? Nesse caso, escolha uma opção que fornece funcionalidades de inserção.

- Você deseja criar visualizações e relatórios enquanto estiver offline? Em caso afirmativo, escolha uma opção com funcionalidades offline.

- Você precisa ter um poder de processamento intenso para treinar modelos de IA grandes ou complexos ou trabalhar com conjuntos grandes de dados? Em caso afirmativo, escolha uma opção que pode se conectar a um cluster de Big Data.

## <a name="capability-matrix"></a>Matriz de funcionalidades

As tabelas a seguir resumem as principais diferenças em funcionalidades. 

### <a name="general-capabilities"></a>Funcionalidades gerais

| | Power BI | Notebooks Jupyter | Zeppelin Notebooks | Microsoft Azure Notebooks |
| --- | --- | --- | --- | --- |
| Conectar-se a um cluster de Big Data para processamento avançado | SIM | sim | sim | Não  |
| Serviço gerenciado | SIM | Sim <sup>1</sup> | Sim <sup>1</sup> | SIM |
| Conectar-se a centenas de fontes de dados | SIM | Não | Não | Não  |
| Funcionalidades offline | Sim <sup>2</sup> | Não  | Não | Não  |
| Funcionalidades de inserção | SIM | Não | Não | Não  |
| Atualização automática de dados | SIM | Não | Não | Não  |
| Acesso a vários pacotes de software livre | Não  | Sim <sup>3</sup> | Sim <sup>3</sup> | Sim <sup>4</sup> |
| Opções de transformação/limpeza de dados | [Power Query](https://powerbi.microsoft.com/blog/getting-started-with-power-query-part-i/), R | 40 linguagens, incluindo Python, R, Julia e Scala | Mais de 20 interpretadores, incluindo Python, JDBC e R | Python, F#, R |
| Preços | Gratuito para o Power BI Desktop (criação); consulte [Preços](https://powerbi.microsoft.com/pricing/) para obter as opções de hospedagem | Grátis | Grátis | Grátis |
| Colaboração de multiusuário | [Sim](/power-bi/service-how-to-collaborate-distribute-dashboards-reports) | Sim (por meio de compartilhamento ou com um servidor de multiusuário como o [JupyterHub](https://github.com/jupyterhub/jupyterhub)) | SIM | Sim (por meio de compartilhamento) |

[1] Quando usado como parte de um cluster HDInsight gerenciado.

[2] Com o uso do Power BI Desktop.

[2] Pesquise o [repositório do Maven](https://search.maven.org/) para obter pacotes contribuídos pela comunidade.

[3] Os pacotes do Python podem ser instalados com o Pip ou o Conda. Os pacotes do R podem ser instalados por meio do CRAN ou do GitHub. Os pacotes em F# podem ser instalados por meio de nuget.org usando o [gerenciador de dependência do Paket](https://fsprojects.github.io/Paket/).

