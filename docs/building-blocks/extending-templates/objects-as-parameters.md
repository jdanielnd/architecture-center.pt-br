---
title: Usar um objeto como um parâmetro em um modelo do Azure Resource Manager
description: Descreve como estender a funcionalidade dos modelos do Azure Resource Manager para usar objetos como parâmetros
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: c1955823b3474efa0abea1d9634add5f13d02eda
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251882"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a><span data-ttu-id="b8360-103">Usar um objeto como um parâmetro em um modelo do Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="b8360-103">Use an object as a parameter in an Azure Resource Manager template</span></span>

<span data-ttu-id="b8360-104">Quando você [criar modelos do Azure Resource Manager][azure-resource-manager-create-template], poderá especificar valores de propriedade de recurso diretamente no modelo ou definir um parâmetro e fornecer valores de durante a implantação.</span><span class="sxs-lookup"><span data-stu-id="b8360-104">When you [author Azure Resource Manager templates][azure-resource-manager-create-template], you can either specify resource property values directly in the template or define a parameter and provide values during deployment.</span></span> <span data-ttu-id="b8360-105">Você pode usar um parâmetro para cada valor de propriedade para implantações pequenas, mas há um limite de 255 parâmetros por implantação.</span><span class="sxs-lookup"><span data-stu-id="b8360-105">It's fine to use a parameter for each property value for small deployments, but there is a limit of 255 parameters per deployment.</span></span> <span data-ttu-id="b8360-106">Depois de obter para implantações maiores e mais complexas, você poderá ficar sem parâmetros.</span><span class="sxs-lookup"><span data-stu-id="b8360-106">Once you get to larger and more complex deployments you may run out of parameters.</span></span>

<span data-ttu-id="b8360-107">Uma maneira de resolver esse problema é usar um objeto como parâmetro em vez de um valor.</span><span class="sxs-lookup"><span data-stu-id="b8360-107">One way to solve this problem is to use an object as a parameter instead of a value.</span></span> <span data-ttu-id="b8360-108">Para fazer isso, defina o parâmetro no modelo e especifique um objeto JSON em vez de um único valor durante a implantação.</span><span class="sxs-lookup"><span data-stu-id="b8360-108">To do this, define the parameter in your template and specify a JSON object instead of a single value during deployment.</span></span> <span data-ttu-id="b8360-109">Em seguida, faça referência às subpropriedades do parâmetro usando a [`parameter()`função][azure-resource-manager-functions] e o operador ponto no seu modelo.</span><span class="sxs-lookup"><span data-stu-id="b8360-109">Then, reference the subproperties of the parameter using the [`parameter()` function][azure-resource-manager-functions] and dot operator in your template.</span></span>

<span data-ttu-id="b8360-110">Vamos dar uma olhada em um exemplo que implanta um recurso de rede virtual.</span><span class="sxs-lookup"><span data-stu-id="b8360-110">Let's take a look at an example that deploys a virtual network resource.</span></span> <span data-ttu-id="b8360-111">Primeiro, vamos especificar um parâmetro `VNetSettings` em nosso modelo e defina o `type` como `object`:</span><span class="sxs-lookup"><span data-stu-id="b8360-111">First, let's specify a `VNetSettings` parameter in our template and set the `type` to `object`:</span></span>

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
<span data-ttu-id="b8360-112">Em seguida, vamos fornecer valores para o objeto `VNetSettings`:</span><span class="sxs-lookup"><span data-stu-id="b8360-112">Next, let's provide values for the `VNetSettings` object:</span></span>

> [!NOTE]
> <span data-ttu-id="b8360-113">Para saber como fornecer valores de parâmetro durante a implantação, veja a seção **Parâmetros** do artigo [Noções básicas sobre a estrutura e a sintaxe dos modelos do Azure Resource Manager][azure-resource-manager-authoring-templates].</span><span class="sxs-lookup"><span data-stu-id="b8360-113">To learn how to provide parameter values during deployment, see the **parameters** section of [understand the structure and syntax of Azure Resource Manager templates][azure-resource-manager-authoring-templates].</span></span> 

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

<span data-ttu-id="b8360-114">Como você pode ver, nosso único parâmetro na verdade especifica três subpropriedades: `name`, `addressPrefixes` e `subnets`.</span><span class="sxs-lookup"><span data-stu-id="b8360-114">As you can see, our single parameter actually specifies three subproperties: `name`, `addressPrefixes`, and `subnets`.</span></span> <span data-ttu-id="b8360-115">Cada uma desses subpropriedades especifica um valor ou outras subpropriedades.</span><span class="sxs-lookup"><span data-stu-id="b8360-115">Each of these subproperties either specifies a value or other subproperties.</span></span> <span data-ttu-id="b8360-116">O resultado é que o nosso único parâmetro especifica todos os valores necessários para implantar a rede virtual.</span><span class="sxs-lookup"><span data-stu-id="b8360-116">The result is that our single parameter specifies all the values necessary to deploy our virtual network.</span></span>

<span data-ttu-id="b8360-117">Agora vamos dar uma olhada no restante do nosso modelo para ver como o objeto `VNetSettings` é usado:</span><span class="sxs-lookup"><span data-stu-id="b8360-117">Now let's have a look at the rest of our template to see how the `VNetSettings` object is used:</span></span>

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
<span data-ttu-id="b8360-118">Os valores de nosso objeto `VNetSettings` são aplicadas para as propriedades solicitadas por nossos recursos de rede virtual usando a função `parameters()` com ambos os índices de matriz do `[]` e o operador ponto.</span><span class="sxs-lookup"><span data-stu-id="b8360-118">The values of our `VNetSettings` object are applied to the properties required by our virtual network resource using the `parameters()` function with both the `[]` array indexer and the dot operator.</span></span> <span data-ttu-id="b8360-119">Essa abordagem funcionará se você simplesmente quiser aplicar estaticamente os valores do objeto do parâmetro ao recurso.</span><span class="sxs-lookup"><span data-stu-id="b8360-119">This approach works if you just want to statically apply the values of the parameter object to the resource.</span></span> <span data-ttu-id="b8360-120">No entanto, se você deseja atribuir uma matriz de valores de propriedade dinamicamente durante a implantação usar um [loop de cópia][azure-resource-manager-create-multiple-instances].</span><span class="sxs-lookup"><span data-stu-id="b8360-120">However, if you want to dynamically assign an array of property values during deployment you can use a [copy loop][azure-resource-manager-create-multiple-instances].</span></span> <span data-ttu-id="b8360-121">Para usar um loop de cópia, você fornece uma matriz JSON dos valores de propriedade de recurso e o loop de cópia aplica dinamicamente os valores às propriedades do recurso.</span><span class="sxs-lookup"><span data-stu-id="b8360-121">To use a copy loop, you provide a JSON array of resource property values and the copy loop dynamically applies the values to the resource's properties.</span></span> 

<span data-ttu-id="b8360-122">Há um problema do qual você deve estar ciente caso use a abordagem dinâmica.</span><span class="sxs-lookup"><span data-stu-id="b8360-122">There is one issue to be aware of if you use the dynamic approach.</span></span> <span data-ttu-id="b8360-123">Para demonstrar o problema, vamos dar uma olhada em uma matriz típica de valores de propriedade.</span><span class="sxs-lookup"><span data-stu-id="b8360-123">To demonstrate the issue, let's take a look at a typical array of property values.</span></span> <span data-ttu-id="b8360-124">Neste exemplo, os valores das nossas propriedades são armazenados em uma variável.</span><span class="sxs-lookup"><span data-stu-id="b8360-124">In this example the values for our properties are stored in a variable.</span></span> <span data-ttu-id="b8360-125">Observe que temos duas matrizes aqui&mdash;uma chamada `firstProperty` e outra chamada `secondProperty`.</span><span class="sxs-lookup"><span data-stu-id="b8360-125">Notice we have two arrays here&mdash;one named `firstProperty` and one named `secondProperty`.</span></span> 

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

<span data-ttu-id="b8360-126">Agora vamos dar uma olhada da maneira que podemos acessar as propriedades na variável usando um loop de cópia.</span><span class="sxs-lookup"><span data-stu-id="b8360-126">Now let's take a look at the way we access the properties in the variable using a copy loop.</span></span>

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

