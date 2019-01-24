---
title: Lista de verificação de DevOps
titleSuffix: Azure Design Review Framework
description: Lista de verificação que fornece diretrizes relacionadas a DevOps.
author: dragon119
ms.date: 01/10/2018
ms.topic: checklist
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: checklist
ms.openlocfilehash: 1a000c811cce57cc9b1fcda84d0eb7e2a1312aca
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486260"
---
# <a name="devops-checklist"></a>Lista de verificação de DevOps

DevOps é a integração do desenvolvimento, garantia de qualidade e operações de TI em uma cultura unificada e um conjunto de processos para entrega de software. Use esta lista de verificação como um ponto de partida para avaliar seu processo e cultura de DevOps.

## <a name="culture"></a>Cultura

**Assegure o alinhamento de negócios entre organizações e equipes.** Conflitos por recursos, finalidade, metas e prioridades dentro de uma organização podem ser um risco para operações bem-sucedidas. Verifique se as equipes de negócios, desenvolvimento e operações estão todas alinhadas.

**Verifique se toda a equipe compreende o ciclo de vida do software.** Sua equipe precisa entender o ciclo de vida geral do aplicativo e em qual parte do ciclo de vida o aplicativo está no momento. Isso ajuda a todos os membros da equipe a saber o que devem estar fazendo agora e para o que eles devem se planejar e se preparar no futuro.

**Reduza o tempo do ciclo.** Busque minimizar o tempo que leva para mover de ideias para software desenvolvido utilizável. Limite o tamanho e o escopo dos lançamentos individuais para manter a carga de teste baixa. Automatize os processos de build, teste, configuração e implantação sempre que possível. Elimine quaisquer obstáculos para comunicação entre os desenvolvedores, bem como aqueles entre os desenvolvedores e as operações.

**Examine e aprimore os processos.** Seus processos e procedimentos, automatizados e manuais, nunca são finais. Agende revisões regulares dos fluxos de trabalho, procedimentos e documentação atuais, com a meta de aperfeiçoamento contínuo.

**Faça planejamento proativo.** Planeje-se proativamente para falhas. Estabeleça processos para identificar rapidamente problemas quando eles ocorrem, escalone para os membros da equipe corretos para corrigi-los e confirme a resolução.

**Aprenda com as falhas.** As falhas são inevitáveis, mas é importante aprender com as falhas para evitar repeti-las. Se uma falha operacional ocorrer, faça a triagem do problema, documente a causa e a solução e compartilhe as lições que foram aprendidas. Sempre que possível, atualize os processos de build para detectar automaticamente esse tipo de falha no futuro.

**Otimize para velocidade e colete dados.** Todo aprimoramento planejado é uma hipótese. Trabalhe com os menores incrementos possíveis. Trate novas ideias como experimentos. Instrumente os experimentos de forma que você possa coletar dados de produção para avaliar a eficiência deles. Esteja preparado para falhar rapidamente se a hipótese estiver errada.

**Reserve tempo para o aprendizado.** Tanto êxitos quanto falhas fornecem boas oportunidades de aprendizado. Antes de passar para novos projetos, reserve tempo suficiente para reunir as lições importantes e verifique se as lições são absorvidas pela sua equipe. Dê também à equipe tempo para desenvolverem suas habilidades, experimentarem e aprenderem sobre novas ferramentas e técnicas.

**Documente as operações.** Documente todas as ferramentas, processos e tarefas automatizadas com o mesmo nível de qualidade que seu código de produto. Documente a arquitetura e o design atual dos sistemas aos quais você dá suporte, juntamente com processos de recuperação e outros procedimentos de manutenção. Concentre-se nas etapas que você realmente executa e não em processos teoricamente ideais. Examine e atualize a documentação regularmente. Para o código, verifique se os comentários significativos são incluídos, especialmente em APIs públicas e use ferramentas para gerar automaticamente a documentação de código sempre que possível.

**Compartilhe conhecimento.** A documentação é útil apenas se as pessoas sabem que ela existe e se podem encontrá-la. Verifique se a documentação está organizada e se pode ser encontrada com facilidade. Seja criativo: Use falas (apresentações informais), vídeos ou boletins informativos para compartilhar conhecimento.

## <a name="development"></a>Desenvolvimento

**Fornece ambientes semelhantes aos de produção aos desenvolvedores.** Se os ambientes de desenvolvimento e teste não corresponderem ao ambiente de produção, será difícil testar e diagnosticar problemas. Portanto, mantenha os ambientes de desenvolvimento e teste o mais próximos possível do ambiente de produção. Verifique se os dados de teste são consistentes com os dados usados em produção, mesmo que se tratem de dados de exemplo e não de dados reais de produção (por motivos de conformidade ou de privacidade). Planeje gerar dados de teste de exemplo e torná-los anônimos.

