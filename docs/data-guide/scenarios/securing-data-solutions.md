---
title: Protegendo soluções de dados
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 57897c31a8abdcd801874bf92d60360f7a80d1fa
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/14/2018
ms.locfileid: "29288918"
---
# <a name="securing-data-solutions"></a>Protegendo soluções de dados

Para muitos, tornar os dados acessíveis na nuvem, especialmente ao realizar a transição do trabalho exclusivamente em armazenamentos de dados locais, pode causar preocupações sobre a maior acessibilidade aos dados e novas maneiras de protegê-los.

## <a name="challenges"></a>Desafios

* Centralizar o monitoramento e a análise de eventos de segurança armazenados em vários logs.
* Implementar o gerenciamento de criptografia e autorização em aplicativos e serviços.
* Garantir que o gerenciamento centralizado de identidade funciona em todos os componentes da solução, sejam locais ou na nuvem.

## <a name="data-protection"></a>Proteção de dados

A primeira etapa para a proteção de informações é identificar o que proteger. Desenvolva diretrizes claras, simples e bem comunicadas para identificar, proteger e monitorar os ativos de dados mais importantes em qualquer lugar em que eles residam. Estabeleça a proteção mais forte para ativos que têm um impacto desproporcional na missão ou na rentabilidade da organização. Eles são conhecidos como ativos de alto valor ou HVAs. Faça uma análise rigorosa das dependências de segurança e ciclo de vida do HVA e estabeleça condições e controles de segurança apropriados. Da mesma forma, identifique e classifique ativos confidenciais e defina as tecnologias e os processos para aplicar controles de segurança automaticamente.

Depois que os dados que você precisa proteger forem identificados, considere como você protegerá os dados *em repouso* e os dados *em trânsito*.

* **Dados em repouso**: dados que existem estaticamente em uma mídia física, seja magnética ou óptica, local ou na nuvem.
* **Dados em trânsito**: dados no momento de sua transferência entre componentes, locais ou programas, como pela rede, em um barramento de serviço (do local para a nuvem e vice-versa) ou durante um processo de entrada/saída.

Para saber mais sobre como proteger seus dados em repouso ou em trânsito, consulte [Melhores práticas de segurança de dados e criptografia do Azure](/azure/security/azure-security-data-encryption-best-practices).

## <a name="access-control"></a>Controle de Acesso

Um aspecto central da proteção dos dados na nuvem é uma combinação de gerenciamento de identidade e controle de acesso. Considerando a variedade e o tipo de serviços de nuvem, bem como a crescente popularidade da [nuvem híbrida](../scenarios/hybrid-on-premises-and-cloud.md), há várias práticas fundamentais que você deve seguir quando se trata de identidade e controle de acesso:

* Centralizar o gerenciamento de identidade.
* Habilitar o SSO (Logon Único).
* Implantar o gerenciamento de senhas.
* Impor o MFA (autenticação multifator) para usuários.
* Usar o RBAC (controle de acesso baseado em função).
* Políticas de Acesso Condicional devem ser configuradas, o que aprimora o conceito clássico de identidade do usuário com propriedades adicionais relacionadas ao local do usuário, tipo de dispositivo, nível de patch e assim por diante.
* Controlar os locais em que os recursos são criados usando o gerenciador de recursos.
* Monitorar ativamente as atividades suspeitas

Para obter mais informações, consulte [Melhores práticas do Azure Identity Management e segurança de controle de acesso](/azure/security/azure-security-identity-management-best-practices).

## <a name="auditing"></a>Auditoria

Além do monitoramento de identidade e acesso mencionado anteriormente, os serviços e aplicativos usados na nuvem devem gerar eventos relacionados à segurança que você possa monitorar. O principal desafio para o monitoramento desses eventos é lidar com as quantidades de logs, a fim de evitar problemas potenciais ou solucionar os problemas passados. Aplicativos baseados em nuvem tendem a conter muitas partes móveis, cuja maioria gera algum nível de log e telemetria. Use a análise e o monitoramento centralizado para ajudá-lo a gerenciar e entender a grande quantidade de informações.

Para obter mais informações, consulte [Log e auditoria do Azure](/azure/security/azure-log-audit).



## <a name="securing-data-solutions-in-azure"></a>Protegendo soluções de dados no Azure

### <a name="encryption"></a>Criptografia

**Máquinas virtuais**. Use o [Azure Disk Encryption](/azure/security/azure-security-disk-encryption) para criptografar os discos anexados em VMs Windows ou Linux. Essa solução é integrada ao [Azure Key Vault](/azure/key-vault/) para controlar e gerenciar as chaves de criptografia de disco e os segredos. 

**Armazenamento do Azure**. Use a [Criptografia de Serviço do Armazenamento do Azure](/azure/storage/common/storage-service-encryption) para criptografar dados em repouso automaticamente no Armazenamento do Azure. A criptografia, a descriptografia e o gerenciamento de chaves são totalmente transparentes para os usuários. Os dados também podem ser protegidos em trânsito por meio da criptografia do lado do cliente com o Azure Key Vault. Para obter mais informações, consulte [Criptografia do lado do cliente e Azure Key Vault para o Armazenamento do Microsoft Azure](/azure/storage/common/storage-client-side-encryption).

