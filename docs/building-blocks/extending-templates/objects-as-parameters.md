---
title: "Usar um objeto como um parâmetro em um modelo do Azure Resource Manager"
description: "Descreve como estender a funcionalidade dos modelos do Azure Resource Manager para usar objetos como parâmetros"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 08ee1cf2924f78ce366c58e20e84a512785f85cc
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a><span data-ttu-id="a4343-103">Usar um objeto como um parâmetro em um modelo do Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="a4343-103">Use an object as a parameter in an Azure Resource Manager template</span></span>

<span data-ttu-id="a4343-104">Quando você [criar modelos do Azure Resource Manager][azure-resource-manager-create-template], poderá especificar valores de propriedade de recurso diretamente no modelo ou definir um parâmetro e fornecer valores de durante a implantação.</span><span class="sxs-lookup"><span data-stu-id="a4343-104">When you [author Azure Resource Manager templates][azure-resource-manager-create-template], you can either specify resource property values directly in the template or define a parameter and provide values during deployment.</span></span> <span data-ttu-id="a4343-105">Você pode usar um parâmetro para cada valor de propriedade para implantações pequenas, mas há um limite de 255 parâmetros por implantação.</span><span class="sxs-lookup"><span data-stu-id="a4343-105">It's fine to use a parameter for each property value for small deployments, but there is a limit of 255 parameters per deployment.</span></span> <span data-ttu-id="a4343-106">Depois de obter para implantações maiores e mais complexas, você poderá ficar sem parâmetros.</span><span class="sxs-lookup"><span data-stu-id="a4343-106">Once you get to larger and more complex deployments you may run out of parameters.</span></span>

<span data-ttu-id="a4343-107">Uma maneira de resolver esse problema é usar um objeto como parâmetro em vez de um valor.</span><span class="sxs-lookup"><span data-stu-id="a4343-107">One way to solve this problem is to use an object as a parameter instead of a value.</span></span> <span data-ttu-id="a4343-108">Para fazer isso, defina o parâmetro no modelo e especifique um objeto JSON em vez de um único valor durante a implantação.</span><span class="sxs-lookup"><span data-stu-id="a4343-108">To do this, define the parameter in your template and specify a JSON object instead of a single value during deployment.</span></span> <span data-ttu-id="a4343-109">Em seguida, faça referência às subpropriedades do parâmetro usando a [`parameter()`função][azure-resource-manager-functions] e o operador ponto no seu modelo.</span><span class="sxs-lookup"><span data-stu-id="a4343-109">Then, reference the subproperties of the parameter using the [`parameter()` function][azure-resource-manager-functions] and dot operator in your template.</span></span>

<span data-ttu-id="a4343-110">Vamos dar uma olhada em um exemplo que implanta um recurso de rede virtual.</span><span class="sxs-lookup"><span data-stu-id="a4343-110">Let's take a look at an example that deploys a virtual network resource.</span></span> <span data-ttu-id="a4343-111">Primeiro, vamos especificar um parâmetro `VNetSettings` em nosso modelo e defina o `type` como `object`:</span><span class="sxs-lookup"><span data-stu-id="a4343-111">First, let's specify a `VNetSettings` parameter in our template and set the `type` to `object`:</span></span>

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
<span data-ttu-id="a4343-112">Em seguida, vamos fornecer valores para o objeto `VNetSettings`:</span><span class="sxs-lookup"><span data-stu-id="a4343-112">Next, let's provide values for the `VNetSettings` object:</span></span>

> [!NOTE]
> <span data-ttu-id="a4343-113">Para saber como fornecer valores de parâmetro durante a implantação, veja a seção **parâmetros** de [noções básicas sobre a estrutura e a sintaxe dos modelos do Azure Resource Manager][azure-resource-manager-authoring-templates].</span><span class="sxs-lookup"><span data-stu-id="a4343-113">To learn how to provide parameter values during deploment, see the **parameters** section of [understand the structure and syntax of Azure Resource Manager templates][azure-resource-manager-authoring-templates].</span></span> 

