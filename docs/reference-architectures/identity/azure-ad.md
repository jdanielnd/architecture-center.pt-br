---
title: Integrar domínios do AD local ao Azure Active Directory
description: Como implementar uma arquitetura de rede híbrida segura usando o Azure Active Directory.
author: telmosampaio
pnp.series.title: Identity management
ms.date: 11/28/2016
pnp.series.next: adds-extend-domain
pnp.series.prev: ./index
cardTitle: Integrate on-premises AD with Azure AD
ms.openlocfilehash: 9475d669b2cb8888a7ceabed7e36317fe63681fd
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/23/2018
---
# <a name="integrate-on-premises-active-directory-domains-with-azure-active-directory"></a>Integrar domínios do Active Directory local ao Azure Active Directory

O Azure Active Directory (Azure AD) é um serviço de identidade e de diretório multilocatário baseado em nuvem. Esta arquitetura de referência mostra as práticas recomendadas para a integração de domínios do Active Directory local com o Azure AD para fornecer autenticação de identidade baseada em nuvem. [**Implante essa solução**.](#deploy-the-solution)

[![0]][0] 

*Baixe um [Arquivo Visio][visio-download] dessa arquitetura.*

> [!NOTE]
> Para simplificar, este mostra apenas as conexões relacionadas diretamente ao Azure AD e não o tráfego relacionado a protocolo que pode ocorrer como parte da federação de identidade e da autenticação. Por exemplo, um aplicativo Web pode redirecionar o navegador da Web para autenticar a solicitação por meio do Azure AD. Uma vez autenticada, a solicitação pode ser retornada ao aplicativo Web, com as informações de identidade apropriadas.
> 

Alguns usos típicos dessa arquitetura de referência:

* Aplicativos Web implantados no Azure que fornecem acesso a usuários remotos que pertencem à sua organização.
* Implementação de recursos de autoatendimento para usuários finais, como redefinir senhas e delegar gerenciamento de grupo. Observe que isso exige o Azure AD Premium Edition.
* Arquiteturas em que a rede local e a VNet do Azure do aplicativo não estão conectadas usando um túnel de VPN ou um circuito ExpressRoute.

> [!NOTE]
> No momento, o Azure AD dá suporte somente para autenticação do usuário. Alguns aplicativos e serviços, como o SQL Server, podem exigir a autenticação do computador e, nesse caso, essa solução não é apropriada.
> 

Para obter considerações adicionais, consulte [Escolher uma solução para a integração do Active Directory local ao Azure][considerations]. 

## <a name="architecture"></a>Arquitetura

A arquitetura tem os seguintes componentes.

* **Locatário do Azure AD**. Uma instância do [Azure AD][azure-active-directory] criada pela sua organização. Ela atua como um serviço de diretório para aplicativos de nuvem armazenando objetos copiados do Active Directory local e fornece serviços de identidade.
* **Sub-rede da camada da Web**. Essa sub-rede contém as VMs que executam um aplicativo Web. O Azure AD pode agir como um agente de identidade para este aplicativo.
* **Servidor do AD DS local**. Um serviço de identidade e diretório local. O diretório do AD DS pode ser sincronizado com o Azure AD para habilitá-lo a autenticar usuários locais.
* **Servidor de sincronização do Azure AD Connect**. Um computador local que executa o serviço de sincronização do [Azure AD Connect][azure-ad-connect]. Este serviço sincroniza as informações mantidas no Active Directory local com o Azure AD. Por exemplo, se você provisionar ou desprovisionar usuários e grupos locais, essas alterações serão propagadas para o Azure AD. 
  
  > [!NOTE]
  > Por motivos de segurança, o Azure AD armazena as senhas de usuário como um hash. Se algum usuário precisar de uma redefinição de senha, isso deverá ser realizado localmente e o novo hash deverá ser enviado ao Azure AD. As edições Azure AD Premium incluem recursos que podem automatizar essa tarefa para permitir que os usuários redefinam suas próprias senhas.
  > 

* **VMs para aplicativos de N camadas**. A implantação inclui a infraestrutura para um aplicativo de N camadas. Para obter mais informações sobre esses recursos, consulte [Executar VMs do Windows para um aplicativo de N camadas][implementing-a-multi-tier-architecture-on-Azure].

## <a name="recommendations"></a>Recomendações

As seguintes recomendações aplicam-se à maioria dos cenários. Siga estas recomendações, a menos que você tenha um requisito específico que as substitua. 

### <a name="azure-ad-connect-sync-service"></a>Serviço de sincronização Azure AD Connect

O serviço de sincronização do Azure AD Connect garante que essas informações de identidade armazenadas na nuvem fiquem consistentes com as mantidas localmente. Instale esse serviço usando o software do Azure AD Connect. 

Antes de implementar a sincronização do Azure AD Connect, determine os requisitos de sincronização da sua organização. Por exemplo, o que deve ser sincronizado, de quais domínios e com que frequência. Para obter mais informações, consulte [Determinar os requisitos de sincronização de diretório][aad-sync-requirements].

Você pode executar o serviço de sincronização do Azure AD Connect em uma VM ou em um computador hospedado localmente. Dependendo da volatilidade das informações no seu diretório do Active Directory, a carga no serviço de sincronização do Azure AD Connect provavelmente não será alta após a sincronização inicial com o Azure AD. Executar o serviço em uma VM torna mais fácil expandir o servidor, se necessário. Monitore a atividade na VM conforme descrito na seção Considerações de monitoramento para determinar se o dimensionamento é necessário.

Se você tiver vários domínios locais em uma floresta, será recomendável armazenar e sincronizar as informações em toda a floresta para um único locatário do Azure AD. Filtre as informações por identidades que ocorrem em mais de um domínio, para que cada identidade apareça apenas uma vez no Azure AD e não seja duplicada. A duplicação poderá levar a inconsistências quando os dados forem sincronizados. Para obter mais informações, consulte a seção Topologia abaixo. 

Use a filtragem para que apenas os dados necessários sejam armazenados no Azure AD. Por exemplo, talvez sua organização não queira armazenar informações sobre contas inativas no Azure AD. A filtragem pode ser baseada em grupo, baseada em domínio, baseada em UO (unidade organizacional) ou baseada em atributo. É possível combinar filtros para gerar regras mais complexas. Por exemplo, você pode sincronizar objetos mantidos em um domínio que tenham um valor específico em um atributo selecionado. Para obter informações detalhadas, consulte [Sincronização do Azure AD Connect: configurar a filtragem][aad-filtering].

Para implementar a alta disponibilidade para o serviço de sincronização do AD Connect, execute um servidor de preparo secundário. Para obter mais informações, consulte a seção Recomendações de topologia.

### <a name="security-recommendations"></a>Recomendações de segurança

**Gerenciamento de senha do usuário.** As edições Azure AD Premium dão suporte para write-back de senha, permitindo que os usuários locais executem redefinições de senha de autoatendimento por meio do portal do Azure. Esse recurso somente deverá ser habilitado depois que você examinar a política de segurança de senha da organização. Por exemplo, você pode restringir quais usuários podem alterar senhas e pode personalizar a experiência de gerenciamento de senha. Para obter mais informações, consulte [Personalizar o gerenciamento de senha para de adequar às necessidades da sua organização][aad-password-management]. 

**Proteja aplicativos locais que podem ser acessados externamente.** Use o proxy de aplicativo do Azure AD para fornecer acesso controlado a aplicativos Web locais para usuários externos por meio do Azure AD. Somente os usuários com credenciais válidas no diretório do Azure terão permissão para usar o aplicativo. Para obter mais informações, consulte o artigo [Habilitar o proxy de aplicativo no portal do Azure][aad-application-proxy].

**Monitore o Azure AD ativamente em busca de sinais de atividade suspeita.**    Considere o uso da edição Azure AD Premium P2, que inclui o Azure AD Identity Protection. O Identity Protection usa algoritmos de aprendizado de máquina adaptáveis e heurística para detectar anomalias e eventos de risco que podem indicar que uma identidade foi comprometida. Por exemplo, ele pode detectar uma atividade possivelmente incomum como atividades de entrada irregulares, entradas de fontes desconhecidas ou de endereços IP com atividade suspeita ou entradas de dispositivos que possam estar infectados. Usando esses dados, o Identity Protection gera relatórios e alertas que permitem investigar esses eventos de risco e executar ações apropriadas. Para obter mais informações, consulte [Azure Active Directory Identity Protection][aad-identity-protection].
  
Você pode usar o recurso de relatório do Azure AD no portal do Azure para monitorar atividades relacionadas à segurança que ocorrem no sistema. Para obter mais informações de como usar esses relatórios, consulte [Guia de relatórios do Azure Active Directory][aad-reporting-guide].

### <a name="topology-recommendations"></a>Recomendações de topologia

Configure o Azure AD Connect para implementar uma topologia que mais se aproxime dos requisitos da sua organização. Algumas topologias com suporte do Azure AD Connect:

* **Uma única floresta e um único diretório do Azure AD**. Nessa topologia, o Azure AD Connect sincroniza objetos e informações de identidade de um ou mais domínios em uma única floresta local em um único locatário do Azure AD. Essa é a topologia padrão implementada pela instalação expressa do Azure AD Connect.
  
  > [!NOTE]
  > Não use vários servidores de sincronização do Azure AD Connect para conectar diferentes domínios na mesma floresta local ao locatário do Azure AD, a menos que você esteja executando um servidor no modo de preparo descrito abaixo.
  > 
  > 

* **Várias florestas e um único diretório do Azure AD**. Nessa topologia, o Azure AD Connect sincroniza objetos e informações de identidade de várias florestas em um único locatário do Azure AD. Use essa topologia se sua organização tiver mais de uma floresta local. Você pode consolidar informações de identidade para que cada usuário único seja representado uma única vez no diretório do Azure AD, ainda que o mesmo usuário exista em mais de uma floresta. Todas as florestas usam o mesmo servidor de sincronização do Azure AD Connect. O servidor de sincronização do Azure AD Connect não precisa fazer parte de nenhum domínio, mas ele deve ser acessível para todas as florestas.
  
  > [!NOTE]
  > Nessa topologia, não use servidores de sincronização do Azure AD Connect separados para conectar cada floresta local a um único locatário do Azure AD. Isso poderá resultar na duplicação de informações de identidade no Azure AD se os usuários estiverem presentes em mais de uma floresta.
  > 
  > 

* **Várias florestas e topologias separadas**. Essa topologia mescla as informações de identidade de florestas separadas em um único locatário do Azure AD, tratando todas as florestas como entidades separadas. Essa topologia será útil se você estiver combinando florestas de diferentes organizações e as informações de identidade de cada usuário forem mantidas em uma única floresta.
  
  > [!NOTE]
  > Se as GALs (listas de endereços global) em cada floresta estiverem sincronizadas, um usuário em uma floresta poderá estar presente em outra como um contato. Isso poderá ocorrer se sua organização tiver implementado o GALSync com o Forefront Identity Manager 2010 ou o Microsoft Identity Manager 2016. Nesse cenário, você pode especificar que os usuários devem ser identificados pelo atributo *Mail*. Você também pode corresponder identidades usando os atributos *ObjectSID* e *msExchMasterAccountSID*. Isso será útil se você tiver uma ou mais florestas de recursos com contas desabilitadas.
  > 
  > 

* **Servidor de preparo**. Nessa configuração, você pode executar uma segunda instância do servidor de sincronização do Azure AD Connect em paralelo com a primeira. Essa estrutura dá suporte a cenários, como:
  
  * Alta disponibilidade.
  * Testar e implantar uma nova configuração do servidor de sincronização do Azure AD Connect.
  * Apresentando um novo servidor e desativando uma configuração antiga. 
    
    Nesses cenários, a segunda instância é executada no *modo de preparo*. O servidor registra os dados de sincronização e os objetos importados no banco de dados, mas não passa os dados para o Azure AD. Se você desabilitar o modo de preparo, o servidor começará a gravar dados no Azure AD e também começará a executar write-backs de senha nos diretórios locais, quando apropriado. Para obter mais informações, consulte [Sincronização do Azure AD Connect: considerações e tarefas operacionais][aad-connect-sync-operational-tasks].

* **Vários diretórios do Azure AD**. É recomendável criar um único diretório do Azure AD para uma organização, mas pode haver situações em que você precise particionar informações em diretórios do Azure AD separados. Nesse caso, evite problemas de sincronização e de write-back de senha garantindo que cada objeto da floresta local apareça apenas em um único diretório do Azure AD. Para implementar este cenário, configure servidores de sincronização do Azure AD Connect separados para cada diretório do Azure AD e use a filtragem de forma que cada servidor de sincronização do Azure AD Connect opere em um conjunto de objetos mutuamente exclusivo. 

Para obter mais informações sobre essas topologias, consulte [Topologias para o Azure AD Connect][aad-topologies].

### <a name="user-authentication"></a>Autenticação de usuário

Por padrão, o servidor de sincronização do Azure AD Connect configura a sincronização de hashes de senha entre o domínio local e o Azure AD, e o serviço do Azure AD pressupõe que os usuários se autenticam fornecendo a mesma senha que usam localmente. Para muitas organizações, isso é apropriado, mas você deve considerar as políticas e a infraestrutura existentes da sua organização. Por exemplo: 

* A política de segurança da sua organização pode proibir a sincronização de hashes de senha para a nuvem. Nesse caso, sua organização deve considerar a [autenticação de passagem](/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication).
* Você pode exigir que os usuários usem um SSO (logon único) perfeito ao acessar os recursos em nuvem por meio de computadores ingressados no domínio na rede corporativa.
* Talvez sua organização já tenha o Serviços de Federação do Active Directory (AD FS) ou uma implantação de provedor de federação de terceiros. Você pode configurar o Azure AD para usar essa infraestrutura para implementar a autenticação e SSO, em vez de usar informações de senha mantidas na nuvem.

Para obter mais informações, consulte [Opções de entrada de usuário do Azure AD Connect][aad-user-sign-in].

### <a name="azure-ad-application-proxy"></a>Proxy de aplicativo do Azure AD 

Use o Azure AD para fornecer acesso a aplicativos locais.

Exponha seus aplicativos Web locais usando conectores de proxy de aplicativo gerenciados pelo componente de proxy de aplicativo do Azure AD. O conector de proxy de aplicativo abre uma conexão de rede de saída com o proxy de aplicativo do Azure AD e as solicitações de usuários remotos são roteadas do Azure AD por essa conexão para os aplicativos Web. Isso elimina a necessidade de abrir portas de entrada no firewall local e reduz a superfície de ataque exposta pela sua organização.

Para obter mais informações, consulte [Publicar aplicativos usando o proxy de aplicativo do Azure AD][aad-application-proxy].

### <a name="object-synchronization"></a>Sincronização de objeto 

A configuração de padrão do Azure AD Connect sincroniza objetos do diretório do Active Directory local com base nas regras especificadas no artigo [Sincronização do Azure AD Connect: noções básicas sobre a configuração padrão][aad-connect-sync-default-rules]. Os objetos que atendem a essas regras são sincronizados enquanto todos os outros objetos são ignorados. Algumas regras de exemplo:

* Os objetos de usuário devem ter um único atributo *sourceAnchor* e o atributo *accountEnabled* deve estar populado.
* Os objetos de usuário devem ter um atributo *sAMAccountName* e não podem começar com o texto *Azure AD_* ou *MSOL_*.

O Azure AD Connect aplica várias regras a objetos de usuário, contato, grupo, ForeignSecurityPrincipal e computador. Use o Editor de Regras de Sincronização instalado com o Azure AD Connect, se você precisar modificar o conjunto de regras padrão. Para obter mais informações, consulte [Sincronização do Azure AD Connect: noções básicas sobre a configuração padrão][aad-connect-sync-default-rules]).

Você também pode definir seus próprios filtros para limitar os objetos a serem sincronizados por domínio ou OU (unidade organizacional). Como alternativa, você pode implementar uma filtragem personalizada mais complexa, como é descrito em [Sincronização do Azure AD Connect: configurar a filtragem][aad-filtering].

### <a name="monitoring"></a>Monitoramento 

O monitoramento de integridade é executado pelos seguintes agentes instalados localmente:

* O Azure AD Connect instala um agente que captura informações sobre operações de sincronização. Use a folha Azure AD Connect Health no portal do Azure para monitorar sua integridade e seu desempenho. Para obter mais informações, consulte [Usando o Azure AD Connect Health para sincronização][aad-health].
* Para monitorar a integridade dos domínios do AD DS e dos diretórios do Azure, instale o Azure AD Connect Health para agente do AD DS em um computador no domínio local. Use a folha do Azure Active Directory Connect Health no portal do Azure para monitoramento de integridade. Para obter mais informações, consulte [Usando o Azure AD Connect Health com o AD DS][aad-health-adds] 
* Instale o Azure AD Connect Health para agente do AD FS para monitorar a integridade dos serviços em execução local e use a folha Azure Active Directory Connect Health no portal do Azure para monitorar o AD FS. Para obter mais informações, consulte [Usando o Azure AD Connect Health com o AD FS][aad-health-adfs]

