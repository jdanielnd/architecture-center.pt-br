---
title: Use o Key Vault para proteger segredos do aplicativo
description: Como usar o serviço Key Vault para armazenar segredos do aplicativo
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
ms.openlocfilehash: 45d1564c255f2450f68c5e92ebe0d7de0c40ae31
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="use-azure-key-vault-to-protect-application-secrets"></a><span data-ttu-id="e9d22-103">Usar o Azure Key Vault para proteger os segredos do aplicativo</span><span class="sxs-lookup"><span data-stu-id="e9d22-103">Use Azure Key Vault to protect application secrets</span></span>

<span data-ttu-id="e9d22-104">[Código de exemplo do ![GitHub](../_images/github.png)][sample application]</span><span class="sxs-lookup"><span data-stu-id="e9d22-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="e9d22-105">É comum ter configurações de aplicativo confidenciais e que devem ser protegidas, por exemplo:</span><span class="sxs-lookup"><span data-stu-id="e9d22-105">It's common to have application settings that are sensitive and must be protected, such as:</span></span>

* <span data-ttu-id="e9d22-106">Cadeias de conexão de banco de dados</span><span class="sxs-lookup"><span data-stu-id="e9d22-106">Database connection strings</span></span>
* <span data-ttu-id="e9d22-107">Senhas</span><span class="sxs-lookup"><span data-stu-id="e9d22-107">Passwords</span></span>
* <span data-ttu-id="e9d22-108">Chaves de criptografia</span><span class="sxs-lookup"><span data-stu-id="e9d22-108">Cryptographic keys</span></span>

<span data-ttu-id="e9d22-109">Como prática de segurança recomendada, nunca armazene esses segredos no controle do código-fonte.</span><span class="sxs-lookup"><span data-stu-id="e9d22-109">As a security best practice, you should never store these secrets in source control.</span></span> <span data-ttu-id="e9d22-110">É muito fácil vazar &mdash;, mesmo se o repositório de código-fonte for privado.</span><span class="sxs-lookup"><span data-stu-id="e9d22-110">It's too easy for them to leak &mdash; even if your source code repository is private.</span></span> <span data-ttu-id="e9d22-111">E não se trata apenas de guardar os segredos do público geral.</span><span class="sxs-lookup"><span data-stu-id="e9d22-111">And it's not just about keeping secrets from the general public.</span></span> <span data-ttu-id="e9d22-112">Em projetos maiores, talvez você queira restringir quais desenvolvedores e operadores podem acessar os segredos de produção.</span><span class="sxs-lookup"><span data-stu-id="e9d22-112">On larger projects, you might want to restrict which developers and operators can access the production secrets.</span></span> <span data-ttu-id="e9d22-113">(As configurações para ambientes de teste ou de desenvolvimento são diferentes).</span><span class="sxs-lookup"><span data-stu-id="e9d22-113">(Settings for test or development environments are different.)</span></span>

<span data-ttu-id="e9d22-114">Uma opção mais segura é armazenar esses segredos no [Azure Key Vault][KeyVault].</span><span class="sxs-lookup"><span data-stu-id="e9d22-114">A more secure option is to store these secrets in [Azure Key Vault][KeyVault].</span></span> <span data-ttu-id="e9d22-115">O Cofre da Chave é um serviço hospedado na nuvem para gerenciamento de chaves de criptografia e de outros segredos.</span><span class="sxs-lookup"><span data-stu-id="e9d22-115">Key Vault is a cloud-hosted service for managing cryptographic keys and other secrets.</span></span> <span data-ttu-id="e9d22-116">Este artigo mostra como usar o Cofre da Chave para armazenar as definições de configuração para seu aplicativo.</span><span class="sxs-lookup"><span data-stu-id="e9d22-116">This article shows how to use Key Vault to store configuration settings for you app.</span></span>

<span data-ttu-id="e9d22-117">No aplicativo [Tailspin Surveys][Surveys] aplicativo, as configurações a seguir são secretas:</span><span class="sxs-lookup"><span data-stu-id="e9d22-117">In the [Tailspin Surveys][Surveys] application, the following settings are secret:</span></span>

* <span data-ttu-id="e9d22-118">A cadeia de conexão do banco de dados.</span><span class="sxs-lookup"><span data-stu-id="e9d22-118">The database connection string.</span></span>
* <span data-ttu-id="e9d22-119">A cadeia de conexão Redis.</span><span class="sxs-lookup"><span data-stu-id="e9d22-119">The Redis connection string.</span></span>
* <span data-ttu-id="e9d22-120">O segredo do cliente para o aplicativo Web.</span><span class="sxs-lookup"><span data-stu-id="e9d22-120">The client secret for the web application.</span></span>