```json
"parameters":{
    "VNetSettings":{
        "value":{
            "name":"VNet1",
            "addressPrefixes": [
                {
                    "name": "firstPrefix",
                    "addressPrefix": "10.0.0.0/22"
                }
            ],
            "subnets":[
                {
                    "name": "firstSubnet",
                    "addressPrefix": "10.0.0.0/24"
                },
                {
                    "name":"secondSubnet",
                    "addressPrefix":"10.0.1.0/24"
                }
            ]
        }
    }
}
```

<span data-ttu-id="a4343-114">Como você pode ver, nosso único parâmetro na verdade especifica três subpropriedades: `name`, `addressPrefixes` e `subnets`.</span><span class="sxs-lookup"><span data-stu-id="a4343-114">As you can see, our single parameter actually specifies three subproperties: `name`, `addressPrefixes`, and `subnets`.</span></span> <span data-ttu-id="a4343-115">Cada uma desses subpropriedades especifica um valor ou outras subpropriedades.</span><span class="sxs-lookup"><span data-stu-id="a4343-115">Each of these subproperties either specifies a value or other subproperties.</span></span> <span data-ttu-id="a4343-116">O resultado é que o nosso único parâmetro especifica todos os valores necessários para implantar a rede virtual.</span><span class="sxs-lookup"><span data-stu-id="a4343-116">The result is that our single parameter specifies all the values necessary to deploy our virtual network.</span></span>

<span data-ttu-id="a4343-117">Agora vamos dar uma olhada no restante do nosso modelo para ver como o objeto `VNetSettings` é usado:</span><span class="sxs-lookup"><span data-stu-id="a4343-117">Now let's have a look at the rest of our template to see how the `VNetSettings` object is used:</span></span>

```json
...
"resources": [
    {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/virtualNetworks",
        "name": "[parameters('VNetSettings').name]",
        "location":"[resourceGroup().location]",
        "properties": {
          "addressSpace":{
              "addressPrefixes": [
                "[parameters('VNetSettings').addressPrefixes[0].addressPrefix]"
              ]
          },
          "subnets":[
              {
                  "name":"[parameters('VNetSettings').subnets[0].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[0].addressPrefix]"
                  }
              },
              {
                  "name":"[parameters('VNetSettings').subnets[1].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[1].addressPrefix]"
                  }
              }
          ]
        }
    }
  ]
```
<span data-ttu-id="a4343-118">Os valores de nosso objeto `VNetSettings` são aplicadas para as propriedades solicitadas por nossos recursos de rede virtual usando a função `parameters()` com ambos os índices de matriz do `[]` e o operador ponto.</span><span class="sxs-lookup"><span data-stu-id="a4343-118">The values of our `VNetSettings` object are applied to the properties required by our virtual network resource using the `parameters()` function with both the `[]` array indexer and the dot operator.</span></span> <span data-ttu-id="a4343-119">Essa abordagem funcionará se você simplesmente quiser aplicar estaticamente os valores do objeto do parâmetro ao recurso.</span><span class="sxs-lookup"><span data-stu-id="a4343-119">This approach works if you just want to statically apply the values of the parameter object to the resource.</span></span> <span data-ttu-id="a4343-120">No entanto, se você deseja atribuir uma matriz de valores de propriedade dinamicamente durante a implantação usar um [loop de cópia][azure-resource-manager-create-multiple-instances].</span><span class="sxs-lookup"><span data-stu-id="a4343-120">However, if you want to dynamically assign an array of property values during deployment you can use a [copy loop][azure-resource-manager-create-multiple-instances].</span></span> <span data-ttu-id="a4343-121">Para usar um loop de cópia, você fornece uma matriz JSON dos valores de propriedade de recurso e o loop de cópia aplica dinamicamente os valores às propriedades do recurso.</span><span class="sxs-lookup"><span data-stu-id="a4343-121">To use a copy loop, you provide a JSON array of resource property values and the copy loop dynamically applies the values to the resource's properties.</span></span> 