Para obter mais informações de como instalar os agentes do AD Connect Health e seus requisitos, consulte [Instalação do Agente do Azure AD Connect Health][aad-agent-installation].

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

O serviço do Azure AD dá suporte à escalabilidade baseada em réplicas, com uma única réplica primária que manipula operações de gravação e várias réplicas secundárias somente leitura. O Azure AD redireciona de forma transparente as tentativas de gravação feitas nas réplicas secundárias para a réplica primária e fornece consistência eventual. Todas as alterações feitas na réplica primária são propagadas para as réplicas secundárias. Essa arquitetura pode ser bem dimensionada porque a maioria das operações no Azure AD são leituras em vez de gravações. Para obter mais informações, consulte [Azure AD: Under the hood of our geo-redundant, highly available, distributed cloud directory][aad-scalability] (Azure AD: nos bastidores do nosso diretório na nuvem com redundância geográfica, distribuído e altamente disponível).

Para o servidor de sincronização do Azure AD Connect, determine quantos objetos você provavelmente vai sincronizar de seu diretório local. Se você tiver menos de 100 mil objetos, será possível usar o software do SQL Server Express LocalDB padrão fornecido com o Azure AD Connect. Se você tiver um número maior de objetos, será necessário instalar uma versão de produção do SQL Server e executar uma instalação personalizada do Azure AD Connect, especificando que ele deve usar uma instância existente do SQL Server.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

