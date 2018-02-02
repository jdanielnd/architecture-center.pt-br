---
title: "Lista de verificação de resiliência"
description: "Lista de verificação que fornece orientação para questões de resiliência durante a criação."
author: petertaylor9999
ms.date: 01/10/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: 51f807715d0ac929806b9a5a13da4efa00566592
ms.sourcegitcommit: a7aae13569e165d4e768ce0aaaac154ba612934f
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/30/2018
---
# <a name="resiliency-checklist"></a>Lista de verificação de resiliência

Resiliência é a capacidade de um sistema de se recuperar de falhas e continuar funcionando, além de ser um dos [pilares da qualidade de software](../guide/pillars.md). Projetar seu aplicativo para garantir a resiliência requer planejamento e redução de uma variedade de modos de falha que podem ocorrer. Use essa lista de verificação para examinar a arquitetura do aplicativo sob um ponto de vista de resiliência. 

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

## <a name="azure-services"></a>Serviços do Azure
Os itens de lista de verificação a seguir se aplicam a serviços específicos no Azure.

- [Serviço de Aplicativo](#app-service)
- [Gateway de Aplicativo](#application-gateway)
- [Cosmos DB](#cosmos-db)
- [Cache Redis](#redis-cache)
- [Search](#search)
- [Armazenamento](#storage)
- [Banco de Dados SQL](#sql-database)
- [SQL Server em execução em uma VM](#sql-server-running-in-a-vm)
- [Gerenciador de Tráfego](#traffic-manager)
- [Máquinas virtuais](#virtual-machines)
- [Rede Virtual](#virtual-network)

### <a name="app-service"></a>Serviço de Aplicativo

**Use a camada Standard ou Premium.** Essas camadas dão suporte a slots de preparo e backups automáticos. Para obter mais informações, consulte [Visão geral aprofundada de planos do Serviço de Aplicativo do Azure](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/)

**Evite expandir ou reduzir verticalmente.** Em vez disso, selecione uma camada e o tamanho de instância que atendem aos seus requisitos de desempenho sob carga típica e, em seguida, [aumente](/azure/app-service-web/web-sites-scale/) as instâncias para manipular as alterações no volume de tráfego. Expandir e reduzir verticalmente pode disparar uma reinicialização do aplicativo.  

**Armazene a configuração como configurações de aplicativo.** Use as configurações de aplicativo para manter definições de configuração como configurações de aplicativo. Defina as configurações em seus modelos do Resource Manager ou usando o PowerShell, para que você possa aplicá-las como parte de um processo automatizado de implantação/atualização, que é mais confiável. Para obter mais informações, consulte [Configurar aplicativos Web no Serviço de Aplicativo do Azure](/azure/app-service-web/web-sites-configure/).

**Crie Planos do Serviço de Aplicativo separados para produção e teste.** Não use slots em sua implantação de produção para teste.  Todos os aplicativos dentro do mesmo Plano do Serviço de Aplicativo compartilham as mesmas instâncias de VM. Se você colocar a produção e implantações de teste no mesmo plano, isso poderá afetar negativamente a implantação de produção. Por exemplo, testes de carga podem prejudicar o site de produção dinâmica. Colocando as implantações de teste em um plano separado, você as isola da versão de produção.  

**Separe aplicativos Web de APIs da Web.** Se sua solução tem um front-end da Web e uma API da Web, considere a possibilidade de decompô-los em aplicativos de Serviço de Aplicativo separados. Esse design torna mais fácil para decompor a solução pela carga de trabalho. Você pode executar o aplicativo Web e a API em Planos do Serviço de Aplicativo separados, de modo que eles possam ser dimensionados de forma independente. Se você não precisar desse nível de escalabilidade inicialmente, poderá implantar os aplicativos no mesmo plano e movê-los para planos separados posteriormente, se necessário.

**Evite usar o recurso de Serviço de Aplicativo de backup para fazer backup de Bancos de Dados SQL do Azure.** Em vez disso, use os [Backups automatizados do Banco de Dados SQL][sql-backup]. O backup do Serviço de Aplicativo exporta o banco de dados para um arquivo .bacpac do SQL, que custa DTUs.  

**Implante em um slot de preparo.** Crie um slot de implantação para preparo. Implante atualizações de aplicativo para o slot de preparo e verifique a implantação antes de trocá-la para produção. Isso reduz a chance de uma atualização inválida em produção. Isso também garante que todas as instâncias sejam aquecidas antes de serem trocadas para produção. Muitos aplicativos têm um tempo significativo de aquecimento e de resfriamento. Para obter mais informações, consulte [Configurar ambientes de preparo para aplicativos Web no Serviço de Aplicativo do Azure](/azure/app-service-web/web-sites-staged-publishing/).

**Crie um slot de implantação para conter a LKG (última implantação válida).** Quando você implanta uma atualização para produção, mova a implantação de produção anterior para o slot da LKG. Isso torna mais fácil reverter uma implantação incorreta. Se você descobrir um problema posteriormente, você poderá reverter rapidamente para a versão LKG. Para obter mais informações, veja [Basic web application](../reference-architectures/app-service-web-app/basic-web-app.md) (Aplicativo Web básico).

**Habilite o log de diagnósticos**, incluindo o log de aplicativo e o log do servidor Web. O registro em log é importante para monitoramento e diagnóstico. Consulte [Habilitar o registro de log de diagnóstico para aplicativos Web no Serviço de Aplicativo do Azure](/azure/app-service-web/web-sites-enable-diagnostic-log/)

**Registre em log para o Armazenamento de Blobs.** Isso torna mais fácil coletar e analisar os dados.

**Criar uma conta de armazenamento separada para logs.** Não use a mesma conta de armazenamento para logs e dados de aplicativo. Isso ajuda a impedir o registro em log de reduzir o desempenho do aplicativo.

**Monitore o desempenho.** Use um serviço de monitoramento de desempenho como o [New Relic](http://newrelic.com/) ou o [Application Insights](/azure/application-insights/app-insights-overview/) para monitorar o desempenho e o comportamento do aplicativo sob carga.  O monitoramento de desempenho fornece informações sobre o aplicativo em tempo real. Ele permite a você diagnosticar problemas e executar a análise de causa raiz de falhas.

### <a name="application-gateway"></a>Gateway de Aplicativo

**Provisione pelo menos duas instâncias.** Implante um Gateway de Aplicativo com pelo menos duas instâncias. Uma única instância é um ponto único de falha. Use duas ou mais instâncias para redundância e escalabilidade. Para se qualificar para o [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/), você deverá provisionar duas ou mais instâncias médias ou maiores.

### <a name="cosmos-db"></a>Cosmos DB

**Replique o banco de dados entre regiões.** O Cosmos DB permite associar qualquer número de regiões do Azure a uma conta de banco de dados do Cosmos DB. Um banco de dados do Azure Cosmos DB pode ter várias regiões de leitura e uma de gravação. Se houver uma falha na região de gravação, você poderá ler de outra réplica. O SDK do cliente lida com isso automaticamente. Você também pode fazer failover da região de gravação para outra região. Para obter mais informações, confira [Como distribuir os dados globalmente com o Azure Cosmos DB](/azure/cosmos-db/distribute-data-globally).

### <a name="redis-cache"></a>Cache Redis

**Configurar a replicação geográfica**. A Replicação geográfica fornece um mecanismo para vincular duas instâncias de Cache Redis do Azure de camada Premium. Os dados gravados no cache primário são replicados para um cache secundário de somente leitura. Para obter mais informações, consulte [Como configurar a replicação geográfica do Cache Redis do Azure](/azure/redis-cache/cache-how-to-geo-replication)

**Configurar a persistência de dados.** A persistência do Redis permite persistir os dados armazenados no Redis. Você também pode tirar instantâneos e fazer backup dos dados, que podem ser carregados em caso de falha de hardware. Para obter mais informações, consulte [Como configurar a persistência de dados para um Cache Redis do Azure Premium](/azure/redis-cache/cache-how-to-premium-persistence)

Caso esteja usando o Cache Redis como um cache de dados temporário e não como um armazenamento persistente, essas recomendações podem não ser aplicadas. 

### <a name="search"></a>Search

**Provisione mais de uma réplica.** Use pelo menos duas réplicas para alta disponibilidade de leitura ou três para alta disponibilidade de leitura/gravação.

**Configure indexadores para implantações de várias regiões.** Se você tiver uma implantação de várias regiões, considere as opções para a continuidade na indexação.

  * Se a fonte de dados é com replicação geográfica, você geralmente deve apontar cada indexador de cada serviço do Azure Search regional para a respectiva réplica de fonte de dados local. No entanto, essa abordagem não é recomendada para grandes conjuntos de dados armazenados no Banco de Dados SQL do Azure. O motivo é que o Azure Search não pode executar a indexação incremental de réplicas de Banco de Dados SQL secundárias, somente de réplicas primárias. Em vez disso, aponte todos os indexadores para a réplica primária. Após um failover, aponte os indexadores do Azure Search para a nova réplica primária.  
  * Se a fonte de dados não é com replicação geográfica, aponte vários indexadores na fonte de dados, de forma que os serviços do Azure Search façam a indexação da fonte de dados de modo contínuo e independente. Para obter mais informações, consulte [Considerações de desempenho e otimização do Azure Search][search-optimization].

### <a name="storage"></a>Armazenamento

**Para dados de aplicativo, use RA-GRS (armazenamento com redundância geográfica com acesso de leitura).** O armazenamento de RA-GRS replica os dados para uma região secundária e fornece acesso somente leitura da região secundária. Se há uma interrupção de armazenamento na região primária, o aplicativo pode ler os dados da região secundária. Para obter mais informações, consulte [Replicação do Armazenamento do Azure](/azure/storage/storage-redundancy/).

**Para discos de VM, use os Managed Disks.** Os [Managed Disks][managed-disks] fornecem melhor confiabilidade para as VMs em um conjunto de disponibilidade porque os discos ficam suficientemente isolados entre si para evitar pontos únicos de falha. Além disso, os Managed Disks não estão sujeitos a limites de IOPS de VHDs criados em uma conta de armazenamento. Para saber mais, consulte [Gerenciar a disponibilidade de máquinas virtuais Windows no Azure][vm-manage-availability].

**Para o Armazenamento de Filas, crie uma fila de backup em outra região.** Para o armazenamento de fila, a utilidade de uma réplica somente leitura é limitada, porque não é possível enfileirar itens nem removê-los da fila. Em vez disso, crie uma fila de backup em uma conta de armazenamento em outra região. Se houver uma interrupção de armazenamento, o aplicativo poderá usar a fila de backup até que a região primária fique disponível novamente. Dessa forma, o aplicativo ainda poderá processar novas solicitações.  

### <a name="sql-database"></a>Banco de dados SQL

**Use a camada Standard ou Premium.** Essas camadas fornecem um período de Recuperação Pontual mais longo (35 dias). Para obter mais informações, consulte [Opções e desempenho de Banco de Dados SQL](/azure/sql-database/sql-database-service-tiers/).

**Habilite a auditoria do Banco de Dados SQL.** A auditoria pode ser usada para diagnosticar erros humanos ou ataques mal-intencionados. Para saber mais, confira [Introdução à Auditoria do Banco de Dados SQL](/azure/sql-database/sql-database-auditing-get-started/).

**Use a Replicação Geográfica Ativa**. Use a replicação geográfica ativa para criar uma réplica secundária legível em uma região diferente.  Se o seu banco de dados primário falhar ou simplesmente precisar ser colocado offline, faça um failover manual para qualquer o banco de dados secundário.  O banco de dados secundário permanece somente leitura até você fazer o failover.  Para saber mais, confira [Replicação Geográfica Ativa para o Banco de Dados SQL](/azure/sql-database/sql-database-geo-replication-overview/).

**Use fragmentação.** Considere o uso de fragmentação para particionar o banco de dados horizontalmente. A fragmentação pode proporcionar o isolamento de falhas. Para saber mais, confira [Expansão com o Banco de Dados SQL do Azure](/azure/sql-database/sql-database-elastic-scale-introduction/).

**Use a Recuperação Pontual para se recuperar de erro humano.**  A Recuperação Pontual retorna seu banco de dados para um ponto anterior no tempo. Para obter mais informações, consulte [Recuperar um Banco de Dados SQL do Azure usando backups de banco de dados automatizados][sql-restore].

**Use a restauração geográfica para recuperar-se de uma interrupção de serviço.** A restauração geográfica restaura um banco de dados de um backup com redundância geográfica.  Para obter mais informações, consulte [Recuperar um Banco de Dados SQL do Azure usando backups de banco de dados automatizados][sql-restore].

### <a name="sql-server-running-in-a-vm"></a>SQL Server em execução em uma VM

**Replicar o banco de dados.** Use os Grupos de Disponibilidade Always On do SQL Server para replicar o banco de dados. Fornece alta disponibilidade se uma instância do SQL Server falha. Para obter mais informações, consulte [Executar VMs do Windows para um aplicativo de N camadas](../reference-architectures/virtual-machines-windows/n-tier.md)

**Fazer backup do banco de dados**. Se você já estiver usando [Backup do Azure](https://azure.microsoft.com/documentation/services/backup/) para fazer backup de suas VMs, considere usar o [Backup do Azure para cargas de trabalho do SQL Server usando o DPM](/azure/backup/backup-azure-backup-sql/). Com essa abordagem, há uma função de administrador de backup para a organização e um procedimento de recuperação unificado para as VMs e o SQL Server. Caso contrário, use o [Backup gerenciado pelo SQL Server para o Microsoft Azure](https://msdn.microsoft.com/library/dn449496.aspx).

### <a name="traffic-manager"></a>Gerenciador de Tráfego

**Faça o failback manual.** Após um failover do Gerenciador de Tráfego, faça o failback manual em vez de automaticamente. Verifique se todos os subsistemas de aplicativo estão íntegros antes de fazer o failback.  Caso contrário, você pode criar uma situação em que o aplicativo fica alternando entre os data centers. Para obter mais informações, consulte [Executar VMs em várias regiões para ter alta disponibilidade](../reference-architectures/virtual-machines-windows/multi-region-application.md).

**Criar um ponto de extremidade de investigação de integridade.** Crie um ponto de extremidade personalizado que relata a integridade geral do aplicativo. Isso permite que o Gerenciador de Tráfego faça failover se qualquer caminho crítico falha, não apenas o front-end. O ponto de extremidade deve retornar um código de erro HTTP se qualquer dependência crítica está não íntegra ou inacessível. No entanto, não relate erros para serviços não críticos. Caso contrário, a investigação de integridade poderá disparar o failover quando ele não for necessário, criando falsos positivos. Para obter mais informações, consulte [Monitoramento e failover do ponto de extremidade do Gerenciador de Tráfego](/azure/traffic-manager/traffic-manager-monitoring/).

### <a name="virtual-machines"></a>Máquinas Virtuais

**Evite executar uma carga de trabalho de produção em uma única VM.** Uma única implantação de VM não é resiliente para manutenção (planejada ou não). Em vez disso, coloque várias VMs em um conjunto de disponibilidade ou [conjunto de dimensionamento de VM](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/), com um balanceador de carga na frente.

**Especifique um conjunto de disponibilidade ao provisionar a VM.** Atualmente, não há nenhuma maneira de adicionar uma VM a um conjunto de disponibilidade depois que a VM é provisionada. Quando você adiciona uma nova VM a um conjunto de disponibilidade existente, verifique se você criou um adaptador de rede para a VM e adicione o adaptador de rede ao pool de endereços de back-end no balanceador de carga. Caso contrário, o balanceador de carga não roteará o tráfego de rede para essa VM.

**Coloque cada camada de aplicativo em um conjunto de disponibilidade separado.** Em um aplicativo de N camadas, não coloque as VMs de camadas diferentes no mesmo conjunto de disponibilidade. VMs em um conjunto de disponibilidade são colocadas em FDs (domínios de falha) e UDs (domínios de atualização). No entanto, para obter o benefício de redundância de FDs e UDs, todas as VMs no conjunto de disponibilidade devem ser capazes de lidar com as mesmas solicitações de cliente.

**Escolha o tamanho de VM correto com base nos requisitos de desempenho.** Ao mover uma carga de trabalho existente para o Azure, comece com o tamanho da VM que mais se aproxima de seus servidores locais. Em seguida, meça o desempenho da carga de trabalho real com relação à CPU, memória e IOPS e ajuste o tamanho, se necessário. Isso ajuda a garantir que o aplicativo se comporte como esperado em um ambiente de nuvem. Além disso, se você precisar de várias NICs, esteja ciente do limite de NIC para cada tamanho.

**Use Managed Disks para VHDs.** Os [Managed Disks][managed-disks] fornecem melhor confiabilidade para as VMs em um conjunto de disponibilidade porque os discos ficam suficientemente isolados entre si para evitar pontos únicos de falha. Além disso, os Managed Disks não estão sujeitos a limites de IOPS de VHDs criados em uma conta de armazenamento. Para saber mais, consulte [Gerenciar a disponibilidade de máquinas virtuais Windows no Azure][vm-manage-availability].

**Instale aplicativos em um disco de dados, não no disco de SO.** Caso contrário, você poderá atingir o limite de tamanho do disco.

**Use o Backup do Azure para fazer backup de VMs.** Os backups protegem contra perda de dados acidental. Para obter mais informações, consulte [Proteger VMs do Azure com um cofre de serviços de recuperação](/azure/backup/backup-azure-vms-first-look-arm/).

**Habilite logs de diagnóstico**, incluindo métricas de integridade básicas, logs de infraestrutura e [diagnóstico de inicialização][boot-diagnostics]. O diagnóstico de inicialização poderá ajudar a diagnosticar uma falha de inicialização se sua VM entrar em um estado não inicializável. Para obter mais informações, consulte [Visão Geral dos Logs de Diagnóstico do Azure][diagnostics-logs].

**Use a Extensão AzureLogCollector.** (Somente VMs do Windows.) Essa extensão agrega logs da plataforma do Azure e carrega-os no Armazenamento do Azure, sem que o operador faça logon remotamente na VM. Para obter mais informações, consulte [Extensão do AzureLogCollector](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

### <a name="virtual-network"></a>Rede Virtual

**Para incluir endereços IP públicos na lista de permissões ou bloqueá-los, adicione um NSG à sub-rede.** Bloqueie o acesso de usuários mal-intencionados ou permita o acesso somente de usuários que têm o privilégio de acessar o aplicativo.  

**Crie uma investigação de integridade personalizada.** As investigações de integridade do balanceador de carga podem testar HTTP ou TCP. Se uma VM for executada em um servidor HTTP, a investigação HTTP será um melhor indicador do status de integridade do que uma investigação TCP. Para uma investigação HTTP, use um ponto de extremidade personalizado que relata a integridade geral do aplicativo, incluindo todas as dependências críticas. Para obter mais informações, veja [Visão geral do Azure Load Balancer](/azure/load-balancer/load-balancer-overview/).

**Não bloqueie a investigação de integridade.** A investigação de integridade do balanceador de carga é enviada de um endereço IP conhecido, 168.63.129.16. Não bloqueie o tráfego de ou para esse IP em nenhuma política de firewall ou regra de NSG (Grupo de Segurança de Rede). Bloquear a investigação de integridade faria com que o balanceador de carga remova a VM da rotação.

**Habilite o registro em log do balanceador de carga.** Os logs de mostram quantas muitas VMs no back-end não estão recebendo o tráfego de rede devido a respostas de investigação com falha. Para obter mais informações, consulte [Log Analytics para o Azure Load Balancer](/azure/load-balancer/load-balancer-monitor-log/).


<!-- links -->
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale/
[asynchronous-c-sharp]: /dotnet/articles/csharp/async
[availability-sets]:/azure/virtual-machines/virtual-machines-windows-manage-availability/
[azure-backup]: https://azure.microsoft.com/documentation/services/backup/
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[circuit-breaker]: ../patterns/circuit-breaker.md
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[fma]: ../resiliency/failure-mode-analysis.md
[resilient-deployment]: ../resiliency/index.md#resilient-deployment
[load-balancer]: /azure/load-balancer/load-balancer-overview/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[monitoring-and-diagnostics-guidance]: ../best-practices/monitoring.md
[resource-manager]: /azure/azure-resource-manager/resource-group-overview/
[retry-pattern]: ../patterns/retry.md
[retry-service-guidance]: ../best-practices/retry-service-specific.md
[search-optimization]: /azure/search/search-performance-optimization/
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[traffic-manager]: /azure/traffic-manager/traffic-manager-overview/
[traffic-manager-routing]: /azure/traffic-manager/traffic-manager-routing-methods/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
[vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview/