<span data-ttu-id="a4343-122">Há um problema do qual você deve estar ciente caso use a abordagem dinâmica.</span><span class="sxs-lookup"><span data-stu-id="a4343-122">There is one issue to be aware of if you use the dynamic approach.</span></span> <span data-ttu-id="a4343-123">Para demonstrar o problema, vamos dar uma olhada em uma matriz típica de valores de propriedade.</span><span class="sxs-lookup"><span data-stu-id="a4343-123">To demonstrate the issue, let's take a look at a typical array of property values.</span></span> <span data-ttu-id="a4343-124">Neste exemplo, os valores das nossas propriedades são armazenados em uma variável.</span><span class="sxs-lookup"><span data-stu-id="a4343-124">In this example the values for our properties are stored in a variable.</span></span> <span data-ttu-id="a4343-125">Observe que temos duas matrizes aqui&mdash;uma chamada `firstProperty` e outra chamada `secondProperty`.</span><span class="sxs-lookup"><span data-stu-id="a4343-125">Notice we have two arrays here&mdash;one named `firstProperty` and one named `secondProperty`.</span></span> 

```json
"variables": {
    "firstProperty": [
        {
            "name": "A",
            "type": "typeA"
        },
        {
            "name": "B",
            "type": "typeB"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ],
    "secondProperty": [
        "one","two", "three"
    ]
}
```

