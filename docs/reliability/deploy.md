---
title: Implantando aplicativos do Azure para resiliência e disponibilidade
description: Uma visão geral das técnicas para criar implantações confiáveis no Azure
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: d8df3fe1c3353c60462959a625ba13ae47b96cf8
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646479"
---
# <a name="deploying-azure-applications-for-resiliency-and-availability"></a>Implantando aplicativos do Azure para resiliência e disponibilidade

Como provisionar e atualizar recursos do Azure, o código do aplicativo e as definições de configuração, um processo repetível e previsível o ajudará a evitar erros e tempo de inatividade. Recomendamos que os processos automatizados de implantação que você pode executar sob demanda e executar novamente se algo falhar. Depois que seus processos de implantação estão funcionando sem problemas, documentação do processo pode mantê-los dessa forma.

## <a name="automate-processes"></a>Automatizar processos

Para ativar recursos sob demanda, implantar soluções rapidamente, minimizar o erro humano e produzir resultados consistentes e reproduzíveis, certifique-se de automatizar implantações e atualizações.

### <a name="automate-as-many-processes-as-possible"></a>Automatizar processos de tantos quanto possível

Os processos de implantação mais confiáveis são automatizados e *idempotente* &mdash; ou seja, repetíveis para produzir os mesmos resultados.

- Para automatizar o provisionamento de recursos do Azure, você pode usar [Terraform](/azure/virtual-machines/windows/infrastructure-automation#terraform), [Ansible](/azure/virtual-machines/windows/infrastructure-automation#ansible), [Chef](/azure/virtual-machines/windows/infrastructure-automation#chef), [Puppet](/azure/virtual-machines/windows/infrastructure-automation#puppet), [Azure PowerShell](/powershell/azure/overview), [interface de linha de comando (CLI) do Azure](/cli/azure), ou [modelos do Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview#template-deployment).
- Para configurar as VMs, você pode usar [cloud_init](/azure/virtual-machines/windows/infrastructure-automation#cloud-init) (para VMs do Linux) ou [configuração de estado de automação do Azure](/azure/automation/automation-dsc-overview) (DSC).
- Para automatizar a implantação do aplicativo, você pode usar [serviços do Azure DevOps](/azure/virtual-machines/windows/infrastructure-automation#azure-devops-services), [Jenkins](/azure/virtual-machines/windows/infrastructure-automation#jenkins), ou outras soluções de CI/CD.

Como prática recomendada, crie um repositório de scripts de automação categorizados para acesso rápido, documentada com explicações de parâmetros e exemplos de uso de script. Mantenha esta documentação em sincronia com suas implantações do Azure e designe uma pessoa primária para gerenciar o repositório.

Scripts de automação também podem ativar recursos sob demanda para recuperação de desastres.

### <a name="use-code-to-provision-and-configure-infrastructure"></a>Use o código para provisionar e configurar a infraestrutura

Essa prática, chamada *infraestrutura como código,* pode usar uma abordagem declarativa ou imperativa (ou uma combinação de ambos).

- [Modelos do Resource Manager](/azure/azure-resource-manager/resource-group-overview#template-deployment) são um exemplo de uma abordagem declarativa.
- [PowerShell](/powershell/azure/overview) scripts são um exemplo de abordagem imperativa.

### <a name="practice-immutable-infrastructure"></a>Infraestrutura imutável de prática

Em outras palavras, não modifique infraestrutura após sua implantação para produção. Depois que tiverem sido aplicadas alterações ad hoc, você pode não saber exatamente o que mudou, portanto, pode ser difícil de solucionar problemas do sistema.

### <a name="automate-and-test-deployment-and-maintenance-tasks"></a>Automatizar e testar tarefas de implantação e manutenção

 Aplicativos distribuídos consistem em várias partes que devem trabalhar em conjunto. Implantação deve tirar vantagem dos mecanismos comprovados, como scripts, que podem atualizar e validar a configuração e automatizar o processo de implantação. Teste todos os processos totalmente para garantir que os erros não causam tempo de inatividade adicional.

### <a name="implement-deployment-security-measures"></a>Implementar medidas de segurança de implantação

Todas as ferramentas de implantação devem incorporar as restrições de segurança para proteger o aplicativo implantado. Definir e impor políticas de implantação com cuidado e minimizar a necessidade de intervenção humana.

## <a name="deploy-to-multiple-regions-and-instances"></a>Implantar em várias regiões e instâncias

Um aplicativo que depende de uma única instância de um serviço cria um ponto único de falha. Para melhorar a resiliência e escalabilidade, provisione várias instâncias.

- Para o [Serviço de Aplicativo do Azure](/azure/app-service/app-service-value-prop-what-is/), selecione um [Plano do Serviço de Aplicativo](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/) que ofereça várias instâncias.
- Para [serviços de nuvem do Azure](/azure/cloud-services/cloud-services-choose-me), configure cada uma das suas funções para usar [várias instâncias](/azure/cloud-services/cloud-services-choose-me/#scaling-and-management).
- Para [máquinas virtuais do Azure](/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json), certifique-se de que sua arquitetura inclui mais de uma VM e que cada VM está incluída em uma [conjunto de disponibilidade](/azure/virtual-machines/virtual-machines-windows-manage-availability/).

### <a name="consider-deploying-across-multiple-regions"></a>Considere a implantação em várias regiões

É recomendável implantar tudo, mas os aplicativos menos críticos e os serviços de aplicativo em várias regiões. Se seu aplicativo for implantado em uma única região, no caso raro que toda a região fica indisponível, o aplicativo também estará indisponível.

Se você optar por implantar em uma única região, considere se preparando para reimplantar para uma região secundária, como uma resposta a uma falha inesperada.

### <a name="redeploy-to-a-secondary-region"></a>Reimplantar para uma região secundária

Se você executar aplicativos e bancos de dados em uma única região primária com nenhuma replicação, sua estratégia de recuperação pode ser reimplantar em outra região. Essa solução é acessível, mas mais apropriado para aplicativos não críticos que podem tolerar tempos de recuperação.

Se você escolher essa estratégia, automatizar o processo de reimplantação tanto quanto possível e incluem cenários de reimplantação em seu teste de resposta de desastres.

Para automatizar o processo de reimplantação, considere o uso de [Azure Site Recovery](/azure/site-recovery/).

## <a name="use-staging-and-production-environments"></a>Usar ambientes de preparo e produção

Com bom uso de ambientes de preparo e produção, você pode enviar atualizações para o ambiente de produção de uma forma altamente controlada e minimizar as interrupções de problemas de implantação não previstos.

- [*Implantação "Blue-green"*](https://martinfowler.com/bliki/BlueGreenDeployment.html) envolve a implantação de uma atualização em um ambiente de produção que é separado do aplicativo ao vivo. Após validar a implantação, alterne o roteamento de tráfego para a versão atualizada. Uma forma de fazer isso é usar o [slots de preparo](/azure/app-service/web-sites-staged-publishing) disponíveis no serviço de aplicativo do Azure para preparar uma implantação antes de movê-lo para produção.
- [*Versões canário*](https://martinfowler.com/bliki/CanaryRelease.html) são semelhantes às implantações "blue-green". Em vez de alternar todo o tráfego para o aplicativo atualizado, você pode rotear apenas uma pequena parte do tráfego para a nova implantação. Se houver um problema, reverta para a implantação antiga. Caso contrário, gradualmente mais tráfego de rota para a nova versão. Se você estiver usando o serviço de aplicativo do Azure, você pode usar o teste no recurso de produção para gerenciar uma versão canário.

## <a name="create-a-rollback-plan-for-deployment-and-updates"></a>Criar um plano de reversão para a implantação e atualizações

Se uma implantação falhar, seu aplicativo pode se tornar indisponível. Para minimizar o tempo de inatividade, projete um processo de reversão para voltar para uma versão válida do último. Inclua uma estratégia para reverter as alterações em bancos de dados e quaisquer outros serviços que seu aplicativo depende.

Se você estiver usando o serviço de aplicativo do Azure, você pode configurar um slot de sites de último BOM e usá-lo para fazer a reversão de uma implantação de aplicativo de API ou web.

## <a name="log-and-audit-deployments"></a>Implantações de log e auditoria

Para capturar como possível de informações muito específicos da versão, implemente uma estratégia de registro em log robusta. Se você usar técnicas de implantação de preparo, mais de uma versão do seu aplicativo será executado na produção. Se ocorrer um problema, determine qual versão está causando a ele.

## <a name="document-release-processes"></a>Processos de lançamento do documento

Sem a documentação do processo de versão detalhadas, um operador pode implantar uma atualização inválida ou pode definir configurações incorretamente para seu aplicativo. Defina e documente claramente seu processo de lançamento e verifique se ele está disponível para toda a equipe de operações.

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Monitorar a integridade de aplicativo para confiabilidade](./monitoring.md)