**Verifique se todos os membros da equipe autorizada podem provisionar a infraestrutura e implantar o aplicativo.** A configuração de recursos de produção e a implantação do aplicativo não devem envolver tarefas manuais complicadas ou conhecimento técnico detalhado do sistema. Qualquer pessoa com as permissões corretas deve ser capaz de criar ou implantar recursos de produção sem precisar da equipe de operações.

> Essa recomendação não implica que qualquer pessoa pode enviar atualizações dinâmicas por push para a implantação de produção. Ela se trata de reduzir o atrito entre as equipes de desenvolvimento de garantia de qualidade para criar ambientes semelhantes aos de produção.

**Instrumente o aplicativo para insight.** Para entender a integridade do seu aplicativo, você precisa saber como ele está desempenhando e se está enfrentando erros ou problemas. Sempre inclua a instrumentação como um requisito de design e compile a instrumentação no aplicativo desde o início. A instrumentação deve incluir o log de eventos para análise da causa raiz, mas também a telemetria e as métricas para monitorar a integridade geral e o uso do aplicativo.

**Acompanhe a dívida técnica.** Em muitos projetos, os cronogramas de lançamento podem ser priorizados sobre a qualidade do código em algum grau. Sempre acompanhe quando isso ocorre. Documente quaisquer atalhos ou outras implementações não ideais e agende um horário no futuro para revisitar esses problemas.

**Considere a possibilidade de enviar atualizações por push diretamente para a produção.** Para reduzir o tempo do ciclo de lançamento geral, considere a possibilidade de enviar por push confirmações de código adequadamente testadas diretamente para a produção. Use [alternâncias de recursos][feature-toggles] para controlar quais recursos estão habilitados. Isso permite que você se mova do desenvolvimento para o lançamento rapidamente, usando os botões para habilitar ou desabilitar os recursos. As alternâncias também são úteis para executar testes como [lançamentos canário][canary-release], em que um recurso específico é implantado em um subconjunto do ambiente de produção.

## <a name="testing"></a>Testando

**Automatize o teste.** Testar manualmente o software é entediante e suscetível a erros. Automatize tarefas comuns de teste e integre os testes em seus processos de build. Testes automatizados garantem cobertura e reprodutibilidade de teste consistentes. Testes de interface do usuário integrados também devem ser executados por uma ferramenta automatizada. O Azure oferece recursos de desenvolvimento e teste que podem lhe ajudar a configurar e executar testes. Para obter mais informações, consulte [Desenvolvimento e teste][dev-test].

**Teste em busca de falhas.** Se um sistema não pode se conectar a um serviço, como ele responde? Ele pode se recuperar depois que o serviço está disponível novamente? Torne o teste de injeção de falha uma parte padrão da análise em ambientes de preparo e de teste. Quando as práticas e o processo de teste estiverem maduros, considere a execução desses testes em produção.

**Teste em produção.** O processo de lançamento não termina com a implantação na produção. Estabeleça testes para garantir que o código implantado funciona conforme o esperado. Para implantações que raramente são atualizadas, agende o teste em produção como parte comum da manutenção.

**Automatize testes de desempenho para identificar problemas de desempenho com antecedência.** O impacto de um problema de desempenho sério pode ser tão grave quanto um bug no código. Embora testes funcionais automatizados possam impedir erros de aplicativo, eles podem não detectar problemas de desempenho. Defina metas de desempenho aceitáveis para métricas como latência, tempos de carregamento e uso de recursos. Inclua testes de desempenho automatizados em seu pipeline de lançamento para verificar se o aplicativo satisfaz essas metas.

**Execute testes de capacidade.** Um aplicativo pode funcionar bem em condições de teste e, em seguida, ter problemas em produção devido a limitações de escala ou de recurso. Sempre defina os limites de capacidade e de uso máximos esperados. Teste para verificar se o aplicativo pode lidar com esses limites, mas também teste o que acontece quando esses limites são excedidos. O teste de capacidade deve ser executado em intervalos regulares.

Após o lançamento inicial, você deve executar testes de capacidade e de desempenho sempre que atualizações são feitas ao código de produção. Use os dados históricos para ajustar testes e para determinar quais tipos de testes precisam ser realizados.

