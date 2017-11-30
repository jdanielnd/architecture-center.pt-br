---
title: Atualizar um recurso em um modelo do Azure Resource Manager
description: Descreve como estender a funcionalidade dos modelos do Azure Resource Manager para atualizar um recurso
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: fc2565819c66ee7695224ef5793e7276e6e552e0
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="b9c42-103">Atualizar um recurso em um modelo do Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="b9c42-103">Update a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="b9c42-104">Existem alguns cenários nos quais você precisa atualizar um recurso durante a implantação.</span><span class="sxs-lookup"><span data-stu-id="b9c42-104">There are some scenarios in which you need to update a resource during a deployment.</span></span> <span data-ttu-id="b9c42-105">Você pode encontrar esse cenário quando não é possível especificar todas as propriedades de um recurso até que outros recursos dependentes sejam criados.</span><span class="sxs-lookup"><span data-stu-id="b9c42-105">You might encounter this scenario when you cannot specify all the properties for a resource until other, dependent resources are created.</span></span> <span data-ttu-id="b9c42-106">Por exemplo, se você criar um pool de back-end para um balanceador de carga, poderá atualizar as NICs (adaptadores de rede) nas VMs (máquinas virtuais) para incluí-las no pool de back-end.</span><span class="sxs-lookup"><span data-stu-id="b9c42-106">For example, if you create a backend pool for a load balancer, you might update the network interfaces (NICs) on your virtual machines (VMs) to include them in the backend pool.</span></span> <span data-ttu-id="b9c42-107">E, embora o Resource Manager dê suporte à atualização de recursos durante a implantação, é necessário projetar o modelo corretamente para evitar erros e garantir que a implantação é tratada como uma atualização.</span><span class="sxs-lookup"><span data-stu-id="b9c42-107">And while Resource Manager supports updating resources during deployment, you must design your template correctly to avoid errors and to ensure the deployment is handled as an update.</span></span>

<span data-ttu-id="b9c42-108">Primeiro, é necessário referenciar o recurso uma vez no modelo para criá-lo e então referenciar o recurso com o mesmo nome para atualizá-lo mais tarde.</span><span class="sxs-lookup"><span data-stu-id="b9c42-108">First, you must reference the resource once in the template to create it and then reference the resource by the same name to update it later.</span></span> <span data-ttu-id="b9c42-109">No entanto, se dois recursos tiverem o mesmo nome em um modelo, o Resource Manager gerará uma exceção.</span><span class="sxs-lookup"><span data-stu-id="b9c42-109">However, if two resources have the same name in a template, Resource Manager throws an exception.</span></span> <span data-ttu-id="b9c42-110">Para evitar esse erro, especifique o recurso atualizado em um segundo modelo que está vinculado ou incluído como um submodelo usando o tipo de recurso `Microsoft.Resources/deployments`.</span><span class="sxs-lookup"><span data-stu-id="b9c42-110">To avoid this error, specify the updated resource in a second template that's either linked or included as a subtemplate using the `Microsoft.Resources/deployments` resource type.</span></span>

<span data-ttu-id="b9c42-111">Em segundo lugar, você deve especificar o nome da propriedade existente a ser alterado ou um novo nome de uma propriedade a ser adicionado ao modelo aninhado.</span><span class="sxs-lookup"><span data-stu-id="b9c42-111">Second, you must either specify the name of the existing property to change or a new name for a property to add in the nested template.</span></span> <span data-ttu-id="b9c42-112">Você também deve especificar as propriedades originais e seus valores originais.</span><span class="sxs-lookup"><span data-stu-id="b9c42-112">You must also specify the original properties and their original values.</span></span> <span data-ttu-id="b9c42-113">Se você não fornecer as propriedades e os valores originais, o Resource Manager considerará que você deseja criar um novo recurso e excluirá o recurso original.</span><span class="sxs-lookup"><span data-stu-id="b9c42-113">If you fail to provide the original properties and values, Resource Manager assumes you want to create a new resource and deletes the original resource.</span></span>

