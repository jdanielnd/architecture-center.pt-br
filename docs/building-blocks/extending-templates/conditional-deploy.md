---
title: Implantar condicionalmente um recurso em um modelo do Azure Resource Manager
description: Descreve como estender a funcionalidade de modelos do Azure Resource Manager para implantar condicionalmente um recurso dependendo do valor de um parâmetro.
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: 0e02fbbd130bd6be2fc10173c8466b028d5d61da
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113460"
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="c0153-103">Implantar condicionalmente um recurso em um modelo do Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="c0153-103">Conditionally deploy a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="c0153-104">Existem alguns cenários em que você precisa projetar seu modelo para implantar um recurso baseado em uma condição, como um valor de parâmetro estar ou não presente.</span><span class="sxs-lookup"><span data-stu-id="c0153-104">There are some scenarios in which you need to design your template to deploy a resource based on a condition, such as whether or not a parameter value is present.</span></span> <span data-ttu-id="c0153-105">Por exemplo, seu modelo pode implantar uma rede virtual e incluir parâmetros para especificar outras redes virtuais para emparelhamento.</span><span class="sxs-lookup"><span data-stu-id="c0153-105">For example, your template may deploy a virtual network and include parameters to specify other virtual networks for peering.</span></span> <span data-ttu-id="c0153-106">Se não tiver especificado nenhum valor de parâmetro para o emparelhamento, você não deseja que o Gerenciador de Recursos implante o recurso de emparelhamento.</span><span class="sxs-lookup"><span data-stu-id="c0153-106">If you've not specified any parameter values for peering, you don't want Resource Manager to deploy the peering resource.</span></span>

<span data-ttu-id="c0153-107">Para isso, use o [elemento condição][azure-resource-manager-condition] no recurso para testar o tamanho da sua matriz de parâmetro.</span><span class="sxs-lookup"><span data-stu-id="c0153-107">To accomplish this, use the [condition element][azure-resource-manager-condition] in the resource to test the length of your parameter array.</span></span> <span data-ttu-id="c0153-108">Se o comprimento for zero, retorne `false` para impedir a implantação, mas para todos os valores maiores que zero, retornar `true` para permitir a implantação.</span><span class="sxs-lookup"><span data-stu-id="c0153-108">If the length is zero, return `false` to prevent deployment, but for all values greater than zero return `true` to allow deployment.</span></span>

## <a name="example-template"></a><span data-ttu-id="c0153-109">Modelo de exemplo</span><span class="sxs-lookup"><span data-stu-id="c0153-109">Example template</span></span>

<span data-ttu-id="c0153-110">Vamos examinar um modelo de exemplo que demonstra isso.</span><span class="sxs-lookup"><span data-stu-id="c0153-110">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="c0153-111">Nosso modelo usa o [elemento condição][azure-resource-manager-condition] para controlar a implantação do recurso `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="c0153-111">Our template uses the [condition element][azure-resource-manager-condition] to control deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource.</span></span> <span data-ttu-id="c0153-112">Esse recurso cria um emparelhamento entre duas Redes Virtuais do Azure na mesma região.</span><span class="sxs-lookup"><span data-stu-id="c0153-112">This resource creates a peering between two Azure Virtual Networks in the same region.</span></span>

<span data-ttu-id="c0153-113">Vamos observar cada seção do modelo.</span><span class="sxs-lookup"><span data-stu-id="c0153-113">Let's take a look at each section of the template.</span></span>

<span data-ttu-id="c0153-114">O elemento `parameters` define um parâmetro único chamado `virtualNetworkPeerings`:</span><span class="sxs-lookup"><span data-stu-id="c0153-114">The `parameters` element defines a single parameter named `virtualNetworkPeerings`:</span></span>

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "virtualNetworkPeerings": {
      "type": "array",
      "defaultValue": []
    }
  },
```

<span data-ttu-id="c0153-115">Nosso parâmetro `virtualNetworkPeerings` é uma `array` e tem o seguinte esquema:</span><span class="sxs-lookup"><span data-stu-id="c0153-115">Our `virtualNetworkPeerings` parameter is an `array` and has the following schema:</span></span>

```json
"virtualNetworkPeerings": [
    {
      "name": "firstVNet/peering1",
      "properties": {
          "remoteVirtualNetwork": {
              "id": "[resourceId('Microsoft.Network/virtualNetworks','secondVNet')]"
          },
          "allowForwardedTraffic": true,
          "allowGatewayTransit": true,
          "useRemoteGateways": false
      }
    }
]
```

<span data-ttu-id="c0153-116">As propriedades no nosso parâmetro especificam as [configurações relacionadas às redes virtuais de emparelhamento][vnet-peering-resource-schema].</span><span class="sxs-lookup"><span data-stu-id="c0153-116">The properties in our parameter specify the [settings related to peering virtual networks][vnet-peering-resource-schema].</span></span> <span data-ttu-id="c0153-117">Vamos fornecer os valores para essas propriedades quando especificamos o recurso `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` na seção de `resources`:</span><span class="sxs-lookup"><span data-stu-id="c0153-117">We'll provide the values for these properties when we specify the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the `resources` section:</span></span>

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "firstVNet", "secondVNet"
      ],
      "copy": {
          "name": "iterator",
          "count": "[length(variables('peerings'))]",
          "mode": "serial"
      },
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
          },
          "variables": {
          },
          "resources": [
            {
              "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
              "apiVersion": "2016-06-01",
              "location": "[resourceGroup().location]",
              "name": "[variables('peerings')[copyIndex()].name]",
              "properties": "[variables('peerings')[copyIndex()].properties]"
            }
          ],
          "outputs": {
          }
        }
      }
    }
]
```

<span data-ttu-id="c0153-118">Há algumas coisas acontecendo nessa parte do nosso modelo.</span><span class="sxs-lookup"><span data-stu-id="c0153-118">There are a couple of things going on in this part of our template.</span></span> <span data-ttu-id="c0153-119">Primeiro, o recurso real sendo implantado é um modelo embutido do tipo `Microsoft.Resources/deployments` que inclui seu próprio modelo que implanta o `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` de fato.</span><span class="sxs-lookup"><span data-stu-id="c0153-119">First, the actual resource being deployed is an inline template of type `Microsoft.Resources/deployments` that includes its own template that actually deploys the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span></span>

<span data-ttu-id="c0153-120">Nosso `name` para o modelo embutido torna-se exclusivo ao se concatenar a iteração atual do `copyIndex()` com o prefixo `vnp-`.</span><span class="sxs-lookup"><span data-stu-id="c0153-120">Our `name` for the inline template is made unique by concatenating the current iteration of the `copyIndex()` with the prefix `vnp-`.</span></span>

<span data-ttu-id="c0153-121">O elemento `condition` especifica que nosso recurso deve ser processado quando a função `greater()` é avaliada como `true`.</span><span class="sxs-lookup"><span data-stu-id="c0153-121">The `condition` element specifies that our resource should be processed when the `greater()` function evaluates to `true`.</span></span> <span data-ttu-id="c0153-122">Aqui estamos testando se a matriz do parâmetro `virtualNetworkPeerings` é `greater()` que zero.</span><span class="sxs-lookup"><span data-stu-id="c0153-122">Here, we're testing if the `virtualNetworkPeerings` parameter array is `greater()` than zero.</span></span> <span data-ttu-id="c0153-123">Se positivo, ela é avaliada como `true`, e a `condition` é atendida.</span><span class="sxs-lookup"><span data-stu-id="c0153-123">If it is, it evaluates to `true` and the `condition` is satisfied.</span></span> <span data-ttu-id="c0153-124">Caso contrário, é `false`.</span><span class="sxs-lookup"><span data-stu-id="c0153-124">Otherwise, it's `false`.</span></span>

<span data-ttu-id="c0153-125">Em seguida, especificamos nosso loop de `copy`.</span><span class="sxs-lookup"><span data-stu-id="c0153-125">Next, we specify our `copy` loop.</span></span> <span data-ttu-id="c0153-126">Trata-se de um loop `serial` que significa que o loop é feito em sequência, com cada recurso aguardando até que o último recurso seja implantado.</span><span class="sxs-lookup"><span data-stu-id="c0153-126">It's a `serial` loop that means the loop is done in sequence, with each resource waiting until the last resource has been deployed.</span></span> <span data-ttu-id="c0153-127">A propriedade `count` especifica a quantidade de vezes que o loop se repete.</span><span class="sxs-lookup"><span data-stu-id="c0153-127">The `count` property specifies the number of times the loop iterates.</span></span> <span data-ttu-id="c0153-128">Aqui, normalmente nós a definiríamos conforme o tamanho da matriz `virtualNetworkPeerings` porque ela contém os objetos de parâmetro que especificam o recurso que queremos implantar.</span><span class="sxs-lookup"><span data-stu-id="c0153-128">Here, normally we'd set it to the length of the `virtualNetworkPeerings` array because it contains the parameter objects specifying the resource we want to deploy.</span></span> <span data-ttu-id="c0153-129">No entanto, se fizermos isso, a validação falhará se a matriz estiver vazia, porque o Gerenciador de Recursos percebe que estamos tentando acessar propriedades que não existem.</span><span class="sxs-lookup"><span data-stu-id="c0153-129">However, if we do that, validation will fail if the array is empty because Resource Manager notices that we are attempting to access properties that do not exist.</span></span> <span data-ttu-id="c0153-130">Mas é possível contornar isso.</span><span class="sxs-lookup"><span data-stu-id="c0153-130">We can work around this, however.</span></span> <span data-ttu-id="c0153-131">Vejamos as variáveis de que precisaremos:</span><span class="sxs-lookup"><span data-stu-id="c0153-131">Let's take a look at the variables we'll need:</span></span>

```json
  "variables": {
    "workaround": {
       "true": "[parameters('virtualNetworkPeerings')]",
       "false": [{
           "name": "workaround",
           "properties": {}
       }]
     },
     "peerings": "[variables('workaround')[string(greater(length(parameters('virtualNetworkPeerings')), 0))]]"
  },
```

<span data-ttu-id="c0153-132">Nossa variável `workaround` inclui duas propriedades, uma nomeada como `true` e outra nomeada como `false`.</span><span class="sxs-lookup"><span data-stu-id="c0153-132">Our `workaround` variable includes two properties, one named `true` and one named `false`.</span></span> <span data-ttu-id="c0153-133">A propriedade `true` é avaliada como o valor da matriz de parâmetros `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="c0153-133">The `true` property evaluates to the value of the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="c0153-134">A propriedade `false` é avaliada como um objeto vazio, incluindo as propriedades nomeadas que o Gerenciador de Recursos espera ver &mdash;. Observe que `false` é, na verdade, uma matriz, assim como nosso parâmetro `virtualNetworkPeerings`, o que satisfaz a validação.</span><span class="sxs-lookup"><span data-stu-id="c0153-134">The `false` property evaluates to an empty object including the named properties that Resource Manager expects to see &mdash; note that `false` is actually an array, just as our `virtualNetworkPeerings` parameter is, which will satisfy validation.</span></span>

<span data-ttu-id="c0153-135">A variável `peerings` usa nossa variável `workaround` mais uma vez testando se o comprimento da matriz de parâmetros `virtualNetworkPeerings` é maior que zero.</span><span class="sxs-lookup"><span data-stu-id="c0153-135">Our `peerings` variable uses our `workaround` variable by once again testing if the length of the `virtualNetworkPeerings` parameter array is greater than zero.</span></span> <span data-ttu-id="c0153-136">Em caso positivo, a `string` é avaliada como `true` e a variável `workaround` é avaliada como a matriz do parâmetro `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="c0153-136">If it is, the `string` evaluates to `true` and the `workaround` variable evaluates to the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="c0153-137">Caso contrário, será avaliada como `false`, e a variável `workaround` é avaliada como nosso objeto vazio no primeiro elemento da matriz.</span><span class="sxs-lookup"><span data-stu-id="c0153-137">Otherwise, it evaluates to `false` and the `workaround` variable evaluates to our empty object in the first element of the array.</span></span>

<span data-ttu-id="c0153-138">Agora que contornamos o problema de validação, podemos simplesmente especificar a implantação do recursos `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` no modelo aninhado, passando o `name` e as `properties` da nossa matriz de parâmetros `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="c0153-138">Now that we've worked around the validation issue, we can simply specify the deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the nested template, passing the `name` and `properties` from our `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="c0153-139">Você pode ver isso no elemento `template` aninhado no elemento `properties` do nosso recurso.</span><span class="sxs-lookup"><span data-stu-id="c0153-139">You can see this in the `template` element nested in the `properties` element of our resource.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="c0153-140">Experimentar o modelo</span><span class="sxs-lookup"><span data-stu-id="c0153-140">Try the template</span></span>

<span data-ttu-id="c0153-141">Um modelo de exemplo está disponível no [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="c0153-141">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="c0153-142">Para implantar o modelo, execute os seguintes comandos da [CLI do Azure][cli]:</span><span class="sxs-lookup"><span data-stu-id="c0153-142">To deploy the template, run the following [Azure CLI][cli] commands:</span></span>

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example2-conditional/deploy.json
```

## <a name="next-steps"></a><span data-ttu-id="c0153-143">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="c0153-143">Next steps</span></span>

* <span data-ttu-id="c0153-144">Como parâmetros de modelo, use objetos em vez de valores escalares.</span><span class="sxs-lookup"><span data-stu-id="c0153-144">Use objects instead of scalar values as template parameters.</span></span> <span data-ttu-id="c0153-145">Confira [Usar um objeto como um parâmetro em um modelo do Azure Resource Manager](./objects-as-parameters.md)</span><span class="sxs-lookup"><span data-stu-id="c0153-145">See [Use an object as a parameter in an Azure Resource Manager template](./objects-as-parameters.md)</span></span>

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-manager-templates-resources#condition
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
