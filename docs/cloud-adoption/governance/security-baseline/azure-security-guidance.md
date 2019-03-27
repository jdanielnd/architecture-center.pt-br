---
title: 'CAF: Guia de segurança do Azure'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Quais diretrizes de segurança a Microsoft fornece?
author: BrianBlanchard
ms.openlocfilehash: 845b02422e2df63425e29acdfd9b43b744934c9f
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241927"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-security-guidance-does-microsoft-provide"></a>Quais diretrizes de segurança a Microsoft fornece?

## <a name="security-guidance-and-tools"></a>Diretrizes e ferramentas de segurança

A Microsoft introduziu a [Plataforma de Confiança do Serviço](https://servicetrust.microsoft.com) e o Gerenciador de Conformidade para ajudar a:

- Superar os desafios de gerenciamento de conformidade
- Cumprir as responsabilidades para atender aos requisitos regulamentares
- Realizar auditorias de autoatendimento e avaliações de risco da utilização de serviços de nuvem corporativos

Essas ferramentas são projetadas para auxiliar as organizações a cumprirem as obrigações de conformidade complexas e melhorarem os recursos de proteção de dados ao escolherem utilizar os serviços do Microsoft Cloud.

A **STP (Plataforma de Confiança do Serviço)** fornece informações detalhadas e ferramentas para ajudar a atender às necessidades de uso dos serviços do Microsoft Cloud, incluindo Azure, Office 365, Dynamics 365 e Windows. A STP é uma solução completa para informações de segurança, regulamentação, conformidade e privacidade relacionadas ao Microsoft Cloud. É onde publicamos as informações e os recursos necessários para realizar avaliações de risco de autoatendimento de serviços de nuvem e ferramentas. A STP foi criada para ajudar a acompanhar as atividades de conformidade regulatória no Azure, incluindo:

- **Gerenciador de Conformidade**: O Gerenciador de Conformidade, uma ferramenta de avaliação de riscos baseada em fluxo de trabalho da Plataforma de Confiança do Serviço da Microsoft, permite rastrear, atribuir e verificar as atividades de conformidade regulatória da organização relacionadas aos serviços do Microsoft Cloud como o Office 365, Dynamics 365 e Azure. Você pode encontrar mais detalhes na próxima seção.
- **Documentos confiáveis**: Atualmente, há três categorias de guias que fornecem recursos abrangentes para avaliar o Microsoft Cloud, saiba mais sobre operações da Microsoft em segurança, conformidade e privacidade, que irão ajudá-lo no aprimoramento dos recursos de proteção de dados. Estão incluídos:
- **Relatórios de auditoria**: Os relatórios de auditoria permitem que você fique atualizado sobre as informações mais recentes relacionadas à conformidade, segurança e privacidade dos serviços do Microsoft Cloud. Isso inclui ISO, SOC, FedRAMP e outros relatórios de auditoria, notificações de avaliação e materiais relacionados a auditorias independentes de terceiros dos serviços do Microsoft Cloud como Azure, Office 365, Dynamics 365, e outros.
- **Guias de proteção de dados**: Os guias de proteção de dados fornecem informações sobre como os serviços do Microsoft Cloud protegem os dados e, como é possível gerenciar a segurança e a conformidade dos dados na nuvem da organização. Isso inclui white papers aprofundados que fornecem detalhes sobre como a Microsoft desenvolve e opera serviços de nuvem, perguntas frequentes, relatórios de avaliações de segurança de fim de ano, resultados de testes de penetração e diretrizes para ajudá-lo a realizar avaliações de risco e aprimorar os recursos de proteção de dados.
- **Blueprint de segurança e conformidade do Azure**: Os blueprints fornecem recursos para auxiliá-lo na criação e inicialização de aplicativos baseados em nuvem, contribuindo para o cumprimento de regulamentações e padrões rigorosos. Com mais certificações do que qualquer outro provedor de nuvem, você pode confiar na implantação das cargas de trabalho críticas no Azure com blueprints, que incluem:

- Visão geral e diretrizes específicas do setor
- Matriz de responsabilidades do cliente
- Arquiteturas de referência com modelos de risco
- Matrizes de implementação de controle
- Automação para implantar arquiteturas de referência
- Recursos de privacidade – Documentação para Avaliações de Impacto da Proteção de Dados, DSRs (Solicitações do Titular dos Dados) e Notificação de Violação de Dados são fornecidos para incorporar ao seu próprio programa de responsabilidade em suporte ao RGPD (Regulamento Geral sobre a Proteção de Dados).

- **Introdução ao RGPD**: Os produtos e serviços da Microsoft ajudam as organizações a atenderem aos requisitos de RGPD enquanto coletam ou processam dados pessoais. A STP foi projetada para fornecer informações sobre os recursos dos serviços da Microsoft que podem ser utilizados para atender aos requisitos específicos de RGPD. A documentação pode ajudá-lo na responsabilidade de GDPR e a reconhecer medidas técnicas e organizacionais. Documentação para Avaliações de Impacto da Proteção de Dados, DSRs (Solicitações do Titular dos Dados) e Notificação de Violação de Dados são fornecidos para incorporar ao seu próprio programa de responsabilidade em suporte ao RGPD.
  - **Solicitações do titular dos dados**: O RGPD concede aos indivíduos (ou titulares dos dados) determinados direitos relacionados com o processamento dos dados pessoais. Isso inclui o direito de corrigir dados imprecisos, apagar dados ou restringir o processamento, bem como receber dados e atender a uma solicitação para transmitir os dados a outro controlador.
  - **Violação de dados**: O RGPD exige requisitos de notificação para controladores de dados e processadores no caso de uma violação de dados pessoais. A STP fornece informações sobre como a Microsoft tenta evitar violações, como a Microsoft detecta uma violação e como a Microsoft irá responder no caso de uma violação e notificá-lo como um controlador de dados.
  - **Avaliação de Impacto da Proteção de Dados**: A Microsoft auxilia os controladores a concluírem as Avaliações de Impacto da Proteção de Dados do RGPD. O RGPD fornece uma lista abrangente de casos em que DPIAs devem ser realizadas, como processamento automatizado para fins de criação de perfis e atividades semelhantes, processamento em larga escala de categorias especiais de dados pessoais e monitoramento sistemático de uma área de acesso público em larga escala.
  - **Outros recursos**: Além das diretrizes de ferramentas abordadas nas seções anteriores, a STP também fornece outros recursos incluindo conformidade regional, recursos adicionais para o Centro de Conformidade e Segurança e perguntas frequentes sobre a Plataforma de Confiança do Serviço, Gerenciador de Conformidade e privacidade/RGPD.
- **Conformidade regional**: A STP fornece vários documentos de diretrizes e conformidade para serviços online da Microsoft para cumprir os requisitos de conformidade de diferentes regiões, incluindo República Tcheca, Polônia e Romênia.

## <a name="unique-intelligent-insights"></a>Insights inteligentes exclusivos

Na medida em que o volume e a complexidade dos sinais de segurança aumentam, determinar se esses sinais são ameaças credíveis para, em seguida, agir, demora muito tempo. A Microsoft oferece uma variedade incomparável de inteligência de segurança oferecida em escala de nuvem para ajudar a detectar e corrigir ameaças rapidamente.

## <a name="azure-threat-intelligence"></a>Inteligência contra ameaças do Azure

Ao usar a opção Inteligência contra Ameaças disponível na Central de Segurança, os administradores de TI poderão identificar as ameaças à segurança no ambiente. Por exemplo, eles podem identificar se determinado computador faz parte de um botnet. Os computadores podem se tornar nós em um botnet quando os invasores instalam de forma ilícita malware que conecta secretamente o computador ao comando e ao controle. A inteligência contra ameaças também pode identificar possíveis ameaças recebidas de canais de comunicação escondidos, como a dark Web.

Para criar essa inteligência contra ameaças, a Central de Segurança usa dados recebidos de várias fontes da Microsoft. A Central de Segurança usa-os para identificar possíveis ameaças contra o ambiente. O painel Inteligência contra Ameaças é composto de três opções principais:

- Tipos de ameaças detectados
- Origem da ameaça
- Mapa de inteligência contra ameaças

## <a name="machine-learning-in-azure-security-center"></a>Machine Learning na Central de Segurança do Azure

A Central de Segurança do Azure analisa profundamente uma grande variedade de dados de várias soluções da Microsoft e de parceiros para ajudá-lo a obter maior segurança. Para aproveitar esses dados, a empresa usa ciência de dados e aprendizado de máquina para prevenção contra ameaças, detecção e, eventualmente, investigação.

Em geral, o Azure Machine Learning ajuda a alcançar dois resultados:

## <a name="next-generation-detection"></a>Detecção de próxima geração

Os invasores estão cada vez mais automatizados e sofisticados. Eles também usam ciência de dados. Eles fazem engenharia reversa de proteções e desenvolvem sistemas que dão suporte a mutações no comportamento. Eles mascaram as atividades como ruídos e aprendem rapidamente com os erros. O aprendizado de máquina ajuda-nos a responder a esses desenvolvimentos.

## <a name="simplified-security-baseline"></a>Linha de Base de Segurança simplificada

Tomar decisões de segurança eficazes não é fácil. Isso exige experiência e especialização em segurança. Embora algumas grandes organizações tenham especialistas nessa área, muitas empresas não. O Azure Machine Learning permite que os clientes se beneficiem com a sabedoria de outras organizações ao tomar decisões de segurança.

## <a name="behavioral-analytics"></a>Análise comportamental

A análise de comportamento é uma técnica que analisa e compara dados em uma coleção de padrões conhecidos. No entanto, esses padrões não são assinaturas simples. Os padrões são determinados por meio de algoritmos complexos de aprendizado de máquina aplicados a conjuntos de dados massivos. Eles também são determinados pela análise cuidadosa de comportamentos mal-intencionados por analistas especialistas. A Central de Segurança do Azure pode usar a análise de comportamento para identificar recursos comprometidos baseado na análise dos logs de máquina virtual, dos logs de dispositivo de rede virtual, dos logs da malha, dos despejos de memória e de outras fontes.