## <a name="example-template"></a><span data-ttu-id="b9c42-114">Modelo de exemplo</span><span class="sxs-lookup"><span data-stu-id="b9c42-114">Example template</span></span>

<span data-ttu-id="b9c42-115">Vamos examinar um modelo de exemplo que demonstra isso.</span><span class="sxs-lookup"><span data-stu-id="b9c42-115">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="b9c42-116">Nosso modelo implanta uma rede virtual chamada `firstVNet` que tem uma sub-rede chamada `firstSubnet`.</span><span class="sxs-lookup"><span data-stu-id="b9c42-116">Our template deploys a virtual network  named `firstVNet` that has one subnet named `firstSubnet`.</span></span> <span data-ttu-id="b9c42-117">Em seguida, ele implanta uma NIC (interface de rede virtual) chamada `nic1` e a associa à nossa sub-rede.</span><span class="sxs-lookup"><span data-stu-id="b9c42-117">It then deploys a virtual network interface (NIC) named `nic1` and associates it with our subnet.</span></span> <span data-ttu-id="b9c42-118">Em seguida, um recurso de implantação denominado `updateVNet` inclui um modelo aninhado que atualiza nosso recurso `firstVNet` ao adicionar uma segunda sub-rede denominada `secondSubnet`.</span><span class="sxs-lookup"><span data-stu-id="b9c42-118">Then, a deployment resource named `updateVNet` includes a nested template that updates our `firstVNet` resource by adding a second subnet named `secondSubnet`.</span></span> 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
      {
      "apiVersion": "2016-03-30",
      "name": "firstVNet",
      "location":"[resourceGroup().location]",
      "type": "Microsoft.Network/virtualNetworks",
      "properties": {
          "addressSpace":{"addressPrefixes": [
              "10.0.0.0/22"
          ]},
          "subnets":[              
              {
                  "name":"firstSubnet",
                  "properties":{
                    "addressPrefix":"10.0.0.0/24"
                  }
              }
            ]
      }
    },
    {
        "apiVersion": "2015-06-15",
        "type":"Microsoft.Network/networkInterfaces",
        "name":"nic1",
        "location":"[resourceGroup().location]",
        "dependsOn": [
            "firstVNet"
        ],
        "properties": {
            "ipConfigurations":[
                {
                    "name":"ipconfig1",
                    "properties": {
                        "privateIPAllocationMethod":"Dynamic",
                        "subnet": {
                            "id": "[concat(resourceId('Microsoft.Network/virtualNetworks','firstVNet'),'/subnets/firstSubnet')]"
                        }
                    }
                }
            ]
        }
    },
    {
      "apiVersion": "2015-01-01",
      "type": "Microsoft.Resources/deployments",
      "name": "updateVNet",
      "dependsOn": [
          "nic1"
      ],
      "properties": {
        "mode": "Incremental",
        "parameters": {},
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
              {
                  "apiVersion": "2016-03-30",
                  "name": "firstVNet",
                  "location":"[resourceGroup().location]",
                  "type": "Microsoft.Network/virtualNetworks",
                  "properties": {
                      "addressSpace": "[reference('firstVNet').addressSpace]",
                      "subnets":[
                          {
                              "name":"[reference('firstVNet').subnets[0].name]",
                              "properties":{
                                  "addressPrefix":"[reference('firstVNet').subnets[0].properties.addressPrefix]"
                                  }
                          },
                          {
                              "name":"secondSubnet",
                              "properties":{
                                  "addressPrefix":"10.0.1.0/24"
                                  }
                          }
                     ]
                  }
              }
          ],
          "outputs": {}
          }
        }
    }
  ],
  "outputs": {}
}
```

<span data-ttu-id="b9c42-119">Vamos dar uma olhada no objeto de recurso de nosso recurso `firstVNet` primeiro.</span><span class="sxs-lookup"><span data-stu-id="b9c42-119">Let's take a look at the resource object for our `firstVNet` resource first.</span></span> <span data-ttu-id="b9c42-120">Observe que especificamos as configurações para nosso `firstVNet` em um modelo aninhado&mdash;. Isso ocorre porque o Resource Manager não permite que o mesmo nome de implantação no mesmo modelo e modelos aninhados são considerados como um modelo diferente.</span><span class="sxs-lookup"><span data-stu-id="b9c42-120">Notice that we respecify the settings for our `firstVNet` in a nested template&mdash;this is because Resource Manager doesn't allow the same deployment name within the same template and nested templates are considered to be a different template.</span></span> <span data-ttu-id="b9c42-121">Ao especificarmos nossos valores para o nosso recurso `firstSubnet`, estamos informando o Resource Manager para atualizar o recurso existente em vez de excluí-lo e reimplantá-lo.</span><span class="sxs-lookup"><span data-stu-id="b9c42-121">By respecifying our values for our `firstSubnet` resource, we are telling Resource Manager to update the existing resource instead of deleting it and redeploying it.</span></span> <span data-ttu-id="b9c42-122">Por fim, nossas novas configurações para `secondSubnet` são selecionadas durante essa atualização.</span><span class="sxs-lookup"><span data-stu-id="b9c42-122">Finally, our new settings for `secondSubnet` are picked up during this update.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="b9c42-123">Experimentar o modelo</span><span class="sxs-lookup"><span data-stu-id="b9c42-123">Try the template</span></span>

<span data-ttu-id="b9c42-124">Para experimentar esse modelo, siga estas etapas:</span><span class="sxs-lookup"><span data-stu-id="b9c42-124">If you would like to experiment with this template, follow these steps:</span></span>

1.  <span data-ttu-id="b9c42-125">Visite o portal do Azure, selecione o ícone **+** e pesquise pelo tipo de recurso **implantação de modelo** e selecione-o.</span><span class="sxs-lookup"><span data-stu-id="b9c42-125">Go to the Azure portal, select the **+** icon, and search for the **template deployment** resource type, and select it.</span></span>
2.  <span data-ttu-id="b9c42-126">Quando chegar à página **Implantação de modelo**, selecione o botão **criar**.</span><span class="sxs-lookup"><span data-stu-id="b9c42-126">Navigate to the **template deployment** page, select the **create** button.</span></span> <span data-ttu-id="b9c42-127">Esse botão abre a folha **implantação personalizada**.</span><span class="sxs-lookup"><span data-stu-id="b9c42-127">This button opens the **custom deployment** blade.</span></span>
3.  <span data-ttu-id="b9c42-128">Selecione o ícone **Editar**.</span><span class="sxs-lookup"><span data-stu-id="b9c42-128">Select the **edit** icon.</span></span>
4.  <span data-ttu-id="b9c42-129">Exclua o modelo vazio.</span><span class="sxs-lookup"><span data-stu-id="b9c42-129">Delete the empty template.</span></span>
5.  <span data-ttu-id="b9c42-130">Copie e cole o modelo de exemplo no painel direito.</span><span class="sxs-lookup"><span data-stu-id="b9c42-130">Copy and paste the sample template into the right pane.</span></span>
6.  <span data-ttu-id="b9c42-131">Selecione o botão **Salvar**.</span><span class="sxs-lookup"><span data-stu-id="b9c42-131">Select the **save** button.</span></span>
7.  <span data-ttu-id="b9c42-132">Você retorna ao painel **Implantação personalizada**, mas desta vez há algumas caixas de listagem.</span><span class="sxs-lookup"><span data-stu-id="b9c42-132">You return to the **custom deployment** pane, but this time there are some drop-down list boxes.</span></span> <span data-ttu-id="b9c42-133">Selecione sua assinatura, crie um novo grupo de recursos ou use o existente e selecione um local.</span><span class="sxs-lookup"><span data-stu-id="b9c42-133">Select your subscription, either create new or use existing resource group, and select a location.</span></span> <span data-ttu-id="b9c42-134">Examine os termos e condições e, em seguida, selecione o botão **Eu concordo**.</span><span class="sxs-lookup"><span data-stu-id="b9c42-134">Review the terms and conditions, then select the **I agree** button.</span></span>
8.  <span data-ttu-id="b9c42-135">Selecione o botão **Comprar**.</span><span class="sxs-lookup"><span data-stu-id="b9c42-135">Select the **purchase** button.</span></span>

<span data-ttu-id="b9c42-136">Depois que a implantação for concluída, abra o grupo de recursos que você especificou no portal.</span><span class="sxs-lookup"><span data-stu-id="b9c42-136">Once deployment has finished, open the resource group you specified in the portal.</span></span> <span data-ttu-id="b9c42-137">Você vê uma rede virtual denominada `firstVNet` e uma NIC denominada `nic1`.</span><span class="sxs-lookup"><span data-stu-id="b9c42-137">You see a virtual network named `firstVNet` and a NIC named `nic1`.</span></span> <span data-ttu-id="b9c42-138">Clique em `firstVNet` e, depois, em `subnets`.</span><span class="sxs-lookup"><span data-stu-id="b9c42-138">Click `firstVNet`, then click `subnets`.</span></span> <span data-ttu-id="b9c42-139">Você verá a `firstSubnet` que foi originalmente criada e a `secondSubnet` que foi adicionada ao recurso `updateVNet`.</span><span class="sxs-lookup"><span data-stu-id="b9c42-139">You see the `firstSubnet` that was originally created, and you see the `secondSubnet` that was added in the `updateVNet` resource.</span></span> 

![Sub-rede original e sub-rede atualizada](../_images/firstVNet-subnets.png)

<span data-ttu-id="b9c42-141">Depois, retorne ao grupo de recursos e clique em `nic1`. Em seguida, clique em `IP configurations`.</span><span class="sxs-lookup"><span data-stu-id="b9c42-141">Then, go back to the resource group and click `nic1` then click `IP configurations`.</span></span> <span data-ttu-id="b9c42-142">Na seção `IP configurations`, a `subnet` é definida como `firstSubnet (10.0.0.0/24)`.</span><span class="sxs-lookup"><span data-stu-id="b9c42-142">In the `IP configurations` section, the `subnet` is set to `firstSubnet (10.0.0.0/24)`.</span></span> 

![definições de configuração de IP da NIC1](../_images/nic1-ipconfigurations.png)

<span data-ttu-id="b9c42-144">A `firstVNet` original foi atualizada em vez de recriada.</span><span class="sxs-lookup"><span data-stu-id="b9c42-144">The original `firstVNet` has been updated instead of recreated.</span></span> <span data-ttu-id="b9c42-145">Se `firstVNet` tivesse sido recriada, a `nic1` não seria associada a `firstVNet`.</span><span class="sxs-lookup"><span data-stu-id="b9c42-145">If `firstVNet` had been recreated, `nic1` would not be associated with `firstVNet`.</span></span>

## <a name="next-steps"></a><span data-ttu-id="b9c42-146">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="b9c42-146">Next steps</span></span>

* <span data-ttu-id="b9c42-147">Essa técnica também é implementada no [projeto de blocos de construção do modelo](https://github.com/mspnp/template-building-blocks) e nas [arquiteturas de referência do Azure](/azure/architecture/reference-architectures/).</span><span class="sxs-lookup"><span data-stu-id="b9c42-147">This technique is implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="b9c42-148">Você pode usá-los para criar sua própria arquitetura ou implantar uma de nossas arquiteturas de referência.</span><span class="sxs-lookup"><span data-stu-id="b9c42-148">You can use these to create your own architecture or deploy one of our reference architectures.</span></span>
