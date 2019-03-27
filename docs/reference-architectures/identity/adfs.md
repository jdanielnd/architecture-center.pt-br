---
title: Estender o AD FS local para o Azure
titleSuffix: Azure Reference Architectures
description: Implemente uma arquitetura de rede híbrida segura com autorização do Serviço de Federação do Active Directory no Azure.
author: telmosampaio
ms.date: 12/18.2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, identity
ms.openlocfilehash: 22a2a2042c85e70d0d5a523c9ecf72395a9e774c
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58244387"
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a>Estender o Serviços de Federação do Active Directory (AD FS) para o Azure

Essa arquitetura de referência implementa uma rede híbrida segura que estende a rede local para o Azure e usa o [Serviços de Federação do Active Directory (AD FS)][active-directory-federation-services] para executar autenticação e autorização federadas para componentes em execução no Azure. [**Implantar esta solução**](#deploy-the-solution).

![Arquitetura de rede híbrida segura com o Active Directory](./images/adfs.png)

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

O AD FS poderá ser hospedado localmente, mas se o aplicativo for um híbrido com algumas partes implementadas no Azure, talvez seja mais eficiente replicar o AD FS na nuvem.

O diagrama mostra os cenários a seguir:

- O código do aplicativo de uma organização parceira acessa um aplicativo Web hospedado dentro de sua VNet do Azure.
- Um usuário externo, registrado com credenciais armazenadas no DS (Active Directory Domain Services) acessa um aplicativo Web hospedado dentro de sua VNet do Azure.
- Um usuário conectado à sua VNet usando um dispositivo autorizado executa um aplicativo Web hospedado dentro de sua VNet do Azure.

Alguns usos típicos dessa arquitetura:

- Aplicativos híbridos nos quais as cargas de trabalho são executadas parcialmente localmente e parcialmente no Azure.
- Soluções que usam a autorização federada para expor aplicativos Web a organizações parceiras.
- Sistemas que dão suporte a acesso de navegadores da Web em execução fora do firewall organizacional.
- Sistemas que permitem que os usuários acessem aplicativos Web se conectando de dispositivos externos autorizados, como computadores remotos, notebooks e outros dispositivos móveis.

Essa arquitetura de referência concentra-se na *federação passiva*, na qual os servidores de federação decidem como e quando autenticar um usuário. O usuário fornece informações de entrada quando o aplicativo é iniciado. Esse mecanismo geralmente é usado por navegadores da Web e envolve um protocolo que redireciona o navegador para um site em que o usuário é autenticado. O AD FS também dá suporte para *federação ativa*, na qual um aplicativo assume a responsabilidade de fornecer credenciais sem interação adicional do usuário, mas esse cenário está fora do escopo dessa arquitetura.

Para obter considerações adicionais, consulte [Escolher uma solução para a integração do Active Directory local ao Azure][considerations].

## <a name="architecture"></a>Arquitetura

Essa arquitetura estende a implementação descrita em [Estendendo o AD DS (Active Directory Domain Services) para o Azure][extending-ad-to-azure]. Ela contém os componentes a seguir.

- **Sub-rede do AD DS**. Os servidores do AD DS estão contidos em suas próprias sub-redes com as regras do NSG (Grupo de Segurança de Rede) atuando como um firewall.

- **Servidores do AD DS**. Controladores de domínio em execução como VMs no Azure. Esses servidores fornecem autenticação de identidades locais dentro do domínio.

- **Sub-rede do AD FS**. Os servidores do AD FS estão localizados em suas próprias sub-redes com regras do NSG atuando como um firewall.

- **Servidores do AD FS**. Os servidores do AD FS fornecem autenticação e autorização federadas. Nessa arquitetura, eles executam as seguintes tarefas:

  - Recebimento de tokens de segurança contendo declarações feitas por um servidor de federação do parceiro em nome de um usuário do parceiro. O AD FS verifica se os tokens são válidos antes de passar as declarações para o aplicativo Web em execução no Azure para autorizar solicitações.

    O aplicativo em execução no Azure é a *terceira parte confiável*. O servidor de federação do parceiro deve emitir declarações que sejam entendidas pelo aplicativo Web. Os servidores de federação do parceiro são chamados de *parceiros de conta*, porque eles enviam solicitações de acesso em nome de contas autenticadas na organização do parceiro. Os servidores do AD FS são chamados de *parceiros de recurso* porque eles fornecem acesso aos recursos (o aplicativo Web).

  - Autenticação e autorização de solicitações recebidas de usuários externos executando um navegador da Web ou um dispositivo que precise de acesso a aplicativos Web, usando o AD DS e o [Serviço de registro de dispositivo do Active Directory][ADDRS].

  Os servidores do AD FS são configurados como um farm acessado por meio de um Azure Load Balancer. Essa implementação melhora a disponibilidade e escalabilidade. Os servidores do AD FS não são expostos diretamente à Internet. Todo o tráfego de Internet é filtrado por meio de servidores proxy de aplicativo Web do AD FS e de uma rede de perímetro (também conhecida como DMZ).

  Para obter mais informações de como funciona o AD FS, consulte [Visão geral dos Serviços de Federação do Active Directory (AD FS)][active-directory-federation-services-overview]. Além disso, o artigo [Implantação do AD FS no Azure][adfs-intro] contém uma introdução passo a passo detalhada da implementação.

- **Sub-rede de proxy do AD FS**. Os servidores proxy do AD FS podem estar contidos em suas próprias sub-redes com regras do NSG fornecendo proteção. Os servidores nesta sub-rede são expostos à Internet por meio de um conjunto de soluções de virtualização de rede que fornecem um firewall entre a sua rede virtual do Azure e a Internet.

- **Servidores WAP (Proxy de aplicativo Web) do AD FS**. Essas VMs atuam como servidores do AD FS para solicitações recebidas de organizações parceiras e de dispositivos externos. Os servidores WAP atuam como um filtro, blindando os servidores do AD FS contra acesso direto da Internet. Assim como acontece com os servidores do AD FS, a implantação de servidores WAP em um farm com balanceamento de carga fornece maior disponibilidade e escalabilidade do que a implantação de uma coleção de servidores autônomos.

  > [!NOTE]
  > Para obter informações detalhadas de como instalar servidores WAP, consulte [Instalar e configurar o servidor proxy de aplicativo Web][install_and_configure_the_web_application_proxy_server]
  >

- **Organização parceira**. Uma organização parceira que executa um aplicativo Web que solicita acesso a um aplicativo Web em execução no Azure. O servidor de federação na organização parceira autentica as solicitações localmente e envia tokens de segurança contendo declarações para o AD FS em execução no Azure. O AD FS no Azure valida os tokens de segurança e, quando eles são válidos, passa as declarações para o aplicativo Web em execução no Azure para autorizá-los.

  > [!NOTE]
  > Você também pode configurar um túnel de VPN usando o gateway do Azure para fornecer acesso direto ao AD FS para parceiros confiáveis. As solicitações recebidas desses parceiros não passam por servidores WAP.
  >

## <a name="recommendations"></a>Recomendações

As seguintes recomendações aplicam-se à maioria dos cenários. Siga estas recomendações, a menos que você tenha um requisito específico que as substitua.

### <a name="networking-recommendations"></a>Recomendações de rede

Configure o adaptador de rede para cada uma das VMs que hospedam o AD FS e os servidores WAP com endereços IP privados estáticos.

Não forneça endereços IP públicos às VMs do AD FS. Para obter mais informações, confira a seção [Considerações de segurança](#security-considerations).

Defina o endereço IP dos servidores DNS (Serviço de Nomes de Domínio) preferencial e secundários para os adaptadores de rede de cada VM do AD FS e do WAP para referenciar as VMs do Active Directory DS. As VMs do Active Directory DS devem estar executando o DNS. Essa etapa é necessária para habilitar cada VM para ingressar no domínio.

### <a name="ad-fs-installation"></a>Instalação do AD FS

O artigo [Implantando um Farm de Servidores de Federação][Deploying_a_federation_server_farm] fornece instruções detalhadas para instalar e configurar o AD FS. Execute as seguintes tarefas antes de configurar o primeiro servidor do AD FS no farm:

1. Obtenha um certificado confiável publicamente para executar a autenticação do servidor. O *nome da entidade* deve conter o nome que os clientes usam para acessar o serviço de federação. Pode ser o nome DNS registrado para o balanceador de carga, por exemplo, *adfs.contoso.com* (evite usar nomes com caracteres curinga, como **.contoso.com*, por motivos de segurança). Use o mesmo certificado em todas as VMs de servidor do AD FS. É possível comprar um certificado de uma autoridade de certificação confiável, mas se a sua organização usa os Serviços de Certificados do Active Directory, você pode criar seus próprios certificados.

    O *nome alternativo da entidade* é usado pelo DRS (serviço de registro de dispositivo) para habilitar o acesso de dispositivos externos. O formato deve ser *enterpriseregistration.contoso.com*.

    Para obter mais informações, consulte [Obter e configurar um certificado de protocolo SSL para o AD FS][adfs_certificates].

2. No controlador de domínio, gere uma nova chave de raiz para o Serviço de Distribuição de Chave. Defina o tempo efetivo para a hora atual menos 10 horas (essa configuração reduz o atraso que pode ocorrer na distribuição e na sincronização de chaves no domínio). Essa etapa é necessária para dar suporte à criação da conta de serviço de grupo que é usada para executar o serviço do AD FS. O comando do PowerShell a seguir mostra um exemplo de como fazer isso:

    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. Adicione cada VM de servidor do AD FS no domínio.

> [!NOTE]
> Para instalar o AD FS, o controlador de domínio que executa a função FSMO (Flexible Single Master Operation) do emulador do PDC (controlador de domínio primário) do domínio deve estar em execução e acessível para as VMs do AD FS. <<RBC: Existe alguma maneira de deixar isso menos repetitivo?>>
>

### <a name="ad-fs-trust"></a>Relação de confiança do AD FS

Estabelece a relação de confiança de federação entre a instalação do AD FS e os servidores de federação de qualquer organização parceira. Configure as filtragens e os mapeamentos de declaração necessários.

- A equipe de DevOps em cada organização parceira deve adicionar um objeto de confiança de terceira parte confiável para os aplicativos Web acessíveis por meio dos servidores do AD FS.
- A equipe de DevOps em sua organização deve configurar a relação de confiança do provedor de declarações para permitir que os servidores do AD FS confiem nas declarações que as organizações parceiras fornecem.
- A equipe de DevOps na organização também deve configurar o AD FS para transmitir declarações para os aplicativos Web da organização.

Para obter mais informações, consulte [Establishing Federation Trust][establishing-federation-trust] (Estabelecendo uma relação de confiança de federação).

Publique os aplicativos Web da organização e disponibilize-os a parceiros externos usando a pré-autenticação por meio dos servidores do WAP. Para obter mais informações, consulte [Publicar aplicativos usando a pré-autenticação do AD FS][publish_applications_using_AD_FS_preauthentication]

O AD FS dá suporte para aumento e transformação de token. O Azure Active Directory não oferece esse recurso. Com o AD FS, ao configurar as relações de confiança, você pode:

- Configurar transformações de declaração para regras de autorização. Por exemplo, você pode mapear a segurança do grupo de uma representação usada por uma organização que não seja parceira da Microsoft para algo que o Active Directory DS possa autorizar em sua organização.
- Transforme declarações de um formato para outro. Por exemplo, você poderá mapear de SAML 2.0 para SAML 1.1 se seu aplicativo somente der suporte para declarações SAML 1.1.

### <a name="ad-fs-monitoring"></a>Monitoramento do AD FS

O [Pacote de Gerenciamento do Microsoft System Center para Serviços de Federação do Active Directory (AD FS) 2012 R2][oms-adfs-pack] fornece monitoramento proativo e reativo de sua implantação do AD FS para o servidor de federação. Este pacote de gerenciamento monitora:

- Eventos que o serviço do AD FS registra em seus logs de eventos.
- Os dados de desempenho que os contadores de desempenho do AD FS coletam.
- A integridade geral do sistema AD FS e dos aplicativos Web (terceiras partes confiáveis) e fornece alertas sobre problemas críticos e avisos.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

As considerações a seguir, resumidas do artigo [Planejar sua implantação do AD FS][plan-your-adfs-deployment], oferecem um ponto inicial para o dimensionamento de farms do AD FS:

- Se você tiver menos de mil usuários, não crie servidores dedicados. Nesse caso, instale o AD FS em cada um dos servidores do Active Directory DS na nuvem. Verifique se há pelo menos dois servidores do Active Directory DS para manter a disponibilidade. Crie um único servidor do WAP.
- Se você tiver entre mil e 15 mil usuários, crie dois servidores do AD FS dedicados e dois servidores do WAP dedicados.
- Se você tiver entre 15 mil e 60 mil usuários, crie entre três e cinco servidores do AD FS dedicados e pelo menos dois servidores do WAP dedicados.

Essas considerações presumem que você esteja usando um tamanho de VM quad-core dual (D4_v2 padrão ou melhor) no Azure.

Se você estiver usando o Banco de Dados Interno do Windows para armazenar dados de configuração do AD FS, haverá o limite de oito servidores do AD FS no farm. Se você conseguir prever que precisará de mais no futuro, use o SQL Server. Para obter mais informações, consulte [A função do banco de dados de configuração do AD FS][adfs-configuration-database].

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Crie um farm do AD FS com pelo menos dois servidores para aumentar a disponibilidade do serviço. Use contas de armazenamento diferentes para cada VM do AD FS no farm. Essa abordagem ajuda a garantir que uma falha em uma única conta de armazenamento não deixe todo o farm inacessível.

Crie conjuntos de disponibilidade do Azure separados para as VMs do AD FS e do WAP. Verifique se há pelo menos duas VMs em cada conjunto. Cada conjunto de disponibilidade deve ter pelo menos dois domínios de atualização e dois domínios de falha.

Configure os balanceadores de carga das VMs do AD FS e das VMs do WAP da seguinte maneira:

- Use um Azure Load Balancer para fornecer acesso externo para as VMs do WAP e um balanceador de carga interno para distribuir a carga entre os servidores do AD FS no farm.
- Passe apenas o tráfego que aparece na porta 443 (HTTPS) para os servidores do AD FS/WAP.
- Forneça um endereço IP estático ao balanceador de carga.
- Crie uma investigação de integridade usando HTTP em `/adfs/probe`. Para saber mais, veja [Verificações de integridade do balanceador de carga de hardware e o proxy de aplicativo Web/AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/).

  > [!NOTE]
  > Os servidores do AD FS usam o protocolo SNI (Indicação de Nome de Servidor), portanto, haverá falha na tentativa de investigar usando um ponto de extremidade HTTPS do balanceador de carga.
  >

- Adicione um registro DNS *A* para o domínio do balanceador de carga do AD FS. Especifique o endereço IP do balanceador de carga e nomeie-o no domínio (como adfs.contoso.com). Esse é o nome que os clientes e os servidores do WAP usam para acessar o farm de servidores do AD FS.

Você pode usar o SQL Server ou o Banco de Dados Interno do Windows para manter as informações de configuração do AD FS. O Banco de Dados Interno do Windows fornece redundância básica. As alterações são gravadas diretamente em apenas um dos bancos de dados do AD FS no cluster do AD FS, enquanto os outros servidores usam a replicação pull para manter seus bancos de dados atualizados. O uso do SQL Server pode fornecer redundância total de banco de dados e alta disponibilidade usando clustering de failover ou espelhamento.

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

A equipe de DevOps deve estar preparada para executar as seguintes tarefas:

- Gerenciamento dos servidores de federação, incluindo o gerenciamento de farm do AD FS, gerenciamento de política de confiança em servidores de federação e gerenciamento dos certificados usados pelos serviços de federação.
- Gerenciamento de servidores do WAP, incluindo o gerenciamento de farm e certificados do WAP.
- Gerenciamento de aplicativos Web, incluindo configuração de terceiras partes confiáveis, métodos de autenticação e mapeamentos de declarações.
- Fazendo backup de componentes do AD FS.

## <a name="security-considerations"></a>Considerações de segurança

O AD FS usa HTTPS, ou seja, verifique se as regras do NSG da sub-rede que contém as VMs da camada da Web permitem solicitações HTTPS. Essas solicitações podem ser originadas na rede local, nas sub-redes que contém a camada da Web, na camada de negócios, na camada de dados, no DMZ privado, no DMZ público e na sub-rede que contém os servidores do AD FS.

Evite a exposição direta dos servidores do AD FS à Internet. Os servidores do AD FS são computadores ingressados no domínio que têm autorização total para conceder tokens de segurança. Se um servidor estiver comprometido, um usuário mal-intencionado poderá emitir tokens de acesso completo a todos os aplicativos Web e a todos os servidores de federação protegidos pelo AD FS. Se o sistema precisar lidar com solicitações de usuários externos que não se conectam de sites de parceiros confiáveis, use os servidores do WAP para lidar com essas solicitações. Para obter mais informações, consulte [Onde colocar um Proxy do servidor de Federação][where-to-place-an-fs-proxy].

Coloque os servidores do AD FS e os servidores do WAP em sub-redes separadas com seus próprios firewalls. Você pode usar as regras do NSG para definir as regras de firewall. Todos os firewalls devem permitir o tráfego na porta 443 (HTTPS).

Restrinja o acesso de entrada direta para os servidores do AD FS e do WAP. Equipe de DevOps só deve ser capaz de se conectar. Não ingresse os servidores do WAP no domínio.

Considere usar um conjunto de soluções de virtualização de rede que registre informações detalhadas sobre o tráfego que atravessa a borda da sua rede virtual para fins de auditoria.

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação para essa arquitetura está disponível no [GitHub][github]. Observe que toda a implantação pode levar até duas horas, o que inclui a criação do gateway de VPN e a execução dos scripts que configuram o Active Directory e o AD FS.

### <a name="prerequisites"></a>Pré-requisitos

1. Clone, crie um fork ou baixe o arquivo zip do [repositório GitHub](https://github.com/mspnp/identity-reference-architectures).

1. Instale a [CLI 2.0 do Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

1. Instale o pacote npm dos [Blocos de construção do Azure](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks).

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

1. Em um prompt de comando, prompt do bash ou prompt do PowerShell, entre na sua conta do Azure da seguinte maneira:

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>Implantar o datacenter local simulado

1. Navegue até a pasta `adfs` do repositório do GitHub.

1. Abra o arquivo `onprem.json` . Procure instâncias de `adminPassword`, `Password` e `SafeModeAdminPassword` e atualize as senhas.

1. Execute o seguinte comando e aguarde a conclusão da implantação:

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-infrastructure"></a>Implantar a infraestrutura do Azure

1. Abra o arquivo `azure.json` .  Pesquise instâncias de `adminPassword` e `Password` adicione valores para as senhas.

1. Execute o seguinte comando e aguarde a conclusão da implantação:

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p azure.json --deploy
    ```

### <a name="set-up-the-ad-fs-farm"></a>Configurar o farm do AD FS

1. Abra o arquivo `adfs-farm-first.json` .  Pesquise `AdminPassword` e substitua a senha padrão.

1. Execute o comando a seguir:

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-first.json --deploy
    ```

1. Abra o arquivo `adfs-farm-rest.json` .  Pesquise `AdminPassword` e substitua a senha padrão.

1. Execute o seguinte comando e aguarde a conclusão da implantação:

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-rest.json --deploy
    ```

### <a name="configure-ad-fs-part-1"></a>Configurar o AD FS (parte 1)

1. Abra uma sessão de área de trabalho remota para a VM chamada `ra-adfs-jb-vm1`, que é a VM de jumpbox. O nome de usuário é `testuser`.

1. No jumpbox, abra uma sessão de área de trabalho remota para a VM chamada `ra-adfs-proxy-vm1`. O endereço IP privado é 10.0.6.4.

1. Nessa sessão da área de trabalho remota, execute o [ISE do PowerShell](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).

1. No PowerShell, navegue até o seguinte diretório:

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. Cole o código abaixo em um painel de script e execute-o:

    ```powershell
    . .\adfs-webproxy.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxyApp -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxyApp
    ```

    No prompt `Get-Credential`, digite a senha que você especificou no arquivo de parâmetros de implantação.

1. Execute o seguinte comando para monitorar o progresso da definição da [DSC](/powershell/dsc/overview/overview):

    ```powershell
    Get-DscConfigurationStatus
    ```

    Pode demorar alguns minutos até ficar consistente. Durante esse tempo, você poderá ver erros do comando. Quando a configuração for bem-sucedida, a saída deverá ser semelhante à seguinte:

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

### <a name="configure-ad-fs-part-2"></a>Configurar o AD FS (parte 2)

1. No jumpbox, abra uma sessão de área de trabalho remota para a VM chamada `ra-adfs-proxy-vm2`. O endereço IP privado é 10.0.6.5.

1. Nessa sessão da área de trabalho remota, execute o [ISE do PowerShell](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).

1. Navegue até o seguinte diretório:

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. Cole o que vem abaixo em um painel de script e execute-o:

    ```powershell
    . .\adfs-webproxy-rest.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxy -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxy
    ```

    No prompt `Get-Credential`, digite a senha que você especificou no arquivo de parâmetros de implantação.

1. Execute o seguinte comando para monitorar o progresso da definição da DSC:

    ```powershell
    Get-DscConfigurationStatus
    ```

    Pode demorar alguns minutos até ficar consistente. Durante esse tempo, você poderá ver erros do comando. Quando a configuração for bem-sucedida, a saída deverá ser semelhante à seguinte:

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

    Às vezes, essa DSC falha. Se a verificação de status mostra `Status=Failure` e `Type=Consistency`, tente executar novamente a etapa 4.

### <a name="sign-into-ad-fs"></a>Entrar no AD FS

1. No jumpbox, abra uma sessão de área de trabalho remota para a VM chamada `ra-adfs-adfs-vm1`. O endereço IP privado é 10.0.5.4.

1. Siga as etapas em [Habilitar a página de logon iniciada pelo IDP](/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-initiatedsignon#enable-the-idp-intiated-sign-on-page) para habilitar a página de logon.

1. No jumpbox, navegue até `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm`. Você poderá receber um aviso de certificado que pode ser ignorado para esse teste.

1. Verifique se a página de conexão da Contoso Corporation é exibida. Entre como **contoso\testuser**.

<!-- links -->
[extending-ad-to-azure]: adds-extend-domain.md

[vm-recommendations]: ../virtual-machines-windows/single-vm.md
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md
[hybrid-azure-on-prem-vpn]: ../hybrid-networking/vpn.md

[azure-cli]: /azure/azure-resource-manager/xplat-cli-azure-resource-manager
[DRS]: https://technet.microsoft.com/library/dn280945.aspx
[where-to-place-an-fs-proxy]: https://technet.microsoft.com/library/dd807048.aspx
[ADDRS]: https://technet.microsoft.com/library/dn486831.aspx
[plan-your-adfs-deployment]: https://msdn.microsoft.com/library/azure/dn151324.aspx
[ad_network_recommendations]: #network_configuration_recommendations_for_AD_DS_VMs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[create_service_account_for_adfs_farm]: https://technet.microsoft.com/library/dd807078.aspx
[adfs-configuration-database]: https://technet.microsoft.com/library/ee913581(v=ws.11).aspx
[active-directory-federation-services]: /windows-server/identity/active-directory-federation-services
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  /windows-server/identity/ad-fs/deployment/deploying-a-federation-server-farm
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: /powershell/azure/overview
[aad]: /azure/active-directory/
[aadb2c]: /azure/active-directory-b2c/
[adfs-intro]: /azure/active-directory/hybrid/whatis-hybrid-identity
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