**Execute testes de penetração de segurança automatizados.** Verificar se seu aplicativo é seguro é tão importante quanto testar qualquer outra funcionalidade. Torne os testes de penetração automatizados uma parte padrão do processo de build e de implantação. Agende testes de segurança e verificações de vulnerabilidade regulares em aplicativos implantados, monitorando em busca de ataques e de portas e pontos de extremidade abertos. Testes automatizados não removem a necessidade de exames de segurança detalhados em intervalos regulares.

**Realize o teste de continuidade de negócios automatizado.** Desenvolva testes para continuidade de negócios de grande escala, inclusive o failover e recuperação de backup. Configure processos automatizados para executar esses testes regularmente.

## <a name="release"></a>Versão

**Automatize as implantações.** Automatize a implantação do aplicativo para ambientes de teste, de preparo e de produção. A automação permite implantações mais rápidas e confiáveis e garante implantações consistentes em qualquer ambiente compatível. Ela remove o risco de erro humano causado por implantações manuais. Ela também facilita o agendamento de lançamentos em horários adequados, para reduzir os efeitos de um potencial tempo de inatividade.

**Use a integração contínua.** A CI (integração contínua) é a prática de mesclar todo o código de desenvolvedor em uma base de código central em um agendamento regular e, em seguida, executar automaticamente processos padrão de build e de teste. A CI garante que toda a equipe possa trabalhar em uma base de código simultaneamente, sem conflitos. Ela também garante que defeitos de código sejam encontrados o mais cedo possível. De preferência, o processo de CI deve ser executado toda vez que o código é confirmado ou que o check-in dele é feito. Na pior das hipóteses, ele deve ser executado uma vez por dia.

> Considerar a adoção um [modelo de desenvolvimento com base em tronco][trunk-based]. Nesse modelo, os desenvolvedores são confirmados para um único branch (o tronco). Há um requisito que confirmações nunca interrompam o build. Este modelo facilita CI, porque todo o trabalho de recurso é feito no tronco e conflitos de mesclagem são resolvidos quando a confirmação ocorre.

**Considere usar a entrega contínua.** A CD (entrega contínua) é a prática de garantir que o código esteja sempre pronto para implantar ao se compilar, testar e implantar automaticamente o código em ambientes semelhantes ao de produção. Adicionar a entrega contínua para criar um pipeline de CI/CD completo ajudará você a detectar defeitos no código assim que possível e garantirá que atualizações testadas corretamente possam ser lançadas em um período muito curto.

> A *implantação* contínua é mais um processo que pega automaticamente quaisquer atualizações que tenham passado pelo pipeline de CI/CD e as implanta na produção. A implantação contínua requer testes automáticos robustos e planejamento de processo avançado e pode não ser apropriada para todas as equipes.

**Faça alterações em incrementos pequenos.** Alterações de código grandes tem um maior potencial para introduzir bugs. Sempre que possível, mantenha as alterações pequenas. Isso limita os possíveis efeitos de cada alteração e torna mais fácil de entender e depurar eventuais problemas.

**Controle a exposição a alterações.** Verifique se você está no controle de quando as atualizações estão visíveis para os usuários finais. Considere o uso de alternâncias de recurso para controlar quando os recursos são habilitados para os usuários finais.

**Implemente estratégias de gerenciamento de versão para reduzir o risco de implantação.** Implantar uma atualização de aplicativo para produção sempre envolve algum risco. Para minimizar esse risco, use estratégias como o uso de [lançamentos canário][canary-release] ou [implantações de tipo azul/verde][blue-green] para implantar atualizações para um subconjunto de usuários. Verifique se a atualização funciona conforme o esperado e, em seguida, distribua a atualização para o restante do sistema.

**Documente todas as alterações.** Atualizações secundárias e alterações de configuração podem ser uma fonte de confusão e de conflito no controle de versão. Mantenha sempre um registro claro das alterações, independentemente de quão pequenas. Registre em log tudo que for alterado, incluindo patches aplicados, alterações de política e alterações de configuração. (Não inclua dados confidenciais nesses logs. Por exemplo, registre em log que uma credencial foi atualizada e quem fez a alteração, mas não registre as credenciais atualizadas.) O registro das alterações deve estar visível para toda a equipe.

**Automatize as Implantações.** Automatize todas as implantações e estabeleça sistemas para detectar eventuais problemas durante a distribuição. Tenha um processo de mitigação para preservar o código existente e os dados de produção, antes que a atualização substitua-os em todas as instâncias de produção. Tenha uma forma automatizada de efetuar roll forward das correções ou de reverter as alterações.