<span data-ttu-id="e9d22-121">O aplicativo Surveys carrega configurações dos seguintes locais:</span><span class="sxs-lookup"><span data-stu-id="e9d22-121">The Surveys application loads configuration settings from the following places:</span></span>

* <span data-ttu-id="e9d22-122">O arquivo appsettings.json</span><span class="sxs-lookup"><span data-stu-id="e9d22-122">The appsettings.json file</span></span>
* <span data-ttu-id="e9d22-123">O [repositório de segredos do usuário][user-secrets] (apenas ambiente de desenvolvimento; para teste)</span><span class="sxs-lookup"><span data-stu-id="e9d22-123">The [user secrets store][user-secrets] (development environment only; for testing)</span></span>
* <span data-ttu-id="e9d22-124">O ambiente de hospedagem (configurações do aplicativo em aplicativos Web do Azure)</span><span class="sxs-lookup"><span data-stu-id="e9d22-124">The hosting environment (app settings in Azure web apps)</span></span>
* <span data-ttu-id="e9d22-125">Key Vault (quando habilitado)</span><span class="sxs-lookup"><span data-stu-id="e9d22-125">Key Vault (when enabled)</span></span>

<span data-ttu-id="e9d22-126">Cada uma delas substitui a anterior, portanto, as configurações armazenadas no Cofre da Chave têm precedência.</span><span class="sxs-lookup"><span data-stu-id="e9d22-126">Each of these overrides the previous one, so any settings stored in Key Vault take precedence.</span></span>

> [!NOTE]
> <span data-ttu-id="e9d22-127">Por padrão, o provedor de configuração do Cofre da Chave está desabilitado.</span><span class="sxs-lookup"><span data-stu-id="e9d22-127">By default, the Key Vault configuration provider is disabled.</span></span> <span data-ttu-id="e9d22-128">Ele não é necessário para executar o aplicativo localmente.</span><span class="sxs-lookup"><span data-stu-id="e9d22-128">It's not needed for running the application locally.</span></span> <span data-ttu-id="e9d22-129">Você poderia habilitá-lo em uma implantação de produção.</span><span class="sxs-lookup"><span data-stu-id="e9d22-129">You would enable it in a production deployment.</span></span>

<span data-ttu-id="e9d22-130">Na inicialização, o aplicativo lê as configurações de cada provedor de configuração registrado, e as usa para preencher um objeto de opções fortemente tipado.</span><span class="sxs-lookup"><span data-stu-id="e9d22-130">At startup, the application reads settings from every registered configuration provider, and uses them to populate a strongly typed options object.</span></span> <span data-ttu-id="e9d22-131">Para obter mais informações, consulte [Como usar opções e objetos de configuração][options].</span><span class="sxs-lookup"><span data-stu-id="e9d22-131">For more information, see [Using Options and configuration objects][options].</span></span>

## <a name="setting-up-key-vault-in-the-surveys-app"></a><span data-ttu-id="e9d22-132">Configuração do Cofre da Chave no aplicativo Surveys</span><span class="sxs-lookup"><span data-stu-id="e9d22-132">Setting up Key Vault in the Surveys app</span></span>
<span data-ttu-id="e9d22-133">Pré-requisitos:</span><span class="sxs-lookup"><span data-stu-id="e9d22-133">Prerequisites:</span></span>

* <span data-ttu-id="e9d22-134">Instale os [Cmdlets do Azure Resource Manager][azure-rm-cmdlets].</span><span class="sxs-lookup"><span data-stu-id="e9d22-134">Install the [Azure Resource Manager Cmdlets][azure-rm-cmdlets].</span></span>
* <span data-ttu-id="e9d22-135">Configurar o aplicativo Surveys conforme descrito em [Executar o aplicativo Surveys][readme].</span><span class="sxs-lookup"><span data-stu-id="e9d22-135">Configure the Surveys application as described in [Run the Surveys application][readme].</span></span>

<span data-ttu-id="e9d22-136">Etapas de alto nível:</span><span class="sxs-lookup"><span data-stu-id="e9d22-136">High-level steps:</span></span>

1. <span data-ttu-id="e9d22-137">Configure um usuário de administração no locatário.</span><span class="sxs-lookup"><span data-stu-id="e9d22-137">Set up an admin user in the tenant.</span></span>
2. <span data-ttu-id="e9d22-138">Configure um certificado de cliente.</span><span class="sxs-lookup"><span data-stu-id="e9d22-138">Set up a client certificate.</span></span>
3. <span data-ttu-id="e9d22-139">Crie um cofre da chave.</span><span class="sxs-lookup"><span data-stu-id="e9d22-139">Create a key vault.</span></span>
4. <span data-ttu-id="e9d22-140">Adicione definições de configuração para o cofre da chave.</span><span class="sxs-lookup"><span data-stu-id="e9d22-140">Add configuration settings to your key vault.</span></span>
5. <span data-ttu-id="e9d22-141">Remova o comentário do código que habilita o cofre da chave.</span><span class="sxs-lookup"><span data-stu-id="e9d22-141">Uncomment the code that enables key vault.</span></span>
6. <span data-ttu-id="e9d22-142">Atualize os segredos do usuário do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="e9d22-142">Update the application's user secrets.</span></span>

### <a name="set-up-an-admin-user"></a><span data-ttu-id="e9d22-143">Configurar um usuário administrador</span><span class="sxs-lookup"><span data-stu-id="e9d22-143">Set up an admin user</span></span>
> [!NOTE]
> <span data-ttu-id="e9d22-144">Para criar um cofre da chave, você deve usar uma conta que possa gerenciar sua assinatura do Azure.</span><span class="sxs-lookup"><span data-stu-id="e9d22-144">To create a key vault, you must use an account which can manage your Azure subscription.</span></span> <span data-ttu-id="e9d22-145">Além disso, qualquer aplicativo que você autorizar para ler do cofre de chaves deve ser registrado no mesmo locatário que aquela conta.</span><span class="sxs-lookup"><span data-stu-id="e9d22-145">Also, any application that you authorize to read from the key vault must be registered in the same tenant as that account.</span></span>
> 
> 

<span data-ttu-id="e9d22-146">Nesta etapa, você confirmará se pode criar um cofre da chave enquanto estiver conectado como um usuário no locatário no qual o aplicativo Surveys está registrado.</span><span class="sxs-lookup"><span data-stu-id="e9d22-146">In this step, you will make sure that you can create a key vault while signed in as a user from the tenant where the Surveys app is registered.</span></span>

<span data-ttu-id="e9d22-147">Crie um usuário administrador no locatário do Azure AD no qual o aplicativo Surveys está registrado.</span><span class="sxs-lookup"><span data-stu-id="e9d22-147">Create an administrator user within the Azure AD tenant where the Surveys application is registered.</span></span>

1. <span data-ttu-id="e9d22-148">Faça logon no [portal do Azure][azure-portal].</span><span class="sxs-lookup"><span data-stu-id="e9d22-148">Log into the [Azure portal][azure-portal].</span></span>
2. <span data-ttu-id="e9d22-149">Selecione o locatário do Azure AD onde seu aplicativo está registrado.</span><span class="sxs-lookup"><span data-stu-id="e9d22-149">Select the Azure AD tenant where your application is registered.</span></span>
3. <span data-ttu-id="e9d22-150">Clique em **Mais serviço** > **SEGURANÇA + IDENTIDADE** > **Azure Active Directory** > **Usuários e grupos** > **Todos os usuários**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-150">Click **More service** > **SECURITY + IDENTITY** > **Azure Active Directory** > **User and groups** > **All users**.</span></span>
4. <span data-ttu-id="e9d22-151">Na parte superior do portal, clique em **Novo usuário**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-151">At the top of the portal, click **New user**.</span></span>
5. <span data-ttu-id="e9d22-152">Preencha os campos e atribua o usuário à função do diretório **Administrador Global**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-152">Fill in the fields and assign the user to the **Global administrator** directory role.</span></span>
6. <span data-ttu-id="e9d22-153">Clique em **Criar**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-153">Click **Create**.</span></span>

![Usuário administrador global](./images/running-the-app/global-admin-user.png)

<span data-ttu-id="e9d22-155">Agora atribua esse usuário como o proprietário da assinatura.</span><span class="sxs-lookup"><span data-stu-id="e9d22-155">Now assign this user as the subscription owner.</span></span>

1. <span data-ttu-id="e9d22-156">No menu Hub, selecione **Assinaturas**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-156">On the Hub menu, select **Subscriptions**.</span></span>

    ![](./images/running-the-app/subscriptions.png)

