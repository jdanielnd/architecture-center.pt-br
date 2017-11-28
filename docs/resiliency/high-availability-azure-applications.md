---
title: Alta disponibilidade para os aplicativos do Azure
description: "Visão geral técnica e informações detalhadas sobre como projetar e criar aplicativos para alta disponibilidade no Microsoft Azure."
author: adamglick
ms.date: 05/31/2017
ms.openlocfilehash: 46b7b802326a8de03546528aaeb1a1c6419d41db
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="high-availability-for-applications-built-on-microsoft-azure"></a>Alta disponibilidade para aplicativos baseados no Microsoft Azure
Um aplicativo altamente disponível absorve flutuações na disponibilidade, carga e falhas temporárias em serviços e hardware dependentes. O aplicativo continua funcionando de forma aceitável, conforme definido pelos requisitos de negócios ou SLAs (contratos de nível de serviço) do aplicativo.

## <a name="azure-high-availability-features"></a>Recursos de alta disponibilidade do Azure
O Azure tem muitos recursos internos da plataforma que oferecem suporte a aplicativos altamente disponíveis. Esta seção descreve alguns desses recursos principais.

### <a name="fabric-controller"></a>Controlador de malha
O controlador de malha do Azure provisiona e monitora a condição das instâncias de computação do Azure. O controlador de malha monitora o status do hardware e do software das instâncias do computador host e convidado. Quando ele detecta uma falha, ele mantém os SLAs automaticamente realocando as instâncias da VM. O conceito de domínios de falha e atualização dá ainda mais suporte ao SLA de computação.

Quando várias instâncias de função do Serviço de Nuvem são implantadas, o Azure implanta essas instâncias em diferentes domínios de falha. Um limite de domínio de falha é basicamente um rack de hardware diferente na mesma região. Domínios de falha reduzem a probabilidade de que uma falha de hardware localizada interrompa o serviço de um aplicativo. Não é possível gerenciar o número de domínios de falha das suas funções de trabalho ou Web. O controlador de malha usa os recursos dedicados que são separados dos aplicativos hospedados pelo Azure. Ele requer 100% de tempo de atividade porque atua como núcleo do sistema do Azure. Ele monitora e gerencia instâncias da função em domínios de falha.

O diagrama a seguir mostra recursos compartilhados do Azure que o controlador de malha implanta e gerencia nos diferentes domínios de falha.

![Exibição simplificada de isolamento de domínio de falha](./images/high-availability-azure-applications/fault-domain-isolation.png)

Embora os domínios de falha sejam separações físicas para mitigar falhas, os domínios de atualização são unidades lógicas de separação da instância que determinam quais instâncias de um serviço serão atualizadas em um momento específico. Por padrão, cinco domínios de atualização são definidos na sua implantação de serviço hospedado. No entanto, você pode alterar esse valor no arquivo de definição de serviço. Por exemplo, se você tiver oito instâncias da função Web, haverá duas instâncias em três domínios de atualização e duas instâncias em um domínio de atualização. O Azure define a sequência de atualização com base no número de domínios de atualização. Para obter mais informações, consulte [Atualizar um serviço de nuvem](/azure/cloud-services/cloud-services-update-azure-service/).

### <a name="features-in-other-services"></a>Recursos em outros serviços
Além dos recursos de plataforma compatíveis com funcionalidades da computação de alta disponibilidade, o Azure insere recursos de alta disponibilidade em outros serviços. Por exemplo, o Armazenamento do Azure mantém pelo menos três réplicas de todos os dados em sua conta de armazenamento do Azure. Ele também habilita a replicação geográfica para armazenar cópias de seus dados em uma região secundária. A Rede de Distribuição de Conteúdo do Microsoft Azure permite que os blobs sejam armazenados em cache em todo o mundo para redundância, escalabilidade e menor latência. Banco de dados SQL do Azure mantém várias réplicas também.

