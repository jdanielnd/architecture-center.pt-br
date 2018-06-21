---
title: Implementar um transformador de propriedade e um coletor em um modelo do Azure Resource Manager
description: Descreve como implementar um transformador de propriedade e um coletor em um modelo do Azure Resource Manager
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 893779e652b845b3d936d11936dc767ef632fa43
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538658"
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a>Implementar um transformador de propriedade e um coletor em um modelo do Azure Resource Manager

Em [usar um objeto como um parâmetro em um modelo do Azure Resource Manager][objects-as-parameters], você aprendeu como armazenar valores de propriedade de recurso em um objeto e então aplicá-los a um recurso durante a implantação. Embora essa seja uma maneira muito útil para gerenciar os parâmetros, ainda exige que você mapear as propriedades do objeto para propriedades de recursos sempre que você pode usá-lo em seu modelo.

Para resolver esse problema, você pode implementar um modelo de transformação e um coletor de propriedade que itere sua matriz de objeto e a transforme no esquema JSON esperado pelo recurso.

> [!IMPORTANT]
> Essa abordagem requer que você tenha uma compreensão profunda dos modelos e das funções do Resource Manager.

Vamos dar uma olhada em como é possível implementar um coletor de propriedade e um transformador com um exemplo que implanta um [grupo de segurança de rede (NSG)][nsg]. O diagrama a seguir mostra a relação entre os nossos modelos e nossos recursos dentro desses modelos:

![arquitetura do coletor de propriedade e do transformador](../_images/collector-transformer.png)

Nosso **modelo de chamada** inclui dois recursos:
* um link de modelo que invoca nosso **modelo de coletor**.
* o recurso NSG para implantar.

Nosso **modelo de coletor** inclui dois recursos:
* um recurso de **âncora**.
* um link de modelo que invoca o modelo de transformação em um loop de cópia.

Nosso **modelo de transformação** inclui um único recurso: um modelo vazio com uma variável que transforma nosso JSON `source` no esquema JSON esperado por nosso recurso NSG no **modelo principal**.

## <a name="parameter-object"></a>Objeto de parâmetro

Usaremos nosso objeto de parâmetro `securityRules` de [objetos como parâmetros][objects-as-parameters]. Nosso **modelo de transformação** transformará cada objeto na matriz de `securityRules` no esquema JSON esperado pelo recurso NSG em nosso **modelo de chamada**.

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

Vamos examinar nosso **modelo de transformação**.

## <a name="transform-template"></a>Modelo de Transformação

Nosso **modelo de transformação** inclui dois parâmetros que são passados do **modelo de coletor**: 
* `source` é um objeto que recebe um dos objetos de valor de propriedade da matriz de propriedade. Em nosso exemplo, cada objeto da matriz de `"securityRules"` será transmitido um de cada vez.
* `state` é uma matriz que recebe os resultados concatenados de todas as transformações anteriores. Esta é a coleção do JSON transformado.

Nossos parâmetros têm esta aparência:

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "source": { "type": "object" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
  },
```

Nosso modelo também define uma variável denominada `instance`. Ele executa a transformação real do nosso objeto `source` para o esquema JSON necessário:

```json
  "variables": {
    "instance": [
      {
        "name": "[parameters('source').name]",
        "properties":{
            "description": "[parameters('source').description]",
            "protocol": "[parameters('source').protocol]",
            "sourcePortRange": "[parameters('source').sourcePortRange]",
            "destinationPortRange": "[parameters('source').destinationPortRange]",
            "sourceAddressPrefix": "[parameters('source').sourceAddressPrefix]",
            "destinationAddressPrefix": "[parameters('source').destinationAddressPrefix]",
            "access": "[parameters('source').access]",
            "priority": "[parameters('source').priority]",
            "direction": "[parameters('source').direction]"            
        }
      }
    ]

  },
```

Por fim, o `output` de nosso modelo concatena as transformações coletadas de nosso parâmetro `state` com a transformação atual executada pelo nossa variável `instance`:

```json
  "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

Em seguida, vamos dar uma olhada no nosso **modelo de coletor** para ver como ele passa os valores de parâmetro.

## <a name="collector-template"></a>Modelo de coletor

Nosso **modelo de coletor** inclui três parâmetros:
* `source` é nossa matriz de objetos de parâmetro completa. Ele é passado pelo **modelo de chamada**. Ele tem o mesmo nome que o parâmetro `source` no nosso **modelo de transformação**, mas há uma diferença importante que você já deve ter notado: essa é a matriz completa, mas nós só passamos um elemento dessa matriz para o **modelo de transformação** por vez.
* `transformTemplateUri` é o URI do nosso **modelo de transformação**. Nós o estamos definindo como um parâmetro aqui para a reutilização do modelo.
* `state` é uma matriz inicialmente vazia que podemos passar para o nosso **modelo de transformação**. Quando o loop de cópia for concluído, ele armazenará a coleção de objetos de parâmetro transformados.

Nossos parâmetros têm esta aparência:

```json
  "parameters": {
    "source": { "type": "array" },
    "transformTemplateUri": { "type": "string" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
``` 

Em seguida, definimos uma variável chamada `count`. Seu valor é o comprimento da matriz de objetos de parâmetro `source`:

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

Como você pode suspeitar, nós o usamos para o número de iterações em nosso loop de cópia.

Agora vamos dar uma olhada em nossos recursos. Definimos dois recursos:
* `loop-0` é o recurso com base em zero para o loop de cópia.
* `loop-` é concatenado com o resultado da função `copyIndex(1)` para gerar um nome exclusivo baseado em iteração para nosso recurso, começando com `1`.

Nossos recursos têm esta aparência:

```json
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "loop-0",
      "properties": {
        "mode": "Incremental",
        "parameters": { },
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": { },
          "variables": { },
          "resources": [ ],
          "outputs": {
            "collection": {
              "type": "array",
              "value": "[parameters('state')]"
            }
          }
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "[concat('loop-', copyindex(1))]",
      "copy": {
        "name": "iterator",
        "count": "[variables('count')]",
        "mode": "serial"
      },
      "dependsOn": [
        "loop-0"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": { "uri": "[parameters('transformTemplateUri')]" },
        "parameters": {
          "source": { "value": "[parameters('source')[copyindex()]]" },
          "state": { "value": "[reference(concat('loop-', copyindex())).outputs.collection.value]" }
        }
      }
    }
  ],
```

Vamos examinar mais detalhadamente os parâmetros que estamos passando para nosso **modelo de transformação** no modelo aninhado. Você se lembra de que nosso parâmetro `source` passa o objeto atual na matriz de objetos de parâmetro `source`. O parâmetro `state` é onde ocorre a coleção, pois obtém a saída da iteração anterior do nosso loop de cópia&mdash;observe que a função `reference()` usa a função `copyIndex()` sem nenhum parâmetro para fazer referência a `name` de nosso objeto de modelo vinculado anterior&mdash;e passa a iteração atual.

Por fim, o `output` de nosso modelo retorna o `output` da última iteração de nosso **modelo de transformação**:

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```
Pode parecer contraintuitivo retornar o `output` da última iteração de nosso **modelo de transformação** para nosso **modelo de chamada** porque ele apareceu quando estávamos armazenando-o em nosso parâmetro `source`. No entanto, lembre-se de que é a última iteração do nosso **modelo de transformação** que contém a matriz completa de objetos de propriedade transformados e que é o que queremos retornar.

Por fim, vamos dar uma olhada em como chamar o **modelo de coletor** do nosso **modelo de chamada**.

## <a name="calling-template"></a>Modelo de chamada

Nosso **modelo de chamada** define um parâmetro único chamado `networkSecurityGroupsSettings`:

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

Em seguida, nosso modelo define uma variável denominada `collectorTemplateUri`:

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

Como se esperaria, esse é o URI para o **modelo de coletor** que será usado pelo nosso recurso de modelo vinculado:

```json
{
    "apiVersion": "2015-01-01",
    "name": "collector",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('linkedTemplateUri')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "source" : {"value": "[parameters('networkSecurityGroupsSettings').securityRules]"},
            "transformTemplateUri": { "value": "[uri(deployment().properties.templateLink.uri, 'transform.json')]"}
        }
    }
}
```

Passamos dois parâmetros para o **modelo de coletor**:
* `source` é a nossa matriz de objetos de propriedade. Em nosso exemplo, ele é nosso parâmetro `networkSecurityGroupsSettings`.
* `transformTemplateUri` é a variável que acabamos de definir com o URI do nosso **modelo de coletor**.

Por fim, nosso recurso `Microsoft.Network/networkSecurityGroups` atribui diretamente o `output` do `collector` vinculado o recurso de modelo para sua propriedade `securityRules`:

```json
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "networkSecurityGroup1",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": "[reference('firstResource').outputs.result.value]"
      }
    }
  ],
  "outputs": {
      "instance":{
          "type": "array",
          "value": "[reference('firstResource').outputs.result.value]"
      }

  }
```

## <a name="next-steps"></a>Próximas etapas

* Essa técnica também é implementada no [projeto de blocos de construção do modelo](https://github.com/mspnp/template-building-blocks) e nas [arquiteturas de referência do Azure](/azure/architecture/reference-architectures/). Você pode usá-los para criar sua própria arquitetura ou implantar uma de nossas arquiteturas de referência.

<!-- links -->
[objects-as-parameters]: ./objects-as-parameters.md
[resource-manager-linked-template]: /azure/azure-resource-manager/resource-group-linked-templates
[resource-manager-variables]: /azure/azure-resource-manager/resource-group-template-functions-deployment
[nsg]: /azure/virtual-network/virtual-networks-nsg