O serviço do Azure AD é distribuído geograficamente e é executado em vários data centers espalhados pelo mundo com failover automático. Se um data center fica indisponível, o Azure AD garante que seus dados de diretório estejam disponíveis para acesso de instância em pelo menos dois data centers dispersos mais regionalmente.

> [!NOTE]
> O SLA (Contrato de Nível de Serviço) dos serviços Azure AD Basic e Premium garante pelo menos 99,9% de disponibilidade. Não há um SLA para a Camada gratuita do Azure AD. Para obter mais informações, consulte [SLA para Azure Active Directory][sla-aad].
> 
> 

Considere provisionar uma segunda instância do servidor de sincronização do Azure AD Connect no modo de preparo para aumentar a disponibilidade, conforme foi discutido na seção de recomendações de topologia. 

Se você não estiver usando a instância do SQL Server Express LocalDB que vem com o Azure AD Connect, considere usar o clustering do SQL para obter alta disponibilidade. Soluções como espelhamento e Always On não têm suporte no Azure AD Connect.

Para obter considerações adicionais de como obter alta disponibilidade do servidor de sincronização do Azure AD Connect e também como se recuperar após uma falha, consulte [Sincronização do Azure AD Connect: considerações e tarefas operacionais][aad-sync-disaster-recovery].

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Há dois aspectos para gerenciar o Azure AD:

