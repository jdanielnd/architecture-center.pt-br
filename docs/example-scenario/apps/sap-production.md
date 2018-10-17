---
title: Executar cargas de trabalho de produção do SAP usando um banco de dados Oracle no Azure
description: Execute uma implantação de produção do SAP no Azure usando um banco de dados Oracle.
author: DharmeshBhagat
ms.date: 9/12/2018
ms.openlocfilehash: 4c0e054a9024cd50581acd5b472a09d6e98d2bed
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/05/2018
ms.locfileid: "48818557"
---
# <a name="running-sap-production-workloads-using-an-oracle-database-on-azure"></a>Executar cargas de trabalho de produção do SAP usando um Oracle Database no Microsoft Azure

Os sistemas SAP são usados para executar aplicativos comerciais críticos. As interrupções suspendem processos importantes e podem gerar aumento de despesas ou perda de receita. Para evitar esses resultados, é necessário ter uma infraestrutura SAP altamente disponível e resiliente quando ocorrem falhas.

A criação de um ambiente SAP altamente disponível exige a eliminação de pontos únicos de falha nos processos e na arquitetura do sistema. Pontos únicos de falha podem ser causados por falhas no site, erros nos componentes do sistema ou até mesmo por erro humano.

Este cenário de exemplo demonstra uma implantação do SAP em VMs (máquinas virtuais) do Windows ou do Linux no Microsoft Azure, juntamente com um Oracle Database de HA (alta disponibilidade). Para a implantação do SAP, você pode usar VMs de vários tamanhos, de acordo com sua necessidade.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Considere este exemplo para os seguintes cenários:

* Cargas de trabalho críticas em execução no SAP.
* Cargas de trabalho não críticas do SAP.
* Ambientes de teste para SAP que simulam um ambiente de alta disponibilidade.

## <a name="architecture"></a>Arquitetura

![Visão geral da arquitetura de um ambiente de produção SAP no Microsoft Azure][architecture]

Este exemplo inclui uma configuração de alta disponibilidade para Oracle Database, serviços centrais SAP e vários servidores de aplicativos SAP em execução em diferentes máquinas virtuais. A Rede do Azure usa uma [topologia de Hub e Spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) para fins de segurança. Os dados fluem pela solução da seguinte maneira:

1. Os usuários acessam o sistema SAP por meio da interface do usuário do SAP, de um navegador da Web ou de outras ferramentas de cliente, como o Microsoft Excel. Uma conexão do ExpressRoute fornece acesso da rede local da organização aos recursos em execução no Microsoft Azure.
2. O ExpressRoute é finalizado no Microsoft Azure, no gateway da VNet (rede virtual) do ExpressRoute. O tráfego de rede é roteado para uma sub-rede do gateway por meio do gateway do ExpressRoute criado na VNet do hub.
3. A VNet é emparelhada com a VNet de spoke. A sub-rede da camada do aplicativo hospeda as máquinas virtuais que executam o SAP em um conjunto de disponibilidade.
4. Os servidores de gerenciamento de identidade fornecem serviços de autenticação para a solução.
5. O jumpbox é usado pelos administradores do sistema para gerenciar com segurança os recursos implantados no Microsoft Azure.

### <a name="components"></a>Componentes

