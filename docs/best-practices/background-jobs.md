---
title: Diretrizes de trabalhos em segundo plano
description: Diretrizes sobre as tarefas em segundo plano executadas independentemente da interface do usuário.
author: dragon119
ms.date: 05/24/2017
pnp.series.title: Best Practices
ms.openlocfilehash: 10c24afee4b880cfbf8ee534f4d7f945d2b046a9
ms.sourcegitcommit: 3426a9c5ed937f097725c487cf3d073ae5e2a347
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/01/2018
---
# <a name="background-jobs"></a>Trabalhos em segundo plano
[!INCLUDE [header](../_includes/header.md)]

Muitos tipos de aplicativos requerem tarefas em segundo plano executadas independentemente da interface do usuário (IU). Os exemplos incluem trabalhos em lotes, tarefas com uso intensivo de processamento e processos de longa duração, como fluxos de trabalho. Os trabalhos em segundo plano podem ser executados sem a necessidade de interação do usuário; o aplicativo pode iniciar o trabalho e continuar processando solicitações interativas de usuários. Isso pode ajudar a reduzir a carga na interface do usuário do aplicativo, que pode melhorar a disponibilidade e reduzir os tempos de resposta interativa.

Por exemplo, se for necessário que um aplicativo gere miniaturas de imagens carregadas por usuários, ele poderá fazer isso como um trabalho em segundo plano e salvar a miniatura no armazenamento quando ela for concluída, sem que o usuário precise aguardar a conclusão do processo. Da mesma forma, um usuário fazendo um pedido pode iniciar um fluxo de trabalho em segundo plano que processa o pedido, enquanto a interface do usuário permite que o usuário continue a navegação no aplicativo Web. Quando o trabalho em segundo plano for concluído, ele poderá atualizar os dados de pedidos armazenados e enviar um email para o usuário confirmando o pedido.

Ao considerar se deve implementar uma tarefa como trabalho em segundo plano, o critério principal é se a tarefa pode ser executada sem interação do usuário e sem que a interface do usuário precise aguardar a conclusão desse trabalho. Tarefas que exigem que o usuário ou a interface do usuário aguardem enquanto são concluídas podem não ser apropriadas como trabalhos em segundo plano.

## <a name="types-of-background-jobs"></a>Tipos de trabalhos em segundo plano
Os trabalhos em segundo plano normalmente incluem um ou mais dos seguintes tipos de trabalho:

* Trabalhos com uso intensivo de CPU, como cálculos matemáticos ou análises de modelo estrutural.
* Trabalhos com uso intensivo de E/S, como a execução de uma série de transações de armazenamento ou a indexação de arquivos.
* Trabalhos em lote, como atualizações de dados que são feitas todas as noites ou processamentos agendados.
* Fluxos de trabalho de longa duração, como o atendimento de pedidos ou o provisionamento de serviços e sistemas.
* Processamento de dados confidenciais em que a tarefa é transferida para um local mais seguro para processamento. Por exemplo, não convém processar dados confidenciais em um aplicativo Web. Em vez disso, você pode usar um padrão como o [Gatekeeper](http://msdn.microsoft.com/library/dn589793.aspx) para transferir os dados para um processo em segundo plano isolado que tenha acesso a armazenamento protegido.

## <a name="triggers"></a>Gatilhos
Os trabalhos em segundo plano podem ser iniciados de diversas maneiras. Eles se encaixam em uma destas categorias:

* [**Gatilhos acionados por eventos**](#event-driven-triggers). A tarefa é iniciada em resposta a um evento, normalmente uma ação executada por um usuário ou uma etapa em um fluxo de trabalho.
* [**Gatilhos acionados por agendamentos**](#schedule-driven-triggers). A tarefa é invocada em um agendamento com base em um temporizador. Pode de tratar de um agendamento recorrente ou de uma invocação única especificada para um momento posterior.

### <a name="event-driven-triggers"></a>Gatilhos acionados por eventos
A invocação acionada por evento usa um gatilho para iniciar a tarefa em segundo plano. Exemplos de uso de gatilhos acionados por eventos incluem:

* A interface do usuário ou outro trabalho coloca uma mensagem em uma fila. A mensagem contém dados sobre uma ação que tenha ocorrido, como o usuário fazer um pedido. A tarefa em segundo plano escuta nessa fila e detecta a chegada de uma nova mensagem. Ela lê a mensagem e usa os dados contidos nela como a entrada para o trabalho em segundo plano.
* A interface do usuário ou outro trabalho salva ou atualiza um valor no armazenamento. A tarefa em segundo plano monitora o armazenamento e detecta alterações. Ela lê os dados e os utiliza como entrada para o trabalho em segundo plano.
* A interface do usuário ou outro trabalho faz uma solicitação a um ponto de extremidade, como um URI HTTPS, ou a uma API exposta como um serviço Web. Ela passa os dados necessários para concluir a tarefa em segundo plano como parte da solicitação. O ponto de extremidade ou serviço Web invoca a tarefa em segundo plano, que usa os dados como entrada.

Exemplos comuns de tarefas que são adequadas a invocações acionadas por evento incluem processamento de imagem, fluxos de trabalho, envio de informações para serviços remotos, envio de mensagens de email e provisionamento de novos usuários em aplicativos multilocatário.

### <a name="schedule-driven-triggers"></a>Gatilhos acionados por agendamentos
A invocação acionada por agendamento usa um temporizador para iniciar a tarefa em segundo plano. Exemplos de uso de gatilhos acionados por agendamento incluem:

* Um temporizador que está em execução localmente no aplicativo ou como parte do sistema operacional do aplicativo e que invoca uma tarefa em segundo plano regularmente.
* Um temporizador que está em execução em um aplicativo diferente ou um serviço de temporizador como o Agendador do Azure envia uma solicitação para um serviço Web ou API regularmente. O serviço Web ou API chama a tarefa em segundo plano.
* Um processo separado ou aplicativo inicia um temporizador que faz com que a tarefa em segundo plano seja invocada após um atraso especificado ou em um momento específico.

Exemplos comuns de tarefas que são adequadas para invocação acionada por agendamento incluem rotinas de processamento em lote (como atualização de listas de produtos relacionados para usuários com base no comportamento recente), tarefas rotineiras de processamento de dados (como atualização de índices ou geração de resultados acumulados), análise de dados para relatórios diários, limpeza de retenção de dados e verificação de consistência de dados.

Se você usar uma tarefa acionada por agendamento que precisa ser executada como uma única instância, lembre-se do seguinte:

* se a instância de computação que está executando o agendador (como uma máquina virtual que usa tarefas agendadas do Windows) for dimensionada, você terá várias instâncias do agendador em execução. Elas poderiam iniciar várias instâncias da tarefa.
* Se as tarefas forem executadas por mais tempo que o período entre os eventos do Agendador, o Agendador pode iniciar outra instância da tarefa enquanto a anterior ainda está em execução.

## <a name="returning-results"></a>Retornando os resultados
Os trabalhos em segundo plano são executados de forma assíncrona em um processo separado, ou até mesmo em um local separado, da interface do usuário ou do processo que invocou a tarefa em segundo plano. Em um cenário ideal, as tarefas em segundo plano são operações para "disparar e esquecer", e o seu progresso de execução não tem impacto sobre a interface do usuário ou o processo de chamada. Isso significa que o processo de chamada não aguarda a conclusão das tarefas. Portanto, ele não detecta automaticamente quando a tarefa termina.

Se você precisar de uma tarefa em segundo plano para se comunicar com a tarefa de chamada para indicar o progresso ou a conclusão, você deve implementar um mecanismo para isso. Alguns exemplos incluem:

* gravar o valor de um indicador de status no armazenamento que fica acessível para a tarefa de interface do usuário ou do autor da chamada, que pode monitorar ou verificar esse valor quando for necessário. Outros dados que a tarefa em segundo plano deve retornar para o chamador podem ser colocados no mesmo armazenamento.
* Estabeleça uma fila de resposta que a interface do usuário ou o chamador detecte. A tarefa em segundo plano pode enviar mensagens para a fila indicando o status e a conclusão. Os dados que a tarefa em segundo plano deve retornar ao chamador podem ser colocados em mensagens. Se estiver usando o Barramento de Serviço do Azure, você pode usar as propriedades **ReplyTo** e **CorrelationId** para implementar essa funcionalidade. Para obter mais informações, consulte [Correlação no Sistema de Mensagens Agenciado do Barramento de Serviço](http://www.cloudcasts.net/devguide/Default.aspx?id=13029).
* Expor uma API ou um ponto de extremidade da tarefa em segundo plano que a interface do usuário ou o chamador pode acessar para obter informações de status. Os dados que a tarefa em segundo plano deve retornar ao chamador podem ser incluídos na resposta.
* Retornar a chamada de tarefa de segundo plano à interface do usuário ou ao chamador por meio de uma API para indicar o status em pontos predefinidos ou após a conclusão. Isso pode ser feito por meio de eventos gerados localmente ou de um mecanismo de publicação e assinatura. Os dados que a tarefa em segundo plano deve retornar para o chamador podem ser incluídos na solicitação ou carga do evento.

## <a name="hosting-environment"></a>Ambiente de hospedagem
Você pode hospedar as tarefas em segundo plano usando diversos serviços diferentes da plataforma Azure:

* [**Aplicativos Web do Azure e Azure WebJobs**](#azure-web-apps-and-webjobs). Você pode usar o WebJobs para executar trabalhos personalizados com base em uma variedade de diferentes tipos de scripts ou programas executáveis no contexto de um aplicativo Web.
* [**Máquinas Virtuais do Azure**](#azure-virtual-machines). Caso tenha um serviço Windows ou queira usar o Agendador de Tarefas do Windows, é comum hospedar as suas tarefas em segundo plano em uma máquina virtual dedicada.
* [**Lote do Azure**](#azure-batch). O Lote é um serviço de plataforma que agenda trabalhos de computação intensiva para execução em uma coleção gerenciada de máquinas virtuais. Ele pode dimensionar automaticamente os recursos de computação.
* [**Serviço de Contêiner do Azure**](#azure-container-service). O Serviço de Contêiner do Azure fornece um ambiente de hospedagem de contêineres no Azure. 
* [**Serviços de Nuvem do Azure**](#azure-cloud-services). Você pode gravar código dentro de uma função que é executada como uma tarefa em segundo plano.

As seções a seguir descrevem cada uma dessas opções em mais detalhes e incluem considerações para ajudá-lo a escolher a opção apropriada.

### <a name="azure-web-apps-and-webjobs"></a>Aplicativos Web do Azure e Azure WebJobs

Você pode usar os Azure WebJobs para executar trabalhos personalizados como tarefas em segundo plano em um aplicativo Web do Azure. O WebJobs é executado no contexto de seu aplicativo Web como um processo contínuo. Ele é executado em resposta a um evento de gatilho do Agendador do Azure ou a fatores externos, como alterações em blobs de armazenamento e filas de mensagens. Os trabalhos podem ser iniciados e interrompidos sob demanda, e serem desligados normalmente. Se um WebJob em execução continuamente falhar, ele será reiniciado automaticamente. As ações de nova tentativa e erro são configuráveis.

Quando você configura um WebJob:

* Se desejar que o trabalho responda a um gatilho acionado por evento, você deverá configurá-lo como **Executar continuamente**. O script ou programa é armazenado na pasta chamada site/wwwroot/app_data/jobs/continuous.
* Se desejar que o trabalho responda a um gatilho acionado por agendamento, configure-o como **Executar segundo agendamento**. O script ou o programa é armazenado na pasta chamada site/wwwroot/app_data/jobs/triggered.
* Se você escolher a opção **Executar sob demanda** ao configurar um trabalho, ele executará o mesmo código que a opção **Executar em um agendamento** quando você iniciá-lo.

O Azure WebJobs é executado na área restrita do aplicativo Web. Isso significa que eles podem acessar variáveis de ambiente e compartilhar informações, como cadeias de conexão, com o aplicativo Web. O trabalho tem acesso ao identificador exclusivo do computador que o executa. A cadeia de conexão chamada **AzureWebJobsStorage** fornece acesso a filas de armazenamento, blobs e tabelas do Azure para dados de aplicativo e acesso ao Barramento de Serviço para mensagens e comunicação. A cadeia de conexão chamada **AzureWebJobsDashboard** fornece acesso aos arquivos de log de ação do trabalho.

Os Azure WebJobs têm as seguintes características:

* **Segurança**: os WebJobs são protegidos pelas credenciais de implantação do aplicativo Web.
* **Tipos de arquivo com suporte**: você pode definir WebJobs usando scripts de comando (.cmd), arquivos em lote (.bat), scripts do PowerShell (.ps1), scripts bash shell (.sh), scripts PHP (.PHP), scripts Python (.py), código JavaScript (.js) e programas executáveis (.exe, .jar e outros).
* **Implantação**: você pode implantar scripts e executáveis usando o [portal do Azure](/azure/app-service-web/web-sites-create-web-jobs), usando o [Visual Studio](/azure/app-service-web/websites-dotnet-deploy-webjobs), usando o [SDK do Azure WebJobs](/azure/azure-webjobs-sdk), ou copiá-los diretamente para os seguintes locais:
  * para execução acionada: site/wwwroot/app_data/jobs/triggered/{nome do trabalho}
  * para execução contínua: site/wwwroot/app_data/jobs/continuous/{nome do trabalho}
* **Registro em log**: Console.Out é tratado (marcado) como INFO. Console.Error é tratado como ERROR. Você pode acessar informações de monitoramento e diagnóstico usando o Portal do Azure. Você pode baixar os arquivos de log diretamente do site. Eles estão salvos nos seguintes locais:
  * Para execução disparada: Vfs/data/jobs/triggered/jobName
  * Para execução contínua: Vfs/data/jobs/continuous/jobName
* **Configuração**: você pode configurar o WebJobs usando o portal, a API REST e o PowerShell. Você pode usar um arquivo de configuração chamado settings.job no mesmo diretório raiz que o script de trabalho para fornecer informações de configuração para um trabalho. Por exemplo:
  * { "stopping_wait_time": 60 }
  * { "is_singleton": true }

#### <a name="considerations"></a>Considerações

* Por padrão, os WebJobs são dimensionados com o aplicativo Web. No entanto, você pode configurar os trabalhos para serem executados em única instância definindo a propriedade de configuração **is_singleton** como **true**. O WebJobs de instância única e útil para tarefas que você não quer dimensionar ou executar como várias instâncias simultâneas, como reindexação, análise de dados e tarefas semelhantes.
* Para minimizar o impacto dos trabalhos no desempenho do aplicativo Web, considere a criação de uma instância vazia do Aplicativo Web do Azure em um novo plano de Serviço de Aplicativo para hospedar WebJobs que pode ter execução de longa duração ou consumir muitos recursos.

### <a name="more-information"></a>Mais informações
* [Recursos recomendados dos Azure WebJobs](/azure/app-service-web/websites-webjobs-resources) listam muitos recursos, downloads e exemplos de WebJobs.

### <a name="azure-virtual-machines"></a>Máquinas Virtuais do Azure
Tarefas em segundo plano podem ser implementadas de forma que as impeça de serem implantadas nos Aplicativos Web ou nos Serviços de Nuvem do Azure, ou essas opções podem não ser convenientes. Exemplos típicos são serviços do Windows e utilitários e programas executáveis de terceiros. Outro exemplo seriam programas escritos para um ambiente de execução que é diferente daquele que hospeda o aplicativo. Por exemplo, pode ser um programa Unix ou Linux que você deseja executar em um aplicativo Windows ou .NET. Você pode escolher entre uma variedade de sistemas operacionais para uma máquina virtual do Azure e executar o serviço ou o executável naquela máquina virtual.

Para ajudar com a escolha de quando usar Máquinas Virtuais, confira a [Comparação de Serviço de Aplicativo, Serviços de Nuvem e Máquinas Virtuais do Azure](/azure/app-service-web/choose-web-site-cloud-service-vm/). Para obter informações sobre as opções para Máquinas Virtuais, consulte [Máquina Virtual e tamanhos de Serviço de Nuvem do Azure](http://msdn.microsoft.com/library/azure/dn197896.aspx). Para obter mais informações sobre os sistemas operacionais e imagens predefinidas disponíveis para Máquinas Virtuais, consulte [Marketplace de Máquinas Virtuais do Azure](https://azure.microsoft.com/gallery/virtual-machines/).

Para iniciar a tarefa em segundo plano em uma máquina virtual separada, você tem várias opções:

* Você pode executar a tarefa sob demanda diretamente do seu aplicativo, enviando uma solicitação para um ponto de extremidade que a tarefa expõe. Isso transmite todos os dados que a tarefa requer. Esse ponto de extremidade invoca a tarefa.
* Você pode configurar a tarefa para que ela seja executada segundo uma agenda usando um agendador ou temporizador disponível em seu sistema operacional escolhido. Por exemplo, no Windows, você pode usar o Agendador de Tarefas do Windows para executar scripts e tarefas. Ou, se tiver o SQL Server instalado na máquina virtual, você poderá usar o SQL Server Agent para executar scripts e tarefas.
* Você pode usar o Agendador do Azure para iniciar a tarefa acrescentando uma mensagem a uma fila que a tarefa escuta ou enviando uma solicitação para uma API que a tarefa expõe.

Consulte a seção anterior [Gatilhos](#triggers) para obter mais informações sobre como você pode iniciar tarefas em segundo plano.  

#### <a name="considerations"></a>Considerações
Considere os seguintes pontos ao decidir se deseja implantar tarefas em segundo plano em uma máquina virtual do Azure:

* Hospedar as tarefas em segundo plano em uma máquina virtual do Azure separada fornece   flexibilidade e permite um controle preciso sobre a iniciação, execução, agendamento e alocação de recurso. No entanto, isso aumentará o custo do tempo de execução se uma máquina virtual deve ser implantada apenas para executar tarefas em segundo plano.
* Não há um recurso para monitorar as tarefas no Portal do Azure e nenhuma funcionalidade de reinicialização automatizada para tarefas que falharam, embora você possa monitorar o status básico da máquina virtual e gerenciá-la usando os [Cmdlets do Azure Resource Manager](https://msdn.microsoft.com/library/mt125356.aspx). No entanto, não há nenhum recurso para controlar os processos e threads em nós de computação. Normalmente, usar uma máquina virtual exigirá mais esforço para implementar um mecanismo que coleta dados da instrumentação na tarefa e do sistema operacional na máquina virtual. Uma solução que pode ser apropriada é usar o [System Center Management Pack para o Azure](https://www.microsoft.com/download/details.aspx?id=50013).
* Você pode considerar a criação de testes de monitoramento que são expostos por meio de pontos de extremidade HTTP. O código para esses testes pode executar verificações de integridade, coletar informações operacionais e estatísticas ou agrupar informações de erro e retorná-las a um aplicativo de gerenciamento. Para obter mais informações, consulte o [Padrão de monitoramento de ponto de extremidade de integridade](http://msdn.microsoft.com/library/dn589789.aspx).

#### <a name="more-information"></a>Mais informações
* [Máquinas Virtuais](https://azure.microsoft.com/services/virtual-machines/) no Azure
* [Perguntas frequentes sobre Máquinas Virtuais do Azure](/azure/virtual-machines/virtual-machines-linux-classic-faq?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)

### <a name="azure-batch"></a>Lote do Azure 

Considere o [Lote do Azure](/azure/batch/) se você precisa executar cargas de trabalho grandes, cargas de trabalho HPC (computação de alto desempenho) em paralelo entre dezenas, centenas ou milhares de máquinas virtuais.  

O serviço do Lote provisiona as VMs, atribui tarefas às VMs, executa as tarefas e monitora o progresso. O Lote pode se escalar horizontal e automaticamente as VMs em resposta à carga de trabalho. O Lote também fornece o agendamento de trabalho. O Lote do Azure oferece suporte a máquinas virtuais Windows e Linux.

#### <a name="considerations"></a>Considerações 

O Lote funciona bem com cargas de trabalho intrinsecamente paralelas. Ele também pode executar cálculos paralelos com uma etapa de redução no final ou executar [aplicativos MPI (Interface de Passagem de Mensagens)](/azure/batch/batch-mpi) para tarefas paralelas que exigem a passagem de mensagem entre nós. 

Um trabalho do Lote do Azure é executado em um pool de nós (VMs). Uma abordagem é alocar um pool somente quando necessário e, em seguida, excluí-lo depois que o trabalho for concluído. Isso maximiza a utilização porque os nós não estão ociosos, mas o trabalho deverá aguardar a alocação dos nós. Como alternativa, você pode criar um pool antecipadamente. Essa abordagem minimiza o tempo que leva para um trabalho Iniciar, mas pode resultar em nós ociosos. Para saber mais, veja [Tempo de vida de nó de computação e pool](/azure/batch/batch-api-basics#pool-and-compute-node-lifetime).

#### <a name="more-information"></a>Mais informações 

* [Executar cargas de trabalho intrinsecamente paralelas com o Lote](/azure/batch/batch-technical-overview) 
* [Desenvolver soluções de computação paralela em larga escala com o Lote](/azure/batch/batch-api-basics) 
* [Soluções HPC e do Lote para cargas de trabalho de computação em larga escala](/azure/batch/batch-hpc-solutions)

### <a name="azure-container-service"></a>Serviço de Contêiner do Azure 

O Serviço de Contêiner do Azure permite configurar e gerenciar um cluster de VMs no Azure para executar aplicativos em contêineres. Ele oferece uma opção de Docker Swarm, DC/SO ou Kubernetes para orquestração. 

Os contêineres podem ser úteis para a execução de trabalhos em segundo plano. Alguns benefícios incluem: 

- Os contêineres dão suporte à hospedagem de alta densidade. Você pode isolar uma tarefa em segundo plano em um contêiner ao colocar vários contêineres em cada VM.
- O orquestrador de contêiner lida com o balanceamento de carga interno, configurando a rede interna e outras tarefas de configuração.
- Os contêineres podem ser iniciados e interrompidos conforme necessário. 
- O Registro de Contêiner do Azure permite que você registre seus contêineres dentro dos limites do Azure. Isso é fornecido com os benefícios de segurança, privacidade e proximidade. 

#### <a name="considerations"></a>Considerações

- Requer uma compreensão de como usar um orquestrador de contêiner. Dependendo do conjunto de qualificações da sua equipe DevOps, isso pode ou não ser um problema.  
- O Serviço de Contêiner é executado em um ambiente de IaaS. Ele fornece um cluster de máquinas virtuais dentro de uma rede virtual dedicada. 

#### <a name="more-information"></a>Mais informações 

* [Introdução às soluções de hospedagem de contêineres do Docker com o Serviço de Contêiner do Azure](/azure/container-service/container-service-intro) 
* [Introdução aos registros de contêiner do Docker privado](/azure/container-registry/container-registry-intro) 

### <a name="azure-cloud-services"></a>Serviços de nuvem do Azure 
Tarefas em segundo plano podem ser executadas dentro de uma função web ou em uma função de trabalho separada. Quando estiver decidindo se prefere usar uma função de trabalho, leve em consideração fatores como requisitos de escalabilidade e elasticidade, tempo de vida da tarefa, cadência de versão, segurança, tolerância a falhas, contenção, complexidade e a arquitetura lógica. Para obter mais informações, consulte [Padrão de consolidação de recursos de computação](http://msdn.microsoft.com/library/dn589778.aspx).

Há várias maneiras de implementar tarefas em segundo plano em uma função de Serviços de Nuvem:

* Criar uma implementação da classe **RoleEntryPoint** na função e usar os seus métodos para executar tarefas em segundo plano. As tarefas são executadas no contexto de WaIISHost.exe. Elas podem usar o método **GetSetting** da classe **CloudConfigurationManager** para carregar definições de configuração. Para obter mais informações, consulte [Ciclo de vida](#lifecycle).
* Use as tarefas de inicialização para executar tarefas em segundo plano quando o aplicativo for iniciado. Para forçar as tarefas a continuar sendo executadas em segundo plano, configure a propriedade **taskType** como **background** (se você não fizer isso, o processo de inicialização do aplicativo será interrompido e aguardará até que a tarefa seja concluída). Para saber mais, confira [Executar Tarefas de Inicialização no Azure](/azure/cloud-services/cloud-services-startup-tasks).
* Use o SDK do WebJobs para implementar tarefas em segundo plano como WebJobs, que são iniciados como uma tarefa de inicialização. Para saber mais, veja [Criar um WebJob .NET no Serviço de Aplicativo do Azure](/azure/app-service-web/websites-dotnet-webjobs-sdk-get-started).
* Use uma tarefa de inicialização para instalar um serviço do Windows que executa uma ou mais tarefas em segundo plano. Você deve definir a propriedade **taskType** para **background** para que o serviço seja executado em segundo plano. Para saber mais, confira [Executar Tarefas de Inicialização no Azure](/azure/cloud-services/cloud-services-startup-tasks).

A principal vantagem de executar tarefas em segundo plano na função web é a economia nos custos de hospedagem porque não há nenhum requisito para implantar funções adicionais.

A execução de tarefas em segundo plano em uma função de trabalho tem várias vantagens:

* Ela permite que você gerencie o dimensionamento separadamente para cada tipo de função. Por exemplo, talvez sejam necessárias mais instâncias de uma função web para dar suporte à carga atual, mas menos instâncias da função de trabalho que executa tarefas em segundo plano. Dimensionando instâncias de computação de tarefas de segundo plano separadamente das funções de interface do usuário, você pode reduzir o custo de hospedagem enquanto mantém um desempenho aceitável.
* Ele descarrega a sobrecarga de processamento de tarefas em segundo plano da função web. A função web que fornece a interface do usuário pode permanecer responsiva e pode significar que serão necessárias menos instâncias para dar suporte a um determinado volume de solicitações de usuários.
* Ela permite que você implemente a separação de preocupações. Cada tipo de função pode implementar um conjunto específico de tarefas claramente definidas e relacionadas. Isso torna a criação e a manutenção de código mais fácil porque há menos interdependência de código e funcionalidade entre cada função.
* Isso pode ajudar a isolar os dados e processos importantes. Por exemplo, as funções web que implementam a interface do usuário não precisam ter acesso a dados que são gerenciados e controlados por uma função de trabalho. Isso pode ser útil para reforçar a segurança, especialmente quando você usa um padrão como o [Padrão Gatekeeper](http://msdn.microsoft.com/library/dn589793.aspx).  

#### <a name="considerations"></a>Considerações
Considere os seguintes pontos ao escolher como e onde implantar tarefas em segundo plano ao usar funções web e de trabalho de Serviços de Nuvem:

* Hospedar tarefas em segundo plano em uma função web existente pode economizar o custo de executar uma função de trabalho separada apenas para essas tarefas. No entanto, é provável que isso afete o desempenho e a disponibilidade do aplicativo caso haja contenção para processamento e outros recursos. Usar uma função de trabalho separada protege a função web do impacto causado por tarefas em segundo plano com uso intensivo de recursos ou de longa duração.
* Se hospedar tarefas em segundo plano usando a classe **RoleEntryPoint** , você poderá mover facilmente para outra função. Por exemplo, se você criar a classe em uma função web e depois decidir que precisa executar as tarefas em uma função de trabalho, será possível mover a implementação da classe **RoleEntryPoint** para a função de trabalho.
* As tarefas de inicialização são projetadas para executar um programa ou um script. Implantar um trabalho em segundo plano como um programa executável poderá ser mais difícil, especialmente se ele também exigir a implantação de assemblies dependentes. Pode ser mais fácil implantar e usar um script para definir um trabalho em segundo plano usando tarefas de inicialização.
* As exceções que levam uma tarefa em segundo plano a falhar têm um impacto diferente dependendo da forma como são hospedadas:
  * Se você usar a abordagem de classe **RoleEntryPoint** , uma tarefa com falha fará com que a função reinicie para que a tarefa seja reiniciada automaticamente. Isso pode afetar a disponibilidade do aplicativo. Para evitar isso, certifique-se de incluir um tratamento de exceções robusto na classe **RoleEntryPoint** e todas as tarefas em segundo plano. Use o código para reiniciar tarefas que falham onde for apropriado e lançar a exceção para reiniciar a função somente se você não puder se recuperar normalmente da falha dentro do seu código.
  * Se usar tarefas de inicialização, você é responsável por gerenciar a execução da tarefa e verificar se ela falhar.
* Gerenciar e monitorar tarefas de inicialização é mais difícil do que usar a abordagem de classe **RoleEntryPoint** . No entanto, o SDK do Azure WebJobs inclui um painel para facilitar o gerenciamento de WebJobs iniciado por meio de tarefas de inicialização.

#### <a name="lifecycle"></a>Ciclo de vida 
 Se você decidir implementar trabalhos em segundo plano para aplicativos de Serviços de Nuvem que usam funções da web e de trabalho usando a classe **RoleEntryPoint** , é importante entender o ciclo de vida dessa classe para usá-la corretamente.

As funções da web e de trabalho passam por um conjunto de fases distintas, como iniciar, executar e parar. A classe **RoleEntryPoint** expõe uma série de eventos que indicam quando essas fases estão ocorrendo. Use-as para inicializar, executar e interromper as tarefas personalizadas em segundo plano. O ciclo completo é:

* O Azure carrega o assembly de função e ele o procura para uma classe que deriva de **RoleEntryPoint**.
* Se ele encontrar essa classe, ele chama **RoleEntryPoint.OnStart()**. Você substitui esse método para inicializar as tarefas em segundo plano.
* Após o método **OnStart** ser concluído, o Azure chamará **Application_Start ()** no arquivo Global do aplicativo se ele estiver presente (por exemplo, Global.asax em uma função web executando ASP.NET).
* As chamadas do **Azure RoleEntryPoint.Run()** em um novo thread de primeiro plano são executadas em paralelo com o **OnStart()**. Você substitui esse método para inicializar as tarefas em segundo plano.
* Quando termina o método Run, o Azure chama primeiro **Application_End()** no arquivo Global do aplicativo se ele estiver presente e, em seguida, chama **RoleEntryPoint.OnStop()**. Você substitui o método **OnStop** para interromper as tarefas em segundo plano, limpar recursos, descartar objetos e fechar conexões que podem ter usado as tarefas.
* O processo de host de função de trabalho do Azure está parado. Neste ponto, a função será reciclada e reiniciada.

Para obter mais detalhes e um exemplo de como usar os métodos da classe **RoleEntryPoint** , consulte [Padrão de consolidação de recursos de computação](http://msdn.microsoft.com/library/dn589778.aspx).

#### <a name="implementation-considerations"></a>Considerações sobre a implementação

Considere os seguintes pontos se você estiver implementando tarefas em segundo plano em uma função Web ou de trabalho:

* A implementação do método padrão **Run** na classe **RoleEntryPoint** contém uma chamada para **Thread.Sleep(Timeout.Infinite)** que mantém a função ativa indefinidamente. Se substituir o método **Run** (que é geralmente necessário para executar tarefas em segundo plano), você não deverá permitir que o seu código saia do método, a menos que você deseje reciclar a instância de função.
* Uma implementação típica do método **Run** inclui código para iniciar cada uma das tarefas em segundo plano e um constructo de loop que verifica periodicamente o estado de todas as tarefas em segundo plano. Ela pode reiniciar qualquer falha ou monitorar os tokens de cancelamento que indicam que os trabalhos foram concluídos.
* Se uma tarefa em segundo plano lançar uma exceção sem tratamento, essa tarefa deve ser reciclada, permitindo que outras tarefas de segundo plano na função continuem em execução. No entanto, se a exceção for causada por corrupção de objetos fora da tarefa, como armazenamento compartilhado, a exceção deverá ser tratada pela sua classe **RoleEntryPoint**, todas as tarefas devem ser canceladas e deve ser permitido que o método **Run** seja encerrado. Em seguida, o Azure reiniciará a função.
* Use o método **OnStop** para pausar ou interromper as tarefas em segundo plano e limpar os recursos. Isso pode envolver a interrupção de tarefas de longa execução ou com várias etapas. É essencial considerar como isso pode ser feito para evitar inconsistências de dados. Se uma instância de função para por algum motivo que não seja um desligamento iniciado pelo usuário, o código em execução no método **OnStop** deve ser concluído em cinco minutos antes da finalização forçada. Certifique-se de que o seu código possa ser concluído naquele momento ou que ele possa tolerar não estar em execução até a conclusão.  
* O Azure Load Balancer inicia o direcionamento do tráfego para a instância de função quando o método **RoleEntryPoint.OnStart** retorna o valor **true**. Portanto, considere colocar todo o código de inicialização no método **OnStart** para que as instâncias de função não inicializadas com êxito não recebam qualquer tráfego.
* Você pode usar as tarefas de inicialização além dos métodos da classe **RoleEntryPoint** . Você deve usar as tarefas de inicialização para inicializar as configurações que precisa alterar no balanceador de carga do Azure, pois essas tarefas serão executadas antes de a função receber quaisquer solicitações. Para saber mais, confira [Executar Tarefas de Inicialização no Azure](/azure/cloud-services/cloud-services-startup-tasks/).
* Se houver um erro em uma tarefa de inicialização, isso poderá forçar a função a reiniciar continuamente. Isso pode impedir que você execute uma Permuta de endereço de VIP (IP virtual) para uma versão preparada anteriormente porque a permuta requer acesso exclusivo à função. Não é possível obtê-lo enquanto a função estiver reiniciando. Para resolver esse problema:
  
  * Adicione o seguinte código ao início dos métodos **OnStart** e **Run** em sua função:
    
    ```C#
    var freeze = CloudConfigurationManager.GetSetting("Freeze");
    if (freeze != null)
    {
     if (Boolean.Parse(freeze))
       {
         Thread.Sleep(System.Threading.Timeout.Infinite);
     }
    }
    ```
    
  * Adicione a definição da configuração **Congelar** como um valor booliano para os arquivos ServiceDefinition.csdef e ServiceConfiguration.\*.cscfg para a função e defina-a como **false**. Se a função entra em um modo de reinicialização repetida, você pode alterar a configuração para **true** para congelar a execução da função e permitir que ela seja trocada por uma versão anterior.

#### <a name="more-information"></a>Mais informações
* [Padrão de consolidação de recursos de computação](http://msdn.microsoft.com/library/dn589778.aspx)
* [Introdução ao SDK do Azure WebJobs](/azure/app-service-web/websites-dotnet-webjobs-sdk-get-started/)


## <a name="partitioning"></a>Particionamento
Se decidir incluir tarefas em segundo plano em uma instância de computação existente (como um aplicativo Web, uma função Web, uma função de trabalho existente ou uma máquina virtual), você deverá considerar como isso afetará os atributos de qualidade da instância de computação e a própria tarefa em segundo plano. Esses fatores vão ajudar você a decidir se deseja colocalizar as tarefas com a instância de computação existente ou separá-las em outra instância de computação:

* **Disponibilidade**: as tarefas em segundo plano podem não precisar ter o mesmo nível de disponibilidade que outras partes do aplicativo, em particular a interface do usuário e outras partes diretamente envolvidas na interação do usuário. Tarefas em segundo plano podem ser mais tolerantes a falhas de conexão repetida, latência e outros fatores que afetam a disponibilidade porque as operações podem ser colocadas na fila. No entanto, deve haver capacidade suficiente para impedir o backup de solicitações que poderiam bloquear as filas e afetar o aplicativo como um todo.
* **Escalabilidade**: é provável que as tarefas em segundo plano tenham um requisito de escalabilidade diferente para a interface do usuário e as partes interativas do aplicativo. Pode ser necessário o dimensionamento da interface do usuário para atender a picos de demanda, enquanto as tarefas pendentes em segundo plano podem ser concluídas em horários com menor demanda por menos instâncias de computação.
* **Resiliência**: a falha de uma instância de computação que hospeda apenas tarefas em segundo plano poderá não afetar de modo fatal o aplicativo como um todo se as solicitações para essas tarefas puderem ser enfileiradas ou adiadas até que a tarefa esteja disponível novamente. Se as tarefas e/ou instância de computação puderem ser reiniciadas dentro de um intervalo apropriado, os usuários do aplicativo poderão não ser afetados.
* **Segurança**: as tarefas em segundo plano podem ter requisitos de segurança ou restrições diferentes da interface de usuário ou de outras partes do aplicativo. Ao usar uma instância de computação separada, você pode especificar um ambiente de segurança diferente para as tarefas. Você também pode usar os padrões como Gatekeeper para isolar as instâncias de computação em segundo plano da interface do usuário para maximizar a segurança e a separação.
* **Desempenho**: você pode escolher o tipo de instância de computação para as tarefas em segundo plano para fazer a correspondência especificamente aos requisitos de desempenho das tarefas. Isso poderá significar usar uma opção de computação mais barata se as tarefas não exigirem os mesmos recursos de processamento que a interface do usuário ou uma instância maior se eles exigirem recursos e capacidade adicionais.
* **Gerenciabilidade**: as tarefas em segundo plano podem ter um ritmo diferente de desenvolvimento e implantação daquele do código do aplicativo principal ou da interface do usuário. Implantá-los para uma instância de computação separada pode simplificar as atualizações e o controle de versão.
* **Custo**: adicionar as instâncias de computação para executar tarefas em segundo plano aumenta os custos de hospedagem. Você deve considerar cuidadosamente a compensação entre a capacidade adicional e esses custos extras.

Para obter mais informações, consulte o [Padrão de eleição de líder](http://msdn.microsoft.com/library/dn568104.aspx) e o [Padrão de consumidores concorrentes](http://msdn.microsoft.com/library/dn568101.aspx).

## <a name="conflicts"></a>Conflitos
Se você tiver várias instâncias de um trabalho em segundo plano, será possível que elas competirão pelo acesso aos recursos e serviços, como bancos de dados e armazenamento. Esse acesso simultâneo pode resultar na contenção de recursos, que pode causar conflitos na disponibilidade dos serviços e na integridade dos dados no armazenamento. Você pode resolver a contenção de recursos usando uma abordagem de bloqueio pessimista. Isso evita que instâncias concorrentes de uma tarefa acessem um serviço simultaneamente ou corrompam dados.

Outra abordagem para resolver conflitos é definir tarefas em segundo plano como um singleton, para que haja sempre apenas uma instância em execução. No entanto, isso elimina os benefícios de desempenho e confiabilidade que uma configuração com várias instâncias pode fornecer. Isso será especialmente verdadeiro se a interface do usuário puder fornecer trabalho suficiente para manter mais de uma tarefa em segundo plano ocupada.

É vital garantir que a tarefa em segundo plano possa ser reiniciada automaticamente e tenha capacidade suficiente para lidar com picos de demanda. Isso pode ser feito por meio da alocação de uma instância de computação com recursos suficientes, da implementação de um mecanismo de enfileiramento que pode armazenar solicitações para execução posterior quando a demanda diminui ou com uma combinação dessas técnicas.

## <a name="coordination"></a>Coordenação
As tarefas em segundo plano podem ser complexas e demandar que várias tarefas individuais sejam executadas para produzir um resultado ou para atender a todos os requisitos. É comum nesses cenários dividir a tarefa em subtarefas ou etapas menores que podem ser executadas por vários consumidores. Trabalhos com várias etapas podem ser mais eficientes e mais flexíveis porque as etapas individuais podem ser reutilizáveis em vários trabalhos. Também é fácil adicionar, remover ou modificar a ordem das etapas.

A coordenação de várias tarefas e etapas pode ser desafiadora, mas existem três padrões comuns que você pode usar para orientar a implementação de uma solução:

* **Decomposição de uma tarefa em várias etapas reutilizáveis**. Pode ser necessário um aplicativo para executar uma variedade de tarefas de diferentes níveis de complexidade nas informações que elas processam. Uma abordagem simples, mas inflexível, para implementar esse aplicativo poderia ser executar o processamento como um módulo monolítico. No entanto, essa abordagem provavelmente reduzirá as oportunidades para refatorar o código, otimizá-lo ou reutilizá-lo se partes do mesmo processamento forem necessárias em outros lugares no aplicativo. Para obter mais informações, consulte [Tubos e filtros padrão](http://msdn.microsoft.com/library/dn568100.aspx).
* **Execução de gerenciamento das etapas de uma tarefa**. Um aplicativo pode executar tarefas que compõem uma série de etapas, algumas das quais podem invocar serviços remotos ou acessar recursos remotos. As etapas individuais podem ser independentes umas das outras, mas elas são organizadas pela lógica de aplicativo que implementa a tarefa. Para obter mais informações, consulte o [Padrão do Supervisor do Agente do Agendador](http://msdn.microsoft.com/library/dn589780.aspx).
* **Gerenciamento de recuperação para as etapas de uma tarefa que falhou**. Um aplicativo poderá precisar desfazer o trabalho executado por uma série de etapas, que juntas definem uma operação eventualmente consistente, se uma ou mais das etapas falhar. Para obter mais informações, consulte [Padrão de transação de compensação](http://msdn.microsoft.com/library/dn589804.aspx).


## <a name="resiliency-considerations"></a>Considerações de resiliência
As tarefas em segundo plano devem ser resilientes para fornecer serviços confiáveis ao aplicativo. Ao planejar e criar tarefas em segundo plano, considere os seguintes pontos:

* As tarefas em segundo plano devem ser capazes de lidar normalmente com as reinicializações de função ou serviço sem corromper os dados ou apresentar inconsistência no aplicativo. Para tarefas de execução longa ou com várias etapas, considere o uso de *pontos de verificação* salvando o estado de trabalhos no armazenamento persistente ou como mensagens em uma fila, se isso for apropriado. Por exemplo, você pode manter informações de estado em uma mensagem em uma fila e atualizar incrementalmente essas informações de estado com o andamento da tarefa para que a tarefa possa ser processada desde o último ponto de verificação conhecido, em vez de reiniciar desde o início. Ao usar as filas do Barramento de Serviço do Azure, você pode usar sessões de mensagens para habilitar o mesmo cenário. As sessões permitem salvar e recuperar o estado de processamento do aplicativo usando os métodos [SetState](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate?view=azureservicebus-4.0.0) e [GetState](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession.getstate?view=azureservicebus-4.0.0). Para obter mais informações sobre a criação de fluxos de trabalho e processos confiáveis de várias etapa, consulte [Padrão de Supervisor de Agente do Agendador](http://msdn.microsoft.com/library/dn589780.aspx).
* Ao usar as funções de trabalho ou web para hospedar várias tarefas em segundo plano, projete a sua substituição do método **Run** para monitorar as tarefas com falhas ou paralisadas e reiniciá-las. Quando isso não for prático e você estiver usando uma função de trabalho, force a função de trabalho a reiniciar saindo do método **Run** .
* Ao usar filas para se comunicar com as tarefas em segundo plano, as filas podem agir como um buffer para armazenar solicitações enviadas para as tarefas, enquanto o aplicativo estiver com carga maior que o normal. Isso permite que as tarefas alcancem a interface do usuário durante os períodos menos ocupados. Isso também significa que a reciclagem de função não bloqueará a interface do usuário. Para obter mais informações, consulte o [Padrão de Nivelamento de Carga Baseado em Fila](http://msdn.microsoft.com/library/dn589783.aspx). Se algumas tarefas forem mais importantes que outras, considere implementar o [Padrão de fila de prioridade](http://msdn.microsoft.com/library/dn589794.aspx) para garantir que essas tarefas sejam executadas antes daquelas menos importantes.
* As tarefas em segundo plano que são iniciadas ou processam mensagens devem ser projetadas para lidar com inconsistências, como mensagens que chegam fora de ordem, mensagens que causam um erro repetidas vezes (conhecidas como *mensagens suspeitas*) e mensagens que são entregues mais de uma vez. Considere o seguinte:
  * As mensagens que devem ser processadas em uma ordem específica, como aquelas que alteram os dados com base em seu valor existente (por exemplo, adicionando um valor a um valor existente), podem não chegar na ordem original que foram enviadas. Como alternativa, eles podem ser tratados por diferentes instâncias de uma tarefa em segundo plano em uma ordem diferente devido a cargas diferentes em cada instância. As mensagens que devem ser processadas em uma ordem específica devem incluir um número de sequência, chave ou outro indicador que as tarefas em segundo plano podem usar para garantir que elas sejam processadas na ordem correta. Se estiver usando o Barramento de Serviço do Azure, você pode usar sessões de mensagens para garantir a ordem de entrega. No entanto, é geralmente mais eficiente, quando possível, projetar o processo para que a ordem das mensagens não seja importante.
  * Normalmente, uma tarefa em segundo plano inspecionará mensagens na fila, o que as oculta temporariamente de outros consumidores de mensagens. Em seguida, ela exclui as mensagens após elas serem processadas com êxito. Se uma tarefa em segundo plano falhar durante o processamento de uma mensagem, essa mensagem reaparecerá na fila após o tempo de inspeção terminar. Ela será processada por outra instância da tarefa ou durante o próximo ciclo de processamento desta instância. Se a mensagem causar consistentemente um erro ao consumidor, ela bloqueará a tarefa, a fila e, por fim, o próprio aplicativo quando a fila ficar cheia. Portanto, é vital detectar e remover mensagens suspeitas da fila. Se você estiver usando o Barramento de Serviço do Azure, as mensagens que causam um erro podem ser movidas automaticamente ou manualmente para uma fila de inatividade associada.
  * As filas são garantidas no *mínimo uma vez* nos mecanismos de entrega, mas elas podem entregar a mesma mensagem mais de uma vez. Além disso, se uma tarefa em segundo plano falhar após processar uma mensagem, mas antes de excluí-la da fila, a mensagem estará disponível para processamento novamente. As tarefas em segundo plano devem ser idempotentes, o que significa que processar a mesma mensagem mais de uma vez não causa um erro ou inconsistência nos dados do aplicativo. Algumas operações são naturalmente idempotentes, como a definição de um valor armazenado para um novo valor específico. No entanto, operações como adicionar um valor a um valor armazenado existente sem verificar que o valor armazenado ainda é o mesmo quando a mensagem foi enviada originalmente causará inconsistências. As filas do Barramento de Serviço do Azure podem ser configuradas para remover automaticamente as mensagens duplicadas.
  * Alguns sistemas de mensagens, como as filas de armazenamento do Azure e as filas do Barramento de Serviço do Azure, dão suporte a uma propriedade de contagem de fila que indica o número de vezes que uma mensagem foi lida na fila. Isso pode ser útil ao lidar com mensagens suspeitas e repetidas. Para obter mais informações, consulte [Prévia de mensagens assíncronas](http://msdn.microsoft.com/library/dn589781.aspx) e [Padrões de idempotência](http://blog.jonathanoliver.com/2010/04/idempotency-patterns/).

## <a name="scaling-and-performance-considerations"></a>Considerações sobre dimensionamento e desempenho
As tarefas em segundo plano devem oferecer desempenho suficiente para garantir que elas não bloqueiem o aplicativo nem causem inconsistências devido à operação atrasada quando o sistema estiver sob carga. Normalmente, o desempenho é aprimorado expandindo as instâncias de computação que hospedam as tarefas em segundo plano. Quando estiver planejando e criando tarefas em segundo plano, considere os seguintes pontos ligados ao desempenho e à escalabilidade:

* O Azure dá suporte ao dimensionamento automático (escalar horizontalmente e escalar verticalmente de volta) com base na demanda atual e na carga ou em um planejamento predefinido, para Aplicativos Web, funções de trabalho e Web dos Serviços de Nuvem e implantações de Máquinas Virtuais hospedadas. Use esse recurso para garantir que o aplicativo como um todo tenha recursos suficientes de desempenho enquanto minimiza os custos de tempo de execução.
* Onde as tarefas em segundo plano têm uma funcionalidade de desempenho diferente de outras partes de um aplicativo de Serviços de Nuvem (por exemplo, a interface do usuário ou os componentes, como a camada de acesso a dados), hospedar as tarefas em segundo plano juntas em uma função de trabalho separada permite que as a interface do usuário e as funções da tarefa de segundo plano sejam dimensionadas de forma independente para gerenciar a carga. Se várias tarefas em segundo plano tiverem recursos de desempenho significativamente diferentes umas das outras, considere dividi-las em funções de trabalho separadas e dimensionar cada tipo de função independentemente. No entanto, observe que isso pode aumentar os custos de tempo de execução em comparação a combinar todas as tarefas em menor funções.
* Simplesmente dimensionar as funções pode não ser suficiente para evitar a perda de desempenho sob carga. Talvez também seja necessário dimensionar as filas de armazenamento e outros recursos para impedir que um ponto único do canal geral de processamento se torne um gargalo. Além disso, considere outras limitações, como a taxa de transferência máxima de armazenamento e outros serviços do aplicativo e as tarefas em segundo plano relacionadas.
* As tarefas em segundo plano devem ser projetadas para dimensionamento. Por exemplo, eles devem ser capazes de detectar dinamicamente o número de filas de armazenamento em uso para escutar ou enviar mensagens à fila apropriada.
* Por padrão, os WebJobs são dimensionados com a respectiva instância de Aplicativos Web do Azure associada. No entanto, se quiser que um WebJob seja executado como uma única instância, você poderá criar um arquivo Settings.job que contém os dados JSON **{ "is_singleton": true }**. Isso forçará o Azure a executar apenas uma instância do WebJob, mesmo se houver várias instâncias do aplicativo Web associado. Isso pode ser uma técnica útil para trabalhos agendados que devem ser executados como uma única instância.

## <a name="related-patterns"></a>Padrões relacionados
* [Prévia de mensagens assíncronas](http://msdn.microsoft.com/library/dn589781.aspx)
* [Diretrizes de dimensionamento automático](http://msdn.microsoft.com/library/dn589774.aspx)
* [Padrão de transação de compensação](http://msdn.microsoft.com/library/dn589804.aspx)
* [Padrão de consumidores concorrentes](http://msdn.microsoft.com/library/dn568101.aspx)
* [Diretrizes de particionamento de computação](http://msdn.microsoft.com/library/dn589773.aspx)
* [Padrão de consolidação de recursos de computação](http://msdn.microsoft.com/library/dn589778.aspx)
* [Padrão de gatekeeper](http://msdn.microsoft.com/library/dn589793.aspx)
* [Padrão de eleição de líder](http://msdn.microsoft.com/library/dn568104.aspx)
* [Padrão de filtros e tubos](http://msdn.microsoft.com/library/dn568100.aspx)
* [Padrão de fila de prioridade](http://msdn.microsoft.com/library/dn589794.aspx)
* [Padrão de nivelamento de carga baseado em fila](http://msdn.microsoft.com/library/dn589783.aspx)
* [Padrão de supervisor de agente do Agendador](http://msdn.microsoft.com/library/dn589780.aspx)

## <a name="more-information"></a>Mais informações
* [Dimensionamento de aplicativos do Azure com funções de trabalho](http://msdn.microsoft.com/library/hh534484.aspx#sec8)
* [Executando tarefas em segundo plano](http://msdn.microsoft.com/library/ff803365.aspx)
* [Ciclo de vida de inicialização de função do Azure](http://blog.syntaxc4.net/post/2011/04/13/windows-azure-role-startup-life-cycle.aspx) (postagem de blog)
* [Ciclo de vida da função de Serviços de Nuvem do Azure](http://channel9.msdn.com/Series/Windows-Azure-Cloud-Services-Tutorials/Windows-Azure-Cloud-Services-Role-Lifecycle) (vídeo)
* [O que é o SDK do Azure WebJobs](https://docs.microsoft.com/azure/app-service-web/websites-dotnet-webjobs-sdk)
* [Executar tarefas em segundo plano com o WebJobs](https://docs.microsoft.com/azure/app-service-web/web-sites-create-web-jobs)
* [Filas do Azure e filas do Barramento de Serviço – comparações e contrastes](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted)
* [Como habilitar o diagnóstico em um serviço de nuvem](https://docs.microsoft.com/azure/cloud-services/cloud-services-dotnet-diagnostics)