* Administrando o Azure AD na nuvem.
* Mantendo os servidores de sincronização do Azure AD Connect.

O Azure AD fornece as seguintes opções para o gerenciamento de domínios e diretórios na nuvem: 

* **Módulo PowerShell do Azure Active Directory**. Use este [módulo][aad-powershell] se precisar gerar script de tarefas administrativas comuns do Azure AD, como gerenciamento de usuários, gerenciamento de domínio e configuração de logon único.
* **Folha Gerenciamento do Azure AD no portal do Azure**. Essa folha fornece uma exibição de gerenciamento interativo do diretório e permite que você controle e configure a maioria dos aspectos do Azure AD. 

O Azure AD Connect instala as seguintes ferramentas para manter os serviços de sincronização do Azure AD Connect de suas máquinas locais:
  
* **Console do Microsoft Azure Active Directory Connect**. Essa ferramenta permite modificar a configuração do servidor do Azure AD Sync, personalizar como a sincronização ocorre, habilitar ou desabilitar o modo de preparo e alternar o modo de conexão do usuário. Observe que você pode habilitar a conexão do Serviço de Federação no Active Directory usando sua infraestrutura local.
* **Synchronization Service Manager**. Use a guia *Operações* nessa ferramenta para gerenciar o processo de sincronização e detectar se houve falha em alguma parte do processo. Você pode disparar sincronizações manualmente usando essa ferramenta. A guia *Conectores* permite que você controle as conexões dos domínios aos quais o mecanismo de sincronização está associado.
* **Editor de regras de sincronização**. Use essa ferramenta para personalizar a maneira como os objetos são transformados quando são copiados entre um diretório local e o Azure AD. Essa ferramenta permite que você especifique atributos e objetos adicionais para sincronização e, em seguida, executa filtros para determinar quais objetos devem ou não devem ser sincronizados. Para obter mais informações, consulte a seção Editor de regra de sincronização do documento [Sincronização do Azure AD Connect: noções básicas sobre a configuração padrão][aad-connect-sync-default-rules].