**Banco de Dados SQL** e **SQL Data Warehouse do Azure**. Use a [TDE](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) (Transparent Data Encryption) para realizar a criptografia e descriptografia em tempo real de bancos de dados, backups associados e arquivos de log de transações, sem a necessidade de nenhuma alteração nos aplicativos. O Banco de Dados SQL também pode usar o [Always Encrypted](/azure/sql-database/sql-database-always-encrypted-azure-key-vault) para ajudar a proteger dados confidenciais em repouso no servidor, durante a movimentação entre o cliente e o servidor e enquanto os dados estiverem em uso. Use o Azure Key Vault para armazenar as chaves de criptografia do Always Encrypted. 

### <a name="rights-management"></a>Gerenciamento de direitos

O [Azure Rights Management](/information-protection/understand-explore/what-is-azure-rms) é um serviço baseado em nuvem que usa políticas de criptografia, identidade e autorização para proteger arquivos e emails. Ele funciona em vários dispositivos – telefones, tablets e computadores. As informações podem ser protegidas em sua organização e fora dela, pois essa proteção permanece com os dados, mesmo quando eles saem dos limites da organização.

### <a name="access-control"></a>Controle de acesso

Use o [RBAC](/azure/active-directory/role-based-access-control-what-is) (Controle de Acesso Baseado em Função) para restringir o acesso aos recursos do Azure com base em funções de usuário. Caso esteja usando o Active Directory local, [sincronize com o Azure AD](/azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements) para fornecer aos usuários uma identidade de nuvem com base em suas respectivas identidades locais.

Use o [Acesso condicional no Azure Active Directory](/azure/active-directory/active-directory-conditional-access-azure-portal) para impor controles de acesso a aplicativos em seu ambiente de acordo com condições específicas. Por exemplo, a declaração da política pode assumir o seguinte formato: _Quando prestadores de serviço tentarem acessar nossos aplicativos de nuvem em redes não confiáveis, bloquear o acesso_. 

O [Azure AD Privileged Identity Management](/azure/active-directory/active-directory-privileged-identity-management-configure) pode ajudá-lo a gerenciar, controlar e monitorar os usuários e quais tipos de tarefas eles estão executando com seus privilégios de administrador. Essa é uma etapa importante para limitar quem em sua organização pode executar operações privilegiadas em aplicativos do Azure AD, Azure, Office 365 ou SaaS, bem como monitorar suas atividades.

### <a name="network"></a>Rede

Para proteger dados em trânsito, sempre use SSL/TLS durante a troca de dados em locais diferentes. Às vezes, você precisa isolar todo o canal de comunicação entre as infraestruturas local e de nuvem usando uma VPN (rede virtual privada) ou o [ExpressRoute](/azure/expressroute/). Para obter mais informações, consulte [Estendendo soluções de dados locais para a nuvem](../scenarios/hybrid-on-premises-and-cloud.md).

Use [NSGs](/azure/virtual-network/virtual-networks-nsg) (grupos de segurança de rede) para reduzir o número de vetores de ataque potenciais. Um grupo de segurança de rede contém uma lista de regras de segurança que permitem ou negam o tráfego de rede de entrada ou saída com base no endereço IP de origem ou destino, na porta e no protocolo. 

Use [pontos de extremidade de serviço da Rede Virtual](/azure/virtual-network/virtual-network-service-endpoints-overview) para proteger os recursos do SQL do Azure ou Armazenamento do Azure, de modo que apenas o tráfego da rede virtual possa acessar esses recursos.

As VMs de uma VNET (Rede Virtual) do Azure podem se comunicar com segurança com outras VNETs usando o [emparelhamento de rede virtual](/azure/virtual-network/virtual-network-peering-overview). O tráfego de rede entre redes virtuais emparelhadas é particular. O tráfego entre as redes virtuais é mantido na rede de backbone da Microsoft.

Para obter mais informações, consulte [Segurança de rede do Azure](/azure/security/azure-network-security)

### <a name="monitoring"></a>Monitoramento

A [Central de Segurança do Azure](/azure/security-center/security-center-intro) coleta, analisa e integra automaticamente os dados de log de seus recursos do Azure, da rede e das soluções de parceiros conectadas, como soluções de firewall, a fim de detectar ameaças reais e reduzir os falsos positivos. 

O [Log Analytics](/azure/log-analytics/log-analytics-overview) fornece acesso centralizado aos logs e ajuda você a analisar os dados e criar alertas personalizados.

A [Detecção de Ameaças do Banco de Dados SQL do Azure](/azure/sql-database/sql-database-threat-detection) detecta atividades anômalas, indicando tentativas incomuns e potencialmente prejudiciais de acessar ou explorar bancos de dados. Os agentes de segurança ou outros administradores designados podem receber uma notificação imediata sobre atividades suspeitas do banco de dados conforme elas ocorrem. Cada notificação fornece detalhes da atividade suspeita e recomenda como investigar e minimizar a ameaça.


