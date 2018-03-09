---
title: "Lista de verificação de resiliência"
description: "Lista de verificação que fornece orientação para questões de resiliência durante a criação."
author: petertaylor9999
ms.date: 01/10/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: ca4bf77c9348f6c656348d9cd61d3a1241d69ba8
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/08/2018
---
# <a name="resiliency-checklist"></a>Lista de verificação de resiliência

Resiliência é a capacidade de um sistema de se recuperar de falhas e continuar funcionando, além de ser um dos [pilares da qualidade de software](../guide/pillars.md). Projetar seu aplicativo para garantir a resiliência requer planejamento e redução de uma variedade de modos de falha que podem ocorrer. Use essa lista de verificação para examinar a arquitetura do aplicativo sob um ponto de vista de resiliência. Examine também a [lista de verificação de resiliência para serviços específicos do Azure](./resiliency-per-service.md).

## <a name="requirements"></a>Requisitos

**Defina os requisitos de disponibilidade do seu cliente.** O cliente terá requisitos de disponibilidade para os componentes em seu aplicativo e isso afetará o design do seu aplicativo. Obtenha o contrato do cliente para os destinos de disponibilidade de cada parte do seu aplicativo, caso contrário, seu design poderá não atender às expectativas do cliente. Para obter mais informações, consulte [Definição dos seus requisitos de resiliência](../resiliency/index.md#defining-your-resiliency-requirements).

## <a name="application-design"></a>Design do aplicativo

**Execute uma FMA (Análise do modo de falha) para seu aplicativo.** A FMA é um processo de criação de resiliência em um aplicativo no início do estágio de design. Para obter mais informações, consulte [Análise do modo de falha][fma]. As metas de uma FMA incluem:  

* Identificar quais tipos de falhas um aplicativo pode experimentar.
* Capturar os possíveis efeitos e o impacto de cada tipo de falha no aplicativo.
* Identificar estratégias de recuperação.
  

**Implante várias instâncias de serviços.** Se seu aplicativo depender de uma única instância de um serviço, ele criará um único ponto de falha. O provisionamento de várias instâncias melhora a resiliência e a escalabilidade. Para o [Serviço de Aplicativo do Azure](/azure/app-service/app-service-value-prop-what-is/), selecione um [Plano do Serviço de Aplicativo](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/) que oferece várias instâncias. Para Serviços de Nuvem do Azure, configure cada uma das suas funções para usar [várias instâncias](/azure/cloud-services/cloud-services-choose-me/#scaling-and-management). Para [VMs (Máquinas Virtuais) do Azure](/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json), verifique se sua arquitetura de VM inclui mais de uma VM e se cada VM está incluída em um [conjunto de disponibilidade][availability-sets].   

**Use o dimensionamento automático para responder a aumentos de carga.** Se seu aplicativo não estiver configurado para expandir automaticamente conforme a carga aumenta, os serviços do aplicativo poderão falhar se ficarem saturados com solicitações de usuário. Para obter mais detalhes, consulte o seguinte:

* Geral: [Lista de verificação de escalabilidade](./scalability.md)
* Serviço de Aplicativo do Azure: [Dimensionar a contagem de instâncias manual ou automaticamente][app-service-autoscale]
* Serviços de Nuvem: [Como dimensionar automaticamente um serviço de nuvem][cloud-service-autoscale]
* Máquinas Virtuais: [Dimensionamento automático e conjuntos de dimensionamento de máquinas virtuais][vmss-autoscale]

**Use o balanceamento de carga para distribuir solicitações.** O balanceamento de carga distribui as solicitações do aplicativo para instâncias de serviço íntegras removendo instâncias não íntegras da rotação. Se o seu serviço usa o Serviço de Aplicativo do Azure ou os Serviços de Nuvem do Azure, o balanceamento de carga dele já foi realizado para você. No entanto, se seu aplicativo usar as VMs do Azure, você precisará provisionar um balanceador de carga. Para obter mais detalhes, veja [Visão geral do Azure Load Balancer](/azure/load-balancer/load-balancer-overview/).

**Configure os Gateways de Aplicativo do Azure para usar várias instâncias.** Dependendo dos requisitos do seu aplicativo, um [Gateway de Aplicativo do Azure](/azure/application-gateway/application-gateway-introduction/) pode ser mais conveniente para distribuir solicitações para os serviços do aplicativo. No entanto, instâncias únicas do serviço de Gateway de Aplicativo não garantidas por um SLA, então é possível que seu aplicativo falhe se a instância do Gateway de Aplicativo falhar. Provisione mais de uma instância média ou maior do Gateway de Aplicativo para garantir a disponibilidade do serviço sob os termos do [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/).

**Use Conjuntos de Disponibilidade do Azure para cada camada de aplicativo.** Colocar suas instâncias em um [conjunto de disponibilidade][availability-sets] proporciona um [SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/) de nível mais elevado. 

**Considere implantar o aplicativo entre várias regiões.** Se seu aplicativo é implantado em uma única região, no caso raro de toda a região ficar não disponível, o aplicativo também fica não disponível. Isso pode ser inaceitável segundo os termos do SLA do aplicativo. Nesse caso, considere implantar o aplicativo e os respectivos serviços entre várias regiões. Uma implantação de várias regiões pode usar um padrão ativo-ativo (distribuindo solicitações entre várias instâncias ativas) ou um padrão ativo-passivo (mantendo uma instância "passiva" em reserva, no caso de falha da instância primária). É recomendável que você implante várias instâncias dos serviços do aplicativo entre pares regionais. Para obter mais informações, consulte [Continuidade dos negócios e recuperação de desastres (BCDR): Regiões Emparelhadas do Azure](/azure/best-practices-availability-paired-regions).

**Use o Gerenciador de Tráfego do Azure para rotear o tráfego do seu aplicativo para regiões diferentes.**  [O Gerenciador de Tráfego do Azure][traffic-manager] executa balanceamento de carga no nível de DNS e roteará o tráfego para regiões diferentes com base no método de [roteamento de tráfego][traffic-manager-routing] especificado por você e na integridade dos pontos de extremidade do aplicativo. Sem o Gerenciador de Tráfego, você está limitado a implantar uma única região, o que limita a escala, aumenta a latência para alguns usuários e resulta em tempo de inatividade do aplicativo no caso de uma interrupção do serviço em toda a região.

**Configure e teste investigações de integridade para balanceadores de carga e gerenciadores de tráfego.** Verifique se a lógica de integridade verifica as partes críticas do sistema e responde apropriadamente a investigações de integridade.

* As investigações de integridade para o [Gerenciador de Tráfego do Azure][traffic-manager] e o [Azure Load Balancer][load-balancer] cumprem uma função específica. Para o Gerenciador de Tráfego, a investigação de integridade determina se o failover para outra região deve ou não ser realizado. Para um balanceador de carga, ela determina se uma VM deve ou não ser removida da rotação.      
* Para uma investigação do Gerenciador de Tráfego, o ponto de extremidade de integridade deve verificar todas as dependências críticas que são implantados na mesma região e cuja falha deve disparar um failover para outra região.  
* Para um balanceador de carga, o ponto de extremidade de integridade deve relatar a integridade da VM. Não inclua outras camadas ou serviços externos. Caso contrário, uma falha que ocorra fora da VM fará com que o balanceador de carga remova a VM da rotação.
* Para diretrizes sobre como implementar a integridade de monitoramento em seu aplicativo, consulte [Padrão de monitoramento de ponto de extremidade de integridade](https://msdn.microsoft.com/library/dn589789.aspx).

**Monitore os serviços de terceiros.** Se seu aplicativo tem dependências de serviços de terceiros, identifique onde e como esses serviços de terceiros podem falhar e o efeito que essas falhas teriam sobre o aplicativo. Um serviço de terceiros pode não incluir monitoramento e diagnóstico, então é importante registrar em log as invocações deles feitas por você e correlacioná-las com o log de diagnósticos e a integridade do aplicativo usando um identificador exclusivo. Para obter mais informações sobre práticas comprovadas de monitoramento e diagnóstico, consulte [Diretrizes de monitoramento e diagnóstico][monitoring-and-diagnostics-guidance].

**verifique se qualquer serviço de terceiros que você consome oferece um SLA.** Se seu aplicativo depende de um serviço de terceiros mas o terceiro não fornece nenhuma garantia de disponibilidade na forma de um SLA, a disponibilidade do aplicativo também não pode ser garantida. A qualidade do SLA é medida de acordo com o componente menos disponível do aplicativo.

**Implemente padrões de resiliência para operações remotas quando apropriado.** Se seu aplicativo depende da comunicação entre serviços remotos, siga os padrões de design para lidar com falhas transitórias, assim como [Padrão de Repetição][retry-pattern] e [Padrão de Disjuntor][circuit-breaker]. Para obter mais informações, consulte [Estratégias de resiliência](../resiliency/index.md#resiliency-strategies).

**Implementar operações assíncronas sempre que possível.** Operações síncronas podem monopolizar os recursos e bloquear outras operações enquanto o chamador aguarda a conclusão do processo. Crie cada parte do seu aplicativo para permitir operações assíncronas sempre que possível. Para obter mais informações sobre como implementar a programação assíncrona em C#, consulte [Programação assíncrona com async e await][asynchronous-c-sharp].

## <a name="data-management"></a>Gerenciamento de dados

**Entenda os métodos de replicação para as fontes de dados do aplicativo.** Os dados do aplicativo são armazenados em diferentes fontes de dados e têm requisitos de disponibilidade diferentes. Avalie os métodos de replicação para cada tipo de armazenamento de dados no Azure, incluindo [Replicação do Armazenamento do Azure](/azure/storage/storage-redundancy/) e [Replicação Geográfica Ativa do Banco de Dados SQL](/azure/sql-database/sql-database-geo-replication-overview/), para garantir que os requisitos de dados do aplicativo sejam atendidos.

**Assegure que nenhuma conta de usuário individual tenha acesso aos dados de backup e de produção.** Os backups de dados são comprometidos se uma única conta de usuário tem permissão para gravar tanto para origens de gravação quanto de backup. Um usuário mal-intencionado pode excluir propositadamente todos os seus dados, enquanto um usuário normal pode excluí-los acidentalmente. Projete seu aplicativo para limitar as permissões de cada conta de usuário para que apenas os usuários que necessitam de acesso de gravação tenham esse acesso e que seja apenas para produção ou para backup, mas não ambos.

**Documente o processo de failover e failback de sua fonte de dados e teste-a.** Caso a fonte de dados falhe de modo catastrófico, será necessário que um operador humano siga um conjunto de instruções documentadas para fazer failover para uma nova fonte de dados. Se as etapas documentadas tiverem erros, um operador não poderá segui-las e fazer o failover do recurso com êxito. Teste regularmente as etapas de instrução para verificar se um operador que as está seguindo é capaz de fazer failover e failback da fonte de dados com êxito.

**Valide os backups de dados.** Verifique regularmente que os dados de backup são o que você espera ao executar um script para validar o esquema, as consultas e a integridade dos dados. Não há razão para ter um backup se ele não é útil para restaurar suas fontes de dados. Registre em log e relate as inconsistências de modo que o serviço de backup possa ser reparado.

**Considere o uso de um tipo de conta de armazenamento com redundância geográfica.** Dados armazenados em uma Conta de Armazenamento do Azure são sempre replicados localmente. No entanto, há várias estratégias de replicação para escolher quando uma Conta de Armazenamento é provisionada. Selecione [RA-GRS (armazenamento com redundância geográfica com acesso de leitura) do Azure](/azure/storage/storage-redundancy/#read-access-geo-redundant-storage) para proteger os dados do aplicativo no caso raro de uma região inteira ficar não disponível.

> [!NOTE]
> Para VMs, não dependa da replicação de RA-GRS para restaurar os discos de VM (arquivos VHD). Em vez disso, use o [Backup do Azure][azure-backup].   
>
>

## <a name="security"></a>Segurança

**Implemente a proteção em nível de aplicativo contra ataques DDoS (ataques de negação de serviço distribuídos).** Os serviços do Azure são protegidos contra ataques de DDoS na camada de rede. No entanto, o Azure não pode proteger contra ataques na camada de aplicativo, porque é difícil distinguir entre solicitações de usuários verdadeiros e solicitações de usuários mal-intencionados. Para obter mais informações sobre como se proteger contra ataques de DDoS da camada de aplicativo, consulte a seção "Proteção contra DDoS" da [Segurança de Rede do Microsoft Azure](http://download.microsoft.com/download/C/A/3/CA3FC5C0-ECE0-4F87-BF4B-D74064A00846/AzureNetworkSecurity_v3_Feb2015.pdf) (download do PDF).

**Implemente o princípio de privilégios mínimos para acesso aos recursos do aplicativo.** O padrão para acesso aos recursos do aplicativo deve ser tão restritivo quanto possível. Conceda permissões de nível superior em uma base de aprovação. Conceder acesso excessivamente permissivo a recursos do aplicativo por padrão pode resultar em exclusão intencional ou acidental de recursos por alguém. O Azure fornece [controle de acesso baseado em função](/azure/active-directory/role-based-access-built-in-roles/) gerenciar os privilégios de usuário, mas é importante verificar permissões com privilégios mínimos para outros recursos que têm seus próprios sistemas de permissões, tais como o SQL Server.

## <a name="testing"></a>Testando

**Execute o teste de failover e de failback para o aplicativo.** Se você ainda não testou totalmente o failover e o failback, você não pode ter certeza de que os serviços dependentes em seu aplicativo voltam ao funcionamento de forma sincronizada durante uma recuperação de desastre. Verifique se os serviços que dependem do seu aplicativo fazem failover e failback na ordem correta.

**Execute testes de injeção de falha em seu aplicativo.** Seu aplicativo pode falhar por vários motivos diferentes, como a expiração do certificado, o esgotamento de recursos do sistema em uma VM ou falhas de armazenamento. Teste seu aplicativo em um ambiente que se assemelhe o máximo possível à produção, simulando ou disparando falhas reais. Por exemplo, exclua certificados, consuma artificialmente recursos do sistema ou exclua uma origem de armazenamento. Verifique a capacidade do aplicativo de se recuperar de todos os tipos de falhas, isoladamente e em conjunto. Verifique se as falhas não estão se propagando ou ocorrendo em cascata pelo sistema.

**Execute testes em produção usando tanto dados de usuário sintéticos quanto reais.** Teste e produção são raramente idênticos, portanto, é importante usar uma implantação canário ou azul/verde e testar seu aplicativo em produção. Isso permite que você teste seu aplicativo em produção sob carga real e verifique se ele funciona conforme o esperado quando totalmente implantado.

## <a name="deployment"></a>Implantação

**Documente o processo de lançamento para o seu aplicativo.** Sem documentação detalhada do processo de lançamento, um operador pode implantar uma atualização inválida ou definir configurações incorretamente para o aplicativo. Defina e documente claramente seu processo de lançamento e verifique se ele está disponível para toda a equipe de operações. 

**Automatize o processo de implantação do aplicativo.** Se é necessário que sua equipe de operações implante manualmente o seu aplicativo, erros humanos podem causar falha na implantação. 

**Projete seu processo de lançamento para maximizar a disponibilidade do aplicativo.** Se o processo de lançamento exigir que os serviços fiquem offline durante a implantação, o aplicativo ficará não disponível até que eles fiquem online novamente. Use a técnica de implantação de [lançamento canário](http://martinfowler.com/bliki/CanaryRelease.html) ou [azul/verde](http://martinfowler.com/bliki/BlueGreenDeployment.html) para implantar seu aplicativo para produção. Ambas as técnicas envolvem implantar seu código de lançamento junto com o código de produção para que os usuários do código de liberação possam ser redirecionados para o código de produção em caso de falha.

**Registre as implantações do aplicativo em log e faça auditoria delas.** Se você usar técnicas de implantação de teste como lançamento canário ou azul/verde, haverá mais de uma versão do aplicativo em execução na produção. Se ocorre um problema, é essencial determinar por qual versão do aplicativo ele é causado. Implemente uma estratégia de registro em log robusta para capturar o máximo possível de informações específicas da versão.

**Tenha um plano de reversão para a implantação.** É possível que a implantação do aplicativo pode falhe e deixe o aplicativo não disponível. Crie um processo de reversão para voltar para uma última versão válida conhecida e minimizar o tempo de inatividade. 

## <a name="operations"></a>Operações

**Implemente as práticas recomendadas para monitoramento e alertas em seu aplicativo.** Sem monitoramento, diagnóstico e alertas adequados, não é possível detectar falhas em seu aplicativo e alertar um operador para corrigi-las. Para obter mais informações, consulte [Diretrizes de monitoramento e diagnóstico][monitoring-and-diagnostics-guidance].

**Meça as estatísticas de chamada remota e disponibilize as informações para a equipe do aplicativo.**  Se você não acompanhar e relatar estatísticas de chamada remota em tempo real e não fornecer uma maneira fácil de examinar essas informações, a equipe de operações não terá uma exibição instantânea da integridade do aplicativo. E se você medir apenas o tempo médio das chamadas remotas, você não terá informações suficientes para revelar problemas nos serviços. Resuma métricas de chamada remota como latência, taxa de transferência e erros no 99º e no 95º percentis. Execute análise estatística nas métricas para revelar os erros que ocorrem dentro de cada percentual.

**Acompanhe o número de exceções transitórias e de novas tentativas durante um período de tempo apropriado.** Se você não controlar e monitorar exceções transitórias e novas tentativas ao longo do tempo, uma falha ou problema poderá ficar oculta pela lógica de repetição do seu aplicativo. Ou seja, se o monitoramento e registro em log mostra somente êxito ou falha de uma operação, o fato de que a operação precisou ser repetida várias vezes devido às exceções ficará oculto. Uma tendência de aumento de exceções ao longo do tempo indica que o serviço está apresentando um problema e pode falhar. Para obter mais informações, consulte [Diretrizes específicas sobre repetição de serviço][retry-service-guidance].

**Implemente um sistema de avisos antecipados que alerta um operador.** Identifique os indicadores chave de desempenho de integridade do aplicativo, tais como exceções transitórias e latência de chamadas remotas e defina valores limite adequados para cada um deles. Envie um alerta para operações quando o valor limite é atingido. Defina esses limites para níveis que identifiquem problemas antes que eles se tornem críticos e exijam uma resposta de recuperação.

**Verifique se mais de uma pessoa da equipe está treinada para monitorar o aplicativo e executar as etapas de recuperação manual.** Se você tem apenas um único operador da equipe que pode monitorar o aplicativo e disparar etapas de recuperação, essa pessoa se torna um ponto único de falha. Treine vários indivíduos sobre detecção e recuperação e verifique se há sempre pelo menos um ativo a qualquer momento.

**Verifique se o aplicativo não é submetido a [limites de assinatura do Azure](/azure/azure-subscription-service-limits/).** As assinaturas do Azure têm limites para determinados tipos de recursos, como o número de grupos de recursos, o número de núcleos e o número de contas de armazenamento.  Se seus requisitos de aplicativo excedem os limites de assinatura do Azure, crie outra assinatura do Azure e provisione recursos suficientes nela.

**Verifique se o aplicativo não é confrontado com [limites por serviço](/azure/azure-subscription-service-limits/).** Os serviços do Azure individuais têm limites de consumo &mdash; por exemplo, limites para armazenamento, taxa de transferência, número de conexões, solicitações por segundo e outras métricas. O aplicativo falhará se ele tentar usar recursos além desses limites. Isso resultará em limitação do serviço e possível tempo de inatividade para os usuários afetados. Dependendo do serviço específico e seus requisitos de aplicativo, você geralmente pode evitar esses limites por meio de expansão, seja escolhendo outro tipo de preço ou adicionando novas instâncias.  

**Projete os requisitos de armazenamento do aplicativo para ficarem dentro das metas de desempenho e escalabilidade do Armazenamento do Azure.** O Armazenamento do Azure foi projetado para funcionar dentro de metas de escalabilidade e desempenho predefinidas, portanto, projete seu aplicativo para utilizar o armazenamento dentro dessas metas. Se você exceder essas metas, o armazenamento do aplicativo sofrerá limitação. Para corrigir isso, provisione contas de armazenamento adicionais. Se você atingir o limite da contas de armazenamentos, provisione assinaturas do Azure adicionais e, em seguida, provisione contas de armazenamento adicionais nelas. Para saber mais, consulte [Metas de desempenho e escalabilidade do Armazenamento do Azure](/azure/storage/storage-scalability-targets/).

**Selecione o tamanho de VM certo para o aplicativo.** Meça a CPU, memória, disco e E/S de suas VMs em produção e verifique se o tamanho da VM que você selecionou é suficiente. Se não é, seu aplicativo pode apresentar problemas de capacidade, tais como VMs se aproximarem de seus limites. Tamanhos de VM são descritos em detalhes em [Tamanhos de máquinas virtuais no Azure](/azure/virtual-machines/virtual-machines-windows-sizes/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

**Determine se a carga de trabalho do aplicativo é estável ou varia ao longo do tempo.** Se sua carga de trabalho varia ao longo do tempo, use conjuntos de dimensionamento de VMs do Azure para dimensionar automaticamente o número de instâncias de VM. Caso contrário, você precisará aumentar ou diminuir manualmente o número de VMs. Para obter mais informações, consulte a [Visão geral de conjuntos de dimensionamento de máquinas virtuais](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/).

**Selecione a camada de serviço certa para o Banco de Dados SQL do Azure.** Se seu aplicativo usa o Banco de Dados SQL do Azure, verifique se você selecionou a camada de serviço apropriada. Se você selecionar uma camada que não é capaz de lidar com os requisitos de DTU (unidade de transação de banco de dados) do aplicativo, o uso de dados será limitado. Para obter mais informações sobre como selecionar o plano de serviço correto, veja [Opções e desempenho de Banco de Dados SQL: saiba o que está disponível em cada camada de serviço](/azure/sql-database/sql-database-service-tiers/).

**Crie um processo para interagir com o suporte do Azure.** Se o processo de contatar o [suporte do Azure](https://azure.microsoft.com/support/plans/) não for definido antes de a necessidade de entrar em contato com o suporte surgir, o tempo de inatividade será prolongado conforme a navegação pelo processo de suporte for realizada pela primeira vez. Inclua o processo de entrar em contato com o suporte e escalar problemas como parte de resiliência do aplicativo desde o início.

**Verifique se o aplicativo não usa mais do que o número máximo de contas de armazenamento por assinatura.** O Azure permite um máximo de 200 contas de armazenamento por assinatura. Se o aplicativo exigir mais contas de armazenamento do que estiverem atualmente disponíveis na assinatura, você precisará de criar uma nova assinatura e criar contas de armazenamento adicionais nela. Para saber mais, confira [Assinatura e limites de serviço, cotas e restrições do Azure](/azure/azure-subscription-service-limits/#storage-limits).

**Verifique se o aplicativo não excede as metas de escalabilidade para discos de máquina virtual.** Uma VM de IaaS do Azure dá suporte à anexação de um número de discos de dados que depende de vários fatores, incluindo o tamanho da VM e o tipo de conta de armazenamento. Se o aplicativo exceder as metas de escalabilidade para discos de máquina virtual, provisione contas de armazenamento adicionais e crie os discos da máquina virtual nelas. Para obter mais informações, consulte [Metas de desempenho e escalabilidade do Armazenamento do Azure](/azure/storage/storage-scalability-targets/#scalability-targets-for-virtual-machine-disks)

## <a name="telemetry"></a>Telemetria

**Registre em log os dados de telemetria enquanto o aplicativo está em execução no ambiente de produção.** Capture informações de telemetria robustas enquanto o aplicativo estiver em execução no ambiente de produção ou você não terá informações suficientes para diagnosticar a causa dos problemas enquanto ele estiver atendendo usuários ativamente. Para obter mais informações, consulte [Monitoramento e diagnóstico][monitoring-and-diagnostics-guidance].

**Implemente registro em log usando um padrão assíncrono.** Se as operações de registro em log são síncronas, elas podem bloquear o código do aplicativo. Verifique se as operações de registro em log são implementadas como operações assíncronas.

**Correlacione dados de log através de limites de serviços.** Em um aplicativo de n camadas típico, uma solicitação de usuário pode atravessar vários limites de serviço. Por exemplo, uma solicitação de usuário normalmente se origina na camada da Web e é passada para a camada de negócios e por fim é persistida na camada de dados. Em cenários mais complexos, uma solicitação de usuário pode ser distribuída a muitos armazenamentos de dados e serviços diferentes. Verifique se o sistema de registro em log correlaciona chamadas entre limites de serviço para que você possa rastrear a solicitação pelo seu aplicativo.

## <a name="azure-resources"></a>Recursos do Azure

**Use modelos do Azure Resource Manager para provisionar recursos.** Modelos do Resource Manager facilitam a automatização de implantações por meio do PowerShell ou da CLI do Azure, o que leva a um processo de implantação mais confiável. Para saber mais, confira [Visão geral do Azure Resource Manager][resource-manager].

**Dê nomes significativos aos recursos.** Dar nomes significativos aos recursos facilita a localização de um recurso específico e o entendimento de sua função. Para obter mais informações, consulte as [Convenções de nomenclatura para os Recursos do Azure](../best-practices/naming-conventions.md)

**Use o RBAC (Controle de Acesso Baseado em Função).** Use o RBAC para controlar o acesso aos recursos do Azure que você implanta. O RBAC permite que você atribua funções de autorização para os membros da equipe DevOps, para impedir exclusão acidental ou alterações a recursos implantados. Para obter mais informações, veja [Introdução ao gerenciamento de acesso no Portal do Azure](/azure/active-directory/role-based-access-control-what-is/)

**Use bloqueios de recursos para recursos críticos, tais como VMs.** Bloqueios de recurso impedem que um operador exclua um recurso acidentalmente. Para saber mais, confira [Bloquear recursos com o Azure Resource Manager](/azure/azure-resource-manager/resource-group-lock-resources/)

**Escolha pares regionais.** Ao implantar em duas regiões, escolha regiões do mesmo par de regiões. No caso de uma interrupção ampla, é priorizada a recuperação de uma região de cada par. Alguns serviços, tais como o armazenamento com redundância geográfica, fornecem replicação automática para a região emparelhada. Para saber mais, consulte [Continuidade dos negócios e recuperação de desastres (BCDR): Regiões Emparelhadas do Azure](/azure/best-practices-availability-paired-regions)

**Organize os grupos de recursos por função e ciclo de vida.**  Em geral, um grupo de recursos deve conter recursos que compartilham o mesmo ciclo de vida. Isso facilita o gerenciamento de implantações, a exclusão de implantações de teste e a atribuição de direitos de acesso, reduzindo a chance de que uma implantação de produção seja acidentalmente excluída ou modificada. Crie grupos de recursos separados para ambientes de produção, desenvolvimento e teste. Em uma implantação de várias regiões, coloque os recursos para cada região em grupos de recursos separados. Isso torna mais fácil reimplantar uma região sem afetar as outras regiões.

## <a name="next-steps"></a>Próximas etapas

- [Lista de verificação de resiliência para serviços específicos do Azure](./resiliency-per-service.md)
- [Análise do modo de falha](../resiliency/failure-mode-analysis.md)


<!-- links -->
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale/
[asynchronous-c-sharp]: /dotnet/articles/csharp/async
[availability-sets]:/azure/virtual-machines/virtual-machines-windows-manage-availability/
[azure-backup]: https://azure.microsoft.com/documentation/services/backup/
[circuit-breaker]: ../patterns/circuit-breaker.md
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale/
[fma]: ../resiliency/failure-mode-analysis.md
[load-balancer]: /azure/load-balancer/load-balancer-overview/
[monitoring-and-diagnostics-guidance]: ../best-practices/monitoring.md
[resource-manager]: /azure/azure-resource-manager/resource-group-overview/
[retry-pattern]: ../patterns/retry.md
[retry-service-guidance]: ../best-practices/retry-service-specific.md
[traffic-manager]: /azure/traffic-manager/traffic-manager-overview/
[traffic-manager-routing]: /azure/traffic-manager/traffic-manager-routing-methods/
[vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview/
