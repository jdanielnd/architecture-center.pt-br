---
title: Usar um objeto como um parâmetro em um modelo do Azure Resource Manager
description: Descreve como estender a funcionalidade dos modelos do Azure Resource Manager para usar objetos como parâmetros
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 76f8b9d459f4ab3147b52762b7c26552ec92c7a3
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847168"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a>Usar um objeto como um parâmetro em um modelo do Azure Resource Manager

Quando você [criar modelos do Azure Resource Manager][azure-resource-manager-create-template], poderá especificar valores de propriedade de recurso diretamente no modelo ou definir um parâmetro e fornecer valores de durante a implantação. Você pode usar um parâmetro para cada valor de propriedade para implantações pequenas, mas há um limite de 255 parâmetros por implantação. Depois de obter para implantações maiores e mais complexas, você poderá ficar sem parâmetros.

Uma maneira de resolver esse problema é usar um objeto como parâmetro em vez de um valor. Para fazer isso, defina o parâmetro no modelo e especifique um objeto JSON em vez de um único valor durante a implantação. Em seguida, faça referência às subpropriedades do parâmetro usando a [`parameter()`função][azure-resource-manager-functions] e o operador ponto no seu modelo.

Vamos dar uma olhada em um exemplo que implanta um recurso de rede virtual. Primeiro, vamos especificar um parâmetro `VNetSettings` em nosso modelo e defina o `type` como `object`:

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
Em seguida, vamos fornecer valores para o objeto `VNetSettings`:

> [!NOTE]
> Para saber como fornecer valores de parâmetro durante a implantação, veja a seção **parâmetros** de [noções básicas sobre a estrutura e a sintaxe dos modelos do Azure Resource Manager][azure-resource-manager-authoring-templates]. 

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

Como você pode ver, nosso único parâmetro na verdade especifica três subpropriedades: `name`, `addressPrefixes` e `subnets`. Cada uma desses subpropriedades especifica um valor ou outras subpropriedades. O resultado é que o nosso único parâmetro especifica todos os valores necessários para implantar a rede virtual.

Agora vamos dar uma olhada no restante do nosso modelo para ver como o objeto `VNetSettings` é usado:

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
Os valores de nosso objeto `VNetSettings` são aplicadas para as propriedades solicitadas por nossos recursos de rede virtual usando a função `parameters()` com ambos os índices de matriz do `[]` e o operador ponto. Essa abordagem funcionará se você simplesmente quiser aplicar estaticamente os valores do objeto do parâmetro ao recurso. No entanto, se você deseja atribuir uma matriz de valores de propriedade dinamicamente durante a implantação usar um [loop de cópia][azure-resource-manager-create-multiple-instances]. Para usar um loop de cópia, você fornece uma matriz JSON dos valores de propriedade de recurso e o loop de cópia aplica dinamicamente os valores às propriedades do recurso. 

Há um problema do qual você deve estar ciente caso use a abordagem dinâmica. Para demonstrar o problema, vamos dar uma olhada em uma matriz típica de valores de propriedade. Neste exemplo, os valores das nossas propriedades são armazenados em uma variável. Observe que temos duas matrizes aqui&mdash;uma chamada `firstProperty` e outra chamada `secondProperty`. 

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

Agora vamos dar uma olhada da maneira que podemos acessar as propriedades na variável usando um loop de cópia.

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

A função `copyIndex()` retorna a iteração atual do loop de cópia e podemos usá-lo como um índice em cada uma das duas matrizes simultaneamente.

Isso funciona bem quando as duas matrizes têm o mesmo tamanho. O problema ocorre se você tiver cometido um erro e as duas matrizes têm diferentes comprimentos&mdash;nesse caso, seu modelo falhará na validação durante a implantação. Você pode evitar esse problema ao incluir todas as suas propriedades em um único objeto, porque ele é muito mais fácil de identificar quando um valor estiver ausente. Por exemplo, vamos dar uma olhada em outro objeto de parâmetro na qual cada elemento da matriz `propertyObject` é a união das matrizes `firstProperty` e `secondProperty` anteriores.

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

Você notou o terceiro elemento na matriz? A propriedade `number` está ausente, mas será muito mais fácil de notar isso quando ela estiver ausente quando você estiver criando os valores de parâmetro dessa maneira.

## <a name="using-a-property-object-in-a-copy-loop"></a>Uso de um objeto de propriedade em um loop de cópia

Essa abordagem é ainda mais útil quando combinada a [serial copy loop][azure-resource-manager-create-multiple], principalmente para implantar os recursos filhos. 

Para demonstrar isso, vamos dar uma olhada em um modelo que implante um [grupo de segurança de rede (NSG)][nsg] com duas regras de segurança. 

Primeiro, vamos dar uma olhada no nosso parâmetros. Quando examinamos nosso modelo, veremos que definimos um parâmetro denominado `networkSecurityGroupsSettings`, que inclui uma matriz chamada `securityRules`. Essa matriz contém dois objetos JSON que especificam um número de configurações para uma regra de segurança.

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

Agora vamos dar uma olhada no nosso modelo. Nosso primeiro recurso chamado `NSG1` implanta o NSG. Nosso segundo recurso chamado `loop-0` executa duas funções: primeiro, ele usa `dependsOn` no NSG para que sua implantação só seja iniciada quando o `NSG1` for concluído e essa é a primeira iteração do loop sequencial. Nosso terceiro recurso é um modelo aninhado que implanta nossas regras de segurança usando um objeto para seus valores de parâmetro como no último exemplo.

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

Vamos examinar mais detalhadamente como podemos especificar os valores de propriedade no recurso filho `securityRules`. Todas as nossas propriedades são referenciadas usando a função `parameter()` e, em seguida, use o operador ponto para fazer referência à nossa matriz `securityRules` indexada pelo valor atual da iteração. Por fim, usamos outro operador de ponto para fazer referência ao nome do objeto. 

## <a name="try-the-template"></a>Experimentar o modelo

Para experimentar esse modelo, siga estas etapas: 

1.  Visite o portal do Azure, selecione o ícone **+** e pesquise pelo tipo de recurso **implantação de modelo** e selecione-o.
2.  Quando chegar à página **Implantação de modelo**, selecione o botão **criar**. Esse botão abre a folha **implantação personalizada**.
3.  Selecione o botão **Editar modelo**.
4.  Exclua o modelo vazio. 
5.  Copie e cole o modelo de exemplo no painel direito.
6.  Selecione o botão **Salvar**.
7.  Quando retornar ao painel de **implantação personalizada**, selecione o botão **Editar parâmetros**.
8.  Na folha **editar parâmetros**, exclua o modelo existente.
9.  Copie e cole o modelo de parâmetro de exemplo acima.
10. Selecione o botão **Salvar**, que fará com que você retorne à folha **implantação personalizada**.
11. Na folha **Implantação personalizada**, selecione sua assinatura, crie um novo grupo de recursos ou use um existente e selecione uma localização. Examine os termos e condições e selecione a caixa de seleção **Eu concordo**.
12. Selecione o botão **Comprar**.

## <a name="next-steps"></a>Próximas etapas

* Você pode expandir essas técnicas para implementar um [transformador de objeto de propriedade e um coletor](./collector.md). As técnicas de transformador e coletor são mais gerais e podem ser vinculadas de seus modelos.
* Esse padrão também é implementado no [projeto de blocos de construção do modelo](https://github.com/mspnp/template-building-blocks) e nas [arquiteturas de referência do Azure](/azure/architecture/reference-architectures/). Você pode examinar os nossos modelos para ver como implementamos essa técnica.

<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg