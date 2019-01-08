---
title: Ambientes de desenvolvimento/teste para cargas de trabalho do SAP
titleSuffix: Azure Example Scenarios
description: Crie um ambiente de desenvolvimento/teste para cargas de trabalho do SAP.
author: AndrewDibbins
ms.date: 7/11/18
ms.custom: fasttrack
ms.openlocfilehash: 3f6c828e8757a3f82ad6972a8f21cd2fed629162
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/20/2018
ms.locfileid: "53643961"
---
# <a name="devtest-environments-for-sap-workloads-on-azure"></a>Ambientes de desenvolvimento/teste para cargas de trabalho do SAP no Azure

Este exemplo mostra como estabelecer um ambiente de desenvolvimento/teste do SAP NetWeaver em um ambiente Windows ou Linux no Azure. O banco de dados usado é o AnyDB, o termo SAP para qualquer DBMS compatível (que não seja SAP HANA). Como essa arquitetura foi projetada para ambientes que não são de produção, ele é implantado com somente uma máquina virtual (VM) e o seu tamanho pode ser alterado para acomodar as necessidades da sua organização.

Para casos de uso de produção veja as arquiteturas SAP de referência disponíveis abaixo:

- [SAP NetWeaver para AnyDB][sap-netweaver]
- [SAP S/4HANA][sap-hana]
- [SAP em instâncias grandes do Azure][sap-large]

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Outros casos de uso relevantes incluem:

- Cargas de trabalho do SAP não críticas e que não são de produção (área restrita, desenvolvimento, teste, garantia de qualidade)
- Cargas de trabalho de negócios não críticas do SAP

## <a name="architecture"></a>Arquitetura

![Diagrama de arquitetura para ambientes de desenvolvimento/teste para cargas de trabalho do SAP](media/architecture-sap-dev-test.png)

Este cenário demonstra o provisionamento de um único banco de dados do sistema do SAP e o servidor de aplicativos SAP em uma única máquina virtual. O fluxo de dados neste cenário ocorre da seguinte forma:

1. Os clientes usam a interface do usuário do SAP ou outras ferramentas de cliente (Excel, um navegador da Web ou outro aplicativo Web) para acessar o sistema SAP com base no Azure.
2. A conectividade é fornecida por meio do uso do ExpressRoute estabelecido. A conexão do ExpressRoute é terminada no Azure no gateway do ExpressRoute. O tráfego de rede é roteado pelo gateway do ExpressRoute para a sub-rede do gateway e desta para a sub-rede spoke da camada de aplicativo (confira o [padrão hub-spoke][hub-spoke]) e por um Gateway de Segurança de Rede até a máquina virtual do aplicativo SAP.
3. Os servidores de gerenciamento de identidade oferecem serviços de autenticação.
4. A caixa de atalhos oferece recursos de gerenciamento local.

### <a name="components"></a>Componentes

- As [Redes Virtuais](/azure/virtual-network/virtual-networks-overview) são a base das comunicações de rede no Azure.
- As [Máquinas Virtuais](/azure/virtual-machines/windows/overview) do Azure oferecem uma infraestrutura sob demanda, de alta capacidade de dimensionamento, virtualizada que usa um servidor Windows ou Linux.
- O [ExpressRoute](/azure/expressroute/expressroute-introduction) permite que você estenda suas redes locais até a nuvem da Microsoft por meio de conexão privada facilitada por um provedor de conectividade.
- [Grupo de segurança de rede](/azure/virtual-network/security-overview) permite limitar o tráfego de rede para recursos em uma rede virtual. Um grupo de segurança de rede contém uma lista de regras de segurança que permitem ou negam o tráfego de rede de entrada ou saída com base no endereço IP de origem ou destino, na porta e no protocolo.
- Os [Grupos de Recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups) atuam como contêineres lógicos para recursos do Azure.

## <a name="considerations"></a>Considerações

### <a name="availability"></a>Disponibilidade

 A Microsoft oferece um contrato de nível de serviço (SLA) para instância de VM individuais. Para saber mais informações sobre o contrato de nível de serviço do Microsoft Azure para máquinas virtuais consulte [SLA para máquinas virtuais](https://azure.microsoft.com/support/legal/sla/virtual-machines)

### <a name="scalability"></a>Escalabilidade

Para obter diretrizes gerais sobre como criar soluções escalonáveis, confira a [lista de verificação de escalabilidade] [ scalability] no Azure Architecture Center.

### <a name="security"></a>Segurança

Para obter orientação geral sobre como criar soluções seguras, confira a [Documentação de segurança do Azure][security].

### <a name="resiliency"></a>Resiliência

Para obter diretrizes gerais sobre como criar soluções resilientes, confira [Projetando aplicativos resilientes para o Azure][resiliency].

## <a name="pricing"></a>Preços

Para ajudá-lo a explorar o custo da execução nesse cenário, todos os serviços são pré-configurados nos exemplos de calculadora de custos abaixo. Para ver como o preço seria alterado em seu caso de uso específico, altere as variáveis apropriadas de acordo com o tráfego esperado.

Fornecemos quatro exemplos de perfis de custo com base na quantidade de tráfego que você espera receber:

|Tamanho|SAPs|Tipo de VM|Armazenamento|Calculadora de Preços do Azure|
|----|----|-------|-------|---------------|
|Pequena|8000|D8s_v3|2xP20, 1xP10|[Pequeno](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|Média|16000|D16s_v3|3xP20, 1xP10|[Médio](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
grande|32000|E32s_v3|3xP20, 1xP10|[Grande](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
Extra grande|64000|M64s|4xP20, 1xP10|[Extra grande](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

> [!NOTE]
> Esse preço é um guia que indica somente as VMs e os custos de armazenamento. Isso exclui rede, armazenamento de backup e encargos de entrada/saída de dados.

- [Pequeno](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): um sistema pequeno é composto por uma VM do tipo D8s_v3 com 8x vCPUs, 32 GB de RAM e 200 GB de armazenamento temporário, além de dois discos de armazenamento premium de 512 GB e um de 128 GB.
- [Médio](https://azure.com/e/465bd07047d148baab032b2f461550cd): um sistema médio é composto por uma VM do tipo D16s_v3 com 16x vCPUs, 64 GB de RAM e 400 GB de armazenamento temporário, além de três discos de armazenamento premium de 512 GB e um de 128 GB.
- [Grande](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): um sistema grande é composto por uma VM do tipo E32s_v3 com 32x vCPUs, 256 GB de RAM e 512 GB de armazenamento temporário, além de três discos de armazenamento premium de 512 GB e um de 128 GB.
- [Extra grande](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): um sistema extragrande é composto por uma VM do tipo M64s com 64x vCPUs, 1024 GB de RAM e 2000 GB de armazenamento temporário, além de quatro discos de armazenamento premium de 512 GB e um de 128 GB.

## <a name="deployment"></a>Implantação

Clique aqui para implantar a infraestrutura subjacente para esse cenário.

<!-- markdownlint-disable MD033 -->

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

<!-- markdownlint-enable MD033 -->

> [!NOTE]
> SAP e Oracle não são instalados durante essa implantação. Você deve implantar esses componentes separadamente.

<!-- links -->
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke
