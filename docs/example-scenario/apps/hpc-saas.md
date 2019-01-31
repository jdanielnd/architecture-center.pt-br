---
title: Um serviço de engenharia auxiliada por computador
titleSuffix: Azure Example Scenarios
description: Fornece uma plataforma de software como um serviço (SaaS) para a engenharia auxiliada por computador (CAE) no Azure.
author: alexbuckgit
ms.date: 08/22/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: HPC
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-hpc-saas.png
ms.openlocfilehash: bd38bd0042fceeab6efe04d7b6d1477d17ada7f5
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908443"
---
# <a name="a-computer-aided-engineering-service-on-azure"></a>Um serviço de engenharia auxiliada por computador no Azure

Este cenário de exemplo demonstra a distribuição de uma plataforma de software como serviço (SaaS) desenvolvida sobre as funcionalidades de computação de alto desempenho (HPC) do Azure. O cenário se baseia em uma solução de engenharia de software. No entanto, a arquitetura também é relevante para outros setores que exijam recursos de HPC como renderização de imagens, modelagem complexa e cálculo de riscos financeiros.

Este exemplo apresenta um provedor de software de engenharia que distribui aplicativos de engenharia auxiliada por computador (CAE) para empresas de fabricação e de engenharia. As soluções de CAE possibilitam a inovação, reduzem os tempos de desenvolvimento e diminuem os custos ao longo do tempo de vida do design de um produto. Tais soluções exigem uma grande quantidade de recursos computacionais e em geral processam grandes volumes de dados. Os altos custos de um dispositivo de HPC local ou de estações de trabalho de ponta muitas vezes afastam essas tecnologias do alcance de pequenas empresas de engenharia, de empreendedores e alunos.

A empresa deseja expandir o mercado para os próprios aplicativos, criando uma plataforma de SaaS apoiada em tecnologias de HPC baseadas em nuvem. Os clientes devem ter condições de pagar pelos recursos computacionais conforme necessário e acessar uma enorme capacidade computacional que seria inviável de outra maneira.

As metas da empresa incluem:

- Tirar proveito das funcionalidades da computação de alto desempenho (HPC) no Azure para acelerar o processo de teste e design do produto.
- Usar as mais recentes inovações de hardware para executar simulações complexas e minimizar os custos de simulações mais simples.
- Permitir renderizações e visualizações realistas em um navegador da Web sem a necessidade de uma estação de trabalho com engenharia de ponta.

## <a name="relevant-use-cases"></a>Casos de uso relevantes

Outros casos de uso relevantes incluem:

- Pesquisas de genoma
- Simulação de clima
- Aplicativos de química computacional

## <a name="architecture"></a>Arquitetura

![Arquitetura de uma solução de SaaS que possibilita funcionalidades de HPC][architecture]

- Os usuários podem acessar máquinas virtuais (VMs) da série NV através de um navegador com uma conexão de RDP com base em HTML5 usando o [serviço Apache Guacamole](https://guacamole.apache.org/). Essas instâncias de VM oferecem GPUs poderosas para tarefas de renderização e colaboração. Os usuários podem editar os designs e visualizar os resultados sem precisar acessar laptops ou dispositivos de computação móvel de ponta. O agendador gira as VMs adicionais com base na heurística definida pelo usuário.
- Em uma sessão de CAD da área de trabalho, os usuários podem enviar cargas de trabalho para execução em nós de cluster de HPC disponíveis. Essas cargas de trabalho executam tarefas como análise de estresse ou cálculos computacionais de dinâmica de fluidos, eliminando a necessidade de clusters de cálculo locais dedicados. Esses nós de cluster podem ser configurados para dimensionamento automático com base na profundidade da fila ou da carga com base na demanda do usuário ativo para recursos computacionais.
- O Serviço de Kubernetes do Azure (AKS) é usado para hospedar os recursos da Web disponíveis para usuários finais.

### <a name="components"></a>Componentes

- As [máquinas virtuais da série H](/azure/virtual-machines/linux/sizes-hpc) são usadas na execução de simulações computacionais intensas, como a modelagem molecular e a dinâmica de fluido computacional. A solução também se vale das tecnologias do tipo conectividade de acesso remoto direto à memória (RDMA) e redes InfiniBand.
- As [máquinas virtuais da série NV](/azure/virtual-machines/windows/sizes-gpu) oferecem aos engenheiros uma funcionalidade de ponta para a estação de trabalho em um navegador da Web padrão. Essas máquinas virtuais têm GPUs Tesla M60 NVIDIA compatíveis com renderização avançada e podem executar cargas de trabalho de precisão única.
- As [ máquinas virtuais de uso geral](/azure/virtual-machines/linux/sizes-general) que executam CentOS lidam com cargas de trabalho mais tradicionais, como aplicativos Web.
- A carga do [Gateway de Aplicativo](/azure/application-gateway/overview) balanceia a carga das solicitações que chegam aos servidores Web.
- O [Serviço de Kubernetes do Azure (AKS)](/azure/aks/intro-kubernetes) é usado para executar cargas de trabalho escalonáveis com um custo menor para simulações que não exijam as funcionalidades de ponta de HPC ou das máquinas virtuais de GPU.
- O [Pacote Altair PBS Works](https://www.pbsworks.com/PBSProduct.aspx?n=PBS-Works-Suite&c=Overview-and-Capabilities) gerencia o fluxo de trabalho de HPC, garantindo que uma quantidade suficiente de instâncias de máquinas virtuais esteja disponível para lidar com a carga atual. Ele também desaloca máquinas virtuais em períodos de menor demanda para reduzir os custos.
- O [Armazenamento de Blobs](/azure/storage/blobs/storage-blobs-introduction) armazena os arquivos compatíveis com trabalhos agendados.

### <a name="alternatives"></a>Alternativas

- O [Azure CycleCloud](/azure/cyclecloud/overview) simplifica a criação, o gerenciamento, a operação e a otimização de clusters de HPC. Ele oferece recursos avançados de política e governança. O CycleCloud é compatível com qualquer agendador de trabalhos ou pilha de software.
- O [Pacote de HPC](/azure/virtual-machines/windows/hpcpack-cluster-options) pode criar e gerenciar um cluster de HPC do Azure para cargas de trabalho baseadas no Windows Server. O Pacote de HPC não é uma opção para cargas de trabalho baseadas em Linux.
- A [Configuração do Estado de Automação do Azure](/azure/automation/automation-dsc-overview) oferece uma abordagem de infraestrutura como código para definição das máquinas virtuais e softwares a ser implantados. As máquinas virtuais podem ser implantadas como parte de um conjunto de dimensionamento de máquinas virtuais, com regras de dimensionamento automático para nós de computação com base no número de trabalhos enviados para a fila de trabalho. Quando uma nova máquina virtual é necessária, ela é provisionada usando a imagem de patch mais recente da galeria de imagens do Azure e o software necessário é instalado e configurado por meio de um script de configuração de DSC do PowerShell.
- [Funções do Azure](/azure/azure-functions/functions-overview)

## <a name="considerations"></a>Considerações

- Por mais que o uso de uma abordagem de infraestrutura como código seja uma ótima maneira de gerenciar as definições de build de máquinas virtuais, usar um script para o provisionamento de uma nova máquina virtual pode levar muito tempo. Essa solução encontrou um meio-termo adequado usando o script de DSC na criação periódica de uma imagem emulada, que pode ser usada no provisionamento de uma nova máquina virtual mais rapidamente do que compilar totalmente uma VM sob demanda usando o DSC. O Azure DevOps Services ou outras ferramentas de CI/CD podem atualizar periodicamente as imagens emuladas usando scripts de DSC.
- Uma consideração importante é o equilíbrio entre os custos totais da solução e a ágil disponibilidade de recursos computacionais. Provisionar um pool de instâncias de máquinas virtuais da série N e colocá-las em um estado desalocado reduz os custos operacionais. Quando uma máquina virtual adicional for necessária, o realocamento de uma instância existente envolverá a ligação da máquina virtual em um host diferente, mas elimina-se o tempo de detecção de barramento PCI exigido pelo sistema operacional para identificar e instalar os drivers da GPU, já que uma máquina virtual que é desprovisionada e provisionada novamente manterá o mesmo barramento PCI para a GPU durante a reinicialização.
- A arquitetura original era totalmente dependente das máquinas virtuais do Azure para execução de simulações. Para reduzir os custos das cargas de trabalho que não exigiam todas as funcionalidades de uma máquina virtual, essas cargas foram postas em contêineres e implantadas para o Serviço de Kubernetes do Azure (AKS).
- A força de trabalho da empresa tinha habilidades existentes em tecnologias de código-fonte aberto. Eles podem tirar proveito dessas habilidades usando tecnologias como o Linux e o Kubernetes no desenvolvimento.

## <a name="pricing"></a>Preços

Para ajudar você a explorar o custo de execução desse cenário, muitos dos serviços exigidos foram pré-configurados em um [exemplo de calculadora de custos][calculator]. Os custos da solução vão depender da quantidade e do volume de serviços necessários para atender à suas necessidades.

As considerações a seguir vão orientar uma parte significativa dos custos para esta solução:

- Os custos da máquina virtual do Azure aumentam de forma linear conforme novas instâncias são provisionadas. As máquinas virtuais desalocadas só geram custos de armazenamento, não de computação. Essas máquinas desalocadas poderão ser realocadas posteriormente, quando houver alta demanda.
- Os custos do Serviço de Kubernetes do Azure são baseados no tipo de VM escolhido para dar suporte à carga de trabalho. Os custos aumentarão de forma linear com base no número de VMs no cluster.

## <a name="next-steps"></a>Próximas etapas

- Leia a [história do cliente Altair][source-document]. Este cenário de exemplo se baseia em uma versão da arquitetura deles.
- Examine outras [soluções de Big Compute](https://azure.microsoft.com/solutions/big-compute) disponíveis no Azure.

<!-- links -->
[architecture]: ./media/architecture-hpc-saas.png
[source-document]: https://customers.microsoft.com/story/altair-manufacturing-azure
[calculator]: https://azure.com/e/3cb9ccdc893f41ffbcdb00c328178ccf