Para obter mais informações e dicas para o gerenciamento do Azure AD Connect, consulte [Sincronização do Azure AD Connect: práticas recomendadas para alterar a configuração padrão][aad-sync-best-practices].

## <a name="security-considerations"></a>Considerações de segurança

Use o controle de acesso condicional para negar solicitações de autenticação de fontes inesperadas:

- Dispare a [MFA (Autenticação Multifator) do Azure][azure-multifactor-authentication] se um usuário tentar se conectar de um local não confiável, como pela Internet, em vez de por uma rede confiável.

- Use o tipo de plataforma de dispositivo do usuário (iOS, Android, Windows Mobile, Windows) para determinar a política de acesso a aplicativos e recursos.

- Registre o estado de habilitação/desabilitação dos dispositivos dos usuários e incorpore essas informações nas verificações de política de acesso. Por exemplo, se o telefone de um usuário for perdido ou roubado, ele deverá ser gravado como desabilitado para impedir que seja usado para obter acesso.

- Controle o acesso de usuário a recursos com base na associação ao grupo. Use as [Regras de associação dinâmica do Azure AD][aad-dynamic-membership-rules] para simplificar a administração de grupo. Para obter uma visão geral de como isso funciona, consulte [Introduction to Dynamic Memberships for Groups][aad-dynamic-memberships] (Introdução às associações dinâmicas a grupos).

- Use políticas de risco de acesso condicional com o Azure AD Identity Protection para fornecer proteção avançada com base em atividades de conexão incomuns ou em outros eventos.

Para obter mais informações, consulte [Acesso condicional no Azure Active Directory][aad-conditional-access].

## <a name="deploy-the-solution"></a>Implantar a solução

Uma implantação de uma arquitetura de referência que implementa essas considerações e recomendações está disponível no GitHub. Essa arquitetura de referência implanta uma rede local simulada no Azure que você pode usar para testar e experimentar. A arquitetura de referência pode ser implantada com VMs Windows ou Linux seguindo as instruções abaixo: 

1. Clique no botão abaixo:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fidentity%2Fazure-ad%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Depois que o link for aberto no portal do Azure, você deve inserir valores para algumas das configurações: 
   * O nome do **Grupo de recursos** já está definido no arquivo de parâmetros, portanto, selecione **Criar novo** e digite `ra-aad-onpremise-rg` na caixa de texto.
   * Selecione a região na caixa suspensa **Local**.
   * Não edite as caixas de texto **URI da raiz do modelo** ou **URI da raiz do parâmetro**.
   * Selecione **Windows** ou **Linux** na caixa suspensa **Tipo de SO**.
   * Analise os termos e condições e clique na caixa de seleção **Concordo com os termos e condições declarados acima**.
   * Clique no botão **Comprar**.
3. Aguarde até que a implantação seja concluída.
4. Os arquivos de parâmetro incluem nomes e senhas de usuário administrador embutidos em código e é altamente recomendável alterá-los imediatamente em todas as VMs. Clique em cada VM no Portal do Azure e, em seguida, clique em **Redefinir senha** na folha **Suporte + solução de problemas**. Selecione **Redefinir senha** na caixa suspensa **Modo** e escolha um novo **Nome de usuário** e uma nova **Senha**. Clique no botão **Atualizar** para manter o novo nome de usuário e senha.

<!-- links -->

[implementing-a-multi-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[aad-agent-installation]: /azure/active-directory/active-directory-aadconnect-health-agent-install
[aad-application-proxy]: /azure/active-directory/active-directory-application-proxy-enable
[aad-conditional-access]: /azure/active-directory//active-directory-conditional-access
[aad-connect-sync-default-rules]: /azure/active-directory/active-directory-aadconnectsync-understanding-default-configuration
[aad-connect-sync-operational-tasks]: /azure/active-directory/active-directory-aadconnectsync-operations#staging-mode
[aad-dynamic-memberships]: https://youtu.be/Tdiz2JqCl9Q
[aad-dynamic-membership-rules]: /azure/active-directory/active-directory-accessmanagement-groups-with-advanced-rules
[aad-editions]: /azure/active-directory/active-directory-editions
[aad-filtering]: /azure/active-directory/active-directory-aadconnectsync-configure-filtering
[aad-health]: /azure/active-directory/active-directory-aadconnect-health-sync
[aad-health-adds]: /azure/active-directory/active-directory-aadconnect-health-adds
[aad-health-adfs]: /azure/active-directory/active-directory-aadconnect-health-adfs
[aad-identity-protection]: /azure/active-directory/active-directory-identityprotection
[aad-password-management]: /azure/active-directory/active-directory-passwords-customize
[aad-powershell]: https://msdn.microsoft.com/library/azure/mt757189.aspx
[aad-reporting-guide]: /azure/active-directory/active-directory-reporting-guide
[aad-scalability]: https://blogs.technet.microsoft.com/enterprisemobility/2014/09/02/azure-ad-under-the-hood-of-our-geo-redundant-highly-available-distributed-cloud-directory/
[aad-sync-best-practices]: /azure/active-directory/active-directory-aadconnectsync-best-practices-changing-default-configuration
[aad-sync-disaster-recovery]: /azure/active-directory/active-directory-aadconnectsync-operations#disaster-recovery
[aad-sync-requirements]: /azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements
[aad-topologies]: /azure/active-directory/active-directory-aadconnect-topologies
[aad-user-sign-in]: /azure/active-directory/active-directory-aadconnect-user-signin
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
[azure-multifactor-authentication]: /azure/multi-factor-authentication/multi-factor-authentication
[considerations]: ./considerations.md
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[sla-aad]: https://azure.microsoft.com/support/legal/sla/active-directory/v1_0/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx


[0]: ./images/azure-ad.png "Arquitetura de identidade de nuvem usando o Azure Active Directory"