<span data-ttu-id="a4343-126">Agora vamos dar uma olhada da maneira que podemos acessar as propriedades na variável usando um loop de cópia.</span><span class="sxs-lookup"><span data-stu-id="a4343-126">Now let's take a look at the way we access the properties in the variable using a copy loop.</span></span>

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    ...
    "copy": {
        "name": "copyLoop1",
        "count": "[length(variables('firstProperty'))]"
    },
    ...
    "properties": {
        "name": { "value": "[variables('firstProperty')[copyIndex()].name]" },
        "type": { "value": "[variables('firstProperty')[copyIndex()].type]" },
        "number": { "value": "[variables('secondProperty')[copyIndex()]]" }
    }
}
```

<span data-ttu-id="a4343-127">A função `copyIndex()` retorna a iteração atual do loop de cópia e podemos usá-lo como um índice em cada uma das duas matrizes simultaneamente.</span><span class="sxs-lookup"><span data-stu-id="a4343-127">The `copyIndex()` function returns the current iteration of the copy loop, and we use that as an index into each of the two arrays simultaneously.</span></span>

<span data-ttu-id="a4343-128">Isso funciona bem quando as duas matrizes têm o mesmo tamanho.</span><span class="sxs-lookup"><span data-stu-id="a4343-128">This works fine when the two arrays are the same length.</span></span> <span data-ttu-id="a4343-129">O problema ocorre se você tiver cometido um erro e as duas matrizes têm diferentes comprimentos&mdash;nesse caso, seu modelo falhará na validação durante a implantação.</span><span class="sxs-lookup"><span data-stu-id="a4343-129">The issue arises if you've made a mistake and the two arrays are different lengths&mdash;in this case your template will fail validation during deployment.</span></span> <span data-ttu-id="a4343-130">Você pode evitar esse problema ao incluir todas as suas propriedades em um único objeto, porque ele é muito mais fácil de identificar quando um valor estiver ausente.</span><span class="sxs-lookup"><span data-stu-id="a4343-130">You can avoid this issue by including all your properties in a single object, because it is much easier to see when a value is missing.</span></span> <span data-ttu-id="a4343-131">Por exemplo, vamos dar uma olhada em outro objeto de parâmetro na qual cada elemento da matriz `propertyObject` é a união das matrizes `firstProperty` e `secondProperty` anteriores.</span><span class="sxs-lookup"><span data-stu-id="a4343-131">For example, let's take a look another parameter object in which each element of the `propertyObject` array is the union of the `firstProperty` and `secondProperty` arrays from earlier.</span></span>

```json
"variables": {
    "propertyObject": [
        {
            "name": "A",
            "type": "typeA",
            "number": "one"
        },
        {
            "name": "B",
            "type": "typeB",
            "number": "two"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ]
}
```

<span data-ttu-id="a4343-132">Você notou o terceiro elemento na matriz?</span><span class="sxs-lookup"><span data-stu-id="a4343-132">Notice the third element in the array?</span></span> <span data-ttu-id="a4343-133">A propriedade `number` está ausente, mas será muito mais fácil de notar isso quando ela estiver ausente quando você estiver criando os valores de parâmetro dessa maneira.</span><span class="sxs-lookup"><span data-stu-id="a4343-133">It's missing the `number` property, but it's much easier to notice that you've missed it when you're authoring the parameter values this way.</span></span>

## <a name="using-a-property-object-in-a-copy-loop"></a><span data-ttu-id="a4343-134">Uso de um objeto de propriedade em um loop de cópia</span><span class="sxs-lookup"><span data-stu-id="a4343-134">Using a property object in a copy loop</span></span>

<span data-ttu-id="a4343-135">Essa abordagem é ainda mais útil quando combinada a [serial copy loop][azure-resource-manager-create-multiple], principalmente para implantar os recursos filhos.</span><span class="sxs-lookup"><span data-stu-id="a4343-135">This approach becomes even more useful when combined with the [serial copy loop][azure-resource-manager-create-multiple], particularly for deploying child resources.</span></span> 

<span data-ttu-id="a4343-136">Para demonstrar isso, vamos dar uma olhada em um modelo que implante um [grupo de segurança de rede (NSG)][nsg] com duas regras de segurança.</span><span class="sxs-lookup"><span data-stu-id="a4343-136">To demonstrate this, let's look at a template that deploys a [network security group (NSG)][nsg] with two security rules.</span></span> 

<span data-ttu-id="a4343-137">Primeiro, vamos dar uma olhada no nosso parâmetros.</span><span class="sxs-lookup"><span data-stu-id="a4343-137">First, let's take a look at our parameters.</span></span> <span data-ttu-id="a4343-138">Quando examinamos nosso modelo, veremos que definimos um parâmetro denominado `networkSecurityGroupsSettings`, que inclui uma matriz chamada `securityRules`.</span><span class="sxs-lookup"><span data-stu-id="a4343-138">When we look at our template we'll see that we've defined one parameter named `networkSecurityGroupsSettings` that includes an array named `securityRules`.</span></span> <span data-ttu-id="a4343-139">Essa matriz contém dois objetos JSON que especificam um número de configurações para uma regra de segurança.</span><span class="sxs-lookup"><span data-stu-id="a4343-139">This array contains two JSON objects that specify a number of settings for a security rule.</span></span>

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters":{ 
      "networkSecurityGroupsSettings": {
      "value": {
          "securityRules": [
            {
              "name": "RDPAllow",
              "description": "allow RDP connections",
              "direction": "Inbound",
              "priority": 100,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.0.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "access": "Allow",
              "protocol": "Tcp"
            },
            {
              "name": "HTTPAllow",
              "description": "allow HTTP connections",
              "direction": "Inbound",
              "priority": 200,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.1.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "80",
              "access": "Allow",
              "protocol": "Tcp"
            }
          ]
        }
      }
    }
  }
```

<span data-ttu-id="a4343-140">Agora vamos dar uma olhada no nosso modelo.</span><span class="sxs-lookup"><span data-stu-id="a4343-140">Now let's take a look at our template.</span></span> <span data-ttu-id="a4343-141">Nosso primeiro recurso chamado `NSG1` implanta o NSG.</span><span class="sxs-lookup"><span data-stu-id="a4343-141">Our first resource named `NSG1` deploys the NSG.</span></span> <span data-ttu-id="a4343-142">Nosso segundo recurso chamado `loop-0` executa duas funções: primeiro, ele usa `dependsOn` no NSG para que sua implantação só seja iniciada quando o `NSG1` for concluído e essa é a primeira iteração do loop sequencial.</span><span class="sxs-lookup"><span data-stu-id="a4343-142">Our second resource named `loop-0` performs two functions: first, it `dependsOn` the NSG so its deployment doesn't begin until `NSG1` is completed, and it is the first iteration of the sequential loop.</span></span> <span data-ttu-id="a4343-143">Nosso terceiro recurso é um modelo aninhado que implanta nossas regras de segurança usando um objeto para seus valores de parâmetro como no último exemplo.</span><span class="sxs-lookup"><span data-stu-id="a4343-143">Our third resource is a nested template that deploys our security rules using an object for its parameter values as in the last example.</span></span>

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "networkSecurityGroupsSettings": {"type":"object"}
  },
  "variables": {},
  "resources": [
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "NSG1",
      "location":"[resourceGroup().location]",
      "properties": {
          "securityRules":[]
      }
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "loop-0",
        "dependsOn": [
            "NSG1"
        ],
        "properties": {
            "mode":"Incremental",
            "parameters":{},
            "template": {
                "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                "contentVersion": "1.0.0.0",
                "parameters": {},
                "variables": {},
                "resources": [],
                "outputs": {}
            }
        }       
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "[concat('loop-', copyIndex(1))]",
        "dependsOn": [
          "[concat('loop-', copyIndex())]"
        ],
        "copy": {
          "name": "iterator",
          "count": "[length(parameters('networkSecurityGroupsSettings').securityRules)]"
        },
        "properties": {
          "mode": "Incremental",
          "template": {
            "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
           "parameters": {},
            "variables": {},
            "resources": [
                {
                    "name": "[concat('NSG1/' , parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].name)]",
                    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
                    "apiVersion": "2016-09-01",
                    "location":"[resourceGroup().location]",
                    "properties":{
                        "description": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].description]",
                        "priority":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].priority]",
                        "protocol":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].protocol]",
                        "sourcePortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourcePortRange]",
                        "destinationPortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationPortRange]",
                        "sourceAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourceAddressPrefix]",
                        "destinationAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationAddressPrefix]",
                        "access":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].access]",
                        "direction":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].direction]"
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

