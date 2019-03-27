---
title: Executar uma VM do Windows no Azure
titleSuffix: Azure Reference Architectures
description: Melhores práticas para execução de uma máquina virtual com Windows no Azure.
author: telmosampaio
ms.date: 12/13/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: a25488357eb11b80e8f79ddae7f7d69735a6bec3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242927"
---
# <a name="run-a-windows-virtual-machine-on-azure"></a>Executar uma máquina virtual com Windows no Azure

O provisionamento de uma VM (máquina virtual) no Azure requer alguns componentes adicionais além da própria VM, incluindo recursos de rede e armazenamento. Este artigo mostra as melhores práticas para executar uma VM do Windows no Azure.

![VM do Windows no Azure](./images/single-vm-diagram.png)

## <a name="resource-group"></a>Grupo de recursos

Um [grupo de recursos][resource-manager-overview] é um contêiner lógico que armazena os recursos relacionados ao Azure. Em geral, recursos de grupo baseados em seu tempo de vida e que vão gerenciá-los.

Coloque recursos estreitamente associados que compartilhem o mesmo ciclo de vida no mesmo [grupo de recursos][resource-manager-overview]. Os grupos de recursos permitem implantar e monitorar recursos como um grupo e rastrear custos de cobrança por grupo de recursos. Também é possível excluir recursos como um conjunto, o que é muito útil para implantações de teste. Atribua nomes de recursos significativos para simplificar a localização de um recurso específico e o reconhecimento de sua função. Para obter mais informações, consulte as [Convenções de nomenclatura recomendadas para os Recursos do Azure][naming-conventions].

## <a name="virtual-machine"></a>Máquina virtual

Você pode provisionar uma VM a partir de uma lista de imagens publicadas, de um arquivo gerenciado personalizado ou um arquivo de VHD (disco rígido virtual) carregado no armazenamento Blobs do Azure.

O Azure oferece vários tamanhos de máquinas virtuais diferentes. Para obter mais informações, confira [Tamanhos das máquinas virtuais no Azure][virtual-machine-sizes]. Se você estiver movendo uma carga de trabalho existente para o Azure, deverá começar com o tamanho da VM que mais se aproxima de seus servidores locais. Em seguida, meça o desempenho da carga de trabalho real em termos de CPU, memória e IOPS (operações de entrada/saída de disco por segundo) e ajuste o tamanho conforme a necessidade.

Em geral, escolha a região do Azure que esteja mais próxima de seus usuários internos ou clientes. Nem todos os tamanhos de VM estão disponíveis em todas as regiões. Para obter mais informações, consulte [Serviços por região][services-by-region]. Para obter uma lista dos tamanhos de VM disponíveis em uma região específica, execute o seguinte comando a partir da interface de linha de comando do Azure (CLI):

```azurecli
az vm list-sizes --location <location>
```

Para obter informações sobre como escolher uma imagem de VM publicada, consulte [Localizar imagens de VM do Windows][select-vm-image].

## <a name="disks"></a>Discos

Para um melhor desempenho de E/S de disco, recomendamos o [Armazenamento Premium][premium-storage], que armazena dados em SSDs (unidades de estado sólido). O custo é baseado na capacidade do disco provisionado. O IOPS e a taxa de transferência também dependem do tamanho do disco. Portanto, ao provisionar um disco, considere todos os três fatores (capacidade, IOPS e taxa de transferência).

Também é recomendável usar [Managed Disks][managed-disks]. Os discos gerenciados simplificam o gerenciamento de disco manipulando o armazenamento por você. Os discos gerenciados não exigem uma conta de armazenamento. Você simplesmente especifica o tamanho e o tipo de disco e ele é implantado como um recurso altamente disponível

O disco de OS é um VHD armazenado no [Armazenamento do Microsoft Azure ][azure-storage], de modo que ele persiste mesmo quando o computador host está desligado. Também é recomendável criar um ou mais [discos de dados][data-disk], que são VHDs persistentes usados para dados de aplicativo. Quando possível, instale aplicativos em um disco de dados, não no disco do sistema operacional. Algumas aplicações legadas podem precisar instalar componentes na unidade C:. Nesse caso, você pode [redimensionar o disco de SO][resize-os-disk] utilizando o PowerShell.

