---
title: Trabalhar com identidades baseadas em declarações em aplicativos multilocatários
description: Como usar declarações para autorização e validação do emissor.
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authenticate
pnp.series.next: signup
ms.openlocfilehash: ffaa6085dd9ca9ddec203e6661575e984b2e25e0
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113579"
---
# <a name="work-with-claims-based-identities"></a>Trabalhar com identidades baseadas em declarações

[Código de exemplo do ![GitHub](../_images/github.png)][sample application]

## <a name="claims-in-azure-ad"></a>Declarações no Azure AD

Quando um usuário faz logon, o Microsoft Azure AD envia um token de ID que contém um conjunto de declarações sobre o usuário. Uma declaração é simplesmente uma informação expressada como um par chave/valor. Por exemplo, `email`=`bob@contoso.com`.  Declarações têm um emissor &mdash; nesse caso, o Azure AD &mdash;, que é a entidade que autentica o usuário e cria as declarações. Você confia nas declarações porque confia no emissor. (Por outro lado, se você não confiar o emissor, não confiará nas declarações.)

Em um alto nível:

1. O usuário é autenticado.
2. O IDP envia um conjunto de declarações.
3. O aplicativo normaliza ou amplia as declarações (opcional).
4. O aplicativo usa as declarações para tomar decisões sobre a autorização.

No OpenID Connect, o conjunto de declarações que você obtém é controlado pelo [parâmetro de escopo] da solicitação de autenticação. No entanto, o Azure AD emite um conjunto limitado de declarações por meio do OpenID Connect; consulte [Tipos de declaração e token com suporte]. Se você quiser mais informações sobre o usuário, será preciso usar a API do Graph do Microsoft Azure AD.

Eis algumas declarações do AAD que normalmente são relevantes para um aplicativo:

| Tipo de declaração no token de ID | DESCRIÇÃO |
| --- | --- |
| aud |A pessoa para quem o token foi emitido. Essa será a ID de cliente do aplicativo. Em geral, você não deve se preocupar com essa declaração, pois o middleware a valida automaticamente. Exemplo: `"91464657-d17a-4327-91f3-2ed99386406f"` |
| groups |Uma lista de grupos do AAD dos quais o usuário é membro. Exemplo: `["93e8f556-8661-4955-87b6-890bc043c30f", "fc781505-18ef-4a31-a7d5-7d931d7b857e"]` |
| iss |O [emissor] do token OIDC. Exemplo: `https://sts.windows.net/b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4/` |
| Nome |O nome de exibição do usuário. Exemplo: `"Alice A."` |
| oid |O identificador do objeto para o usuário no AAD. Esse valor é o identificador imutável e não reutilizável do usuário. Use esse valor e não o email, como um identificador exclusivo para os usuários, pois endereços de email podem mudar. Se você usar a API do Graph do Microsoft Azure AD no seu aplicativo, a ID de objeto é o valor usado para consultar informações do perfil. Exemplo: `"59f9d2dc-995a-4ddf-915e-b3bb314a7fa4"` |
| roles |Uma lista de funções de aplicativo para o usuário.    Exemplo: `["SurveyCreator"]` |
| tid |ID do locatário. Esse valor é um identificador exclusivo do locatário no Azure AD. Exemplo: `"b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4"` |
| unique_name |Um nome de exibição do usuário que pode ser lido por humanos. Exemplo: `"alice@contoso.com"` |
| upn |Nome do usuário principal. Exemplo: `"alice@contoso.com"` |

Essa tabela lista os tipos de declaração que aparecem no token de ID. No ASP.NET Core, o middleware do OpenID Connect converte alguns dos tipos de declaração quando popula a coleção Declarações do usuário principal:

* oid > `http://schemas.microsoft.com/identity/claims/objectidentifier`
* tid > `http://schemas.microsoft.com/identity/claims/tenantid`
* unique_name > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`
* upn > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`

## <a name="claims-transformations"></a>Transformações de declarações

Durante o fluxo de autenticação, talvez você queira modificar as declarações obtidas do IDP. No ASP.NET Core, você pode executar a transformação de declarações dentro do evento **AuthenticationValidated** no middleware do OpenID Connect. (Confira [Eventos de autenticação].)

As declarações que você adicionar durante **AuthenticationValidated** serão armazenadas no cookie de autenticação de sessão. Elas não serão enviadas por push de volta ao Azure AD.

Veja alguns exemplos de transformação de declarações:

* **Normalização de declarações normalização**ou uniformizar as declarações entre os usuários. Isso é especialmente relevante se você estiver obtendo declarações de vários IDPs que podem usar tipos de declaração diferentes para informações semelhantes. Por exemplo, o Azure AD envia uma declaração "upn" que contém o email do usuário. Outros IDPs podem enviar uma declaração "email". O código abaixo converte a declaração "upn" em uma declaração "email":

  ```csharp
  var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
  if (!string.IsNullOrWhiteSpace(email))
  {
      identity.AddClaim(new Claim(ClaimTypes.Email, email));
  }
  ```

* Adicione **valores de declaração padrão** para declarações que não estão presentes &mdash; por exemplo, atribuir um usuário a uma função padrão. Em alguns casos, isso pode simplificar a lógica da autorização.
* Adicione **tipos de declaração personalizada** com informações específicas do aplicativo sobre o usuário. Por exemplo, você pode armazenar informações sobre o usuário em um banco de dados. Você poderia adicionar uma declaração personalizada com essas informações ao tíquete de autenticação. A declaração é armazenada em um cookie, por isso basta obtê-la no banco de dados uma vez por sessão de logon. Por outro lado, também é recomendável evitar a criação de cookies excessivamente grandes, portanto você precisa considerar o equilíbrio entre o tamanho do cookie e as pesquisas de banco de dados.

Depois que o fluxo de autenticação for concluído, as declarações estarão disponíveis em `HttpContext.User`. Nesse ponto, você deve tratá-los como uma coleção &mdash; somente leitura, por exemplo, usando-os para tomar decisões de autorização.

## <a name="issuer-validation"></a>Validação do emissor

No OpenID Connect, a declaração do emissor ("iss") identifica o IDP que emitiu o token de ID. Parte do fluxo de autenticação OIDC serve para verificar se a declaração do emissor corresponde ao emissor verdadeiro. O middleware OIDC lida com isso para você.

No Azure AD, o valor de emissor é exclusivo por locatário do AD (`https://sts.windows.net/<tenantID>`). Portanto, o aplicativo deve fazer uma verificação adicional para ter certeza de que o emissor representa um locatário que tem permissão para entrar no aplicativo.

Para um aplicativo de locatário único, você pode apenas verificar se o emissor é seu próprio locatário. Na verdade, o middleware OIDC faz isso automaticamente por padrão. Em um aplicativo multilocatário, você precisa permitir vários emissores que correspondem a locatários diferentes. Veja uma abordagem geral que pode ser usada:

* Nas opções de middleware OIDC, defina **ValidateIssuer** como false. Isso desliga a verificação automática.
* Quando um locatário se inscrever, armazene o locatário e o emissor em seu banco de dados do usuário.
* Sempre que um usuário fizer logon, pesquise o emissor no banco de dados. Se o emissor não for encontrado, significa que o locatário ainda não se inscreveu. Você pode redirecioná-lo para uma página de inscrição.
* Você também pode colocar determinados locatários na lista negra, por exemplo, clientes que não pagaram suas assinaturas.

Para ver uma discussão mais detalhada, confira [Inscrição e integração de locatário em aplicativos multilocatário][signup].

## <a name="using-claims-for-authorization"></a>Usando declarações para autorização

Com declarações, a identidade de um usuário não é uma entidade monolítica. Por exemplo, um usuário pode ter um endereço de email, número de telefone, aniversário, sexo, etc. O IDP do usuário pode armazenar todas essas informações. Porém, ao autenticar o usuário, você normalmente obtém um subconjunto dessas declarações. Neste modelo, a identidade do usuário é simplesmente um pacote de declarações. Ao tomar decisões de autorização sobre um usuário, você procura por determinados conjuntos de declarações. Em outras palavras, a pergunta “O usuário X pode executar a ação Y?” se torna “O usuário X tem a declaração Z?”.

Veja alguns padrões básicos para verificar declarações.

* Para verificar se o usuário tem uma declaração específica com um valor específico:

   ```csharp
   if (User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
   ```

   Esse código verifica se o usuário tem uma declaração Função com o valor "Admin". Ele aborda corretamente o caso de usuários que não tenham a declaração Função ou tenham várias declarações Função.
  
   A classe **ClaimTypes** define constantes para tipos de declaração comuns. No entanto, você pode usar qualquer valor da cadeia de caracteres para o tipo de declaração.
* Para obter um único valor de um tipo de declaração, quando você espera que haja no máximo um valor:

  ```csharp
  string email = User.FindFirst(ClaimTypes.Email)?.Value;
  ```

* Para obter todos os valores de um tipo de declaração:

  ```csharp
  IEnumerable<Claim> groups = User.FindAll("groups");
  ```

Para obter mais informações, consulte [Autorização baseada em recursos e em funções em aplicativos multilocatários][authorization].

[**Avançar**][signup]

<!-- links -->

[parâmetro de escopo]: https://nat.sakimura.org/2012/01/26/scopes-and-claims-in-openid-connect/
[Tipos de declaração e token com suporte]: /azure/active-directory/active-directory-token-and-claims/
[emissor]: https://openid.net/specs/openid-connect-core-1_0.html#IDToken
[Eventos de autenticação]: authenticate.md#authentication-events
[signup]: signup.md
[Claims-Based Authorization]: /aspnet/core/security/authorization/claims
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[authorization]: authorize.md
