---
title: Executar uma VM do Linux no Azure
titleSuffix: Azure Reference Architectures
description: Melhores práticas para execução de uma máquina virtual do Linux no Azure.
author: telmosampaio
ms.date: 09/13/2018
ms.custom: seodec18
ms.openlocfilehash: b8e0f758765f45eabf5fe3480c99cc54bf5d0ee8
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120043"
---
# <a name="run-a-linux-virtual-machine-on-azure"></a>Executar uma máquina virtual com Linux no Azure

Esse artigo mostra práticas comprovadas para execução de uma VM (máquina virtual) com Linux no Azure. Ela inclui recomendações para provisionar a VM juntamente com os componentes de rede e armazenamento. [**Implantar esta solução**](#deploy-the-solution).

[0]: ./images/single-vm-diagram.png "VM única do Linux no Azure"

## <a name="components"></a>Componentes

O provisionamento de uma VM do Azure requer alguns componentes adicionais além da própria VM, incluindo recursos de rede, armazenamento e computação.

- **Grupo de recursos**. Um [grupo de recursos][resource-manager-overview] é um contêiner lógico que armazena os recursos relacionados ao Azure. Em geral, recursos de grupo baseados em seu tempo de vida e que vão gerenciá-los.

- **VM**. Você pode provisionar uma VM a partir de uma lista de imagens publicadas, de um arquivo gerenciado personalizado ou um arquivo de VHD (disco rígido virtual) carregado no armazenamento Blobs do Azure. O Azure dá suporte à execução de várias distribuições Linux populares, incluindo CentOS, Debian, Red Hat Enterprise, Ubuntu e FreeBSD. Para obter mais informações, veja [Azure e Linux][azure-linux].

- **Managed Disks**. Os [Azure Managed Disks][managed-disks] simplificam o gerenciamento de disco ao manipular o armazenamento para você. O disco de OS é um VHD armazenado no [Armazenamento do Microsoft Azure ][azure-storage], de modo que ele persiste mesmo quando o computador host está desligado. Para VMs do Linux, o disco de SO é `/dev/sda1`. Também é recomendável criar um ou mais [discos de dados][data-disk], que são VHDs persistentes usados para dados de aplicativo.

- **Disco temporário**.  A VM é criada com um disco temporário. Esse disco é armazenado em uma unidade física no computador host. Ele *não* é salvo no Armazenamento do Microsoft Azure e pode ser excluído durante a reinicialização e outros eventos de ciclo de vida da VM. Use esse disco somente para dados temporários, como arquivos de paginação ou de permuta. Para VMs do Linux, o disco temporário é `/dev/sdb1` e é montado em `/mnt/resource` ou `/mnt`.

- **Rede virtual (VNet)**. Cada VM do Azure é implantada em uma VNet que pode ser segmentada em várias sub-redes.

- **NIC (adaptador de rede)**. A NIC permite que a VM se comunique com a rede virtual.

- **Endereço IP público**. Um endereço IP público é necessário para se comunicar com a VM &mdash;, por exemplo, via SSH.

- **DNS do Azure**. [DNS do Azure][azure-dns] é um serviço de hospedagem para domínios DNS, que fornece resolução de nomes usando a infraestrutura do Microsoft Azure. Ao hospedar seus domínios no Azure, você pode gerenciar seus registros DNS usando as mesmas credenciais, APIs, ferramentas e cobrança que seus outros serviços do Azure.

- **NSG (grupo de segurança de rede)**. Os [Grupos de segurança de rede][nsg] são utilizados para permitir ou recusar o tráfego de rede a VMs. Os NSGs podem ser associados com sub-redes ou com instâncias VM individuais.

- **Diagnóstico**.  O registro de diagnóstico é crucial para o gerenciamento e solução de problemas da VM.

## <a name="vm-recommendations"></a>Recomendações de VM

O Azure oferece vários tamanhos de máquinas virtuais diferentes. Para obter mais informações, confira [Tamanhos das máquinas virtuais no Azure][virtual-machine-sizes]. Se você estiver movendo uma carga de trabalho existente para o Azure, deverá começar com o tamanho da VM que mais se aproxima de seus servidores locais. Em seguida, meça o desempenho da carga de trabalho real com relação à CPU, memória e operações de entrada/saída de disco e ajuste o tamanho conforme necessário. Se você precisar de várias NICs para sua VM, esteja ciente de que um número máximo de NICs está definido para cada [tamanho de VM][vm-size-tables].

Em geral, escolha a região do Azure que esteja mais próxima de seus usuários internos ou clientes. No entanto, nem todos os tamanhos de VM estão disponíveis em todas as regiões. Para obter mais informações, consulte [Serviços por região][services-by-region]. Para obter uma lista dos tamanhos de VM disponíveis em uma região específica, execute o seguinte comando a partir da interface de linha de comando do Azure (CLI):

```azurecli
az vm list-sizes --location <location>
```

Para obter informações sobre como escolher uma imagem de VM publicada, consulte [Localizar imagens de VM do Linux][select-vm-image].

Habilite o monitoramento e diagnóstico, incluindo métricas de integridade básicas, logs de infraestrutura de diagnóstico e [diagnóstico de inicialização][boot-diagnostics]. O diagnóstico de inicialização poderá ajudar a diagnosticar uma falha de inicialização se sua VM entrar em um estado não inicializável. Para saber mais, confira [Habilitar monitoramento e diagnóstico][enable-monitoring].

## <a name="disk-and-storage-recommendations"></a>Recomendações de disco e de armazenamento

Para um melhor desempenho de E/S de disco, recomendamos o [Armazenamento Premium][premium-storage], que armazena dados em SSDs (unidades de estado sólido). O custo é baseado na capacidade do disco provisionado. O IOPS e a taxa de transferência (ou seja, a taxa de transferência de dados) também dependem do tamanho do disco. Portanto, ao provisionar um disco, considere todos os três fatores (capacidade, IOPS e taxa de transferência).

Também é recomendável usar [Managed Disks][managed-disks]. Os discos gerenciados não exigem uma conta de armazenamento. Você simplesmente especifica o tamanho e o tipo de disco e é implantado como um recurso altamente disponível.

Adicione um ou mais discos de dados. Quando você cria um VHD, ele não está formatado. Faça logon na VM para formatar o disco. No shell do Linux, os discos de dados são exibidos como `/dev/sdc`, `/dev/sdd` e assim por diante. Você pode executar `lsblk` para listar os dispositivos de bloco, incluindo os discos. Para usar um disco de dados, crie uma partição e o sistema de arquivos e monte o disco. Por exemplo: 

```bash
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

Quando você adiciona um disco de dados, uma ID de LUN (número de unidade lógica) é atribuída ao disco. Opcionalmente, você poderá especificar a ID do LUN &mdash; por exemplo, se estiver substituindo um disco e quiser manter a mesma ID de LUN ou tiver um aplicativo que está procurando uma ID de LUN específica. No entanto, lembre-se de que as IDs de LUN devem ser exclusivas para cada disco.

Convém alterar o agendador de E/S para otimizar o desempenho em SSDs, pois os discos para VMs com contas de armazenamento premium são SSDs. Uma recomendação comum é usar o agendador NOOP para SSDs, mas é necessário usar uma ferramenta como [iostat] para monitorar o desempenho de E/S de disco para sua carga de trabalho.

Crie uma conta de armazenamento para manter os logs de diagnóstico. Uma conta LRS (armazenamento com redundância local) padrão é suficiente para os logs de diagnóstico.

> [!NOTE]
> Se você não estiver usando Managed Disks, crie contas de armazenamento do Azure separadas para que cada VM contenha os VHDs (discos rígidos virtuais), evitando, assim, atingir os [limites (de IOPS)][vm-disk-limits] para contas de armazenamento. Esteja ciente dos limites de E/S totais da conta de armazenamento. Para saber mais, confira [limites de disco da máquina virtual][vm-disk-limits].

## <a name="network-recommendations"></a>Recomendações de rede

Esse endereço IP público pode ser dinâmico ou estático. O padrão é dinâmico.

- Reserve um [endereço IP estático][static-ip] se precisar de um endereço IP fixo que não mudará &mdash; por exemplo, se precisar criar um registro A no DNS ou se precisar adicionar o endereço IP a uma lista segura.
- Você também pode criar um FQDN (nome de domínio totalmente qualificado) para o endereço IP. Em seguida, é possível registrar um [registro CNAME][cname-record] no DNS que aponta para o FQDN. Para saber mais, consulte [Criar um nome de domínio totalmente qualificado no Portal do Azure][fqdn]. Use o [DNS do Azure][azure-dns] ou outro serviço DNS.

Todos os NSGs contêm um conjunto de [regras padrão][nsg-default-rules], incluindo uma regra que bloqueia todo o tráfego de Internet de entrada. As regras padrão não podem ser excluídas, mas outras regras podem substituí-las. Para habilitar o tráfego de Internet, crie regras que permitam o tráfego de entrada em portas específicas &mdash; por exemplo, a porta 80 para HTTP.

Para habilitar o SSH, adicione uma regra NSG que permita o tráfego de entrada para a porta TCP 22.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Você pode escalar uma VM vertical ou horizontalmente [alterando o tamanho da VM][vm-resize]. Para aumentar horizontalmente, coloque duas ou mais VMs atrás de um balanceador de carga. Para obter mais informações, consulte a [Arquitetura de referência de N camadas](./n-tier-cassandra.md).

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Para obter maior disponibilidade, implante várias VMs em um conjunto de disponibilidade. Isso também fornece [SLA (Contrato de Nível de Serviço)][vm-sla] mais elevado.

Sua VM pode ser afetada por uma [manutenção planejada][planned-maintenance] ou [manutenção não planejada][manage-vm-availability]. Você pode usar os[ logs de reinicialização da VM][reboot-logs] para determinar se uma reinicialização da VM foi causada por manutenção planejada.

Para se proteger contra perda acidental de dados durante operações normais (por exemplo, devido ao erro do usuário), você também deve implementar backups pontuais usando [instantâneos de blob][blob-snapshot] ou outra ferramenta.

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

**Grupos de recursos**. Coloque recursos estreitamente associados que compartilhem o mesmo ciclo de vida no mesmo [grupo de recursos][resource-manager-overview]. Os grupos de recursos permitem implantar e monitorar recursos como um grupo e rastrear custos de cobrança por grupo de recursos. Também é possível excluir recursos como um conjunto, o que é muito útil para implantações de teste. Atribua nomes de recursos significativos para simplificar a localização de um recurso específico e o reconhecimento de sua função. Para obter mais informações, consulte as [Convenções de nomenclatura recomendadas para os Recursos do Azure][naming-conventions].

**SSH**. Antes de criar uma VM do Linux, gere um par de chaves pública-privada do RSA de 2048 bits. Use o arquivo de chave pública ao criar a VM. Para saber mais, veja [Como usar SSH com Linux e Mac no Azure][ssh-linux].

**Interromper uma VM**. O Azure faz uma distinção entre os estados "parado" e "desalocado". Você será cobrado quando o status da VM for interrompido, mas não quando a VM for desalocada. No Portal do Azure, o botão **Parar** desaloca a VM. No entanto, se você desligar por meio do sistema operacional enquanto estiver conectado, a VM será interrompida, mas **não** desalocada e, portanto, você ainda será cobrado.

**Excluir uma VM**.  Se você excluir uma VM, os VHDs não serão excluídos. Isso significa que você poderá excluir com segurança a VM sem perda de dados. No entanto, você ainda será cobrado pelo armazenamento. Para excluir o VHD, exclua o arquivo do [Armazenamento de blobs][blob-storage]. Para evitar a exclusão acidental, use um [bloqueio de recurso][resource-lock] para bloquear o grupo de recursos inteiro ou bloquear recursos individuais, como uma VM.

## <a name="security-considerations"></a>Considerações de segurança

Use a [Central de Segurança do Azure][security-center] para obter uma exibição central do estado da segurança de seus recursos do Azure. A Central de Segurança monitora problemas de segurança potenciais e fornece uma visão abrangente da integridade de segurança de sua implantação. A Central de Segurança é configurada por assinatura do Azure. Habilite a coleta de dados de segurança, conforme descrito no [Guia de início rápido da Central de Segurança do Azure][security-center-get-started]. Depois que a coleta de dados for habilitada, a Central de Segurança examinará automaticamente todas as VMs criadas nessa assinatura.

**Gerenciamento de patch**. Se for habilitada, a Central de Segurança verificará se quaisquer atualizações críticas e de segurança estão ausentes.

**Antimalware**.  Se for habilitada, a Central de Segurança verificará se o software antimalware está instalado. Você também pode usar a Central de Segurança para instalar o software antimalware por meio do Portal do Azure.

**Operações**. Use o [RBAC (controle de acesso baseado em função)][rbac] para controlar o acesso aos recursos do Azure implantados. O RBAC permite atribuir funções de autorização aos membros de sua equipe de DevOps. Por exemplo, a função Leitor pode exibir os recursos do Azure, mas não criar, gerenciar nem excluí-los. Algumas funções são específicas a determinados tipos de recursos do Azure. Por exemplo, a função Colaborador da Máquina Virtual pode reiniciar ou desalocar uma VM, redefinir a senha de administrador, criar uma nova VM e, assim por diante. Outras [funções RBAC internas][rbac-roles] que podem ser úteis para esta arquitetura incluem [Usuário de DevTest Labs][rbac-devtest] e [Colaborador de Rede][rbac-network]. Um usuário pode ser atribuído a várias funções, e você pode criar funções personalizadas para permissões ainda mais refinadas.

> [!NOTE]
> O RBAC não limita as ações que podem ser executadas por um usuário conectado a uma VM. Essas permissões são determinadas pelo tipo de conta no SO convidado.

Use os [logs de auditoria][audit-logs] para ver as ações de provisionamento e outros eventos da VM.

**Criptografia de dados**. Considere o uso do [Azure Disk Encryption][disk-encryption] se você precisar criptografar os discos do sistema operacional e de dados.

**Proteção contra DDoS**. É recomendável habilitar a [Proteção contra DDoS Standard](/azure/virtual-network/ddos-protection-overview), que fornece a mitigação de DDoS adicional para os recursos em uma VNet. Embora a proteção contra DDoS básica seja habilitada automaticamente como parte da plataforma Azure, a Proteção contra DDoS Standard fornece funcionalidades de mitigação ajustadas especificamente para os recursos da Rede Virtual do Azure.

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação está disponível no [GitHub][github-folder]. Ela implanta o seguinte:

- Uma rede virtual com uma única sub-rede denominada **Web** usada para hospedar a VM.
- Um NSG com duas regras de entrada para permitir tráfego HTTP e SSH para a VM.
- Uma VM que executa a versão mais recente do Ubuntu 16.04.3 LTS.
- Uma extensão de script personalizado de exemplo que formata os dois discos de dados e implanta o Servidor HTTP do Apache para a VM do Ubuntu.

### <a name="prerequisites"></a>Pré-requisitos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution-using-azbb"></a>Implantar a solução usando azbb

1. Crie um par de chaves SSH. Para obter mais informações, consulte [Como criar e usar um par de chaves SSH pública e privada para VMs Linux no Azure](/azure/virtual-machines/linux/mac-create-ssh-keys).

2. Navegue até a pasta `virtual-machines/single-vm/parameters/linux` do repositório que você fez o download na etapa de pré-requisitos acima.

3. Abra o arquivo `single-vm-v2.json` e insira um nome de usuário e a chave SSH pública entre aspas, depois salve o arquivo.

    ```json
    "adminUsername": "<your username>",
    "sshPublicKey": "ssh-rsa AAAAB3NzaC1...",
    ```

4. Execute `azbb` para implantar a VM de exemplo conforme mostrado abaixo.

    ```azurecli
    azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
    ```

Para verificar a implantação, execute o seguinte comando da CLI do Azure para localizar o endereço IP público da VM:

```azurecli
az vm show -n ra-single-linux-vm1 -g <resource-group-name> -d -o table
```

Se você navegar até esse endereço em um navegador da Web, deve ver a página inicial padrão do Apache2.

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-linux]: /azure/virtual-machines/virtual-machines-linux-azure-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-linux-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[azure-dns]: /azure/dns/dns-overview
[fqdn]: /azure/virtual-machines/virtual-machines-linux-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
[naming-conventions]: ../../best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-linux-planned-maintenance
[premium-storage]: /azure/virtual-machines/linux/premium-storage
[premium-storage-supported]: /azure/virtual-machines/linux/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-linux-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssh-linux]: /azure/virtual-machines/virtual-machines-linux-mac-create-ssh-keys
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-linux-sizes
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
