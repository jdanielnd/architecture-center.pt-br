---
title: Atualizar um recurso em um modelo do Azure Resource Manager
description: Descreve como estender a funcionalidade dos modelos do Azure Resource Manager para atualizar um recurso.
author: petertay
ms.date: 10/31/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: de76e69e94917bbbe94c0f87fda2cdbe415181dc
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487129"
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="ba467-103">Atualizar um recurso em um modelo do Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="ba467-103">Update a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="ba467-104">Existem alguns cenários nos quais você precisa atualizar um recurso durante a implantação.</span><span class="sxs-lookup"><span data-stu-id="ba467-104">There are some scenarios in which you need to update a resource during a deployment.</span></span> <span data-ttu-id="ba467-105">Você pode encontrar esse cenário quando não é possível especificar todas as propriedades de um recurso até que outros recursos dependentes sejam criados.</span><span class="sxs-lookup"><span data-stu-id="ba467-105">You might encounter this scenario when you cannot specify all the properties for a resource until other, dependent resources are created.</span></span> <span data-ttu-id="ba467-106">Por exemplo, se você criar um pool de back-end para um balanceador de carga, poderá atualizar as NICs (adaptadores de rede) nas VMs (máquinas virtuais) para incluí-las no pool de back-end.</span><span class="sxs-lookup"><span data-stu-id="ba467-106">For example, if you create a backend pool for a load balancer, you might update the network interfaces (NICs) on your virtual machines (VMs) to include them in the backend pool.</span></span> <span data-ttu-id="ba467-107">E, embora o Resource Manager dê suporte à atualização de recursos durante a implantação, é necessário projetar o modelo corretamente para evitar erros e garantir que a implantação é tratada como uma atualização.</span><span class="sxs-lookup"><span data-stu-id="ba467-107">And while Resource Manager supports updating resources during deployment, you must design your template correctly to avoid errors and to ensure the deployment is handled as an update.</span></span>

<span data-ttu-id="ba467-108">Primeiro, é necessário referenciar o recurso uma vez no modelo para criá-lo e então referenciar o recurso com o mesmo nome para atualizá-lo mais tarde.</span><span class="sxs-lookup"><span data-stu-id="ba467-108">First, you must reference the resource once in the template to create it and then reference the resource by the same name to update it later.</span></span> <span data-ttu-id="ba467-109">No entanto, se dois recursos tiverem o mesmo nome em um modelo, o Resource Manager gerará uma exceção.</span><span class="sxs-lookup"><span data-stu-id="ba467-109">However, if two resources have the same name in a template, Resource Manager throws an exception.</span></span> <span data-ttu-id="ba467-110">Para evitar esse erro, especifique o recurso atualizado em um segundo modelo que está vinculado ou incluído como um submodelo usando o tipo de recurso `Microsoft.Resources/deployments`.</span><span class="sxs-lookup"><span data-stu-id="ba467-110">To avoid this error, specify the updated resource in a second template that's either linked or included as a subtemplate using the `Microsoft.Resources/deployments` resource type.</span></span>

<span data-ttu-id="ba467-111">Em segundo lugar, você deve especificar o nome da propriedade existente a ser alterado ou um novo nome de uma propriedade a ser adicionado ao modelo aninhado.</span><span class="sxs-lookup"><span data-stu-id="ba467-111">Second, you must either specify the name of the existing property to change or a new name for a property to add in the nested template.</span></span> <span data-ttu-id="ba467-112">Você também deve especificar as propriedades originais e seus valores originais.</span><span class="sxs-lookup"><span data-stu-id="ba467-112">You must also specify the original properties and their original values.</span></span> <span data-ttu-id="ba467-113">Se você não fornecer as propriedades e os valores originais, o Resource Manager considerará que você deseja criar um novo recurso e excluirá o recurso original.</span><span class="sxs-lookup"><span data-stu-id="ba467-113">If you fail to provide the original properties and values, Resource Manager assumes you want to create a new resource and deletes the original resource.</span></span>

## <a name="example-template"></a><span data-ttu-id="ba467-114">Modelo de exemplo</span><span class="sxs-lookup"><span data-stu-id="ba467-114">Example template</span></span>

<span data-ttu-id="ba467-115">Vamos examinar um modelo de exemplo que demonstra isso.</span><span class="sxs-lookup"><span data-stu-id="ba467-115">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="ba467-116">Nosso modelo implanta uma rede virtual chamada `firstVNet` que tem uma sub-rede chamada `firstSubnet`.</span><span class="sxs-lookup"><span data-stu-id="ba467-116">Our template deploys a virtual network named `firstVNet` that has one subnet named `firstSubnet`.</span></span> <span data-ttu-id="ba467-117">Em seguida, ele implanta uma NIC (interface de rede virtual) chamada `nic1` e a associa à nossa sub-rede.</span><span class="sxs-lookup"><span data-stu-id="ba467-117">It then deploys a virtual network interface (NIC) named `nic1` and associates it with our subnet.</span></span> <span data-ttu-id="ba467-118">Em seguida, um recurso de implantação denominado `updateVNet` inclui um modelo aninhado que atualiza nosso recurso `firstVNet` ao adicionar uma segunda sub-rede denominada `secondSubnet`.</span><span class="sxs-lookup"><span data-stu-id="ba467-118">Then, a deployment resource named `updateVNet` includes a nested template that updates our `firstVNet` resource by adding a second subnet named `secondSubnet`.</span></span>

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
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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

