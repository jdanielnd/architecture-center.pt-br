---
title: Coletar dados de inventário para uma propriedade digital
titleSuffix: Enterprise Cloud Adoption
description: Como criar um inventário para uma propriedade digital
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 5c2f270cf8de81c8a94d1f924f51611e657ed0ed
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481789"
---
# <a name="enterprise-cloud-adoption-gather-inventory-data-for-a-digital-estate"></a><span data-ttu-id="2098e-103">Adoção da Nuvem Empresarial: Coletar dados de inventário para uma propriedade digital</span><span class="sxs-lookup"><span data-stu-id="2098e-103">Enterprise Cloud Adoption: Gather inventory data for a digital estate</span></span>

<span data-ttu-id="2098e-104">Desenvolver um inventário é a primeira etapa no [Planejamento de propriedade digital](overview.md).</span><span class="sxs-lookup"><span data-stu-id="2098e-104">Developing an inventory is the first step in [Digital Estate Planning](overview.md).</span></span> <span data-ttu-id="2098e-105">Nesse processo, uma lista de ativos de TI que dão suporte a funções de negócios específicas seria coletada para análise e racionalização posterior.</span><span class="sxs-lookup"><span data-stu-id="2098e-105">In this process, a list of IT assets that support specific business functions would be collected for later analysis and rationalization.</span></span> <span data-ttu-id="2098e-106">Este artigo pressupõe que uma abordagem de baixo para cima à análise é mais apropriada para necessidades de planejamento.</span><span class="sxs-lookup"><span data-stu-id="2098e-106">This article assumes that a bottom-up approach to analysis is most appropriate for planning needs.</span></span> <span data-ttu-id="2098e-107">Para saber mais, confira [Abordagens para planejamento de propriedade digital](./approach.md).</span><span class="sxs-lookup"><span data-stu-id="2098e-107">For more information, see [Approaches to digital estate planning](./approach.md).</span></span>

## <a name="how-can-a-digital-estate-be-inventoried"></a><span data-ttu-id="2098e-108">Como uma propriedade digital pode ser inserida no inventário?</span><span class="sxs-lookup"><span data-stu-id="2098e-108">How can a digital estate be inventoried?</span></span>

<span data-ttu-id="2098e-109">O inventário que dá suporte a uma propriedade digital muda dependendo da Transformação Digital desejada e a Jornada de Transformação correspondente.</span><span class="sxs-lookup"><span data-stu-id="2098e-109">The inventory supporting a digital estate changes depending on the desired Digital Transformation and corresponding Transformation Journey.</span></span>

- <span data-ttu-id="2098e-110">Transformação operacional: Durante uma transformação operacional, geralmente aconselhamos a coleta do inventário de ferramentas de exame que possam criar uma lista centralizada de todas as VMs e servidores.</span><span class="sxs-lookup"><span data-stu-id="2098e-110">Operational Transformation: During an operational transformation, it is often advised that the inventory be collected from scanning tools which can create a centralized list of all VMs and servers.</span></span> <span data-ttu-id="2098e-111">Algumas ferramentas também podem criar mapeamentos de rede e dependências, o que ajudará a definir o alinhamento da carga de trabalho.</span><span class="sxs-lookup"><span data-stu-id="2098e-111">Some tools can also create network mappings and dependencies, which will help define workload alignment.</span></span>

- <span data-ttu-id="2098e-112">Transformação incremental: O inventário para uma transformação incremental começa com o cliente.</span><span class="sxs-lookup"><span data-stu-id="2098e-112">Incremental Transformation: Inventory for an incremental transformation begins with the customer.</span></span> <span data-ttu-id="2098e-113">O mapeamento da experiência do cliente do início ao fim é um bom lugar para começar.</span><span class="sxs-lookup"><span data-stu-id="2098e-113">Mapping the customer experience from start to finish is a good place to begin.</span></span> <span data-ttu-id="2098e-114">O alinhamento mapeado para aplicativos, APIs, dados e outros ativos criará um inventário detalhado para análise.</span><span class="sxs-lookup"><span data-stu-id="2098e-114">Aligning that map to applications, APIs, data, and other assets will create a detailed inventory for analysis.</span></span>

- <span data-ttu-id="2098e-115">Transformação Interruptiva: A transformação interruptiva se concentra no produto ou serviço.</span><span class="sxs-lookup"><span data-stu-id="2098e-115">Disruptive Transformation: Disruptive transformation focuses on the product or service.</span></span> <span data-ttu-id="2098e-116">A partir daí, um inventário incluiria um mapeamento das oportunidades de interrupção do mercado e dos recursos necessários.</span><span class="sxs-lookup"><span data-stu-id="2098e-116">From there an inventory would include a mapping of the opportunities to disrupt the market and the capabilities needed.</span></span>

## <a name="accuracy-and-completeness-of-an-inventory"></a><span data-ttu-id="2098e-117">Precisão e integridade de um inventário</span><span class="sxs-lookup"><span data-stu-id="2098e-117">Accuracy and completeness of an inventory</span></span>

<span data-ttu-id="2098e-118">Raramente, um inventário fica totalmente concluído em sua primeira iteração.</span><span class="sxs-lookup"><span data-stu-id="2098e-118">An inventory is seldom fully complete in its first iteration.</span></span> <span data-ttu-id="2098e-119">É altamente recomendável que vários membros da equipe de Estratégia de Nuvem se alinhem aos participantes e usuários avançados para validar o inventário.</span><span class="sxs-lookup"><span data-stu-id="2098e-119">It is highly advised that various members of the Cloud Strategy team align stakeholders and power users to validate the inventory.</span></span> <span data-ttu-id="2098e-120">Quando possível, outras ferramentas, como a análise de rede e de dependência, podem ser usadas para identificar os ativos que estão recebendo tráfego, mas que não estão no inventário.</span><span class="sxs-lookup"><span data-stu-id="2098e-120">When possible, additional tools like network and dependency analysis can be used to identify assets that are being sent traffic, but are not in the inventory.</span></span>

## <a name="next-steps"></a><span data-ttu-id="2098e-121">Próximas etapas</span><span class="sxs-lookup"><span data-stu-id="2098e-121">Next steps</span></span>

<span data-ttu-id="2098e-122">Após a compilação e validação de um inventário, ele pode ser racionalizado.</span><span class="sxs-lookup"><span data-stu-id="2098e-122">Once an inventory is compiled and validated, it can rationalized.</span></span> <span data-ttu-id="2098e-123">Racionalização de Inventário é a próxima etapa para planejamento de propriedade digital.</span><span class="sxs-lookup"><span data-stu-id="2098e-123">Inventory Rationalization is the next step to digital estate planning.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="2098e-124">Racionalizar a propriedade digital</span><span class="sxs-lookup"><span data-stu-id="2098e-124">Rationalize the digital estate</span></span>](rationalize.md)