**Considere tornar a infraestrutura imutável.** Infraestrutura imutável é o princípio segundo o qual não se deve modificar a infraestrutura após sua implantação para produção. Caso contrário, você pode entrar em um estado em que foram aplicadas alterações ad hoc, tornando difícil saber exatamente o que foi alterado. A infraestrutura imutável funciona substituindo-se servidores inteiros como parte de qualquer nova implantação. Isso permite que o código e o ambiente de hospedagem sejam testado e implantados como um bloco. Uma vez implantados, os componentes da infraestrutura não são modificados até o próximo ciclo de build de implantação.

## <a name="monitoring"></a>Monitoramento

**Torne os sistemas observáveis.** A equipe de operações deve ter sempre visibilidade clara da integridade e do status de um sistema ou serviço. Configure pontos de extremidade de integridade externos para monitorar o status e verifique se os aplicativos estão codificados para instrumentar as métricas de operações. Use um esquema comum e consistente que permita correlacionar eventos entre sistemas. [Diagnóstico do Azure][azure-diagnostics] e [Application Insights][app-insights] são o método padrão de acompanhamento da integridade e do status de recursos do Azure. O Microsoft [Operation Management Suite][oms] também fornece monitoramento centralizado e gerenciamento de soluções de nuvem ou híbridas.

**Agregue e correlacione os logs e métricas**. Um sistema de telemetria corretamente instrumentado fornecerá uma grande quantidade de dados de desempenho brutos e logs de eventos. Verifique se os dados de telemetria e de log são processados e correlacionados em um curto período de tempo, para que a equipe de operações sempre tenha uma imagem atualizada da integridade do sistema. Organize e exiba dados de maneiras que proporcionem uma exibição coesa de eventuais problemas para que, sempre que possível, fique claro quando os eventos estão correlacionados.

> Consulte sua política de retenção da empresa para requisitos de como os dados são processados e por quanto tempo eles devem ser armazenados.

**Implemente notificações e alertas automatizados.** Configure ferramentas de monitoramento como [Azure Monitor][azure-monitor] para detectar padrões ou condições que indicam problemas potenciais ou atuais e envie alertas para os membros da equipe que podem solucioná-los. Ajuste os alertas para evitar falsos positivos.

**Monitore ativos e recursos quanto à expiração.** Alguns recursos e ativos como certificados expiram após um determinado período de tempo. Verifique se você está controlando quais ativos expiraram, quando eles expiram e quais serviços ou recursos dependem deles. Use processos automatizados para monitorar esses ativos. Notifique a equipe de operações antes da expiração de um ativo e escale se a expiração ameaça interromper o aplicativo.

## <a name="management"></a>Gerenciamento

**Automatize as tarefas de operações.** Manipular processos de operações repetitivas manualmente é propenso a erros. Automatize essas tarefas sempre que possível para garantir qualidade e uma execução consistente. O código que implementa a automação deve ter controle de versão no controle do código-fonte. Assim como acontece com qualquer outro código, ferramentas de automação devem ser testadas.

**Adote uma abordagem de infraestrutura como código para provisionamento.** Minimize a quantidade de configuração manual necessária para provisionar recursos. Em vez disso, use scripts e modelos do [Azure Resource Manager][resource-manager]. Mantenha os scripts e modelos no controle do código-fonte, bem como qualquer outro código que você mantenha.

**Considere o uso de contêineres.** Contêineres fornecem uma interface baseada em pacote padrão para implantar aplicativos. Usando contêineres, um aplicativo é implantado usando pacotes independentes que incluem qualquer software, dependências e arquivos necessários para executar o aplicativo, o que simplifica o processo de implantação.

Contêineres também criam uma camada de abstração entre o aplicativo e o sistema operacional subjacente, o que proporciona consistência entre ambientes. Essa abstração também pode isolar um contêiner de outros processos ou aplicativos em execução em um host.

**Implemente resiliência e autorrecuperação.** A resiliência é a capacidade de um aplicativo se recuperar de falhas. Estratégias para garantir resiliência incluem fazer novas tentativas em caso de falhas transitórias e fazer failover para uma instância secundária ou até mesmo outra região. Para obter mais informações, consulte [Desenvolvimento de aplicativos resilientes para o Azure][resiliency]. Instrumente seus aplicativos para que os problemas sejam reportados imediatamente e você possa gerenciar interrupções ou outras falhas de sistema.

**Tenha um manual de operações.** Um manual de operações ou *runbook* documenta os procedimentos e as informações de gerenciamento necessárias para a equipe de operações manter um sistema. Documente também quaisquer cenários de operações e planos de mitigação que possam surgir durante uma falha ou outra interrupção do seu serviço. Crie essa documentação durante o processo de desenvolvimento e mantenha-a atualizada daí em diante. Este é um documento dinâmico e deve ser revisado, testado e aprimorado regularmente.