<span data-ttu-id="a4343-144">Vamos examinar mais detalhadamente como podemos especificar os valores de propriedade no recurso filho `securityRules`.</span><span class="sxs-lookup"><span data-stu-id="a4343-144">Let's take a closer look at how we specify our property values in the `securityRules` child resource.</span></span> <span data-ttu-id="a4343-145">Todas as nossas propriedades são referenciadas usando a função `parameter()` e, em seguida, use o operador ponto para fazer referência à nossa matriz `securityRules` indexada pelo valor atual da iteração.</span><span class="sxs-lookup"><span data-stu-id="a4343-145">All of our properties are referenced using the `parameter()` function, and then we use the dot operator to reference our `securityRules` array, indexed by the current value of the iteration.</span></span> <span data-ttu-id="a4343-146">Por fim, usamos outro operador de ponto para fazer referência ao nome do objeto.</span><span class="sxs-lookup"><span data-stu-id="a4343-146">Finally, we use another dot operator to reference the name of the object.</span></span> 

## <a name="try-the-template"></a><span data-ttu-id="a4343-147">Experimentar o modelo</span><span class="sxs-lookup"><span data-stu-id="a4343-147">Try the template</span></span>

<span data-ttu-id="a4343-148">Para experimentar esse modelo, siga estas etapas:</span><span class="sxs-lookup"><span data-stu-id="a4343-148">If you would like to experiment with this template, follow these steps:</span></span> 

