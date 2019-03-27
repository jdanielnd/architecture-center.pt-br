---
title: 'CAF: Coletar dados de inventário para uma propriedade digital'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
description: Como coletar dados de inventário para um estado digital.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 0c68ff1e5ff51395698d37fb9b59c7a76c9479b7
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242547"
---
# <a name="gather-inventory-data-for-a-digital-estate"></a><span data-ttu-id="47131-103">Coletar dados de inventário para uma propriedade digital</span><span class="sxs-lookup"><span data-stu-id="47131-103">Gather inventory data for a digital estate</span></span>

<span data-ttu-id="47131-104">Desenvolver um inventário é a primeira etapa no [Planejamento de propriedade digital](overview.md).</span><span class="sxs-lookup"><span data-stu-id="47131-104">Developing an inventory is the first step in [Digital Estate Planning](overview.md).</span></span> <span data-ttu-id="47131-105">Nesse processo, uma lista de ativos de TI que dão suporte a funções de negócios específicas seria coletada para análise e racionalização posterior.</span><span class="sxs-lookup"><span data-stu-id="47131-105">In this process, a list of IT assets that support specific business functions would be collected for later analysis and rationalization.</span></span> <span data-ttu-id="47131-106">Este artigo pressupõe que uma abordagem de baixo para cima à análise é mais apropriada para necessidades de planejamento.</span><span class="sxs-lookup"><span data-stu-id="47131-106">This article assumes that a bottom-up approach to analysis is most appropriate for planning needs.</span></span> <span data-ttu-id="47131-107">Para saber mais, confira [Abordagens para planejamento de propriedade digital](./approach.md).</span><span class="sxs-lookup"><span data-stu-id="47131-107">For more information, see [Approaches to digital estate planning](./approach.md).</span></span>

## <a name="take-inventory-of-a-digital-estate"></a><span data-ttu-id="47131-108">Faça o inventário de um espaço digital</span><span class="sxs-lookup"><span data-stu-id="47131-108">Take inventory of a digital estate</span></span>

<span data-ttu-id="47131-109">O inventário que dá suporte a uma propriedade digital muda dependendo da Transformação Digital desejada e a Jornada de Transformação correspondente.</span><span class="sxs-lookup"><span data-stu-id="47131-109">The inventory supporting a digital estate changes depending on the desired digital transformation and corresponding transformation journey.</span></span>

- <span data-ttu-id="47131-110">**Transformação operacional**.</span><span class="sxs-lookup"><span data-stu-id="47131-110">**Operational transformation**.</span></span> <span data-ttu-id="47131-111">Durante uma transformação operacional, geralmente aconselhamos a coleta do inventário de ferramentas de exame que possam criar uma lista centralizada de todas as VMs e servidores.</span><span class="sxs-lookup"><span data-stu-id="47131-111">During an operational transformation, it is often advised that the inventory be collected from scanning tools which can create a centralized list of all VMs and servers.</span></span> <span data-ttu-id="47131-112">Algumas ferramentas também podem criar mapeamentos de rede e dependências, o que ajudará a definir o alinhamento da carga de trabalho.</span><span class="sxs-lookup"><span data-stu-id="47131-112">Some tools can also create network mappings and dependencies, which will help define workload alignment.</span></span>

- <span data-ttu-id="47131-113">**Transformação incremental.**</span><span class="sxs-lookup"><span data-stu-id="47131-113">**Incremental transformation.**</span></span> <span data-ttu-id="47131-114">O inventário para uma transformação incremental começa com o cliente.</span><span class="sxs-lookup"><span data-stu-id="47131-114">Inventory for an incremental transformation begins with the customer.</span></span> <span data-ttu-id="47131-115">O mapeamento da experiência do cliente do início ao fim é um bom lugar para começar.</span><span class="sxs-lookup"><span data-stu-id="47131-115">Mapping the customer experience from start to finish is a good place to begin.</span></span> <span data-ttu-id="47131-116">O alinhamento mapeado para aplicativos, APIs, dados e outros ativos criará um inventário detalhado para análise.</span><span class="sxs-lookup"><span data-stu-id="47131-116">Aligning that map to applications, APIs, data, and other assets will create a detailed inventory for analysis.</span></span>

- <span data-ttu-id="47131-117">**Transformação Interruptiva.**</span><span class="sxs-lookup"><span data-stu-id="47131-117">**Disruptive transformation.**</span></span> <span data-ttu-id="47131-118">A transformação interruptiva se concentra no produto ou serviço.</span><span class="sxs-lookup"><span data-stu-id="47131-118">Disruptive transformation focuses on the product or service.</span></span> <span data-ttu-id="47131-119">A partir daí, um inventário incluiria um mapeamento das oportunidades de interrupção do mercado e dos recursos necessários.</span><span class="sxs-lookup"><span data-stu-id="47131-119">From there an inventory would include a mapping of the opportunities to disrupt the market and the capabilities needed.</span></span>

## <a name="accuracy-and-completeness-of-an-inventory"></a><span data-ttu-id="47131-120">Precisão e integridade de um inventário</span><span class="sxs-lookup"><span data-stu-id="47131-120">Accuracy and completeness of an inventory</span></span>

<span data-ttu-id="47131-121">Raramente, um inventário fica totalmente concluído em sua primeira iteração.</span><span class="sxs-lookup"><span data-stu-id="47131-121">An inventory is seldom fully complete in its first iteration.</span></span> <span data-ttu-id="47131-122">É altamente recomendável que vários membros da equipe de Estratégia de Nuvem se alinhem aos participantes e usuários avançados para validar o inventário.</span><span class="sxs-lookup"><span data-stu-id="47131-122">It is highly advised that various members of the Cloud Strategy team align stakeholders and power users to validate the inventory.</span></span> <span data-ttu-id="47131-123">Quando possível, outras ferramentas, como a análise de rede e de dependência, podem ser usadas para identificar os ativos que estão recebendo tráfego, mas que não estão no inventário.</span><span class="sxs-lookup"><span data-stu-id="47131-123">When possible, additional tools like network and dependency analysis can be used to identify assets that are being sent traffic, but are not in the inventory.</span></span>

## <a name="next-steps"></a><span data-ttu-id="47131-124">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="47131-124">Next steps</span></span>

<span data-ttu-id="47131-125">Após a compilação e validação de um inventário, ele pode ser racionalizado.</span><span class="sxs-lookup"><span data-stu-id="47131-125">Once an inventory is compiled and validated, it can rationalized.</span></span> <span data-ttu-id="47131-126">Racionalização de Inventário é a próxima etapa para planejamento de propriedade digital.</span><span class="sxs-lookup"><span data-stu-id="47131-126">Inventory Rationalization is the next step to digital estate planning.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="47131-127">Racionalizar a propriedade digital</span><span class="sxs-lookup"><span data-stu-id="47131-127">Rationalize the digital estate</span></span>](rationalize.md)