Documentação compartilhada é algo essencial. Incentivamos que os membros da equipe contribuam e compartilhem conhecimento. A equipe inteira deve ter acesso a documentos. Torna mais fácil para qualquer pessoa da equipe ajudar a manter os documentos atualizados.

**Documente procedimentos de permanência.** Verifique se procedimentos, agendas e tarefas de permanência são documentados e compartilhados com todos os membros da equipe. Mantenha essas informações atualizadas em todos os momentos.

**Documente procedimentos de escalonamento para dependências de terceiros.** Se seu aplicativo depende de serviços de terceiros externos que você não controla diretamente, você deve ter um plano para lidar com interrupções. Crie documentação para seus processos de mitigação planejados. Inclua contatos de suporte e caminhos de escalonamento.

**Use gerenciamento de configuração.** Alterações de configuração devem ser planejadas, visíveis para as operações e registradas. Isso pode assumir a forma de um banco de dados de gerenciamento de configuração ou uma abordagem de configuração como código. A configuração deve ser examinada regularmente para garantir que o que é esperado esteja realmente em vigor.

**Obtenha um plano de suporte do Azure e entenda o processo.** O Azure oferece um número de [planos de suporte][azure-support-plans]. Determinar o plano certo para suas necessidades e verifique se toda a equipe sabe como usá-lo. Os membros da equipe devem entender os detalhes do plano, como funciona o processo de suporte e como abrir um tíquete de suporte com o Azure. Se você estiver prevendo um evento de grande escala, o suporte do Azure pode ajudar a aumentar os limites do serviço. Para obter mais informações, consulte as [Perguntas Frequentes do Suporte do Azure](https://azure.microsoft.com/support/faq/).

**Siga os princípios de privilégios mínimos ao conceder acesso aos recursos.** Gerencie o acesso aos recursos cuidadosamente. O acesso deve ser negado por padrão, a menos que um usuário receba acesso a um recurso explicitamente. Conceda a um usuário apenas o acesso ao que ele precisa para concluir suas tarefas. Acompanhe as permissões de usuário e realize auditorias de segurança frequentes.

**Use o controle de acesso baseado em função.** Atribuir contas de usuário e o acesso a recursos não deve ser um processo manual. Use o RBAC ([controle de acesso baseado em função][rbac]) para conceder acesso com base em identidades e grupos do [Azure Active Directory][azure-ad].

**Use um sistema de acompanhamento de bugs para acompanhar problemas.** Sem uma boa maneira de acompanhar problemas, é fácil deixar itens passarem despercebidos, duplicar o trabalho ou introduzir problemas adicionais. Não confie na comunicação informal para acompanhar o status de bugs. Use uma ferramenta de acompanhamento de bugs para registrar detalhes sobre problemas, atribuir recursos para solucioná-los e fornecer um log de auditoria de progresso e status.

**Gerencie todos os recursos em um sistema de gerenciamento de alterações.** Todos os aspectos do processo de DevOps devem ser incluídos em um sistema de gerenciamento e de controle de versão para que as alterações possam ser facilmente acompanhadas e auditadas. Isso inclui código, infraestrutura, configuração, documentação e scripts. Trate todos esses tipos de recursos como código durante o processo de build/teste/análise.

**Use listas de verificação.** Crie listas de verificação de operações para garantir que os processos sejam seguidos. É comum ignorar alguma coisa em um manual grande, mas seguir uma lista de verificação pode forçar a atenção a detalhes que poderiam, caso contrário, passar despercebidos. Mantenha as listas de verificação e procure continuamente maneiras de automatizar tarefas e simplificar processos.

Para obter mais informações sobre DevOps, consulte [Quais as novidades no DevOps?][what-is-devops] no site do Visual Studio.

<!-- links -->

[app-insights]: /azure/application-insights/
[azure-ad]: https://azure.microsoft.com/services/active-directory/
[azure-diagnostics]: /azure/monitoring-and-diagnostics/azure-diagnostics
[azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview
[azure-support-plans]: https://azure.microsoft.com/support/plans/
[blue-green]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[canary-release]:https://martinfowler.com/bliki/CanaryRelease.html
[dev-test]: https://azure.microsoft.com/solutions/dev-test/
[feature-toggles]: https://www.martinfowler.com/articles/feature-toggles.html
[oms]: https://www.microsoft.com/cloud-platform/operations-management-suite
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resiliency]: ../resiliency/index.md
[resource-manager]: /azure/azure-resource-manager/
[trunk-based]: https://trunkbaseddevelopment.com/
[what-is-devops]: https://www.visualstudio.com/learn/what-is-devops/