A VM também é criada com um disco temporário (a unidade `D:` no Windows). Esse disco é armazenado em uma unidade física no computador host. Ele *não* é salvo no Armazenamento do Microsoft Azure e pode ser excluído durante a reinicialização e outros eventos de ciclo de vida da VM. Use esse disco somente para dados temporários, como arquivos de paginação ou de permuta.

## <a name="network"></a>Rede

Os componentes de rede incluem os seguintes recursos:

- **Rede virtual**. Cada VM é implantada em uma rede virtual que pode ser segmentada em várias sub-redes.

- **NIC (adaptador de rede)**. A NIC permite que a VM se comunique com a rede virtual. Se você precisar de vários NICs para sua VM, esteja ciente de que existe um número máximo de NICs definido para cada [tamanho de VM][vm-size-tables].

- **Endereço IP público**. Um endereço IP público é necessário para se comunicar com a VM &mdash; por exemplo, sobre RDP (área de trabalho remota). Esse endereço IP público pode ser dinâmico ou estático. O padrão é dinâmico.

- Reserve um [endereço IP estático][static-ip] se precisar de um endereço IP fixo que não mudará, &mdash;por exemplo, se precisar criar um registro 'A' de DNS ou adicionar o IP endereço para uma lista segura.
- Você também pode criar um FQDN (nome de domínio totalmente qualificado) para o endereço IP. Em seguida, é possível registrar um [registro CNAME][cname-record] no DNS que aponta para o FQDN. Para saber mais, consulte [Criar um nome de domínio totalmente qualificado no Portal do Azure][fqdn].

- **NSG (grupo de segurança de rede)**. Os [Grupos de segurança de rede][nsg] são utilizados para permitir ou recusar o tráfego de rede a VMs. Os NSGs podem ser associados com sub-redes ou com instâncias VM individuais.

Todos os NSGs contêm um conjunto de [regras padrão][nsg-default-rules], incluindo uma regra que bloqueia todo o tráfego de Internet de entrada. As regras padrão não podem ser excluídas, mas outras regras podem substituí-las. Para habilitar o tráfego de Internet, crie regras que permitam o tráfego de entrada em portas específicas &mdash; por exemplo, a porta 80 para HTTP. Para habilitar o RDP, adicione uma regra NSG que permita o tráfego de entrada na porta TCP 3389.

## <a name="operations"></a>Operações

**Diagnóstico**. Habilite o monitoramento e diagnóstico, incluindo métricas de integridade básicas, logs de infraestrutura de diagnóstico e [diagnóstico de inicialização][boot-diagnostics]. O diagnóstico de inicialização poderá ajudar a diagnosticar uma falha de inicialização se sua VM entrar em um estado não inicializável. Crie uma conta do Armazenamento do Azure para armazenar os logs. Uma conta LRS (armazenamento com redundância local) padrão é suficiente para os logs de diagnóstico. Para saber mais, confira [Habilitar monitoramento e diagnóstico][enable-monitoring].

