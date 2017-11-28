---
title: Criar uma floresta de recursos do AD DS no Azure
description: "Como criar um domínio confiável do Active Directory no Azure.\nguidance,vpn-gateway,expressroute,load-balancer,virtual-network,active-directory"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: bb7e57af2afacf1faa7679c854bf49217918eba8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a>Criar uma floresta de recursos do AD DS (Active Directory Domain Services) no Azure

Essa arquitetura de referência mostra como criar um domínio separado do Active Directory no Azure que é de confiança nos domínios na sua floresta local do AD. [**Implantar esta solução**.](#deploy-the-solution)

[![0]][0] 

*Baixe um [arquivo Visio][visio-download] dessa arquitetura.*

O AD DS (Active Directory Domain Services) armazena informações de identidade em uma estrutura hierárquica. O nó superior na estrutura hierárquica é conhecido como uma floresta. Uma floresta contém domínios e domínios contêm outros tipos de objetos. Essa arquitetura de referência cria uma floresta do AD DS no Azure com uma relação de confiança de saída unidirecional com um domínio local. A floresta no Azure contém um domínio que não existe localmente. Devido à relação de confiança, os logons realizados nos domínios locais pode ser confiáveis para acessar recursos em um domínio separado do Azure. 

Usos típicos para essa arquitetura incluem manter uma separação segurança para objetos e identidades mantidos na nuvem e migrar os domínios individuais do local para a nuvem. 

Para ver considerações adicionais, consulte [Escolher uma solução para integrar o Active Directory local ao Azure][considerations]. 

## <a name="architecture"></a>Arquitetura

A arquitetura tem os seguintes componentes.

* **Rede local**. A rede local contém sua própria floresta e domínios do Active Directory.
* **Servidores do Active Directory**. Esses são controladores de domínio que implementam serviços de domínio em execução como VMs na nuvem. Esses servidores hospedam uma floresta que contém um ou mais domínios, separados daqueles locais.
* **Relação de confiança unidirecional**. O exemplo no diagrama mostra uma relação de confiança unidirecional do domínio no Azure para o domínio local. Essa relação permite que os usuários locais acessem os recursos no domínio no Azure, mas não o oposto. É possível criar uma relação de confiança bidirecional se os usuários de nuvem também precisarem de acesso aos recursos locais.
* **Sub-rede do Active Directory**. Os servidores do AD DS são hospedados em uma sub-rede separada. As regras de NSG (grupo de segurança de rede) protegem os servidores do AD DS e fornecem um firewall em tráfego de fontes inesperadas.
* **Gateway do Azure**. O gateway do Azure fornece uma conexão entre a rede local e a VNet do Azure. Essa pode ser uma [conexão VPN][azure-vpn-gateway] ou do [Azure ExpressRoute][azure-expressroute]. Para obter mais informações, consulte [Implementar uma arquitetura de rede híbrida segura no Azure][implementing-a-secure-hybrid-network-architecture].

## <a name="recommendations"></a>Recomendações

Para obter recomendações específicas sobre a implementação do Active Directory no Azure, consulte os seguintes artigos:

- [Extensão do AD DS (Active Directory Domain Services) para o Azure][adds-extend-domain]. 
- [Diretrizes para implantar o Active Directory do Windows Server em máquinas virtuais do Azure][ad-azure-guidelines].

### <a name="trust"></a>Confiar

Os domínios locais estão contidos em uma floresta diferente dos domínios na nuvem. Para habilitar a autenticação de usuários locais na nuvem, os domínios no Azure precisam confiar no domínio de logon na floresta local. Da mesma forma, se a nuvem fornece um domínio de logon para usuários externos, pode ser necessário que a floresta local confie no domínio de nuvem.

Você pode estabelecer relações de confiança no nível de floresta [criando relações de confiança de floresta][creating-forest-trusts] ou no nível do domínio [criando relações de confiança externas][creating-external-trusts]. Uma confiança no nível de floresta cria uma relação entre todos os domínios nas duas florestas. Uma relação de confiança no nível de domínio externo só cria uma relação entre dois domínios especificados. Você só deve criar relações de confiança no nível de domínio externo entre domínios de florestas diferentes.

Relações de confiança podem ser unidirecionais ou bidirecionais:

* Uma relação de confiança unidirecional permite que os usuários em um domínio ou floresta (conhecido como o domínio ou a floresta de *entrada*) para acessar os recursos contidos em outro (o domínio ou a floresta de *saída*).
* Uma relação de confiança bidirecional permite que os usuários no domínio ou floresta acessem os recursos mantidos no outro.

A tabela a seguir resume as configurações de confiança para alguns cenários simples:

| Cenário | Relação de confiança local | Relação de confiança na nuvem |
| --- | --- | --- |
| Os usuários locais precisam de acesso aos recursos na nuvem, mas não vice-versa |Unidirecional de entrada |Unidirecional de saída |
| Os usuários na nuvem precisam de acesso aos recursos locais, mas não vice-versa |Unidirecional de saída |Unidirecional de entrada |
| Os usuários na nuvem e local precisam de acesso aos recursos mantidos tanto na nuvem quanto locais |Bidirecional de entrada e saída |Bidirecional de entrada e saída |

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

O Active Directory é automaticamente escalonável para controladores de domínio que fazem parte do mesmo domínio. As solicitações são distribuídas por todos os controladores em um domínio. Você pode adicionar outro controlador de domínio e ele é sincronizado automaticamente com o domínio. Não configure um balanceador de carga separado para direcionar o tráfego para os controladores no domínio. Verifique se todos os controladores de domínio têm recursos de memória e armazenamento suficientes para lidar com o banco de dados do domínio. Deixe todas as VMs do controlador de domínio com o mesmo tamanho.

## <a name="availability-considerations"></a>Considerações sobre disponibilidade

Provisione pelo menos dois controladores de domínio para cada domínio. Isso habilita a replicação automática entre os servidores. Crie um conjunto de disponibilidade para as VMs que atuam como servidores do Active Directory que tratam de cada domínio. Coloque pelo menos dois servidores neste conjunto de disponibilidade.

Além disso, considere a possibilidade de designar um ou mais servidores em cada domínio como [mestres de operações em espera][standby-operations-masters] em caso de falha de conectividade com um servidor que atua na função de FSMO (operações principais únicas flexíveis).

## <a name="manageability-considerations"></a>Considerações sobre capacidade de gerenciamento

Para obter informações sobre as considerações de gerenciamento e monitoramento, consulte [Estendendo o Active Directory para o Azure][adds-extend-domain]. 
 
Para obter informações adicionais, consulte [Monitoramento do Active Directory][monitoring_ad]. Você pode instalar ferramentas como o [Microsoft Systems Center][microsoft_systems_center] em um servidor de monitoramento na sub-rede de gerenciamento para ajudar a executar essas tarefas.

## <a name="security-considerations"></a>Considerações de segurança

As relações de confiança no nível de floresta são transitivas. Se você estabelecer uma relação de confiança no nível de floresta entre uma floresta local e uma floresta na nuvem, essa relação de confiança é estendida para outros novos domínios criados em uma dessas florestas. Se você usar domínios para fornecer separação para fins de segurança, considere criar relações de confiança somente no nível de domínio. As relações de confiança no nível de domínio são não transitivas.

Para considerações de segurança específicas do Active Directory, consulte a seção de considerações de segurança em [Estendendo o Active Directory para o Azure][adds-extend-domain].

## <a name="deploy-the-solution"></a>Implantar a solução

Uma solução está disponível no [GitHub][github] para implantar essa arquitetura de referência. Você precisará da versão mais recente da CLI do Azure para executar o script do Powershell que implanta a solução. Para implantar a arquitetura de referência, siga estas etapas:

1. Baixe ou clone a pasta da solução do [Github] [ github] para seu computador local.

2. Abra a CLI do Azure e navegue até a pasta da solução local.

3. Execute o comando a seguir:
   
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    Substitua `<subscription id>` por sua ID da assinatura do Azure.
   
    Para `<location>`, especifique uma região do Azure, tal como `eastus` ou `westus`.
   
    O parâmetro `<mode>` controla a granularidade da implantação e pode ser um dos seguintes valores:
   
   * `Onpremise`: implanta o ambiente local simulado.
   * `Infrastructure`: implanta a infraestrutura e jump box da VNet no Azure.
   * `CreateVpn`: implanta o gateway de rede virtual do Azure e conecta-o à rede local simulada.
   * `AzureADDS`: implanta as VMs que atuam como servidores do Active Directory DS, implante o Active Directory nessas VMs e implanta o domínio no Azure.
   * `WebTier`: implanta as VMs e o balanceador de carga na camada da Web.
   * `Prepare`: implanta todas as implantações anteriores. **Essa é a opção recomendada se você não tem uma rede local existente, mas deseja implantar a arquitetura de referência completa descrita acima para teste ou avaliação.** 
   * `Workload`: implanta as VMs e o balanceador de carga na camada de dados e comercial. Observe que essas VMs não são incluídas na implantação `Prepare`.

4. Aguarde até que a implantação seja concluída. Se você estiver realizando a implantação de `Prepare`, isso levará várias horas.
     
5. Se você estiver usando a configuração local simulada, defina a relação de confiança de entrada:
   
   1. Conecte-se ao jump box (*ra-adtrust-mgmt-vm1* no grupo de recursos *ra-adtrust-security-rg*). Faça logon como *testuser* com a senha *AweS0me@PW*.
   2. No jump box, abra uma sessão RDP na primeira VM no domínio *contoso.com* (o domínio local). Essa VM tem o endereço IP 192.168.0.4. O nome de usuário é *contoso\testuser* com a senha *AweS0me@PW*.
   3. Baixe o script [incoming-trust.ps1][incoming-trust] e execute-o para criar a relação de confiança de entrada do *treyresearch.com*.

6. Se você estiver usando sua própria infraestrutura local:
   
   1. Baixe o script [incoming-trust.ps1][incoming-trust].
   2. Edite o script e substitua o valor da variável `$TrustedDomainName` pelo nome do seu próprio domínio.
   3. Execute o script.

7. No jump-box, conecte-se à primeira VM no domínio *treyresearch.com* (o domínio na nuvem). Essa VM tem o endereço IP 10.0.4.4. O nome de usuário é *treyresearch\testuser* com a senha *AweS0me@PW*.

8. Baixe o script [outgoing-trust.ps1][outgoing-trust] e execute-o para criar a relação de confiança de entrada do *treyresearch.com*. Se você estiver usando seus próprios computadores locais, edite o script primeiro. Definir a variável `$TrustedDomainName` para o nome do seu domínio local e especifique os endereços IP dos servidores do Active Directory DS para esse domínio na variável `$TrustedDomainDnsIpAddresses`.

9. Aguarde alguns minutos para que as etapas anteriores sejam concluídas, conecte-se a uma VM local e execute as etapas descritas no artigo [Verificar uma relação de confiança][verify-a-trust] para determinar se a relação de confiança entre os domínios *contoso.com* e *treyresearch.com* está configurada corretamente.

## <a name="next-steps"></a>Próximas etapas

* Conheça as práticas recomendadas para [estender seu domínio do AD DS local para o Azure][adds-extend-domain]
* Conheça as práticas recomendadas para [criar uma infraestrutura do AD FS][adfs] no Azure.

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "Proteger a arquitetura de rede híbrida com domínios separados do Active Directory"