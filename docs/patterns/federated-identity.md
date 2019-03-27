---
title: Padrão de identidade federada
titleSuffix: Cloud Design Patterns
description: Delegar autenticação a um provedor de identidade externa.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f4e8d2dff1c653cf2b8ef0109f1cc3027e5428b9
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242317"
---
# <a name="federated-identity-pattern"></a>Padrão de identidade federada

[!INCLUDE [header](../_includes/header.md)]

Autenticação de delegado a um provedor de identidade externo. Isso pode simplificar o desenvolvimento, minimizar a necessidade de administração de usuário e melhorar a experiência do usuário do aplicativo.

## <a name="context-and-problem"></a>Contexto e problema

Normalmente, os usuários precisam trabalhar com vários aplicativos fornecidos e hospedados por diferentes organizações com as quais eles mantém uma relação comercial. Esses usuários podem precisar usar credenciais específicas (e diferentes) para cada um. Isso pode:

- **Causar uma experiência de usuário não contígua**. Os usuários muitas vezes esquecem as credenciais de logon quando usam várias delas.

- **Expor vulnerabilidades de segurança**. Quando um usuário sai da empresa, a conta deve ser desprovisionada imediatamente. É fácil ignorar isso em grandes organizações.

- **Complicações no gerenciamento de usuários**. Os administradores devem gerenciar as credenciais para todos os usuários e realizar tarefas adicionais, como fornecer lembretes de senha.

Os usuários normalmente preferem usar as mesmas credenciais para todos esses aplicativos.

## <a name="solution"></a>Solução

Implemente um mecanismo de autenticação que pode usar identidade federada. Separe a autenticação do usuário do código de aplicativo e delegue a autenticação a um provedor de identidade confiável. Isso pode simplificar o desenvolvimento e permitir que os usuários se autentiquem usando uma variedade maior de IdP (provedores de identidade), minimizando a sobrecarga administrativa. Ele também permite separar claramente a autenticação da autorização.

Os provedores de identidade confiável incluem diretórios corporativos, serviços de federação local, outros STS (serviços de token de segurança) fornecidos por parceiros de negócios ou provedores de identidade sociais que podem autenticar os usuários que têm, por exemplo, uma conta da Microsoft, Google, Yahoo! ou Facebook.

A figura ilustra o Padrão de identidade federada quando um aplicativo cliente precisa acessar um serviço que requer autenticação. A autenticação é realizada por um IdP que funciona junto com um STS. O IdP emite tokens de segurança que fornecem informações sobre o usuário autenticado. Essas informações, chamadas de declarações, incluem a identidade do usuário e também podem incluir outras informações, como associação de função e os direitos de acesso mais granulares.

![Uma visão geral da autenticação federada](./_images/federated-identity-overview.png)

Esse modelo geralmente é chamado de controle de acesso baseado em declarações. Os aplicativos e serviços autorizam o acesso aos recursos e funcionalidades com base nas declarações contidas no token. O serviço que exige autenticação deve confiar no IdP. O aplicativo cliente contata o IdP que executa a autenticação. Se a autenticação for bem-sucedida, o IdP retornará um token que contém as declarações que identificam o usuário para o STS (observe que o IdP e o STS podem ser o mesmo serviço). O STS pode transformar e aumentar as declarações no token com base nas regras predefinidas, antes de retorná-lo para o cliente. O aplicativo cliente pode, então, transmitir esse token para o serviço como uma prova de sua identidade.

> Podem existir STSs adicionais na cadeia de confiança. Por exemplo, no cenário descrito posteriormente, um STS local confia em outro STS que é responsável por acessar um provedor de identidade para autenticar o usuário. Essa abordagem é comum em cenários empresariais em que há um local STS e um diretório.

A autenticação federada fornece uma solução baseada em padrões para o problema de confiar em identidades entre diferentes domínios e pode dar suporte a logon único. Ela está se tornando mais comum em todos os tipos de aplicativos, especialmente aplicativos hospedados em nuvem, pois ele dá suporte a logon único sem a necessidade de uma conexão de rede direta com provedores de identidade. O usuário não precisa digitar as credenciais para cada aplicativo. Isso aumenta a segurança, pois impede a criação de credenciais necessárias para acessar vários aplicativos diferentes e também oculta as credenciais do usuário de todos, exceto do provedor de identidade original. Os aplicativos veem apenas as informações de identidade autenticada contidas no token.

A identidade federada também traz a grande vantagem de que o gerenciamento da identidade e das credenciais é responsabilidade do provedor de identidade. O aplicativo ou serviço não precisa fornecer recursos de gerenciamento de identidade. Além disso, em cenários corporativos, o diretório corporativo não precisa conhecer o usuário se confiar no provedor de identidade. Isso remove toda a sobrecarga administrativa de gerenciar a identidade do usuário dentro do diretório.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao projetar aplicativos que implementam a autenticação federada:

- A autenticação pode ser um ponto único de falha. Se você implantar seu aplicativo em vários datacenters, considere implantar seu mecanismo de gerenciamento de identidade para os mesmos datacenters para manter a disponibilidade e a confiabilidade do aplicativo.

- As ferramentas de autenticação permitem configurar o controle de acesso baseado em declarações de função contidas no token de autenticação. Isso geralmente é conhecido como RBAC (controle de acesso baseado em função) e pode proporcionar um nível mais granular de controle de acesso aos recursos e funcionalidades.

- Ao contrário de um diretório corporativo, a autenticação baseada em declarações usando provedores de identidade social geralmente não fornecem informações sobre o usuário autenticado além de um endereço de email e, talvez, um nome. Alguns provedores de identidade social, como uma conta da Microsoft, fornecem apenas um identificador exclusivo. O aplicativo geralmente precisa manter algumas informações sobre os usuários registrados e corresponder tais informações com o identificador contido nas declarações no token. Normalmente isso é feito por meio do registro quando o usuário acessa o aplicativo pela primeira vez e as informações são injetadas no token como declarações adicionais depois de cada autenticação.

- Se houver mais de um provedor de identidade configurado para o STS, ele deverá detectar para qual provedor de identidade o usuário deve ser redirecionado para autenticação. Esse processo é chamado de descoberta de realm inicial. O STS pode fazer isso automaticamente com base em um endereço de email ou nome de usuário fornecidos pelo usuário, um subdomínio do aplicativo que o usuário está acessando, o escopo do endereço IP do usuário ou no conteúdo de um cookie armazenado no navegador do usuário. Por exemplo, se o usuário inseriu um endereço de email no domínio da Microsoft, como user@live.com, o STS redirecionará o usuário para a página de entrada da conta da Microsoft. Em visitas posteriores, o STS poderia usar um cookie para indicar que a última entrada ocorreu com uma conta da Microsoft. Se a descoberta automática não puder determinar o realm inicial, o STS exibirá uma página de descoberta de realm inicial que lista os provedores de identidade confiável e o usuário precisará selecionar aquela que ele deseja usar.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Esse padrão é útil em cenários como:

- **Logon único na empresa**. Nesse cenário, você precisa autenticar os funcionários em aplicativos corporativos que são hospedados na nuvem fora do limite de segurança corporativo, sem exigir que eles entrem toda vez que visitarem um aplicativo. A experiência do usuário é igual a usar aplicativos locais em que estão autenticados ao entrar em uma rede corporativa e nela eles têm acesso a todos os aplicativos relevantes sem precisar entrar novamente.

- **Identidade federada com vários parceiros**. Nesse cenário, você precisa autenticar os funcionários corporativos e parceiros de negócios que não têm contas no diretório corporativo. Isso é comum em aplicativos entre empresas, aplicativos que se integram com serviços de terceiros e pontos em que empresas com diferentes sistemas de TI mesclam ou compartilham recursos.

- **Identidade federada em aplicativos SaaS**. Neste cenário, os fornecedores independentes oferecem um serviço pronto para usar para vários clientes ou locatários. Cada locatário é autenticado usando um provedor de identidade adequado. Por exemplo, os usuários empresariais usarão suas credenciais corporativas, enquanto os consumidores e clientes do locatário usarão as credenciais de identidade social.

Esse padrão pode não ser útil nas seguintes situações:

- Todos os usuários do aplicativo podem ser autenticados por um provedor de identidade e não há nenhum requisito para autenticar-se usando outro provedor de identidade. Isso é comum em aplicativos de negócios que usam um diretório corporativo (acessível de dentro do aplicativo) para autenticação, usando uma VPN ou (em um cenário hospedado em nuvem) por meio de uma conexão de rede virtual entre o diretório local e o aplicativo.

- O aplicativo foi criado originalmente usando um mecanismo de autenticação diferente, talvez com repositórios de usuário personalizados ou não tem a capacidade de lidar com os padrões de negociação usados pelas tecnologias baseadas em declarações. Adaptar a autenticação e o controle de acesso baseados em declarações para aplicativos existentes pode ser muito complexo e provavelmente não econômico.

## <a name="example"></a>Exemplo

Uma organização hospeda um aplicativo SaaS (software como serviço) de vários locatários no Microsoft Azure. O aplicativo inclui um site que os locatários podem usar para gerenciar o aplicativo para seus próprios usuários. O aplicativo permite que os locatários acessem o site usando uma identidade federada que é gerada pelo Serviços de Federação do Active Directory (AD FS) quando um usuário é autenticado pelo Active Directory da própria organização.

![Como os usuários em um assinante de grandes empresas acessam o aplicativo](./_images/federated-identity-multitenat.png)

A figura mostra como os locatários são autenticados com seu próprio provedor de identidade (etapa 1), nesse caso o ADFS. Depois de autenticar com êxito um locatário, o ADFS emite um token. O navegador do cliente encaminha esse token para o provedor de federação do aplicativo SaaS, que confia nos tokens emitidos pelo AD FS do locatário, a fim de retornar um token válido para o provedor de federação de SaaS (etapa 2). Se necessário, o provedor de federação de SaaS executa uma transformação sobre as declarações no token para declarações que o aplicativo reconhece (etapa 3) antes de retornar o novo token para o navegador do cliente. O aplicativo confia nos tokens emitidos pelo provedor de federação de SaaS e usa as declarações no token para aplicar as regras de autorização (etapa 4).

Os locatários não precisam se lembrar das credenciais separadas para acessar o aplicativo e um administrador de empresa do locatário pode configurar seu no próprio ADFS a lista de usuários que podem acessar o aplicativo.

## <a name="related-guidance"></a>Diretrizes relacionadas

- [Microsoft Azure Active Directory](https://azure.microsoft.com/services/active-directory/)
- [Active Directory Domain Services](https://msdn.microsoft.com/library/bb897402.aspx)
- [Serviços de Federação do Active Directory (AD FS)](https://msdn.microsoft.com/library/bb897402.aspx)
- [Gerenciamento de identidades para aplicativos multilocatários no Microsoft Azure](/azure/architecture/multitenant-identity)
- [Aplicativos multilocatários no Azure](/azure/dotnet-develop-multitenant-applications)
