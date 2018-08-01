---
title: 'Adotando o Azure: conceitos básicos'
description: Descreve o nível da base de conhecimento que uma empresa precisa para adotar o Azure
author: petertay
ms.openlocfilehash: b5a0a4a2c4ed1d06c97774b0eca643a89a5a2110
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229160"
---
# <a name="adopting-microsoft-azure-foundational"></a>Adotando o Microsoft Azure: conceitos básicos

Para uma organização iniciante em tecnologias de nuvem, pode ser difícil decidir o lugar certo para iniciar sua jornada de adoção. O objetivo do estágio básico de adoção é fornecer um ponto de partida. Depois que as pessoas dentro da organização já tiverem trabalhado este estágio, elas terão todos os dados de conhecimento e habilidades necessárias para implantar os recursos de computação para uma carga de trabalho simples no Azure. 

> [!NOTE]
> Este guia não abrange o desenvolvimento de aplicativos. Para obter mais informações sobre como desenvolver aplicativos no Azure, consulte o [Guia de arquitetura de aplicativo do Azure](/azure/architecture/guide/).

O público para este estágio do guia é composto pelas seguintes personas dentro de sua organização:

- *Financeiro:* proprietário do compromisso financeiro para o Azure, responsável por desenvolver políticas e procedimentos para controlar os custos de consumo de recursos, incluindo cobrança e estorno.
- *TI Central:* responsável pela governança dos recursos de nuvem da sua organização, incluindo o gerenciamento de recursos e o acesso, bem como a integridade e o monitoramento da carga de trabalho.
- *Proprietários da carga de trabalho:* todas as funções de desenvolvimento que estão envolvidas na implantação de cargas de trabalho no Azure, incluindo desenvolvedores, testadores e técnicos de build.

## <a name="section-1-azure-basics"></a>Seção 1: Noções básicas do Azure

Esta seção introdutória destina-se às personas de *finanças* e da *TI central*. O foco desta seção é adquirir uma compreensão básica de [como o Azure funciona](azure-explainer.md) em preparação para aprender mais sobre o [conceito de governança de nuvem](governance-explainer.md). Ele também pode ser útil para que os *proprietários da carga de trabalho* em sua organização possam revisar este conteúdo e, assim, entender como o acesso aos recursos é gerenciado.

## <a name="section-2-governance-design-guide"></a>Seção 2: Guia de design de governança

Agora que você entende como funciona o Azure e os conceitos básicos de governança de nuvem, a primeira etapa na adoção do Azure está aprender sobre o [gerenciamento de acesso de recursos](azure-resource-access.md) no Azure. Este artigo descreve os serviços do Azure para fazer solicitações de acesso de recursos e os controles usados para validar a essas solicitações.

A próxima etapa é aprender como [criar um modelo de governança](governance-how-to.md) para uma única equipe. Este artigo descreve como configurar os serviços de gerenciamento de acesso de recursos e os controles sobre os quais você aprendeu anteriormente.

## <a name="section-3-implementing-a-basic-resource-access-management-model"></a>Seção 3: Implementação de um modelo de gerenciamento de acesso a recursos básico

A etapa final na sua jornada de adoção é saber como implementar o modelo de governança criado anteriormente. 

Para começar, sua organização exige uma conta do Azure. Se sua organização tiver um [Microsoft Enterprise Agreement](https://www.microsoft.com/licensing/licensing-programs/enterprise.aspx) existente que não inclui o Azure, este pode ser adicionado mediante uma confirmação de pagamento adiantado. Consulte [licenciamento do Azure para a empresa](https://azure.microsoft.com/pricing/enterprise-agreement/) para obter mais informações. 

Ao criar sua conta do Azure, especifique uma pessoa em sua organização para ser o **proprietário da conta** do Azure. Em seguida, um locatário do Azure Active Directory (Azure AD) é criado por padrão. O **proprietário da conta** do Azure deve [criar a conta de usuário](/azure/active-directory/add-users-azure-active-directory) para a pessoa em sua organização que é o **proprietário da carga de trabalho**. 

Em seguida, o **proprietário da conta** do Azure deve [criar uma assinatura](https://docs.microsoft.com/partner-center/create-a-new-subscription) e [associar o locatário do Azure AD](/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory) a ele.

Por fim, agora que a assinatura está criada e seu locatário do Azure Active Directory está associado a ela, você poderá [adicionar o **proprietário da carga de trabalho** à assinatura com a função de **proprietário** interna](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-for-a-subscription-in-azure-portal).

## <a name="section-4-deploy-a-basic-workload-architecture-to-azure"></a>Seção 4: Implantar uma arquitetura de carga de trabalho básica no Azure

O público-alvo desta seção é a persona do *proprietário da carga de trabalho*. *Os proprietários da carga de trabalho* definem a computação e os requisitos de rede para suas cargas de trabalho, selecionam os recursos corretos para atender a esses requisitos e implantam-nos no Azure. 

Para o estágio básico de adoção, o proprietário da carga de trabalho pode selecionar um aplicativo Web básico ou uma rede virtual (VNet) e a máquina virtual (VM). Para obter mais informações sobre esses tipos diferentes de opções de computação no Azure, examine a [visão geral das opções de computação do Azure](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json).

Independentemente de qual opção de computação for selecionada, cada uma dessas implantações requer um **grupo de recursos**. Seu **proprietário da carga de trabalho** deve [criar o grupo de recursos](/azure/azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy). Como parte da implantação, o **proprietário da carga de trabalho** especifica um nome para o grupo de recursos. Este nome será usado nas seções a seguir.

### <a name="basic-web-application-paas"></a>Aplicativo Web básico (PaaS)

Para um aplicativo Web básico, selecione um dos guias de início rápido de 5 minutos da [documentação de aplicativos Web](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json) e siga as etapas. 

> [!NOTE]
> Alguns dos guias de início rápido implantarão um grupo de recursos por padrão. Neste caso, o **proprietário da carga de trabalho** não precisa criar um grupo de recursos explicitamente. Caso contrário, implante o aplicativo Web para o grupo de recursos criado acima.

Após a implantação de sua carga de trabalho simples, você pode aprender mais sobre as práticas comprovadas para a implantação de um [aplicativo Web básico](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) no Azure.

### <a name="single-windows-or-linux-vm-iaas"></a>VM única Windows ou Linux (IaaS)

Para uma carga de trabalho simples que é executada em uma máquina virtual, a primeira etapa é implantar uma rede virtual. Todos os recursos de IaaS no Azure como máquinas virtuais, balanceadores de carga e gateways exigem uma rede virtual. Saiba mais sobre [redes virtuais do Azure](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json) e, em seguida, siga as etapas para [implantar uma rede virtual no Azure usando o portal](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Quando você especifica as configurações para a rede virtual no portal do Azure, especifique o nome do grupo de recursos criado acima.

A próxima etapa é decidir se deseja implantar uma VM única Windows ou Linux. Para uma VM Windows, siga as etapas para [implantar uma VM Windows no Azure com o portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Novamente, quando você especifica as configurações para a máquina virtual no portal do Azure, especifique o nome do grupo de recursos criado acima.

Depois que você tiver seguido as etapas e implantado a máquina virtual, você pode aprender sobre as [práticas comprovadas para executar uma VM Windows no Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json). Para uma VM Linux, siga as etapas para [implantar uma VM Linux no Azure com o portal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Você também pode aprender mais sobre [práticas comprovadas para executar uma VM Linux no Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).

## <a name="next-steps"></a>Próximas etapas

A próxima etapa na preparação da nuvem é o [ **estágio intermediário**](../intermediate-stage/overview.md). No estágio intermediário, você aprenderá como estender sua rede local que executa várias cargas de trabalho e modelos de governança para várias equipes.