* As [Redes Virtuais](/azure/virtual-network/virtual-networks-overview) são usadas neste cenário para criar uma topologia Hub e Spoke virtual no Microsoft Azure.
* As [Máquinas Virtuais](/azure/virtual-machines/windows/overview) fornecem os recursos de computação para cada camada da solução. Cada cluster de máquinas virtuais é configurado como um [conjunto de disponibilidade](/azure/virtual-machines/windows/regions-and-availability#availability-sets).
* O [ExpressRoute](/azure/expressroute/expressroute-introduction) estende a rede local para a nuvem da Microsoft por meio de uma conexão privada estabelecida por um provedor de conectividade.
* Os [NSGs (Grupos de Segurança de Rede)](/azure/virtual-network/security-overview) limitam o acesso de rede aos recursos de uma rede virtual. Um NSG contém uma lista de regras de segurança que permitem ou impedem o tráfego de rede com base no endereço IP, na porta e no protocolo de origem ou destino. 
* Os [Grupos de Recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups) atuam como contêineres lógicos para recursos do Azure.

### <a name="alternatives"></a>Alternativas

O SAP fornece opções flexíveis para diferentes combinações de sistema operacional, sistema de gerenciamento de banco de dados e tipos de VM em um ambiente do Microsoft Azure. Para saber mais, confira a [nota SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533), "Aplicativos SAP no Azure: produtos com suporte e tipos de VM do Azure".

## <a name="considerations"></a>Considerações

Há práticas recomendadas para criação de ambientes SAP de alta disponibilidade no Microsoft Azure. Para saber mais, confira [Arquitetura de alta disponibilidade e cenários para SAP NetWeaver](/azure/virtual-machines/workloads/sap/sap-high-availability-architecture-scenarios).
Para saber mais, confira [Alta disponibilidade de aplicativos SAP em VMs do Microsoft Azure](/azure/virtual-machines/workloads/sap/high-availability-guide).
* Os Oracle Databases também fornecem práticas recomendadas para Microsoft Azure. Para saber mais, confira [Criando e implementando um Oracle Database no Microsoft Azure](/azure/virtual-machines/workloads/oracle/oracle-design). 
* O Oracle Data Guard é usado para eliminar pontos únicos de falha em Oracle Databases críticos. Para saber mais, confira [Implementando o Oracle Data Guard em uma máquina virtual do Linux no Microsoft Azure](/azure/virtual-machines/workloads/oracle/configure-oracle-dataguard).

O Microsoft Azure oferece serviços de infraestrutura que podem ser usados para implantar produtos SAP com um Oracle Database. Para saber mais, confira [Implantando um DBMS da Oracle no Microsoft Azure para uma carga de trabalho SAP](/azure/virtual-machines/workloads/sap/dbms_guide_oracle).

## <a name="pricing"></a>Preços

Para ajudá-lo a explorar o custo da execução nesse cenário, todos os serviços são pré-configurados nos exemplos de calculadora de custos abaixo. Para ver como o preço seria alterado em seu caso de uso específico, altere as variáveis apropriadas de acordo com o tráfego esperado.

Fornecemos quatro exemplos de perfis de custo com base na quantidade de tráfego que você espera receber:

|Tamanho|SAPs|Tipo de VM de BD|Armazenamento de BD|VM (A)SCS|Armazenamento (A)SCS|Tipo de VM de aplicativo|Armazenamento de aplicativo|Calculadora de Preços do Azure|
|----|----|-------|-------|-----|---|---|--------|---------------|
|Pequena|30000|DS13_v2|4xP20, 1xP20|DS11_v2|1xP10|DS13_v2|1xP10|[Pequeno](https://azure.com/e/45880ba0bfdf47d497851a7cf2650c7c)|
|Média|70000|DS14_v2|6xP20, 1xP20|DS11_v2|1xP10|4x DS13_v2|1xP10|[Médio](https://azure.com/e/9a523f79591347ca9a48c3aaa1406f8a)|
grande|180000|E32s_v3|5xP30, 1xP20|DS11_v2|1xP10|6x DS14_v2|1xP10|[Grande](https://azure.com/e/f70fccf571e948c4b37d4fecc07cbf42)|
Extra grande|250.000|M64s|6xP30, 1xP30|DS11_v2|1xP10|10x DS14_v2|1xP10|[Extra grande](https://azure.com/e/58c636922cf94faf9650f583ff35e97b)|

> [!NOTE]
> Esse preço é um guia e indica apenas os custos de armazenamento e VMs. Isso exclui rede, armazenamento de backup e encargos de entrada/saída de dados.

* [Pequeno](https://azure.com/e/45880ba0bfdf47d497851a7cf2650c7c): um sistema pequeno consiste em um tipo de VM DS13_v2 para o servidor de banco de dados com 8 vCPUs, 56 GB de RAM e 112 GB de armazenamento temporário, além de cinco discos de armazenamento premium de 512 GB. Um servidor do SAP Central Instance usando um tipo de VM DS11_v2 com 2 vCPUs, 14 GB de RAM e 28 GB de armazenamento temporário. Um tipo de VM única DS13_v2 para o servidor de aplicativos SAP com 8 vCPUs, 56 GB de RAM e 400 GB de armazenamento temporário, além de um disco de armazenamento premium de 128 GB.

* [Médio](https://azure.com/e/9a523f79591347ca9a48c3aaa1406f8a): um sistema médio consiste em um tipo de VM DS14_v2 para o servidor de banco de dados com 16 vCPUs, 112 GB de RAM e 800 GB de armazenamento temporário, além de sete discos de armazenamento premium de 512 GB. Um servidor do SAP Central Instance usando um tipo de VM DS11_v2 com 2 vCPUs, 14 GB de RAM e 28 GB de armazenamento temporário. Quatro tipos de VM DS13_v2 para o servidor de aplicativos SAP com 8 vCPUs, 56 GB de RAM e 400 GB de armazenamento temporário, além de um disco de armazenamento premium de 128 GB.

* [Grande](https://azure.com/e/f70fccf571e948c4b37d4fecc07cbf42): um sistema grande consiste em um tipo de VM E32s_v3 para o servidor de banco de dados com 32 vCPUs, 256 GB de RAM e 800 GB de armazenamento temporário, além de três discos de armazenamento premium de 512 GB e um de 128 GB. Um servidor do SAP Central Instance usando um tipo de VM DS11_v2 com 2 vCPUs, 14 GB de RAM e 28 GB de armazenamento temporário. Seis tipos de VM DS14_v2 para o servidor de aplicativos SAP com 16 vCPUs, 112 GB de RAM e 224 GB de armazenamento temporário, além de seis discos de armazenamento premium de 128 GB.

* [Extra grande](https://azure.com/e/58c636922cf94faf9650f583ff35e97b): um sistema extra grande consiste em um tipo de VM M64s para o servidor de banco de dados com 64 vCPUs, 1.024 GB de RAM e 2.000 GB de armazenamento temporário, além de sete discos de armazenamento premium de 1.024 GB. Um servidor do SAP Central Instance usando um tipo de VM DS11_v2 com 2 vCPUs, 14 GB de RAM e 28 GB de armazenamento temporário. Dez tipos de VM DS14_v2 para os servidores de aplicativos SAP com 16 vCPUs, 112 GB de RAM e 224 GB de armazenamento temporário, além de dez discos de armazenamento premium de 128 GB.

## <a name="deployment"></a>Implantação

Use o seguinte link para implantar a infraestrutura subjacente para este cenário.

<a
href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-3tier-distributed-ora%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

> [!NOTE]
> SAP e Oracle não são instalados durante essa implantação. Você deve implantar esses componentes separadamente.

## <a name="related-resources"></a>Recursos relacionados

Para saber mais sobre como executar cargas de trabalho de produção SAP no Microsoft Azure, confira as seguintes arquiteturas de referência:
* [SAP NetWeaver para AnyDB](/azure/architecture/reference-architectures/sap/sap-netweaver) 
* [SAP S/4HANA](/azure/architecture/reference-architectures/sap/sap-s4hana)
* [Instâncias grandes do SAP HANA](/azure/architecture/reference-architectures/sap/hana-large-instances)

<!-- links -->
[architecture]: media/architecture-sap-production.png