Para ver uma discussão mais aprofundada sobre os recursos de disponibilidade da plataforma do Azure, consulte [Orientações técnicas de resiliência](index.md). Veja também as [Práticas recomendadas para a criação de serviços em grande escala no Microsoft Azure](https://azure.microsoft.com/blog/best-practices-for-designing-large-scale-services-on-windows-azure/).

Embora o Azure ofereça vários recursos compatíveis com a alta disponibilidade, é importante compreender suas limitações:

* Para computação, o Azure garante que as funções estejam disponíveis e em execução, mas ele não detecta se seu aplicativo está em execução ou sobrecarregado.
* Para banco de dados SQL do Azure, os dados são replicados de forma síncrona dentro da região. Você pode optar pela replicação geográfica ativa, que permite até quatro cópias adicionais de banco de dados na mesma região (ou em regiões diferentes). Embora essas réplicas de banco de dados não sejam backups pontuais, o Banco de Dados SQL fornecem recursos de backup pontual. Para obter mais informações, consulte [Recuperar um Banco de Dados SQL do Azure usando backups de dados automatizados: restauração pontual](/azure/sql-database/sql-database-recovery-using-backups#point-in-time-restore).
* Para o Armazenamento do Azure, dados de tabela e de blobs são replicados por padrão em uma região alternativa. No entanto, você não pode acessar as réplicas até que o Microsoft decida efetuar failover para o site alternativo. Um failover de região ocorre apenas durante uma interrupção prolongada do serviço de toda a região e não há nenhum SLA para tempo de failover geográfico. Também é importante observar que qualquer dado corrompido se espalha rapidamente nas réplicas. Por essas razões, é necessário complementar os recursos de disponibilidade da plataforma com recursos de disponibilidade específicos do aplicativo, incluindo o recurso de instantâneo de blobs para criar de backups pontuais dos dados de blobs.

### <a name="availability-sets-for-azure-virtual-machines"></a>Conjuntos de disponibilidade para Máquinas Virtuais do Azure
Esse documento se concentra primariamente em serviços de nuvem, que usam um modelo PaaS (plataforma como serviço). No entanto, também há recursos de disponibilidade específicos para Máquinas Virtuais do Azure, que usam um modelo IaaS (infraestrutura como serviço). Para alcançar alta disponibilidade com Máquinas Virtuais, é necessário usar conjuntos de disponibilidade, que têm função semelhante aos domínios de falha e de atualização. Em um conjunto de disponibilidade, o Azure posiciona as máquinas virtuais de forma a impedir as falhas de hardware localizadas e que as atividades de manutenção desativem todos os computadores nesse grupo. Conjuntos de disponibilidade são necessários para obter o SLA do Azure para disponibilidade de máquinas virtuais.

O diagrama a seguir mostra dois conjuntos de disponibilidade para Web e máquinas virtuais do SQL Server, respectivamente.

![Conjuntos de disponibilidade para Máquinas Virtuais do Azure](./images/high-availability-azure-applications/availability-set-for-azure-virtual-machines.png)

> [!NOTE]
> No diagrama anterior, o SQL Server está instalado e em execução em máquinas virtuais. Isso é diferente do Banco de Dados SQL do Azure, que fornece um banco de dados como um serviço gerenciado.
> 
> 

## <a name="application-strategies-for-high-availability"></a>Estratégias de aplicativo para alta disponibilidade
A maioria das estratégias de aplicativo para alta disponibilidade envolve a redundância ou a remoção de dependências de hardware entre componentes de aplicativos. O design do aplicativo deve dar suporte a tolerância a falhas durante tempo de inatividade esporádico do Azure ou serviços de terceiros. As seções a seguir descrevem padrões de aplicativo para aumentar a disponibilidade dos serviços de nuvem.

### <a name="asynchronous-communication-and-durable-queues"></a>Comunicação assíncrona e filas duráveis
Para aumentar a disponibilidade nos aplicativos do Azure, considere usar comunicação assíncrona entre serviços com acoplamento flexível. Nesse padrão, as mensagens são gravadas em filas de armazenamento ou em filas do Barramento de Serviço do Azure para processamento posterior. Quando uma mensagem é escrita na fila, o controle a retorna imediatamente para o remetente. Outro serviço do aplicativo (geralmente implementado como uma função de trabalho) processa a mensagem. Se o serviço de processamento parar de funcionar, as mensagens serão acumuladas na fila até que o serviço de processamento seja restaurado. Não há nenhuma dependência direta entre o remetente de front-end e o processador de mensagens. Isso elimina a necessidade de chamadas de serviço síncronas que podem causar gargalos em aplicativos distribuídos.

Uma variação desse padrão armazena informações sobre chamadas de banco de dados com falha no Armazenamento do Azure (blobs, tabelas ou filas) ou filas do Barramento de Serviço. Por exemplo, uma chamada síncrona de um aplicativo para outro serviço (como o banco de dados SQL do Azure) falha repetidamente. Você pode serializar essa solicitação em armazenamento durável. Em algum momento posterior, quando o serviço ou o banco de dados estiver novamente online, o aplicativo poderá enviar novamente a solicitação do armazenamento. A diferença nesse modelo é que o local intermediário é usado somente durante falhas e não é uma parte regular do fluxo de trabalho do aplicativo.

Em ambos os cenários, a comunicação assíncrona e o armazenamento intermediário impedem que um serviço de back-end inativo interrompa o aplicativo inteiro. As filas servem como um intermediário lógico. Para saber mais sobre como escolher os serviços de enfileiramento, consulte [Filas do Azure e do Barramento de Serviço do Azure &mdash; comparações e contrastes](/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted/).

### <a name="fault-detection-and-retry-logic"></a>Lógica de detecção e repetição de falhas
Um aspecto principal do design de aplicativos altamente disponíveis é usar a lógica de repetição no código para tratar normalmente um serviço temporariamente indisponível. As versões recentes de SDKs para o Armazenamento do Azure e o Barramento de Serviço do Azure são nativamente compatíveis com repetições. Para obter mais informações sobre como fornecer a lógica de repetição personalizada para seu aplicativo, consulte o [Padrão de repetição](../patterns/retry.md).

### <a name="reference-data-pattern-for-high-availability"></a>Padrão de dados de referência para alta disponibilidade
Dados de referência são os dados somente leitura de um aplicativo. Esses dados fornecem o contexto de negócios dentro do qual o aplicativo gera dados transacionais durante uma operação de negócios. A integridade dos dados transacionais depende do instantâneo dos dados de referência no momento de conclusão da transação.

Dados de referência são necessários para a operação correta do aplicativo. Diversos aplicativos criam e mantêm dados de referência; sistemas MDM (Gerenciamento de Dados Mestre) executam essa função com frequência. Esses sistemas são responsáveis pelo ciclo de vida dos dados de referência. Catálogo de produtos, mestre de funcionários, mestre de peças e mestre de equipamentos são exemplos de dados de referência. Os dados de referência também podem se originar fora da organização, por exemplo, CEPs ou alíquota de imposto. Estratégias para aumentar a disponibilidade dos dados de referência normalmente são menos difíceis que para dados transacionais. Dados de referência têm a vantagem de ser, na maior parte, imutáveis.

Funções Web e de trabalho do Azure que consomem dados de referência podem ser transformadas em autônomas no tempo de execução implantando os dados de referência junto com o aplicativo. Essa abordagem é ideal se o tamanho do armazenamento local permitir essa implantação. Bancos de dados SQL inseridos, bancos de dados NoSQL ou arquivos XML implantados localmente ajudam na autonomia de unidades de escala de computação do Azure. No entanto, você deve ter um mecanismo para atualizar os dados em cada função sem a necessidade de reimplantação. Para fazer isso, coloque as atualizações dos dados de referência em um ponto de extremidade de armazenamento de nuvem (por exemplo, o Armazenamento de Blobs do Azure ou o Banco de Dados SQL). Adicione código a cada função que baixa as atualizações de dados em nós de computação na inicialização da função. Como alternativa, adicione o código que permite que um administrador execute um download forçado nas instâncias da função.

Para aumentar a disponibilidade, as funções também devem conter um conjunto de dados de referência caso o armazenamento esteja inativo. As funções podem começar com um conjunto básico de dados de referência até que os recursos de armazenamento estejam disponíveis para as atualizações.

![Alta disponibilidade de aplicativo através de nós de computação autônomos](./images/high-availability-azure-applications/application-high-availability-through-autonomous-compute-nodes.png)

Com esse padrão, as novas implantações ou instâncias de função podem levar mais tempo para iniciar se você estiver implantando ou baixando grandes volumes de dados de referência. Essa compensação pode ser aceitável para a autonomia de ter dados de referência imediatamente disponíveis em cada função em vez de depender de serviços de armazenamento externo.

### <a name="transactional-data-pattern-for-high-availability"></a>Padrão de dados transacionais para alta disponibilidade
Dados transacionais são os dados que o aplicativo gera em um contexto de negócios. Dados transacionais são uma combinação do conjunto de processos de negócios que o aplicativo implementa e dos dados de referência que permitem esses processos. Exemplos de dados transacionais podem incluir pedidos, aviso de envio avançado, faturas e oportunidades de CRM (gerenciamento de relacionamento com o cliente). Os dados transacionais são fornecidos para sistemas externos para manter um registro ou para processamento adicional.

Os dados de referência podem ser alterados nos sistemas que são responsáveis por esses dados. Portanto, os dados transacionais devem salvar o contexto de dados de referência pontual para minimizar as dependências externas para consistência semântica. Por exemplo, um produto pode ser removido do catálogo vários meses depois que um pedido é atendido. É recomendável armazenar o máximo de contexto de dados de referência possível com a transação. Essa abordagem preserva a semântica associada à transação, mesmo se os dados de referência forem alterados após a captura da transação.

Conforme mencionado anteriormente, arquiteturas que usam acoplamento flexível e comunicação assíncrona podem fornecer níveis mais altos de disponibilidade. Isso também é verdadeiro para dados transacionais, mas a implementação é mais complexa. Os padrões transacionais tradicionais normalmente contam com o banco de dados para garantir a transação. Quando você introduz camadas intermediárias, o código do aplicativo deve tratar corretamente os dados em várias camadas para garantir consistência e durabilidade suficientes.

A seguinte sequência descreve um fluxo de trabalho que separa a captura de dados transacionais de seu processamento:

1. Nó de computação na Web: apresente dados de referência.
2. Armazenamento externo: salve dados transacionais intermediários.
3. Nó de computação na Web: conclua a transação do usuário final.
4. Nó de computação na Web: envie os dados transacionais concluídos com o contexto de dados de referência para um armazenamento durável temporário com garantia de oferecer uma resposta previsível.
5. Nó de computação na Web: informe ao usuário final a conclusão da transação.
6. Nó de computação em segundo plano: extraia os dados transacionais, realize processamento adicional, se necessário e envie-os para o local de armazenamento final no sistema atual.

O diagrama a seguir mostra uma possível implementação desse projeto em um serviço de nuvem hospedado pelo Azure.

![Alta disponibilidade através de acoplamento flexível](./images/disaster-recovery-high-availability-azure-applications/application-high-availability-through-loose-coupling.png)

As setas tracejadas no diagrama acima indicam processamento assíncrono. A função web front-end não está ciente desse processamento assíncrono. Isso resulta no armazenamento da transação no seu destino final com referência ao sistema atual. Devido à latência apresentada neste modelo assíncrono, os dados transacionais não estão imediatamente disponíveis para consulta. Portanto, cada unidade de dados transacionais precisa ser salva em um cache ou uma sessão de usuário para atender às necessidades imediatas da interface do usuário.

A função web independe do restante da infraestrutura. O perfil de disponibilidade é uma combinação da função web e fila do Azure e não de toda a infraestrutura. Além de alta disponibilidade, essa abordagem permite que a função web seja escala horizontalmente, independentemente do armazenamento de back-end. Esse modelo de alta disponibilidade pode ter um impacto sobre a economia das operações. Componentes adicionais, como filas do Azure e funções de trabalho, podem afetar os custos mensais com uso.

Observe que o diagrama anterior mostra uma implementação dessa abordagem desacoplada para dados transacionais. Há muitas outras implementações possíveis. A lista a seguir fornece algumas alternativas:

* Uma função de trabalho deve ser colocada entre a função web e a fila de armazenamento.
* Uma fila do Barramento de Serviço pode ser usada em vez de uma fila de armazenamento do Azure.
* O destino final pode ser o armazenamento do Azure ou um provedor de banco de dados diferente.
* O Cache do Azure pode ser usado na camada Web para fornecer os requisitos imediatos de cache após a transação.

### <a name="scalability-patterns"></a>Padrões de escalabilidade
É importante observar que a escalabilidade do serviço de nuvem afeta diretamente a disponibilidade. Se uma carga maior fizer com que o serviço não responda, a impressão do usuário será que o aplicativo está inativo. Siga as práticas comprovadas para escalabilidade com base em sua carga esperada de aplicativo e em futuras expectativas. Maximizar a escala envolve várias considerações, como o uso de uma única conta ou de várias contas de armazenamento, compartilhando entre vários bancos de dados e estratégias de cache. Para ver informações detalhadas sobre esses padrões, consulte [Práticas recomendadas para projetar serviços em grande escala no Microsoft Azure](https://azure.microsoft.com/blog/best-practices-for-designing-large-scale-services-on-windows-azure/).

## <a name="next-steps"></a>Próximas etapas
Essa série de documentos abrange a recuperação de desastres e alta disponibilidade para aplicativos criados no Microsoft Azure. O próximo artigo dessa série é [Recuperação de desastre para aplicativos criados no Microsoft Azure](disaster-recovery-azure-applications.md).