<span data-ttu-id="b8360-127">A função `copyIndex()` retorna a iteração atual do loop de cópia e podemos usá-lo como um índice em cada uma das duas matrizes simultaneamente.</span><span class="sxs-lookup"><span data-stu-id="b8360-127">The `copyIndex()` function returns the current iteration of the copy loop, and we use that as an index into each of the two arrays simultaneously.</span></span>

<span data-ttu-id="b8360-128">Isso funciona bem quando as duas matrizes têm o mesmo tamanho.</span><span class="sxs-lookup"><span data-stu-id="b8360-128">This works fine when the two arrays are the same length.</span></span> <span data-ttu-id="b8360-129">O problema ocorre se você tiver cometido um erro e as duas matrizes têm diferentes comprimentos&mdash;nesse caso, seu modelo falhará na validação durante a implantação.</span><span class="sxs-lookup"><span data-stu-id="b8360-129">The issue arises if you've made a mistake and the two arrays are different lengths&mdash;in this case your template will fail validation during deployment.</span></span> <span data-ttu-id="b8360-130">Você pode evitar esse problema ao incluir todas as suas propriedades em um único objeto, porque ele é muito mais fácil de identificar quando um valor estiver ausente.</span><span class="sxs-lookup"><span data-stu-id="b8360-130">You can avoid this issue by including all your properties in a single object, because it is much easier to see when a value is missing.</span></span> <span data-ttu-id="b8360-131">Por exemplo, vamos dar uma olhada em outro objeto de parâmetro na qual cada elemento da matriz `propertyObject` é a união das matrizes `firstProperty` e `secondProperty` anteriores.</span><span class="sxs-lookup"><span data-stu-id="b8360-131">For example, let's take a look another parameter object in which each element of the `propertyObject` array is the union of the `firstProperty` and `secondProperty` arrays from earlier.</span></span>

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

<span data-ttu-id="b8360-132">Você notou o terceiro elemento na matriz?</span><span class="sxs-lookup"><span data-stu-id="b8360-132">Notice the third element in the array?</span></span> <span data-ttu-id="b8360-133">A propriedade `number` está ausente, mas será muito mais fácil de notar isso quando ela estiver ausente quando você estiver criando os valores de parâmetro dessa maneira.</span><span class="sxs-lookup"><span data-stu-id="b8360-133">It's missing the `number` property, but it's much easier to notice that you've missed it when you're authoring the parameter values this way.</span></span>

## <a name="using-a-property-object-in-a-copy-loop"></a><span data-ttu-id="b8360-134">Uso de um objeto de propriedade em um loop de cópia</span><span class="sxs-lookup"><span data-stu-id="b8360-134">Using a property object in a copy loop</span></span>

<span data-ttu-id="b8360-135">Essa abordagem é ainda mais útil quando combinada a [serial copy loop][azure-resource-manager-create-multiple], principalmente para implantar os recursos filhos.</span><span class="sxs-lookup"><span data-stu-id="b8360-135">This approach becomes even more useful when combined with the [serial copy loop][azure-resource-manager-create-multiple], particularly for deploying child resources.</span></span> 

<span data-ttu-id="b8360-136">Para demonstrar isso, vamos dar uma olhada em um modelo que implante um [grupo de segurança de rede (NSG)][nsg] com duas regras de segurança.</span><span class="sxs-lookup"><span data-stu-id="b8360-136">To demonstrate this, let's look at a template that deploys a [network security group (NSG)][nsg] with two security rules.</span></span> 

<span data-ttu-id="b8360-137">Primeiro, vamos dar uma olhada no nosso parâmetros.</span><span class="sxs-lookup"><span data-stu-id="b8360-137">First, let's take a look at our parameters.</span></span> <span data-ttu-id="b8360-138">Quando examinamos nosso modelo, veremos que definimos um parâmetro denominado `networkSecurityGroupsSettings`, que inclui uma matriz chamada `securityRules`.</span><span class="sxs-lookup"><span data-stu-id="b8360-138">When we look at our template we'll see that we've defined one parameter named `networkSecurityGroupsSettings` that includes an array named `securityRules`.</span></span> <span data-ttu-id="b8360-139">Essa matriz contém dois objetos JSON que especificam um número de configurações para uma regra de segurança.</span><span class="sxs-lookup"><span data-stu-id="b8360-139">This array contains two JSON objects that specify a number of settings for a security rule.</span></span>

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

<span data-ttu-id="b8360-140">Agora vamos dar uma olhada no nosso modelo.</span><span class="sxs-lookup"><span data-stu-id="b8360-140">Now let's take a look at our template.</span></span> <span data-ttu-id="b8360-141">Nosso primeiro recurso chamado `NSG1` implanta o NSG.</span><span class="sxs-lookup"><span data-stu-id="b8360-141">Our first resource named `NSG1` deploys the NSG.</span></span> <span data-ttu-id="b8360-142">Nosso segundo recurso chamado `loop-0` executa duas funções: primeiro, ele usa `dependsOn` no NSG para que sua implantação só seja iniciada quando o `NSG1` for concluído e essa é a primeira iteração do loop sequencial.</span><span class="sxs-lookup"><span data-stu-id="b8360-142">Our second resource named `loop-0` performs two functions: first, it `dependsOn` the NSG so its deployment doesn't begin until `NSG1` is completed, and it is the first iteration of the sequential loop.</span></span> <span data-ttu-id="b8360-143">Nosso terceiro recurso é um modelo aninhado que implanta nossas regras de segurança usando um objeto para seus valores de parâmetro como no último exemplo.</span><span class="sxs-lookup"><span data-stu-id="b8360-143">Our third resource is a nested template that deploys our security rules using an object for its parameter values as in the last example.</span></span>

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
                "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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

<span data-ttu-id="b8360-144">Vamos examinar mais detalhadamente como podemos especificar os valores de propriedade no recurso filho `securityRules`.</span><span class="sxs-lookup"><span data-stu-id="b8360-144">Let's take a closer look at how we specify our property values in the `securityRules` child resource.</span></span> <span data-ttu-id="b8360-145">Todas as nossas propriedades são referenciadas usando a função `parameter()` e, em seguida, use o operador ponto para fazer referência à nossa matriz `securityRules` indexada pelo valor atual da iteração.</span><span class="sxs-lookup"><span data-stu-id="b8360-145">All of our properties are referenced using the `parameter()` function, and then we use the dot operator to reference our `securityRules` array, indexed by the current value of the iteration.</span></span> <span data-ttu-id="b8360-146">Por fim, usamos outro operador de ponto para fazer referência ao nome do objeto.</span><span class="sxs-lookup"><span data-stu-id="b8360-146">Finally, we use another dot operator to reference the name of the object.</span></span> 

## <a name="try-the-template"></a><span data-ttu-id="b8360-147">Experimentar o modelo</span><span class="sxs-lookup"><span data-stu-id="b8360-147">Try the template</span></span>

<span data-ttu-id="b8360-148">Um modelo de exemplo está disponível no [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="b8360-148">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="b8360-149">Para implantar o modelo, clone o repositório e execute os seguintes comandos da [CLI do Azure][cli]:</span><span class="sxs-lookup"><span data-stu-id="b8360-149">To deploy the template, clone the repo and run the following [Azure CLI][cli] commands:</span></span>

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example3-object-param
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example3-object-param/deploy.json \
    --parameters deploy.parameters.json
```

## <a name="next-steps"></a><span data-ttu-id="b8360-150">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="b8360-150">Next steps</span></span>

- <span data-ttu-id="b8360-151">Saiba como criar um modelo que itera por meio de uma matriz de objetos e os transforma em um esquema JSON.</span><span class="sxs-lookup"><span data-stu-id="b8360-151">Learn how to create a template that iterates through an object array and transforms it into a JSON schema.</span></span> <span data-ttu-id="b8360-152">Confira [Implementar um transformador de propriedade e um coletor em um modelo do Azure Resource Manager](./collector.md)</span><span class="sxs-lookup"><span data-stu-id="b8360-152">See [Implement a property transformer and collector in an Azure Resource Manager template](./collector.md)</span></span>


<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