2. <span data-ttu-id="e9d22-157">Selecione a assinatura que você deseja que o administrador acesse.</span><span class="sxs-lookup"><span data-stu-id="e9d22-157">Select the subscription that you want the administrator to access.</span></span>
3. <span data-ttu-id="e9d22-158">Na folha da assinatura, selecione **Controle de acesso (IAM)**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-158">In the subscription blade, select **Access control (IAM)**.</span></span>
4. <span data-ttu-id="e9d22-159">Clique em **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-159">Click **Add**.</span></span>
4. <span data-ttu-id="e9d22-160">Em **Função**, selecione **Proprietário**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-160">Under **Role**, select **Owner**.</span></span>
5. <span data-ttu-id="e9d22-161">Digite o endereço de email do usuário que você deseja adicionar como proprietário.</span><span class="sxs-lookup"><span data-stu-id="e9d22-161">Type the email address of the user you want to add as owner.</span></span>
6. <span data-ttu-id="e9d22-162">Selecione o usuário e clique em **Salvar**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-162">Select the user and click **Save**.</span></span>

### <a name="set-up-a-client-certificate"></a><span data-ttu-id="e9d22-163">Configurar um certificado de cliente</span><span class="sxs-lookup"><span data-stu-id="e9d22-163">Set up a client certificate</span></span>
1. <span data-ttu-id="e9d22-164">Execute o script do PowerShell [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="e9d22-164">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    <span data-ttu-id="e9d22-165">Para o parâmetro `Subject` , digite qualquer nome, como "surveysapp".</span><span class="sxs-lookup"><span data-stu-id="e9d22-165">For the `Subject` parameter, enter any name, such as "surveysapp".</span></span> <span data-ttu-id="e9d22-166">O script gera um certificado autoassinado e o armazena no repositório de certificados "Usuário Atual/Pessoal".</span><span class="sxs-lookup"><span data-stu-id="e9d22-166">The script generates a self-signed certificate and stores it in the "Current User/Personal" certificate store.</span></span> <span data-ttu-id="e9d22-167">A saída do script é um fragmento JSON.</span><span class="sxs-lookup"><span data-stu-id="e9d22-167">The output from the script is a JSON fragment.</span></span> <span data-ttu-id="e9d22-168">Copie esse valor.</span><span class="sxs-lookup"><span data-stu-id="e9d22-168">Copy this value.</span></span>

2. <span data-ttu-id="e9d22-169">No [portal do Azure][azure-portal], alterne para o diretório no qual o aplicativo Surveys é registrado selecionando a sua conta no canto superior direito do portal.</span><span class="sxs-lookup"><span data-stu-id="e9d22-169">In the [Azure portal][azure-portal], switch to the directory where the Surveys application is registered, by selecting your account in the top right corner of the portal.</span></span>

3. <span data-ttu-id="e9d22-170">Selecione **Azure Active Directory** > **Registros de Aplicativo** > Surveys</span><span class="sxs-lookup"><span data-stu-id="e9d22-170">Select **Azure Active Directory** > **App Registrations** > Surveys</span></span>

4.  <span data-ttu-id="e9d22-171">Clique em **Manifesto** e então em **Editar**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-171">Click **Manifest** and then **Edit**.</span></span>

5.  <span data-ttu-id="e9d22-172">Cole a saída do script na propriedade `keyCredentials` .</span><span class="sxs-lookup"><span data-stu-id="e9d22-172">Paste the output from the script into the `keyCredentials` property.</span></span> <span data-ttu-id="e9d22-173">O arquivo deve ser semelhante ao seguinte:</span><span class="sxs-lookup"><span data-stu-id="e9d22-173">It should look similar to the following:</span></span>
        
    ```json
    "keyCredentials": [
        {
        "type": "AsymmetricX509Cert",
        "usage": "Verify",
        "keyId": "29d4f7db-0539-455e-b708-....",
        "customKeyIdentifier": "ZEPpP/+KJe2fVDBNaPNOTDoJMac=",
        "value": "MIIDAjCCAeqgAwIBAgIQFxeRiU59eL.....
        }
    ],
    ```          

6. <span data-ttu-id="e9d22-174">Clique em **Salvar**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-174">Click **Save**.</span></span>  

7. <span data-ttu-id="e9d22-175">Repita as etapas 3 a 6 para adicionar o mesmo fragmento JSON ao manifesto do aplicativo da API Web (Surveys.WebAPI).</span><span class="sxs-lookup"><span data-stu-id="e9d22-175">Repeat steps 3-6 to add the same JSON fragment to the application manifest of the web API (Surveys.WebAPI).</span></span>

8. <span data-ttu-id="e9d22-176">Na janela do PowerShell, execute o seguinte comando para obter a impressão digital do certificado.</span><span class="sxs-lookup"><span data-stu-id="e9d22-176">From the PowerShell window, run the following command to get the thumbprint of the certificate.</span></span>
   
    ```
    certutil -store -user my [subject]
    ```
    
    <span data-ttu-id="e9d22-177">Para `[subject]`, use o valor especificado para o Assunto no script do PowerShell.</span><span class="sxs-lookup"><span data-stu-id="e9d22-177">For `[subject]`, use the value that you specified for Subject in the PowerShell script.</span></span> <span data-ttu-id="e9d22-178">A impressão digital é listada em "Cert Hash(sha1)".</span><span class="sxs-lookup"><span data-stu-id="e9d22-178">The thumbprint is listed under "Cert Hash(sha1)".</span></span> <span data-ttu-id="e9d22-179">Copie esse valor.</span><span class="sxs-lookup"><span data-stu-id="e9d22-179">Copy this value.</span></span> <span data-ttu-id="e9d22-180">Posteriormente, você usará a impressão digital.</span><span class="sxs-lookup"><span data-stu-id="e9d22-180">You will use the thumbprint later.</span></span>

### <a name="create-a-key-vault"></a><span data-ttu-id="e9d22-181">Criar um cofre de chave</span><span class="sxs-lookup"><span data-stu-id="e9d22-181">Create a key vault</span></span>
1. <span data-ttu-id="e9d22-182">Execute o script do PowerShell [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="e9d22-182">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```
   
    <span data-ttu-id="e9d22-183">Quando receber a solicitação pelas credenciais, entre como o usuário do Azure AD que você criou anteriormente.</span><span class="sxs-lookup"><span data-stu-id="e9d22-183">When prompted for credentials, sign in as the Azure AD user that you created earlier.</span></span> <span data-ttu-id="e9d22-184">O script cria um novo grupo de recursos e um novo cofre da chave dentro desse grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="e9d22-184">The script creates a new resource group, and a new key vault within that resource group.</span></span> 
   
2. <span data-ttu-id="e9d22-185">Execute SetupKeyVault.ps outra vez da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="e9d22-185">Run SetupKeyVault.ps again as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<Surveys app id>>", "<<Surveys.WebAPI app ID>>")
    ```
   
    <span data-ttu-id="e9d22-186">Defina os seguintes valores de parâmetro:</span><span class="sxs-lookup"><span data-stu-id="e9d22-186">Set the following parameter values:</span></span>
   
       * <span data-ttu-id="e9d22-187">nome do cofre da chave = o nome que você deu ao cofre da chave na etapa anterior.</span><span class="sxs-lookup"><span data-stu-id="e9d22-187">key vault name = The name that you gave the key vault in the previous step.</span></span>
       * <span data-ttu-id="e9d22-188">ID do aplicativo Surveys = a ID do aplicativo Web Surveys.</span><span class="sxs-lookup"><span data-stu-id="e9d22-188">Surveys app ID = The application ID for the Surveys web application.</span></span>
       * <span data-ttu-id="e9d22-189">ID do aplicativo Surveys.WebApi = a ID do aplicativo Surveys.WebAPI.</span><span class="sxs-lookup"><span data-stu-id="e9d22-189">Surveys.WebApi app ID = The application ID for the Surveys.WebAPI application.</span></span>
         
    <span data-ttu-id="e9d22-190">Exemplo:</span><span class="sxs-lookup"><span data-stu-id="e9d22-190">Example:</span></span>
     
    ```
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
    ```
    
    <span data-ttu-id="e9d22-191">Esse script autoriza o aplicativo Web e a API Web a recuperar os segredos de seu cofre da chave.</span><span class="sxs-lookup"><span data-stu-id="e9d22-191">This script authorizes the web app and web API to retrieve secrets from your key vault.</span></span> <span data-ttu-id="e9d22-192">Consulte [Introdução ao Azure Key Vault](/azure/key-vault/key-vault-get-started/) para obter mais informações.</span><span class="sxs-lookup"><span data-stu-id="e9d22-192">See [Get started with Azure Key Vault](/azure/key-vault/key-vault-get-started/) for more information.</span></span>

### <a name="add-configuration-settings-to-your-key-vault"></a><span data-ttu-id="e9d22-193">Adicionar definições de configuração para o cofre da chave</span><span class="sxs-lookup"><span data-stu-id="e9d22-193">Add configuration settings to your key vault</span></span>
1. <span data-ttu-id="e9d22-194">Execute SetupKeyVault.ps da seguinte maneira:</span><span class="sxs-lookup"><span data-stu-id="e9d22-194">Run SetupKeyVault.ps as follows::</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Redis--Configuration -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true" 
    ```
    <span data-ttu-id="e9d22-195">onde </span><span class="sxs-lookup"><span data-stu-id="e9d22-195">where</span></span>
   
   * <span data-ttu-id="e9d22-196">nome do cofre da chave = o nome que você deu ao cofre da chave na etapa anterior.</span><span class="sxs-lookup"><span data-stu-id="e9d22-196">key vault name = The name that you gave the key vault in the previous step.</span></span>
   * <span data-ttu-id="e9d22-197">nome DNS de Redis = o nome DNS de sua instância de cache Redis.</span><span class="sxs-lookup"><span data-stu-id="e9d22-197">Redis DNS name = The DNS name of your Redis cache instance.</span></span>
   * <span data-ttu-id="e9d22-198">chave de acesso Redis = a chave de acesso da sua instância de cache Redis.</span><span class="sxs-lookup"><span data-stu-id="e9d22-198">Redis access key = The access key for your Redis cache instance.</span></span>
     
2. <span data-ttu-id="e9d22-199">Agora, convém testar se você armazenou os segredos no cofre da chave com êxito.</span><span class="sxs-lookup"><span data-stu-id="e9d22-199">At this point, it's a good idea to test whether you successfully stored the secrets to key vault.</span></span> <span data-ttu-id="e9d22-200">Execute o seguinte comando do PowerShell:</span><span class="sxs-lookup"><span data-stu-id="e9d22-200">Run the following PowerShell command:</span></span>
   
    ```
    Get-AzureKeyVaultSecret <<key vault name>> Redis--Configuration | Select-Object *
    ```

3. <span data-ttu-id="e9d22-201">Execute SetupKeyVault.ps novamente para adicionar a cadeia de conexão do banco de dados:</span><span class="sxs-lookup"><span data-stu-id="e9d22-201">Run SetupKeyVault.ps again to add the database connection string:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Data--SurveysConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```
   
    <span data-ttu-id="e9d22-202">em que `<<DB connection string>>` é o valor da cadeia de conexão do banco de dados.</span><span class="sxs-lookup"><span data-stu-id="e9d22-202">where `<<DB connection string>>` is the value of the database connection string.</span></span>
   
    <span data-ttu-id="e9d22-203">Para testar com o banco de dados local, copie a cadeia de conexão do arquivo Tailspin.Surveys.Web/appsettings.json.</span><span class="sxs-lookup"><span data-stu-id="e9d22-203">For testing with the local database, copy the connection string from the Tailspin.Surveys.Web/appsettings.json file.</span></span> <span data-ttu-id="e9d22-204">Se você fizer isso, altere as duas barras invertidas ('\\\\') para uma barra invertida.</span><span class="sxs-lookup"><span data-stu-id="e9d22-204">If you do that, make sure to change the double backslash ('\\\\') into a single backslash.</span></span> <span data-ttu-id="e9d22-205">A barra dupla invertida é um caractere de escape no arquivo JSON.</span><span class="sxs-lookup"><span data-stu-id="e9d22-205">The double backslash is an escape character in the JSON file.</span></span>
   
    <span data-ttu-id="e9d22-206">Exemplo:</span><span class="sxs-lookup"><span data-stu-id="e9d22-206">Example:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName Data--SurveysConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true" 
    ```

### <a name="uncomment-the-code-that-enables-key-vault"></a><span data-ttu-id="e9d22-207">Remover o comentário do código que habilita o Cofre da Chave</span><span class="sxs-lookup"><span data-stu-id="e9d22-207">Uncomment the code that enables Key Vault</span></span>
1. <span data-ttu-id="e9d22-208">Abra a solução Tailspin.Surveys.</span><span class="sxs-lookup"><span data-stu-id="e9d22-208">Open the Tailspin.Surveys solution.</span></span>
2. <span data-ttu-id="e9d22-209">No Tailspin.Surveys.Web/Startup.cs, localize o seguinte bloco de código e remova a marca de comentário.</span><span class="sxs-lookup"><span data-stu-id="e9d22-209">In Tailspin.Surveys.Web/Startup.cs, locate the following code block and uncomment it.</span></span>
   
    ```csharp
    //var config = builder.Build();
    //builder.AddAzureKeyVault(
    //    $"https://{config["KeyVault:Name"]}.vault.azure.net/",
    //    config["AzureAd:ClientId"],
    //    config["AzureAd:ClientSecret"]);
    ```
3. <span data-ttu-id="e9d22-210">No Tailspin.Surveys.Web/Startup.cs, localize o código que registra o `ICredentialService`.</span><span class="sxs-lookup"><span data-stu-id="e9d22-210">In Tailspin.Surveys.Web/Startup.cs, locate the code that registers the `ICredentialService`.</span></span> <span data-ttu-id="e9d22-211">Remova a marca de comentário da linha que usa `CertificateCredentialService` e comente a linha que usa `ClientCredentialService`:</span><span class="sxs-lookup"><span data-stu-id="e9d22-211">Uncomment the line that uses `CertificateCredentialService`, and comment out the line that uses `ClientCredentialService`:</span></span>
   
    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```
   
    <span data-ttu-id="e9d22-212">Essa alteração habilita o aplicativo Web a usar [Asserção de cliente][client-assertion] para obter tokens de acesso OAuth.</span><span class="sxs-lookup"><span data-stu-id="e9d22-212">This change enables the web app to use [Client assertion][client-assertion] to get OAuth access tokens.</span></span> <span data-ttu-id="e9d22-213">Com a declaração de cliente, você não precisa de um segredo do cliente OAuth.</span><span class="sxs-lookup"><span data-stu-id="e9d22-213">With client assertion, you don't need an OAuth client secret.</span></span> <span data-ttu-id="e9d22-214">Como alternativa, você pode armazenar o segredo do cliente no cofre da chave.</span><span class="sxs-lookup"><span data-stu-id="e9d22-214">Alternatively, you could store the client secret in key vault.</span></span> <span data-ttu-id="e9d22-215">No entanto, o cofre da chave e a declaração de cliente usam um cliente de certificado, portanto, se você habilitar o cofre da chave, convém habilitar a declaração de cliente também.</span><span class="sxs-lookup"><span data-stu-id="e9d22-215">However, key vault and client assertion both use a client certificate, so if you enable key vault, it's a good practice to enable client assertion as well.</span></span>

### <a name="update-the-user-secrets"></a><span data-ttu-id="e9d22-216">Atualizar os segredos do usuário</span><span class="sxs-lookup"><span data-stu-id="e9d22-216">Update the user secrets</span></span>
<span data-ttu-id="e9d22-217">No Gerenciador de Soluções, clique com o botão direito no projeto Tailspin.Surveys.Web e selecione **Gerenciar Segredos do Usuário**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-217">In Solution Explorer, right-click the Tailspin.Surveys.Web project and select **Manage User Secrets**.</span></span> <span data-ttu-id="e9d22-218">No arquivo secrets.json, exclua o JSON existente e cole o seguinte:</span><span class="sxs-lookup"><span data-stu-id="e9d22-218">In the secrets.json file, delete the existing JSON and paste in the following:</span></span>

    ```
    {
      "AzureAd": {
        "ClientId": "[Surveys web app client ID]",
        "ClientSecret": "[Surveys web app client secret]",
        "PostLogoutRedirectUri": "https://localhost:44300/",
        "WebApiResourceId": "[App ID URI of your Surveys.WebAPI application]",
        "Asymmetric": {
          "CertificateThumbprint": "[certificate thumbprint. Example: 105b2ff3bc842c53582661716db1b7cdc6b43ec9]",
          "StoreName": "My",
          "StoreLocation": "CurrentUser",
          "ValidationRequired": "false"
        }
      },
      "KeyVault": {
        "Name": "[key vault name]"
      }
    }
    ```

<span data-ttu-id="e9d22-219">Substitua as entradas entre [colchetes] pelos valores corretos.</span><span class="sxs-lookup"><span data-stu-id="e9d22-219">Replace the entries in [square brackets] with the correct values.</span></span>

* <span data-ttu-id="e9d22-220">`AzureAd:ClientId`: a ID do cliente do aplicativo Surveys.</span><span class="sxs-lookup"><span data-stu-id="e9d22-220">`AzureAd:ClientId`: The client ID of the Surveys app.</span></span>
* <span data-ttu-id="e9d22-221">`AzureAd:ClientSecret`: a chave que foi gerada quando você registrou o aplicativo de pesquisas no Azure AD.</span><span class="sxs-lookup"><span data-stu-id="e9d22-221">`AzureAd:ClientSecret`: The key that you generated when you registered the Surveys application in Azure AD.</span></span>
* <span data-ttu-id="e9d22-222">`AzureAd:WebApiResourceId`: o URI da ID do Aplicativo que você especificou quando criou o aplicativo Surveys.WebAPI no Azure AD.</span><span class="sxs-lookup"><span data-stu-id="e9d22-222">`AzureAd:WebApiResourceId`: The App ID URI that you specified when you created the Surveys.WebAPI application in Azure AD.</span></span>
* <span data-ttu-id="e9d22-223">`Asymmetric:CertificateThumbprint`: a impressão digital do certificado que você obteve anteriormente, quando criou o certificado do cliente.</span><span class="sxs-lookup"><span data-stu-id="e9d22-223">`Asymmetric:CertificateThumbprint`: The certificate thumbprint that you got previously, when you created the client certificate.</span></span>
* <span data-ttu-id="e9d22-224">`KeyVault:Name`: o nome de seu cofre da chave.</span><span class="sxs-lookup"><span data-stu-id="e9d22-224">`KeyVault:Name`: The name of your key vault.</span></span>

> [!NOTE]
> <span data-ttu-id="e9d22-225">`Asymmetric:ValidationRequired` é false, pois o certificado que você criou anteriormente não foi assinado por uma AC (autoridade de certificação) raiz.</span><span class="sxs-lookup"><span data-stu-id="e9d22-225">`Asymmetric:ValidationRequired` is false because the certificate that you created previously was not signed by a root certificate authority (CA).</span></span> <span data-ttu-id="e9d22-226">Em produção, use um certificado assinado por uma AC raiz e defina `ValidationRequired` como true.</span><span class="sxs-lookup"><span data-stu-id="e9d22-226">In production, use a certificate that is signed by a root CA and set `ValidationRequired` to true.</span></span>
> 
> 

<span data-ttu-id="e9d22-227">Salve o arquivo secrets.json atualizado.</span><span class="sxs-lookup"><span data-stu-id="e9d22-227">Save the updated secrets.json file.</span></span>

<span data-ttu-id="e9d22-228">Em seguida, no Gerenciador de Soluções, clique com o botão direito do mouse no projeto Tailspin.Surveys.WebApi e selecione **Gerenciar Segredos do Usuário**.</span><span class="sxs-lookup"><span data-stu-id="e9d22-228">Next, in Solution Explorer, right-click the Tailspin.Surveys.WebApi project and select **Manage User Secrets**.</span></span> <span data-ttu-id="e9d22-229">Exclua o arquivo JSON existente e cole o seguinte:</span><span class="sxs-lookup"><span data-stu-id="e9d22-229">Delete the existing JSON and paste in the following:</span></span>

```
{
  "AzureAd": {
    "ClientId": "[Surveys.WebAPI client ID]",
    "WebApiResourceId": "https://tailspin5.onmicrosoft.com/surveys.webapi",
    "Asymmetric": {
      "CertificateThumbprint": "[certificate thumbprint]",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "ValidationRequired": "false"
    }
  },
  "KeyVault": {
    "Name": "[key vault name]"
  }
}
```

<span data-ttu-id="e9d22-230">Substitua as entradas entre [colchetes] e salve o arquivo secrets.json.</span><span class="sxs-lookup"><span data-stu-id="e9d22-230">Replace the entries in [square brackets] and save the secrets.json file.</span></span>

> [!NOTE]
> <span data-ttu-id="e9d22-231">Para a API Web, use a ID do cliente para o aplicativo Surveys.WebAPI, não para o aplicativo Surveys.</span><span class="sxs-lookup"><span data-stu-id="e9d22-231">For the web API, make sure to use the client ID for the Surveys.WebAPI application, not the Surveys application.</span></span>
> 
> 

<span data-ttu-id="e9d22-232">[**Avançar**][adfs]</span><span class="sxs-lookup"><span data-stu-id="e9d22-232">[**Next**][adfs]</span></span>

<!-- Links -->
[adfs]: ./adfs.md
[authorize-app]: /azure/key-vault/key-vault-get-started//#authorize
[azure-portal]: https://portal.azure.com
[azure-rm-cmdlets]: https://msdn.microsoft.com/library/mt125356.aspx
[client-assertion]: client-assertion.md
[configuration]: /aspnet/core/fundamentals/configuration
[KeyVault]: https://azure.microsoft.com/services/key-vault/
[key-tags]: https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_Keytags
[Microsoft.Azure.KeyVault]: https://www.nuget.org/packages/Microsoft.Azure.KeyVault/
[options]: /aspnet/core/fundamentals/configuration#using-options-and-configuration-objects
[readme]: ./run-the-app.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[user-secrets]: http://go.microsoft.com/fwlink/?LinkID=532709
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
