---
title: Executar uma VM do Windows no Azure
description: Como executar uma VM do Windows no Azure, atentando-se para a escalabilidade, resiliência, capacidade de gerenciamento e segurança.
author: telmosampaio
ms.date: 04/03/2018
ms.openlocfilehash: a20359f90e7b20486defce3110b2db6f7e0027ba
ms.sourcegitcommit: 25bf02e89ab4609ae1b2eb4867767678a9480402
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/14/2018
ms.locfileid: "45584690"
---
# <a name="run-a-windows-vm-on-azure"></a>Executar uma VM do Windows no Azure

Esse artigo descreve um conjunto de práticas comprovadas para executar uma VM (máquina virtual) no Azure. Ela inclui recomendações para provisionar a VM juntamente com os componentes de rede e armazenamento. [**Implante essa solução.**](#deploy-the-solution)

![[0]][0]

## <a name="components"></a>Componentes

O provisionamento de uma VM do Azure requer alguns componentes adicionais além da própria VM, incluindo recursos de rede, armazenamento e computação.

* **Grupo de recursos.** Um [grupo de recursos][resource-manager-overview] é um contêiner lógico que armazena os recursos relacionados ao Azure. Em geral, recursos de grupo baseados em seu tempo de vida e que vão gerenciá-los. 

* **VM**. Você pode provisionar uma VM a partir de uma lista de imagens publicadas, de um arquivo gerenciado personalizado ou um arquivo de VHD (disco rígido virtual) carregado no armazenamento Blobs do Azure.

* **Managed Disks**. Os [Azure Managed Disks][managed-disks] simplificam o gerenciamento de disco ao manipular o armazenamento para você. O disco de OS é um VHD armazenado no [Armazenamento do Microsoft Azure ][azure-storage], de modo que ele persiste mesmo quando o computador host está desligado. Também é recomendável criar um ou mais [discos de dados][data-disk], que são VHDs persistentes usados para dados de aplicativo.

* **Disco temporário.** A VM é criada com um disco temporário (a unidade `D:` no Windows). Esse disco é armazenado em uma unidade física no computador host. Ele *não* é salvo no Armazenamento do Microsoft Azure e pode ser excluído durante a reinicialização e outros eventos de ciclo de vida da VM. Use esse disco somente para dados temporários, como arquivos de paginação ou de permuta.

* **Rede virtual (VNet).** Cada VM do Azure é implantada em uma VNet que pode ser segmentada em várias sub-redes.

* **NIC (adaptador de rede)**. A NIC permite que a VM se comunique com a rede virtual.  

* **Endereço IP público.** Um endereço IP público é necessário para se comunicar com a VM &mdash; por exemplo, sobre RDP (área de trabalho remota).  

* **DNS do Azure**. [DNS do Azure][azure-dns] é um serviço de hospedagem para domínios DNS, que fornece resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure.

* **NSG (grupo de segurança de rede)**. Os [Grupos de segurança de rede][nsg] são utilizados para permitir ou recusar o tráfego de rede a VMs. Os NSGs podem ser associados com sub-redes ou com instâncias VM individuais. 

* **Diagnósticos.** O registro de diagnóstico é crucial para o gerenciamento e solução de problemas da VM.

## <a name="vm-recommendations"></a>Recomendações de VM

O Azure oferece vários tamanhos de máquinas virtuais diferentes. Para obter mais informações, confira [Tamanhos das máquinas virtuais no Azure][virtual-machine-sizes]. Se você estiver movendo uma carga de trabalho existente para o Azure, deverá começar com o tamanho da VM que mais se aproxima de seus servidores locais. Em seguida, meça o desempenho da carga de trabalho real com relação à CPU, memória e operações de entrada/saída de disco e ajuste o tamanho conforme necessário. Se você precisar de várias NICs para sua VM, esteja ciente de que um número máximo de NICs está definido para cada [tamanho de VM][vm-size-tables].

Em geral, escolha a região do Azure que esteja mais próxima de seus usuários internos ou clientes. No entanto, nem todos os tamanhos de VM estão disponíveis em todas as regiões. Para obter mais informações, consulte [Serviços por região][services-by-region]. Para obter uma lista dos tamanhos de VM disponíveis em uma região específica, execute o seguinte comando a partir da interface de linha de comando do Azure (CLI):

```
az vm list-sizes --location <location>
```

Para obter informações sobre como escolher uma imagem de VM publicada, consulte [Localizar imagens de VM do Windows][select-vm-image].

Habilite o monitoramento e diagnóstico, incluindo métricas de integridade básicas, logs de infraestrutura de diagnóstico e [diagnóstico de inicialização][boot-diagnostics]. O diagnóstico de inicialização poderá ajudar a diagnosticar uma falha de inicialização se sua VM entrar em um estado não inicializável. Para saber mais, confira [Habilitar monitoramento e diagnóstico][enable-monitoring].  

## <a name="disk-and-storage-recommendations"></a>Recomendações de disco e de armazenamento

Para um melhor desempenho de E/S de disco, recomendamos o [Armazenamento Premium][premium-storage], que armazena dados em SSDs (unidades de estado sólido). O custo é baseado na capacidade do disco provisionado. O IOPS e a taxa de transferência (ou seja, a taxa de transferência de dados) também dependem do tamanho do disco. Portanto, ao provisionar um disco, considere todos os três fatores (capacidade, IOPS e taxa de transferência). 

Também é recomendável usar [Managed Disks][managed-disks]. Os discos gerenciados não exigem uma conta de armazenamento. Você simplesmente especifica o tamanho e o tipo de disco e é implantado como um recurso altamente disponível.

Adicione um ou mais discos de dados. Quando você cria um VHD, ele não está formatado. Faça logon na VM para formatar o disco. Quando possível, instale aplicativos em um disco de dados, não no disco do sistema operacional. Algumas aplicações legadas podem precisar instalar componentes na unidade C:. Nesse caso, você pode [redimensionar o disco de SO][resize-os-disk] utilizando o PowerShell.

Crie uma conta de armazenamento para manter os logs de diagnóstico. Uma conta LRS (armazenamento com redundância local) padrão é suficiente para os logs de diagnóstico.

> [!NOTE]
> Se você não estiver usando Managed Disks, crie contas de armazenamento do Azure separadas para que cada VM contenha os VHDs (discos rígidos virtuais), evitando, assim, atingir os [limites (de IOPS)][vm-disk-limits] para contas de armazenamento. Esteja ciente dos limites de E/S totais da conta de armazenamento. Para saber mais, confira [limites de disco da máquina virtual][vm-disk-limits].


## <a name="network-recommendations"></a>Recomendações de rede

Esse endereço IP público pode ser dinâmico ou estático. O padrão é dinâmico.

* Reserve um [endereço IP estático][static-ip] se precisar de um endereço IP fixo que não mudará, &mdash;por exemplo, se precisar criar um registro 'A' de DNS ou adicionar o IP endereço para uma lista segura.
* Você também pode criar um FQDN (nome de domínio totalmente qualificado) para o endereço IP. Em seguida, é possível registrar um [registro CNAME][cname-record] no DNS que aponta para o FQDN. Para saber mais, consulte [Criar um nome de domínio totalmente qualificado no Portal do Azure][fqdn].

Todos os NSGs contêm um conjunto de [regras padrão][nsg-default-rules], incluindo uma regra que bloqueia todo o tráfego de Internet de entrada. As regras padrão não podem ser excluídas, mas outras regras podem substituí-las. Para habilitar o tráfego de Internet, crie regras que permitam o tráfego de entrada em portas específicas &mdash; por exemplo, a porta 80 para HTTP.

Para habilitar o RDP, adicione uma regra NSG que permita o tráfego de entrada na porta TCP 3389.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Você pode escalar uma VM vertical ou horizontalmente [alterando o tamanho da VM][vm-resize]. Para aumentar horizontalmente, coloque duas ou mais VMs atrás de um balanceador de carga. Para obter mais informações, consulte a [Arquitetura de referência de N camadas](./n-tier-sql-server.md).

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Para obter maior disponibilidade, implante várias VMs em um conjunto de disponibilidade. Isso também fornece [SLA (Contrato de Nível de Serviço)][vm-sla] mais elevado.

Sua VM pode ser afetada por uma [manutenção planejada][planned-maintenance] ou [manutenção não planejada][manage-vm-availability]. Você pode usar os[ logs de reinicialização da VM][reboot-logs] para determinar se uma reinicialização da VM foi causada por manutenção planejada.

Para se proteger contra perda acidental de dados durante operações normais (por exemplo, devido ao erro do usuário), você também deve implementar backups pontuais usando [instantâneos de blob][blob-snapshot] ou outra ferramenta.

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

**Grupos de recursos.** Coloque recursos estreitamente associados que compartilhem o mesmo ciclo de vida no mesmo [grupo de recursos][resource-manager-overview]. Os grupos de recursos permitem implantar e monitorar recursos como um grupo e rastrear custos de cobrança por grupo de recursos. Também é possível excluir recursos como um conjunto, o que é muito útil para implantações de teste. Atribua nomes de recursos significativos para simplificar a localização de um recurso específico e o reconhecimento de sua função. Para obter mais informações, consulte as [Convenções de nomenclatura recomendadas para os Recursos do Azure][naming-conventions].

**Interrompendo uma VM.** O Azure faz uma distinção entre os estados "parado" e "desalocado". Você será cobrado quando o status da VM for interrompido, mas não quando a VM for desalocada. No Portal do Azure, o botão **Parar** desaloca a VM. No entanto, se você desligar por meio do sistema operacional enquanto estiver conectado, a VM será interrompida, mas **não** desalocada e, portanto, você ainda será cobrado.

**Excluindo uma VM.** Se você excluir uma VM, os VHDs não serão excluídos. Isso significa que você poderá excluir com segurança a VM sem perda de dados. No entanto, você ainda será cobrado pelo armazenamento. Para excluir o VHD, exclua o arquivo do [Armazenamento de blobs][blob-storage]. Para evitar a exclusão acidental, use um [bloqueio de recurso][resource-lock] para bloquear o grupo de recursos inteiro ou bloquear recursos individuais, como uma VM.

## <a name="security-considerations"></a>Considerações de segurança

Use a [Central de Segurança do Azure][security-center] para obter uma exibição central do estado da segurança de seus recursos do Azure. A Central de Segurança monitora problemas de segurança potenciais e fornece uma visão abrangente da integridade de segurança de sua implantação. A Central de Segurança é configurada por assinatura do Azure. Habilite a coleta de dados de segurança, conforme descrito no [Guia de início rápido da Central de Segurança do Azure][security-center-get-started]. Depois que a coleta de dados for habilitada, a Central de Segurança examinará automaticamente todas as VMs criadas nessa assinatura.

**Gerenciamento de patch.** Se for habilitada, a Central de Segurança verificará se quaisquer atualizações críticas e de segurança estão ausentes. Use as [Configurações da Política de Grupo][group-policy] na VM para habilitar as atualizações automáticas do sistema.

**Antimalware.** Se for habilitada, a Central de Segurança verificará se o software antimalware está instalado. Você também pode usar a Central de Segurança para instalar o software antimalware por meio do Portal do Azure.

**Operações.** Use o [RBAC (controle de acesso baseado em função)][rbac] para controlar o acesso aos recursos do Azure implantados. O RBAC permite atribuir funções de autorização aos membros de sua equipe de DevOps. Por exemplo, a função Leitor pode exibir os recursos do Azure, mas não criar, gerenciar nem excluí-los. Algumas funções são específicas a determinados tipos de recursos do Azure. Por exemplo, a função Colaborador da Máquina Virtual pode reiniciar ou desalocar uma VM, redefinir a senha de administrador, criar uma nova VM e, assim por diante. Outras [funções RBAC internas][rbac-roles] que podem ser úteis para esta arquitetura incluem [Usuário de DevTest Labs][rbac-devtest] e [Colaborador de Rede][rbac-network]. Um usuário pode ser atribuído a várias funções, e você pode criar funções personalizadas para permissões ainda mais refinadas.

> [!NOTE]
> O RBAC não limita as ações que podem ser executadas por um usuário conectado a uma VM. Essas permissões são determinadas pelo tipo de conta no SO convidado.   

Use os [logs de auditoria][audit-logs] para ver as ações de provisionamento e outros eventos da VM.

**Criptografia de dados.** Considere o uso do [Azure Disk Encryption][disk-encryption] se você precisar criptografar os discos do sistema operacional e de dados. 

**Proteção contra DDoS**. É recomendável habilitar a [Proteção contra DDoS Standard](/azure/virtual-network/ddos-protection-overview), que fornece a mitigação de DDoS adicional para os recursos em uma VNet. Embora a proteção contra DDoS básica seja habilitada automaticamente como parte da plataforma Azure, a Proteção contra DDoS Standard fornece funcionalidades de mitigação ajustadas especificamente para os recursos da Rede Virtual do Azure.  

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para essa arquitetura está disponível no [GitHub][github-folder]. Ela implanta o seguinte:

  * Uma rede virtual com uma única sub-rede denominada **Web** usada para hospedar a VM.
  * Um NSG com duas regras de entrada para permitir tráfego RDP e HTTP para a VM.
  * Uma VM que executa a versão mais recente do Windows Server 2016 Datacenter Edition.
  * Uma extensão de script personalizado de exemplo que formata os dois discos de dados e um script do PowerShell DSC que implanta o IIS (Serviços de Informações da Internet).

### <a name="prerequisites"></a>Pré-requisitos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution-using-azbb"></a>Implantar a solução usando azbb

Para implantar essa arquitetura de referência, siga estas etapas:

1. Navegue até a pasta `virtual-machines\single-vm\parameters\windows` do repositório que você fez o download na etapa de pré-requisitos acima.

2. Abra o arquivo `single-vm-v2.json` e insira um nome de usuário e senha entre aspas, depois salve o arquivo.

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. Execute `azbb` para implantar a VM de exemplo conforme mostrado abaixo.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

Para verificar a implantação, execute o seguinte comando da CLI do Azure para localizar o endereço IP público da VM:

```bash
az vm show -n ra-single-windows-vm1 -g <resource-group-name> -d -o table
```

Se você navegar até esse endereço em um navegador da Web, deve ver a página inicial padrão do IIS.

Para obter informações sobre como personalizar essa implantação, visite nosso [repositório GitHub][git].

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-windows-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-windows-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[group-policy]: https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn595129(v=ws.11)
[log-collector]: https://azure.microsoft.com/blog/simplifying-virtual-machine-troubleshooting-using-azure-log-collector/
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
[naming-conventions]: ../../best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-windows-planned-maintenance
[premium-storage]: /azure/virtual-machines/windows/premium-storage
[premium-storage-supported]: /azure/virtual-machines/windows/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resize-os-disk]: /azure/virtual-machines/virtual-machines-windows-expand-os-disk
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-windows-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-windows-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Única arquitetura de VM do Windows no Azure"