<span data-ttu-id="ba467-119">Vamos dar uma olhada no objeto de recurso de nosso recurso `firstVNet` primeiro.</span><span class="sxs-lookup"><span data-stu-id="ba467-119">Let's take a look at the resource object for our `firstVNet` resource first.</span></span> <span data-ttu-id="ba467-120">Observe que especificamos as configurações para nosso `firstVNet` em um modelo aninhado&mdash;. Isso ocorre porque o Resource Manager não permite que o mesmo nome de implantação no mesmo modelo e modelos aninhados são considerados como um modelo diferente.</span><span class="sxs-lookup"><span data-stu-id="ba467-120">Notice that we respecify the settings for our `firstVNet` in a nested template&mdash;this is because Resource Manager doesn't allow the same deployment name within the same template and nested templates are considered to be a different template.</span></span> <span data-ttu-id="ba467-121">Ao especificarmos nossos valores para o nosso recurso `firstSubnet`, estamos informando o Resource Manager para atualizar o recurso existente em vez de excluí-lo e reimplantá-lo.</span><span class="sxs-lookup"><span data-stu-id="ba467-121">By respecifying our values for our `firstSubnet` resource, we are telling Resource Manager to update the existing resource instead of deleting it and redeploying it.</span></span> <span data-ttu-id="ba467-122">Por fim, nossas novas configurações para `secondSubnet` são selecionadas durante essa atualização.</span><span class="sxs-lookup"><span data-stu-id="ba467-122">Finally, our new settings for `secondSubnet` are picked up during this update.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="ba467-123">Experimentar o modelo</span><span class="sxs-lookup"><span data-stu-id="ba467-123">Try the template</span></span>

<span data-ttu-id="ba467-124">Um modelo de exemplo está disponível no [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="ba467-124">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="ba467-125">Para implantar o modelo, execute os seguintes comandos da [CLI do Azure][cli]:</span><span class="sxs-lookup"><span data-stu-id="ba467-125">To deploy the template, run the following [Azure CLI][cli] commands:</span></span>

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example1-update/deploy.json
```

<span data-ttu-id="ba467-126">Depois que a implantação for concluída, abra o grupo de recursos que você especificou no portal.</span><span class="sxs-lookup"><span data-stu-id="ba467-126">Once deployment has finished, open the resource group you specified in the portal.</span></span> <span data-ttu-id="ba467-127">Você vê uma rede virtual denominada `firstVNet` e uma NIC denominada `nic1`.</span><span class="sxs-lookup"><span data-stu-id="ba467-127">You see a virtual network named `firstVNet` and a NIC named `nic1`.</span></span> <span data-ttu-id="ba467-128">Clique em `firstVNet` e, depois, em `subnets`.</span><span class="sxs-lookup"><span data-stu-id="ba467-128">Click `firstVNet`, then click `subnets`.</span></span> <span data-ttu-id="ba467-129">Você verá a `firstSubnet` que foi originalmente criada e a `secondSubnet` que foi adicionada ao recurso `updateVNet`.</span><span class="sxs-lookup"><span data-stu-id="ba467-129">You see the `firstSubnet` that was originally created, and you see the `secondSubnet` that was added in the `updateVNet` resource.</span></span>

![Sub-rede original e sub-rede atualizada](../_images/firstVNet-subnets.png)

<span data-ttu-id="ba467-131">Depois, retorne ao grupo de recursos e clique em `nic1`. Em seguida, clique em `IP configurations`.</span><span class="sxs-lookup"><span data-stu-id="ba467-131">Then, go back to the resource group and click `nic1` then click `IP configurations`.</span></span> <span data-ttu-id="ba467-132">Na seção `IP configurations`, a `subnet` é definida como `firstSubnet (10.0.0.0/24)`.</span><span class="sxs-lookup"><span data-stu-id="ba467-132">In the `IP configurations` section, the `subnet` is set to `firstSubnet (10.0.0.0/24)`.</span></span>

![definições de configuração de IP da NIC1](../_images/nic1-ipconfigurations.png)

<span data-ttu-id="ba467-134">A `firstVNet` original foi atualizada em vez de recriada.</span><span class="sxs-lookup"><span data-stu-id="ba467-134">The original `firstVNet` has been updated instead of recreated.</span></span> <span data-ttu-id="ba467-135">Se `firstVNet` tivesse sido recriada, a `nic1` não seria associada a `firstVNet`.</span><span class="sxs-lookup"><span data-stu-id="ba467-135">If `firstVNet` had been recreated, `nic1` would not be associated with `firstVNet`.</span></span>

## <a name="next-steps"></a><span data-ttu-id="ba467-136">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="ba467-136">Next steps</span></span>

* <span data-ttu-id="ba467-137">Saiba como implantar um recurso com base em uma condição, como se um valor de parâmetro estivesse presente.</span><span class="sxs-lookup"><span data-stu-id="ba467-137">Learn how deploy a resource based on a condition, such as whether a parameter value is present.</span></span> <span data-ttu-id="ba467-138">Confira [Implantar condicionalmente um recurso em um modelo do Azure Resource Manager](./conditional-deploy.md).</span><span class="sxs-lookup"><span data-stu-id="ba467-138">See [Conditionally deploy a resource in an Azure Resource Manager template](./conditional-deploy.md).</span></span>

[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