**Disponibilidade**. Sua VM pode ser afetada por [manutenção planejada][planned-maintenance] ou por [tempo de inatividade não planejado][manage-vm-availability]. Você pode usar os[ logs de reinicialização da VM][reboot-logs] para determinar se uma reinicialização da VM foi causada por manutenção planejada. Para obter maior disponibilidade, implante várias VMs em um [conjunto de disponibilidade](/azure/virtual-machines/windows/manage-availability#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy). Essa configuração fornece um maior [SLA (contrato de nível de serviço)][vm-sla].

**Backups** Para se proteger contra perda acidental de dados, use o serviço [Backup do Azure](/azure/backup/) a fim de fazer o backup das VMs em um armazenamento com redundância geográfica. O Backup do Azure fornece backups consistentes com o aplicativo.

**Interromper uma VM**. O Azure faz uma distinção entre os estados "parado" e "desalocado". Você será cobrado quando o status da VM for interrompido, mas não quando a VM for desalocada. No Portal do Azure, o botão **Parar** desaloca a VM. No entanto, se você desligar por meio do sistema operacional enquanto estiver conectado, a VM será interrompida, mas **não** desalocada e, portanto, você ainda será cobrado.

**Excluir uma VM**.  Se você excluir uma VM, os VHDs não serão excluídos. Isso significa que você poderá excluir com segurança a VM sem perda de dados. No entanto, você ainda será cobrado pelo armazenamento. Para excluir o VHD, exclua o arquivo do [Armazenamento de blobs][blob-storage]. Para evitar a exclusão acidental, use um [bloqueio de recurso][resource-lock] para bloquear o grupo de recursos inteiro ou bloquear recursos individuais, como uma VM.

## <a name="security-considerations"></a>Considerações de segurança

Use a [Central de Segurança do Azure][security-center] para obter uma exibição central do estado da segurança de seus recursos do Azure. A Central de Segurança monitora problemas de segurança potenciais e fornece uma visão abrangente da integridade de segurança de sua implantação. A Central de Segurança é configurada por assinatura do Azure. Habilite a coleta de dados de segurança conforme descrito em [Integrar a assinatura do Azure à Central de Segurança Standard][security-center-get-started]. Depois que a coleta de dados for habilitada, a Central de Segurança examinará automaticamente todas as VMs criadas nessa assinatura.

**Gerenciamento de patch**. Se for habilitada, a Central de Segurança verificará se quaisquer atualizações críticas e de segurança estão ausentes. Use as [Configurações da Política de Grupo][group-policy] na VM para habilitar as atualizações automáticas do sistema.

**Antimalware**.  Se for habilitada, a Central de Segurança verificará se o software antimalware está instalado. Você também pode usar a Central de Segurança para instalar o software antimalware por meio do Portal do Azure.

**Controle de acesso**. Use o [RBAC (controle de acesso baseado em função)][rbac] para controlar o acesso aos recursos do Azure. O RBAC permite atribuir funções de autorização aos membros de sua equipe de DevOps. Por exemplo, a função Leitor pode exibir os recursos do Azure, mas não criar, gerenciar nem excluí-los. Algumas permissões são específicas para determinado tipo de recurso do Azure. Por exemplo, a função Colaborador da Máquina Virtual pode reiniciar ou desalocar uma VM, redefinir a senha de administrador, criar uma nova VM e, assim por diante. Outras [funções RBAC internas][rbac-roles] que podem ser úteis para esta arquitetura incluem [Usuário de DevTest Labs][rbac-devtest] e [Colaborador de Rede][rbac-network]. 

> [!NOTE]
> O RBAC não limita as ações que podem ser executadas por um usuário conectado a uma VM. Essas permissões são determinadas pelo tipo de conta no SO convidado.

**Logs de auditoria**. Use os [logs de auditoria][audit-logs] para ver as ações de provisionamento e outros eventos da VM.

**Criptografia de dados**. Use o [Azure Disk Encryption][disk-encryption] se você precisa criptografar os discos do sistema operacional e de dados.

## <a name="next-steps"></a>Próximas etapas

- Para provisionar uma VM do Windows, confira [Criar e gerenciar VMs do Windows com o Azure PowerShell](/azure/virtual-machines/windows/tutorial-manage-vm)
- Para uma arquitetura de N camadas completa em VMs do Windows, confira [Aplicativo de N camadas do Windows no Azure com o SQL Server](./n-tier-sql-server.md).

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[azure-storage]: /azure/storage/storage-introduction
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-windows-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-windows-portal-create-fqdn
[group-policy]: /windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
[naming-conventions]: ../../best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-windows-planned-maintenance
[premium-storage]: /azure/virtual-machines/windows/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
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
[vm-size-tables]: /azure/virtual-machines/virtual-machines-windows-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
