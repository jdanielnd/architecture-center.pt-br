---
title: Sobre o aplicativo Tailspin Surveys
description: Uma visão geral do aplicativo Tailspin Surveys.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: de2830b5c492e027c189a79e45ccc6634cab436b
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58247247"
---
# <a name="the-tailspin-scenario"></a><span data-ttu-id="d262b-103">O cenário da Tailspin</span><span class="sxs-lookup"><span data-stu-id="d262b-103">The Tailspin scenario</span></span>

<span data-ttu-id="d262b-104">[Código de exemplo do ![GitHub](../_images/github.png)][sample application]</span><span class="sxs-lookup"><span data-stu-id="d262b-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="d262b-105">A Tailspin é uma empresa fictícia que está desenvolvendo um aplicativo SaaS chamado Surveys.</span><span class="sxs-lookup"><span data-stu-id="d262b-105">Tailspin is a fictitious company that is developing a SaaS application named Surveys.</span></span> <span data-ttu-id="d262b-106">Esse aplicativo permite às organizações criar e publicar pesquisas online.</span><span class="sxs-lookup"><span data-stu-id="d262b-106">This application enables organizations to create and publish online surveys.</span></span>

* <span data-ttu-id="d262b-107">Uma organização pode se inscrever para obter o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="d262b-107">An organization can sign up for the application.</span></span>
* <span data-ttu-id="d262b-108">Depois que a organização estiver inscrita, os usuários podem entrar no aplicativo com suas credenciais organizacionais.</span><span class="sxs-lookup"><span data-stu-id="d262b-108">After the organization is signed up, users can sign into the application with their organizational credentials.</span></span>
* <span data-ttu-id="d262b-109">Os usuários podem criar, editar e publicar pesquisas.</span><span class="sxs-lookup"><span data-stu-id="d262b-109">Users can create, edit, and publish surveys.</span></span>

> [!NOTE]
> <span data-ttu-id="d262b-110">Para começar a usar o aplicativo, confira [Executar o aplicativo Surveys].</span><span class="sxs-lookup"><span data-stu-id="d262b-110">To get started with the application, see [Run the Surveys application].</span></span>

## <a name="users-can-create-edit-and-view-surveys"></a><span data-ttu-id="d262b-111">Os usuários podem criar, editar e publicar pesquisas</span><span class="sxs-lookup"><span data-stu-id="d262b-111">Users can create, edit, and view surveys</span></span>

<span data-ttu-id="d262b-112">Um usuário autenticado pode exibir todas as pesquisas que criou, ou com as quais tem direitos de colaborador, e criar novas pesquisas.</span><span class="sxs-lookup"><span data-stu-id="d262b-112">An authenticated user can view all the surveys that he or she has created or has contributor rights to, and create new surveys.</span></span> <span data-ttu-id="d262b-113">Observe que o usuário é conectado com sua identidade organizacional, `bob@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="d262b-113">Notice that the user is signed in with his organizational identity, `bob@contoso.com`.</span></span>

![Aplicativo de pesquisas](./images/surveys-screenshot.png)

<span data-ttu-id="d262b-115">Esta captura de tela mostra a página Editar Pesquisa:</span><span class="sxs-lookup"><span data-stu-id="d262b-115">This screenshot shows the Edit Survey page:</span></span>

![Editar pesquisa](./images/edit-survey.png)

<span data-ttu-id="d262b-117">Os usuários também podem exibir as pesquisas criadas por outros usuários no mesmo locatário.</span><span class="sxs-lookup"><span data-stu-id="d262b-117">Users can also view any surveys created by other users within the same tenant.</span></span>

![Pesquisas de locatário](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a><span data-ttu-id="d262b-119">Os proprietários das pesquisas podem convidar colaboradores</span><span class="sxs-lookup"><span data-stu-id="d262b-119">Survey owners can invite contributors</span></span>

<span data-ttu-id="d262b-120">Quando um usuário cria uma pesquisa, ele ou ela pode convidar outras pessoas para ser colaboradores da pesquisa.</span><span class="sxs-lookup"><span data-stu-id="d262b-120">When a user creates a survey, he or she can invite other people to be contributors on the survey.</span></span> <span data-ttu-id="d262b-121">Os colaboradores podem editar a pesquisa, mas não podem excluí-la ou publicá-la.</span><span class="sxs-lookup"><span data-stu-id="d262b-121">Contributors can edit the survey, but cannot delete or publish it.</span></span>

![Adicionar colaborador](./images/add-contributor.png)

<span data-ttu-id="d262b-123">Um usuário pode adicionar colaboradores de outros locatários, o que permite o compartilhamento de recursos entre locatários.</span><span class="sxs-lookup"><span data-stu-id="d262b-123">A user can add contributors from other tenants, which enables cross-tenant sharing of resources.</span></span> <span data-ttu-id="d262b-124">Nesta captura de tela, Bob (`bob@contoso.com`) está adicionando Alice (`alice@fabrikam.com`) como colaboradora para uma pesquisa criada por ele.</span><span class="sxs-lookup"><span data-stu-id="d262b-124">In this screenshot, Bob (`bob@contoso.com`) is adding Alice (`alice@fabrikam.com`) as a contributor to a survey that Bob created.</span></span>

<span data-ttu-id="d262b-125">Quando Beatriz fizer logon, ela verá a pesquisa listada em “Pesquisas com as quais posso contribuir”.</span><span class="sxs-lookup"><span data-stu-id="d262b-125">When Alice logs in, she sees the survey listed under "Surveys I can contribute to".</span></span>

![Colaborador da pesquisa](./images/contributor.png)

<span data-ttu-id="d262b-127">Observe que Beatriz entra em seu próprio locatário, não como uma convidada do locatário da Contoso.</span><span class="sxs-lookup"><span data-stu-id="d262b-127">Note that Alice signs into her own tenant, not as a guest of the Contoso tenant.</span></span> <span data-ttu-id="d262b-128">Alice tem permissões de colaboradora apenas para essa pesquisa &mdash; ela não pode exibir outros pesquisas do locatário Contoso.</span><span class="sxs-lookup"><span data-stu-id="d262b-128">Alice has contributor permissions only for that survey &mdash; she cannot view other surveys from the Contoso tenant.</span></span>

## <a name="architecture"></a><span data-ttu-id="d262b-129">Arquitetura</span><span class="sxs-lookup"><span data-stu-id="d262b-129">Architecture</span></span>

<span data-ttu-id="d262b-130">O aplicativo Surveys consiste em um front-end da Web e um back-end de API da Web.</span><span class="sxs-lookup"><span data-stu-id="d262b-130">The Surveys application consists of a web front end and a web API backend.</span></span> <span data-ttu-id="d262b-131">Ambos são implementados usando o [ASP.NET Core].</span><span class="sxs-lookup"><span data-stu-id="d262b-131">Both are implemented using [ASP.NET Core].</span></span>

<span data-ttu-id="d262b-132">O aplicativo Web usa o Azure AD (Azure Active Directory) para autenticar usuários.</span><span class="sxs-lookup"><span data-stu-id="d262b-132">The web application uses Azure Active Directory (Azure AD) to authenticate users.</span></span> <span data-ttu-id="d262b-133">Ele também chama o Azure AD para obter tokens de acesso do OAuth 2 para a API Web.</span><span class="sxs-lookup"><span data-stu-id="d262b-133">The web application also calls Azure AD to get OAuth 2 access tokens for the Web API.</span></span> <span data-ttu-id="d262b-134">Tokens de acesso são armazenados no Cache Redis do Azure.</span><span class="sxs-lookup"><span data-stu-id="d262b-134">Access tokens are cached in Azure Redis Cache.</span></span> <span data-ttu-id="d262b-135">O cache permite que várias instâncias compartilhem o mesmo cache de token (por exemplo, em um farm de servidores).</span><span class="sxs-lookup"><span data-stu-id="d262b-135">The cache enables multiple instances to share the same token cache (e.g., in a server farm).</span></span>

![Arquitetura](./images/architecture.png)

<span data-ttu-id="d262b-137">[**Avançar**][authentication]</span><span class="sxs-lookup"><span data-stu-id="d262b-137">[**Next**][authentication]</span></span>

<!-- links -->

[authentication]: authenticate.md

[Executar o aplicativo Surveys]: ./run-the-app.md
[Run the Surveys application]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