1.  <span data-ttu-id="a4343-149">Visite o portal do Azure, selecione o ícone **+** e pesquise pelo tipo de recurso **implantação de modelo** e selecione-o.</span><span class="sxs-lookup"><span data-stu-id="a4343-149">Go to the Azure portal, select the **+** icon, and search for the **template deployment** resource type, and select it.</span></span>
2.  <span data-ttu-id="a4343-150">Quando chegar à página **Implantação de modelo**, selecione o botão **criar**.</span><span class="sxs-lookup"><span data-stu-id="a4343-150">Navigate to the **template deployment** page, select the **create** button.</span></span> <span data-ttu-id="a4343-151">Esse botão abre a folha **Implantação personalizada**.</span><span class="sxs-lookup"><span data-stu-id="a4343-151">This button opens the **custom deployment** blade.</span></span>
3.  <span data-ttu-id="a4343-152">Selecione o botão **Editar modelo**.</span><span class="sxs-lookup"><span data-stu-id="a4343-152">Select the **edit template** button.</span></span>
4.  <span data-ttu-id="a4343-153">Exclua o modelo vazio.</span><span class="sxs-lookup"><span data-stu-id="a4343-153">Delete the empty template.</span></span> 
5.  <span data-ttu-id="a4343-154">Copie e cole o modelo de exemplo no painel direito.</span><span class="sxs-lookup"><span data-stu-id="a4343-154">Copy and paste the sample template into the right pane.</span></span>
6.  <span data-ttu-id="a4343-155">Selecione o botão **Salvar**.</span><span class="sxs-lookup"><span data-stu-id="a4343-155">Select the **save** button.</span></span>
7.  <span data-ttu-id="a4343-156">Quando retornar ao painel de **implantação personalizada**, selecione o botão **Editar parâmetros**.</span><span class="sxs-lookup"><span data-stu-id="a4343-156">When you are returned to the **custom deployment** pane, select the **edit parameters** button.</span></span>
8.  <span data-ttu-id="a4343-157">Na folha **editar parâmetros**, exclua o modelo existente.</span><span class="sxs-lookup"><span data-stu-id="a4343-157">On the **edit parameters** blade, delete the existing template.</span></span>
9.  <span data-ttu-id="a4343-158">Copie e cole o modelo de parâmetro de exemplo acima.</span><span class="sxs-lookup"><span data-stu-id="a4343-158">Copy and paste the sample parameter template from above.</span></span>
10. <span data-ttu-id="a4343-159">Selecione o botão **Salvar**, que fará com que você retorne à folha **implantação personalizada**.</span><span class="sxs-lookup"><span data-stu-id="a4343-159">Select the **save** button, which returns you to the **custom deployment** blade.</span></span>
11. <span data-ttu-id="a4343-160">Na folha **Implantação personalizada**, selecione sua assinatura, crie um novo grupo de recursos ou use um existente e selecione uma localização.</span><span class="sxs-lookup"><span data-stu-id="a4343-160">On the **custom deployment** blade, select your subscription, either create new or use existing resource group, and select a location.</span></span> <span data-ttu-id="a4343-161">Examine os termos e condições e selecione a caixa de seleção **Eu concordo**.</span><span class="sxs-lookup"><span data-stu-id="a4343-161">Review the terms and conditions, and select the **I agree** checkbox.</span></span>
12. <span data-ttu-id="a4343-162">Selecione o botão **Comprar**.</span><span class="sxs-lookup"><span data-stu-id="a4343-162">Select the **purchase** button.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a4343-163">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="a4343-163">Next steps</span></span>

* <span data-ttu-id="a4343-164">Você pode expandir essas técnicas para implementar um [transformador de objeto de propriedade e um coletor](./collector.md).</span><span class="sxs-lookup"><span data-stu-id="a4343-164">You can expand upon these techniques to implement a [property object transformer and collector](./collector.md).</span></span> <span data-ttu-id="a4343-165">As técnicas de transformador e coletor são mais gerais e podem ser vinculadas de seus modelos.</span><span class="sxs-lookup"><span data-stu-id="a4343-165">The transformer and collector techniques are more general and can be linked from your templates.</span></span>
* <span data-ttu-id="a4343-166">Esse padrão também é implementado no [projeto de blocos de construção do modelo](https://github.com/mspnp/template-building-blocks) e nas [arquiteturas de referência do Azure](/azure/architecture/reference-architectures/).</span><span class="sxs-lookup"><span data-stu-id="a4343-166">This technique is also implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="a4343-167">Você pode examinar os nossos modelos para ver como implementamos essa técnica.</span><span class="sxs-lookup"><span data-stu-id="a4343-167">You can review our templates to see how we've implemented this technique.</span></span>

<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg