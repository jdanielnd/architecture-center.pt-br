---
title: Implantar condicionalmente um recurso em um modelo do Azure Resource Manager
description: "Descreve como estender a funcionalidade de modelos do Azure Resource Manager para implantar condicionalmente um recurso dependendo do valor de um parâmetro"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: e911e7dc41b4f71ebfaf13a00f8cdbb5b4e2578b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a>Implantar condicionalmente um recurso em um modelo do Azure Resource Manager

Existem alguns cenários em que você precisa projetar seu modelo para implantar um recurso baseado em uma condição, como um valor de parâmetro estar ou não presente. Por exemplo, seu modelo pode implantar uma rede virtual e incluir parâmetros para especificar outras redes virtuais para emparelhamento. Se não tiver especificado nenhum valor de parâmetro para o emparelhamento, você não deseja que o Gerenciador de Recursos implante o recurso de emparelhamento.

Para isso, use o [`condition` elemento][azure-resource-manager-condition] no recurso para testar o tamanho da sua matriz de parâmetro. Se o comprimento for zero, retorne `false` para impedir a implantação, mas para todos os valores maiores que zero, retornar `true` para permitir a implantação.

## <a name="example-template"></a>Modelo de exemplo

Vamos examinar um modelo de exemplo que demonstra isso. Nosso modelo usa o [`condition` elemento][azure-resource-manager-condition] para controlar a implantação do recurso `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`. Esse recurso cria um emparelhamento entre duas Redes Virtuais do Azure na mesma região.

Vamos observar cada seção do modelo.

O elemento `parameters` define um parâmetro único chamado `virtualNetworkPeerings`: 

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
Nosso parâmetro `virtualNetworkPeerings` é uma `array` e tem o seguinte esquema:

```json
"virtualNetworkPeerings": [
    {
        "remoteVirtualNetwork": {
            "name": "my-other-virtual-network"
        },
        "allowForwardedTraffic": true,
        "allowGatewayTransit": true,
        "useRemoteGateways": false
    }
]
```

As propriedades no nosso parâmetro especificam as [configurações relacionadas às redes virtuais de emparelhamento][vnet-peering-resource-schema]. Vamos fornecer os valores para essas propriedades quando especificamos o recurso `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` na seção de `resources`:

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "virtualNetworks"
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
Há algumas coisas acontecendo nessa parte do nosso modelo. Primeiro, o recurso real sendo implantado é um modelo embutido do tipo `Microsoft.Resources/deployments` que inclui seu próprio modelo que implanta o `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` de fato.

Nosso `name` para o modelo embutido torna-se exclusivo ao se concatenar a iteração atual do `copyIndex()` com o prefixo `vnp-`. 

O elemento `condition` especifica que nosso recurso deve ser processado quando a função `greater()` é avaliada como `true`. Aqui estamos testando se a matriz do parâmetro `virtualNetworkPeerings` é `greater()` que zero. Se positivo, ela é avaliada como `true`, e a `condition` é atendida. Caso contrário, é `false`.

Em seguida, especificamos nosso loop de `copy`. Trata-se de um loop `serial` que significa que o loop é feito em sequência, com cada recurso aguardando até que o último recurso seja implantado. A propriedade `count` especifica a quantidade de vezes que o loop se repete. Aqui, normalmente nós a definiríamos conforme o tamanho da matriz `virtualNetworkPeerings` porque ela contém os objetos de parâmetro que especificam o recurso que queremos implantar. No entanto, se fizermos isso, a validação falhará se a matriz estiver vazia, porque o Gerenciador de Recursos percebe que estamos tentando acessar propriedades que não existem. Mas é possível contornar isso. Vejamos as variáveis de que precisaremos:

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

Nossa variável `workaround` inclui duas propriedades, uma nomeada como `true` e outra nomeada como `false`. A propriedade `true` é avaliada como o valor da matriz de parâmetros `virtualNetworkPeerings`. A propriedade `false` é avaliada como um objeto vazio, incluindo as propriedades nomeadas que o Gerenciador de Recursos espera ver &mdash; observe que `false` é, na verdade, uma matriz, assim como nosso parâmetro `virtualNetworkPeerings`, o que satisfaz a validação. 

A variável `peerings` usa nossa variável `workaround` mais uma vez testando se o comprimento da matriz de parâmetros `virtualNetworkPeerings` é maior que zero. Em caso positivo, `string` é avaliada como `true` e a variável `workaround` é avaliada como a matriz de parâmetros `virtualNetworkPeerings`. Caso contrário, será avaliada como `false`, e a variável `workaround` é avaliada como nosso objeto vazio no primeiro elemento da matriz.

Agora que contornamos o problema de validação, podemos simplesmente especificar a implantação do recursos `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` no modelo aninhado, passando o `name` e as `properties` da nossa matriz de parâmetros `virtualNetworkPeerings`. Você pode ver isso no elemento `template` aninhado no elemento `properties` do nosso recurso.

## <a name="next-steps"></a>Próximas etapas

* Essa técnica também é implementada no [projeto de blocos de construção do modelo](https://github.com/mspnp/template-building-blocks) e nas [arquiteturas de referência do Azure](/azure/architecture/reference-architectures/). Você pode usá-los para criar sua própria arquitetura ou implantar uma de nossas arquiteturas de referência.

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-group-authoring-templates#resources
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings